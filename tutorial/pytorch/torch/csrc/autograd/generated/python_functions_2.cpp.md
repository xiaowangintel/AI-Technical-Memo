# python_functions_2.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/autograd/generated/python_functions_2.cpp`
- Repository: `/root/xw/pytorch`
- Purpose (EN): Defines generated backward-function classes and helpers used by the autograd engine.
- 目的 (CN): 定义自动求导引擎使用的生成式反向函数类与辅助逻辑。
- Lines: 7087
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
31: static PyObject* THPAddBackward0_alpha_getter(THPCppFunction *self, void *_unused) {
32:   HANDLE_TH_ERRORS
33:   auto prop = static_cast<AddBackward0*>(self->cdata.get())->alpha;
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
54: static struct PyGetSetDef AddBackward0_properties[] = {
55:   THP_FUNCTION_DEFAULT_PROPERTIES,
56:   {(char*)"_saved_alpha", (getter)THPAddBackward0_alpha_getter, nullptr, nullptr, nullptr},
57:   {nullptr} /* sentinel */
58: };
59: 
60: 
61: 
62: static struct PyGetSetDef AddBackward1_properties[] = {
63:   THP_FUNCTION_DEFAULT_PROPERTIES,
64: 
65:   {nullptr} /* sentinel */
66: };
67: 
68: static PyObject* THPAddcdivBackward0_tensor1_getter(THPCppFunction *self, void *_unused) {
69:   HANDLE_TH_ERRORS
70:   const auto& prop = static_cast<AddcdivBackward0*>(self->cdata.get())->tensor1_;
71:   return THPVariable_Wrap(prop.unpack(self->cdata));
72:   END_HANDLE_TH_ERRORS
73: }
74: 
75: static PyObject* THPAddcdivBackward0_tensor1_raw_getter(THPCppFunction *self, void *_unused) {
76:   HANDLE_TH_ERRORS
77:   const auto& prop = static_cast<AddcdivBackward0*>(self->cdata.get())->tensor1_;
78:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
79:   return obj.release().ptr();
80:   END_HANDLE_TH_ERRORS
```

- EN: These lines pull in dependencies such as `torch/csrc/autograd/generated/python_functions.h`, `Python.h`, `ATen/ATen.h`, establishing the headers needed by the implementation. They also open, refine, or close the relevant C++ namespace scope. The main execution path in this span is carried by `addClass`, `_initFunctionPyTypeObject`, `Py_INCREF`. Because this is generated binding code, the span mostly registers or forwards APIs into a mechanically produced Python-facing surface.
- CN: 这些行引入了依赖，例如 `torch/csrc/autograd/generated/python_functions.h`, `Python.h`, `ATen/ATen.h`，为后续实现建立所需的头文件基础。 它们还会打开、细化或关闭相应的 C++ 命名空间作用域。 这一段的主要执行路径由 `addClass`, `_initFunctionPyTypeObject`, `Py_INCREF` 等函数/方法承载。 由于这是生成的绑定代码，这一段主要是在机械化生成的 Python 接口层上完成 API 注册或转发。
### Lines 81-160

```cpp
 81: }
 82: 
 83: static PyObject* THPAddcdivBackward0_tensor2_getter(THPCppFunction *self, void *_unused) {
 84:   HANDLE_TH_ERRORS
 85:   const auto& prop = static_cast<AddcdivBackward0*>(self->cdata.get())->tensor2_;
 86:   return THPVariable_Wrap(prop.unpack(self->cdata));
 87:   END_HANDLE_TH_ERRORS
 88: }
 89: 
 90: static PyObject* THPAddcdivBackward0_tensor2_raw_getter(THPCppFunction *self, void *_unused) {
 91:   HANDLE_TH_ERRORS
 92:   const auto& prop = static_cast<AddcdivBackward0*>(self->cdata.get())->tensor2_;
 93:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
 94:   return obj.release().ptr();
 95:   END_HANDLE_TH_ERRORS
 96: }
 97: 
 98: static PyObject* THPAddcdivBackward0_value_getter(THPCppFunction *self, void *_unused) {
 99:   HANDLE_TH_ERRORS
100:   auto prop = static_cast<AddcdivBackward0*>(self->cdata.get())->value;
101:   if (prop.isComplex()) {
102:     auto cprop = prop.to<c10::complex<double>>();
103:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
104:   } else if (prop.isFloatingPoint()) {
105:     return PyFloat_FromDouble(prop.to<double>());
106:   } else if (prop.isIntegral(/*includeBool=*/false)) {
107:     return PyLong_FromLong(prop.to<int64_t>());
108:   } else if (prop.isBoolean()) {
109:     if (prop.to<bool>()) {
110:       Py_RETURN_TRUE;
111:     } else {
112:       Py_RETURN_FALSE;
113:     }
114:   } else {
115:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
116:     return nullptr;
117:   }
118:   END_HANDLE_TH_ERRORS
119: }
120: 
121: static struct PyGetSetDef AddcdivBackward0_properties[] = {
122:   THP_FUNCTION_DEFAULT_PROPERTIES,
123:   {(char*)"_saved_tensor1", (getter)THPAddcdivBackward0_tensor1_getter, nullptr, nullptr, nullptr},
124:   {(char*)"_raw_saved_tensor1", (getter)THPAddcdivBackward0_tensor1_raw_getter, nullptr, nullptr, nullptr},
125:   {(char*)"_saved_tensor2", (getter)THPAddcdivBackward0_tensor2_getter, nullptr, nullptr, nullptr},
126:   {(char*)"_raw_saved_tensor2", (getter)THPAddcdivBackward0_tensor2_raw_getter, nullptr, nullptr, nullptr},
127:   {(char*)"_saved_value", (getter)THPAddcdivBackward0_value_getter, nullptr, nullptr, nullptr},
128:   {nullptr} /* sentinel */
129: };
130: 
131: static PyObject* THPAngleBackward0_self_getter(THPCppFunction *self, void *_unused) {
132:   HANDLE_TH_ERRORS
133:   const auto& prop = static_cast<AngleBackward0*>(self->cdata.get())->self_;
134:   return THPVariable_Wrap(prop.unpack(self->cdata));
135:   END_HANDLE_TH_ERRORS
136: }
137: 
138: static PyObject* THPAngleBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
139:   HANDLE_TH_ERRORS
140:   const auto& prop = static_cast<AngleBackward0*>(self->cdata.get())->self_;
141:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
142:   return obj.release().ptr();
143:   END_HANDLE_TH_ERRORS
144: }
145: 
146: static struct PyGetSetDef AngleBackward0_properties[] = {
147:   THP_FUNCTION_DEFAULT_PROPERTIES,
148:   {(char*)"_saved_self", (getter)THPAngleBackward0_self_getter, nullptr, nullptr, nullptr},
149:   {(char*)"_raw_saved_self", (getter)THPAngleBackward0_self_raw_getter, nullptr, nullptr, nullptr},
150:   {nullptr} /* sentinel */
151: };
152: 
153: 
154: 
155: static struct PyGetSetDef AcoshBackward1_properties[] = {
156:   THP_FUNCTION_DEFAULT_PROPERTIES,
157: 
158:   {nullptr} /* sentinel */
159: };
160: 
```

- EN: The main execution path in this span is carried by `THPAddcdivBackward0_tensor2_getter`, `THPVariable_Wrap`, `THPAddcdivBackward0_tensor2_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPAddcdivBackward0_tensor2_getter`, `THPVariable_Wrap`, `THPAddcdivBackward0_tensor2_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 161-240

```cpp
161: static PyObject* THPAsinhBackward0_self_getter(THPCppFunction *self, void *_unused) {
162:   HANDLE_TH_ERRORS
163:   const auto& prop = static_cast<AsinhBackward0*>(self->cdata.get())->self_;
164:   return THPVariable_Wrap(prop.unpack(self->cdata));
165:   END_HANDLE_TH_ERRORS
166: }
167: 
168: static PyObject* THPAsinhBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
169:   HANDLE_TH_ERRORS
170:   const auto& prop = static_cast<AsinhBackward0*>(self->cdata.get())->self_;
171:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
172:   return obj.release().ptr();
173:   END_HANDLE_TH_ERRORS
174: }
175: 
176: static struct PyGetSetDef AsinhBackward0_properties[] = {
177:   THP_FUNCTION_DEFAULT_PROPERTIES,
178:   {(char*)"_saved_self", (getter)THPAsinhBackward0_self_getter, nullptr, nullptr, nullptr},
179:   {(char*)"_raw_saved_self", (getter)THPAsinhBackward0_self_raw_getter, nullptr, nullptr, nullptr},
180:   {nullptr} /* sentinel */
181: };
182: 
183: 
184: 
185: static struct PyGetSetDef AsinhBackward1_properties[] = {
186:   THP_FUNCTION_DEFAULT_PROPERTIES,
187: 
188:   {nullptr} /* sentinel */
189: };
190: 
191: static PyObject* THPAsStridedBackward1_size_getter(THPCppFunction *self, void *_unused) {
192:   HANDLE_TH_ERRORS
193:   auto prop = static_cast<AsStridedBackward1*>(self->cdata.get())->size;
194:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
195:   for (auto i : c10::irange(prop.size())) {
196:       auto si = prop[i];
197:       if (auto m = si.maybe_as_int()) {
198:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
199:       } else {
200:         auto py_symint = py::cast(si).release().ptr();
201:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
202:       }
203:   }
204:   return tup;
205:   END_HANDLE_TH_ERRORS
206: }
207: 
208: static PyObject* THPAsStridedBackward1_storage_offset_getter(THPCppFunction *self, void *_unused) {
209:   HANDLE_TH_ERRORS
210:   auto opt_prop = static_cast<AsStridedBackward1*>(self->cdata.get())->storage_offset;
211:   if (!opt_prop.has_value()) {
212:     Py_RETURN_NONE;
213:   }
214:   auto prop = opt_prop.value();
215:   if (auto m = prop.maybe_as_int()) {
216:     return PyLong_FromUnsignedLong(*m);
217:   } else {
218:     return py::cast(prop).release().ptr();
219:   }
220:   END_HANDLE_TH_ERRORS
221: }
222: 
223: static PyObject* THPAsStridedBackward1_stride_getter(THPCppFunction *self, void *_unused) {
224:   HANDLE_TH_ERRORS
225:   auto prop = static_cast<AsStridedBackward1*>(self->cdata.get())->stride;
226:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
227:   for (auto i : c10::irange(prop.size())) {
228:       auto si = prop[i];
229:       if (auto m = si.maybe_as_int()) {
230:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
231:       } else {
232:         auto py_symint = py::cast(si).release().ptr();
233:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
234:       }
235:   }
236:   return tup;
237:   END_HANDLE_TH_ERRORS
238: }
239: 
240: static struct PyGetSetDef AsStridedBackward1_properties[] = {
```

- EN: The main execution path in this span is carried by `THPAsinhBackward0_self_getter`, `THPVariable_Wrap`, `THPAsinhBackward0_self_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPAsinhBackward0_self_getter`, `THPVariable_Wrap`, `THPAsinhBackward0_self_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 241-320

```cpp
241:   THP_FUNCTION_DEFAULT_PROPERTIES,
242:   {(char*)"_saved_size", (getter)THPAsStridedBackward1_size_getter, nullptr, nullptr, nullptr},
243:   {(char*)"_saved_storage_offset", (getter)THPAsStridedBackward1_storage_offset_getter, nullptr, nullptr, nullptr},
244:   {(char*)"_saved_stride", (getter)THPAsStridedBackward1_stride_getter, nullptr, nullptr, nullptr},
245:   {nullptr} /* sentinel */
246: };
247: 
248: static PyObject* THPAtan2Backward0_other_getter(THPCppFunction *self, void *_unused) {
249:   HANDLE_TH_ERRORS
250:   const auto& prop = static_cast<Atan2Backward0*>(self->cdata.get())->other_;
251:   return THPVariable_Wrap(prop.unpack(self->cdata));
252:   END_HANDLE_TH_ERRORS
253: }
254: 
255: static PyObject* THPAtan2Backward0_other_raw_getter(THPCppFunction *self, void *_unused) {
256:   HANDLE_TH_ERRORS
257:   const auto& prop = static_cast<Atan2Backward0*>(self->cdata.get())->other_;
258:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
259:   return obj.release().ptr();
260:   END_HANDLE_TH_ERRORS
261: }
262: 
263: static PyObject* THPAtan2Backward0_self_getter(THPCppFunction *self, void *_unused) {
264:   HANDLE_TH_ERRORS
265:   const auto& prop = static_cast<Atan2Backward0*>(self->cdata.get())->self_;
266:   return THPVariable_Wrap(prop.unpack(self->cdata));
267:   END_HANDLE_TH_ERRORS
268: }
269: 
270: static PyObject* THPAtan2Backward0_self_raw_getter(THPCppFunction *self, void *_unused) {
271:   HANDLE_TH_ERRORS
272:   const auto& prop = static_cast<Atan2Backward0*>(self->cdata.get())->self_;
273:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
274:   return obj.release().ptr();
275:   END_HANDLE_TH_ERRORS
276: }
277: 
278: static struct PyGetSetDef Atan2Backward0_properties[] = {
279:   THP_FUNCTION_DEFAULT_PROPERTIES,
280:   {(char*)"_saved_other", (getter)THPAtan2Backward0_other_getter, nullptr, nullptr, nullptr},
281:   {(char*)"_raw_saved_other", (getter)THPAtan2Backward0_other_raw_getter, nullptr, nullptr, nullptr},
282:   {(char*)"_saved_self", (getter)THPAtan2Backward0_self_getter, nullptr, nullptr, nullptr},
283:   {(char*)"_raw_saved_self", (getter)THPAtan2Backward0_self_raw_getter, nullptr, nullptr, nullptr},
284:   {nullptr} /* sentinel */
285: };
286: 
287: static PyObject* THPBaddbmmBackward0_alpha_getter(THPCppFunction *self, void *_unused) {
288:   HANDLE_TH_ERRORS
289:   auto prop = static_cast<BaddbmmBackward0*>(self->cdata.get())->alpha;
290:   if (prop.isComplex()) {
291:     auto cprop = prop.to<c10::complex<double>>();
292:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
293:   } else if (prop.isFloatingPoint()) {
294:     return PyFloat_FromDouble(prop.to<double>());
295:   } else if (prop.isIntegral(/*includeBool=*/false)) {
296:     return PyLong_FromLong(prop.to<int64_t>());
297:   } else if (prop.isBoolean()) {
298:     if (prop.to<bool>()) {
299:       Py_RETURN_TRUE;
300:     } else {
301:       Py_RETURN_FALSE;
302:     }
303:   } else {
304:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
305:     return nullptr;
306:   }
307:   END_HANDLE_TH_ERRORS
308: }
309: 
310: static PyObject* THPBaddbmmBackward0_batch1_getter(THPCppFunction *self, void *_unused) {
311:   HANDLE_TH_ERRORS
312:   const auto& prop = static_cast<BaddbmmBackward0*>(self->cdata.get())->batch1_;
313:   return THPVariable_Wrap(prop.unpack(self->cdata));
314:   END_HANDLE_TH_ERRORS
315: }
316: 
317: static PyObject* THPBaddbmmBackward0_batch1_raw_getter(THPCppFunction *self, void *_unused) {
318:   HANDLE_TH_ERRORS
319:   const auto& prop = static_cast<BaddbmmBackward0*>(self->cdata.get())->batch1_;
320:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
```

- EN: The main execution path in this span is carried by `THPAtan2Backward0_other_getter`, `THPVariable_Wrap`, `THPAtan2Backward0_other_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPAtan2Backward0_other_getter`, `THPVariable_Wrap`, `THPAtan2Backward0_other_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 321-400

```cpp
321:   return obj.release().ptr();
322:   END_HANDLE_TH_ERRORS
323: }
324: 
325: static PyObject* THPBaddbmmBackward0_batch2_getter(THPCppFunction *self, void *_unused) {
326:   HANDLE_TH_ERRORS
327:   const auto& prop = static_cast<BaddbmmBackward0*>(self->cdata.get())->batch2_;
328:   return THPVariable_Wrap(prop.unpack(self->cdata));
329:   END_HANDLE_TH_ERRORS
330: }
331: 
332: static PyObject* THPBaddbmmBackward0_batch2_raw_getter(THPCppFunction *self, void *_unused) {
333:   HANDLE_TH_ERRORS
334:   const auto& prop = static_cast<BaddbmmBackward0*>(self->cdata.get())->batch2_;
335:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
336:   return obj.release().ptr();
337:   END_HANDLE_TH_ERRORS
338: }
339: 
340: static PyObject* THPBaddbmmBackward0_beta_getter(THPCppFunction *self, void *_unused) {
341:   HANDLE_TH_ERRORS
342:   auto prop = static_cast<BaddbmmBackward0*>(self->cdata.get())->beta;
343:   if (prop.isComplex()) {
344:     auto cprop = prop.to<c10::complex<double>>();
345:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
346:   } else if (prop.isFloatingPoint()) {
347:     return PyFloat_FromDouble(prop.to<double>());
348:   } else if (prop.isIntegral(/*includeBool=*/false)) {
349:     return PyLong_FromLong(prop.to<int64_t>());
350:   } else if (prop.isBoolean()) {
351:     if (prop.to<bool>()) {
352:       Py_RETURN_TRUE;
353:     } else {
354:       Py_RETURN_FALSE;
355:     }
356:   } else {
357:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
358:     return nullptr;
359:   }
360:   END_HANDLE_TH_ERRORS
361: }
362: 
363: static struct PyGetSetDef BaddbmmBackward0_properties[] = {
364:   THP_FUNCTION_DEFAULT_PROPERTIES,
365:   {(char*)"_saved_alpha", (getter)THPBaddbmmBackward0_alpha_getter, nullptr, nullptr, nullptr},
366:   {(char*)"_saved_batch1", (getter)THPBaddbmmBackward0_batch1_getter, nullptr, nullptr, nullptr},
367:   {(char*)"_raw_saved_batch1", (getter)THPBaddbmmBackward0_batch1_raw_getter, nullptr, nullptr, nullptr},
368:   {(char*)"_saved_batch2", (getter)THPBaddbmmBackward0_batch2_getter, nullptr, nullptr, nullptr},
369:   {(char*)"_raw_saved_batch2", (getter)THPBaddbmmBackward0_batch2_raw_getter, nullptr, nullptr, nullptr},
370:   {(char*)"_saved_beta", (getter)THPBaddbmmBackward0_beta_getter, nullptr, nullptr, nullptr},
371:   {nullptr} /* sentinel */
372: };
373: 
374: 
375: 
376: static struct PyGetSetDef BernoulliBackward0_properties[] = {
377:   THP_FUNCTION_DEFAULT_PROPERTIES,
378: 
379:   {nullptr} /* sentinel */
380: };
381: 
382: 
383: 
384: static struct PyGetSetDef ChunkBackward0_properties[] = {
385:   THP_FUNCTION_DEFAULT_PROPERTIES,
386: 
387:   {nullptr} /* sentinel */
388: };
389: 
390: static PyObject* THPChunkBackwardAutogradNestedTensor0_chunks_getter(THPCppFunction *self, void *_unused) {
391:   HANDLE_TH_ERRORS
392:   auto prop = static_cast<ChunkBackwardAutogradNestedTensor0*>(self->cdata.get())->chunks;
393:   return PyLong_FromUnsignedLong((int64_t) prop);
394:   END_HANDLE_TH_ERRORS
395: }
396: 
397: static PyObject* THPChunkBackwardAutogradNestedTensor0_dim_getter(THPCppFunction *self, void *_unused) {
398:   HANDLE_TH_ERRORS
399:   auto prop = static_cast<ChunkBackwardAutogradNestedTensor0*>(self->cdata.get())->dim;
400:   return PyLong_FromUnsignedLong((int64_t) prop);
```

- EN: The main execution path in this span is carried by `THPBaddbmmBackward0_batch2_getter`, `THPVariable_Wrap`, `THPBaddbmmBackward0_batch2_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPBaddbmmBackward0_batch2_getter`, `THPVariable_Wrap`, `THPBaddbmmBackward0_batch2_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 401-480

```cpp
401:   END_HANDLE_TH_ERRORS
402: }
403: 
404: static PyObject* THPChunkBackwardAutogradNestedTensor0_self_getter(THPCppFunction *self, void *_unused) {
405:   HANDLE_TH_ERRORS
406:   const auto& prop = static_cast<ChunkBackwardAutogradNestedTensor0*>(self->cdata.get())->self_;
407:   return THPVariable_Wrap(prop.unpack(self->cdata));
408:   END_HANDLE_TH_ERRORS
409: }
410: 
411: static PyObject* THPChunkBackwardAutogradNestedTensor0_self_raw_getter(THPCppFunction *self, void *_unused) {
412:   HANDLE_TH_ERRORS
413:   const auto& prop = static_cast<ChunkBackwardAutogradNestedTensor0*>(self->cdata.get())->self_;
414:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
415:   return obj.release().ptr();
416:   END_HANDLE_TH_ERRORS
417: }
418: 
419: static struct PyGetSetDef ChunkBackwardAutogradNestedTensor0_properties[] = {
420:   THP_FUNCTION_DEFAULT_PROPERTIES,
421:   {(char*)"_saved_chunks", (getter)THPChunkBackwardAutogradNestedTensor0_chunks_getter, nullptr, nullptr, nullptr},
422:   {(char*)"_saved_dim", (getter)THPChunkBackwardAutogradNestedTensor0_dim_getter, nullptr, nullptr, nullptr},
423:   {(char*)"_saved_self", (getter)THPChunkBackwardAutogradNestedTensor0_self_getter, nullptr, nullptr, nullptr},
424:   {(char*)"_raw_saved_self", (getter)THPChunkBackwardAutogradNestedTensor0_self_raw_getter, nullptr, nullptr, nullptr},
425:   {nullptr} /* sentinel */
426: };
427: 
428: static PyObject* THPCholeskyInverseBackward0_self_getter(THPCppFunction *self, void *_unused) {
429:   HANDLE_TH_ERRORS
430:   const auto& prop = static_cast<CholeskyInverseBackward0*>(self->cdata.get())->self_;
431:   return THPVariable_Wrap(prop.unpack(self->cdata));
432:   END_HANDLE_TH_ERRORS
433: }
434: 
435: static PyObject* THPCholeskyInverseBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
436:   HANDLE_TH_ERRORS
437:   const auto& prop = static_cast<CholeskyInverseBackward0*>(self->cdata.get())->self_;
438:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
439:   return obj.release().ptr();
440:   END_HANDLE_TH_ERRORS
441: }
442: 
443: static PyObject* THPCholeskyInverseBackward0_upper_getter(THPCppFunction *self, void *_unused) {
444:   HANDLE_TH_ERRORS
445:   auto prop = static_cast<CholeskyInverseBackward0*>(self->cdata.get())->upper;
446:   if (prop) {
447:     Py_RETURN_TRUE;
448:   } else {
449:     Py_RETURN_FALSE;
450:   }
451:   END_HANDLE_TH_ERRORS
452: }
453: 
454: static PyObject* THPCholeskyInverseBackward0_result_getter(THPCppFunction *self, void *_unused) {
455:   HANDLE_TH_ERRORS
456:   const auto& prop = static_cast<CholeskyInverseBackward0*>(self->cdata.get())->result_;
457:   return THPVariable_Wrap(prop.unpack(self->cdata));
458:   END_HANDLE_TH_ERRORS
459: }
460: 
461: static PyObject* THPCholeskyInverseBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
462:   HANDLE_TH_ERRORS
463:   const auto& prop = static_cast<CholeskyInverseBackward0*>(self->cdata.get())->result_;
464:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
465:   return obj.release().ptr();
466:   END_HANDLE_TH_ERRORS
467: }
468: 
469: static struct PyGetSetDef CholeskyInverseBackward0_properties[] = {
470:   THP_FUNCTION_DEFAULT_PROPERTIES,
471:   {(char*)"_saved_self", (getter)THPCholeskyInverseBackward0_self_getter, nullptr, nullptr, nullptr},
472:   {(char*)"_raw_saved_self", (getter)THPCholeskyInverseBackward0_self_raw_getter, nullptr, nullptr, nullptr},
473:   {(char*)"_saved_upper", (getter)THPCholeskyInverseBackward0_upper_getter, nullptr, nullptr, nullptr},
474:   {(char*)"_saved_result", (getter)THPCholeskyInverseBackward0_result_getter, nullptr, nullptr, nullptr},
475:   {(char*)"_raw_saved_result", (getter)THPCholeskyInverseBackward0_result_raw_getter, nullptr, nullptr, nullptr},
476:   {nullptr} /* sentinel */
477: };
478: 
479: static PyObject* THPClampBackward0_max_getter(THPCppFunction *self, void *_unused) {
480:   HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPChunkBackwardAutogradNestedTensor0_self_getter`, `THPVariable_Wrap`, `THPChunkBackwardAutogradNestedTensor0_self_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPChunkBackwardAutogradNestedTensor0_self_getter`, `THPVariable_Wrap`, `THPChunkBackwardAutogradNestedTensor0_self_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 481-560

```cpp
481:   const auto& prop = static_cast<ClampBackward0*>(self->cdata.get())->max_;
482:   return THPVariable_Wrap(prop.unpack(self->cdata));
483:   END_HANDLE_TH_ERRORS
484: }
485: 
486: static PyObject* THPClampBackward0_max_raw_getter(THPCppFunction *self, void *_unused) {
487:   HANDLE_TH_ERRORS
488:   const auto& prop = static_cast<ClampBackward0*>(self->cdata.get())->max_;
489:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
490:   return obj.release().ptr();
491:   END_HANDLE_TH_ERRORS
492: }
493: 
494: static PyObject* THPClampBackward0_min_getter(THPCppFunction *self, void *_unused) {
495:   HANDLE_TH_ERRORS
496:   const auto& prop = static_cast<ClampBackward0*>(self->cdata.get())->min_;
497:   return THPVariable_Wrap(prop.unpack(self->cdata));
498:   END_HANDLE_TH_ERRORS
499: }
500: 
501: static PyObject* THPClampBackward0_min_raw_getter(THPCppFunction *self, void *_unused) {
502:   HANDLE_TH_ERRORS
503:   const auto& prop = static_cast<ClampBackward0*>(self->cdata.get())->min_;
504:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
505:   return obj.release().ptr();
506:   END_HANDLE_TH_ERRORS
507: }
508: 
509: static PyObject* THPClampBackward0_self_getter(THPCppFunction *self, void *_unused) {
510:   HANDLE_TH_ERRORS
511:   const auto& prop = static_cast<ClampBackward0*>(self->cdata.get())->self_;
512:   return THPVariable_Wrap(prop.unpack(self->cdata));
513:   END_HANDLE_TH_ERRORS
514: }
515: 
516: static PyObject* THPClampBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
517:   HANDLE_TH_ERRORS
518:   const auto& prop = static_cast<ClampBackward0*>(self->cdata.get())->self_;
519:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
520:   return obj.release().ptr();
521:   END_HANDLE_TH_ERRORS
522: }
523: 
524: static struct PyGetSetDef ClampBackward0_properties[] = {
525:   THP_FUNCTION_DEFAULT_PROPERTIES,
526:   {(char*)"_saved_max", (getter)THPClampBackward0_max_getter, nullptr, nullptr, nullptr},
527:   {(char*)"_raw_saved_max", (getter)THPClampBackward0_max_raw_getter, nullptr, nullptr, nullptr},
528:   {(char*)"_saved_min", (getter)THPClampBackward0_min_getter, nullptr, nullptr, nullptr},
529:   {(char*)"_raw_saved_min", (getter)THPClampBackward0_min_raw_getter, nullptr, nullptr, nullptr},
530:   {(char*)"_saved_self", (getter)THPClampBackward0_self_getter, nullptr, nullptr, nullptr},
531:   {(char*)"_raw_saved_self", (getter)THPClampBackward0_self_raw_getter, nullptr, nullptr, nullptr},
532:   {nullptr} /* sentinel */
533: };
534: 
535: static PyObject* THPClampBackward1_max_getter(THPCppFunction *self, void *_unused) {
536:   HANDLE_TH_ERRORS
537:   auto opt_prop = static_cast<ClampBackward1*>(self->cdata.get())->max;
538:   if (!opt_prop.has_value()) {
539:     Py_RETURN_NONE;
540:   }
541:   auto prop = opt_prop.value();
542:   if (prop.isComplex()) {
543:     auto cprop = prop.to<c10::complex<double>>();
544:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
545:   } else if (prop.isFloatingPoint()) {
546:     return PyFloat_FromDouble(prop.to<double>());
547:   } else if (prop.isIntegral(/*includeBool=*/false)) {
548:     return PyLong_FromLong(prop.to<int64_t>());
549:   } else if (prop.isBoolean()) {
550:     if (prop.to<bool>()) {
551:       Py_RETURN_TRUE;
552:     } else {
553:       Py_RETURN_FALSE;
554:     }
555:   } else {
556:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
557:     return nullptr;
558:   }
559:   END_HANDLE_TH_ERRORS
560: }
```

- EN: The main execution path in this span is carried by `THPVariable_Wrap`, `THPClampBackward0_max_raw_getter`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_Wrap`, `THPClampBackward0_max_raw_getter`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 561-640

```cpp
561: 
562: static PyObject* THPClampBackward1_min_getter(THPCppFunction *self, void *_unused) {
563:   HANDLE_TH_ERRORS
564:   auto opt_prop = static_cast<ClampBackward1*>(self->cdata.get())->min;
565:   if (!opt_prop.has_value()) {
566:     Py_RETURN_NONE;
567:   }
568:   auto prop = opt_prop.value();
569:   if (prop.isComplex()) {
570:     auto cprop = prop.to<c10::complex<double>>();
571:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
572:   } else if (prop.isFloatingPoint()) {
573:     return PyFloat_FromDouble(prop.to<double>());
574:   } else if (prop.isIntegral(/*includeBool=*/false)) {
575:     return PyLong_FromLong(prop.to<int64_t>());
576:   } else if (prop.isBoolean()) {
577:     if (prop.to<bool>()) {
578:       Py_RETURN_TRUE;
579:     } else {
580:       Py_RETURN_FALSE;
581:     }
582:   } else {
583:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
584:     return nullptr;
585:   }
586:   END_HANDLE_TH_ERRORS
587: }
588: 
589: static PyObject* THPClampBackward1_self_getter(THPCppFunction *self, void *_unused) {
590:   HANDLE_TH_ERRORS
591:   const auto& prop = static_cast<ClampBackward1*>(self->cdata.get())->self_;
592:   return THPVariable_Wrap(prop.unpack(self->cdata));
593:   END_HANDLE_TH_ERRORS
594: }
595: 
596: static PyObject* THPClampBackward1_self_raw_getter(THPCppFunction *self, void *_unused) {
597:   HANDLE_TH_ERRORS
598:   const auto& prop = static_cast<ClampBackward1*>(self->cdata.get())->self_;
599:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
600:   return obj.release().ptr();
601:   END_HANDLE_TH_ERRORS
602: }
603: 
604: static struct PyGetSetDef ClampBackward1_properties[] = {
605:   THP_FUNCTION_DEFAULT_PROPERTIES,
606:   {(char*)"_saved_max", (getter)THPClampBackward1_max_getter, nullptr, nullptr, nullptr},
607:   {(char*)"_saved_min", (getter)THPClampBackward1_min_getter, nullptr, nullptr, nullptr},
608:   {(char*)"_saved_self", (getter)THPClampBackward1_self_getter, nullptr, nullptr, nullptr},
609:   {(char*)"_raw_saved_self", (getter)THPClampBackward1_self_raw_getter, nullptr, nullptr, nullptr},
610:   {nullptr} /* sentinel */
611: };
612: 
613: static PyObject* THPClampMaxBackward0_max_getter(THPCppFunction *self, void *_unused) {
614:   HANDLE_TH_ERRORS
615:   auto prop = static_cast<ClampMaxBackward0*>(self->cdata.get())->max;
616:   if (prop.isComplex()) {
617:     auto cprop = prop.to<c10::complex<double>>();
618:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
619:   } else if (prop.isFloatingPoint()) {
620:     return PyFloat_FromDouble(prop.to<double>());
621:   } else if (prop.isIntegral(/*includeBool=*/false)) {
622:     return PyLong_FromLong(prop.to<int64_t>());
623:   } else if (prop.isBoolean()) {
624:     if (prop.to<bool>()) {
625:       Py_RETURN_TRUE;
626:     } else {
627:       Py_RETURN_FALSE;
628:     }
629:   } else {
630:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
631:     return nullptr;
632:   }
633:   END_HANDLE_TH_ERRORS
634: }
635: 
636: static PyObject* THPClampMaxBackward0_self_getter(THPCppFunction *self, void *_unused) {
637:   HANDLE_TH_ERRORS
638:   const auto& prop = static_cast<ClampMaxBackward0*>(self->cdata.get())->self_;
639:   return THPVariable_Wrap(prop.unpack(self->cdata));
640:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPClampBackward1_min_getter`, `PyComplex_FromDoubles`, `PyFloat_FromDouble`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPClampBackward1_min_getter`, `PyComplex_FromDoubles`, `PyFloat_FromDouble` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 641-720

```cpp
641: }
642: 
643: static PyObject* THPClampMaxBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
644:   HANDLE_TH_ERRORS
645:   const auto& prop = static_cast<ClampMaxBackward0*>(self->cdata.get())->self_;
646:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
647:   return obj.release().ptr();
648:   END_HANDLE_TH_ERRORS
649: }
650: 
651: static struct PyGetSetDef ClampMaxBackward0_properties[] = {
652:   THP_FUNCTION_DEFAULT_PROPERTIES,
653:   {(char*)"_saved_max", (getter)THPClampMaxBackward0_max_getter, nullptr, nullptr, nullptr},
654:   {(char*)"_saved_self", (getter)THPClampMaxBackward0_self_getter, nullptr, nullptr, nullptr},
655:   {(char*)"_raw_saved_self", (getter)THPClampMaxBackward0_self_raw_getter, nullptr, nullptr, nullptr},
656:   {nullptr} /* sentinel */
657: };
658: 
659: static PyObject* THPClampMaxBackward1_max_getter(THPCppFunction *self, void *_unused) {
660:   HANDLE_TH_ERRORS
661:   const auto& prop = static_cast<ClampMaxBackward1*>(self->cdata.get())->max_;
662:   return THPVariable_Wrap(prop.unpack(self->cdata));
663:   END_HANDLE_TH_ERRORS
664: }
665: 
666: static PyObject* THPClampMaxBackward1_max_raw_getter(THPCppFunction *self, void *_unused) {
667:   HANDLE_TH_ERRORS
668:   const auto& prop = static_cast<ClampMaxBackward1*>(self->cdata.get())->max_;
669:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
670:   return obj.release().ptr();
671:   END_HANDLE_TH_ERRORS
672: }
673: 
674: static PyObject* THPClampMaxBackward1_self_getter(THPCppFunction *self, void *_unused) {
675:   HANDLE_TH_ERRORS
676:   const auto& prop = static_cast<ClampMaxBackward1*>(self->cdata.get())->self_;
677:   return THPVariable_Wrap(prop.unpack(self->cdata));
678:   END_HANDLE_TH_ERRORS
679: }
680: 
681: static PyObject* THPClampMaxBackward1_self_raw_getter(THPCppFunction *self, void *_unused) {
682:   HANDLE_TH_ERRORS
683:   const auto& prop = static_cast<ClampMaxBackward1*>(self->cdata.get())->self_;
684:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
685:   return obj.release().ptr();
686:   END_HANDLE_TH_ERRORS
687: }
688: 
689: static struct PyGetSetDef ClampMaxBackward1_properties[] = {
690:   THP_FUNCTION_DEFAULT_PROPERTIES,
691:   {(char*)"_saved_max", (getter)THPClampMaxBackward1_max_getter, nullptr, nullptr, nullptr},
692:   {(char*)"_raw_saved_max", (getter)THPClampMaxBackward1_max_raw_getter, nullptr, nullptr, nullptr},
693:   {(char*)"_saved_self", (getter)THPClampMaxBackward1_self_getter, nullptr, nullptr, nullptr},
694:   {(char*)"_raw_saved_self", (getter)THPClampMaxBackward1_self_raw_getter, nullptr, nullptr, nullptr},
695:   {nullptr} /* sentinel */
696: };
697: 
698: 
699: 
700: static struct PyGetSetDef LazyCloneBackward0_properties[] = {
701:   THP_FUNCTION_DEFAULT_PROPERTIES,
702: 
703:   {nullptr} /* sentinel */
704: };
705: 
706: static PyObject* THPComplexBackward0_imag_getter(THPCppFunction *self, void *_unused) {
707:   HANDLE_TH_ERRORS
708:   const auto& prop = static_cast<ComplexBackward0*>(self->cdata.get())->imag_;
709:   return THPVariable_Wrap(prop.unpack(self->cdata));
710:   END_HANDLE_TH_ERRORS
711: }
712: 
713: static PyObject* THPComplexBackward0_imag_raw_getter(THPCppFunction *self, void *_unused) {
714:   HANDLE_TH_ERRORS
715:   const auto& prop = static_cast<ComplexBackward0*>(self->cdata.get())->imag_;
716:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
717:   return obj.release().ptr();
718:   END_HANDLE_TH_ERRORS
719: }
720: 
```

- EN: The main execution path in this span is carried by `THPClampMaxBackward0_self_raw_getter`, `cast`, `THPClampMaxBackward1_max_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPClampMaxBackward0_self_raw_getter`, `cast`, `THPClampMaxBackward1_max_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 721-800

```cpp
721: static PyObject* THPComplexBackward0_real_getter(THPCppFunction *self, void *_unused) {
722:   HANDLE_TH_ERRORS
723:   const auto& prop = static_cast<ComplexBackward0*>(self->cdata.get())->real_;
724:   return THPVariable_Wrap(prop.unpack(self->cdata));
725:   END_HANDLE_TH_ERRORS
726: }
727: 
728: static PyObject* THPComplexBackward0_real_raw_getter(THPCppFunction *self, void *_unused) {
729:   HANDLE_TH_ERRORS
730:   const auto& prop = static_cast<ComplexBackward0*>(self->cdata.get())->real_;
731:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
732:   return obj.release().ptr();
733:   END_HANDLE_TH_ERRORS
734: }
735: 
736: static struct PyGetSetDef ComplexBackward0_properties[] = {
737:   THP_FUNCTION_DEFAULT_PROPERTIES,
738:   {(char*)"_saved_imag", (getter)THPComplexBackward0_imag_getter, nullptr, nullptr, nullptr},
739:   {(char*)"_raw_saved_imag", (getter)THPComplexBackward0_imag_raw_getter, nullptr, nullptr, nullptr},
740:   {(char*)"_saved_real", (getter)THPComplexBackward0_real_getter, nullptr, nullptr, nullptr},
741:   {(char*)"_raw_saved_real", (getter)THPComplexBackward0_real_raw_getter, nullptr, nullptr, nullptr},
742:   {nullptr} /* sentinel */
743: };
744: 
745: static PyObject* THPPolarBackward0_result_getter(THPCppFunction *self, void *_unused) {
746:   HANDLE_TH_ERRORS
747:   const auto& prop = static_cast<PolarBackward0*>(self->cdata.get())->result_;
748:   return THPVariable_Wrap(prop.unpack(self->cdata));
749:   END_HANDLE_TH_ERRORS
750: }
751: 
752: static PyObject* THPPolarBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
753:   HANDLE_TH_ERRORS
754:   const auto& prop = static_cast<PolarBackward0*>(self->cdata.get())->result_;
755:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
756:   return obj.release().ptr();
757:   END_HANDLE_TH_ERRORS
758: }
759: 
760: static struct PyGetSetDef PolarBackward0_properties[] = {
761:   THP_FUNCTION_DEFAULT_PROPERTIES,
762:   {(char*)"_saved_result", (getter)THPPolarBackward0_result_getter, nullptr, nullptr, nullptr},
763:   {(char*)"_raw_saved_result", (getter)THPPolarBackward0_result_raw_getter, nullptr, nullptr, nullptr},
764:   {nullptr} /* sentinel */
765: };
766: 
767: static PyObject* THPCummaxBackward0_dim_getter(THPCppFunction *self, void *_unused) {
768:   HANDLE_TH_ERRORS
769:   auto prop = static_cast<CummaxBackward0*>(self->cdata.get())->dim;
770:   return PyLong_FromUnsignedLong((int64_t) prop);
771:   END_HANDLE_TH_ERRORS
772: }
773: 
774: static PyObject* THPCummaxBackward0_self_getter(THPCppFunction *self, void *_unused) {
775:   HANDLE_TH_ERRORS
776:   const auto& prop = static_cast<CummaxBackward0*>(self->cdata.get())->self_;
777:   return THPVariable_Wrap(prop.unpack(self->cdata));
778:   END_HANDLE_TH_ERRORS
779: }
780: 
781: static PyObject* THPCummaxBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
782:   HANDLE_TH_ERRORS
783:   const auto& prop = static_cast<CummaxBackward0*>(self->cdata.get())->self_;
784:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
785:   return obj.release().ptr();
786:   END_HANDLE_TH_ERRORS
787: }
788: 
789: static PyObject* THPCummaxBackward0_indices_getter(THPCppFunction *self, void *_unused) {
790:   HANDLE_TH_ERRORS
791:   const auto& prop = static_cast<CummaxBackward0*>(self->cdata.get())->indices_;
792:   return THPVariable_Wrap(prop.unpack(self->cdata));
793:   END_HANDLE_TH_ERRORS
794: }
795: 
796: static PyObject* THPCummaxBackward0_indices_raw_getter(THPCppFunction *self, void *_unused) {
797:   HANDLE_TH_ERRORS
798:   const auto& prop = static_cast<CummaxBackward0*>(self->cdata.get())->indices_;
799:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
800:   return obj.release().ptr();
```

- EN: The main execution path in this span is carried by `THPComplexBackward0_real_getter`, `THPVariable_Wrap`, `THPComplexBackward0_real_raw_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPComplexBackward0_real_getter`, `THPVariable_Wrap`, `THPComplexBackward0_real_raw_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 801-880

```cpp
801:   END_HANDLE_TH_ERRORS
802: }
803: 
804: static struct PyGetSetDef CummaxBackward0_properties[] = {
805:   THP_FUNCTION_DEFAULT_PROPERTIES,
806:   {(char*)"_saved_dim", (getter)THPCummaxBackward0_dim_getter, nullptr, nullptr, nullptr},
807:   {(char*)"_saved_self", (getter)THPCummaxBackward0_self_getter, nullptr, nullptr, nullptr},
808:   {(char*)"_raw_saved_self", (getter)THPCummaxBackward0_self_raw_getter, nullptr, nullptr, nullptr},
809:   {(char*)"_saved_indices", (getter)THPCummaxBackward0_indices_getter, nullptr, nullptr, nullptr},
810:   {(char*)"_raw_saved_indices", (getter)THPCummaxBackward0_indices_raw_getter, nullptr, nullptr, nullptr},
811:   {nullptr} /* sentinel */
812: };
813: 
814: 
815: 
816: static struct PyGetSetDef BlockDiagBackward0_properties[] = {
817:   THP_FUNCTION_DEFAULT_PROPERTIES,
818: 
819:   {nullptr} /* sentinel */
820: };
821: 
822: static PyObject* THPDiagonalBackwardBackward0_dim1_getter(THPCppFunction *self, void *_unused) {
823:   HANDLE_TH_ERRORS
824:   auto prop = static_cast<DiagonalBackwardBackward0*>(self->cdata.get())->dim1;
825:   return PyLong_FromUnsignedLong((int64_t) prop);
826:   END_HANDLE_TH_ERRORS
827: }
828: 
829: static PyObject* THPDiagonalBackwardBackward0_dim2_getter(THPCppFunction *self, void *_unused) {
830:   HANDLE_TH_ERRORS
831:   auto prop = static_cast<DiagonalBackwardBackward0*>(self->cdata.get())->dim2;
832:   return PyLong_FromUnsignedLong((int64_t) prop);
833:   END_HANDLE_TH_ERRORS
834: }
835: 
836: static PyObject* THPDiagonalBackwardBackward0_offset_getter(THPCppFunction *self, void *_unused) {
837:   HANDLE_TH_ERRORS
838:   auto prop = static_cast<DiagonalBackwardBackward0*>(self->cdata.get())->offset;
839:   return PyLong_FromUnsignedLong((int64_t) prop);
840:   END_HANDLE_TH_ERRORS
841: }
842: 
843: static struct PyGetSetDef DiagonalBackwardBackward0_properties[] = {
844:   THP_FUNCTION_DEFAULT_PROPERTIES,
845:   {(char*)"_saved_dim1", (getter)THPDiagonalBackwardBackward0_dim1_getter, nullptr, nullptr, nullptr},
846:   {(char*)"_saved_dim2", (getter)THPDiagonalBackwardBackward0_dim2_getter, nullptr, nullptr, nullptr},
847:   {(char*)"_saved_offset", (getter)THPDiagonalBackwardBackward0_offset_getter, nullptr, nullptr, nullptr},
848:   {nullptr} /* sentinel */
849: };
850: 
851: static PyObject* THPDotBackward0_self_getter(THPCppFunction *self, void *_unused) {
852:   HANDLE_TH_ERRORS
853:   const auto& prop = static_cast<DotBackward0*>(self->cdata.get())->self_;
854:   return THPVariable_Wrap(prop.unpack(self->cdata));
855:   END_HANDLE_TH_ERRORS
856: }
857: 
858: static PyObject* THPDotBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
859:   HANDLE_TH_ERRORS
860:   const auto& prop = static_cast<DotBackward0*>(self->cdata.get())->self_;
861:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
862:   return obj.release().ptr();
863:   END_HANDLE_TH_ERRORS
864: }
865: 
866: static PyObject* THPDotBackward0_tensor_getter(THPCppFunction *self, void *_unused) {
867:   HANDLE_TH_ERRORS
868:   const auto& prop = static_cast<DotBackward0*>(self->cdata.get())->tensor_;
869:   return THPVariable_Wrap(prop.unpack(self->cdata));
870:   END_HANDLE_TH_ERRORS
871: }
872: 
873: static PyObject* THPDotBackward0_tensor_raw_getter(THPCppFunction *self, void *_unused) {
874:   HANDLE_TH_ERRORS
875:   const auto& prop = static_cast<DotBackward0*>(self->cdata.get())->tensor_;
876:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
877:   return obj.release().ptr();
878:   END_HANDLE_TH_ERRORS
879: }
880: 
```

- EN: The main execution path in this span is carried by `THPDiagonalBackwardBackward0_dim1_getter`, `PyLong_FromUnsignedLong`, `THPDiagonalBackwardBackward0_dim2_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPDiagonalBackwardBackward0_dim1_getter`, `PyLong_FromUnsignedLong`, `THPDiagonalBackwardBackward0_dim2_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 881-960

```cpp
881: static struct PyGetSetDef DotBackward0_properties[] = {
882:   THP_FUNCTION_DEFAULT_PROPERTIES,
883:   {(char*)"_saved_self", (getter)THPDotBackward0_self_getter, nullptr, nullptr, nullptr},
884:   {(char*)"_raw_saved_self", (getter)THPDotBackward0_self_raw_getter, nullptr, nullptr, nullptr},
885:   {(char*)"_saved_tensor", (getter)THPDotBackward0_tensor_getter, nullptr, nullptr, nullptr},
886:   {(char*)"_raw_saved_tensor", (getter)THPDotBackward0_tensor_raw_getter, nullptr, nullptr, nullptr},
887:   {nullptr} /* sentinel */
888: };
889: 
890: static PyObject* THPNativeDropoutBackward0_p_getter(THPCppFunction *self, void *_unused) {
891:   HANDLE_TH_ERRORS
892:   auto prop = static_cast<NativeDropoutBackward0*>(self->cdata.get())->p;
893:   return PyFloat_FromDouble((double) prop);
894:   END_HANDLE_TH_ERRORS
895: }
896: 
897: static PyObject* THPNativeDropoutBackward0_result1_getter(THPCppFunction *self, void *_unused) {
898:   HANDLE_TH_ERRORS
899:   const auto& prop = static_cast<NativeDropoutBackward0*>(self->cdata.get())->result1_;
900:   return THPVariable_Wrap(prop.unpack(self->cdata));
901:   END_HANDLE_TH_ERRORS
902: }
903: 
904: static PyObject* THPNativeDropoutBackward0_result1_raw_getter(THPCppFunction *self, void *_unused) {
905:   HANDLE_TH_ERRORS
906:   const auto& prop = static_cast<NativeDropoutBackward0*>(self->cdata.get())->result1_;
907:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
908:   return obj.release().ptr();
909:   END_HANDLE_TH_ERRORS
910: }
911: 
912: static struct PyGetSetDef NativeDropoutBackward0_properties[] = {
913:   THP_FUNCTION_DEFAULT_PROPERTIES,
914:   {(char*)"_saved_p", (getter)THPNativeDropoutBackward0_p_getter, nullptr, nullptr, nullptr},
915:   {(char*)"_saved_result1", (getter)THPNativeDropoutBackward0_result1_getter, nullptr, nullptr, nullptr},
916:   {(char*)"_raw_saved_result1", (getter)THPNativeDropoutBackward0_result1_raw_getter, nullptr, nullptr, nullptr},
917:   {nullptr} /* sentinel */
918: };
919: 
920: static PyObject* THPExpBackward0_result_getter(THPCppFunction *self, void *_unused) {
921:   HANDLE_TH_ERRORS
922:   const auto& prop = static_cast<ExpBackward0*>(self->cdata.get())->result_;
923:   return THPVariable_Wrap(prop.unpack(self->cdata));
924:   END_HANDLE_TH_ERRORS
925: }
926: 
927: static PyObject* THPExpBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
928:   HANDLE_TH_ERRORS
929:   const auto& prop = static_cast<ExpBackward0*>(self->cdata.get())->result_;
930:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
931:   return obj.release().ptr();
932:   END_HANDLE_TH_ERRORS
933: }
934: 
935: static struct PyGetSetDef ExpBackward0_properties[] = {
936:   THP_FUNCTION_DEFAULT_PROPERTIES,
937:   {(char*)"_saved_result", (getter)THPExpBackward0_result_getter, nullptr, nullptr, nullptr},
938:   {(char*)"_raw_saved_result", (getter)THPExpBackward0_result_raw_getter, nullptr, nullptr, nullptr},
939:   {nullptr} /* sentinel */
940: };
941: 
942: static PyObject* THPExp2Backward0_result_getter(THPCppFunction *self, void *_unused) {
943:   HANDLE_TH_ERRORS
944:   const auto& prop = static_cast<Exp2Backward0*>(self->cdata.get())->result_;
945:   return THPVariable_Wrap(prop.unpack(self->cdata));
946:   END_HANDLE_TH_ERRORS
947: }
948: 
949: static PyObject* THPExp2Backward0_result_raw_getter(THPCppFunction *self, void *_unused) {
950:   HANDLE_TH_ERRORS
951:   const auto& prop = static_cast<Exp2Backward0*>(self->cdata.get())->result_;
952:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
953:   return obj.release().ptr();
954:   END_HANDLE_TH_ERRORS
955: }
956: 
957: static struct PyGetSetDef Exp2Backward0_properties[] = {
958:   THP_FUNCTION_DEFAULT_PROPERTIES,
959:   {(char*)"_saved_result", (getter)THPExp2Backward0_result_getter, nullptr, nullptr, nullptr},
960:   {(char*)"_raw_saved_result", (getter)THPExp2Backward0_result_raw_getter, nullptr, nullptr, nullptr},
```

- EN: The main execution path in this span is carried by `THPNativeDropoutBackward0_p_getter`, `PyFloat_FromDouble`, `THPNativeDropoutBackward0_result1_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPNativeDropoutBackward0_p_getter`, `PyFloat_FromDouble`, `THPNativeDropoutBackward0_result1_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 961-1040

```cpp
 961:   {nullptr} /* sentinel */
 962: };
 963: 
 964: static PyObject* THPFakeQuantizePerTensorAffineCachemaskBackward0_mask_getter(THPCppFunction *self, void *_unused) {
 965:   HANDLE_TH_ERRORS
 966:   const auto& prop = static_cast<FakeQuantizePerTensorAffineCachemaskBackward0*>(self->cdata.get())->mask_;
 967:   return THPVariable_Wrap(prop.unpack(self->cdata));
 968:   END_HANDLE_TH_ERRORS
 969: }
 970: 
 971: static PyObject* THPFakeQuantizePerTensorAffineCachemaskBackward0_mask_raw_getter(THPCppFunction *self, void *_unused) {
 972:   HANDLE_TH_ERRORS
 973:   const auto& prop = static_cast<FakeQuantizePerTensorAffineCachemaskBackward0*>(self->cdata.get())->mask_;
 974:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
 975:   return obj.release().ptr();
 976:   END_HANDLE_TH_ERRORS
 977: }
 978: 
 979: static struct PyGetSetDef FakeQuantizePerTensorAffineCachemaskBackward0_properties[] = {
 980:   THP_FUNCTION_DEFAULT_PROPERTIES,
 981:   {(char*)"_saved_mask", (getter)THPFakeQuantizePerTensorAffineCachemaskBackward0_mask_getter, nullptr, nullptr, nullptr},
 982:   {(char*)"_raw_saved_mask", (getter)THPFakeQuantizePerTensorAffineCachemaskBackward0_mask_raw_getter, nullptr, nullptr, nullptr},
 983:   {nullptr} /* sentinel */
 984: };
 985: 
 986: static PyObject* THPFakeQuantizePerTensorAffineCachemaskTensorQparamsBackward0_mask_getter(THPCppFunction *self, void *_unused) {
 987:   HANDLE_TH_ERRORS
 988:   const auto& prop = static_cast<FakeQuantizePerTensorAffineCachemaskTensorQparamsBackward0*>(self->cdata.get())->mask_;
 989:   return THPVariable_Wrap(prop.unpack(self->cdata));
 990:   END_HANDLE_TH_ERRORS
 991: }
 992: 
 993: static PyObject* THPFakeQuantizePerTensorAffineCachemaskTensorQparamsBackward0_mask_raw_getter(THPCppFunction *self, void *_unused) {
 994:   HANDLE_TH_ERRORS
 995:   const auto& prop = static_cast<FakeQuantizePerTensorAffineCachemaskTensorQparamsBackward0*>(self->cdata.get())->mask_;
 996:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
 997:   return obj.release().ptr();
 998:   END_HANDLE_TH_ERRORS
 999: }
1000: 
1001: static struct PyGetSetDef FakeQuantizePerTensorAffineCachemaskTensorQparamsBackward0_properties[] = {
1002:   THP_FUNCTION_DEFAULT_PROPERTIES,
1003:   {(char*)"_saved_mask", (getter)THPFakeQuantizePerTensorAffineCachemaskTensorQparamsBackward0_mask_getter, nullptr, nullptr, nullptr},
1004:   {(char*)"_raw_saved_mask", (getter)THPFakeQuantizePerTensorAffineCachemaskTensorQparamsBackward0_mask_raw_getter, nullptr, nullptr, nullptr},
1005:   {nullptr} /* sentinel */
1006: };
1007: 
1008: static PyObject* THPFakeQuantizeLearnablePerTensorAffineBackward0_grad_factor_getter(THPCppFunction *self, void *_unused) {
1009:   HANDLE_TH_ERRORS
1010:   auto prop = static_cast<FakeQuantizeLearnablePerTensorAffineBackward0*>(self->cdata.get())->grad_factor;
1011:   return PyFloat_FromDouble((double) prop);
1012:   END_HANDLE_TH_ERRORS
1013: }
1014: 
1015: static PyObject* THPFakeQuantizeLearnablePerTensorAffineBackward0_quant_max_getter(THPCppFunction *self, void *_unused) {
1016:   HANDLE_TH_ERRORS
1017:   auto prop = static_cast<FakeQuantizeLearnablePerTensorAffineBackward0*>(self->cdata.get())->quant_max;
1018:   return PyLong_FromUnsignedLong((int64_t) prop);
1019:   END_HANDLE_TH_ERRORS
1020: }
1021: 
1022: static PyObject* THPFakeQuantizeLearnablePerTensorAffineBackward0_quant_min_getter(THPCppFunction *self, void *_unused) {
1023:   HANDLE_TH_ERRORS
1024:   auto prop = static_cast<FakeQuantizeLearnablePerTensorAffineBackward0*>(self->cdata.get())->quant_min;
1025:   return PyLong_FromUnsignedLong((int64_t) prop);
1026:   END_HANDLE_TH_ERRORS
1027: }
1028: 
1029: static PyObject* THPFakeQuantizeLearnablePerTensorAffineBackward0_scale_getter(THPCppFunction *self, void *_unused) {
1030:   HANDLE_TH_ERRORS
1031:   const auto& prop = static_cast<FakeQuantizeLearnablePerTensorAffineBackward0*>(self->cdata.get())->scale_;
1032:   return THPVariable_Wrap(prop.unpack(self->cdata));
1033:   END_HANDLE_TH_ERRORS
1034: }
1035: 
1036: static PyObject* THPFakeQuantizeLearnablePerTensorAffineBackward0_scale_raw_getter(THPCppFunction *self, void *_unused) {
1037:   HANDLE_TH_ERRORS
1038:   const auto& prop = static_cast<FakeQuantizeLearnablePerTensorAffineBackward0*>(self->cdata.get())->scale_;
1039:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1040:   return obj.release().ptr();
```

- EN: The main execution path in this span is carried by `THPFakeQuantizePerTensorAffineCachemaskBackward0_mask_getter`, `THPVariable_Wrap`, `THPFakeQuantizePerTensorAffineCachemaskBackward0_mask_raw_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPFakeQuantizePerTensorAffineCachemaskBackward0_mask_getter`, `THPVariable_Wrap`, `THPFakeQuantizePerTensorAffineCachemaskBackward0_mask_raw_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1041-1120

```cpp
1041:   END_HANDLE_TH_ERRORS
1042: }
1043: 
1044: static PyObject* THPFakeQuantizeLearnablePerTensorAffineBackward0_self_getter(THPCppFunction *self, void *_unused) {
1045:   HANDLE_TH_ERRORS
1046:   const auto& prop = static_cast<FakeQuantizeLearnablePerTensorAffineBackward0*>(self->cdata.get())->self_;
1047:   return THPVariable_Wrap(prop.unpack(self->cdata));
1048:   END_HANDLE_TH_ERRORS
1049: }
1050: 
1051: static PyObject* THPFakeQuantizeLearnablePerTensorAffineBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
1052:   HANDLE_TH_ERRORS
1053:   const auto& prop = static_cast<FakeQuantizeLearnablePerTensorAffineBackward0*>(self->cdata.get())->self_;
1054:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1055:   return obj.release().ptr();
1056:   END_HANDLE_TH_ERRORS
1057: }
1058: 
1059: static PyObject* THPFakeQuantizeLearnablePerTensorAffineBackward0_zero_point_getter(THPCppFunction *self, void *_unused) {
1060:   HANDLE_TH_ERRORS
1061:   const auto& prop = static_cast<FakeQuantizeLearnablePerTensorAffineBackward0*>(self->cdata.get())->zero_point_;
1062:   return THPVariable_Wrap(prop.unpack(self->cdata));
1063:   END_HANDLE_TH_ERRORS
1064: }
1065: 
1066: static PyObject* THPFakeQuantizeLearnablePerTensorAffineBackward0_zero_point_raw_getter(THPCppFunction *self, void *_unused) {
1067:   HANDLE_TH_ERRORS
1068:   const auto& prop = static_cast<FakeQuantizeLearnablePerTensorAffineBackward0*>(self->cdata.get())->zero_point_;
1069:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1070:   return obj.release().ptr();
1071:   END_HANDLE_TH_ERRORS
1072: }
1073: 
1074: static struct PyGetSetDef FakeQuantizeLearnablePerTensorAffineBackward0_properties[] = {
1075:   THP_FUNCTION_DEFAULT_PROPERTIES,
1076:   {(char*)"_saved_grad_factor", (getter)THPFakeQuantizeLearnablePerTensorAffineBackward0_grad_factor_getter, nullptr, nullptr, nullptr},
1077:   {(char*)"_saved_quant_max", (getter)THPFakeQuantizeLearnablePerTensorAffineBackward0_quant_max_getter, nullptr, nullptr, nullptr},
1078:   {(char*)"_saved_quant_min", (getter)THPFakeQuantizeLearnablePerTensorAffineBackward0_quant_min_getter, nullptr, nullptr, nullptr},
1079:   {(char*)"_saved_scale", (getter)THPFakeQuantizeLearnablePerTensorAffineBackward0_scale_getter, nullptr, nullptr, nullptr},
1080:   {(char*)"_raw_saved_scale", (getter)THPFakeQuantizeLearnablePerTensorAffineBackward0_scale_raw_getter, nullptr, nullptr, nullptr},
1081:   {(char*)"_saved_self", (getter)THPFakeQuantizeLearnablePerTensorAffineBackward0_self_getter, nullptr, nullptr, nullptr},
1082:   {(char*)"_raw_saved_self", (getter)THPFakeQuantizeLearnablePerTensorAffineBackward0_self_raw_getter, nullptr, nullptr, nullptr},
1083:   {(char*)"_saved_zero_point", (getter)THPFakeQuantizeLearnablePerTensorAffineBackward0_zero_point_getter, nullptr, nullptr, nullptr},
1084:   {(char*)"_raw_saved_zero_point", (getter)THPFakeQuantizeLearnablePerTensorAffineBackward0_zero_point_raw_getter, nullptr, nullptr, nullptr},
1085:   {nullptr} /* sentinel */
1086: };
1087: 
1088: 
1089: 
1090: static struct PyGetSetDef FracBackward0_properties[] = {
1091:   THP_FUNCTION_DEFAULT_PROPERTIES,
1092: 
1093:   {nullptr} /* sentinel */
1094: };
1095: 
1096: static PyObject* THPHardswishBackward0_self_getter(THPCppFunction *self, void *_unused) {
1097:   HANDLE_TH_ERRORS
1098:   const auto& prop = static_cast<HardswishBackward0*>(self->cdata.get())->self_;
1099:   return THPVariable_Wrap(prop.unpack(self->cdata));
1100:   END_HANDLE_TH_ERRORS
1101: }
1102: 
1103: static PyObject* THPHardswishBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
1104:   HANDLE_TH_ERRORS
1105:   const auto& prop = static_cast<HardswishBackward0*>(self->cdata.get())->self_;
1106:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1107:   return obj.release().ptr();
1108:   END_HANDLE_TH_ERRORS
1109: }
1110: 
1111: static struct PyGetSetDef HardswishBackward0_properties[] = {
1112:   THP_FUNCTION_DEFAULT_PROPERTIES,
1113:   {(char*)"_saved_self", (getter)THPHardswishBackward0_self_getter, nullptr, nullptr, nullptr},
1114:   {(char*)"_raw_saved_self", (getter)THPHardswishBackward0_self_raw_getter, nullptr, nullptr, nullptr},
1115:   {nullptr} /* sentinel */
1116: };
1117: 
1118: static PyObject* THPHypotBackward0_other_getter(THPCppFunction *self, void *_unused) {
1119:   HANDLE_TH_ERRORS
1120:   const auto& prop = static_cast<HypotBackward0*>(self->cdata.get())->other_;
```

- EN: The main execution path in this span is carried by `THPFakeQuantizeLearnablePerTensorAffineBackward0_self_getter`, `THPVariable_Wrap`, `THPFakeQuantizeLearnablePerTensorAffineBackward0_self_raw_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPFakeQuantizeLearnablePerTensorAffineBackward0_self_getter`, `THPVariable_Wrap`, `THPFakeQuantizeLearnablePerTensorAffineBackward0_self_raw_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1121-1200

```cpp
1121:   return THPVariable_Wrap(prop.unpack(self->cdata));
1122:   END_HANDLE_TH_ERRORS
1123: }
1124: 
1125: static PyObject* THPHypotBackward0_other_raw_getter(THPCppFunction *self, void *_unused) {
1126:   HANDLE_TH_ERRORS
1127:   const auto& prop = static_cast<HypotBackward0*>(self->cdata.get())->other_;
1128:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1129:   return obj.release().ptr();
1130:   END_HANDLE_TH_ERRORS
1131: }
1132: 
1133: static PyObject* THPHypotBackward0_self_getter(THPCppFunction *self, void *_unused) {
1134:   HANDLE_TH_ERRORS
1135:   const auto& prop = static_cast<HypotBackward0*>(self->cdata.get())->self_;
1136:   return THPVariable_Wrap(prop.unpack(self->cdata));
1137:   END_HANDLE_TH_ERRORS
1138: }
1139: 
1140: static PyObject* THPHypotBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
1141:   HANDLE_TH_ERRORS
1142:   const auto& prop = static_cast<HypotBackward0*>(self->cdata.get())->self_;
1143:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1144:   return obj.release().ptr();
1145:   END_HANDLE_TH_ERRORS
1146: }
1147: 
1148: static PyObject* THPHypotBackward0_result_getter(THPCppFunction *self, void *_unused) {
1149:   HANDLE_TH_ERRORS
1150:   const auto& prop = static_cast<HypotBackward0*>(self->cdata.get())->result_;
1151:   return THPVariable_Wrap(prop.unpack(self->cdata));
1152:   END_HANDLE_TH_ERRORS
1153: }
1154: 
1155: static PyObject* THPHypotBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
1156:   HANDLE_TH_ERRORS
1157:   const auto& prop = static_cast<HypotBackward0*>(self->cdata.get())->result_;
1158:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1159:   return obj.release().ptr();
1160:   END_HANDLE_TH_ERRORS
1161: }
1162: 
1163: static struct PyGetSetDef HypotBackward0_properties[] = {
1164:   THP_FUNCTION_DEFAULT_PROPERTIES,
1165:   {(char*)"_saved_other", (getter)THPHypotBackward0_other_getter, nullptr, nullptr, nullptr},
1166:   {(char*)"_raw_saved_other", (getter)THPHypotBackward0_other_raw_getter, nullptr, nullptr, nullptr},
1167:   {(char*)"_saved_self", (getter)THPHypotBackward0_self_getter, nullptr, nullptr, nullptr},
1168:   {(char*)"_raw_saved_self", (getter)THPHypotBackward0_self_raw_getter, nullptr, nullptr, nullptr},
1169:   {(char*)"_saved_result", (getter)THPHypotBackward0_result_getter, nullptr, nullptr, nullptr},
1170:   {(char*)"_raw_saved_result", (getter)THPHypotBackward0_result_raw_getter, nullptr, nullptr, nullptr},
1171:   {nullptr} /* sentinel */
1172: };
1173: 
1174: static PyObject* THPIgammacBackward0_other_getter(THPCppFunction *self, void *_unused) {
1175:   HANDLE_TH_ERRORS
1176:   const auto& prop = static_cast<IgammacBackward0*>(self->cdata.get())->other_;
1177:   return THPVariable_Wrap(prop.unpack(self->cdata));
1178:   END_HANDLE_TH_ERRORS
1179: }
1180: 
1181: static PyObject* THPIgammacBackward0_other_raw_getter(THPCppFunction *self, void *_unused) {
1182:   HANDLE_TH_ERRORS
1183:   const auto& prop = static_cast<IgammacBackward0*>(self->cdata.get())->other_;
1184:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1185:   return obj.release().ptr();
1186:   END_HANDLE_TH_ERRORS
1187: }
1188: 
1189: static PyObject* THPIgammacBackward0_self_getter(THPCppFunction *self, void *_unused) {
1190:   HANDLE_TH_ERRORS
1191:   const auto& prop = static_cast<IgammacBackward0*>(self->cdata.get())->self_;
1192:   return THPVariable_Wrap(prop.unpack(self->cdata));
1193:   END_HANDLE_TH_ERRORS
1194: }
1195: 
1196: static PyObject* THPIgammacBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
1197:   HANDLE_TH_ERRORS
1198:   const auto& prop = static_cast<IgammacBackward0*>(self->cdata.get())->self_;
1199:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1200:   return obj.release().ptr();
```

- EN: The main execution path in this span is carried by `THPVariable_Wrap`, `THPHypotBackward0_other_raw_getter`, `cast`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_Wrap`, `THPHypotBackward0_other_raw_getter`, `cast` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1201-1280

```cpp
1201:   END_HANDLE_TH_ERRORS
1202: }
1203: 
1204: static struct PyGetSetDef IgammacBackward0_properties[] = {
1205:   THP_FUNCTION_DEFAULT_PROPERTIES,
1206:   {(char*)"_saved_other", (getter)THPIgammacBackward0_other_getter, nullptr, nullptr, nullptr},
1207:   {(char*)"_raw_saved_other", (getter)THPIgammacBackward0_other_raw_getter, nullptr, nullptr, nullptr},
1208:   {(char*)"_saved_self", (getter)THPIgammacBackward0_self_getter, nullptr, nullptr, nullptr},
1209:   {(char*)"_raw_saved_self", (getter)THPIgammacBackward0_self_raw_getter, nullptr, nullptr, nullptr},
1210:   {nullptr} /* sentinel */
1211: };
1212: 
1213: static PyObject* THPUnsafeMaskedIndexBackward0_indices_getter(THPCppFunction *self, void *_unused) {
1214:   HANDLE_TH_ERRORS
1215:   const auto *node = static_cast<UnsafeMaskedIndexBackward0*>(self->cdata.get());
1216:   const auto& prop = node->indices_;
1217:   if (node->indices_released_) {
1218:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
1219:     return nullptr;
1220:   }
1221:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
1222:   for (auto i: c10::irange(prop.size())) {
1223:     PyTuple_SetItem(tup, (Py_ssize_t) i, THPVariable_Wrap(prop[i].unpack(self->cdata)));
1224:   }
1225:   return tup;
1226:   END_HANDLE_TH_ERRORS
1227: }
1228: 
1229: static PyObject* THPUnsafeMaskedIndexBackward0_indices_raw_getter(THPCppFunction *self, void *_unused) {
1230:   HANDLE_TH_ERRORS
1231:   const auto *node = static_cast<UnsafeMaskedIndexBackward0*>(self->cdata.get());
1232:   const auto& prop = node->indices_;
1233:   if (node->indices_released_) {
1234:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
1235:     return nullptr;
1236:   }
1237:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
1238:   for (auto i : c10::irange(prop.size())) {
1239:     pybind11::object obj = pybind11::cast(prop[i], pybind11::return_value_policy::reference);
1240:     PyTuple_SetItem(tup, (Py_ssize_t) i, obj.release().ptr());
1241:   }
1242:   return tup;
1243:   END_HANDLE_TH_ERRORS
1244: }
1245: 
1246: static PyObject* THPUnsafeMaskedIndexBackward0_mask_getter(THPCppFunction *self, void *_unused) {
1247:   HANDLE_TH_ERRORS
1248:   const auto& prop = static_cast<UnsafeMaskedIndexBackward0*>(self->cdata.get())->mask_;
1249:   return THPVariable_Wrap(prop.unpack(self->cdata));
1250:   END_HANDLE_TH_ERRORS
1251: }
1252: 
1253: static PyObject* THPUnsafeMaskedIndexBackward0_mask_raw_getter(THPCppFunction *self, void *_unused) {
1254:   HANDLE_TH_ERRORS
1255:   const auto& prop = static_cast<UnsafeMaskedIndexBackward0*>(self->cdata.get())->mask_;
1256:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1257:   return obj.release().ptr();
1258:   END_HANDLE_TH_ERRORS
1259: }
1260: 
1261: static PyObject* THPUnsafeMaskedIndexBackward0_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
1262:   HANDLE_TH_ERRORS
1263:   auto prop = static_cast<UnsafeMaskedIndexBackward0*>(self->cdata.get())->self_sym_sizes;
1264:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
1265:   for (auto i : c10::irange(prop.size())) {
1266:       auto si = prop[i];
1267:       if (auto m = si.maybe_as_int()) {
1268:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
1269:       } else {
1270:         auto py_symint = py::cast(si).release().ptr();
1271:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
1272:       }
1273:   }
1274:   return tup;
1275:   END_HANDLE_TH_ERRORS
1276: }
1277: 
1278: static struct PyGetSetDef UnsafeMaskedIndexBackward0_properties[] = {
1279:   THP_FUNCTION_DEFAULT_PROPERTIES,
1280:   {(char*)"_saved_indices", (getter)THPUnsafeMaskedIndexBackward0_indices_getter, nullptr, nullptr, nullptr},
```

- EN: The main execution path in this span is carried by `THPUnsafeMaskedIndexBackward0_indices_getter`, `PyErr_SetString`, `PyTuple_New`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPUnsafeMaskedIndexBackward0_indices_getter`, `PyErr_SetString`, `PyTuple_New` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1281-1360

```cpp
1281:   {(char*)"_raw_saved_indices", (getter)THPUnsafeMaskedIndexBackward0_indices_raw_getter, nullptr, nullptr, nullptr},
1282:   {(char*)"_saved_mask", (getter)THPUnsafeMaskedIndexBackward0_mask_getter, nullptr, nullptr, nullptr},
1283:   {(char*)"_raw_saved_mask", (getter)THPUnsafeMaskedIndexBackward0_mask_raw_getter, nullptr, nullptr, nullptr},
1284:   {(char*)"_saved_self_sym_sizes", (getter)THPUnsafeMaskedIndexBackward0_self_sym_sizes_getter, nullptr, nullptr, nullptr},
1285:   {nullptr} /* sentinel */
1286: };
1287: 
1288: static PyObject* THPLdexpBackward0_other_getter(THPCppFunction *self, void *_unused) {
1289:   HANDLE_TH_ERRORS
1290:   const auto& prop = static_cast<LdexpBackward0*>(self->cdata.get())->other_;
1291:   return THPVariable_Wrap(prop.unpack(self->cdata));
1292:   END_HANDLE_TH_ERRORS
1293: }
1294: 
1295: static PyObject* THPLdexpBackward0_other_raw_getter(THPCppFunction *self, void *_unused) {
1296:   HANDLE_TH_ERRORS
1297:   const auto& prop = static_cast<LdexpBackward0*>(self->cdata.get())->other_;
1298:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1299:   return obj.release().ptr();
1300:   END_HANDLE_TH_ERRORS
1301: }
1302: 
1303: static PyObject* THPLdexpBackward0_result_getter(THPCppFunction *self, void *_unused) {
1304:   HANDLE_TH_ERRORS
1305:   const auto& prop = static_cast<LdexpBackward0*>(self->cdata.get())->result_;
1306:   return THPVariable_Wrap(prop.unpack(self->cdata));
1307:   END_HANDLE_TH_ERRORS
1308: }
1309: 
1310: static PyObject* THPLdexpBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
1311:   HANDLE_TH_ERRORS
1312:   const auto& prop = static_cast<LdexpBackward0*>(self->cdata.get())->result_;
1313:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1314:   return obj.release().ptr();
1315:   END_HANDLE_TH_ERRORS
1316: }
1317: 
1318: static struct PyGetSetDef LdexpBackward0_properties[] = {
1319:   THP_FUNCTION_DEFAULT_PROPERTIES,
1320:   {(char*)"_saved_other", (getter)THPLdexpBackward0_other_getter, nullptr, nullptr, nullptr},
1321:   {(char*)"_raw_saved_other", (getter)THPLdexpBackward0_other_raw_getter, nullptr, nullptr, nullptr},
1322:   {(char*)"_saved_result", (getter)THPLdexpBackward0_result_getter, nullptr, nullptr, nullptr},
1323:   {(char*)"_raw_saved_result", (getter)THPLdexpBackward0_result_raw_getter, nullptr, nullptr, nullptr},
1324:   {nullptr} /* sentinel */
1325: };
1326: 
1327: static PyObject* THPLogBackward0_self_getter(THPCppFunction *self, void *_unused) {
1328:   HANDLE_TH_ERRORS
1329:   const auto& prop = static_cast<LogBackward0*>(self->cdata.get())->self_;
1330:   return THPVariable_Wrap(prop.unpack(self->cdata));
1331:   END_HANDLE_TH_ERRORS
1332: }
1333: 
1334: static PyObject* THPLogBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
1335:   HANDLE_TH_ERRORS
1336:   const auto& prop = static_cast<LogBackward0*>(self->cdata.get())->self_;
1337:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1338:   return obj.release().ptr();
1339:   END_HANDLE_TH_ERRORS
1340: }
1341: 
1342: static struct PyGetSetDef LogBackward0_properties[] = {
1343:   THP_FUNCTION_DEFAULT_PROPERTIES,
1344:   {(char*)"_saved_self", (getter)THPLogBackward0_self_getter, nullptr, nullptr, nullptr},
1345:   {(char*)"_raw_saved_self", (getter)THPLogBackward0_self_raw_getter, nullptr, nullptr, nullptr},
1346:   {nullptr} /* sentinel */
1347: };
1348: 
1349: static PyObject* THPLog2Backward0_self_getter(THPCppFunction *self, void *_unused) {
1350:   HANDLE_TH_ERRORS
1351:   const auto& prop = static_cast<Log2Backward0*>(self->cdata.get())->self_;
1352:   return THPVariable_Wrap(prop.unpack(self->cdata));
1353:   END_HANDLE_TH_ERRORS
1354: }
1355: 
1356: static PyObject* THPLog2Backward0_self_raw_getter(THPCppFunction *self, void *_unused) {
1357:   HANDLE_TH_ERRORS
1358:   const auto& prop = static_cast<Log2Backward0*>(self->cdata.get())->self_;
1359:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1360:   return obj.release().ptr();
```

- EN: The main execution path in this span is carried by `THPLdexpBackward0_other_getter`, `THPVariable_Wrap`, `THPLdexpBackward0_other_raw_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPLdexpBackward0_other_getter`, `THPVariable_Wrap`, `THPLdexpBackward0_other_raw_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1361-1440

```cpp
1361:   END_HANDLE_TH_ERRORS
1362: }
1363: 
1364: static struct PyGetSetDef Log2Backward0_properties[] = {
1365:   THP_FUNCTION_DEFAULT_PROPERTIES,
1366:   {(char*)"_saved_self", (getter)THPLog2Backward0_self_getter, nullptr, nullptr, nullptr},
1367:   {(char*)"_raw_saved_self", (getter)THPLog2Backward0_self_raw_getter, nullptr, nullptr, nullptr},
1368:   {nullptr} /* sentinel */
1369: };
1370: 
1371: static PyObject* THPLogaddexpBackward0_other_getter(THPCppFunction *self, void *_unused) {
1372:   HANDLE_TH_ERRORS
1373:   const auto& prop = static_cast<LogaddexpBackward0*>(self->cdata.get())->other_;
1374:   return THPVariable_Wrap(prop.unpack(self->cdata));
1375:   END_HANDLE_TH_ERRORS
1376: }
1377: 
1378: static PyObject* THPLogaddexpBackward0_other_raw_getter(THPCppFunction *self, void *_unused) {
1379:   HANDLE_TH_ERRORS
1380:   const auto& prop = static_cast<LogaddexpBackward0*>(self->cdata.get())->other_;
1381:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1382:   return obj.release().ptr();
1383:   END_HANDLE_TH_ERRORS
1384: }
1385: 
1386: static PyObject* THPLogaddexpBackward0_self_getter(THPCppFunction *self, void *_unused) {
1387:   HANDLE_TH_ERRORS
1388:   const auto& prop = static_cast<LogaddexpBackward0*>(self->cdata.get())->self_;
1389:   return THPVariable_Wrap(prop.unpack(self->cdata));
1390:   END_HANDLE_TH_ERRORS
1391: }
1392: 
1393: static PyObject* THPLogaddexpBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
1394:   HANDLE_TH_ERRORS
1395:   const auto& prop = static_cast<LogaddexpBackward0*>(self->cdata.get())->self_;
1396:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1397:   return obj.release().ptr();
1398:   END_HANDLE_TH_ERRORS
1399: }
1400: 
1401: static struct PyGetSetDef LogaddexpBackward0_properties[] = {
1402:   THP_FUNCTION_DEFAULT_PROPERTIES,
1403:   {(char*)"_saved_other", (getter)THPLogaddexpBackward0_other_getter, nullptr, nullptr, nullptr},
1404:   {(char*)"_raw_saved_other", (getter)THPLogaddexpBackward0_other_raw_getter, nullptr, nullptr, nullptr},
1405:   {(char*)"_saved_self", (getter)THPLogaddexpBackward0_self_getter, nullptr, nullptr, nullptr},
1406:   {(char*)"_raw_saved_self", (getter)THPLogaddexpBackward0_self_raw_getter, nullptr, nullptr, nullptr},
1407:   {nullptr} /* sentinel */
1408: };
1409: 
1410: static PyObject* THPSpecialZetaBackward0_other_getter(THPCppFunction *self, void *_unused) {
1411:   HANDLE_TH_ERRORS
1412:   const auto& prop = static_cast<SpecialZetaBackward0*>(self->cdata.get())->other_;
1413:   return THPVariable_Wrap(prop.unpack(self->cdata));
1414:   END_HANDLE_TH_ERRORS
1415: }
1416: 
1417: static PyObject* THPSpecialZetaBackward0_other_raw_getter(THPCppFunction *self, void *_unused) {
1418:   HANDLE_TH_ERRORS
1419:   const auto& prop = static_cast<SpecialZetaBackward0*>(self->cdata.get())->other_;
1420:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1421:   return obj.release().ptr();
1422:   END_HANDLE_TH_ERRORS
1423: }
1424: 
1425: static PyObject* THPSpecialZetaBackward0_self_getter(THPCppFunction *self, void *_unused) {
1426:   HANDLE_TH_ERRORS
1427:   const auto& prop = static_cast<SpecialZetaBackward0*>(self->cdata.get())->self_;
1428:   return THPVariable_Wrap(prop.unpack(self->cdata));
1429:   END_HANDLE_TH_ERRORS
1430: }
1431: 
1432: static PyObject* THPSpecialZetaBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
1433:   HANDLE_TH_ERRORS
1434:   const auto& prop = static_cast<SpecialZetaBackward0*>(self->cdata.get())->self_;
1435:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1436:   return obj.release().ptr();
1437:   END_HANDLE_TH_ERRORS
1438: }
1439: 
1440: static struct PyGetSetDef SpecialZetaBackward0_properties[] = {
```

- EN: The main execution path in this span is carried by `THPLogaddexpBackward0_other_getter`, `THPVariable_Wrap`, `THPLogaddexpBackward0_other_raw_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPLogaddexpBackward0_other_getter`, `THPVariable_Wrap`, `THPLogaddexpBackward0_other_raw_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1441-1520

```cpp
1441:   THP_FUNCTION_DEFAULT_PROPERTIES,
1442:   {(char*)"_saved_other", (getter)THPSpecialZetaBackward0_other_getter, nullptr, nullptr, nullptr},
1443:   {(char*)"_raw_saved_other", (getter)THPSpecialZetaBackward0_other_raw_getter, nullptr, nullptr, nullptr},
1444:   {(char*)"_saved_self", (getter)THPSpecialZetaBackward0_self_getter, nullptr, nullptr, nullptr},
1445:   {(char*)"_raw_saved_self", (getter)THPSpecialZetaBackward0_self_raw_getter, nullptr, nullptr, nullptr},
1446:   {nullptr} /* sentinel */
1447: };
1448: 
1449: static PyObject* THPSpecialZetaBackward1_other_getter(THPCppFunction *self, void *_unused) {
1450:   HANDLE_TH_ERRORS
1451:   const auto& prop = static_cast<SpecialZetaBackward1*>(self->cdata.get())->other_;
1452:   return THPVariable_Wrap(prop.unpack(self->cdata));
1453:   END_HANDLE_TH_ERRORS
1454: }
1455: 
1456: static PyObject* THPSpecialZetaBackward1_other_raw_getter(THPCppFunction *self, void *_unused) {
1457:   HANDLE_TH_ERRORS
1458:   const auto& prop = static_cast<SpecialZetaBackward1*>(self->cdata.get())->other_;
1459:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1460:   return obj.release().ptr();
1461:   END_HANDLE_TH_ERRORS
1462: }
1463: 
1464: static PyObject* THPSpecialZetaBackward1_self_getter(THPCppFunction *self, void *_unused) {
1465:   HANDLE_TH_ERRORS
1466:   auto prop = static_cast<SpecialZetaBackward1*>(self->cdata.get())->self;
1467:   if (prop.isComplex()) {
1468:     auto cprop = prop.to<c10::complex<double>>();
1469:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
1470:   } else if (prop.isFloatingPoint()) {
1471:     return PyFloat_FromDouble(prop.to<double>());
1472:   } else if (prop.isIntegral(/*includeBool=*/false)) {
1473:     return PyLong_FromLong(prop.to<int64_t>());
1474:   } else if (prop.isBoolean()) {
1475:     if (prop.to<bool>()) {
1476:       Py_RETURN_TRUE;
1477:     } else {
1478:       Py_RETURN_FALSE;
1479:     }
1480:   } else {
1481:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
1482:     return nullptr;
1483:   }
1484:   END_HANDLE_TH_ERRORS
1485: }
1486: 
1487: static struct PyGetSetDef SpecialZetaBackward1_properties[] = {
1488:   THP_FUNCTION_DEFAULT_PROPERTIES,
1489:   {(char*)"_saved_other", (getter)THPSpecialZetaBackward1_other_getter, nullptr, nullptr, nullptr},
1490:   {(char*)"_raw_saved_other", (getter)THPSpecialZetaBackward1_other_raw_getter, nullptr, nullptr, nullptr},
1491:   {(char*)"_saved_self", (getter)THPSpecialZetaBackward1_self_getter, nullptr, nullptr, nullptr},
1492:   {nullptr} /* sentinel */
1493: };
1494: 
1495: 
1496: 
1497: static struct PyGetSetDef SpecialZetaBackward2_properties[] = {
1498:   THP_FUNCTION_DEFAULT_PROPERTIES,
1499: 
1500:   {nullptr} /* sentinel */
1501: };
1502: 
1503: static PyObject* THPMaskedScatterBackwardBackward0_mask_getter(THPCppFunction *self, void *_unused) {
1504:   HANDLE_TH_ERRORS
1505:   const auto& prop = static_cast<MaskedScatterBackwardBackward0*>(self->cdata.get())->mask_;
1506:   return THPVariable_Wrap(prop.unpack(self->cdata));
1507:   END_HANDLE_TH_ERRORS
1508: }
1509: 
1510: static PyObject* THPMaskedScatterBackwardBackward0_mask_raw_getter(THPCppFunction *self, void *_unused) {
1511:   HANDLE_TH_ERRORS
1512:   const auto& prop = static_cast<MaskedScatterBackwardBackward0*>(self->cdata.get())->mask_;
1513:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1514:   return obj.release().ptr();
1515:   END_HANDLE_TH_ERRORS
1516: }
1517: 
1518: static struct PyGetSetDef MaskedScatterBackwardBackward0_properties[] = {
1519:   THP_FUNCTION_DEFAULT_PROPERTIES,
1520:   {(char*)"_saved_mask", (getter)THPMaskedScatterBackwardBackward0_mask_getter, nullptr, nullptr, nullptr},
```

- EN: The main execution path in this span is carried by `THPSpecialZetaBackward1_other_getter`, `THPVariable_Wrap`, `THPSpecialZetaBackward1_other_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPSpecialZetaBackward1_other_getter`, `THPVariable_Wrap`, `THPSpecialZetaBackward1_other_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1521-1600

```cpp
1521:   {(char*)"_raw_saved_mask", (getter)THPMaskedScatterBackwardBackward0_mask_raw_getter, nullptr, nullptr, nullptr},
1522:   {nullptr} /* sentinel */
1523: };
1524: 
1525: static PyObject* THPMaskedSelectBackward0_mask_getter(THPCppFunction *self, void *_unused) {
1526:   HANDLE_TH_ERRORS
1527:   const auto& prop = static_cast<MaskedSelectBackward0*>(self->cdata.get())->mask_;
1528:   return THPVariable_Wrap(prop.unpack(self->cdata));
1529:   END_HANDLE_TH_ERRORS
1530: }
1531: 
1532: static PyObject* THPMaskedSelectBackward0_mask_raw_getter(THPCppFunction *self, void *_unused) {
1533:   HANDLE_TH_ERRORS
1534:   const auto& prop = static_cast<MaskedSelectBackward0*>(self->cdata.get())->mask_;
1535:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1536:   return obj.release().ptr();
1537:   END_HANDLE_TH_ERRORS
1538: }
1539: 
1540: static PyObject* THPMaskedSelectBackward0_self_getter(THPCppFunction *self, void *_unused) {
1541:   HANDLE_TH_ERRORS
1542:   const auto& prop = static_cast<MaskedSelectBackward0*>(self->cdata.get())->self_;
1543:   return THPVariable_Wrap(prop.unpack(self->cdata));
1544:   END_HANDLE_TH_ERRORS
1545: }
1546: 
1547: static PyObject* THPMaskedSelectBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
1548:   HANDLE_TH_ERRORS
1549:   const auto& prop = static_cast<MaskedSelectBackward0*>(self->cdata.get())->self_;
1550:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1551:   return obj.release().ptr();
1552:   END_HANDLE_TH_ERRORS
1553: }
1554: 
1555: static struct PyGetSetDef MaskedSelectBackward0_properties[] = {
1556:   THP_FUNCTION_DEFAULT_PROPERTIES,
1557:   {(char*)"_saved_mask", (getter)THPMaskedSelectBackward0_mask_getter, nullptr, nullptr, nullptr},
1558:   {(char*)"_raw_saved_mask", (getter)THPMaskedSelectBackward0_mask_raw_getter, nullptr, nullptr, nullptr},
1559:   {(char*)"_saved_self", (getter)THPMaskedSelectBackward0_self_getter, nullptr, nullptr, nullptr},
1560:   {(char*)"_raw_saved_self", (getter)THPMaskedSelectBackward0_self_raw_getter, nullptr, nullptr, nullptr},
1561:   {nullptr} /* sentinel */
1562: };
1563: 
1564: static PyObject* THPFmaxBackward0_other_getter(THPCppFunction *self, void *_unused) {
1565:   HANDLE_TH_ERRORS
1566:   const auto& prop = static_cast<FmaxBackward0*>(self->cdata.get())->other_;
1567:   return THPVariable_Wrap(prop.unpack(self->cdata));
1568:   END_HANDLE_TH_ERRORS
1569: }
1570: 
1571: static PyObject* THPFmaxBackward0_other_raw_getter(THPCppFunction *self, void *_unused) {
1572:   HANDLE_TH_ERRORS
1573:   const auto& prop = static_cast<FmaxBackward0*>(self->cdata.get())->other_;
1574:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1575:   return obj.release().ptr();
1576:   END_HANDLE_TH_ERRORS
1577: }
1578: 
1579: static PyObject* THPFmaxBackward0_self_getter(THPCppFunction *self, void *_unused) {
1580:   HANDLE_TH_ERRORS
1581:   const auto& prop = static_cast<FmaxBackward0*>(self->cdata.get())->self_;
1582:   return THPVariable_Wrap(prop.unpack(self->cdata));
1583:   END_HANDLE_TH_ERRORS
1584: }
1585: 
1586: static PyObject* THPFmaxBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
1587:   HANDLE_TH_ERRORS
1588:   const auto& prop = static_cast<FmaxBackward0*>(self->cdata.get())->self_;
1589:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1590:   return obj.release().ptr();
1591:   END_HANDLE_TH_ERRORS
1592: }
1593: 
1594: static struct PyGetSetDef FmaxBackward0_properties[] = {
1595:   THP_FUNCTION_DEFAULT_PROPERTIES,
1596:   {(char*)"_saved_other", (getter)THPFmaxBackward0_other_getter, nullptr, nullptr, nullptr},
1597:   {(char*)"_raw_saved_other", (getter)THPFmaxBackward0_other_raw_getter, nullptr, nullptr, nullptr},
1598:   {(char*)"_saved_self", (getter)THPFmaxBackward0_self_getter, nullptr, nullptr, nullptr},
1599:   {(char*)"_raw_saved_self", (getter)THPFmaxBackward0_self_raw_getter, nullptr, nullptr, nullptr},
1600:   {nullptr} /* sentinel */
```

- EN: The main execution path in this span is carried by `THPMaskedSelectBackward0_mask_getter`, `THPVariable_Wrap`, `THPMaskedSelectBackward0_mask_raw_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPMaskedSelectBackward0_mask_getter`, `THPVariable_Wrap`, `THPMaskedSelectBackward0_mask_raw_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1601-1680

```cpp
1601: };
1602: 
1603: static PyObject* THPNativeBatchNormLegitBackward0_eps_getter(THPCppFunction *self, void *_unused) {
1604:   HANDLE_TH_ERRORS
1605:   auto prop = static_cast<NativeBatchNormLegitBackward0*>(self->cdata.get())->eps;
1606:   return PyFloat_FromDouble((double) prop);
1607:   END_HANDLE_TH_ERRORS
1608: }
1609: 
1610: static PyObject* THPNativeBatchNormLegitBackward0_input_getter(THPCppFunction *self, void *_unused) {
1611:   HANDLE_TH_ERRORS
1612:   const auto& prop = static_cast<NativeBatchNormLegitBackward0*>(self->cdata.get())->input_;
1613:   return THPVariable_Wrap(prop.unpack(self->cdata));
1614:   END_HANDLE_TH_ERRORS
1615: }
1616: 
1617: static PyObject* THPNativeBatchNormLegitBackward0_input_raw_getter(THPCppFunction *self, void *_unused) {
1618:   HANDLE_TH_ERRORS
1619:   const auto& prop = static_cast<NativeBatchNormLegitBackward0*>(self->cdata.get())->input_;
1620:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1621:   return obj.release().ptr();
1622:   END_HANDLE_TH_ERRORS
1623: }
1624: 
1625: static PyObject* THPNativeBatchNormLegitBackward0_running_mean_getter(THPCppFunction *self, void *_unused) {
1626:   HANDLE_TH_ERRORS
1627:   const auto& prop = static_cast<NativeBatchNormLegitBackward0*>(self->cdata.get())->running_mean_;
1628:   return THPVariable_Wrap(prop.unpack(self->cdata));
1629:   END_HANDLE_TH_ERRORS
1630: }
1631: 
1632: static PyObject* THPNativeBatchNormLegitBackward0_running_mean_raw_getter(THPCppFunction *self, void *_unused) {
1633:   HANDLE_TH_ERRORS
1634:   const auto& prop = static_cast<NativeBatchNormLegitBackward0*>(self->cdata.get())->running_mean_;
1635:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1636:   return obj.release().ptr();
1637:   END_HANDLE_TH_ERRORS
1638: }
1639: 
1640: static PyObject* THPNativeBatchNormLegitBackward0_running_var_getter(THPCppFunction *self, void *_unused) {
1641:   HANDLE_TH_ERRORS
1642:   const auto& prop = static_cast<NativeBatchNormLegitBackward0*>(self->cdata.get())->running_var_;
1643:   return THPVariable_Wrap(prop.unpack(self->cdata));
1644:   END_HANDLE_TH_ERRORS
1645: }
1646: 
1647: static PyObject* THPNativeBatchNormLegitBackward0_running_var_raw_getter(THPCppFunction *self, void *_unused) {
1648:   HANDLE_TH_ERRORS
1649:   const auto& prop = static_cast<NativeBatchNormLegitBackward0*>(self->cdata.get())->running_var_;
1650:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1651:   return obj.release().ptr();
1652:   END_HANDLE_TH_ERRORS
1653: }
1654: 
1655: static PyObject* THPNativeBatchNormLegitBackward0_training_getter(THPCppFunction *self, void *_unused) {
1656:   HANDLE_TH_ERRORS
1657:   auto prop = static_cast<NativeBatchNormLegitBackward0*>(self->cdata.get())->training;
1658:   if (prop) {
1659:     Py_RETURN_TRUE;
1660:   } else {
1661:     Py_RETURN_FALSE;
1662:   }
1663:   END_HANDLE_TH_ERRORS
1664: }
1665: 
1666: static PyObject* THPNativeBatchNormLegitBackward0_weight_getter(THPCppFunction *self, void *_unused) {
1667:   HANDLE_TH_ERRORS
1668:   const auto& prop = static_cast<NativeBatchNormLegitBackward0*>(self->cdata.get())->weight_;
1669:   return THPVariable_Wrap(prop.unpack(self->cdata));
1670:   END_HANDLE_TH_ERRORS
1671: }
1672: 
1673: static PyObject* THPNativeBatchNormLegitBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
1674:   HANDLE_TH_ERRORS
1675:   const auto& prop = static_cast<NativeBatchNormLegitBackward0*>(self->cdata.get())->weight_;
1676:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1677:   return obj.release().ptr();
1678:   END_HANDLE_TH_ERRORS
1679: }
1680: 
```

- EN: The main execution path in this span is carried by `THPNativeBatchNormLegitBackward0_eps_getter`, `PyFloat_FromDouble`, `THPNativeBatchNormLegitBackward0_input_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPNativeBatchNormLegitBackward0_eps_getter`, `PyFloat_FromDouble`, `THPNativeBatchNormLegitBackward0_input_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1681-1760

```cpp
1681: static PyObject* THPNativeBatchNormLegitBackward0_result1_getter(THPCppFunction *self, void *_unused) {
1682:   HANDLE_TH_ERRORS
1683:   const auto& prop = static_cast<NativeBatchNormLegitBackward0*>(self->cdata.get())->result1_;
1684:   return THPVariable_Wrap(prop.unpack(self->cdata));
1685:   END_HANDLE_TH_ERRORS
1686: }
1687: 
1688: static PyObject* THPNativeBatchNormLegitBackward0_result1_raw_getter(THPCppFunction *self, void *_unused) {
1689:   HANDLE_TH_ERRORS
1690:   const auto& prop = static_cast<NativeBatchNormLegitBackward0*>(self->cdata.get())->result1_;
1691:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1692:   return obj.release().ptr();
1693:   END_HANDLE_TH_ERRORS
1694: }
1695: 
1696: static PyObject* THPNativeBatchNormLegitBackward0_result2_getter(THPCppFunction *self, void *_unused) {
1697:   HANDLE_TH_ERRORS
1698:   const auto& prop = static_cast<NativeBatchNormLegitBackward0*>(self->cdata.get())->result2_;
1699:   return THPVariable_Wrap(prop.unpack(self->cdata));
1700:   END_HANDLE_TH_ERRORS
1701: }
1702: 
1703: static PyObject* THPNativeBatchNormLegitBackward0_result2_raw_getter(THPCppFunction *self, void *_unused) {
1704:   HANDLE_TH_ERRORS
1705:   const auto& prop = static_cast<NativeBatchNormLegitBackward0*>(self->cdata.get())->result2_;
1706:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1707:   return obj.release().ptr();
1708:   END_HANDLE_TH_ERRORS
1709: }
1710: 
1711: static struct PyGetSetDef NativeBatchNormLegitBackward0_properties[] = {
1712:   THP_FUNCTION_DEFAULT_PROPERTIES,
1713:   {(char*)"_saved_eps", (getter)THPNativeBatchNormLegitBackward0_eps_getter, nullptr, nullptr, nullptr},
1714:   {(char*)"_saved_input", (getter)THPNativeBatchNormLegitBackward0_input_getter, nullptr, nullptr, nullptr},
1715:   {(char*)"_raw_saved_input", (getter)THPNativeBatchNormLegitBackward0_input_raw_getter, nullptr, nullptr, nullptr},
1716:   {(char*)"_saved_running_mean", (getter)THPNativeBatchNormLegitBackward0_running_mean_getter, nullptr, nullptr, nullptr},
1717:   {(char*)"_raw_saved_running_mean", (getter)THPNativeBatchNormLegitBackward0_running_mean_raw_getter, nullptr, nullptr, nullptr},
1718:   {(char*)"_saved_running_var", (getter)THPNativeBatchNormLegitBackward0_running_var_getter, nullptr, nullptr, nullptr},
1719:   {(char*)"_raw_saved_running_var", (getter)THPNativeBatchNormLegitBackward0_running_var_raw_getter, nullptr, nullptr, nullptr},
1720:   {(char*)"_saved_training", (getter)THPNativeBatchNormLegitBackward0_training_getter, nullptr, nullptr, nullptr},
1721:   {(char*)"_saved_weight", (getter)THPNativeBatchNormLegitBackward0_weight_getter, nullptr, nullptr, nullptr},
1722:   {(char*)"_raw_saved_weight", (getter)THPNativeBatchNormLegitBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
1723:   {(char*)"_saved_result1", (getter)THPNativeBatchNormLegitBackward0_result1_getter, nullptr, nullptr, nullptr},
1724:   {(char*)"_raw_saved_result1", (getter)THPNativeBatchNormLegitBackward0_result1_raw_getter, nullptr, nullptr, nullptr},
1725:   {(char*)"_saved_result2", (getter)THPNativeBatchNormLegitBackward0_result2_getter, nullptr, nullptr, nullptr},
1726:   {(char*)"_raw_saved_result2", (getter)THPNativeBatchNormLegitBackward0_result2_raw_getter, nullptr, nullptr, nullptr},
1727:   {nullptr} /* sentinel */
1728: };
1729: 
1730: static PyObject* THPNativeBatchNormLegitBackward1_eps_getter(THPCppFunction *self, void *_unused) {
1731:   HANDLE_TH_ERRORS
1732:   auto prop = static_cast<NativeBatchNormLegitBackward1*>(self->cdata.get())->eps;
1733:   return PyFloat_FromDouble((double) prop);
1734:   END_HANDLE_TH_ERRORS
1735: }
1736: 
1737: static PyObject* THPNativeBatchNormLegitBackward1_input_getter(THPCppFunction *self, void *_unused) {
1738:   HANDLE_TH_ERRORS
1739:   const auto& prop = static_cast<NativeBatchNormLegitBackward1*>(self->cdata.get())->input_;
1740:   return THPVariable_Wrap(prop.unpack(self->cdata));
1741:   END_HANDLE_TH_ERRORS
1742: }
1743: 
1744: static PyObject* THPNativeBatchNormLegitBackward1_input_raw_getter(THPCppFunction *self, void *_unused) {
1745:   HANDLE_TH_ERRORS
1746:   const auto& prop = static_cast<NativeBatchNormLegitBackward1*>(self->cdata.get())->input_;
1747:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1748:   return obj.release().ptr();
1749:   END_HANDLE_TH_ERRORS
1750: }
1751: 
1752: static PyObject* THPNativeBatchNormLegitBackward1_training_getter(THPCppFunction *self, void *_unused) {
1753:   HANDLE_TH_ERRORS
1754:   auto prop = static_cast<NativeBatchNormLegitBackward1*>(self->cdata.get())->training;
1755:   if (prop) {
1756:     Py_RETURN_TRUE;
1757:   } else {
1758:     Py_RETURN_FALSE;
1759:   }
1760:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPNativeBatchNormLegitBackward0_result1_getter`, `THPVariable_Wrap`, `THPNativeBatchNormLegitBackward0_result1_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPNativeBatchNormLegitBackward0_result1_getter`, `THPVariable_Wrap`, `THPNativeBatchNormLegitBackward0_result1_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1761-1840

```cpp
1761: }
1762: 
1763: static PyObject* THPNativeBatchNormLegitBackward1_weight_getter(THPCppFunction *self, void *_unused) {
1764:   HANDLE_TH_ERRORS
1765:   const auto& prop = static_cast<NativeBatchNormLegitBackward1*>(self->cdata.get())->weight_;
1766:   return THPVariable_Wrap(prop.unpack(self->cdata));
1767:   END_HANDLE_TH_ERRORS
1768: }
1769: 
1770: static PyObject* THPNativeBatchNormLegitBackward1_weight_raw_getter(THPCppFunction *self, void *_unused) {
1771:   HANDLE_TH_ERRORS
1772:   const auto& prop = static_cast<NativeBatchNormLegitBackward1*>(self->cdata.get())->weight_;
1773:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1774:   return obj.release().ptr();
1775:   END_HANDLE_TH_ERRORS
1776: }
1777: 
1778: static PyObject* THPNativeBatchNormLegitBackward1_result1_getter(THPCppFunction *self, void *_unused) {
1779:   HANDLE_TH_ERRORS
1780:   const auto& prop = static_cast<NativeBatchNormLegitBackward1*>(self->cdata.get())->result1_;
1781:   return THPVariable_Wrap(prop.unpack(self->cdata));
1782:   END_HANDLE_TH_ERRORS
1783: }
1784: 
1785: static PyObject* THPNativeBatchNormLegitBackward1_result1_raw_getter(THPCppFunction *self, void *_unused) {
1786:   HANDLE_TH_ERRORS
1787:   const auto& prop = static_cast<NativeBatchNormLegitBackward1*>(self->cdata.get())->result1_;
1788:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1789:   return obj.release().ptr();
1790:   END_HANDLE_TH_ERRORS
1791: }
1792: 
1793: static PyObject* THPNativeBatchNormLegitBackward1_result2_getter(THPCppFunction *self, void *_unused) {
1794:   HANDLE_TH_ERRORS
1795:   const auto& prop = static_cast<NativeBatchNormLegitBackward1*>(self->cdata.get())->result2_;
1796:   return THPVariable_Wrap(prop.unpack(self->cdata));
1797:   END_HANDLE_TH_ERRORS
1798: }
1799: 
1800: static PyObject* THPNativeBatchNormLegitBackward1_result2_raw_getter(THPCppFunction *self, void *_unused) {
1801:   HANDLE_TH_ERRORS
1802:   const auto& prop = static_cast<NativeBatchNormLegitBackward1*>(self->cdata.get())->result2_;
1803:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1804:   return obj.release().ptr();
1805:   END_HANDLE_TH_ERRORS
1806: }
1807: 
1808: static struct PyGetSetDef NativeBatchNormLegitBackward1_properties[] = {
1809:   THP_FUNCTION_DEFAULT_PROPERTIES,
1810:   {(char*)"_saved_eps", (getter)THPNativeBatchNormLegitBackward1_eps_getter, nullptr, nullptr, nullptr},
1811:   {(char*)"_saved_input", (getter)THPNativeBatchNormLegitBackward1_input_getter, nullptr, nullptr, nullptr},
1812:   {(char*)"_raw_saved_input", (getter)THPNativeBatchNormLegitBackward1_input_raw_getter, nullptr, nullptr, nullptr},
1813:   {(char*)"_saved_training", (getter)THPNativeBatchNormLegitBackward1_training_getter, nullptr, nullptr, nullptr},
1814:   {(char*)"_saved_weight", (getter)THPNativeBatchNormLegitBackward1_weight_getter, nullptr, nullptr, nullptr},
1815:   {(char*)"_raw_saved_weight", (getter)THPNativeBatchNormLegitBackward1_weight_raw_getter, nullptr, nullptr, nullptr},
1816:   {(char*)"_saved_result1", (getter)THPNativeBatchNormLegitBackward1_result1_getter, nullptr, nullptr, nullptr},
1817:   {(char*)"_raw_saved_result1", (getter)THPNativeBatchNormLegitBackward1_result1_raw_getter, nullptr, nullptr, nullptr},
1818:   {(char*)"_saved_result2", (getter)THPNativeBatchNormLegitBackward1_result2_getter, nullptr, nullptr, nullptr},
1819:   {(char*)"_raw_saved_result2", (getter)THPNativeBatchNormLegitBackward1_result2_raw_getter, nullptr, nullptr, nullptr},
1820:   {nullptr} /* sentinel */
1821: };
1822: 
1823: static PyObject* THPNativeLayerNormBackwardBackward0_grad_out_getter(THPCppFunction *self, void *_unused) {
1824:   HANDLE_TH_ERRORS
1825:   const auto& prop = static_cast<NativeLayerNormBackwardBackward0*>(self->cdata.get())->grad_out_;
1826:   return THPVariable_Wrap(prop.unpack(self->cdata));
1827:   END_HANDLE_TH_ERRORS
1828: }
1829: 
1830: static PyObject* THPNativeLayerNormBackwardBackward0_grad_out_raw_getter(THPCppFunction *self, void *_unused) {
1831:   HANDLE_TH_ERRORS
1832:   const auto& prop = static_cast<NativeLayerNormBackwardBackward0*>(self->cdata.get())->grad_out_;
1833:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1834:   return obj.release().ptr();
1835:   END_HANDLE_TH_ERRORS
1836: }
1837: 
1838: static PyObject* THPNativeLayerNormBackwardBackward0_input_getter(THPCppFunction *self, void *_unused) {
1839:   HANDLE_TH_ERRORS
1840:   const auto& prop = static_cast<NativeLayerNormBackwardBackward0*>(self->cdata.get())->input_;
```

- EN: The main execution path in this span is carried by `THPNativeBatchNormLegitBackward1_weight_getter`, `THPVariable_Wrap`, `THPNativeBatchNormLegitBackward1_weight_raw_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPNativeBatchNormLegitBackward1_weight_getter`, `THPVariable_Wrap`, `THPNativeBatchNormLegitBackward1_weight_raw_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1841-1920

```cpp
1841:   return THPVariable_Wrap(prop.unpack(self->cdata));
1842:   END_HANDLE_TH_ERRORS
1843: }
1844: 
1845: static PyObject* THPNativeLayerNormBackwardBackward0_input_raw_getter(THPCppFunction *self, void *_unused) {
1846:   HANDLE_TH_ERRORS
1847:   const auto& prop = static_cast<NativeLayerNormBackwardBackward0*>(self->cdata.get())->input_;
1848:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1849:   return obj.release().ptr();
1850:   END_HANDLE_TH_ERRORS
1851: }
1852: 
1853: static PyObject* THPNativeLayerNormBackwardBackward0_mean_getter(THPCppFunction *self, void *_unused) {
1854:   HANDLE_TH_ERRORS
1855:   const auto& prop = static_cast<NativeLayerNormBackwardBackward0*>(self->cdata.get())->mean_;
1856:   return THPVariable_Wrap(prop.unpack(self->cdata));
1857:   END_HANDLE_TH_ERRORS
1858: }
1859: 
1860: static PyObject* THPNativeLayerNormBackwardBackward0_mean_raw_getter(THPCppFunction *self, void *_unused) {
1861:   HANDLE_TH_ERRORS
1862:   const auto& prop = static_cast<NativeLayerNormBackwardBackward0*>(self->cdata.get())->mean_;
1863:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1864:   return obj.release().ptr();
1865:   END_HANDLE_TH_ERRORS
1866: }
1867: 
1868: static PyObject* THPNativeLayerNormBackwardBackward0_normalized_shape_getter(THPCppFunction *self, void *_unused) {
1869:   HANDLE_TH_ERRORS
1870:   auto prop = static_cast<NativeLayerNormBackwardBackward0*>(self->cdata.get())->normalized_shape;
1871:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
1872:   for (auto i : c10::irange(prop.size())) {
1873:       auto si = prop[i];
1874:       if (auto m = si.maybe_as_int()) {
1875:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
1876:       } else {
1877:         auto py_symint = py::cast(si).release().ptr();
1878:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
1879:       }
1880:   }
1881:   return tup;
1882:   END_HANDLE_TH_ERRORS
1883: }
1884: 
1885: static PyObject* THPNativeLayerNormBackwardBackward0_rstd_getter(THPCppFunction *self, void *_unused) {
1886:   HANDLE_TH_ERRORS
1887:   const auto& prop = static_cast<NativeLayerNormBackwardBackward0*>(self->cdata.get())->rstd_;
1888:   return THPVariable_Wrap(prop.unpack(self->cdata));
1889:   END_HANDLE_TH_ERRORS
1890: }
1891: 
1892: static PyObject* THPNativeLayerNormBackwardBackward0_rstd_raw_getter(THPCppFunction *self, void *_unused) {
1893:   HANDLE_TH_ERRORS
1894:   const auto& prop = static_cast<NativeLayerNormBackwardBackward0*>(self->cdata.get())->rstd_;
1895:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1896:   return obj.release().ptr();
1897:   END_HANDLE_TH_ERRORS
1898: }
1899: 
1900: static PyObject* THPNativeLayerNormBackwardBackward0_weight_getter(THPCppFunction *self, void *_unused) {
1901:   HANDLE_TH_ERRORS
1902:   const auto& prop = static_cast<NativeLayerNormBackwardBackward0*>(self->cdata.get())->weight_;
1903:   return THPVariable_Wrap(prop.unpack(self->cdata));
1904:   END_HANDLE_TH_ERRORS
1905: }
1906: 
1907: static PyObject* THPNativeLayerNormBackwardBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
1908:   HANDLE_TH_ERRORS
1909:   const auto& prop = static_cast<NativeLayerNormBackwardBackward0*>(self->cdata.get())->weight_;
1910:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1911:   return obj.release().ptr();
1912:   END_HANDLE_TH_ERRORS
1913: }
1914: 
1915: static struct PyGetSetDef NativeLayerNormBackwardBackward0_properties[] = {
1916:   THP_FUNCTION_DEFAULT_PROPERTIES,
1917:   {(char*)"_saved_grad_out", (getter)THPNativeLayerNormBackwardBackward0_grad_out_getter, nullptr, nullptr, nullptr},
1918:   {(char*)"_raw_saved_grad_out", (getter)THPNativeLayerNormBackwardBackward0_grad_out_raw_getter, nullptr, nullptr, nullptr},
1919:   {(char*)"_saved_input", (getter)THPNativeLayerNormBackwardBackward0_input_getter, nullptr, nullptr, nullptr},
1920:   {(char*)"_raw_saved_input", (getter)THPNativeLayerNormBackwardBackward0_input_raw_getter, nullptr, nullptr, nullptr},
```

- EN: The main execution path in this span is carried by `THPVariable_Wrap`, `THPNativeLayerNormBackwardBackward0_input_raw_getter`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_Wrap`, `THPNativeLayerNormBackwardBackward0_input_raw_getter`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1921-2000

```cpp
1921:   {(char*)"_saved_mean", (getter)THPNativeLayerNormBackwardBackward0_mean_getter, nullptr, nullptr, nullptr},
1922:   {(char*)"_raw_saved_mean", (getter)THPNativeLayerNormBackwardBackward0_mean_raw_getter, nullptr, nullptr, nullptr},
1923:   {(char*)"_saved_normalized_shape", (getter)THPNativeLayerNormBackwardBackward0_normalized_shape_getter, nullptr, nullptr, nullptr},
1924:   {(char*)"_saved_rstd", (getter)THPNativeLayerNormBackwardBackward0_rstd_getter, nullptr, nullptr, nullptr},
1925:   {(char*)"_raw_saved_rstd", (getter)THPNativeLayerNormBackwardBackward0_rstd_raw_getter, nullptr, nullptr, nullptr},
1926:   {(char*)"_saved_weight", (getter)THPNativeLayerNormBackwardBackward0_weight_getter, nullptr, nullptr, nullptr},
1927:   {(char*)"_raw_saved_weight", (getter)THPNativeLayerNormBackwardBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
1928:   {nullptr} /* sentinel */
1929: };
1930: 
1931: static PyObject* THPBatchNormWithUpdateBackward0_eps_getter(THPCppFunction *self, void *_unused) {
1932:   HANDLE_TH_ERRORS
1933:   auto prop = static_cast<BatchNormWithUpdateBackward0*>(self->cdata.get())->eps;
1934:   return PyFloat_FromDouble((double) prop);
1935:   END_HANDLE_TH_ERRORS
1936: }
1937: 
1938: static PyObject* THPBatchNormWithUpdateBackward0_input_getter(THPCppFunction *self, void *_unused) {
1939:   HANDLE_TH_ERRORS
1940:   const auto& prop = static_cast<BatchNormWithUpdateBackward0*>(self->cdata.get())->input_;
1941:   return THPVariable_Wrap(prop.unpack(self->cdata));
1942:   END_HANDLE_TH_ERRORS
1943: }
1944: 
1945: static PyObject* THPBatchNormWithUpdateBackward0_input_raw_getter(THPCppFunction *self, void *_unused) {
1946:   HANDLE_TH_ERRORS
1947:   const auto& prop = static_cast<BatchNormWithUpdateBackward0*>(self->cdata.get())->input_;
1948:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1949:   return obj.release().ptr();
1950:   END_HANDLE_TH_ERRORS
1951: }
1952: 
1953: static PyObject* THPBatchNormWithUpdateBackward0_running_mean_getter(THPCppFunction *self, void *_unused) {
1954:   HANDLE_TH_ERRORS
1955:   const auto& prop = static_cast<BatchNormWithUpdateBackward0*>(self->cdata.get())->running_mean_;
1956:   return THPVariable_Wrap(prop.unpack(self->cdata));
1957:   END_HANDLE_TH_ERRORS
1958: }
1959: 
1960: static PyObject* THPBatchNormWithUpdateBackward0_running_mean_raw_getter(THPCppFunction *self, void *_unused) {
1961:   HANDLE_TH_ERRORS
1962:   const auto& prop = static_cast<BatchNormWithUpdateBackward0*>(self->cdata.get())->running_mean_;
1963:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1964:   return obj.release().ptr();
1965:   END_HANDLE_TH_ERRORS
1966: }
1967: 
1968: static PyObject* THPBatchNormWithUpdateBackward0_running_var_getter(THPCppFunction *self, void *_unused) {
1969:   HANDLE_TH_ERRORS
1970:   const auto& prop = static_cast<BatchNormWithUpdateBackward0*>(self->cdata.get())->running_var_;
1971:   return THPVariable_Wrap(prop.unpack(self->cdata));
1972:   END_HANDLE_TH_ERRORS
1973: }
1974: 
1975: static PyObject* THPBatchNormWithUpdateBackward0_running_var_raw_getter(THPCppFunction *self, void *_unused) {
1976:   HANDLE_TH_ERRORS
1977:   const auto& prop = static_cast<BatchNormWithUpdateBackward0*>(self->cdata.get())->running_var_;
1978:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1979:   return obj.release().ptr();
1980:   END_HANDLE_TH_ERRORS
1981: }
1982: 
1983: static PyObject* THPBatchNormWithUpdateBackward0_weight_getter(THPCppFunction *self, void *_unused) {
1984:   HANDLE_TH_ERRORS
1985:   const auto& prop = static_cast<BatchNormWithUpdateBackward0*>(self->cdata.get())->weight_;
1986:   return THPVariable_Wrap(prop.unpack(self->cdata));
1987:   END_HANDLE_TH_ERRORS
1988: }
1989: 
1990: static PyObject* THPBatchNormWithUpdateBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
1991:   HANDLE_TH_ERRORS
1992:   const auto& prop = static_cast<BatchNormWithUpdateBackward0*>(self->cdata.get())->weight_;
1993:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1994:   return obj.release().ptr();
1995:   END_HANDLE_TH_ERRORS
1996: }
1997: 
1998: static PyObject* THPBatchNormWithUpdateBackward0_result1_getter(THPCppFunction *self, void *_unused) {
1999:   HANDLE_TH_ERRORS
2000:   const auto& prop = static_cast<BatchNormWithUpdateBackward0*>(self->cdata.get())->result1_;
```

- EN: The main execution path in this span is carried by `THPBatchNormWithUpdateBackward0_eps_getter`, `PyFloat_FromDouble`, `THPBatchNormWithUpdateBackward0_input_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPBatchNormWithUpdateBackward0_eps_getter`, `PyFloat_FromDouble`, `THPBatchNormWithUpdateBackward0_input_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2001-2080

```cpp
2001:   return THPVariable_Wrap(prop.unpack(self->cdata));
2002:   END_HANDLE_TH_ERRORS
2003: }
2004: 
2005: static PyObject* THPBatchNormWithUpdateBackward0_result1_raw_getter(THPCppFunction *self, void *_unused) {
2006:   HANDLE_TH_ERRORS
2007:   const auto& prop = static_cast<BatchNormWithUpdateBackward0*>(self->cdata.get())->result1_;
2008:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2009:   return obj.release().ptr();
2010:   END_HANDLE_TH_ERRORS
2011: }
2012: 
2013: static PyObject* THPBatchNormWithUpdateBackward0_result2_getter(THPCppFunction *self, void *_unused) {
2014:   HANDLE_TH_ERRORS
2015:   const auto& prop = static_cast<BatchNormWithUpdateBackward0*>(self->cdata.get())->result2_;
2016:   return THPVariable_Wrap(prop.unpack(self->cdata));
2017:   END_HANDLE_TH_ERRORS
2018: }
2019: 
2020: static PyObject* THPBatchNormWithUpdateBackward0_result2_raw_getter(THPCppFunction *self, void *_unused) {
2021:   HANDLE_TH_ERRORS
2022:   const auto& prop = static_cast<BatchNormWithUpdateBackward0*>(self->cdata.get())->result2_;
2023:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2024:   return obj.release().ptr();
2025:   END_HANDLE_TH_ERRORS
2026: }
2027: 
2028: static PyObject* THPBatchNormWithUpdateBackward0_result3_getter(THPCppFunction *self, void *_unused) {
2029:   HANDLE_TH_ERRORS
2030:   const auto& prop = static_cast<BatchNormWithUpdateBackward0*>(self->cdata.get())->result3_;
2031:   return THPVariable_Wrap(prop.unpack(self->cdata));
2032:   END_HANDLE_TH_ERRORS
2033: }
2034: 
2035: static PyObject* THPBatchNormWithUpdateBackward0_result3_raw_getter(THPCppFunction *self, void *_unused) {
2036:   HANDLE_TH_ERRORS
2037:   const auto& prop = static_cast<BatchNormWithUpdateBackward0*>(self->cdata.get())->result3_;
2038:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2039:   return obj.release().ptr();
2040:   END_HANDLE_TH_ERRORS
2041: }
2042: 
2043: static struct PyGetSetDef BatchNormWithUpdateBackward0_properties[] = {
2044:   THP_FUNCTION_DEFAULT_PROPERTIES,
2045:   {(char*)"_saved_eps", (getter)THPBatchNormWithUpdateBackward0_eps_getter, nullptr, nullptr, nullptr},
2046:   {(char*)"_saved_input", (getter)THPBatchNormWithUpdateBackward0_input_getter, nullptr, nullptr, nullptr},
2047:   {(char*)"_raw_saved_input", (getter)THPBatchNormWithUpdateBackward0_input_raw_getter, nullptr, nullptr, nullptr},
2048:   {(char*)"_saved_running_mean", (getter)THPBatchNormWithUpdateBackward0_running_mean_getter, nullptr, nullptr, nullptr},
2049:   {(char*)"_raw_saved_running_mean", (getter)THPBatchNormWithUpdateBackward0_running_mean_raw_getter, nullptr, nullptr, nullptr},
2050:   {(char*)"_saved_running_var", (getter)THPBatchNormWithUpdateBackward0_running_var_getter, nullptr, nullptr, nullptr},
2051:   {(char*)"_raw_saved_running_var", (getter)THPBatchNormWithUpdateBackward0_running_var_raw_getter, nullptr, nullptr, nullptr},
2052:   {(char*)"_saved_weight", (getter)THPBatchNormWithUpdateBackward0_weight_getter, nullptr, nullptr, nullptr},
2053:   {(char*)"_raw_saved_weight", (getter)THPBatchNormWithUpdateBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
2054:   {(char*)"_saved_result1", (getter)THPBatchNormWithUpdateBackward0_result1_getter, nullptr, nullptr, nullptr},
2055:   {(char*)"_raw_saved_result1", (getter)THPBatchNormWithUpdateBackward0_result1_raw_getter, nullptr, nullptr, nullptr},
2056:   {(char*)"_saved_result2", (getter)THPBatchNormWithUpdateBackward0_result2_getter, nullptr, nullptr, nullptr},
2057:   {(char*)"_raw_saved_result2", (getter)THPBatchNormWithUpdateBackward0_result2_raw_getter, nullptr, nullptr, nullptr},
2058:   {(char*)"_saved_result3", (getter)THPBatchNormWithUpdateBackward0_result3_getter, nullptr, nullptr, nullptr},
2059:   {(char*)"_raw_saved_result3", (getter)THPBatchNormWithUpdateBackward0_result3_raw_getter, nullptr, nullptr, nullptr},
2060:   {nullptr} /* sentinel */
2061: };
2062: 
2063: static PyObject* THPNormBackward0_p_getter(THPCppFunction *self, void *_unused) {
2064:   HANDLE_TH_ERRORS
2065:   auto prop = static_cast<NormBackward0*>(self->cdata.get())->p;
2066:   if (prop.isComplex()) {
2067:     auto cprop = prop.to<c10::complex<double>>();
2068:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
2069:   } else if (prop.isFloatingPoint()) {
2070:     return PyFloat_FromDouble(prop.to<double>());
2071:   } else if (prop.isIntegral(/*includeBool=*/false)) {
2072:     return PyLong_FromLong(prop.to<int64_t>());
2073:   } else if (prop.isBoolean()) {
2074:     if (prop.to<bool>()) {
2075:       Py_RETURN_TRUE;
2076:     } else {
2077:       Py_RETURN_FALSE;
2078:     }
2079:   } else {
2080:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
```

- EN: The main execution path in this span is carried by `THPVariable_Wrap`, `THPBatchNormWithUpdateBackward0_result1_raw_getter`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_Wrap`, `THPBatchNormWithUpdateBackward0_result1_raw_getter`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2081-2160

```cpp
2081:     return nullptr;
2082:   }
2083:   END_HANDLE_TH_ERRORS
2084: }
2085: 
2086: static PyObject* THPNormBackward0_self_getter(THPCppFunction *self, void *_unused) {
2087:   HANDLE_TH_ERRORS
2088:   const auto& prop = static_cast<NormBackward0*>(self->cdata.get())->self_;
2089:   return THPVariable_Wrap(prop.unpack(self->cdata));
2090:   END_HANDLE_TH_ERRORS
2091: }
2092: 
2093: static PyObject* THPNormBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
2094:   HANDLE_TH_ERRORS
2095:   const auto& prop = static_cast<NormBackward0*>(self->cdata.get())->self_;
2096:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2097:   return obj.release().ptr();
2098:   END_HANDLE_TH_ERRORS
2099: }
2100: 
2101: static PyObject* THPNormBackward0_result_getter(THPCppFunction *self, void *_unused) {
2102:   HANDLE_TH_ERRORS
2103:   const auto& prop = static_cast<NormBackward0*>(self->cdata.get())->result_;
2104:   return THPVariable_Wrap(prop.unpack(self->cdata));
2105:   END_HANDLE_TH_ERRORS
2106: }
2107: 
2108: static PyObject* THPNormBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
2109:   HANDLE_TH_ERRORS
2110:   const auto& prop = static_cast<NormBackward0*>(self->cdata.get())->result_;
2111:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2112:   return obj.release().ptr();
2113:   END_HANDLE_TH_ERRORS
2114: }
2115: 
2116: static struct PyGetSetDef NormBackward0_properties[] = {
2117:   THP_FUNCTION_DEFAULT_PROPERTIES,
2118:   {(char*)"_saved_p", (getter)THPNormBackward0_p_getter, nullptr, nullptr, nullptr},
2119:   {(char*)"_saved_self", (getter)THPNormBackward0_self_getter, nullptr, nullptr, nullptr},
2120:   {(char*)"_raw_saved_self", (getter)THPNormBackward0_self_raw_getter, nullptr, nullptr, nullptr},
2121:   {(char*)"_saved_result", (getter)THPNormBackward0_result_getter, nullptr, nullptr, nullptr},
2122:   {(char*)"_raw_saved_result", (getter)THPNormBackward0_result_raw_getter, nullptr, nullptr, nullptr},
2123:   {nullptr} /* sentinel */
2124: };
2125: 
2126: static PyObject* THPNormBackward1_dim_getter(THPCppFunction *self, void *_unused) {
2127:   HANDLE_TH_ERRORS
2128:   auto prop = static_cast<NormBackward1*>(self->cdata.get())->dim;
2129:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
2130:   for (auto i : c10::irange(prop.size())) {
2131:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
2132:   }
2133:   return tup;
2134:   END_HANDLE_TH_ERRORS
2135: }
2136: 
2137: static PyObject* THPNormBackward1_keepdim_getter(THPCppFunction *self, void *_unused) {
2138:   HANDLE_TH_ERRORS
2139:   auto prop = static_cast<NormBackward1*>(self->cdata.get())->keepdim;
2140:   if (prop) {
2141:     Py_RETURN_TRUE;
2142:   } else {
2143:     Py_RETURN_FALSE;
2144:   }
2145:   END_HANDLE_TH_ERRORS
2146: }
2147: 
2148: static PyObject* THPNormBackward1_p_getter(THPCppFunction *self, void *_unused) {
2149:   HANDLE_TH_ERRORS
2150:   auto opt_prop = static_cast<NormBackward1*>(self->cdata.get())->p;
2151:   if (!opt_prop.has_value()) {
2152:     Py_RETURN_NONE;
2153:   }
2154:   auto prop = opt_prop.value();
2155:   if (prop.isComplex()) {
2156:     auto cprop = prop.to<c10::complex<double>>();
2157:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
2158:   } else if (prop.isFloatingPoint()) {
2159:     return PyFloat_FromDouble(prop.to<double>());
2160:   } else if (prop.isIntegral(/*includeBool=*/false)) {
```

- EN: The main execution path in this span is carried by `THPNormBackward0_self_getter`, `THPVariable_Wrap`, `THPNormBackward0_self_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPNormBackward0_self_getter`, `THPVariable_Wrap`, `THPNormBackward0_self_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2161-2240

```cpp
2161:     return PyLong_FromLong(prop.to<int64_t>());
2162:   } else if (prop.isBoolean()) {
2163:     if (prop.to<bool>()) {
2164:       Py_RETURN_TRUE;
2165:     } else {
2166:       Py_RETURN_FALSE;
2167:     }
2168:   } else {
2169:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
2170:     return nullptr;
2171:   }
2172:   END_HANDLE_TH_ERRORS
2173: }
2174: 
2175: static PyObject* THPNormBackward1_self_getter(THPCppFunction *self, void *_unused) {
2176:   HANDLE_TH_ERRORS
2177:   const auto& prop = static_cast<NormBackward1*>(self->cdata.get())->self_;
2178:   return THPVariable_Wrap(prop.unpack(self->cdata));
2179:   END_HANDLE_TH_ERRORS
2180: }
2181: 
2182: static PyObject* THPNormBackward1_self_raw_getter(THPCppFunction *self, void *_unused) {
2183:   HANDLE_TH_ERRORS
2184:   const auto& prop = static_cast<NormBackward1*>(self->cdata.get())->self_;
2185:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2186:   return obj.release().ptr();
2187:   END_HANDLE_TH_ERRORS
2188: }
2189: 
2190: static PyObject* THPNormBackward1_result_getter(THPCppFunction *self, void *_unused) {
2191:   HANDLE_TH_ERRORS
2192:   const auto& prop = static_cast<NormBackward1*>(self->cdata.get())->result_;
2193:   return THPVariable_Wrap(prop.unpack(self->cdata));
2194:   END_HANDLE_TH_ERRORS
2195: }
2196: 
2197: static PyObject* THPNormBackward1_result_raw_getter(THPCppFunction *self, void *_unused) {
2198:   HANDLE_TH_ERRORS
2199:   const auto& prop = static_cast<NormBackward1*>(self->cdata.get())->result_;
2200:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2201:   return obj.release().ptr();
2202:   END_HANDLE_TH_ERRORS
2203: }
2204: 
2205: static struct PyGetSetDef NormBackward1_properties[] = {
2206:   THP_FUNCTION_DEFAULT_PROPERTIES,
2207:   {(char*)"_saved_dim", (getter)THPNormBackward1_dim_getter, nullptr, nullptr, nullptr},
2208:   {(char*)"_saved_keepdim", (getter)THPNormBackward1_keepdim_getter, nullptr, nullptr, nullptr},
2209:   {(char*)"_saved_p", (getter)THPNormBackward1_p_getter, nullptr, nullptr, nullptr},
2210:   {(char*)"_saved_self", (getter)THPNormBackward1_self_getter, nullptr, nullptr, nullptr},
2211:   {(char*)"_raw_saved_self", (getter)THPNormBackward1_self_raw_getter, nullptr, nullptr, nullptr},
2212:   {(char*)"_saved_result", (getter)THPNormBackward1_result_getter, nullptr, nullptr, nullptr},
2213:   {(char*)"_raw_saved_result", (getter)THPNormBackward1_result_raw_getter, nullptr, nullptr, nullptr},
2214:   {nullptr} /* sentinel */
2215: };
2216: 
2217: static PyObject* THPNormBackward2_p_getter(THPCppFunction *self, void *_unused) {
2218:   HANDLE_TH_ERRORS
2219:   auto opt_prop = static_cast<NormBackward2*>(self->cdata.get())->p;
2220:   if (!opt_prop.has_value()) {
2221:     Py_RETURN_NONE;
2222:   }
2223:   auto prop = opt_prop.value();
2224:   if (prop.isComplex()) {
2225:     auto cprop = prop.to<c10::complex<double>>();
2226:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
2227:   } else if (prop.isFloatingPoint()) {
2228:     return PyFloat_FromDouble(prop.to<double>());
2229:   } else if (prop.isIntegral(/*includeBool=*/false)) {
2230:     return PyLong_FromLong(prop.to<int64_t>());
2231:   } else if (prop.isBoolean()) {
2232:     if (prop.to<bool>()) {
2233:       Py_RETURN_TRUE;
2234:     } else {
2235:       Py_RETURN_FALSE;
2236:     }
2237:   } else {
2238:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
2239:     return nullptr;
2240:   }
```

- EN: The main execution path in this span is carried by `PyLong_FromLong`, `PyErr_SetString`, `THPNormBackward1_self_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyLong_FromLong`, `PyErr_SetString`, `THPNormBackward1_self_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2241-2320

```cpp
2241:   END_HANDLE_TH_ERRORS
2242: }
2243: 
2244: static PyObject* THPNormBackward2_self_getter(THPCppFunction *self, void *_unused) {
2245:   HANDLE_TH_ERRORS
2246:   const auto& prop = static_cast<NormBackward2*>(self->cdata.get())->self_;
2247:   return THPVariable_Wrap(prop.unpack(self->cdata));
2248:   END_HANDLE_TH_ERRORS
2249: }
2250: 
2251: static PyObject* THPNormBackward2_self_raw_getter(THPCppFunction *self, void *_unused) {
2252:   HANDLE_TH_ERRORS
2253:   const auto& prop = static_cast<NormBackward2*>(self->cdata.get())->self_;
2254:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2255:   return obj.release().ptr();
2256:   END_HANDLE_TH_ERRORS
2257: }
2258: 
2259: static PyObject* THPNormBackward2_result_getter(THPCppFunction *self, void *_unused) {
2260:   HANDLE_TH_ERRORS
2261:   const auto& prop = static_cast<NormBackward2*>(self->cdata.get())->result_;
2262:   return THPVariable_Wrap(prop.unpack(self->cdata));
2263:   END_HANDLE_TH_ERRORS
2264: }
2265: 
2266: static PyObject* THPNormBackward2_result_raw_getter(THPCppFunction *self, void *_unused) {
2267:   HANDLE_TH_ERRORS
2268:   const auto& prop = static_cast<NormBackward2*>(self->cdata.get())->result_;
2269:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2270:   return obj.release().ptr();
2271:   END_HANDLE_TH_ERRORS
2272: }
2273: 
2274: static struct PyGetSetDef NormBackward2_properties[] = {
2275:   THP_FUNCTION_DEFAULT_PROPERTIES,
2276:   {(char*)"_saved_p", (getter)THPNormBackward2_p_getter, nullptr, nullptr, nullptr},
2277:   {(char*)"_saved_self", (getter)THPNormBackward2_self_getter, nullptr, nullptr, nullptr},
2278:   {(char*)"_raw_saved_self", (getter)THPNormBackward2_self_raw_getter, nullptr, nullptr, nullptr},
2279:   {(char*)"_saved_result", (getter)THPNormBackward2_result_getter, nullptr, nullptr, nullptr},
2280:   {(char*)"_raw_saved_result", (getter)THPNormBackward2_result_raw_getter, nullptr, nullptr, nullptr},
2281:   {nullptr} /* sentinel */
2282: };
2283: 
2284: static PyObject* THPNormBackward3_dim_getter(THPCppFunction *self, void *_unused) {
2285:   HANDLE_TH_ERRORS
2286:   auto prop = static_cast<NormBackward3*>(self->cdata.get())->dim;
2287:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
2288:   for (auto i : c10::irange(prop.size())) {
2289:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
2290:   }
2291:   return tup;
2292:   END_HANDLE_TH_ERRORS
2293: }
2294: 
2295: static PyObject* THPNormBackward3_keepdim_getter(THPCppFunction *self, void *_unused) {
2296:   HANDLE_TH_ERRORS
2297:   auto prop = static_cast<NormBackward3*>(self->cdata.get())->keepdim;
2298:   if (prop) {
2299:     Py_RETURN_TRUE;
2300:   } else {
2301:     Py_RETURN_FALSE;
2302:   }
2303:   END_HANDLE_TH_ERRORS
2304: }
2305: 
2306: static PyObject* THPNormBackward3_p_getter(THPCppFunction *self, void *_unused) {
2307:   HANDLE_TH_ERRORS
2308:   auto opt_prop = static_cast<NormBackward3*>(self->cdata.get())->p;
2309:   if (!opt_prop.has_value()) {
2310:     Py_RETURN_NONE;
2311:   }
2312:   auto prop = opt_prop.value();
2313:   if (prop.isComplex()) {
2314:     auto cprop = prop.to<c10::complex<double>>();
2315:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
2316:   } else if (prop.isFloatingPoint()) {
2317:     return PyFloat_FromDouble(prop.to<double>());
2318:   } else if (prop.isIntegral(/*includeBool=*/false)) {
2319:     return PyLong_FromLong(prop.to<int64_t>());
2320:   } else if (prop.isBoolean()) {
```

- EN: The main execution path in this span is carried by `THPNormBackward2_self_getter`, `THPVariable_Wrap`, `THPNormBackward2_self_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPNormBackward2_self_getter`, `THPVariable_Wrap`, `THPNormBackward2_self_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2321-2400

```cpp
2321:     if (prop.to<bool>()) {
2322:       Py_RETURN_TRUE;
2323:     } else {
2324:       Py_RETURN_FALSE;
2325:     }
2326:   } else {
2327:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
2328:     return nullptr;
2329:   }
2330:   END_HANDLE_TH_ERRORS
2331: }
2332: 
2333: static PyObject* THPNormBackward3_self_getter(THPCppFunction *self, void *_unused) {
2334:   HANDLE_TH_ERRORS
2335:   const auto& prop = static_cast<NormBackward3*>(self->cdata.get())->self_;
2336:   return THPVariable_Wrap(prop.unpack(self->cdata));
2337:   END_HANDLE_TH_ERRORS
2338: }
2339: 
2340: static PyObject* THPNormBackward3_self_raw_getter(THPCppFunction *self, void *_unused) {
2341:   HANDLE_TH_ERRORS
2342:   const auto& prop = static_cast<NormBackward3*>(self->cdata.get())->self_;
2343:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2344:   return obj.release().ptr();
2345:   END_HANDLE_TH_ERRORS
2346: }
2347: 
2348: static PyObject* THPNormBackward3_result_getter(THPCppFunction *self, void *_unused) {
2349:   HANDLE_TH_ERRORS
2350:   const auto& prop = static_cast<NormBackward3*>(self->cdata.get())->result_;
2351:   return THPVariable_Wrap(prop.unpack(self->cdata));
2352:   END_HANDLE_TH_ERRORS
2353: }
2354: 
2355: static PyObject* THPNormBackward3_result_raw_getter(THPCppFunction *self, void *_unused) {
2356:   HANDLE_TH_ERRORS
2357:   const auto& prop = static_cast<NormBackward3*>(self->cdata.get())->result_;
2358:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2359:   return obj.release().ptr();
2360:   END_HANDLE_TH_ERRORS
2361: }
2362: 
2363: static struct PyGetSetDef NormBackward3_properties[] = {
2364:   THP_FUNCTION_DEFAULT_PROPERTIES,
2365:   {(char*)"_saved_dim", (getter)THPNormBackward3_dim_getter, nullptr, nullptr, nullptr},
2366:   {(char*)"_saved_keepdim", (getter)THPNormBackward3_keepdim_getter, nullptr, nullptr, nullptr},
2367:   {(char*)"_saved_p", (getter)THPNormBackward3_p_getter, nullptr, nullptr, nullptr},
2368:   {(char*)"_saved_self", (getter)THPNormBackward3_self_getter, nullptr, nullptr, nullptr},
2369:   {(char*)"_raw_saved_self", (getter)THPNormBackward3_self_raw_getter, nullptr, nullptr, nullptr},
2370:   {(char*)"_saved_result", (getter)THPNormBackward3_result_getter, nullptr, nullptr, nullptr},
2371:   {(char*)"_raw_saved_result", (getter)THPNormBackward3_result_raw_getter, nullptr, nullptr, nullptr},
2372:   {nullptr} /* sentinel */
2373: };
2374: 
2375: static PyObject* THPPdistBackward0_p_getter(THPCppFunction *self, void *_unused) {
2376:   HANDLE_TH_ERRORS
2377:   auto prop = static_cast<PdistBackward0*>(self->cdata.get())->p;
2378:   return PyFloat_FromDouble((double) prop);
2379:   END_HANDLE_TH_ERRORS
2380: }
2381: 
2382: static PyObject* THPPdistBackward0_self_getter(THPCppFunction *self, void *_unused) {
2383:   HANDLE_TH_ERRORS
2384:   const auto& prop = static_cast<PdistBackward0*>(self->cdata.get())->self_;
2385:   return THPVariable_Wrap(prop.unpack(self->cdata));
2386:   END_HANDLE_TH_ERRORS
2387: }
2388: 
2389: static PyObject* THPPdistBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
2390:   HANDLE_TH_ERRORS
2391:   const auto& prop = static_cast<PdistBackward0*>(self->cdata.get())->self_;
2392:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2393:   return obj.release().ptr();
2394:   END_HANDLE_TH_ERRORS
2395: }
2396: 
2397: static PyObject* THPPdistBackward0_result_getter(THPCppFunction *self, void *_unused) {
2398:   HANDLE_TH_ERRORS
2399:   const auto& prop = static_cast<PdistBackward0*>(self->cdata.get())->result_;
2400:   return THPVariable_Wrap(prop.unpack(self->cdata));
```

- EN: The main execution path in this span is carried by `PyErr_SetString`, `THPNormBackward3_self_getter`, `THPVariable_Wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyErr_SetString`, `THPNormBackward3_self_getter`, `THPVariable_Wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2401-2480

```cpp
2401:   END_HANDLE_TH_ERRORS
2402: }
2403: 
2404: static PyObject* THPPdistBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
2405:   HANDLE_TH_ERRORS
2406:   const auto& prop = static_cast<PdistBackward0*>(self->cdata.get())->result_;
2407:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2408:   return obj.release().ptr();
2409:   END_HANDLE_TH_ERRORS
2410: }
2411: 
2412: static struct PyGetSetDef PdistBackward0_properties[] = {
2413:   THP_FUNCTION_DEFAULT_PROPERTIES,
2414:   {(char*)"_saved_p", (getter)THPPdistBackward0_p_getter, nullptr, nullptr, nullptr},
2415:   {(char*)"_saved_self", (getter)THPPdistBackward0_self_getter, nullptr, nullptr, nullptr},
2416:   {(char*)"_raw_saved_self", (getter)THPPdistBackward0_self_raw_getter, nullptr, nullptr, nullptr},
2417:   {(char*)"_saved_result", (getter)THPPdistBackward0_result_getter, nullptr, nullptr, nullptr},
2418:   {(char*)"_raw_saved_result", (getter)THPPdistBackward0_result_raw_getter, nullptr, nullptr, nullptr},
2419:   {nullptr} /* sentinel */
2420: };
2421: 
2422: static PyObject* THPEuclideanDistBackward0_x1_getter(THPCppFunction *self, void *_unused) {
2423:   HANDLE_TH_ERRORS
2424:   const auto& prop = static_cast<EuclideanDistBackward0*>(self->cdata.get())->x1_;
2425:   return THPVariable_Wrap(prop.unpack(self->cdata));
2426:   END_HANDLE_TH_ERRORS
2427: }
2428: 
2429: static PyObject* THPEuclideanDistBackward0_x1_raw_getter(THPCppFunction *self, void *_unused) {
2430:   HANDLE_TH_ERRORS
2431:   const auto& prop = static_cast<EuclideanDistBackward0*>(self->cdata.get())->x1_;
2432:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2433:   return obj.release().ptr();
2434:   END_HANDLE_TH_ERRORS
2435: }
2436: 
2437: static PyObject* THPEuclideanDistBackward0_x2_getter(THPCppFunction *self, void *_unused) {
2438:   HANDLE_TH_ERRORS
2439:   const auto& prop = static_cast<EuclideanDistBackward0*>(self->cdata.get())->x2_;
2440:   return THPVariable_Wrap(prop.unpack(self->cdata));
2441:   END_HANDLE_TH_ERRORS
2442: }
2443: 
2444: static PyObject* THPEuclideanDistBackward0_x2_raw_getter(THPCppFunction *self, void *_unused) {
2445:   HANDLE_TH_ERRORS
2446:   const auto& prop = static_cast<EuclideanDistBackward0*>(self->cdata.get())->x2_;
2447:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2448:   return obj.release().ptr();
2449:   END_HANDLE_TH_ERRORS
2450: }
2451: 
2452: static PyObject* THPEuclideanDistBackward0_result_getter(THPCppFunction *self, void *_unused) {
2453:   HANDLE_TH_ERRORS
2454:   const auto& prop = static_cast<EuclideanDistBackward0*>(self->cdata.get())->result_;
2455:   return THPVariable_Wrap(prop.unpack(self->cdata));
2456:   END_HANDLE_TH_ERRORS
2457: }
2458: 
2459: static PyObject* THPEuclideanDistBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
2460:   HANDLE_TH_ERRORS
2461:   const auto& prop = static_cast<EuclideanDistBackward0*>(self->cdata.get())->result_;
2462:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2463:   return obj.release().ptr();
2464:   END_HANDLE_TH_ERRORS
2465: }
2466: 
2467: static struct PyGetSetDef EuclideanDistBackward0_properties[] = {
2468:   THP_FUNCTION_DEFAULT_PROPERTIES,
2469:   {(char*)"_saved_x1", (getter)THPEuclideanDistBackward0_x1_getter, nullptr, nullptr, nullptr},
2470:   {(char*)"_raw_saved_x1", (getter)THPEuclideanDistBackward0_x1_raw_getter, nullptr, nullptr, nullptr},
2471:   {(char*)"_saved_x2", (getter)THPEuclideanDistBackward0_x2_getter, nullptr, nullptr, nullptr},
2472:   {(char*)"_raw_saved_x2", (getter)THPEuclideanDistBackward0_x2_raw_getter, nullptr, nullptr, nullptr},
2473:   {(char*)"_saved_result", (getter)THPEuclideanDistBackward0_result_getter, nullptr, nullptr, nullptr},
2474:   {(char*)"_raw_saved_result", (getter)THPEuclideanDistBackward0_result_raw_getter, nullptr, nullptr, nullptr},
2475:   {nullptr} /* sentinel */
2476: };
2477: 
2478: 
2479: 
2480: static struct PyGetSetDef CdistBackwardBackward0_properties[] = {
```

- EN: The main execution path in this span is carried by `THPPdistBackward0_result_raw_getter`, `cast`, `THPEuclideanDistBackward0_x1_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPPdistBackward0_result_raw_getter`, `cast`, `THPEuclideanDistBackward0_x1_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2481-2560

```cpp
2481:   THP_FUNCTION_DEFAULT_PROPERTIES,
2482: 
2483:   {nullptr} /* sentinel */
2484: };
2485: 
2486: static PyObject* THPLinalgHouseholderProductBackward0_input_getter(THPCppFunction *self, void *_unused) {
2487:   HANDLE_TH_ERRORS
2488:   const auto& prop = static_cast<LinalgHouseholderProductBackward0*>(self->cdata.get())->input_;
2489:   return THPVariable_Wrap(prop.unpack(self->cdata));
2490:   END_HANDLE_TH_ERRORS
2491: }
2492: 
2493: static PyObject* THPLinalgHouseholderProductBackward0_input_raw_getter(THPCppFunction *self, void *_unused) {
2494:   HANDLE_TH_ERRORS
2495:   const auto& prop = static_cast<LinalgHouseholderProductBackward0*>(self->cdata.get())->input_;
2496:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2497:   return obj.release().ptr();
2498:   END_HANDLE_TH_ERRORS
2499: }
2500: 
2501: static PyObject* THPLinalgHouseholderProductBackward0_tau_getter(THPCppFunction *self, void *_unused) {
2502:   HANDLE_TH_ERRORS
2503:   const auto& prop = static_cast<LinalgHouseholderProductBackward0*>(self->cdata.get())->tau_;
2504:   return THPVariable_Wrap(prop.unpack(self->cdata));
2505:   END_HANDLE_TH_ERRORS
2506: }
2507: 
2508: static PyObject* THPLinalgHouseholderProductBackward0_tau_raw_getter(THPCppFunction *self, void *_unused) {
2509:   HANDLE_TH_ERRORS
2510:   const auto& prop = static_cast<LinalgHouseholderProductBackward0*>(self->cdata.get())->tau_;
2511:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2512:   return obj.release().ptr();
2513:   END_HANDLE_TH_ERRORS
2514: }
2515: 
2516: static PyObject* THPLinalgHouseholderProductBackward0_result_getter(THPCppFunction *self, void *_unused) {
2517:   HANDLE_TH_ERRORS
2518:   const auto& prop = static_cast<LinalgHouseholderProductBackward0*>(self->cdata.get())->result_;
2519:   return THPVariable_Wrap(prop.unpack(self->cdata));
2520:   END_HANDLE_TH_ERRORS
2521: }
2522: 
2523: static PyObject* THPLinalgHouseholderProductBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
2524:   HANDLE_TH_ERRORS
2525:   const auto& prop = static_cast<LinalgHouseholderProductBackward0*>(self->cdata.get())->result_;
2526:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2527:   return obj.release().ptr();
2528:   END_HANDLE_TH_ERRORS
2529: }
2530: 
2531: static struct PyGetSetDef LinalgHouseholderProductBackward0_properties[] = {
2532:   THP_FUNCTION_DEFAULT_PROPERTIES,
2533:   {(char*)"_saved_input", (getter)THPLinalgHouseholderProductBackward0_input_getter, nullptr, nullptr, nullptr},
2534:   {(char*)"_raw_saved_input", (getter)THPLinalgHouseholderProductBackward0_input_raw_getter, nullptr, nullptr, nullptr},
2535:   {(char*)"_saved_tau", (getter)THPLinalgHouseholderProductBackward0_tau_getter, nullptr, nullptr, nullptr},
2536:   {(char*)"_raw_saved_tau", (getter)THPLinalgHouseholderProductBackward0_tau_raw_getter, nullptr, nullptr, nullptr},
2537:   {(char*)"_saved_result", (getter)THPLinalgHouseholderProductBackward0_result_getter, nullptr, nullptr, nullptr},
2538:   {(char*)"_raw_saved_result", (getter)THPLinalgHouseholderProductBackward0_result_raw_getter, nullptr, nullptr, nullptr},
2539:   {nullptr} /* sentinel */
2540: };
2541: 
2542: static PyObject* THPPutBackward0_accumulate_getter(THPCppFunction *self, void *_unused) {
2543:   HANDLE_TH_ERRORS
2544:   auto prop = static_cast<PutBackward0*>(self->cdata.get())->accumulate;
2545:   if (prop) {
2546:     Py_RETURN_TRUE;
2547:   } else {
2548:     Py_RETURN_FALSE;
2549:   }
2550:   END_HANDLE_TH_ERRORS
2551: }
2552: 
2553: static PyObject* THPPutBackward0_index_getter(THPCppFunction *self, void *_unused) {
2554:   HANDLE_TH_ERRORS
2555:   const auto& prop = static_cast<PutBackward0*>(self->cdata.get())->index_;
2556:   return THPVariable_Wrap(prop.unpack(self->cdata));
2557:   END_HANDLE_TH_ERRORS
2558: }
2559: 
2560: static PyObject* THPPutBackward0_index_raw_getter(THPCppFunction *self, void *_unused) {
```

- EN: The main execution path in this span is carried by `THPLinalgHouseholderProductBackward0_input_getter`, `THPVariable_Wrap`, `THPLinalgHouseholderProductBackward0_input_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPLinalgHouseholderProductBackward0_input_getter`, `THPVariable_Wrap`, `THPLinalgHouseholderProductBackward0_input_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2561-2640

```cpp
2561:   HANDLE_TH_ERRORS
2562:   const auto& prop = static_cast<PutBackward0*>(self->cdata.get())->index_;
2563:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2564:   return obj.release().ptr();
2565:   END_HANDLE_TH_ERRORS
2566: }
2567: 
2568: static PyObject* THPPutBackward0_source_getter(THPCppFunction *self, void *_unused) {
2569:   HANDLE_TH_ERRORS
2570:   const auto& prop = static_cast<PutBackward0*>(self->cdata.get())->source_;
2571:   return THPVariable_Wrap(prop.unpack(self->cdata));
2572:   END_HANDLE_TH_ERRORS
2573: }
2574: 
2575: static PyObject* THPPutBackward0_source_raw_getter(THPCppFunction *self, void *_unused) {
2576:   HANDLE_TH_ERRORS
2577:   const auto& prop = static_cast<PutBackward0*>(self->cdata.get())->source_;
2578:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2579:   return obj.release().ptr();
2580:   END_HANDLE_TH_ERRORS
2581: }
2582: 
2583: static struct PyGetSetDef PutBackward0_properties[] = {
2584:   THP_FUNCTION_DEFAULT_PROPERTIES,
2585:   {(char*)"_saved_accumulate", (getter)THPPutBackward0_accumulate_getter, nullptr, nullptr, nullptr},
2586:   {(char*)"_saved_index", (getter)THPPutBackward0_index_getter, nullptr, nullptr, nullptr},
2587:   {(char*)"_raw_saved_index", (getter)THPPutBackward0_index_raw_getter, nullptr, nullptr, nullptr},
2588:   {(char*)"_saved_source", (getter)THPPutBackward0_source_getter, nullptr, nullptr, nullptr},
2589:   {(char*)"_raw_saved_source", (getter)THPPutBackward0_source_raw_getter, nullptr, nullptr, nullptr},
2590:   {nullptr} /* sentinel */
2591: };
2592: 
2593: 
2594: 
2595: static struct PyGetSetDef Rad2DegBackward0_properties[] = {
2596:   THP_FUNCTION_DEFAULT_PROPERTIES,
2597: 
2598:   {nullptr} /* sentinel */
2599: };
2600: 
2601: 
2602: 
2603: static struct PyGetSetDef RandomBackward0_properties[] = {
2604:   THP_FUNCTION_DEFAULT_PROPERTIES,
2605: 
2606:   {nullptr} /* sentinel */
2607: };
2608: 
2609: 
2610: 
2611: static struct PyGetSetDef RandomBackward1_properties[] = {
2612:   THP_FUNCTION_DEFAULT_PROPERTIES,
2613: 
2614:   {nullptr} /* sentinel */
2615: };
2616: 
2617: 
2618: 
2619: static struct PyGetSetDef RandomBackward2_properties[] = {
2620:   THP_FUNCTION_DEFAULT_PROPERTIES,
2621: 
2622:   {nullptr} /* sentinel */
2623: };
2624: 
2625: static PyObject* THPRepeatBackward0_repeats_getter(THPCppFunction *self, void *_unused) {
2626:   HANDLE_TH_ERRORS
2627:   auto prop = static_cast<RepeatBackward0*>(self->cdata.get())->repeats;
2628:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
2629:   for (auto i : c10::irange(prop.size())) {
2630:       auto si = prop[i];
2631:       if (auto m = si.maybe_as_int()) {
2632:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
2633:       } else {
2634:         auto py_symint = py::cast(si).release().ptr();
2635:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
2636:       }
2637:   }
2638:   return tup;
2639:   END_HANDLE_TH_ERRORS
2640: }
```

- EN: The main execution path in this span is carried by `cast`, `THPPutBackward0_source_getter`, `THPVariable_Wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `cast`, `THPPutBackward0_source_getter`, `THPVariable_Wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2641-2720

```cpp
2641: 
2642: static PyObject* THPRepeatBackward0_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
2643:   HANDLE_TH_ERRORS
2644:   auto prop = static_cast<RepeatBackward0*>(self->cdata.get())->self_sym_sizes;
2645:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
2646:   for (auto i : c10::irange(prop.size())) {
2647:       auto si = prop[i];
2648:       if (auto m = si.maybe_as_int()) {
2649:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
2650:       } else {
2651:         auto py_symint = py::cast(si).release().ptr();
2652:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
2653:       }
2654:   }
2655:   return tup;
2656:   END_HANDLE_TH_ERRORS
2657: }
2658: 
2659: static struct PyGetSetDef RepeatBackward0_properties[] = {
2660:   THP_FUNCTION_DEFAULT_PROPERTIES,
2661:   {(char*)"_saved_repeats", (getter)THPRepeatBackward0_repeats_getter, nullptr, nullptr, nullptr},
2662:   {(char*)"_saved_self_sym_sizes", (getter)THPRepeatBackward0_self_sym_sizes_getter, nullptr, nullptr, nullptr},
2663:   {nullptr} /* sentinel */
2664: };
2665: 
2666: static PyObject* THPRsqrtBackward0_result_getter(THPCppFunction *self, void *_unused) {
2667:   HANDLE_TH_ERRORS
2668:   const auto& prop = static_cast<RsqrtBackward0*>(self->cdata.get())->result_;
2669:   return THPVariable_Wrap(prop.unpack(self->cdata));
2670:   END_HANDLE_TH_ERRORS
2671: }
2672: 
2673: static PyObject* THPRsqrtBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
2674:   HANDLE_TH_ERRORS
2675:   const auto& prop = static_cast<RsqrtBackward0*>(self->cdata.get())->result_;
2676:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2677:   return obj.release().ptr();
2678:   END_HANDLE_TH_ERRORS
2679: }
2680: 
2681: static struct PyGetSetDef RsqrtBackward0_properties[] = {
2682:   THP_FUNCTION_DEFAULT_PROPERTIES,
2683:   {(char*)"_saved_result", (getter)THPRsqrtBackward0_result_getter, nullptr, nullptr, nullptr},
2684:   {(char*)"_raw_saved_result", (getter)THPRsqrtBackward0_result_raw_getter, nullptr, nullptr, nullptr},
2685:   {nullptr} /* sentinel */
2686: };
2687: 
2688: static PyObject* THPScatterBackward0_dim_getter(THPCppFunction *self, void *_unused) {
2689:   HANDLE_TH_ERRORS
2690:   auto prop = static_cast<ScatterBackward0*>(self->cdata.get())->dim;
2691:   return PyLong_FromUnsignedLong((int64_t) prop);
2692:   END_HANDLE_TH_ERRORS
2693: }
2694: 
2695: static PyObject* THPScatterBackward0_index_getter(THPCppFunction *self, void *_unused) {
2696:   HANDLE_TH_ERRORS
2697:   const auto& prop = static_cast<ScatterBackward0*>(self->cdata.get())->index_;
2698:   return THPVariable_Wrap(prop.unpack(self->cdata));
2699:   END_HANDLE_TH_ERRORS
2700: }
2701: 
2702: static PyObject* THPScatterBackward0_index_raw_getter(THPCppFunction *self, void *_unused) {
2703:   HANDLE_TH_ERRORS
2704:   const auto& prop = static_cast<ScatterBackward0*>(self->cdata.get())->index_;
2705:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2706:   return obj.release().ptr();
2707:   END_HANDLE_TH_ERRORS
2708: }
2709: 
2710: static struct PyGetSetDef ScatterBackward0_properties[] = {
2711:   THP_FUNCTION_DEFAULT_PROPERTIES,
2712:   {(char*)"_saved_dim", (getter)THPScatterBackward0_dim_getter, nullptr, nullptr, nullptr},
2713:   {(char*)"_saved_index", (getter)THPScatterBackward0_index_getter, nullptr, nullptr, nullptr},
2714:   {(char*)"_raw_saved_index", (getter)THPScatterBackward0_index_raw_getter, nullptr, nullptr, nullptr},
2715:   {nullptr} /* sentinel */
2716: };
2717: 
2718: static PyObject* THPScatterBackward1_dim_getter(THPCppFunction *self, void *_unused) {
2719:   HANDLE_TH_ERRORS
2720:   auto prop = static_cast<ScatterBackward1*>(self->cdata.get())->dim;
```

- EN: The main execution path in this span is carried by `THPRepeatBackward0_self_sym_sizes_getter`, `PyTuple_New`, `PyTuple_SetItem`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPRepeatBackward0_self_sym_sizes_getter`, `PyTuple_New`, `PyTuple_SetItem` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2721-2800

```cpp
2721:   return PyLong_FromUnsignedLong((int64_t) prop);
2722:   END_HANDLE_TH_ERRORS
2723: }
2724: 
2725: static PyObject* THPScatterBackward1_index_getter(THPCppFunction *self, void *_unused) {
2726:   HANDLE_TH_ERRORS
2727:   const auto& prop = static_cast<ScatterBackward1*>(self->cdata.get())->index_;
2728:   return THPVariable_Wrap(prop.unpack(self->cdata));
2729:   END_HANDLE_TH_ERRORS
2730: }
2731: 
2732: static PyObject* THPScatterBackward1_index_raw_getter(THPCppFunction *self, void *_unused) {
2733:   HANDLE_TH_ERRORS
2734:   const auto& prop = static_cast<ScatterBackward1*>(self->cdata.get())->index_;
2735:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2736:   return obj.release().ptr();
2737:   END_HANDLE_TH_ERRORS
2738: }
2739: 
2740: static struct PyGetSetDef ScatterBackward1_properties[] = {
2741:   THP_FUNCTION_DEFAULT_PROPERTIES,
2742:   {(char*)"_saved_dim", (getter)THPScatterBackward1_dim_getter, nullptr, nullptr, nullptr},
2743:   {(char*)"_saved_index", (getter)THPScatterBackward1_index_getter, nullptr, nullptr, nullptr},
2744:   {(char*)"_raw_saved_index", (getter)THPScatterBackward1_index_raw_getter, nullptr, nullptr, nullptr},
2745:   {nullptr} /* sentinel */
2746: };
2747: 
2748: static PyObject* THPSortBackward0_dim_getter(THPCppFunction *self, void *_unused) {
2749:   HANDLE_TH_ERRORS
2750:   auto prop = static_cast<SortBackward0*>(self->cdata.get())->dim;
2751:   return PyLong_FromUnsignedLong((int64_t) prop);
2752:   END_HANDLE_TH_ERRORS
2753: }
2754: 
2755: static PyObject* THPSortBackward0_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
2756:   HANDLE_TH_ERRORS
2757:   auto prop = static_cast<SortBackward0*>(self->cdata.get())->self_sym_sizes;
2758:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
2759:   for (auto i : c10::irange(prop.size())) {
2760:       auto si = prop[i];
2761:       if (auto m = si.maybe_as_int()) {
2762:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
2763:       } else {
2764:         auto py_symint = py::cast(si).release().ptr();
2765:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
2766:       }
2767:   }
2768:   return tup;
2769:   END_HANDLE_TH_ERRORS
2770: }
2771: 
2772: static PyObject* THPSortBackward0_indices_getter(THPCppFunction *self, void *_unused) {
2773:   HANDLE_TH_ERRORS
2774:   const auto& prop = static_cast<SortBackward0*>(self->cdata.get())->indices_;
2775:   return THPVariable_Wrap(prop.unpack(self->cdata));
2776:   END_HANDLE_TH_ERRORS
2777: }
2778: 
2779: static PyObject* THPSortBackward0_indices_raw_getter(THPCppFunction *self, void *_unused) {
2780:   HANDLE_TH_ERRORS
2781:   const auto& prop = static_cast<SortBackward0*>(self->cdata.get())->indices_;
2782:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2783:   return obj.release().ptr();
2784:   END_HANDLE_TH_ERRORS
2785: }
2786: 
2787: static struct PyGetSetDef SortBackward0_properties[] = {
2788:   THP_FUNCTION_DEFAULT_PROPERTIES,
2789:   {(char*)"_saved_dim", (getter)THPSortBackward0_dim_getter, nullptr, nullptr, nullptr},
2790:   {(char*)"_saved_self_sym_sizes", (getter)THPSortBackward0_self_sym_sizes_getter, nullptr, nullptr, nullptr},
2791:   {(char*)"_saved_indices", (getter)THPSortBackward0_indices_getter, nullptr, nullptr, nullptr},
2792:   {(char*)"_raw_saved_indices", (getter)THPSortBackward0_indices_raw_getter, nullptr, nullptr, nullptr},
2793:   {nullptr} /* sentinel */
2794: };
2795: 
2796: static PyObject* THPSortBackward1_dim_getter(THPCppFunction *self, void *_unused) {
2797:   HANDLE_TH_ERRORS
2798:   auto prop = static_cast<SortBackward1*>(self->cdata.get())->dim;
2799:   return PyLong_FromUnsignedLong((int64_t) prop);
2800:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `PyLong_FromUnsignedLong`, `THPScatterBackward1_index_getter`, `THPVariable_Wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyLong_FromUnsignedLong`, `THPScatterBackward1_index_getter`, `THPVariable_Wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2801-2880

```cpp
2801: }
2802: 
2803: static PyObject* THPSortBackward1_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
2804:   HANDLE_TH_ERRORS
2805:   auto prop = static_cast<SortBackward1*>(self->cdata.get())->self_sym_sizes;
2806:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
2807:   for (auto i : c10::irange(prop.size())) {
2808:       auto si = prop[i];
2809:       if (auto m = si.maybe_as_int()) {
2810:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
2811:       } else {
2812:         auto py_symint = py::cast(si).release().ptr();
2813:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
2814:       }
2815:   }
2816:   return tup;
2817:   END_HANDLE_TH_ERRORS
2818: }
2819: 
2820: static PyObject* THPSortBackward1_indices_getter(THPCppFunction *self, void *_unused) {
2821:   HANDLE_TH_ERRORS
2822:   const auto& prop = static_cast<SortBackward1*>(self->cdata.get())->indices_;
2823:   return THPVariable_Wrap(prop.unpack(self->cdata));
2824:   END_HANDLE_TH_ERRORS
2825: }
2826: 
2827: static PyObject* THPSortBackward1_indices_raw_getter(THPCppFunction *self, void *_unused) {
2828:   HANDLE_TH_ERRORS
2829:   const auto& prop = static_cast<SortBackward1*>(self->cdata.get())->indices_;
2830:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2831:   return obj.release().ptr();
2832:   END_HANDLE_TH_ERRORS
2833: }
2834: 
2835: static struct PyGetSetDef SortBackward1_properties[] = {
2836:   THP_FUNCTION_DEFAULT_PROPERTIES,
2837:   {(char*)"_saved_dim", (getter)THPSortBackward1_dim_getter, nullptr, nullptr, nullptr},
2838:   {(char*)"_saved_self_sym_sizes", (getter)THPSortBackward1_self_sym_sizes_getter, nullptr, nullptr, nullptr},
2839:   {(char*)"_saved_indices", (getter)THPSortBackward1_indices_getter, nullptr, nullptr, nullptr},
2840:   {(char*)"_raw_saved_indices", (getter)THPSortBackward1_indices_raw_getter, nullptr, nullptr, nullptr},
2841:   {nullptr} /* sentinel */
2842: };
2843: 
2844: static PyObject* THPSqueezeBackward3_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
2845:   HANDLE_TH_ERRORS
2846:   auto prop = static_cast<SqueezeBackward3*>(self->cdata.get())->self_sym_sizes;
2847:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
2848:   for (auto i : c10::irange(prop.size())) {
2849:       auto si = prop[i];
2850:       if (auto m = si.maybe_as_int()) {
2851:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
2852:       } else {
2853:         auto py_symint = py::cast(si).release().ptr();
2854:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
2855:       }
2856:   }
2857:   return tup;
2858:   END_HANDLE_TH_ERRORS
2859: }
2860: 
2861: static struct PyGetSetDef SqueezeBackward3_properties[] = {
2862:   THP_FUNCTION_DEFAULT_PROPERTIES,
2863:   {(char*)"_saved_self_sym_sizes", (getter)THPSqueezeBackward3_self_sym_sizes_getter, nullptr, nullptr, nullptr},
2864:   {nullptr} /* sentinel */
2865: };
2866: 
2867: static PyObject* THPSqueezeBackward4_dim_getter(THPCppFunction *self, void *_unused) {
2868:   HANDLE_TH_ERRORS
2869:   auto prop = static_cast<SqueezeBackward4*>(self->cdata.get())->dim;
2870:   return PyLong_FromUnsignedLong((int64_t) prop);
2871:   END_HANDLE_TH_ERRORS
2872: }
2873: 
2874: static PyObject* THPSqueezeBackward4_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
2875:   HANDLE_TH_ERRORS
2876:   auto prop = static_cast<SqueezeBackward4*>(self->cdata.get())->self_sym_sizes;
2877:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
2878:   for (auto i : c10::irange(prop.size())) {
2879:       auto si = prop[i];
2880:       if (auto m = si.maybe_as_int()) {
```

- EN: The main execution path in this span is carried by `THPSortBackward1_self_sym_sizes_getter`, `PyTuple_New`, `PyTuple_SetItem`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPSortBackward1_self_sym_sizes_getter`, `PyTuple_New`, `PyTuple_SetItem` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2881-2960

```cpp
2881:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
2882:       } else {
2883:         auto py_symint = py::cast(si).release().ptr();
2884:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
2885:       }
2886:   }
2887:   return tup;
2888:   END_HANDLE_TH_ERRORS
2889: }
2890: 
2891: static struct PyGetSetDef SqueezeBackward4_properties[] = {
2892:   THP_FUNCTION_DEFAULT_PROPERTIES,
2893:   {(char*)"_saved_dim", (getter)THPSqueezeBackward4_dim_getter, nullptr, nullptr, nullptr},
2894:   {(char*)"_saved_self_sym_sizes", (getter)THPSqueezeBackward4_self_sym_sizes_getter, nullptr, nullptr, nullptr},
2895:   {nullptr} /* sentinel */
2896: };
2897: 
2898: static PyObject* THPSqueezeBackward5_dim_getter(THPCppFunction *self, void *_unused) {
2899:   HANDLE_TH_ERRORS
2900:   auto prop = static_cast<SqueezeBackward5*>(self->cdata.get())->dim;
2901:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
2902:   for (auto i : c10::irange(prop.size())) {
2903:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
2904:   }
2905:   return tup;
2906:   END_HANDLE_TH_ERRORS
2907: }
2908: 
2909: static PyObject* THPSqueezeBackward5_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
2910:   HANDLE_TH_ERRORS
2911:   auto prop = static_cast<SqueezeBackward5*>(self->cdata.get())->self_sym_sizes;
2912:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
2913:   for (auto i : c10::irange(prop.size())) {
2914:       auto si = prop[i];
2915:       if (auto m = si.maybe_as_int()) {
2916:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
2917:       } else {
2918:         auto py_symint = py::cast(si).release().ptr();
2919:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
2920:       }
2921:   }
2922:   return tup;
2923:   END_HANDLE_TH_ERRORS
2924: }
2925: 
2926: static struct PyGetSetDef SqueezeBackward5_properties[] = {
2927:   THP_FUNCTION_DEFAULT_PROPERTIES,
2928:   {(char*)"_saved_dim", (getter)THPSqueezeBackward5_dim_getter, nullptr, nullptr, nullptr},
2929:   {(char*)"_saved_self_sym_sizes", (getter)THPSqueezeBackward5_self_sym_sizes_getter, nullptr, nullptr, nullptr},
2930:   {nullptr} /* sentinel */
2931: };
2932: 
2933: static PyObject* THPSumBackward0_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
2934:   HANDLE_TH_ERRORS
2935:   auto prop = static_cast<SumBackward0*>(self->cdata.get())->self_sym_sizes;
2936:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
2937:   for (auto i : c10::irange(prop.size())) {
2938:       auto si = prop[i];
2939:       if (auto m = si.maybe_as_int()) {
2940:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
2941:       } else {
2942:         auto py_symint = py::cast(si).release().ptr();
2943:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
2944:       }
2945:   }
2946:   return tup;
2947:   END_HANDLE_TH_ERRORS
2948: }
2949: 
2950: static struct PyGetSetDef SumBackward0_properties[] = {
2951:   THP_FUNCTION_DEFAULT_PROPERTIES,
2952:   {(char*)"_saved_self_sym_sizes", (getter)THPSumBackward0_self_sym_sizes_getter, nullptr, nullptr, nullptr},
2953:   {nullptr} /* sentinel */
2954: };
2955: 
2956: static PyObject* THPSumBackwardAutogradNestedTensor0_self_getter(THPCppFunction *self, void *_unused) {
2957:   HANDLE_TH_ERRORS
2958:   const auto& prop = static_cast<SumBackwardAutogradNestedTensor0*>(self->cdata.get())->self_;
2959:   return THPVariable_Wrap(prop.unpack(self->cdata));
2960:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `PyTuple_SetItem`, `cast`, `THPSqueezeBackward5_dim_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `PyTuple_SetItem`, `cast`, `THPSqueezeBackward5_dim_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 2961-3040

```cpp
2961: }
2962: 
2963: static PyObject* THPSumBackwardAutogradNestedTensor0_self_raw_getter(THPCppFunction *self, void *_unused) {
2964:   HANDLE_TH_ERRORS
2965:   const auto& prop = static_cast<SumBackwardAutogradNestedTensor0*>(self->cdata.get())->self_;
2966:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2967:   return obj.release().ptr();
2968:   END_HANDLE_TH_ERRORS
2969: }
2970: 
2971: static struct PyGetSetDef SumBackwardAutogradNestedTensor0_properties[] = {
2972:   THP_FUNCTION_DEFAULT_PROPERTIES,
2973:   {(char*)"_saved_self", (getter)THPSumBackwardAutogradNestedTensor0_self_getter, nullptr, nullptr, nullptr},
2974:   {(char*)"_raw_saved_self", (getter)THPSumBackwardAutogradNestedTensor0_self_raw_getter, nullptr, nullptr, nullptr},
2975:   {nullptr} /* sentinel */
2976: };
2977: 
2978: static PyObject* THPSumBackward1_dim_getter(THPCppFunction *self, void *_unused) {
2979:   HANDLE_TH_ERRORS
2980:   auto opt_prop = static_cast<SumBackward1*>(self->cdata.get())->dim;
2981:   if (!opt_prop.list.has_value()) {
2982:     Py_RETURN_NONE;
2983:   }
2984:   auto prop = opt_prop.list.value();
2985:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
2986:   for (auto i : c10::irange(prop.size())) {
2987:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
2988:   }
2989:   return tup;
2990:   END_HANDLE_TH_ERRORS
2991: }
2992: 
2993: static PyObject* THPSumBackward1_keepdim_getter(THPCppFunction *self, void *_unused) {
2994:   HANDLE_TH_ERRORS
2995:   auto prop = static_cast<SumBackward1*>(self->cdata.get())->keepdim;
2996:   if (prop) {
2997:     Py_RETURN_TRUE;
2998:   } else {
2999:     Py_RETURN_FALSE;
3000:   }
3001:   END_HANDLE_TH_ERRORS
3002: }
3003: 
3004: static PyObject* THPSumBackward1_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
3005:   HANDLE_TH_ERRORS
3006:   auto prop = static_cast<SumBackward1*>(self->cdata.get())->self_sym_sizes;
3007:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3008:   for (auto i : c10::irange(prop.size())) {
3009:       auto si = prop[i];
3010:       if (auto m = si.maybe_as_int()) {
3011:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3012:       } else {
3013:         auto py_symint = py::cast(si).release().ptr();
3014:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3015:       }
3016:   }
3017:   return tup;
3018:   END_HANDLE_TH_ERRORS
3019: }
3020: 
3021: static struct PyGetSetDef SumBackward1_properties[] = {
3022:   THP_FUNCTION_DEFAULT_PROPERTIES,
3023:   {(char*)"_saved_dim", (getter)THPSumBackward1_dim_getter, nullptr, nullptr, nullptr},
3024:   {(char*)"_saved_keepdim", (getter)THPSumBackward1_keepdim_getter, nullptr, nullptr, nullptr},
3025:   {(char*)"_saved_self_sym_sizes", (getter)THPSumBackward1_self_sym_sizes_getter, nullptr, nullptr, nullptr},
3026:   {nullptr} /* sentinel */
3027: };
3028: 
3029: static PyObject* THPSumBackwardAutogradNestedTensor1_dim_getter(THPCppFunction *self, void *_unused) {
3030:   HANDLE_TH_ERRORS
3031:   auto opt_prop = static_cast<SumBackwardAutogradNestedTensor1*>(self->cdata.get())->dim;
3032:   if (!opt_prop.list.has_value()) {
3033:     Py_RETURN_NONE;
3034:   }
3035:   auto prop = opt_prop.list.value();
3036:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3037:   for (auto i : c10::irange(prop.size())) {
3038:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
3039:   }
3040:   return tup;
```

- EN: The main execution path in this span is carried by `THPSumBackwardAutogradNestedTensor0_self_raw_getter`, `cast`, `THPSumBackward1_dim_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPSumBackwardAutogradNestedTensor0_self_raw_getter`, `cast`, `THPSumBackward1_dim_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 3041-3120

```cpp
3041:   END_HANDLE_TH_ERRORS
3042: }
3043: 
3044: static PyObject* THPSumBackwardAutogradNestedTensor1_keepdim_getter(THPCppFunction *self, void *_unused) {
3045:   HANDLE_TH_ERRORS
3046:   auto prop = static_cast<SumBackwardAutogradNestedTensor1*>(self->cdata.get())->keepdim;
3047:   if (prop) {
3048:     Py_RETURN_TRUE;
3049:   } else {
3050:     Py_RETURN_FALSE;
3051:   }
3052:   END_HANDLE_TH_ERRORS
3053: }
3054: 
3055: static PyObject* THPSumBackwardAutogradNestedTensor1_self_getter(THPCppFunction *self, void *_unused) {
3056:   HANDLE_TH_ERRORS
3057:   const auto& prop = static_cast<SumBackwardAutogradNestedTensor1*>(self->cdata.get())->self_;
3058:   return THPVariable_Wrap(prop.unpack(self->cdata));
3059:   END_HANDLE_TH_ERRORS
3060: }
3061: 
3062: static PyObject* THPSumBackwardAutogradNestedTensor1_self_raw_getter(THPCppFunction *self, void *_unused) {
3063:   HANDLE_TH_ERRORS
3064:   const auto& prop = static_cast<SumBackwardAutogradNestedTensor1*>(self->cdata.get())->self_;
3065:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3066:   return obj.release().ptr();
3067:   END_HANDLE_TH_ERRORS
3068: }
3069: 
3070: static struct PyGetSetDef SumBackwardAutogradNestedTensor1_properties[] = {
3071:   THP_FUNCTION_DEFAULT_PROPERTIES,
3072:   {(char*)"_saved_dim", (getter)THPSumBackwardAutogradNestedTensor1_dim_getter, nullptr, nullptr, nullptr},
3073:   {(char*)"_saved_keepdim", (getter)THPSumBackwardAutogradNestedTensor1_keepdim_getter, nullptr, nullptr, nullptr},
3074:   {(char*)"_saved_self", (getter)THPSumBackwardAutogradNestedTensor1_self_getter, nullptr, nullptr, nullptr},
3075:   {(char*)"_raw_saved_self", (getter)THPSumBackwardAutogradNestedTensor1_self_raw_getter, nullptr, nullptr, nullptr},
3076:   {nullptr} /* sentinel */
3077: };
3078: 
3079: static PyObject* THPNansumBackward0_dim_getter(THPCppFunction *self, void *_unused) {
3080:   HANDLE_TH_ERRORS
3081:   auto opt_prop = static_cast<NansumBackward0*>(self->cdata.get())->dim;
3082:   if (!opt_prop.list.has_value()) {
3083:     Py_RETURN_NONE;
3084:   }
3085:   auto prop = opt_prop.list.value();
3086:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3087:   for (auto i : c10::irange(prop.size())) {
3088:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
3089:   }
3090:   return tup;
3091:   END_HANDLE_TH_ERRORS
3092: }
3093: 
3094: static PyObject* THPNansumBackward0_keepdim_getter(THPCppFunction *self, void *_unused) {
3095:   HANDLE_TH_ERRORS
3096:   auto prop = static_cast<NansumBackward0*>(self->cdata.get())->keepdim;
3097:   if (prop) {
3098:     Py_RETURN_TRUE;
3099:   } else {
3100:     Py_RETURN_FALSE;
3101:   }
3102:   END_HANDLE_TH_ERRORS
3103: }
3104: 
3105: static PyObject* THPNansumBackward0_self_getter(THPCppFunction *self, void *_unused) {
3106:   HANDLE_TH_ERRORS
3107:   const auto& prop = static_cast<NansumBackward0*>(self->cdata.get())->self_;
3108:   return THPVariable_Wrap(prop.unpack(self->cdata));
3109:   END_HANDLE_TH_ERRORS
3110: }
3111: 
3112: static PyObject* THPNansumBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
3113:   HANDLE_TH_ERRORS
3114:   const auto& prop = static_cast<NansumBackward0*>(self->cdata.get())->self_;
3115:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3116:   return obj.release().ptr();
3117:   END_HANDLE_TH_ERRORS
3118: }
3119: 
3120: static struct PyGetSetDef NansumBackward0_properties[] = {
```

- EN: The main execution path in this span is carried by `THPSumBackwardAutogradNestedTensor1_keepdim_getter`, `THPSumBackwardAutogradNestedTensor1_self_getter`, `THPVariable_Wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPSumBackwardAutogradNestedTensor1_keepdim_getter`, `THPSumBackwardAutogradNestedTensor1_self_getter`, `THPVariable_Wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 3121-3200

```cpp
3121:   THP_FUNCTION_DEFAULT_PROPERTIES,
3122:   {(char*)"_saved_dim", (getter)THPNansumBackward0_dim_getter, nullptr, nullptr, nullptr},
3123:   {(char*)"_saved_keepdim", (getter)THPNansumBackward0_keepdim_getter, nullptr, nullptr, nullptr},
3124:   {(char*)"_saved_self", (getter)THPNansumBackward0_self_getter, nullptr, nullptr, nullptr},
3125:   {(char*)"_raw_saved_self", (getter)THPNansumBackward0_self_raw_getter, nullptr, nullptr, nullptr},
3126:   {nullptr} /* sentinel */
3127: };
3128: 
3129: static PyObject* THPLinalgEigBackward0_eigenvalues_getter(THPCppFunction *self, void *_unused) {
3130:   HANDLE_TH_ERRORS
3131:   const auto& prop = static_cast<LinalgEigBackward0*>(self->cdata.get())->eigenvalues_;
3132:   return THPVariable_Wrap(prop.unpack(self->cdata));
3133:   END_HANDLE_TH_ERRORS
3134: }
3135: 
3136: static PyObject* THPLinalgEigBackward0_eigenvalues_raw_getter(THPCppFunction *self, void *_unused) {
3137:   HANDLE_TH_ERRORS
3138:   const auto& prop = static_cast<LinalgEigBackward0*>(self->cdata.get())->eigenvalues_;
3139:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3140:   return obj.release().ptr();
3141:   END_HANDLE_TH_ERRORS
3142: }
3143: 
3144: static PyObject* THPLinalgEigBackward0_eigenvectors_getter(THPCppFunction *self, void *_unused) {
3145:   HANDLE_TH_ERRORS
3146:   const auto& prop = static_cast<LinalgEigBackward0*>(self->cdata.get())->eigenvectors_;
3147:   return THPVariable_Wrap(prop.unpack(self->cdata));
3148:   END_HANDLE_TH_ERRORS
3149: }
3150: 
3151: static PyObject* THPLinalgEigBackward0_eigenvectors_raw_getter(THPCppFunction *self, void *_unused) {
3152:   HANDLE_TH_ERRORS
3153:   const auto& prop = static_cast<LinalgEigBackward0*>(self->cdata.get())->eigenvectors_;
3154:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3155:   return obj.release().ptr();
3156:   END_HANDLE_TH_ERRORS
3157: }
3158: 
3159: static struct PyGetSetDef LinalgEigBackward0_properties[] = {
3160:   THP_FUNCTION_DEFAULT_PROPERTIES,
3161:   {(char*)"_saved_eigenvalues", (getter)THPLinalgEigBackward0_eigenvalues_getter, nullptr, nullptr, nullptr},
3162:   {(char*)"_raw_saved_eigenvalues", (getter)THPLinalgEigBackward0_eigenvalues_raw_getter, nullptr, nullptr, nullptr},
3163:   {(char*)"_saved_eigenvectors", (getter)THPLinalgEigBackward0_eigenvectors_getter, nullptr, nullptr, nullptr},
3164:   {(char*)"_raw_saved_eigenvectors", (getter)THPLinalgEigBackward0_eigenvectors_raw_getter, nullptr, nullptr, nullptr},
3165:   {nullptr} /* sentinel */
3166: };
3167: 
3168: static PyObject* THPRot90Backward0_dims_getter(THPCppFunction *self, void *_unused) {
3169:   HANDLE_TH_ERRORS
3170:   auto prop = static_cast<Rot90Backward0*>(self->cdata.get())->dims;
3171:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3172:   for (auto i : c10::irange(prop.size())) {
3173:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
3174:   }
3175:   return tup;
3176:   END_HANDLE_TH_ERRORS
3177: }
3178: 
3179: static PyObject* THPRot90Backward0_k_getter(THPCppFunction *self, void *_unused) {
3180:   HANDLE_TH_ERRORS
3181:   auto prop = static_cast<Rot90Backward0*>(self->cdata.get())->k;
3182:   return PyLong_FromUnsignedLong((int64_t) prop);
3183:   END_HANDLE_TH_ERRORS
3184: }
3185: 
3186: static struct PyGetSetDef Rot90Backward0_properties[] = {
3187:   THP_FUNCTION_DEFAULT_PROPERTIES,
3188:   {(char*)"_saved_dims", (getter)THPRot90Backward0_dims_getter, nullptr, nullptr, nullptr},
3189:   {(char*)"_saved_k", (getter)THPRot90Backward0_k_getter, nullptr, nullptr, nullptr},
3190:   {nullptr} /* sentinel */
3191: };
3192: 
3193: static PyObject* THPTrilBackward0_diagonal_getter(THPCppFunction *self, void *_unused) {
3194:   HANDLE_TH_ERRORS
3195:   auto prop = static_cast<TrilBackward0*>(self->cdata.get())->diagonal;
3196:   if (auto m = prop.maybe_as_int()) {
3197:     return PyLong_FromUnsignedLong(*m);
3198:   } else {
3199:     return py::cast(prop).release().ptr();
3200:   }
```

- EN: The main execution path in this span is carried by `THPLinalgEigBackward0_eigenvalues_getter`, `THPVariable_Wrap`, `THPLinalgEigBackward0_eigenvalues_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPLinalgEigBackward0_eigenvalues_getter`, `THPVariable_Wrap`, `THPLinalgEigBackward0_eigenvalues_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3201-3280

```cpp
3201:   END_HANDLE_TH_ERRORS
3202: }
3203: 
3204: static struct PyGetSetDef TrilBackward0_properties[] = {
3205:   THP_FUNCTION_DEFAULT_PROPERTIES,
3206:   {(char*)"_saved_diagonal", (getter)THPTrilBackward0_diagonal_getter, nullptr, nullptr, nullptr},
3207:   {nullptr} /* sentinel */
3208: };
3209: 
3210: 
3211: 
3212: static struct PyGetSetDef TruncBackward0_properties[] = {
3213:   THP_FUNCTION_DEFAULT_PROPERTIES,
3214: 
3215:   {nullptr} /* sentinel */
3216: };
3217: 
3218: static PyObject* THPToDenseBackward0_self_getter(THPCppFunction *self, void *_unused) {
3219:   HANDLE_TH_ERRORS
3220:   const auto& prop = static_cast<ToDenseBackward0*>(self->cdata.get())->self_;
3221:   return THPVariable_Wrap(prop.unpack(self->cdata));
3222:   END_HANDLE_TH_ERRORS
3223: }
3224: 
3225: static PyObject* THPToDenseBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
3226:   HANDLE_TH_ERRORS
3227:   const auto& prop = static_cast<ToDenseBackward0*>(self->cdata.get())->self_;
3228:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3229:   return obj.release().ptr();
3230:   END_HANDLE_TH_ERRORS
3231: }
3232: 
3233: static struct PyGetSetDef ToDenseBackward0_properties[] = {
3234:   THP_FUNCTION_DEFAULT_PROPERTIES,
3235:   {(char*)"_saved_self", (getter)THPToDenseBackward0_self_getter, nullptr, nullptr, nullptr},
3236:   {(char*)"_raw_saved_self", (getter)THPToDenseBackward0_self_raw_getter, nullptr, nullptr, nullptr},
3237:   {nullptr} /* sentinel */
3238: };
3239: 
3240: static PyObject* THPVarBackward0_correction_getter(THPCppFunction *self, void *_unused) {
3241:   HANDLE_TH_ERRORS
3242:   auto opt_prop = static_cast<VarBackward0*>(self->cdata.get())->correction;
3243:   if (!opt_prop.has_value()) {
3244:     Py_RETURN_NONE;
3245:   }
3246:   auto prop = opt_prop.value();
3247:   if (prop.isComplex()) {
3248:     auto cprop = prop.to<c10::complex<double>>();
3249:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
3250:   } else if (prop.isFloatingPoint()) {
3251:     return PyFloat_FromDouble(prop.to<double>());
3252:   } else if (prop.isIntegral(/*includeBool=*/false)) {
3253:     return PyLong_FromLong(prop.to<int64_t>());
3254:   } else if (prop.isBoolean()) {
3255:     if (prop.to<bool>()) {
3256:       Py_RETURN_TRUE;
3257:     } else {
3258:       Py_RETURN_FALSE;
3259:     }
3260:   } else {
3261:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
3262:     return nullptr;
3263:   }
3264:   END_HANDLE_TH_ERRORS
3265: }
3266: 
3267: static PyObject* THPVarBackward0_dim_getter(THPCppFunction *self, void *_unused) {
3268:   HANDLE_TH_ERRORS
3269:   auto opt_prop = static_cast<VarBackward0*>(self->cdata.get())->dim;
3270:   if (!opt_prop.list.has_value()) {
3271:     Py_RETURN_NONE;
3272:   }
3273:   auto prop = opt_prop.list.value();
3274:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3275:   for (auto i : c10::irange(prop.size())) {
3276:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
3277:   }
3278:   return tup;
3279:   END_HANDLE_TH_ERRORS
3280: }
```

- EN: The main execution path in this span is carried by `THPToDenseBackward0_self_getter`, `THPVariable_Wrap`, `THPToDenseBackward0_self_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPToDenseBackward0_self_getter`, `THPVariable_Wrap`, `THPToDenseBackward0_self_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3281-3360

```cpp
3281: 
3282: static PyObject* THPVarBackward0_keepdim_getter(THPCppFunction *self, void *_unused) {
3283:   HANDLE_TH_ERRORS
3284:   auto prop = static_cast<VarBackward0*>(self->cdata.get())->keepdim;
3285:   if (prop) {
3286:     Py_RETURN_TRUE;
3287:   } else {
3288:     Py_RETURN_FALSE;
3289:   }
3290:   END_HANDLE_TH_ERRORS
3291: }
3292: 
3293: static PyObject* THPVarBackward0_self_getter(THPCppFunction *self, void *_unused) {
3294:   HANDLE_TH_ERRORS
3295:   const auto& prop = static_cast<VarBackward0*>(self->cdata.get())->self_;
3296:   return THPVariable_Wrap(prop.unpack(self->cdata));
3297:   END_HANDLE_TH_ERRORS
3298: }
3299: 
3300: static PyObject* THPVarBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
3301:   HANDLE_TH_ERRORS
3302:   const auto& prop = static_cast<VarBackward0*>(self->cdata.get())->self_;
3303:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3304:   return obj.release().ptr();
3305:   END_HANDLE_TH_ERRORS
3306: }
3307: 
3308: static struct PyGetSetDef VarBackward0_properties[] = {
3309:   THP_FUNCTION_DEFAULT_PROPERTIES,
3310:   {(char*)"_saved_correction", (getter)THPVarBackward0_correction_getter, nullptr, nullptr, nullptr},
3311:   {(char*)"_saved_dim", (getter)THPVarBackward0_dim_getter, nullptr, nullptr, nullptr},
3312:   {(char*)"_saved_keepdim", (getter)THPVarBackward0_keepdim_getter, nullptr, nullptr, nullptr},
3313:   {(char*)"_saved_self", (getter)THPVarBackward0_self_getter, nullptr, nullptr, nullptr},
3314:   {(char*)"_raw_saved_self", (getter)THPVarBackward0_self_raw_getter, nullptr, nullptr, nullptr},
3315:   {nullptr} /* sentinel */
3316: };
3317: 
3318: static PyObject* THPVarMeanBackward0_correction_getter(THPCppFunction *self, void *_unused) {
3319:   HANDLE_TH_ERRORS
3320:   auto opt_prop = static_cast<VarMeanBackward0*>(self->cdata.get())->correction;
3321:   if (!opt_prop.has_value()) {
3322:     Py_RETURN_NONE;
3323:   }
3324:   auto prop = opt_prop.value();
3325:   if (prop.isComplex()) {
3326:     auto cprop = prop.to<c10::complex<double>>();
3327:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
3328:   } else if (prop.isFloatingPoint()) {
3329:     return PyFloat_FromDouble(prop.to<double>());
3330:   } else if (prop.isIntegral(/*includeBool=*/false)) {
3331:     return PyLong_FromLong(prop.to<int64_t>());
3332:   } else if (prop.isBoolean()) {
3333:     if (prop.to<bool>()) {
3334:       Py_RETURN_TRUE;
3335:     } else {
3336:       Py_RETURN_FALSE;
3337:     }
3338:   } else {
3339:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
3340:     return nullptr;
3341:   }
3342:   END_HANDLE_TH_ERRORS
3343: }
3344: 
3345: static PyObject* THPVarMeanBackward0_dim_getter(THPCppFunction *self, void *_unused) {
3346:   HANDLE_TH_ERRORS
3347:   auto opt_prop = static_cast<VarMeanBackward0*>(self->cdata.get())->dim;
3348:   if (!opt_prop.list.has_value()) {
3349:     Py_RETURN_NONE;
3350:   }
3351:   auto prop = opt_prop.list.value();
3352:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3353:   for (auto i : c10::irange(prop.size())) {
3354:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
3355:   }
3356:   return tup;
3357:   END_HANDLE_TH_ERRORS
3358: }
3359: 
3360: static PyObject* THPVarMeanBackward0_keepdim_getter(THPCppFunction *self, void *_unused) {
```

- EN: The main execution path in this span is carried by `THPVarBackward0_keepdim_getter`, `THPVarBackward0_self_getter`, `THPVariable_Wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVarBackward0_keepdim_getter`, `THPVarBackward0_self_getter`, `THPVariable_Wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3361-3440

```cpp
3361:   HANDLE_TH_ERRORS
3362:   auto prop = static_cast<VarMeanBackward0*>(self->cdata.get())->keepdim;
3363:   if (prop) {
3364:     Py_RETURN_TRUE;
3365:   } else {
3366:     Py_RETURN_FALSE;
3367:   }
3368:   END_HANDLE_TH_ERRORS
3369: }
3370: 
3371: static PyObject* THPVarMeanBackward0_self_getter(THPCppFunction *self, void *_unused) {
3372:   HANDLE_TH_ERRORS
3373:   const auto& prop = static_cast<VarMeanBackward0*>(self->cdata.get())->self_;
3374:   return THPVariable_Wrap(prop.unpack(self->cdata));
3375:   END_HANDLE_TH_ERRORS
3376: }
3377: 
3378: static PyObject* THPVarMeanBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
3379:   HANDLE_TH_ERRORS
3380:   const auto& prop = static_cast<VarMeanBackward0*>(self->cdata.get())->self_;
3381:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3382:   return obj.release().ptr();
3383:   END_HANDLE_TH_ERRORS
3384: }
3385: 
3386: static struct PyGetSetDef VarMeanBackward0_properties[] = {
3387:   THP_FUNCTION_DEFAULT_PROPERTIES,
3388:   {(char*)"_saved_correction", (getter)THPVarMeanBackward0_correction_getter, nullptr, nullptr, nullptr},
3389:   {(char*)"_saved_dim", (getter)THPVarMeanBackward0_dim_getter, nullptr, nullptr, nullptr},
3390:   {(char*)"_saved_keepdim", (getter)THPVarMeanBackward0_keepdim_getter, nullptr, nullptr, nullptr},
3391:   {(char*)"_saved_self", (getter)THPVarMeanBackward0_self_getter, nullptr, nullptr, nullptr},
3392:   {(char*)"_raw_saved_self", (getter)THPVarMeanBackward0_self_raw_getter, nullptr, nullptr, nullptr},
3393:   {nullptr} /* sentinel */
3394: };
3395: 
3396: static PyObject* THPSparseCooTensorWithDimsAndTensorsBackward0_result_getter(THPCppFunction *self, void *_unused) {
3397:   HANDLE_TH_ERRORS
3398:   const auto& prop = static_cast<SparseCooTensorWithDimsAndTensorsBackward0*>(self->cdata.get())->result_;
3399:   return THPVariable_Wrap(prop.unpack(self->cdata));
3400:   END_HANDLE_TH_ERRORS
3401: }
3402: 
3403: static PyObject* THPSparseCooTensorWithDimsAndTensorsBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
3404:   HANDLE_TH_ERRORS
3405:   const auto& prop = static_cast<SparseCooTensorWithDimsAndTensorsBackward0*>(self->cdata.get())->result_;
3406:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3407:   return obj.release().ptr();
3408:   END_HANDLE_TH_ERRORS
3409: }
3410: 
3411: static struct PyGetSetDef SparseCooTensorWithDimsAndTensorsBackward0_properties[] = {
3412:   THP_FUNCTION_DEFAULT_PROPERTIES,
3413:   {(char*)"_saved_result", (getter)THPSparseCooTensorWithDimsAndTensorsBackward0_result_getter, nullptr, nullptr, nullptr},
3414:   {(char*)"_raw_saved_result", (getter)THPSparseCooTensorWithDimsAndTensorsBackward0_result_raw_getter, nullptr, nullptr, nullptr},
3415:   {nullptr} /* sentinel */
3416: };
3417: 
3418: 
3419: 
3420: static struct PyGetSetDef StandardGammaGradBackward0_properties[] = {
3421:   THP_FUNCTION_DEFAULT_PROPERTIES,
3422: 
3423:   {nullptr} /* sentinel */
3424: };
3425: 
3426: static PyObject* THPBinaryCrossEntropyWithLogitsBackward0_pos_weight_getter(THPCppFunction *self, void *_unused) {
3427:   HANDLE_TH_ERRORS
3428:   const auto& prop = static_cast<BinaryCrossEntropyWithLogitsBackward0*>(self->cdata.get())->pos_weight_;
3429:   return THPVariable_Wrap(prop.unpack(self->cdata));
3430:   END_HANDLE_TH_ERRORS
3431: }
3432: 
3433: static PyObject* THPBinaryCrossEntropyWithLogitsBackward0_pos_weight_raw_getter(THPCppFunction *self, void *_unused) {
3434:   HANDLE_TH_ERRORS
3435:   const auto& prop = static_cast<BinaryCrossEntropyWithLogitsBackward0*>(self->cdata.get())->pos_weight_;
3436:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3437:   return obj.release().ptr();
3438:   END_HANDLE_TH_ERRORS
3439: }
3440: 
```

- EN: The main execution path in this span is carried by `THPVarMeanBackward0_self_getter`, `THPVariable_Wrap`, `THPVarMeanBackward0_self_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVarMeanBackward0_self_getter`, `THPVariable_Wrap`, `THPVarMeanBackward0_self_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3441-3520

```cpp
3441: static PyObject* THPBinaryCrossEntropyWithLogitsBackward0_reduction_getter(THPCppFunction *self, void *_unused) {
3442:   HANDLE_TH_ERRORS
3443:   auto prop = static_cast<BinaryCrossEntropyWithLogitsBackward0*>(self->cdata.get())->reduction;
3444:   return PyLong_FromUnsignedLong((int64_t) prop);
3445:   END_HANDLE_TH_ERRORS
3446: }
3447: 
3448: static PyObject* THPBinaryCrossEntropyWithLogitsBackward0_self_getter(THPCppFunction *self, void *_unused) {
3449:   HANDLE_TH_ERRORS
3450:   const auto& prop = static_cast<BinaryCrossEntropyWithLogitsBackward0*>(self->cdata.get())->self_;
3451:   return THPVariable_Wrap(prop.unpack(self->cdata));
3452:   END_HANDLE_TH_ERRORS
3453: }
3454: 
3455: static PyObject* THPBinaryCrossEntropyWithLogitsBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
3456:   HANDLE_TH_ERRORS
3457:   const auto& prop = static_cast<BinaryCrossEntropyWithLogitsBackward0*>(self->cdata.get())->self_;
3458:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3459:   return obj.release().ptr();
3460:   END_HANDLE_TH_ERRORS
3461: }
3462: 
3463: static PyObject* THPBinaryCrossEntropyWithLogitsBackward0_target_getter(THPCppFunction *self, void *_unused) {
3464:   HANDLE_TH_ERRORS
3465:   const auto& prop = static_cast<BinaryCrossEntropyWithLogitsBackward0*>(self->cdata.get())->target_;
3466:   return THPVariable_Wrap(prop.unpack(self->cdata));
3467:   END_HANDLE_TH_ERRORS
3468: }
3469: 
3470: static PyObject* THPBinaryCrossEntropyWithLogitsBackward0_target_raw_getter(THPCppFunction *self, void *_unused) {
3471:   HANDLE_TH_ERRORS
3472:   const auto& prop = static_cast<BinaryCrossEntropyWithLogitsBackward0*>(self->cdata.get())->target_;
3473:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3474:   return obj.release().ptr();
3475:   END_HANDLE_TH_ERRORS
3476: }
3477: 
3478: static PyObject* THPBinaryCrossEntropyWithLogitsBackward0_weight_getter(THPCppFunction *self, void *_unused) {
3479:   HANDLE_TH_ERRORS
3480:   const auto& prop = static_cast<BinaryCrossEntropyWithLogitsBackward0*>(self->cdata.get())->weight_;
3481:   return THPVariable_Wrap(prop.unpack(self->cdata));
3482:   END_HANDLE_TH_ERRORS
3483: }
3484: 
3485: static PyObject* THPBinaryCrossEntropyWithLogitsBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
3486:   HANDLE_TH_ERRORS
3487:   const auto& prop = static_cast<BinaryCrossEntropyWithLogitsBackward0*>(self->cdata.get())->weight_;
3488:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3489:   return obj.release().ptr();
3490:   END_HANDLE_TH_ERRORS
3491: }
3492: 
3493: static struct PyGetSetDef BinaryCrossEntropyWithLogitsBackward0_properties[] = {
3494:   THP_FUNCTION_DEFAULT_PROPERTIES,
3495:   {(char*)"_saved_pos_weight", (getter)THPBinaryCrossEntropyWithLogitsBackward0_pos_weight_getter, nullptr, nullptr, nullptr},
3496:   {(char*)"_raw_saved_pos_weight", (getter)THPBinaryCrossEntropyWithLogitsBackward0_pos_weight_raw_getter, nullptr, nullptr, nullptr},
3497:   {(char*)"_saved_reduction", (getter)THPBinaryCrossEntropyWithLogitsBackward0_reduction_getter, nullptr, nullptr, nullptr},
3498:   {(char*)"_saved_self", (getter)THPBinaryCrossEntropyWithLogitsBackward0_self_getter, nullptr, nullptr, nullptr},
3499:   {(char*)"_raw_saved_self", (getter)THPBinaryCrossEntropyWithLogitsBackward0_self_raw_getter, nullptr, nullptr, nullptr},
3500:   {(char*)"_saved_target", (getter)THPBinaryCrossEntropyWithLogitsBackward0_target_getter, nullptr, nullptr, nullptr},
3501:   {(char*)"_raw_saved_target", (getter)THPBinaryCrossEntropyWithLogitsBackward0_target_raw_getter, nullptr, nullptr, nullptr},
3502:   {(char*)"_saved_weight", (getter)THPBinaryCrossEntropyWithLogitsBackward0_weight_getter, nullptr, nullptr, nullptr},
3503:   {(char*)"_raw_saved_weight", (getter)THPBinaryCrossEntropyWithLogitsBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
3504:   {nullptr} /* sentinel */
3505: };
3506: 
3507: static PyObject* THPMultiMarginLossBackward0_margin_getter(THPCppFunction *self, void *_unused) {
3508:   HANDLE_TH_ERRORS
3509:   auto prop = static_cast<MultiMarginLossBackward0*>(self->cdata.get())->margin;
3510:   if (prop.isComplex()) {
3511:     auto cprop = prop.to<c10::complex<double>>();
3512:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
3513:   } else if (prop.isFloatingPoint()) {
3514:     return PyFloat_FromDouble(prop.to<double>());
3515:   } else if (prop.isIntegral(/*includeBool=*/false)) {
3516:     return PyLong_FromLong(prop.to<int64_t>());
3517:   } else if (prop.isBoolean()) {
3518:     if (prop.to<bool>()) {
3519:       Py_RETURN_TRUE;
3520:     } else {
```

- EN: The main execution path in this span is carried by `THPBinaryCrossEntropyWithLogitsBackward0_reduction_getter`, `PyLong_FromUnsignedLong`, `THPBinaryCrossEntropyWithLogitsBackward0_self_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPBinaryCrossEntropyWithLogitsBackward0_reduction_getter`, `PyLong_FromUnsignedLong`, `THPBinaryCrossEntropyWithLogitsBackward0_self_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3521-3600

```cpp
3521:       Py_RETURN_FALSE;
3522:     }
3523:   } else {
3524:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
3525:     return nullptr;
3526:   }
3527:   END_HANDLE_TH_ERRORS
3528: }
3529: 
3530: static PyObject* THPMultiMarginLossBackward0_p_getter(THPCppFunction *self, void *_unused) {
3531:   HANDLE_TH_ERRORS
3532:   auto prop = static_cast<MultiMarginLossBackward0*>(self->cdata.get())->p;
3533:   if (prop.isComplex()) {
3534:     auto cprop = prop.to<c10::complex<double>>();
3535:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
3536:   } else if (prop.isFloatingPoint()) {
3537:     return PyFloat_FromDouble(prop.to<double>());
3538:   } else if (prop.isIntegral(/*includeBool=*/false)) {
3539:     return PyLong_FromLong(prop.to<int64_t>());
3540:   } else if (prop.isBoolean()) {
3541:     if (prop.to<bool>()) {
3542:       Py_RETURN_TRUE;
3543:     } else {
3544:       Py_RETURN_FALSE;
3545:     }
3546:   } else {
3547:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
3548:     return nullptr;
3549:   }
3550:   END_HANDLE_TH_ERRORS
3551: }
3552: 
3553: static PyObject* THPMultiMarginLossBackward0_reduction_getter(THPCppFunction *self, void *_unused) {
3554:   HANDLE_TH_ERRORS
3555:   auto prop = static_cast<MultiMarginLossBackward0*>(self->cdata.get())->reduction;
3556:   return PyLong_FromUnsignedLong((int64_t) prop);
3557:   END_HANDLE_TH_ERRORS
3558: }
3559: 
3560: static PyObject* THPMultiMarginLossBackward0_self_getter(THPCppFunction *self, void *_unused) {
3561:   HANDLE_TH_ERRORS
3562:   const auto& prop = static_cast<MultiMarginLossBackward0*>(self->cdata.get())->self_;
3563:   return THPVariable_Wrap(prop.unpack(self->cdata));
3564:   END_HANDLE_TH_ERRORS
3565: }
3566: 
3567: static PyObject* THPMultiMarginLossBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
3568:   HANDLE_TH_ERRORS
3569:   const auto& prop = static_cast<MultiMarginLossBackward0*>(self->cdata.get())->self_;
3570:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3571:   return obj.release().ptr();
3572:   END_HANDLE_TH_ERRORS
3573: }
3574: 
3575: static PyObject* THPMultiMarginLossBackward0_target_getter(THPCppFunction *self, void *_unused) {
3576:   HANDLE_TH_ERRORS
3577:   const auto& prop = static_cast<MultiMarginLossBackward0*>(self->cdata.get())->target_;
3578:   return THPVariable_Wrap(prop.unpack(self->cdata));
3579:   END_HANDLE_TH_ERRORS
3580: }
3581: 
3582: static PyObject* THPMultiMarginLossBackward0_target_raw_getter(THPCppFunction *self, void *_unused) {
3583:   HANDLE_TH_ERRORS
3584:   const auto& prop = static_cast<MultiMarginLossBackward0*>(self->cdata.get())->target_;
3585:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3586:   return obj.release().ptr();
3587:   END_HANDLE_TH_ERRORS
3588: }
3589: 
3590: static PyObject* THPMultiMarginLossBackward0_weight_getter(THPCppFunction *self, void *_unused) {
3591:   HANDLE_TH_ERRORS
3592:   const auto& prop = static_cast<MultiMarginLossBackward0*>(self->cdata.get())->weight_;
3593:   return THPVariable_Wrap(prop.unpack(self->cdata));
3594:   END_HANDLE_TH_ERRORS
3595: }
3596: 
3597: static PyObject* THPMultiMarginLossBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
3598:   HANDLE_TH_ERRORS
3599:   const auto& prop = static_cast<MultiMarginLossBackward0*>(self->cdata.get())->weight_;
3600:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
```

- EN: The main execution path in this span is carried by `PyErr_SetString`, `THPMultiMarginLossBackward0_p_getter`, `PyComplex_FromDoubles`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyErr_SetString`, `THPMultiMarginLossBackward0_p_getter`, `PyComplex_FromDoubles` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3601-3680

```cpp
3601:   return obj.release().ptr();
3602:   END_HANDLE_TH_ERRORS
3603: }
3604: 
3605: static struct PyGetSetDef MultiMarginLossBackward0_properties[] = {
3606:   THP_FUNCTION_DEFAULT_PROPERTIES,
3607:   {(char*)"_saved_margin", (getter)THPMultiMarginLossBackward0_margin_getter, nullptr, nullptr, nullptr},
3608:   {(char*)"_saved_p", (getter)THPMultiMarginLossBackward0_p_getter, nullptr, nullptr, nullptr},
3609:   {(char*)"_saved_reduction", (getter)THPMultiMarginLossBackward0_reduction_getter, nullptr, nullptr, nullptr},
3610:   {(char*)"_saved_self", (getter)THPMultiMarginLossBackward0_self_getter, nullptr, nullptr, nullptr},
3611:   {(char*)"_raw_saved_self", (getter)THPMultiMarginLossBackward0_self_raw_getter, nullptr, nullptr, nullptr},
3612:   {(char*)"_saved_target", (getter)THPMultiMarginLossBackward0_target_getter, nullptr, nullptr, nullptr},
3613:   {(char*)"_raw_saved_target", (getter)THPMultiMarginLossBackward0_target_raw_getter, nullptr, nullptr, nullptr},
3614:   {(char*)"_saved_weight", (getter)THPMultiMarginLossBackward0_weight_getter, nullptr, nullptr, nullptr},
3615:   {(char*)"_raw_saved_weight", (getter)THPMultiMarginLossBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
3616:   {nullptr} /* sentinel */
3617: };
3618: 
3619: static PyObject* THPMultilabelMarginLossBackward0_reduction_getter(THPCppFunction *self, void *_unused) {
3620:   HANDLE_TH_ERRORS
3621:   auto prop = static_cast<MultilabelMarginLossBackward0*>(self->cdata.get())->reduction;
3622:   return PyLong_FromUnsignedLong((int64_t) prop);
3623:   END_HANDLE_TH_ERRORS
3624: }
3625: 
3626: static PyObject* THPMultilabelMarginLossBackward0_self_getter(THPCppFunction *self, void *_unused) {
3627:   HANDLE_TH_ERRORS
3628:   const auto& prop = static_cast<MultilabelMarginLossBackward0*>(self->cdata.get())->self_;
3629:   return THPVariable_Wrap(prop.unpack(self->cdata));
3630:   END_HANDLE_TH_ERRORS
3631: }
3632: 
3633: static PyObject* THPMultilabelMarginLossBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
3634:   HANDLE_TH_ERRORS
3635:   const auto& prop = static_cast<MultilabelMarginLossBackward0*>(self->cdata.get())->self_;
3636:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3637:   return obj.release().ptr();
3638:   END_HANDLE_TH_ERRORS
3639: }
3640: 
3641: static PyObject* THPMultilabelMarginLossBackward0_target_getter(THPCppFunction *self, void *_unused) {
3642:   HANDLE_TH_ERRORS
3643:   const auto& prop = static_cast<MultilabelMarginLossBackward0*>(self->cdata.get())->target_;
3644:   return THPVariable_Wrap(prop.unpack(self->cdata));
3645:   END_HANDLE_TH_ERRORS
3646: }
3647: 
3648: static PyObject* THPMultilabelMarginLossBackward0_target_raw_getter(THPCppFunction *self, void *_unused) {
3649:   HANDLE_TH_ERRORS
3650:   const auto& prop = static_cast<MultilabelMarginLossBackward0*>(self->cdata.get())->target_;
3651:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3652:   return obj.release().ptr();
3653:   END_HANDLE_TH_ERRORS
3654: }
3655: 
3656: static PyObject* THPMultilabelMarginLossBackward0_is_target_getter(THPCppFunction *self, void *_unused) {
3657:   HANDLE_TH_ERRORS
3658:   const auto& prop = static_cast<MultilabelMarginLossBackward0*>(self->cdata.get())->is_target_;
3659:   return THPVariable_Wrap(prop.unpack(self->cdata));
3660:   END_HANDLE_TH_ERRORS
3661: }
3662: 
3663: static PyObject* THPMultilabelMarginLossBackward0_is_target_raw_getter(THPCppFunction *self, void *_unused) {
3664:   HANDLE_TH_ERRORS
3665:   const auto& prop = static_cast<MultilabelMarginLossBackward0*>(self->cdata.get())->is_target_;
3666:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3667:   return obj.release().ptr();
3668:   END_HANDLE_TH_ERRORS
3669: }
3670: 
3671: static struct PyGetSetDef MultilabelMarginLossBackward0_properties[] = {
3672:   THP_FUNCTION_DEFAULT_PROPERTIES,
3673:   {(char*)"_saved_reduction", (getter)THPMultilabelMarginLossBackward0_reduction_getter, nullptr, nullptr, nullptr},
3674:   {(char*)"_saved_self", (getter)THPMultilabelMarginLossBackward0_self_getter, nullptr, nullptr, nullptr},
3675:   {(char*)"_raw_saved_self", (getter)THPMultilabelMarginLossBackward0_self_raw_getter, nullptr, nullptr, nullptr},
3676:   {(char*)"_saved_target", (getter)THPMultilabelMarginLossBackward0_target_getter, nullptr, nullptr, nullptr},
3677:   {(char*)"_raw_saved_target", (getter)THPMultilabelMarginLossBackward0_target_raw_getter, nullptr, nullptr, nullptr},
3678:   {(char*)"_saved_is_target", (getter)THPMultilabelMarginLossBackward0_is_target_getter, nullptr, nullptr, nullptr},
3679:   {(char*)"_raw_saved_is_target", (getter)THPMultilabelMarginLossBackward0_is_target_raw_getter, nullptr, nullptr, nullptr},
3680:   {nullptr} /* sentinel */
```

- EN: The main execution path in this span is carried by `THPMultilabelMarginLossBackward0_reduction_getter`, `PyLong_FromUnsignedLong`, `THPMultilabelMarginLossBackward0_self_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPMultilabelMarginLossBackward0_reduction_getter`, `PyLong_FromUnsignedLong`, `THPMultilabelMarginLossBackward0_self_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3681-3760

```cpp
3681: };
3682: 
3683: static PyObject* THPNllLossBackward0_ignore_index_getter(THPCppFunction *self, void *_unused) {
3684:   HANDLE_TH_ERRORS
3685:   auto prop = static_cast<NllLossBackward0*>(self->cdata.get())->ignore_index;
3686:   if (auto m = prop.maybe_as_int()) {
3687:     return PyLong_FromUnsignedLong(*m);
3688:   } else {
3689:     return py::cast(prop).release().ptr();
3690:   }
3691:   END_HANDLE_TH_ERRORS
3692: }
3693: 
3694: static PyObject* THPNllLossBackward0_reduction_getter(THPCppFunction *self, void *_unused) {
3695:   HANDLE_TH_ERRORS
3696:   auto prop = static_cast<NllLossBackward0*>(self->cdata.get())->reduction;
3697:   return PyLong_FromUnsignedLong((int64_t) prop);
3698:   END_HANDLE_TH_ERRORS
3699: }
3700: 
3701: static PyObject* THPNllLossBackward0_self_getter(THPCppFunction *self, void *_unused) {
3702:   HANDLE_TH_ERRORS
3703:   const auto& prop = static_cast<NllLossBackward0*>(self->cdata.get())->self_;
3704:   return THPVariable_Wrap(prop.unpack(self->cdata));
3705:   END_HANDLE_TH_ERRORS
3706: }
3707: 
3708: static PyObject* THPNllLossBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
3709:   HANDLE_TH_ERRORS
3710:   const auto& prop = static_cast<NllLossBackward0*>(self->cdata.get())->self_;
3711:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3712:   return obj.release().ptr();
3713:   END_HANDLE_TH_ERRORS
3714: }
3715: 
3716: static PyObject* THPNllLossBackward0_target_getter(THPCppFunction *self, void *_unused) {
3717:   HANDLE_TH_ERRORS
3718:   const auto& prop = static_cast<NllLossBackward0*>(self->cdata.get())->target_;
3719:   return THPVariable_Wrap(prop.unpack(self->cdata));
3720:   END_HANDLE_TH_ERRORS
3721: }
3722: 
3723: static PyObject* THPNllLossBackward0_target_raw_getter(THPCppFunction *self, void *_unused) {
3724:   HANDLE_TH_ERRORS
3725:   const auto& prop = static_cast<NllLossBackward0*>(self->cdata.get())->target_;
3726:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3727:   return obj.release().ptr();
3728:   END_HANDLE_TH_ERRORS
3729: }
3730: 
3731: static PyObject* THPNllLossBackward0_weight_getter(THPCppFunction *self, void *_unused) {
3732:   HANDLE_TH_ERRORS
3733:   const auto& prop = static_cast<NllLossBackward0*>(self->cdata.get())->weight_;
3734:   return THPVariable_Wrap(prop.unpack(self->cdata));
3735:   END_HANDLE_TH_ERRORS
3736: }
3737: 
3738: static PyObject* THPNllLossBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
3739:   HANDLE_TH_ERRORS
3740:   const auto& prop = static_cast<NllLossBackward0*>(self->cdata.get())->weight_;
3741:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3742:   return obj.release().ptr();
3743:   END_HANDLE_TH_ERRORS
3744: }
3745: 
3746: static PyObject* THPNllLossBackward0_total_weight_getter(THPCppFunction *self, void *_unused) {
3747:   HANDLE_TH_ERRORS
3748:   const auto& prop = static_cast<NllLossBackward0*>(self->cdata.get())->total_weight_;
3749:   return THPVariable_Wrap(prop.unpack(self->cdata));
3750:   END_HANDLE_TH_ERRORS
3751: }
3752: 
3753: static PyObject* THPNllLossBackward0_total_weight_raw_getter(THPCppFunction *self, void *_unused) {
3754:   HANDLE_TH_ERRORS
3755:   const auto& prop = static_cast<NllLossBackward0*>(self->cdata.get())->total_weight_;
3756:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3757:   return obj.release().ptr();
3758:   END_HANDLE_TH_ERRORS
3759: }
3760: 
```

- EN: The main execution path in this span is carried by `THPNllLossBackward0_ignore_index_getter`, `PyLong_FromUnsignedLong`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPNllLossBackward0_ignore_index_getter`, `PyLong_FromUnsignedLong`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3761-3840

```cpp
3761: static struct PyGetSetDef NllLossBackward0_properties[] = {
3762:   THP_FUNCTION_DEFAULT_PROPERTIES,
3763:   {(char*)"_saved_ignore_index", (getter)THPNllLossBackward0_ignore_index_getter, nullptr, nullptr, nullptr},
3764:   {(char*)"_saved_reduction", (getter)THPNllLossBackward0_reduction_getter, nullptr, nullptr, nullptr},
3765:   {(char*)"_saved_self", (getter)THPNllLossBackward0_self_getter, nullptr, nullptr, nullptr},
3766:   {(char*)"_raw_saved_self", (getter)THPNllLossBackward0_self_raw_getter, nullptr, nullptr, nullptr},
3767:   {(char*)"_saved_target", (getter)THPNllLossBackward0_target_getter, nullptr, nullptr, nullptr},
3768:   {(char*)"_raw_saved_target", (getter)THPNllLossBackward0_target_raw_getter, nullptr, nullptr, nullptr},
3769:   {(char*)"_saved_weight", (getter)THPNllLossBackward0_weight_getter, nullptr, nullptr, nullptr},
3770:   {(char*)"_raw_saved_weight", (getter)THPNllLossBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
3771:   {(char*)"_saved_total_weight", (getter)THPNllLossBackward0_total_weight_getter, nullptr, nullptr, nullptr},
3772:   {(char*)"_raw_saved_total_weight", (getter)THPNllLossBackward0_total_weight_raw_getter, nullptr, nullptr, nullptr},
3773:   {nullptr} /* sentinel */
3774: };
3775: 
3776: static PyObject* THPPreluKernelBackward0_self_getter(THPCppFunction *self, void *_unused) {
3777:   HANDLE_TH_ERRORS
3778:   const auto& prop = static_cast<PreluKernelBackward0*>(self->cdata.get())->self_;
3779:   return THPVariable_Wrap(prop.unpack(self->cdata));
3780:   END_HANDLE_TH_ERRORS
3781: }
3782: 
3783: static PyObject* THPPreluKernelBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
3784:   HANDLE_TH_ERRORS
3785:   const auto& prop = static_cast<PreluKernelBackward0*>(self->cdata.get())->self_;
3786:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3787:   return obj.release().ptr();
3788:   END_HANDLE_TH_ERRORS
3789: }
3790: 
3791: static PyObject* THPPreluKernelBackward0_weight_getter(THPCppFunction *self, void *_unused) {
3792:   HANDLE_TH_ERRORS
3793:   const auto& prop = static_cast<PreluKernelBackward0*>(self->cdata.get())->weight_;
3794:   return THPVariable_Wrap(prop.unpack(self->cdata));
3795:   END_HANDLE_TH_ERRORS
3796: }
3797: 
3798: static PyObject* THPPreluKernelBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
3799:   HANDLE_TH_ERRORS
3800:   const auto& prop = static_cast<PreluKernelBackward0*>(self->cdata.get())->weight_;
3801:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3802:   return obj.release().ptr();
3803:   END_HANDLE_TH_ERRORS
3804: }
3805: 
3806: static struct PyGetSetDef PreluKernelBackward0_properties[] = {
3807:   THP_FUNCTION_DEFAULT_PROPERTIES,
3808:   {(char*)"_saved_self", (getter)THPPreluKernelBackward0_self_getter, nullptr, nullptr, nullptr},
3809:   {(char*)"_raw_saved_self", (getter)THPPreluKernelBackward0_self_raw_getter, nullptr, nullptr, nullptr},
3810:   {(char*)"_saved_weight", (getter)THPPreluKernelBackward0_weight_getter, nullptr, nullptr, nullptr},
3811:   {(char*)"_raw_saved_weight", (getter)THPPreluKernelBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
3812:   {nullptr} /* sentinel */
3813: };
3814: 
3815: static PyObject* THPRreluWithNoiseBackward0_lower_getter(THPCppFunction *self, void *_unused) {
3816:   HANDLE_TH_ERRORS
3817:   auto prop = static_cast<RreluWithNoiseBackward0*>(self->cdata.get())->lower;
3818:   if (prop.isComplex()) {
3819:     auto cprop = prop.to<c10::complex<double>>();
3820:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
3821:   } else if (prop.isFloatingPoint()) {
3822:     return PyFloat_FromDouble(prop.to<double>());
3823:   } else if (prop.isIntegral(/*includeBool=*/false)) {
3824:     return PyLong_FromLong(prop.to<int64_t>());
3825:   } else if (prop.isBoolean()) {
3826:     if (prop.to<bool>()) {
3827:       Py_RETURN_TRUE;
3828:     } else {
3829:       Py_RETURN_FALSE;
3830:     }
3831:   } else {
3832:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
3833:     return nullptr;
3834:   }
3835:   END_HANDLE_TH_ERRORS
3836: }
3837: 
3838: static PyObject* THPRreluWithNoiseBackward0_noise_getter(THPCppFunction *self, void *_unused) {
3839:   HANDLE_TH_ERRORS
3840:   const auto& prop = static_cast<RreluWithNoiseBackward0*>(self->cdata.get())->noise_;
```

- EN: The main execution path in this span is carried by `THPPreluKernelBackward0_self_getter`, `THPVariable_Wrap`, `THPPreluKernelBackward0_self_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPPreluKernelBackward0_self_getter`, `THPVariable_Wrap`, `THPPreluKernelBackward0_self_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3841-3920

```cpp
3841:   return THPVariable_Wrap(prop.unpack(self->cdata));
3842:   END_HANDLE_TH_ERRORS
3843: }
3844: 
3845: static PyObject* THPRreluWithNoiseBackward0_noise_raw_getter(THPCppFunction *self, void *_unused) {
3846:   HANDLE_TH_ERRORS
3847:   const auto& prop = static_cast<RreluWithNoiseBackward0*>(self->cdata.get())->noise_;
3848:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3849:   return obj.release().ptr();
3850:   END_HANDLE_TH_ERRORS
3851: }
3852: 
3853: static PyObject* THPRreluWithNoiseBackward0_self_getter(THPCppFunction *self, void *_unused) {
3854:   HANDLE_TH_ERRORS
3855:   const auto& prop = static_cast<RreluWithNoiseBackward0*>(self->cdata.get())->self_;
3856:   return THPVariable_Wrap(prop.unpack(self->cdata));
3857:   END_HANDLE_TH_ERRORS
3858: }
3859: 
3860: static PyObject* THPRreluWithNoiseBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
3861:   HANDLE_TH_ERRORS
3862:   const auto& prop = static_cast<RreluWithNoiseBackward0*>(self->cdata.get())->self_;
3863:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3864:   return obj.release().ptr();
3865:   END_HANDLE_TH_ERRORS
3866: }
3867: 
3868: static PyObject* THPRreluWithNoiseBackward0_training_getter(THPCppFunction *self, void *_unused) {
3869:   HANDLE_TH_ERRORS
3870:   auto prop = static_cast<RreluWithNoiseBackward0*>(self->cdata.get())->training;
3871:   if (prop) {
3872:     Py_RETURN_TRUE;
3873:   } else {
3874:     Py_RETURN_FALSE;
3875:   }
3876:   END_HANDLE_TH_ERRORS
3877: }
3878: 
3879: static PyObject* THPRreluWithNoiseBackward0_upper_getter(THPCppFunction *self, void *_unused) {
3880:   HANDLE_TH_ERRORS
3881:   auto prop = static_cast<RreluWithNoiseBackward0*>(self->cdata.get())->upper;
3882:   if (prop.isComplex()) {
3883:     auto cprop = prop.to<c10::complex<double>>();
3884:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
3885:   } else if (prop.isFloatingPoint()) {
3886:     return PyFloat_FromDouble(prop.to<double>());
3887:   } else if (prop.isIntegral(/*includeBool=*/false)) {
3888:     return PyLong_FromLong(prop.to<int64_t>());
3889:   } else if (prop.isBoolean()) {
3890:     if (prop.to<bool>()) {
3891:       Py_RETURN_TRUE;
3892:     } else {
3893:       Py_RETURN_FALSE;
3894:     }
3895:   } else {
3896:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
3897:     return nullptr;
3898:   }
3899:   END_HANDLE_TH_ERRORS
3900: }
3901: 
3902: static struct PyGetSetDef RreluWithNoiseBackward0_properties[] = {
3903:   THP_FUNCTION_DEFAULT_PROPERTIES,
3904:   {(char*)"_saved_lower", (getter)THPRreluWithNoiseBackward0_lower_getter, nullptr, nullptr, nullptr},
3905:   {(char*)"_saved_noise", (getter)THPRreluWithNoiseBackward0_noise_getter, nullptr, nullptr, nullptr},
3906:   {(char*)"_raw_saved_noise", (getter)THPRreluWithNoiseBackward0_noise_raw_getter, nullptr, nullptr, nullptr},
3907:   {(char*)"_saved_self", (getter)THPRreluWithNoiseBackward0_self_getter, nullptr, nullptr, nullptr},
3908:   {(char*)"_raw_saved_self", (getter)THPRreluWithNoiseBackward0_self_raw_getter, nullptr, nullptr, nullptr},
3909:   {(char*)"_saved_training", (getter)THPRreluWithNoiseBackward0_training_getter, nullptr, nullptr, nullptr},
3910:   {(char*)"_saved_upper", (getter)THPRreluWithNoiseBackward0_upper_getter, nullptr, nullptr, nullptr},
3911:   {nullptr} /* sentinel */
3912: };
3913: 
3914: static PyObject* THPSparseSoftmaxBackward0_dim_getter(THPCppFunction *self, void *_unused) {
3915:   HANDLE_TH_ERRORS
3916:   auto prop = static_cast<SparseSoftmaxBackward0*>(self->cdata.get())->dim;
3917:   return PyLong_FromUnsignedLong((int64_t) prop);
3918:   END_HANDLE_TH_ERRORS
3919: }
3920: 
```

- EN: The main execution path in this span is carried by `THPVariable_Wrap`, `THPRreluWithNoiseBackward0_noise_raw_getter`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_Wrap`, `THPRreluWithNoiseBackward0_noise_raw_getter`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3921-4000

```cpp
3921: static PyObject* THPSparseSoftmaxBackward0_self_getter(THPCppFunction *self, void *_unused) {
3922:   HANDLE_TH_ERRORS
3923:   const auto& prop = static_cast<SparseSoftmaxBackward0*>(self->cdata.get())->self_;
3924:   return THPVariable_Wrap(prop.unpack(self->cdata));
3925:   END_HANDLE_TH_ERRORS
3926: }
3927: 
3928: static PyObject* THPSparseSoftmaxBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
3929:   HANDLE_TH_ERRORS
3930:   const auto& prop = static_cast<SparseSoftmaxBackward0*>(self->cdata.get())->self_;
3931:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3932:   return obj.release().ptr();
3933:   END_HANDLE_TH_ERRORS
3934: }
3935: 
3936: static PyObject* THPSparseSoftmaxBackward0_result_getter(THPCppFunction *self, void *_unused) {
3937:   HANDLE_TH_ERRORS
3938:   const auto& prop = static_cast<SparseSoftmaxBackward0*>(self->cdata.get())->result_;
3939:   return THPVariable_Wrap(prop.unpack(self->cdata));
3940:   END_HANDLE_TH_ERRORS
3941: }
3942: 
3943: static PyObject* THPSparseSoftmaxBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
3944:   HANDLE_TH_ERRORS
3945:   const auto& prop = static_cast<SparseSoftmaxBackward0*>(self->cdata.get())->result_;
3946:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3947:   return obj.release().ptr();
3948:   END_HANDLE_TH_ERRORS
3949: }
3950: 
3951: static struct PyGetSetDef SparseSoftmaxBackward0_properties[] = {
3952:   THP_FUNCTION_DEFAULT_PROPERTIES,
3953:   {(char*)"_saved_dim", (getter)THPSparseSoftmaxBackward0_dim_getter, nullptr, nullptr, nullptr},
3954:   {(char*)"_saved_self", (getter)THPSparseSoftmaxBackward0_self_getter, nullptr, nullptr, nullptr},
3955:   {(char*)"_raw_saved_self", (getter)THPSparseSoftmaxBackward0_self_raw_getter, nullptr, nullptr, nullptr},
3956:   {(char*)"_saved_result", (getter)THPSparseSoftmaxBackward0_result_getter, nullptr, nullptr, nullptr},
3957:   {(char*)"_raw_saved_result", (getter)THPSparseSoftmaxBackward0_result_raw_getter, nullptr, nullptr, nullptr},
3958:   {nullptr} /* sentinel */
3959: };
3960: 
3961: static PyObject* THPSparseSparseMatmulBackward0_other_getter(THPCppFunction *self, void *_unused) {
3962:   HANDLE_TH_ERRORS
3963:   const auto& prop = static_cast<SparseSparseMatmulBackward0*>(self->cdata.get())->other_;
3964:   return THPVariable_Wrap(prop.unpack(self->cdata));
3965:   END_HANDLE_TH_ERRORS
3966: }
3967: 
3968: static PyObject* THPSparseSparseMatmulBackward0_other_raw_getter(THPCppFunction *self, void *_unused) {
3969:   HANDLE_TH_ERRORS
3970:   const auto& prop = static_cast<SparseSparseMatmulBackward0*>(self->cdata.get())->other_;
3971:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3972:   return obj.release().ptr();
3973:   END_HANDLE_TH_ERRORS
3974: }
3975: 
3976: static PyObject* THPSparseSparseMatmulBackward0_self_getter(THPCppFunction *self, void *_unused) {
3977:   HANDLE_TH_ERRORS
3978:   const auto& prop = static_cast<SparseSparseMatmulBackward0*>(self->cdata.get())->self_;
3979:   return THPVariable_Wrap(prop.unpack(self->cdata));
3980:   END_HANDLE_TH_ERRORS
3981: }
3982: 
3983: static PyObject* THPSparseSparseMatmulBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
3984:   HANDLE_TH_ERRORS
3985:   const auto& prop = static_cast<SparseSparseMatmulBackward0*>(self->cdata.get())->self_;
3986:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3987:   return obj.release().ptr();
3988:   END_HANDLE_TH_ERRORS
3989: }
3990: 
3991: static struct PyGetSetDef SparseSparseMatmulBackward0_properties[] = {
3992:   THP_FUNCTION_DEFAULT_PROPERTIES,
3993:   {(char*)"_saved_other", (getter)THPSparseSparseMatmulBackward0_other_getter, nullptr, nullptr, nullptr},
3994:   {(char*)"_raw_saved_other", (getter)THPSparseSparseMatmulBackward0_other_raw_getter, nullptr, nullptr, nullptr},
3995:   {(char*)"_saved_self", (getter)THPSparseSparseMatmulBackward0_self_getter, nullptr, nullptr, nullptr},
3996:   {(char*)"_raw_saved_self", (getter)THPSparseSparseMatmulBackward0_self_raw_getter, nullptr, nullptr, nullptr},
3997:   {nullptr} /* sentinel */
3998: };
3999: 
4000: static PyObject* THPThresholdBackward1_self_getter(THPCppFunction *self, void *_unused) {
```

- EN: The main execution path in this span is carried by `THPSparseSoftmaxBackward0_self_getter`, `THPVariable_Wrap`, `THPSparseSoftmaxBackward0_self_raw_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPSparseSoftmaxBackward0_self_getter`, `THPVariable_Wrap`, `THPSparseSoftmaxBackward0_self_raw_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4001-4080

```cpp
4001:   HANDLE_TH_ERRORS
4002:   const auto& prop = static_cast<ThresholdBackward1*>(self->cdata.get())->self_;
4003:   return THPVariable_Wrap(prop.unpack(self->cdata));
4004:   END_HANDLE_TH_ERRORS
4005: }
4006: 
4007: static PyObject* THPThresholdBackward1_self_raw_getter(THPCppFunction *self, void *_unused) {
4008:   HANDLE_TH_ERRORS
4009:   const auto& prop = static_cast<ThresholdBackward1*>(self->cdata.get())->self_;
4010:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4011:   return obj.release().ptr();
4012:   END_HANDLE_TH_ERRORS
4013: }
4014: 
4015: static PyObject* THPThresholdBackward1_threshold_getter(THPCppFunction *self, void *_unused) {
4016:   HANDLE_TH_ERRORS
4017:   auto prop = static_cast<ThresholdBackward1*>(self->cdata.get())->threshold;
4018:   if (prop.isComplex()) {
4019:     auto cprop = prop.to<c10::complex<double>>();
4020:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
4021:   } else if (prop.isFloatingPoint()) {
4022:     return PyFloat_FromDouble(prop.to<double>());
4023:   } else if (prop.isIntegral(/*includeBool=*/false)) {
4024:     return PyLong_FromLong(prop.to<int64_t>());
4025:   } else if (prop.isBoolean()) {
4026:     if (prop.to<bool>()) {
4027:       Py_RETURN_TRUE;
4028:     } else {
4029:       Py_RETURN_FALSE;
4030:     }
4031:   } else {
4032:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
4033:     return nullptr;
4034:   }
4035:   END_HANDLE_TH_ERRORS
4036: }
4037: 
4038: static struct PyGetSetDef ThresholdBackward1_properties[] = {
4039:   THP_FUNCTION_DEFAULT_PROPERTIES,
4040:   {(char*)"_saved_self", (getter)THPThresholdBackward1_self_getter, nullptr, nullptr, nullptr},
4041:   {(char*)"_raw_saved_self", (getter)THPThresholdBackward1_self_raw_getter, nullptr, nullptr, nullptr},
4042:   {(char*)"_saved_threshold", (getter)THPThresholdBackward1_threshold_getter, nullptr, nullptr, nullptr},
4043:   {nullptr} /* sentinel */
4044: };
4045: 
4046: static PyObject* THPUpsampleNearestExact2DBackward0_output_size_getter(THPCppFunction *self, void *_unused) {
4047:   HANDLE_TH_ERRORS
4048:   auto prop = static_cast<UpsampleNearestExact2DBackward0*>(self->cdata.get())->output_size;
4049:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4050:   for (auto i : c10::irange(prop.size())) {
4051:       auto si = prop[i];
4052:       if (auto m = si.maybe_as_int()) {
4053:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4054:       } else {
4055:         auto py_symint = py::cast(si).release().ptr();
4056:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4057:       }
4058:   }
4059:   return tup;
4060:   END_HANDLE_TH_ERRORS
4061: }
4062: 
4063: static PyObject* THPUpsampleNearestExact2DBackward0_scales_h_getter(THPCppFunction *self, void *_unused) {
4064:   HANDLE_TH_ERRORS
4065:   auto opt_prop = static_cast<UpsampleNearestExact2DBackward0*>(self->cdata.get())->scales_h;
4066:   if (!opt_prop.has_value()) {
4067:     Py_RETURN_NONE;
4068:   }
4069:   auto prop = opt_prop.value();
4070:   return PyFloat_FromDouble((double) prop);
4071:   END_HANDLE_TH_ERRORS
4072: }
4073: 
4074: static PyObject* THPUpsampleNearestExact2DBackward0_scales_w_getter(THPCppFunction *self, void *_unused) {
4075:   HANDLE_TH_ERRORS
4076:   auto opt_prop = static_cast<UpsampleNearestExact2DBackward0*>(self->cdata.get())->scales_w;
4077:   if (!opt_prop.has_value()) {
4078:     Py_RETURN_NONE;
4079:   }
4080:   auto prop = opt_prop.value();
```

- EN: The main execution path in this span is carried by `THPVariable_Wrap`, `THPThresholdBackward1_self_raw_getter`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_Wrap`, `THPThresholdBackward1_self_raw_getter`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4081-4160

```cpp
4081:   return PyFloat_FromDouble((double) prop);
4082:   END_HANDLE_TH_ERRORS
4083: }
4084: 
4085: static PyObject* THPUpsampleNearestExact2DBackward0_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
4086:   HANDLE_TH_ERRORS
4087:   auto prop = static_cast<UpsampleNearestExact2DBackward0*>(self->cdata.get())->self_sym_sizes;
4088:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4089:   for (auto i : c10::irange(prop.size())) {
4090:       auto si = prop[i];
4091:       if (auto m = si.maybe_as_int()) {
4092:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4093:       } else {
4094:         auto py_symint = py::cast(si).release().ptr();
4095:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4096:       }
4097:   }
4098:   return tup;
4099:   END_HANDLE_TH_ERRORS
4100: }
4101: 
4102: static struct PyGetSetDef UpsampleNearestExact2DBackward0_properties[] = {
4103:   THP_FUNCTION_DEFAULT_PROPERTIES,
4104:   {(char*)"_saved_output_size", (getter)THPUpsampleNearestExact2DBackward0_output_size_getter, nullptr, nullptr, nullptr},
4105:   {(char*)"_saved_scales_h", (getter)THPUpsampleNearestExact2DBackward0_scales_h_getter, nullptr, nullptr, nullptr},
4106:   {(char*)"_saved_scales_w", (getter)THPUpsampleNearestExact2DBackward0_scales_w_getter, nullptr, nullptr, nullptr},
4107:   {(char*)"_saved_self_sym_sizes", (getter)THPUpsampleNearestExact2DBackward0_self_sym_sizes_getter, nullptr, nullptr, nullptr},
4108:   {nullptr} /* sentinel */
4109: };
4110: 
4111: static PyObject* THPUpsampleNearestExact3DBackward0_output_size_getter(THPCppFunction *self, void *_unused) {
4112:   HANDLE_TH_ERRORS
4113:   auto prop = static_cast<UpsampleNearestExact3DBackward0*>(self->cdata.get())->output_size;
4114:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4115:   for (auto i : c10::irange(prop.size())) {
4116:       auto si = prop[i];
4117:       if (auto m = si.maybe_as_int()) {
4118:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4119:       } else {
4120:         auto py_symint = py::cast(si).release().ptr();
4121:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4122:       }
4123:   }
4124:   return tup;
4125:   END_HANDLE_TH_ERRORS
4126: }
4127: 
4128: static PyObject* THPUpsampleNearestExact3DBackward0_scales_d_getter(THPCppFunction *self, void *_unused) {
4129:   HANDLE_TH_ERRORS
4130:   auto opt_prop = static_cast<UpsampleNearestExact3DBackward0*>(self->cdata.get())->scales_d;
4131:   if (!opt_prop.has_value()) {
4132:     Py_RETURN_NONE;
4133:   }
4134:   auto prop = opt_prop.value();
4135:   return PyFloat_FromDouble((double) prop);
4136:   END_HANDLE_TH_ERRORS
4137: }
4138: 
4139: static PyObject* THPUpsampleNearestExact3DBackward0_scales_h_getter(THPCppFunction *self, void *_unused) {
4140:   HANDLE_TH_ERRORS
4141:   auto opt_prop = static_cast<UpsampleNearestExact3DBackward0*>(self->cdata.get())->scales_h;
4142:   if (!opt_prop.has_value()) {
4143:     Py_RETURN_NONE;
4144:   }
4145:   auto prop = opt_prop.value();
4146:   return PyFloat_FromDouble((double) prop);
4147:   END_HANDLE_TH_ERRORS
4148: }
4149: 
4150: static PyObject* THPUpsampleNearestExact3DBackward0_scales_w_getter(THPCppFunction *self, void *_unused) {
4151:   HANDLE_TH_ERRORS
4152:   auto opt_prop = static_cast<UpsampleNearestExact3DBackward0*>(self->cdata.get())->scales_w;
4153:   if (!opt_prop.has_value()) {
4154:     Py_RETURN_NONE;
4155:   }
4156:   auto prop = opt_prop.value();
4157:   return PyFloat_FromDouble((double) prop);
4158:   END_HANDLE_TH_ERRORS
4159: }
4160: 
```

- EN: The main execution path in this span is carried by `PyFloat_FromDouble`, `THPUpsampleNearestExact2DBackward0_self_sym_sizes_getter`, `PyTuple_New`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyFloat_FromDouble`, `THPUpsampleNearestExact2DBackward0_self_sym_sizes_getter`, `PyTuple_New` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4161-4240

```cpp
4161: static PyObject* THPUpsampleNearestExact3DBackward0_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
4162:   HANDLE_TH_ERRORS
4163:   auto prop = static_cast<UpsampleNearestExact3DBackward0*>(self->cdata.get())->self_sym_sizes;
4164:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4165:   for (auto i : c10::irange(prop.size())) {
4166:       auto si = prop[i];
4167:       if (auto m = si.maybe_as_int()) {
4168:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4169:       } else {
4170:         auto py_symint = py::cast(si).release().ptr();
4171:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4172:       }
4173:   }
4174:   return tup;
4175:   END_HANDLE_TH_ERRORS
4176: }
4177: 
4178: static struct PyGetSetDef UpsampleNearestExact3DBackward0_properties[] = {
4179:   THP_FUNCTION_DEFAULT_PROPERTIES,
4180:   {(char*)"_saved_output_size", (getter)THPUpsampleNearestExact3DBackward0_output_size_getter, nullptr, nullptr, nullptr},
4181:   {(char*)"_saved_scales_d", (getter)THPUpsampleNearestExact3DBackward0_scales_d_getter, nullptr, nullptr, nullptr},
4182:   {(char*)"_saved_scales_h", (getter)THPUpsampleNearestExact3DBackward0_scales_h_getter, nullptr, nullptr, nullptr},
4183:   {(char*)"_saved_scales_w", (getter)THPUpsampleNearestExact3DBackward0_scales_w_getter, nullptr, nullptr, nullptr},
4184:   {(char*)"_saved_self_sym_sizes", (getter)THPUpsampleNearestExact3DBackward0_self_sym_sizes_getter, nullptr, nullptr, nullptr},
4185:   {nullptr} /* sentinel */
4186: };
4187: 
4188: static PyObject* THPAvgPool2DBackward0_ceil_mode_getter(THPCppFunction *self, void *_unused) {
4189:   HANDLE_TH_ERRORS
4190:   auto prop = static_cast<AvgPool2DBackward0*>(self->cdata.get())->ceil_mode;
4191:   if (prop) {
4192:     Py_RETURN_TRUE;
4193:   } else {
4194:     Py_RETURN_FALSE;
4195:   }
4196:   END_HANDLE_TH_ERRORS
4197: }
4198: 
4199: static PyObject* THPAvgPool2DBackward0_count_include_pad_getter(THPCppFunction *self, void *_unused) {
4200:   HANDLE_TH_ERRORS
4201:   auto prop = static_cast<AvgPool2DBackward0*>(self->cdata.get())->count_include_pad;
4202:   if (prop) {
4203:     Py_RETURN_TRUE;
4204:   } else {
4205:     Py_RETURN_FALSE;
4206:   }
4207:   END_HANDLE_TH_ERRORS
4208: }
4209: 
4210: static PyObject* THPAvgPool2DBackward0_divisor_override_getter(THPCppFunction *self, void *_unused) {
4211:   HANDLE_TH_ERRORS
4212:   auto opt_prop = static_cast<AvgPool2DBackward0*>(self->cdata.get())->divisor_override;
4213:   if (!opt_prop.has_value()) {
4214:     Py_RETURN_NONE;
4215:   }
4216:   auto prop = opt_prop.value();
4217:   return PyLong_FromUnsignedLong((int64_t) prop);
4218:   END_HANDLE_TH_ERRORS
4219: }
4220: 
4221: static PyObject* THPAvgPool2DBackward0_kernel_size_getter(THPCppFunction *self, void *_unused) {
4222:   HANDLE_TH_ERRORS
4223:   auto prop = static_cast<AvgPool2DBackward0*>(self->cdata.get())->kernel_size;
4224:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4225:   for (auto i : c10::irange(prop.size())) {
4226:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
4227:   }
4228:   return tup;
4229:   END_HANDLE_TH_ERRORS
4230: }
4231: 
4232: static PyObject* THPAvgPool2DBackward0_padding_getter(THPCppFunction *self, void *_unused) {
4233:   HANDLE_TH_ERRORS
4234:   auto prop = static_cast<AvgPool2DBackward0*>(self->cdata.get())->padding;
4235:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4236:   for (auto i : c10::irange(prop.size())) {
4237:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
4238:   }
4239:   return tup;
4240:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPUpsampleNearestExact3DBackward0_self_sym_sizes_getter`, `PyTuple_New`, `PyTuple_SetItem`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPUpsampleNearestExact3DBackward0_self_sym_sizes_getter`, `PyTuple_New`, `PyTuple_SetItem` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 4241-4320

```cpp
4241: }
4242: 
4243: static PyObject* THPAvgPool2DBackward0_self_getter(THPCppFunction *self, void *_unused) {
4244:   HANDLE_TH_ERRORS
4245:   const auto& prop = static_cast<AvgPool2DBackward0*>(self->cdata.get())->self_;
4246:   return THPVariable_Wrap(prop.unpack(self->cdata));
4247:   END_HANDLE_TH_ERRORS
4248: }
4249: 
4250: static PyObject* THPAvgPool2DBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
4251:   HANDLE_TH_ERRORS
4252:   const auto& prop = static_cast<AvgPool2DBackward0*>(self->cdata.get())->self_;
4253:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4254:   return obj.release().ptr();
4255:   END_HANDLE_TH_ERRORS
4256: }
4257: 
4258: static PyObject* THPAvgPool2DBackward0_stride_getter(THPCppFunction *self, void *_unused) {
4259:   HANDLE_TH_ERRORS
4260:   auto prop = static_cast<AvgPool2DBackward0*>(self->cdata.get())->stride;
4261:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4262:   for (auto i : c10::irange(prop.size())) {
4263:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
4264:   }
4265:   return tup;
4266:   END_HANDLE_TH_ERRORS
4267: }
4268: 
4269: static struct PyGetSetDef AvgPool2DBackward0_properties[] = {
4270:   THP_FUNCTION_DEFAULT_PROPERTIES,
4271:   {(char*)"_saved_ceil_mode", (getter)THPAvgPool2DBackward0_ceil_mode_getter, nullptr, nullptr, nullptr},
4272:   {(char*)"_saved_count_include_pad", (getter)THPAvgPool2DBackward0_count_include_pad_getter, nullptr, nullptr, nullptr},
4273:   {(char*)"_saved_divisor_override", (getter)THPAvgPool2DBackward0_divisor_override_getter, nullptr, nullptr, nullptr},
4274:   {(char*)"_saved_kernel_size", (getter)THPAvgPool2DBackward0_kernel_size_getter, nullptr, nullptr, nullptr},
4275:   {(char*)"_saved_padding", (getter)THPAvgPool2DBackward0_padding_getter, nullptr, nullptr, nullptr},
4276:   {(char*)"_saved_self", (getter)THPAvgPool2DBackward0_self_getter, nullptr, nullptr, nullptr},
4277:   {(char*)"_raw_saved_self", (getter)THPAvgPool2DBackward0_self_raw_getter, nullptr, nullptr, nullptr},
4278:   {(char*)"_saved_stride", (getter)THPAvgPool2DBackward0_stride_getter, nullptr, nullptr, nullptr},
4279:   {nullptr} /* sentinel */
4280: };
4281: 
4282: static PyObject* THPMpsConvolutionBackwardBackward0_dilation_getter(THPCppFunction *self, void *_unused) {
4283:   HANDLE_TH_ERRORS
4284:   auto prop = static_cast<MpsConvolutionBackwardBackward0*>(self->cdata.get())->dilation;
4285:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4286:   for (auto i : c10::irange(prop.size())) {
4287:       auto si = prop[i];
4288:       if (auto m = si.maybe_as_int()) {
4289:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4290:       } else {
4291:         auto py_symint = py::cast(si).release().ptr();
4292:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4293:       }
4294:   }
4295:   return tup;
4296:   END_HANDLE_TH_ERRORS
4297: }
4298: 
4299: static PyObject* THPMpsConvolutionBackwardBackward0_grad_output_getter(THPCppFunction *self, void *_unused) {
4300:   HANDLE_TH_ERRORS
4301:   const auto& prop = static_cast<MpsConvolutionBackwardBackward0*>(self->cdata.get())->grad_output_;
4302:   return THPVariable_Wrap(prop.unpack(self->cdata));
4303:   END_HANDLE_TH_ERRORS
4304: }
4305: 
4306: static PyObject* THPMpsConvolutionBackwardBackward0_grad_output_raw_getter(THPCppFunction *self, void *_unused) {
4307:   HANDLE_TH_ERRORS
4308:   const auto& prop = static_cast<MpsConvolutionBackwardBackward0*>(self->cdata.get())->grad_output_;
4309:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4310:   return obj.release().ptr();
4311:   END_HANDLE_TH_ERRORS
4312: }
4313: 
4314: static PyObject* THPMpsConvolutionBackwardBackward0_groups_getter(THPCppFunction *self, void *_unused) {
4315:   HANDLE_TH_ERRORS
4316:   auto prop = static_cast<MpsConvolutionBackwardBackward0*>(self->cdata.get())->groups;
4317:   if (auto m = prop.maybe_as_int()) {
4318:     return PyLong_FromUnsignedLong(*m);
4319:   } else {
4320:     return py::cast(prop).release().ptr();
```

- EN: The main execution path in this span is carried by `THPAvgPool2DBackward0_self_getter`, `THPVariable_Wrap`, `THPAvgPool2DBackward0_self_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPAvgPool2DBackward0_self_getter`, `THPVariable_Wrap`, `THPAvgPool2DBackward0_self_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 4321-4400

```cpp
4321:   }
4322:   END_HANDLE_TH_ERRORS
4323: }
4324: 
4325: static PyObject* THPMpsConvolutionBackwardBackward0_padding_getter(THPCppFunction *self, void *_unused) {
4326:   HANDLE_TH_ERRORS
4327:   auto prop = static_cast<MpsConvolutionBackwardBackward0*>(self->cdata.get())->padding;
4328:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4329:   for (auto i : c10::irange(prop.size())) {
4330:       auto si = prop[i];
4331:       if (auto m = si.maybe_as_int()) {
4332:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4333:       } else {
4334:         auto py_symint = py::cast(si).release().ptr();
4335:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4336:       }
4337:   }
4338:   return tup;
4339:   END_HANDLE_TH_ERRORS
4340: }
4341: 
4342: static PyObject* THPMpsConvolutionBackwardBackward0_self_getter(THPCppFunction *self, void *_unused) {
4343:   HANDLE_TH_ERRORS
4344:   const auto& prop = static_cast<MpsConvolutionBackwardBackward0*>(self->cdata.get())->self_;
4345:   return THPVariable_Wrap(prop.unpack(self->cdata));
4346:   END_HANDLE_TH_ERRORS
4347: }
4348: 
4349: static PyObject* THPMpsConvolutionBackwardBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
4350:   HANDLE_TH_ERRORS
4351:   const auto& prop = static_cast<MpsConvolutionBackwardBackward0*>(self->cdata.get())->self_;
4352:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4353:   return obj.release().ptr();
4354:   END_HANDLE_TH_ERRORS
4355: }
4356: 
4357: static PyObject* THPMpsConvolutionBackwardBackward0_stride_getter(THPCppFunction *self, void *_unused) {
4358:   HANDLE_TH_ERRORS
4359:   auto prop = static_cast<MpsConvolutionBackwardBackward0*>(self->cdata.get())->stride;
4360:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4361:   for (auto i : c10::irange(prop.size())) {
4362:       auto si = prop[i];
4363:       if (auto m = si.maybe_as_int()) {
4364:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4365:       } else {
4366:         auto py_symint = py::cast(si).release().ptr();
4367:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4368:       }
4369:   }
4370:   return tup;
4371:   END_HANDLE_TH_ERRORS
4372: }
4373: 
4374: static PyObject* THPMpsConvolutionBackwardBackward0_weight_getter(THPCppFunction *self, void *_unused) {
4375:   HANDLE_TH_ERRORS
4376:   const auto& prop = static_cast<MpsConvolutionBackwardBackward0*>(self->cdata.get())->weight_;
4377:   return THPVariable_Wrap(prop.unpack(self->cdata));
4378:   END_HANDLE_TH_ERRORS
4379: }
4380: 
4381: static PyObject* THPMpsConvolutionBackwardBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
4382:   HANDLE_TH_ERRORS
4383:   const auto& prop = static_cast<MpsConvolutionBackwardBackward0*>(self->cdata.get())->weight_;
4384:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4385:   return obj.release().ptr();
4386:   END_HANDLE_TH_ERRORS
4387: }
4388: 
4389: static struct PyGetSetDef MpsConvolutionBackwardBackward0_properties[] = {
4390:   THP_FUNCTION_DEFAULT_PROPERTIES,
4391:   {(char*)"_saved_dilation", (getter)THPMpsConvolutionBackwardBackward0_dilation_getter, nullptr, nullptr, nullptr},
4392:   {(char*)"_saved_grad_output", (getter)THPMpsConvolutionBackwardBackward0_grad_output_getter, nullptr, nullptr, nullptr},
4393:   {(char*)"_raw_saved_grad_output", (getter)THPMpsConvolutionBackwardBackward0_grad_output_raw_getter, nullptr, nullptr, nullptr},
4394:   {(char*)"_saved_groups", (getter)THPMpsConvolutionBackwardBackward0_groups_getter, nullptr, nullptr, nullptr},
4395:   {(char*)"_saved_padding", (getter)THPMpsConvolutionBackwardBackward0_padding_getter, nullptr, nullptr, nullptr},
4396:   {(char*)"_saved_self", (getter)THPMpsConvolutionBackwardBackward0_self_getter, nullptr, nullptr, nullptr},
4397:   {(char*)"_raw_saved_self", (getter)THPMpsConvolutionBackwardBackward0_self_raw_getter, nullptr, nullptr, nullptr},
4398:   {(char*)"_saved_stride", (getter)THPMpsConvolutionBackwardBackward0_stride_getter, nullptr, nullptr, nullptr},
4399:   {(char*)"_saved_weight", (getter)THPMpsConvolutionBackwardBackward0_weight_getter, nullptr, nullptr, nullptr},
4400:   {(char*)"_raw_saved_weight", (getter)THPMpsConvolutionBackwardBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
```

- EN: The main execution path in this span is carried by `THPMpsConvolutionBackwardBackward0_padding_getter`, `PyTuple_New`, `PyTuple_SetItem`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPMpsConvolutionBackwardBackward0_padding_getter`, `PyTuple_New`, `PyTuple_SetItem` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4401-4480

```cpp
4401:   {nullptr} /* sentinel */
4402: };
4403: 
4404: static PyObject* THPSlowConvDilated3DBackward0_bias_sym_sizes_opt_getter(THPCppFunction *self, void *_unused) {
4405:   HANDLE_TH_ERRORS
4406:   auto opt_prop = static_cast<SlowConvDilated3DBackward0*>(self->cdata.get())->bias_sym_sizes_opt;
4407:   if (!opt_prop.list.has_value()) {
4408:     Py_RETURN_NONE;
4409:   }
4410:   auto prop = opt_prop.list.value();
4411:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4412:   for (auto i : c10::irange(prop.size())) {
4413:       auto si = prop[i];
4414:       if (auto m = si.maybe_as_int()) {
4415:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4416:       } else {
4417:         auto py_symint = py::cast(si).release().ptr();
4418:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4419:       }
4420:   }
4421:   return tup;
4422:   END_HANDLE_TH_ERRORS
4423: }
4424: 
4425: static PyObject* THPSlowConvDilated3DBackward0_dilation_getter(THPCppFunction *self, void *_unused) {
4426:   HANDLE_TH_ERRORS
4427:   auto prop = static_cast<SlowConvDilated3DBackward0*>(self->cdata.get())->dilation;
4428:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4429:   for (auto i : c10::irange(prop.size())) {
4430:       auto si = prop[i];
4431:       if (auto m = si.maybe_as_int()) {
4432:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4433:       } else {
4434:         auto py_symint = py::cast(si).release().ptr();
4435:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4436:       }
4437:   }
4438:   return tup;
4439:   END_HANDLE_TH_ERRORS
4440: }
4441: 
4442: static PyObject* THPSlowConvDilated3DBackward0_padding_getter(THPCppFunction *self, void *_unused) {
4443:   HANDLE_TH_ERRORS
4444:   auto prop = static_cast<SlowConvDilated3DBackward0*>(self->cdata.get())->padding;
4445:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4446:   for (auto i : c10::irange(prop.size())) {
4447:       auto si = prop[i];
4448:       if (auto m = si.maybe_as_int()) {
4449:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4450:       } else {
4451:         auto py_symint = py::cast(si).release().ptr();
4452:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4453:       }
4454:   }
4455:   return tup;
4456:   END_HANDLE_TH_ERRORS
4457: }
4458: 
4459: static PyObject* THPSlowConvDilated3DBackward0_self_getter(THPCppFunction *self, void *_unused) {
4460:   HANDLE_TH_ERRORS
4461:   const auto& prop = static_cast<SlowConvDilated3DBackward0*>(self->cdata.get())->self_;
4462:   return THPVariable_Wrap(prop.unpack(self->cdata));
4463:   END_HANDLE_TH_ERRORS
4464: }
4465: 
4466: static PyObject* THPSlowConvDilated3DBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
4467:   HANDLE_TH_ERRORS
4468:   const auto& prop = static_cast<SlowConvDilated3DBackward0*>(self->cdata.get())->self_;
4469:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4470:   return obj.release().ptr();
4471:   END_HANDLE_TH_ERRORS
4472: }
4473: 
4474: static PyObject* THPSlowConvDilated3DBackward0_stride_getter(THPCppFunction *self, void *_unused) {
4475:   HANDLE_TH_ERRORS
4476:   auto prop = static_cast<SlowConvDilated3DBackward0*>(self->cdata.get())->stride;
4477:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4478:   for (auto i : c10::irange(prop.size())) {
4479:       auto si = prop[i];
4480:       if (auto m = si.maybe_as_int()) {
```

- EN: The main execution path in this span is carried by `THPSlowConvDilated3DBackward0_bias_sym_sizes_opt_getter`, `PyTuple_New`, `PyTuple_SetItem`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPSlowConvDilated3DBackward0_bias_sym_sizes_opt_getter`, `PyTuple_New`, `PyTuple_SetItem` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4481-4560

```cpp
4481:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4482:       } else {
4483:         auto py_symint = py::cast(si).release().ptr();
4484:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4485:       }
4486:   }
4487:   return tup;
4488:   END_HANDLE_TH_ERRORS
4489: }
4490: 
4491: static PyObject* THPSlowConvDilated3DBackward0_weight_getter(THPCppFunction *self, void *_unused) {
4492:   HANDLE_TH_ERRORS
4493:   const auto& prop = static_cast<SlowConvDilated3DBackward0*>(self->cdata.get())->weight_;
4494:   return THPVariable_Wrap(prop.unpack(self->cdata));
4495:   END_HANDLE_TH_ERRORS
4496: }
4497: 
4498: static PyObject* THPSlowConvDilated3DBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
4499:   HANDLE_TH_ERRORS
4500:   const auto& prop = static_cast<SlowConvDilated3DBackward0*>(self->cdata.get())->weight_;
4501:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4502:   return obj.release().ptr();
4503:   END_HANDLE_TH_ERRORS
4504: }
4505: 
4506: static struct PyGetSetDef SlowConvDilated3DBackward0_properties[] = {
4507:   THP_FUNCTION_DEFAULT_PROPERTIES,
4508:   {(char*)"_saved_bias_sym_sizes_opt", (getter)THPSlowConvDilated3DBackward0_bias_sym_sizes_opt_getter, nullptr, nullptr, nullptr},
4509:   {(char*)"_saved_dilation", (getter)THPSlowConvDilated3DBackward0_dilation_getter, nullptr, nullptr, nullptr},
4510:   {(char*)"_saved_padding", (getter)THPSlowConvDilated3DBackward0_padding_getter, nullptr, nullptr, nullptr},
4511:   {(char*)"_saved_self", (getter)THPSlowConvDilated3DBackward0_self_getter, nullptr, nullptr, nullptr},
4512:   {(char*)"_raw_saved_self", (getter)THPSlowConvDilated3DBackward0_self_raw_getter, nullptr, nullptr, nullptr},
4513:   {(char*)"_saved_stride", (getter)THPSlowConvDilated3DBackward0_stride_getter, nullptr, nullptr, nullptr},
4514:   {(char*)"_saved_weight", (getter)THPSlowConvDilated3DBackward0_weight_getter, nullptr, nullptr, nullptr},
4515:   {(char*)"_raw_saved_weight", (getter)THPSlowConvDilated3DBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
4516:   {nullptr} /* sentinel */
4517: };
4518: 
4519: static PyObject* THPAdaptiveAvgPool2DBackwardBackward0_grad_output_sym_argsize_minus_1_getter(THPCppFunction *self, void *_unused) {
4520:   HANDLE_TH_ERRORS
4521:   auto prop = static_cast<AdaptiveAvgPool2DBackwardBackward0*>(self->cdata.get())->grad_output_sym_argsize_minus_1;
4522:   if (auto m = prop.maybe_as_int()) {
4523:     return PyLong_FromUnsignedLong(*m);
4524:   } else {
4525:     return py::cast(prop).release().ptr();
4526:   }
4527:   END_HANDLE_TH_ERRORS
4528: }
4529: 
4530: static PyObject* THPAdaptiveAvgPool2DBackwardBackward0_grad_output_sym_argsize_minus_2_getter(THPCppFunction *self, void *_unused) {
4531:   HANDLE_TH_ERRORS
4532:   auto prop = static_cast<AdaptiveAvgPool2DBackwardBackward0*>(self->cdata.get())->grad_output_sym_argsize_minus_2;
4533:   if (auto m = prop.maybe_as_int()) {
4534:     return PyLong_FromUnsignedLong(*m);
4535:   } else {
4536:     return py::cast(prop).release().ptr();
4537:   }
4538:   END_HANDLE_TH_ERRORS
4539: }
4540: 
4541: static struct PyGetSetDef AdaptiveAvgPool2DBackwardBackward0_properties[] = {
4542:   THP_FUNCTION_DEFAULT_PROPERTIES,
4543:   {(char*)"_saved_grad_output_sym_argsize_minus_1", (getter)THPAdaptiveAvgPool2DBackwardBackward0_grad_output_sym_argsize_minus_1_getter, nullptr, nullptr, nullptr},
4544:   {(char*)"_saved_grad_output_sym_argsize_minus_2", (getter)THPAdaptiveAvgPool2DBackwardBackward0_grad_output_sym_argsize_minus_2_getter, nullptr, nullptr, nullptr},
4545:   {nullptr} /* sentinel */
4546: };
4547: 
4548: static PyObject* THPFractionalMaxPool2DBackwardBackward0_indices_getter(THPCppFunction *self, void *_unused) {
4549:   HANDLE_TH_ERRORS
4550:   const auto& prop = static_cast<FractionalMaxPool2DBackwardBackward0*>(self->cdata.get())->indices_;
4551:   return THPVariable_Wrap(prop.unpack(self->cdata));
4552:   END_HANDLE_TH_ERRORS
4553: }
4554: 
4555: static PyObject* THPFractionalMaxPool2DBackwardBackward0_indices_raw_getter(THPCppFunction *self, void *_unused) {
4556:   HANDLE_TH_ERRORS
4557:   const auto& prop = static_cast<FractionalMaxPool2DBackwardBackward0*>(self->cdata.get())->indices_;
4558:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4559:   return obj.release().ptr();
4560:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `PyTuple_SetItem`, `cast`, `THPSlowConvDilated3DBackward0_weight_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyTuple_SetItem`, `cast`, `THPSlowConvDilated3DBackward0_weight_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4561-4640

```cpp
4561: }
4562: 
4563: static struct PyGetSetDef FractionalMaxPool2DBackwardBackward0_properties[] = {
4564:   THP_FUNCTION_DEFAULT_PROPERTIES,
4565:   {(char*)"_saved_indices", (getter)THPFractionalMaxPool2DBackwardBackward0_indices_getter, nullptr, nullptr, nullptr},
4566:   {(char*)"_raw_saved_indices", (getter)THPFractionalMaxPool2DBackwardBackward0_indices_raw_getter, nullptr, nullptr, nullptr},
4567:   {nullptr} /* sentinel */
4568: };
4569: 
4570: static PyObject* THPMaxPool3DWithIndicesBackwardBackward0_indices_getter(THPCppFunction *self, void *_unused) {
4571:   HANDLE_TH_ERRORS
4572:   const auto& prop = static_cast<MaxPool3DWithIndicesBackwardBackward0*>(self->cdata.get())->indices_;
4573:   return THPVariable_Wrap(prop.unpack(self->cdata));
4574:   END_HANDLE_TH_ERRORS
4575: }
4576: 
4577: static PyObject* THPMaxPool3DWithIndicesBackwardBackward0_indices_raw_getter(THPCppFunction *self, void *_unused) {
4578:   HANDLE_TH_ERRORS
4579:   const auto& prop = static_cast<MaxPool3DWithIndicesBackwardBackward0*>(self->cdata.get())->indices_;
4580:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4581:   return obj.release().ptr();
4582:   END_HANDLE_TH_ERRORS
4583: }
4584: 
4585: static struct PyGetSetDef MaxPool3DWithIndicesBackwardBackward0_properties[] = {
4586:   THP_FUNCTION_DEFAULT_PROPERTIES,
4587:   {(char*)"_saved_indices", (getter)THPMaxPool3DWithIndicesBackwardBackward0_indices_getter, nullptr, nullptr, nullptr},
4588:   {(char*)"_raw_saved_indices", (getter)THPMaxPool3DWithIndicesBackwardBackward0_indices_raw_getter, nullptr, nullptr, nullptr},
4589:   {nullptr} /* sentinel */
4590: };
4591: 
4592: static PyObject* THPReflectionPad1DBackwardBackward0_padding_getter(THPCppFunction *self, void *_unused) {
4593:   HANDLE_TH_ERRORS
4594:   auto prop = static_cast<ReflectionPad1DBackwardBackward0*>(self->cdata.get())->padding;
4595:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4596:   for (auto i : c10::irange(prop.size())) {
4597:       auto si = prop[i];
4598:       if (auto m = si.maybe_as_int()) {
4599:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4600:       } else {
4601:         auto py_symint = py::cast(si).release().ptr();
4602:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4603:       }
4604:   }
4605:   return tup;
4606:   END_HANDLE_TH_ERRORS
4607: }
4608: 
4609: static struct PyGetSetDef ReflectionPad1DBackwardBackward0_properties[] = {
4610:   THP_FUNCTION_DEFAULT_PROPERTIES,
4611:   {(char*)"_saved_padding", (getter)THPReflectionPad1DBackwardBackward0_padding_getter, nullptr, nullptr, nullptr},
4612:   {nullptr} /* sentinel */
4613: };
4614: 
4615: static PyObject* THPReflectionPad2DBackwardBackward0_padding_getter(THPCppFunction *self, void *_unused) {
4616:   HANDLE_TH_ERRORS
4617:   auto prop = static_cast<ReflectionPad2DBackwardBackward0*>(self->cdata.get())->padding;
4618:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4619:   for (auto i : c10::irange(prop.size())) {
4620:       auto si = prop[i];
4621:       if (auto m = si.maybe_as_int()) {
4622:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4623:       } else {
4624:         auto py_symint = py::cast(si).release().ptr();
4625:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4626:       }
4627:   }
4628:   return tup;
4629:   END_HANDLE_TH_ERRORS
4630: }
4631: 
4632: static struct PyGetSetDef ReflectionPad2DBackwardBackward0_properties[] = {
4633:   THP_FUNCTION_DEFAULT_PROPERTIES,
4634:   {(char*)"_saved_padding", (getter)THPReflectionPad2DBackwardBackward0_padding_getter, nullptr, nullptr, nullptr},
4635:   {nullptr} /* sentinel */
4636: };
4637: 
4638: static PyObject* THPReplicationPad1DBackwardBackward0_padding_getter(THPCppFunction *self, void *_unused) {
4639:   HANDLE_TH_ERRORS
4640:   auto prop = static_cast<ReplicationPad1DBackwardBackward0*>(self->cdata.get())->padding;
```

- EN: The main execution path in this span is carried by `THPMaxPool3DWithIndicesBackwardBackward0_indices_getter`, `THPVariable_Wrap`, `THPMaxPool3DWithIndicesBackwardBackward0_indices_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPMaxPool3DWithIndicesBackwardBackward0_indices_getter`, `THPVariable_Wrap`, `THPMaxPool3DWithIndicesBackwardBackward0_indices_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4641-4720

```cpp
4641:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4642:   for (auto i : c10::irange(prop.size())) {
4643:       auto si = prop[i];
4644:       if (auto m = si.maybe_as_int()) {
4645:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4646:       } else {
4647:         auto py_symint = py::cast(si).release().ptr();
4648:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4649:       }
4650:   }
4651:   return tup;
4652:   END_HANDLE_TH_ERRORS
4653: }
4654: 
4655: static struct PyGetSetDef ReplicationPad1DBackwardBackward0_properties[] = {
4656:   THP_FUNCTION_DEFAULT_PROPERTIES,
4657:   {(char*)"_saved_padding", (getter)THPReplicationPad1DBackwardBackward0_padding_getter, nullptr, nullptr, nullptr},
4658:   {nullptr} /* sentinel */
4659: };
4660: 
4661: static PyObject* THPReplicationPad3DBackwardBackward0_padding_getter(THPCppFunction *self, void *_unused) {
4662:   HANDLE_TH_ERRORS
4663:   auto prop = static_cast<ReplicationPad3DBackwardBackward0*>(self->cdata.get())->padding;
4664:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4665:   for (auto i : c10::irange(prop.size())) {
4666:       auto si = prop[i];
4667:       if (auto m = si.maybe_as_int()) {
4668:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4669:       } else {
4670:         auto py_symint = py::cast(si).release().ptr();
4671:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4672:       }
4673:   }
4674:   return tup;
4675:   END_HANDLE_TH_ERRORS
4676: }
4677: 
4678: static struct PyGetSetDef ReplicationPad3DBackwardBackward0_properties[] = {
4679:   THP_FUNCTION_DEFAULT_PROPERTIES,
4680:   {(char*)"_saved_padding", (getter)THPReplicationPad3DBackwardBackward0_padding_getter, nullptr, nullptr, nullptr},
4681:   {nullptr} /* sentinel */
4682: };
4683: 
4684: static PyObject* THPSoftMarginLossBackwardBackward0_grad_output_getter(THPCppFunction *self, void *_unused) {
4685:   HANDLE_TH_ERRORS
4686:   const auto& prop = static_cast<SoftMarginLossBackwardBackward0*>(self->cdata.get())->grad_output_;
4687:   return THPVariable_Wrap(prop.unpack(self->cdata));
4688:   END_HANDLE_TH_ERRORS
4689: }
4690: 
4691: static PyObject* THPSoftMarginLossBackwardBackward0_grad_output_raw_getter(THPCppFunction *self, void *_unused) {
4692:   HANDLE_TH_ERRORS
4693:   const auto& prop = static_cast<SoftMarginLossBackwardBackward0*>(self->cdata.get())->grad_output_;
4694:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4695:   return obj.release().ptr();
4696:   END_HANDLE_TH_ERRORS
4697: }
4698: 
4699: static PyObject* THPSoftMarginLossBackwardBackward0_reduction_getter(THPCppFunction *self, void *_unused) {
4700:   HANDLE_TH_ERRORS
4701:   auto prop = static_cast<SoftMarginLossBackwardBackward0*>(self->cdata.get())->reduction;
4702:   return PyLong_FromUnsignedLong((int64_t) prop);
4703:   END_HANDLE_TH_ERRORS
4704: }
4705: 
4706: static PyObject* THPSoftMarginLossBackwardBackward0_self_getter(THPCppFunction *self, void *_unused) {
4707:   HANDLE_TH_ERRORS
4708:   const auto& prop = static_cast<SoftMarginLossBackwardBackward0*>(self->cdata.get())->self_;
4709:   return THPVariable_Wrap(prop.unpack(self->cdata));
4710:   END_HANDLE_TH_ERRORS
4711: }
4712: 
4713: static PyObject* THPSoftMarginLossBackwardBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
4714:   HANDLE_TH_ERRORS
4715:   const auto& prop = static_cast<SoftMarginLossBackwardBackward0*>(self->cdata.get())->self_;
4716:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4717:   return obj.release().ptr();
4718:   END_HANDLE_TH_ERRORS
4719: }
4720: 
```

- EN: The main execution path in this span is carried by `PyTuple_New`, `PyTuple_SetItem`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyTuple_New`, `PyTuple_SetItem`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4721-4800

```cpp
4721: static PyObject* THPSoftMarginLossBackwardBackward0_target_getter(THPCppFunction *self, void *_unused) {
4722:   HANDLE_TH_ERRORS
4723:   const auto& prop = static_cast<SoftMarginLossBackwardBackward0*>(self->cdata.get())->target_;
4724:   return THPVariable_Wrap(prop.unpack(self->cdata));
4725:   END_HANDLE_TH_ERRORS
4726: }
4727: 
4728: static PyObject* THPSoftMarginLossBackwardBackward0_target_raw_getter(THPCppFunction *self, void *_unused) {
4729:   HANDLE_TH_ERRORS
4730:   const auto& prop = static_cast<SoftMarginLossBackwardBackward0*>(self->cdata.get())->target_;
4731:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4732:   return obj.release().ptr();
4733:   END_HANDLE_TH_ERRORS
4734: }
4735: 
4736: static struct PyGetSetDef SoftMarginLossBackwardBackward0_properties[] = {
4737:   THP_FUNCTION_DEFAULT_PROPERTIES,
4738:   {(char*)"_saved_grad_output", (getter)THPSoftMarginLossBackwardBackward0_grad_output_getter, nullptr, nullptr, nullptr},
4739:   {(char*)"_raw_saved_grad_output", (getter)THPSoftMarginLossBackwardBackward0_grad_output_raw_getter, nullptr, nullptr, nullptr},
4740:   {(char*)"_saved_reduction", (getter)THPSoftMarginLossBackwardBackward0_reduction_getter, nullptr, nullptr, nullptr},
4741:   {(char*)"_saved_self", (getter)THPSoftMarginLossBackwardBackward0_self_getter, nullptr, nullptr, nullptr},
4742:   {(char*)"_raw_saved_self", (getter)THPSoftMarginLossBackwardBackward0_self_raw_getter, nullptr, nullptr, nullptr},
4743:   {(char*)"_saved_target", (getter)THPSoftMarginLossBackwardBackward0_target_getter, nullptr, nullptr, nullptr},
4744:   {(char*)"_raw_saved_target", (getter)THPSoftMarginLossBackwardBackward0_target_raw_getter, nullptr, nullptr, nullptr},
4745:   {nullptr} /* sentinel */
4746: };
4747: 
4748: static PyObject* THPSoftshrinkBackwardBackward0_lambd_getter(THPCppFunction *self, void *_unused) {
4749:   HANDLE_TH_ERRORS
4750:   auto prop = static_cast<SoftshrinkBackwardBackward0*>(self->cdata.get())->lambd;
4751:   if (prop.isComplex()) {
4752:     auto cprop = prop.to<c10::complex<double>>();
4753:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
4754:   } else if (prop.isFloatingPoint()) {
4755:     return PyFloat_FromDouble(prop.to<double>());
4756:   } else if (prop.isIntegral(/*includeBool=*/false)) {
4757:     return PyLong_FromLong(prop.to<int64_t>());
4758:   } else if (prop.isBoolean()) {
4759:     if (prop.to<bool>()) {
4760:       Py_RETURN_TRUE;
4761:     } else {
4762:       Py_RETURN_FALSE;
4763:     }
4764:   } else {
4765:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
4766:     return nullptr;
4767:   }
4768:   END_HANDLE_TH_ERRORS
4769: }
4770: 
4771: static PyObject* THPSoftshrinkBackwardBackward0_self_getter(THPCppFunction *self, void *_unused) {
4772:   HANDLE_TH_ERRORS
4773:   const auto& prop = static_cast<SoftshrinkBackwardBackward0*>(self->cdata.get())->self_;
4774:   return THPVariable_Wrap(prop.unpack(self->cdata));
4775:   END_HANDLE_TH_ERRORS
4776: }
4777: 
4778: static PyObject* THPSoftshrinkBackwardBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
4779:   HANDLE_TH_ERRORS
4780:   const auto& prop = static_cast<SoftshrinkBackwardBackward0*>(self->cdata.get())->self_;
4781:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4782:   return obj.release().ptr();
4783:   END_HANDLE_TH_ERRORS
4784: }
4785: 
4786: static struct PyGetSetDef SoftshrinkBackwardBackward0_properties[] = {
4787:   THP_FUNCTION_DEFAULT_PROPERTIES,
4788:   {(char*)"_saved_lambd", (getter)THPSoftshrinkBackwardBackward0_lambd_getter, nullptr, nullptr, nullptr},
4789:   {(char*)"_saved_self", (getter)THPSoftshrinkBackwardBackward0_self_getter, nullptr, nullptr, nullptr},
4790:   {(char*)"_raw_saved_self", (getter)THPSoftshrinkBackwardBackward0_self_raw_getter, nullptr, nullptr, nullptr},
4791:   {nullptr} /* sentinel */
4792: };
4793: 
4794: static PyObject* THPUpsampleBilinear2DBackwardBackward0_align_corners_getter(THPCppFunction *self, void *_unused) {
4795:   HANDLE_TH_ERRORS
4796:   auto prop = static_cast<UpsampleBilinear2DBackwardBackward0*>(self->cdata.get())->align_corners;
4797:   if (prop) {
4798:     Py_RETURN_TRUE;
4799:   } else {
4800:     Py_RETURN_FALSE;
```

- EN: The main execution path in this span is carried by `THPSoftMarginLossBackwardBackward0_target_getter`, `THPVariable_Wrap`, `THPSoftMarginLossBackwardBackward0_target_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPSoftMarginLossBackwardBackward0_target_getter`, `THPVariable_Wrap`, `THPSoftMarginLossBackwardBackward0_target_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4801-4880

```cpp
4801:   }
4802:   END_HANDLE_TH_ERRORS
4803: }
4804: 
4805: static PyObject* THPUpsampleBilinear2DBackwardBackward0_output_size_getter(THPCppFunction *self, void *_unused) {
4806:   HANDLE_TH_ERRORS
4807:   auto prop = static_cast<UpsampleBilinear2DBackwardBackward0*>(self->cdata.get())->output_size;
4808:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4809:   for (auto i : c10::irange(prop.size())) {
4810:       auto si = prop[i];
4811:       if (auto m = si.maybe_as_int()) {
4812:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4813:       } else {
4814:         auto py_symint = py::cast(si).release().ptr();
4815:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4816:       }
4817:   }
4818:   return tup;
4819:   END_HANDLE_TH_ERRORS
4820: }
4821: 
4822: static PyObject* THPUpsampleBilinear2DBackwardBackward0_scales_h_getter(THPCppFunction *self, void *_unused) {
4823:   HANDLE_TH_ERRORS
4824:   auto opt_prop = static_cast<UpsampleBilinear2DBackwardBackward0*>(self->cdata.get())->scales_h;
4825:   if (!opt_prop.has_value()) {
4826:     Py_RETURN_NONE;
4827:   }
4828:   auto prop = opt_prop.value();
4829:   return PyFloat_FromDouble((double) prop);
4830:   END_HANDLE_TH_ERRORS
4831: }
4832: 
4833: static PyObject* THPUpsampleBilinear2DBackwardBackward0_scales_w_getter(THPCppFunction *self, void *_unused) {
4834:   HANDLE_TH_ERRORS
4835:   auto opt_prop = static_cast<UpsampleBilinear2DBackwardBackward0*>(self->cdata.get())->scales_w;
4836:   if (!opt_prop.has_value()) {
4837:     Py_RETURN_NONE;
4838:   }
4839:   auto prop = opt_prop.value();
4840:   return PyFloat_FromDouble((double) prop);
4841:   END_HANDLE_TH_ERRORS
4842: }
4843: 
4844: static struct PyGetSetDef UpsampleBilinear2DBackwardBackward0_properties[] = {
4845:   THP_FUNCTION_DEFAULT_PROPERTIES,
4846:   {(char*)"_saved_align_corners", (getter)THPUpsampleBilinear2DBackwardBackward0_align_corners_getter, nullptr, nullptr, nullptr},
4847:   {(char*)"_saved_output_size", (getter)THPUpsampleBilinear2DBackwardBackward0_output_size_getter, nullptr, nullptr, nullptr},
4848:   {(char*)"_saved_scales_h", (getter)THPUpsampleBilinear2DBackwardBackward0_scales_h_getter, nullptr, nullptr, nullptr},
4849:   {(char*)"_saved_scales_w", (getter)THPUpsampleBilinear2DBackwardBackward0_scales_w_getter, nullptr, nullptr, nullptr},
4850:   {nullptr} /* sentinel */
4851: };
4852: 
4853: static PyObject* THPUpsampleBicubic2DAaBackwardBackward0_align_corners_getter(THPCppFunction *self, void *_unused) {
4854:   HANDLE_TH_ERRORS
4855:   auto prop = static_cast<UpsampleBicubic2DAaBackwardBackward0*>(self->cdata.get())->align_corners;
4856:   if (prop) {
4857:     Py_RETURN_TRUE;
4858:   } else {
4859:     Py_RETURN_FALSE;
4860:   }
4861:   END_HANDLE_TH_ERRORS
4862: }
4863: 
4864: static PyObject* THPUpsampleBicubic2DAaBackwardBackward0_output_size_getter(THPCppFunction *self, void *_unused) {
4865:   HANDLE_TH_ERRORS
4866:   auto prop = static_cast<UpsampleBicubic2DAaBackwardBackward0*>(self->cdata.get())->output_size;
4867:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4868:   for (auto i : c10::irange(prop.size())) {
4869:       auto si = prop[i];
4870:       if (auto m = si.maybe_as_int()) {
4871:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4872:       } else {
4873:         auto py_symint = py::cast(si).release().ptr();
4874:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4875:       }
4876:   }
4877:   return tup;
4878:   END_HANDLE_TH_ERRORS
4879: }
4880: 
```

- EN: The main execution path in this span is carried by `THPUpsampleBilinear2DBackwardBackward0_output_size_getter`, `PyTuple_New`, `PyTuple_SetItem`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPUpsampleBilinear2DBackwardBackward0_output_size_getter`, `PyTuple_New`, `PyTuple_SetItem` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4881-4960

```cpp
4881: static PyObject* THPUpsampleBicubic2DAaBackwardBackward0_scales_h_getter(THPCppFunction *self, void *_unused) {
4882:   HANDLE_TH_ERRORS
4883:   auto opt_prop = static_cast<UpsampleBicubic2DAaBackwardBackward0*>(self->cdata.get())->scales_h;
4884:   if (!opt_prop.has_value()) {
4885:     Py_RETURN_NONE;
4886:   }
4887:   auto prop = opt_prop.value();
4888:   return PyFloat_FromDouble((double) prop);
4889:   END_HANDLE_TH_ERRORS
4890: }
4891: 
4892: static PyObject* THPUpsampleBicubic2DAaBackwardBackward0_scales_w_getter(THPCppFunction *self, void *_unused) {
4893:   HANDLE_TH_ERRORS
4894:   auto opt_prop = static_cast<UpsampleBicubic2DAaBackwardBackward0*>(self->cdata.get())->scales_w;
4895:   if (!opt_prop.has_value()) {
4896:     Py_RETURN_NONE;
4897:   }
4898:   auto prop = opt_prop.value();
4899:   return PyFloat_FromDouble((double) prop);
4900:   END_HANDLE_TH_ERRORS
4901: }
4902: 
4903: static struct PyGetSetDef UpsampleBicubic2DAaBackwardBackward0_properties[] = {
4904:   THP_FUNCTION_DEFAULT_PROPERTIES,
4905:   {(char*)"_saved_align_corners", (getter)THPUpsampleBicubic2DAaBackwardBackward0_align_corners_getter, nullptr, nullptr, nullptr},
4906:   {(char*)"_saved_output_size", (getter)THPUpsampleBicubic2DAaBackwardBackward0_output_size_getter, nullptr, nullptr, nullptr},
4907:   {(char*)"_saved_scales_h", (getter)THPUpsampleBicubic2DAaBackwardBackward0_scales_h_getter, nullptr, nullptr, nullptr},
4908:   {(char*)"_saved_scales_w", (getter)THPUpsampleBicubic2DAaBackwardBackward0_scales_w_getter, nullptr, nullptr, nullptr},
4909:   {nullptr} /* sentinel */
4910: };
4911: 
4912: static PyObject* THPUpsampleLanczos2DAaBackwardBackward0_align_corners_getter(THPCppFunction *self, void *_unused) {
4913:   HANDLE_TH_ERRORS
4914:   auto prop = static_cast<UpsampleLanczos2DAaBackwardBackward0*>(self->cdata.get())->align_corners;
4915:   if (prop) {
4916:     Py_RETURN_TRUE;
4917:   } else {
4918:     Py_RETURN_FALSE;
4919:   }
4920:   END_HANDLE_TH_ERRORS
4921: }
4922: 
4923: static PyObject* THPUpsampleLanczos2DAaBackwardBackward0_output_size_getter(THPCppFunction *self, void *_unused) {
4924:   HANDLE_TH_ERRORS
4925:   auto prop = static_cast<UpsampleLanczos2DAaBackwardBackward0*>(self->cdata.get())->output_size;
4926:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4927:   for (auto i : c10::irange(prop.size())) {
4928:       auto si = prop[i];
4929:       if (auto m = si.maybe_as_int()) {
4930:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4931:       } else {
4932:         auto py_symint = py::cast(si).release().ptr();
4933:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4934:       }
4935:   }
4936:   return tup;
4937:   END_HANDLE_TH_ERRORS
4938: }
4939: 
4940: static PyObject* THPUpsampleLanczos2DAaBackwardBackward0_scales_h_getter(THPCppFunction *self, void *_unused) {
4941:   HANDLE_TH_ERRORS
4942:   auto opt_prop = static_cast<UpsampleLanczos2DAaBackwardBackward0*>(self->cdata.get())->scales_h;
4943:   if (!opt_prop.has_value()) {
4944:     Py_RETURN_NONE;
4945:   }
4946:   auto prop = opt_prop.value();
4947:   return PyFloat_FromDouble((double) prop);
4948:   END_HANDLE_TH_ERRORS
4949: }
4950: 
4951: static PyObject* THPUpsampleLanczos2DAaBackwardBackward0_scales_w_getter(THPCppFunction *self, void *_unused) {
4952:   HANDLE_TH_ERRORS
4953:   auto opt_prop = static_cast<UpsampleLanczos2DAaBackwardBackward0*>(self->cdata.get())->scales_w;
4954:   if (!opt_prop.has_value()) {
4955:     Py_RETURN_NONE;
4956:   }
4957:   auto prop = opt_prop.value();
4958:   return PyFloat_FromDouble((double) prop);
4959:   END_HANDLE_TH_ERRORS
4960: }
```

- EN: The main execution path in this span is carried by `THPUpsampleBicubic2DAaBackwardBackward0_scales_h_getter`, `PyFloat_FromDouble`, `THPUpsampleBicubic2DAaBackwardBackward0_scales_w_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPUpsampleBicubic2DAaBackwardBackward0_scales_h_getter`, `PyFloat_FromDouble`, `THPUpsampleBicubic2DAaBackwardBackward0_scales_w_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4961-5040

```cpp
4961: 
4962: static struct PyGetSetDef UpsampleLanczos2DAaBackwardBackward0_properties[] = {
4963:   THP_FUNCTION_DEFAULT_PROPERTIES,
4964:   {(char*)"_saved_align_corners", (getter)THPUpsampleLanczos2DAaBackwardBackward0_align_corners_getter, nullptr, nullptr, nullptr},
4965:   {(char*)"_saved_output_size", (getter)THPUpsampleLanczos2DAaBackwardBackward0_output_size_getter, nullptr, nullptr, nullptr},
4966:   {(char*)"_saved_scales_h", (getter)THPUpsampleLanczos2DAaBackwardBackward0_scales_h_getter, nullptr, nullptr, nullptr},
4967:   {(char*)"_saved_scales_w", (getter)THPUpsampleLanczos2DAaBackwardBackward0_scales_w_getter, nullptr, nullptr, nullptr},
4968:   {nullptr} /* sentinel */
4969: };
4970: 
4971: static PyObject* THPUpsampleNearestExact1DBackwardBackward0_output_size_getter(THPCppFunction *self, void *_unused) {
4972:   HANDLE_TH_ERRORS
4973:   auto prop = static_cast<UpsampleNearestExact1DBackwardBackward0*>(self->cdata.get())->output_size;
4974:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4975:   for (auto i : c10::irange(prop.size())) {
4976:       auto si = prop[i];
4977:       if (auto m = si.maybe_as_int()) {
4978:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4979:       } else {
4980:         auto py_symint = py::cast(si).release().ptr();
4981:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4982:       }
4983:   }
4984:   return tup;
4985:   END_HANDLE_TH_ERRORS
4986: }
4987: 
4988: static PyObject* THPUpsampleNearestExact1DBackwardBackward0_scales_getter(THPCppFunction *self, void *_unused) {
4989:   HANDLE_TH_ERRORS
4990:   auto opt_prop = static_cast<UpsampleNearestExact1DBackwardBackward0*>(self->cdata.get())->scales;
4991:   if (!opt_prop.has_value()) {
4992:     Py_RETURN_NONE;
4993:   }
4994:   auto prop = opt_prop.value();
4995:   return PyFloat_FromDouble((double) prop);
4996:   END_HANDLE_TH_ERRORS
4997: }
4998: 
4999: static struct PyGetSetDef UpsampleNearestExact1DBackwardBackward0_properties[] = {
5000:   THP_FUNCTION_DEFAULT_PROPERTIES,
5001:   {(char*)"_saved_output_size", (getter)THPUpsampleNearestExact1DBackwardBackward0_output_size_getter, nullptr, nullptr, nullptr},
5002:   {(char*)"_saved_scales", (getter)THPUpsampleNearestExact1DBackwardBackward0_scales_getter, nullptr, nullptr, nullptr},
5003:   {nullptr} /* sentinel */
5004: };
5005: 
5006: static PyObject* THPUpsampleNearestExact2DBackwardBackward0_output_size_getter(THPCppFunction *self, void *_unused) {
5007:   HANDLE_TH_ERRORS
5008:   auto prop = static_cast<UpsampleNearestExact2DBackwardBackward0*>(self->cdata.get())->output_size;
5009:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5010:   for (auto i : c10::irange(prop.size())) {
5011:       auto si = prop[i];
5012:       if (auto m = si.maybe_as_int()) {
5013:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
5014:       } else {
5015:         auto py_symint = py::cast(si).release().ptr();
5016:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
5017:       }
5018:   }
5019:   return tup;
5020:   END_HANDLE_TH_ERRORS
5021: }
5022: 
5023: static PyObject* THPUpsampleNearestExact2DBackwardBackward0_scales_h_getter(THPCppFunction *self, void *_unused) {
5024:   HANDLE_TH_ERRORS
5025:   auto opt_prop = static_cast<UpsampleNearestExact2DBackwardBackward0*>(self->cdata.get())->scales_h;
5026:   if (!opt_prop.has_value()) {
5027:     Py_RETURN_NONE;
5028:   }
5029:   auto prop = opt_prop.value();
5030:   return PyFloat_FromDouble((double) prop);
5031:   END_HANDLE_TH_ERRORS
5032: }
5033: 
5034: static PyObject* THPUpsampleNearestExact2DBackwardBackward0_scales_w_getter(THPCppFunction *self, void *_unused) {
5035:   HANDLE_TH_ERRORS
5036:   auto opt_prop = static_cast<UpsampleNearestExact2DBackwardBackward0*>(self->cdata.get())->scales_w;
5037:   if (!opt_prop.has_value()) {
5038:     Py_RETURN_NONE;
5039:   }
5040:   auto prop = opt_prop.value();
```

- EN: The main execution path in this span is carried by `THPUpsampleNearestExact1DBackwardBackward0_output_size_getter`, `PyTuple_New`, `PyTuple_SetItem`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPUpsampleNearestExact1DBackwardBackward0_output_size_getter`, `PyTuple_New`, `PyTuple_SetItem` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5041-5120

```cpp
5041:   return PyFloat_FromDouble((double) prop);
5042:   END_HANDLE_TH_ERRORS
5043: }
5044: 
5045: static struct PyGetSetDef UpsampleNearestExact2DBackwardBackward0_properties[] = {
5046:   THP_FUNCTION_DEFAULT_PROPERTIES,
5047:   {(char*)"_saved_output_size", (getter)THPUpsampleNearestExact2DBackwardBackward0_output_size_getter, nullptr, nullptr, nullptr},
5048:   {(char*)"_saved_scales_h", (getter)THPUpsampleNearestExact2DBackwardBackward0_scales_h_getter, nullptr, nullptr, nullptr},
5049:   {(char*)"_saved_scales_w", (getter)THPUpsampleNearestExact2DBackwardBackward0_scales_w_getter, nullptr, nullptr, nullptr},
5050:   {nullptr} /* sentinel */
5051: };
5052: 
5053: static PyObject* THPMiopenConvolutionBackward0_bias_sym_sizes_opt_getter(THPCppFunction *self, void *_unused) {
5054:   HANDLE_TH_ERRORS
5055:   auto opt_prop = static_cast<MiopenConvolutionBackward0*>(self->cdata.get())->bias_sym_sizes_opt;
5056:   if (!opt_prop.list.has_value()) {
5057:     Py_RETURN_NONE;
5058:   }
5059:   auto prop = opt_prop.list.value();
5060:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5061:   for (auto i : c10::irange(prop.size())) {
5062:       auto si = prop[i];
5063:       if (auto m = si.maybe_as_int()) {
5064:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
5065:       } else {
5066:         auto py_symint = py::cast(si).release().ptr();
5067:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
5068:       }
5069:   }
5070:   return tup;
5071:   END_HANDLE_TH_ERRORS
5072: }
5073: 
5074: static PyObject* THPMiopenConvolutionBackward0_dilation_getter(THPCppFunction *self, void *_unused) {
5075:   HANDLE_TH_ERRORS
5076:   auto prop = static_cast<MiopenConvolutionBackward0*>(self->cdata.get())->dilation;
5077:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5078:   for (auto i : c10::irange(prop.size())) {
5079:       auto si = prop[i];
5080:       if (auto m = si.maybe_as_int()) {
5081:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
5082:       } else {
5083:         auto py_symint = py::cast(si).release().ptr();
5084:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
5085:       }
5086:   }
5087:   return tup;
5088:   END_HANDLE_TH_ERRORS
5089: }
5090: 
5091: static PyObject* THPMiopenConvolutionBackward0_groups_getter(THPCppFunction *self, void *_unused) {
5092:   HANDLE_TH_ERRORS
5093:   auto prop = static_cast<MiopenConvolutionBackward0*>(self->cdata.get())->groups;
5094:   if (auto m = prop.maybe_as_int()) {
5095:     return PyLong_FromUnsignedLong(*m);
5096:   } else {
5097:     return py::cast(prop).release().ptr();
5098:   }
5099:   END_HANDLE_TH_ERRORS
5100: }
5101: 
5102: static PyObject* THPMiopenConvolutionBackward0_padding_getter(THPCppFunction *self, void *_unused) {
5103:   HANDLE_TH_ERRORS
5104:   auto prop = static_cast<MiopenConvolutionBackward0*>(self->cdata.get())->padding;
5105:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5106:   for (auto i : c10::irange(prop.size())) {
5107:       auto si = prop[i];
5108:       if (auto m = si.maybe_as_int()) {
5109:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
5110:       } else {
5111:         auto py_symint = py::cast(si).release().ptr();
5112:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
5113:       }
5114:   }
5115:   return tup;
5116:   END_HANDLE_TH_ERRORS
5117: }
5118: 
5119: static PyObject* THPMiopenConvolutionBackward0_self_getter(THPCppFunction *self, void *_unused) {
5120:   HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `PyFloat_FromDouble`, `THPMiopenConvolutionBackward0_bias_sym_sizes_opt_getter`, `PyTuple_New`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyFloat_FromDouble`, `THPMiopenConvolutionBackward0_bias_sym_sizes_opt_getter`, `PyTuple_New` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5121-5200

```cpp
5121:   const auto& prop = static_cast<MiopenConvolutionBackward0*>(self->cdata.get())->self_;
5122:   return THPVariable_Wrap(prop.unpack(self->cdata));
5123:   END_HANDLE_TH_ERRORS
5124: }
5125: 
5126: static PyObject* THPMiopenConvolutionBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
5127:   HANDLE_TH_ERRORS
5128:   const auto& prop = static_cast<MiopenConvolutionBackward0*>(self->cdata.get())->self_;
5129:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5130:   return obj.release().ptr();
5131:   END_HANDLE_TH_ERRORS
5132: }
5133: 
5134: static PyObject* THPMiopenConvolutionBackward0_stride_getter(THPCppFunction *self, void *_unused) {
5135:   HANDLE_TH_ERRORS
5136:   auto prop = static_cast<MiopenConvolutionBackward0*>(self->cdata.get())->stride;
5137:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5138:   for (auto i : c10::irange(prop.size())) {
5139:       auto si = prop[i];
5140:       if (auto m = si.maybe_as_int()) {
5141:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
5142:       } else {
5143:         auto py_symint = py::cast(si).release().ptr();
5144:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
5145:       }
5146:   }
5147:   return tup;
5148:   END_HANDLE_TH_ERRORS
5149: }
5150: 
5151: static PyObject* THPMiopenConvolutionBackward0_weight_getter(THPCppFunction *self, void *_unused) {
5152:   HANDLE_TH_ERRORS
5153:   const auto& prop = static_cast<MiopenConvolutionBackward0*>(self->cdata.get())->weight_;
5154:   return THPVariable_Wrap(prop.unpack(self->cdata));
5155:   END_HANDLE_TH_ERRORS
5156: }
5157: 
5158: static PyObject* THPMiopenConvolutionBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
5159:   HANDLE_TH_ERRORS
5160:   const auto& prop = static_cast<MiopenConvolutionBackward0*>(self->cdata.get())->weight_;
5161:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5162:   return obj.release().ptr();
5163:   END_HANDLE_TH_ERRORS
5164: }
5165: 
5166: static struct PyGetSetDef MiopenConvolutionBackward0_properties[] = {
5167:   THP_FUNCTION_DEFAULT_PROPERTIES,
5168:   {(char*)"_saved_bias_sym_sizes_opt", (getter)THPMiopenConvolutionBackward0_bias_sym_sizes_opt_getter, nullptr, nullptr, nullptr},
5169:   {(char*)"_saved_dilation", (getter)THPMiopenConvolutionBackward0_dilation_getter, nullptr, nullptr, nullptr},
5170:   {(char*)"_saved_groups", (getter)THPMiopenConvolutionBackward0_groups_getter, nullptr, nullptr, nullptr},
5171:   {(char*)"_saved_padding", (getter)THPMiopenConvolutionBackward0_padding_getter, nullptr, nullptr, nullptr},
5172:   {(char*)"_saved_self", (getter)THPMiopenConvolutionBackward0_self_getter, nullptr, nullptr, nullptr},
5173:   {(char*)"_raw_saved_self", (getter)THPMiopenConvolutionBackward0_self_raw_getter, nullptr, nullptr, nullptr},
5174:   {(char*)"_saved_stride", (getter)THPMiopenConvolutionBackward0_stride_getter, nullptr, nullptr, nullptr},
5175:   {(char*)"_saved_weight", (getter)THPMiopenConvolutionBackward0_weight_getter, nullptr, nullptr, nullptr},
5176:   {(char*)"_raw_saved_weight", (getter)THPMiopenConvolutionBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
5177:   {nullptr} /* sentinel */
5178: };
5179: 
5180: static PyObject* THPMiopenRnnBackward0_batch_first_getter(THPCppFunction *self, void *_unused) {
5181:   HANDLE_TH_ERRORS
5182:   auto prop = static_cast<MiopenRnnBackward0*>(self->cdata.get())->batch_first;
5183:   if (prop) {
5184:     Py_RETURN_TRUE;
5185:   } else {
5186:     Py_RETURN_FALSE;
5187:   }
5188:   END_HANDLE_TH_ERRORS
5189: }
5190: 
5191: static PyObject* THPMiopenRnnBackward0_batch_sizes_getter(THPCppFunction *self, void *_unused) {
5192:   HANDLE_TH_ERRORS
5193:   auto prop = static_cast<MiopenRnnBackward0*>(self->cdata.get())->batch_sizes;
5194:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5195:   for (auto i : c10::irange(prop.size())) {
5196:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
5197:   }
5198:   return tup;
5199:   END_HANDLE_TH_ERRORS
5200: }
```

- EN: The main execution path in this span is carried by `THPVariable_Wrap`, `THPMiopenConvolutionBackward0_self_raw_getter`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_Wrap`, `THPMiopenConvolutionBackward0_self_raw_getter`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5201-5280

```cpp
5201: 
5202: static PyObject* THPMiopenRnnBackward0_bidirectional_getter(THPCppFunction *self, void *_unused) {
5203:   HANDLE_TH_ERRORS
5204:   auto prop = static_cast<MiopenRnnBackward0*>(self->cdata.get())->bidirectional;
5205:   if (prop) {
5206:     Py_RETURN_TRUE;
5207:   } else {
5208:     Py_RETURN_FALSE;
5209:   }
5210:   END_HANDLE_TH_ERRORS
5211: }
5212: 
5213: static PyObject* THPMiopenRnnBackward0_cx_getter(THPCppFunction *self, void *_unused) {
5214:   HANDLE_TH_ERRORS
5215:   const auto& prop = static_cast<MiopenRnnBackward0*>(self->cdata.get())->cx_;
5216:   return THPVariable_Wrap(prop.unpack(self->cdata));
5217:   END_HANDLE_TH_ERRORS
5218: }
5219: 
5220: static PyObject* THPMiopenRnnBackward0_cx_raw_getter(THPCppFunction *self, void *_unused) {
5221:   HANDLE_TH_ERRORS
5222:   const auto& prop = static_cast<MiopenRnnBackward0*>(self->cdata.get())->cx_;
5223:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5224:   return obj.release().ptr();
5225:   END_HANDLE_TH_ERRORS
5226: }
5227: 
5228: static PyObject* THPMiopenRnnBackward0_dropout_getter(THPCppFunction *self, void *_unused) {
5229:   HANDLE_TH_ERRORS
5230:   auto prop = static_cast<MiopenRnnBackward0*>(self->cdata.get())->dropout;
5231:   return PyFloat_FromDouble((double) prop);
5232:   END_HANDLE_TH_ERRORS
5233: }
5234: 
5235: static PyObject* THPMiopenRnnBackward0_dropout_state_getter(THPCppFunction *self, void *_unused) {
5236:   HANDLE_TH_ERRORS
5237:   const auto& prop = static_cast<MiopenRnnBackward0*>(self->cdata.get())->dropout_state_;
5238:   return THPVariable_Wrap(prop.unpack(self->cdata));
5239:   END_HANDLE_TH_ERRORS
5240: }
5241: 
5242: static PyObject* THPMiopenRnnBackward0_dropout_state_raw_getter(THPCppFunction *self, void *_unused) {
5243:   HANDLE_TH_ERRORS
5244:   const auto& prop = static_cast<MiopenRnnBackward0*>(self->cdata.get())->dropout_state_;
5245:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5246:   return obj.release().ptr();
5247:   END_HANDLE_TH_ERRORS
5248: }
5249: 
5250: static PyObject* THPMiopenRnnBackward0_hidden_size_getter(THPCppFunction *self, void *_unused) {
5251:   HANDLE_TH_ERRORS
5252:   auto prop = static_cast<MiopenRnnBackward0*>(self->cdata.get())->hidden_size;
5253:   return PyLong_FromUnsignedLong((int64_t) prop);
5254:   END_HANDLE_TH_ERRORS
5255: }
5256: 
5257: static PyObject* THPMiopenRnnBackward0_hx_getter(THPCppFunction *self, void *_unused) {
5258:   HANDLE_TH_ERRORS
5259:   const auto& prop = static_cast<MiopenRnnBackward0*>(self->cdata.get())->hx_;
5260:   return THPVariable_Wrap(prop.unpack(self->cdata));
5261:   END_HANDLE_TH_ERRORS
5262: }
5263: 
5264: static PyObject* THPMiopenRnnBackward0_hx_raw_getter(THPCppFunction *self, void *_unused) {
5265:   HANDLE_TH_ERRORS
5266:   const auto& prop = static_cast<MiopenRnnBackward0*>(self->cdata.get())->hx_;
5267:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5268:   return obj.release().ptr();
5269:   END_HANDLE_TH_ERRORS
5270: }
5271: 
5272: static PyObject* THPMiopenRnnBackward0_input_getter(THPCppFunction *self, void *_unused) {
5273:   HANDLE_TH_ERRORS
5274:   const auto& prop = static_cast<MiopenRnnBackward0*>(self->cdata.get())->input_;
5275:   return THPVariable_Wrap(prop.unpack(self->cdata));
5276:   END_HANDLE_TH_ERRORS
5277: }
5278: 
5279: static PyObject* THPMiopenRnnBackward0_input_raw_getter(THPCppFunction *self, void *_unused) {
5280:   HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPMiopenRnnBackward0_bidirectional_getter`, `THPMiopenRnnBackward0_cx_getter`, `THPVariable_Wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPMiopenRnnBackward0_bidirectional_getter`, `THPMiopenRnnBackward0_cx_getter`, `THPVariable_Wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5281-5360

```cpp
5281:   const auto& prop = static_cast<MiopenRnnBackward0*>(self->cdata.get())->input_;
5282:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5283:   return obj.release().ptr();
5284:   END_HANDLE_TH_ERRORS
5285: }
5286: 
5287: static PyObject* THPMiopenRnnBackward0_mode_getter(THPCppFunction *self, void *_unused) {
5288:   HANDLE_TH_ERRORS
5289:   auto prop = static_cast<MiopenRnnBackward0*>(self->cdata.get())->mode;
5290:   return PyLong_FromUnsignedLong((int64_t) prop);
5291:   END_HANDLE_TH_ERRORS
5292: }
5293: 
5294: static PyObject* THPMiopenRnnBackward0_num_layers_getter(THPCppFunction *self, void *_unused) {
5295:   HANDLE_TH_ERRORS
5296:   auto prop = static_cast<MiopenRnnBackward0*>(self->cdata.get())->num_layers;
5297:   return PyLong_FromUnsignedLong((int64_t) prop);
5298:   END_HANDLE_TH_ERRORS
5299: }
5300: 
5301: static PyObject* THPMiopenRnnBackward0_train_getter(THPCppFunction *self, void *_unused) {
5302:   HANDLE_TH_ERRORS
5303:   auto prop = static_cast<MiopenRnnBackward0*>(self->cdata.get())->train;
5304:   if (prop) {
5305:     Py_RETURN_TRUE;
5306:   } else {
5307:     Py_RETURN_FALSE;
5308:   }
5309:   END_HANDLE_TH_ERRORS
5310: }
5311: 
5312: static PyObject* THPMiopenRnnBackward0_weight_getter(THPCppFunction *self, void *_unused) {
5313:   HANDLE_TH_ERRORS
5314:   const auto *node = static_cast<MiopenRnnBackward0*>(self->cdata.get());
5315:   const auto& prop = node->weight_;
5316:   if (node->weight_released_) {
5317:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
5318:     return nullptr;
5319:   }
5320:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5321:   for (auto i: c10::irange(prop.size())) {
5322:     PyTuple_SetItem(tup, (Py_ssize_t) i, THPVariable_Wrap(prop[i].unpack(self->cdata)));
5323:   }
5324:   return tup;
5325:   END_HANDLE_TH_ERRORS
5326: }
5327: 
5328: static PyObject* THPMiopenRnnBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
5329:   HANDLE_TH_ERRORS
5330:   const auto *node = static_cast<MiopenRnnBackward0*>(self->cdata.get());
5331:   const auto& prop = node->weight_;
5332:   if (node->weight_released_) {
5333:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
5334:     return nullptr;
5335:   }
5336:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5337:   for (auto i : c10::irange(prop.size())) {
5338:     pybind11::object obj = pybind11::cast(prop[i], pybind11::return_value_policy::reference);
5339:     PyTuple_SetItem(tup, (Py_ssize_t) i, obj.release().ptr());
5340:   }
5341:   return tup;
5342:   END_HANDLE_TH_ERRORS
5343: }
5344: 
5345: static PyObject* THPMiopenRnnBackward0_weight_stride0_getter(THPCppFunction *self, void *_unused) {
5346:   HANDLE_TH_ERRORS
5347:   auto prop = static_cast<MiopenRnnBackward0*>(self->cdata.get())->weight_stride0;
5348:   return PyLong_FromUnsignedLong((int64_t) prop);
5349:   END_HANDLE_TH_ERRORS
5350: }
5351: 
5352: static PyObject* THPMiopenRnnBackward0_result0_getter(THPCppFunction *self, void *_unused) {
5353:   HANDLE_TH_ERRORS
5354:   const auto& prop = static_cast<MiopenRnnBackward0*>(self->cdata.get())->result0_;
5355:   return THPVariable_Wrap(prop.unpack(self->cdata));
5356:   END_HANDLE_TH_ERRORS
5357: }
5358: 
5359: static PyObject* THPMiopenRnnBackward0_result0_raw_getter(THPCppFunction *self, void *_unused) {
5360:   HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `cast`, `THPMiopenRnnBackward0_mode_getter`, `PyLong_FromUnsignedLong`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `cast`, `THPMiopenRnnBackward0_mode_getter`, `PyLong_FromUnsignedLong` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5361-5440

```cpp
5361:   const auto& prop = static_cast<MiopenRnnBackward0*>(self->cdata.get())->result0_;
5362:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5363:   return obj.release().ptr();
5364:   END_HANDLE_TH_ERRORS
5365: }
5366: 
5367: static PyObject* THPMiopenRnnBackward0_result3_getter(THPCppFunction *self, void *_unused) {
5368:   HANDLE_TH_ERRORS
5369:   const auto& prop = static_cast<MiopenRnnBackward0*>(self->cdata.get())->result3_;
5370:   return THPVariable_Wrap(prop.unpack(self->cdata));
5371:   END_HANDLE_TH_ERRORS
5372: }
5373: 
5374: static PyObject* THPMiopenRnnBackward0_result3_raw_getter(THPCppFunction *self, void *_unused) {
5375:   HANDLE_TH_ERRORS
5376:   const auto& prop = static_cast<MiopenRnnBackward0*>(self->cdata.get())->result3_;
5377:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5378:   return obj.release().ptr();
5379:   END_HANDLE_TH_ERRORS
5380: }
5381: 
5382: static PyObject* THPMiopenRnnBackward0_result4_getter(THPCppFunction *self, void *_unused) {
5383:   HANDLE_TH_ERRORS
5384:   const auto& prop = static_cast<MiopenRnnBackward0*>(self->cdata.get())->result4_;
5385:   return THPVariable_Wrap(prop.unpack(self->cdata));
5386:   END_HANDLE_TH_ERRORS
5387: }
5388: 
5389: static PyObject* THPMiopenRnnBackward0_result4_raw_getter(THPCppFunction *self, void *_unused) {
5390:   HANDLE_TH_ERRORS
5391:   const auto& prop = static_cast<MiopenRnnBackward0*>(self->cdata.get())->result4_;
5392:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5393:   return obj.release().ptr();
5394:   END_HANDLE_TH_ERRORS
5395: }
5396: 
5397: static struct PyGetSetDef MiopenRnnBackward0_properties[] = {
5398:   THP_FUNCTION_DEFAULT_PROPERTIES,
5399:   {(char*)"_saved_batch_first", (getter)THPMiopenRnnBackward0_batch_first_getter, nullptr, nullptr, nullptr},
5400:   {(char*)"_saved_batch_sizes", (getter)THPMiopenRnnBackward0_batch_sizes_getter, nullptr, nullptr, nullptr},
5401:   {(char*)"_saved_bidirectional", (getter)THPMiopenRnnBackward0_bidirectional_getter, nullptr, nullptr, nullptr},
5402:   {(char*)"_saved_cx", (getter)THPMiopenRnnBackward0_cx_getter, nullptr, nullptr, nullptr},
5403:   {(char*)"_raw_saved_cx", (getter)THPMiopenRnnBackward0_cx_raw_getter, nullptr, nullptr, nullptr},
5404:   {(char*)"_saved_dropout", (getter)THPMiopenRnnBackward0_dropout_getter, nullptr, nullptr, nullptr},
5405:   {(char*)"_saved_dropout_state", (getter)THPMiopenRnnBackward0_dropout_state_getter, nullptr, nullptr, nullptr},
5406:   {(char*)"_raw_saved_dropout_state", (getter)THPMiopenRnnBackward0_dropout_state_raw_getter, nullptr, nullptr, nullptr},
5407:   {(char*)"_saved_hidden_size", (getter)THPMiopenRnnBackward0_hidden_size_getter, nullptr, nullptr, nullptr},
5408:   {(char*)"_saved_hx", (getter)THPMiopenRnnBackward0_hx_getter, nullptr, nullptr, nullptr},
5409:   {(char*)"_raw_saved_hx", (getter)THPMiopenRnnBackward0_hx_raw_getter, nullptr, nullptr, nullptr},
5410:   {(char*)"_saved_input", (getter)THPMiopenRnnBackward0_input_getter, nullptr, nullptr, nullptr},
5411:   {(char*)"_raw_saved_input", (getter)THPMiopenRnnBackward0_input_raw_getter, nullptr, nullptr, nullptr},
5412:   {(char*)"_saved_mode", (getter)THPMiopenRnnBackward0_mode_getter, nullptr, nullptr, nullptr},
5413:   {(char*)"_saved_num_layers", (getter)THPMiopenRnnBackward0_num_layers_getter, nullptr, nullptr, nullptr},
5414:   {(char*)"_saved_train", (getter)THPMiopenRnnBackward0_train_getter, nullptr, nullptr, nullptr},
5415:   {(char*)"_saved_weight", (getter)THPMiopenRnnBackward0_weight_getter, nullptr, nullptr, nullptr},
5416:   {(char*)"_raw_saved_weight", (getter)THPMiopenRnnBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
5417:   {(char*)"_saved_weight_stride0", (getter)THPMiopenRnnBackward0_weight_stride0_getter, nullptr, nullptr, nullptr},
5418:   {(char*)"_saved_result0", (getter)THPMiopenRnnBackward0_result0_getter, nullptr, nullptr, nullptr},
5419:   {(char*)"_raw_saved_result0", (getter)THPMiopenRnnBackward0_result0_raw_getter, nullptr, nullptr, nullptr},
5420:   {(char*)"_saved_result3", (getter)THPMiopenRnnBackward0_result3_getter, nullptr, nullptr, nullptr},
5421:   {(char*)"_raw_saved_result3", (getter)THPMiopenRnnBackward0_result3_raw_getter, nullptr, nullptr, nullptr},
5422:   {(char*)"_saved_result4", (getter)THPMiopenRnnBackward0_result4_getter, nullptr, nullptr, nullptr},
5423:   {(char*)"_raw_saved_result4", (getter)THPMiopenRnnBackward0_result4_raw_getter, nullptr, nullptr, nullptr},
5424:   {nullptr} /* sentinel */
5425: };
5426: 
5427: static PyObject* THPMkldnnRnnLayerBackward0_batch_first_getter(THPCppFunction *self, void *_unused) {
5428:   HANDLE_TH_ERRORS
5429:   auto prop = static_cast<MkldnnRnnLayerBackward0*>(self->cdata.get())->batch_first;
5430:   if (prop) {
5431:     Py_RETURN_TRUE;
5432:   } else {
5433:     Py_RETURN_FALSE;
5434:   }
5435:   END_HANDLE_TH_ERRORS
5436: }
5437: 
5438: static PyObject* THPMkldnnRnnLayerBackward0_batch_sizes_getter(THPCppFunction *self, void *_unused) {
5439:   HANDLE_TH_ERRORS
5440:   auto prop = static_cast<MkldnnRnnLayerBackward0*>(self->cdata.get())->batch_sizes;
```

- EN: The main execution path in this span is carried by `cast`, `THPMiopenRnnBackward0_result3_getter`, `THPVariable_Wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `cast`, `THPMiopenRnnBackward0_result3_getter`, `THPVariable_Wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5441-5520

```cpp
5441:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5442:   for (auto i : c10::irange(prop.size())) {
5443:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
5444:   }
5445:   return tup;
5446:   END_HANDLE_TH_ERRORS
5447: }
5448: 
5449: static PyObject* THPMkldnnRnnLayerBackward0_bidirectional_getter(THPCppFunction *self, void *_unused) {
5450:   HANDLE_TH_ERRORS
5451:   auto prop = static_cast<MkldnnRnnLayerBackward0*>(self->cdata.get())->bidirectional;
5452:   if (prop) {
5453:     Py_RETURN_TRUE;
5454:   } else {
5455:     Py_RETURN_FALSE;
5456:   }
5457:   END_HANDLE_TH_ERRORS
5458: }
5459: 
5460: static PyObject* THPMkldnnRnnLayerBackward0_cx__getter(THPCppFunction *self, void *_unused) {
5461:   HANDLE_TH_ERRORS
5462:   const auto& prop = static_cast<MkldnnRnnLayerBackward0*>(self->cdata.get())->cx__;
5463:   return THPVariable_Wrap(prop.unpack(self->cdata));
5464:   END_HANDLE_TH_ERRORS
5465: }
5466: 
5467: static PyObject* THPMkldnnRnnLayerBackward0_cx__raw_getter(THPCppFunction *self, void *_unused) {
5468:   HANDLE_TH_ERRORS
5469:   const auto& prop = static_cast<MkldnnRnnLayerBackward0*>(self->cdata.get())->cx__;
5470:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5471:   return obj.release().ptr();
5472:   END_HANDLE_TH_ERRORS
5473: }
5474: 
5475: static PyObject* THPMkldnnRnnLayerBackward0_has_biases_getter(THPCppFunction *self, void *_unused) {
5476:   HANDLE_TH_ERRORS
5477:   auto prop = static_cast<MkldnnRnnLayerBackward0*>(self->cdata.get())->has_biases;
5478:   if (prop) {
5479:     Py_RETURN_TRUE;
5480:   } else {
5481:     Py_RETURN_FALSE;
5482:   }
5483:   END_HANDLE_TH_ERRORS
5484: }
5485: 
5486: static PyObject* THPMkldnnRnnLayerBackward0_hidden_size_getter(THPCppFunction *self, void *_unused) {
5487:   HANDLE_TH_ERRORS
5488:   auto prop = static_cast<MkldnnRnnLayerBackward0*>(self->cdata.get())->hidden_size;
5489:   return PyLong_FromUnsignedLong((int64_t) prop);
5490:   END_HANDLE_TH_ERRORS
5491: }
5492: 
5493: static PyObject* THPMkldnnRnnLayerBackward0_hx__getter(THPCppFunction *self, void *_unused) {
5494:   HANDLE_TH_ERRORS
5495:   const auto& prop = static_cast<MkldnnRnnLayerBackward0*>(self->cdata.get())->hx__;
5496:   return THPVariable_Wrap(prop.unpack(self->cdata));
5497:   END_HANDLE_TH_ERRORS
5498: }
5499: 
5500: static PyObject* THPMkldnnRnnLayerBackward0_hx__raw_getter(THPCppFunction *self, void *_unused) {
5501:   HANDLE_TH_ERRORS
5502:   const auto& prop = static_cast<MkldnnRnnLayerBackward0*>(self->cdata.get())->hx__;
5503:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5504:   return obj.release().ptr();
5505:   END_HANDLE_TH_ERRORS
5506: }
5507: 
5508: static PyObject* THPMkldnnRnnLayerBackward0_input_getter(THPCppFunction *self, void *_unused) {
5509:   HANDLE_TH_ERRORS
5510:   const auto& prop = static_cast<MkldnnRnnLayerBackward0*>(self->cdata.get())->input_;
5511:   return THPVariable_Wrap(prop.unpack(self->cdata));
5512:   END_HANDLE_TH_ERRORS
5513: }
5514: 
5515: static PyObject* THPMkldnnRnnLayerBackward0_input_raw_getter(THPCppFunction *self, void *_unused) {
5516:   HANDLE_TH_ERRORS
5517:   const auto& prop = static_cast<MkldnnRnnLayerBackward0*>(self->cdata.get())->input_;
5518:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5519:   return obj.release().ptr();
5520:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `PyTuple_New`, `PyTuple_SetItem`, `THPMkldnnRnnLayerBackward0_bidirectional_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyTuple_New`, `PyTuple_SetItem`, `THPMkldnnRnnLayerBackward0_bidirectional_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5521-5600

```cpp
5521: }
5522: 
5523: static PyObject* THPMkldnnRnnLayerBackward0_mode_getter(THPCppFunction *self, void *_unused) {
5524:   HANDLE_TH_ERRORS
5525:   auto prop = static_cast<MkldnnRnnLayerBackward0*>(self->cdata.get())->mode;
5526:   return PyLong_FromUnsignedLong((int64_t) prop);
5527:   END_HANDLE_TH_ERRORS
5528: }
5529: 
5530: static PyObject* THPMkldnnRnnLayerBackward0_num_layers_getter(THPCppFunction *self, void *_unused) {
5531:   HANDLE_TH_ERRORS
5532:   auto prop = static_cast<MkldnnRnnLayerBackward0*>(self->cdata.get())->num_layers;
5533:   return PyLong_FromUnsignedLong((int64_t) prop);
5534:   END_HANDLE_TH_ERRORS
5535: }
5536: 
5537: static PyObject* THPMkldnnRnnLayerBackward0_reverse_getter(THPCppFunction *self, void *_unused) {
5538:   HANDLE_TH_ERRORS
5539:   auto prop = static_cast<MkldnnRnnLayerBackward0*>(self->cdata.get())->reverse;
5540:   if (prop) {
5541:     Py_RETURN_TRUE;
5542:   } else {
5543:     Py_RETURN_FALSE;
5544:   }
5545:   END_HANDLE_TH_ERRORS
5546: }
5547: 
5548: static PyObject* THPMkldnnRnnLayerBackward0_train_getter(THPCppFunction *self, void *_unused) {
5549:   HANDLE_TH_ERRORS
5550:   auto prop = static_cast<MkldnnRnnLayerBackward0*>(self->cdata.get())->train;
5551:   if (prop) {
5552:     Py_RETURN_TRUE;
5553:   } else {
5554:     Py_RETURN_FALSE;
5555:   }
5556:   END_HANDLE_TH_ERRORS
5557: }
5558: 
5559: static PyObject* THPMkldnnRnnLayerBackward0_weight0_getter(THPCppFunction *self, void *_unused) {
5560:   HANDLE_TH_ERRORS
5561:   const auto& prop = static_cast<MkldnnRnnLayerBackward0*>(self->cdata.get())->weight0_;
5562:   return THPVariable_Wrap(prop.unpack(self->cdata));
5563:   END_HANDLE_TH_ERRORS
5564: }
5565: 
5566: static PyObject* THPMkldnnRnnLayerBackward0_weight0_raw_getter(THPCppFunction *self, void *_unused) {
5567:   HANDLE_TH_ERRORS
5568:   const auto& prop = static_cast<MkldnnRnnLayerBackward0*>(self->cdata.get())->weight0_;
5569:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5570:   return obj.release().ptr();
5571:   END_HANDLE_TH_ERRORS
5572: }
5573: 
5574: static PyObject* THPMkldnnRnnLayerBackward0_weight1_getter(THPCppFunction *self, void *_unused) {
5575:   HANDLE_TH_ERRORS
5576:   const auto& prop = static_cast<MkldnnRnnLayerBackward0*>(self->cdata.get())->weight1_;
5577:   return THPVariable_Wrap(prop.unpack(self->cdata));
5578:   END_HANDLE_TH_ERRORS
5579: }
5580: 
5581: static PyObject* THPMkldnnRnnLayerBackward0_weight1_raw_getter(THPCppFunction *self, void *_unused) {
5582:   HANDLE_TH_ERRORS
5583:   const auto& prop = static_cast<MkldnnRnnLayerBackward0*>(self->cdata.get())->weight1_;
5584:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5585:   return obj.release().ptr();
5586:   END_HANDLE_TH_ERRORS
5587: }
5588: 
5589: static PyObject* THPMkldnnRnnLayerBackward0_weight2_getter(THPCppFunction *self, void *_unused) {
5590:   HANDLE_TH_ERRORS
5591:   const auto& prop = static_cast<MkldnnRnnLayerBackward0*>(self->cdata.get())->weight2_;
5592:   return THPVariable_Wrap(prop.unpack(self->cdata));
5593:   END_HANDLE_TH_ERRORS
5594: }
5595: 
5596: static PyObject* THPMkldnnRnnLayerBackward0_weight2_raw_getter(THPCppFunction *self, void *_unused) {
5597:   HANDLE_TH_ERRORS
5598:   const auto& prop = static_cast<MkldnnRnnLayerBackward0*>(self->cdata.get())->weight2_;
5599:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5600:   return obj.release().ptr();
```

- EN: The main execution path in this span is carried by `THPMkldnnRnnLayerBackward0_mode_getter`, `PyLong_FromUnsignedLong`, `THPMkldnnRnnLayerBackward0_num_layers_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPMkldnnRnnLayerBackward0_mode_getter`, `PyLong_FromUnsignedLong`, `THPMkldnnRnnLayerBackward0_num_layers_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5601-5680

```cpp
5601:   END_HANDLE_TH_ERRORS
5602: }
5603: 
5604: static PyObject* THPMkldnnRnnLayerBackward0_weight3_getter(THPCppFunction *self, void *_unused) {
5605:   HANDLE_TH_ERRORS
5606:   const auto& prop = static_cast<MkldnnRnnLayerBackward0*>(self->cdata.get())->weight3_;
5607:   return THPVariable_Wrap(prop.unpack(self->cdata));
5608:   END_HANDLE_TH_ERRORS
5609: }
5610: 
5611: static PyObject* THPMkldnnRnnLayerBackward0_weight3_raw_getter(THPCppFunction *self, void *_unused) {
5612:   HANDLE_TH_ERRORS
5613:   const auto& prop = static_cast<MkldnnRnnLayerBackward0*>(self->cdata.get())->weight3_;
5614:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5615:   return obj.release().ptr();
5616:   END_HANDLE_TH_ERRORS
5617: }
5618: 
5619: static PyObject* THPMkldnnRnnLayerBackward0_result0_getter(THPCppFunction *self, void *_unused) {
5620:   HANDLE_TH_ERRORS
5621:   const auto& prop = static_cast<MkldnnRnnLayerBackward0*>(self->cdata.get())->result0_;
5622:   return THPVariable_Wrap(prop.unpack(self->cdata));
5623:   END_HANDLE_TH_ERRORS
5624: }
5625: 
5626: static PyObject* THPMkldnnRnnLayerBackward0_result0_raw_getter(THPCppFunction *self, void *_unused) {
5627:   HANDLE_TH_ERRORS
5628:   const auto& prop = static_cast<MkldnnRnnLayerBackward0*>(self->cdata.get())->result0_;
5629:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5630:   return obj.release().ptr();
5631:   END_HANDLE_TH_ERRORS
5632: }
5633: 
5634: static PyObject* THPMkldnnRnnLayerBackward0_result1_getter(THPCppFunction *self, void *_unused) {
5635:   HANDLE_TH_ERRORS
5636:   const auto& prop = static_cast<MkldnnRnnLayerBackward0*>(self->cdata.get())->result1_;
5637:   return THPVariable_Wrap(prop.unpack(self->cdata));
5638:   END_HANDLE_TH_ERRORS
5639: }
5640: 
5641: static PyObject* THPMkldnnRnnLayerBackward0_result1_raw_getter(THPCppFunction *self, void *_unused) {
5642:   HANDLE_TH_ERRORS
5643:   const auto& prop = static_cast<MkldnnRnnLayerBackward0*>(self->cdata.get())->result1_;
5644:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5645:   return obj.release().ptr();
5646:   END_HANDLE_TH_ERRORS
5647: }
5648: 
5649: static PyObject* THPMkldnnRnnLayerBackward0_result2_getter(THPCppFunction *self, void *_unused) {
5650:   HANDLE_TH_ERRORS
5651:   const auto& prop = static_cast<MkldnnRnnLayerBackward0*>(self->cdata.get())->result2_;
5652:   return THPVariable_Wrap(prop.unpack(self->cdata));
5653:   END_HANDLE_TH_ERRORS
5654: }
5655: 
5656: static PyObject* THPMkldnnRnnLayerBackward0_result2_raw_getter(THPCppFunction *self, void *_unused) {
5657:   HANDLE_TH_ERRORS
5658:   const auto& prop = static_cast<MkldnnRnnLayerBackward0*>(self->cdata.get())->result2_;
5659:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5660:   return obj.release().ptr();
5661:   END_HANDLE_TH_ERRORS
5662: }
5663: 
5664: static PyObject* THPMkldnnRnnLayerBackward0_result3_getter(THPCppFunction *self, void *_unused) {
5665:   HANDLE_TH_ERRORS
5666:   const auto& prop = static_cast<MkldnnRnnLayerBackward0*>(self->cdata.get())->result3_;
5667:   return THPVariable_Wrap(prop.unpack(self->cdata));
5668:   END_HANDLE_TH_ERRORS
5669: }
5670: 
5671: static PyObject* THPMkldnnRnnLayerBackward0_result3_raw_getter(THPCppFunction *self, void *_unused) {
5672:   HANDLE_TH_ERRORS
5673:   const auto& prop = static_cast<MkldnnRnnLayerBackward0*>(self->cdata.get())->result3_;
5674:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5675:   return obj.release().ptr();
5676:   END_HANDLE_TH_ERRORS
5677: }
5678: 
5679: static struct PyGetSetDef MkldnnRnnLayerBackward0_properties[] = {
5680:   THP_FUNCTION_DEFAULT_PROPERTIES,
```

- EN: The main execution path in this span is carried by `THPMkldnnRnnLayerBackward0_weight3_getter`, `THPVariable_Wrap`, `THPMkldnnRnnLayerBackward0_weight3_raw_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPMkldnnRnnLayerBackward0_weight3_getter`, `THPVariable_Wrap`, `THPMkldnnRnnLayerBackward0_weight3_raw_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5681-5760

```cpp
5681:   {(char*)"_saved_batch_first", (getter)THPMkldnnRnnLayerBackward0_batch_first_getter, nullptr, nullptr, nullptr},
5682:   {(char*)"_saved_batch_sizes", (getter)THPMkldnnRnnLayerBackward0_batch_sizes_getter, nullptr, nullptr, nullptr},
5683:   {(char*)"_saved_bidirectional", (getter)THPMkldnnRnnLayerBackward0_bidirectional_getter, nullptr, nullptr, nullptr},
5684:   {(char*)"_saved_cx_", (getter)THPMkldnnRnnLayerBackward0_cx__getter, nullptr, nullptr, nullptr},
5685:   {(char*)"_raw_saved_cx_", (getter)THPMkldnnRnnLayerBackward0_cx__raw_getter, nullptr, nullptr, nullptr},
5686:   {(char*)"_saved_has_biases", (getter)THPMkldnnRnnLayerBackward0_has_biases_getter, nullptr, nullptr, nullptr},
5687:   {(char*)"_saved_hidden_size", (getter)THPMkldnnRnnLayerBackward0_hidden_size_getter, nullptr, nullptr, nullptr},
5688:   {(char*)"_saved_hx_", (getter)THPMkldnnRnnLayerBackward0_hx__getter, nullptr, nullptr, nullptr},
5689:   {(char*)"_raw_saved_hx_", (getter)THPMkldnnRnnLayerBackward0_hx__raw_getter, nullptr, nullptr, nullptr},
5690:   {(char*)"_saved_input", (getter)THPMkldnnRnnLayerBackward0_input_getter, nullptr, nullptr, nullptr},
5691:   {(char*)"_raw_saved_input", (getter)THPMkldnnRnnLayerBackward0_input_raw_getter, nullptr, nullptr, nullptr},
5692:   {(char*)"_saved_mode", (getter)THPMkldnnRnnLayerBackward0_mode_getter, nullptr, nullptr, nullptr},
5693:   {(char*)"_saved_num_layers", (getter)THPMkldnnRnnLayerBackward0_num_layers_getter, nullptr, nullptr, nullptr},
5694:   {(char*)"_saved_reverse", (getter)THPMkldnnRnnLayerBackward0_reverse_getter, nullptr, nullptr, nullptr},
5695:   {(char*)"_saved_train", (getter)THPMkldnnRnnLayerBackward0_train_getter, nullptr, nullptr, nullptr},
5696:   {(char*)"_saved_weight0", (getter)THPMkldnnRnnLayerBackward0_weight0_getter, nullptr, nullptr, nullptr},
5697:   {(char*)"_raw_saved_weight0", (getter)THPMkldnnRnnLayerBackward0_weight0_raw_getter, nullptr, nullptr, nullptr},
5698:   {(char*)"_saved_weight1", (getter)THPMkldnnRnnLayerBackward0_weight1_getter, nullptr, nullptr, nullptr},
5699:   {(char*)"_raw_saved_weight1", (getter)THPMkldnnRnnLayerBackward0_weight1_raw_getter, nullptr, nullptr, nullptr},
5700:   {(char*)"_saved_weight2", (getter)THPMkldnnRnnLayerBackward0_weight2_getter, nullptr, nullptr, nullptr},
5701:   {(char*)"_raw_saved_weight2", (getter)THPMkldnnRnnLayerBackward0_weight2_raw_getter, nullptr, nullptr, nullptr},
5702:   {(char*)"_saved_weight3", (getter)THPMkldnnRnnLayerBackward0_weight3_getter, nullptr, nullptr, nullptr},
5703:   {(char*)"_raw_saved_weight3", (getter)THPMkldnnRnnLayerBackward0_weight3_raw_getter, nullptr, nullptr, nullptr},
5704:   {(char*)"_saved_result0", (getter)THPMkldnnRnnLayerBackward0_result0_getter, nullptr, nullptr, nullptr},
5705:   {(char*)"_raw_saved_result0", (getter)THPMkldnnRnnLayerBackward0_result0_raw_getter, nullptr, nullptr, nullptr},
5706:   {(char*)"_saved_result1", (getter)THPMkldnnRnnLayerBackward0_result1_getter, nullptr, nullptr, nullptr},
5707:   {(char*)"_raw_saved_result1", (getter)THPMkldnnRnnLayerBackward0_result1_raw_getter, nullptr, nullptr, nullptr},
5708:   {(char*)"_saved_result2", (getter)THPMkldnnRnnLayerBackward0_result2_getter, nullptr, nullptr, nullptr},
5709:   {(char*)"_raw_saved_result2", (getter)THPMkldnnRnnLayerBackward0_result2_raw_getter, nullptr, nullptr, nullptr},
5710:   {(char*)"_saved_result3", (getter)THPMkldnnRnnLayerBackward0_result3_getter, nullptr, nullptr, nullptr},
5711:   {(char*)"_raw_saved_result3", (getter)THPMkldnnRnnLayerBackward0_result3_raw_getter, nullptr, nullptr, nullptr},
5712:   {nullptr} /* sentinel */
5713: };
5714: 
5715: static PyObject* THPMkldnnMaxPool3DBackward0_ceil_mode_getter(THPCppFunction *self, void *_unused) {
5716:   HANDLE_TH_ERRORS
5717:   auto prop = static_cast<MkldnnMaxPool3DBackward0*>(self->cdata.get())->ceil_mode;
5718:   if (prop) {
5719:     Py_RETURN_TRUE;
5720:   } else {
5721:     Py_RETURN_FALSE;
5722:   }
5723:   END_HANDLE_TH_ERRORS
5724: }
5725: 
5726: static PyObject* THPMkldnnMaxPool3DBackward0_dilation_getter(THPCppFunction *self, void *_unused) {
5727:   HANDLE_TH_ERRORS
5728:   auto prop = static_cast<MkldnnMaxPool3DBackward0*>(self->cdata.get())->dilation;
5729:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5730:   for (auto i : c10::irange(prop.size())) {
5731:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
5732:   }
5733:   return tup;
5734:   END_HANDLE_TH_ERRORS
5735: }
5736: 
5737: static PyObject* THPMkldnnMaxPool3DBackward0_kernel_size_getter(THPCppFunction *self, void *_unused) {
5738:   HANDLE_TH_ERRORS
5739:   auto prop = static_cast<MkldnnMaxPool3DBackward0*>(self->cdata.get())->kernel_size;
5740:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5741:   for (auto i : c10::irange(prop.size())) {
5742:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
5743:   }
5744:   return tup;
5745:   END_HANDLE_TH_ERRORS
5746: }
5747: 
5748: static PyObject* THPMkldnnMaxPool3DBackward0_padding_getter(THPCppFunction *self, void *_unused) {
5749:   HANDLE_TH_ERRORS
5750:   auto prop = static_cast<MkldnnMaxPool3DBackward0*>(self->cdata.get())->padding;
5751:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5752:   for (auto i : c10::irange(prop.size())) {
5753:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
5754:   }
5755:   return tup;
5756:   END_HANDLE_TH_ERRORS
5757: }
5758: 
5759: static PyObject* THPMkldnnMaxPool3DBackward0_self_getter(THPCppFunction *self, void *_unused) {
5760:   HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPMkldnnMaxPool3DBackward0_ceil_mode_getter`, `THPMkldnnMaxPool3DBackward0_dilation_getter`, `PyTuple_New`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPMkldnnMaxPool3DBackward0_ceil_mode_getter`, `THPMkldnnMaxPool3DBackward0_dilation_getter`, `PyTuple_New` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 5761-5840

```cpp
5761:   const auto& prop = static_cast<MkldnnMaxPool3DBackward0*>(self->cdata.get())->self_;
5762:   return THPVariable_Wrap(prop.unpack(self->cdata));
5763:   END_HANDLE_TH_ERRORS
5764: }
5765: 
5766: static PyObject* THPMkldnnMaxPool3DBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
5767:   HANDLE_TH_ERRORS
5768:   const auto& prop = static_cast<MkldnnMaxPool3DBackward0*>(self->cdata.get())->self_;
5769:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5770:   return obj.release().ptr();
5771:   END_HANDLE_TH_ERRORS
5772: }
5773: 
5774: static PyObject* THPMkldnnMaxPool3DBackward0_stride_getter(THPCppFunction *self, void *_unused) {
5775:   HANDLE_TH_ERRORS
5776:   auto prop = static_cast<MkldnnMaxPool3DBackward0*>(self->cdata.get())->stride;
5777:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5778:   for (auto i : c10::irange(prop.size())) {
5779:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
5780:   }
5781:   return tup;
5782:   END_HANDLE_TH_ERRORS
5783: }
5784: 
5785: static PyObject* THPMkldnnMaxPool3DBackward0_result_getter(THPCppFunction *self, void *_unused) {
5786:   HANDLE_TH_ERRORS
5787:   const auto& prop = static_cast<MkldnnMaxPool3DBackward0*>(self->cdata.get())->result_;
5788:   return THPVariable_Wrap(prop.unpack(self->cdata));
5789:   END_HANDLE_TH_ERRORS
5790: }
5791: 
5792: static PyObject* THPMkldnnMaxPool3DBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
5793:   HANDLE_TH_ERRORS
5794:   const auto& prop = static_cast<MkldnnMaxPool3DBackward0*>(self->cdata.get())->result_;
5795:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5796:   return obj.release().ptr();
5797:   END_HANDLE_TH_ERRORS
5798: }
5799: 
5800: static struct PyGetSetDef MkldnnMaxPool3DBackward0_properties[] = {
5801:   THP_FUNCTION_DEFAULT_PROPERTIES,
5802:   {(char*)"_saved_ceil_mode", (getter)THPMkldnnMaxPool3DBackward0_ceil_mode_getter, nullptr, nullptr, nullptr},
5803:   {(char*)"_saved_dilation", (getter)THPMkldnnMaxPool3DBackward0_dilation_getter, nullptr, nullptr, nullptr},
5804:   {(char*)"_saved_kernel_size", (getter)THPMkldnnMaxPool3DBackward0_kernel_size_getter, nullptr, nullptr, nullptr},
5805:   {(char*)"_saved_padding", (getter)THPMkldnnMaxPool3DBackward0_padding_getter, nullptr, nullptr, nullptr},
5806:   {(char*)"_saved_self", (getter)THPMkldnnMaxPool3DBackward0_self_getter, nullptr, nullptr, nullptr},
5807:   {(char*)"_raw_saved_self", (getter)THPMkldnnMaxPool3DBackward0_self_raw_getter, nullptr, nullptr, nullptr},
5808:   {(char*)"_saved_stride", (getter)THPMkldnnMaxPool3DBackward0_stride_getter, nullptr, nullptr, nullptr},
5809:   {(char*)"_saved_result", (getter)THPMkldnnMaxPool3DBackward0_result_getter, nullptr, nullptr, nullptr},
5810:   {(char*)"_raw_saved_result", (getter)THPMkldnnMaxPool3DBackward0_result_raw_getter, nullptr, nullptr, nullptr},
5811:   {nullptr} /* sentinel */
5812: };
5813: 
5814: static PyObject* THPMkldnnReshapeBackward0_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
5815:   HANDLE_TH_ERRORS
5816:   auto prop = static_cast<MkldnnReshapeBackward0*>(self->cdata.get())->self_sym_sizes;
5817:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5818:   for (auto i : c10::irange(prop.size())) {
5819:       auto si = prop[i];
5820:       if (auto m = si.maybe_as_int()) {
5821:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
5822:       } else {
5823:         auto py_symint = py::cast(si).release().ptr();
5824:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
5825:       }
5826:   }
5827:   return tup;
5828:   END_HANDLE_TH_ERRORS
5829: }
5830: 
5831: static struct PyGetSetDef MkldnnReshapeBackward0_properties[] = {
5832:   THP_FUNCTION_DEFAULT_PROPERTIES,
5833:   {(char*)"_saved_self_sym_sizes", (getter)THPMkldnnReshapeBackward0_self_sym_sizes_getter, nullptr, nullptr, nullptr},
5834:   {nullptr} /* sentinel */
5835: };
5836: 
5837: static PyObject* THPNestedTensorFromTensorListBackward0_list_getter(THPCppFunction *self, void *_unused) {
5838:   HANDLE_TH_ERRORS
5839:   const auto *node = static_cast<NestedTensorFromTensorListBackward0*>(self->cdata.get());
5840:   const auto& prop = node->list_;
```

- EN: The main execution path in this span is carried by `THPVariable_Wrap`, `THPMkldnnMaxPool3DBackward0_self_raw_getter`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_Wrap`, `THPMkldnnMaxPool3DBackward0_self_raw_getter`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 5841-5920

```cpp
5841:   if (node->list_released_) {
5842:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
5843:     return nullptr;
5844:   }
5845:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5846:   for (auto i: c10::irange(prop.size())) {
5847:     PyTuple_SetItem(tup, (Py_ssize_t) i, THPVariable_Wrap(prop[i].unpack(self->cdata)));
5848:   }
5849:   return tup;
5850:   END_HANDLE_TH_ERRORS
5851: }
5852: 
5853: static PyObject* THPNestedTensorFromTensorListBackward0_list_raw_getter(THPCppFunction *self, void *_unused) {
5854:   HANDLE_TH_ERRORS
5855:   const auto *node = static_cast<NestedTensorFromTensorListBackward0*>(self->cdata.get());
5856:   const auto& prop = node->list_;
5857:   if (node->list_released_) {
5858:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
5859:     return nullptr;
5860:   }
5861:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5862:   for (auto i : c10::irange(prop.size())) {
5863:     pybind11::object obj = pybind11::cast(prop[i], pybind11::return_value_policy::reference);
5864:     PyTuple_SetItem(tup, (Py_ssize_t) i, obj.release().ptr());
5865:   }
5866:   return tup;
5867:   END_HANDLE_TH_ERRORS
5868: }
5869: 
5870: static struct PyGetSetDef NestedTensorFromTensorListBackward0_properties[] = {
5871:   THP_FUNCTION_DEFAULT_PROPERTIES,
5872:   {(char*)"_saved_list", (getter)THPNestedTensorFromTensorListBackward0_list_getter, nullptr, nullptr, nullptr},
5873:   {(char*)"_raw_saved_list", (getter)THPNestedTensorFromTensorListBackward0_list_raw_getter, nullptr, nullptr, nullptr},
5874:   {nullptr} /* sentinel */
5875: };
5876: 
5877: static PyObject* THPScaledDotProductCudnnAttentionBackward0_attn_bias_getter(THPCppFunction *self, void *_unused) {
5878:   HANDLE_TH_ERRORS
5879:   const auto& prop = static_cast<ScaledDotProductCudnnAttentionBackward0*>(self->cdata.get())->attn_bias_;
5880:   return THPVariable_Wrap(prop.unpack(self->cdata));
5881:   END_HANDLE_TH_ERRORS
5882: }
5883: 
5884: static PyObject* THPScaledDotProductCudnnAttentionBackward0_attn_bias_raw_getter(THPCppFunction *self, void *_unused) {
5885:   HANDLE_TH_ERRORS
5886:   const auto& prop = static_cast<ScaledDotProductCudnnAttentionBackward0*>(self->cdata.get())->attn_bias_;
5887:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5888:   return obj.release().ptr();
5889:   END_HANDLE_TH_ERRORS
5890: }
5891: 
5892: static PyObject* THPScaledDotProductCudnnAttentionBackward0_dropout_p_getter(THPCppFunction *self, void *_unused) {
5893:   HANDLE_TH_ERRORS
5894:   auto prop = static_cast<ScaledDotProductCudnnAttentionBackward0*>(self->cdata.get())->dropout_p;
5895:   return PyFloat_FromDouble((double) prop);
5896:   END_HANDLE_TH_ERRORS
5897: }
5898: 
5899: static PyObject* THPScaledDotProductCudnnAttentionBackward0_is_causal_getter(THPCppFunction *self, void *_unused) {
5900:   HANDLE_TH_ERRORS
5901:   auto prop = static_cast<ScaledDotProductCudnnAttentionBackward0*>(self->cdata.get())->is_causal;
5902:   if (prop) {
5903:     Py_RETURN_TRUE;
5904:   } else {
5905:     Py_RETURN_FALSE;
5906:   }
5907:   END_HANDLE_TH_ERRORS
5908: }
5909: 
5910: static PyObject* THPScaledDotProductCudnnAttentionBackward0_key_getter(THPCppFunction *self, void *_unused) {
5911:   HANDLE_TH_ERRORS
5912:   const auto& prop = static_cast<ScaledDotProductCudnnAttentionBackward0*>(self->cdata.get())->key_;
5913:   return THPVariable_Wrap(prop.unpack(self->cdata));
5914:   END_HANDLE_TH_ERRORS
5915: }
5916: 
5917: static PyObject* THPScaledDotProductCudnnAttentionBackward0_key_raw_getter(THPCppFunction *self, void *_unused) {
5918:   HANDLE_TH_ERRORS
5919:   const auto& prop = static_cast<ScaledDotProductCudnnAttentionBackward0*>(self->cdata.get())->key_;
5920:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
```

- EN: The main execution path in this span is carried by `PyErr_SetString`, `PyTuple_New`, `PyTuple_SetItem`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyErr_SetString`, `PyTuple_New`, `PyTuple_SetItem` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5921-6000

```cpp
5921:   return obj.release().ptr();
5922:   END_HANDLE_TH_ERRORS
5923: }
5924: 
5925: static PyObject* THPScaledDotProductCudnnAttentionBackward0_query_getter(THPCppFunction *self, void *_unused) {
5926:   HANDLE_TH_ERRORS
5927:   const auto& prop = static_cast<ScaledDotProductCudnnAttentionBackward0*>(self->cdata.get())->query_;
5928:   return THPVariable_Wrap(prop.unpack(self->cdata));
5929:   END_HANDLE_TH_ERRORS
5930: }
5931: 
5932: static PyObject* THPScaledDotProductCudnnAttentionBackward0_query_raw_getter(THPCppFunction *self, void *_unused) {
5933:   HANDLE_TH_ERRORS
5934:   const auto& prop = static_cast<ScaledDotProductCudnnAttentionBackward0*>(self->cdata.get())->query_;
5935:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5936:   return obj.release().ptr();
5937:   END_HANDLE_TH_ERRORS
5938: }
5939: 
5940: static PyObject* THPScaledDotProductCudnnAttentionBackward0_scale_getter(THPCppFunction *self, void *_unused) {
5941:   HANDLE_TH_ERRORS
5942:   auto opt_prop = static_cast<ScaledDotProductCudnnAttentionBackward0*>(self->cdata.get())->scale;
5943:   if (!opt_prop.has_value()) {
5944:     Py_RETURN_NONE;
5945:   }
5946:   auto prop = opt_prop.value();
5947:   return PyFloat_FromDouble((double) prop);
5948:   END_HANDLE_TH_ERRORS
5949: }
5950: 
5951: static PyObject* THPScaledDotProductCudnnAttentionBackward0_value_getter(THPCppFunction *self, void *_unused) {
5952:   HANDLE_TH_ERRORS
5953:   const auto& prop = static_cast<ScaledDotProductCudnnAttentionBackward0*>(self->cdata.get())->value_;
5954:   return THPVariable_Wrap(prop.unpack(self->cdata));
5955:   END_HANDLE_TH_ERRORS
5956: }
5957: 
5958: static PyObject* THPScaledDotProductCudnnAttentionBackward0_value_raw_getter(THPCppFunction *self, void *_unused) {
5959:   HANDLE_TH_ERRORS
5960:   const auto& prop = static_cast<ScaledDotProductCudnnAttentionBackward0*>(self->cdata.get())->value_;
5961:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5962:   return obj.release().ptr();
5963:   END_HANDLE_TH_ERRORS
5964: }
5965: 
5966: static PyObject* THPScaledDotProductCudnnAttentionBackward0_cum_seq_k_getter(THPCppFunction *self, void *_unused) {
5967:   HANDLE_TH_ERRORS
5968:   const auto& prop = static_cast<ScaledDotProductCudnnAttentionBackward0*>(self->cdata.get())->cum_seq_k_;
5969:   return THPVariable_Wrap(prop.unpack(self->cdata));
5970:   END_HANDLE_TH_ERRORS
5971: }
5972: 
5973: static PyObject* THPScaledDotProductCudnnAttentionBackward0_cum_seq_k_raw_getter(THPCppFunction *self, void *_unused) {
5974:   HANDLE_TH_ERRORS
5975:   const auto& prop = static_cast<ScaledDotProductCudnnAttentionBackward0*>(self->cdata.get())->cum_seq_k_;
5976:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5977:   return obj.release().ptr();
5978:   END_HANDLE_TH_ERRORS
5979: }
5980: 
5981: static PyObject* THPScaledDotProductCudnnAttentionBackward0_cum_seq_q_getter(THPCppFunction *self, void *_unused) {
5982:   HANDLE_TH_ERRORS
5983:   const auto& prop = static_cast<ScaledDotProductCudnnAttentionBackward0*>(self->cdata.get())->cum_seq_q_;
5984:   return THPVariable_Wrap(prop.unpack(self->cdata));
5985:   END_HANDLE_TH_ERRORS
5986: }
5987: 
5988: static PyObject* THPScaledDotProductCudnnAttentionBackward0_cum_seq_q_raw_getter(THPCppFunction *self, void *_unused) {
5989:   HANDLE_TH_ERRORS
5990:   const auto& prop = static_cast<ScaledDotProductCudnnAttentionBackward0*>(self->cdata.get())->cum_seq_q_;
5991:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5992:   return obj.release().ptr();
5993:   END_HANDLE_TH_ERRORS
5994: }
5995: 
5996: static PyObject* THPScaledDotProductCudnnAttentionBackward0_logsumexp_getter(THPCppFunction *self, void *_unused) {
5997:   HANDLE_TH_ERRORS
5998:   const auto& prop = static_cast<ScaledDotProductCudnnAttentionBackward0*>(self->cdata.get())->logsumexp_;
5999:   return THPVariable_Wrap(prop.unpack(self->cdata));
6000:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPScaledDotProductCudnnAttentionBackward0_query_getter`, `THPVariable_Wrap`, `THPScaledDotProductCudnnAttentionBackward0_query_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPScaledDotProductCudnnAttentionBackward0_query_getter`, `THPVariable_Wrap`, `THPScaledDotProductCudnnAttentionBackward0_query_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 6001-6080

```cpp
6001: }
6002: 
6003: static PyObject* THPScaledDotProductCudnnAttentionBackward0_logsumexp_raw_getter(THPCppFunction *self, void *_unused) {
6004:   HANDLE_TH_ERRORS
6005:   const auto& prop = static_cast<ScaledDotProductCudnnAttentionBackward0*>(self->cdata.get())->logsumexp_;
6006:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
6007:   return obj.release().ptr();
6008:   END_HANDLE_TH_ERRORS
6009: }
6010: 
6011: static PyObject* THPScaledDotProductCudnnAttentionBackward0_max_k_getter(THPCppFunction *self, void *_unused) {
6012:   HANDLE_TH_ERRORS
6013:   auto prop = static_cast<ScaledDotProductCudnnAttentionBackward0*>(self->cdata.get())->max_k;
6014:   if (auto m = prop.maybe_as_int()) {
6015:     return PyLong_FromUnsignedLong(*m);
6016:   } else {
6017:     return py::cast(prop).release().ptr();
6018:   }
6019:   END_HANDLE_TH_ERRORS
6020: }
6021: 
6022: static PyObject* THPScaledDotProductCudnnAttentionBackward0_max_q_getter(THPCppFunction *self, void *_unused) {
6023:   HANDLE_TH_ERRORS
6024:   auto prop = static_cast<ScaledDotProductCudnnAttentionBackward0*>(self->cdata.get())->max_q;
6025:   if (auto m = prop.maybe_as_int()) {
6026:     return PyLong_FromUnsignedLong(*m);
6027:   } else {
6028:     return py::cast(prop).release().ptr();
6029:   }
6030:   END_HANDLE_TH_ERRORS
6031: }
6032: 
6033: static PyObject* THPScaledDotProductCudnnAttentionBackward0_output_getter(THPCppFunction *self, void *_unused) {
6034:   HANDLE_TH_ERRORS
6035:   const auto& prop = static_cast<ScaledDotProductCudnnAttentionBackward0*>(self->cdata.get())->output_;
6036:   return THPVariable_Wrap(prop.unpack(self->cdata));
6037:   END_HANDLE_TH_ERRORS
6038: }
6039: 
6040: static PyObject* THPScaledDotProductCudnnAttentionBackward0_output_raw_getter(THPCppFunction *self, void *_unused) {
6041:   HANDLE_TH_ERRORS
6042:   const auto& prop = static_cast<ScaledDotProductCudnnAttentionBackward0*>(self->cdata.get())->output_;
6043:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
6044:   return obj.release().ptr();
6045:   END_HANDLE_TH_ERRORS
6046: }
6047: 
6048: static PyObject* THPScaledDotProductCudnnAttentionBackward0_philox_offset_getter(THPCppFunction *self, void *_unused) {
6049:   HANDLE_TH_ERRORS
6050:   const auto& prop = static_cast<ScaledDotProductCudnnAttentionBackward0*>(self->cdata.get())->philox_offset_;
6051:   return THPVariable_Wrap(prop.unpack(self->cdata));
6052:   END_HANDLE_TH_ERRORS
6053: }
6054: 
6055: static PyObject* THPScaledDotProductCudnnAttentionBackward0_philox_offset_raw_getter(THPCppFunction *self, void *_unused) {
6056:   HANDLE_TH_ERRORS
6057:   const auto& prop = static_cast<ScaledDotProductCudnnAttentionBackward0*>(self->cdata.get())->philox_offset_;
6058:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
6059:   return obj.release().ptr();
6060:   END_HANDLE_TH_ERRORS
6061: }
6062: 
6063: static PyObject* THPScaledDotProductCudnnAttentionBackward0_philox_seed_getter(THPCppFunction *self, void *_unused) {
6064:   HANDLE_TH_ERRORS
6065:   const auto& prop = static_cast<ScaledDotProductCudnnAttentionBackward0*>(self->cdata.get())->philox_seed_;
6066:   return THPVariable_Wrap(prop.unpack(self->cdata));
6067:   END_HANDLE_TH_ERRORS
6068: }
6069: 
6070: static PyObject* THPScaledDotProductCudnnAttentionBackward0_philox_seed_raw_getter(THPCppFunction *self, void *_unused) {
6071:   HANDLE_TH_ERRORS
6072:   const auto& prop = static_cast<ScaledDotProductCudnnAttentionBackward0*>(self->cdata.get())->philox_seed_;
6073:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
6074:   return obj.release().ptr();
6075:   END_HANDLE_TH_ERRORS
6076: }
6077: 
6078: static struct PyGetSetDef ScaledDotProductCudnnAttentionBackward0_properties[] = {
6079:   THP_FUNCTION_DEFAULT_PROPERTIES,
6080:   {(char*)"_saved_attn_bias", (getter)THPScaledDotProductCudnnAttentionBackward0_attn_bias_getter, nullptr, nullptr, nullptr},
```

- EN: The main execution path in this span is carried by `THPScaledDotProductCudnnAttentionBackward0_logsumexp_raw_getter`, `cast`, `THPScaledDotProductCudnnAttentionBackward0_max_k_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPScaledDotProductCudnnAttentionBackward0_logsumexp_raw_getter`, `cast`, `THPScaledDotProductCudnnAttentionBackward0_max_k_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 6081-6160

```cpp
6081:   {(char*)"_raw_saved_attn_bias", (getter)THPScaledDotProductCudnnAttentionBackward0_attn_bias_raw_getter, nullptr, nullptr, nullptr},
6082:   {(char*)"_saved_dropout_p", (getter)THPScaledDotProductCudnnAttentionBackward0_dropout_p_getter, nullptr, nullptr, nullptr},
6083:   {(char*)"_saved_is_causal", (getter)THPScaledDotProductCudnnAttentionBackward0_is_causal_getter, nullptr, nullptr, nullptr},
6084:   {(char*)"_saved_key", (getter)THPScaledDotProductCudnnAttentionBackward0_key_getter, nullptr, nullptr, nullptr},
6085:   {(char*)"_raw_saved_key", (getter)THPScaledDotProductCudnnAttentionBackward0_key_raw_getter, nullptr, nullptr, nullptr},
6086:   {(char*)"_saved_query", (getter)THPScaledDotProductCudnnAttentionBackward0_query_getter, nullptr, nullptr, nullptr},
6087:   {(char*)"_raw_saved_query", (getter)THPScaledDotProductCudnnAttentionBackward0_query_raw_getter, nullptr, nullptr, nullptr},
6088:   {(char*)"_saved_scale", (getter)THPScaledDotProductCudnnAttentionBackward0_scale_getter, nullptr, nullptr, nullptr},
6089:   {(char*)"_saved_value", (getter)THPScaledDotProductCudnnAttentionBackward0_value_getter, nullptr, nullptr, nullptr},
6090:   {(char*)"_raw_saved_value", (getter)THPScaledDotProductCudnnAttentionBackward0_value_raw_getter, nullptr, nullptr, nullptr},
6091:   {(char*)"_saved_cum_seq_k", (getter)THPScaledDotProductCudnnAttentionBackward0_cum_seq_k_getter, nullptr, nullptr, nullptr},
6092:   {(char*)"_raw_saved_cum_seq_k", (getter)THPScaledDotProductCudnnAttentionBackward0_cum_seq_k_raw_getter, nullptr, nullptr, nullptr},
6093:   {(char*)"_saved_cum_seq_q", (getter)THPScaledDotProductCudnnAttentionBackward0_cum_seq_q_getter, nullptr, nullptr, nullptr},
6094:   {(char*)"_raw_saved_cum_seq_q", (getter)THPScaledDotProductCudnnAttentionBackward0_cum_seq_q_raw_getter, nullptr, nullptr, nullptr},
6095:   {(char*)"_saved_logsumexp", (getter)THPScaledDotProductCudnnAttentionBackward0_logsumexp_getter, nullptr, nullptr, nullptr},
6096:   {(char*)"_raw_saved_logsumexp", (getter)THPScaledDotProductCudnnAttentionBackward0_logsumexp_raw_getter, nullptr, nullptr, nullptr},
6097:   {(char*)"_saved_max_k", (getter)THPScaledDotProductCudnnAttentionBackward0_max_k_getter, nullptr, nullptr, nullptr},
6098:   {(char*)"_saved_max_q", (getter)THPScaledDotProductCudnnAttentionBackward0_max_q_getter, nullptr, nullptr, nullptr},
6099:   {(char*)"_saved_output", (getter)THPScaledDotProductCudnnAttentionBackward0_output_getter, nullptr, nullptr, nullptr},
6100:   {(char*)"_raw_saved_output", (getter)THPScaledDotProductCudnnAttentionBackward0_output_raw_getter, nullptr, nullptr, nullptr},
6101:   {(char*)"_saved_philox_offset", (getter)THPScaledDotProductCudnnAttentionBackward0_philox_offset_getter, nullptr, nullptr, nullptr},
6102:   {(char*)"_raw_saved_philox_offset", (getter)THPScaledDotProductCudnnAttentionBackward0_philox_offset_raw_getter, nullptr, nullptr, nullptr},
6103:   {(char*)"_saved_philox_seed", (getter)THPScaledDotProductCudnnAttentionBackward0_philox_seed_getter, nullptr, nullptr, nullptr},
6104:   {(char*)"_raw_saved_philox_seed", (getter)THPScaledDotProductCudnnAttentionBackward0_philox_seed_raw_getter, nullptr, nullptr, nullptr},
6105:   {nullptr} /* sentinel */
6106: };
6107: 
6108: static PyObject* THPScatterReduceBackward0_dim_getter(THPCppFunction *self, void *_unused) {
6109:   HANDLE_TH_ERRORS
6110:   auto prop = static_cast<ScatterReduceBackward0*>(self->cdata.get())->dim;
6111:   return PyLong_FromUnsignedLong((int64_t) prop);
6112:   END_HANDLE_TH_ERRORS
6113: }
6114: 
6115: static PyObject* THPScatterReduceBackward0_include_self_getter(THPCppFunction *self, void *_unused) {
6116:   HANDLE_TH_ERRORS
6117:   auto prop = static_cast<ScatterReduceBackward0*>(self->cdata.get())->include_self;
6118:   if (prop) {
6119:     Py_RETURN_TRUE;
6120:   } else {
6121:     Py_RETURN_FALSE;
6122:   }
6123:   END_HANDLE_TH_ERRORS
6124: }
6125: 
6126: static PyObject* THPScatterReduceBackward0_index_getter(THPCppFunction *self, void *_unused) {
6127:   HANDLE_TH_ERRORS
6128:   const auto& prop = static_cast<ScatterReduceBackward0*>(self->cdata.get())->index_;
6129:   return THPVariable_Wrap(prop.unpack(self->cdata));
6130:   END_HANDLE_TH_ERRORS
6131: }
6132: 
6133: static PyObject* THPScatterReduceBackward0_index_raw_getter(THPCppFunction *self, void *_unused) {
6134:   HANDLE_TH_ERRORS
6135:   const auto& prop = static_cast<ScatterReduceBackward0*>(self->cdata.get())->index_;
6136:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
6137:   return obj.release().ptr();
6138:   END_HANDLE_TH_ERRORS
6139: }
6140: 
6141: static PyObject* THPScatterReduceBackward0_reduce_getter(THPCppFunction *self, void *_unused) {
6142:   HANDLE_TH_ERRORS
6143:   auto prop = static_cast<ScatterReduceBackward0*>(self->cdata.get())->reduce;
6144:   return PyUnicode_FromStringAndSize(prop.data(), prop.size());
6145:   END_HANDLE_TH_ERRORS
6146: }
6147: 
6148: static PyObject* THPScatterReduceBackward0_self_getter(THPCppFunction *self, void *_unused) {
6149:   HANDLE_TH_ERRORS
6150:   const auto& prop = static_cast<ScatterReduceBackward0*>(self->cdata.get())->self_;
6151:   return THPVariable_Wrap(prop.unpack(self->cdata));
6152:   END_HANDLE_TH_ERRORS
6153: }
6154: 
6155: static PyObject* THPScatterReduceBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
6156:   HANDLE_TH_ERRORS
6157:   const auto& prop = static_cast<ScatterReduceBackward0*>(self->cdata.get())->self_;
6158:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
6159:   return obj.release().ptr();
6160:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPScatterReduceBackward0_dim_getter`, `PyLong_FromUnsignedLong`, `THPScatterReduceBackward0_include_self_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPScatterReduceBackward0_dim_getter`, `PyLong_FromUnsignedLong`, `THPScatterReduceBackward0_include_self_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 6161-6240

```cpp
6161: }
6162: 
6163: static PyObject* THPScatterReduceBackward0_src_getter(THPCppFunction *self, void *_unused) {
6164:   HANDLE_TH_ERRORS
6165:   const auto& prop = static_cast<ScatterReduceBackward0*>(self->cdata.get())->src_;
6166:   return THPVariable_Wrap(prop.unpack(self->cdata));
6167:   END_HANDLE_TH_ERRORS
6168: }
6169: 
6170: static PyObject* THPScatterReduceBackward0_src_raw_getter(THPCppFunction *self, void *_unused) {
6171:   HANDLE_TH_ERRORS
6172:   const auto& prop = static_cast<ScatterReduceBackward0*>(self->cdata.get())->src_;
6173:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
6174:   return obj.release().ptr();
6175:   END_HANDLE_TH_ERRORS
6176: }
6177: 
6178: static PyObject* THPScatterReduceBackward0_result_getter(THPCppFunction *self, void *_unused) {
6179:   HANDLE_TH_ERRORS
6180:   const auto& prop = static_cast<ScatterReduceBackward0*>(self->cdata.get())->result_;
6181:   return THPVariable_Wrap(prop.unpack(self->cdata));
6182:   END_HANDLE_TH_ERRORS
6183: }
6184: 
6185: static PyObject* THPScatterReduceBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
6186:   HANDLE_TH_ERRORS
6187:   const auto& prop = static_cast<ScatterReduceBackward0*>(self->cdata.get())->result_;
6188:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
6189:   return obj.release().ptr();
6190:   END_HANDLE_TH_ERRORS
6191: }
6192: 
6193: static struct PyGetSetDef ScatterReduceBackward0_properties[] = {
6194:   THP_FUNCTION_DEFAULT_PROPERTIES,
6195:   {(char*)"_saved_dim", (getter)THPScatterReduceBackward0_dim_getter, nullptr, nullptr, nullptr},
6196:   {(char*)"_saved_include_self", (getter)THPScatterReduceBackward0_include_self_getter, nullptr, nullptr, nullptr},
6197:   {(char*)"_saved_index", (getter)THPScatterReduceBackward0_index_getter, nullptr, nullptr, nullptr},
6198:   {(char*)"_raw_saved_index", (getter)THPScatterReduceBackward0_index_raw_getter, nullptr, nullptr, nullptr},
6199:   {(char*)"_saved_reduce", (getter)THPScatterReduceBackward0_reduce_getter, nullptr, nullptr, nullptr},
6200:   {(char*)"_saved_self", (getter)THPScatterReduceBackward0_self_getter, nullptr, nullptr, nullptr},
6201:   {(char*)"_raw_saved_self", (getter)THPScatterReduceBackward0_self_raw_getter, nullptr, nullptr, nullptr},
6202:   {(char*)"_saved_src", (getter)THPScatterReduceBackward0_src_getter, nullptr, nullptr, nullptr},
6203:   {(char*)"_raw_saved_src", (getter)THPScatterReduceBackward0_src_raw_getter, nullptr, nullptr, nullptr},
6204:   {(char*)"_saved_result", (getter)THPScatterReduceBackward0_result_getter, nullptr, nullptr, nullptr},
6205:   {(char*)"_raw_saved_result", (getter)THPScatterReduceBackward0_result_raw_getter, nullptr, nullptr, nullptr},
6206:   {nullptr} /* sentinel */
6207: };
6208: 
6209: static PyObject* THPForeachPowBackward0_exponent_getter(THPCppFunction *self, void *_unused) {
6210:   HANDLE_TH_ERRORS
6211:   const auto *node = static_cast<ForeachPowBackward0*>(self->cdata.get());
6212:   const auto& prop = node->exponent_;
6213:   if (node->exponent_released_) {
6214:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
6215:     return nullptr;
6216:   }
6217:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
6218:   for (auto i: c10::irange(prop.size())) {
6219:     PyTuple_SetItem(tup, (Py_ssize_t) i, THPVariable_Wrap(prop[i].unpack(self->cdata)));
6220:   }
6221:   return tup;
6222:   END_HANDLE_TH_ERRORS
6223: }
6224: 
6225: static PyObject* THPForeachPowBackward0_exponent_raw_getter(THPCppFunction *self, void *_unused) {
6226:   HANDLE_TH_ERRORS
6227:   const auto *node = static_cast<ForeachPowBackward0*>(self->cdata.get());
6228:   const auto& prop = node->exponent_;
6229:   if (node->exponent_released_) {
6230:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
6231:     return nullptr;
6232:   }
6233:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
6234:   for (auto i : c10::irange(prop.size())) {
6235:     pybind11::object obj = pybind11::cast(prop[i], pybind11::return_value_policy::reference);
6236:     PyTuple_SetItem(tup, (Py_ssize_t) i, obj.release().ptr());
6237:   }
6238:   return tup;
6239:   END_HANDLE_TH_ERRORS
6240: }
```

- EN: The main execution path in this span is carried by `THPScatterReduceBackward0_src_getter`, `THPVariable_Wrap`, `THPScatterReduceBackward0_src_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPScatterReduceBackward0_src_getter`, `THPVariable_Wrap`, `THPScatterReduceBackward0_src_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 6241-6320

```cpp
6241: 
6242: static PyObject* THPForeachPowBackward0_self_getter(THPCppFunction *self, void *_unused) {
6243:   HANDLE_TH_ERRORS
6244:   const auto *node = static_cast<ForeachPowBackward0*>(self->cdata.get());
6245:   const auto& prop = node->self_;
6246:   if (node->self_released_) {
6247:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
6248:     return nullptr;
6249:   }
6250:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
6251:   for (auto i: c10::irange(prop.size())) {
6252:     PyTuple_SetItem(tup, (Py_ssize_t) i, THPVariable_Wrap(prop[i].unpack(self->cdata)));
6253:   }
6254:   return tup;
6255:   END_HANDLE_TH_ERRORS
6256: }
6257: 
6258: static PyObject* THPForeachPowBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
6259:   HANDLE_TH_ERRORS
6260:   const auto *node = static_cast<ForeachPowBackward0*>(self->cdata.get());
6261:   const auto& prop = node->self_;
6262:   if (node->self_released_) {
6263:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
6264:     return nullptr;
6265:   }
6266:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
6267:   for (auto i : c10::irange(prop.size())) {
6268:     pybind11::object obj = pybind11::cast(prop[i], pybind11::return_value_policy::reference);
6269:     PyTuple_SetItem(tup, (Py_ssize_t) i, obj.release().ptr());
6270:   }
6271:   return tup;
6272:   END_HANDLE_TH_ERRORS
6273: }
6274: 
6275: static PyObject* THPForeachPowBackward0_result_getter(THPCppFunction *self, void *_unused) {
6276:   HANDLE_TH_ERRORS
6277:   const auto *node = static_cast<ForeachPowBackward0*>(self->cdata.get());
6278:   const auto& prop = node->result_;
6279:   if (node->result_released_) {
6280:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
6281:     return nullptr;
6282:   }
6283:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
6284:   for (auto i: c10::irange(prop.size())) {
6285:     PyTuple_SetItem(tup, (Py_ssize_t) i, THPVariable_Wrap(prop[i].unpack(self->cdata)));
6286:   }
6287:   return tup;
6288:   END_HANDLE_TH_ERRORS
6289: }
6290: 
6291: static PyObject* THPForeachPowBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
6292:   HANDLE_TH_ERRORS
6293:   const auto *node = static_cast<ForeachPowBackward0*>(self->cdata.get());
6294:   const auto& prop = node->result_;
6295:   if (node->result_released_) {
6296:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
6297:     return nullptr;
6298:   }
6299:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
6300:   for (auto i : c10::irange(prop.size())) {
6301:     pybind11::object obj = pybind11::cast(prop[i], pybind11::return_value_policy::reference);
6302:     PyTuple_SetItem(tup, (Py_ssize_t) i, obj.release().ptr());
6303:   }
6304:   return tup;
6305:   END_HANDLE_TH_ERRORS
6306: }
6307: 
6308: static struct PyGetSetDef ForeachPowBackward0_properties[] = {
6309:   THP_FUNCTION_DEFAULT_PROPERTIES,
6310:   {(char*)"_saved_exponent", (getter)THPForeachPowBackward0_exponent_getter, nullptr, nullptr, nullptr},
6311:   {(char*)"_raw_saved_exponent", (getter)THPForeachPowBackward0_exponent_raw_getter, nullptr, nullptr, nullptr},
6312:   {(char*)"_saved_self", (getter)THPForeachPowBackward0_self_getter, nullptr, nullptr, nullptr},
6313:   {(char*)"_raw_saved_self", (getter)THPForeachPowBackward0_self_raw_getter, nullptr, nullptr, nullptr},
6314:   {(char*)"_saved_result", (getter)THPForeachPowBackward0_result_getter, nullptr, nullptr, nullptr},
6315:   {(char*)"_raw_saved_result", (getter)THPForeachPowBackward0_result_raw_getter, nullptr, nullptr, nullptr},
6316:   {nullptr} /* sentinel */
6317: };
6318: 
6319: static PyObject* THPForeachPowBackward1_exponent_getter(THPCppFunction *self, void *_unused) {
6320:   HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPForeachPowBackward0_self_getter`, `PyErr_SetString`, `PyTuple_New`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPForeachPowBackward0_self_getter`, `PyErr_SetString`, `PyTuple_New` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 6321-6400

```cpp
6321:   const auto *node = static_cast<ForeachPowBackward1*>(self->cdata.get());
6322:   const auto& prop = node->exponent;
6323:   if (node->exponent_released_) {
6324:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
6325:     return nullptr;
6326:   }
6327:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
6328:   for (auto i: c10::irange(prop.size())) {
6329:     if (prop[i].isComplex()) {
6330:       auto cprop = prop[i].to<c10::complex<double>>();
6331:       PyTuple_SetItem(tup, (Py_ssize_t) i, PyComplex_FromDoubles(cprop.real(), cprop.imag()));
6332:     } else if (prop[i].isFloatingPoint()) {
6333:       auto double_prop = prop[i].to<double>();
6334:       PyTuple_SetItem(tup, (Py_ssize_t) i, PyFloat_FromDouble(double_prop));
6335:     } else if (prop[i].isIntegral(/*includeBool=*/false)) {
6336:       auto long_prop = prop[i].to<int64_t>();
6337:       PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromLong(long_prop));
6338:     } else if (prop[i].isBoolean()) {
6339:       if (prop[i].to<bool>()) {
6340:         PyTuple_SetItem(tup, (Py_ssize_t) i, Py_True);
6341:       } else {
6342:         PyTuple_SetItem(tup, (Py_ssize_t) i, Py_False);
6343:       }
6344:     } else {
6345:       PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
6346:       return nullptr;
6347:     }
6348:   }
6349:   return tup;
6350:   END_HANDLE_TH_ERRORS
6351: }
6352: 
6353: static PyObject* THPForeachPowBackward1_self_getter(THPCppFunction *self, void *_unused) {
6354:   HANDLE_TH_ERRORS
6355:   const auto *node = static_cast<ForeachPowBackward1*>(self->cdata.get());
6356:   const auto& prop = node->self_;
6357:   if (node->self_released_) {
6358:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
6359:     return nullptr;
6360:   }
6361:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
6362:   for (auto i: c10::irange(prop.size())) {
6363:     PyTuple_SetItem(tup, (Py_ssize_t) i, THPVariable_Wrap(prop[i].unpack(self->cdata)));
6364:   }
6365:   return tup;
6366:   END_HANDLE_TH_ERRORS
6367: }
6368: 
6369: static PyObject* THPForeachPowBackward1_self_raw_getter(THPCppFunction *self, void *_unused) {
6370:   HANDLE_TH_ERRORS
6371:   const auto *node = static_cast<ForeachPowBackward1*>(self->cdata.get());
6372:   const auto& prop = node->self_;
6373:   if (node->self_released_) {
6374:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
6375:     return nullptr;
6376:   }
6377:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
6378:   for (auto i : c10::irange(prop.size())) {
6379:     pybind11::object obj = pybind11::cast(prop[i], pybind11::return_value_policy::reference);
6380:     PyTuple_SetItem(tup, (Py_ssize_t) i, obj.release().ptr());
6381:   }
6382:   return tup;
6383:   END_HANDLE_TH_ERRORS
6384: }
6385: 
6386: static struct PyGetSetDef ForeachPowBackward1_properties[] = {
6387:   THP_FUNCTION_DEFAULT_PROPERTIES,
6388:   {(char*)"_saved_exponent", (getter)THPForeachPowBackward1_exponent_getter, nullptr, nullptr, nullptr},
6389:   {(char*)"_saved_self", (getter)THPForeachPowBackward1_self_getter, nullptr, nullptr, nullptr},
6390:   {(char*)"_raw_saved_self", (getter)THPForeachPowBackward1_self_raw_getter, nullptr, nullptr, nullptr},
6391:   {nullptr} /* sentinel */
6392: };
6393: 
6394: static PyObject* THPForeachPowBackward2_exponent_getter(THPCppFunction *self, void *_unused) {
6395:   HANDLE_TH_ERRORS
6396:   const auto *node = static_cast<ForeachPowBackward2*>(self->cdata.get());
6397:   const auto& prop = node->exponent_;
6398:   if (node->exponent_released_) {
6399:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
6400:     return nullptr;
```

- EN: The main execution path in this span is carried by `PyErr_SetString`, `PyTuple_New`, `PyTuple_SetItem`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyErr_SetString`, `PyTuple_New`, `PyTuple_SetItem` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 6401-6480

```cpp
6401:   }
6402:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
6403:   for (auto i: c10::irange(prop.size())) {
6404:     PyTuple_SetItem(tup, (Py_ssize_t) i, THPVariable_Wrap(prop[i].unpack(self->cdata)));
6405:   }
6406:   return tup;
6407:   END_HANDLE_TH_ERRORS
6408: }
6409: 
6410: static PyObject* THPForeachPowBackward2_exponent_raw_getter(THPCppFunction *self, void *_unused) {
6411:   HANDLE_TH_ERRORS
6412:   const auto *node = static_cast<ForeachPowBackward2*>(self->cdata.get());
6413:   const auto& prop = node->exponent_;
6414:   if (node->exponent_released_) {
6415:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
6416:     return nullptr;
6417:   }
6418:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
6419:   for (auto i : c10::irange(prop.size())) {
6420:     pybind11::object obj = pybind11::cast(prop[i], pybind11::return_value_policy::reference);
6421:     PyTuple_SetItem(tup, (Py_ssize_t) i, obj.release().ptr());
6422:   }
6423:   return tup;
6424:   END_HANDLE_TH_ERRORS
6425: }
6426: 
6427: static PyObject* THPForeachPowBackward2_self_getter(THPCppFunction *self, void *_unused) {
6428:   HANDLE_TH_ERRORS
6429:   auto prop = static_cast<ForeachPowBackward2*>(self->cdata.get())->self;
6430:   if (prop.isComplex()) {
6431:     auto cprop = prop.to<c10::complex<double>>();
6432:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
6433:   } else if (prop.isFloatingPoint()) {
6434:     return PyFloat_FromDouble(prop.to<double>());
6435:   } else if (prop.isIntegral(/*includeBool=*/false)) {
6436:     return PyLong_FromLong(prop.to<int64_t>());
6437:   } else if (prop.isBoolean()) {
6438:     if (prop.to<bool>()) {
6439:       Py_RETURN_TRUE;
6440:     } else {
6441:       Py_RETURN_FALSE;
6442:     }
6443:   } else {
6444:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
6445:     return nullptr;
6446:   }
6447:   END_HANDLE_TH_ERRORS
6448: }
6449: 
6450: static PyObject* THPForeachPowBackward2_result_getter(THPCppFunction *self, void *_unused) {
6451:   HANDLE_TH_ERRORS
6452:   const auto *node = static_cast<ForeachPowBackward2*>(self->cdata.get());
6453:   const auto& prop = node->result_;
6454:   if (node->result_released_) {
6455:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
6456:     return nullptr;
6457:   }
6458:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
6459:   for (auto i: c10::irange(prop.size())) {
6460:     PyTuple_SetItem(tup, (Py_ssize_t) i, THPVariable_Wrap(prop[i].unpack(self->cdata)));
6461:   }
6462:   return tup;
6463:   END_HANDLE_TH_ERRORS
6464: }
6465: 
6466: static PyObject* THPForeachPowBackward2_result_raw_getter(THPCppFunction *self, void *_unused) {
6467:   HANDLE_TH_ERRORS
6468:   const auto *node = static_cast<ForeachPowBackward2*>(self->cdata.get());
6469:   const auto& prop = node->result_;
6470:   if (node->result_released_) {
6471:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
6472:     return nullptr;
6473:   }
6474:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
6475:   for (auto i : c10::irange(prop.size())) {
6476:     pybind11::object obj = pybind11::cast(prop[i], pybind11::return_value_policy::reference);
6477:     PyTuple_SetItem(tup, (Py_ssize_t) i, obj.release().ptr());
6478:   }
6479:   return tup;
6480:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `PyTuple_New`, `PyTuple_SetItem`, `THPForeachPowBackward2_exponent_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyTuple_New`, `PyTuple_SetItem`, `THPForeachPowBackward2_exponent_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 6481-6560

```cpp
6481: }
6482: 
6483: static struct PyGetSetDef ForeachPowBackward2_properties[] = {
6484:   THP_FUNCTION_DEFAULT_PROPERTIES,
6485:   {(char*)"_saved_exponent", (getter)THPForeachPowBackward2_exponent_getter, nullptr, nullptr, nullptr},
6486:   {(char*)"_raw_saved_exponent", (getter)THPForeachPowBackward2_exponent_raw_getter, nullptr, nullptr, nullptr},
6487:   {(char*)"_saved_self", (getter)THPForeachPowBackward2_self_getter, nullptr, nullptr, nullptr},
6488:   {(char*)"_saved_result", (getter)THPForeachPowBackward2_result_getter, nullptr, nullptr, nullptr},
6489:   {(char*)"_raw_saved_result", (getter)THPForeachPowBackward2_result_raw_getter, nullptr, nullptr, nullptr},
6490:   {nullptr} /* sentinel */
6491: };
6492: 
6493: static PyObject* THPForeachNormBackward0_ord_getter(THPCppFunction *self, void *_unused) {
6494:   HANDLE_TH_ERRORS
6495:   auto prop = static_cast<ForeachNormBackward0*>(self->cdata.get())->ord;
6496:   if (prop.isComplex()) {
6497:     auto cprop = prop.to<c10::complex<double>>();
6498:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
6499:   } else if (prop.isFloatingPoint()) {
6500:     return PyFloat_FromDouble(prop.to<double>());
6501:   } else if (prop.isIntegral(/*includeBool=*/false)) {
6502:     return PyLong_FromLong(prop.to<int64_t>());
6503:   } else if (prop.isBoolean()) {
6504:     if (prop.to<bool>()) {
6505:       Py_RETURN_TRUE;
6506:     } else {
6507:       Py_RETURN_FALSE;
6508:     }
6509:   } else {
6510:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
6511:     return nullptr;
6512:   }
6513:   END_HANDLE_TH_ERRORS
6514: }
6515: 
6516: static PyObject* THPForeachNormBackward0_self_getter(THPCppFunction *self, void *_unused) {
6517:   HANDLE_TH_ERRORS
6518:   const auto *node = static_cast<ForeachNormBackward0*>(self->cdata.get());
6519:   const auto& prop = node->self_;
6520:   if (node->self_released_) {
6521:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
6522:     return nullptr;
6523:   }
6524:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
6525:   for (auto i: c10::irange(prop.size())) {
6526:     PyTuple_SetItem(tup, (Py_ssize_t) i, THPVariable_Wrap(prop[i].unpack(self->cdata)));
6527:   }
6528:   return tup;
6529:   END_HANDLE_TH_ERRORS
6530: }
6531: 
6532: static PyObject* THPForeachNormBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
6533:   HANDLE_TH_ERRORS
6534:   const auto *node = static_cast<ForeachNormBackward0*>(self->cdata.get());
6535:   const auto& prop = node->self_;
6536:   if (node->self_released_) {
6537:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
6538:     return nullptr;
6539:   }
6540:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
6541:   for (auto i : c10::irange(prop.size())) {
6542:     pybind11::object obj = pybind11::cast(prop[i], pybind11::return_value_policy::reference);
6543:     PyTuple_SetItem(tup, (Py_ssize_t) i, obj.release().ptr());
6544:   }
6545:   return tup;
6546:   END_HANDLE_TH_ERRORS
6547: }
6548: 
6549: static PyObject* THPForeachNormBackward0_result_getter(THPCppFunction *self, void *_unused) {
6550:   HANDLE_TH_ERRORS
6551:   const auto *node = static_cast<ForeachNormBackward0*>(self->cdata.get());
6552:   const auto& prop = node->result_;
6553:   if (node->result_released_) {
6554:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
6555:     return nullptr;
6556:   }
6557:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
6558:   for (auto i: c10::irange(prop.size())) {
6559:     PyTuple_SetItem(tup, (Py_ssize_t) i, THPVariable_Wrap(prop[i].unpack(self->cdata)));
6560:   }
```

- EN: The main execution path in this span is carried by `THPForeachNormBackward0_ord_getter`, `PyComplex_FromDoubles`, `PyFloat_FromDouble`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPForeachNormBackward0_ord_getter`, `PyComplex_FromDoubles`, `PyFloat_FromDouble` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 6561-6640

```cpp
6561:   return tup;
6562:   END_HANDLE_TH_ERRORS
6563: }
6564: 
6565: static PyObject* THPForeachNormBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
6566:   HANDLE_TH_ERRORS
6567:   const auto *node = static_cast<ForeachNormBackward0*>(self->cdata.get());
6568:   const auto& prop = node->result_;
6569:   if (node->result_released_) {
6570:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
6571:     return nullptr;
6572:   }
6573:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
6574:   for (auto i : c10::irange(prop.size())) {
6575:     pybind11::object obj = pybind11::cast(prop[i], pybind11::return_value_policy::reference);
6576:     PyTuple_SetItem(tup, (Py_ssize_t) i, obj.release().ptr());
6577:   }
6578:   return tup;
6579:   END_HANDLE_TH_ERRORS
6580: }
6581: 
6582: static struct PyGetSetDef ForeachNormBackward0_properties[] = {
6583:   THP_FUNCTION_DEFAULT_PROPERTIES,
6584:   {(char*)"_saved_ord", (getter)THPForeachNormBackward0_ord_getter, nullptr, nullptr, nullptr},
6585:   {(char*)"_saved_self", (getter)THPForeachNormBackward0_self_getter, nullptr, nullptr, nullptr},
6586:   {(char*)"_raw_saved_self", (getter)THPForeachNormBackward0_self_raw_getter, nullptr, nullptr, nullptr},
6587:   {(char*)"_saved_result", (getter)THPForeachNormBackward0_result_getter, nullptr, nullptr, nullptr},
6588:   {(char*)"_raw_saved_result", (getter)THPForeachNormBackward0_result_raw_getter, nullptr, nullptr, nullptr},
6589:   {nullptr} /* sentinel */
6590: };
6591: 
6592: static PyObject* THPDiagonalBackward0_copy_dim1_getter(THPCppFunction *self, void *_unused) {
6593:   HANDLE_TH_ERRORS
6594:   auto prop = static_cast<DiagonalBackward0_copy*>(self->cdata.get())->dim1;
6595:   return PyLong_FromUnsignedLong((int64_t) prop);
6596:   END_HANDLE_TH_ERRORS
6597: }
6598: 
6599: static PyObject* THPDiagonalBackward0_copy_dim2_getter(THPCppFunction *self, void *_unused) {
6600:   HANDLE_TH_ERRORS
6601:   auto prop = static_cast<DiagonalBackward0_copy*>(self->cdata.get())->dim2;
6602:   return PyLong_FromUnsignedLong((int64_t) prop);
6603:   END_HANDLE_TH_ERRORS
6604: }
6605: 
6606: static PyObject* THPDiagonalBackward0_copy_offset_getter(THPCppFunction *self, void *_unused) {
6607:   HANDLE_TH_ERRORS
6608:   auto prop = static_cast<DiagonalBackward0_copy*>(self->cdata.get())->offset;
6609:   return PyLong_FromUnsignedLong((int64_t) prop);
6610:   END_HANDLE_TH_ERRORS
6611: }
6612: 
6613: static PyObject* THPDiagonalBackward0_copy_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
6614:   HANDLE_TH_ERRORS
6615:   auto prop = static_cast<DiagonalBackward0_copy*>(self->cdata.get())->self_sym_sizes;
6616:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
6617:   for (auto i : c10::irange(prop.size())) {
6618:       auto si = prop[i];
6619:       if (auto m = si.maybe_as_int()) {
6620:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
6621:       } else {
6622:         auto py_symint = py::cast(si).release().ptr();
6623:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
6624:       }
6625:   }
6626:   return tup;
6627:   END_HANDLE_TH_ERRORS
6628: }
6629: 
6630: static struct PyGetSetDef DiagonalBackward0_copy_properties[] = {
6631:   THP_FUNCTION_DEFAULT_PROPERTIES,
6632:   {(char*)"_saved_dim1", (getter)THPDiagonalBackward0_copy_dim1_getter, nullptr, nullptr, nullptr},
6633:   {(char*)"_saved_dim2", (getter)THPDiagonalBackward0_copy_dim2_getter, nullptr, nullptr, nullptr},
6634:   {(char*)"_saved_offset", (getter)THPDiagonalBackward0_copy_offset_getter, nullptr, nullptr, nullptr},
6635:   {(char*)"_saved_self_sym_sizes", (getter)THPDiagonalBackward0_copy_self_sym_sizes_getter, nullptr, nullptr, nullptr},
6636:   {nullptr} /* sentinel */
6637: };
6638: 
6639: static PyObject* THPExpandBackward0_copy_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
6640:   HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPForeachNormBackward0_result_raw_getter`, `PyErr_SetString`, `PyTuple_New`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPForeachNormBackward0_result_raw_getter`, `PyErr_SetString`, `PyTuple_New` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 6641-6720

```cpp
6641:   auto prop = static_cast<ExpandBackward0_copy*>(self->cdata.get())->self_sym_sizes;
6642:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
6643:   for (auto i : c10::irange(prop.size())) {
6644:       auto si = prop[i];
6645:       if (auto m = si.maybe_as_int()) {
6646:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
6647:       } else {
6648:         auto py_symint = py::cast(si).release().ptr();
6649:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
6650:       }
6651:   }
6652:   return tup;
6653:   END_HANDLE_TH_ERRORS
6654: }
6655: 
6656: static struct PyGetSetDef ExpandBackward0_copy_properties[] = {
6657:   THP_FUNCTION_DEFAULT_PROPERTIES,
6658:   {(char*)"_saved_self_sym_sizes", (getter)THPExpandBackward0_copy_self_sym_sizes_getter, nullptr, nullptr, nullptr},
6659:   {nullptr} /* sentinel */
6660: };
6661: 
6662: static PyObject* THPPermuteBackward0_copy_dims_getter(THPCppFunction *self, void *_unused) {
6663:   HANDLE_TH_ERRORS
6664:   auto prop = static_cast<PermuteBackward0_copy*>(self->cdata.get())->dims;
6665:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
6666:   for (auto i : c10::irange(prop.size())) {
6667:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
6668:   }
6669:   return tup;
6670:   END_HANDLE_TH_ERRORS
6671: }
6672: 
6673: static struct PyGetSetDef PermuteBackward0_copy_properties[] = {
6674:   THP_FUNCTION_DEFAULT_PROPERTIES,
6675:   {(char*)"_saved_dims", (getter)THPPermuteBackward0_copy_dims_getter, nullptr, nullptr, nullptr},
6676:   {nullptr} /* sentinel */
6677: };
6678: 
6679: static PyObject* THPReshapeAliasBackward0_copy_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
6680:   HANDLE_TH_ERRORS
6681:   auto prop = static_cast<ReshapeAliasBackward0_copy*>(self->cdata.get())->self_sym_sizes;
6682:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
6683:   for (auto i : c10::irange(prop.size())) {
6684:       auto si = prop[i];
6685:       if (auto m = si.maybe_as_int()) {
6686:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
6687:       } else {
6688:         auto py_symint = py::cast(si).release().ptr();
6689:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
6690:       }
6691:   }
6692:   return tup;
6693:   END_HANDLE_TH_ERRORS
6694: }
6695: 
6696: static struct PyGetSetDef ReshapeAliasBackward0_copy_properties[] = {
6697:   THP_FUNCTION_DEFAULT_PROPERTIES,
6698:   {(char*)"_saved_self_sym_sizes", (getter)THPReshapeAliasBackward0_copy_self_sym_sizes_getter, nullptr, nullptr, nullptr},
6699:   {nullptr} /* sentinel */
6700: };
6701: 
6702: 
6703: 
6704: static struct PyGetSetDef TBackward0_copy_properties[] = {
6705:   THP_FUNCTION_DEFAULT_PROPERTIES,
6706: 
6707:   {nullptr} /* sentinel */
6708: };
6709: 
6710: static PyObject* THPUnsqueezeBackward0_copy_dim_getter(THPCppFunction *self, void *_unused) {
6711:   HANDLE_TH_ERRORS
6712:   auto prop = static_cast<UnsqueezeBackward0_copy*>(self->cdata.get())->dim;
6713:   return PyLong_FromUnsignedLong((int64_t) prop);
6714:   END_HANDLE_TH_ERRORS
6715: }
6716: 
6717: static struct PyGetSetDef UnsqueezeBackward0_copy_properties[] = {
6718:   THP_FUNCTION_DEFAULT_PROPERTIES,
6719:   {(char*)"_saved_dim", (getter)THPUnsqueezeBackward0_copy_dim_getter, nullptr, nullptr, nullptr},
6720:   {nullptr} /* sentinel */
```

- EN: The main execution path in this span is carried by `PyTuple_New`, `PyTuple_SetItem`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyTuple_New`, `PyTuple_SetItem`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 6721-6800

```cpp
6721: };
6722: 
6723: static PyObject* THPViewBackward0_copy_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
6724:   HANDLE_TH_ERRORS
6725:   auto prop = static_cast<ViewBackward0_copy*>(self->cdata.get())->self_sym_sizes;
6726:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
6727:   for (auto i : c10::irange(prop.size())) {
6728:       auto si = prop[i];
6729:       if (auto m = si.maybe_as_int()) {
6730:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
6731:       } else {
6732:         auto py_symint = py::cast(si).release().ptr();
6733:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
6734:       }
6735:   }
6736:   return tup;
6737:   END_HANDLE_TH_ERRORS
6738: }
6739: 
6740: static struct PyGetSetDef ViewBackward0_copy_properties[] = {
6741:   THP_FUNCTION_DEFAULT_PROPERTIES,
6742:   {(char*)"_saved_self_sym_sizes", (getter)THPViewBackward0_copy_self_sym_sizes_getter, nullptr, nullptr, nullptr},
6743:   {nullptr} /* sentinel */
6744: };
6745: 
6746: static PyObject* THPViewBackwardAutogradNestedTensor0_copy_self_getter(THPCppFunction *self, void *_unused) {
6747:   HANDLE_TH_ERRORS
6748:   const auto& prop = static_cast<ViewBackwardAutogradNestedTensor0_copy*>(self->cdata.get())->self_;
6749:   return THPVariable_Wrap(prop.unpack(self->cdata));
6750:   END_HANDLE_TH_ERRORS
6751: }
6752: 
6753: static PyObject* THPViewBackwardAutogradNestedTensor0_copy_self_raw_getter(THPCppFunction *self, void *_unused) {
6754:   HANDLE_TH_ERRORS
6755:   const auto& prop = static_cast<ViewBackwardAutogradNestedTensor0_copy*>(self->cdata.get())->self_;
6756:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
6757:   return obj.release().ptr();
6758:   END_HANDLE_TH_ERRORS
6759: }
6760: 
6761: static struct PyGetSetDef ViewBackwardAutogradNestedTensor0_copy_properties[] = {
6762:   THP_FUNCTION_DEFAULT_PROPERTIES,
6763:   {(char*)"_saved_self", (getter)THPViewBackwardAutogradNestedTensor0_copy_self_getter, nullptr, nullptr, nullptr},
6764:   {(char*)"_raw_saved_self", (getter)THPViewBackwardAutogradNestedTensor0_copy_self_raw_getter, nullptr, nullptr, nullptr},
6765:   {nullptr} /* sentinel */
6766: };
6767: 
6768: static PyObject* THPTestAutogradMultipleDispatchViewBackward0_copy_self_getter(THPCppFunction *self, void *_unused) {
6769:   HANDLE_TH_ERRORS
6770:   const auto& prop = static_cast<TestAutogradMultipleDispatchViewBackward0_copy*>(self->cdata.get())->self_;
6771:   return THPVariable_Wrap(prop.unpack(self->cdata));
6772:   END_HANDLE_TH_ERRORS
6773: }
6774: 
6775: static PyObject* THPTestAutogradMultipleDispatchViewBackward0_copy_self_raw_getter(THPCppFunction *self, void *_unused) {
6776:   HANDLE_TH_ERRORS
6777:   const auto& prop = static_cast<TestAutogradMultipleDispatchViewBackward0_copy*>(self->cdata.get())->self_;
6778:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
6779:   return obj.release().ptr();
6780:   END_HANDLE_TH_ERRORS
6781: }
6782: 
6783: static struct PyGetSetDef TestAutogradMultipleDispatchViewBackward0_copy_properties[] = {
6784:   THP_FUNCTION_DEFAULT_PROPERTIES,
6785:   {(char*)"_saved_self", (getter)THPTestAutogradMultipleDispatchViewBackward0_copy_self_getter, nullptr, nullptr, nullptr},
6786:   {(char*)"_raw_saved_self", (getter)THPTestAutogradMultipleDispatchViewBackward0_copy_self_raw_getter, nullptr, nullptr, nullptr},
6787:   {nullptr} /* sentinel */
6788: };
6789: 
6790: static PyObject* THPTestAutogradMultipleDispatchViewBackwardAutogradCUDA0_copy_self_getter(THPCppFunction *self, void *_unused) {
6791:   HANDLE_TH_ERRORS
6792:   const auto& prop = static_cast<TestAutogradMultipleDispatchViewBackwardAutogradCUDA0_copy*>(self->cdata.get())->self_;
6793:   return THPVariable_Wrap(prop.unpack(self->cdata));
6794:   END_HANDLE_TH_ERRORS
6795: }
6796: 
6797: static PyObject* THPTestAutogradMultipleDispatchViewBackwardAutogradCUDA0_copy_self_raw_getter(THPCppFunction *self, void *_unused) {
6798:   HANDLE_TH_ERRORS
6799:   const auto& prop = static_cast<TestAutogradMultipleDispatchViewBackwardAutogradCUDA0_copy*>(self->cdata.get())->self_;
6800:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
```

- EN: The main execution path in this span is carried by `THPViewBackward0_copy_self_sym_sizes_getter`, `PyTuple_New`, `PyTuple_SetItem`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPViewBackward0_copy_self_sym_sizes_getter`, `PyTuple_New`, `PyTuple_SetItem` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 6801-6880

```cpp
6801:   return obj.release().ptr();
6802:   END_HANDLE_TH_ERRORS
6803: }
6804: 
6805: static struct PyGetSetDef TestAutogradMultipleDispatchViewBackwardAutogradCUDA0_copy_properties[] = {
6806:   THP_FUNCTION_DEFAULT_PROPERTIES,
6807:   {(char*)"_saved_self", (getter)THPTestAutogradMultipleDispatchViewBackwardAutogradCUDA0_copy_self_getter, nullptr, nullptr, nullptr},
6808:   {(char*)"_raw_saved_self", (getter)THPTestAutogradMultipleDispatchViewBackwardAutogradCUDA0_copy_self_raw_getter, nullptr, nullptr, nullptr},
6809:   {nullptr} /* sentinel */
6810: };
6811: 
6812: void initialize_autogenerated_functions_2(PyObject* module) {
6813:   static PyTypeObject AddBackward0Class;
6814:   addClass<AddBackward0>(module, AddBackward0Class, "AddBackward0", AddBackward0_properties);
6815:   static PyTypeObject AddBackward1Class;
6816:   addClass<AddBackward1>(module, AddBackward1Class, "AddBackward1", AddBackward1_properties);
6817:   static PyTypeObject AddcdivBackward0Class;
6818:   addClass<AddcdivBackward0>(module, AddcdivBackward0Class, "AddcdivBackward0", AddcdivBackward0_properties);
6819:   static PyTypeObject AngleBackward0Class;
6820:   addClass<AngleBackward0>(module, AngleBackward0Class, "AngleBackward0", AngleBackward0_properties);
6821:   static PyTypeObject AcoshBackward1Class;
6822:   addClass<AcoshBackward1>(module, AcoshBackward1Class, "AcoshBackward1", AcoshBackward1_properties);
6823:   static PyTypeObject AsinhBackward0Class;
6824:   addClass<AsinhBackward0>(module, AsinhBackward0Class, "AsinhBackward0", AsinhBackward0_properties);
6825:   static PyTypeObject AsinhBackward1Class;
6826:   addClass<AsinhBackward1>(module, AsinhBackward1Class, "AsinhBackward1", AsinhBackward1_properties);
6827:   static PyTypeObject AsStridedBackward1Class;
6828:   addClass<AsStridedBackward1>(module, AsStridedBackward1Class, "AsStridedBackward1", AsStridedBackward1_properties);
6829:   static PyTypeObject Atan2Backward0Class;
6830:   addClass<Atan2Backward0>(module, Atan2Backward0Class, "Atan2Backward0", Atan2Backward0_properties);
6831:   static PyTypeObject BaddbmmBackward0Class;
6832:   addClass<BaddbmmBackward0>(module, BaddbmmBackward0Class, "BaddbmmBackward0", BaddbmmBackward0_properties);
6833:   static PyTypeObject BernoulliBackward0Class;
6834:   addClass<BernoulliBackward0>(module, BernoulliBackward0Class, "BernoulliBackward0", BernoulliBackward0_properties);
6835:   static PyTypeObject ChunkBackward0Class;
6836:   addClass<ChunkBackward0>(module, ChunkBackward0Class, "ChunkBackward0", ChunkBackward0_properties);
6837:   static PyTypeObject ChunkBackwardAutogradNestedTensor0Class;
6838:   addClass<ChunkBackwardAutogradNestedTensor0>(module, ChunkBackwardAutogradNestedTensor0Class, "ChunkBackwardAutogradNestedTensor0", ChunkBackwardAutogradNestedTensor0_properties);
6839:   static PyTypeObject CholeskyInverseBackward0Class;
6840:   addClass<CholeskyInverseBackward0>(module, CholeskyInverseBackward0Class, "CholeskyInverseBackward0", CholeskyInverseBackward0_properties);
6841:   static PyTypeObject ClampBackward0Class;
6842:   addClass<ClampBackward0>(module, ClampBackward0Class, "ClampBackward0", ClampBackward0_properties);
6843:   static PyTypeObject ClampBackward1Class;
6844:   addClass<ClampBackward1>(module, ClampBackward1Class, "ClampBackward1", ClampBackward1_properties);
6845:   static PyTypeObject ClampMaxBackward0Class;
6846:   addClass<ClampMaxBackward0>(module, ClampMaxBackward0Class, "ClampMaxBackward0", ClampMaxBackward0_properties);
6847:   static PyTypeObject ClampMaxBackward1Class;
6848:   addClass<ClampMaxBackward1>(module, ClampMaxBackward1Class, "ClampMaxBackward1", ClampMaxBackward1_properties);
6849:   static PyTypeObject LazyCloneBackward0Class;
6850:   addClass<LazyCloneBackward0>(module, LazyCloneBackward0Class, "LazyCloneBackward0", LazyCloneBackward0_properties);
6851:   static PyTypeObject ComplexBackward0Class;
6852:   addClass<ComplexBackward0>(module, ComplexBackward0Class, "ComplexBackward0", ComplexBackward0_properties);
6853:   static PyTypeObject PolarBackward0Class;
6854:   addClass<PolarBackward0>(module, PolarBackward0Class, "PolarBackward0", PolarBackward0_properties);
6855:   static PyTypeObject CummaxBackward0Class;
6856:   addClass<CummaxBackward0>(module, CummaxBackward0Class, "CummaxBackward0", CummaxBackward0_properties);
6857:   static PyTypeObject BlockDiagBackward0Class;
6858:   addClass<BlockDiagBackward0>(module, BlockDiagBackward0Class, "BlockDiagBackward0", BlockDiagBackward0_properties);
6859:   static PyTypeObject DiagonalBackwardBackward0Class;
6860:   addClass<DiagonalBackwardBackward0>(module, DiagonalBackwardBackward0Class, "DiagonalBackwardBackward0", DiagonalBackwardBackward0_properties);
6861:   static PyTypeObject DotBackward0Class;
6862:   addClass<DotBackward0>(module, DotBackward0Class, "DotBackward0", DotBackward0_properties);
6863:   static PyTypeObject NativeDropoutBackward0Class;
6864:   addClass<NativeDropoutBackward0>(module, NativeDropoutBackward0Class, "NativeDropoutBackward0", NativeDropoutBackward0_properties);
6865:   static PyTypeObject ExpBackward0Class;
6866:   addClass<ExpBackward0>(module, ExpBackward0Class, "ExpBackward0", ExpBackward0_properties);
6867:   static PyTypeObject Exp2Backward0Class;
6868:   addClass<Exp2Backward0>(module, Exp2Backward0Class, "Exp2Backward0", Exp2Backward0_properties);
6869:   static PyTypeObject FakeQuantizePerTensorAffineCachemaskBackward0Class;
6870:   addClass<FakeQuantizePerTensorAffineCachemaskBackward0>(module, FakeQuantizePerTensorAffineCachemaskBackward0Class, "FakeQuantizePerTensorAffineCachemaskBackward0", FakeQuantizePerTensorAffineCachemaskBackward0_properties);
6871:   static PyTypeObject FakeQuantizePerTensorAffineCachemaskTensorQparamsBackward0Class;
6872:   addClass<FakeQuantizePerTensorAffineCachemaskTensorQparamsBackward0>(module, FakeQuantizePerTensorAffineCachemaskTensorQparamsBackward0Class, "FakeQuantizePerTensorAffineCachemaskTensorQparamsBackward0", FakeQuantizePerTensorAffineCachemaskTensorQparamsBackward0_properties);
6873:   static PyTypeObject FakeQuantizeLearnablePerTensorAffineBackward0Class;
6874:   addClass<FakeQuantizeLearnablePerTensorAffineBackward0>(module, FakeQuantizeLearnablePerTensorAffineBackward0Class, "FakeQuantizeLearnablePerTensorAffineBackward0", FakeQuantizeLearnablePerTensorAffineBackward0_properties);
6875:   static PyTypeObject FracBackward0Class;
6876:   addClass<FracBackward0>(module, FracBackward0Class, "FracBackward0", FracBackward0_properties);
6877:   static PyTypeObject HardswishBackward0Class;
6878:   addClass<HardswishBackward0>(module, HardswishBackward0Class, "HardswishBackward0", HardswishBackward0_properties);
6879:   static PyTypeObject HypotBackward0Class;
6880:   addClass<HypotBackward0>(module, HypotBackward0Class, "HypotBackward0", HypotBackward0_properties);
```

- EN: The main execution path in this span is carried by `initialize_autogenerated_functions_2`. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `initialize_autogenerated_functions_2` 等函数/方法承载。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 6881-6960

```cpp
6881:   static PyTypeObject IgammacBackward0Class;
6882:   addClass<IgammacBackward0>(module, IgammacBackward0Class, "IgammacBackward0", IgammacBackward0_properties);
6883:   static PyTypeObject UnsafeMaskedIndexBackward0Class;
6884:   addClass<UnsafeMaskedIndexBackward0>(module, UnsafeMaskedIndexBackward0Class, "UnsafeMaskedIndexBackward0", UnsafeMaskedIndexBackward0_properties);
6885:   static PyTypeObject LdexpBackward0Class;
6886:   addClass<LdexpBackward0>(module, LdexpBackward0Class, "LdexpBackward0", LdexpBackward0_properties);
6887:   static PyTypeObject LogBackward0Class;
6888:   addClass<LogBackward0>(module, LogBackward0Class, "LogBackward0", LogBackward0_properties);
6889:   static PyTypeObject Log2Backward0Class;
6890:   addClass<Log2Backward0>(module, Log2Backward0Class, "Log2Backward0", Log2Backward0_properties);
6891:   static PyTypeObject LogaddexpBackward0Class;
6892:   addClass<LogaddexpBackward0>(module, LogaddexpBackward0Class, "LogaddexpBackward0", LogaddexpBackward0_properties);
6893:   static PyTypeObject SpecialZetaBackward0Class;
6894:   addClass<SpecialZetaBackward0>(module, SpecialZetaBackward0Class, "SpecialZetaBackward0", SpecialZetaBackward0_properties);
6895:   static PyTypeObject SpecialZetaBackward1Class;
6896:   addClass<SpecialZetaBackward1>(module, SpecialZetaBackward1Class, "SpecialZetaBackward1", SpecialZetaBackward1_properties);
6897:   static PyTypeObject SpecialZetaBackward2Class;
6898:   addClass<SpecialZetaBackward2>(module, SpecialZetaBackward2Class, "SpecialZetaBackward2", SpecialZetaBackward2_properties);
6899:   static PyTypeObject MaskedScatterBackwardBackward0Class;
6900:   addClass<MaskedScatterBackwardBackward0>(module, MaskedScatterBackwardBackward0Class, "MaskedScatterBackwardBackward0", MaskedScatterBackwardBackward0_properties);
6901:   static PyTypeObject MaskedSelectBackward0Class;
6902:   addClass<MaskedSelectBackward0>(module, MaskedSelectBackward0Class, "MaskedSelectBackward0", MaskedSelectBackward0_properties);
6903:   static PyTypeObject FmaxBackward0Class;
6904:   addClass<FmaxBackward0>(module, FmaxBackward0Class, "FmaxBackward0", FmaxBackward0_properties);
6905:   static PyTypeObject NativeBatchNormLegitBackward0Class;
6906:   addClass<NativeBatchNormLegitBackward0>(module, NativeBatchNormLegitBackward0Class, "NativeBatchNormLegitBackward0", NativeBatchNormLegitBackward0_properties);
6907:   static PyTypeObject NativeBatchNormLegitBackward1Class;
6908:   addClass<NativeBatchNormLegitBackward1>(module, NativeBatchNormLegitBackward1Class, "NativeBatchNormLegitBackward1", NativeBatchNormLegitBackward1_properties);
6909:   static PyTypeObject NativeLayerNormBackwardBackward0Class;
6910:   addClass<NativeLayerNormBackwardBackward0>(module, NativeLayerNormBackwardBackward0Class, "NativeLayerNormBackwardBackward0", NativeLayerNormBackwardBackward0_properties);
6911:   static PyTypeObject BatchNormWithUpdateBackward0Class;
6912:   addClass<BatchNormWithUpdateBackward0>(module, BatchNormWithUpdateBackward0Class, "BatchNormWithUpdateBackward0", BatchNormWithUpdateBackward0_properties);
6913:   static PyTypeObject NormBackward0Class;
6914:   addClass<NormBackward0>(module, NormBackward0Class, "NormBackward0", NormBackward0_properties);
6915:   static PyTypeObject NormBackward1Class;
6916:   addClass<NormBackward1>(module, NormBackward1Class, "NormBackward1", NormBackward1_properties);
6917:   static PyTypeObject NormBackward2Class;
6918:   addClass<NormBackward2>(module, NormBackward2Class, "NormBackward2", NormBackward2_properties);
6919:   static PyTypeObject NormBackward3Class;
6920:   addClass<NormBackward3>(module, NormBackward3Class, "NormBackward3", NormBackward3_properties);
6921:   static PyTypeObject PdistBackward0Class;
6922:   addClass<PdistBackward0>(module, PdistBackward0Class, "PdistBackward0", PdistBackward0_properties);
6923:   static PyTypeObject EuclideanDistBackward0Class;
6924:   addClass<EuclideanDistBackward0>(module, EuclideanDistBackward0Class, "EuclideanDistBackward0", EuclideanDistBackward0_properties);
6925:   static PyTypeObject CdistBackwardBackward0Class;
6926:   addClass<CdistBackwardBackward0>(module, CdistBackwardBackward0Class, "CdistBackwardBackward0", CdistBackwardBackward0_properties);
6927:   static PyTypeObject LinalgHouseholderProductBackward0Class;
6928:   addClass<LinalgHouseholderProductBackward0>(module, LinalgHouseholderProductBackward0Class, "LinalgHouseholderProductBackward0", LinalgHouseholderProductBackward0_properties);
6929:   static PyTypeObject PutBackward0Class;
6930:   addClass<PutBackward0>(module, PutBackward0Class, "PutBackward0", PutBackward0_properties);
6931:   static PyTypeObject Rad2DegBackward0Class;
6932:   addClass<Rad2DegBackward0>(module, Rad2DegBackward0Class, "Rad2DegBackward0", Rad2DegBackward0_properties);
6933:   static PyTypeObject RandomBackward0Class;
6934:   addClass<RandomBackward0>(module, RandomBackward0Class, "RandomBackward0", RandomBackward0_properties);
6935:   static PyTypeObject RandomBackward1Class;
6936:   addClass<RandomBackward1>(module, RandomBackward1Class, "RandomBackward1", RandomBackward1_properties);
6937:   static PyTypeObject RandomBackward2Class;
6938:   addClass<RandomBackward2>(module, RandomBackward2Class, "RandomBackward2", RandomBackward2_properties);
6939:   static PyTypeObject RepeatBackward0Class;
6940:   addClass<RepeatBackward0>(module, RepeatBackward0Class, "RepeatBackward0", RepeatBackward0_properties);
6941:   static PyTypeObject RsqrtBackward0Class;
6942:   addClass<RsqrtBackward0>(module, RsqrtBackward0Class, "RsqrtBackward0", RsqrtBackward0_properties);
6943:   static PyTypeObject ScatterBackward0Class;
6944:   addClass<ScatterBackward0>(module, ScatterBackward0Class, "ScatterBackward0", ScatterBackward0_properties);
6945:   static PyTypeObject ScatterBackward1Class;
6946:   addClass<ScatterBackward1>(module, ScatterBackward1Class, "ScatterBackward1", ScatterBackward1_properties);
6947:   static PyTypeObject SortBackward0Class;
6948:   addClass<SortBackward0>(module, SortBackward0Class, "SortBackward0", SortBackward0_properties);
6949:   static PyTypeObject SortBackward1Class;
6950:   addClass<SortBackward1>(module, SortBackward1Class, "SortBackward1", SortBackward1_properties);
6951:   static PyTypeObject SqueezeBackward3Class;
6952:   addClass<SqueezeBackward3>(module, SqueezeBackward3Class, "SqueezeBackward3", SqueezeBackward3_properties);
6953:   static PyTypeObject SqueezeBackward4Class;
6954:   addClass<SqueezeBackward4>(module, SqueezeBackward4Class, "SqueezeBackward4", SqueezeBackward4_properties);
6955:   static PyTypeObject SqueezeBackward5Class;
6956:   addClass<SqueezeBackward5>(module, SqueezeBackward5Class, "SqueezeBackward5", SqueezeBackward5_properties);
6957:   static PyTypeObject SumBackward0Class;
6958:   addClass<SumBackward0>(module, SumBackward0Class, "SumBackward0", SumBackward0_properties);
6959:   static PyTypeObject SumBackwardAutogradNestedTensor0Class;
6960:   addClass<SumBackwardAutogradNestedTensor0>(module, SumBackwardAutogradNestedTensor0Class, "SumBackwardAutogradNestedTensor0", SumBackwardAutogradNestedTensor0_properties);
```

- EN: Autograd-related metadata or graph connectivity is updated here.
- CN: 这里会更新与自动求导相关的元数据或计算图连接关系。
### Lines 6961-7040

```cpp
6961:   static PyTypeObject SumBackward1Class;
6962:   addClass<SumBackward1>(module, SumBackward1Class, "SumBackward1", SumBackward1_properties);
6963:   static PyTypeObject SumBackwardAutogradNestedTensor1Class;
6964:   addClass<SumBackwardAutogradNestedTensor1>(module, SumBackwardAutogradNestedTensor1Class, "SumBackwardAutogradNestedTensor1", SumBackwardAutogradNestedTensor1_properties);
6965:   static PyTypeObject NansumBackward0Class;
6966:   addClass<NansumBackward0>(module, NansumBackward0Class, "NansumBackward0", NansumBackward0_properties);
6967:   static PyTypeObject LinalgEigBackward0Class;
6968:   addClass<LinalgEigBackward0>(module, LinalgEigBackward0Class, "LinalgEigBackward0", LinalgEigBackward0_properties);
6969:   static PyTypeObject Rot90Backward0Class;
6970:   addClass<Rot90Backward0>(module, Rot90Backward0Class, "Rot90Backward0", Rot90Backward0_properties);
6971:   static PyTypeObject TrilBackward0Class;
6972:   addClass<TrilBackward0>(module, TrilBackward0Class, "TrilBackward0", TrilBackward0_properties);
6973:   static PyTypeObject TruncBackward0Class;
6974:   addClass<TruncBackward0>(module, TruncBackward0Class, "TruncBackward0", TruncBackward0_properties);
6975:   static PyTypeObject ToDenseBackward0Class;
6976:   addClass<ToDenseBackward0>(module, ToDenseBackward0Class, "ToDenseBackward0", ToDenseBackward0_properties);
6977:   static PyTypeObject VarBackward0Class;
6978:   addClass<VarBackward0>(module, VarBackward0Class, "VarBackward0", VarBackward0_properties);
6979:   static PyTypeObject VarMeanBackward0Class;
6980:   addClass<VarMeanBackward0>(module, VarMeanBackward0Class, "VarMeanBackward0", VarMeanBackward0_properties);
6981:   static PyTypeObject SparseCooTensorWithDimsAndTensorsBackward0Class;
6982:   addClass<SparseCooTensorWithDimsAndTensorsBackward0>(module, SparseCooTensorWithDimsAndTensorsBackward0Class, "SparseCooTensorWithDimsAndTensorsBackward0", SparseCooTensorWithDimsAndTensorsBackward0_properties);
6983:   static PyTypeObject StandardGammaGradBackward0Class;
6984:   addClass<StandardGammaGradBackward0>(module, StandardGammaGradBackward0Class, "StandardGammaGradBackward0", StandardGammaGradBackward0_properties);
6985:   static PyTypeObject BinaryCrossEntropyWithLogitsBackward0Class;
6986:   addClass<BinaryCrossEntropyWithLogitsBackward0>(module, BinaryCrossEntropyWithLogitsBackward0Class, "BinaryCrossEntropyWithLogitsBackward0", BinaryCrossEntropyWithLogitsBackward0_properties);
6987:   static PyTypeObject MultiMarginLossBackward0Class;
6988:   addClass<MultiMarginLossBackward0>(module, MultiMarginLossBackward0Class, "MultiMarginLossBackward0", MultiMarginLossBackward0_properties);
6989:   static PyTypeObject MultilabelMarginLossBackward0Class;
6990:   addClass<MultilabelMarginLossBackward0>(module, MultilabelMarginLossBackward0Class, "MultilabelMarginLossBackward0", MultilabelMarginLossBackward0_properties);
6991:   static PyTypeObject NllLossBackward0Class;
6992:   addClass<NllLossBackward0>(module, NllLossBackward0Class, "NllLossBackward0", NllLossBackward0_properties);
6993:   static PyTypeObject PreluKernelBackward0Class;
6994:   addClass<PreluKernelBackward0>(module, PreluKernelBackward0Class, "PreluKernelBackward0", PreluKernelBackward0_properties);
6995:   static PyTypeObject RreluWithNoiseBackward0Class;
6996:   addClass<RreluWithNoiseBackward0>(module, RreluWithNoiseBackward0Class, "RreluWithNoiseBackward0", RreluWithNoiseBackward0_properties);
6997:   static PyTypeObject SparseSoftmaxBackward0Class;
6998:   addClass<SparseSoftmaxBackward0>(module, SparseSoftmaxBackward0Class, "SparseSoftmaxBackward0", SparseSoftmaxBackward0_properties);
6999:   static PyTypeObject SparseSparseMatmulBackward0Class;
7000:   addClass<SparseSparseMatmulBackward0>(module, SparseSparseMatmulBackward0Class, "SparseSparseMatmulBackward0", SparseSparseMatmulBackward0_properties);
7001:   static PyTypeObject ThresholdBackward1Class;
7002:   addClass<ThresholdBackward1>(module, ThresholdBackward1Class, "ThresholdBackward1", ThresholdBackward1_properties);
7003:   static PyTypeObject UpsampleNearestExact2DBackward0Class;
7004:   addClass<UpsampleNearestExact2DBackward0>(module, UpsampleNearestExact2DBackward0Class, "UpsampleNearestExact2DBackward0", UpsampleNearestExact2DBackward0_properties);
7005:   static PyTypeObject UpsampleNearestExact3DBackward0Class;
7006:   addClass<UpsampleNearestExact3DBackward0>(module, UpsampleNearestExact3DBackward0Class, "UpsampleNearestExact3DBackward0", UpsampleNearestExact3DBackward0_properties);
7007:   static PyTypeObject AvgPool2DBackward0Class;
7008:   addClass<AvgPool2DBackward0>(module, AvgPool2DBackward0Class, "AvgPool2DBackward0", AvgPool2DBackward0_properties);
7009:   static PyTypeObject MpsConvolutionBackwardBackward0Class;
7010:   addClass<MpsConvolutionBackwardBackward0>(module, MpsConvolutionBackwardBackward0Class, "MpsConvolutionBackwardBackward0", MpsConvolutionBackwardBackward0_properties);
7011:   static PyTypeObject SlowConvDilated3DBackward0Class;
7012:   addClass<SlowConvDilated3DBackward0>(module, SlowConvDilated3DBackward0Class, "SlowConvDilated3DBackward0", SlowConvDilated3DBackward0_properties);
7013:   static PyTypeObject AdaptiveAvgPool2DBackwardBackward0Class;
7014:   addClass<AdaptiveAvgPool2DBackwardBackward0>(module, AdaptiveAvgPool2DBackwardBackward0Class, "AdaptiveAvgPool2DBackwardBackward0", AdaptiveAvgPool2DBackwardBackward0_properties);
7015:   static PyTypeObject FractionalMaxPool2DBackwardBackward0Class;
7016:   addClass<FractionalMaxPool2DBackwardBackward0>(module, FractionalMaxPool2DBackwardBackward0Class, "FractionalMaxPool2DBackwardBackward0", FractionalMaxPool2DBackwardBackward0_properties);
7017:   static PyTypeObject MaxPool3DWithIndicesBackwardBackward0Class;
7018:   addClass<MaxPool3DWithIndicesBackwardBackward0>(module, MaxPool3DWithIndicesBackwardBackward0Class, "MaxPool3DWithIndicesBackwardBackward0", MaxPool3DWithIndicesBackwardBackward0_properties);
7019:   static PyTypeObject ReflectionPad1DBackwardBackward0Class;
7020:   addClass<ReflectionPad1DBackwardBackward0>(module, ReflectionPad1DBackwardBackward0Class, "ReflectionPad1DBackwardBackward0", ReflectionPad1DBackwardBackward0_properties);
7021:   static PyTypeObject ReflectionPad2DBackwardBackward0Class;
7022:   addClass<ReflectionPad2DBackwardBackward0>(module, ReflectionPad2DBackwardBackward0Class, "ReflectionPad2DBackwardBackward0", ReflectionPad2DBackwardBackward0_properties);
7023:   static PyTypeObject ReplicationPad1DBackwardBackward0Class;
7024:   addClass<ReplicationPad1DBackwardBackward0>(module, ReplicationPad1DBackwardBackward0Class, "ReplicationPad1DBackwardBackward0", ReplicationPad1DBackwardBackward0_properties);
7025:   static PyTypeObject ReplicationPad3DBackwardBackward0Class;
7026:   addClass<ReplicationPad3DBackwardBackward0>(module, ReplicationPad3DBackwardBackward0Class, "ReplicationPad3DBackwardBackward0", ReplicationPad3DBackwardBackward0_properties);
7027:   static PyTypeObject SoftMarginLossBackwardBackward0Class;
7028:   addClass<SoftMarginLossBackwardBackward0>(module, SoftMarginLossBackwardBackward0Class, "SoftMarginLossBackwardBackward0", SoftMarginLossBackwardBackward0_properties);
7029:   static PyTypeObject SoftshrinkBackwardBackward0Class;
7030:   addClass<SoftshrinkBackwardBackward0>(module, SoftshrinkBackwardBackward0Class, "SoftshrinkBackwardBackward0", SoftshrinkBackwardBackward0_properties);
7031:   static PyTypeObject UpsampleBilinear2DBackwardBackward0Class;
7032:   addClass<UpsampleBilinear2DBackwardBackward0>(module, UpsampleBilinear2DBackwardBackward0Class, "UpsampleBilinear2DBackwardBackward0", UpsampleBilinear2DBackwardBackward0_properties);
7033:   static PyTypeObject UpsampleBicubic2DAaBackwardBackward0Class;
7034:   addClass<UpsampleBicubic2DAaBackwardBackward0>(module, UpsampleBicubic2DAaBackwardBackward0Class, "UpsampleBicubic2DAaBackwardBackward0", UpsampleBicubic2DAaBackwardBackward0_properties);
7035:   static PyTypeObject UpsampleLanczos2DAaBackwardBackward0Class;
7036:   addClass<UpsampleLanczos2DAaBackwardBackward0>(module, UpsampleLanczos2DAaBackwardBackward0Class, "UpsampleLanczos2DAaBackwardBackward0", UpsampleLanczos2DAaBackwardBackward0_properties);
7037:   static PyTypeObject UpsampleNearestExact1DBackwardBackward0Class;
7038:   addClass<UpsampleNearestExact1DBackwardBackward0>(module, UpsampleNearestExact1DBackwardBackward0Class, "UpsampleNearestExact1DBackwardBackward0", UpsampleNearestExact1DBackwardBackward0_properties);
7039:   static PyTypeObject UpsampleNearestExact2DBackwardBackward0Class;
7040:   addClass<UpsampleNearestExact2DBackwardBackward0>(module, UpsampleNearestExact2DBackwardBackward0Class, "UpsampleNearestExact2DBackwardBackward0", UpsampleNearestExact2DBackwardBackward0_properties);
```

- EN: Autograd-related metadata or graph connectivity is updated here.
- CN: 这里会更新与自动求导相关的元数据或计算图连接关系。
### Lines 7041-7087

```cpp
7041:   static PyTypeObject MiopenConvolutionBackward0Class;
7042:   addClass<MiopenConvolutionBackward0>(module, MiopenConvolutionBackward0Class, "MiopenConvolutionBackward0", MiopenConvolutionBackward0_properties);
7043:   static PyTypeObject MiopenRnnBackward0Class;
7044:   addClass<MiopenRnnBackward0>(module, MiopenRnnBackward0Class, "MiopenRnnBackward0", MiopenRnnBackward0_properties);
7045:   static PyTypeObject MkldnnRnnLayerBackward0Class;
7046:   addClass<MkldnnRnnLayerBackward0>(module, MkldnnRnnLayerBackward0Class, "MkldnnRnnLayerBackward0", MkldnnRnnLayerBackward0_properties);
7047:   static PyTypeObject MkldnnMaxPool3DBackward0Class;
7048:   addClass<MkldnnMaxPool3DBackward0>(module, MkldnnMaxPool3DBackward0Class, "MkldnnMaxPool3DBackward0", MkldnnMaxPool3DBackward0_properties);
7049:   static PyTypeObject MkldnnReshapeBackward0Class;
7050:   addClass<MkldnnReshapeBackward0>(module, MkldnnReshapeBackward0Class, "MkldnnReshapeBackward0", MkldnnReshapeBackward0_properties);
7051:   static PyTypeObject NestedTensorFromTensorListBackward0Class;
7052:   addClass<NestedTensorFromTensorListBackward0>(module, NestedTensorFromTensorListBackward0Class, "NestedTensorFromTensorListBackward0", NestedTensorFromTensorListBackward0_properties);
7053:   static PyTypeObject ScaledDotProductCudnnAttentionBackward0Class;
7054:   addClass<ScaledDotProductCudnnAttentionBackward0>(module, ScaledDotProductCudnnAttentionBackward0Class, "ScaledDotProductCudnnAttentionBackward0", ScaledDotProductCudnnAttentionBackward0_properties);
7055:   static PyTypeObject ScatterReduceBackward0Class;
7056:   addClass<ScatterReduceBackward0>(module, ScatterReduceBackward0Class, "ScatterReduceBackward0", ScatterReduceBackward0_properties);
7057:   static PyTypeObject ForeachPowBackward0Class;
7058:   addClass<ForeachPowBackward0>(module, ForeachPowBackward0Class, "ForeachPowBackward0", ForeachPowBackward0_properties);
7059:   static PyTypeObject ForeachPowBackward1Class;
7060:   addClass<ForeachPowBackward1>(module, ForeachPowBackward1Class, "ForeachPowBackward1", ForeachPowBackward1_properties);
7061:   static PyTypeObject ForeachPowBackward2Class;
7062:   addClass<ForeachPowBackward2>(module, ForeachPowBackward2Class, "ForeachPowBackward2", ForeachPowBackward2_properties);
7063:   static PyTypeObject ForeachNormBackward0Class;
7064:   addClass<ForeachNormBackward0>(module, ForeachNormBackward0Class, "ForeachNormBackward0", ForeachNormBackward0_properties);
7065:   static PyTypeObject DiagonalBackward0_copyClass;
7066:   addClass<DiagonalBackward0_copy>(module, DiagonalBackward0_copyClass, "DiagonalBackward0_copy", DiagonalBackward0_copy_properties);
7067:   static PyTypeObject ExpandBackward0_copyClass;
7068:   addClass<ExpandBackward0_copy>(module, ExpandBackward0_copyClass, "ExpandBackward0_copy", ExpandBackward0_copy_properties);
7069:   static PyTypeObject PermuteBackward0_copyClass;
7070:   addClass<PermuteBackward0_copy>(module, PermuteBackward0_copyClass, "PermuteBackward0_copy", PermuteBackward0_copy_properties);
7071:   static PyTypeObject ReshapeAliasBackward0_copyClass;
7072:   addClass<ReshapeAliasBackward0_copy>(module, ReshapeAliasBackward0_copyClass, "ReshapeAliasBackward0_copy", ReshapeAliasBackward0_copy_properties);
7073:   static PyTypeObject TBackward0_copyClass;
7074:   addClass<TBackward0_copy>(module, TBackward0_copyClass, "TBackward0_copy", TBackward0_copy_properties);
7075:   static PyTypeObject UnsqueezeBackward0_copyClass;
7076:   addClass<UnsqueezeBackward0_copy>(module, UnsqueezeBackward0_copyClass, "UnsqueezeBackward0_copy", UnsqueezeBackward0_copy_properties);
7077:   static PyTypeObject ViewBackward0_copyClass;
7078:   addClass<ViewBackward0_copy>(module, ViewBackward0_copyClass, "ViewBackward0_copy", ViewBackward0_copy_properties);
7079:   static PyTypeObject ViewBackwardAutogradNestedTensor0_copyClass;
7080:   addClass<ViewBackwardAutogradNestedTensor0_copy>(module, ViewBackwardAutogradNestedTensor0_copyClass, "ViewBackwardAutogradNestedTensor0_copy", ViewBackwardAutogradNestedTensor0_copy_properties);
7081:   static PyTypeObject TestAutogradMultipleDispatchViewBackward0_copyClass;
7082:   addClass<TestAutogradMultipleDispatchViewBackward0_copy>(module, TestAutogradMultipleDispatchViewBackward0_copyClass, "TestAutogradMultipleDispatchViewBackward0_copy", TestAutogradMultipleDispatchViewBackward0_copy_properties);
7083:   static PyTypeObject TestAutogradMultipleDispatchViewBackwardAutogradCUDA0_copyClass;
7084:   addClass<TestAutogradMultipleDispatchViewBackwardAutogradCUDA0_copy>(module, TestAutogradMultipleDispatchViewBackwardAutogradCUDA0_copyClass, "TestAutogradMultipleDispatchViewBackwardAutogradCUDA0_copy", TestAutogradMultipleDispatchViewBackwardAutogradCUDA0_copy_properties);
7085: }
7086: 
7087: } // namespace torch::autograd::generated
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
- Key symbols / 关键符号: `addClass`, `THPAddBackward0_alpha_getter`, `PyComplex_FromDoubles`, `PyFloat_FromDouble`, `PyLong_FromLong`, `THPAddcdivBackward0_tensor1_getter`, `THPVariable_Wrap`, `THPAddcdivBackward0_tensor1_raw_getter`, `THPAddcdivBackward0_tensor2_getter`, `THPAddcdivBackward0_tensor2_raw_getter`
- Related subsystems / 相关子系统: Autograd / 自动求导, ATen operator layer / ATen 算子层, c10 core utilities / c10 核心工具, Python binding layer / Python 绑定层
