# python_functions_3.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/autograd/generated/python_functions_3.cpp`
- Repository: `/root/xw/pytorch`
- Purpose (EN): Defines generated backward-function classes and helpers used by the autograd engine.
- 目的 (CN): 定义自动求导引擎使用的生成式反向函数类与辅助逻辑。
- Lines: 6802
- Language: C++ / C++

## Line-by-Line Analysis / 逐行分析

- Note / 说明: This file is generated or mechanically expanded, so the analysis groups contiguous line ranges rather than paraphrasing every repeated wrapper individually. / 该文件为生成代码或机械展开代码，因此分析按连续行范围组织，而不是逐个重复改写每个封装模板。
### Lines 1-80

```cpp
 1: #include <torch/csrc/autograd/generated/python_functions.h>
 2: 
 3: // @generated from ../tools/autograd/templates/python_functions.cpp
 4: 
 5: #include <Python.h>
 6: #include <ATen/ATen.h>
 7: 
 8: #include <c10/core/SymNodeImpl.h>
 9: #include "torch/csrc/autograd/generated/Functions.h"
10: #include "torch/csrc/autograd/python_cpp_function.h"
11: #include <torch/csrc/autograd/python_variable.h>
12: #include <torch/csrc/autograd/saved_variable.h>
13: #include <torch/csrc/utils/pybind.h>
14: #include <pybind11/pybind11.h>
15: #include <torch/csrc/utils/pybind.h>
16: 
17: // NOTE: See [Sharded File] comment in VariableType
18: 
19: namespace torch::autograd::generated {
20: 
21: template<typename C>
22: static void addClass(PyObject* module, PyTypeObject& type, const char* name,
23:   PyGetSetDef* function_properties=NULL, PyMethodDef* function_methods=NULL)
24: {
25:   _initFunctionPyTypeObject(type, name, function_properties, function_methods);
26:   Py_INCREF(&type);
27:   PyModule_AddObject(module, name, (PyObject*)&type);
28:   registerCppFunction(typeid(C), &type);
29: }
30: 
31: static PyObject* THPAddbmmBackward0_alpha_getter(THPCppFunction *self, void *_unused) {
32:   HANDLE_TH_ERRORS
33:   auto prop = static_cast<AddbmmBackward0*>(self->cdata.get())->alpha;
34:   if (prop.isComplex()) {
35:     auto cprop = prop.to<c10::complex<double>>();
36:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
37:   } else if (prop.isFloatingPoint()) {
38:     return PyFloat_FromDouble(prop.to<double>());
39:   } else if (prop.isIntegral(/*includeBool=*/false)) {
40:     return PyLong_FromLong(prop.to<int64_t>());
41:   } else if (prop.isBoolean()) {
42:     if (prop.to<bool>()) {
43:       Py_RETURN_TRUE;
44:     } else {
45:       Py_RETURN_FALSE;
46:     }
47:   } else {
48:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
49:     return nullptr;
50:   }
51:   END_HANDLE_TH_ERRORS
52: }
53: 
54: static PyObject* THPAddbmmBackward0_batch1_getter(THPCppFunction *self, void *_unused) {
55:   HANDLE_TH_ERRORS
56:   const auto& prop = static_cast<AddbmmBackward0*>(self->cdata.get())->batch1_;
57:   return THPVariable_Wrap(prop.unpack(self->cdata));
58:   END_HANDLE_TH_ERRORS
59: }
60: 
61: static PyObject* THPAddbmmBackward0_batch1_raw_getter(THPCppFunction *self, void *_unused) {
62:   HANDLE_TH_ERRORS
63:   const auto& prop = static_cast<AddbmmBackward0*>(self->cdata.get())->batch1_;
64:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
65:   return obj.release().ptr();
66:   END_HANDLE_TH_ERRORS
67: }
68: 
69: static PyObject* THPAddbmmBackward0_batch1_sym_argsize_0_getter(THPCppFunction *self, void *_unused) {
70:   HANDLE_TH_ERRORS
71:   auto prop = static_cast<AddbmmBackward0*>(self->cdata.get())->batch1_sym_argsize_0;
72:   if (auto m = prop.maybe_as_int()) {
73:     return PyLong_FromUnsignedLong(*m);
74:   } else {
75:     return py::cast(prop).release().ptr();
76:   }
77:   END_HANDLE_TH_ERRORS
78: }
79: 
80: static PyObject* THPAddbmmBackward0_batch1_sym_argsize_1_getter(THPCppFunction *self, void *_unused) {
```

- EN: These lines pull in dependencies such as `torch/csrc/autograd/generated/python_functions.h`, `Python.h`, `ATen/ATen.h`, establishing the headers needed by the implementation. They also open, refine, or close the relevant C++ namespace scope. The main execution path in this span is carried by `addClass`, `_initFunctionPyTypeObject`, `Py_INCREF`. Because this is generated binding code, the span mostly registers or forwards APIs into a mechanically produced Python-facing surface.
- CN: 这些行引入了依赖，例如 `torch/csrc/autograd/generated/python_functions.h`, `Python.h`, `ATen/ATen.h`，为后续实现建立所需的头文件基础。 它们还会打开、细化或关闭相应的 C++ 命名空间作用域。 这一段的主要执行路径由 `addClass`, `_initFunctionPyTypeObject`, `Py_INCREF` 等函数/方法承载。 由于这是生成的绑定代码，这一段主要是在机械化生成的 Python 接口层上完成 API 注册或转发。
### Lines 81-160

```cpp
 81:   HANDLE_TH_ERRORS
 82:   auto prop = static_cast<AddbmmBackward0*>(self->cdata.get())->batch1_sym_argsize_1;
 83:   if (auto m = prop.maybe_as_int()) {
 84:     return PyLong_FromUnsignedLong(*m);
 85:   } else {
 86:     return py::cast(prop).release().ptr();
 87:   }
 88:   END_HANDLE_TH_ERRORS
 89: }
 90: 
 91: static PyObject* THPAddbmmBackward0_batch2_getter(THPCppFunction *self, void *_unused) {
 92:   HANDLE_TH_ERRORS
 93:   const auto& prop = static_cast<AddbmmBackward0*>(self->cdata.get())->batch2_;
 94:   return THPVariable_Wrap(prop.unpack(self->cdata));
 95:   END_HANDLE_TH_ERRORS
 96: }
 97: 
 98: static PyObject* THPAddbmmBackward0_batch2_raw_getter(THPCppFunction *self, void *_unused) {
 99:   HANDLE_TH_ERRORS
100:   const auto& prop = static_cast<AddbmmBackward0*>(self->cdata.get())->batch2_;
101:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
102:   return obj.release().ptr();
103:   END_HANDLE_TH_ERRORS
104: }
105: 
106: static PyObject* THPAddbmmBackward0_batch2_sym_argsize_2_getter(THPCppFunction *self, void *_unused) {
107:   HANDLE_TH_ERRORS
108:   auto prop = static_cast<AddbmmBackward0*>(self->cdata.get())->batch2_sym_argsize_2;
109:   if (auto m = prop.maybe_as_int()) {
110:     return PyLong_FromUnsignedLong(*m);
111:   } else {
112:     return py::cast(prop).release().ptr();
113:   }
114:   END_HANDLE_TH_ERRORS
115: }
116: 
117: static PyObject* THPAddbmmBackward0_beta_getter(THPCppFunction *self, void *_unused) {
118:   HANDLE_TH_ERRORS
119:   auto prop = static_cast<AddbmmBackward0*>(self->cdata.get())->beta;
120:   if (prop.isComplex()) {
121:     auto cprop = prop.to<c10::complex<double>>();
122:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
123:   } else if (prop.isFloatingPoint()) {
124:     return PyFloat_FromDouble(prop.to<double>());
125:   } else if (prop.isIntegral(/*includeBool=*/false)) {
126:     return PyLong_FromLong(prop.to<int64_t>());
127:   } else if (prop.isBoolean()) {
128:     if (prop.to<bool>()) {
129:       Py_RETURN_TRUE;
130:     } else {
131:       Py_RETURN_FALSE;
132:     }
133:   } else {
134:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
135:     return nullptr;
136:   }
137:   END_HANDLE_TH_ERRORS
138: }
139: 
140: static struct PyGetSetDef AddbmmBackward0_properties[] = {
141:   THP_FUNCTION_DEFAULT_PROPERTIES,
142:   {(char*)"_saved_alpha", (getter)THPAddbmmBackward0_alpha_getter, nullptr, nullptr, nullptr},
143:   {(char*)"_saved_batch1", (getter)THPAddbmmBackward0_batch1_getter, nullptr, nullptr, nullptr},
144:   {(char*)"_raw_saved_batch1", (getter)THPAddbmmBackward0_batch1_raw_getter, nullptr, nullptr, nullptr},
145:   {(char*)"_saved_batch1_sym_argsize_0", (getter)THPAddbmmBackward0_batch1_sym_argsize_0_getter, nullptr, nullptr, nullptr},
146:   {(char*)"_saved_batch1_sym_argsize_1", (getter)THPAddbmmBackward0_batch1_sym_argsize_1_getter, nullptr, nullptr, nullptr},
147:   {(char*)"_saved_batch2", (getter)THPAddbmmBackward0_batch2_getter, nullptr, nullptr, nullptr},
148:   {(char*)"_raw_saved_batch2", (getter)THPAddbmmBackward0_batch2_raw_getter, nullptr, nullptr, nullptr},
149:   {(char*)"_saved_batch2_sym_argsize_2", (getter)THPAddbmmBackward0_batch2_sym_argsize_2_getter, nullptr, nullptr, nullptr},
150:   {(char*)"_saved_beta", (getter)THPAddbmmBackward0_beta_getter, nullptr, nullptr, nullptr},
151:   {nullptr} /* sentinel */
152: };
153: 
154: static PyObject* THPAffineGridGeneratorBackward0_align_corners_getter(THPCppFunction *self, void *_unused) {
155:   HANDLE_TH_ERRORS
156:   auto prop = static_cast<AffineGridGeneratorBackward0*>(self->cdata.get())->align_corners;
157:   if (prop) {
158:     Py_RETURN_TRUE;
159:   } else {
160:     Py_RETURN_FALSE;
```

- EN: The main execution path in this span is carried by `PyLong_FromUnsignedLong`, `cast`, `THPAddbmmBackward0_batch2_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyLong_FromUnsignedLong`, `cast`, `THPAddbmmBackward0_batch2_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 161-240

```cpp
161:   }
162:   END_HANDLE_TH_ERRORS
163: }
164: 
165: static PyObject* THPAffineGridGeneratorBackward0_size_getter(THPCppFunction *self, void *_unused) {
166:   HANDLE_TH_ERRORS
167:   auto prop = static_cast<AffineGridGeneratorBackward0*>(self->cdata.get())->size;
168:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
169:   for (auto i : c10::irange(prop.size())) {
170:       auto si = prop[i];
171:       if (auto m = si.maybe_as_int()) {
172:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
173:       } else {
174:         auto py_symint = py::cast(si).release().ptr();
175:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
176:       }
177:   }
178:   return tup;
179:   END_HANDLE_TH_ERRORS
180: }
181: 
182: static struct PyGetSetDef AffineGridGeneratorBackward0_properties[] = {
183:   THP_FUNCTION_DEFAULT_PROPERTIES,
184:   {(char*)"_saved_align_corners", (getter)THPAffineGridGeneratorBackward0_align_corners_getter, nullptr, nullptr, nullptr},
185:   {(char*)"_saved_size", (getter)THPAffineGridGeneratorBackward0_size_getter, nullptr, nullptr, nullptr},
186:   {nullptr} /* sentinel */
187: };
188: 
189: static PyObject* THPAtanhBackward0_self_getter(THPCppFunction *self, void *_unused) {
190:   HANDLE_TH_ERRORS
191:   const auto& prop = static_cast<AtanhBackward0*>(self->cdata.get())->self_;
192:   return THPVariable_Wrap(prop.unpack(self->cdata));
193:   END_HANDLE_TH_ERRORS
194: }
195: 
196: static PyObject* THPAtanhBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
197:   HANDLE_TH_ERRORS
198:   const auto& prop = static_cast<AtanhBackward0*>(self->cdata.get())->self_;
199:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
200:   return obj.release().ptr();
201:   END_HANDLE_TH_ERRORS
202: }
203: 
204: static struct PyGetSetDef AtanhBackward0_properties[] = {
205:   THP_FUNCTION_DEFAULT_PROPERTIES,
206:   {(char*)"_saved_self", (getter)THPAtanhBackward0_self_getter, nullptr, nullptr, nullptr},
207:   {(char*)"_raw_saved_self", (getter)THPAtanhBackward0_self_raw_getter, nullptr, nullptr, nullptr},
208:   {nullptr} /* sentinel */
209: };
210: 
211: 
212: 
213: static struct PyGetSetDef BernoulliBackward1_properties[] = {
214:   THP_FUNCTION_DEFAULT_PROPERTIES,
215: 
216:   {nullptr} /* sentinel */
217: };
218: 
219: 
220: 
221: static struct PyGetSetDef BernoulliBackward2_properties[] = {
222:   THP_FUNCTION_DEFAULT_PROPERTIES,
223: 
224:   {nullptr} /* sentinel */
225: };
226: 
227: static PyObject* THPBmmBackward0_mat2_getter(THPCppFunction *self, void *_unused) {
228:   HANDLE_TH_ERRORS
229:   const auto& prop = static_cast<BmmBackward0*>(self->cdata.get())->mat2_;
230:   return THPVariable_Wrap(prop.unpack(self->cdata));
231:   END_HANDLE_TH_ERRORS
232: }
233: 
234: static PyObject* THPBmmBackward0_mat2_raw_getter(THPCppFunction *self, void *_unused) {
235:   HANDLE_TH_ERRORS
236:   const auto& prop = static_cast<BmmBackward0*>(self->cdata.get())->mat2_;
237:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
238:   return obj.release().ptr();
239:   END_HANDLE_TH_ERRORS
240: }
```

- EN: The main execution path in this span is carried by `THPAffineGridGeneratorBackward0_size_getter`, `PyTuple_New`, `PyTuple_SetItem`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPAffineGridGeneratorBackward0_size_getter`, `PyTuple_New`, `PyTuple_SetItem` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 241-320

```cpp
241: 
242: static PyObject* THPBmmBackward0_self_getter(THPCppFunction *self, void *_unused) {
243:   HANDLE_TH_ERRORS
244:   const auto& prop = static_cast<BmmBackward0*>(self->cdata.get())->self_;
245:   return THPVariable_Wrap(prop.unpack(self->cdata));
246:   END_HANDLE_TH_ERRORS
247: }
248: 
249: static PyObject* THPBmmBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
250:   HANDLE_TH_ERRORS
251:   const auto& prop = static_cast<BmmBackward0*>(self->cdata.get())->self_;
252:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
253:   return obj.release().ptr();
254:   END_HANDLE_TH_ERRORS
255: }
256: 
257: static struct PyGetSetDef BmmBackward0_properties[] = {
258:   THP_FUNCTION_DEFAULT_PROPERTIES,
259:   {(char*)"_saved_mat2", (getter)THPBmmBackward0_mat2_getter, nullptr, nullptr, nullptr},
260:   {(char*)"_raw_saved_mat2", (getter)THPBmmBackward0_mat2_raw_getter, nullptr, nullptr, nullptr},
261:   {(char*)"_saved_self", (getter)THPBmmBackward0_self_getter, nullptr, nullptr, nullptr},
262:   {(char*)"_raw_saved_self", (getter)THPBmmBackward0_self_raw_getter, nullptr, nullptr, nullptr},
263:   {nullptr} /* sentinel */
264: };
265: 
266: 
267: 
268: static struct PyGetSetDef CauchyBackward0_properties[] = {
269:   THP_FUNCTION_DEFAULT_PROPERTIES,
270: 
271:   {nullptr} /* sentinel */
272: };
273: 
274: static PyObject* THPClampMinBackward0_min_getter(THPCppFunction *self, void *_unused) {
275:   HANDLE_TH_ERRORS
276:   auto prop = static_cast<ClampMinBackward0*>(self->cdata.get())->min;
277:   if (prop.isComplex()) {
278:     auto cprop = prop.to<c10::complex<double>>();
279:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
280:   } else if (prop.isFloatingPoint()) {
281:     return PyFloat_FromDouble(prop.to<double>());
282:   } else if (prop.isIntegral(/*includeBool=*/false)) {
283:     return PyLong_FromLong(prop.to<int64_t>());
284:   } else if (prop.isBoolean()) {
285:     if (prop.to<bool>()) {
286:       Py_RETURN_TRUE;
287:     } else {
288:       Py_RETURN_FALSE;
289:     }
290:   } else {
291:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
292:     return nullptr;
293:   }
294:   END_HANDLE_TH_ERRORS
295: }
296: 
297: static PyObject* THPClampMinBackward0_self_getter(THPCppFunction *self, void *_unused) {
298:   HANDLE_TH_ERRORS
299:   const auto& prop = static_cast<ClampMinBackward0*>(self->cdata.get())->self_;
300:   return THPVariable_Wrap(prop.unpack(self->cdata));
301:   END_HANDLE_TH_ERRORS
302: }
303: 
304: static PyObject* THPClampMinBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
305:   HANDLE_TH_ERRORS
306:   const auto& prop = static_cast<ClampMinBackward0*>(self->cdata.get())->self_;
307:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
308:   return obj.release().ptr();
309:   END_HANDLE_TH_ERRORS
310: }
311: 
312: static struct PyGetSetDef ClampMinBackward0_properties[] = {
313:   THP_FUNCTION_DEFAULT_PROPERTIES,
314:   {(char*)"_saved_min", (getter)THPClampMinBackward0_min_getter, nullptr, nullptr, nullptr},
315:   {(char*)"_saved_self", (getter)THPClampMinBackward0_self_getter, nullptr, nullptr, nullptr},
316:   {(char*)"_raw_saved_self", (getter)THPClampMinBackward0_self_raw_getter, nullptr, nullptr, nullptr},
317:   {nullptr} /* sentinel */
318: };
319: 
320: static PyObject* THPClampMinBackward1_min_getter(THPCppFunction *self, void *_unused) {
```

- EN: The main execution path in this span is carried by `THPBmmBackward0_self_getter`, `THPVariable_Wrap`, `THPBmmBackward0_self_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPBmmBackward0_self_getter`, `THPVariable_Wrap`, `THPBmmBackward0_self_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 321-400

```cpp
321:   HANDLE_TH_ERRORS
322:   const auto& prop = static_cast<ClampMinBackward1*>(self->cdata.get())->min_;
323:   return THPVariable_Wrap(prop.unpack(self->cdata));
324:   END_HANDLE_TH_ERRORS
325: }
326: 
327: static PyObject* THPClampMinBackward1_min_raw_getter(THPCppFunction *self, void *_unused) {
328:   HANDLE_TH_ERRORS
329:   const auto& prop = static_cast<ClampMinBackward1*>(self->cdata.get())->min_;
330:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
331:   return obj.release().ptr();
332:   END_HANDLE_TH_ERRORS
333: }
334: 
335: static PyObject* THPClampMinBackward1_self_getter(THPCppFunction *self, void *_unused) {
336:   HANDLE_TH_ERRORS
337:   const auto& prop = static_cast<ClampMinBackward1*>(self->cdata.get())->self_;
338:   return THPVariable_Wrap(prop.unpack(self->cdata));
339:   END_HANDLE_TH_ERRORS
340: }
341: 
342: static PyObject* THPClampMinBackward1_self_raw_getter(THPCppFunction *self, void *_unused) {
343:   HANDLE_TH_ERRORS
344:   const auto& prop = static_cast<ClampMinBackward1*>(self->cdata.get())->self_;
345:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
346:   return obj.release().ptr();
347:   END_HANDLE_TH_ERRORS
348: }
349: 
350: static struct PyGetSetDef ClampMinBackward1_properties[] = {
351:   THP_FUNCTION_DEFAULT_PROPERTIES,
352:   {(char*)"_saved_min", (getter)THPClampMinBackward1_min_getter, nullptr, nullptr, nullptr},
353:   {(char*)"_raw_saved_min", (getter)THPClampMinBackward1_min_raw_getter, nullptr, nullptr, nullptr},
354:   {(char*)"_saved_self", (getter)THPClampMinBackward1_self_getter, nullptr, nullptr, nullptr},
355:   {(char*)"_raw_saved_self", (getter)THPClampMinBackward1_self_raw_getter, nullptr, nullptr, nullptr},
356:   {nullptr} /* sentinel */
357: };
358: 
359: 
360: 
361: static struct PyGetSetDef CoalesceBackward0_properties[] = {
362:   THP_FUNCTION_DEFAULT_PROPERTIES,
363: 
364:   {nullptr} /* sentinel */
365: };
366: 
367: 
368: 
369: static struct PyGetSetDef ConjPhysicalBackward1_properties[] = {
370:   THP_FUNCTION_DEFAULT_PROPERTIES,
371: 
372:   {nullptr} /* sentinel */
373: };
374: 
375: static PyObject* THPCumprodBackward0_dim_getter(THPCppFunction *self, void *_unused) {
376:   HANDLE_TH_ERRORS
377:   auto prop = static_cast<CumprodBackward0*>(self->cdata.get())->dim;
378:   return PyLong_FromUnsignedLong((int64_t) prop);
379:   END_HANDLE_TH_ERRORS
380: }
381: 
382: static PyObject* THPCumprodBackward0_self_getter(THPCppFunction *self, void *_unused) {
383:   HANDLE_TH_ERRORS
384:   const auto& prop = static_cast<CumprodBackward0*>(self->cdata.get())->self_;
385:   return THPVariable_Wrap(prop.unpack(self->cdata));
386:   END_HANDLE_TH_ERRORS
387: }
388: 
389: static PyObject* THPCumprodBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
390:   HANDLE_TH_ERRORS
391:   const auto& prop = static_cast<CumprodBackward0*>(self->cdata.get())->self_;
392:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
393:   return obj.release().ptr();
394:   END_HANDLE_TH_ERRORS
395: }
396: 
397: static PyObject* THPCumprodBackward0_result_getter(THPCppFunction *self, void *_unused) {
398:   HANDLE_TH_ERRORS
399:   const auto& prop = static_cast<CumprodBackward0*>(self->cdata.get())->result_;
400:   return THPVariable_Wrap(prop.unpack(self->cdata));
```

- EN: The main execution path in this span is carried by `THPVariable_Wrap`, `THPClampMinBackward1_min_raw_getter`, `cast`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_Wrap`, `THPClampMinBackward1_min_raw_getter`, `cast` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 401-480

```cpp
401:   END_HANDLE_TH_ERRORS
402: }
403: 
404: static PyObject* THPCumprodBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
405:   HANDLE_TH_ERRORS
406:   const auto& prop = static_cast<CumprodBackward0*>(self->cdata.get())->result_;
407:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
408:   return obj.release().ptr();
409:   END_HANDLE_TH_ERRORS
410: }
411: 
412: static struct PyGetSetDef CumprodBackward0_properties[] = {
413:   THP_FUNCTION_DEFAULT_PROPERTIES,
414:   {(char*)"_saved_dim", (getter)THPCumprodBackward0_dim_getter, nullptr, nullptr, nullptr},
415:   {(char*)"_saved_self", (getter)THPCumprodBackward0_self_getter, nullptr, nullptr, nullptr},
416:   {(char*)"_raw_saved_self", (getter)THPCumprodBackward0_self_raw_getter, nullptr, nullptr, nullptr},
417:   {(char*)"_saved_result", (getter)THPCumprodBackward0_result_getter, nullptr, nullptr, nullptr},
418:   {(char*)"_raw_saved_result", (getter)THPCumprodBackward0_result_raw_getter, nullptr, nullptr, nullptr},
419:   {nullptr} /* sentinel */
420: };
421: 
422: static PyObject* THPLinalgDetBackward0_A_getter(THPCppFunction *self, void *_unused) {
423:   HANDLE_TH_ERRORS
424:   const auto& prop = static_cast<LinalgDetBackward0*>(self->cdata.get())->A_;
425:   return THPVariable_Wrap(prop.unpack(self->cdata));
426:   END_HANDLE_TH_ERRORS
427: }
428: 
429: static PyObject* THPLinalgDetBackward0_A_raw_getter(THPCppFunction *self, void *_unused) {
430:   HANDLE_TH_ERRORS
431:   const auto& prop = static_cast<LinalgDetBackward0*>(self->cdata.get())->A_;
432:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
433:   return obj.release().ptr();
434:   END_HANDLE_TH_ERRORS
435: }
436: 
437: static PyObject* THPLinalgDetBackward0_LU_getter(THPCppFunction *self, void *_unused) {
438:   HANDLE_TH_ERRORS
439:   const auto& prop = static_cast<LinalgDetBackward0*>(self->cdata.get())->LU_;
440:   return THPVariable_Wrap(prop.unpack(self->cdata));
441:   END_HANDLE_TH_ERRORS
442: }
443: 
444: static PyObject* THPLinalgDetBackward0_LU_raw_getter(THPCppFunction *self, void *_unused) {
445:   HANDLE_TH_ERRORS
446:   const auto& prop = static_cast<LinalgDetBackward0*>(self->cdata.get())->LU_;
447:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
448:   return obj.release().ptr();
449:   END_HANDLE_TH_ERRORS
450: }
451: 
452: static PyObject* THPLinalgDetBackward0_pivots_getter(THPCppFunction *self, void *_unused) {
453:   HANDLE_TH_ERRORS
454:   const auto& prop = static_cast<LinalgDetBackward0*>(self->cdata.get())->pivots_;
455:   return THPVariable_Wrap(prop.unpack(self->cdata));
456:   END_HANDLE_TH_ERRORS
457: }
458: 
459: static PyObject* THPLinalgDetBackward0_pivots_raw_getter(THPCppFunction *self, void *_unused) {
460:   HANDLE_TH_ERRORS
461:   const auto& prop = static_cast<LinalgDetBackward0*>(self->cdata.get())->pivots_;
462:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
463:   return obj.release().ptr();
464:   END_HANDLE_TH_ERRORS
465: }
466: 
467: static PyObject* THPLinalgDetBackward0_result_getter(THPCppFunction *self, void *_unused) {
468:   HANDLE_TH_ERRORS
469:   const auto& prop = static_cast<LinalgDetBackward0*>(self->cdata.get())->result_;
470:   return THPVariable_Wrap(prop.unpack(self->cdata));
471:   END_HANDLE_TH_ERRORS
472: }
473: 
474: static PyObject* THPLinalgDetBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
475:   HANDLE_TH_ERRORS
476:   const auto& prop = static_cast<LinalgDetBackward0*>(self->cdata.get())->result_;
477:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
478:   return obj.release().ptr();
479:   END_HANDLE_TH_ERRORS
480: }
```

- EN: The main execution path in this span is carried by `THPCumprodBackward0_result_raw_getter`, `cast`, `THPLinalgDetBackward0_A_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPCumprodBackward0_result_raw_getter`, `cast`, `THPLinalgDetBackward0_A_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 481-560

```cpp
481: 
482: static struct PyGetSetDef LinalgDetBackward0_properties[] = {
483:   THP_FUNCTION_DEFAULT_PROPERTIES,
484:   {(char*)"_saved_A", (getter)THPLinalgDetBackward0_A_getter, nullptr, nullptr, nullptr},
485:   {(char*)"_raw_saved_A", (getter)THPLinalgDetBackward0_A_raw_getter, nullptr, nullptr, nullptr},
486:   {(char*)"_saved_LU", (getter)THPLinalgDetBackward0_LU_getter, nullptr, nullptr, nullptr},
487:   {(char*)"_raw_saved_LU", (getter)THPLinalgDetBackward0_LU_raw_getter, nullptr, nullptr, nullptr},
488:   {(char*)"_saved_pivots", (getter)THPLinalgDetBackward0_pivots_getter, nullptr, nullptr, nullptr},
489:   {(char*)"_raw_saved_pivots", (getter)THPLinalgDetBackward0_pivots_raw_getter, nullptr, nullptr, nullptr},
490:   {(char*)"_saved_result", (getter)THPLinalgDetBackward0_result_getter, nullptr, nullptr, nullptr},
491:   {(char*)"_raw_saved_result", (getter)THPLinalgDetBackward0_result_raw_getter, nullptr, nullptr, nullptr},
492:   {nullptr} /* sentinel */
493: };
494: 
495: static PyObject* THPDiagEmbedBackward0_dim1_getter(THPCppFunction *self, void *_unused) {
496:   HANDLE_TH_ERRORS
497:   auto prop = static_cast<DiagEmbedBackward0*>(self->cdata.get())->dim1;
498:   return PyLong_FromUnsignedLong((int64_t) prop);
499:   END_HANDLE_TH_ERRORS
500: }
501: 
502: static PyObject* THPDiagEmbedBackward0_dim2_getter(THPCppFunction *self, void *_unused) {
503:   HANDLE_TH_ERRORS
504:   auto prop = static_cast<DiagEmbedBackward0*>(self->cdata.get())->dim2;
505:   return PyLong_FromUnsignedLong((int64_t) prop);
506:   END_HANDLE_TH_ERRORS
507: }
508: 
509: static PyObject* THPDiagEmbedBackward0_offset_getter(THPCppFunction *self, void *_unused) {
510:   HANDLE_TH_ERRORS
511:   auto prop = static_cast<DiagEmbedBackward0*>(self->cdata.get())->offset;
512:   return PyLong_FromUnsignedLong((int64_t) prop);
513:   END_HANDLE_TH_ERRORS
514: }
515: 
516: static struct PyGetSetDef DiagEmbedBackward0_properties[] = {
517:   THP_FUNCTION_DEFAULT_PROPERTIES,
518:   {(char*)"_saved_dim1", (getter)THPDiagEmbedBackward0_dim1_getter, nullptr, nullptr, nullptr},
519:   {(char*)"_saved_dim2", (getter)THPDiagEmbedBackward0_dim2_getter, nullptr, nullptr, nullptr},
520:   {(char*)"_saved_offset", (getter)THPDiagEmbedBackward0_offset_getter, nullptr, nullptr, nullptr},
521:   {nullptr} /* sentinel */
522: };
523: 
524: static PyObject* THPDiagonalBackward0_dim1_getter(THPCppFunction *self, void *_unused) {
525:   HANDLE_TH_ERRORS
526:   auto prop = static_cast<DiagonalBackward0*>(self->cdata.get())->dim1;
527:   return PyLong_FromUnsignedLong((int64_t) prop);
528:   END_HANDLE_TH_ERRORS
529: }
530: 
531: static PyObject* THPDiagonalBackward0_dim2_getter(THPCppFunction *self, void *_unused) {
532:   HANDLE_TH_ERRORS
533:   auto prop = static_cast<DiagonalBackward0*>(self->cdata.get())->dim2;
534:   return PyLong_FromUnsignedLong((int64_t) prop);
535:   END_HANDLE_TH_ERRORS
536: }
537: 
538: static PyObject* THPDiagonalBackward0_offset_getter(THPCppFunction *self, void *_unused) {
539:   HANDLE_TH_ERRORS
540:   auto prop = static_cast<DiagonalBackward0*>(self->cdata.get())->offset;
541:   return PyLong_FromUnsignedLong((int64_t) prop);
542:   END_HANDLE_TH_ERRORS
543: }
544: 
545: static PyObject* THPDiagonalBackward0_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
546:   HANDLE_TH_ERRORS
547:   auto prop = static_cast<DiagonalBackward0*>(self->cdata.get())->self_sym_sizes;
548:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
549:   for (auto i : c10::irange(prop.size())) {
550:       auto si = prop[i];
551:       if (auto m = si.maybe_as_int()) {
552:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
553:       } else {
554:         auto py_symint = py::cast(si).release().ptr();
555:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
556:       }
557:   }
558:   return tup;
559:   END_HANDLE_TH_ERRORS
560: }
```

- EN: The main execution path in this span is carried by `THPDiagEmbedBackward0_dim1_getter`, `PyLong_FromUnsignedLong`, `THPDiagEmbedBackward0_dim2_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPDiagEmbedBackward0_dim1_getter`, `PyLong_FromUnsignedLong`, `THPDiagEmbedBackward0_dim2_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 561-640

```cpp
561: 
562: static struct PyGetSetDef DiagonalBackward0_properties[] = {
563:   THP_FUNCTION_DEFAULT_PROPERTIES,
564:   {(char*)"_saved_dim1", (getter)THPDiagonalBackward0_dim1_getter, nullptr, nullptr, nullptr},
565:   {(char*)"_saved_dim2", (getter)THPDiagonalBackward0_dim2_getter, nullptr, nullptr, nullptr},
566:   {(char*)"_saved_offset", (getter)THPDiagonalBackward0_offset_getter, nullptr, nullptr, nullptr},
567:   {(char*)"_saved_self_sym_sizes", (getter)THPDiagonalBackward0_self_sym_sizes_getter, nullptr, nullptr, nullptr},
568:   {nullptr} /* sentinel */
569: };
570: 
571: static PyObject* THPNativeDropoutBackwardBackward0_grad_output_getter(THPCppFunction *self, void *_unused) {
572:   HANDLE_TH_ERRORS
573:   const auto& prop = static_cast<NativeDropoutBackwardBackward0*>(self->cdata.get())->grad_output_;
574:   return THPVariable_Wrap(prop.unpack(self->cdata));
575:   END_HANDLE_TH_ERRORS
576: }
577: 
578: static PyObject* THPNativeDropoutBackwardBackward0_grad_output_raw_getter(THPCppFunction *self, void *_unused) {
579:   HANDLE_TH_ERRORS
580:   const auto& prop = static_cast<NativeDropoutBackwardBackward0*>(self->cdata.get())->grad_output_;
581:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
582:   return obj.release().ptr();
583:   END_HANDLE_TH_ERRORS
584: }
585: 
586: static PyObject* THPNativeDropoutBackwardBackward0_mask_getter(THPCppFunction *self, void *_unused) {
587:   HANDLE_TH_ERRORS
588:   const auto& prop = static_cast<NativeDropoutBackwardBackward0*>(self->cdata.get())->mask_;
589:   return THPVariable_Wrap(prop.unpack(self->cdata));
590:   END_HANDLE_TH_ERRORS
591: }
592: 
593: static PyObject* THPNativeDropoutBackwardBackward0_mask_raw_getter(THPCppFunction *self, void *_unused) {
594:   HANDLE_TH_ERRORS
595:   const auto& prop = static_cast<NativeDropoutBackwardBackward0*>(self->cdata.get())->mask_;
596:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
597:   return obj.release().ptr();
598:   END_HANDLE_TH_ERRORS
599: }
600: 
601: static PyObject* THPNativeDropoutBackwardBackward0_scale_getter(THPCppFunction *self, void *_unused) {
602:   HANDLE_TH_ERRORS
603:   auto prop = static_cast<NativeDropoutBackwardBackward0*>(self->cdata.get())->scale;
604:   return PyFloat_FromDouble((double) prop);
605:   END_HANDLE_TH_ERRORS
606: }
607: 
608: static struct PyGetSetDef NativeDropoutBackwardBackward0_properties[] = {
609:   THP_FUNCTION_DEFAULT_PROPERTIES,
610:   {(char*)"_saved_grad_output", (getter)THPNativeDropoutBackwardBackward0_grad_output_getter, nullptr, nullptr, nullptr},
611:   {(char*)"_raw_saved_grad_output", (getter)THPNativeDropoutBackwardBackward0_grad_output_raw_getter, nullptr, nullptr, nullptr},
612:   {(char*)"_saved_mask", (getter)THPNativeDropoutBackwardBackward0_mask_getter, nullptr, nullptr, nullptr},
613:   {(char*)"_raw_saved_mask", (getter)THPNativeDropoutBackwardBackward0_mask_raw_getter, nullptr, nullptr, nullptr},
614:   {(char*)"_saved_scale", (getter)THPNativeDropoutBackwardBackward0_scale_getter, nullptr, nullptr, nullptr},
615:   {nullptr} /* sentinel */
616: };
617: 
618: 
619: 
620: static struct PyGetSetDef EqBackward0_properties[] = {
621:   THP_FUNCTION_DEFAULT_PROPERTIES,
622: 
623:   {nullptr} /* sentinel */
624: };
625: 
626: 
627: 
628: static struct PyGetSetDef EqBackward1_properties[] = {
629:   THP_FUNCTION_DEFAULT_PROPERTIES,
630: 
631:   {nullptr} /* sentinel */
632: };
633: 
634: static PyObject* THPExpandBackward0_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
635:   HANDLE_TH_ERRORS
636:   auto prop = static_cast<ExpandBackward0*>(self->cdata.get())->self_sym_sizes;
637:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
638:   for (auto i : c10::irange(prop.size())) {
639:       auto si = prop[i];
640:       if (auto m = si.maybe_as_int()) {
```

- EN: The main execution path in this span is carried by `THPNativeDropoutBackwardBackward0_grad_output_getter`, `THPVariable_Wrap`, `THPNativeDropoutBackwardBackward0_grad_output_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPNativeDropoutBackwardBackward0_grad_output_getter`, `THPVariable_Wrap`, `THPNativeDropoutBackwardBackward0_grad_output_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 641-720

```cpp
641:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
642:       } else {
643:         auto py_symint = py::cast(si).release().ptr();
644:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
645:       }
646:   }
647:   return tup;
648:   END_HANDLE_TH_ERRORS
649: }
650: 
651: static struct PyGetSetDef ExpandBackward0_properties[] = {
652:   THP_FUNCTION_DEFAULT_PROPERTIES,
653:   {(char*)"_saved_self_sym_sizes", (getter)THPExpandBackward0_self_sym_sizes_getter, nullptr, nullptr, nullptr},
654:   {nullptr} /* sentinel */
655: };
656: 
657: 
658: 
659: static struct PyGetSetDef FloorBackward0_properties[] = {
660:   THP_FUNCTION_DEFAULT_PROPERTIES,
661: 
662:   {nullptr} /* sentinel */
663: };
664: 
665: static PyObject* THPFrexpBackward0_exponent_getter(THPCppFunction *self, void *_unused) {
666:   HANDLE_TH_ERRORS
667:   const auto& prop = static_cast<FrexpBackward0*>(self->cdata.get())->exponent_;
668:   return THPVariable_Wrap(prop.unpack(self->cdata));
669:   END_HANDLE_TH_ERRORS
670: }
671: 
672: static PyObject* THPFrexpBackward0_exponent_raw_getter(THPCppFunction *self, void *_unused) {
673:   HANDLE_TH_ERRORS
674:   const auto& prop = static_cast<FrexpBackward0*>(self->cdata.get())->exponent_;
675:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
676:   return obj.release().ptr();
677:   END_HANDLE_TH_ERRORS
678: }
679: 
680: static struct PyGetSetDef FrexpBackward0_properties[] = {
681:   THP_FUNCTION_DEFAULT_PROPERTIES,
682:   {(char*)"_saved_exponent", (getter)THPFrexpBackward0_exponent_getter, nullptr, nullptr, nullptr},
683:   {(char*)"_raw_saved_exponent", (getter)THPFrexpBackward0_exponent_raw_getter, nullptr, nullptr, nullptr},
684:   {nullptr} /* sentinel */
685: };
686: 
687: static PyObject* THPGatherBackward0_dim_getter(THPCppFunction *self, void *_unused) {
688:   HANDLE_TH_ERRORS
689:   auto prop = static_cast<GatherBackward0*>(self->cdata.get())->dim;
690:   return PyLong_FromUnsignedLong((int64_t) prop);
691:   END_HANDLE_TH_ERRORS
692: }
693: 
694: static PyObject* THPGatherBackward0_index_getter(THPCppFunction *self, void *_unused) {
695:   HANDLE_TH_ERRORS
696:   const auto& prop = static_cast<GatherBackward0*>(self->cdata.get())->index_;
697:   return THPVariable_Wrap(prop.unpack(self->cdata));
698:   END_HANDLE_TH_ERRORS
699: }
700: 
701: static PyObject* THPGatherBackward0_index_raw_getter(THPCppFunction *self, void *_unused) {
702:   HANDLE_TH_ERRORS
703:   const auto& prop = static_cast<GatherBackward0*>(self->cdata.get())->index_;
704:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
705:   return obj.release().ptr();
706:   END_HANDLE_TH_ERRORS
707: }
708: 
709: static PyObject* THPGatherBackward0_self_getter(THPCppFunction *self, void *_unused) {
710:   HANDLE_TH_ERRORS
711:   const auto& prop = static_cast<GatherBackward0*>(self->cdata.get())->self_;
712:   return THPVariable_Wrap(prop.unpack(self->cdata));
713:   END_HANDLE_TH_ERRORS
714: }
715: 
716: static PyObject* THPGatherBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
717:   HANDLE_TH_ERRORS
718:   const auto& prop = static_cast<GatherBackward0*>(self->cdata.get())->self_;
719:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
720:   return obj.release().ptr();
```

- EN: The main execution path in this span is carried by `PyTuple_SetItem`, `cast`, `THPFrexpBackward0_exponent_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyTuple_SetItem`, `cast`, `THPFrexpBackward0_exponent_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 721-800

```cpp
721:   END_HANDLE_TH_ERRORS
722: }
723: 
724: static PyObject* THPGatherBackward0_sparse_grad_getter(THPCppFunction *self, void *_unused) {
725:   HANDLE_TH_ERRORS
726:   auto prop = static_cast<GatherBackward0*>(self->cdata.get())->sparse_grad;
727:   if (prop) {
728:     Py_RETURN_TRUE;
729:   } else {
730:     Py_RETURN_FALSE;
731:   }
732:   END_HANDLE_TH_ERRORS
733: }
734: 
735: static struct PyGetSetDef GatherBackward0_properties[] = {
736:   THP_FUNCTION_DEFAULT_PROPERTIES,
737:   {(char*)"_saved_dim", (getter)THPGatherBackward0_dim_getter, nullptr, nullptr, nullptr},
738:   {(char*)"_saved_index", (getter)THPGatherBackward0_index_getter, nullptr, nullptr, nullptr},
739:   {(char*)"_raw_saved_index", (getter)THPGatherBackward0_index_raw_getter, nullptr, nullptr, nullptr},
740:   {(char*)"_saved_self", (getter)THPGatherBackward0_self_getter, nullptr, nullptr, nullptr},
741:   {(char*)"_raw_saved_self", (getter)THPGatherBackward0_self_raw_getter, nullptr, nullptr, nullptr},
742:   {(char*)"_saved_sparse_grad", (getter)THPGatherBackward0_sparse_grad_getter, nullptr, nullptr, nullptr},
743:   {nullptr} /* sentinel */
744: };
745: 
746: 
747: 
748: static struct PyGetSetDef GeBackward0_properties[] = {
749:   THP_FUNCTION_DEFAULT_PROPERTIES,
750: 
751:   {nullptr} /* sentinel */
752: };
753: 
754: 
755: 
756: static struct PyGetSetDef GeBackward1_properties[] = {
757:   THP_FUNCTION_DEFAULT_PROPERTIES,
758: 
759:   {nullptr} /* sentinel */
760: };
761: 
762: static PyObject* THPGridSampler3DBackwardBackward0_align_corners_getter(THPCppFunction *self, void *_unused) {
763:   HANDLE_TH_ERRORS
764:   auto prop = static_cast<GridSampler3DBackwardBackward0*>(self->cdata.get())->align_corners;
765:   if (prop) {
766:     Py_RETURN_TRUE;
767:   } else {
768:     Py_RETURN_FALSE;
769:   }
770:   END_HANDLE_TH_ERRORS
771: }
772: 
773: static PyObject* THPGridSampler3DBackwardBackward0_grad_output_getter(THPCppFunction *self, void *_unused) {
774:   HANDLE_TH_ERRORS
775:   const auto& prop = static_cast<GridSampler3DBackwardBackward0*>(self->cdata.get())->grad_output_;
776:   return THPVariable_Wrap(prop.unpack(self->cdata));
777:   END_HANDLE_TH_ERRORS
778: }
779: 
780: static PyObject* THPGridSampler3DBackwardBackward0_grad_output_raw_getter(THPCppFunction *self, void *_unused) {
781:   HANDLE_TH_ERRORS
782:   const auto& prop = static_cast<GridSampler3DBackwardBackward0*>(self->cdata.get())->grad_output_;
783:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
784:   return obj.release().ptr();
785:   END_HANDLE_TH_ERRORS
786: }
787: 
788: static PyObject* THPGridSampler3DBackwardBackward0_grid_getter(THPCppFunction *self, void *_unused) {
789:   HANDLE_TH_ERRORS
790:   const auto& prop = static_cast<GridSampler3DBackwardBackward0*>(self->cdata.get())->grid_;
791:   return THPVariable_Wrap(prop.unpack(self->cdata));
792:   END_HANDLE_TH_ERRORS
793: }
794: 
795: static PyObject* THPGridSampler3DBackwardBackward0_grid_raw_getter(THPCppFunction *self, void *_unused) {
796:   HANDLE_TH_ERRORS
797:   const auto& prop = static_cast<GridSampler3DBackwardBackward0*>(self->cdata.get())->grid_;
798:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
799:   return obj.release().ptr();
800:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPGatherBackward0_sparse_grad_getter`, `THPGridSampler3DBackwardBackward0_align_corners_getter`, `THPGridSampler3DBackwardBackward0_grad_output_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPGatherBackward0_sparse_grad_getter`, `THPGridSampler3DBackwardBackward0_align_corners_getter`, `THPGridSampler3DBackwardBackward0_grad_output_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 801-880

```cpp
801: }
802: 
803: static PyObject* THPGridSampler3DBackwardBackward0_input_getter(THPCppFunction *self, void *_unused) {
804:   HANDLE_TH_ERRORS
805:   const auto& prop = static_cast<GridSampler3DBackwardBackward0*>(self->cdata.get())->input_;
806:   return THPVariable_Wrap(prop.unpack(self->cdata));
807:   END_HANDLE_TH_ERRORS
808: }
809: 
810: static PyObject* THPGridSampler3DBackwardBackward0_input_raw_getter(THPCppFunction *self, void *_unused) {
811:   HANDLE_TH_ERRORS
812:   const auto& prop = static_cast<GridSampler3DBackwardBackward0*>(self->cdata.get())->input_;
813:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
814:   return obj.release().ptr();
815:   END_HANDLE_TH_ERRORS
816: }
817: 
818: static PyObject* THPGridSampler3DBackwardBackward0_interpolation_mode_getter(THPCppFunction *self, void *_unused) {
819:   HANDLE_TH_ERRORS
820:   auto prop = static_cast<GridSampler3DBackwardBackward0*>(self->cdata.get())->interpolation_mode;
821:   return PyLong_FromUnsignedLong((int64_t) prop);
822:   END_HANDLE_TH_ERRORS
823: }
824: 
825: static PyObject* THPGridSampler3DBackwardBackward0_padding_mode_getter(THPCppFunction *self, void *_unused) {
826:   HANDLE_TH_ERRORS
827:   auto prop = static_cast<GridSampler3DBackwardBackward0*>(self->cdata.get())->padding_mode;
828:   return PyLong_FromUnsignedLong((int64_t) prop);
829:   END_HANDLE_TH_ERRORS
830: }
831: 
832: static struct PyGetSetDef GridSampler3DBackwardBackward0_properties[] = {
833:   THP_FUNCTION_DEFAULT_PROPERTIES,
834:   {(char*)"_saved_align_corners", (getter)THPGridSampler3DBackwardBackward0_align_corners_getter, nullptr, nullptr, nullptr},
835:   {(char*)"_saved_grad_output", (getter)THPGridSampler3DBackwardBackward0_grad_output_getter, nullptr, nullptr, nullptr},
836:   {(char*)"_raw_saved_grad_output", (getter)THPGridSampler3DBackwardBackward0_grad_output_raw_getter, nullptr, nullptr, nullptr},
837:   {(char*)"_saved_grid", (getter)THPGridSampler3DBackwardBackward0_grid_getter, nullptr, nullptr, nullptr},
838:   {(char*)"_raw_saved_grid", (getter)THPGridSampler3DBackwardBackward0_grid_raw_getter, nullptr, nullptr, nullptr},
839:   {(char*)"_saved_input", (getter)THPGridSampler3DBackwardBackward0_input_getter, nullptr, nullptr, nullptr},
840:   {(char*)"_raw_saved_input", (getter)THPGridSampler3DBackwardBackward0_input_raw_getter, nullptr, nullptr, nullptr},
841:   {(char*)"_saved_interpolation_mode", (getter)THPGridSampler3DBackwardBackward0_interpolation_mode_getter, nullptr, nullptr, nullptr},
842:   {(char*)"_saved_padding_mode", (getter)THPGridSampler3DBackwardBackward0_padding_mode_getter, nullptr, nullptr, nullptr},
843:   {nullptr} /* sentinel */
844: };
845: 
846: static PyObject* THPHardsigmoidBackward0_self_getter(THPCppFunction *self, void *_unused) {
847:   HANDLE_TH_ERRORS
848:   const auto& prop = static_cast<HardsigmoidBackward0*>(self->cdata.get())->self_;
849:   return THPVariable_Wrap(prop.unpack(self->cdata));
850:   END_HANDLE_TH_ERRORS
851: }
852: 
853: static PyObject* THPHardsigmoidBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
854:   HANDLE_TH_ERRORS
855:   const auto& prop = static_cast<HardsigmoidBackward0*>(self->cdata.get())->self_;
856:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
857:   return obj.release().ptr();
858:   END_HANDLE_TH_ERRORS
859: }
860: 
861: static struct PyGetSetDef HardsigmoidBackward0_properties[] = {
862:   THP_FUNCTION_DEFAULT_PROPERTIES,
863:   {(char*)"_saved_self", (getter)THPHardsigmoidBackward0_self_getter, nullptr, nullptr, nullptr},
864:   {(char*)"_raw_saved_self", (getter)THPHardsigmoidBackward0_self_raw_getter, nullptr, nullptr, nullptr},
865:   {nullptr} /* sentinel */
866: };
867: 
868: static PyObject* THPI0Backward0_self_getter(THPCppFunction *self, void *_unused) {
869:   HANDLE_TH_ERRORS
870:   const auto& prop = static_cast<I0Backward0*>(self->cdata.get())->self_;
871:   return THPVariable_Wrap(prop.unpack(self->cdata));
872:   END_HANDLE_TH_ERRORS
873: }
874: 
875: static PyObject* THPI0Backward0_self_raw_getter(THPCppFunction *self, void *_unused) {
876:   HANDLE_TH_ERRORS
877:   const auto& prop = static_cast<I0Backward0*>(self->cdata.get())->self_;
878:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
879:   return obj.release().ptr();
880:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPGridSampler3DBackwardBackward0_input_getter`, `THPVariable_Wrap`, `THPGridSampler3DBackwardBackward0_input_raw_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPGridSampler3DBackwardBackward0_input_getter`, `THPVariable_Wrap`, `THPGridSampler3DBackwardBackward0_input_raw_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 881-960

```cpp
881: }
882: 
883: static struct PyGetSetDef I0Backward0_properties[] = {
884:   THP_FUNCTION_DEFAULT_PROPERTIES,
885:   {(char*)"_saved_self", (getter)THPI0Backward0_self_getter, nullptr, nullptr, nullptr},
886:   {(char*)"_raw_saved_self", (getter)THPI0Backward0_self_raw_getter, nullptr, nullptr, nullptr},
887:   {nullptr} /* sentinel */
888: };
889: 
890: static PyObject* THPUnsafeIndexBackward0_indices_getter(THPCppFunction *self, void *_unused) {
891:   HANDLE_TH_ERRORS
892:   const auto *node = static_cast<UnsafeIndexBackward0*>(self->cdata.get());
893:   const auto& prop = node->indices_;
894:   if (node->indices_released_) {
895:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
896:     return nullptr;
897:   }
898:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
899:   for (auto i: c10::irange(prop.size())) {
900:     PyTuple_SetItem(tup, (Py_ssize_t) i, THPVariable_Wrap(prop[i].unpack(self->cdata)));
901:   }
902:   return tup;
903:   END_HANDLE_TH_ERRORS
904: }
905: 
906: static PyObject* THPUnsafeIndexBackward0_indices_raw_getter(THPCppFunction *self, void *_unused) {
907:   HANDLE_TH_ERRORS
908:   const auto *node = static_cast<UnsafeIndexBackward0*>(self->cdata.get());
909:   const auto& prop = node->indices_;
910:   if (node->indices_released_) {
911:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
912:     return nullptr;
913:   }
914:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
915:   for (auto i : c10::irange(prop.size())) {
916:     pybind11::object obj = pybind11::cast(prop[i], pybind11::return_value_policy::reference);
917:     PyTuple_SetItem(tup, (Py_ssize_t) i, obj.release().ptr());
918:   }
919:   return tup;
920:   END_HANDLE_TH_ERRORS
921: }
922: 
923: static PyObject* THPUnsafeIndexBackward0_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
924:   HANDLE_TH_ERRORS
925:   auto prop = static_cast<UnsafeIndexBackward0*>(self->cdata.get())->self_sym_sizes;
926:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
927:   for (auto i : c10::irange(prop.size())) {
928:       auto si = prop[i];
929:       if (auto m = si.maybe_as_int()) {
930:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
931:       } else {
932:         auto py_symint = py::cast(si).release().ptr();
933:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
934:       }
935:   }
936:   return tup;
937:   END_HANDLE_TH_ERRORS
938: }
939: 
940: static struct PyGetSetDef UnsafeIndexBackward0_properties[] = {
941:   THP_FUNCTION_DEFAULT_PROPERTIES,
942:   {(char*)"_saved_indices", (getter)THPUnsafeIndexBackward0_indices_getter, nullptr, nullptr, nullptr},
943:   {(char*)"_raw_saved_indices", (getter)THPUnsafeIndexBackward0_indices_raw_getter, nullptr, nullptr, nullptr},
944:   {(char*)"_saved_self_sym_sizes", (getter)THPUnsafeIndexBackward0_self_sym_sizes_getter, nullptr, nullptr, nullptr},
945:   {nullptr} /* sentinel */
946: };
947: 
948: static PyObject* THPLinalgInvExBackward0_inverse_getter(THPCppFunction *self, void *_unused) {
949:   HANDLE_TH_ERRORS
950:   const auto& prop = static_cast<LinalgInvExBackward0*>(self->cdata.get())->inverse_;
951:   return THPVariable_Wrap(prop.unpack(self->cdata));
952:   END_HANDLE_TH_ERRORS
953: }
954: 
955: static PyObject* THPLinalgInvExBackward0_inverse_raw_getter(THPCppFunction *self, void *_unused) {
956:   HANDLE_TH_ERRORS
957:   const auto& prop = static_cast<LinalgInvExBackward0*>(self->cdata.get())->inverse_;
958:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
959:   return obj.release().ptr();
960:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPUnsafeIndexBackward0_indices_getter`, `PyErr_SetString`, `PyTuple_New`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPUnsafeIndexBackward0_indices_getter`, `PyErr_SetString`, `PyTuple_New` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 961-1040

```cpp
 961: }
 962: 
 963: static struct PyGetSetDef LinalgInvExBackward0_properties[] = {
 964:   THP_FUNCTION_DEFAULT_PROPERTIES,
 965:   {(char*)"_saved_inverse", (getter)THPLinalgInvExBackward0_inverse_getter, nullptr, nullptr, nullptr},
 966:   {(char*)"_raw_saved_inverse", (getter)THPLinalgInvExBackward0_inverse_raw_getter, nullptr, nullptr, nullptr},
 967:   {nullptr} /* sentinel */
 968: };
 969: 
 970: static PyObject* THPLerpBackward0_weight_getter(THPCppFunction *self, void *_unused) {
 971:   HANDLE_TH_ERRORS
 972:   auto prop = static_cast<LerpBackward0*>(self->cdata.get())->weight;
 973:   if (prop.isComplex()) {
 974:     auto cprop = prop.to<c10::complex<double>>();
 975:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
 976:   } else if (prop.isFloatingPoint()) {
 977:     return PyFloat_FromDouble(prop.to<double>());
 978:   } else if (prop.isIntegral(/*includeBool=*/false)) {
 979:     return PyLong_FromLong(prop.to<int64_t>());
 980:   } else if (prop.isBoolean()) {
 981:     if (prop.to<bool>()) {
 982:       Py_RETURN_TRUE;
 983:     } else {
 984:       Py_RETURN_FALSE;
 985:     }
 986:   } else {
 987:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
 988:     return nullptr;
 989:   }
 990:   END_HANDLE_TH_ERRORS
 991: }
 992: 
 993: static struct PyGetSetDef LerpBackward0_properties[] = {
 994:   THP_FUNCTION_DEFAULT_PROPERTIES,
 995:   {(char*)"_saved_weight", (getter)THPLerpBackward0_weight_getter, nullptr, nullptr, nullptr},
 996:   {nullptr} /* sentinel */
 997: };
 998: 
 999: static PyObject* THPLerpBackward1_end_getter(THPCppFunction *self, void *_unused) {
1000:   HANDLE_TH_ERRORS
1001:   const auto& prop = static_cast<LerpBackward1*>(self->cdata.get())->end_;
1002:   return THPVariable_Wrap(prop.unpack(self->cdata));
1003:   END_HANDLE_TH_ERRORS
1004: }
1005: 
1006: static PyObject* THPLerpBackward1_end_raw_getter(THPCppFunction *self, void *_unused) {
1007:   HANDLE_TH_ERRORS
1008:   const auto& prop = static_cast<LerpBackward1*>(self->cdata.get())->end_;
1009:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1010:   return obj.release().ptr();
1011:   END_HANDLE_TH_ERRORS
1012: }
1013: 
1014: static PyObject* THPLerpBackward1_self_getter(THPCppFunction *self, void *_unused) {
1015:   HANDLE_TH_ERRORS
1016:   const auto& prop = static_cast<LerpBackward1*>(self->cdata.get())->self_;
1017:   return THPVariable_Wrap(prop.unpack(self->cdata));
1018:   END_HANDLE_TH_ERRORS
1019: }
1020: 
1021: static PyObject* THPLerpBackward1_self_raw_getter(THPCppFunction *self, void *_unused) {
1022:   HANDLE_TH_ERRORS
1023:   const auto& prop = static_cast<LerpBackward1*>(self->cdata.get())->self_;
1024:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1025:   return obj.release().ptr();
1026:   END_HANDLE_TH_ERRORS
1027: }
1028: 
1029: static PyObject* THPLerpBackward1_weight_getter(THPCppFunction *self, void *_unused) {
1030:   HANDLE_TH_ERRORS
1031:   const auto& prop = static_cast<LerpBackward1*>(self->cdata.get())->weight_;
1032:   return THPVariable_Wrap(prop.unpack(self->cdata));
1033:   END_HANDLE_TH_ERRORS
1034: }
1035: 
1036: static PyObject* THPLerpBackward1_weight_raw_getter(THPCppFunction *self, void *_unused) {
1037:   HANDLE_TH_ERRORS
1038:   const auto& prop = static_cast<LerpBackward1*>(self->cdata.get())->weight_;
1039:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1040:   return obj.release().ptr();
```

- EN: The main execution path in this span is carried by `THPLerpBackward0_weight_getter`, `PyComplex_FromDoubles`, `PyFloat_FromDouble`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPLerpBackward0_weight_getter`, `PyComplex_FromDoubles`, `PyFloat_FromDouble` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1041-1120

```cpp
1041:   END_HANDLE_TH_ERRORS
1042: }
1043: 
1044: static struct PyGetSetDef LerpBackward1_properties[] = {
1045:   THP_FUNCTION_DEFAULT_PROPERTIES,
1046:   {(char*)"_saved_end", (getter)THPLerpBackward1_end_getter, nullptr, nullptr, nullptr},
1047:   {(char*)"_raw_saved_end", (getter)THPLerpBackward1_end_raw_getter, nullptr, nullptr, nullptr},
1048:   {(char*)"_saved_self", (getter)THPLerpBackward1_self_getter, nullptr, nullptr, nullptr},
1049:   {(char*)"_raw_saved_self", (getter)THPLerpBackward1_self_raw_getter, nullptr, nullptr, nullptr},
1050:   {(char*)"_saved_weight", (getter)THPLerpBackward1_weight_getter, nullptr, nullptr, nullptr},
1051:   {(char*)"_raw_saved_weight", (getter)THPLerpBackward1_weight_raw_getter, nullptr, nullptr, nullptr},
1052:   {nullptr} /* sentinel */
1053: };
1054: 
1055: static PyObject* THPSpecialXlog1PyBackward0_other_getter(THPCppFunction *self, void *_unused) {
1056:   HANDLE_TH_ERRORS
1057:   const auto& prop = static_cast<SpecialXlog1PyBackward0*>(self->cdata.get())->other_;
1058:   return THPVariable_Wrap(prop.unpack(self->cdata));
1059:   END_HANDLE_TH_ERRORS
1060: }
1061: 
1062: static PyObject* THPSpecialXlog1PyBackward0_other_raw_getter(THPCppFunction *self, void *_unused) {
1063:   HANDLE_TH_ERRORS
1064:   const auto& prop = static_cast<SpecialXlog1PyBackward0*>(self->cdata.get())->other_;
1065:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1066:   return obj.release().ptr();
1067:   END_HANDLE_TH_ERRORS
1068: }
1069: 
1070: static PyObject* THPSpecialXlog1PyBackward0_self_getter(THPCppFunction *self, void *_unused) {
1071:   HANDLE_TH_ERRORS
1072:   const auto& prop = static_cast<SpecialXlog1PyBackward0*>(self->cdata.get())->self_;
1073:   return THPVariable_Wrap(prop.unpack(self->cdata));
1074:   END_HANDLE_TH_ERRORS
1075: }
1076: 
1077: static PyObject* THPSpecialXlog1PyBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
1078:   HANDLE_TH_ERRORS
1079:   const auto& prop = static_cast<SpecialXlog1PyBackward0*>(self->cdata.get())->self_;
1080:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1081:   return obj.release().ptr();
1082:   END_HANDLE_TH_ERRORS
1083: }
1084: 
1085: static struct PyGetSetDef SpecialXlog1PyBackward0_properties[] = {
1086:   THP_FUNCTION_DEFAULT_PROPERTIES,
1087:   {(char*)"_saved_other", (getter)THPSpecialXlog1PyBackward0_other_getter, nullptr, nullptr, nullptr},
1088:   {(char*)"_raw_saved_other", (getter)THPSpecialXlog1PyBackward0_other_raw_getter, nullptr, nullptr, nullptr},
1089:   {(char*)"_saved_self", (getter)THPSpecialXlog1PyBackward0_self_getter, nullptr, nullptr, nullptr},
1090:   {(char*)"_raw_saved_self", (getter)THPSpecialXlog1PyBackward0_self_raw_getter, nullptr, nullptr, nullptr},
1091:   {nullptr} /* sentinel */
1092: };
1093: 
1094: static PyObject* THPSpecialXlog1PyBackward1_other_getter(THPCppFunction *self, void *_unused) {
1095:   HANDLE_TH_ERRORS
1096:   const auto& prop = static_cast<SpecialXlog1PyBackward1*>(self->cdata.get())->other_;
1097:   return THPVariable_Wrap(prop.unpack(self->cdata));
1098:   END_HANDLE_TH_ERRORS
1099: }
1100: 
1101: static PyObject* THPSpecialXlog1PyBackward1_other_raw_getter(THPCppFunction *self, void *_unused) {
1102:   HANDLE_TH_ERRORS
1103:   const auto& prop = static_cast<SpecialXlog1PyBackward1*>(self->cdata.get())->other_;
1104:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1105:   return obj.release().ptr();
1106:   END_HANDLE_TH_ERRORS
1107: }
1108: 
1109: static PyObject* THPSpecialXlog1PyBackward1_self_getter(THPCppFunction *self, void *_unused) {
1110:   HANDLE_TH_ERRORS
1111:   auto prop = static_cast<SpecialXlog1PyBackward1*>(self->cdata.get())->self;
1112:   if (prop.isComplex()) {
1113:     auto cprop = prop.to<c10::complex<double>>();
1114:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
1115:   } else if (prop.isFloatingPoint()) {
1116:     return PyFloat_FromDouble(prop.to<double>());
1117:   } else if (prop.isIntegral(/*includeBool=*/false)) {
1118:     return PyLong_FromLong(prop.to<int64_t>());
1119:   } else if (prop.isBoolean()) {
1120:     if (prop.to<bool>()) {
```

- EN: The main execution path in this span is carried by `THPSpecialXlog1PyBackward0_other_getter`, `THPVariable_Wrap`, `THPSpecialXlog1PyBackward0_other_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPSpecialXlog1PyBackward0_other_getter`, `THPVariable_Wrap`, `THPSpecialXlog1PyBackward0_other_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1121-1200

```cpp
1121:       Py_RETURN_TRUE;
1122:     } else {
1123:       Py_RETURN_FALSE;
1124:     }
1125:   } else {
1126:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
1127:     return nullptr;
1128:   }
1129:   END_HANDLE_TH_ERRORS
1130: }
1131: 
1132: static struct PyGetSetDef SpecialXlog1PyBackward1_properties[] = {
1133:   THP_FUNCTION_DEFAULT_PROPERTIES,
1134:   {(char*)"_saved_other", (getter)THPSpecialXlog1PyBackward1_other_getter, nullptr, nullptr, nullptr},
1135:   {(char*)"_raw_saved_other", (getter)THPSpecialXlog1PyBackward1_other_raw_getter, nullptr, nullptr, nullptr},
1136:   {(char*)"_saved_self", (getter)THPSpecialXlog1PyBackward1_self_getter, nullptr, nullptr, nullptr},
1137:   {nullptr} /* sentinel */
1138: };
1139: 
1140: static PyObject* THPSpecialXlog1PyBackward2_other_getter(THPCppFunction *self, void *_unused) {
1141:   HANDLE_TH_ERRORS
1142:   auto prop = static_cast<SpecialXlog1PyBackward2*>(self->cdata.get())->other;
1143:   if (prop.isComplex()) {
1144:     auto cprop = prop.to<c10::complex<double>>();
1145:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
1146:   } else if (prop.isFloatingPoint()) {
1147:     return PyFloat_FromDouble(prop.to<double>());
1148:   } else if (prop.isIntegral(/*includeBool=*/false)) {
1149:     return PyLong_FromLong(prop.to<int64_t>());
1150:   } else if (prop.isBoolean()) {
1151:     if (prop.to<bool>()) {
1152:       Py_RETURN_TRUE;
1153:     } else {
1154:       Py_RETURN_FALSE;
1155:     }
1156:   } else {
1157:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
1158:     return nullptr;
1159:   }
1160:   END_HANDLE_TH_ERRORS
1161: }
1162: 
1163: static PyObject* THPSpecialXlog1PyBackward2_self_getter(THPCppFunction *self, void *_unused) {
1164:   HANDLE_TH_ERRORS
1165:   const auto& prop = static_cast<SpecialXlog1PyBackward2*>(self->cdata.get())->self_;
1166:   return THPVariable_Wrap(prop.unpack(self->cdata));
1167:   END_HANDLE_TH_ERRORS
1168: }
1169: 
1170: static PyObject* THPSpecialXlog1PyBackward2_self_raw_getter(THPCppFunction *self, void *_unused) {
1171:   HANDLE_TH_ERRORS
1172:   const auto& prop = static_cast<SpecialXlog1PyBackward2*>(self->cdata.get())->self_;
1173:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1174:   return obj.release().ptr();
1175:   END_HANDLE_TH_ERRORS
1176: }
1177: 
1178: static struct PyGetSetDef SpecialXlog1PyBackward2_properties[] = {
1179:   THP_FUNCTION_DEFAULT_PROPERTIES,
1180:   {(char*)"_saved_other", (getter)THPSpecialXlog1PyBackward2_other_getter, nullptr, nullptr, nullptr},
1181:   {(char*)"_saved_self", (getter)THPSpecialXlog1PyBackward2_self_getter, nullptr, nullptr, nullptr},
1182:   {(char*)"_raw_saved_self", (getter)THPSpecialXlog1PyBackward2_self_raw_getter, nullptr, nullptr, nullptr},
1183:   {nullptr} /* sentinel */
1184: };
1185: 
1186: 
1187: 
1188: static struct PyGetSetDef LtBackward0_properties[] = {
1189:   THP_FUNCTION_DEFAULT_PROPERTIES,
1190: 
1191:   {nullptr} /* sentinel */
1192: };
1193: 
1194: 
1195: 
1196: static struct PyGetSetDef LtBackward1_properties[] = {
1197:   THP_FUNCTION_DEFAULT_PROPERTIES,
1198: 
1199:   {nullptr} /* sentinel */
1200: };
```

- EN: The main execution path in this span is carried by `PyErr_SetString`, `THPSpecialXlog1PyBackward2_other_getter`, `PyComplex_FromDoubles`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyErr_SetString`, `THPSpecialXlog1PyBackward2_other_getter`, `PyComplex_FromDoubles` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1201-1280

```cpp
1201: 
1202: static PyObject* THPMeanBackward0_self_sym_numel_getter(THPCppFunction *self, void *_unused) {
1203:   HANDLE_TH_ERRORS
1204:   auto prop = static_cast<MeanBackward0*>(self->cdata.get())->self_sym_numel;
1205:   if (auto m = prop.maybe_as_int()) {
1206:     return PyLong_FromUnsignedLong(*m);
1207:   } else {
1208:     return py::cast(prop).release().ptr();
1209:   }
1210:   END_HANDLE_TH_ERRORS
1211: }
1212: 
1213: static PyObject* THPMeanBackward0_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
1214:   HANDLE_TH_ERRORS
1215:   auto prop = static_cast<MeanBackward0*>(self->cdata.get())->self_sym_sizes;
1216:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
1217:   for (auto i : c10::irange(prop.size())) {
1218:       auto si = prop[i];
1219:       if (auto m = si.maybe_as_int()) {
1220:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
1221:       } else {
1222:         auto py_symint = py::cast(si).release().ptr();
1223:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
1224:       }
1225:   }
1226:   return tup;
1227:   END_HANDLE_TH_ERRORS
1228: }
1229: 
1230: static struct PyGetSetDef MeanBackward0_properties[] = {
1231:   THP_FUNCTION_DEFAULT_PROPERTIES,
1232:   {(char*)"_saved_self_sym_numel", (getter)THPMeanBackward0_self_sym_numel_getter, nullptr, nullptr, nullptr},
1233:   {(char*)"_saved_self_sym_sizes", (getter)THPMeanBackward0_self_sym_sizes_getter, nullptr, nullptr, nullptr},
1234:   {nullptr} /* sentinel */
1235: };
1236: 
1237: static PyObject* THPMeanBackwardAutogradNestedTensor0_self_getter(THPCppFunction *self, void *_unused) {
1238:   HANDLE_TH_ERRORS
1239:   const auto& prop = static_cast<MeanBackwardAutogradNestedTensor0*>(self->cdata.get())->self_;
1240:   return THPVariable_Wrap(prop.unpack(self->cdata));
1241:   END_HANDLE_TH_ERRORS
1242: }
1243: 
1244: static PyObject* THPMeanBackwardAutogradNestedTensor0_self_raw_getter(THPCppFunction *self, void *_unused) {
1245:   HANDLE_TH_ERRORS
1246:   const auto& prop = static_cast<MeanBackwardAutogradNestedTensor0*>(self->cdata.get())->self_;
1247:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1248:   return obj.release().ptr();
1249:   END_HANDLE_TH_ERRORS
1250: }
1251: 
1252: static PyObject* THPMeanBackwardAutogradNestedTensor0_self_sym_numel_getter(THPCppFunction *self, void *_unused) {
1253:   HANDLE_TH_ERRORS
1254:   auto prop = static_cast<MeanBackwardAutogradNestedTensor0*>(self->cdata.get())->self_sym_numel;
1255:   if (auto m = prop.maybe_as_int()) {
1256:     return PyLong_FromUnsignedLong(*m);
1257:   } else {
1258:     return py::cast(prop).release().ptr();
1259:   }
1260:   END_HANDLE_TH_ERRORS
1261: }
1262: 
1263: static struct PyGetSetDef MeanBackwardAutogradNestedTensor0_properties[] = {
1264:   THP_FUNCTION_DEFAULT_PROPERTIES,
1265:   {(char*)"_saved_self", (getter)THPMeanBackwardAutogradNestedTensor0_self_getter, nullptr, nullptr, nullptr},
1266:   {(char*)"_raw_saved_self", (getter)THPMeanBackwardAutogradNestedTensor0_self_raw_getter, nullptr, nullptr, nullptr},
1267:   {(char*)"_saved_self_sym_numel", (getter)THPMeanBackwardAutogradNestedTensor0_self_sym_numel_getter, nullptr, nullptr, nullptr},
1268:   {nullptr} /* sentinel */
1269: };
1270: 
1271: static PyObject* THPMeanBackward1_dim_getter(THPCppFunction *self, void *_unused) {
1272:   HANDLE_TH_ERRORS
1273:   auto opt_prop = static_cast<MeanBackward1*>(self->cdata.get())->dim;
1274:   if (!opt_prop.list.has_value()) {
1275:     Py_RETURN_NONE;
1276:   }
1277:   auto prop = opt_prop.list.value();
1278:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
1279:   for (auto i : c10::irange(prop.size())) {
1280:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
```

- EN: The main execution path in this span is carried by `THPMeanBackward0_self_sym_numel_getter`, `PyLong_FromUnsignedLong`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPMeanBackward0_self_sym_numel_getter`, `PyLong_FromUnsignedLong`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 1281-1360

```cpp
1281:   }
1282:   return tup;
1283:   END_HANDLE_TH_ERRORS
1284: }
1285: 
1286: static PyObject* THPMeanBackward1_keepdim_getter(THPCppFunction *self, void *_unused) {
1287:   HANDLE_TH_ERRORS
1288:   auto prop = static_cast<MeanBackward1*>(self->cdata.get())->keepdim;
1289:   if (prop) {
1290:     Py_RETURN_TRUE;
1291:   } else {
1292:     Py_RETURN_FALSE;
1293:   }
1294:   END_HANDLE_TH_ERRORS
1295: }
1296: 
1297: static PyObject* THPMeanBackward1_self_sym_numel_getter(THPCppFunction *self, void *_unused) {
1298:   HANDLE_TH_ERRORS
1299:   auto prop = static_cast<MeanBackward1*>(self->cdata.get())->self_sym_numel;
1300:   if (auto m = prop.maybe_as_int()) {
1301:     return PyLong_FromUnsignedLong(*m);
1302:   } else {
1303:     return py::cast(prop).release().ptr();
1304:   }
1305:   END_HANDLE_TH_ERRORS
1306: }
1307: 
1308: static PyObject* THPMeanBackward1_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
1309:   HANDLE_TH_ERRORS
1310:   auto prop = static_cast<MeanBackward1*>(self->cdata.get())->self_sym_sizes;
1311:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
1312:   for (auto i : c10::irange(prop.size())) {
1313:       auto si = prop[i];
1314:       if (auto m = si.maybe_as_int()) {
1315:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
1316:       } else {
1317:         auto py_symint = py::cast(si).release().ptr();
1318:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
1319:       }
1320:   }
1321:   return tup;
1322:   END_HANDLE_TH_ERRORS
1323: }
1324: 
1325: static struct PyGetSetDef MeanBackward1_properties[] = {
1326:   THP_FUNCTION_DEFAULT_PROPERTIES,
1327:   {(char*)"_saved_dim", (getter)THPMeanBackward1_dim_getter, nullptr, nullptr, nullptr},
1328:   {(char*)"_saved_keepdim", (getter)THPMeanBackward1_keepdim_getter, nullptr, nullptr, nullptr},
1329:   {(char*)"_saved_self_sym_numel", (getter)THPMeanBackward1_self_sym_numel_getter, nullptr, nullptr, nullptr},
1330:   {(char*)"_saved_self_sym_sizes", (getter)THPMeanBackward1_self_sym_sizes_getter, nullptr, nullptr, nullptr},
1331:   {nullptr} /* sentinel */
1332: };
1333: 
1334: static PyObject* THPMinBackward0_dim_getter(THPCppFunction *self, void *_unused) {
1335:   HANDLE_TH_ERRORS
1336:   auto prop = static_cast<MinBackward0*>(self->cdata.get())->dim;
1337:   return PyLong_FromUnsignedLong((int64_t) prop);
1338:   END_HANDLE_TH_ERRORS
1339: }
1340: 
1341: static PyObject* THPMinBackward0_keepdim_getter(THPCppFunction *self, void *_unused) {
1342:   HANDLE_TH_ERRORS
1343:   auto prop = static_cast<MinBackward0*>(self->cdata.get())->keepdim;
1344:   if (prop) {
1345:     Py_RETURN_TRUE;
1346:   } else {
1347:     Py_RETURN_FALSE;
1348:   }
1349:   END_HANDLE_TH_ERRORS
1350: }
1351: 
1352: static PyObject* THPMinBackward0_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
1353:   HANDLE_TH_ERRORS
1354:   auto prop = static_cast<MinBackward0*>(self->cdata.get())->self_sym_sizes;
1355:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
1356:   for (auto i : c10::irange(prop.size())) {
1357:       auto si = prop[i];
1358:       if (auto m = si.maybe_as_int()) {
1359:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
1360:       } else {
```

- EN: The main execution path in this span is carried by `THPMeanBackward1_keepdim_getter`, `THPMeanBackward1_self_sym_numel_getter`, `PyLong_FromUnsignedLong`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPMeanBackward1_keepdim_getter`, `THPMeanBackward1_self_sym_numel_getter`, `PyLong_FromUnsignedLong` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1361-1440

```cpp
1361:         auto py_symint = py::cast(si).release().ptr();
1362:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
1363:       }
1364:   }
1365:   return tup;
1366:   END_HANDLE_TH_ERRORS
1367: }
1368: 
1369: static PyObject* THPMinBackward0_indices_getter(THPCppFunction *self, void *_unused) {
1370:   HANDLE_TH_ERRORS
1371:   const auto& prop = static_cast<MinBackward0*>(self->cdata.get())->indices_;
1372:   return THPVariable_Wrap(prop.unpack(self->cdata));
1373:   END_HANDLE_TH_ERRORS
1374: }
1375: 
1376: static PyObject* THPMinBackward0_indices_raw_getter(THPCppFunction *self, void *_unused) {
1377:   HANDLE_TH_ERRORS
1378:   const auto& prop = static_cast<MinBackward0*>(self->cdata.get())->indices_;
1379:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1380:   return obj.release().ptr();
1381:   END_HANDLE_TH_ERRORS
1382: }
1383: 
1384: static struct PyGetSetDef MinBackward0_properties[] = {
1385:   THP_FUNCTION_DEFAULT_PROPERTIES,
1386:   {(char*)"_saved_dim", (getter)THPMinBackward0_dim_getter, nullptr, nullptr, nullptr},
1387:   {(char*)"_saved_keepdim", (getter)THPMinBackward0_keepdim_getter, nullptr, nullptr, nullptr},
1388:   {(char*)"_saved_self_sym_sizes", (getter)THPMinBackward0_self_sym_sizes_getter, nullptr, nullptr, nullptr},
1389:   {(char*)"_saved_indices", (getter)THPMinBackward0_indices_getter, nullptr, nullptr, nullptr},
1390:   {(char*)"_raw_saved_indices", (getter)THPMinBackward0_indices_raw_getter, nullptr, nullptr, nullptr},
1391:   {nullptr} /* sentinel */
1392: };
1393: 
1394: static PyObject* THPMinBackward1_self_getter(THPCppFunction *self, void *_unused) {
1395:   HANDLE_TH_ERRORS
1396:   const auto& prop = static_cast<MinBackward1*>(self->cdata.get())->self_;
1397:   return THPVariable_Wrap(prop.unpack(self->cdata));
1398:   END_HANDLE_TH_ERRORS
1399: }
1400: 
1401: static PyObject* THPMinBackward1_self_raw_getter(THPCppFunction *self, void *_unused) {
1402:   HANDLE_TH_ERRORS
1403:   const auto& prop = static_cast<MinBackward1*>(self->cdata.get())->self_;
1404:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1405:   return obj.release().ptr();
1406:   END_HANDLE_TH_ERRORS
1407: }
1408: 
1409: static PyObject* THPMinBackward1_result_getter(THPCppFunction *self, void *_unused) {
1410:   HANDLE_TH_ERRORS
1411:   const auto& prop = static_cast<MinBackward1*>(self->cdata.get())->result_;
1412:   return THPVariable_Wrap(prop.unpack(self->cdata));
1413:   END_HANDLE_TH_ERRORS
1414: }
1415: 
1416: static PyObject* THPMinBackward1_result_raw_getter(THPCppFunction *self, void *_unused) {
1417:   HANDLE_TH_ERRORS
1418:   const auto& prop = static_cast<MinBackward1*>(self->cdata.get())->result_;
1419:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1420:   return obj.release().ptr();
1421:   END_HANDLE_TH_ERRORS
1422: }
1423: 
1424: static struct PyGetSetDef MinBackward1_properties[] = {
1425:   THP_FUNCTION_DEFAULT_PROPERTIES,
1426:   {(char*)"_saved_self", (getter)THPMinBackward1_self_getter, nullptr, nullptr, nullptr},
1427:   {(char*)"_raw_saved_self", (getter)THPMinBackward1_self_raw_getter, nullptr, nullptr, nullptr},
1428:   {(char*)"_saved_result", (getter)THPMinBackward1_result_getter, nullptr, nullptr, nullptr},
1429:   {(char*)"_raw_saved_result", (getter)THPMinBackward1_result_raw_getter, nullptr, nullptr, nullptr},
1430:   {nullptr} /* sentinel */
1431: };
1432: 
1433: static PyObject* THPFminBackward0_other_getter(THPCppFunction *self, void *_unused) {
1434:   HANDLE_TH_ERRORS
1435:   const auto& prop = static_cast<FminBackward0*>(self->cdata.get())->other_;
1436:   return THPVariable_Wrap(prop.unpack(self->cdata));
1437:   END_HANDLE_TH_ERRORS
1438: }
1439: 
1440: static PyObject* THPFminBackward0_other_raw_getter(THPCppFunction *self, void *_unused) {
```

- EN: The main execution path in this span is carried by `cast`, `PyTuple_SetItem`, `THPMinBackward0_indices_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `cast`, `PyTuple_SetItem`, `THPMinBackward0_indices_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1441-1520

```cpp
1441:   HANDLE_TH_ERRORS
1442:   const auto& prop = static_cast<FminBackward0*>(self->cdata.get())->other_;
1443:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1444:   return obj.release().ptr();
1445:   END_HANDLE_TH_ERRORS
1446: }
1447: 
1448: static PyObject* THPFminBackward0_self_getter(THPCppFunction *self, void *_unused) {
1449:   HANDLE_TH_ERRORS
1450:   const auto& prop = static_cast<FminBackward0*>(self->cdata.get())->self_;
1451:   return THPVariable_Wrap(prop.unpack(self->cdata));
1452:   END_HANDLE_TH_ERRORS
1453: }
1454: 
1455: static PyObject* THPFminBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
1456:   HANDLE_TH_ERRORS
1457:   const auto& prop = static_cast<FminBackward0*>(self->cdata.get())->self_;
1458:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1459:   return obj.release().ptr();
1460:   END_HANDLE_TH_ERRORS
1461: }
1462: 
1463: static struct PyGetSetDef FminBackward0_properties[] = {
1464:   THP_FUNCTION_DEFAULT_PROPERTIES,
1465:   {(char*)"_saved_other", (getter)THPFminBackward0_other_getter, nullptr, nullptr, nullptr},
1466:   {(char*)"_raw_saved_other", (getter)THPFminBackward0_other_raw_getter, nullptr, nullptr, nullptr},
1467:   {(char*)"_saved_self", (getter)THPFminBackward0_self_getter, nullptr, nullptr, nullptr},
1468:   {(char*)"_raw_saved_self", (getter)THPFminBackward0_self_raw_getter, nullptr, nullptr, nullptr},
1469:   {nullptr} /* sentinel */
1470: };
1471: 
1472: static PyObject* THPMmBackward0_mat2_getter(THPCppFunction *self, void *_unused) {
1473:   HANDLE_TH_ERRORS
1474:   const auto& prop = static_cast<MmBackward0*>(self->cdata.get())->mat2_;
1475:   return THPVariable_Wrap(prop.unpack(self->cdata));
1476:   END_HANDLE_TH_ERRORS
1477: }
1478: 
1479: static PyObject* THPMmBackward0_mat2_raw_getter(THPCppFunction *self, void *_unused) {
1480:   HANDLE_TH_ERRORS
1481:   const auto& prop = static_cast<MmBackward0*>(self->cdata.get())->mat2_;
1482:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1483:   return obj.release().ptr();
1484:   END_HANDLE_TH_ERRORS
1485: }
1486: 
1487: static PyObject* THPMmBackward0_mat2_sym_sizes_getter(THPCppFunction *self, void *_unused) {
1488:   HANDLE_TH_ERRORS
1489:   auto prop = static_cast<MmBackward0*>(self->cdata.get())->mat2_sym_sizes;
1490:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
1491:   for (auto i : c10::irange(prop.size())) {
1492:       auto si = prop[i];
1493:       if (auto m = si.maybe_as_int()) {
1494:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
1495:       } else {
1496:         auto py_symint = py::cast(si).release().ptr();
1497:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
1498:       }
1499:   }
1500:   return tup;
1501:   END_HANDLE_TH_ERRORS
1502: }
1503: 
1504: static PyObject* THPMmBackward0_mat2_sym_strides_getter(THPCppFunction *self, void *_unused) {
1505:   HANDLE_TH_ERRORS
1506:   auto prop = static_cast<MmBackward0*>(self->cdata.get())->mat2_sym_strides;
1507:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
1508:   for (auto i : c10::irange(prop.size())) {
1509:       auto si = prop[i];
1510:       if (auto m = si.maybe_as_int()) {
1511:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
1512:       } else {
1513:         auto py_symint = py::cast(si).release().ptr();
1514:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
1515:       }
1516:   }
1517:   return tup;
1518:   END_HANDLE_TH_ERRORS
1519: }
1520: 
```

- EN: The main execution path in this span is carried by `cast`, `THPFminBackward0_self_getter`, `THPVariable_Wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `cast`, `THPFminBackward0_self_getter`, `THPVariable_Wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1521-1600

```cpp
1521: static PyObject* THPMmBackward0_self_getter(THPCppFunction *self, void *_unused) {
1522:   HANDLE_TH_ERRORS
1523:   const auto& prop = static_cast<MmBackward0*>(self->cdata.get())->self_;
1524:   return THPVariable_Wrap(prop.unpack(self->cdata));
1525:   END_HANDLE_TH_ERRORS
1526: }
1527: 
1528: static PyObject* THPMmBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
1529:   HANDLE_TH_ERRORS
1530:   const auto& prop = static_cast<MmBackward0*>(self->cdata.get())->self_;
1531:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1532:   return obj.release().ptr();
1533:   END_HANDLE_TH_ERRORS
1534: }
1535: 
1536: static PyObject* THPMmBackward0_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
1537:   HANDLE_TH_ERRORS
1538:   auto prop = static_cast<MmBackward0*>(self->cdata.get())->self_sym_sizes;
1539:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
1540:   for (auto i : c10::irange(prop.size())) {
1541:       auto si = prop[i];
1542:       if (auto m = si.maybe_as_int()) {
1543:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
1544:       } else {
1545:         auto py_symint = py::cast(si).release().ptr();
1546:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
1547:       }
1548:   }
1549:   return tup;
1550:   END_HANDLE_TH_ERRORS
1551: }
1552: 
1553: static PyObject* THPMmBackward0_self_sym_strides_getter(THPCppFunction *self, void *_unused) {
1554:   HANDLE_TH_ERRORS
1555:   auto prop = static_cast<MmBackward0*>(self->cdata.get())->self_sym_strides;
1556:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
1557:   for (auto i : c10::irange(prop.size())) {
1558:       auto si = prop[i];
1559:       if (auto m = si.maybe_as_int()) {
1560:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
1561:       } else {
1562:         auto py_symint = py::cast(si).release().ptr();
1563:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
1564:       }
1565:   }
1566:   return tup;
1567:   END_HANDLE_TH_ERRORS
1568: }
1569: 
1570: static struct PyGetSetDef MmBackward0_properties[] = {
1571:   THP_FUNCTION_DEFAULT_PROPERTIES,
1572:   {(char*)"_saved_mat2", (getter)THPMmBackward0_mat2_getter, nullptr, nullptr, nullptr},
1573:   {(char*)"_raw_saved_mat2", (getter)THPMmBackward0_mat2_raw_getter, nullptr, nullptr, nullptr},
1574:   {(char*)"_saved_mat2_sym_sizes", (getter)THPMmBackward0_mat2_sym_sizes_getter, nullptr, nullptr, nullptr},
1575:   {(char*)"_saved_mat2_sym_strides", (getter)THPMmBackward0_mat2_sym_strides_getter, nullptr, nullptr, nullptr},
1576:   {(char*)"_saved_self", (getter)THPMmBackward0_self_getter, nullptr, nullptr, nullptr},
1577:   {(char*)"_raw_saved_self", (getter)THPMmBackward0_self_raw_getter, nullptr, nullptr, nullptr},
1578:   {(char*)"_saved_self_sym_sizes", (getter)THPMmBackward0_self_sym_sizes_getter, nullptr, nullptr, nullptr},
1579:   {(char*)"_saved_self_sym_strides", (getter)THPMmBackward0_self_sym_strides_getter, nullptr, nullptr, nullptr},
1580:   {nullptr} /* sentinel */
1581: };
1582: 
1583: static PyObject* THPMvBackward0_self_getter(THPCppFunction *self, void *_unused) {
1584:   HANDLE_TH_ERRORS
1585:   const auto& prop = static_cast<MvBackward0*>(self->cdata.get())->self_;
1586:   return THPVariable_Wrap(prop.unpack(self->cdata));
1587:   END_HANDLE_TH_ERRORS
1588: }
1589: 
1590: static PyObject* THPMvBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
1591:   HANDLE_TH_ERRORS
1592:   const auto& prop = static_cast<MvBackward0*>(self->cdata.get())->self_;
1593:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1594:   return obj.release().ptr();
1595:   END_HANDLE_TH_ERRORS
1596: }
1597: 
1598: static PyObject* THPMvBackward0_vec_getter(THPCppFunction *self, void *_unused) {
1599:   HANDLE_TH_ERRORS
1600:   const auto& prop = static_cast<MvBackward0*>(self->cdata.get())->vec_;
```

- EN: The main execution path in this span is carried by `THPMmBackward0_self_getter`, `THPVariable_Wrap`, `THPMmBackward0_self_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPMmBackward0_self_getter`, `THPVariable_Wrap`, `THPMmBackward0_self_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1601-1680

```cpp
1601:   return THPVariable_Wrap(prop.unpack(self->cdata));
1602:   END_HANDLE_TH_ERRORS
1603: }
1604: 
1605: static PyObject* THPMvBackward0_vec_raw_getter(THPCppFunction *self, void *_unused) {
1606:   HANDLE_TH_ERRORS
1607:   const auto& prop = static_cast<MvBackward0*>(self->cdata.get())->vec_;
1608:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1609:   return obj.release().ptr();
1610:   END_HANDLE_TH_ERRORS
1611: }
1612: 
1613: static struct PyGetSetDef MvBackward0_properties[] = {
1614:   THP_FUNCTION_DEFAULT_PROPERTIES,
1615:   {(char*)"_saved_self", (getter)THPMvBackward0_self_getter, nullptr, nullptr, nullptr},
1616:   {(char*)"_raw_saved_self", (getter)THPMvBackward0_self_raw_getter, nullptr, nullptr, nullptr},
1617:   {(char*)"_saved_vec", (getter)THPMvBackward0_vec_getter, nullptr, nullptr, nullptr},
1618:   {(char*)"_raw_saved_vec", (getter)THPMvBackward0_vec_raw_getter, nullptr, nullptr, nullptr},
1619:   {nullptr} /* sentinel */
1620: };
1621: 
1622: static PyObject* THPNativeBatchNormLegitNoTrainingBackward0_eps_getter(THPCppFunction *self, void *_unused) {
1623:   HANDLE_TH_ERRORS
1624:   auto prop = static_cast<NativeBatchNormLegitNoTrainingBackward0*>(self->cdata.get())->eps;
1625:   return PyFloat_FromDouble((double) prop);
1626:   END_HANDLE_TH_ERRORS
1627: }
1628: 
1629: static PyObject* THPNativeBatchNormLegitNoTrainingBackward0_input_getter(THPCppFunction *self, void *_unused) {
1630:   HANDLE_TH_ERRORS
1631:   const auto& prop = static_cast<NativeBatchNormLegitNoTrainingBackward0*>(self->cdata.get())->input_;
1632:   return THPVariable_Wrap(prop.unpack(self->cdata));
1633:   END_HANDLE_TH_ERRORS
1634: }
1635: 
1636: static PyObject* THPNativeBatchNormLegitNoTrainingBackward0_input_raw_getter(THPCppFunction *self, void *_unused) {
1637:   HANDLE_TH_ERRORS
1638:   const auto& prop = static_cast<NativeBatchNormLegitNoTrainingBackward0*>(self->cdata.get())->input_;
1639:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1640:   return obj.release().ptr();
1641:   END_HANDLE_TH_ERRORS
1642: }
1643: 
1644: static PyObject* THPNativeBatchNormLegitNoTrainingBackward0_running_mean_getter(THPCppFunction *self, void *_unused) {
1645:   HANDLE_TH_ERRORS
1646:   const auto& prop = static_cast<NativeBatchNormLegitNoTrainingBackward0*>(self->cdata.get())->running_mean_;
1647:   return THPVariable_Wrap(prop.unpack(self->cdata));
1648:   END_HANDLE_TH_ERRORS
1649: }
1650: 
1651: static PyObject* THPNativeBatchNormLegitNoTrainingBackward0_running_mean_raw_getter(THPCppFunction *self, void *_unused) {
1652:   HANDLE_TH_ERRORS
1653:   const auto& prop = static_cast<NativeBatchNormLegitNoTrainingBackward0*>(self->cdata.get())->running_mean_;
1654:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1655:   return obj.release().ptr();
1656:   END_HANDLE_TH_ERRORS
1657: }
1658: 
1659: static PyObject* THPNativeBatchNormLegitNoTrainingBackward0_running_var_getter(THPCppFunction *self, void *_unused) {
1660:   HANDLE_TH_ERRORS
1661:   const auto& prop = static_cast<NativeBatchNormLegitNoTrainingBackward0*>(self->cdata.get())->running_var_;
1662:   return THPVariable_Wrap(prop.unpack(self->cdata));
1663:   END_HANDLE_TH_ERRORS
1664: }
1665: 
1666: static PyObject* THPNativeBatchNormLegitNoTrainingBackward0_running_var_raw_getter(THPCppFunction *self, void *_unused) {
1667:   HANDLE_TH_ERRORS
1668:   const auto& prop = static_cast<NativeBatchNormLegitNoTrainingBackward0*>(self->cdata.get())->running_var_;
1669:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1670:   return obj.release().ptr();
1671:   END_HANDLE_TH_ERRORS
1672: }
1673: 
1674: static PyObject* THPNativeBatchNormLegitNoTrainingBackward0_weight_getter(THPCppFunction *self, void *_unused) {
1675:   HANDLE_TH_ERRORS
1676:   const auto& prop = static_cast<NativeBatchNormLegitNoTrainingBackward0*>(self->cdata.get())->weight_;
1677:   return THPVariable_Wrap(prop.unpack(self->cdata));
1678:   END_HANDLE_TH_ERRORS
1679: }
1680: 
```

- EN: The main execution path in this span is carried by `THPVariable_Wrap`, `THPMvBackward0_vec_raw_getter`, `cast`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_Wrap`, `THPMvBackward0_vec_raw_getter`, `cast` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1681-1760

```cpp
1681: static PyObject* THPNativeBatchNormLegitNoTrainingBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
1682:   HANDLE_TH_ERRORS
1683:   const auto& prop = static_cast<NativeBatchNormLegitNoTrainingBackward0*>(self->cdata.get())->weight_;
1684:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1685:   return obj.release().ptr();
1686:   END_HANDLE_TH_ERRORS
1687: }
1688: 
1689: static PyObject* THPNativeBatchNormLegitNoTrainingBackward0_result1_getter(THPCppFunction *self, void *_unused) {
1690:   HANDLE_TH_ERRORS
1691:   const auto& prop = static_cast<NativeBatchNormLegitNoTrainingBackward0*>(self->cdata.get())->result1_;
1692:   return THPVariable_Wrap(prop.unpack(self->cdata));
1693:   END_HANDLE_TH_ERRORS
1694: }
1695: 
1696: static PyObject* THPNativeBatchNormLegitNoTrainingBackward0_result1_raw_getter(THPCppFunction *self, void *_unused) {
1697:   HANDLE_TH_ERRORS
1698:   const auto& prop = static_cast<NativeBatchNormLegitNoTrainingBackward0*>(self->cdata.get())->result1_;
1699:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1700:   return obj.release().ptr();
1701:   END_HANDLE_TH_ERRORS
1702: }
1703: 
1704: static PyObject* THPNativeBatchNormLegitNoTrainingBackward0_result2_getter(THPCppFunction *self, void *_unused) {
1705:   HANDLE_TH_ERRORS
1706:   const auto& prop = static_cast<NativeBatchNormLegitNoTrainingBackward0*>(self->cdata.get())->result2_;
1707:   return THPVariable_Wrap(prop.unpack(self->cdata));
1708:   END_HANDLE_TH_ERRORS
1709: }
1710: 
1711: static PyObject* THPNativeBatchNormLegitNoTrainingBackward0_result2_raw_getter(THPCppFunction *self, void *_unused) {
1712:   HANDLE_TH_ERRORS
1713:   const auto& prop = static_cast<NativeBatchNormLegitNoTrainingBackward0*>(self->cdata.get())->result2_;
1714:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1715:   return obj.release().ptr();
1716:   END_HANDLE_TH_ERRORS
1717: }
1718: 
1719: static struct PyGetSetDef NativeBatchNormLegitNoTrainingBackward0_properties[] = {
1720:   THP_FUNCTION_DEFAULT_PROPERTIES,
1721:   {(char*)"_saved_eps", (getter)THPNativeBatchNormLegitNoTrainingBackward0_eps_getter, nullptr, nullptr, nullptr},
1722:   {(char*)"_saved_input", (getter)THPNativeBatchNormLegitNoTrainingBackward0_input_getter, nullptr, nullptr, nullptr},
1723:   {(char*)"_raw_saved_input", (getter)THPNativeBatchNormLegitNoTrainingBackward0_input_raw_getter, nullptr, nullptr, nullptr},
1724:   {(char*)"_saved_running_mean", (getter)THPNativeBatchNormLegitNoTrainingBackward0_running_mean_getter, nullptr, nullptr, nullptr},
1725:   {(char*)"_raw_saved_running_mean", (getter)THPNativeBatchNormLegitNoTrainingBackward0_running_mean_raw_getter, nullptr, nullptr, nullptr},
1726:   {(char*)"_saved_running_var", (getter)THPNativeBatchNormLegitNoTrainingBackward0_running_var_getter, nullptr, nullptr, nullptr},
1727:   {(char*)"_raw_saved_running_var", (getter)THPNativeBatchNormLegitNoTrainingBackward0_running_var_raw_getter, nullptr, nullptr, nullptr},
1728:   {(char*)"_saved_weight", (getter)THPNativeBatchNormLegitNoTrainingBackward0_weight_getter, nullptr, nullptr, nullptr},
1729:   {(char*)"_raw_saved_weight", (getter)THPNativeBatchNormLegitNoTrainingBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
1730:   {(char*)"_saved_result1", (getter)THPNativeBatchNormLegitNoTrainingBackward0_result1_getter, nullptr, nullptr, nullptr},
1731:   {(char*)"_raw_saved_result1", (getter)THPNativeBatchNormLegitNoTrainingBackward0_result1_raw_getter, nullptr, nullptr, nullptr},
1732:   {(char*)"_saved_result2", (getter)THPNativeBatchNormLegitNoTrainingBackward0_result2_getter, nullptr, nullptr, nullptr},
1733:   {(char*)"_raw_saved_result2", (getter)THPNativeBatchNormLegitNoTrainingBackward0_result2_raw_getter, nullptr, nullptr, nullptr},
1734:   {nullptr} /* sentinel */
1735: };
1736: 
1737: static PyObject* THPNativeGroupNormBackward0_C_getter(THPCppFunction *self, void *_unused) {
1738:   HANDLE_TH_ERRORS
1739:   auto prop = static_cast<NativeGroupNormBackward0*>(self->cdata.get())->C;
1740:   if (auto m = prop.maybe_as_int()) {
1741:     return PyLong_FromUnsignedLong(*m);
1742:   } else {
1743:     return py::cast(prop).release().ptr();
1744:   }
1745:   END_HANDLE_TH_ERRORS
1746: }
1747: 
1748: static PyObject* THPNativeGroupNormBackward0_HxW_getter(THPCppFunction *self, void *_unused) {
1749:   HANDLE_TH_ERRORS
1750:   auto prop = static_cast<NativeGroupNormBackward0*>(self->cdata.get())->HxW;
1751:   if (auto m = prop.maybe_as_int()) {
1752:     return PyLong_FromUnsignedLong(*m);
1753:   } else {
1754:     return py::cast(prop).release().ptr();
1755:   }
1756:   END_HANDLE_TH_ERRORS
1757: }
1758: 
1759: static PyObject* THPNativeGroupNormBackward0_N_getter(THPCppFunction *self, void *_unused) {
1760:   HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPNativeBatchNormLegitNoTrainingBackward0_weight_raw_getter`, `cast`, `THPNativeBatchNormLegitNoTrainingBackward0_result1_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPNativeBatchNormLegitNoTrainingBackward0_weight_raw_getter`, `cast`, `THPNativeBatchNormLegitNoTrainingBackward0_result1_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1761-1840

```cpp
1761:   auto prop = static_cast<NativeGroupNormBackward0*>(self->cdata.get())->N;
1762:   if (auto m = prop.maybe_as_int()) {
1763:     return PyLong_FromUnsignedLong(*m);
1764:   } else {
1765:     return py::cast(prop).release().ptr();
1766:   }
1767:   END_HANDLE_TH_ERRORS
1768: }
1769: 
1770: static PyObject* THPNativeGroupNormBackward0_eps_getter(THPCppFunction *self, void *_unused) {
1771:   HANDLE_TH_ERRORS
1772:   auto prop = static_cast<NativeGroupNormBackward0*>(self->cdata.get())->eps;
1773:   return PyFloat_FromDouble((double) prop);
1774:   END_HANDLE_TH_ERRORS
1775: }
1776: 
1777: static PyObject* THPNativeGroupNormBackward0_group_getter(THPCppFunction *self, void *_unused) {
1778:   HANDLE_TH_ERRORS
1779:   auto prop = static_cast<NativeGroupNormBackward0*>(self->cdata.get())->group;
1780:   return PyLong_FromUnsignedLong((int64_t) prop);
1781:   END_HANDLE_TH_ERRORS
1782: }
1783: 
1784: static PyObject* THPNativeGroupNormBackward0_input_getter(THPCppFunction *self, void *_unused) {
1785:   HANDLE_TH_ERRORS
1786:   const auto& prop = static_cast<NativeGroupNormBackward0*>(self->cdata.get())->input_;
1787:   return THPVariable_Wrap(prop.unpack(self->cdata));
1788:   END_HANDLE_TH_ERRORS
1789: }
1790: 
1791: static PyObject* THPNativeGroupNormBackward0_input_raw_getter(THPCppFunction *self, void *_unused) {
1792:   HANDLE_TH_ERRORS
1793:   const auto& prop = static_cast<NativeGroupNormBackward0*>(self->cdata.get())->input_;
1794:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1795:   return obj.release().ptr();
1796:   END_HANDLE_TH_ERRORS
1797: }
1798: 
1799: static PyObject* THPNativeGroupNormBackward0_weight_getter(THPCppFunction *self, void *_unused) {
1800:   HANDLE_TH_ERRORS
1801:   const auto& prop = static_cast<NativeGroupNormBackward0*>(self->cdata.get())->weight_;
1802:   return THPVariable_Wrap(prop.unpack(self->cdata));
1803:   END_HANDLE_TH_ERRORS
1804: }
1805: 
1806: static PyObject* THPNativeGroupNormBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
1807:   HANDLE_TH_ERRORS
1808:   const auto& prop = static_cast<NativeGroupNormBackward0*>(self->cdata.get())->weight_;
1809:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1810:   return obj.release().ptr();
1811:   END_HANDLE_TH_ERRORS
1812: }
1813: 
1814: static PyObject* THPNativeGroupNormBackward0_result1_getter(THPCppFunction *self, void *_unused) {
1815:   HANDLE_TH_ERRORS
1816:   const auto& prop = static_cast<NativeGroupNormBackward0*>(self->cdata.get())->result1_;
1817:   return THPVariable_Wrap(prop.unpack(self->cdata));
1818:   END_HANDLE_TH_ERRORS
1819: }
1820: 
1821: static PyObject* THPNativeGroupNormBackward0_result1_raw_getter(THPCppFunction *self, void *_unused) {
1822:   HANDLE_TH_ERRORS
1823:   const auto& prop = static_cast<NativeGroupNormBackward0*>(self->cdata.get())->result1_;
1824:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1825:   return obj.release().ptr();
1826:   END_HANDLE_TH_ERRORS
1827: }
1828: 
1829: static PyObject* THPNativeGroupNormBackward0_result2_getter(THPCppFunction *self, void *_unused) {
1830:   HANDLE_TH_ERRORS
1831:   const auto& prop = static_cast<NativeGroupNormBackward0*>(self->cdata.get())->result2_;
1832:   return THPVariable_Wrap(prop.unpack(self->cdata));
1833:   END_HANDLE_TH_ERRORS
1834: }
1835: 
1836: static PyObject* THPNativeGroupNormBackward0_result2_raw_getter(THPCppFunction *self, void *_unused) {
1837:   HANDLE_TH_ERRORS
1838:   const auto& prop = static_cast<NativeGroupNormBackward0*>(self->cdata.get())->result2_;
1839:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1840:   return obj.release().ptr();
```

- EN: The main execution path in this span is carried by `PyLong_FromUnsignedLong`, `cast`, `THPNativeGroupNormBackward0_eps_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyLong_FromUnsignedLong`, `cast`, `THPNativeGroupNormBackward0_eps_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1841-1920

```cpp
1841:   END_HANDLE_TH_ERRORS
1842: }
1843: 
1844: static struct PyGetSetDef NativeGroupNormBackward0_properties[] = {
1845:   THP_FUNCTION_DEFAULT_PROPERTIES,
1846:   {(char*)"_saved_C", (getter)THPNativeGroupNormBackward0_C_getter, nullptr, nullptr, nullptr},
1847:   {(char*)"_saved_HxW", (getter)THPNativeGroupNormBackward0_HxW_getter, nullptr, nullptr, nullptr},
1848:   {(char*)"_saved_N", (getter)THPNativeGroupNormBackward0_N_getter, nullptr, nullptr, nullptr},
1849:   {(char*)"_saved_eps", (getter)THPNativeGroupNormBackward0_eps_getter, nullptr, nullptr, nullptr},
1850:   {(char*)"_saved_group", (getter)THPNativeGroupNormBackward0_group_getter, nullptr, nullptr, nullptr},
1851:   {(char*)"_saved_input", (getter)THPNativeGroupNormBackward0_input_getter, nullptr, nullptr, nullptr},
1852:   {(char*)"_raw_saved_input", (getter)THPNativeGroupNormBackward0_input_raw_getter, nullptr, nullptr, nullptr},
1853:   {(char*)"_saved_weight", (getter)THPNativeGroupNormBackward0_weight_getter, nullptr, nullptr, nullptr},
1854:   {(char*)"_raw_saved_weight", (getter)THPNativeGroupNormBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
1855:   {(char*)"_saved_result1", (getter)THPNativeGroupNormBackward0_result1_getter, nullptr, nullptr, nullptr},
1856:   {(char*)"_raw_saved_result1", (getter)THPNativeGroupNormBackward0_result1_raw_getter, nullptr, nullptr, nullptr},
1857:   {(char*)"_saved_result2", (getter)THPNativeGroupNormBackward0_result2_getter, nullptr, nullptr, nullptr},
1858:   {(char*)"_raw_saved_result2", (getter)THPNativeGroupNormBackward0_result2_raw_getter, nullptr, nullptr, nullptr},
1859:   {nullptr} /* sentinel */
1860: };
1861: 
1862: static PyObject* THPLinalgVectorNormBackward0_dim_getter(THPCppFunction *self, void *_unused) {
1863:   HANDLE_TH_ERRORS
1864:   auto opt_prop = static_cast<LinalgVectorNormBackward0*>(self->cdata.get())->dim;
1865:   if (!opt_prop.list.has_value()) {
1866:     Py_RETURN_NONE;
1867:   }
1868:   auto prop = opt_prop.list.value();
1869:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
1870:   for (auto i : c10::irange(prop.size())) {
1871:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
1872:   }
1873:   return tup;
1874:   END_HANDLE_TH_ERRORS
1875: }
1876: 
1877: static PyObject* THPLinalgVectorNormBackward0_keepdim_getter(THPCppFunction *self, void *_unused) {
1878:   HANDLE_TH_ERRORS
1879:   auto prop = static_cast<LinalgVectorNormBackward0*>(self->cdata.get())->keepdim;
1880:   if (prop) {
1881:     Py_RETURN_TRUE;
1882:   } else {
1883:     Py_RETURN_FALSE;
1884:   }
1885:   END_HANDLE_TH_ERRORS
1886: }
1887: 
1888: static PyObject* THPLinalgVectorNormBackward0_ord_getter(THPCppFunction *self, void *_unused) {
1889:   HANDLE_TH_ERRORS
1890:   auto prop = static_cast<LinalgVectorNormBackward0*>(self->cdata.get())->ord;
1891:   if (prop.isComplex()) {
1892:     auto cprop = prop.to<c10::complex<double>>();
1893:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
1894:   } else if (prop.isFloatingPoint()) {
1895:     return PyFloat_FromDouble(prop.to<double>());
1896:   } else if (prop.isIntegral(/*includeBool=*/false)) {
1897:     return PyLong_FromLong(prop.to<int64_t>());
1898:   } else if (prop.isBoolean()) {
1899:     if (prop.to<bool>()) {
1900:       Py_RETURN_TRUE;
1901:     } else {
1902:       Py_RETURN_FALSE;
1903:     }
1904:   } else {
1905:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
1906:     return nullptr;
1907:   }
1908:   END_HANDLE_TH_ERRORS
1909: }
1910: 
1911: static PyObject* THPLinalgVectorNormBackward0_self_getter(THPCppFunction *self, void *_unused) {
1912:   HANDLE_TH_ERRORS
1913:   const auto& prop = static_cast<LinalgVectorNormBackward0*>(self->cdata.get())->self_;
1914:   return THPVariable_Wrap(prop.unpack(self->cdata));
1915:   END_HANDLE_TH_ERRORS
1916: }
1917: 
1918: static PyObject* THPLinalgVectorNormBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
1919:   HANDLE_TH_ERRORS
1920:   const auto& prop = static_cast<LinalgVectorNormBackward0*>(self->cdata.get())->self_;
```

- EN: The main execution path in this span is carried by `THPLinalgVectorNormBackward0_dim_getter`, `PyTuple_New`, `PyTuple_SetItem`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPLinalgVectorNormBackward0_dim_getter`, `PyTuple_New`, `PyTuple_SetItem` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1921-2000

```cpp
1921:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1922:   return obj.release().ptr();
1923:   END_HANDLE_TH_ERRORS
1924: }
1925: 
1926: static PyObject* THPLinalgVectorNormBackward0_result_getter(THPCppFunction *self, void *_unused) {
1927:   HANDLE_TH_ERRORS
1928:   const auto& prop = static_cast<LinalgVectorNormBackward0*>(self->cdata.get())->result_;
1929:   return THPVariable_Wrap(prop.unpack(self->cdata));
1930:   END_HANDLE_TH_ERRORS
1931: }
1932: 
1933: static PyObject* THPLinalgVectorNormBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
1934:   HANDLE_TH_ERRORS
1935:   const auto& prop = static_cast<LinalgVectorNormBackward0*>(self->cdata.get())->result_;
1936:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1937:   return obj.release().ptr();
1938:   END_HANDLE_TH_ERRORS
1939: }
1940: 
1941: static struct PyGetSetDef LinalgVectorNormBackward0_properties[] = {
1942:   THP_FUNCTION_DEFAULT_PROPERTIES,
1943:   {(char*)"_saved_dim", (getter)THPLinalgVectorNormBackward0_dim_getter, nullptr, nullptr, nullptr},
1944:   {(char*)"_saved_keepdim", (getter)THPLinalgVectorNormBackward0_keepdim_getter, nullptr, nullptr, nullptr},
1945:   {(char*)"_saved_ord", (getter)THPLinalgVectorNormBackward0_ord_getter, nullptr, nullptr, nullptr},
1946:   {(char*)"_saved_self", (getter)THPLinalgVectorNormBackward0_self_getter, nullptr, nullptr, nullptr},
1947:   {(char*)"_raw_saved_self", (getter)THPLinalgVectorNormBackward0_self_raw_getter, nullptr, nullptr, nullptr},
1948:   {(char*)"_saved_result", (getter)THPLinalgVectorNormBackward0_result_getter, nullptr, nullptr, nullptr},
1949:   {(char*)"_raw_saved_result", (getter)THPLinalgVectorNormBackward0_result_raw_getter, nullptr, nullptr, nullptr},
1950:   {nullptr} /* sentinel */
1951: };
1952: 
1953: 
1954: 
1955: static struct PyGetSetDef PdistBackwardBackward0_properties[] = {
1956:   THP_FUNCTION_DEFAULT_PROPERTIES,
1957: 
1958:   {nullptr} /* sentinel */
1959: };
1960: 
1961: static PyObject* THPOrmqrBackward0_input2_getter(THPCppFunction *self, void *_unused) {
1962:   HANDLE_TH_ERRORS
1963:   const auto& prop = static_cast<OrmqrBackward0*>(self->cdata.get())->input2_;
1964:   return THPVariable_Wrap(prop.unpack(self->cdata));
1965:   END_HANDLE_TH_ERRORS
1966: }
1967: 
1968: static PyObject* THPOrmqrBackward0_input2_raw_getter(THPCppFunction *self, void *_unused) {
1969:   HANDLE_TH_ERRORS
1970:   const auto& prop = static_cast<OrmqrBackward0*>(self->cdata.get())->input2_;
1971:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1972:   return obj.release().ptr();
1973:   END_HANDLE_TH_ERRORS
1974: }
1975: 
1976: static PyObject* THPOrmqrBackward0_input3_getter(THPCppFunction *self, void *_unused) {
1977:   HANDLE_TH_ERRORS
1978:   const auto& prop = static_cast<OrmqrBackward0*>(self->cdata.get())->input3_;
1979:   return THPVariable_Wrap(prop.unpack(self->cdata));
1980:   END_HANDLE_TH_ERRORS
1981: }
1982: 
1983: static PyObject* THPOrmqrBackward0_input3_raw_getter(THPCppFunction *self, void *_unused) {
1984:   HANDLE_TH_ERRORS
1985:   const auto& prop = static_cast<OrmqrBackward0*>(self->cdata.get())->input3_;
1986:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1987:   return obj.release().ptr();
1988:   END_HANDLE_TH_ERRORS
1989: }
1990: 
1991: static PyObject* THPOrmqrBackward0_left_getter(THPCppFunction *self, void *_unused) {
1992:   HANDLE_TH_ERRORS
1993:   auto prop = static_cast<OrmqrBackward0*>(self->cdata.get())->left;
1994:   if (prop) {
1995:     Py_RETURN_TRUE;
1996:   } else {
1997:     Py_RETURN_FALSE;
1998:   }
1999:   END_HANDLE_TH_ERRORS
2000: }
```

- EN: The main execution path in this span is carried by `cast`, `THPLinalgVectorNormBackward0_result_getter`, `THPVariable_Wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `cast`, `THPLinalgVectorNormBackward0_result_getter`, `THPVariable_Wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2001-2080

```cpp
2001: 
2002: static PyObject* THPOrmqrBackward0_self_getter(THPCppFunction *self, void *_unused) {
2003:   HANDLE_TH_ERRORS
2004:   const auto& prop = static_cast<OrmqrBackward0*>(self->cdata.get())->self_;
2005:   return THPVariable_Wrap(prop.unpack(self->cdata));
2006:   END_HANDLE_TH_ERRORS
2007: }
2008: 
2009: static PyObject* THPOrmqrBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
2010:   HANDLE_TH_ERRORS
2011:   const auto& prop = static_cast<OrmqrBackward0*>(self->cdata.get())->self_;
2012:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2013:   return obj.release().ptr();
2014:   END_HANDLE_TH_ERRORS
2015: }
2016: 
2017: static PyObject* THPOrmqrBackward0_transpose_getter(THPCppFunction *self, void *_unused) {
2018:   HANDLE_TH_ERRORS
2019:   auto prop = static_cast<OrmqrBackward0*>(self->cdata.get())->transpose;
2020:   if (prop) {
2021:     Py_RETURN_TRUE;
2022:   } else {
2023:     Py_RETURN_FALSE;
2024:   }
2025:   END_HANDLE_TH_ERRORS
2026: }
2027: 
2028: static PyObject* THPOrmqrBackward0_result_getter(THPCppFunction *self, void *_unused) {
2029:   HANDLE_TH_ERRORS
2030:   const auto& prop = static_cast<OrmqrBackward0*>(self->cdata.get())->result_;
2031:   return THPVariable_Wrap(prop.unpack(self->cdata));
2032:   END_HANDLE_TH_ERRORS
2033: }
2034: 
2035: static PyObject* THPOrmqrBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
2036:   HANDLE_TH_ERRORS
2037:   const auto& prop = static_cast<OrmqrBackward0*>(self->cdata.get())->result_;
2038:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2039:   return obj.release().ptr();
2040:   END_HANDLE_TH_ERRORS
2041: }
2042: 
2043: static struct PyGetSetDef OrmqrBackward0_properties[] = {
2044:   THP_FUNCTION_DEFAULT_PROPERTIES,
2045:   {(char*)"_saved_input2", (getter)THPOrmqrBackward0_input2_getter, nullptr, nullptr, nullptr},
2046:   {(char*)"_raw_saved_input2", (getter)THPOrmqrBackward0_input2_raw_getter, nullptr, nullptr, nullptr},
2047:   {(char*)"_saved_input3", (getter)THPOrmqrBackward0_input3_getter, nullptr, nullptr, nullptr},
2048:   {(char*)"_raw_saved_input3", (getter)THPOrmqrBackward0_input3_raw_getter, nullptr, nullptr, nullptr},
2049:   {(char*)"_saved_left", (getter)THPOrmqrBackward0_left_getter, nullptr, nullptr, nullptr},
2050:   {(char*)"_saved_self", (getter)THPOrmqrBackward0_self_getter, nullptr, nullptr, nullptr},
2051:   {(char*)"_raw_saved_self", (getter)THPOrmqrBackward0_self_raw_getter, nullptr, nullptr, nullptr},
2052:   {(char*)"_saved_transpose", (getter)THPOrmqrBackward0_transpose_getter, nullptr, nullptr, nullptr},
2053:   {(char*)"_saved_result", (getter)THPOrmqrBackward0_result_getter, nullptr, nullptr, nullptr},
2054:   {(char*)"_raw_saved_result", (getter)THPOrmqrBackward0_result_raw_getter, nullptr, nullptr, nullptr},
2055:   {nullptr} /* sentinel */
2056: };
2057: 
2058: static PyObject* THPPermuteBackward0_dims_getter(THPCppFunction *self, void *_unused) {
2059:   HANDLE_TH_ERRORS
2060:   auto prop = static_cast<PermuteBackward0*>(self->cdata.get())->dims;
2061:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
2062:   for (auto i : c10::irange(prop.size())) {
2063:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
2064:   }
2065:   return tup;
2066:   END_HANDLE_TH_ERRORS
2067: }
2068: 
2069: static struct PyGetSetDef PermuteBackward0_properties[] = {
2070:   THP_FUNCTION_DEFAULT_PROPERTIES,
2071:   {(char*)"_saved_dims", (getter)THPPermuteBackward0_dims_getter, nullptr, nullptr, nullptr},
2072:   {nullptr} /* sentinel */
2073: };
2074: 
2075: static PyObject* THPReshapeAliasBackward0_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
2076:   HANDLE_TH_ERRORS
2077:   auto prop = static_cast<ReshapeAliasBackward0*>(self->cdata.get())->self_sym_sizes;
2078:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
2079:   for (auto i : c10::irange(prop.size())) {
2080:       auto si = prop[i];
```

- EN: The main execution path in this span is carried by `THPOrmqrBackward0_self_getter`, `THPVariable_Wrap`, `THPOrmqrBackward0_self_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPOrmqrBackward0_self_getter`, `THPVariable_Wrap`, `THPOrmqrBackward0_self_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2081-2160

```cpp
2081:       if (auto m = si.maybe_as_int()) {
2082:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
2083:       } else {
2084:         auto py_symint = py::cast(si).release().ptr();
2085:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
2086:       }
2087:   }
2088:   return tup;
2089:   END_HANDLE_TH_ERRORS
2090: }
2091: 
2092: static struct PyGetSetDef ReshapeAliasBackward0_properties[] = {
2093:   THP_FUNCTION_DEFAULT_PROPERTIES,
2094:   {(char*)"_saved_self_sym_sizes", (getter)THPReshapeAliasBackward0_self_sym_sizes_getter, nullptr, nullptr, nullptr},
2095:   {nullptr} /* sentinel */
2096: };
2097: 
2098: static PyObject* THPSelectBackward0_dim_getter(THPCppFunction *self, void *_unused) {
2099:   HANDLE_TH_ERRORS
2100:   auto prop = static_cast<SelectBackward0*>(self->cdata.get())->dim;
2101:   return PyLong_FromUnsignedLong((int64_t) prop);
2102:   END_HANDLE_TH_ERRORS
2103: }
2104: 
2105: static PyObject* THPSelectBackward0_index_getter(THPCppFunction *self, void *_unused) {
2106:   HANDLE_TH_ERRORS
2107:   auto prop = static_cast<SelectBackward0*>(self->cdata.get())->index;
2108:   if (auto m = prop.maybe_as_int()) {
2109:     return PyLong_FromUnsignedLong(*m);
2110:   } else {
2111:     return py::cast(prop).release().ptr();
2112:   }
2113:   END_HANDLE_TH_ERRORS
2114: }
2115: 
2116: static PyObject* THPSelectBackward0_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
2117:   HANDLE_TH_ERRORS
2118:   auto prop = static_cast<SelectBackward0*>(self->cdata.get())->self_sym_sizes;
2119:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
2120:   for (auto i : c10::irange(prop.size())) {
2121:       auto si = prop[i];
2122:       if (auto m = si.maybe_as_int()) {
2123:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
2124:       } else {
2125:         auto py_symint = py::cast(si).release().ptr();
2126:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
2127:       }
2128:   }
2129:   return tup;
2130:   END_HANDLE_TH_ERRORS
2131: }
2132: 
2133: static struct PyGetSetDef SelectBackward0_properties[] = {
2134:   THP_FUNCTION_DEFAULT_PROPERTIES,
2135:   {(char*)"_saved_dim", (getter)THPSelectBackward0_dim_getter, nullptr, nullptr, nullptr},
2136:   {(char*)"_saved_index", (getter)THPSelectBackward0_index_getter, nullptr, nullptr, nullptr},
2137:   {(char*)"_saved_self_sym_sizes", (getter)THPSelectBackward0_self_sym_sizes_getter, nullptr, nullptr, nullptr},
2138:   {nullptr} /* sentinel */
2139: };
2140: 
2141: static PyObject* THPSelectBackwardAutogradNestedTensor0_dim_getter(THPCppFunction *self, void *_unused) {
2142:   HANDLE_TH_ERRORS
2143:   auto prop = static_cast<SelectBackwardAutogradNestedTensor0*>(self->cdata.get())->dim;
2144:   return PyLong_FromUnsignedLong((int64_t) prop);
2145:   END_HANDLE_TH_ERRORS
2146: }
2147: 
2148: static PyObject* THPSelectBackwardAutogradNestedTensor0_index_getter(THPCppFunction *self, void *_unused) {
2149:   HANDLE_TH_ERRORS
2150:   auto prop = static_cast<SelectBackwardAutogradNestedTensor0*>(self->cdata.get())->index;
2151:   if (auto m = prop.maybe_as_int()) {
2152:     return PyLong_FromUnsignedLong(*m);
2153:   } else {
2154:     return py::cast(prop).release().ptr();
2155:   }
2156:   END_HANDLE_TH_ERRORS
2157: }
2158: 
2159: static PyObject* THPSelectBackwardAutogradNestedTensor0_self_getter(THPCppFunction *self, void *_unused) {
2160:   HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `PyTuple_SetItem`, `cast`, `THPSelectBackward0_dim_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `PyTuple_SetItem`, `cast`, `THPSelectBackward0_dim_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 2161-2240

```cpp
2161:   const auto& prop = static_cast<SelectBackwardAutogradNestedTensor0*>(self->cdata.get())->self_;
2162:   return THPVariable_Wrap(prop.unpack(self->cdata));
2163:   END_HANDLE_TH_ERRORS
2164: }
2165: 
2166: static PyObject* THPSelectBackwardAutogradNestedTensor0_self_raw_getter(THPCppFunction *self, void *_unused) {
2167:   HANDLE_TH_ERRORS
2168:   const auto& prop = static_cast<SelectBackwardAutogradNestedTensor0*>(self->cdata.get())->self_;
2169:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2170:   return obj.release().ptr();
2171:   END_HANDLE_TH_ERRORS
2172: }
2173: 
2174: static struct PyGetSetDef SelectBackwardAutogradNestedTensor0_properties[] = {
2175:   THP_FUNCTION_DEFAULT_PROPERTIES,
2176:   {(char*)"_saved_dim", (getter)THPSelectBackwardAutogradNestedTensor0_dim_getter, nullptr, nullptr, nullptr},
2177:   {(char*)"_saved_index", (getter)THPSelectBackwardAutogradNestedTensor0_index_getter, nullptr, nullptr, nullptr},
2178:   {(char*)"_saved_self", (getter)THPSelectBackwardAutogradNestedTensor0_self_getter, nullptr, nullptr, nullptr},
2179:   {(char*)"_raw_saved_self", (getter)THPSelectBackwardAutogradNestedTensor0_self_raw_getter, nullptr, nullptr, nullptr},
2180:   {nullptr} /* sentinel */
2181: };
2182: 
2183: static PyObject* THPLogitBackward0_eps_getter(THPCppFunction *self, void *_unused) {
2184:   HANDLE_TH_ERRORS
2185:   auto opt_prop = static_cast<LogitBackward0*>(self->cdata.get())->eps;
2186:   if (!opt_prop.has_value()) {
2187:     Py_RETURN_NONE;
2188:   }
2189:   auto prop = opt_prop.value();
2190:   return PyFloat_FromDouble((double) prop);
2191:   END_HANDLE_TH_ERRORS
2192: }
2193: 
2194: static PyObject* THPLogitBackward0_self_getter(THPCppFunction *self, void *_unused) {
2195:   HANDLE_TH_ERRORS
2196:   const auto& prop = static_cast<LogitBackward0*>(self->cdata.get())->self_;
2197:   return THPVariable_Wrap(prop.unpack(self->cdata));
2198:   END_HANDLE_TH_ERRORS
2199: }
2200: 
2201: static PyObject* THPLogitBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
2202:   HANDLE_TH_ERRORS
2203:   const auto& prop = static_cast<LogitBackward0*>(self->cdata.get())->self_;
2204:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2205:   return obj.release().ptr();
2206:   END_HANDLE_TH_ERRORS
2207: }
2208: 
2209: static struct PyGetSetDef LogitBackward0_properties[] = {
2210:   THP_FUNCTION_DEFAULT_PROPERTIES,
2211:   {(char*)"_saved_eps", (getter)THPLogitBackward0_eps_getter, nullptr, nullptr, nullptr},
2212:   {(char*)"_saved_self", (getter)THPLogitBackward0_self_getter, nullptr, nullptr, nullptr},
2213:   {(char*)"_raw_saved_self", (getter)THPLogitBackward0_self_raw_getter, nullptr, nullptr, nullptr},
2214:   {nullptr} /* sentinel */
2215: };
2216: 
2217: 
2218: 
2219: static struct PyGetSetDef SignBackward0_properties[] = {
2220:   THP_FUNCTION_DEFAULT_PROPERTIES,
2221: 
2222:   {nullptr} /* sentinel */
2223: };
2224: 
2225: static PyObject* THPSinhBackward0_self_getter(THPCppFunction *self, void *_unused) {
2226:   HANDLE_TH_ERRORS
2227:   const auto& prop = static_cast<SinhBackward0*>(self->cdata.get())->self_;
2228:   return THPVariable_Wrap(prop.unpack(self->cdata));
2229:   END_HANDLE_TH_ERRORS
2230: }
2231: 
2232: static PyObject* THPSinhBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
2233:   HANDLE_TH_ERRORS
2234:   const auto& prop = static_cast<SinhBackward0*>(self->cdata.get())->self_;
2235:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2236:   return obj.release().ptr();
2237:   END_HANDLE_TH_ERRORS
2238: }
2239: 
2240: static struct PyGetSetDef SinhBackward0_properties[] = {
```

- EN: The main execution path in this span is carried by `THPVariable_Wrap`, `THPSelectBackwardAutogradNestedTensor0_self_raw_getter`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_Wrap`, `THPSelectBackwardAutogradNestedTensor0_self_raw_getter`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 2241-2320

```cpp
2241:   THP_FUNCTION_DEFAULT_PROPERTIES,
2242:   {(char*)"_saved_self", (getter)THPSinhBackward0_self_getter, nullptr, nullptr, nullptr},
2243:   {(char*)"_raw_saved_self", (getter)THPSinhBackward0_self_raw_getter, nullptr, nullptr, nullptr},
2244:   {nullptr} /* sentinel */
2245: };
2246: 
2247: static PyObject* THPSliceBackwardBackward0_dim_getter(THPCppFunction *self, void *_unused) {
2248:   HANDLE_TH_ERRORS
2249:   auto prop = static_cast<SliceBackwardBackward0*>(self->cdata.get())->dim;
2250:   return PyLong_FromUnsignedLong((int64_t) prop);
2251:   END_HANDLE_TH_ERRORS
2252: }
2253: 
2254: static PyObject* THPSliceBackwardBackward0_end_getter(THPCppFunction *self, void *_unused) {
2255:   HANDLE_TH_ERRORS
2256:   auto prop = static_cast<SliceBackwardBackward0*>(self->cdata.get())->end;
2257:   if (auto m = prop.maybe_as_int()) {
2258:     return PyLong_FromUnsignedLong(*m);
2259:   } else {
2260:     return py::cast(prop).release().ptr();
2261:   }
2262:   END_HANDLE_TH_ERRORS
2263: }
2264: 
2265: static PyObject* THPSliceBackwardBackward0_start_getter(THPCppFunction *self, void *_unused) {
2266:   HANDLE_TH_ERRORS
2267:   auto prop = static_cast<SliceBackwardBackward0*>(self->cdata.get())->start;
2268:   if (auto m = prop.maybe_as_int()) {
2269:     return PyLong_FromUnsignedLong(*m);
2270:   } else {
2271:     return py::cast(prop).release().ptr();
2272:   }
2273:   END_HANDLE_TH_ERRORS
2274: }
2275: 
2276: static PyObject* THPSliceBackwardBackward0_step_getter(THPCppFunction *self, void *_unused) {
2277:   HANDLE_TH_ERRORS
2278:   auto prop = static_cast<SliceBackwardBackward0*>(self->cdata.get())->step;
2279:   if (auto m = prop.maybe_as_int()) {
2280:     return PyLong_FromUnsignedLong(*m);
2281:   } else {
2282:     return py::cast(prop).release().ptr();
2283:   }
2284:   END_HANDLE_TH_ERRORS
2285: }
2286: 
2287: static struct PyGetSetDef SliceBackwardBackward0_properties[] = {
2288:   THP_FUNCTION_DEFAULT_PROPERTIES,
2289:   {(char*)"_saved_dim", (getter)THPSliceBackwardBackward0_dim_getter, nullptr, nullptr, nullptr},
2290:   {(char*)"_saved_end", (getter)THPSliceBackwardBackward0_end_getter, nullptr, nullptr, nullptr},
2291:   {(char*)"_saved_start", (getter)THPSliceBackwardBackward0_start_getter, nullptr, nullptr, nullptr},
2292:   {(char*)"_saved_step", (getter)THPSliceBackwardBackward0_step_getter, nullptr, nullptr, nullptr},
2293:   {nullptr} /* sentinel */
2294: };
2295: 
2296: static PyObject* THPUnsafeSplitBackward0_dim_getter(THPCppFunction *self, void *_unused) {
2297:   HANDLE_TH_ERRORS
2298:   auto prop = static_cast<UnsafeSplitBackward0*>(self->cdata.get())->dim;
2299:   return PyLong_FromUnsignedLong((int64_t) prop);
2300:   END_HANDLE_TH_ERRORS
2301: }
2302: 
2303: static PyObject* THPUnsafeSplitBackward0_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
2304:   HANDLE_TH_ERRORS
2305:   auto prop = static_cast<UnsafeSplitBackward0*>(self->cdata.get())->self_sym_sizes;
2306:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
2307:   for (auto i : c10::irange(prop.size())) {
2308:       auto si = prop[i];
2309:       if (auto m = si.maybe_as_int()) {
2310:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
2311:       } else {
2312:         auto py_symint = py::cast(si).release().ptr();
2313:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
2314:       }
2315:   }
2316:   return tup;
2317:   END_HANDLE_TH_ERRORS
2318: }
2319: 
2320: static PyObject* THPUnsafeSplitBackward0_split_size_getter(THPCppFunction *self, void *_unused) {
```

- EN: The main execution path in this span is carried by `THPSliceBackwardBackward0_dim_getter`, `PyLong_FromUnsignedLong`, `THPSliceBackwardBackward0_end_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPSliceBackwardBackward0_dim_getter`, `PyLong_FromUnsignedLong`, `THPSliceBackwardBackward0_end_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2321-2400

```cpp
2321:   HANDLE_TH_ERRORS
2322:   auto prop = static_cast<UnsafeSplitBackward0*>(self->cdata.get())->split_size;
2323:   if (auto m = prop.maybe_as_int()) {
2324:     return PyLong_FromUnsignedLong(*m);
2325:   } else {
2326:     return py::cast(prop).release().ptr();
2327:   }
2328:   END_HANDLE_TH_ERRORS
2329: }
2330: 
2331: static struct PyGetSetDef UnsafeSplitBackward0_properties[] = {
2332:   THP_FUNCTION_DEFAULT_PROPERTIES,
2333:   {(char*)"_saved_dim", (getter)THPUnsafeSplitBackward0_dim_getter, nullptr, nullptr, nullptr},
2334:   {(char*)"_saved_self_sym_sizes", (getter)THPUnsafeSplitBackward0_self_sym_sizes_getter, nullptr, nullptr, nullptr},
2335:   {(char*)"_saved_split_size", (getter)THPUnsafeSplitBackward0_split_size_getter, nullptr, nullptr, nullptr},
2336:   {nullptr} /* sentinel */
2337: };
2338: 
2339: static PyObject* THPSqrtBackward0_result_getter(THPCppFunction *self, void *_unused) {
2340:   HANDLE_TH_ERRORS
2341:   const auto& prop = static_cast<SqrtBackward0*>(self->cdata.get())->result_;
2342:   return THPVariable_Wrap(prop.unpack(self->cdata));
2343:   END_HANDLE_TH_ERRORS
2344: }
2345: 
2346: static PyObject* THPSqrtBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
2347:   HANDLE_TH_ERRORS
2348:   const auto& prop = static_cast<SqrtBackward0*>(self->cdata.get())->result_;
2349:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2350:   return obj.release().ptr();
2351:   END_HANDLE_TH_ERRORS
2352: }
2353: 
2354: static struct PyGetSetDef SqrtBackward0_properties[] = {
2355:   THP_FUNCTION_DEFAULT_PROPERTIES,
2356:   {(char*)"_saved_result", (getter)THPSqrtBackward0_result_getter, nullptr, nullptr, nullptr},
2357:   {(char*)"_raw_saved_result", (getter)THPSqrtBackward0_result_raw_getter, nullptr, nullptr, nullptr},
2358:   {nullptr} /* sentinel */
2359: };
2360: 
2361: static PyObject* THPStdBackward0_correction_getter(THPCppFunction *self, void *_unused) {
2362:   HANDLE_TH_ERRORS
2363:   auto opt_prop = static_cast<StdBackward0*>(self->cdata.get())->correction;
2364:   if (!opt_prop.has_value()) {
2365:     Py_RETURN_NONE;
2366:   }
2367:   auto prop = opt_prop.value();
2368:   if (prop.isComplex()) {
2369:     auto cprop = prop.to<c10::complex<double>>();
2370:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
2371:   } else if (prop.isFloatingPoint()) {
2372:     return PyFloat_FromDouble(prop.to<double>());
2373:   } else if (prop.isIntegral(/*includeBool=*/false)) {
2374:     return PyLong_FromLong(prop.to<int64_t>());
2375:   } else if (prop.isBoolean()) {
2376:     if (prop.to<bool>()) {
2377:       Py_RETURN_TRUE;
2378:     } else {
2379:       Py_RETURN_FALSE;
2380:     }
2381:   } else {
2382:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
2383:     return nullptr;
2384:   }
2385:   END_HANDLE_TH_ERRORS
2386: }
2387: 
2388: static PyObject* THPStdBackward0_dim_getter(THPCppFunction *self, void *_unused) {
2389:   HANDLE_TH_ERRORS
2390:   auto opt_prop = static_cast<StdBackward0*>(self->cdata.get())->dim;
2391:   if (!opt_prop.list.has_value()) {
2392:     Py_RETURN_NONE;
2393:   }
2394:   auto prop = opt_prop.list.value();
2395:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
2396:   for (auto i : c10::irange(prop.size())) {
2397:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
2398:   }
2399:   return tup;
2400:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `PyLong_FromUnsignedLong`, `cast`, `THPSqrtBackward0_result_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyLong_FromUnsignedLong`, `cast`, `THPSqrtBackward0_result_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2401-2480

```cpp
2401: }
2402: 
2403: static PyObject* THPStdBackward0_keepdim_getter(THPCppFunction *self, void *_unused) {
2404:   HANDLE_TH_ERRORS
2405:   auto prop = static_cast<StdBackward0*>(self->cdata.get())->keepdim;
2406:   if (prop) {
2407:     Py_RETURN_TRUE;
2408:   } else {
2409:     Py_RETURN_FALSE;
2410:   }
2411:   END_HANDLE_TH_ERRORS
2412: }
2413: 
2414: static PyObject* THPStdBackward0_self_getter(THPCppFunction *self, void *_unused) {
2415:   HANDLE_TH_ERRORS
2416:   const auto& prop = static_cast<StdBackward0*>(self->cdata.get())->self_;
2417:   return THPVariable_Wrap(prop.unpack(self->cdata));
2418:   END_HANDLE_TH_ERRORS
2419: }
2420: 
2421: static PyObject* THPStdBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
2422:   HANDLE_TH_ERRORS
2423:   const auto& prop = static_cast<StdBackward0*>(self->cdata.get())->self_;
2424:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2425:   return obj.release().ptr();
2426:   END_HANDLE_TH_ERRORS
2427: }
2428: 
2429: static PyObject* THPStdBackward0_result_getter(THPCppFunction *self, void *_unused) {
2430:   HANDLE_TH_ERRORS
2431:   const auto& prop = static_cast<StdBackward0*>(self->cdata.get())->result_;
2432:   return THPVariable_Wrap(prop.unpack(self->cdata));
2433:   END_HANDLE_TH_ERRORS
2434: }
2435: 
2436: static PyObject* THPStdBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
2437:   HANDLE_TH_ERRORS
2438:   const auto& prop = static_cast<StdBackward0*>(self->cdata.get())->result_;
2439:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2440:   return obj.release().ptr();
2441:   END_HANDLE_TH_ERRORS
2442: }
2443: 
2444: static struct PyGetSetDef StdBackward0_properties[] = {
2445:   THP_FUNCTION_DEFAULT_PROPERTIES,
2446:   {(char*)"_saved_correction", (getter)THPStdBackward0_correction_getter, nullptr, nullptr, nullptr},
2447:   {(char*)"_saved_dim", (getter)THPStdBackward0_dim_getter, nullptr, nullptr, nullptr},
2448:   {(char*)"_saved_keepdim", (getter)THPStdBackward0_keepdim_getter, nullptr, nullptr, nullptr},
2449:   {(char*)"_saved_self", (getter)THPStdBackward0_self_getter, nullptr, nullptr, nullptr},
2450:   {(char*)"_raw_saved_self", (getter)THPStdBackward0_self_raw_getter, nullptr, nullptr, nullptr},
2451:   {(char*)"_saved_result", (getter)THPStdBackward0_result_getter, nullptr, nullptr, nullptr},
2452:   {(char*)"_raw_saved_result", (getter)THPStdBackward0_result_raw_getter, nullptr, nullptr, nullptr},
2453:   {nullptr} /* sentinel */
2454: };
2455: 
2456: static PyObject* THPStdMeanBackward0_correction_getter(THPCppFunction *self, void *_unused) {
2457:   HANDLE_TH_ERRORS
2458:   auto opt_prop = static_cast<StdMeanBackward0*>(self->cdata.get())->correction;
2459:   if (!opt_prop.has_value()) {
2460:     Py_RETURN_NONE;
2461:   }
2462:   auto prop = opt_prop.value();
2463:   if (prop.isComplex()) {
2464:     auto cprop = prop.to<c10::complex<double>>();
2465:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
2466:   } else if (prop.isFloatingPoint()) {
2467:     return PyFloat_FromDouble(prop.to<double>());
2468:   } else if (prop.isIntegral(/*includeBool=*/false)) {
2469:     return PyLong_FromLong(prop.to<int64_t>());
2470:   } else if (prop.isBoolean()) {
2471:     if (prop.to<bool>()) {
2472:       Py_RETURN_TRUE;
2473:     } else {
2474:       Py_RETURN_FALSE;
2475:     }
2476:   } else {
2477:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
2478:     return nullptr;
2479:   }
2480:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPStdBackward0_keepdim_getter`, `THPStdBackward0_self_getter`, `THPVariable_Wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPStdBackward0_keepdim_getter`, `THPStdBackward0_self_getter`, `THPVariable_Wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2481-2560

```cpp
2481: }
2482: 
2483: static PyObject* THPStdMeanBackward0_dim_getter(THPCppFunction *self, void *_unused) {
2484:   HANDLE_TH_ERRORS
2485:   auto opt_prop = static_cast<StdMeanBackward0*>(self->cdata.get())->dim;
2486:   if (!opt_prop.list.has_value()) {
2487:     Py_RETURN_NONE;
2488:   }
2489:   auto prop = opt_prop.list.value();
2490:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
2491:   for (auto i : c10::irange(prop.size())) {
2492:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
2493:   }
2494:   return tup;
2495:   END_HANDLE_TH_ERRORS
2496: }
2497: 
2498: static PyObject* THPStdMeanBackward0_keepdim_getter(THPCppFunction *self, void *_unused) {
2499:   HANDLE_TH_ERRORS
2500:   auto prop = static_cast<StdMeanBackward0*>(self->cdata.get())->keepdim;
2501:   if (prop) {
2502:     Py_RETURN_TRUE;
2503:   } else {
2504:     Py_RETURN_FALSE;
2505:   }
2506:   END_HANDLE_TH_ERRORS
2507: }
2508: 
2509: static PyObject* THPStdMeanBackward0_self_getter(THPCppFunction *self, void *_unused) {
2510:   HANDLE_TH_ERRORS
2511:   const auto& prop = static_cast<StdMeanBackward0*>(self->cdata.get())->self_;
2512:   return THPVariable_Wrap(prop.unpack(self->cdata));
2513:   END_HANDLE_TH_ERRORS
2514: }
2515: 
2516: static PyObject* THPStdMeanBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
2517:   HANDLE_TH_ERRORS
2518:   const auto& prop = static_cast<StdMeanBackward0*>(self->cdata.get())->self_;
2519:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2520:   return obj.release().ptr();
2521:   END_HANDLE_TH_ERRORS
2522: }
2523: 
2524: static PyObject* THPStdMeanBackward0_result0_getter(THPCppFunction *self, void *_unused) {
2525:   HANDLE_TH_ERRORS
2526:   const auto& prop = static_cast<StdMeanBackward0*>(self->cdata.get())->result0_;
2527:   return THPVariable_Wrap(prop.unpack(self->cdata));
2528:   END_HANDLE_TH_ERRORS
2529: }
2530: 
2531: static PyObject* THPStdMeanBackward0_result0_raw_getter(THPCppFunction *self, void *_unused) {
2532:   HANDLE_TH_ERRORS
2533:   const auto& prop = static_cast<StdMeanBackward0*>(self->cdata.get())->result0_;
2534:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2535:   return obj.release().ptr();
2536:   END_HANDLE_TH_ERRORS
2537: }
2538: 
2539: static struct PyGetSetDef StdMeanBackward0_properties[] = {
2540:   THP_FUNCTION_DEFAULT_PROPERTIES,
2541:   {(char*)"_saved_correction", (getter)THPStdMeanBackward0_correction_getter, nullptr, nullptr, nullptr},
2542:   {(char*)"_saved_dim", (getter)THPStdMeanBackward0_dim_getter, nullptr, nullptr, nullptr},
2543:   {(char*)"_saved_keepdim", (getter)THPStdMeanBackward0_keepdim_getter, nullptr, nullptr, nullptr},
2544:   {(char*)"_saved_self", (getter)THPStdMeanBackward0_self_getter, nullptr, nullptr, nullptr},
2545:   {(char*)"_raw_saved_self", (getter)THPStdMeanBackward0_self_raw_getter, nullptr, nullptr, nullptr},
2546:   {(char*)"_saved_result0", (getter)THPStdMeanBackward0_result0_getter, nullptr, nullptr, nullptr},
2547:   {(char*)"_raw_saved_result0", (getter)THPStdMeanBackward0_result0_raw_getter, nullptr, nullptr, nullptr},
2548:   {nullptr} /* sentinel */
2549: };
2550: 
2551: static PyObject* THPRsubBackward0_alpha_getter(THPCppFunction *self, void *_unused) {
2552:   HANDLE_TH_ERRORS
2553:   auto prop = static_cast<RsubBackward0*>(self->cdata.get())->alpha;
2554:   if (prop.isComplex()) {
2555:     auto cprop = prop.to<c10::complex<double>>();
2556:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
2557:   } else if (prop.isFloatingPoint()) {
2558:     return PyFloat_FromDouble(prop.to<double>());
2559:   } else if (prop.isIntegral(/*includeBool=*/false)) {
2560:     return PyLong_FromLong(prop.to<int64_t>());
```

- EN: The main execution path in this span is carried by `THPStdMeanBackward0_dim_getter`, `PyTuple_New`, `PyTuple_SetItem`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPStdMeanBackward0_dim_getter`, `PyTuple_New`, `PyTuple_SetItem` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2561-2640

```cpp
2561:   } else if (prop.isBoolean()) {
2562:     if (prop.to<bool>()) {
2563:       Py_RETURN_TRUE;
2564:     } else {
2565:       Py_RETURN_FALSE;
2566:     }
2567:   } else {
2568:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
2569:     return nullptr;
2570:   }
2571:   END_HANDLE_TH_ERRORS
2572: }
2573: 
2574: static struct PyGetSetDef RsubBackward0_properties[] = {
2575:   THP_FUNCTION_DEFAULT_PROPERTIES,
2576:   {(char*)"_saved_alpha", (getter)THPRsubBackward0_alpha_getter, nullptr, nullptr, nullptr},
2577:   {nullptr} /* sentinel */
2578: };
2579: 
2580: static PyObject* THPRsubBackward1_alpha_getter(THPCppFunction *self, void *_unused) {
2581:   HANDLE_TH_ERRORS
2582:   auto prop = static_cast<RsubBackward1*>(self->cdata.get())->alpha;
2583:   if (prop.isComplex()) {
2584:     auto cprop = prop.to<c10::complex<double>>();
2585:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
2586:   } else if (prop.isFloatingPoint()) {
2587:     return PyFloat_FromDouble(prop.to<double>());
2588:   } else if (prop.isIntegral(/*includeBool=*/false)) {
2589:     return PyLong_FromLong(prop.to<int64_t>());
2590:   } else if (prop.isBoolean()) {
2591:     if (prop.to<bool>()) {
2592:       Py_RETURN_TRUE;
2593:     } else {
2594:       Py_RETURN_FALSE;
2595:     }
2596:   } else {
2597:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
2598:     return nullptr;
2599:   }
2600:   END_HANDLE_TH_ERRORS
2601: }
2602: 
2603: static struct PyGetSetDef RsubBackward1_properties[] = {
2604:   THP_FUNCTION_DEFAULT_PROPERTIES,
2605:   {(char*)"_saved_alpha", (getter)THPRsubBackward1_alpha_getter, nullptr, nullptr, nullptr},
2606:   {nullptr} /* sentinel */
2607: };
2608: 
2609: 
2610: 
2611: static struct PyGetSetDef TBackward0_properties[] = {
2612:   THP_FUNCTION_DEFAULT_PROPERTIES,
2613: 
2614:   {nullptr} /* sentinel */
2615: };
2616: 
2617: static PyObject* THPFlipBackward0_dims_getter(THPCppFunction *self, void *_unused) {
2618:   HANDLE_TH_ERRORS
2619:   auto prop = static_cast<FlipBackward0*>(self->cdata.get())->dims;
2620:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
2621:   for (auto i : c10::irange(prop.size())) {
2622:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
2623:   }
2624:   return tup;
2625:   END_HANDLE_TH_ERRORS
2626: }
2627: 
2628: static struct PyGetSetDef FlipBackward0_properties[] = {
2629:   THP_FUNCTION_DEFAULT_PROPERTIES,
2630:   {(char*)"_saved_dims", (getter)THPFlipBackward0_dims_getter, nullptr, nullptr, nullptr},
2631:   {nullptr} /* sentinel */
2632: };
2633: 
2634: static PyObject* THPRollBackward0_dims_getter(THPCppFunction *self, void *_unused) {
2635:   HANDLE_TH_ERRORS
2636:   auto prop = static_cast<RollBackward0*>(self->cdata.get())->dims;
2637:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
2638:   for (auto i : c10::irange(prop.size())) {
2639:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
2640:   }
```

- EN: The main execution path in this span is carried by `PyErr_SetString`, `THPRsubBackward1_alpha_getter`, `PyComplex_FromDoubles`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyErr_SetString`, `THPRsubBackward1_alpha_getter`, `PyComplex_FromDoubles` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2641-2720

```cpp
2641:   return tup;
2642:   END_HANDLE_TH_ERRORS
2643: }
2644: 
2645: static PyObject* THPRollBackward0_shifts_getter(THPCppFunction *self, void *_unused) {
2646:   HANDLE_TH_ERRORS
2647:   auto prop = static_cast<RollBackward0*>(self->cdata.get())->shifts;
2648:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
2649:   for (auto i : c10::irange(prop.size())) {
2650:       auto si = prop[i];
2651:       if (auto m = si.maybe_as_int()) {
2652:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
2653:       } else {
2654:         auto py_symint = py::cast(si).release().ptr();
2655:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
2656:       }
2657:   }
2658:   return tup;
2659:   END_HANDLE_TH_ERRORS
2660: }
2661: 
2662: static struct PyGetSetDef RollBackward0_properties[] = {
2663:   THP_FUNCTION_DEFAULT_PROPERTIES,
2664:   {(char*)"_saved_dims", (getter)THPRollBackward0_dims_getter, nullptr, nullptr, nullptr},
2665:   {(char*)"_saved_shifts", (getter)THPRollBackward0_shifts_getter, nullptr, nullptr, nullptr},
2666:   {nullptr} /* sentinel */
2667: };
2668: 
2669: static PyObject* THPTransposeBackward0_dim0_getter(THPCppFunction *self, void *_unused) {
2670:   HANDLE_TH_ERRORS
2671:   auto prop = static_cast<TransposeBackward0*>(self->cdata.get())->dim0;
2672:   return PyLong_FromUnsignedLong((int64_t) prop);
2673:   END_HANDLE_TH_ERRORS
2674: }
2675: 
2676: static PyObject* THPTransposeBackward0_dim1_getter(THPCppFunction *self, void *_unused) {
2677:   HANDLE_TH_ERRORS
2678:   auto prop = static_cast<TransposeBackward0*>(self->cdata.get())->dim1;
2679:   return PyLong_FromUnsignedLong((int64_t) prop);
2680:   END_HANDLE_TH_ERRORS
2681: }
2682: 
2683: static struct PyGetSetDef TransposeBackward0_properties[] = {
2684:   THP_FUNCTION_DEFAULT_PROPERTIES,
2685:   {(char*)"_saved_dim0", (getter)THPTransposeBackward0_dim0_getter, nullptr, nullptr, nullptr},
2686:   {(char*)"_saved_dim1", (getter)THPTransposeBackward0_dim1_getter, nullptr, nullptr, nullptr},
2687:   {nullptr} /* sentinel */
2688: };
2689: 
2690: static PyObject* THPTransposeBackward1_dim0_getter(THPCppFunction *self, void *_unused) {
2691:   HANDLE_TH_ERRORS
2692:   auto prop = static_cast<TransposeBackward1*>(self->cdata.get())->dim0;
2693:   return PyLong_FromUnsignedLong((int64_t) prop);
2694:   END_HANDLE_TH_ERRORS
2695: }
2696: 
2697: static PyObject* THPTransposeBackward1_dim1_getter(THPCppFunction *self, void *_unused) {
2698:   HANDLE_TH_ERRORS
2699:   auto prop = static_cast<TransposeBackward1*>(self->cdata.get())->dim1;
2700:   return PyLong_FromUnsignedLong((int64_t) prop);
2701:   END_HANDLE_TH_ERRORS
2702: }
2703: 
2704: static struct PyGetSetDef TransposeBackward1_properties[] = {
2705:   THP_FUNCTION_DEFAULT_PROPERTIES,
2706:   {(char*)"_saved_dim0", (getter)THPTransposeBackward1_dim0_getter, nullptr, nullptr, nullptr},
2707:   {(char*)"_saved_dim1", (getter)THPTransposeBackward1_dim1_getter, nullptr, nullptr, nullptr},
2708:   {nullptr} /* sentinel */
2709: };
2710: 
2711: static PyObject* THPTriangularSolveBackward0_A_getter(THPCppFunction *self, void *_unused) {
2712:   HANDLE_TH_ERRORS
2713:   const auto& prop = static_cast<TriangularSolveBackward0*>(self->cdata.get())->A_;
2714:   return THPVariable_Wrap(prop.unpack(self->cdata));
2715:   END_HANDLE_TH_ERRORS
2716: }
2717: 
2718: static PyObject* THPTriangularSolveBackward0_A_raw_getter(THPCppFunction *self, void *_unused) {
2719:   HANDLE_TH_ERRORS
2720:   const auto& prop = static_cast<TriangularSolveBackward0*>(self->cdata.get())->A_;
```

- EN: The main execution path in this span is carried by `THPRollBackward0_shifts_getter`, `PyTuple_New`, `PyTuple_SetItem`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPRollBackward0_shifts_getter`, `PyTuple_New`, `PyTuple_SetItem` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2721-2800

```cpp
2721:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2722:   return obj.release().ptr();
2723:   END_HANDLE_TH_ERRORS
2724: }
2725: 
2726: static PyObject* THPTriangularSolveBackward0_self_getter(THPCppFunction *self, void *_unused) {
2727:   HANDLE_TH_ERRORS
2728:   const auto& prop = static_cast<TriangularSolveBackward0*>(self->cdata.get())->self_;
2729:   return THPVariable_Wrap(prop.unpack(self->cdata));
2730:   END_HANDLE_TH_ERRORS
2731: }
2732: 
2733: static PyObject* THPTriangularSolveBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
2734:   HANDLE_TH_ERRORS
2735:   const auto& prop = static_cast<TriangularSolveBackward0*>(self->cdata.get())->self_;
2736:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2737:   return obj.release().ptr();
2738:   END_HANDLE_TH_ERRORS
2739: }
2740: 
2741: static PyObject* THPTriangularSolveBackward0_transpose_getter(THPCppFunction *self, void *_unused) {
2742:   HANDLE_TH_ERRORS
2743:   auto prop = static_cast<TriangularSolveBackward0*>(self->cdata.get())->transpose;
2744:   if (prop) {
2745:     Py_RETURN_TRUE;
2746:   } else {
2747:     Py_RETURN_FALSE;
2748:   }
2749:   END_HANDLE_TH_ERRORS
2750: }
2751: 
2752: static PyObject* THPTriangularSolveBackward0_unitriangular_getter(THPCppFunction *self, void *_unused) {
2753:   HANDLE_TH_ERRORS
2754:   auto prop = static_cast<TriangularSolveBackward0*>(self->cdata.get())->unitriangular;
2755:   if (prop) {
2756:     Py_RETURN_TRUE;
2757:   } else {
2758:     Py_RETURN_FALSE;
2759:   }
2760:   END_HANDLE_TH_ERRORS
2761: }
2762: 
2763: static PyObject* THPTriangularSolveBackward0_upper_getter(THPCppFunction *self, void *_unused) {
2764:   HANDLE_TH_ERRORS
2765:   auto prop = static_cast<TriangularSolveBackward0*>(self->cdata.get())->upper;
2766:   if (prop) {
2767:     Py_RETURN_TRUE;
2768:   } else {
2769:     Py_RETURN_FALSE;
2770:   }
2771:   END_HANDLE_TH_ERRORS
2772: }
2773: 
2774: static PyObject* THPTriangularSolveBackward0_solution_getter(THPCppFunction *self, void *_unused) {
2775:   HANDLE_TH_ERRORS
2776:   const auto& prop = static_cast<TriangularSolveBackward0*>(self->cdata.get())->solution_;
2777:   return THPVariable_Wrap(prop.unpack(self->cdata));
2778:   END_HANDLE_TH_ERRORS
2779: }
2780: 
2781: static PyObject* THPTriangularSolveBackward0_solution_raw_getter(THPCppFunction *self, void *_unused) {
2782:   HANDLE_TH_ERRORS
2783:   const auto& prop = static_cast<TriangularSolveBackward0*>(self->cdata.get())->solution_;
2784:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2785:   return obj.release().ptr();
2786:   END_HANDLE_TH_ERRORS
2787: }
2788: 
2789: static struct PyGetSetDef TriangularSolveBackward0_properties[] = {
2790:   THP_FUNCTION_DEFAULT_PROPERTIES,
2791:   {(char*)"_saved_A", (getter)THPTriangularSolveBackward0_A_getter, nullptr, nullptr, nullptr},
2792:   {(char*)"_raw_saved_A", (getter)THPTriangularSolveBackward0_A_raw_getter, nullptr, nullptr, nullptr},
2793:   {(char*)"_saved_self", (getter)THPTriangularSolveBackward0_self_getter, nullptr, nullptr, nullptr},
2794:   {(char*)"_raw_saved_self", (getter)THPTriangularSolveBackward0_self_raw_getter, nullptr, nullptr, nullptr},
2795:   {(char*)"_saved_transpose", (getter)THPTriangularSolveBackward0_transpose_getter, nullptr, nullptr, nullptr},
2796:   {(char*)"_saved_unitriangular", (getter)THPTriangularSolveBackward0_unitriangular_getter, nullptr, nullptr, nullptr},
2797:   {(char*)"_saved_upper", (getter)THPTriangularSolveBackward0_upper_getter, nullptr, nullptr, nullptr},
2798:   {(char*)"_saved_solution", (getter)THPTriangularSolveBackward0_solution_getter, nullptr, nullptr, nullptr},
2799:   {(char*)"_raw_saved_solution", (getter)THPTriangularSolveBackward0_solution_raw_getter, nullptr, nullptr, nullptr},
2800:   {nullptr} /* sentinel */
```

- EN: The main execution path in this span is carried by `cast`, `THPTriangularSolveBackward0_self_getter`, `THPVariable_Wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `cast`, `THPTriangularSolveBackward0_self_getter`, `THPVariable_Wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2801-2880

```cpp
2801: };
2802: 
2803: static PyObject* THPLinalgSolveTriangularBackward0_left_getter(THPCppFunction *self, void *_unused) {
2804:   HANDLE_TH_ERRORS
2805:   auto prop = static_cast<LinalgSolveTriangularBackward0*>(self->cdata.get())->left;
2806:   if (prop) {
2807:     Py_RETURN_TRUE;
2808:   } else {
2809:     Py_RETURN_FALSE;
2810:   }
2811:   END_HANDLE_TH_ERRORS
2812: }
2813: 
2814: static PyObject* THPLinalgSolveTriangularBackward0_self_getter(THPCppFunction *self, void *_unused) {
2815:   HANDLE_TH_ERRORS
2816:   const auto& prop = static_cast<LinalgSolveTriangularBackward0*>(self->cdata.get())->self_;
2817:   return THPVariable_Wrap(prop.unpack(self->cdata));
2818:   END_HANDLE_TH_ERRORS
2819: }
2820: 
2821: static PyObject* THPLinalgSolveTriangularBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
2822:   HANDLE_TH_ERRORS
2823:   const auto& prop = static_cast<LinalgSolveTriangularBackward0*>(self->cdata.get())->self_;
2824:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2825:   return obj.release().ptr();
2826:   END_HANDLE_TH_ERRORS
2827: }
2828: 
2829: static PyObject* THPLinalgSolveTriangularBackward0_unitriangular_getter(THPCppFunction *self, void *_unused) {
2830:   HANDLE_TH_ERRORS
2831:   auto prop = static_cast<LinalgSolveTriangularBackward0*>(self->cdata.get())->unitriangular;
2832:   if (prop) {
2833:     Py_RETURN_TRUE;
2834:   } else {
2835:     Py_RETURN_FALSE;
2836:   }
2837:   END_HANDLE_TH_ERRORS
2838: }
2839: 
2840: static PyObject* THPLinalgSolveTriangularBackward0_upper_getter(THPCppFunction *self, void *_unused) {
2841:   HANDLE_TH_ERRORS
2842:   auto prop = static_cast<LinalgSolveTriangularBackward0*>(self->cdata.get())->upper;
2843:   if (prop) {
2844:     Py_RETURN_TRUE;
2845:   } else {
2846:     Py_RETURN_FALSE;
2847:   }
2848:   END_HANDLE_TH_ERRORS
2849: }
2850: 
2851: static PyObject* THPLinalgSolveTriangularBackward0_result_getter(THPCppFunction *self, void *_unused) {
2852:   HANDLE_TH_ERRORS
2853:   const auto& prop = static_cast<LinalgSolveTriangularBackward0*>(self->cdata.get())->result_;
2854:   return THPVariable_Wrap(prop.unpack(self->cdata));
2855:   END_HANDLE_TH_ERRORS
2856: }
2857: 
2858: static PyObject* THPLinalgSolveTriangularBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
2859:   HANDLE_TH_ERRORS
2860:   const auto& prop = static_cast<LinalgSolveTriangularBackward0*>(self->cdata.get())->result_;
2861:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2862:   return obj.release().ptr();
2863:   END_HANDLE_TH_ERRORS
2864: }
2865: 
2866: static struct PyGetSetDef LinalgSolveTriangularBackward0_properties[] = {
2867:   THP_FUNCTION_DEFAULT_PROPERTIES,
2868:   {(char*)"_saved_left", (getter)THPLinalgSolveTriangularBackward0_left_getter, nullptr, nullptr, nullptr},
2869:   {(char*)"_saved_self", (getter)THPLinalgSolveTriangularBackward0_self_getter, nullptr, nullptr, nullptr},
2870:   {(char*)"_raw_saved_self", (getter)THPLinalgSolveTriangularBackward0_self_raw_getter, nullptr, nullptr, nullptr},
2871:   {(char*)"_saved_unitriangular", (getter)THPLinalgSolveTriangularBackward0_unitriangular_getter, nullptr, nullptr, nullptr},
2872:   {(char*)"_saved_upper", (getter)THPLinalgSolveTriangularBackward0_upper_getter, nullptr, nullptr, nullptr},
2873:   {(char*)"_saved_result", (getter)THPLinalgSolveTriangularBackward0_result_getter, nullptr, nullptr, nullptr},
2874:   {(char*)"_raw_saved_result", (getter)THPLinalgSolveTriangularBackward0_result_raw_getter, nullptr, nullptr, nullptr},
2875:   {nullptr} /* sentinel */
2876: };
2877: 
2878: static PyObject* THPTriuBackward0_diagonal_getter(THPCppFunction *self, void *_unused) {
2879:   HANDLE_TH_ERRORS
2880:   auto prop = static_cast<TriuBackward0*>(self->cdata.get())->diagonal;
```

- EN: The main execution path in this span is carried by `THPLinalgSolveTriangularBackward0_left_getter`, `THPLinalgSolveTriangularBackward0_self_getter`, `THPVariable_Wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPLinalgSolveTriangularBackward0_left_getter`, `THPLinalgSolveTriangularBackward0_self_getter`, `THPVariable_Wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2881-2960

```cpp
2881:   if (auto m = prop.maybe_as_int()) {
2882:     return PyLong_FromUnsignedLong(*m);
2883:   } else {
2884:     return py::cast(prop).release().ptr();
2885:   }
2886:   END_HANDLE_TH_ERRORS
2887: }
2888: 
2889: static struct PyGetSetDef TriuBackward0_properties[] = {
2890:   THP_FUNCTION_DEFAULT_PROPERTIES,
2891:   {(char*)"_saved_diagonal", (getter)THPTriuBackward0_diagonal_getter, nullptr, nullptr, nullptr},
2892:   {nullptr} /* sentinel */
2893: };
2894: 
2895: static PyObject* THPToSparseBackward0_self_self_sym_blocksize_opt_getter(THPCppFunction *self, void *_unused) {
2896:   HANDLE_TH_ERRORS
2897:   auto opt_prop = static_cast<ToSparseBackward0*>(self->cdata.get())->self_self_sym_blocksize_opt;
2898:   if (!opt_prop.list.has_value()) {
2899:     Py_RETURN_NONE;
2900:   }
2901:   auto prop = opt_prop.list.value();
2902:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
2903:   for (auto i : c10::irange(prop.size())) {
2904:       auto si = prop[i];
2905:       if (auto m = si.maybe_as_int()) {
2906:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
2907:       } else {
2908:         auto py_symint = py::cast(si).release().ptr();
2909:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
2910:       }
2911:   }
2912:   return tup;
2913:   END_HANDLE_TH_ERRORS
2914: }
2915: 
2916: static struct PyGetSetDef ToSparseBackward0_properties[] = {
2917:   THP_FUNCTION_DEFAULT_PROPERTIES,
2918:   {(char*)"_saved_self_self_sym_blocksize_opt", (getter)THPToSparseBackward0_self_self_sym_blocksize_opt_getter, nullptr, nullptr, nullptr},
2919:   {nullptr} /* sentinel */
2920: };
2921: 
2922: static PyObject* THPToSparseBackward1_self_self_sym_blocksize_opt_getter(THPCppFunction *self, void *_unused) {
2923:   HANDLE_TH_ERRORS
2924:   auto opt_prop = static_cast<ToSparseBackward1*>(self->cdata.get())->self_self_sym_blocksize_opt;
2925:   if (!opt_prop.list.has_value()) {
2926:     Py_RETURN_NONE;
2927:   }
2928:   auto prop = opt_prop.list.value();
2929:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
2930:   for (auto i : c10::irange(prop.size())) {
2931:       auto si = prop[i];
2932:       if (auto m = si.maybe_as_int()) {
2933:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
2934:       } else {
2935:         auto py_symint = py::cast(si).release().ptr();
2936:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
2937:       }
2938:   }
2939:   return tup;
2940:   END_HANDLE_TH_ERRORS
2941: }
2942: 
2943: static struct PyGetSetDef ToSparseBackward1_properties[] = {
2944:   THP_FUNCTION_DEFAULT_PROPERTIES,
2945:   {(char*)"_saved_self_self_sym_blocksize_opt", (getter)THPToSparseBackward1_self_self_sym_blocksize_opt_getter, nullptr, nullptr, nullptr},
2946:   {nullptr} /* sentinel */
2947: };
2948: 
2949: 
2950: 
2951: static struct PyGetSetDef Unique2Backward0_properties[] = {
2952:   THP_FUNCTION_DEFAULT_PROPERTIES,
2953: 
2954:   {nullptr} /* sentinel */
2955: };
2956: 
2957: static PyObject* THPViewBackward0_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
2958:   HANDLE_TH_ERRORS
2959:   auto prop = static_cast<ViewBackward0*>(self->cdata.get())->self_sym_sizes;
2960:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
```

- EN: The main execution path in this span is carried by `PyLong_FromUnsignedLong`, `cast`, `THPToSparseBackward0_self_self_sym_blocksize_opt_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyLong_FromUnsignedLong`, `cast`, `THPToSparseBackward0_self_self_sym_blocksize_opt_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2961-3040

```cpp
2961:   for (auto i : c10::irange(prop.size())) {
2962:       auto si = prop[i];
2963:       if (auto m = si.maybe_as_int()) {
2964:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
2965:       } else {
2966:         auto py_symint = py::cast(si).release().ptr();
2967:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
2968:       }
2969:   }
2970:   return tup;
2971:   END_HANDLE_TH_ERRORS
2972: }
2973: 
2974: static struct PyGetSetDef ViewBackward0_properties[] = {
2975:   THP_FUNCTION_DEFAULT_PROPERTIES,
2976:   {(char*)"_saved_self_sym_sizes", (getter)THPViewBackward0_self_sym_sizes_getter, nullptr, nullptr, nullptr},
2977:   {nullptr} /* sentinel */
2978: };
2979: 
2980: static PyObject* THPViewBackwardAutogradNestedTensor0_self_getter(THPCppFunction *self, void *_unused) {
2981:   HANDLE_TH_ERRORS
2982:   const auto& prop = static_cast<ViewBackwardAutogradNestedTensor0*>(self->cdata.get())->self_;
2983:   return THPVariable_Wrap(prop.unpack(self->cdata));
2984:   END_HANDLE_TH_ERRORS
2985: }
2986: 
2987: static PyObject* THPViewBackwardAutogradNestedTensor0_self_raw_getter(THPCppFunction *self, void *_unused) {
2988:   HANDLE_TH_ERRORS
2989:   const auto& prop = static_cast<ViewBackwardAutogradNestedTensor0*>(self->cdata.get())->self_;
2990:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2991:   return obj.release().ptr();
2992:   END_HANDLE_TH_ERRORS
2993: }
2994: 
2995: static struct PyGetSetDef ViewBackwardAutogradNestedTensor0_properties[] = {
2996:   THP_FUNCTION_DEFAULT_PROPERTIES,
2997:   {(char*)"_saved_self", (getter)THPViewBackwardAutogradNestedTensor0_self_getter, nullptr, nullptr, nullptr},
2998:   {(char*)"_raw_saved_self", (getter)THPViewBackwardAutogradNestedTensor0_self_raw_getter, nullptr, nullptr, nullptr},
2999:   {nullptr} /* sentinel */
3000: };
3001: 
3002: static PyObject* THPTrilinearBackward0_expand1_getter(THPCppFunction *self, void *_unused) {
3003:   HANDLE_TH_ERRORS
3004:   auto prop = static_cast<TrilinearBackward0*>(self->cdata.get())->expand1;
3005:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3006:   for (auto i : c10::irange(prop.size())) {
3007:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
3008:   }
3009:   return tup;
3010:   END_HANDLE_TH_ERRORS
3011: }
3012: 
3013: static PyObject* THPTrilinearBackward0_expand2_getter(THPCppFunction *self, void *_unused) {
3014:   HANDLE_TH_ERRORS
3015:   auto prop = static_cast<TrilinearBackward0*>(self->cdata.get())->expand2;
3016:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3017:   for (auto i : c10::irange(prop.size())) {
3018:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
3019:   }
3020:   return tup;
3021:   END_HANDLE_TH_ERRORS
3022: }
3023: 
3024: static PyObject* THPTrilinearBackward0_expand3_getter(THPCppFunction *self, void *_unused) {
3025:   HANDLE_TH_ERRORS
3026:   auto prop = static_cast<TrilinearBackward0*>(self->cdata.get())->expand3;
3027:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3028:   for (auto i : c10::irange(prop.size())) {
3029:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
3030:   }
3031:   return tup;
3032:   END_HANDLE_TH_ERRORS
3033: }
3034: 
3035: static PyObject* THPTrilinearBackward0_i1_getter(THPCppFunction *self, void *_unused) {
3036:   HANDLE_TH_ERRORS
3037:   const auto& prop = static_cast<TrilinearBackward0*>(self->cdata.get())->i1_;
3038:   return THPVariable_Wrap(prop.unpack(self->cdata));
3039:   END_HANDLE_TH_ERRORS
3040: }
```

- EN: The main execution path in this span is carried by `PyTuple_SetItem`, `cast`, `THPViewBackwardAutogradNestedTensor0_self_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `PyTuple_SetItem`, `cast`, `THPViewBackwardAutogradNestedTensor0_self_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 3041-3120

```cpp
3041: 
3042: static PyObject* THPTrilinearBackward0_i1_raw_getter(THPCppFunction *self, void *_unused) {
3043:   HANDLE_TH_ERRORS
3044:   const auto& prop = static_cast<TrilinearBackward0*>(self->cdata.get())->i1_;
3045:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3046:   return obj.release().ptr();
3047:   END_HANDLE_TH_ERRORS
3048: }
3049: 
3050: static PyObject* THPTrilinearBackward0_i2_getter(THPCppFunction *self, void *_unused) {
3051:   HANDLE_TH_ERRORS
3052:   const auto& prop = static_cast<TrilinearBackward0*>(self->cdata.get())->i2_;
3053:   return THPVariable_Wrap(prop.unpack(self->cdata));
3054:   END_HANDLE_TH_ERRORS
3055: }
3056: 
3057: static PyObject* THPTrilinearBackward0_i2_raw_getter(THPCppFunction *self, void *_unused) {
3058:   HANDLE_TH_ERRORS
3059:   const auto& prop = static_cast<TrilinearBackward0*>(self->cdata.get())->i2_;
3060:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3061:   return obj.release().ptr();
3062:   END_HANDLE_TH_ERRORS
3063: }
3064: 
3065: static PyObject* THPTrilinearBackward0_i3_getter(THPCppFunction *self, void *_unused) {
3066:   HANDLE_TH_ERRORS
3067:   const auto& prop = static_cast<TrilinearBackward0*>(self->cdata.get())->i3_;
3068:   return THPVariable_Wrap(prop.unpack(self->cdata));
3069:   END_HANDLE_TH_ERRORS
3070: }
3071: 
3072: static PyObject* THPTrilinearBackward0_i3_raw_getter(THPCppFunction *self, void *_unused) {
3073:   HANDLE_TH_ERRORS
3074:   const auto& prop = static_cast<TrilinearBackward0*>(self->cdata.get())->i3_;
3075:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3076:   return obj.release().ptr();
3077:   END_HANDLE_TH_ERRORS
3078: }
3079: 
3080: static PyObject* THPTrilinearBackward0_sumdim_getter(THPCppFunction *self, void *_unused) {
3081:   HANDLE_TH_ERRORS
3082:   auto prop = static_cast<TrilinearBackward0*>(self->cdata.get())->sumdim;
3083:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3084:   for (auto i : c10::irange(prop.size())) {
3085:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
3086:   }
3087:   return tup;
3088:   END_HANDLE_TH_ERRORS
3089: }
3090: 
3091: static struct PyGetSetDef TrilinearBackward0_properties[] = {
3092:   THP_FUNCTION_DEFAULT_PROPERTIES,
3093:   {(char*)"_saved_expand1", (getter)THPTrilinearBackward0_expand1_getter, nullptr, nullptr, nullptr},
3094:   {(char*)"_saved_expand2", (getter)THPTrilinearBackward0_expand2_getter, nullptr, nullptr, nullptr},
3095:   {(char*)"_saved_expand3", (getter)THPTrilinearBackward0_expand3_getter, nullptr, nullptr, nullptr},
3096:   {(char*)"_saved_i1", (getter)THPTrilinearBackward0_i1_getter, nullptr, nullptr, nullptr},
3097:   {(char*)"_raw_saved_i1", (getter)THPTrilinearBackward0_i1_raw_getter, nullptr, nullptr, nullptr},
3098:   {(char*)"_saved_i2", (getter)THPTrilinearBackward0_i2_getter, nullptr, nullptr, nullptr},
3099:   {(char*)"_raw_saved_i2", (getter)THPTrilinearBackward0_i2_raw_getter, nullptr, nullptr, nullptr},
3100:   {(char*)"_saved_i3", (getter)THPTrilinearBackward0_i3_getter, nullptr, nullptr, nullptr},
3101:   {(char*)"_raw_saved_i3", (getter)THPTrilinearBackward0_i3_raw_getter, nullptr, nullptr, nullptr},
3102:   {(char*)"_saved_sumdim", (getter)THPTrilinearBackward0_sumdim_getter, nullptr, nullptr, nullptr},
3103:   {nullptr} /* sentinel */
3104: };
3105: 
3106: static PyObject* THPBinaryCrossEntropyBackward0_reduction_getter(THPCppFunction *self, void *_unused) {
3107:   HANDLE_TH_ERRORS
3108:   auto prop = static_cast<BinaryCrossEntropyBackward0*>(self->cdata.get())->reduction;
3109:   return PyLong_FromUnsignedLong((int64_t) prop);
3110:   END_HANDLE_TH_ERRORS
3111: }
3112: 
3113: static PyObject* THPBinaryCrossEntropyBackward0_self_getter(THPCppFunction *self, void *_unused) {
3114:   HANDLE_TH_ERRORS
3115:   const auto& prop = static_cast<BinaryCrossEntropyBackward0*>(self->cdata.get())->self_;
3116:   return THPVariable_Wrap(prop.unpack(self->cdata));
3117:   END_HANDLE_TH_ERRORS
3118: }
3119: 
3120: static PyObject* THPBinaryCrossEntropyBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
```

- EN: The main execution path in this span is carried by `THPTrilinearBackward0_i1_raw_getter`, `cast`, `THPTrilinearBackward0_i2_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPTrilinearBackward0_i1_raw_getter`, `cast`, `THPTrilinearBackward0_i2_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3121-3200

```cpp
3121:   HANDLE_TH_ERRORS
3122:   const auto& prop = static_cast<BinaryCrossEntropyBackward0*>(self->cdata.get())->self_;
3123:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3124:   return obj.release().ptr();
3125:   END_HANDLE_TH_ERRORS
3126: }
3127: 
3128: static PyObject* THPBinaryCrossEntropyBackward0_target_getter(THPCppFunction *self, void *_unused) {
3129:   HANDLE_TH_ERRORS
3130:   const auto& prop = static_cast<BinaryCrossEntropyBackward0*>(self->cdata.get())->target_;
3131:   return THPVariable_Wrap(prop.unpack(self->cdata));
3132:   END_HANDLE_TH_ERRORS
3133: }
3134: 
3135: static PyObject* THPBinaryCrossEntropyBackward0_target_raw_getter(THPCppFunction *self, void *_unused) {
3136:   HANDLE_TH_ERRORS
3137:   const auto& prop = static_cast<BinaryCrossEntropyBackward0*>(self->cdata.get())->target_;
3138:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3139:   return obj.release().ptr();
3140:   END_HANDLE_TH_ERRORS
3141: }
3142: 
3143: static PyObject* THPBinaryCrossEntropyBackward0_weight_getter(THPCppFunction *self, void *_unused) {
3144:   HANDLE_TH_ERRORS
3145:   const auto& prop = static_cast<BinaryCrossEntropyBackward0*>(self->cdata.get())->weight_;
3146:   return THPVariable_Wrap(prop.unpack(self->cdata));
3147:   END_HANDLE_TH_ERRORS
3148: }
3149: 
3150: static PyObject* THPBinaryCrossEntropyBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
3151:   HANDLE_TH_ERRORS
3152:   const auto& prop = static_cast<BinaryCrossEntropyBackward0*>(self->cdata.get())->weight_;
3153:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3154:   return obj.release().ptr();
3155:   END_HANDLE_TH_ERRORS
3156: }
3157: 
3158: static struct PyGetSetDef BinaryCrossEntropyBackward0_properties[] = {
3159:   THP_FUNCTION_DEFAULT_PROPERTIES,
3160:   {(char*)"_saved_reduction", (getter)THPBinaryCrossEntropyBackward0_reduction_getter, nullptr, nullptr, nullptr},
3161:   {(char*)"_saved_self", (getter)THPBinaryCrossEntropyBackward0_self_getter, nullptr, nullptr, nullptr},
3162:   {(char*)"_raw_saved_self", (getter)THPBinaryCrossEntropyBackward0_self_raw_getter, nullptr, nullptr, nullptr},
3163:   {(char*)"_saved_target", (getter)THPBinaryCrossEntropyBackward0_target_getter, nullptr, nullptr, nullptr},
3164:   {(char*)"_raw_saved_target", (getter)THPBinaryCrossEntropyBackward0_target_raw_getter, nullptr, nullptr, nullptr},
3165:   {(char*)"_saved_weight", (getter)THPBinaryCrossEntropyBackward0_weight_getter, nullptr, nullptr, nullptr},
3166:   {(char*)"_raw_saved_weight", (getter)THPBinaryCrossEntropyBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
3167:   {nullptr} /* sentinel */
3168: };
3169: 
3170: 
3171: 
3172: static struct PyGetSetDef EmbeddingRenormBackward0_properties[] = {
3173:   THP_FUNCTION_DEFAULT_PROPERTIES,
3174: 
3175:   {nullptr} /* sentinel */
3176: };
3177: 
3178: static PyObject* THPHuberLossBackward0_delta_getter(THPCppFunction *self, void *_unused) {
3179:   HANDLE_TH_ERRORS
3180:   auto prop = static_cast<HuberLossBackward0*>(self->cdata.get())->delta;
3181:   return PyFloat_FromDouble((double) prop);
3182:   END_HANDLE_TH_ERRORS
3183: }
3184: 
3185: static PyObject* THPHuberLossBackward0_reduction_getter(THPCppFunction *self, void *_unused) {
3186:   HANDLE_TH_ERRORS
3187:   auto prop = static_cast<HuberLossBackward0*>(self->cdata.get())->reduction;
3188:   return PyLong_FromUnsignedLong((int64_t) prop);
3189:   END_HANDLE_TH_ERRORS
3190: }
3191: 
3192: static PyObject* THPHuberLossBackward0_self_getter(THPCppFunction *self, void *_unused) {
3193:   HANDLE_TH_ERRORS
3194:   const auto& prop = static_cast<HuberLossBackward0*>(self->cdata.get())->self_;
3195:   return THPVariable_Wrap(prop.unpack(self->cdata));
3196:   END_HANDLE_TH_ERRORS
3197: }
3198: 
3199: static PyObject* THPHuberLossBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
3200:   HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `cast`, `THPBinaryCrossEntropyBackward0_target_getter`, `THPVariable_Wrap`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `cast`, `THPBinaryCrossEntropyBackward0_target_getter`, `THPVariable_Wrap` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3201-3280

```cpp
3201:   const auto& prop = static_cast<HuberLossBackward0*>(self->cdata.get())->self_;
3202:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3203:   return obj.release().ptr();
3204:   END_HANDLE_TH_ERRORS
3205: }
3206: 
3207: static PyObject* THPHuberLossBackward0_target_getter(THPCppFunction *self, void *_unused) {
3208:   HANDLE_TH_ERRORS
3209:   const auto& prop = static_cast<HuberLossBackward0*>(self->cdata.get())->target_;
3210:   return THPVariable_Wrap(prop.unpack(self->cdata));
3211:   END_HANDLE_TH_ERRORS
3212: }
3213: 
3214: static PyObject* THPHuberLossBackward0_target_raw_getter(THPCppFunction *self, void *_unused) {
3215:   HANDLE_TH_ERRORS
3216:   const auto& prop = static_cast<HuberLossBackward0*>(self->cdata.get())->target_;
3217:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3218:   return obj.release().ptr();
3219:   END_HANDLE_TH_ERRORS
3220: }
3221: 
3222: static struct PyGetSetDef HuberLossBackward0_properties[] = {
3223:   THP_FUNCTION_DEFAULT_PROPERTIES,
3224:   {(char*)"_saved_delta", (getter)THPHuberLossBackward0_delta_getter, nullptr, nullptr, nullptr},
3225:   {(char*)"_saved_reduction", (getter)THPHuberLossBackward0_reduction_getter, nullptr, nullptr, nullptr},
3226:   {(char*)"_saved_self", (getter)THPHuberLossBackward0_self_getter, nullptr, nullptr, nullptr},
3227:   {(char*)"_raw_saved_self", (getter)THPHuberLossBackward0_self_raw_getter, nullptr, nullptr, nullptr},
3228:   {(char*)"_saved_target", (getter)THPHuberLossBackward0_target_getter, nullptr, nullptr, nullptr},
3229:   {(char*)"_raw_saved_target", (getter)THPHuberLossBackward0_target_raw_getter, nullptr, nullptr, nullptr},
3230:   {nullptr} /* sentinel */
3231: };
3232: 
3233: static PyObject* THPSiluBackward0_self_getter(THPCppFunction *self, void *_unused) {
3234:   HANDLE_TH_ERRORS
3235:   const auto& prop = static_cast<SiluBackward0*>(self->cdata.get())->self_;
3236:   return THPVariable_Wrap(prop.unpack(self->cdata));
3237:   END_HANDLE_TH_ERRORS
3238: }
3239: 
3240: static PyObject* THPSiluBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
3241:   HANDLE_TH_ERRORS
3242:   const auto& prop = static_cast<SiluBackward0*>(self->cdata.get())->self_;
3243:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3244:   return obj.release().ptr();
3245:   END_HANDLE_TH_ERRORS
3246: }
3247: 
3248: static struct PyGetSetDef SiluBackward0_properties[] = {
3249:   THP_FUNCTION_DEFAULT_PROPERTIES,
3250:   {(char*)"_saved_self", (getter)THPSiluBackward0_self_getter, nullptr, nullptr, nullptr},
3251:   {(char*)"_raw_saved_self", (getter)THPSiluBackward0_self_raw_getter, nullptr, nullptr, nullptr},
3252:   {nullptr} /* sentinel */
3253: };
3254: 
3255: static PyObject* THPCeluBackward0_alpha_getter(THPCppFunction *self, void *_unused) {
3256:   HANDLE_TH_ERRORS
3257:   auto prop = static_cast<CeluBackward0*>(self->cdata.get())->alpha;
3258:   if (prop.isComplex()) {
3259:     auto cprop = prop.to<c10::complex<double>>();
3260:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
3261:   } else if (prop.isFloatingPoint()) {
3262:     return PyFloat_FromDouble(prop.to<double>());
3263:   } else if (prop.isIntegral(/*includeBool=*/false)) {
3264:     return PyLong_FromLong(prop.to<int64_t>());
3265:   } else if (prop.isBoolean()) {
3266:     if (prop.to<bool>()) {
3267:       Py_RETURN_TRUE;
3268:     } else {
3269:       Py_RETURN_FALSE;
3270:     }
3271:   } else {
3272:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
3273:     return nullptr;
3274:   }
3275:   END_HANDLE_TH_ERRORS
3276: }
3277: 
3278: static PyObject* THPCeluBackward0_self_getter(THPCppFunction *self, void *_unused) {
3279:   HANDLE_TH_ERRORS
3280:   const auto& prop = static_cast<CeluBackward0*>(self->cdata.get())->self_;
```

- EN: The main execution path in this span is carried by `cast`, `THPHuberLossBackward0_target_getter`, `THPVariable_Wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `cast`, `THPHuberLossBackward0_target_getter`, `THPVariable_Wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3281-3360

```cpp
3281:   return THPVariable_Wrap(prop.unpack(self->cdata));
3282:   END_HANDLE_TH_ERRORS
3283: }
3284: 
3285: static PyObject* THPCeluBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
3286:   HANDLE_TH_ERRORS
3287:   const auto& prop = static_cast<CeluBackward0*>(self->cdata.get())->self_;
3288:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3289:   return obj.release().ptr();
3290:   END_HANDLE_TH_ERRORS
3291: }
3292: 
3293: static struct PyGetSetDef CeluBackward0_properties[] = {
3294:   THP_FUNCTION_DEFAULT_PROPERTIES,
3295:   {(char*)"_saved_alpha", (getter)THPCeluBackward0_alpha_getter, nullptr, nullptr, nullptr},
3296:   {(char*)"_saved_self", (getter)THPCeluBackward0_self_getter, nullptr, nullptr, nullptr},
3297:   {(char*)"_raw_saved_self", (getter)THPCeluBackward0_self_raw_getter, nullptr, nullptr, nullptr},
3298:   {nullptr} /* sentinel */
3299: };
3300: 
3301: static PyObject* THPCeluBackward1_alpha_getter(THPCppFunction *self, void *_unused) {
3302:   HANDLE_TH_ERRORS
3303:   auto prop = static_cast<CeluBackward1*>(self->cdata.get())->alpha;
3304:   if (prop.isComplex()) {
3305:     auto cprop = prop.to<c10::complex<double>>();
3306:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
3307:   } else if (prop.isFloatingPoint()) {
3308:     return PyFloat_FromDouble(prop.to<double>());
3309:   } else if (prop.isIntegral(/*includeBool=*/false)) {
3310:     return PyLong_FromLong(prop.to<int64_t>());
3311:   } else if (prop.isBoolean()) {
3312:     if (prop.to<bool>()) {
3313:       Py_RETURN_TRUE;
3314:     } else {
3315:       Py_RETURN_FALSE;
3316:     }
3317:   } else {
3318:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
3319:     return nullptr;
3320:   }
3321:   END_HANDLE_TH_ERRORS
3322: }
3323: 
3324: static PyObject* THPCeluBackward1_result_getter(THPCppFunction *self, void *_unused) {
3325:   HANDLE_TH_ERRORS
3326:   const auto& prop = static_cast<CeluBackward1*>(self->cdata.get())->result_;
3327:   return THPVariable_Wrap(prop.unpack(self->cdata));
3328:   END_HANDLE_TH_ERRORS
3329: }
3330: 
3331: static PyObject* THPCeluBackward1_result_raw_getter(THPCppFunction *self, void *_unused) {
3332:   HANDLE_TH_ERRORS
3333:   const auto& prop = static_cast<CeluBackward1*>(self->cdata.get())->result_;
3334:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3335:   return obj.release().ptr();
3336:   END_HANDLE_TH_ERRORS
3337: }
3338: 
3339: static struct PyGetSetDef CeluBackward1_properties[] = {
3340:   THP_FUNCTION_DEFAULT_PROPERTIES,
3341:   {(char*)"_saved_alpha", (getter)THPCeluBackward1_alpha_getter, nullptr, nullptr, nullptr},
3342:   {(char*)"_saved_result", (getter)THPCeluBackward1_result_getter, nullptr, nullptr, nullptr},
3343:   {(char*)"_raw_saved_result", (getter)THPCeluBackward1_result_raw_getter, nullptr, nullptr, nullptr},
3344:   {nullptr} /* sentinel */
3345: };
3346: 
3347: static PyObject* THPSparseLogSoftmaxBackward0_dim_getter(THPCppFunction *self, void *_unused) {
3348:   HANDLE_TH_ERRORS
3349:   auto prop = static_cast<SparseLogSoftmaxBackward0*>(self->cdata.get())->dim;
3350:   return PyLong_FromUnsignedLong((int64_t) prop);
3351:   END_HANDLE_TH_ERRORS
3352: }
3353: 
3354: static PyObject* THPSparseLogSoftmaxBackward0_self_getter(THPCppFunction *self, void *_unused) {
3355:   HANDLE_TH_ERRORS
3356:   const auto& prop = static_cast<SparseLogSoftmaxBackward0*>(self->cdata.get())->self_;
3357:   return THPVariable_Wrap(prop.unpack(self->cdata));
3358:   END_HANDLE_TH_ERRORS
3359: }
3360: 
```

- EN: The main execution path in this span is carried by `THPVariable_Wrap`, `THPCeluBackward0_self_raw_getter`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_Wrap`, `THPCeluBackward0_self_raw_getter`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3361-3440

```cpp
3361: static PyObject* THPSparseLogSoftmaxBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
3362:   HANDLE_TH_ERRORS
3363:   const auto& prop = static_cast<SparseLogSoftmaxBackward0*>(self->cdata.get())->self_;
3364:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3365:   return obj.release().ptr();
3366:   END_HANDLE_TH_ERRORS
3367: }
3368: 
3369: static PyObject* THPSparseLogSoftmaxBackward0_result_getter(THPCppFunction *self, void *_unused) {
3370:   HANDLE_TH_ERRORS
3371:   const auto& prop = static_cast<SparseLogSoftmaxBackward0*>(self->cdata.get())->result_;
3372:   return THPVariable_Wrap(prop.unpack(self->cdata));
3373:   END_HANDLE_TH_ERRORS
3374: }
3375: 
3376: static PyObject* THPSparseLogSoftmaxBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
3377:   HANDLE_TH_ERRORS
3378:   const auto& prop = static_cast<SparseLogSoftmaxBackward0*>(self->cdata.get())->result_;
3379:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3380:   return obj.release().ptr();
3381:   END_HANDLE_TH_ERRORS
3382: }
3383: 
3384: static struct PyGetSetDef SparseLogSoftmaxBackward0_properties[] = {
3385:   THP_FUNCTION_DEFAULT_PROPERTIES,
3386:   {(char*)"_saved_dim", (getter)THPSparseLogSoftmaxBackward0_dim_getter, nullptr, nullptr, nullptr},
3387:   {(char*)"_saved_self", (getter)THPSparseLogSoftmaxBackward0_self_getter, nullptr, nullptr, nullptr},
3388:   {(char*)"_raw_saved_self", (getter)THPSparseLogSoftmaxBackward0_self_raw_getter, nullptr, nullptr, nullptr},
3389:   {(char*)"_saved_result", (getter)THPSparseLogSoftmaxBackward0_result_getter, nullptr, nullptr, nullptr},
3390:   {(char*)"_raw_saved_result", (getter)THPSparseLogSoftmaxBackward0_result_raw_getter, nullptr, nullptr, nullptr},
3391:   {nullptr} /* sentinel */
3392: };
3393: 
3394: static PyObject* THPMaskedSoftmaxBackward0_dim_getter(THPCppFunction *self, void *_unused) {
3395:   HANDLE_TH_ERRORS
3396:   auto opt_prop = static_cast<MaskedSoftmaxBackward0*>(self->cdata.get())->dim;
3397:   if (!opt_prop.has_value()) {
3398:     Py_RETURN_NONE;
3399:   }
3400:   auto prop = opt_prop.value();
3401:   return PyLong_FromUnsignedLong((int64_t) prop);
3402:   END_HANDLE_TH_ERRORS
3403: }
3404: 
3405: static PyObject* THPMaskedSoftmaxBackward0_mask_getter(THPCppFunction *self, void *_unused) {
3406:   HANDLE_TH_ERRORS
3407:   const auto& prop = static_cast<MaskedSoftmaxBackward0*>(self->cdata.get())->mask_;
3408:   return THPVariable_Wrap(prop.unpack(self->cdata));
3409:   END_HANDLE_TH_ERRORS
3410: }
3411: 
3412: static PyObject* THPMaskedSoftmaxBackward0_mask_raw_getter(THPCppFunction *self, void *_unused) {
3413:   HANDLE_TH_ERRORS
3414:   const auto& prop = static_cast<MaskedSoftmaxBackward0*>(self->cdata.get())->mask_;
3415:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3416:   return obj.release().ptr();
3417:   END_HANDLE_TH_ERRORS
3418: }
3419: 
3420: static PyObject* THPMaskedSoftmaxBackward0_result_getter(THPCppFunction *self, void *_unused) {
3421:   HANDLE_TH_ERRORS
3422:   const auto& prop = static_cast<MaskedSoftmaxBackward0*>(self->cdata.get())->result_;
3423:   return THPVariable_Wrap(prop.unpack(self->cdata));
3424:   END_HANDLE_TH_ERRORS
3425: }
3426: 
3427: static PyObject* THPMaskedSoftmaxBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
3428:   HANDLE_TH_ERRORS
3429:   const auto& prop = static_cast<MaskedSoftmaxBackward0*>(self->cdata.get())->result_;
3430:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3431:   return obj.release().ptr();
3432:   END_HANDLE_TH_ERRORS
3433: }
3434: 
3435: static struct PyGetSetDef MaskedSoftmaxBackward0_properties[] = {
3436:   THP_FUNCTION_DEFAULT_PROPERTIES,
3437:   {(char*)"_saved_dim", (getter)THPMaskedSoftmaxBackward0_dim_getter, nullptr, nullptr, nullptr},
3438:   {(char*)"_saved_mask", (getter)THPMaskedSoftmaxBackward0_mask_getter, nullptr, nullptr, nullptr},
3439:   {(char*)"_raw_saved_mask", (getter)THPMaskedSoftmaxBackward0_mask_raw_getter, nullptr, nullptr, nullptr},
3440:   {(char*)"_saved_result", (getter)THPMaskedSoftmaxBackward0_result_getter, nullptr, nullptr, nullptr},
```

- EN: The main execution path in this span is carried by `THPSparseLogSoftmaxBackward0_self_raw_getter`, `cast`, `THPSparseLogSoftmaxBackward0_result_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPSparseLogSoftmaxBackward0_self_raw_getter`, `cast`, `THPSparseLogSoftmaxBackward0_result_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3441-3520

```cpp
3441:   {(char*)"_raw_saved_result", (getter)THPMaskedSoftmaxBackward0_result_raw_getter, nullptr, nullptr, nullptr},
3442:   {nullptr} /* sentinel */
3443: };
3444: 
3445: static PyObject* THPSoftmaxBackward0_dim_getter(THPCppFunction *self, void *_unused) {
3446:   HANDLE_TH_ERRORS
3447:   auto prop = static_cast<SoftmaxBackward0*>(self->cdata.get())->dim;
3448:   return PyLong_FromUnsignedLong((int64_t) prop);
3449:   END_HANDLE_TH_ERRORS
3450: }
3451: 
3452: static PyObject* THPSoftmaxBackward0_result_getter(THPCppFunction *self, void *_unused) {
3453:   HANDLE_TH_ERRORS
3454:   const auto& prop = static_cast<SoftmaxBackward0*>(self->cdata.get())->result_;
3455:   return THPVariable_Wrap(prop.unpack(self->cdata));
3456:   END_HANDLE_TH_ERRORS
3457: }
3458: 
3459: static PyObject* THPSoftmaxBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
3460:   HANDLE_TH_ERRORS
3461:   const auto& prop = static_cast<SoftmaxBackward0*>(self->cdata.get())->result_;
3462:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3463:   return obj.release().ptr();
3464:   END_HANDLE_TH_ERRORS
3465: }
3466: 
3467: static struct PyGetSetDef SoftmaxBackward0_properties[] = {
3468:   THP_FUNCTION_DEFAULT_PROPERTIES,
3469:   {(char*)"_saved_dim", (getter)THPSoftmaxBackward0_dim_getter, nullptr, nullptr, nullptr},
3470:   {(char*)"_saved_result", (getter)THPSoftmaxBackward0_result_getter, nullptr, nullptr, nullptr},
3471:   {(char*)"_raw_saved_result", (getter)THPSoftmaxBackward0_result_raw_getter, nullptr, nullptr, nullptr},
3472:   {nullptr} /* sentinel */
3473: };
3474: 
3475: static PyObject* THPThresholdBackward0_self_getter(THPCppFunction *self, void *_unused) {
3476:   HANDLE_TH_ERRORS
3477:   const auto& prop = static_cast<ThresholdBackward0*>(self->cdata.get())->self_;
3478:   return THPVariable_Wrap(prop.unpack(self->cdata));
3479:   END_HANDLE_TH_ERRORS
3480: }
3481: 
3482: static PyObject* THPThresholdBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
3483:   HANDLE_TH_ERRORS
3484:   const auto& prop = static_cast<ThresholdBackward0*>(self->cdata.get())->self_;
3485:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3486:   return obj.release().ptr();
3487:   END_HANDLE_TH_ERRORS
3488: }
3489: 
3490: static PyObject* THPThresholdBackward0_threshold_getter(THPCppFunction *self, void *_unused) {
3491:   HANDLE_TH_ERRORS
3492:   auto prop = static_cast<ThresholdBackward0*>(self->cdata.get())->threshold;
3493:   if (prop.isComplex()) {
3494:     auto cprop = prop.to<c10::complex<double>>();
3495:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
3496:   } else if (prop.isFloatingPoint()) {
3497:     return PyFloat_FromDouble(prop.to<double>());
3498:   } else if (prop.isIntegral(/*includeBool=*/false)) {
3499:     return PyLong_FromLong(prop.to<int64_t>());
3500:   } else if (prop.isBoolean()) {
3501:     if (prop.to<bool>()) {
3502:       Py_RETURN_TRUE;
3503:     } else {
3504:       Py_RETURN_FALSE;
3505:     }
3506:   } else {
3507:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
3508:     return nullptr;
3509:   }
3510:   END_HANDLE_TH_ERRORS
3511: }
3512: 
3513: static struct PyGetSetDef ThresholdBackward0_properties[] = {
3514:   THP_FUNCTION_DEFAULT_PROPERTIES,
3515:   {(char*)"_saved_self", (getter)THPThresholdBackward0_self_getter, nullptr, nullptr, nullptr},
3516:   {(char*)"_raw_saved_self", (getter)THPThresholdBackward0_self_raw_getter, nullptr, nullptr, nullptr},
3517:   {(char*)"_saved_threshold", (getter)THPThresholdBackward0_threshold_getter, nullptr, nullptr, nullptr},
3518:   {nullptr} /* sentinel */
3519: };
3520: 
```

- EN: The main execution path in this span is carried by `THPSoftmaxBackward0_dim_getter`, `PyLong_FromUnsignedLong`, `THPSoftmaxBackward0_result_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPSoftmaxBackward0_dim_getter`, `PyLong_FromUnsignedLong`, `THPSoftmaxBackward0_result_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3521-3600

```cpp
3521: static PyObject* THPReplicationPad3DBackward0_padding_getter(THPCppFunction *self, void *_unused) {
3522:   HANDLE_TH_ERRORS
3523:   auto prop = static_cast<ReplicationPad3DBackward0*>(self->cdata.get())->padding;
3524:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3525:   for (auto i : c10::irange(prop.size())) {
3526:       auto si = prop[i];
3527:       if (auto m = si.maybe_as_int()) {
3528:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3529:       } else {
3530:         auto py_symint = py::cast(si).release().ptr();
3531:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3532:       }
3533:   }
3534:   return tup;
3535:   END_HANDLE_TH_ERRORS
3536: }
3537: 
3538: static PyObject* THPReplicationPad3DBackward0_self_getter(THPCppFunction *self, void *_unused) {
3539:   HANDLE_TH_ERRORS
3540:   const auto& prop = static_cast<ReplicationPad3DBackward0*>(self->cdata.get())->self_;
3541:   return THPVariable_Wrap(prop.unpack(self->cdata));
3542:   END_HANDLE_TH_ERRORS
3543: }
3544: 
3545: static PyObject* THPReplicationPad3DBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
3546:   HANDLE_TH_ERRORS
3547:   const auto& prop = static_cast<ReplicationPad3DBackward0*>(self->cdata.get())->self_;
3548:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3549:   return obj.release().ptr();
3550:   END_HANDLE_TH_ERRORS
3551: }
3552: 
3553: static struct PyGetSetDef ReplicationPad3DBackward0_properties[] = {
3554:   THP_FUNCTION_DEFAULT_PROPERTIES,
3555:   {(char*)"_saved_padding", (getter)THPReplicationPad3DBackward0_padding_getter, nullptr, nullptr, nullptr},
3556:   {(char*)"_saved_self", (getter)THPReplicationPad3DBackward0_self_getter, nullptr, nullptr, nullptr},
3557:   {(char*)"_raw_saved_self", (getter)THPReplicationPad3DBackward0_self_raw_getter, nullptr, nullptr, nullptr},
3558:   {nullptr} /* sentinel */
3559: };
3560: 
3561: static PyObject* THPUpsampleLinear1DBackward0_align_corners_getter(THPCppFunction *self, void *_unused) {
3562:   HANDLE_TH_ERRORS
3563:   auto prop = static_cast<UpsampleLinear1DBackward0*>(self->cdata.get())->align_corners;
3564:   if (prop) {
3565:     Py_RETURN_TRUE;
3566:   } else {
3567:     Py_RETURN_FALSE;
3568:   }
3569:   END_HANDLE_TH_ERRORS
3570: }
3571: 
3572: static PyObject* THPUpsampleLinear1DBackward0_output_size_getter(THPCppFunction *self, void *_unused) {
3573:   HANDLE_TH_ERRORS
3574:   auto prop = static_cast<UpsampleLinear1DBackward0*>(self->cdata.get())->output_size;
3575:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3576:   for (auto i : c10::irange(prop.size())) {
3577:       auto si = prop[i];
3578:       if (auto m = si.maybe_as_int()) {
3579:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3580:       } else {
3581:         auto py_symint = py::cast(si).release().ptr();
3582:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3583:       }
3584:   }
3585:   return tup;
3586:   END_HANDLE_TH_ERRORS
3587: }
3588: 
3589: static PyObject* THPUpsampleLinear1DBackward0_scales_getter(THPCppFunction *self, void *_unused) {
3590:   HANDLE_TH_ERRORS
3591:   auto opt_prop = static_cast<UpsampleLinear1DBackward0*>(self->cdata.get())->scales;
3592:   if (!opt_prop.has_value()) {
3593:     Py_RETURN_NONE;
3594:   }
3595:   auto prop = opt_prop.value();
3596:   return PyFloat_FromDouble((double) prop);
3597:   END_HANDLE_TH_ERRORS
3598: }
3599: 
3600: static PyObject* THPUpsampleLinear1DBackward0_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
```

- EN: The main execution path in this span is carried by `THPReplicationPad3DBackward0_padding_getter`, `PyTuple_New`, `PyTuple_SetItem`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPReplicationPad3DBackward0_padding_getter`, `PyTuple_New`, `PyTuple_SetItem` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3601-3680

```cpp
3601:   HANDLE_TH_ERRORS
3602:   auto prop = static_cast<UpsampleLinear1DBackward0*>(self->cdata.get())->self_sym_sizes;
3603:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3604:   for (auto i : c10::irange(prop.size())) {
3605:       auto si = prop[i];
3606:       if (auto m = si.maybe_as_int()) {
3607:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3608:       } else {
3609:         auto py_symint = py::cast(si).release().ptr();
3610:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3611:       }
3612:   }
3613:   return tup;
3614:   END_HANDLE_TH_ERRORS
3615: }
3616: 
3617: static struct PyGetSetDef UpsampleLinear1DBackward0_properties[] = {
3618:   THP_FUNCTION_DEFAULT_PROPERTIES,
3619:   {(char*)"_saved_align_corners", (getter)THPUpsampleLinear1DBackward0_align_corners_getter, nullptr, nullptr, nullptr},
3620:   {(char*)"_saved_output_size", (getter)THPUpsampleLinear1DBackward0_output_size_getter, nullptr, nullptr, nullptr},
3621:   {(char*)"_saved_scales", (getter)THPUpsampleLinear1DBackward0_scales_getter, nullptr, nullptr, nullptr},
3622:   {(char*)"_saved_self_sym_sizes", (getter)THPUpsampleLinear1DBackward0_self_sym_sizes_getter, nullptr, nullptr, nullptr},
3623:   {nullptr} /* sentinel */
3624: };
3625: 
3626: static PyObject* THPUpsampleBilinear2DBackward0_align_corners_getter(THPCppFunction *self, void *_unused) {
3627:   HANDLE_TH_ERRORS
3628:   auto prop = static_cast<UpsampleBilinear2DBackward0*>(self->cdata.get())->align_corners;
3629:   if (prop) {
3630:     Py_RETURN_TRUE;
3631:   } else {
3632:     Py_RETURN_FALSE;
3633:   }
3634:   END_HANDLE_TH_ERRORS
3635: }
3636: 
3637: static PyObject* THPUpsampleBilinear2DBackward0_output_size_getter(THPCppFunction *self, void *_unused) {
3638:   HANDLE_TH_ERRORS
3639:   auto prop = static_cast<UpsampleBilinear2DBackward0*>(self->cdata.get())->output_size;
3640:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3641:   for (auto i : c10::irange(prop.size())) {
3642:       auto si = prop[i];
3643:       if (auto m = si.maybe_as_int()) {
3644:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3645:       } else {
3646:         auto py_symint = py::cast(si).release().ptr();
3647:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3648:       }
3649:   }
3650:   return tup;
3651:   END_HANDLE_TH_ERRORS
3652: }
3653: 
3654: static PyObject* THPUpsampleBilinear2DBackward0_scales_h_getter(THPCppFunction *self, void *_unused) {
3655:   HANDLE_TH_ERRORS
3656:   auto opt_prop = static_cast<UpsampleBilinear2DBackward0*>(self->cdata.get())->scales_h;
3657:   if (!opt_prop.has_value()) {
3658:     Py_RETURN_NONE;
3659:   }
3660:   auto prop = opt_prop.value();
3661:   return PyFloat_FromDouble((double) prop);
3662:   END_HANDLE_TH_ERRORS
3663: }
3664: 
3665: static PyObject* THPUpsampleBilinear2DBackward0_scales_w_getter(THPCppFunction *self, void *_unused) {
3666:   HANDLE_TH_ERRORS
3667:   auto opt_prop = static_cast<UpsampleBilinear2DBackward0*>(self->cdata.get())->scales_w;
3668:   if (!opt_prop.has_value()) {
3669:     Py_RETURN_NONE;
3670:   }
3671:   auto prop = opt_prop.value();
3672:   return PyFloat_FromDouble((double) prop);
3673:   END_HANDLE_TH_ERRORS
3674: }
3675: 
3676: static PyObject* THPUpsampleBilinear2DBackward0_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
3677:   HANDLE_TH_ERRORS
3678:   auto prop = static_cast<UpsampleBilinear2DBackward0*>(self->cdata.get())->self_sym_sizes;
3679:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3680:   for (auto i : c10::irange(prop.size())) {
```

- EN: The main execution path in this span is carried by `PyTuple_New`, `PyTuple_SetItem`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyTuple_New`, `PyTuple_SetItem`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3681-3760

```cpp
3681:       auto si = prop[i];
3682:       if (auto m = si.maybe_as_int()) {
3683:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3684:       } else {
3685:         auto py_symint = py::cast(si).release().ptr();
3686:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3687:       }
3688:   }
3689:   return tup;
3690:   END_HANDLE_TH_ERRORS
3691: }
3692: 
3693: static struct PyGetSetDef UpsampleBilinear2DBackward0_properties[] = {
3694:   THP_FUNCTION_DEFAULT_PROPERTIES,
3695:   {(char*)"_saved_align_corners", (getter)THPUpsampleBilinear2DBackward0_align_corners_getter, nullptr, nullptr, nullptr},
3696:   {(char*)"_saved_output_size", (getter)THPUpsampleBilinear2DBackward0_output_size_getter, nullptr, nullptr, nullptr},
3697:   {(char*)"_saved_scales_h", (getter)THPUpsampleBilinear2DBackward0_scales_h_getter, nullptr, nullptr, nullptr},
3698:   {(char*)"_saved_scales_w", (getter)THPUpsampleBilinear2DBackward0_scales_w_getter, nullptr, nullptr, nullptr},
3699:   {(char*)"_saved_self_sym_sizes", (getter)THPUpsampleBilinear2DBackward0_self_sym_sizes_getter, nullptr, nullptr, nullptr},
3700:   {nullptr} /* sentinel */
3701: };
3702: 
3703: static PyObject* THPUpsampleBicubic2DBackward0_align_corners_getter(THPCppFunction *self, void *_unused) {
3704:   HANDLE_TH_ERRORS
3705:   auto prop = static_cast<UpsampleBicubic2DBackward0*>(self->cdata.get())->align_corners;
3706:   if (prop) {
3707:     Py_RETURN_TRUE;
3708:   } else {
3709:     Py_RETURN_FALSE;
3710:   }
3711:   END_HANDLE_TH_ERRORS
3712: }
3713: 
3714: static PyObject* THPUpsampleBicubic2DBackward0_output_size_getter(THPCppFunction *self, void *_unused) {
3715:   HANDLE_TH_ERRORS
3716:   auto prop = static_cast<UpsampleBicubic2DBackward0*>(self->cdata.get())->output_size;
3717:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3718:   for (auto i : c10::irange(prop.size())) {
3719:       auto si = prop[i];
3720:       if (auto m = si.maybe_as_int()) {
3721:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3722:       } else {
3723:         auto py_symint = py::cast(si).release().ptr();
3724:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3725:       }
3726:   }
3727:   return tup;
3728:   END_HANDLE_TH_ERRORS
3729: }
3730: 
3731: static PyObject* THPUpsampleBicubic2DBackward0_scales_h_getter(THPCppFunction *self, void *_unused) {
3732:   HANDLE_TH_ERRORS
3733:   auto opt_prop = static_cast<UpsampleBicubic2DBackward0*>(self->cdata.get())->scales_h;
3734:   if (!opt_prop.has_value()) {
3735:     Py_RETURN_NONE;
3736:   }
3737:   auto prop = opt_prop.value();
3738:   return PyFloat_FromDouble((double) prop);
3739:   END_HANDLE_TH_ERRORS
3740: }
3741: 
3742: static PyObject* THPUpsampleBicubic2DBackward0_scales_w_getter(THPCppFunction *self, void *_unused) {
3743:   HANDLE_TH_ERRORS
3744:   auto opt_prop = static_cast<UpsampleBicubic2DBackward0*>(self->cdata.get())->scales_w;
3745:   if (!opt_prop.has_value()) {
3746:     Py_RETURN_NONE;
3747:   }
3748:   auto prop = opt_prop.value();
3749:   return PyFloat_FromDouble((double) prop);
3750:   END_HANDLE_TH_ERRORS
3751: }
3752: 
3753: static PyObject* THPUpsampleBicubic2DBackward0_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
3754:   HANDLE_TH_ERRORS
3755:   auto prop = static_cast<UpsampleBicubic2DBackward0*>(self->cdata.get())->self_sym_sizes;
3756:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3757:   for (auto i : c10::irange(prop.size())) {
3758:       auto si = prop[i];
3759:       if (auto m = si.maybe_as_int()) {
3760:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
```

- EN: The main execution path in this span is carried by `PyTuple_SetItem`, `cast`, `THPUpsampleBicubic2DBackward0_align_corners_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyTuple_SetItem`, `cast`, `THPUpsampleBicubic2DBackward0_align_corners_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3761-3840

```cpp
3761:       } else {
3762:         auto py_symint = py::cast(si).release().ptr();
3763:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3764:       }
3765:   }
3766:   return tup;
3767:   END_HANDLE_TH_ERRORS
3768: }
3769: 
3770: static struct PyGetSetDef UpsampleBicubic2DBackward0_properties[] = {
3771:   THP_FUNCTION_DEFAULT_PROPERTIES,
3772:   {(char*)"_saved_align_corners", (getter)THPUpsampleBicubic2DBackward0_align_corners_getter, nullptr, nullptr, nullptr},
3773:   {(char*)"_saved_output_size", (getter)THPUpsampleBicubic2DBackward0_output_size_getter, nullptr, nullptr, nullptr},
3774:   {(char*)"_saved_scales_h", (getter)THPUpsampleBicubic2DBackward0_scales_h_getter, nullptr, nullptr, nullptr},
3775:   {(char*)"_saved_scales_w", (getter)THPUpsampleBicubic2DBackward0_scales_w_getter, nullptr, nullptr, nullptr},
3776:   {(char*)"_saved_self_sym_sizes", (getter)THPUpsampleBicubic2DBackward0_self_sym_sizes_getter, nullptr, nullptr, nullptr},
3777:   {nullptr} /* sentinel */
3778: };
3779: 
3780: static PyObject* THPUpsampleNearest2DBackward0_output_size_getter(THPCppFunction *self, void *_unused) {
3781:   HANDLE_TH_ERRORS
3782:   auto prop = static_cast<UpsampleNearest2DBackward0*>(self->cdata.get())->output_size;
3783:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3784:   for (auto i : c10::irange(prop.size())) {
3785:       auto si = prop[i];
3786:       if (auto m = si.maybe_as_int()) {
3787:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3788:       } else {
3789:         auto py_symint = py::cast(si).release().ptr();
3790:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3791:       }
3792:   }
3793:   return tup;
3794:   END_HANDLE_TH_ERRORS
3795: }
3796: 
3797: static PyObject* THPUpsampleNearest2DBackward0_scales_h_getter(THPCppFunction *self, void *_unused) {
3798:   HANDLE_TH_ERRORS
3799:   auto opt_prop = static_cast<UpsampleNearest2DBackward0*>(self->cdata.get())->scales_h;
3800:   if (!opt_prop.has_value()) {
3801:     Py_RETURN_NONE;
3802:   }
3803:   auto prop = opt_prop.value();
3804:   return PyFloat_FromDouble((double) prop);
3805:   END_HANDLE_TH_ERRORS
3806: }
3807: 
3808: static PyObject* THPUpsampleNearest2DBackward0_scales_w_getter(THPCppFunction *self, void *_unused) {
3809:   HANDLE_TH_ERRORS
3810:   auto opt_prop = static_cast<UpsampleNearest2DBackward0*>(self->cdata.get())->scales_w;
3811:   if (!opt_prop.has_value()) {
3812:     Py_RETURN_NONE;
3813:   }
3814:   auto prop = opt_prop.value();
3815:   return PyFloat_FromDouble((double) prop);
3816:   END_HANDLE_TH_ERRORS
3817: }
3818: 
3819: static PyObject* THPUpsampleNearest2DBackward0_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
3820:   HANDLE_TH_ERRORS
3821:   auto prop = static_cast<UpsampleNearest2DBackward0*>(self->cdata.get())->self_sym_sizes;
3822:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3823:   for (auto i : c10::irange(prop.size())) {
3824:       auto si = prop[i];
3825:       if (auto m = si.maybe_as_int()) {
3826:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3827:       } else {
3828:         auto py_symint = py::cast(si).release().ptr();
3829:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3830:       }
3831:   }
3832:   return tup;
3833:   END_HANDLE_TH_ERRORS
3834: }
3835: 
3836: static struct PyGetSetDef UpsampleNearest2DBackward0_properties[] = {
3837:   THP_FUNCTION_DEFAULT_PROPERTIES,
3838:   {(char*)"_saved_output_size", (getter)THPUpsampleNearest2DBackward0_output_size_getter, nullptr, nullptr, nullptr},
3839:   {(char*)"_saved_scales_h", (getter)THPUpsampleNearest2DBackward0_scales_h_getter, nullptr, nullptr, nullptr},
3840:   {(char*)"_saved_scales_w", (getter)THPUpsampleNearest2DBackward0_scales_w_getter, nullptr, nullptr, nullptr},
```

- EN: The main execution path in this span is carried by `cast`, `PyTuple_SetItem`, `THPUpsampleNearest2DBackward0_output_size_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `cast`, `PyTuple_SetItem`, `THPUpsampleNearest2DBackward0_output_size_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3841-3920

```cpp
3841:   {(char*)"_saved_self_sym_sizes", (getter)THPUpsampleNearest2DBackward0_self_sym_sizes_getter, nullptr, nullptr, nullptr},
3842:   {nullptr} /* sentinel */
3843: };
3844: 
3845: static PyObject* THPPixelShuffleBackward0_upscale_factor_getter(THPCppFunction *self, void *_unused) {
3846:   HANDLE_TH_ERRORS
3847:   auto prop = static_cast<PixelShuffleBackward0*>(self->cdata.get())->upscale_factor;
3848:   return PyLong_FromUnsignedLong((int64_t) prop);
3849:   END_HANDLE_TH_ERRORS
3850: }
3851: 
3852: static struct PyGetSetDef PixelShuffleBackward0_properties[] = {
3853:   THP_FUNCTION_DEFAULT_PROPERTIES,
3854:   {(char*)"_saved_upscale_factor", (getter)THPPixelShuffleBackward0_upscale_factor_getter, nullptr, nullptr, nullptr},
3855:   {nullptr} /* sentinel */
3856: };
3857: 
3858: static PyObject* THPAdaptiveAvgPool3DBackward0_self_getter(THPCppFunction *self, void *_unused) {
3859:   HANDLE_TH_ERRORS
3860:   const auto& prop = static_cast<AdaptiveAvgPool3DBackward0*>(self->cdata.get())->self_;
3861:   return THPVariable_Wrap(prop.unpack(self->cdata));
3862:   END_HANDLE_TH_ERRORS
3863: }
3864: 
3865: static PyObject* THPAdaptiveAvgPool3DBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
3866:   HANDLE_TH_ERRORS
3867:   const auto& prop = static_cast<AdaptiveAvgPool3DBackward0*>(self->cdata.get())->self_;
3868:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3869:   return obj.release().ptr();
3870:   END_HANDLE_TH_ERRORS
3871: }
3872: 
3873: static struct PyGetSetDef AdaptiveAvgPool3DBackward0_properties[] = {
3874:   THP_FUNCTION_DEFAULT_PROPERTIES,
3875:   {(char*)"_saved_self", (getter)THPAdaptiveAvgPool3DBackward0_self_getter, nullptr, nullptr, nullptr},
3876:   {(char*)"_raw_saved_self", (getter)THPAdaptiveAvgPool3DBackward0_self_raw_getter, nullptr, nullptr, nullptr},
3877:   {nullptr} /* sentinel */
3878: };
3879: 
3880: static PyObject* THPAdaptiveMaxPool2DBackward0_self_getter(THPCppFunction *self, void *_unused) {
3881:   HANDLE_TH_ERRORS
3882:   const auto& prop = static_cast<AdaptiveMaxPool2DBackward0*>(self->cdata.get())->self_;
3883:   return THPVariable_Wrap(prop.unpack(self->cdata));
3884:   END_HANDLE_TH_ERRORS
3885: }
3886: 
3887: static PyObject* THPAdaptiveMaxPool2DBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
3888:   HANDLE_TH_ERRORS
3889:   const auto& prop = static_cast<AdaptiveMaxPool2DBackward0*>(self->cdata.get())->self_;
3890:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3891:   return obj.release().ptr();
3892:   END_HANDLE_TH_ERRORS
3893: }
3894: 
3895: static PyObject* THPAdaptiveMaxPool2DBackward0_result1_getter(THPCppFunction *self, void *_unused) {
3896:   HANDLE_TH_ERRORS
3897:   const auto& prop = static_cast<AdaptiveMaxPool2DBackward0*>(self->cdata.get())->result1_;
3898:   return THPVariable_Wrap(prop.unpack(self->cdata));
3899:   END_HANDLE_TH_ERRORS
3900: }
3901: 
3902: static PyObject* THPAdaptiveMaxPool2DBackward0_result1_raw_getter(THPCppFunction *self, void *_unused) {
3903:   HANDLE_TH_ERRORS
3904:   const auto& prop = static_cast<AdaptiveMaxPool2DBackward0*>(self->cdata.get())->result1_;
3905:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3906:   return obj.release().ptr();
3907:   END_HANDLE_TH_ERRORS
3908: }
3909: 
3910: static struct PyGetSetDef AdaptiveMaxPool2DBackward0_properties[] = {
3911:   THP_FUNCTION_DEFAULT_PROPERTIES,
3912:   {(char*)"_saved_self", (getter)THPAdaptiveMaxPool2DBackward0_self_getter, nullptr, nullptr, nullptr},
3913:   {(char*)"_raw_saved_self", (getter)THPAdaptiveMaxPool2DBackward0_self_raw_getter, nullptr, nullptr, nullptr},
3914:   {(char*)"_saved_result1", (getter)THPAdaptiveMaxPool2DBackward0_result1_getter, nullptr, nullptr, nullptr},
3915:   {(char*)"_raw_saved_result1", (getter)THPAdaptiveMaxPool2DBackward0_result1_raw_getter, nullptr, nullptr, nullptr},
3916:   {nullptr} /* sentinel */
3917: };
3918: 
3919: static PyObject* THPAdaptiveMaxPool3DBackward0_self_getter(THPCppFunction *self, void *_unused) {
3920:   HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPPixelShuffleBackward0_upscale_factor_getter`, `PyLong_FromUnsignedLong`, `THPAdaptiveAvgPool3DBackward0_self_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPPixelShuffleBackward0_upscale_factor_getter`, `PyLong_FromUnsignedLong`, `THPAdaptiveAvgPool3DBackward0_self_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3921-4000

```cpp
3921:   const auto& prop = static_cast<AdaptiveMaxPool3DBackward0*>(self->cdata.get())->self_;
3922:   return THPVariable_Wrap(prop.unpack(self->cdata));
3923:   END_HANDLE_TH_ERRORS
3924: }
3925: 
3926: static PyObject* THPAdaptiveMaxPool3DBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
3927:   HANDLE_TH_ERRORS
3928:   const auto& prop = static_cast<AdaptiveMaxPool3DBackward0*>(self->cdata.get())->self_;
3929:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3930:   return obj.release().ptr();
3931:   END_HANDLE_TH_ERRORS
3932: }
3933: 
3934: static PyObject* THPAdaptiveMaxPool3DBackward0_result1_getter(THPCppFunction *self, void *_unused) {
3935:   HANDLE_TH_ERRORS
3936:   const auto& prop = static_cast<AdaptiveMaxPool3DBackward0*>(self->cdata.get())->result1_;
3937:   return THPVariable_Wrap(prop.unpack(self->cdata));
3938:   END_HANDLE_TH_ERRORS
3939: }
3940: 
3941: static PyObject* THPAdaptiveMaxPool3DBackward0_result1_raw_getter(THPCppFunction *self, void *_unused) {
3942:   HANDLE_TH_ERRORS
3943:   const auto& prop = static_cast<AdaptiveMaxPool3DBackward0*>(self->cdata.get())->result1_;
3944:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3945:   return obj.release().ptr();
3946:   END_HANDLE_TH_ERRORS
3947: }
3948: 
3949: static struct PyGetSetDef AdaptiveMaxPool3DBackward0_properties[] = {
3950:   THP_FUNCTION_DEFAULT_PROPERTIES,
3951:   {(char*)"_saved_self", (getter)THPAdaptiveMaxPool3DBackward0_self_getter, nullptr, nullptr, nullptr},
3952:   {(char*)"_raw_saved_self", (getter)THPAdaptiveMaxPool3DBackward0_self_raw_getter, nullptr, nullptr, nullptr},
3953:   {(char*)"_saved_result1", (getter)THPAdaptiveMaxPool3DBackward0_result1_getter, nullptr, nullptr, nullptr},
3954:   {(char*)"_raw_saved_result1", (getter)THPAdaptiveMaxPool3DBackward0_result1_raw_getter, nullptr, nullptr, nullptr},
3955:   {nullptr} /* sentinel */
3956: };
3957: 
3958: static PyObject* THPFractionalMaxPool2DBackward0_kernel_size_getter(THPCppFunction *self, void *_unused) {
3959:   HANDLE_TH_ERRORS
3960:   auto prop = static_cast<FractionalMaxPool2DBackward0*>(self->cdata.get())->kernel_size;
3961:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3962:   for (auto i : c10::irange(prop.size())) {
3963:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
3964:   }
3965:   return tup;
3966:   END_HANDLE_TH_ERRORS
3967: }
3968: 
3969: static PyObject* THPFractionalMaxPool2DBackward0_output_size_getter(THPCppFunction *self, void *_unused) {
3970:   HANDLE_TH_ERRORS
3971:   auto prop = static_cast<FractionalMaxPool2DBackward0*>(self->cdata.get())->output_size;
3972:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3973:   for (auto i : c10::irange(prop.size())) {
3974:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
3975:   }
3976:   return tup;
3977:   END_HANDLE_TH_ERRORS
3978: }
3979: 
3980: static PyObject* THPFractionalMaxPool2DBackward0_self_getter(THPCppFunction *self, void *_unused) {
3981:   HANDLE_TH_ERRORS
3982:   const auto& prop = static_cast<FractionalMaxPool2DBackward0*>(self->cdata.get())->self_;
3983:   return THPVariable_Wrap(prop.unpack(self->cdata));
3984:   END_HANDLE_TH_ERRORS
3985: }
3986: 
3987: static PyObject* THPFractionalMaxPool2DBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
3988:   HANDLE_TH_ERRORS
3989:   const auto& prop = static_cast<FractionalMaxPool2DBackward0*>(self->cdata.get())->self_;
3990:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3991:   return obj.release().ptr();
3992:   END_HANDLE_TH_ERRORS
3993: }
3994: 
3995: static PyObject* THPFractionalMaxPool2DBackward0_result1_getter(THPCppFunction *self, void *_unused) {
3996:   HANDLE_TH_ERRORS
3997:   const auto& prop = static_cast<FractionalMaxPool2DBackward0*>(self->cdata.get())->result1_;
3998:   return THPVariable_Wrap(prop.unpack(self->cdata));
3999:   END_HANDLE_TH_ERRORS
4000: }
```

- EN: The main execution path in this span is carried by `THPVariable_Wrap`, `THPAdaptiveMaxPool3DBackward0_self_raw_getter`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_Wrap`, `THPAdaptiveMaxPool3DBackward0_self_raw_getter`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 4001-4080

```cpp
4001: 
4002: static PyObject* THPFractionalMaxPool2DBackward0_result1_raw_getter(THPCppFunction *self, void *_unused) {
4003:   HANDLE_TH_ERRORS
4004:   const auto& prop = static_cast<FractionalMaxPool2DBackward0*>(self->cdata.get())->result1_;
4005:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4006:   return obj.release().ptr();
4007:   END_HANDLE_TH_ERRORS
4008: }
4009: 
4010: static struct PyGetSetDef FractionalMaxPool2DBackward0_properties[] = {
4011:   THP_FUNCTION_DEFAULT_PROPERTIES,
4012:   {(char*)"_saved_kernel_size", (getter)THPFractionalMaxPool2DBackward0_kernel_size_getter, nullptr, nullptr, nullptr},
4013:   {(char*)"_saved_output_size", (getter)THPFractionalMaxPool2DBackward0_output_size_getter, nullptr, nullptr, nullptr},
4014:   {(char*)"_saved_self", (getter)THPFractionalMaxPool2DBackward0_self_getter, nullptr, nullptr, nullptr},
4015:   {(char*)"_raw_saved_self", (getter)THPFractionalMaxPool2DBackward0_self_raw_getter, nullptr, nullptr, nullptr},
4016:   {(char*)"_saved_result1", (getter)THPFractionalMaxPool2DBackward0_result1_getter, nullptr, nullptr, nullptr},
4017:   {(char*)"_raw_saved_result1", (getter)THPFractionalMaxPool2DBackward0_result1_raw_getter, nullptr, nullptr, nullptr},
4018:   {nullptr} /* sentinel */
4019: };
4020: 
4021: static PyObject* THPLinearBackwardBackward0_grad_output_getter(THPCppFunction *self, void *_unused) {
4022:   HANDLE_TH_ERRORS
4023:   const auto& prop = static_cast<LinearBackwardBackward0*>(self->cdata.get())->grad_output_;
4024:   return THPVariable_Wrap(prop.unpack(self->cdata));
4025:   END_HANDLE_TH_ERRORS
4026: }
4027: 
4028: static PyObject* THPLinearBackwardBackward0_grad_output_raw_getter(THPCppFunction *self, void *_unused) {
4029:   HANDLE_TH_ERRORS
4030:   const auto& prop = static_cast<LinearBackwardBackward0*>(self->cdata.get())->grad_output_;
4031:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4032:   return obj.release().ptr();
4033:   END_HANDLE_TH_ERRORS
4034: }
4035: 
4036: static PyObject* THPLinearBackwardBackward0_self_getter(THPCppFunction *self, void *_unused) {
4037:   HANDLE_TH_ERRORS
4038:   const auto& prop = static_cast<LinearBackwardBackward0*>(self->cdata.get())->self_;
4039:   return THPVariable_Wrap(prop.unpack(self->cdata));
4040:   END_HANDLE_TH_ERRORS
4041: }
4042: 
4043: static PyObject* THPLinearBackwardBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
4044:   HANDLE_TH_ERRORS
4045:   const auto& prop = static_cast<LinearBackwardBackward0*>(self->cdata.get())->self_;
4046:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4047:   return obj.release().ptr();
4048:   END_HANDLE_TH_ERRORS
4049: }
4050: 
4051: static PyObject* THPLinearBackwardBackward0_weight_getter(THPCppFunction *self, void *_unused) {
4052:   HANDLE_TH_ERRORS
4053:   const auto& prop = static_cast<LinearBackwardBackward0*>(self->cdata.get())->weight_;
4054:   return THPVariable_Wrap(prop.unpack(self->cdata));
4055:   END_HANDLE_TH_ERRORS
4056: }
4057: 
4058: static PyObject* THPLinearBackwardBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
4059:   HANDLE_TH_ERRORS
4060:   const auto& prop = static_cast<LinearBackwardBackward0*>(self->cdata.get())->weight_;
4061:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4062:   return obj.release().ptr();
4063:   END_HANDLE_TH_ERRORS
4064: }
4065: 
4066: static struct PyGetSetDef LinearBackwardBackward0_properties[] = {
4067:   THP_FUNCTION_DEFAULT_PROPERTIES,
4068:   {(char*)"_saved_grad_output", (getter)THPLinearBackwardBackward0_grad_output_getter, nullptr, nullptr, nullptr},
4069:   {(char*)"_raw_saved_grad_output", (getter)THPLinearBackwardBackward0_grad_output_raw_getter, nullptr, nullptr, nullptr},
4070:   {(char*)"_saved_self", (getter)THPLinearBackwardBackward0_self_getter, nullptr, nullptr, nullptr},
4071:   {(char*)"_raw_saved_self", (getter)THPLinearBackwardBackward0_self_raw_getter, nullptr, nullptr, nullptr},
4072:   {(char*)"_saved_weight", (getter)THPLinearBackwardBackward0_weight_getter, nullptr, nullptr, nullptr},
4073:   {(char*)"_raw_saved_weight", (getter)THPLinearBackwardBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
4074:   {nullptr} /* sentinel */
4075: };
4076: 
4077: static PyObject* THPMaxPool2DBackward0_ceil_mode_getter(THPCppFunction *self, void *_unused) {
4078:   HANDLE_TH_ERRORS
4079:   auto prop = static_cast<MaxPool2DBackward0*>(self->cdata.get())->ceil_mode;
4080:   if (prop) {
```

- EN: The main execution path in this span is carried by `THPFractionalMaxPool2DBackward0_result1_raw_getter`, `cast`, `THPLinearBackwardBackward0_grad_output_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPFractionalMaxPool2DBackward0_result1_raw_getter`, `cast`, `THPLinearBackwardBackward0_grad_output_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 4081-4160

```cpp
4081:     Py_RETURN_TRUE;
4082:   } else {
4083:     Py_RETURN_FALSE;
4084:   }
4085:   END_HANDLE_TH_ERRORS
4086: }
4087: 
4088: static PyObject* THPMaxPool2DBackward0_dilation_getter(THPCppFunction *self, void *_unused) {
4089:   HANDLE_TH_ERRORS
4090:   auto prop = static_cast<MaxPool2DBackward0*>(self->cdata.get())->dilation;
4091:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4092:   for (auto i : c10::irange(prop.size())) {
4093:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
4094:   }
4095:   return tup;
4096:   END_HANDLE_TH_ERRORS
4097: }
4098: 
4099: static PyObject* THPMaxPool2DBackward0_kernel_size_getter(THPCppFunction *self, void *_unused) {
4100:   HANDLE_TH_ERRORS
4101:   auto prop = static_cast<MaxPool2DBackward0*>(self->cdata.get())->kernel_size;
4102:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4103:   for (auto i : c10::irange(prop.size())) {
4104:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
4105:   }
4106:   return tup;
4107:   END_HANDLE_TH_ERRORS
4108: }
4109: 
4110: static PyObject* THPMaxPool2DBackward0_padding_getter(THPCppFunction *self, void *_unused) {
4111:   HANDLE_TH_ERRORS
4112:   auto prop = static_cast<MaxPool2DBackward0*>(self->cdata.get())->padding;
4113:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4114:   for (auto i : c10::irange(prop.size())) {
4115:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
4116:   }
4117:   return tup;
4118:   END_HANDLE_TH_ERRORS
4119: }
4120: 
4121: static PyObject* THPMaxPool2DBackward0_self_getter(THPCppFunction *self, void *_unused) {
4122:   HANDLE_TH_ERRORS
4123:   const auto& prop = static_cast<MaxPool2DBackward0*>(self->cdata.get())->self_;
4124:   return THPVariable_Wrap(prop.unpack(self->cdata));
4125:   END_HANDLE_TH_ERRORS
4126: }
4127: 
4128: static PyObject* THPMaxPool2DBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
4129:   HANDLE_TH_ERRORS
4130:   const auto& prop = static_cast<MaxPool2DBackward0*>(self->cdata.get())->self_;
4131:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4132:   return obj.release().ptr();
4133:   END_HANDLE_TH_ERRORS
4134: }
4135: 
4136: static PyObject* THPMaxPool2DBackward0_stride_getter(THPCppFunction *self, void *_unused) {
4137:   HANDLE_TH_ERRORS
4138:   auto prop = static_cast<MaxPool2DBackward0*>(self->cdata.get())->stride;
4139:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4140:   for (auto i : c10::irange(prop.size())) {
4141:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
4142:   }
4143:   return tup;
4144:   END_HANDLE_TH_ERRORS
4145: }
4146: 
4147: static struct PyGetSetDef MaxPool2DBackward0_properties[] = {
4148:   THP_FUNCTION_DEFAULT_PROPERTIES,
4149:   {(char*)"_saved_ceil_mode", (getter)THPMaxPool2DBackward0_ceil_mode_getter, nullptr, nullptr, nullptr},
4150:   {(char*)"_saved_dilation", (getter)THPMaxPool2DBackward0_dilation_getter, nullptr, nullptr, nullptr},
4151:   {(char*)"_saved_kernel_size", (getter)THPMaxPool2DBackward0_kernel_size_getter, nullptr, nullptr, nullptr},
4152:   {(char*)"_saved_padding", (getter)THPMaxPool2DBackward0_padding_getter, nullptr, nullptr, nullptr},
4153:   {(char*)"_saved_self", (getter)THPMaxPool2DBackward0_self_getter, nullptr, nullptr, nullptr},
4154:   {(char*)"_raw_saved_self", (getter)THPMaxPool2DBackward0_self_raw_getter, nullptr, nullptr, nullptr},
4155:   {(char*)"_saved_stride", (getter)THPMaxPool2DBackward0_stride_getter, nullptr, nullptr, nullptr},
4156:   {nullptr} /* sentinel */
4157: };
4158: 
4159: static PyObject* THPMaxUnpool2DBackward0_indices_getter(THPCppFunction *self, void *_unused) {
4160:   HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPMaxPool2DBackward0_dilation_getter`, `PyTuple_New`, `PyTuple_SetItem`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPMaxPool2DBackward0_dilation_getter`, `PyTuple_New`, `PyTuple_SetItem` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 4161-4240

```cpp
4161:   const auto& prop = static_cast<MaxUnpool2DBackward0*>(self->cdata.get())->indices_;
4162:   return THPVariable_Wrap(prop.unpack(self->cdata));
4163:   END_HANDLE_TH_ERRORS
4164: }
4165: 
4166: static PyObject* THPMaxUnpool2DBackward0_indices_raw_getter(THPCppFunction *self, void *_unused) {
4167:   HANDLE_TH_ERRORS
4168:   const auto& prop = static_cast<MaxUnpool2DBackward0*>(self->cdata.get())->indices_;
4169:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4170:   return obj.release().ptr();
4171:   END_HANDLE_TH_ERRORS
4172: }
4173: 
4174: static struct PyGetSetDef MaxUnpool2DBackward0_properties[] = {
4175:   THP_FUNCTION_DEFAULT_PROPERTIES,
4176:   {(char*)"_saved_indices", (getter)THPMaxUnpool2DBackward0_indices_getter, nullptr, nullptr, nullptr},
4177:   {(char*)"_raw_saved_indices", (getter)THPMaxUnpool2DBackward0_indices_raw_getter, nullptr, nullptr, nullptr},
4178:   {nullptr} /* sentinel */
4179: };
4180: 
4181: static PyObject* THPMaxUnpool3DBackward0_indices_getter(THPCppFunction *self, void *_unused) {
4182:   HANDLE_TH_ERRORS
4183:   const auto& prop = static_cast<MaxUnpool3DBackward0*>(self->cdata.get())->indices_;
4184:   return THPVariable_Wrap(prop.unpack(self->cdata));
4185:   END_HANDLE_TH_ERRORS
4186: }
4187: 
4188: static PyObject* THPMaxUnpool3DBackward0_indices_raw_getter(THPCppFunction *self, void *_unused) {
4189:   HANDLE_TH_ERRORS
4190:   const auto& prop = static_cast<MaxUnpool3DBackward0*>(self->cdata.get())->indices_;
4191:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4192:   return obj.release().ptr();
4193:   END_HANDLE_TH_ERRORS
4194: }
4195: 
4196: static struct PyGetSetDef MaxUnpool3DBackward0_properties[] = {
4197:   THP_FUNCTION_DEFAULT_PROPERTIES,
4198:   {(char*)"_saved_indices", (getter)THPMaxUnpool3DBackward0_indices_getter, nullptr, nullptr, nullptr},
4199:   {(char*)"_raw_saved_indices", (getter)THPMaxUnpool3DBackward0_indices_raw_getter, nullptr, nullptr, nullptr},
4200:   {nullptr} /* sentinel */
4201: };
4202: 
4203: static PyObject* THPSlowConvTranspose2DBackward0_bias_sym_sizes_opt_getter(THPCppFunction *self, void *_unused) {
4204:   HANDLE_TH_ERRORS
4205:   auto opt_prop = static_cast<SlowConvTranspose2DBackward0*>(self->cdata.get())->bias_sym_sizes_opt;
4206:   if (!opt_prop.list.has_value()) {
4207:     Py_RETURN_NONE;
4208:   }
4209:   auto prop = opt_prop.list.value();
4210:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4211:   for (auto i : c10::irange(prop.size())) {
4212:       auto si = prop[i];
4213:       if (auto m = si.maybe_as_int()) {
4214:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4215:       } else {
4216:         auto py_symint = py::cast(si).release().ptr();
4217:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4218:       }
4219:   }
4220:   return tup;
4221:   END_HANDLE_TH_ERRORS
4222: }
4223: 
4224: static PyObject* THPSlowConvTranspose2DBackward0_dilation_getter(THPCppFunction *self, void *_unused) {
4225:   HANDLE_TH_ERRORS
4226:   auto prop = static_cast<SlowConvTranspose2DBackward0*>(self->cdata.get())->dilation;
4227:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4228:   for (auto i : c10::irange(prop.size())) {
4229:       auto si = prop[i];
4230:       if (auto m = si.maybe_as_int()) {
4231:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4232:       } else {
4233:         auto py_symint = py::cast(si).release().ptr();
4234:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4235:       }
4236:   }
4237:   return tup;
4238:   END_HANDLE_TH_ERRORS
4239: }
4240: 
```

- EN: The main execution path in this span is carried by `THPVariable_Wrap`, `THPMaxUnpool2DBackward0_indices_raw_getter`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_Wrap`, `THPMaxUnpool2DBackward0_indices_raw_getter`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4241-4320

```cpp
4241: static PyObject* THPSlowConvTranspose2DBackward0_output_padding_getter(THPCppFunction *self, void *_unused) {
4242:   HANDLE_TH_ERRORS
4243:   auto prop = static_cast<SlowConvTranspose2DBackward0*>(self->cdata.get())->output_padding;
4244:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4245:   for (auto i : c10::irange(prop.size())) {
4246:       auto si = prop[i];
4247:       if (auto m = si.maybe_as_int()) {
4248:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4249:       } else {
4250:         auto py_symint = py::cast(si).release().ptr();
4251:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4252:       }
4253:   }
4254:   return tup;
4255:   END_HANDLE_TH_ERRORS
4256: }
4257: 
4258: static PyObject* THPSlowConvTranspose2DBackward0_padding_getter(THPCppFunction *self, void *_unused) {
4259:   HANDLE_TH_ERRORS
4260:   auto prop = static_cast<SlowConvTranspose2DBackward0*>(self->cdata.get())->padding;
4261:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4262:   for (auto i : c10::irange(prop.size())) {
4263:       auto si = prop[i];
4264:       if (auto m = si.maybe_as_int()) {
4265:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4266:       } else {
4267:         auto py_symint = py::cast(si).release().ptr();
4268:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4269:       }
4270:   }
4271:   return tup;
4272:   END_HANDLE_TH_ERRORS
4273: }
4274: 
4275: static PyObject* THPSlowConvTranspose2DBackward0_self_getter(THPCppFunction *self, void *_unused) {
4276:   HANDLE_TH_ERRORS
4277:   const auto& prop = static_cast<SlowConvTranspose2DBackward0*>(self->cdata.get())->self_;
4278:   return THPVariable_Wrap(prop.unpack(self->cdata));
4279:   END_HANDLE_TH_ERRORS
4280: }
4281: 
4282: static PyObject* THPSlowConvTranspose2DBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
4283:   HANDLE_TH_ERRORS
4284:   const auto& prop = static_cast<SlowConvTranspose2DBackward0*>(self->cdata.get())->self_;
4285:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4286:   return obj.release().ptr();
4287:   END_HANDLE_TH_ERRORS
4288: }
4289: 
4290: static PyObject* THPSlowConvTranspose2DBackward0_stride_getter(THPCppFunction *self, void *_unused) {
4291:   HANDLE_TH_ERRORS
4292:   auto prop = static_cast<SlowConvTranspose2DBackward0*>(self->cdata.get())->stride;
4293:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4294:   for (auto i : c10::irange(prop.size())) {
4295:       auto si = prop[i];
4296:       if (auto m = si.maybe_as_int()) {
4297:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4298:       } else {
4299:         auto py_symint = py::cast(si).release().ptr();
4300:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4301:       }
4302:   }
4303:   return tup;
4304:   END_HANDLE_TH_ERRORS
4305: }
4306: 
4307: static PyObject* THPSlowConvTranspose2DBackward0_weight_getter(THPCppFunction *self, void *_unused) {
4308:   HANDLE_TH_ERRORS
4309:   const auto& prop = static_cast<SlowConvTranspose2DBackward0*>(self->cdata.get())->weight_;
4310:   return THPVariable_Wrap(prop.unpack(self->cdata));
4311:   END_HANDLE_TH_ERRORS
4312: }
4313: 
4314: static PyObject* THPSlowConvTranspose2DBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
4315:   HANDLE_TH_ERRORS
4316:   const auto& prop = static_cast<SlowConvTranspose2DBackward0*>(self->cdata.get())->weight_;
4317:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4318:   return obj.release().ptr();
4319:   END_HANDLE_TH_ERRORS
4320: }
```

- EN: The main execution path in this span is carried by `THPSlowConvTranspose2DBackward0_output_padding_getter`, `PyTuple_New`, `PyTuple_SetItem`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPSlowConvTranspose2DBackward0_output_padding_getter`, `PyTuple_New`, `PyTuple_SetItem` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4321-4400

```cpp
4321: 
4322: static struct PyGetSetDef SlowConvTranspose2DBackward0_properties[] = {
4323:   THP_FUNCTION_DEFAULT_PROPERTIES,
4324:   {(char*)"_saved_bias_sym_sizes_opt", (getter)THPSlowConvTranspose2DBackward0_bias_sym_sizes_opt_getter, nullptr, nullptr, nullptr},
4325:   {(char*)"_saved_dilation", (getter)THPSlowConvTranspose2DBackward0_dilation_getter, nullptr, nullptr, nullptr},
4326:   {(char*)"_saved_output_padding", (getter)THPSlowConvTranspose2DBackward0_output_padding_getter, nullptr, nullptr, nullptr},
4327:   {(char*)"_saved_padding", (getter)THPSlowConvTranspose2DBackward0_padding_getter, nullptr, nullptr, nullptr},
4328:   {(char*)"_saved_self", (getter)THPSlowConvTranspose2DBackward0_self_getter, nullptr, nullptr, nullptr},
4329:   {(char*)"_raw_saved_self", (getter)THPSlowConvTranspose2DBackward0_self_raw_getter, nullptr, nullptr, nullptr},
4330:   {(char*)"_saved_stride", (getter)THPSlowConvTranspose2DBackward0_stride_getter, nullptr, nullptr, nullptr},
4331:   {(char*)"_saved_weight", (getter)THPSlowConvTranspose2DBackward0_weight_getter, nullptr, nullptr, nullptr},
4332:   {(char*)"_raw_saved_weight", (getter)THPSlowConvTranspose2DBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
4333:   {nullptr} /* sentinel */
4334: };
4335: 
4336: static PyObject* THPSlowConv2DBackwardBackward0_grad_output_getter(THPCppFunction *self, void *_unused) {
4337:   HANDLE_TH_ERRORS
4338:   const auto& prop = static_cast<SlowConv2DBackwardBackward0*>(self->cdata.get())->grad_output_;
4339:   return THPVariable_Wrap(prop.unpack(self->cdata));
4340:   END_HANDLE_TH_ERRORS
4341: }
4342: 
4343: static PyObject* THPSlowConv2DBackwardBackward0_grad_output_raw_getter(THPCppFunction *self, void *_unused) {
4344:   HANDLE_TH_ERRORS
4345:   const auto& prop = static_cast<SlowConv2DBackwardBackward0*>(self->cdata.get())->grad_output_;
4346:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4347:   return obj.release().ptr();
4348:   END_HANDLE_TH_ERRORS
4349: }
4350: 
4351: static PyObject* THPSlowConv2DBackwardBackward0_padding_getter(THPCppFunction *self, void *_unused) {
4352:   HANDLE_TH_ERRORS
4353:   auto prop = static_cast<SlowConv2DBackwardBackward0*>(self->cdata.get())->padding;
4354:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4355:   for (auto i : c10::irange(prop.size())) {
4356:       auto si = prop[i];
4357:       if (auto m = si.maybe_as_int()) {
4358:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4359:       } else {
4360:         auto py_symint = py::cast(si).release().ptr();
4361:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4362:       }
4363:   }
4364:   return tup;
4365:   END_HANDLE_TH_ERRORS
4366: }
4367: 
4368: static PyObject* THPSlowConv2DBackwardBackward0_self_getter(THPCppFunction *self, void *_unused) {
4369:   HANDLE_TH_ERRORS
4370:   const auto& prop = static_cast<SlowConv2DBackwardBackward0*>(self->cdata.get())->self_;
4371:   return THPVariable_Wrap(prop.unpack(self->cdata));
4372:   END_HANDLE_TH_ERRORS
4373: }
4374: 
4375: static PyObject* THPSlowConv2DBackwardBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
4376:   HANDLE_TH_ERRORS
4377:   const auto& prop = static_cast<SlowConv2DBackwardBackward0*>(self->cdata.get())->self_;
4378:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4379:   return obj.release().ptr();
4380:   END_HANDLE_TH_ERRORS
4381: }
4382: 
4383: static PyObject* THPSlowConv2DBackwardBackward0_stride_getter(THPCppFunction *self, void *_unused) {
4384:   HANDLE_TH_ERRORS
4385:   auto prop = static_cast<SlowConv2DBackwardBackward0*>(self->cdata.get())->stride;
4386:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4387:   for (auto i : c10::irange(prop.size())) {
4388:       auto si = prop[i];
4389:       if (auto m = si.maybe_as_int()) {
4390:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4391:       } else {
4392:         auto py_symint = py::cast(si).release().ptr();
4393:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4394:       }
4395:   }
4396:   return tup;
4397:   END_HANDLE_TH_ERRORS
4398: }
4399: 
4400: static PyObject* THPSlowConv2DBackwardBackward0_weight_getter(THPCppFunction *self, void *_unused) {
```

- EN: The main execution path in this span is carried by `THPSlowConv2DBackwardBackward0_grad_output_getter`, `THPVariable_Wrap`, `THPSlowConv2DBackwardBackward0_grad_output_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPSlowConv2DBackwardBackward0_grad_output_getter`, `THPVariable_Wrap`, `THPSlowConv2DBackwardBackward0_grad_output_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4401-4480

```cpp
4401:   HANDLE_TH_ERRORS
4402:   const auto& prop = static_cast<SlowConv2DBackwardBackward0*>(self->cdata.get())->weight_;
4403:   return THPVariable_Wrap(prop.unpack(self->cdata));
4404:   END_HANDLE_TH_ERRORS
4405: }
4406: 
4407: static PyObject* THPSlowConv2DBackwardBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
4408:   HANDLE_TH_ERRORS
4409:   const auto& prop = static_cast<SlowConv2DBackwardBackward0*>(self->cdata.get())->weight_;
4410:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4411:   return obj.release().ptr();
4412:   END_HANDLE_TH_ERRORS
4413: }
4414: 
4415: static struct PyGetSetDef SlowConv2DBackwardBackward0_properties[] = {
4416:   THP_FUNCTION_DEFAULT_PROPERTIES,
4417:   {(char*)"_saved_grad_output", (getter)THPSlowConv2DBackwardBackward0_grad_output_getter, nullptr, nullptr, nullptr},
4418:   {(char*)"_raw_saved_grad_output", (getter)THPSlowConv2DBackwardBackward0_grad_output_raw_getter, nullptr, nullptr, nullptr},
4419:   {(char*)"_saved_padding", (getter)THPSlowConv2DBackwardBackward0_padding_getter, nullptr, nullptr, nullptr},
4420:   {(char*)"_saved_self", (getter)THPSlowConv2DBackwardBackward0_self_getter, nullptr, nullptr, nullptr},
4421:   {(char*)"_raw_saved_self", (getter)THPSlowConv2DBackwardBackward0_self_raw_getter, nullptr, nullptr, nullptr},
4422:   {(char*)"_saved_stride", (getter)THPSlowConv2DBackwardBackward0_stride_getter, nullptr, nullptr, nullptr},
4423:   {(char*)"_saved_weight", (getter)THPSlowConv2DBackwardBackward0_weight_getter, nullptr, nullptr, nullptr},
4424:   {(char*)"_raw_saved_weight", (getter)THPSlowConv2DBackwardBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
4425:   {nullptr} /* sentinel */
4426: };
4427: 
4428: static PyObject* THPConvDepthwise3DBackward0_bias_sym_sizes_opt_getter(THPCppFunction *self, void *_unused) {
4429:   HANDLE_TH_ERRORS
4430:   auto opt_prop = static_cast<ConvDepthwise3DBackward0*>(self->cdata.get())->bias_sym_sizes_opt;
4431:   if (!opt_prop.list.has_value()) {
4432:     Py_RETURN_NONE;
4433:   }
4434:   auto prop = opt_prop.list.value();
4435:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4436:   for (auto i : c10::irange(prop.size())) {
4437:       auto si = prop[i];
4438:       if (auto m = si.maybe_as_int()) {
4439:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4440:       } else {
4441:         auto py_symint = py::cast(si).release().ptr();
4442:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4443:       }
4444:   }
4445:   return tup;
4446:   END_HANDLE_TH_ERRORS
4447: }
4448: 
4449: static PyObject* THPConvDepthwise3DBackward0_dilation_getter(THPCppFunction *self, void *_unused) {
4450:   HANDLE_TH_ERRORS
4451:   auto prop = static_cast<ConvDepthwise3DBackward0*>(self->cdata.get())->dilation;
4452:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4453:   for (auto i : c10::irange(prop.size())) {
4454:       auto si = prop[i];
4455:       if (auto m = si.maybe_as_int()) {
4456:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4457:       } else {
4458:         auto py_symint = py::cast(si).release().ptr();
4459:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4460:       }
4461:   }
4462:   return tup;
4463:   END_HANDLE_TH_ERRORS
4464: }
4465: 
4466: static PyObject* THPConvDepthwise3DBackward0_padding_getter(THPCppFunction *self, void *_unused) {
4467:   HANDLE_TH_ERRORS
4468:   auto prop = static_cast<ConvDepthwise3DBackward0*>(self->cdata.get())->padding;
4469:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4470:   for (auto i : c10::irange(prop.size())) {
4471:       auto si = prop[i];
4472:       if (auto m = si.maybe_as_int()) {
4473:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4474:       } else {
4475:         auto py_symint = py::cast(si).release().ptr();
4476:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4477:       }
4478:   }
4479:   return tup;
4480:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPVariable_Wrap`, `THPSlowConv2DBackwardBackward0_weight_raw_getter`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_Wrap`, `THPSlowConv2DBackwardBackward0_weight_raw_getter`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4481-4560

```cpp
4481: }
4482: 
4483: static PyObject* THPConvDepthwise3DBackward0_self_getter(THPCppFunction *self, void *_unused) {
4484:   HANDLE_TH_ERRORS
4485:   const auto& prop = static_cast<ConvDepthwise3DBackward0*>(self->cdata.get())->self_;
4486:   return THPVariable_Wrap(prop.unpack(self->cdata));
4487:   END_HANDLE_TH_ERRORS
4488: }
4489: 
4490: static PyObject* THPConvDepthwise3DBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
4491:   HANDLE_TH_ERRORS
4492:   const auto& prop = static_cast<ConvDepthwise3DBackward0*>(self->cdata.get())->self_;
4493:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4494:   return obj.release().ptr();
4495:   END_HANDLE_TH_ERRORS
4496: }
4497: 
4498: static PyObject* THPConvDepthwise3DBackward0_stride_getter(THPCppFunction *self, void *_unused) {
4499:   HANDLE_TH_ERRORS
4500:   auto prop = static_cast<ConvDepthwise3DBackward0*>(self->cdata.get())->stride;
4501:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4502:   for (auto i : c10::irange(prop.size())) {
4503:       auto si = prop[i];
4504:       if (auto m = si.maybe_as_int()) {
4505:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4506:       } else {
4507:         auto py_symint = py::cast(si).release().ptr();
4508:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4509:       }
4510:   }
4511:   return tup;
4512:   END_HANDLE_TH_ERRORS
4513: }
4514: 
4515: static PyObject* THPConvDepthwise3DBackward0_weight_getter(THPCppFunction *self, void *_unused) {
4516:   HANDLE_TH_ERRORS
4517:   const auto& prop = static_cast<ConvDepthwise3DBackward0*>(self->cdata.get())->weight_;
4518:   return THPVariable_Wrap(prop.unpack(self->cdata));
4519:   END_HANDLE_TH_ERRORS
4520: }
4521: 
4522: static PyObject* THPConvDepthwise3DBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
4523:   HANDLE_TH_ERRORS
4524:   const auto& prop = static_cast<ConvDepthwise3DBackward0*>(self->cdata.get())->weight_;
4525:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4526:   return obj.release().ptr();
4527:   END_HANDLE_TH_ERRORS
4528: }
4529: 
4530: static struct PyGetSetDef ConvDepthwise3DBackward0_properties[] = {
4531:   THP_FUNCTION_DEFAULT_PROPERTIES,
4532:   {(char*)"_saved_bias_sym_sizes_opt", (getter)THPConvDepthwise3DBackward0_bias_sym_sizes_opt_getter, nullptr, nullptr, nullptr},
4533:   {(char*)"_saved_dilation", (getter)THPConvDepthwise3DBackward0_dilation_getter, nullptr, nullptr, nullptr},
4534:   {(char*)"_saved_padding", (getter)THPConvDepthwise3DBackward0_padding_getter, nullptr, nullptr, nullptr},
4535:   {(char*)"_saved_self", (getter)THPConvDepthwise3DBackward0_self_getter, nullptr, nullptr, nullptr},
4536:   {(char*)"_raw_saved_self", (getter)THPConvDepthwise3DBackward0_self_raw_getter, nullptr, nullptr, nullptr},
4537:   {(char*)"_saved_stride", (getter)THPConvDepthwise3DBackward0_stride_getter, nullptr, nullptr, nullptr},
4538:   {(char*)"_saved_weight", (getter)THPConvDepthwise3DBackward0_weight_getter, nullptr, nullptr, nullptr},
4539:   {(char*)"_raw_saved_weight", (getter)THPConvDepthwise3DBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
4540:   {nullptr} /* sentinel */
4541: };
4542: 
4543: static PyObject* THPSlowConvDilated2DBackward0_bias_sym_sizes_opt_getter(THPCppFunction *self, void *_unused) {
4544:   HANDLE_TH_ERRORS
4545:   auto opt_prop = static_cast<SlowConvDilated2DBackward0*>(self->cdata.get())->bias_sym_sizes_opt;
4546:   if (!opt_prop.list.has_value()) {
4547:     Py_RETURN_NONE;
4548:   }
4549:   auto prop = opt_prop.list.value();
4550:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4551:   for (auto i : c10::irange(prop.size())) {
4552:       auto si = prop[i];
4553:       if (auto m = si.maybe_as_int()) {
4554:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4555:       } else {
4556:         auto py_symint = py::cast(si).release().ptr();
4557:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4558:       }
4559:   }
4560:   return tup;
```

- EN: The main execution path in this span is carried by `THPConvDepthwise3DBackward0_self_getter`, `THPVariable_Wrap`, `THPConvDepthwise3DBackward0_self_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPConvDepthwise3DBackward0_self_getter`, `THPVariable_Wrap`, `THPConvDepthwise3DBackward0_self_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4561-4640

```cpp
4561:   END_HANDLE_TH_ERRORS
4562: }
4563: 
4564: static PyObject* THPSlowConvDilated2DBackward0_dilation_getter(THPCppFunction *self, void *_unused) {
4565:   HANDLE_TH_ERRORS
4566:   auto prop = static_cast<SlowConvDilated2DBackward0*>(self->cdata.get())->dilation;
4567:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4568:   for (auto i : c10::irange(prop.size())) {
4569:       auto si = prop[i];
4570:       if (auto m = si.maybe_as_int()) {
4571:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4572:       } else {
4573:         auto py_symint = py::cast(si).release().ptr();
4574:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4575:       }
4576:   }
4577:   return tup;
4578:   END_HANDLE_TH_ERRORS
4579: }
4580: 
4581: static PyObject* THPSlowConvDilated2DBackward0_padding_getter(THPCppFunction *self, void *_unused) {
4582:   HANDLE_TH_ERRORS
4583:   auto prop = static_cast<SlowConvDilated2DBackward0*>(self->cdata.get())->padding;
4584:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4585:   for (auto i : c10::irange(prop.size())) {
4586:       auto si = prop[i];
4587:       if (auto m = si.maybe_as_int()) {
4588:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4589:       } else {
4590:         auto py_symint = py::cast(si).release().ptr();
4591:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4592:       }
4593:   }
4594:   return tup;
4595:   END_HANDLE_TH_ERRORS
4596: }
4597: 
4598: static PyObject* THPSlowConvDilated2DBackward0_self_getter(THPCppFunction *self, void *_unused) {
4599:   HANDLE_TH_ERRORS
4600:   const auto& prop = static_cast<SlowConvDilated2DBackward0*>(self->cdata.get())->self_;
4601:   return THPVariable_Wrap(prop.unpack(self->cdata));
4602:   END_HANDLE_TH_ERRORS
4603: }
4604: 
4605: static PyObject* THPSlowConvDilated2DBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
4606:   HANDLE_TH_ERRORS
4607:   const auto& prop = static_cast<SlowConvDilated2DBackward0*>(self->cdata.get())->self_;
4608:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4609:   return obj.release().ptr();
4610:   END_HANDLE_TH_ERRORS
4611: }
4612: 
4613: static PyObject* THPSlowConvDilated2DBackward0_stride_getter(THPCppFunction *self, void *_unused) {
4614:   HANDLE_TH_ERRORS
4615:   auto prop = static_cast<SlowConvDilated2DBackward0*>(self->cdata.get())->stride;
4616:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4617:   for (auto i : c10::irange(prop.size())) {
4618:       auto si = prop[i];
4619:       if (auto m = si.maybe_as_int()) {
4620:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4621:       } else {
4622:         auto py_symint = py::cast(si).release().ptr();
4623:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4624:       }
4625:   }
4626:   return tup;
4627:   END_HANDLE_TH_ERRORS
4628: }
4629: 
4630: static PyObject* THPSlowConvDilated2DBackward0_weight_getter(THPCppFunction *self, void *_unused) {
4631:   HANDLE_TH_ERRORS
4632:   const auto& prop = static_cast<SlowConvDilated2DBackward0*>(self->cdata.get())->weight_;
4633:   return THPVariable_Wrap(prop.unpack(self->cdata));
4634:   END_HANDLE_TH_ERRORS
4635: }
4636: 
4637: static PyObject* THPSlowConvDilated2DBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
4638:   HANDLE_TH_ERRORS
4639:   const auto& prop = static_cast<SlowConvDilated2DBackward0*>(self->cdata.get())->weight_;
4640:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
```

- EN: The main execution path in this span is carried by `THPSlowConvDilated2DBackward0_dilation_getter`, `PyTuple_New`, `PyTuple_SetItem`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPSlowConvDilated2DBackward0_dilation_getter`, `PyTuple_New`, `PyTuple_SetItem` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4641-4720

```cpp
4641:   return obj.release().ptr();
4642:   END_HANDLE_TH_ERRORS
4643: }
4644: 
4645: static struct PyGetSetDef SlowConvDilated2DBackward0_properties[] = {
4646:   THP_FUNCTION_DEFAULT_PROPERTIES,
4647:   {(char*)"_saved_bias_sym_sizes_opt", (getter)THPSlowConvDilated2DBackward0_bias_sym_sizes_opt_getter, nullptr, nullptr, nullptr},
4648:   {(char*)"_saved_dilation", (getter)THPSlowConvDilated2DBackward0_dilation_getter, nullptr, nullptr, nullptr},
4649:   {(char*)"_saved_padding", (getter)THPSlowConvDilated2DBackward0_padding_getter, nullptr, nullptr, nullptr},
4650:   {(char*)"_saved_self", (getter)THPSlowConvDilated2DBackward0_self_getter, nullptr, nullptr, nullptr},
4651:   {(char*)"_raw_saved_self", (getter)THPSlowConvDilated2DBackward0_self_raw_getter, nullptr, nullptr, nullptr},
4652:   {(char*)"_saved_stride", (getter)THPSlowConvDilated2DBackward0_stride_getter, nullptr, nullptr, nullptr},
4653:   {(char*)"_saved_weight", (getter)THPSlowConvDilated2DBackward0_weight_getter, nullptr, nullptr, nullptr},
4654:   {(char*)"_raw_saved_weight", (getter)THPSlowConvDilated2DBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
4655:   {nullptr} /* sentinel */
4656: };
4657: 
4658: static PyObject* THPCol2ImBackward0_dilation_getter(THPCppFunction *self, void *_unused) {
4659:   HANDLE_TH_ERRORS
4660:   auto prop = static_cast<Col2ImBackward0*>(self->cdata.get())->dilation;
4661:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4662:   for (auto i : c10::irange(prop.size())) {
4663:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
4664:   }
4665:   return tup;
4666:   END_HANDLE_TH_ERRORS
4667: }
4668: 
4669: static PyObject* THPCol2ImBackward0_kernel_size_getter(THPCppFunction *self, void *_unused) {
4670:   HANDLE_TH_ERRORS
4671:   auto prop = static_cast<Col2ImBackward0*>(self->cdata.get())->kernel_size;
4672:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4673:   for (auto i : c10::irange(prop.size())) {
4674:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
4675:   }
4676:   return tup;
4677:   END_HANDLE_TH_ERRORS
4678: }
4679: 
4680: static PyObject* THPCol2ImBackward0_padding_getter(THPCppFunction *self, void *_unused) {
4681:   HANDLE_TH_ERRORS
4682:   auto prop = static_cast<Col2ImBackward0*>(self->cdata.get())->padding;
4683:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4684:   for (auto i : c10::irange(prop.size())) {
4685:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
4686:   }
4687:   return tup;
4688:   END_HANDLE_TH_ERRORS
4689: }
4690: 
4691: static PyObject* THPCol2ImBackward0_stride_getter(THPCppFunction *self, void *_unused) {
4692:   HANDLE_TH_ERRORS
4693:   auto prop = static_cast<Col2ImBackward0*>(self->cdata.get())->stride;
4694:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4695:   for (auto i : c10::irange(prop.size())) {
4696:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
4697:   }
4698:   return tup;
4699:   END_HANDLE_TH_ERRORS
4700: }
4701: 
4702: static struct PyGetSetDef Col2ImBackward0_properties[] = {
4703:   THP_FUNCTION_DEFAULT_PROPERTIES,
4704:   {(char*)"_saved_dilation", (getter)THPCol2ImBackward0_dilation_getter, nullptr, nullptr, nullptr},
4705:   {(char*)"_saved_kernel_size", (getter)THPCol2ImBackward0_kernel_size_getter, nullptr, nullptr, nullptr},
4706:   {(char*)"_saved_padding", (getter)THPCol2ImBackward0_padding_getter, nullptr, nullptr, nullptr},
4707:   {(char*)"_saved_stride", (getter)THPCol2ImBackward0_stride_getter, nullptr, nullptr, nullptr},
4708:   {nullptr} /* sentinel */
4709: };
4710: 
4711: static PyObject* THPAvgPool2DBackwardBackward0_ceil_mode_getter(THPCppFunction *self, void *_unused) {
4712:   HANDLE_TH_ERRORS
4713:   auto prop = static_cast<AvgPool2DBackwardBackward0*>(self->cdata.get())->ceil_mode;
4714:   if (prop) {
4715:     Py_RETURN_TRUE;
4716:   } else {
4717:     Py_RETURN_FALSE;
4718:   }
4719:   END_HANDLE_TH_ERRORS
4720: }
```

- EN: The main execution path in this span is carried by `THPCol2ImBackward0_dilation_getter`, `PyTuple_New`, `PyTuple_SetItem`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPCol2ImBackward0_dilation_getter`, `PyTuple_New`, `PyTuple_SetItem` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 4721-4800

```cpp
4721: 
4722: static PyObject* THPAvgPool2DBackwardBackward0_count_include_pad_getter(THPCppFunction *self, void *_unused) {
4723:   HANDLE_TH_ERRORS
4724:   auto prop = static_cast<AvgPool2DBackwardBackward0*>(self->cdata.get())->count_include_pad;
4725:   if (prop) {
4726:     Py_RETURN_TRUE;
4727:   } else {
4728:     Py_RETURN_FALSE;
4729:   }
4730:   END_HANDLE_TH_ERRORS
4731: }
4732: 
4733: static PyObject* THPAvgPool2DBackwardBackward0_divisor_override_getter(THPCppFunction *self, void *_unused) {
4734:   HANDLE_TH_ERRORS
4735:   auto opt_prop = static_cast<AvgPool2DBackwardBackward0*>(self->cdata.get())->divisor_override;
4736:   if (!opt_prop.has_value()) {
4737:     Py_RETURN_NONE;
4738:   }
4739:   auto prop = opt_prop.value();
4740:   return PyLong_FromUnsignedLong((int64_t) prop);
4741:   END_HANDLE_TH_ERRORS
4742: }
4743: 
4744: static PyObject* THPAvgPool2DBackwardBackward0_kernel_size_getter(THPCppFunction *self, void *_unused) {
4745:   HANDLE_TH_ERRORS
4746:   auto prop = static_cast<AvgPool2DBackwardBackward0*>(self->cdata.get())->kernel_size;
4747:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4748:   for (auto i : c10::irange(prop.size())) {
4749:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
4750:   }
4751:   return tup;
4752:   END_HANDLE_TH_ERRORS
4753: }
4754: 
4755: static PyObject* THPAvgPool2DBackwardBackward0_padding_getter(THPCppFunction *self, void *_unused) {
4756:   HANDLE_TH_ERRORS
4757:   auto prop = static_cast<AvgPool2DBackwardBackward0*>(self->cdata.get())->padding;
4758:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4759:   for (auto i : c10::irange(prop.size())) {
4760:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
4761:   }
4762:   return tup;
4763:   END_HANDLE_TH_ERRORS
4764: }
4765: 
4766: static PyObject* THPAvgPool2DBackwardBackward0_stride_getter(THPCppFunction *self, void *_unused) {
4767:   HANDLE_TH_ERRORS
4768:   auto prop = static_cast<AvgPool2DBackwardBackward0*>(self->cdata.get())->stride;
4769:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4770:   for (auto i : c10::irange(prop.size())) {
4771:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
4772:   }
4773:   return tup;
4774:   END_HANDLE_TH_ERRORS
4775: }
4776: 
4777: static struct PyGetSetDef AvgPool2DBackwardBackward0_properties[] = {
4778:   THP_FUNCTION_DEFAULT_PROPERTIES,
4779:   {(char*)"_saved_ceil_mode", (getter)THPAvgPool2DBackwardBackward0_ceil_mode_getter, nullptr, nullptr, nullptr},
4780:   {(char*)"_saved_count_include_pad", (getter)THPAvgPool2DBackwardBackward0_count_include_pad_getter, nullptr, nullptr, nullptr},
4781:   {(char*)"_saved_divisor_override", (getter)THPAvgPool2DBackwardBackward0_divisor_override_getter, nullptr, nullptr, nullptr},
4782:   {(char*)"_saved_kernel_size", (getter)THPAvgPool2DBackwardBackward0_kernel_size_getter, nullptr, nullptr, nullptr},
4783:   {(char*)"_saved_padding", (getter)THPAvgPool2DBackwardBackward0_padding_getter, nullptr, nullptr, nullptr},
4784:   {(char*)"_saved_stride", (getter)THPAvgPool2DBackwardBackward0_stride_getter, nullptr, nullptr, nullptr},
4785:   {nullptr} /* sentinel */
4786: };
4787: 
4788: static PyObject* THPNllLossBackwardBackward0_ignore_index_getter(THPCppFunction *self, void *_unused) {
4789:   HANDLE_TH_ERRORS
4790:   auto prop = static_cast<NllLossBackwardBackward0*>(self->cdata.get())->ignore_index;
4791:   if (auto m = prop.maybe_as_int()) {
4792:     return PyLong_FromUnsignedLong(*m);
4793:   } else {
4794:     return py::cast(prop).release().ptr();
4795:   }
4796:   END_HANDLE_TH_ERRORS
4797: }
4798: 
4799: static PyObject* THPNllLossBackwardBackward0_reduction_getter(THPCppFunction *self, void *_unused) {
4800:   HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPAvgPool2DBackwardBackward0_count_include_pad_getter`, `THPAvgPool2DBackwardBackward0_divisor_override_getter`, `PyLong_FromUnsignedLong`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPAvgPool2DBackwardBackward0_count_include_pad_getter`, `THPAvgPool2DBackwardBackward0_divisor_override_getter`, `PyLong_FromUnsignedLong` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 4801-4880

```cpp
4801:   auto prop = static_cast<NllLossBackwardBackward0*>(self->cdata.get())->reduction;
4802:   return PyLong_FromUnsignedLong((int64_t) prop);
4803:   END_HANDLE_TH_ERRORS
4804: }
4805: 
4806: static PyObject* THPNllLossBackwardBackward0_target_getter(THPCppFunction *self, void *_unused) {
4807:   HANDLE_TH_ERRORS
4808:   const auto& prop = static_cast<NllLossBackwardBackward0*>(self->cdata.get())->target_;
4809:   return THPVariable_Wrap(prop.unpack(self->cdata));
4810:   END_HANDLE_TH_ERRORS
4811: }
4812: 
4813: static PyObject* THPNllLossBackwardBackward0_target_raw_getter(THPCppFunction *self, void *_unused) {
4814:   HANDLE_TH_ERRORS
4815:   const auto& prop = static_cast<NllLossBackwardBackward0*>(self->cdata.get())->target_;
4816:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4817:   return obj.release().ptr();
4818:   END_HANDLE_TH_ERRORS
4819: }
4820: 
4821: static PyObject* THPNllLossBackwardBackward0_weight_getter(THPCppFunction *self, void *_unused) {
4822:   HANDLE_TH_ERRORS
4823:   const auto& prop = static_cast<NllLossBackwardBackward0*>(self->cdata.get())->weight_;
4824:   return THPVariable_Wrap(prop.unpack(self->cdata));
4825:   END_HANDLE_TH_ERRORS
4826: }
4827: 
4828: static PyObject* THPNllLossBackwardBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
4829:   HANDLE_TH_ERRORS
4830:   const auto& prop = static_cast<NllLossBackwardBackward0*>(self->cdata.get())->weight_;
4831:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4832:   return obj.release().ptr();
4833:   END_HANDLE_TH_ERRORS
4834: }
4835: 
4836: static struct PyGetSetDef NllLossBackwardBackward0_properties[] = {
4837:   THP_FUNCTION_DEFAULT_PROPERTIES,
4838:   {(char*)"_saved_ignore_index", (getter)THPNllLossBackwardBackward0_ignore_index_getter, nullptr, nullptr, nullptr},
4839:   {(char*)"_saved_reduction", (getter)THPNllLossBackwardBackward0_reduction_getter, nullptr, nullptr, nullptr},
4840:   {(char*)"_saved_target", (getter)THPNllLossBackwardBackward0_target_getter, nullptr, nullptr, nullptr},
4841:   {(char*)"_raw_saved_target", (getter)THPNllLossBackwardBackward0_target_raw_getter, nullptr, nullptr, nullptr},
4842:   {(char*)"_saved_weight", (getter)THPNllLossBackwardBackward0_weight_getter, nullptr, nullptr, nullptr},
4843:   {(char*)"_raw_saved_weight", (getter)THPNllLossBackwardBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
4844:   {nullptr} /* sentinel */
4845: };
4846: 
4847: static PyObject* THPReflectionPad3DBackwardBackward0_padding_getter(THPCppFunction *self, void *_unused) {
4848:   HANDLE_TH_ERRORS
4849:   auto prop = static_cast<ReflectionPad3DBackwardBackward0*>(self->cdata.get())->padding;
4850:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4851:   for (auto i : c10::irange(prop.size())) {
4852:       auto si = prop[i];
4853:       if (auto m = si.maybe_as_int()) {
4854:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4855:       } else {
4856:         auto py_symint = py::cast(si).release().ptr();
4857:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4858:       }
4859:   }
4860:   return tup;
4861:   END_HANDLE_TH_ERRORS
4862: }
4863: 
4864: static struct PyGetSetDef ReflectionPad3DBackwardBackward0_properties[] = {
4865:   THP_FUNCTION_DEFAULT_PROPERTIES,
4866:   {(char*)"_saved_padding", (getter)THPReflectionPad3DBackwardBackward0_padding_getter, nullptr, nullptr, nullptr},
4867:   {nullptr} /* sentinel */
4868: };
4869: 
4870: static PyObject* THPReplicationPad2DBackwardBackward0_padding_getter(THPCppFunction *self, void *_unused) {
4871:   HANDLE_TH_ERRORS
4872:   auto prop = static_cast<ReplicationPad2DBackwardBackward0*>(self->cdata.get())->padding;
4873:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4874:   for (auto i : c10::irange(prop.size())) {
4875:       auto si = prop[i];
4876:       if (auto m = si.maybe_as_int()) {
4877:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4878:       } else {
4879:         auto py_symint = py::cast(si).release().ptr();
4880:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
```

- EN: The main execution path in this span is carried by `PyLong_FromUnsignedLong`, `THPNllLossBackwardBackward0_target_getter`, `THPVariable_Wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyLong_FromUnsignedLong`, `THPNllLossBackwardBackward0_target_getter`, `THPVariable_Wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4881-4960

```cpp
4881:       }
4882:   }
4883:   return tup;
4884:   END_HANDLE_TH_ERRORS
4885: }
4886: 
4887: static struct PyGetSetDef ReplicationPad2DBackwardBackward0_properties[] = {
4888:   THP_FUNCTION_DEFAULT_PROPERTIES,
4889:   {(char*)"_saved_padding", (getter)THPReplicationPad2DBackwardBackward0_padding_getter, nullptr, nullptr, nullptr},
4890:   {nullptr} /* sentinel */
4891: };
4892: 
4893: static PyObject* THPSparseMmReduceImplBackward0_other_getter(THPCppFunction *self, void *_unused) {
4894:   HANDLE_TH_ERRORS
4895:   const auto& prop = static_cast<SparseMmReduceImplBackward0*>(self->cdata.get())->other_;
4896:   return THPVariable_Wrap(prop.unpack(self->cdata));
4897:   END_HANDLE_TH_ERRORS
4898: }
4899: 
4900: static PyObject* THPSparseMmReduceImplBackward0_other_raw_getter(THPCppFunction *self, void *_unused) {
4901:   HANDLE_TH_ERRORS
4902:   const auto& prop = static_cast<SparseMmReduceImplBackward0*>(self->cdata.get())->other_;
4903:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4904:   return obj.release().ptr();
4905:   END_HANDLE_TH_ERRORS
4906: }
4907: 
4908: static PyObject* THPSparseMmReduceImplBackward0_reduce_getter(THPCppFunction *self, void *_unused) {
4909:   HANDLE_TH_ERRORS
4910:   auto prop = static_cast<SparseMmReduceImplBackward0*>(self->cdata.get())->reduce;
4911:   return PyUnicode_FromStringAndSize(prop.data(), prop.size());
4912:   END_HANDLE_TH_ERRORS
4913: }
4914: 
4915: static PyObject* THPSparseMmReduceImplBackward0_self_getter(THPCppFunction *self, void *_unused) {
4916:   HANDLE_TH_ERRORS
4917:   const auto& prop = static_cast<SparseMmReduceImplBackward0*>(self->cdata.get())->self_;
4918:   return THPVariable_Wrap(prop.unpack(self->cdata));
4919:   END_HANDLE_TH_ERRORS
4920: }
4921: 
4922: static PyObject* THPSparseMmReduceImplBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
4923:   HANDLE_TH_ERRORS
4924:   const auto& prop = static_cast<SparseMmReduceImplBackward0*>(self->cdata.get())->self_;
4925:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4926:   return obj.release().ptr();
4927:   END_HANDLE_TH_ERRORS
4928: }
4929: 
4930: static PyObject* THPSparseMmReduceImplBackward0_result1_getter(THPCppFunction *self, void *_unused) {
4931:   HANDLE_TH_ERRORS
4932:   const auto& prop = static_cast<SparseMmReduceImplBackward0*>(self->cdata.get())->result1_;
4933:   return THPVariable_Wrap(prop.unpack(self->cdata));
4934:   END_HANDLE_TH_ERRORS
4935: }
4936: 
4937: static PyObject* THPSparseMmReduceImplBackward0_result1_raw_getter(THPCppFunction *self, void *_unused) {
4938:   HANDLE_TH_ERRORS
4939:   const auto& prop = static_cast<SparseMmReduceImplBackward0*>(self->cdata.get())->result1_;
4940:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4941:   return obj.release().ptr();
4942:   END_HANDLE_TH_ERRORS
4943: }
4944: 
4945: static struct PyGetSetDef SparseMmReduceImplBackward0_properties[] = {
4946:   THP_FUNCTION_DEFAULT_PROPERTIES,
4947:   {(char*)"_saved_other", (getter)THPSparseMmReduceImplBackward0_other_getter, nullptr, nullptr, nullptr},
4948:   {(char*)"_raw_saved_other", (getter)THPSparseMmReduceImplBackward0_other_raw_getter, nullptr, nullptr, nullptr},
4949:   {(char*)"_saved_reduce", (getter)THPSparseMmReduceImplBackward0_reduce_getter, nullptr, nullptr, nullptr},
4950:   {(char*)"_saved_self", (getter)THPSparseMmReduceImplBackward0_self_getter, nullptr, nullptr, nullptr},
4951:   {(char*)"_raw_saved_self", (getter)THPSparseMmReduceImplBackward0_self_raw_getter, nullptr, nullptr, nullptr},
4952:   {(char*)"_saved_result1", (getter)THPSparseMmReduceImplBackward0_result1_getter, nullptr, nullptr, nullptr},
4953:   {(char*)"_raw_saved_result1", (getter)THPSparseMmReduceImplBackward0_result1_raw_getter, nullptr, nullptr, nullptr},
4954:   {nullptr} /* sentinel */
4955: };
4956: 
4957: static PyObject* THPSmoothL1LossBackwardBackward0_beta_getter(THPCppFunction *self, void *_unused) {
4958:   HANDLE_TH_ERRORS
4959:   auto prop = static_cast<SmoothL1LossBackwardBackward0*>(self->cdata.get())->beta;
4960:   return PyFloat_FromDouble((double) prop);
```

- EN: The main execution path in this span is carried by `THPSparseMmReduceImplBackward0_other_getter`, `THPVariable_Wrap`, `THPSparseMmReduceImplBackward0_other_raw_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPSparseMmReduceImplBackward0_other_getter`, `THPVariable_Wrap`, `THPSparseMmReduceImplBackward0_other_raw_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4961-5040

```cpp
4961:   END_HANDLE_TH_ERRORS
4962: }
4963: 
4964: static PyObject* THPSmoothL1LossBackwardBackward0_grad_output_getter(THPCppFunction *self, void *_unused) {
4965:   HANDLE_TH_ERRORS
4966:   const auto& prop = static_cast<SmoothL1LossBackwardBackward0*>(self->cdata.get())->grad_output_;
4967:   return THPVariable_Wrap(prop.unpack(self->cdata));
4968:   END_HANDLE_TH_ERRORS
4969: }
4970: 
4971: static PyObject* THPSmoothL1LossBackwardBackward0_grad_output_raw_getter(THPCppFunction *self, void *_unused) {
4972:   HANDLE_TH_ERRORS
4973:   const auto& prop = static_cast<SmoothL1LossBackwardBackward0*>(self->cdata.get())->grad_output_;
4974:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4975:   return obj.release().ptr();
4976:   END_HANDLE_TH_ERRORS
4977: }
4978: 
4979: static PyObject* THPSmoothL1LossBackwardBackward0_reduction_getter(THPCppFunction *self, void *_unused) {
4980:   HANDLE_TH_ERRORS
4981:   auto prop = static_cast<SmoothL1LossBackwardBackward0*>(self->cdata.get())->reduction;
4982:   return PyLong_FromUnsignedLong((int64_t) prop);
4983:   END_HANDLE_TH_ERRORS
4984: }
4985: 
4986: static PyObject* THPSmoothL1LossBackwardBackward0_self_getter(THPCppFunction *self, void *_unused) {
4987:   HANDLE_TH_ERRORS
4988:   const auto& prop = static_cast<SmoothL1LossBackwardBackward0*>(self->cdata.get())->self_;
4989:   return THPVariable_Wrap(prop.unpack(self->cdata));
4990:   END_HANDLE_TH_ERRORS
4991: }
4992: 
4993: static PyObject* THPSmoothL1LossBackwardBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
4994:   HANDLE_TH_ERRORS
4995:   const auto& prop = static_cast<SmoothL1LossBackwardBackward0*>(self->cdata.get())->self_;
4996:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4997:   return obj.release().ptr();
4998:   END_HANDLE_TH_ERRORS
4999: }
5000: 
5001: static PyObject* THPSmoothL1LossBackwardBackward0_target_getter(THPCppFunction *self, void *_unused) {
5002:   HANDLE_TH_ERRORS
5003:   const auto& prop = static_cast<SmoothL1LossBackwardBackward0*>(self->cdata.get())->target_;
5004:   return THPVariable_Wrap(prop.unpack(self->cdata));
5005:   END_HANDLE_TH_ERRORS
5006: }
5007: 
5008: static PyObject* THPSmoothL1LossBackwardBackward0_target_raw_getter(THPCppFunction *self, void *_unused) {
5009:   HANDLE_TH_ERRORS
5010:   const auto& prop = static_cast<SmoothL1LossBackwardBackward0*>(self->cdata.get())->target_;
5011:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5012:   return obj.release().ptr();
5013:   END_HANDLE_TH_ERRORS
5014: }
5015: 
5016: static struct PyGetSetDef SmoothL1LossBackwardBackward0_properties[] = {
5017:   THP_FUNCTION_DEFAULT_PROPERTIES,
5018:   {(char*)"_saved_beta", (getter)THPSmoothL1LossBackwardBackward0_beta_getter, nullptr, nullptr, nullptr},
5019:   {(char*)"_saved_grad_output", (getter)THPSmoothL1LossBackwardBackward0_grad_output_getter, nullptr, nullptr, nullptr},
5020:   {(char*)"_raw_saved_grad_output", (getter)THPSmoothL1LossBackwardBackward0_grad_output_raw_getter, nullptr, nullptr, nullptr},
5021:   {(char*)"_saved_reduction", (getter)THPSmoothL1LossBackwardBackward0_reduction_getter, nullptr, nullptr, nullptr},
5022:   {(char*)"_saved_self", (getter)THPSmoothL1LossBackwardBackward0_self_getter, nullptr, nullptr, nullptr},
5023:   {(char*)"_raw_saved_self", (getter)THPSmoothL1LossBackwardBackward0_self_raw_getter, nullptr, nullptr, nullptr},
5024:   {(char*)"_saved_target", (getter)THPSmoothL1LossBackwardBackward0_target_getter, nullptr, nullptr, nullptr},
5025:   {(char*)"_raw_saved_target", (getter)THPSmoothL1LossBackwardBackward0_target_raw_getter, nullptr, nullptr, nullptr},
5026:   {nullptr} /* sentinel */
5027: };
5028: 
5029: static PyObject* THPHuberLossBackwardBackward0_delta_getter(THPCppFunction *self, void *_unused) {
5030:   HANDLE_TH_ERRORS
5031:   auto prop = static_cast<HuberLossBackwardBackward0*>(self->cdata.get())->delta;
5032:   return PyFloat_FromDouble((double) prop);
5033:   END_HANDLE_TH_ERRORS
5034: }
5035: 
5036: static PyObject* THPHuberLossBackwardBackward0_grad_output_getter(THPCppFunction *self, void *_unused) {
5037:   HANDLE_TH_ERRORS
5038:   const auto& prop = static_cast<HuberLossBackwardBackward0*>(self->cdata.get())->grad_output_;
5039:   return THPVariable_Wrap(prop.unpack(self->cdata));
5040:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPSmoothL1LossBackwardBackward0_grad_output_getter`, `THPVariable_Wrap`, `THPSmoothL1LossBackwardBackward0_grad_output_raw_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPSmoothL1LossBackwardBackward0_grad_output_getter`, `THPVariable_Wrap`, `THPSmoothL1LossBackwardBackward0_grad_output_raw_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5041-5120

```cpp
5041: }
5042: 
5043: static PyObject* THPHuberLossBackwardBackward0_grad_output_raw_getter(THPCppFunction *self, void *_unused) {
5044:   HANDLE_TH_ERRORS
5045:   const auto& prop = static_cast<HuberLossBackwardBackward0*>(self->cdata.get())->grad_output_;
5046:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5047:   return obj.release().ptr();
5048:   END_HANDLE_TH_ERRORS
5049: }
5050: 
5051: static PyObject* THPHuberLossBackwardBackward0_reduction_getter(THPCppFunction *self, void *_unused) {
5052:   HANDLE_TH_ERRORS
5053:   auto prop = static_cast<HuberLossBackwardBackward0*>(self->cdata.get())->reduction;
5054:   return PyLong_FromUnsignedLong((int64_t) prop);
5055:   END_HANDLE_TH_ERRORS
5056: }
5057: 
5058: static PyObject* THPHuberLossBackwardBackward0_self_getter(THPCppFunction *self, void *_unused) {
5059:   HANDLE_TH_ERRORS
5060:   const auto& prop = static_cast<HuberLossBackwardBackward0*>(self->cdata.get())->self_;
5061:   return THPVariable_Wrap(prop.unpack(self->cdata));
5062:   END_HANDLE_TH_ERRORS
5063: }
5064: 
5065: static PyObject* THPHuberLossBackwardBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
5066:   HANDLE_TH_ERRORS
5067:   const auto& prop = static_cast<HuberLossBackwardBackward0*>(self->cdata.get())->self_;
5068:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5069:   return obj.release().ptr();
5070:   END_HANDLE_TH_ERRORS
5071: }
5072: 
5073: static PyObject* THPHuberLossBackwardBackward0_target_getter(THPCppFunction *self, void *_unused) {
5074:   HANDLE_TH_ERRORS
5075:   const auto& prop = static_cast<HuberLossBackwardBackward0*>(self->cdata.get())->target_;
5076:   return THPVariable_Wrap(prop.unpack(self->cdata));
5077:   END_HANDLE_TH_ERRORS
5078: }
5079: 
5080: static PyObject* THPHuberLossBackwardBackward0_target_raw_getter(THPCppFunction *self, void *_unused) {
5081:   HANDLE_TH_ERRORS
5082:   const auto& prop = static_cast<HuberLossBackwardBackward0*>(self->cdata.get())->target_;
5083:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5084:   return obj.release().ptr();
5085:   END_HANDLE_TH_ERRORS
5086: }
5087: 
5088: static struct PyGetSetDef HuberLossBackwardBackward0_properties[] = {
5089:   THP_FUNCTION_DEFAULT_PROPERTIES,
5090:   {(char*)"_saved_delta", (getter)THPHuberLossBackwardBackward0_delta_getter, nullptr, nullptr, nullptr},
5091:   {(char*)"_saved_grad_output", (getter)THPHuberLossBackwardBackward0_grad_output_getter, nullptr, nullptr, nullptr},
5092:   {(char*)"_raw_saved_grad_output", (getter)THPHuberLossBackwardBackward0_grad_output_raw_getter, nullptr, nullptr, nullptr},
5093:   {(char*)"_saved_reduction", (getter)THPHuberLossBackwardBackward0_reduction_getter, nullptr, nullptr, nullptr},
5094:   {(char*)"_saved_self", (getter)THPHuberLossBackwardBackward0_self_getter, nullptr, nullptr, nullptr},
5095:   {(char*)"_raw_saved_self", (getter)THPHuberLossBackwardBackward0_self_raw_getter, nullptr, nullptr, nullptr},
5096:   {(char*)"_saved_target", (getter)THPHuberLossBackwardBackward0_target_getter, nullptr, nullptr, nullptr},
5097:   {(char*)"_raw_saved_target", (getter)THPHuberLossBackwardBackward0_target_raw_getter, nullptr, nullptr, nullptr},
5098:   {nullptr} /* sentinel */
5099: };
5100: 
5101: static PyObject* THPUpsampleBicubic2DBackwardBackward0_align_corners_getter(THPCppFunction *self, void *_unused) {
5102:   HANDLE_TH_ERRORS
5103:   auto prop = static_cast<UpsampleBicubic2DBackwardBackward0*>(self->cdata.get())->align_corners;
5104:   if (prop) {
5105:     Py_RETURN_TRUE;
5106:   } else {
5107:     Py_RETURN_FALSE;
5108:   }
5109:   END_HANDLE_TH_ERRORS
5110: }
5111: 
5112: static PyObject* THPUpsampleBicubic2DBackwardBackward0_output_size_getter(THPCppFunction *self, void *_unused) {
5113:   HANDLE_TH_ERRORS
5114:   auto prop = static_cast<UpsampleBicubic2DBackwardBackward0*>(self->cdata.get())->output_size;
5115:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5116:   for (auto i : c10::irange(prop.size())) {
5117:       auto si = prop[i];
5118:       if (auto m = si.maybe_as_int()) {
5119:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
5120:       } else {
```

- EN: The main execution path in this span is carried by `THPHuberLossBackwardBackward0_grad_output_raw_getter`, `cast`, `THPHuberLossBackwardBackward0_reduction_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPHuberLossBackwardBackward0_grad_output_raw_getter`, `cast`, `THPHuberLossBackwardBackward0_reduction_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5121-5200

```cpp
5121:         auto py_symint = py::cast(si).release().ptr();
5122:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
5123:       }
5124:   }
5125:   return tup;
5126:   END_HANDLE_TH_ERRORS
5127: }
5128: 
5129: static PyObject* THPUpsampleBicubic2DBackwardBackward0_scales_h_getter(THPCppFunction *self, void *_unused) {
5130:   HANDLE_TH_ERRORS
5131:   auto opt_prop = static_cast<UpsampleBicubic2DBackwardBackward0*>(self->cdata.get())->scales_h;
5132:   if (!opt_prop.has_value()) {
5133:     Py_RETURN_NONE;
5134:   }
5135:   auto prop = opt_prop.value();
5136:   return PyFloat_FromDouble((double) prop);
5137:   END_HANDLE_TH_ERRORS
5138: }
5139: 
5140: static PyObject* THPUpsampleBicubic2DBackwardBackward0_scales_w_getter(THPCppFunction *self, void *_unused) {
5141:   HANDLE_TH_ERRORS
5142:   auto opt_prop = static_cast<UpsampleBicubic2DBackwardBackward0*>(self->cdata.get())->scales_w;
5143:   if (!opt_prop.has_value()) {
5144:     Py_RETURN_NONE;
5145:   }
5146:   auto prop = opt_prop.value();
5147:   return PyFloat_FromDouble((double) prop);
5148:   END_HANDLE_TH_ERRORS
5149: }
5150: 
5151: static struct PyGetSetDef UpsampleBicubic2DBackwardBackward0_properties[] = {
5152:   THP_FUNCTION_DEFAULT_PROPERTIES,
5153:   {(char*)"_saved_align_corners", (getter)THPUpsampleBicubic2DBackwardBackward0_align_corners_getter, nullptr, nullptr, nullptr},
5154:   {(char*)"_saved_output_size", (getter)THPUpsampleBicubic2DBackwardBackward0_output_size_getter, nullptr, nullptr, nullptr},
5155:   {(char*)"_saved_scales_h", (getter)THPUpsampleBicubic2DBackwardBackward0_scales_h_getter, nullptr, nullptr, nullptr},
5156:   {(char*)"_saved_scales_w", (getter)THPUpsampleBicubic2DBackwardBackward0_scales_w_getter, nullptr, nullptr, nullptr},
5157:   {nullptr} /* sentinel */
5158: };
5159: 
5160: static PyObject* THPUpsampleTrilinear3DBackwardBackward0_align_corners_getter(THPCppFunction *self, void *_unused) {
5161:   HANDLE_TH_ERRORS
5162:   auto prop = static_cast<UpsampleTrilinear3DBackwardBackward0*>(self->cdata.get())->align_corners;
5163:   if (prop) {
5164:     Py_RETURN_TRUE;
5165:   } else {
5166:     Py_RETURN_FALSE;
5167:   }
5168:   END_HANDLE_TH_ERRORS
5169: }
5170: 
5171: static PyObject* THPUpsampleTrilinear3DBackwardBackward0_output_size_getter(THPCppFunction *self, void *_unused) {
5172:   HANDLE_TH_ERRORS
5173:   auto prop = static_cast<UpsampleTrilinear3DBackwardBackward0*>(self->cdata.get())->output_size;
5174:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5175:   for (auto i : c10::irange(prop.size())) {
5176:       auto si = prop[i];
5177:       if (auto m = si.maybe_as_int()) {
5178:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
5179:       } else {
5180:         auto py_symint = py::cast(si).release().ptr();
5181:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
5182:       }
5183:   }
5184:   return tup;
5185:   END_HANDLE_TH_ERRORS
5186: }
5187: 
5188: static PyObject* THPUpsampleTrilinear3DBackwardBackward0_scales_d_getter(THPCppFunction *self, void *_unused) {
5189:   HANDLE_TH_ERRORS
5190:   auto opt_prop = static_cast<UpsampleTrilinear3DBackwardBackward0*>(self->cdata.get())->scales_d;
5191:   if (!opt_prop.has_value()) {
5192:     Py_RETURN_NONE;
5193:   }
5194:   auto prop = opt_prop.value();
5195:   return PyFloat_FromDouble((double) prop);
5196:   END_HANDLE_TH_ERRORS
5197: }
5198: 
5199: static PyObject* THPUpsampleTrilinear3DBackwardBackward0_scales_h_getter(THPCppFunction *self, void *_unused) {
5200:   HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `cast`, `PyTuple_SetItem`, `THPUpsampleBicubic2DBackwardBackward0_scales_h_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `cast`, `PyTuple_SetItem`, `THPUpsampleBicubic2DBackwardBackward0_scales_h_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5201-5280

```cpp
5201:   auto opt_prop = static_cast<UpsampleTrilinear3DBackwardBackward0*>(self->cdata.get())->scales_h;
5202:   if (!opt_prop.has_value()) {
5203:     Py_RETURN_NONE;
5204:   }
5205:   auto prop = opt_prop.value();
5206:   return PyFloat_FromDouble((double) prop);
5207:   END_HANDLE_TH_ERRORS
5208: }
5209: 
5210: static PyObject* THPUpsampleTrilinear3DBackwardBackward0_scales_w_getter(THPCppFunction *self, void *_unused) {
5211:   HANDLE_TH_ERRORS
5212:   auto opt_prop = static_cast<UpsampleTrilinear3DBackwardBackward0*>(self->cdata.get())->scales_w;
5213:   if (!opt_prop.has_value()) {
5214:     Py_RETURN_NONE;
5215:   }
5216:   auto prop = opt_prop.value();
5217:   return PyFloat_FromDouble((double) prop);
5218:   END_HANDLE_TH_ERRORS
5219: }
5220: 
5221: static struct PyGetSetDef UpsampleTrilinear3DBackwardBackward0_properties[] = {
5222:   THP_FUNCTION_DEFAULT_PROPERTIES,
5223:   {(char*)"_saved_align_corners", (getter)THPUpsampleTrilinear3DBackwardBackward0_align_corners_getter, nullptr, nullptr, nullptr},
5224:   {(char*)"_saved_output_size", (getter)THPUpsampleTrilinear3DBackwardBackward0_output_size_getter, nullptr, nullptr, nullptr},
5225:   {(char*)"_saved_scales_d", (getter)THPUpsampleTrilinear3DBackwardBackward0_scales_d_getter, nullptr, nullptr, nullptr},
5226:   {(char*)"_saved_scales_h", (getter)THPUpsampleTrilinear3DBackwardBackward0_scales_h_getter, nullptr, nullptr, nullptr},
5227:   {(char*)"_saved_scales_w", (getter)THPUpsampleTrilinear3DBackwardBackward0_scales_w_getter, nullptr, nullptr, nullptr},
5228:   {nullptr} /* sentinel */
5229: };
5230: 
5231: static PyObject* THPUpsampleNearest3DBackwardBackward0_output_size_getter(THPCppFunction *self, void *_unused) {
5232:   HANDLE_TH_ERRORS
5233:   auto prop = static_cast<UpsampleNearest3DBackwardBackward0*>(self->cdata.get())->output_size;
5234:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5235:   for (auto i : c10::irange(prop.size())) {
5236:       auto si = prop[i];
5237:       if (auto m = si.maybe_as_int()) {
5238:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
5239:       } else {
5240:         auto py_symint = py::cast(si).release().ptr();
5241:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
5242:       }
5243:   }
5244:   return tup;
5245:   END_HANDLE_TH_ERRORS
5246: }
5247: 
5248: static PyObject* THPUpsampleNearest3DBackwardBackward0_scales_d_getter(THPCppFunction *self, void *_unused) {
5249:   HANDLE_TH_ERRORS
5250:   auto opt_prop = static_cast<UpsampleNearest3DBackwardBackward0*>(self->cdata.get())->scales_d;
5251:   if (!opt_prop.has_value()) {
5252:     Py_RETURN_NONE;
5253:   }
5254:   auto prop = opt_prop.value();
5255:   return PyFloat_FromDouble((double) prop);
5256:   END_HANDLE_TH_ERRORS
5257: }
5258: 
5259: static PyObject* THPUpsampleNearest3DBackwardBackward0_scales_h_getter(THPCppFunction *self, void *_unused) {
5260:   HANDLE_TH_ERRORS
5261:   auto opt_prop = static_cast<UpsampleNearest3DBackwardBackward0*>(self->cdata.get())->scales_h;
5262:   if (!opt_prop.has_value()) {
5263:     Py_RETURN_NONE;
5264:   }
5265:   auto prop = opt_prop.value();
5266:   return PyFloat_FromDouble((double) prop);
5267:   END_HANDLE_TH_ERRORS
5268: }
5269: 
5270: static PyObject* THPUpsampleNearest3DBackwardBackward0_scales_w_getter(THPCppFunction *self, void *_unused) {
5271:   HANDLE_TH_ERRORS
5272:   auto opt_prop = static_cast<UpsampleNearest3DBackwardBackward0*>(self->cdata.get())->scales_w;
5273:   if (!opt_prop.has_value()) {
5274:     Py_RETURN_NONE;
5275:   }
5276:   auto prop = opt_prop.value();
5277:   return PyFloat_FromDouble((double) prop);
5278:   END_HANDLE_TH_ERRORS
5279: }
5280: 
```

- EN: The main execution path in this span is carried by `PyFloat_FromDouble`, `THPUpsampleTrilinear3DBackwardBackward0_scales_w_getter`, `THPUpsampleNearest3DBackwardBackward0_output_size_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyFloat_FromDouble`, `THPUpsampleTrilinear3DBackwardBackward0_scales_w_getter`, `THPUpsampleNearest3DBackwardBackward0_output_size_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5281-5360

```cpp
5281: static struct PyGetSetDef UpsampleNearest3DBackwardBackward0_properties[] = {
5282:   THP_FUNCTION_DEFAULT_PROPERTIES,
5283:   {(char*)"_saved_output_size", (getter)THPUpsampleNearest3DBackwardBackward0_output_size_getter, nullptr, nullptr, nullptr},
5284:   {(char*)"_saved_scales_d", (getter)THPUpsampleNearest3DBackwardBackward0_scales_d_getter, nullptr, nullptr, nullptr},
5285:   {(char*)"_saved_scales_h", (getter)THPUpsampleNearest3DBackwardBackward0_scales_h_getter, nullptr, nullptr, nullptr},
5286:   {(char*)"_saved_scales_w", (getter)THPUpsampleNearest3DBackwardBackward0_scales_w_getter, nullptr, nullptr, nullptr},
5287:   {nullptr} /* sentinel */
5288: };
5289: 
5290: static PyObject* THPUpsampleNearestExact3DBackwardBackward0_output_size_getter(THPCppFunction *self, void *_unused) {
5291:   HANDLE_TH_ERRORS
5292:   auto prop = static_cast<UpsampleNearestExact3DBackwardBackward0*>(self->cdata.get())->output_size;
5293:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5294:   for (auto i : c10::irange(prop.size())) {
5295:       auto si = prop[i];
5296:       if (auto m = si.maybe_as_int()) {
5297:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
5298:       } else {
5299:         auto py_symint = py::cast(si).release().ptr();
5300:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
5301:       }
5302:   }
5303:   return tup;
5304:   END_HANDLE_TH_ERRORS
5305: }
5306: 
5307: static PyObject* THPUpsampleNearestExact3DBackwardBackward0_scales_d_getter(THPCppFunction *self, void *_unused) {
5308:   HANDLE_TH_ERRORS
5309:   auto opt_prop = static_cast<UpsampleNearestExact3DBackwardBackward0*>(self->cdata.get())->scales_d;
5310:   if (!opt_prop.has_value()) {
5311:     Py_RETURN_NONE;
5312:   }
5313:   auto prop = opt_prop.value();
5314:   return PyFloat_FromDouble((double) prop);
5315:   END_HANDLE_TH_ERRORS
5316: }
5317: 
5318: static PyObject* THPUpsampleNearestExact3DBackwardBackward0_scales_h_getter(THPCppFunction *self, void *_unused) {
5319:   HANDLE_TH_ERRORS
5320:   auto opt_prop = static_cast<UpsampleNearestExact3DBackwardBackward0*>(self->cdata.get())->scales_h;
5321:   if (!opt_prop.has_value()) {
5322:     Py_RETURN_NONE;
5323:   }
5324:   auto prop = opt_prop.value();
5325:   return PyFloat_FromDouble((double) prop);
5326:   END_HANDLE_TH_ERRORS
5327: }
5328: 
5329: static PyObject* THPUpsampleNearestExact3DBackwardBackward0_scales_w_getter(THPCppFunction *self, void *_unused) {
5330:   HANDLE_TH_ERRORS
5331:   auto opt_prop = static_cast<UpsampleNearestExact3DBackwardBackward0*>(self->cdata.get())->scales_w;
5332:   if (!opt_prop.has_value()) {
5333:     Py_RETURN_NONE;
5334:   }
5335:   auto prop = opt_prop.value();
5336:   return PyFloat_FromDouble((double) prop);
5337:   END_HANDLE_TH_ERRORS
5338: }
5339: 
5340: static struct PyGetSetDef UpsampleNearestExact3DBackwardBackward0_properties[] = {
5341:   THP_FUNCTION_DEFAULT_PROPERTIES,
5342:   {(char*)"_saved_output_size", (getter)THPUpsampleNearestExact3DBackwardBackward0_output_size_getter, nullptr, nullptr, nullptr},
5343:   {(char*)"_saved_scales_d", (getter)THPUpsampleNearestExact3DBackwardBackward0_scales_d_getter, nullptr, nullptr, nullptr},
5344:   {(char*)"_saved_scales_h", (getter)THPUpsampleNearestExact3DBackwardBackward0_scales_h_getter, nullptr, nullptr, nullptr},
5345:   {(char*)"_saved_scales_w", (getter)THPUpsampleNearestExact3DBackwardBackward0_scales_w_getter, nullptr, nullptr, nullptr},
5346:   {nullptr} /* sentinel */
5347: };
5348: 
5349: static PyObject* THPCudnnConvolutionBackward0_dilation_getter(THPCppFunction *self, void *_unused) {
5350:   HANDLE_TH_ERRORS
5351:   auto prop = static_cast<CudnnConvolutionBackward0*>(self->cdata.get())->dilation;
5352:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5353:   for (auto i : c10::irange(prop.size())) {
5354:       auto si = prop[i];
5355:       if (auto m = si.maybe_as_int()) {
5356:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
5357:       } else {
5358:         auto py_symint = py::cast(si).release().ptr();
5359:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
5360:       }
```

- EN: The main execution path in this span is carried by `THPUpsampleNearestExact3DBackwardBackward0_output_size_getter`, `PyTuple_New`, `PyTuple_SetItem`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPUpsampleNearestExact3DBackwardBackward0_output_size_getter`, `PyTuple_New`, `PyTuple_SetItem` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5361-5440

```cpp
5361:   }
5362:   return tup;
5363:   END_HANDLE_TH_ERRORS
5364: }
5365: 
5366: static PyObject* THPCudnnConvolutionBackward0_groups_getter(THPCppFunction *self, void *_unused) {
5367:   HANDLE_TH_ERRORS
5368:   auto prop = static_cast<CudnnConvolutionBackward0*>(self->cdata.get())->groups;
5369:   if (auto m = prop.maybe_as_int()) {
5370:     return PyLong_FromUnsignedLong(*m);
5371:   } else {
5372:     return py::cast(prop).release().ptr();
5373:   }
5374:   END_HANDLE_TH_ERRORS
5375: }
5376: 
5377: static PyObject* THPCudnnConvolutionBackward0_padding_getter(THPCppFunction *self, void *_unused) {
5378:   HANDLE_TH_ERRORS
5379:   auto prop = static_cast<CudnnConvolutionBackward0*>(self->cdata.get())->padding;
5380:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5381:   for (auto i : c10::irange(prop.size())) {
5382:       auto si = prop[i];
5383:       if (auto m = si.maybe_as_int()) {
5384:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
5385:       } else {
5386:         auto py_symint = py::cast(si).release().ptr();
5387:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
5388:       }
5389:   }
5390:   return tup;
5391:   END_HANDLE_TH_ERRORS
5392: }
5393: 
5394: static PyObject* THPCudnnConvolutionBackward0_self_getter(THPCppFunction *self, void *_unused) {
5395:   HANDLE_TH_ERRORS
5396:   const auto& prop = static_cast<CudnnConvolutionBackward0*>(self->cdata.get())->self_;
5397:   return THPVariable_Wrap(prop.unpack(self->cdata));
5398:   END_HANDLE_TH_ERRORS
5399: }
5400: 
5401: static PyObject* THPCudnnConvolutionBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
5402:   HANDLE_TH_ERRORS
5403:   const auto& prop = static_cast<CudnnConvolutionBackward0*>(self->cdata.get())->self_;
5404:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5405:   return obj.release().ptr();
5406:   END_HANDLE_TH_ERRORS
5407: }
5408: 
5409: static PyObject* THPCudnnConvolutionBackward0_stride_getter(THPCppFunction *self, void *_unused) {
5410:   HANDLE_TH_ERRORS
5411:   auto prop = static_cast<CudnnConvolutionBackward0*>(self->cdata.get())->stride;
5412:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5413:   for (auto i : c10::irange(prop.size())) {
5414:       auto si = prop[i];
5415:       if (auto m = si.maybe_as_int()) {
5416:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
5417:       } else {
5418:         auto py_symint = py::cast(si).release().ptr();
5419:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
5420:       }
5421:   }
5422:   return tup;
5423:   END_HANDLE_TH_ERRORS
5424: }
5425: 
5426: static PyObject* THPCudnnConvolutionBackward0_weight_getter(THPCppFunction *self, void *_unused) {
5427:   HANDLE_TH_ERRORS
5428:   const auto& prop = static_cast<CudnnConvolutionBackward0*>(self->cdata.get())->weight_;
5429:   return THPVariable_Wrap(prop.unpack(self->cdata));
5430:   END_HANDLE_TH_ERRORS
5431: }
5432: 
5433: static PyObject* THPCudnnConvolutionBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
5434:   HANDLE_TH_ERRORS
5435:   const auto& prop = static_cast<CudnnConvolutionBackward0*>(self->cdata.get())->weight_;
5436:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5437:   return obj.release().ptr();
5438:   END_HANDLE_TH_ERRORS
5439: }
5440: 
```

- EN: The main execution path in this span is carried by `THPCudnnConvolutionBackward0_groups_getter`, `PyLong_FromUnsignedLong`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPCudnnConvolutionBackward0_groups_getter`, `PyLong_FromUnsignedLong`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5441-5520

```cpp
5441: static struct PyGetSetDef CudnnConvolutionBackward0_properties[] = {
5442:   THP_FUNCTION_DEFAULT_PROPERTIES,
5443:   {(char*)"_saved_dilation", (getter)THPCudnnConvolutionBackward0_dilation_getter, nullptr, nullptr, nullptr},
5444:   {(char*)"_saved_groups", (getter)THPCudnnConvolutionBackward0_groups_getter, nullptr, nullptr, nullptr},
5445:   {(char*)"_saved_padding", (getter)THPCudnnConvolutionBackward0_padding_getter, nullptr, nullptr, nullptr},
5446:   {(char*)"_saved_self", (getter)THPCudnnConvolutionBackward0_self_getter, nullptr, nullptr, nullptr},
5447:   {(char*)"_raw_saved_self", (getter)THPCudnnConvolutionBackward0_self_raw_getter, nullptr, nullptr, nullptr},
5448:   {(char*)"_saved_stride", (getter)THPCudnnConvolutionBackward0_stride_getter, nullptr, nullptr, nullptr},
5449:   {(char*)"_saved_weight", (getter)THPCudnnConvolutionBackward0_weight_getter, nullptr, nullptr, nullptr},
5450:   {(char*)"_raw_saved_weight", (getter)THPCudnnConvolutionBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
5451:   {nullptr} /* sentinel */
5452: };
5453: 
5454: 
5455: 
5456: static struct PyGetSetDef CudnnRnnBackwardBackward0_properties[] = {
5457:   THP_FUNCTION_DEFAULT_PROPERTIES,
5458: 
5459:   {nullptr} /* sentinel */
5460: };
5461: 
5462: static PyObject* THPMiopenCtcLossBackward0_zero_infinity_getter(THPCppFunction *self, void *_unused) {
5463:   HANDLE_TH_ERRORS
5464:   auto prop = static_cast<MiopenCtcLossBackward0*>(self->cdata.get())->zero_infinity;
5465:   if (prop) {
5466:     Py_RETURN_TRUE;
5467:   } else {
5468:     Py_RETURN_FALSE;
5469:   }
5470:   END_HANDLE_TH_ERRORS
5471: }
5472: 
5473: static PyObject* THPMiopenCtcLossBackward0_result0_getter(THPCppFunction *self, void *_unused) {
5474:   HANDLE_TH_ERRORS
5475:   const auto& prop = static_cast<MiopenCtcLossBackward0*>(self->cdata.get())->result0_;
5476:   return THPVariable_Wrap(prop.unpack(self->cdata));
5477:   END_HANDLE_TH_ERRORS
5478: }
5479: 
5480: static PyObject* THPMiopenCtcLossBackward0_result0_raw_getter(THPCppFunction *self, void *_unused) {
5481:   HANDLE_TH_ERRORS
5482:   const auto& prop = static_cast<MiopenCtcLossBackward0*>(self->cdata.get())->result0_;
5483:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5484:   return obj.release().ptr();
5485:   END_HANDLE_TH_ERRORS
5486: }
5487: 
5488: static PyObject* THPMiopenCtcLossBackward0_result1_getter(THPCppFunction *self, void *_unused) {
5489:   HANDLE_TH_ERRORS
5490:   const auto& prop = static_cast<MiopenCtcLossBackward0*>(self->cdata.get())->result1_;
5491:   return THPVariable_Wrap(prop.unpack(self->cdata));
5492:   END_HANDLE_TH_ERRORS
5493: }
5494: 
5495: static PyObject* THPMiopenCtcLossBackward0_result1_raw_getter(THPCppFunction *self, void *_unused) {
5496:   HANDLE_TH_ERRORS
5497:   const auto& prop = static_cast<MiopenCtcLossBackward0*>(self->cdata.get())->result1_;
5498:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5499:   return obj.release().ptr();
5500:   END_HANDLE_TH_ERRORS
5501: }
5502: 
5503: static struct PyGetSetDef MiopenCtcLossBackward0_properties[] = {
5504:   THP_FUNCTION_DEFAULT_PROPERTIES,
5505:   {(char*)"_saved_zero_infinity", (getter)THPMiopenCtcLossBackward0_zero_infinity_getter, nullptr, nullptr, nullptr},
5506:   {(char*)"_saved_result0", (getter)THPMiopenCtcLossBackward0_result0_getter, nullptr, nullptr, nullptr},
5507:   {(char*)"_raw_saved_result0", (getter)THPMiopenCtcLossBackward0_result0_raw_getter, nullptr, nullptr, nullptr},
5508:   {(char*)"_saved_result1", (getter)THPMiopenCtcLossBackward0_result1_getter, nullptr, nullptr, nullptr},
5509:   {(char*)"_raw_saved_result1", (getter)THPMiopenCtcLossBackward0_result1_raw_getter, nullptr, nullptr, nullptr},
5510:   {nullptr} /* sentinel */
5511: };
5512: 
5513: static PyObject* THPMiopenCtcLossBackward1_zero_infinity_getter(THPCppFunction *self, void *_unused) {
5514:   HANDLE_TH_ERRORS
5515:   auto prop = static_cast<MiopenCtcLossBackward1*>(self->cdata.get())->zero_infinity;
5516:   if (prop) {
5517:     Py_RETURN_TRUE;
5518:   } else {
5519:     Py_RETURN_FALSE;
5520:   }
```

- EN: The main execution path in this span is carried by `THPMiopenCtcLossBackward0_zero_infinity_getter`, `THPMiopenCtcLossBackward0_result0_getter`, `THPVariable_Wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPMiopenCtcLossBackward0_zero_infinity_getter`, `THPMiopenCtcLossBackward0_result0_getter`, `THPVariable_Wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5521-5600

```cpp
5521:   END_HANDLE_TH_ERRORS
5522: }
5523: 
5524: static PyObject* THPMiopenCtcLossBackward1_result0_getter(THPCppFunction *self, void *_unused) {
5525:   HANDLE_TH_ERRORS
5526:   const auto& prop = static_cast<MiopenCtcLossBackward1*>(self->cdata.get())->result0_;
5527:   return THPVariable_Wrap(prop.unpack(self->cdata));
5528:   END_HANDLE_TH_ERRORS
5529: }
5530: 
5531: static PyObject* THPMiopenCtcLossBackward1_result0_raw_getter(THPCppFunction *self, void *_unused) {
5532:   HANDLE_TH_ERRORS
5533:   const auto& prop = static_cast<MiopenCtcLossBackward1*>(self->cdata.get())->result0_;
5534:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5535:   return obj.release().ptr();
5536:   END_HANDLE_TH_ERRORS
5537: }
5538: 
5539: static PyObject* THPMiopenCtcLossBackward1_result1_getter(THPCppFunction *self, void *_unused) {
5540:   HANDLE_TH_ERRORS
5541:   const auto& prop = static_cast<MiopenCtcLossBackward1*>(self->cdata.get())->result1_;
5542:   return THPVariable_Wrap(prop.unpack(self->cdata));
5543:   END_HANDLE_TH_ERRORS
5544: }
5545: 
5546: static PyObject* THPMiopenCtcLossBackward1_result1_raw_getter(THPCppFunction *self, void *_unused) {
5547:   HANDLE_TH_ERRORS
5548:   const auto& prop = static_cast<MiopenCtcLossBackward1*>(self->cdata.get())->result1_;
5549:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5550:   return obj.release().ptr();
5551:   END_HANDLE_TH_ERRORS
5552: }
5553: 
5554: static struct PyGetSetDef MiopenCtcLossBackward1_properties[] = {
5555:   THP_FUNCTION_DEFAULT_PROPERTIES,
5556:   {(char*)"_saved_zero_infinity", (getter)THPMiopenCtcLossBackward1_zero_infinity_getter, nullptr, nullptr, nullptr},
5557:   {(char*)"_saved_result0", (getter)THPMiopenCtcLossBackward1_result0_getter, nullptr, nullptr, nullptr},
5558:   {(char*)"_raw_saved_result0", (getter)THPMiopenCtcLossBackward1_result0_raw_getter, nullptr, nullptr, nullptr},
5559:   {(char*)"_saved_result1", (getter)THPMiopenCtcLossBackward1_result1_getter, nullptr, nullptr, nullptr},
5560:   {(char*)"_raw_saved_result1", (getter)THPMiopenCtcLossBackward1_result1_raw_getter, nullptr, nullptr, nullptr},
5561:   {nullptr} /* sentinel */
5562: };
5563: 
5564: static PyObject* THPNestedFromPaddedBackward0_fuse_transform_0213_getter(THPCppFunction *self, void *_unused) {
5565:   HANDLE_TH_ERRORS
5566:   auto prop = static_cast<NestedFromPaddedBackward0*>(self->cdata.get())->fuse_transform_0213;
5567:   if (prop) {
5568:     Py_RETURN_TRUE;
5569:   } else {
5570:     Py_RETURN_FALSE;
5571:   }
5572:   END_HANDLE_TH_ERRORS
5573: }
5574: 
5575: static PyObject* THPNestedFromPaddedBackward0_padded_getter(THPCppFunction *self, void *_unused) {
5576:   HANDLE_TH_ERRORS
5577:   const auto& prop = static_cast<NestedFromPaddedBackward0*>(self->cdata.get())->padded_;
5578:   return THPVariable_Wrap(prop.unpack(self->cdata));
5579:   END_HANDLE_TH_ERRORS
5580: }
5581: 
5582: static PyObject* THPNestedFromPaddedBackward0_padded_raw_getter(THPCppFunction *self, void *_unused) {
5583:   HANDLE_TH_ERRORS
5584:   const auto& prop = static_cast<NestedFromPaddedBackward0*>(self->cdata.get())->padded_;
5585:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5586:   return obj.release().ptr();
5587:   END_HANDLE_TH_ERRORS
5588: }
5589: 
5590: static struct PyGetSetDef NestedFromPaddedBackward0_properties[] = {
5591:   THP_FUNCTION_DEFAULT_PROPERTIES,
5592:   {(char*)"_saved_fuse_transform_0213", (getter)THPNestedFromPaddedBackward0_fuse_transform_0213_getter, nullptr, nullptr, nullptr},
5593:   {(char*)"_saved_padded", (getter)THPNestedFromPaddedBackward0_padded_getter, nullptr, nullptr, nullptr},
5594:   {(char*)"_raw_saved_padded", (getter)THPNestedFromPaddedBackward0_padded_raw_getter, nullptr, nullptr, nullptr},
5595:   {nullptr} /* sentinel */
5596: };
5597: 
5598: 
5599: 
5600: static struct PyGetSetDef NestedViewFromBufferBackward0_properties[] = {
```

- EN: The main execution path in this span is carried by `THPMiopenCtcLossBackward1_result0_getter`, `THPVariable_Wrap`, `THPMiopenCtcLossBackward1_result0_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPMiopenCtcLossBackward1_result0_getter`, `THPVariable_Wrap`, `THPMiopenCtcLossBackward1_result0_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5601-5680

```cpp
5601:   THP_FUNCTION_DEFAULT_PROPERTIES,
5602: 
5603:   {nullptr} /* sentinel */
5604: };
5605: 
5606: 
5607: 
5608: static struct PyGetSetDef NestedViewFromJaggedBackward0_properties[] = {
5609:   THP_FUNCTION_DEFAULT_PROPERTIES,
5610: 
5611:   {nullptr} /* sentinel */
5612: };
5613: 
5614: static PyObject* THPSafeSoftmaxBackward0_dim_getter(THPCppFunction *self, void *_unused) {
5615:   HANDLE_TH_ERRORS
5616:   auto prop = static_cast<SafeSoftmaxBackward0*>(self->cdata.get())->dim;
5617:   return PyLong_FromUnsignedLong((int64_t) prop);
5618:   END_HANDLE_TH_ERRORS
5619: }
5620: 
5621: static PyObject* THPSafeSoftmaxBackward0_result_getter(THPCppFunction *self, void *_unused) {
5622:   HANDLE_TH_ERRORS
5623:   const auto& prop = static_cast<SafeSoftmaxBackward0*>(self->cdata.get())->result_;
5624:   return THPVariable_Wrap(prop.unpack(self->cdata));
5625:   END_HANDLE_TH_ERRORS
5626: }
5627: 
5628: static PyObject* THPSafeSoftmaxBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
5629:   HANDLE_TH_ERRORS
5630:   const auto& prop = static_cast<SafeSoftmaxBackward0*>(self->cdata.get())->result_;
5631:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5632:   return obj.release().ptr();
5633:   END_HANDLE_TH_ERRORS
5634: }
5635: 
5636: static struct PyGetSetDef SafeSoftmaxBackward0_properties[] = {
5637:   THP_FUNCTION_DEFAULT_PROPERTIES,
5638:   {(char*)"_saved_dim", (getter)THPSafeSoftmaxBackward0_dim_getter, nullptr, nullptr, nullptr},
5639:   {(char*)"_saved_result", (getter)THPSafeSoftmaxBackward0_result_getter, nullptr, nullptr, nullptr},
5640:   {(char*)"_raw_saved_result", (getter)THPSafeSoftmaxBackward0_result_raw_getter, nullptr, nullptr, nullptr},
5641:   {nullptr} /* sentinel */
5642: };
5643: 
5644: static PyObject* THPScaledDotProductEfficientAttentionBackward0_attn_bias_getter(THPCppFunction *self, void *_unused) {
5645:   HANDLE_TH_ERRORS
5646:   const auto& prop = static_cast<ScaledDotProductEfficientAttentionBackward0*>(self->cdata.get())->attn_bias_;
5647:   return THPVariable_Wrap(prop.unpack(self->cdata));
5648:   END_HANDLE_TH_ERRORS
5649: }
5650: 
5651: static PyObject* THPScaledDotProductEfficientAttentionBackward0_attn_bias_raw_getter(THPCppFunction *self, void *_unused) {
5652:   HANDLE_TH_ERRORS
5653:   const auto& prop = static_cast<ScaledDotProductEfficientAttentionBackward0*>(self->cdata.get())->attn_bias_;
5654:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5655:   return obj.release().ptr();
5656:   END_HANDLE_TH_ERRORS
5657: }
5658: 
5659: static PyObject* THPScaledDotProductEfficientAttentionBackward0_dropout_p_getter(THPCppFunction *self, void *_unused) {
5660:   HANDLE_TH_ERRORS
5661:   auto prop = static_cast<ScaledDotProductEfficientAttentionBackward0*>(self->cdata.get())->dropout_p;
5662:   return PyFloat_FromDouble((double) prop);
5663:   END_HANDLE_TH_ERRORS
5664: }
5665: 
5666: static PyObject* THPScaledDotProductEfficientAttentionBackward0_is_causal_getter(THPCppFunction *self, void *_unused) {
5667:   HANDLE_TH_ERRORS
5668:   auto prop = static_cast<ScaledDotProductEfficientAttentionBackward0*>(self->cdata.get())->is_causal;
5669:   if (prop) {
5670:     Py_RETURN_TRUE;
5671:   } else {
5672:     Py_RETURN_FALSE;
5673:   }
5674:   END_HANDLE_TH_ERRORS
5675: }
5676: 
5677: static PyObject* THPScaledDotProductEfficientAttentionBackward0_key_getter(THPCppFunction *self, void *_unused) {
5678:   HANDLE_TH_ERRORS
5679:   const auto& prop = static_cast<ScaledDotProductEfficientAttentionBackward0*>(self->cdata.get())->key_;
5680:   return THPVariable_Wrap(prop.unpack(self->cdata));
```

- EN: The main execution path in this span is carried by `THPSafeSoftmaxBackward0_dim_getter`, `PyLong_FromUnsignedLong`, `THPSafeSoftmaxBackward0_result_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPSafeSoftmaxBackward0_dim_getter`, `PyLong_FromUnsignedLong`, `THPSafeSoftmaxBackward0_result_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5681-5760

```cpp
5681:   END_HANDLE_TH_ERRORS
5682: }
5683: 
5684: static PyObject* THPScaledDotProductEfficientAttentionBackward0_key_raw_getter(THPCppFunction *self, void *_unused) {
5685:   HANDLE_TH_ERRORS
5686:   const auto& prop = static_cast<ScaledDotProductEfficientAttentionBackward0*>(self->cdata.get())->key_;
5687:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5688:   return obj.release().ptr();
5689:   END_HANDLE_TH_ERRORS
5690: }
5691: 
5692: static PyObject* THPScaledDotProductEfficientAttentionBackward0_query_getter(THPCppFunction *self, void *_unused) {
5693:   HANDLE_TH_ERRORS
5694:   const auto& prop = static_cast<ScaledDotProductEfficientAttentionBackward0*>(self->cdata.get())->query_;
5695:   return THPVariable_Wrap(prop.unpack(self->cdata));
5696:   END_HANDLE_TH_ERRORS
5697: }
5698: 
5699: static PyObject* THPScaledDotProductEfficientAttentionBackward0_query_raw_getter(THPCppFunction *self, void *_unused) {
5700:   HANDLE_TH_ERRORS
5701:   const auto& prop = static_cast<ScaledDotProductEfficientAttentionBackward0*>(self->cdata.get())->query_;
5702:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5703:   return obj.release().ptr();
5704:   END_HANDLE_TH_ERRORS
5705: }
5706: 
5707: static PyObject* THPScaledDotProductEfficientAttentionBackward0_scale_getter(THPCppFunction *self, void *_unused) {
5708:   HANDLE_TH_ERRORS
5709:   auto opt_prop = static_cast<ScaledDotProductEfficientAttentionBackward0*>(self->cdata.get())->scale;
5710:   if (!opt_prop.has_value()) {
5711:     Py_RETURN_NONE;
5712:   }
5713:   auto prop = opt_prop.value();
5714:   return PyFloat_FromDouble((double) prop);
5715:   END_HANDLE_TH_ERRORS
5716: }
5717: 
5718: static PyObject* THPScaledDotProductEfficientAttentionBackward0_value_getter(THPCppFunction *self, void *_unused) {
5719:   HANDLE_TH_ERRORS
5720:   const auto& prop = static_cast<ScaledDotProductEfficientAttentionBackward0*>(self->cdata.get())->value_;
5721:   return THPVariable_Wrap(prop.unpack(self->cdata));
5722:   END_HANDLE_TH_ERRORS
5723: }
5724: 
5725: static PyObject* THPScaledDotProductEfficientAttentionBackward0_value_raw_getter(THPCppFunction *self, void *_unused) {
5726:   HANDLE_TH_ERRORS
5727:   const auto& prop = static_cast<ScaledDotProductEfficientAttentionBackward0*>(self->cdata.get())->value_;
5728:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5729:   return obj.release().ptr();
5730:   END_HANDLE_TH_ERRORS
5731: }
5732: 
5733: static PyObject* THPScaledDotProductEfficientAttentionBackward0_log_sumexp_getter(THPCppFunction *self, void *_unused) {
5734:   HANDLE_TH_ERRORS
5735:   const auto& prop = static_cast<ScaledDotProductEfficientAttentionBackward0*>(self->cdata.get())->log_sumexp_;
5736:   return THPVariable_Wrap(prop.unpack(self->cdata));
5737:   END_HANDLE_TH_ERRORS
5738: }
5739: 
5740: static PyObject* THPScaledDotProductEfficientAttentionBackward0_log_sumexp_raw_getter(THPCppFunction *self, void *_unused) {
5741:   HANDLE_TH_ERRORS
5742:   const auto& prop = static_cast<ScaledDotProductEfficientAttentionBackward0*>(self->cdata.get())->log_sumexp_;
5743:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5744:   return obj.release().ptr();
5745:   END_HANDLE_TH_ERRORS
5746: }
5747: 
5748: static PyObject* THPScaledDotProductEfficientAttentionBackward0_output_getter(THPCppFunction *self, void *_unused) {
5749:   HANDLE_TH_ERRORS
5750:   const auto& prop = static_cast<ScaledDotProductEfficientAttentionBackward0*>(self->cdata.get())->output_;
5751:   return THPVariable_Wrap(prop.unpack(self->cdata));
5752:   END_HANDLE_TH_ERRORS
5753: }
5754: 
5755: static PyObject* THPScaledDotProductEfficientAttentionBackward0_output_raw_getter(THPCppFunction *self, void *_unused) {
5756:   HANDLE_TH_ERRORS
5757:   const auto& prop = static_cast<ScaledDotProductEfficientAttentionBackward0*>(self->cdata.get())->output_;
5758:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5759:   return obj.release().ptr();
5760:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPScaledDotProductEfficientAttentionBackward0_key_raw_getter`, `cast`, `THPScaledDotProductEfficientAttentionBackward0_query_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPScaledDotProductEfficientAttentionBackward0_key_raw_getter`, `cast`, `THPScaledDotProductEfficientAttentionBackward0_query_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5761-5840

```cpp
5761: }
5762: 
5763: static PyObject* THPScaledDotProductEfficientAttentionBackward0_philox_offset_getter(THPCppFunction *self, void *_unused) {
5764:   HANDLE_TH_ERRORS
5765:   const auto& prop = static_cast<ScaledDotProductEfficientAttentionBackward0*>(self->cdata.get())->philox_offset_;
5766:   return THPVariable_Wrap(prop.unpack(self->cdata));
5767:   END_HANDLE_TH_ERRORS
5768: }
5769: 
5770: static PyObject* THPScaledDotProductEfficientAttentionBackward0_philox_offset_raw_getter(THPCppFunction *self, void *_unused) {
5771:   HANDLE_TH_ERRORS
5772:   const auto& prop = static_cast<ScaledDotProductEfficientAttentionBackward0*>(self->cdata.get())->philox_offset_;
5773:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5774:   return obj.release().ptr();
5775:   END_HANDLE_TH_ERRORS
5776: }
5777: 
5778: static PyObject* THPScaledDotProductEfficientAttentionBackward0_philox_seed_getter(THPCppFunction *self, void *_unused) {
5779:   HANDLE_TH_ERRORS
5780:   const auto& prop = static_cast<ScaledDotProductEfficientAttentionBackward0*>(self->cdata.get())->philox_seed_;
5781:   return THPVariable_Wrap(prop.unpack(self->cdata));
5782:   END_HANDLE_TH_ERRORS
5783: }
5784: 
5785: static PyObject* THPScaledDotProductEfficientAttentionBackward0_philox_seed_raw_getter(THPCppFunction *self, void *_unused) {
5786:   HANDLE_TH_ERRORS
5787:   const auto& prop = static_cast<ScaledDotProductEfficientAttentionBackward0*>(self->cdata.get())->philox_seed_;
5788:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5789:   return obj.release().ptr();
5790:   END_HANDLE_TH_ERRORS
5791: }
5792: 
5793: static struct PyGetSetDef ScaledDotProductEfficientAttentionBackward0_properties[] = {
5794:   THP_FUNCTION_DEFAULT_PROPERTIES,
5795:   {(char*)"_saved_attn_bias", (getter)THPScaledDotProductEfficientAttentionBackward0_attn_bias_getter, nullptr, nullptr, nullptr},
5796:   {(char*)"_raw_saved_attn_bias", (getter)THPScaledDotProductEfficientAttentionBackward0_attn_bias_raw_getter, nullptr, nullptr, nullptr},
5797:   {(char*)"_saved_dropout_p", (getter)THPScaledDotProductEfficientAttentionBackward0_dropout_p_getter, nullptr, nullptr, nullptr},
5798:   {(char*)"_saved_is_causal", (getter)THPScaledDotProductEfficientAttentionBackward0_is_causal_getter, nullptr, nullptr, nullptr},
5799:   {(char*)"_saved_key", (getter)THPScaledDotProductEfficientAttentionBackward0_key_getter, nullptr, nullptr, nullptr},
5800:   {(char*)"_raw_saved_key", (getter)THPScaledDotProductEfficientAttentionBackward0_key_raw_getter, nullptr, nullptr, nullptr},
5801:   {(char*)"_saved_query", (getter)THPScaledDotProductEfficientAttentionBackward0_query_getter, nullptr, nullptr, nullptr},
5802:   {(char*)"_raw_saved_query", (getter)THPScaledDotProductEfficientAttentionBackward0_query_raw_getter, nullptr, nullptr, nullptr},
5803:   {(char*)"_saved_scale", (getter)THPScaledDotProductEfficientAttentionBackward0_scale_getter, nullptr, nullptr, nullptr},
5804:   {(char*)"_saved_value", (getter)THPScaledDotProductEfficientAttentionBackward0_value_getter, nullptr, nullptr, nullptr},
5805:   {(char*)"_raw_saved_value", (getter)THPScaledDotProductEfficientAttentionBackward0_value_raw_getter, nullptr, nullptr, nullptr},
5806:   {(char*)"_saved_log_sumexp", (getter)THPScaledDotProductEfficientAttentionBackward0_log_sumexp_getter, nullptr, nullptr, nullptr},
5807:   {(char*)"_raw_saved_log_sumexp", (getter)THPScaledDotProductEfficientAttentionBackward0_log_sumexp_raw_getter, nullptr, nullptr, nullptr},
5808:   {(char*)"_saved_output", (getter)THPScaledDotProductEfficientAttentionBackward0_output_getter, nullptr, nullptr, nullptr},
5809:   {(char*)"_raw_saved_output", (getter)THPScaledDotProductEfficientAttentionBackward0_output_raw_getter, nullptr, nullptr, nullptr},
5810:   {(char*)"_saved_philox_offset", (getter)THPScaledDotProductEfficientAttentionBackward0_philox_offset_getter, nullptr, nullptr, nullptr},
5811:   {(char*)"_raw_saved_philox_offset", (getter)THPScaledDotProductEfficientAttentionBackward0_philox_offset_raw_getter, nullptr, nullptr, nullptr},
5812:   {(char*)"_saved_philox_seed", (getter)THPScaledDotProductEfficientAttentionBackward0_philox_seed_getter, nullptr, nullptr, nullptr},
5813:   {(char*)"_raw_saved_philox_seed", (getter)THPScaledDotProductEfficientAttentionBackward0_philox_seed_raw_getter, nullptr, nullptr, nullptr},
5814:   {nullptr} /* sentinel */
5815: };
5816: 
5817: static PyObject* THPEfficientAttentionBackward0_bias_getter(THPCppFunction *self, void *_unused) {
5818:   HANDLE_TH_ERRORS
5819:   const auto& prop = static_cast<EfficientAttentionBackward0*>(self->cdata.get())->bias_;
5820:   return THPVariable_Wrap(prop.unpack(self->cdata));
5821:   END_HANDLE_TH_ERRORS
5822: }
5823: 
5824: static PyObject* THPEfficientAttentionBackward0_bias_raw_getter(THPCppFunction *self, void *_unused) {
5825:   HANDLE_TH_ERRORS
5826:   const auto& prop = static_cast<EfficientAttentionBackward0*>(self->cdata.get())->bias_;
5827:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5828:   return obj.release().ptr();
5829:   END_HANDLE_TH_ERRORS
5830: }
5831: 
5832: static PyObject* THPEfficientAttentionBackward0_cu_seqlens_k_getter(THPCppFunction *self, void *_unused) {
5833:   HANDLE_TH_ERRORS
5834:   const auto& prop = static_cast<EfficientAttentionBackward0*>(self->cdata.get())->cu_seqlens_k_;
5835:   return THPVariable_Wrap(prop.unpack(self->cdata));
5836:   END_HANDLE_TH_ERRORS
5837: }
5838: 
5839: static PyObject* THPEfficientAttentionBackward0_cu_seqlens_k_raw_getter(THPCppFunction *self, void *_unused) {
5840:   HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPScaledDotProductEfficientAttentionBackward0_philox_offset_getter`, `THPVariable_Wrap`, `THPScaledDotProductEfficientAttentionBackward0_philox_offset_raw_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPScaledDotProductEfficientAttentionBackward0_philox_offset_getter`, `THPVariable_Wrap`, `THPScaledDotProductEfficientAttentionBackward0_philox_offset_raw_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5841-5920

```cpp
5841:   const auto& prop = static_cast<EfficientAttentionBackward0*>(self->cdata.get())->cu_seqlens_k_;
5842:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5843:   return obj.release().ptr();
5844:   END_HANDLE_TH_ERRORS
5845: }
5846: 
5847: static PyObject* THPEfficientAttentionBackward0_cu_seqlens_q_getter(THPCppFunction *self, void *_unused) {
5848:   HANDLE_TH_ERRORS
5849:   const auto& prop = static_cast<EfficientAttentionBackward0*>(self->cdata.get())->cu_seqlens_q_;
5850:   return THPVariable_Wrap(prop.unpack(self->cdata));
5851:   END_HANDLE_TH_ERRORS
5852: }
5853: 
5854: static PyObject* THPEfficientAttentionBackward0_cu_seqlens_q_raw_getter(THPCppFunction *self, void *_unused) {
5855:   HANDLE_TH_ERRORS
5856:   const auto& prop = static_cast<EfficientAttentionBackward0*>(self->cdata.get())->cu_seqlens_q_;
5857:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5858:   return obj.release().ptr();
5859:   END_HANDLE_TH_ERRORS
5860: }
5861: 
5862: static PyObject* THPEfficientAttentionBackward0_custom_mask_type_getter(THPCppFunction *self, void *_unused) {
5863:   HANDLE_TH_ERRORS
5864:   auto prop = static_cast<EfficientAttentionBackward0*>(self->cdata.get())->custom_mask_type;
5865:   return PyLong_FromUnsignedLong((int64_t) prop);
5866:   END_HANDLE_TH_ERRORS
5867: }
5868: 
5869: static PyObject* THPEfficientAttentionBackward0_dropout_p_getter(THPCppFunction *self, void *_unused) {
5870:   HANDLE_TH_ERRORS
5871:   auto prop = static_cast<EfficientAttentionBackward0*>(self->cdata.get())->dropout_p;
5872:   return PyFloat_FromDouble((double) prop);
5873:   END_HANDLE_TH_ERRORS
5874: }
5875: 
5876: static PyObject* THPEfficientAttentionBackward0_key_getter(THPCppFunction *self, void *_unused) {
5877:   HANDLE_TH_ERRORS
5878:   const auto& prop = static_cast<EfficientAttentionBackward0*>(self->cdata.get())->key_;
5879:   return THPVariable_Wrap(prop.unpack(self->cdata));
5880:   END_HANDLE_TH_ERRORS
5881: }
5882: 
5883: static PyObject* THPEfficientAttentionBackward0_key_raw_getter(THPCppFunction *self, void *_unused) {
5884:   HANDLE_TH_ERRORS
5885:   const auto& prop = static_cast<EfficientAttentionBackward0*>(self->cdata.get())->key_;
5886:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5887:   return obj.release().ptr();
5888:   END_HANDLE_TH_ERRORS
5889: }
5890: 
5891: static PyObject* THPEfficientAttentionBackward0_query_getter(THPCppFunction *self, void *_unused) {
5892:   HANDLE_TH_ERRORS
5893:   const auto& prop = static_cast<EfficientAttentionBackward0*>(self->cdata.get())->query_;
5894:   return THPVariable_Wrap(prop.unpack(self->cdata));
5895:   END_HANDLE_TH_ERRORS
5896: }
5897: 
5898: static PyObject* THPEfficientAttentionBackward0_query_raw_getter(THPCppFunction *self, void *_unused) {
5899:   HANDLE_TH_ERRORS
5900:   const auto& prop = static_cast<EfficientAttentionBackward0*>(self->cdata.get())->query_;
5901:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5902:   return obj.release().ptr();
5903:   END_HANDLE_TH_ERRORS
5904: }
5905: 
5906: static PyObject* THPEfficientAttentionBackward0_scale_getter(THPCppFunction *self, void *_unused) {
5907:   HANDLE_TH_ERRORS
5908:   auto opt_prop = static_cast<EfficientAttentionBackward0*>(self->cdata.get())->scale;
5909:   if (!opt_prop.has_value()) {
5910:     Py_RETURN_NONE;
5911:   }
5912:   auto prop = opt_prop.value();
5913:   return PyFloat_FromDouble((double) prop);
5914:   END_HANDLE_TH_ERRORS
5915: }
5916: 
5917: static PyObject* THPEfficientAttentionBackward0_value_getter(THPCppFunction *self, void *_unused) {
5918:   HANDLE_TH_ERRORS
5919:   const auto& prop = static_cast<EfficientAttentionBackward0*>(self->cdata.get())->value_;
5920:   return THPVariable_Wrap(prop.unpack(self->cdata));
```

- EN: The main execution path in this span is carried by `cast`, `THPEfficientAttentionBackward0_cu_seqlens_q_getter`, `THPVariable_Wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `cast`, `THPEfficientAttentionBackward0_cu_seqlens_q_getter`, `THPVariable_Wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5921-6000

```cpp
5921:   END_HANDLE_TH_ERRORS
5922: }
5923: 
5924: static PyObject* THPEfficientAttentionBackward0_value_raw_getter(THPCppFunction *self, void *_unused) {
5925:   HANDLE_TH_ERRORS
5926:   const auto& prop = static_cast<EfficientAttentionBackward0*>(self->cdata.get())->value_;
5927:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5928:   return obj.release().ptr();
5929:   END_HANDLE_TH_ERRORS
5930: }
5931: 
5932: static PyObject* THPEfficientAttentionBackward0_logsumexp_getter(THPCppFunction *self, void *_unused) {
5933:   HANDLE_TH_ERRORS
5934:   const auto& prop = static_cast<EfficientAttentionBackward0*>(self->cdata.get())->logsumexp_;
5935:   return THPVariable_Wrap(prop.unpack(self->cdata));
5936:   END_HANDLE_TH_ERRORS
5937: }
5938: 
5939: static PyObject* THPEfficientAttentionBackward0_logsumexp_raw_getter(THPCppFunction *self, void *_unused) {
5940:   HANDLE_TH_ERRORS
5941:   const auto& prop = static_cast<EfficientAttentionBackward0*>(self->cdata.get())->logsumexp_;
5942:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5943:   return obj.release().ptr();
5944:   END_HANDLE_TH_ERRORS
5945: }
5946: 
5947: static PyObject* THPEfficientAttentionBackward0_max_seqlen_batch_k_getter(THPCppFunction *self, void *_unused) {
5948:   HANDLE_TH_ERRORS
5949:   auto prop = static_cast<EfficientAttentionBackward0*>(self->cdata.get())->max_seqlen_batch_k;
5950:   if (auto m = prop.maybe_as_int()) {
5951:     return PyLong_FromUnsignedLong(*m);
5952:   } else {
5953:     return py::cast(prop).release().ptr();
5954:   }
5955:   END_HANDLE_TH_ERRORS
5956: }
5957: 
5958: static PyObject* THPEfficientAttentionBackward0_max_seqlen_batch_q_getter(THPCppFunction *self, void *_unused) {
5959:   HANDLE_TH_ERRORS
5960:   auto prop = static_cast<EfficientAttentionBackward0*>(self->cdata.get())->max_seqlen_batch_q;
5961:   if (auto m = prop.maybe_as_int()) {
5962:     return PyLong_FromUnsignedLong(*m);
5963:   } else {
5964:     return py::cast(prop).release().ptr();
5965:   }
5966:   END_HANDLE_TH_ERRORS
5967: }
5968: 
5969: static PyObject* THPEfficientAttentionBackward0_output_getter(THPCppFunction *self, void *_unused) {
5970:   HANDLE_TH_ERRORS
5971:   const auto& prop = static_cast<EfficientAttentionBackward0*>(self->cdata.get())->output_;
5972:   return THPVariable_Wrap(prop.unpack(self->cdata));
5973:   END_HANDLE_TH_ERRORS
5974: }
5975: 
5976: static PyObject* THPEfficientAttentionBackward0_output_raw_getter(THPCppFunction *self, void *_unused) {
5977:   HANDLE_TH_ERRORS
5978:   const auto& prop = static_cast<EfficientAttentionBackward0*>(self->cdata.get())->output_;
5979:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5980:   return obj.release().ptr();
5981:   END_HANDLE_TH_ERRORS
5982: }
5983: 
5984: static PyObject* THPEfficientAttentionBackward0_philox_offset_getter(THPCppFunction *self, void *_unused) {
5985:   HANDLE_TH_ERRORS
5986:   const auto& prop = static_cast<EfficientAttentionBackward0*>(self->cdata.get())->philox_offset_;
5987:   return THPVariable_Wrap(prop.unpack(self->cdata));
5988:   END_HANDLE_TH_ERRORS
5989: }
5990: 
5991: static PyObject* THPEfficientAttentionBackward0_philox_offset_raw_getter(THPCppFunction *self, void *_unused) {
5992:   HANDLE_TH_ERRORS
5993:   const auto& prop = static_cast<EfficientAttentionBackward0*>(self->cdata.get())->philox_offset_;
5994:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5995:   return obj.release().ptr();
5996:   END_HANDLE_TH_ERRORS
5997: }
5998: 
5999: static PyObject* THPEfficientAttentionBackward0_philox_seed_getter(THPCppFunction *self, void *_unused) {
6000:   HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPEfficientAttentionBackward0_value_raw_getter`, `cast`, `THPEfficientAttentionBackward0_logsumexp_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPEfficientAttentionBackward0_value_raw_getter`, `cast`, `THPEfficientAttentionBackward0_logsumexp_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 6001-6080

```cpp
6001:   const auto& prop = static_cast<EfficientAttentionBackward0*>(self->cdata.get())->philox_seed_;
6002:   return THPVariable_Wrap(prop.unpack(self->cdata));
6003:   END_HANDLE_TH_ERRORS
6004: }
6005: 
6006: static PyObject* THPEfficientAttentionBackward0_philox_seed_raw_getter(THPCppFunction *self, void *_unused) {
6007:   HANDLE_TH_ERRORS
6008:   const auto& prop = static_cast<EfficientAttentionBackward0*>(self->cdata.get())->philox_seed_;
6009:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
6010:   return obj.release().ptr();
6011:   END_HANDLE_TH_ERRORS
6012: }
6013: 
6014: static struct PyGetSetDef EfficientAttentionBackward0_properties[] = {
6015:   THP_FUNCTION_DEFAULT_PROPERTIES,
6016:   {(char*)"_saved_bias", (getter)THPEfficientAttentionBackward0_bias_getter, nullptr, nullptr, nullptr},
6017:   {(char*)"_raw_saved_bias", (getter)THPEfficientAttentionBackward0_bias_raw_getter, nullptr, nullptr, nullptr},
6018:   {(char*)"_saved_cu_seqlens_k", (getter)THPEfficientAttentionBackward0_cu_seqlens_k_getter, nullptr, nullptr, nullptr},
6019:   {(char*)"_raw_saved_cu_seqlens_k", (getter)THPEfficientAttentionBackward0_cu_seqlens_k_raw_getter, nullptr, nullptr, nullptr},
6020:   {(char*)"_saved_cu_seqlens_q", (getter)THPEfficientAttentionBackward0_cu_seqlens_q_getter, nullptr, nullptr, nullptr},
6021:   {(char*)"_raw_saved_cu_seqlens_q", (getter)THPEfficientAttentionBackward0_cu_seqlens_q_raw_getter, nullptr, nullptr, nullptr},
6022:   {(char*)"_saved_custom_mask_type", (getter)THPEfficientAttentionBackward0_custom_mask_type_getter, nullptr, nullptr, nullptr},
6023:   {(char*)"_saved_dropout_p", (getter)THPEfficientAttentionBackward0_dropout_p_getter, nullptr, nullptr, nullptr},
6024:   {(char*)"_saved_key", (getter)THPEfficientAttentionBackward0_key_getter, nullptr, nullptr, nullptr},
6025:   {(char*)"_raw_saved_key", (getter)THPEfficientAttentionBackward0_key_raw_getter, nullptr, nullptr, nullptr},
6026:   {(char*)"_saved_query", (getter)THPEfficientAttentionBackward0_query_getter, nullptr, nullptr, nullptr},
6027:   {(char*)"_raw_saved_query", (getter)THPEfficientAttentionBackward0_query_raw_getter, nullptr, nullptr, nullptr},
6028:   {(char*)"_saved_scale", (getter)THPEfficientAttentionBackward0_scale_getter, nullptr, nullptr, nullptr},
6029:   {(char*)"_saved_value", (getter)THPEfficientAttentionBackward0_value_getter, nullptr, nullptr, nullptr},
6030:   {(char*)"_raw_saved_value", (getter)THPEfficientAttentionBackward0_value_raw_getter, nullptr, nullptr, nullptr},
6031:   {(char*)"_saved_logsumexp", (getter)THPEfficientAttentionBackward0_logsumexp_getter, nullptr, nullptr, nullptr},
6032:   {(char*)"_raw_saved_logsumexp", (getter)THPEfficientAttentionBackward0_logsumexp_raw_getter, nullptr, nullptr, nullptr},
6033:   {(char*)"_saved_max_seqlen_batch_k", (getter)THPEfficientAttentionBackward0_max_seqlen_batch_k_getter, nullptr, nullptr, nullptr},
6034:   {(char*)"_saved_max_seqlen_batch_q", (getter)THPEfficientAttentionBackward0_max_seqlen_batch_q_getter, nullptr, nullptr, nullptr},
6035:   {(char*)"_saved_output", (getter)THPEfficientAttentionBackward0_output_getter, nullptr, nullptr, nullptr},
6036:   {(char*)"_raw_saved_output", (getter)THPEfficientAttentionBackward0_output_raw_getter, nullptr, nullptr, nullptr},
6037:   {(char*)"_saved_philox_offset", (getter)THPEfficientAttentionBackward0_philox_offset_getter, nullptr, nullptr, nullptr},
6038:   {(char*)"_raw_saved_philox_offset", (getter)THPEfficientAttentionBackward0_philox_offset_raw_getter, nullptr, nullptr, nullptr},
6039:   {(char*)"_saved_philox_seed", (getter)THPEfficientAttentionBackward0_philox_seed_getter, nullptr, nullptr, nullptr},
6040:   {(char*)"_raw_saved_philox_seed", (getter)THPEfficientAttentionBackward0_philox_seed_raw_getter, nullptr, nullptr, nullptr},
6041:   {nullptr} /* sentinel */
6042: };
6043: 
6044: static PyObject* THPFftR2CBackward0_dim_getter(THPCppFunction *self, void *_unused) {
6045:   HANDLE_TH_ERRORS
6046:   auto prop = static_cast<FftR2CBackward0*>(self->cdata.get())->dim;
6047:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
6048:   for (auto i : c10::irange(prop.size())) {
6049:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
6050:   }
6051:   return tup;
6052:   END_HANDLE_TH_ERRORS
6053: }
6054: 
6055: static PyObject* THPFftR2CBackward0_normalization_getter(THPCppFunction *self, void *_unused) {
6056:   HANDLE_TH_ERRORS
6057:   auto prop = static_cast<FftR2CBackward0*>(self->cdata.get())->normalization;
6058:   return PyLong_FromUnsignedLong((int64_t) prop);
6059:   END_HANDLE_TH_ERRORS
6060: }
6061: 
6062: static PyObject* THPFftR2CBackward0_onesided_getter(THPCppFunction *self, void *_unused) {
6063:   HANDLE_TH_ERRORS
6064:   auto prop = static_cast<FftR2CBackward0*>(self->cdata.get())->onesided;
6065:   if (prop) {
6066:     Py_RETURN_TRUE;
6067:   } else {
6068:     Py_RETURN_FALSE;
6069:   }
6070:   END_HANDLE_TH_ERRORS
6071: }
6072: 
6073: static PyObject* THPFftR2CBackward0_self_getter(THPCppFunction *self, void *_unused) {
6074:   HANDLE_TH_ERRORS
6075:   const auto& prop = static_cast<FftR2CBackward0*>(self->cdata.get())->self_;
6076:   return THPVariable_Wrap(prop.unpack(self->cdata));
6077:   END_HANDLE_TH_ERRORS
6078: }
6079: 
6080: static PyObject* THPFftR2CBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
```

- EN: The main execution path in this span is carried by `THPVariable_Wrap`, `THPEfficientAttentionBackward0_philox_seed_raw_getter`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_Wrap`, `THPEfficientAttentionBackward0_philox_seed_raw_getter`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 6081-6160

```cpp
6081:   HANDLE_TH_ERRORS
6082:   const auto& prop = static_cast<FftR2CBackward0*>(self->cdata.get())->self_;
6083:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
6084:   return obj.release().ptr();
6085:   END_HANDLE_TH_ERRORS
6086: }
6087: 
6088: static struct PyGetSetDef FftR2CBackward0_properties[] = {
6089:   THP_FUNCTION_DEFAULT_PROPERTIES,
6090:   {(char*)"_saved_dim", (getter)THPFftR2CBackward0_dim_getter, nullptr, nullptr, nullptr},
6091:   {(char*)"_saved_normalization", (getter)THPFftR2CBackward0_normalization_getter, nullptr, nullptr, nullptr},
6092:   {(char*)"_saved_onesided", (getter)THPFftR2CBackward0_onesided_getter, nullptr, nullptr, nullptr},
6093:   {(char*)"_saved_self", (getter)THPFftR2CBackward0_self_getter, nullptr, nullptr, nullptr},
6094:   {(char*)"_raw_saved_self", (getter)THPFftR2CBackward0_self_raw_getter, nullptr, nullptr, nullptr},
6095:   {nullptr} /* sentinel */
6096: };
6097: 
6098: static PyObject* THPThnnFusedGruCellBackward0_hidden_bias_getter(THPCppFunction *self, void *_unused) {
6099:   HANDLE_TH_ERRORS
6100:   const auto& prop = static_cast<ThnnFusedGruCellBackward0*>(self->cdata.get())->hidden_bias_;
6101:   return THPVariable_Wrap(prop.unpack(self->cdata));
6102:   END_HANDLE_TH_ERRORS
6103: }
6104: 
6105: static PyObject* THPThnnFusedGruCellBackward0_hidden_bias_raw_getter(THPCppFunction *self, void *_unused) {
6106:   HANDLE_TH_ERRORS
6107:   const auto& prop = static_cast<ThnnFusedGruCellBackward0*>(self->cdata.get())->hidden_bias_;
6108:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
6109:   return obj.release().ptr();
6110:   END_HANDLE_TH_ERRORS
6111: }
6112: 
6113: static PyObject* THPThnnFusedGruCellBackward0_hidden_gates_getter(THPCppFunction *self, void *_unused) {
6114:   HANDLE_TH_ERRORS
6115:   const auto& prop = static_cast<ThnnFusedGruCellBackward0*>(self->cdata.get())->hidden_gates_;
6116:   return THPVariable_Wrap(prop.unpack(self->cdata));
6117:   END_HANDLE_TH_ERRORS
6118: }
6119: 
6120: static PyObject* THPThnnFusedGruCellBackward0_hidden_gates_raw_getter(THPCppFunction *self, void *_unused) {
6121:   HANDLE_TH_ERRORS
6122:   const auto& prop = static_cast<ThnnFusedGruCellBackward0*>(self->cdata.get())->hidden_gates_;
6123:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
6124:   return obj.release().ptr();
6125:   END_HANDLE_TH_ERRORS
6126: }
6127: 
6128: static PyObject* THPThnnFusedGruCellBackward0_hx_getter(THPCppFunction *self, void *_unused) {
6129:   HANDLE_TH_ERRORS
6130:   const auto& prop = static_cast<ThnnFusedGruCellBackward0*>(self->cdata.get())->hx_;
6131:   return THPVariable_Wrap(prop.unpack(self->cdata));
6132:   END_HANDLE_TH_ERRORS
6133: }
6134: 
6135: static PyObject* THPThnnFusedGruCellBackward0_hx_raw_getter(THPCppFunction *self, void *_unused) {
6136:   HANDLE_TH_ERRORS
6137:   const auto& prop = static_cast<ThnnFusedGruCellBackward0*>(self->cdata.get())->hx_;
6138:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
6139:   return obj.release().ptr();
6140:   END_HANDLE_TH_ERRORS
6141: }
6142: 
6143: static PyObject* THPThnnFusedGruCellBackward0_input_bias_getter(THPCppFunction *self, void *_unused) {
6144:   HANDLE_TH_ERRORS
6145:   const auto& prop = static_cast<ThnnFusedGruCellBackward0*>(self->cdata.get())->input_bias_;
6146:   return THPVariable_Wrap(prop.unpack(self->cdata));
6147:   END_HANDLE_TH_ERRORS
6148: }
6149: 
6150: static PyObject* THPThnnFusedGruCellBackward0_input_bias_raw_getter(THPCppFunction *self, void *_unused) {
6151:   HANDLE_TH_ERRORS
6152:   const auto& prop = static_cast<ThnnFusedGruCellBackward0*>(self->cdata.get())->input_bias_;
6153:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
6154:   return obj.release().ptr();
6155:   END_HANDLE_TH_ERRORS
6156: }
6157: 
6158: static PyObject* THPThnnFusedGruCellBackward0_input_gates_getter(THPCppFunction *self, void *_unused) {
6159:   HANDLE_TH_ERRORS
6160:   const auto& prop = static_cast<ThnnFusedGruCellBackward0*>(self->cdata.get())->input_gates_;
```

- EN: The main execution path in this span is carried by `cast`, `THPThnnFusedGruCellBackward0_hidden_bias_getter`, `THPVariable_Wrap`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `cast`, `THPThnnFusedGruCellBackward0_hidden_bias_getter`, `THPVariable_Wrap` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 6161-6240

```cpp
6161:   return THPVariable_Wrap(prop.unpack(self->cdata));
6162:   END_HANDLE_TH_ERRORS
6163: }
6164: 
6165: static PyObject* THPThnnFusedGruCellBackward0_input_gates_raw_getter(THPCppFunction *self, void *_unused) {
6166:   HANDLE_TH_ERRORS
6167:   const auto& prop = static_cast<ThnnFusedGruCellBackward0*>(self->cdata.get())->input_gates_;
6168:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
6169:   return obj.release().ptr();
6170:   END_HANDLE_TH_ERRORS
6171: }
6172: 
6173: static PyObject* THPThnnFusedGruCellBackward0_result1_getter(THPCppFunction *self, void *_unused) {
6174:   HANDLE_TH_ERRORS
6175:   const auto& prop = static_cast<ThnnFusedGruCellBackward0*>(self->cdata.get())->result1_;
6176:   return THPVariable_Wrap(prop.unpack(self->cdata));
6177:   END_HANDLE_TH_ERRORS
6178: }
6179: 
6180: static PyObject* THPThnnFusedGruCellBackward0_result1_raw_getter(THPCppFunction *self, void *_unused) {
6181:   HANDLE_TH_ERRORS
6182:   const auto& prop = static_cast<ThnnFusedGruCellBackward0*>(self->cdata.get())->result1_;
6183:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
6184:   return obj.release().ptr();
6185:   END_HANDLE_TH_ERRORS
6186: }
6187: 
6188: static struct PyGetSetDef ThnnFusedGruCellBackward0_properties[] = {
6189:   THP_FUNCTION_DEFAULT_PROPERTIES,
6190:   {(char*)"_saved_hidden_bias", (getter)THPThnnFusedGruCellBackward0_hidden_bias_getter, nullptr, nullptr, nullptr},
6191:   {(char*)"_raw_saved_hidden_bias", (getter)THPThnnFusedGruCellBackward0_hidden_bias_raw_getter, nullptr, nullptr, nullptr},
6192:   {(char*)"_saved_hidden_gates", (getter)THPThnnFusedGruCellBackward0_hidden_gates_getter, nullptr, nullptr, nullptr},
6193:   {(char*)"_raw_saved_hidden_gates", (getter)THPThnnFusedGruCellBackward0_hidden_gates_raw_getter, nullptr, nullptr, nullptr},
6194:   {(char*)"_saved_hx", (getter)THPThnnFusedGruCellBackward0_hx_getter, nullptr, nullptr, nullptr},
6195:   {(char*)"_raw_saved_hx", (getter)THPThnnFusedGruCellBackward0_hx_raw_getter, nullptr, nullptr, nullptr},
6196:   {(char*)"_saved_input_bias", (getter)THPThnnFusedGruCellBackward0_input_bias_getter, nullptr, nullptr, nullptr},
6197:   {(char*)"_raw_saved_input_bias", (getter)THPThnnFusedGruCellBackward0_input_bias_raw_getter, nullptr, nullptr, nullptr},
6198:   {(char*)"_saved_input_gates", (getter)THPThnnFusedGruCellBackward0_input_gates_getter, nullptr, nullptr, nullptr},
6199:   {(char*)"_raw_saved_input_gates", (getter)THPThnnFusedGruCellBackward0_input_gates_raw_getter, nullptr, nullptr, nullptr},
6200:   {(char*)"_saved_result1", (getter)THPThnnFusedGruCellBackward0_result1_getter, nullptr, nullptr, nullptr},
6201:   {(char*)"_raw_saved_result1", (getter)THPThnnFusedGruCellBackward0_result1_raw_getter, nullptr, nullptr, nullptr},
6202:   {nullptr} /* sentinel */
6203: };
6204: 
6205: static PyObject* THPPackPaddedSequenceBackward0_batch_first_getter(THPCppFunction *self, void *_unused) {
6206:   HANDLE_TH_ERRORS
6207:   auto prop = static_cast<PackPaddedSequenceBackward0*>(self->cdata.get())->batch_first;
6208:   if (prop) {
6209:     Py_RETURN_TRUE;
6210:   } else {
6211:     Py_RETURN_FALSE;
6212:   }
6213:   END_HANDLE_TH_ERRORS
6214: }
6215: 
6216: static PyObject* THPPackPaddedSequenceBackward0_input_sym_sizes_getter(THPCppFunction *self, void *_unused) {
6217:   HANDLE_TH_ERRORS
6218:   auto prop = static_cast<PackPaddedSequenceBackward0*>(self->cdata.get())->input_sym_sizes;
6219:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
6220:   for (auto i : c10::irange(prop.size())) {
6221:       auto si = prop[i];
6222:       if (auto m = si.maybe_as_int()) {
6223:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
6224:       } else {
6225:         auto py_symint = py::cast(si).release().ptr();
6226:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
6227:       }
6228:   }
6229:   return tup;
6230:   END_HANDLE_TH_ERRORS
6231: }
6232: 
6233: static PyObject* THPPackPaddedSequenceBackward0_result1_getter(THPCppFunction *self, void *_unused) {
6234:   HANDLE_TH_ERRORS
6235:   const auto& prop = static_cast<PackPaddedSequenceBackward0*>(self->cdata.get())->result1_;
6236:   return THPVariable_Wrap(prop.unpack(self->cdata));
6237:   END_HANDLE_TH_ERRORS
6238: }
6239: 
6240: static PyObject* THPPackPaddedSequenceBackward0_result1_raw_getter(THPCppFunction *self, void *_unused) {
```

- EN: The main execution path in this span is carried by `THPVariable_Wrap`, `THPThnnFusedGruCellBackward0_input_gates_raw_getter`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_Wrap`, `THPThnnFusedGruCellBackward0_input_gates_raw_getter`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 6241-6320

```cpp
6241:   HANDLE_TH_ERRORS
6242:   const auto& prop = static_cast<PackPaddedSequenceBackward0*>(self->cdata.get())->result1_;
6243:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
6244:   return obj.release().ptr();
6245:   END_HANDLE_TH_ERRORS
6246: }
6247: 
6248: static struct PyGetSetDef PackPaddedSequenceBackward0_properties[] = {
6249:   THP_FUNCTION_DEFAULT_PROPERTIES,
6250:   {(char*)"_saved_batch_first", (getter)THPPackPaddedSequenceBackward0_batch_first_getter, nullptr, nullptr, nullptr},
6251:   {(char*)"_saved_input_sym_sizes", (getter)THPPackPaddedSequenceBackward0_input_sym_sizes_getter, nullptr, nullptr, nullptr},
6252:   {(char*)"_saved_result1", (getter)THPPackPaddedSequenceBackward0_result1_getter, nullptr, nullptr, nullptr},
6253:   {(char*)"_raw_saved_result1", (getter)THPPackPaddedSequenceBackward0_result1_raw_getter, nullptr, nullptr, nullptr},
6254:   {nullptr} /* sentinel */
6255: };
6256: 
6257: static PyObject* THPNarrowCopyBackward0_dim_getter(THPCppFunction *self, void *_unused) {
6258:   HANDLE_TH_ERRORS
6259:   auto prop = static_cast<NarrowCopyBackward0*>(self->cdata.get())->dim;
6260:   return PyLong_FromUnsignedLong((int64_t) prop);
6261:   END_HANDLE_TH_ERRORS
6262: }
6263: 
6264: static PyObject* THPNarrowCopyBackward0_length_getter(THPCppFunction *self, void *_unused) {
6265:   HANDLE_TH_ERRORS
6266:   auto prop = static_cast<NarrowCopyBackward0*>(self->cdata.get())->length;
6267:   if (auto m = prop.maybe_as_int()) {
6268:     return PyLong_FromUnsignedLong(*m);
6269:   } else {
6270:     return py::cast(prop).release().ptr();
6271:   }
6272:   END_HANDLE_TH_ERRORS
6273: }
6274: 
6275: static PyObject* THPNarrowCopyBackward0_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
6276:   HANDLE_TH_ERRORS
6277:   auto prop = static_cast<NarrowCopyBackward0*>(self->cdata.get())->self_sym_sizes;
6278:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
6279:   for (auto i : c10::irange(prop.size())) {
6280:       auto si = prop[i];
6281:       if (auto m = si.maybe_as_int()) {
6282:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
6283:       } else {
6284:         auto py_symint = py::cast(si).release().ptr();
6285:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
6286:       }
6287:   }
6288:   return tup;
6289:   END_HANDLE_TH_ERRORS
6290: }
6291: 
6292: static PyObject* THPNarrowCopyBackward0_start_getter(THPCppFunction *self, void *_unused) {
6293:   HANDLE_TH_ERRORS
6294:   auto prop = static_cast<NarrowCopyBackward0*>(self->cdata.get())->start;
6295:   if (auto m = prop.maybe_as_int()) {
6296:     return PyLong_FromUnsignedLong(*m);
6297:   } else {
6298:     return py::cast(prop).release().ptr();
6299:   }
6300:   END_HANDLE_TH_ERRORS
6301: }
6302: 
6303: static struct PyGetSetDef NarrowCopyBackward0_properties[] = {
6304:   THP_FUNCTION_DEFAULT_PROPERTIES,
6305:   {(char*)"_saved_dim", (getter)THPNarrowCopyBackward0_dim_getter, nullptr, nullptr, nullptr},
6306:   {(char*)"_saved_length", (getter)THPNarrowCopyBackward0_length_getter, nullptr, nullptr, nullptr},
6307:   {(char*)"_saved_self_sym_sizes", (getter)THPNarrowCopyBackward0_self_sym_sizes_getter, nullptr, nullptr, nullptr},
6308:   {(char*)"_saved_start", (getter)THPNarrowCopyBackward0_start_getter, nullptr, nullptr, nullptr},
6309:   {nullptr} /* sentinel */
6310: };
6311: 
6312: static PyObject* THPForeachDivBackward0_other_getter(THPCppFunction *self, void *_unused) {
6313:   HANDLE_TH_ERRORS
6314:   const auto *node = static_cast<ForeachDivBackward0*>(self->cdata.get());
6315:   const auto& prop = node->other_;
6316:   if (node->other_released_) {
6317:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
6318:     return nullptr;
6319:   }
6320:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
```

- EN: The main execution path in this span is carried by `cast`, `THPNarrowCopyBackward0_dim_getter`, `PyLong_FromUnsignedLong`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `cast`, `THPNarrowCopyBackward0_dim_getter`, `PyLong_FromUnsignedLong` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 6321-6400

```cpp
6321:   for (auto i: c10::irange(prop.size())) {
6322:     PyTuple_SetItem(tup, (Py_ssize_t) i, THPVariable_Wrap(prop[i].unpack(self->cdata)));
6323:   }
6324:   return tup;
6325:   END_HANDLE_TH_ERRORS
6326: }
6327: 
6328: static PyObject* THPForeachDivBackward0_other_raw_getter(THPCppFunction *self, void *_unused) {
6329:   HANDLE_TH_ERRORS
6330:   const auto *node = static_cast<ForeachDivBackward0*>(self->cdata.get());
6331:   const auto& prop = node->other_;
6332:   if (node->other_released_) {
6333:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
6334:     return nullptr;
6335:   }
6336:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
6337:   for (auto i : c10::irange(prop.size())) {
6338:     pybind11::object obj = pybind11::cast(prop[i], pybind11::return_value_policy::reference);
6339:     PyTuple_SetItem(tup, (Py_ssize_t) i, obj.release().ptr());
6340:   }
6341:   return tup;
6342:   END_HANDLE_TH_ERRORS
6343: }
6344: 
6345: static PyObject* THPForeachDivBackward0_self_getter(THPCppFunction *self, void *_unused) {
6346:   HANDLE_TH_ERRORS
6347:   const auto *node = static_cast<ForeachDivBackward0*>(self->cdata.get());
6348:   const auto& prop = node->self_;
6349:   if (node->self_released_) {
6350:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
6351:     return nullptr;
6352:   }
6353:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
6354:   for (auto i: c10::irange(prop.size())) {
6355:     PyTuple_SetItem(tup, (Py_ssize_t) i, THPVariable_Wrap(prop[i].unpack(self->cdata)));
6356:   }
6357:   return tup;
6358:   END_HANDLE_TH_ERRORS
6359: }
6360: 
6361: static PyObject* THPForeachDivBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
6362:   HANDLE_TH_ERRORS
6363:   const auto *node = static_cast<ForeachDivBackward0*>(self->cdata.get());
6364:   const auto& prop = node->self_;
6365:   if (node->self_released_) {
6366:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
6367:     return nullptr;
6368:   }
6369:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
6370:   for (auto i : c10::irange(prop.size())) {
6371:     pybind11::object obj = pybind11::cast(prop[i], pybind11::return_value_policy::reference);
6372:     PyTuple_SetItem(tup, (Py_ssize_t) i, obj.release().ptr());
6373:   }
6374:   return tup;
6375:   END_HANDLE_TH_ERRORS
6376: }
6377: 
6378: static struct PyGetSetDef ForeachDivBackward0_properties[] = {
6379:   THP_FUNCTION_DEFAULT_PROPERTIES,
6380:   {(char*)"_saved_other", (getter)THPForeachDivBackward0_other_getter, nullptr, nullptr, nullptr},
6381:   {(char*)"_raw_saved_other", (getter)THPForeachDivBackward0_other_raw_getter, nullptr, nullptr, nullptr},
6382:   {(char*)"_saved_self", (getter)THPForeachDivBackward0_self_getter, nullptr, nullptr, nullptr},
6383:   {(char*)"_raw_saved_self", (getter)THPForeachDivBackward0_self_raw_getter, nullptr, nullptr, nullptr},
6384:   {nullptr} /* sentinel */
6385: };
6386: 
6387: static PyObject* THPSelectBackward0_copy_dim_getter(THPCppFunction *self, void *_unused) {
6388:   HANDLE_TH_ERRORS
6389:   auto prop = static_cast<SelectBackward0_copy*>(self->cdata.get())->dim;
6390:   return PyLong_FromUnsignedLong((int64_t) prop);
6391:   END_HANDLE_TH_ERRORS
6392: }
6393: 
6394: static PyObject* THPSelectBackward0_copy_index_getter(THPCppFunction *self, void *_unused) {
6395:   HANDLE_TH_ERRORS
6396:   auto prop = static_cast<SelectBackward0_copy*>(self->cdata.get())->index;
6397:   if (auto m = prop.maybe_as_int()) {
6398:     return PyLong_FromUnsignedLong(*m);
6399:   } else {
6400:     return py::cast(prop).release().ptr();
```

- EN: The main execution path in this span is carried by `PyTuple_SetItem`, `THPForeachDivBackward0_other_raw_getter`, `PyErr_SetString`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyTuple_SetItem`, `THPForeachDivBackward0_other_raw_getter`, `PyErr_SetString` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 6401-6480

```cpp
6401:   }
6402:   END_HANDLE_TH_ERRORS
6403: }
6404: 
6405: static PyObject* THPSelectBackward0_copy_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
6406:   HANDLE_TH_ERRORS
6407:   auto prop = static_cast<SelectBackward0_copy*>(self->cdata.get())->self_sym_sizes;
6408:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
6409:   for (auto i : c10::irange(prop.size())) {
6410:       auto si = prop[i];
6411:       if (auto m = si.maybe_as_int()) {
6412:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
6413:       } else {
6414:         auto py_symint = py::cast(si).release().ptr();
6415:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
6416:       }
6417:   }
6418:   return tup;
6419:   END_HANDLE_TH_ERRORS
6420: }
6421: 
6422: static struct PyGetSetDef SelectBackward0_copy_properties[] = {
6423:   THP_FUNCTION_DEFAULT_PROPERTIES,
6424:   {(char*)"_saved_dim", (getter)THPSelectBackward0_copy_dim_getter, nullptr, nullptr, nullptr},
6425:   {(char*)"_saved_index", (getter)THPSelectBackward0_copy_index_getter, nullptr, nullptr, nullptr},
6426:   {(char*)"_saved_self_sym_sizes", (getter)THPSelectBackward0_copy_self_sym_sizes_getter, nullptr, nullptr, nullptr},
6427:   {nullptr} /* sentinel */
6428: };
6429: 
6430: static PyObject* THPSelectBackwardAutogradNestedTensor0_copy_dim_getter(THPCppFunction *self, void *_unused) {
6431:   HANDLE_TH_ERRORS
6432:   auto prop = static_cast<SelectBackwardAutogradNestedTensor0_copy*>(self->cdata.get())->dim;
6433:   return PyLong_FromUnsignedLong((int64_t) prop);
6434:   END_HANDLE_TH_ERRORS
6435: }
6436: 
6437: static PyObject* THPSelectBackwardAutogradNestedTensor0_copy_index_getter(THPCppFunction *self, void *_unused) {
6438:   HANDLE_TH_ERRORS
6439:   auto prop = static_cast<SelectBackwardAutogradNestedTensor0_copy*>(self->cdata.get())->index;
6440:   if (auto m = prop.maybe_as_int()) {
6441:     return PyLong_FromUnsignedLong(*m);
6442:   } else {
6443:     return py::cast(prop).release().ptr();
6444:   }
6445:   END_HANDLE_TH_ERRORS
6446: }
6447: 
6448: static PyObject* THPSelectBackwardAutogradNestedTensor0_copy_self_getter(THPCppFunction *self, void *_unused) {
6449:   HANDLE_TH_ERRORS
6450:   const auto& prop = static_cast<SelectBackwardAutogradNestedTensor0_copy*>(self->cdata.get())->self_;
6451:   return THPVariable_Wrap(prop.unpack(self->cdata));
6452:   END_HANDLE_TH_ERRORS
6453: }
6454: 
6455: static PyObject* THPSelectBackwardAutogradNestedTensor0_copy_self_raw_getter(THPCppFunction *self, void *_unused) {
6456:   HANDLE_TH_ERRORS
6457:   const auto& prop = static_cast<SelectBackwardAutogradNestedTensor0_copy*>(self->cdata.get())->self_;
6458:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
6459:   return obj.release().ptr();
6460:   END_HANDLE_TH_ERRORS
6461: }
6462: 
6463: static struct PyGetSetDef SelectBackwardAutogradNestedTensor0_copy_properties[] = {
6464:   THP_FUNCTION_DEFAULT_PROPERTIES,
6465:   {(char*)"_saved_dim", (getter)THPSelectBackwardAutogradNestedTensor0_copy_dim_getter, nullptr, nullptr, nullptr},
6466:   {(char*)"_saved_index", (getter)THPSelectBackwardAutogradNestedTensor0_copy_index_getter, nullptr, nullptr, nullptr},
6467:   {(char*)"_saved_self", (getter)THPSelectBackwardAutogradNestedTensor0_copy_self_getter, nullptr, nullptr, nullptr},
6468:   {(char*)"_raw_saved_self", (getter)THPSelectBackwardAutogradNestedTensor0_copy_self_raw_getter, nullptr, nullptr, nullptr},
6469:   {nullptr} /* sentinel */
6470: };
6471: 
6472: static PyObject* THPUnfoldBackward0_copy_dimension_getter(THPCppFunction *self, void *_unused) {
6473:   HANDLE_TH_ERRORS
6474:   auto prop = static_cast<UnfoldBackward0_copy*>(self->cdata.get())->dimension;
6475:   return PyLong_FromUnsignedLong((int64_t) prop);
6476:   END_HANDLE_TH_ERRORS
6477: }
6478: 
6479: static PyObject* THPUnfoldBackward0_copy_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
6480:   HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPSelectBackward0_copy_self_sym_sizes_getter`, `PyTuple_New`, `PyTuple_SetItem`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPSelectBackward0_copy_self_sym_sizes_getter`, `PyTuple_New`, `PyTuple_SetItem` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 6481-6560

```cpp
6481:   auto prop = static_cast<UnfoldBackward0_copy*>(self->cdata.get())->self_sym_sizes;
6482:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
6483:   for (auto i : c10::irange(prop.size())) {
6484:       auto si = prop[i];
6485:       if (auto m = si.maybe_as_int()) {
6486:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
6487:       } else {
6488:         auto py_symint = py::cast(si).release().ptr();
6489:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
6490:       }
6491:   }
6492:   return tup;
6493:   END_HANDLE_TH_ERRORS
6494: }
6495: 
6496: static PyObject* THPUnfoldBackward0_copy_size_getter(THPCppFunction *self, void *_unused) {
6497:   HANDLE_TH_ERRORS
6498:   auto prop = static_cast<UnfoldBackward0_copy*>(self->cdata.get())->size;
6499:   return PyLong_FromUnsignedLong((int64_t) prop);
6500:   END_HANDLE_TH_ERRORS
6501: }
6502: 
6503: static PyObject* THPUnfoldBackward0_copy_step_getter(THPCppFunction *self, void *_unused) {
6504:   HANDLE_TH_ERRORS
6505:   auto prop = static_cast<UnfoldBackward0_copy*>(self->cdata.get())->step;
6506:   return PyLong_FromUnsignedLong((int64_t) prop);
6507:   END_HANDLE_TH_ERRORS
6508: }
6509: 
6510: static struct PyGetSetDef UnfoldBackward0_copy_properties[] = {
6511:   THP_FUNCTION_DEFAULT_PROPERTIES,
6512:   {(char*)"_saved_dimension", (getter)THPUnfoldBackward0_copy_dimension_getter, nullptr, nullptr, nullptr},
6513:   {(char*)"_saved_self_sym_sizes", (getter)THPUnfoldBackward0_copy_self_sym_sizes_getter, nullptr, nullptr, nullptr},
6514:   {(char*)"_saved_size", (getter)THPUnfoldBackward0_copy_size_getter, nullptr, nullptr, nullptr},
6515:   {(char*)"_saved_step", (getter)THPUnfoldBackward0_copy_step_getter, nullptr, nullptr, nullptr},
6516:   {nullptr} /* sentinel */
6517: };
6518: 
6519: 
6520: 
6521: static struct PyGetSetDef NestedViewFromJaggedBackward0_copy_properties[] = {
6522:   THP_FUNCTION_DEFAULT_PROPERTIES,
6523: 
6524:   {nullptr} /* sentinel */
6525: };
6526: 
6527: void initialize_autogenerated_functions_3(PyObject* module) {
6528:   static PyTypeObject AddbmmBackward0Class;
6529:   addClass<AddbmmBackward0>(module, AddbmmBackward0Class, "AddbmmBackward0", AddbmmBackward0_properties);
6530:   static PyTypeObject AffineGridGeneratorBackward0Class;
6531:   addClass<AffineGridGeneratorBackward0>(module, AffineGridGeneratorBackward0Class, "AffineGridGeneratorBackward0", AffineGridGeneratorBackward0_properties);
6532:   static PyTypeObject AtanhBackward0Class;
6533:   addClass<AtanhBackward0>(module, AtanhBackward0Class, "AtanhBackward0", AtanhBackward0_properties);
6534:   static PyTypeObject BernoulliBackward1Class;
6535:   addClass<BernoulliBackward1>(module, BernoulliBackward1Class, "BernoulliBackward1", BernoulliBackward1_properties);
6536:   static PyTypeObject BernoulliBackward2Class;
6537:   addClass<BernoulliBackward2>(module, BernoulliBackward2Class, "BernoulliBackward2", BernoulliBackward2_properties);
6538:   static PyTypeObject BmmBackward0Class;
6539:   addClass<BmmBackward0>(module, BmmBackward0Class, "BmmBackward0", BmmBackward0_properties);
6540:   static PyTypeObject CauchyBackward0Class;
6541:   addClass<CauchyBackward0>(module, CauchyBackward0Class, "CauchyBackward0", CauchyBackward0_properties);
6542:   static PyTypeObject ClampMinBackward0Class;
6543:   addClass<ClampMinBackward0>(module, ClampMinBackward0Class, "ClampMinBackward0", ClampMinBackward0_properties);
6544:   static PyTypeObject ClampMinBackward1Class;
6545:   addClass<ClampMinBackward1>(module, ClampMinBackward1Class, "ClampMinBackward1", ClampMinBackward1_properties);
6546:   static PyTypeObject CoalesceBackward0Class;
6547:   addClass<CoalesceBackward0>(module, CoalesceBackward0Class, "CoalesceBackward0", CoalesceBackward0_properties);
6548:   static PyTypeObject ConjPhysicalBackward1Class;
6549:   addClass<ConjPhysicalBackward1>(module, ConjPhysicalBackward1Class, "ConjPhysicalBackward1", ConjPhysicalBackward1_properties);
6550:   static PyTypeObject CumprodBackward0Class;
6551:   addClass<CumprodBackward0>(module, CumprodBackward0Class, "CumprodBackward0", CumprodBackward0_properties);
6552:   static PyTypeObject LinalgDetBackward0Class;
6553:   addClass<LinalgDetBackward0>(module, LinalgDetBackward0Class, "LinalgDetBackward0", LinalgDetBackward0_properties);
6554:   static PyTypeObject DiagEmbedBackward0Class;
6555:   addClass<DiagEmbedBackward0>(module, DiagEmbedBackward0Class, "DiagEmbedBackward0", DiagEmbedBackward0_properties);
6556:   static PyTypeObject DiagonalBackward0Class;
6557:   addClass<DiagonalBackward0>(module, DiagonalBackward0Class, "DiagonalBackward0", DiagonalBackward0_properties);
6558:   static PyTypeObject NativeDropoutBackwardBackward0Class;
6559:   addClass<NativeDropoutBackwardBackward0>(module, NativeDropoutBackwardBackward0Class, "NativeDropoutBackwardBackward0", NativeDropoutBackwardBackward0_properties);
6560:   static PyTypeObject EqBackward0Class;
```

- EN: The main execution path in this span is carried by `PyTuple_New`, `PyTuple_SetItem`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyTuple_New`, `PyTuple_SetItem`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 6561-6640

```cpp
6561:   addClass<EqBackward0>(module, EqBackward0Class, "EqBackward0", EqBackward0_properties);
6562:   static PyTypeObject EqBackward1Class;
6563:   addClass<EqBackward1>(module, EqBackward1Class, "EqBackward1", EqBackward1_properties);
6564:   static PyTypeObject ExpandBackward0Class;
6565:   addClass<ExpandBackward0>(module, ExpandBackward0Class, "ExpandBackward0", ExpandBackward0_properties);
6566:   static PyTypeObject FloorBackward0Class;
6567:   addClass<FloorBackward0>(module, FloorBackward0Class, "FloorBackward0", FloorBackward0_properties);
6568:   static PyTypeObject FrexpBackward0Class;
6569:   addClass<FrexpBackward0>(module, FrexpBackward0Class, "FrexpBackward0", FrexpBackward0_properties);
6570:   static PyTypeObject GatherBackward0Class;
6571:   addClass<GatherBackward0>(module, GatherBackward0Class, "GatherBackward0", GatherBackward0_properties);
6572:   static PyTypeObject GeBackward0Class;
6573:   addClass<GeBackward0>(module, GeBackward0Class, "GeBackward0", GeBackward0_properties);
6574:   static PyTypeObject GeBackward1Class;
6575:   addClass<GeBackward1>(module, GeBackward1Class, "GeBackward1", GeBackward1_properties);
6576:   static PyTypeObject GridSampler3DBackwardBackward0Class;
6577:   addClass<GridSampler3DBackwardBackward0>(module, GridSampler3DBackwardBackward0Class, "GridSampler3DBackwardBackward0", GridSampler3DBackwardBackward0_properties);
6578:   static PyTypeObject HardsigmoidBackward0Class;
6579:   addClass<HardsigmoidBackward0>(module, HardsigmoidBackward0Class, "HardsigmoidBackward0", HardsigmoidBackward0_properties);
6580:   static PyTypeObject I0Backward0Class;
6581:   addClass<I0Backward0>(module, I0Backward0Class, "I0Backward0", I0Backward0_properties);
6582:   static PyTypeObject UnsafeIndexBackward0Class;
6583:   addClass<UnsafeIndexBackward0>(module, UnsafeIndexBackward0Class, "UnsafeIndexBackward0", UnsafeIndexBackward0_properties);
6584:   static PyTypeObject LinalgInvExBackward0Class;
6585:   addClass<LinalgInvExBackward0>(module, LinalgInvExBackward0Class, "LinalgInvExBackward0", LinalgInvExBackward0_properties);
6586:   static PyTypeObject LerpBackward0Class;
6587:   addClass<LerpBackward0>(module, LerpBackward0Class, "LerpBackward0", LerpBackward0_properties);
6588:   static PyTypeObject LerpBackward1Class;
6589:   addClass<LerpBackward1>(module, LerpBackward1Class, "LerpBackward1", LerpBackward1_properties);
6590:   static PyTypeObject SpecialXlog1PyBackward0Class;
6591:   addClass<SpecialXlog1PyBackward0>(module, SpecialXlog1PyBackward0Class, "SpecialXlog1PyBackward0", SpecialXlog1PyBackward0_properties);
6592:   static PyTypeObject SpecialXlog1PyBackward1Class;
6593:   addClass<SpecialXlog1PyBackward1>(module, SpecialXlog1PyBackward1Class, "SpecialXlog1PyBackward1", SpecialXlog1PyBackward1_properties);
6594:   static PyTypeObject SpecialXlog1PyBackward2Class;
6595:   addClass<SpecialXlog1PyBackward2>(module, SpecialXlog1PyBackward2Class, "SpecialXlog1PyBackward2", SpecialXlog1PyBackward2_properties);
6596:   static PyTypeObject LtBackward0Class;
6597:   addClass<LtBackward0>(module, LtBackward0Class, "LtBackward0", LtBackward0_properties);
6598:   static PyTypeObject LtBackward1Class;
6599:   addClass<LtBackward1>(module, LtBackward1Class, "LtBackward1", LtBackward1_properties);
6600:   static PyTypeObject MeanBackward0Class;
6601:   addClass<MeanBackward0>(module, MeanBackward0Class, "MeanBackward0", MeanBackward0_properties);
6602:   static PyTypeObject MeanBackwardAutogradNestedTensor0Class;
6603:   addClass<MeanBackwardAutogradNestedTensor0>(module, MeanBackwardAutogradNestedTensor0Class, "MeanBackwardAutogradNestedTensor0", MeanBackwardAutogradNestedTensor0_properties);
6604:   static PyTypeObject MeanBackward1Class;
6605:   addClass<MeanBackward1>(module, MeanBackward1Class, "MeanBackward1", MeanBackward1_properties);
6606:   static PyTypeObject MinBackward0Class;
6607:   addClass<MinBackward0>(module, MinBackward0Class, "MinBackward0", MinBackward0_properties);
6608:   static PyTypeObject MinBackward1Class;
6609:   addClass<MinBackward1>(module, MinBackward1Class, "MinBackward1", MinBackward1_properties);
6610:   static PyTypeObject FminBackward0Class;
6611:   addClass<FminBackward0>(module, FminBackward0Class, "FminBackward0", FminBackward0_properties);
6612:   static PyTypeObject MmBackward0Class;
6613:   addClass<MmBackward0>(module, MmBackward0Class, "MmBackward0", MmBackward0_properties);
6614:   static PyTypeObject MvBackward0Class;
6615:   addClass<MvBackward0>(module, MvBackward0Class, "MvBackward0", MvBackward0_properties);
6616:   static PyTypeObject NativeBatchNormLegitNoTrainingBackward0Class;
6617:   addClass<NativeBatchNormLegitNoTrainingBackward0>(module, NativeBatchNormLegitNoTrainingBackward0Class, "NativeBatchNormLegitNoTrainingBackward0", NativeBatchNormLegitNoTrainingBackward0_properties);
6618:   static PyTypeObject NativeGroupNormBackward0Class;
6619:   addClass<NativeGroupNormBackward0>(module, NativeGroupNormBackward0Class, "NativeGroupNormBackward0", NativeGroupNormBackward0_properties);
6620:   static PyTypeObject LinalgVectorNormBackward0Class;
6621:   addClass<LinalgVectorNormBackward0>(module, LinalgVectorNormBackward0Class, "LinalgVectorNormBackward0", LinalgVectorNormBackward0_properties);
6622:   static PyTypeObject PdistBackwardBackward0Class;
6623:   addClass<PdistBackwardBackward0>(module, PdistBackwardBackward0Class, "PdistBackwardBackward0", PdistBackwardBackward0_properties);
6624:   static PyTypeObject OrmqrBackward0Class;
6625:   addClass<OrmqrBackward0>(module, OrmqrBackward0Class, "OrmqrBackward0", OrmqrBackward0_properties);
6626:   static PyTypeObject PermuteBackward0Class;
6627:   addClass<PermuteBackward0>(module, PermuteBackward0Class, "PermuteBackward0", PermuteBackward0_properties);
6628:   static PyTypeObject ReshapeAliasBackward0Class;
6629:   addClass<ReshapeAliasBackward0>(module, ReshapeAliasBackward0Class, "ReshapeAliasBackward0", ReshapeAliasBackward0_properties);
6630:   static PyTypeObject SelectBackward0Class;
6631:   addClass<SelectBackward0>(module, SelectBackward0Class, "SelectBackward0", SelectBackward0_properties);
6632:   static PyTypeObject SelectBackwardAutogradNestedTensor0Class;
6633:   addClass<SelectBackwardAutogradNestedTensor0>(module, SelectBackwardAutogradNestedTensor0Class, "SelectBackwardAutogradNestedTensor0", SelectBackwardAutogradNestedTensor0_properties);
6634:   static PyTypeObject LogitBackward0Class;
6635:   addClass<LogitBackward0>(module, LogitBackward0Class, "LogitBackward0", LogitBackward0_properties);
6636:   static PyTypeObject SignBackward0Class;
6637:   addClass<SignBackward0>(module, SignBackward0Class, "SignBackward0", SignBackward0_properties);
6638:   static PyTypeObject SinhBackward0Class;
6639:   addClass<SinhBackward0>(module, SinhBackward0Class, "SinhBackward0", SinhBackward0_properties);
6640:   static PyTypeObject SliceBackwardBackward0Class;
```

- EN: Autograd-related metadata or graph connectivity is updated here.
- CN: 这里会更新与自动求导相关的元数据或计算图连接关系。
### Lines 6641-6720

```cpp
6641:   addClass<SliceBackwardBackward0>(module, SliceBackwardBackward0Class, "SliceBackwardBackward0", SliceBackwardBackward0_properties);
6642:   static PyTypeObject UnsafeSplitBackward0Class;
6643:   addClass<UnsafeSplitBackward0>(module, UnsafeSplitBackward0Class, "UnsafeSplitBackward0", UnsafeSplitBackward0_properties);
6644:   static PyTypeObject SqrtBackward0Class;
6645:   addClass<SqrtBackward0>(module, SqrtBackward0Class, "SqrtBackward0", SqrtBackward0_properties);
6646:   static PyTypeObject StdBackward0Class;
6647:   addClass<StdBackward0>(module, StdBackward0Class, "StdBackward0", StdBackward0_properties);
6648:   static PyTypeObject StdMeanBackward0Class;
6649:   addClass<StdMeanBackward0>(module, StdMeanBackward0Class, "StdMeanBackward0", StdMeanBackward0_properties);
6650:   static PyTypeObject RsubBackward0Class;
6651:   addClass<RsubBackward0>(module, RsubBackward0Class, "RsubBackward0", RsubBackward0_properties);
6652:   static PyTypeObject RsubBackward1Class;
6653:   addClass<RsubBackward1>(module, RsubBackward1Class, "RsubBackward1", RsubBackward1_properties);
6654:   static PyTypeObject TBackward0Class;
6655:   addClass<TBackward0>(module, TBackward0Class, "TBackward0", TBackward0_properties);
6656:   static PyTypeObject FlipBackward0Class;
6657:   addClass<FlipBackward0>(module, FlipBackward0Class, "FlipBackward0", FlipBackward0_properties);
6658:   static PyTypeObject RollBackward0Class;
6659:   addClass<RollBackward0>(module, RollBackward0Class, "RollBackward0", RollBackward0_properties);
6660:   static PyTypeObject TransposeBackward0Class;
6661:   addClass<TransposeBackward0>(module, TransposeBackward0Class, "TransposeBackward0", TransposeBackward0_properties);
6662:   static PyTypeObject TransposeBackward1Class;
6663:   addClass<TransposeBackward1>(module, TransposeBackward1Class, "TransposeBackward1", TransposeBackward1_properties);
6664:   static PyTypeObject TriangularSolveBackward0Class;
6665:   addClass<TriangularSolveBackward0>(module, TriangularSolveBackward0Class, "TriangularSolveBackward0", TriangularSolveBackward0_properties);
6666:   static PyTypeObject LinalgSolveTriangularBackward0Class;
6667:   addClass<LinalgSolveTriangularBackward0>(module, LinalgSolveTriangularBackward0Class, "LinalgSolveTriangularBackward0", LinalgSolveTriangularBackward0_properties);
6668:   static PyTypeObject TriuBackward0Class;
6669:   addClass<TriuBackward0>(module, TriuBackward0Class, "TriuBackward0", TriuBackward0_properties);
6670:   static PyTypeObject ToSparseBackward0Class;
6671:   addClass<ToSparseBackward0>(module, ToSparseBackward0Class, "ToSparseBackward0", ToSparseBackward0_properties);
6672:   static PyTypeObject ToSparseBackward1Class;
6673:   addClass<ToSparseBackward1>(module, ToSparseBackward1Class, "ToSparseBackward1", ToSparseBackward1_properties);
6674:   static PyTypeObject Unique2Backward0Class;
6675:   addClass<Unique2Backward0>(module, Unique2Backward0Class, "Unique2Backward0", Unique2Backward0_properties);
6676:   static PyTypeObject ViewBackward0Class;
6677:   addClass<ViewBackward0>(module, ViewBackward0Class, "ViewBackward0", ViewBackward0_properties);
6678:   static PyTypeObject ViewBackwardAutogradNestedTensor0Class;
6679:   addClass<ViewBackwardAutogradNestedTensor0>(module, ViewBackwardAutogradNestedTensor0Class, "ViewBackwardAutogradNestedTensor0", ViewBackwardAutogradNestedTensor0_properties);
6680:   static PyTypeObject TrilinearBackward0Class;
6681:   addClass<TrilinearBackward0>(module, TrilinearBackward0Class, "TrilinearBackward0", TrilinearBackward0_properties);
6682:   static PyTypeObject BinaryCrossEntropyBackward0Class;
6683:   addClass<BinaryCrossEntropyBackward0>(module, BinaryCrossEntropyBackward0Class, "BinaryCrossEntropyBackward0", BinaryCrossEntropyBackward0_properties);
6684:   static PyTypeObject EmbeddingRenormBackward0Class;
6685:   addClass<EmbeddingRenormBackward0>(module, EmbeddingRenormBackward0Class, "EmbeddingRenormBackward0", EmbeddingRenormBackward0_properties);
6686:   static PyTypeObject HuberLossBackward0Class;
6687:   addClass<HuberLossBackward0>(module, HuberLossBackward0Class, "HuberLossBackward0", HuberLossBackward0_properties);
6688:   static PyTypeObject SiluBackward0Class;
6689:   addClass<SiluBackward0>(module, SiluBackward0Class, "SiluBackward0", SiluBackward0_properties);
6690:   static PyTypeObject CeluBackward0Class;
6691:   addClass<CeluBackward0>(module, CeluBackward0Class, "CeluBackward0", CeluBackward0_properties);
6692:   static PyTypeObject CeluBackward1Class;
6693:   addClass<CeluBackward1>(module, CeluBackward1Class, "CeluBackward1", CeluBackward1_properties);
6694:   static PyTypeObject SparseLogSoftmaxBackward0Class;
6695:   addClass<SparseLogSoftmaxBackward0>(module, SparseLogSoftmaxBackward0Class, "SparseLogSoftmaxBackward0", SparseLogSoftmaxBackward0_properties);
6696:   static PyTypeObject MaskedSoftmaxBackward0Class;
6697:   addClass<MaskedSoftmaxBackward0>(module, MaskedSoftmaxBackward0Class, "MaskedSoftmaxBackward0", MaskedSoftmaxBackward0_properties);
6698:   static PyTypeObject SoftmaxBackward0Class;
6699:   addClass<SoftmaxBackward0>(module, SoftmaxBackward0Class, "SoftmaxBackward0", SoftmaxBackward0_properties);
6700:   static PyTypeObject ThresholdBackward0Class;
6701:   addClass<ThresholdBackward0>(module, ThresholdBackward0Class, "ThresholdBackward0", ThresholdBackward0_properties);
6702:   static PyTypeObject ReplicationPad3DBackward0Class;
6703:   addClass<ReplicationPad3DBackward0>(module, ReplicationPad3DBackward0Class, "ReplicationPad3DBackward0", ReplicationPad3DBackward0_properties);
6704:   static PyTypeObject UpsampleLinear1DBackward0Class;
6705:   addClass<UpsampleLinear1DBackward0>(module, UpsampleLinear1DBackward0Class, "UpsampleLinear1DBackward0", UpsampleLinear1DBackward0_properties);
6706:   static PyTypeObject UpsampleBilinear2DBackward0Class;
6707:   addClass<UpsampleBilinear2DBackward0>(module, UpsampleBilinear2DBackward0Class, "UpsampleBilinear2DBackward0", UpsampleBilinear2DBackward0_properties);
6708:   static PyTypeObject UpsampleBicubic2DBackward0Class;
6709:   addClass<UpsampleBicubic2DBackward0>(module, UpsampleBicubic2DBackward0Class, "UpsampleBicubic2DBackward0", UpsampleBicubic2DBackward0_properties);
6710:   static PyTypeObject UpsampleNearest2DBackward0Class;
6711:   addClass<UpsampleNearest2DBackward0>(module, UpsampleNearest2DBackward0Class, "UpsampleNearest2DBackward0", UpsampleNearest2DBackward0_properties);
6712:   static PyTypeObject PixelShuffleBackward0Class;
6713:   addClass<PixelShuffleBackward0>(module, PixelShuffleBackward0Class, "PixelShuffleBackward0", PixelShuffleBackward0_properties);
6714:   static PyTypeObject AdaptiveAvgPool3DBackward0Class;
6715:   addClass<AdaptiveAvgPool3DBackward0>(module, AdaptiveAvgPool3DBackward0Class, "AdaptiveAvgPool3DBackward0", AdaptiveAvgPool3DBackward0_properties);
6716:   static PyTypeObject AdaptiveMaxPool2DBackward0Class;
6717:   addClass<AdaptiveMaxPool2DBackward0>(module, AdaptiveMaxPool2DBackward0Class, "AdaptiveMaxPool2DBackward0", AdaptiveMaxPool2DBackward0_properties);
6718:   static PyTypeObject AdaptiveMaxPool3DBackward0Class;
6719:   addClass<AdaptiveMaxPool3DBackward0>(module, AdaptiveMaxPool3DBackward0Class, "AdaptiveMaxPool3DBackward0", AdaptiveMaxPool3DBackward0_properties);
6720:   static PyTypeObject FractionalMaxPool2DBackward0Class;
```

- EN: Autograd-related metadata or graph connectivity is updated here.
- CN: 这里会更新与自动求导相关的元数据或计算图连接关系。
### Lines 6721-6800

```cpp
6721:   addClass<FractionalMaxPool2DBackward0>(module, FractionalMaxPool2DBackward0Class, "FractionalMaxPool2DBackward0", FractionalMaxPool2DBackward0_properties);
6722:   static PyTypeObject LinearBackwardBackward0Class;
6723:   addClass<LinearBackwardBackward0>(module, LinearBackwardBackward0Class, "LinearBackwardBackward0", LinearBackwardBackward0_properties);
6724:   static PyTypeObject MaxPool2DBackward0Class;
6725:   addClass<MaxPool2DBackward0>(module, MaxPool2DBackward0Class, "MaxPool2DBackward0", MaxPool2DBackward0_properties);
6726:   static PyTypeObject MaxUnpool2DBackward0Class;
6727:   addClass<MaxUnpool2DBackward0>(module, MaxUnpool2DBackward0Class, "MaxUnpool2DBackward0", MaxUnpool2DBackward0_properties);
6728:   static PyTypeObject MaxUnpool3DBackward0Class;
6729:   addClass<MaxUnpool3DBackward0>(module, MaxUnpool3DBackward0Class, "MaxUnpool3DBackward0", MaxUnpool3DBackward0_properties);
6730:   static PyTypeObject SlowConvTranspose2DBackward0Class;
6731:   addClass<SlowConvTranspose2DBackward0>(module, SlowConvTranspose2DBackward0Class, "SlowConvTranspose2DBackward0", SlowConvTranspose2DBackward0_properties);
6732:   static PyTypeObject SlowConv2DBackwardBackward0Class;
6733:   addClass<SlowConv2DBackwardBackward0>(module, SlowConv2DBackwardBackward0Class, "SlowConv2DBackwardBackward0", SlowConv2DBackwardBackward0_properties);
6734:   static PyTypeObject ConvDepthwise3DBackward0Class;
6735:   addClass<ConvDepthwise3DBackward0>(module, ConvDepthwise3DBackward0Class, "ConvDepthwise3DBackward0", ConvDepthwise3DBackward0_properties);
6736:   static PyTypeObject SlowConvDilated2DBackward0Class;
6737:   addClass<SlowConvDilated2DBackward0>(module, SlowConvDilated2DBackward0Class, "SlowConvDilated2DBackward0", SlowConvDilated2DBackward0_properties);
6738:   static PyTypeObject Col2ImBackward0Class;
6739:   addClass<Col2ImBackward0>(module, Col2ImBackward0Class, "Col2ImBackward0", Col2ImBackward0_properties);
6740:   static PyTypeObject AvgPool2DBackwardBackward0Class;
6741:   addClass<AvgPool2DBackwardBackward0>(module, AvgPool2DBackwardBackward0Class, "AvgPool2DBackwardBackward0", AvgPool2DBackwardBackward0_properties);
6742:   static PyTypeObject NllLossBackwardBackward0Class;
6743:   addClass<NllLossBackwardBackward0>(module, NllLossBackwardBackward0Class, "NllLossBackwardBackward0", NllLossBackwardBackward0_properties);
6744:   static PyTypeObject ReflectionPad3DBackwardBackward0Class;
6745:   addClass<ReflectionPad3DBackwardBackward0>(module, ReflectionPad3DBackwardBackward0Class, "ReflectionPad3DBackwardBackward0", ReflectionPad3DBackwardBackward0_properties);
6746:   static PyTypeObject ReplicationPad2DBackwardBackward0Class;
6747:   addClass<ReplicationPad2DBackwardBackward0>(module, ReplicationPad2DBackwardBackward0Class, "ReplicationPad2DBackwardBackward0", ReplicationPad2DBackwardBackward0_properties);
6748:   static PyTypeObject SparseMmReduceImplBackward0Class;
6749:   addClass<SparseMmReduceImplBackward0>(module, SparseMmReduceImplBackward0Class, "SparseMmReduceImplBackward0", SparseMmReduceImplBackward0_properties);
6750:   static PyTypeObject SmoothL1LossBackwardBackward0Class;
6751:   addClass<SmoothL1LossBackwardBackward0>(module, SmoothL1LossBackwardBackward0Class, "SmoothL1LossBackwardBackward0", SmoothL1LossBackwardBackward0_properties);
6752:   static PyTypeObject HuberLossBackwardBackward0Class;
6753:   addClass<HuberLossBackwardBackward0>(module, HuberLossBackwardBackward0Class, "HuberLossBackwardBackward0", HuberLossBackwardBackward0_properties);
6754:   static PyTypeObject UpsampleBicubic2DBackwardBackward0Class;
6755:   addClass<UpsampleBicubic2DBackwardBackward0>(module, UpsampleBicubic2DBackwardBackward0Class, "UpsampleBicubic2DBackwardBackward0", UpsampleBicubic2DBackwardBackward0_properties);
6756:   static PyTypeObject UpsampleTrilinear3DBackwardBackward0Class;
6757:   addClass<UpsampleTrilinear3DBackwardBackward0>(module, UpsampleTrilinear3DBackwardBackward0Class, "UpsampleTrilinear3DBackwardBackward0", UpsampleTrilinear3DBackwardBackward0_properties);
6758:   static PyTypeObject UpsampleNearest3DBackwardBackward0Class;
6759:   addClass<UpsampleNearest3DBackwardBackward0>(module, UpsampleNearest3DBackwardBackward0Class, "UpsampleNearest3DBackwardBackward0", UpsampleNearest3DBackwardBackward0_properties);
6760:   static PyTypeObject UpsampleNearestExact3DBackwardBackward0Class;
6761:   addClass<UpsampleNearestExact3DBackwardBackward0>(module, UpsampleNearestExact3DBackwardBackward0Class, "UpsampleNearestExact3DBackwardBackward0", UpsampleNearestExact3DBackwardBackward0_properties);
6762:   static PyTypeObject CudnnConvolutionBackward0Class;
6763:   addClass<CudnnConvolutionBackward0>(module, CudnnConvolutionBackward0Class, "CudnnConvolutionBackward0", CudnnConvolutionBackward0_properties);
6764:   static PyTypeObject CudnnRnnBackwardBackward0Class;
6765:   addClass<CudnnRnnBackwardBackward0>(module, CudnnRnnBackwardBackward0Class, "CudnnRnnBackwardBackward0", CudnnRnnBackwardBackward0_properties);
6766:   static PyTypeObject MiopenCtcLossBackward0Class;
6767:   addClass<MiopenCtcLossBackward0>(module, MiopenCtcLossBackward0Class, "MiopenCtcLossBackward0", MiopenCtcLossBackward0_properties);
6768:   static PyTypeObject MiopenCtcLossBackward1Class;
6769:   addClass<MiopenCtcLossBackward1>(module, MiopenCtcLossBackward1Class, "MiopenCtcLossBackward1", MiopenCtcLossBackward1_properties);
6770:   static PyTypeObject NestedFromPaddedBackward0Class;
6771:   addClass<NestedFromPaddedBackward0>(module, NestedFromPaddedBackward0Class, "NestedFromPaddedBackward0", NestedFromPaddedBackward0_properties);
6772:   static PyTypeObject NestedViewFromBufferBackward0Class;
6773:   addClass<NestedViewFromBufferBackward0>(module, NestedViewFromBufferBackward0Class, "NestedViewFromBufferBackward0", NestedViewFromBufferBackward0_properties);
6774:   static PyTypeObject NestedViewFromJaggedBackward0Class;
6775:   addClass<NestedViewFromJaggedBackward0>(module, NestedViewFromJaggedBackward0Class, "NestedViewFromJaggedBackward0", NestedViewFromJaggedBackward0_properties);
6776:   static PyTypeObject SafeSoftmaxBackward0Class;
6777:   addClass<SafeSoftmaxBackward0>(module, SafeSoftmaxBackward0Class, "SafeSoftmaxBackward0", SafeSoftmaxBackward0_properties);
6778:   static PyTypeObject ScaledDotProductEfficientAttentionBackward0Class;
6779:   addClass<ScaledDotProductEfficientAttentionBackward0>(module, ScaledDotProductEfficientAttentionBackward0Class, "ScaledDotProductEfficientAttentionBackward0", ScaledDotProductEfficientAttentionBackward0_properties);
6780:   static PyTypeObject EfficientAttentionBackward0Class;
6781:   addClass<EfficientAttentionBackward0>(module, EfficientAttentionBackward0Class, "EfficientAttentionBackward0", EfficientAttentionBackward0_properties);
6782:   static PyTypeObject FftR2CBackward0Class;
6783:   addClass<FftR2CBackward0>(module, FftR2CBackward0Class, "FftR2CBackward0", FftR2CBackward0_properties);
6784:   static PyTypeObject ThnnFusedGruCellBackward0Class;
6785:   addClass<ThnnFusedGruCellBackward0>(module, ThnnFusedGruCellBackward0Class, "ThnnFusedGruCellBackward0", ThnnFusedGruCellBackward0_properties);
6786:   static PyTypeObject PackPaddedSequenceBackward0Class;
6787:   addClass<PackPaddedSequenceBackward0>(module, PackPaddedSequenceBackward0Class, "PackPaddedSequenceBackward0", PackPaddedSequenceBackward0_properties);
6788:   static PyTypeObject NarrowCopyBackward0Class;
6789:   addClass<NarrowCopyBackward0>(module, NarrowCopyBackward0Class, "NarrowCopyBackward0", NarrowCopyBackward0_properties);
6790:   static PyTypeObject ForeachDivBackward0Class;
6791:   addClass<ForeachDivBackward0>(module, ForeachDivBackward0Class, "ForeachDivBackward0", ForeachDivBackward0_properties);
6792:   static PyTypeObject SelectBackward0_copyClass;
6793:   addClass<SelectBackward0_copy>(module, SelectBackward0_copyClass, "SelectBackward0_copy", SelectBackward0_copy_properties);
6794:   static PyTypeObject SelectBackwardAutogradNestedTensor0_copyClass;
6795:   addClass<SelectBackwardAutogradNestedTensor0_copy>(module, SelectBackwardAutogradNestedTensor0_copyClass, "SelectBackwardAutogradNestedTensor0_copy", SelectBackwardAutogradNestedTensor0_copy_properties);
6796:   static PyTypeObject UnfoldBackward0_copyClass;
6797:   addClass<UnfoldBackward0_copy>(module, UnfoldBackward0_copyClass, "UnfoldBackward0_copy", UnfoldBackward0_copy_properties);
6798:   static PyTypeObject NestedViewFromJaggedBackward0_copyClass;
6799:   addClass<NestedViewFromJaggedBackward0_copy>(module, NestedViewFromJaggedBackward0_copyClass, "NestedViewFromJaggedBackward0_copy", NestedViewFromJaggedBackward0_copy_properties);
6800: }
```

- EN: Autograd-related metadata or graph connectivity is updated here.
- CN: 这里会更新与自动求导相关的元数据或计算图连接关系。
### Lines 6801-6802

```cpp
6801: 
6802: } // namespace torch::autograd::generated
```

- EN: They also open, refine, or close the relevant C++ namespace scope.
- CN: 它们还会打开、细化或关闭相应的 C++ 命名空间作用域。

## Key Concepts / 关键概念
- Autograd graph and gradient metadata / 自动求导图与梯度元数据
- Generated code patterns / 生成代码模式
- Gradient bookkeeping and autograd history / 梯度记录与自动求导历史
- Python/C++ binding boundary / Python/C++ 绑定边界
- Tensor/value representation management / 张量/数值表示管理
- Primary symbol `addClass` / 核心符号 `addClass`

## Dependencies / 依赖关系
- Direct includes / 直接包含: `torch/csrc/autograd/generated/python_functions.h`, `Python.h`, `ATen/ATen.h`, `c10/core/SymNodeImpl.h`, `torch/csrc/autograd/generated/Functions.h`, `torch/csrc/autograd/python_cpp_function.h`, `torch/csrc/autograd/python_variable.h`, `torch/csrc/autograd/saved_variable.h`, `torch/csrc/utils/pybind.h`, `pybind11/pybind11.h`
- Include roots / 头文件根模块: `ATen`, `c10`, `pybind11`, `torch`
- Key symbols / 关键符号: `addClass`, `THPAddbmmBackward0_alpha_getter`, `PyComplex_FromDoubles`, `PyFloat_FromDouble`, `PyLong_FromLong`, `THPAddbmmBackward0_batch1_getter`, `THPVariable_Wrap`, `THPAddbmmBackward0_batch1_raw_getter`, `THPAddbmmBackward0_batch1_sym_argsize_0_getter`, `PyLong_FromUnsignedLong`
- Related subsystems / 相关子系统: Autograd / 自动求导, ATen operator layer / ATen 算子层, c10 core utilities / c10 核心工具, Python binding layer / Python 绑定层
