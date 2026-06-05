# thread_pool.cpp — Code Analysis / 代码分析

## Source / 来源
- File / 文件: `torch/csrc/lazy/core/thread_pool.cpp`
- Repository / 仓库: `pytorch`
- Purpose / 作用:
  - EN: Implements the shared Lazy Tensor IR, hashing, shape, and execution support.
  - CN: 实现通用 Lazy Tensor IR、哈希、形状和执行支撑逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-33
```cpp
 1 | #include <torch/csrc/lazy/core/thread_pool.h>
 2 | 
 3 | #include <c10/util/Logging.h>
 4 | #include <c10/util/irange.h>
 5 | #include <c10/util/thread_name.h>
 6 | #include <torch/csrc/lazy/core/config.h>
 7 | #include <torch/csrc/lazy/core/metrics.h>
 8 | 
 9 | #include <condition_variable>
10 | #include <deque>
11 | #include <exception>
12 | #include <mutex>
13 | #include <thread>
14 | 
15 | namespace torch::lazy {
16 | namespace {
17 | 
18 | class ThreadPool {
19 |  public:
20 |   explicit ThreadPool(size_t num_threads) {
21 |     threads_.reserve(num_threads);
22 |     for ([[maybe_unused]] const auto i : c10::irange(num_threads)) {
23 |       threads_.emplace_back([this]() {
24 |         c10::setThreadName("pt_thread_pool");
25 |         Worker();
26 |       });
27 |     }
28 |   }
29 |   ThreadPool(const ThreadPool&) = delete;
30 |   ThreadPool(ThreadPool&&) = delete;
31 |   ThreadPool& operator=(const ThreadPool&) = delete;
32 |   ThreadPool& operator=(ThreadPool&&) = delete;
33 | 
```
- EN: Brings in project headers such as `<torch/csrc/lazy/core/thread_pool.h>`, `<c10/util/Logging.h>`, `<c10/util/irange.h>`, `<c10/util/thread_name.h>` and system or third-party headers such as `<condition_variable>`, `<deque>`, `<exception>`, `<mutex>` so this section can use their types, APIs, or macros. Places the implementation in namespace scopes (`torch::lazy`) so ownership matches the PyTorch subsystem layout. Defines or extends data abstractions such as `ThreadPool` that structure the state handled by this file. Implements routines such as `ThreadPool` that expose the key API or control flow of this region.
- CN: 这里引入了项目头文件，例如 `<torch/csrc/lazy/core/thread_pool.h>`、`<c10/util/Logging.h>`、`<c10/util/irange.h>`、`<c10/util/thread_name.h>`以及系统或第三方头文件，例如 `<condition_variable>`、`<deque>`、`<exception>`、`<mutex>`，使本段代码能够使用相关类型、API 或宏。 把实现放入命名空间作用域（`torch::lazy`），使其归属与 PyTorch 子系统布局保持一致。 定义或扩展了 `ThreadPool` 等数据抽象，用来组织本文件处理的状态。 实现了 `ThreadPool` 等例程，它们构成了这一段的关键 API 或控制流程。

### Lines 34-61
```cpp
34 |   ~ThreadPool() {
35 |     {
36 |       std::lock_guard<std::mutex> lock(mutex_);
37 |       exiting_ = true;
38 |       cv_.notify_all();
39 |     }
40 |     for (auto& thread : threads_) {
41 |       thread.join();
42 |     }
43 |   }
44 | 
45 |   void Schedule(std::function<void()> closure) {
46 |     // If we have more work scheduled than waiting worker threads, just schedule
47 |     // it on a separate thread. This prevents tricky thread-pool-size-deadlocks
48 |     // caused by an undersized thread pool and closures that end up doing sync
49 |     // waits on the pool threads.
50 |     {
51 |       std::unique_lock<std::mutex> lock(mutex_);
52 |       if (work_.size() < waiting_) {
53 |         work_.emplace_back(std::move(closure));
54 |         lock.unlock();
55 |         cv_.notify_one();
56 |         return;
57 |       }
58 |     }
59 |     ScheduleOnThread(std::move(closure));
60 |   }
61 | 
```
- EN: Implements routines such as `lock`, `Schedule` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; uses move semantics to avoid unnecessary copies.
- CN: 实现了 `lock`、`Schedule` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；使用移动语义来避免不必要的拷贝。

### Lines 62-96
```cpp
62 |  private:
63 |   void Worker() {
64 |     while (true) {
65 |       std::function<void()> closure = GetWork();
66 |       if (closure == nullptr) {
67 |         break;
68 |       }
69 |       try {
70 |         closure();
71 |       } catch (const std::exception& ex) {
72 |         TORCH_LAZY_COUNTER("ThreadPoolException", 1);
73 |         LOG(ERROR) << "Exception from running thread pool closure: "
74 |                    << ex.what();
75 |       }
76 |     }
77 |   }
78 | 
79 |   void ScheduleOnThread(std::function<void()> closure) {
80 |     std::thread thread(std::move(closure));
81 |     thread.detach();
82 |   }
83 | 
84 |   std::function<void()> GetWork() {
85 |     std::unique_lock<std::mutex> lock(mutex_);
86 |     ++waiting_;
87 |     cv_.wait(lock, [this] { return exiting_ || !work_.empty(); });
88 |     --waiting_;
89 |     if (work_.empty()) {
90 |       return nullptr;
91 |     }
92 |     std::function<void()> closure(std::move(work_.front()));
93 |     work_.pop_front();
94 |     return closure;
95 |   }
96 | 
```
- EN: Implements routines such as `Worker`, `ScheduleOnThread`, `thread`, `lock` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; uses move semantics to avoid unnecessary copies.
- CN: 实现了 `Worker`、`ScheduleOnThread`、`thread`、`lock` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；使用移动语义来避免不必要的拷贝。

### Lines 97-122
```cpp
 97 |   std::vector<std::thread> threads_;
 98 |   std::mutex mutex_;
 99 |   std::condition_variable cv_;
100 |   bool exiting_ = false;
101 |   std::deque<std::function<void()>> work_;
102 |   size_t waiting_ = 0;
103 | };
104 | 
105 | ThreadPool* GetIoThreadPool() {
106 |   static ThreadPool* pool =
107 |       new ThreadPool(FLAGS_torch_lazy_io_thread_pool_size);
108 |   return pool;
109 | }
110 | 
111 | } // namespace
112 | 
113 | class Completion::Data {
114 |  public:
115 |   void Wait() {
116 |     std::unique_lock<std::mutex> lock(mutex_);
117 |     cv_.wait(lock, [this] { return completed_; });
118 |     if (exptr_ != nullptr) {
119 |       std::rethrow_exception(exptr_);
120 |     }
121 |   }
122 | 
```
- EN: Defines or extends data abstractions such as `Completion` that structure the state handled by this file. Implements routines such as `GetIoThreadPool`, `ThreadPool`, `Wait`, `lock` that expose the key API or control flow of this region. At the statement level, this block declares abstract hooks that derived backends must provide; returns computed state or forwards values to callers; stores long-lived member state for later calls.
- CN: 定义或扩展了 `Completion` 等数据抽象，用来组织本文件处理的状态。 实现了 `GetIoThreadPool`、`ThreadPool`、`Wait`、`lock` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段声明了需要由派生后端提供的抽象钩子；返回计算状态或把值转交给调用方；保存供后续调用使用的长期成员状态。

### Lines 123-161
```cpp
123 |   static std::function<void()> GetCompleter(
124 |       const std::shared_ptr<Data>& data,
125 |       std::function<void()> closure) {
126 |     auto closure_wrapper = [closure = std::move(closure), data]() {
127 |       std::exception_ptr exptr;
128 |       try {
129 |         closure();
130 |       } catch (...) {
131 |         exptr = std::current_exception();
132 |       }
133 |       data->Complete(exptr);
134 |     };
135 |     return closure_wrapper;
136 |   }
137 | 
138 |  private:
139 |   void Complete(std::exception_ptr exptr) {
140 |     std::lock_guard<std::mutex> lock(mutex_);
141 |     exptr_ = std::move(exptr);
142 |     completed_ = true;
143 |     cv_.notify_all();
144 |   }
145 | 
146 |   std::mutex mutex_;
147 |   std::condition_variable cv_;
148 |   bool completed_ = false;
149 |   std::exception_ptr exptr_;
150 | };
151 | 
152 | Completion::Completion(std::shared_ptr<Data> data) : data_(std::move(data)) {}
153 | 
154 | void Completion::Wait() {
155 |   data_->Wait();
156 | }
157 | 
158 | void ScheduleIoClosure(std::function<void()> closure) {
159 |   GetIoThreadPool()->Schedule(std::move(closure));
160 | }
161 | 
```
- EN: Implements routines such as `Complete`, `lock`, `ScheduleIoClosure` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; uses move semantics to avoid unnecessary copies; stores long-lived member state for later calls.
- CN: 实现了 `Complete`、`lock`、`ScheduleIoClosure` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；使用移动语义来避免不必要的拷贝；保存供后续调用使用的长期成员状态。

### Lines 162-169
```cpp
162 | Completion ScheduleIoClosureWithCompletion(std::function<void()> closure) {
163 |   auto data = std::make_shared<Completion::Data>();
164 |   GetIoThreadPool()->Schedule(
165 |       Completion::Data::GetCompleter(data, std::move(closure)));
166 |   return Completion(std::move(data));
167 | }
168 | 
169 | } // namespace torch::lazy
```
- EN: Implements routines such as `ScheduleIoClosureWithCompletion`, `Completion` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; uses move semantics to avoid unnecessary copies.
- CN: 实现了 `ScheduleIoClosureWithCompletion`、`Completion` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；使用移动语义来避免不必要的拷贝。

## Key Concepts / 关键概念

- **Primary types / 主要类型**
  - EN: `ThreadPool`, `Completion`.
  - CN: `ThreadPool`、`Completion`。
- **Important routines / 重要例程**
  - EN: `ThreadPool`, `lock`, `Schedule`, `Worker`, `ScheduleOnThread`, `thread`, `GetIoThreadPool`, `Wait`.
  - CN: `ThreadPool`、`lock`、`Schedule`、`Worker`、`ScheduleOnThread`、`thread`、`GetIoThreadPool`、`Wait`。
- **Subsystem scope / 子系统作用域**
  - EN: The code lives under `torch::lazy`, indicating the owning PyTorch subsystem.
  - CN: 代码位于 `torch::lazy` 命名空间下，这说明了其所属的 PyTorch 子系统。

## Dependencies / 依赖关系

- Local includes / 本地头文件: `<torch/csrc/lazy/core/thread_pool.h>`, `<c10/util/Logging.h>`, `<c10/util/irange.h>`, `<c10/util/thread_name.h>`, `<torch/csrc/lazy/core/config.h>`, `<torch/csrc/lazy/core/metrics.h>`
- External includes / 外部头文件: `<condition_variable>`, `<deque>`, `<exception>`, `<mutex>`, `<thread>`
- Relationship / 关系:
  - EN: The unit relies on neighboring PyTorch headers for subsystem-specific types, dispatcher hooks, and utility helpers. Standard-library or third-party headers provide generic facilities such as containers, threading, math, or backend integration points. Its exported or visible routines are then consumed by callers in the same namespace and by higher-level runtime paths elsewhere in PyTorch.
  - CN: 该编译单元依赖相邻的 PyTorch 头文件，以获得子系统类型、分发器钩子和辅助工具。 标准库或第三方头文件提供了通用能力，例如容器、线程、数学支持或后端集成入口。 随后，这里导出或可见的例程会被同一命名空间中的调用方，以及 PyTorch 其他位置的更高层运行时路径所使用。
