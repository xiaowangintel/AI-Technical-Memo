# throughput_benchmark-inl.h — Code Analysis / 代码分析

## Source / 来源
- File / 文件: `torch/csrc/utils/throughput_benchmark-inl.h`
- Repository / 仓库: `pytorch`
- Purpose / 作用:
  - EN: Provides miscellaneous C++/Python interop, argument parsing, and runtime utility helpers.
  - CN: 提供杂项的 C++/Python 互操作、参数解析和运行时工具辅助函数。

## Line-by-Line Analysis / 逐行分析

### Lines 1-27
```cpp
 1 | #pragma once
 2 | 
 3 | #include <random>
 4 | #include <thread>
 5 | 
 6 | #include <torch/csrc/autograd/profiler.h>
 7 | #include <torch/csrc/jit/python/pybind_utils.h>
 8 | #include <torch/csrc/utils/pybind.h>
 9 | 
10 | #include <ATen/Parallel.h>
11 | #include <ATen/autocast_mode.h>
12 | #include <c10/core/GradMode.h>
13 | #include <c10/core/impl/LocalDispatchKeySet.h>
14 | #include <c10/util/irange.h>
15 | 
16 | namespace torch::throughput_benchmark::detail {
17 | 
18 | template <class Input, class Output, class Model>
19 | BenchmarkExecutionStats BenchmarkHelper<Input, Output, Model>::benchmark(
20 |     const BenchmarkConfig& config) const {
21 |   CHECK(initialized_);
22 |   TORCH_CHECK(
23 |       config.num_worker_threads == 1,
24 |       "Only parallelization by callers is supported");
25 | 
26 |   LOG(INFO) << at::get_parallel_info();
27 | 
```
- EN: Brings in project headers such as `<torch/csrc/autograd/profiler.h>`, `<torch/csrc/jit/python/pybind_utils.h>`, `<torch/csrc/utils/pybind.h>`, `<ATen/Parallel.h>` and system or third-party headers such as `<random>`, `<thread>` so this section can use their types, APIs, or macros. Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Places the implementation in namespace scopes (`torch::throughput_benchmark::detail`) so ownership matches the PyTorch subsystem layout. Defines or extends data abstractions such as `Input`, `Output`, `Model` that structure the state handled by this file.
- CN: 这里引入了项目头文件，例如 `<torch/csrc/autograd/profiler.h>`、`<torch/csrc/jit/python/pybind_utils.h>`、`<torch/csrc/utils/pybind.h>`、`<ATen/Parallel.h>`以及系统或第三方头文件，例如 `<random>`、`<thread>`，使本段代码能够使用相关类型、API 或宏。 使用预处理指令来控制编译开关、平台分支或导出声明。 把实现放入命名空间作用域（`torch::throughput_benchmark::detail`），使其归属与 PyTorch 子系统布局保持一致。 定义或扩展了 `Input`、`Output`、`Model` 等数据抽象，用来组织本文件处理的状态。

### Lines 28-64
```cpp
28 |   // We pre-generate inputs here for each of the threads. This allows us to
29 |   // safely move inputs out for each of the threads independently and thus avoid
30 |   // overhead from the benchmark runner itself
31 |   std::vector<std::vector<Input>> thread_inputs(config.num_calling_threads);
32 |   std::vector<size_t> input_iters(config.num_calling_threads);
33 |   {
34 |     std::random_device seeder;
35 |     std::mt19937 engine(seeder());
36 |     TORCH_CHECK(
37 |         !inputs_.empty(),
38 |         "Please provide benchmark inputs."
39 |         "Did you forget to call add_input()? ");
40 |     std::uniform_int_distribution<int> dist(0, inputs_.size() - 1);
41 | 
42 |     for (const auto thread_id : c10::irange(config.num_calling_threads)) {
43 |       // Just in case we generate num_iters inputs for each of the threads
44 |       // This was if one thread does all the work we will be fine
45 |       for (const auto i [[maybe_unused]] :
46 |            c10::irange(config.num_iters + config.num_warmup_iters)) {
47 |         thread_inputs[thread_id].push_back(cloneInput(inputs_[dist(engine)]));
48 |       }
49 |       input_iters[thread_id] = 0;
50 |     }
51 |   }
52 | 
53 |   std::mutex m;
54 |   std::condition_variable worker_main_cv;
55 |   std::condition_variable main_worker_cv;
56 |   // TODO: add GUARDED_BY once it is available
57 |   int64_t initialized{0};
58 |   int64_t finished{0};
59 |   bool start{false};
60 |   std::atomic<int64_t> num_attempted_iters{0};
61 |   std::vector<std::thread> callers;
62 | 
63 |   callers.reserve(config.num_calling_threads);
64 | 
```
- EN: Implements routines such as `thread_inputs`, `input_iters`, `engine`, `dist` that expose the key API or control flow of this region. At the statement level, this block declares abstract hooks that derived backends must provide; validates assumptions and reports descriptive failures.
- CN: 实现了 `thread_inputs`、`input_iters`、`engine`、`dist` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段声明了需要由派生后端提供的抽象钩子；校验前提条件并报告明确错误。

### Lines 65-88
```cpp
65 |   static constexpr auto& DEVICES = at::autocast::_AUTOCAST_SUPPORTED_DEVICES;
66 |   std::array<bool, DEVICES.size()> autocast_enabled;
67 |   std::array<at::ScalarType, DEVICES.size()> autocast_dtype;
68 |   for (size_t i = 0; i < DEVICES.size(); i++) {
69 |     autocast_enabled[i] = at::autocast::is_autocast_enabled(DEVICES[i]);
70 |     autocast_dtype[i] = at::autocast::get_autocast_dtype(DEVICES[i]);
71 |   }
72 |   bool autocast_cache_enabled = at::autocast::is_autocast_cache_enabled();
73 |   bool tls_grad_enabled = c10::GradMode::is_enabled();
74 |   c10::impl::LocalDispatchKeySet tls_key_set =
75 |       c10::impl::tls_local_dispatch_key_set();
76 | 
77 |   for (const auto thread_id : c10::irange(config.num_calling_threads)) {
78 |     callers.emplace_back([&, thread_id]() {
79 |       // We use conditional variable as a barrier to make sure each thread
80 |       // performs required warmeup iterations before we start measuring
81 |       c10::GradMode::set_enabled(tls_grad_enabled);
82 |       c10::impl::_force_tls_local_dispatch_key_set(tls_key_set);
83 |       for (size_t i = 0; i < DEVICES.size(); i++) {
84 |         at::autocast::set_autocast_enabled(DEVICES[i], autocast_enabled[i]);
85 |         at::autocast::set_autocast_dtype(DEVICES[i], autocast_dtype[i]);
86 |       }
87 |       at::autocast::set_autocast_cache_enabled(autocast_cache_enabled);
88 | 
```
- EN: At the statement level, this block declares abstract hooks that derived backends must provide.
- CN: 在语句层面，这一段声明了需要由派生后端提供的抽象钩子。

### Lines 89-123
```cpp
 89 |       for (const auto j : c10::irange(config.num_warmup_iters)) {
 90 |         (void)j;
 91 |         runOnce(std::move(thread_inputs[thread_id][input_iters[thread_id]]));
 92 |         ++input_iters[thread_id];
 93 |       }
 94 |       {
 95 |         std::unique_lock<std::mutex> lock(m);
 96 |         ++initialized;
 97 |         worker_main_cv.notify_one();
 98 |         // NOLINTNEXTLINE(bugprone-infinite-loop)
 99 |         while (!start) {
100 |           main_worker_cv.wait(lock);
101 |         }
102 |       }
103 |       LOG(INFO) << "Starting forward thread " << thread_id;
104 |       while (num_attempted_iters.fetch_add(1) < config.num_iters) {
105 |         runOnce(std::move(thread_inputs[thread_id][input_iters[thread_id]]));
106 |         ++input_iters[thread_id];
107 |       }
108 | 
109 |       {
110 |         std::unique_lock<std::mutex> lock(m);
111 |         ++finished;
112 |         worker_main_cv.notify_one();
113 |         LOG(INFO) << "Shutting down forward thread " << thread_id
114 |                   << ". Total number of finished threads: " << finished;
115 |       }
116 |     });
117 |   }
118 | 
119 |   using Clock = std::chrono::high_resolution_clock;
120 |   using RecordProfile = torch::autograd::profiler::RecordProfile;
121 |   using TimePoint = std::chrono::time_point<Clock>;
122 |   TimePoint start_time;
123 | 
```
- EN: Implements routines such as `lock` that expose the key API or control flow of this region. At the statement level, this block uses move semantics to avoid unnecessary copies.
- CN: 实现了 `lock` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段使用移动语义来避免不必要的拷贝。

### Lines 124-150
```cpp
124 |   std::unique_ptr<RecordProfile> profiler_guard;
125 |   {
126 |     std::unique_lock<std::mutex> lock(m);
127 |     while (initialized != config.num_calling_threads) {
128 |       worker_main_cv.wait(lock);
129 |     }
130 |     if (!config.profiler_output_path.empty()) {
131 |       LOG(INFO) << "Using Autograd profiler. Trace will be saved to "
132 |                 << config.profiler_output_path;
133 |       profiler_guard =
134 |           std::make_unique<RecordProfile>(config.profiler_output_path);
135 |     }
136 |     LOG(INFO) << "Starting threads";
137 |     start = true;
138 |     start_time = Clock::now();
139 |   }
140 | 
141 |   main_worker_cv.notify_all();
142 |   {
143 |     std::unique_lock<std::mutex> lock(m);
144 |     worker_main_cv.wait(
145 |         lock, [&]() { return finished == config.num_calling_threads; });
146 |   }
147 |   auto end_time = std::chrono::high_resolution_clock::now();
148 |   profiler_guard.reset();
149 |   LOG(INFO) << "Finished benchmark";
150 | 
```
- EN: Implements routines such as `lock` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `lock` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 151-171
```cpp
151 |   BenchmarkExecutionStats stats;
152 |   // NOLINTNEXTLINE(cppcoreguidelines-narrowing-conversions,bugprone-narrowing-conversions)
153 |   float total_time_ms = std::chrono::duration_cast<std::chrono::nanoseconds>(
154 |                             end_time - start_time)
155 |                             .count() /
156 |       1000.0 / 1000.0;
157 |   // We use config.num_iters instead of num_attempted_iters as it is
158 |   // repsesatative of the real work done. Last attempted iteration on each
159 |   // calling threads doesn't represent the real work (i.e. running the model)
160 |   stats.latency_avg_ms =
161 |       // NOLINTNEXTLINE(cppcoreguidelines-narrowing-conversions,bugprone-narrowing-conversions)
162 |       total_time_ms * config.num_calling_threads / config.num_iters;
163 |   stats.num_iters = config.num_iters;
164 | 
165 |   for (auto& t : callers) {
166 |     t.join();
167 |   }
168 |   return stats;
169 | }
170 | 
171 | } // namespace torch::throughput_benchmark::detail
```
- EN: At the statement level, this block returns computed state or forwards values to callers.
- CN: 在语句层面，这一段返回计算状态或把值转交给调用方。

## Key Concepts / 关键概念

- **Primary types / 主要类型**
  - EN: `Input`, `Output`, `Model`.
  - CN: `Input`、`Output`、`Model`。
- **Important routines / 重要例程**
  - EN: `thread_inputs`, `input_iters`, `engine`, `dist`, `lock`.
  - CN: `thread_inputs`、`input_iters`、`engine`、`dist`、`lock`。
- **Subsystem scope / 子系统作用域**
  - EN: The code lives under `torch::throughput_benchmark::detail`, indicating the owning PyTorch subsystem.
  - CN: 代码位于 `torch::throughput_benchmark::detail` 命名空间下，这说明了其所属的 PyTorch 子系统。

## Dependencies / 依赖关系

- Local includes / 本地头文件: `<torch/csrc/autograd/profiler.h>`, `<torch/csrc/jit/python/pybind_utils.h>`, `<torch/csrc/utils/pybind.h>`, `<ATen/Parallel.h>`, `<ATen/autocast_mode.h>`, `<c10/core/GradMode.h>`, `<c10/core/impl/LocalDispatchKeySet.h>`, `<c10/util/irange.h>`
- External includes / 外部头文件: `<random>`, `<thread>`
- Relationship / 关系:
  - EN: The unit relies on neighboring PyTorch headers for subsystem-specific types, dispatcher hooks, and utility helpers. Standard-library or third-party headers provide generic facilities such as containers, threading, math, or backend integration points. Its exported or visible routines are then consumed by callers in the same namespace and by higher-level runtime paths elsewhere in PyTorch.
  - CN: 该编译单元依赖相邻的 PyTorch 头文件，以获得子系统类型、分发器钩子和辅助工具。 标准库或第三方头文件提供了通用能力，例如容器、线程、数学支持或后端集成入口。 随后，这里导出或可见的例程会被同一命名空间中的调用方，以及 PyTorch 其他位置的更高层运行时路径所使用。
