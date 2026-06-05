# perf.h — Code Analysis / 代码分析

## Source / 来源
- File / 文件: `torch/csrc/profiler/perf.h`
- Repository / 仓库: `pytorch`
- Purpose / 作用:
  - EN: Implements PyTorch runtime profiling, Kineto integration, and trace collection.
  - CN: 实现 PyTorch 运行时 profiling、Kineto 集成与跟踪采集。

## Line-by-Line Analysis / 逐行分析

### Lines 1-32
```cpp
 1 | #pragma once
 2 | 
 3 | #include <array>
 4 | #include <cstdint>
 5 | #include <memory>
 6 | #include <stack>
 7 | #include <string>
 8 | #include <unordered_map>
 9 | #include <utility>
10 | #include <vector>
11 | 
12 | #include <torch/csrc/profiler/events.h>
13 | 
14 | #include <c10/util/Exception.h>
15 | 
16 | namespace torch::profiler::impl::linux_perf {
17 | 
18 | /*
19 |  * Maximum number of events supported
20 |  * This stems from the hardware limitation on CPU performance counters, and the
21 |  * fact that we don't support time multiplexing just yet.
22 |  * Time multiplexing involves scaling the counter values proportional to
23 |  * the enabled and running time or running the workload multiple times.
24 |  */
25 | constexpr uint8_t MAX_EVENTS = 4;
26 | 
27 | struct PerfCounter {
28 |   uint64_t value; /* The value of the event */
29 |   uint64_t time_enabled; /* for TIME_ENABLED */
30 |   uint64_t time_running; /* for TIME_RUNNING */
31 | };
32 | 
```
- EN: Brings in project headers such as `<torch/csrc/profiler/events.h>`, `<c10/util/Exception.h>` and system or third-party headers such as `<array>`, `<cstdint>`, `<memory>`, `<stack>` so this section can use their types, APIs, or macros. Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Places the implementation in namespace scopes (`torch::profiler::impl::linux_perf`) so ownership matches the PyTorch subsystem layout. Defines or extends data abstractions such as `PerfCounter` that structure the state handled by this file.
- CN: 这里引入了项目头文件，例如 `<torch/csrc/profiler/events.h>`、`<c10/util/Exception.h>`以及系统或第三方头文件，例如 `<array>`、`<cstdint>`、`<memory>`、`<stack>`，使本段代码能够使用相关类型、API 或宏。 使用预处理指令来控制编译开关、平台分支或导出声明。 把实现放入命名空间作用域（`torch::profiler::impl::linux_perf`），使其归属与 PyTorch 子系统布局保持一致。 定义或扩展了 `PerfCounter` 等数据抽象，用来组织本文件处理的状态。

### Lines 33-63
```cpp
33 | /*
34 |  * Basic perf event handler for Android and Linux
35 |  */
36 | class PerfEvent {
37 |  public:
38 |   explicit PerfEvent(std::string& name) : name_(name) {}
39 | 
40 |   PerfEvent(const PerfEvent& other) = delete;
41 |   PerfEvent& operator=(const PerfEvent&) = delete;
42 |   PerfEvent& operator=(PerfEvent&& other) noexcept {
43 |     if (this != &other) {
44 |       fd_ = other.fd_;
45 |       other.fd_ = -1;
46 |       name_ = std::move(other.name_);
47 |     }
48 |     return *this;
49 |   }
50 | 
51 |   PerfEvent(PerfEvent&& other) noexcept {
52 |     *this = std::move(other);
53 |   }
54 | 
55 |   ~PerfEvent();
56 | 
57 |   /* Setup perf events with the Linux Kernel, attaches perf to this process
58 |    * using perf_event_open(2) */
59 |   void Init();
60 | 
61 |   /* Stop incrementing hardware counters for this event */
62 |   void Disable() const;
63 | 
```
- EN: Defines or extends data abstractions such as `PerfEvent` that structure the state handled by this file. Implements routines such as `PerfEvent`, `Init`, `Disable` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; uses move semantics to avoid unnecessary copies.
- CN: 定义或扩展了 `PerfEvent` 等数据抽象，用来组织本文件处理的状态。 实现了 `PerfEvent`、`Init`、`Disable` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；使用移动语义来避免不必要的拷贝。

### Lines 64-92
```cpp
64 |   /* Start counting hardware event from this point on */
65 |   void Enable() const;
66 | 
67 |   /* Zero out the counts for this event */
68 |   void Reset() const;
69 | 
70 |   /* Returns PerfCounter values for this event from kernel, on non supported
71 |    * platforms this always returns zero */
72 |   uint64_t ReadCounter() const;
73 | 
74 |  private:
75 |   /* Name of the event */
76 |   std::string name_;
77 | 
78 |   int fd_ = -1;
79 | };
80 | 
81 | class PerfProfiler {
82 |  public:
83 |   /* Configure all the events and track them as individual PerfEvent */
84 |   void Configure(std::vector<std::string>& event_names);
85 | 
86 |   /* Enable events counting from here */
87 |   void Enable();
88 | 
89 |   /* Disable counting and fill in the caller supplied container with delta
90 |    * calculated from the start count values since last Enable() */
91 |   void Disable(perf_counters_t& /*vals*/);
92 | 
```
- EN: Defines or extends data abstractions such as `PerfProfiler` that structure the state handled by this file. Implements routines such as `Enable`, `Reset`, `ReadCounter`, `Configure`, `Disable` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; stores long-lived member state for later calls.
- CN: 定义或扩展了 `PerfProfiler` 等数据抽象，用来组织本文件处理的状态。 实现了 `Enable`、`Reset`、`ReadCounter`、`Configure`、`Disable` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；保存供后续调用使用的长期成员状态。

### Lines 93-101
```cpp
 93 |  private:
 94 |   uint64_t CalcDelta(uint64_t start, uint64_t end) const;
 95 |   void StartCounting() const;
 96 |   void StopCounting() const;
 97 | 
 98 |   std::vector<PerfEvent> events_;
 99 |   std::stack<perf_counters_t> start_values_;
100 | };
101 | } // namespace torch::profiler::impl::linux_perf
```
- EN: Declares routines such as `CalcDelta`, `StartCounting`, `StopCounting` that expose the key API or control flow of this region. At the statement level, this block stores long-lived member state for later calls.
- CN: 声明了 `CalcDelta`、`StartCounting`、`StopCounting` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段保存供后续调用使用的长期成员状态。

## Key Concepts / 关键概念

- **Primary types / 主要类型**
  - EN: `PerfEvent`, `PerfProfiler`, `PerfCounter`.
  - CN: `PerfEvent`、`PerfProfiler`、`PerfCounter`。
- **Important routines / 重要例程**
  - EN: `PerfEvent`, `Init`, `Disable`, `Enable`, `Reset`, `ReadCounter`, `Configure`, `CalcDelta`.
  - CN: `PerfEvent`、`Init`、`Disable`、`Enable`、`Reset`、`ReadCounter`、`Configure`、`CalcDelta`。
- **Subsystem scope / 子系统作用域**
  - EN: The code lives under `torch::profiler::impl::linux_perf`, indicating the owning PyTorch subsystem.
  - CN: 代码位于 `torch::profiler::impl::linux_perf` 命名空间下，这说明了其所属的 PyTorch 子系统。

## Dependencies / 依赖关系

- Local includes / 本地头文件: `<torch/csrc/profiler/events.h>`, `<c10/util/Exception.h>`
- External includes / 外部头文件: `<array>`, `<cstdint>`, `<memory>`, `<stack>`, `<string>`, `<unordered_map>`, `<utility>`, `<vector>`
- Relationship / 关系:
  - EN: The unit relies on neighboring PyTorch headers for subsystem-specific types, dispatcher hooks, and utility helpers. Standard-library or third-party headers provide generic facilities such as containers, threading, math, or backend integration points. Its exported or visible routines are then consumed by callers in the same namespace and by higher-level runtime paths elsewhere in PyTorch.
  - CN: 该编译单元依赖相邻的 PyTorch 头文件，以获得子系统类型、分发器钩子和辅助工具。 标准库或第三方头文件提供了通用能力，例如容器、线程、数学支持或后端集成入口。 随后，这里导出或可见的例程会被同一命名空间中的调用方，以及 PyTorch 其他位置的更高层运行时路径所使用。
