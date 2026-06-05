# ir.h — Code Analysis / 代码分析

## Source / 来源
- File / 文件: `torch/csrc/lazy/core/ir.h`
- Repository / 仓库: `pytorch`
- Purpose / 作用:
  - EN: Implements the shared Lazy Tensor IR, hashing, shape, and execution support.
  - CN: 实现通用 Lazy Tensor IR、哈希、形状和执行支撑逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-37
```cpp
 1 | #pragma once
 2 | 
 3 | #include <ATen/core/symbol.h>
 4 | 
 5 | #include <functional>
 6 | #include <memory>
 7 | #include <set>
 8 | #include <string>
 9 | #include <unordered_map>
10 | #include <unordered_set>
11 | #include <utility>
12 | #include <vector>
13 | 
14 | #include <c10/core/ScalarType.h>
15 | #include <c10/util/ArrayRef.h>
16 | #include <c10/util/Flags.h>
17 | #include <torch/csrc/lazy/core/hash.h>
18 | #include <torch/csrc/lazy/core/ir_metadata.h>
19 | #include <torch/csrc/lazy/core/shape.h>
20 | 
21 | TORCH_DECLARE_bool(ltc_enable_dynamic_shapes);
22 | 
23 | namespace torch::lazy {
24 | 
25 | static const hash_t kHashSeed(static_cast<uint32_t>(0x5a2d296e9));
26 | 
27 | class Node;
28 | struct Output;
29 | struct Value;
30 | 
31 | using NodePtr = std::shared_ptr<Node>;
32 | 
33 | // The Kind of operation a Node can be associated to.
34 | struct TORCH_API OpKind {
35 |   OpKind() = default;
36 |   explicit OpKind(c10::Symbol op) : op(op) {}
37 | 
```
- EN: Brings in project headers such as `<ATen/core/symbol.h>`, `<c10/core/ScalarType.h>`, `<c10/util/ArrayRef.h>`, `<c10/util/Flags.h>` and system or third-party headers such as `<functional>`, `<memory>`, `<set>`, `<string>` so this section can use their types, APIs, or macros. Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Places the implementation in namespace scopes (`torch::lazy`) so ownership matches the PyTorch subsystem layout. Defines or extends data abstractions such as `Node`, `Output`, `Value`, `OpKind` that structure the state handled by this file.
- CN: 这里引入了项目头文件，例如 `<ATen/core/symbol.h>`、`<c10/core/ScalarType.h>`、`<c10/util/ArrayRef.h>`、`<c10/util/Flags.h>`以及系统或第三方头文件，例如 `<functional>`、`<memory>`、`<set>`、`<string>`，使本段代码能够使用相关类型、API 或宏。 使用预处理指令来控制编译开关、平台分支或导出声明。 把实现放入命名空间作用域（`torch::lazy`），使其归属与 PyTorch 子系统布局保持一致。 定义或扩展了 `Node`、`Output`、`Value`、`OpKind` 等数据抽象，用来组织本文件处理的状态。

### Lines 38-68
```cpp
38 |   bool operator==(const OpKind& rhs) const {
39 |     return op == rhs.op;
40 |   }
41 |   bool operator!=(const OpKind& rhs) const {
42 |     return !operator==(rhs);
43 |   }
44 |   bool operator<(const OpKind& rhs) const {
45 |     return c10::unique_t(op) < c10::unique_t(rhs.op);
46 |   }
47 | 
48 |   hash_t hash() const;
49 | 
50 |   std::string ToString() const {
51 |     return op.toQualString();
52 |   }
53 | 
54 |   // Retrieves an existing operation object, or creates a new one. Operations
55 |   // that are specific to lazy tensors, should live within the 'lazy_tensors::'
56 |   // namespace.
57 |   static OpKind Get(const std::string& name);
58 | 
59 |   c10::Symbol op;
60 | };
61 | 
62 | inline std::ostream& operator<<(std::ostream& stream, const OpKind& op) {
63 |   stream << op.ToString();
64 |   return stream;
65 | }
66 | 
67 | using OpList = c10::ArrayRef<Value>;
68 | 
```
- EN: Implements routines such as `hash`, `ToString`, `Get` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `hash`、`ToString`、`Get` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 69-107
```cpp
 69 | hash_t OperandHashes(
 70 |     const OpList& operands,
 71 |     const hash_t& seed,
 72 |     bool bakeInSizes);
 73 | // A node in the graph. Nodes for operations which require extra data to be
 74 | // stored for lowering should inherit from this class and add an operation
 75 | // specific member there. For example, a constant might create a new
 76 | // NodeConstant class (inheriting from Node) with an extra lazy_tensors::Literal
 77 | // field, or a tensor value might create a new NodeTensor with a computation
 78 | // client data handle in it.
 79 | class TORCH_API Node {
 80 |  public:
 81 |   static bool enableDynamicShape();
 82 | 
 83 |   // Creates a new node with the given op name. The op is a unique identifier
 84 |   // for the operation. The num_outputs tells how many outputs a given operation
 85 |   // generates.
 86 |   //
 87 |   // None leaf node's node_hash does not contains shape information always.
 88 |   // So we pass in the hash value rather than a function.
 89 |   Node(OpKind op, size_t num_outputs);
 90 | 
 91 |   // Construct node with operands and shapes
 92 |   Node(
 93 |       OpKind op,
 94 |       OpList operands,
 95 |       std::vector<Shape>&& shapes,
 96 |       size_t num_outputs = 1);
 97 | 
 98 |   // Construct node with operands and no shape
 99 |   Node(OpKind op, OpList operands, size_t num_outputs = 1);
100 | 
101 |   // Construct node with shape and no operands
102 |   Node(OpKind op, Shape shape, size_t num_outputs = 1);
103 | 
104 |   Node(const Node& rhs);
105 | 
106 |   Node(Node&& rhs);
107 | 
```
- EN: Defines or extends data abstractions such as `and`, `Node` that structure the state handled by this file. Implements routines such as `OperandHashes`, `enableDynamicShape` that expose the key API or control flow of this region.
- CN: 定义或扩展了 `and`、`Node` 等数据抽象，用来组织本文件处理的状态。 实现了 `OperandHashes`、`enableDynamicShape` 等例程，它们构成了这一段的关键 API 或控制流程。

### Lines 108-145
```cpp
108 |   virtual ~Node();
109 | 
110 |   Node& operator=(const Node& rhs);
111 | 
112 |   Node& operator=(Node&& rhs);
113 | 
114 |   const OpKind& op() const {
115 |     return op_;
116 |   }
117 | 
118 |   size_t num_outputs() const {
119 |     return num_outputs_;
120 |   }
121 | 
122 |   // Retrieves the full shape of the IR Node.
123 |   virtual c10::ArrayRef<Shape> shapes() const;
124 | 
125 |   virtual const Shape& shape(size_t output_index = 0) const;
126 | 
127 |   // Add the shape computed by the shape_fn
128 |   void addComputedShape(const std::function<Shape()>& shape_fn);
129 | 
130 |   // Compute the shape using the provided shape_fn if not previously cached
131 |   Shape computeShape(const std::function<Shape()>& shape_fn);
132 | 
133 |   virtual const std::vector<Output>& operands() const;
134 | 
135 |   virtual const Output& operand(size_t i) const;
136 | 
137 |   // Gets operand at index i if index is valid, or kNullOutput otherwise.
138 |   virtual const Output& nullable_operand(size_t i) const;
139 | 
140 |   // Returns the hash of the dag used to look up the compiled graph
141 |   virtual hash_t hash() const = 0;
142 | 
143 |   // Returns the hash of the dag used to for shape caching
144 |   virtual hash_t shapeHash() const = 0;
145 | 
```
- EN: Implements routines such as `op`, `num_outputs`, `shapes`, `shape`, `addComputedShape` that expose the key API or control flow of this region. At the statement level, this block declares abstract hooks that derived backends must provide; returns computed state or forwards values to callers; stores long-lived member state for later calls.
- CN: 实现了 `op`、`num_outputs`、`shapes`、`shape`、`addComputedShape` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段声明了需要由派生后端提供的抽象钩子；返回计算状态或把值转交给调用方；保存供后续调用使用的长期成员状态。

### Lines 146-184
```cpp
146 |   const MetaData& metadata() const {
147 |     return metadata_;
148 |   }
149 | 
150 |   UserMetaData* user_metadata() const {
151 |     return user_metadata_.get();
152 |   }
153 | 
154 |   std::shared_ptr<UserMetaData> SetUserMetadata(
155 |       std::shared_ptr<UserMetaData> user_meta) {
156 |     std::swap(user_metadata_, user_meta);
157 |     return user_meta;
158 |   }
159 | 
160 |   virtual std::string ToString() const;
161 | 
162 |  private:
163 |   // The ID of the operation captured by this node.
164 |   OpKind op_;
165 |   size_t num_outputs_ = 1;
166 | 
167 |   // The IR specific metadata attached to the IR node.
168 |   MetaData metadata_;
169 |   // The IR framework user can attach a user defined metadata object deriving
170 |   // from UserMetaData.
171 |   std::shared_ptr<UserMetaData> user_metadata_;
172 | 
173 |  protected:
174 |   // Adds node's index output number as operand.
175 |   void AddOperand(const NodePtr& node, size_t index = 0);
176 | 
177 |   std::vector<Shape> shapes_;
178 |   // A node holds a real reference to its operands.
179 |   std::vector<NodePtr> operands_;
180 |   // Outputs do not hold references on the nodes, and neither do the uses, since
181 |   // otherwise we get into circular reference counting.
182 |   std::vector<Output> operands_as_outputs_;
183 | };
184 | 
```
- EN: Implements routines such as `metadata`, `user_metadata`, `SetUserMetadata`, `ToString`, `AddOperand` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; stores long-lived member state for later calls.
- CN: 实现了 `metadata`、`user_metadata`、`SetUserMetadata`、`ToString`、`AddOperand` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；保存供后续调用使用的长期成员状态。

### Lines 185-222
```cpp
185 | inline std::ostream& operator<<(std::ostream& stream, const Node& node) {
186 |   stream << node.ToString();
187 |   return stream;
188 | }
189 | 
190 | // Note: Keep this version of NodeCast for smooth PyTorch/XLA migration, and
191 | // clean up once the migration is done.
192 | template <typename T>
193 | const T* NodeCast(const Node* node, OpKind op) {
194 |   if (op != node->op()) {
195 |     return nullptr;
196 |   }
197 | #ifdef NDEBUG
198 |   return static_cast<const T*>(node);
199 | #else
200 |   return &dynamic_cast<const T&>(*node);
201 | #endif
202 | }
203 | 
204 | template <typename T>
205 | const T* NodeCast(const Node* node) {
206 |   if (T::ClassOpKind() != node->op()) {
207 |     return nullptr;
208 |   }
209 |   // TODO: Some IR classes share the same opkind, such as Mean and MeanDim, so
210 |   // static_cast is not safe here. Unless we have opkind unique for each class,
211 |   // we have to use dynamic_cast here.
212 |   return dynamic_cast<const T*>(node);
213 | }
214 | 
215 | // Represents a specific output produced by a node. Since the output of a node
216 | // can be composed by multiple outputs, the node+index coordinates fully qualify
217 | // each single output.
218 | struct TORCH_API Output {
219 |   struct Hasher {
220 |     size_t operator()(const Output& output) const;
221 |   };
222 | 
```
- EN: Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Defines or extends data abstractions such as `Output`, `Hasher` that structure the state handled by this file. Implements routines such as `NodeCast`, `operator` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 使用预处理指令来控制编译开关、平台分支或导出声明。 定义或扩展了 `Output`、`Hasher` 等数据抽象，用来组织本文件处理的状态。 实现了 `NodeCast`、`operator` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 223-260
```cpp
223 |   Output() = default;
224 |   explicit Output(const Node* node, size_t index = 0)
225 |       : node(node), index(index) {}
226 | 
227 |   hash_t hash() const;
228 |   hash_t shapeHash() const;
229 | 
230 |   bool operator==(const Output& rhs) const {
231 |     return node == rhs.node && index == rhs.index;
232 |   }
233 | 
234 |   // To compare the operands of to-be-constructed node and to-be-reused node
235 |   bool operator==(const Value& rhs) const;
236 | 
237 |   bool operator!=(const Output& rhs) const {
238 |     return !operator==(rhs);
239 |   }
240 | 
241 |   const Shape& shape() const {
242 |     return node->shape(index);
243 |   }
244 | 
245 |   std::string ToString() const;
246 | 
247 |   // The node providing the output.
248 |   const Node* node{nullptr};
249 |   // The index in the node's output this output refers to.
250 |   size_t index{0};
251 | };
252 | 
253 | inline std::ostream& operator<<(std::ostream& stream, const Output& output) {
254 |   stream << output.ToString();
255 |   return stream;
256 | }
257 | 
258 | template <typename T>
259 | using OutputMap = std::unordered_map<Output, T, Output::Hasher>;
260 | 
```
- EN: Implements routines such as `Output`, `hash`, `shapeHash`, `shape`, `ToString` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `Output`、`hash`、`shapeHash`、`shape`、`ToString` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 261-293
```cpp
261 | // Represents an input/operand for a Node object.
262 | struct TORCH_API Value {
263 |   Value() = default;
264 |   /* implicit */ Value(NodePtr&& node, size_t index = 0)
265 |       : node(std::move(node)), index(index) {}
266 |   /* implicit */ Value(const NodePtr& node, size_t index = 0)
267 |       : node(node), index(index) {}
268 | 
269 |   hash_t hash() const;
270 |   hash_t shapeHash() const;
271 | 
272 |   operator bool() const {
273 |     return node != nullptr;
274 |   }
275 | 
276 |   operator Output() const {
277 |     return Output(node.get(), index);
278 |   }
279 | 
280 |   const Shape& shape() const {
281 |     return node->shape(index);
282 |   }
283 | 
284 |   Node* operator->() const {
285 |     return node.get();
286 |   }
287 | 
288 |   NodePtr node;
289 |   size_t index = 0;
290 | };
291 | 
292 | } // namespace torch::lazy
293 | 
```
- EN: Defines or extends data abstractions such as `Value` that structure the state handled by this file. Implements routines such as `hash`, `shapeHash`, `bool`, `Output`, `shape` that expose the key API or control flow of this region. At the statement level, this block declares abstract hooks that derived backends must provide; returns computed state or forwards values to callers; uses move semantics to avoid unnecessary copies.
- CN: 定义或扩展了 `Value` 等数据抽象，用来组织本文件处理的状态。 实现了 `hash`、`shapeHash`、`bool`、`Output`、`shape` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段声明了需要由派生后端提供的抽象钩子；返回计算状态或把值转交给调用方；使用移动语义来避免不必要的拷贝。

### Lines 294-297
```cpp
294 | namespace c10 {
295 | // Explicit template instantiation to make ArrayRef<Value> work
296 | template class at::ArrayRef<torch::lazy::Value>;
297 | } // namespace c10
```
- EN: Places the implementation in namespace scopes (`c10`) so ownership matches the PyTorch subsystem layout. Defines or extends data abstractions such as `at` that structure the state handled by this file.
- CN: 把实现放入命名空间作用域（`c10`），使其归属与 PyTorch 子系统布局保持一致。 定义或扩展了 `at` 等数据抽象，用来组织本文件处理的状态。

## Key Concepts / 关键概念

- **Primary types / 主要类型**
  - EN: `Node`, `and`, `at`, `Output`, `Value`, `OpKind`.
  - CN: `Node`、`and`、`at`、`Output`、`Value`、`OpKind`。
- **Important routines / 重要例程**
  - EN: `kHashSeed`, `OpKind`, `hash`, `ToString`, `Get`, `OperandHashes`, `enableDynamicShape`, `op`.
  - CN: `kHashSeed`、`OpKind`、`hash`、`ToString`、`Get`、`OperandHashes`、`enableDynamicShape`、`op`。
- **Subsystem scope / 子系统作用域**
  - EN: The code lives under `torch::lazy`, `c10`, indicating the owning PyTorch subsystem.
  - CN: 代码位于 `torch::lazy`、`c10` 命名空间下，这说明了其所属的 PyTorch 子系统。

## Dependencies / 依赖关系

- Local includes / 本地头文件: `<ATen/core/symbol.h>`, `<c10/core/ScalarType.h>`, `<c10/util/ArrayRef.h>`, `<c10/util/Flags.h>`, `<torch/csrc/lazy/core/hash.h>`, `<torch/csrc/lazy/core/ir_metadata.h>`, `<torch/csrc/lazy/core/shape.h>`
- External includes / 外部头文件: `<functional>`, `<memory>`, `<set>`, `<string>`, `<unordered_map>`, `<unordered_set>`, `<utility>`, `<vector>`
- Relationship / 关系:
  - EN: The unit relies on neighboring PyTorch headers for subsystem-specific types, dispatcher hooks, and utility helpers. Standard-library or third-party headers provide generic facilities such as containers, threading, math, or backend integration points. Its exported or visible routines are then consumed by callers in the same namespace and by higher-level runtime paths elsewhere in PyTorch.
  - CN: 该编译单元依赖相邻的 PyTorch 头文件，以获得子系统类型、分发器钩子和辅助工具。 标准库或第三方头文件提供了通用能力，例如容器、线程、数学支持或后端集成入口。 随后，这里导出或可见的例程会被同一命名空间中的调用方，以及 PyTorch 其他位置的更高层运行时路径所使用。
