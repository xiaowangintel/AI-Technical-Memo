# python_numbers.h — Code Analysis / 代码分析

## Source / 来源
- File / 文件: `torch/csrc/utils/python_numbers.h`
- Repository / 仓库: `pytorch`
- Purpose / 作用:
  - EN: Bridges C++ components to Python-facing APIs or bindings.
  - CN: 把 C++ 组件桥接到面向 Python 的 API 或绑定。

## Line-by-Line Analysis / 逐行分析

### Lines 1-39
```cpp
 1 | #pragma once
 2 | 
 3 | #include <c10/core/Device.h>
 4 | #include <torch/csrc/Exceptions.h>
 5 | #include <torch/csrc/jit/frontend/tracer.h>
 6 | #include <torch/csrc/python_headers.h>
 7 | #include <torch/csrc/utils/object_ptr.h>
 8 | #include <torch/csrc/utils/tensor_numpy.h>
 9 | #include <cstdint>
10 | #include <limits>
11 | #include <stdexcept>
12 | 
13 | // largest integer that can be represented consecutively in a double
14 | const int64_t DOUBLE_INT_MAX = 9007199254740992;
15 | 
16 | inline PyObject* THPUtils_packDeviceIndex(c10::DeviceIndex value) {
17 |   return PyLong_FromLong(value);
18 | }
19 | 
20 | inline PyObject* THPUtils_packInt32(int32_t value) {
21 |   return PyLong_FromLong(value);
22 | }
23 | 
24 | inline PyObject* THPUtils_packInt64(int64_t value) {
25 |   return PyLong_FromLongLong(value);
26 | }
27 | 
28 | inline PyObject* THPUtils_packUInt32(uint32_t value) {
29 |   return PyLong_FromUnsignedLong(value);
30 | }
31 | 
32 | inline PyObject* THPUtils_packUInt64(uint64_t value) {
33 |   return PyLong_FromUnsignedLongLong(value);
34 | }
35 | 
36 | inline PyObject* THPUtils_packDoubleAsInt(double value) {
37 |   return PyLong_FromDouble(value);
38 | }
39 | 
```
- EN: Brings in project headers such as `<c10/core/Device.h>`, `<torch/csrc/Exceptions.h>`, `<torch/csrc/jit/frontend/tracer.h>`, `<torch/csrc/python_headers.h>` and system or third-party headers such as `<cstdint>`, `<limits>`, `<stdexcept>` so this section can use their types, APIs, or macros. Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Implements routines such as `THPUtils_packDeviceIndex`, `PyLong_FromLong`, `THPUtils_packInt32`, `THPUtils_packInt64`, `PyLong_FromLongLong` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 这里引入了项目头文件，例如 `<c10/core/Device.h>`、`<torch/csrc/Exceptions.h>`、`<torch/csrc/jit/frontend/tracer.h>`、`<torch/csrc/python_headers.h>`以及系统或第三方头文件，例如 `<cstdint>`、`<limits>`、`<stdexcept>`，使本段代码能够使用相关类型、API 或宏。 使用预处理指令来控制编译开关、平台分支或导出声明。 实现了 `THPUtils_packDeviceIndex`、`PyLong_FromLong`、`THPUtils_packInt32`、`THPUtils_packInt64`、`PyLong_FromLongLong` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 40-72
```cpp
40 | inline bool THPUtils_checkLongExact(PyObject* obj) {
41 |   return PyLong_CheckExact(obj) && !PyBool_Check(obj);
42 | }
43 | 
44 | inline bool THPUtils_checkLong(PyObject* obj) {
45 |   // Fast path
46 |   if (THPUtils_checkLongExact(obj)) {
47 |     return true;
48 |   }
49 | 
50 | #ifdef USE_NUMPY
51 |   if (torch::utils::is_numpy_int(obj)) {
52 |     return true;
53 |   }
54 | #endif
55 | 
56 |   return PyLong_Check(obj) && !PyBool_Check(obj);
57 | }
58 | 
59 | inline int32_t THPUtils_unpackInt(PyObject* obj) {
60 |   int overflow = 0;
61 |   long value = PyLong_AsLongAndOverflow(obj, &overflow);
62 |   if (value == -1 && PyErr_Occurred()) {
63 |     throw python_error();
64 |   }
65 |   TORCH_CHECK_VALUE(overflow == 0, "Overflow when unpacking long long");
66 |   TORCH_CHECK_VALUE(
67 |       value <= std::numeric_limits<int32_t>::max() &&
68 |           value >= std::numeric_limits<int32_t>::min(),
69 |       "Overflow when unpacking long");
70 |   return (int32_t)value;
71 | }
72 | 
```
- EN: Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Implements routines such as `THPUtils_checkLongExact`, `PyLong_CheckExact`, `THPUtils_checkLong`, `PyLong_Check`, `THPUtils_unpackInt` that expose the key API or control flow of this region. At the statement level, this block declares abstract hooks that derived backends must provide; returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 使用预处理指令来控制编译开关、平台分支或导出声明。 实现了 `THPUtils_checkLongExact`、`PyLong_CheckExact`、`THPUtils_checkLong`、`PyLong_Check`、`THPUtils_unpackInt` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段声明了需要由派生后端提供的抽象钩子；返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

### Lines 73-103
```cpp
 73 | inline int64_t THPUtils_unpackLong(PyObject* obj) {
 74 |   int overflow = 0;
 75 |   long long value = PyLong_AsLongLongAndOverflow(obj, &overflow);
 76 |   if (value == -1 && PyErr_Occurred()) {
 77 |     throw python_error();
 78 |   }
 79 |   TORCH_CHECK_VALUE(overflow == 0, "Overflow when unpacking long long");
 80 |   return (int64_t)value;
 81 | }
 82 | 
 83 | inline uint32_t THPUtils_unpackUInt32(PyObject* obj) {
 84 |   unsigned long value = PyLong_AsUnsignedLong(obj);
 85 |   if (PyErr_Occurred()) {
 86 |     throw python_error();
 87 |   }
 88 |   TORCH_CHECK_VALUE(
 89 |       value <= std::numeric_limits<uint32_t>::max(),
 90 |       "Overflow when unpacking long long");
 91 |   return (uint32_t)value;
 92 | }
 93 | 
 94 | inline uint64_t THPUtils_unpackUInt64(PyObject* obj) {
 95 |   unsigned long long value = PyLong_AsUnsignedLongLong(obj);
 96 |   if (PyErr_Occurred()) {
 97 |     throw python_error();
 98 |   }
 99 |   return (uint64_t)value;
100 | }
101 | 
102 | bool THPUtils_checkIndex(PyObject* obj);
103 | 
```
- EN: Implements routines such as `THPUtils_unpackLong`, `python_error`, `THPUtils_unpackUInt32`, `THPUtils_unpackUInt64`, `THPUtils_checkIndex` that expose the key API or control flow of this region. At the statement level, this block declares abstract hooks that derived backends must provide; returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 实现了 `THPUtils_unpackLong`、`python_error`、`THPUtils_unpackUInt32`、`THPUtils_unpackUInt64`、`THPUtils_checkIndex` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段声明了需要由派生后端提供的抽象钩子；返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

### Lines 104-135
```cpp
104 | inline int64_t THPUtils_unpackIndex(PyObject* obj) {
105 |   if (!THPUtils_checkLong(obj)) {
106 |     auto index = THPObjectPtr(PyNumber_Index(obj));
107 |     if (index == nullptr) {
108 |       throw python_error();
109 |     }
110 |     // NB: This needs to be called before `index` goes out of scope and the
111 |     // underlying object's refcount is decremented
112 |     return THPUtils_unpackLong(index.get());
113 |   }
114 |   return THPUtils_unpackLong(obj);
115 | }
116 | 
117 | inline bool THPUtils_unpackBool(PyObject* obj) {
118 |   if (Py_IsTrue(obj)) {
119 |     return true;
120 |   } else if (Py_IsFalse(obj)) {
121 |     return false;
122 |   } else {
123 |     TORCH_CHECK(false, "couldn't convert python object to boolean");
124 |   }
125 | }
126 | 
127 | inline bool THPUtils_checkBool(PyObject* obj) {
128 | #ifdef USE_NUMPY
129 |   if (torch::utils::is_numpy_bool(obj)) {
130 |     return true;
131 |   }
132 | #endif
133 |   return PyBool_Check(obj);
134 | }
135 | 
```
- EN: Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Implements routines such as `THPUtils_unpackIndex`, `python_error`, `THPUtils_unpackLong`, `THPUtils_unpackBool`, `THPUtils_checkBool` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 使用预处理指令来控制编译开关、平台分支或导出声明。 实现了 `THPUtils_unpackIndex`、`python_error`、`THPUtils_unpackLong`、`THPUtils_unpackBool`、`THPUtils_checkBool` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

### Lines 136-164
```cpp
136 | inline bool THPUtils_checkDouble(PyObject* obj) {
137 | #ifdef USE_NUMPY
138 |   if (torch::utils::is_numpy_scalar(obj)) {
139 |     return true;
140 |   }
141 | #endif
142 |   return PyFloat_Check(obj) || PyLong_Check(obj);
143 | }
144 | 
145 | inline double THPUtils_unpackDouble(PyObject* obj) {
146 |   if (PyFloat_Check(obj)) {
147 |     return PyFloat_AS_DOUBLE(obj);
148 |   }
149 |   double value = PyFloat_AsDouble(obj);
150 |   if (value == -1 && PyErr_Occurred()) {
151 |     throw python_error();
152 |   }
153 |   return value;
154 | }
155 | 
156 | inline c10::complex<double> THPUtils_unpackComplexDouble(PyObject* obj) {
157 |   Py_complex value = PyComplex_AsCComplex(obj);
158 |   if (value.real == -1.0 && PyErr_Occurred()) {
159 |     throw python_error();
160 |   }
161 | 
162 |   return c10::complex<double>(value.real, value.imag);
163 | }
164 | 
```
- EN: Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Implements routines such as `THPUtils_checkDouble`, `PyFloat_Check`, `THPUtils_unpackDouble`, `PyFloat_AS_DOUBLE`, `python_error` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 使用预处理指令来控制编译开关、平台分支或导出声明。 实现了 `THPUtils_checkDouble`、`PyFloat_Check`、`THPUtils_unpackDouble`、`PyFloat_AS_DOUBLE`、`python_error` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 165-195
```cpp
165 | inline bool THPUtils_unpackNumberAsBool(PyObject* obj) {
166 | #ifdef USE_NUMPY
167 |   // Handle NumPy boolean scalars (np.bool_)
168 |   if (torch::utils::is_numpy_bool(obj)) {
169 |     int truth = PyObject_IsTrue(obj);
170 |     if (truth == -1) {
171 |       throw python_error();
172 |     }
173 |     return truth != 0;
174 |   }
175 | #endif
176 |   if (PyFloat_Check(obj)) {
177 |     return (bool)PyFloat_AS_DOUBLE(obj);
178 |   }
179 | 
180 |   if (PyComplex_Check(obj)) {
181 |     double real_val = PyComplex_RealAsDouble(obj);
182 |     double imag_val = PyComplex_ImagAsDouble(obj);
183 |     return !(real_val == 0 && imag_val == 0);
184 |   }
185 | 
186 |   int overflow = 0;
187 |   long long value = PyLong_AsLongLongAndOverflow(obj, &overflow);
188 |   if (value == -1 && PyErr_Occurred()) {
189 |     throw python_error();
190 |   }
191 |   // No need to check overflow, because when overflow occurred, it should
192 |   // return true in order to keep the same behavior of numpy.
193 |   return (bool)value;
194 | }
195 | 
```
- EN: Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Implements routines such as `THPUtils_unpackNumberAsBool`, `python_error` that expose the key API or control flow of this region. At the statement level, this block declares abstract hooks that derived backends must provide; returns computed state or forwards values to callers.
- CN: 使用预处理指令来控制编译开关、平台分支或导出声明。 实现了 `THPUtils_unpackNumberAsBool`、`python_error` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段声明了需要由派生后端提供的抽象钩子；返回计算状态或把值转交给调用方。

### Lines 196-227
```cpp
196 | inline c10::DeviceIndex THPUtils_unpackDeviceIndex(PyObject* obj) {
197 |   int overflow = 0;
198 |   long value = PyLong_AsLongAndOverflow(obj, &overflow);
199 |   if (value == -1 && PyErr_Occurred()) {
200 |     throw python_error();
201 |   }
202 |   TORCH_CHECK(overflow == 0, "Overflow when unpacking DeviceIndex");
203 |   TORCH_CHECK(
204 |       value <= std::numeric_limits<c10::DeviceIndex>::max() &&
205 |           value >= std::numeric_limits<c10::DeviceIndex>::min(),
206 |       "Overflow when unpacking DeviceIndex");
207 |   return (c10::DeviceIndex)value;
208 | }
209 | 
210 | template <typename T>
211 | inline T THPUtils_unpackInteger(PyObject* obj) {
212 |   int overflow = -1;
213 |   const auto value = PyLong_AsLongLongAndOverflow(obj, &overflow);
214 |   if (value == -1 && PyErr_Occurred()) {
215 |     throw python_error();
216 |   }
217 |   if (!overflow) {
218 |     return static_cast<int64_t>(value);
219 |   }
220 |   // try unsigned
221 |   const auto uvalue = PyLong_AsUnsignedLongLong(obj);
222 |   if (uvalue == static_cast<std::decay_t<decltype(uvalue)>>(-1) &&
223 |       PyErr_Occurred()) {
224 |     throw python_error();
225 |   }
226 |   return static_cast<uint64_t>(uvalue);
227 | }
```
- EN: Implements routines such as `THPUtils_unpackDeviceIndex`, `python_error`, `THPUtils_unpackInteger` that expose the key API or control flow of this region. At the statement level, this block declares abstract hooks that derived backends must provide; returns computed state or forwards values to callers; validates assumptions and reports descriptive failures.
- CN: 实现了 `THPUtils_unpackDeviceIndex`、`python_error`、`THPUtils_unpackInteger` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段声明了需要由派生后端提供的抽象钩子；返回计算状态或把值转交给调用方；校验前提条件并报告明确错误。

## Key Concepts / 关键概念

- **Important routines / 重要例程**
  - EN: `THPUtils_packDeviceIndex`, `PyLong_FromLong`, `THPUtils_packInt32`, `THPUtils_packInt64`, `PyLong_FromLongLong`, `THPUtils_packUInt32`, `PyLong_FromUnsignedLong`, `THPUtils_packUInt64`.
  - CN: `THPUtils_packDeviceIndex`、`PyLong_FromLong`、`THPUtils_packInt32`、`THPUtils_packInt64`、`PyLong_FromLongLong`、`THPUtils_packUInt32`、`PyLong_FromUnsignedLong`、`THPUtils_packUInt64`。

## Dependencies / 依赖关系

- Local includes / 本地头文件: `<c10/core/Device.h>`, `<torch/csrc/Exceptions.h>`, `<torch/csrc/jit/frontend/tracer.h>`, `<torch/csrc/python_headers.h>`, `<torch/csrc/utils/object_ptr.h>`, `<torch/csrc/utils/tensor_numpy.h>`
- External includes / 外部头文件: `<cstdint>`, `<limits>`, `<stdexcept>`
- Relationship / 关系:
  - EN: The unit relies on neighboring PyTorch headers for subsystem-specific types, dispatcher hooks, and utility helpers. Standard-library or third-party headers provide generic facilities such as containers, threading, math, or backend integration points. Its exported or visible routines are then consumed by callers in the same namespace and by higher-level runtime paths elsewhere in PyTorch.
  - CN: 该编译单元依赖相邻的 PyTorch 头文件，以获得子系统类型、分发器钩子和辅助工具。 标准库或第三方头文件提供了通用能力，例如容器、线程、数学支持或后端集成入口。 随后，这里导出或可见的例程会被同一命名空间中的调用方，以及 PyTorch 其他位置的更高层运行时路径所使用。
