# throughput_benchmark.cpp — Code Analysis / 代码分析

## Source / 来源
- File / 文件: `torch/csrc/utils/throughput_benchmark.cpp`
- Repository / 仓库: `pytorch`
- Purpose / 作用:
  - EN: Provides miscellaneous C++/Python interop, argument parsing, and runtime utility helpers.
  - CN: 提供杂项的 C++/Python 互操作、参数解析和运行时工具辅助函数。

## Line-by-Line Analysis / 逐行分析

### Lines 1-23
```cpp
 1 | #include <torch/csrc/utils/throughput_benchmark.h>
 2 | 
 3 | #include <torch/csrc/jit/python/pybind_utils.h>
 4 | 
 5 | namespace torch::throughput_benchmark {
 6 | 
 7 | std::ostream& operator<<(
 8 |     std::ostream& os,
 9 |     const BenchmarkExecutionStats& value) {
10 |   return os << "Average latency / iter (ms): " << value.latency_avg_ms
11 |             << "\n Total number of iters: " << value.num_iters;
12 | }
13 | 
14 | void ThroughputBenchmark::addInput(py::args args, py::kwargs kwargs) {
15 |   CHECK(script_module_.initialized() ^ module_.initialized());
16 |   if (script_module_.initialized()) {
17 |     script_module_.addInput(std::move(args), std::move(kwargs));
18 |   } else {
19 |     CHECK(module_.initialized());
20 |     module_.addInput(std::move(args), std::move(kwargs));
21 |   }
22 | }
23 | 
```
- EN: Brings in project headers such as `<torch/csrc/utils/throughput_benchmark.h>`, `<torch/csrc/jit/python/pybind_utils.h>` so this section can use their types, APIs, or macros. Places the implementation in namespace scopes (`torch::throughput_benchmark`) so ownership matches the PyTorch subsystem layout. At the statement level, this block returns computed state or forwards values to callers; uses move semantics to avoid unnecessary copies.
- CN: 这里引入了项目头文件，例如 `<torch/csrc/utils/throughput_benchmark.h>`、`<torch/csrc/jit/python/pybind_utils.h>`，使本段代码能够使用相关类型、API 或宏。 把实现放入命名空间作用域（`torch::throughput_benchmark`），使其归属与 PyTorch 子系统布局保持一致。 在语句层面，这一段返回计算状态或把值转交给调用方；使用移动语义来避免不必要的拷贝。

### Lines 24-46
```cpp
24 | py::object ThroughputBenchmark::runOnce(
25 |     const py::args& args,
26 |     const py::kwargs& kwargs) {
27 |   CHECK(script_module_.initialized() ^ module_.initialized());
28 |   if (script_module_.initialized()) {
29 |     c10::IValue result;
30 |     {
31 |       pybind11::gil_scoped_release no_gil_guard;
32 |       result = script_module_.runOnce(args, kwargs);
33 |     }
34 |     return jit::toPyObject(std::move(result));
35 |   } else {
36 |     CHECK(module_.initialized());
37 |     return module_.runOnce(args, kwargs);
38 |   }
39 | }
40 | 
41 | ThroughputBenchmark::ThroughputBenchmark(const jit::Module& script_module)
42 |     : script_module_(script_module) {}
43 | 
44 | ThroughputBenchmark::ThroughputBenchmark(py::object module)
45 |     : module_(std::move(module)) {}
46 | 
```
- EN: At the statement level, this block returns computed state or forwards values to callers; uses move semantics to avoid unnecessary copies.
- CN: 在语句层面，这一段返回计算状态或把值转交给调用方；使用移动语义来避免不必要的拷贝。

### Lines 47-73
```cpp
47 | BenchmarkExecutionStats ThroughputBenchmark::benchmark(
48 |     const BenchmarkConfig& config) const {
49 |   CHECK(script_module_.initialized() ^ module_.initialized());
50 |   // Main benchmark thread doesn't hold the GIL after scheduling worker threads
51 |   // But for now we don't release it as we will be implicitly manipulating with
52 |   // py::object ref. counts in the case of nn.Module benchmarking.
53 |   if (script_module_.initialized()) {
54 |     return script_module_.benchmark(config);
55 |   } else {
56 |     CHECK(module_.initialized());
57 |     TORCH_WARN(
58 |         "Starting benchmark on an nn.Module. This can be slow due "
59 |         "to Python GIL.For proper inference simulation you might want to switch to "
60 |         "a ScriptModule instead");
61 |     return module_.benchmark(config);
62 |   }
63 | }
64 | 
65 | namespace detail {
66 | 
67 | template <>
68 | void ScriptModuleBenchmark::runOnce(ScriptModuleInput&& input) const {
69 |   CHECK(initialized_);
70 |   // TODO: provide guarantees that compiler won't optimize this out
71 |   model_.get_method("forward").function()(std::move(input));
72 | }
73 | 
```
- EN: Places the implementation in namespace scopes (`detail`) so ownership matches the PyTorch subsystem layout. At the statement level, this block returns computed state or forwards values to callers; uses move semantics to avoid unnecessary copies.
- CN: 把实现放入命名空间作用域（`detail`），使其归属与 PyTorch 子系统布局保持一致。 在语句层面，这一段返回计算状态或把值转交给调用方；使用移动语义来避免不必要的拷贝。

### Lines 74-101
```cpp
 74 | template <>
 75 | ScriptModuleOutput ScriptModuleBenchmark::runOnce(
 76 |     const py::args& args,
 77 |     const py::kwargs& kwargs) const {
 78 |   CHECK(initialized_);
 79 |   auto& function = model_.get_method("forward").function();
 80 |   ScriptModuleInput stack = jit::createStackForSchema(
 81 |       function.getSchema(), args, kwargs, model_._ivalue());
 82 |   return function(std::move(stack));
 83 | }
 84 | 
 85 | template <>
 86 | // NOLINTNEXTLINE(*-rvalue-reference-param-not-moved)
 87 | void ModuleBenchmark::runOnce(ModuleInput&& input) const {
 88 |   CHECK(initialized_);
 89 |   pybind11::gil_scoped_acquire gil_guard;
 90 |   model_(*input.args, **input.kwargs);
 91 | }
 92 | 
 93 | template <>
 94 | ModuleOutput ModuleBenchmark::runOnce(
 95 |     const py::args& args,
 96 |     const py::kwargs& kwargs) const {
 97 |   CHECK(initialized_);
 98 |   pybind11::gil_scoped_acquire gil_guard;
 99 |   return model_(*args, **kwargs);
100 | }
101 | 
```
- EN: Implements routines such as `function`, `model_` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; uses move semantics to avoid unnecessary copies.
- CN: 实现了 `function`、`model_` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；使用移动语义来避免不必要的拷贝。

### Lines 102-131
```cpp
102 | template <>
103 | // NOLINTNEXTLINE(*-rvalue-reference-param-not-moved)
104 | void ScriptModuleBenchmark::addInput(py::args&& args, py::kwargs&& kwargs) {
105 |   jit::Stack stack = jit::createStackForSchema(
106 |       model_.get_method("forward").function().getSchema(),
107 |       args,
108 |       kwargs,
109 |       model_._ivalue());
110 |   inputs_.emplace_back(std::move(stack));
111 | }
112 | 
113 | template <>
114 | void ScriptModuleBenchmark::addInput(ScriptModuleInput&& input) {
115 |   input.insert(input.begin(), model_._ivalue());
116 |   inputs_.emplace_back(std::move(input));
117 | }
118 | 
119 | template <>
120 | void ModuleBenchmark::addInput(py::args&& args, py::kwargs&& kwargs) {
121 |   inputs_.emplace_back(std::move(args), std::move(kwargs));
122 | }
123 | 
124 | template <>
125 | ModuleInput cloneInput<ModuleInput>(const ModuleInput& input) {
126 |   pybind11::gil_scoped_acquire gil_guard;
127 |   py::args args = input.args;
128 |   py::kwargs kwargs = input.kwargs;
129 |   return {std::move(args), std::move(kwargs)};
130 | }
131 | 
```
- EN: At the statement level, this block returns computed state or forwards values to callers; uses move semantics to avoid unnecessary copies; builds container state that later execution depends on.
- CN: 在语句层面，这一段返回计算状态或把值转交给调用方；使用移动语义来避免不必要的拷贝；构建后续执行依赖的容器状态。

### Lines 132-140
```cpp
132 | template <>
133 | ScriptModuleInput cloneInput<ScriptModuleInput>(
134 |     const ScriptModuleInput& input) {
135 |   return input;
136 | }
137 | 
138 | } // namespace detail
139 | 
140 | } // namespace torch::throughput_benchmark
```
- EN: At the statement level, this block returns computed state or forwards values to callers.
- CN: 在语句层面，这一段返回计算状态或把值转交给调用方。

## Key Concepts / 关键概念

- **Important routines / 重要例程**
  - EN: `function`, `model_`.
  - CN: `function`、`model_`。
- **Subsystem scope / 子系统作用域**
  - EN: The code lives under `torch::throughput_benchmark`, `detail`, indicating the owning PyTorch subsystem.
  - CN: 代码位于 `torch::throughput_benchmark`、`detail` 命名空间下，这说明了其所属的 PyTorch 子系统。

## Dependencies / 依赖关系

- Local includes / 本地头文件: `<torch/csrc/utils/throughput_benchmark.h>`, `<torch/csrc/jit/python/pybind_utils.h>`
- External includes / 外部头文件: None / 无
- Relationship / 关系:
  - EN: The unit relies on neighboring PyTorch headers for subsystem-specific types, dispatcher hooks, and utility helpers. Its exported or visible routines are then consumed by callers in the same namespace and by higher-level runtime paths elsewhere in PyTorch.
  - CN: 该编译单元依赖相邻的 PyTorch 头文件，以获得子系统类型、分发器钩子和辅助工具。 随后，这里导出或可见的例程会被同一命名空间中的调用方，以及 PyTorch 其他位置的更高层运行时路径所使用。
