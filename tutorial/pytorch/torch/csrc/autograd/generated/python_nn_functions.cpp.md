# python_nn_functions.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/autograd/generated/python_nn_functions.cpp`
- Repository: `/root/xw/pytorch`
- Purpose (EN): Defines generated backward-function classes and helpers used by the autograd engine.
- 目的 (CN): 定义自动求导引擎使用的生成式反向函数类与辅助逻辑。
- Lines: 3493
- Language: C++ / C++

## Line-by-Line Analysis / 逐行分析

- Note / 说明: This file is generated or mechanically expanded, so the analysis groups contiguous line ranges rather than paraphrasing every repeated wrapper individually. / 该文件为生成代码或机械展开代码，因此分析按连续行范围组织，而不是逐个重复改写每个封装模板。
### Lines 1-40

```cpp
 1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
 2: // @generated from ../tools/autograd/templates/python_nn_functions.cpp
 3: 
 4: #include "torch/csrc/Device.h"
 5: #include "torch/csrc/DynamicTypes.h"
 6: #include "torch/csrc/Exceptions.h"
 7: #include "torch/csrc/autograd/python_nn_functions.h"
 8: #include "torch/csrc/autograd/generated/python_return_types.h"
 9: #include "torch/csrc/autograd/python_variable.h"
10: #include "torch/csrc/autograd/utils/wrap_outputs.h"
11: #include "torch/csrc/autograd/utils/python_arg_parsing.h"
12: #include "torch/csrc/utils/pycfunction_helpers.h"
13: #include "torch/csrc/utils/python_arg_parser.h"
14: #include "torch/csrc/utils/structseq.h"
15: #include "torch/csrc/utils/tensor_memoryformats.h"
16: 
17: #ifndef AT_PER_OPERATOR_HEADERS
18: #include <ATen/Functions.h>
19: #else
20: #include <ATen/ops/_conv_depthwise2d.h>
21: #include <ATen/ops/_pad_circular.h>
22: #include <ATen/ops/_pad_enum.h>
23: #include <ATen/ops/_test_ambiguous_defaults.h>
24: #include <ATen/ops/_test_optional_filled_intlist.h>
25: #include <ATen/ops/_test_optional_floatlist.h>
26: #include <ATen/ops/_test_optional_intlist.h>
27: #include <ATen/ops/_test_string_default.h>
28: #include <ATen/ops/_test_warn_in_autograd.h>
29: #include <ATen/ops/_upsample_bicubic2d_aa.h>
30: #include <ATen/ops/_upsample_bilinear2d_aa.h>
31: #include <ATen/ops/_upsample_lanczos2d_aa.h>
32: #include <ATen/ops/_upsample_nearest_exact1d.h>
33: #include <ATen/ops/_upsample_nearest_exact2d.h>
34: #include <ATen/ops/_upsample_nearest_exact3d.h>
35: #include <ATen/ops/adaptive_avg_pool2d.h>
36: #include <ATen/ops/adaptive_avg_pool3d.h>
37: #include <ATen/ops/adaptive_max_pool2d.h>
38: #include <ATen/ops/adaptive_max_pool3d.h>
39: #include <ATen/ops/avg_pool2d.h>
40: #include <ATen/ops/avg_pool3d.h>
```

- EN: These lines pull in dependencies such as `torch/csrc/Device.h`, `torch/csrc/DynamicTypes.h`, `torch/csrc/Exceptions.h`, establishing the headers needed by the implementation. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这些行引入了依赖，例如 `torch/csrc/Device.h`, `torch/csrc/DynamicTypes.h`, `torch/csrc/Exceptions.h`，为后续实现建立所需的头文件基础。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 41-80

```cpp
41: #include <ATen/ops/binary_cross_entropy.h>
42: #include <ATen/ops/col2im.h>
43: #include <ATen/ops/conv_depthwise3d.h>
44: #include <ATen/ops/cross_entropy_loss.h>
45: #include <ATen/ops/elu.h>
46: #include <ATen/ops/elu.h>
47: #include <ATen/ops/flatten_dense_tensors.h>
48: #include <ATen/ops/fractional_max_pool2d.h>
49: #include <ATen/ops/fractional_max_pool3d.h>
50: #include <ATen/ops/gelu.h>
51: #include <ATen/ops/gelu.h>
52: #include <ATen/ops/glu.h>
53: #include <ATen/ops/hardsigmoid.h>
54: #include <ATen/ops/hardsigmoid.h>
55: #include <ATen/ops/hardswish.h>
56: #include <ATen/ops/hardswish.h>
57: #include <ATen/ops/hardtanh.h>
58: #include <ATen/ops/hardtanh.h>
59: #include <ATen/ops/huber_loss.h>
60: #include <ATen/ops/im2col.h>
61: #include <ATen/ops/l1_loss.h>
62: #include <ATen/ops/leaky_relu.h>
63: #include <ATen/ops/leaky_relu.h>
64: #include <ATen/ops/linear.h>
65: #include <ATen/ops/log_sigmoid.h>
66: #include <ATen/ops/max_pool2d_with_indices.h>
67: #include <ATen/ops/max_pool3d_with_indices.h>
68: #include <ATen/ops/max_unpool2d.h>
69: #include <ATen/ops/max_unpool3d.h>
70: #include <ATen/ops/mish.h>
71: #include <ATen/ops/mish.h>
72: #include <ATen/ops/mkldnn_linear.h>
73: #include <ATen/ops/mkldnn_reorder_conv2d_weight.h>
74: #include <ATen/ops/mkldnn_reorder_conv3d_weight.h>
75: #include <ATen/ops/mse_loss.h>
76: #include <ATen/ops/multi_margin_loss.h>
77: #include <ATen/ops/multilabel_margin_loss.h>
78: #include <ATen/ops/nll_loss.h>
79: #include <ATen/ops/nll_loss2d.h>
80: #include <ATen/ops/nll_loss_nd.h>
```

- EN: These lines pull in dependencies such as `ATen/ops/binary_cross_entropy.h`, `ATen/ops/col2im.h`, `ATen/ops/conv_depthwise3d.h`, establishing the headers needed by the implementation.
- CN: 这些行引入了依赖，例如 `ATen/ops/binary_cross_entropy.h`, `ATen/ops/col2im.h`, `ATen/ops/conv_depthwise3d.h`，为后续实现建立所需的头文件基础。
### Lines 81-120

```cpp
 81: #include <ATen/ops/one_hot.h>
 82: #include <ATen/ops/pad.h>
 83: #include <ATen/ops/pad_sequence.h>
 84: #include <ATen/ops/reflection_pad1d.h>
 85: #include <ATen/ops/reflection_pad2d.h>
 86: #include <ATen/ops/reflection_pad3d.h>
 87: #include <ATen/ops/relu6.h>
 88: #include <ATen/ops/relu6.h>
 89: #include <ATen/ops/replication_pad1d.h>
 90: #include <ATen/ops/replication_pad2d.h>
 91: #include <ATen/ops/replication_pad3d.h>
 92: #include <ATen/ops/rrelu_with_noise.h>
 93: #include <ATen/ops/rrelu_with_noise.h>
 94: #include <ATen/ops/scaled_dot_product_attention.h>
 95: #include <ATen/ops/silu.h>
 96: #include <ATen/ops/silu.h>
 97: #include <ATen/ops/slow_conv3d.h>
 98: #include <ATen/ops/slow_conv_dilated2d.h>
 99: #include <ATen/ops/slow_conv_dilated3d.h>
100: #include <ATen/ops/slow_conv_transpose2d.h>
101: #include <ATen/ops/slow_conv_transpose3d.h>
102: #include <ATen/ops/smooth_l1_loss.h>
103: #include <ATen/ops/soft_margin_loss.h>
104: #include <ATen/ops/softplus.h>
105: #include <ATen/ops/softshrink.h>
106: #include <ATen/ops/thnn_conv2d.h>
107: #include <ATen/ops/unflatten_dense_tensors.h>
108: #include <ATen/ops/upsample_bicubic2d.h>
109: #include <ATen/ops/upsample_bilinear2d.h>
110: #include <ATen/ops/upsample_linear1d.h>
111: #include <ATen/ops/upsample_nearest1d.h>
112: #include <ATen/ops/upsample_nearest2d.h>
113: #include <ATen/ops/upsample_nearest3d.h>
114: #include <ATen/ops/upsample_trilinear3d.h>
115: #endif
116: 
117: using at::Tensor;
118: using at::Scalar;
119: using at::MemoryFormat;
120: using at::Generator;
```

- EN: These lines pull in dependencies such as `ATen/ops/one_hot.h`, `ATen/ops/pad.h`, `ATen/ops/pad_sequence.h`, establishing the headers needed by the implementation.
- CN: 这些行引入了依赖，例如 `ATen/ops/one_hot.h`, `ATen/ops/pad.h`, `ATen/ops/pad_sequence.h`，为后续实现建立所需的头文件基础。
### Lines 121-160

```cpp
121: using at::IntArrayRef;
122: using at::ArrayRef;
123: 
124: using namespace torch::autograd::utils;
125: 
126: namespace torch::autograd {
127: 
128: static PyObject* THPNNVariableFunctionsModule = nullptr;
129: 
130: static PyObject * THPVariable__parse_to(PyObject* module, PyObject* args, PyObject* kwargs)
131: {
132:   HANDLE_TH_ERRORS
133:   static PythonArgParser parser({
134:     "to(Device device=None, ScalarType dtype=None, bool non_blocking=False, bool copy=False, *, MemoryFormat? memory_format=None)",
135:     "to(ScalarType dtype, bool non_blocking=False, bool copy=False, *, MemoryFormat? memory_format=None)",
136:     "to(Tensor tensor, bool non_blocking=False, bool copy=False, *, MemoryFormat? memory_format=None)",
137:   });
138:   ParsedArgs<5> parsed_args;
139:   auto r = parser.parse(args, kwargs, parsed_args);
140:   if (r.has_torch_function()) {
141:     return handle_torch_function(r, args, kwargs, THPNNVariableFunctionsModule, "torch.nn", "_parse_to");
142:   }
143:   auto parsed = parse_to_conversion(r, /*allow_copy*/ false); // we don't want copy for nn.Module.to
144:   auto& device = std::get<0>(parsed);
145:   auto& scalarType = std::get<1>(parsed);
146:   auto non_blocking = std::get<2>(parsed);
147:   auto opt_memory_format = std::get<4>(parsed);
148:   auto tuple = THPObjectPtr{PyTuple_New(4)};
149:   if (!tuple) throw python_error();
150:   if (device) {
151:     PyTuple_SET_ITEM(tuple.get(), 0, THPDevice_New(*device));
152:   } else {
153:     Py_INCREF(Py_None);
154:     PyTuple_SET_ITEM(tuple.get(), 0, Py_None);
155:   }
156:   if (scalarType) {
157:     PyTuple_SET_ITEM(tuple.get(), 1, Py_NewRef(torch::getTHPDtype(*scalarType)));
158:   } else {
159:     Py_INCREF(Py_None);
160:     PyTuple_SET_ITEM(tuple.get(), 1, Py_None);
```

- EN: They also open, refine, or close the relevant C++ namespace scope. The main execution path in this span is carried by `THPVariable__parse_to`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 它们还会打开、细化或关闭相应的 C++ 命名空间作用域。 这一段的主要执行路径由 `THPVariable__parse_to`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 161-200

```cpp
161:   }
162:   PyTuple_SET_ITEM(tuple.get(), 2, torch::autograd::utils::wrap(non_blocking));
163:   if (opt_memory_format.has_value()) {
164:     PyTuple_SET_ITEM(tuple.get(), 3, Py_NewRef(torch::utils::getTHPMemoryFormat(opt_memory_format.value())));
165:   } else {
166:     Py_INCREF(Py_None);
167:     PyTuple_SET_ITEM(tuple.get(), 3, Py_None);
168:   }
169:   return tuple.release();
170:   END_HANDLE_TH_ERRORS
171: }
172: 
173: // generated forward declarations start here
174: 
175: static PyObject * THPVariable__conv_depthwise2d(PyObject* self_, PyObject* args, PyObject* kwargs);
176: static PyObject * THPVariable__pad_circular(PyObject* self_, PyObject* args, PyObject* kwargs);
177: static PyObject * THPVariable__pad_enum(PyObject* self_, PyObject* args, PyObject* kwargs);
178: static PyObject * THPVariable__test_ambiguous_defaults(PyObject* self_, PyObject* args, PyObject* kwargs);
179: static PyObject * THPVariable__test_optional_filled_intlist(PyObject* self_, PyObject* args, PyObject* kwargs);
180: static PyObject * THPVariable__test_optional_floatlist(PyObject* self_, PyObject* args, PyObject* kwargs);
181: static PyObject * THPVariable__test_optional_intlist(PyObject* self_, PyObject* args, PyObject* kwargs);
182: static PyObject * THPVariable__test_string_default(PyObject* self_, PyObject* args, PyObject* kwargs);
183: static PyObject * THPVariable__test_warn_in_autograd(PyObject* self_, PyObject* args, PyObject* kwargs);
184: static PyObject * THPVariable__upsample_bicubic2d_aa(PyObject* self_, PyObject* args, PyObject* kwargs);
185: static PyObject * THPVariable__upsample_bilinear2d_aa(PyObject* self_, PyObject* args, PyObject* kwargs);
186: static PyObject * THPVariable__upsample_lanczos2d_aa(PyObject* self_, PyObject* args, PyObject* kwargs);
187: static PyObject * THPVariable__upsample_nearest_exact1d(PyObject* self_, PyObject* args, PyObject* kwargs);
188: static PyObject * THPVariable__upsample_nearest_exact2d(PyObject* self_, PyObject* args, PyObject* kwargs);
189: static PyObject * THPVariable__upsample_nearest_exact3d(PyObject* self_, PyObject* args, PyObject* kwargs);
190: static PyObject * THPVariable_adaptive_avg_pool2d(PyObject* self_, PyObject* args, PyObject* kwargs);
191: static PyObject * THPVariable_adaptive_avg_pool3d(PyObject* self_, PyObject* args, PyObject* kwargs);
192: static PyObject * THPVariable_adaptive_max_pool2d(PyObject* self_, PyObject* args, PyObject* kwargs);
193: static PyObject * THPVariable_adaptive_max_pool3d(PyObject* self_, PyObject* args, PyObject* kwargs);
194: static PyObject * THPVariable_avg_pool2d(PyObject* self_, PyObject* args, PyObject* kwargs);
195: static PyObject * THPVariable_avg_pool3d(PyObject* self_, PyObject* args, PyObject* kwargs);
196: static PyObject * THPVariable_binary_cross_entropy(PyObject* self_, PyObject* args, PyObject* kwargs);
197: static PyObject * THPVariable_col2im(PyObject* self_, PyObject* args, PyObject* kwargs);
198: static PyObject * THPVariable_conv_depthwise3d(PyObject* self_, PyObject* args, PyObject* kwargs);
199: static PyObject * THPVariable_cross_entropy_loss(PyObject* self_, PyObject* args, PyObject* kwargs);
200: static PyObject * THPVariable_elu(PyObject* self_, PyObject* args, PyObject* kwargs);
```

- EN: The main execution path in this span is carried by `PyTuple_SET_ITEM`, `Py_INCREF`, `THPVariable__conv_depthwise2d`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyTuple_SET_ITEM`, `Py_INCREF`, `THPVariable__conv_depthwise2d` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 201-240

```cpp
201: static PyObject * THPVariable_elu_(PyObject* self_, PyObject* args, PyObject* kwargs);
202: static PyObject * THPVariable_flatten_dense_tensors(PyObject* self_, PyObject* args, PyObject* kwargs);
203: static PyObject * THPVariable_fractional_max_pool2d(PyObject* self_, PyObject* args, PyObject* kwargs);
204: static PyObject * THPVariable_fractional_max_pool3d(PyObject* self_, PyObject* args, PyObject* kwargs);
205: static PyObject * THPVariable_gelu(PyObject* self_, PyObject* args, PyObject* kwargs);
206: static PyObject * THPVariable_gelu_(PyObject* self_, PyObject* args, PyObject* kwargs);
207: static PyObject * THPVariable_glu(PyObject* self_, PyObject* args, PyObject* kwargs);
208: static PyObject * THPVariable_hardsigmoid(PyObject* self_, PyObject* args, PyObject* kwargs);
209: static PyObject * THPVariable_hardsigmoid_(PyObject* self_, PyObject* args, PyObject* kwargs);
210: static PyObject * THPVariable_hardswish(PyObject* self_, PyObject* args, PyObject* kwargs);
211: static PyObject * THPVariable_hardswish_(PyObject* self_, PyObject* args, PyObject* kwargs);
212: static PyObject * THPVariable_hardtanh(PyObject* self_, PyObject* args, PyObject* kwargs);
213: static PyObject * THPVariable_hardtanh_(PyObject* self_, PyObject* args, PyObject* kwargs);
214: static PyObject * THPVariable_huber_loss(PyObject* self_, PyObject* args, PyObject* kwargs);
215: static PyObject * THPVariable_im2col(PyObject* self_, PyObject* args, PyObject* kwargs);
216: static PyObject * THPVariable_l1_loss(PyObject* self_, PyObject* args, PyObject* kwargs);
217: static PyObject * THPVariable_leaky_relu(PyObject* self_, PyObject* args, PyObject* kwargs);
218: static PyObject * THPVariable_leaky_relu_(PyObject* self_, PyObject* args, PyObject* kwargs);
219: static PyObject * THPVariable_linear(PyObject* self_, PyObject* args, PyObject* kwargs);
220: static PyObject * THPVariable_log_sigmoid(PyObject* self_, PyObject* args, PyObject* kwargs);
221: static PyObject * THPVariable_max_pool2d_with_indices(PyObject* self_, PyObject* args, PyObject* kwargs);
222: static PyObject * THPVariable_max_pool3d_with_indices(PyObject* self_, PyObject* args, PyObject* kwargs);
223: static PyObject * THPVariable_max_unpool2d(PyObject* self_, PyObject* args, PyObject* kwargs);
224: static PyObject * THPVariable_max_unpool3d(PyObject* self_, PyObject* args, PyObject* kwargs);
225: static PyObject * THPVariable_mish(PyObject* self_, PyObject* args, PyObject* kwargs);
226: static PyObject * THPVariable_mish_(PyObject* self_, PyObject* args, PyObject* kwargs);
227: static PyObject * THPVariable_mkldnn_linear(PyObject* self_, PyObject* args, PyObject* kwargs);
228: static PyObject * THPVariable_mkldnn_reorder_conv2d_weight(PyObject* self_, PyObject* args, PyObject* kwargs);
229: static PyObject * THPVariable_mkldnn_reorder_conv3d_weight(PyObject* self_, PyObject* args, PyObject* kwargs);
230: static PyObject * THPVariable_mse_loss(PyObject* self_, PyObject* args, PyObject* kwargs);
231: static PyObject * THPVariable_multi_margin_loss(PyObject* self_, PyObject* args, PyObject* kwargs);
232: static PyObject * THPVariable_multilabel_margin_loss(PyObject* self_, PyObject* args, PyObject* kwargs);
233: static PyObject * THPVariable_nll_loss(PyObject* self_, PyObject* args, PyObject* kwargs);
234: static PyObject * THPVariable_nll_loss2d(PyObject* self_, PyObject* args, PyObject* kwargs);
235: static PyObject * THPVariable_nll_loss_nd(PyObject* self_, PyObject* args, PyObject* kwargs);
236: static PyObject * THPVariable_one_hot(PyObject* self_, PyObject* args, PyObject* kwargs);
237: static PyObject * THPVariable_pad(PyObject* self_, PyObject* args, PyObject* kwargs);
238: static PyObject * THPVariable_pad_sequence(PyObject* self_, PyObject* args, PyObject* kwargs);
239: static PyObject * THPVariable_reflection_pad1d(PyObject* self_, PyObject* args, PyObject* kwargs);
240: static PyObject * THPVariable_reflection_pad2d(PyObject* self_, PyObject* args, PyObject* kwargs);
```

- EN: The main execution path in this span is carried by `THPVariable_elu_`, `THPVariable_flatten_dense_tensors`, `THPVariable_fractional_max_pool2d`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_elu_`, `THPVariable_flatten_dense_tensors`, `THPVariable_fractional_max_pool2d` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 241-280

```cpp
241: static PyObject * THPVariable_reflection_pad3d(PyObject* self_, PyObject* args, PyObject* kwargs);
242: static PyObject * THPVariable_relu6(PyObject* self_, PyObject* args, PyObject* kwargs);
243: static PyObject * THPVariable_relu6_(PyObject* self_, PyObject* args, PyObject* kwargs);
244: static PyObject * THPVariable_replication_pad1d(PyObject* self_, PyObject* args, PyObject* kwargs);
245: static PyObject * THPVariable_replication_pad2d(PyObject* self_, PyObject* args, PyObject* kwargs);
246: static PyObject * THPVariable_replication_pad3d(PyObject* self_, PyObject* args, PyObject* kwargs);
247: static PyObject * THPVariable_rrelu_with_noise(PyObject* self_, PyObject* args, PyObject* kwargs);
248: static PyObject * THPVariable_rrelu_with_noise_(PyObject* self_, PyObject* args, PyObject* kwargs);
249: static PyObject * THPVariable_scaled_dot_product_attention(PyObject* self_, PyObject* args, PyObject* kwargs);
250: static PyObject * THPVariable_silu(PyObject* self_, PyObject* args, PyObject* kwargs);
251: static PyObject * THPVariable_silu_(PyObject* self_, PyObject* args, PyObject* kwargs);
252: static PyObject * THPVariable_slow_conv3d(PyObject* self_, PyObject* args, PyObject* kwargs);
253: static PyObject * THPVariable_slow_conv_dilated2d(PyObject* self_, PyObject* args, PyObject* kwargs);
254: static PyObject * THPVariable_slow_conv_dilated3d(PyObject* self_, PyObject* args, PyObject* kwargs);
255: static PyObject * THPVariable_slow_conv_transpose2d(PyObject* self_, PyObject* args, PyObject* kwargs);
256: static PyObject * THPVariable_slow_conv_transpose3d(PyObject* self_, PyObject* args, PyObject* kwargs);
257: static PyObject * THPVariable_smooth_l1_loss(PyObject* self_, PyObject* args, PyObject* kwargs);
258: static PyObject * THPVariable_soft_margin_loss(PyObject* self_, PyObject* args, PyObject* kwargs);
259: static PyObject * THPVariable_softplus(PyObject* self_, PyObject* args, PyObject* kwargs);
260: static PyObject * THPVariable_softshrink(PyObject* self_, PyObject* args, PyObject* kwargs);
261: static PyObject * THPVariable_thnn_conv2d(PyObject* self_, PyObject* args, PyObject* kwargs);
262: static PyObject * THPVariable_unflatten_dense_tensors(PyObject* self_, PyObject* args, PyObject* kwargs);
263: static PyObject * THPVariable_upsample_bicubic2d(PyObject* self_, PyObject* args, PyObject* kwargs);
264: static PyObject * THPVariable_upsample_bilinear2d(PyObject* self_, PyObject* args, PyObject* kwargs);
265: static PyObject * THPVariable_upsample_linear1d(PyObject* self_, PyObject* args, PyObject* kwargs);
266: static PyObject * THPVariable_upsample_nearest1d(PyObject* self_, PyObject* args, PyObject* kwargs);
267: static PyObject * THPVariable_upsample_nearest2d(PyObject* self_, PyObject* args, PyObject* kwargs);
268: static PyObject * THPVariable_upsample_nearest3d(PyObject* self_, PyObject* args, PyObject* kwargs);
269: static PyObject * THPVariable_upsample_trilinear3d(PyObject* self_, PyObject* args, PyObject* kwargs);
270: 
271: static PyMethodDef nn_functions[] = {
272:   {"_parse_to", castPyCFunctionWithKeywords(THPVariable__parse_to),
273:     METH_VARARGS | METH_KEYWORDS, nullptr},
274:   {"_conv_depthwise2d", castPyCFunctionWithKeywords(THPVariable__conv_depthwise2d), METH_VARARGS | METH_KEYWORDS, nullptr},
275:   {"_pad_circular", castPyCFunctionWithKeywords(THPVariable__pad_circular), METH_VARARGS | METH_KEYWORDS, nullptr},
276:   {"_pad_enum", castPyCFunctionWithKeywords(THPVariable__pad_enum), METH_VARARGS | METH_KEYWORDS, nullptr},
277:   {"_test_ambiguous_defaults", castPyCFunctionWithKeywords(THPVariable__test_ambiguous_defaults), METH_VARARGS | METH_KEYWORDS, nullptr},
278:   {"_test_optional_filled_intlist", castPyCFunctionWithKeywords(THPVariable__test_optional_filled_intlist), METH_VARARGS | METH_KEYWORDS, nullptr},
279:   {"_test_optional_floatlist", castPyCFunctionWithKeywords(THPVariable__test_optional_floatlist), METH_VARARGS | METH_KEYWORDS, nullptr},
280:   {"_test_optional_intlist", castPyCFunctionWithKeywords(THPVariable__test_optional_intlist), METH_VARARGS | METH_KEYWORDS, nullptr},
```

- EN: The main execution path in this span is carried by `THPVariable_reflection_pad3d`, `THPVariable_relu6`, `THPVariable_relu6_`. Because this is generated binding code, the span mostly registers or forwards APIs into a mechanically produced Python-facing surface. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_reflection_pad3d`, `THPVariable_relu6`, `THPVariable_relu6_` 等函数/方法承载。 由于这是生成的绑定代码，这一段主要是在机械化生成的 Python 接口层上完成 API 注册或转发。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 281-320

```cpp
281:   {"_test_string_default", castPyCFunctionWithKeywords(THPVariable__test_string_default), METH_VARARGS | METH_KEYWORDS, nullptr},
282:   {"_test_warn_in_autograd", castPyCFunctionWithKeywords(THPVariable__test_warn_in_autograd), METH_VARARGS | METH_KEYWORDS, nullptr},
283:   {"_upsample_bicubic2d_aa", castPyCFunctionWithKeywords(THPVariable__upsample_bicubic2d_aa), METH_VARARGS | METH_KEYWORDS, nullptr},
284:   {"_upsample_bilinear2d_aa", castPyCFunctionWithKeywords(THPVariable__upsample_bilinear2d_aa), METH_VARARGS | METH_KEYWORDS, nullptr},
285:   {"_upsample_lanczos2d_aa", castPyCFunctionWithKeywords(THPVariable__upsample_lanczos2d_aa), METH_VARARGS | METH_KEYWORDS, nullptr},
286:   {"_upsample_nearest_exact1d", castPyCFunctionWithKeywords(THPVariable__upsample_nearest_exact1d), METH_VARARGS | METH_KEYWORDS, nullptr},
287:   {"_upsample_nearest_exact2d", castPyCFunctionWithKeywords(THPVariable__upsample_nearest_exact2d), METH_VARARGS | METH_KEYWORDS, nullptr},
288:   {"_upsample_nearest_exact3d", castPyCFunctionWithKeywords(THPVariable__upsample_nearest_exact3d), METH_VARARGS | METH_KEYWORDS, nullptr},
289:   {"adaptive_avg_pool2d", castPyCFunctionWithKeywords(THPVariable_adaptive_avg_pool2d), METH_VARARGS | METH_KEYWORDS, nullptr},
290:   {"adaptive_avg_pool3d", castPyCFunctionWithKeywords(THPVariable_adaptive_avg_pool3d), METH_VARARGS | METH_KEYWORDS, nullptr},
291:   {"adaptive_max_pool2d", castPyCFunctionWithKeywords(THPVariable_adaptive_max_pool2d), METH_VARARGS | METH_KEYWORDS, nullptr},
292:   {"adaptive_max_pool3d", castPyCFunctionWithKeywords(THPVariable_adaptive_max_pool3d), METH_VARARGS | METH_KEYWORDS, nullptr},
293:   {"avg_pool2d", castPyCFunctionWithKeywords(THPVariable_avg_pool2d), METH_VARARGS | METH_KEYWORDS, nullptr},
294:   {"avg_pool3d", castPyCFunctionWithKeywords(THPVariable_avg_pool3d), METH_VARARGS | METH_KEYWORDS, nullptr},
295:   {"binary_cross_entropy", castPyCFunctionWithKeywords(THPVariable_binary_cross_entropy), METH_VARARGS | METH_KEYWORDS, nullptr},
296:   {"col2im", castPyCFunctionWithKeywords(THPVariable_col2im), METH_VARARGS | METH_KEYWORDS, nullptr},
297:   {"conv_depthwise3d", castPyCFunctionWithKeywords(THPVariable_conv_depthwise3d), METH_VARARGS | METH_KEYWORDS, nullptr},
298:   {"cross_entropy_loss", castPyCFunctionWithKeywords(THPVariable_cross_entropy_loss), METH_VARARGS | METH_KEYWORDS, nullptr},
299:   {"elu", castPyCFunctionWithKeywords(THPVariable_elu), METH_VARARGS | METH_KEYWORDS, nullptr},
300:   {"elu_", castPyCFunctionWithKeywords(THPVariable_elu_), METH_VARARGS | METH_KEYWORDS, nullptr},
301:   {"flatten_dense_tensors", castPyCFunctionWithKeywords(THPVariable_flatten_dense_tensors), METH_VARARGS | METH_KEYWORDS, nullptr},
302:   {"fractional_max_pool2d", castPyCFunctionWithKeywords(THPVariable_fractional_max_pool2d), METH_VARARGS | METH_KEYWORDS, nullptr},
303:   {"fractional_max_pool3d", castPyCFunctionWithKeywords(THPVariable_fractional_max_pool3d), METH_VARARGS | METH_KEYWORDS, nullptr},
304:   {"gelu", castPyCFunctionWithKeywords(THPVariable_gelu), METH_VARARGS | METH_KEYWORDS, nullptr},
305:   {"gelu_", castPyCFunctionWithKeywords(THPVariable_gelu_), METH_VARARGS | METH_KEYWORDS, nullptr},
306:   {"glu", castPyCFunctionWithKeywords(THPVariable_glu), METH_VARARGS | METH_KEYWORDS, nullptr},
307:   {"hardsigmoid", castPyCFunctionWithKeywords(THPVariable_hardsigmoid), METH_VARARGS | METH_KEYWORDS, nullptr},
308:   {"hardsigmoid_", castPyCFunctionWithKeywords(THPVariable_hardsigmoid_), METH_VARARGS | METH_KEYWORDS, nullptr},
309:   {"hardswish", castPyCFunctionWithKeywords(THPVariable_hardswish), METH_VARARGS | METH_KEYWORDS, nullptr},
310:   {"hardswish_", castPyCFunctionWithKeywords(THPVariable_hardswish_), METH_VARARGS | METH_KEYWORDS, nullptr},
311:   {"hardtanh", castPyCFunctionWithKeywords(THPVariable_hardtanh), METH_VARARGS | METH_KEYWORDS, nullptr},
312:   {"hardtanh_", castPyCFunctionWithKeywords(THPVariable_hardtanh_), METH_VARARGS | METH_KEYWORDS, nullptr},
313:   {"huber_loss", castPyCFunctionWithKeywords(THPVariable_huber_loss), METH_VARARGS | METH_KEYWORDS, nullptr},
314:   {"im2col", castPyCFunctionWithKeywords(THPVariable_im2col), METH_VARARGS | METH_KEYWORDS, nullptr},
315:   {"l1_loss", castPyCFunctionWithKeywords(THPVariable_l1_loss), METH_VARARGS | METH_KEYWORDS, nullptr},
316:   {"leaky_relu", castPyCFunctionWithKeywords(THPVariable_leaky_relu), METH_VARARGS | METH_KEYWORDS, nullptr},
317:   {"leaky_relu_", castPyCFunctionWithKeywords(THPVariable_leaky_relu_), METH_VARARGS | METH_KEYWORDS, nullptr},
318:   {"linear", castPyCFunctionWithKeywords(THPVariable_linear), METH_VARARGS | METH_KEYWORDS, nullptr},
319:   {"log_sigmoid", castPyCFunctionWithKeywords(THPVariable_log_sigmoid), METH_VARARGS | METH_KEYWORDS, nullptr},
320:   {"max_pool2d_with_indices", castPyCFunctionWithKeywords(THPVariable_max_pool2d_with_indices), METH_VARARGS | METH_KEYWORDS, nullptr},
```

- EN: The main execution path in this span is carried by `castPyCFunctionWithKeywords`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `castPyCFunctionWithKeywords` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 321-360

```cpp
321:   {"max_pool3d_with_indices", castPyCFunctionWithKeywords(THPVariable_max_pool3d_with_indices), METH_VARARGS | METH_KEYWORDS, nullptr},
322:   {"max_unpool2d", castPyCFunctionWithKeywords(THPVariable_max_unpool2d), METH_VARARGS | METH_KEYWORDS, nullptr},
323:   {"max_unpool3d", castPyCFunctionWithKeywords(THPVariable_max_unpool3d), METH_VARARGS | METH_KEYWORDS, nullptr},
324:   {"mish", castPyCFunctionWithKeywords(THPVariable_mish), METH_VARARGS | METH_KEYWORDS, nullptr},
325:   {"mish_", castPyCFunctionWithKeywords(THPVariable_mish_), METH_VARARGS | METH_KEYWORDS, nullptr},
326:   {"mkldnn_linear", castPyCFunctionWithKeywords(THPVariable_mkldnn_linear), METH_VARARGS | METH_KEYWORDS, nullptr},
327:   {"mkldnn_reorder_conv2d_weight", castPyCFunctionWithKeywords(THPVariable_mkldnn_reorder_conv2d_weight), METH_VARARGS | METH_KEYWORDS, nullptr},
328:   {"mkldnn_reorder_conv3d_weight", castPyCFunctionWithKeywords(THPVariable_mkldnn_reorder_conv3d_weight), METH_VARARGS | METH_KEYWORDS, nullptr},
329:   {"mse_loss", castPyCFunctionWithKeywords(THPVariable_mse_loss), METH_VARARGS | METH_KEYWORDS, nullptr},
330:   {"multi_margin_loss", castPyCFunctionWithKeywords(THPVariable_multi_margin_loss), METH_VARARGS | METH_KEYWORDS, nullptr},
331:   {"multilabel_margin_loss", castPyCFunctionWithKeywords(THPVariable_multilabel_margin_loss), METH_VARARGS | METH_KEYWORDS, nullptr},
332:   {"nll_loss", castPyCFunctionWithKeywords(THPVariable_nll_loss), METH_VARARGS | METH_KEYWORDS, nullptr},
333:   {"nll_loss2d", castPyCFunctionWithKeywords(THPVariable_nll_loss2d), METH_VARARGS | METH_KEYWORDS, nullptr},
334:   {"nll_loss_nd", castPyCFunctionWithKeywords(THPVariable_nll_loss_nd), METH_VARARGS | METH_KEYWORDS, nullptr},
335:   {"one_hot", castPyCFunctionWithKeywords(THPVariable_one_hot), METH_VARARGS | METH_KEYWORDS, nullptr},
336:   {"pad", castPyCFunctionWithKeywords(THPVariable_pad), METH_VARARGS | METH_KEYWORDS, nullptr},
337:   {"pad_sequence", castPyCFunctionWithKeywords(THPVariable_pad_sequence), METH_VARARGS | METH_KEYWORDS, nullptr},
338:   {"reflection_pad1d", castPyCFunctionWithKeywords(THPVariable_reflection_pad1d), METH_VARARGS | METH_KEYWORDS, nullptr},
339:   {"reflection_pad2d", castPyCFunctionWithKeywords(THPVariable_reflection_pad2d), METH_VARARGS | METH_KEYWORDS, nullptr},
340:   {"reflection_pad3d", castPyCFunctionWithKeywords(THPVariable_reflection_pad3d), METH_VARARGS | METH_KEYWORDS, nullptr},
341:   {"relu6", castPyCFunctionWithKeywords(THPVariable_relu6), METH_VARARGS | METH_KEYWORDS, nullptr},
342:   {"relu6_", castPyCFunctionWithKeywords(THPVariable_relu6_), METH_VARARGS | METH_KEYWORDS, nullptr},
343:   {"replication_pad1d", castPyCFunctionWithKeywords(THPVariable_replication_pad1d), METH_VARARGS | METH_KEYWORDS, nullptr},
344:   {"replication_pad2d", castPyCFunctionWithKeywords(THPVariable_replication_pad2d), METH_VARARGS | METH_KEYWORDS, nullptr},
345:   {"replication_pad3d", castPyCFunctionWithKeywords(THPVariable_replication_pad3d), METH_VARARGS | METH_KEYWORDS, nullptr},
346:   {"rrelu_with_noise", castPyCFunctionWithKeywords(THPVariable_rrelu_with_noise), METH_VARARGS | METH_KEYWORDS, nullptr},
347:   {"rrelu_with_noise_", castPyCFunctionWithKeywords(THPVariable_rrelu_with_noise_), METH_VARARGS | METH_KEYWORDS, nullptr},
348:   {"scaled_dot_product_attention", castPyCFunctionWithKeywords(THPVariable_scaled_dot_product_attention), METH_VARARGS | METH_KEYWORDS, nullptr},
349:   {"silu", castPyCFunctionWithKeywords(THPVariable_silu), METH_VARARGS | METH_KEYWORDS, nullptr},
350:   {"silu_", castPyCFunctionWithKeywords(THPVariable_silu_), METH_VARARGS | METH_KEYWORDS, nullptr},
351:   {"slow_conv3d", castPyCFunctionWithKeywords(THPVariable_slow_conv3d), METH_VARARGS | METH_KEYWORDS, nullptr},
352:   {"slow_conv_dilated2d", castPyCFunctionWithKeywords(THPVariable_slow_conv_dilated2d), METH_VARARGS | METH_KEYWORDS, nullptr},
353:   {"slow_conv_dilated3d", castPyCFunctionWithKeywords(THPVariable_slow_conv_dilated3d), METH_VARARGS | METH_KEYWORDS, nullptr},
354:   {"slow_conv_transpose2d", castPyCFunctionWithKeywords(THPVariable_slow_conv_transpose2d), METH_VARARGS | METH_KEYWORDS, nullptr},
355:   {"slow_conv_transpose3d", castPyCFunctionWithKeywords(THPVariable_slow_conv_transpose3d), METH_VARARGS | METH_KEYWORDS, nullptr},
356:   {"smooth_l1_loss", castPyCFunctionWithKeywords(THPVariable_smooth_l1_loss), METH_VARARGS | METH_KEYWORDS, nullptr},
357:   {"soft_margin_loss", castPyCFunctionWithKeywords(THPVariable_soft_margin_loss), METH_VARARGS | METH_KEYWORDS, nullptr},
358:   {"softplus", castPyCFunctionWithKeywords(THPVariable_softplus), METH_VARARGS | METH_KEYWORDS, nullptr},
359:   {"softshrink", castPyCFunctionWithKeywords(THPVariable_softshrink), METH_VARARGS | METH_KEYWORDS, nullptr},
360:   {"thnn_conv2d", castPyCFunctionWithKeywords(THPVariable_thnn_conv2d), METH_VARARGS | METH_KEYWORDS, nullptr},
```

- EN: The main execution path in this span is carried by `castPyCFunctionWithKeywords`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `castPyCFunctionWithKeywords` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 361-400

```cpp
361:   {"unflatten_dense_tensors", castPyCFunctionWithKeywords(THPVariable_unflatten_dense_tensors), METH_VARARGS | METH_KEYWORDS, nullptr},
362:   {"upsample_bicubic2d", castPyCFunctionWithKeywords(THPVariable_upsample_bicubic2d), METH_VARARGS | METH_KEYWORDS, nullptr},
363:   {"upsample_bilinear2d", castPyCFunctionWithKeywords(THPVariable_upsample_bilinear2d), METH_VARARGS | METH_KEYWORDS, nullptr},
364:   {"upsample_linear1d", castPyCFunctionWithKeywords(THPVariable_upsample_linear1d), METH_VARARGS | METH_KEYWORDS, nullptr},
365:   {"upsample_nearest1d", castPyCFunctionWithKeywords(THPVariable_upsample_nearest1d), METH_VARARGS | METH_KEYWORDS, nullptr},
366:   {"upsample_nearest2d", castPyCFunctionWithKeywords(THPVariable_upsample_nearest2d), METH_VARARGS | METH_KEYWORDS, nullptr},
367:   {"upsample_nearest3d", castPyCFunctionWithKeywords(THPVariable_upsample_nearest3d), METH_VARARGS | METH_KEYWORDS, nullptr},
368:   {"upsample_trilinear3d", castPyCFunctionWithKeywords(THPVariable_upsample_trilinear3d), METH_VARARGS | METH_KEYWORDS, nullptr},
369:   {nullptr}
370: };
371: 
372: void initNNFunctions(PyObject* module) {
373:   static struct PyModuleDef def = {
374:      PyModuleDef_HEAD_INIT,
375:      "torch._C._nn",
376:      nullptr,
377:      -1,
378:      nn_functions
379:   };
380:   PyObject* nn = PyModule_Create(&def);
381:   THPNNVariableFunctionsModule = nn;
382:   if (!nn) {
383:     throw python_error();
384:   }
385:   // steals a reference to nn
386:   if (PyModule_AddObject(module, "_nn", nn) != 0) {
387:     throw python_error();
388:   }
389: }
390: 
391: // generated methods start here
392: 
393: // _conv_depthwise2d
394: static PyObject * THPVariable__conv_depthwise2d(PyObject* self_, PyObject* args, PyObject* kwargs)
395: {
396:   HANDLE_TH_ERRORS
397:   static PythonArgParser parser({
398:     "_conv_depthwise2d(Tensor input, Tensor weight, SymIntArrayRef[2] kernel_size, Tensor? bias, SymIntArrayRef[2] stride, SymIntArrayRef[2] padding, SymIntArrayRef[2] dilation, *, Tensor out=None)",
399:   }, /*traceable=*/true);
400: 
```

- EN: The main execution path in this span is carried by `castPyCFunctionWithKeywords`, `initNNFunctions`, `PyModule_Create`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `castPyCFunctionWithKeywords`, `initNNFunctions`, `PyModule_Create` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 401-440

```cpp
401:   ParsedArgs<8> parsed_args;
402:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
403:   if(_r.has_torch_function()) {
404:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
405:   }
406:   if (_r.isNone(7)) {
407:     // aten::_conv_depthwise2d(Tensor self, Tensor weight, SymInt[2] kernel_size, Tensor? bias, SymInt[2] stride, SymInt[2] padding, SymInt[2] dilation) -> Tensor
408: 
409:     auto dispatch__conv_depthwise2d = [](const at::Tensor & self, const at::Tensor & weight, c10::SymIntArrayRef kernel_size, const ::std::optional<at::Tensor> & bias, c10::SymIntArrayRef stride, c10::SymIntArrayRef padding, c10::SymIntArrayRef dilation) -> at::Tensor {
410:       pybind11::gil_scoped_release no_gil;
411:       return at::_conv_depthwise2d_symint(self, weight, kernel_size, bias, stride, padding, dilation);
412:     };
413:     return wrap(dispatch__conv_depthwise2d(_r.tensor(0), _r.tensor(1), _r.symintlist(2), _r.optionalTensor(3), _r.symintlist(4), _r.symintlist(5), _r.symintlist(6)));
414:   } else {
415:     // aten::_conv_depthwise2d.out(Tensor self, Tensor weight, SymInt[2] kernel_size, Tensor? bias, SymInt[2] stride, SymInt[2] padding, SymInt[2] dilation, *, Tensor(a!) out) -> Tensor(a!)
416: 
417:     auto dispatch__conv_depthwise2d_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & weight, c10::SymIntArrayRef kernel_size, const ::std::optional<at::Tensor> & bias, c10::SymIntArrayRef stride, c10::SymIntArrayRef padding, c10::SymIntArrayRef dilation) -> at::Tensor {
418:       pybind11::gil_scoped_release no_gil;
419:       return at::_conv_depthwise2d_symint_out(out, self, weight, kernel_size, bias, stride, padding, dilation);
420:     };
421:     return wrap(dispatch__conv_depthwise2d_out(_r.tensor(7), _r.tensor(0), _r.tensor(1), _r.symintlist(2), _r.optionalTensor(3), _r.symintlist(4), _r.symintlist(5), _r.symintlist(6)));
422:   }
423:   Py_RETURN_NONE;
424:   END_HANDLE_TH_ERRORS
425: }
426: 
427: // _pad_circular
428: static PyObject * THPVariable__pad_circular(PyObject* self_, PyObject* args, PyObject* kwargs)
429: {
430:   HANDLE_TH_ERRORS
431:   static PythonArgParser parser({
432:     "_pad_circular(Tensor input, SymIntArrayRef pad)",
433:   }, /*traceable=*/true);
434: 
435:   ParsedArgs<2> parsed_args;
436:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
437:   if(_r.has_torch_function()) {
438:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
439:   }
440:   // aten::_pad_circular(Tensor self, SymInt[] pad) -> Tensor
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `_conv_depthwise2d`, `_conv_depthwise2d_symint`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `_conv_depthwise2d`, `_conv_depthwise2d_symint` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 441-480

```cpp
441: 
442:   auto dispatch__pad_circular = [](const at::Tensor & self, c10::SymIntArrayRef pad) -> at::Tensor {
443:     pybind11::gil_scoped_release no_gil;
444:     return at::_pad_circular_symint(self, pad);
445:   };
446:   return wrap(dispatch__pad_circular(_r.tensor(0), _r.symintlist(1)));
447:   Py_RETURN_NONE;
448:   END_HANDLE_TH_ERRORS
449: }
450: 
451: // _pad_enum
452: static PyObject * THPVariable__pad_enum(PyObject* self_, PyObject* args, PyObject* kwargs)
453: {
454:   HANDLE_TH_ERRORS
455:   static PythonArgParser parser({
456:     "_pad_enum(Tensor input, SymIntArrayRef pad, int64_t mode, double? value=None)",
457:   }, /*traceable=*/true);
458: 
459:   ParsedArgs<4> parsed_args;
460:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
461:   if(_r.has_torch_function()) {
462:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
463:   }
464:   // aten::_pad_enum(Tensor self, SymInt[] pad, int mode, float? value=None) -> Tensor
465: 
466:   auto dispatch__pad_enum = [](const at::Tensor & self, c10::SymIntArrayRef pad, int64_t mode, ::std::optional<double> value) -> at::Tensor {
467:     pybind11::gil_scoped_release no_gil;
468:     return at::_pad_enum_symint(self, pad, mode, value);
469:   };
470:   return wrap(dispatch__pad_enum(_r.tensor(0), _r.symintlist(1), _r.toInt64(2), _r.toDoubleOptional(3)));
471:   Py_RETURN_NONE;
472:   END_HANDLE_TH_ERRORS
473: }
474: 
475: \
476: // _test_ambiguous_defaults
477: static PyObject * THPVariable__test_ambiguous_defaults(PyObject* self_, PyObject* args, PyObject* kwargs)
478: {
479:   HANDLE_TH_ERRORS
480:   static PythonArgParser parser({
```

- EN: The main execution path in this span is carried by `_pad_circular_symint`, `wrap`, `THPVariable__pad_enum`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `_pad_circular_symint`, `wrap`, `THPVariable__pad_enum` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 481-520

```cpp
481:     "_test_ambiguous_defaults(Tensor dummy, int64_t a=1, int64_t b=1)",
482:     "_test_ambiguous_defaults(Tensor dummy, int64_t a=2, c10::string_view b=\"2\")",
483:   }, /*traceable=*/true);
484: 
485:   ParsedArgs<3> parsed_args;
486:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
487:   if(_r.has_torch_function()) {
488:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
489:   }
490:   switch (_r.idx) {
491:     case 0: {
492:       // aten::_test_ambiguous_defaults.a(Tensor dummy, int a=1, int b=1) -> Tensor
493: 
494:       auto dispatch__test_ambiguous_defaults = [](const at::Tensor & dummy, int64_t a, int64_t b) -> at::Tensor {
495:         pybind11::gil_scoped_release no_gil;
496:         return at::_test_ambiguous_defaults(dummy, a, b);
497:       };
498:       return wrap(dispatch__test_ambiguous_defaults(_r.tensor(0), _r.toInt64(1), _r.toInt64(2)));
499:     }
500:     case 1: {
501:       // aten::_test_ambiguous_defaults.b(Tensor dummy, int a=2, str b="2") -> Tensor
502: 
503:       auto dispatch__test_ambiguous_defaults = [](const at::Tensor & dummy, int64_t a, c10::string_view b) -> at::Tensor {
504:         pybind11::gil_scoped_release no_gil;
505:         return at::_test_ambiguous_defaults(dummy, a, b);
506:       };
507:       return wrap(dispatch__test_ambiguous_defaults(_r.tensor(0), _r.toInt64(1), _r.stringView(2)));
508:     }
509:   }
510:   Py_RETURN_NONE;
511:   END_HANDLE_TH_ERRORS
512: }
513: 
514: // _test_optional_filled_intlist
515: static PyObject * THPVariable__test_optional_filled_intlist(PyObject* self_, PyObject* args, PyObject* kwargs)
516: {
517:   HANDLE_TH_ERRORS
518:   static PythonArgParser parser({
519:     "_test_optional_filled_intlist(Tensor values, IntArrayRef[2]? addends)",
520:   }, /*traceable=*/true);
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `_test_ambiguous_defaults`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `_test_ambiguous_defaults`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 521-560

```cpp
521: 
522:   ParsedArgs<2> parsed_args;
523:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
524:   if(_r.has_torch_function()) {
525:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
526:   }
527:   // aten::_test_optional_filled_intlist(Tensor values, int[2]? addends) -> Tensor
528: 
529:   auto dispatch__test_optional_filled_intlist = [](const at::Tensor & values, at::OptionalIntArrayRef addends) -> at::Tensor {
530:     pybind11::gil_scoped_release no_gil;
531:     return at::_test_optional_filled_intlist(values, addends);
532:   };
533:   return wrap(dispatch__test_optional_filled_intlist(_r.tensor(0), _r.intlistOptional(1)));
534:   Py_RETURN_NONE;
535:   END_HANDLE_TH_ERRORS
536: }
537: 
538: // _test_optional_floatlist
539: static PyObject * THPVariable__test_optional_floatlist(PyObject* self_, PyObject* args, PyObject* kwargs)
540: {
541:   HANDLE_TH_ERRORS
542:   static PythonArgParser parser({
543:     "_test_optional_floatlist(Tensor values, ArrayRef<double>? addends)",
544:   }, /*traceable=*/true);
545: 
546:   ParsedArgs<2> parsed_args;
547:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
548:   if(_r.has_torch_function()) {
549:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
550:   }
551:   // aten::_test_optional_floatlist(Tensor values, float[]? addends) -> Tensor
552: 
553:   auto dispatch__test_optional_floatlist = [](const at::Tensor & values, ::std::optional<at::ArrayRef<double>> addends) -> at::Tensor {
554:     pybind11::gil_scoped_release no_gil;
555:     return at::_test_optional_floatlist(values, addends);
556:   };
557:   return wrap(dispatch__test_optional_floatlist(_r.tensor(0), _r.doublelistOptional(1)));
558:   Py_RETURN_NONE;
559:   END_HANDLE_TH_ERRORS
560: }
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `_test_optional_filled_intlist`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `_test_optional_filled_intlist`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 561-600

```cpp
561: 
562: // _test_optional_intlist
563: static PyObject * THPVariable__test_optional_intlist(PyObject* self_, PyObject* args, PyObject* kwargs)
564: {
565:   HANDLE_TH_ERRORS
566:   static PythonArgParser parser({
567:     "_test_optional_intlist(Tensor values, IntArrayRef? addends)",
568:   }, /*traceable=*/true);
569: 
570:   ParsedArgs<2> parsed_args;
571:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
572:   if(_r.has_torch_function()) {
573:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
574:   }
575:   // aten::_test_optional_intlist(Tensor values, int[]? addends) -> Tensor
576: 
577:   auto dispatch__test_optional_intlist = [](const at::Tensor & values, at::OptionalIntArrayRef addends) -> at::Tensor {
578:     pybind11::gil_scoped_release no_gil;
579:     return at::_test_optional_intlist(values, addends);
580:   };
581:   return wrap(dispatch__test_optional_intlist(_r.tensor(0), _r.intlistOptional(1)));
582:   Py_RETURN_NONE;
583:   END_HANDLE_TH_ERRORS
584: }
585: 
586: // _test_string_default
587: static PyObject * THPVariable__test_string_default(PyObject* self_, PyObject* args, PyObject* kwargs)
588: {
589:   HANDLE_TH_ERRORS
590:   static PythonArgParser parser({
591:     "_test_string_default(Tensor dummy, c10::string_view a=\"\\\"'\\\\\", c10::string_view b=\"\\\"'\\\\\")",
592:   }, /*traceable=*/true);
593: 
594:   ParsedArgs<3> parsed_args;
595:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
596:   if(_r.has_torch_function()) {
597:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
598:   }
599:   // aten::_test_string_default(Tensor dummy, str a="\"'\\", str b='"\'\\') -> Tensor
600: 
```

- EN: The main execution path in this span is carried by `THPVariable__test_optional_intlist`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable__test_optional_intlist`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 601-640

```cpp
601:   auto dispatch__test_string_default = [](const at::Tensor & dummy, c10::string_view a, c10::string_view b) -> at::Tensor {
602:     pybind11::gil_scoped_release no_gil;
603:     return at::_test_string_default(dummy, a, b);
604:   };
605:   return wrap(dispatch__test_string_default(_r.tensor(0), _r.stringView(1), _r.stringView(2)));
606:   Py_RETURN_NONE;
607:   END_HANDLE_TH_ERRORS
608: }
609: 
610: // _test_warn_in_autograd
611: static PyObject * THPVariable__test_warn_in_autograd(PyObject* self_, PyObject* args, PyObject* kwargs)
612: {
613:   HANDLE_TH_ERRORS
614:   static PythonArgParser parser({
615:     "_test_warn_in_autograd(Tensor input)",
616:   }, /*traceable=*/true);
617: 
618:   ParsedArgs<1> parsed_args;
619:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
620:   if(_r.has_torch_function()) {
621:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
622:   }
623:   // aten::_test_warn_in_autograd(Tensor self) -> Tensor
624: 
625:   auto dispatch__test_warn_in_autograd = [](const at::Tensor & self) -> at::Tensor {
626:     pybind11::gil_scoped_release no_gil;
627:     return at::_test_warn_in_autograd(self);
628:   };
629:   return wrap(dispatch__test_warn_in_autograd(_r.tensor(0)));
630:   Py_RETURN_NONE;
631:   END_HANDLE_TH_ERRORS
632: }
633: 
634: \
635: // _upsample_bicubic2d_aa
636: static PyObject * THPVariable__upsample_bicubic2d_aa(PyObject* self_, PyObject* args, PyObject* kwargs)
637: {
638:   HANDLE_TH_ERRORS
639:   static PythonArgParser parser({
640:     "_upsample_bicubic2d_aa(Tensor input, SymIntArrayRef? output_size, bool align_corners, ArrayRef<double>? scale_factors)",
```

- EN: The main execution path in this span is carried by `_test_string_default`, `wrap`, `THPVariable__test_warn_in_autograd`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `_test_string_default`, `wrap`, `THPVariable__test_warn_in_autograd` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 641-680

```cpp
641:     "_upsample_bicubic2d_aa(Tensor input, SymIntArrayRef[2] output_size, bool align_corners, double? scales_h=None, double? scales_w=None, *, Tensor out=None)",
642:   }, /*traceable=*/true);
643: 
644:   ParsedArgs<6> parsed_args;
645:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
646:   if(_r.has_torch_function()) {
647:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
648:   }
649:   switch (_r.idx) {
650:     case 0: {
651:       // aten::_upsample_bicubic2d_aa.vec(Tensor input, SymInt[]? output_size, bool align_corners, float[]? scale_factors) -> Tensor
652: 
653:       auto dispatch__upsample_bicubic2d_aa = [](const at::Tensor & input, at::OptionalSymIntArrayRef output_size, bool align_corners, ::std::optional<at::ArrayRef<double>> scale_factors) -> at::Tensor {
654:         pybind11::gil_scoped_release no_gil;
655:         return at::_upsample_bicubic2d_aa_symint(input, output_size, align_corners, scale_factors);
656:       };
657:       return wrap(dispatch__upsample_bicubic2d_aa(_r.tensor(0), _r.symintlistOptional(1), _r.toBool(2), _r.doublelistOptional(3)));
658:     }
659:     case 1: {
660:       if (_r.isNone(5)) {
661:         // aten::_upsample_bicubic2d_aa(Tensor self, SymInt[2] output_size, bool align_corners, float? scales_h=None, float? scales_w=None) -> Tensor
662: 
663:         auto dispatch__upsample_bicubic2d_aa = [](const at::Tensor & self, c10::SymIntArrayRef output_size, bool align_corners, ::std::optional<double> scales_h, ::std::optional<double> scales_w) -> at::Tensor {
664:           pybind11::gil_scoped_release no_gil;
665:           return at::_upsample_bicubic2d_aa_symint(self, output_size, align_corners, scales_h, scales_w);
666:         };
667:         return wrap(dispatch__upsample_bicubic2d_aa(_r.tensor(0), _r.symintlist(1), _r.toBool(2), _r.toDoubleOptional(3), _r.toDoubleOptional(4)));
668:       } else {
669:         // aten::_upsample_bicubic2d_aa.out(Tensor self, SymInt[2] output_size, bool align_corners, float? scales_h=None, float? scales_w=None, *, Tensor(a!) out) -> Tensor(a!)
670: 
671:         auto dispatch__upsample_bicubic2d_aa_out = [](at::Tensor out, const at::Tensor & self, c10::SymIntArrayRef output_size, bool align_corners, ::std::optional<double> scales_h, ::std::optional<double> scales_w) -> at::Tensor {
672:           pybind11::gil_scoped_release no_gil;
673:           return at::_upsample_bicubic2d_aa_symint_out(out, self, output_size, align_corners, scales_h, scales_w);
674:         };
675:         return wrap(dispatch__upsample_bicubic2d_aa_out(_r.tensor(5), _r.tensor(0), _r.symintlist(1), _r.toBool(2), _r.toDoubleOptional(3), _r.toDoubleOptional(4)));
676:       }
677:     }
678:   }
679:   Py_RETURN_NONE;
680:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `_upsample_bicubic2d_aa_symint`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `_upsample_bicubic2d_aa_symint`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 681-720

```cpp
681: }
682: 
683: \
684: // _upsample_bilinear2d_aa
685: static PyObject * THPVariable__upsample_bilinear2d_aa(PyObject* self_, PyObject* args, PyObject* kwargs)
686: {
687:   HANDLE_TH_ERRORS
688:   static PythonArgParser parser({
689:     "_upsample_bilinear2d_aa(Tensor input, SymIntArrayRef? output_size, bool align_corners, ArrayRef<double>? scale_factors)",
690:     "_upsample_bilinear2d_aa(Tensor input, SymIntArrayRef[2] output_size, bool align_corners, double? scales_h=None, double? scales_w=None, *, Tensor out=None)",
691:   }, /*traceable=*/true);
692: 
693:   ParsedArgs<6> parsed_args;
694:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
695:   if(_r.has_torch_function()) {
696:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
697:   }
698:   switch (_r.idx) {
699:     case 0: {
700:       // aten::_upsample_bilinear2d_aa.vec(Tensor input, SymInt[]? output_size, bool align_corners, float[]? scale_factors) -> Tensor
701: 
702:       auto dispatch__upsample_bilinear2d_aa = [](const at::Tensor & input, at::OptionalSymIntArrayRef output_size, bool align_corners, ::std::optional<at::ArrayRef<double>> scale_factors) -> at::Tensor {
703:         pybind11::gil_scoped_release no_gil;
704:         return at::_upsample_bilinear2d_aa_symint(input, output_size, align_corners, scale_factors);
705:       };
706:       return wrap(dispatch__upsample_bilinear2d_aa(_r.tensor(0), _r.symintlistOptional(1), _r.toBool(2), _r.doublelistOptional(3)));
707:     }
708:     case 1: {
709:       if (_r.isNone(5)) {
710:         // aten::_upsample_bilinear2d_aa(Tensor self, SymInt[2] output_size, bool align_corners, float? scales_h=None, float? scales_w=None) -> Tensor
711: 
712:         auto dispatch__upsample_bilinear2d_aa = [](const at::Tensor & self, c10::SymIntArrayRef output_size, bool align_corners, ::std::optional<double> scales_h, ::std::optional<double> scales_w) -> at::Tensor {
713:           pybind11::gil_scoped_release no_gil;
714:           return at::_upsample_bilinear2d_aa_symint(self, output_size, align_corners, scales_h, scales_w);
715:         };
716:         return wrap(dispatch__upsample_bilinear2d_aa(_r.tensor(0), _r.symintlist(1), _r.toBool(2), _r.toDoubleOptional(3), _r.toDoubleOptional(4)));
717:       } else {
718:         // aten::_upsample_bilinear2d_aa.out(Tensor self, SymInt[2] output_size, bool align_corners, float? scales_h=None, float? scales_w=None, *, Tensor(a!) out) -> Tensor(a!)
719: 
720:         auto dispatch__upsample_bilinear2d_aa_out = [](at::Tensor out, const at::Tensor & self, c10::SymIntArrayRef output_size, bool align_corners, ::std::optional<double> scales_h, ::std::optional<double> scales_w) -> at::Tensor {
```

- EN: The main execution path in this span is carried by `THPVariable__upsample_bilinear2d_aa`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable__upsample_bilinear2d_aa`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 721-760

```cpp
721:           pybind11::gil_scoped_release no_gil;
722:           return at::_upsample_bilinear2d_aa_symint_out(out, self, output_size, align_corners, scales_h, scales_w);
723:         };
724:         return wrap(dispatch__upsample_bilinear2d_aa_out(_r.tensor(5), _r.tensor(0), _r.symintlist(1), _r.toBool(2), _r.toDoubleOptional(3), _r.toDoubleOptional(4)));
725:       }
726:     }
727:   }
728:   Py_RETURN_NONE;
729:   END_HANDLE_TH_ERRORS
730: }
731: 
732: \
733: // _upsample_lanczos2d_aa
734: static PyObject * THPVariable__upsample_lanczos2d_aa(PyObject* self_, PyObject* args, PyObject* kwargs)
735: {
736:   HANDLE_TH_ERRORS
737:   static PythonArgParser parser({
738:     "_upsample_lanczos2d_aa(Tensor input, SymIntArrayRef? output_size, bool align_corners, ArrayRef<double>? scale_factors)",
739:     "_upsample_lanczos2d_aa(Tensor input, SymIntArrayRef[2] output_size, bool align_corners, double? scales_h=None, double? scales_w=None, *, Tensor out=None)",
740:   }, /*traceable=*/true);
741: 
742:   ParsedArgs<6> parsed_args;
743:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
744:   if(_r.has_torch_function()) {
745:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
746:   }
747:   switch (_r.idx) {
748:     case 0: {
749:       // aten::_upsample_lanczos2d_aa.vec(Tensor input, SymInt[]? output_size, bool align_corners, float[]? scale_factors) -> Tensor
750: 
751:       auto dispatch__upsample_lanczos2d_aa = [](const at::Tensor & input, at::OptionalSymIntArrayRef output_size, bool align_corners, ::std::optional<at::ArrayRef<double>> scale_factors) -> at::Tensor {
752:         pybind11::gil_scoped_release no_gil;
753:         return at::_upsample_lanczos2d_aa_symint(input, output_size, align_corners, scale_factors);
754:       };
755:       return wrap(dispatch__upsample_lanczos2d_aa(_r.tensor(0), _r.symintlistOptional(1), _r.toBool(2), _r.doublelistOptional(3)));
756:     }
757:     case 1: {
758:       if (_r.isNone(5)) {
759:         // aten::_upsample_lanczos2d_aa(Tensor self, SymInt[2] output_size, bool align_corners, float? scales_h=None, float? scales_w=None) -> Tensor
760: 
```

- EN: The main execution path in this span is carried by `_upsample_bilinear2d_aa_symint_out`, `wrap`, `THPVariable__upsample_lanczos2d_aa`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `_upsample_bilinear2d_aa_symint_out`, `wrap`, `THPVariable__upsample_lanczos2d_aa` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 761-800

```cpp
761:         auto dispatch__upsample_lanczos2d_aa = [](const at::Tensor & self, c10::SymIntArrayRef output_size, bool align_corners, ::std::optional<double> scales_h, ::std::optional<double> scales_w) -> at::Tensor {
762:           pybind11::gil_scoped_release no_gil;
763:           return at::_upsample_lanczos2d_aa_symint(self, output_size, align_corners, scales_h, scales_w);
764:         };
765:         return wrap(dispatch__upsample_lanczos2d_aa(_r.tensor(0), _r.symintlist(1), _r.toBool(2), _r.toDoubleOptional(3), _r.toDoubleOptional(4)));
766:       } else {
767:         // aten::_upsample_lanczos2d_aa.out(Tensor self, SymInt[2] output_size, bool align_corners, float? scales_h=None, float? scales_w=None, *, Tensor(a!) out) -> Tensor(a!)
768: 
769:         auto dispatch__upsample_lanczos2d_aa_out = [](at::Tensor out, const at::Tensor & self, c10::SymIntArrayRef output_size, bool align_corners, ::std::optional<double> scales_h, ::std::optional<double> scales_w) -> at::Tensor {
770:           pybind11::gil_scoped_release no_gil;
771:           return at::_upsample_lanczos2d_aa_symint_out(out, self, output_size, align_corners, scales_h, scales_w);
772:         };
773:         return wrap(dispatch__upsample_lanczos2d_aa_out(_r.tensor(5), _r.tensor(0), _r.symintlist(1), _r.toBool(2), _r.toDoubleOptional(3), _r.toDoubleOptional(4)));
774:       }
775:     }
776:   }
777:   Py_RETURN_NONE;
778:   END_HANDLE_TH_ERRORS
779: }
780: 
781: \
782: // _upsample_nearest_exact1d
783: static PyObject * THPVariable__upsample_nearest_exact1d(PyObject* self_, PyObject* args, PyObject* kwargs)
784: {
785:   HANDLE_TH_ERRORS
786:   static PythonArgParser parser({
787:     "_upsample_nearest_exact1d(Tensor input, SymIntArrayRef? output_size, ArrayRef<double>? scale_factors)",
788:     "_upsample_nearest_exact1d(Tensor input, SymIntArrayRef[1] output_size, double? scales=None, *, Tensor out=None)",
789:   }, /*traceable=*/true);
790: 
791:   ParsedArgs<4> parsed_args;
792:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
793:   if(_r.has_torch_function()) {
794:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
795:   }
796:   switch (_r.idx) {
797:     case 0: {
798:       // aten::_upsample_nearest_exact1d.vec(Tensor input, SymInt[]? output_size, float[]? scale_factors) -> Tensor
799: 
800:       auto dispatch__upsample_nearest_exact1d = [](const at::Tensor & input, at::OptionalSymIntArrayRef output_size, ::std::optional<at::ArrayRef<double>> scale_factors) -> at::Tensor {
```

- EN: The main execution path in this span is carried by `_upsample_lanczos2d_aa_symint`, `wrap`, `_upsample_lanczos2d_aa_symint_out`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `_upsample_lanczos2d_aa_symint`, `wrap`, `_upsample_lanczos2d_aa_symint_out` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 801-840

```cpp
801:         pybind11::gil_scoped_release no_gil;
802:         return at::_upsample_nearest_exact1d_symint(input, output_size, scale_factors);
803:       };
804:       return wrap(dispatch__upsample_nearest_exact1d(_r.tensor(0), _r.symintlistOptional(1), _r.doublelistOptional(2)));
805:     }
806:     case 1: {
807:       if (_r.isNone(3)) {
808:         // aten::_upsample_nearest_exact1d(Tensor self, SymInt[1] output_size, float? scales=None) -> Tensor
809: 
810:         auto dispatch__upsample_nearest_exact1d = [](const at::Tensor & self, c10::SymIntArrayRef output_size, ::std::optional<double> scales) -> at::Tensor {
811:           pybind11::gil_scoped_release no_gil;
812:           return at::_upsample_nearest_exact1d_symint(self, output_size, scales);
813:         };
814:         return wrap(dispatch__upsample_nearest_exact1d(_r.tensor(0), _r.symintlist(1), _r.toDoubleOptional(2)));
815:       } else {
816:         // aten::_upsample_nearest_exact1d.out(Tensor self, SymInt[1] output_size, float? scales=None, *, Tensor(a!) out) -> Tensor(a!)
817: 
818:         auto dispatch__upsample_nearest_exact1d_out = [](at::Tensor out, const at::Tensor & self, c10::SymIntArrayRef output_size, ::std::optional<double> scales) -> at::Tensor {
819:           pybind11::gil_scoped_release no_gil;
820:           return at::_upsample_nearest_exact1d_symint_out(out, self, output_size, scales);
821:         };
822:         return wrap(dispatch__upsample_nearest_exact1d_out(_r.tensor(3), _r.tensor(0), _r.symintlist(1), _r.toDoubleOptional(2)));
823:       }
824:     }
825:   }
826:   Py_RETURN_NONE;
827:   END_HANDLE_TH_ERRORS
828: }
829: 
830: \
831: // _upsample_nearest_exact2d
832: static PyObject * THPVariable__upsample_nearest_exact2d(PyObject* self_, PyObject* args, PyObject* kwargs)
833: {
834:   HANDLE_TH_ERRORS
835:   static PythonArgParser parser({
836:     "_upsample_nearest_exact2d(Tensor input, SymIntArrayRef? output_size, ArrayRef<double>? scale_factors)",
837:     "_upsample_nearest_exact2d(Tensor input, SymIntArrayRef[2] output_size, double? scales_h=None, double? scales_w=None, *, Tensor out=None)",
838:   }, /*traceable=*/true);
839: 
840:   ParsedArgs<5> parsed_args;
```

- EN: The main execution path in this span is carried by `_upsample_nearest_exact1d_symint`, `wrap`, `_upsample_nearest_exact1d`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `_upsample_nearest_exact1d_symint`, `wrap`, `_upsample_nearest_exact1d` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 841-880

```cpp
841:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
842:   if(_r.has_torch_function()) {
843:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
844:   }
845:   switch (_r.idx) {
846:     case 0: {
847:       // aten::_upsample_nearest_exact2d.vec(Tensor input, SymInt[]? output_size, float[]? scale_factors) -> Tensor
848: 
849:       auto dispatch__upsample_nearest_exact2d = [](const at::Tensor & input, at::OptionalSymIntArrayRef output_size, ::std::optional<at::ArrayRef<double>> scale_factors) -> at::Tensor {
850:         pybind11::gil_scoped_release no_gil;
851:         return at::_upsample_nearest_exact2d_symint(input, output_size, scale_factors);
852:       };
853:       return wrap(dispatch__upsample_nearest_exact2d(_r.tensor(0), _r.symintlistOptional(1), _r.doublelistOptional(2)));
854:     }
855:     case 1: {
856:       if (_r.isNone(4)) {
857:         // aten::_upsample_nearest_exact2d(Tensor self, SymInt[2] output_size, float? scales_h=None, float? scales_w=None) -> Tensor
858: 
859:         auto dispatch__upsample_nearest_exact2d = [](const at::Tensor & self, c10::SymIntArrayRef output_size, ::std::optional<double> scales_h, ::std::optional<double> scales_w) -> at::Tensor {
860:           pybind11::gil_scoped_release no_gil;
861:           return at::_upsample_nearest_exact2d_symint(self, output_size, scales_h, scales_w);
862:         };
863:         return wrap(dispatch__upsample_nearest_exact2d(_r.tensor(0), _r.symintlist(1), _r.toDoubleOptional(2), _r.toDoubleOptional(3)));
864:       } else {
865:         // aten::_upsample_nearest_exact2d.out(Tensor self, SymInt[2] output_size, float? scales_h=None, float? scales_w=None, *, Tensor(a!) out) -> Tensor(a!)
866: 
867:         auto dispatch__upsample_nearest_exact2d_out = [](at::Tensor out, const at::Tensor & self, c10::SymIntArrayRef output_size, ::std::optional<double> scales_h, ::std::optional<double> scales_w) -> at::Tensor {
868:           pybind11::gil_scoped_release no_gil;
869:           return at::_upsample_nearest_exact2d_symint_out(out, self, output_size, scales_h, scales_w);
870:         };
871:         return wrap(dispatch__upsample_nearest_exact2d_out(_r.tensor(4), _r.tensor(0), _r.symintlist(1), _r.toDoubleOptional(2), _r.toDoubleOptional(3)));
872:       }
873:     }
874:   }
875:   Py_RETURN_NONE;
876:   END_HANDLE_TH_ERRORS
877: }
878: 
879: \
880: // _upsample_nearest_exact3d
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `_upsample_nearest_exact2d_symint`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `_upsample_nearest_exact2d_symint`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 881-920

```cpp
881: static PyObject * THPVariable__upsample_nearest_exact3d(PyObject* self_, PyObject* args, PyObject* kwargs)
882: {
883:   HANDLE_TH_ERRORS
884:   static PythonArgParser parser({
885:     "_upsample_nearest_exact3d(Tensor input, SymIntArrayRef? output_size, ArrayRef<double>? scale_factors)",
886:     "_upsample_nearest_exact3d(Tensor input, SymIntArrayRef[3] output_size, double? scales_d=None, double? scales_h=None, double? scales_w=None, *, Tensor out=None)",
887:   }, /*traceable=*/true);
888: 
889:   ParsedArgs<6> parsed_args;
890:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
891:   if(_r.has_torch_function()) {
892:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
893:   }
894:   switch (_r.idx) {
895:     case 0: {
896:       // aten::_upsample_nearest_exact3d.vec(Tensor input, SymInt[]? output_size, float[]? scale_factors) -> Tensor
897: 
898:       auto dispatch__upsample_nearest_exact3d = [](const at::Tensor & input, at::OptionalSymIntArrayRef output_size, ::std::optional<at::ArrayRef<double>> scale_factors) -> at::Tensor {
899:         pybind11::gil_scoped_release no_gil;
900:         return at::_upsample_nearest_exact3d_symint(input, output_size, scale_factors);
901:       };
902:       return wrap(dispatch__upsample_nearest_exact3d(_r.tensor(0), _r.symintlistOptional(1), _r.doublelistOptional(2)));
903:     }
904:     case 1: {
905:       if (_r.isNone(5)) {
906:         // aten::_upsample_nearest_exact3d(Tensor self, SymInt[3] output_size, float? scales_d=None, float? scales_h=None, float? scales_w=None) -> Tensor
907: 
908:         auto dispatch__upsample_nearest_exact3d = [](const at::Tensor & self, c10::SymIntArrayRef output_size, ::std::optional<double> scales_d, ::std::optional<double> scales_h, ::std::optional<double> scales_w) -> at::Tensor {
909:           pybind11::gil_scoped_release no_gil;
910:           return at::_upsample_nearest_exact3d_symint(self, output_size, scales_d, scales_h, scales_w);
911:         };
912:         return wrap(dispatch__upsample_nearest_exact3d(_r.tensor(0), _r.symintlist(1), _r.toDoubleOptional(2), _r.toDoubleOptional(3), _r.toDoubleOptional(4)));
913:       } else {
914:         // aten::_upsample_nearest_exact3d.out(Tensor self, SymInt[3] output_size, float? scales_d=None, float? scales_h=None, float? scales_w=None, *, Tensor(a!) out) -> Tensor(a!)
915: 
916:         auto dispatch__upsample_nearest_exact3d_out = [](at::Tensor out, const at::Tensor & self, c10::SymIntArrayRef output_size, ::std::optional<double> scales_d, ::std::optional<double> scales_h, ::std::optional<double> scales_w) -> at::Tensor {
917:           pybind11::gil_scoped_release no_gil;
918:           return at::_upsample_nearest_exact3d_symint_out(out, self, output_size, scales_d, scales_h, scales_w);
919:         };
920:         return wrap(dispatch__upsample_nearest_exact3d_out(_r.tensor(5), _r.tensor(0), _r.symintlist(1), _r.toDoubleOptional(2), _r.toDoubleOptional(3), _r.toDoubleOptional(4)));
```

- EN: The main execution path in this span is carried by `THPVariable__upsample_nearest_exact3d`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable__upsample_nearest_exact3d`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 921-960

```cpp
921:       }
922:     }
923:   }
924:   Py_RETURN_NONE;
925:   END_HANDLE_TH_ERRORS
926: }
927: 
928: // adaptive_avg_pool2d
929: static PyObject * THPVariable_adaptive_avg_pool2d(PyObject* self_, PyObject* args, PyObject* kwargs)
930: {
931:   HANDLE_TH_ERRORS
932:   static PythonArgParser parser({
933:     "adaptive_avg_pool2d(Tensor input, SymIntArrayRef[2] output_size, *, Tensor out=None)",
934:   }, /*traceable=*/true);
935: 
936:   ParsedArgs<3> parsed_args;
937:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
938:   if(_r.has_torch_function()) {
939:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
940:   }
941:   if (_r.isNone(2)) {
942:     // aten::adaptive_avg_pool2d(Tensor self, SymInt[2] output_size) -> Tensor
943: 
944:     auto dispatch_adaptive_avg_pool2d = [](const at::Tensor & self, c10::SymIntArrayRef output_size) -> at::Tensor {
945:       pybind11::gil_scoped_release no_gil;
946:       return at::adaptive_avg_pool2d_symint(self, output_size);
947:     };
948:     return wrap(dispatch_adaptive_avg_pool2d(_r.tensor(0), _r.symintlist(1)));
949:   } else {
950:     // aten::adaptive_avg_pool2d.out(Tensor self, SymInt[2] output_size, *, Tensor(a!) out) -> Tensor(a!)
951: 
952:     auto dispatch_adaptive_avg_pool2d_out = [](at::Tensor out, const at::Tensor & self, c10::SymIntArrayRef output_size) -> at::Tensor {
953:       pybind11::gil_scoped_release no_gil;
954:       return at::adaptive_avg_pool2d_symint_out(out, self, output_size);
955:     };
956:     return wrap(dispatch_adaptive_avg_pool2d_out(_r.tensor(2), _r.tensor(0), _r.symintlist(1)));
957:   }
958:   Py_RETURN_NONE;
959:   END_HANDLE_TH_ERRORS
960: }
```

- EN: The main execution path in this span is carried by `THPVariable_adaptive_avg_pool2d`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_adaptive_avg_pool2d`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 961-1000

```cpp
 961: 
 962: // adaptive_avg_pool3d
 963: static PyObject * THPVariable_adaptive_avg_pool3d(PyObject* self_, PyObject* args, PyObject* kwargs)
 964: {
 965:   HANDLE_TH_ERRORS
 966:   static PythonArgParser parser({
 967:     "adaptive_avg_pool3d(Tensor input, SymIntArrayRef[3] output_size, *, Tensor out=None)",
 968:   }, /*traceable=*/true);
 969: 
 970:   ParsedArgs<3> parsed_args;
 971:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
 972:   if(_r.has_torch_function()) {
 973:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
 974:   }
 975:   if (_r.isNone(2)) {
 976:     // aten::adaptive_avg_pool3d(Tensor self, SymInt[3] output_size) -> Tensor
 977: 
 978:     auto dispatch_adaptive_avg_pool3d = [](const at::Tensor & self, c10::SymIntArrayRef output_size) -> at::Tensor {
 979:       pybind11::gil_scoped_release no_gil;
 980:       return at::adaptive_avg_pool3d_symint(self, output_size);
 981:     };
 982:     return wrap(dispatch_adaptive_avg_pool3d(_r.tensor(0), _r.symintlist(1)));
 983:   } else {
 984:     // aten::adaptive_avg_pool3d.out(Tensor self, SymInt[3] output_size, *, Tensor(a!) out) -> Tensor(a!)
 985: 
 986:     auto dispatch_adaptive_avg_pool3d_out = [](at::Tensor out, const at::Tensor & self, c10::SymIntArrayRef output_size) -> at::Tensor {
 987:       pybind11::gil_scoped_release no_gil;
 988:       return at::adaptive_avg_pool3d_symint_out(out, self, output_size);
 989:     };
 990:     return wrap(dispatch_adaptive_avg_pool3d_out(_r.tensor(2), _r.tensor(0), _r.symintlist(1)));
 991:   }
 992:   Py_RETURN_NONE;
 993:   END_HANDLE_TH_ERRORS
 994: }
 995: 
 996: // adaptive_max_pool2d
 997: static PyObject * THPVariable_adaptive_max_pool2d(PyObject* self_, PyObject* args, PyObject* kwargs)
 998: {
 999:   HANDLE_TH_ERRORS
1000:   static PythonArgParser parser({
```

- EN: The main execution path in this span is carried by `THPVariable_adaptive_avg_pool3d`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_adaptive_avg_pool3d`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 1001-1040

```cpp
1001:     "adaptive_max_pool2d(Tensor input, IntArrayRef[2] output_size, *, TensorList[2] out=None)",
1002:   }, /*traceable=*/true);
1003: 
1004:   ParsedArgs<3> parsed_args;
1005:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1006:   if(_r.has_torch_function()) {
1007:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
1008:   }
1009:   if (_r.isNone(2)) {
1010:     // aten::adaptive_max_pool2d(Tensor self, int[2] output_size) -> (Tensor, Tensor)
1011: 
1012:     auto dispatch_adaptive_max_pool2d = [](const at::Tensor & self, at::IntArrayRef output_size) -> ::std::tuple<at::Tensor,at::Tensor> {
1013:       pybind11::gil_scoped_release no_gil;
1014:       return at::adaptive_max_pool2d(self, output_size);
1015:     };
1016:     return wrap(dispatch_adaptive_max_pool2d(_r.tensor(0), _r.intlist(1)));
1017:   } else {
1018:     // aten::adaptive_max_pool2d.out(Tensor self, int[2] output_size, *, Tensor(a!) out, Tensor(b!) indices) -> (Tensor(a!), Tensor(b!))
1019:     auto out = _r.tensorlist_n<2>(2);
1020:     auto dispatch_adaptive_max_pool2d_out = [](at::Tensor & out, at::Tensor & indices, const at::Tensor & self, at::IntArrayRef output_size) -> ::std::tuple<at::Tensor,at::Tensor> {
1021:       pybind11::gil_scoped_release no_gil;
1022:       return at::adaptive_max_pool2d_out(out, indices, self, output_size);
1023:     };
1024:     return wrap(dispatch_adaptive_max_pool2d_out(out[0], out[1], _r.tensor(0), _r.intlist(1)));
1025:   }
1026:   Py_RETURN_NONE;
1027:   END_HANDLE_TH_ERRORS
1028: }
1029: 
1030: // adaptive_max_pool3d
1031: static PyObject * THPVariable_adaptive_max_pool3d(PyObject* self_, PyObject* args, PyObject* kwargs)
1032: {
1033:   HANDLE_TH_ERRORS
1034:   static PythonArgParser parser({
1035:     "adaptive_max_pool3d(Tensor input, IntArrayRef[3] output_size, *, TensorList[2] out=None)",
1036:   }, /*traceable=*/true);
1037: 
1038:   ParsedArgs<3> parsed_args;
1039:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1040:   if(_r.has_torch_function()) {
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `adaptive_max_pool2d`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `adaptive_max_pool2d`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 1041-1080

```cpp
1041:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
1042:   }
1043:   if (_r.isNone(2)) {
1044:     // aten::adaptive_max_pool3d(Tensor self, int[3] output_size) -> (Tensor, Tensor)
1045: 
1046:     auto dispatch_adaptive_max_pool3d = [](const at::Tensor & self, at::IntArrayRef output_size) -> ::std::tuple<at::Tensor,at::Tensor> {
1047:       pybind11::gil_scoped_release no_gil;
1048:       return at::adaptive_max_pool3d(self, output_size);
1049:     };
1050:     return wrap(dispatch_adaptive_max_pool3d(_r.tensor(0), _r.intlist(1)));
1051:   } else {
1052:     // aten::adaptive_max_pool3d.out(Tensor self, int[3] output_size, *, Tensor(a!) out, Tensor(b!) indices) -> (Tensor(a!), Tensor(b!))
1053:     auto out = _r.tensorlist_n<2>(2);
1054:     auto dispatch_adaptive_max_pool3d_out = [](at::Tensor & out, at::Tensor & indices, const at::Tensor & self, at::IntArrayRef output_size) -> ::std::tuple<at::Tensor,at::Tensor> {
1055:       pybind11::gil_scoped_release no_gil;
1056:       return at::adaptive_max_pool3d_out(out, indices, self, output_size);
1057:     };
1058:     return wrap(dispatch_adaptive_max_pool3d_out(out[0], out[1], _r.tensor(0), _r.intlist(1)));
1059:   }
1060:   Py_RETURN_NONE;
1061:   END_HANDLE_TH_ERRORS
1062: }
1063: 
1064: // avg_pool2d
1065: static PyObject * THPVariable_avg_pool2d(PyObject* self_, PyObject* args, PyObject* kwargs)
1066: {
1067:   HANDLE_TH_ERRORS
1068:   static PythonArgParser parser({
1069:     "avg_pool2d(Tensor input, IntArrayRef[2] kernel_size, IntArrayRef[2] stride=None, IntArrayRef[2] padding=0, bool ceil_mode=False, bool count_include_pad=True, int64_t? divisor_override=None, *, Tensor out=None)",
1070:   }, /*traceable=*/true);
1071: 
1072:   ParsedArgs<8> parsed_args;
1073:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1074:   if(_r.has_torch_function()) {
1075:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
1076:   }
1077:   if (_r.isNone(7)) {
1078:     // aten::avg_pool2d(Tensor self, int[2] kernel_size, int[2] stride=[], int[2] padding=0, bool ceil_mode=False, bool count_include_pad=True, int? divisor_override=None) -> Tensor
1079: 
1080:     auto dispatch_avg_pool2d = [](const at::Tensor & self, at::IntArrayRef kernel_size, at::IntArrayRef stride, at::IntArrayRef padding, bool ceil_mode, bool count_include_pad, ::std::optional<int64_t> divisor_override) -> at::Tensor {
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `adaptive_max_pool3d`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `adaptive_max_pool3d`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 1081-1120

```cpp
1081:       pybind11::gil_scoped_release no_gil;
1082:       return at::avg_pool2d(self, kernel_size, stride, padding, ceil_mode, count_include_pad, divisor_override);
1083:     };
1084:     return wrap(dispatch_avg_pool2d(_r.tensor(0), _r.intlist(1), _r.intlist(2), _r.intlist(3), _r.toBool(4), _r.toBool(5), _r.toInt64Optional(6)));
1085:   } else {
1086:     // aten::avg_pool2d.out(Tensor self, int[2] kernel_size, int[2] stride=[], int[2] padding=0, bool ceil_mode=False, bool count_include_pad=True, int? divisor_override=None, *, Tensor(a!) out) -> Tensor(a!)
1087: 
1088:     auto dispatch_avg_pool2d_out = [](at::Tensor out, const at::Tensor & self, at::IntArrayRef kernel_size, at::IntArrayRef stride, at::IntArrayRef padding, bool ceil_mode, bool count_include_pad, ::std::optional<int64_t> divisor_override) -> at::Tensor {
1089:       pybind11::gil_scoped_release no_gil;
1090:       return at::avg_pool2d_out(out, self, kernel_size, stride, padding, ceil_mode, count_include_pad, divisor_override);
1091:     };
1092:     return wrap(dispatch_avg_pool2d_out(_r.tensor(7), _r.tensor(0), _r.intlist(1), _r.intlist(2), _r.intlist(3), _r.toBool(4), _r.toBool(5), _r.toInt64Optional(6)));
1093:   }
1094:   Py_RETURN_NONE;
1095:   END_HANDLE_TH_ERRORS
1096: }
1097: 
1098: // avg_pool3d
1099: static PyObject * THPVariable_avg_pool3d(PyObject* self_, PyObject* args, PyObject* kwargs)
1100: {
1101:   HANDLE_TH_ERRORS
1102:   static PythonArgParser parser({
1103:     "avg_pool3d(Tensor input, IntArrayRef[3] kernel_size, IntArrayRef[3] stride=None, IntArrayRef[3] padding=0, bool ceil_mode=False, bool count_include_pad=True, int64_t? divisor_override=None, *, Tensor out=None)",
1104:   }, /*traceable=*/true);
1105: 
1106:   ParsedArgs<8> parsed_args;
1107:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1108:   if(_r.has_torch_function()) {
1109:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
1110:   }
1111:   if (_r.isNone(7)) {
1112:     // aten::avg_pool3d(Tensor self, int[3] kernel_size, int[3] stride=[], int[3] padding=0, bool ceil_mode=False, bool count_include_pad=True, int? divisor_override=None) -> Tensor
1113: 
1114:     auto dispatch_avg_pool3d = [](const at::Tensor & self, at::IntArrayRef kernel_size, at::IntArrayRef stride, at::IntArrayRef padding, bool ceil_mode, bool count_include_pad, ::std::optional<int64_t> divisor_override) -> at::Tensor {
1115:       pybind11::gil_scoped_release no_gil;
1116:       return at::avg_pool3d(self, kernel_size, stride, padding, ceil_mode, count_include_pad, divisor_override);
1117:     };
1118:     return wrap(dispatch_avg_pool3d(_r.tensor(0), _r.intlist(1), _r.intlist(2), _r.intlist(3), _r.toBool(4), _r.toBool(5), _r.toInt64Optional(6)));
1119:   } else {
1120:     // aten::avg_pool3d.out(Tensor self, int[3] kernel_size, int[3] stride=[], int[3] padding=0, bool ceil_mode=False, bool count_include_pad=True, int? divisor_override=None, *, Tensor(a!) out) -> Tensor(a!)
```

- EN: The main execution path in this span is carried by `avg_pool2d`, `wrap`, `avg_pool2d_out`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `avg_pool2d`, `wrap`, `avg_pool2d_out` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 1121-1160

```cpp
1121: 
1122:     auto dispatch_avg_pool3d_out = [](at::Tensor out, const at::Tensor & self, at::IntArrayRef kernel_size, at::IntArrayRef stride, at::IntArrayRef padding, bool ceil_mode, bool count_include_pad, ::std::optional<int64_t> divisor_override) -> at::Tensor {
1123:       pybind11::gil_scoped_release no_gil;
1124:       return at::avg_pool3d_out(out, self, kernel_size, stride, padding, ceil_mode, count_include_pad, divisor_override);
1125:     };
1126:     return wrap(dispatch_avg_pool3d_out(_r.tensor(7), _r.tensor(0), _r.intlist(1), _r.intlist(2), _r.intlist(3), _r.toBool(4), _r.toBool(5), _r.toInt64Optional(6)));
1127:   }
1128:   Py_RETURN_NONE;
1129:   END_HANDLE_TH_ERRORS
1130: }
1131: 
1132: // binary_cross_entropy
1133: static PyObject * THPVariable_binary_cross_entropy(PyObject* self_, PyObject* args, PyObject* kwargs)
1134: {
1135:   HANDLE_TH_ERRORS
1136:   static PythonArgParser parser({
1137:     "binary_cross_entropy(Tensor input, Tensor target, Tensor? weight=None, int64_t reduction=at::Reduction::Mean, *, Tensor out=None)",
1138:   }, /*traceable=*/true);
1139: 
1140:   ParsedArgs<5> parsed_args;
1141:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1142:   if(_r.has_torch_function()) {
1143:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
1144:   }
1145:   if (_r.isNone(4)) {
1146:     // aten::binary_cross_entropy(Tensor self, Tensor target, Tensor? weight=None, int reduction=Mean) -> Tensor
1147: 
1148:     auto dispatch_binary_cross_entropy = [](const at::Tensor & self, const at::Tensor & target, const ::std::optional<at::Tensor> & weight, int64_t reduction) -> at::Tensor {
1149:       pybind11::gil_scoped_release no_gil;
1150:       return at::binary_cross_entropy(self, target, weight, reduction);
1151:     };
1152:     return wrap(dispatch_binary_cross_entropy(_r.tensor(0), _r.tensor(1), _r.optionalTensor(2), _r.toInt64(3)));
1153:   } else {
1154:     // aten::binary_cross_entropy.out(Tensor self, Tensor target, Tensor? weight=None, int reduction=Mean, *, Tensor(a!) out) -> Tensor(a!)
1155: 
1156:     auto dispatch_binary_cross_entropy_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & target, const ::std::optional<at::Tensor> & weight, int64_t reduction) -> at::Tensor {
1157:       pybind11::gil_scoped_release no_gil;
1158:       return at::binary_cross_entropy_out(out, self, target, weight, reduction);
1159:     };
1160:     return wrap(dispatch_binary_cross_entropy_out(_r.tensor(4), _r.tensor(0), _r.tensor(1), _r.optionalTensor(2), _r.toInt64(3)));
```

- EN: The main execution path in this span is carried by `avg_pool3d_out`, `wrap`, `THPVariable_binary_cross_entropy`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `avg_pool3d_out`, `wrap`, `THPVariable_binary_cross_entropy` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 1161-1200

```cpp
1161:   }
1162:   Py_RETURN_NONE;
1163:   END_HANDLE_TH_ERRORS
1164: }
1165: 
1166: // col2im
1167: static PyObject * THPVariable_col2im(PyObject* self_, PyObject* args, PyObject* kwargs)
1168: {
1169:   HANDLE_TH_ERRORS
1170:   static PythonArgParser parser({
1171:     "col2im(Tensor input, SymIntArrayRef[2] output_size, IntArrayRef[2] kernel_size, IntArrayRef[2] dilation, IntArrayRef[2] padding, IntArrayRef[2] stride, *, Tensor out=None)",
1172:   }, /*traceable=*/true);
1173: 
1174:   ParsedArgs<7> parsed_args;
1175:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1176:   if(_r.has_torch_function()) {
1177:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
1178:   }
1179:   if (_r.isNone(6)) {
1180:     // aten::col2im(Tensor self, SymInt[2] output_size, int[2] kernel_size, int[2] dilation, int[2] padding, int[2] stride) -> Tensor
1181: 
1182:     auto dispatch_col2im = [](const at::Tensor & self, c10::SymIntArrayRef output_size, at::IntArrayRef kernel_size, at::IntArrayRef dilation, at::IntArrayRef padding, at::IntArrayRef stride) -> at::Tensor {
1183:       pybind11::gil_scoped_release no_gil;
1184:       return at::col2im_symint(self, output_size, kernel_size, dilation, padding, stride);
1185:     };
1186:     return wrap(dispatch_col2im(_r.tensor(0), _r.symintlist(1), _r.intlist(2), _r.intlist(3), _r.intlist(4), _r.intlist(5)));
1187:   } else {
1188:     // aten::col2im.out(Tensor self, SymInt[2] output_size, int[2] kernel_size, int[2] dilation, int[2] padding, int[2] stride, *, Tensor(a!) out) -> Tensor(a!)
1189: 
1190:     auto dispatch_col2im_out = [](at::Tensor out, const at::Tensor & self, c10::SymIntArrayRef output_size, at::IntArrayRef kernel_size, at::IntArrayRef dilation, at::IntArrayRef padding, at::IntArrayRef stride) -> at::Tensor {
1191:       pybind11::gil_scoped_release no_gil;
1192:       return at::col2im_symint_out(out, self, output_size, kernel_size, dilation, padding, stride);
1193:     };
1194:     return wrap(dispatch_col2im_out(_r.tensor(6), _r.tensor(0), _r.symintlist(1), _r.intlist(2), _r.intlist(3), _r.intlist(4), _r.intlist(5)));
1195:   }
1196:   Py_RETURN_NONE;
1197:   END_HANDLE_TH_ERRORS
1198: }
1199: 
1200: // conv_depthwise3d
```

- EN: The main execution path in this span is carried by `THPVariable_col2im`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_col2im`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 1201-1240

```cpp
1201: static PyObject * THPVariable_conv_depthwise3d(PyObject* self_, PyObject* args, PyObject* kwargs)
1202: {
1203:   HANDLE_TH_ERRORS
1204:   static PythonArgParser parser({
1205:     "conv_depthwise3d(Tensor input, Tensor weight, SymIntArrayRef[3] kernel_size, Tensor? bias, SymIntArrayRef[3] stride, SymIntArrayRef[3] padding, SymIntArrayRef[3] dilation)",
1206:   }, /*traceable=*/true);
1207: 
1208:   ParsedArgs<7> parsed_args;
1209:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1210:   if(_r.has_torch_function()) {
1211:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
1212:   }
1213:   // aten::conv_depthwise3d(Tensor self, Tensor weight, SymInt[3] kernel_size, Tensor? bias, SymInt[3] stride, SymInt[3] padding, SymInt[3] dilation) -> Tensor
1214: 
1215:   auto dispatch_conv_depthwise3d = [](const at::Tensor & self, const at::Tensor & weight, c10::SymIntArrayRef kernel_size, const ::std::optional<at::Tensor> & bias, c10::SymIntArrayRef stride, c10::SymIntArrayRef padding, c10::SymIntArrayRef dilation) -> at::Tensor {
1216:     pybind11::gil_scoped_release no_gil;
1217:     return at::conv_depthwise3d_symint(self, weight, kernel_size, bias, stride, padding, dilation);
1218:   };
1219:   return wrap(dispatch_conv_depthwise3d(_r.tensor(0), _r.tensor(1), _r.symintlist(2), _r.optionalTensor(3), _r.symintlist(4), _r.symintlist(5), _r.symintlist(6)));
1220:   Py_RETURN_NONE;
1221:   END_HANDLE_TH_ERRORS
1222: }
1223: 
1224: // cross_entropy_loss
1225: static PyObject * THPVariable_cross_entropy_loss(PyObject* self_, PyObject* args, PyObject* kwargs)
1226: {
1227:   HANDLE_TH_ERRORS
1228:   static PythonArgParser parser({
1229:     "cross_entropy_loss(Tensor input, Tensor target, Tensor? weight=None, int64_t reduction=at::Reduction::Mean, SymInt ignore_index=-100, double label_smoothing=0.0)",
1230:   }, /*traceable=*/true);
1231: 
1232:   ParsedArgs<6> parsed_args;
1233:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1234:   if(_r.has_torch_function()) {
1235:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
1236:   }
1237:   // aten::cross_entropy_loss(Tensor self, Tensor target, Tensor? weight=None, int reduction=Mean, SymInt ignore_index=-100, float label_smoothing=0.0) -> Tensor
1238: 
1239:   auto dispatch_cross_entropy_loss = [](const at::Tensor & self, const at::Tensor & target, const ::std::optional<at::Tensor> & weight, int64_t reduction, c10::SymInt ignore_index, double label_smoothing) -> at::Tensor {
1240:     pybind11::gil_scoped_release no_gil;
```

- EN: The main execution path in this span is carried by `THPVariable_conv_depthwise3d`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_conv_depthwise3d`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 1241-1280

```cpp
1241:     return at::cross_entropy_loss_symint(self, target, weight, reduction, ignore_index, label_smoothing);
1242:   };
1243:   return wrap(dispatch_cross_entropy_loss(_r.tensor(0), _r.tensor(1), _r.optionalTensor(2), _r.toInt64(3), _r.toSymInt(4), _r.toDouble(5)));
1244:   Py_RETURN_NONE;
1245:   END_HANDLE_TH_ERRORS
1246: }
1247: 
1248: // elu
1249: static PyObject * THPVariable_elu(PyObject* self_, PyObject* args, PyObject* kwargs)
1250: {
1251:   HANDLE_TH_ERRORS
1252:   static PythonArgParser parser({
1253:     "elu(Tensor input, Scalar alpha=1, Scalar scale=1, Scalar input_scale=1, *, Tensor out=None)",
1254:   }, /*traceable=*/true);
1255: 
1256:   ParsedArgs<5> parsed_args;
1257:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1258:   if(_r.has_torch_function()) {
1259:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
1260:   }
1261:   if (_r.isNone(4)) {
1262:     // aten::elu(Tensor self, Scalar alpha=1, Scalar scale=1, Scalar input_scale=1) -> Tensor
1263: 
1264:     auto dispatch_elu = [](const at::Tensor & self, const at::Scalar & alpha, const at::Scalar & scale, const at::Scalar & input_scale) -> at::Tensor {
1265:       pybind11::gil_scoped_release no_gil;
1266:       return at::elu(self, alpha, scale, input_scale);
1267:     };
1268:     return wrap(dispatch_elu(_r.tensor(0), _r.scalar(1), _r.scalar(2), _r.scalar(3)));
1269:   } else {
1270:     // aten::elu.out(Tensor self, Scalar alpha=1, Scalar scale=1, Scalar input_scale=1, *, Tensor(a!) out) -> Tensor(a!)
1271: 
1272:     auto dispatch_elu_out = [](at::Tensor out, const at::Tensor & self, const at::Scalar & alpha, const at::Scalar & scale, const at::Scalar & input_scale) -> at::Tensor {
1273:       pybind11::gil_scoped_release no_gil;
1274:       return at::elu_out(out, self, alpha, scale, input_scale);
1275:     };
1276:     return wrap(dispatch_elu_out(_r.tensor(4), _r.tensor(0), _r.scalar(1), _r.scalar(2), _r.scalar(3)));
1277:   }
1278:   Py_RETURN_NONE;
1279:   END_HANDLE_TH_ERRORS
1280: }
```

- EN: The main execution path in this span is carried by `cross_entropy_loss_symint`, `wrap`, `THPVariable_elu`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `cross_entropy_loss_symint`, `wrap`, `THPVariable_elu` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1281-1320

```cpp
1281: 
1282: // elu_
1283: static PyObject * THPVariable_elu_(PyObject* self_, PyObject* args, PyObject* kwargs)
1284: {
1285:   HANDLE_TH_ERRORS
1286:   static PythonArgParser parser({
1287:     "elu_(Tensor input, Scalar alpha=1, Scalar scale=1, Scalar input_scale=1)",
1288:   }, /*traceable=*/true);
1289: 
1290:   ParsedArgs<4> parsed_args;
1291:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1292:   if(_r.has_torch_function()) {
1293:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
1294:   }
1295:   // aten::elu_(Tensor(a!) self, Scalar alpha=1, Scalar scale=1, Scalar input_scale=1) -> Tensor(a!)
1296: 
1297:   auto dispatch_elu_ = [](at::Tensor self, const at::Scalar & alpha, const at::Scalar & scale, const at::Scalar & input_scale) -> at::Tensor {
1298:     pybind11::gil_scoped_release no_gil;
1299:     return at::elu_(self, alpha, scale, input_scale);
1300:   };
1301:   return wrap(dispatch_elu_(_r.tensor(0), _r.scalar(1), _r.scalar(2), _r.scalar(3)));
1302:   Py_RETURN_NONE;
1303:   END_HANDLE_TH_ERRORS
1304: }
1305: 
1306: // flatten_dense_tensors
1307: static PyObject * THPVariable_flatten_dense_tensors(PyObject* self_, PyObject* args, PyObject* kwargs)
1308: {
1309:   HANDLE_TH_ERRORS
1310:   static PythonArgParser parser({
1311:     "flatten_dense_tensors(TensorList tensors)",
1312:   }, /*traceable=*/true);
1313: 
1314:   ParsedArgs<1> parsed_args;
1315:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1316:   if(_r.has_torch_function()) {
1317:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
1318:   }
1319:   // aten::flatten_dense_tensors(Tensor[] tensors) -> Tensor
1320: 
```

- EN: The main execution path in this span is carried by `THPVariable_elu_`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_elu_`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1321-1360

```cpp
1321:   auto dispatch_flatten_dense_tensors = [](at::TensorList tensors) -> at::Tensor {
1322:     pybind11::gil_scoped_release no_gil;
1323:     return at::flatten_dense_tensors(tensors);
1324:   };
1325:   return wrap(dispatch_flatten_dense_tensors(_r.tensorlist(0)));
1326:   Py_RETURN_NONE;
1327:   END_HANDLE_TH_ERRORS
1328: }
1329: 
1330: // fractional_max_pool2d
1331: static PyObject * THPVariable_fractional_max_pool2d(PyObject* self_, PyObject* args, PyObject* kwargs)
1332: {
1333:   HANDLE_TH_ERRORS
1334:   static PythonArgParser parser({
1335:     "fractional_max_pool2d(Tensor input, IntArrayRef[2] kernel_size, IntArrayRef[2] output_size, Tensor random_samples, *, TensorList[2] out=None)",
1336:   }, /*traceable=*/true);
1337: 
1338:   ParsedArgs<5> parsed_args;
1339:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1340:   if(_r.has_torch_function()) {
1341:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
1342:   }
1343:   if (_r.isNone(4)) {
1344:     // aten::fractional_max_pool2d(Tensor self, int[2] kernel_size, int[2] output_size, Tensor random_samples) -> (Tensor, Tensor)
1345: 
1346:     auto dispatch_fractional_max_pool2d = [](const at::Tensor & self, at::IntArrayRef kernel_size, at::IntArrayRef output_size, const at::Tensor & random_samples) -> ::std::tuple<at::Tensor,at::Tensor> {
1347:       pybind11::gil_scoped_release no_gil;
1348:       return at::fractional_max_pool2d(self, kernel_size, output_size, random_samples);
1349:     };
1350:     return wrap(dispatch_fractional_max_pool2d(_r.tensor(0), _r.intlist(1), _r.intlist(2), _r.tensor(3)));
1351:   } else {
1352:     // aten::fractional_max_pool2d.output(Tensor self, int[2] kernel_size, int[2] output_size, Tensor random_samples, *, Tensor(a!) output, Tensor(b!) indices) -> (Tensor(a!), Tensor(b!))
1353:     auto out = _r.tensorlist_n<2>(4);
1354:     auto dispatch_fractional_max_pool2d_out = [](at::Tensor & output, at::Tensor & indices, const at::Tensor & self, at::IntArrayRef kernel_size, at::IntArrayRef output_size, const at::Tensor & random_samples) -> ::std::tuple<at::Tensor,at::Tensor> {
1355:       pybind11::gil_scoped_release no_gil;
1356:       return at::fractional_max_pool2d_out(output, indices, self, kernel_size, output_size, random_samples);
1357:     };
1358:     return wrap(dispatch_fractional_max_pool2d_out(out[0], out[1], _r.tensor(0), _r.intlist(1), _r.intlist(2), _r.tensor(3)));
1359:   }
1360:   Py_RETURN_NONE;
```

- EN: The main execution path in this span is carried by `flatten_dense_tensors`, `wrap`, `THPVariable_fractional_max_pool2d`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `flatten_dense_tensors`, `wrap`, `THPVariable_fractional_max_pool2d` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 1361-1400

```cpp
1361:   END_HANDLE_TH_ERRORS
1362: }
1363: 
1364: // fractional_max_pool3d
1365: static PyObject * THPVariable_fractional_max_pool3d(PyObject* self_, PyObject* args, PyObject* kwargs)
1366: {
1367:   HANDLE_TH_ERRORS
1368:   static PythonArgParser parser({
1369:     "fractional_max_pool3d(Tensor input, IntArrayRef[3] kernel_size, IntArrayRef[3] output_size, Tensor random_samples, *, TensorList[2] out=None)",
1370:   }, /*traceable=*/true);
1371: 
1372:   ParsedArgs<5> parsed_args;
1373:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1374:   if(_r.has_torch_function()) {
1375:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
1376:   }
1377:   if (_r.isNone(4)) {
1378:     // aten::fractional_max_pool3d(Tensor self, int[3] kernel_size, int[3] output_size, Tensor random_samples) -> (Tensor, Tensor)
1379: 
1380:     auto dispatch_fractional_max_pool3d = [](const at::Tensor & self, at::IntArrayRef kernel_size, at::IntArrayRef output_size, const at::Tensor & random_samples) -> ::std::tuple<at::Tensor,at::Tensor> {
1381:       pybind11::gil_scoped_release no_gil;
1382:       return at::fractional_max_pool3d(self, kernel_size, output_size, random_samples);
1383:     };
1384:     return wrap(dispatch_fractional_max_pool3d(_r.tensor(0), _r.intlist(1), _r.intlist(2), _r.tensor(3)));
1385:   } else {
1386:     // aten::fractional_max_pool3d.output(Tensor self, int[3] kernel_size, int[3] output_size, Tensor random_samples, *, Tensor(a!) output, Tensor(b!) indices) -> (Tensor(a!), Tensor(b!))
1387:     auto out = _r.tensorlist_n<2>(4);
1388:     auto dispatch_fractional_max_pool3d_out = [](at::Tensor & output, at::Tensor & indices, const at::Tensor & self, at::IntArrayRef kernel_size, at::IntArrayRef output_size, const at::Tensor & random_samples) -> ::std::tuple<at::Tensor,at::Tensor> {
1389:       pybind11::gil_scoped_release no_gil;
1390:       return at::fractional_max_pool3d_out(output, indices, self, kernel_size, output_size, random_samples);
1391:     };
1392:     return wrap(dispatch_fractional_max_pool3d_out(out[0], out[1], _r.tensor(0), _r.intlist(1), _r.intlist(2), _r.tensor(3)));
1393:   }
1394:   Py_RETURN_NONE;
1395:   END_HANDLE_TH_ERRORS
1396: }
1397: 
1398: // gelu
1399: static PyObject * THPVariable_gelu(PyObject* self_, PyObject* args, PyObject* kwargs)
1400: {
```

- EN: The main execution path in this span is carried by `THPVariable_fractional_max_pool3d`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_fractional_max_pool3d`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 1401-1440

```cpp
1401:   HANDLE_TH_ERRORS
1402:   static PythonArgParser parser({
1403:     "gelu(Tensor input, *, c10::string_view approximate=\"none\", Tensor out=None)",
1404:   }, /*traceable=*/true);
1405: 
1406:   ParsedArgs<3> parsed_args;
1407:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1408:   if(_r.has_torch_function()) {
1409:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
1410:   }
1411:   if (_r.isNone(2)) {
1412:     // aten::gelu(Tensor self, *, str approximate='none') -> Tensor
1413: 
1414:     auto dispatch_gelu = [](const at::Tensor & self, c10::string_view approximate) -> at::Tensor {
1415:       pybind11::gil_scoped_release no_gil;
1416:       return at::gelu(self, approximate);
1417:     };
1418:     return wrap(dispatch_gelu(_r.tensor(0), _r.stringView(1)));
1419:   } else {
1420:     // aten::gelu.out(Tensor self, *, str approximate='none', Tensor(a!) out) -> Tensor(a!)
1421: 
1422:     auto dispatch_gelu_out = [](at::Tensor out, const at::Tensor & self, c10::string_view approximate) -> at::Tensor {
1423:       pybind11::gil_scoped_release no_gil;
1424:       return at::gelu_out(out, self, approximate);
1425:     };
1426:     return wrap(dispatch_gelu_out(_r.tensor(2), _r.tensor(0), _r.stringView(1)));
1427:   }
1428:   Py_RETURN_NONE;
1429:   END_HANDLE_TH_ERRORS
1430: }
1431: 
1432: // gelu_
1433: static PyObject * THPVariable_gelu_(PyObject* self_, PyObject* args, PyObject* kwargs)
1434: {
1435:   HANDLE_TH_ERRORS
1436:   static PythonArgParser parser({
1437:     "gelu_(Tensor input, *, c10::string_view approximate=\"none\")",
1438:   }, /*traceable=*/true);
1439: 
1440:   ParsedArgs<2> parsed_args;
```

- EN: The main execution path in this span is carried by `parser`, `handle_torch_function`, `gelu`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `parser`, `handle_torch_function`, `gelu` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1441-1480

```cpp
1441:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1442:   if(_r.has_torch_function()) {
1443:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
1444:   }
1445:   // aten::gelu_(Tensor(a!) self, *, str approximate='none') -> Tensor(a!)
1446: 
1447:   auto dispatch_gelu_ = [](at::Tensor self, c10::string_view approximate) -> at::Tensor {
1448:     pybind11::gil_scoped_release no_gil;
1449:     return at::gelu_(self, approximate);
1450:   };
1451:   return wrap(dispatch_gelu_(_r.tensor(0), _r.stringView(1)));
1452:   Py_RETURN_NONE;
1453:   END_HANDLE_TH_ERRORS
1454: }
1455: 
1456: // glu
1457: static PyObject * THPVariable_glu(PyObject* self_, PyObject* args, PyObject* kwargs)
1458: {
1459:   HANDLE_TH_ERRORS
1460:   static PythonArgParser parser({
1461:     "glu(Tensor input, int64_t dim=-1, *, Tensor out=None)",
1462:   }, /*traceable=*/true);
1463: 
1464:   ParsedArgs<3> parsed_args;
1465:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1466:   if(_r.has_torch_function()) {
1467:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
1468:   }
1469:   if (_r.isNone(2)) {
1470:     // aten::glu(Tensor self, int dim=-1) -> Tensor
1471: 
1472:     auto dispatch_glu = [](const at::Tensor & self, int64_t dim) -> at::Tensor {
1473:       pybind11::gil_scoped_release no_gil;
1474:       return at::glu(self, dim);
1475:     };
1476:     return wrap(dispatch_glu(_r.tensor(0), _r.toInt64(1)));
1477:   } else {
1478:     // aten::glu.out(Tensor self, int dim=-1, *, Tensor(a!) out) -> Tensor(a!)
1479: 
1480:     auto dispatch_glu_out = [](at::Tensor out, const at::Tensor & self, int64_t dim) -> at::Tensor {
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `gelu_`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `gelu_`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1481-1520

```cpp
1481:       pybind11::gil_scoped_release no_gil;
1482:       return at::glu_out(out, self, dim);
1483:     };
1484:     return wrap(dispatch_glu_out(_r.tensor(2), _r.tensor(0), _r.toInt64(1)));
1485:   }
1486:   Py_RETURN_NONE;
1487:   END_HANDLE_TH_ERRORS
1488: }
1489: 
1490: // hardsigmoid
1491: static PyObject * THPVariable_hardsigmoid(PyObject* self_, PyObject* args, PyObject* kwargs)
1492: {
1493:   HANDLE_TH_ERRORS
1494:   static PythonArgParser parser({
1495:     "hardsigmoid(Tensor input, *, Tensor out=None)",
1496:   }, /*traceable=*/true);
1497: 
1498:   ParsedArgs<2> parsed_args;
1499:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1500:   if(_r.has_torch_function()) {
1501:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
1502:   }
1503:   if (_r.isNone(1)) {
1504:     // aten::hardsigmoid(Tensor self) -> Tensor
1505: 
1506:     auto dispatch_hardsigmoid = [](const at::Tensor & self) -> at::Tensor {
1507:       pybind11::gil_scoped_release no_gil;
1508:       return at::hardsigmoid(self);
1509:     };
1510:     return wrap(dispatch_hardsigmoid(_r.tensor(0)));
1511:   } else {
1512:     // aten::hardsigmoid.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
1513: 
1514:     auto dispatch_hardsigmoid_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
1515:       pybind11::gil_scoped_release no_gil;
1516:       return at::hardsigmoid_out(out, self);
1517:     };
1518:     return wrap(dispatch_hardsigmoid_out(_r.tensor(1), _r.tensor(0)));
1519:   }
1520:   Py_RETURN_NONE;
```

- EN: The main execution path in this span is carried by `glu_out`, `wrap`, `THPVariable_hardsigmoid`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `glu_out`, `wrap`, `THPVariable_hardsigmoid` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1521-1560

```cpp
1521:   END_HANDLE_TH_ERRORS
1522: }
1523: 
1524: // hardsigmoid_
1525: static PyObject * THPVariable_hardsigmoid_(PyObject* self_, PyObject* args, PyObject* kwargs)
1526: {
1527:   HANDLE_TH_ERRORS
1528:   static PythonArgParser parser({
1529:     "hardsigmoid_(Tensor input)",
1530:   }, /*traceable=*/true);
1531: 
1532:   ParsedArgs<1> parsed_args;
1533:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1534:   if(_r.has_torch_function()) {
1535:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
1536:   }
1537:   // aten::hardsigmoid_(Tensor(a!) self) -> Tensor(a!)
1538: 
1539:   auto dispatch_hardsigmoid_ = [](at::Tensor self) -> at::Tensor {
1540:     pybind11::gil_scoped_release no_gil;
1541:     return at::hardsigmoid_(self);
1542:   };
1543:   return wrap(dispatch_hardsigmoid_(_r.tensor(0)));
1544:   Py_RETURN_NONE;
1545:   END_HANDLE_TH_ERRORS
1546: }
1547: 
1548: // hardswish
1549: static PyObject * THPVariable_hardswish(PyObject* self_, PyObject* args, PyObject* kwargs)
1550: {
1551:   HANDLE_TH_ERRORS
1552:   static PythonArgParser parser({
1553:     "hardswish(Tensor input, *, Tensor out=None)",
1554:   }, /*traceable=*/true);
1555: 
1556:   ParsedArgs<2> parsed_args;
1557:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1558:   if(_r.has_torch_function()) {
1559:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
1560:   }
```

- EN: The main execution path in this span is carried by `THPVariable_hardsigmoid_`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_hardsigmoid_`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1561-1600

```cpp
1561:   if (_r.isNone(1)) {
1562:     // aten::hardswish(Tensor self) -> Tensor
1563: 
1564:     auto dispatch_hardswish = [](const at::Tensor & self) -> at::Tensor {
1565:       pybind11::gil_scoped_release no_gil;
1566:       return at::hardswish(self);
1567:     };
1568:     return wrap(dispatch_hardswish(_r.tensor(0)));
1569:   } else {
1570:     // aten::hardswish.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
1571: 
1572:     auto dispatch_hardswish_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
1573:       pybind11::gil_scoped_release no_gil;
1574:       return at::hardswish_out(out, self);
1575:     };
1576:     return wrap(dispatch_hardswish_out(_r.tensor(1), _r.tensor(0)));
1577:   }
1578:   Py_RETURN_NONE;
1579:   END_HANDLE_TH_ERRORS
1580: }
1581: 
1582: // hardswish_
1583: static PyObject * THPVariable_hardswish_(PyObject* self_, PyObject* args, PyObject* kwargs)
1584: {
1585:   HANDLE_TH_ERRORS
1586:   static PythonArgParser parser({
1587:     "hardswish_(Tensor input)",
1588:   }, /*traceable=*/true);
1589: 
1590:   ParsedArgs<1> parsed_args;
1591:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1592:   if(_r.has_torch_function()) {
1593:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
1594:   }
1595:   // aten::hardswish_(Tensor(a!) self) -> Tensor(a!)
1596: 
1597:   auto dispatch_hardswish_ = [](at::Tensor self) -> at::Tensor {
1598:     pybind11::gil_scoped_release no_gil;
1599:     return at::hardswish_(self);
1600:   };
```

- EN: The main execution path in this span is carried by `hardswish`, `wrap`, `hardswish_out`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `hardswish`, `wrap`, `hardswish_out` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1601-1640

```cpp
1601:   return wrap(dispatch_hardswish_(_r.tensor(0)));
1602:   Py_RETURN_NONE;
1603:   END_HANDLE_TH_ERRORS
1604: }
1605: 
1606: // hardtanh
1607: static PyObject * THPVariable_hardtanh(PyObject* self_, PyObject* args, PyObject* kwargs)
1608: {
1609:   HANDLE_TH_ERRORS
1610:   static PythonArgParser parser({
1611:     "hardtanh(Tensor input, Scalar min_val=-1, Scalar max_val=1, *, Tensor out=None)",
1612:   }, /*traceable=*/true);
1613: 
1614:   ParsedArgs<4> parsed_args;
1615:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1616:   if(_r.has_torch_function()) {
1617:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
1618:   }
1619:   if (_r.isNone(3)) {
1620:     // aten::hardtanh(Tensor self, Scalar min_val=-1, Scalar max_val=1) -> Tensor
1621: 
1622:     auto dispatch_hardtanh = [](const at::Tensor & self, const at::Scalar & min_val, const at::Scalar & max_val) -> at::Tensor {
1623:       pybind11::gil_scoped_release no_gil;
1624:       return at::hardtanh(self, min_val, max_val);
1625:     };
1626:     return wrap(dispatch_hardtanh(_r.tensor(0), _r.scalar(1), _r.scalar(2)));
1627:   } else {
1628:     // aten::hardtanh.out(Tensor self, Scalar min_val=-1, Scalar max_val=1, *, Tensor(a!) out) -> Tensor(a!)
1629: 
1630:     auto dispatch_hardtanh_out = [](at::Tensor out, const at::Tensor & self, const at::Scalar & min_val, const at::Scalar & max_val) -> at::Tensor {
1631:       pybind11::gil_scoped_release no_gil;
1632:       return at::hardtanh_out(out, self, min_val, max_val);
1633:     };
1634:     return wrap(dispatch_hardtanh_out(_r.tensor(3), _r.tensor(0), _r.scalar(1), _r.scalar(2)));
1635:   }
1636:   Py_RETURN_NONE;
1637:   END_HANDLE_TH_ERRORS
1638: }
1639: 
1640: // hardtanh_
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_hardtanh`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_hardtanh`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1641-1680

```cpp
1641: static PyObject * THPVariable_hardtanh_(PyObject* self_, PyObject* args, PyObject* kwargs)
1642: {
1643:   HANDLE_TH_ERRORS
1644:   static PythonArgParser parser({
1645:     "hardtanh_(Tensor input, Scalar min_val=-1, Scalar max_val=1)",
1646:   }, /*traceable=*/true);
1647: 
1648:   ParsedArgs<3> parsed_args;
1649:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1650:   if(_r.has_torch_function()) {
1651:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
1652:   }
1653:   // aten::hardtanh_(Tensor(a!) self, Scalar min_val=-1, Scalar max_val=1) -> Tensor(a!)
1654: 
1655:   auto dispatch_hardtanh_ = [](at::Tensor self, const at::Scalar & min_val, const at::Scalar & max_val) -> at::Tensor {
1656:     pybind11::gil_scoped_release no_gil;
1657:     return at::hardtanh_(self, min_val, max_val);
1658:   };
1659:   return wrap(dispatch_hardtanh_(_r.tensor(0), _r.scalar(1), _r.scalar(2)));
1660:   Py_RETURN_NONE;
1661:   END_HANDLE_TH_ERRORS
1662: }
1663: 
1664: // huber_loss
1665: static PyObject * THPVariable_huber_loss(PyObject* self_, PyObject* args, PyObject* kwargs)
1666: {
1667:   HANDLE_TH_ERRORS
1668:   static PythonArgParser parser({
1669:     "huber_loss(Tensor input, Tensor target, int64_t reduction=at::Reduction::Mean, double delta=1.0, *, Tensor out=None)",
1670:   }, /*traceable=*/true);
1671: 
1672:   ParsedArgs<5> parsed_args;
1673:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1674:   if(_r.has_torch_function()) {
1675:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
1676:   }
1677:   if (_r.isNone(4)) {
1678:     // aten::huber_loss(Tensor self, Tensor target, int reduction=Mean, float delta=1.0) -> Tensor
1679: 
1680:     auto dispatch_huber_loss = [](const at::Tensor & self, const at::Tensor & target, int64_t reduction, double delta) -> at::Tensor {
```

- EN: The main execution path in this span is carried by `THPVariable_hardtanh_`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_hardtanh_`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1681-1720

```cpp
1681:       pybind11::gil_scoped_release no_gil;
1682:       return at::huber_loss(self, target, reduction, delta);
1683:     };
1684:     return wrap(dispatch_huber_loss(_r.tensor(0), _r.tensor(1), _r.toInt64(2), _r.toDouble(3)));
1685:   } else {
1686:     // aten::huber_loss.out(Tensor self, Tensor target, int reduction=Mean, float delta=1.0, *, Tensor(a!) out) -> Tensor(a!)
1687: 
1688:     auto dispatch_huber_loss_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & target, int64_t reduction, double delta) -> at::Tensor {
1689:       pybind11::gil_scoped_release no_gil;
1690:       return at::huber_loss_out(out, self, target, reduction, delta);
1691:     };
1692:     return wrap(dispatch_huber_loss_out(_r.tensor(4), _r.tensor(0), _r.tensor(1), _r.toInt64(2), _r.toDouble(3)));
1693:   }
1694:   Py_RETURN_NONE;
1695:   END_HANDLE_TH_ERRORS
1696: }
1697: 
1698: // im2col
1699: static PyObject * THPVariable_im2col(PyObject* self_, PyObject* args, PyObject* kwargs)
1700: {
1701:   HANDLE_TH_ERRORS
1702:   static PythonArgParser parser({
1703:     "im2col(Tensor input, IntArrayRef[2] kernel_size, IntArrayRef[2] dilation, IntArrayRef[2] padding, IntArrayRef[2] stride, *, Tensor out=None)",
1704:   }, /*traceable=*/true);
1705: 
1706:   ParsedArgs<6> parsed_args;
1707:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1708:   if(_r.has_torch_function()) {
1709:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
1710:   }
1711:   if (_r.isNone(5)) {
1712:     // aten::im2col(Tensor self, int[2] kernel_size, int[2] dilation, int[2] padding, int[2] stride) -> Tensor
1713: 
1714:     auto dispatch_im2col = [](const at::Tensor & self, at::IntArrayRef kernel_size, at::IntArrayRef dilation, at::IntArrayRef padding, at::IntArrayRef stride) -> at::Tensor {
1715:       pybind11::gil_scoped_release no_gil;
1716:       return at::im2col(self, kernel_size, dilation, padding, stride);
1717:     };
1718:     return wrap(dispatch_im2col(_r.tensor(0), _r.intlist(1), _r.intlist(2), _r.intlist(3), _r.intlist(4)));
1719:   } else {
1720:     // aten::im2col.out(Tensor self, int[2] kernel_size, int[2] dilation, int[2] padding, int[2] stride, *, Tensor(a!) out) -> Tensor(a!)
```

- EN: The main execution path in this span is carried by `huber_loss`, `wrap`, `huber_loss_out`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `huber_loss`, `wrap`, `huber_loss_out` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 1721-1760

```cpp
1721: 
1722:     auto dispatch_im2col_out = [](at::Tensor out, const at::Tensor & self, at::IntArrayRef kernel_size, at::IntArrayRef dilation, at::IntArrayRef padding, at::IntArrayRef stride) -> at::Tensor {
1723:       pybind11::gil_scoped_release no_gil;
1724:       return at::im2col_out(out, self, kernel_size, dilation, padding, stride);
1725:     };
1726:     return wrap(dispatch_im2col_out(_r.tensor(5), _r.tensor(0), _r.intlist(1), _r.intlist(2), _r.intlist(3), _r.intlist(4)));
1727:   }
1728:   Py_RETURN_NONE;
1729:   END_HANDLE_TH_ERRORS
1730: }
1731: 
1732: // l1_loss
1733: static PyObject * THPVariable_l1_loss(PyObject* self_, PyObject* args, PyObject* kwargs)
1734: {
1735:   HANDLE_TH_ERRORS
1736:   static PythonArgParser parser({
1737:     "l1_loss(Tensor input, Tensor target, int64_t reduction=at::Reduction::Mean)",
1738:   }, /*traceable=*/true);
1739: 
1740:   ParsedArgs<3> parsed_args;
1741:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1742:   if(_r.has_torch_function()) {
1743:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
1744:   }
1745:   // aten::l1_loss(Tensor self, Tensor target, int reduction=Mean) -> Tensor
1746: 
1747:   auto dispatch_l1_loss = [](const at::Tensor & self, const at::Tensor & target, int64_t reduction) -> at::Tensor {
1748:     pybind11::gil_scoped_release no_gil;
1749:     return at::l1_loss(self, target, reduction);
1750:   };
1751:   return wrap(dispatch_l1_loss(_r.tensor(0), _r.tensor(1), _r.toInt64(2)));
1752:   Py_RETURN_NONE;
1753:   END_HANDLE_TH_ERRORS
1754: }
1755: 
1756: // leaky_relu
1757: static PyObject * THPVariable_leaky_relu(PyObject* self_, PyObject* args, PyObject* kwargs)
1758: {
1759:   HANDLE_TH_ERRORS
1760:   static PythonArgParser parser({
```

- EN: The main execution path in this span is carried by `im2col_out`, `wrap`, `THPVariable_l1_loss`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `im2col_out`, `wrap`, `THPVariable_l1_loss` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 1761-1800

```cpp
1761:     "leaky_relu(Tensor input, Scalar negative_slope=0.01, *, Tensor out=None)",
1762:   }, /*traceable=*/true);
1763: 
1764:   ParsedArgs<3> parsed_args;
1765:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1766:   if(_r.has_torch_function()) {
1767:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
1768:   }
1769:   if (_r.isNone(2)) {
1770:     // aten::leaky_relu(Tensor self, Scalar negative_slope=0.01) -> Tensor
1771: 
1772:     auto dispatch_leaky_relu = [](const at::Tensor & self, const at::Scalar & negative_slope) -> at::Tensor {
1773:       pybind11::gil_scoped_release no_gil;
1774:       return at::leaky_relu(self, negative_slope);
1775:     };
1776:     return wrap(dispatch_leaky_relu(_r.tensor(0), _r.scalar(1)));
1777:   } else {
1778:     // aten::leaky_relu.out(Tensor self, Scalar negative_slope=0.01, *, Tensor(a!) out) -> Tensor(a!)
1779: 
1780:     auto dispatch_leaky_relu_out = [](at::Tensor out, const at::Tensor & self, const at::Scalar & negative_slope) -> at::Tensor {
1781:       pybind11::gil_scoped_release no_gil;
1782:       return at::leaky_relu_out(out, self, negative_slope);
1783:     };
1784:     return wrap(dispatch_leaky_relu_out(_r.tensor(2), _r.tensor(0), _r.scalar(1)));
1785:   }
1786:   Py_RETURN_NONE;
1787:   END_HANDLE_TH_ERRORS
1788: }
1789: 
1790: // leaky_relu_
1791: static PyObject * THPVariable_leaky_relu_(PyObject* self_, PyObject* args, PyObject* kwargs)
1792: {
1793:   HANDLE_TH_ERRORS
1794:   static PythonArgParser parser({
1795:     "leaky_relu_(Tensor input, Scalar negative_slope=0.01)",
1796:   }, /*traceable=*/true);
1797: 
1798:   ParsedArgs<2> parsed_args;
1799:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1800:   if(_r.has_torch_function()) {
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `leaky_relu`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `leaky_relu`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1801-1840

```cpp
1801:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
1802:   }
1803:   // aten::leaky_relu_(Tensor(a!) self, Scalar negative_slope=0.01) -> Tensor(a!)
1804: 
1805:   auto dispatch_leaky_relu_ = [](at::Tensor self, const at::Scalar & negative_slope) -> at::Tensor {
1806:     pybind11::gil_scoped_release no_gil;
1807:     return at::leaky_relu_(self, negative_slope);
1808:   };
1809:   return wrap(dispatch_leaky_relu_(_r.tensor(0), _r.scalar(1)));
1810:   Py_RETURN_NONE;
1811:   END_HANDLE_TH_ERRORS
1812: }
1813: 
1814: // linear
1815: static PyObject * THPVariable_linear(PyObject* self_, PyObject* args, PyObject* kwargs)
1816: {
1817:   HANDLE_TH_ERRORS
1818:   static PythonArgParser parser({
1819:     "linear(Tensor input, Tensor weight, Tensor? bias=None, *, Tensor out=None)",
1820:   }, /*traceable=*/true);
1821: 
1822:   ParsedArgs<4> parsed_args;
1823:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1824:   if(_r.has_torch_function()) {
1825:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
1826:   }
1827:   if (_r.isNone(3)) {
1828:     // aten::linear(Tensor input, Tensor weight, Tensor? bias=None) -> Tensor
1829: 
1830:     auto dispatch_linear = [](const at::Tensor & input, const at::Tensor & weight, const ::std::optional<at::Tensor> & bias) -> at::Tensor {
1831:       pybind11::gil_scoped_release no_gil;
1832:       return at::linear(input, weight, bias);
1833:     };
1834:     return wrap(dispatch_linear(_r.tensor(0), _r.tensor(1), _r.optionalTensor(2)));
1835:   } else {
1836:     // aten::linear.out(Tensor input, Tensor weight, Tensor? bias=None, *, Tensor(a!) out) -> Tensor(a!)
1837: 
1838:     auto dispatch_linear_out = [](at::Tensor out, const at::Tensor & input, const at::Tensor & weight, const ::std::optional<at::Tensor> & bias) -> at::Tensor {
1839:       pybind11::gil_scoped_release no_gil;
1840:       return at::linear_out(out, input, weight, bias);
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `leaky_relu_`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `leaky_relu_`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1841-1880

```cpp
1841:     };
1842:     return wrap(dispatch_linear_out(_r.tensor(3), _r.tensor(0), _r.tensor(1), _r.optionalTensor(2)));
1843:   }
1844:   Py_RETURN_NONE;
1845:   END_HANDLE_TH_ERRORS
1846: }
1847: 
1848: // log_sigmoid
1849: static PyObject * THPVariable_log_sigmoid(PyObject* self_, PyObject* args, PyObject* kwargs)
1850: {
1851:   HANDLE_TH_ERRORS
1852:   static PythonArgParser parser({
1853:     "log_sigmoid(Tensor input, *, Tensor out=None)",
1854:   }, /*traceable=*/true);
1855: 
1856:   ParsedArgs<2> parsed_args;
1857:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1858:   if(_r.has_torch_function()) {
1859:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
1860:   }
1861:   if (_r.isNone(1)) {
1862:     // aten::log_sigmoid(Tensor self) -> Tensor
1863: 
1864:     auto dispatch_log_sigmoid = [](const at::Tensor & self) -> at::Tensor {
1865:       pybind11::gil_scoped_release no_gil;
1866:       return at::log_sigmoid(self);
1867:     };
1868:     return wrap(dispatch_log_sigmoid(_r.tensor(0)));
1869:   } else {
1870:     // aten::log_sigmoid.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
1871: 
1872:     auto dispatch_log_sigmoid_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
1873:       pybind11::gil_scoped_release no_gil;
1874:       return at::log_sigmoid_out(out, self);
1875:     };
1876:     return wrap(dispatch_log_sigmoid_out(_r.tensor(1), _r.tensor(0)));
1877:   }
1878:   Py_RETURN_NONE;
1879:   END_HANDLE_TH_ERRORS
1880: }
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_log_sigmoid`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_log_sigmoid`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1881-1920

```cpp
1881: 
1882: // max_pool2d_with_indices
1883: static PyObject * THPVariable_max_pool2d_with_indices(PyObject* self_, PyObject* args, PyObject* kwargs)
1884: {
1885:   HANDLE_TH_ERRORS
1886:   static PythonArgParser parser({
1887:     "max_pool2d_with_indices(Tensor input, IntArrayRef[2] kernel_size, IntArrayRef[2] stride=None, IntArrayRef[2] padding=0, IntArrayRef[2] dilation=1, bool ceil_mode=False, *, TensorList[2] out=None)",
1888:   }, /*traceable=*/true);
1889: 
1890:   ParsedArgs<7> parsed_args;
1891:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1892:   if(_r.has_torch_function()) {
1893:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
1894:   }
1895:   if (_r.isNone(6)) {
1896:     // aten::max_pool2d_with_indices(Tensor self, int[2] kernel_size, int[2] stride=[], int[2] padding=0, int[2] dilation=1, bool ceil_mode=False) -> (Tensor, Tensor)
1897: 
1898:     auto dispatch_max_pool2d_with_indices = [](const at::Tensor & self, at::IntArrayRef kernel_size, at::IntArrayRef stride, at::IntArrayRef padding, at::IntArrayRef dilation, bool ceil_mode) -> ::std::tuple<at::Tensor,at::Tensor> {
1899:       pybind11::gil_scoped_release no_gil;
1900:       return at::max_pool2d_with_indices(self, kernel_size, stride, padding, dilation, ceil_mode);
1901:     };
1902:     return wrap(dispatch_max_pool2d_with_indices(_r.tensor(0), _r.intlist(1), _r.intlist(2), _r.intlist(3), _r.intlist(4), _r.toBool(5)));
1903:   } else {
1904:     // aten::max_pool2d_with_indices.out(Tensor self, int[2] kernel_size, int[2] stride=[], int[2] padding=0, int[2] dilation=1, bool ceil_mode=False, *, Tensor(a!) out, Tensor(b!) indices) -> (Tensor(a!), Tensor(b!))
1905:     auto out = _r.tensorlist_n<2>(6);
1906:     auto dispatch_max_pool2d_with_indices_out = [](at::Tensor & out, at::Tensor & indices, const at::Tensor & self, at::IntArrayRef kernel_size, at::IntArrayRef stride, at::IntArrayRef padding, at::IntArrayRef dilation, bool ceil_mode) -> ::std::tuple<at::Tensor,at::Tensor> {
1907:       pybind11::gil_scoped_release no_gil;
1908:       return at::max_pool2d_with_indices_out(out, indices, self, kernel_size, stride, padding, dilation, ceil_mode);
1909:     };
1910:     return wrap(dispatch_max_pool2d_with_indices_out(out[0], out[1], _r.tensor(0), _r.intlist(1), _r.intlist(2), _r.intlist(3), _r.intlist(4), _r.toBool(5)));
1911:   }
1912:   Py_RETURN_NONE;
1913:   END_HANDLE_TH_ERRORS
1914: }
1915: 
1916: // max_pool3d_with_indices
1917: static PyObject * THPVariable_max_pool3d_with_indices(PyObject* self_, PyObject* args, PyObject* kwargs)
1918: {
1919:   HANDLE_TH_ERRORS
1920:   static PythonArgParser parser({
```

- EN: The main execution path in this span is carried by `THPVariable_max_pool2d_with_indices`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_max_pool2d_with_indices`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 1921-1960

```cpp
1921:     "max_pool3d_with_indices(Tensor input, IntArrayRef[3] kernel_size, IntArrayRef[3] stride=None, IntArrayRef[3] padding=0, IntArrayRef[3] dilation=1, bool ceil_mode=False, *, TensorList[2] out=None)",
1922:   }, /*traceable=*/true);
1923: 
1924:   ParsedArgs<7> parsed_args;
1925:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1926:   if(_r.has_torch_function()) {
1927:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
1928:   }
1929:   if (_r.isNone(6)) {
1930:     // aten::max_pool3d_with_indices(Tensor self, int[3] kernel_size, int[3] stride=[], int[3] padding=0, int[3] dilation=1, bool ceil_mode=False) -> (Tensor, Tensor)
1931: 
1932:     auto dispatch_max_pool3d_with_indices = [](const at::Tensor & self, at::IntArrayRef kernel_size, at::IntArrayRef stride, at::IntArrayRef padding, at::IntArrayRef dilation, bool ceil_mode) -> ::std::tuple<at::Tensor,at::Tensor> {
1933:       pybind11::gil_scoped_release no_gil;
1934:       return at::max_pool3d_with_indices(self, kernel_size, stride, padding, dilation, ceil_mode);
1935:     };
1936:     return wrap(dispatch_max_pool3d_with_indices(_r.tensor(0), _r.intlist(1), _r.intlist(2), _r.intlist(3), _r.intlist(4), _r.toBool(5)));
1937:   } else {
1938:     // aten::max_pool3d_with_indices.out(Tensor self, int[3] kernel_size, int[3] stride=[], int[3] padding=0, int[3] dilation=1, bool ceil_mode=False, *, Tensor(a!) out, Tensor(b!) indices) -> (Tensor(a!), Tensor(b!))
1939:     auto out = _r.tensorlist_n<2>(6);
1940:     auto dispatch_max_pool3d_with_indices_out = [](at::Tensor & out, at::Tensor & indices, const at::Tensor & self, at::IntArrayRef kernel_size, at::IntArrayRef stride, at::IntArrayRef padding, at::IntArrayRef dilation, bool ceil_mode) -> ::std::tuple<at::Tensor,at::Tensor> {
1941:       pybind11::gil_scoped_release no_gil;
1942:       return at::max_pool3d_with_indices_out(out, indices, self, kernel_size, stride, padding, dilation, ceil_mode);
1943:     };
1944:     return wrap(dispatch_max_pool3d_with_indices_out(out[0], out[1], _r.tensor(0), _r.intlist(1), _r.intlist(2), _r.intlist(3), _r.intlist(4), _r.toBool(5)));
1945:   }
1946:   Py_RETURN_NONE;
1947:   END_HANDLE_TH_ERRORS
1948: }
1949: 
1950: // max_unpool2d
1951: static PyObject * THPVariable_max_unpool2d(PyObject* self_, PyObject* args, PyObject* kwargs)
1952: {
1953:   HANDLE_TH_ERRORS
1954:   static PythonArgParser parser({
1955:     "max_unpool2d(Tensor input, Tensor indices, SymIntArrayRef[2] output_size, *, Tensor out=None)",
1956:   }, /*traceable=*/true);
1957: 
1958:   ParsedArgs<4> parsed_args;
1959:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1960:   if(_r.has_torch_function()) {
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `max_pool3d_with_indices`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `max_pool3d_with_indices`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 1961-2000

```cpp
1961:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
1962:   }
1963:   if (_r.isNone(3)) {
1964:     // aten::max_unpool2d(Tensor self, Tensor indices, SymInt[2] output_size) -> Tensor
1965: 
1966:     auto dispatch_max_unpool2d = [](const at::Tensor & self, const at::Tensor & indices, c10::SymIntArrayRef output_size) -> at::Tensor {
1967:       pybind11::gil_scoped_release no_gil;
1968:       return at::max_unpool2d_symint(self, indices, output_size);
1969:     };
1970:     return wrap(dispatch_max_unpool2d(_r.tensor(0), _r.tensor(1), _r.symintlist(2)));
1971:   } else {
1972:     // aten::max_unpool2d.out(Tensor self, Tensor indices, SymInt[2] output_size, *, Tensor(a!) out) -> Tensor(a!)
1973: 
1974:     auto dispatch_max_unpool2d_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & indices, c10::SymIntArrayRef output_size) -> at::Tensor {
1975:       pybind11::gil_scoped_release no_gil;
1976:       return at::max_unpool2d_symint_out(out, self, indices, output_size);
1977:     };
1978:     return wrap(dispatch_max_unpool2d_out(_r.tensor(3), _r.tensor(0), _r.tensor(1), _r.symintlist(2)));
1979:   }
1980:   Py_RETURN_NONE;
1981:   END_HANDLE_TH_ERRORS
1982: }
1983: 
1984: // max_unpool3d
1985: static PyObject * THPVariable_max_unpool3d(PyObject* self_, PyObject* args, PyObject* kwargs)
1986: {
1987:   HANDLE_TH_ERRORS
1988:   static PythonArgParser parser({
1989:     "max_unpool3d(Tensor input, Tensor indices, SymIntArrayRef[3] output_size, IntArrayRef[3] stride, IntArrayRef[3] padding, *, Tensor out=None)",
1990:   }, /*traceable=*/true);
1991: 
1992:   ParsedArgs<6> parsed_args;
1993:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1994:   if(_r.has_torch_function()) {
1995:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
1996:   }
1997:   if (_r.isNone(5)) {
1998:     // aten::max_unpool3d(Tensor self, Tensor indices, SymInt[3] output_size, int[3] stride, int[3] padding) -> Tensor
1999: 
2000:     auto dispatch_max_unpool3d = [](const at::Tensor & self, const at::Tensor & indices, c10::SymIntArrayRef output_size, at::IntArrayRef stride, at::IntArrayRef padding) -> at::Tensor {
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `max_unpool2d`, `max_unpool2d_symint`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `max_unpool2d`, `max_unpool2d_symint` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 2001-2040

```cpp
2001:       pybind11::gil_scoped_release no_gil;
2002:       return at::max_unpool3d_symint(self, indices, output_size, stride, padding);
2003:     };
2004:     return wrap(dispatch_max_unpool3d(_r.tensor(0), _r.tensor(1), _r.symintlist(2), _r.intlist(3), _r.intlist(4)));
2005:   } else {
2006:     // aten::max_unpool3d.out(Tensor self, Tensor indices, SymInt[3] output_size, int[3] stride, int[3] padding, *, Tensor(a!) out) -> Tensor(a!)
2007: 
2008:     auto dispatch_max_unpool3d_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & indices, c10::SymIntArrayRef output_size, at::IntArrayRef stride, at::IntArrayRef padding) -> at::Tensor {
2009:       pybind11::gil_scoped_release no_gil;
2010:       return at::max_unpool3d_symint_out(out, self, indices, output_size, stride, padding);
2011:     };
2012:     return wrap(dispatch_max_unpool3d_out(_r.tensor(5), _r.tensor(0), _r.tensor(1), _r.symintlist(2), _r.intlist(3), _r.intlist(4)));
2013:   }
2014:   Py_RETURN_NONE;
2015:   END_HANDLE_TH_ERRORS
2016: }
2017: 
2018: // mish
2019: static PyObject * THPVariable_mish(PyObject* self_, PyObject* args, PyObject* kwargs)
2020: {
2021:   HANDLE_TH_ERRORS
2022:   static PythonArgParser parser({
2023:     "mish(Tensor input, *, Tensor out=None)",
2024:   }, /*traceable=*/true);
2025: 
2026:   ParsedArgs<2> parsed_args;
2027:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2028:   if(_r.has_torch_function()) {
2029:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
2030:   }
2031:   if (_r.isNone(1)) {
2032:     // aten::mish(Tensor self) -> Tensor
2033: 
2034:     auto dispatch_mish = [](const at::Tensor & self) -> at::Tensor {
2035:       pybind11::gil_scoped_release no_gil;
2036:       return at::mish(self);
2037:     };
2038:     return wrap(dispatch_mish(_r.tensor(0)));
2039:   } else {
2040:     // aten::mish.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
```

- EN: The main execution path in this span is carried by `max_unpool3d_symint`, `wrap`, `max_unpool3d_symint_out`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `max_unpool3d_symint`, `wrap`, `max_unpool3d_symint_out` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 2041-2080

```cpp
2041: 
2042:     auto dispatch_mish_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
2043:       pybind11::gil_scoped_release no_gil;
2044:       return at::mish_out(out, self);
2045:     };
2046:     return wrap(dispatch_mish_out(_r.tensor(1), _r.tensor(0)));
2047:   }
2048:   Py_RETURN_NONE;
2049:   END_HANDLE_TH_ERRORS
2050: }
2051: 
2052: // mish_
2053: static PyObject * THPVariable_mish_(PyObject* self_, PyObject* args, PyObject* kwargs)
2054: {
2055:   HANDLE_TH_ERRORS
2056:   static PythonArgParser parser({
2057:     "mish_(Tensor input)",
2058:   }, /*traceable=*/true);
2059: 
2060:   ParsedArgs<1> parsed_args;
2061:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2062:   if(_r.has_torch_function()) {
2063:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
2064:   }
2065:   // aten::mish_(Tensor(a!) self) -> Tensor(a!)
2066: 
2067:   auto dispatch_mish_ = [](at::Tensor self) -> at::Tensor {
2068:     pybind11::gil_scoped_release no_gil;
2069:     return at::mish_(self);
2070:   };
2071:   return wrap(dispatch_mish_(_r.tensor(0)));
2072:   Py_RETURN_NONE;
2073:   END_HANDLE_TH_ERRORS
2074: }
2075: 
2076: // mkldnn_linear
2077: static PyObject * THPVariable_mkldnn_linear(PyObject* self_, PyObject* args, PyObject* kwargs)
2078: {
2079:   HANDLE_TH_ERRORS
2080:   static PythonArgParser parser({
```

- EN: The main execution path in this span is carried by `mish_out`, `wrap`, `THPVariable_mish_`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `mish_out`, `wrap`, `THPVariable_mish_` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2081-2120

```cpp
2081:     "mkldnn_linear(Tensor input, Tensor weight, Tensor? bias=None)",
2082:   }, /*traceable=*/true);
2083: 
2084:   ParsedArgs<3> parsed_args;
2085:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2086:   if(_r.has_torch_function()) {
2087:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
2088:   }
2089:   // aten::mkldnn_linear(Tensor self, Tensor weight, Tensor? bias=None) -> Tensor
2090: 
2091:   auto dispatch_mkldnn_linear = [](const at::Tensor & self, const at::Tensor & weight, const ::std::optional<at::Tensor> & bias) -> at::Tensor {
2092:     pybind11::gil_scoped_release no_gil;
2093:     return at::mkldnn_linear(self, weight, bias);
2094:   };
2095:   return wrap(dispatch_mkldnn_linear(_r.tensor(0), _r.tensor(1), _r.optionalTensor(2)));
2096:   Py_RETURN_NONE;
2097:   END_HANDLE_TH_ERRORS
2098: }
2099: 
2100: // mkldnn_reorder_conv2d_weight
2101: static PyObject * THPVariable_mkldnn_reorder_conv2d_weight(PyObject* self_, PyObject* args, PyObject* kwargs)
2102: {
2103:   HANDLE_TH_ERRORS
2104:   static PythonArgParser parser({
2105:     "mkldnn_reorder_conv2d_weight(Tensor input, SymIntArrayRef[2] padding=0, SymIntArrayRef[2] stride=1, SymIntArrayRef[2] dilation=1, SymInt groups=1, SymIntArrayRef? input_size=None)",
2106:   }, /*traceable=*/true);
2107: 
2108:   ParsedArgs<6> parsed_args;
2109:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2110:   if(_r.has_torch_function()) {
2111:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
2112:   }
2113:   // aten::mkldnn_reorder_conv2d_weight(Tensor self, SymInt[2] padding=0, SymInt[2] stride=1, SymInt[2] dilation=1, SymInt groups=1, SymInt[]? input_size=None) -> Tensor
2114: 
2115:   auto dispatch_mkldnn_reorder_conv2d_weight = [](const at::Tensor & self, c10::SymIntArrayRef padding, c10::SymIntArrayRef stride, c10::SymIntArrayRef dilation, c10::SymInt groups, at::OptionalSymIntArrayRef input_size) -> at::Tensor {
2116:     pybind11::gil_scoped_release no_gil;
2117:     return at::mkldnn_reorder_conv2d_weight_symint(self, padding, stride, dilation, groups, input_size);
2118:   };
2119:   return wrap(dispatch_mkldnn_reorder_conv2d_weight(_r.tensor(0), _r.symintlist(1), _r.symintlist(2), _r.symintlist(3), _r.toSymInt(4), _r.symintlistOptional(5)));
2120:   Py_RETURN_NONE;
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `mkldnn_linear`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `mkldnn_linear`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 2121-2160

```cpp
2121:   END_HANDLE_TH_ERRORS
2122: }
2123: 
2124: // mkldnn_reorder_conv3d_weight
2125: static PyObject * THPVariable_mkldnn_reorder_conv3d_weight(PyObject* self_, PyObject* args, PyObject* kwargs)
2126: {
2127:   HANDLE_TH_ERRORS
2128:   static PythonArgParser parser({
2129:     "mkldnn_reorder_conv3d_weight(Tensor input, SymIntArrayRef[3] padding=0, SymIntArrayRef[3] stride=1, SymIntArrayRef[3] dilation=1, SymInt groups=1, SymIntArrayRef? input_size=None)",
2130:   }, /*traceable=*/true);
2131: 
2132:   ParsedArgs<6> parsed_args;
2133:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2134:   if(_r.has_torch_function()) {
2135:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
2136:   }
2137:   // aten::mkldnn_reorder_conv3d_weight(Tensor self, SymInt[3] padding=0, SymInt[3] stride=1, SymInt[3] dilation=1, SymInt groups=1, SymInt[]? input_size=None) -> Tensor
2138: 
2139:   auto dispatch_mkldnn_reorder_conv3d_weight = [](const at::Tensor & self, c10::SymIntArrayRef padding, c10::SymIntArrayRef stride, c10::SymIntArrayRef dilation, c10::SymInt groups, at::OptionalSymIntArrayRef input_size) -> at::Tensor {
2140:     pybind11::gil_scoped_release no_gil;
2141:     return at::mkldnn_reorder_conv3d_weight_symint(self, padding, stride, dilation, groups, input_size);
2142:   };
2143:   return wrap(dispatch_mkldnn_reorder_conv3d_weight(_r.tensor(0), _r.symintlist(1), _r.symintlist(2), _r.symintlist(3), _r.toSymInt(4), _r.symintlistOptional(5)));
2144:   Py_RETURN_NONE;
2145:   END_HANDLE_TH_ERRORS
2146: }
2147: 
2148: // mse_loss
2149: static PyObject * THPVariable_mse_loss(PyObject* self_, PyObject* args, PyObject* kwargs)
2150: {
2151:   HANDLE_TH_ERRORS
2152:   static PythonArgParser parser({
2153:     "mse_loss(Tensor input, Tensor target, int64_t reduction=at::Reduction::Mean, *, Tensor out=None)",
2154:   }, /*traceable=*/true);
2155: 
2156:   ParsedArgs<4> parsed_args;
2157:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2158:   if(_r.has_torch_function()) {
2159:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
2160:   }
```

- EN: The main execution path in this span is carried by `THPVariable_mkldnn_reorder_conv3d_weight`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_mkldnn_reorder_conv3d_weight`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 2161-2200

```cpp
2161:   if (_r.isNone(3)) {
2162:     // aten::mse_loss(Tensor self, Tensor target, int reduction=Mean) -> Tensor
2163: 
2164:     auto dispatch_mse_loss = [](const at::Tensor & self, const at::Tensor & target, int64_t reduction) -> at::Tensor {
2165:       pybind11::gil_scoped_release no_gil;
2166:       return at::mse_loss(self, target, reduction);
2167:     };
2168:     return wrap(dispatch_mse_loss(_r.tensor(0), _r.tensor(1), _r.toInt64(2)));
2169:   } else {
2170:     // aten::mse_loss.out(Tensor self, Tensor target, int reduction=Mean, *, Tensor(a!) out) -> Tensor(a!)
2171: 
2172:     auto dispatch_mse_loss_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & target, int64_t reduction) -> at::Tensor {
2173:       pybind11::gil_scoped_release no_gil;
2174:       return at::mse_loss_out(out, self, target, reduction);
2175:     };
2176:     return wrap(dispatch_mse_loss_out(_r.tensor(3), _r.tensor(0), _r.tensor(1), _r.toInt64(2)));
2177:   }
2178:   Py_RETURN_NONE;
2179:   END_HANDLE_TH_ERRORS
2180: }
2181: 
2182: // multi_margin_loss
2183: static PyObject * THPVariable_multi_margin_loss(PyObject* self_, PyObject* args, PyObject* kwargs)
2184: {
2185:   HANDLE_TH_ERRORS
2186:   static PythonArgParser parser({
2187:     "multi_margin_loss(Tensor input, Tensor target, Scalar p=1, Scalar margin=1, Tensor? weight=None, int64_t reduction=at::Reduction::Mean, *, Tensor out=None)",
2188:   }, /*traceable=*/true);
2189: 
2190:   ParsedArgs<7> parsed_args;
2191:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2192:   if(_r.has_torch_function()) {
2193:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
2194:   }
2195:   if (_r.isNone(6)) {
2196:     // aten::multi_margin_loss(Tensor self, Tensor target, Scalar p=1, Scalar margin=1, Tensor? weight=None, int reduction=Mean) -> Tensor
2197: 
2198:     auto dispatch_multi_margin_loss = [](const at::Tensor & self, const at::Tensor & target, const at::Scalar & p, const at::Scalar & margin, const ::std::optional<at::Tensor> & weight, int64_t reduction) -> at::Tensor {
2199:       pybind11::gil_scoped_release no_gil;
2200:       return at::multi_margin_loss(self, target, p, margin, weight, reduction);
```

- EN: The main execution path in this span is carried by `mse_loss`, `wrap`, `mse_loss_out`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `mse_loss`, `wrap`, `mse_loss_out` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2201-2240

```cpp
2201:     };
2202:     return wrap(dispatch_multi_margin_loss(_r.tensor(0), _r.tensor(1), _r.scalar(2), _r.scalar(3), _r.optionalTensor(4), _r.toInt64(5)));
2203:   } else {
2204:     // aten::multi_margin_loss.out(Tensor self, Tensor target, Scalar p=1, Scalar margin=1, Tensor? weight=None, int reduction=Mean, *, Tensor(a!) out) -> Tensor(a!)
2205: 
2206:     auto dispatch_multi_margin_loss_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & target, const at::Scalar & p, const at::Scalar & margin, const ::std::optional<at::Tensor> & weight, int64_t reduction) -> at::Tensor {
2207:       pybind11::gil_scoped_release no_gil;
2208:       return at::multi_margin_loss_out(out, self, target, p, margin, weight, reduction);
2209:     };
2210:     return wrap(dispatch_multi_margin_loss_out(_r.tensor(6), _r.tensor(0), _r.tensor(1), _r.scalar(2), _r.scalar(3), _r.optionalTensor(4), _r.toInt64(5)));
2211:   }
2212:   Py_RETURN_NONE;
2213:   END_HANDLE_TH_ERRORS
2214: }
2215: 
2216: // multilabel_margin_loss
2217: static PyObject * THPVariable_multilabel_margin_loss(PyObject* self_, PyObject* args, PyObject* kwargs)
2218: {
2219:   HANDLE_TH_ERRORS
2220:   static PythonArgParser parser({
2221:     "multilabel_margin_loss(Tensor input, Tensor target, int64_t reduction=at::Reduction::Mean, *, Tensor out=None)",
2222:   }, /*traceable=*/true);
2223: 
2224:   ParsedArgs<4> parsed_args;
2225:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2226:   if(_r.has_torch_function()) {
2227:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
2228:   }
2229:   if (_r.isNone(3)) {
2230:     // aten::multilabel_margin_loss(Tensor self, Tensor target, int reduction=Mean) -> Tensor
2231: 
2232:     auto dispatch_multilabel_margin_loss = [](const at::Tensor & self, const at::Tensor & target, int64_t reduction) -> at::Tensor {
2233:       pybind11::gil_scoped_release no_gil;
2234:       return at::multilabel_margin_loss(self, target, reduction);
2235:     };
2236:     return wrap(dispatch_multilabel_margin_loss(_r.tensor(0), _r.tensor(1), _r.toInt64(2)));
2237:   } else {
2238:     // aten::multilabel_margin_loss.out(Tensor self, Tensor target, int reduction=Mean, *, Tensor(a!) out) -> Tensor(a!)
2239: 
2240:     auto dispatch_multilabel_margin_loss_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & target, int64_t reduction) -> at::Tensor {
```

- EN: The main execution path in this span is carried by `wrap`, `multi_margin_loss_out`, `THPVariable_multilabel_margin_loss`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `multi_margin_loss_out`, `THPVariable_multilabel_margin_loss` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2241-2280

```cpp
2241:       pybind11::gil_scoped_release no_gil;
2242:       return at::multilabel_margin_loss_out(out, self, target, reduction);
2243:     };
2244:     return wrap(dispatch_multilabel_margin_loss_out(_r.tensor(3), _r.tensor(0), _r.tensor(1), _r.toInt64(2)));
2245:   }
2246:   Py_RETURN_NONE;
2247:   END_HANDLE_TH_ERRORS
2248: }
2249: 
2250: // nll_loss
2251: static PyObject * THPVariable_nll_loss(PyObject* self_, PyObject* args, PyObject* kwargs)
2252: {
2253:   HANDLE_TH_ERRORS
2254:   static PythonArgParser parser({
2255:     "nll_loss(Tensor input, Tensor target, Tensor? weight=None, int64_t reduction=at::Reduction::Mean, SymInt ignore_index=-100, *, Tensor out=None)",
2256:   }, /*traceable=*/true);
2257: 
2258:   ParsedArgs<6> parsed_args;
2259:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2260:   if(_r.has_torch_function()) {
2261:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
2262:   }
2263:   if (_r.isNone(5)) {
2264:     // aten::nll_loss(Tensor self, Tensor target, Tensor? weight=None, int reduction=Mean, SymInt ignore_index=-100) -> Tensor
2265: 
2266:     auto dispatch_nll_loss = [](const at::Tensor & self, const at::Tensor & target, const ::std::optional<at::Tensor> & weight, int64_t reduction, c10::SymInt ignore_index) -> at::Tensor {
2267:       pybind11::gil_scoped_release no_gil;
2268:       return at::nll_loss_symint(self, target, weight, reduction, ignore_index);
2269:     };
2270:     return wrap(dispatch_nll_loss(_r.tensor(0), _r.tensor(1), _r.optionalTensor(2), _r.toInt64(3), _r.toSymInt(4)));
2271:   } else {
2272:     // aten::nll_loss.out(Tensor self, Tensor target, Tensor? weight=None, int reduction=Mean, SymInt ignore_index=-100, *, Tensor(a!) out) -> Tensor(a!)
2273: 
2274:     auto dispatch_nll_loss_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & target, const ::std::optional<at::Tensor> & weight, int64_t reduction, c10::SymInt ignore_index) -> at::Tensor {
2275:       pybind11::gil_scoped_release no_gil;
2276:       return at::nll_loss_symint_out(out, self, target, weight, reduction, ignore_index);
2277:     };
2278:     return wrap(dispatch_nll_loss_out(_r.tensor(5), _r.tensor(0), _r.tensor(1), _r.optionalTensor(2), _r.toInt64(3), _r.toSymInt(4)));
2279:   }
2280:   Py_RETURN_NONE;
```

- EN: The main execution path in this span is carried by `multilabel_margin_loss_out`, `wrap`, `THPVariable_nll_loss`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `multilabel_margin_loss_out`, `wrap`, `THPVariable_nll_loss` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2281-2320

```cpp
2281:   END_HANDLE_TH_ERRORS
2282: }
2283: 
2284: // nll_loss2d
2285: static PyObject * THPVariable_nll_loss2d(PyObject* self_, PyObject* args, PyObject* kwargs)
2286: {
2287:   HANDLE_TH_ERRORS
2288:   static PythonArgParser parser({
2289:     "nll_loss2d(Tensor input, Tensor target, Tensor? weight=None, int64_t reduction=at::Reduction::Mean, SymInt ignore_index=-100, *, Tensor out=None)",
2290:   }, /*traceable=*/true);
2291: 
2292:   ParsedArgs<6> parsed_args;
2293:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2294:   if(_r.has_torch_function()) {
2295:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
2296:   }
2297:   if (_r.isNone(5)) {
2298:     // aten::nll_loss2d(Tensor self, Tensor target, Tensor? weight=None, int reduction=Mean, SymInt ignore_index=-100) -> Tensor
2299: 
2300:     auto dispatch_nll_loss2d = [](const at::Tensor & self, const at::Tensor & target, const ::std::optional<at::Tensor> & weight, int64_t reduction, c10::SymInt ignore_index) -> at::Tensor {
2301:       pybind11::gil_scoped_release no_gil;
2302:       return at::nll_loss2d_symint(self, target, weight, reduction, ignore_index);
2303:     };
2304:     return wrap(dispatch_nll_loss2d(_r.tensor(0), _r.tensor(1), _r.optionalTensor(2), _r.toInt64(3), _r.toSymInt(4)));
2305:   } else {
2306:     // aten::nll_loss2d.out(Tensor self, Tensor target, Tensor? weight=None, int reduction=Mean, SymInt ignore_index=-100, *, Tensor(a!) out) -> Tensor(a!)
2307: 
2308:     auto dispatch_nll_loss2d_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & target, const ::std::optional<at::Tensor> & weight, int64_t reduction, c10::SymInt ignore_index) -> at::Tensor {
2309:       pybind11::gil_scoped_release no_gil;
2310:       return at::nll_loss2d_symint_out(out, self, target, weight, reduction, ignore_index);
2311:     };
2312:     return wrap(dispatch_nll_loss2d_out(_r.tensor(5), _r.tensor(0), _r.tensor(1), _r.optionalTensor(2), _r.toInt64(3), _r.toSymInt(4)));
2313:   }
2314:   Py_RETURN_NONE;
2315:   END_HANDLE_TH_ERRORS
2316: }
2317: 
2318: // nll_loss_nd
2319: static PyObject * THPVariable_nll_loss_nd(PyObject* self_, PyObject* args, PyObject* kwargs)
2320: {
```

- EN: The main execution path in this span is carried by `THPVariable_nll_loss2d`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_nll_loss2d`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2321-2360

```cpp
2321:   HANDLE_TH_ERRORS
2322:   static PythonArgParser parser({
2323:     "nll_loss_nd(Tensor input, Tensor target, Tensor? weight=None, int64_t reduction=at::Reduction::Mean, SymInt ignore_index=-100)",
2324:   }, /*traceable=*/true);
2325: 
2326:   ParsedArgs<5> parsed_args;
2327:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2328:   if(_r.has_torch_function()) {
2329:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
2330:   }
2331:   // aten::nll_loss_nd(Tensor self, Tensor target, Tensor? weight=None, int reduction=Mean, SymInt ignore_index=-100) -> Tensor
2332: 
2333:   auto dispatch_nll_loss_nd = [](const at::Tensor & self, const at::Tensor & target, const ::std::optional<at::Tensor> & weight, int64_t reduction, c10::SymInt ignore_index) -> at::Tensor {
2334:     pybind11::gil_scoped_release no_gil;
2335:     return at::nll_loss_nd_symint(self, target, weight, reduction, ignore_index);
2336:   };
2337:   return wrap(dispatch_nll_loss_nd(_r.tensor(0), _r.tensor(1), _r.optionalTensor(2), _r.toInt64(3), _r.toSymInt(4)));
2338:   Py_RETURN_NONE;
2339:   END_HANDLE_TH_ERRORS
2340: }
2341: 
2342: // one_hot
2343: static PyObject * THPVariable_one_hot(PyObject* self_, PyObject* args, PyObject* kwargs)
2344: {
2345:   HANDLE_TH_ERRORS
2346:   static PythonArgParser parser({
2347:     "one_hot(Tensor input, int64_t num_classes=-1)",
2348:   }, /*traceable=*/true);
2349: 
2350:   ParsedArgs<2> parsed_args;
2351:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2352:   if(_r.has_torch_function()) {
2353:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
2354:   }
2355:   // aten::one_hot(Tensor self, int num_classes=-1) -> Tensor
2356: 
2357:   auto dispatch_one_hot = [](const at::Tensor & self, int64_t num_classes) -> at::Tensor {
2358:     pybind11::gil_scoped_release no_gil;
2359:     return at::one_hot(self, num_classes);
2360:   };
```

- EN: The main execution path in this span is carried by `parser`, `handle_torch_function`, `nll_loss_nd`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `parser`, `handle_torch_function`, `nll_loss_nd` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2361-2400

```cpp
2361:   return wrap(dispatch_one_hot(_r.tensor(0), _r.toInt64(1)));
2362:   Py_RETURN_NONE;
2363:   END_HANDLE_TH_ERRORS
2364: }
2365: 
2366: // pad
2367: static PyObject * THPVariable_pad(PyObject* self_, PyObject* args, PyObject* kwargs)
2368: {
2369:   HANDLE_TH_ERRORS
2370:   static PythonArgParser parser({
2371:     "pad(Tensor input, SymIntArrayRef pad, c10::string_view mode=\"constant\", double? value=None)",
2372:   }, /*traceable=*/true);
2373: 
2374:   ParsedArgs<4> parsed_args;
2375:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2376:   if(_r.has_torch_function()) {
2377:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
2378:   }
2379:   // aten::pad(Tensor self, SymInt[] pad, str mode="constant", float? value=None) -> Tensor
2380: 
2381:   auto dispatch_pad = [](const at::Tensor & self, c10::SymIntArrayRef pad, c10::string_view mode, ::std::optional<double> value) -> at::Tensor {
2382:     pybind11::gil_scoped_release no_gil;
2383:     return at::pad_symint(self, pad, mode, value);
2384:   };
2385:   return wrap(dispatch_pad(_r.tensor(0), _r.symintlist(1), _r.stringView(2), _r.toDoubleOptional(3)));
2386:   Py_RETURN_NONE;
2387:   END_HANDLE_TH_ERRORS
2388: }
2389: 
2390: // pad_sequence
2391: static PyObject * THPVariable_pad_sequence(PyObject* self_, PyObject* args, PyObject* kwargs)
2392: {
2393:   HANDLE_TH_ERRORS
2394:   static PythonArgParser parser({
2395:     "pad_sequence(TensorList sequences, bool batch_first=False, double padding_value=0.0, c10::string_view padding_side=\"right\")",
2396:   }, /*traceable=*/true);
2397: 
2398:   ParsedArgs<4> parsed_args;
2399:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2400:   if(_r.has_torch_function()) {
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_pad`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_pad`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 2401-2440

```cpp
2401:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
2402:   }
2403:   // aten::pad_sequence(Tensor[] sequences, bool batch_first=False, float padding_value=0.0, str padding_side="right") -> Tensor
2404: 
2405:   auto dispatch_pad_sequence = [](at::TensorList sequences, bool batch_first, double padding_value, c10::string_view padding_side) -> at::Tensor {
2406:     pybind11::gil_scoped_release no_gil;
2407:     return at::pad_sequence(sequences, batch_first, padding_value, padding_side);
2408:   };
2409:   return wrap(dispatch_pad_sequence(_r.tensorlist(0), _r.toBool(1), _r.toDouble(2), _r.stringView(3)));
2410:   Py_RETURN_NONE;
2411:   END_HANDLE_TH_ERRORS
2412: }
2413: 
2414: // reflection_pad1d
2415: static PyObject * THPVariable_reflection_pad1d(PyObject* self_, PyObject* args, PyObject* kwargs)
2416: {
2417:   HANDLE_TH_ERRORS
2418:   static PythonArgParser parser({
2419:     "reflection_pad1d(Tensor input, SymIntArrayRef[2] padding, *, Tensor out=None)",
2420:   }, /*traceable=*/true);
2421: 
2422:   ParsedArgs<3> parsed_args;
2423:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2424:   if(_r.has_torch_function()) {
2425:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
2426:   }
2427:   if (_r.isNone(2)) {
2428:     // aten::reflection_pad1d(Tensor self, SymInt[2] padding) -> Tensor
2429: 
2430:     auto dispatch_reflection_pad1d = [](const at::Tensor & self, c10::SymIntArrayRef padding) -> at::Tensor {
2431:       pybind11::gil_scoped_release no_gil;
2432:       return at::reflection_pad1d_symint(self, padding);
2433:     };
2434:     return wrap(dispatch_reflection_pad1d(_r.tensor(0), _r.symintlist(1)));
2435:   } else {
2436:     // aten::reflection_pad1d.out(Tensor self, SymInt[2] padding, *, Tensor(a!) out) -> Tensor(a!)
2437: 
2438:     auto dispatch_reflection_pad1d_out = [](at::Tensor out, const at::Tensor & self, c10::SymIntArrayRef padding) -> at::Tensor {
2439:       pybind11::gil_scoped_release no_gil;
2440:       return at::reflection_pad1d_symint_out(out, self, padding);
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `pad_sequence`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `pad_sequence`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 2441-2480

```cpp
2441:     };
2442:     return wrap(dispatch_reflection_pad1d_out(_r.tensor(2), _r.tensor(0), _r.symintlist(1)));
2443:   }
2444:   Py_RETURN_NONE;
2445:   END_HANDLE_TH_ERRORS
2446: }
2447: 
2448: // reflection_pad2d
2449: static PyObject * THPVariable_reflection_pad2d(PyObject* self_, PyObject* args, PyObject* kwargs)
2450: {
2451:   HANDLE_TH_ERRORS
2452:   static PythonArgParser parser({
2453:     "reflection_pad2d(Tensor input, SymIntArrayRef[4] padding, *, Tensor out=None)",
2454:   }, /*traceable=*/true);
2455: 
2456:   ParsedArgs<3> parsed_args;
2457:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2458:   if(_r.has_torch_function()) {
2459:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
2460:   }
2461:   if (_r.isNone(2)) {
2462:     // aten::reflection_pad2d(Tensor self, SymInt[4] padding) -> Tensor
2463: 
2464:     auto dispatch_reflection_pad2d = [](const at::Tensor & self, c10::SymIntArrayRef padding) -> at::Tensor {
2465:       pybind11::gil_scoped_release no_gil;
2466:       return at::reflection_pad2d_symint(self, padding);
2467:     };
2468:     return wrap(dispatch_reflection_pad2d(_r.tensor(0), _r.symintlist(1)));
2469:   } else {
2470:     // aten::reflection_pad2d.out(Tensor self, SymInt[4] padding, *, Tensor(a!) out) -> Tensor(a!)
2471: 
2472:     auto dispatch_reflection_pad2d_out = [](at::Tensor out, const at::Tensor & self, c10::SymIntArrayRef padding) -> at::Tensor {
2473:       pybind11::gil_scoped_release no_gil;
2474:       return at::reflection_pad2d_symint_out(out, self, padding);
2475:     };
2476:     return wrap(dispatch_reflection_pad2d_out(_r.tensor(2), _r.tensor(0), _r.symintlist(1)));
2477:   }
2478:   Py_RETURN_NONE;
2479:   END_HANDLE_TH_ERRORS
2480: }
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_reflection_pad2d`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_reflection_pad2d`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 2481-2520

```cpp
2481: 
2482: // reflection_pad3d
2483: static PyObject * THPVariable_reflection_pad3d(PyObject* self_, PyObject* args, PyObject* kwargs)
2484: {
2485:   HANDLE_TH_ERRORS
2486:   static PythonArgParser parser({
2487:     "reflection_pad3d(Tensor input, SymIntArrayRef[6] padding, *, Tensor out=None)",
2488:   }, /*traceable=*/true);
2489: 
2490:   ParsedArgs<3> parsed_args;
2491:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2492:   if(_r.has_torch_function()) {
2493:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
2494:   }
2495:   if (_r.isNone(2)) {
2496:     // aten::reflection_pad3d(Tensor self, SymInt[6] padding) -> Tensor
2497: 
2498:     auto dispatch_reflection_pad3d = [](const at::Tensor & self, c10::SymIntArrayRef padding) -> at::Tensor {
2499:       pybind11::gil_scoped_release no_gil;
2500:       return at::reflection_pad3d_symint(self, padding);
2501:     };
2502:     return wrap(dispatch_reflection_pad3d(_r.tensor(0), _r.symintlist(1)));
2503:   } else {
2504:     // aten::reflection_pad3d.out(Tensor self, SymInt[6] padding, *, Tensor(a!) out) -> Tensor(a!)
2505: 
2506:     auto dispatch_reflection_pad3d_out = [](at::Tensor out, const at::Tensor & self, c10::SymIntArrayRef padding) -> at::Tensor {
2507:       pybind11::gil_scoped_release no_gil;
2508:       return at::reflection_pad3d_symint_out(out, self, padding);
2509:     };
2510:     return wrap(dispatch_reflection_pad3d_out(_r.tensor(2), _r.tensor(0), _r.symintlist(1)));
2511:   }
2512:   Py_RETURN_NONE;
2513:   END_HANDLE_TH_ERRORS
2514: }
2515: 
2516: // relu6
2517: static PyObject * THPVariable_relu6(PyObject* self_, PyObject* args, PyObject* kwargs)
2518: {
2519:   HANDLE_TH_ERRORS
2520:   static PythonArgParser parser({
```

- EN: The main execution path in this span is carried by `THPVariable_reflection_pad3d`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_reflection_pad3d`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 2521-2560

```cpp
2521:     "relu6(Tensor input)",
2522:   }, /*traceable=*/true);
2523: 
2524:   ParsedArgs<1> parsed_args;
2525:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2526:   if(_r.has_torch_function()) {
2527:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
2528:   }
2529:   // aten::relu6(Tensor self) -> Tensor
2530: 
2531:   auto dispatch_relu6 = [](const at::Tensor & self) -> at::Tensor {
2532:     pybind11::gil_scoped_release no_gil;
2533:     return at::relu6(self);
2534:   };
2535:   return wrap(dispatch_relu6(_r.tensor(0)));
2536:   Py_RETURN_NONE;
2537:   END_HANDLE_TH_ERRORS
2538: }
2539: 
2540: // relu6_
2541: static PyObject * THPVariable_relu6_(PyObject* self_, PyObject* args, PyObject* kwargs)
2542: {
2543:   HANDLE_TH_ERRORS
2544:   static PythonArgParser parser({
2545:     "relu6_(Tensor input)",
2546:   }, /*traceable=*/true);
2547: 
2548:   ParsedArgs<1> parsed_args;
2549:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2550:   if(_r.has_torch_function()) {
2551:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
2552:   }
2553:   // aten::relu6_(Tensor(a!) self) -> Tensor(a!)
2554: 
2555:   auto dispatch_relu6_ = [](at::Tensor self) -> at::Tensor {
2556:     pybind11::gil_scoped_release no_gil;
2557:     return at::relu6_(self);
2558:   };
2559:   return wrap(dispatch_relu6_(_r.tensor(0)));
2560:   Py_RETURN_NONE;
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `relu6`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `relu6`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2561-2600

```cpp
2561:   END_HANDLE_TH_ERRORS
2562: }
2563: 
2564: // replication_pad1d
2565: static PyObject * THPVariable_replication_pad1d(PyObject* self_, PyObject* args, PyObject* kwargs)
2566: {
2567:   HANDLE_TH_ERRORS
2568:   static PythonArgParser parser({
2569:     "replication_pad1d(Tensor input, SymIntArrayRef[2] padding, *, Tensor out=None)",
2570:   }, /*traceable=*/true);
2571: 
2572:   ParsedArgs<3> parsed_args;
2573:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2574:   if(_r.has_torch_function()) {
2575:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
2576:   }
2577:   if (_r.isNone(2)) {
2578:     // aten::replication_pad1d(Tensor self, SymInt[2] padding) -> Tensor
2579: 
2580:     auto dispatch_replication_pad1d = [](const at::Tensor & self, c10::SymIntArrayRef padding) -> at::Tensor {
2581:       pybind11::gil_scoped_release no_gil;
2582:       return at::replication_pad1d_symint(self, padding);
2583:     };
2584:     return wrap(dispatch_replication_pad1d(_r.tensor(0), _r.symintlist(1)));
2585:   } else {
2586:     // aten::replication_pad1d.out(Tensor self, SymInt[2] padding, *, Tensor(a!) out) -> Tensor(a!)
2587: 
2588:     auto dispatch_replication_pad1d_out = [](at::Tensor out, const at::Tensor & self, c10::SymIntArrayRef padding) -> at::Tensor {
2589:       pybind11::gil_scoped_release no_gil;
2590:       return at::replication_pad1d_symint_out(out, self, padding);
2591:     };
2592:     return wrap(dispatch_replication_pad1d_out(_r.tensor(2), _r.tensor(0), _r.symintlist(1)));
2593:   }
2594:   Py_RETURN_NONE;
2595:   END_HANDLE_TH_ERRORS
2596: }
2597: 
2598: // replication_pad2d
2599: static PyObject * THPVariable_replication_pad2d(PyObject* self_, PyObject* args, PyObject* kwargs)
2600: {
```

- EN: The main execution path in this span is carried by `THPVariable_replication_pad1d`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_replication_pad1d`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 2601-2640

```cpp
2601:   HANDLE_TH_ERRORS
2602:   static PythonArgParser parser({
2603:     "replication_pad2d(Tensor input, SymIntArrayRef[4] padding, *, Tensor out=None)",
2604:   }, /*traceable=*/true);
2605: 
2606:   ParsedArgs<3> parsed_args;
2607:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2608:   if(_r.has_torch_function()) {
2609:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
2610:   }
2611:   if (_r.isNone(2)) {
2612:     // aten::replication_pad2d(Tensor self, SymInt[4] padding) -> Tensor
2613: 
2614:     auto dispatch_replication_pad2d = [](const at::Tensor & self, c10::SymIntArrayRef padding) -> at::Tensor {
2615:       pybind11::gil_scoped_release no_gil;
2616:       return at::replication_pad2d_symint(self, padding);
2617:     };
2618:     return wrap(dispatch_replication_pad2d(_r.tensor(0), _r.symintlist(1)));
2619:   } else {
2620:     // aten::replication_pad2d.out(Tensor self, SymInt[4] padding, *, Tensor(a!) out) -> Tensor(a!)
2621: 
2622:     auto dispatch_replication_pad2d_out = [](at::Tensor out, const at::Tensor & self, c10::SymIntArrayRef padding) -> at::Tensor {
2623:       pybind11::gil_scoped_release no_gil;
2624:       return at::replication_pad2d_symint_out(out, self, padding);
2625:     };
2626:     return wrap(dispatch_replication_pad2d_out(_r.tensor(2), _r.tensor(0), _r.symintlist(1)));
2627:   }
2628:   Py_RETURN_NONE;
2629:   END_HANDLE_TH_ERRORS
2630: }
2631: 
2632: // replication_pad3d
2633: static PyObject * THPVariable_replication_pad3d(PyObject* self_, PyObject* args, PyObject* kwargs)
2634: {
2635:   HANDLE_TH_ERRORS
2636:   static PythonArgParser parser({
2637:     "replication_pad3d(Tensor input, SymIntArrayRef[6] padding, *, Tensor out=None)",
2638:   }, /*traceable=*/true);
2639: 
2640:   ParsedArgs<3> parsed_args;
```

- EN: The main execution path in this span is carried by `parser`, `handle_torch_function`, `replication_pad2d`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `parser`, `handle_torch_function`, `replication_pad2d` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 2641-2680

```cpp
2641:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2642:   if(_r.has_torch_function()) {
2643:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
2644:   }
2645:   if (_r.isNone(2)) {
2646:     // aten::replication_pad3d(Tensor self, SymInt[6] padding) -> Tensor
2647: 
2648:     auto dispatch_replication_pad3d = [](const at::Tensor & self, c10::SymIntArrayRef padding) -> at::Tensor {
2649:       pybind11::gil_scoped_release no_gil;
2650:       return at::replication_pad3d_symint(self, padding);
2651:     };
2652:     return wrap(dispatch_replication_pad3d(_r.tensor(0), _r.symintlist(1)));
2653:   } else {
2654:     // aten::replication_pad3d.out(Tensor self, SymInt[6] padding, *, Tensor(a!) out) -> Tensor(a!)
2655: 
2656:     auto dispatch_replication_pad3d_out = [](at::Tensor out, const at::Tensor & self, c10::SymIntArrayRef padding) -> at::Tensor {
2657:       pybind11::gil_scoped_release no_gil;
2658:       return at::replication_pad3d_symint_out(out, self, padding);
2659:     };
2660:     return wrap(dispatch_replication_pad3d_out(_r.tensor(2), _r.tensor(0), _r.symintlist(1)));
2661:   }
2662:   Py_RETURN_NONE;
2663:   END_HANDLE_TH_ERRORS
2664: }
2665: 
2666: // rrelu_with_noise
2667: static PyObject * THPVariable_rrelu_with_noise(PyObject* self_, PyObject* args, PyObject* kwargs)
2668: {
2669:   HANDLE_TH_ERRORS
2670:   static PythonArgParser parser({
2671:     "rrelu_with_noise(Tensor input, Tensor noise, Scalar lower=0.125, Scalar upper=0.3333333333333333, bool training=False, Generator? generator=None, *, Tensor out=None)",
2672:   }, /*traceable=*/true);
2673: 
2674:   ParsedArgs<7> parsed_args;
2675:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2676:   if(_r.has_torch_function()) {
2677:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
2678:   }
2679:   if (_r.isNone(6)) {
2680:     // aten::rrelu_with_noise(Tensor self, Tensor(b!) noise, Scalar lower=0.125, Scalar upper=0.3333333333333333, bool training=False, Generator? generator=None) -> Tensor
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `replication_pad3d`, `replication_pad3d_symint`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `replication_pad3d`, `replication_pad3d_symint` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 2681-2720

```cpp
2681: 
2682:     auto dispatch_rrelu_with_noise = [](const at::Tensor & self, at::Tensor noise, const at::Scalar & lower, const at::Scalar & upper, bool training, ::std::optional<at::Generator> generator) -> at::Tensor {
2683:       pybind11::gil_scoped_release no_gil;
2684:       return at::rrelu_with_noise(self, noise, lower, upper, training, generator);
2685:     };
2686:     return wrap(dispatch_rrelu_with_noise(_r.tensor(0), _r.tensor(1), _r.scalar(2), _r.scalar(3), _r.toBool(4), _r.generator(5)));
2687:   } else {
2688:     // aten::rrelu_with_noise.out(Tensor self, Tensor(b!) noise, Scalar lower=0.125, Scalar upper=0.3333333333333333, bool training=False, Generator? generator=None, *, Tensor(a!) out) -> Tensor(a!)
2689: 
2690:     auto dispatch_rrelu_with_noise_out = [](at::Tensor out, const at::Tensor & self, at::Tensor noise, const at::Scalar & lower, const at::Scalar & upper, bool training, ::std::optional<at::Generator> generator) -> at::Tensor {
2691:       pybind11::gil_scoped_release no_gil;
2692:       return at::rrelu_with_noise_out(out, self, noise, lower, upper, training, generator);
2693:     };
2694:     return wrap(dispatch_rrelu_with_noise_out(_r.tensor(6), _r.tensor(0), _r.tensor(1), _r.scalar(2), _r.scalar(3), _r.toBool(4), _r.generator(5)));
2695:   }
2696:   Py_RETURN_NONE;
2697:   END_HANDLE_TH_ERRORS
2698: }
2699: 
2700: // rrelu_with_noise_
2701: static PyObject * THPVariable_rrelu_with_noise_(PyObject* self_, PyObject* args, PyObject* kwargs)
2702: {
2703:   HANDLE_TH_ERRORS
2704:   static PythonArgParser parser({
2705:     "rrelu_with_noise_(Tensor input, Tensor noise, Scalar lower=0.125, Scalar upper=0.3333333333333333, bool training=False, Generator? generator=None)",
2706:   }, /*traceable=*/true);
2707: 
2708:   ParsedArgs<6> parsed_args;
2709:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2710:   if(_r.has_torch_function()) {
2711:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
2712:   }
2713:   // aten::rrelu_with_noise_(Tensor(a!) self, Tensor(b!) noise, Scalar lower=0.125, Scalar upper=0.3333333333333333, bool training=False, Generator? generator=None) -> Tensor(a!)
2714: 
2715:   auto dispatch_rrelu_with_noise_ = [](at::Tensor self, at::Tensor noise, const at::Scalar & lower, const at::Scalar & upper, bool training, ::std::optional<at::Generator> generator) -> at::Tensor {
2716:     pybind11::gil_scoped_release no_gil;
2717:     return at::rrelu_with_noise_(self, noise, lower, upper, training, generator);
2718:   };
2719:   return wrap(dispatch_rrelu_with_noise_(_r.tensor(0), _r.tensor(1), _r.scalar(2), _r.scalar(3), _r.toBool(4), _r.generator(5)));
2720:   Py_RETURN_NONE;
```

- EN: The main execution path in this span is carried by `rrelu_with_noise`, `wrap`, `rrelu_with_noise_out`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `rrelu_with_noise`, `wrap`, `rrelu_with_noise_out` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2721-2760

```cpp
2721:   END_HANDLE_TH_ERRORS
2722: }
2723: 
2724: // scaled_dot_product_attention
2725: static PyObject * THPVariable_scaled_dot_product_attention(PyObject* self_, PyObject* args, PyObject* kwargs)
2726: {
2727:   HANDLE_TH_ERRORS
2728:   static PythonArgParser parser({
2729:     "scaled_dot_product_attention(Tensor query, Tensor key, Tensor value, Tensor? attn_mask=None, double dropout_p=0.0, bool is_causal=False, *, double? scale=None, bool enable_gqa=False)",
2730:   }, /*traceable=*/true);
2731: 
2732:   ParsedArgs<8> parsed_args;
2733:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2734:   if(_r.has_torch_function()) {
2735:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
2736:   }
2737:   // aten::scaled_dot_product_attention(Tensor query, Tensor key, Tensor value, Tensor? attn_mask=None, float dropout_p=0.0, bool is_causal=False, *, float? scale=None, bool enable_gqa=False) -> Tensor
2738: 
2739:   auto dispatch_scaled_dot_product_attention = [](const at::Tensor & query, const at::Tensor & key, const at::Tensor & value, const ::std::optional<at::Tensor> & attn_mask, double dropout_p, bool is_causal, ::std::optional<double> scale, bool enable_gqa) -> at::Tensor {
2740:     pybind11::gil_scoped_release no_gil;
2741:     return at::scaled_dot_product_attention(query, key, value, attn_mask, dropout_p, is_causal, scale, enable_gqa);
2742:   };
2743:   return wrap(dispatch_scaled_dot_product_attention(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.optionalTensor(3), _r.toDouble(4), _r.toBool(5), _r.toDoubleOptional(6), _r.toBool(7)));
2744:   Py_RETURN_NONE;
2745:   END_HANDLE_TH_ERRORS
2746: }
2747: 
2748: // silu
2749: static PyObject * THPVariable_silu(PyObject* self_, PyObject* args, PyObject* kwargs)
2750: {
2751:   HANDLE_TH_ERRORS
2752:   static PythonArgParser parser({
2753:     "silu(Tensor input, *, Tensor out=None)",
2754:   }, /*traceable=*/true);
2755: 
2756:   ParsedArgs<2> parsed_args;
2757:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2758:   if(_r.has_torch_function()) {
2759:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
2760:   }
```

- EN: The main execution path in this span is carried by `THPVariable_scaled_dot_product_attention`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_scaled_dot_product_attention`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2761-2800

```cpp
2761:   if (_r.isNone(1)) {
2762:     // aten::silu(Tensor self) -> Tensor
2763: 
2764:     auto dispatch_silu = [](const at::Tensor & self) -> at::Tensor {
2765:       pybind11::gil_scoped_release no_gil;
2766:       return at::silu(self);
2767:     };
2768:     return wrap(dispatch_silu(_r.tensor(0)));
2769:   } else {
2770:     // aten::silu.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
2771: 
2772:     auto dispatch_silu_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
2773:       pybind11::gil_scoped_release no_gil;
2774:       return at::silu_out(out, self);
2775:     };
2776:     return wrap(dispatch_silu_out(_r.tensor(1), _r.tensor(0)));
2777:   }
2778:   Py_RETURN_NONE;
2779:   END_HANDLE_TH_ERRORS
2780: }
2781: 
2782: // silu_
2783: static PyObject * THPVariable_silu_(PyObject* self_, PyObject* args, PyObject* kwargs)
2784: {
2785:   HANDLE_TH_ERRORS
2786:   static PythonArgParser parser({
2787:     "silu_(Tensor input)",
2788:   }, /*traceable=*/true);
2789: 
2790:   ParsedArgs<1> parsed_args;
2791:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2792:   if(_r.has_torch_function()) {
2793:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
2794:   }
2795:   // aten::silu_(Tensor(a!) self) -> Tensor(a!)
2796: 
2797:   auto dispatch_silu_ = [](at::Tensor self) -> at::Tensor {
2798:     pybind11::gil_scoped_release no_gil;
2799:     return at::silu_(self);
2800:   };
```

- EN: The main execution path in this span is carried by `silu`, `wrap`, `silu_out`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `silu`, `wrap`, `silu_out` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2801-2840

```cpp
2801:   return wrap(dispatch_silu_(_r.tensor(0)));
2802:   Py_RETURN_NONE;
2803:   END_HANDLE_TH_ERRORS
2804: }
2805: 
2806: // slow_conv3d
2807: static PyObject * THPVariable_slow_conv3d(PyObject* self_, PyObject* args, PyObject* kwargs)
2808: {
2809:   HANDLE_TH_ERRORS
2810:   static PythonArgParser parser({
2811:     "slow_conv3d(Tensor input, Tensor weight, SymIntArrayRef[3] kernel_size, Tensor? bias=None, SymIntArrayRef[3] stride=1, SymIntArrayRef[3] padding=0, *, Tensor out=None)",
2812:   }, /*traceable=*/true);
2813: 
2814:   ParsedArgs<7> parsed_args;
2815:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2816:   if(_r.has_torch_function()) {
2817:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
2818:   }
2819:   if (_r.isNone(6)) {
2820:     // aten::slow_conv3d(Tensor self, Tensor weight, SymInt[3] kernel_size, Tensor? bias=None, SymInt[3] stride=1, SymInt[3] padding=0) -> Tensor
2821: 
2822:     auto dispatch_slow_conv3d = [](const at::Tensor & self, const at::Tensor & weight, c10::SymIntArrayRef kernel_size, const ::std::optional<at::Tensor> & bias, c10::SymIntArrayRef stride, c10::SymIntArrayRef padding) -> at::Tensor {
2823:       pybind11::gil_scoped_release no_gil;
2824:       return at::slow_conv3d_symint(self, weight, kernel_size, bias, stride, padding);
2825:     };
2826:     return wrap(dispatch_slow_conv3d(_r.tensor(0), _r.tensor(1), _r.symintlist(2), _r.optionalTensor(3), _r.symintlist(4), _r.symintlist(5)));
2827:   } else {
2828:     // aten::slow_conv3d.out(Tensor self, Tensor weight, SymInt[3] kernel_size, Tensor? bias=None, SymInt[3] stride=1, SymInt[3] padding=0, *, Tensor(a!) out) -> Tensor(a!)
2829: 
2830:     auto dispatch_slow_conv3d_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & weight, c10::SymIntArrayRef kernel_size, const ::std::optional<at::Tensor> & bias, c10::SymIntArrayRef stride, c10::SymIntArrayRef padding) -> at::Tensor {
2831:       pybind11::gil_scoped_release no_gil;
2832:       return at::slow_conv3d_symint_out(out, self, weight, kernel_size, bias, stride, padding);
2833:     };
2834:     return wrap(dispatch_slow_conv3d_out(_r.tensor(6), _r.tensor(0), _r.tensor(1), _r.symintlist(2), _r.optionalTensor(3), _r.symintlist(4), _r.symintlist(5)));
2835:   }
2836:   Py_RETURN_NONE;
2837:   END_HANDLE_TH_ERRORS
2838: }
2839: 
2840: // slow_conv_dilated2d
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_slow_conv3d`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_slow_conv3d`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 2841-2880

```cpp
2841: static PyObject * THPVariable_slow_conv_dilated2d(PyObject* self_, PyObject* args, PyObject* kwargs)
2842: {
2843:   HANDLE_TH_ERRORS
2844:   static PythonArgParser parser({
2845:     "slow_conv_dilated2d(Tensor input, Tensor weight, SymIntArrayRef[2] kernel_size, Tensor? bias=None, SymIntArrayRef[2] stride=1, SymIntArrayRef[2] padding=0, SymIntArrayRef[2] dilation=1)",
2846:   }, /*traceable=*/true);
2847: 
2848:   ParsedArgs<7> parsed_args;
2849:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2850:   if(_r.has_torch_function()) {
2851:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
2852:   }
2853:   // aten::slow_conv_dilated2d(Tensor self, Tensor weight, SymInt[2] kernel_size, Tensor? bias=None, SymInt[2] stride=1, SymInt[2] padding=0, SymInt[2] dilation=1) -> Tensor
2854: 
2855:   auto dispatch_slow_conv_dilated2d = [](const at::Tensor & self, const at::Tensor & weight, c10::SymIntArrayRef kernel_size, const ::std::optional<at::Tensor> & bias, c10::SymIntArrayRef stride, c10::SymIntArrayRef padding, c10::SymIntArrayRef dilation) -> at::Tensor {
2856:     pybind11::gil_scoped_release no_gil;
2857:     return at::slow_conv_dilated2d_symint(self, weight, kernel_size, bias, stride, padding, dilation);
2858:   };
2859:   return wrap(dispatch_slow_conv_dilated2d(_r.tensor(0), _r.tensor(1), _r.symintlist(2), _r.optionalTensor(3), _r.symintlist(4), _r.symintlist(5), _r.symintlist(6)));
2860:   Py_RETURN_NONE;
2861:   END_HANDLE_TH_ERRORS
2862: }
2863: 
2864: // slow_conv_dilated3d
2865: static PyObject * THPVariable_slow_conv_dilated3d(PyObject* self_, PyObject* args, PyObject* kwargs)
2866: {
2867:   HANDLE_TH_ERRORS
2868:   static PythonArgParser parser({
2869:     "slow_conv_dilated3d(Tensor input, Tensor weight, SymIntArrayRef[3] kernel_size, Tensor? bias=None, SymIntArrayRef[3] stride=1, SymIntArrayRef[3] padding=0, SymIntArrayRef[3] dilation=1)",
2870:   }, /*traceable=*/true);
2871: 
2872:   ParsedArgs<7> parsed_args;
2873:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2874:   if(_r.has_torch_function()) {
2875:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
2876:   }
2877:   // aten::slow_conv_dilated3d(Tensor self, Tensor weight, SymInt[3] kernel_size, Tensor? bias=None, SymInt[3] stride=1, SymInt[3] padding=0, SymInt[3] dilation=1) -> Tensor
2878: 
2879:   auto dispatch_slow_conv_dilated3d = [](const at::Tensor & self, const at::Tensor & weight, c10::SymIntArrayRef kernel_size, const ::std::optional<at::Tensor> & bias, c10::SymIntArrayRef stride, c10::SymIntArrayRef padding, c10::SymIntArrayRef dilation) -> at::Tensor {
2880:     pybind11::gil_scoped_release no_gil;
```

- EN: The main execution path in this span is carried by `THPVariable_slow_conv_dilated2d`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_slow_conv_dilated2d`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 2881-2920

```cpp
2881:     return at::slow_conv_dilated3d_symint(self, weight, kernel_size, bias, stride, padding, dilation);
2882:   };
2883:   return wrap(dispatch_slow_conv_dilated3d(_r.tensor(0), _r.tensor(1), _r.symintlist(2), _r.optionalTensor(3), _r.symintlist(4), _r.symintlist(5), _r.symintlist(6)));
2884:   Py_RETURN_NONE;
2885:   END_HANDLE_TH_ERRORS
2886: }
2887: 
2888: // slow_conv_transpose2d
2889: static PyObject * THPVariable_slow_conv_transpose2d(PyObject* self_, PyObject* args, PyObject* kwargs)
2890: {
2891:   HANDLE_TH_ERRORS
2892:   static PythonArgParser parser({
2893:     "slow_conv_transpose2d(Tensor input, Tensor weight, SymIntArrayRef[2] kernel_size, Tensor? bias=None, SymIntArrayRef[2] stride=1, SymIntArrayRef[2] padding=0, SymIntArrayRef[2] output_padding=0, SymIntArrayRef[2] dilation=1, *, Tensor out=None)",
2894:   }, /*traceable=*/true);
2895: 
2896:   ParsedArgs<9> parsed_args;
2897:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2898:   if(_r.has_torch_function()) {
2899:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
2900:   }
2901:   if (_r.isNone(8)) {
2902:     // aten::slow_conv_transpose2d(Tensor self, Tensor weight, SymInt[2] kernel_size, Tensor? bias=None, SymInt[2] stride=1, SymInt[2] padding=0, SymInt[2] output_padding=0, SymInt[2] dilation=1) -> Tensor
2903: 
2904:     auto dispatch_slow_conv_transpose2d = [](const at::Tensor & self, const at::Tensor & weight, c10::SymIntArrayRef kernel_size, const ::std::optional<at::Tensor> & bias, c10::SymIntArrayRef stride, c10::SymIntArrayRef padding, c10::SymIntArrayRef output_padding, c10::SymIntArrayRef dilation) -> at::Tensor {
2905:       pybind11::gil_scoped_release no_gil;
2906:       return at::slow_conv_transpose2d_symint(self, weight, kernel_size, bias, stride, padding, output_padding, dilation);
2907:     };
2908:     return wrap(dispatch_slow_conv_transpose2d(_r.tensor(0), _r.tensor(1), _r.symintlist(2), _r.optionalTensor(3), _r.symintlist(4), _r.symintlist(5), _r.symintlist(6), _r.symintlist(7)));
2909:   } else {
2910:     // aten::slow_conv_transpose2d.out(Tensor self, Tensor weight, SymInt[2] kernel_size, Tensor? bias=None, SymInt[2] stride=1, SymInt[2] padding=0, SymInt[2] output_padding=0, SymInt[2] dilation=1, *, Tensor(a!) out) -> Tensor(a!)
2911: 
2912:     auto dispatch_slow_conv_transpose2d_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & weight, c10::SymIntArrayRef kernel_size, const ::std::optional<at::Tensor> & bias, c10::SymIntArrayRef stride, c10::SymIntArrayRef padding, c10::SymIntArrayRef output_padding, c10::SymIntArrayRef dilation) -> at::Tensor {
2913:       pybind11::gil_scoped_release no_gil;
2914:       return at::slow_conv_transpose2d_symint_out(out, self, weight, kernel_size, bias, stride, padding, output_padding, dilation);
2915:     };
2916:     return wrap(dispatch_slow_conv_transpose2d_out(_r.tensor(8), _r.tensor(0), _r.tensor(1), _r.symintlist(2), _r.optionalTensor(3), _r.symintlist(4), _r.symintlist(5), _r.symintlist(6), _r.symintlist(7)));
2917:   }
2918:   Py_RETURN_NONE;
2919:   END_HANDLE_TH_ERRORS
2920: }
```

- EN: The main execution path in this span is carried by `slow_conv_dilated3d_symint`, `wrap`, `THPVariable_slow_conv_transpose2d`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `slow_conv_dilated3d_symint`, `wrap`, `THPVariable_slow_conv_transpose2d` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 2921-2960

```cpp
2921: 
2922: // slow_conv_transpose3d
2923: static PyObject * THPVariable_slow_conv_transpose3d(PyObject* self_, PyObject* args, PyObject* kwargs)
2924: {
2925:   HANDLE_TH_ERRORS
2926:   static PythonArgParser parser({
2927:     "slow_conv_transpose3d(Tensor input, Tensor weight, SymIntArrayRef[3] kernel_size, Tensor? bias=None, SymIntArrayRef[3] stride=1, SymIntArrayRef[3] padding=0, SymIntArrayRef[3] output_padding=0, SymIntArrayRef[3] dilation=1, *, Tensor out=None)",
2928:   }, /*traceable=*/true);
2929: 
2930:   ParsedArgs<9> parsed_args;
2931:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2932:   if(_r.has_torch_function()) {
2933:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
2934:   }
2935:   if (_r.isNone(8)) {
2936:     // aten::slow_conv_transpose3d(Tensor self, Tensor weight, SymInt[3] kernel_size, Tensor? bias=None, SymInt[3] stride=1, SymInt[3] padding=0, SymInt[3] output_padding=0, SymInt[3] dilation=1) -> Tensor
2937: 
2938:     auto dispatch_slow_conv_transpose3d = [](const at::Tensor & self, const at::Tensor & weight, c10::SymIntArrayRef kernel_size, const ::std::optional<at::Tensor> & bias, c10::SymIntArrayRef stride, c10::SymIntArrayRef padding, c10::SymIntArrayRef output_padding, c10::SymIntArrayRef dilation) -> at::Tensor {
2939:       pybind11::gil_scoped_release no_gil;
2940:       return at::slow_conv_transpose3d_symint(self, weight, kernel_size, bias, stride, padding, output_padding, dilation);
2941:     };
2942:     return wrap(dispatch_slow_conv_transpose3d(_r.tensor(0), _r.tensor(1), _r.symintlist(2), _r.optionalTensor(3), _r.symintlist(4), _r.symintlist(5), _r.symintlist(6), _r.symintlist(7)));
2943:   } else {
2944:     // aten::slow_conv_transpose3d.out(Tensor self, Tensor weight, SymInt[3] kernel_size, Tensor? bias=None, SymInt[3] stride=1, SymInt[3] padding=0, SymInt[3] output_padding=0, SymInt[3] dilation=1, *, Tensor(a!) out) -> Tensor(a!)
2945: 
2946:     auto dispatch_slow_conv_transpose3d_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & weight, c10::SymIntArrayRef kernel_size, const ::std::optional<at::Tensor> & bias, c10::SymIntArrayRef stride, c10::SymIntArrayRef padding, c10::SymIntArrayRef output_padding, c10::SymIntArrayRef dilation) -> at::Tensor {
2947:       pybind11::gil_scoped_release no_gil;
2948:       return at::slow_conv_transpose3d_symint_out(out, self, weight, kernel_size, bias, stride, padding, output_padding, dilation);
2949:     };
2950:     return wrap(dispatch_slow_conv_transpose3d_out(_r.tensor(8), _r.tensor(0), _r.tensor(1), _r.symintlist(2), _r.optionalTensor(3), _r.symintlist(4), _r.symintlist(5), _r.symintlist(6), _r.symintlist(7)));
2951:   }
2952:   Py_RETURN_NONE;
2953:   END_HANDLE_TH_ERRORS
2954: }
2955: 
2956: // smooth_l1_loss
2957: static PyObject * THPVariable_smooth_l1_loss(PyObject* self_, PyObject* args, PyObject* kwargs)
2958: {
2959:   HANDLE_TH_ERRORS
2960:   static PythonArgParser parser({
```

- EN: The main execution path in this span is carried by `THPVariable_slow_conv_transpose3d`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_slow_conv_transpose3d`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 2961-3000

```cpp
2961:     "smooth_l1_loss(Tensor input, Tensor target, int64_t reduction=at::Reduction::Mean, double beta=1.0, *, Tensor out=None)",
2962:   }, /*traceable=*/true);
2963: 
2964:   ParsedArgs<5> parsed_args;
2965:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2966:   if(_r.has_torch_function()) {
2967:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
2968:   }
2969:   if (_r.isNone(4)) {
2970:     // aten::smooth_l1_loss(Tensor self, Tensor target, int reduction=Mean, float beta=1.0) -> Tensor
2971: 
2972:     auto dispatch_smooth_l1_loss = [](const at::Tensor & self, const at::Tensor & target, int64_t reduction, double beta) -> at::Tensor {
2973:       pybind11::gil_scoped_release no_gil;
2974:       return at::smooth_l1_loss(self, target, reduction, beta);
2975:     };
2976:     return wrap(dispatch_smooth_l1_loss(_r.tensor(0), _r.tensor(1), _r.toInt64(2), _r.toDouble(3)));
2977:   } else {
2978:     // aten::smooth_l1_loss.out(Tensor self, Tensor target, int reduction=Mean, float beta=1.0, *, Tensor(a!) out) -> Tensor(a!)
2979: 
2980:     auto dispatch_smooth_l1_loss_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & target, int64_t reduction, double beta) -> at::Tensor {
2981:       pybind11::gil_scoped_release no_gil;
2982:       return at::smooth_l1_loss_out(out, self, target, reduction, beta);
2983:     };
2984:     return wrap(dispatch_smooth_l1_loss_out(_r.tensor(4), _r.tensor(0), _r.tensor(1), _r.toInt64(2), _r.toDouble(3)));
2985:   }
2986:   Py_RETURN_NONE;
2987:   END_HANDLE_TH_ERRORS
2988: }
2989: 
2990: // soft_margin_loss
2991: static PyObject * THPVariable_soft_margin_loss(PyObject* self_, PyObject* args, PyObject* kwargs)
2992: {
2993:   HANDLE_TH_ERRORS
2994:   static PythonArgParser parser({
2995:     "soft_margin_loss(Tensor input, Tensor target, int64_t reduction=at::Reduction::Mean, *, Tensor out=None)",
2996:   }, /*traceable=*/true);
2997: 
2998:   ParsedArgs<4> parsed_args;
2999:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3000:   if(_r.has_torch_function()) {
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `smooth_l1_loss`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `smooth_l1_loss`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3001-3040

```cpp
3001:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
3002:   }
3003:   if (_r.isNone(3)) {
3004:     // aten::soft_margin_loss(Tensor self, Tensor target, int reduction=Mean) -> Tensor
3005: 
3006:     auto dispatch_soft_margin_loss = [](const at::Tensor & self, const at::Tensor & target, int64_t reduction) -> at::Tensor {
3007:       pybind11::gil_scoped_release no_gil;
3008:       return at::soft_margin_loss(self, target, reduction);
3009:     };
3010:     return wrap(dispatch_soft_margin_loss(_r.tensor(0), _r.tensor(1), _r.toInt64(2)));
3011:   } else {
3012:     // aten::soft_margin_loss.out(Tensor self, Tensor target, int reduction=Mean, *, Tensor(a!) out) -> Tensor(a!)
3013: 
3014:     auto dispatch_soft_margin_loss_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & target, int64_t reduction) -> at::Tensor {
3015:       pybind11::gil_scoped_release no_gil;
3016:       return at::soft_margin_loss_out(out, self, target, reduction);
3017:     };
3018:     return wrap(dispatch_soft_margin_loss_out(_r.tensor(3), _r.tensor(0), _r.tensor(1), _r.toInt64(2)));
3019:   }
3020:   Py_RETURN_NONE;
3021:   END_HANDLE_TH_ERRORS
3022: }
3023: 
3024: // softplus
3025: static PyObject * THPVariable_softplus(PyObject* self_, PyObject* args, PyObject* kwargs)
3026: {
3027:   HANDLE_TH_ERRORS
3028:   static PythonArgParser parser({
3029:     "softplus(Tensor input, Scalar beta=1, Scalar threshold=20, *, Tensor out=None)",
3030:   }, /*traceable=*/true);
3031: 
3032:   ParsedArgs<4> parsed_args;
3033:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3034:   if(_r.has_torch_function()) {
3035:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
3036:   }
3037:   if (_r.isNone(3)) {
3038:     // aten::softplus(Tensor self, Scalar beta=1, Scalar threshold=20) -> Tensor
3039: 
3040:     auto dispatch_softplus = [](const at::Tensor & self, const at::Scalar & beta, const at::Scalar & threshold) -> at::Tensor {
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `soft_margin_loss`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `soft_margin_loss`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3041-3080

```cpp
3041:       pybind11::gil_scoped_release no_gil;
3042:       return at::softplus(self, beta, threshold);
3043:     };
3044:     return wrap(dispatch_softplus(_r.tensor(0), _r.scalar(1), _r.scalar(2)));
3045:   } else {
3046:     // aten::softplus.out(Tensor self, Scalar beta=1, Scalar threshold=20, *, Tensor(a!) out) -> Tensor(a!)
3047: 
3048:     auto dispatch_softplus_out = [](at::Tensor out, const at::Tensor & self, const at::Scalar & beta, const at::Scalar & threshold) -> at::Tensor {
3049:       pybind11::gil_scoped_release no_gil;
3050:       return at::softplus_out(out, self, beta, threshold);
3051:     };
3052:     return wrap(dispatch_softplus_out(_r.tensor(3), _r.tensor(0), _r.scalar(1), _r.scalar(2)));
3053:   }
3054:   Py_RETURN_NONE;
3055:   END_HANDLE_TH_ERRORS
3056: }
3057: 
3058: // softshrink
3059: static PyObject * THPVariable_softshrink(PyObject* self_, PyObject* args, PyObject* kwargs)
3060: {
3061:   HANDLE_TH_ERRORS
3062:   static PythonArgParser parser({
3063:     "softshrink(Tensor input, Scalar lambd=0.5, *, Tensor out=None)",
3064:   }, /*traceable=*/true);
3065: 
3066:   ParsedArgs<3> parsed_args;
3067:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3068:   if(_r.has_torch_function()) {
3069:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
3070:   }
3071:   if (_r.isNone(2)) {
3072:     // aten::softshrink(Tensor self, Scalar lambd=0.5) -> Tensor
3073: 
3074:     auto dispatch_softshrink = [](const at::Tensor & self, const at::Scalar & lambd) -> at::Tensor {
3075:       pybind11::gil_scoped_release no_gil;
3076:       return at::softshrink(self, lambd);
3077:     };
3078:     return wrap(dispatch_softshrink(_r.tensor(0), _r.scalar(1)));
3079:   } else {
3080:     // aten::softshrink.out(Tensor self, Scalar lambd=0.5, *, Tensor(a!) out) -> Tensor(a!)
```

- EN: The main execution path in this span is carried by `softplus`, `wrap`, `softplus_out`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `softplus`, `wrap`, `softplus_out` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3081-3120

```cpp
3081: 
3082:     auto dispatch_softshrink_out = [](at::Tensor out, const at::Tensor & self, const at::Scalar & lambd) -> at::Tensor {
3083:       pybind11::gil_scoped_release no_gil;
3084:       return at::softshrink_out(out, self, lambd);
3085:     };
3086:     return wrap(dispatch_softshrink_out(_r.tensor(2), _r.tensor(0), _r.scalar(1)));
3087:   }
3088:   Py_RETURN_NONE;
3089:   END_HANDLE_TH_ERRORS
3090: }
3091: 
3092: // thnn_conv2d
3093: static PyObject * THPVariable_thnn_conv2d(PyObject* self_, PyObject* args, PyObject* kwargs)
3094: {
3095:   HANDLE_TH_ERRORS
3096:   static PythonArgParser parser({
3097:     "thnn_conv2d(Tensor input, Tensor weight, SymIntArrayRef[2] kernel_size, Tensor? bias=None, SymIntArrayRef[2] stride=1, SymIntArrayRef[2] padding=0, *, Tensor out=None)",
3098:   }, /*traceable=*/true);
3099: 
3100:   ParsedArgs<7> parsed_args;
3101:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3102:   if(_r.has_torch_function()) {
3103:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
3104:   }
3105:   if (_r.isNone(6)) {
3106:     // aten::thnn_conv2d(Tensor self, Tensor weight, SymInt[2] kernel_size, Tensor? bias=None, SymInt[2] stride=1, SymInt[2] padding=0) -> Tensor
3107: 
3108:     auto dispatch_thnn_conv2d = [](const at::Tensor & self, const at::Tensor & weight, c10::SymIntArrayRef kernel_size, const ::std::optional<at::Tensor> & bias, c10::SymIntArrayRef stride, c10::SymIntArrayRef padding) -> at::Tensor {
3109:       pybind11::gil_scoped_release no_gil;
3110:       return at::thnn_conv2d_symint(self, weight, kernel_size, bias, stride, padding);
3111:     };
3112:     return wrap(dispatch_thnn_conv2d(_r.tensor(0), _r.tensor(1), _r.symintlist(2), _r.optionalTensor(3), _r.symintlist(4), _r.symintlist(5)));
3113:   } else {
3114:     // aten::thnn_conv2d.out(Tensor self, Tensor weight, SymInt[2] kernel_size, Tensor? bias=None, SymInt[2] stride=1, SymInt[2] padding=0, *, Tensor(a!) out) -> Tensor(a!)
3115: 
3116:     auto dispatch_thnn_conv2d_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & weight, c10::SymIntArrayRef kernel_size, const ::std::optional<at::Tensor> & bias, c10::SymIntArrayRef stride, c10::SymIntArrayRef padding) -> at::Tensor {
3117:       pybind11::gil_scoped_release no_gil;
3118:       return at::thnn_conv2d_symint_out(out, self, weight, kernel_size, bias, stride, padding);
3119:     };
3120:     return wrap(dispatch_thnn_conv2d_out(_r.tensor(6), _r.tensor(0), _r.tensor(1), _r.symintlist(2), _r.optionalTensor(3), _r.symintlist(4), _r.symintlist(5)));
```

- EN: The main execution path in this span is carried by `softshrink_out`, `wrap`, `THPVariable_thnn_conv2d`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `softshrink_out`, `wrap`, `THPVariable_thnn_conv2d` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 3121-3160

```cpp
3121:   }
3122:   Py_RETURN_NONE;
3123:   END_HANDLE_TH_ERRORS
3124: }
3125: 
3126: // unflatten_dense_tensors
3127: static PyObject * THPVariable_unflatten_dense_tensors(PyObject* self_, PyObject* args, PyObject* kwargs)
3128: {
3129:   HANDLE_TH_ERRORS
3130:   static PythonArgParser parser({
3131:     "unflatten_dense_tensors(Tensor flat, TensorList tensors)",
3132:   }, /*traceable=*/true);
3133: 
3134:   ParsedArgs<2> parsed_args;
3135:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3136:   if(_r.has_torch_function()) {
3137:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
3138:   }
3139:   // aten::unflatten_dense_tensors(Tensor flat, Tensor[] tensors) -> Tensor[]
3140: 
3141:   auto dispatch_unflatten_dense_tensors = [](const at::Tensor & flat, at::TensorList tensors) -> ::std::vector<at::Tensor> {
3142:     pybind11::gil_scoped_release no_gil;
3143:     return at::unflatten_dense_tensors(flat, tensors);
3144:   };
3145:   return wrap(dispatch_unflatten_dense_tensors(_r.tensor(0), _r.tensorlist(1)));
3146:   Py_RETURN_NONE;
3147:   END_HANDLE_TH_ERRORS
3148: }
3149: 
3150: \
3151: // upsample_bicubic2d
3152: static PyObject * THPVariable_upsample_bicubic2d(PyObject* self_, PyObject* args, PyObject* kwargs)
3153: {
3154:   HANDLE_TH_ERRORS
3155:   static PythonArgParser parser({
3156:     "upsample_bicubic2d(Tensor input, SymIntArrayRef? output_size, bool align_corners, ArrayRef<double>? scale_factors)",
3157:     "upsample_bicubic2d(Tensor input, SymIntArrayRef[2] output_size, bool align_corners, double? scales_h=None, double? scales_w=None, *, Tensor out=None)",
3158:   }, /*traceable=*/true);
3159: 
3160:   ParsedArgs<6> parsed_args;
```

- EN: The main execution path in this span is carried by `THPVariable_unflatten_dense_tensors`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_unflatten_dense_tensors`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 3161-3200

```cpp
3161:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3162:   if(_r.has_torch_function()) {
3163:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
3164:   }
3165:   switch (_r.idx) {
3166:     case 0: {
3167:       // aten::upsample_bicubic2d.vec(Tensor input, SymInt[]? output_size, bool align_corners, float[]? scale_factors) -> Tensor
3168: 
3169:       auto dispatch_upsample_bicubic2d = [](const at::Tensor & input, at::OptionalSymIntArrayRef output_size, bool align_corners, ::std::optional<at::ArrayRef<double>> scale_factors) -> at::Tensor {
3170:         pybind11::gil_scoped_release no_gil;
3171:         return at::upsample_bicubic2d_symint(input, output_size, align_corners, scale_factors);
3172:       };
3173:       return wrap(dispatch_upsample_bicubic2d(_r.tensor(0), _r.symintlistOptional(1), _r.toBool(2), _r.doublelistOptional(3)));
3174:     }
3175:     case 1: {
3176:       if (_r.isNone(5)) {
3177:         // aten::upsample_bicubic2d(Tensor self, SymInt[2] output_size, bool align_corners, float? scales_h=None, float? scales_w=None) -> Tensor
3178: 
3179:         auto dispatch_upsample_bicubic2d = [](const at::Tensor & self, c10::SymIntArrayRef output_size, bool align_corners, ::std::optional<double> scales_h, ::std::optional<double> scales_w) -> at::Tensor {
3180:           pybind11::gil_scoped_release no_gil;
3181:           return at::upsample_bicubic2d_symint(self, output_size, align_corners, scales_h, scales_w);
3182:         };
3183:         return wrap(dispatch_upsample_bicubic2d(_r.tensor(0), _r.symintlist(1), _r.toBool(2), _r.toDoubleOptional(3), _r.toDoubleOptional(4)));
3184:       } else {
3185:         // aten::upsample_bicubic2d.out(Tensor self, SymInt[2] output_size, bool align_corners, float? scales_h=None, float? scales_w=None, *, Tensor(a!) out) -> Tensor(a!)
3186: 
3187:         auto dispatch_upsample_bicubic2d_out = [](at::Tensor out, const at::Tensor & self, c10::SymIntArrayRef output_size, bool align_corners, ::std::optional<double> scales_h, ::std::optional<double> scales_w) -> at::Tensor {
3188:           pybind11::gil_scoped_release no_gil;
3189:           return at::upsample_bicubic2d_symint_out(out, self, output_size, align_corners, scales_h, scales_w);
3190:         };
3191:         return wrap(dispatch_upsample_bicubic2d_out(_r.tensor(5), _r.tensor(0), _r.symintlist(1), _r.toBool(2), _r.toDoubleOptional(3), _r.toDoubleOptional(4)));
3192:       }
3193:     }
3194:   }
3195:   Py_RETURN_NONE;
3196:   END_HANDLE_TH_ERRORS
3197: }
3198: 
3199: \
3200: // upsample_bilinear2d
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `upsample_bicubic2d_symint`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `upsample_bicubic2d_symint`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 3201-3240

```cpp
3201: static PyObject * THPVariable_upsample_bilinear2d(PyObject* self_, PyObject* args, PyObject* kwargs)
3202: {
3203:   HANDLE_TH_ERRORS
3204:   static PythonArgParser parser({
3205:     "upsample_bilinear2d(Tensor input, SymIntArrayRef? output_size, bool align_corners, ArrayRef<double>? scale_factors)",
3206:     "upsample_bilinear2d(Tensor input, SymIntArrayRef[2] output_size, bool align_corners, double? scales_h=None, double? scales_w=None, *, Tensor out=None)",
3207:   }, /*traceable=*/true);
3208: 
3209:   ParsedArgs<6> parsed_args;
3210:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3211:   if(_r.has_torch_function()) {
3212:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
3213:   }
3214:   switch (_r.idx) {
3215:     case 0: {
3216:       // aten::upsample_bilinear2d.vec(Tensor input, SymInt[]? output_size, bool align_corners, float[]? scale_factors) -> Tensor
3217: 
3218:       auto dispatch_upsample_bilinear2d = [](const at::Tensor & input, at::OptionalSymIntArrayRef output_size, bool align_corners, ::std::optional<at::ArrayRef<double>> scale_factors) -> at::Tensor {
3219:         pybind11::gil_scoped_release no_gil;
3220:         return at::upsample_bilinear2d_symint(input, output_size, align_corners, scale_factors);
3221:       };
3222:       return wrap(dispatch_upsample_bilinear2d(_r.tensor(0), _r.symintlistOptional(1), _r.toBool(2), _r.doublelistOptional(3)));
3223:     }
3224:     case 1: {
3225:       if (_r.isNone(5)) {
3226:         // aten::upsample_bilinear2d(Tensor self, SymInt[2] output_size, bool align_corners, float? scales_h=None, float? scales_w=None) -> Tensor
3227: 
3228:         auto dispatch_upsample_bilinear2d = [](const at::Tensor & self, c10::SymIntArrayRef output_size, bool align_corners, ::std::optional<double> scales_h, ::std::optional<double> scales_w) -> at::Tensor {
3229:           pybind11::gil_scoped_release no_gil;
3230:           return at::upsample_bilinear2d_symint(self, output_size, align_corners, scales_h, scales_w);
3231:         };
3232:         return wrap(dispatch_upsample_bilinear2d(_r.tensor(0), _r.symintlist(1), _r.toBool(2), _r.toDoubleOptional(3), _r.toDoubleOptional(4)));
3233:       } else {
3234:         // aten::upsample_bilinear2d.out(Tensor self, SymInt[2] output_size, bool align_corners, float? scales_h=None, float? scales_w=None, *, Tensor(a!) out) -> Tensor(a!)
3235: 
3236:         auto dispatch_upsample_bilinear2d_out = [](at::Tensor out, const at::Tensor & self, c10::SymIntArrayRef output_size, bool align_corners, ::std::optional<double> scales_h, ::std::optional<double> scales_w) -> at::Tensor {
3237:           pybind11::gil_scoped_release no_gil;
3238:           return at::upsample_bilinear2d_symint_out(out, self, output_size, align_corners, scales_h, scales_w);
3239:         };
3240:         return wrap(dispatch_upsample_bilinear2d_out(_r.tensor(5), _r.tensor(0), _r.symintlist(1), _r.toBool(2), _r.toDoubleOptional(3), _r.toDoubleOptional(4)));
```

- EN: The main execution path in this span is carried by `THPVariable_upsample_bilinear2d`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_upsample_bilinear2d`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 3241-3280

```cpp
3241:       }
3242:     }
3243:   }
3244:   Py_RETURN_NONE;
3245:   END_HANDLE_TH_ERRORS
3246: }
3247: 
3248: \
3249: // upsample_linear1d
3250: static PyObject * THPVariable_upsample_linear1d(PyObject* self_, PyObject* args, PyObject* kwargs)
3251: {
3252:   HANDLE_TH_ERRORS
3253:   static PythonArgParser parser({
3254:     "upsample_linear1d(Tensor input, SymIntArrayRef? output_size, bool align_corners, ArrayRef<double>? scale_factors)",
3255:     "upsample_linear1d(Tensor input, SymIntArrayRef[1] output_size, bool align_corners, double? scales=None, *, Tensor out=None)",
3256:   }, /*traceable=*/true);
3257: 
3258:   ParsedArgs<5> parsed_args;
3259:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3260:   if(_r.has_torch_function()) {
3261:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
3262:   }
3263:   switch (_r.idx) {
3264:     case 0: {
3265:       // aten::upsample_linear1d.vec(Tensor input, SymInt[]? output_size, bool align_corners, float[]? scale_factors) -> Tensor
3266: 
3267:       auto dispatch_upsample_linear1d = [](const at::Tensor & input, at::OptionalSymIntArrayRef output_size, bool align_corners, ::std::optional<at::ArrayRef<double>> scale_factors) -> at::Tensor {
3268:         pybind11::gil_scoped_release no_gil;
3269:         return at::upsample_linear1d_symint(input, output_size, align_corners, scale_factors);
3270:       };
3271:       return wrap(dispatch_upsample_linear1d(_r.tensor(0), _r.symintlistOptional(1), _r.toBool(2), _r.doublelistOptional(3)));
3272:     }
3273:     case 1: {
3274:       if (_r.isNone(4)) {
3275:         // aten::upsample_linear1d(Tensor self, SymInt[1] output_size, bool align_corners, float? scales=None) -> Tensor
3276: 
3277:         auto dispatch_upsample_linear1d = [](const at::Tensor & self, c10::SymIntArrayRef output_size, bool align_corners, ::std::optional<double> scales) -> at::Tensor {
3278:           pybind11::gil_scoped_release no_gil;
3279:           return at::upsample_linear1d_symint(self, output_size, align_corners, scales);
3280:         };
```

- EN: The main execution path in this span is carried by `THPVariable_upsample_linear1d`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_upsample_linear1d`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 3281-3320

```cpp
3281:         return wrap(dispatch_upsample_linear1d(_r.tensor(0), _r.symintlist(1), _r.toBool(2), _r.toDoubleOptional(3)));
3282:       } else {
3283:         // aten::upsample_linear1d.out(Tensor self, SymInt[1] output_size, bool align_corners, float? scales=None, *, Tensor(a!) out) -> Tensor(a!)
3284: 
3285:         auto dispatch_upsample_linear1d_out = [](at::Tensor out, const at::Tensor & self, c10::SymIntArrayRef output_size, bool align_corners, ::std::optional<double> scales) -> at::Tensor {
3286:           pybind11::gil_scoped_release no_gil;
3287:           return at::upsample_linear1d_symint_out(out, self, output_size, align_corners, scales);
3288:         };
3289:         return wrap(dispatch_upsample_linear1d_out(_r.tensor(4), _r.tensor(0), _r.symintlist(1), _r.toBool(2), _r.toDoubleOptional(3)));
3290:       }
3291:     }
3292:   }
3293:   Py_RETURN_NONE;
3294:   END_HANDLE_TH_ERRORS
3295: }
3296: 
3297: \
3298: // upsample_nearest1d
3299: static PyObject * THPVariable_upsample_nearest1d(PyObject* self_, PyObject* args, PyObject* kwargs)
3300: {
3301:   HANDLE_TH_ERRORS
3302:   static PythonArgParser parser({
3303:     "upsample_nearest1d(Tensor input, SymIntArrayRef? output_size, ArrayRef<double>? scale_factors)",
3304:     "upsample_nearest1d(Tensor input, SymIntArrayRef[1] output_size, double? scales=None, *, Tensor out=None)",
3305:   }, /*traceable=*/true);
3306: 
3307:   ParsedArgs<4> parsed_args;
3308:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3309:   if(_r.has_torch_function()) {
3310:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
3311:   }
3312:   switch (_r.idx) {
3313:     case 0: {
3314:       // aten::upsample_nearest1d.vec(Tensor input, SymInt[]? output_size, float[]? scale_factors) -> Tensor
3315: 
3316:       auto dispatch_upsample_nearest1d = [](const at::Tensor & input, at::OptionalSymIntArrayRef output_size, ::std::optional<at::ArrayRef<double>> scale_factors) -> at::Tensor {
3317:         pybind11::gil_scoped_release no_gil;
3318:         return at::upsample_nearest1d_symint(input, output_size, scale_factors);
3319:       };
3320:       return wrap(dispatch_upsample_nearest1d(_r.tensor(0), _r.symintlistOptional(1), _r.doublelistOptional(2)));
```

- EN: The main execution path in this span is carried by `wrap`, `upsample_linear1d_symint_out`, `THPVariable_upsample_nearest1d`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `wrap`, `upsample_linear1d_symint_out`, `THPVariable_upsample_nearest1d` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 3321-3360

```cpp
3321:     }
3322:     case 1: {
3323:       if (_r.isNone(3)) {
3324:         // aten::upsample_nearest1d(Tensor self, SymInt[1] output_size, float? scales=None) -> Tensor
3325: 
3326:         auto dispatch_upsample_nearest1d = [](const at::Tensor & self, c10::SymIntArrayRef output_size, ::std::optional<double> scales) -> at::Tensor {
3327:           pybind11::gil_scoped_release no_gil;
3328:           return at::upsample_nearest1d_symint(self, output_size, scales);
3329:         };
3330:         return wrap(dispatch_upsample_nearest1d(_r.tensor(0), _r.symintlist(1), _r.toDoubleOptional(2)));
3331:       } else {
3332:         // aten::upsample_nearest1d.out(Tensor self, SymInt[1] output_size, float? scales=None, *, Tensor(a!) out) -> Tensor(a!)
3333: 
3334:         auto dispatch_upsample_nearest1d_out = [](at::Tensor out, const at::Tensor & self, c10::SymIntArrayRef output_size, ::std::optional<double> scales) -> at::Tensor {
3335:           pybind11::gil_scoped_release no_gil;
3336:           return at::upsample_nearest1d_symint_out(out, self, output_size, scales);
3337:         };
3338:         return wrap(dispatch_upsample_nearest1d_out(_r.tensor(3), _r.tensor(0), _r.symintlist(1), _r.toDoubleOptional(2)));
3339:       }
3340:     }
3341:   }
3342:   Py_RETURN_NONE;
3343:   END_HANDLE_TH_ERRORS
3344: }
3345: 
3346: \
3347: // upsample_nearest2d
3348: static PyObject * THPVariable_upsample_nearest2d(PyObject* self_, PyObject* args, PyObject* kwargs)
3349: {
3350:   HANDLE_TH_ERRORS
3351:   static PythonArgParser parser({
3352:     "upsample_nearest2d(Tensor input, SymIntArrayRef? output_size, ArrayRef<double>? scale_factors)",
3353:     "upsample_nearest2d(Tensor input, SymIntArrayRef[2] output_size, double? scales_h=None, double? scales_w=None, *, Tensor out=None)",
3354:   }, /*traceable=*/true);
3355: 
3356:   ParsedArgs<5> parsed_args;
3357:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3358:   if(_r.has_torch_function()) {
3359:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
3360:   }
```

- EN: The main execution path in this span is carried by `upsample_nearest1d`, `upsample_nearest1d_symint`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `upsample_nearest1d`, `upsample_nearest1d_symint`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 3361-3400

```cpp
3361:   switch (_r.idx) {
3362:     case 0: {
3363:       // aten::upsample_nearest2d.vec(Tensor input, SymInt[]? output_size, float[]? scale_factors) -> Tensor
3364: 
3365:       auto dispatch_upsample_nearest2d = [](const at::Tensor & input, at::OptionalSymIntArrayRef output_size, ::std::optional<at::ArrayRef<double>> scale_factors) -> at::Tensor {
3366:         pybind11::gil_scoped_release no_gil;
3367:         return at::upsample_nearest2d_symint(input, output_size, scale_factors);
3368:       };
3369:       return wrap(dispatch_upsample_nearest2d(_r.tensor(0), _r.symintlistOptional(1), _r.doublelistOptional(2)));
3370:     }
3371:     case 1: {
3372:       if (_r.isNone(4)) {
3373:         // aten::upsample_nearest2d(Tensor self, SymInt[2] output_size, float? scales_h=None, float? scales_w=None) -> Tensor
3374: 
3375:         auto dispatch_upsample_nearest2d = [](const at::Tensor & self, c10::SymIntArrayRef output_size, ::std::optional<double> scales_h, ::std::optional<double> scales_w) -> at::Tensor {
3376:           pybind11::gil_scoped_release no_gil;
3377:           return at::upsample_nearest2d_symint(self, output_size, scales_h, scales_w);
3378:         };
3379:         return wrap(dispatch_upsample_nearest2d(_r.tensor(0), _r.symintlist(1), _r.toDoubleOptional(2), _r.toDoubleOptional(3)));
3380:       } else {
3381:         // aten::upsample_nearest2d.out(Tensor self, SymInt[2] output_size, float? scales_h=None, float? scales_w=None, *, Tensor(a!) out) -> Tensor(a!)
3382: 
3383:         auto dispatch_upsample_nearest2d_out = [](at::Tensor out, const at::Tensor & self, c10::SymIntArrayRef output_size, ::std::optional<double> scales_h, ::std::optional<double> scales_w) -> at::Tensor {
3384:           pybind11::gil_scoped_release no_gil;
3385:           return at::upsample_nearest2d_symint_out(out, self, output_size, scales_h, scales_w);
3386:         };
3387:         return wrap(dispatch_upsample_nearest2d_out(_r.tensor(4), _r.tensor(0), _r.symintlist(1), _r.toDoubleOptional(2), _r.toDoubleOptional(3)));
3388:       }
3389:     }
3390:   }
3391:   Py_RETURN_NONE;
3392:   END_HANDLE_TH_ERRORS
3393: }
3394: 
3395: \
3396: // upsample_nearest3d
3397: static PyObject * THPVariable_upsample_nearest3d(PyObject* self_, PyObject* args, PyObject* kwargs)
3398: {
3399:   HANDLE_TH_ERRORS
3400:   static PythonArgParser parser({
```

- EN: The main execution path in this span is carried by `upsample_nearest2d_symint`, `wrap`, `upsample_nearest2d`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `upsample_nearest2d_symint`, `wrap`, `upsample_nearest2d` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 3401-3440

```cpp
3401:     "upsample_nearest3d(Tensor input, SymIntArrayRef? output_size, ArrayRef<double>? scale_factors)",
3402:     "upsample_nearest3d(Tensor input, SymIntArrayRef[3] output_size, double? scales_d=None, double? scales_h=None, double? scales_w=None, *, Tensor out=None)",
3403:   }, /*traceable=*/true);
3404: 
3405:   ParsedArgs<6> parsed_args;
3406:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3407:   if(_r.has_torch_function()) {
3408:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
3409:   }
3410:   switch (_r.idx) {
3411:     case 0: {
3412:       // aten::upsample_nearest3d.vec(Tensor input, SymInt[]? output_size, float[]? scale_factors) -> Tensor
3413: 
3414:       auto dispatch_upsample_nearest3d = [](const at::Tensor & input, at::OptionalSymIntArrayRef output_size, ::std::optional<at::ArrayRef<double>> scale_factors) -> at::Tensor {
3415:         pybind11::gil_scoped_release no_gil;
3416:         return at::upsample_nearest3d_symint(input, output_size, scale_factors);
3417:       };
3418:       return wrap(dispatch_upsample_nearest3d(_r.tensor(0), _r.symintlistOptional(1), _r.doublelistOptional(2)));
3419:     }
3420:     case 1: {
3421:       if (_r.isNone(5)) {
3422:         // aten::upsample_nearest3d(Tensor self, SymInt[3] output_size, float? scales_d=None, float? scales_h=None, float? scales_w=None) -> Tensor
3423: 
3424:         auto dispatch_upsample_nearest3d = [](const at::Tensor & self, c10::SymIntArrayRef output_size, ::std::optional<double> scales_d, ::std::optional<double> scales_h, ::std::optional<double> scales_w) -> at::Tensor {
3425:           pybind11::gil_scoped_release no_gil;
3426:           return at::upsample_nearest3d_symint(self, output_size, scales_d, scales_h, scales_w);
3427:         };
3428:         return wrap(dispatch_upsample_nearest3d(_r.tensor(0), _r.symintlist(1), _r.toDoubleOptional(2), _r.toDoubleOptional(3), _r.toDoubleOptional(4)));
3429:       } else {
3430:         // aten::upsample_nearest3d.out(Tensor self, SymInt[3] output_size, float? scales_d=None, float? scales_h=None, float? scales_w=None, *, Tensor(a!) out) -> Tensor(a!)
3431: 
3432:         auto dispatch_upsample_nearest3d_out = [](at::Tensor out, const at::Tensor & self, c10::SymIntArrayRef output_size, ::std::optional<double> scales_d, ::std::optional<double> scales_h, ::std::optional<double> scales_w) -> at::Tensor {
3433:           pybind11::gil_scoped_release no_gil;
3434:           return at::upsample_nearest3d_symint_out(out, self, output_size, scales_d, scales_h, scales_w);
3435:         };
3436:         return wrap(dispatch_upsample_nearest3d_out(_r.tensor(5), _r.tensor(0), _r.symintlist(1), _r.toDoubleOptional(2), _r.toDoubleOptional(3), _r.toDoubleOptional(4)));
3437:       }
3438:     }
3439:   }
3440:   Py_RETURN_NONE;
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `upsample_nearest3d_symint`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `upsample_nearest3d_symint`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3441-3480

```cpp
3441:   END_HANDLE_TH_ERRORS
3442: }
3443: 
3444: \
3445: // upsample_trilinear3d
3446: static PyObject * THPVariable_upsample_trilinear3d(PyObject* self_, PyObject* args, PyObject* kwargs)
3447: {
3448:   HANDLE_TH_ERRORS
3449:   static PythonArgParser parser({
3450:     "upsample_trilinear3d(Tensor input, SymIntArrayRef? output_size, bool align_corners, ArrayRef<double>? scale_factors)",
3451:     "upsample_trilinear3d(Tensor input, SymIntArrayRef[3] output_size, bool align_corners, double? scales_d=None, double? scales_h=None, double? scales_w=None, *, Tensor out=None)",
3452:   }, /*traceable=*/true);
3453: 
3454:   ParsedArgs<7> parsed_args;
3455:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3456:   if(_r.has_torch_function()) {
3457:     return handle_torch_function(_r, nullptr, args, kwargs, THPNNVariableFunctionsModule, "torch.nn");
3458:   }
3459:   switch (_r.idx) {
3460:     case 0: {
3461:       // aten::upsample_trilinear3d.vec(Tensor input, SymInt[]? output_size, bool align_corners, float[]? scale_factors) -> Tensor
3462: 
3463:       auto dispatch_upsample_trilinear3d = [](const at::Tensor & input, at::OptionalSymIntArrayRef output_size, bool align_corners, ::std::optional<at::ArrayRef<double>> scale_factors) -> at::Tensor {
3464:         pybind11::gil_scoped_release no_gil;
3465:         return at::upsample_trilinear3d_symint(input, output_size, align_corners, scale_factors);
3466:       };
3467:       return wrap(dispatch_upsample_trilinear3d(_r.tensor(0), _r.symintlistOptional(1), _r.toBool(2), _r.doublelistOptional(3)));
3468:     }
3469:     case 1: {
3470:       if (_r.isNone(6)) {
3471:         // aten::upsample_trilinear3d(Tensor self, SymInt[3] output_size, bool align_corners, float? scales_d=None, float? scales_h=None, float? scales_w=None) -> Tensor
3472: 
3473:         auto dispatch_upsample_trilinear3d = [](const at::Tensor & self, c10::SymIntArrayRef output_size, bool align_corners, ::std::optional<double> scales_d, ::std::optional<double> scales_h, ::std::optional<double> scales_w) -> at::Tensor {
3474:           pybind11::gil_scoped_release no_gil;
3475:           return at::upsample_trilinear3d_symint(self, output_size, align_corners, scales_d, scales_h, scales_w);
3476:         };
3477:         return wrap(dispatch_upsample_trilinear3d(_r.tensor(0), _r.symintlist(1), _r.toBool(2), _r.toDoubleOptional(3), _r.toDoubleOptional(4), _r.toDoubleOptional(5)));
3478:       } else {
3479:         // aten::upsample_trilinear3d.out(Tensor self, SymInt[3] output_size, bool align_corners, float? scales_d=None, float? scales_h=None, float? scales_w=None, *, Tensor(a!) out) -> Tensor(a!)
3480: 
```

- EN: The main execution path in this span is carried by `THPVariable_upsample_trilinear3d`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_upsample_trilinear3d`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 3481-3493

```cpp
3481:         auto dispatch_upsample_trilinear3d_out = [](at::Tensor out, const at::Tensor & self, c10::SymIntArrayRef output_size, bool align_corners, ::std::optional<double> scales_d, ::std::optional<double> scales_h, ::std::optional<double> scales_w) -> at::Tensor {
3482:           pybind11::gil_scoped_release no_gil;
3483:           return at::upsample_trilinear3d_symint_out(out, self, output_size, align_corners, scales_d, scales_h, scales_w);
3484:         };
3485:         return wrap(dispatch_upsample_trilinear3d_out(_r.tensor(6), _r.tensor(0), _r.symintlist(1), _r.toBool(2), _r.toDoubleOptional(3), _r.toDoubleOptional(4), _r.toDoubleOptional(5)));
3486:       }
3487:     }
3488:   }
3489:   Py_RETURN_NONE;
3490:   END_HANDLE_TH_ERRORS
3491: }
3492: 
3493: } // namespace torch::autograd
```

- EN: They also open, refine, or close the relevant C++ namespace scope. The main execution path in this span is carried by `upsample_trilinear3d_symint_out`, `wrap`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 它们还会打开、细化或关闭相应的 C++ 命名空间作用域。 这一段的主要执行路径由 `upsample_trilinear3d_symint_out`, `wrap` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。

## Key Concepts / 关键概念
- Autograd graph and gradient metadata / 自动求导图与梯度元数据
- Generated code patterns / 生成代码模式
- Python/C++ binding boundary / Python/C++ 绑定边界
- Tensor/value representation management / 张量/数值表示管理
- Primary symbol `THPVariable__parse_to` / 核心符号 `THPVariable__parse_to`
- Primary symbol `handle_torch_function` / 核心符号 `handle_torch_function`

## Dependencies / 依赖关系
- Direct includes / 直接包含: `torch/csrc/Device.h`, `torch/csrc/DynamicTypes.h`, `torch/csrc/Exceptions.h`, `torch/csrc/autograd/python_nn_functions.h`, `torch/csrc/autograd/generated/python_return_types.h`, `torch/csrc/autograd/python_variable.h`, `torch/csrc/autograd/utils/wrap_outputs.h`, `torch/csrc/autograd/utils/python_arg_parsing.h`, `torch/csrc/utils/pycfunction_helpers.h`, `torch/csrc/utils/python_arg_parser.h`
- Include roots / 头文件根模块: `ATen`, `torch`
- Key symbols / 关键符号: `THPVariable__parse_to`, `handle_torch_function`, `THPVariable__conv_depthwise2d`, `THPVariable__pad_circular`, `THPVariable__pad_enum`, `THPVariable__test_ambiguous_defaults`, `THPVariable__test_optional_filled_intlist`, `THPVariable__test_optional_floatlist`, `THPVariable__test_optional_intlist`, `THPVariable__test_string_default`
- Related subsystems / 相关子系统: Autograd / 自动求导, ATen operator layer / ATen 算子层, c10 core utilities / c10 核心工具, Python binding layer / Python 绑定层
