# ir_builder.h — Code Analysis / 代码分析

## Source / 来源
- File / 文件: `torch/csrc/lazy/core/ir_builder.h`
- Repository / 仓库: `pytorch`
- Purpose / 作用:
  - EN: Implements the shared Lazy Tensor IR, hashing, shape, and execution support.
  - CN: 实现通用 Lazy Tensor IR、哈希、形状和执行支撑逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-32
```cpp
 1 | #pragma once
 2 | 
 3 | #include <c10/core/ScalarType.h>
 4 | #include <torch/csrc/lazy/backend/backend_interface.h>
 5 | #include <torch/csrc/lazy/core/config.h>
 6 | #include <torch/csrc/lazy/core/ir.h>
 7 | #include <torch/csrc/lazy/core/tensor.h>
 8 | #include <torch/csrc/lazy/core/trie.h>
 9 | #include <optional>
10 | #include <vector>
11 | 
12 | // This file is part of the backend interface. So, ops shouldn't be added or
13 | // removed without due process The exception to this being the view ops which
14 | // will be removed soon pending functionalization
15 | 
16 | namespace torch::lazy {
17 | 
18 | template <typename T, typename... Args>
19 | NodePtr ReuseNode(Args&&... args) {
20 |   if (FLAGS_torch_lazy_reuse_ir) {
21 |     return LookupNodeFromTrieCache<T>(std::forward<Args>(args)...);
22 |   }
23 |   return nullptr;
24 | }
25 | 
26 | // Caching an IR node into TrieCache
27 | static inline void CacheNode(NodePtr node) {
28 |   if (FLAGS_torch_lazy_reuse_ir) {
29 |     TrieCache::Get()->Insert(std::move(node));
30 |   }
31 | }
32 | 
```
- EN: Brings in project headers such as `<c10/core/ScalarType.h>`, `<torch/csrc/lazy/backend/backend_interface.h>`, `<torch/csrc/lazy/core/config.h>`, `<torch/csrc/lazy/core/ir.h>` and system or third-party headers such as `<optional>`, `<vector>` so this section can use their types, APIs, or macros. Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Places the implementation in namespace scopes (`torch::lazy`) so ownership matches the PyTorch subsystem layout. Implements routines such as `ReuseNode`, `CacheNode` that expose the key API or control flow of this region.
- CN: 这里引入了项目头文件，例如 `<c10/core/ScalarType.h>`、`<torch/csrc/lazy/backend/backend_interface.h>`、`<torch/csrc/lazy/core/config.h>`、`<torch/csrc/lazy/core/ir.h>`以及系统或第三方头文件，例如 `<optional>`、`<vector>`，使本段代码能够使用相关类型、API 或宏。 使用预处理指令来控制编译开关、平台分支或导出声明。 把实现放入命名空间作用域（`torch::lazy`），使其归属与 PyTorch 子系统布局保持一致。 实现了 `ReuseNode`、`CacheNode` 等例程，它们构成了这一段的关键 API 或控制流程。

### Lines 33-49
```cpp
33 | template <typename T, typename... Args>
34 | NodePtr MakeNode(Args&&... args) {
35 |   return std::make_shared<T>(std::forward<Args>(args)...);
36 | }
37 | 
38 | // op is passed in for a more efficient node casting, see the implementation of
39 | // NodeCast
40 | template <typename T, typename... Args>
41 | NodePtr ReuseOrMakeNode(Args&&... args) {
42 |   NodePtr node = ReuseNode<T>(std::forward<Args>(args)...);
43 |   if (!node) {
44 |     node = MakeNode<T>(std::forward<Args>(args)...);
45 |     CacheNode(node);
46 |   }
47 |   return node;
48 | }
49 | 
```
- EN: Implements routines such as `MakeNode`, `ReuseOrMakeNode` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `MakeNode`、`ReuseOrMakeNode` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 50-80
```cpp
50 | struct IrBuilder {
51 |   virtual NodePtr MakeDeviceData(
52 |       const std::shared_ptr<BackendData>& data) const = 0;
53 |   virtual NodePtr MakeScalar(
54 |       const at::Scalar& value,
55 |       const at::ScalarType& type) const = 0;
56 |   virtual NodePtr MakeExpand(
57 |       const Value& input0,
58 |       const std::vector<int64_t>& size,
59 |       const bool& is_scalar_expand) const = 0;
60 |   virtual NodePtr MakeCast(
61 |       const Value& input0,
62 |       const at::ScalarType& dtype,
63 |       const std::optional<at::ScalarType>& stype = std::nullopt) const = 0;
64 |   virtual NodePtr MakeTensorList(const OpList& inputs) const = 0;
65 |   virtual NodePtr MakeGeneric(
66 |       const OpKind& op,
67 |       const OpList& operands,
68 |       const Shape& shape,
69 |       const size_t& num_outputs = 1,
70 |       const hash_t& hash_seed = static_cast<uint32_t>(0x5a2d296e9)) const = 0;
71 | 
72 |   // dynamic ir nodes
73 |   virtual NodePtr MakeSizeNode(const Value& input, size_t dim) const = 0;
74 |   virtual NodePtr MakeSizeAdd(const Value& a, const Value& b) const = 0;
75 |   virtual NodePtr MakeSizeMul(const Value& a, const Value& b) const = 0;
76 |   virtual NodePtr MakeSizeDiv(const Value& a, const Value& b) const = 0;
77 | 
78 |   virtual ~IrBuilder() = default;
79 | };
80 | 
```
- EN: Defines or extends data abstractions such as `IrBuilder` that structure the state handled by this file. Implements routines such as `MakeDeviceData`, `MakeScalar`, `MakeExpand`, `MakeCast`, `MakeTensorList` that expose the key API or control flow of this region. At the statement level, this block declares abstract hooks that derived backends must provide.
- CN: 定义或扩展了 `IrBuilder` 等数据抽象，用来组织本文件处理的状态。 实现了 `MakeDeviceData`、`MakeScalar`、`MakeExpand`、`MakeCast`、`MakeTensorList` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段声明了需要由派生后端提供的抽象钩子。

### Lines 81-104
```cpp
 81 | static inline NodePtr MakeDeviceData(const std::shared_ptr<BackendData>& data) {
 82 |   return getIrBuilder()->MakeDeviceData(data);
 83 | }
 84 | static inline NodePtr MakeScalar(
 85 |     const at::Scalar& value,
 86 |     const at::ScalarType& type) {
 87 |   return getIrBuilder()->MakeScalar(value, type);
 88 | }
 89 | static inline NodePtr MakeExpand(
 90 |     const Value& input0,
 91 |     const std::vector<int64_t>& size,
 92 |     const bool& is_scalar_expand) {
 93 |   return getIrBuilder()->MakeExpand(input0, size, is_scalar_expand);
 94 | }
 95 | static inline NodePtr MakeCast(
 96 |     const Value& input0,
 97 |     const at::ScalarType& dtype,
 98 |     const std::optional<at::ScalarType>& stype = std::nullopt) {
 99 |   return getIrBuilder()->MakeCast(input0, dtype, stype);
100 | }
101 | static inline NodePtr MakeTensorList(const OpList& inputs) {
102 |   return getIrBuilder()->MakeTensorList(inputs);
103 | }
104 | static inline NodePtr MakeGeneric(
```
- EN: Implements routines such as `MakeDeviceData`, `getIrBuilder`, `MakeScalar`, `MakeExpand`, `MakeCast` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `MakeDeviceData`、`getIrBuilder`、`MakeScalar`、`MakeExpand`、`MakeCast` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 105-127
```cpp
105 |     const OpKind& op,
106 |     const OpList& operands,
107 |     const Shape& shape,
108 |     const size_t& num_outputs = 1,
109 |     const hash_t& hash_seed = static_cast<uint32_t>(0x5a2d296e9)) {
110 |   return getIrBuilder()->MakeGeneric(
111 |       op, operands, shape, num_outputs, hash_seed);
112 | }
113 | 
114 | // dynamic ir nodes
115 | static inline NodePtr MakeSizeNode(const Value& input, size_t dim) {
116 |   return getIrBuilder()->MakeSizeNode(input, dim);
117 | }
118 | static inline NodePtr MakeSizeAdd(const Value& a, const Value& b) {
119 |   return getIrBuilder()->MakeSizeAdd(a, b);
120 | }
121 | static inline NodePtr MakeSizeMul(const Value& a, const Value& b) {
122 |   return getIrBuilder()->MakeSizeAdd(a, b);
123 | }
124 | static inline NodePtr MakeSizeDiv(const Value& a, const Value& b) {
125 |   return getIrBuilder()->MakeSizeDiv(a, b);
126 | }
127 | 
```
- EN: Implements routines such as `getIrBuilder`, `MakeSizeNode`, `MakeSizeAdd`, `MakeSizeMul`, `MakeSizeDiv` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `getIrBuilder`、`MakeSizeNode`、`MakeSizeAdd`、`MakeSizeMul`、`MakeSizeDiv` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 128-148
```cpp
128 | inline Value GetSymIntValue(const c10::SymInt& a) {
129 |   if (auto ma = a.maybe_as_int()) {
130 |     return Value(MakeScalar(*ma, at::kLong), 0);
131 |   } else {
132 |     return Value(
133 |         dynamic_cast<torch::lazy::SymNodeImpl*>(a.toSymNodeImplUnowned())
134 |             ->node_,
135 |         0);
136 |   }
137 | }
138 | 
139 | // TODO: this should return Value
140 | inline std::vector<int64_t> GetSymIntArrayRefValue(c10::SymIntArrayRef arr) {
141 |   std::vector<int64_t> r;
142 |   for (const auto& a : arr) {
143 |     r.emplace_back(a.guard_int(__FILE__, __LINE__));
144 |   }
145 |   return r;
146 | }
147 | 
148 | } // namespace torch::lazy
```
- EN: Implements routines such as `GetSymIntValue`, `Value`, `GetSymIntArrayRefValue` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `GetSymIntValue`、`Value`、`GetSymIntArrayRefValue` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

## Key Concepts / 关键概念

- **Primary types / 主要类型**
  - EN: `IrBuilder`.
  - CN: `IrBuilder`。
- **Important routines / 重要例程**
  - EN: `ReuseNode`, `CacheNode`, `MakeNode`, `ReuseOrMakeNode`, `MakeDeviceData`, `MakeScalar`, `MakeExpand`, `MakeCast`.
  - CN: `ReuseNode`、`CacheNode`、`MakeNode`、`ReuseOrMakeNode`、`MakeDeviceData`、`MakeScalar`、`MakeExpand`、`MakeCast`。
- **Subsystem scope / 子系统作用域**
  - EN: The code lives under `torch::lazy`, indicating the owning PyTorch subsystem.
  - CN: 代码位于 `torch::lazy` 命名空间下，这说明了其所属的 PyTorch 子系统。

## Dependencies / 依赖关系

- Local includes / 本地头文件: `<c10/core/ScalarType.h>`, `<torch/csrc/lazy/backend/backend_interface.h>`, `<torch/csrc/lazy/core/config.h>`, `<torch/csrc/lazy/core/ir.h>`, `<torch/csrc/lazy/core/tensor.h>`, `<torch/csrc/lazy/core/trie.h>`
- External includes / 外部头文件: `<optional>`, `<vector>`
- Relationship / 关系:
  - EN: The unit relies on neighboring PyTorch headers for subsystem-specific types, dispatcher hooks, and utility helpers. Standard-library or third-party headers provide generic facilities such as containers, threading, math, or backend integration points. Its exported or visible routines are then consumed by callers in the same namespace and by higher-level runtime paths elsewhere in PyTorch.
  - CN: 该编译单元依赖相邻的 PyTorch 头文件，以获得子系统类型、分发器钩子和辅助工具。 标准库或第三方头文件提供了通用能力，例如容器、线程、数学支持或后端集成入口。 随后，这里导出或可见的例程会被同一命名空间中的调用方，以及 PyTorch 其他位置的更高层运行时路径所使用。
