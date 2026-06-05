# python_fft_functions.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/autograd/generated/python_fft_functions.cpp`
- Repository: `/root/xw/pytorch`
- Purpose (EN): Defines generated backward-function classes and helpers used by the autograd engine.
- 目的 (CN): 定义自动求导引擎使用的生成式反向函数类与辅助逻辑。
- Lines: 890
- Language: C++ / C++

## Line-by-Line Analysis / 逐行分析

- Note / 说明: This file is generated or mechanically expanded, so the analysis groups contiguous line ranges rather than paraphrasing every repeated wrapper individually. / 该文件为生成代码或机械展开代码，因此分析按连续行范围组织，而不是逐个重复改写每个封装模板。
### Lines 1-20

```cpp
 1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
 2: // @generated from ../tools/autograd/templates/python_fft_functions.cpp
 3: 
 4: #include "torch/csrc/Device.h"
 5: #include "torch/csrc/DynamicTypes.h"
 6: #include "torch/csrc/Exceptions.h"
 7: #include "torch/csrc/autograd/python_fft_functions.h"
 8: #include "torch/csrc/autograd/generated/python_return_types.h"
 9: #include "torch/csrc/autograd/python_variable.h"
10: #include "torch/csrc/autograd/utils/wrap_outputs.h"
11: #include "torch/csrc/autograd/utils/python_arg_parsing.h"
12: #include "torch/csrc/autograd/generated/variable_factories.h"
13: #include "torch/csrc/utils/out_types.h"
14: #include "torch/csrc/utils/pycfunction_helpers.h"
15: #include "torch/csrc/utils/python_arg_parser.h"
16: #include "torch/csrc/utils/structseq.h"
17: #include "torch/csrc/utils/device_lazy_init.h"
18: 
19: #include <ATen/core/Tensor.h>
20: 
```

- EN: These lines pull in dependencies such as `torch/csrc/Device.h`, `torch/csrc/DynamicTypes.h`, `torch/csrc/Exceptions.h`, establishing the headers needed by the implementation.
- CN: 这些行引入了依赖，例如 `torch/csrc/Device.h`, `torch/csrc/DynamicTypes.h`, `torch/csrc/Exceptions.h`，为后续实现建立所需的头文件基础。
### Lines 21-40

```cpp
21: #ifndef AT_PER_OPERATOR_HEADERS
22: #include <ATen/Functions.h>
23: #else
24: #include <ATen/ops/fft_fft.h>
25: #include <ATen/ops/fft_fft2.h>
26: #include <ATen/ops/fft_fftfreq.h>
27: #include <ATen/ops/fft_fftn.h>
28: #include <ATen/ops/fft_fftshift.h>
29: #include <ATen/ops/fft_hfft.h>
30: #include <ATen/ops/fft_hfft2.h>
31: #include <ATen/ops/fft_hfftn.h>
32: #include <ATen/ops/fft_ifft.h>
33: #include <ATen/ops/fft_ifft2.h>
34: #include <ATen/ops/fft_ifftn.h>
35: #include <ATen/ops/fft_ifftshift.h>
36: #include <ATen/ops/fft_ihfft.h>
37: #include <ATen/ops/fft_ihfft2.h>
38: #include <ATen/ops/fft_ihfftn.h>
39: #include <ATen/ops/fft_irfft.h>
40: #include <ATen/ops/fft_irfft2.h>
```

- EN: These lines pull in dependencies such as `ATen/Functions.h`, `ATen/ops/fft_fft.h`, `ATen/ops/fft_fft2.h`, establishing the headers needed by the implementation. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这些行引入了依赖，例如 `ATen/Functions.h`, `ATen/ops/fft_fft.h`, `ATen/ops/fft_fft2.h`，为后续实现建立所需的头文件基础。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 41-60

```cpp
41: #include <ATen/ops/fft_irfftn.h>
42: #include <ATen/ops/fft_rfft.h>
43: #include <ATen/ops/fft_rfft2.h>
44: #include <ATen/ops/fft_rfftfreq.h>
45: #include <ATen/ops/fft_rfftn.h>
46: #endif
47: 
48: using at::Tensor;
49: using at::Device;
50: using at::Layout;
51: using at::Scalar;
52: using at::ScalarType;
53: using at::Backend;
54: using at::OptionalDeviceGuard;
55: using at::DeviceGuard;
56: using at::TensorOptions;
57: using at::IntArrayRef;
58: using at::Generator;
59: using at::TensorList;
60: using at::Dimname;
```

- EN: These lines pull in dependencies such as `ATen/ops/fft_irfftn.h`, `ATen/ops/fft_rfft.h`, `ATen/ops/fft_rfft2.h`, establishing the headers needed by the implementation. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这些行引入了依赖，例如 `ATen/ops/fft_irfftn.h`, `ATen/ops/fft_rfft.h`, `ATen/ops/fft_rfft2.h`，为后续实现建立所需的头文件基础。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 61-80

```cpp
61: using at::DimnameList;
62: 
63: using torch::utils::check_out_type_matches;
64: using namespace torch::autograd::utils;
65: 
66: namespace torch::autograd {
67: 
68: // generated forward declarations start here
69: 
70: static PyObject * THPVariable_fft_fft(PyObject* self_, PyObject* args, PyObject* kwargs);
71: static PyObject * THPVariable_fft_fft2(PyObject* self_, PyObject* args, PyObject* kwargs);
72: static PyObject * THPVariable_fft_fftfreq(PyObject* self_, PyObject* args, PyObject* kwargs);
73: static PyObject * THPVariable_fft_fftn(PyObject* self_, PyObject* args, PyObject* kwargs);
74: static PyObject * THPVariable_fft_fftshift(PyObject* self_, PyObject* args, PyObject* kwargs);
75: static PyObject * THPVariable_fft_hfft(PyObject* self_, PyObject* args, PyObject* kwargs);
76: static PyObject * THPVariable_fft_hfft2(PyObject* self_, PyObject* args, PyObject* kwargs);
77: static PyObject * THPVariable_fft_hfftn(PyObject* self_, PyObject* args, PyObject* kwargs);
78: static PyObject * THPVariable_fft_ifft(PyObject* self_, PyObject* args, PyObject* kwargs);
79: static PyObject * THPVariable_fft_ifft2(PyObject* self_, PyObject* args, PyObject* kwargs);
80: static PyObject * THPVariable_fft_ifftn(PyObject* self_, PyObject* args, PyObject* kwargs);
```

- EN: They also open, refine, or close the relevant C++ namespace scope. The main execution path in this span is carried by `THPVariable_fft_fft`, `THPVariable_fft_fft2`, `THPVariable_fft_fftfreq`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 它们还会打开、细化或关闭相应的 C++ 命名空间作用域。 这一段的主要执行路径由 `THPVariable_fft_fft`, `THPVariable_fft_fft2`, `THPVariable_fft_fftfreq` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 81-100

```cpp
 81: static PyObject * THPVariable_fft_ifftshift(PyObject* self_, PyObject* args, PyObject* kwargs);
 82: static PyObject * THPVariable_fft_ihfft(PyObject* self_, PyObject* args, PyObject* kwargs);
 83: static PyObject * THPVariable_fft_ihfft2(PyObject* self_, PyObject* args, PyObject* kwargs);
 84: static PyObject * THPVariable_fft_ihfftn(PyObject* self_, PyObject* args, PyObject* kwargs);
 85: static PyObject * THPVariable_fft_irfft(PyObject* self_, PyObject* args, PyObject* kwargs);
 86: static PyObject * THPVariable_fft_irfft2(PyObject* self_, PyObject* args, PyObject* kwargs);
 87: static PyObject * THPVariable_fft_irfftn(PyObject* self_, PyObject* args, PyObject* kwargs);
 88: static PyObject * THPVariable_fft_rfft(PyObject* self_, PyObject* args, PyObject* kwargs);
 89: static PyObject * THPVariable_fft_rfft2(PyObject* self_, PyObject* args, PyObject* kwargs);
 90: static PyObject * THPVariable_fft_rfftfreq(PyObject* self_, PyObject* args, PyObject* kwargs);
 91: static PyObject * THPVariable_fft_rfftn(PyObject* self_, PyObject* args, PyObject* kwargs);
 92: 
 93: static PyMethodDef fft_functions[] = {
 94:   {"fft_fft", castPyCFunctionWithKeywords(THPVariable_fft_fft), METH_VARARGS | METH_KEYWORDS, nullptr},
 95:   {"fft_fft2", castPyCFunctionWithKeywords(THPVariable_fft_fft2), METH_VARARGS | METH_KEYWORDS, nullptr},
 96:   {"fft_fftfreq", castPyCFunctionWithKeywords(THPVariable_fft_fftfreq), METH_VARARGS | METH_KEYWORDS, nullptr},
 97:   {"fft_fftn", castPyCFunctionWithKeywords(THPVariable_fft_fftn), METH_VARARGS | METH_KEYWORDS, nullptr},
 98:   {"fft_fftshift", castPyCFunctionWithKeywords(THPVariable_fft_fftshift), METH_VARARGS | METH_KEYWORDS, nullptr},
 99:   {"fft_hfft", castPyCFunctionWithKeywords(THPVariable_fft_hfft), METH_VARARGS | METH_KEYWORDS, nullptr},
100:   {"fft_hfft2", castPyCFunctionWithKeywords(THPVariable_fft_hfft2), METH_VARARGS | METH_KEYWORDS, nullptr},
```

- EN: The main execution path in this span is carried by `THPVariable_fft_ifftshift`, `THPVariable_fft_ihfft`, `THPVariable_fft_ihfft2`. Because this is generated binding code, the span mostly registers or forwards APIs into a mechanically produced Python-facing surface. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_fft_ifftshift`, `THPVariable_fft_ihfft`, `THPVariable_fft_ihfft2` 等函数/方法承载。 由于这是生成的绑定代码，这一段主要是在机械化生成的 Python 接口层上完成 API 注册或转发。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 101-120

```cpp
101:   {"fft_hfftn", castPyCFunctionWithKeywords(THPVariable_fft_hfftn), METH_VARARGS | METH_KEYWORDS, nullptr},
102:   {"fft_ifft", castPyCFunctionWithKeywords(THPVariable_fft_ifft), METH_VARARGS | METH_KEYWORDS, nullptr},
103:   {"fft_ifft2", castPyCFunctionWithKeywords(THPVariable_fft_ifft2), METH_VARARGS | METH_KEYWORDS, nullptr},
104:   {"fft_ifftn", castPyCFunctionWithKeywords(THPVariable_fft_ifftn), METH_VARARGS | METH_KEYWORDS, nullptr},
105:   {"fft_ifftshift", castPyCFunctionWithKeywords(THPVariable_fft_ifftshift), METH_VARARGS | METH_KEYWORDS, nullptr},
106:   {"fft_ihfft", castPyCFunctionWithKeywords(THPVariable_fft_ihfft), METH_VARARGS | METH_KEYWORDS, nullptr},
107:   {"fft_ihfft2", castPyCFunctionWithKeywords(THPVariable_fft_ihfft2), METH_VARARGS | METH_KEYWORDS, nullptr},
108:   {"fft_ihfftn", castPyCFunctionWithKeywords(THPVariable_fft_ihfftn), METH_VARARGS | METH_KEYWORDS, nullptr},
109:   {"fft_irfft", castPyCFunctionWithKeywords(THPVariable_fft_irfft), METH_VARARGS | METH_KEYWORDS, nullptr},
110:   {"fft_irfft2", castPyCFunctionWithKeywords(THPVariable_fft_irfft2), METH_VARARGS | METH_KEYWORDS, nullptr},
111:   {"fft_irfftn", castPyCFunctionWithKeywords(THPVariable_fft_irfftn), METH_VARARGS | METH_KEYWORDS, nullptr},
112:   {"fft_rfft", castPyCFunctionWithKeywords(THPVariable_fft_rfft), METH_VARARGS | METH_KEYWORDS, nullptr},
113:   {"fft_rfft2", castPyCFunctionWithKeywords(THPVariable_fft_rfft2), METH_VARARGS | METH_KEYWORDS, nullptr},
114:   {"fft_rfftfreq", castPyCFunctionWithKeywords(THPVariable_fft_rfftfreq), METH_VARARGS | METH_KEYWORDS, nullptr},
115:   {"fft_rfftn", castPyCFunctionWithKeywords(THPVariable_fft_rfftn), METH_VARARGS | METH_KEYWORDS, nullptr},
116:   {NULL}
117: };
118: 
119: static PyObject* THPFFTVariableFunctionsModule = NULL;
120: 
```

- EN: The main execution path in this span is carried by `castPyCFunctionWithKeywords`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `castPyCFunctionWithKeywords` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 121-140

```cpp
121: void initFFTFunctions(PyObject* module) {
122:   static struct PyModuleDef def = {
123:      PyModuleDef_HEAD_INIT,
124:      "torch._C._fft",
125:      NULL,
126:      -1,
127:      fft_functions
128:   };
129:   PyObject* fft = PyModule_Create(&def);
130:   THPFFTVariableFunctionsModule = fft;
131:   if (!fft) {
132:     throw python_error();
133:   }
134:   // steals a reference to fft
135:   if (PyModule_AddObject(module, "_fft", fft) != 0) {
136:     throw python_error();
137:   }
138: }
139: 
140: // generated methods start here
```

- EN: The main execution path in this span is carried by `initFFTFunctions`, `PyModule_Create`, `python_error`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `initFFTFunctions`, `PyModule_Create`, `python_error` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 141-160

```cpp
141: 
142: // fft_fft
143: static PyObject * THPVariable_fft_fft(PyObject* self_, PyObject* args, PyObject* kwargs)
144: {
145:   HANDLE_TH_ERRORS
146:   static PythonArgParser parser({
147:     "fft_fft(Tensor input, SymInt? n=None, int64_t dim=-1, c10::string_view? norm=None, *, Tensor out=None)",
148:   }, /*traceable=*/true);
149: 
150:   ParsedArgs<5> parsed_args;
151:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
152:   if(_r.has_torch_function()) {
153:     return handle_torch_function(_r, nullptr, args, kwargs, THPFFTVariableFunctionsModule, "torch.fft");
154:   }
155:   if (_r.isNone(4)) {
156:     // aten::fft_fft(Tensor self, SymInt? n=None, int dim=-1, str? norm=None) -> Tensor
157: 
158:     auto dispatch_fft_fft = [](const at::Tensor & self, ::std::optional<c10::SymInt> n, int64_t dim, ::std::optional<c10::string_view> norm) -> at::Tensor {
159:       pybind11::gil_scoped_release no_gil;
160:       return at::fft_fft_symint(self, n, dim, norm);
```

- EN: The main execution path in this span is carried by `THPVariable_fft_fft`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_fft_fft`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 161-180

```cpp
161:     };
162:     return wrap(dispatch_fft_fft(_r.tensor(0), _r.toSymIntOptional(1), _r.toInt64(2), _r.stringViewOptional(3)));
163:   } else {
164:     // aten::fft_fft.out(Tensor self, SymInt? n=None, int dim=-1, str? norm=None, *, Tensor(a!) out) -> Tensor(a!)
165: 
166:     auto dispatch_fft_fft_out = [](at::Tensor out, const at::Tensor & self, ::std::optional<c10::SymInt> n, int64_t dim, ::std::optional<c10::string_view> norm) -> at::Tensor {
167:       pybind11::gil_scoped_release no_gil;
168:       return at::fft_fft_symint_out(out, self, n, dim, norm);
169:     };
170:     return wrap(dispatch_fft_fft_out(_r.tensor(4), _r.tensor(0), _r.toSymIntOptional(1), _r.toInt64(2), _r.stringViewOptional(3)));
171:   }
172:   Py_RETURN_NONE;
173:   END_HANDLE_TH_ERRORS
174: }
175: 
176: // fft_fft2
177: static PyObject * THPVariable_fft_fft2(PyObject* self_, PyObject* args, PyObject* kwargs)
178: {
179:   HANDLE_TH_ERRORS
180:   static PythonArgParser parser({
```

- EN: The main execution path in this span is carried by `wrap`, `fft_fft_symint_out`, `THPVariable_fft_fft2`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `fft_fft_symint_out`, `THPVariable_fft_fft2` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 181-200

```cpp
181:     "fft_fft2(Tensor input, SymIntArrayRef[1]? s=None, IntArrayRef[1] dim={-2,-1}, c10::string_view? norm=None, *, Tensor out=None)",
182:   }, /*traceable=*/true);
183: 
184:   ParsedArgs<5> parsed_args;
185:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
186:   if(_r.has_torch_function()) {
187:     return handle_torch_function(_r, nullptr, args, kwargs, THPFFTVariableFunctionsModule, "torch.fft");
188:   }
189:   if (_r.isNone(4)) {
190:     // aten::fft_fft2(Tensor self, SymInt[1]? s=None, int[1] dim=[-2,-1], str? norm=None) -> Tensor
191: 
192:     auto dispatch_fft_fft2 = [](const at::Tensor & self, at::OptionalSymIntArrayRef s, at::IntArrayRef dim, ::std::optional<c10::string_view> norm) -> at::Tensor {
193:       pybind11::gil_scoped_release no_gil;
194:       return at::fft_fft2_symint(self, s, dim, norm);
195:     };
196:     return wrap(dispatch_fft_fft2(_r.tensor(0), _r.symintlistOptional(1), _r.intlist(2), _r.stringViewOptional(3)));
197:   } else {
198:     // aten::fft_fft2.out(Tensor self, SymInt[1]? s=None, int[1] dim=[-2,-1], str? norm=None, *, Tensor(a!) out) -> Tensor(a!)
199: 
200:     auto dispatch_fft_fft2_out = [](at::Tensor out, const at::Tensor & self, at::OptionalSymIntArrayRef s, at::IntArrayRef dim, ::std::optional<c10::string_view> norm) -> at::Tensor {
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `fft_fft2`, `fft_fft2_symint`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `fft_fft2`, `fft_fft2_symint` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 201-220

```cpp
201:       pybind11::gil_scoped_release no_gil;
202:       return at::fft_fft2_symint_out(out, self, s, dim, norm);
203:     };
204:     return wrap(dispatch_fft_fft2_out(_r.tensor(4), _r.tensor(0), _r.symintlistOptional(1), _r.intlist(2), _r.stringViewOptional(3)));
205:   }
206:   Py_RETURN_NONE;
207:   END_HANDLE_TH_ERRORS
208: }
209: 
210: // fft_fftfreq
211: static PyObject * THPVariable_fft_fftfreq(PyObject* self_, PyObject* args, PyObject* kwargs)
212: {
213:   HANDLE_TH_ERRORS
214:   static PythonArgParser parser({
215:     "fft_fftfreq(int64_t n, double d=1.0, *, Tensor out=None, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=False, bool? requires_grad=False)",
216:   }, /*traceable=*/true);
217: 
218:   ParsedArgs<8> parsed_args;
219:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
220:   if(_r.has_torch_function()) {
```

- EN: The main execution path in this span is carried by `fft_fft2_symint_out`, `wrap`, `THPVariable_fft_fftfreq`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `fft_fft2_symint_out`, `wrap`, `THPVariable_fft_fftfreq` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 221-240

```cpp
221:     return handle_torch_function(_r, nullptr, args, kwargs, THPFFTVariableFunctionsModule, "torch.fft");
222:   }
223:   if (_r.isNone(2)) {
224:     // aten::fft_fftfreq(int n, float d=1.0, *, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=None) -> Tensor
225:     const auto options = TensorOptions()
226:         .dtype(_r.scalartypeOptional(3))
227:         .device(_r.deviceWithDefault(5, torch::tensors::get_default_device()))
228:         .layout(_r.layoutOptional(4))
229:         .requires_grad(_r.toBool(7))
230:         .pinned_memory(_r.toBool(6));
231:     torch::utils::maybe_initialize_device(options);
232: 
233:     auto dispatch_fft_fftfreq = [](int64_t n, double d, at::TensorOptions options) -> at::Tensor {
234:       pybind11::gil_scoped_release no_gil;
235:       return torch::fft_fftfreq(n, d, options);
236:     };
237:     return wrap(dispatch_fft_fftfreq(_r.toInt64(0), _r.toDouble(1), options));
238:   } else {
239:     // aten::fft_fftfreq.out(int n, float d=1.0, *, Tensor(a!) out) -> Tensor(a!)
240:     check_out_type_matches(_r.tensor(2), _r.scalartypeOptional(3),
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `fft_fftfreq`, `TensorOptions`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `fft_fftfreq`, `TensorOptions` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 241-260

```cpp
241:                            _r.isNone(3), _r.layoutOptional(4),
242:                            _r.deviceWithDefault(5, torch::tensors::get_default_device()), _r.isNone(5));
243: 
244:     auto dispatch_fft_fftfreq_out = [](at::Tensor out, int64_t n, double d) -> at::Tensor {
245:       pybind11::gil_scoped_release no_gil;
246:       return at::fft_fftfreq_out(out, n, d);
247:     };
248:     return wrap(dispatch_fft_fftfreq_out(_r.tensor(2), _r.toInt64(0), _r.toDouble(1)).set_requires_grad(_r.toBool(7)));
249:   }
250:   Py_RETURN_NONE;
251:   END_HANDLE_TH_ERRORS
252: }
253: 
254: // fft_fftn
255: static PyObject * THPVariable_fft_fftn(PyObject* self_, PyObject* args, PyObject* kwargs)
256: {
257:   HANDLE_TH_ERRORS
258:   static PythonArgParser parser({
259:     "fft_fftn(Tensor input, SymIntArrayRef[1]? s=None, IntArrayRef[1]? dim=None, c10::string_view? norm=None, *, Tensor out=None)",
260:   }, /*traceable=*/true);
```

- EN: The main execution path in this span is carried by `fft_fftfreq_out`, `wrap`, `THPVariable_fft_fftn`. Autograd-related metadata or graph connectivity is updated here. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `fft_fftfreq_out`, `wrap`, `THPVariable_fft_fftn` 等函数/方法承载。 这里会更新与自动求导相关的元数据或计算图连接关系。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 261-280

```cpp
261: 
262:   ParsedArgs<5> parsed_args;
263:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
264:   if(_r.has_torch_function()) {
265:     return handle_torch_function(_r, nullptr, args, kwargs, THPFFTVariableFunctionsModule, "torch.fft");
266:   }
267:   if (_r.isNone(4)) {
268:     // aten::fft_fftn(Tensor self, SymInt[1]? s=None, int[1]? dim=None, str? norm=None) -> Tensor
269: 
270:     auto dispatch_fft_fftn = [](const at::Tensor & self, at::OptionalSymIntArrayRef s, at::OptionalIntArrayRef dim, ::std::optional<c10::string_view> norm) -> at::Tensor {
271:       pybind11::gil_scoped_release no_gil;
272:       return at::fft_fftn_symint(self, s, dim, norm);
273:     };
274:     return wrap(dispatch_fft_fftn(_r.tensor(0), _r.symintlistOptional(1), _r.intlistOptional(2), _r.stringViewOptional(3)));
275:   } else {
276:     // aten::fft_fftn.out(Tensor self, SymInt[1]? s=None, int[1]? dim=None, str? norm=None, *, Tensor(a!) out) -> Tensor(a!)
277: 
278:     auto dispatch_fft_fftn_out = [](at::Tensor out, const at::Tensor & self, at::OptionalSymIntArrayRef s, at::OptionalIntArrayRef dim, ::std::optional<c10::string_view> norm) -> at::Tensor {
279:       pybind11::gil_scoped_release no_gil;
280:       return at::fft_fftn_symint_out(out, self, s, dim, norm);
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `fft_fftn`, `fft_fftn_symint`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `fft_fftn`, `fft_fftn_symint` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 281-300

```cpp
281:     };
282:     return wrap(dispatch_fft_fftn_out(_r.tensor(4), _r.tensor(0), _r.symintlistOptional(1), _r.intlistOptional(2), _r.stringViewOptional(3)));
283:   }
284:   Py_RETURN_NONE;
285:   END_HANDLE_TH_ERRORS
286: }
287: 
288: // fft_fftshift
289: static PyObject * THPVariable_fft_fftshift(PyObject* self_, PyObject* args, PyObject* kwargs)
290: {
291:   HANDLE_TH_ERRORS
292:   static PythonArgParser parser({
293:     "fft_fftshift(Tensor input, IntArrayRef[1]? dim=None)",
294:   }, /*traceable=*/true);
295: 
296:   ParsedArgs<2> parsed_args;
297:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
298:   if(_r.has_torch_function()) {
299:     return handle_torch_function(_r, nullptr, args, kwargs, THPFFTVariableFunctionsModule, "torch.fft");
300:   }
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_fft_fftshift`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_fft_fftshift`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 301-320

```cpp
301:   // aten::fft_fftshift(Tensor self, int[1]? dim=None) -> Tensor
302: 
303:   auto dispatch_fft_fftshift = [](const at::Tensor & self, at::OptionalIntArrayRef dim) -> at::Tensor {
304:     pybind11::gil_scoped_release no_gil;
305:     return at::fft_fftshift(self, dim);
306:   };
307:   return wrap(dispatch_fft_fftshift(_r.tensor(0), _r.intlistOptional(1)));
308:   Py_RETURN_NONE;
309:   END_HANDLE_TH_ERRORS
310: }
311: 
312: // fft_hfft
313: static PyObject * THPVariable_fft_hfft(PyObject* self_, PyObject* args, PyObject* kwargs)
314: {
315:   HANDLE_TH_ERRORS
316:   static PythonArgParser parser({
317:     "fft_hfft(Tensor input, SymInt? n=None, int64_t dim=-1, c10::string_view? norm=None, *, Tensor out=None)",
318:   }, /*traceable=*/true);
319: 
320:   ParsedArgs<5> parsed_args;
```

- EN: The main execution path in this span is carried by `fft_fftshift`, `wrap`, `THPVariable_fft_hfft`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `fft_fftshift`, `wrap`, `THPVariable_fft_hfft` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 321-340

```cpp
321:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
322:   if(_r.has_torch_function()) {
323:     return handle_torch_function(_r, nullptr, args, kwargs, THPFFTVariableFunctionsModule, "torch.fft");
324:   }
325:   if (_r.isNone(4)) {
326:     // aten::fft_hfft(Tensor self, SymInt? n=None, int dim=-1, str? norm=None) -> Tensor
327: 
328:     auto dispatch_fft_hfft = [](const at::Tensor & self, ::std::optional<c10::SymInt> n, int64_t dim, ::std::optional<c10::string_view> norm) -> at::Tensor {
329:       pybind11::gil_scoped_release no_gil;
330:       return at::fft_hfft_symint(self, n, dim, norm);
331:     };
332:     return wrap(dispatch_fft_hfft(_r.tensor(0), _r.toSymIntOptional(1), _r.toInt64(2), _r.stringViewOptional(3)));
333:   } else {
334:     // aten::fft_hfft.out(Tensor self, SymInt? n=None, int dim=-1, str? norm=None, *, Tensor(a!) out) -> Tensor(a!)
335: 
336:     auto dispatch_fft_hfft_out = [](at::Tensor out, const at::Tensor & self, ::std::optional<c10::SymInt> n, int64_t dim, ::std::optional<c10::string_view> norm) -> at::Tensor {
337:       pybind11::gil_scoped_release no_gil;
338:       return at::fft_hfft_symint_out(out, self, n, dim, norm);
339:     };
340:     return wrap(dispatch_fft_hfft_out(_r.tensor(4), _r.tensor(0), _r.toSymIntOptional(1), _r.toInt64(2), _r.stringViewOptional(3)));
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `fft_hfft`, `fft_hfft_symint`. Control-flow branches in this range handle alternative runtime cases or iterative work. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `fft_hfft`, `fft_hfft_symint` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 341-360

```cpp
341:   }
342:   Py_RETURN_NONE;
343:   END_HANDLE_TH_ERRORS
344: }
345: 
346: // fft_hfft2
347: static PyObject * THPVariable_fft_hfft2(PyObject* self_, PyObject* args, PyObject* kwargs)
348: {
349:   HANDLE_TH_ERRORS
350:   static PythonArgParser parser({
351:     "fft_hfft2(Tensor input, SymIntArrayRef[1]? s=None, IntArrayRef[1] dim={-2,-1}, c10::string_view? norm=None, *, Tensor out=None)",
352:   }, /*traceable=*/true);
353: 
354:   ParsedArgs<5> parsed_args;
355:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
356:   if(_r.has_torch_function()) {
357:     return handle_torch_function(_r, nullptr, args, kwargs, THPFFTVariableFunctionsModule, "torch.fft");
358:   }
359:   if (_r.isNone(4)) {
360:     // aten::fft_hfft2(Tensor self, SymInt[1]? s=None, int[1] dim=[-2,-1], str? norm=None) -> Tensor
```

- EN: The main execution path in this span is carried by `THPVariable_fft_hfft2`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_fft_hfft2`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 361-380

```cpp
361: 
362:     auto dispatch_fft_hfft2 = [](const at::Tensor & self, at::OptionalSymIntArrayRef s, at::IntArrayRef dim, ::std::optional<c10::string_view> norm) -> at::Tensor {
363:       pybind11::gil_scoped_release no_gil;
364:       return at::fft_hfft2_symint(self, s, dim, norm);
365:     };
366:     return wrap(dispatch_fft_hfft2(_r.tensor(0), _r.symintlistOptional(1), _r.intlist(2), _r.stringViewOptional(3)));
367:   } else {
368:     // aten::fft_hfft2.out(Tensor self, SymInt[1]? s=None, int[1] dim=[-2,-1], str? norm=None, *, Tensor(a!) out) -> Tensor(a!)
369: 
370:     auto dispatch_fft_hfft2_out = [](at::Tensor out, const at::Tensor & self, at::OptionalSymIntArrayRef s, at::IntArrayRef dim, ::std::optional<c10::string_view> norm) -> at::Tensor {
371:       pybind11::gil_scoped_release no_gil;
372:       return at::fft_hfft2_symint_out(out, self, s, dim, norm);
373:     };
374:     return wrap(dispatch_fft_hfft2_out(_r.tensor(4), _r.tensor(0), _r.symintlistOptional(1), _r.intlist(2), _r.stringViewOptional(3)));
375:   }
376:   Py_RETURN_NONE;
377:   END_HANDLE_TH_ERRORS
378: }
379: 
380: // fft_hfftn
```

- EN: The main execution path in this span is carried by `fft_hfft2_symint`, `wrap`, `fft_hfft2_symint_out`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `fft_hfft2_symint`, `wrap`, `fft_hfft2_symint_out` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 381-400

```cpp
381: static PyObject * THPVariable_fft_hfftn(PyObject* self_, PyObject* args, PyObject* kwargs)
382: {
383:   HANDLE_TH_ERRORS
384:   static PythonArgParser parser({
385:     "fft_hfftn(Tensor input, SymIntArrayRef[1]? s=None, IntArrayRef[1]? dim=None, c10::string_view? norm=None, *, Tensor out=None)",
386:   }, /*traceable=*/true);
387: 
388:   ParsedArgs<5> parsed_args;
389:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
390:   if(_r.has_torch_function()) {
391:     return handle_torch_function(_r, nullptr, args, kwargs, THPFFTVariableFunctionsModule, "torch.fft");
392:   }
393:   if (_r.isNone(4)) {
394:     // aten::fft_hfftn(Tensor self, SymInt[1]? s=None, int[1]? dim=None, str? norm=None) -> Tensor
395: 
396:     auto dispatch_fft_hfftn = [](const at::Tensor & self, at::OptionalSymIntArrayRef s, at::OptionalIntArrayRef dim, ::std::optional<c10::string_view> norm) -> at::Tensor {
397:       pybind11::gil_scoped_release no_gil;
398:       return at::fft_hfftn_symint(self, s, dim, norm);
399:     };
400:     return wrap(dispatch_fft_hfftn(_r.tensor(0), _r.symintlistOptional(1), _r.intlistOptional(2), _r.stringViewOptional(3)));
```

- EN: The main execution path in this span is carried by `THPVariable_fft_hfftn`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_fft_hfftn`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 401-420

```cpp
401:   } else {
402:     // aten::fft_hfftn.out(Tensor self, SymInt[1]? s=None, int[1]? dim=None, str? norm=None, *, Tensor(a!) out) -> Tensor(a!)
403: 
404:     auto dispatch_fft_hfftn_out = [](at::Tensor out, const at::Tensor & self, at::OptionalSymIntArrayRef s, at::OptionalIntArrayRef dim, ::std::optional<c10::string_view> norm) -> at::Tensor {
405:       pybind11::gil_scoped_release no_gil;
406:       return at::fft_hfftn_symint_out(out, self, s, dim, norm);
407:     };
408:     return wrap(dispatch_fft_hfftn_out(_r.tensor(4), _r.tensor(0), _r.symintlistOptional(1), _r.intlistOptional(2), _r.stringViewOptional(3)));
409:   }
410:   Py_RETURN_NONE;
411:   END_HANDLE_TH_ERRORS
412: }
413: 
414: // fft_ifft
415: static PyObject * THPVariable_fft_ifft(PyObject* self_, PyObject* args, PyObject* kwargs)
416: {
417:   HANDLE_TH_ERRORS
418:   static PythonArgParser parser({
419:     "fft_ifft(Tensor input, SymInt? n=None, int64_t dim=-1, c10::string_view? norm=None, *, Tensor out=None)",
420:   }, /*traceable=*/true);
```

- EN: The main execution path in this span is carried by `fft_hfftn_symint_out`, `wrap`, `THPVariable_fft_ifft`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `fft_hfftn_symint_out`, `wrap`, `THPVariable_fft_ifft` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 421-440

```cpp
421: 
422:   ParsedArgs<5> parsed_args;
423:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
424:   if(_r.has_torch_function()) {
425:     return handle_torch_function(_r, nullptr, args, kwargs, THPFFTVariableFunctionsModule, "torch.fft");
426:   }
427:   if (_r.isNone(4)) {
428:     // aten::fft_ifft(Tensor self, SymInt? n=None, int dim=-1, str? norm=None) -> Tensor
429: 
430:     auto dispatch_fft_ifft = [](const at::Tensor & self, ::std::optional<c10::SymInt> n, int64_t dim, ::std::optional<c10::string_view> norm) -> at::Tensor {
431:       pybind11::gil_scoped_release no_gil;
432:       return at::fft_ifft_symint(self, n, dim, norm);
433:     };
434:     return wrap(dispatch_fft_ifft(_r.tensor(0), _r.toSymIntOptional(1), _r.toInt64(2), _r.stringViewOptional(3)));
435:   } else {
436:     // aten::fft_ifft.out(Tensor self, SymInt? n=None, int dim=-1, str? norm=None, *, Tensor(a!) out) -> Tensor(a!)
437: 
438:     auto dispatch_fft_ifft_out = [](at::Tensor out, const at::Tensor & self, ::std::optional<c10::SymInt> n, int64_t dim, ::std::optional<c10::string_view> norm) -> at::Tensor {
439:       pybind11::gil_scoped_release no_gil;
440:       return at::fft_ifft_symint_out(out, self, n, dim, norm);
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `fft_ifft`, `fft_ifft_symint`. Control-flow branches in this range handle alternative runtime cases or iterative work. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `fft_ifft`, `fft_ifft_symint` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 441-460

```cpp
441:     };
442:     return wrap(dispatch_fft_ifft_out(_r.tensor(4), _r.tensor(0), _r.toSymIntOptional(1), _r.toInt64(2), _r.stringViewOptional(3)));
443:   }
444:   Py_RETURN_NONE;
445:   END_HANDLE_TH_ERRORS
446: }
447: 
448: // fft_ifft2
449: static PyObject * THPVariable_fft_ifft2(PyObject* self_, PyObject* args, PyObject* kwargs)
450: {
451:   HANDLE_TH_ERRORS
452:   static PythonArgParser parser({
453:     "fft_ifft2(Tensor input, SymIntArrayRef[1]? s=None, IntArrayRef[1] dim={-2,-1}, c10::string_view? norm=None, *, Tensor out=None)",
454:   }, /*traceable=*/true);
455: 
456:   ParsedArgs<5> parsed_args;
457:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
458:   if(_r.has_torch_function()) {
459:     return handle_torch_function(_r, nullptr, args, kwargs, THPFFTVariableFunctionsModule, "torch.fft");
460:   }
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_fft_ifft2`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_fft_ifft2`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 461-480

```cpp
461:   if (_r.isNone(4)) {
462:     // aten::fft_ifft2(Tensor self, SymInt[1]? s=None, int[1] dim=[-2,-1], str? norm=None) -> Tensor
463: 
464:     auto dispatch_fft_ifft2 = [](const at::Tensor & self, at::OptionalSymIntArrayRef s, at::IntArrayRef dim, ::std::optional<c10::string_view> norm) -> at::Tensor {
465:       pybind11::gil_scoped_release no_gil;
466:       return at::fft_ifft2_symint(self, s, dim, norm);
467:     };
468:     return wrap(dispatch_fft_ifft2(_r.tensor(0), _r.symintlistOptional(1), _r.intlist(2), _r.stringViewOptional(3)));
469:   } else {
470:     // aten::fft_ifft2.out(Tensor self, SymInt[1]? s=None, int[1] dim=[-2,-1], str? norm=None, *, Tensor(a!) out) -> Tensor(a!)
471: 
472:     auto dispatch_fft_ifft2_out = [](at::Tensor out, const at::Tensor & self, at::OptionalSymIntArrayRef s, at::IntArrayRef dim, ::std::optional<c10::string_view> norm) -> at::Tensor {
473:       pybind11::gil_scoped_release no_gil;
474:       return at::fft_ifft2_symint_out(out, self, s, dim, norm);
475:     };
476:     return wrap(dispatch_fft_ifft2_out(_r.tensor(4), _r.tensor(0), _r.symintlistOptional(1), _r.intlist(2), _r.stringViewOptional(3)));
477:   }
478:   Py_RETURN_NONE;
479:   END_HANDLE_TH_ERRORS
480: }
```

- EN: The main execution path in this span is carried by `fft_ifft2`, `fft_ifft2_symint`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `fft_ifft2`, `fft_ifft2_symint`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 481-500

```cpp
481: 
482: // fft_ifftn
483: static PyObject * THPVariable_fft_ifftn(PyObject* self_, PyObject* args, PyObject* kwargs)
484: {
485:   HANDLE_TH_ERRORS
486:   static PythonArgParser parser({
487:     "fft_ifftn(Tensor input, SymIntArrayRef[1]? s=None, IntArrayRef[1]? dim=None, c10::string_view? norm=None, *, Tensor out=None)",
488:   }, /*traceable=*/true);
489: 
490:   ParsedArgs<5> parsed_args;
491:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
492:   if(_r.has_torch_function()) {
493:     return handle_torch_function(_r, nullptr, args, kwargs, THPFFTVariableFunctionsModule, "torch.fft");
494:   }
495:   if (_r.isNone(4)) {
496:     // aten::fft_ifftn(Tensor self, SymInt[1]? s=None, int[1]? dim=None, str? norm=None) -> Tensor
497: 
498:     auto dispatch_fft_ifftn = [](const at::Tensor & self, at::OptionalSymIntArrayRef s, at::OptionalIntArrayRef dim, ::std::optional<c10::string_view> norm) -> at::Tensor {
499:       pybind11::gil_scoped_release no_gil;
500:       return at::fft_ifftn_symint(self, s, dim, norm);
```

- EN: The main execution path in this span is carried by `THPVariable_fft_ifftn`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_fft_ifftn`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 501-520

```cpp
501:     };
502:     return wrap(dispatch_fft_ifftn(_r.tensor(0), _r.symintlistOptional(1), _r.intlistOptional(2), _r.stringViewOptional(3)));
503:   } else {
504:     // aten::fft_ifftn.out(Tensor self, SymInt[1]? s=None, int[1]? dim=None, str? norm=None, *, Tensor(a!) out) -> Tensor(a!)
505: 
506:     auto dispatch_fft_ifftn_out = [](at::Tensor out, const at::Tensor & self, at::OptionalSymIntArrayRef s, at::OptionalIntArrayRef dim, ::std::optional<c10::string_view> norm) -> at::Tensor {
507:       pybind11::gil_scoped_release no_gil;
508:       return at::fft_ifftn_symint_out(out, self, s, dim, norm);
509:     };
510:     return wrap(dispatch_fft_ifftn_out(_r.tensor(4), _r.tensor(0), _r.symintlistOptional(1), _r.intlistOptional(2), _r.stringViewOptional(3)));
511:   }
512:   Py_RETURN_NONE;
513:   END_HANDLE_TH_ERRORS
514: }
515: 
516: // fft_ifftshift
517: static PyObject * THPVariable_fft_ifftshift(PyObject* self_, PyObject* args, PyObject* kwargs)
518: {
519:   HANDLE_TH_ERRORS
520:   static PythonArgParser parser({
```

- EN: The main execution path in this span is carried by `wrap`, `fft_ifftn_symint_out`, `THPVariable_fft_ifftshift`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `wrap`, `fft_ifftn_symint_out`, `THPVariable_fft_ifftshift` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 521-540

```cpp
521:     "fft_ifftshift(Tensor input, IntArrayRef[1]? dim=None)",
522:   }, /*traceable=*/true);
523: 
524:   ParsedArgs<2> parsed_args;
525:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
526:   if(_r.has_torch_function()) {
527:     return handle_torch_function(_r, nullptr, args, kwargs, THPFFTVariableFunctionsModule, "torch.fft");
528:   }
529:   // aten::fft_ifftshift(Tensor self, int[1]? dim=None) -> Tensor
530: 
531:   auto dispatch_fft_ifftshift = [](const at::Tensor & self, at::OptionalIntArrayRef dim) -> at::Tensor {
532:     pybind11::gil_scoped_release no_gil;
533:     return at::fft_ifftshift(self, dim);
534:   };
535:   return wrap(dispatch_fft_ifftshift(_r.tensor(0), _r.intlistOptional(1)));
536:   Py_RETURN_NONE;
537:   END_HANDLE_TH_ERRORS
538: }
539: 
540: // fft_ihfft
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `fft_ifftshift`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `fft_ifftshift`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 541-560

```cpp
541: static PyObject * THPVariable_fft_ihfft(PyObject* self_, PyObject* args, PyObject* kwargs)
542: {
543:   HANDLE_TH_ERRORS
544:   static PythonArgParser parser({
545:     "fft_ihfft(Tensor input, SymInt? n=None, int64_t dim=-1, c10::string_view? norm=None, *, Tensor out=None)",
546:   }, /*traceable=*/true);
547: 
548:   ParsedArgs<5> parsed_args;
549:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
550:   if(_r.has_torch_function()) {
551:     return handle_torch_function(_r, nullptr, args, kwargs, THPFFTVariableFunctionsModule, "torch.fft");
552:   }
553:   if (_r.isNone(4)) {
554:     // aten::fft_ihfft(Tensor self, SymInt? n=None, int dim=-1, str? norm=None) -> Tensor
555: 
556:     auto dispatch_fft_ihfft = [](const at::Tensor & self, ::std::optional<c10::SymInt> n, int64_t dim, ::std::optional<c10::string_view> norm) -> at::Tensor {
557:       pybind11::gil_scoped_release no_gil;
558:       return at::fft_ihfft_symint(self, n, dim, norm);
559:     };
560:     return wrap(dispatch_fft_ihfft(_r.tensor(0), _r.toSymIntOptional(1), _r.toInt64(2), _r.stringViewOptional(3)));
```

- EN: The main execution path in this span is carried by `THPVariable_fft_ihfft`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_fft_ihfft`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 561-580

```cpp
561:   } else {
562:     // aten::fft_ihfft.out(Tensor self, SymInt? n=None, int dim=-1, str? norm=None, *, Tensor(a!) out) -> Tensor(a!)
563: 
564:     auto dispatch_fft_ihfft_out = [](at::Tensor out, const at::Tensor & self, ::std::optional<c10::SymInt> n, int64_t dim, ::std::optional<c10::string_view> norm) -> at::Tensor {
565:       pybind11::gil_scoped_release no_gil;
566:       return at::fft_ihfft_symint_out(out, self, n, dim, norm);
567:     };
568:     return wrap(dispatch_fft_ihfft_out(_r.tensor(4), _r.tensor(0), _r.toSymIntOptional(1), _r.toInt64(2), _r.stringViewOptional(3)));
569:   }
570:   Py_RETURN_NONE;
571:   END_HANDLE_TH_ERRORS
572: }
573: 
574: // fft_ihfft2
575: static PyObject * THPVariable_fft_ihfft2(PyObject* self_, PyObject* args, PyObject* kwargs)
576: {
577:   HANDLE_TH_ERRORS
578:   static PythonArgParser parser({
579:     "fft_ihfft2(Tensor input, SymIntArrayRef[1]? s=None, IntArrayRef[1] dim={-2,-1}, c10::string_view? norm=None, *, Tensor out=None)",
580:   }, /*traceable=*/true);
```

- EN: The main execution path in this span is carried by `fft_ihfft_symint_out`, `wrap`, `THPVariable_fft_ihfft2`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `fft_ihfft_symint_out`, `wrap`, `THPVariable_fft_ihfft2` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 581-600

```cpp
581: 
582:   ParsedArgs<5> parsed_args;
583:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
584:   if(_r.has_torch_function()) {
585:     return handle_torch_function(_r, nullptr, args, kwargs, THPFFTVariableFunctionsModule, "torch.fft");
586:   }
587:   if (_r.isNone(4)) {
588:     // aten::fft_ihfft2(Tensor self, SymInt[1]? s=None, int[1] dim=[-2,-1], str? norm=None) -> Tensor
589: 
590:     auto dispatch_fft_ihfft2 = [](const at::Tensor & self, at::OptionalSymIntArrayRef s, at::IntArrayRef dim, ::std::optional<c10::string_view> norm) -> at::Tensor {
591:       pybind11::gil_scoped_release no_gil;
592:       return at::fft_ihfft2_symint(self, s, dim, norm);
593:     };
594:     return wrap(dispatch_fft_ihfft2(_r.tensor(0), _r.symintlistOptional(1), _r.intlist(2), _r.stringViewOptional(3)));
595:   } else {
596:     // aten::fft_ihfft2.out(Tensor self, SymInt[1]? s=None, int[1] dim=[-2,-1], str? norm=None, *, Tensor(a!) out) -> Tensor(a!)
597: 
598:     auto dispatch_fft_ihfft2_out = [](at::Tensor out, const at::Tensor & self, at::OptionalSymIntArrayRef s, at::IntArrayRef dim, ::std::optional<c10::string_view> norm) -> at::Tensor {
599:       pybind11::gil_scoped_release no_gil;
600:       return at::fft_ihfft2_symint_out(out, self, s, dim, norm);
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `fft_ihfft2`, `fft_ihfft2_symint`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `fft_ihfft2`, `fft_ihfft2_symint` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 601-620

```cpp
601:     };
602:     return wrap(dispatch_fft_ihfft2_out(_r.tensor(4), _r.tensor(0), _r.symintlistOptional(1), _r.intlist(2), _r.stringViewOptional(3)));
603:   }
604:   Py_RETURN_NONE;
605:   END_HANDLE_TH_ERRORS
606: }
607: 
608: // fft_ihfftn
609: static PyObject * THPVariable_fft_ihfftn(PyObject* self_, PyObject* args, PyObject* kwargs)
610: {
611:   HANDLE_TH_ERRORS
612:   static PythonArgParser parser({
613:     "fft_ihfftn(Tensor input, SymIntArrayRef[1]? s=None, IntArrayRef[1]? dim=None, c10::string_view? norm=None, *, Tensor out=None)",
614:   }, /*traceable=*/true);
615: 
616:   ParsedArgs<5> parsed_args;
617:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
618:   if(_r.has_torch_function()) {
619:     return handle_torch_function(_r, nullptr, args, kwargs, THPFFTVariableFunctionsModule, "torch.fft");
620:   }
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_fft_ihfftn`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_fft_ihfftn`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 621-640

```cpp
621:   if (_r.isNone(4)) {
622:     // aten::fft_ihfftn(Tensor self, SymInt[1]? s=None, int[1]? dim=None, str? norm=None) -> Tensor
623: 
624:     auto dispatch_fft_ihfftn = [](const at::Tensor & self, at::OptionalSymIntArrayRef s, at::OptionalIntArrayRef dim, ::std::optional<c10::string_view> norm) -> at::Tensor {
625:       pybind11::gil_scoped_release no_gil;
626:       return at::fft_ihfftn_symint(self, s, dim, norm);
627:     };
628:     return wrap(dispatch_fft_ihfftn(_r.tensor(0), _r.symintlistOptional(1), _r.intlistOptional(2), _r.stringViewOptional(3)));
629:   } else {
630:     // aten::fft_ihfftn.out(Tensor self, SymInt[1]? s=None, int[1]? dim=None, str? norm=None, *, Tensor(a!) out) -> Tensor(a!)
631: 
632:     auto dispatch_fft_ihfftn_out = [](at::Tensor out, const at::Tensor & self, at::OptionalSymIntArrayRef s, at::OptionalIntArrayRef dim, ::std::optional<c10::string_view> norm) -> at::Tensor {
633:       pybind11::gil_scoped_release no_gil;
634:       return at::fft_ihfftn_symint_out(out, self, s, dim, norm);
635:     };
636:     return wrap(dispatch_fft_ihfftn_out(_r.tensor(4), _r.tensor(0), _r.symintlistOptional(1), _r.intlistOptional(2), _r.stringViewOptional(3)));
637:   }
638:   Py_RETURN_NONE;
639:   END_HANDLE_TH_ERRORS
640: }
```

- EN: The main execution path in this span is carried by `fft_ihfftn`, `fft_ihfftn_symint`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `fft_ihfftn`, `fft_ihfftn_symint`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 641-660

```cpp
641: 
642: // fft_irfft
643: static PyObject * THPVariable_fft_irfft(PyObject* self_, PyObject* args, PyObject* kwargs)
644: {
645:   HANDLE_TH_ERRORS
646:   static PythonArgParser parser({
647:     "fft_irfft(Tensor input, SymInt? n=None, int64_t dim=-1, c10::string_view? norm=None, *, Tensor out=None)",
648:   }, /*traceable=*/true);
649: 
650:   ParsedArgs<5> parsed_args;
651:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
652:   if(_r.has_torch_function()) {
653:     return handle_torch_function(_r, nullptr, args, kwargs, THPFFTVariableFunctionsModule, "torch.fft");
654:   }
655:   if (_r.isNone(4)) {
656:     // aten::fft_irfft(Tensor self, SymInt? n=None, int dim=-1, str? norm=None) -> Tensor
657: 
658:     auto dispatch_fft_irfft = [](const at::Tensor & self, ::std::optional<c10::SymInt> n, int64_t dim, ::std::optional<c10::string_view> norm) -> at::Tensor {
659:       pybind11::gil_scoped_release no_gil;
660:       return at::fft_irfft_symint(self, n, dim, norm);
```

- EN: The main execution path in this span is carried by `THPVariable_fft_irfft`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_fft_irfft`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 661-680

```cpp
661:     };
662:     return wrap(dispatch_fft_irfft(_r.tensor(0), _r.toSymIntOptional(1), _r.toInt64(2), _r.stringViewOptional(3)));
663:   } else {
664:     // aten::fft_irfft.out(Tensor self, SymInt? n=None, int dim=-1, str? norm=None, *, Tensor(a!) out) -> Tensor(a!)
665: 
666:     auto dispatch_fft_irfft_out = [](at::Tensor out, const at::Tensor & self, ::std::optional<c10::SymInt> n, int64_t dim, ::std::optional<c10::string_view> norm) -> at::Tensor {
667:       pybind11::gil_scoped_release no_gil;
668:       return at::fft_irfft_symint_out(out, self, n, dim, norm);
669:     };
670:     return wrap(dispatch_fft_irfft_out(_r.tensor(4), _r.tensor(0), _r.toSymIntOptional(1), _r.toInt64(2), _r.stringViewOptional(3)));
671:   }
672:   Py_RETURN_NONE;
673:   END_HANDLE_TH_ERRORS
674: }
675: 
676: // fft_irfft2
677: static PyObject * THPVariable_fft_irfft2(PyObject* self_, PyObject* args, PyObject* kwargs)
678: {
679:   HANDLE_TH_ERRORS
680:   static PythonArgParser parser({
```

- EN: The main execution path in this span is carried by `wrap`, `fft_irfft_symint_out`, `THPVariable_fft_irfft2`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `fft_irfft_symint_out`, `THPVariable_fft_irfft2` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 681-700

```cpp
681:     "fft_irfft2(Tensor input, SymIntArrayRef[1]? s=None, IntArrayRef[1] dim={-2,-1}, c10::string_view? norm=None, *, Tensor out=None)",
682:   }, /*traceable=*/true);
683: 
684:   ParsedArgs<5> parsed_args;
685:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
686:   if(_r.has_torch_function()) {
687:     return handle_torch_function(_r, nullptr, args, kwargs, THPFFTVariableFunctionsModule, "torch.fft");
688:   }
689:   if (_r.isNone(4)) {
690:     // aten::fft_irfft2(Tensor self, SymInt[1]? s=None, int[1] dim=[-2,-1], str? norm=None) -> Tensor
691: 
692:     auto dispatch_fft_irfft2 = [](const at::Tensor & self, at::OptionalSymIntArrayRef s, at::IntArrayRef dim, ::std::optional<c10::string_view> norm) -> at::Tensor {
693:       pybind11::gil_scoped_release no_gil;
694:       return at::fft_irfft2_symint(self, s, dim, norm);
695:     };
696:     return wrap(dispatch_fft_irfft2(_r.tensor(0), _r.symintlistOptional(1), _r.intlist(2), _r.stringViewOptional(3)));
697:   } else {
698:     // aten::fft_irfft2.out(Tensor self, SymInt[1]? s=None, int[1] dim=[-2,-1], str? norm=None, *, Tensor(a!) out) -> Tensor(a!)
699: 
700:     auto dispatch_fft_irfft2_out = [](at::Tensor out, const at::Tensor & self, at::OptionalSymIntArrayRef s, at::IntArrayRef dim, ::std::optional<c10::string_view> norm) -> at::Tensor {
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `fft_irfft2`, `fft_irfft2_symint`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `fft_irfft2`, `fft_irfft2_symint` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 701-720

```cpp
701:       pybind11::gil_scoped_release no_gil;
702:       return at::fft_irfft2_symint_out(out, self, s, dim, norm);
703:     };
704:     return wrap(dispatch_fft_irfft2_out(_r.tensor(4), _r.tensor(0), _r.symintlistOptional(1), _r.intlist(2), _r.stringViewOptional(3)));
705:   }
706:   Py_RETURN_NONE;
707:   END_HANDLE_TH_ERRORS
708: }
709: 
710: // fft_irfftn
711: static PyObject * THPVariable_fft_irfftn(PyObject* self_, PyObject* args, PyObject* kwargs)
712: {
713:   HANDLE_TH_ERRORS
714:   static PythonArgParser parser({
715:     "fft_irfftn(Tensor input, SymIntArrayRef[1]? s=None, IntArrayRef[1]? dim=None, c10::string_view? norm=None, *, Tensor out=None)",
716:   }, /*traceable=*/true);
717: 
718:   ParsedArgs<5> parsed_args;
719:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
720:   if(_r.has_torch_function()) {
```

- EN: The main execution path in this span is carried by `fft_irfft2_symint_out`, `wrap`, `THPVariable_fft_irfftn`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `fft_irfft2_symint_out`, `wrap`, `THPVariable_fft_irfftn` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 721-740

```cpp
721:     return handle_torch_function(_r, nullptr, args, kwargs, THPFFTVariableFunctionsModule, "torch.fft");
722:   }
723:   if (_r.isNone(4)) {
724:     // aten::fft_irfftn(Tensor self, SymInt[1]? s=None, int[1]? dim=None, str? norm=None) -> Tensor
725: 
726:     auto dispatch_fft_irfftn = [](const at::Tensor & self, at::OptionalSymIntArrayRef s, at::OptionalIntArrayRef dim, ::std::optional<c10::string_view> norm) -> at::Tensor {
727:       pybind11::gil_scoped_release no_gil;
728:       return at::fft_irfftn_symint(self, s, dim, norm);
729:     };
730:     return wrap(dispatch_fft_irfftn(_r.tensor(0), _r.symintlistOptional(1), _r.intlistOptional(2), _r.stringViewOptional(3)));
731:   } else {
732:     // aten::fft_irfftn.out(Tensor self, SymInt[1]? s=None, int[1]? dim=None, str? norm=None, *, Tensor(a!) out) -> Tensor(a!)
733: 
734:     auto dispatch_fft_irfftn_out = [](at::Tensor out, const at::Tensor & self, at::OptionalSymIntArrayRef s, at::OptionalIntArrayRef dim, ::std::optional<c10::string_view> norm) -> at::Tensor {
735:       pybind11::gil_scoped_release no_gil;
736:       return at::fft_irfftn_symint_out(out, self, s, dim, norm);
737:     };
738:     return wrap(dispatch_fft_irfftn_out(_r.tensor(4), _r.tensor(0), _r.symintlistOptional(1), _r.intlistOptional(2), _r.stringViewOptional(3)));
739:   }
740:   Py_RETURN_NONE;
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `fft_irfftn`, `fft_irfftn_symint`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `fft_irfftn`, `fft_irfftn_symint` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 741-760

```cpp
741:   END_HANDLE_TH_ERRORS
742: }
743: 
744: // fft_rfft
745: static PyObject * THPVariable_fft_rfft(PyObject* self_, PyObject* args, PyObject* kwargs)
746: {
747:   HANDLE_TH_ERRORS
748:   static PythonArgParser parser({
749:     "fft_rfft(Tensor input, SymInt? n=None, int64_t dim=-1, c10::string_view? norm=None, *, Tensor out=None)",
750:   }, /*traceable=*/true);
751: 
752:   ParsedArgs<5> parsed_args;
753:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
754:   if(_r.has_torch_function()) {
755:     return handle_torch_function(_r, nullptr, args, kwargs, THPFFTVariableFunctionsModule, "torch.fft");
756:   }
757:   if (_r.isNone(4)) {
758:     // aten::fft_rfft(Tensor self, SymInt? n=None, int dim=-1, str? norm=None) -> Tensor
759: 
760:     auto dispatch_fft_rfft = [](const at::Tensor & self, ::std::optional<c10::SymInt> n, int64_t dim, ::std::optional<c10::string_view> norm) -> at::Tensor {
```

- EN: The main execution path in this span is carried by `THPVariable_fft_rfft`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_fft_rfft`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 761-780

```cpp
761:       pybind11::gil_scoped_release no_gil;
762:       return at::fft_rfft_symint(self, n, dim, norm);
763:     };
764:     return wrap(dispatch_fft_rfft(_r.tensor(0), _r.toSymIntOptional(1), _r.toInt64(2), _r.stringViewOptional(3)));
765:   } else {
766:     // aten::fft_rfft.out(Tensor self, SymInt? n=None, int dim=-1, str? norm=None, *, Tensor(a!) out) -> Tensor(a!)
767: 
768:     auto dispatch_fft_rfft_out = [](at::Tensor out, const at::Tensor & self, ::std::optional<c10::SymInt> n, int64_t dim, ::std::optional<c10::string_view> norm) -> at::Tensor {
769:       pybind11::gil_scoped_release no_gil;
770:       return at::fft_rfft_symint_out(out, self, n, dim, norm);
771:     };
772:     return wrap(dispatch_fft_rfft_out(_r.tensor(4), _r.tensor(0), _r.toSymIntOptional(1), _r.toInt64(2), _r.stringViewOptional(3)));
773:   }
774:   Py_RETURN_NONE;
775:   END_HANDLE_TH_ERRORS
776: }
777: 
778: // fft_rfft2
779: static PyObject * THPVariable_fft_rfft2(PyObject* self_, PyObject* args, PyObject* kwargs)
780: {
```

- EN: The main execution path in this span is carried by `fft_rfft_symint`, `wrap`, `fft_rfft_symint_out`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `fft_rfft_symint`, `wrap`, `fft_rfft_symint_out` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 781-800

```cpp
781:   HANDLE_TH_ERRORS
782:   static PythonArgParser parser({
783:     "fft_rfft2(Tensor input, SymIntArrayRef[1]? s=None, IntArrayRef[1] dim={-2,-1}, c10::string_view? norm=None, *, Tensor out=None)",
784:   }, /*traceable=*/true);
785: 
786:   ParsedArgs<5> parsed_args;
787:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
788:   if(_r.has_torch_function()) {
789:     return handle_torch_function(_r, nullptr, args, kwargs, THPFFTVariableFunctionsModule, "torch.fft");
790:   }
791:   if (_r.isNone(4)) {
792:     // aten::fft_rfft2(Tensor self, SymInt[1]? s=None, int[1] dim=[-2,-1], str? norm=None) -> Tensor
793: 
794:     auto dispatch_fft_rfft2 = [](const at::Tensor & self, at::OptionalSymIntArrayRef s, at::IntArrayRef dim, ::std::optional<c10::string_view> norm) -> at::Tensor {
795:       pybind11::gil_scoped_release no_gil;
796:       return at::fft_rfft2_symint(self, s, dim, norm);
797:     };
798:     return wrap(dispatch_fft_rfft2(_r.tensor(0), _r.symintlistOptional(1), _r.intlist(2), _r.stringViewOptional(3)));
799:   } else {
800:     // aten::fft_rfft2.out(Tensor self, SymInt[1]? s=None, int[1] dim=[-2,-1], str? norm=None, *, Tensor(a!) out) -> Tensor(a!)
```

- EN: The main execution path in this span is carried by `parser`, `handle_torch_function`, `fft_rfft2`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `parser`, `handle_torch_function`, `fft_rfft2` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 801-820

```cpp
801: 
802:     auto dispatch_fft_rfft2_out = [](at::Tensor out, const at::Tensor & self, at::OptionalSymIntArrayRef s, at::IntArrayRef dim, ::std::optional<c10::string_view> norm) -> at::Tensor {
803:       pybind11::gil_scoped_release no_gil;
804:       return at::fft_rfft2_symint_out(out, self, s, dim, norm);
805:     };
806:     return wrap(dispatch_fft_rfft2_out(_r.tensor(4), _r.tensor(0), _r.symintlistOptional(1), _r.intlist(2), _r.stringViewOptional(3)));
807:   }
808:   Py_RETURN_NONE;
809:   END_HANDLE_TH_ERRORS
810: }
811: 
812: // fft_rfftfreq
813: static PyObject * THPVariable_fft_rfftfreq(PyObject* self_, PyObject* args, PyObject* kwargs)
814: {
815:   HANDLE_TH_ERRORS
816:   static PythonArgParser parser({
817:     "fft_rfftfreq(int64_t n, double d=1.0, *, Tensor out=None, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=False, bool? requires_grad=False)",
818:   }, /*traceable=*/true);
819: 
820:   ParsedArgs<8> parsed_args;
```

- EN: The main execution path in this span is carried by `fft_rfft2_symint_out`, `wrap`, `THPVariable_fft_rfftfreq`. Autograd-related metadata or graph connectivity is updated here. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `fft_rfft2_symint_out`, `wrap`, `THPVariable_fft_rfftfreq` 等函数/方法承载。 这里会更新与自动求导相关的元数据或计算图连接关系。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 821-840

```cpp
821:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
822:   if(_r.has_torch_function()) {
823:     return handle_torch_function(_r, nullptr, args, kwargs, THPFFTVariableFunctionsModule, "torch.fft");
824:   }
825:   if (_r.isNone(2)) {
826:     // aten::fft_rfftfreq(int n, float d=1.0, *, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=None) -> Tensor
827:     const auto options = TensorOptions()
828:         .dtype(_r.scalartypeOptional(3))
829:         .device(_r.deviceWithDefault(5, torch::tensors::get_default_device()))
830:         .layout(_r.layoutOptional(4))
831:         .requires_grad(_r.toBool(7))
832:         .pinned_memory(_r.toBool(6));
833:     torch::utils::maybe_initialize_device(options);
834: 
835:     auto dispatch_fft_rfftfreq = [](int64_t n, double d, at::TensorOptions options) -> at::Tensor {
836:       pybind11::gil_scoped_release no_gil;
837:       return torch::fft_rfftfreq(n, d, options);
838:     };
839:     return wrap(dispatch_fft_rfftfreq(_r.toInt64(0), _r.toDouble(1), options));
840:   } else {
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `fft_rfftfreq`, `TensorOptions`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `fft_rfftfreq`, `TensorOptions` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 841-860

```cpp
841:     // aten::fft_rfftfreq.out(int n, float d=1.0, *, Tensor(a!) out) -> Tensor(a!)
842:     check_out_type_matches(_r.tensor(2), _r.scalartypeOptional(3),
843:                            _r.isNone(3), _r.layoutOptional(4),
844:                            _r.deviceWithDefault(5, torch::tensors::get_default_device()), _r.isNone(5));
845: 
846:     auto dispatch_fft_rfftfreq_out = [](at::Tensor out, int64_t n, double d) -> at::Tensor {
847:       pybind11::gil_scoped_release no_gil;
848:       return at::fft_rfftfreq_out(out, n, d);
849:     };
850:     return wrap(dispatch_fft_rfftfreq_out(_r.tensor(2), _r.toInt64(0), _r.toDouble(1)).set_requires_grad(_r.toBool(7)));
851:   }
852:   Py_RETURN_NONE;
853:   END_HANDLE_TH_ERRORS
854: }
855: 
856: // fft_rfftn
857: static PyObject * THPVariable_fft_rfftn(PyObject* self_, PyObject* args, PyObject* kwargs)
858: {
859:   HANDLE_TH_ERRORS
860:   static PythonArgParser parser({
```

- EN: The main execution path in this span is carried by `check_out_type_matches`, `fft_rfftfreq_out`, `wrap`. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `check_out_type_matches`, `fft_rfftfreq_out`, `wrap` 等函数/方法承载。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 861-880

```cpp
861:     "fft_rfftn(Tensor input, SymIntArrayRef[1]? s=None, IntArrayRef[1]? dim=None, c10::string_view? norm=None, *, Tensor out=None)",
862:   }, /*traceable=*/true);
863: 
864:   ParsedArgs<5> parsed_args;
865:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
866:   if(_r.has_torch_function()) {
867:     return handle_torch_function(_r, nullptr, args, kwargs, THPFFTVariableFunctionsModule, "torch.fft");
868:   }
869:   if (_r.isNone(4)) {
870:     // aten::fft_rfftn(Tensor self, SymInt[1]? s=None, int[1]? dim=None, str? norm=None) -> Tensor
871: 
872:     auto dispatch_fft_rfftn = [](const at::Tensor & self, at::OptionalSymIntArrayRef s, at::OptionalIntArrayRef dim, ::std::optional<c10::string_view> norm) -> at::Tensor {
873:       pybind11::gil_scoped_release no_gil;
874:       return at::fft_rfftn_symint(self, s, dim, norm);
875:     };
876:     return wrap(dispatch_fft_rfftn(_r.tensor(0), _r.symintlistOptional(1), _r.intlistOptional(2), _r.stringViewOptional(3)));
877:   } else {
878:     // aten::fft_rfftn.out(Tensor self, SymInt[1]? s=None, int[1]? dim=None, str? norm=None, *, Tensor(a!) out) -> Tensor(a!)
879: 
880:     auto dispatch_fft_rfftn_out = [](at::Tensor out, const at::Tensor & self, at::OptionalSymIntArrayRef s, at::OptionalIntArrayRef dim, ::std::optional<c10::string_view> norm) -> at::Tensor {
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `fft_rfftn`, `fft_rfftn_symint`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `fft_rfftn`, `fft_rfftn_symint` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 881-890

```cpp
881:       pybind11::gil_scoped_release no_gil;
882:       return at::fft_rfftn_symint_out(out, self, s, dim, norm);
883:     };
884:     return wrap(dispatch_fft_rfftn_out(_r.tensor(4), _r.tensor(0), _r.symintlistOptional(1), _r.intlistOptional(2), _r.stringViewOptional(3)));
885:   }
886:   Py_RETURN_NONE;
887:   END_HANDLE_TH_ERRORS
888: }
889: 
890: } // namespace torch::autograd
```

- EN: They also open, refine, or close the relevant C++ namespace scope. The main execution path in this span is carried by `fft_rfftn_symint_out`, `wrap`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 它们还会打开、细化或关闭相应的 C++ 命名空间作用域。 这一段的主要执行路径由 `fft_rfftn_symint_out`, `wrap` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。

## Key Concepts / 关键概念
- Autograd graph and gradient metadata / 自动求导图与梯度元数据
- Generated code patterns / 生成代码模式
- Gradient bookkeeping and autograd history / 梯度记录与自动求导历史
- Python/C++ binding boundary / Python/C++ 绑定边界
- Tensor/value representation management / 张量/数值表示管理
- Primary symbol `THPVariable_fft_fft` / 核心符号 `THPVariable_fft_fft`

## Dependencies / 依赖关系
- Direct includes / 直接包含: `torch/csrc/Device.h`, `torch/csrc/DynamicTypes.h`, `torch/csrc/Exceptions.h`, `torch/csrc/autograd/python_fft_functions.h`, `torch/csrc/autograd/generated/python_return_types.h`, `torch/csrc/autograd/python_variable.h`, `torch/csrc/autograd/utils/wrap_outputs.h`, `torch/csrc/autograd/utils/python_arg_parsing.h`, `torch/csrc/autograd/generated/variable_factories.h`, `torch/csrc/utils/out_types.h`
- Include roots / 头文件根模块: `ATen`, `torch`
- Key symbols / 关键符号: `THPVariable_fft_fft`, `THPVariable_fft_fft2`, `THPVariable_fft_fftfreq`, `THPVariable_fft_fftn`, `THPVariable_fft_fftshift`, `THPVariable_fft_hfft`, `THPVariable_fft_hfft2`, `THPVariable_fft_hfftn`, `THPVariable_fft_ifft`, `THPVariable_fft_ifft2`
- Related subsystems / 相关子系统: Autograd / 自动求导, ATen operator layer / ATen 算子层, c10 core utilities / c10 核心工具, Python binding layer / Python 绑定层
