# python_functions_0.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/autograd/generated/python_functions_0.cpp`
- Repository: `/root/xw/pytorch`
- Purpose (EN): Defines generated backward-function classes and helpers used by the autograd engine.
- 目的 (CN): 定义自动求导引擎使用的生成式反向函数类与辅助逻辑。
- Lines: 6903
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
31: static PyObject* THPAddcmulBackward0_tensor1_getter(THPCppFunction *self, void *_unused) {
32:   HANDLE_TH_ERRORS
33:   const auto& prop = static_cast<AddcmulBackward0*>(self->cdata.get())->tensor1_;
34:   return THPVariable_Wrap(prop.unpack(self->cdata));
35:   END_HANDLE_TH_ERRORS
36: }
37: 
38: static PyObject* THPAddcmulBackward0_tensor1_raw_getter(THPCppFunction *self, void *_unused) {
39:   HANDLE_TH_ERRORS
40:   const auto& prop = static_cast<AddcmulBackward0*>(self->cdata.get())->tensor1_;
41:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
42:   return obj.release().ptr();
43:   END_HANDLE_TH_ERRORS
44: }
45: 
46: static PyObject* THPAddcmulBackward0_tensor2_getter(THPCppFunction *self, void *_unused) {
47:   HANDLE_TH_ERRORS
48:   const auto& prop = static_cast<AddcmulBackward0*>(self->cdata.get())->tensor2_;
49:   return THPVariable_Wrap(prop.unpack(self->cdata));
50:   END_HANDLE_TH_ERRORS
51: }
52: 
53: static PyObject* THPAddcmulBackward0_tensor2_raw_getter(THPCppFunction *self, void *_unused) {
54:   HANDLE_TH_ERRORS
55:   const auto& prop = static_cast<AddcmulBackward0*>(self->cdata.get())->tensor2_;
56:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
57:   return obj.release().ptr();
58:   END_HANDLE_TH_ERRORS
59: }
60: 
61: static PyObject* THPAddcmulBackward0_value_getter(THPCppFunction *self, void *_unused) {
62:   HANDLE_TH_ERRORS
63:   auto prop = static_cast<AddcmulBackward0*>(self->cdata.get())->value;
64:   if (prop.isComplex()) {
65:     auto cprop = prop.to<c10::complex<double>>();
66:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
67:   } else if (prop.isFloatingPoint()) {
68:     return PyFloat_FromDouble(prop.to<double>());
69:   } else if (prop.isIntegral(/*includeBool=*/false)) {
70:     return PyLong_FromLong(prop.to<int64_t>());
71:   } else if (prop.isBoolean()) {
72:     if (prop.to<bool>()) {
73:       Py_RETURN_TRUE;
74:     } else {
75:       Py_RETURN_FALSE;
76:     }
77:   } else {
78:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
79:     return nullptr;
80:   }
```

- EN: These lines pull in dependencies such as `torch/csrc/autograd/generated/python_functions.h`, `Python.h`, `ATen/ATen.h`, establishing the headers needed by the implementation. They also open, refine, or close the relevant C++ namespace scope. The main execution path in this span is carried by `addClass`, `_initFunctionPyTypeObject`, `Py_INCREF`. Because this is generated binding code, the span mostly registers or forwards APIs into a mechanically produced Python-facing surface.
- CN: 这些行引入了依赖，例如 `torch/csrc/autograd/generated/python_functions.h`, `Python.h`, `ATen/ATen.h`，为后续实现建立所需的头文件基础。 它们还会打开、细化或关闭相应的 C++ 命名空间作用域。 这一段的主要执行路径由 `addClass`, `_initFunctionPyTypeObject`, `Py_INCREF` 等函数/方法承载。 由于这是生成的绑定代码，这一段主要是在机械化生成的 Python 接口层上完成 API 注册或转发。
### Lines 81-160

```cpp
 81:   END_HANDLE_TH_ERRORS
 82: }
 83: 
 84: static struct PyGetSetDef AddcmulBackward0_properties[] = {
 85:   THP_FUNCTION_DEFAULT_PROPERTIES,
 86:   {(char*)"_saved_tensor1", (getter)THPAddcmulBackward0_tensor1_getter, nullptr, nullptr, nullptr},
 87:   {(char*)"_raw_saved_tensor1", (getter)THPAddcmulBackward0_tensor1_raw_getter, nullptr, nullptr, nullptr},
 88:   {(char*)"_saved_tensor2", (getter)THPAddcmulBackward0_tensor2_getter, nullptr, nullptr, nullptr},
 89:   {(char*)"_raw_saved_tensor2", (getter)THPAddcmulBackward0_tensor2_raw_getter, nullptr, nullptr, nullptr},
 90:   {(char*)"_saved_value", (getter)THPAddcmulBackward0_value_getter, nullptr, nullptr, nullptr},
 91:   {nullptr} /* sentinel */
 92: };
 93: 
 94: static PyObject* THPAddmmBackward0_alpha_getter(THPCppFunction *self, void *_unused) {
 95:   HANDLE_TH_ERRORS
 96:   auto prop = static_cast<AddmmBackward0*>(self->cdata.get())->alpha;
 97:   if (prop.isComplex()) {
 98:     auto cprop = prop.to<c10::complex<double>>();
 99:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
100:   } else if (prop.isFloatingPoint()) {
101:     return PyFloat_FromDouble(prop.to<double>());
102:   } else if (prop.isIntegral(/*includeBool=*/false)) {
103:     return PyLong_FromLong(prop.to<int64_t>());
104:   } else if (prop.isBoolean()) {
105:     if (prop.to<bool>()) {
106:       Py_RETURN_TRUE;
107:     } else {
108:       Py_RETURN_FALSE;
109:     }
110:   } else {
111:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
112:     return nullptr;
113:   }
114:   END_HANDLE_TH_ERRORS
115: }
116: 
117: static PyObject* THPAddmmBackward0_beta_getter(THPCppFunction *self, void *_unused) {
118:   HANDLE_TH_ERRORS
119:   auto prop = static_cast<AddmmBackward0*>(self->cdata.get())->beta;
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
140: static PyObject* THPAddmmBackward0_mat1_getter(THPCppFunction *self, void *_unused) {
141:   HANDLE_TH_ERRORS
142:   const auto& prop = static_cast<AddmmBackward0*>(self->cdata.get())->mat1_;
143:   return THPVariable_Wrap(prop.unpack(self->cdata));
144:   END_HANDLE_TH_ERRORS
145: }
146: 
147: static PyObject* THPAddmmBackward0_mat1_raw_getter(THPCppFunction *self, void *_unused) {
148:   HANDLE_TH_ERRORS
149:   const auto& prop = static_cast<AddmmBackward0*>(self->cdata.get())->mat1_;
150:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
151:   return obj.release().ptr();
152:   END_HANDLE_TH_ERRORS
153: }
154: 
155: static PyObject* THPAddmmBackward0_mat1_sym_sizes_getter(THPCppFunction *self, void *_unused) {
156:   HANDLE_TH_ERRORS
157:   auto prop = static_cast<AddmmBackward0*>(self->cdata.get())->mat1_sym_sizes;
158:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
159:   for (auto i : c10::irange(prop.size())) {
160:       auto si = prop[i];
```

- EN: The main execution path in this span is carried by `THPAddmmBackward0_alpha_getter`, `PyComplex_FromDoubles`, `PyFloat_FromDouble`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPAddmmBackward0_alpha_getter`, `PyComplex_FromDoubles`, `PyFloat_FromDouble` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 161-240

```cpp
161:       if (auto m = si.maybe_as_int()) {
162:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
163:       } else {
164:         auto py_symint = py::cast(si).release().ptr();
165:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
166:       }
167:   }
168:   return tup;
169:   END_HANDLE_TH_ERRORS
170: }
171: 
172: static PyObject* THPAddmmBackward0_mat1_sym_strides_getter(THPCppFunction *self, void *_unused) {
173:   HANDLE_TH_ERRORS
174:   auto prop = static_cast<AddmmBackward0*>(self->cdata.get())->mat1_sym_strides;
175:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
176:   for (auto i : c10::irange(prop.size())) {
177:       auto si = prop[i];
178:       if (auto m = si.maybe_as_int()) {
179:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
180:       } else {
181:         auto py_symint = py::cast(si).release().ptr();
182:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
183:       }
184:   }
185:   return tup;
186:   END_HANDLE_TH_ERRORS
187: }
188: 
189: static PyObject* THPAddmmBackward0_mat2_getter(THPCppFunction *self, void *_unused) {
190:   HANDLE_TH_ERRORS
191:   const auto& prop = static_cast<AddmmBackward0*>(self->cdata.get())->mat2_;
192:   return THPVariable_Wrap(prop.unpack(self->cdata));
193:   END_HANDLE_TH_ERRORS
194: }
195: 
196: static PyObject* THPAddmmBackward0_mat2_raw_getter(THPCppFunction *self, void *_unused) {
197:   HANDLE_TH_ERRORS
198:   const auto& prop = static_cast<AddmmBackward0*>(self->cdata.get())->mat2_;
199:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
200:   return obj.release().ptr();
201:   END_HANDLE_TH_ERRORS
202: }
203: 
204: static PyObject* THPAddmmBackward0_mat2_sym_sizes_getter(THPCppFunction *self, void *_unused) {
205:   HANDLE_TH_ERRORS
206:   auto prop = static_cast<AddmmBackward0*>(self->cdata.get())->mat2_sym_sizes;
207:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
208:   for (auto i : c10::irange(prop.size())) {
209:       auto si = prop[i];
210:       if (auto m = si.maybe_as_int()) {
211:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
212:       } else {
213:         auto py_symint = py::cast(si).release().ptr();
214:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
215:       }
216:   }
217:   return tup;
218:   END_HANDLE_TH_ERRORS
219: }
220: 
221: static PyObject* THPAddmmBackward0_mat2_sym_strides_getter(THPCppFunction *self, void *_unused) {
222:   HANDLE_TH_ERRORS
223:   auto prop = static_cast<AddmmBackward0*>(self->cdata.get())->mat2_sym_strides;
224:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
225:   for (auto i : c10::irange(prop.size())) {
226:       auto si = prop[i];
227:       if (auto m = si.maybe_as_int()) {
228:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
229:       } else {
230:         auto py_symint = py::cast(si).release().ptr();
231:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
232:       }
233:   }
234:   return tup;
235:   END_HANDLE_TH_ERRORS
236: }
237: 
238: static struct PyGetSetDef AddmmBackward0_properties[] = {
239:   THP_FUNCTION_DEFAULT_PROPERTIES,
240:   {(char*)"_saved_alpha", (getter)THPAddmmBackward0_alpha_getter, nullptr, nullptr, nullptr},
```

- EN: The main execution path in this span is carried by `PyTuple_SetItem`, `cast`, `THPAddmmBackward0_mat1_sym_strides_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyTuple_SetItem`, `cast`, `THPAddmmBackward0_mat1_sym_strides_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 241-320

```cpp
241:   {(char*)"_saved_beta", (getter)THPAddmmBackward0_beta_getter, nullptr, nullptr, nullptr},
242:   {(char*)"_saved_mat1", (getter)THPAddmmBackward0_mat1_getter, nullptr, nullptr, nullptr},
243:   {(char*)"_raw_saved_mat1", (getter)THPAddmmBackward0_mat1_raw_getter, nullptr, nullptr, nullptr},
244:   {(char*)"_saved_mat1_sym_sizes", (getter)THPAddmmBackward0_mat1_sym_sizes_getter, nullptr, nullptr, nullptr},
245:   {(char*)"_saved_mat1_sym_strides", (getter)THPAddmmBackward0_mat1_sym_strides_getter, nullptr, nullptr, nullptr},
246:   {(char*)"_saved_mat2", (getter)THPAddmmBackward0_mat2_getter, nullptr, nullptr, nullptr},
247:   {(char*)"_raw_saved_mat2", (getter)THPAddmmBackward0_mat2_raw_getter, nullptr, nullptr, nullptr},
248:   {(char*)"_saved_mat2_sym_sizes", (getter)THPAddmmBackward0_mat2_sym_sizes_getter, nullptr, nullptr, nullptr},
249:   {(char*)"_saved_mat2_sym_strides", (getter)THPAddmmBackward0_mat2_sym_strides_getter, nullptr, nullptr, nullptr},
250:   {nullptr} /* sentinel */
251: };
252: 
253: static PyObject* THPSparseAddmmBackward0_alpha_getter(THPCppFunction *self, void *_unused) {
254:   HANDLE_TH_ERRORS
255:   auto prop = static_cast<SparseAddmmBackward0*>(self->cdata.get())->alpha;
256:   if (prop.isComplex()) {
257:     auto cprop = prop.to<c10::complex<double>>();
258:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
259:   } else if (prop.isFloatingPoint()) {
260:     return PyFloat_FromDouble(prop.to<double>());
261:   } else if (prop.isIntegral(/*includeBool=*/false)) {
262:     return PyLong_FromLong(prop.to<int64_t>());
263:   } else if (prop.isBoolean()) {
264:     if (prop.to<bool>()) {
265:       Py_RETURN_TRUE;
266:     } else {
267:       Py_RETURN_FALSE;
268:     }
269:   } else {
270:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
271:     return nullptr;
272:   }
273:   END_HANDLE_TH_ERRORS
274: }
275: 
276: static PyObject* THPSparseAddmmBackward0_beta_getter(THPCppFunction *self, void *_unused) {
277:   HANDLE_TH_ERRORS
278:   auto prop = static_cast<SparseAddmmBackward0*>(self->cdata.get())->beta;
279:   if (prop.isComplex()) {
280:     auto cprop = prop.to<c10::complex<double>>();
281:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
282:   } else if (prop.isFloatingPoint()) {
283:     return PyFloat_FromDouble(prop.to<double>());
284:   } else if (prop.isIntegral(/*includeBool=*/false)) {
285:     return PyLong_FromLong(prop.to<int64_t>());
286:   } else if (prop.isBoolean()) {
287:     if (prop.to<bool>()) {
288:       Py_RETURN_TRUE;
289:     } else {
290:       Py_RETURN_FALSE;
291:     }
292:   } else {
293:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
294:     return nullptr;
295:   }
296:   END_HANDLE_TH_ERRORS
297: }
298: 
299: static PyObject* THPSparseAddmmBackward0_mat1_getter(THPCppFunction *self, void *_unused) {
300:   HANDLE_TH_ERRORS
301:   const auto& prop = static_cast<SparseAddmmBackward0*>(self->cdata.get())->mat1_;
302:   return THPVariable_Wrap(prop.unpack(self->cdata));
303:   END_HANDLE_TH_ERRORS
304: }
305: 
306: static PyObject* THPSparseAddmmBackward0_mat1_raw_getter(THPCppFunction *self, void *_unused) {
307:   HANDLE_TH_ERRORS
308:   const auto& prop = static_cast<SparseAddmmBackward0*>(self->cdata.get())->mat1_;
309:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
310:   return obj.release().ptr();
311:   END_HANDLE_TH_ERRORS
312: }
313: 
314: static PyObject* THPSparseAddmmBackward0_mat2_getter(THPCppFunction *self, void *_unused) {
315:   HANDLE_TH_ERRORS
316:   const auto& prop = static_cast<SparseAddmmBackward0*>(self->cdata.get())->mat2_;
317:   return THPVariable_Wrap(prop.unpack(self->cdata));
318:   END_HANDLE_TH_ERRORS
319: }
320: 
```

- EN: The main execution path in this span is carried by `THPSparseAddmmBackward0_alpha_getter`, `PyComplex_FromDoubles`, `PyFloat_FromDouble`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPSparseAddmmBackward0_alpha_getter`, `PyComplex_FromDoubles`, `PyFloat_FromDouble` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 321-400

```cpp
321: static PyObject* THPSparseAddmmBackward0_mat2_raw_getter(THPCppFunction *self, void *_unused) {
322:   HANDLE_TH_ERRORS
323:   const auto& prop = static_cast<SparseAddmmBackward0*>(self->cdata.get())->mat2_;
324:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
325:   return obj.release().ptr();
326:   END_HANDLE_TH_ERRORS
327: }
328: 
329: static PyObject* THPSparseAddmmBackward0_mat2_sym_sizes_getter(THPCppFunction *self, void *_unused) {
330:   HANDLE_TH_ERRORS
331:   auto prop = static_cast<SparseAddmmBackward0*>(self->cdata.get())->mat2_sym_sizes;
332:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
333:   for (auto i : c10::irange(prop.size())) {
334:       auto si = prop[i];
335:       if (auto m = si.maybe_as_int()) {
336:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
337:       } else {
338:         auto py_symint = py::cast(si).release().ptr();
339:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
340:       }
341:   }
342:   return tup;
343:   END_HANDLE_TH_ERRORS
344: }
345: 
346: static PyObject* THPSparseAddmmBackward0_mat2_sym_strides_getter(THPCppFunction *self, void *_unused) {
347:   HANDLE_TH_ERRORS
348:   auto prop = static_cast<SparseAddmmBackward0*>(self->cdata.get())->mat2_sym_strides;
349:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
350:   for (auto i : c10::irange(prop.size())) {
351:       auto si = prop[i];
352:       if (auto m = si.maybe_as_int()) {
353:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
354:       } else {
355:         auto py_symint = py::cast(si).release().ptr();
356:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
357:       }
358:   }
359:   return tup;
360:   END_HANDLE_TH_ERRORS
361: }
362: 
363: static struct PyGetSetDef SparseAddmmBackward0_properties[] = {
364:   THP_FUNCTION_DEFAULT_PROPERTIES,
365:   {(char*)"_saved_alpha", (getter)THPSparseAddmmBackward0_alpha_getter, nullptr, nullptr, nullptr},
366:   {(char*)"_saved_beta", (getter)THPSparseAddmmBackward0_beta_getter, nullptr, nullptr, nullptr},
367:   {(char*)"_saved_mat1", (getter)THPSparseAddmmBackward0_mat1_getter, nullptr, nullptr, nullptr},
368:   {(char*)"_raw_saved_mat1", (getter)THPSparseAddmmBackward0_mat1_raw_getter, nullptr, nullptr, nullptr},
369:   {(char*)"_saved_mat2", (getter)THPSparseAddmmBackward0_mat2_getter, nullptr, nullptr, nullptr},
370:   {(char*)"_raw_saved_mat2", (getter)THPSparseAddmmBackward0_mat2_raw_getter, nullptr, nullptr, nullptr},
371:   {(char*)"_saved_mat2_sym_sizes", (getter)THPSparseAddmmBackward0_mat2_sym_sizes_getter, nullptr, nullptr, nullptr},
372:   {(char*)"_saved_mat2_sym_strides", (getter)THPSparseAddmmBackward0_mat2_sym_strides_getter, nullptr, nullptr, nullptr},
373:   {nullptr} /* sentinel */
374: };
375: 
376: static PyObject* THPAcoshBackward0_self_getter(THPCppFunction *self, void *_unused) {
377:   HANDLE_TH_ERRORS
378:   const auto& prop = static_cast<AcoshBackward0*>(self->cdata.get())->self_;
379:   return THPVariable_Wrap(prop.unpack(self->cdata));
380:   END_HANDLE_TH_ERRORS
381: }
382: 
383: static PyObject* THPAcoshBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
384:   HANDLE_TH_ERRORS
385:   const auto& prop = static_cast<AcoshBackward0*>(self->cdata.get())->self_;
386:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
387:   return obj.release().ptr();
388:   END_HANDLE_TH_ERRORS
389: }
390: 
391: static struct PyGetSetDef AcoshBackward0_properties[] = {
392:   THP_FUNCTION_DEFAULT_PROPERTIES,
393:   {(char*)"_saved_self", (getter)THPAcoshBackward0_self_getter, nullptr, nullptr, nullptr},
394:   {(char*)"_raw_saved_self", (getter)THPAcoshBackward0_self_raw_getter, nullptr, nullptr, nullptr},
395:   {nullptr} /* sentinel */
396: };
397: 
398: 
399: 
400: static struct PyGetSetDef AtanhBackward1_properties[] = {
```

- EN: The main execution path in this span is carried by `THPSparseAddmmBackward0_mat2_raw_getter`, `cast`, `THPSparseAddmmBackward0_mat2_sym_sizes_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPSparseAddmmBackward0_mat2_raw_getter`, `cast`, `THPSparseAddmmBackward0_mat2_sym_sizes_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 401-480

```cpp
401:   THP_FUNCTION_DEFAULT_PROPERTIES,
402: 
403:   {nullptr} /* sentinel */
404: };
405: 
406: static PyObject* THPAsStridedBackward0_size_getter(THPCppFunction *self, void *_unused) {
407:   HANDLE_TH_ERRORS
408:   auto prop = static_cast<AsStridedBackward0*>(self->cdata.get())->size;
409:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
410:   for (auto i : c10::irange(prop.size())) {
411:       auto si = prop[i];
412:       if (auto m = si.maybe_as_int()) {
413:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
414:       } else {
415:         auto py_symint = py::cast(si).release().ptr();
416:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
417:       }
418:   }
419:   return tup;
420:   END_HANDLE_TH_ERRORS
421: }
422: 
423: static PyObject* THPAsStridedBackward0_storage_offset_getter(THPCppFunction *self, void *_unused) {
424:   HANDLE_TH_ERRORS
425:   auto opt_prop = static_cast<AsStridedBackward0*>(self->cdata.get())->storage_offset;
426:   if (!opt_prop.has_value()) {
427:     Py_RETURN_NONE;
428:   }
429:   auto prop = opt_prop.value();
430:   if (auto m = prop.maybe_as_int()) {
431:     return PyLong_FromUnsignedLong(*m);
432:   } else {
433:     return py::cast(prop).release().ptr();
434:   }
435:   END_HANDLE_TH_ERRORS
436: }
437: 
438: static PyObject* THPAsStridedBackward0_stride_getter(THPCppFunction *self, void *_unused) {
439:   HANDLE_TH_ERRORS
440:   auto prop = static_cast<AsStridedBackward0*>(self->cdata.get())->stride;
441:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
442:   for (auto i : c10::irange(prop.size())) {
443:       auto si = prop[i];
444:       if (auto m = si.maybe_as_int()) {
445:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
446:       } else {
447:         auto py_symint = py::cast(si).release().ptr();
448:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
449:       }
450:   }
451:   return tup;
452:   END_HANDLE_TH_ERRORS
453: }
454: 
455: static struct PyGetSetDef AsStridedBackward0_properties[] = {
456:   THP_FUNCTION_DEFAULT_PROPERTIES,
457:   {(char*)"_saved_size", (getter)THPAsStridedBackward0_size_getter, nullptr, nullptr, nullptr},
458:   {(char*)"_saved_storage_offset", (getter)THPAsStridedBackward0_storage_offset_getter, nullptr, nullptr, nullptr},
459:   {(char*)"_saved_stride", (getter)THPAsStridedBackward0_stride_getter, nullptr, nullptr, nullptr},
460:   {nullptr} /* sentinel */
461: };
462: 
463: static PyObject* THPCatBackward0_dim_getter(THPCppFunction *self, void *_unused) {
464:   HANDLE_TH_ERRORS
465:   auto prop = static_cast<CatBackward0*>(self->cdata.get())->dim;
466:   return PyLong_FromUnsignedLong((int64_t) prop);
467:   END_HANDLE_TH_ERRORS
468: }
469: 
470: static struct PyGetSetDef CatBackward0_properties[] = {
471:   THP_FUNCTION_DEFAULT_PROPERTIES,
472:   {(char*)"_saved_dim", (getter)THPCatBackward0_dim_getter, nullptr, nullptr, nullptr},
473:   {nullptr} /* sentinel */
474: };
475: 
476: static PyObject* THPCholeskyBackward0_upper_getter(THPCppFunction *self, void *_unused) {
477:   HANDLE_TH_ERRORS
478:   auto prop = static_cast<CholeskyBackward0*>(self->cdata.get())->upper;
479:   if (prop) {
480:     Py_RETURN_TRUE;
```

- EN: The main execution path in this span is carried by `THPAsStridedBackward0_size_getter`, `PyTuple_New`, `PyTuple_SetItem`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPAsStridedBackward0_size_getter`, `PyTuple_New`, `PyTuple_SetItem` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 481-560

```cpp
481:   } else {
482:     Py_RETURN_FALSE;
483:   }
484:   END_HANDLE_TH_ERRORS
485: }
486: 
487: static PyObject* THPCholeskyBackward0_result_getter(THPCppFunction *self, void *_unused) {
488:   HANDLE_TH_ERRORS
489:   const auto& prop = static_cast<CholeskyBackward0*>(self->cdata.get())->result_;
490:   return THPVariable_Wrap(prop.unpack(self->cdata));
491:   END_HANDLE_TH_ERRORS
492: }
493: 
494: static PyObject* THPCholeskyBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
495:   HANDLE_TH_ERRORS
496:   const auto& prop = static_cast<CholeskyBackward0*>(self->cdata.get())->result_;
497:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
498:   return obj.release().ptr();
499:   END_HANDLE_TH_ERRORS
500: }
501: 
502: static struct PyGetSetDef CholeskyBackward0_properties[] = {
503:   THP_FUNCTION_DEFAULT_PROPERTIES,
504:   {(char*)"_saved_upper", (getter)THPCholeskyBackward0_upper_getter, nullptr, nullptr, nullptr},
505:   {(char*)"_saved_result", (getter)THPCholeskyBackward0_result_getter, nullptr, nullptr, nullptr},
506:   {(char*)"_raw_saved_result", (getter)THPCholeskyBackward0_result_raw_getter, nullptr, nullptr, nullptr},
507:   {nullptr} /* sentinel */
508: };
509: 
510: 
511: 
512: static struct PyGetSetDef ToCopyBackward0_properties[] = {
513:   THP_FUNCTION_DEFAULT_PROPERTIES,
514: 
515:   {nullptr} /* sentinel */
516: };
517: 
518: 
519: 
520: static struct PyGetSetDef ConjBackward0_properties[] = {
521:   THP_FUNCTION_DEFAULT_PROPERTIES,
522: 
523:   {nullptr} /* sentinel */
524: };
525: 
526: 
527: 
528: static struct PyGetSetDef NegViewBackward0_properties[] = {
529:   THP_FUNCTION_DEFAULT_PROPERTIES,
530: 
531:   {nullptr} /* sentinel */
532: };
533: 
534: 
535: 
536: static struct PyGetSetDef ConjPhysicalBackward0_properties[] = {
537:   THP_FUNCTION_DEFAULT_PROPERTIES,
538: 
539:   {nullptr} /* sentinel */
540: };
541: 
542: static PyObject* THPCosBackward0_self_getter(THPCppFunction *self, void *_unused) {
543:   HANDLE_TH_ERRORS
544:   const auto& prop = static_cast<CosBackward0*>(self->cdata.get())->self_;
545:   return THPVariable_Wrap(prop.unpack(self->cdata));
546:   END_HANDLE_TH_ERRORS
547: }
548: 
549: static PyObject* THPCosBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
550:   HANDLE_TH_ERRORS
551:   const auto& prop = static_cast<CosBackward0*>(self->cdata.get())->self_;
552:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
553:   return obj.release().ptr();
554:   END_HANDLE_TH_ERRORS
555: }
556: 
557: static struct PyGetSetDef CosBackward0_properties[] = {
558:   THP_FUNCTION_DEFAULT_PROPERTIES,
559:   {(char*)"_saved_self", (getter)THPCosBackward0_self_getter, nullptr, nullptr, nullptr},
560:   {(char*)"_raw_saved_self", (getter)THPCosBackward0_self_raw_getter, nullptr, nullptr, nullptr},
```

- EN: The main execution path in this span is carried by `THPCholeskyBackward0_result_getter`, `THPVariable_Wrap`, `THPCholeskyBackward0_result_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPCholeskyBackward0_result_getter`, `THPVariable_Wrap`, `THPCholeskyBackward0_result_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 561-640

```cpp
561:   {nullptr} /* sentinel */
562: };
563: 
564: static PyObject* THPCumsumBackward0_dim_getter(THPCppFunction *self, void *_unused) {
565:   HANDLE_TH_ERRORS
566:   auto prop = static_cast<CumsumBackward0*>(self->cdata.get())->dim;
567:   return PyLong_FromUnsignedLong((int64_t) prop);
568:   END_HANDLE_TH_ERRORS
569: }
570: 
571: static struct PyGetSetDef CumsumBackward0_properties[] = {
572:   THP_FUNCTION_DEFAULT_PROPERTIES,
573:   {(char*)"_saved_dim", (getter)THPCumsumBackward0_dim_getter, nullptr, nullptr, nullptr},
574:   {nullptr} /* sentinel */
575: };
576: 
577: static PyObject* THPCtcLossBackward0_blank_getter(THPCppFunction *self, void *_unused) {
578:   HANDLE_TH_ERRORS
579:   auto prop = static_cast<CtcLossBackward0*>(self->cdata.get())->blank;
580:   return PyLong_FromUnsignedLong((int64_t) prop);
581:   END_HANDLE_TH_ERRORS
582: }
583: 
584: static PyObject* THPCtcLossBackward0_input_lengths_getter(THPCppFunction *self, void *_unused) {
585:   HANDLE_TH_ERRORS
586:   auto prop = static_cast<CtcLossBackward0*>(self->cdata.get())->input_lengths;
587:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
588:   for (auto i : c10::irange(prop.size())) {
589:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
590:   }
591:   return tup;
592:   END_HANDLE_TH_ERRORS
593: }
594: 
595: static PyObject* THPCtcLossBackward0_log_probs_getter(THPCppFunction *self, void *_unused) {
596:   HANDLE_TH_ERRORS
597:   const auto& prop = static_cast<CtcLossBackward0*>(self->cdata.get())->log_probs_;
598:   return THPVariable_Wrap(prop.unpack(self->cdata));
599:   END_HANDLE_TH_ERRORS
600: }
601: 
602: static PyObject* THPCtcLossBackward0_log_probs_raw_getter(THPCppFunction *self, void *_unused) {
603:   HANDLE_TH_ERRORS
604:   const auto& prop = static_cast<CtcLossBackward0*>(self->cdata.get())->log_probs_;
605:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
606:   return obj.release().ptr();
607:   END_HANDLE_TH_ERRORS
608: }
609: 
610: static PyObject* THPCtcLossBackward0_target_lengths_getter(THPCppFunction *self, void *_unused) {
611:   HANDLE_TH_ERRORS
612:   auto prop = static_cast<CtcLossBackward0*>(self->cdata.get())->target_lengths;
613:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
614:   for (auto i : c10::irange(prop.size())) {
615:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
616:   }
617:   return tup;
618:   END_HANDLE_TH_ERRORS
619: }
620: 
621: static PyObject* THPCtcLossBackward0_targets_getter(THPCppFunction *self, void *_unused) {
622:   HANDLE_TH_ERRORS
623:   const auto& prop = static_cast<CtcLossBackward0*>(self->cdata.get())->targets_;
624:   return THPVariable_Wrap(prop.unpack(self->cdata));
625:   END_HANDLE_TH_ERRORS
626: }
627: 
628: static PyObject* THPCtcLossBackward0_targets_raw_getter(THPCppFunction *self, void *_unused) {
629:   HANDLE_TH_ERRORS
630:   const auto& prop = static_cast<CtcLossBackward0*>(self->cdata.get())->targets_;
631:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
632:   return obj.release().ptr();
633:   END_HANDLE_TH_ERRORS
634: }
635: 
636: static PyObject* THPCtcLossBackward0_zero_infinity_getter(THPCppFunction *self, void *_unused) {
637:   HANDLE_TH_ERRORS
638:   auto prop = static_cast<CtcLossBackward0*>(self->cdata.get())->zero_infinity;
639:   if (prop) {
640:     Py_RETURN_TRUE;
```

- EN: The main execution path in this span is carried by `THPCumsumBackward0_dim_getter`, `PyLong_FromUnsignedLong`, `THPCtcLossBackward0_blank_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPCumsumBackward0_dim_getter`, `PyLong_FromUnsignedLong`, `THPCtcLossBackward0_blank_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 641-720

```cpp
641:   } else {
642:     Py_RETURN_FALSE;
643:   }
644:   END_HANDLE_TH_ERRORS
645: }
646: 
647: static PyObject* THPCtcLossBackward0_result0_getter(THPCppFunction *self, void *_unused) {
648:   HANDLE_TH_ERRORS
649:   const auto& prop = static_cast<CtcLossBackward0*>(self->cdata.get())->result0_;
650:   return THPVariable_Wrap(prop.unpack(self->cdata));
651:   END_HANDLE_TH_ERRORS
652: }
653: 
654: static PyObject* THPCtcLossBackward0_result0_raw_getter(THPCppFunction *self, void *_unused) {
655:   HANDLE_TH_ERRORS
656:   const auto& prop = static_cast<CtcLossBackward0*>(self->cdata.get())->result0_;
657:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
658:   return obj.release().ptr();
659:   END_HANDLE_TH_ERRORS
660: }
661: 
662: static PyObject* THPCtcLossBackward0_result1_getter(THPCppFunction *self, void *_unused) {
663:   HANDLE_TH_ERRORS
664:   const auto& prop = static_cast<CtcLossBackward0*>(self->cdata.get())->result1_;
665:   return THPVariable_Wrap(prop.unpack(self->cdata));
666:   END_HANDLE_TH_ERRORS
667: }
668: 
669: static PyObject* THPCtcLossBackward0_result1_raw_getter(THPCppFunction *self, void *_unused) {
670:   HANDLE_TH_ERRORS
671:   const auto& prop = static_cast<CtcLossBackward0*>(self->cdata.get())->result1_;
672:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
673:   return obj.release().ptr();
674:   END_HANDLE_TH_ERRORS
675: }
676: 
677: static struct PyGetSetDef CtcLossBackward0_properties[] = {
678:   THP_FUNCTION_DEFAULT_PROPERTIES,
679:   {(char*)"_saved_blank", (getter)THPCtcLossBackward0_blank_getter, nullptr, nullptr, nullptr},
680:   {(char*)"_saved_input_lengths", (getter)THPCtcLossBackward0_input_lengths_getter, nullptr, nullptr, nullptr},
681:   {(char*)"_saved_log_probs", (getter)THPCtcLossBackward0_log_probs_getter, nullptr, nullptr, nullptr},
682:   {(char*)"_raw_saved_log_probs", (getter)THPCtcLossBackward0_log_probs_raw_getter, nullptr, nullptr, nullptr},
683:   {(char*)"_saved_target_lengths", (getter)THPCtcLossBackward0_target_lengths_getter, nullptr, nullptr, nullptr},
684:   {(char*)"_saved_targets", (getter)THPCtcLossBackward0_targets_getter, nullptr, nullptr, nullptr},
685:   {(char*)"_raw_saved_targets", (getter)THPCtcLossBackward0_targets_raw_getter, nullptr, nullptr, nullptr},
686:   {(char*)"_saved_zero_infinity", (getter)THPCtcLossBackward0_zero_infinity_getter, nullptr, nullptr, nullptr},
687:   {(char*)"_saved_result0", (getter)THPCtcLossBackward0_result0_getter, nullptr, nullptr, nullptr},
688:   {(char*)"_raw_saved_result0", (getter)THPCtcLossBackward0_result0_raw_getter, nullptr, nullptr, nullptr},
689:   {(char*)"_saved_result1", (getter)THPCtcLossBackward0_result1_getter, nullptr, nullptr, nullptr},
690:   {(char*)"_raw_saved_result1", (getter)THPCtcLossBackward0_result1_raw_getter, nullptr, nullptr, nullptr},
691:   {nullptr} /* sentinel */
692: };
693: 
694: static PyObject* THPCtcLossBackward1_blank_getter(THPCppFunction *self, void *_unused) {
695:   HANDLE_TH_ERRORS
696:   auto prop = static_cast<CtcLossBackward1*>(self->cdata.get())->blank;
697:   return PyLong_FromUnsignedLong((int64_t) prop);
698:   END_HANDLE_TH_ERRORS
699: }
700: 
701: static PyObject* THPCtcLossBackward1_input_lengths_getter(THPCppFunction *self, void *_unused) {
702:   HANDLE_TH_ERRORS
703:   const auto& prop = static_cast<CtcLossBackward1*>(self->cdata.get())->input_lengths_;
704:   return THPVariable_Wrap(prop.unpack(self->cdata));
705:   END_HANDLE_TH_ERRORS
706: }
707: 
708: static PyObject* THPCtcLossBackward1_input_lengths_raw_getter(THPCppFunction *self, void *_unused) {
709:   HANDLE_TH_ERRORS
710:   const auto& prop = static_cast<CtcLossBackward1*>(self->cdata.get())->input_lengths_;
711:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
712:   return obj.release().ptr();
713:   END_HANDLE_TH_ERRORS
714: }
715: 
716: static PyObject* THPCtcLossBackward1_log_probs_getter(THPCppFunction *self, void *_unused) {
717:   HANDLE_TH_ERRORS
718:   const auto& prop = static_cast<CtcLossBackward1*>(self->cdata.get())->log_probs_;
719:   return THPVariable_Wrap(prop.unpack(self->cdata));
720:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPCtcLossBackward0_result0_getter`, `THPVariable_Wrap`, `THPCtcLossBackward0_result0_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPCtcLossBackward0_result0_getter`, `THPVariable_Wrap`, `THPCtcLossBackward0_result0_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 721-800

```cpp
721: }
722: 
723: static PyObject* THPCtcLossBackward1_log_probs_raw_getter(THPCppFunction *self, void *_unused) {
724:   HANDLE_TH_ERRORS
725:   const auto& prop = static_cast<CtcLossBackward1*>(self->cdata.get())->log_probs_;
726:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
727:   return obj.release().ptr();
728:   END_HANDLE_TH_ERRORS
729: }
730: 
731: static PyObject* THPCtcLossBackward1_target_lengths_getter(THPCppFunction *self, void *_unused) {
732:   HANDLE_TH_ERRORS
733:   const auto& prop = static_cast<CtcLossBackward1*>(self->cdata.get())->target_lengths_;
734:   return THPVariable_Wrap(prop.unpack(self->cdata));
735:   END_HANDLE_TH_ERRORS
736: }
737: 
738: static PyObject* THPCtcLossBackward1_target_lengths_raw_getter(THPCppFunction *self, void *_unused) {
739:   HANDLE_TH_ERRORS
740:   const auto& prop = static_cast<CtcLossBackward1*>(self->cdata.get())->target_lengths_;
741:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
742:   return obj.release().ptr();
743:   END_HANDLE_TH_ERRORS
744: }
745: 
746: static PyObject* THPCtcLossBackward1_targets_getter(THPCppFunction *self, void *_unused) {
747:   HANDLE_TH_ERRORS
748:   const auto& prop = static_cast<CtcLossBackward1*>(self->cdata.get())->targets_;
749:   return THPVariable_Wrap(prop.unpack(self->cdata));
750:   END_HANDLE_TH_ERRORS
751: }
752: 
753: static PyObject* THPCtcLossBackward1_targets_raw_getter(THPCppFunction *self, void *_unused) {
754:   HANDLE_TH_ERRORS
755:   const auto& prop = static_cast<CtcLossBackward1*>(self->cdata.get())->targets_;
756:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
757:   return obj.release().ptr();
758:   END_HANDLE_TH_ERRORS
759: }
760: 
761: static PyObject* THPCtcLossBackward1_zero_infinity_getter(THPCppFunction *self, void *_unused) {
762:   HANDLE_TH_ERRORS
763:   auto prop = static_cast<CtcLossBackward1*>(self->cdata.get())->zero_infinity;
764:   if (prop) {
765:     Py_RETURN_TRUE;
766:   } else {
767:     Py_RETURN_FALSE;
768:   }
769:   END_HANDLE_TH_ERRORS
770: }
771: 
772: static PyObject* THPCtcLossBackward1_result0_getter(THPCppFunction *self, void *_unused) {
773:   HANDLE_TH_ERRORS
774:   const auto& prop = static_cast<CtcLossBackward1*>(self->cdata.get())->result0_;
775:   return THPVariable_Wrap(prop.unpack(self->cdata));
776:   END_HANDLE_TH_ERRORS
777: }
778: 
779: static PyObject* THPCtcLossBackward1_result0_raw_getter(THPCppFunction *self, void *_unused) {
780:   HANDLE_TH_ERRORS
781:   const auto& prop = static_cast<CtcLossBackward1*>(self->cdata.get())->result0_;
782:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
783:   return obj.release().ptr();
784:   END_HANDLE_TH_ERRORS
785: }
786: 
787: static PyObject* THPCtcLossBackward1_result1_getter(THPCppFunction *self, void *_unused) {
788:   HANDLE_TH_ERRORS
789:   const auto& prop = static_cast<CtcLossBackward1*>(self->cdata.get())->result1_;
790:   return THPVariable_Wrap(prop.unpack(self->cdata));
791:   END_HANDLE_TH_ERRORS
792: }
793: 
794: static PyObject* THPCtcLossBackward1_result1_raw_getter(THPCppFunction *self, void *_unused) {
795:   HANDLE_TH_ERRORS
796:   const auto& prop = static_cast<CtcLossBackward1*>(self->cdata.get())->result1_;
797:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
798:   return obj.release().ptr();
799:   END_HANDLE_TH_ERRORS
800: }
```

- EN: The main execution path in this span is carried by `THPCtcLossBackward1_log_probs_raw_getter`, `cast`, `THPCtcLossBackward1_target_lengths_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPCtcLossBackward1_log_probs_raw_getter`, `cast`, `THPCtcLossBackward1_target_lengths_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 801-880

```cpp
801: 
802: static struct PyGetSetDef CtcLossBackward1_properties[] = {
803:   THP_FUNCTION_DEFAULT_PROPERTIES,
804:   {(char*)"_saved_blank", (getter)THPCtcLossBackward1_blank_getter, nullptr, nullptr, nullptr},
805:   {(char*)"_saved_input_lengths", (getter)THPCtcLossBackward1_input_lengths_getter, nullptr, nullptr, nullptr},
806:   {(char*)"_raw_saved_input_lengths", (getter)THPCtcLossBackward1_input_lengths_raw_getter, nullptr, nullptr, nullptr},
807:   {(char*)"_saved_log_probs", (getter)THPCtcLossBackward1_log_probs_getter, nullptr, nullptr, nullptr},
808:   {(char*)"_raw_saved_log_probs", (getter)THPCtcLossBackward1_log_probs_raw_getter, nullptr, nullptr, nullptr},
809:   {(char*)"_saved_target_lengths", (getter)THPCtcLossBackward1_target_lengths_getter, nullptr, nullptr, nullptr},
810:   {(char*)"_raw_saved_target_lengths", (getter)THPCtcLossBackward1_target_lengths_raw_getter, nullptr, nullptr, nullptr},
811:   {(char*)"_saved_targets", (getter)THPCtcLossBackward1_targets_getter, nullptr, nullptr, nullptr},
812:   {(char*)"_raw_saved_targets", (getter)THPCtcLossBackward1_targets_raw_getter, nullptr, nullptr, nullptr},
813:   {(char*)"_saved_zero_infinity", (getter)THPCtcLossBackward1_zero_infinity_getter, nullptr, nullptr, nullptr},
814:   {(char*)"_saved_result0", (getter)THPCtcLossBackward1_result0_getter, nullptr, nullptr, nullptr},
815:   {(char*)"_raw_saved_result0", (getter)THPCtcLossBackward1_result0_raw_getter, nullptr, nullptr, nullptr},
816:   {(char*)"_saved_result1", (getter)THPCtcLossBackward1_result1_getter, nullptr, nullptr, nullptr},
817:   {(char*)"_raw_saved_result1", (getter)THPCtcLossBackward1_result1_raw_getter, nullptr, nullptr, nullptr},
818:   {nullptr} /* sentinel */
819: };
820: 
821: static PyObject* THPVdotBackward0_other_getter(THPCppFunction *self, void *_unused) {
822:   HANDLE_TH_ERRORS
823:   const auto& prop = static_cast<VdotBackward0*>(self->cdata.get())->other_;
824:   return THPVariable_Wrap(prop.unpack(self->cdata));
825:   END_HANDLE_TH_ERRORS
826: }
827: 
828: static PyObject* THPVdotBackward0_other_raw_getter(THPCppFunction *self, void *_unused) {
829:   HANDLE_TH_ERRORS
830:   const auto& prop = static_cast<VdotBackward0*>(self->cdata.get())->other_;
831:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
832:   return obj.release().ptr();
833:   END_HANDLE_TH_ERRORS
834: }
835: 
836: static PyObject* THPVdotBackward0_self_getter(THPCppFunction *self, void *_unused) {
837:   HANDLE_TH_ERRORS
838:   const auto& prop = static_cast<VdotBackward0*>(self->cdata.get())->self_;
839:   return THPVariable_Wrap(prop.unpack(self->cdata));
840:   END_HANDLE_TH_ERRORS
841: }
842: 
843: static PyObject* THPVdotBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
844:   HANDLE_TH_ERRORS
845:   const auto& prop = static_cast<VdotBackward0*>(self->cdata.get())->self_;
846:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
847:   return obj.release().ptr();
848:   END_HANDLE_TH_ERRORS
849: }
850: 
851: static struct PyGetSetDef VdotBackward0_properties[] = {
852:   THP_FUNCTION_DEFAULT_PROPERTIES,
853:   {(char*)"_saved_other", (getter)THPVdotBackward0_other_getter, nullptr, nullptr, nullptr},
854:   {(char*)"_raw_saved_other", (getter)THPVdotBackward0_other_raw_getter, nullptr, nullptr, nullptr},
855:   {(char*)"_saved_self", (getter)THPVdotBackward0_self_getter, nullptr, nullptr, nullptr},
856:   {(char*)"_raw_saved_self", (getter)THPVdotBackward0_self_raw_getter, nullptr, nullptr, nullptr},
857:   {nullptr} /* sentinel */
858: };
859: 
860: static PyObject* THPFusedDropoutBackward0_p_getter(THPCppFunction *self, void *_unused) {
861:   HANDLE_TH_ERRORS
862:   auto prop = static_cast<FusedDropoutBackward0*>(self->cdata.get())->p;
863:   return PyFloat_FromDouble((double) prop);
864:   END_HANDLE_TH_ERRORS
865: }
866: 
867: static PyObject* THPFusedDropoutBackward0_result1_getter(THPCppFunction *self, void *_unused) {
868:   HANDLE_TH_ERRORS
869:   const auto& prop = static_cast<FusedDropoutBackward0*>(self->cdata.get())->result1_;
870:   return THPVariable_Wrap(prop.unpack(self->cdata));
871:   END_HANDLE_TH_ERRORS
872: }
873: 
874: static PyObject* THPFusedDropoutBackward0_result1_raw_getter(THPCppFunction *self, void *_unused) {
875:   HANDLE_TH_ERRORS
876:   const auto& prop = static_cast<FusedDropoutBackward0*>(self->cdata.get())->result1_;
877:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
878:   return obj.release().ptr();
879:   END_HANDLE_TH_ERRORS
880: }
```

- EN: The main execution path in this span is carried by `THPVdotBackward0_other_getter`, `THPVariable_Wrap`, `THPVdotBackward0_other_raw_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVdotBackward0_other_getter`, `THPVariable_Wrap`, `THPVdotBackward0_other_raw_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 881-960

```cpp
881: 
882: static struct PyGetSetDef FusedDropoutBackward0_properties[] = {
883:   THP_FUNCTION_DEFAULT_PROPERTIES,
884:   {(char*)"_saved_p", (getter)THPFusedDropoutBackward0_p_getter, nullptr, nullptr, nullptr},
885:   {(char*)"_saved_result1", (getter)THPFusedDropoutBackward0_result1_getter, nullptr, nullptr, nullptr},
886:   {(char*)"_raw_saved_result1", (getter)THPFusedDropoutBackward0_result1_raw_getter, nullptr, nullptr, nullptr},
887:   {nullptr} /* sentinel */
888: };
889: 
890: static PyObject* THPExpm1Backward0_result_getter(THPCppFunction *self, void *_unused) {
891:   HANDLE_TH_ERRORS
892:   const auto& prop = static_cast<Expm1Backward0*>(self->cdata.get())->result_;
893:   return THPVariable_Wrap(prop.unpack(self->cdata));
894:   END_HANDLE_TH_ERRORS
895: }
896: 
897: static PyObject* THPExpm1Backward0_result_raw_getter(THPCppFunction *self, void *_unused) {
898:   HANDLE_TH_ERRORS
899:   const auto& prop = static_cast<Expm1Backward0*>(self->cdata.get())->result_;
900:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
901:   return obj.release().ptr();
902:   END_HANDLE_TH_ERRORS
903: }
904: 
905: static struct PyGetSetDef Expm1Backward0_properties[] = {
906:   THP_FUNCTION_DEFAULT_PROPERTIES,
907:   {(char*)"_saved_result", (getter)THPExpm1Backward0_result_getter, nullptr, nullptr, nullptr},
908:   {(char*)"_raw_saved_result", (getter)THPExpm1Backward0_result_raw_getter, nullptr, nullptr, nullptr},
909:   {nullptr} /* sentinel */
910: };
911: 
912: 
913: 
914: static struct PyGetSetDef ExponentialBackward0_properties[] = {
915:   THP_FUNCTION_DEFAULT_PROPERTIES,
916: 
917:   {nullptr} /* sentinel */
918: };
919: 
920: static PyObject* THPFakeQuantizePerChannelAffineCachemaskBackward0_mask_getter(THPCppFunction *self, void *_unused) {
921:   HANDLE_TH_ERRORS
922:   const auto& prop = static_cast<FakeQuantizePerChannelAffineCachemaskBackward0*>(self->cdata.get())->mask_;
923:   return THPVariable_Wrap(prop.unpack(self->cdata));
924:   END_HANDLE_TH_ERRORS
925: }
926: 
927: static PyObject* THPFakeQuantizePerChannelAffineCachemaskBackward0_mask_raw_getter(THPCppFunction *self, void *_unused) {
928:   HANDLE_TH_ERRORS
929:   const auto& prop = static_cast<FakeQuantizePerChannelAffineCachemaskBackward0*>(self->cdata.get())->mask_;
930:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
931:   return obj.release().ptr();
932:   END_HANDLE_TH_ERRORS
933: }
934: 
935: static struct PyGetSetDef FakeQuantizePerChannelAffineCachemaskBackward0_properties[] = {
936:   THP_FUNCTION_DEFAULT_PROPERTIES,
937:   {(char*)"_saved_mask", (getter)THPFakeQuantizePerChannelAffineCachemaskBackward0_mask_getter, nullptr, nullptr, nullptr},
938:   {(char*)"_raw_saved_mask", (getter)THPFakeQuantizePerChannelAffineCachemaskBackward0_mask_raw_getter, nullptr, nullptr, nullptr},
939:   {nullptr} /* sentinel */
940: };
941: 
942: static PyObject* THPFusedMovingAvgObsFqHelperBackward0_mask_getter(THPCppFunction *self, void *_unused) {
943:   HANDLE_TH_ERRORS
944:   const auto& prop = static_cast<FusedMovingAvgObsFqHelperBackward0*>(self->cdata.get())->mask_;
945:   return THPVariable_Wrap(prop.unpack(self->cdata));
946:   END_HANDLE_TH_ERRORS
947: }
948: 
949: static PyObject* THPFusedMovingAvgObsFqHelperBackward0_mask_raw_getter(THPCppFunction *self, void *_unused) {
950:   HANDLE_TH_ERRORS
951:   const auto& prop = static_cast<FusedMovingAvgObsFqHelperBackward0*>(self->cdata.get())->mask_;
952:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
953:   return obj.release().ptr();
954:   END_HANDLE_TH_ERRORS
955: }
956: 
957: static struct PyGetSetDef FusedMovingAvgObsFqHelperBackward0_properties[] = {
958:   THP_FUNCTION_DEFAULT_PROPERTIES,
959:   {(char*)"_saved_mask", (getter)THPFusedMovingAvgObsFqHelperBackward0_mask_getter, nullptr, nullptr, nullptr},
960:   {(char*)"_raw_saved_mask", (getter)THPFusedMovingAvgObsFqHelperBackward0_mask_raw_getter, nullptr, nullptr, nullptr},
```

- EN: The main execution path in this span is carried by `THPExpm1Backward0_result_getter`, `THPVariable_Wrap`, `THPExpm1Backward0_result_raw_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPExpm1Backward0_result_getter`, `THPVariable_Wrap`, `THPExpm1Backward0_result_raw_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 961-1040

```cpp
 961:   {nullptr} /* sentinel */
 962: };
 963: 
 964: 
 965: 
 966: static struct PyGetSetDef FillBackward0_properties[] = {
 967:   THP_FUNCTION_DEFAULT_PROPERTIES,
 968: 
 969:   {nullptr} /* sentinel */
 970: };
 971: 
 972: 
 973: 
 974: static struct PyGetSetDef FillBackward1_properties[] = {
 975:   THP_FUNCTION_DEFAULT_PROPERTIES,
 976: 
 977:   {nullptr} /* sentinel */
 978: };
 979: 
 980: 
 981: 
 982: static struct PyGetSetDef GeometricBackward0_properties[] = {
 983:   THP_FUNCTION_DEFAULT_PROPERTIES,
 984: 
 985:   {nullptr} /* sentinel */
 986: };
 987: 
 988: static PyObject* THPGridSampler2DBackwardBackward0_align_corners_getter(THPCppFunction *self, void *_unused) {
 989:   HANDLE_TH_ERRORS
 990:   auto prop = static_cast<GridSampler2DBackwardBackward0*>(self->cdata.get())->align_corners;
 991:   if (prop) {
 992:     Py_RETURN_TRUE;
 993:   } else {
 994:     Py_RETURN_FALSE;
 995:   }
 996:   END_HANDLE_TH_ERRORS
 997: }
 998: 
 999: static PyObject* THPGridSampler2DBackwardBackward0_grad_output_getter(THPCppFunction *self, void *_unused) {
1000:   HANDLE_TH_ERRORS
1001:   const auto& prop = static_cast<GridSampler2DBackwardBackward0*>(self->cdata.get())->grad_output_;
1002:   return THPVariable_Wrap(prop.unpack(self->cdata));
1003:   END_HANDLE_TH_ERRORS
1004: }
1005: 
1006: static PyObject* THPGridSampler2DBackwardBackward0_grad_output_raw_getter(THPCppFunction *self, void *_unused) {
1007:   HANDLE_TH_ERRORS
1008:   const auto& prop = static_cast<GridSampler2DBackwardBackward0*>(self->cdata.get())->grad_output_;
1009:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1010:   return obj.release().ptr();
1011:   END_HANDLE_TH_ERRORS
1012: }
1013: 
1014: static PyObject* THPGridSampler2DBackwardBackward0_grid_getter(THPCppFunction *self, void *_unused) {
1015:   HANDLE_TH_ERRORS
1016:   const auto& prop = static_cast<GridSampler2DBackwardBackward0*>(self->cdata.get())->grid_;
1017:   return THPVariable_Wrap(prop.unpack(self->cdata));
1018:   END_HANDLE_TH_ERRORS
1019: }
1020: 
1021: static PyObject* THPGridSampler2DBackwardBackward0_grid_raw_getter(THPCppFunction *self, void *_unused) {
1022:   HANDLE_TH_ERRORS
1023:   const auto& prop = static_cast<GridSampler2DBackwardBackward0*>(self->cdata.get())->grid_;
1024:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1025:   return obj.release().ptr();
1026:   END_HANDLE_TH_ERRORS
1027: }
1028: 
1029: static PyObject* THPGridSampler2DBackwardBackward0_input_getter(THPCppFunction *self, void *_unused) {
1030:   HANDLE_TH_ERRORS
1031:   const auto& prop = static_cast<GridSampler2DBackwardBackward0*>(self->cdata.get())->input_;
1032:   return THPVariable_Wrap(prop.unpack(self->cdata));
1033:   END_HANDLE_TH_ERRORS
1034: }
1035: 
1036: static PyObject* THPGridSampler2DBackwardBackward0_input_raw_getter(THPCppFunction *self, void *_unused) {
1037:   HANDLE_TH_ERRORS
1038:   const auto& prop = static_cast<GridSampler2DBackwardBackward0*>(self->cdata.get())->input_;
1039:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1040:   return obj.release().ptr();
```

- EN: The main execution path in this span is carried by `THPGridSampler2DBackwardBackward0_align_corners_getter`, `THPGridSampler2DBackwardBackward0_grad_output_getter`, `THPVariable_Wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPGridSampler2DBackwardBackward0_align_corners_getter`, `THPGridSampler2DBackwardBackward0_grad_output_getter`, `THPVariable_Wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1041-1120

```cpp
1041:   END_HANDLE_TH_ERRORS
1042: }
1043: 
1044: static PyObject* THPGridSampler2DBackwardBackward0_interpolation_mode_getter(THPCppFunction *self, void *_unused) {
1045:   HANDLE_TH_ERRORS
1046:   auto prop = static_cast<GridSampler2DBackwardBackward0*>(self->cdata.get())->interpolation_mode;
1047:   return PyLong_FromUnsignedLong((int64_t) prop);
1048:   END_HANDLE_TH_ERRORS
1049: }
1050: 
1051: static PyObject* THPGridSampler2DBackwardBackward0_padding_mode_getter(THPCppFunction *self, void *_unused) {
1052:   HANDLE_TH_ERRORS
1053:   auto prop = static_cast<GridSampler2DBackwardBackward0*>(self->cdata.get())->padding_mode;
1054:   return PyLong_FromUnsignedLong((int64_t) prop);
1055:   END_HANDLE_TH_ERRORS
1056: }
1057: 
1058: static struct PyGetSetDef GridSampler2DBackwardBackward0_properties[] = {
1059:   THP_FUNCTION_DEFAULT_PROPERTIES,
1060:   {(char*)"_saved_align_corners", (getter)THPGridSampler2DBackwardBackward0_align_corners_getter, nullptr, nullptr, nullptr},
1061:   {(char*)"_saved_grad_output", (getter)THPGridSampler2DBackwardBackward0_grad_output_getter, nullptr, nullptr, nullptr},
1062:   {(char*)"_raw_saved_grad_output", (getter)THPGridSampler2DBackwardBackward0_grad_output_raw_getter, nullptr, nullptr, nullptr},
1063:   {(char*)"_saved_grid", (getter)THPGridSampler2DBackwardBackward0_grid_getter, nullptr, nullptr, nullptr},
1064:   {(char*)"_raw_saved_grid", (getter)THPGridSampler2DBackwardBackward0_grid_raw_getter, nullptr, nullptr, nullptr},
1065:   {(char*)"_saved_input", (getter)THPGridSampler2DBackwardBackward0_input_getter, nullptr, nullptr, nullptr},
1066:   {(char*)"_raw_saved_input", (getter)THPGridSampler2DBackwardBackward0_input_raw_getter, nullptr, nullptr, nullptr},
1067:   {(char*)"_saved_interpolation_mode", (getter)THPGridSampler2DBackwardBackward0_interpolation_mode_getter, nullptr, nullptr, nullptr},
1068:   {(char*)"_saved_padding_mode", (getter)THPGridSampler2DBackwardBackward0_padding_mode_getter, nullptr, nullptr, nullptr},
1069:   {nullptr} /* sentinel */
1070: };
1071: 
1072: static PyObject* THPSpecialI1EBackward0_self_getter(THPCppFunction *self, void *_unused) {
1073:   HANDLE_TH_ERRORS
1074:   const auto& prop = static_cast<SpecialI1EBackward0*>(self->cdata.get())->self_;
1075:   return THPVariable_Wrap(prop.unpack(self->cdata));
1076:   END_HANDLE_TH_ERRORS
1077: }
1078: 
1079: static PyObject* THPSpecialI1EBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
1080:   HANDLE_TH_ERRORS
1081:   const auto& prop = static_cast<SpecialI1EBackward0*>(self->cdata.get())->self_;
1082:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1083:   return obj.release().ptr();
1084:   END_HANDLE_TH_ERRORS
1085: }
1086: 
1087: static PyObject* THPSpecialI1EBackward0_result_getter(THPCppFunction *self, void *_unused) {
1088:   HANDLE_TH_ERRORS
1089:   const auto& prop = static_cast<SpecialI1EBackward0*>(self->cdata.get())->result_;
1090:   return THPVariable_Wrap(prop.unpack(self->cdata));
1091:   END_HANDLE_TH_ERRORS
1092: }
1093: 
1094: static PyObject* THPSpecialI1EBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
1095:   HANDLE_TH_ERRORS
1096:   const auto& prop = static_cast<SpecialI1EBackward0*>(self->cdata.get())->result_;
1097:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1098:   return obj.release().ptr();
1099:   END_HANDLE_TH_ERRORS
1100: }
1101: 
1102: static struct PyGetSetDef SpecialI1EBackward0_properties[] = {
1103:   THP_FUNCTION_DEFAULT_PROPERTIES,
1104:   {(char*)"_saved_self", (getter)THPSpecialI1EBackward0_self_getter, nullptr, nullptr, nullptr},
1105:   {(char*)"_raw_saved_self", (getter)THPSpecialI1EBackward0_self_raw_getter, nullptr, nullptr, nullptr},
1106:   {(char*)"_saved_result", (getter)THPSpecialI1EBackward0_result_getter, nullptr, nullptr, nullptr},
1107:   {(char*)"_raw_saved_result", (getter)THPSpecialI1EBackward0_result_raw_getter, nullptr, nullptr, nullptr},
1108:   {nullptr} /* sentinel */
1109: };
1110: 
1111: static PyObject* THPIgammaBackward0_other_getter(THPCppFunction *self, void *_unused) {
1112:   HANDLE_TH_ERRORS
1113:   const auto& prop = static_cast<IgammaBackward0*>(self->cdata.get())->other_;
1114:   return THPVariable_Wrap(prop.unpack(self->cdata));
1115:   END_HANDLE_TH_ERRORS
1116: }
1117: 
1118: static PyObject* THPIgammaBackward0_other_raw_getter(THPCppFunction *self, void *_unused) {
1119:   HANDLE_TH_ERRORS
1120:   const auto& prop = static_cast<IgammaBackward0*>(self->cdata.get())->other_;
```

- EN: The main execution path in this span is carried by `THPGridSampler2DBackwardBackward0_interpolation_mode_getter`, `PyLong_FromUnsignedLong`, `THPGridSampler2DBackwardBackward0_padding_mode_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPGridSampler2DBackwardBackward0_interpolation_mode_getter`, `PyLong_FromUnsignedLong`, `THPGridSampler2DBackwardBackward0_padding_mode_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1121-1200

```cpp
1121:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1122:   return obj.release().ptr();
1123:   END_HANDLE_TH_ERRORS
1124: }
1125: 
1126: static PyObject* THPIgammaBackward0_self_getter(THPCppFunction *self, void *_unused) {
1127:   HANDLE_TH_ERRORS
1128:   const auto& prop = static_cast<IgammaBackward0*>(self->cdata.get())->self_;
1129:   return THPVariable_Wrap(prop.unpack(self->cdata));
1130:   END_HANDLE_TH_ERRORS
1131: }
1132: 
1133: static PyObject* THPIgammaBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
1134:   HANDLE_TH_ERRORS
1135:   const auto& prop = static_cast<IgammaBackward0*>(self->cdata.get())->self_;
1136:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1137:   return obj.release().ptr();
1138:   END_HANDLE_TH_ERRORS
1139: }
1140: 
1141: static struct PyGetSetDef IgammaBackward0_properties[] = {
1142:   THP_FUNCTION_DEFAULT_PROPERTIES,
1143:   {(char*)"_saved_other", (getter)THPIgammaBackward0_other_getter, nullptr, nullptr, nullptr},
1144:   {(char*)"_raw_saved_other", (getter)THPIgammaBackward0_other_raw_getter, nullptr, nullptr, nullptr},
1145:   {(char*)"_saved_self", (getter)THPIgammaBackward0_self_getter, nullptr, nullptr, nullptr},
1146:   {(char*)"_raw_saved_self", (getter)THPIgammaBackward0_self_raw_getter, nullptr, nullptr, nullptr},
1147:   {nullptr} /* sentinel */
1148: };
1149: 
1150: static PyObject* THPIndexCopyBackward0_dim_getter(THPCppFunction *self, void *_unused) {
1151:   HANDLE_TH_ERRORS
1152:   auto prop = static_cast<IndexCopyBackward0*>(self->cdata.get())->dim;
1153:   return PyLong_FromUnsignedLong((int64_t) prop);
1154:   END_HANDLE_TH_ERRORS
1155: }
1156: 
1157: static PyObject* THPIndexCopyBackward0_index_getter(THPCppFunction *self, void *_unused) {
1158:   HANDLE_TH_ERRORS
1159:   const auto& prop = static_cast<IndexCopyBackward0*>(self->cdata.get())->index_;
1160:   return THPVariable_Wrap(prop.unpack(self->cdata));
1161:   END_HANDLE_TH_ERRORS
1162: }
1163: 
1164: static PyObject* THPIndexCopyBackward0_index_raw_getter(THPCppFunction *self, void *_unused) {
1165:   HANDLE_TH_ERRORS
1166:   const auto& prop = static_cast<IndexCopyBackward0*>(self->cdata.get())->index_;
1167:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1168:   return obj.release().ptr();
1169:   END_HANDLE_TH_ERRORS
1170: }
1171: 
1172: static PyObject* THPIndexCopyBackward0_source_getter(THPCppFunction *self, void *_unused) {
1173:   HANDLE_TH_ERRORS
1174:   const auto& prop = static_cast<IndexCopyBackward0*>(self->cdata.get())->source_;
1175:   return THPVariable_Wrap(prop.unpack(self->cdata));
1176:   END_HANDLE_TH_ERRORS
1177: }
1178: 
1179: static PyObject* THPIndexCopyBackward0_source_raw_getter(THPCppFunction *self, void *_unused) {
1180:   HANDLE_TH_ERRORS
1181:   const auto& prop = static_cast<IndexCopyBackward0*>(self->cdata.get())->source_;
1182:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1183:   return obj.release().ptr();
1184:   END_HANDLE_TH_ERRORS
1185: }
1186: 
1187: static PyObject* THPIndexCopyBackward0_source_dim_getter(THPCppFunction *self, void *_unused) {
1188:   HANDLE_TH_ERRORS
1189:   auto prop = static_cast<IndexCopyBackward0*>(self->cdata.get())->source_dim;
1190:   return PyLong_FromUnsignedLong((int64_t) prop);
1191:   END_HANDLE_TH_ERRORS
1192: }
1193: 
1194: static struct PyGetSetDef IndexCopyBackward0_properties[] = {
1195:   THP_FUNCTION_DEFAULT_PROPERTIES,
1196:   {(char*)"_saved_dim", (getter)THPIndexCopyBackward0_dim_getter, nullptr, nullptr, nullptr},
1197:   {(char*)"_saved_index", (getter)THPIndexCopyBackward0_index_getter, nullptr, nullptr, nullptr},
1198:   {(char*)"_raw_saved_index", (getter)THPIndexCopyBackward0_index_raw_getter, nullptr, nullptr, nullptr},
1199:   {(char*)"_saved_source", (getter)THPIndexCopyBackward0_source_getter, nullptr, nullptr, nullptr},
1200:   {(char*)"_raw_saved_source", (getter)THPIndexCopyBackward0_source_raw_getter, nullptr, nullptr, nullptr},
```

- EN: The main execution path in this span is carried by `cast`, `THPIgammaBackward0_self_getter`, `THPVariable_Wrap`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `cast`, `THPIgammaBackward0_self_getter`, `THPVariable_Wrap` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1201-1280

```cpp
1201:   {(char*)"_saved_source_dim", (getter)THPIndexCopyBackward0_source_dim_getter, nullptr, nullptr, nullptr},
1202:   {nullptr} /* sentinel */
1203: };
1204: 
1205: static PyObject* THPUnsafeIndexPutBackward0_accumulate_getter(THPCppFunction *self, void *_unused) {
1206:   HANDLE_TH_ERRORS
1207:   auto prop = static_cast<UnsafeIndexPutBackward0*>(self->cdata.get())->accumulate;
1208:   if (prop) {
1209:     Py_RETURN_TRUE;
1210:   } else {
1211:     Py_RETURN_FALSE;
1212:   }
1213:   END_HANDLE_TH_ERRORS
1214: }
1215: 
1216: static PyObject* THPUnsafeIndexPutBackward0_indices_getter(THPCppFunction *self, void *_unused) {
1217:   HANDLE_TH_ERRORS
1218:   const auto *node = static_cast<UnsafeIndexPutBackward0*>(self->cdata.get());
1219:   const auto& prop = node->indices_;
1220:   if (node->indices_released_) {
1221:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
1222:     return nullptr;
1223:   }
1224:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
1225:   for (auto i: c10::irange(prop.size())) {
1226:     PyTuple_SetItem(tup, (Py_ssize_t) i, THPVariable_Wrap(prop[i].unpack(self->cdata)));
1227:   }
1228:   return tup;
1229:   END_HANDLE_TH_ERRORS
1230: }
1231: 
1232: static PyObject* THPUnsafeIndexPutBackward0_indices_raw_getter(THPCppFunction *self, void *_unused) {
1233:   HANDLE_TH_ERRORS
1234:   const auto *node = static_cast<UnsafeIndexPutBackward0*>(self->cdata.get());
1235:   const auto& prop = node->indices_;
1236:   if (node->indices_released_) {
1237:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
1238:     return nullptr;
1239:   }
1240:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
1241:   for (auto i : c10::irange(prop.size())) {
1242:     pybind11::object obj = pybind11::cast(prop[i], pybind11::return_value_policy::reference);
1243:     PyTuple_SetItem(tup, (Py_ssize_t) i, obj.release().ptr());
1244:   }
1245:   return tup;
1246:   END_HANDLE_TH_ERRORS
1247: }
1248: 
1249: static struct PyGetSetDef UnsafeIndexPutBackward0_properties[] = {
1250:   THP_FUNCTION_DEFAULT_PROPERTIES,
1251:   {(char*)"_saved_accumulate", (getter)THPUnsafeIndexPutBackward0_accumulate_getter, nullptr, nullptr, nullptr},
1252:   {(char*)"_saved_indices", (getter)THPUnsafeIndexPutBackward0_indices_getter, nullptr, nullptr, nullptr},
1253:   {(char*)"_raw_saved_indices", (getter)THPUnsafeIndexPutBackward0_indices_raw_getter, nullptr, nullptr, nullptr},
1254:   {nullptr} /* sentinel */
1255: };
1256: 
1257: static PyObject* THPIndexSelectBackward0_dim_getter(THPCppFunction *self, void *_unused) {
1258:   HANDLE_TH_ERRORS
1259:   auto prop = static_cast<IndexSelectBackward0*>(self->cdata.get())->dim;
1260:   return PyLong_FromUnsignedLong((int64_t) prop);
1261:   END_HANDLE_TH_ERRORS
1262: }
1263: 
1264: static PyObject* THPIndexSelectBackward0_index_getter(THPCppFunction *self, void *_unused) {
1265:   HANDLE_TH_ERRORS
1266:   const auto& prop = static_cast<IndexSelectBackward0*>(self->cdata.get())->index_;
1267:   return THPVariable_Wrap(prop.unpack(self->cdata));
1268:   END_HANDLE_TH_ERRORS
1269: }
1270: 
1271: static PyObject* THPIndexSelectBackward0_index_raw_getter(THPCppFunction *self, void *_unused) {
1272:   HANDLE_TH_ERRORS
1273:   const auto& prop = static_cast<IndexSelectBackward0*>(self->cdata.get())->index_;
1274:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1275:   return obj.release().ptr();
1276:   END_HANDLE_TH_ERRORS
1277: }
1278: 
1279: static PyObject* THPIndexSelectBackward0_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
1280:   HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPUnsafeIndexPutBackward0_accumulate_getter`, `THPUnsafeIndexPutBackward0_indices_getter`, `PyErr_SetString`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPUnsafeIndexPutBackward0_accumulate_getter`, `THPUnsafeIndexPutBackward0_indices_getter`, `PyErr_SetString` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1281-1360

```cpp
1281:   auto prop = static_cast<IndexSelectBackward0*>(self->cdata.get())->self_sym_sizes;
1282:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
1283:   for (auto i : c10::irange(prop.size())) {
1284:       auto si = prop[i];
1285:       if (auto m = si.maybe_as_int()) {
1286:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
1287:       } else {
1288:         auto py_symint = py::cast(si).release().ptr();
1289:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
1290:       }
1291:   }
1292:   return tup;
1293:   END_HANDLE_TH_ERRORS
1294: }
1295: 
1296: static struct PyGetSetDef IndexSelectBackward0_properties[] = {
1297:   THP_FUNCTION_DEFAULT_PROPERTIES,
1298:   {(char*)"_saved_dim", (getter)THPIndexSelectBackward0_dim_getter, nullptr, nullptr, nullptr},
1299:   {(char*)"_saved_index", (getter)THPIndexSelectBackward0_index_getter, nullptr, nullptr, nullptr},
1300:   {(char*)"_raw_saved_index", (getter)THPIndexSelectBackward0_index_raw_getter, nullptr, nullptr, nullptr},
1301:   {(char*)"_saved_self_sym_sizes", (getter)THPIndexSelectBackward0_self_sym_sizes_getter, nullptr, nullptr, nullptr},
1302:   {nullptr} /* sentinel */
1303: };
1304: 
1305: 
1306: 
1307: static struct PyGetSetDef LeBackward0_properties[] = {
1308:   THP_FUNCTION_DEFAULT_PROPERTIES,
1309: 
1310:   {nullptr} /* sentinel */
1311: };
1312: 
1313: 
1314: 
1315: static struct PyGetSetDef LeBackward1_properties[] = {
1316:   THP_FUNCTION_DEFAULT_PROPERTIES,
1317: 
1318:   {nullptr} /* sentinel */
1319: };
1320: 
1321: static PyObject* THPLgammaBackward0_self_getter(THPCppFunction *self, void *_unused) {
1322:   HANDLE_TH_ERRORS
1323:   const auto& prop = static_cast<LgammaBackward0*>(self->cdata.get())->self_;
1324:   return THPVariable_Wrap(prop.unpack(self->cdata));
1325:   END_HANDLE_TH_ERRORS
1326: }
1327: 
1328: static PyObject* THPLgammaBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
1329:   HANDLE_TH_ERRORS
1330:   const auto& prop = static_cast<LgammaBackward0*>(self->cdata.get())->self_;
1331:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1332:   return obj.release().ptr();
1333:   END_HANDLE_TH_ERRORS
1334: }
1335: 
1336: static struct PyGetSetDef LgammaBackward0_properties[] = {
1337:   THP_FUNCTION_DEFAULT_PROPERTIES,
1338:   {(char*)"_saved_self", (getter)THPLgammaBackward0_self_getter, nullptr, nullptr, nullptr},
1339:   {(char*)"_raw_saved_self", (getter)THPLgammaBackward0_self_raw_getter, nullptr, nullptr, nullptr},
1340:   {nullptr} /* sentinel */
1341: };
1342: 
1343: static PyObject* THPLog10Backward0_self_getter(THPCppFunction *self, void *_unused) {
1344:   HANDLE_TH_ERRORS
1345:   const auto& prop = static_cast<Log10Backward0*>(self->cdata.get())->self_;
1346:   return THPVariable_Wrap(prop.unpack(self->cdata));
1347:   END_HANDLE_TH_ERRORS
1348: }
1349: 
1350: static PyObject* THPLog10Backward0_self_raw_getter(THPCppFunction *self, void *_unused) {
1351:   HANDLE_TH_ERRORS
1352:   const auto& prop = static_cast<Log10Backward0*>(self->cdata.get())->self_;
1353:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1354:   return obj.release().ptr();
1355:   END_HANDLE_TH_ERRORS
1356: }
1357: 
1358: static struct PyGetSetDef Log10Backward0_properties[] = {
1359:   THP_FUNCTION_DEFAULT_PROPERTIES,
1360:   {(char*)"_saved_self", (getter)THPLog10Backward0_self_getter, nullptr, nullptr, nullptr},
```

- EN: The main execution path in this span is carried by `PyTuple_New`, `PyTuple_SetItem`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyTuple_New`, `PyTuple_SetItem`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1361-1440

```cpp
1361:   {(char*)"_raw_saved_self", (getter)THPLog10Backward0_self_raw_getter, nullptr, nullptr, nullptr},
1362:   {nullptr} /* sentinel */
1363: };
1364: 
1365: static PyObject* THPLog1PBackward0_self_getter(THPCppFunction *self, void *_unused) {
1366:   HANDLE_TH_ERRORS
1367:   const auto& prop = static_cast<Log1PBackward0*>(self->cdata.get())->self_;
1368:   return THPVariable_Wrap(prop.unpack(self->cdata));
1369:   END_HANDLE_TH_ERRORS
1370: }
1371: 
1372: static PyObject* THPLog1PBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
1373:   HANDLE_TH_ERRORS
1374:   const auto& prop = static_cast<Log1PBackward0*>(self->cdata.get())->self_;
1375:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1376:   return obj.release().ptr();
1377:   END_HANDLE_TH_ERRORS
1378: }
1379: 
1380: static struct PyGetSetDef Log1PBackward0_properties[] = {
1381:   THP_FUNCTION_DEFAULT_PROPERTIES,
1382:   {(char*)"_saved_self", (getter)THPLog1PBackward0_self_getter, nullptr, nullptr, nullptr},
1383:   {(char*)"_raw_saved_self", (getter)THPLog1PBackward0_self_raw_getter, nullptr, nullptr, nullptr},
1384:   {nullptr} /* sentinel */
1385: };
1386: 
1387: static PyObject* THPLogaddexp2Backward0_other_getter(THPCppFunction *self, void *_unused) {
1388:   HANDLE_TH_ERRORS
1389:   const auto& prop = static_cast<Logaddexp2Backward0*>(self->cdata.get())->other_;
1390:   return THPVariable_Wrap(prop.unpack(self->cdata));
1391:   END_HANDLE_TH_ERRORS
1392: }
1393: 
1394: static PyObject* THPLogaddexp2Backward0_other_raw_getter(THPCppFunction *self, void *_unused) {
1395:   HANDLE_TH_ERRORS
1396:   const auto& prop = static_cast<Logaddexp2Backward0*>(self->cdata.get())->other_;
1397:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1398:   return obj.release().ptr();
1399:   END_HANDLE_TH_ERRORS
1400: }
1401: 
1402: static PyObject* THPLogaddexp2Backward0_self_getter(THPCppFunction *self, void *_unused) {
1403:   HANDLE_TH_ERRORS
1404:   const auto& prop = static_cast<Logaddexp2Backward0*>(self->cdata.get())->self_;
1405:   return THPVariable_Wrap(prop.unpack(self->cdata));
1406:   END_HANDLE_TH_ERRORS
1407: }
1408: 
1409: static PyObject* THPLogaddexp2Backward0_self_raw_getter(THPCppFunction *self, void *_unused) {
1410:   HANDLE_TH_ERRORS
1411:   const auto& prop = static_cast<Logaddexp2Backward0*>(self->cdata.get())->self_;
1412:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1413:   return obj.release().ptr();
1414:   END_HANDLE_TH_ERRORS
1415: }
1416: 
1417: static struct PyGetSetDef Logaddexp2Backward0_properties[] = {
1418:   THP_FUNCTION_DEFAULT_PROPERTIES,
1419:   {(char*)"_saved_other", (getter)THPLogaddexp2Backward0_other_getter, nullptr, nullptr, nullptr},
1420:   {(char*)"_raw_saved_other", (getter)THPLogaddexp2Backward0_other_raw_getter, nullptr, nullptr, nullptr},
1421:   {(char*)"_saved_self", (getter)THPLogaddexp2Backward0_self_getter, nullptr, nullptr, nullptr},
1422:   {(char*)"_raw_saved_self", (getter)THPLogaddexp2Backward0_self_raw_getter, nullptr, nullptr, nullptr},
1423:   {nullptr} /* sentinel */
1424: };
1425: 
1426: static PyObject* THPLogsumexpBackward0_dim_getter(THPCppFunction *self, void *_unused) {
1427:   HANDLE_TH_ERRORS
1428:   auto prop = static_cast<LogsumexpBackward0*>(self->cdata.get())->dim;
1429:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
1430:   for (auto i : c10::irange(prop.size())) {
1431:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
1432:   }
1433:   return tup;
1434:   END_HANDLE_TH_ERRORS
1435: }
1436: 
1437: static PyObject* THPLogsumexpBackward0_keepdim_getter(THPCppFunction *self, void *_unused) {
1438:   HANDLE_TH_ERRORS
1439:   auto prop = static_cast<LogsumexpBackward0*>(self->cdata.get())->keepdim;
1440:   if (prop) {
```

- EN: The main execution path in this span is carried by `THPLog1PBackward0_self_getter`, `THPVariable_Wrap`, `THPLog1PBackward0_self_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPLog1PBackward0_self_getter`, `THPVariable_Wrap`, `THPLog1PBackward0_self_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1441-1520

```cpp
1441:     Py_RETURN_TRUE;
1442:   } else {
1443:     Py_RETURN_FALSE;
1444:   }
1445:   END_HANDLE_TH_ERRORS
1446: }
1447: 
1448: static PyObject* THPLogsumexpBackward0_self_getter(THPCppFunction *self, void *_unused) {
1449:   HANDLE_TH_ERRORS
1450:   const auto& prop = static_cast<LogsumexpBackward0*>(self->cdata.get())->self_;
1451:   return THPVariable_Wrap(prop.unpack(self->cdata));
1452:   END_HANDLE_TH_ERRORS
1453: }
1454: 
1455: static PyObject* THPLogsumexpBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
1456:   HANDLE_TH_ERRORS
1457:   const auto& prop = static_cast<LogsumexpBackward0*>(self->cdata.get())->self_;
1458:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1459:   return obj.release().ptr();
1460:   END_HANDLE_TH_ERRORS
1461: }
1462: 
1463: static PyObject* THPLogsumexpBackward0_result_getter(THPCppFunction *self, void *_unused) {
1464:   HANDLE_TH_ERRORS
1465:   const auto& prop = static_cast<LogsumexpBackward0*>(self->cdata.get())->result_;
1466:   return THPVariable_Wrap(prop.unpack(self->cdata));
1467:   END_HANDLE_TH_ERRORS
1468: }
1469: 
1470: static PyObject* THPLogsumexpBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
1471:   HANDLE_TH_ERRORS
1472:   const auto& prop = static_cast<LogsumexpBackward0*>(self->cdata.get())->result_;
1473:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1474:   return obj.release().ptr();
1475:   END_HANDLE_TH_ERRORS
1476: }
1477: 
1478: static struct PyGetSetDef LogsumexpBackward0_properties[] = {
1479:   THP_FUNCTION_DEFAULT_PROPERTIES,
1480:   {(char*)"_saved_dim", (getter)THPLogsumexpBackward0_dim_getter, nullptr, nullptr, nullptr},
1481:   {(char*)"_saved_keepdim", (getter)THPLogsumexpBackward0_keepdim_getter, nullptr, nullptr, nullptr},
1482:   {(char*)"_saved_self", (getter)THPLogsumexpBackward0_self_getter, nullptr, nullptr, nullptr},
1483:   {(char*)"_raw_saved_self", (getter)THPLogsumexpBackward0_self_raw_getter, nullptr, nullptr, nullptr},
1484:   {(char*)"_saved_result", (getter)THPLogsumexpBackward0_result_getter, nullptr, nullptr, nullptr},
1485:   {(char*)"_raw_saved_result", (getter)THPLogsumexpBackward0_result_raw_getter, nullptr, nullptr, nullptr},
1486:   {nullptr} /* sentinel */
1487: };
1488: 
1489: static PyObject* THPLinalgLstsqBackward0_b_getter(THPCppFunction *self, void *_unused) {
1490:   HANDLE_TH_ERRORS
1491:   const auto& prop = static_cast<LinalgLstsqBackward0*>(self->cdata.get())->b_;
1492:   return THPVariable_Wrap(prop.unpack(self->cdata));
1493:   END_HANDLE_TH_ERRORS
1494: }
1495: 
1496: static PyObject* THPLinalgLstsqBackward0_b_raw_getter(THPCppFunction *self, void *_unused) {
1497:   HANDLE_TH_ERRORS
1498:   const auto& prop = static_cast<LinalgLstsqBackward0*>(self->cdata.get())->b_;
1499:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1500:   return obj.release().ptr();
1501:   END_HANDLE_TH_ERRORS
1502: }
1503: 
1504: static PyObject* THPLinalgLstsqBackward0_self_getter(THPCppFunction *self, void *_unused) {
1505:   HANDLE_TH_ERRORS
1506:   const auto& prop = static_cast<LinalgLstsqBackward0*>(self->cdata.get())->self_;
1507:   return THPVariable_Wrap(prop.unpack(self->cdata));
1508:   END_HANDLE_TH_ERRORS
1509: }
1510: 
1511: static PyObject* THPLinalgLstsqBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
1512:   HANDLE_TH_ERRORS
1513:   const auto& prop = static_cast<LinalgLstsqBackward0*>(self->cdata.get())->self_;
1514:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1515:   return obj.release().ptr();
1516:   END_HANDLE_TH_ERRORS
1517: }
1518: 
1519: static PyObject* THPLinalgLstsqBackward0_solution_getter(THPCppFunction *self, void *_unused) {
1520:   HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPLogsumexpBackward0_self_getter`, `THPVariable_Wrap`, `THPLogsumexpBackward0_self_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPLogsumexpBackward0_self_getter`, `THPVariable_Wrap`, `THPLogsumexpBackward0_self_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1521-1600

```cpp
1521:   const auto& prop = static_cast<LinalgLstsqBackward0*>(self->cdata.get())->solution_;
1522:   return THPVariable_Wrap(prop.unpack(self->cdata));
1523:   END_HANDLE_TH_ERRORS
1524: }
1525: 
1526: static PyObject* THPLinalgLstsqBackward0_solution_raw_getter(THPCppFunction *self, void *_unused) {
1527:   HANDLE_TH_ERRORS
1528:   const auto& prop = static_cast<LinalgLstsqBackward0*>(self->cdata.get())->solution_;
1529:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1530:   return obj.release().ptr();
1531:   END_HANDLE_TH_ERRORS
1532: }
1533: 
1534: static struct PyGetSetDef LinalgLstsqBackward0_properties[] = {
1535:   THP_FUNCTION_DEFAULT_PROPERTIES,
1536:   {(char*)"_saved_b", (getter)THPLinalgLstsqBackward0_b_getter, nullptr, nullptr, nullptr},
1537:   {(char*)"_raw_saved_b", (getter)THPLinalgLstsqBackward0_b_raw_getter, nullptr, nullptr, nullptr},
1538:   {(char*)"_saved_self", (getter)THPLinalgLstsqBackward0_self_getter, nullptr, nullptr, nullptr},
1539:   {(char*)"_raw_saved_self", (getter)THPLinalgLstsqBackward0_self_raw_getter, nullptr, nullptr, nullptr},
1540:   {(char*)"_saved_solution", (getter)THPLinalgLstsqBackward0_solution_getter, nullptr, nullptr, nullptr},
1541:   {(char*)"_raw_saved_solution", (getter)THPLinalgLstsqBackward0_solution_raw_getter, nullptr, nullptr, nullptr},
1542:   {nullptr} /* sentinel */
1543: };
1544: 
1545: static PyObject* THPLinalgLuBackward0_pivot_getter(THPCppFunction *self, void *_unused) {
1546:   HANDLE_TH_ERRORS
1547:   auto prop = static_cast<LinalgLuBackward0*>(self->cdata.get())->pivot;
1548:   if (prop) {
1549:     Py_RETURN_TRUE;
1550:   } else {
1551:     Py_RETURN_FALSE;
1552:   }
1553:   END_HANDLE_TH_ERRORS
1554: }
1555: 
1556: static PyObject* THPLinalgLuBackward0_L_getter(THPCppFunction *self, void *_unused) {
1557:   HANDLE_TH_ERRORS
1558:   const auto& prop = static_cast<LinalgLuBackward0*>(self->cdata.get())->L_;
1559:   return THPVariable_Wrap(prop.unpack(self->cdata));
1560:   END_HANDLE_TH_ERRORS
1561: }
1562: 
1563: static PyObject* THPLinalgLuBackward0_L_raw_getter(THPCppFunction *self, void *_unused) {
1564:   HANDLE_TH_ERRORS
1565:   const auto& prop = static_cast<LinalgLuBackward0*>(self->cdata.get())->L_;
1566:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1567:   return obj.release().ptr();
1568:   END_HANDLE_TH_ERRORS
1569: }
1570: 
1571: static PyObject* THPLinalgLuBackward0_P_getter(THPCppFunction *self, void *_unused) {
1572:   HANDLE_TH_ERRORS
1573:   const auto& prop = static_cast<LinalgLuBackward0*>(self->cdata.get())->P_;
1574:   return THPVariable_Wrap(prop.unpack(self->cdata));
1575:   END_HANDLE_TH_ERRORS
1576: }
1577: 
1578: static PyObject* THPLinalgLuBackward0_P_raw_getter(THPCppFunction *self, void *_unused) {
1579:   HANDLE_TH_ERRORS
1580:   const auto& prop = static_cast<LinalgLuBackward0*>(self->cdata.get())->P_;
1581:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1582:   return obj.release().ptr();
1583:   END_HANDLE_TH_ERRORS
1584: }
1585: 
1586: static PyObject* THPLinalgLuBackward0_U_getter(THPCppFunction *self, void *_unused) {
1587:   HANDLE_TH_ERRORS
1588:   const auto& prop = static_cast<LinalgLuBackward0*>(self->cdata.get())->U_;
1589:   return THPVariable_Wrap(prop.unpack(self->cdata));
1590:   END_HANDLE_TH_ERRORS
1591: }
1592: 
1593: static PyObject* THPLinalgLuBackward0_U_raw_getter(THPCppFunction *self, void *_unused) {
1594:   HANDLE_TH_ERRORS
1595:   const auto& prop = static_cast<LinalgLuBackward0*>(self->cdata.get())->U_;
1596:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1597:   return obj.release().ptr();
1598:   END_HANDLE_TH_ERRORS
1599: }
1600: 
```

- EN: The main execution path in this span is carried by `THPVariable_Wrap`, `THPLinalgLstsqBackward0_solution_raw_getter`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_Wrap`, `THPLinalgLstsqBackward0_solution_raw_getter`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1601-1680

```cpp
1601: static struct PyGetSetDef LinalgLuBackward0_properties[] = {
1602:   THP_FUNCTION_DEFAULT_PROPERTIES,
1603:   {(char*)"_saved_pivot", (getter)THPLinalgLuBackward0_pivot_getter, nullptr, nullptr, nullptr},
1604:   {(char*)"_saved_L", (getter)THPLinalgLuBackward0_L_getter, nullptr, nullptr, nullptr},
1605:   {(char*)"_raw_saved_L", (getter)THPLinalgLuBackward0_L_raw_getter, nullptr, nullptr, nullptr},
1606:   {(char*)"_saved_P", (getter)THPLinalgLuBackward0_P_getter, nullptr, nullptr, nullptr},
1607:   {(char*)"_raw_saved_P", (getter)THPLinalgLuBackward0_P_raw_getter, nullptr, nullptr, nullptr},
1608:   {(char*)"_saved_U", (getter)THPLinalgLuBackward0_U_getter, nullptr, nullptr, nullptr},
1609:   {(char*)"_raw_saved_U", (getter)THPLinalgLuBackward0_U_raw_getter, nullptr, nullptr, nullptr},
1610:   {nullptr} /* sentinel */
1611: };
1612: 
1613: static PyObject* THPLuUnpackBackward0_LU_data_sym_argsize_minus_1_getter(THPCppFunction *self, void *_unused) {
1614:   HANDLE_TH_ERRORS
1615:   auto prop = static_cast<LuUnpackBackward0*>(self->cdata.get())->LU_data_sym_argsize_minus_1;
1616:   if (auto m = prop.maybe_as_int()) {
1617:     return PyLong_FromUnsignedLong(*m);
1618:   } else {
1619:     return py::cast(prop).release().ptr();
1620:   }
1621:   END_HANDLE_TH_ERRORS
1622: }
1623: 
1624: static PyObject* THPLuUnpackBackward0_LU_data_sym_argsize_minus_2_getter(THPCppFunction *self, void *_unused) {
1625:   HANDLE_TH_ERRORS
1626:   auto prop = static_cast<LuUnpackBackward0*>(self->cdata.get())->LU_data_sym_argsize_minus_2;
1627:   if (auto m = prop.maybe_as_int()) {
1628:     return PyLong_FromUnsignedLong(*m);
1629:   } else {
1630:     return py::cast(prop).release().ptr();
1631:   }
1632:   END_HANDLE_TH_ERRORS
1633: }
1634: 
1635: static struct PyGetSetDef LuUnpackBackward0_properties[] = {
1636:   THP_FUNCTION_DEFAULT_PROPERTIES,
1637:   {(char*)"_saved_LU_data_sym_argsize_minus_1", (getter)THPLuUnpackBackward0_LU_data_sym_argsize_minus_1_getter, nullptr, nullptr, nullptr},
1638:   {(char*)"_saved_LU_data_sym_argsize_minus_2", (getter)THPLuUnpackBackward0_LU_data_sym_argsize_minus_2_getter, nullptr, nullptr, nullptr},
1639:   {nullptr} /* sentinel */
1640: };
1641: 
1642: static PyObject* THPLinalgMatrixExpBackward0_self_getter(THPCppFunction *self, void *_unused) {
1643:   HANDLE_TH_ERRORS
1644:   const auto& prop = static_cast<LinalgMatrixExpBackward0*>(self->cdata.get())->self_;
1645:   return THPVariable_Wrap(prop.unpack(self->cdata));
1646:   END_HANDLE_TH_ERRORS
1647: }
1648: 
1649: static PyObject* THPLinalgMatrixExpBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
1650:   HANDLE_TH_ERRORS
1651:   const auto& prop = static_cast<LinalgMatrixExpBackward0*>(self->cdata.get())->self_;
1652:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1653:   return obj.release().ptr();
1654:   END_HANDLE_TH_ERRORS
1655: }
1656: 
1657: static struct PyGetSetDef LinalgMatrixExpBackward0_properties[] = {
1658:   THP_FUNCTION_DEFAULT_PROPERTIES,
1659:   {(char*)"_saved_self", (getter)THPLinalgMatrixExpBackward0_self_getter, nullptr, nullptr, nullptr},
1660:   {(char*)"_raw_saved_self", (getter)THPLinalgMatrixExpBackward0_self_raw_getter, nullptr, nullptr, nullptr},
1661:   {nullptr} /* sentinel */
1662: };
1663: 
1664: static PyObject* THPMaxBackward0_dim_getter(THPCppFunction *self, void *_unused) {
1665:   HANDLE_TH_ERRORS
1666:   auto prop = static_cast<MaxBackward0*>(self->cdata.get())->dim;
1667:   return PyLong_FromUnsignedLong((int64_t) prop);
1668:   END_HANDLE_TH_ERRORS
1669: }
1670: 
1671: static PyObject* THPMaxBackward0_keepdim_getter(THPCppFunction *self, void *_unused) {
1672:   HANDLE_TH_ERRORS
1673:   auto prop = static_cast<MaxBackward0*>(self->cdata.get())->keepdim;
1674:   if (prop) {
1675:     Py_RETURN_TRUE;
1676:   } else {
1677:     Py_RETURN_FALSE;
1678:   }
1679:   END_HANDLE_TH_ERRORS
1680: }
```

- EN: The main execution path in this span is carried by `THPLuUnpackBackward0_LU_data_sym_argsize_minus_1_getter`, `PyLong_FromUnsignedLong`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPLuUnpackBackward0_LU_data_sym_argsize_minus_1_getter`, `PyLong_FromUnsignedLong`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1681-1760

```cpp
1681: 
1682: static PyObject* THPMaxBackward0_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
1683:   HANDLE_TH_ERRORS
1684:   auto prop = static_cast<MaxBackward0*>(self->cdata.get())->self_sym_sizes;
1685:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
1686:   for (auto i : c10::irange(prop.size())) {
1687:       auto si = prop[i];
1688:       if (auto m = si.maybe_as_int()) {
1689:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
1690:       } else {
1691:         auto py_symint = py::cast(si).release().ptr();
1692:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
1693:       }
1694:   }
1695:   return tup;
1696:   END_HANDLE_TH_ERRORS
1697: }
1698: 
1699: static PyObject* THPMaxBackward0_indices_getter(THPCppFunction *self, void *_unused) {
1700:   HANDLE_TH_ERRORS
1701:   const auto& prop = static_cast<MaxBackward0*>(self->cdata.get())->indices_;
1702:   return THPVariable_Wrap(prop.unpack(self->cdata));
1703:   END_HANDLE_TH_ERRORS
1704: }
1705: 
1706: static PyObject* THPMaxBackward0_indices_raw_getter(THPCppFunction *self, void *_unused) {
1707:   HANDLE_TH_ERRORS
1708:   const auto& prop = static_cast<MaxBackward0*>(self->cdata.get())->indices_;
1709:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1710:   return obj.release().ptr();
1711:   END_HANDLE_TH_ERRORS
1712: }
1713: 
1714: static struct PyGetSetDef MaxBackward0_properties[] = {
1715:   THP_FUNCTION_DEFAULT_PROPERTIES,
1716:   {(char*)"_saved_dim", (getter)THPMaxBackward0_dim_getter, nullptr, nullptr, nullptr},
1717:   {(char*)"_saved_keepdim", (getter)THPMaxBackward0_keepdim_getter, nullptr, nullptr, nullptr},
1718:   {(char*)"_saved_self_sym_sizes", (getter)THPMaxBackward0_self_sym_sizes_getter, nullptr, nullptr, nullptr},
1719:   {(char*)"_saved_indices", (getter)THPMaxBackward0_indices_getter, nullptr, nullptr, nullptr},
1720:   {(char*)"_raw_saved_indices", (getter)THPMaxBackward0_indices_raw_getter, nullptr, nullptr, nullptr},
1721:   {nullptr} /* sentinel */
1722: };
1723: 
1724: static PyObject* THPMaxBackward1_self_getter(THPCppFunction *self, void *_unused) {
1725:   HANDLE_TH_ERRORS
1726:   const auto& prop = static_cast<MaxBackward1*>(self->cdata.get())->self_;
1727:   return THPVariable_Wrap(prop.unpack(self->cdata));
1728:   END_HANDLE_TH_ERRORS
1729: }
1730: 
1731: static PyObject* THPMaxBackward1_self_raw_getter(THPCppFunction *self, void *_unused) {
1732:   HANDLE_TH_ERRORS
1733:   const auto& prop = static_cast<MaxBackward1*>(self->cdata.get())->self_;
1734:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1735:   return obj.release().ptr();
1736:   END_HANDLE_TH_ERRORS
1737: }
1738: 
1739: static PyObject* THPMaxBackward1_result_getter(THPCppFunction *self, void *_unused) {
1740:   HANDLE_TH_ERRORS
1741:   const auto& prop = static_cast<MaxBackward1*>(self->cdata.get())->result_;
1742:   return THPVariable_Wrap(prop.unpack(self->cdata));
1743:   END_HANDLE_TH_ERRORS
1744: }
1745: 
1746: static PyObject* THPMaxBackward1_result_raw_getter(THPCppFunction *self, void *_unused) {
1747:   HANDLE_TH_ERRORS
1748:   const auto& prop = static_cast<MaxBackward1*>(self->cdata.get())->result_;
1749:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1750:   return obj.release().ptr();
1751:   END_HANDLE_TH_ERRORS
1752: }
1753: 
1754: static struct PyGetSetDef MaxBackward1_properties[] = {
1755:   THP_FUNCTION_DEFAULT_PROPERTIES,
1756:   {(char*)"_saved_self", (getter)THPMaxBackward1_self_getter, nullptr, nullptr, nullptr},
1757:   {(char*)"_raw_saved_self", (getter)THPMaxBackward1_self_raw_getter, nullptr, nullptr, nullptr},
1758:   {(char*)"_saved_result", (getter)THPMaxBackward1_result_getter, nullptr, nullptr, nullptr},
1759:   {(char*)"_raw_saved_result", (getter)THPMaxBackward1_result_raw_getter, nullptr, nullptr, nullptr},
1760:   {nullptr} /* sentinel */
```

- EN: The main execution path in this span is carried by `THPMaxBackward0_self_sym_sizes_getter`, `PyTuple_New`, `PyTuple_SetItem`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPMaxBackward0_self_sym_sizes_getter`, `PyTuple_New`, `PyTuple_SetItem` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1761-1840

```cpp
1761: };
1762: 
1763: static PyObject* THPAminmaxBackward0_dim_getter(THPCppFunction *self, void *_unused) {
1764:   HANDLE_TH_ERRORS
1765:   auto opt_prop = static_cast<AminmaxBackward0*>(self->cdata.get())->dim;
1766:   if (!opt_prop.has_value()) {
1767:     Py_RETURN_NONE;
1768:   }
1769:   auto prop = opt_prop.value();
1770:   return PyLong_FromUnsignedLong((int64_t) prop);
1771:   END_HANDLE_TH_ERRORS
1772: }
1773: 
1774: static PyObject* THPAminmaxBackward0_keepdim_getter(THPCppFunction *self, void *_unused) {
1775:   HANDLE_TH_ERRORS
1776:   auto prop = static_cast<AminmaxBackward0*>(self->cdata.get())->keepdim;
1777:   if (prop) {
1778:     Py_RETURN_TRUE;
1779:   } else {
1780:     Py_RETURN_FALSE;
1781:   }
1782:   END_HANDLE_TH_ERRORS
1783: }
1784: 
1785: static PyObject* THPAminmaxBackward0_self_getter(THPCppFunction *self, void *_unused) {
1786:   HANDLE_TH_ERRORS
1787:   const auto& prop = static_cast<AminmaxBackward0*>(self->cdata.get())->self_;
1788:   return THPVariable_Wrap(prop.unpack(self->cdata));
1789:   END_HANDLE_TH_ERRORS
1790: }
1791: 
1792: static PyObject* THPAminmaxBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
1793:   HANDLE_TH_ERRORS
1794:   const auto& prop = static_cast<AminmaxBackward0*>(self->cdata.get())->self_;
1795:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1796:   return obj.release().ptr();
1797:   END_HANDLE_TH_ERRORS
1798: }
1799: 
1800: static PyObject* THPAminmaxBackward0_max_getter(THPCppFunction *self, void *_unused) {
1801:   HANDLE_TH_ERRORS
1802:   const auto& prop = static_cast<AminmaxBackward0*>(self->cdata.get())->max_;
1803:   return THPVariable_Wrap(prop.unpack(self->cdata));
1804:   END_HANDLE_TH_ERRORS
1805: }
1806: 
1807: static PyObject* THPAminmaxBackward0_max_raw_getter(THPCppFunction *self, void *_unused) {
1808:   HANDLE_TH_ERRORS
1809:   const auto& prop = static_cast<AminmaxBackward0*>(self->cdata.get())->max_;
1810:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1811:   return obj.release().ptr();
1812:   END_HANDLE_TH_ERRORS
1813: }
1814: 
1815: static PyObject* THPAminmaxBackward0_min_getter(THPCppFunction *self, void *_unused) {
1816:   HANDLE_TH_ERRORS
1817:   const auto& prop = static_cast<AminmaxBackward0*>(self->cdata.get())->min_;
1818:   return THPVariable_Wrap(prop.unpack(self->cdata));
1819:   END_HANDLE_TH_ERRORS
1820: }
1821: 
1822: static PyObject* THPAminmaxBackward0_min_raw_getter(THPCppFunction *self, void *_unused) {
1823:   HANDLE_TH_ERRORS
1824:   const auto& prop = static_cast<AminmaxBackward0*>(self->cdata.get())->min_;
1825:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1826:   return obj.release().ptr();
1827:   END_HANDLE_TH_ERRORS
1828: }
1829: 
1830: static struct PyGetSetDef AminmaxBackward0_properties[] = {
1831:   THP_FUNCTION_DEFAULT_PROPERTIES,
1832:   {(char*)"_saved_dim", (getter)THPAminmaxBackward0_dim_getter, nullptr, nullptr, nullptr},
1833:   {(char*)"_saved_keepdim", (getter)THPAminmaxBackward0_keepdim_getter, nullptr, nullptr, nullptr},
1834:   {(char*)"_saved_self", (getter)THPAminmaxBackward0_self_getter, nullptr, nullptr, nullptr},
1835:   {(char*)"_raw_saved_self", (getter)THPAminmaxBackward0_self_raw_getter, nullptr, nullptr, nullptr},
1836:   {(char*)"_saved_max", (getter)THPAminmaxBackward0_max_getter, nullptr, nullptr, nullptr},
1837:   {(char*)"_raw_saved_max", (getter)THPAminmaxBackward0_max_raw_getter, nullptr, nullptr, nullptr},
1838:   {(char*)"_saved_min", (getter)THPAminmaxBackward0_min_getter, nullptr, nullptr, nullptr},
1839:   {(char*)"_raw_saved_min", (getter)THPAminmaxBackward0_min_raw_getter, nullptr, nullptr, nullptr},
1840:   {nullptr} /* sentinel */
```

- EN: The main execution path in this span is carried by `THPAminmaxBackward0_dim_getter`, `PyLong_FromUnsignedLong`, `THPAminmaxBackward0_keepdim_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPAminmaxBackward0_dim_getter`, `PyLong_FromUnsignedLong`, `THPAminmaxBackward0_keepdim_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1841-1920

```cpp
1841: };
1842: 
1843: static PyObject* THPModeBackward0_dim_getter(THPCppFunction *self, void *_unused) {
1844:   HANDLE_TH_ERRORS
1845:   auto prop = static_cast<ModeBackward0*>(self->cdata.get())->dim;
1846:   return PyLong_FromUnsignedLong((int64_t) prop);
1847:   END_HANDLE_TH_ERRORS
1848: }
1849: 
1850: static PyObject* THPModeBackward0_keepdim_getter(THPCppFunction *self, void *_unused) {
1851:   HANDLE_TH_ERRORS
1852:   auto prop = static_cast<ModeBackward0*>(self->cdata.get())->keepdim;
1853:   if (prop) {
1854:     Py_RETURN_TRUE;
1855:   } else {
1856:     Py_RETURN_FALSE;
1857:   }
1858:   END_HANDLE_TH_ERRORS
1859: }
1860: 
1861: static PyObject* THPModeBackward0_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
1862:   HANDLE_TH_ERRORS
1863:   auto prop = static_cast<ModeBackward0*>(self->cdata.get())->self_sym_sizes;
1864:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
1865:   for (auto i : c10::irange(prop.size())) {
1866:       auto si = prop[i];
1867:       if (auto m = si.maybe_as_int()) {
1868:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
1869:       } else {
1870:         auto py_symint = py::cast(si).release().ptr();
1871:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
1872:       }
1873:   }
1874:   return tup;
1875:   END_HANDLE_TH_ERRORS
1876: }
1877: 
1878: static PyObject* THPModeBackward0_indices_getter(THPCppFunction *self, void *_unused) {
1879:   HANDLE_TH_ERRORS
1880:   const auto& prop = static_cast<ModeBackward0*>(self->cdata.get())->indices_;
1881:   return THPVariable_Wrap(prop.unpack(self->cdata));
1882:   END_HANDLE_TH_ERRORS
1883: }
1884: 
1885: static PyObject* THPModeBackward0_indices_raw_getter(THPCppFunction *self, void *_unused) {
1886:   HANDLE_TH_ERRORS
1887:   const auto& prop = static_cast<ModeBackward0*>(self->cdata.get())->indices_;
1888:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1889:   return obj.release().ptr();
1890:   END_HANDLE_TH_ERRORS
1891: }
1892: 
1893: static struct PyGetSetDef ModeBackward0_properties[] = {
1894:   THP_FUNCTION_DEFAULT_PROPERTIES,
1895:   {(char*)"_saved_dim", (getter)THPModeBackward0_dim_getter, nullptr, nullptr, nullptr},
1896:   {(char*)"_saved_keepdim", (getter)THPModeBackward0_keepdim_getter, nullptr, nullptr, nullptr},
1897:   {(char*)"_saved_self_sym_sizes", (getter)THPModeBackward0_self_sym_sizes_getter, nullptr, nullptr, nullptr},
1898:   {(char*)"_saved_indices", (getter)THPModeBackward0_indices_getter, nullptr, nullptr, nullptr},
1899:   {(char*)"_raw_saved_indices", (getter)THPModeBackward0_indices_raw_getter, nullptr, nullptr, nullptr},
1900:   {nullptr} /* sentinel */
1901: };
1902: 
1903: static PyObject* THPMulBackward0_other_getter(THPCppFunction *self, void *_unused) {
1904:   HANDLE_TH_ERRORS
1905:   const auto& prop = static_cast<MulBackward0*>(self->cdata.get())->other_;
1906:   return THPVariable_Wrap(prop.unpack(self->cdata));
1907:   END_HANDLE_TH_ERRORS
1908: }
1909: 
1910: static PyObject* THPMulBackward0_other_raw_getter(THPCppFunction *self, void *_unused) {
1911:   HANDLE_TH_ERRORS
1912:   const auto& prop = static_cast<MulBackward0*>(self->cdata.get())->other_;
1913:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1914:   return obj.release().ptr();
1915:   END_HANDLE_TH_ERRORS
1916: }
1917: 
1918: static PyObject* THPMulBackward0_self_getter(THPCppFunction *self, void *_unused) {
1919:   HANDLE_TH_ERRORS
1920:   const auto& prop = static_cast<MulBackward0*>(self->cdata.get())->self_;
```

- EN: The main execution path in this span is carried by `THPModeBackward0_dim_getter`, `PyLong_FromUnsignedLong`, `THPModeBackward0_keepdim_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPModeBackward0_dim_getter`, `PyLong_FromUnsignedLong`, `THPModeBackward0_keepdim_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1921-2000

```cpp
1921:   return THPVariable_Wrap(prop.unpack(self->cdata));
1922:   END_HANDLE_TH_ERRORS
1923: }
1924: 
1925: static PyObject* THPMulBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
1926:   HANDLE_TH_ERRORS
1927:   const auto& prop = static_cast<MulBackward0*>(self->cdata.get())->self_;
1928:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1929:   return obj.release().ptr();
1930:   END_HANDLE_TH_ERRORS
1931: }
1932: 
1933: static struct PyGetSetDef MulBackward0_properties[] = {
1934:   THP_FUNCTION_DEFAULT_PROPERTIES,
1935:   {(char*)"_saved_other", (getter)THPMulBackward0_other_getter, nullptr, nullptr, nullptr},
1936:   {(char*)"_raw_saved_other", (getter)THPMulBackward0_other_raw_getter, nullptr, nullptr, nullptr},
1937:   {(char*)"_saved_self", (getter)THPMulBackward0_self_getter, nullptr, nullptr, nullptr},
1938:   {(char*)"_raw_saved_self", (getter)THPMulBackward0_self_raw_getter, nullptr, nullptr, nullptr},
1939:   {nullptr} /* sentinel */
1940: };
1941: 
1942: static PyObject* THPMulBackward1_other_getter(THPCppFunction *self, void *_unused) {
1943:   HANDLE_TH_ERRORS
1944:   auto prop = static_cast<MulBackward1*>(self->cdata.get())->other;
1945:   if (prop.isComplex()) {
1946:     auto cprop = prop.to<c10::complex<double>>();
1947:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
1948:   } else if (prop.isFloatingPoint()) {
1949:     return PyFloat_FromDouble(prop.to<double>());
1950:   } else if (prop.isIntegral(/*includeBool=*/false)) {
1951:     return PyLong_FromLong(prop.to<int64_t>());
1952:   } else if (prop.isBoolean()) {
1953:     if (prop.to<bool>()) {
1954:       Py_RETURN_TRUE;
1955:     } else {
1956:       Py_RETURN_FALSE;
1957:     }
1958:   } else {
1959:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
1960:     return nullptr;
1961:   }
1962:   END_HANDLE_TH_ERRORS
1963: }
1964: 
1965: static struct PyGetSetDef MulBackward1_properties[] = {
1966:   THP_FUNCTION_DEFAULT_PROPERTIES,
1967:   {(char*)"_saved_other", (getter)THPMulBackward1_other_getter, nullptr, nullptr, nullptr},
1968:   {nullptr} /* sentinel */
1969: };
1970: 
1971: static PyObject* THPMvlgammaBackward0_p_getter(THPCppFunction *self, void *_unused) {
1972:   HANDLE_TH_ERRORS
1973:   auto prop = static_cast<MvlgammaBackward0*>(self->cdata.get())->p;
1974:   return PyLong_FromUnsignedLong((int64_t) prop);
1975:   END_HANDLE_TH_ERRORS
1976: }
1977: 
1978: static PyObject* THPMvlgammaBackward0_self_getter(THPCppFunction *self, void *_unused) {
1979:   HANDLE_TH_ERRORS
1980:   const auto& prop = static_cast<MvlgammaBackward0*>(self->cdata.get())->self_;
1981:   return THPVariable_Wrap(prop.unpack(self->cdata));
1982:   END_HANDLE_TH_ERRORS
1983: }
1984: 
1985: static PyObject* THPMvlgammaBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
1986:   HANDLE_TH_ERRORS
1987:   const auto& prop = static_cast<MvlgammaBackward0*>(self->cdata.get())->self_;
1988:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1989:   return obj.release().ptr();
1990:   END_HANDLE_TH_ERRORS
1991: }
1992: 
1993: static struct PyGetSetDef MvlgammaBackward0_properties[] = {
1994:   THP_FUNCTION_DEFAULT_PROPERTIES,
1995:   {(char*)"_saved_p", (getter)THPMvlgammaBackward0_p_getter, nullptr, nullptr, nullptr},
1996:   {(char*)"_saved_self", (getter)THPMvlgammaBackward0_self_getter, nullptr, nullptr, nullptr},
1997:   {(char*)"_raw_saved_self", (getter)THPMvlgammaBackward0_self_raw_getter, nullptr, nullptr, nullptr},
1998:   {nullptr} /* sentinel */
1999: };
2000: 
```

- EN: The main execution path in this span is carried by `THPVariable_Wrap`, `THPMulBackward0_self_raw_getter`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_Wrap`, `THPMulBackward0_self_raw_getter`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2001-2080

```cpp
2001: 
2002: 
2003: static struct PyGetSetDef NegBackward0_properties[] = {
2004:   THP_FUNCTION_DEFAULT_PROPERTIES,
2005: 
2006:   {nullptr} /* sentinel */
2007: };
2008: 
2009: 
2010: 
2011: static struct PyGetSetDef NormalBackward0_properties[] = {
2012:   THP_FUNCTION_DEFAULT_PROPERTIES,
2013: 
2014:   {nullptr} /* sentinel */
2015: };
2016: 
2017: static PyObject* THPPowBackward0_exponent_getter(THPCppFunction *self, void *_unused) {
2018:   HANDLE_TH_ERRORS
2019:   auto prop = static_cast<PowBackward0*>(self->cdata.get())->exponent;
2020:   if (prop.isComplex()) {
2021:     auto cprop = prop.to<c10::complex<double>>();
2022:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
2023:   } else if (prop.isFloatingPoint()) {
2024:     return PyFloat_FromDouble(prop.to<double>());
2025:   } else if (prop.isIntegral(/*includeBool=*/false)) {
2026:     return PyLong_FromLong(prop.to<int64_t>());
2027:   } else if (prop.isBoolean()) {
2028:     if (prop.to<bool>()) {
2029:       Py_RETURN_TRUE;
2030:     } else {
2031:       Py_RETURN_FALSE;
2032:     }
2033:   } else {
2034:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
2035:     return nullptr;
2036:   }
2037:   END_HANDLE_TH_ERRORS
2038: }
2039: 
2040: static PyObject* THPPowBackward0_self_getter(THPCppFunction *self, void *_unused) {
2041:   HANDLE_TH_ERRORS
2042:   const auto& prop = static_cast<PowBackward0*>(self->cdata.get())->self_;
2043:   return THPVariable_Wrap(prop.unpack(self->cdata));
2044:   END_HANDLE_TH_ERRORS
2045: }
2046: 
2047: static PyObject* THPPowBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
2048:   HANDLE_TH_ERRORS
2049:   const auto& prop = static_cast<PowBackward0*>(self->cdata.get())->self_;
2050:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2051:   return obj.release().ptr();
2052:   END_HANDLE_TH_ERRORS
2053: }
2054: 
2055: static struct PyGetSetDef PowBackward0_properties[] = {
2056:   THP_FUNCTION_DEFAULT_PROPERTIES,
2057:   {(char*)"_saved_exponent", (getter)THPPowBackward0_exponent_getter, nullptr, nullptr, nullptr},
2058:   {(char*)"_saved_self", (getter)THPPowBackward0_self_getter, nullptr, nullptr, nullptr},
2059:   {(char*)"_raw_saved_self", (getter)THPPowBackward0_self_raw_getter, nullptr, nullptr, nullptr},
2060:   {nullptr} /* sentinel */
2061: };
2062: 
2063: static PyObject* THPPowBackward1_exponent_getter(THPCppFunction *self, void *_unused) {
2064:   HANDLE_TH_ERRORS
2065:   const auto& prop = static_cast<PowBackward1*>(self->cdata.get())->exponent_;
2066:   return THPVariable_Wrap(prop.unpack(self->cdata));
2067:   END_HANDLE_TH_ERRORS
2068: }
2069: 
2070: static PyObject* THPPowBackward1_exponent_raw_getter(THPCppFunction *self, void *_unused) {
2071:   HANDLE_TH_ERRORS
2072:   const auto& prop = static_cast<PowBackward1*>(self->cdata.get())->exponent_;
2073:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2074:   return obj.release().ptr();
2075:   END_HANDLE_TH_ERRORS
2076: }
2077: 
2078: static PyObject* THPPowBackward1_self_getter(THPCppFunction *self, void *_unused) {
2079:   HANDLE_TH_ERRORS
2080:   const auto& prop = static_cast<PowBackward1*>(self->cdata.get())->self_;
```

- EN: The main execution path in this span is carried by `THPPowBackward0_exponent_getter`, `PyComplex_FromDoubles`, `PyFloat_FromDouble`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPPowBackward0_exponent_getter`, `PyComplex_FromDoubles`, `PyFloat_FromDouble` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2081-2160

```cpp
2081:   return THPVariable_Wrap(prop.unpack(self->cdata));
2082:   END_HANDLE_TH_ERRORS
2083: }
2084: 
2085: static PyObject* THPPowBackward1_self_raw_getter(THPCppFunction *self, void *_unused) {
2086:   HANDLE_TH_ERRORS
2087:   const auto& prop = static_cast<PowBackward1*>(self->cdata.get())->self_;
2088:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2089:   return obj.release().ptr();
2090:   END_HANDLE_TH_ERRORS
2091: }
2092: 
2093: static PyObject* THPPowBackward1_result_getter(THPCppFunction *self, void *_unused) {
2094:   HANDLE_TH_ERRORS
2095:   const auto& prop = static_cast<PowBackward1*>(self->cdata.get())->result_;
2096:   return THPVariable_Wrap(prop.unpack(self->cdata));
2097:   END_HANDLE_TH_ERRORS
2098: }
2099: 
2100: static PyObject* THPPowBackward1_result_raw_getter(THPCppFunction *self, void *_unused) {
2101:   HANDLE_TH_ERRORS
2102:   const auto& prop = static_cast<PowBackward1*>(self->cdata.get())->result_;
2103:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2104:   return obj.release().ptr();
2105:   END_HANDLE_TH_ERRORS
2106: }
2107: 
2108: static struct PyGetSetDef PowBackward1_properties[] = {
2109:   THP_FUNCTION_DEFAULT_PROPERTIES,
2110:   {(char*)"_saved_exponent", (getter)THPPowBackward1_exponent_getter, nullptr, nullptr, nullptr},
2111:   {(char*)"_raw_saved_exponent", (getter)THPPowBackward1_exponent_raw_getter, nullptr, nullptr, nullptr},
2112:   {(char*)"_saved_self", (getter)THPPowBackward1_self_getter, nullptr, nullptr, nullptr},
2113:   {(char*)"_raw_saved_self", (getter)THPPowBackward1_self_raw_getter, nullptr, nullptr, nullptr},
2114:   {(char*)"_saved_result", (getter)THPPowBackward1_result_getter, nullptr, nullptr, nullptr},
2115:   {(char*)"_raw_saved_result", (getter)THPPowBackward1_result_raw_getter, nullptr, nullptr, nullptr},
2116:   {nullptr} /* sentinel */
2117: };
2118: 
2119: static PyObject* THPPowBackward2_exponent_getter(THPCppFunction *self, void *_unused) {
2120:   HANDLE_TH_ERRORS
2121:   const auto& prop = static_cast<PowBackward2*>(self->cdata.get())->exponent_;
2122:   return THPVariable_Wrap(prop.unpack(self->cdata));
2123:   END_HANDLE_TH_ERRORS
2124: }
2125: 
2126: static PyObject* THPPowBackward2_exponent_raw_getter(THPCppFunction *self, void *_unused) {
2127:   HANDLE_TH_ERRORS
2128:   const auto& prop = static_cast<PowBackward2*>(self->cdata.get())->exponent_;
2129:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2130:   return obj.release().ptr();
2131:   END_HANDLE_TH_ERRORS
2132: }
2133: 
2134: static PyObject* THPPowBackward2_self_getter(THPCppFunction *self, void *_unused) {
2135:   HANDLE_TH_ERRORS
2136:   auto prop = static_cast<PowBackward2*>(self->cdata.get())->self;
2137:   if (prop.isComplex()) {
2138:     auto cprop = prop.to<c10::complex<double>>();
2139:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
2140:   } else if (prop.isFloatingPoint()) {
2141:     return PyFloat_FromDouble(prop.to<double>());
2142:   } else if (prop.isIntegral(/*includeBool=*/false)) {
2143:     return PyLong_FromLong(prop.to<int64_t>());
2144:   } else if (prop.isBoolean()) {
2145:     if (prop.to<bool>()) {
2146:       Py_RETURN_TRUE;
2147:     } else {
2148:       Py_RETURN_FALSE;
2149:     }
2150:   } else {
2151:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
2152:     return nullptr;
2153:   }
2154:   END_HANDLE_TH_ERRORS
2155: }
2156: 
2157: static PyObject* THPPowBackward2_result_getter(THPCppFunction *self, void *_unused) {
2158:   HANDLE_TH_ERRORS
2159:   const auto& prop = static_cast<PowBackward2*>(self->cdata.get())->result_;
2160:   return THPVariable_Wrap(prop.unpack(self->cdata));
```

- EN: The main execution path in this span is carried by `THPVariable_Wrap`, `THPPowBackward1_self_raw_getter`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_Wrap`, `THPPowBackward1_self_raw_getter`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2161-2240

```cpp
2161:   END_HANDLE_TH_ERRORS
2162: }
2163: 
2164: static PyObject* THPPowBackward2_result_raw_getter(THPCppFunction *self, void *_unused) {
2165:   HANDLE_TH_ERRORS
2166:   const auto& prop = static_cast<PowBackward2*>(self->cdata.get())->result_;
2167:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2168:   return obj.release().ptr();
2169:   END_HANDLE_TH_ERRORS
2170: }
2171: 
2172: static struct PyGetSetDef PowBackward2_properties[] = {
2173:   THP_FUNCTION_DEFAULT_PROPERTIES,
2174:   {(char*)"_saved_exponent", (getter)THPPowBackward2_exponent_getter, nullptr, nullptr, nullptr},
2175:   {(char*)"_raw_saved_exponent", (getter)THPPowBackward2_exponent_raw_getter, nullptr, nullptr, nullptr},
2176:   {(char*)"_saved_self", (getter)THPPowBackward2_self_getter, nullptr, nullptr, nullptr},
2177:   {(char*)"_saved_result", (getter)THPPowBackward2_result_getter, nullptr, nullptr, nullptr},
2178:   {(char*)"_raw_saved_result", (getter)THPPowBackward2_result_raw_getter, nullptr, nullptr, nullptr},
2179:   {nullptr} /* sentinel */
2180: };
2181: 
2182: static PyObject* THPReciprocalBackward0_result_getter(THPCppFunction *self, void *_unused) {
2183:   HANDLE_TH_ERRORS
2184:   const auto& prop = static_cast<ReciprocalBackward0*>(self->cdata.get())->result_;
2185:   return THPVariable_Wrap(prop.unpack(self->cdata));
2186:   END_HANDLE_TH_ERRORS
2187: }
2188: 
2189: static PyObject* THPReciprocalBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
2190:   HANDLE_TH_ERRORS
2191:   const auto& prop = static_cast<ReciprocalBackward0*>(self->cdata.get())->result_;
2192:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2193:   return obj.release().ptr();
2194:   END_HANDLE_TH_ERRORS
2195: }
2196: 
2197: static struct PyGetSetDef ReciprocalBackward0_properties[] = {
2198:   THP_FUNCTION_DEFAULT_PROPERTIES,
2199:   {(char*)"_saved_result", (getter)THPReciprocalBackward0_result_getter, nullptr, nullptr, nullptr},
2200:   {(char*)"_raw_saved_result", (getter)THPReciprocalBackward0_result_raw_getter, nullptr, nullptr, nullptr},
2201:   {nullptr} /* sentinel */
2202: };
2203: 
2204: static PyObject* THPSpecialEntrBackward0_self_getter(THPCppFunction *self, void *_unused) {
2205:   HANDLE_TH_ERRORS
2206:   const auto& prop = static_cast<SpecialEntrBackward0*>(self->cdata.get())->self_;
2207:   return THPVariable_Wrap(prop.unpack(self->cdata));
2208:   END_HANDLE_TH_ERRORS
2209: }
2210: 
2211: static PyObject* THPSpecialEntrBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
2212:   HANDLE_TH_ERRORS
2213:   const auto& prop = static_cast<SpecialEntrBackward0*>(self->cdata.get())->self_;
2214:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2215:   return obj.release().ptr();
2216:   END_HANDLE_TH_ERRORS
2217: }
2218: 
2219: static struct PyGetSetDef SpecialEntrBackward0_properties[] = {
2220:   THP_FUNCTION_DEFAULT_PROPERTIES,
2221:   {(char*)"_saved_self", (getter)THPSpecialEntrBackward0_self_getter, nullptr, nullptr, nullptr},
2222:   {(char*)"_raw_saved_self", (getter)THPSpecialEntrBackward0_self_raw_getter, nullptr, nullptr, nullptr},
2223:   {nullptr} /* sentinel */
2224: };
2225: 
2226: static PyObject* THPSpecialNdtriBackward0_result_getter(THPCppFunction *self, void *_unused) {
2227:   HANDLE_TH_ERRORS
2228:   const auto& prop = static_cast<SpecialNdtriBackward0*>(self->cdata.get())->result_;
2229:   return THPVariable_Wrap(prop.unpack(self->cdata));
2230:   END_HANDLE_TH_ERRORS
2231: }
2232: 
2233: static PyObject* THPSpecialNdtriBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
2234:   HANDLE_TH_ERRORS
2235:   const auto& prop = static_cast<SpecialNdtriBackward0*>(self->cdata.get())->result_;
2236:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2237:   return obj.release().ptr();
2238:   END_HANDLE_TH_ERRORS
2239: }
2240: 
```

- EN: The main execution path in this span is carried by `THPPowBackward2_result_raw_getter`, `cast`, `THPReciprocalBackward0_result_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPPowBackward2_result_raw_getter`, `cast`, `THPReciprocalBackward0_result_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2241-2320

```cpp
2241: static struct PyGetSetDef SpecialNdtriBackward0_properties[] = {
2242:   THP_FUNCTION_DEFAULT_PROPERTIES,
2243:   {(char*)"_saved_result", (getter)THPSpecialNdtriBackward0_result_getter, nullptr, nullptr, nullptr},
2244:   {(char*)"_raw_saved_result", (getter)THPSpecialNdtriBackward0_result_raw_getter, nullptr, nullptr, nullptr},
2245:   {nullptr} /* sentinel */
2246: };
2247: 
2248: static PyObject* THPSgnBackward0_self_getter(THPCppFunction *self, void *_unused) {
2249:   HANDLE_TH_ERRORS
2250:   const auto& prop = static_cast<SgnBackward0*>(self->cdata.get())->self_;
2251:   return THPVariable_Wrap(prop.unpack(self->cdata));
2252:   END_HANDLE_TH_ERRORS
2253: }
2254: 
2255: static PyObject* THPSgnBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
2256:   HANDLE_TH_ERRORS
2257:   const auto& prop = static_cast<SgnBackward0*>(self->cdata.get())->self_;
2258:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2259:   return obj.release().ptr();
2260:   END_HANDLE_TH_ERRORS
2261: }
2262: 
2263: static PyObject* THPSgnBackward0_result_getter(THPCppFunction *self, void *_unused) {
2264:   HANDLE_TH_ERRORS
2265:   const auto& prop = static_cast<SgnBackward0*>(self->cdata.get())->result_;
2266:   return THPVariable_Wrap(prop.unpack(self->cdata));
2267:   END_HANDLE_TH_ERRORS
2268: }
2269: 
2270: static PyObject* THPSgnBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
2271:   HANDLE_TH_ERRORS
2272:   const auto& prop = static_cast<SgnBackward0*>(self->cdata.get())->result_;
2273:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2274:   return obj.release().ptr();
2275:   END_HANDLE_TH_ERRORS
2276: }
2277: 
2278: static struct PyGetSetDef SgnBackward0_properties[] = {
2279:   THP_FUNCTION_DEFAULT_PROPERTIES,
2280:   {(char*)"_saved_self", (getter)THPSgnBackward0_self_getter, nullptr, nullptr, nullptr},
2281:   {(char*)"_raw_saved_self", (getter)THPSgnBackward0_self_raw_getter, nullptr, nullptr, nullptr},
2282:   {(char*)"_saved_result", (getter)THPSgnBackward0_result_getter, nullptr, nullptr, nullptr},
2283:   {(char*)"_raw_saved_result", (getter)THPSgnBackward0_result_raw_getter, nullptr, nullptr, nullptr},
2284:   {nullptr} /* sentinel */
2285: };
2286: 
2287: static PyObject* THPSinBackward0_self_getter(THPCppFunction *self, void *_unused) {
2288:   HANDLE_TH_ERRORS
2289:   const auto& prop = static_cast<SinBackward0*>(self->cdata.get())->self_;
2290:   return THPVariable_Wrap(prop.unpack(self->cdata));
2291:   END_HANDLE_TH_ERRORS
2292: }
2293: 
2294: static PyObject* THPSinBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
2295:   HANDLE_TH_ERRORS
2296:   const auto& prop = static_cast<SinBackward0*>(self->cdata.get())->self_;
2297:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2298:   return obj.release().ptr();
2299:   END_HANDLE_TH_ERRORS
2300: }
2301: 
2302: static struct PyGetSetDef SinBackward0_properties[] = {
2303:   THP_FUNCTION_DEFAULT_PROPERTIES,
2304:   {(char*)"_saved_self", (getter)THPSinBackward0_self_getter, nullptr, nullptr, nullptr},
2305:   {(char*)"_raw_saved_self", (getter)THPSinBackward0_self_raw_getter, nullptr, nullptr, nullptr},
2306:   {nullptr} /* sentinel */
2307: };
2308: 
2309: static PyObject* THPDiagonalScatterBackward0_dim1_getter(THPCppFunction *self, void *_unused) {
2310:   HANDLE_TH_ERRORS
2311:   auto prop = static_cast<DiagonalScatterBackward0*>(self->cdata.get())->dim1;
2312:   return PyLong_FromUnsignedLong((int64_t) prop);
2313:   END_HANDLE_TH_ERRORS
2314: }
2315: 
2316: static PyObject* THPDiagonalScatterBackward0_dim2_getter(THPCppFunction *self, void *_unused) {
2317:   HANDLE_TH_ERRORS
2318:   auto prop = static_cast<DiagonalScatterBackward0*>(self->cdata.get())->dim2;
2319:   return PyLong_FromUnsignedLong((int64_t) prop);
2320:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPSgnBackward0_self_getter`, `THPVariable_Wrap`, `THPSgnBackward0_self_raw_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPSgnBackward0_self_getter`, `THPVariable_Wrap`, `THPSgnBackward0_self_raw_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2321-2400

```cpp
2321: }
2322: 
2323: static PyObject* THPDiagonalScatterBackward0_offset_getter(THPCppFunction *self, void *_unused) {
2324:   HANDLE_TH_ERRORS
2325:   auto prop = static_cast<DiagonalScatterBackward0*>(self->cdata.get())->offset;
2326:   return PyLong_FromUnsignedLong((int64_t) prop);
2327:   END_HANDLE_TH_ERRORS
2328: }
2329: 
2330: static struct PyGetSetDef DiagonalScatterBackward0_properties[] = {
2331:   THP_FUNCTION_DEFAULT_PROPERTIES,
2332:   {(char*)"_saved_dim1", (getter)THPDiagonalScatterBackward0_dim1_getter, nullptr, nullptr, nullptr},
2333:   {(char*)"_saved_dim2", (getter)THPDiagonalScatterBackward0_dim2_getter, nullptr, nullptr, nullptr},
2334:   {(char*)"_saved_offset", (getter)THPDiagonalScatterBackward0_offset_getter, nullptr, nullptr, nullptr},
2335:   {nullptr} /* sentinel */
2336: };
2337: 
2338: static PyObject* THPAsStridedScatterBackward0_size_getter(THPCppFunction *self, void *_unused) {
2339:   HANDLE_TH_ERRORS
2340:   auto prop = static_cast<AsStridedScatterBackward0*>(self->cdata.get())->size;
2341:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
2342:   for (auto i : c10::irange(prop.size())) {
2343:       auto si = prop[i];
2344:       if (auto m = si.maybe_as_int()) {
2345:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
2346:       } else {
2347:         auto py_symint = py::cast(si).release().ptr();
2348:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
2349:       }
2350:   }
2351:   return tup;
2352:   END_HANDLE_TH_ERRORS
2353: }
2354: 
2355: static PyObject* THPAsStridedScatterBackward0_storage_offset_getter(THPCppFunction *self, void *_unused) {
2356:   HANDLE_TH_ERRORS
2357:   auto opt_prop = static_cast<AsStridedScatterBackward0*>(self->cdata.get())->storage_offset;
2358:   if (!opt_prop.has_value()) {
2359:     Py_RETURN_NONE;
2360:   }
2361:   auto prop = opt_prop.value();
2362:   if (auto m = prop.maybe_as_int()) {
2363:     return PyLong_FromUnsignedLong(*m);
2364:   } else {
2365:     return py::cast(prop).release().ptr();
2366:   }
2367:   END_HANDLE_TH_ERRORS
2368: }
2369: 
2370: static PyObject* THPAsStridedScatterBackward0_stride_getter(THPCppFunction *self, void *_unused) {
2371:   HANDLE_TH_ERRORS
2372:   auto prop = static_cast<AsStridedScatterBackward0*>(self->cdata.get())->stride;
2373:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
2374:   for (auto i : c10::irange(prop.size())) {
2375:       auto si = prop[i];
2376:       if (auto m = si.maybe_as_int()) {
2377:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
2378:       } else {
2379:         auto py_symint = py::cast(si).release().ptr();
2380:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
2381:       }
2382:   }
2383:   return tup;
2384:   END_HANDLE_TH_ERRORS
2385: }
2386: 
2387: static struct PyGetSetDef AsStridedScatterBackward0_properties[] = {
2388:   THP_FUNCTION_DEFAULT_PROPERTIES,
2389:   {(char*)"_saved_size", (getter)THPAsStridedScatterBackward0_size_getter, nullptr, nullptr, nullptr},
2390:   {(char*)"_saved_storage_offset", (getter)THPAsStridedScatterBackward0_storage_offset_getter, nullptr, nullptr, nullptr},
2391:   {(char*)"_saved_stride", (getter)THPAsStridedScatterBackward0_stride_getter, nullptr, nullptr, nullptr},
2392:   {nullptr} /* sentinel */
2393: };
2394: 
2395: static PyObject* THPSplitBackward0_dim_getter(THPCppFunction *self, void *_unused) {
2396:   HANDLE_TH_ERRORS
2397:   auto prop = static_cast<SplitBackward0*>(self->cdata.get())->dim;
2398:   return PyLong_FromUnsignedLong((int64_t) prop);
2399:   END_HANDLE_TH_ERRORS
2400: }
```

- EN: The main execution path in this span is carried by `THPDiagonalScatterBackward0_offset_getter`, `PyLong_FromUnsignedLong`, `THPAsStridedScatterBackward0_size_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPDiagonalScatterBackward0_offset_getter`, `PyLong_FromUnsignedLong`, `THPAsStridedScatterBackward0_size_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2401-2480

```cpp
2401: 
2402: static PyObject* THPSplitBackward0_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
2403:   HANDLE_TH_ERRORS
2404:   auto prop = static_cast<SplitBackward0*>(self->cdata.get())->self_sym_sizes;
2405:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
2406:   for (auto i : c10::irange(prop.size())) {
2407:       auto si = prop[i];
2408:       if (auto m = si.maybe_as_int()) {
2409:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
2410:       } else {
2411:         auto py_symint = py::cast(si).release().ptr();
2412:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
2413:       }
2414:   }
2415:   return tup;
2416:   END_HANDLE_TH_ERRORS
2417: }
2418: 
2419: static PyObject* THPSplitBackward0_split_size_getter(THPCppFunction *self, void *_unused) {
2420:   HANDLE_TH_ERRORS
2421:   auto prop = static_cast<SplitBackward0*>(self->cdata.get())->split_size;
2422:   if (auto m = prop.maybe_as_int()) {
2423:     return PyLong_FromUnsignedLong(*m);
2424:   } else {
2425:     return py::cast(prop).release().ptr();
2426:   }
2427:   END_HANDLE_TH_ERRORS
2428: }
2429: 
2430: static struct PyGetSetDef SplitBackward0_properties[] = {
2431:   THP_FUNCTION_DEFAULT_PROPERTIES,
2432:   {(char*)"_saved_dim", (getter)THPSplitBackward0_dim_getter, nullptr, nullptr, nullptr},
2433:   {(char*)"_saved_self_sym_sizes", (getter)THPSplitBackward0_self_sym_sizes_getter, nullptr, nullptr, nullptr},
2434:   {(char*)"_saved_split_size", (getter)THPSplitBackward0_split_size_getter, nullptr, nullptr, nullptr},
2435:   {nullptr} /* sentinel */
2436: };
2437: 
2438: static PyObject* THPSqueezeBackward0_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
2439:   HANDLE_TH_ERRORS
2440:   auto prop = static_cast<SqueezeBackward0*>(self->cdata.get())->self_sym_sizes;
2441:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
2442:   for (auto i : c10::irange(prop.size())) {
2443:       auto si = prop[i];
2444:       if (auto m = si.maybe_as_int()) {
2445:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
2446:       } else {
2447:         auto py_symint = py::cast(si).release().ptr();
2448:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
2449:       }
2450:   }
2451:   return tup;
2452:   END_HANDLE_TH_ERRORS
2453: }
2454: 
2455: static struct PyGetSetDef SqueezeBackward0_properties[] = {
2456:   THP_FUNCTION_DEFAULT_PROPERTIES,
2457:   {(char*)"_saved_self_sym_sizes", (getter)THPSqueezeBackward0_self_sym_sizes_getter, nullptr, nullptr, nullptr},
2458:   {nullptr} /* sentinel */
2459: };
2460: 
2461: static PyObject* THPSqueezeBackward1_dim_getter(THPCppFunction *self, void *_unused) {
2462:   HANDLE_TH_ERRORS
2463:   auto prop = static_cast<SqueezeBackward1*>(self->cdata.get())->dim;
2464:   return PyLong_FromUnsignedLong((int64_t) prop);
2465:   END_HANDLE_TH_ERRORS
2466: }
2467: 
2468: static PyObject* THPSqueezeBackward1_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
2469:   HANDLE_TH_ERRORS
2470:   auto prop = static_cast<SqueezeBackward1*>(self->cdata.get())->self_sym_sizes;
2471:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
2472:   for (auto i : c10::irange(prop.size())) {
2473:       auto si = prop[i];
2474:       if (auto m = si.maybe_as_int()) {
2475:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
2476:       } else {
2477:         auto py_symint = py::cast(si).release().ptr();
2478:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
2479:       }
2480:   }
```

- EN: The main execution path in this span is carried by `THPSplitBackward0_self_sym_sizes_getter`, `PyTuple_New`, `PyTuple_SetItem`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPSplitBackward0_self_sym_sizes_getter`, `PyTuple_New`, `PyTuple_SetItem` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2481-2560

```cpp
2481:   return tup;
2482:   END_HANDLE_TH_ERRORS
2483: }
2484: 
2485: static struct PyGetSetDef SqueezeBackward1_properties[] = {
2486:   THP_FUNCTION_DEFAULT_PROPERTIES,
2487:   {(char*)"_saved_dim", (getter)THPSqueezeBackward1_dim_getter, nullptr, nullptr, nullptr},
2488:   {(char*)"_saved_self_sym_sizes", (getter)THPSqueezeBackward1_self_sym_sizes_getter, nullptr, nullptr, nullptr},
2489:   {nullptr} /* sentinel */
2490: };
2491: 
2492: static PyObject* THPSqueezeBackwardAutogradNestedTensor0_dim_getter(THPCppFunction *self, void *_unused) {
2493:   HANDLE_TH_ERRORS
2494:   auto prop = static_cast<SqueezeBackwardAutogradNestedTensor0*>(self->cdata.get())->dim;
2495:   return PyLong_FromUnsignedLong((int64_t) prop);
2496:   END_HANDLE_TH_ERRORS
2497: }
2498: 
2499: static struct PyGetSetDef SqueezeBackwardAutogradNestedTensor0_properties[] = {
2500:   THP_FUNCTION_DEFAULT_PROPERTIES,
2501:   {(char*)"_saved_dim", (getter)THPSqueezeBackwardAutogradNestedTensor0_dim_getter, nullptr, nullptr, nullptr},
2502:   {nullptr} /* sentinel */
2503: };
2504: 
2505: static PyObject* THPSqueezeBackward2_dim_getter(THPCppFunction *self, void *_unused) {
2506:   HANDLE_TH_ERRORS
2507:   auto prop = static_cast<SqueezeBackward2*>(self->cdata.get())->dim;
2508:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
2509:   for (auto i : c10::irange(prop.size())) {
2510:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
2511:   }
2512:   return tup;
2513:   END_HANDLE_TH_ERRORS
2514: }
2515: 
2516: static PyObject* THPSqueezeBackward2_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
2517:   HANDLE_TH_ERRORS
2518:   auto prop = static_cast<SqueezeBackward2*>(self->cdata.get())->self_sym_sizes;
2519:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
2520:   for (auto i : c10::irange(prop.size())) {
2521:       auto si = prop[i];
2522:       if (auto m = si.maybe_as_int()) {
2523:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
2524:       } else {
2525:         auto py_symint = py::cast(si).release().ptr();
2526:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
2527:       }
2528:   }
2529:   return tup;
2530:   END_HANDLE_TH_ERRORS
2531: }
2532: 
2533: static struct PyGetSetDef SqueezeBackward2_properties[] = {
2534:   THP_FUNCTION_DEFAULT_PROPERTIES,
2535:   {(char*)"_saved_dim", (getter)THPSqueezeBackward2_dim_getter, nullptr, nullptr, nullptr},
2536:   {(char*)"_saved_self_sym_sizes", (getter)THPSqueezeBackward2_self_sym_sizes_getter, nullptr, nullptr, nullptr},
2537:   {nullptr} /* sentinel */
2538: };
2539: 
2540: static PyObject* THPSqueezeBackwardAutogradNestedTensor1_dim_getter(THPCppFunction *self, void *_unused) {
2541:   HANDLE_TH_ERRORS
2542:   auto prop = static_cast<SqueezeBackwardAutogradNestedTensor1*>(self->cdata.get())->dim;
2543:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
2544:   for (auto i : c10::irange(prop.size())) {
2545:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
2546:   }
2547:   return tup;
2548:   END_HANDLE_TH_ERRORS
2549: }
2550: 
2551: static PyObject* THPSqueezeBackwardAutogradNestedTensor1_self_dim_getter(THPCppFunction *self, void *_unused) {
2552:   HANDLE_TH_ERRORS
2553:   auto prop = static_cast<SqueezeBackwardAutogradNestedTensor1*>(self->cdata.get())->self_dim;
2554:   return PyLong_FromUnsignedLong((int64_t) prop);
2555:   END_HANDLE_TH_ERRORS
2556: }
2557: 
2558: static struct PyGetSetDef SqueezeBackwardAutogradNestedTensor1_properties[] = {
2559:   THP_FUNCTION_DEFAULT_PROPERTIES,
2560:   {(char*)"_saved_dim", (getter)THPSqueezeBackwardAutogradNestedTensor1_dim_getter, nullptr, nullptr, nullptr},
```

- EN: The main execution path in this span is carried by `THPSqueezeBackwardAutogradNestedTensor0_dim_getter`, `PyLong_FromUnsignedLong`, `THPSqueezeBackward2_dim_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPSqueezeBackwardAutogradNestedTensor0_dim_getter`, `PyLong_FromUnsignedLong`, `THPSqueezeBackward2_dim_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 2561-2640

```cpp
2561:   {(char*)"_saved_self_dim", (getter)THPSqueezeBackwardAutogradNestedTensor1_self_dim_getter, nullptr, nullptr, nullptr},
2562:   {nullptr} /* sentinel */
2563: };
2564: 
2565: static PyObject* THPLinalgEighBackward0_eigenvalues_getter(THPCppFunction *self, void *_unused) {
2566:   HANDLE_TH_ERRORS
2567:   const auto& prop = static_cast<LinalgEighBackward0*>(self->cdata.get())->eigenvalues_;
2568:   return THPVariable_Wrap(prop.unpack(self->cdata));
2569:   END_HANDLE_TH_ERRORS
2570: }
2571: 
2572: static PyObject* THPLinalgEighBackward0_eigenvalues_raw_getter(THPCppFunction *self, void *_unused) {
2573:   HANDLE_TH_ERRORS
2574:   const auto& prop = static_cast<LinalgEighBackward0*>(self->cdata.get())->eigenvalues_;
2575:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2576:   return obj.release().ptr();
2577:   END_HANDLE_TH_ERRORS
2578: }
2579: 
2580: static PyObject* THPLinalgEighBackward0_eigenvectors_getter(THPCppFunction *self, void *_unused) {
2581:   HANDLE_TH_ERRORS
2582:   const auto& prop = static_cast<LinalgEighBackward0*>(self->cdata.get())->eigenvectors_;
2583:   return THPVariable_Wrap(prop.unpack(self->cdata));
2584:   END_HANDLE_TH_ERRORS
2585: }
2586: 
2587: static PyObject* THPLinalgEighBackward0_eigenvectors_raw_getter(THPCppFunction *self, void *_unused) {
2588:   HANDLE_TH_ERRORS
2589:   const auto& prop = static_cast<LinalgEighBackward0*>(self->cdata.get())->eigenvectors_;
2590:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2591:   return obj.release().ptr();
2592:   END_HANDLE_TH_ERRORS
2593: }
2594: 
2595: static struct PyGetSetDef LinalgEighBackward0_properties[] = {
2596:   THP_FUNCTION_DEFAULT_PROPERTIES,
2597:   {(char*)"_saved_eigenvalues", (getter)THPLinalgEighBackward0_eigenvalues_getter, nullptr, nullptr, nullptr},
2598:   {(char*)"_raw_saved_eigenvalues", (getter)THPLinalgEighBackward0_eigenvalues_raw_getter, nullptr, nullptr, nullptr},
2599:   {(char*)"_saved_eigenvectors", (getter)THPLinalgEighBackward0_eigenvectors_getter, nullptr, nullptr, nullptr},
2600:   {(char*)"_raw_saved_eigenvectors", (getter)THPLinalgEighBackward0_eigenvectors_raw_getter, nullptr, nullptr, nullptr},
2601:   {nullptr} /* sentinel */
2602: };
2603: 
2604: static PyObject* THPToMkldnnBackward0_self_getter(THPCppFunction *self, void *_unused) {
2605:   HANDLE_TH_ERRORS
2606:   const auto& prop = static_cast<ToMkldnnBackward0*>(self->cdata.get())->self_;
2607:   return THPVariable_Wrap(prop.unpack(self->cdata));
2608:   END_HANDLE_TH_ERRORS
2609: }
2610: 
2611: static PyObject* THPToMkldnnBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
2612:   HANDLE_TH_ERRORS
2613:   const auto& prop = static_cast<ToMkldnnBackward0*>(self->cdata.get())->self_;
2614:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2615:   return obj.release().ptr();
2616:   END_HANDLE_TH_ERRORS
2617: }
2618: 
2619: static struct PyGetSetDef ToMkldnnBackward0_properties[] = {
2620:   THP_FUNCTION_DEFAULT_PROPERTIES,
2621:   {(char*)"_saved_self", (getter)THPToMkldnnBackward0_self_getter, nullptr, nullptr, nullptr},
2622:   {(char*)"_raw_saved_self", (getter)THPToMkldnnBackward0_self_raw_getter, nullptr, nullptr, nullptr},
2623:   {nullptr} /* sentinel */
2624: };
2625: 
2626: 
2627: 
2628: static struct PyGetSetDef UniqueBackward0_properties[] = {
2629:   THP_FUNCTION_DEFAULT_PROPERTIES,
2630: 
2631:   {nullptr} /* sentinel */
2632: };
2633: 
2634: 
2635: 
2636: static struct PyGetSetDef LiftBackward0_properties[] = {
2637:   THP_FUNCTION_DEFAULT_PROPERTIES,
2638: 
2639:   {nullptr} /* sentinel */
2640: };
```

- EN: The main execution path in this span is carried by `THPLinalgEighBackward0_eigenvalues_getter`, `THPVariable_Wrap`, `THPLinalgEighBackward0_eigenvalues_raw_getter`. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPLinalgEighBackward0_eigenvalues_getter`, `THPVariable_Wrap`, `THPLinalgEighBackward0_eigenvalues_raw_getter` 等函数/方法承载。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2641-2720

```cpp
2641: 
2642: 
2643: 
2644: static struct PyGetSetDef LiftFreshBackward0_properties[] = {
2645:   THP_FUNCTION_DEFAULT_PROPERTIES,
2646: 
2647:   {nullptr} /* sentinel */
2648: };
2649: 
2650: static PyObject* THPWhereBackward0_condition_getter(THPCppFunction *self, void *_unused) {
2651:   HANDLE_TH_ERRORS
2652:   const auto& prop = static_cast<WhereBackward0*>(self->cdata.get())->condition_;
2653:   return THPVariable_Wrap(prop.unpack(self->cdata));
2654:   END_HANDLE_TH_ERRORS
2655: }
2656: 
2657: static PyObject* THPWhereBackward0_condition_raw_getter(THPCppFunction *self, void *_unused) {
2658:   HANDLE_TH_ERRORS
2659:   const auto& prop = static_cast<WhereBackward0*>(self->cdata.get())->condition_;
2660:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2661:   return obj.release().ptr();
2662:   END_HANDLE_TH_ERRORS
2663: }
2664: 
2665: static struct PyGetSetDef WhereBackward0_properties[] = {
2666:   THP_FUNCTION_DEFAULT_PROPERTIES,
2667:   {(char*)"_saved_condition", (getter)THPWhereBackward0_condition_getter, nullptr, nullptr, nullptr},
2668:   {(char*)"_raw_saved_condition", (getter)THPWhereBackward0_condition_raw_getter, nullptr, nullptr, nullptr},
2669:   {nullptr} /* sentinel */
2670: };
2671: 
2672: static PyObject* THPWeightNormInterfaceBackward0_dim_getter(THPCppFunction *self, void *_unused) {
2673:   HANDLE_TH_ERRORS
2674:   auto prop = static_cast<WeightNormInterfaceBackward0*>(self->cdata.get())->dim;
2675:   return PyLong_FromUnsignedLong((int64_t) prop);
2676:   END_HANDLE_TH_ERRORS
2677: }
2678: 
2679: static PyObject* THPWeightNormInterfaceBackward0_g_getter(THPCppFunction *self, void *_unused) {
2680:   HANDLE_TH_ERRORS
2681:   const auto& prop = static_cast<WeightNormInterfaceBackward0*>(self->cdata.get())->g_;
2682:   return THPVariable_Wrap(prop.unpack(self->cdata));
2683:   END_HANDLE_TH_ERRORS
2684: }
2685: 
2686: static PyObject* THPWeightNormInterfaceBackward0_g_raw_getter(THPCppFunction *self, void *_unused) {
2687:   HANDLE_TH_ERRORS
2688:   const auto& prop = static_cast<WeightNormInterfaceBackward0*>(self->cdata.get())->g_;
2689:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2690:   return obj.release().ptr();
2691:   END_HANDLE_TH_ERRORS
2692: }
2693: 
2694: static PyObject* THPWeightNormInterfaceBackward0_v_getter(THPCppFunction *self, void *_unused) {
2695:   HANDLE_TH_ERRORS
2696:   const auto& prop = static_cast<WeightNormInterfaceBackward0*>(self->cdata.get())->v_;
2697:   return THPVariable_Wrap(prop.unpack(self->cdata));
2698:   END_HANDLE_TH_ERRORS
2699: }
2700: 
2701: static PyObject* THPWeightNormInterfaceBackward0_v_raw_getter(THPCppFunction *self, void *_unused) {
2702:   HANDLE_TH_ERRORS
2703:   const auto& prop = static_cast<WeightNormInterfaceBackward0*>(self->cdata.get())->v_;
2704:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2705:   return obj.release().ptr();
2706:   END_HANDLE_TH_ERRORS
2707: }
2708: 
2709: static PyObject* THPWeightNormInterfaceBackward0_result1_getter(THPCppFunction *self, void *_unused) {
2710:   HANDLE_TH_ERRORS
2711:   const auto& prop = static_cast<WeightNormInterfaceBackward0*>(self->cdata.get())->result1_;
2712:   return THPVariable_Wrap(prop.unpack(self->cdata));
2713:   END_HANDLE_TH_ERRORS
2714: }
2715: 
2716: static PyObject* THPWeightNormInterfaceBackward0_result1_raw_getter(THPCppFunction *self, void *_unused) {
2717:   HANDLE_TH_ERRORS
2718:   const auto& prop = static_cast<WeightNormInterfaceBackward0*>(self->cdata.get())->result1_;
2719:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2720:   return obj.release().ptr();
```

- EN: The main execution path in this span is carried by `THPWhereBackward0_condition_getter`, `THPVariable_Wrap`, `THPWhereBackward0_condition_raw_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPWhereBackward0_condition_getter`, `THPVariable_Wrap`, `THPWhereBackward0_condition_raw_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2721-2800

```cpp
2721:   END_HANDLE_TH_ERRORS
2722: }
2723: 
2724: static struct PyGetSetDef WeightNormInterfaceBackward0_properties[] = {
2725:   THP_FUNCTION_DEFAULT_PROPERTIES,
2726:   {(char*)"_saved_dim", (getter)THPWeightNormInterfaceBackward0_dim_getter, nullptr, nullptr, nullptr},
2727:   {(char*)"_saved_g", (getter)THPWeightNormInterfaceBackward0_g_getter, nullptr, nullptr, nullptr},
2728:   {(char*)"_raw_saved_g", (getter)THPWeightNormInterfaceBackward0_g_raw_getter, nullptr, nullptr, nullptr},
2729:   {(char*)"_saved_v", (getter)THPWeightNormInterfaceBackward0_v_getter, nullptr, nullptr, nullptr},
2730:   {(char*)"_raw_saved_v", (getter)THPWeightNormInterfaceBackward0_v_raw_getter, nullptr, nullptr, nullptr},
2731:   {(char*)"_saved_result1", (getter)THPWeightNormInterfaceBackward0_result1_getter, nullptr, nullptr, nullptr},
2732:   {(char*)"_raw_saved_result1", (getter)THPWeightNormInterfaceBackward0_result1_raw_getter, nullptr, nullptr, nullptr},
2733:   {nullptr} /* sentinel */
2734: };
2735: 
2736: 
2737: 
2738: static struct PyGetSetDef ZeroBackward0_properties[] = {
2739:   THP_FUNCTION_DEFAULT_PROPERTIES,
2740: 
2741:   {nullptr} /* sentinel */
2742: };
2743: 
2744: static PyObject* THPSparseMaskBackward0_mask_getter(THPCppFunction *self, void *_unused) {
2745:   HANDLE_TH_ERRORS
2746:   const auto& prop = static_cast<SparseMaskBackward0*>(self->cdata.get())->mask_;
2747:   return THPVariable_Wrap(prop.unpack(self->cdata));
2748:   END_HANDLE_TH_ERRORS
2749: }
2750: 
2751: static PyObject* THPSparseMaskBackward0_mask_raw_getter(THPCppFunction *self, void *_unused) {
2752:   HANDLE_TH_ERRORS
2753:   const auto& prop = static_cast<SparseMaskBackward0*>(self->cdata.get())->mask_;
2754:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2755:   return obj.release().ptr();
2756:   END_HANDLE_TH_ERRORS
2757: }
2758: 
2759: static struct PyGetSetDef SparseMaskBackward0_properties[] = {
2760:   THP_FUNCTION_DEFAULT_PROPERTIES,
2761:   {(char*)"_saved_mask", (getter)THPSparseMaskBackward0_mask_getter, nullptr, nullptr, nullptr},
2762:   {(char*)"_raw_saved_mask", (getter)THPSparseMaskBackward0_mask_raw_getter, nullptr, nullptr, nullptr},
2763:   {nullptr} /* sentinel */
2764: };
2765: 
2766: static PyObject* THPSparseSumBackward0_dim_getter(THPCppFunction *self, void *_unused) {
2767:   HANDLE_TH_ERRORS
2768:   auto prop = static_cast<SparseSumBackward0*>(self->cdata.get())->dim;
2769:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
2770:   for (auto i : c10::irange(prop.size())) {
2771:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
2772:   }
2773:   return tup;
2774:   END_HANDLE_TH_ERRORS
2775: }
2776: 
2777: static PyObject* THPSparseSumBackward0_self_getter(THPCppFunction *self, void *_unused) {
2778:   HANDLE_TH_ERRORS
2779:   const auto& prop = static_cast<SparseSumBackward0*>(self->cdata.get())->self_;
2780:   return THPVariable_Wrap(prop.unpack(self->cdata));
2781:   END_HANDLE_TH_ERRORS
2782: }
2783: 
2784: static PyObject* THPSparseSumBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
2785:   HANDLE_TH_ERRORS
2786:   const auto& prop = static_cast<SparseSumBackward0*>(self->cdata.get())->self_;
2787:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2788:   return obj.release().ptr();
2789:   END_HANDLE_TH_ERRORS
2790: }
2791: 
2792: static struct PyGetSetDef SparseSumBackward0_properties[] = {
2793:   THP_FUNCTION_DEFAULT_PROPERTIES,
2794:   {(char*)"_saved_dim", (getter)THPSparseSumBackward0_dim_getter, nullptr, nullptr, nullptr},
2795:   {(char*)"_saved_self", (getter)THPSparseSumBackward0_self_getter, nullptr, nullptr, nullptr},
2796:   {(char*)"_raw_saved_self", (getter)THPSparseSumBackward0_self_raw_getter, nullptr, nullptr, nullptr},
2797:   {nullptr} /* sentinel */
2798: };
2799: 
2800: static PyObject* THPStandardGammaBackward0_self_getter(THPCppFunction *self, void *_unused) {
```

- EN: The main execution path in this span is carried by `THPSparseMaskBackward0_mask_getter`, `THPVariable_Wrap`, `THPSparseMaskBackward0_mask_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPSparseMaskBackward0_mask_getter`, `THPVariable_Wrap`, `THPSparseMaskBackward0_mask_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2801-2880

```cpp
2801:   HANDLE_TH_ERRORS
2802:   const auto& prop = static_cast<StandardGammaBackward0*>(self->cdata.get())->self_;
2803:   return THPVariable_Wrap(prop.unpack(self->cdata));
2804:   END_HANDLE_TH_ERRORS
2805: }
2806: 
2807: static PyObject* THPStandardGammaBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
2808:   HANDLE_TH_ERRORS
2809:   const auto& prop = static_cast<StandardGammaBackward0*>(self->cdata.get())->self_;
2810:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2811:   return obj.release().ptr();
2812:   END_HANDLE_TH_ERRORS
2813: }
2814: 
2815: static PyObject* THPStandardGammaBackward0_result_getter(THPCppFunction *self, void *_unused) {
2816:   HANDLE_TH_ERRORS
2817:   const auto& prop = static_cast<StandardGammaBackward0*>(self->cdata.get())->result_;
2818:   return THPVariable_Wrap(prop.unpack(self->cdata));
2819:   END_HANDLE_TH_ERRORS
2820: }
2821: 
2822: static PyObject* THPStandardGammaBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
2823:   HANDLE_TH_ERRORS
2824:   const auto& prop = static_cast<StandardGammaBackward0*>(self->cdata.get())->result_;
2825:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2826:   return obj.release().ptr();
2827:   END_HANDLE_TH_ERRORS
2828: }
2829: 
2830: static struct PyGetSetDef StandardGammaBackward0_properties[] = {
2831:   THP_FUNCTION_DEFAULT_PROPERTIES,
2832:   {(char*)"_saved_self", (getter)THPStandardGammaBackward0_self_getter, nullptr, nullptr, nullptr},
2833:   {(char*)"_raw_saved_self", (getter)THPStandardGammaBackward0_self_raw_getter, nullptr, nullptr, nullptr},
2834:   {(char*)"_saved_result", (getter)THPStandardGammaBackward0_result_getter, nullptr, nullptr, nullptr},
2835:   {(char*)"_raw_saved_result", (getter)THPStandardGammaBackward0_result_raw_getter, nullptr, nullptr, nullptr},
2836:   {nullptr} /* sentinel */
2837: };
2838: 
2839: static PyObject* THPValuesBackward0_self_getter(THPCppFunction *self, void *_unused) {
2840:   HANDLE_TH_ERRORS
2841:   const auto& prop = static_cast<ValuesBackward0*>(self->cdata.get())->self_;
2842:   return THPVariable_Wrap(prop.unpack(self->cdata));
2843:   END_HANDLE_TH_ERRORS
2844: }
2845: 
2846: static PyObject* THPValuesBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
2847:   HANDLE_TH_ERRORS
2848:   const auto& prop = static_cast<ValuesBackward0*>(self->cdata.get())->self_;
2849:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2850:   return obj.release().ptr();
2851:   END_HANDLE_TH_ERRORS
2852: }
2853: 
2854: static struct PyGetSetDef ValuesBackward0_properties[] = {
2855:   THP_FUNCTION_DEFAULT_PROPERTIES,
2856:   {(char*)"_saved_self", (getter)THPValuesBackward0_self_getter, nullptr, nullptr, nullptr},
2857:   {(char*)"_raw_saved_self", (getter)THPValuesBackward0_self_raw_getter, nullptr, nullptr, nullptr},
2858:   {nullptr} /* sentinel */
2859: };
2860: 
2861: static PyObject* THPValuesBackwardAutogradNestedTensor0_self_getter(THPCppFunction *self, void *_unused) {
2862:   HANDLE_TH_ERRORS
2863:   const auto& prop = static_cast<ValuesBackwardAutogradNestedTensor0*>(self->cdata.get())->self_;
2864:   return THPVariable_Wrap(prop.unpack(self->cdata));
2865:   END_HANDLE_TH_ERRORS
2866: }
2867: 
2868: static PyObject* THPValuesBackwardAutogradNestedTensor0_self_raw_getter(THPCppFunction *self, void *_unused) {
2869:   HANDLE_TH_ERRORS
2870:   const auto& prop = static_cast<ValuesBackwardAutogradNestedTensor0*>(self->cdata.get())->self_;
2871:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2872:   return obj.release().ptr();
2873:   END_HANDLE_TH_ERRORS
2874: }
2875: 
2876: static struct PyGetSetDef ValuesBackwardAutogradNestedTensor0_properties[] = {
2877:   THP_FUNCTION_DEFAULT_PROPERTIES,
2878:   {(char*)"_saved_self", (getter)THPValuesBackwardAutogradNestedTensor0_self_getter, nullptr, nullptr, nullptr},
2879:   {(char*)"_raw_saved_self", (getter)THPValuesBackwardAutogradNestedTensor0_self_raw_getter, nullptr, nullptr, nullptr},
2880:   {nullptr} /* sentinel */
```

- EN: The main execution path in this span is carried by `THPVariable_Wrap`, `THPStandardGammaBackward0_self_raw_getter`, `cast`. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_Wrap`, `THPStandardGammaBackward0_self_raw_getter`, `cast` 等函数/方法承载。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2881-2960

```cpp
2881: };
2882: 
2883: static PyObject* THPEmbeddingDenseBackwardBackward0_indices_getter(THPCppFunction *self, void *_unused) {
2884:   HANDLE_TH_ERRORS
2885:   const auto& prop = static_cast<EmbeddingDenseBackwardBackward0*>(self->cdata.get())->indices_;
2886:   return THPVariable_Wrap(prop.unpack(self->cdata));
2887:   END_HANDLE_TH_ERRORS
2888: }
2889: 
2890: static PyObject* THPEmbeddingDenseBackwardBackward0_indices_raw_getter(THPCppFunction *self, void *_unused) {
2891:   HANDLE_TH_ERRORS
2892:   const auto& prop = static_cast<EmbeddingDenseBackwardBackward0*>(self->cdata.get())->indices_;
2893:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2894:   return obj.release().ptr();
2895:   END_HANDLE_TH_ERRORS
2896: }
2897: 
2898: static PyObject* THPEmbeddingDenseBackwardBackward0_padding_idx_getter(THPCppFunction *self, void *_unused) {
2899:   HANDLE_TH_ERRORS
2900:   auto prop = static_cast<EmbeddingDenseBackwardBackward0*>(self->cdata.get())->padding_idx;
2901:   if (auto m = prop.maybe_as_int()) {
2902:     return PyLong_FromUnsignedLong(*m);
2903:   } else {
2904:     return py::cast(prop).release().ptr();
2905:   }
2906:   END_HANDLE_TH_ERRORS
2907: }
2908: 
2909: static struct PyGetSetDef EmbeddingDenseBackwardBackward0_properties[] = {
2910:   THP_FUNCTION_DEFAULT_PROPERTIES,
2911:   {(char*)"_saved_indices", (getter)THPEmbeddingDenseBackwardBackward0_indices_getter, nullptr, nullptr, nullptr},
2912:   {(char*)"_raw_saved_indices", (getter)THPEmbeddingDenseBackwardBackward0_indices_raw_getter, nullptr, nullptr, nullptr},
2913:   {(char*)"_saved_padding_idx", (getter)THPEmbeddingDenseBackwardBackward0_padding_idx_getter, nullptr, nullptr, nullptr},
2914:   {nullptr} /* sentinel */
2915: };
2916: 
2917: static PyObject* THPEmbeddingBagBackward0_indices_getter(THPCppFunction *self, void *_unused) {
2918:   HANDLE_TH_ERRORS
2919:   const auto& prop = static_cast<EmbeddingBagBackward0*>(self->cdata.get())->indices_;
2920:   return THPVariable_Wrap(prop.unpack(self->cdata));
2921:   END_HANDLE_TH_ERRORS
2922: }
2923: 
2924: static PyObject* THPEmbeddingBagBackward0_indices_raw_getter(THPCppFunction *self, void *_unused) {
2925:   HANDLE_TH_ERRORS
2926:   const auto& prop = static_cast<EmbeddingBagBackward0*>(self->cdata.get())->indices_;
2927:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2928:   return obj.release().ptr();
2929:   END_HANDLE_TH_ERRORS
2930: }
2931: 
2932: static PyObject* THPEmbeddingBagBackward0_mode_getter(THPCppFunction *self, void *_unused) {
2933:   HANDLE_TH_ERRORS
2934:   auto prop = static_cast<EmbeddingBagBackward0*>(self->cdata.get())->mode;
2935:   return PyLong_FromUnsignedLong((int64_t) prop);
2936:   END_HANDLE_TH_ERRORS
2937: }
2938: 
2939: static PyObject* THPEmbeddingBagBackward0_offsets_getter(THPCppFunction *self, void *_unused) {
2940:   HANDLE_TH_ERRORS
2941:   const auto& prop = static_cast<EmbeddingBagBackward0*>(self->cdata.get())->offsets_;
2942:   return THPVariable_Wrap(prop.unpack(self->cdata));
2943:   END_HANDLE_TH_ERRORS
2944: }
2945: 
2946: static PyObject* THPEmbeddingBagBackward0_offsets_raw_getter(THPCppFunction *self, void *_unused) {
2947:   HANDLE_TH_ERRORS
2948:   const auto& prop = static_cast<EmbeddingBagBackward0*>(self->cdata.get())->offsets_;
2949:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2950:   return obj.release().ptr();
2951:   END_HANDLE_TH_ERRORS
2952: }
2953: 
2954: static PyObject* THPEmbeddingBagBackward0_padding_idx_getter(THPCppFunction *self, void *_unused) {
2955:   HANDLE_TH_ERRORS
2956:   auto prop = static_cast<EmbeddingBagBackward0*>(self->cdata.get())->padding_idx;
2957:   return PyLong_FromUnsignedLong((int64_t) prop);
2958:   END_HANDLE_TH_ERRORS
2959: }
2960: 
```

- EN: The main execution path in this span is carried by `THPEmbeddingDenseBackwardBackward0_indices_getter`, `THPVariable_Wrap`, `THPEmbeddingDenseBackwardBackward0_indices_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPEmbeddingDenseBackwardBackward0_indices_getter`, `THPVariable_Wrap`, `THPEmbeddingDenseBackwardBackward0_indices_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2961-3040

```cpp
2961: static PyObject* THPEmbeddingBagBackward0_per_sample_weights_getter(THPCppFunction *self, void *_unused) {
2962:   HANDLE_TH_ERRORS
2963:   const auto& prop = static_cast<EmbeddingBagBackward0*>(self->cdata.get())->per_sample_weights_;
2964:   return THPVariable_Wrap(prop.unpack(self->cdata));
2965:   END_HANDLE_TH_ERRORS
2966: }
2967: 
2968: static PyObject* THPEmbeddingBagBackward0_per_sample_weights_raw_getter(THPCppFunction *self, void *_unused) {
2969:   HANDLE_TH_ERRORS
2970:   const auto& prop = static_cast<EmbeddingBagBackward0*>(self->cdata.get())->per_sample_weights_;
2971:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2972:   return obj.release().ptr();
2973:   END_HANDLE_TH_ERRORS
2974: }
2975: 
2976: static PyObject* THPEmbeddingBagBackward0_scale_grad_by_freq_getter(THPCppFunction *self, void *_unused) {
2977:   HANDLE_TH_ERRORS
2978:   auto prop = static_cast<EmbeddingBagBackward0*>(self->cdata.get())->scale_grad_by_freq;
2979:   if (prop) {
2980:     Py_RETURN_TRUE;
2981:   } else {
2982:     Py_RETURN_FALSE;
2983:   }
2984:   END_HANDLE_TH_ERRORS
2985: }
2986: 
2987: static PyObject* THPEmbeddingBagBackward0_sparse_getter(THPCppFunction *self, void *_unused) {
2988:   HANDLE_TH_ERRORS
2989:   auto prop = static_cast<EmbeddingBagBackward0*>(self->cdata.get())->sparse;
2990:   if (prop) {
2991:     Py_RETURN_TRUE;
2992:   } else {
2993:     Py_RETURN_FALSE;
2994:   }
2995:   END_HANDLE_TH_ERRORS
2996: }
2997: 
2998: static PyObject* THPEmbeddingBagBackward0_weight_getter(THPCppFunction *self, void *_unused) {
2999:   HANDLE_TH_ERRORS
3000:   const auto& prop = static_cast<EmbeddingBagBackward0*>(self->cdata.get())->weight_;
3001:   return THPVariable_Wrap(prop.unpack(self->cdata));
3002:   END_HANDLE_TH_ERRORS
3003: }
3004: 
3005: static PyObject* THPEmbeddingBagBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
3006:   HANDLE_TH_ERRORS
3007:   const auto& prop = static_cast<EmbeddingBagBackward0*>(self->cdata.get())->weight_;
3008:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3009:   return obj.release().ptr();
3010:   END_HANDLE_TH_ERRORS
3011: }
3012: 
3013: static PyObject* THPEmbeddingBagBackward0_weight_sym_argsize_0_getter(THPCppFunction *self, void *_unused) {
3014:   HANDLE_TH_ERRORS
3015:   auto prop = static_cast<EmbeddingBagBackward0*>(self->cdata.get())->weight_sym_argsize_0;
3016:   if (auto m = prop.maybe_as_int()) {
3017:     return PyLong_FromUnsignedLong(*m);
3018:   } else {
3019:     return py::cast(prop).release().ptr();
3020:   }
3021:   END_HANDLE_TH_ERRORS
3022: }
3023: 
3024: static PyObject* THPEmbeddingBagBackward0_result1_getter(THPCppFunction *self, void *_unused) {
3025:   HANDLE_TH_ERRORS
3026:   const auto& prop = static_cast<EmbeddingBagBackward0*>(self->cdata.get())->result1_;
3027:   return THPVariable_Wrap(prop.unpack(self->cdata));
3028:   END_HANDLE_TH_ERRORS
3029: }
3030: 
3031: static PyObject* THPEmbeddingBagBackward0_result1_raw_getter(THPCppFunction *self, void *_unused) {
3032:   HANDLE_TH_ERRORS
3033:   const auto& prop = static_cast<EmbeddingBagBackward0*>(self->cdata.get())->result1_;
3034:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3035:   return obj.release().ptr();
3036:   END_HANDLE_TH_ERRORS
3037: }
3038: 
3039: static PyObject* THPEmbeddingBagBackward0_result2_getter(THPCppFunction *self, void *_unused) {
3040:   HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPEmbeddingBagBackward0_per_sample_weights_getter`, `THPVariable_Wrap`, `THPEmbeddingBagBackward0_per_sample_weights_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPEmbeddingBagBackward0_per_sample_weights_getter`, `THPVariable_Wrap`, `THPEmbeddingBagBackward0_per_sample_weights_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3041-3120

```cpp
3041:   const auto& prop = static_cast<EmbeddingBagBackward0*>(self->cdata.get())->result2_;
3042:   return THPVariable_Wrap(prop.unpack(self->cdata));
3043:   END_HANDLE_TH_ERRORS
3044: }
3045: 
3046: static PyObject* THPEmbeddingBagBackward0_result2_raw_getter(THPCppFunction *self, void *_unused) {
3047:   HANDLE_TH_ERRORS
3048:   const auto& prop = static_cast<EmbeddingBagBackward0*>(self->cdata.get())->result2_;
3049:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3050:   return obj.release().ptr();
3051:   END_HANDLE_TH_ERRORS
3052: }
3053: 
3054: static PyObject* THPEmbeddingBagBackward0_result3_getter(THPCppFunction *self, void *_unused) {
3055:   HANDLE_TH_ERRORS
3056:   const auto& prop = static_cast<EmbeddingBagBackward0*>(self->cdata.get())->result3_;
3057:   return THPVariable_Wrap(prop.unpack(self->cdata));
3058:   END_HANDLE_TH_ERRORS
3059: }
3060: 
3061: static PyObject* THPEmbeddingBagBackward0_result3_raw_getter(THPCppFunction *self, void *_unused) {
3062:   HANDLE_TH_ERRORS
3063:   const auto& prop = static_cast<EmbeddingBagBackward0*>(self->cdata.get())->result3_;
3064:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3065:   return obj.release().ptr();
3066:   END_HANDLE_TH_ERRORS
3067: }
3068: 
3069: static struct PyGetSetDef EmbeddingBagBackward0_properties[] = {
3070:   THP_FUNCTION_DEFAULT_PROPERTIES,
3071:   {(char*)"_saved_indices", (getter)THPEmbeddingBagBackward0_indices_getter, nullptr, nullptr, nullptr},
3072:   {(char*)"_raw_saved_indices", (getter)THPEmbeddingBagBackward0_indices_raw_getter, nullptr, nullptr, nullptr},
3073:   {(char*)"_saved_mode", (getter)THPEmbeddingBagBackward0_mode_getter, nullptr, nullptr, nullptr},
3074:   {(char*)"_saved_offsets", (getter)THPEmbeddingBagBackward0_offsets_getter, nullptr, nullptr, nullptr},
3075:   {(char*)"_raw_saved_offsets", (getter)THPEmbeddingBagBackward0_offsets_raw_getter, nullptr, nullptr, nullptr},
3076:   {(char*)"_saved_padding_idx", (getter)THPEmbeddingBagBackward0_padding_idx_getter, nullptr, nullptr, nullptr},
3077:   {(char*)"_saved_per_sample_weights", (getter)THPEmbeddingBagBackward0_per_sample_weights_getter, nullptr, nullptr, nullptr},
3078:   {(char*)"_raw_saved_per_sample_weights", (getter)THPEmbeddingBagBackward0_per_sample_weights_raw_getter, nullptr, nullptr, nullptr},
3079:   {(char*)"_saved_scale_grad_by_freq", (getter)THPEmbeddingBagBackward0_scale_grad_by_freq_getter, nullptr, nullptr, nullptr},
3080:   {(char*)"_saved_sparse", (getter)THPEmbeddingBagBackward0_sparse_getter, nullptr, nullptr, nullptr},
3081:   {(char*)"_saved_weight", (getter)THPEmbeddingBagBackward0_weight_getter, nullptr, nullptr, nullptr},
3082:   {(char*)"_raw_saved_weight", (getter)THPEmbeddingBagBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
3083:   {(char*)"_saved_weight_sym_argsize_0", (getter)THPEmbeddingBagBackward0_weight_sym_argsize_0_getter, nullptr, nullptr, nullptr},
3084:   {(char*)"_saved_result1", (getter)THPEmbeddingBagBackward0_result1_getter, nullptr, nullptr, nullptr},
3085:   {(char*)"_raw_saved_result1", (getter)THPEmbeddingBagBackward0_result1_raw_getter, nullptr, nullptr, nullptr},
3086:   {(char*)"_saved_result2", (getter)THPEmbeddingBagBackward0_result2_getter, nullptr, nullptr, nullptr},
3087:   {(char*)"_raw_saved_result2", (getter)THPEmbeddingBagBackward0_result2_raw_getter, nullptr, nullptr, nullptr},
3088:   {(char*)"_saved_result3", (getter)THPEmbeddingBagBackward0_result3_getter, nullptr, nullptr, nullptr},
3089:   {(char*)"_raw_saved_result3", (getter)THPEmbeddingBagBackward0_result3_raw_getter, nullptr, nullptr, nullptr},
3090:   {nullptr} /* sentinel */
3091: };
3092: 
3093: static PyObject* THPGeluBackwardBackward0_approximate_getter(THPCppFunction *self, void *_unused) {
3094:   HANDLE_TH_ERRORS
3095:   auto prop = static_cast<GeluBackwardBackward0*>(self->cdata.get())->approximate;
3096:   return PyUnicode_FromStringAndSize(prop.data(), prop.size());
3097:   END_HANDLE_TH_ERRORS
3098: }
3099: 
3100: static PyObject* THPGeluBackwardBackward0_grad_output_getter(THPCppFunction *self, void *_unused) {
3101:   HANDLE_TH_ERRORS
3102:   const auto& prop = static_cast<GeluBackwardBackward0*>(self->cdata.get())->grad_output_;
3103:   return THPVariable_Wrap(prop.unpack(self->cdata));
3104:   END_HANDLE_TH_ERRORS
3105: }
3106: 
3107: static PyObject* THPGeluBackwardBackward0_grad_output_raw_getter(THPCppFunction *self, void *_unused) {
3108:   HANDLE_TH_ERRORS
3109:   const auto& prop = static_cast<GeluBackwardBackward0*>(self->cdata.get())->grad_output_;
3110:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3111:   return obj.release().ptr();
3112:   END_HANDLE_TH_ERRORS
3113: }
3114: 
3115: static PyObject* THPGeluBackwardBackward0_self_getter(THPCppFunction *self, void *_unused) {
3116:   HANDLE_TH_ERRORS
3117:   const auto& prop = static_cast<GeluBackwardBackward0*>(self->cdata.get())->self_;
3118:   return THPVariable_Wrap(prop.unpack(self->cdata));
3119:   END_HANDLE_TH_ERRORS
3120: }
```

- EN: The main execution path in this span is carried by `THPVariable_Wrap`, `THPEmbeddingBagBackward0_result2_raw_getter`, `cast`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_Wrap`, `THPEmbeddingBagBackward0_result2_raw_getter`, `cast` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3121-3200

```cpp
3121: 
3122: static PyObject* THPGeluBackwardBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
3123:   HANDLE_TH_ERRORS
3124:   const auto& prop = static_cast<GeluBackwardBackward0*>(self->cdata.get())->self_;
3125:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3126:   return obj.release().ptr();
3127:   END_HANDLE_TH_ERRORS
3128: }
3129: 
3130: static struct PyGetSetDef GeluBackwardBackward0_properties[] = {
3131:   THP_FUNCTION_DEFAULT_PROPERTIES,
3132:   {(char*)"_saved_approximate", (getter)THPGeluBackwardBackward0_approximate_getter, nullptr, nullptr, nullptr},
3133:   {(char*)"_saved_grad_output", (getter)THPGeluBackwardBackward0_grad_output_getter, nullptr, nullptr, nullptr},
3134:   {(char*)"_raw_saved_grad_output", (getter)THPGeluBackwardBackward0_grad_output_raw_getter, nullptr, nullptr, nullptr},
3135:   {(char*)"_saved_self", (getter)THPGeluBackwardBackward0_self_getter, nullptr, nullptr, nullptr},
3136:   {(char*)"_raw_saved_self", (getter)THPGeluBackwardBackward0_self_raw_getter, nullptr, nullptr, nullptr},
3137:   {nullptr} /* sentinel */
3138: };
3139: 
3140: static PyObject* THPLogSoftmaxBackward0_dim_getter(THPCppFunction *self, void *_unused) {
3141:   HANDLE_TH_ERRORS
3142:   auto prop = static_cast<LogSoftmaxBackward0*>(self->cdata.get())->dim;
3143:   return PyLong_FromUnsignedLong((int64_t) prop);
3144:   END_HANDLE_TH_ERRORS
3145: }
3146: 
3147: static PyObject* THPLogSoftmaxBackward0_result_getter(THPCppFunction *self, void *_unused) {
3148:   HANDLE_TH_ERRORS
3149:   const auto& prop = static_cast<LogSoftmaxBackward0*>(self->cdata.get())->result_;
3150:   return THPVariable_Wrap(prop.unpack(self->cdata));
3151:   END_HANDLE_TH_ERRORS
3152: }
3153: 
3154: static PyObject* THPLogSoftmaxBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
3155:   HANDLE_TH_ERRORS
3156:   const auto& prop = static_cast<LogSoftmaxBackward0*>(self->cdata.get())->result_;
3157:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3158:   return obj.release().ptr();
3159:   END_HANDLE_TH_ERRORS
3160: }
3161: 
3162: static struct PyGetSetDef LogSoftmaxBackward0_properties[] = {
3163:   THP_FUNCTION_DEFAULT_PROPERTIES,
3164:   {(char*)"_saved_dim", (getter)THPLogSoftmaxBackward0_dim_getter, nullptr, nullptr, nullptr},
3165:   {(char*)"_saved_result", (getter)THPLogSoftmaxBackward0_result_getter, nullptr, nullptr, nullptr},
3166:   {(char*)"_raw_saved_result", (getter)THPLogSoftmaxBackward0_result_raw_getter, nullptr, nullptr, nullptr},
3167:   {nullptr} /* sentinel */
3168: };
3169: 
3170: static PyObject* THPPreluKernelBackwardBackward0_grad_output_getter(THPCppFunction *self, void *_unused) {
3171:   HANDLE_TH_ERRORS
3172:   const auto& prop = static_cast<PreluKernelBackwardBackward0*>(self->cdata.get())->grad_output_;
3173:   return THPVariable_Wrap(prop.unpack(self->cdata));
3174:   END_HANDLE_TH_ERRORS
3175: }
3176: 
3177: static PyObject* THPPreluKernelBackwardBackward0_grad_output_raw_getter(THPCppFunction *self, void *_unused) {
3178:   HANDLE_TH_ERRORS
3179:   const auto& prop = static_cast<PreluKernelBackwardBackward0*>(self->cdata.get())->grad_output_;
3180:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3181:   return obj.release().ptr();
3182:   END_HANDLE_TH_ERRORS
3183: }
3184: 
3185: static PyObject* THPPreluKernelBackwardBackward0_self_getter(THPCppFunction *self, void *_unused) {
3186:   HANDLE_TH_ERRORS
3187:   const auto& prop = static_cast<PreluKernelBackwardBackward0*>(self->cdata.get())->self_;
3188:   return THPVariable_Wrap(prop.unpack(self->cdata));
3189:   END_HANDLE_TH_ERRORS
3190: }
3191: 
3192: static PyObject* THPPreluKernelBackwardBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
3193:   HANDLE_TH_ERRORS
3194:   const auto& prop = static_cast<PreluKernelBackwardBackward0*>(self->cdata.get())->self_;
3195:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3196:   return obj.release().ptr();
3197:   END_HANDLE_TH_ERRORS
3198: }
3199: 
3200: static PyObject* THPPreluKernelBackwardBackward0_weight_getter(THPCppFunction *self, void *_unused) {
```

- EN: The main execution path in this span is carried by `THPGeluBackwardBackward0_self_raw_getter`, `cast`, `THPLogSoftmaxBackward0_dim_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPGeluBackwardBackward0_self_raw_getter`, `cast`, `THPLogSoftmaxBackward0_dim_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3201-3280

```cpp
3201:   HANDLE_TH_ERRORS
3202:   const auto& prop = static_cast<PreluKernelBackwardBackward0*>(self->cdata.get())->weight_;
3203:   return THPVariable_Wrap(prop.unpack(self->cdata));
3204:   END_HANDLE_TH_ERRORS
3205: }
3206: 
3207: static PyObject* THPPreluKernelBackwardBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
3208:   HANDLE_TH_ERRORS
3209:   const auto& prop = static_cast<PreluKernelBackwardBackward0*>(self->cdata.get())->weight_;
3210:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3211:   return obj.release().ptr();
3212:   END_HANDLE_TH_ERRORS
3213: }
3214: 
3215: static struct PyGetSetDef PreluKernelBackwardBackward0_properties[] = {
3216:   THP_FUNCTION_DEFAULT_PROPERTIES,
3217:   {(char*)"_saved_grad_output", (getter)THPPreluKernelBackwardBackward0_grad_output_getter, nullptr, nullptr, nullptr},
3218:   {(char*)"_raw_saved_grad_output", (getter)THPPreluKernelBackwardBackward0_grad_output_raw_getter, nullptr, nullptr, nullptr},
3219:   {(char*)"_saved_self", (getter)THPPreluKernelBackwardBackward0_self_getter, nullptr, nullptr, nullptr},
3220:   {(char*)"_raw_saved_self", (getter)THPPreluKernelBackwardBackward0_self_raw_getter, nullptr, nullptr, nullptr},
3221:   {(char*)"_saved_weight", (getter)THPPreluKernelBackwardBackward0_weight_getter, nullptr, nullptr, nullptr},
3222:   {(char*)"_raw_saved_weight", (getter)THPPreluKernelBackwardBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
3223:   {nullptr} /* sentinel */
3224: };
3225: 
3226: static PyObject* THPReflectionPad1DBackward0_padding_getter(THPCppFunction *self, void *_unused) {
3227:   HANDLE_TH_ERRORS
3228:   auto prop = static_cast<ReflectionPad1DBackward0*>(self->cdata.get())->padding;
3229:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3230:   for (auto i : c10::irange(prop.size())) {
3231:       auto si = prop[i];
3232:       if (auto m = si.maybe_as_int()) {
3233:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3234:       } else {
3235:         auto py_symint = py::cast(si).release().ptr();
3236:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3237:       }
3238:   }
3239:   return tup;
3240:   END_HANDLE_TH_ERRORS
3241: }
3242: 
3243: static PyObject* THPReflectionPad1DBackward0_self_getter(THPCppFunction *self, void *_unused) {
3244:   HANDLE_TH_ERRORS
3245:   const auto& prop = static_cast<ReflectionPad1DBackward0*>(self->cdata.get())->self_;
3246:   return THPVariable_Wrap(prop.unpack(self->cdata));
3247:   END_HANDLE_TH_ERRORS
3248: }
3249: 
3250: static PyObject* THPReflectionPad1DBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
3251:   HANDLE_TH_ERRORS
3252:   const auto& prop = static_cast<ReflectionPad1DBackward0*>(self->cdata.get())->self_;
3253:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3254:   return obj.release().ptr();
3255:   END_HANDLE_TH_ERRORS
3256: }
3257: 
3258: static struct PyGetSetDef ReflectionPad1DBackward0_properties[] = {
3259:   THP_FUNCTION_DEFAULT_PROPERTIES,
3260:   {(char*)"_saved_padding", (getter)THPReflectionPad1DBackward0_padding_getter, nullptr, nullptr, nullptr},
3261:   {(char*)"_saved_self", (getter)THPReflectionPad1DBackward0_self_getter, nullptr, nullptr, nullptr},
3262:   {(char*)"_raw_saved_self", (getter)THPReflectionPad1DBackward0_self_raw_getter, nullptr, nullptr, nullptr},
3263:   {nullptr} /* sentinel */
3264: };
3265: 
3266: static PyObject* THPUpsampleBicubic2DAaBackward0_align_corners_getter(THPCppFunction *self, void *_unused) {
3267:   HANDLE_TH_ERRORS
3268:   auto prop = static_cast<UpsampleBicubic2DAaBackward0*>(self->cdata.get())->align_corners;
3269:   if (prop) {
3270:     Py_RETURN_TRUE;
3271:   } else {
3272:     Py_RETURN_FALSE;
3273:   }
3274:   END_HANDLE_TH_ERRORS
3275: }
3276: 
3277: static PyObject* THPUpsampleBicubic2DAaBackward0_output_size_getter(THPCppFunction *self, void *_unused) {
3278:   HANDLE_TH_ERRORS
3279:   auto prop = static_cast<UpsampleBicubic2DAaBackward0*>(self->cdata.get())->output_size;
3280:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
```

- EN: The main execution path in this span is carried by `THPVariable_Wrap`, `THPPreluKernelBackwardBackward0_weight_raw_getter`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_Wrap`, `THPPreluKernelBackwardBackward0_weight_raw_getter`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3281-3360

```cpp
3281:   for (auto i : c10::irange(prop.size())) {
3282:       auto si = prop[i];
3283:       if (auto m = si.maybe_as_int()) {
3284:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3285:       } else {
3286:         auto py_symint = py::cast(si).release().ptr();
3287:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3288:       }
3289:   }
3290:   return tup;
3291:   END_HANDLE_TH_ERRORS
3292: }
3293: 
3294: static PyObject* THPUpsampleBicubic2DAaBackward0_scales_h_getter(THPCppFunction *self, void *_unused) {
3295:   HANDLE_TH_ERRORS
3296:   auto opt_prop = static_cast<UpsampleBicubic2DAaBackward0*>(self->cdata.get())->scales_h;
3297:   if (!opt_prop.has_value()) {
3298:     Py_RETURN_NONE;
3299:   }
3300:   auto prop = opt_prop.value();
3301:   return PyFloat_FromDouble((double) prop);
3302:   END_HANDLE_TH_ERRORS
3303: }
3304: 
3305: static PyObject* THPUpsampleBicubic2DAaBackward0_scales_w_getter(THPCppFunction *self, void *_unused) {
3306:   HANDLE_TH_ERRORS
3307:   auto opt_prop = static_cast<UpsampleBicubic2DAaBackward0*>(self->cdata.get())->scales_w;
3308:   if (!opt_prop.has_value()) {
3309:     Py_RETURN_NONE;
3310:   }
3311:   auto prop = opt_prop.value();
3312:   return PyFloat_FromDouble((double) prop);
3313:   END_HANDLE_TH_ERRORS
3314: }
3315: 
3316: static PyObject* THPUpsampleBicubic2DAaBackward0_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
3317:   HANDLE_TH_ERRORS
3318:   auto prop = static_cast<UpsampleBicubic2DAaBackward0*>(self->cdata.get())->self_sym_sizes;
3319:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3320:   for (auto i : c10::irange(prop.size())) {
3321:       auto si = prop[i];
3322:       if (auto m = si.maybe_as_int()) {
3323:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3324:       } else {
3325:         auto py_symint = py::cast(si).release().ptr();
3326:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3327:       }
3328:   }
3329:   return tup;
3330:   END_HANDLE_TH_ERRORS
3331: }
3332: 
3333: static struct PyGetSetDef UpsampleBicubic2DAaBackward0_properties[] = {
3334:   THP_FUNCTION_DEFAULT_PROPERTIES,
3335:   {(char*)"_saved_align_corners", (getter)THPUpsampleBicubic2DAaBackward0_align_corners_getter, nullptr, nullptr, nullptr},
3336:   {(char*)"_saved_output_size", (getter)THPUpsampleBicubic2DAaBackward0_output_size_getter, nullptr, nullptr, nullptr},
3337:   {(char*)"_saved_scales_h", (getter)THPUpsampleBicubic2DAaBackward0_scales_h_getter, nullptr, nullptr, nullptr},
3338:   {(char*)"_saved_scales_w", (getter)THPUpsampleBicubic2DAaBackward0_scales_w_getter, nullptr, nullptr, nullptr},
3339:   {(char*)"_saved_self_sym_sizes", (getter)THPUpsampleBicubic2DAaBackward0_self_sym_sizes_getter, nullptr, nullptr, nullptr},
3340:   {nullptr} /* sentinel */
3341: };
3342: 
3343: static PyObject* THPUpsampleLanczos2DAaBackward0_align_corners_getter(THPCppFunction *self, void *_unused) {
3344:   HANDLE_TH_ERRORS
3345:   auto prop = static_cast<UpsampleLanczos2DAaBackward0*>(self->cdata.get())->align_corners;
3346:   if (prop) {
3347:     Py_RETURN_TRUE;
3348:   } else {
3349:     Py_RETURN_FALSE;
3350:   }
3351:   END_HANDLE_TH_ERRORS
3352: }
3353: 
3354: static PyObject* THPUpsampleLanczos2DAaBackward0_output_size_getter(THPCppFunction *self, void *_unused) {
3355:   HANDLE_TH_ERRORS
3356:   auto prop = static_cast<UpsampleLanczos2DAaBackward0*>(self->cdata.get())->output_size;
3357:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3358:   for (auto i : c10::irange(prop.size())) {
3359:       auto si = prop[i];
3360:       if (auto m = si.maybe_as_int()) {
```

- EN: The main execution path in this span is carried by `PyTuple_SetItem`, `cast`, `THPUpsampleBicubic2DAaBackward0_scales_h_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyTuple_SetItem`, `cast`, `THPUpsampleBicubic2DAaBackward0_scales_h_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3361-3440

```cpp
3361:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3362:       } else {
3363:         auto py_symint = py::cast(si).release().ptr();
3364:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3365:       }
3366:   }
3367:   return tup;
3368:   END_HANDLE_TH_ERRORS
3369: }
3370: 
3371: static PyObject* THPUpsampleLanczos2DAaBackward0_scales_h_getter(THPCppFunction *self, void *_unused) {
3372:   HANDLE_TH_ERRORS
3373:   auto opt_prop = static_cast<UpsampleLanczos2DAaBackward0*>(self->cdata.get())->scales_h;
3374:   if (!opt_prop.has_value()) {
3375:     Py_RETURN_NONE;
3376:   }
3377:   auto prop = opt_prop.value();
3378:   return PyFloat_FromDouble((double) prop);
3379:   END_HANDLE_TH_ERRORS
3380: }
3381: 
3382: static PyObject* THPUpsampleLanczos2DAaBackward0_scales_w_getter(THPCppFunction *self, void *_unused) {
3383:   HANDLE_TH_ERRORS
3384:   auto opt_prop = static_cast<UpsampleLanczos2DAaBackward0*>(self->cdata.get())->scales_w;
3385:   if (!opt_prop.has_value()) {
3386:     Py_RETURN_NONE;
3387:   }
3388:   auto prop = opt_prop.value();
3389:   return PyFloat_FromDouble((double) prop);
3390:   END_HANDLE_TH_ERRORS
3391: }
3392: 
3393: static PyObject* THPUpsampleLanczos2DAaBackward0_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
3394:   HANDLE_TH_ERRORS
3395:   auto prop = static_cast<UpsampleLanczos2DAaBackward0*>(self->cdata.get())->self_sym_sizes;
3396:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3397:   for (auto i : c10::irange(prop.size())) {
3398:       auto si = prop[i];
3399:       if (auto m = si.maybe_as_int()) {
3400:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3401:       } else {
3402:         auto py_symint = py::cast(si).release().ptr();
3403:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3404:       }
3405:   }
3406:   return tup;
3407:   END_HANDLE_TH_ERRORS
3408: }
3409: 
3410: static struct PyGetSetDef UpsampleLanczos2DAaBackward0_properties[] = {
3411:   THP_FUNCTION_DEFAULT_PROPERTIES,
3412:   {(char*)"_saved_align_corners", (getter)THPUpsampleLanczos2DAaBackward0_align_corners_getter, nullptr, nullptr, nullptr},
3413:   {(char*)"_saved_output_size", (getter)THPUpsampleLanczos2DAaBackward0_output_size_getter, nullptr, nullptr, nullptr},
3414:   {(char*)"_saved_scales_h", (getter)THPUpsampleLanczos2DAaBackward0_scales_h_getter, nullptr, nullptr, nullptr},
3415:   {(char*)"_saved_scales_w", (getter)THPUpsampleLanczos2DAaBackward0_scales_w_getter, nullptr, nullptr, nullptr},
3416:   {(char*)"_saved_self_sym_sizes", (getter)THPUpsampleLanczos2DAaBackward0_self_sym_sizes_getter, nullptr, nullptr, nullptr},
3417:   {nullptr} /* sentinel */
3418: };
3419: 
3420: static PyObject* THPUpsampleTrilinear3DBackward0_align_corners_getter(THPCppFunction *self, void *_unused) {
3421:   HANDLE_TH_ERRORS
3422:   auto prop = static_cast<UpsampleTrilinear3DBackward0*>(self->cdata.get())->align_corners;
3423:   if (prop) {
3424:     Py_RETURN_TRUE;
3425:   } else {
3426:     Py_RETURN_FALSE;
3427:   }
3428:   END_HANDLE_TH_ERRORS
3429: }
3430: 
3431: static PyObject* THPUpsampleTrilinear3DBackward0_output_size_getter(THPCppFunction *self, void *_unused) {
3432:   HANDLE_TH_ERRORS
3433:   auto prop = static_cast<UpsampleTrilinear3DBackward0*>(self->cdata.get())->output_size;
3434:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3435:   for (auto i : c10::irange(prop.size())) {
3436:       auto si = prop[i];
3437:       if (auto m = si.maybe_as_int()) {
3438:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3439:       } else {
3440:         auto py_symint = py::cast(si).release().ptr();
```

- EN: The main execution path in this span is carried by `PyTuple_SetItem`, `cast`, `THPUpsampleLanczos2DAaBackward0_scales_h_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyTuple_SetItem`, `cast`, `THPUpsampleLanczos2DAaBackward0_scales_h_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3441-3520

```cpp
3441:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3442:       }
3443:   }
3444:   return tup;
3445:   END_HANDLE_TH_ERRORS
3446: }
3447: 
3448: static PyObject* THPUpsampleTrilinear3DBackward0_scales_d_getter(THPCppFunction *self, void *_unused) {
3449:   HANDLE_TH_ERRORS
3450:   auto opt_prop = static_cast<UpsampleTrilinear3DBackward0*>(self->cdata.get())->scales_d;
3451:   if (!opt_prop.has_value()) {
3452:     Py_RETURN_NONE;
3453:   }
3454:   auto prop = opt_prop.value();
3455:   return PyFloat_FromDouble((double) prop);
3456:   END_HANDLE_TH_ERRORS
3457: }
3458: 
3459: static PyObject* THPUpsampleTrilinear3DBackward0_scales_h_getter(THPCppFunction *self, void *_unused) {
3460:   HANDLE_TH_ERRORS
3461:   auto opt_prop = static_cast<UpsampleTrilinear3DBackward0*>(self->cdata.get())->scales_h;
3462:   if (!opt_prop.has_value()) {
3463:     Py_RETURN_NONE;
3464:   }
3465:   auto prop = opt_prop.value();
3466:   return PyFloat_FromDouble((double) prop);
3467:   END_HANDLE_TH_ERRORS
3468: }
3469: 
3470: static PyObject* THPUpsampleTrilinear3DBackward0_scales_w_getter(THPCppFunction *self, void *_unused) {
3471:   HANDLE_TH_ERRORS
3472:   auto opt_prop = static_cast<UpsampleTrilinear3DBackward0*>(self->cdata.get())->scales_w;
3473:   if (!opt_prop.has_value()) {
3474:     Py_RETURN_NONE;
3475:   }
3476:   auto prop = opt_prop.value();
3477:   return PyFloat_FromDouble((double) prop);
3478:   END_HANDLE_TH_ERRORS
3479: }
3480: 
3481: static PyObject* THPUpsampleTrilinear3DBackward0_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
3482:   HANDLE_TH_ERRORS
3483:   auto prop = static_cast<UpsampleTrilinear3DBackward0*>(self->cdata.get())->self_sym_sizes;
3484:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3485:   for (auto i : c10::irange(prop.size())) {
3486:       auto si = prop[i];
3487:       if (auto m = si.maybe_as_int()) {
3488:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3489:       } else {
3490:         auto py_symint = py::cast(si).release().ptr();
3491:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3492:       }
3493:   }
3494:   return tup;
3495:   END_HANDLE_TH_ERRORS
3496: }
3497: 
3498: static struct PyGetSetDef UpsampleTrilinear3DBackward0_properties[] = {
3499:   THP_FUNCTION_DEFAULT_PROPERTIES,
3500:   {(char*)"_saved_align_corners", (getter)THPUpsampleTrilinear3DBackward0_align_corners_getter, nullptr, nullptr, nullptr},
3501:   {(char*)"_saved_output_size", (getter)THPUpsampleTrilinear3DBackward0_output_size_getter, nullptr, nullptr, nullptr},
3502:   {(char*)"_saved_scales_d", (getter)THPUpsampleTrilinear3DBackward0_scales_d_getter, nullptr, nullptr, nullptr},
3503:   {(char*)"_saved_scales_h", (getter)THPUpsampleTrilinear3DBackward0_scales_h_getter, nullptr, nullptr, nullptr},
3504:   {(char*)"_saved_scales_w", (getter)THPUpsampleTrilinear3DBackward0_scales_w_getter, nullptr, nullptr, nullptr},
3505:   {(char*)"_saved_self_sym_sizes", (getter)THPUpsampleTrilinear3DBackward0_self_sym_sizes_getter, nullptr, nullptr, nullptr},
3506:   {nullptr} /* sentinel */
3507: };
3508: 
3509: static PyObject* THPUpsampleNearest3DBackward0_output_size_getter(THPCppFunction *self, void *_unused) {
3510:   HANDLE_TH_ERRORS
3511:   auto prop = static_cast<UpsampleNearest3DBackward0*>(self->cdata.get())->output_size;
3512:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3513:   for (auto i : c10::irange(prop.size())) {
3514:       auto si = prop[i];
3515:       if (auto m = si.maybe_as_int()) {
3516:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3517:       } else {
3518:         auto py_symint = py::cast(si).release().ptr();
3519:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3520:       }
```

- EN: The main execution path in this span is carried by `PyTuple_SetItem`, `THPUpsampleTrilinear3DBackward0_scales_d_getter`, `PyFloat_FromDouble`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyTuple_SetItem`, `THPUpsampleTrilinear3DBackward0_scales_d_getter`, `PyFloat_FromDouble` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3521-3600

```cpp
3521:   }
3522:   return tup;
3523:   END_HANDLE_TH_ERRORS
3524: }
3525: 
3526: static PyObject* THPUpsampleNearest3DBackward0_scales_d_getter(THPCppFunction *self, void *_unused) {
3527:   HANDLE_TH_ERRORS
3528:   auto opt_prop = static_cast<UpsampleNearest3DBackward0*>(self->cdata.get())->scales_d;
3529:   if (!opt_prop.has_value()) {
3530:     Py_RETURN_NONE;
3531:   }
3532:   auto prop = opt_prop.value();
3533:   return PyFloat_FromDouble((double) prop);
3534:   END_HANDLE_TH_ERRORS
3535: }
3536: 
3537: static PyObject* THPUpsampleNearest3DBackward0_scales_h_getter(THPCppFunction *self, void *_unused) {
3538:   HANDLE_TH_ERRORS
3539:   auto opt_prop = static_cast<UpsampleNearest3DBackward0*>(self->cdata.get())->scales_h;
3540:   if (!opt_prop.has_value()) {
3541:     Py_RETURN_NONE;
3542:   }
3543:   auto prop = opt_prop.value();
3544:   return PyFloat_FromDouble((double) prop);
3545:   END_HANDLE_TH_ERRORS
3546: }
3547: 
3548: static PyObject* THPUpsampleNearest3DBackward0_scales_w_getter(THPCppFunction *self, void *_unused) {
3549:   HANDLE_TH_ERRORS
3550:   auto opt_prop = static_cast<UpsampleNearest3DBackward0*>(self->cdata.get())->scales_w;
3551:   if (!opt_prop.has_value()) {
3552:     Py_RETURN_NONE;
3553:   }
3554:   auto prop = opt_prop.value();
3555:   return PyFloat_FromDouble((double) prop);
3556:   END_HANDLE_TH_ERRORS
3557: }
3558: 
3559: static PyObject* THPUpsampleNearest3DBackward0_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
3560:   HANDLE_TH_ERRORS
3561:   auto prop = static_cast<UpsampleNearest3DBackward0*>(self->cdata.get())->self_sym_sizes;
3562:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3563:   for (auto i : c10::irange(prop.size())) {
3564:       auto si = prop[i];
3565:       if (auto m = si.maybe_as_int()) {
3566:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3567:       } else {
3568:         auto py_symint = py::cast(si).release().ptr();
3569:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3570:       }
3571:   }
3572:   return tup;
3573:   END_HANDLE_TH_ERRORS
3574: }
3575: 
3576: static struct PyGetSetDef UpsampleNearest3DBackward0_properties[] = {
3577:   THP_FUNCTION_DEFAULT_PROPERTIES,
3578:   {(char*)"_saved_output_size", (getter)THPUpsampleNearest3DBackward0_output_size_getter, nullptr, nullptr, nullptr},
3579:   {(char*)"_saved_scales_d", (getter)THPUpsampleNearest3DBackward0_scales_d_getter, nullptr, nullptr, nullptr},
3580:   {(char*)"_saved_scales_h", (getter)THPUpsampleNearest3DBackward0_scales_h_getter, nullptr, nullptr, nullptr},
3581:   {(char*)"_saved_scales_w", (getter)THPUpsampleNearest3DBackward0_scales_w_getter, nullptr, nullptr, nullptr},
3582:   {(char*)"_saved_self_sym_sizes", (getter)THPUpsampleNearest3DBackward0_self_sym_sizes_getter, nullptr, nullptr, nullptr},
3583:   {nullptr} /* sentinel */
3584: };
3585: 
3586: static PyObject* THPPixelUnshuffleBackward0_downscale_factor_getter(THPCppFunction *self, void *_unused) {
3587:   HANDLE_TH_ERRORS
3588:   auto prop = static_cast<PixelUnshuffleBackward0*>(self->cdata.get())->downscale_factor;
3589:   return PyLong_FromUnsignedLong((int64_t) prop);
3590:   END_HANDLE_TH_ERRORS
3591: }
3592: 
3593: static struct PyGetSetDef PixelUnshuffleBackward0_properties[] = {
3594:   THP_FUNCTION_DEFAULT_PROPERTIES,
3595:   {(char*)"_saved_downscale_factor", (getter)THPPixelUnshuffleBackward0_downscale_factor_getter, nullptr, nullptr, nullptr},
3596:   {nullptr} /* sentinel */
3597: };
3598: 
3599: static PyObject* THPLinearBackward0_input_getter(THPCppFunction *self, void *_unused) {
3600:   HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPUpsampleNearest3DBackward0_scales_d_getter`, `PyFloat_FromDouble`, `THPUpsampleNearest3DBackward0_scales_h_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPUpsampleNearest3DBackward0_scales_d_getter`, `PyFloat_FromDouble`, `THPUpsampleNearest3DBackward0_scales_h_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3601-3680

```cpp
3601:   const auto& prop = static_cast<LinearBackward0*>(self->cdata.get())->input_;
3602:   return THPVariable_Wrap(prop.unpack(self->cdata));
3603:   END_HANDLE_TH_ERRORS
3604: }
3605: 
3606: static PyObject* THPLinearBackward0_input_raw_getter(THPCppFunction *self, void *_unused) {
3607:   HANDLE_TH_ERRORS
3608:   const auto& prop = static_cast<LinearBackward0*>(self->cdata.get())->input_;
3609:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3610:   return obj.release().ptr();
3611:   END_HANDLE_TH_ERRORS
3612: }
3613: 
3614: static PyObject* THPLinearBackward0_weight_getter(THPCppFunction *self, void *_unused) {
3615:   HANDLE_TH_ERRORS
3616:   const auto& prop = static_cast<LinearBackward0*>(self->cdata.get())->weight_;
3617:   return THPVariable_Wrap(prop.unpack(self->cdata));
3618:   END_HANDLE_TH_ERRORS
3619: }
3620: 
3621: static PyObject* THPLinearBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
3622:   HANDLE_TH_ERRORS
3623:   const auto& prop = static_cast<LinearBackward0*>(self->cdata.get())->weight_;
3624:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3625:   return obj.release().ptr();
3626:   END_HANDLE_TH_ERRORS
3627: }
3628: 
3629: static struct PyGetSetDef LinearBackward0_properties[] = {
3630:   THP_FUNCTION_DEFAULT_PROPERTIES,
3631:   {(char*)"_saved_input", (getter)THPLinearBackward0_input_getter, nullptr, nullptr, nullptr},
3632:   {(char*)"_raw_saved_input", (getter)THPLinearBackward0_input_raw_getter, nullptr, nullptr, nullptr},
3633:   {(char*)"_saved_weight", (getter)THPLinearBackward0_weight_getter, nullptr, nullptr, nullptr},
3634:   {(char*)"_raw_saved_weight", (getter)THPLinearBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
3635:   {nullptr} /* sentinel */
3636: };
3637: 
3638: static PyObject* THPMaxPool2DWithIndicesBackward0_ceil_mode_getter(THPCppFunction *self, void *_unused) {
3639:   HANDLE_TH_ERRORS
3640:   auto prop = static_cast<MaxPool2DWithIndicesBackward0*>(self->cdata.get())->ceil_mode;
3641:   if (prop) {
3642:     Py_RETURN_TRUE;
3643:   } else {
3644:     Py_RETURN_FALSE;
3645:   }
3646:   END_HANDLE_TH_ERRORS
3647: }
3648: 
3649: static PyObject* THPMaxPool2DWithIndicesBackward0_dilation_getter(THPCppFunction *self, void *_unused) {
3650:   HANDLE_TH_ERRORS
3651:   auto prop = static_cast<MaxPool2DWithIndicesBackward0*>(self->cdata.get())->dilation;
3652:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3653:   for (auto i : c10::irange(prop.size())) {
3654:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
3655:   }
3656:   return tup;
3657:   END_HANDLE_TH_ERRORS
3658: }
3659: 
3660: static PyObject* THPMaxPool2DWithIndicesBackward0_kernel_size_getter(THPCppFunction *self, void *_unused) {
3661:   HANDLE_TH_ERRORS
3662:   auto prop = static_cast<MaxPool2DWithIndicesBackward0*>(self->cdata.get())->kernel_size;
3663:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3664:   for (auto i : c10::irange(prop.size())) {
3665:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
3666:   }
3667:   return tup;
3668:   END_HANDLE_TH_ERRORS
3669: }
3670: 
3671: static PyObject* THPMaxPool2DWithIndicesBackward0_padding_getter(THPCppFunction *self, void *_unused) {
3672:   HANDLE_TH_ERRORS
3673:   auto prop = static_cast<MaxPool2DWithIndicesBackward0*>(self->cdata.get())->padding;
3674:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3675:   for (auto i : c10::irange(prop.size())) {
3676:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
3677:   }
3678:   return tup;
3679:   END_HANDLE_TH_ERRORS
3680: }
```

- EN: The main execution path in this span is carried by `THPVariable_Wrap`, `THPLinearBackward0_input_raw_getter`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_Wrap`, `THPLinearBackward0_input_raw_getter`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 3681-3760

```cpp
3681: 
3682: static PyObject* THPMaxPool2DWithIndicesBackward0_self_getter(THPCppFunction *self, void *_unused) {
3683:   HANDLE_TH_ERRORS
3684:   const auto& prop = static_cast<MaxPool2DWithIndicesBackward0*>(self->cdata.get())->self_;
3685:   return THPVariable_Wrap(prop.unpack(self->cdata));
3686:   END_HANDLE_TH_ERRORS
3687: }
3688: 
3689: static PyObject* THPMaxPool2DWithIndicesBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
3690:   HANDLE_TH_ERRORS
3691:   const auto& prop = static_cast<MaxPool2DWithIndicesBackward0*>(self->cdata.get())->self_;
3692:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3693:   return obj.release().ptr();
3694:   END_HANDLE_TH_ERRORS
3695: }
3696: 
3697: static PyObject* THPMaxPool2DWithIndicesBackward0_stride_getter(THPCppFunction *self, void *_unused) {
3698:   HANDLE_TH_ERRORS
3699:   auto prop = static_cast<MaxPool2DWithIndicesBackward0*>(self->cdata.get())->stride;
3700:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3701:   for (auto i : c10::irange(prop.size())) {
3702:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
3703:   }
3704:   return tup;
3705:   END_HANDLE_TH_ERRORS
3706: }
3707: 
3708: static PyObject* THPMaxPool2DWithIndicesBackward0_result1_getter(THPCppFunction *self, void *_unused) {
3709:   HANDLE_TH_ERRORS
3710:   const auto& prop = static_cast<MaxPool2DWithIndicesBackward0*>(self->cdata.get())->result1_;
3711:   return THPVariable_Wrap(prop.unpack(self->cdata));
3712:   END_HANDLE_TH_ERRORS
3713: }
3714: 
3715: static PyObject* THPMaxPool2DWithIndicesBackward0_result1_raw_getter(THPCppFunction *self, void *_unused) {
3716:   HANDLE_TH_ERRORS
3717:   const auto& prop = static_cast<MaxPool2DWithIndicesBackward0*>(self->cdata.get())->result1_;
3718:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3719:   return obj.release().ptr();
3720:   END_HANDLE_TH_ERRORS
3721: }
3722: 
3723: static struct PyGetSetDef MaxPool2DWithIndicesBackward0_properties[] = {
3724:   THP_FUNCTION_DEFAULT_PROPERTIES,
3725:   {(char*)"_saved_ceil_mode", (getter)THPMaxPool2DWithIndicesBackward0_ceil_mode_getter, nullptr, nullptr, nullptr},
3726:   {(char*)"_saved_dilation", (getter)THPMaxPool2DWithIndicesBackward0_dilation_getter, nullptr, nullptr, nullptr},
3727:   {(char*)"_saved_kernel_size", (getter)THPMaxPool2DWithIndicesBackward0_kernel_size_getter, nullptr, nullptr, nullptr},
3728:   {(char*)"_saved_padding", (getter)THPMaxPool2DWithIndicesBackward0_padding_getter, nullptr, nullptr, nullptr},
3729:   {(char*)"_saved_self", (getter)THPMaxPool2DWithIndicesBackward0_self_getter, nullptr, nullptr, nullptr},
3730:   {(char*)"_raw_saved_self", (getter)THPMaxPool2DWithIndicesBackward0_self_raw_getter, nullptr, nullptr, nullptr},
3731:   {(char*)"_saved_stride", (getter)THPMaxPool2DWithIndicesBackward0_stride_getter, nullptr, nullptr, nullptr},
3732:   {(char*)"_saved_result1", (getter)THPMaxPool2DWithIndicesBackward0_result1_getter, nullptr, nullptr, nullptr},
3733:   {(char*)"_raw_saved_result1", (getter)THPMaxPool2DWithIndicesBackward0_result1_raw_getter, nullptr, nullptr, nullptr},
3734:   {nullptr} /* sentinel */
3735: };
3736: 
3737: static PyObject* THPConvolutionBackward0_bias_sym_sizes_opt_getter(THPCppFunction *self, void *_unused) {
3738:   HANDLE_TH_ERRORS
3739:   auto opt_prop = static_cast<ConvolutionBackward0*>(self->cdata.get())->bias_sym_sizes_opt;
3740:   if (!opt_prop.list.has_value()) {
3741:     Py_RETURN_NONE;
3742:   }
3743:   auto prop = opt_prop.list.value();
3744:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3745:   for (auto i : c10::irange(prop.size())) {
3746:       auto si = prop[i];
3747:       if (auto m = si.maybe_as_int()) {
3748:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3749:       } else {
3750:         auto py_symint = py::cast(si).release().ptr();
3751:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3752:       }
3753:   }
3754:   return tup;
3755:   END_HANDLE_TH_ERRORS
3756: }
3757: 
3758: static PyObject* THPConvolutionBackward0_dilation_getter(THPCppFunction *self, void *_unused) {
3759:   HANDLE_TH_ERRORS
3760:   auto prop = static_cast<ConvolutionBackward0*>(self->cdata.get())->dilation;
```

- EN: The main execution path in this span is carried by `THPMaxPool2DWithIndicesBackward0_self_getter`, `THPVariable_Wrap`, `THPMaxPool2DWithIndicesBackward0_self_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPMaxPool2DWithIndicesBackward0_self_getter`, `THPVariable_Wrap`, `THPMaxPool2DWithIndicesBackward0_self_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 3761-3840

```cpp
3761:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3762:   for (auto i : c10::irange(prop.size())) {
3763:       auto si = prop[i];
3764:       if (auto m = si.maybe_as_int()) {
3765:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3766:       } else {
3767:         auto py_symint = py::cast(si).release().ptr();
3768:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3769:       }
3770:   }
3771:   return tup;
3772:   END_HANDLE_TH_ERRORS
3773: }
3774: 
3775: static PyObject* THPConvolutionBackward0_groups_getter(THPCppFunction *self, void *_unused) {
3776:   HANDLE_TH_ERRORS
3777:   auto prop = static_cast<ConvolutionBackward0*>(self->cdata.get())->groups;
3778:   if (auto m = prop.maybe_as_int()) {
3779:     return PyLong_FromUnsignedLong(*m);
3780:   } else {
3781:     return py::cast(prop).release().ptr();
3782:   }
3783:   END_HANDLE_TH_ERRORS
3784: }
3785: 
3786: static PyObject* THPConvolutionBackward0_input_getter(THPCppFunction *self, void *_unused) {
3787:   HANDLE_TH_ERRORS
3788:   const auto& prop = static_cast<ConvolutionBackward0*>(self->cdata.get())->input_;
3789:   return THPVariable_Wrap(prop.unpack(self->cdata));
3790:   END_HANDLE_TH_ERRORS
3791: }
3792: 
3793: static PyObject* THPConvolutionBackward0_input_raw_getter(THPCppFunction *self, void *_unused) {
3794:   HANDLE_TH_ERRORS
3795:   const auto& prop = static_cast<ConvolutionBackward0*>(self->cdata.get())->input_;
3796:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3797:   return obj.release().ptr();
3798:   END_HANDLE_TH_ERRORS
3799: }
3800: 
3801: static PyObject* THPConvolutionBackward0_output_padding_getter(THPCppFunction *self, void *_unused) {
3802:   HANDLE_TH_ERRORS
3803:   auto prop = static_cast<ConvolutionBackward0*>(self->cdata.get())->output_padding;
3804:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3805:   for (auto i : c10::irange(prop.size())) {
3806:       auto si = prop[i];
3807:       if (auto m = si.maybe_as_int()) {
3808:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3809:       } else {
3810:         auto py_symint = py::cast(si).release().ptr();
3811:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3812:       }
3813:   }
3814:   return tup;
3815:   END_HANDLE_TH_ERRORS
3816: }
3817: 
3818: static PyObject* THPConvolutionBackward0_padding_getter(THPCppFunction *self, void *_unused) {
3819:   HANDLE_TH_ERRORS
3820:   auto prop = static_cast<ConvolutionBackward0*>(self->cdata.get())->padding;
3821:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3822:   for (auto i : c10::irange(prop.size())) {
3823:       auto si = prop[i];
3824:       if (auto m = si.maybe_as_int()) {
3825:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3826:       } else {
3827:         auto py_symint = py::cast(si).release().ptr();
3828:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3829:       }
3830:   }
3831:   return tup;
3832:   END_HANDLE_TH_ERRORS
3833: }
3834: 
3835: static PyObject* THPConvolutionBackward0_stride_getter(THPCppFunction *self, void *_unused) {
3836:   HANDLE_TH_ERRORS
3837:   auto prop = static_cast<ConvolutionBackward0*>(self->cdata.get())->stride;
3838:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3839:   for (auto i : c10::irange(prop.size())) {
3840:       auto si = prop[i];
```

- EN: The main execution path in this span is carried by `PyTuple_New`, `PyTuple_SetItem`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyTuple_New`, `PyTuple_SetItem`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3841-3920

```cpp
3841:       if (auto m = si.maybe_as_int()) {
3842:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3843:       } else {
3844:         auto py_symint = py::cast(si).release().ptr();
3845:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3846:       }
3847:   }
3848:   return tup;
3849:   END_HANDLE_TH_ERRORS
3850: }
3851: 
3852: static PyObject* THPConvolutionBackward0_transposed_getter(THPCppFunction *self, void *_unused) {
3853:   HANDLE_TH_ERRORS
3854:   auto prop = static_cast<ConvolutionBackward0*>(self->cdata.get())->transposed;
3855:   if (prop) {
3856:     Py_RETURN_TRUE;
3857:   } else {
3858:     Py_RETURN_FALSE;
3859:   }
3860:   END_HANDLE_TH_ERRORS
3861: }
3862: 
3863: static PyObject* THPConvolutionBackward0_weight_getter(THPCppFunction *self, void *_unused) {
3864:   HANDLE_TH_ERRORS
3865:   const auto& prop = static_cast<ConvolutionBackward0*>(self->cdata.get())->weight_;
3866:   return THPVariable_Wrap(prop.unpack(self->cdata));
3867:   END_HANDLE_TH_ERRORS
3868: }
3869: 
3870: static PyObject* THPConvolutionBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
3871:   HANDLE_TH_ERRORS
3872:   const auto& prop = static_cast<ConvolutionBackward0*>(self->cdata.get())->weight_;
3873:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3874:   return obj.release().ptr();
3875:   END_HANDLE_TH_ERRORS
3876: }
3877: 
3878: static struct PyGetSetDef ConvolutionBackward0_properties[] = {
3879:   THP_FUNCTION_DEFAULT_PROPERTIES,
3880:   {(char*)"_saved_bias_sym_sizes_opt", (getter)THPConvolutionBackward0_bias_sym_sizes_opt_getter, nullptr, nullptr, nullptr},
3881:   {(char*)"_saved_dilation", (getter)THPConvolutionBackward0_dilation_getter, nullptr, nullptr, nullptr},
3882:   {(char*)"_saved_groups", (getter)THPConvolutionBackward0_groups_getter, nullptr, nullptr, nullptr},
3883:   {(char*)"_saved_input", (getter)THPConvolutionBackward0_input_getter, nullptr, nullptr, nullptr},
3884:   {(char*)"_raw_saved_input", (getter)THPConvolutionBackward0_input_raw_getter, nullptr, nullptr, nullptr},
3885:   {(char*)"_saved_output_padding", (getter)THPConvolutionBackward0_output_padding_getter, nullptr, nullptr, nullptr},
3886:   {(char*)"_saved_padding", (getter)THPConvolutionBackward0_padding_getter, nullptr, nullptr, nullptr},
3887:   {(char*)"_saved_stride", (getter)THPConvolutionBackward0_stride_getter, nullptr, nullptr, nullptr},
3888:   {(char*)"_saved_transposed", (getter)THPConvolutionBackward0_transposed_getter, nullptr, nullptr, nullptr},
3889:   {(char*)"_saved_weight", (getter)THPConvolutionBackward0_weight_getter, nullptr, nullptr, nullptr},
3890:   {(char*)"_raw_saved_weight", (getter)THPConvolutionBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
3891:   {nullptr} /* sentinel */
3892: };
3893: 
3894: static PyObject* THPConvolutionBackward1_bias_sym_sizes_opt_getter(THPCppFunction *self, void *_unused) {
3895:   HANDLE_TH_ERRORS
3896:   auto opt_prop = static_cast<ConvolutionBackward1*>(self->cdata.get())->bias_sym_sizes_opt;
3897:   if (!opt_prop.list.has_value()) {
3898:     Py_RETURN_NONE;
3899:   }
3900:   auto prop = opt_prop.list.value();
3901:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3902:   for (auto i : c10::irange(prop.size())) {
3903:       auto si = prop[i];
3904:       if (auto m = si.maybe_as_int()) {
3905:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3906:       } else {
3907:         auto py_symint = py::cast(si).release().ptr();
3908:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3909:       }
3910:   }
3911:   return tup;
3912:   END_HANDLE_TH_ERRORS
3913: }
3914: 
3915: static PyObject* THPConvolutionBackward1_dilation_getter(THPCppFunction *self, void *_unused) {
3916:   HANDLE_TH_ERRORS
3917:   auto prop = static_cast<ConvolutionBackward1*>(self->cdata.get())->dilation;
3918:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3919:   for (auto i : c10::irange(prop.size())) {
3920:       auto si = prop[i];
```

- EN: The main execution path in this span is carried by `PyTuple_SetItem`, `cast`, `THPConvolutionBackward0_transposed_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyTuple_SetItem`, `cast`, `THPConvolutionBackward0_transposed_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3921-4000

```cpp
3921:       if (auto m = si.maybe_as_int()) {
3922:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3923:       } else {
3924:         auto py_symint = py::cast(si).release().ptr();
3925:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3926:       }
3927:   }
3928:   return tup;
3929:   END_HANDLE_TH_ERRORS
3930: }
3931: 
3932: static PyObject* THPConvolutionBackward1_groups_getter(THPCppFunction *self, void *_unused) {
3933:   HANDLE_TH_ERRORS
3934:   auto prop = static_cast<ConvolutionBackward1*>(self->cdata.get())->groups;
3935:   if (auto m = prop.maybe_as_int()) {
3936:     return PyLong_FromUnsignedLong(*m);
3937:   } else {
3938:     return py::cast(prop).release().ptr();
3939:   }
3940:   END_HANDLE_TH_ERRORS
3941: }
3942: 
3943: static PyObject* THPConvolutionBackward1_input_getter(THPCppFunction *self, void *_unused) {
3944:   HANDLE_TH_ERRORS
3945:   const auto& prop = static_cast<ConvolutionBackward1*>(self->cdata.get())->input_;
3946:   return THPVariable_Wrap(prop.unpack(self->cdata));
3947:   END_HANDLE_TH_ERRORS
3948: }
3949: 
3950: static PyObject* THPConvolutionBackward1_input_raw_getter(THPCppFunction *self, void *_unused) {
3951:   HANDLE_TH_ERRORS
3952:   const auto& prop = static_cast<ConvolutionBackward1*>(self->cdata.get())->input_;
3953:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3954:   return obj.release().ptr();
3955:   END_HANDLE_TH_ERRORS
3956: }
3957: 
3958: static PyObject* THPConvolutionBackward1_output_padding_getter(THPCppFunction *self, void *_unused) {
3959:   HANDLE_TH_ERRORS
3960:   auto prop = static_cast<ConvolutionBackward1*>(self->cdata.get())->output_padding;
3961:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3962:   for (auto i : c10::irange(prop.size())) {
3963:       auto si = prop[i];
3964:       if (auto m = si.maybe_as_int()) {
3965:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3966:       } else {
3967:         auto py_symint = py::cast(si).release().ptr();
3968:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3969:       }
3970:   }
3971:   return tup;
3972:   END_HANDLE_TH_ERRORS
3973: }
3974: 
3975: static PyObject* THPConvolutionBackward1_padding_getter(THPCppFunction *self, void *_unused) {
3976:   HANDLE_TH_ERRORS
3977:   auto prop = static_cast<ConvolutionBackward1*>(self->cdata.get())->padding;
3978:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3979:   for (auto i : c10::irange(prop.size())) {
3980:       auto si = prop[i];
3981:       if (auto m = si.maybe_as_int()) {
3982:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3983:       } else {
3984:         auto py_symint = py::cast(si).release().ptr();
3985:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3986:       }
3987:   }
3988:   return tup;
3989:   END_HANDLE_TH_ERRORS
3990: }
3991: 
3992: static PyObject* THPConvolutionBackward1_stride_getter(THPCppFunction *self, void *_unused) {
3993:   HANDLE_TH_ERRORS
3994:   auto prop = static_cast<ConvolutionBackward1*>(self->cdata.get())->stride;
3995:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3996:   for (auto i : c10::irange(prop.size())) {
3997:       auto si = prop[i];
3998:       if (auto m = si.maybe_as_int()) {
3999:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4000:       } else {
```

- EN: The main execution path in this span is carried by `PyTuple_SetItem`, `cast`, `THPConvolutionBackward1_groups_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyTuple_SetItem`, `cast`, `THPConvolutionBackward1_groups_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4001-4080

```cpp
4001:         auto py_symint = py::cast(si).release().ptr();
4002:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4003:       }
4004:   }
4005:   return tup;
4006:   END_HANDLE_TH_ERRORS
4007: }
4008: 
4009: static PyObject* THPConvolutionBackward1_transposed_getter(THPCppFunction *self, void *_unused) {
4010:   HANDLE_TH_ERRORS
4011:   auto prop = static_cast<ConvolutionBackward1*>(self->cdata.get())->transposed;
4012:   if (prop) {
4013:     Py_RETURN_TRUE;
4014:   } else {
4015:     Py_RETURN_FALSE;
4016:   }
4017:   END_HANDLE_TH_ERRORS
4018: }
4019: 
4020: static PyObject* THPConvolutionBackward1_weight_getter(THPCppFunction *self, void *_unused) {
4021:   HANDLE_TH_ERRORS
4022:   const auto& prop = static_cast<ConvolutionBackward1*>(self->cdata.get())->weight_;
4023:   return THPVariable_Wrap(prop.unpack(self->cdata));
4024:   END_HANDLE_TH_ERRORS
4025: }
4026: 
4027: static PyObject* THPConvolutionBackward1_weight_raw_getter(THPCppFunction *self, void *_unused) {
4028:   HANDLE_TH_ERRORS
4029:   const auto& prop = static_cast<ConvolutionBackward1*>(self->cdata.get())->weight_;
4030:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4031:   return obj.release().ptr();
4032:   END_HANDLE_TH_ERRORS
4033: }
4034: 
4035: static struct PyGetSetDef ConvolutionBackward1_properties[] = {
4036:   THP_FUNCTION_DEFAULT_PROPERTIES,
4037:   {(char*)"_saved_bias_sym_sizes_opt", (getter)THPConvolutionBackward1_bias_sym_sizes_opt_getter, nullptr, nullptr, nullptr},
4038:   {(char*)"_saved_dilation", (getter)THPConvolutionBackward1_dilation_getter, nullptr, nullptr, nullptr},
4039:   {(char*)"_saved_groups", (getter)THPConvolutionBackward1_groups_getter, nullptr, nullptr, nullptr},
4040:   {(char*)"_saved_input", (getter)THPConvolutionBackward1_input_getter, nullptr, nullptr, nullptr},
4041:   {(char*)"_raw_saved_input", (getter)THPConvolutionBackward1_input_raw_getter, nullptr, nullptr, nullptr},
4042:   {(char*)"_saved_output_padding", (getter)THPConvolutionBackward1_output_padding_getter, nullptr, nullptr, nullptr},
4043:   {(char*)"_saved_padding", (getter)THPConvolutionBackward1_padding_getter, nullptr, nullptr, nullptr},
4044:   {(char*)"_saved_stride", (getter)THPConvolutionBackward1_stride_getter, nullptr, nullptr, nullptr},
4045:   {(char*)"_saved_transposed", (getter)THPConvolutionBackward1_transposed_getter, nullptr, nullptr, nullptr},
4046:   {(char*)"_saved_weight", (getter)THPConvolutionBackward1_weight_getter, nullptr, nullptr, nullptr},
4047:   {(char*)"_raw_saved_weight", (getter)THPConvolutionBackward1_weight_raw_getter, nullptr, nullptr, nullptr},
4048:   {nullptr} /* sentinel */
4049: };
4050: 
4051: static PyObject* THPConvolutionBackwardOverrideableBackward0_dilation_getter(THPCppFunction *self, void *_unused) {
4052:   HANDLE_TH_ERRORS
4053:   auto prop = static_cast<ConvolutionBackwardOverrideableBackward0*>(self->cdata.get())->dilation;
4054:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4055:   for (auto i : c10::irange(prop.size())) {
4056:       auto si = prop[i];
4057:       if (auto m = si.maybe_as_int()) {
4058:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4059:       } else {
4060:         auto py_symint = py::cast(si).release().ptr();
4061:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4062:       }
4063:   }
4064:   return tup;
4065:   END_HANDLE_TH_ERRORS
4066: }
4067: 
4068: static PyObject* THPConvolutionBackwardOverrideableBackward0_grad_output_getter(THPCppFunction *self, void *_unused) {
4069:   HANDLE_TH_ERRORS
4070:   const auto& prop = static_cast<ConvolutionBackwardOverrideableBackward0*>(self->cdata.get())->grad_output_;
4071:   return THPVariable_Wrap(prop.unpack(self->cdata));
4072:   END_HANDLE_TH_ERRORS
4073: }
4074: 
4075: static PyObject* THPConvolutionBackwardOverrideableBackward0_grad_output_raw_getter(THPCppFunction *self, void *_unused) {
4076:   HANDLE_TH_ERRORS
4077:   const auto& prop = static_cast<ConvolutionBackwardOverrideableBackward0*>(self->cdata.get())->grad_output_;
4078:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4079:   return obj.release().ptr();
4080:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `cast`, `PyTuple_SetItem`, `THPConvolutionBackward1_transposed_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `cast`, `PyTuple_SetItem`, `THPConvolutionBackward1_transposed_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4081-4160

```cpp
4081: }
4082: 
4083: static PyObject* THPConvolutionBackwardOverrideableBackward0_groups_getter(THPCppFunction *self, void *_unused) {
4084:   HANDLE_TH_ERRORS
4085:   auto prop = static_cast<ConvolutionBackwardOverrideableBackward0*>(self->cdata.get())->groups;
4086:   if (auto m = prop.maybe_as_int()) {
4087:     return PyLong_FromUnsignedLong(*m);
4088:   } else {
4089:     return py::cast(prop).release().ptr();
4090:   }
4091:   END_HANDLE_TH_ERRORS
4092: }
4093: 
4094: static PyObject* THPConvolutionBackwardOverrideableBackward0_input_getter(THPCppFunction *self, void *_unused) {
4095:   HANDLE_TH_ERRORS
4096:   const auto& prop = static_cast<ConvolutionBackwardOverrideableBackward0*>(self->cdata.get())->input_;
4097:   return THPVariable_Wrap(prop.unpack(self->cdata));
4098:   END_HANDLE_TH_ERRORS
4099: }
4100: 
4101: static PyObject* THPConvolutionBackwardOverrideableBackward0_input_raw_getter(THPCppFunction *self, void *_unused) {
4102:   HANDLE_TH_ERRORS
4103:   const auto& prop = static_cast<ConvolutionBackwardOverrideableBackward0*>(self->cdata.get())->input_;
4104:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4105:   return obj.release().ptr();
4106:   END_HANDLE_TH_ERRORS
4107: }
4108: 
4109: static PyObject* THPConvolutionBackwardOverrideableBackward0_output_padding_getter(THPCppFunction *self, void *_unused) {
4110:   HANDLE_TH_ERRORS
4111:   auto prop = static_cast<ConvolutionBackwardOverrideableBackward0*>(self->cdata.get())->output_padding;
4112:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4113:   for (auto i : c10::irange(prop.size())) {
4114:       auto si = prop[i];
4115:       if (auto m = si.maybe_as_int()) {
4116:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4117:       } else {
4118:         auto py_symint = py::cast(si).release().ptr();
4119:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4120:       }
4121:   }
4122:   return tup;
4123:   END_HANDLE_TH_ERRORS
4124: }
4125: 
4126: static PyObject* THPConvolutionBackwardOverrideableBackward0_padding_getter(THPCppFunction *self, void *_unused) {
4127:   HANDLE_TH_ERRORS
4128:   auto prop = static_cast<ConvolutionBackwardOverrideableBackward0*>(self->cdata.get())->padding;
4129:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4130:   for (auto i : c10::irange(prop.size())) {
4131:       auto si = prop[i];
4132:       if (auto m = si.maybe_as_int()) {
4133:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4134:       } else {
4135:         auto py_symint = py::cast(si).release().ptr();
4136:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4137:       }
4138:   }
4139:   return tup;
4140:   END_HANDLE_TH_ERRORS
4141: }
4142: 
4143: static PyObject* THPConvolutionBackwardOverrideableBackward0_stride_getter(THPCppFunction *self, void *_unused) {
4144:   HANDLE_TH_ERRORS
4145:   auto prop = static_cast<ConvolutionBackwardOverrideableBackward0*>(self->cdata.get())->stride;
4146:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4147:   for (auto i : c10::irange(prop.size())) {
4148:       auto si = prop[i];
4149:       if (auto m = si.maybe_as_int()) {
4150:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4151:       } else {
4152:         auto py_symint = py::cast(si).release().ptr();
4153:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4154:       }
4155:   }
4156:   return tup;
4157:   END_HANDLE_TH_ERRORS
4158: }
4159: 
4160: static PyObject* THPConvolutionBackwardOverrideableBackward0_transposed_getter(THPCppFunction *self, void *_unused) {
```

- EN: The main execution path in this span is carried by `THPConvolutionBackwardOverrideableBackward0_groups_getter`, `PyLong_FromUnsignedLong`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPConvolutionBackwardOverrideableBackward0_groups_getter`, `PyLong_FromUnsignedLong`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4161-4240

```cpp
4161:   HANDLE_TH_ERRORS
4162:   auto prop = static_cast<ConvolutionBackwardOverrideableBackward0*>(self->cdata.get())->transposed;
4163:   if (prop) {
4164:     Py_RETURN_TRUE;
4165:   } else {
4166:     Py_RETURN_FALSE;
4167:   }
4168:   END_HANDLE_TH_ERRORS
4169: }
4170: 
4171: static PyObject* THPConvolutionBackwardOverrideableBackward0_weight_getter(THPCppFunction *self, void *_unused) {
4172:   HANDLE_TH_ERRORS
4173:   const auto& prop = static_cast<ConvolutionBackwardOverrideableBackward0*>(self->cdata.get())->weight_;
4174:   return THPVariable_Wrap(prop.unpack(self->cdata));
4175:   END_HANDLE_TH_ERRORS
4176: }
4177: 
4178: static PyObject* THPConvolutionBackwardOverrideableBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
4179:   HANDLE_TH_ERRORS
4180:   const auto& prop = static_cast<ConvolutionBackwardOverrideableBackward0*>(self->cdata.get())->weight_;
4181:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4182:   return obj.release().ptr();
4183:   END_HANDLE_TH_ERRORS
4184: }
4185: 
4186: static struct PyGetSetDef ConvolutionBackwardOverrideableBackward0_properties[] = {
4187:   THP_FUNCTION_DEFAULT_PROPERTIES,
4188:   {(char*)"_saved_dilation", (getter)THPConvolutionBackwardOverrideableBackward0_dilation_getter, nullptr, nullptr, nullptr},
4189:   {(char*)"_saved_grad_output", (getter)THPConvolutionBackwardOverrideableBackward0_grad_output_getter, nullptr, nullptr, nullptr},
4190:   {(char*)"_raw_saved_grad_output", (getter)THPConvolutionBackwardOverrideableBackward0_grad_output_raw_getter, nullptr, nullptr, nullptr},
4191:   {(char*)"_saved_groups", (getter)THPConvolutionBackwardOverrideableBackward0_groups_getter, nullptr, nullptr, nullptr},
4192:   {(char*)"_saved_input", (getter)THPConvolutionBackwardOverrideableBackward0_input_getter, nullptr, nullptr, nullptr},
4193:   {(char*)"_raw_saved_input", (getter)THPConvolutionBackwardOverrideableBackward0_input_raw_getter, nullptr, nullptr, nullptr},
4194:   {(char*)"_saved_output_padding", (getter)THPConvolutionBackwardOverrideableBackward0_output_padding_getter, nullptr, nullptr, nullptr},
4195:   {(char*)"_saved_padding", (getter)THPConvolutionBackwardOverrideableBackward0_padding_getter, nullptr, nullptr, nullptr},
4196:   {(char*)"_saved_stride", (getter)THPConvolutionBackwardOverrideableBackward0_stride_getter, nullptr, nullptr, nullptr},
4197:   {(char*)"_saved_transposed", (getter)THPConvolutionBackwardOverrideableBackward0_transposed_getter, nullptr, nullptr, nullptr},
4198:   {(char*)"_saved_weight", (getter)THPConvolutionBackwardOverrideableBackward0_weight_getter, nullptr, nullptr, nullptr},
4199:   {(char*)"_raw_saved_weight", (getter)THPConvolutionBackwardOverrideableBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
4200:   {nullptr} /* sentinel */
4201: };
4202: 
4203: static PyObject* THPSlowConv2DBackward0_kernel_size_getter(THPCppFunction *self, void *_unused) {
4204:   HANDLE_TH_ERRORS
4205:   auto prop = static_cast<SlowConv2DBackward0*>(self->cdata.get())->kernel_size;
4206:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4207:   for (auto i : c10::irange(prop.size())) {
4208:       auto si = prop[i];
4209:       if (auto m = si.maybe_as_int()) {
4210:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4211:       } else {
4212:         auto py_symint = py::cast(si).release().ptr();
4213:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4214:       }
4215:   }
4216:   return tup;
4217:   END_HANDLE_TH_ERRORS
4218: }
4219: 
4220: static PyObject* THPSlowConv2DBackward0_padding_getter(THPCppFunction *self, void *_unused) {
4221:   HANDLE_TH_ERRORS
4222:   auto prop = static_cast<SlowConv2DBackward0*>(self->cdata.get())->padding;
4223:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4224:   for (auto i : c10::irange(prop.size())) {
4225:       auto si = prop[i];
4226:       if (auto m = si.maybe_as_int()) {
4227:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4228:       } else {
4229:         auto py_symint = py::cast(si).release().ptr();
4230:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4231:       }
4232:   }
4233:   return tup;
4234:   END_HANDLE_TH_ERRORS
4235: }
4236: 
4237: static PyObject* THPSlowConv2DBackward0_self_getter(THPCppFunction *self, void *_unused) {
4238:   HANDLE_TH_ERRORS
4239:   const auto& prop = static_cast<SlowConv2DBackward0*>(self->cdata.get())->self_;
4240:   return THPVariable_Wrap(prop.unpack(self->cdata));
```

- EN: The main execution path in this span is carried by `THPConvolutionBackwardOverrideableBackward0_weight_getter`, `THPVariable_Wrap`, `THPConvolutionBackwardOverrideableBackward0_weight_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPConvolutionBackwardOverrideableBackward0_weight_getter`, `THPVariable_Wrap`, `THPConvolutionBackwardOverrideableBackward0_weight_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 4241-4320

```cpp
4241:   END_HANDLE_TH_ERRORS
4242: }
4243: 
4244: static PyObject* THPSlowConv2DBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
4245:   HANDLE_TH_ERRORS
4246:   const auto& prop = static_cast<SlowConv2DBackward0*>(self->cdata.get())->self_;
4247:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4248:   return obj.release().ptr();
4249:   END_HANDLE_TH_ERRORS
4250: }
4251: 
4252: static PyObject* THPSlowConv2DBackward0_stride_getter(THPCppFunction *self, void *_unused) {
4253:   HANDLE_TH_ERRORS
4254:   auto prop = static_cast<SlowConv2DBackward0*>(self->cdata.get())->stride;
4255:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4256:   for (auto i : c10::irange(prop.size())) {
4257:       auto si = prop[i];
4258:       if (auto m = si.maybe_as_int()) {
4259:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4260:       } else {
4261:         auto py_symint = py::cast(si).release().ptr();
4262:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4263:       }
4264:   }
4265:   return tup;
4266:   END_HANDLE_TH_ERRORS
4267: }
4268: 
4269: static PyObject* THPSlowConv2DBackward0_weight_getter(THPCppFunction *self, void *_unused) {
4270:   HANDLE_TH_ERRORS
4271:   const auto& prop = static_cast<SlowConv2DBackward0*>(self->cdata.get())->weight_;
4272:   return THPVariable_Wrap(prop.unpack(self->cdata));
4273:   END_HANDLE_TH_ERRORS
4274: }
4275: 
4276: static PyObject* THPSlowConv2DBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
4277:   HANDLE_TH_ERRORS
4278:   const auto& prop = static_cast<SlowConv2DBackward0*>(self->cdata.get())->weight_;
4279:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4280:   return obj.release().ptr();
4281:   END_HANDLE_TH_ERRORS
4282: }
4283: 
4284: static struct PyGetSetDef SlowConv2DBackward0_properties[] = {
4285:   THP_FUNCTION_DEFAULT_PROPERTIES,
4286:   {(char*)"_saved_kernel_size", (getter)THPSlowConv2DBackward0_kernel_size_getter, nullptr, nullptr, nullptr},
4287:   {(char*)"_saved_padding", (getter)THPSlowConv2DBackward0_padding_getter, nullptr, nullptr, nullptr},
4288:   {(char*)"_saved_self", (getter)THPSlowConv2DBackward0_self_getter, nullptr, nullptr, nullptr},
4289:   {(char*)"_raw_saved_self", (getter)THPSlowConv2DBackward0_self_raw_getter, nullptr, nullptr, nullptr},
4290:   {(char*)"_saved_stride", (getter)THPSlowConv2DBackward0_stride_getter, nullptr, nullptr, nullptr},
4291:   {(char*)"_saved_weight", (getter)THPSlowConv2DBackward0_weight_getter, nullptr, nullptr, nullptr},
4292:   {(char*)"_raw_saved_weight", (getter)THPSlowConv2DBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
4293:   {nullptr} /* sentinel */
4294: };
4295: 
4296: static PyObject* THPEluBackwardBackward0_alpha_getter(THPCppFunction *self, void *_unused) {
4297:   HANDLE_TH_ERRORS
4298:   auto prop = static_cast<EluBackwardBackward0*>(self->cdata.get())->alpha;
4299:   if (prop.isComplex()) {
4300:     auto cprop = prop.to<c10::complex<double>>();
4301:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
4302:   } else if (prop.isFloatingPoint()) {
4303:     return PyFloat_FromDouble(prop.to<double>());
4304:   } else if (prop.isIntegral(/*includeBool=*/false)) {
4305:     return PyLong_FromLong(prop.to<int64_t>());
4306:   } else if (prop.isBoolean()) {
4307:     if (prop.to<bool>()) {
4308:       Py_RETURN_TRUE;
4309:     } else {
4310:       Py_RETURN_FALSE;
4311:     }
4312:   } else {
4313:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
4314:     return nullptr;
4315:   }
4316:   END_HANDLE_TH_ERRORS
4317: }
4318: 
4319: static PyObject* THPEluBackwardBackward0_grad_output_getter(THPCppFunction *self, void *_unused) {
4320:   HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPSlowConv2DBackward0_self_raw_getter`, `cast`, `THPSlowConv2DBackward0_stride_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPSlowConv2DBackward0_self_raw_getter`, `cast`, `THPSlowConv2DBackward0_stride_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 4321-4400

```cpp
4321:   const auto& prop = static_cast<EluBackwardBackward0*>(self->cdata.get())->grad_output_;
4322:   return THPVariable_Wrap(prop.unpack(self->cdata));
4323:   END_HANDLE_TH_ERRORS
4324: }
4325: 
4326: static PyObject* THPEluBackwardBackward0_grad_output_raw_getter(THPCppFunction *self, void *_unused) {
4327:   HANDLE_TH_ERRORS
4328:   const auto& prop = static_cast<EluBackwardBackward0*>(self->cdata.get())->grad_output_;
4329:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4330:   return obj.release().ptr();
4331:   END_HANDLE_TH_ERRORS
4332: }
4333: 
4334: static PyObject* THPEluBackwardBackward0_input_scale_getter(THPCppFunction *self, void *_unused) {
4335:   HANDLE_TH_ERRORS
4336:   auto prop = static_cast<EluBackwardBackward0*>(self->cdata.get())->input_scale;
4337:   if (prop.isComplex()) {
4338:     auto cprop = prop.to<c10::complex<double>>();
4339:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
4340:   } else if (prop.isFloatingPoint()) {
4341:     return PyFloat_FromDouble(prop.to<double>());
4342:   } else if (prop.isIntegral(/*includeBool=*/false)) {
4343:     return PyLong_FromLong(prop.to<int64_t>());
4344:   } else if (prop.isBoolean()) {
4345:     if (prop.to<bool>()) {
4346:       Py_RETURN_TRUE;
4347:     } else {
4348:       Py_RETURN_FALSE;
4349:     }
4350:   } else {
4351:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
4352:     return nullptr;
4353:   }
4354:   END_HANDLE_TH_ERRORS
4355: }
4356: 
4357: static PyObject* THPEluBackwardBackward0_is_result_getter(THPCppFunction *self, void *_unused) {
4358:   HANDLE_TH_ERRORS
4359:   auto prop = static_cast<EluBackwardBackward0*>(self->cdata.get())->is_result;
4360:   if (prop) {
4361:     Py_RETURN_TRUE;
4362:   } else {
4363:     Py_RETURN_FALSE;
4364:   }
4365:   END_HANDLE_TH_ERRORS
4366: }
4367: 
4368: static PyObject* THPEluBackwardBackward0_scale_getter(THPCppFunction *self, void *_unused) {
4369:   HANDLE_TH_ERRORS
4370:   auto prop = static_cast<EluBackwardBackward0*>(self->cdata.get())->scale;
4371:   if (prop.isComplex()) {
4372:     auto cprop = prop.to<c10::complex<double>>();
4373:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
4374:   } else if (prop.isFloatingPoint()) {
4375:     return PyFloat_FromDouble(prop.to<double>());
4376:   } else if (prop.isIntegral(/*includeBool=*/false)) {
4377:     return PyLong_FromLong(prop.to<int64_t>());
4378:   } else if (prop.isBoolean()) {
4379:     if (prop.to<bool>()) {
4380:       Py_RETURN_TRUE;
4381:     } else {
4382:       Py_RETURN_FALSE;
4383:     }
4384:   } else {
4385:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
4386:     return nullptr;
4387:   }
4388:   END_HANDLE_TH_ERRORS
4389: }
4390: 
4391: static PyObject* THPEluBackwardBackward0_self_or_result_getter(THPCppFunction *self, void *_unused) {
4392:   HANDLE_TH_ERRORS
4393:   const auto& prop = static_cast<EluBackwardBackward0*>(self->cdata.get())->self_or_result_;
4394:   return THPVariable_Wrap(prop.unpack(self->cdata));
4395:   END_HANDLE_TH_ERRORS
4396: }
4397: 
4398: static PyObject* THPEluBackwardBackward0_self_or_result_raw_getter(THPCppFunction *self, void *_unused) {
4399:   HANDLE_TH_ERRORS
4400:   const auto& prop = static_cast<EluBackwardBackward0*>(self->cdata.get())->self_or_result_;
```

- EN: The main execution path in this span is carried by `THPVariable_Wrap`, `THPEluBackwardBackward0_grad_output_raw_getter`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_Wrap`, `THPEluBackwardBackward0_grad_output_raw_getter`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4401-4480

```cpp
4401:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4402:   return obj.release().ptr();
4403:   END_HANDLE_TH_ERRORS
4404: }
4405: 
4406: static struct PyGetSetDef EluBackwardBackward0_properties[] = {
4407:   THP_FUNCTION_DEFAULT_PROPERTIES,
4408:   {(char*)"_saved_alpha", (getter)THPEluBackwardBackward0_alpha_getter, nullptr, nullptr, nullptr},
4409:   {(char*)"_saved_grad_output", (getter)THPEluBackwardBackward0_grad_output_getter, nullptr, nullptr, nullptr},
4410:   {(char*)"_raw_saved_grad_output", (getter)THPEluBackwardBackward0_grad_output_raw_getter, nullptr, nullptr, nullptr},
4411:   {(char*)"_saved_input_scale", (getter)THPEluBackwardBackward0_input_scale_getter, nullptr, nullptr, nullptr},
4412:   {(char*)"_saved_is_result", (getter)THPEluBackwardBackward0_is_result_getter, nullptr, nullptr, nullptr},
4413:   {(char*)"_saved_scale", (getter)THPEluBackwardBackward0_scale_getter, nullptr, nullptr, nullptr},
4414:   {(char*)"_saved_self_or_result", (getter)THPEluBackwardBackward0_self_or_result_getter, nullptr, nullptr, nullptr},
4415:   {(char*)"_raw_saved_self_or_result", (getter)THPEluBackwardBackward0_self_or_result_raw_getter, nullptr, nullptr, nullptr},
4416:   {nullptr} /* sentinel */
4417: };
4418: 
4419: static PyObject* THPFractionalMaxPool3DBackwardBackward0_indices_getter(THPCppFunction *self, void *_unused) {
4420:   HANDLE_TH_ERRORS
4421:   const auto& prop = static_cast<FractionalMaxPool3DBackwardBackward0*>(self->cdata.get())->indices_;
4422:   return THPVariable_Wrap(prop.unpack(self->cdata));
4423:   END_HANDLE_TH_ERRORS
4424: }
4425: 
4426: static PyObject* THPFractionalMaxPool3DBackwardBackward0_indices_raw_getter(THPCppFunction *self, void *_unused) {
4427:   HANDLE_TH_ERRORS
4428:   const auto& prop = static_cast<FractionalMaxPool3DBackwardBackward0*>(self->cdata.get())->indices_;
4429:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4430:   return obj.release().ptr();
4431:   END_HANDLE_TH_ERRORS
4432: }
4433: 
4434: static struct PyGetSetDef FractionalMaxPool3DBackwardBackward0_properties[] = {
4435:   THP_FUNCTION_DEFAULT_PROPERTIES,
4436:   {(char*)"_saved_indices", (getter)THPFractionalMaxPool3DBackwardBackward0_indices_getter, nullptr, nullptr, nullptr},
4437:   {(char*)"_raw_saved_indices", (getter)THPFractionalMaxPool3DBackwardBackward0_indices_raw_getter, nullptr, nullptr, nullptr},
4438:   {nullptr} /* sentinel */
4439: };
4440: 
4441: static PyObject* THPMseLossBackwardBackward0_grad_output_getter(THPCppFunction *self, void *_unused) {
4442:   HANDLE_TH_ERRORS
4443:   const auto& prop = static_cast<MseLossBackwardBackward0*>(self->cdata.get())->grad_output_;
4444:   return THPVariable_Wrap(prop.unpack(self->cdata));
4445:   END_HANDLE_TH_ERRORS
4446: }
4447: 
4448: static PyObject* THPMseLossBackwardBackward0_grad_output_raw_getter(THPCppFunction *self, void *_unused) {
4449:   HANDLE_TH_ERRORS
4450:   const auto& prop = static_cast<MseLossBackwardBackward0*>(self->cdata.get())->grad_output_;
4451:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4452:   return obj.release().ptr();
4453:   END_HANDLE_TH_ERRORS
4454: }
4455: 
4456: static PyObject* THPMseLossBackwardBackward0_reduction_getter(THPCppFunction *self, void *_unused) {
4457:   HANDLE_TH_ERRORS
4458:   auto prop = static_cast<MseLossBackwardBackward0*>(self->cdata.get())->reduction;
4459:   return PyLong_FromUnsignedLong((int64_t) prop);
4460:   END_HANDLE_TH_ERRORS
4461: }
4462: 
4463: static PyObject* THPMseLossBackwardBackward0_self_getter(THPCppFunction *self, void *_unused) {
4464:   HANDLE_TH_ERRORS
4465:   const auto& prop = static_cast<MseLossBackwardBackward0*>(self->cdata.get())->self_;
4466:   return THPVariable_Wrap(prop.unpack(self->cdata));
4467:   END_HANDLE_TH_ERRORS
4468: }
4469: 
4470: static PyObject* THPMseLossBackwardBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
4471:   HANDLE_TH_ERRORS
4472:   const auto& prop = static_cast<MseLossBackwardBackward0*>(self->cdata.get())->self_;
4473:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4474:   return obj.release().ptr();
4475:   END_HANDLE_TH_ERRORS
4476: }
4477: 
4478: static PyObject* THPMseLossBackwardBackward0_target_getter(THPCppFunction *self, void *_unused) {
4479:   HANDLE_TH_ERRORS
4480:   const auto& prop = static_cast<MseLossBackwardBackward0*>(self->cdata.get())->target_;
```

- EN: The main execution path in this span is carried by `cast`, `THPFractionalMaxPool3DBackwardBackward0_indices_getter`, `THPVariable_Wrap`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `cast`, `THPFractionalMaxPool3DBackwardBackward0_indices_getter`, `THPVariable_Wrap` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4481-4560

```cpp
4481:   return THPVariable_Wrap(prop.unpack(self->cdata));
4482:   END_HANDLE_TH_ERRORS
4483: }
4484: 
4485: static PyObject* THPMseLossBackwardBackward0_target_raw_getter(THPCppFunction *self, void *_unused) {
4486:   HANDLE_TH_ERRORS
4487:   const auto& prop = static_cast<MseLossBackwardBackward0*>(self->cdata.get())->target_;
4488:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4489:   return obj.release().ptr();
4490:   END_HANDLE_TH_ERRORS
4491: }
4492: 
4493: static struct PyGetSetDef MseLossBackwardBackward0_properties[] = {
4494:   THP_FUNCTION_DEFAULT_PROPERTIES,
4495:   {(char*)"_saved_grad_output", (getter)THPMseLossBackwardBackward0_grad_output_getter, nullptr, nullptr, nullptr},
4496:   {(char*)"_raw_saved_grad_output", (getter)THPMseLossBackwardBackward0_grad_output_raw_getter, nullptr, nullptr, nullptr},
4497:   {(char*)"_saved_reduction", (getter)THPMseLossBackwardBackward0_reduction_getter, nullptr, nullptr, nullptr},
4498:   {(char*)"_saved_self", (getter)THPMseLossBackwardBackward0_self_getter, nullptr, nullptr, nullptr},
4499:   {(char*)"_raw_saved_self", (getter)THPMseLossBackwardBackward0_self_raw_getter, nullptr, nullptr, nullptr},
4500:   {(char*)"_saved_target", (getter)THPMseLossBackwardBackward0_target_getter, nullptr, nullptr, nullptr},
4501:   {(char*)"_raw_saved_target", (getter)THPMseLossBackwardBackward0_target_raw_getter, nullptr, nullptr, nullptr},
4502:   {nullptr} /* sentinel */
4503: };
4504: 
4505: static PyObject* THPRreluWithNoiseBackwardBackward0_lower_getter(THPCppFunction *self, void *_unused) {
4506:   HANDLE_TH_ERRORS
4507:   auto prop = static_cast<RreluWithNoiseBackwardBackward0*>(self->cdata.get())->lower;
4508:   if (prop.isComplex()) {
4509:     auto cprop = prop.to<c10::complex<double>>();
4510:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
4511:   } else if (prop.isFloatingPoint()) {
4512:     return PyFloat_FromDouble(prop.to<double>());
4513:   } else if (prop.isIntegral(/*includeBool=*/false)) {
4514:     return PyLong_FromLong(prop.to<int64_t>());
4515:   } else if (prop.isBoolean()) {
4516:     if (prop.to<bool>()) {
4517:       Py_RETURN_TRUE;
4518:     } else {
4519:       Py_RETURN_FALSE;
4520:     }
4521:   } else {
4522:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
4523:     return nullptr;
4524:   }
4525:   END_HANDLE_TH_ERRORS
4526: }
4527: 
4528: static PyObject* THPRreluWithNoiseBackwardBackward0_noise_getter(THPCppFunction *self, void *_unused) {
4529:   HANDLE_TH_ERRORS
4530:   const auto& prop = static_cast<RreluWithNoiseBackwardBackward0*>(self->cdata.get())->noise_;
4531:   return THPVariable_Wrap(prop.unpack(self->cdata));
4532:   END_HANDLE_TH_ERRORS
4533: }
4534: 
4535: static PyObject* THPRreluWithNoiseBackwardBackward0_noise_raw_getter(THPCppFunction *self, void *_unused) {
4536:   HANDLE_TH_ERRORS
4537:   const auto& prop = static_cast<RreluWithNoiseBackwardBackward0*>(self->cdata.get())->noise_;
4538:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4539:   return obj.release().ptr();
4540:   END_HANDLE_TH_ERRORS
4541: }
4542: 
4543: static PyObject* THPRreluWithNoiseBackwardBackward0_self_getter(THPCppFunction *self, void *_unused) {
4544:   HANDLE_TH_ERRORS
4545:   const auto& prop = static_cast<RreluWithNoiseBackwardBackward0*>(self->cdata.get())->self_;
4546:   return THPVariable_Wrap(prop.unpack(self->cdata));
4547:   END_HANDLE_TH_ERRORS
4548: }
4549: 
4550: static PyObject* THPRreluWithNoiseBackwardBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
4551:   HANDLE_TH_ERRORS
4552:   const auto& prop = static_cast<RreluWithNoiseBackwardBackward0*>(self->cdata.get())->self_;
4553:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4554:   return obj.release().ptr();
4555:   END_HANDLE_TH_ERRORS
4556: }
4557: 
4558: static PyObject* THPRreluWithNoiseBackwardBackward0_training_getter(THPCppFunction *self, void *_unused) {
4559:   HANDLE_TH_ERRORS
4560:   auto prop = static_cast<RreluWithNoiseBackwardBackward0*>(self->cdata.get())->training;
```

- EN: The main execution path in this span is carried by `THPVariable_Wrap`, `THPMseLossBackwardBackward0_target_raw_getter`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_Wrap`, `THPMseLossBackwardBackward0_target_raw_getter`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4561-4640

```cpp
4561:   if (prop) {
4562:     Py_RETURN_TRUE;
4563:   } else {
4564:     Py_RETURN_FALSE;
4565:   }
4566:   END_HANDLE_TH_ERRORS
4567: }
4568: 
4569: static PyObject* THPRreluWithNoiseBackwardBackward0_upper_getter(THPCppFunction *self, void *_unused) {
4570:   HANDLE_TH_ERRORS
4571:   auto prop = static_cast<RreluWithNoiseBackwardBackward0*>(self->cdata.get())->upper;
4572:   if (prop.isComplex()) {
4573:     auto cprop = prop.to<c10::complex<double>>();
4574:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
4575:   } else if (prop.isFloatingPoint()) {
4576:     return PyFloat_FromDouble(prop.to<double>());
4577:   } else if (prop.isIntegral(/*includeBool=*/false)) {
4578:     return PyLong_FromLong(prop.to<int64_t>());
4579:   } else if (prop.isBoolean()) {
4580:     if (prop.to<bool>()) {
4581:       Py_RETURN_TRUE;
4582:     } else {
4583:       Py_RETURN_FALSE;
4584:     }
4585:   } else {
4586:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
4587:     return nullptr;
4588:   }
4589:   END_HANDLE_TH_ERRORS
4590: }
4591: 
4592: static struct PyGetSetDef RreluWithNoiseBackwardBackward0_properties[] = {
4593:   THP_FUNCTION_DEFAULT_PROPERTIES,
4594:   {(char*)"_saved_lower", (getter)THPRreluWithNoiseBackwardBackward0_lower_getter, nullptr, nullptr, nullptr},
4595:   {(char*)"_saved_noise", (getter)THPRreluWithNoiseBackwardBackward0_noise_getter, nullptr, nullptr, nullptr},
4596:   {(char*)"_raw_saved_noise", (getter)THPRreluWithNoiseBackwardBackward0_noise_raw_getter, nullptr, nullptr, nullptr},
4597:   {(char*)"_saved_self", (getter)THPRreluWithNoiseBackwardBackward0_self_getter, nullptr, nullptr, nullptr},
4598:   {(char*)"_raw_saved_self", (getter)THPRreluWithNoiseBackwardBackward0_self_raw_getter, nullptr, nullptr, nullptr},
4599:   {(char*)"_saved_training", (getter)THPRreluWithNoiseBackwardBackward0_training_getter, nullptr, nullptr, nullptr},
4600:   {(char*)"_saved_upper", (getter)THPRreluWithNoiseBackwardBackward0_upper_getter, nullptr, nullptr, nullptr},
4601:   {nullptr} /* sentinel */
4602: };
4603: 
4604: static PyObject* THPSoftplusBackwardBackward0_beta_getter(THPCppFunction *self, void *_unused) {
4605:   HANDLE_TH_ERRORS
4606:   auto prop = static_cast<SoftplusBackwardBackward0*>(self->cdata.get())->beta;
4607:   if (prop.isComplex()) {
4608:     auto cprop = prop.to<c10::complex<double>>();
4609:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
4610:   } else if (prop.isFloatingPoint()) {
4611:     return PyFloat_FromDouble(prop.to<double>());
4612:   } else if (prop.isIntegral(/*includeBool=*/false)) {
4613:     return PyLong_FromLong(prop.to<int64_t>());
4614:   } else if (prop.isBoolean()) {
4615:     if (prop.to<bool>()) {
4616:       Py_RETURN_TRUE;
4617:     } else {
4618:       Py_RETURN_FALSE;
4619:     }
4620:   } else {
4621:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
4622:     return nullptr;
4623:   }
4624:   END_HANDLE_TH_ERRORS
4625: }
4626: 
4627: static PyObject* THPSoftplusBackwardBackward0_grad_output_getter(THPCppFunction *self, void *_unused) {
4628:   HANDLE_TH_ERRORS
4629:   const auto& prop = static_cast<SoftplusBackwardBackward0*>(self->cdata.get())->grad_output_;
4630:   return THPVariable_Wrap(prop.unpack(self->cdata));
4631:   END_HANDLE_TH_ERRORS
4632: }
4633: 
4634: static PyObject* THPSoftplusBackwardBackward0_grad_output_raw_getter(THPCppFunction *self, void *_unused) {
4635:   HANDLE_TH_ERRORS
4636:   const auto& prop = static_cast<SoftplusBackwardBackward0*>(self->cdata.get())->grad_output_;
4637:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4638:   return obj.release().ptr();
4639:   END_HANDLE_TH_ERRORS
4640: }
```

- EN: The main execution path in this span is carried by `THPRreluWithNoiseBackwardBackward0_upper_getter`, `PyComplex_FromDoubles`, `PyFloat_FromDouble`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPRreluWithNoiseBackwardBackward0_upper_getter`, `PyComplex_FromDoubles`, `PyFloat_FromDouble` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4641-4720

```cpp
4641: 
4642: static PyObject* THPSoftplusBackwardBackward0_self_getter(THPCppFunction *self, void *_unused) {
4643:   HANDLE_TH_ERRORS
4644:   const auto& prop = static_cast<SoftplusBackwardBackward0*>(self->cdata.get())->self_;
4645:   return THPVariable_Wrap(prop.unpack(self->cdata));
4646:   END_HANDLE_TH_ERRORS
4647: }
4648: 
4649: static PyObject* THPSoftplusBackwardBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
4650:   HANDLE_TH_ERRORS
4651:   const auto& prop = static_cast<SoftplusBackwardBackward0*>(self->cdata.get())->self_;
4652:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4653:   return obj.release().ptr();
4654:   END_HANDLE_TH_ERRORS
4655: }
4656: 
4657: static PyObject* THPSoftplusBackwardBackward0_threshold_getter(THPCppFunction *self, void *_unused) {
4658:   HANDLE_TH_ERRORS
4659:   auto prop = static_cast<SoftplusBackwardBackward0*>(self->cdata.get())->threshold;
4660:   if (prop.isComplex()) {
4661:     auto cprop = prop.to<c10::complex<double>>();
4662:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
4663:   } else if (prop.isFloatingPoint()) {
4664:     return PyFloat_FromDouble(prop.to<double>());
4665:   } else if (prop.isIntegral(/*includeBool=*/false)) {
4666:     return PyLong_FromLong(prop.to<int64_t>());
4667:   } else if (prop.isBoolean()) {
4668:     if (prop.to<bool>()) {
4669:       Py_RETURN_TRUE;
4670:     } else {
4671:       Py_RETURN_FALSE;
4672:     }
4673:   } else {
4674:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
4675:     return nullptr;
4676:   }
4677:   END_HANDLE_TH_ERRORS
4678: }
4679: 
4680: static struct PyGetSetDef SoftplusBackwardBackward0_properties[] = {
4681:   THP_FUNCTION_DEFAULT_PROPERTIES,
4682:   {(char*)"_saved_beta", (getter)THPSoftplusBackwardBackward0_beta_getter, nullptr, nullptr, nullptr},
4683:   {(char*)"_saved_grad_output", (getter)THPSoftplusBackwardBackward0_grad_output_getter, nullptr, nullptr, nullptr},
4684:   {(char*)"_raw_saved_grad_output", (getter)THPSoftplusBackwardBackward0_grad_output_raw_getter, nullptr, nullptr, nullptr},
4685:   {(char*)"_saved_self", (getter)THPSoftplusBackwardBackward0_self_getter, nullptr, nullptr, nullptr},
4686:   {(char*)"_raw_saved_self", (getter)THPSoftplusBackwardBackward0_self_raw_getter, nullptr, nullptr, nullptr},
4687:   {(char*)"_saved_threshold", (getter)THPSoftplusBackwardBackward0_threshold_getter, nullptr, nullptr, nullptr},
4688:   {nullptr} /* sentinel */
4689: };
4690: 
4691: static PyObject* THPThresholdBackwardBackward0_self_getter(THPCppFunction *self, void *_unused) {
4692:   HANDLE_TH_ERRORS
4693:   const auto& prop = static_cast<ThresholdBackwardBackward0*>(self->cdata.get())->self_;
4694:   return THPVariable_Wrap(prop.unpack(self->cdata));
4695:   END_HANDLE_TH_ERRORS
4696: }
4697: 
4698: static PyObject* THPThresholdBackwardBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
4699:   HANDLE_TH_ERRORS
4700:   const auto& prop = static_cast<ThresholdBackwardBackward0*>(self->cdata.get())->self_;
4701:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4702:   return obj.release().ptr();
4703:   END_HANDLE_TH_ERRORS
4704: }
4705: 
4706: static PyObject* THPThresholdBackwardBackward0_threshold_getter(THPCppFunction *self, void *_unused) {
4707:   HANDLE_TH_ERRORS
4708:   auto prop = static_cast<ThresholdBackwardBackward0*>(self->cdata.get())->threshold;
4709:   if (prop.isComplex()) {
4710:     auto cprop = prop.to<c10::complex<double>>();
4711:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
4712:   } else if (prop.isFloatingPoint()) {
4713:     return PyFloat_FromDouble(prop.to<double>());
4714:   } else if (prop.isIntegral(/*includeBool=*/false)) {
4715:     return PyLong_FromLong(prop.to<int64_t>());
4716:   } else if (prop.isBoolean()) {
4717:     if (prop.to<bool>()) {
4718:       Py_RETURN_TRUE;
4719:     } else {
4720:       Py_RETURN_FALSE;
```

- EN: The main execution path in this span is carried by `THPSoftplusBackwardBackward0_self_getter`, `THPVariable_Wrap`, `THPSoftplusBackwardBackward0_self_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPSoftplusBackwardBackward0_self_getter`, `THPVariable_Wrap`, `THPSoftplusBackwardBackward0_self_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4721-4800

```cpp
4721:     }
4722:   } else {
4723:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
4724:     return nullptr;
4725:   }
4726:   END_HANDLE_TH_ERRORS
4727: }
4728: 
4729: static struct PyGetSetDef ThresholdBackwardBackward0_properties[] = {
4730:   THP_FUNCTION_DEFAULT_PROPERTIES,
4731:   {(char*)"_saved_self", (getter)THPThresholdBackwardBackward0_self_getter, nullptr, nullptr, nullptr},
4732:   {(char*)"_raw_saved_self", (getter)THPThresholdBackwardBackward0_self_raw_getter, nullptr, nullptr, nullptr},
4733:   {(char*)"_saved_threshold", (getter)THPThresholdBackwardBackward0_threshold_getter, nullptr, nullptr, nullptr},
4734:   {nullptr} /* sentinel */
4735: };
4736: 
4737: static PyObject* THPUpsampleBilinear2DAaBackwardBackward0_align_corners_getter(THPCppFunction *self, void *_unused) {
4738:   HANDLE_TH_ERRORS
4739:   auto prop = static_cast<UpsampleBilinear2DAaBackwardBackward0*>(self->cdata.get())->align_corners;
4740:   if (prop) {
4741:     Py_RETURN_TRUE;
4742:   } else {
4743:     Py_RETURN_FALSE;
4744:   }
4745:   END_HANDLE_TH_ERRORS
4746: }
4747: 
4748: static PyObject* THPUpsampleBilinear2DAaBackwardBackward0_output_size_getter(THPCppFunction *self, void *_unused) {
4749:   HANDLE_TH_ERRORS
4750:   auto prop = static_cast<UpsampleBilinear2DAaBackwardBackward0*>(self->cdata.get())->output_size;
4751:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4752:   for (auto i : c10::irange(prop.size())) {
4753:       auto si = prop[i];
4754:       if (auto m = si.maybe_as_int()) {
4755:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4756:       } else {
4757:         auto py_symint = py::cast(si).release().ptr();
4758:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4759:       }
4760:   }
4761:   return tup;
4762:   END_HANDLE_TH_ERRORS
4763: }
4764: 
4765: static PyObject* THPUpsampleBilinear2DAaBackwardBackward0_scales_h_getter(THPCppFunction *self, void *_unused) {
4766:   HANDLE_TH_ERRORS
4767:   auto opt_prop = static_cast<UpsampleBilinear2DAaBackwardBackward0*>(self->cdata.get())->scales_h;
4768:   if (!opt_prop.has_value()) {
4769:     Py_RETURN_NONE;
4770:   }
4771:   auto prop = opt_prop.value();
4772:   return PyFloat_FromDouble((double) prop);
4773:   END_HANDLE_TH_ERRORS
4774: }
4775: 
4776: static PyObject* THPUpsampleBilinear2DAaBackwardBackward0_scales_w_getter(THPCppFunction *self, void *_unused) {
4777:   HANDLE_TH_ERRORS
4778:   auto opt_prop = static_cast<UpsampleBilinear2DAaBackwardBackward0*>(self->cdata.get())->scales_w;
4779:   if (!opt_prop.has_value()) {
4780:     Py_RETURN_NONE;
4781:   }
4782:   auto prop = opt_prop.value();
4783:   return PyFloat_FromDouble((double) prop);
4784:   END_HANDLE_TH_ERRORS
4785: }
4786: 
4787: static struct PyGetSetDef UpsampleBilinear2DAaBackwardBackward0_properties[] = {
4788:   THP_FUNCTION_DEFAULT_PROPERTIES,
4789:   {(char*)"_saved_align_corners", (getter)THPUpsampleBilinear2DAaBackwardBackward0_align_corners_getter, nullptr, nullptr, nullptr},
4790:   {(char*)"_saved_output_size", (getter)THPUpsampleBilinear2DAaBackwardBackward0_output_size_getter, nullptr, nullptr, nullptr},
4791:   {(char*)"_saved_scales_h", (getter)THPUpsampleBilinear2DAaBackwardBackward0_scales_h_getter, nullptr, nullptr, nullptr},
4792:   {(char*)"_saved_scales_w", (getter)THPUpsampleBilinear2DAaBackwardBackward0_scales_w_getter, nullptr, nullptr, nullptr},
4793:   {nullptr} /* sentinel */
4794: };
4795: 
4796: static PyObject* THPSigmoidBackwardBackward0_grad_output_getter(THPCppFunction *self, void *_unused) {
4797:   HANDLE_TH_ERRORS
4798:   const auto& prop = static_cast<SigmoidBackwardBackward0*>(self->cdata.get())->grad_output_;
4799:   return THPVariable_Wrap(prop.unpack(self->cdata));
4800:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `PyErr_SetString`, `THPUpsampleBilinear2DAaBackwardBackward0_align_corners_getter`, `THPUpsampleBilinear2DAaBackwardBackward0_output_size_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyErr_SetString`, `THPUpsampleBilinear2DAaBackwardBackward0_align_corners_getter`, `THPUpsampleBilinear2DAaBackwardBackward0_output_size_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4801-4880

```cpp
4801: }
4802: 
4803: static PyObject* THPSigmoidBackwardBackward0_grad_output_raw_getter(THPCppFunction *self, void *_unused) {
4804:   HANDLE_TH_ERRORS
4805:   const auto& prop = static_cast<SigmoidBackwardBackward0*>(self->cdata.get())->grad_output_;
4806:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4807:   return obj.release().ptr();
4808:   END_HANDLE_TH_ERRORS
4809: }
4810: 
4811: static PyObject* THPSigmoidBackwardBackward0_output_getter(THPCppFunction *self, void *_unused) {
4812:   HANDLE_TH_ERRORS
4813:   const auto& prop = static_cast<SigmoidBackwardBackward0*>(self->cdata.get())->output_;
4814:   return THPVariable_Wrap(prop.unpack(self->cdata));
4815:   END_HANDLE_TH_ERRORS
4816: }
4817: 
4818: static PyObject* THPSigmoidBackwardBackward0_output_raw_getter(THPCppFunction *self, void *_unused) {
4819:   HANDLE_TH_ERRORS
4820:   const auto& prop = static_cast<SigmoidBackwardBackward0*>(self->cdata.get())->output_;
4821:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4822:   return obj.release().ptr();
4823:   END_HANDLE_TH_ERRORS
4824: }
4825: 
4826: static struct PyGetSetDef SigmoidBackwardBackward0_properties[] = {
4827:   THP_FUNCTION_DEFAULT_PROPERTIES,
4828:   {(char*)"_saved_grad_output", (getter)THPSigmoidBackwardBackward0_grad_output_getter, nullptr, nullptr, nullptr},
4829:   {(char*)"_raw_saved_grad_output", (getter)THPSigmoidBackwardBackward0_grad_output_raw_getter, nullptr, nullptr, nullptr},
4830:   {(char*)"_saved_output", (getter)THPSigmoidBackwardBackward0_output_getter, nullptr, nullptr, nullptr},
4831:   {(char*)"_raw_saved_output", (getter)THPSigmoidBackwardBackward0_output_raw_getter, nullptr, nullptr, nullptr},
4832:   {nullptr} /* sentinel */
4833: };
4834: 
4835: static PyObject* THPTanhBackwardBackward0_grad_output_getter(THPCppFunction *self, void *_unused) {
4836:   HANDLE_TH_ERRORS
4837:   const auto& prop = static_cast<TanhBackwardBackward0*>(self->cdata.get())->grad_output_;
4838:   return THPVariable_Wrap(prop.unpack(self->cdata));
4839:   END_HANDLE_TH_ERRORS
4840: }
4841: 
4842: static PyObject* THPTanhBackwardBackward0_grad_output_raw_getter(THPCppFunction *self, void *_unused) {
4843:   HANDLE_TH_ERRORS
4844:   const auto& prop = static_cast<TanhBackwardBackward0*>(self->cdata.get())->grad_output_;
4845:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4846:   return obj.release().ptr();
4847:   END_HANDLE_TH_ERRORS
4848: }
4849: 
4850: static PyObject* THPTanhBackwardBackward0_output_getter(THPCppFunction *self, void *_unused) {
4851:   HANDLE_TH_ERRORS
4852:   const auto& prop = static_cast<TanhBackwardBackward0*>(self->cdata.get())->output_;
4853:   return THPVariable_Wrap(prop.unpack(self->cdata));
4854:   END_HANDLE_TH_ERRORS
4855: }
4856: 
4857: static PyObject* THPTanhBackwardBackward0_output_raw_getter(THPCppFunction *self, void *_unused) {
4858:   HANDLE_TH_ERRORS
4859:   const auto& prop = static_cast<TanhBackwardBackward0*>(self->cdata.get())->output_;
4860:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4861:   return obj.release().ptr();
4862:   END_HANDLE_TH_ERRORS
4863: }
4864: 
4865: static struct PyGetSetDef TanhBackwardBackward0_properties[] = {
4866:   THP_FUNCTION_DEFAULT_PROPERTIES,
4867:   {(char*)"_saved_grad_output", (getter)THPTanhBackwardBackward0_grad_output_getter, nullptr, nullptr, nullptr},
4868:   {(char*)"_raw_saved_grad_output", (getter)THPTanhBackwardBackward0_grad_output_raw_getter, nullptr, nullptr, nullptr},
4869:   {(char*)"_saved_output", (getter)THPTanhBackwardBackward0_output_getter, nullptr, nullptr, nullptr},
4870:   {(char*)"_raw_saved_output", (getter)THPTanhBackwardBackward0_output_raw_getter, nullptr, nullptr, nullptr},
4871:   {nullptr} /* sentinel */
4872: };
4873: 
4874: static PyObject* THPCudnnCtcLossBackward0_zero_infinity_getter(THPCppFunction *self, void *_unused) {
4875:   HANDLE_TH_ERRORS
4876:   auto prop = static_cast<CudnnCtcLossBackward0*>(self->cdata.get())->zero_infinity;
4877:   if (prop) {
4878:     Py_RETURN_TRUE;
4879:   } else {
4880:     Py_RETURN_FALSE;
```

- EN: The main execution path in this span is carried by `THPSigmoidBackwardBackward0_grad_output_raw_getter`, `cast`, `THPSigmoidBackwardBackward0_output_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPSigmoidBackwardBackward0_grad_output_raw_getter`, `cast`, `THPSigmoidBackwardBackward0_output_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4881-4960

```cpp
4881:   }
4882:   END_HANDLE_TH_ERRORS
4883: }
4884: 
4885: static PyObject* THPCudnnCtcLossBackward0_result0_getter(THPCppFunction *self, void *_unused) {
4886:   HANDLE_TH_ERRORS
4887:   const auto& prop = static_cast<CudnnCtcLossBackward0*>(self->cdata.get())->result0_;
4888:   return THPVariable_Wrap(prop.unpack(self->cdata));
4889:   END_HANDLE_TH_ERRORS
4890: }
4891: 
4892: static PyObject* THPCudnnCtcLossBackward0_result0_raw_getter(THPCppFunction *self, void *_unused) {
4893:   HANDLE_TH_ERRORS
4894:   const auto& prop = static_cast<CudnnCtcLossBackward0*>(self->cdata.get())->result0_;
4895:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4896:   return obj.release().ptr();
4897:   END_HANDLE_TH_ERRORS
4898: }
4899: 
4900: static PyObject* THPCudnnCtcLossBackward0_result1_getter(THPCppFunction *self, void *_unused) {
4901:   HANDLE_TH_ERRORS
4902:   const auto& prop = static_cast<CudnnCtcLossBackward0*>(self->cdata.get())->result1_;
4903:   return THPVariable_Wrap(prop.unpack(self->cdata));
4904:   END_HANDLE_TH_ERRORS
4905: }
4906: 
4907: static PyObject* THPCudnnCtcLossBackward0_result1_raw_getter(THPCppFunction *self, void *_unused) {
4908:   HANDLE_TH_ERRORS
4909:   const auto& prop = static_cast<CudnnCtcLossBackward0*>(self->cdata.get())->result1_;
4910:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4911:   return obj.release().ptr();
4912:   END_HANDLE_TH_ERRORS
4913: }
4914: 
4915: static struct PyGetSetDef CudnnCtcLossBackward0_properties[] = {
4916:   THP_FUNCTION_DEFAULT_PROPERTIES,
4917:   {(char*)"_saved_zero_infinity", (getter)THPCudnnCtcLossBackward0_zero_infinity_getter, nullptr, nullptr, nullptr},
4918:   {(char*)"_saved_result0", (getter)THPCudnnCtcLossBackward0_result0_getter, nullptr, nullptr, nullptr},
4919:   {(char*)"_raw_saved_result0", (getter)THPCudnnCtcLossBackward0_result0_raw_getter, nullptr, nullptr, nullptr},
4920:   {(char*)"_saved_result1", (getter)THPCudnnCtcLossBackward0_result1_getter, nullptr, nullptr, nullptr},
4921:   {(char*)"_raw_saved_result1", (getter)THPCudnnCtcLossBackward0_result1_raw_getter, nullptr, nullptr, nullptr},
4922:   {nullptr} /* sentinel */
4923: };
4924: 
4925: static PyObject* THPCudnnCtcLossBackward1_zero_infinity_getter(THPCppFunction *self, void *_unused) {
4926:   HANDLE_TH_ERRORS
4927:   auto prop = static_cast<CudnnCtcLossBackward1*>(self->cdata.get())->zero_infinity;
4928:   if (prop) {
4929:     Py_RETURN_TRUE;
4930:   } else {
4931:     Py_RETURN_FALSE;
4932:   }
4933:   END_HANDLE_TH_ERRORS
4934: }
4935: 
4936: static PyObject* THPCudnnCtcLossBackward1_result0_getter(THPCppFunction *self, void *_unused) {
4937:   HANDLE_TH_ERRORS
4938:   const auto& prop = static_cast<CudnnCtcLossBackward1*>(self->cdata.get())->result0_;
4939:   return THPVariable_Wrap(prop.unpack(self->cdata));
4940:   END_HANDLE_TH_ERRORS
4941: }
4942: 
4943: static PyObject* THPCudnnCtcLossBackward1_result0_raw_getter(THPCppFunction *self, void *_unused) {
4944:   HANDLE_TH_ERRORS
4945:   const auto& prop = static_cast<CudnnCtcLossBackward1*>(self->cdata.get())->result0_;
4946:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4947:   return obj.release().ptr();
4948:   END_HANDLE_TH_ERRORS
4949: }
4950: 
4951: static PyObject* THPCudnnCtcLossBackward1_result1_getter(THPCppFunction *self, void *_unused) {
4952:   HANDLE_TH_ERRORS
4953:   const auto& prop = static_cast<CudnnCtcLossBackward1*>(self->cdata.get())->result1_;
4954:   return THPVariable_Wrap(prop.unpack(self->cdata));
4955:   END_HANDLE_TH_ERRORS
4956: }
4957: 
4958: static PyObject* THPCudnnCtcLossBackward1_result1_raw_getter(THPCppFunction *self, void *_unused) {
4959:   HANDLE_TH_ERRORS
4960:   const auto& prop = static_cast<CudnnCtcLossBackward1*>(self->cdata.get())->result1_;
```

- EN: The main execution path in this span is carried by `THPCudnnCtcLossBackward0_result0_getter`, `THPVariable_Wrap`, `THPCudnnCtcLossBackward0_result0_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPCudnnCtcLossBackward0_result0_getter`, `THPVariable_Wrap`, `THPCudnnCtcLossBackward0_result0_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4961-5040

```cpp
4961:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4962:   return obj.release().ptr();
4963:   END_HANDLE_TH_ERRORS
4964: }
4965: 
4966: static struct PyGetSetDef CudnnCtcLossBackward1_properties[] = {
4967:   THP_FUNCTION_DEFAULT_PROPERTIES,
4968:   {(char*)"_saved_zero_infinity", (getter)THPCudnnCtcLossBackward1_zero_infinity_getter, nullptr, nullptr, nullptr},
4969:   {(char*)"_saved_result0", (getter)THPCudnnCtcLossBackward1_result0_getter, nullptr, nullptr, nullptr},
4970:   {(char*)"_raw_saved_result0", (getter)THPCudnnCtcLossBackward1_result0_raw_getter, nullptr, nullptr, nullptr},
4971:   {(char*)"_saved_result1", (getter)THPCudnnCtcLossBackward1_result1_getter, nullptr, nullptr, nullptr},
4972:   {(char*)"_raw_saved_result1", (getter)THPCudnnCtcLossBackward1_result1_raw_getter, nullptr, nullptr, nullptr},
4973:   {nullptr} /* sentinel */
4974: };
4975: 
4976: static PyObject* THPCudnnConvolutionTransposeBackward0_dilation_getter(THPCppFunction *self, void *_unused) {
4977:   HANDLE_TH_ERRORS
4978:   auto prop = static_cast<CudnnConvolutionTransposeBackward0*>(self->cdata.get())->dilation;
4979:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4980:   for (auto i : c10::irange(prop.size())) {
4981:       auto si = prop[i];
4982:       if (auto m = si.maybe_as_int()) {
4983:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4984:       } else {
4985:         auto py_symint = py::cast(si).release().ptr();
4986:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4987:       }
4988:   }
4989:   return tup;
4990:   END_HANDLE_TH_ERRORS
4991: }
4992: 
4993: static PyObject* THPCudnnConvolutionTransposeBackward0_groups_getter(THPCppFunction *self, void *_unused) {
4994:   HANDLE_TH_ERRORS
4995:   auto prop = static_cast<CudnnConvolutionTransposeBackward0*>(self->cdata.get())->groups;
4996:   if (auto m = prop.maybe_as_int()) {
4997:     return PyLong_FromUnsignedLong(*m);
4998:   } else {
4999:     return py::cast(prop).release().ptr();
5000:   }
5001:   END_HANDLE_TH_ERRORS
5002: }
5003: 
5004: static PyObject* THPCudnnConvolutionTransposeBackward0_output_padding_getter(THPCppFunction *self, void *_unused) {
5005:   HANDLE_TH_ERRORS
5006:   auto prop = static_cast<CudnnConvolutionTransposeBackward0*>(self->cdata.get())->output_padding;
5007:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5008:   for (auto i : c10::irange(prop.size())) {
5009:       auto si = prop[i];
5010:       if (auto m = si.maybe_as_int()) {
5011:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
5012:       } else {
5013:         auto py_symint = py::cast(si).release().ptr();
5014:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
5015:       }
5016:   }
5017:   return tup;
5018:   END_HANDLE_TH_ERRORS
5019: }
5020: 
5021: static PyObject* THPCudnnConvolutionTransposeBackward0_padding_getter(THPCppFunction *self, void *_unused) {
5022:   HANDLE_TH_ERRORS
5023:   auto prop = static_cast<CudnnConvolutionTransposeBackward0*>(self->cdata.get())->padding;
5024:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5025:   for (auto i : c10::irange(prop.size())) {
5026:       auto si = prop[i];
5027:       if (auto m = si.maybe_as_int()) {
5028:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
5029:       } else {
5030:         auto py_symint = py::cast(si).release().ptr();
5031:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
5032:       }
5033:   }
5034:   return tup;
5035:   END_HANDLE_TH_ERRORS
5036: }
5037: 
5038: static PyObject* THPCudnnConvolutionTransposeBackward0_self_getter(THPCppFunction *self, void *_unused) {
5039:   HANDLE_TH_ERRORS
5040:   const auto& prop = static_cast<CudnnConvolutionTransposeBackward0*>(self->cdata.get())->self_;
```

- EN: The main execution path in this span is carried by `cast`, `THPCudnnConvolutionTransposeBackward0_dilation_getter`, `PyTuple_New`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `cast`, `THPCudnnConvolutionTransposeBackward0_dilation_getter`, `PyTuple_New` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5041-5120

```cpp
5041:   return THPVariable_Wrap(prop.unpack(self->cdata));
5042:   END_HANDLE_TH_ERRORS
5043: }
5044: 
5045: static PyObject* THPCudnnConvolutionTransposeBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
5046:   HANDLE_TH_ERRORS
5047:   const auto& prop = static_cast<CudnnConvolutionTransposeBackward0*>(self->cdata.get())->self_;
5048:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5049:   return obj.release().ptr();
5050:   END_HANDLE_TH_ERRORS
5051: }
5052: 
5053: static PyObject* THPCudnnConvolutionTransposeBackward0_stride_getter(THPCppFunction *self, void *_unused) {
5054:   HANDLE_TH_ERRORS
5055:   auto prop = static_cast<CudnnConvolutionTransposeBackward0*>(self->cdata.get())->stride;
5056:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5057:   for (auto i : c10::irange(prop.size())) {
5058:       auto si = prop[i];
5059:       if (auto m = si.maybe_as_int()) {
5060:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
5061:       } else {
5062:         auto py_symint = py::cast(si).release().ptr();
5063:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
5064:       }
5065:   }
5066:   return tup;
5067:   END_HANDLE_TH_ERRORS
5068: }
5069: 
5070: static PyObject* THPCudnnConvolutionTransposeBackward0_weight_getter(THPCppFunction *self, void *_unused) {
5071:   HANDLE_TH_ERRORS
5072:   const auto& prop = static_cast<CudnnConvolutionTransposeBackward0*>(self->cdata.get())->weight_;
5073:   return THPVariable_Wrap(prop.unpack(self->cdata));
5074:   END_HANDLE_TH_ERRORS
5075: }
5076: 
5077: static PyObject* THPCudnnConvolutionTransposeBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
5078:   HANDLE_TH_ERRORS
5079:   const auto& prop = static_cast<CudnnConvolutionTransposeBackward0*>(self->cdata.get())->weight_;
5080:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5081:   return obj.release().ptr();
5082:   END_HANDLE_TH_ERRORS
5083: }
5084: 
5085: static struct PyGetSetDef CudnnConvolutionTransposeBackward0_properties[] = {
5086:   THP_FUNCTION_DEFAULT_PROPERTIES,
5087:   {(char*)"_saved_dilation", (getter)THPCudnnConvolutionTransposeBackward0_dilation_getter, nullptr, nullptr, nullptr},
5088:   {(char*)"_saved_groups", (getter)THPCudnnConvolutionTransposeBackward0_groups_getter, nullptr, nullptr, nullptr},
5089:   {(char*)"_saved_output_padding", (getter)THPCudnnConvolutionTransposeBackward0_output_padding_getter, nullptr, nullptr, nullptr},
5090:   {(char*)"_saved_padding", (getter)THPCudnnConvolutionTransposeBackward0_padding_getter, nullptr, nullptr, nullptr},
5091:   {(char*)"_saved_self", (getter)THPCudnnConvolutionTransposeBackward0_self_getter, nullptr, nullptr, nullptr},
5092:   {(char*)"_raw_saved_self", (getter)THPCudnnConvolutionTransposeBackward0_self_raw_getter, nullptr, nullptr, nullptr},
5093:   {(char*)"_saved_stride", (getter)THPCudnnConvolutionTransposeBackward0_stride_getter, nullptr, nullptr, nullptr},
5094:   {(char*)"_saved_weight", (getter)THPCudnnConvolutionTransposeBackward0_weight_getter, nullptr, nullptr, nullptr},
5095:   {(char*)"_raw_saved_weight", (getter)THPCudnnConvolutionTransposeBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
5096:   {nullptr} /* sentinel */
5097: };
5098: 
5099: static PyObject* THPCudnnGridSamplerBackwardBackward0_grad_output_getter(THPCppFunction *self, void *_unused) {
5100:   HANDLE_TH_ERRORS
5101:   const auto& prop = static_cast<CudnnGridSamplerBackwardBackward0*>(self->cdata.get())->grad_output_;
5102:   return THPVariable_Wrap(prop.unpack(self->cdata));
5103:   END_HANDLE_TH_ERRORS
5104: }
5105: 
5106: static PyObject* THPCudnnGridSamplerBackwardBackward0_grad_output_raw_getter(THPCppFunction *self, void *_unused) {
5107:   HANDLE_TH_ERRORS
5108:   const auto& prop = static_cast<CudnnGridSamplerBackwardBackward0*>(self->cdata.get())->grad_output_;
5109:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5110:   return obj.release().ptr();
5111:   END_HANDLE_TH_ERRORS
5112: }
5113: 
5114: static PyObject* THPCudnnGridSamplerBackwardBackward0_grid_getter(THPCppFunction *self, void *_unused) {
5115:   HANDLE_TH_ERRORS
5116:   const auto& prop = static_cast<CudnnGridSamplerBackwardBackward0*>(self->cdata.get())->grid_;
5117:   return THPVariable_Wrap(prop.unpack(self->cdata));
5118:   END_HANDLE_TH_ERRORS
5119: }
5120: 
```

- EN: The main execution path in this span is carried by `THPVariable_Wrap`, `THPCudnnConvolutionTransposeBackward0_self_raw_getter`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_Wrap`, `THPCudnnConvolutionTransposeBackward0_self_raw_getter`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5121-5200

```cpp
5121: static PyObject* THPCudnnGridSamplerBackwardBackward0_grid_raw_getter(THPCppFunction *self, void *_unused) {
5122:   HANDLE_TH_ERRORS
5123:   const auto& prop = static_cast<CudnnGridSamplerBackwardBackward0*>(self->cdata.get())->grid_;
5124:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5125:   return obj.release().ptr();
5126:   END_HANDLE_TH_ERRORS
5127: }
5128: 
5129: static PyObject* THPCudnnGridSamplerBackwardBackward0_self_getter(THPCppFunction *self, void *_unused) {
5130:   HANDLE_TH_ERRORS
5131:   const auto& prop = static_cast<CudnnGridSamplerBackwardBackward0*>(self->cdata.get())->self_;
5132:   return THPVariable_Wrap(prop.unpack(self->cdata));
5133:   END_HANDLE_TH_ERRORS
5134: }
5135: 
5136: static PyObject* THPCudnnGridSamplerBackwardBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
5137:   HANDLE_TH_ERRORS
5138:   const auto& prop = static_cast<CudnnGridSamplerBackwardBackward0*>(self->cdata.get())->self_;
5139:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5140:   return obj.release().ptr();
5141:   END_HANDLE_TH_ERRORS
5142: }
5143: 
5144: static struct PyGetSetDef CudnnGridSamplerBackwardBackward0_properties[] = {
5145:   THP_FUNCTION_DEFAULT_PROPERTIES,
5146:   {(char*)"_saved_grad_output", (getter)THPCudnnGridSamplerBackwardBackward0_grad_output_getter, nullptr, nullptr, nullptr},
5147:   {(char*)"_raw_saved_grad_output", (getter)THPCudnnGridSamplerBackwardBackward0_grad_output_raw_getter, nullptr, nullptr, nullptr},
5148:   {(char*)"_saved_grid", (getter)THPCudnnGridSamplerBackwardBackward0_grid_getter, nullptr, nullptr, nullptr},
5149:   {(char*)"_raw_saved_grid", (getter)THPCudnnGridSamplerBackwardBackward0_grid_raw_getter, nullptr, nullptr, nullptr},
5150:   {(char*)"_saved_self", (getter)THPCudnnGridSamplerBackwardBackward0_self_getter, nullptr, nullptr, nullptr},
5151:   {(char*)"_raw_saved_self", (getter)THPCudnnGridSamplerBackwardBackward0_self_raw_getter, nullptr, nullptr, nullptr},
5152:   {nullptr} /* sentinel */
5153: };
5154: 
5155: static PyObject* THPCudnnAffineGridGeneratorBackward0_C_getter(THPCppFunction *self, void *_unused) {
5156:   HANDLE_TH_ERRORS
5157:   auto prop = static_cast<CudnnAffineGridGeneratorBackward0*>(self->cdata.get())->C;
5158:   return PyLong_FromUnsignedLong((int64_t) prop);
5159:   END_HANDLE_TH_ERRORS
5160: }
5161: 
5162: static PyObject* THPCudnnAffineGridGeneratorBackward0_H_getter(THPCppFunction *self, void *_unused) {
5163:   HANDLE_TH_ERRORS
5164:   auto prop = static_cast<CudnnAffineGridGeneratorBackward0*>(self->cdata.get())->H;
5165:   return PyLong_FromUnsignedLong((int64_t) prop);
5166:   END_HANDLE_TH_ERRORS
5167: }
5168: 
5169: static PyObject* THPCudnnAffineGridGeneratorBackward0_N_getter(THPCppFunction *self, void *_unused) {
5170:   HANDLE_TH_ERRORS
5171:   auto prop = static_cast<CudnnAffineGridGeneratorBackward0*>(self->cdata.get())->N;
5172:   return PyLong_FromUnsignedLong((int64_t) prop);
5173:   END_HANDLE_TH_ERRORS
5174: }
5175: 
5176: static PyObject* THPCudnnAffineGridGeneratorBackward0_W_getter(THPCppFunction *self, void *_unused) {
5177:   HANDLE_TH_ERRORS
5178:   auto prop = static_cast<CudnnAffineGridGeneratorBackward0*>(self->cdata.get())->W;
5179:   return PyLong_FromUnsignedLong((int64_t) prop);
5180:   END_HANDLE_TH_ERRORS
5181: }
5182: 
5183: static struct PyGetSetDef CudnnAffineGridGeneratorBackward0_properties[] = {
5184:   THP_FUNCTION_DEFAULT_PROPERTIES,
5185:   {(char*)"_saved_C", (getter)THPCudnnAffineGridGeneratorBackward0_C_getter, nullptr, nullptr, nullptr},
5186:   {(char*)"_saved_H", (getter)THPCudnnAffineGridGeneratorBackward0_H_getter, nullptr, nullptr, nullptr},
5187:   {(char*)"_saved_N", (getter)THPCudnnAffineGridGeneratorBackward0_N_getter, nullptr, nullptr, nullptr},
5188:   {(char*)"_saved_W", (getter)THPCudnnAffineGridGeneratorBackward0_W_getter, nullptr, nullptr, nullptr},
5189:   {nullptr} /* sentinel */
5190: };
5191: 
5192: static PyObject* THPCudnnBatchNormBackwardBackward0_epsilon_getter(THPCppFunction *self, void *_unused) {
5193:   HANDLE_TH_ERRORS
5194:   auto prop = static_cast<CudnnBatchNormBackwardBackward0*>(self->cdata.get())->epsilon;
5195:   return PyFloat_FromDouble((double) prop);
5196:   END_HANDLE_TH_ERRORS
5197: }
5198: 
5199: static PyObject* THPCudnnBatchNormBackwardBackward0_grad_output_getter(THPCppFunction *self, void *_unused) {
5200:   HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPCudnnGridSamplerBackwardBackward0_grid_raw_getter`, `cast`, `THPCudnnGridSamplerBackwardBackward0_self_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPCudnnGridSamplerBackwardBackward0_grid_raw_getter`, `cast`, `THPCudnnGridSamplerBackwardBackward0_self_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5201-5280

```cpp
5201:   const auto& prop = static_cast<CudnnBatchNormBackwardBackward0*>(self->cdata.get())->grad_output_;
5202:   return THPVariable_Wrap(prop.unpack(self->cdata));
5203:   END_HANDLE_TH_ERRORS
5204: }
5205: 
5206: static PyObject* THPCudnnBatchNormBackwardBackward0_grad_output_raw_getter(THPCppFunction *self, void *_unused) {
5207:   HANDLE_TH_ERRORS
5208:   const auto& prop = static_cast<CudnnBatchNormBackwardBackward0*>(self->cdata.get())->grad_output_;
5209:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5210:   return obj.release().ptr();
5211:   END_HANDLE_TH_ERRORS
5212: }
5213: 
5214: static PyObject* THPCudnnBatchNormBackwardBackward0_input_getter(THPCppFunction *self, void *_unused) {
5215:   HANDLE_TH_ERRORS
5216:   const auto& prop = static_cast<CudnnBatchNormBackwardBackward0*>(self->cdata.get())->input_;
5217:   return THPVariable_Wrap(prop.unpack(self->cdata));
5218:   END_HANDLE_TH_ERRORS
5219: }
5220: 
5221: static PyObject* THPCudnnBatchNormBackwardBackward0_input_raw_getter(THPCppFunction *self, void *_unused) {
5222:   HANDLE_TH_ERRORS
5223:   const auto& prop = static_cast<CudnnBatchNormBackwardBackward0*>(self->cdata.get())->input_;
5224:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5225:   return obj.release().ptr();
5226:   END_HANDLE_TH_ERRORS
5227: }
5228: 
5229: static PyObject* THPCudnnBatchNormBackwardBackward0_reserveSpace_getter(THPCppFunction *self, void *_unused) {
5230:   HANDLE_TH_ERRORS
5231:   const auto& prop = static_cast<CudnnBatchNormBackwardBackward0*>(self->cdata.get())->reserveSpace_;
5232:   return THPVariable_Wrap(prop.unpack(self->cdata));
5233:   END_HANDLE_TH_ERRORS
5234: }
5235: 
5236: static PyObject* THPCudnnBatchNormBackwardBackward0_reserveSpace_raw_getter(THPCppFunction *self, void *_unused) {
5237:   HANDLE_TH_ERRORS
5238:   const auto& prop = static_cast<CudnnBatchNormBackwardBackward0*>(self->cdata.get())->reserveSpace_;
5239:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5240:   return obj.release().ptr();
5241:   END_HANDLE_TH_ERRORS
5242: }
5243: 
5244: static PyObject* THPCudnnBatchNormBackwardBackward0_running_mean_getter(THPCppFunction *self, void *_unused) {
5245:   HANDLE_TH_ERRORS
5246:   const auto& prop = static_cast<CudnnBatchNormBackwardBackward0*>(self->cdata.get())->running_mean_;
5247:   return THPVariable_Wrap(prop.unpack(self->cdata));
5248:   END_HANDLE_TH_ERRORS
5249: }
5250: 
5251: static PyObject* THPCudnnBatchNormBackwardBackward0_running_mean_raw_getter(THPCppFunction *self, void *_unused) {
5252:   HANDLE_TH_ERRORS
5253:   const auto& prop = static_cast<CudnnBatchNormBackwardBackward0*>(self->cdata.get())->running_mean_;
5254:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5255:   return obj.release().ptr();
5256:   END_HANDLE_TH_ERRORS
5257: }
5258: 
5259: static PyObject* THPCudnnBatchNormBackwardBackward0_running_var_getter(THPCppFunction *self, void *_unused) {
5260:   HANDLE_TH_ERRORS
5261:   const auto& prop = static_cast<CudnnBatchNormBackwardBackward0*>(self->cdata.get())->running_var_;
5262:   return THPVariable_Wrap(prop.unpack(self->cdata));
5263:   END_HANDLE_TH_ERRORS
5264: }
5265: 
5266: static PyObject* THPCudnnBatchNormBackwardBackward0_running_var_raw_getter(THPCppFunction *self, void *_unused) {
5267:   HANDLE_TH_ERRORS
5268:   const auto& prop = static_cast<CudnnBatchNormBackwardBackward0*>(self->cdata.get())->running_var_;
5269:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5270:   return obj.release().ptr();
5271:   END_HANDLE_TH_ERRORS
5272: }
5273: 
5274: static PyObject* THPCudnnBatchNormBackwardBackward0_save_mean_getter(THPCppFunction *self, void *_unused) {
5275:   HANDLE_TH_ERRORS
5276:   const auto& prop = static_cast<CudnnBatchNormBackwardBackward0*>(self->cdata.get())->save_mean_;
5277:   return THPVariable_Wrap(prop.unpack(self->cdata));
5278:   END_HANDLE_TH_ERRORS
5279: }
5280: 
```

- EN: The main execution path in this span is carried by `THPVariable_Wrap`, `THPCudnnBatchNormBackwardBackward0_grad_output_raw_getter`, `cast`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_Wrap`, `THPCudnnBatchNormBackwardBackward0_grad_output_raw_getter`, `cast` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5281-5360

```cpp
5281: static PyObject* THPCudnnBatchNormBackwardBackward0_save_mean_raw_getter(THPCppFunction *self, void *_unused) {
5282:   HANDLE_TH_ERRORS
5283:   const auto& prop = static_cast<CudnnBatchNormBackwardBackward0*>(self->cdata.get())->save_mean_;
5284:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5285:   return obj.release().ptr();
5286:   END_HANDLE_TH_ERRORS
5287: }
5288: 
5289: static PyObject* THPCudnnBatchNormBackwardBackward0_save_var_getter(THPCppFunction *self, void *_unused) {
5290:   HANDLE_TH_ERRORS
5291:   const auto& prop = static_cast<CudnnBatchNormBackwardBackward0*>(self->cdata.get())->save_var_;
5292:   return THPVariable_Wrap(prop.unpack(self->cdata));
5293:   END_HANDLE_TH_ERRORS
5294: }
5295: 
5296: static PyObject* THPCudnnBatchNormBackwardBackward0_save_var_raw_getter(THPCppFunction *self, void *_unused) {
5297:   HANDLE_TH_ERRORS
5298:   const auto& prop = static_cast<CudnnBatchNormBackwardBackward0*>(self->cdata.get())->save_var_;
5299:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5300:   return obj.release().ptr();
5301:   END_HANDLE_TH_ERRORS
5302: }
5303: 
5304: static PyObject* THPCudnnBatchNormBackwardBackward0_weight_getter(THPCppFunction *self, void *_unused) {
5305:   HANDLE_TH_ERRORS
5306:   const auto& prop = static_cast<CudnnBatchNormBackwardBackward0*>(self->cdata.get())->weight_;
5307:   return THPVariable_Wrap(prop.unpack(self->cdata));
5308:   END_HANDLE_TH_ERRORS
5309: }
5310: 
5311: static PyObject* THPCudnnBatchNormBackwardBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
5312:   HANDLE_TH_ERRORS
5313:   const auto& prop = static_cast<CudnnBatchNormBackwardBackward0*>(self->cdata.get())->weight_;
5314:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5315:   return obj.release().ptr();
5316:   END_HANDLE_TH_ERRORS
5317: }
5318: 
5319: static struct PyGetSetDef CudnnBatchNormBackwardBackward0_properties[] = {
5320:   THP_FUNCTION_DEFAULT_PROPERTIES,
5321:   {(char*)"_saved_epsilon", (getter)THPCudnnBatchNormBackwardBackward0_epsilon_getter, nullptr, nullptr, nullptr},
5322:   {(char*)"_saved_grad_output", (getter)THPCudnnBatchNormBackwardBackward0_grad_output_getter, nullptr, nullptr, nullptr},
5323:   {(char*)"_raw_saved_grad_output", (getter)THPCudnnBatchNormBackwardBackward0_grad_output_raw_getter, nullptr, nullptr, nullptr},
5324:   {(char*)"_saved_input", (getter)THPCudnnBatchNormBackwardBackward0_input_getter, nullptr, nullptr, nullptr},
5325:   {(char*)"_raw_saved_input", (getter)THPCudnnBatchNormBackwardBackward0_input_raw_getter, nullptr, nullptr, nullptr},
5326:   {(char*)"_saved_reserveSpace", (getter)THPCudnnBatchNormBackwardBackward0_reserveSpace_getter, nullptr, nullptr, nullptr},
5327:   {(char*)"_raw_saved_reserveSpace", (getter)THPCudnnBatchNormBackwardBackward0_reserveSpace_raw_getter, nullptr, nullptr, nullptr},
5328:   {(char*)"_saved_running_mean", (getter)THPCudnnBatchNormBackwardBackward0_running_mean_getter, nullptr, nullptr, nullptr},
5329:   {(char*)"_raw_saved_running_mean", (getter)THPCudnnBatchNormBackwardBackward0_running_mean_raw_getter, nullptr, nullptr, nullptr},
5330:   {(char*)"_saved_running_var", (getter)THPCudnnBatchNormBackwardBackward0_running_var_getter, nullptr, nullptr, nullptr},
5331:   {(char*)"_raw_saved_running_var", (getter)THPCudnnBatchNormBackwardBackward0_running_var_raw_getter, nullptr, nullptr, nullptr},
5332:   {(char*)"_saved_save_mean", (getter)THPCudnnBatchNormBackwardBackward0_save_mean_getter, nullptr, nullptr, nullptr},
5333:   {(char*)"_raw_saved_save_mean", (getter)THPCudnnBatchNormBackwardBackward0_save_mean_raw_getter, nullptr, nullptr, nullptr},
5334:   {(char*)"_saved_save_var", (getter)THPCudnnBatchNormBackwardBackward0_save_var_getter, nullptr, nullptr, nullptr},
5335:   {(char*)"_raw_saved_save_var", (getter)THPCudnnBatchNormBackwardBackward0_save_var_raw_getter, nullptr, nullptr, nullptr},
5336:   {(char*)"_saved_weight", (getter)THPCudnnBatchNormBackwardBackward0_weight_getter, nullptr, nullptr, nullptr},
5337:   {(char*)"_raw_saved_weight", (getter)THPCudnnBatchNormBackwardBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
5338:   {nullptr} /* sentinel */
5339: };
5340: 
5341: static PyObject* THPCudnnRnnBackward0_batch_first_getter(THPCppFunction *self, void *_unused) {
5342:   HANDLE_TH_ERRORS
5343:   auto prop = static_cast<CudnnRnnBackward0*>(self->cdata.get())->batch_first;
5344:   if (prop) {
5345:     Py_RETURN_TRUE;
5346:   } else {
5347:     Py_RETURN_FALSE;
5348:   }
5349:   END_HANDLE_TH_ERRORS
5350: }
5351: 
5352: static PyObject* THPCudnnRnnBackward0_batch_sizes_getter(THPCppFunction *self, void *_unused) {
5353:   HANDLE_TH_ERRORS
5354:   auto prop = static_cast<CudnnRnnBackward0*>(self->cdata.get())->batch_sizes;
5355:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5356:   for (auto i : c10::irange(prop.size())) {
5357:       auto si = prop[i];
5358:       if (auto m = si.maybe_as_int()) {
5359:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
5360:       } else {
```

- EN: The main execution path in this span is carried by `THPCudnnBatchNormBackwardBackward0_save_mean_raw_getter`, `cast`, `THPCudnnBatchNormBackwardBackward0_save_var_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPCudnnBatchNormBackwardBackward0_save_mean_raw_getter`, `cast`, `THPCudnnBatchNormBackwardBackward0_save_var_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5361-5440

```cpp
5361:         auto py_symint = py::cast(si).release().ptr();
5362:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
5363:       }
5364:   }
5365:   return tup;
5366:   END_HANDLE_TH_ERRORS
5367: }
5368: 
5369: static PyObject* THPCudnnRnnBackward0_bidirectional_getter(THPCppFunction *self, void *_unused) {
5370:   HANDLE_TH_ERRORS
5371:   auto prop = static_cast<CudnnRnnBackward0*>(self->cdata.get())->bidirectional;
5372:   if (prop) {
5373:     Py_RETURN_TRUE;
5374:   } else {
5375:     Py_RETURN_FALSE;
5376:   }
5377:   END_HANDLE_TH_ERRORS
5378: }
5379: 
5380: static PyObject* THPCudnnRnnBackward0_cx_getter(THPCppFunction *self, void *_unused) {
5381:   HANDLE_TH_ERRORS
5382:   const auto& prop = static_cast<CudnnRnnBackward0*>(self->cdata.get())->cx_;
5383:   return THPVariable_Wrap(prop.unpack(self->cdata));
5384:   END_HANDLE_TH_ERRORS
5385: }
5386: 
5387: static PyObject* THPCudnnRnnBackward0_cx_raw_getter(THPCppFunction *self, void *_unused) {
5388:   HANDLE_TH_ERRORS
5389:   const auto& prop = static_cast<CudnnRnnBackward0*>(self->cdata.get())->cx_;
5390:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5391:   return obj.release().ptr();
5392:   END_HANDLE_TH_ERRORS
5393: }
5394: 
5395: static PyObject* THPCudnnRnnBackward0_dropout_getter(THPCppFunction *self, void *_unused) {
5396:   HANDLE_TH_ERRORS
5397:   auto prop = static_cast<CudnnRnnBackward0*>(self->cdata.get())->dropout;
5398:   return PyFloat_FromDouble((double) prop);
5399:   END_HANDLE_TH_ERRORS
5400: }
5401: 
5402: static PyObject* THPCudnnRnnBackward0_dropout_state_getter(THPCppFunction *self, void *_unused) {
5403:   HANDLE_TH_ERRORS
5404:   const auto& prop = static_cast<CudnnRnnBackward0*>(self->cdata.get())->dropout_state_;
5405:   return THPVariable_Wrap(prop.unpack(self->cdata));
5406:   END_HANDLE_TH_ERRORS
5407: }
5408: 
5409: static PyObject* THPCudnnRnnBackward0_dropout_state_raw_getter(THPCppFunction *self, void *_unused) {
5410:   HANDLE_TH_ERRORS
5411:   const auto& prop = static_cast<CudnnRnnBackward0*>(self->cdata.get())->dropout_state_;
5412:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5413:   return obj.release().ptr();
5414:   END_HANDLE_TH_ERRORS
5415: }
5416: 
5417: static PyObject* THPCudnnRnnBackward0_hidden_size_getter(THPCppFunction *self, void *_unused) {
5418:   HANDLE_TH_ERRORS
5419:   auto prop = static_cast<CudnnRnnBackward0*>(self->cdata.get())->hidden_size;
5420:   if (auto m = prop.maybe_as_int()) {
5421:     return PyLong_FromUnsignedLong(*m);
5422:   } else {
5423:     return py::cast(prop).release().ptr();
5424:   }
5425:   END_HANDLE_TH_ERRORS
5426: }
5427: 
5428: static PyObject* THPCudnnRnnBackward0_hx_getter(THPCppFunction *self, void *_unused) {
5429:   HANDLE_TH_ERRORS
5430:   const auto& prop = static_cast<CudnnRnnBackward0*>(self->cdata.get())->hx_;
5431:   return THPVariable_Wrap(prop.unpack(self->cdata));
5432:   END_HANDLE_TH_ERRORS
5433: }
5434: 
5435: static PyObject* THPCudnnRnnBackward0_hx_raw_getter(THPCppFunction *self, void *_unused) {
5436:   HANDLE_TH_ERRORS
5437:   const auto& prop = static_cast<CudnnRnnBackward0*>(self->cdata.get())->hx_;
5438:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5439:   return obj.release().ptr();
5440:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `cast`, `PyTuple_SetItem`, `THPCudnnRnnBackward0_bidirectional_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `cast`, `PyTuple_SetItem`, `THPCudnnRnnBackward0_bidirectional_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5441-5520

```cpp
5441: }
5442: 
5443: static PyObject* THPCudnnRnnBackward0_input_getter(THPCppFunction *self, void *_unused) {
5444:   HANDLE_TH_ERRORS
5445:   const auto& prop = static_cast<CudnnRnnBackward0*>(self->cdata.get())->input_;
5446:   return THPVariable_Wrap(prop.unpack(self->cdata));
5447:   END_HANDLE_TH_ERRORS
5448: }
5449: 
5450: static PyObject* THPCudnnRnnBackward0_input_raw_getter(THPCppFunction *self, void *_unused) {
5451:   HANDLE_TH_ERRORS
5452:   const auto& prop = static_cast<CudnnRnnBackward0*>(self->cdata.get())->input_;
5453:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5454:   return obj.release().ptr();
5455:   END_HANDLE_TH_ERRORS
5456: }
5457: 
5458: static PyObject* THPCudnnRnnBackward0_mode_getter(THPCppFunction *self, void *_unused) {
5459:   HANDLE_TH_ERRORS
5460:   auto prop = static_cast<CudnnRnnBackward0*>(self->cdata.get())->mode;
5461:   return PyLong_FromUnsignedLong((int64_t) prop);
5462:   END_HANDLE_TH_ERRORS
5463: }
5464: 
5465: static PyObject* THPCudnnRnnBackward0_num_layers_getter(THPCppFunction *self, void *_unused) {
5466:   HANDLE_TH_ERRORS
5467:   auto prop = static_cast<CudnnRnnBackward0*>(self->cdata.get())->num_layers;
5468:   return PyLong_FromUnsignedLong((int64_t) prop);
5469:   END_HANDLE_TH_ERRORS
5470: }
5471: 
5472: static PyObject* THPCudnnRnnBackward0_proj_size_getter(THPCppFunction *self, void *_unused) {
5473:   HANDLE_TH_ERRORS
5474:   auto prop = static_cast<CudnnRnnBackward0*>(self->cdata.get())->proj_size;
5475:   if (auto m = prop.maybe_as_int()) {
5476:     return PyLong_FromUnsignedLong(*m);
5477:   } else {
5478:     return py::cast(prop).release().ptr();
5479:   }
5480:   END_HANDLE_TH_ERRORS
5481: }
5482: 
5483: static PyObject* THPCudnnRnnBackward0_train_getter(THPCppFunction *self, void *_unused) {
5484:   HANDLE_TH_ERRORS
5485:   auto prop = static_cast<CudnnRnnBackward0*>(self->cdata.get())->train;
5486:   if (prop) {
5487:     Py_RETURN_TRUE;
5488:   } else {
5489:     Py_RETURN_FALSE;
5490:   }
5491:   END_HANDLE_TH_ERRORS
5492: }
5493: 
5494: static PyObject* THPCudnnRnnBackward0_weight_getter(THPCppFunction *self, void *_unused) {
5495:   HANDLE_TH_ERRORS
5496:   const auto *node = static_cast<CudnnRnnBackward0*>(self->cdata.get());
5497:   const auto& prop = node->weight_;
5498:   if (node->weight_released_) {
5499:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
5500:     return nullptr;
5501:   }
5502:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5503:   for (auto i: c10::irange(prop.size())) {
5504:     PyTuple_SetItem(tup, (Py_ssize_t) i, THPVariable_Wrap(prop[i].unpack(self->cdata)));
5505:   }
5506:   return tup;
5507:   END_HANDLE_TH_ERRORS
5508: }
5509: 
5510: static PyObject* THPCudnnRnnBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
5511:   HANDLE_TH_ERRORS
5512:   const auto *node = static_cast<CudnnRnnBackward0*>(self->cdata.get());
5513:   const auto& prop = node->weight_;
5514:   if (node->weight_released_) {
5515:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
5516:     return nullptr;
5517:   }
5518:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5519:   for (auto i : c10::irange(prop.size())) {
5520:     pybind11::object obj = pybind11::cast(prop[i], pybind11::return_value_policy::reference);
```

- EN: The main execution path in this span is carried by `THPCudnnRnnBackward0_input_getter`, `THPVariable_Wrap`, `THPCudnnRnnBackward0_input_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPCudnnRnnBackward0_input_getter`, `THPVariable_Wrap`, `THPCudnnRnnBackward0_input_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5521-5600

```cpp
5521:     PyTuple_SetItem(tup, (Py_ssize_t) i, obj.release().ptr());
5522:   }
5523:   return tup;
5524:   END_HANDLE_TH_ERRORS
5525: }
5526: 
5527: static PyObject* THPCudnnRnnBackward0_weight_stride0_getter(THPCppFunction *self, void *_unused) {
5528:   HANDLE_TH_ERRORS
5529:   auto prop = static_cast<CudnnRnnBackward0*>(self->cdata.get())->weight_stride0;
5530:   return PyLong_FromUnsignedLong((int64_t) prop);
5531:   END_HANDLE_TH_ERRORS
5532: }
5533: 
5534: static PyObject* THPCudnnRnnBackward0_result0_getter(THPCppFunction *self, void *_unused) {
5535:   HANDLE_TH_ERRORS
5536:   const auto& prop = static_cast<CudnnRnnBackward0*>(self->cdata.get())->result0_;
5537:   return THPVariable_Wrap(prop.unpack(self->cdata));
5538:   END_HANDLE_TH_ERRORS
5539: }
5540: 
5541: static PyObject* THPCudnnRnnBackward0_result0_raw_getter(THPCppFunction *self, void *_unused) {
5542:   HANDLE_TH_ERRORS
5543:   const auto& prop = static_cast<CudnnRnnBackward0*>(self->cdata.get())->result0_;
5544:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5545:   return obj.release().ptr();
5546:   END_HANDLE_TH_ERRORS
5547: }
5548: 
5549: static PyObject* THPCudnnRnnBackward0_result3_getter(THPCppFunction *self, void *_unused) {
5550:   HANDLE_TH_ERRORS
5551:   const auto& prop = static_cast<CudnnRnnBackward0*>(self->cdata.get())->result3_;
5552:   return THPVariable_Wrap(prop.unpack(self->cdata));
5553:   END_HANDLE_TH_ERRORS
5554: }
5555: 
5556: static PyObject* THPCudnnRnnBackward0_result3_raw_getter(THPCppFunction *self, void *_unused) {
5557:   HANDLE_TH_ERRORS
5558:   const auto& prop = static_cast<CudnnRnnBackward0*>(self->cdata.get())->result3_;
5559:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5560:   return obj.release().ptr();
5561:   END_HANDLE_TH_ERRORS
5562: }
5563: 
5564: static PyObject* THPCudnnRnnBackward0_result4_getter(THPCppFunction *self, void *_unused) {
5565:   HANDLE_TH_ERRORS
5566:   const auto& prop = static_cast<CudnnRnnBackward0*>(self->cdata.get())->result4_;
5567:   return THPVariable_Wrap(prop.unpack(self->cdata));
5568:   END_HANDLE_TH_ERRORS
5569: }
5570: 
5571: static PyObject* THPCudnnRnnBackward0_result4_raw_getter(THPCppFunction *self, void *_unused) {
5572:   HANDLE_TH_ERRORS
5573:   const auto& prop = static_cast<CudnnRnnBackward0*>(self->cdata.get())->result4_;
5574:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5575:   return obj.release().ptr();
5576:   END_HANDLE_TH_ERRORS
5577: }
5578: 
5579: static struct PyGetSetDef CudnnRnnBackward0_properties[] = {
5580:   THP_FUNCTION_DEFAULT_PROPERTIES,
5581:   {(char*)"_saved_batch_first", (getter)THPCudnnRnnBackward0_batch_first_getter, nullptr, nullptr, nullptr},
5582:   {(char*)"_saved_batch_sizes", (getter)THPCudnnRnnBackward0_batch_sizes_getter, nullptr, nullptr, nullptr},
5583:   {(char*)"_saved_bidirectional", (getter)THPCudnnRnnBackward0_bidirectional_getter, nullptr, nullptr, nullptr},
5584:   {(char*)"_saved_cx", (getter)THPCudnnRnnBackward0_cx_getter, nullptr, nullptr, nullptr},
5585:   {(char*)"_raw_saved_cx", (getter)THPCudnnRnnBackward0_cx_raw_getter, nullptr, nullptr, nullptr},
5586:   {(char*)"_saved_dropout", (getter)THPCudnnRnnBackward0_dropout_getter, nullptr, nullptr, nullptr},
5587:   {(char*)"_saved_dropout_state", (getter)THPCudnnRnnBackward0_dropout_state_getter, nullptr, nullptr, nullptr},
5588:   {(char*)"_raw_saved_dropout_state", (getter)THPCudnnRnnBackward0_dropout_state_raw_getter, nullptr, nullptr, nullptr},
5589:   {(char*)"_saved_hidden_size", (getter)THPCudnnRnnBackward0_hidden_size_getter, nullptr, nullptr, nullptr},
5590:   {(char*)"_saved_hx", (getter)THPCudnnRnnBackward0_hx_getter, nullptr, nullptr, nullptr},
5591:   {(char*)"_raw_saved_hx", (getter)THPCudnnRnnBackward0_hx_raw_getter, nullptr, nullptr, nullptr},
5592:   {(char*)"_saved_input", (getter)THPCudnnRnnBackward0_input_getter, nullptr, nullptr, nullptr},
5593:   {(char*)"_raw_saved_input", (getter)THPCudnnRnnBackward0_input_raw_getter, nullptr, nullptr, nullptr},
5594:   {(char*)"_saved_mode", (getter)THPCudnnRnnBackward0_mode_getter, nullptr, nullptr, nullptr},
5595:   {(char*)"_saved_num_layers", (getter)THPCudnnRnnBackward0_num_layers_getter, nullptr, nullptr, nullptr},
5596:   {(char*)"_saved_proj_size", (getter)THPCudnnRnnBackward0_proj_size_getter, nullptr, nullptr, nullptr},
5597:   {(char*)"_saved_train", (getter)THPCudnnRnnBackward0_train_getter, nullptr, nullptr, nullptr},
5598:   {(char*)"_saved_weight", (getter)THPCudnnRnnBackward0_weight_getter, nullptr, nullptr, nullptr},
5599:   {(char*)"_raw_saved_weight", (getter)THPCudnnRnnBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
5600:   {(char*)"_saved_weight_stride0", (getter)THPCudnnRnnBackward0_weight_stride0_getter, nullptr, nullptr, nullptr},
```

- EN: The main execution path in this span is carried by `PyTuple_SetItem`, `THPCudnnRnnBackward0_weight_stride0_getter`, `PyLong_FromUnsignedLong`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyTuple_SetItem`, `THPCudnnRnnBackward0_weight_stride0_getter`, `PyLong_FromUnsignedLong` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5601-5680

```cpp
5601:   {(char*)"_saved_result0", (getter)THPCudnnRnnBackward0_result0_getter, nullptr, nullptr, nullptr},
5602:   {(char*)"_raw_saved_result0", (getter)THPCudnnRnnBackward0_result0_raw_getter, nullptr, nullptr, nullptr},
5603:   {(char*)"_saved_result3", (getter)THPCudnnRnnBackward0_result3_getter, nullptr, nullptr, nullptr},
5604:   {(char*)"_raw_saved_result3", (getter)THPCudnnRnnBackward0_result3_raw_getter, nullptr, nullptr, nullptr},
5605:   {(char*)"_saved_result4", (getter)THPCudnnRnnBackward0_result4_getter, nullptr, nullptr, nullptr},
5606:   {(char*)"_raw_saved_result4", (getter)THPCudnnRnnBackward0_result4_raw_getter, nullptr, nullptr, nullptr},
5607:   {nullptr} /* sentinel */
5608: };
5609: 
5610: static PyObject* THPMiopenBatchNormBackwardBackward0_epsilon_getter(THPCppFunction *self, void *_unused) {
5611:   HANDLE_TH_ERRORS
5612:   auto prop = static_cast<MiopenBatchNormBackwardBackward0*>(self->cdata.get())->epsilon;
5613:   return PyFloat_FromDouble((double) prop);
5614:   END_HANDLE_TH_ERRORS
5615: }
5616: 
5617: static PyObject* THPMiopenBatchNormBackwardBackward0_grad_output_getter(THPCppFunction *self, void *_unused) {
5618:   HANDLE_TH_ERRORS
5619:   const auto& prop = static_cast<MiopenBatchNormBackwardBackward0*>(self->cdata.get())->grad_output_;
5620:   return THPVariable_Wrap(prop.unpack(self->cdata));
5621:   END_HANDLE_TH_ERRORS
5622: }
5623: 
5624: static PyObject* THPMiopenBatchNormBackwardBackward0_grad_output_raw_getter(THPCppFunction *self, void *_unused) {
5625:   HANDLE_TH_ERRORS
5626:   const auto& prop = static_cast<MiopenBatchNormBackwardBackward0*>(self->cdata.get())->grad_output_;
5627:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5628:   return obj.release().ptr();
5629:   END_HANDLE_TH_ERRORS
5630: }
5631: 
5632: static PyObject* THPMiopenBatchNormBackwardBackward0_input_getter(THPCppFunction *self, void *_unused) {
5633:   HANDLE_TH_ERRORS
5634:   const auto& prop = static_cast<MiopenBatchNormBackwardBackward0*>(self->cdata.get())->input_;
5635:   return THPVariable_Wrap(prop.unpack(self->cdata));
5636:   END_HANDLE_TH_ERRORS
5637: }
5638: 
5639: static PyObject* THPMiopenBatchNormBackwardBackward0_input_raw_getter(THPCppFunction *self, void *_unused) {
5640:   HANDLE_TH_ERRORS
5641:   const auto& prop = static_cast<MiopenBatchNormBackwardBackward0*>(self->cdata.get())->input_;
5642:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5643:   return obj.release().ptr();
5644:   END_HANDLE_TH_ERRORS
5645: }
5646: 
5647: static PyObject* THPMiopenBatchNormBackwardBackward0_running_mean_getter(THPCppFunction *self, void *_unused) {
5648:   HANDLE_TH_ERRORS
5649:   const auto& prop = static_cast<MiopenBatchNormBackwardBackward0*>(self->cdata.get())->running_mean_;
5650:   return THPVariable_Wrap(prop.unpack(self->cdata));
5651:   END_HANDLE_TH_ERRORS
5652: }
5653: 
5654: static PyObject* THPMiopenBatchNormBackwardBackward0_running_mean_raw_getter(THPCppFunction *self, void *_unused) {
5655:   HANDLE_TH_ERRORS
5656:   const auto& prop = static_cast<MiopenBatchNormBackwardBackward0*>(self->cdata.get())->running_mean_;
5657:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5658:   return obj.release().ptr();
5659:   END_HANDLE_TH_ERRORS
5660: }
5661: 
5662: static PyObject* THPMiopenBatchNormBackwardBackward0_running_var_getter(THPCppFunction *self, void *_unused) {
5663:   HANDLE_TH_ERRORS
5664:   const auto& prop = static_cast<MiopenBatchNormBackwardBackward0*>(self->cdata.get())->running_var_;
5665:   return THPVariable_Wrap(prop.unpack(self->cdata));
5666:   END_HANDLE_TH_ERRORS
5667: }
5668: 
5669: static PyObject* THPMiopenBatchNormBackwardBackward0_running_var_raw_getter(THPCppFunction *self, void *_unused) {
5670:   HANDLE_TH_ERRORS
5671:   const auto& prop = static_cast<MiopenBatchNormBackwardBackward0*>(self->cdata.get())->running_var_;
5672:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5673:   return obj.release().ptr();
5674:   END_HANDLE_TH_ERRORS
5675: }
5676: 
5677: static PyObject* THPMiopenBatchNormBackwardBackward0_save_mean_getter(THPCppFunction *self, void *_unused) {
5678:   HANDLE_TH_ERRORS
5679:   const auto& prop = static_cast<MiopenBatchNormBackwardBackward0*>(self->cdata.get())->save_mean_;
5680:   return THPVariable_Wrap(prop.unpack(self->cdata));
```

- EN: The main execution path in this span is carried by `THPMiopenBatchNormBackwardBackward0_epsilon_getter`, `PyFloat_FromDouble`, `THPMiopenBatchNormBackwardBackward0_grad_output_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPMiopenBatchNormBackwardBackward0_epsilon_getter`, `PyFloat_FromDouble`, `THPMiopenBatchNormBackwardBackward0_grad_output_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5681-5760

```cpp
5681:   END_HANDLE_TH_ERRORS
5682: }
5683: 
5684: static PyObject* THPMiopenBatchNormBackwardBackward0_save_mean_raw_getter(THPCppFunction *self, void *_unused) {
5685:   HANDLE_TH_ERRORS
5686:   const auto& prop = static_cast<MiopenBatchNormBackwardBackward0*>(self->cdata.get())->save_mean_;
5687:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5688:   return obj.release().ptr();
5689:   END_HANDLE_TH_ERRORS
5690: }
5691: 
5692: static PyObject* THPMiopenBatchNormBackwardBackward0_save_var_getter(THPCppFunction *self, void *_unused) {
5693:   HANDLE_TH_ERRORS
5694:   const auto& prop = static_cast<MiopenBatchNormBackwardBackward0*>(self->cdata.get())->save_var_;
5695:   return THPVariable_Wrap(prop.unpack(self->cdata));
5696:   END_HANDLE_TH_ERRORS
5697: }
5698: 
5699: static PyObject* THPMiopenBatchNormBackwardBackward0_save_var_raw_getter(THPCppFunction *self, void *_unused) {
5700:   HANDLE_TH_ERRORS
5701:   const auto& prop = static_cast<MiopenBatchNormBackwardBackward0*>(self->cdata.get())->save_var_;
5702:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5703:   return obj.release().ptr();
5704:   END_HANDLE_TH_ERRORS
5705: }
5706: 
5707: static PyObject* THPMiopenBatchNormBackwardBackward0_weight_getter(THPCppFunction *self, void *_unused) {
5708:   HANDLE_TH_ERRORS
5709:   const auto& prop = static_cast<MiopenBatchNormBackwardBackward0*>(self->cdata.get())->weight_;
5710:   return THPVariable_Wrap(prop.unpack(self->cdata));
5711:   END_HANDLE_TH_ERRORS
5712: }
5713: 
5714: static PyObject* THPMiopenBatchNormBackwardBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
5715:   HANDLE_TH_ERRORS
5716:   const auto& prop = static_cast<MiopenBatchNormBackwardBackward0*>(self->cdata.get())->weight_;
5717:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5718:   return obj.release().ptr();
5719:   END_HANDLE_TH_ERRORS
5720: }
5721: 
5722: static struct PyGetSetDef MiopenBatchNormBackwardBackward0_properties[] = {
5723:   THP_FUNCTION_DEFAULT_PROPERTIES,
5724:   {(char*)"_saved_epsilon", (getter)THPMiopenBatchNormBackwardBackward0_epsilon_getter, nullptr, nullptr, nullptr},
5725:   {(char*)"_saved_grad_output", (getter)THPMiopenBatchNormBackwardBackward0_grad_output_getter, nullptr, nullptr, nullptr},
5726:   {(char*)"_raw_saved_grad_output", (getter)THPMiopenBatchNormBackwardBackward0_grad_output_raw_getter, nullptr, nullptr, nullptr},
5727:   {(char*)"_saved_input", (getter)THPMiopenBatchNormBackwardBackward0_input_getter, nullptr, nullptr, nullptr},
5728:   {(char*)"_raw_saved_input", (getter)THPMiopenBatchNormBackwardBackward0_input_raw_getter, nullptr, nullptr, nullptr},
5729:   {(char*)"_saved_running_mean", (getter)THPMiopenBatchNormBackwardBackward0_running_mean_getter, nullptr, nullptr, nullptr},
5730:   {(char*)"_raw_saved_running_mean", (getter)THPMiopenBatchNormBackwardBackward0_running_mean_raw_getter, nullptr, nullptr, nullptr},
5731:   {(char*)"_saved_running_var", (getter)THPMiopenBatchNormBackwardBackward0_running_var_getter, nullptr, nullptr, nullptr},
5732:   {(char*)"_raw_saved_running_var", (getter)THPMiopenBatchNormBackwardBackward0_running_var_raw_getter, nullptr, nullptr, nullptr},
5733:   {(char*)"_saved_save_mean", (getter)THPMiopenBatchNormBackwardBackward0_save_mean_getter, nullptr, nullptr, nullptr},
5734:   {(char*)"_raw_saved_save_mean", (getter)THPMiopenBatchNormBackwardBackward0_save_mean_raw_getter, nullptr, nullptr, nullptr},
5735:   {(char*)"_saved_save_var", (getter)THPMiopenBatchNormBackwardBackward0_save_var_getter, nullptr, nullptr, nullptr},
5736:   {(char*)"_raw_saved_save_var", (getter)THPMiopenBatchNormBackwardBackward0_save_var_raw_getter, nullptr, nullptr, nullptr},
5737:   {(char*)"_saved_weight", (getter)THPMiopenBatchNormBackwardBackward0_weight_getter, nullptr, nullptr, nullptr},
5738:   {(char*)"_raw_saved_weight", (getter)THPMiopenBatchNormBackwardBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
5739:   {nullptr} /* sentinel */
5740: };
5741: 
5742: static PyObject* THPMkldnnConvolutionBackward0_bias_sym_sizes_opt_getter(THPCppFunction *self, void *_unused) {
5743:   HANDLE_TH_ERRORS
5744:   auto opt_prop = static_cast<MkldnnConvolutionBackward0*>(self->cdata.get())->bias_sym_sizes_opt;
5745:   if (!opt_prop.list.has_value()) {
5746:     Py_RETURN_NONE;
5747:   }
5748:   auto prop = opt_prop.list.value();
5749:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5750:   for (auto i : c10::irange(prop.size())) {
5751:       auto si = prop[i];
5752:       if (auto m = si.maybe_as_int()) {
5753:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
5754:       } else {
5755:         auto py_symint = py::cast(si).release().ptr();
5756:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
5757:       }
5758:   }
5759:   return tup;
5760:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPMiopenBatchNormBackwardBackward0_save_mean_raw_getter`, `cast`, `THPMiopenBatchNormBackwardBackward0_save_var_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPMiopenBatchNormBackwardBackward0_save_mean_raw_getter`, `cast`, `THPMiopenBatchNormBackwardBackward0_save_var_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5761-5840

```cpp
5761: }
5762: 
5763: static PyObject* THPMkldnnConvolutionBackward0_dilation_getter(THPCppFunction *self, void *_unused) {
5764:   HANDLE_TH_ERRORS
5765:   auto prop = static_cast<MkldnnConvolutionBackward0*>(self->cdata.get())->dilation;
5766:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5767:   for (auto i : c10::irange(prop.size())) {
5768:       auto si = prop[i];
5769:       if (auto m = si.maybe_as_int()) {
5770:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
5771:       } else {
5772:         auto py_symint = py::cast(si).release().ptr();
5773:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
5774:       }
5775:   }
5776:   return tup;
5777:   END_HANDLE_TH_ERRORS
5778: }
5779: 
5780: static PyObject* THPMkldnnConvolutionBackward0_groups_getter(THPCppFunction *self, void *_unused) {
5781:   HANDLE_TH_ERRORS
5782:   auto prop = static_cast<MkldnnConvolutionBackward0*>(self->cdata.get())->groups;
5783:   if (auto m = prop.maybe_as_int()) {
5784:     return PyLong_FromUnsignedLong(*m);
5785:   } else {
5786:     return py::cast(prop).release().ptr();
5787:   }
5788:   END_HANDLE_TH_ERRORS
5789: }
5790: 
5791: static PyObject* THPMkldnnConvolutionBackward0_padding_getter(THPCppFunction *self, void *_unused) {
5792:   HANDLE_TH_ERRORS
5793:   auto prop = static_cast<MkldnnConvolutionBackward0*>(self->cdata.get())->padding;
5794:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5795:   for (auto i : c10::irange(prop.size())) {
5796:       auto si = prop[i];
5797:       if (auto m = si.maybe_as_int()) {
5798:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
5799:       } else {
5800:         auto py_symint = py::cast(si).release().ptr();
5801:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
5802:       }
5803:   }
5804:   return tup;
5805:   END_HANDLE_TH_ERRORS
5806: }
5807: 
5808: static PyObject* THPMkldnnConvolutionBackward0_self_getter(THPCppFunction *self, void *_unused) {
5809:   HANDLE_TH_ERRORS
5810:   const auto& prop = static_cast<MkldnnConvolutionBackward0*>(self->cdata.get())->self_;
5811:   return THPVariable_Wrap(prop.unpack(self->cdata));
5812:   END_HANDLE_TH_ERRORS
5813: }
5814: 
5815: static PyObject* THPMkldnnConvolutionBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
5816:   HANDLE_TH_ERRORS
5817:   const auto& prop = static_cast<MkldnnConvolutionBackward0*>(self->cdata.get())->self_;
5818:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5819:   return obj.release().ptr();
5820:   END_HANDLE_TH_ERRORS
5821: }
5822: 
5823: static PyObject* THPMkldnnConvolutionBackward0_stride_getter(THPCppFunction *self, void *_unused) {
5824:   HANDLE_TH_ERRORS
5825:   auto prop = static_cast<MkldnnConvolutionBackward0*>(self->cdata.get())->stride;
5826:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5827:   for (auto i : c10::irange(prop.size())) {
5828:       auto si = prop[i];
5829:       if (auto m = si.maybe_as_int()) {
5830:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
5831:       } else {
5832:         auto py_symint = py::cast(si).release().ptr();
5833:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
5834:       }
5835:   }
5836:   return tup;
5837:   END_HANDLE_TH_ERRORS
5838: }
5839: 
5840: static PyObject* THPMkldnnConvolutionBackward0_weight_getter(THPCppFunction *self, void *_unused) {
```

- EN: The main execution path in this span is carried by `THPMkldnnConvolutionBackward0_dilation_getter`, `PyTuple_New`, `PyTuple_SetItem`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPMkldnnConvolutionBackward0_dilation_getter`, `PyTuple_New`, `PyTuple_SetItem` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5841-5920

```cpp
5841:   HANDLE_TH_ERRORS
5842:   const auto& prop = static_cast<MkldnnConvolutionBackward0*>(self->cdata.get())->weight_;
5843:   return THPVariable_Wrap(prop.unpack(self->cdata));
5844:   END_HANDLE_TH_ERRORS
5845: }
5846: 
5847: static PyObject* THPMkldnnConvolutionBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
5848:   HANDLE_TH_ERRORS
5849:   const auto& prop = static_cast<MkldnnConvolutionBackward0*>(self->cdata.get())->weight_;
5850:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5851:   return obj.release().ptr();
5852:   END_HANDLE_TH_ERRORS
5853: }
5854: 
5855: static struct PyGetSetDef MkldnnConvolutionBackward0_properties[] = {
5856:   THP_FUNCTION_DEFAULT_PROPERTIES,
5857:   {(char*)"_saved_bias_sym_sizes_opt", (getter)THPMkldnnConvolutionBackward0_bias_sym_sizes_opt_getter, nullptr, nullptr, nullptr},
5858:   {(char*)"_saved_dilation", (getter)THPMkldnnConvolutionBackward0_dilation_getter, nullptr, nullptr, nullptr},
5859:   {(char*)"_saved_groups", (getter)THPMkldnnConvolutionBackward0_groups_getter, nullptr, nullptr, nullptr},
5860:   {(char*)"_saved_padding", (getter)THPMkldnnConvolutionBackward0_padding_getter, nullptr, nullptr, nullptr},
5861:   {(char*)"_saved_self", (getter)THPMkldnnConvolutionBackward0_self_getter, nullptr, nullptr, nullptr},
5862:   {(char*)"_raw_saved_self", (getter)THPMkldnnConvolutionBackward0_self_raw_getter, nullptr, nullptr, nullptr},
5863:   {(char*)"_saved_stride", (getter)THPMkldnnConvolutionBackward0_stride_getter, nullptr, nullptr, nullptr},
5864:   {(char*)"_saved_weight", (getter)THPMkldnnConvolutionBackward0_weight_getter, nullptr, nullptr, nullptr},
5865:   {(char*)"_raw_saved_weight", (getter)THPMkldnnConvolutionBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
5866:   {nullptr} /* sentinel */
5867: };
5868: 
5869: static PyObject* THPMkldnnLinearBackward0_self_getter(THPCppFunction *self, void *_unused) {
5870:   HANDLE_TH_ERRORS
5871:   const auto& prop = static_cast<MkldnnLinearBackward0*>(self->cdata.get())->self_;
5872:   return THPVariable_Wrap(prop.unpack(self->cdata));
5873:   END_HANDLE_TH_ERRORS
5874: }
5875: 
5876: static PyObject* THPMkldnnLinearBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
5877:   HANDLE_TH_ERRORS
5878:   const auto& prop = static_cast<MkldnnLinearBackward0*>(self->cdata.get())->self_;
5879:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5880:   return obj.release().ptr();
5881:   END_HANDLE_TH_ERRORS
5882: }
5883: 
5884: static PyObject* THPMkldnnLinearBackward0_weight_getter(THPCppFunction *self, void *_unused) {
5885:   HANDLE_TH_ERRORS
5886:   const auto& prop = static_cast<MkldnnLinearBackward0*>(self->cdata.get())->weight_;
5887:   return THPVariable_Wrap(prop.unpack(self->cdata));
5888:   END_HANDLE_TH_ERRORS
5889: }
5890: 
5891: static PyObject* THPMkldnnLinearBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
5892:   HANDLE_TH_ERRORS
5893:   const auto& prop = static_cast<MkldnnLinearBackward0*>(self->cdata.get())->weight_;
5894:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5895:   return obj.release().ptr();
5896:   END_HANDLE_TH_ERRORS
5897: }
5898: 
5899: static struct PyGetSetDef MkldnnLinearBackward0_properties[] = {
5900:   THP_FUNCTION_DEFAULT_PROPERTIES,
5901:   {(char*)"_saved_self", (getter)THPMkldnnLinearBackward0_self_getter, nullptr, nullptr, nullptr},
5902:   {(char*)"_raw_saved_self", (getter)THPMkldnnLinearBackward0_self_raw_getter, nullptr, nullptr, nullptr},
5903:   {(char*)"_saved_weight", (getter)THPMkldnnLinearBackward0_weight_getter, nullptr, nullptr, nullptr},
5904:   {(char*)"_raw_saved_weight", (getter)THPMkldnnLinearBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
5905:   {nullptr} /* sentinel */
5906: };
5907: 
5908: static PyObject* THPToPaddedTensorBackward0_self_getter(THPCppFunction *self, void *_unused) {
5909:   HANDLE_TH_ERRORS
5910:   const auto& prop = static_cast<ToPaddedTensorBackward0*>(self->cdata.get())->self_;
5911:   return THPVariable_Wrap(prop.unpack(self->cdata));
5912:   END_HANDLE_TH_ERRORS
5913: }
5914: 
5915: static PyObject* THPToPaddedTensorBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
5916:   HANDLE_TH_ERRORS
5917:   const auto& prop = static_cast<ToPaddedTensorBackward0*>(self->cdata.get())->self_;
5918:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5919:   return obj.release().ptr();
5920:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPVariable_Wrap`, `THPMkldnnConvolutionBackward0_weight_raw_getter`, `cast`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_Wrap`, `THPMkldnnConvolutionBackward0_weight_raw_getter`, `cast` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5921-6000

```cpp
5921: }
5922: 
5923: static struct PyGetSetDef ToPaddedTensorBackward0_properties[] = {
5924:   THP_FUNCTION_DEFAULT_PROPERTIES,
5925:   {(char*)"_saved_self", (getter)THPToPaddedTensorBackward0_self_getter, nullptr, nullptr, nullptr},
5926:   {(char*)"_raw_saved_self", (getter)THPToPaddedTensorBackward0_self_raw_getter, nullptr, nullptr, nullptr},
5927:   {nullptr} /* sentinel */
5928: };
5929: 
5930: static PyObject* THPFlashAttentionBackward0_cum_seq_k_getter(THPCppFunction *self, void *_unused) {
5931:   HANDLE_TH_ERRORS
5932:   const auto& prop = static_cast<FlashAttentionBackward0*>(self->cdata.get())->cum_seq_k_;
5933:   return THPVariable_Wrap(prop.unpack(self->cdata));
5934:   END_HANDLE_TH_ERRORS
5935: }
5936: 
5937: static PyObject* THPFlashAttentionBackward0_cum_seq_k_raw_getter(THPCppFunction *self, void *_unused) {
5938:   HANDLE_TH_ERRORS
5939:   const auto& prop = static_cast<FlashAttentionBackward0*>(self->cdata.get())->cum_seq_k_;
5940:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5941:   return obj.release().ptr();
5942:   END_HANDLE_TH_ERRORS
5943: }
5944: 
5945: static PyObject* THPFlashAttentionBackward0_cum_seq_q_getter(THPCppFunction *self, void *_unused) {
5946:   HANDLE_TH_ERRORS
5947:   const auto& prop = static_cast<FlashAttentionBackward0*>(self->cdata.get())->cum_seq_q_;
5948:   return THPVariable_Wrap(prop.unpack(self->cdata));
5949:   END_HANDLE_TH_ERRORS
5950: }
5951: 
5952: static PyObject* THPFlashAttentionBackward0_cum_seq_q_raw_getter(THPCppFunction *self, void *_unused) {
5953:   HANDLE_TH_ERRORS
5954:   const auto& prop = static_cast<FlashAttentionBackward0*>(self->cdata.get())->cum_seq_q_;
5955:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5956:   return obj.release().ptr();
5957:   END_HANDLE_TH_ERRORS
5958: }
5959: 
5960: static PyObject* THPFlashAttentionBackward0_dropout_p_getter(THPCppFunction *self, void *_unused) {
5961:   HANDLE_TH_ERRORS
5962:   auto prop = static_cast<FlashAttentionBackward0*>(self->cdata.get())->dropout_p;
5963:   return PyFloat_FromDouble((double) prop);
5964:   END_HANDLE_TH_ERRORS
5965: }
5966: 
5967: static PyObject* THPFlashAttentionBackward0_is_causal_getter(THPCppFunction *self, void *_unused) {
5968:   HANDLE_TH_ERRORS
5969:   auto prop = static_cast<FlashAttentionBackward0*>(self->cdata.get())->is_causal;
5970:   if (prop) {
5971:     Py_RETURN_TRUE;
5972:   } else {
5973:     Py_RETURN_FALSE;
5974:   }
5975:   END_HANDLE_TH_ERRORS
5976: }
5977: 
5978: static PyObject* THPFlashAttentionBackward0_key_getter(THPCppFunction *self, void *_unused) {
5979:   HANDLE_TH_ERRORS
5980:   const auto& prop = static_cast<FlashAttentionBackward0*>(self->cdata.get())->key_;
5981:   return THPVariable_Wrap(prop.unpack(self->cdata));
5982:   END_HANDLE_TH_ERRORS
5983: }
5984: 
5985: static PyObject* THPFlashAttentionBackward0_key_raw_getter(THPCppFunction *self, void *_unused) {
5986:   HANDLE_TH_ERRORS
5987:   const auto& prop = static_cast<FlashAttentionBackward0*>(self->cdata.get())->key_;
5988:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5989:   return obj.release().ptr();
5990:   END_HANDLE_TH_ERRORS
5991: }
5992: 
5993: static PyObject* THPFlashAttentionBackward0_max_k_getter(THPCppFunction *self, void *_unused) {
5994:   HANDLE_TH_ERRORS
5995:   auto prop = static_cast<FlashAttentionBackward0*>(self->cdata.get())->max_k;
5996:   if (auto m = prop.maybe_as_int()) {
5997:     return PyLong_FromUnsignedLong(*m);
5998:   } else {
5999:     return py::cast(prop).release().ptr();
6000:   }
```

- EN: The main execution path in this span is carried by `THPFlashAttentionBackward0_cum_seq_k_getter`, `THPVariable_Wrap`, `THPFlashAttentionBackward0_cum_seq_k_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPFlashAttentionBackward0_cum_seq_k_getter`, `THPVariable_Wrap`, `THPFlashAttentionBackward0_cum_seq_k_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 6001-6080

```cpp
6001:   END_HANDLE_TH_ERRORS
6002: }
6003: 
6004: static PyObject* THPFlashAttentionBackward0_max_q_getter(THPCppFunction *self, void *_unused) {
6005:   HANDLE_TH_ERRORS
6006:   auto prop = static_cast<FlashAttentionBackward0*>(self->cdata.get())->max_q;
6007:   if (auto m = prop.maybe_as_int()) {
6008:     return PyLong_FromUnsignedLong(*m);
6009:   } else {
6010:     return py::cast(prop).release().ptr();
6011:   }
6012:   END_HANDLE_TH_ERRORS
6013: }
6014: 
6015: static PyObject* THPFlashAttentionBackward0_query_getter(THPCppFunction *self, void *_unused) {
6016:   HANDLE_TH_ERRORS
6017:   const auto& prop = static_cast<FlashAttentionBackward0*>(self->cdata.get())->query_;
6018:   return THPVariable_Wrap(prop.unpack(self->cdata));
6019:   END_HANDLE_TH_ERRORS
6020: }
6021: 
6022: static PyObject* THPFlashAttentionBackward0_query_raw_getter(THPCppFunction *self, void *_unused) {
6023:   HANDLE_TH_ERRORS
6024:   const auto& prop = static_cast<FlashAttentionBackward0*>(self->cdata.get())->query_;
6025:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
6026:   return obj.release().ptr();
6027:   END_HANDLE_TH_ERRORS
6028: }
6029: 
6030: static PyObject* THPFlashAttentionBackward0_scale_getter(THPCppFunction *self, void *_unused) {
6031:   HANDLE_TH_ERRORS
6032:   auto opt_prop = static_cast<FlashAttentionBackward0*>(self->cdata.get())->scale;
6033:   if (!opt_prop.has_value()) {
6034:     Py_RETURN_NONE;
6035:   }
6036:   auto prop = opt_prop.value();
6037:   return PyFloat_FromDouble((double) prop);
6038:   END_HANDLE_TH_ERRORS
6039: }
6040: 
6041: static PyObject* THPFlashAttentionBackward0_value_getter(THPCppFunction *self, void *_unused) {
6042:   HANDLE_TH_ERRORS
6043:   const auto& prop = static_cast<FlashAttentionBackward0*>(self->cdata.get())->value_;
6044:   return THPVariable_Wrap(prop.unpack(self->cdata));
6045:   END_HANDLE_TH_ERRORS
6046: }
6047: 
6048: static PyObject* THPFlashAttentionBackward0_value_raw_getter(THPCppFunction *self, void *_unused) {
6049:   HANDLE_TH_ERRORS
6050:   const auto& prop = static_cast<FlashAttentionBackward0*>(self->cdata.get())->value_;
6051:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
6052:   return obj.release().ptr();
6053:   END_HANDLE_TH_ERRORS
6054: }
6055: 
6056: static PyObject* THPFlashAttentionBackward0_window_size_left_getter(THPCppFunction *self, void *_unused) {
6057:   HANDLE_TH_ERRORS
6058:   auto opt_prop = static_cast<FlashAttentionBackward0*>(self->cdata.get())->window_size_left;
6059:   if (!opt_prop.has_value()) {
6060:     Py_RETURN_NONE;
6061:   }
6062:   auto prop = opt_prop.value();
6063:   if (auto m = prop.maybe_as_int()) {
6064:     return PyLong_FromUnsignedLong(*m);
6065:   } else {
6066:     return py::cast(prop).release().ptr();
6067:   }
6068:   END_HANDLE_TH_ERRORS
6069: }
6070: 
6071: static PyObject* THPFlashAttentionBackward0_window_size_right_getter(THPCppFunction *self, void *_unused) {
6072:   HANDLE_TH_ERRORS
6073:   auto opt_prop = static_cast<FlashAttentionBackward0*>(self->cdata.get())->window_size_right;
6074:   if (!opt_prop.has_value()) {
6075:     Py_RETURN_NONE;
6076:   }
6077:   auto prop = opt_prop.value();
6078:   if (auto m = prop.maybe_as_int()) {
6079:     return PyLong_FromUnsignedLong(*m);
6080:   } else {
```

- EN: The main execution path in this span is carried by `THPFlashAttentionBackward0_max_q_getter`, `PyLong_FromUnsignedLong`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPFlashAttentionBackward0_max_q_getter`, `PyLong_FromUnsignedLong`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 6081-6160

```cpp
6081:     return py::cast(prop).release().ptr();
6082:   }
6083:   END_HANDLE_TH_ERRORS
6084: }
6085: 
6086: static PyObject* THPFlashAttentionBackward0_output_getter(THPCppFunction *self, void *_unused) {
6087:   HANDLE_TH_ERRORS
6088:   const auto& prop = static_cast<FlashAttentionBackward0*>(self->cdata.get())->output_;
6089:   return THPVariable_Wrap(prop.unpack(self->cdata));
6090:   END_HANDLE_TH_ERRORS
6091: }
6092: 
6093: static PyObject* THPFlashAttentionBackward0_output_raw_getter(THPCppFunction *self, void *_unused) {
6094:   HANDLE_TH_ERRORS
6095:   const auto& prop = static_cast<FlashAttentionBackward0*>(self->cdata.get())->output_;
6096:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
6097:   return obj.release().ptr();
6098:   END_HANDLE_TH_ERRORS
6099: }
6100: 
6101: static PyObject* THPFlashAttentionBackward0_rng_state_getter(THPCppFunction *self, void *_unused) {
6102:   HANDLE_TH_ERRORS
6103:   const auto& prop = static_cast<FlashAttentionBackward0*>(self->cdata.get())->rng_state_;
6104:   return THPVariable_Wrap(prop.unpack(self->cdata));
6105:   END_HANDLE_TH_ERRORS
6106: }
6107: 
6108: static PyObject* THPFlashAttentionBackward0_rng_state_raw_getter(THPCppFunction *self, void *_unused) {
6109:   HANDLE_TH_ERRORS
6110:   const auto& prop = static_cast<FlashAttentionBackward0*>(self->cdata.get())->rng_state_;
6111:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
6112:   return obj.release().ptr();
6113:   END_HANDLE_TH_ERRORS
6114: }
6115: 
6116: static PyObject* THPFlashAttentionBackward0_softmax_logsumexp_getter(THPCppFunction *self, void *_unused) {
6117:   HANDLE_TH_ERRORS
6118:   const auto& prop = static_cast<FlashAttentionBackward0*>(self->cdata.get())->softmax_logsumexp_;
6119:   return THPVariable_Wrap(prop.unpack(self->cdata));
6120:   END_HANDLE_TH_ERRORS
6121: }
6122: 
6123: static PyObject* THPFlashAttentionBackward0_softmax_logsumexp_raw_getter(THPCppFunction *self, void *_unused) {
6124:   HANDLE_TH_ERRORS
6125:   const auto& prop = static_cast<FlashAttentionBackward0*>(self->cdata.get())->softmax_logsumexp_;
6126:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
6127:   return obj.release().ptr();
6128:   END_HANDLE_TH_ERRORS
6129: }
6130: 
6131: static PyObject* THPFlashAttentionBackward0_unused_getter(THPCppFunction *self, void *_unused) {
6132:   HANDLE_TH_ERRORS
6133:   const auto& prop = static_cast<FlashAttentionBackward0*>(self->cdata.get())->unused_;
6134:   return THPVariable_Wrap(prop.unpack(self->cdata));
6135:   END_HANDLE_TH_ERRORS
6136: }
6137: 
6138: static PyObject* THPFlashAttentionBackward0_unused_raw_getter(THPCppFunction *self, void *_unused) {
6139:   HANDLE_TH_ERRORS
6140:   const auto& prop = static_cast<FlashAttentionBackward0*>(self->cdata.get())->unused_;
6141:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
6142:   return obj.release().ptr();
6143:   END_HANDLE_TH_ERRORS
6144: }
6145: 
6146: static struct PyGetSetDef FlashAttentionBackward0_properties[] = {
6147:   THP_FUNCTION_DEFAULT_PROPERTIES,
6148:   {(char*)"_saved_cum_seq_k", (getter)THPFlashAttentionBackward0_cum_seq_k_getter, nullptr, nullptr, nullptr},
6149:   {(char*)"_raw_saved_cum_seq_k", (getter)THPFlashAttentionBackward0_cum_seq_k_raw_getter, nullptr, nullptr, nullptr},
6150:   {(char*)"_saved_cum_seq_q", (getter)THPFlashAttentionBackward0_cum_seq_q_getter, nullptr, nullptr, nullptr},
6151:   {(char*)"_raw_saved_cum_seq_q", (getter)THPFlashAttentionBackward0_cum_seq_q_raw_getter, nullptr, nullptr, nullptr},
6152:   {(char*)"_saved_dropout_p", (getter)THPFlashAttentionBackward0_dropout_p_getter, nullptr, nullptr, nullptr},
6153:   {(char*)"_saved_is_causal", (getter)THPFlashAttentionBackward0_is_causal_getter, nullptr, nullptr, nullptr},
6154:   {(char*)"_saved_key", (getter)THPFlashAttentionBackward0_key_getter, nullptr, nullptr, nullptr},
6155:   {(char*)"_raw_saved_key", (getter)THPFlashAttentionBackward0_key_raw_getter, nullptr, nullptr, nullptr},
6156:   {(char*)"_saved_max_k", (getter)THPFlashAttentionBackward0_max_k_getter, nullptr, nullptr, nullptr},
6157:   {(char*)"_saved_max_q", (getter)THPFlashAttentionBackward0_max_q_getter, nullptr, nullptr, nullptr},
6158:   {(char*)"_saved_query", (getter)THPFlashAttentionBackward0_query_getter, nullptr, nullptr, nullptr},
6159:   {(char*)"_raw_saved_query", (getter)THPFlashAttentionBackward0_query_raw_getter, nullptr, nullptr, nullptr},
6160:   {(char*)"_saved_scale", (getter)THPFlashAttentionBackward0_scale_getter, nullptr, nullptr, nullptr},
```

- EN: The main execution path in this span is carried by `cast`, `THPFlashAttentionBackward0_output_getter`, `THPVariable_Wrap`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `cast`, `THPFlashAttentionBackward0_output_getter`, `THPVariable_Wrap` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 6161-6240

```cpp
6161:   {(char*)"_saved_value", (getter)THPFlashAttentionBackward0_value_getter, nullptr, nullptr, nullptr},
6162:   {(char*)"_raw_saved_value", (getter)THPFlashAttentionBackward0_value_raw_getter, nullptr, nullptr, nullptr},
6163:   {(char*)"_saved_window_size_left", (getter)THPFlashAttentionBackward0_window_size_left_getter, nullptr, nullptr, nullptr},
6164:   {(char*)"_saved_window_size_right", (getter)THPFlashAttentionBackward0_window_size_right_getter, nullptr, nullptr, nullptr},
6165:   {(char*)"_saved_output", (getter)THPFlashAttentionBackward0_output_getter, nullptr, nullptr, nullptr},
6166:   {(char*)"_raw_saved_output", (getter)THPFlashAttentionBackward0_output_raw_getter, nullptr, nullptr, nullptr},
6167:   {(char*)"_saved_rng_state", (getter)THPFlashAttentionBackward0_rng_state_getter, nullptr, nullptr, nullptr},
6168:   {(char*)"_raw_saved_rng_state", (getter)THPFlashAttentionBackward0_rng_state_raw_getter, nullptr, nullptr, nullptr},
6169:   {(char*)"_saved_softmax_logsumexp", (getter)THPFlashAttentionBackward0_softmax_logsumexp_getter, nullptr, nullptr, nullptr},
6170:   {(char*)"_raw_saved_softmax_logsumexp", (getter)THPFlashAttentionBackward0_softmax_logsumexp_raw_getter, nullptr, nullptr, nullptr},
6171:   {(char*)"_saved_unused", (getter)THPFlashAttentionBackward0_unused_getter, nullptr, nullptr, nullptr},
6172:   {(char*)"_raw_saved_unused", (getter)THPFlashAttentionBackward0_unused_raw_getter, nullptr, nullptr, nullptr},
6173:   {nullptr} /* sentinel */
6174: };
6175: 
6176: static PyObject* THPUnbindBackward0_dim_getter(THPCppFunction *self, void *_unused) {
6177:   HANDLE_TH_ERRORS
6178:   auto prop = static_cast<UnbindBackward0*>(self->cdata.get())->dim;
6179:   return PyLong_FromUnsignedLong((int64_t) prop);
6180:   END_HANDLE_TH_ERRORS
6181: }
6182: 
6183: static struct PyGetSetDef UnbindBackward0_properties[] = {
6184:   THP_FUNCTION_DEFAULT_PROPERTIES,
6185:   {(char*)"_saved_dim", (getter)THPUnbindBackward0_dim_getter, nullptr, nullptr, nullptr},
6186:   {nullptr} /* sentinel */
6187: };
6188: 
6189: static PyObject* THPUnbindBackwardAutogradNestedTensor0_dim_getter(THPCppFunction *self, void *_unused) {
6190:   HANDLE_TH_ERRORS
6191:   auto prop = static_cast<UnbindBackwardAutogradNestedTensor0*>(self->cdata.get())->dim;
6192:   return PyLong_FromUnsignedLong((int64_t) prop);
6193:   END_HANDLE_TH_ERRORS
6194: }
6195: 
6196: static PyObject* THPUnbindBackwardAutogradNestedTensor0_self_getter(THPCppFunction *self, void *_unused) {
6197:   HANDLE_TH_ERRORS
6198:   const auto& prop = static_cast<UnbindBackwardAutogradNestedTensor0*>(self->cdata.get())->self_;
6199:   return THPVariable_Wrap(prop.unpack(self->cdata));
6200:   END_HANDLE_TH_ERRORS
6201: }
6202: 
6203: static PyObject* THPUnbindBackwardAutogradNestedTensor0_self_raw_getter(THPCppFunction *self, void *_unused) {
6204:   HANDLE_TH_ERRORS
6205:   const auto& prop = static_cast<UnbindBackwardAutogradNestedTensor0*>(self->cdata.get())->self_;
6206:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
6207:   return obj.release().ptr();
6208:   END_HANDLE_TH_ERRORS
6209: }
6210: 
6211: static struct PyGetSetDef UnbindBackwardAutogradNestedTensor0_properties[] = {
6212:   THP_FUNCTION_DEFAULT_PROPERTIES,
6213:   {(char*)"_saved_dim", (getter)THPUnbindBackwardAutogradNestedTensor0_dim_getter, nullptr, nullptr, nullptr},
6214:   {(char*)"_saved_self", (getter)THPUnbindBackwardAutogradNestedTensor0_self_getter, nullptr, nullptr, nullptr},
6215:   {(char*)"_raw_saved_self", (getter)THPUnbindBackwardAutogradNestedTensor0_self_raw_getter, nullptr, nullptr, nullptr},
6216:   {nullptr} /* sentinel */
6217: };
6218: 
6219: static PyObject* THPSegmentReduceBackward0_axis_getter(THPCppFunction *self, void *_unused) {
6220:   HANDLE_TH_ERRORS
6221:   auto prop = static_cast<SegmentReduceBackward0*>(self->cdata.get())->axis;
6222:   return PyLong_FromUnsignedLong((int64_t) prop);
6223:   END_HANDLE_TH_ERRORS
6224: }
6225: 
6226: static PyObject* THPSegmentReduceBackward0_data_getter(THPCppFunction *self, void *_unused) {
6227:   HANDLE_TH_ERRORS
6228:   const auto& prop = static_cast<SegmentReduceBackward0*>(self->cdata.get())->data_;
6229:   return THPVariable_Wrap(prop.unpack(self->cdata));
6230:   END_HANDLE_TH_ERRORS
6231: }
6232: 
6233: static PyObject* THPSegmentReduceBackward0_data_raw_getter(THPCppFunction *self, void *_unused) {
6234:   HANDLE_TH_ERRORS
6235:   const auto& prop = static_cast<SegmentReduceBackward0*>(self->cdata.get())->data_;
6236:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
6237:   return obj.release().ptr();
6238:   END_HANDLE_TH_ERRORS
6239: }
6240: 
```

- EN: The main execution path in this span is carried by `THPUnbindBackward0_dim_getter`, `PyLong_FromUnsignedLong`, `THPUnbindBackwardAutogradNestedTensor0_dim_getter`. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPUnbindBackward0_dim_getter`, `PyLong_FromUnsignedLong`, `THPUnbindBackwardAutogradNestedTensor0_dim_getter` 等函数/方法承载。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 6241-6320

```cpp
6241: static PyObject* THPSegmentReduceBackward0_initial_getter(THPCppFunction *self, void *_unused) {
6242:   HANDLE_TH_ERRORS
6243:   auto opt_prop = static_cast<SegmentReduceBackward0*>(self->cdata.get())->initial;
6244:   if (!opt_prop.has_value()) {
6245:     Py_RETURN_NONE;
6246:   }
6247:   auto prop = opt_prop.value();
6248:   if (prop.isComplex()) {
6249:     auto cprop = prop.to<c10::complex<double>>();
6250:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
6251:   } else if (prop.isFloatingPoint()) {
6252:     return PyFloat_FromDouble(prop.to<double>());
6253:   } else if (prop.isIntegral(/*includeBool=*/false)) {
6254:     return PyLong_FromLong(prop.to<int64_t>());
6255:   } else if (prop.isBoolean()) {
6256:     if (prop.to<bool>()) {
6257:       Py_RETURN_TRUE;
6258:     } else {
6259:       Py_RETURN_FALSE;
6260:     }
6261:   } else {
6262:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
6263:     return nullptr;
6264:   }
6265:   END_HANDLE_TH_ERRORS
6266: }
6267: 
6268: static PyObject* THPSegmentReduceBackward0_lengths_getter(THPCppFunction *self, void *_unused) {
6269:   HANDLE_TH_ERRORS
6270:   const auto& prop = static_cast<SegmentReduceBackward0*>(self->cdata.get())->lengths_;
6271:   return THPVariable_Wrap(prop.unpack(self->cdata));
6272:   END_HANDLE_TH_ERRORS
6273: }
6274: 
6275: static PyObject* THPSegmentReduceBackward0_lengths_raw_getter(THPCppFunction *self, void *_unused) {
6276:   HANDLE_TH_ERRORS
6277:   const auto& prop = static_cast<SegmentReduceBackward0*>(self->cdata.get())->lengths_;
6278:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
6279:   return obj.release().ptr();
6280:   END_HANDLE_TH_ERRORS
6281: }
6282: 
6283: static PyObject* THPSegmentReduceBackward0_offsets_getter(THPCppFunction *self, void *_unused) {
6284:   HANDLE_TH_ERRORS
6285:   const auto& prop = static_cast<SegmentReduceBackward0*>(self->cdata.get())->offsets_;
6286:   return THPVariable_Wrap(prop.unpack(self->cdata));
6287:   END_HANDLE_TH_ERRORS
6288: }
6289: 
6290: static PyObject* THPSegmentReduceBackward0_offsets_raw_getter(THPCppFunction *self, void *_unused) {
6291:   HANDLE_TH_ERRORS
6292:   const auto& prop = static_cast<SegmentReduceBackward0*>(self->cdata.get())->offsets_;
6293:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
6294:   return obj.release().ptr();
6295:   END_HANDLE_TH_ERRORS
6296: }
6297: 
6298: static PyObject* THPSegmentReduceBackward0_reduce_getter(THPCppFunction *self, void *_unused) {
6299:   HANDLE_TH_ERRORS
6300:   auto prop = static_cast<SegmentReduceBackward0*>(self->cdata.get())->reduce;
6301:   return PyUnicode_FromStringAndSize(prop.data(), prop.size());
6302:   END_HANDLE_TH_ERRORS
6303: }
6304: 
6305: static PyObject* THPSegmentReduceBackward0_result_getter(THPCppFunction *self, void *_unused) {
6306:   HANDLE_TH_ERRORS
6307:   const auto& prop = static_cast<SegmentReduceBackward0*>(self->cdata.get())->result_;
6308:   return THPVariable_Wrap(prop.unpack(self->cdata));
6309:   END_HANDLE_TH_ERRORS
6310: }
6311: 
6312: static PyObject* THPSegmentReduceBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
6313:   HANDLE_TH_ERRORS
6314:   const auto& prop = static_cast<SegmentReduceBackward0*>(self->cdata.get())->result_;
6315:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
6316:   return obj.release().ptr();
6317:   END_HANDLE_TH_ERRORS
6318: }
6319: 
6320: static struct PyGetSetDef SegmentReduceBackward0_properties[] = {
```

- EN: The main execution path in this span is carried by `THPSegmentReduceBackward0_initial_getter`, `PyComplex_FromDoubles`, `PyFloat_FromDouble`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPSegmentReduceBackward0_initial_getter`, `PyComplex_FromDoubles`, `PyFloat_FromDouble` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 6321-6400

```cpp
6321:   THP_FUNCTION_DEFAULT_PROPERTIES,
6322:   {(char*)"_saved_axis", (getter)THPSegmentReduceBackward0_axis_getter, nullptr, nullptr, nullptr},
6323:   {(char*)"_saved_data", (getter)THPSegmentReduceBackward0_data_getter, nullptr, nullptr, nullptr},
6324:   {(char*)"_raw_saved_data", (getter)THPSegmentReduceBackward0_data_raw_getter, nullptr, nullptr, nullptr},
6325:   {(char*)"_saved_initial", (getter)THPSegmentReduceBackward0_initial_getter, nullptr, nullptr, nullptr},
6326:   {(char*)"_saved_lengths", (getter)THPSegmentReduceBackward0_lengths_getter, nullptr, nullptr, nullptr},
6327:   {(char*)"_raw_saved_lengths", (getter)THPSegmentReduceBackward0_lengths_raw_getter, nullptr, nullptr, nullptr},
6328:   {(char*)"_saved_offsets", (getter)THPSegmentReduceBackward0_offsets_getter, nullptr, nullptr, nullptr},
6329:   {(char*)"_raw_saved_offsets", (getter)THPSegmentReduceBackward0_offsets_raw_getter, nullptr, nullptr, nullptr},
6330:   {(char*)"_saved_reduce", (getter)THPSegmentReduceBackward0_reduce_getter, nullptr, nullptr, nullptr},
6331:   {(char*)"_saved_result", (getter)THPSegmentReduceBackward0_result_getter, nullptr, nullptr, nullptr},
6332:   {(char*)"_raw_saved_result", (getter)THPSegmentReduceBackward0_result_raw_getter, nullptr, nullptr, nullptr},
6333:   {nullptr} /* sentinel */
6334: };
6335: 
6336: 
6337: 
6338: static struct PyGetSetDef PinMemoryBackward0_properties[] = {
6339:   THP_FUNCTION_DEFAULT_PROPERTIES,
6340: 
6341:   {nullptr} /* sentinel */
6342: };
6343: 
6344: static PyObject* THPTestAutogradMultipleDispatchBackward0_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
6345:   HANDLE_TH_ERRORS
6346:   auto prop = static_cast<TestAutogradMultipleDispatchBackward0*>(self->cdata.get())->self_sym_sizes;
6347:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
6348:   for (auto i : c10::irange(prop.size())) {
6349:       auto si = prop[i];
6350:       if (auto m = si.maybe_as_int()) {
6351:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
6352:       } else {
6353:         auto py_symint = py::cast(si).release().ptr();
6354:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
6355:       }
6356:   }
6357:   return tup;
6358:   END_HANDLE_TH_ERRORS
6359: }
6360: 
6361: static struct PyGetSetDef TestAutogradMultipleDispatchBackward0_properties[] = {
6362:   THP_FUNCTION_DEFAULT_PROPERTIES,
6363:   {(char*)"_saved_self_sym_sizes", (getter)THPTestAutogradMultipleDispatchBackward0_self_sym_sizes_getter, nullptr, nullptr, nullptr},
6364:   {nullptr} /* sentinel */
6365: };
6366: 
6367: 
6368: 
6369: static struct PyGetSetDef TestAutogradMultipleDispatchBackwardAutogradNestedTensor0_properties[] = {
6370:   THP_FUNCTION_DEFAULT_PROPERTIES,
6371: 
6372:   {nullptr} /* sentinel */
6373: };
6374: 
6375: static PyObject* THPTestAutogradMultipleDispatchBackwardAutogradCUDA0_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
6376:   HANDLE_TH_ERRORS
6377:   auto prop = static_cast<TestAutogradMultipleDispatchBackwardAutogradCUDA0*>(self->cdata.get())->self_sym_sizes;
6378:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
6379:   for (auto i : c10::irange(prop.size())) {
6380:       auto si = prop[i];
6381:       if (auto m = si.maybe_as_int()) {
6382:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
6383:       } else {
6384:         auto py_symint = py::cast(si).release().ptr();
6385:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
6386:       }
6387:   }
6388:   return tup;
6389:   END_HANDLE_TH_ERRORS
6390: }
6391: 
6392: static struct PyGetSetDef TestAutogradMultipleDispatchBackwardAutogradCUDA0_properties[] = {
6393:   THP_FUNCTION_DEFAULT_PROPERTIES,
6394:   {(char*)"_saved_self_sym_sizes", (getter)THPTestAutogradMultipleDispatchBackwardAutogradCUDA0_self_sym_sizes_getter, nullptr, nullptr, nullptr},
6395:   {nullptr} /* sentinel */
6396: };
6397: 
6398: 
6399: 
6400: static struct PyGetSetDef TestAutogradMultipleDispatchBackwardAutogradNestedTensor1_properties[] = {
```

- EN: The main execution path in this span is carried by `THPTestAutogradMultipleDispatchBackward0_self_sym_sizes_getter`, `PyTuple_New`, `PyTuple_SetItem`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPTestAutogradMultipleDispatchBackward0_self_sym_sizes_getter`, `PyTuple_New`, `PyTuple_SetItem` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 6401-6480

```cpp
6401:   THP_FUNCTION_DEFAULT_PROPERTIES,
6402: 
6403:   {nullptr} /* sentinel */
6404: };
6405: 
6406: static PyObject* THPForeachMinimumBackward0_scalar_getter(THPCppFunction *self, void *_unused) {
6407:   HANDLE_TH_ERRORS
6408:   auto prop = static_cast<ForeachMinimumBackward0*>(self->cdata.get())->scalar;
6409:   if (prop.isComplex()) {
6410:     auto cprop = prop.to<c10::complex<double>>();
6411:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
6412:   } else if (prop.isFloatingPoint()) {
6413:     return PyFloat_FromDouble(prop.to<double>());
6414:   } else if (prop.isIntegral(/*includeBool=*/false)) {
6415:     return PyLong_FromLong(prop.to<int64_t>());
6416:   } else if (prop.isBoolean()) {
6417:     if (prop.to<bool>()) {
6418:       Py_RETURN_TRUE;
6419:     } else {
6420:       Py_RETURN_FALSE;
6421:     }
6422:   } else {
6423:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
6424:     return nullptr;
6425:   }
6426:   END_HANDLE_TH_ERRORS
6427: }
6428: 
6429: static PyObject* THPForeachMinimumBackward0_self_getter(THPCppFunction *self, void *_unused) {
6430:   HANDLE_TH_ERRORS
6431:   const auto *node = static_cast<ForeachMinimumBackward0*>(self->cdata.get());
6432:   const auto& prop = node->self_;
6433:   if (node->self_released_) {
6434:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
6435:     return nullptr;
6436:   }
6437:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
6438:   for (auto i: c10::irange(prop.size())) {
6439:     PyTuple_SetItem(tup, (Py_ssize_t) i, THPVariable_Wrap(prop[i].unpack(self->cdata)));
6440:   }
6441:   return tup;
6442:   END_HANDLE_TH_ERRORS
6443: }
6444: 
6445: static PyObject* THPForeachMinimumBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
6446:   HANDLE_TH_ERRORS
6447:   const auto *node = static_cast<ForeachMinimumBackward0*>(self->cdata.get());
6448:   const auto& prop = node->self_;
6449:   if (node->self_released_) {
6450:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
6451:     return nullptr;
6452:   }
6453:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
6454:   for (auto i : c10::irange(prop.size())) {
6455:     pybind11::object obj = pybind11::cast(prop[i], pybind11::return_value_policy::reference);
6456:     PyTuple_SetItem(tup, (Py_ssize_t) i, obj.release().ptr());
6457:   }
6458:   return tup;
6459:   END_HANDLE_TH_ERRORS
6460: }
6461: 
6462: static struct PyGetSetDef ForeachMinimumBackward0_properties[] = {
6463:   THP_FUNCTION_DEFAULT_PROPERTIES,
6464:   {(char*)"_saved_scalar", (getter)THPForeachMinimumBackward0_scalar_getter, nullptr, nullptr, nullptr},
6465:   {(char*)"_saved_self", (getter)THPForeachMinimumBackward0_self_getter, nullptr, nullptr, nullptr},
6466:   {(char*)"_raw_saved_self", (getter)THPForeachMinimumBackward0_self_raw_getter, nullptr, nullptr, nullptr},
6467:   {nullptr} /* sentinel */
6468: };
6469: 
6470: static PyObject* THPForeachMinimumBackward1_scalars_getter(THPCppFunction *self, void *_unused) {
6471:   HANDLE_TH_ERRORS
6472:   const auto *node = static_cast<ForeachMinimumBackward1*>(self->cdata.get());
6473:   const auto& prop = node->scalars;
6474:   if (node->scalars_released_) {
6475:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
6476:     return nullptr;
6477:   }
6478:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
6479:   for (auto i: c10::irange(prop.size())) {
6480:     if (prop[i].isComplex()) {
```

- EN: The main execution path in this span is carried by `THPForeachMinimumBackward0_scalar_getter`, `PyComplex_FromDoubles`, `PyFloat_FromDouble`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPForeachMinimumBackward0_scalar_getter`, `PyComplex_FromDoubles`, `PyFloat_FromDouble` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 6481-6560

```cpp
6481:       auto cprop = prop[i].to<c10::complex<double>>();
6482:       PyTuple_SetItem(tup, (Py_ssize_t) i, PyComplex_FromDoubles(cprop.real(), cprop.imag()));
6483:     } else if (prop[i].isFloatingPoint()) {
6484:       auto double_prop = prop[i].to<double>();
6485:       PyTuple_SetItem(tup, (Py_ssize_t) i, PyFloat_FromDouble(double_prop));
6486:     } else if (prop[i].isIntegral(/*includeBool=*/false)) {
6487:       auto long_prop = prop[i].to<int64_t>();
6488:       PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromLong(long_prop));
6489:     } else if (prop[i].isBoolean()) {
6490:       if (prop[i].to<bool>()) {
6491:         PyTuple_SetItem(tup, (Py_ssize_t) i, Py_True);
6492:       } else {
6493:         PyTuple_SetItem(tup, (Py_ssize_t) i, Py_False);
6494:       }
6495:     } else {
6496:       PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
6497:       return nullptr;
6498:     }
6499:   }
6500:   return tup;
6501:   END_HANDLE_TH_ERRORS
6502: }
6503: 
6504: static PyObject* THPForeachMinimumBackward1_self_getter(THPCppFunction *self, void *_unused) {
6505:   HANDLE_TH_ERRORS
6506:   const auto *node = static_cast<ForeachMinimumBackward1*>(self->cdata.get());
6507:   const auto& prop = node->self_;
6508:   if (node->self_released_) {
6509:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
6510:     return nullptr;
6511:   }
6512:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
6513:   for (auto i: c10::irange(prop.size())) {
6514:     PyTuple_SetItem(tup, (Py_ssize_t) i, THPVariable_Wrap(prop[i].unpack(self->cdata)));
6515:   }
6516:   return tup;
6517:   END_HANDLE_TH_ERRORS
6518: }
6519: 
6520: static PyObject* THPForeachMinimumBackward1_self_raw_getter(THPCppFunction *self, void *_unused) {
6521:   HANDLE_TH_ERRORS
6522:   const auto *node = static_cast<ForeachMinimumBackward1*>(self->cdata.get());
6523:   const auto& prop = node->self_;
6524:   if (node->self_released_) {
6525:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
6526:     return nullptr;
6527:   }
6528:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
6529:   for (auto i : c10::irange(prop.size())) {
6530:     pybind11::object obj = pybind11::cast(prop[i], pybind11::return_value_policy::reference);
6531:     PyTuple_SetItem(tup, (Py_ssize_t) i, obj.release().ptr());
6532:   }
6533:   return tup;
6534:   END_HANDLE_TH_ERRORS
6535: }
6536: 
6537: static struct PyGetSetDef ForeachMinimumBackward1_properties[] = {
6538:   THP_FUNCTION_DEFAULT_PROPERTIES,
6539:   {(char*)"_saved_scalars", (getter)THPForeachMinimumBackward1_scalars_getter, nullptr, nullptr, nullptr},
6540:   {(char*)"_saved_self", (getter)THPForeachMinimumBackward1_self_getter, nullptr, nullptr, nullptr},
6541:   {(char*)"_raw_saved_self", (getter)THPForeachMinimumBackward1_self_raw_getter, nullptr, nullptr, nullptr},
6542:   {nullptr} /* sentinel */
6543: };
6544: 
6545: static PyObject* THPAsStridedBackward0_copy_size_getter(THPCppFunction *self, void *_unused) {
6546:   HANDLE_TH_ERRORS
6547:   auto prop = static_cast<AsStridedBackward0_copy*>(self->cdata.get())->size;
6548:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
6549:   for (auto i : c10::irange(prop.size())) {
6550:       auto si = prop[i];
6551:       if (auto m = si.maybe_as_int()) {
6552:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
6553:       } else {
6554:         auto py_symint = py::cast(si).release().ptr();
6555:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
6556:       }
6557:   }
6558:   return tup;
6559:   END_HANDLE_TH_ERRORS
6560: }
```

- EN: The main execution path in this span is carried by `PyTuple_SetItem`, `PyErr_SetString`, `THPForeachMinimumBackward1_self_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyTuple_SetItem`, `PyErr_SetString`, `THPForeachMinimumBackward1_self_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 6561-6640

```cpp
6561: 
6562: static PyObject* THPAsStridedBackward0_copy_storage_offset_getter(THPCppFunction *self, void *_unused) {
6563:   HANDLE_TH_ERRORS
6564:   auto opt_prop = static_cast<AsStridedBackward0_copy*>(self->cdata.get())->storage_offset;
6565:   if (!opt_prop.has_value()) {
6566:     Py_RETURN_NONE;
6567:   }
6568:   auto prop = opt_prop.value();
6569:   if (auto m = prop.maybe_as_int()) {
6570:     return PyLong_FromUnsignedLong(*m);
6571:   } else {
6572:     return py::cast(prop).release().ptr();
6573:   }
6574:   END_HANDLE_TH_ERRORS
6575: }
6576: 
6577: static PyObject* THPAsStridedBackward0_copy_stride_getter(THPCppFunction *self, void *_unused) {
6578:   HANDLE_TH_ERRORS
6579:   auto prop = static_cast<AsStridedBackward0_copy*>(self->cdata.get())->stride;
6580:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
6581:   for (auto i : c10::irange(prop.size())) {
6582:       auto si = prop[i];
6583:       if (auto m = si.maybe_as_int()) {
6584:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
6585:       } else {
6586:         auto py_symint = py::cast(si).release().ptr();
6587:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
6588:       }
6589:   }
6590:   return tup;
6591:   END_HANDLE_TH_ERRORS
6592: }
6593: 
6594: static struct PyGetSetDef AsStridedBackward0_copy_properties[] = {
6595:   THP_FUNCTION_DEFAULT_PROPERTIES,
6596:   {(char*)"_saved_size", (getter)THPAsStridedBackward0_copy_size_getter, nullptr, nullptr, nullptr},
6597:   {(char*)"_saved_storage_offset", (getter)THPAsStridedBackward0_copy_storage_offset_getter, nullptr, nullptr, nullptr},
6598:   {(char*)"_saved_stride", (getter)THPAsStridedBackward0_copy_stride_getter, nullptr, nullptr, nullptr},
6599:   {nullptr} /* sentinel */
6600: };
6601: 
6602: 
6603: 
6604: static struct PyGetSetDef ConjBackward0_copy_properties[] = {
6605:   THP_FUNCTION_DEFAULT_PROPERTIES,
6606: 
6607:   {nullptr} /* sentinel */
6608: };
6609: 
6610: 
6611: 
6612: static struct PyGetSetDef NegViewBackward0_copy_properties[] = {
6613:   THP_FUNCTION_DEFAULT_PROPERTIES,
6614: 
6615:   {nullptr} /* sentinel */
6616: };
6617: 
6618: 
6619: 
6620: static struct PyGetSetDef ViewAsComplexBackward0_copy_properties[] = {
6621:   THP_FUNCTION_DEFAULT_PROPERTIES,
6622: 
6623:   {nullptr} /* sentinel */
6624: };
6625: 
6626: 
6627: 
6628: static struct PyGetSetDef NestedViewFromBufferBackward0_copy_properties[] = {
6629:   THP_FUNCTION_DEFAULT_PROPERTIES,
6630: 
6631:   {nullptr} /* sentinel */
6632: };
6633: 
6634: void initialize_autogenerated_functions_0(PyObject* module) {
6635:   static PyTypeObject AddcmulBackward0Class;
6636:   addClass<AddcmulBackward0>(module, AddcmulBackward0Class, "AddcmulBackward0", AddcmulBackward0_properties);
6637:   static PyTypeObject AddmmBackward0Class;
6638:   addClass<AddmmBackward0>(module, AddmmBackward0Class, "AddmmBackward0", AddmmBackward0_properties);
6639:   static PyTypeObject SparseAddmmBackward0Class;
6640:   addClass<SparseAddmmBackward0>(module, SparseAddmmBackward0Class, "SparseAddmmBackward0", SparseAddmmBackward0_properties);
```

- EN: The main execution path in this span is carried by `THPAsStridedBackward0_copy_storage_offset_getter`, `PyLong_FromUnsignedLong`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPAsStridedBackward0_copy_storage_offset_getter`, `PyLong_FromUnsignedLong`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 6641-6720

```cpp
6641:   static PyTypeObject AcoshBackward0Class;
6642:   addClass<AcoshBackward0>(module, AcoshBackward0Class, "AcoshBackward0", AcoshBackward0_properties);
6643:   static PyTypeObject AtanhBackward1Class;
6644:   addClass<AtanhBackward1>(module, AtanhBackward1Class, "AtanhBackward1", AtanhBackward1_properties);
6645:   static PyTypeObject AsStridedBackward0Class;
6646:   addClass<AsStridedBackward0>(module, AsStridedBackward0Class, "AsStridedBackward0", AsStridedBackward0_properties);
6647:   static PyTypeObject CatBackward0Class;
6648:   addClass<CatBackward0>(module, CatBackward0Class, "CatBackward0", CatBackward0_properties);
6649:   static PyTypeObject CholeskyBackward0Class;
6650:   addClass<CholeskyBackward0>(module, CholeskyBackward0Class, "CholeskyBackward0", CholeskyBackward0_properties);
6651:   static PyTypeObject ToCopyBackward0Class;
6652:   addClass<ToCopyBackward0>(module, ToCopyBackward0Class, "ToCopyBackward0", ToCopyBackward0_properties);
6653:   static PyTypeObject ConjBackward0Class;
6654:   addClass<ConjBackward0>(module, ConjBackward0Class, "ConjBackward0", ConjBackward0_properties);
6655:   static PyTypeObject NegViewBackward0Class;
6656:   addClass<NegViewBackward0>(module, NegViewBackward0Class, "NegViewBackward0", NegViewBackward0_properties);
6657:   static PyTypeObject ConjPhysicalBackward0Class;
6658:   addClass<ConjPhysicalBackward0>(module, ConjPhysicalBackward0Class, "ConjPhysicalBackward0", ConjPhysicalBackward0_properties);
6659:   static PyTypeObject CosBackward0Class;
6660:   addClass<CosBackward0>(module, CosBackward0Class, "CosBackward0", CosBackward0_properties);
6661:   static PyTypeObject CumsumBackward0Class;
6662:   addClass<CumsumBackward0>(module, CumsumBackward0Class, "CumsumBackward0", CumsumBackward0_properties);
6663:   static PyTypeObject CtcLossBackward0Class;
6664:   addClass<CtcLossBackward0>(module, CtcLossBackward0Class, "CtcLossBackward0", CtcLossBackward0_properties);
6665:   static PyTypeObject CtcLossBackward1Class;
6666:   addClass<CtcLossBackward1>(module, CtcLossBackward1Class, "CtcLossBackward1", CtcLossBackward1_properties);
6667:   static PyTypeObject VdotBackward0Class;
6668:   addClass<VdotBackward0>(module, VdotBackward0Class, "VdotBackward0", VdotBackward0_properties);
6669:   static PyTypeObject FusedDropoutBackward0Class;
6670:   addClass<FusedDropoutBackward0>(module, FusedDropoutBackward0Class, "FusedDropoutBackward0", FusedDropoutBackward0_properties);
6671:   static PyTypeObject Expm1Backward0Class;
6672:   addClass<Expm1Backward0>(module, Expm1Backward0Class, "Expm1Backward0", Expm1Backward0_properties);
6673:   static PyTypeObject ExponentialBackward0Class;
6674:   addClass<ExponentialBackward0>(module, ExponentialBackward0Class, "ExponentialBackward0", ExponentialBackward0_properties);
6675:   static PyTypeObject FakeQuantizePerChannelAffineCachemaskBackward0Class;
6676:   addClass<FakeQuantizePerChannelAffineCachemaskBackward0>(module, FakeQuantizePerChannelAffineCachemaskBackward0Class, "FakeQuantizePerChannelAffineCachemaskBackward0", FakeQuantizePerChannelAffineCachemaskBackward0_properties);
6677:   static PyTypeObject FusedMovingAvgObsFqHelperBackward0Class;
6678:   addClass<FusedMovingAvgObsFqHelperBackward0>(module, FusedMovingAvgObsFqHelperBackward0Class, "FusedMovingAvgObsFqHelperBackward0", FusedMovingAvgObsFqHelperBackward0_properties);
6679:   static PyTypeObject FillBackward0Class;
6680:   addClass<FillBackward0>(module, FillBackward0Class, "FillBackward0", FillBackward0_properties);
6681:   static PyTypeObject FillBackward1Class;
6682:   addClass<FillBackward1>(module, FillBackward1Class, "FillBackward1", FillBackward1_properties);
6683:   static PyTypeObject GeometricBackward0Class;
6684:   addClass<GeometricBackward0>(module, GeometricBackward0Class, "GeometricBackward0", GeometricBackward0_properties);
6685:   static PyTypeObject GridSampler2DBackwardBackward0Class;
6686:   addClass<GridSampler2DBackwardBackward0>(module, GridSampler2DBackwardBackward0Class, "GridSampler2DBackwardBackward0", GridSampler2DBackwardBackward0_properties);
6687:   static PyTypeObject SpecialI1EBackward0Class;
6688:   addClass<SpecialI1EBackward0>(module, SpecialI1EBackward0Class, "SpecialI1EBackward0", SpecialI1EBackward0_properties);
6689:   static PyTypeObject IgammaBackward0Class;
6690:   addClass<IgammaBackward0>(module, IgammaBackward0Class, "IgammaBackward0", IgammaBackward0_properties);
6691:   static PyTypeObject IndexCopyBackward0Class;
6692:   addClass<IndexCopyBackward0>(module, IndexCopyBackward0Class, "IndexCopyBackward0", IndexCopyBackward0_properties);
6693:   static PyTypeObject UnsafeIndexPutBackward0Class;
6694:   addClass<UnsafeIndexPutBackward0>(module, UnsafeIndexPutBackward0Class, "UnsafeIndexPutBackward0", UnsafeIndexPutBackward0_properties);
6695:   static PyTypeObject IndexSelectBackward0Class;
6696:   addClass<IndexSelectBackward0>(module, IndexSelectBackward0Class, "IndexSelectBackward0", IndexSelectBackward0_properties);
6697:   static PyTypeObject LeBackward0Class;
6698:   addClass<LeBackward0>(module, LeBackward0Class, "LeBackward0", LeBackward0_properties);
6699:   static PyTypeObject LeBackward1Class;
6700:   addClass<LeBackward1>(module, LeBackward1Class, "LeBackward1", LeBackward1_properties);
6701:   static PyTypeObject LgammaBackward0Class;
6702:   addClass<LgammaBackward0>(module, LgammaBackward0Class, "LgammaBackward0", LgammaBackward0_properties);
6703:   static PyTypeObject Log10Backward0Class;
6704:   addClass<Log10Backward0>(module, Log10Backward0Class, "Log10Backward0", Log10Backward0_properties);
6705:   static PyTypeObject Log1PBackward0Class;
6706:   addClass<Log1PBackward0>(module, Log1PBackward0Class, "Log1PBackward0", Log1PBackward0_properties);
6707:   static PyTypeObject Logaddexp2Backward0Class;
6708:   addClass<Logaddexp2Backward0>(module, Logaddexp2Backward0Class, "Logaddexp2Backward0", Logaddexp2Backward0_properties);
6709:   static PyTypeObject LogsumexpBackward0Class;
6710:   addClass<LogsumexpBackward0>(module, LogsumexpBackward0Class, "LogsumexpBackward0", LogsumexpBackward0_properties);
6711:   static PyTypeObject LinalgLstsqBackward0Class;
6712:   addClass<LinalgLstsqBackward0>(module, LinalgLstsqBackward0Class, "LinalgLstsqBackward0", LinalgLstsqBackward0_properties);
6713:   static PyTypeObject LinalgLuBackward0Class;
6714:   addClass<LinalgLuBackward0>(module, LinalgLuBackward0Class, "LinalgLuBackward0", LinalgLuBackward0_properties);
6715:   static PyTypeObject LuUnpackBackward0Class;
6716:   addClass<LuUnpackBackward0>(module, LuUnpackBackward0Class, "LuUnpackBackward0", LuUnpackBackward0_properties);
6717:   static PyTypeObject LinalgMatrixExpBackward0Class;
6718:   addClass<LinalgMatrixExpBackward0>(module, LinalgMatrixExpBackward0Class, "LinalgMatrixExpBackward0", LinalgMatrixExpBackward0_properties);
6719:   static PyTypeObject MaxBackward0Class;
6720:   addClass<MaxBackward0>(module, MaxBackward0Class, "MaxBackward0", MaxBackward0_properties);
```

- EN: This range continues the surrounding implementation through local state updates, helper invocations, and data movement.
- CN: 这一段通过局部状态更新、辅助函数调用与数据传递来延续周围实现。
### Lines 6721-6800

```cpp
6721:   static PyTypeObject MaxBackward1Class;
6722:   addClass<MaxBackward1>(module, MaxBackward1Class, "MaxBackward1", MaxBackward1_properties);
6723:   static PyTypeObject AminmaxBackward0Class;
6724:   addClass<AminmaxBackward0>(module, AminmaxBackward0Class, "AminmaxBackward0", AminmaxBackward0_properties);
6725:   static PyTypeObject ModeBackward0Class;
6726:   addClass<ModeBackward0>(module, ModeBackward0Class, "ModeBackward0", ModeBackward0_properties);
6727:   static PyTypeObject MulBackward0Class;
6728:   addClass<MulBackward0>(module, MulBackward0Class, "MulBackward0", MulBackward0_properties);
6729:   static PyTypeObject MulBackward1Class;
6730:   addClass<MulBackward1>(module, MulBackward1Class, "MulBackward1", MulBackward1_properties);
6731:   static PyTypeObject MvlgammaBackward0Class;
6732:   addClass<MvlgammaBackward0>(module, MvlgammaBackward0Class, "MvlgammaBackward0", MvlgammaBackward0_properties);
6733:   static PyTypeObject NegBackward0Class;
6734:   addClass<NegBackward0>(module, NegBackward0Class, "NegBackward0", NegBackward0_properties);
6735:   static PyTypeObject NormalBackward0Class;
6736:   addClass<NormalBackward0>(module, NormalBackward0Class, "NormalBackward0", NormalBackward0_properties);
6737:   static PyTypeObject PowBackward0Class;
6738:   addClass<PowBackward0>(module, PowBackward0Class, "PowBackward0", PowBackward0_properties);
6739:   static PyTypeObject PowBackward1Class;
6740:   addClass<PowBackward1>(module, PowBackward1Class, "PowBackward1", PowBackward1_properties);
6741:   static PyTypeObject PowBackward2Class;
6742:   addClass<PowBackward2>(module, PowBackward2Class, "PowBackward2", PowBackward2_properties);
6743:   static PyTypeObject ReciprocalBackward0Class;
6744:   addClass<ReciprocalBackward0>(module, ReciprocalBackward0Class, "ReciprocalBackward0", ReciprocalBackward0_properties);
6745:   static PyTypeObject SpecialEntrBackward0Class;
6746:   addClass<SpecialEntrBackward0>(module, SpecialEntrBackward0Class, "SpecialEntrBackward0", SpecialEntrBackward0_properties);
6747:   static PyTypeObject SpecialNdtriBackward0Class;
6748:   addClass<SpecialNdtriBackward0>(module, SpecialNdtriBackward0Class, "SpecialNdtriBackward0", SpecialNdtriBackward0_properties);
6749:   static PyTypeObject SgnBackward0Class;
6750:   addClass<SgnBackward0>(module, SgnBackward0Class, "SgnBackward0", SgnBackward0_properties);
6751:   static PyTypeObject SinBackward0Class;
6752:   addClass<SinBackward0>(module, SinBackward0Class, "SinBackward0", SinBackward0_properties);
6753:   static PyTypeObject DiagonalScatterBackward0Class;
6754:   addClass<DiagonalScatterBackward0>(module, DiagonalScatterBackward0Class, "DiagonalScatterBackward0", DiagonalScatterBackward0_properties);
6755:   static PyTypeObject AsStridedScatterBackward0Class;
6756:   addClass<AsStridedScatterBackward0>(module, AsStridedScatterBackward0Class, "AsStridedScatterBackward0", AsStridedScatterBackward0_properties);
6757:   static PyTypeObject SplitBackward0Class;
6758:   addClass<SplitBackward0>(module, SplitBackward0Class, "SplitBackward0", SplitBackward0_properties);
6759:   static PyTypeObject SqueezeBackward0Class;
6760:   addClass<SqueezeBackward0>(module, SqueezeBackward0Class, "SqueezeBackward0", SqueezeBackward0_properties);
6761:   static PyTypeObject SqueezeBackward1Class;
6762:   addClass<SqueezeBackward1>(module, SqueezeBackward1Class, "SqueezeBackward1", SqueezeBackward1_properties);
6763:   static PyTypeObject SqueezeBackwardAutogradNestedTensor0Class;
6764:   addClass<SqueezeBackwardAutogradNestedTensor0>(module, SqueezeBackwardAutogradNestedTensor0Class, "SqueezeBackwardAutogradNestedTensor0", SqueezeBackwardAutogradNestedTensor0_properties);
6765:   static PyTypeObject SqueezeBackward2Class;
6766:   addClass<SqueezeBackward2>(module, SqueezeBackward2Class, "SqueezeBackward2", SqueezeBackward2_properties);
6767:   static PyTypeObject SqueezeBackwardAutogradNestedTensor1Class;
6768:   addClass<SqueezeBackwardAutogradNestedTensor1>(module, SqueezeBackwardAutogradNestedTensor1Class, "SqueezeBackwardAutogradNestedTensor1", SqueezeBackwardAutogradNestedTensor1_properties);
6769:   static PyTypeObject LinalgEighBackward0Class;
6770:   addClass<LinalgEighBackward0>(module, LinalgEighBackward0Class, "LinalgEighBackward0", LinalgEighBackward0_properties);
6771:   static PyTypeObject ToMkldnnBackward0Class;
6772:   addClass<ToMkldnnBackward0>(module, ToMkldnnBackward0Class, "ToMkldnnBackward0", ToMkldnnBackward0_properties);
6773:   static PyTypeObject UniqueBackward0Class;
6774:   addClass<UniqueBackward0>(module, UniqueBackward0Class, "UniqueBackward0", UniqueBackward0_properties);
6775:   static PyTypeObject LiftBackward0Class;
6776:   addClass<LiftBackward0>(module, LiftBackward0Class, "LiftBackward0", LiftBackward0_properties);
6777:   static PyTypeObject LiftFreshBackward0Class;
6778:   addClass<LiftFreshBackward0>(module, LiftFreshBackward0Class, "LiftFreshBackward0", LiftFreshBackward0_properties);
6779:   static PyTypeObject WhereBackward0Class;
6780:   addClass<WhereBackward0>(module, WhereBackward0Class, "WhereBackward0", WhereBackward0_properties);
6781:   static PyTypeObject WeightNormInterfaceBackward0Class;
6782:   addClass<WeightNormInterfaceBackward0>(module, WeightNormInterfaceBackward0Class, "WeightNormInterfaceBackward0", WeightNormInterfaceBackward0_properties);
6783:   static PyTypeObject ZeroBackward0Class;
6784:   addClass<ZeroBackward0>(module, ZeroBackward0Class, "ZeroBackward0", ZeroBackward0_properties);
6785:   static PyTypeObject SparseMaskBackward0Class;
6786:   addClass<SparseMaskBackward0>(module, SparseMaskBackward0Class, "SparseMaskBackward0", SparseMaskBackward0_properties);
6787:   static PyTypeObject SparseSumBackward0Class;
6788:   addClass<SparseSumBackward0>(module, SparseSumBackward0Class, "SparseSumBackward0", SparseSumBackward0_properties);
6789:   static PyTypeObject StandardGammaBackward0Class;
6790:   addClass<StandardGammaBackward0>(module, StandardGammaBackward0Class, "StandardGammaBackward0", StandardGammaBackward0_properties);
6791:   static PyTypeObject ValuesBackward0Class;
6792:   addClass<ValuesBackward0>(module, ValuesBackward0Class, "ValuesBackward0", ValuesBackward0_properties);
6793:   static PyTypeObject ValuesBackwardAutogradNestedTensor0Class;
6794:   addClass<ValuesBackwardAutogradNestedTensor0>(module, ValuesBackwardAutogradNestedTensor0Class, "ValuesBackwardAutogradNestedTensor0", ValuesBackwardAutogradNestedTensor0_properties);
6795:   static PyTypeObject EmbeddingDenseBackwardBackward0Class;
6796:   addClass<EmbeddingDenseBackwardBackward0>(module, EmbeddingDenseBackwardBackward0Class, "EmbeddingDenseBackwardBackward0", EmbeddingDenseBackwardBackward0_properties);
6797:   static PyTypeObject EmbeddingBagBackward0Class;
6798:   addClass<EmbeddingBagBackward0>(module, EmbeddingBagBackward0Class, "EmbeddingBagBackward0", EmbeddingBagBackward0_properties);
6799:   static PyTypeObject GeluBackwardBackward0Class;
6800:   addClass<GeluBackwardBackward0>(module, GeluBackwardBackward0Class, "GeluBackwardBackward0", GeluBackwardBackward0_properties);
```

- EN: Autograd-related metadata or graph connectivity is updated here.
- CN: 这里会更新与自动求导相关的元数据或计算图连接关系。
### Lines 6801-6880

```cpp
6801:   static PyTypeObject LogSoftmaxBackward0Class;
6802:   addClass<LogSoftmaxBackward0>(module, LogSoftmaxBackward0Class, "LogSoftmaxBackward0", LogSoftmaxBackward0_properties);
6803:   static PyTypeObject PreluKernelBackwardBackward0Class;
6804:   addClass<PreluKernelBackwardBackward0>(module, PreluKernelBackwardBackward0Class, "PreluKernelBackwardBackward0", PreluKernelBackwardBackward0_properties);
6805:   static PyTypeObject ReflectionPad1DBackward0Class;
6806:   addClass<ReflectionPad1DBackward0>(module, ReflectionPad1DBackward0Class, "ReflectionPad1DBackward0", ReflectionPad1DBackward0_properties);
6807:   static PyTypeObject UpsampleBicubic2DAaBackward0Class;
6808:   addClass<UpsampleBicubic2DAaBackward0>(module, UpsampleBicubic2DAaBackward0Class, "UpsampleBicubic2DAaBackward0", UpsampleBicubic2DAaBackward0_properties);
6809:   static PyTypeObject UpsampleLanczos2DAaBackward0Class;
6810:   addClass<UpsampleLanczos2DAaBackward0>(module, UpsampleLanczos2DAaBackward0Class, "UpsampleLanczos2DAaBackward0", UpsampleLanczos2DAaBackward0_properties);
6811:   static PyTypeObject UpsampleTrilinear3DBackward0Class;
6812:   addClass<UpsampleTrilinear3DBackward0>(module, UpsampleTrilinear3DBackward0Class, "UpsampleTrilinear3DBackward0", UpsampleTrilinear3DBackward0_properties);
6813:   static PyTypeObject UpsampleNearest3DBackward0Class;
6814:   addClass<UpsampleNearest3DBackward0>(module, UpsampleNearest3DBackward0Class, "UpsampleNearest3DBackward0", UpsampleNearest3DBackward0_properties);
6815:   static PyTypeObject PixelUnshuffleBackward0Class;
6816:   addClass<PixelUnshuffleBackward0>(module, PixelUnshuffleBackward0Class, "PixelUnshuffleBackward0", PixelUnshuffleBackward0_properties);
6817:   static PyTypeObject LinearBackward0Class;
6818:   addClass<LinearBackward0>(module, LinearBackward0Class, "LinearBackward0", LinearBackward0_properties);
6819:   static PyTypeObject MaxPool2DWithIndicesBackward0Class;
6820:   addClass<MaxPool2DWithIndicesBackward0>(module, MaxPool2DWithIndicesBackward0Class, "MaxPool2DWithIndicesBackward0", MaxPool2DWithIndicesBackward0_properties);
6821:   static PyTypeObject ConvolutionBackward0Class;
6822:   addClass<ConvolutionBackward0>(module, ConvolutionBackward0Class, "ConvolutionBackward0", ConvolutionBackward0_properties);
6823:   static PyTypeObject ConvolutionBackward1Class;
6824:   addClass<ConvolutionBackward1>(module, ConvolutionBackward1Class, "ConvolutionBackward1", ConvolutionBackward1_properties);
6825:   static PyTypeObject ConvolutionBackwardOverrideableBackward0Class;
6826:   addClass<ConvolutionBackwardOverrideableBackward0>(module, ConvolutionBackwardOverrideableBackward0Class, "ConvolutionBackwardOverrideableBackward0", ConvolutionBackwardOverrideableBackward0_properties);
6827:   static PyTypeObject SlowConv2DBackward0Class;
6828:   addClass<SlowConv2DBackward0>(module, SlowConv2DBackward0Class, "SlowConv2DBackward0", SlowConv2DBackward0_properties);
6829:   static PyTypeObject EluBackwardBackward0Class;
6830:   addClass<EluBackwardBackward0>(module, EluBackwardBackward0Class, "EluBackwardBackward0", EluBackwardBackward0_properties);
6831:   static PyTypeObject FractionalMaxPool3DBackwardBackward0Class;
6832:   addClass<FractionalMaxPool3DBackwardBackward0>(module, FractionalMaxPool3DBackwardBackward0Class, "FractionalMaxPool3DBackwardBackward0", FractionalMaxPool3DBackwardBackward0_properties);
6833:   static PyTypeObject MseLossBackwardBackward0Class;
6834:   addClass<MseLossBackwardBackward0>(module, MseLossBackwardBackward0Class, "MseLossBackwardBackward0", MseLossBackwardBackward0_properties);
6835:   static PyTypeObject RreluWithNoiseBackwardBackward0Class;
6836:   addClass<RreluWithNoiseBackwardBackward0>(module, RreluWithNoiseBackwardBackward0Class, "RreluWithNoiseBackwardBackward0", RreluWithNoiseBackwardBackward0_properties);
6837:   static PyTypeObject SoftplusBackwardBackward0Class;
6838:   addClass<SoftplusBackwardBackward0>(module, SoftplusBackwardBackward0Class, "SoftplusBackwardBackward0", SoftplusBackwardBackward0_properties);
6839:   static PyTypeObject ThresholdBackwardBackward0Class;
6840:   addClass<ThresholdBackwardBackward0>(module, ThresholdBackwardBackward0Class, "ThresholdBackwardBackward0", ThresholdBackwardBackward0_properties);
6841:   static PyTypeObject UpsampleBilinear2DAaBackwardBackward0Class;
6842:   addClass<UpsampleBilinear2DAaBackwardBackward0>(module, UpsampleBilinear2DAaBackwardBackward0Class, "UpsampleBilinear2DAaBackwardBackward0", UpsampleBilinear2DAaBackwardBackward0_properties);
6843:   static PyTypeObject SigmoidBackwardBackward0Class;
6844:   addClass<SigmoidBackwardBackward0>(module, SigmoidBackwardBackward0Class, "SigmoidBackwardBackward0", SigmoidBackwardBackward0_properties);
6845:   static PyTypeObject TanhBackwardBackward0Class;
6846:   addClass<TanhBackwardBackward0>(module, TanhBackwardBackward0Class, "TanhBackwardBackward0", TanhBackwardBackward0_properties);
6847:   static PyTypeObject CudnnCtcLossBackward0Class;
6848:   addClass<CudnnCtcLossBackward0>(module, CudnnCtcLossBackward0Class, "CudnnCtcLossBackward0", CudnnCtcLossBackward0_properties);
6849:   static PyTypeObject CudnnCtcLossBackward1Class;
6850:   addClass<CudnnCtcLossBackward1>(module, CudnnCtcLossBackward1Class, "CudnnCtcLossBackward1", CudnnCtcLossBackward1_properties);
6851:   static PyTypeObject CudnnConvolutionTransposeBackward0Class;
6852:   addClass<CudnnConvolutionTransposeBackward0>(module, CudnnConvolutionTransposeBackward0Class, "CudnnConvolutionTransposeBackward0", CudnnConvolutionTransposeBackward0_properties);
6853:   static PyTypeObject CudnnGridSamplerBackwardBackward0Class;
6854:   addClass<CudnnGridSamplerBackwardBackward0>(module, CudnnGridSamplerBackwardBackward0Class, "CudnnGridSamplerBackwardBackward0", CudnnGridSamplerBackwardBackward0_properties);
6855:   static PyTypeObject CudnnAffineGridGeneratorBackward0Class;
6856:   addClass<CudnnAffineGridGeneratorBackward0>(module, CudnnAffineGridGeneratorBackward0Class, "CudnnAffineGridGeneratorBackward0", CudnnAffineGridGeneratorBackward0_properties);
6857:   static PyTypeObject CudnnBatchNormBackwardBackward0Class;
6858:   addClass<CudnnBatchNormBackwardBackward0>(module, CudnnBatchNormBackwardBackward0Class, "CudnnBatchNormBackwardBackward0", CudnnBatchNormBackwardBackward0_properties);
6859:   static PyTypeObject CudnnRnnBackward0Class;
6860:   addClass<CudnnRnnBackward0>(module, CudnnRnnBackward0Class, "CudnnRnnBackward0", CudnnRnnBackward0_properties);
6861:   static PyTypeObject MiopenBatchNormBackwardBackward0Class;
6862:   addClass<MiopenBatchNormBackwardBackward0>(module, MiopenBatchNormBackwardBackward0Class, "MiopenBatchNormBackwardBackward0", MiopenBatchNormBackwardBackward0_properties);
6863:   static PyTypeObject MkldnnConvolutionBackward0Class;
6864:   addClass<MkldnnConvolutionBackward0>(module, MkldnnConvolutionBackward0Class, "MkldnnConvolutionBackward0", MkldnnConvolutionBackward0_properties);
6865:   static PyTypeObject MkldnnLinearBackward0Class;
6866:   addClass<MkldnnLinearBackward0>(module, MkldnnLinearBackward0Class, "MkldnnLinearBackward0", MkldnnLinearBackward0_properties);
6867:   static PyTypeObject ToPaddedTensorBackward0Class;
6868:   addClass<ToPaddedTensorBackward0>(module, ToPaddedTensorBackward0Class, "ToPaddedTensorBackward0", ToPaddedTensorBackward0_properties);
6869:   static PyTypeObject FlashAttentionBackward0Class;
6870:   addClass<FlashAttentionBackward0>(module, FlashAttentionBackward0Class, "FlashAttentionBackward0", FlashAttentionBackward0_properties);
6871:   static PyTypeObject UnbindBackward0Class;
6872:   addClass<UnbindBackward0>(module, UnbindBackward0Class, "UnbindBackward0", UnbindBackward0_properties);
6873:   static PyTypeObject UnbindBackwardAutogradNestedTensor0Class;
6874:   addClass<UnbindBackwardAutogradNestedTensor0>(module, UnbindBackwardAutogradNestedTensor0Class, "UnbindBackwardAutogradNestedTensor0", UnbindBackwardAutogradNestedTensor0_properties);
6875:   static PyTypeObject SegmentReduceBackward0Class;
6876:   addClass<SegmentReduceBackward0>(module, SegmentReduceBackward0Class, "SegmentReduceBackward0", SegmentReduceBackward0_properties);
6877:   static PyTypeObject PinMemoryBackward0Class;
6878:   addClass<PinMemoryBackward0>(module, PinMemoryBackward0Class, "PinMemoryBackward0", PinMemoryBackward0_properties);
6879:   static PyTypeObject TestAutogradMultipleDispatchBackward0Class;
6880:   addClass<TestAutogradMultipleDispatchBackward0>(module, TestAutogradMultipleDispatchBackward0Class, "TestAutogradMultipleDispatchBackward0", TestAutogradMultipleDispatchBackward0_properties);
```

- EN: Autograd-related metadata or graph connectivity is updated here.
- CN: 这里会更新与自动求导相关的元数据或计算图连接关系。
### Lines 6881-6903

```cpp
6881:   static PyTypeObject TestAutogradMultipleDispatchBackwardAutogradNestedTensor0Class;
6882:   addClass<TestAutogradMultipleDispatchBackwardAutogradNestedTensor0>(module, TestAutogradMultipleDispatchBackwardAutogradNestedTensor0Class, "TestAutogradMultipleDispatchBackwardAutogradNestedTensor0", TestAutogradMultipleDispatchBackwardAutogradNestedTensor0_properties);
6883:   static PyTypeObject TestAutogradMultipleDispatchBackwardAutogradCUDA0Class;
6884:   addClass<TestAutogradMultipleDispatchBackwardAutogradCUDA0>(module, TestAutogradMultipleDispatchBackwardAutogradCUDA0Class, "TestAutogradMultipleDispatchBackwardAutogradCUDA0", TestAutogradMultipleDispatchBackwardAutogradCUDA0_properties);
6885:   static PyTypeObject TestAutogradMultipleDispatchBackwardAutogradNestedTensor1Class;
6886:   addClass<TestAutogradMultipleDispatchBackwardAutogradNestedTensor1>(module, TestAutogradMultipleDispatchBackwardAutogradNestedTensor1Class, "TestAutogradMultipleDispatchBackwardAutogradNestedTensor1", TestAutogradMultipleDispatchBackwardAutogradNestedTensor1_properties);
6887:   static PyTypeObject ForeachMinimumBackward0Class;
6888:   addClass<ForeachMinimumBackward0>(module, ForeachMinimumBackward0Class, "ForeachMinimumBackward0", ForeachMinimumBackward0_properties);
6889:   static PyTypeObject ForeachMinimumBackward1Class;
6890:   addClass<ForeachMinimumBackward1>(module, ForeachMinimumBackward1Class, "ForeachMinimumBackward1", ForeachMinimumBackward1_properties);
6891:   static PyTypeObject AsStridedBackward0_copyClass;
6892:   addClass<AsStridedBackward0_copy>(module, AsStridedBackward0_copyClass, "AsStridedBackward0_copy", AsStridedBackward0_copy_properties);
6893:   static PyTypeObject ConjBackward0_copyClass;
6894:   addClass<ConjBackward0_copy>(module, ConjBackward0_copyClass, "ConjBackward0_copy", ConjBackward0_copy_properties);
6895:   static PyTypeObject NegViewBackward0_copyClass;
6896:   addClass<NegViewBackward0_copy>(module, NegViewBackward0_copyClass, "NegViewBackward0_copy", NegViewBackward0_copy_properties);
6897:   static PyTypeObject ViewAsComplexBackward0_copyClass;
6898:   addClass<ViewAsComplexBackward0_copy>(module, ViewAsComplexBackward0_copyClass, "ViewAsComplexBackward0_copy", ViewAsComplexBackward0_copy_properties);
6899:   static PyTypeObject NestedViewFromBufferBackward0_copyClass;
6900:   addClass<NestedViewFromBufferBackward0_copy>(module, NestedViewFromBufferBackward0_copyClass, "NestedViewFromBufferBackward0_copy", NestedViewFromBufferBackward0_copy_properties);
6901: }
6902: 
6903: } // namespace torch::autograd::generated
```

- EN: They also open, refine, or close the relevant C++ namespace scope. Autograd-related metadata or graph connectivity is updated here.
- CN: 它们还会打开、细化或关闭相应的 C++ 命名空间作用域。 这里会更新与自动求导相关的元数据或计算图连接关系。

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
- Key symbols / 关键符号: `addClass`, `THPAddcmulBackward0_tensor1_getter`, `THPVariable_Wrap`, `THPAddcmulBackward0_tensor1_raw_getter`, `THPAddcmulBackward0_tensor2_getter`, `THPAddcmulBackward0_tensor2_raw_getter`, `THPAddcmulBackward0_value_getter`, `PyComplex_FromDoubles`, `PyFloat_FromDouble`, `PyLong_FromLong`
- Related subsystems / 相关子系统: Autograd / 自动求导, ATen operator layer / ATen 算子层, c10 core utilities / c10 核心工具, Python binding layer / Python 绑定层
