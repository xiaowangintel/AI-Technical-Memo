# perf.cpp — Code Analysis / 代码分析

## Source / 来源
- File / 文件: `torch/csrc/profiler/perf.cpp`
- Repository / 仓库: `pytorch`
- Purpose / 作用:
  - EN: Implements PyTorch runtime profiling, Kineto integration, and trace collection.
  - CN: 实现 PyTorch 运行时 profiling、Kineto 集成与跟踪采集。

## Line-by-Line Analysis / 逐行分析

### Lines 1-38
```cpp
 1 | #include <unordered_map>
 2 | #include <unordered_set>
 3 | 
 4 | #include <c10/util/error.h>
 5 | #include <torch/csrc/profiler/perf-inl.h>
 6 | #include <torch/csrc/profiler/perf.h>
 7 | 
 8 | namespace torch::profiler::impl::linux_perf {
 9 | 
10 | #if defined(__ANDROID__) || defined(__linux__)
11 | 
12 | /*
13 |  * PerfEvent
14 |  * ---------
15 |  */
16 | 
17 | /*
18 |  * Syscall wrapper for perf_event_open(2)
19 |  */
20 | inline static long perf_event_open(
21 |     struct perf_event_attr* hw_event,
22 |     pid_t pid,
23 |     int cpu,
24 |     int group_fd,
25 |     unsigned long flags) {
26 |   return syscall(__NR_perf_event_open, hw_event, pid, cpu, group_fd, flags);
27 | }
28 | 
29 | // TODO sync with Kineto level abstract events in profiler/events.h
30 | static const std::unordered_map<
31 |     std::string,
32 |     std::pair<perf_type_id, /* perf event type */ uint32_t>>
33 |     EventTable{
34 |         {"cycles",
35 |          std::make_pair(PERF_TYPE_HARDWARE, PERF_COUNT_HW_CPU_CYCLES)},
36 |         {"instructions",
37 |          std::make_pair(PERF_TYPE_HARDWARE, PERF_COUNT_HW_INSTRUCTIONS)},
38 | 
```
- EN: Brings in project headers such as `<c10/util/error.h>`, `<torch/csrc/profiler/perf-inl.h>`, `<torch/csrc/profiler/perf.h>` and system or third-party headers such as `<unordered_map>`, `<unordered_set>` so this section can use their types, APIs, or macros. Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Places the implementation in namespace scopes (`torch::profiler::impl::linux_perf`) so ownership matches the PyTorch subsystem layout. Defines or extends data abstractions such as `perf_event_attr` that structure the state handled by this file.
- CN: 这里引入了项目头文件，例如 `<c10/util/error.h>`、`<torch/csrc/profiler/perf-inl.h>`、`<torch/csrc/profiler/perf.h>`以及系统或第三方头文件，例如 `<unordered_map>`、`<unordered_set>`，使本段代码能够使用相关类型、API 或宏。 使用预处理指令来控制编译开关、平台分支或导出声明。 把实现放入命名空间作用域（`torch::profiler::impl::linux_perf`），使其归属与 PyTorch 子系统布局保持一致。 定义或扩展了 `perf_event_attr` 等数据抽象，用来组织本文件处理的状态。

### Lines 39-67
```cpp
39 |         // Non Standard events for testing
40 |         {"pagefaults",
41 |          std::make_pair(PERF_TYPE_SOFTWARE, PERF_COUNT_SW_PAGE_FAULTS)},
42 |         {"backend-stall-cycles",
43 |          std::make_pair(
44 |              PERF_TYPE_HARDWARE,
45 |              PERF_COUNT_HW_STALLED_CYCLES_BACKEND)},
46 |         {"frontend-stall-cycles",
47 |          std::make_pair(
48 |              PERF_TYPE_HARDWARE,
49 |              PERF_COUNT_HW_STALLED_CYCLES_FRONTEND)}};
50 | 
51 | PerfEvent::~PerfEvent() {
52 |   if (fd_ > -1) {
53 |     close(fd_);
54 |   }
55 |   fd_ = -1; // poison
56 | }
57 | 
58 | void PerfEvent::Init() {
59 |   TORCH_CHECK(!name_.empty(), "Invalid profiler event name");
60 | 
61 |   auto const it = EventTable.find(name_);
62 |   if (it == EventTable.end()) {
63 |     TORCH_CHECK(false, "Unsupported profiler event name: ", name_);
64 |   }
65 | 
66 |   struct perf_event_attr attr{};
67 | 
```
- EN: Defines or extends data abstractions such as `perf_event_attr` that structure the state handled by this file. At the statement level, this block validates assumptions and reports descriptive failures.
- CN: 定义或扩展了 `perf_event_attr` 等数据抽象，用来组织本文件处理的状态。 在语句层面，这一段校验前提条件并报告明确错误。

### Lines 68-96
```cpp
68 |   attr.size = sizeof(perf_event_attr);
69 |   attr.type = it->second.first;
70 |   attr.config = it->second.second;
71 |   attr.disabled = 1;
72 |   attr.inherit = 1;
73 |   attr.exclude_kernel = 1; // TBD
74 |   attr.exclude_hv = 1;
75 |   /*
76 |    * These can be used to calculate estimated totals if the PMU is overcommitted
77 |    * and multiplexing is happening
78 |    */
79 |   attr.read_format =
80 |       PERF_FORMAT_TOTAL_TIME_ENABLED | PERF_FORMAT_TOTAL_TIME_RUNNING;
81 | 
82 |   pid_t pid = getpid(); // this pid
83 |   int cpu = -1; // all cpus
84 |   int group_fd = -1;
85 |   unsigned long flags = 0;
86 | 
87 |   fd_ = static_cast<int>(perf_event_open(&attr, pid, cpu, group_fd, flags));
88 |   if (fd_ == -1) {
89 |     TORCH_CHECK(
90 |         false,
91 |         "perf_event_open() failed, error: ",
92 |         c10::utils::str_error(errno));
93 |   }
94 |   Reset();
95 | }
96 | 
```
- EN: At the statement level, this block declares abstract hooks that derived backends must provide; validates assumptions and reports descriptive failures.
- CN: 在语句层面，这一段声明了需要由派生后端提供的抽象钩子；校验前提条件并报告明确错误。

### Lines 97-133
```cpp
 97 | uint64_t PerfEvent::ReadCounter() const {
 98 |   PerfCounter counter{};
 99 |   long n = read(fd_, &counter, sizeof(PerfCounter));
100 |   TORCH_CHECK(
101 |       n == sizeof(counter),
102 |       "Read failed for Perf event fd, event : ",
103 |       name_,
104 |       ", error: ",
105 |       c10::utils::str_error(errno));
106 |   TORCH_CHECK(
107 |       counter.time_enabled == counter.time_running,
108 |       "Hardware performance counter time multiplexing is not handled yet",
109 |       ", name: ",
110 |       name_,
111 |       ", enabled: ",
112 |       counter.time_enabled,
113 |       ", running: ",
114 |       counter.time_running);
115 |   return counter.value;
116 | }
117 | 
118 | #else /* __ANDROID__ || __linux__ */
119 | /*
120 |  * Shim class for unsupported platforms - this will always return 0 counter
121 |  * value
122 |  */
123 | 
124 | PerfEvent::~PerfEvent() {}
125 | 
126 | void PerfEvent::Init() {}
127 | 
128 | uint64_t PerfEvent::ReadCounter() const {
129 |   return 0;
130 | }
131 | 
132 | #endif /* __ANDROID__ || __linux__ */
133 | 
```
- EN: Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Defines or extends data abstractions such as `for` that structure the state handled by this file. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 使用预处理指令来控制编译开关、平台分支或导出声明。 定义或扩展了 `for` 等数据抽象，用来组织本文件处理的状态。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

### Lines 134-172
```cpp
134 | /*
135 |  * PerfProfiler
136 |  * ------------
137 |  */
138 | 
139 | void PerfProfiler::Configure(std::vector<std::string>& event_names) {
140 |   TORCH_CHECK(
141 |       event_names.size() <= MAX_EVENTS,
142 |       "Too many events to configure, configured: ",
143 |       event_names.size(),
144 |       ", max allowed:",
145 |       MAX_EVENTS);
146 |   std::unordered_set<std::string> s(event_names.begin(), event_names.end());
147 |   TORCH_CHECK(
148 |       s.size() == event_names.size(), "Duplicate event names are not allowed!")
149 |   for (auto name : event_names) {
150 |     events_.emplace_back(name);
151 |     events_.back().Init();
152 |   }
153 | 
154 |   // TODO
155 |   // Reset pthreadpool here to make sure we can attach to new children
156 |   // threads
157 | }
158 | 
159 | void PerfProfiler::Enable() {
160 |   if (!start_values_.empty()) {
161 |     StopCounting();
162 |   }
163 | 
164 |   start_values_.emplace(events_.size(), 0);
165 | 
166 |   auto& sv = start_values_.top();
167 |   for (unsigned i = 0; i < events_.size(); ++i) {
168 |     sv[i] = events_[i].ReadCounter();
169 |   }
170 |   StartCounting();
171 | }
172 | 
```
- EN: Implements routines such as `s` that expose the key API or control flow of this region. At the statement level, this block declares abstract hooks that derived backends must provide; validates assumptions and reports descriptive failures; builds container state that later execution depends on.
- CN: 实现了 `s` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段声明了需要由派生后端提供的抽象钩子；校验前提条件并报告明确错误；构建后续执行依赖的容器状态。

### Lines 173-194
```cpp
173 | void PerfProfiler::Disable(perf_counters_t& vals) {
174 |   StopCounting();
175 |   TORCH_CHECK(
176 |       vals.size() == events_.size(),
177 |       "Can not fit all perf counters in the supplied container");
178 |   TORCH_CHECK(
179 |       !start_values_.empty(), "PerfProfiler must be enabled before disabling");
180 | 
181 |   /* Always connecting this disable event to the last enable event i.e. using
182 |    * whatever is on the top of the start counter value stack. */
183 |   perf_counters_t& sv = start_values_.top();
184 |   for (unsigned i = 0; i < events_.size(); ++i) {
185 |     vals[i] = CalcDelta(sv[i], events_[i].ReadCounter());
186 |   }
187 |   start_values_.pop();
188 | 
189 |   // Restore it for a parent
190 |   if (!start_values_.empty()) {
191 |     StartCounting();
192 |   }
193 | }
194 | } // namespace torch::profiler::impl::linux_perf
```
- EN: At the statement level, this block declares abstract hooks that derived backends must provide; validates assumptions and reports descriptive failures.
- CN: 在语句层面，这一段声明了需要由派生后端提供的抽象钩子；校验前提条件并报告明确错误。

## Key Concepts / 关键概念

- **Primary types / 主要类型**
  - EN: `for`, `perf_event_attr`.
  - CN: `for`、`perf_event_attr`。
- **Important routines / 重要例程**
  - EN: `perf_event_open`, `syscall`, `s`.
  - CN: `perf_event_open`、`syscall`、`s`。
- **Subsystem scope / 子系统作用域**
  - EN: The code lives under `torch::profiler::impl::linux_perf`, indicating the owning PyTorch subsystem.
  - CN: 代码位于 `torch::profiler::impl::linux_perf` 命名空间下，这说明了其所属的 PyTorch 子系统。

## Dependencies / 依赖关系

- Local includes / 本地头文件: `<c10/util/error.h>`, `<torch/csrc/profiler/perf-inl.h>`, `<torch/csrc/profiler/perf.h>`
- External includes / 外部头文件: `<unordered_map>`, `<unordered_set>`
- Relationship / 关系:
  - EN: The unit relies on neighboring PyTorch headers for subsystem-specific types, dispatcher hooks, and utility helpers. Standard-library or third-party headers provide generic facilities such as containers, threading, math, or backend integration points. Its exported or visible routines are then consumed by callers in the same namespace and by higher-level runtime paths elsewhere in PyTorch.
  - CN: 该编译单元依赖相邻的 PyTorch 头文件，以获得子系统类型、分发器钩子和辅助工具。 标准库或第三方头文件提供了通用能力，例如容器、线程、数学支持或后端集成入口。 随后，这里导出或可见的例程会被同一命名空间中的调用方，以及 PyTorch 其他位置的更高层运行时路径所使用。
