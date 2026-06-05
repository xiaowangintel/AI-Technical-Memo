# ir_dump_util.cpp — Code Analysis / 代码分析

## Source / 来源
- File / 文件: `torch/csrc/lazy/core/ir_dump_util.cpp`
- Repository / 仓库: `pytorch`
- Purpose / 作用:
  - EN: Implements the shared Lazy Tensor IR, hashing, shape, and execution support.
  - CN: 实现通用 Lazy Tensor IR、哈希、形状和执行支撑逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-29
```cpp
 1 | #include <torch/csrc/lazy/core/ir_dump_util.h>
 2 | 
 3 | #include <c10/util/irange.h>
 4 | #include <torch/csrc/lazy/backend/backend_interface.h>
 5 | #include <torch/csrc/lazy/backend/lowering_context.h>
 6 | #include <torch/csrc/lazy/core/ir_util.h>
 7 | #include <optional>
 8 | 
 9 | #include <regex>
10 | #include <sstream>
11 | #include <unordered_map>
12 | 
13 | namespace torch::lazy {
14 | namespace {
15 | 
16 | using NodeIdMap = std::unordered_map<const Node*, size_t>;
17 | 
18 | struct AttrTag {
19 |   std::string name;
20 |   std::string value;
21 |   std::string::size_type pos = 0;
22 | };
23 | 
24 | std::string::size_type SkipTagSeparator(
25 |     const std::string& node_string,
26 |     std::string::size_type pos) {
27 |   return node_string.compare(pos, 2, ", ") == 0 ? pos + 2 : pos;
28 | }
29 | 
```
- EN: Brings in project headers such as `<torch/csrc/lazy/core/ir_dump_util.h>`, `<c10/util/irange.h>`, `<torch/csrc/lazy/backend/backend_interface.h>`, `<torch/csrc/lazy/backend/lowering_context.h>` and system or third-party headers such as `<optional>`, `<regex>`, `<sstream>`, `<unordered_map>` so this section can use their types, APIs, or macros. Places the implementation in namespace scopes (`torch::lazy`) so ownership matches the PyTorch subsystem layout. Defines or extends data abstractions such as `AttrTag` that structure the state handled by this file. Implements routines such as `SkipTagSeparator` that expose the key API or control flow of this region.
- CN: 这里引入了项目头文件，例如 `<torch/csrc/lazy/core/ir_dump_util.h>`、`<c10/util/irange.h>`、`<torch/csrc/lazy/backend/backend_interface.h>`、`<torch/csrc/lazy/backend/lowering_context.h>`以及系统或第三方头文件，例如 `<optional>`、`<regex>`、`<sstream>`、`<unordered_map>`，使本段代码能够使用相关类型、API 或宏。 把实现放入命名空间作用域（`torch::lazy`），使其归属与 PyTorch 子系统布局保持一致。 定义或扩展了 `AttrTag` 等数据抽象，用来组织本文件处理的状态。 实现了 `SkipTagSeparator` 等例程，它们构成了这一段的关键 API 或控制流程。

### Lines 30-61
```cpp
30 | std::optional<AttrTag> ParseAttrTag(
31 |     const std::string& node_string,
32 |     std::string::size_type pos) {
33 |   // @lint-ignore-every CLANGTIDY facebook-hte-StdRegexIsAwful
34 |   const std::regex tag_regex("^([a-zA-Z0-9_]+)=");
35 |   std::smatch match;
36 |   // @lint-ignore-every CLANGTIDY facebook-hte-StdRegexIsAwful
37 |   if (!std::regex_search(
38 |           node_string.begin() + static_cast<std::ptrdiff_t>(pos),
39 |           node_string.end(),
40 |           match,
41 |           tag_regex)) {
42 |     return std::nullopt;
43 |   }
44 | 
45 |   std::string::size_type vpos = match[1].second - node_string.begin() + 1;
46 |   std::optional<char> nested_open;
47 |   std::optional<char> nested_close;
48 |   size_t nest_count = 1;
49 |   AttrTag tag;
50 |   tag.name = match[1].str();
51 |   for (pos = vpos; pos < node_string.size(); ++pos) {
52 |     if (!nested_open.has_value()) {
53 |       if (SkipTagSeparator(node_string, pos) != pos) {
54 |         break;
55 |       }
56 |       // NOLINTNEXTLINE(bugprone-switch-missing-default-case)
57 |       switch (node_string[pos]) {
58 |         case '(':
59 |           nested_open = node_string[pos];
60 |           nested_close = ')';
61 |           break;
```
- EN: Implements routines such as `ParseAttrTag`, `tag_regex` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `ParseAttrTag`、`tag_regex` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 62-94
```cpp
62 |         case '[':
63 |           nested_open = node_string[pos];
64 |           nested_close = ']';
65 |           break;
66 |         case '{':
67 |           nested_open = node_string[pos];
68 |           nested_close = '}';
69 |           break;
70 |       }
71 |     } else if (node_string[pos] == nested_close) {
72 |       --nest_count;
73 |       if (nest_count == 0) {
74 |         nest_count = 1;
75 |         nested_open.reset();
76 |         nested_close.reset();
77 |       }
78 |     } else if (node_string[pos] == nested_open) {
79 |       ++nest_count;
80 |     }
81 |   }
82 |   tag.value = node_string.substr(vpos, pos - vpos);
83 |   tag.pos = pos;
84 |   return tag;
85 | }
86 | 
87 | NodeIdMap GenerateIdMap(c10::ArrayRef<const Node*> post_order) {
88 |   NodeIdMap id_map;
89 |   for (auto node : post_order) {
90 |     TORCH_CHECK(id_map.emplace(node, id_map.size()).second, node->ToString());
91 |   }
92 |   return id_map;
93 | }
94 | 
```
- EN: Implements routines such as `GenerateIdMap` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures; builds container state that later execution depends on.
- CN: 实现了 `GenerateIdMap` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误；构建后续执行依赖的容器状态。

### Lines 95-132
```cpp
 95 | std::unordered_map<const Node*, size_t> GetRootsIds(
 96 |     c10::ArrayRef<const Node*> roots) {
 97 |   std::unordered_map<const Node*, size_t> roots_ids;
 98 |   for (const auto i : c10::irange(roots.size())) {
 99 |     roots_ids[roots[i]] = i;
100 |   }
101 |   return roots_ids;
102 | }
103 | 
104 | std::optional<size_t> GetRootNodeId(
105 |     const Node* node,
106 |     const std::unordered_map<const Node*, size_t>& roots_ids) {
107 |   auto it = roots_ids.find(node);
108 |   if (it == roots_ids.end()) {
109 |     return std::nullopt;
110 |   }
111 |   return it->second;
112 | }
113 | 
114 | std::vector<AttrTag> GetNodeTags(const Node* node) {
115 |   std::string node_string = node->ToString();
116 |   std::string op_string = node->op().ToString();
117 |   std::string::size_type pos = node_string.find(op_string);
118 |   TORCH_CHECK(pos != std::string::npos, node_string, " : ", op_string);
119 |   pos += op_string.size();
120 |   std::vector<AttrTag> tags;
121 |   for (;;) {
122 |     pos = SkipTagSeparator(node_string, pos);
123 |     auto tag = ParseAttrTag(node_string, pos);
124 |     if (!tag) {
125 |       break;
126 |     }
127 |     pos = tag->pos;
128 |     tags.push_back(std::move(*tag));
129 |   }
130 |   return tags;
131 | }
132 | 
```
- EN: Implements routines such as `GetRootsIds`, `GetRootNodeId`, `GetNodeTags` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures; uses move semantics to avoid unnecessary copies.
- CN: 实现了 `GetRootsIds`、`GetRootNodeId`、`GetNodeTags` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误；使用移动语义来避免不必要的拷贝。

### Lines 133-161
```cpp
133 | std::string GenerateDotNodeLabel(
134 |     const Node* node,
135 |     const std::unordered_map<const Node*, size_t>& roots_ids) {
136 |   static const size_t kMaxValueSize = 64;
137 |   std::stringstream ss;
138 |   ss << node->op() << "\\n" << node->shape();
139 |   for (auto& tag : GetNodeTags(node)) {
140 |     ss << "\\n" << tag.name << '=';
141 |     if (tag.value.size() < kMaxValueSize) {
142 |       ss << tag.value;
143 |     } else {
144 |       ss << tag.value.substr(0, kMaxValueSize) << "...";
145 |     }
146 |   }
147 |   auto opt_root_id = GetRootNodeId(node, roots_ids);
148 |   if (opt_root_id) {
149 |     ss << "\\nROOT=" << *opt_root_id;
150 |   }
151 |   return ss.str();
152 | }
153 | 
154 | std::string GenerateDotNodeSpec(
155 |     const Node* node,
156 |     const std::unordered_map<const Node*, size_t>& roots_ids) {
157 |   std::stringstream ss;
158 |   ss << "label=\"" << GenerateDotNodeLabel(node, roots_ids) << '"';
159 |   return ss.str();
160 | }
161 | 
```
- EN: Implements routines such as `GenerateDotNodeLabel`, `GenerateDotNodeSpec` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `GenerateDotNodeLabel`、`GenerateDotNodeSpec` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 162-189
```cpp
162 | std::string GenerateTextNodeSpec(const Node* node, const NodeIdMap& id_map) {
163 |   std::stringstream ss;
164 |   ss << node->shapes() << ' ' << node->op() << '(';
165 |   size_t count = 0;
166 |   for (auto& output : node->operands()) {
167 |     if (count > 0) {
168 |       ss << ", ";
169 |     }
170 |     ss << '%' << id_map.at(output.node);
171 |     if (output.node->num_outputs() > 1) {
172 |       ss << '.' << output.index;
173 |     }
174 |     ++count;
175 |   }
176 |   ss << ')';
177 |   for (auto& tag : GetNodeTags(node)) {
178 |     ss << ", " << tag.name << '=' << tag.value;
179 |   }
180 |   return ss.str();
181 | }
182 | 
183 | } // namespace
184 | 
185 | std::string DumpUtil::ToDot(c10::ArrayRef<const Node*> nodes) {
186 |   auto post_order = Util::ComputePostOrder(nodes);
187 |   return PostOrderToDot(post_order, nodes);
188 | }
189 | 
```
- EN: Implements routines such as `GenerateTextNodeSpec`, `PostOrderToDot` that expose the key API or control flow of this region. At the statement level, this block declares abstract hooks that derived backends must provide; returns computed state or forwards values to callers.
- CN: 实现了 `GenerateTextNodeSpec`、`PostOrderToDot` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段声明了需要由派生后端提供的抽象钩子；返回计算状态或把值转交给调用方。

### Lines 190-229
```cpp
190 | std::string DumpUtil::PostOrderToDot(
191 |     c10::ArrayRef<const Node*> post_order,
192 |     c10::ArrayRef<const Node*> roots) {
193 |   std::unordered_map<const Node*, size_t> roots_ids = GetRootsIds(roots);
194 |   NodeIdMap id_map = GenerateIdMap(post_order);
195 |   std::stringstream ss;
196 |   ss << "digraph G {\n";
197 |   for (auto node : post_order) {
198 |     ss << "  node" << id_map.at(node) << " ["
199 |        << GenerateDotNodeSpec(node, roots_ids) << "]\n";
200 |   }
201 |   for (auto it = post_order.rbegin(); it != post_order.rend(); ++it) {
202 |     const Node* node = *it;
203 |     size_t id = id_map.at(node);
204 |     for (const auto i : c10::irange(node->operands().size())) {
205 |       const Output& output = node->operand(i);
206 |       ss << "  node" << id_map.at(output.node) << " -> node" << id;
207 |       if (node->operands().size() > 1) {
208 |         ss << " [label=\"i=" << i;
209 |         if (output.node->num_outputs() > 1) {
210 |           ss << ",o=" << output.index;
211 |         }
212 |         ss << "\"]\n";
213 |       } else {
214 |         if (output.node->num_outputs() > 1) {
215 |           ss << " [label=\"o=" << output.index << "\"]";
216 |         }
217 |         ss << '\n';
218 |       }
219 |     }
220 |   }
221 |   ss << "}\n";
222 |   return ss.str();
223 | }
224 | 
225 | std::string DumpUtil::ToText(c10::ArrayRef<const Node*> nodes) {
226 |   auto post_order = Util::ComputePostOrder(nodes);
227 |   return PostOrderToText(post_order, nodes);
228 | }
229 | 
```
- EN: Implements routines such as `PostOrderToText` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `PostOrderToText` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 230-261
```cpp
230 | std::string DumpUtil::PostOrderToText(
231 |     c10::ArrayRef<const Node*> post_order,
232 |     c10::ArrayRef<const Node*> roots) {
233 |   std::unordered_map<const Node*, size_t> roots_ids = GetRootsIds(roots);
234 |   NodeIdMap id_map = GenerateIdMap(post_order);
235 |   std::stringstream ss;
236 |   ss << "IR {\n";
237 |   for (auto node : post_order) {
238 |     auto opt_root_id = GetRootNodeId(node, roots_ids);
239 |     ss << "  %" << id_map.at(node) << " = "
240 |        << GenerateTextNodeSpec(node, id_map);
241 |     if (opt_root_id) {
242 |       ss << ", ROOT=" << *opt_root_id;
243 |     }
244 |     ss << ", NodeType=" << typeid(*node).name();
245 |     ss << '\n';
246 |   }
247 |   ss << "}\n";
248 |   return ss.str();
249 | }
250 | 
251 | std::string DumpUtil::ToBackend(
252 |     c10::ArrayRef<Value> values,
253 |     const BackendDevice& device) {
254 |   auto lowering_ctx = LoweringContext::Create("IrToBackend", device);
255 |   for (auto& ir_value : values) {
256 |     lowering_ctx->AddResult(ir_value);
257 |   }
258 |   auto computation = lowering_ctx->Build();
259 |   return getBackend()->GetComputationBackendText(computation);
260 | }
261 | 
```
- EN: Implements routines such as `getBackend` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `getBackend` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 262-262
```cpp
262 | } // namespace torch::lazy
```
- EN: Continues the file's main role: Implements the shared Lazy Tensor IR, hashing, shape, and execution support.
- CN: 继续承担本文件的主要职责：实现通用 Lazy Tensor IR、哈希、形状和执行支撑逻辑。

## Key Concepts / 关键概念

- **Primary types / 主要类型**
  - EN: `AttrTag`.
  - CN: `AttrTag`。
- **Important routines / 重要例程**
  - EN: `SkipTagSeparator`, `ParseAttrTag`, `tag_regex`, `GenerateIdMap`, `GetRootsIds`, `GetRootNodeId`, `GetNodeTags`, `GenerateDotNodeLabel`.
  - CN: `SkipTagSeparator`、`ParseAttrTag`、`tag_regex`、`GenerateIdMap`、`GetRootsIds`、`GetRootNodeId`、`GetNodeTags`、`GenerateDotNodeLabel`。
- **Subsystem scope / 子系统作用域**
  - EN: The code lives under `torch::lazy`, indicating the owning PyTorch subsystem.
  - CN: 代码位于 `torch::lazy` 命名空间下，这说明了其所属的 PyTorch 子系统。

## Dependencies / 依赖关系

- Local includes / 本地头文件: `<torch/csrc/lazy/core/ir_dump_util.h>`, `<c10/util/irange.h>`, `<torch/csrc/lazy/backend/backend_interface.h>`, `<torch/csrc/lazy/backend/lowering_context.h>`, `<torch/csrc/lazy/core/ir_util.h>`
- External includes / 外部头文件: `<optional>`, `<regex>`, `<sstream>`, `<unordered_map>`
- Relationship / 关系:
  - EN: The unit relies on neighboring PyTorch headers for subsystem-specific types, dispatcher hooks, and utility helpers. Standard-library or third-party headers provide generic facilities such as containers, threading, math, or backend integration points. Its exported or visible routines are then consumed by callers in the same namespace and by higher-level runtime paths elsewhere in PyTorch.
  - CN: 该编译单元依赖相邻的 PyTorch 头文件，以获得子系统类型、分发器钩子和辅助工具。 标准库或第三方头文件提供了通用能力，例如容器、线程、数学支持或后端集成入口。 随后，这里导出或可见的例程会被同一命名空间中的调用方，以及 PyTorch 其他位置的更高层运行时路径所使用。
