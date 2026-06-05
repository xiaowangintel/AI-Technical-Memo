# tensor_impl.cpp — Code Analysis / 代码分析

## Source / 来源
- File / 文件: `torch/csrc/lazy/core/tensor_impl.cpp`
- Repository / 仓库: `pytorch`
- Purpose / 作用:
  - EN: Implements the shared Lazy Tensor IR, hashing, shape, and execution support.
  - CN: 实现通用 Lazy Tensor IR、哈希、形状和执行支撑逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-39
```cpp
 1 | #include <torch/csrc/lazy/core/tensor_impl.h>
 2 | 
 3 | #include <c10/core/impl/DeviceGuardImplInterface.h>
 4 | #include <c10/macros/Macros.h>
 5 | #include <c10/util/irange.h>
 6 | #include <torch/csrc/lazy/core/tensor_util.h>
 7 | 
 8 | namespace torch::lazy {
 9 | namespace {
10 | 
11 | // LTCGuardImpl is used by CompositeExplicitAutograd ops or eager fallbacks to
12 | // make sure that some particular tensors within the life scope of the guard are
13 | // on the same device. For example, in RegisterCompositeExplicitAutograd.cpp,
14 | // outputs of each op are examined if they are on same device as the supplied
15 | // TensorOptions. For more information, see DeviceGuard.h. For ops that have LTC
16 | // native function implementations, this guard is omitted.
17 | thread_local c10::Device g_device(c10::DeviceType::Lazy);
18 | 
19 | struct LTCGuardImpl : public c10::impl::DeviceGuardImplInterface {
20 |   at::DeviceType type() const override {
21 |     return at::DeviceType::Lazy;
22 |   }
23 | 
24 |   c10::Device exchangeDevice(c10::Device device) const override {
25 |     TORCH_INTERNAL_ASSERT(device.type() == c10::DeviceType::Lazy);
26 |     auto old_device = g_device;
27 |     g_device = device;
28 |     return old_device;
29 |   }
30 | 
31 |   c10::Device getDevice() const override {
32 |     return g_device;
33 |   }
34 | 
35 |   void setDevice(c10::Device device) const override {
36 |     TORCH_INTERNAL_ASSERT(device.type() == c10::DeviceType::Lazy);
37 |     g_device = device;
38 |   }
39 | 
```
- EN: Brings in project headers such as `<torch/csrc/lazy/core/tensor_impl.h>`, `<c10/core/impl/DeviceGuardImplInterface.h>`, `<c10/macros/Macros.h>`, `<c10/util/irange.h>` so this section can use their types, APIs, or macros. Places the implementation in namespace scopes (`torch::lazy`) so ownership matches the PyTorch subsystem layout. Defines or extends data abstractions such as `LTCGuardImpl` that structure the state handled by this file. Implements routines such as `g_device`, `type`, `exchangeDevice`, `getDevice`, `setDevice` that expose the key API or control flow of this region.
- CN: 这里引入了项目头文件，例如 `<torch/csrc/lazy/core/tensor_impl.h>`、`<c10/core/impl/DeviceGuardImplInterface.h>`、`<c10/macros/Macros.h>`、`<c10/util/irange.h>`，使本段代码能够使用相关类型、API 或宏。 把实现放入命名空间作用域（`torch::lazy`），使其归属与 PyTorch 子系统布局保持一致。 定义或扩展了 `LTCGuardImpl` 等数据抽象，用来组织本文件处理的状态。 实现了 `g_device`、`type`、`exchangeDevice`、`getDevice`、`setDevice` 等例程，它们构成了这一段的关键 API 或控制流程。

### Lines 40-76
```cpp
40 |   void uncheckedSetDevice(c10::Device device) const noexcept override {
41 |     TORCH_INTERNAL_ASSERT(device.type() == c10::DeviceType::Lazy);
42 |     g_device = device;
43 |   }
44 | 
45 |   c10::Stream getStream(c10::Device device) const noexcept override {
46 |     TORCH_INTERNAL_ASSERT(device.type() == c10::DeviceType::Lazy);
47 |     return c10::Stream(c10::Stream::DEFAULT, device);
48 |   }
49 | 
50 |   c10::Stream exchangeStream(c10::Stream _unused) const noexcept override {
51 |     return c10::Stream(c10::Stream::DEFAULT, g_device);
52 |   }
53 | 
54 |   c10::DeviceIndex deviceCount() const noexcept override {
55 |     // This will get called when autograd initializes its device pool
56 |     // regardless whether we have a backend registered aforehand.
57 |     if (!hasBackend()) {
58 |       return 0;
59 |     }
60 | 
61 |     return static_cast<c10::DeviceIndex>(
62 |         getBackend()->GetBackendDevices().size());
63 |   }
64 | };
65 | 
66 | C10_REGISTER_GUARD_IMPL(Lazy, LTCGuardImpl)
67 | 
68 | } // namespace
69 | 
70 | // TODO(whc) when do we want to clone vs share?
71 | LTCTensorImpl::LTCTensorImpl(const LazyTensorPtr& tensor)
72 |     : LTCTensorImpl(LazyTensor(*tensor)) {}
73 | 
74 | LTCTensorImpl::LTCTensorImpl(const LazyTensor& tensor)
75 |     : LTCTensorImpl(LazyTensor(tensor)) {}
76 | 
```
- EN: At the statement level, this block returns computed state or forwards values to callers.
- CN: 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 77-116
```cpp
 77 | LTCTensorImpl::LTCTensorImpl(LazyTensor&& tensor)
 78 |     : c10::TensorImpl(
 79 |           c10::DispatchKeySet{
 80 |               c10::DispatchKey::Lazy,
 81 |               c10::DispatchKey::AutogradLazy},
 82 |           c10::scalarTypeToTypeMeta(tensor.dtype()),
 83 |           backendDeviceToAtenDevice(tensor.GetDevice())),
 84 |       tensor_(c10::make_intrusive<LazyTensor>(std::move(tensor))) {
 85 |   set_custom_sizes_strides(SizesStridesPolicy::CustomSizes);
 86 | }
 87 | 
 88 | void LTCTensorImpl::set_tensor(const LazyTensorPtr& lazy_tensor) {
 89 |   tensor_ = c10::make_intrusive<LazyTensor>(*lazy_tensor);
 90 |   generation_ = 0;
 91 | }
 92 | 
 93 | c10::intrusive_ptr<c10::TensorImpl> LTCTensorImpl::shallow_copy_and_detach(
 94 |     const c10::VariableVersion& version_counter,
 95 |     bool allow_tensor_metadata_change) const {
 96 |   auto impl = c10::make_intrusive<LTCTensorImpl>(tensor_);
 97 |   copy_tensor_metadata(
 98 |       /*src_impl=*/this,
 99 |       /*dest_impl=*/impl.get(),
100 |       /*version_counter=*/version_counter,
101 |       /*allow_tensor_metadata_change=*/allow_tensor_metadata_change);
102 |   return impl;
103 | }
104 | 
105 | c10::intrusive_ptr<c10::TensorImpl> LTCTensorImpl::shallow_copy_and_detach(
106 |     c10::VariableVersion&& version_counter,
107 |     bool allow_tensor_metadata_change) const {
108 |   auto impl = c10::make_intrusive<LTCTensorImpl>(tensor_);
109 |   copy_tensor_metadata(
110 |       /*src_impl=*/this,
111 |       /*dest_impl=*/impl.get(),
112 |       /*version_counter=*/std::move(version_counter),
113 |       /*allow_tensor_metadata_change=*/allow_tensor_metadata_change);
114 |   return impl;
115 | }
116 | 
```
- EN: At the statement level, this block declares abstract hooks that derived backends must provide; returns computed state or forwards values to callers; uses move semantics to avoid unnecessary copies.
- CN: 在语句层面，这一段声明了需要由派生后端提供的抽象钩子；返回计算状态或把值转交给调用方；使用移动语义来避免不必要的拷贝。

### Lines 117-141
```cpp
117 | void LTCTensorImpl::shallow_copy_from(
118 |     const c10::intrusive_ptr<TensorImpl>& impl) {
119 |   LTCTensorImpl* ltc_impl = dynamic_cast<LTCTensorImpl*>(impl.get());
120 |   TORCH_INTERNAL_ASSERT(ltc_impl);
121 |   copy_tensor_metadata(
122 |       /*src_impl=*/ltc_impl,
123 |       /*dest_impl=*/this,
124 |       /*version_counter=*/version_counter(),
125 |       /*allow_tensor_metadata_change=*/allow_tensor_metadata_change());
126 |   ltc_impl->tensor_->ShallowCopyTo(tensor_);
127 |   generation_ = 0;
128 | }
129 | 
130 | c10::SymIntArrayRef LTCTensorImpl::sym_strides_custom() const {
131 |   return c10::fromIntArrayRefKnownNonNegative(strides_custom());
132 | }
133 | 
134 | c10::SymIntArrayRef LTCTensorImpl::sym_sizes_custom() const {
135 |   return c10::fromIntArrayRefKnownNonNegative(sizes_custom());
136 | }
137 | 
138 | c10::SymInt LTCTensorImpl::sym_numel_custom() const {
139 |   return numel_custom();
140 | }
141 | 
```
- EN: Implements routines such as `numel_custom` that expose the key API or control flow of this region. At the statement level, this block declares abstract hooks that derived backends must provide; returns computed state or forwards values to callers.
- CN: 实现了 `numel_custom` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段声明了需要由派生后端提供的抽象钩子；返回计算状态或把值转交给调用方。

### Lines 142-178
```cpp
142 | void LTCTensorImpl::setup_size_properties() {
143 |   size_t generation = tensor_->generation();
144 |   if (generation != generation_) {
145 |     // Fill up the basic dimension data members which the base class
146 |     // implementation uses in its APIs.
147 |     auto shape = tensor_->shape();
148 |     // We can't call refresh_numel() given we override sizes() too.
149 |     numel_ = static_cast<int64_t>(shape.Get().numel());
150 |     sizes_and_strides_.set_sizes(shape.Get().sizes());
151 |     // We can't call empty_tensor_restride(c10::MemoryFormat::Contiguous) given
152 |     // we override sizes() too.
153 |     auto updated_strides = ComputeArrayStrides(shape.Get().sizes());
154 |     for (const auto i : c10::irange(updated_strides.size())) {
155 |       sizes_and_strides_.stride_at_unchecked(i) = updated_strides[i];
156 |     }
157 |     generation_ = generation;
158 |   }
159 | }
160 | 
161 | at::IntArrayRef LTCTensorImpl::sizes_custom() const {
162 |   // NOLINTNEXTLINE(cppcoreguidelines-pro-type-const-cast)
163 |   const_cast<LTCTensorImpl*>(this)->setup_size_properties();
164 |   return sizes_default();
165 | }
166 | 
167 | at::IntArrayRef LTCTensorImpl::strides_custom() const {
168 |   // NOLINTNEXTLINE(cppcoreguidelines-pro-type-const-cast)
169 |   const_cast<LTCTensorImpl*>(this)->setup_size_properties();
170 |   return strides_default();
171 | }
172 | 
173 | int64_t LTCTensorImpl::dim_custom() const {
174 |   // NOLINTNEXTLINE(cppcoreguidelines-pro-type-const-cast)
175 |   const_cast<LTCTensorImpl*>(this)->setup_size_properties();
176 |   return dim_default();
177 | }
178 | 
```
- EN: Implements routines such as `sizes_default`, `strides_default`, `dim_default` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `sizes_default`、`strides_default`、`dim_default` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 179-216
```cpp
179 | int64_t LTCTensorImpl::numel_custom() const {
180 |   // NOLINTNEXTLINE(cppcoreguidelines-pro-type-const-cast)
181 |   const_cast<LTCTensorImpl*>(this)->setup_size_properties();
182 |   return numel_default();
183 | }
184 | 
185 | int64_t LTCTensorImpl::storage_offset_custom() const {
186 |   return 0;
187 | }
188 | 
189 | bool LTCTensorImpl::is_strides_like_custom(
190 |     c10::MemoryFormat memory_format) const {
191 |   TORCH_INTERNAL_ASSERT(memory_format != at::MemoryFormat::Contiguous);
192 |   return false;
193 | }
194 | 
195 | c10::SymBool LTCTensorImpl::sym_is_non_overlapping_and_dense_custom() const {
196 |   // This should be true, but false as a temporary fix for a PyTorch core issue,
197 |   // according to https://github.com/pytorch/xla/pull/2682.
198 |   return false;
199 | }
200 | 
201 | c10::SymBool LTCTensorImpl::sym_is_contiguous_custom(
202 |     c10::MemoryFormat _unused) const {
203 |   // TODO(ezyang): I don't think this branch is actually necessary
204 |   // TODO(ezyang): I don't think this logic is right, shouldn't we pass on
205 |   // the memory format?
206 |   const auto data = tensor_->CurrentTensorData();
207 |   if (data.has_value()) {
208 |     return data->is_contiguous();
209 |   }
210 |   // Only check that the storage is already contiguous.
211 |   TORCH_CHECK(is_contiguous_, "Non-contiguous storage for lazy tensor");
212 |   // TODO: I don't think logic is right, we should check the requested memory
213 |   // format before returning true
214 |   return true;
215 | }
216 | 
```
- EN: Implements routines such as `numel_default` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 实现了 `numel_default` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

### Lines 217-217
```cpp
217 | } // namespace torch::lazy
```
- EN: Continues the file's main role: Implements the shared Lazy Tensor IR, hashing, shape, and execution support.
- CN: 继续承担本文件的主要职责：实现通用 Lazy Tensor IR、哈希、形状和执行支撑逻辑。

## Key Concepts / 关键概念

- **Primary types / 主要类型**
  - EN: `LTCGuardImpl`.
  - CN: `LTCGuardImpl`。
- **Important routines / 重要例程**
  - EN: `g_device`, `type`, `exchangeDevice`, `getDevice`, `setDevice`, `numel_custom`, `sizes_default`, `strides_default`.
  - CN: `g_device`、`type`、`exchangeDevice`、`getDevice`、`setDevice`、`numel_custom`、`sizes_default`、`strides_default`。
- **Subsystem scope / 子系统作用域**
  - EN: The code lives under `torch::lazy`, indicating the owning PyTorch subsystem.
  - CN: 代码位于 `torch::lazy` 命名空间下，这说明了其所属的 PyTorch 子系统。

## Dependencies / 依赖关系

- Local includes / 本地头文件: `<torch/csrc/lazy/core/tensor_impl.h>`, `<c10/core/impl/DeviceGuardImplInterface.h>`, `<c10/macros/Macros.h>`, `<c10/util/irange.h>`, `<torch/csrc/lazy/core/tensor_util.h>`
- External includes / 外部头文件: None / 无
- Relationship / 关系:
  - EN: The unit relies on neighboring PyTorch headers for subsystem-specific types, dispatcher hooks, and utility helpers. Its exported or visible routines are then consumed by callers in the same namespace and by higher-level runtime paths elsewhere in PyTorch.
  - CN: 该编译单元依赖相邻的 PyTorch 头文件，以获得子系统类型、分发器钩子和辅助工具。 随后，这里导出或可见的例程会被同一命名空间中的调用方，以及 PyTorch 其他位置的更高层运行时路径所使用。
