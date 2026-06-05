# util.h — Code Analysis / 代码分析

## Source / 来源
- File / 文件: `torch/csrc/profiler/util.h`
- Repository / 仓库: `pytorch`
- Purpose / 作用:
  - EN: Implements PyTorch runtime profiling, Kineto integration, and trace collection.
  - CN: 实现 PyTorch 运行时 profiling、Kineto 集成与跟踪采集。

## Line-by-Line Analysis / 逐行分析

### Lines 1-36
```cpp
 1 | #pragma once
 2 | 
 3 | #include <cstddef>
 4 | #include <cstdint>
 5 | #include <list>
 6 | #include <string>
 7 | #include <unordered_map>
 8 | #include <vector>
 9 | 
10 | #include <ATen/record_function.h>
11 | #include <c10/macros/Macros.h>
12 | #include <c10/util/hash.h>
13 | #include <torch/csrc/Export.h>
14 | #include <torch/csrc/jit/frontend/source_range.h>
15 | #include <optional>
16 | 
17 | // TODO: replace with pytorch/rfcs#43 when it is ready.
18 | #define SOFT_ASSERT(cond, ...)                         \
19 |   [&]() -> bool {                                      \
20 |     if (C10_UNLIKELY(!(cond))) {                       \
21 |       torch::profiler::impl::logSoftAssert(            \
22 |           __func__,                                    \
23 |           __FILE__,                                    \
24 |           static_cast<uint32_t>(__LINE__),             \
25 |           #cond,                                       \
26 |           ::c10::str(__VA_ARGS__));                    \
27 |       if (torch::profiler::impl::softAssertRaises()) { \
28 |         TORCH_INTERNAL_ASSERT(cond, __VA_ARGS__);      \
29 |       } else {                                         \
30 |         TORCH_WARN_ONCE(__VA_ARGS__);                  \
31 |       }                                                \
32 |       return false;                                    \
33 |     }                                                  \
34 |     return true;                                       \
35 |   }()
36 | 
```
- EN: Brings in project headers such as `<ATen/record_function.h>`, `<c10/macros/Macros.h>`, `<c10/util/hash.h>`, `<torch/csrc/Export.h>` and system or third-party headers such as `<cstddef>`, `<cstdint>`, `<list>`, `<string>` so this section can use their types, APIs, or macros. Uses preprocessor directives to guard compilation, platform branches, or exported declarations. At the statement level, this block returns computed state or forwards values to callers.
- CN: 这里引入了项目头文件，例如 `<ATen/record_function.h>`、`<c10/macros/Macros.h>`、`<c10/util/hash.h>`、`<torch/csrc/Export.h>`以及系统或第三方头文件，例如 `<cstddef>`、`<cstdint>`、`<list>`、`<string>`，使本段代码能够使用相关类型、API 或宏。 使用预处理指令来控制编译开关、平台分支或导出声明。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 37-72
```cpp
37 | namespace torch::profiler::impl {
38 | TORCH_API bool softAssertRaises();
39 | TORCH_API void setSoftAssertRaises(std::optional<bool> value);
40 | TORCH_API void logSoftAssert(
41 |     const char* func,
42 |     const char* file,
43 |     uint32_t line,
44 |     const char* cond,
45 |     const char* args);
46 | inline void logSoftAssert(
47 |     const char* func,
48 |     const char* file,
49 |     uint32_t line,
50 |     const char* cond,
51 |     ::c10::detail::CompileTimeEmptyString args) {
52 |   logSoftAssert(func, file, line, cond, (const char*)args);
53 | }
54 | TORCH_API void logSoftAssert(
55 |     const char* func,
56 |     const char* file,
57 |     uint32_t line,
58 |     const char* cond,
59 |     const std::string& args);
60 | 
61 | using shape =
62 |     std::variant<std::vector<int64_t>, std::vector<std::vector<int64_t>>>;
63 | constexpr int TENSOR_LIST_DISPLAY_LENGTH_LIMIT = 30;
64 | 
65 | std::string getNvtxStr(
66 |     const char* name,
67 |     int64_t sequence_nr,
68 |     const std::vector<std::vector<int64_t>>& shapes,
69 |     at::RecordFunctionHandle op_id = 0,
70 |     const std::list<std::pair<at::RecordFunctionHandle, int>>& input_op_ids =
71 |         {});
72 | 
```
- EN: Places the implementation in namespace scopes (`torch::profiler::impl`) so ownership matches the PyTorch subsystem layout. Implements routines such as `softAssertRaises`, `setSoftAssertRaises`, `logSoftAssert` that expose the key API or control flow of this region.
- CN: 把实现放入命名空间作用域（`torch::profiler::impl`），使其归属与 PyTorch 子系统布局保持一致。 实现了 `softAssertRaises`、`setSoftAssertRaises`、`logSoftAssert` 等例程，它们构成了这一段的关键 API 或控制流程。

### Lines 73-102
```cpp
 73 | struct TORCH_API FileLineFunc {
 74 |   std::string filename;
 75 |   size_t line;
 76 |   std::string funcname;
 77 | };
 78 | 
 79 | struct TORCH_API SaveNcclMetaConfig {
 80 |   bool truncate;
 81 |   bool introspectMetadata;
 82 |   bool introspectInputs;
 83 |   bool introspectOutputs;
 84 | 
 85 |   // Default constructor with default values
 86 |   SaveNcclMetaConfig()
 87 |       : truncate(true),
 88 |         introspectMetadata(true),
 89 |         introspectInputs(false),
 90 |         introspectOutputs(false) {}
 91 | 
 92 |   SaveNcclMetaConfig(
 93 |       bool truncate,
 94 |       bool introspectMetadata,
 95 |       bool introspectInputs,
 96 |       bool introspectOutputs)
 97 |       : truncate(truncate),
 98 |         introspectMetadata(introspectMetadata),
 99 |         introspectInputs(introspectInputs),
100 |         introspectOutputs(introspectOutputs) {}
101 | };
102 | 
```
- EN: Defines or extends data abstractions such as `FileLineFunc`, `SaveNcclMetaConfig` that structure the state handled by this file.
- CN: 定义或扩展了 `FileLineFunc`、`SaveNcclMetaConfig` 等数据抽象，用来组织本文件处理的状态。

### Lines 103-138
```cpp
103 | TORCH_API std::vector<FileLineFunc> prepareCallstack(
104 |     const std::vector<jit::StackEntry>& cs);
105 | TORCH_API std::vector<std::string> callstackStr(
106 |     const std::vector<FileLineFunc>& cs);
107 | TORCH_API std::string stacksToStr(
108 |     const std::vector<std::string>& stacks,
109 |     const char* delim);
110 | TORCH_API std::vector<std::vector<int64_t>> inputSizes(
111 |     const at::RecordFunction& fn,
112 |     const bool flatten_list_enabled = false);
113 | TORCH_API std::string variantShapesToStr(const std::vector<shape>& shapes);
114 | TORCH_API std::string shapesToStr(
115 |     const std::vector<std::vector<int64_t>>& shapes);
116 | TORCH_API std::string strListToStr(const std::vector<std::string>& types);
117 | TORCH_API std::string inputOpIdsToStr(
118 |     const std::list<std::pair<at::RecordFunctionHandle, int>>& input_op_ids);
119 | TORCH_API std::string ivalueToStr(const c10::IValue& val, bool isString);
120 | TORCH_API std::string ivalueListToStr(const std::vector<c10::IValue>& list);
121 | TORCH_API std::vector<std::string> inputTypes(const at::RecordFunction& fn);
122 | 
123 | std::unordered_map<std::string, c10::IValue> TORCH_API
124 | saveExtraArgs(const at::RecordFunction& fn);
125 | std::unordered_map<std::string, std::string> TORCH_API saveNcclMeta(
126 |     const at::RecordFunction& fn,
127 |     const SaveNcclMetaConfig& config = SaveNcclMetaConfig());
128 | int getTensorStartHint(const at::Tensor& t);
129 | bool checkFunctionOutputsForLogging(const at::RecordFunction& fn);
130 | bool checkFunctionInputsForLogging(const at::RecordFunction& fn);
131 | std::pair<bool, std::variant<int, std::vector<int>>> findStartAddrForTensors(
132 |     const c10::IValue& val);
133 | uint64_t TORCH_API computeFlops(
134 |     const std::string& op_name,
135 |     const std::unordered_map<std::string, c10::IValue>& extra_args);
136 | 
137 | std::string shapeToStr(const std::vector<int64_t>& shape);
138 | 
```
- EN: Declares routines such as `prepareCallstack`, `callstackStr`, `stacksToStr`, `inputSizes`, `variantShapesToStr` that expose the key API or control flow of this region.
- CN: 声明了 `prepareCallstack`、`callstackStr`、`stacksToStr`、`inputSizes`、`variantShapesToStr` 等例程，它们构成了这一段的关键 API 或控制流程。

### Lines 139-176
```cpp
139 | template <typename T>
140 | class TORCH_API GlobalStateManager {
141 |  public:
142 |   static GlobalStateManager& singleton() {
143 |     /* library-local */ static GlobalStateManager singleton_;
144 |     return singleton_;
145 |   }
146 | 
147 |   static void push(std::shared_ptr<T>&& state) {
148 |     if (singleton().state_) {
149 |       LOG(WARNING) << "GlobalStatePtr already exists!";
150 |     } else {
151 |       singleton().state_ = std::move(state);
152 |     }
153 |   }
154 | 
155 |   static auto* get() {
156 |     return singleton().state_.get();
157 |   }
158 | 
159 |   static std::shared_ptr<T> pop() {
160 |     auto out = singleton().state_;
161 |     singleton().state_.reset();
162 |     return out;
163 |   }
164 | 
165 |  private:
166 |   GlobalStateManager() = default;
167 | 
168 |   std::shared_ptr<T> state_;
169 | };
170 | 
171 | struct HashCombine {
172 |   template <typename T0, typename T1>
173 |   size_t operator()(const std::pair<T0, T1>& i) {
174 |     return c10::get_hash((*this)(i.first), (*this)(i.second));
175 |   }
176 | 
```
- EN: Defines or extends data abstractions such as `GlobalStateManager`, `HashCombine` that structure the state handled by this file. Implements routines such as `singleton`, `push`, `get`, `pop`, `operator` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; uses move semantics to avoid unnecessary copies; stores long-lived member state for later calls.
- CN: 定义或扩展了 `GlobalStateManager`、`HashCombine` 等数据抽象，用来组织本文件处理的状态。 实现了 `singleton`、`push`、`get`、`pop`、`operator` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；使用移动语义来避免不必要的拷贝；保存供后续调用使用的长期成员状态。

### Lines 177-210
```cpp
177 |   template <typename... Args>
178 |   size_t operator()(const std::tuple<Args...>& i) {
179 |     return c10::get_hash(i);
180 |   }
181 | 
182 |   template <typename T>
183 |   size_t operator()(const T& i) {
184 |     return c10::get_hash(i);
185 |   }
186 | };
187 | 
188 | #ifdef USE_DISTRIBUTED
189 | constexpr auto kCommsName = "Collective name";
190 | constexpr auto kDtype = "dtype";
191 | constexpr auto kInMsgNelems = "In msg nelems";
192 | constexpr auto kOutMsgNelems = "Out msg nelems";
193 | constexpr auto kInSplit = "In split size";
194 | constexpr auto kOutSplit = "Out split size";
195 | constexpr auto kGlobalRankStart = "Global rank start";
196 | constexpr auto kGlobalRankStride = "Global rank stride";
197 | constexpr auto kGroupSize = "Group size";
198 | constexpr auto kProcessGroupName = "Process Group Name";
199 | constexpr auto kProcessGroupDesc = "Process Group Description";
200 | constexpr auto kGroupRanks = "Process Group Ranks";
201 | constexpr auto kRank = "Rank";
202 | constexpr auto kP2pSrc = "Src Rank";
203 | constexpr auto kP2pDst = "Dst Rank";
204 | constexpr auto kSeqNum = "Seq";
205 | constexpr auto kInTensorsStart = "Input Tensors start";
206 | constexpr auto kOutTensorsStart = "Output Tensors start";
207 | constexpr auto kIsAsynchronizedOp = "Is asynchronized op";
208 | constexpr auto kCommsId = "Comms Id";
209 | #endif // USE_DISTRIBUTED
210 | 
```
- EN: Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Implements routines such as `operator` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 使用预处理指令来控制编译开关、平台分支或导出声明。 实现了 `operator` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 211-211
```cpp
211 | } // namespace torch::profiler::impl
```
- EN: Continues the file's main role: Implements PyTorch runtime profiling, Kineto integration, and trace collection.
- CN: 继续承担本文件的主要职责：实现 PyTorch 运行时 profiling、Kineto 集成与跟踪采集。

## Key Concepts / 关键概念

- **Primary types / 主要类型**
  - EN: `GlobalStateManager`, `FileLineFunc`, `SaveNcclMetaConfig`, `HashCombine`.
  - CN: `GlobalStateManager`、`FileLineFunc`、`SaveNcclMetaConfig`、`HashCombine`。
- **Important routines / 重要例程**
  - EN: `softAssertRaises`, `setSoftAssertRaises`, `logSoftAssert`, `prepareCallstack`, `callstackStr`, `stacksToStr`, `inputSizes`, `variantShapesToStr`.
  - CN: `softAssertRaises`、`setSoftAssertRaises`、`logSoftAssert`、`prepareCallstack`、`callstackStr`、`stacksToStr`、`inputSizes`、`variantShapesToStr`。
- **Subsystem scope / 子系统作用域**
  - EN: The code lives under `torch::profiler::impl`, indicating the owning PyTorch subsystem.
  - CN: 代码位于 `torch::profiler::impl` 命名空间下，这说明了其所属的 PyTorch 子系统。

## Dependencies / 依赖关系

- Local includes / 本地头文件: `<ATen/record_function.h>`, `<c10/macros/Macros.h>`, `<c10/util/hash.h>`, `<torch/csrc/Export.h>`, `<torch/csrc/jit/frontend/source_range.h>`
- External includes / 外部头文件: `<cstddef>`, `<cstdint>`, `<list>`, `<string>`, `<unordered_map>`, `<vector>`, `<optional>`
- Relationship / 关系:
  - EN: The unit relies on neighboring PyTorch headers for subsystem-specific types, dispatcher hooks, and utility helpers. Standard-library or third-party headers provide generic facilities such as containers, threading, math, or backend integration points. Its exported or visible routines are then consumed by callers in the same namespace and by higher-level runtime paths elsewhere in PyTorch.
  - CN: 该编译单元依赖相邻的 PyTorch 头文件，以获得子系统类型、分发器钩子和辅助工具。 标准库或第三方头文件提供了通用能力，例如容器、线程、数学支持或后端集成入口。 随后，这里导出或可见的例程会被同一命名空间中的调用方，以及 PyTorch 其他位置的更高层运行时路径所使用。
