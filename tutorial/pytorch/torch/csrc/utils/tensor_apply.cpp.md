# tensor_apply.cpp — Code Analysis / 代码分析

## Source / 来源
- File / 文件: `torch/csrc/utils/tensor_apply.cpp`
- Repository / 仓库: `pytorch`
- Purpose / 作用:
  - EN: Provides miscellaneous C++/Python interop, argument parsing, and runtime utility helpers.
  - CN: 提供杂项的 C++/Python 互操作、参数解析和运行时工具辅助函数。

## Line-by-Line Analysis / 逐行分析

### Lines 1-27
```cpp
 1 | #include <torch/csrc/utils/tensor_apply.h>
 2 | 
 3 | #include <ATen/ExpandUtils.h>
 4 | #include <c10/util/irange.h>
 5 | 
 6 | #include <torch/csrc/Exceptions.h>
 7 | #include <torch/csrc/utils/python_scalars.h>
 8 | 
 9 | using namespace at;
10 | 
11 | namespace torch::utils {
12 | 
13 | struct StridedData {
14 |   StridedData(const Tensor& tensor)
15 |       : data(tensor.data_ptr()),
16 |         strides(tensor.strides()),
17 |         elementSize(tensor.element_size()) {}
18 | 
19 |   void* data;
20 |   IntArrayRef strides;
21 |   int64_t elementSize;
22 | 
23 |   void step(int dim) {
24 |     data = (char*)data + (strides[dim] * elementSize);
25 |   }
26 | };
27 | 
```
- EN: Brings in project headers such as `<torch/csrc/utils/tensor_apply.h>`, `<ATen/ExpandUtils.h>`, `<c10/util/irange.h>`, `<torch/csrc/Exceptions.h>` so this section can use their types, APIs, or macros. Places the implementation in namespace scopes (`torch::utils`) so ownership matches the PyTorch subsystem layout. Defines or extends data abstractions such as `StridedData` that structure the state handled by this file. Implements routines such as `step` that expose the key API or control flow of this region.
- CN: 这里引入了项目头文件，例如 `<torch/csrc/utils/tensor_apply.h>`、`<ATen/ExpandUtils.h>`、`<c10/util/irange.h>`、`<torch/csrc/Exceptions.h>`，使本段代码能够使用相关类型、API 或宏。 把实现放入命名空间作用域（`torch::utils`），使其归属与 PyTorch 子系统布局保持一致。 定义或扩展了 `StridedData` 等数据抽象，用来组织本文件处理的状态。 实现了 `step` 等例程，它们构成了这一段的关键 API 或控制流程。

### Lines 28-52
```cpp
28 | template <size_t N>
29 | static void recursive_apply(
30 |     IntArrayRef sizes,
31 |     ScalarType scalarType,
32 |     int64_t dim,
33 |     PyObject* fn,
34 |     std::array<StridedData, N> strided_data) {
35 |   int64_t ndim = static_cast<int64_t>(sizes.size());
36 |   if (dim == ndim) {
37 |     auto args = THPObjectPtr(PyTuple_New(N));
38 |     if (!args)
39 |       throw python_error();
40 |     for (const auto i : c10::irange(N)) {
41 |       PyObject* arg = load_scalar(strided_data[i].data, scalarType);
42 |       if (!arg)
43 |         throw python_error();
44 |       PyTuple_SET_ITEM(args.get(), i, arg);
45 |     }
46 |     auto ret = THPObjectPtr(PyObject_CallObject(fn, args.get()));
47 |     if (!ret)
48 |       throw python_error();
49 |     store_scalar(strided_data[0].data, scalarType, ret.get());
50 |     return;
51 |   }
52 | 
```
- EN: Implements routines such as `recursive_apply`, `python_error` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `recursive_apply`、`python_error` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 53-72
```cpp
53 |   auto n = sizes[dim];
54 |   for ([[maybe_unused]] const auto i : c10::irange(n)) {
55 |     recursive_apply(sizes, scalarType, dim + 1, fn, strided_data);
56 |     for (auto& td : strided_data) {
57 |       td.step(dim);
58 |     }
59 |   }
60 | }
61 | 
62 | const Tensor& apply_(const Tensor& self, PyObject* fn) {
63 |   if (self.is_meta()) {
64 |     return self; // Just skip
65 |   }
66 |   TORCH_CHECK_TYPE(
67 |       self.device().is_cpu(), "apply_ is only implemented on CPU tensors");
68 |   auto scalarType = self.scalar_type();
69 |   recursive_apply<1>(self.sizes(), scalarType, 0, fn, {{self}});
70 |   return self;
71 | }
72 | 
```
- EN: Implements routines such as `apply_` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 实现了 `apply_` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

### Lines 73-91
```cpp
73 | const Tensor& map_(const Tensor& self, const Tensor& other_, PyObject* fn) {
74 |   TORCH_CHECK_TYPE(
75 |       other_.options().type_equal(self.options()),
76 |       "map_: expected ",
77 |       self.toString(),
78 |       " for 'other' (got ",
79 |       other_.toString(),
80 |       ")");
81 |   if (self.is_meta()) {
82 |     return self; // Just skip
83 |   }
84 |   TORCH_CHECK_TYPE(
85 |       self.device().is_cpu(), "map_ is only implemented on CPU tensors");
86 |   c10::MaybeOwned<Tensor> other = expand_inplace(self, other_, "map_");
87 |   auto scalarType = self.scalar_type();
88 |   recursive_apply<2>(self.sizes(), scalarType, 0, fn, {{self, *other}});
89 |   return self;
90 | }
91 | 
```
- EN: Implements routines such as `map_` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 实现了 `map_` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

### Lines 92-115
```cpp
 92 | const Tensor& map2_(
 93 |     const Tensor& self,
 94 |     const Tensor& x_,
 95 |     const Tensor& y_,
 96 |     PyObject* fn) {
 97 |   TORCH_CHECK_TYPE(
 98 |       x_.options().type_equal(self.options()),
 99 |       "map2_: expected ",
100 |       self.toString(),
101 |       " for argument 'x' (got ",
102 |       x_.toString(),
103 |       ")");
104 |   TORCH_CHECK_TYPE(
105 |       y_.options().type_equal(self.options()),
106 |       "map2_: expected ",
107 |       self.toString(),
108 |       " for argument 'y' (got ",
109 |       y_.toString(),
110 |       ")");
111 |   if (self.is_meta()) {
112 |     return self; // Just skip
113 |   }
114 |   TORCH_CHECK_TYPE(
115 |       (self.device().is_cpu() && x_.device().is_cpu() && y_.device().is_cpu()),
```
- EN: Implements routines such as `map2_` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 实现了 `map2_` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

### Lines 116-128
```cpp
116 |       "map2_ is only implemented on CPU tensors");
117 |   auto others = expand_inplace(self, x_, y_, "map2_");
118 |   auto scalarType = self.scalar_type();
119 |   recursive_apply<3>(
120 |       self.sizes(),
121 |       scalarType,
122 |       0,
123 |       fn,
124 |       {{self, *std::get<0>(others), *std::get<1>(others)}});
125 |   return self;
126 | }
127 | 
128 | } // namespace torch::utils
```
- EN: At the statement level, this block returns computed state or forwards values to callers.
- CN: 在语句层面，这一段返回计算状态或把值转交给调用方。

## Key Concepts / 关键概念

- **Primary types / 主要类型**
  - EN: `StridedData`.
  - CN: `StridedData`。
- **Important routines / 重要例程**
  - EN: `step`, `recursive_apply`, `python_error`, `apply_`, `map_`, `map2_`.
  - CN: `step`、`recursive_apply`、`python_error`、`apply_`、`map_`、`map2_`。
- **Subsystem scope / 子系统作用域**
  - EN: The code lives under `torch::utils`, indicating the owning PyTorch subsystem.
  - CN: 代码位于 `torch::utils` 命名空间下，这说明了其所属的 PyTorch 子系统。

## Dependencies / 依赖关系

- Local includes / 本地头文件: `<torch/csrc/utils/tensor_apply.h>`, `<ATen/ExpandUtils.h>`, `<c10/util/irange.h>`, `<torch/csrc/Exceptions.h>`, `<torch/csrc/utils/python_scalars.h>`
- External includes / 外部头文件: None / 无
- Relationship / 关系:
  - EN: The unit relies on neighboring PyTorch headers for subsystem-specific types, dispatcher hooks, and utility helpers. Its exported or visible routines are then consumed by callers in the same namespace and by higher-level runtime paths elsewhere in PyTorch.
  - CN: 该编译单元依赖相邻的 PyTorch 头文件，以获得子系统类型、分发器钩子和辅助工具。 随后，这里导出或可见的例程会被同一命名空间中的调用方，以及 PyTorch 其他位置的更高层运行时路径所使用。
