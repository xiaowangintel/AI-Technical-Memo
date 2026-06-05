# tensor.h — Code Analysis / 代码分析

## Source / 来源
- File / 文件: `torch/csrc/lazy/core/tensor.h`
- Repository / 仓库: `pytorch`
- Purpose / 作用:
  - EN: Implements the shared Lazy Tensor IR, hashing, shape, and execution support.
  - CN: 实现通用 Lazy Tensor IR、哈希、形状和执行支撑逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-32
```cpp
 1 | #pragma once
 2 | 
 3 | #include <c10/core/SymNodeImpl.h>
 4 | #include <c10/util/intrusive_ptr.h>
 5 | #include <torch/csrc/lazy/backend/backend_data.h>
 6 | #include <torch/csrc/lazy/backend/backend_device.h>
 7 | #include <torch/csrc/lazy/core/ir.h>
 8 | #include <torch/csrc/lazy/core/util.h>
 9 | 
10 | namespace torch::lazy {
11 | 
12 | class TORCH_API SymNodeImpl : public c10::SymNodeImpl {
13 |  public:
14 |   SymNodeImpl(NodePtr ptr) : node_(std::move(ptr)) {}
15 |   NodePtr node_;
16 | };
17 | 
18 | class LazyTensor;
19 | using LazyTensorPtr = c10::intrusive_ptr<LazyTensor>;
20 | 
21 | class TORCH_API LazyTensor : public c10::intrusive_ptr_target {
22 |  public:
23 |   // This is the core lazy tensor data structure where all the tensor data is
24 |   // held. The lazy tensor is nothing more than a shared pointer to a Data
25 |   // object.
26 |   struct Data {
27 |     Data(BackendDataPtr handle, BackendDevice device)
28 |         : handle(std::move(handle)),
29 |           device(std::move(device)),
30 |           unique_id(GetNextTensorId()) {}
31 |     Data(Value ir_value, BackendDevice device)
32 |         : ir_value(std::move(ir_value)),
```
- EN: Brings in project headers such as `<c10/core/SymNodeImpl.h>`, `<c10/util/intrusive_ptr.h>`, `<torch/csrc/lazy/backend/backend_data.h>`, `<torch/csrc/lazy/backend/backend_device.h>` so this section can use their types, APIs, or macros. Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Places the implementation in namespace scopes (`torch::lazy`) so ownership matches the PyTorch subsystem layout. Defines or extends data abstractions such as `SymNodeImpl`, `LazyTensor`, `Data` that structure the state handled by this file.
- CN: 这里引入了项目头文件，例如 `<c10/core/SymNodeImpl.h>`、`<c10/util/intrusive_ptr.h>`、`<torch/csrc/lazy/backend/backend_data.h>`、`<torch/csrc/lazy/backend/backend_device.h>`，使本段代码能够使用相关类型、API 或宏。 使用预处理指令来控制编译开关、平台分支或导出声明。 把实现放入命名空间作用域（`torch::lazy`），使其归属与 PyTorch 子系统布局保持一致。 定义或扩展了 `SymNodeImpl`、`LazyTensor`、`Data` 等数据抽象，用来组织本文件处理的状态。

### Lines 33-67
```cpp
33 |           device(std::move(device)),
34 |           unique_id(GetNextTensorId()) {}
35 |     Data(at::Tensor tensor_data, BackendDevice device)
36 |         : tensor_data(std::move(tensor_data)),
37 |           device(std::move(device)),
38 |           unique_id(GetNextTensorId()) {}
39 |     // TODO(alanwaketan): Remove this ctor. This is a
40 |     // temporary ctor to ease XLA LTC migration. It depends on
41 |     // XLA's Functionalization integration.
42 |     Data(BackendDevice device)
43 |         : device(std::move(device)), unique_id(GetNextTensorId()) {}
44 | 
45 |     Data(Data&& other) = delete;
46 |     Data(const Data&) = delete;
47 |     Data& operator=(const Data&) = delete;
48 |     Data& operator=(Data&&) = delete;
49 |     virtual ~Data();
50 | 
51 |     BackendDataPtr handle;
52 |     Value ir_value;
53 |     std::optional<at::Tensor> tensor_data;
54 |     // NOLINTNEXTLINE(cppcoreguidelines-avoid-const-or-ref-data-members)
55 |     const BackendDevice device;
56 |     // NOLINTNEXTLINE(cppcoreguidelines-avoid-const-or-ref-data-members)
57 |     const int64_t unique_id = 0;
58 |     size_t generation = 1;
59 |   };
60 | 
61 |   static LazyTensorPtr Create(
62 |       const at::Tensor& tensor,
63 |       const BackendDevice& device);
64 |   static LazyTensorPtr Create(Value ir_value, const BackendDevice& device);
65 |   static LazyTensorPtr Create(const BackendDataPtr& handle);
66 |   static LazyTensorPtr Create(std::shared_ptr<Data> data);
67 | 
```
- EN: Implements routines such as `Create` that expose the key API or control flow of this region. At the statement level, this block declares abstract hooks that derived backends must provide; uses move semantics to avoid unnecessary copies.
- CN: 实现了 `Create` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段声明了需要由派生后端提供的抽象钩子；使用移动语义来避免不必要的拷贝。

### Lines 68-107
```cpp
 68 |   // The default ctor previously created a null LazyTensor (one with no 'data'
 69 |   // obj). Creating a null LazyTensor is no longer possible, since the same can
 70 |   // be achieved by creating a null LazyTensorPtr and it is way too confusing to
 71 |   // have to check both lazy_tensor_ptr && *lazy_tensor_ptr, so everywhere that
 72 |   // used to rely on a LazyTensor obj with a null Data can now rely on a null
 73 |   // LazyTensorPtr instead.
 74 |   LazyTensor() = delete;
 75 |   LazyTensor(const LazyTensor&) = default;
 76 |   LazyTensor(LazyTensor&&) noexcept = default;
 77 |   LazyTensor& operator=(const LazyTensor&) = default;
 78 |   LazyTensor& operator=(LazyTensor&&) noexcept = default;
 79 | 
 80 |   ~LazyTensor() override = default;
 81 | 
 82 |   size_t generation() const {
 83 |     return data()->generation;
 84 |   }
 85 | 
 86 |   // Override it to use your own Shape.
 87 |   virtual int64_t size(int64_t dim) const;
 88 | 
 89 |   // Override it to use your own graph executor.
 90 |   virtual at::Tensor ToTensor(bool detached);
 91 | 
 92 |   void ShallowCopyTo(const LazyTensorPtr& dest) const;
 93 | 
 94 |   // Assigns the tensor value to the lazy tensor.
 95 |   void SetTensor(at::Tensor tensor);
 96 | 
 97 |   void UpdateFromTensor(const at::Tensor& tensor, bool sync);
 98 |   void UpdateFromTensorOut(const at::Tensor& tensor);
 99 |   void UpdateFromTensorOut(const LazyTensorPtr& tensor);
100 | 
101 |   const std::shared_ptr<Data>& data() const;
102 | 
103 |   // Override it to use your own type conversion.
104 |   virtual at::ScalarType dtype() const;
105 | 
106 |   MaybeRef<Shape> shape() const;
107 | 
```
- EN: Implements routines such as `generation`, `size`, `ToTensor`, `ShallowCopyTo`, `SetTensor` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `generation`、`size`、`ToTensor`、`ShallowCopyTo`、`SetTensor` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 108-146
```cpp
108 |   const BackendDevice& GetDevice() const;
109 |   int64_t GetUniqueId() const;
110 | 
111 |   // Fetches the data behind the tensor. If the tensor has a graph defining
112 |   // its current value, executes the graph and fetches the data result.
113 |   BackendDataPtr GetDataHandle();
114 | 
115 |   // Fetches the current value of the data, which can be missing (nullptr)
116 |   // in case the tensor has a graph defining its current value,
117 |   BackendDataPtr CurrentDataHandle() const;
118 | 
119 |   void SetDataHandle(BackendDataPtr handle);
120 |   void SetDataHandle(BackendDataPtr handle, bool sync);
121 | 
122 |   // Retrieves the current IR Node, or nullptr in case no active IR Node is
123 |   // available.
124 |   Value CurrentIrValue() const;
125 | 
126 |   // Retrieves the IR Node representing this LazyTensor. One will be created if
127 |   // missing. Note that although this is a const API, it actually changes the
128 |   // internal state of the object.
129 |   Value GetIrValue() const;
130 | 
131 |   void SetIrValue(Value ir_value);
132 |   void SetInPlaceIrValue(Value ir_value);
133 | 
134 |   std::optional<at::Tensor> CurrentTensorData() const;
135 | 
136 |   std::vector<LazyTensorPtr> MakeOutputTensors(const NodePtr& node) const;
137 | 
138 |   LazyTensorPtr CopyTensorToDevice(const BackendDevice& device);
139 | 
140 |   // Applies the queue of operations in preparation for using the data.
141 |   // Override it to use your own graph executor.
142 |   virtual void ApplyPendingGraph();
143 | 
144 |   // Override it to set extra information.
145 |   virtual void AssignIrValue(Value ir_value) const;
146 | 
```
- EN: Declares routines such as `GetDevice`, `GetUniqueId`, `GetDataHandle`, `CurrentDataHandle`, `SetDataHandle` that expose the key API or control flow of this region.
- CN: 声明了 `GetDevice`、`GetUniqueId`、`GetDataHandle`、`CurrentDataHandle`、`SetDataHandle` 等例程，它们构成了这一段的关键 API 或控制流程。

### Lines 147-184
```cpp
147 |  protected:
148 |   explicit LazyTensor(std::shared_ptr<Data> data);
149 | 
150 |   void SetTensorData(at::Tensor tensor_data);
151 | 
152 |   // We build a graph accumulating operations, but at a given point we
153 |   // need to force a rendering, otherwise the graph can grow without control.
154 |   // Think:
155 |   //   for i in range(0, 100000):
156 |   //     a = a + b
157 |   void TryLimitGraphSize();
158 | 
159 |   // Override it to instantiate your own data.
160 |   virtual Value GetIrValueForTensor(
161 |       const at::Tensor& tensor,
162 |       const BackendDevice& device) const;
163 | 
164 |   Value CreateTensorNode(const BackendDataPtr& data, bool read_only) const;
165 | 
166 |  private:
167 |   LazyTensor(const at::Tensor& tensor, const BackendDevice& device);
168 |   LazyTensor(Value ir_value, const BackendDevice& device);
169 |   explicit LazyTensor(const BackendDataPtr& handle);
170 | 
171 |   static int64_t GetNextTensorId();
172 | 
173 |   std::shared_ptr<Data> data_;
174 | };
175 | 
176 | // Utils to convert at::Tensor to LazyTensor, and vice versa.
177 | 
178 | // Section 0: c10::Tensorlist ==> lazy::TensorList
179 | // note: GetTensorList is not totally parallel to GetLtcTensor; A TensorList
180 | // skips
181 | //       the LazyTensor wrappers, assuming that the list of underlying IR nodes
182 | //       is actually more useful for downstream computations.  TBD.
183 | TORCH_API torch::lazy::Value GetTensorList(at::ITensorListRef tensors);
184 | 
```
- EN: Declares routines such as `LazyTensor`, `SetTensorData`, `TryLimitGraphSize`, `GetIrValueForTensor`, `CreateTensorNode` that expose the key API or control flow of this region. At the statement level, this block stores long-lived member state for later calls.
- CN: 声明了 `LazyTensor`、`SetTensorData`、`TryLimitGraphSize`、`GetIrValueForTensor`、`CreateTensorNode` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段保存供后续调用使用的长期成员状态。

### Lines 185-222
```cpp
185 | // Section 1: at::Tensor => LazyTensor.
186 | // Extracts the LazyTensor out of an at::Tensor. Returns a null LazyTensor
187 | // if the tensor is not a lazy tensor.
188 | TORCH_API LazyTensorPtr TryGetLtcTensor(const at::Tensor& tensor);
189 | 
190 | // Extracts the LazyTensor out of an at::Tensor. Throws an exception
191 | // if the tensor is not a lazy tensor.
192 | TORCH_API LazyTensorPtr GetLtcTensor(const at::Tensor& tensor);
193 | 
194 | // Same as above, applied to a list of tensors.
195 | TORCH_API std::vector<LazyTensorPtr> GetLtcTensors(
196 |     c10::ArrayRef<at::Tensor> tensors);
197 | 
198 | // If tensor is a lazy tensor type, returns the LazyTensor embedded within it,
199 | // otherwise creates a new lazy tensor type with tensor as data.
200 | TORCH_API LazyTensorPtr GetOrCreateLtcTensor(
201 |     const std::optional<at::Tensor>& tensor,
202 |     const BackendDevice& device);
203 | 
204 | TORCH_API LazyTensorPtr GetLtcTensorOrCreateForWrappedNumber(
205 |     const at::Tensor& tensor,
206 |     const BackendDevice& device);
207 | 
208 | // Section 2: LazyTensor => at::Tensor.
209 | // Creates an ATen tensor from an LazyTensor.
210 | TORCH_API at::Tensor CreateAtenFromLtcTensor(const LazyTensorPtr& ltc_tensor);
211 | TORCH_API at::Tensor CreateAtenFromLtcTensor(LazyTensor&& ltc_tensor);
212 | 
213 | // Note [Lazy Tensor Functionalization]
214 | // The functionalization pass is implemented by wrapping all TensorImpl
215 | // objects in C++ with an extra FunctionalTensorWrapper object,
216 | // that knows how to perform functionalization
217 | //
218 | // Certain functions in the aten API serve as entry/exit points for
219 | // functionalization, where we need to perform the wrapping/unwrapping:
220 | // - aten::to.device
221 | // - aten::empty
222 | 
```
- EN: Declares routines such as `TryGetLtcTensor`, `GetLtcTensor`, `GetLtcTensors`, `GetOrCreateLtcTensor`, `GetLtcTensorOrCreateForWrappedNumber` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 声明了 `TryGetLtcTensor`、`GetLtcTensor`、`GetLtcTensors`、`GetOrCreateLtcTensor`、`GetLtcTensorOrCreateForWrappedNumber` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 223-259
```cpp
223 | // Given a non-lazy tensor, this function creates a lazy tensor on the specified
224 | // (lazy) device. The functionalize_output determines whether or not we should
225 | // wrap the output in a "functional wrapper".
226 | //
227 | // How do you know whether to pass true/false for functionalize_output?
228 | //
229 | // Case 1: nonlazy -> lazy
230 | //   If you're implementing a function that takes in nonlazy tensors and returns
231 | //   lazy tensors, then you should think of that function as an "entrypoint" to
232 | //   functionalization, and use functionalize_output=true Examples include:
233 | //   - factory functions (the LTC kernel for at::empty)
234 | //   - CPU -> Lazy device conversions (the LTC kernel for at::to_device)
235 | //
236 | // Case 2: lazy -> lazy
237 | //   If you're implementing a function that takes in lazy tensors and returns
238 | //   lazy tensors,
239 | //   **but** requires creating lazy tensors internally,
240 | //   then you can assume that the current function is running inside of some
241 | //   outer context where functionalization is already running, that will take
242 | //   care of doing the wrapping for you, and use functionalize_output=true
243 | //   Examples include:
244 | //   - CPU fallback (takes in lazy tensors, converts to cpu, calls kernel,
245 | //   converts returns back to lazy tensors).
246 | TORCH_API at::Tensor to_lazy_tensor(
247 |     const at::Tensor& self,
248 |     const c10::TensorOptions& options,
249 |     at::Device device,
250 |     bool non_blocking,
251 |     bool functionalize_output);
252 | 
253 | template <size_t... Indices>
254 | auto TupleAtenFromLtcTensorsImpl(
255 |     const std::vector<LazyTensorPtr>& tensors,
256 |     std::index_sequence<Indices...> /*unused*/) {
257 |   return std::make_tuple(CreateAtenFromLtcTensor(tensors[Indices])...);
258 | }
259 | 
```
- EN: Implements routines such as `to_lazy_tensor`, `TupleAtenFromLtcTensorsImpl` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `to_lazy_tensor`、`TupleAtenFromLtcTensorsImpl` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 260-265
```cpp
260 | template <size_t N>
261 | auto TupleAtenFromLtcTensors(const std::vector<LazyTensorPtr>& tensors) {
262 |   return TupleAtenFromLtcTensorsImpl(tensors, std::make_index_sequence<N>{});
263 | }
264 | 
265 | } // namespace torch::lazy
```
- EN: Implements routines such as `TupleAtenFromLtcTensors` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `TupleAtenFromLtcTensors` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

## Key Concepts / 关键概念

- **Primary types / 主要类型**
  - EN: `SymNodeImpl`, `LazyTensor`, `Data`.
  - CN: `SymNodeImpl`、`LazyTensor`、`Data`。
- **Important routines / 重要例程**
  - EN: `SymNodeImpl`, `Create`, `generation`, `size`, `ToTensor`, `ShallowCopyTo`, `SetTensor`, `UpdateFromTensor`.
  - CN: `SymNodeImpl`、`Create`、`generation`、`size`、`ToTensor`、`ShallowCopyTo`、`SetTensor`、`UpdateFromTensor`。
- **Subsystem scope / 子系统作用域**
  - EN: The code lives under `torch::lazy`, indicating the owning PyTorch subsystem.
  - CN: 代码位于 `torch::lazy` 命名空间下，这说明了其所属的 PyTorch 子系统。

## Dependencies / 依赖关系

- Local includes / 本地头文件: `<c10/core/SymNodeImpl.h>`, `<c10/util/intrusive_ptr.h>`, `<torch/csrc/lazy/backend/backend_data.h>`, `<torch/csrc/lazy/backend/backend_device.h>`, `<torch/csrc/lazy/core/ir.h>`, `<torch/csrc/lazy/core/util.h>`
- External includes / 外部头文件: None / 无
- Relationship / 关系:
  - EN: The unit relies on neighboring PyTorch headers for subsystem-specific types, dispatcher hooks, and utility helpers. Its exported or visible routines are then consumed by callers in the same namespace and by higher-level runtime paths elsewhere in PyTorch.
  - CN: 该编译单元依赖相邻的 PyTorch 头文件，以获得子系统类型、分发器钩子和辅助工具。 随后，这里导出或可见的例程会被同一命名空间中的调用方，以及 PyTorch 其他位置的更高层运行时路径所使用。
