# ir.cpp — Code Analysis / 代码分析

## Source / 来源
- File / 文件: `torch/csrc/lazy/core/ir.cpp`
- Repository / 仓库: `pytorch`
- Purpose / 作用:
  - EN: Implements the shared Lazy Tensor IR, hashing, shape, and execution support.
  - CN: 实现通用 Lazy Tensor IR、哈希、形状和执行支撑逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-35
```cpp
 1 | #include <c10/util/env.h>
 2 | #include <torch/csrc/lazy/core/cache.h>
 3 | #include <torch/csrc/lazy/core/config.h>
 4 | #include <torch/csrc/lazy/core/ir.h>
 5 | #include <torch/csrc/lazy/core/ir_metadata.h>
 6 | 
 7 | // Enables caching on for dynamic shapes (aka disable hash on shapes)
 8 | // clang-format off
 9 | C10_DEFINE_bool(
10 |     ltc_enable_dynamic_shapes,
11 |     false,
12 |     "Whether dynamic shape is enabled")
13 | 
14 | namespace torch::lazy {
15 | static const torch::lazy::Output kNullOutput = torch::lazy::Output();
16 | 
17 | size_t Output::Hasher::operator()(const Output& output) const {
18 |   return StdHashCombine(
19 |       reinterpret_cast<std::ptrdiff_t>(output.node), output.index);
20 | }
21 | 
22 | hash_t Output::hash() const {
23 |   return HashCombine(node->hash(), Hash(index));
24 | }
25 | 
26 | hash_t Output::shapeHash() const {
27 |   return HashCombine(node->shapeHash(), Hash(index));
28 | }
29 | 
30 | std::string Output::ToString() const {
31 |   std::stringstream ss;
32 |   ss << node->ToString() << ", index=" << index;
33 |   return ss.str();
34 | }
35 | 
```
- EN: Brings in project headers such as `<c10/util/env.h>`, `<torch/csrc/lazy/core/cache.h>`, `<torch/csrc/lazy/core/config.h>`, `<torch/csrc/lazy/core/ir.h>` so this section can use their types, APIs, or macros. Places the implementation in namespace scopes (`torch::lazy`) so ownership matches the PyTorch subsystem layout. Implements routines such as `StdHashCombine`, `HashCombine` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 这里引入了项目头文件，例如 `<c10/util/env.h>`、`<torch/csrc/lazy/core/cache.h>`、`<torch/csrc/lazy/core/config.h>`、`<torch/csrc/lazy/core/ir.h>`，使本段代码能够使用相关类型、API 或宏。 把实现放入命名空间作用域（`torch::lazy`），使其归属与 PyTorch 子系统布局保持一致。 实现了 `StdHashCombine`、`HashCombine` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 36-65
```cpp
36 | bool Output::operator==(const Value& rhs) const {
37 |   // Either side could be kNullValue which has node as nullptr
38 |   return (!node == !rhs.node) &&
39 |       (!node || (node->hash() == rhs.node->hash() && index == rhs.index));
40 | }
41 | 
42 | hash_t Value::hash() const {
43 |   return HashCombine(node->hash(), Hash(index));
44 | }
45 | 
46 | hash_t Value::shapeHash() const {
47 |   return HashCombine(node->shapeHash(), Hash(index));
48 | }
49 | 
50 | OpKind OpKind::Get(const std::string& name) {
51 |   return OpKind(c10::Symbol::fromQualString(name));
52 | }
53 | 
54 | hash_t OpKind::hash() const {
55 |   return StringHash(op.toQualString());
56 | }
57 | 
58 | bool Node::enableDynamicShape() {
59 |   static bool enabled = c10::utils::has_env("LTC_ENABLE_DYNAMIC_SHAPES");
60 |   return enabled || FLAGS_ltc_enable_dynamic_shapes;
61 | }
62 | 
63 | Node::Node(OpKind op, size_t num_outputs)
64 |     : op_(op), num_outputs_(num_outputs), metadata_(GetMetaDataIfDebugging()) {}
65 | 
```
- EN: Implements routines such as `HashCombine`, `OpKind`, `StringHash` that expose the key API or control flow of this region. Reads environment switches (`LTC_ENABLE_DYNAMIC_SHAPES`) to tune runtime behavior. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `HashCombine`、`OpKind`、`StringHash` 等例程，它们构成了这一段的关键 API 或控制流程。 读取环境变量开关（`LTC_ENABLE_DYNAMIC_SHAPES`）来调整运行时行为。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 66-104
```cpp
 66 | Node::Node(
 67 |     OpKind op,
 68 |     OpList operands,
 69 |     // NOLINTNEXTLINE(cppcoreguidelines-rvalue-reference-param-not-moved)
 70 |     std::vector<Shape>&& shapes,
 71 |     size_t num_outputs)
 72 |     : Node(op, num_outputs) {
 73 |   // Move shapes into node
 74 |   shapes_.insert(
 75 |       shapes_.end(),
 76 |       std::make_move_iterator(shapes.begin()),
 77 |       std::make_move_iterator(shapes.end()));
 78 | 
 79 |   for (auto& operand : operands) {
 80 |     // Ideally, optional operands should be filtered by the leaf node classes,
 81 |     // but it's just much easier to do it here.
 82 |     // TODO(alanwaketan): Find a way to move the below logic to the leaf node
 83 |     // classes.
 84 |     if (!operand) {
 85 |       continue;
 86 |     }
 87 | 
 88 |     AddOperand(operand.node, operand.index);
 89 |   }
 90 | }
 91 | 
 92 | Node::Node(OpKind op, OpList operands, size_t num_outputs)
 93 |     : Node(op, operands, std::vector<Shape>{}, num_outputs) {}
 94 | 
 95 | Node::Node(OpKind op, Shape shape, size_t num_outputs) : Node(op, num_outputs) {
 96 |   shapes_.push_back(std::move(shape));
 97 | }
 98 | 
 99 | Node::Node(const Node& rhs) = default;
100 | 
101 | Node::Node(Node&& rhs) = default;
102 | 
103 | Node::~Node() = default;
104 | 
```
- EN: At the statement level, this block uses move semantics to avoid unnecessary copies; builds container state that later execution depends on.
- CN: 在语句层面，这一段使用移动语义来避免不必要的拷贝；构建后续执行依赖的容器状态。

### Lines 105-142
```cpp
105 | Node& Node::operator=(const Node& rhs) = default;
106 | 
107 | Node& Node::operator=(Node&& rhs) = default;
108 | 
109 | // Retrieves the full shape of the IR Node.
110 | c10::ArrayRef<Shape> Node::shapes() const {
111 |   return shapes_;
112 | }
113 | 
114 | // Retrieves the shape of the output at a given index.
115 | const Shape& Node::shape(size_t output_index) const {
116 |   return shapes_.at(output_index);
117 | }
118 | 
119 | // Add the shape computed by the shape_fn
120 | 
121 | void Node::addComputedShape(const std::function<Shape()>& shape_fn) {
122 |   shapes_.push_back(computeShape(shape_fn));
123 | }
124 | 
125 | using ShapeCache = Cache<hash_t, Shape, HashReducer>;
126 | 
127 | // Compute the shape using the provided shape_fn.
128 | Shape Node::computeShape(const std::function<Shape()>& shape_fn) {
129 |   static ShapeCache* cache = new ShapeCache(FLAGS_torch_lazy_shape_cache_size);
130 | 
131 |   auto hash = shapeHash();
132 |   auto shape = cache->Get(hash);
133 |   if (shape == nullptr) {
134 |     shape = cache->Add(hash, std::make_shared<Shape>(shape_fn()));
135 |   }
136 |   return *shape;
137 | }
138 | 
139 | const std::vector<Output>& Node::operands() const {
140 |   return operands_as_outputs_;
141 | }
142 | 
```
- EN: At the statement level, this block returns computed state or forwards values to callers; stores long-lived member state for later calls.
- CN: 在语句层面，这一段返回计算状态或把值转交给调用方；保存供后续调用使用的长期成员状态。

### Lines 143-172
```cpp
143 | const Output& Node::operand(size_t i) const {
144 |   return operands_as_outputs_.at(i);
145 | }
146 | 
147 | const Output& Node::nullable_operand(size_t i) const {
148 |   // We use kNullOutput instead of kNullValue here to avoid implicit casting,
149 |   // which would prevent this method from returning a reference.
150 |   return i < operands_as_outputs_.size() ? operand(i) : kNullOutput;
151 | }
152 | 
153 | std::string Node::ToString() const {
154 |   std::stringstream ss;
155 |   ss << shapes() << ' ' << op();
156 |   if (num_outputs() > 1) {
157 |     ss << ", num_outputs=" << num_outputs();
158 |   }
159 |   if (!metadata().scope.empty()) {
160 |     ss << ", scope=" << metadata().scope;
161 |   }
162 |   EmitShortFrameInfo(ss, metadata().frame_info);
163 |   return ss.str();
164 | }
165 | 
166 | void Node::AddOperand(const NodePtr& node, size_t index) {
167 |   TORCH_CHECK_LT(index, node->num_outputs());
168 |   operands_.push_back(node);
169 |   operands_as_outputs_.emplace_back(operands_.back().get(), index);
170 | }
171 | 
172 | } // namespace torch::lazy
```
- EN: Implements routines such as `shapes` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 实现了 `shapes` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

## Key Concepts / 关键概念

- **Important routines / 重要例程**
  - EN: `StdHashCombine`, `HashCombine`, `OpKind`, `StringHash`, `shapes`.
  - CN: `StdHashCombine`、`HashCombine`、`OpKind`、`StringHash`、`shapes`。
- **Subsystem scope / 子系统作用域**
  - EN: The code lives under `torch::lazy`, indicating the owning PyTorch subsystem.
  - CN: 代码位于 `torch::lazy` 命名空间下，这说明了其所属的 PyTorch 子系统。

## Dependencies / 依赖关系

- Local includes / 本地头文件: `<c10/util/env.h>`, `<torch/csrc/lazy/core/cache.h>`, `<torch/csrc/lazy/core/config.h>`, `<torch/csrc/lazy/core/ir.h>`, `<torch/csrc/lazy/core/ir_metadata.h>`
- External includes / 外部头文件: None / 无
- Relationship / 关系:
  - EN: The unit relies on neighboring PyTorch headers for subsystem-specific types, dispatcher hooks, and utility helpers. Its exported or visible routines are then consumed by callers in the same namespace and by higher-level runtime paths elsewhere in PyTorch.
  - CN: 该编译单元依赖相邻的 PyTorch 头文件，以获得子系统类型、分发器钩子和辅助工具。 随后，这里导出或可见的例程会被同一命名空间中的调用方，以及 PyTorch 其他位置的更高层运行时路径所使用。
