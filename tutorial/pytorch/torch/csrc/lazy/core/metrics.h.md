# metrics.h — Code Analysis / 代码分析

## Source / 来源
- File / 文件: `torch/csrc/lazy/core/metrics.h`
- Repository / 仓库: `pytorch`
- Purpose / 作用:
  - EN: Implements the shared Lazy Tensor IR, hashing, shape, and execution support.
  - CN: 实现通用 Lazy Tensor IR、哈希、形状和执行支撑逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-39
```cpp
 1 | /**
 2 |  * This file is adapted from PyTorch/XLA
 3 |  * https://github.com/pytorch/xla/blob/e0e5f937a0ba8d904f9608137dc8c51ba439df2d/third_party/xla_client/metrics.h
 4 |  */
 5 | 
 6 | #pragma once
 7 | 
 8 | #include <atomic>
 9 | #include <functional>
10 | #include <map>
11 | #include <memory>
12 | #include <mutex>
13 | #include <string>
14 | #include <vector>
15 | 
16 | #include <c10/macros/Export.h>
17 | 
18 | namespace torch::lazy {
19 | 
20 | struct TORCH_API Sample {
21 |   Sample() = default;
22 |   Sample(int64_t timestamp_ns, double value)
23 |       : timestamp_ns(timestamp_ns), value(value) {}
24 | 
25 |   int64_t timestamp_ns = 0;
26 |   double value = 0;
27 | };
28 | 
29 | using MetricReprFn = std::function<std::string(double)>;
30 | 
31 | // Class used to collect time-stamped numeric samples. The samples are stored in
32 | // a circular buffer whose size can be configured at constructor time.
33 | class TORCH_API MetricData {
34 |  public:
35 |   // Creates a new MetricData object with the internal circular buffer storing
36 |   // max_samples samples. The repr_fn argument allow to specify a function which
37 |   // pretty-prints a sample value.
38 |   MetricData(MetricReprFn repr_fn, size_t max_samples);
39 | 
```
- EN: Brings in project headers such as `<c10/macros/Export.h>` and system or third-party headers such as `<atomic>`, `<functional>`, `<map>`, `<memory>` so this section can use their types, APIs, or macros. Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Places the implementation in namespace scopes (`torch::lazy`) so ownership matches the PyTorch subsystem layout. Defines or extends data abstractions such as `MetricData`, `Sample` that structure the state handled by this file.
- CN: 这里引入了项目头文件，例如 `<c10/macros/Export.h>`以及系统或第三方头文件，例如 `<atomic>`、`<functional>`、`<map>`、`<memory>`，使本段代码能够使用相关类型、API 或宏。 使用预处理指令来控制编译开关、平台分支或导出声明。 把实现放入命名空间作用域（`torch::lazy`），使其归属与 PyTorch 子系统布局保持一致。 定义或扩展了 `MetricData`、`Sample` 等数据抽象，用来组织本文件处理的状态。

### Lines 40-76
```cpp
40 |   // Returns the total values of all the samples being posted to this metric.
41 |   double Accumulator() const;
42 | 
43 |   size_t TotalSamples() const;
44 | 
45 |   void AddSample(int64_t timestamp_ns, double value);
46 | 
47 |   // Returns a vector with all the current samples, from the oldest to the
48 |   // newer. If accumulator is not nullptr, it will receive the current value of
49 |   // the metrics' accumulator (the sum of all posted values). If total_samples
50 |   // is not nullptr, it will receive the count of the posted values.
51 |   std::vector<Sample> Samples(double* accumulator, size_t* total_samples) const;
52 | 
53 |   std::string Repr(double value) const {
54 |     return repr_fn_(value);
55 |   }
56 | 
57 |   void Reset();
58 | 
59 |   bool IsValid() const {
60 |     return TotalSamples() > 0;
61 |   }
62 | 
63 |  private:
64 |   mutable std::mutex lock_;
65 |   MetricReprFn repr_fn_;
66 |   size_t count_ = 0;
67 |   std::vector<Sample> samples_;
68 |   double accumulator_ = 0.0;
69 | };
70 | 
71 | // Counters are a very lightweight form of metrics which do not need to track
72 | // sample time.
73 | class TORCH_API CounterData {
74 |  public:
75 |   CounterData() : value_(0) {}
76 | 
```
- EN: Defines or extends data abstractions such as `CounterData` that structure the state handled by this file. Implements routines such as `Accumulator`, `TotalSamples`, `AddSample`, `Samples`, `Repr` that expose the key API or control flow of this region. At the statement level, this block declares abstract hooks that derived backends must provide; returns computed state or forwards values to callers; stores long-lived member state for later calls.
- CN: 定义或扩展了 `CounterData` 等数据抽象，用来组织本文件处理的状态。 实现了 `Accumulator`、`TotalSamples`、`AddSample`、`Samples`、`Repr` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段声明了需要由派生后端提供的抽象钩子；返回计算状态或把值转交给调用方；保存供后续调用使用的长期成员状态。

### Lines 77-114
```cpp
 77 |   void AddValue(int64_t value) {
 78 |     value_ += value;
 79 |   }
 80 | 
 81 |   int64_t Value() const {
 82 |     return value_;
 83 |   }
 84 | 
 85 |   void Reset() {
 86 |     value_ = 0;
 87 |   }
 88 | 
 89 |   bool IsValid() const {
 90 |     return value_ > 0;
 91 |   }
 92 | 
 93 |  private:
 94 |   std::atomic<int64_t> value_;
 95 | };
 96 | 
 97 | class TORCH_API MetricsArena {
 98 |  public:
 99 |   static MetricsArena* Get();
100 | 
101 |   void ResetCounters();
102 |   void ResetMetrics();
103 | 
104 |   // Registers a new metric in the global arena.
105 |   void RegisterMetric(
106 |       const std::string& name,
107 |       MetricReprFn repr_fn,
108 |       size_t max_samples,
109 |       std::shared_ptr<MetricData>* data);
110 | 
111 |   void RegisterCounter(
112 |       const std::string& name,
113 |       std::shared_ptr<CounterData>* data);
114 | 
```
- EN: Defines or extends data abstractions such as `MetricsArena` that structure the state handled by this file. Implements routines such as `AddValue`, `Value`, `Reset`, `IsValid`, `Get` that expose the key API or control flow of this region. At the statement level, this block declares abstract hooks that derived backends must provide; returns computed state or forwards values to callers; stores long-lived member state for later calls.
- CN: 定义或扩展了 `MetricsArena` 等数据抽象，用来组织本文件处理的状态。 实现了 `AddValue`、`Value`、`Reset`、`IsValid`、`Get` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段声明了需要由派生后端提供的抽象钩子；返回计算状态或把值转交给调用方；保存供后续调用使用的长期成员状态。

### Lines 115-143
```cpp
115 |   void ForEachMetric(
116 |       const std::function<void(const std::string&, MetricData*)>& metric_func);
117 | 
118 |   void ForEachCounter(
119 |       const std::function<void(const std::string&, CounterData*)>&
120 |           counter_func);
121 | 
122 |   std::vector<std::string> GetMetricNames();
123 | 
124 |   MetricData* GetMetric(const std::string& name);
125 | 
126 |   std::vector<std::string> GetCounterNames();
127 | 
128 |   CounterData* GetCounter(const std::string& name);
129 | 
130 |  private:
131 |   std::mutex lock_;
132 |   std::map<std::string, std::shared_ptr<MetricData>> metrics_;
133 |   std::map<std::string, std::shared_ptr<CounterData>> counters_;
134 | };
135 | 
136 | // Emits the value in a to_string() conversion.
137 | TORCH_API std::string MetricFnValue(double value);
138 | // Emits the value in a humanized bytes representation.
139 | TORCH_API std::string MetricFnBytes(double value);
140 | // Emits the value in a humanized time representation. The value is expressed in
141 | // nanoseconds EPOCH time.
142 | TORCH_API std::string MetricFnTime(double value);
143 | 
```
- EN: Declares routines such as `ForEachMetric`, `ForEachCounter`, `GetMetricNames`, `GetMetric`, `GetCounterNames` that expose the key API or control flow of this region. At the statement level, this block stores long-lived member state for later calls.
- CN: 声明了 `ForEachMetric`、`ForEachCounter`、`GetMetricNames`、`GetMetric`、`GetCounterNames` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段保存供后续调用使用的长期成员状态。

### Lines 144-183
```cpp
144 | // The typical use of a Metric is one in which it gets created either in a
145 | // global scope context:
146 | //   static Metric* metric = new Metric("RpcCount");
147 | // Or within a function scope:
148 | //   void MyFunction(...) {
149 | //     static Metric* metric = new Metric("RpcCount");
150 | //     ...
151 | //     metric->AddSample(ts_nanos, some_value);
152 | //   }
153 | class TORCH_API Metric {
154 |  public:
155 |   explicit Metric(
156 |       std::string name,
157 |       MetricReprFn repr_fn = MetricFnValue,
158 |       size_t max_samples = 0);
159 | 
160 |   const std::string& Name() const {
161 |     return name_;
162 |   }
163 | 
164 |   double Accumulator() const;
165 | 
166 |   void AddSample(int64_t timestamp_ns, double value);
167 | 
168 |   void AddSample(double value);
169 | 
170 |   std::vector<Sample> Samples(double* accumulator, size_t* total_samples) const;
171 | 
172 |   std::string Repr(double value) const;
173 | 
174 |  private:
175 |   MetricData* GetData() const;
176 | 
177 |   std::string name_;
178 |   MetricReprFn repr_fn_;
179 |   size_t max_samples_;
180 |   mutable std::shared_ptr<MetricData> data_ptr_;
181 |   mutable std::atomic<MetricData*> data_;
182 | };
183 | 
```
- EN: Defines or extends data abstractions such as `Metric` that structure the state handled by this file. Implements routines such as `Metric`, `Name`, `Accumulator`, `AddSample`, `Samples` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; stores long-lived member state for later calls.
- CN: 定义或扩展了 `Metric` 等数据抽象，用来组织本文件处理的状态。 实现了 `Metric`、`Name`、`Accumulator`、`AddSample`、`Samples` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；保存供后续调用使用的长期成员状态。

### Lines 184-218
```cpp
184 | // A Counter is a lightweight form of metric which tracks an integer value which
185 | // can increase or decrease.
186 | // A typical use is as:
187 | //   static Counter* counter = new Counter("MyCounter");
188 | //   ...
189 | //   counter->AddValue(+1);
190 | class TORCH_API Counter {
191 |  public:
192 |   explicit Counter(std::string name);
193 | 
194 |   void AddValue(int64_t value) {
195 |     GetData()->AddValue(value);
196 |   }
197 | 
198 |   int64_t Value() const {
199 |     return GetData()->Value();
200 |   }
201 | 
202 |  private:
203 |   CounterData* GetData() const;
204 | 
205 |   std::string name_;
206 |   mutable std::shared_ptr<CounterData> data_ptr_;
207 |   mutable std::atomic<CounterData*> data_;
208 | };
209 | 
210 | #define TORCH_LAZY_COUNTER(name, value)        \
211 |   do {                                         \
212 |     static ::torch::lazy::Counter* __counter = \
213 |         new ::torch::lazy::Counter(name);      \
214 |     __counter->AddValue(value);                \
215 |   } while (0)
216 | 
217 | #define TORCH_LAZY_FN_COUNTER(ns) TORCH_LAZY_COUNTER(c10::str(ns, __func__), 1)
218 | 
```
- EN: Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Defines or extends data abstractions such as `Counter` that structure the state handled by this file. Implements routines such as `Counter`, `AddValue`, `Value`, `GetData` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; stores long-lived member state for later calls.
- CN: 使用预处理指令来控制编译开关、平台分支或导出声明。 定义或扩展了 `Counter` 等数据抽象，用来组织本文件处理的状态。 实现了 `Counter`、`AddValue`、`Value`、`GetData` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；保存供后续调用使用的长期成员状态。

### Lines 219-254
```cpp
219 | #define TORCH_LAZY_VALUE_METRIC(name, value)                         \
220 |   do {                                                               \
221 |     static ::torch::lazy::Metric* __metric =                         \
222 |         new ::torch::lazy::Metric(name, torch::lazy::MetricFnValue); \
223 |     __metric->AddSample(value);                                      \
224 |   } while (0)
225 | 
226 | // Creates a report with the current metrics statistics.
227 | TORCH_API std::string CreateMetricReport();
228 | 
229 | // Creates a report with the selected metrics statistics.
230 | TORCH_API std::string CreateMetricReport(
231 |     const std::vector<std::string>& counter_names,
232 |     const std::vector<std::string>& metric_names);
233 | 
234 | // Returns the currently registered metric names. Note that the list can grow
235 | // since metrics are usually function initialized (they are static function
236 | // variables).
237 | TORCH_API std::vector<std::string> GetMetricNames();
238 | 
239 | // Retrieves the metric data of a given metric, or nullptr if such metric does
240 | // not exist.
241 | TORCH_API MetricData* GetMetric(const std::string& name);
242 | 
243 | // Returns the currently registered counter names. Note that the list can grow
244 | // since counters are usually function initialized (they are static function
245 | // variables).
246 | TORCH_API std::vector<std::string> GetCounterNames();
247 | 
248 | // Retrieves the counter data of a given counter, or nullptr if such counter
249 | // does not exist.
250 | TORCH_API CounterData* GetCounter(const std::string& name);
251 | 
252 | // Retrieves the current EPOCH time in nanoseconds.
253 | TORCH_API int64_t NowNs();
254 | 
```
- EN: Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Implements routines such as `CreateMetricReport`, `GetMetricNames`, `GetMetric`, `GetCounterNames`, `GetCounter` that expose the key API or control flow of this region.
- CN: 使用预处理指令来控制编译开关、平台分支或导出声明。 实现了 `CreateMetricReport`、`GetMetricNames`、`GetMetric`、`GetCounterNames`、`GetCounter` 等例程，它们构成了这一段的关键 API 或控制流程。

### Lines 255-287
```cpp
255 | // Scope based utility class TORCH_API to measure the time the code takes within
256 | // a given C++ scope.
257 | class TORCH_API TimedSection {
258 |  public:
259 |   explicit TimedSection(Metric* metric) : metric_(metric), start_(NowNs()) {}
260 | 
261 |   TimedSection(TimedSection&& other) = delete;
262 |   TimedSection(const TimedSection&) = delete;
263 |   TimedSection& operator=(const TimedSection&) = delete;
264 |   TimedSection& operator=(TimedSection&&) = delete;
265 |   ~TimedSection() {
266 |     int64_t now = NowNs();
267 |     metric_->AddSample(now, static_cast<double>(now - start_));
268 |   }
269 | 
270 |   double Elapsed() const {
271 |     return 1e-9 * static_cast<double>(NowNs() - start_);
272 |   }
273 | 
274 |  private:
275 |   Metric* metric_;
276 |   int64_t start_;
277 | };
278 | 
279 | #define TORCH_LAZY_TIMED(name)                                  \
280 |   static torch::lazy::Metric* timed_metric =                    \
281 |       new torch::lazy::Metric(name, torch::lazy::MetricFnTime); \
282 |   torch::lazy::TimedSection timed_section(timed_metric)
283 | 
284 | #define TORCH_LAZY_FN_COUNTER_TIMED_TRACING(ns) \
285 |   TORCH_LAZY_FN_COUNTER(ns);                    \
286 |   TORCH_LAZY_TIMED("LazyTracing")
287 | 
```
- EN: Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Defines or extends data abstractions such as `to`, `TimedSection` that structure the state handled by this file. Implements routines such as `TimedSection`, `Elapsed`, `timed_section` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; stores long-lived member state for later calls.
- CN: 使用预处理指令来控制编译开关、平台分支或导出声明。 定义或扩展了 `to`、`TimedSection` 等数据抽象，用来组织本文件处理的状态。 实现了 `TimedSection`、`Elapsed`、`timed_section` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；保存供后续调用使用的长期成员状态。

### Lines 288-288
```cpp
288 | } // namespace torch::lazy
```
- EN: Continues the file's main role: Implements the shared Lazy Tensor IR, hashing, shape, and execution support.
- CN: 继续承担本文件的主要职责：实现通用 Lazy Tensor IR、哈希、形状和执行支撑逻辑。

## Key Concepts / 关键概念

- **Primary types / 主要类型**
  - EN: `MetricData`, `CounterData`, `MetricsArena`, `Metric`, `Counter`, `to`.
  - CN: `MetricData`、`CounterData`、`MetricsArena`、`Metric`、`Counter`、`to`。
- **Important routines / 重要例程**
  - EN: `Accumulator`, `TotalSamples`, `AddSample`, `Samples`, `Repr`, `repr_fn_`, `Reset`, `IsValid`.
  - CN: `Accumulator`、`TotalSamples`、`AddSample`、`Samples`、`Repr`、`repr_fn_`、`Reset`、`IsValid`。
- **Subsystem scope / 子系统作用域**
  - EN: The code lives under `torch::lazy`, indicating the owning PyTorch subsystem.
  - CN: 代码位于 `torch::lazy` 命名空间下，这说明了其所属的 PyTorch 子系统。

## Dependencies / 依赖关系

- Local includes / 本地头文件: `<c10/macros/Export.h>`
- External includes / 外部头文件: `<atomic>`, `<functional>`, `<map>`, `<memory>`, `<mutex>`, `<string>`, `<vector>`
- Relationship / 关系:
  - EN: The unit relies on neighboring PyTorch headers for subsystem-specific types, dispatcher hooks, and utility helpers. Standard-library or third-party headers provide generic facilities such as containers, threading, math, or backend integration points. Its exported or visible routines are then consumed by callers in the same namespace and by higher-level runtime paths elsewhere in PyTorch.
  - CN: 该编译单元依赖相邻的 PyTorch 头文件，以获得子系统类型、分发器钩子和辅助工具。 标准库或第三方头文件提供了通用能力，例如容器、线程、数学支持或后端集成入口。 随后，这里导出或可见的例程会被同一命名空间中的调用方，以及 PyTorch 其他位置的更高层运行时路径所使用。
