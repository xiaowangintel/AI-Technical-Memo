# debug_util.cpp — Code Analysis / 代码分析

## Source / 来源
- File / 文件: `torch/csrc/lazy/core/debug_util.cpp`
- Repository / 仓库: `pytorch`
- Purpose / 作用:
  - EN: Implements the shared Lazy Tensor IR, hashing, shape, and execution support.
  - CN: 实现通用 Lazy Tensor IR、哈希、形状和执行支撑逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-36
```cpp
 1 | #include <c10/util/env.h>
 2 | #include <c10/util/irange.h>
 3 | #include <torch/csrc/lazy/core/debug_util.h>
 4 | 
 5 | #include <torch/csrc/lazy/backend/backend_device.h>
 6 | #include <torch/csrc/lazy/core/helpers.h>
 7 | #include <torch/csrc/lazy/core/ir.h>
 8 | #include <torch/csrc/lazy/core/ir_dump_util.h>
 9 | #include <torch/csrc/lazy/core/unique.h>
10 | 
11 | #include <fstream>
12 | #include <mutex>
13 | #include <sstream>
14 | #include <unordered_set>
15 | 
16 | namespace torch::lazy {
17 | namespace {
18 | 
19 | std::string GetEnvString(const char* name, const std::string& defval) {
20 |   const auto env = c10::utils::get_env(name);
21 |   return env.value_or(defval);
22 | }
23 | 
24 | DebugUtil::GraphFormat DefaultGraphFormat() {
25 |   std::string fmt_str = GetEnvString("LTC_SAVE_TENSORS_FMT", "text");
26 |   if (fmt_str == "text") {
27 |     return DebugUtil::GraphFormat::kText;
28 |   } else if (fmt_str == "backend") {
29 |     return DebugUtil::GraphFormat::kBackend;
30 |   } else if (fmt_str == "dot") {
31 |     return DebugUtil::GraphFormat::kDot;
32 |   }
33 |   LOG(ERROR) << "Invalid save graph format: " << fmt_str;
34 |   return DebugUtil::GraphFormat::kText;
35 | }
36 | 
```
- EN: Brings in project headers such as `<c10/util/env.h>`, `<c10/util/irange.h>`, `<torch/csrc/lazy/core/debug_util.h>`, `<torch/csrc/lazy/backend/backend_device.h>` and system or third-party headers such as `<fstream>`, `<mutex>`, `<sstream>`, `<unordered_set>` so this section can use their types, APIs, or macros. Places the implementation in namespace scopes (`torch::lazy`) so ownership matches the PyTorch subsystem layout. Implements routines such as `GetEnvString`, `DefaultGraphFormat` that expose the key API or control flow of this region. Reads environment switches (`LTC_SAVE_TENSORS_FMT`) to tune runtime behavior.
- CN: 这里引入了项目头文件，例如 `<c10/util/env.h>`、`<c10/util/irange.h>`、`<torch/csrc/lazy/core/debug_util.h>`、`<torch/csrc/lazy/backend/backend_device.h>`以及系统或第三方头文件，例如 `<fstream>`、`<mutex>`、`<sstream>`、`<unordered_set>`，使本段代码能够使用相关类型、API 或宏。 把实现放入命名空间作用域（`torch::lazy`），使其归属与 PyTorch 子系统布局保持一致。 实现了 `GetEnvString`、`DefaultGraphFormat` 等例程，它们构成了这一段的关键 API 或控制流程。 读取环境变量开关（`LTC_SAVE_TENSORS_FMT`）来调整运行时行为。

### Lines 37-74
```cpp
37 | std::unordered_set<std::string>* LoadExperiments() {
38 |   std::unique_ptr<std::unordered_set<std::string>> xset =
39 |       std::make_unique<std::unordered_set<std::string>>();
40 |   std::string experiments = GetEnvString("LTC_EXPERIMENTAL", "");
41 |   std::vector<std::string> experiment_list =
42 |       torch::lazy::StrSplit(experiments, ':');
43 |   for (auto& name : experiment_list) {
44 |     xset->insert(name);
45 |   }
46 |   return xset.release();
47 | }
48 | 
49 | } // namespace
50 | 
51 | static std::vector<SourceLocation> NoPythonFrames() {
52 |   SourceLocation dummy_loc;
53 |   dummy_loc.file = "No Python Frames";
54 |   return {dummy_loc};
55 | }
56 | 
57 | std::function<std::vector<SourceLocation>()>& GetPythonFramesFunction() {
58 |   static std::function<std::vector<SourceLocation>()> func_ = NoPythonFrames;
59 |   return func_;
60 | }
61 | 
62 | DebugUtil::GraphFormat DebugUtil::GetDefaultGraphFormat() {
63 |   static GraphFormat format = DefaultGraphFormat();
64 |   return format;
65 | }
66 | 
67 | std::string GetFirstUserFrameInPython() {
68 |   std::string empty;
69 |   if (!torch::lazy::GetPythonFramesFunction()) {
70 |     return empty;
71 |   }
72 | 
73 |   auto frames = torch::lazy::GetPythonFramesFunction()();
74 | 
```
- EN: Implements routines such as `LoadExperiments`, `NoPythonFrames`, `GetFirstUserFrameInPython` that expose the key API or control flow of this region. Reads environment switches (`LTC_EXPERIMENTAL`) to tune runtime behavior. At the statement level, this block returns computed state or forwards values to callers; builds container state that later execution depends on; stores long-lived member state for later calls.
- CN: 实现了 `LoadExperiments`、`NoPythonFrames`、`GetFirstUserFrameInPython` 等例程，它们构成了这一段的关键 API 或控制流程。 读取环境变量开关（`LTC_EXPERIMENTAL`）来调整运行时行为。 在语句层面，这一段返回计算状态或把值转交给调用方；构建后续执行依赖的容器状态；保存供后续调用使用的长期成员状态。

### Lines 75-106
```cpp
 75 |   for (auto i = frames.size(); i > 0; i--) {
 76 |     auto& loc = frames[i - 1];
 77 |     if (loc.file.find("site-packages") == std::string::npos) {
 78 |       std::stringstream ss;
 79 |       ss << loc.file << ' ' << loc.function << ' ' << loc.line;
 80 |       return ss.str();
 81 |     }
 82 |   }
 83 |   return empty;
 84 | }
 85 | 
 86 | std::string DebugUtil::GetTensorsGraphInfo(
 87 |     c10::ArrayRef<torch::lazy::LazyTensorPtr> tensors,
 88 |     const std::vector<size_t>* indices,
 89 |     GraphFormat format) {
 90 |   std::vector<const torch::lazy::Node*> root_nodes;
 91 |   std::vector<torch::lazy::Value> root_values;
 92 |   std::vector<torch::lazy::hash_t> root_hashes;
 93 |   torch::lazy::Unique<torch::lazy::BackendDevice> unique_device;
 94 |   if (indices != nullptr) {
 95 |     for (auto index : *indices) {
 96 |       const torch::lazy::LazyTensorPtr& tensor = tensors[index];
 97 |       torch::lazy::Value ir_value = tensor->CurrentIrValue();
 98 |       if (ir_value) {
 99 |         root_nodes.push_back(ir_value.node.get());
100 |         root_hashes.push_back(ir_value.hash());
101 |         root_values.push_back(std::move(ir_value));
102 |         unique_device.set(tensor->GetDevice());
103 |       }
104 |     }
105 |   } else {
106 |     for (auto& tensor : tensors) {
```
- EN: At the statement level, this block returns computed state or forwards values to callers; uses move semantics to avoid unnecessary copies.
- CN: 在语句层面，这一段返回计算状态或把值转交给调用方；使用移动语义来避免不必要的拷贝。

### Lines 107-133
```cpp
107 |       torch::lazy::Value ir_value = tensor->CurrentIrValue();
108 |       if (ir_value) {
109 |         root_nodes.push_back(ir_value.node.get());
110 |         root_hashes.push_back(ir_value.hash());
111 |         root_values.push_back(std::move(ir_value));
112 |         unique_device.set(tensor->GetDevice());
113 |       }
114 |     }
115 |   }
116 |   std::stringstream ss;
117 |   // Call into a function pointer that may backed by python or empty depending
118 |   // on runtime
119 |   std::vector<SourceLocation> frames = GetPythonFramesFunction()();
120 |   ss << "Python Stacktrace:\n";
121 |   for (auto& location : frames) {
122 |     ss << "  " << location.function << " (" << location.file << ':'
123 |        << location.line << ")\n";
124 |   }
125 |   ss << "\nHashes: (";
126 |   for (const auto i : c10::irange(root_hashes.size())) {
127 |     if (i > 0) {
128 |       ss << ", ";
129 |     }
130 |     ss << torch::lazy::HashToString(root_hashes[i]);
131 |   }
132 |   ss << ")\n";
133 | 
```
- EN: At the statement level, this block uses move semantics to avoid unnecessary copies.
- CN: 在语句层面，这一段使用移动语义来避免不必要的拷贝。

### Lines 134-170
```cpp
134 |   std::string graph_str;
135 |   if (format == GraphFormat::kText) {
136 |     graph_str = torch::lazy::DumpUtil::ToText(root_nodes);
137 |   } else if (format == GraphFormat::kDot) {
138 |     graph_str = torch::lazy::DumpUtil::ToDot(root_nodes);
139 |   } else if (format == GraphFormat::kBackend) {
140 |     graph_str = torch::lazy::DumpUtil::ToBackend(
141 |         root_values,
142 |         unique_device ? *unique_device : torch::lazy::BackendDevice());
143 |   } else {
144 |     LOG(ERROR) << "Invalid graph format: " << format;
145 |   }
146 |   ss << "\n## BEGIN_GRAPH\n" << graph_str << "\n## END_GRAPH\n\n";
147 |   return ss.str();
148 | }
149 | 
150 | void DebugUtil::SaveTensorsGraphInfo(
151 |     const char* name,
152 |     c10::ArrayRef<torch::lazy::LazyTensorPtr> tensors,
153 |     const std::vector<size_t>* indices,
154 |     GraphFormat format) {
155 |   static const std::string save_file =
156 |       GetEnvString("LTC_SAVE_TENSORS_FILE", "");
157 |   if (!save_file.empty()) {
158 |     static std::mutex lock;
159 |     std::string info = GetTensorsGraphInfo(tensors, indices, format);
160 |     std::lock_guard<std::mutex> guard(lock);
161 |     std::ofstream graph_file(save_file, std::ios_base::app);
162 |     graph_file << '[' << name << "]\n" << info << '\n';
163 |   }
164 | }
165 | 
166 | bool DebugUtil::ExperimentEnabled(const std::string& name) {
167 |   static const std::unordered_set<std::string>* xset = LoadExperiments();
168 |   return xset->find(name) != xset->end();
169 | }
170 | 
```
- EN: Implements routines such as `guard`, `graph_file` that expose the key API or control flow of this region. Reads environment switches (`LTC_SAVE_TENSORS_FILE`) to tune runtime behavior. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `guard`、`graph_file` 等例程，它们构成了这一段的关键 API 或控制流程。 读取环境变量开关（`LTC_SAVE_TENSORS_FILE`）来调整运行时行为。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 171-171
```cpp
171 | } // namespace torch::lazy
```
- EN: Continues the file's main role: Implements the shared Lazy Tensor IR, hashing, shape, and execution support.
- CN: 继续承担本文件的主要职责：实现通用 Lazy Tensor IR、哈希、形状和执行支撑逻辑。

## Key Concepts / 关键概念

- **Important routines / 重要例程**
  - EN: `GetEnvString`, `DefaultGraphFormat`, `LoadExperiments`, `NoPythonFrames`, `GetFirstUserFrameInPython`, `guard`, `graph_file`.
  - CN: `GetEnvString`、`DefaultGraphFormat`、`LoadExperiments`、`NoPythonFrames`、`GetFirstUserFrameInPython`、`guard`、`graph_file`。
- **Subsystem scope / 子系统作用域**
  - EN: The code lives under `torch::lazy`, indicating the owning PyTorch subsystem.
  - CN: 代码位于 `torch::lazy` 命名空间下，这说明了其所属的 PyTorch 子系统。

## Dependencies / 依赖关系

- Local includes / 本地头文件: `<c10/util/env.h>`, `<c10/util/irange.h>`, `<torch/csrc/lazy/core/debug_util.h>`, `<torch/csrc/lazy/backend/backend_device.h>`, `<torch/csrc/lazy/core/helpers.h>`, `<torch/csrc/lazy/core/ir.h>`, `<torch/csrc/lazy/core/ir_dump_util.h>`, `<torch/csrc/lazy/core/unique.h>`
- External includes / 外部头文件: `<fstream>`, `<mutex>`, `<sstream>`, `<unordered_set>`
- Relationship / 关系:
  - EN: The unit relies on neighboring PyTorch headers for subsystem-specific types, dispatcher hooks, and utility helpers. Standard-library or third-party headers provide generic facilities such as containers, threading, math, or backend integration points. Its exported or visible routines are then consumed by callers in the same namespace and by higher-level runtime paths elsewhere in PyTorch.
  - CN: 该编译单元依赖相邻的 PyTorch 头文件，以获得子系统类型、分发器钩子和辅助工具。 标准库或第三方头文件提供了通用能力，例如容器、线程、数学支持或后端集成入口。 随后，这里导出或可见的例程会被同一命名空间中的调用方，以及 PyTorch 其他位置的更高层运行时路径所使用。
