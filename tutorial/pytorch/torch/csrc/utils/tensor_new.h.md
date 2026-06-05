# tensor_new.h — Code Analysis / 代码分析

## Source / 来源
- File / 文件: `torch/csrc/utils/tensor_new.h`
- Repository / 仓库: `pytorch`
- Purpose / 作用:
  - EN: Provides miscellaneous C++/Python interop, argument parsing, and runtime utility helpers.
  - CN: 提供杂项的 C++/Python 互操作、参数解析和运行时工具辅助函数。

## Line-by-Line Analysis / 逐行分析

### Lines 1-31
```cpp
 1 | #pragma once
 2 | 
 3 | #include <torch/csrc/python_headers.h>
 4 | #include <torch/csrc/utils/python_arg_parser.h>
 5 | 
 6 | #include <ATen/core/Tensor.h>
 7 | 
 8 | namespace torch::utils {
 9 | 
10 | // NOTE: [torch.tensor, lift_fresh, and device movement]
11 | //
12 | // The `only_lift_cpu_tensors` flag controls what happens on torch.tensor([1, 2,
13 | // 3], device="cuda") (or any non-CPU devices).
14 | //
15 | // If false (default):
16 | // - the data gets moved into a CPU Tensor
17 | // - then, it gets moved to cuda (via .to)
18 | // - finally, we call lift_fresh() on it.
19 | // Steps 1 and 2 happen with all modes disabled.
20 | //
21 | // If true:
22 | // - the data gets moved into a CPU Tensor (with correct dtype)
23 | // - we call lift_fresh() on it
24 | // - finally, we move it to cuda (via .to)
25 | // Step 1 happens with all modes disabled.
26 | //
27 | // `only_lift_cpu_tensors=true` is useful to prevent CUDA initialization under
28 | // FakeTensorMode because it avoids moving concrete data to CUDA.
29 | TORCH_API bool only_lift_cpu_tensors();
30 | TORCH_API void set_only_lift_cpu_tensors(bool value);
31 | 
```
- EN: Brings in project headers such as `<torch/csrc/python_headers.h>`, `<torch/csrc/utils/python_arg_parser.h>`, `<ATen/core/Tensor.h>` so this section can use their types, APIs, or macros. Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Places the implementation in namespace scopes (`torch::utils`) so ownership matches the PyTorch subsystem layout. Implements routines such as `only_lift_cpu_tensors`, `set_only_lift_cpu_tensors` that expose the key API or control flow of this region.
- CN: 这里引入了项目头文件，例如 `<torch/csrc/python_headers.h>`、`<torch/csrc/utils/python_arg_parser.h>`、`<ATen/core/Tensor.h>`，使本段代码能够使用相关类型、API 或宏。 使用预处理指令来控制编译开关、平台分支或导出声明。 把实现放入命名空间作用域（`torch::utils`），使其归属与 PyTorch 子系统布局保持一致。 实现了 `only_lift_cpu_tensors`、`set_only_lift_cpu_tensors` 等例程，它们构成了这一段的关键 API 或控制流程。

### Lines 32-57
```cpp
32 | at::Tensor base_tensor_ctor(PyObject* args, PyObject* kwargs);
33 | TORCH_PYTHON_API at::Tensor legacy_tensor_ctor(
34 |     c10::DispatchKey dispatch_key,
35 |     at::ScalarType scalar_type,
36 |     PyObject* args,
37 |     PyObject* kwargs);
38 | at::Tensor legacy_tensor_new(
39 |     c10::DispatchKey dispatch_key,
40 |     at::ScalarType scalar_type,
41 |     PyObject* args,
42 |     PyObject* kwargs);
43 | at::Tensor indexing_tensor_from_data(
44 |     c10::TensorOptions options,
45 |     at::ScalarType scalar_type,
46 |     std::optional<at::Device> device,
47 |     PyObject* data);
48 | at::Tensor sparse_coo_tensor_ctor(
49 |     c10::DispatchKey dispatch_key,
50 |     at::ScalarType scalar_type,
51 |     PythonArgs& r);
52 | void _validate_sparse_coo_tensor_args(
53 |     c10::DispatchKey dispatch_key,
54 |     at::ScalarType scalar_type,
55 |     PyObject* args,
56 |     PyObject* kwargs);
57 | 
```
- EN: Declares routines such as `base_tensor_ctor`, `legacy_tensor_ctor`, `legacy_tensor_new`, `indexing_tensor_from_data`, `sparse_coo_tensor_ctor` that expose the key API or control flow of this region.
- CN: 声明了 `base_tensor_ctor`、`legacy_tensor_ctor`、`legacy_tensor_new`、`indexing_tensor_from_data`、`sparse_coo_tensor_ctor` 等例程，它们构成了这一段的关键 API 或控制流程。

### Lines 58-78
```cpp
58 | at::Tensor sparse_compressed_tensor_ctor(
59 |     c10::DispatchKey dispatch_key,
60 |     at::ScalarType scalar_type,
61 |     PythonArgs& r);
62 | at::Tensor sparse_csr_tensor_ctor(
63 |     c10::DispatchKey dispatch_key,
64 |     at::ScalarType scalar_type,
65 |     PythonArgs& r);
66 | at::Tensor sparse_csc_tensor_ctor(
67 |     c10::DispatchKey dispatch_key,
68 |     at::ScalarType scalar_type,
69 |     PythonArgs& r);
70 | at::Tensor sparse_bsr_tensor_ctor(
71 |     c10::DispatchKey dispatch_key,
72 |     at::ScalarType scalar_type,
73 |     PythonArgs& r);
74 | at::Tensor sparse_bsc_tensor_ctor(
75 |     c10::DispatchKey dispatch_key,
76 |     at::ScalarType scalar_type,
77 |     PythonArgs& r);
78 | 
```
- EN: Declares routines such as `sparse_compressed_tensor_ctor`, `sparse_csr_tensor_ctor`, `sparse_csc_tensor_ctor`, `sparse_bsr_tensor_ctor`, `sparse_bsc_tensor_ctor` that expose the key API or control flow of this region.
- CN: 声明了 `sparse_compressed_tensor_ctor`、`sparse_csr_tensor_ctor`、`sparse_csc_tensor_ctor`、`sparse_bsr_tensor_ctor`、`sparse_bsc_tensor_ctor` 等例程，它们构成了这一段的关键 API 或控制流程。

### Lines 79-104
```cpp
 79 | void _validate_sparse_compressed_tensor_args(
 80 |     c10::DispatchKey dispatch_key,
 81 |     at::ScalarType scalar_type,
 82 |     PyObject* args,
 83 |     PyObject* kwargs);
 84 | void _validate_sparse_csr_tensor_args(
 85 |     c10::DispatchKey dispatch_key,
 86 |     at::ScalarType scalar_type,
 87 |     PyObject* args,
 88 |     PyObject* kwargs);
 89 | void _validate_sparse_csc_tensor_args(
 90 |     c10::DispatchKey dispatch_key,
 91 |     at::ScalarType scalar_type,
 92 |     PyObject* args,
 93 |     PyObject* kwargs);
 94 | void _validate_sparse_bsr_tensor_args(
 95 |     c10::DispatchKey dispatch_key,
 96 |     at::ScalarType scalar_type,
 97 |     PyObject* args,
 98 |     PyObject* kwargs);
 99 | void _validate_sparse_bsc_tensor_args(
100 |     c10::DispatchKey dispatch_key,
101 |     at::ScalarType scalar_type,
102 |     PyObject* args,
103 |     PyObject* kwargs);
104 | 
```
- EN: Declares routines such as `_validate_sparse_compressed_tensor_args`, `_validate_sparse_csr_tensor_args`, `_validate_sparse_csc_tensor_args`, `_validate_sparse_bsr_tensor_args`, `_validate_sparse_bsc_tensor_args` that expose the key API or control flow of this region.
- CN: 声明了 `_validate_sparse_compressed_tensor_args`、`_validate_sparse_csr_tensor_args`、`_validate_sparse_csc_tensor_args`、`_validate_sparse_bsr_tensor_args`、`_validate_sparse_bsc_tensor_args` 等例程，它们构成了这一段的关键 API 或控制流程。

### Lines 105-128
```cpp
105 | at::Tensor tensor_ctor(
106 |     c10::DispatchKey dispatch_key,
107 |     at::ScalarType scalar_type,
108 |     PythonArgs& r);
109 | at::Tensor as_tensor(
110 |     c10::DispatchKey dispatch_key,
111 |     at::ScalarType scalar_type,
112 |     PythonArgs& r);
113 | at::Tensor new_tensor(
114 |     c10::DispatchKey dispatch_key,
115 |     at::ScalarType scalar_type,
116 |     PyObject* args,
117 |     PyObject* kwargs);
118 | at::Tensor new_ones(
119 |     c10::DispatchKey dispatch_key,
120 |     at::ScalarType scalar_type,
121 |     PyObject* args,
122 |     PyObject* kwargs);
123 | at::Tensor tensor_frombuffer(
124 |     PyObject* buffer,
125 |     at::ScalarType dtype,
126 |     int64_t count,
127 |     int64_t offset,
128 |     bool requires_grad);
```
- EN: Declares routines such as `tensor_ctor`, `as_tensor`, `new_tensor`, `new_ones`, `tensor_frombuffer` that expose the key API or control flow of this region.
- CN: 声明了 `tensor_ctor`、`as_tensor`、`new_tensor`、`new_ones`、`tensor_frombuffer` 等例程，它们构成了这一段的关键 API 或控制流程。

### Lines 129-136
```cpp
129 | at::Tensor tensor_fromDLPack(PyObject* data);
130 | at::Tensor asarray(
131 |     PyObject* obj,
132 |     std::optional<c10::ScalarType> dtype,
133 |     std::optional<c10::Device> device,
134 |     std::optional<bool> copy,
135 |     std::optional<bool> requires_grad);
136 | } // namespace torch::utils
```
- EN: Declares routines such as `tensor_fromDLPack`, `asarray` that expose the key API or control flow of this region.
- CN: 声明了 `tensor_fromDLPack`、`asarray` 等例程，它们构成了这一段的关键 API 或控制流程。

## Key Concepts / 关键概念

- **Important routines / 重要例程**
  - EN: `only_lift_cpu_tensors`, `set_only_lift_cpu_tensors`, `base_tensor_ctor`, `legacy_tensor_ctor`, `legacy_tensor_new`, `indexing_tensor_from_data`, `sparse_coo_tensor_ctor`, `_validate_sparse_coo_tensor_args`.
  - CN: `only_lift_cpu_tensors`、`set_only_lift_cpu_tensors`、`base_tensor_ctor`、`legacy_tensor_ctor`、`legacy_tensor_new`、`indexing_tensor_from_data`、`sparse_coo_tensor_ctor`、`_validate_sparse_coo_tensor_args`。
- **Subsystem scope / 子系统作用域**
  - EN: The code lives under `torch::utils`, indicating the owning PyTorch subsystem.
  - CN: 代码位于 `torch::utils` 命名空间下，这说明了其所属的 PyTorch 子系统。

## Dependencies / 依赖关系

- Local includes / 本地头文件: `<torch/csrc/python_headers.h>`, `<torch/csrc/utils/python_arg_parser.h>`, `<ATen/core/Tensor.h>`
- External includes / 外部头文件: None / 无
- Relationship / 关系:
  - EN: The unit relies on neighboring PyTorch headers for subsystem-specific types, dispatcher hooks, and utility helpers. Its exported or visible routines are then consumed by callers in the same namespace and by higher-level runtime paths elsewhere in PyTorch.
  - CN: 该编译单元依赖相邻的 PyTorch 头文件，以获得子系统类型、分发器钩子和辅助工具。 随后，这里导出或可见的例程会被同一命名空间中的调用方，以及 PyTorch 其他位置的更高层运行时路径所使用。
