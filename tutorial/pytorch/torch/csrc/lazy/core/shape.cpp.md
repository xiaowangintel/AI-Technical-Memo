# shape.cpp — Code Analysis / 代码分析

## Source / 来源
- File / 文件: `torch/csrc/lazy/core/shape.cpp`
- Repository / 仓库: `pytorch`
- Purpose / 作用:
  - EN: Defines or computes tensor shape metadata used by the lazy subsystem.
  - CN: 定义或计算 Lazy 子系统使用的张量形状元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-29
```cpp
 1 | #include <c10/util/env.h>
 2 | #include <torch/csrc/lazy/core/shape.h>
 3 | #include <torch/csrc/lazy/core/tensor.h>
 4 | 
 5 | #include <utility>
 6 | 
 7 | C10_DEFINE_bool(
 8 |     ltc_enable_symbolic_shapes,
 9 |     false,
10 |     "Enables calculation of if dims are symbolic")
11 | 
12 | namespace torch::lazy {
13 | 
14 | Shape::Shape(
15 |     at::ScalarType scalar_type,
16 |     c10::ArrayRef<int64_t> sizes,
17 |     std::optional<std::vector<bool>> is_symbolic)
18 |     : scalar_type_(scalar_type),
19 |       sizes_(sizes.begin(), sizes.end()),
20 |       is_symbolic_(std::move(is_symbolic)) {}
21 | 
22 | std::string Shape::to_string() const {
23 |   return c10::str(toString(scalar_type_), "[", c10::Join(",", sizes_), "]");
24 | }
25 | 
26 | bool Shape::operator==(const Shape& other) const {
27 |   return scalar_type_ == other.scalar_type_ && sizes_ == other.sizes_;
28 | }
29 | 
```
- EN: Brings in project headers such as `<c10/util/env.h>`, `<torch/csrc/lazy/core/shape.h>`, `<torch/csrc/lazy/core/tensor.h>` and system or third-party headers such as `<utility>` so this section can use their types, APIs, or macros. Places the implementation in namespace scopes (`torch::lazy`) so ownership matches the PyTorch subsystem layout. At the statement level, this block returns computed state or forwards values to callers; uses move semantics to avoid unnecessary copies; stores long-lived member state for later calls.
- CN: 这里引入了项目头文件，例如 `<c10/util/env.h>`、`<torch/csrc/lazy/core/shape.h>`、`<torch/csrc/lazy/core/tensor.h>`以及系统或第三方头文件，例如 `<utility>`，使本段代码能够使用相关类型、API 或宏。 把实现放入命名空间作用域（`torch::lazy`），使其归属与 PyTorch 子系统布局保持一致。 在语句层面，这一段返回计算状态或把值转交给调用方；使用移动语义来避免不必要的拷贝；保存供后续调用使用的长期成员状态。

### Lines 30-58
```cpp
30 | std::ostream& operator<<(std::ostream& out, const Shape& shape) {
31 |   return out << shape.to_string();
32 | }
33 | 
34 | size_t Shape::numel() const {
35 |   size_t elts = 1;
36 |   for (auto size : sizes_) {
37 |     elts *= size;
38 |   }
39 |   return elts;
40 | }
41 | 
42 | hash_t Shape::hash(bool bakeInSizes) const {
43 |   if (bakeInSizes) {
44 |     return HashCombine(
45 |         Hash(scalar_type_),
46 |         DataHash(sizes_.data(), sizes_.size() * sizeof(int64_t)));
47 |   } else {
48 |     return HashCombine(Hash(scalar_type_), Hash(sizes_.size()));
49 |   }
50 | }
51 | 
52 | Shape Shape::with_symbolic_dims(
53 |     std::optional<std::vector<bool>> symbolic_dims) const {
54 |   Shape copy = *this;
55 |   copy.is_symbolic_ = std::move(symbolic_dims);
56 |   return copy;
57 | }
58 | 
```
- EN: Implements routines such as `HashCombine` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; uses move semantics to avoid unnecessary copies.
- CN: 实现了 `HashCombine` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；使用移动语义来避免不必要的拷贝。

### Lines 59-89
```cpp
59 | bool symbolicShapeEnabled() {
60 |   static bool enabled = c10::utils::has_env("LTC_ENABLE_SYMBOLIC_SHAPES");
61 |   return enabled || FLAGS_ltc_enable_symbolic_shapes;
62 | }
63 | 
64 | static c10::SymbolicShape get_symbolic_shape(at::Tensor& tensor) {
65 |   auto ltc_tensor = TryGetLtcTensor(tensor);
66 |   if (!ltc_tensor) {
67 |     // Set Concrete sizes for Concrete tensors
68 |     return c10::SymbolicShape(tensor.sizes());
69 |   }
70 |   const Shape& input_shape = ltc_tensor->GetIrValue()->shape();
71 |   auto& is_symbolic = input_shape.is_symbolic();
72 |   if (!is_symbolic.has_value()) {
73 |     return c10::SymbolicShape();
74 |   }
75 |   auto sizes = input_shape.sizes();
76 |   TORCH_INTERNAL_ASSERT(
77 |       sizes.size() == is_symbolic->size(),
78 |       "Dims of two values are not consistent");
79 |   std::vector<std::optional<int64_t>> symbolic_dims;
80 |   for (size_t i = 0; i < sizes.size(); i++) {
81 |     if (is_symbolic->at(i)) {
82 |       symbolic_dims.emplace_back(std::nullopt);
83 |     } else {
84 |       symbolic_dims.emplace_back(sizes.at(i));
85 |     }
86 |   }
87 |   return c10::SymbolicShape(symbolic_dims);
88 | }
89 | 
```
- EN: Implements routines such as `symbolicShapeEnabled`, `get_symbolic_shape` that expose the key API or control flow of this region. Reads environment switches (`LTC_ENABLE_SYMBOLIC_SHAPES`) to tune runtime behavior. At the statement level, this block declares abstract hooks that derived backends must provide; returns computed state or forwards values to callers.
- CN: 实现了 `symbolicShapeEnabled`、`get_symbolic_shape` 等例程，它们构成了这一段的关键 API 或控制流程。 读取环境变量开关（`LTC_ENABLE_SYMBOLIC_SHAPES`）来调整运行时行为。 在语句层面，这一段声明了需要由派生后端提供的抽象钩子；返回计算状态或把值转交给调用方。

### Lines 90-113
```cpp
 90 | void applySymbolicShapesOnLT(
 91 |     const char* schema_str,
 92 |     std::vector<c10::IValue> args,
 93 |     std::vector<Shape>& result_shapes) {
 94 |   std::vector<jit::SSAInput> converted_args;
 95 |   // TODO: Determine if there are any unknown values in LazyTensor
 96 |   const c10::FunctionSchema& schema =
 97 |       jit::getOperatorForLiteral(schema_str)->schema();
 98 | 
 99 |   for (auto& arg : args) {
100 |     // Handle list of tensors
101 |     if (arg.isTensorList()) {
102 |       at::List<at::Tensor> tensor_list = arg.toTensorList();
103 |       for (at::Tensor tensor : tensor_list) {
104 |         converted_args.emplace_back(get_symbolic_shape(tensor));
105 |       }
106 |     } else if (arg.isTensor()) {
107 |       auto ss = get_symbolic_shape(arg.toTensor());
108 |       converted_args.emplace_back(ss);
109 |     } else {
110 |       // If we need to support symbolic ints, here is the place
111 |       // to add it.
112 |       converted_args.emplace_back(arg);
113 |     }
```
- EN: Implements routines such as `applySymbolicShapesOnLT` that expose the key API or control flow of this region.
- CN: 实现了 `applySymbolicShapesOnLT` 等例程，它们构成了这一段的关键 API 或控制流程。

### Lines 114-134
```cpp
114 |   }
115 |   auto res_symbolic = jit::calculateSymbolicShapesOnOp(&schema, converted_args);
116 |   if (!res_symbolic) {
117 |     for (auto& result_shape : result_shapes) {
118 |       result_shape = result_shape.with_symbolic_dims(std::nullopt);
119 |     }
120 |   } else {
121 |     TORCH_INTERNAL_ASSERT(
122 |         res_symbolic->size() == result_shapes.size(),
123 |         "Result shape size is not consistent");
124 |     for (size_t i = 0; i < res_symbolic->size(); i++) {
125 |       auto sym_dims = res_symbolic->at(i).symbolicDims();
126 |       if (sym_dims.has_value()) {
127 |         result_shapes[i] =
128 |             result_shapes[i].with_symbolic_dims(std::move(sym_dims));
129 |       }
130 |     }
131 |   }
132 | }
133 | 
134 | } // namespace torch::lazy
```
- EN: At the statement level, this block declares abstract hooks that derived backends must provide; uses move semantics to avoid unnecessary copies.
- CN: 在语句层面，这一段声明了需要由派生后端提供的抽象钩子；使用移动语义来避免不必要的拷贝。

## Key Concepts / 关键概念

- **Important routines / 重要例程**
  - EN: `HashCombine`, `symbolicShapeEnabled`, `get_symbolic_shape`, `applySymbolicShapesOnLT`.
  - CN: `HashCombine`、`symbolicShapeEnabled`、`get_symbolic_shape`、`applySymbolicShapesOnLT`。
- **Subsystem scope / 子系统作用域**
  - EN: The code lives under `torch::lazy`, indicating the owning PyTorch subsystem.
  - CN: 代码位于 `torch::lazy` 命名空间下，这说明了其所属的 PyTorch 子系统。

## Dependencies / 依赖关系

- Local includes / 本地头文件: `<c10/util/env.h>`, `<torch/csrc/lazy/core/shape.h>`, `<torch/csrc/lazy/core/tensor.h>`
- External includes / 外部头文件: `<utility>`
- Relationship / 关系:
  - EN: The unit relies on neighboring PyTorch headers for subsystem-specific types, dispatcher hooks, and utility helpers. Standard-library or third-party headers provide generic facilities such as containers, threading, math, or backend integration points. Its exported or visible routines are then consumed by callers in the same namespace and by higher-level runtime paths elsewhere in PyTorch.
  - CN: 该编译单元依赖相邻的 PyTorch 头文件，以获得子系统类型、分发器钩子和辅助工具。 标准库或第三方头文件提供了通用能力，例如容器、线程、数学支持或后端集成入口。 随后，这里导出或可见的例程会被同一命名空间中的调用方，以及 PyTorch 其他位置的更高层运行时路径所使用。
