# python_special_functions.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/autograd/generated/python_special_functions.cpp`
- Repository: `/root/xw/pytorch`
- Purpose (EN): Defines generated backward-function classes and helpers used by the autograd engine.
- 目的 (CN): 定义自动求导引擎使用的生成式反向函数类与辅助逻辑。
- Lines: 2801
- Language: C++ / C++

## Line-by-Line Analysis / 逐行分析

- Note / 说明: This file is generated or mechanically expanded, so the analysis groups contiguous line ranges rather than paraphrasing every repeated wrapper individually. / 该文件为生成代码或机械展开代码，因此分析按连续行范围组织，而不是逐个重复改写每个封装模板。
### Lines 1-40

```cpp
 1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
 2: // @generated from ../tools/autograd/templates/python_special_functions.cpp
 3: 
 4: #include "torch/csrc/Device.h"
 5: #include "torch/csrc/DynamicTypes.h"
 6: #include "torch/csrc/Exceptions.h"
 7: #include "torch/csrc/autograd/python_special_functions.h"
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
19: #ifndef AT_PER_OPERATOR_HEADERS
20: #include <ATen/Functions.h>
21: #else
22: #include <ATen/ops/special_airy_ai.h>
23: #include <ATen/ops/special_bessel_j0.h>
24: #include <ATen/ops/special_bessel_j1.h>
25: #include <ATen/ops/special_bessel_y0.h>
26: #include <ATen/ops/special_bessel_y1.h>
27: #include <ATen/ops/special_chebyshev_polynomial_t.h>
28: #include <ATen/ops/special_chebyshev_polynomial_u.h>
29: #include <ATen/ops/special_chebyshev_polynomial_v.h>
30: #include <ATen/ops/special_chebyshev_polynomial_w.h>
31: #include <ATen/ops/special_digamma.h>
32: #include <ATen/ops/special_entr.h>
33: #include <ATen/ops/special_erf.h>
34: #include <ATen/ops/special_erfc.h>
35: #include <ATen/ops/special_erfcx.h>
36: #include <ATen/ops/special_erfinv.h>
37: #include <ATen/ops/special_exp2.h>
38: #include <ATen/ops/special_expit.h>
39: #include <ATen/ops/special_expm1.h>
40: #include <ATen/ops/special_gammainc.h>
```

- EN: These lines pull in dependencies such as `torch/csrc/Device.h`, `torch/csrc/DynamicTypes.h`, `torch/csrc/Exceptions.h`, establishing the headers needed by the implementation. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这些行引入了依赖，例如 `torch/csrc/Device.h`, `torch/csrc/DynamicTypes.h`, `torch/csrc/Exceptions.h`，为后续实现建立所需的头文件基础。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 41-80

```cpp
41: #include <ATen/ops/special_gammaincc.h>
42: #include <ATen/ops/special_gammaln.h>
43: #include <ATen/ops/special_hermite_polynomial_h.h>
44: #include <ATen/ops/special_hermite_polynomial_he.h>
45: #include <ATen/ops/special_i0.h>
46: #include <ATen/ops/special_i0e.h>
47: #include <ATen/ops/special_i1.h>
48: #include <ATen/ops/special_i1e.h>
49: #include <ATen/ops/special_laguerre_polynomial_l.h>
50: #include <ATen/ops/special_legendre_polynomial_p.h>
51: #include <ATen/ops/special_log1p.h>
52: #include <ATen/ops/special_log_ndtr.h>
53: #include <ATen/ops/special_log_softmax.h>
54: #include <ATen/ops/special_logit.h>
55: #include <ATen/ops/special_logsumexp.h>
56: #include <ATen/ops/special_modified_bessel_i0.h>
57: #include <ATen/ops/special_modified_bessel_i1.h>
58: #include <ATen/ops/special_modified_bessel_k0.h>
59: #include <ATen/ops/special_modified_bessel_k1.h>
60: #include <ATen/ops/special_multigammaln.h>
61: #include <ATen/ops/special_ndtr.h>
62: #include <ATen/ops/special_ndtri.h>
63: #include <ATen/ops/special_polygamma.h>
64: #include <ATen/ops/special_psi.h>
65: #include <ATen/ops/special_round.h>
66: #include <ATen/ops/special_scaled_modified_bessel_k0.h>
67: #include <ATen/ops/special_scaled_modified_bessel_k1.h>
68: #include <ATen/ops/special_shifted_chebyshev_polynomial_t.h>
69: #include <ATen/ops/special_shifted_chebyshev_polynomial_u.h>
70: #include <ATen/ops/special_shifted_chebyshev_polynomial_v.h>
71: #include <ATen/ops/special_shifted_chebyshev_polynomial_w.h>
72: #include <ATen/ops/special_sinc.h>
73: #include <ATen/ops/special_softmax.h>
74: #include <ATen/ops/special_spherical_bessel_j0.h>
75: #include <ATen/ops/special_xlog1py.h>
76: #include <ATen/ops/special_xlogy.h>
77: #include <ATen/ops/special_zeta.h>
78: #endif
79: 
80: using at::Tensor;
```

- EN: These lines pull in dependencies such as `ATen/ops/special_gammaincc.h`, `ATen/ops/special_gammaln.h`, `ATen/ops/special_hermite_polynomial_h.h`, establishing the headers needed by the implementation.
- CN: 这些行引入了依赖，例如 `ATen/ops/special_gammaincc.h`, `ATen/ops/special_gammaln.h`, `ATen/ops/special_hermite_polynomial_h.h`，为后续实现建立所需的头文件基础。
### Lines 81-120

```cpp
 81: using at::Device;
 82: using at::Layout;
 83: using at::Scalar;
 84: using at::ScalarType;
 85: using at::Backend;
 86: using at::OptionalDeviceGuard;
 87: using at::DeviceGuard;
 88: using at::TensorOptions;
 89: using at::IntArrayRef;
 90: using at::Generator;
 91: using at::TensorList;
 92: using at::Dimname;
 93: using at::DimnameList;
 94: 
 95: using torch::utils::check_out_type_matches;
 96: using namespace torch::autograd::utils;
 97: 
 98: namespace torch::autograd {
 99: 
100: // generated forward declarations start here
101: 
102: static PyObject * THPVariable_special_airy_ai(PyObject* self_, PyObject* args, PyObject* kwargs);
103: static PyObject * THPVariable_special_bessel_j0(PyObject* self_, PyObject* args, PyObject* kwargs);
104: static PyObject * THPVariable_special_bessel_j1(PyObject* self_, PyObject* args, PyObject* kwargs);
105: static PyObject * THPVariable_special_bessel_y0(PyObject* self_, PyObject* args, PyObject* kwargs);
106: static PyObject * THPVariable_special_bessel_y1(PyObject* self_, PyObject* args, PyObject* kwargs);
107: static PyObject * THPVariable_special_chebyshev_polynomial_t(PyObject* self_, PyObject* args, PyObject* kwargs);
108: static PyObject * THPVariable_special_chebyshev_polynomial_u(PyObject* self_, PyObject* args, PyObject* kwargs);
109: static PyObject * THPVariable_special_chebyshev_polynomial_v(PyObject* self_, PyObject* args, PyObject* kwargs);
110: static PyObject * THPVariable_special_chebyshev_polynomial_w(PyObject* self_, PyObject* args, PyObject* kwargs);
111: static PyObject * THPVariable_special_digamma(PyObject* self_, PyObject* args, PyObject* kwargs);
112: static PyObject * THPVariable_special_entr(PyObject* self_, PyObject* args, PyObject* kwargs);
113: static PyObject * THPVariable_special_erf(PyObject* self_, PyObject* args, PyObject* kwargs);
114: static PyObject * THPVariable_special_erfc(PyObject* self_, PyObject* args, PyObject* kwargs);
115: static PyObject * THPVariable_special_erfcx(PyObject* self_, PyObject* args, PyObject* kwargs);
116: static PyObject * THPVariable_special_erfinv(PyObject* self_, PyObject* args, PyObject* kwargs);
117: static PyObject * THPVariable_special_exp2(PyObject* self_, PyObject* args, PyObject* kwargs);
118: static PyObject * THPVariable_special_expit(PyObject* self_, PyObject* args, PyObject* kwargs);
119: static PyObject * THPVariable_special_expm1(PyObject* self_, PyObject* args, PyObject* kwargs);
120: static PyObject * THPVariable_special_gammainc(PyObject* self_, PyObject* args, PyObject* kwargs);
```

- EN: They also open, refine, or close the relevant C++ namespace scope. The main execution path in this span is carried by `THPVariable_special_airy_ai`, `THPVariable_special_bessel_j0`, `THPVariable_special_bessel_j1`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 它们还会打开、细化或关闭相应的 C++ 命名空间作用域。 这一段的主要执行路径由 `THPVariable_special_airy_ai`, `THPVariable_special_bessel_j0`, `THPVariable_special_bessel_j1` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 121-160

```cpp
121: static PyObject * THPVariable_special_gammaincc(PyObject* self_, PyObject* args, PyObject* kwargs);
122: static PyObject * THPVariable_special_gammaln(PyObject* self_, PyObject* args, PyObject* kwargs);
123: static PyObject * THPVariable_special_hermite_polynomial_h(PyObject* self_, PyObject* args, PyObject* kwargs);
124: static PyObject * THPVariable_special_hermite_polynomial_he(PyObject* self_, PyObject* args, PyObject* kwargs);
125: static PyObject * THPVariable_special_i0(PyObject* self_, PyObject* args, PyObject* kwargs);
126: static PyObject * THPVariable_special_i0e(PyObject* self_, PyObject* args, PyObject* kwargs);
127: static PyObject * THPVariable_special_i1(PyObject* self_, PyObject* args, PyObject* kwargs);
128: static PyObject * THPVariable_special_i1e(PyObject* self_, PyObject* args, PyObject* kwargs);
129: static PyObject * THPVariable_special_laguerre_polynomial_l(PyObject* self_, PyObject* args, PyObject* kwargs);
130: static PyObject * THPVariable_special_legendre_polynomial_p(PyObject* self_, PyObject* args, PyObject* kwargs);
131: static PyObject * THPVariable_special_log1p(PyObject* self_, PyObject* args, PyObject* kwargs);
132: static PyObject * THPVariable_special_log_ndtr(PyObject* self_, PyObject* args, PyObject* kwargs);
133: static PyObject * THPVariable_special_log_softmax(PyObject* self_, PyObject* args, PyObject* kwargs);
134: static PyObject * THPVariable_special_logit(PyObject* self_, PyObject* args, PyObject* kwargs);
135: static PyObject * THPVariable_special_logsumexp(PyObject* self_, PyObject* args, PyObject* kwargs);
136: static PyObject * THPVariable_special_modified_bessel_i0(PyObject* self_, PyObject* args, PyObject* kwargs);
137: static PyObject * THPVariable_special_modified_bessel_i1(PyObject* self_, PyObject* args, PyObject* kwargs);
138: static PyObject * THPVariable_special_modified_bessel_k0(PyObject* self_, PyObject* args, PyObject* kwargs);
139: static PyObject * THPVariable_special_modified_bessel_k1(PyObject* self_, PyObject* args, PyObject* kwargs);
140: static PyObject * THPVariable_special_multigammaln(PyObject* self_, PyObject* args, PyObject* kwargs);
141: static PyObject * THPVariable_special_ndtr(PyObject* self_, PyObject* args, PyObject* kwargs);
142: static PyObject * THPVariable_special_ndtri(PyObject* self_, PyObject* args, PyObject* kwargs);
143: static PyObject * THPVariable_special_polygamma(PyObject* self_, PyObject* args, PyObject* kwargs);
144: static PyObject * THPVariable_special_psi(PyObject* self_, PyObject* args, PyObject* kwargs);
145: static PyObject * THPVariable_special_round(PyObject* self_, PyObject* args, PyObject* kwargs);
146: static PyObject * THPVariable_special_scaled_modified_bessel_k0(PyObject* self_, PyObject* args, PyObject* kwargs);
147: static PyObject * THPVariable_special_scaled_modified_bessel_k1(PyObject* self_, PyObject* args, PyObject* kwargs);
148: static PyObject * THPVariable_special_shifted_chebyshev_polynomial_t(PyObject* self_, PyObject* args, PyObject* kwargs);
149: static PyObject * THPVariable_special_shifted_chebyshev_polynomial_u(PyObject* self_, PyObject* args, PyObject* kwargs);
150: static PyObject * THPVariable_special_shifted_chebyshev_polynomial_v(PyObject* self_, PyObject* args, PyObject* kwargs);
151: static PyObject * THPVariable_special_shifted_chebyshev_polynomial_w(PyObject* self_, PyObject* args, PyObject* kwargs);
152: static PyObject * THPVariable_special_sinc(PyObject* self_, PyObject* args, PyObject* kwargs);
153: static PyObject * THPVariable_special_softmax(PyObject* self_, PyObject* args, PyObject* kwargs);
154: static PyObject * THPVariable_special_spherical_bessel_j0(PyObject* self_, PyObject* args, PyObject* kwargs);
155: static PyObject * THPVariable_special_xlog1py(PyObject* self_, PyObject* args, PyObject* kwargs);
156: static PyObject * THPVariable_special_xlogy(PyObject* self_, PyObject* args, PyObject* kwargs);
157: static PyObject * THPVariable_special_zeta(PyObject* self_, PyObject* args, PyObject* kwargs);
158: 
159: static PyMethodDef special_functions[] = {
160:   {"special_airy_ai", castPyCFunctionWithKeywords(THPVariable_special_airy_ai), METH_VARARGS | METH_KEYWORDS, nullptr},
```

- EN: The main execution path in this span is carried by `THPVariable_special_gammaincc`, `THPVariable_special_gammaln`, `THPVariable_special_hermite_polynomial_h`. Because this is generated binding code, the span mostly registers or forwards APIs into a mechanically produced Python-facing surface. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_special_gammaincc`, `THPVariable_special_gammaln`, `THPVariable_special_hermite_polynomial_h` 等函数/方法承载。 由于这是生成的绑定代码，这一段主要是在机械化生成的 Python 接口层上完成 API 注册或转发。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 161-200

```cpp
161:   {"special_bessel_j0", castPyCFunctionWithKeywords(THPVariable_special_bessel_j0), METH_VARARGS | METH_KEYWORDS, nullptr},
162:   {"special_bessel_j1", castPyCFunctionWithKeywords(THPVariable_special_bessel_j1), METH_VARARGS | METH_KEYWORDS, nullptr},
163:   {"special_bessel_y0", castPyCFunctionWithKeywords(THPVariable_special_bessel_y0), METH_VARARGS | METH_KEYWORDS, nullptr},
164:   {"special_bessel_y1", castPyCFunctionWithKeywords(THPVariable_special_bessel_y1), METH_VARARGS | METH_KEYWORDS, nullptr},
165:   {"special_chebyshev_polynomial_t", castPyCFunctionWithKeywords(THPVariable_special_chebyshev_polynomial_t), METH_VARARGS | METH_KEYWORDS, nullptr},
166:   {"special_chebyshev_polynomial_u", castPyCFunctionWithKeywords(THPVariable_special_chebyshev_polynomial_u), METH_VARARGS | METH_KEYWORDS, nullptr},
167:   {"special_chebyshev_polynomial_v", castPyCFunctionWithKeywords(THPVariable_special_chebyshev_polynomial_v), METH_VARARGS | METH_KEYWORDS, nullptr},
168:   {"special_chebyshev_polynomial_w", castPyCFunctionWithKeywords(THPVariable_special_chebyshev_polynomial_w), METH_VARARGS | METH_KEYWORDS, nullptr},
169:   {"special_digamma", castPyCFunctionWithKeywords(THPVariable_special_digamma), METH_VARARGS | METH_KEYWORDS, nullptr},
170:   {"special_entr", castPyCFunctionWithKeywords(THPVariable_special_entr), METH_VARARGS | METH_KEYWORDS, nullptr},
171:   {"special_erf", castPyCFunctionWithKeywords(THPVariable_special_erf), METH_VARARGS | METH_KEYWORDS, nullptr},
172:   {"special_erfc", castPyCFunctionWithKeywords(THPVariable_special_erfc), METH_VARARGS | METH_KEYWORDS, nullptr},
173:   {"special_erfcx", castPyCFunctionWithKeywords(THPVariable_special_erfcx), METH_VARARGS | METH_KEYWORDS, nullptr},
174:   {"special_erfinv", castPyCFunctionWithKeywords(THPVariable_special_erfinv), METH_VARARGS | METH_KEYWORDS, nullptr},
175:   {"special_exp2", castPyCFunctionWithKeywords(THPVariable_special_exp2), METH_VARARGS | METH_KEYWORDS, nullptr},
176:   {"special_expit", castPyCFunctionWithKeywords(THPVariable_special_expit), METH_VARARGS | METH_KEYWORDS, nullptr},
177:   {"special_expm1", castPyCFunctionWithKeywords(THPVariable_special_expm1), METH_VARARGS | METH_KEYWORDS, nullptr},
178:   {"special_gammainc", castPyCFunctionWithKeywords(THPVariable_special_gammainc), METH_VARARGS | METH_KEYWORDS, nullptr},
179:   {"special_gammaincc", castPyCFunctionWithKeywords(THPVariable_special_gammaincc), METH_VARARGS | METH_KEYWORDS, nullptr},
180:   {"special_gammaln", castPyCFunctionWithKeywords(THPVariable_special_gammaln), METH_VARARGS | METH_KEYWORDS, nullptr},
181:   {"special_hermite_polynomial_h", castPyCFunctionWithKeywords(THPVariable_special_hermite_polynomial_h), METH_VARARGS | METH_KEYWORDS, nullptr},
182:   {"special_hermite_polynomial_he", castPyCFunctionWithKeywords(THPVariable_special_hermite_polynomial_he), METH_VARARGS | METH_KEYWORDS, nullptr},
183:   {"special_i0", castPyCFunctionWithKeywords(THPVariable_special_i0), METH_VARARGS | METH_KEYWORDS, nullptr},
184:   {"special_i0e", castPyCFunctionWithKeywords(THPVariable_special_i0e), METH_VARARGS | METH_KEYWORDS, nullptr},
185:   {"special_i1", castPyCFunctionWithKeywords(THPVariable_special_i1), METH_VARARGS | METH_KEYWORDS, nullptr},
186:   {"special_i1e", castPyCFunctionWithKeywords(THPVariable_special_i1e), METH_VARARGS | METH_KEYWORDS, nullptr},
187:   {"special_laguerre_polynomial_l", castPyCFunctionWithKeywords(THPVariable_special_laguerre_polynomial_l), METH_VARARGS | METH_KEYWORDS, nullptr},
188:   {"special_legendre_polynomial_p", castPyCFunctionWithKeywords(THPVariable_special_legendre_polynomial_p), METH_VARARGS | METH_KEYWORDS, nullptr},
189:   {"special_log1p", castPyCFunctionWithKeywords(THPVariable_special_log1p), METH_VARARGS | METH_KEYWORDS, nullptr},
190:   {"special_log_ndtr", castPyCFunctionWithKeywords(THPVariable_special_log_ndtr), METH_VARARGS | METH_KEYWORDS, nullptr},
191:   {"special_log_softmax", castPyCFunctionWithKeywords(THPVariable_special_log_softmax), METH_VARARGS | METH_KEYWORDS, nullptr},
192:   {"special_logit", castPyCFunctionWithKeywords(THPVariable_special_logit), METH_VARARGS | METH_KEYWORDS, nullptr},
193:   {"special_logsumexp", castPyCFunctionWithKeywords(THPVariable_special_logsumexp), METH_VARARGS | METH_KEYWORDS, nullptr},
194:   {"special_modified_bessel_i0", castPyCFunctionWithKeywords(THPVariable_special_modified_bessel_i0), METH_VARARGS | METH_KEYWORDS, nullptr},
195:   {"special_modified_bessel_i1", castPyCFunctionWithKeywords(THPVariable_special_modified_bessel_i1), METH_VARARGS | METH_KEYWORDS, nullptr},
196:   {"special_modified_bessel_k0", castPyCFunctionWithKeywords(THPVariable_special_modified_bessel_k0), METH_VARARGS | METH_KEYWORDS, nullptr},
197:   {"special_modified_bessel_k1", castPyCFunctionWithKeywords(THPVariable_special_modified_bessel_k1), METH_VARARGS | METH_KEYWORDS, nullptr},
198:   {"special_multigammaln", castPyCFunctionWithKeywords(THPVariable_special_multigammaln), METH_VARARGS | METH_KEYWORDS, nullptr},
199:   {"special_ndtr", castPyCFunctionWithKeywords(THPVariable_special_ndtr), METH_VARARGS | METH_KEYWORDS, nullptr},
200:   {"special_ndtri", castPyCFunctionWithKeywords(THPVariable_special_ndtri), METH_VARARGS | METH_KEYWORDS, nullptr},
```

- EN: The main execution path in this span is carried by `castPyCFunctionWithKeywords`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `castPyCFunctionWithKeywords` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 201-240

```cpp
201:   {"special_polygamma", castPyCFunctionWithKeywords(THPVariable_special_polygamma), METH_VARARGS | METH_KEYWORDS, nullptr},
202:   {"special_psi", castPyCFunctionWithKeywords(THPVariable_special_psi), METH_VARARGS | METH_KEYWORDS, nullptr},
203:   {"special_round", castPyCFunctionWithKeywords(THPVariable_special_round), METH_VARARGS | METH_KEYWORDS, nullptr},
204:   {"special_scaled_modified_bessel_k0", castPyCFunctionWithKeywords(THPVariable_special_scaled_modified_bessel_k0), METH_VARARGS | METH_KEYWORDS, nullptr},
205:   {"special_scaled_modified_bessel_k1", castPyCFunctionWithKeywords(THPVariable_special_scaled_modified_bessel_k1), METH_VARARGS | METH_KEYWORDS, nullptr},
206:   {"special_shifted_chebyshev_polynomial_t", castPyCFunctionWithKeywords(THPVariable_special_shifted_chebyshev_polynomial_t), METH_VARARGS | METH_KEYWORDS, nullptr},
207:   {"special_shifted_chebyshev_polynomial_u", castPyCFunctionWithKeywords(THPVariable_special_shifted_chebyshev_polynomial_u), METH_VARARGS | METH_KEYWORDS, nullptr},
208:   {"special_shifted_chebyshev_polynomial_v", castPyCFunctionWithKeywords(THPVariable_special_shifted_chebyshev_polynomial_v), METH_VARARGS | METH_KEYWORDS, nullptr},
209:   {"special_shifted_chebyshev_polynomial_w", castPyCFunctionWithKeywords(THPVariable_special_shifted_chebyshev_polynomial_w), METH_VARARGS | METH_KEYWORDS, nullptr},
210:   {"special_sinc", castPyCFunctionWithKeywords(THPVariable_special_sinc), METH_VARARGS | METH_KEYWORDS, nullptr},
211:   {"special_softmax", castPyCFunctionWithKeywords(THPVariable_special_softmax), METH_VARARGS | METH_KEYWORDS, nullptr},
212:   {"special_spherical_bessel_j0", castPyCFunctionWithKeywords(THPVariable_special_spherical_bessel_j0), METH_VARARGS | METH_KEYWORDS, nullptr},
213:   {"special_xlog1py", castPyCFunctionWithKeywords(THPVariable_special_xlog1py), METH_VARARGS | METH_KEYWORDS, nullptr},
214:   {"special_xlogy", castPyCFunctionWithKeywords(THPVariable_special_xlogy), METH_VARARGS | METH_KEYWORDS, nullptr},
215:   {"special_zeta", castPyCFunctionWithKeywords(THPVariable_special_zeta), METH_VARARGS | METH_KEYWORDS, nullptr},
216:   {NULL}
217: };
218: 
219: static PyObject* THPSpecialVariableFunctionsModule = NULL;
220: 
221: void initSpecialFunctions(PyObject* module) {
222:   static struct PyModuleDef def = {
223:      PyModuleDef_HEAD_INIT,
224:      "torch._C._special",
225:      NULL,
226:      -1,
227:      special_functions
228:   };
229:   PyObject* special = PyModule_Create(&def);
230:   THPSpecialVariableFunctionsModule = special;
231:   if (!special) {
232:     throw python_error();
233:   }
234:   // steals a reference to special
235:   if (PyModule_AddObject(module, "_special", special) != 0) {
236:     throw python_error();
237:   }
238: }
239: 
240: // generated methods start here
```

- EN: The main execution path in this span is carried by `castPyCFunctionWithKeywords`, `initSpecialFunctions`, `PyModule_Create`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `castPyCFunctionWithKeywords`, `initSpecialFunctions`, `PyModule_Create` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 241-280

```cpp
241: 
242: // special_airy_ai
243: static PyObject * THPVariable_special_airy_ai(PyObject* self_, PyObject* args, PyObject* kwargs)
244: {
245:   HANDLE_TH_ERRORS
246:   static PythonArgParser parser({
247:     "special_airy_ai(Tensor x, *, Tensor out=None)",
248:   }, /*traceable=*/true);
249: 
250:   ParsedArgs<2> parsed_args;
251:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
252:   if(_r.has_torch_function()) {
253:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
254:   }
255:   if (_r.isNone(1)) {
256:     // aten::special_airy_ai(Tensor x) -> Tensor
257: 
258:     auto dispatch_special_airy_ai = [](const at::Tensor & x) -> at::Tensor {
259:       pybind11::gil_scoped_release no_gil;
260:       return at::special_airy_ai(x);
261:     };
262:     return wrap(dispatch_special_airy_ai(_r.tensor(0)));
263:   } else {
264:     // aten::special_airy_ai.out(Tensor x, *, Tensor(a!) out) -> Tensor(a!)
265: 
266:     auto dispatch_special_airy_ai_out = [](at::Tensor out, const at::Tensor & x) -> at::Tensor {
267:       pybind11::gil_scoped_release no_gil;
268:       return at::special_airy_ai_out(out, x);
269:     };
270:     return wrap(dispatch_special_airy_ai_out(_r.tensor(1), _r.tensor(0)));
271:   }
272:   Py_RETURN_NONE;
273:   END_HANDLE_TH_ERRORS
274: }
275: 
276: // special_bessel_j0
277: static PyObject * THPVariable_special_bessel_j0(PyObject* self_, PyObject* args, PyObject* kwargs)
278: {
279:   HANDLE_TH_ERRORS
280:   static PythonArgParser parser({
```

- EN: The main execution path in this span is carried by `THPVariable_special_airy_ai`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_special_airy_ai`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 281-320

```cpp
281:     "special_bessel_j0(Tensor input, *, Tensor out=None)",
282:   }, /*traceable=*/true);
283: 
284:   ParsedArgs<2> parsed_args;
285:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
286:   if(_r.has_torch_function()) {
287:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
288:   }
289:   if (_r.isNone(1)) {
290:     // aten::special_bessel_j0(Tensor self) -> Tensor
291: 
292:     auto dispatch_special_bessel_j0 = [](const at::Tensor & self) -> at::Tensor {
293:       pybind11::gil_scoped_release no_gil;
294:       return at::special_bessel_j0(self);
295:     };
296:     return wrap(dispatch_special_bessel_j0(_r.tensor(0)));
297:   } else {
298:     // aten::special_bessel_j0.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
299: 
300:     auto dispatch_special_bessel_j0_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
301:       pybind11::gil_scoped_release no_gil;
302:       return at::special_bessel_j0_out(out, self);
303:     };
304:     return wrap(dispatch_special_bessel_j0_out(_r.tensor(1), _r.tensor(0)));
305:   }
306:   Py_RETURN_NONE;
307:   END_HANDLE_TH_ERRORS
308: }
309: 
310: // special_bessel_j1
311: static PyObject * THPVariable_special_bessel_j1(PyObject* self_, PyObject* args, PyObject* kwargs)
312: {
313:   HANDLE_TH_ERRORS
314:   static PythonArgParser parser({
315:     "special_bessel_j1(Tensor input, *, Tensor out=None)",
316:   }, /*traceable=*/true);
317: 
318:   ParsedArgs<2> parsed_args;
319:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
320:   if(_r.has_torch_function()) {
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `special_bessel_j0`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `special_bessel_j0`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 321-360

```cpp
321:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
322:   }
323:   if (_r.isNone(1)) {
324:     // aten::special_bessel_j1(Tensor self) -> Tensor
325: 
326:     auto dispatch_special_bessel_j1 = [](const at::Tensor & self) -> at::Tensor {
327:       pybind11::gil_scoped_release no_gil;
328:       return at::special_bessel_j1(self);
329:     };
330:     return wrap(dispatch_special_bessel_j1(_r.tensor(0)));
331:   } else {
332:     // aten::special_bessel_j1.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
333: 
334:     auto dispatch_special_bessel_j1_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
335:       pybind11::gil_scoped_release no_gil;
336:       return at::special_bessel_j1_out(out, self);
337:     };
338:     return wrap(dispatch_special_bessel_j1_out(_r.tensor(1), _r.tensor(0)));
339:   }
340:   Py_RETURN_NONE;
341:   END_HANDLE_TH_ERRORS
342: }
343: 
344: // special_bessel_y0
345: static PyObject * THPVariable_special_bessel_y0(PyObject* self_, PyObject* args, PyObject* kwargs)
346: {
347:   HANDLE_TH_ERRORS
348:   static PythonArgParser parser({
349:     "special_bessel_y0(Tensor input, *, Tensor out=None)",
350:   }, /*traceable=*/true);
351: 
352:   ParsedArgs<2> parsed_args;
353:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
354:   if(_r.has_torch_function()) {
355:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
356:   }
357:   if (_r.isNone(1)) {
358:     // aten::special_bessel_y0(Tensor self) -> Tensor
359: 
360:     auto dispatch_special_bessel_y0 = [](const at::Tensor & self) -> at::Tensor {
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `special_bessel_j1`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `special_bessel_j1`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 361-400

```cpp
361:       pybind11::gil_scoped_release no_gil;
362:       return at::special_bessel_y0(self);
363:     };
364:     return wrap(dispatch_special_bessel_y0(_r.tensor(0)));
365:   } else {
366:     // aten::special_bessel_y0.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
367: 
368:     auto dispatch_special_bessel_y0_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
369:       pybind11::gil_scoped_release no_gil;
370:       return at::special_bessel_y0_out(out, self);
371:     };
372:     return wrap(dispatch_special_bessel_y0_out(_r.tensor(1), _r.tensor(0)));
373:   }
374:   Py_RETURN_NONE;
375:   END_HANDLE_TH_ERRORS
376: }
377: 
378: // special_bessel_y1
379: static PyObject * THPVariable_special_bessel_y1(PyObject* self_, PyObject* args, PyObject* kwargs)
380: {
381:   HANDLE_TH_ERRORS
382:   static PythonArgParser parser({
383:     "special_bessel_y1(Tensor input, *, Tensor out=None)",
384:   }, /*traceable=*/true);
385: 
386:   ParsedArgs<2> parsed_args;
387:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
388:   if(_r.has_torch_function()) {
389:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
390:   }
391:   if (_r.isNone(1)) {
392:     // aten::special_bessel_y1(Tensor self) -> Tensor
393: 
394:     auto dispatch_special_bessel_y1 = [](const at::Tensor & self) -> at::Tensor {
395:       pybind11::gil_scoped_release no_gil;
396:       return at::special_bessel_y1(self);
397:     };
398:     return wrap(dispatch_special_bessel_y1(_r.tensor(0)));
399:   } else {
400:     // aten::special_bessel_y1.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
```

- EN: The main execution path in this span is carried by `special_bessel_y0`, `wrap`, `special_bessel_y0_out`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `special_bessel_y0`, `wrap`, `special_bessel_y0_out` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 401-440

```cpp
401: 
402:     auto dispatch_special_bessel_y1_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
403:       pybind11::gil_scoped_release no_gil;
404:       return at::special_bessel_y1_out(out, self);
405:     };
406:     return wrap(dispatch_special_bessel_y1_out(_r.tensor(1), _r.tensor(0)));
407:   }
408:   Py_RETURN_NONE;
409:   END_HANDLE_TH_ERRORS
410: }
411: 
412: \
413: // special_chebyshev_polynomial_t
414: static PyObject * THPVariable_special_chebyshev_polynomial_t(PyObject* self_, PyObject* args, PyObject* kwargs)
415: {
416:   HANDLE_TH_ERRORS
417:   static PythonArgParser parser({
418:     "special_chebyshev_polynomial_t(Tensor x, Tensor n, *, Tensor out=None)",
419:     "special_chebyshev_polynomial_t(Scalar x, Tensor n, *, Tensor out=None)",
420:     "special_chebyshev_polynomial_t(Tensor x, Scalar n, *, Tensor out=None)",
421:   }, /*traceable=*/true);
422: 
423:   ParsedArgs<3> parsed_args;
424:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
425:   if(_r.has_torch_function()) {
426:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
427:   }
428:   switch (_r.idx) {
429:     case 0: {
430:       if (_r.isNone(2)) {
431:         // aten::special_chebyshev_polynomial_t(Tensor x, Tensor n) -> Tensor
432: 
433:         auto dispatch_special_chebyshev_polynomial_t = [](const at::Tensor & x, const at::Tensor & n) -> at::Tensor {
434:           pybind11::gil_scoped_release no_gil;
435:           return at::special_chebyshev_polynomial_t(x, n);
436:         };
437:         return wrap(dispatch_special_chebyshev_polynomial_t(_r.tensor(0), _r.tensor(1)));
438:       } else {
439:         // aten::special_chebyshev_polynomial_t.out(Tensor x, Tensor n, *, Tensor(a!) out) -> Tensor(a!)
440: 
```

- EN: The main execution path in this span is carried by `special_bessel_y1_out`, `wrap`, `THPVariable_special_chebyshev_polynomial_t`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `special_bessel_y1_out`, `wrap`, `THPVariable_special_chebyshev_polynomial_t` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 441-480

```cpp
441:         auto dispatch_special_chebyshev_polynomial_t_out = [](at::Tensor out, const at::Tensor & x, const at::Tensor & n) -> at::Tensor {
442:           pybind11::gil_scoped_release no_gil;
443:           return at::special_chebyshev_polynomial_t_out(out, x, n);
444:         };
445:         return wrap(dispatch_special_chebyshev_polynomial_t_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
446:       }
447:     }
448:     case 1: {
449:       if (_r.isNone(2)) {
450:         // aten::special_chebyshev_polynomial_t.x_scalar(Scalar x, Tensor n) -> Tensor
451: 
452:         auto dispatch_special_chebyshev_polynomial_t = [](const at::Scalar & x, const at::Tensor & n) -> at::Tensor {
453:           pybind11::gil_scoped_release no_gil;
454:           return at::special_chebyshev_polynomial_t(x, n);
455:         };
456:         return wrap(dispatch_special_chebyshev_polynomial_t(_r.scalar(0), _r.tensor(1)));
457:       } else {
458:         // aten::special_chebyshev_polynomial_t.x_scalar_out(Scalar x, Tensor n, *, Tensor(a!) out) -> Tensor(a!)
459: 
460:         auto dispatch_special_chebyshev_polynomial_t_out = [](at::Tensor out, const at::Scalar & x, const at::Tensor & n) -> at::Tensor {
461:           pybind11::gil_scoped_release no_gil;
462:           return at::special_chebyshev_polynomial_t_out(out, x, n);
463:         };
464:         return wrap(dispatch_special_chebyshev_polynomial_t_out(_r.tensor(2), _r.scalar(0), _r.tensor(1)));
465:       }
466:     }
467:     case 2: {
468:       if (_r.isNone(2)) {
469:         // aten::special_chebyshev_polynomial_t.n_scalar(Tensor x, Scalar n) -> Tensor
470: 
471:         auto dispatch_special_chebyshev_polynomial_t = [](const at::Tensor & x, const at::Scalar & n) -> at::Tensor {
472:           pybind11::gil_scoped_release no_gil;
473:           return at::special_chebyshev_polynomial_t(x, n);
474:         };
475:         return wrap(dispatch_special_chebyshev_polynomial_t(_r.tensor(0), _r.scalar(1)));
476:       } else {
477:         // aten::special_chebyshev_polynomial_t.n_scalar_out(Tensor x, Scalar n, *, Tensor(a!) out) -> Tensor(a!)
478: 
479:         auto dispatch_special_chebyshev_polynomial_t_out = [](at::Tensor out, const at::Tensor & x, const at::Scalar & n) -> at::Tensor {
480:           pybind11::gil_scoped_release no_gil;
```

- EN: The main execution path in this span is carried by `special_chebyshev_polynomial_t_out`, `wrap`, `special_chebyshev_polynomial_t`. Control-flow branches in this range handle alternative runtime cases or iterative work. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `special_chebyshev_polynomial_t_out`, `wrap`, `special_chebyshev_polynomial_t` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 481-520

```cpp
481:           return at::special_chebyshev_polynomial_t_out(out, x, n);
482:         };
483:         return wrap(dispatch_special_chebyshev_polynomial_t_out(_r.tensor(2), _r.tensor(0), _r.scalar(1)));
484:       }
485:     }
486:   }
487:   Py_RETURN_NONE;
488:   END_HANDLE_TH_ERRORS
489: }
490: 
491: \
492: // special_chebyshev_polynomial_u
493: static PyObject * THPVariable_special_chebyshev_polynomial_u(PyObject* self_, PyObject* args, PyObject* kwargs)
494: {
495:   HANDLE_TH_ERRORS
496:   static PythonArgParser parser({
497:     "special_chebyshev_polynomial_u(Tensor x, Tensor n, *, Tensor out=None)",
498:     "special_chebyshev_polynomial_u(Scalar x, Tensor n, *, Tensor out=None)",
499:     "special_chebyshev_polynomial_u(Tensor x, Scalar n, *, Tensor out=None)",
500:   }, /*traceable=*/true);
501: 
502:   ParsedArgs<3> parsed_args;
503:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
504:   if(_r.has_torch_function()) {
505:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
506:   }
507:   switch (_r.idx) {
508:     case 0: {
509:       if (_r.isNone(2)) {
510:         // aten::special_chebyshev_polynomial_u(Tensor x, Tensor n) -> Tensor
511: 
512:         auto dispatch_special_chebyshev_polynomial_u = [](const at::Tensor & x, const at::Tensor & n) -> at::Tensor {
513:           pybind11::gil_scoped_release no_gil;
514:           return at::special_chebyshev_polynomial_u(x, n);
515:         };
516:         return wrap(dispatch_special_chebyshev_polynomial_u(_r.tensor(0), _r.tensor(1)));
517:       } else {
518:         // aten::special_chebyshev_polynomial_u.out(Tensor x, Tensor n, *, Tensor(a!) out) -> Tensor(a!)
519: 
520:         auto dispatch_special_chebyshev_polynomial_u_out = [](at::Tensor out, const at::Tensor & x, const at::Tensor & n) -> at::Tensor {
```

- EN: The main execution path in this span is carried by `special_chebyshev_polynomial_t_out`, `wrap`, `THPVariable_special_chebyshev_polynomial_u`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `special_chebyshev_polynomial_t_out`, `wrap`, `THPVariable_special_chebyshev_polynomial_u` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 521-560

```cpp
521:           pybind11::gil_scoped_release no_gil;
522:           return at::special_chebyshev_polynomial_u_out(out, x, n);
523:         };
524:         return wrap(dispatch_special_chebyshev_polynomial_u_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
525:       }
526:     }
527:     case 1: {
528:       if (_r.isNone(2)) {
529:         // aten::special_chebyshev_polynomial_u.x_scalar(Scalar x, Tensor n) -> Tensor
530: 
531:         auto dispatch_special_chebyshev_polynomial_u = [](const at::Scalar & x, const at::Tensor & n) -> at::Tensor {
532:           pybind11::gil_scoped_release no_gil;
533:           return at::special_chebyshev_polynomial_u(x, n);
534:         };
535:         return wrap(dispatch_special_chebyshev_polynomial_u(_r.scalar(0), _r.tensor(1)));
536:       } else {
537:         // aten::special_chebyshev_polynomial_u.x_scalar_out(Scalar x, Tensor n, *, Tensor(a!) out) -> Tensor(a!)
538: 
539:         auto dispatch_special_chebyshev_polynomial_u_out = [](at::Tensor out, const at::Scalar & x, const at::Tensor & n) -> at::Tensor {
540:           pybind11::gil_scoped_release no_gil;
541:           return at::special_chebyshev_polynomial_u_out(out, x, n);
542:         };
543:         return wrap(dispatch_special_chebyshev_polynomial_u_out(_r.tensor(2), _r.scalar(0), _r.tensor(1)));
544:       }
545:     }
546:     case 2: {
547:       if (_r.isNone(2)) {
548:         // aten::special_chebyshev_polynomial_u.n_scalar(Tensor x, Scalar n) -> Tensor
549: 
550:         auto dispatch_special_chebyshev_polynomial_u = [](const at::Tensor & x, const at::Scalar & n) -> at::Tensor {
551:           pybind11::gil_scoped_release no_gil;
552:           return at::special_chebyshev_polynomial_u(x, n);
553:         };
554:         return wrap(dispatch_special_chebyshev_polynomial_u(_r.tensor(0), _r.scalar(1)));
555:       } else {
556:         // aten::special_chebyshev_polynomial_u.n_scalar_out(Tensor x, Scalar n, *, Tensor(a!) out) -> Tensor(a!)
557: 
558:         auto dispatch_special_chebyshev_polynomial_u_out = [](at::Tensor out, const at::Tensor & x, const at::Scalar & n) -> at::Tensor {
559:           pybind11::gil_scoped_release no_gil;
560:           return at::special_chebyshev_polynomial_u_out(out, x, n);
```

- EN: The main execution path in this span is carried by `special_chebyshev_polynomial_u_out`, `wrap`, `special_chebyshev_polynomial_u`. Control-flow branches in this range handle alternative runtime cases or iterative work. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `special_chebyshev_polynomial_u_out`, `wrap`, `special_chebyshev_polynomial_u` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 561-600

```cpp
561:         };
562:         return wrap(dispatch_special_chebyshev_polynomial_u_out(_r.tensor(2), _r.tensor(0), _r.scalar(1)));
563:       }
564:     }
565:   }
566:   Py_RETURN_NONE;
567:   END_HANDLE_TH_ERRORS
568: }
569: 
570: \
571: // special_chebyshev_polynomial_v
572: static PyObject * THPVariable_special_chebyshev_polynomial_v(PyObject* self_, PyObject* args, PyObject* kwargs)
573: {
574:   HANDLE_TH_ERRORS
575:   static PythonArgParser parser({
576:     "special_chebyshev_polynomial_v(Tensor x, Tensor n, *, Tensor out=None)",
577:     "special_chebyshev_polynomial_v(Scalar x, Tensor n, *, Tensor out=None)",
578:     "special_chebyshev_polynomial_v(Tensor x, Scalar n, *, Tensor out=None)",
579:   }, /*traceable=*/true);
580: 
581:   ParsedArgs<3> parsed_args;
582:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
583:   if(_r.has_torch_function()) {
584:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
585:   }
586:   switch (_r.idx) {
587:     case 0: {
588:       if (_r.isNone(2)) {
589:         // aten::special_chebyshev_polynomial_v(Tensor x, Tensor n) -> Tensor
590: 
591:         auto dispatch_special_chebyshev_polynomial_v = [](const at::Tensor & x, const at::Tensor & n) -> at::Tensor {
592:           pybind11::gil_scoped_release no_gil;
593:           return at::special_chebyshev_polynomial_v(x, n);
594:         };
595:         return wrap(dispatch_special_chebyshev_polynomial_v(_r.tensor(0), _r.tensor(1)));
596:       } else {
597:         // aten::special_chebyshev_polynomial_v.out(Tensor x, Tensor n, *, Tensor(a!) out) -> Tensor(a!)
598: 
599:         auto dispatch_special_chebyshev_polynomial_v_out = [](at::Tensor out, const at::Tensor & x, const at::Tensor & n) -> at::Tensor {
600:           pybind11::gil_scoped_release no_gil;
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_special_chebyshev_polynomial_v`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_special_chebyshev_polynomial_v`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 601-640

```cpp
601:           return at::special_chebyshev_polynomial_v_out(out, x, n);
602:         };
603:         return wrap(dispatch_special_chebyshev_polynomial_v_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
604:       }
605:     }
606:     case 1: {
607:       if (_r.isNone(2)) {
608:         // aten::special_chebyshev_polynomial_v.x_scalar(Scalar x, Tensor n) -> Tensor
609: 
610:         auto dispatch_special_chebyshev_polynomial_v = [](const at::Scalar & x, const at::Tensor & n) -> at::Tensor {
611:           pybind11::gil_scoped_release no_gil;
612:           return at::special_chebyshev_polynomial_v(x, n);
613:         };
614:         return wrap(dispatch_special_chebyshev_polynomial_v(_r.scalar(0), _r.tensor(1)));
615:       } else {
616:         // aten::special_chebyshev_polynomial_v.x_scalar_out(Scalar x, Tensor n, *, Tensor(a!) out) -> Tensor(a!)
617: 
618:         auto dispatch_special_chebyshev_polynomial_v_out = [](at::Tensor out, const at::Scalar & x, const at::Tensor & n) -> at::Tensor {
619:           pybind11::gil_scoped_release no_gil;
620:           return at::special_chebyshev_polynomial_v_out(out, x, n);
621:         };
622:         return wrap(dispatch_special_chebyshev_polynomial_v_out(_r.tensor(2), _r.scalar(0), _r.tensor(1)));
623:       }
624:     }
625:     case 2: {
626:       if (_r.isNone(2)) {
627:         // aten::special_chebyshev_polynomial_v.n_scalar(Tensor x, Scalar n) -> Tensor
628: 
629:         auto dispatch_special_chebyshev_polynomial_v = [](const at::Tensor & x, const at::Scalar & n) -> at::Tensor {
630:           pybind11::gil_scoped_release no_gil;
631:           return at::special_chebyshev_polynomial_v(x, n);
632:         };
633:         return wrap(dispatch_special_chebyshev_polynomial_v(_r.tensor(0), _r.scalar(1)));
634:       } else {
635:         // aten::special_chebyshev_polynomial_v.n_scalar_out(Tensor x, Scalar n, *, Tensor(a!) out) -> Tensor(a!)
636: 
637:         auto dispatch_special_chebyshev_polynomial_v_out = [](at::Tensor out, const at::Tensor & x, const at::Scalar & n) -> at::Tensor {
638:           pybind11::gil_scoped_release no_gil;
639:           return at::special_chebyshev_polynomial_v_out(out, x, n);
640:         };
```

- EN: The main execution path in this span is carried by `special_chebyshev_polynomial_v_out`, `wrap`, `special_chebyshev_polynomial_v`. Control-flow branches in this range handle alternative runtime cases or iterative work. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `special_chebyshev_polynomial_v_out`, `wrap`, `special_chebyshev_polynomial_v` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 641-680

```cpp
641:         return wrap(dispatch_special_chebyshev_polynomial_v_out(_r.tensor(2), _r.tensor(0), _r.scalar(1)));
642:       }
643:     }
644:   }
645:   Py_RETURN_NONE;
646:   END_HANDLE_TH_ERRORS
647: }
648: 
649: \
650: // special_chebyshev_polynomial_w
651: static PyObject * THPVariable_special_chebyshev_polynomial_w(PyObject* self_, PyObject* args, PyObject* kwargs)
652: {
653:   HANDLE_TH_ERRORS
654:   static PythonArgParser parser({
655:     "special_chebyshev_polynomial_w(Tensor x, Tensor n, *, Tensor out=None)",
656:     "special_chebyshev_polynomial_w(Scalar x, Tensor n, *, Tensor out=None)",
657:     "special_chebyshev_polynomial_w(Tensor x, Scalar n, *, Tensor out=None)",
658:   }, /*traceable=*/true);
659: 
660:   ParsedArgs<3> parsed_args;
661:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
662:   if(_r.has_torch_function()) {
663:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
664:   }
665:   switch (_r.idx) {
666:     case 0: {
667:       if (_r.isNone(2)) {
668:         // aten::special_chebyshev_polynomial_w(Tensor x, Tensor n) -> Tensor
669: 
670:         auto dispatch_special_chebyshev_polynomial_w = [](const at::Tensor & x, const at::Tensor & n) -> at::Tensor {
671:           pybind11::gil_scoped_release no_gil;
672:           return at::special_chebyshev_polynomial_w(x, n);
673:         };
674:         return wrap(dispatch_special_chebyshev_polynomial_w(_r.tensor(0), _r.tensor(1)));
675:       } else {
676:         // aten::special_chebyshev_polynomial_w.out(Tensor x, Tensor n, *, Tensor(a!) out) -> Tensor(a!)
677: 
678:         auto dispatch_special_chebyshev_polynomial_w_out = [](at::Tensor out, const at::Tensor & x, const at::Tensor & n) -> at::Tensor {
679:           pybind11::gil_scoped_release no_gil;
680:           return at::special_chebyshev_polynomial_w_out(out, x, n);
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_special_chebyshev_polynomial_w`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_special_chebyshev_polynomial_w`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 681-720

```cpp
681:         };
682:         return wrap(dispatch_special_chebyshev_polynomial_w_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
683:       }
684:     }
685:     case 1: {
686:       if (_r.isNone(2)) {
687:         // aten::special_chebyshev_polynomial_w.x_scalar(Scalar x, Tensor n) -> Tensor
688: 
689:         auto dispatch_special_chebyshev_polynomial_w = [](const at::Scalar & x, const at::Tensor & n) -> at::Tensor {
690:           pybind11::gil_scoped_release no_gil;
691:           return at::special_chebyshev_polynomial_w(x, n);
692:         };
693:         return wrap(dispatch_special_chebyshev_polynomial_w(_r.scalar(0), _r.tensor(1)));
694:       } else {
695:         // aten::special_chebyshev_polynomial_w.x_scalar_out(Scalar x, Tensor n, *, Tensor(a!) out) -> Tensor(a!)
696: 
697:         auto dispatch_special_chebyshev_polynomial_w_out = [](at::Tensor out, const at::Scalar & x, const at::Tensor & n) -> at::Tensor {
698:           pybind11::gil_scoped_release no_gil;
699:           return at::special_chebyshev_polynomial_w_out(out, x, n);
700:         };
701:         return wrap(dispatch_special_chebyshev_polynomial_w_out(_r.tensor(2), _r.scalar(0), _r.tensor(1)));
702:       }
703:     }
704:     case 2: {
705:       if (_r.isNone(2)) {
706:         // aten::special_chebyshev_polynomial_w.n_scalar(Tensor x, Scalar n) -> Tensor
707: 
708:         auto dispatch_special_chebyshev_polynomial_w = [](const at::Tensor & x, const at::Scalar & n) -> at::Tensor {
709:           pybind11::gil_scoped_release no_gil;
710:           return at::special_chebyshev_polynomial_w(x, n);
711:         };
712:         return wrap(dispatch_special_chebyshev_polynomial_w(_r.tensor(0), _r.scalar(1)));
713:       } else {
714:         // aten::special_chebyshev_polynomial_w.n_scalar_out(Tensor x, Scalar n, *, Tensor(a!) out) -> Tensor(a!)
715: 
716:         auto dispatch_special_chebyshev_polynomial_w_out = [](at::Tensor out, const at::Tensor & x, const at::Scalar & n) -> at::Tensor {
717:           pybind11::gil_scoped_release no_gil;
718:           return at::special_chebyshev_polynomial_w_out(out, x, n);
719:         };
720:         return wrap(dispatch_special_chebyshev_polynomial_w_out(_r.tensor(2), _r.tensor(0), _r.scalar(1)));
```

- EN: The main execution path in this span is carried by `wrap`, `special_chebyshev_polynomial_w`, `special_chebyshev_polynomial_w_out`. Control-flow branches in this range handle alternative runtime cases or iterative work. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `special_chebyshev_polynomial_w`, `special_chebyshev_polynomial_w_out` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 721-760

```cpp
721:       }
722:     }
723:   }
724:   Py_RETURN_NONE;
725:   END_HANDLE_TH_ERRORS
726: }
727: 
728: // special_digamma
729: static PyObject * THPVariable_special_digamma(PyObject* self_, PyObject* args, PyObject* kwargs)
730: {
731:   HANDLE_TH_ERRORS
732:   static PythonArgParser parser({
733:     "special_digamma(Tensor input, *, Tensor out=None)",
734:   }, /*traceable=*/true);
735: 
736:   ParsedArgs<2> parsed_args;
737:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
738:   if(_r.has_torch_function()) {
739:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
740:   }
741:   if (_r.isNone(1)) {
742:     // aten::special_digamma(Tensor self) -> Tensor
743: 
744:     auto dispatch_special_digamma = [](const at::Tensor & self) -> at::Tensor {
745:       pybind11::gil_scoped_release no_gil;
746:       return at::special_digamma(self);
747:     };
748:     return wrap(dispatch_special_digamma(_r.tensor(0)));
749:   } else {
750:     // aten::special_digamma.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
751: 
752:     auto dispatch_special_digamma_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
753:       pybind11::gil_scoped_release no_gil;
754:       return at::special_digamma_out(out, self);
755:     };
756:     return wrap(dispatch_special_digamma_out(_r.tensor(1), _r.tensor(0)));
757:   }
758:   Py_RETURN_NONE;
759:   END_HANDLE_TH_ERRORS
760: }
```

- EN: The main execution path in this span is carried by `THPVariable_special_digamma`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_special_digamma`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 761-800

```cpp
761: 
762: // special_entr
763: static PyObject * THPVariable_special_entr(PyObject* self_, PyObject* args, PyObject* kwargs)
764: {
765:   HANDLE_TH_ERRORS
766:   static PythonArgParser parser({
767:     "special_entr(Tensor input, *, Tensor out=None)",
768:   }, /*traceable=*/true);
769: 
770:   ParsedArgs<2> parsed_args;
771:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
772:   if(_r.has_torch_function()) {
773:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
774:   }
775:   if (_r.isNone(1)) {
776:     // aten::special_entr(Tensor self) -> Tensor
777: 
778:     auto dispatch_special_entr = [](const at::Tensor & self) -> at::Tensor {
779:       pybind11::gil_scoped_release no_gil;
780:       return at::special_entr(self);
781:     };
782:     return wrap(dispatch_special_entr(_r.tensor(0)));
783:   } else {
784:     // aten::special_entr.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
785: 
786:     auto dispatch_special_entr_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
787:       pybind11::gil_scoped_release no_gil;
788:       return at::special_entr_out(out, self);
789:     };
790:     return wrap(dispatch_special_entr_out(_r.tensor(1), _r.tensor(0)));
791:   }
792:   Py_RETURN_NONE;
793:   END_HANDLE_TH_ERRORS
794: }
795: 
796: // special_erf
797: static PyObject * THPVariable_special_erf(PyObject* self_, PyObject* args, PyObject* kwargs)
798: {
799:   HANDLE_TH_ERRORS
800:   static PythonArgParser parser({
```

- EN: The main execution path in this span is carried by `THPVariable_special_entr`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_special_entr`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 801-840

```cpp
801:     "special_erf(Tensor input, *, Tensor out=None)",
802:   }, /*traceable=*/true);
803: 
804:   ParsedArgs<2> parsed_args;
805:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
806:   if(_r.has_torch_function()) {
807:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
808:   }
809:   if (_r.isNone(1)) {
810:     // aten::special_erf(Tensor self) -> Tensor
811: 
812:     auto dispatch_special_erf = [](const at::Tensor & self) -> at::Tensor {
813:       pybind11::gil_scoped_release no_gil;
814:       return at::special_erf(self);
815:     };
816:     return wrap(dispatch_special_erf(_r.tensor(0)));
817:   } else {
818:     // aten::special_erf.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
819: 
820:     auto dispatch_special_erf_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
821:       pybind11::gil_scoped_release no_gil;
822:       return at::special_erf_out(out, self);
823:     };
824:     return wrap(dispatch_special_erf_out(_r.tensor(1), _r.tensor(0)));
825:   }
826:   Py_RETURN_NONE;
827:   END_HANDLE_TH_ERRORS
828: }
829: 
830: // special_erfc
831: static PyObject * THPVariable_special_erfc(PyObject* self_, PyObject* args, PyObject* kwargs)
832: {
833:   HANDLE_TH_ERRORS
834:   static PythonArgParser parser({
835:     "special_erfc(Tensor input, *, Tensor out=None)",
836:   }, /*traceable=*/true);
837: 
838:   ParsedArgs<2> parsed_args;
839:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
840:   if(_r.has_torch_function()) {
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `special_erf`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `special_erf`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 841-880

```cpp
841:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
842:   }
843:   if (_r.isNone(1)) {
844:     // aten::special_erfc(Tensor self) -> Tensor
845: 
846:     auto dispatch_special_erfc = [](const at::Tensor & self) -> at::Tensor {
847:       pybind11::gil_scoped_release no_gil;
848:       return at::special_erfc(self);
849:     };
850:     return wrap(dispatch_special_erfc(_r.tensor(0)));
851:   } else {
852:     // aten::special_erfc.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
853: 
854:     auto dispatch_special_erfc_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
855:       pybind11::gil_scoped_release no_gil;
856:       return at::special_erfc_out(out, self);
857:     };
858:     return wrap(dispatch_special_erfc_out(_r.tensor(1), _r.tensor(0)));
859:   }
860:   Py_RETURN_NONE;
861:   END_HANDLE_TH_ERRORS
862: }
863: 
864: // special_erfcx
865: static PyObject * THPVariable_special_erfcx(PyObject* self_, PyObject* args, PyObject* kwargs)
866: {
867:   HANDLE_TH_ERRORS
868:   static PythonArgParser parser({
869:     "special_erfcx(Tensor input, *, Tensor out=None)",
870:   }, /*traceable=*/true);
871: 
872:   ParsedArgs<2> parsed_args;
873:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
874:   if(_r.has_torch_function()) {
875:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
876:   }
877:   if (_r.isNone(1)) {
878:     // aten::special_erfcx(Tensor self) -> Tensor
879: 
880:     auto dispatch_special_erfcx = [](const at::Tensor & self) -> at::Tensor {
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `special_erfc`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `special_erfc`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 881-920

```cpp
881:       pybind11::gil_scoped_release no_gil;
882:       return at::special_erfcx(self);
883:     };
884:     return wrap(dispatch_special_erfcx(_r.tensor(0)));
885:   } else {
886:     // aten::special_erfcx.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
887: 
888:     auto dispatch_special_erfcx_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
889:       pybind11::gil_scoped_release no_gil;
890:       return at::special_erfcx_out(out, self);
891:     };
892:     return wrap(dispatch_special_erfcx_out(_r.tensor(1), _r.tensor(0)));
893:   }
894:   Py_RETURN_NONE;
895:   END_HANDLE_TH_ERRORS
896: }
897: 
898: // special_erfinv
899: static PyObject * THPVariable_special_erfinv(PyObject* self_, PyObject* args, PyObject* kwargs)
900: {
901:   HANDLE_TH_ERRORS
902:   static PythonArgParser parser({
903:     "special_erfinv(Tensor input, *, Tensor out=None)",
904:   }, /*traceable=*/true);
905: 
906:   ParsedArgs<2> parsed_args;
907:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
908:   if(_r.has_torch_function()) {
909:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
910:   }
911:   if (_r.isNone(1)) {
912:     // aten::special_erfinv(Tensor self) -> Tensor
913: 
914:     auto dispatch_special_erfinv = [](const at::Tensor & self) -> at::Tensor {
915:       pybind11::gil_scoped_release no_gil;
916:       return at::special_erfinv(self);
917:     };
918:     return wrap(dispatch_special_erfinv(_r.tensor(0)));
919:   } else {
920:     // aten::special_erfinv.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
```

- EN: The main execution path in this span is carried by `special_erfcx`, `wrap`, `special_erfcx_out`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `special_erfcx`, `wrap`, `special_erfcx_out` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 921-960

```cpp
921: 
922:     auto dispatch_special_erfinv_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
923:       pybind11::gil_scoped_release no_gil;
924:       return at::special_erfinv_out(out, self);
925:     };
926:     return wrap(dispatch_special_erfinv_out(_r.tensor(1), _r.tensor(0)));
927:   }
928:   Py_RETURN_NONE;
929:   END_HANDLE_TH_ERRORS
930: }
931: 
932: // special_exp2
933: static PyObject * THPVariable_special_exp2(PyObject* self_, PyObject* args, PyObject* kwargs)
934: {
935:   HANDLE_TH_ERRORS
936:   static PythonArgParser parser({
937:     "special_exp2(Tensor input, *, Tensor out=None)",
938:   }, /*traceable=*/true);
939: 
940:   ParsedArgs<2> parsed_args;
941:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
942:   if(_r.has_torch_function()) {
943:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
944:   }
945:   if (_r.isNone(1)) {
946:     // aten::special_exp2(Tensor self) -> Tensor
947: 
948:     auto dispatch_special_exp2 = [](const at::Tensor & self) -> at::Tensor {
949:       pybind11::gil_scoped_release no_gil;
950:       return at::special_exp2(self);
951:     };
952:     return wrap(dispatch_special_exp2(_r.tensor(0)));
953:   } else {
954:     // aten::special_exp2.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
955: 
956:     auto dispatch_special_exp2_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
957:       pybind11::gil_scoped_release no_gil;
958:       return at::special_exp2_out(out, self);
959:     };
960:     return wrap(dispatch_special_exp2_out(_r.tensor(1), _r.tensor(0)));
```

- EN: The main execution path in this span is carried by `special_erfinv_out`, `wrap`, `THPVariable_special_exp2`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `special_erfinv_out`, `wrap`, `THPVariable_special_exp2` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 961-1000

```cpp
 961:   }
 962:   Py_RETURN_NONE;
 963:   END_HANDLE_TH_ERRORS
 964: }
 965: 
 966: // special_expit
 967: static PyObject * THPVariable_special_expit(PyObject* self_, PyObject* args, PyObject* kwargs)
 968: {
 969:   HANDLE_TH_ERRORS
 970:   static PythonArgParser parser({
 971:     "special_expit(Tensor input, *, Tensor out=None)",
 972:   }, /*traceable=*/true);
 973: 
 974:   ParsedArgs<2> parsed_args;
 975:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
 976:   if(_r.has_torch_function()) {
 977:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
 978:   }
 979:   if (_r.isNone(1)) {
 980:     // aten::special_expit(Tensor self) -> Tensor
 981: 
 982:     auto dispatch_special_expit = [](const at::Tensor & self) -> at::Tensor {
 983:       pybind11::gil_scoped_release no_gil;
 984:       return at::special_expit(self);
 985:     };
 986:     return wrap(dispatch_special_expit(_r.tensor(0)));
 987:   } else {
 988:     // aten::special_expit.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
 989: 
 990:     auto dispatch_special_expit_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
 991:       pybind11::gil_scoped_release no_gil;
 992:       return at::special_expit_out(out, self);
 993:     };
 994:     return wrap(dispatch_special_expit_out(_r.tensor(1), _r.tensor(0)));
 995:   }
 996:   Py_RETURN_NONE;
 997:   END_HANDLE_TH_ERRORS
 998: }
 999: 
1000: // special_expm1
```

- EN: The main execution path in this span is carried by `THPVariable_special_expit`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_special_expit`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1001-1040

```cpp
1001: static PyObject * THPVariable_special_expm1(PyObject* self_, PyObject* args, PyObject* kwargs)
1002: {
1003:   HANDLE_TH_ERRORS
1004:   static PythonArgParser parser({
1005:     "special_expm1(Tensor input, *, Tensor out=None)",
1006:   }, /*traceable=*/true);
1007: 
1008:   ParsedArgs<2> parsed_args;
1009:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1010:   if(_r.has_torch_function()) {
1011:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
1012:   }
1013:   if (_r.isNone(1)) {
1014:     // aten::special_expm1(Tensor self) -> Tensor
1015: 
1016:     auto dispatch_special_expm1 = [](const at::Tensor & self) -> at::Tensor {
1017:       pybind11::gil_scoped_release no_gil;
1018:       return at::special_expm1(self);
1019:     };
1020:     return wrap(dispatch_special_expm1(_r.tensor(0)));
1021:   } else {
1022:     // aten::special_expm1.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
1023: 
1024:     auto dispatch_special_expm1_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
1025:       pybind11::gil_scoped_release no_gil;
1026:       return at::special_expm1_out(out, self);
1027:     };
1028:     return wrap(dispatch_special_expm1_out(_r.tensor(1), _r.tensor(0)));
1029:   }
1030:   Py_RETURN_NONE;
1031:   END_HANDLE_TH_ERRORS
1032: }
1033: 
1034: // special_gammainc
1035: static PyObject * THPVariable_special_gammainc(PyObject* self_, PyObject* args, PyObject* kwargs)
1036: {
1037:   HANDLE_TH_ERRORS
1038:   static PythonArgParser parser({
1039:     "special_gammainc(Tensor input, Tensor other, *, Tensor out=None)",
1040:   }, /*traceable=*/true);
```

- EN: The main execution path in this span is carried by `THPVariable_special_expm1`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_special_expm1`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1041-1080

```cpp
1041: 
1042:   ParsedArgs<3> parsed_args;
1043:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1044:   if(_r.has_torch_function()) {
1045:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
1046:   }
1047:   if (_r.isNone(2)) {
1048:     // aten::special_gammainc(Tensor self, Tensor other) -> Tensor
1049: 
1050:     auto dispatch_special_gammainc = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
1051:       pybind11::gil_scoped_release no_gil;
1052:       return at::special_gammainc(self, other);
1053:     };
1054:     return wrap(dispatch_special_gammainc(_r.tensor(0), _r.tensor(1)));
1055:   } else {
1056:     // aten::special_gammainc.out(Tensor self, Tensor other, *, Tensor(a!) out) -> Tensor(a!)
1057: 
1058:     auto dispatch_special_gammainc_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
1059:       pybind11::gil_scoped_release no_gil;
1060:       return at::special_gammainc_out(out, self, other);
1061:     };
1062:     return wrap(dispatch_special_gammainc_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
1063:   }
1064:   Py_RETURN_NONE;
1065:   END_HANDLE_TH_ERRORS
1066: }
1067: 
1068: // special_gammaincc
1069: static PyObject * THPVariable_special_gammaincc(PyObject* self_, PyObject* args, PyObject* kwargs)
1070: {
1071:   HANDLE_TH_ERRORS
1072:   static PythonArgParser parser({
1073:     "special_gammaincc(Tensor input, Tensor other, *, Tensor out=None)",
1074:   }, /*traceable=*/true);
1075: 
1076:   ParsedArgs<3> parsed_args;
1077:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1078:   if(_r.has_torch_function()) {
1079:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
1080:   }
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `special_gammainc`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `special_gammainc`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1081-1120

```cpp
1081:   if (_r.isNone(2)) {
1082:     // aten::special_gammaincc(Tensor self, Tensor other) -> Tensor
1083: 
1084:     auto dispatch_special_gammaincc = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
1085:       pybind11::gil_scoped_release no_gil;
1086:       return at::special_gammaincc(self, other);
1087:     };
1088:     return wrap(dispatch_special_gammaincc(_r.tensor(0), _r.tensor(1)));
1089:   } else {
1090:     // aten::special_gammaincc.out(Tensor self, Tensor other, *, Tensor(a!) out) -> Tensor(a!)
1091: 
1092:     auto dispatch_special_gammaincc_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
1093:       pybind11::gil_scoped_release no_gil;
1094:       return at::special_gammaincc_out(out, self, other);
1095:     };
1096:     return wrap(dispatch_special_gammaincc_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
1097:   }
1098:   Py_RETURN_NONE;
1099:   END_HANDLE_TH_ERRORS
1100: }
1101: 
1102: // special_gammaln
1103: static PyObject * THPVariable_special_gammaln(PyObject* self_, PyObject* args, PyObject* kwargs)
1104: {
1105:   HANDLE_TH_ERRORS
1106:   static PythonArgParser parser({
1107:     "special_gammaln(Tensor input, *, Tensor out=None)",
1108:   }, /*traceable=*/true);
1109: 
1110:   ParsedArgs<2> parsed_args;
1111:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1112:   if(_r.has_torch_function()) {
1113:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
1114:   }
1115:   if (_r.isNone(1)) {
1116:     // aten::special_gammaln(Tensor self) -> Tensor
1117: 
1118:     auto dispatch_special_gammaln = [](const at::Tensor & self) -> at::Tensor {
1119:       pybind11::gil_scoped_release no_gil;
1120:       return at::special_gammaln(self);
```

- EN: The main execution path in this span is carried by `special_gammaincc`, `wrap`, `special_gammaincc_out`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `special_gammaincc`, `wrap`, `special_gammaincc_out` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1121-1160

```cpp
1121:     };
1122:     return wrap(dispatch_special_gammaln(_r.tensor(0)));
1123:   } else {
1124:     // aten::special_gammaln.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
1125: 
1126:     auto dispatch_special_gammaln_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
1127:       pybind11::gil_scoped_release no_gil;
1128:       return at::special_gammaln_out(out, self);
1129:     };
1130:     return wrap(dispatch_special_gammaln_out(_r.tensor(1), _r.tensor(0)));
1131:   }
1132:   Py_RETURN_NONE;
1133:   END_HANDLE_TH_ERRORS
1134: }
1135: 
1136: \
1137: // special_hermite_polynomial_h
1138: static PyObject * THPVariable_special_hermite_polynomial_h(PyObject* self_, PyObject* args, PyObject* kwargs)
1139: {
1140:   HANDLE_TH_ERRORS
1141:   static PythonArgParser parser({
1142:     "special_hermite_polynomial_h(Tensor x, Tensor n, *, Tensor out=None)",
1143:     "special_hermite_polynomial_h(Scalar x, Tensor n, *, Tensor out=None)",
1144:     "special_hermite_polynomial_h(Tensor x, Scalar n, *, Tensor out=None)",
1145:   }, /*traceable=*/true);
1146: 
1147:   ParsedArgs<3> parsed_args;
1148:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1149:   if(_r.has_torch_function()) {
1150:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
1151:   }
1152:   switch (_r.idx) {
1153:     case 0: {
1154:       if (_r.isNone(2)) {
1155:         // aten::special_hermite_polynomial_h(Tensor x, Tensor n) -> Tensor
1156: 
1157:         auto dispatch_special_hermite_polynomial_h = [](const at::Tensor & x, const at::Tensor & n) -> at::Tensor {
1158:           pybind11::gil_scoped_release no_gil;
1159:           return at::special_hermite_polynomial_h(x, n);
1160:         };
```

- EN: The main execution path in this span is carried by `wrap`, `special_gammaln_out`, `THPVariable_special_hermite_polynomial_h`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `special_gammaln_out`, `THPVariable_special_hermite_polynomial_h` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1161-1200

```cpp
1161:         return wrap(dispatch_special_hermite_polynomial_h(_r.tensor(0), _r.tensor(1)));
1162:       } else {
1163:         // aten::special_hermite_polynomial_h.out(Tensor x, Tensor n, *, Tensor(a!) out) -> Tensor(a!)
1164: 
1165:         auto dispatch_special_hermite_polynomial_h_out = [](at::Tensor out, const at::Tensor & x, const at::Tensor & n) -> at::Tensor {
1166:           pybind11::gil_scoped_release no_gil;
1167:           return at::special_hermite_polynomial_h_out(out, x, n);
1168:         };
1169:         return wrap(dispatch_special_hermite_polynomial_h_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
1170:       }
1171:     }
1172:     case 1: {
1173:       if (_r.isNone(2)) {
1174:         // aten::special_hermite_polynomial_h.x_scalar(Scalar x, Tensor n) -> Tensor
1175: 
1176:         auto dispatch_special_hermite_polynomial_h = [](const at::Scalar & x, const at::Tensor & n) -> at::Tensor {
1177:           pybind11::gil_scoped_release no_gil;
1178:           return at::special_hermite_polynomial_h(x, n);
1179:         };
1180:         return wrap(dispatch_special_hermite_polynomial_h(_r.scalar(0), _r.tensor(1)));
1181:       } else {
1182:         // aten::special_hermite_polynomial_h.x_scalar_out(Scalar x, Tensor n, *, Tensor(a!) out) -> Tensor(a!)
1183: 
1184:         auto dispatch_special_hermite_polynomial_h_out = [](at::Tensor out, const at::Scalar & x, const at::Tensor & n) -> at::Tensor {
1185:           pybind11::gil_scoped_release no_gil;
1186:           return at::special_hermite_polynomial_h_out(out, x, n);
1187:         };
1188:         return wrap(dispatch_special_hermite_polynomial_h_out(_r.tensor(2), _r.scalar(0), _r.tensor(1)));
1189:       }
1190:     }
1191:     case 2: {
1192:       if (_r.isNone(2)) {
1193:         // aten::special_hermite_polynomial_h.n_scalar(Tensor x, Scalar n) -> Tensor
1194: 
1195:         auto dispatch_special_hermite_polynomial_h = [](const at::Tensor & x, const at::Scalar & n) -> at::Tensor {
1196:           pybind11::gil_scoped_release no_gil;
1197:           return at::special_hermite_polynomial_h(x, n);
1198:         };
1199:         return wrap(dispatch_special_hermite_polynomial_h(_r.tensor(0), _r.scalar(1)));
1200:       } else {
```

- EN: The main execution path in this span is carried by `wrap`, `special_hermite_polynomial_h_out`, `special_hermite_polynomial_h`. Control-flow branches in this range handle alternative runtime cases or iterative work. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `special_hermite_polynomial_h_out`, `special_hermite_polynomial_h` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1201-1240

```cpp
1201:         // aten::special_hermite_polynomial_h.n_scalar_out(Tensor x, Scalar n, *, Tensor(a!) out) -> Tensor(a!)
1202: 
1203:         auto dispatch_special_hermite_polynomial_h_out = [](at::Tensor out, const at::Tensor & x, const at::Scalar & n) -> at::Tensor {
1204:           pybind11::gil_scoped_release no_gil;
1205:           return at::special_hermite_polynomial_h_out(out, x, n);
1206:         };
1207:         return wrap(dispatch_special_hermite_polynomial_h_out(_r.tensor(2), _r.tensor(0), _r.scalar(1)));
1208:       }
1209:     }
1210:   }
1211:   Py_RETURN_NONE;
1212:   END_HANDLE_TH_ERRORS
1213: }
1214: 
1215: \
1216: // special_hermite_polynomial_he
1217: static PyObject * THPVariable_special_hermite_polynomial_he(PyObject* self_, PyObject* args, PyObject* kwargs)
1218: {
1219:   HANDLE_TH_ERRORS
1220:   static PythonArgParser parser({
1221:     "special_hermite_polynomial_he(Tensor x, Tensor n, *, Tensor out=None)",
1222:     "special_hermite_polynomial_he(Scalar x, Tensor n, *, Tensor out=None)",
1223:     "special_hermite_polynomial_he(Tensor x, Scalar n, *, Tensor out=None)",
1224:   }, /*traceable=*/true);
1225: 
1226:   ParsedArgs<3> parsed_args;
1227:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1228:   if(_r.has_torch_function()) {
1229:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
1230:   }
1231:   switch (_r.idx) {
1232:     case 0: {
1233:       if (_r.isNone(2)) {
1234:         // aten::special_hermite_polynomial_he(Tensor x, Tensor n) -> Tensor
1235: 
1236:         auto dispatch_special_hermite_polynomial_he = [](const at::Tensor & x, const at::Tensor & n) -> at::Tensor {
1237:           pybind11::gil_scoped_release no_gil;
1238:           return at::special_hermite_polynomial_he(x, n);
1239:         };
1240:         return wrap(dispatch_special_hermite_polynomial_he(_r.tensor(0), _r.tensor(1)));
```

- EN: The main execution path in this span is carried by `special_hermite_polynomial_h_out`, `wrap`, `THPVariable_special_hermite_polynomial_he`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `special_hermite_polynomial_h_out`, `wrap`, `THPVariable_special_hermite_polynomial_he` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1241-1280

```cpp
1241:       } else {
1242:         // aten::special_hermite_polynomial_he.out(Tensor x, Tensor n, *, Tensor(a!) out) -> Tensor(a!)
1243: 
1244:         auto dispatch_special_hermite_polynomial_he_out = [](at::Tensor out, const at::Tensor & x, const at::Tensor & n) -> at::Tensor {
1245:           pybind11::gil_scoped_release no_gil;
1246:           return at::special_hermite_polynomial_he_out(out, x, n);
1247:         };
1248:         return wrap(dispatch_special_hermite_polynomial_he_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
1249:       }
1250:     }
1251:     case 1: {
1252:       if (_r.isNone(2)) {
1253:         // aten::special_hermite_polynomial_he.x_scalar(Scalar x, Tensor n) -> Tensor
1254: 
1255:         auto dispatch_special_hermite_polynomial_he = [](const at::Scalar & x, const at::Tensor & n) -> at::Tensor {
1256:           pybind11::gil_scoped_release no_gil;
1257:           return at::special_hermite_polynomial_he(x, n);
1258:         };
1259:         return wrap(dispatch_special_hermite_polynomial_he(_r.scalar(0), _r.tensor(1)));
1260:       } else {
1261:         // aten::special_hermite_polynomial_he.x_scalar_out(Scalar x, Tensor n, *, Tensor(a!) out) -> Tensor(a!)
1262: 
1263:         auto dispatch_special_hermite_polynomial_he_out = [](at::Tensor out, const at::Scalar & x, const at::Tensor & n) -> at::Tensor {
1264:           pybind11::gil_scoped_release no_gil;
1265:           return at::special_hermite_polynomial_he_out(out, x, n);
1266:         };
1267:         return wrap(dispatch_special_hermite_polynomial_he_out(_r.tensor(2), _r.scalar(0), _r.tensor(1)));
1268:       }
1269:     }
1270:     case 2: {
1271:       if (_r.isNone(2)) {
1272:         // aten::special_hermite_polynomial_he.n_scalar(Tensor x, Scalar n) -> Tensor
1273: 
1274:         auto dispatch_special_hermite_polynomial_he = [](const at::Tensor & x, const at::Scalar & n) -> at::Tensor {
1275:           pybind11::gil_scoped_release no_gil;
1276:           return at::special_hermite_polynomial_he(x, n);
1277:         };
1278:         return wrap(dispatch_special_hermite_polynomial_he(_r.tensor(0), _r.scalar(1)));
1279:       } else {
1280:         // aten::special_hermite_polynomial_he.n_scalar_out(Tensor x, Scalar n, *, Tensor(a!) out) -> Tensor(a!)
```

- EN: The main execution path in this span is carried by `special_hermite_polynomial_he_out`, `wrap`, `special_hermite_polynomial_he`. Control-flow branches in this range handle alternative runtime cases or iterative work. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `special_hermite_polynomial_he_out`, `wrap`, `special_hermite_polynomial_he` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1281-1320

```cpp
1281: 
1282:         auto dispatch_special_hermite_polynomial_he_out = [](at::Tensor out, const at::Tensor & x, const at::Scalar & n) -> at::Tensor {
1283:           pybind11::gil_scoped_release no_gil;
1284:           return at::special_hermite_polynomial_he_out(out, x, n);
1285:         };
1286:         return wrap(dispatch_special_hermite_polynomial_he_out(_r.tensor(2), _r.tensor(0), _r.scalar(1)));
1287:       }
1288:     }
1289:   }
1290:   Py_RETURN_NONE;
1291:   END_HANDLE_TH_ERRORS
1292: }
1293: 
1294: // special_i0
1295: static PyObject * THPVariable_special_i0(PyObject* self_, PyObject* args, PyObject* kwargs)
1296: {
1297:   HANDLE_TH_ERRORS
1298:   static PythonArgParser parser({
1299:     "special_i0(Tensor input, *, Tensor out=None)",
1300:   }, /*traceable=*/true);
1301: 
1302:   ParsedArgs<2> parsed_args;
1303:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1304:   if(_r.has_torch_function()) {
1305:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
1306:   }
1307:   if (_r.isNone(1)) {
1308:     // aten::special_i0(Tensor self) -> Tensor
1309: 
1310:     auto dispatch_special_i0 = [](const at::Tensor & self) -> at::Tensor {
1311:       pybind11::gil_scoped_release no_gil;
1312:       return at::special_i0(self);
1313:     };
1314:     return wrap(dispatch_special_i0(_r.tensor(0)));
1315:   } else {
1316:     // aten::special_i0.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
1317: 
1318:     auto dispatch_special_i0_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
1319:       pybind11::gil_scoped_release no_gil;
1320:       return at::special_i0_out(out, self);
```

- EN: The main execution path in this span is carried by `special_hermite_polynomial_he_out`, `wrap`, `THPVariable_special_i0`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `special_hermite_polynomial_he_out`, `wrap`, `THPVariable_special_i0` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1321-1360

```cpp
1321:     };
1322:     return wrap(dispatch_special_i0_out(_r.tensor(1), _r.tensor(0)));
1323:   }
1324:   Py_RETURN_NONE;
1325:   END_HANDLE_TH_ERRORS
1326: }
1327: 
1328: // special_i0e
1329: static PyObject * THPVariable_special_i0e(PyObject* self_, PyObject* args, PyObject* kwargs)
1330: {
1331:   HANDLE_TH_ERRORS
1332:   static PythonArgParser parser({
1333:     "special_i0e(Tensor input, *, Tensor out=None)",
1334:   }, /*traceable=*/true);
1335: 
1336:   ParsedArgs<2> parsed_args;
1337:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1338:   if(_r.has_torch_function()) {
1339:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
1340:   }
1341:   if (_r.isNone(1)) {
1342:     // aten::special_i0e(Tensor self) -> Tensor
1343: 
1344:     auto dispatch_special_i0e = [](const at::Tensor & self) -> at::Tensor {
1345:       pybind11::gil_scoped_release no_gil;
1346:       return at::special_i0e(self);
1347:     };
1348:     return wrap(dispatch_special_i0e(_r.tensor(0)));
1349:   } else {
1350:     // aten::special_i0e.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
1351: 
1352:     auto dispatch_special_i0e_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
1353:       pybind11::gil_scoped_release no_gil;
1354:       return at::special_i0e_out(out, self);
1355:     };
1356:     return wrap(dispatch_special_i0e_out(_r.tensor(1), _r.tensor(0)));
1357:   }
1358:   Py_RETURN_NONE;
1359:   END_HANDLE_TH_ERRORS
1360: }
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_special_i0e`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_special_i0e`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1361-1400

```cpp
1361: 
1362: // special_i1
1363: static PyObject * THPVariable_special_i1(PyObject* self_, PyObject* args, PyObject* kwargs)
1364: {
1365:   HANDLE_TH_ERRORS
1366:   static PythonArgParser parser({
1367:     "special_i1(Tensor input, *, Tensor out=None)",
1368:   }, /*traceable=*/true);
1369: 
1370:   ParsedArgs<2> parsed_args;
1371:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1372:   if(_r.has_torch_function()) {
1373:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
1374:   }
1375:   if (_r.isNone(1)) {
1376:     // aten::special_i1(Tensor self) -> Tensor
1377: 
1378:     auto dispatch_special_i1 = [](const at::Tensor & self) -> at::Tensor {
1379:       pybind11::gil_scoped_release no_gil;
1380:       return at::special_i1(self);
1381:     };
1382:     return wrap(dispatch_special_i1(_r.tensor(0)));
1383:   } else {
1384:     // aten::special_i1.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
1385: 
1386:     auto dispatch_special_i1_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
1387:       pybind11::gil_scoped_release no_gil;
1388:       return at::special_i1_out(out, self);
1389:     };
1390:     return wrap(dispatch_special_i1_out(_r.tensor(1), _r.tensor(0)));
1391:   }
1392:   Py_RETURN_NONE;
1393:   END_HANDLE_TH_ERRORS
1394: }
1395: 
1396: // special_i1e
1397: static PyObject * THPVariable_special_i1e(PyObject* self_, PyObject* args, PyObject* kwargs)
1398: {
1399:   HANDLE_TH_ERRORS
1400:   static PythonArgParser parser({
```

- EN: The main execution path in this span is carried by `THPVariable_special_i1`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_special_i1`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1401-1440

```cpp
1401:     "special_i1e(Tensor input, *, Tensor out=None)",
1402:   }, /*traceable=*/true);
1403: 
1404:   ParsedArgs<2> parsed_args;
1405:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1406:   if(_r.has_torch_function()) {
1407:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
1408:   }
1409:   if (_r.isNone(1)) {
1410:     // aten::special_i1e(Tensor self) -> Tensor
1411: 
1412:     auto dispatch_special_i1e = [](const at::Tensor & self) -> at::Tensor {
1413:       pybind11::gil_scoped_release no_gil;
1414:       return at::special_i1e(self);
1415:     };
1416:     return wrap(dispatch_special_i1e(_r.tensor(0)));
1417:   } else {
1418:     // aten::special_i1e.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
1419: 
1420:     auto dispatch_special_i1e_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
1421:       pybind11::gil_scoped_release no_gil;
1422:       return at::special_i1e_out(out, self);
1423:     };
1424:     return wrap(dispatch_special_i1e_out(_r.tensor(1), _r.tensor(0)));
1425:   }
1426:   Py_RETURN_NONE;
1427:   END_HANDLE_TH_ERRORS
1428: }
1429: 
1430: \
1431: // special_laguerre_polynomial_l
1432: static PyObject * THPVariable_special_laguerre_polynomial_l(PyObject* self_, PyObject* args, PyObject* kwargs)
1433: {
1434:   HANDLE_TH_ERRORS
1435:   static PythonArgParser parser({
1436:     "special_laguerre_polynomial_l(Tensor x, Tensor n, *, Tensor out=None)",
1437:     "special_laguerre_polynomial_l(Scalar x, Tensor n, *, Tensor out=None)",
1438:     "special_laguerre_polynomial_l(Tensor x, Scalar n, *, Tensor out=None)",
1439:   }, /*traceable=*/true);
1440: 
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `special_i1e`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `special_i1e`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1441-1480

```cpp
1441:   ParsedArgs<3> parsed_args;
1442:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1443:   if(_r.has_torch_function()) {
1444:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
1445:   }
1446:   switch (_r.idx) {
1447:     case 0: {
1448:       if (_r.isNone(2)) {
1449:         // aten::special_laguerre_polynomial_l(Tensor x, Tensor n) -> Tensor
1450: 
1451:         auto dispatch_special_laguerre_polynomial_l = [](const at::Tensor & x, const at::Tensor & n) -> at::Tensor {
1452:           pybind11::gil_scoped_release no_gil;
1453:           return at::special_laguerre_polynomial_l(x, n);
1454:         };
1455:         return wrap(dispatch_special_laguerre_polynomial_l(_r.tensor(0), _r.tensor(1)));
1456:       } else {
1457:         // aten::special_laguerre_polynomial_l.out(Tensor x, Tensor n, *, Tensor(a!) out) -> Tensor(a!)
1458: 
1459:         auto dispatch_special_laguerre_polynomial_l_out = [](at::Tensor out, const at::Tensor & x, const at::Tensor & n) -> at::Tensor {
1460:           pybind11::gil_scoped_release no_gil;
1461:           return at::special_laguerre_polynomial_l_out(out, x, n);
1462:         };
1463:         return wrap(dispatch_special_laguerre_polynomial_l_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
1464:       }
1465:     }
1466:     case 1: {
1467:       if (_r.isNone(2)) {
1468:         // aten::special_laguerre_polynomial_l.x_scalar(Scalar x, Tensor n) -> Tensor
1469: 
1470:         auto dispatch_special_laguerre_polynomial_l = [](const at::Scalar & x, const at::Tensor & n) -> at::Tensor {
1471:           pybind11::gil_scoped_release no_gil;
1472:           return at::special_laguerre_polynomial_l(x, n);
1473:         };
1474:         return wrap(dispatch_special_laguerre_polynomial_l(_r.scalar(0), _r.tensor(1)));
1475:       } else {
1476:         // aten::special_laguerre_polynomial_l.x_scalar_out(Scalar x, Tensor n, *, Tensor(a!) out) -> Tensor(a!)
1477: 
1478:         auto dispatch_special_laguerre_polynomial_l_out = [](at::Tensor out, const at::Scalar & x, const at::Tensor & n) -> at::Tensor {
1479:           pybind11::gil_scoped_release no_gil;
1480:           return at::special_laguerre_polynomial_l_out(out, x, n);
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `special_laguerre_polynomial_l`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `special_laguerre_polynomial_l`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1481-1520

```cpp
1481:         };
1482:         return wrap(dispatch_special_laguerre_polynomial_l_out(_r.tensor(2), _r.scalar(0), _r.tensor(1)));
1483:       }
1484:     }
1485:     case 2: {
1486:       if (_r.isNone(2)) {
1487:         // aten::special_laguerre_polynomial_l.n_scalar(Tensor x, Scalar n) -> Tensor
1488: 
1489:         auto dispatch_special_laguerre_polynomial_l = [](const at::Tensor & x, const at::Scalar & n) -> at::Tensor {
1490:           pybind11::gil_scoped_release no_gil;
1491:           return at::special_laguerre_polynomial_l(x, n);
1492:         };
1493:         return wrap(dispatch_special_laguerre_polynomial_l(_r.tensor(0), _r.scalar(1)));
1494:       } else {
1495:         // aten::special_laguerre_polynomial_l.n_scalar_out(Tensor x, Scalar n, *, Tensor(a!) out) -> Tensor(a!)
1496: 
1497:         auto dispatch_special_laguerre_polynomial_l_out = [](at::Tensor out, const at::Tensor & x, const at::Scalar & n) -> at::Tensor {
1498:           pybind11::gil_scoped_release no_gil;
1499:           return at::special_laguerre_polynomial_l_out(out, x, n);
1500:         };
1501:         return wrap(dispatch_special_laguerre_polynomial_l_out(_r.tensor(2), _r.tensor(0), _r.scalar(1)));
1502:       }
1503:     }
1504:   }
1505:   Py_RETURN_NONE;
1506:   END_HANDLE_TH_ERRORS
1507: }
1508: 
1509: \
1510: // special_legendre_polynomial_p
1511: static PyObject * THPVariable_special_legendre_polynomial_p(PyObject* self_, PyObject* args, PyObject* kwargs)
1512: {
1513:   HANDLE_TH_ERRORS
1514:   static PythonArgParser parser({
1515:     "special_legendre_polynomial_p(Tensor x, Tensor n, *, Tensor out=None)",
1516:     "special_legendre_polynomial_p(Scalar x, Tensor n, *, Tensor out=None)",
1517:     "special_legendre_polynomial_p(Tensor x, Scalar n, *, Tensor out=None)",
1518:   }, /*traceable=*/true);
1519: 
1520:   ParsedArgs<3> parsed_args;
```

- EN: The main execution path in this span is carried by `wrap`, `special_laguerre_polynomial_l`, `special_laguerre_polynomial_l_out`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `special_laguerre_polynomial_l`, `special_laguerre_polynomial_l_out` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1521-1560

```cpp
1521:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1522:   if(_r.has_torch_function()) {
1523:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
1524:   }
1525:   switch (_r.idx) {
1526:     case 0: {
1527:       if (_r.isNone(2)) {
1528:         // aten::special_legendre_polynomial_p(Tensor x, Tensor n) -> Tensor
1529: 
1530:         auto dispatch_special_legendre_polynomial_p = [](const at::Tensor & x, const at::Tensor & n) -> at::Tensor {
1531:           pybind11::gil_scoped_release no_gil;
1532:           return at::special_legendre_polynomial_p(x, n);
1533:         };
1534:         return wrap(dispatch_special_legendre_polynomial_p(_r.tensor(0), _r.tensor(1)));
1535:       } else {
1536:         // aten::special_legendre_polynomial_p.out(Tensor x, Tensor n, *, Tensor(a!) out) -> Tensor(a!)
1537: 
1538:         auto dispatch_special_legendre_polynomial_p_out = [](at::Tensor out, const at::Tensor & x, const at::Tensor & n) -> at::Tensor {
1539:           pybind11::gil_scoped_release no_gil;
1540:           return at::special_legendre_polynomial_p_out(out, x, n);
1541:         };
1542:         return wrap(dispatch_special_legendre_polynomial_p_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
1543:       }
1544:     }
1545:     case 1: {
1546:       if (_r.isNone(2)) {
1547:         // aten::special_legendre_polynomial_p.x_scalar(Scalar x, Tensor n) -> Tensor
1548: 
1549:         auto dispatch_special_legendre_polynomial_p = [](const at::Scalar & x, const at::Tensor & n) -> at::Tensor {
1550:           pybind11::gil_scoped_release no_gil;
1551:           return at::special_legendre_polynomial_p(x, n);
1552:         };
1553:         return wrap(dispatch_special_legendre_polynomial_p(_r.scalar(0), _r.tensor(1)));
1554:       } else {
1555:         // aten::special_legendre_polynomial_p.x_scalar_out(Scalar x, Tensor n, *, Tensor(a!) out) -> Tensor(a!)
1556: 
1557:         auto dispatch_special_legendre_polynomial_p_out = [](at::Tensor out, const at::Scalar & x, const at::Tensor & n) -> at::Tensor {
1558:           pybind11::gil_scoped_release no_gil;
1559:           return at::special_legendre_polynomial_p_out(out, x, n);
1560:         };
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `special_legendre_polynomial_p`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `special_legendre_polynomial_p`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1561-1600

```cpp
1561:         return wrap(dispatch_special_legendre_polynomial_p_out(_r.tensor(2), _r.scalar(0), _r.tensor(1)));
1562:       }
1563:     }
1564:     case 2: {
1565:       if (_r.isNone(2)) {
1566:         // aten::special_legendre_polynomial_p.n_scalar(Tensor x, Scalar n) -> Tensor
1567: 
1568:         auto dispatch_special_legendre_polynomial_p = [](const at::Tensor & x, const at::Scalar & n) -> at::Tensor {
1569:           pybind11::gil_scoped_release no_gil;
1570:           return at::special_legendre_polynomial_p(x, n);
1571:         };
1572:         return wrap(dispatch_special_legendre_polynomial_p(_r.tensor(0), _r.scalar(1)));
1573:       } else {
1574:         // aten::special_legendre_polynomial_p.n_scalar_out(Tensor x, Scalar n, *, Tensor(a!) out) -> Tensor(a!)
1575: 
1576:         auto dispatch_special_legendre_polynomial_p_out = [](at::Tensor out, const at::Tensor & x, const at::Scalar & n) -> at::Tensor {
1577:           pybind11::gil_scoped_release no_gil;
1578:           return at::special_legendre_polynomial_p_out(out, x, n);
1579:         };
1580:         return wrap(dispatch_special_legendre_polynomial_p_out(_r.tensor(2), _r.tensor(0), _r.scalar(1)));
1581:       }
1582:     }
1583:   }
1584:   Py_RETURN_NONE;
1585:   END_HANDLE_TH_ERRORS
1586: }
1587: 
1588: // special_log1p
1589: static PyObject * THPVariable_special_log1p(PyObject* self_, PyObject* args, PyObject* kwargs)
1590: {
1591:   HANDLE_TH_ERRORS
1592:   static PythonArgParser parser({
1593:     "special_log1p(Tensor input, *, Tensor out=None)",
1594:   }, /*traceable=*/true);
1595: 
1596:   ParsedArgs<2> parsed_args;
1597:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1598:   if(_r.has_torch_function()) {
1599:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
1600:   }
```

- EN: The main execution path in this span is carried by `wrap`, `special_legendre_polynomial_p`, `special_legendre_polynomial_p_out`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `special_legendre_polynomial_p`, `special_legendre_polynomial_p_out` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1601-1640

```cpp
1601:   if (_r.isNone(1)) {
1602:     // aten::special_log1p(Tensor self) -> Tensor
1603: 
1604:     auto dispatch_special_log1p = [](const at::Tensor & self) -> at::Tensor {
1605:       pybind11::gil_scoped_release no_gil;
1606:       return at::special_log1p(self);
1607:     };
1608:     return wrap(dispatch_special_log1p(_r.tensor(0)));
1609:   } else {
1610:     // aten::special_log1p.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
1611: 
1612:     auto dispatch_special_log1p_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
1613:       pybind11::gil_scoped_release no_gil;
1614:       return at::special_log1p_out(out, self);
1615:     };
1616:     return wrap(dispatch_special_log1p_out(_r.tensor(1), _r.tensor(0)));
1617:   }
1618:   Py_RETURN_NONE;
1619:   END_HANDLE_TH_ERRORS
1620: }
1621: 
1622: // special_log_ndtr
1623: static PyObject * THPVariable_special_log_ndtr(PyObject* self_, PyObject* args, PyObject* kwargs)
1624: {
1625:   HANDLE_TH_ERRORS
1626:   static PythonArgParser parser({
1627:     "special_log_ndtr(Tensor input, *, Tensor out=None)",
1628:   }, /*traceable=*/true);
1629: 
1630:   ParsedArgs<2> parsed_args;
1631:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1632:   if(_r.has_torch_function()) {
1633:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
1634:   }
1635:   if (_r.isNone(1)) {
1636:     // aten::special_log_ndtr(Tensor self) -> Tensor
1637: 
1638:     auto dispatch_special_log_ndtr = [](const at::Tensor & self) -> at::Tensor {
1639:       pybind11::gil_scoped_release no_gil;
1640:       return at::special_log_ndtr(self);
```

- EN: The main execution path in this span is carried by `special_log1p`, `wrap`, `special_log1p_out`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `special_log1p`, `wrap`, `special_log1p_out` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1641-1680

```cpp
1641:     };
1642:     return wrap(dispatch_special_log_ndtr(_r.tensor(0)));
1643:   } else {
1644:     // aten::special_log_ndtr.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
1645: 
1646:     auto dispatch_special_log_ndtr_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
1647:       pybind11::gil_scoped_release no_gil;
1648:       return at::special_log_ndtr_out(out, self);
1649:     };
1650:     return wrap(dispatch_special_log_ndtr_out(_r.tensor(1), _r.tensor(0)));
1651:   }
1652:   Py_RETURN_NONE;
1653:   END_HANDLE_TH_ERRORS
1654: }
1655: 
1656: // special_log_softmax
1657: static PyObject * THPVariable_special_log_softmax(PyObject* self_, PyObject* args, PyObject* kwargs)
1658: {
1659:   HANDLE_TH_ERRORS
1660:   static PythonArgParser parser({
1661:     "special_log_softmax(Tensor input, int64_t dim, *, ScalarType? dtype=None)",
1662:   }, /*traceable=*/true);
1663: 
1664:   ParsedArgs<3> parsed_args;
1665:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1666:   if(_r.has_torch_function()) {
1667:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
1668:   }
1669:   // aten::special_log_softmax(Tensor self, int dim, *, ScalarType? dtype=None) -> Tensor
1670: 
1671:   auto dispatch_special_log_softmax = [](const at::Tensor & self, int64_t dim, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
1672:     pybind11::gil_scoped_release no_gil;
1673:     return at::special_log_softmax(self, dim, dtype);
1674:   };
1675:   return wrap(dispatch_special_log_softmax(_r.tensor(0), _r.toInt64(1), _r.scalartypeOptional(2)));
1676:   Py_RETURN_NONE;
1677:   END_HANDLE_TH_ERRORS
1678: }
1679: 
1680: // special_logit
```

- EN: The main execution path in this span is carried by `wrap`, `special_log_ndtr_out`, `THPVariable_special_log_softmax`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `special_log_ndtr_out`, `THPVariable_special_log_softmax` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1681-1720

```cpp
1681: static PyObject * THPVariable_special_logit(PyObject* self_, PyObject* args, PyObject* kwargs)
1682: {
1683:   HANDLE_TH_ERRORS
1684:   static PythonArgParser parser({
1685:     "special_logit(Tensor input, double? eps=None, *, Tensor out=None)",
1686:   }, /*traceable=*/true);
1687: 
1688:   ParsedArgs<3> parsed_args;
1689:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1690:   if(_r.has_torch_function()) {
1691:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
1692:   }
1693:   if (_r.isNone(2)) {
1694:     // aten::special_logit(Tensor self, float? eps=None) -> Tensor
1695: 
1696:     auto dispatch_special_logit = [](const at::Tensor & self, ::std::optional<double> eps) -> at::Tensor {
1697:       pybind11::gil_scoped_release no_gil;
1698:       return at::special_logit(self, eps);
1699:     };
1700:     return wrap(dispatch_special_logit(_r.tensor(0), _r.toDoubleOptional(1)));
1701:   } else {
1702:     // aten::special_logit.out(Tensor self, float? eps=None, *, Tensor(a!) out) -> Tensor(a!)
1703: 
1704:     auto dispatch_special_logit_out = [](at::Tensor out, const at::Tensor & self, ::std::optional<double> eps) -> at::Tensor {
1705:       pybind11::gil_scoped_release no_gil;
1706:       return at::special_logit_out(out, self, eps);
1707:     };
1708:     return wrap(dispatch_special_logit_out(_r.tensor(2), _r.tensor(0), _r.toDoubleOptional(1)));
1709:   }
1710:   Py_RETURN_NONE;
1711:   END_HANDLE_TH_ERRORS
1712: }
1713: 
1714: // special_logsumexp
1715: static PyObject * THPVariable_special_logsumexp(PyObject* self_, PyObject* args, PyObject* kwargs)
1716: {
1717:   HANDLE_TH_ERRORS
1718:   static PythonArgParser parser({
1719:     "special_logsumexp(Tensor input, IntArrayRef[1] dim, bool keepdim=False, *, Tensor out=None)",
1720:   }, /*traceable=*/true);
```

- EN: The main execution path in this span is carried by `THPVariable_special_logit`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_special_logit`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 1721-1760

```cpp
1721: 
1722:   ParsedArgs<4> parsed_args;
1723:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1724:   if(_r.has_torch_function()) {
1725:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
1726:   }
1727:   if (_r.isNone(3)) {
1728:     // aten::special_logsumexp(Tensor self, int[1] dim, bool keepdim=False) -> Tensor
1729: 
1730:     auto dispatch_special_logsumexp = [](const at::Tensor & self, at::IntArrayRef dim, bool keepdim) -> at::Tensor {
1731:       pybind11::gil_scoped_release no_gil;
1732:       return at::special_logsumexp(self, dim, keepdim);
1733:     };
1734:     return wrap(dispatch_special_logsumexp(_r.tensor(0), _r.intlist(1), _r.toBool(2)));
1735:   } else {
1736:     // aten::special_logsumexp.out(Tensor self, int[1] dim, bool keepdim=False, *, Tensor(a!) out) -> Tensor(a!)
1737: 
1738:     auto dispatch_special_logsumexp_out = [](at::Tensor out, const at::Tensor & self, at::IntArrayRef dim, bool keepdim) -> at::Tensor {
1739:       pybind11::gil_scoped_release no_gil;
1740:       return at::special_logsumexp_out(out, self, dim, keepdim);
1741:     };
1742:     return wrap(dispatch_special_logsumexp_out(_r.tensor(3), _r.tensor(0), _r.intlist(1), _r.toBool(2)));
1743:   }
1744:   Py_RETURN_NONE;
1745:   END_HANDLE_TH_ERRORS
1746: }
1747: 
1748: // special_modified_bessel_i0
1749: static PyObject * THPVariable_special_modified_bessel_i0(PyObject* self_, PyObject* args, PyObject* kwargs)
1750: {
1751:   HANDLE_TH_ERRORS
1752:   static PythonArgParser parser({
1753:     "special_modified_bessel_i0(Tensor input, *, Tensor out=None)",
1754:   }, /*traceable=*/true);
1755: 
1756:   ParsedArgs<2> parsed_args;
1757:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1758:   if(_r.has_torch_function()) {
1759:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
1760:   }
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `special_logsumexp`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `special_logsumexp`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 1761-1800

```cpp
1761:   if (_r.isNone(1)) {
1762:     // aten::special_modified_bessel_i0(Tensor self) -> Tensor
1763: 
1764:     auto dispatch_special_modified_bessel_i0 = [](const at::Tensor & self) -> at::Tensor {
1765:       pybind11::gil_scoped_release no_gil;
1766:       return at::special_modified_bessel_i0(self);
1767:     };
1768:     return wrap(dispatch_special_modified_bessel_i0(_r.tensor(0)));
1769:   } else {
1770:     // aten::special_modified_bessel_i0.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
1771: 
1772:     auto dispatch_special_modified_bessel_i0_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
1773:       pybind11::gil_scoped_release no_gil;
1774:       return at::special_modified_bessel_i0_out(out, self);
1775:     };
1776:     return wrap(dispatch_special_modified_bessel_i0_out(_r.tensor(1), _r.tensor(0)));
1777:   }
1778:   Py_RETURN_NONE;
1779:   END_HANDLE_TH_ERRORS
1780: }
1781: 
1782: // special_modified_bessel_i1
1783: static PyObject * THPVariable_special_modified_bessel_i1(PyObject* self_, PyObject* args, PyObject* kwargs)
1784: {
1785:   HANDLE_TH_ERRORS
1786:   static PythonArgParser parser({
1787:     "special_modified_bessel_i1(Tensor input, *, Tensor out=None)",
1788:   }, /*traceable=*/true);
1789: 
1790:   ParsedArgs<2> parsed_args;
1791:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1792:   if(_r.has_torch_function()) {
1793:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
1794:   }
1795:   if (_r.isNone(1)) {
1796:     // aten::special_modified_bessel_i1(Tensor self) -> Tensor
1797: 
1798:     auto dispatch_special_modified_bessel_i1 = [](const at::Tensor & self) -> at::Tensor {
1799:       pybind11::gil_scoped_release no_gil;
1800:       return at::special_modified_bessel_i1(self);
```

- EN: The main execution path in this span is carried by `special_modified_bessel_i0`, `wrap`, `special_modified_bessel_i0_out`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `special_modified_bessel_i0`, `wrap`, `special_modified_bessel_i0_out` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1801-1840

```cpp
1801:     };
1802:     return wrap(dispatch_special_modified_bessel_i1(_r.tensor(0)));
1803:   } else {
1804:     // aten::special_modified_bessel_i1.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
1805: 
1806:     auto dispatch_special_modified_bessel_i1_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
1807:       pybind11::gil_scoped_release no_gil;
1808:       return at::special_modified_bessel_i1_out(out, self);
1809:     };
1810:     return wrap(dispatch_special_modified_bessel_i1_out(_r.tensor(1), _r.tensor(0)));
1811:   }
1812:   Py_RETURN_NONE;
1813:   END_HANDLE_TH_ERRORS
1814: }
1815: 
1816: // special_modified_bessel_k0
1817: static PyObject * THPVariable_special_modified_bessel_k0(PyObject* self_, PyObject* args, PyObject* kwargs)
1818: {
1819:   HANDLE_TH_ERRORS
1820:   static PythonArgParser parser({
1821:     "special_modified_bessel_k0(Tensor input, *, Tensor out=None)",
1822:   }, /*traceable=*/true);
1823: 
1824:   ParsedArgs<2> parsed_args;
1825:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1826:   if(_r.has_torch_function()) {
1827:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
1828:   }
1829:   if (_r.isNone(1)) {
1830:     // aten::special_modified_bessel_k0(Tensor self) -> Tensor
1831: 
1832:     auto dispatch_special_modified_bessel_k0 = [](const at::Tensor & self) -> at::Tensor {
1833:       pybind11::gil_scoped_release no_gil;
1834:       return at::special_modified_bessel_k0(self);
1835:     };
1836:     return wrap(dispatch_special_modified_bessel_k0(_r.tensor(0)));
1837:   } else {
1838:     // aten::special_modified_bessel_k0.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
1839: 
1840:     auto dispatch_special_modified_bessel_k0_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
```

- EN: The main execution path in this span is carried by `wrap`, `special_modified_bessel_i1_out`, `THPVariable_special_modified_bessel_k0`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `special_modified_bessel_i1_out`, `THPVariable_special_modified_bessel_k0` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1841-1880

```cpp
1841:       pybind11::gil_scoped_release no_gil;
1842:       return at::special_modified_bessel_k0_out(out, self);
1843:     };
1844:     return wrap(dispatch_special_modified_bessel_k0_out(_r.tensor(1), _r.tensor(0)));
1845:   }
1846:   Py_RETURN_NONE;
1847:   END_HANDLE_TH_ERRORS
1848: }
1849: 
1850: // special_modified_bessel_k1
1851: static PyObject * THPVariable_special_modified_bessel_k1(PyObject* self_, PyObject* args, PyObject* kwargs)
1852: {
1853:   HANDLE_TH_ERRORS
1854:   static PythonArgParser parser({
1855:     "special_modified_bessel_k1(Tensor input, *, Tensor out=None)",
1856:   }, /*traceable=*/true);
1857: 
1858:   ParsedArgs<2> parsed_args;
1859:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1860:   if(_r.has_torch_function()) {
1861:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
1862:   }
1863:   if (_r.isNone(1)) {
1864:     // aten::special_modified_bessel_k1(Tensor self) -> Tensor
1865: 
1866:     auto dispatch_special_modified_bessel_k1 = [](const at::Tensor & self) -> at::Tensor {
1867:       pybind11::gil_scoped_release no_gil;
1868:       return at::special_modified_bessel_k1(self);
1869:     };
1870:     return wrap(dispatch_special_modified_bessel_k1(_r.tensor(0)));
1871:   } else {
1872:     // aten::special_modified_bessel_k1.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
1873: 
1874:     auto dispatch_special_modified_bessel_k1_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
1875:       pybind11::gil_scoped_release no_gil;
1876:       return at::special_modified_bessel_k1_out(out, self);
1877:     };
1878:     return wrap(dispatch_special_modified_bessel_k1_out(_r.tensor(1), _r.tensor(0)));
1879:   }
1880:   Py_RETURN_NONE;
```

- EN: The main execution path in this span is carried by `special_modified_bessel_k0_out`, `wrap`, `THPVariable_special_modified_bessel_k1`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `special_modified_bessel_k0_out`, `wrap`, `THPVariable_special_modified_bessel_k1` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1881-1920

```cpp
1881:   END_HANDLE_TH_ERRORS
1882: }
1883: 
1884: // special_multigammaln
1885: static PyObject * THPVariable_special_multigammaln(PyObject* self_, PyObject* args, PyObject* kwargs)
1886: {
1887:   HANDLE_TH_ERRORS
1888:   static PythonArgParser parser({
1889:     "special_multigammaln(Tensor input, int64_t p, *, Tensor out=None)",
1890:   }, /*traceable=*/true);
1891: 
1892:   ParsedArgs<3> parsed_args;
1893:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1894:   if(_r.has_torch_function()) {
1895:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
1896:   }
1897:   if (_r.isNone(2)) {
1898:     // aten::special_multigammaln(Tensor self, int p) -> Tensor
1899: 
1900:     auto dispatch_special_multigammaln = [](const at::Tensor & self, int64_t p) -> at::Tensor {
1901:       pybind11::gil_scoped_release no_gil;
1902:       return at::special_multigammaln(self, p);
1903:     };
1904:     return wrap(dispatch_special_multigammaln(_r.tensor(0), _r.toInt64(1)));
1905:   } else {
1906:     // aten::special_multigammaln.out(Tensor self, int p, *, Tensor(a!) out) -> Tensor(a!)
1907: 
1908:     auto dispatch_special_multigammaln_out = [](at::Tensor out, const at::Tensor & self, int64_t p) -> at::Tensor {
1909:       pybind11::gil_scoped_release no_gil;
1910:       return at::special_multigammaln_out(out, self, p);
1911:     };
1912:     return wrap(dispatch_special_multigammaln_out(_r.tensor(2), _r.tensor(0), _r.toInt64(1)));
1913:   }
1914:   Py_RETURN_NONE;
1915:   END_HANDLE_TH_ERRORS
1916: }
1917: 
1918: // special_ndtr
1919: static PyObject * THPVariable_special_ndtr(PyObject* self_, PyObject* args, PyObject* kwargs)
1920: {
```

- EN: The main execution path in this span is carried by `THPVariable_special_multigammaln`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_special_multigammaln`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1921-1960

```cpp
1921:   HANDLE_TH_ERRORS
1922:   static PythonArgParser parser({
1923:     "special_ndtr(Tensor input, *, Tensor out=None)",
1924:   }, /*traceable=*/true);
1925: 
1926:   ParsedArgs<2> parsed_args;
1927:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1928:   if(_r.has_torch_function()) {
1929:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
1930:   }
1931:   if (_r.isNone(1)) {
1932:     // aten::special_ndtr(Tensor self) -> Tensor
1933: 
1934:     auto dispatch_special_ndtr = [](const at::Tensor & self) -> at::Tensor {
1935:       pybind11::gil_scoped_release no_gil;
1936:       return at::special_ndtr(self);
1937:     };
1938:     return wrap(dispatch_special_ndtr(_r.tensor(0)));
1939:   } else {
1940:     // aten::special_ndtr.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
1941: 
1942:     auto dispatch_special_ndtr_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
1943:       pybind11::gil_scoped_release no_gil;
1944:       return at::special_ndtr_out(out, self);
1945:     };
1946:     return wrap(dispatch_special_ndtr_out(_r.tensor(1), _r.tensor(0)));
1947:   }
1948:   Py_RETURN_NONE;
1949:   END_HANDLE_TH_ERRORS
1950: }
1951: 
1952: // special_ndtri
1953: static PyObject * THPVariable_special_ndtri(PyObject* self_, PyObject* args, PyObject* kwargs)
1954: {
1955:   HANDLE_TH_ERRORS
1956:   static PythonArgParser parser({
1957:     "special_ndtri(Tensor input, *, Tensor out=None)",
1958:   }, /*traceable=*/true);
1959: 
1960:   ParsedArgs<2> parsed_args;
```

- EN: The main execution path in this span is carried by `parser`, `handle_torch_function`, `special_ndtr`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `parser`, `handle_torch_function`, `special_ndtr` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1961-2000

```cpp
1961:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1962:   if(_r.has_torch_function()) {
1963:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
1964:   }
1965:   if (_r.isNone(1)) {
1966:     // aten::special_ndtri(Tensor self) -> Tensor
1967: 
1968:     auto dispatch_special_ndtri = [](const at::Tensor & self) -> at::Tensor {
1969:       pybind11::gil_scoped_release no_gil;
1970:       return at::special_ndtri(self);
1971:     };
1972:     return wrap(dispatch_special_ndtri(_r.tensor(0)));
1973:   } else {
1974:     // aten::special_ndtri.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
1975: 
1976:     auto dispatch_special_ndtri_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
1977:       pybind11::gil_scoped_release no_gil;
1978:       return at::special_ndtri_out(out, self);
1979:     };
1980:     return wrap(dispatch_special_ndtri_out(_r.tensor(1), _r.tensor(0)));
1981:   }
1982:   Py_RETURN_NONE;
1983:   END_HANDLE_TH_ERRORS
1984: }
1985: 
1986: // special_polygamma
1987: static PyObject * THPVariable_special_polygamma(PyObject* self_, PyObject* args, PyObject* kwargs)
1988: {
1989:   HANDLE_TH_ERRORS
1990:   static PythonArgParser parser({
1991:     "special_polygamma(int64_t n, Tensor input, *, Tensor out=None)",
1992:   }, /*traceable=*/true);
1993: 
1994:   ParsedArgs<3> parsed_args;
1995:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1996:   if(_r.has_torch_function()) {
1997:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
1998:   }
1999:   if (_r.isNone(2)) {
2000:     // aten::special_polygamma(int n, Tensor self) -> Tensor
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `special_ndtri`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `special_ndtri`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2001-2040

```cpp
2001: 
2002:     auto dispatch_special_polygamma = [](int64_t n, const at::Tensor & self) -> at::Tensor {
2003:       pybind11::gil_scoped_release no_gil;
2004:       return at::special_polygamma(n, self);
2005:     };
2006:     return wrap(dispatch_special_polygamma(_r.toInt64(0), _r.tensor(1)));
2007:   } else {
2008:     // aten::special_polygamma.out(int n, Tensor self, *, Tensor(a!) out) -> Tensor(a!)
2009: 
2010:     auto dispatch_special_polygamma_out = [](at::Tensor out, int64_t n, const at::Tensor & self) -> at::Tensor {
2011:       pybind11::gil_scoped_release no_gil;
2012:       return at::special_polygamma_out(out, n, self);
2013:     };
2014:     return wrap(dispatch_special_polygamma_out(_r.tensor(2), _r.toInt64(0), _r.tensor(1)));
2015:   }
2016:   Py_RETURN_NONE;
2017:   END_HANDLE_TH_ERRORS
2018: }
2019: 
2020: // special_psi
2021: static PyObject * THPVariable_special_psi(PyObject* self_, PyObject* args, PyObject* kwargs)
2022: {
2023:   HANDLE_TH_ERRORS
2024:   static PythonArgParser parser({
2025:     "special_psi(Tensor input, *, Tensor out=None)",
2026:   }, /*traceable=*/true);
2027: 
2028:   ParsedArgs<2> parsed_args;
2029:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2030:   if(_r.has_torch_function()) {
2031:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
2032:   }
2033:   if (_r.isNone(1)) {
2034:     // aten::special_psi(Tensor self) -> Tensor
2035: 
2036:     auto dispatch_special_psi = [](const at::Tensor & self) -> at::Tensor {
2037:       pybind11::gil_scoped_release no_gil;
2038:       return at::special_psi(self);
2039:     };
2040:     return wrap(dispatch_special_psi(_r.tensor(0)));
```

- EN: The main execution path in this span is carried by `special_polygamma`, `wrap`, `special_polygamma_out`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `special_polygamma`, `wrap`, `special_polygamma_out` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2041-2080

```cpp
2041:   } else {
2042:     // aten::special_psi.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
2043: 
2044:     auto dispatch_special_psi_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
2045:       pybind11::gil_scoped_release no_gil;
2046:       return at::special_psi_out(out, self);
2047:     };
2048:     return wrap(dispatch_special_psi_out(_r.tensor(1), _r.tensor(0)));
2049:   }
2050:   Py_RETURN_NONE;
2051:   END_HANDLE_TH_ERRORS
2052: }
2053: 
2054: // special_round
2055: static PyObject * THPVariable_special_round(PyObject* self_, PyObject* args, PyObject* kwargs)
2056: {
2057:   HANDLE_TH_ERRORS
2058:   static PythonArgParser parser({
2059:     "special_round(Tensor input, *, int64_t decimals=0, Tensor out=None)",
2060:   }, /*traceable=*/true);
2061: 
2062:   ParsedArgs<3> parsed_args;
2063:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2064:   if(_r.has_torch_function()) {
2065:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
2066:   }
2067:   if (_r.isNone(2)) {
2068:     // aten::special_round(Tensor self, *, int decimals=0) -> Tensor
2069: 
2070:     auto dispatch_special_round = [](const at::Tensor & self, int64_t decimals) -> at::Tensor {
2071:       pybind11::gil_scoped_release no_gil;
2072:       return at::special_round(self, decimals);
2073:     };
2074:     return wrap(dispatch_special_round(_r.tensor(0), _r.toInt64(1)));
2075:   } else {
2076:     // aten::special_round.out(Tensor self, *, int decimals=0, Tensor(a!) out) -> Tensor(a!)
2077: 
2078:     auto dispatch_special_round_out = [](at::Tensor out, const at::Tensor & self, int64_t decimals) -> at::Tensor {
2079:       pybind11::gil_scoped_release no_gil;
2080:       return at::special_round_out(out, self, decimals);
```

- EN: The main execution path in this span is carried by `special_psi_out`, `wrap`, `THPVariable_special_round`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `special_psi_out`, `wrap`, `THPVariable_special_round` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2081-2120

```cpp
2081:     };
2082:     return wrap(dispatch_special_round_out(_r.tensor(2), _r.tensor(0), _r.toInt64(1)));
2083:   }
2084:   Py_RETURN_NONE;
2085:   END_HANDLE_TH_ERRORS
2086: }
2087: 
2088: // special_scaled_modified_bessel_k0
2089: static PyObject * THPVariable_special_scaled_modified_bessel_k0(PyObject* self_, PyObject* args, PyObject* kwargs)
2090: {
2091:   HANDLE_TH_ERRORS
2092:   static PythonArgParser parser({
2093:     "special_scaled_modified_bessel_k0(Tensor x, *, Tensor out=None)",
2094:   }, /*traceable=*/true);
2095: 
2096:   ParsedArgs<2> parsed_args;
2097:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2098:   if(_r.has_torch_function()) {
2099:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
2100:   }
2101:   if (_r.isNone(1)) {
2102:     // aten::special_scaled_modified_bessel_k0(Tensor x) -> Tensor
2103: 
2104:     auto dispatch_special_scaled_modified_bessel_k0 = [](const at::Tensor & x) -> at::Tensor {
2105:       pybind11::gil_scoped_release no_gil;
2106:       return at::special_scaled_modified_bessel_k0(x);
2107:     };
2108:     return wrap(dispatch_special_scaled_modified_bessel_k0(_r.tensor(0)));
2109:   } else {
2110:     // aten::special_scaled_modified_bessel_k0.out(Tensor x, *, Tensor(a!) out) -> Tensor(a!)
2111: 
2112:     auto dispatch_special_scaled_modified_bessel_k0_out = [](at::Tensor out, const at::Tensor & x) -> at::Tensor {
2113:       pybind11::gil_scoped_release no_gil;
2114:       return at::special_scaled_modified_bessel_k0_out(out, x);
2115:     };
2116:     return wrap(dispatch_special_scaled_modified_bessel_k0_out(_r.tensor(1), _r.tensor(0)));
2117:   }
2118:   Py_RETURN_NONE;
2119:   END_HANDLE_TH_ERRORS
2120: }
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_special_scaled_modified_bessel_k0`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_special_scaled_modified_bessel_k0`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2121-2160

```cpp
2121: 
2122: // special_scaled_modified_bessel_k1
2123: static PyObject * THPVariable_special_scaled_modified_bessel_k1(PyObject* self_, PyObject* args, PyObject* kwargs)
2124: {
2125:   HANDLE_TH_ERRORS
2126:   static PythonArgParser parser({
2127:     "special_scaled_modified_bessel_k1(Tensor x, *, Tensor out=None)",
2128:   }, /*traceable=*/true);
2129: 
2130:   ParsedArgs<2> parsed_args;
2131:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2132:   if(_r.has_torch_function()) {
2133:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
2134:   }
2135:   if (_r.isNone(1)) {
2136:     // aten::special_scaled_modified_bessel_k1(Tensor x) -> Tensor
2137: 
2138:     auto dispatch_special_scaled_modified_bessel_k1 = [](const at::Tensor & x) -> at::Tensor {
2139:       pybind11::gil_scoped_release no_gil;
2140:       return at::special_scaled_modified_bessel_k1(x);
2141:     };
2142:     return wrap(dispatch_special_scaled_modified_bessel_k1(_r.tensor(0)));
2143:   } else {
2144:     // aten::special_scaled_modified_bessel_k1.out(Tensor x, *, Tensor(a!) out) -> Tensor(a!)
2145: 
2146:     auto dispatch_special_scaled_modified_bessel_k1_out = [](at::Tensor out, const at::Tensor & x) -> at::Tensor {
2147:       pybind11::gil_scoped_release no_gil;
2148:       return at::special_scaled_modified_bessel_k1_out(out, x);
2149:     };
2150:     return wrap(dispatch_special_scaled_modified_bessel_k1_out(_r.tensor(1), _r.tensor(0)));
2151:   }
2152:   Py_RETURN_NONE;
2153:   END_HANDLE_TH_ERRORS
2154: }
2155: 
2156: \
2157: // special_shifted_chebyshev_polynomial_t
2158: static PyObject * THPVariable_special_shifted_chebyshev_polynomial_t(PyObject* self_, PyObject* args, PyObject* kwargs)
2159: {
2160:   HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPVariable_special_scaled_modified_bessel_k1`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_special_scaled_modified_bessel_k1`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2161-2200

```cpp
2161:   static PythonArgParser parser({
2162:     "special_shifted_chebyshev_polynomial_t(Tensor x, Tensor n, *, Tensor out=None)",
2163:     "special_shifted_chebyshev_polynomial_t(Scalar x, Tensor n, *, Tensor out=None)",
2164:     "special_shifted_chebyshev_polynomial_t(Tensor x, Scalar n, *, Tensor out=None)",
2165:   }, /*traceable=*/true);
2166: 
2167:   ParsedArgs<3> parsed_args;
2168:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2169:   if(_r.has_torch_function()) {
2170:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
2171:   }
2172:   switch (_r.idx) {
2173:     case 0: {
2174:       if (_r.isNone(2)) {
2175:         // aten::special_shifted_chebyshev_polynomial_t(Tensor x, Tensor n) -> Tensor
2176: 
2177:         auto dispatch_special_shifted_chebyshev_polynomial_t = [](const at::Tensor & x, const at::Tensor & n) -> at::Tensor {
2178:           pybind11::gil_scoped_release no_gil;
2179:           return at::special_shifted_chebyshev_polynomial_t(x, n);
2180:         };
2181:         return wrap(dispatch_special_shifted_chebyshev_polynomial_t(_r.tensor(0), _r.tensor(1)));
2182:       } else {
2183:         // aten::special_shifted_chebyshev_polynomial_t.out(Tensor x, Tensor n, *, Tensor(a!) out) -> Tensor(a!)
2184: 
2185:         auto dispatch_special_shifted_chebyshev_polynomial_t_out = [](at::Tensor out, const at::Tensor & x, const at::Tensor & n) -> at::Tensor {
2186:           pybind11::gil_scoped_release no_gil;
2187:           return at::special_shifted_chebyshev_polynomial_t_out(out, x, n);
2188:         };
2189:         return wrap(dispatch_special_shifted_chebyshev_polynomial_t_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
2190:       }
2191:     }
2192:     case 1: {
2193:       if (_r.isNone(2)) {
2194:         // aten::special_shifted_chebyshev_polynomial_t.x_scalar(Scalar x, Tensor n) -> Tensor
2195: 
2196:         auto dispatch_special_shifted_chebyshev_polynomial_t = [](const at::Scalar & x, const at::Tensor & n) -> at::Tensor {
2197:           pybind11::gil_scoped_release no_gil;
2198:           return at::special_shifted_chebyshev_polynomial_t(x, n);
2199:         };
2200:         return wrap(dispatch_special_shifted_chebyshev_polynomial_t(_r.scalar(0), _r.tensor(1)));
```

- EN: The main execution path in this span is carried by `parser`, `handle_torch_function`, `special_shifted_chebyshev_polynomial_t`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `parser`, `handle_torch_function`, `special_shifted_chebyshev_polynomial_t` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2201-2240

```cpp
2201:       } else {
2202:         // aten::special_shifted_chebyshev_polynomial_t.x_scalar_out(Scalar x, Tensor n, *, Tensor(a!) out) -> Tensor(a!)
2203: 
2204:         auto dispatch_special_shifted_chebyshev_polynomial_t_out = [](at::Tensor out, const at::Scalar & x, const at::Tensor & n) -> at::Tensor {
2205:           pybind11::gil_scoped_release no_gil;
2206:           return at::special_shifted_chebyshev_polynomial_t_out(out, x, n);
2207:         };
2208:         return wrap(dispatch_special_shifted_chebyshev_polynomial_t_out(_r.tensor(2), _r.scalar(0), _r.tensor(1)));
2209:       }
2210:     }
2211:     case 2: {
2212:       if (_r.isNone(2)) {
2213:         // aten::special_shifted_chebyshev_polynomial_t.n_scalar(Tensor x, Scalar n) -> Tensor
2214: 
2215:         auto dispatch_special_shifted_chebyshev_polynomial_t = [](const at::Tensor & x, const at::Scalar & n) -> at::Tensor {
2216:           pybind11::gil_scoped_release no_gil;
2217:           return at::special_shifted_chebyshev_polynomial_t(x, n);
2218:         };
2219:         return wrap(dispatch_special_shifted_chebyshev_polynomial_t(_r.tensor(0), _r.scalar(1)));
2220:       } else {
2221:         // aten::special_shifted_chebyshev_polynomial_t.n_scalar_out(Tensor x, Scalar n, *, Tensor(a!) out) -> Tensor(a!)
2222: 
2223:         auto dispatch_special_shifted_chebyshev_polynomial_t_out = [](at::Tensor out, const at::Tensor & x, const at::Scalar & n) -> at::Tensor {
2224:           pybind11::gil_scoped_release no_gil;
2225:           return at::special_shifted_chebyshev_polynomial_t_out(out, x, n);
2226:         };
2227:         return wrap(dispatch_special_shifted_chebyshev_polynomial_t_out(_r.tensor(2), _r.tensor(0), _r.scalar(1)));
2228:       }
2229:     }
2230:   }
2231:   Py_RETURN_NONE;
2232:   END_HANDLE_TH_ERRORS
2233: }
2234: 
2235: \
2236: // special_shifted_chebyshev_polynomial_u
2237: static PyObject * THPVariable_special_shifted_chebyshev_polynomial_u(PyObject* self_, PyObject* args, PyObject* kwargs)
2238: {
2239:   HANDLE_TH_ERRORS
2240:   static PythonArgParser parser({
```

- EN: The main execution path in this span is carried by `special_shifted_chebyshev_polynomial_t_out`, `wrap`, `special_shifted_chebyshev_polynomial_t`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `special_shifted_chebyshev_polynomial_t_out`, `wrap`, `special_shifted_chebyshev_polynomial_t` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2241-2280

```cpp
2241:     "special_shifted_chebyshev_polynomial_u(Tensor x, Tensor n, *, Tensor out=None)",
2242:     "special_shifted_chebyshev_polynomial_u(Scalar x, Tensor n, *, Tensor out=None)",
2243:     "special_shifted_chebyshev_polynomial_u(Tensor x, Scalar n, *, Tensor out=None)",
2244:   }, /*traceable=*/true);
2245: 
2246:   ParsedArgs<3> parsed_args;
2247:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2248:   if(_r.has_torch_function()) {
2249:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
2250:   }
2251:   switch (_r.idx) {
2252:     case 0: {
2253:       if (_r.isNone(2)) {
2254:         // aten::special_shifted_chebyshev_polynomial_u(Tensor x, Tensor n) -> Tensor
2255: 
2256:         auto dispatch_special_shifted_chebyshev_polynomial_u = [](const at::Tensor & x, const at::Tensor & n) -> at::Tensor {
2257:           pybind11::gil_scoped_release no_gil;
2258:           return at::special_shifted_chebyshev_polynomial_u(x, n);
2259:         };
2260:         return wrap(dispatch_special_shifted_chebyshev_polynomial_u(_r.tensor(0), _r.tensor(1)));
2261:       } else {
2262:         // aten::special_shifted_chebyshev_polynomial_u.out(Tensor x, Tensor n, *, Tensor(a!) out) -> Tensor(a!)
2263: 
2264:         auto dispatch_special_shifted_chebyshev_polynomial_u_out = [](at::Tensor out, const at::Tensor & x, const at::Tensor & n) -> at::Tensor {
2265:           pybind11::gil_scoped_release no_gil;
2266:           return at::special_shifted_chebyshev_polynomial_u_out(out, x, n);
2267:         };
2268:         return wrap(dispatch_special_shifted_chebyshev_polynomial_u_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
2269:       }
2270:     }
2271:     case 1: {
2272:       if (_r.isNone(2)) {
2273:         // aten::special_shifted_chebyshev_polynomial_u.x_scalar(Scalar x, Tensor n) -> Tensor
2274: 
2275:         auto dispatch_special_shifted_chebyshev_polynomial_u = [](const at::Scalar & x, const at::Tensor & n) -> at::Tensor {
2276:           pybind11::gil_scoped_release no_gil;
2277:           return at::special_shifted_chebyshev_polynomial_u(x, n);
2278:         };
2279:         return wrap(dispatch_special_shifted_chebyshev_polynomial_u(_r.scalar(0), _r.tensor(1)));
2280:       } else {
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `special_shifted_chebyshev_polynomial_u`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `special_shifted_chebyshev_polynomial_u`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2281-2320

```cpp
2281:         // aten::special_shifted_chebyshev_polynomial_u.x_scalar_out(Scalar x, Tensor n, *, Tensor(a!) out) -> Tensor(a!)
2282: 
2283:         auto dispatch_special_shifted_chebyshev_polynomial_u_out = [](at::Tensor out, const at::Scalar & x, const at::Tensor & n) -> at::Tensor {
2284:           pybind11::gil_scoped_release no_gil;
2285:           return at::special_shifted_chebyshev_polynomial_u_out(out, x, n);
2286:         };
2287:         return wrap(dispatch_special_shifted_chebyshev_polynomial_u_out(_r.tensor(2), _r.scalar(0), _r.tensor(1)));
2288:       }
2289:     }
2290:     case 2: {
2291:       if (_r.isNone(2)) {
2292:         // aten::special_shifted_chebyshev_polynomial_u.n_scalar(Tensor x, Scalar n) -> Tensor
2293: 
2294:         auto dispatch_special_shifted_chebyshev_polynomial_u = [](const at::Tensor & x, const at::Scalar & n) -> at::Tensor {
2295:           pybind11::gil_scoped_release no_gil;
2296:           return at::special_shifted_chebyshev_polynomial_u(x, n);
2297:         };
2298:         return wrap(dispatch_special_shifted_chebyshev_polynomial_u(_r.tensor(0), _r.scalar(1)));
2299:       } else {
2300:         // aten::special_shifted_chebyshev_polynomial_u.n_scalar_out(Tensor x, Scalar n, *, Tensor(a!) out) -> Tensor(a!)
2301: 
2302:         auto dispatch_special_shifted_chebyshev_polynomial_u_out = [](at::Tensor out, const at::Tensor & x, const at::Scalar & n) -> at::Tensor {
2303:           pybind11::gil_scoped_release no_gil;
2304:           return at::special_shifted_chebyshev_polynomial_u_out(out, x, n);
2305:         };
2306:         return wrap(dispatch_special_shifted_chebyshev_polynomial_u_out(_r.tensor(2), _r.tensor(0), _r.scalar(1)));
2307:       }
2308:     }
2309:   }
2310:   Py_RETURN_NONE;
2311:   END_HANDLE_TH_ERRORS
2312: }
2313: 
2314: \
2315: // special_shifted_chebyshev_polynomial_v
2316: static PyObject * THPVariable_special_shifted_chebyshev_polynomial_v(PyObject* self_, PyObject* args, PyObject* kwargs)
2317: {
2318:   HANDLE_TH_ERRORS
2319:   static PythonArgParser parser({
2320:     "special_shifted_chebyshev_polynomial_v(Tensor x, Tensor n, *, Tensor out=None)",
```

- EN: The main execution path in this span is carried by `special_shifted_chebyshev_polynomial_u_out`, `wrap`, `special_shifted_chebyshev_polynomial_u`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `special_shifted_chebyshev_polynomial_u_out`, `wrap`, `special_shifted_chebyshev_polynomial_u` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2321-2360

```cpp
2321:     "special_shifted_chebyshev_polynomial_v(Scalar x, Tensor n, *, Tensor out=None)",
2322:     "special_shifted_chebyshev_polynomial_v(Tensor x, Scalar n, *, Tensor out=None)",
2323:   }, /*traceable=*/true);
2324: 
2325:   ParsedArgs<3> parsed_args;
2326:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2327:   if(_r.has_torch_function()) {
2328:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
2329:   }
2330:   switch (_r.idx) {
2331:     case 0: {
2332:       if (_r.isNone(2)) {
2333:         // aten::special_shifted_chebyshev_polynomial_v(Tensor x, Tensor n) -> Tensor
2334: 
2335:         auto dispatch_special_shifted_chebyshev_polynomial_v = [](const at::Tensor & x, const at::Tensor & n) -> at::Tensor {
2336:           pybind11::gil_scoped_release no_gil;
2337:           return at::special_shifted_chebyshev_polynomial_v(x, n);
2338:         };
2339:         return wrap(dispatch_special_shifted_chebyshev_polynomial_v(_r.tensor(0), _r.tensor(1)));
2340:       } else {
2341:         // aten::special_shifted_chebyshev_polynomial_v.out(Tensor x, Tensor n, *, Tensor(a!) out) -> Tensor(a!)
2342: 
2343:         auto dispatch_special_shifted_chebyshev_polynomial_v_out = [](at::Tensor out, const at::Tensor & x, const at::Tensor & n) -> at::Tensor {
2344:           pybind11::gil_scoped_release no_gil;
2345:           return at::special_shifted_chebyshev_polynomial_v_out(out, x, n);
2346:         };
2347:         return wrap(dispatch_special_shifted_chebyshev_polynomial_v_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
2348:       }
2349:     }
2350:     case 1: {
2351:       if (_r.isNone(2)) {
2352:         // aten::special_shifted_chebyshev_polynomial_v.x_scalar(Scalar x, Tensor n) -> Tensor
2353: 
2354:         auto dispatch_special_shifted_chebyshev_polynomial_v = [](const at::Scalar & x, const at::Tensor & n) -> at::Tensor {
2355:           pybind11::gil_scoped_release no_gil;
2356:           return at::special_shifted_chebyshev_polynomial_v(x, n);
2357:         };
2358:         return wrap(dispatch_special_shifted_chebyshev_polynomial_v(_r.scalar(0), _r.tensor(1)));
2359:       } else {
2360:         // aten::special_shifted_chebyshev_polynomial_v.x_scalar_out(Scalar x, Tensor n, *, Tensor(a!) out) -> Tensor(a!)
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `special_shifted_chebyshev_polynomial_v`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `special_shifted_chebyshev_polynomial_v`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2361-2400

```cpp
2361: 
2362:         auto dispatch_special_shifted_chebyshev_polynomial_v_out = [](at::Tensor out, const at::Scalar & x, const at::Tensor & n) -> at::Tensor {
2363:           pybind11::gil_scoped_release no_gil;
2364:           return at::special_shifted_chebyshev_polynomial_v_out(out, x, n);
2365:         };
2366:         return wrap(dispatch_special_shifted_chebyshev_polynomial_v_out(_r.tensor(2), _r.scalar(0), _r.tensor(1)));
2367:       }
2368:     }
2369:     case 2: {
2370:       if (_r.isNone(2)) {
2371:         // aten::special_shifted_chebyshev_polynomial_v.n_scalar(Tensor x, Scalar n) -> Tensor
2372: 
2373:         auto dispatch_special_shifted_chebyshev_polynomial_v = [](const at::Tensor & x, const at::Scalar & n) -> at::Tensor {
2374:           pybind11::gil_scoped_release no_gil;
2375:           return at::special_shifted_chebyshev_polynomial_v(x, n);
2376:         };
2377:         return wrap(dispatch_special_shifted_chebyshev_polynomial_v(_r.tensor(0), _r.scalar(1)));
2378:       } else {
2379:         // aten::special_shifted_chebyshev_polynomial_v.n_scalar_out(Tensor x, Scalar n, *, Tensor(a!) out) -> Tensor(a!)
2380: 
2381:         auto dispatch_special_shifted_chebyshev_polynomial_v_out = [](at::Tensor out, const at::Tensor & x, const at::Scalar & n) -> at::Tensor {
2382:           pybind11::gil_scoped_release no_gil;
2383:           return at::special_shifted_chebyshev_polynomial_v_out(out, x, n);
2384:         };
2385:         return wrap(dispatch_special_shifted_chebyshev_polynomial_v_out(_r.tensor(2), _r.tensor(0), _r.scalar(1)));
2386:       }
2387:     }
2388:   }
2389:   Py_RETURN_NONE;
2390:   END_HANDLE_TH_ERRORS
2391: }
2392: 
2393: \
2394: // special_shifted_chebyshev_polynomial_w
2395: static PyObject * THPVariable_special_shifted_chebyshev_polynomial_w(PyObject* self_, PyObject* args, PyObject* kwargs)
2396: {
2397:   HANDLE_TH_ERRORS
2398:   static PythonArgParser parser({
2399:     "special_shifted_chebyshev_polynomial_w(Tensor x, Tensor n, *, Tensor out=None)",
2400:     "special_shifted_chebyshev_polynomial_w(Scalar x, Tensor n, *, Tensor out=None)",
```

- EN: The main execution path in this span is carried by `special_shifted_chebyshev_polynomial_v_out`, `wrap`, `special_shifted_chebyshev_polynomial_v`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `special_shifted_chebyshev_polynomial_v_out`, `wrap`, `special_shifted_chebyshev_polynomial_v` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2401-2440

```cpp
2401:     "special_shifted_chebyshev_polynomial_w(Tensor x, Scalar n, *, Tensor out=None)",
2402:   }, /*traceable=*/true);
2403: 
2404:   ParsedArgs<3> parsed_args;
2405:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2406:   if(_r.has_torch_function()) {
2407:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
2408:   }
2409:   switch (_r.idx) {
2410:     case 0: {
2411:       if (_r.isNone(2)) {
2412:         // aten::special_shifted_chebyshev_polynomial_w(Tensor x, Tensor n) -> Tensor
2413: 
2414:         auto dispatch_special_shifted_chebyshev_polynomial_w = [](const at::Tensor & x, const at::Tensor & n) -> at::Tensor {
2415:           pybind11::gil_scoped_release no_gil;
2416:           return at::special_shifted_chebyshev_polynomial_w(x, n);
2417:         };
2418:         return wrap(dispatch_special_shifted_chebyshev_polynomial_w(_r.tensor(0), _r.tensor(1)));
2419:       } else {
2420:         // aten::special_shifted_chebyshev_polynomial_w.out(Tensor x, Tensor n, *, Tensor(a!) out) -> Tensor(a!)
2421: 
2422:         auto dispatch_special_shifted_chebyshev_polynomial_w_out = [](at::Tensor out, const at::Tensor & x, const at::Tensor & n) -> at::Tensor {
2423:           pybind11::gil_scoped_release no_gil;
2424:           return at::special_shifted_chebyshev_polynomial_w_out(out, x, n);
2425:         };
2426:         return wrap(dispatch_special_shifted_chebyshev_polynomial_w_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
2427:       }
2428:     }
2429:     case 1: {
2430:       if (_r.isNone(2)) {
2431:         // aten::special_shifted_chebyshev_polynomial_w.x_scalar(Scalar x, Tensor n) -> Tensor
2432: 
2433:         auto dispatch_special_shifted_chebyshev_polynomial_w = [](const at::Scalar & x, const at::Tensor & n) -> at::Tensor {
2434:           pybind11::gil_scoped_release no_gil;
2435:           return at::special_shifted_chebyshev_polynomial_w(x, n);
2436:         };
2437:         return wrap(dispatch_special_shifted_chebyshev_polynomial_w(_r.scalar(0), _r.tensor(1)));
2438:       } else {
2439:         // aten::special_shifted_chebyshev_polynomial_w.x_scalar_out(Scalar x, Tensor n, *, Tensor(a!) out) -> Tensor(a!)
2440: 
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `special_shifted_chebyshev_polynomial_w`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `special_shifted_chebyshev_polynomial_w`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2441-2480

```cpp
2441:         auto dispatch_special_shifted_chebyshev_polynomial_w_out = [](at::Tensor out, const at::Scalar & x, const at::Tensor & n) -> at::Tensor {
2442:           pybind11::gil_scoped_release no_gil;
2443:           return at::special_shifted_chebyshev_polynomial_w_out(out, x, n);
2444:         };
2445:         return wrap(dispatch_special_shifted_chebyshev_polynomial_w_out(_r.tensor(2), _r.scalar(0), _r.tensor(1)));
2446:       }
2447:     }
2448:     case 2: {
2449:       if (_r.isNone(2)) {
2450:         // aten::special_shifted_chebyshev_polynomial_w.n_scalar(Tensor x, Scalar n) -> Tensor
2451: 
2452:         auto dispatch_special_shifted_chebyshev_polynomial_w = [](const at::Tensor & x, const at::Scalar & n) -> at::Tensor {
2453:           pybind11::gil_scoped_release no_gil;
2454:           return at::special_shifted_chebyshev_polynomial_w(x, n);
2455:         };
2456:         return wrap(dispatch_special_shifted_chebyshev_polynomial_w(_r.tensor(0), _r.scalar(1)));
2457:       } else {
2458:         // aten::special_shifted_chebyshev_polynomial_w.n_scalar_out(Tensor x, Scalar n, *, Tensor(a!) out) -> Tensor(a!)
2459: 
2460:         auto dispatch_special_shifted_chebyshev_polynomial_w_out = [](at::Tensor out, const at::Tensor & x, const at::Scalar & n) -> at::Tensor {
2461:           pybind11::gil_scoped_release no_gil;
2462:           return at::special_shifted_chebyshev_polynomial_w_out(out, x, n);
2463:         };
2464:         return wrap(dispatch_special_shifted_chebyshev_polynomial_w_out(_r.tensor(2), _r.tensor(0), _r.scalar(1)));
2465:       }
2466:     }
2467:   }
2468:   Py_RETURN_NONE;
2469:   END_HANDLE_TH_ERRORS
2470: }
2471: 
2472: // special_sinc
2473: static PyObject * THPVariable_special_sinc(PyObject* self_, PyObject* args, PyObject* kwargs)
2474: {
2475:   HANDLE_TH_ERRORS
2476:   static PythonArgParser parser({
2477:     "special_sinc(Tensor input, *, Tensor out=None)",
2478:   }, /*traceable=*/true);
2479: 
2480:   ParsedArgs<2> parsed_args;
```

- EN: The main execution path in this span is carried by `special_shifted_chebyshev_polynomial_w_out`, `wrap`, `special_shifted_chebyshev_polynomial_w`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `special_shifted_chebyshev_polynomial_w_out`, `wrap`, `special_shifted_chebyshev_polynomial_w` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2481-2520

```cpp
2481:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2482:   if(_r.has_torch_function()) {
2483:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
2484:   }
2485:   if (_r.isNone(1)) {
2486:     // aten::special_sinc(Tensor self) -> Tensor
2487: 
2488:     auto dispatch_special_sinc = [](const at::Tensor & self) -> at::Tensor {
2489:       pybind11::gil_scoped_release no_gil;
2490:       return at::special_sinc(self);
2491:     };
2492:     return wrap(dispatch_special_sinc(_r.tensor(0)));
2493:   } else {
2494:     // aten::special_sinc.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
2495: 
2496:     auto dispatch_special_sinc_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
2497:       pybind11::gil_scoped_release no_gil;
2498:       return at::special_sinc_out(out, self);
2499:     };
2500:     return wrap(dispatch_special_sinc_out(_r.tensor(1), _r.tensor(0)));
2501:   }
2502:   Py_RETURN_NONE;
2503:   END_HANDLE_TH_ERRORS
2504: }
2505: 
2506: // special_softmax
2507: static PyObject * THPVariable_special_softmax(PyObject* self_, PyObject* args, PyObject* kwargs)
2508: {
2509:   HANDLE_TH_ERRORS
2510:   static PythonArgParser parser({
2511:     "special_softmax(Tensor input, int64_t dim, ScalarType? dtype=None)",
2512:   }, /*traceable=*/true);
2513: 
2514:   ParsedArgs<3> parsed_args;
2515:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2516:   if(_r.has_torch_function()) {
2517:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
2518:   }
2519:   // aten::special_softmax(Tensor self, int dim, ScalarType? dtype=None) -> Tensor
2520: 
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `special_sinc`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `special_sinc`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2521-2560

```cpp
2521:   auto dispatch_special_softmax = [](const at::Tensor & self, int64_t dim, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
2522:     pybind11::gil_scoped_release no_gil;
2523:     return at::special_softmax(self, dim, dtype);
2524:   };
2525:   return wrap(dispatch_special_softmax(_r.tensor(0), _r.toInt64(1), _r.scalartypeOptional(2)));
2526:   Py_RETURN_NONE;
2527:   END_HANDLE_TH_ERRORS
2528: }
2529: 
2530: // special_spherical_bessel_j0
2531: static PyObject * THPVariable_special_spherical_bessel_j0(PyObject* self_, PyObject* args, PyObject* kwargs)
2532: {
2533:   HANDLE_TH_ERRORS
2534:   static PythonArgParser parser({
2535:     "special_spherical_bessel_j0(Tensor x, *, Tensor out=None)",
2536:   }, /*traceable=*/true);
2537: 
2538:   ParsedArgs<2> parsed_args;
2539:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2540:   if(_r.has_torch_function()) {
2541:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
2542:   }
2543:   if (_r.isNone(1)) {
2544:     // aten::special_spherical_bessel_j0(Tensor x) -> Tensor
2545: 
2546:     auto dispatch_special_spherical_bessel_j0 = [](const at::Tensor & x) -> at::Tensor {
2547:       pybind11::gil_scoped_release no_gil;
2548:       return at::special_spherical_bessel_j0(x);
2549:     };
2550:     return wrap(dispatch_special_spherical_bessel_j0(_r.tensor(0)));
2551:   } else {
2552:     // aten::special_spherical_bessel_j0.out(Tensor x, *, Tensor(a!) out) -> Tensor(a!)
2553: 
2554:     auto dispatch_special_spherical_bessel_j0_out = [](at::Tensor out, const at::Tensor & x) -> at::Tensor {
2555:       pybind11::gil_scoped_release no_gil;
2556:       return at::special_spherical_bessel_j0_out(out, x);
2557:     };
2558:     return wrap(dispatch_special_spherical_bessel_j0_out(_r.tensor(1), _r.tensor(0)));
2559:   }
2560:   Py_RETURN_NONE;
```

- EN: The main execution path in this span is carried by `special_softmax`, `wrap`, `THPVariable_special_spherical_bessel_j0`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `special_softmax`, `wrap`, `THPVariable_special_spherical_bessel_j0` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2561-2600

```cpp
2561:   END_HANDLE_TH_ERRORS
2562: }
2563: 
2564: \
2565: // special_xlog1py
2566: static PyObject * THPVariable_special_xlog1py(PyObject* self_, PyObject* args, PyObject* kwargs)
2567: {
2568:   HANDLE_TH_ERRORS
2569:   static PythonArgParser parser({
2570:     "special_xlog1py(Tensor input, Tensor other, *, Tensor out=None)",
2571:     "special_xlog1py(Scalar self, Tensor other, *, Tensor out=None)",
2572:     "special_xlog1py(Tensor input, Scalar other, *, Tensor out=None)",
2573:   }, /*traceable=*/true);
2574: 
2575:   ParsedArgs<3> parsed_args;
2576:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2577:   if(_r.has_torch_function()) {
2578:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
2579:   }
2580:   switch (_r.idx) {
2581:     case 0: {
2582:       if (_r.isNone(2)) {
2583:         // aten::special_xlog1py(Tensor self, Tensor other) -> Tensor
2584: 
2585:         auto dispatch_special_xlog1py = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
2586:           pybind11::gil_scoped_release no_gil;
2587:           return at::special_xlog1py(self, other);
2588:         };
2589:         return wrap(dispatch_special_xlog1py(_r.tensor(0), _r.tensor(1)));
2590:       } else {
2591:         // aten::special_xlog1py.out(Tensor self, Tensor other, *, Tensor(a!) out) -> Tensor(a!)
2592: 
2593:         auto dispatch_special_xlog1py_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
2594:           pybind11::gil_scoped_release no_gil;
2595:           return at::special_xlog1py_out(out, self, other);
2596:         };
2597:         return wrap(dispatch_special_xlog1py_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
2598:       }
2599:     }
2600:     case 1: {
```

- EN: The main execution path in this span is carried by `THPVariable_special_xlog1py`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_special_xlog1py`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2601-2640

```cpp
2601:       if (_r.isNone(2)) {
2602:         // aten::special_xlog1py.self_scalar(Scalar self, Tensor other) -> Tensor
2603: 
2604:         auto dispatch_special_xlog1py = [](const at::Scalar & self, const at::Tensor & other) -> at::Tensor {
2605:           pybind11::gil_scoped_release no_gil;
2606:           return at::special_xlog1py(self, other);
2607:         };
2608:         return wrap(dispatch_special_xlog1py(_r.scalar(0), _r.tensor(1)));
2609:       } else {
2610:         // aten::special_xlog1py.self_scalar_out(Scalar self, Tensor other, *, Tensor(a!) out) -> Tensor(a!)
2611: 
2612:         auto dispatch_special_xlog1py_out = [](at::Tensor out, const at::Scalar & self, const at::Tensor & other) -> at::Tensor {
2613:           pybind11::gil_scoped_release no_gil;
2614:           return at::special_xlog1py_out(out, self, other);
2615:         };
2616:         return wrap(dispatch_special_xlog1py_out(_r.tensor(2), _r.scalar(0), _r.tensor(1)));
2617:       }
2618:     }
2619:     case 2: {
2620:       if (_r.isNone(2)) {
2621:         // aten::special_xlog1py.other_scalar(Tensor self, Scalar other) -> Tensor
2622: 
2623:         auto dispatch_special_xlog1py = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
2624:           pybind11::gil_scoped_release no_gil;
2625:           return at::special_xlog1py(self, other);
2626:         };
2627:         return wrap(dispatch_special_xlog1py(_r.tensor(0), _r.scalar(1)));
2628:       } else {
2629:         // aten::special_xlog1py.other_scalar_out(Tensor self, Scalar other, *, Tensor(a!) out) -> Tensor(a!)
2630: 
2631:         auto dispatch_special_xlog1py_out = [](at::Tensor out, const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
2632:           pybind11::gil_scoped_release no_gil;
2633:           return at::special_xlog1py_out(out, self, other);
2634:         };
2635:         return wrap(dispatch_special_xlog1py_out(_r.tensor(2), _r.tensor(0), _r.scalar(1)));
2636:       }
2637:     }
2638:   }
2639:   Py_RETURN_NONE;
2640:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `special_xlog1py`, `wrap`, `special_xlog1py_out`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `special_xlog1py`, `wrap`, `special_xlog1py_out` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2641-2680

```cpp
2641: }
2642: 
2643: \
2644: // special_xlogy
2645: static PyObject * THPVariable_special_xlogy(PyObject* self_, PyObject* args, PyObject* kwargs)
2646: {
2647:   HANDLE_TH_ERRORS
2648:   static PythonArgParser parser({
2649:     "special_xlogy(Tensor input, Tensor other, *, Tensor out=None)",
2650:     "special_xlogy(Scalar self, Tensor other, *, Tensor out=None)",
2651:     "special_xlogy(Tensor input, Scalar other, *, Tensor out=None)",
2652:   }, /*traceable=*/true);
2653: 
2654:   ParsedArgs<3> parsed_args;
2655:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2656:   if(_r.has_torch_function()) {
2657:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
2658:   }
2659:   switch (_r.idx) {
2660:     case 0: {
2661:       if (_r.isNone(2)) {
2662:         // aten::special_xlogy(Tensor self, Tensor other) -> Tensor
2663: 
2664:         auto dispatch_special_xlogy = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
2665:           pybind11::gil_scoped_release no_gil;
2666:           return at::special_xlogy(self, other);
2667:         };
2668:         return wrap(dispatch_special_xlogy(_r.tensor(0), _r.tensor(1)));
2669:       } else {
2670:         // aten::special_xlogy.out(Tensor self, Tensor other, *, Tensor(a!) out) -> Tensor(a!)
2671: 
2672:         auto dispatch_special_xlogy_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
2673:           pybind11::gil_scoped_release no_gil;
2674:           return at::special_xlogy_out(out, self, other);
2675:         };
2676:         return wrap(dispatch_special_xlogy_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
2677:       }
2678:     }
2679:     case 1: {
2680:       if (_r.isNone(2)) {
```

- EN: The main execution path in this span is carried by `THPVariable_special_xlogy`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_special_xlogy`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2681-2720

```cpp
2681:         // aten::special_xlogy.self_scalar(Scalar self, Tensor other) -> Tensor
2682: 
2683:         auto dispatch_special_xlogy = [](const at::Scalar & self, const at::Tensor & other) -> at::Tensor {
2684:           pybind11::gil_scoped_release no_gil;
2685:           return at::special_xlogy(self, other);
2686:         };
2687:         return wrap(dispatch_special_xlogy(_r.scalar(0), _r.tensor(1)));
2688:       } else {
2689:         // aten::special_xlogy.self_scalar_out(Scalar self, Tensor other, *, Tensor(a!) out) -> Tensor(a!)
2690: 
2691:         auto dispatch_special_xlogy_out = [](at::Tensor out, const at::Scalar & self, const at::Tensor & other) -> at::Tensor {
2692:           pybind11::gil_scoped_release no_gil;
2693:           return at::special_xlogy_out(out, self, other);
2694:         };
2695:         return wrap(dispatch_special_xlogy_out(_r.tensor(2), _r.scalar(0), _r.tensor(1)));
2696:       }
2697:     }
2698:     case 2: {
2699:       if (_r.isNone(2)) {
2700:         // aten::special_xlogy.other_scalar(Tensor self, Scalar other) -> Tensor
2701: 
2702:         auto dispatch_special_xlogy = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
2703:           pybind11::gil_scoped_release no_gil;
2704:           return at::special_xlogy(self, other);
2705:         };
2706:         return wrap(dispatch_special_xlogy(_r.tensor(0), _r.scalar(1)));
2707:       } else {
2708:         // aten::special_xlogy.other_scalar_out(Tensor self, Scalar other, *, Tensor(a!) out) -> Tensor(a!)
2709: 
2710:         auto dispatch_special_xlogy_out = [](at::Tensor out, const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
2711:           pybind11::gil_scoped_release no_gil;
2712:           return at::special_xlogy_out(out, self, other);
2713:         };
2714:         return wrap(dispatch_special_xlogy_out(_r.tensor(2), _r.tensor(0), _r.scalar(1)));
2715:       }
2716:     }
2717:   }
2718:   Py_RETURN_NONE;
2719:   END_HANDLE_TH_ERRORS
2720: }
```

- EN: The main execution path in this span is carried by `special_xlogy`, `wrap`, `special_xlogy_out`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `special_xlogy`, `wrap`, `special_xlogy_out` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2721-2760

```cpp
2721: 
2722: \
2723: // special_zeta
2724: static PyObject * THPVariable_special_zeta(PyObject* self_, PyObject* args, PyObject* kwargs)
2725: {
2726:   HANDLE_TH_ERRORS
2727:   static PythonArgParser parser({
2728:     "special_zeta(Tensor input, Tensor other, *, Tensor out=None)",
2729:     "special_zeta(Scalar self, Tensor other, *, Tensor out=None)",
2730:     "special_zeta(Tensor input, Scalar other, *, Tensor out=None)",
2731:   }, /*traceable=*/true);
2732: 
2733:   ParsedArgs<3> parsed_args;
2734:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2735:   if(_r.has_torch_function()) {
2736:     return handle_torch_function(_r, nullptr, args, kwargs, THPSpecialVariableFunctionsModule, "torch.special");
2737:   }
2738:   switch (_r.idx) {
2739:     case 0: {
2740:       if (_r.isNone(2)) {
2741:         // aten::special_zeta(Tensor self, Tensor other) -> Tensor
2742: 
2743:         auto dispatch_special_zeta = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
2744:           pybind11::gil_scoped_release no_gil;
2745:           return at::special_zeta(self, other);
2746:         };
2747:         return wrap(dispatch_special_zeta(_r.tensor(0), _r.tensor(1)));
2748:       } else {
2749:         // aten::special_zeta.out(Tensor self, Tensor other, *, Tensor(a!) out) -> Tensor(a!)
2750: 
2751:         auto dispatch_special_zeta_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
2752:           pybind11::gil_scoped_release no_gil;
2753:           return at::special_zeta_out(out, self, other);
2754:         };
2755:         return wrap(dispatch_special_zeta_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
2756:       }
2757:     }
2758:     case 1: {
2759:       if (_r.isNone(2)) {
2760:         // aten::special_zeta.self_scalar(Scalar self, Tensor other) -> Tensor
```

- EN: The main execution path in this span is carried by `THPVariable_special_zeta`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_special_zeta`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2761-2800

```cpp
2761: 
2762:         auto dispatch_special_zeta = [](const at::Scalar & self, const at::Tensor & other) -> at::Tensor {
2763:           pybind11::gil_scoped_release no_gil;
2764:           return at::special_zeta(self, other);
2765:         };
2766:         return wrap(dispatch_special_zeta(_r.scalar(0), _r.tensor(1)));
2767:       } else {
2768:         // aten::special_zeta.self_scalar_out(Scalar self, Tensor other, *, Tensor(a!) out) -> Tensor(a!)
2769: 
2770:         auto dispatch_special_zeta_out = [](at::Tensor out, const at::Scalar & self, const at::Tensor & other) -> at::Tensor {
2771:           pybind11::gil_scoped_release no_gil;
2772:           return at::special_zeta_out(out, self, other);
2773:         };
2774:         return wrap(dispatch_special_zeta_out(_r.tensor(2), _r.scalar(0), _r.tensor(1)));
2775:       }
2776:     }
2777:     case 2: {
2778:       if (_r.isNone(2)) {
2779:         // aten::special_zeta.other_scalar(Tensor self, Scalar other) -> Tensor
2780: 
2781:         auto dispatch_special_zeta = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
2782:           pybind11::gil_scoped_release no_gil;
2783:           return at::special_zeta(self, other);
2784:         };
2785:         return wrap(dispatch_special_zeta(_r.tensor(0), _r.scalar(1)));
2786:       } else {
2787:         // aten::special_zeta.other_scalar_out(Tensor self, Scalar other, *, Tensor(a!) out) -> Tensor(a!)
2788: 
2789:         auto dispatch_special_zeta_out = [](at::Tensor out, const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
2790:           pybind11::gil_scoped_release no_gil;
2791:           return at::special_zeta_out(out, self, other);
2792:         };
2793:         return wrap(dispatch_special_zeta_out(_r.tensor(2), _r.tensor(0), _r.scalar(1)));
2794:       }
2795:     }
2796:   }
2797:   Py_RETURN_NONE;
2798:   END_HANDLE_TH_ERRORS
2799: }
2800: 
```

- EN: The main execution path in this span is carried by `special_zeta`, `wrap`, `special_zeta_out`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `special_zeta`, `wrap`, `special_zeta_out` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2801-2801

```cpp
2801: } // namespace torch::autograd
```

- EN: They also open, refine, or close the relevant C++ namespace scope.
- CN: 它们还会打开、细化或关闭相应的 C++ 命名空间作用域。

## Key Concepts / 关键概念
- Autograd graph and gradient metadata / 自动求导图与梯度元数据
- Generated code patterns / 生成代码模式
- Python/C++ binding boundary / Python/C++ 绑定边界
- Tensor/value representation management / 张量/数值表示管理
- Primary symbol `THPVariable_special_airy_ai` / 核心符号 `THPVariable_special_airy_ai`
- Primary symbol `THPVariable_special_bessel_j0` / 核心符号 `THPVariable_special_bessel_j0`

## Dependencies / 依赖关系
- Direct includes / 直接包含: `torch/csrc/Device.h`, `torch/csrc/DynamicTypes.h`, `torch/csrc/Exceptions.h`, `torch/csrc/autograd/python_special_functions.h`, `torch/csrc/autograd/generated/python_return_types.h`, `torch/csrc/autograd/python_variable.h`, `torch/csrc/autograd/utils/wrap_outputs.h`, `torch/csrc/autograd/utils/python_arg_parsing.h`, `torch/csrc/autograd/generated/variable_factories.h`, `torch/csrc/utils/out_types.h`
- Include roots / 头文件根模块: `ATen`, `torch`
- Key symbols / 关键符号: `THPVariable_special_airy_ai`, `THPVariable_special_bessel_j0`, `THPVariable_special_bessel_j1`, `THPVariable_special_bessel_y0`, `THPVariable_special_bessel_y1`, `THPVariable_special_chebyshev_polynomial_t`, `THPVariable_special_chebyshev_polynomial_u`, `THPVariable_special_chebyshev_polynomial_v`, `THPVariable_special_chebyshev_polynomial_w`, `THPVariable_special_digamma`
- Related subsystems / 相关子系统: Autograd / 自动求导, ATen operator layer / ATen 算子层, Python binding layer / Python 绑定层
