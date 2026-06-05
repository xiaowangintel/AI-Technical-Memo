# ts_lowering_context.h — Code Analysis / 代码分析

## Source / 来源
- File / 文件: `torch/csrc/lazy/ts_backend/ts_lowering_context.h`
- Repository / 仓库: `pytorch`
- Purpose / 作用:
  - EN: Defines lazy backend abstractions that isolate device-specific behavior.
  - CN: 定义 Lazy 后端抽象，用于隔离设备相关行为。

## Line-by-Line Analysis / 逐行分析

### Lines 1-28
```cpp
 1 | #pragma once
 2 | 
 3 | #include <sstream>
 4 | 
 5 | #include <c10/util/Exception.h>
 6 | #include <torch/csrc/api/include/torch/jit.h>
 7 | #include <torch/csrc/jit/runtime/graph_executor.h>
 8 | #include <torch/csrc/lazy/backend/lowering_context.h>
 9 | #include <torch/csrc/lazy/core/ir.h>
10 | #include <torch/csrc/lazy/ts_backend/ts_node_lowering.h>
11 | 
12 | namespace torch::lazy {
13 | 
14 | using TSOpVector = std::vector<torch::jit::Value*>;
15 | 
16 | class TORCH_API TSComputation : public Computation {
17 |  public:
18 |   TSComputation(const std::shared_ptr<torch::jit::Graph>& graph)
19 |       : graph_(graph), graph_executor_(graph, "") {
20 |     for (torch::jit::Value* input : graph_->inputs()) {
21 |       parameter_names_.push_back(input->debugName());
22 |     }
23 |   }
24 | 
25 |   int parameters_size() const override {
26 |     return static_cast<int>(parameter_names_.size());
27 |   }
28 | 
```
- EN: Brings in project headers such as `<c10/util/Exception.h>`, `<torch/csrc/api/include/torch/jit.h>`, `<torch/csrc/jit/runtime/graph_executor.h>`, `<torch/csrc/lazy/backend/lowering_context.h>` and system or third-party headers such as `<sstream>` so this section can use their types, APIs, or macros. Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Places the implementation in namespace scopes (`torch::lazy`) so ownership matches the PyTorch subsystem layout. Defines or extends data abstractions such as `TSComputation` that structure the state handled by this file.
- CN: 这里引入了项目头文件，例如 `<c10/util/Exception.h>`、`<torch/csrc/api/include/torch/jit.h>`、`<torch/csrc/jit/runtime/graph_executor.h>`、`<torch/csrc/lazy/backend/lowering_context.h>`以及系统或第三方头文件，例如 `<sstream>`，使本段代码能够使用相关类型、API 或宏。 使用预处理指令来控制编译开关、平台分支或导出声明。 把实现放入命名空间作用域（`torch::lazy`），使其归属与 PyTorch 子系统布局保持一致。 定义或扩展了 `TSComputation` 等数据抽象，用来组织本文件处理的状态。

### Lines 29-58
```cpp
29 |   const std::vector<Shape>& parameter_shapes() const override {
30 |     TORCH_CHECK(
31 |         false, "TODO(whc) implement TS computation shapes or change interface");
32 |     return parameter_shapes_;
33 |   }
34 | 
35 |   const std::vector<std::string>& parameter_names() const override {
36 |     return parameter_names_;
37 |   }
38 | 
39 |   const Shape& result_shape() const override {
40 |     TORCH_CHECK(
41 |         false, "TODO(whc) implement TS computation shapes or change interface");
42 |     return result_shape_;
43 |   }
44 | 
45 |   const std::string to_string() const override {
46 |     std::ostringstream oss;
47 |     oss << *graph_;
48 |     return oss.str();
49 |   }
50 | 
51 |   std::shared_ptr<torch::jit::Graph> graph() const {
52 |     return graph_;
53 |   }
54 | 
55 |   torch::jit::GraphExecutor& graph_executor() {
56 |     return graph_executor_;
57 |   }
58 | 
```
- EN: Implements routines such as `parameter_shapes`, `parameter_names`, `result_shape`, `to_string`, `graph` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures; stores long-lived member state for later calls.
- CN: 实现了 `parameter_shapes`、`parameter_names`、`result_shape`、`to_string`、`graph` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误；保存供后续调用使用的长期成员状态。

### Lines 59-90
```cpp
59 |  private:
60 |   std::shared_ptr<torch::jit::Graph> graph_;
61 |   torch::jit::GraphExecutor graph_executor_;
62 |   std::vector<std::string> parameter_names_;
63 |   std::vector<Shape> parameter_shapes_;
64 |   Shape result_shape_;
65 | };
66 | 
67 | class TORCH_API TSLoweringContext : public LoweringContext {
68 |  public:
69 |   TSLoweringContext(const std::string& name, const BackendDevice device);
70 | 
71 |   TSLoweringContext(
72 |       const std::string& name,
73 |       BackendDevice device,
74 |       c10::ArrayRef<const Node*> post_order,
75 |       Util::EmissionMap emit_status);
76 | 
77 |   size_t AddResult(const Output& output) override {
78 |     return AddResult(GetOutputOp(output));
79 |   }
80 | 
81 |   void AddParameter(
82 |       const torch::lazy::Output& output,
83 |       size_t index,
84 |       const Shape& shape,
85 |       const std::string& name) override {
86 |     TORCH_INTERNAL_ASSERT(false, "not implemented");
87 |   }
88 | 
89 |   void Lower(const Node* node);
90 | 
```
- EN: Defines or extends data abstractions such as `TSLoweringContext` that structure the state handled by this file. Implements routines such as `TSLoweringContext`, `AddResult`, `AddParameter`, `Lower` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; stores long-lived member state for later calls.
- CN: 定义或扩展了 `TSLoweringContext` 等数据抽象，用来组织本文件处理的状态。 实现了 `TSLoweringContext`、`AddResult`、`AddParameter`、`Lower` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；保存供后续调用使用的长期成员状态。

### Lines 91-118
```cpp
 91 |   ComputationPtr Build() override {
 92 |     for (torch::jit::Value* output : root_tuple_) {
 93 |       graph_->block()->registerOutput(output);
 94 |     }
 95 |     return std::make_shared<TSComputation>(graph_);
 96 |   }
 97 | 
 98 |   // Retrieves the lowered operation for an output. If the requested output is
 99 |   // not available yet, the graph behind the output's Node is lowered, and the
100 |   // corresponding TS operation returned.
101 |   torch::jit::Value* GetOutputOp(const Output& output) {
102 |     auto it = emitted_outputs_.find(output);
103 |     if (it == emitted_outputs_.end()) {
104 |       auto post_order = Util::ComputePostOrder(output.node, &emit_status_);
105 |       for (auto node : post_order) {
106 |         Lower(node);
107 |       }
108 |       // At this point the output better be present, otherwise there is an issue
109 |       // with the lowering code.
110 |       it = emitted_outputs_.find(output);
111 |       TORCH_CHECK(
112 |           it != emitted_outputs_.end(),
113 |           "No TS operation emitted for output: ",
114 |           output.ToString());
115 |     }
116 |     return it->second;
117 |   }
118 | 
```
- EN: Implements routines such as `Build`, `GetOutputOp` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 实现了 `Build`、`GetOutputOp` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

### Lines 119-150
```cpp
119 |   // Assigns the given TS operation to the specified output. As outputs are
120 |   // lowered in a post-order fashion, later nodes should always find their
121 |   // operands among the emitted outputs.
122 |   void AssignOutputOp(const Output& output, torch::jit::Value* op);
123 | 
124 |   // If a parameter associated with data has already been declared, it will be
125 |   // returned. Otherwise a new one will be created, associated with the tensor
126 |   // held in data.
127 |   torch::jit::Value* GetParameter(const BackendDataPtr& data);
128 | 
129 |   std::shared_ptr<torch::jit::Graph> graph() const {
130 |     return graph_;
131 |   }
132 | 
133 |  private:
134 |   struct Parameter {
135 |     torch::jit::Value* param{nullptr};
136 |     size_t index = 0;
137 |   };
138 | 
139 |   size_t AddResult(torch::jit::Value* op) {
140 |     root_tuple_.push_back(op);
141 |     return root_tuple_.size() - 1;
142 |   }
143 | 
144 |   std::shared_ptr<torch::jit::Graph> graph_;
145 |   std::shared_ptr<torch::jit::GraphFunction> function_;
146 |   std::unordered_map<BackendData::Handle, Parameter> parameters_map_;
147 |   std::vector<torch::jit::Value*> root_tuple_;
148 |   OutputMap<torch::jit::Value*> emitted_outputs_;
149 | };
150 | 
```
- EN: Defines or extends data abstractions such as `Parameter` that structure the state handled by this file. Implements routines such as `AssignOutputOp`, `GetParameter`, `graph`, `AddResult` that expose the key API or control flow of this region. At the statement level, this block declares abstract hooks that derived backends must provide; returns computed state or forwards values to callers; stores long-lived member state for later calls.
- CN: 定义或扩展了 `Parameter` 等数据抽象，用来组织本文件处理的状态。 实现了 `AssignOutputOp`、`GetParameter`、`graph`、`AddResult` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段声明了需要由派生后端提供的抽象钩子；返回计算状态或把值转交给调用方；保存供后续调用使用的长期成员状态。

### Lines 151-151
```cpp
151 | } // namespace torch::lazy
```
- EN: Continues the file's main role: Defines lazy backend abstractions that isolate device-specific behavior.
- CN: 继续承担本文件的主要职责：定义 Lazy 后端抽象，用于隔离设备相关行为。

## Key Concepts / 关键概念

- **Primary types / 主要类型**
  - EN: `TSComputation`, `TSLoweringContext`, `Parameter`.
  - CN: `TSComputation`、`TSLoweringContext`、`Parameter`。
- **Important routines / 重要例程**
  - EN: `TSComputation`, `parameters_size`, `parameter_shapes`, `parameter_names`, `result_shape`, `to_string`, `graph`, `graph_executor`.
  - CN: `TSComputation`、`parameters_size`、`parameter_shapes`、`parameter_names`、`result_shape`、`to_string`、`graph`、`graph_executor`。
- **Subsystem scope / 子系统作用域**
  - EN: The code lives under `torch::lazy`, indicating the owning PyTorch subsystem.
  - CN: 代码位于 `torch::lazy` 命名空间下，这说明了其所属的 PyTorch 子系统。

## Dependencies / 依赖关系

- Local includes / 本地头文件: `<c10/util/Exception.h>`, `<torch/csrc/api/include/torch/jit.h>`, `<torch/csrc/jit/runtime/graph_executor.h>`, `<torch/csrc/lazy/backend/lowering_context.h>`, `<torch/csrc/lazy/core/ir.h>`, `<torch/csrc/lazy/ts_backend/ts_node_lowering.h>`
- External includes / 外部头文件: `<sstream>`
- Relationship / 关系:
  - EN: The unit relies on neighboring PyTorch headers for subsystem-specific types, dispatcher hooks, and utility helpers. Standard-library or third-party headers provide generic facilities such as containers, threading, math, or backend integration points. Its exported or visible routines are then consumed by callers in the same namespace and by higher-level runtime paths elsewhere in PyTorch.
  - CN: 该编译单元依赖相邻的 PyTorch 头文件，以获得子系统类型、分发器钩子和辅助工具。 标准库或第三方头文件提供了通用能力，例如容器、线程、数学支持或后端集成入口。 随后，这里导出或可见的例程会被同一命名空间中的调用方，以及 PyTorch 其他位置的更高层运行时路径所使用。
