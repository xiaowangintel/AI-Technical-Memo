# python_functions_1.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/autograd/generated/python_functions_1.cpp`
- Repository: `/root/xw/pytorch`
- Purpose (EN): Defines generated backward-function classes and helpers used by the autograd engine.
- 目的 (CN): 定义自动求导引擎使用的生成式反向函数类与辅助逻辑。
- Lines: 6512
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
31: static PyObject* THPAbsBackward0_self_getter(THPCppFunction *self, void *_unused) {
32:   HANDLE_TH_ERRORS
33:   const auto& prop = static_cast<AbsBackward0*>(self->cdata.get())->self_;
34:   return THPVariable_Wrap(prop.unpack(self->cdata));
35:   END_HANDLE_TH_ERRORS
36: }
37: 
38: static PyObject* THPAbsBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
39:   HANDLE_TH_ERRORS
40:   const auto& prop = static_cast<AbsBackward0*>(self->cdata.get())->self_;
41:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
42:   return obj.release().ptr();
43:   END_HANDLE_TH_ERRORS
44: }
45: 
46: static struct PyGetSetDef AbsBackward0_properties[] = {
47:   THP_FUNCTION_DEFAULT_PROPERTIES,
48:   {(char*)"_saved_self", (getter)THPAbsBackward0_self_getter, nullptr, nullptr, nullptr},
49:   {(char*)"_raw_saved_self", (getter)THPAbsBackward0_self_raw_getter, nullptr, nullptr, nullptr},
50:   {nullptr} /* sentinel */
51: };
52: 
53: static PyObject* THPAddmvBackward0_alpha_getter(THPCppFunction *self, void *_unused) {
54:   HANDLE_TH_ERRORS
55:   auto prop = static_cast<AddmvBackward0*>(self->cdata.get())->alpha;
56:   if (prop.isComplex()) {
57:     auto cprop = prop.to<c10::complex<double>>();
58:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
59:   } else if (prop.isFloatingPoint()) {
60:     return PyFloat_FromDouble(prop.to<double>());
61:   } else if (prop.isIntegral(/*includeBool=*/false)) {
62:     return PyLong_FromLong(prop.to<int64_t>());
63:   } else if (prop.isBoolean()) {
64:     if (prop.to<bool>()) {
65:       Py_RETURN_TRUE;
66:     } else {
67:       Py_RETURN_FALSE;
68:     }
69:   } else {
70:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
71:     return nullptr;
72:   }
73:   END_HANDLE_TH_ERRORS
74: }
75: 
76: static PyObject* THPAddmvBackward0_beta_getter(THPCppFunction *self, void *_unused) {
77:   HANDLE_TH_ERRORS
78:   auto prop = static_cast<AddmvBackward0*>(self->cdata.get())->beta;
79:   if (prop.isComplex()) {
80:     auto cprop = prop.to<c10::complex<double>>();
```

- EN: These lines pull in dependencies such as `torch/csrc/autograd/generated/python_functions.h`, `Python.h`, `ATen/ATen.h`, establishing the headers needed by the implementation. They also open, refine, or close the relevant C++ namespace scope. The main execution path in this span is carried by `addClass`, `_initFunctionPyTypeObject`, `Py_INCREF`. Because this is generated binding code, the span mostly registers or forwards APIs into a mechanically produced Python-facing surface.
- CN: 这些行引入了依赖，例如 `torch/csrc/autograd/generated/python_functions.h`, `Python.h`, `ATen/ATen.h`，为后续实现建立所需的头文件基础。 它们还会打开、细化或关闭相应的 C++ 命名空间作用域。 这一段的主要执行路径由 `addClass`, `_initFunctionPyTypeObject`, `Py_INCREF` 等函数/方法承载。 由于这是生成的绑定代码，这一段主要是在机械化生成的 Python 接口层上完成 API 注册或转发。
### Lines 81-160

```cpp
 81:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
 82:   } else if (prop.isFloatingPoint()) {
 83:     return PyFloat_FromDouble(prop.to<double>());
 84:   } else if (prop.isIntegral(/*includeBool=*/false)) {
 85:     return PyLong_FromLong(prop.to<int64_t>());
 86:   } else if (prop.isBoolean()) {
 87:     if (prop.to<bool>()) {
 88:       Py_RETURN_TRUE;
 89:     } else {
 90:       Py_RETURN_FALSE;
 91:     }
 92:   } else {
 93:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
 94:     return nullptr;
 95:   }
 96:   END_HANDLE_TH_ERRORS
 97: }
 98: 
 99: static PyObject* THPAddmvBackward0_mat_getter(THPCppFunction *self, void *_unused) {
100:   HANDLE_TH_ERRORS
101:   const auto& prop = static_cast<AddmvBackward0*>(self->cdata.get())->mat_;
102:   return THPVariable_Wrap(prop.unpack(self->cdata));
103:   END_HANDLE_TH_ERRORS
104: }
105: 
106: static PyObject* THPAddmvBackward0_mat_raw_getter(THPCppFunction *self, void *_unused) {
107:   HANDLE_TH_ERRORS
108:   const auto& prop = static_cast<AddmvBackward0*>(self->cdata.get())->mat_;
109:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
110:   return obj.release().ptr();
111:   END_HANDLE_TH_ERRORS
112: }
113: 
114: static PyObject* THPAddmvBackward0_vec_getter(THPCppFunction *self, void *_unused) {
115:   HANDLE_TH_ERRORS
116:   const auto& prop = static_cast<AddmvBackward0*>(self->cdata.get())->vec_;
117:   return THPVariable_Wrap(prop.unpack(self->cdata));
118:   END_HANDLE_TH_ERRORS
119: }
120: 
121: static PyObject* THPAddmvBackward0_vec_raw_getter(THPCppFunction *self, void *_unused) {
122:   HANDLE_TH_ERRORS
123:   const auto& prop = static_cast<AddmvBackward0*>(self->cdata.get())->vec_;
124:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
125:   return obj.release().ptr();
126:   END_HANDLE_TH_ERRORS
127: }
128: 
129: static struct PyGetSetDef AddmvBackward0_properties[] = {
130:   THP_FUNCTION_DEFAULT_PROPERTIES,
131:   {(char*)"_saved_alpha", (getter)THPAddmvBackward0_alpha_getter, nullptr, nullptr, nullptr},
132:   {(char*)"_saved_beta", (getter)THPAddmvBackward0_beta_getter, nullptr, nullptr, nullptr},
133:   {(char*)"_saved_mat", (getter)THPAddmvBackward0_mat_getter, nullptr, nullptr, nullptr},
134:   {(char*)"_raw_saved_mat", (getter)THPAddmvBackward0_mat_raw_getter, nullptr, nullptr, nullptr},
135:   {(char*)"_saved_vec", (getter)THPAddmvBackward0_vec_getter, nullptr, nullptr, nullptr},
136:   {(char*)"_raw_saved_vec", (getter)THPAddmvBackward0_vec_raw_getter, nullptr, nullptr, nullptr},
137:   {nullptr} /* sentinel */
138: };
139: 
140: static PyObject* THPAddrBackward0_alpha_getter(THPCppFunction *self, void *_unused) {
141:   HANDLE_TH_ERRORS
142:   auto prop = static_cast<AddrBackward0*>(self->cdata.get())->alpha;
143:   if (prop.isComplex()) {
144:     auto cprop = prop.to<c10::complex<double>>();
145:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
146:   } else if (prop.isFloatingPoint()) {
147:     return PyFloat_FromDouble(prop.to<double>());
148:   } else if (prop.isIntegral(/*includeBool=*/false)) {
149:     return PyLong_FromLong(prop.to<int64_t>());
150:   } else if (prop.isBoolean()) {
151:     if (prop.to<bool>()) {
152:       Py_RETURN_TRUE;
153:     } else {
154:       Py_RETURN_FALSE;
155:     }
156:   } else {
157:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
158:     return nullptr;
159:   }
160:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `PyComplex_FromDoubles`, `PyFloat_FromDouble`, `PyLong_FromLong`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyComplex_FromDoubles`, `PyFloat_FromDouble`, `PyLong_FromLong` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 161-240

```cpp
161: }
162: 
163: static PyObject* THPAddrBackward0_beta_getter(THPCppFunction *self, void *_unused) {
164:   HANDLE_TH_ERRORS
165:   auto prop = static_cast<AddrBackward0*>(self->cdata.get())->beta;
166:   if (prop.isComplex()) {
167:     auto cprop = prop.to<c10::complex<double>>();
168:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
169:   } else if (prop.isFloatingPoint()) {
170:     return PyFloat_FromDouble(prop.to<double>());
171:   } else if (prop.isIntegral(/*includeBool=*/false)) {
172:     return PyLong_FromLong(prop.to<int64_t>());
173:   } else if (prop.isBoolean()) {
174:     if (prop.to<bool>()) {
175:       Py_RETURN_TRUE;
176:     } else {
177:       Py_RETURN_FALSE;
178:     }
179:   } else {
180:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
181:     return nullptr;
182:   }
183:   END_HANDLE_TH_ERRORS
184: }
185: 
186: static PyObject* THPAddrBackward0_vec1_getter(THPCppFunction *self, void *_unused) {
187:   HANDLE_TH_ERRORS
188:   const auto& prop = static_cast<AddrBackward0*>(self->cdata.get())->vec1_;
189:   return THPVariable_Wrap(prop.unpack(self->cdata));
190:   END_HANDLE_TH_ERRORS
191: }
192: 
193: static PyObject* THPAddrBackward0_vec1_raw_getter(THPCppFunction *self, void *_unused) {
194:   HANDLE_TH_ERRORS
195:   const auto& prop = static_cast<AddrBackward0*>(self->cdata.get())->vec1_;
196:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
197:   return obj.release().ptr();
198:   END_HANDLE_TH_ERRORS
199: }
200: 
201: static PyObject* THPAddrBackward0_vec2_getter(THPCppFunction *self, void *_unused) {
202:   HANDLE_TH_ERRORS
203:   const auto& prop = static_cast<AddrBackward0*>(self->cdata.get())->vec2_;
204:   return THPVariable_Wrap(prop.unpack(self->cdata));
205:   END_HANDLE_TH_ERRORS
206: }
207: 
208: static PyObject* THPAddrBackward0_vec2_raw_getter(THPCppFunction *self, void *_unused) {
209:   HANDLE_TH_ERRORS
210:   const auto& prop = static_cast<AddrBackward0*>(self->cdata.get())->vec2_;
211:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
212:   return obj.release().ptr();
213:   END_HANDLE_TH_ERRORS
214: }
215: 
216: static struct PyGetSetDef AddrBackward0_properties[] = {
217:   THP_FUNCTION_DEFAULT_PROPERTIES,
218:   {(char*)"_saved_alpha", (getter)THPAddrBackward0_alpha_getter, nullptr, nullptr, nullptr},
219:   {(char*)"_saved_beta", (getter)THPAddrBackward0_beta_getter, nullptr, nullptr, nullptr},
220:   {(char*)"_saved_vec1", (getter)THPAddrBackward0_vec1_getter, nullptr, nullptr, nullptr},
221:   {(char*)"_raw_saved_vec1", (getter)THPAddrBackward0_vec1_raw_getter, nullptr, nullptr, nullptr},
222:   {(char*)"_saved_vec2", (getter)THPAddrBackward0_vec2_getter, nullptr, nullptr, nullptr},
223:   {(char*)"_raw_saved_vec2", (getter)THPAddrBackward0_vec2_raw_getter, nullptr, nullptr, nullptr},
224:   {nullptr} /* sentinel */
225: };
226: 
227: 
228: 
229: static struct PyGetSetDef AliasBackward0_properties[] = {
230:   THP_FUNCTION_DEFAULT_PROPERTIES,
231: 
232:   {nullptr} /* sentinel */
233: };
234: 
235: static PyObject* THPAtanBackward0_self_getter(THPCppFunction *self, void *_unused) {
236:   HANDLE_TH_ERRORS
237:   const auto& prop = static_cast<AtanBackward0*>(self->cdata.get())->self_;
238:   return THPVariable_Wrap(prop.unpack(self->cdata));
239:   END_HANDLE_TH_ERRORS
240: }
```

- EN: The main execution path in this span is carried by `THPAddrBackward0_beta_getter`, `PyComplex_FromDoubles`, `PyFloat_FromDouble`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPAddrBackward0_beta_getter`, `PyComplex_FromDoubles`, `PyFloat_FromDouble` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 241-320

```cpp
241: 
242: static PyObject* THPAtanBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
243:   HANDLE_TH_ERRORS
244:   const auto& prop = static_cast<AtanBackward0*>(self->cdata.get())->self_;
245:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
246:   return obj.release().ptr();
247:   END_HANDLE_TH_ERRORS
248: }
249: 
250: static struct PyGetSetDef AtanBackward0_properties[] = {
251:   THP_FUNCTION_DEFAULT_PROPERTIES,
252:   {(char*)"_saved_self", (getter)THPAtanBackward0_self_getter, nullptr, nullptr, nullptr},
253:   {(char*)"_raw_saved_self", (getter)THPAtanBackward0_self_raw_getter, nullptr, nullptr, nullptr},
254:   {nullptr} /* sentinel */
255: };
256: 
257: 
258: 
259: static struct PyGetSetDef CeilBackward0_properties[] = {
260:   THP_FUNCTION_DEFAULT_PROPERTIES,
261: 
262:   {nullptr} /* sentinel */
263: };
264: 
265: static PyObject* THPLinalgCholeskyExBackward0_upper_getter(THPCppFunction *self, void *_unused) {
266:   HANDLE_TH_ERRORS
267:   auto prop = static_cast<LinalgCholeskyExBackward0*>(self->cdata.get())->upper;
268:   if (prop) {
269:     Py_RETURN_TRUE;
270:   } else {
271:     Py_RETURN_FALSE;
272:   }
273:   END_HANDLE_TH_ERRORS
274: }
275: 
276: static PyObject* THPLinalgCholeskyExBackward0_L_getter(THPCppFunction *self, void *_unused) {
277:   HANDLE_TH_ERRORS
278:   const auto& prop = static_cast<LinalgCholeskyExBackward0*>(self->cdata.get())->L_;
279:   return THPVariable_Wrap(prop.unpack(self->cdata));
280:   END_HANDLE_TH_ERRORS
281: }
282: 
283: static PyObject* THPLinalgCholeskyExBackward0_L_raw_getter(THPCppFunction *self, void *_unused) {
284:   HANDLE_TH_ERRORS
285:   const auto& prop = static_cast<LinalgCholeskyExBackward0*>(self->cdata.get())->L_;
286:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
287:   return obj.release().ptr();
288:   END_HANDLE_TH_ERRORS
289: }
290: 
291: static struct PyGetSetDef LinalgCholeskyExBackward0_properties[] = {
292:   THP_FUNCTION_DEFAULT_PROPERTIES,
293:   {(char*)"_saved_upper", (getter)THPLinalgCholeskyExBackward0_upper_getter, nullptr, nullptr, nullptr},
294:   {(char*)"_saved_L", (getter)THPLinalgCholeskyExBackward0_L_getter, nullptr, nullptr, nullptr},
295:   {(char*)"_raw_saved_L", (getter)THPLinalgCholeskyExBackward0_L_raw_getter, nullptr, nullptr, nullptr},
296:   {nullptr} /* sentinel */
297: };
298: 
299: static PyObject* THPCholeskySolveBackward0_input2_getter(THPCppFunction *self, void *_unused) {
300:   HANDLE_TH_ERRORS
301:   const auto& prop = static_cast<CholeskySolveBackward0*>(self->cdata.get())->input2_;
302:   return THPVariable_Wrap(prop.unpack(self->cdata));
303:   END_HANDLE_TH_ERRORS
304: }
305: 
306: static PyObject* THPCholeskySolveBackward0_input2_raw_getter(THPCppFunction *self, void *_unused) {
307:   HANDLE_TH_ERRORS
308:   const auto& prop = static_cast<CholeskySolveBackward0*>(self->cdata.get())->input2_;
309:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
310:   return obj.release().ptr();
311:   END_HANDLE_TH_ERRORS
312: }
313: 
314: static PyObject* THPCholeskySolveBackward0_self_getter(THPCppFunction *self, void *_unused) {
315:   HANDLE_TH_ERRORS
316:   const auto& prop = static_cast<CholeskySolveBackward0*>(self->cdata.get())->self_;
317:   return THPVariable_Wrap(prop.unpack(self->cdata));
318:   END_HANDLE_TH_ERRORS
319: }
320: 
```

- EN: The main execution path in this span is carried by `THPAtanBackward0_self_raw_getter`, `cast`, `THPLinalgCholeskyExBackward0_upper_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPAtanBackward0_self_raw_getter`, `cast`, `THPLinalgCholeskyExBackward0_upper_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 321-400

```cpp
321: static PyObject* THPCholeskySolveBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
322:   HANDLE_TH_ERRORS
323:   const auto& prop = static_cast<CholeskySolveBackward0*>(self->cdata.get())->self_;
324:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
325:   return obj.release().ptr();
326:   END_HANDLE_TH_ERRORS
327: }
328: 
329: static PyObject* THPCholeskySolveBackward0_upper_getter(THPCppFunction *self, void *_unused) {
330:   HANDLE_TH_ERRORS
331:   auto prop = static_cast<CholeskySolveBackward0*>(self->cdata.get())->upper;
332:   if (prop) {
333:     Py_RETURN_TRUE;
334:   } else {
335:     Py_RETURN_FALSE;
336:   }
337:   END_HANDLE_TH_ERRORS
338: }
339: 
340: static PyObject* THPCholeskySolveBackward0_result_getter(THPCppFunction *self, void *_unused) {
341:   HANDLE_TH_ERRORS
342:   const auto& prop = static_cast<CholeskySolveBackward0*>(self->cdata.get())->result_;
343:   return THPVariable_Wrap(prop.unpack(self->cdata));
344:   END_HANDLE_TH_ERRORS
345: }
346: 
347: static PyObject* THPCholeskySolveBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
348:   HANDLE_TH_ERRORS
349:   const auto& prop = static_cast<CholeskySolveBackward0*>(self->cdata.get())->result_;
350:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
351:   return obj.release().ptr();
352:   END_HANDLE_TH_ERRORS
353: }
354: 
355: static struct PyGetSetDef CholeskySolveBackward0_properties[] = {
356:   THP_FUNCTION_DEFAULT_PROPERTIES,
357:   {(char*)"_saved_input2", (getter)THPCholeskySolveBackward0_input2_getter, nullptr, nullptr, nullptr},
358:   {(char*)"_raw_saved_input2", (getter)THPCholeskySolveBackward0_input2_raw_getter, nullptr, nullptr, nullptr},
359:   {(char*)"_saved_self", (getter)THPCholeskySolveBackward0_self_getter, nullptr, nullptr, nullptr},
360:   {(char*)"_raw_saved_self", (getter)THPCholeskySolveBackward0_self_raw_getter, nullptr, nullptr, nullptr},
361:   {(char*)"_saved_upper", (getter)THPCholeskySolveBackward0_upper_getter, nullptr, nullptr, nullptr},
362:   {(char*)"_saved_result", (getter)THPCholeskySolveBackward0_result_getter, nullptr, nullptr, nullptr},
363:   {(char*)"_raw_saved_result", (getter)THPCholeskySolveBackward0_result_raw_getter, nullptr, nullptr, nullptr},
364:   {nullptr} /* sentinel */
365: };
366: 
367: 
368: 
369: static struct PyGetSetDef CloneBackward0_properties[] = {
370:   THP_FUNCTION_DEFAULT_PROPERTIES,
371: 
372:   {nullptr} /* sentinel */
373: };
374: 
375: static PyObject* THPCoshBackward0_self_getter(THPCppFunction *self, void *_unused) {
376:   HANDLE_TH_ERRORS
377:   const auto& prop = static_cast<CoshBackward0*>(self->cdata.get())->self_;
378:   return THPVariable_Wrap(prop.unpack(self->cdata));
379:   END_HANDLE_TH_ERRORS
380: }
381: 
382: static PyObject* THPCoshBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
383:   HANDLE_TH_ERRORS
384:   const auto& prop = static_cast<CoshBackward0*>(self->cdata.get())->self_;
385:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
386:   return obj.release().ptr();
387:   END_HANDLE_TH_ERRORS
388: }
389: 
390: static struct PyGetSetDef CoshBackward0_properties[] = {
391:   THP_FUNCTION_DEFAULT_PROPERTIES,
392:   {(char*)"_saved_self", (getter)THPCoshBackward0_self_getter, nullptr, nullptr, nullptr},
393:   {(char*)"_raw_saved_self", (getter)THPCoshBackward0_self_raw_getter, nullptr, nullptr, nullptr},
394:   {nullptr} /* sentinel */
395: };
396: 
397: static PyObject* THPLinalgCrossBackward0_dim_getter(THPCppFunction *self, void *_unused) {
398:   HANDLE_TH_ERRORS
399:   auto prop = static_cast<LinalgCrossBackward0*>(self->cdata.get())->dim;
400:   return PyLong_FromUnsignedLong((int64_t) prop);
```

- EN: The main execution path in this span is carried by `THPCholeskySolveBackward0_self_raw_getter`, `cast`, `THPCholeskySolveBackward0_upper_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPCholeskySolveBackward0_self_raw_getter`, `cast`, `THPCholeskySolveBackward0_upper_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 401-480

```cpp
401:   END_HANDLE_TH_ERRORS
402: }
403: 
404: static PyObject* THPLinalgCrossBackward0_other_getter(THPCppFunction *self, void *_unused) {
405:   HANDLE_TH_ERRORS
406:   const auto& prop = static_cast<LinalgCrossBackward0*>(self->cdata.get())->other_;
407:   return THPVariable_Wrap(prop.unpack(self->cdata));
408:   END_HANDLE_TH_ERRORS
409: }
410: 
411: static PyObject* THPLinalgCrossBackward0_other_raw_getter(THPCppFunction *self, void *_unused) {
412:   HANDLE_TH_ERRORS
413:   const auto& prop = static_cast<LinalgCrossBackward0*>(self->cdata.get())->other_;
414:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
415:   return obj.release().ptr();
416:   END_HANDLE_TH_ERRORS
417: }
418: 
419: static PyObject* THPLinalgCrossBackward0_self_getter(THPCppFunction *self, void *_unused) {
420:   HANDLE_TH_ERRORS
421:   const auto& prop = static_cast<LinalgCrossBackward0*>(self->cdata.get())->self_;
422:   return THPVariable_Wrap(prop.unpack(self->cdata));
423:   END_HANDLE_TH_ERRORS
424: }
425: 
426: static PyObject* THPLinalgCrossBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
427:   HANDLE_TH_ERRORS
428:   const auto& prop = static_cast<LinalgCrossBackward0*>(self->cdata.get())->self_;
429:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
430:   return obj.release().ptr();
431:   END_HANDLE_TH_ERRORS
432: }
433: 
434: static struct PyGetSetDef LinalgCrossBackward0_properties[] = {
435:   THP_FUNCTION_DEFAULT_PROPERTIES,
436:   {(char*)"_saved_dim", (getter)THPLinalgCrossBackward0_dim_getter, nullptr, nullptr, nullptr},
437:   {(char*)"_saved_other", (getter)THPLinalgCrossBackward0_other_getter, nullptr, nullptr, nullptr},
438:   {(char*)"_raw_saved_other", (getter)THPLinalgCrossBackward0_other_raw_getter, nullptr, nullptr, nullptr},
439:   {(char*)"_saved_self", (getter)THPLinalgCrossBackward0_self_getter, nullptr, nullptr, nullptr},
440:   {(char*)"_raw_saved_self", (getter)THPLinalgCrossBackward0_self_raw_getter, nullptr, nullptr, nullptr},
441:   {nullptr} /* sentinel */
442: };
443: 
444: static PyObject* THPLogcumsumexpBackward0_dim_getter(THPCppFunction *self, void *_unused) {
445:   HANDLE_TH_ERRORS
446:   auto prop = static_cast<LogcumsumexpBackward0*>(self->cdata.get())->dim;
447:   return PyLong_FromUnsignedLong((int64_t) prop);
448:   END_HANDLE_TH_ERRORS
449: }
450: 
451: static PyObject* THPLogcumsumexpBackward0_self_getter(THPCppFunction *self, void *_unused) {
452:   HANDLE_TH_ERRORS
453:   const auto& prop = static_cast<LogcumsumexpBackward0*>(self->cdata.get())->self_;
454:   return THPVariable_Wrap(prop.unpack(self->cdata));
455:   END_HANDLE_TH_ERRORS
456: }
457: 
458: static PyObject* THPLogcumsumexpBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
459:   HANDLE_TH_ERRORS
460:   const auto& prop = static_cast<LogcumsumexpBackward0*>(self->cdata.get())->self_;
461:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
462:   return obj.release().ptr();
463:   END_HANDLE_TH_ERRORS
464: }
465: 
466: static PyObject* THPLogcumsumexpBackward0_result_getter(THPCppFunction *self, void *_unused) {
467:   HANDLE_TH_ERRORS
468:   const auto& prop = static_cast<LogcumsumexpBackward0*>(self->cdata.get())->result_;
469:   return THPVariable_Wrap(prop.unpack(self->cdata));
470:   END_HANDLE_TH_ERRORS
471: }
472: 
473: static PyObject* THPLogcumsumexpBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
474:   HANDLE_TH_ERRORS
475:   const auto& prop = static_cast<LogcumsumexpBackward0*>(self->cdata.get())->result_;
476:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
477:   return obj.release().ptr();
478:   END_HANDLE_TH_ERRORS
479: }
480: 
```

- EN: The main execution path in this span is carried by `THPLinalgCrossBackward0_other_getter`, `THPVariable_Wrap`, `THPLinalgCrossBackward0_other_raw_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPLinalgCrossBackward0_other_getter`, `THPVariable_Wrap`, `THPLinalgCrossBackward0_other_raw_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 481-560

```cpp
481: static struct PyGetSetDef LogcumsumexpBackward0_properties[] = {
482:   THP_FUNCTION_DEFAULT_PROPERTIES,
483:   {(char*)"_saved_dim", (getter)THPLogcumsumexpBackward0_dim_getter, nullptr, nullptr, nullptr},
484:   {(char*)"_saved_self", (getter)THPLogcumsumexpBackward0_self_getter, nullptr, nullptr, nullptr},
485:   {(char*)"_raw_saved_self", (getter)THPLogcumsumexpBackward0_self_raw_getter, nullptr, nullptr, nullptr},
486:   {(char*)"_saved_result", (getter)THPLogcumsumexpBackward0_result_getter, nullptr, nullptr, nullptr},
487:   {(char*)"_raw_saved_result", (getter)THPLogcumsumexpBackward0_result_raw_getter, nullptr, nullptr, nullptr},
488:   {nullptr} /* sentinel */
489: };
490: 
491: static PyObject* THPCumminBackward0_dim_getter(THPCppFunction *self, void *_unused) {
492:   HANDLE_TH_ERRORS
493:   auto prop = static_cast<CumminBackward0*>(self->cdata.get())->dim;
494:   return PyLong_FromUnsignedLong((int64_t) prop);
495:   END_HANDLE_TH_ERRORS
496: }
497: 
498: static PyObject* THPCumminBackward0_self_getter(THPCppFunction *self, void *_unused) {
499:   HANDLE_TH_ERRORS
500:   const auto& prop = static_cast<CumminBackward0*>(self->cdata.get())->self_;
501:   return THPVariable_Wrap(prop.unpack(self->cdata));
502:   END_HANDLE_TH_ERRORS
503: }
504: 
505: static PyObject* THPCumminBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
506:   HANDLE_TH_ERRORS
507:   const auto& prop = static_cast<CumminBackward0*>(self->cdata.get())->self_;
508:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
509:   return obj.release().ptr();
510:   END_HANDLE_TH_ERRORS
511: }
512: 
513: static PyObject* THPCumminBackward0_indices_getter(THPCppFunction *self, void *_unused) {
514:   HANDLE_TH_ERRORS
515:   const auto& prop = static_cast<CumminBackward0*>(self->cdata.get())->indices_;
516:   return THPVariable_Wrap(prop.unpack(self->cdata));
517:   END_HANDLE_TH_ERRORS
518: }
519: 
520: static PyObject* THPCumminBackward0_indices_raw_getter(THPCppFunction *self, void *_unused) {
521:   HANDLE_TH_ERRORS
522:   const auto& prop = static_cast<CumminBackward0*>(self->cdata.get())->indices_;
523:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
524:   return obj.release().ptr();
525:   END_HANDLE_TH_ERRORS
526: }
527: 
528: static struct PyGetSetDef CumminBackward0_properties[] = {
529:   THP_FUNCTION_DEFAULT_PROPERTIES,
530:   {(char*)"_saved_dim", (getter)THPCumminBackward0_dim_getter, nullptr, nullptr, nullptr},
531:   {(char*)"_saved_self", (getter)THPCumminBackward0_self_getter, nullptr, nullptr, nullptr},
532:   {(char*)"_raw_saved_self", (getter)THPCumminBackward0_self_raw_getter, nullptr, nullptr, nullptr},
533:   {(char*)"_saved_indices", (getter)THPCumminBackward0_indices_getter, nullptr, nullptr, nullptr},
534:   {(char*)"_raw_saved_indices", (getter)THPCumminBackward0_indices_raw_getter, nullptr, nullptr, nullptr},
535:   {nullptr} /* sentinel */
536: };
537: 
538: static PyObject* THPConvTbcBackward0_bias_getter(THPCppFunction *self, void *_unused) {
539:   HANDLE_TH_ERRORS
540:   const auto& prop = static_cast<ConvTbcBackward0*>(self->cdata.get())->bias_;
541:   return THPVariable_Wrap(prop.unpack(self->cdata));
542:   END_HANDLE_TH_ERRORS
543: }
544: 
545: static PyObject* THPConvTbcBackward0_bias_raw_getter(THPCppFunction *self, void *_unused) {
546:   HANDLE_TH_ERRORS
547:   const auto& prop = static_cast<ConvTbcBackward0*>(self->cdata.get())->bias_;
548:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
549:   return obj.release().ptr();
550:   END_HANDLE_TH_ERRORS
551: }
552: 
553: static PyObject* THPConvTbcBackward0_pad_getter(THPCppFunction *self, void *_unused) {
554:   HANDLE_TH_ERRORS
555:   auto prop = static_cast<ConvTbcBackward0*>(self->cdata.get())->pad;
556:   return PyLong_FromUnsignedLong((int64_t) prop);
557:   END_HANDLE_TH_ERRORS
558: }
559: 
560: static PyObject* THPConvTbcBackward0_self_getter(THPCppFunction *self, void *_unused) {
```

- EN: The main execution path in this span is carried by `THPCumminBackward0_dim_getter`, `PyLong_FromUnsignedLong`, `THPCumminBackward0_self_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPCumminBackward0_dim_getter`, `PyLong_FromUnsignedLong`, `THPCumminBackward0_self_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 561-640

```cpp
561:   HANDLE_TH_ERRORS
562:   const auto& prop = static_cast<ConvTbcBackward0*>(self->cdata.get())->self_;
563:   return THPVariable_Wrap(prop.unpack(self->cdata));
564:   END_HANDLE_TH_ERRORS
565: }
566: 
567: static PyObject* THPConvTbcBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
568:   HANDLE_TH_ERRORS
569:   const auto& prop = static_cast<ConvTbcBackward0*>(self->cdata.get())->self_;
570:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
571:   return obj.release().ptr();
572:   END_HANDLE_TH_ERRORS
573: }
574: 
575: static PyObject* THPConvTbcBackward0_weight_getter(THPCppFunction *self, void *_unused) {
576:   HANDLE_TH_ERRORS
577:   const auto& prop = static_cast<ConvTbcBackward0*>(self->cdata.get())->weight_;
578:   return THPVariable_Wrap(prop.unpack(self->cdata));
579:   END_HANDLE_TH_ERRORS
580: }
581: 
582: static PyObject* THPConvTbcBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
583:   HANDLE_TH_ERRORS
584:   const auto& prop = static_cast<ConvTbcBackward0*>(self->cdata.get())->weight_;
585:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
586:   return obj.release().ptr();
587:   END_HANDLE_TH_ERRORS
588: }
589: 
590: static struct PyGetSetDef ConvTbcBackward0_properties[] = {
591:   THP_FUNCTION_DEFAULT_PROPERTIES,
592:   {(char*)"_saved_bias", (getter)THPConvTbcBackward0_bias_getter, nullptr, nullptr, nullptr},
593:   {(char*)"_raw_saved_bias", (getter)THPConvTbcBackward0_bias_raw_getter, nullptr, nullptr, nullptr},
594:   {(char*)"_saved_pad", (getter)THPConvTbcBackward0_pad_getter, nullptr, nullptr, nullptr},
595:   {(char*)"_saved_self", (getter)THPConvTbcBackward0_self_getter, nullptr, nullptr, nullptr},
596:   {(char*)"_raw_saved_self", (getter)THPConvTbcBackward0_self_raw_getter, nullptr, nullptr, nullptr},
597:   {(char*)"_saved_weight", (getter)THPConvTbcBackward0_weight_getter, nullptr, nullptr, nullptr},
598:   {(char*)"_raw_saved_weight", (getter)THPConvTbcBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
599:   {nullptr} /* sentinel */
600: };
601: 
602: static PyObject* THPDivBackward0_other_getter(THPCppFunction *self, void *_unused) {
603:   HANDLE_TH_ERRORS
604:   const auto& prop = static_cast<DivBackward0*>(self->cdata.get())->other_;
605:   return THPVariable_Wrap(prop.unpack(self->cdata));
606:   END_HANDLE_TH_ERRORS
607: }
608: 
609: static PyObject* THPDivBackward0_other_raw_getter(THPCppFunction *self, void *_unused) {
610:   HANDLE_TH_ERRORS
611:   const auto& prop = static_cast<DivBackward0*>(self->cdata.get())->other_;
612:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
613:   return obj.release().ptr();
614:   END_HANDLE_TH_ERRORS
615: }
616: 
617: static PyObject* THPDivBackward0_self_getter(THPCppFunction *self, void *_unused) {
618:   HANDLE_TH_ERRORS
619:   const auto& prop = static_cast<DivBackward0*>(self->cdata.get())->self_;
620:   return THPVariable_Wrap(prop.unpack(self->cdata));
621:   END_HANDLE_TH_ERRORS
622: }
623: 
624: static PyObject* THPDivBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
625:   HANDLE_TH_ERRORS
626:   const auto& prop = static_cast<DivBackward0*>(self->cdata.get())->self_;
627:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
628:   return obj.release().ptr();
629:   END_HANDLE_TH_ERRORS
630: }
631: 
632: static struct PyGetSetDef DivBackward0_properties[] = {
633:   THP_FUNCTION_DEFAULT_PROPERTIES,
634:   {(char*)"_saved_other", (getter)THPDivBackward0_other_getter, nullptr, nullptr, nullptr},
635:   {(char*)"_raw_saved_other", (getter)THPDivBackward0_other_raw_getter, nullptr, nullptr, nullptr},
636:   {(char*)"_saved_self", (getter)THPDivBackward0_self_getter, nullptr, nullptr, nullptr},
637:   {(char*)"_raw_saved_self", (getter)THPDivBackward0_self_raw_getter, nullptr, nullptr, nullptr},
638:   {nullptr} /* sentinel */
639: };
640: 
```

- EN: The main execution path in this span is carried by `THPVariable_Wrap`, `THPConvTbcBackward0_self_raw_getter`, `cast`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_Wrap`, `THPConvTbcBackward0_self_raw_getter`, `cast` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 641-720

```cpp
641: static PyObject* THPDivBackward1_other_getter(THPCppFunction *self, void *_unused) {
642:   HANDLE_TH_ERRORS
643:   auto prop = static_cast<DivBackward1*>(self->cdata.get())->other;
644:   if (prop.isComplex()) {
645:     auto cprop = prop.to<c10::complex<double>>();
646:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
647:   } else if (prop.isFloatingPoint()) {
648:     return PyFloat_FromDouble(prop.to<double>());
649:   } else if (prop.isIntegral(/*includeBool=*/false)) {
650:     return PyLong_FromLong(prop.to<int64_t>());
651:   } else if (prop.isBoolean()) {
652:     if (prop.to<bool>()) {
653:       Py_RETURN_TRUE;
654:     } else {
655:       Py_RETURN_FALSE;
656:     }
657:   } else {
658:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
659:     return nullptr;
660:   }
661:   END_HANDLE_TH_ERRORS
662: }
663: 
664: static struct PyGetSetDef DivBackward1_properties[] = {
665:   THP_FUNCTION_DEFAULT_PROPERTIES,
666:   {(char*)"_saved_other", (getter)THPDivBackward1_other_getter, nullptr, nullptr, nullptr},
667:   {nullptr} /* sentinel */
668: };
669: 
670: static PyObject* THPDivBackward2_other_getter(THPCppFunction *self, void *_unused) {
671:   HANDLE_TH_ERRORS
672:   const auto& prop = static_cast<DivBackward2*>(self->cdata.get())->other_;
673:   return THPVariable_Wrap(prop.unpack(self->cdata));
674:   END_HANDLE_TH_ERRORS
675: }
676: 
677: static PyObject* THPDivBackward2_other_raw_getter(THPCppFunction *self, void *_unused) {
678:   HANDLE_TH_ERRORS
679:   const auto& prop = static_cast<DivBackward2*>(self->cdata.get())->other_;
680:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
681:   return obj.release().ptr();
682:   END_HANDLE_TH_ERRORS
683: }
684: 
685: static PyObject* THPDivBackward2_rounding_mode_getter(THPCppFunction *self, void *_unused) {
686:   HANDLE_TH_ERRORS
687:   auto opt_prop = static_cast<DivBackward2*>(self->cdata.get())->rounding_mode;
688:   if (!opt_prop.has_value()) {
689:     Py_RETURN_NONE;
690:   }
691:   auto prop = opt_prop.value();
692:   return PyUnicode_FromStringAndSize(prop.data(), prop.size());
693:   END_HANDLE_TH_ERRORS
694: }
695: 
696: static PyObject* THPDivBackward2_self_getter(THPCppFunction *self, void *_unused) {
697:   HANDLE_TH_ERRORS
698:   const auto& prop = static_cast<DivBackward2*>(self->cdata.get())->self_;
699:   return THPVariable_Wrap(prop.unpack(self->cdata));
700:   END_HANDLE_TH_ERRORS
701: }
702: 
703: static PyObject* THPDivBackward2_self_raw_getter(THPCppFunction *self, void *_unused) {
704:   HANDLE_TH_ERRORS
705:   const auto& prop = static_cast<DivBackward2*>(self->cdata.get())->self_;
706:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
707:   return obj.release().ptr();
708:   END_HANDLE_TH_ERRORS
709: }
710: 
711: static struct PyGetSetDef DivBackward2_properties[] = {
712:   THP_FUNCTION_DEFAULT_PROPERTIES,
713:   {(char*)"_saved_other", (getter)THPDivBackward2_other_getter, nullptr, nullptr, nullptr},
714:   {(char*)"_raw_saved_other", (getter)THPDivBackward2_other_raw_getter, nullptr, nullptr, nullptr},
715:   {(char*)"_saved_rounding_mode", (getter)THPDivBackward2_rounding_mode_getter, nullptr, nullptr, nullptr},
716:   {(char*)"_saved_self", (getter)THPDivBackward2_self_getter, nullptr, nullptr, nullptr},
717:   {(char*)"_raw_saved_self", (getter)THPDivBackward2_self_raw_getter, nullptr, nullptr, nullptr},
718:   {nullptr} /* sentinel */
719: };
720: 
```

- EN: The main execution path in this span is carried by `THPDivBackward1_other_getter`, `PyComplex_FromDoubles`, `PyFloat_FromDouble`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPDivBackward1_other_getter`, `PyComplex_FromDoubles`, `PyFloat_FromDouble` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 721-800

```cpp
721: static PyObject* THPDivBackward3_other_getter(THPCppFunction *self, void *_unused) {
722:   HANDLE_TH_ERRORS
723:   auto prop = static_cast<DivBackward3*>(self->cdata.get())->other;
724:   if (prop.isComplex()) {
725:     auto cprop = prop.to<c10::complex<double>>();
726:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
727:   } else if (prop.isFloatingPoint()) {
728:     return PyFloat_FromDouble(prop.to<double>());
729:   } else if (prop.isIntegral(/*includeBool=*/false)) {
730:     return PyLong_FromLong(prop.to<int64_t>());
731:   } else if (prop.isBoolean()) {
732:     if (prop.to<bool>()) {
733:       Py_RETURN_TRUE;
734:     } else {
735:       Py_RETURN_FALSE;
736:     }
737:   } else {
738:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
739:     return nullptr;
740:   }
741:   END_HANDLE_TH_ERRORS
742: }
743: 
744: static PyObject* THPDivBackward3_rounding_mode_getter(THPCppFunction *self, void *_unused) {
745:   HANDLE_TH_ERRORS
746:   auto opt_prop = static_cast<DivBackward3*>(self->cdata.get())->rounding_mode;
747:   if (!opt_prop.has_value()) {
748:     Py_RETURN_NONE;
749:   }
750:   auto prop = opt_prop.value();
751:   return PyUnicode_FromStringAndSize(prop.data(), prop.size());
752:   END_HANDLE_TH_ERRORS
753: }
754: 
755: static struct PyGetSetDef DivBackward3_properties[] = {
756:   THP_FUNCTION_DEFAULT_PROPERTIES,
757:   {(char*)"_saved_other", (getter)THPDivBackward3_other_getter, nullptr, nullptr, nullptr},
758:   {(char*)"_saved_rounding_mode", (getter)THPDivBackward3_rounding_mode_getter, nullptr, nullptr, nullptr},
759:   {nullptr} /* sentinel */
760: };
761: 
762: static PyObject* THPErfBackward0_self_getter(THPCppFunction *self, void *_unused) {
763:   HANDLE_TH_ERRORS
764:   const auto& prop = static_cast<ErfBackward0*>(self->cdata.get())->self_;
765:   return THPVariable_Wrap(prop.unpack(self->cdata));
766:   END_HANDLE_TH_ERRORS
767: }
768: 
769: static PyObject* THPErfBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
770:   HANDLE_TH_ERRORS
771:   const auto& prop = static_cast<ErfBackward0*>(self->cdata.get())->self_;
772:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
773:   return obj.release().ptr();
774:   END_HANDLE_TH_ERRORS
775: }
776: 
777: static struct PyGetSetDef ErfBackward0_properties[] = {
778:   THP_FUNCTION_DEFAULT_PROPERTIES,
779:   {(char*)"_saved_self", (getter)THPErfBackward0_self_getter, nullptr, nullptr, nullptr},
780:   {(char*)"_raw_saved_self", (getter)THPErfBackward0_self_raw_getter, nullptr, nullptr, nullptr},
781:   {nullptr} /* sentinel */
782: };
783: 
784: static PyObject* THPFakeQuantizeLearnablePerChannelAffineBackward0_axis_getter(THPCppFunction *self, void *_unused) {
785:   HANDLE_TH_ERRORS
786:   auto prop = static_cast<FakeQuantizeLearnablePerChannelAffineBackward0*>(self->cdata.get())->axis;
787:   return PyLong_FromUnsignedLong((int64_t) prop);
788:   END_HANDLE_TH_ERRORS
789: }
790: 
791: static PyObject* THPFakeQuantizeLearnablePerChannelAffineBackward0_grad_factor_getter(THPCppFunction *self, void *_unused) {
792:   HANDLE_TH_ERRORS
793:   auto prop = static_cast<FakeQuantizeLearnablePerChannelAffineBackward0*>(self->cdata.get())->grad_factor;
794:   return PyFloat_FromDouble((double) prop);
795:   END_HANDLE_TH_ERRORS
796: }
797: 
798: static PyObject* THPFakeQuantizeLearnablePerChannelAffineBackward0_quant_max_getter(THPCppFunction *self, void *_unused) {
799:   HANDLE_TH_ERRORS
800:   auto prop = static_cast<FakeQuantizeLearnablePerChannelAffineBackward0*>(self->cdata.get())->quant_max;
```

- EN: The main execution path in this span is carried by `THPDivBackward3_other_getter`, `PyComplex_FromDoubles`, `PyFloat_FromDouble`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPDivBackward3_other_getter`, `PyComplex_FromDoubles`, `PyFloat_FromDouble` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 801-880

```cpp
801:   return PyLong_FromUnsignedLong((int64_t) prop);
802:   END_HANDLE_TH_ERRORS
803: }
804: 
805: static PyObject* THPFakeQuantizeLearnablePerChannelAffineBackward0_quant_min_getter(THPCppFunction *self, void *_unused) {
806:   HANDLE_TH_ERRORS
807:   auto prop = static_cast<FakeQuantizeLearnablePerChannelAffineBackward0*>(self->cdata.get())->quant_min;
808:   return PyLong_FromUnsignedLong((int64_t) prop);
809:   END_HANDLE_TH_ERRORS
810: }
811: 
812: static PyObject* THPFakeQuantizeLearnablePerChannelAffineBackward0_scale_getter(THPCppFunction *self, void *_unused) {
813:   HANDLE_TH_ERRORS
814:   const auto& prop = static_cast<FakeQuantizeLearnablePerChannelAffineBackward0*>(self->cdata.get())->scale_;
815:   return THPVariable_Wrap(prop.unpack(self->cdata));
816:   END_HANDLE_TH_ERRORS
817: }
818: 
819: static PyObject* THPFakeQuantizeLearnablePerChannelAffineBackward0_scale_raw_getter(THPCppFunction *self, void *_unused) {
820:   HANDLE_TH_ERRORS
821:   const auto& prop = static_cast<FakeQuantizeLearnablePerChannelAffineBackward0*>(self->cdata.get())->scale_;
822:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
823:   return obj.release().ptr();
824:   END_HANDLE_TH_ERRORS
825: }
826: 
827: static PyObject* THPFakeQuantizeLearnablePerChannelAffineBackward0_self_getter(THPCppFunction *self, void *_unused) {
828:   HANDLE_TH_ERRORS
829:   const auto& prop = static_cast<FakeQuantizeLearnablePerChannelAffineBackward0*>(self->cdata.get())->self_;
830:   return THPVariable_Wrap(prop.unpack(self->cdata));
831:   END_HANDLE_TH_ERRORS
832: }
833: 
834: static PyObject* THPFakeQuantizeLearnablePerChannelAffineBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
835:   HANDLE_TH_ERRORS
836:   const auto& prop = static_cast<FakeQuantizeLearnablePerChannelAffineBackward0*>(self->cdata.get())->self_;
837:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
838:   return obj.release().ptr();
839:   END_HANDLE_TH_ERRORS
840: }
841: 
842: static PyObject* THPFakeQuantizeLearnablePerChannelAffineBackward0_zero_point_getter(THPCppFunction *self, void *_unused) {
843:   HANDLE_TH_ERRORS
844:   const auto& prop = static_cast<FakeQuantizeLearnablePerChannelAffineBackward0*>(self->cdata.get())->zero_point_;
845:   return THPVariable_Wrap(prop.unpack(self->cdata));
846:   END_HANDLE_TH_ERRORS
847: }
848: 
849: static PyObject* THPFakeQuantizeLearnablePerChannelAffineBackward0_zero_point_raw_getter(THPCppFunction *self, void *_unused) {
850:   HANDLE_TH_ERRORS
851:   const auto& prop = static_cast<FakeQuantizeLearnablePerChannelAffineBackward0*>(self->cdata.get())->zero_point_;
852:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
853:   return obj.release().ptr();
854:   END_HANDLE_TH_ERRORS
855: }
856: 
857: static struct PyGetSetDef FakeQuantizeLearnablePerChannelAffineBackward0_properties[] = {
858:   THP_FUNCTION_DEFAULT_PROPERTIES,
859:   {(char*)"_saved_axis", (getter)THPFakeQuantizeLearnablePerChannelAffineBackward0_axis_getter, nullptr, nullptr, nullptr},
860:   {(char*)"_saved_grad_factor", (getter)THPFakeQuantizeLearnablePerChannelAffineBackward0_grad_factor_getter, nullptr, nullptr, nullptr},
861:   {(char*)"_saved_quant_max", (getter)THPFakeQuantizeLearnablePerChannelAffineBackward0_quant_max_getter, nullptr, nullptr, nullptr},
862:   {(char*)"_saved_quant_min", (getter)THPFakeQuantizeLearnablePerChannelAffineBackward0_quant_min_getter, nullptr, nullptr, nullptr},
863:   {(char*)"_saved_scale", (getter)THPFakeQuantizeLearnablePerChannelAffineBackward0_scale_getter, nullptr, nullptr, nullptr},
864:   {(char*)"_raw_saved_scale", (getter)THPFakeQuantizeLearnablePerChannelAffineBackward0_scale_raw_getter, nullptr, nullptr, nullptr},
865:   {(char*)"_saved_self", (getter)THPFakeQuantizeLearnablePerChannelAffineBackward0_self_getter, nullptr, nullptr, nullptr},
866:   {(char*)"_raw_saved_self", (getter)THPFakeQuantizeLearnablePerChannelAffineBackward0_self_raw_getter, nullptr, nullptr, nullptr},
867:   {(char*)"_saved_zero_point", (getter)THPFakeQuantizeLearnablePerChannelAffineBackward0_zero_point_getter, nullptr, nullptr, nullptr},
868:   {(char*)"_raw_saved_zero_point", (getter)THPFakeQuantizeLearnablePerChannelAffineBackward0_zero_point_raw_getter, nullptr, nullptr, nullptr},
869:   {nullptr} /* sentinel */
870: };
871: 
872: 
873: 
874: static struct PyGetSetDef FillBackward2_properties[] = {
875:   THP_FUNCTION_DEFAULT_PROPERTIES,
876: 
877:   {nullptr} /* sentinel */
878: };
879: 
880: 
```

- EN: The main execution path in this span is carried by `PyLong_FromUnsignedLong`, `THPFakeQuantizeLearnablePerChannelAffineBackward0_quant_min_getter`, `THPFakeQuantizeLearnablePerChannelAffineBackward0_scale_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyLong_FromUnsignedLong`, `THPFakeQuantizeLearnablePerChannelAffineBackward0_quant_min_getter`, `THPFakeQuantizeLearnablePerChannelAffineBackward0_scale_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 881-960

```cpp
881: 
882: static struct PyGetSetDef FillBackward3_properties[] = {
883:   THP_FUNCTION_DEFAULT_PROPERTIES,
884: 
885:   {nullptr} /* sentinel */
886: };
887: 
888: static PyObject* THPGridSampler3DBackward0_align_corners_getter(THPCppFunction *self, void *_unused) {
889:   HANDLE_TH_ERRORS
890:   auto prop = static_cast<GridSampler3DBackward0*>(self->cdata.get())->align_corners;
891:   if (prop) {
892:     Py_RETURN_TRUE;
893:   } else {
894:     Py_RETURN_FALSE;
895:   }
896:   END_HANDLE_TH_ERRORS
897: }
898: 
899: static PyObject* THPGridSampler3DBackward0_grid_getter(THPCppFunction *self, void *_unused) {
900:   HANDLE_TH_ERRORS
901:   const auto& prop = static_cast<GridSampler3DBackward0*>(self->cdata.get())->grid_;
902:   return THPVariable_Wrap(prop.unpack(self->cdata));
903:   END_HANDLE_TH_ERRORS
904: }
905: 
906: static PyObject* THPGridSampler3DBackward0_grid_raw_getter(THPCppFunction *self, void *_unused) {
907:   HANDLE_TH_ERRORS
908:   const auto& prop = static_cast<GridSampler3DBackward0*>(self->cdata.get())->grid_;
909:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
910:   return obj.release().ptr();
911:   END_HANDLE_TH_ERRORS
912: }
913: 
914: static PyObject* THPGridSampler3DBackward0_input_getter(THPCppFunction *self, void *_unused) {
915:   HANDLE_TH_ERRORS
916:   const auto& prop = static_cast<GridSampler3DBackward0*>(self->cdata.get())->input_;
917:   return THPVariable_Wrap(prop.unpack(self->cdata));
918:   END_HANDLE_TH_ERRORS
919: }
920: 
921: static PyObject* THPGridSampler3DBackward0_input_raw_getter(THPCppFunction *self, void *_unused) {
922:   HANDLE_TH_ERRORS
923:   const auto& prop = static_cast<GridSampler3DBackward0*>(self->cdata.get())->input_;
924:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
925:   return obj.release().ptr();
926:   END_HANDLE_TH_ERRORS
927: }
928: 
929: static PyObject* THPGridSampler3DBackward0_interpolation_mode_getter(THPCppFunction *self, void *_unused) {
930:   HANDLE_TH_ERRORS
931:   auto prop = static_cast<GridSampler3DBackward0*>(self->cdata.get())->interpolation_mode;
932:   return PyLong_FromUnsignedLong((int64_t) prop);
933:   END_HANDLE_TH_ERRORS
934: }
935: 
936: static PyObject* THPGridSampler3DBackward0_padding_mode_getter(THPCppFunction *self, void *_unused) {
937:   HANDLE_TH_ERRORS
938:   auto prop = static_cast<GridSampler3DBackward0*>(self->cdata.get())->padding_mode;
939:   return PyLong_FromUnsignedLong((int64_t) prop);
940:   END_HANDLE_TH_ERRORS
941: }
942: 
943: static struct PyGetSetDef GridSampler3DBackward0_properties[] = {
944:   THP_FUNCTION_DEFAULT_PROPERTIES,
945:   {(char*)"_saved_align_corners", (getter)THPGridSampler3DBackward0_align_corners_getter, nullptr, nullptr, nullptr},
946:   {(char*)"_saved_grid", (getter)THPGridSampler3DBackward0_grid_getter, nullptr, nullptr, nullptr},
947:   {(char*)"_raw_saved_grid", (getter)THPGridSampler3DBackward0_grid_raw_getter, nullptr, nullptr, nullptr},
948:   {(char*)"_saved_input", (getter)THPGridSampler3DBackward0_input_getter, nullptr, nullptr, nullptr},
949:   {(char*)"_raw_saved_input", (getter)THPGridSampler3DBackward0_input_raw_getter, nullptr, nullptr, nullptr},
950:   {(char*)"_saved_interpolation_mode", (getter)THPGridSampler3DBackward0_interpolation_mode_getter, nullptr, nullptr, nullptr},
951:   {(char*)"_saved_padding_mode", (getter)THPGridSampler3DBackward0_padding_mode_getter, nullptr, nullptr, nullptr},
952:   {nullptr} /* sentinel */
953: };
954: 
955: static PyObject* THPGridSampler2DCpuFallbackBackward0_align_corners_getter(THPCppFunction *self, void *_unused) {
956:   HANDLE_TH_ERRORS
957:   auto prop = static_cast<GridSampler2DCpuFallbackBackward0*>(self->cdata.get())->align_corners;
958:   if (prop) {
959:     Py_RETURN_TRUE;
960:   } else {
```

- EN: The main execution path in this span is carried by `THPGridSampler3DBackward0_align_corners_getter`, `THPGridSampler3DBackward0_grid_getter`, `THPVariable_Wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPGridSampler3DBackward0_align_corners_getter`, `THPGridSampler3DBackward0_grid_getter`, `THPVariable_Wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 961-1040

```cpp
 961:     Py_RETURN_FALSE;
 962:   }
 963:   END_HANDLE_TH_ERRORS
 964: }
 965: 
 966: static PyObject* THPGridSampler2DCpuFallbackBackward0_grid_getter(THPCppFunction *self, void *_unused) {
 967:   HANDLE_TH_ERRORS
 968:   const auto& prop = static_cast<GridSampler2DCpuFallbackBackward0*>(self->cdata.get())->grid_;
 969:   return THPVariable_Wrap(prop.unpack(self->cdata));
 970:   END_HANDLE_TH_ERRORS
 971: }
 972: 
 973: static PyObject* THPGridSampler2DCpuFallbackBackward0_grid_raw_getter(THPCppFunction *self, void *_unused) {
 974:   HANDLE_TH_ERRORS
 975:   const auto& prop = static_cast<GridSampler2DCpuFallbackBackward0*>(self->cdata.get())->grid_;
 976:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
 977:   return obj.release().ptr();
 978:   END_HANDLE_TH_ERRORS
 979: }
 980: 
 981: static PyObject* THPGridSampler2DCpuFallbackBackward0_input_getter(THPCppFunction *self, void *_unused) {
 982:   HANDLE_TH_ERRORS
 983:   const auto& prop = static_cast<GridSampler2DCpuFallbackBackward0*>(self->cdata.get())->input_;
 984:   return THPVariable_Wrap(prop.unpack(self->cdata));
 985:   END_HANDLE_TH_ERRORS
 986: }
 987: 
 988: static PyObject* THPGridSampler2DCpuFallbackBackward0_input_raw_getter(THPCppFunction *self, void *_unused) {
 989:   HANDLE_TH_ERRORS
 990:   const auto& prop = static_cast<GridSampler2DCpuFallbackBackward0*>(self->cdata.get())->input_;
 991:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
 992:   return obj.release().ptr();
 993:   END_HANDLE_TH_ERRORS
 994: }
 995: 
 996: static PyObject* THPGridSampler2DCpuFallbackBackward0_interpolation_mode_getter(THPCppFunction *self, void *_unused) {
 997:   HANDLE_TH_ERRORS
 998:   auto prop = static_cast<GridSampler2DCpuFallbackBackward0*>(self->cdata.get())->interpolation_mode;
 999:   return PyLong_FromUnsignedLong((int64_t) prop);
1000:   END_HANDLE_TH_ERRORS
1001: }
1002: 
1003: static PyObject* THPGridSampler2DCpuFallbackBackward0_padding_mode_getter(THPCppFunction *self, void *_unused) {
1004:   HANDLE_TH_ERRORS
1005:   auto prop = static_cast<GridSampler2DCpuFallbackBackward0*>(self->cdata.get())->padding_mode;
1006:   return PyLong_FromUnsignedLong((int64_t) prop);
1007:   END_HANDLE_TH_ERRORS
1008: }
1009: 
1010: static struct PyGetSetDef GridSampler2DCpuFallbackBackward0_properties[] = {
1011:   THP_FUNCTION_DEFAULT_PROPERTIES,
1012:   {(char*)"_saved_align_corners", (getter)THPGridSampler2DCpuFallbackBackward0_align_corners_getter, nullptr, nullptr, nullptr},
1013:   {(char*)"_saved_grid", (getter)THPGridSampler2DCpuFallbackBackward0_grid_getter, nullptr, nullptr, nullptr},
1014:   {(char*)"_raw_saved_grid", (getter)THPGridSampler2DCpuFallbackBackward0_grid_raw_getter, nullptr, nullptr, nullptr},
1015:   {(char*)"_saved_input", (getter)THPGridSampler2DCpuFallbackBackward0_input_getter, nullptr, nullptr, nullptr},
1016:   {(char*)"_raw_saved_input", (getter)THPGridSampler2DCpuFallbackBackward0_input_raw_getter, nullptr, nullptr, nullptr},
1017:   {(char*)"_saved_interpolation_mode", (getter)THPGridSampler2DCpuFallbackBackward0_interpolation_mode_getter, nullptr, nullptr, nullptr},
1018:   {(char*)"_saved_padding_mode", (getter)THPGridSampler2DCpuFallbackBackward0_padding_mode_getter, nullptr, nullptr, nullptr},
1019:   {nullptr} /* sentinel */
1020: };
1021: 
1022: static PyObject* THPHardswishBackwardBackward0_grad_output_getter(THPCppFunction *self, void *_unused) {
1023:   HANDLE_TH_ERRORS
1024:   const auto& prop = static_cast<HardswishBackwardBackward0*>(self->cdata.get())->grad_output_;
1025:   return THPVariable_Wrap(prop.unpack(self->cdata));
1026:   END_HANDLE_TH_ERRORS
1027: }
1028: 
1029: static PyObject* THPHardswishBackwardBackward0_grad_output_raw_getter(THPCppFunction *self, void *_unused) {
1030:   HANDLE_TH_ERRORS
1031:   const auto& prop = static_cast<HardswishBackwardBackward0*>(self->cdata.get())->grad_output_;
1032:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1033:   return obj.release().ptr();
1034:   END_HANDLE_TH_ERRORS
1035: }
1036: 
1037: static PyObject* THPHardswishBackwardBackward0_self_getter(THPCppFunction *self, void *_unused) {
1038:   HANDLE_TH_ERRORS
1039:   const auto& prop = static_cast<HardswishBackwardBackward0*>(self->cdata.get())->self_;
1040:   return THPVariable_Wrap(prop.unpack(self->cdata));
```

- EN: The main execution path in this span is carried by `THPGridSampler2DCpuFallbackBackward0_grid_getter`, `THPVariable_Wrap`, `THPGridSampler2DCpuFallbackBackward0_grid_raw_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPGridSampler2DCpuFallbackBackward0_grid_getter`, `THPVariable_Wrap`, `THPGridSampler2DCpuFallbackBackward0_grid_raw_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1041-1120

```cpp
1041:   END_HANDLE_TH_ERRORS
1042: }
1043: 
1044: static PyObject* THPHardswishBackwardBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
1045:   HANDLE_TH_ERRORS
1046:   const auto& prop = static_cast<HardswishBackwardBackward0*>(self->cdata.get())->self_;
1047:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1048:   return obj.release().ptr();
1049:   END_HANDLE_TH_ERRORS
1050: }
1051: 
1052: static struct PyGetSetDef HardswishBackwardBackward0_properties[] = {
1053:   THP_FUNCTION_DEFAULT_PROPERTIES,
1054:   {(char*)"_saved_grad_output", (getter)THPHardswishBackwardBackward0_grad_output_getter, nullptr, nullptr, nullptr},
1055:   {(char*)"_raw_saved_grad_output", (getter)THPHardswishBackwardBackward0_grad_output_raw_getter, nullptr, nullptr, nullptr},
1056:   {(char*)"_saved_self", (getter)THPHardswishBackwardBackward0_self_getter, nullptr, nullptr, nullptr},
1057:   {(char*)"_raw_saved_self", (getter)THPHardswishBackwardBackward0_self_raw_getter, nullptr, nullptr, nullptr},
1058:   {nullptr} /* sentinel */
1059: };
1060: 
1061: static PyObject* THPIndexBackward0_indices_getter(THPCppFunction *self, void *_unused) {
1062:   HANDLE_TH_ERRORS
1063:   const auto *node = static_cast<IndexBackward0*>(self->cdata.get());
1064:   const auto& prop = node->indices_;
1065:   if (node->indices_released_) {
1066:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
1067:     return nullptr;
1068:   }
1069:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
1070:   for (auto i: c10::irange(prop.size())) {
1071:     PyTuple_SetItem(tup, (Py_ssize_t) i, THPVariable_Wrap(prop[i].unpack(self->cdata)));
1072:   }
1073:   return tup;
1074:   END_HANDLE_TH_ERRORS
1075: }
1076: 
1077: static PyObject* THPIndexBackward0_indices_raw_getter(THPCppFunction *self, void *_unused) {
1078:   HANDLE_TH_ERRORS
1079:   const auto *node = static_cast<IndexBackward0*>(self->cdata.get());
1080:   const auto& prop = node->indices_;
1081:   if (node->indices_released_) {
1082:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
1083:     return nullptr;
1084:   }
1085:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
1086:   for (auto i : c10::irange(prop.size())) {
1087:     pybind11::object obj = pybind11::cast(prop[i], pybind11::return_value_policy::reference);
1088:     PyTuple_SetItem(tup, (Py_ssize_t) i, obj.release().ptr());
1089:   }
1090:   return tup;
1091:   END_HANDLE_TH_ERRORS
1092: }
1093: 
1094: static PyObject* THPIndexBackward0_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
1095:   HANDLE_TH_ERRORS
1096:   auto prop = static_cast<IndexBackward0*>(self->cdata.get())->self_sym_sizes;
1097:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
1098:   for (auto i : c10::irange(prop.size())) {
1099:       auto si = prop[i];
1100:       if (auto m = si.maybe_as_int()) {
1101:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
1102:       } else {
1103:         auto py_symint = py::cast(si).release().ptr();
1104:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
1105:       }
1106:   }
1107:   return tup;
1108:   END_HANDLE_TH_ERRORS
1109: }
1110: 
1111: static struct PyGetSetDef IndexBackward0_properties[] = {
1112:   THP_FUNCTION_DEFAULT_PROPERTIES,
1113:   {(char*)"_saved_indices", (getter)THPIndexBackward0_indices_getter, nullptr, nullptr, nullptr},
1114:   {(char*)"_raw_saved_indices", (getter)THPIndexBackward0_indices_raw_getter, nullptr, nullptr, nullptr},
1115:   {(char*)"_saved_self_sym_sizes", (getter)THPIndexBackward0_self_sym_sizes_getter, nullptr, nullptr, nullptr},
1116:   {nullptr} /* sentinel */
1117: };
1118: 
1119: static PyObject* THPIndexReduceBackward0_dim_getter(THPCppFunction *self, void *_unused) {
1120:   HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPHardswishBackwardBackward0_self_raw_getter`, `cast`, `THPIndexBackward0_indices_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPHardswishBackwardBackward0_self_raw_getter`, `cast`, `THPIndexBackward0_indices_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1121-1200

```cpp
1121:   auto prop = static_cast<IndexReduceBackward0*>(self->cdata.get())->dim;
1122:   return PyLong_FromUnsignedLong((int64_t) prop);
1123:   END_HANDLE_TH_ERRORS
1124: }
1125: 
1126: static PyObject* THPIndexReduceBackward0_include_self_getter(THPCppFunction *self, void *_unused) {
1127:   HANDLE_TH_ERRORS
1128:   auto prop = static_cast<IndexReduceBackward0*>(self->cdata.get())->include_self;
1129:   if (prop) {
1130:     Py_RETURN_TRUE;
1131:   } else {
1132:     Py_RETURN_FALSE;
1133:   }
1134:   END_HANDLE_TH_ERRORS
1135: }
1136: 
1137: static PyObject* THPIndexReduceBackward0_index_getter(THPCppFunction *self, void *_unused) {
1138:   HANDLE_TH_ERRORS
1139:   const auto& prop = static_cast<IndexReduceBackward0*>(self->cdata.get())->index_;
1140:   return THPVariable_Wrap(prop.unpack(self->cdata));
1141:   END_HANDLE_TH_ERRORS
1142: }
1143: 
1144: static PyObject* THPIndexReduceBackward0_index_raw_getter(THPCppFunction *self, void *_unused) {
1145:   HANDLE_TH_ERRORS
1146:   const auto& prop = static_cast<IndexReduceBackward0*>(self->cdata.get())->index_;
1147:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1148:   return obj.release().ptr();
1149:   END_HANDLE_TH_ERRORS
1150: }
1151: 
1152: static PyObject* THPIndexReduceBackward0_reduce_getter(THPCppFunction *self, void *_unused) {
1153:   HANDLE_TH_ERRORS
1154:   auto prop = static_cast<IndexReduceBackward0*>(self->cdata.get())->reduce;
1155:   return PyUnicode_FromStringAndSize(prop.data(), prop.size());
1156:   END_HANDLE_TH_ERRORS
1157: }
1158: 
1159: static PyObject* THPIndexReduceBackward0_self_getter(THPCppFunction *self, void *_unused) {
1160:   HANDLE_TH_ERRORS
1161:   const auto& prop = static_cast<IndexReduceBackward0*>(self->cdata.get())->self_;
1162:   return THPVariable_Wrap(prop.unpack(self->cdata));
1163:   END_HANDLE_TH_ERRORS
1164: }
1165: 
1166: static PyObject* THPIndexReduceBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
1167:   HANDLE_TH_ERRORS
1168:   const auto& prop = static_cast<IndexReduceBackward0*>(self->cdata.get())->self_;
1169:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1170:   return obj.release().ptr();
1171:   END_HANDLE_TH_ERRORS
1172: }
1173: 
1174: static PyObject* THPIndexReduceBackward0_source_getter(THPCppFunction *self, void *_unused) {
1175:   HANDLE_TH_ERRORS
1176:   const auto& prop = static_cast<IndexReduceBackward0*>(self->cdata.get())->source_;
1177:   return THPVariable_Wrap(prop.unpack(self->cdata));
1178:   END_HANDLE_TH_ERRORS
1179: }
1180: 
1181: static PyObject* THPIndexReduceBackward0_source_raw_getter(THPCppFunction *self, void *_unused) {
1182:   HANDLE_TH_ERRORS
1183:   const auto& prop = static_cast<IndexReduceBackward0*>(self->cdata.get())->source_;
1184:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1185:   return obj.release().ptr();
1186:   END_HANDLE_TH_ERRORS
1187: }
1188: 
1189: static PyObject* THPIndexReduceBackward0_result_getter(THPCppFunction *self, void *_unused) {
1190:   HANDLE_TH_ERRORS
1191:   const auto& prop = static_cast<IndexReduceBackward0*>(self->cdata.get())->result_;
1192:   return THPVariable_Wrap(prop.unpack(self->cdata));
1193:   END_HANDLE_TH_ERRORS
1194: }
1195: 
1196: static PyObject* THPIndexReduceBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
1197:   HANDLE_TH_ERRORS
1198:   const auto& prop = static_cast<IndexReduceBackward0*>(self->cdata.get())->result_;
1199:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1200:   return obj.release().ptr();
```

- EN: The main execution path in this span is carried by `PyLong_FromUnsignedLong`, `THPIndexReduceBackward0_include_self_getter`, `THPIndexReduceBackward0_index_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyLong_FromUnsignedLong`, `THPIndexReduceBackward0_include_self_getter`, `THPIndexReduceBackward0_index_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1201-1280

```cpp
1201:   END_HANDLE_TH_ERRORS
1202: }
1203: 
1204: static struct PyGetSetDef IndexReduceBackward0_properties[] = {
1205:   THP_FUNCTION_DEFAULT_PROPERTIES,
1206:   {(char*)"_saved_dim", (getter)THPIndexReduceBackward0_dim_getter, nullptr, nullptr, nullptr},
1207:   {(char*)"_saved_include_self", (getter)THPIndexReduceBackward0_include_self_getter, nullptr, nullptr, nullptr},
1208:   {(char*)"_saved_index", (getter)THPIndexReduceBackward0_index_getter, nullptr, nullptr, nullptr},
1209:   {(char*)"_raw_saved_index", (getter)THPIndexReduceBackward0_index_raw_getter, nullptr, nullptr, nullptr},
1210:   {(char*)"_saved_reduce", (getter)THPIndexReduceBackward0_reduce_getter, nullptr, nullptr, nullptr},
1211:   {(char*)"_saved_self", (getter)THPIndexReduceBackward0_self_getter, nullptr, nullptr, nullptr},
1212:   {(char*)"_raw_saved_self", (getter)THPIndexReduceBackward0_self_raw_getter, nullptr, nullptr, nullptr},
1213:   {(char*)"_saved_source", (getter)THPIndexReduceBackward0_source_getter, nullptr, nullptr, nullptr},
1214:   {(char*)"_raw_saved_source", (getter)THPIndexReduceBackward0_source_raw_getter, nullptr, nullptr, nullptr},
1215:   {(char*)"_saved_result", (getter)THPIndexReduceBackward0_result_getter, nullptr, nullptr, nullptr},
1216:   {(char*)"_raw_saved_result", (getter)THPIndexReduceBackward0_result_raw_getter, nullptr, nullptr, nullptr},
1217:   {nullptr} /* sentinel */
1218: };
1219: 
1220: static PyObject* THPIndexFillBackward0_dim_getter(THPCppFunction *self, void *_unused) {
1221:   HANDLE_TH_ERRORS
1222:   auto prop = static_cast<IndexFillBackward0*>(self->cdata.get())->dim;
1223:   return PyLong_FromUnsignedLong((int64_t) prop);
1224:   END_HANDLE_TH_ERRORS
1225: }
1226: 
1227: static PyObject* THPIndexFillBackward0_index_getter(THPCppFunction *self, void *_unused) {
1228:   HANDLE_TH_ERRORS
1229:   const auto& prop = static_cast<IndexFillBackward0*>(self->cdata.get())->index_;
1230:   return THPVariable_Wrap(prop.unpack(self->cdata));
1231:   END_HANDLE_TH_ERRORS
1232: }
1233: 
1234: static PyObject* THPIndexFillBackward0_index_raw_getter(THPCppFunction *self, void *_unused) {
1235:   HANDLE_TH_ERRORS
1236:   const auto& prop = static_cast<IndexFillBackward0*>(self->cdata.get())->index_;
1237:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1238:   return obj.release().ptr();
1239:   END_HANDLE_TH_ERRORS
1240: }
1241: 
1242: static struct PyGetSetDef IndexFillBackward0_properties[] = {
1243:   THP_FUNCTION_DEFAULT_PROPERTIES,
1244:   {(char*)"_saved_dim", (getter)THPIndexFillBackward0_dim_getter, nullptr, nullptr, nullptr},
1245:   {(char*)"_saved_index", (getter)THPIndexFillBackward0_index_getter, nullptr, nullptr, nullptr},
1246:   {(char*)"_raw_saved_index", (getter)THPIndexFillBackward0_index_raw_getter, nullptr, nullptr, nullptr},
1247:   {nullptr} /* sentinel */
1248: };
1249: 
1250: static PyObject* THPIndexFillBackward1_dim_getter(THPCppFunction *self, void *_unused) {
1251:   HANDLE_TH_ERRORS
1252:   auto prop = static_cast<IndexFillBackward1*>(self->cdata.get())->dim;
1253:   return PyLong_FromUnsignedLong((int64_t) prop);
1254:   END_HANDLE_TH_ERRORS
1255: }
1256: 
1257: static PyObject* THPIndexFillBackward1_index_getter(THPCppFunction *self, void *_unused) {
1258:   HANDLE_TH_ERRORS
1259:   const auto& prop = static_cast<IndexFillBackward1*>(self->cdata.get())->index_;
1260:   return THPVariable_Wrap(prop.unpack(self->cdata));
1261:   END_HANDLE_TH_ERRORS
1262: }
1263: 
1264: static PyObject* THPIndexFillBackward1_index_raw_getter(THPCppFunction *self, void *_unused) {
1265:   HANDLE_TH_ERRORS
1266:   const auto& prop = static_cast<IndexFillBackward1*>(self->cdata.get())->index_;
1267:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1268:   return obj.release().ptr();
1269:   END_HANDLE_TH_ERRORS
1270: }
1271: 
1272: static struct PyGetSetDef IndexFillBackward1_properties[] = {
1273:   THP_FUNCTION_DEFAULT_PROPERTIES,
1274:   {(char*)"_saved_dim", (getter)THPIndexFillBackward1_dim_getter, nullptr, nullptr, nullptr},
1275:   {(char*)"_saved_index", (getter)THPIndexFillBackward1_index_getter, nullptr, nullptr, nullptr},
1276:   {(char*)"_raw_saved_index", (getter)THPIndexFillBackward1_index_raw_getter, nullptr, nullptr, nullptr},
1277:   {nullptr} /* sentinel */
1278: };
1279: 
1280: static PyObject* THPIndexPutImplBackward0_accumulate_getter(THPCppFunction *self, void *_unused) {
```

- EN: The main execution path in this span is carried by `THPIndexFillBackward0_dim_getter`, `PyLong_FromUnsignedLong`, `THPIndexFillBackward0_index_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPIndexFillBackward0_dim_getter`, `PyLong_FromUnsignedLong`, `THPIndexFillBackward0_index_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1281-1360

```cpp
1281:   HANDLE_TH_ERRORS
1282:   auto prop = static_cast<IndexPutImplBackward0*>(self->cdata.get())->accumulate;
1283:   if (prop) {
1284:     Py_RETURN_TRUE;
1285:   } else {
1286:     Py_RETURN_FALSE;
1287:   }
1288:   END_HANDLE_TH_ERRORS
1289: }
1290: 
1291: static PyObject* THPIndexPutImplBackward0_indices_getter(THPCppFunction *self, void *_unused) {
1292:   HANDLE_TH_ERRORS
1293:   const auto *node = static_cast<IndexPutImplBackward0*>(self->cdata.get());
1294:   const auto& prop = node->indices_;
1295:   if (node->indices_released_) {
1296:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
1297:     return nullptr;
1298:   }
1299:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
1300:   for (auto i: c10::irange(prop.size())) {
1301:     PyTuple_SetItem(tup, (Py_ssize_t) i, THPVariable_Wrap(prop[i].unpack(self->cdata)));
1302:   }
1303:   return tup;
1304:   END_HANDLE_TH_ERRORS
1305: }
1306: 
1307: static PyObject* THPIndexPutImplBackward0_indices_raw_getter(THPCppFunction *self, void *_unused) {
1308:   HANDLE_TH_ERRORS
1309:   const auto *node = static_cast<IndexPutImplBackward0*>(self->cdata.get());
1310:   const auto& prop = node->indices_;
1311:   if (node->indices_released_) {
1312:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
1313:     return nullptr;
1314:   }
1315:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
1316:   for (auto i : c10::irange(prop.size())) {
1317:     pybind11::object obj = pybind11::cast(prop[i], pybind11::return_value_policy::reference);
1318:     PyTuple_SetItem(tup, (Py_ssize_t) i, obj.release().ptr());
1319:   }
1320:   return tup;
1321:   END_HANDLE_TH_ERRORS
1322: }
1323: 
1324: static struct PyGetSetDef IndexPutImplBackward0_properties[] = {
1325:   THP_FUNCTION_DEFAULT_PROPERTIES,
1326:   {(char*)"_saved_accumulate", (getter)THPIndexPutImplBackward0_accumulate_getter, nullptr, nullptr, nullptr},
1327:   {(char*)"_saved_indices", (getter)THPIndexPutImplBackward0_indices_getter, nullptr, nullptr, nullptr},
1328:   {(char*)"_raw_saved_indices", (getter)THPIndexPutImplBackward0_indices_raw_getter, nullptr, nullptr, nullptr},
1329:   {nullptr} /* sentinel */
1330: };
1331: 
1332: static PyObject* THPLinalgPinvBackward0_self_getter(THPCppFunction *self, void *_unused) {
1333:   HANDLE_TH_ERRORS
1334:   const auto& prop = static_cast<LinalgPinvBackward0*>(self->cdata.get())->self_;
1335:   return THPVariable_Wrap(prop.unpack(self->cdata));
1336:   END_HANDLE_TH_ERRORS
1337: }
1338: 
1339: static PyObject* THPLinalgPinvBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
1340:   HANDLE_TH_ERRORS
1341:   const auto& prop = static_cast<LinalgPinvBackward0*>(self->cdata.get())->self_;
1342:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1343:   return obj.release().ptr();
1344:   END_HANDLE_TH_ERRORS
1345: }
1346: 
1347: static PyObject* THPLinalgPinvBackward0_result_getter(THPCppFunction *self, void *_unused) {
1348:   HANDLE_TH_ERRORS
1349:   const auto& prop = static_cast<LinalgPinvBackward0*>(self->cdata.get())->result_;
1350:   return THPVariable_Wrap(prop.unpack(self->cdata));
1351:   END_HANDLE_TH_ERRORS
1352: }
1353: 
1354: static PyObject* THPLinalgPinvBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
1355:   HANDLE_TH_ERRORS
1356:   const auto& prop = static_cast<LinalgPinvBackward0*>(self->cdata.get())->result_;
1357:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1358:   return obj.release().ptr();
1359:   END_HANDLE_TH_ERRORS
1360: }
```

- EN: The main execution path in this span is carried by `THPIndexPutImplBackward0_indices_getter`, `PyErr_SetString`, `PyTuple_New`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPIndexPutImplBackward0_indices_getter`, `PyErr_SetString`, `PyTuple_New` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1361-1440

```cpp
1361: 
1362: static struct PyGetSetDef LinalgPinvBackward0_properties[] = {
1363:   THP_FUNCTION_DEFAULT_PROPERTIES,
1364:   {(char*)"_saved_self", (getter)THPLinalgPinvBackward0_self_getter, nullptr, nullptr, nullptr},
1365:   {(char*)"_raw_saved_self", (getter)THPLinalgPinvBackward0_self_raw_getter, nullptr, nullptr, nullptr},
1366:   {(char*)"_saved_result", (getter)THPLinalgPinvBackward0_result_getter, nullptr, nullptr, nullptr},
1367:   {(char*)"_raw_saved_result", (getter)THPLinalgPinvBackward0_result_raw_getter, nullptr, nullptr, nullptr},
1368:   {nullptr} /* sentinel */
1369: };
1370: 
1371: static PyObject* THPKthvalueBackward0_dim_getter(THPCppFunction *self, void *_unused) {
1372:   HANDLE_TH_ERRORS
1373:   auto prop = static_cast<KthvalueBackward0*>(self->cdata.get())->dim;
1374:   return PyLong_FromUnsignedLong((int64_t) prop);
1375:   END_HANDLE_TH_ERRORS
1376: }
1377: 
1378: static PyObject* THPKthvalueBackward0_keepdim_getter(THPCppFunction *self, void *_unused) {
1379:   HANDLE_TH_ERRORS
1380:   auto prop = static_cast<KthvalueBackward0*>(self->cdata.get())->keepdim;
1381:   if (prop) {
1382:     Py_RETURN_TRUE;
1383:   } else {
1384:     Py_RETURN_FALSE;
1385:   }
1386:   END_HANDLE_TH_ERRORS
1387: }
1388: 
1389: static PyObject* THPKthvalueBackward0_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
1390:   HANDLE_TH_ERRORS
1391:   auto prop = static_cast<KthvalueBackward0*>(self->cdata.get())->self_sym_sizes;
1392:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
1393:   for (auto i : c10::irange(prop.size())) {
1394:       auto si = prop[i];
1395:       if (auto m = si.maybe_as_int()) {
1396:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
1397:       } else {
1398:         auto py_symint = py::cast(si).release().ptr();
1399:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
1400:       }
1401:   }
1402:   return tup;
1403:   END_HANDLE_TH_ERRORS
1404: }
1405: 
1406: static PyObject* THPKthvalueBackward0_indices_getter(THPCppFunction *self, void *_unused) {
1407:   HANDLE_TH_ERRORS
1408:   const auto& prop = static_cast<KthvalueBackward0*>(self->cdata.get())->indices_;
1409:   return THPVariable_Wrap(prop.unpack(self->cdata));
1410:   END_HANDLE_TH_ERRORS
1411: }
1412: 
1413: static PyObject* THPKthvalueBackward0_indices_raw_getter(THPCppFunction *self, void *_unused) {
1414:   HANDLE_TH_ERRORS
1415:   const auto& prop = static_cast<KthvalueBackward0*>(self->cdata.get())->indices_;
1416:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1417:   return obj.release().ptr();
1418:   END_HANDLE_TH_ERRORS
1419: }
1420: 
1421: static struct PyGetSetDef KthvalueBackward0_properties[] = {
1422:   THP_FUNCTION_DEFAULT_PROPERTIES,
1423:   {(char*)"_saved_dim", (getter)THPKthvalueBackward0_dim_getter, nullptr, nullptr, nullptr},
1424:   {(char*)"_saved_keepdim", (getter)THPKthvalueBackward0_keepdim_getter, nullptr, nullptr, nullptr},
1425:   {(char*)"_saved_self_sym_sizes", (getter)THPKthvalueBackward0_self_sym_sizes_getter, nullptr, nullptr, nullptr},
1426:   {(char*)"_saved_indices", (getter)THPKthvalueBackward0_indices_getter, nullptr, nullptr, nullptr},
1427:   {(char*)"_raw_saved_indices", (getter)THPKthvalueBackward0_indices_raw_getter, nullptr, nullptr, nullptr},
1428:   {nullptr} /* sentinel */
1429: };
1430: 
1431: static PyObject* THPDigammaBackward0_self_getter(THPCppFunction *self, void *_unused) {
1432:   HANDLE_TH_ERRORS
1433:   const auto& prop = static_cast<DigammaBackward0*>(self->cdata.get())->self_;
1434:   return THPVariable_Wrap(prop.unpack(self->cdata));
1435:   END_HANDLE_TH_ERRORS
1436: }
1437: 
1438: static PyObject* THPDigammaBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
1439:   HANDLE_TH_ERRORS
1440:   const auto& prop = static_cast<DigammaBackward0*>(self->cdata.get())->self_;
```

- EN: The main execution path in this span is carried by `THPKthvalueBackward0_dim_getter`, `PyLong_FromUnsignedLong`, `THPKthvalueBackward0_keepdim_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPKthvalueBackward0_dim_getter`, `PyLong_FromUnsignedLong`, `THPKthvalueBackward0_keepdim_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1441-1520

```cpp
1441:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1442:   return obj.release().ptr();
1443:   END_HANDLE_TH_ERRORS
1444: }
1445: 
1446: static struct PyGetSetDef DigammaBackward0_properties[] = {
1447:   THP_FUNCTION_DEFAULT_PROPERTIES,
1448:   {(char*)"_saved_self", (getter)THPDigammaBackward0_self_getter, nullptr, nullptr, nullptr},
1449:   {(char*)"_raw_saved_self", (getter)THPDigammaBackward0_self_raw_getter, nullptr, nullptr, nullptr},
1450:   {nullptr} /* sentinel */
1451: };
1452: 
1453: static PyObject* THPPolygammaBackward0_n_getter(THPCppFunction *self, void *_unused) {
1454:   HANDLE_TH_ERRORS
1455:   auto prop = static_cast<PolygammaBackward0*>(self->cdata.get())->n;
1456:   return PyLong_FromUnsignedLong((int64_t) prop);
1457:   END_HANDLE_TH_ERRORS
1458: }
1459: 
1460: static PyObject* THPPolygammaBackward0_self_getter(THPCppFunction *self, void *_unused) {
1461:   HANDLE_TH_ERRORS
1462:   const auto& prop = static_cast<PolygammaBackward0*>(self->cdata.get())->self_;
1463:   return THPVariable_Wrap(prop.unpack(self->cdata));
1464:   END_HANDLE_TH_ERRORS
1465: }
1466: 
1467: static PyObject* THPPolygammaBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
1468:   HANDLE_TH_ERRORS
1469:   const auto& prop = static_cast<PolygammaBackward0*>(self->cdata.get())->self_;
1470:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1471:   return obj.release().ptr();
1472:   END_HANDLE_TH_ERRORS
1473: }
1474: 
1475: static struct PyGetSetDef PolygammaBackward0_properties[] = {
1476:   THP_FUNCTION_DEFAULT_PROPERTIES,
1477:   {(char*)"_saved_n", (getter)THPPolygammaBackward0_n_getter, nullptr, nullptr, nullptr},
1478:   {(char*)"_saved_self", (getter)THPPolygammaBackward0_self_getter, nullptr, nullptr, nullptr},
1479:   {(char*)"_raw_saved_self", (getter)THPPolygammaBackward0_self_raw_getter, nullptr, nullptr, nullptr},
1480:   {nullptr} /* sentinel */
1481: };
1482: 
1483: static PyObject* THPPolygammaBackward1_n_getter(THPCppFunction *self, void *_unused) {
1484:   HANDLE_TH_ERRORS
1485:   auto prop = static_cast<PolygammaBackward1*>(self->cdata.get())->n;
1486:   return PyLong_FromUnsignedLong((int64_t) prop);
1487:   END_HANDLE_TH_ERRORS
1488: }
1489: 
1490: static PyObject* THPPolygammaBackward1_self_getter(THPCppFunction *self, void *_unused) {
1491:   HANDLE_TH_ERRORS
1492:   const auto& prop = static_cast<PolygammaBackward1*>(self->cdata.get())->self_;
1493:   return THPVariable_Wrap(prop.unpack(self->cdata));
1494:   END_HANDLE_TH_ERRORS
1495: }
1496: 
1497: static PyObject* THPPolygammaBackward1_self_raw_getter(THPCppFunction *self, void *_unused) {
1498:   HANDLE_TH_ERRORS
1499:   const auto& prop = static_cast<PolygammaBackward1*>(self->cdata.get())->self_;
1500:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1501:   return obj.release().ptr();
1502:   END_HANDLE_TH_ERRORS
1503: }
1504: 
1505: static struct PyGetSetDef PolygammaBackward1_properties[] = {
1506:   THP_FUNCTION_DEFAULT_PROPERTIES,
1507:   {(char*)"_saved_n", (getter)THPPolygammaBackward1_n_getter, nullptr, nullptr, nullptr},
1508:   {(char*)"_saved_self", (getter)THPPolygammaBackward1_self_getter, nullptr, nullptr, nullptr},
1509:   {(char*)"_raw_saved_self", (getter)THPPolygammaBackward1_self_raw_getter, nullptr, nullptr, nullptr},
1510:   {nullptr} /* sentinel */
1511: };
1512: 
1513: static PyObject* THPXlogyBackward0_other_getter(THPCppFunction *self, void *_unused) {
1514:   HANDLE_TH_ERRORS
1515:   const auto& prop = static_cast<XlogyBackward0*>(self->cdata.get())->other_;
1516:   return THPVariable_Wrap(prop.unpack(self->cdata));
1517:   END_HANDLE_TH_ERRORS
1518: }
1519: 
1520: static PyObject* THPXlogyBackward0_other_raw_getter(THPCppFunction *self, void *_unused) {
```

- EN: The main execution path in this span is carried by `cast`, `THPPolygammaBackward0_n_getter`, `PyLong_FromUnsignedLong`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `cast`, `THPPolygammaBackward0_n_getter`, `PyLong_FromUnsignedLong` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1521-1600

```cpp
1521:   HANDLE_TH_ERRORS
1522:   const auto& prop = static_cast<XlogyBackward0*>(self->cdata.get())->other_;
1523:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1524:   return obj.release().ptr();
1525:   END_HANDLE_TH_ERRORS
1526: }
1527: 
1528: static PyObject* THPXlogyBackward0_self_getter(THPCppFunction *self, void *_unused) {
1529:   HANDLE_TH_ERRORS
1530:   const auto& prop = static_cast<XlogyBackward0*>(self->cdata.get())->self_;
1531:   return THPVariable_Wrap(prop.unpack(self->cdata));
1532:   END_HANDLE_TH_ERRORS
1533: }
1534: 
1535: static PyObject* THPXlogyBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
1536:   HANDLE_TH_ERRORS
1537:   const auto& prop = static_cast<XlogyBackward0*>(self->cdata.get())->self_;
1538:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1539:   return obj.release().ptr();
1540:   END_HANDLE_TH_ERRORS
1541: }
1542: 
1543: static struct PyGetSetDef XlogyBackward0_properties[] = {
1544:   THP_FUNCTION_DEFAULT_PROPERTIES,
1545:   {(char*)"_saved_other", (getter)THPXlogyBackward0_other_getter, nullptr, nullptr, nullptr},
1546:   {(char*)"_raw_saved_other", (getter)THPXlogyBackward0_other_raw_getter, nullptr, nullptr, nullptr},
1547:   {(char*)"_saved_self", (getter)THPXlogyBackward0_self_getter, nullptr, nullptr, nullptr},
1548:   {(char*)"_raw_saved_self", (getter)THPXlogyBackward0_self_raw_getter, nullptr, nullptr, nullptr},
1549:   {nullptr} /* sentinel */
1550: };
1551: 
1552: static PyObject* THPXlogyBackward1_other_getter(THPCppFunction *self, void *_unused) {
1553:   HANDLE_TH_ERRORS
1554:   const auto& prop = static_cast<XlogyBackward1*>(self->cdata.get())->other_;
1555:   return THPVariable_Wrap(prop.unpack(self->cdata));
1556:   END_HANDLE_TH_ERRORS
1557: }
1558: 
1559: static PyObject* THPXlogyBackward1_other_raw_getter(THPCppFunction *self, void *_unused) {
1560:   HANDLE_TH_ERRORS
1561:   const auto& prop = static_cast<XlogyBackward1*>(self->cdata.get())->other_;
1562:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1563:   return obj.release().ptr();
1564:   END_HANDLE_TH_ERRORS
1565: }
1566: 
1567: static PyObject* THPXlogyBackward1_self_getter(THPCppFunction *self, void *_unused) {
1568:   HANDLE_TH_ERRORS
1569:   auto prop = static_cast<XlogyBackward1*>(self->cdata.get())->self;
1570:   if (prop.isComplex()) {
1571:     auto cprop = prop.to<c10::complex<double>>();
1572:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
1573:   } else if (prop.isFloatingPoint()) {
1574:     return PyFloat_FromDouble(prop.to<double>());
1575:   } else if (prop.isIntegral(/*includeBool=*/false)) {
1576:     return PyLong_FromLong(prop.to<int64_t>());
1577:   } else if (prop.isBoolean()) {
1578:     if (prop.to<bool>()) {
1579:       Py_RETURN_TRUE;
1580:     } else {
1581:       Py_RETURN_FALSE;
1582:     }
1583:   } else {
1584:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
1585:     return nullptr;
1586:   }
1587:   END_HANDLE_TH_ERRORS
1588: }
1589: 
1590: static struct PyGetSetDef XlogyBackward1_properties[] = {
1591:   THP_FUNCTION_DEFAULT_PROPERTIES,
1592:   {(char*)"_saved_other", (getter)THPXlogyBackward1_other_getter, nullptr, nullptr, nullptr},
1593:   {(char*)"_raw_saved_other", (getter)THPXlogyBackward1_other_raw_getter, nullptr, nullptr, nullptr},
1594:   {(char*)"_saved_self", (getter)THPXlogyBackward1_self_getter, nullptr, nullptr, nullptr},
1595:   {nullptr} /* sentinel */
1596: };
1597: 
1598: static PyObject* THPXlogyBackward2_other_getter(THPCppFunction *self, void *_unused) {
1599:   HANDLE_TH_ERRORS
1600:   auto prop = static_cast<XlogyBackward2*>(self->cdata.get())->other;
```

- EN: The main execution path in this span is carried by `cast`, `THPXlogyBackward0_self_getter`, `THPVariable_Wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `cast`, `THPXlogyBackward0_self_getter`, `THPVariable_Wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1601-1680

```cpp
1601:   if (prop.isComplex()) {
1602:     auto cprop = prop.to<c10::complex<double>>();
1603:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
1604:   } else if (prop.isFloatingPoint()) {
1605:     return PyFloat_FromDouble(prop.to<double>());
1606:   } else if (prop.isIntegral(/*includeBool=*/false)) {
1607:     return PyLong_FromLong(prop.to<int64_t>());
1608:   } else if (prop.isBoolean()) {
1609:     if (prop.to<bool>()) {
1610:       Py_RETURN_TRUE;
1611:     } else {
1612:       Py_RETURN_FALSE;
1613:     }
1614:   } else {
1615:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
1616:     return nullptr;
1617:   }
1618:   END_HANDLE_TH_ERRORS
1619: }
1620: 
1621: static PyObject* THPXlogyBackward2_self_getter(THPCppFunction *self, void *_unused) {
1622:   HANDLE_TH_ERRORS
1623:   const auto& prop = static_cast<XlogyBackward2*>(self->cdata.get())->self_;
1624:   return THPVariable_Wrap(prop.unpack(self->cdata));
1625:   END_HANDLE_TH_ERRORS
1626: }
1627: 
1628: static PyObject* THPXlogyBackward2_self_raw_getter(THPCppFunction *self, void *_unused) {
1629:   HANDLE_TH_ERRORS
1630:   const auto& prop = static_cast<XlogyBackward2*>(self->cdata.get())->self_;
1631:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1632:   return obj.release().ptr();
1633:   END_HANDLE_TH_ERRORS
1634: }
1635: 
1636: static struct PyGetSetDef XlogyBackward2_properties[] = {
1637:   THP_FUNCTION_DEFAULT_PROPERTIES,
1638:   {(char*)"_saved_other", (getter)THPXlogyBackward2_other_getter, nullptr, nullptr, nullptr},
1639:   {(char*)"_saved_self", (getter)THPXlogyBackward2_self_getter, nullptr, nullptr, nullptr},
1640:   {(char*)"_raw_saved_self", (getter)THPXlogyBackward2_self_raw_getter, nullptr, nullptr, nullptr},
1641:   {nullptr} /* sentinel */
1642: };
1643: 
1644: static PyObject* THPLinalgLuFactorExBackward0_pivot_getter(THPCppFunction *self, void *_unused) {
1645:   HANDLE_TH_ERRORS
1646:   auto prop = static_cast<LinalgLuFactorExBackward0*>(self->cdata.get())->pivot;
1647:   if (prop) {
1648:     Py_RETURN_TRUE;
1649:   } else {
1650:     Py_RETURN_FALSE;
1651:   }
1652:   END_HANDLE_TH_ERRORS
1653: }
1654: 
1655: static PyObject* THPLinalgLuFactorExBackward0_LU_getter(THPCppFunction *self, void *_unused) {
1656:   HANDLE_TH_ERRORS
1657:   const auto& prop = static_cast<LinalgLuFactorExBackward0*>(self->cdata.get())->LU_;
1658:   return THPVariable_Wrap(prop.unpack(self->cdata));
1659:   END_HANDLE_TH_ERRORS
1660: }
1661: 
1662: static PyObject* THPLinalgLuFactorExBackward0_LU_raw_getter(THPCppFunction *self, void *_unused) {
1663:   HANDLE_TH_ERRORS
1664:   const auto& prop = static_cast<LinalgLuFactorExBackward0*>(self->cdata.get())->LU_;
1665:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1666:   return obj.release().ptr();
1667:   END_HANDLE_TH_ERRORS
1668: }
1669: 
1670: static PyObject* THPLinalgLuFactorExBackward0_pivots_getter(THPCppFunction *self, void *_unused) {
1671:   HANDLE_TH_ERRORS
1672:   const auto& prop = static_cast<LinalgLuFactorExBackward0*>(self->cdata.get())->pivots_;
1673:   return THPVariable_Wrap(prop.unpack(self->cdata));
1674:   END_HANDLE_TH_ERRORS
1675: }
1676: 
1677: static PyObject* THPLinalgLuFactorExBackward0_pivots_raw_getter(THPCppFunction *self, void *_unused) {
1678:   HANDLE_TH_ERRORS
1679:   const auto& prop = static_cast<LinalgLuFactorExBackward0*>(self->cdata.get())->pivots_;
1680:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
```

- EN: The main execution path in this span is carried by `PyComplex_FromDoubles`, `PyFloat_FromDouble`, `PyLong_FromLong`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyComplex_FromDoubles`, `PyFloat_FromDouble`, `PyLong_FromLong` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1681-1760

```cpp
1681:   return obj.release().ptr();
1682:   END_HANDLE_TH_ERRORS
1683: }
1684: 
1685: static struct PyGetSetDef LinalgLuFactorExBackward0_properties[] = {
1686:   THP_FUNCTION_DEFAULT_PROPERTIES,
1687:   {(char*)"_saved_pivot", (getter)THPLinalgLuFactorExBackward0_pivot_getter, nullptr, nullptr, nullptr},
1688:   {(char*)"_saved_LU", (getter)THPLinalgLuFactorExBackward0_LU_getter, nullptr, nullptr, nullptr},
1689:   {(char*)"_raw_saved_LU", (getter)THPLinalgLuFactorExBackward0_LU_raw_getter, nullptr, nullptr, nullptr},
1690:   {(char*)"_saved_pivots", (getter)THPLinalgLuFactorExBackward0_pivots_getter, nullptr, nullptr, nullptr},
1691:   {(char*)"_raw_saved_pivots", (getter)THPLinalgLuFactorExBackward0_pivots_raw_getter, nullptr, nullptr, nullptr},
1692:   {nullptr} /* sentinel */
1693: };
1694: 
1695: static PyObject* THPMaximumBackward0_other_getter(THPCppFunction *self, void *_unused) {
1696:   HANDLE_TH_ERRORS
1697:   const auto& prop = static_cast<MaximumBackward0*>(self->cdata.get())->other_;
1698:   return THPVariable_Wrap(prop.unpack(self->cdata));
1699:   END_HANDLE_TH_ERRORS
1700: }
1701: 
1702: static PyObject* THPMaximumBackward0_other_raw_getter(THPCppFunction *self, void *_unused) {
1703:   HANDLE_TH_ERRORS
1704:   const auto& prop = static_cast<MaximumBackward0*>(self->cdata.get())->other_;
1705:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1706:   return obj.release().ptr();
1707:   END_HANDLE_TH_ERRORS
1708: }
1709: 
1710: static PyObject* THPMaximumBackward0_self_getter(THPCppFunction *self, void *_unused) {
1711:   HANDLE_TH_ERRORS
1712:   const auto& prop = static_cast<MaximumBackward0*>(self->cdata.get())->self_;
1713:   return THPVariable_Wrap(prop.unpack(self->cdata));
1714:   END_HANDLE_TH_ERRORS
1715: }
1716: 
1717: static PyObject* THPMaximumBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
1718:   HANDLE_TH_ERRORS
1719:   const auto& prop = static_cast<MaximumBackward0*>(self->cdata.get())->self_;
1720:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1721:   return obj.release().ptr();
1722:   END_HANDLE_TH_ERRORS
1723: }
1724: 
1725: static struct PyGetSetDef MaximumBackward0_properties[] = {
1726:   THP_FUNCTION_DEFAULT_PROPERTIES,
1727:   {(char*)"_saved_other", (getter)THPMaximumBackward0_other_getter, nullptr, nullptr, nullptr},
1728:   {(char*)"_raw_saved_other", (getter)THPMaximumBackward0_other_raw_getter, nullptr, nullptr, nullptr},
1729:   {(char*)"_saved_self", (getter)THPMaximumBackward0_self_getter, nullptr, nullptr, nullptr},
1730:   {(char*)"_raw_saved_self", (getter)THPMaximumBackward0_self_raw_getter, nullptr, nullptr, nullptr},
1731:   {nullptr} /* sentinel */
1732: };
1733: 
1734: static PyObject* THPMinimumBackward0_other_getter(THPCppFunction *self, void *_unused) {
1735:   HANDLE_TH_ERRORS
1736:   const auto& prop = static_cast<MinimumBackward0*>(self->cdata.get())->other_;
1737:   return THPVariable_Wrap(prop.unpack(self->cdata));
1738:   END_HANDLE_TH_ERRORS
1739: }
1740: 
1741: static PyObject* THPMinimumBackward0_other_raw_getter(THPCppFunction *self, void *_unused) {
1742:   HANDLE_TH_ERRORS
1743:   const auto& prop = static_cast<MinimumBackward0*>(self->cdata.get())->other_;
1744:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1745:   return obj.release().ptr();
1746:   END_HANDLE_TH_ERRORS
1747: }
1748: 
1749: static PyObject* THPMinimumBackward0_self_getter(THPCppFunction *self, void *_unused) {
1750:   HANDLE_TH_ERRORS
1751:   const auto& prop = static_cast<MinimumBackward0*>(self->cdata.get())->self_;
1752:   return THPVariable_Wrap(prop.unpack(self->cdata));
1753:   END_HANDLE_TH_ERRORS
1754: }
1755: 
1756: static PyObject* THPMinimumBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
1757:   HANDLE_TH_ERRORS
1758:   const auto& prop = static_cast<MinimumBackward0*>(self->cdata.get())->self_;
1759:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1760:   return obj.release().ptr();
```

- EN: The main execution path in this span is carried by `THPMaximumBackward0_other_getter`, `THPVariable_Wrap`, `THPMaximumBackward0_other_raw_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPMaximumBackward0_other_getter`, `THPVariable_Wrap`, `THPMaximumBackward0_other_raw_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1761-1840

```cpp
1761:   END_HANDLE_TH_ERRORS
1762: }
1763: 
1764: static struct PyGetSetDef MinimumBackward0_properties[] = {
1765:   THP_FUNCTION_DEFAULT_PROPERTIES,
1766:   {(char*)"_saved_other", (getter)THPMinimumBackward0_other_getter, nullptr, nullptr, nullptr},
1767:   {(char*)"_raw_saved_other", (getter)THPMinimumBackward0_other_raw_getter, nullptr, nullptr, nullptr},
1768:   {(char*)"_saved_self", (getter)THPMinimumBackward0_self_getter, nullptr, nullptr, nullptr},
1769:   {(char*)"_raw_saved_self", (getter)THPMinimumBackward0_self_raw_getter, nullptr, nullptr, nullptr},
1770:   {nullptr} /* sentinel */
1771: };
1772: 
1773: static PyObject* THPAmaxBackward0_dim_getter(THPCppFunction *self, void *_unused) {
1774:   HANDLE_TH_ERRORS
1775:   auto prop = static_cast<AmaxBackward0*>(self->cdata.get())->dim;
1776:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
1777:   for (auto i : c10::irange(prop.size())) {
1778:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
1779:   }
1780:   return tup;
1781:   END_HANDLE_TH_ERRORS
1782: }
1783: 
1784: static PyObject* THPAmaxBackward0_keepdim_getter(THPCppFunction *self, void *_unused) {
1785:   HANDLE_TH_ERRORS
1786:   auto prop = static_cast<AmaxBackward0*>(self->cdata.get())->keepdim;
1787:   if (prop) {
1788:     Py_RETURN_TRUE;
1789:   } else {
1790:     Py_RETURN_FALSE;
1791:   }
1792:   END_HANDLE_TH_ERRORS
1793: }
1794: 
1795: static PyObject* THPAmaxBackward0_self_getter(THPCppFunction *self, void *_unused) {
1796:   HANDLE_TH_ERRORS
1797:   const auto& prop = static_cast<AmaxBackward0*>(self->cdata.get())->self_;
1798:   return THPVariable_Wrap(prop.unpack(self->cdata));
1799:   END_HANDLE_TH_ERRORS
1800: }
1801: 
1802: static PyObject* THPAmaxBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
1803:   HANDLE_TH_ERRORS
1804:   const auto& prop = static_cast<AmaxBackward0*>(self->cdata.get())->self_;
1805:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1806:   return obj.release().ptr();
1807:   END_HANDLE_TH_ERRORS
1808: }
1809: 
1810: static PyObject* THPAmaxBackward0_result_getter(THPCppFunction *self, void *_unused) {
1811:   HANDLE_TH_ERRORS
1812:   const auto& prop = static_cast<AmaxBackward0*>(self->cdata.get())->result_;
1813:   return THPVariable_Wrap(prop.unpack(self->cdata));
1814:   END_HANDLE_TH_ERRORS
1815: }
1816: 
1817: static PyObject* THPAmaxBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
1818:   HANDLE_TH_ERRORS
1819:   const auto& prop = static_cast<AmaxBackward0*>(self->cdata.get())->result_;
1820:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1821:   return obj.release().ptr();
1822:   END_HANDLE_TH_ERRORS
1823: }
1824: 
1825: static struct PyGetSetDef AmaxBackward0_properties[] = {
1826:   THP_FUNCTION_DEFAULT_PROPERTIES,
1827:   {(char*)"_saved_dim", (getter)THPAmaxBackward0_dim_getter, nullptr, nullptr, nullptr},
1828:   {(char*)"_saved_keepdim", (getter)THPAmaxBackward0_keepdim_getter, nullptr, nullptr, nullptr},
1829:   {(char*)"_saved_self", (getter)THPAmaxBackward0_self_getter, nullptr, nullptr, nullptr},
1830:   {(char*)"_raw_saved_self", (getter)THPAmaxBackward0_self_raw_getter, nullptr, nullptr, nullptr},
1831:   {(char*)"_saved_result", (getter)THPAmaxBackward0_result_getter, nullptr, nullptr, nullptr},
1832:   {(char*)"_raw_saved_result", (getter)THPAmaxBackward0_result_raw_getter, nullptr, nullptr, nullptr},
1833:   {nullptr} /* sentinel */
1834: };
1835: 
1836: static PyObject* THPAminBackward0_dim_getter(THPCppFunction *self, void *_unused) {
1837:   HANDLE_TH_ERRORS
1838:   auto prop = static_cast<AminBackward0*>(self->cdata.get())->dim;
1839:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
1840:   for (auto i : c10::irange(prop.size())) {
```

- EN: The main execution path in this span is carried by `THPAmaxBackward0_dim_getter`, `PyTuple_New`, `PyTuple_SetItem`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPAmaxBackward0_dim_getter`, `PyTuple_New`, `PyTuple_SetItem` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1841-1920

```cpp
1841:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
1842:   }
1843:   return tup;
1844:   END_HANDLE_TH_ERRORS
1845: }
1846: 
1847: static PyObject* THPAminBackward0_keepdim_getter(THPCppFunction *self, void *_unused) {
1848:   HANDLE_TH_ERRORS
1849:   auto prop = static_cast<AminBackward0*>(self->cdata.get())->keepdim;
1850:   if (prop) {
1851:     Py_RETURN_TRUE;
1852:   } else {
1853:     Py_RETURN_FALSE;
1854:   }
1855:   END_HANDLE_TH_ERRORS
1856: }
1857: 
1858: static PyObject* THPAminBackward0_self_getter(THPCppFunction *self, void *_unused) {
1859:   HANDLE_TH_ERRORS
1860:   const auto& prop = static_cast<AminBackward0*>(self->cdata.get())->self_;
1861:   return THPVariable_Wrap(prop.unpack(self->cdata));
1862:   END_HANDLE_TH_ERRORS
1863: }
1864: 
1865: static PyObject* THPAminBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
1866:   HANDLE_TH_ERRORS
1867:   const auto& prop = static_cast<AminBackward0*>(self->cdata.get())->self_;
1868:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1869:   return obj.release().ptr();
1870:   END_HANDLE_TH_ERRORS
1871: }
1872: 
1873: static PyObject* THPAminBackward0_result_getter(THPCppFunction *self, void *_unused) {
1874:   HANDLE_TH_ERRORS
1875:   const auto& prop = static_cast<AminBackward0*>(self->cdata.get())->result_;
1876:   return THPVariable_Wrap(prop.unpack(self->cdata));
1877:   END_HANDLE_TH_ERRORS
1878: }
1879: 
1880: static PyObject* THPAminBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
1881:   HANDLE_TH_ERRORS
1882:   const auto& prop = static_cast<AminBackward0*>(self->cdata.get())->result_;
1883:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1884:   return obj.release().ptr();
1885:   END_HANDLE_TH_ERRORS
1886: }
1887: 
1888: static struct PyGetSetDef AminBackward0_properties[] = {
1889:   THP_FUNCTION_DEFAULT_PROPERTIES,
1890:   {(char*)"_saved_dim", (getter)THPAminBackward0_dim_getter, nullptr, nullptr, nullptr},
1891:   {(char*)"_saved_keepdim", (getter)THPAminBackward0_keepdim_getter, nullptr, nullptr, nullptr},
1892:   {(char*)"_saved_self", (getter)THPAminBackward0_self_getter, nullptr, nullptr, nullptr},
1893:   {(char*)"_raw_saved_self", (getter)THPAminBackward0_self_raw_getter, nullptr, nullptr, nullptr},
1894:   {(char*)"_saved_result", (getter)THPAminBackward0_result_getter, nullptr, nullptr, nullptr},
1895:   {(char*)"_raw_saved_result", (getter)THPAminBackward0_result_raw_getter, nullptr, nullptr, nullptr},
1896:   {nullptr} /* sentinel */
1897: };
1898: 
1899: static PyObject* THPGroupedMmBackward0_mat2_getter(THPCppFunction *self, void *_unused) {
1900:   HANDLE_TH_ERRORS
1901:   const auto& prop = static_cast<GroupedMmBackward0*>(self->cdata.get())->mat2_;
1902:   return THPVariable_Wrap(prop.unpack(self->cdata));
1903:   END_HANDLE_TH_ERRORS
1904: }
1905: 
1906: static PyObject* THPGroupedMmBackward0_mat2_raw_getter(THPCppFunction *self, void *_unused) {
1907:   HANDLE_TH_ERRORS
1908:   const auto& prop = static_cast<GroupedMmBackward0*>(self->cdata.get())->mat2_;
1909:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1910:   return obj.release().ptr();
1911:   END_HANDLE_TH_ERRORS
1912: }
1913: 
1914: static PyObject* THPGroupedMmBackward0_mat2_sym_sizes_getter(THPCppFunction *self, void *_unused) {
1915:   HANDLE_TH_ERRORS
1916:   auto prop = static_cast<GroupedMmBackward0*>(self->cdata.get())->mat2_sym_sizes;
1917:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
1918:   for (auto i : c10::irange(prop.size())) {
1919:       auto si = prop[i];
1920:       if (auto m = si.maybe_as_int()) {
```

- EN: The main execution path in this span is carried by `PyTuple_SetItem`, `THPAminBackward0_keepdim_getter`, `THPAminBackward0_self_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyTuple_SetItem`, `THPAminBackward0_keepdim_getter`, `THPAminBackward0_self_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1921-2000

```cpp
1921:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
1922:       } else {
1923:         auto py_symint = py::cast(si).release().ptr();
1924:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
1925:       }
1926:   }
1927:   return tup;
1928:   END_HANDLE_TH_ERRORS
1929: }
1930: 
1931: static PyObject* THPGroupedMmBackward0_mat2_sym_strides_getter(THPCppFunction *self, void *_unused) {
1932:   HANDLE_TH_ERRORS
1933:   auto prop = static_cast<GroupedMmBackward0*>(self->cdata.get())->mat2_sym_strides;
1934:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
1935:   for (auto i : c10::irange(prop.size())) {
1936:       auto si = prop[i];
1937:       if (auto m = si.maybe_as_int()) {
1938:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
1939:       } else {
1940:         auto py_symint = py::cast(si).release().ptr();
1941:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
1942:       }
1943:   }
1944:   return tup;
1945:   END_HANDLE_TH_ERRORS
1946: }
1947: 
1948: static PyObject* THPGroupedMmBackward0_offs_getter(THPCppFunction *self, void *_unused) {
1949:   HANDLE_TH_ERRORS
1950:   const auto& prop = static_cast<GroupedMmBackward0*>(self->cdata.get())->offs_;
1951:   return THPVariable_Wrap(prop.unpack(self->cdata));
1952:   END_HANDLE_TH_ERRORS
1953: }
1954: 
1955: static PyObject* THPGroupedMmBackward0_offs_raw_getter(THPCppFunction *self, void *_unused) {
1956:   HANDLE_TH_ERRORS
1957:   const auto& prop = static_cast<GroupedMmBackward0*>(self->cdata.get())->offs_;
1958:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1959:   return obj.release().ptr();
1960:   END_HANDLE_TH_ERRORS
1961: }
1962: 
1963: static PyObject* THPGroupedMmBackward0_self_getter(THPCppFunction *self, void *_unused) {
1964:   HANDLE_TH_ERRORS
1965:   const auto& prop = static_cast<GroupedMmBackward0*>(self->cdata.get())->self_;
1966:   return THPVariable_Wrap(prop.unpack(self->cdata));
1967:   END_HANDLE_TH_ERRORS
1968: }
1969: 
1970: static PyObject* THPGroupedMmBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
1971:   HANDLE_TH_ERRORS
1972:   const auto& prop = static_cast<GroupedMmBackward0*>(self->cdata.get())->self_;
1973:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1974:   return obj.release().ptr();
1975:   END_HANDLE_TH_ERRORS
1976: }
1977: 
1978: static PyObject* THPGroupedMmBackward0_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
1979:   HANDLE_TH_ERRORS
1980:   auto prop = static_cast<GroupedMmBackward0*>(self->cdata.get())->self_sym_sizes;
1981:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
1982:   for (auto i : c10::irange(prop.size())) {
1983:       auto si = prop[i];
1984:       if (auto m = si.maybe_as_int()) {
1985:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
1986:       } else {
1987:         auto py_symint = py::cast(si).release().ptr();
1988:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
1989:       }
1990:   }
1991:   return tup;
1992:   END_HANDLE_TH_ERRORS
1993: }
1994: 
1995: static PyObject* THPGroupedMmBackward0_self_sym_strides_getter(THPCppFunction *self, void *_unused) {
1996:   HANDLE_TH_ERRORS
1997:   auto prop = static_cast<GroupedMmBackward0*>(self->cdata.get())->self_sym_strides;
1998:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
1999:   for (auto i : c10::irange(prop.size())) {
2000:       auto si = prop[i];
```

- EN: The main execution path in this span is carried by `PyTuple_SetItem`, `cast`, `THPGroupedMmBackward0_mat2_sym_strides_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyTuple_SetItem`, `cast`, `THPGroupedMmBackward0_mat2_sym_strides_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2001-2080

```cpp
2001:       if (auto m = si.maybe_as_int()) {
2002:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
2003:       } else {
2004:         auto py_symint = py::cast(si).release().ptr();
2005:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
2006:       }
2007:   }
2008:   return tup;
2009:   END_HANDLE_TH_ERRORS
2010: }
2011: 
2012: static struct PyGetSetDef GroupedMmBackward0_properties[] = {
2013:   THP_FUNCTION_DEFAULT_PROPERTIES,
2014:   {(char*)"_saved_mat2", (getter)THPGroupedMmBackward0_mat2_getter, nullptr, nullptr, nullptr},
2015:   {(char*)"_raw_saved_mat2", (getter)THPGroupedMmBackward0_mat2_raw_getter, nullptr, nullptr, nullptr},
2016:   {(char*)"_saved_mat2_sym_sizes", (getter)THPGroupedMmBackward0_mat2_sym_sizes_getter, nullptr, nullptr, nullptr},
2017:   {(char*)"_saved_mat2_sym_strides", (getter)THPGroupedMmBackward0_mat2_sym_strides_getter, nullptr, nullptr, nullptr},
2018:   {(char*)"_saved_offs", (getter)THPGroupedMmBackward0_offs_getter, nullptr, nullptr, nullptr},
2019:   {(char*)"_raw_saved_offs", (getter)THPGroupedMmBackward0_offs_raw_getter, nullptr, nullptr, nullptr},
2020:   {(char*)"_saved_self", (getter)THPGroupedMmBackward0_self_getter, nullptr, nullptr, nullptr},
2021:   {(char*)"_raw_saved_self", (getter)THPGroupedMmBackward0_self_raw_getter, nullptr, nullptr, nullptr},
2022:   {(char*)"_saved_self_sym_sizes", (getter)THPGroupedMmBackward0_self_sym_sizes_getter, nullptr, nullptr, nullptr},
2023:   {(char*)"_saved_self_sym_strides", (getter)THPGroupedMmBackward0_self_sym_strides_getter, nullptr, nullptr, nullptr},
2024:   {nullptr} /* sentinel */
2025: };
2026: 
2027: static PyObject* THPNanToNumBackward0_self_getter(THPCppFunction *self, void *_unused) {
2028:   HANDLE_TH_ERRORS
2029:   const auto& prop = static_cast<NanToNumBackward0*>(self->cdata.get())->self_;
2030:   return THPVariable_Wrap(prop.unpack(self->cdata));
2031:   END_HANDLE_TH_ERRORS
2032: }
2033: 
2034: static PyObject* THPNanToNumBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
2035:   HANDLE_TH_ERRORS
2036:   const auto& prop = static_cast<NanToNumBackward0*>(self->cdata.get())->self_;
2037:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2038:   return obj.release().ptr();
2039:   END_HANDLE_TH_ERRORS
2040: }
2041: 
2042: static struct PyGetSetDef NanToNumBackward0_properties[] = {
2043:   THP_FUNCTION_DEFAULT_PROPERTIES,
2044:   {(char*)"_saved_self", (getter)THPNanToNumBackward0_self_getter, nullptr, nullptr, nullptr},
2045:   {(char*)"_raw_saved_self", (getter)THPNanToNumBackward0_self_raw_getter, nullptr, nullptr, nullptr},
2046:   {nullptr} /* sentinel */
2047: };
2048: 
2049: static PyObject* THPNativeBatchNormBackward0_eps_getter(THPCppFunction *self, void *_unused) {
2050:   HANDLE_TH_ERRORS
2051:   auto prop = static_cast<NativeBatchNormBackward0*>(self->cdata.get())->eps;
2052:   return PyFloat_FromDouble((double) prop);
2053:   END_HANDLE_TH_ERRORS
2054: }
2055: 
2056: static PyObject* THPNativeBatchNormBackward0_input_getter(THPCppFunction *self, void *_unused) {
2057:   HANDLE_TH_ERRORS
2058:   const auto& prop = static_cast<NativeBatchNormBackward0*>(self->cdata.get())->input_;
2059:   return THPVariable_Wrap(prop.unpack(self->cdata));
2060:   END_HANDLE_TH_ERRORS
2061: }
2062: 
2063: static PyObject* THPNativeBatchNormBackward0_input_raw_getter(THPCppFunction *self, void *_unused) {
2064:   HANDLE_TH_ERRORS
2065:   const auto& prop = static_cast<NativeBatchNormBackward0*>(self->cdata.get())->input_;
2066:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2067:   return obj.release().ptr();
2068:   END_HANDLE_TH_ERRORS
2069: }
2070: 
2071: static PyObject* THPNativeBatchNormBackward0_running_mean_getter(THPCppFunction *self, void *_unused) {
2072:   HANDLE_TH_ERRORS
2073:   const auto& prop = static_cast<NativeBatchNormBackward0*>(self->cdata.get())->running_mean_;
2074:   return THPVariable_Wrap(prop.unpack(self->cdata));
2075:   END_HANDLE_TH_ERRORS
2076: }
2077: 
2078: static PyObject* THPNativeBatchNormBackward0_running_mean_raw_getter(THPCppFunction *self, void *_unused) {
2079:   HANDLE_TH_ERRORS
2080:   const auto& prop = static_cast<NativeBatchNormBackward0*>(self->cdata.get())->running_mean_;
```

- EN: The main execution path in this span is carried by `PyTuple_SetItem`, `cast`, `THPNanToNumBackward0_self_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyTuple_SetItem`, `cast`, `THPNanToNumBackward0_self_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2081-2160

```cpp
2081:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2082:   return obj.release().ptr();
2083:   END_HANDLE_TH_ERRORS
2084: }
2085: 
2086: static PyObject* THPNativeBatchNormBackward0_running_var_getter(THPCppFunction *self, void *_unused) {
2087:   HANDLE_TH_ERRORS
2088:   const auto& prop = static_cast<NativeBatchNormBackward0*>(self->cdata.get())->running_var_;
2089:   return THPVariable_Wrap(prop.unpack(self->cdata));
2090:   END_HANDLE_TH_ERRORS
2091: }
2092: 
2093: static PyObject* THPNativeBatchNormBackward0_running_var_raw_getter(THPCppFunction *self, void *_unused) {
2094:   HANDLE_TH_ERRORS
2095:   const auto& prop = static_cast<NativeBatchNormBackward0*>(self->cdata.get())->running_var_;
2096:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2097:   return obj.release().ptr();
2098:   END_HANDLE_TH_ERRORS
2099: }
2100: 
2101: static PyObject* THPNativeBatchNormBackward0_training_getter(THPCppFunction *self, void *_unused) {
2102:   HANDLE_TH_ERRORS
2103:   auto prop = static_cast<NativeBatchNormBackward0*>(self->cdata.get())->training;
2104:   if (prop) {
2105:     Py_RETURN_TRUE;
2106:   } else {
2107:     Py_RETURN_FALSE;
2108:   }
2109:   END_HANDLE_TH_ERRORS
2110: }
2111: 
2112: static PyObject* THPNativeBatchNormBackward0_weight_getter(THPCppFunction *self, void *_unused) {
2113:   HANDLE_TH_ERRORS
2114:   const auto& prop = static_cast<NativeBatchNormBackward0*>(self->cdata.get())->weight_;
2115:   return THPVariable_Wrap(prop.unpack(self->cdata));
2116:   END_HANDLE_TH_ERRORS
2117: }
2118: 
2119: static PyObject* THPNativeBatchNormBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
2120:   HANDLE_TH_ERRORS
2121:   const auto& prop = static_cast<NativeBatchNormBackward0*>(self->cdata.get())->weight_;
2122:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2123:   return obj.release().ptr();
2124:   END_HANDLE_TH_ERRORS
2125: }
2126: 
2127: static PyObject* THPNativeBatchNormBackward0_result1_getter(THPCppFunction *self, void *_unused) {
2128:   HANDLE_TH_ERRORS
2129:   const auto& prop = static_cast<NativeBatchNormBackward0*>(self->cdata.get())->result1_;
2130:   return THPVariable_Wrap(prop.unpack(self->cdata));
2131:   END_HANDLE_TH_ERRORS
2132: }
2133: 
2134: static PyObject* THPNativeBatchNormBackward0_result1_raw_getter(THPCppFunction *self, void *_unused) {
2135:   HANDLE_TH_ERRORS
2136:   const auto& prop = static_cast<NativeBatchNormBackward0*>(self->cdata.get())->result1_;
2137:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2138:   return obj.release().ptr();
2139:   END_HANDLE_TH_ERRORS
2140: }
2141: 
2142: static PyObject* THPNativeBatchNormBackward0_result2_getter(THPCppFunction *self, void *_unused) {
2143:   HANDLE_TH_ERRORS
2144:   const auto& prop = static_cast<NativeBatchNormBackward0*>(self->cdata.get())->result2_;
2145:   return THPVariable_Wrap(prop.unpack(self->cdata));
2146:   END_HANDLE_TH_ERRORS
2147: }
2148: 
2149: static PyObject* THPNativeBatchNormBackward0_result2_raw_getter(THPCppFunction *self, void *_unused) {
2150:   HANDLE_TH_ERRORS
2151:   const auto& prop = static_cast<NativeBatchNormBackward0*>(self->cdata.get())->result2_;
2152:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2153:   return obj.release().ptr();
2154:   END_HANDLE_TH_ERRORS
2155: }
2156: 
2157: static struct PyGetSetDef NativeBatchNormBackward0_properties[] = {
2158:   THP_FUNCTION_DEFAULT_PROPERTIES,
2159:   {(char*)"_saved_eps", (getter)THPNativeBatchNormBackward0_eps_getter, nullptr, nullptr, nullptr},
2160:   {(char*)"_saved_input", (getter)THPNativeBatchNormBackward0_input_getter, nullptr, nullptr, nullptr},
```

- EN: The main execution path in this span is carried by `cast`, `THPNativeBatchNormBackward0_running_var_getter`, `THPVariable_Wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `cast`, `THPNativeBatchNormBackward0_running_var_getter`, `THPVariable_Wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2161-2240

```cpp
2161:   {(char*)"_raw_saved_input", (getter)THPNativeBatchNormBackward0_input_raw_getter, nullptr, nullptr, nullptr},
2162:   {(char*)"_saved_running_mean", (getter)THPNativeBatchNormBackward0_running_mean_getter, nullptr, nullptr, nullptr},
2163:   {(char*)"_raw_saved_running_mean", (getter)THPNativeBatchNormBackward0_running_mean_raw_getter, nullptr, nullptr, nullptr},
2164:   {(char*)"_saved_running_var", (getter)THPNativeBatchNormBackward0_running_var_getter, nullptr, nullptr, nullptr},
2165:   {(char*)"_raw_saved_running_var", (getter)THPNativeBatchNormBackward0_running_var_raw_getter, nullptr, nullptr, nullptr},
2166:   {(char*)"_saved_training", (getter)THPNativeBatchNormBackward0_training_getter, nullptr, nullptr, nullptr},
2167:   {(char*)"_saved_weight", (getter)THPNativeBatchNormBackward0_weight_getter, nullptr, nullptr, nullptr},
2168:   {(char*)"_raw_saved_weight", (getter)THPNativeBatchNormBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
2169:   {(char*)"_saved_result1", (getter)THPNativeBatchNormBackward0_result1_getter, nullptr, nullptr, nullptr},
2170:   {(char*)"_raw_saved_result1", (getter)THPNativeBatchNormBackward0_result1_raw_getter, nullptr, nullptr, nullptr},
2171:   {(char*)"_saved_result2", (getter)THPNativeBatchNormBackward0_result2_getter, nullptr, nullptr, nullptr},
2172:   {(char*)"_raw_saved_result2", (getter)THPNativeBatchNormBackward0_result2_raw_getter, nullptr, nullptr, nullptr},
2173:   {nullptr} /* sentinel */
2174: };
2175: 
2176: static PyObject* THPNativeBatchNormBackwardBackward0_eps_getter(THPCppFunction *self, void *_unused) {
2177:   HANDLE_TH_ERRORS
2178:   auto prop = static_cast<NativeBatchNormBackwardBackward0*>(self->cdata.get())->eps;
2179:   return PyFloat_FromDouble((double) prop);
2180:   END_HANDLE_TH_ERRORS
2181: }
2182: 
2183: static PyObject* THPNativeBatchNormBackwardBackward0_grad_out_getter(THPCppFunction *self, void *_unused) {
2184:   HANDLE_TH_ERRORS
2185:   const auto& prop = static_cast<NativeBatchNormBackwardBackward0*>(self->cdata.get())->grad_out_;
2186:   return THPVariable_Wrap(prop.unpack(self->cdata));
2187:   END_HANDLE_TH_ERRORS
2188: }
2189: 
2190: static PyObject* THPNativeBatchNormBackwardBackward0_grad_out_raw_getter(THPCppFunction *self, void *_unused) {
2191:   HANDLE_TH_ERRORS
2192:   const auto& prop = static_cast<NativeBatchNormBackwardBackward0*>(self->cdata.get())->grad_out_;
2193:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2194:   return obj.release().ptr();
2195:   END_HANDLE_TH_ERRORS
2196: }
2197: 
2198: static PyObject* THPNativeBatchNormBackwardBackward0_input_getter(THPCppFunction *self, void *_unused) {
2199:   HANDLE_TH_ERRORS
2200:   const auto& prop = static_cast<NativeBatchNormBackwardBackward0*>(self->cdata.get())->input_;
2201:   return THPVariable_Wrap(prop.unpack(self->cdata));
2202:   END_HANDLE_TH_ERRORS
2203: }
2204: 
2205: static PyObject* THPNativeBatchNormBackwardBackward0_input_raw_getter(THPCppFunction *self, void *_unused) {
2206:   HANDLE_TH_ERRORS
2207:   const auto& prop = static_cast<NativeBatchNormBackwardBackward0*>(self->cdata.get())->input_;
2208:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2209:   return obj.release().ptr();
2210:   END_HANDLE_TH_ERRORS
2211: }
2212: 
2213: static PyObject* THPNativeBatchNormBackwardBackward0_running_mean_getter(THPCppFunction *self, void *_unused) {
2214:   HANDLE_TH_ERRORS
2215:   const auto& prop = static_cast<NativeBatchNormBackwardBackward0*>(self->cdata.get())->running_mean_;
2216:   return THPVariable_Wrap(prop.unpack(self->cdata));
2217:   END_HANDLE_TH_ERRORS
2218: }
2219: 
2220: static PyObject* THPNativeBatchNormBackwardBackward0_running_mean_raw_getter(THPCppFunction *self, void *_unused) {
2221:   HANDLE_TH_ERRORS
2222:   const auto& prop = static_cast<NativeBatchNormBackwardBackward0*>(self->cdata.get())->running_mean_;
2223:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2224:   return obj.release().ptr();
2225:   END_HANDLE_TH_ERRORS
2226: }
2227: 
2228: static PyObject* THPNativeBatchNormBackwardBackward0_running_var_getter(THPCppFunction *self, void *_unused) {
2229:   HANDLE_TH_ERRORS
2230:   const auto& prop = static_cast<NativeBatchNormBackwardBackward0*>(self->cdata.get())->running_var_;
2231:   return THPVariable_Wrap(prop.unpack(self->cdata));
2232:   END_HANDLE_TH_ERRORS
2233: }
2234: 
2235: static PyObject* THPNativeBatchNormBackwardBackward0_running_var_raw_getter(THPCppFunction *self, void *_unused) {
2236:   HANDLE_TH_ERRORS
2237:   const auto& prop = static_cast<NativeBatchNormBackwardBackward0*>(self->cdata.get())->running_var_;
2238:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2239:   return obj.release().ptr();
2240:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPNativeBatchNormBackwardBackward0_eps_getter`, `PyFloat_FromDouble`, `THPNativeBatchNormBackwardBackward0_grad_out_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPNativeBatchNormBackwardBackward0_eps_getter`, `PyFloat_FromDouble`, `THPNativeBatchNormBackwardBackward0_grad_out_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2241-2320

```cpp
2241: }
2242: 
2243: static PyObject* THPNativeBatchNormBackwardBackward0_save_invstd_getter(THPCppFunction *self, void *_unused) {
2244:   HANDLE_TH_ERRORS
2245:   const auto& prop = static_cast<NativeBatchNormBackwardBackward0*>(self->cdata.get())->save_invstd_;
2246:   return THPVariable_Wrap(prop.unpack(self->cdata));
2247:   END_HANDLE_TH_ERRORS
2248: }
2249: 
2250: static PyObject* THPNativeBatchNormBackwardBackward0_save_invstd_raw_getter(THPCppFunction *self, void *_unused) {
2251:   HANDLE_TH_ERRORS
2252:   const auto& prop = static_cast<NativeBatchNormBackwardBackward0*>(self->cdata.get())->save_invstd_;
2253:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2254:   return obj.release().ptr();
2255:   END_HANDLE_TH_ERRORS
2256: }
2257: 
2258: static PyObject* THPNativeBatchNormBackwardBackward0_save_mean_getter(THPCppFunction *self, void *_unused) {
2259:   HANDLE_TH_ERRORS
2260:   const auto& prop = static_cast<NativeBatchNormBackwardBackward0*>(self->cdata.get())->save_mean_;
2261:   return THPVariable_Wrap(prop.unpack(self->cdata));
2262:   END_HANDLE_TH_ERRORS
2263: }
2264: 
2265: static PyObject* THPNativeBatchNormBackwardBackward0_save_mean_raw_getter(THPCppFunction *self, void *_unused) {
2266:   HANDLE_TH_ERRORS
2267:   const auto& prop = static_cast<NativeBatchNormBackwardBackward0*>(self->cdata.get())->save_mean_;
2268:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2269:   return obj.release().ptr();
2270:   END_HANDLE_TH_ERRORS
2271: }
2272: 
2273: static PyObject* THPNativeBatchNormBackwardBackward0_train_getter(THPCppFunction *self, void *_unused) {
2274:   HANDLE_TH_ERRORS
2275:   auto prop = static_cast<NativeBatchNormBackwardBackward0*>(self->cdata.get())->train;
2276:   if (prop) {
2277:     Py_RETURN_TRUE;
2278:   } else {
2279:     Py_RETURN_FALSE;
2280:   }
2281:   END_HANDLE_TH_ERRORS
2282: }
2283: 
2284: static PyObject* THPNativeBatchNormBackwardBackward0_weight_getter(THPCppFunction *self, void *_unused) {
2285:   HANDLE_TH_ERRORS
2286:   const auto& prop = static_cast<NativeBatchNormBackwardBackward0*>(self->cdata.get())->weight_;
2287:   return THPVariable_Wrap(prop.unpack(self->cdata));
2288:   END_HANDLE_TH_ERRORS
2289: }
2290: 
2291: static PyObject* THPNativeBatchNormBackwardBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
2292:   HANDLE_TH_ERRORS
2293:   const auto& prop = static_cast<NativeBatchNormBackwardBackward0*>(self->cdata.get())->weight_;
2294:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2295:   return obj.release().ptr();
2296:   END_HANDLE_TH_ERRORS
2297: }
2298: 
2299: static struct PyGetSetDef NativeBatchNormBackwardBackward0_properties[] = {
2300:   THP_FUNCTION_DEFAULT_PROPERTIES,
2301:   {(char*)"_saved_eps", (getter)THPNativeBatchNormBackwardBackward0_eps_getter, nullptr, nullptr, nullptr},
2302:   {(char*)"_saved_grad_out", (getter)THPNativeBatchNormBackwardBackward0_grad_out_getter, nullptr, nullptr, nullptr},
2303:   {(char*)"_raw_saved_grad_out", (getter)THPNativeBatchNormBackwardBackward0_grad_out_raw_getter, nullptr, nullptr, nullptr},
2304:   {(char*)"_saved_input", (getter)THPNativeBatchNormBackwardBackward0_input_getter, nullptr, nullptr, nullptr},
2305:   {(char*)"_raw_saved_input", (getter)THPNativeBatchNormBackwardBackward0_input_raw_getter, nullptr, nullptr, nullptr},
2306:   {(char*)"_saved_running_mean", (getter)THPNativeBatchNormBackwardBackward0_running_mean_getter, nullptr, nullptr, nullptr},
2307:   {(char*)"_raw_saved_running_mean", (getter)THPNativeBatchNormBackwardBackward0_running_mean_raw_getter, nullptr, nullptr, nullptr},
2308:   {(char*)"_saved_running_var", (getter)THPNativeBatchNormBackwardBackward0_running_var_getter, nullptr, nullptr, nullptr},
2309:   {(char*)"_raw_saved_running_var", (getter)THPNativeBatchNormBackwardBackward0_running_var_raw_getter, nullptr, nullptr, nullptr},
2310:   {(char*)"_saved_save_invstd", (getter)THPNativeBatchNormBackwardBackward0_save_invstd_getter, nullptr, nullptr, nullptr},
2311:   {(char*)"_raw_saved_save_invstd", (getter)THPNativeBatchNormBackwardBackward0_save_invstd_raw_getter, nullptr, nullptr, nullptr},
2312:   {(char*)"_saved_save_mean", (getter)THPNativeBatchNormBackwardBackward0_save_mean_getter, nullptr, nullptr, nullptr},
2313:   {(char*)"_raw_saved_save_mean", (getter)THPNativeBatchNormBackwardBackward0_save_mean_raw_getter, nullptr, nullptr, nullptr},
2314:   {(char*)"_saved_train", (getter)THPNativeBatchNormBackwardBackward0_train_getter, nullptr, nullptr, nullptr},
2315:   {(char*)"_saved_weight", (getter)THPNativeBatchNormBackwardBackward0_weight_getter, nullptr, nullptr, nullptr},
2316:   {(char*)"_raw_saved_weight", (getter)THPNativeBatchNormBackwardBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
2317:   {nullptr} /* sentinel */
2318: };
2319: 
2320: static PyObject* THPNativeLayerNormBackward0_bias_getter(THPCppFunction *self, void *_unused) {
```

- EN: The main execution path in this span is carried by `THPNativeBatchNormBackwardBackward0_save_invstd_getter`, `THPVariable_Wrap`, `THPNativeBatchNormBackwardBackward0_save_invstd_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPNativeBatchNormBackwardBackward0_save_invstd_getter`, `THPVariable_Wrap`, `THPNativeBatchNormBackwardBackward0_save_invstd_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2321-2400

```cpp
2321:   HANDLE_TH_ERRORS
2322:   const auto& prop = static_cast<NativeLayerNormBackward0*>(self->cdata.get())->bias_;
2323:   return THPVariable_Wrap(prop.unpack(self->cdata));
2324:   END_HANDLE_TH_ERRORS
2325: }
2326: 
2327: static PyObject* THPNativeLayerNormBackward0_bias_raw_getter(THPCppFunction *self, void *_unused) {
2328:   HANDLE_TH_ERRORS
2329:   const auto& prop = static_cast<NativeLayerNormBackward0*>(self->cdata.get())->bias_;
2330:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2331:   return obj.release().ptr();
2332:   END_HANDLE_TH_ERRORS
2333: }
2334: 
2335: static PyObject* THPNativeLayerNormBackward0_input_getter(THPCppFunction *self, void *_unused) {
2336:   HANDLE_TH_ERRORS
2337:   const auto& prop = static_cast<NativeLayerNormBackward0*>(self->cdata.get())->input_;
2338:   return THPVariable_Wrap(prop.unpack(self->cdata));
2339:   END_HANDLE_TH_ERRORS
2340: }
2341: 
2342: static PyObject* THPNativeLayerNormBackward0_input_raw_getter(THPCppFunction *self, void *_unused) {
2343:   HANDLE_TH_ERRORS
2344:   const auto& prop = static_cast<NativeLayerNormBackward0*>(self->cdata.get())->input_;
2345:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2346:   return obj.release().ptr();
2347:   END_HANDLE_TH_ERRORS
2348: }
2349: 
2350: static PyObject* THPNativeLayerNormBackward0_normalized_shape_getter(THPCppFunction *self, void *_unused) {
2351:   HANDLE_TH_ERRORS
2352:   auto prop = static_cast<NativeLayerNormBackward0*>(self->cdata.get())->normalized_shape;
2353:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
2354:   for (auto i : c10::irange(prop.size())) {
2355:       auto si = prop[i];
2356:       if (auto m = si.maybe_as_int()) {
2357:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
2358:       } else {
2359:         auto py_symint = py::cast(si).release().ptr();
2360:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
2361:       }
2362:   }
2363:   return tup;
2364:   END_HANDLE_TH_ERRORS
2365: }
2366: 
2367: static PyObject* THPNativeLayerNormBackward0_weight_getter(THPCppFunction *self, void *_unused) {
2368:   HANDLE_TH_ERRORS
2369:   const auto& prop = static_cast<NativeLayerNormBackward0*>(self->cdata.get())->weight_;
2370:   return THPVariable_Wrap(prop.unpack(self->cdata));
2371:   END_HANDLE_TH_ERRORS
2372: }
2373: 
2374: static PyObject* THPNativeLayerNormBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
2375:   HANDLE_TH_ERRORS
2376:   const auto& prop = static_cast<NativeLayerNormBackward0*>(self->cdata.get())->weight_;
2377:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2378:   return obj.release().ptr();
2379:   END_HANDLE_TH_ERRORS
2380: }
2381: 
2382: static PyObject* THPNativeLayerNormBackward0_result1_getter(THPCppFunction *self, void *_unused) {
2383:   HANDLE_TH_ERRORS
2384:   const auto& prop = static_cast<NativeLayerNormBackward0*>(self->cdata.get())->result1_;
2385:   return THPVariable_Wrap(prop.unpack(self->cdata));
2386:   END_HANDLE_TH_ERRORS
2387: }
2388: 
2389: static PyObject* THPNativeLayerNormBackward0_result1_raw_getter(THPCppFunction *self, void *_unused) {
2390:   HANDLE_TH_ERRORS
2391:   const auto& prop = static_cast<NativeLayerNormBackward0*>(self->cdata.get())->result1_;
2392:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2393:   return obj.release().ptr();
2394:   END_HANDLE_TH_ERRORS
2395: }
2396: 
2397: static PyObject* THPNativeLayerNormBackward0_result2_getter(THPCppFunction *self, void *_unused) {
2398:   HANDLE_TH_ERRORS
2399:   const auto& prop = static_cast<NativeLayerNormBackward0*>(self->cdata.get())->result2_;
2400:   return THPVariable_Wrap(prop.unpack(self->cdata));
```

- EN: The main execution path in this span is carried by `THPVariable_Wrap`, `THPNativeLayerNormBackward0_bias_raw_getter`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_Wrap`, `THPNativeLayerNormBackward0_bias_raw_getter`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2401-2480

```cpp
2401:   END_HANDLE_TH_ERRORS
2402: }
2403: 
2404: static PyObject* THPNativeLayerNormBackward0_result2_raw_getter(THPCppFunction *self, void *_unused) {
2405:   HANDLE_TH_ERRORS
2406:   const auto& prop = static_cast<NativeLayerNormBackward0*>(self->cdata.get())->result2_;
2407:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2408:   return obj.release().ptr();
2409:   END_HANDLE_TH_ERRORS
2410: }
2411: 
2412: static struct PyGetSetDef NativeLayerNormBackward0_properties[] = {
2413:   THP_FUNCTION_DEFAULT_PROPERTIES,
2414:   {(char*)"_saved_bias", (getter)THPNativeLayerNormBackward0_bias_getter, nullptr, nullptr, nullptr},
2415:   {(char*)"_raw_saved_bias", (getter)THPNativeLayerNormBackward0_bias_raw_getter, nullptr, nullptr, nullptr},
2416:   {(char*)"_saved_input", (getter)THPNativeLayerNormBackward0_input_getter, nullptr, nullptr, nullptr},
2417:   {(char*)"_raw_saved_input", (getter)THPNativeLayerNormBackward0_input_raw_getter, nullptr, nullptr, nullptr},
2418:   {(char*)"_saved_normalized_shape", (getter)THPNativeLayerNormBackward0_normalized_shape_getter, nullptr, nullptr, nullptr},
2419:   {(char*)"_saved_weight", (getter)THPNativeLayerNormBackward0_weight_getter, nullptr, nullptr, nullptr},
2420:   {(char*)"_raw_saved_weight", (getter)THPNativeLayerNormBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
2421:   {(char*)"_saved_result1", (getter)THPNativeLayerNormBackward0_result1_getter, nullptr, nullptr, nullptr},
2422:   {(char*)"_raw_saved_result1", (getter)THPNativeLayerNormBackward0_result1_raw_getter, nullptr, nullptr, nullptr},
2423:   {(char*)"_saved_result2", (getter)THPNativeLayerNormBackward0_result2_getter, nullptr, nullptr, nullptr},
2424:   {(char*)"_raw_saved_result2", (getter)THPNativeLayerNormBackward0_result2_raw_getter, nullptr, nullptr, nullptr},
2425:   {nullptr} /* sentinel */
2426: };
2427: 
2428: 
2429: 
2430: static struct PyGetSetDef NeBackward0_properties[] = {
2431:   THP_FUNCTION_DEFAULT_PROPERTIES,
2432: 
2433:   {nullptr} /* sentinel */
2434: };
2435: 
2436: 
2437: 
2438: static struct PyGetSetDef NeBackward1_properties[] = {
2439:   THP_FUNCTION_DEFAULT_PROPERTIES,
2440: 
2441:   {nullptr} /* sentinel */
2442: };
2443: 
2444: static PyObject* THPBatchNormBackwardBackward0_eps_getter(THPCppFunction *self, void *_unused) {
2445:   HANDLE_TH_ERRORS
2446:   auto prop = static_cast<BatchNormBackwardBackward0*>(self->cdata.get())->eps;
2447:   return PyFloat_FromDouble((double) prop);
2448:   END_HANDLE_TH_ERRORS
2449: }
2450: 
2451: static PyObject* THPBatchNormBackwardBackward0_grad_out_getter(THPCppFunction *self, void *_unused) {
2452:   HANDLE_TH_ERRORS
2453:   const auto& prop = static_cast<BatchNormBackwardBackward0*>(self->cdata.get())->grad_out_;
2454:   return THPVariable_Wrap(prop.unpack(self->cdata));
2455:   END_HANDLE_TH_ERRORS
2456: }
2457: 
2458: static PyObject* THPBatchNormBackwardBackward0_grad_out_raw_getter(THPCppFunction *self, void *_unused) {
2459:   HANDLE_TH_ERRORS
2460:   const auto& prop = static_cast<BatchNormBackwardBackward0*>(self->cdata.get())->grad_out_;
2461:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2462:   return obj.release().ptr();
2463:   END_HANDLE_TH_ERRORS
2464: }
2465: 
2466: static PyObject* THPBatchNormBackwardBackward0_input_getter(THPCppFunction *self, void *_unused) {
2467:   HANDLE_TH_ERRORS
2468:   const auto& prop = static_cast<BatchNormBackwardBackward0*>(self->cdata.get())->input_;
2469:   return THPVariable_Wrap(prop.unpack(self->cdata));
2470:   END_HANDLE_TH_ERRORS
2471: }
2472: 
2473: static PyObject* THPBatchNormBackwardBackward0_input_raw_getter(THPCppFunction *self, void *_unused) {
2474:   HANDLE_TH_ERRORS
2475:   const auto& prop = static_cast<BatchNormBackwardBackward0*>(self->cdata.get())->input_;
2476:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2477:   return obj.release().ptr();
2478:   END_HANDLE_TH_ERRORS
2479: }
2480: 
```

- EN: The main execution path in this span is carried by `THPNativeLayerNormBackward0_result2_raw_getter`, `cast`, `THPBatchNormBackwardBackward0_eps_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPNativeLayerNormBackward0_result2_raw_getter`, `cast`, `THPBatchNormBackwardBackward0_eps_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2481-2560

```cpp
2481: static PyObject* THPBatchNormBackwardBackward0_reserve_getter(THPCppFunction *self, void *_unused) {
2482:   HANDLE_TH_ERRORS
2483:   const auto& prop = static_cast<BatchNormBackwardBackward0*>(self->cdata.get())->reserve_;
2484:   return THPVariable_Wrap(prop.unpack(self->cdata));
2485:   END_HANDLE_TH_ERRORS
2486: }
2487: 
2488: static PyObject* THPBatchNormBackwardBackward0_reserve_raw_getter(THPCppFunction *self, void *_unused) {
2489:   HANDLE_TH_ERRORS
2490:   const auto& prop = static_cast<BatchNormBackwardBackward0*>(self->cdata.get())->reserve_;
2491:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2492:   return obj.release().ptr();
2493:   END_HANDLE_TH_ERRORS
2494: }
2495: 
2496: static PyObject* THPBatchNormBackwardBackward0_running_mean_getter(THPCppFunction *self, void *_unused) {
2497:   HANDLE_TH_ERRORS
2498:   const auto& prop = static_cast<BatchNormBackwardBackward0*>(self->cdata.get())->running_mean_;
2499:   return THPVariable_Wrap(prop.unpack(self->cdata));
2500:   END_HANDLE_TH_ERRORS
2501: }
2502: 
2503: static PyObject* THPBatchNormBackwardBackward0_running_mean_raw_getter(THPCppFunction *self, void *_unused) {
2504:   HANDLE_TH_ERRORS
2505:   const auto& prop = static_cast<BatchNormBackwardBackward0*>(self->cdata.get())->running_mean_;
2506:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2507:   return obj.release().ptr();
2508:   END_HANDLE_TH_ERRORS
2509: }
2510: 
2511: static PyObject* THPBatchNormBackwardBackward0_running_var_getter(THPCppFunction *self, void *_unused) {
2512:   HANDLE_TH_ERRORS
2513:   const auto& prop = static_cast<BatchNormBackwardBackward0*>(self->cdata.get())->running_var_;
2514:   return THPVariable_Wrap(prop.unpack(self->cdata));
2515:   END_HANDLE_TH_ERRORS
2516: }
2517: 
2518: static PyObject* THPBatchNormBackwardBackward0_running_var_raw_getter(THPCppFunction *self, void *_unused) {
2519:   HANDLE_TH_ERRORS
2520:   const auto& prop = static_cast<BatchNormBackwardBackward0*>(self->cdata.get())->running_var_;
2521:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2522:   return obj.release().ptr();
2523:   END_HANDLE_TH_ERRORS
2524: }
2525: 
2526: static PyObject* THPBatchNormBackwardBackward0_save_mean_getter(THPCppFunction *self, void *_unused) {
2527:   HANDLE_TH_ERRORS
2528:   const auto& prop = static_cast<BatchNormBackwardBackward0*>(self->cdata.get())->save_mean_;
2529:   return THPVariable_Wrap(prop.unpack(self->cdata));
2530:   END_HANDLE_TH_ERRORS
2531: }
2532: 
2533: static PyObject* THPBatchNormBackwardBackward0_save_mean_raw_getter(THPCppFunction *self, void *_unused) {
2534:   HANDLE_TH_ERRORS
2535:   const auto& prop = static_cast<BatchNormBackwardBackward0*>(self->cdata.get())->save_mean_;
2536:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2537:   return obj.release().ptr();
2538:   END_HANDLE_TH_ERRORS
2539: }
2540: 
2541: static PyObject* THPBatchNormBackwardBackward0_save_var_getter(THPCppFunction *self, void *_unused) {
2542:   HANDLE_TH_ERRORS
2543:   const auto& prop = static_cast<BatchNormBackwardBackward0*>(self->cdata.get())->save_var_;
2544:   return THPVariable_Wrap(prop.unpack(self->cdata));
2545:   END_HANDLE_TH_ERRORS
2546: }
2547: 
2548: static PyObject* THPBatchNormBackwardBackward0_save_var_raw_getter(THPCppFunction *self, void *_unused) {
2549:   HANDLE_TH_ERRORS
2550:   const auto& prop = static_cast<BatchNormBackwardBackward0*>(self->cdata.get())->save_var_;
2551:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2552:   return obj.release().ptr();
2553:   END_HANDLE_TH_ERRORS
2554: }
2555: 
2556: static PyObject* THPBatchNormBackwardBackward0_update_getter(THPCppFunction *self, void *_unused) {
2557:   HANDLE_TH_ERRORS
2558:   auto prop = static_cast<BatchNormBackwardBackward0*>(self->cdata.get())->update;
2559:   if (prop) {
2560:     Py_RETURN_TRUE;
```

- EN: The main execution path in this span is carried by `THPBatchNormBackwardBackward0_reserve_getter`, `THPVariable_Wrap`, `THPBatchNormBackwardBackward0_reserve_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPBatchNormBackwardBackward0_reserve_getter`, `THPVariable_Wrap`, `THPBatchNormBackwardBackward0_reserve_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2561-2640

```cpp
2561:   } else {
2562:     Py_RETURN_FALSE;
2563:   }
2564:   END_HANDLE_TH_ERRORS
2565: }
2566: 
2567: static PyObject* THPBatchNormBackwardBackward0_weight_getter(THPCppFunction *self, void *_unused) {
2568:   HANDLE_TH_ERRORS
2569:   const auto& prop = static_cast<BatchNormBackwardBackward0*>(self->cdata.get())->weight_;
2570:   return THPVariable_Wrap(prop.unpack(self->cdata));
2571:   END_HANDLE_TH_ERRORS
2572: }
2573: 
2574: static PyObject* THPBatchNormBackwardBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
2575:   HANDLE_TH_ERRORS
2576:   const auto& prop = static_cast<BatchNormBackwardBackward0*>(self->cdata.get())->weight_;
2577:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2578:   return obj.release().ptr();
2579:   END_HANDLE_TH_ERRORS
2580: }
2581: 
2582: static struct PyGetSetDef BatchNormBackwardBackward0_properties[] = {
2583:   THP_FUNCTION_DEFAULT_PROPERTIES,
2584:   {(char*)"_saved_eps", (getter)THPBatchNormBackwardBackward0_eps_getter, nullptr, nullptr, nullptr},
2585:   {(char*)"_saved_grad_out", (getter)THPBatchNormBackwardBackward0_grad_out_getter, nullptr, nullptr, nullptr},
2586:   {(char*)"_raw_saved_grad_out", (getter)THPBatchNormBackwardBackward0_grad_out_raw_getter, nullptr, nullptr, nullptr},
2587:   {(char*)"_saved_input", (getter)THPBatchNormBackwardBackward0_input_getter, nullptr, nullptr, nullptr},
2588:   {(char*)"_raw_saved_input", (getter)THPBatchNormBackwardBackward0_input_raw_getter, nullptr, nullptr, nullptr},
2589:   {(char*)"_saved_reserve", (getter)THPBatchNormBackwardBackward0_reserve_getter, nullptr, nullptr, nullptr},
2590:   {(char*)"_raw_saved_reserve", (getter)THPBatchNormBackwardBackward0_reserve_raw_getter, nullptr, nullptr, nullptr},
2591:   {(char*)"_saved_running_mean", (getter)THPBatchNormBackwardBackward0_running_mean_getter, nullptr, nullptr, nullptr},
2592:   {(char*)"_raw_saved_running_mean", (getter)THPBatchNormBackwardBackward0_running_mean_raw_getter, nullptr, nullptr, nullptr},
2593:   {(char*)"_saved_running_var", (getter)THPBatchNormBackwardBackward0_running_var_getter, nullptr, nullptr, nullptr},
2594:   {(char*)"_raw_saved_running_var", (getter)THPBatchNormBackwardBackward0_running_var_raw_getter, nullptr, nullptr, nullptr},
2595:   {(char*)"_saved_save_mean", (getter)THPBatchNormBackwardBackward0_save_mean_getter, nullptr, nullptr, nullptr},
2596:   {(char*)"_raw_saved_save_mean", (getter)THPBatchNormBackwardBackward0_save_mean_raw_getter, nullptr, nullptr, nullptr},
2597:   {(char*)"_saved_save_var", (getter)THPBatchNormBackwardBackward0_save_var_getter, nullptr, nullptr, nullptr},
2598:   {(char*)"_raw_saved_save_var", (getter)THPBatchNormBackwardBackward0_save_var_raw_getter, nullptr, nullptr, nullptr},
2599:   {(char*)"_saved_update", (getter)THPBatchNormBackwardBackward0_update_getter, nullptr, nullptr, nullptr},
2600:   {(char*)"_saved_weight", (getter)THPBatchNormBackwardBackward0_weight_getter, nullptr, nullptr, nullptr},
2601:   {(char*)"_raw_saved_weight", (getter)THPBatchNormBackwardBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
2602:   {nullptr} /* sentinel */
2603: };
2604: 
2605: static PyObject* THPNextafterBackward0_other_getter(THPCppFunction *self, void *_unused) {
2606:   HANDLE_TH_ERRORS
2607:   const auto& prop = static_cast<NextafterBackward0*>(self->cdata.get())->other_;
2608:   return THPVariable_Wrap(prop.unpack(self->cdata));
2609:   END_HANDLE_TH_ERRORS
2610: }
2611: 
2612: static PyObject* THPNextafterBackward0_other_raw_getter(THPCppFunction *self, void *_unused) {
2613:   HANDLE_TH_ERRORS
2614:   const auto& prop = static_cast<NextafterBackward0*>(self->cdata.get())->other_;
2615:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2616:   return obj.release().ptr();
2617:   END_HANDLE_TH_ERRORS
2618: }
2619: 
2620: static PyObject* THPNextafterBackward0_self_getter(THPCppFunction *self, void *_unused) {
2621:   HANDLE_TH_ERRORS
2622:   const auto& prop = static_cast<NextafterBackward0*>(self->cdata.get())->self_;
2623:   return THPVariable_Wrap(prop.unpack(self->cdata));
2624:   END_HANDLE_TH_ERRORS
2625: }
2626: 
2627: static PyObject* THPNextafterBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
2628:   HANDLE_TH_ERRORS
2629:   const auto& prop = static_cast<NextafterBackward0*>(self->cdata.get())->self_;
2630:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2631:   return obj.release().ptr();
2632:   END_HANDLE_TH_ERRORS
2633: }
2634: 
2635: static struct PyGetSetDef NextafterBackward0_properties[] = {
2636:   THP_FUNCTION_DEFAULT_PROPERTIES,
2637:   {(char*)"_saved_other", (getter)THPNextafterBackward0_other_getter, nullptr, nullptr, nullptr},
2638:   {(char*)"_raw_saved_other", (getter)THPNextafterBackward0_other_raw_getter, nullptr, nullptr, nullptr},
2639:   {(char*)"_saved_self", (getter)THPNextafterBackward0_self_getter, nullptr, nullptr, nullptr},
2640:   {(char*)"_raw_saved_self", (getter)THPNextafterBackward0_self_raw_getter, nullptr, nullptr, nullptr},
```

- EN: The main execution path in this span is carried by `THPBatchNormBackwardBackward0_weight_getter`, `THPVariable_Wrap`, `THPBatchNormBackwardBackward0_weight_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPBatchNormBackwardBackward0_weight_getter`, `THPVariable_Wrap`, `THPBatchNormBackwardBackward0_weight_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2641-2720

```cpp
2641:   {nullptr} /* sentinel */
2642: };
2643: 
2644: static PyObject* THPCdistBackward0_p_getter(THPCppFunction *self, void *_unused) {
2645:   HANDLE_TH_ERRORS
2646:   auto prop = static_cast<CdistBackward0*>(self->cdata.get())->p;
2647:   return PyFloat_FromDouble((double) prop);
2648:   END_HANDLE_TH_ERRORS
2649: }
2650: 
2651: static PyObject* THPCdistBackward0_x1_getter(THPCppFunction *self, void *_unused) {
2652:   HANDLE_TH_ERRORS
2653:   const auto& prop = static_cast<CdistBackward0*>(self->cdata.get())->x1_;
2654:   return THPVariable_Wrap(prop.unpack(self->cdata));
2655:   END_HANDLE_TH_ERRORS
2656: }
2657: 
2658: static PyObject* THPCdistBackward0_x1_raw_getter(THPCppFunction *self, void *_unused) {
2659:   HANDLE_TH_ERRORS
2660:   const auto& prop = static_cast<CdistBackward0*>(self->cdata.get())->x1_;
2661:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2662:   return obj.release().ptr();
2663:   END_HANDLE_TH_ERRORS
2664: }
2665: 
2666: static PyObject* THPCdistBackward0_x2_getter(THPCppFunction *self, void *_unused) {
2667:   HANDLE_TH_ERRORS
2668:   const auto& prop = static_cast<CdistBackward0*>(self->cdata.get())->x2_;
2669:   return THPVariable_Wrap(prop.unpack(self->cdata));
2670:   END_HANDLE_TH_ERRORS
2671: }
2672: 
2673: static PyObject* THPCdistBackward0_x2_raw_getter(THPCppFunction *self, void *_unused) {
2674:   HANDLE_TH_ERRORS
2675:   const auto& prop = static_cast<CdistBackward0*>(self->cdata.get())->x2_;
2676:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2677:   return obj.release().ptr();
2678:   END_HANDLE_TH_ERRORS
2679: }
2680: 
2681: static PyObject* THPCdistBackward0_result_getter(THPCppFunction *self, void *_unused) {
2682:   HANDLE_TH_ERRORS
2683:   const auto& prop = static_cast<CdistBackward0*>(self->cdata.get())->result_;
2684:   return THPVariable_Wrap(prop.unpack(self->cdata));
2685:   END_HANDLE_TH_ERRORS
2686: }
2687: 
2688: static PyObject* THPCdistBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
2689:   HANDLE_TH_ERRORS
2690:   const auto& prop = static_cast<CdistBackward0*>(self->cdata.get())->result_;
2691:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2692:   return obj.release().ptr();
2693:   END_HANDLE_TH_ERRORS
2694: }
2695: 
2696: static struct PyGetSetDef CdistBackward0_properties[] = {
2697:   THP_FUNCTION_DEFAULT_PROPERTIES,
2698:   {(char*)"_saved_p", (getter)THPCdistBackward0_p_getter, nullptr, nullptr, nullptr},
2699:   {(char*)"_saved_x1", (getter)THPCdistBackward0_x1_getter, nullptr, nullptr, nullptr},
2700:   {(char*)"_raw_saved_x1", (getter)THPCdistBackward0_x1_raw_getter, nullptr, nullptr, nullptr},
2701:   {(char*)"_saved_x2", (getter)THPCdistBackward0_x2_getter, nullptr, nullptr, nullptr},
2702:   {(char*)"_raw_saved_x2", (getter)THPCdistBackward0_x2_raw_getter, nullptr, nullptr, nullptr},
2703:   {(char*)"_saved_result", (getter)THPCdistBackward0_result_getter, nullptr, nullptr, nullptr},
2704:   {(char*)"_raw_saved_result", (getter)THPCdistBackward0_result_raw_getter, nullptr, nullptr, nullptr},
2705:   {nullptr} /* sentinel */
2706: };
2707: 
2708: static PyObject* THPNormalBackward1_mean_sym_sizes_getter(THPCppFunction *self, void *_unused) {
2709:   HANDLE_TH_ERRORS
2710:   auto prop = static_cast<NormalBackward1*>(self->cdata.get())->mean_sym_sizes;
2711:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
2712:   for (auto i : c10::irange(prop.size())) {
2713:       auto si = prop[i];
2714:       if (auto m = si.maybe_as_int()) {
2715:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
2716:       } else {
2717:         auto py_symint = py::cast(si).release().ptr();
2718:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
2719:       }
2720:   }
```

- EN: The main execution path in this span is carried by `THPCdistBackward0_p_getter`, `PyFloat_FromDouble`, `THPCdistBackward0_x1_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPCdistBackward0_p_getter`, `PyFloat_FromDouble`, `THPCdistBackward0_x1_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2721-2800

```cpp
2721:   return tup;
2722:   END_HANDLE_TH_ERRORS
2723: }
2724: 
2725: static struct PyGetSetDef NormalBackward1_properties[] = {
2726:   THP_FUNCTION_DEFAULT_PROPERTIES,
2727:   {(char*)"_saved_mean_sym_sizes", (getter)THPNormalBackward1_mean_sym_sizes_getter, nullptr, nullptr, nullptr},
2728:   {nullptr} /* sentinel */
2729: };
2730: 
2731: static PyObject* THPNormalBackward2_std_sym_sizes_getter(THPCppFunction *self, void *_unused) {
2732:   HANDLE_TH_ERRORS
2733:   auto prop = static_cast<NormalBackward2*>(self->cdata.get())->std_sym_sizes;
2734:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
2735:   for (auto i : c10::irange(prop.size())) {
2736:       auto si = prop[i];
2737:       if (auto m = si.maybe_as_int()) {
2738:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
2739:       } else {
2740:         auto py_symint = py::cast(si).release().ptr();
2741:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
2742:       }
2743:   }
2744:   return tup;
2745:   END_HANDLE_TH_ERRORS
2746: }
2747: 
2748: static struct PyGetSetDef NormalBackward2_properties[] = {
2749:   THP_FUNCTION_DEFAULT_PROPERTIES,
2750:   {(char*)"_saved_std_sym_sizes", (getter)THPNormalBackward2_std_sym_sizes_getter, nullptr, nullptr, nullptr},
2751:   {nullptr} /* sentinel */
2752: };
2753: 
2754: static PyObject* THPNormalBackward3_mean_sym_sizes_getter(THPCppFunction *self, void *_unused) {
2755:   HANDLE_TH_ERRORS
2756:   auto prop = static_cast<NormalBackward3*>(self->cdata.get())->mean_sym_sizes;
2757:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
2758:   for (auto i : c10::irange(prop.size())) {
2759:       auto si = prop[i];
2760:       if (auto m = si.maybe_as_int()) {
2761:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
2762:       } else {
2763:         auto py_symint = py::cast(si).release().ptr();
2764:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
2765:       }
2766:   }
2767:   return tup;
2768:   END_HANDLE_TH_ERRORS
2769: }
2770: 
2771: static PyObject* THPNormalBackward3_std_sym_sizes_getter(THPCppFunction *self, void *_unused) {
2772:   HANDLE_TH_ERRORS
2773:   auto prop = static_cast<NormalBackward3*>(self->cdata.get())->std_sym_sizes;
2774:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
2775:   for (auto i : c10::irange(prop.size())) {
2776:       auto si = prop[i];
2777:       if (auto m = si.maybe_as_int()) {
2778:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
2779:       } else {
2780:         auto py_symint = py::cast(si).release().ptr();
2781:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
2782:       }
2783:   }
2784:   return tup;
2785:   END_HANDLE_TH_ERRORS
2786: }
2787: 
2788: static struct PyGetSetDef NormalBackward3_properties[] = {
2789:   THP_FUNCTION_DEFAULT_PROPERTIES,
2790:   {(char*)"_saved_mean_sym_sizes", (getter)THPNormalBackward3_mean_sym_sizes_getter, nullptr, nullptr, nullptr},
2791:   {(char*)"_saved_std_sym_sizes", (getter)THPNormalBackward3_std_sym_sizes_getter, nullptr, nullptr, nullptr},
2792:   {nullptr} /* sentinel */
2793: };
2794: 
2795: static PyObject* THPSpecialLogNdtrBackward0_self_getter(THPCppFunction *self, void *_unused) {
2796:   HANDLE_TH_ERRORS
2797:   const auto& prop = static_cast<SpecialLogNdtrBackward0*>(self->cdata.get())->self_;
2798:   return THPVariable_Wrap(prop.unpack(self->cdata));
2799:   END_HANDLE_TH_ERRORS
2800: }
```

- EN: The main execution path in this span is carried by `THPNormalBackward2_std_sym_sizes_getter`, `PyTuple_New`, `PyTuple_SetItem`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPNormalBackward2_std_sym_sizes_getter`, `PyTuple_New`, `PyTuple_SetItem` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2801-2880

```cpp
2801: 
2802: static PyObject* THPSpecialLogNdtrBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
2803:   HANDLE_TH_ERRORS
2804:   const auto& prop = static_cast<SpecialLogNdtrBackward0*>(self->cdata.get())->self_;
2805:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2806:   return obj.release().ptr();
2807:   END_HANDLE_TH_ERRORS
2808: }
2809: 
2810: static PyObject* THPSpecialLogNdtrBackward0_result_getter(THPCppFunction *self, void *_unused) {
2811:   HANDLE_TH_ERRORS
2812:   const auto& prop = static_cast<SpecialLogNdtrBackward0*>(self->cdata.get())->result_;
2813:   return THPVariable_Wrap(prop.unpack(self->cdata));
2814:   END_HANDLE_TH_ERRORS
2815: }
2816: 
2817: static PyObject* THPSpecialLogNdtrBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
2818:   HANDLE_TH_ERRORS
2819:   const auto& prop = static_cast<SpecialLogNdtrBackward0*>(self->cdata.get())->result_;
2820:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2821:   return obj.release().ptr();
2822:   END_HANDLE_TH_ERRORS
2823: }
2824: 
2825: static struct PyGetSetDef SpecialLogNdtrBackward0_properties[] = {
2826:   THP_FUNCTION_DEFAULT_PROPERTIES,
2827:   {(char*)"_saved_self", (getter)THPSpecialLogNdtrBackward0_self_getter, nullptr, nullptr, nullptr},
2828:   {(char*)"_raw_saved_self", (getter)THPSpecialLogNdtrBackward0_self_raw_getter, nullptr, nullptr, nullptr},
2829:   {(char*)"_saved_result", (getter)THPSpecialLogNdtrBackward0_result_getter, nullptr, nullptr, nullptr},
2830:   {(char*)"_raw_saved_result", (getter)THPSpecialLogNdtrBackward0_result_raw_getter, nullptr, nullptr, nullptr},
2831:   {nullptr} /* sentinel */
2832: };
2833: 
2834: 
2835: 
2836: static struct PyGetSetDef RoundBackward0_properties[] = {
2837:   THP_FUNCTION_DEFAULT_PROPERTIES,
2838: 
2839:   {nullptr} /* sentinel */
2840: };
2841: 
2842: 
2843: 
2844: static struct PyGetSetDef RoundBackward1_properties[] = {
2845:   THP_FUNCTION_DEFAULT_PROPERTIES,
2846: 
2847:   {nullptr} /* sentinel */
2848: };
2849: 
2850: static PyObject* THPScatterAddBackward0_dim_getter(THPCppFunction *self, void *_unused) {
2851:   HANDLE_TH_ERRORS
2852:   auto prop = static_cast<ScatterAddBackward0*>(self->cdata.get())->dim;
2853:   return PyLong_FromUnsignedLong((int64_t) prop);
2854:   END_HANDLE_TH_ERRORS
2855: }
2856: 
2857: static PyObject* THPScatterAddBackward0_index_getter(THPCppFunction *self, void *_unused) {
2858:   HANDLE_TH_ERRORS
2859:   const auto& prop = static_cast<ScatterAddBackward0*>(self->cdata.get())->index_;
2860:   return THPVariable_Wrap(prop.unpack(self->cdata));
2861:   END_HANDLE_TH_ERRORS
2862: }
2863: 
2864: static PyObject* THPScatterAddBackward0_index_raw_getter(THPCppFunction *self, void *_unused) {
2865:   HANDLE_TH_ERRORS
2866:   const auto& prop = static_cast<ScatterAddBackward0*>(self->cdata.get())->index_;
2867:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2868:   return obj.release().ptr();
2869:   END_HANDLE_TH_ERRORS
2870: }
2871: 
2872: static struct PyGetSetDef ScatterAddBackward0_properties[] = {
2873:   THP_FUNCTION_DEFAULT_PROPERTIES,
2874:   {(char*)"_saved_dim", (getter)THPScatterAddBackward0_dim_getter, nullptr, nullptr, nullptr},
2875:   {(char*)"_saved_index", (getter)THPScatterAddBackward0_index_getter, nullptr, nullptr, nullptr},
2876:   {(char*)"_raw_saved_index", (getter)THPScatterAddBackward0_index_raw_getter, nullptr, nullptr, nullptr},
2877:   {nullptr} /* sentinel */
2878: };
2879: 
2880: static PyObject* THPSelectBackwardBackward0_dim_getter(THPCppFunction *self, void *_unused) {
```

- EN: The main execution path in this span is carried by `THPSpecialLogNdtrBackward0_self_raw_getter`, `cast`, `THPSpecialLogNdtrBackward0_result_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPSpecialLogNdtrBackward0_self_raw_getter`, `cast`, `THPSpecialLogNdtrBackward0_result_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2881-2960

```cpp
2881:   HANDLE_TH_ERRORS
2882:   auto prop = static_cast<SelectBackwardBackward0*>(self->cdata.get())->dim;
2883:   return PyLong_FromUnsignedLong((int64_t) prop);
2884:   END_HANDLE_TH_ERRORS
2885: }
2886: 
2887: static PyObject* THPSelectBackwardBackward0_index_getter(THPCppFunction *self, void *_unused) {
2888:   HANDLE_TH_ERRORS
2889:   auto prop = static_cast<SelectBackwardBackward0*>(self->cdata.get())->index;
2890:   if (auto m = prop.maybe_as_int()) {
2891:     return PyLong_FromUnsignedLong(*m);
2892:   } else {
2893:     return py::cast(prop).release().ptr();
2894:   }
2895:   END_HANDLE_TH_ERRORS
2896: }
2897: 
2898: static struct PyGetSetDef SelectBackwardBackward0_properties[] = {
2899:   THP_FUNCTION_DEFAULT_PROPERTIES,
2900:   {(char*)"_saved_dim", (getter)THPSelectBackwardBackward0_dim_getter, nullptr, nullptr, nullptr},
2901:   {(char*)"_saved_index", (getter)THPSelectBackwardBackward0_index_getter, nullptr, nullptr, nullptr},
2902:   {nullptr} /* sentinel */
2903: };
2904: 
2905: static PyObject* THPSigmoidBackward0_result_getter(THPCppFunction *self, void *_unused) {
2906:   HANDLE_TH_ERRORS
2907:   const auto& prop = static_cast<SigmoidBackward0*>(self->cdata.get())->result_;
2908:   return THPVariable_Wrap(prop.unpack(self->cdata));
2909:   END_HANDLE_TH_ERRORS
2910: }
2911: 
2912: static PyObject* THPSigmoidBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
2913:   HANDLE_TH_ERRORS
2914:   const auto& prop = static_cast<SigmoidBackward0*>(self->cdata.get())->result_;
2915:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2916:   return obj.release().ptr();
2917:   END_HANDLE_TH_ERRORS
2918: }
2919: 
2920: static struct PyGetSetDef SigmoidBackward0_properties[] = {
2921:   THP_FUNCTION_DEFAULT_PROPERTIES,
2922:   {(char*)"_saved_result", (getter)THPSigmoidBackward0_result_getter, nullptr, nullptr, nullptr},
2923:   {(char*)"_raw_saved_result", (getter)THPSigmoidBackward0_result_raw_getter, nullptr, nullptr, nullptr},
2924:   {nullptr} /* sentinel */
2925: };
2926: 
2927: static PyObject* THPSliceScatterBackward0_dim_getter(THPCppFunction *self, void *_unused) {
2928:   HANDLE_TH_ERRORS
2929:   auto prop = static_cast<SliceScatterBackward0*>(self->cdata.get())->dim;
2930:   return PyLong_FromUnsignedLong((int64_t) prop);
2931:   END_HANDLE_TH_ERRORS
2932: }
2933: 
2934: static PyObject* THPSliceScatterBackward0_end_getter(THPCppFunction *self, void *_unused) {
2935:   HANDLE_TH_ERRORS
2936:   auto opt_prop = static_cast<SliceScatterBackward0*>(self->cdata.get())->end;
2937:   if (!opt_prop.has_value()) {
2938:     Py_RETURN_NONE;
2939:   }
2940:   auto prop = opt_prop.value();
2941:   if (auto m = prop.maybe_as_int()) {
2942:     return PyLong_FromUnsignedLong(*m);
2943:   } else {
2944:     return py::cast(prop).release().ptr();
2945:   }
2946:   END_HANDLE_TH_ERRORS
2947: }
2948: 
2949: static PyObject* THPSliceScatterBackward0_start_getter(THPCppFunction *self, void *_unused) {
2950:   HANDLE_TH_ERRORS
2951:   auto opt_prop = static_cast<SliceScatterBackward0*>(self->cdata.get())->start;
2952:   if (!opt_prop.has_value()) {
2953:     Py_RETURN_NONE;
2954:   }
2955:   auto prop = opt_prop.value();
2956:   if (auto m = prop.maybe_as_int()) {
2957:     return PyLong_FromUnsignedLong(*m);
2958:   } else {
2959:     return py::cast(prop).release().ptr();
2960:   }
```

- EN: The main execution path in this span is carried by `PyLong_FromUnsignedLong`, `THPSelectBackwardBackward0_index_getter`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyLong_FromUnsignedLong`, `THPSelectBackwardBackward0_index_getter`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2961-3040

```cpp
2961:   END_HANDLE_TH_ERRORS
2962: }
2963: 
2964: static PyObject* THPSliceScatterBackward0_step_getter(THPCppFunction *self, void *_unused) {
2965:   HANDLE_TH_ERRORS
2966:   auto prop = static_cast<SliceScatterBackward0*>(self->cdata.get())->step;
2967:   if (auto m = prop.maybe_as_int()) {
2968:     return PyLong_FromUnsignedLong(*m);
2969:   } else {
2970:     return py::cast(prop).release().ptr();
2971:   }
2972:   END_HANDLE_TH_ERRORS
2973: }
2974: 
2975: static struct PyGetSetDef SliceScatterBackward0_properties[] = {
2976:   THP_FUNCTION_DEFAULT_PROPERTIES,
2977:   {(char*)"_saved_dim", (getter)THPSliceScatterBackward0_dim_getter, nullptr, nullptr, nullptr},
2978:   {(char*)"_saved_end", (getter)THPSliceScatterBackward0_end_getter, nullptr, nullptr, nullptr},
2979:   {(char*)"_saved_start", (getter)THPSliceScatterBackward0_start_getter, nullptr, nullptr, nullptr},
2980:   {(char*)"_saved_step", (getter)THPSliceScatterBackward0_step_getter, nullptr, nullptr, nullptr},
2981:   {nullptr} /* sentinel */
2982: };
2983: 
2984: static PyObject* THPSplitWithSizesBackward0_dim_getter(THPCppFunction *self, void *_unused) {
2985:   HANDLE_TH_ERRORS
2986:   auto prop = static_cast<SplitWithSizesBackward0*>(self->cdata.get())->dim;
2987:   return PyLong_FromUnsignedLong((int64_t) prop);
2988:   END_HANDLE_TH_ERRORS
2989: }
2990: 
2991: static PyObject* THPSplitWithSizesBackward0_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
2992:   HANDLE_TH_ERRORS
2993:   auto prop = static_cast<SplitWithSizesBackward0*>(self->cdata.get())->self_sym_sizes;
2994:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
2995:   for (auto i : c10::irange(prop.size())) {
2996:       auto si = prop[i];
2997:       if (auto m = si.maybe_as_int()) {
2998:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
2999:       } else {
3000:         auto py_symint = py::cast(si).release().ptr();
3001:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3002:       }
3003:   }
3004:   return tup;
3005:   END_HANDLE_TH_ERRORS
3006: }
3007: 
3008: static PyObject* THPSplitWithSizesBackward0_split_sizes_getter(THPCppFunction *self, void *_unused) {
3009:   HANDLE_TH_ERRORS
3010:   auto prop = static_cast<SplitWithSizesBackward0*>(self->cdata.get())->split_sizes;
3011:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3012:   for (auto i : c10::irange(prop.size())) {
3013:       auto si = prop[i];
3014:       if (auto m = si.maybe_as_int()) {
3015:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3016:       } else {
3017:         auto py_symint = py::cast(si).release().ptr();
3018:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3019:       }
3020:   }
3021:   return tup;
3022:   END_HANDLE_TH_ERRORS
3023: }
3024: 
3025: static struct PyGetSetDef SplitWithSizesBackward0_properties[] = {
3026:   THP_FUNCTION_DEFAULT_PROPERTIES,
3027:   {(char*)"_saved_dim", (getter)THPSplitWithSizesBackward0_dim_getter, nullptr, nullptr, nullptr},
3028:   {(char*)"_saved_self_sym_sizes", (getter)THPSplitWithSizesBackward0_self_sym_sizes_getter, nullptr, nullptr, nullptr},
3029:   {(char*)"_saved_split_sizes", (getter)THPSplitWithSizesBackward0_split_sizes_getter, nullptr, nullptr, nullptr},
3030:   {nullptr} /* sentinel */
3031: };
3032: 
3033: static PyObject* THPSplitWithSizesBackwardAutogradNestedTensor0_dim_getter(THPCppFunction *self, void *_unused) {
3034:   HANDLE_TH_ERRORS
3035:   auto prop = static_cast<SplitWithSizesBackwardAutogradNestedTensor0*>(self->cdata.get())->dim;
3036:   return PyLong_FromUnsignedLong((int64_t) prop);
3037:   END_HANDLE_TH_ERRORS
3038: }
3039: 
3040: static PyObject* THPSplitWithSizesBackwardAutogradNestedTensor0_self_getter(THPCppFunction *self, void *_unused) {
```

- EN: The main execution path in this span is carried by `THPSliceScatterBackward0_step_getter`, `PyLong_FromUnsignedLong`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPSliceScatterBackward0_step_getter`, `PyLong_FromUnsignedLong`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 3041-3120

```cpp
3041:   HANDLE_TH_ERRORS
3042:   const auto& prop = static_cast<SplitWithSizesBackwardAutogradNestedTensor0*>(self->cdata.get())->self_;
3043:   return THPVariable_Wrap(prop.unpack(self->cdata));
3044:   END_HANDLE_TH_ERRORS
3045: }
3046: 
3047: static PyObject* THPSplitWithSizesBackwardAutogradNestedTensor0_self_raw_getter(THPCppFunction *self, void *_unused) {
3048:   HANDLE_TH_ERRORS
3049:   const auto& prop = static_cast<SplitWithSizesBackwardAutogradNestedTensor0*>(self->cdata.get())->self_;
3050:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3051:   return obj.release().ptr();
3052:   END_HANDLE_TH_ERRORS
3053: }
3054: 
3055: static PyObject* THPSplitWithSizesBackwardAutogradNestedTensor0_split_sizes_getter(THPCppFunction *self, void *_unused) {
3056:   HANDLE_TH_ERRORS
3057:   auto prop = static_cast<SplitWithSizesBackwardAutogradNestedTensor0*>(self->cdata.get())->split_sizes;
3058:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3059:   for (auto i : c10::irange(prop.size())) {
3060:       auto si = prop[i];
3061:       if (auto m = si.maybe_as_int()) {
3062:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3063:       } else {
3064:         auto py_symint = py::cast(si).release().ptr();
3065:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3066:       }
3067:   }
3068:   return tup;
3069:   END_HANDLE_TH_ERRORS
3070: }
3071: 
3072: static struct PyGetSetDef SplitWithSizesBackwardAutogradNestedTensor0_properties[] = {
3073:   THP_FUNCTION_DEFAULT_PROPERTIES,
3074:   {(char*)"_saved_dim", (getter)THPSplitWithSizesBackwardAutogradNestedTensor0_dim_getter, nullptr, nullptr, nullptr},
3075:   {(char*)"_saved_self", (getter)THPSplitWithSizesBackwardAutogradNestedTensor0_self_getter, nullptr, nullptr, nullptr},
3076:   {(char*)"_raw_saved_self", (getter)THPSplitWithSizesBackwardAutogradNestedTensor0_self_raw_getter, nullptr, nullptr, nullptr},
3077:   {(char*)"_saved_split_sizes", (getter)THPSplitWithSizesBackwardAutogradNestedTensor0_split_sizes_getter, nullptr, nullptr, nullptr},
3078:   {nullptr} /* sentinel */
3079: };
3080: 
3081: 
3082: 
3083: static struct PyGetSetDef TBackward1_properties[] = {
3084:   THP_FUNCTION_DEFAULT_PROPERTIES,
3085: 
3086:   {nullptr} /* sentinel */
3087: };
3088: 
3089: static PyObject* THPTanhBackward0_result_getter(THPCppFunction *self, void *_unused) {
3090:   HANDLE_TH_ERRORS
3091:   const auto& prop = static_cast<TanhBackward0*>(self->cdata.get())->result_;
3092:   return THPVariable_Wrap(prop.unpack(self->cdata));
3093:   END_HANDLE_TH_ERRORS
3094: }
3095: 
3096: static PyObject* THPTanhBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
3097:   HANDLE_TH_ERRORS
3098:   const auto& prop = static_cast<TanhBackward0*>(self->cdata.get())->result_;
3099:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3100:   return obj.release().ptr();
3101:   END_HANDLE_TH_ERRORS
3102: }
3103: 
3104: static struct PyGetSetDef TanhBackward0_properties[] = {
3105:   THP_FUNCTION_DEFAULT_PROPERTIES,
3106:   {(char*)"_saved_result", (getter)THPTanhBackward0_result_getter, nullptr, nullptr, nullptr},
3107:   {(char*)"_raw_saved_result", (getter)THPTanhBackward0_result_raw_getter, nullptr, nullptr, nullptr},
3108:   {nullptr} /* sentinel */
3109: };
3110: 
3111: static PyObject* THPTopkBackward0_dim_getter(THPCppFunction *self, void *_unused) {
3112:   HANDLE_TH_ERRORS
3113:   auto prop = static_cast<TopkBackward0*>(self->cdata.get())->dim;
3114:   return PyLong_FromUnsignedLong((int64_t) prop);
3115:   END_HANDLE_TH_ERRORS
3116: }
3117: 
3118: static PyObject* THPTopkBackward0_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
3119:   HANDLE_TH_ERRORS
3120:   auto prop = static_cast<TopkBackward0*>(self->cdata.get())->self_sym_sizes;
```

- EN: The main execution path in this span is carried by `THPVariable_Wrap`, `THPSplitWithSizesBackwardAutogradNestedTensor0_self_raw_getter`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_Wrap`, `THPSplitWithSizesBackwardAutogradNestedTensor0_self_raw_getter`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 3121-3200

```cpp
3121:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3122:   for (auto i : c10::irange(prop.size())) {
3123:       auto si = prop[i];
3124:       if (auto m = si.maybe_as_int()) {
3125:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3126:       } else {
3127:         auto py_symint = py::cast(si).release().ptr();
3128:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3129:       }
3130:   }
3131:   return tup;
3132:   END_HANDLE_TH_ERRORS
3133: }
3134: 
3135: static PyObject* THPTopkBackward0_indices_getter(THPCppFunction *self, void *_unused) {
3136:   HANDLE_TH_ERRORS
3137:   const auto& prop = static_cast<TopkBackward0*>(self->cdata.get())->indices_;
3138:   return THPVariable_Wrap(prop.unpack(self->cdata));
3139:   END_HANDLE_TH_ERRORS
3140: }
3141: 
3142: static PyObject* THPTopkBackward0_indices_raw_getter(THPCppFunction *self, void *_unused) {
3143:   HANDLE_TH_ERRORS
3144:   const auto& prop = static_cast<TopkBackward0*>(self->cdata.get())->indices_;
3145:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3146:   return obj.release().ptr();
3147:   END_HANDLE_TH_ERRORS
3148: }
3149: 
3150: static struct PyGetSetDef TopkBackward0_properties[] = {
3151:   THP_FUNCTION_DEFAULT_PROPERTIES,
3152:   {(char*)"_saved_dim", (getter)THPTopkBackward0_dim_getter, nullptr, nullptr, nullptr},
3153:   {(char*)"_saved_self_sym_sizes", (getter)THPTopkBackward0_self_sym_sizes_getter, nullptr, nullptr, nullptr},
3154:   {(char*)"_saved_indices", (getter)THPTopkBackward0_indices_getter, nullptr, nullptr, nullptr},
3155:   {(char*)"_raw_saved_indices", (getter)THPTopkBackward0_indices_raw_getter, nullptr, nullptr, nullptr},
3156:   {nullptr} /* sentinel */
3157: };
3158: 
3159: static PyObject* THPToSparseCscBackward0_self_self_sym_blocksize_opt_getter(THPCppFunction *self, void *_unused) {
3160:   HANDLE_TH_ERRORS
3161:   auto opt_prop = static_cast<ToSparseCscBackward0*>(self->cdata.get())->self_self_sym_blocksize_opt;
3162:   if (!opt_prop.list.has_value()) {
3163:     Py_RETURN_NONE;
3164:   }
3165:   auto prop = opt_prop.list.value();
3166:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3167:   for (auto i : c10::irange(prop.size())) {
3168:       auto si = prop[i];
3169:       if (auto m = si.maybe_as_int()) {
3170:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3171:       } else {
3172:         auto py_symint = py::cast(si).release().ptr();
3173:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3174:       }
3175:   }
3176:   return tup;
3177:   END_HANDLE_TH_ERRORS
3178: }
3179: 
3180: static struct PyGetSetDef ToSparseCscBackward0_properties[] = {
3181:   THP_FUNCTION_DEFAULT_PROPERTIES,
3182:   {(char*)"_saved_self_self_sym_blocksize_opt", (getter)THPToSparseCscBackward0_self_self_sym_blocksize_opt_getter, nullptr, nullptr, nullptr},
3183:   {nullptr} /* sentinel */
3184: };
3185: 
3186: static PyObject* THPToSparseBscBackward0_self_self_sym_blocksize_opt_getter(THPCppFunction *self, void *_unused) {
3187:   HANDLE_TH_ERRORS
3188:   auto opt_prop = static_cast<ToSparseBscBackward0*>(self->cdata.get())->self_self_sym_blocksize_opt;
3189:   if (!opt_prop.list.has_value()) {
3190:     Py_RETURN_NONE;
3191:   }
3192:   auto prop = opt_prop.list.value();
3193:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3194:   for (auto i : c10::irange(prop.size())) {
3195:       auto si = prop[i];
3196:       if (auto m = si.maybe_as_int()) {
3197:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3198:       } else {
3199:         auto py_symint = py::cast(si).release().ptr();
3200:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
```

- EN: The main execution path in this span is carried by `PyTuple_New`, `PyTuple_SetItem`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyTuple_New`, `PyTuple_SetItem`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3201-3280

```cpp
3201:       }
3202:   }
3203:   return tup;
3204:   END_HANDLE_TH_ERRORS
3205: }
3206: 
3207: static struct PyGetSetDef ToSparseBscBackward0_properties[] = {
3208:   THP_FUNCTION_DEFAULT_PROPERTIES,
3209:   {(char*)"_saved_self_self_sym_blocksize_opt", (getter)THPToSparseBscBackward0_self_self_sym_blocksize_opt_getter, nullptr, nullptr, nullptr},
3210:   {nullptr} /* sentinel */
3211: };
3212: 
3213: static PyObject* THPUnfoldBackwardBackward0_dim_getter(THPCppFunction *self, void *_unused) {
3214:   HANDLE_TH_ERRORS
3215:   auto prop = static_cast<UnfoldBackwardBackward0*>(self->cdata.get())->dim;
3216:   return PyLong_FromUnsignedLong((int64_t) prop);
3217:   END_HANDLE_TH_ERRORS
3218: }
3219: 
3220: static PyObject* THPUnfoldBackwardBackward0_size_getter(THPCppFunction *self, void *_unused) {
3221:   HANDLE_TH_ERRORS
3222:   auto prop = static_cast<UnfoldBackwardBackward0*>(self->cdata.get())->size;
3223:   return PyLong_FromUnsignedLong((int64_t) prop);
3224:   END_HANDLE_TH_ERRORS
3225: }
3226: 
3227: static PyObject* THPUnfoldBackwardBackward0_step_getter(THPCppFunction *self, void *_unused) {
3228:   HANDLE_TH_ERRORS
3229:   auto prop = static_cast<UnfoldBackwardBackward0*>(self->cdata.get())->step;
3230:   return PyLong_FromUnsignedLong((int64_t) prop);
3231:   END_HANDLE_TH_ERRORS
3232: }
3233: 
3234: static struct PyGetSetDef UnfoldBackwardBackward0_properties[] = {
3235:   THP_FUNCTION_DEFAULT_PROPERTIES,
3236:   {(char*)"_saved_dim", (getter)THPUnfoldBackwardBackward0_dim_getter, nullptr, nullptr, nullptr},
3237:   {(char*)"_saved_size", (getter)THPUnfoldBackwardBackward0_size_getter, nullptr, nullptr, nullptr},
3238:   {(char*)"_saved_step", (getter)THPUnfoldBackwardBackward0_step_getter, nullptr, nullptr, nullptr},
3239:   {nullptr} /* sentinel */
3240: };
3241: 
3242: 
3243: 
3244: static struct PyGetSetDef UniformBackward0_properties[] = {
3245:   THP_FUNCTION_DEFAULT_PROPERTIES,
3246: 
3247:   {nullptr} /* sentinel */
3248: };
3249: 
3250: 
3251: 
3252: static struct PyGetSetDef UniqueDimBackward0_properties[] = {
3253:   THP_FUNCTION_DEFAULT_PROPERTIES,
3254: 
3255:   {nullptr} /* sentinel */
3256: };
3257: 
3258: 
3259: 
3260: static struct PyGetSetDef UniqueConsecutiveBackward0_properties[] = {
3261:   THP_FUNCTION_DEFAULT_PROPERTIES,
3262: 
3263:   {nullptr} /* sentinel */
3264: };
3265: 
3266: static PyObject* THPBinaryCrossEntropyBackwardBackward0_grad_output_getter(THPCppFunction *self, void *_unused) {
3267:   HANDLE_TH_ERRORS
3268:   const auto& prop = static_cast<BinaryCrossEntropyBackwardBackward0*>(self->cdata.get())->grad_output_;
3269:   return THPVariable_Wrap(prop.unpack(self->cdata));
3270:   END_HANDLE_TH_ERRORS
3271: }
3272: 
3273: static PyObject* THPBinaryCrossEntropyBackwardBackward0_grad_output_raw_getter(THPCppFunction *self, void *_unused) {
3274:   HANDLE_TH_ERRORS
3275:   const auto& prop = static_cast<BinaryCrossEntropyBackwardBackward0*>(self->cdata.get())->grad_output_;
3276:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3277:   return obj.release().ptr();
3278:   END_HANDLE_TH_ERRORS
3279: }
3280: 
```

- EN: The main execution path in this span is carried by `THPUnfoldBackwardBackward0_dim_getter`, `PyLong_FromUnsignedLong`, `THPUnfoldBackwardBackward0_size_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPUnfoldBackwardBackward0_dim_getter`, `PyLong_FromUnsignedLong`, `THPUnfoldBackwardBackward0_size_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3281-3360

```cpp
3281: static PyObject* THPBinaryCrossEntropyBackwardBackward0_reduction_getter(THPCppFunction *self, void *_unused) {
3282:   HANDLE_TH_ERRORS
3283:   auto prop = static_cast<BinaryCrossEntropyBackwardBackward0*>(self->cdata.get())->reduction;
3284:   return PyLong_FromUnsignedLong((int64_t) prop);
3285:   END_HANDLE_TH_ERRORS
3286: }
3287: 
3288: static PyObject* THPBinaryCrossEntropyBackwardBackward0_self_getter(THPCppFunction *self, void *_unused) {
3289:   HANDLE_TH_ERRORS
3290:   const auto& prop = static_cast<BinaryCrossEntropyBackwardBackward0*>(self->cdata.get())->self_;
3291:   return THPVariable_Wrap(prop.unpack(self->cdata));
3292:   END_HANDLE_TH_ERRORS
3293: }
3294: 
3295: static PyObject* THPBinaryCrossEntropyBackwardBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
3296:   HANDLE_TH_ERRORS
3297:   const auto& prop = static_cast<BinaryCrossEntropyBackwardBackward0*>(self->cdata.get())->self_;
3298:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3299:   return obj.release().ptr();
3300:   END_HANDLE_TH_ERRORS
3301: }
3302: 
3303: static PyObject* THPBinaryCrossEntropyBackwardBackward0_target_getter(THPCppFunction *self, void *_unused) {
3304:   HANDLE_TH_ERRORS
3305:   const auto& prop = static_cast<BinaryCrossEntropyBackwardBackward0*>(self->cdata.get())->target_;
3306:   return THPVariable_Wrap(prop.unpack(self->cdata));
3307:   END_HANDLE_TH_ERRORS
3308: }
3309: 
3310: static PyObject* THPBinaryCrossEntropyBackwardBackward0_target_raw_getter(THPCppFunction *self, void *_unused) {
3311:   HANDLE_TH_ERRORS
3312:   const auto& prop = static_cast<BinaryCrossEntropyBackwardBackward0*>(self->cdata.get())->target_;
3313:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3314:   return obj.release().ptr();
3315:   END_HANDLE_TH_ERRORS
3316: }
3317: 
3318: static PyObject* THPBinaryCrossEntropyBackwardBackward0_weight_getter(THPCppFunction *self, void *_unused) {
3319:   HANDLE_TH_ERRORS
3320:   const auto& prop = static_cast<BinaryCrossEntropyBackwardBackward0*>(self->cdata.get())->weight_;
3321:   return THPVariable_Wrap(prop.unpack(self->cdata));
3322:   END_HANDLE_TH_ERRORS
3323: }
3324: 
3325: static PyObject* THPBinaryCrossEntropyBackwardBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
3326:   HANDLE_TH_ERRORS
3327:   const auto& prop = static_cast<BinaryCrossEntropyBackwardBackward0*>(self->cdata.get())->weight_;
3328:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3329:   return obj.release().ptr();
3330:   END_HANDLE_TH_ERRORS
3331: }
3332: 
3333: static struct PyGetSetDef BinaryCrossEntropyBackwardBackward0_properties[] = {
3334:   THP_FUNCTION_DEFAULT_PROPERTIES,
3335:   {(char*)"_saved_grad_output", (getter)THPBinaryCrossEntropyBackwardBackward0_grad_output_getter, nullptr, nullptr, nullptr},
3336:   {(char*)"_raw_saved_grad_output", (getter)THPBinaryCrossEntropyBackwardBackward0_grad_output_raw_getter, nullptr, nullptr, nullptr},
3337:   {(char*)"_saved_reduction", (getter)THPBinaryCrossEntropyBackwardBackward0_reduction_getter, nullptr, nullptr, nullptr},
3338:   {(char*)"_saved_self", (getter)THPBinaryCrossEntropyBackwardBackward0_self_getter, nullptr, nullptr, nullptr},
3339:   {(char*)"_raw_saved_self", (getter)THPBinaryCrossEntropyBackwardBackward0_self_raw_getter, nullptr, nullptr, nullptr},
3340:   {(char*)"_saved_target", (getter)THPBinaryCrossEntropyBackwardBackward0_target_getter, nullptr, nullptr, nullptr},
3341:   {(char*)"_raw_saved_target", (getter)THPBinaryCrossEntropyBackwardBackward0_target_raw_getter, nullptr, nullptr, nullptr},
3342:   {(char*)"_saved_weight", (getter)THPBinaryCrossEntropyBackwardBackward0_weight_getter, nullptr, nullptr, nullptr},
3343:   {(char*)"_raw_saved_weight", (getter)THPBinaryCrossEntropyBackwardBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
3344:   {nullptr} /* sentinel */
3345: };
3346: 
3347: static PyObject* THPMseLossBackward0_reduction_getter(THPCppFunction *self, void *_unused) {
3348:   HANDLE_TH_ERRORS
3349:   auto prop = static_cast<MseLossBackward0*>(self->cdata.get())->reduction;
3350:   return PyLong_FromUnsignedLong((int64_t) prop);
3351:   END_HANDLE_TH_ERRORS
3352: }
3353: 
3354: static PyObject* THPMseLossBackward0_self_getter(THPCppFunction *self, void *_unused) {
3355:   HANDLE_TH_ERRORS
3356:   const auto& prop = static_cast<MseLossBackward0*>(self->cdata.get())->self_;
3357:   return THPVariable_Wrap(prop.unpack(self->cdata));
3358:   END_HANDLE_TH_ERRORS
3359: }
3360: 
```

- EN: The main execution path in this span is carried by `THPBinaryCrossEntropyBackwardBackward0_reduction_getter`, `PyLong_FromUnsignedLong`, `THPBinaryCrossEntropyBackwardBackward0_self_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPBinaryCrossEntropyBackwardBackward0_reduction_getter`, `PyLong_FromUnsignedLong`, `THPBinaryCrossEntropyBackwardBackward0_self_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3361-3440

```cpp
3361: static PyObject* THPMseLossBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
3362:   HANDLE_TH_ERRORS
3363:   const auto& prop = static_cast<MseLossBackward0*>(self->cdata.get())->self_;
3364:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3365:   return obj.release().ptr();
3366:   END_HANDLE_TH_ERRORS
3367: }
3368: 
3369: static PyObject* THPMseLossBackward0_target_getter(THPCppFunction *self, void *_unused) {
3370:   HANDLE_TH_ERRORS
3371:   const auto& prop = static_cast<MseLossBackward0*>(self->cdata.get())->target_;
3372:   return THPVariable_Wrap(prop.unpack(self->cdata));
3373:   END_HANDLE_TH_ERRORS
3374: }
3375: 
3376: static PyObject* THPMseLossBackward0_target_raw_getter(THPCppFunction *self, void *_unused) {
3377:   HANDLE_TH_ERRORS
3378:   const auto& prop = static_cast<MseLossBackward0*>(self->cdata.get())->target_;
3379:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3380:   return obj.release().ptr();
3381:   END_HANDLE_TH_ERRORS
3382: }
3383: 
3384: static struct PyGetSetDef MseLossBackward0_properties[] = {
3385:   THP_FUNCTION_DEFAULT_PROPERTIES,
3386:   {(char*)"_saved_reduction", (getter)THPMseLossBackward0_reduction_getter, nullptr, nullptr, nullptr},
3387:   {(char*)"_saved_self", (getter)THPMseLossBackward0_self_getter, nullptr, nullptr, nullptr},
3388:   {(char*)"_raw_saved_self", (getter)THPMseLossBackward0_self_raw_getter, nullptr, nullptr, nullptr},
3389:   {(char*)"_saved_target", (getter)THPMseLossBackward0_target_getter, nullptr, nullptr, nullptr},
3390:   {(char*)"_raw_saved_target", (getter)THPMseLossBackward0_target_raw_getter, nullptr, nullptr, nullptr},
3391:   {nullptr} /* sentinel */
3392: };
3393: 
3394: static PyObject* THPNllLoss2DBackward0_ignore_index_getter(THPCppFunction *self, void *_unused) {
3395:   HANDLE_TH_ERRORS
3396:   auto prop = static_cast<NllLoss2DBackward0*>(self->cdata.get())->ignore_index;
3397:   if (auto m = prop.maybe_as_int()) {
3398:     return PyLong_FromUnsignedLong(*m);
3399:   } else {
3400:     return py::cast(prop).release().ptr();
3401:   }
3402:   END_HANDLE_TH_ERRORS
3403: }
3404: 
3405: static PyObject* THPNllLoss2DBackward0_reduction_getter(THPCppFunction *self, void *_unused) {
3406:   HANDLE_TH_ERRORS
3407:   auto prop = static_cast<NllLoss2DBackward0*>(self->cdata.get())->reduction;
3408:   return PyLong_FromUnsignedLong((int64_t) prop);
3409:   END_HANDLE_TH_ERRORS
3410: }
3411: 
3412: static PyObject* THPNllLoss2DBackward0_self_getter(THPCppFunction *self, void *_unused) {
3413:   HANDLE_TH_ERRORS
3414:   const auto& prop = static_cast<NllLoss2DBackward0*>(self->cdata.get())->self_;
3415:   return THPVariable_Wrap(prop.unpack(self->cdata));
3416:   END_HANDLE_TH_ERRORS
3417: }
3418: 
3419: static PyObject* THPNllLoss2DBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
3420:   HANDLE_TH_ERRORS
3421:   const auto& prop = static_cast<NllLoss2DBackward0*>(self->cdata.get())->self_;
3422:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3423:   return obj.release().ptr();
3424:   END_HANDLE_TH_ERRORS
3425: }
3426: 
3427: static PyObject* THPNllLoss2DBackward0_target_getter(THPCppFunction *self, void *_unused) {
3428:   HANDLE_TH_ERRORS
3429:   const auto& prop = static_cast<NllLoss2DBackward0*>(self->cdata.get())->target_;
3430:   return THPVariable_Wrap(prop.unpack(self->cdata));
3431:   END_HANDLE_TH_ERRORS
3432: }
3433: 
3434: static PyObject* THPNllLoss2DBackward0_target_raw_getter(THPCppFunction *self, void *_unused) {
3435:   HANDLE_TH_ERRORS
3436:   const auto& prop = static_cast<NllLoss2DBackward0*>(self->cdata.get())->target_;
3437:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3438:   return obj.release().ptr();
3439:   END_HANDLE_TH_ERRORS
3440: }
```

- EN: The main execution path in this span is carried by `THPMseLossBackward0_self_raw_getter`, `cast`, `THPMseLossBackward0_target_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPMseLossBackward0_self_raw_getter`, `cast`, `THPMseLossBackward0_target_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3441-3520

```cpp
3441: 
3442: static PyObject* THPNllLoss2DBackward0_weight_getter(THPCppFunction *self, void *_unused) {
3443:   HANDLE_TH_ERRORS
3444:   const auto& prop = static_cast<NllLoss2DBackward0*>(self->cdata.get())->weight_;
3445:   return THPVariable_Wrap(prop.unpack(self->cdata));
3446:   END_HANDLE_TH_ERRORS
3447: }
3448: 
3449: static PyObject* THPNllLoss2DBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
3450:   HANDLE_TH_ERRORS
3451:   const auto& prop = static_cast<NllLoss2DBackward0*>(self->cdata.get())->weight_;
3452:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3453:   return obj.release().ptr();
3454:   END_HANDLE_TH_ERRORS
3455: }
3456: 
3457: static PyObject* THPNllLoss2DBackward0_total_weight_getter(THPCppFunction *self, void *_unused) {
3458:   HANDLE_TH_ERRORS
3459:   const auto& prop = static_cast<NllLoss2DBackward0*>(self->cdata.get())->total_weight_;
3460:   return THPVariable_Wrap(prop.unpack(self->cdata));
3461:   END_HANDLE_TH_ERRORS
3462: }
3463: 
3464: static PyObject* THPNllLoss2DBackward0_total_weight_raw_getter(THPCppFunction *self, void *_unused) {
3465:   HANDLE_TH_ERRORS
3466:   const auto& prop = static_cast<NllLoss2DBackward0*>(self->cdata.get())->total_weight_;
3467:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3468:   return obj.release().ptr();
3469:   END_HANDLE_TH_ERRORS
3470: }
3471: 
3472: static struct PyGetSetDef NllLoss2DBackward0_properties[] = {
3473:   THP_FUNCTION_DEFAULT_PROPERTIES,
3474:   {(char*)"_saved_ignore_index", (getter)THPNllLoss2DBackward0_ignore_index_getter, nullptr, nullptr, nullptr},
3475:   {(char*)"_saved_reduction", (getter)THPNllLoss2DBackward0_reduction_getter, nullptr, nullptr, nullptr},
3476:   {(char*)"_saved_self", (getter)THPNllLoss2DBackward0_self_getter, nullptr, nullptr, nullptr},
3477:   {(char*)"_raw_saved_self", (getter)THPNllLoss2DBackward0_self_raw_getter, nullptr, nullptr, nullptr},
3478:   {(char*)"_saved_target", (getter)THPNllLoss2DBackward0_target_getter, nullptr, nullptr, nullptr},
3479:   {(char*)"_raw_saved_target", (getter)THPNllLoss2DBackward0_target_raw_getter, nullptr, nullptr, nullptr},
3480:   {(char*)"_saved_weight", (getter)THPNllLoss2DBackward0_weight_getter, nullptr, nullptr, nullptr},
3481:   {(char*)"_raw_saved_weight", (getter)THPNllLoss2DBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
3482:   {(char*)"_saved_total_weight", (getter)THPNllLoss2DBackward0_total_weight_getter, nullptr, nullptr, nullptr},
3483:   {(char*)"_raw_saved_total_weight", (getter)THPNllLoss2DBackward0_total_weight_raw_getter, nullptr, nullptr, nullptr},
3484:   {nullptr} /* sentinel */
3485: };
3486: 
3487: static PyObject* THPSoftMarginLossBackward0_reduction_getter(THPCppFunction *self, void *_unused) {
3488:   HANDLE_TH_ERRORS
3489:   auto prop = static_cast<SoftMarginLossBackward0*>(self->cdata.get())->reduction;
3490:   return PyLong_FromUnsignedLong((int64_t) prop);
3491:   END_HANDLE_TH_ERRORS
3492: }
3493: 
3494: static PyObject* THPSoftMarginLossBackward0_self_getter(THPCppFunction *self, void *_unused) {
3495:   HANDLE_TH_ERRORS
3496:   const auto& prop = static_cast<SoftMarginLossBackward0*>(self->cdata.get())->self_;
3497:   return THPVariable_Wrap(prop.unpack(self->cdata));
3498:   END_HANDLE_TH_ERRORS
3499: }
3500: 
3501: static PyObject* THPSoftMarginLossBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
3502:   HANDLE_TH_ERRORS
3503:   const auto& prop = static_cast<SoftMarginLossBackward0*>(self->cdata.get())->self_;
3504:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3505:   return obj.release().ptr();
3506:   END_HANDLE_TH_ERRORS
3507: }
3508: 
3509: static PyObject* THPSoftMarginLossBackward0_target_getter(THPCppFunction *self, void *_unused) {
3510:   HANDLE_TH_ERRORS
3511:   const auto& prop = static_cast<SoftMarginLossBackward0*>(self->cdata.get())->target_;
3512:   return THPVariable_Wrap(prop.unpack(self->cdata));
3513:   END_HANDLE_TH_ERRORS
3514: }
3515: 
3516: static PyObject* THPSoftMarginLossBackward0_target_raw_getter(THPCppFunction *self, void *_unused) {
3517:   HANDLE_TH_ERRORS
3518:   const auto& prop = static_cast<SoftMarginLossBackward0*>(self->cdata.get())->target_;
3519:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3520:   return obj.release().ptr();
```

- EN: The main execution path in this span is carried by `THPNllLoss2DBackward0_weight_getter`, `THPVariable_Wrap`, `THPNllLoss2DBackward0_weight_raw_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPNllLoss2DBackward0_weight_getter`, `THPVariable_Wrap`, `THPNllLoss2DBackward0_weight_raw_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3521-3600

```cpp
3521:   END_HANDLE_TH_ERRORS
3522: }
3523: 
3524: static struct PyGetSetDef SoftMarginLossBackward0_properties[] = {
3525:   THP_FUNCTION_DEFAULT_PROPERTIES,
3526:   {(char*)"_saved_reduction", (getter)THPSoftMarginLossBackward0_reduction_getter, nullptr, nullptr, nullptr},
3527:   {(char*)"_saved_self", (getter)THPSoftMarginLossBackward0_self_getter, nullptr, nullptr, nullptr},
3528:   {(char*)"_raw_saved_self", (getter)THPSoftMarginLossBackward0_self_raw_getter, nullptr, nullptr, nullptr},
3529:   {(char*)"_saved_target", (getter)THPSoftMarginLossBackward0_target_getter, nullptr, nullptr, nullptr},
3530:   {(char*)"_raw_saved_target", (getter)THPSoftMarginLossBackward0_target_raw_getter, nullptr, nullptr, nullptr},
3531:   {nullptr} /* sentinel */
3532: };
3533: 
3534: static PyObject* THPMishBackward0_self_getter(THPCppFunction *self, void *_unused) {
3535:   HANDLE_TH_ERRORS
3536:   const auto& prop = static_cast<MishBackward0*>(self->cdata.get())->self_;
3537:   return THPVariable_Wrap(prop.unpack(self->cdata));
3538:   END_HANDLE_TH_ERRORS
3539: }
3540: 
3541: static PyObject* THPMishBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
3542:   HANDLE_TH_ERRORS
3543:   const auto& prop = static_cast<MishBackward0*>(self->cdata.get())->self_;
3544:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3545:   return obj.release().ptr();
3546:   END_HANDLE_TH_ERRORS
3547: }
3548: 
3549: static struct PyGetSetDef MishBackward0_properties[] = {
3550:   THP_FUNCTION_DEFAULT_PROPERTIES,
3551:   {(char*)"_saved_self", (getter)THPMishBackward0_self_getter, nullptr, nullptr, nullptr},
3552:   {(char*)"_raw_saved_self", (getter)THPMishBackward0_self_raw_getter, nullptr, nullptr, nullptr},
3553:   {nullptr} /* sentinel */
3554: };
3555: 
3556: static PyObject* THPGeluBackward0_approximate_getter(THPCppFunction *self, void *_unused) {
3557:   HANDLE_TH_ERRORS
3558:   auto prop = static_cast<GeluBackward0*>(self->cdata.get())->approximate;
3559:   return PyUnicode_FromStringAndSize(prop.data(), prop.size());
3560:   END_HANDLE_TH_ERRORS
3561: }
3562: 
3563: static PyObject* THPGeluBackward0_self_getter(THPCppFunction *self, void *_unused) {
3564:   HANDLE_TH_ERRORS
3565:   const auto& prop = static_cast<GeluBackward0*>(self->cdata.get())->self_;
3566:   return THPVariable_Wrap(prop.unpack(self->cdata));
3567:   END_HANDLE_TH_ERRORS
3568: }
3569: 
3570: static PyObject* THPGeluBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
3571:   HANDLE_TH_ERRORS
3572:   const auto& prop = static_cast<GeluBackward0*>(self->cdata.get())->self_;
3573:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3574:   return obj.release().ptr();
3575:   END_HANDLE_TH_ERRORS
3576: }
3577: 
3578: static struct PyGetSetDef GeluBackward0_properties[] = {
3579:   THP_FUNCTION_DEFAULT_PROPERTIES,
3580:   {(char*)"_saved_approximate", (getter)THPGeluBackward0_approximate_getter, nullptr, nullptr, nullptr},
3581:   {(char*)"_saved_self", (getter)THPGeluBackward0_self_getter, nullptr, nullptr, nullptr},
3582:   {(char*)"_raw_saved_self", (getter)THPGeluBackward0_self_raw_getter, nullptr, nullptr, nullptr},
3583:   {nullptr} /* sentinel */
3584: };
3585: 
3586: static PyObject* THPGluBackward0_dim_getter(THPCppFunction *self, void *_unused) {
3587:   HANDLE_TH_ERRORS
3588:   auto prop = static_cast<GluBackward0*>(self->cdata.get())->dim;
3589:   return PyLong_FromUnsignedLong((int64_t) prop);
3590:   END_HANDLE_TH_ERRORS
3591: }
3592: 
3593: static PyObject* THPGluBackward0_self_getter(THPCppFunction *self, void *_unused) {
3594:   HANDLE_TH_ERRORS
3595:   const auto& prop = static_cast<GluBackward0*>(self->cdata.get())->self_;
3596:   return THPVariable_Wrap(prop.unpack(self->cdata));
3597:   END_HANDLE_TH_ERRORS
3598: }
3599: 
3600: static PyObject* THPGluBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
```

- EN: The main execution path in this span is carried by `THPMishBackward0_self_getter`, `THPVariable_Wrap`, `THPMishBackward0_self_raw_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPMishBackward0_self_getter`, `THPVariable_Wrap`, `THPMishBackward0_self_raw_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3601-3680

```cpp
3601:   HANDLE_TH_ERRORS
3602:   const auto& prop = static_cast<GluBackward0*>(self->cdata.get())->self_;
3603:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3604:   return obj.release().ptr();
3605:   END_HANDLE_TH_ERRORS
3606: }
3607: 
3608: static struct PyGetSetDef GluBackward0_properties[] = {
3609:   THP_FUNCTION_DEFAULT_PROPERTIES,
3610:   {(char*)"_saved_dim", (getter)THPGluBackward0_dim_getter, nullptr, nullptr, nullptr},
3611:   {(char*)"_saved_self", (getter)THPGluBackward0_self_getter, nullptr, nullptr, nullptr},
3612:   {(char*)"_raw_saved_self", (getter)THPGluBackward0_self_raw_getter, nullptr, nullptr, nullptr},
3613:   {nullptr} /* sentinel */
3614: };
3615: 
3616: static PyObject* THPHardshrinkBackward0_lambd_getter(THPCppFunction *self, void *_unused) {
3617:   HANDLE_TH_ERRORS
3618:   auto prop = static_cast<HardshrinkBackward0*>(self->cdata.get())->lambd;
3619:   if (prop.isComplex()) {
3620:     auto cprop = prop.to<c10::complex<double>>();
3621:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
3622:   } else if (prop.isFloatingPoint()) {
3623:     return PyFloat_FromDouble(prop.to<double>());
3624:   } else if (prop.isIntegral(/*includeBool=*/false)) {
3625:     return PyLong_FromLong(prop.to<int64_t>());
3626:   } else if (prop.isBoolean()) {
3627:     if (prop.to<bool>()) {
3628:       Py_RETURN_TRUE;
3629:     } else {
3630:       Py_RETURN_FALSE;
3631:     }
3632:   } else {
3633:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
3634:     return nullptr;
3635:   }
3636:   END_HANDLE_TH_ERRORS
3637: }
3638: 
3639: static PyObject* THPHardshrinkBackward0_self_getter(THPCppFunction *self, void *_unused) {
3640:   HANDLE_TH_ERRORS
3641:   const auto& prop = static_cast<HardshrinkBackward0*>(self->cdata.get())->self_;
3642:   return THPVariable_Wrap(prop.unpack(self->cdata));
3643:   END_HANDLE_TH_ERRORS
3644: }
3645: 
3646: static PyObject* THPHardshrinkBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
3647:   HANDLE_TH_ERRORS
3648:   const auto& prop = static_cast<HardshrinkBackward0*>(self->cdata.get())->self_;
3649:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3650:   return obj.release().ptr();
3651:   END_HANDLE_TH_ERRORS
3652: }
3653: 
3654: static struct PyGetSetDef HardshrinkBackward0_properties[] = {
3655:   THP_FUNCTION_DEFAULT_PROPERTIES,
3656:   {(char*)"_saved_lambd", (getter)THPHardshrinkBackward0_lambd_getter, nullptr, nullptr, nullptr},
3657:   {(char*)"_saved_self", (getter)THPHardshrinkBackward0_self_getter, nullptr, nullptr, nullptr},
3658:   {(char*)"_raw_saved_self", (getter)THPHardshrinkBackward0_self_raw_getter, nullptr, nullptr, nullptr},
3659:   {nullptr} /* sentinel */
3660: };
3661: 
3662: static PyObject* THPHardtanhBackward0_max_val_getter(THPCppFunction *self, void *_unused) {
3663:   HANDLE_TH_ERRORS
3664:   auto prop = static_cast<HardtanhBackward0*>(self->cdata.get())->max_val;
3665:   if (prop.isComplex()) {
3666:     auto cprop = prop.to<c10::complex<double>>();
3667:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
3668:   } else if (prop.isFloatingPoint()) {
3669:     return PyFloat_FromDouble(prop.to<double>());
3670:   } else if (prop.isIntegral(/*includeBool=*/false)) {
3671:     return PyLong_FromLong(prop.to<int64_t>());
3672:   } else if (prop.isBoolean()) {
3673:     if (prop.to<bool>()) {
3674:       Py_RETURN_TRUE;
3675:     } else {
3676:       Py_RETURN_FALSE;
3677:     }
3678:   } else {
3679:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
3680:     return nullptr;
```

- EN: The main execution path in this span is carried by `cast`, `THPHardshrinkBackward0_lambd_getter`, `PyComplex_FromDoubles`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `cast`, `THPHardshrinkBackward0_lambd_getter`, `PyComplex_FromDoubles` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3681-3760

```cpp
3681:   }
3682:   END_HANDLE_TH_ERRORS
3683: }
3684: 
3685: static PyObject* THPHardtanhBackward0_min_val_getter(THPCppFunction *self, void *_unused) {
3686:   HANDLE_TH_ERRORS
3687:   auto prop = static_cast<HardtanhBackward0*>(self->cdata.get())->min_val;
3688:   if (prop.isComplex()) {
3689:     auto cprop = prop.to<c10::complex<double>>();
3690:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
3691:   } else if (prop.isFloatingPoint()) {
3692:     return PyFloat_FromDouble(prop.to<double>());
3693:   } else if (prop.isIntegral(/*includeBool=*/false)) {
3694:     return PyLong_FromLong(prop.to<int64_t>());
3695:   } else if (prop.isBoolean()) {
3696:     if (prop.to<bool>()) {
3697:       Py_RETURN_TRUE;
3698:     } else {
3699:       Py_RETURN_FALSE;
3700:     }
3701:   } else {
3702:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
3703:     return nullptr;
3704:   }
3705:   END_HANDLE_TH_ERRORS
3706: }
3707: 
3708: static PyObject* THPHardtanhBackward0_self_getter(THPCppFunction *self, void *_unused) {
3709:   HANDLE_TH_ERRORS
3710:   const auto& prop = static_cast<HardtanhBackward0*>(self->cdata.get())->self_;
3711:   return THPVariable_Wrap(prop.unpack(self->cdata));
3712:   END_HANDLE_TH_ERRORS
3713: }
3714: 
3715: static PyObject* THPHardtanhBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
3716:   HANDLE_TH_ERRORS
3717:   const auto& prop = static_cast<HardtanhBackward0*>(self->cdata.get())->self_;
3718:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3719:   return obj.release().ptr();
3720:   END_HANDLE_TH_ERRORS
3721: }
3722: 
3723: static struct PyGetSetDef HardtanhBackward0_properties[] = {
3724:   THP_FUNCTION_DEFAULT_PROPERTIES,
3725:   {(char*)"_saved_max_val", (getter)THPHardtanhBackward0_max_val_getter, nullptr, nullptr, nullptr},
3726:   {(char*)"_saved_min_val", (getter)THPHardtanhBackward0_min_val_getter, nullptr, nullptr, nullptr},
3727:   {(char*)"_saved_self", (getter)THPHardtanhBackward0_self_getter, nullptr, nullptr, nullptr},
3728:   {(char*)"_raw_saved_self", (getter)THPHardtanhBackward0_self_raw_getter, nullptr, nullptr, nullptr},
3729:   {nullptr} /* sentinel */
3730: };
3731: 
3732: static PyObject* THPLeakyReluBackward0_negative_slope_getter(THPCppFunction *self, void *_unused) {
3733:   HANDLE_TH_ERRORS
3734:   auto prop = static_cast<LeakyReluBackward0*>(self->cdata.get())->negative_slope;
3735:   if (prop.isComplex()) {
3736:     auto cprop = prop.to<c10::complex<double>>();
3737:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
3738:   } else if (prop.isFloatingPoint()) {
3739:     return PyFloat_FromDouble(prop.to<double>());
3740:   } else if (prop.isIntegral(/*includeBool=*/false)) {
3741:     return PyLong_FromLong(prop.to<int64_t>());
3742:   } else if (prop.isBoolean()) {
3743:     if (prop.to<bool>()) {
3744:       Py_RETURN_TRUE;
3745:     } else {
3746:       Py_RETURN_FALSE;
3747:     }
3748:   } else {
3749:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
3750:     return nullptr;
3751:   }
3752:   END_HANDLE_TH_ERRORS
3753: }
3754: 
3755: static PyObject* THPLeakyReluBackward0_self_getter(THPCppFunction *self, void *_unused) {
3756:   HANDLE_TH_ERRORS
3757:   const auto& prop = static_cast<LeakyReluBackward0*>(self->cdata.get())->self_;
3758:   return THPVariable_Wrap(prop.unpack(self->cdata));
3759:   END_HANDLE_TH_ERRORS
3760: }
```

- EN: The main execution path in this span is carried by `THPHardtanhBackward0_min_val_getter`, `PyComplex_FromDoubles`, `PyFloat_FromDouble`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPHardtanhBackward0_min_val_getter`, `PyComplex_FromDoubles`, `PyFloat_FromDouble` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3761-3840

```cpp
3761: 
3762: static PyObject* THPLeakyReluBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
3763:   HANDLE_TH_ERRORS
3764:   const auto& prop = static_cast<LeakyReluBackward0*>(self->cdata.get())->self_;
3765:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3766:   return obj.release().ptr();
3767:   END_HANDLE_TH_ERRORS
3768: }
3769: 
3770: static struct PyGetSetDef LeakyReluBackward0_properties[] = {
3771:   THP_FUNCTION_DEFAULT_PROPERTIES,
3772:   {(char*)"_saved_negative_slope", (getter)THPLeakyReluBackward0_negative_slope_getter, nullptr, nullptr, nullptr},
3773:   {(char*)"_saved_self", (getter)THPLeakyReluBackward0_self_getter, nullptr, nullptr, nullptr},
3774:   {(char*)"_raw_saved_self", (getter)THPLeakyReluBackward0_self_raw_getter, nullptr, nullptr, nullptr},
3775:   {nullptr} /* sentinel */
3776: };
3777: 
3778: static PyObject* THPLogSigmoidBackward0_self_getter(THPCppFunction *self, void *_unused) {
3779:   HANDLE_TH_ERRORS
3780:   const auto& prop = static_cast<LogSigmoidBackward0*>(self->cdata.get())->self_;
3781:   return THPVariable_Wrap(prop.unpack(self->cdata));
3782:   END_HANDLE_TH_ERRORS
3783: }
3784: 
3785: static PyObject* THPLogSigmoidBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
3786:   HANDLE_TH_ERRORS
3787:   const auto& prop = static_cast<LogSigmoidBackward0*>(self->cdata.get())->self_;
3788:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3789:   return obj.release().ptr();
3790:   END_HANDLE_TH_ERRORS
3791: }
3792: 
3793: static PyObject* THPLogSigmoidBackward0_buffer_getter(THPCppFunction *self, void *_unused) {
3794:   HANDLE_TH_ERRORS
3795:   const auto& prop = static_cast<LogSigmoidBackward0*>(self->cdata.get())->buffer_;
3796:   return THPVariable_Wrap(prop.unpack(self->cdata));
3797:   END_HANDLE_TH_ERRORS
3798: }
3799: 
3800: static PyObject* THPLogSigmoidBackward0_buffer_raw_getter(THPCppFunction *self, void *_unused) {
3801:   HANDLE_TH_ERRORS
3802:   const auto& prop = static_cast<LogSigmoidBackward0*>(self->cdata.get())->buffer_;
3803:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3804:   return obj.release().ptr();
3805:   END_HANDLE_TH_ERRORS
3806: }
3807: 
3808: static struct PyGetSetDef LogSigmoidBackward0_properties[] = {
3809:   THP_FUNCTION_DEFAULT_PROPERTIES,
3810:   {(char*)"_saved_self", (getter)THPLogSigmoidBackward0_self_getter, nullptr, nullptr, nullptr},
3811:   {(char*)"_raw_saved_self", (getter)THPLogSigmoidBackward0_self_raw_getter, nullptr, nullptr, nullptr},
3812:   {(char*)"_saved_buffer", (getter)THPLogSigmoidBackward0_buffer_getter, nullptr, nullptr, nullptr},
3813:   {(char*)"_raw_saved_buffer", (getter)THPLogSigmoidBackward0_buffer_raw_getter, nullptr, nullptr, nullptr},
3814:   {nullptr} /* sentinel */
3815: };
3816: 
3817: static PyObject* THPRreluWithNoiseBackward1_lower_getter(THPCppFunction *self, void *_unused) {
3818:   HANDLE_TH_ERRORS
3819:   auto prop = static_cast<RreluWithNoiseBackward1*>(self->cdata.get())->lower;
3820:   if (prop.isComplex()) {
3821:     auto cprop = prop.to<c10::complex<double>>();
3822:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
3823:   } else if (prop.isFloatingPoint()) {
3824:     return PyFloat_FromDouble(prop.to<double>());
3825:   } else if (prop.isIntegral(/*includeBool=*/false)) {
3826:     return PyLong_FromLong(prop.to<int64_t>());
3827:   } else if (prop.isBoolean()) {
3828:     if (prop.to<bool>()) {
3829:       Py_RETURN_TRUE;
3830:     } else {
3831:       Py_RETURN_FALSE;
3832:     }
3833:   } else {
3834:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
3835:     return nullptr;
3836:   }
3837:   END_HANDLE_TH_ERRORS
3838: }
3839: 
3840: static PyObject* THPRreluWithNoiseBackward1_noise_getter(THPCppFunction *self, void *_unused) {
```

- EN: The main execution path in this span is carried by `THPLeakyReluBackward0_self_raw_getter`, `cast`, `THPLogSigmoidBackward0_self_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPLeakyReluBackward0_self_raw_getter`, `cast`, `THPLogSigmoidBackward0_self_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3841-3920

```cpp
3841:   HANDLE_TH_ERRORS
3842:   const auto& prop = static_cast<RreluWithNoiseBackward1*>(self->cdata.get())->noise_;
3843:   return THPVariable_Wrap(prop.unpack(self->cdata));
3844:   END_HANDLE_TH_ERRORS
3845: }
3846: 
3847: static PyObject* THPRreluWithNoiseBackward1_noise_raw_getter(THPCppFunction *self, void *_unused) {
3848:   HANDLE_TH_ERRORS
3849:   const auto& prop = static_cast<RreluWithNoiseBackward1*>(self->cdata.get())->noise_;
3850:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3851:   return obj.release().ptr();
3852:   END_HANDLE_TH_ERRORS
3853: }
3854: 
3855: static PyObject* THPRreluWithNoiseBackward1_training_getter(THPCppFunction *self, void *_unused) {
3856:   HANDLE_TH_ERRORS
3857:   auto prop = static_cast<RreluWithNoiseBackward1*>(self->cdata.get())->training;
3858:   if (prop) {
3859:     Py_RETURN_TRUE;
3860:   } else {
3861:     Py_RETURN_FALSE;
3862:   }
3863:   END_HANDLE_TH_ERRORS
3864: }
3865: 
3866: static PyObject* THPRreluWithNoiseBackward1_upper_getter(THPCppFunction *self, void *_unused) {
3867:   HANDLE_TH_ERRORS
3868:   auto prop = static_cast<RreluWithNoiseBackward1*>(self->cdata.get())->upper;
3869:   if (prop.isComplex()) {
3870:     auto cprop = prop.to<c10::complex<double>>();
3871:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
3872:   } else if (prop.isFloatingPoint()) {
3873:     return PyFloat_FromDouble(prop.to<double>());
3874:   } else if (prop.isIntegral(/*includeBool=*/false)) {
3875:     return PyLong_FromLong(prop.to<int64_t>());
3876:   } else if (prop.isBoolean()) {
3877:     if (prop.to<bool>()) {
3878:       Py_RETURN_TRUE;
3879:     } else {
3880:       Py_RETURN_FALSE;
3881:     }
3882:   } else {
3883:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
3884:     return nullptr;
3885:   }
3886:   END_HANDLE_TH_ERRORS
3887: }
3888: 
3889: static PyObject* THPRreluWithNoiseBackward1_result_getter(THPCppFunction *self, void *_unused) {
3890:   HANDLE_TH_ERRORS
3891:   const auto& prop = static_cast<RreluWithNoiseBackward1*>(self->cdata.get())->result_;
3892:   return THPVariable_Wrap(prop.unpack(self->cdata));
3893:   END_HANDLE_TH_ERRORS
3894: }
3895: 
3896: static PyObject* THPRreluWithNoiseBackward1_result_raw_getter(THPCppFunction *self, void *_unused) {
3897:   HANDLE_TH_ERRORS
3898:   const auto& prop = static_cast<RreluWithNoiseBackward1*>(self->cdata.get())->result_;
3899:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3900:   return obj.release().ptr();
3901:   END_HANDLE_TH_ERRORS
3902: }
3903: 
3904: static struct PyGetSetDef RreluWithNoiseBackward1_properties[] = {
3905:   THP_FUNCTION_DEFAULT_PROPERTIES,
3906:   {(char*)"_saved_lower", (getter)THPRreluWithNoiseBackward1_lower_getter, nullptr, nullptr, nullptr},
3907:   {(char*)"_saved_noise", (getter)THPRreluWithNoiseBackward1_noise_getter, nullptr, nullptr, nullptr},
3908:   {(char*)"_raw_saved_noise", (getter)THPRreluWithNoiseBackward1_noise_raw_getter, nullptr, nullptr, nullptr},
3909:   {(char*)"_saved_training", (getter)THPRreluWithNoiseBackward1_training_getter, nullptr, nullptr, nullptr},
3910:   {(char*)"_saved_upper", (getter)THPRreluWithNoiseBackward1_upper_getter, nullptr, nullptr, nullptr},
3911:   {(char*)"_saved_result", (getter)THPRreluWithNoiseBackward1_result_getter, nullptr, nullptr, nullptr},
3912:   {(char*)"_raw_saved_result", (getter)THPRreluWithNoiseBackward1_result_raw_getter, nullptr, nullptr, nullptr},
3913:   {nullptr} /* sentinel */
3914: };
3915: 
3916: static PyObject* THPSoftshrinkBackward0_lambd_getter(THPCppFunction *self, void *_unused) {
3917:   HANDLE_TH_ERRORS
3918:   auto prop = static_cast<SoftshrinkBackward0*>(self->cdata.get())->lambd;
3919:   if (prop.isComplex()) {
3920:     auto cprop = prop.to<c10::complex<double>>();
```

- EN: The main execution path in this span is carried by `THPVariable_Wrap`, `THPRreluWithNoiseBackward1_noise_raw_getter`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_Wrap`, `THPRreluWithNoiseBackward1_noise_raw_getter`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3921-4000

```cpp
3921:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
3922:   } else if (prop.isFloatingPoint()) {
3923:     return PyFloat_FromDouble(prop.to<double>());
3924:   } else if (prop.isIntegral(/*includeBool=*/false)) {
3925:     return PyLong_FromLong(prop.to<int64_t>());
3926:   } else if (prop.isBoolean()) {
3927:     if (prop.to<bool>()) {
3928:       Py_RETURN_TRUE;
3929:     } else {
3930:       Py_RETURN_FALSE;
3931:     }
3932:   } else {
3933:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
3934:     return nullptr;
3935:   }
3936:   END_HANDLE_TH_ERRORS
3937: }
3938: 
3939: static PyObject* THPSoftshrinkBackward0_self_getter(THPCppFunction *self, void *_unused) {
3940:   HANDLE_TH_ERRORS
3941:   const auto& prop = static_cast<SoftshrinkBackward0*>(self->cdata.get())->self_;
3942:   return THPVariable_Wrap(prop.unpack(self->cdata));
3943:   END_HANDLE_TH_ERRORS
3944: }
3945: 
3946: static PyObject* THPSoftshrinkBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
3947:   HANDLE_TH_ERRORS
3948:   const auto& prop = static_cast<SoftshrinkBackward0*>(self->cdata.get())->self_;
3949:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3950:   return obj.release().ptr();
3951:   END_HANDLE_TH_ERRORS
3952: }
3953: 
3954: static struct PyGetSetDef SoftshrinkBackward0_properties[] = {
3955:   THP_FUNCTION_DEFAULT_PROPERTIES,
3956:   {(char*)"_saved_lambd", (getter)THPSoftshrinkBackward0_lambd_getter, nullptr, nullptr, nullptr},
3957:   {(char*)"_saved_self", (getter)THPSoftshrinkBackward0_self_getter, nullptr, nullptr, nullptr},
3958:   {(char*)"_raw_saved_self", (getter)THPSoftshrinkBackward0_self_raw_getter, nullptr, nullptr, nullptr},
3959:   {nullptr} /* sentinel */
3960: };
3961: 
3962: static PyObject* THPReflectionPad3DBackward0_padding_getter(THPCppFunction *self, void *_unused) {
3963:   HANDLE_TH_ERRORS
3964:   auto prop = static_cast<ReflectionPad3DBackward0*>(self->cdata.get())->padding;
3965:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3966:   for (auto i : c10::irange(prop.size())) {
3967:       auto si = prop[i];
3968:       if (auto m = si.maybe_as_int()) {
3969:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3970:       } else {
3971:         auto py_symint = py::cast(si).release().ptr();
3972:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3973:       }
3974:   }
3975:   return tup;
3976:   END_HANDLE_TH_ERRORS
3977: }
3978: 
3979: static PyObject* THPReflectionPad3DBackward0_self_getter(THPCppFunction *self, void *_unused) {
3980:   HANDLE_TH_ERRORS
3981:   const auto& prop = static_cast<ReflectionPad3DBackward0*>(self->cdata.get())->self_;
3982:   return THPVariable_Wrap(prop.unpack(self->cdata));
3983:   END_HANDLE_TH_ERRORS
3984: }
3985: 
3986: static PyObject* THPReflectionPad3DBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
3987:   HANDLE_TH_ERRORS
3988:   const auto& prop = static_cast<ReflectionPad3DBackward0*>(self->cdata.get())->self_;
3989:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3990:   return obj.release().ptr();
3991:   END_HANDLE_TH_ERRORS
3992: }
3993: 
3994: static struct PyGetSetDef ReflectionPad3DBackward0_properties[] = {
3995:   THP_FUNCTION_DEFAULT_PROPERTIES,
3996:   {(char*)"_saved_padding", (getter)THPReflectionPad3DBackward0_padding_getter, nullptr, nullptr, nullptr},
3997:   {(char*)"_saved_self", (getter)THPReflectionPad3DBackward0_self_getter, nullptr, nullptr, nullptr},
3998:   {(char*)"_raw_saved_self", (getter)THPReflectionPad3DBackward0_self_raw_getter, nullptr, nullptr, nullptr},
3999:   {nullptr} /* sentinel */
4000: };
```

- EN: The main execution path in this span is carried by `PyComplex_FromDoubles`, `PyFloat_FromDouble`, `PyLong_FromLong`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyComplex_FromDoubles`, `PyFloat_FromDouble`, `PyLong_FromLong` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4001-4080

```cpp
4001: 
4002: static PyObject* THPReplicationPad1DBackward0_padding_getter(THPCppFunction *self, void *_unused) {
4003:   HANDLE_TH_ERRORS
4004:   auto prop = static_cast<ReplicationPad1DBackward0*>(self->cdata.get())->padding;
4005:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4006:   for (auto i : c10::irange(prop.size())) {
4007:       auto si = prop[i];
4008:       if (auto m = si.maybe_as_int()) {
4009:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4010:       } else {
4011:         auto py_symint = py::cast(si).release().ptr();
4012:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4013:       }
4014:   }
4015:   return tup;
4016:   END_HANDLE_TH_ERRORS
4017: }
4018: 
4019: static PyObject* THPReplicationPad1DBackward0_self_getter(THPCppFunction *self, void *_unused) {
4020:   HANDLE_TH_ERRORS
4021:   const auto& prop = static_cast<ReplicationPad1DBackward0*>(self->cdata.get())->self_;
4022:   return THPVariable_Wrap(prop.unpack(self->cdata));
4023:   END_HANDLE_TH_ERRORS
4024: }
4025: 
4026: static PyObject* THPReplicationPad1DBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
4027:   HANDLE_TH_ERRORS
4028:   const auto& prop = static_cast<ReplicationPad1DBackward0*>(self->cdata.get())->self_;
4029:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4030:   return obj.release().ptr();
4031:   END_HANDLE_TH_ERRORS
4032: }
4033: 
4034: static struct PyGetSetDef ReplicationPad1DBackward0_properties[] = {
4035:   THP_FUNCTION_DEFAULT_PROPERTIES,
4036:   {(char*)"_saved_padding", (getter)THPReplicationPad1DBackward0_padding_getter, nullptr, nullptr, nullptr},
4037:   {(char*)"_saved_self", (getter)THPReplicationPad1DBackward0_self_getter, nullptr, nullptr, nullptr},
4038:   {(char*)"_raw_saved_self", (getter)THPReplicationPad1DBackward0_self_raw_getter, nullptr, nullptr, nullptr},
4039:   {nullptr} /* sentinel */
4040: };
4041: 
4042: static PyObject* THPReplicationPad2DBackward0_padding_getter(THPCppFunction *self, void *_unused) {
4043:   HANDLE_TH_ERRORS
4044:   auto prop = static_cast<ReplicationPad2DBackward0*>(self->cdata.get())->padding;
4045:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4046:   for (auto i : c10::irange(prop.size())) {
4047:       auto si = prop[i];
4048:       if (auto m = si.maybe_as_int()) {
4049:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4050:       } else {
4051:         auto py_symint = py::cast(si).release().ptr();
4052:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4053:       }
4054:   }
4055:   return tup;
4056:   END_HANDLE_TH_ERRORS
4057: }
4058: 
4059: static PyObject* THPReplicationPad2DBackward0_self_getter(THPCppFunction *self, void *_unused) {
4060:   HANDLE_TH_ERRORS
4061:   const auto& prop = static_cast<ReplicationPad2DBackward0*>(self->cdata.get())->self_;
4062:   return THPVariable_Wrap(prop.unpack(self->cdata));
4063:   END_HANDLE_TH_ERRORS
4064: }
4065: 
4066: static PyObject* THPReplicationPad2DBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
4067:   HANDLE_TH_ERRORS
4068:   const auto& prop = static_cast<ReplicationPad2DBackward0*>(self->cdata.get())->self_;
4069:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4070:   return obj.release().ptr();
4071:   END_HANDLE_TH_ERRORS
4072: }
4073: 
4074: static struct PyGetSetDef ReplicationPad2DBackward0_properties[] = {
4075:   THP_FUNCTION_DEFAULT_PROPERTIES,
4076:   {(char*)"_saved_padding", (getter)THPReplicationPad2DBackward0_padding_getter, nullptr, nullptr, nullptr},
4077:   {(char*)"_saved_self", (getter)THPReplicationPad2DBackward0_self_getter, nullptr, nullptr, nullptr},
4078:   {(char*)"_raw_saved_self", (getter)THPReplicationPad2DBackward0_self_raw_getter, nullptr, nullptr, nullptr},
4079:   {nullptr} /* sentinel */
4080: };
```

- EN: The main execution path in this span is carried by `THPReplicationPad1DBackward0_padding_getter`, `PyTuple_New`, `PyTuple_SetItem`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPReplicationPad1DBackward0_padding_getter`, `PyTuple_New`, `PyTuple_SetItem` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4081-4160

```cpp
4081: 
4082: static PyObject* THPUpsampleNearest1DBackward0_output_size_getter(THPCppFunction *self, void *_unused) {
4083:   HANDLE_TH_ERRORS
4084:   auto prop = static_cast<UpsampleNearest1DBackward0*>(self->cdata.get())->output_size;
4085:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4086:   for (auto i : c10::irange(prop.size())) {
4087:       auto si = prop[i];
4088:       if (auto m = si.maybe_as_int()) {
4089:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4090:       } else {
4091:         auto py_symint = py::cast(si).release().ptr();
4092:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4093:       }
4094:   }
4095:   return tup;
4096:   END_HANDLE_TH_ERRORS
4097: }
4098: 
4099: static PyObject* THPUpsampleNearest1DBackward0_scales_getter(THPCppFunction *self, void *_unused) {
4100:   HANDLE_TH_ERRORS
4101:   auto opt_prop = static_cast<UpsampleNearest1DBackward0*>(self->cdata.get())->scales;
4102:   if (!opt_prop.has_value()) {
4103:     Py_RETURN_NONE;
4104:   }
4105:   auto prop = opt_prop.value();
4106:   return PyFloat_FromDouble((double) prop);
4107:   END_HANDLE_TH_ERRORS
4108: }
4109: 
4110: static PyObject* THPUpsampleNearest1DBackward0_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
4111:   HANDLE_TH_ERRORS
4112:   auto prop = static_cast<UpsampleNearest1DBackward0*>(self->cdata.get())->self_sym_sizes;
4113:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4114:   for (auto i : c10::irange(prop.size())) {
4115:       auto si = prop[i];
4116:       if (auto m = si.maybe_as_int()) {
4117:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4118:       } else {
4119:         auto py_symint = py::cast(si).release().ptr();
4120:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4121:       }
4122:   }
4123:   return tup;
4124:   END_HANDLE_TH_ERRORS
4125: }
4126: 
4127: static struct PyGetSetDef UpsampleNearest1DBackward0_properties[] = {
4128:   THP_FUNCTION_DEFAULT_PROPERTIES,
4129:   {(char*)"_saved_output_size", (getter)THPUpsampleNearest1DBackward0_output_size_getter, nullptr, nullptr, nullptr},
4130:   {(char*)"_saved_scales", (getter)THPUpsampleNearest1DBackward0_scales_getter, nullptr, nullptr, nullptr},
4131:   {(char*)"_saved_self_sym_sizes", (getter)THPUpsampleNearest1DBackward0_self_sym_sizes_getter, nullptr, nullptr, nullptr},
4132:   {nullptr} /* sentinel */
4133: };
4134: 
4135: static PyObject* THPUpsampleNearestExact1DBackward0_output_size_getter(THPCppFunction *self, void *_unused) {
4136:   HANDLE_TH_ERRORS
4137:   auto prop = static_cast<UpsampleNearestExact1DBackward0*>(self->cdata.get())->output_size;
4138:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4139:   for (auto i : c10::irange(prop.size())) {
4140:       auto si = prop[i];
4141:       if (auto m = si.maybe_as_int()) {
4142:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4143:       } else {
4144:         auto py_symint = py::cast(si).release().ptr();
4145:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4146:       }
4147:   }
4148:   return tup;
4149:   END_HANDLE_TH_ERRORS
4150: }
4151: 
4152: static PyObject* THPUpsampleNearestExact1DBackward0_scales_getter(THPCppFunction *self, void *_unused) {
4153:   HANDLE_TH_ERRORS
4154:   auto opt_prop = static_cast<UpsampleNearestExact1DBackward0*>(self->cdata.get())->scales;
4155:   if (!opt_prop.has_value()) {
4156:     Py_RETURN_NONE;
4157:   }
4158:   auto prop = opt_prop.value();
4159:   return PyFloat_FromDouble((double) prop);
4160:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPUpsampleNearest1DBackward0_output_size_getter`, `PyTuple_New`, `PyTuple_SetItem`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPUpsampleNearest1DBackward0_output_size_getter`, `PyTuple_New`, `PyTuple_SetItem` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4161-4240

```cpp
4161: }
4162: 
4163: static PyObject* THPUpsampleNearestExact1DBackward0_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
4164:   HANDLE_TH_ERRORS
4165:   auto prop = static_cast<UpsampleNearestExact1DBackward0*>(self->cdata.get())->self_sym_sizes;
4166:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4167:   for (auto i : c10::irange(prop.size())) {
4168:       auto si = prop[i];
4169:       if (auto m = si.maybe_as_int()) {
4170:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4171:       } else {
4172:         auto py_symint = py::cast(si).release().ptr();
4173:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4174:       }
4175:   }
4176:   return tup;
4177:   END_HANDLE_TH_ERRORS
4178: }
4179: 
4180: static struct PyGetSetDef UpsampleNearestExact1DBackward0_properties[] = {
4181:   THP_FUNCTION_DEFAULT_PROPERTIES,
4182:   {(char*)"_saved_output_size", (getter)THPUpsampleNearestExact1DBackward0_output_size_getter, nullptr, nullptr, nullptr},
4183:   {(char*)"_saved_scales", (getter)THPUpsampleNearestExact1DBackward0_scales_getter, nullptr, nullptr, nullptr},
4184:   {(char*)"_saved_self_sym_sizes", (getter)THPUpsampleNearestExact1DBackward0_self_sym_sizes_getter, nullptr, nullptr, nullptr},
4185:   {nullptr} /* sentinel */
4186: };
4187: 
4188: static PyObject* THPFractionalMaxPool3DBackward0_kernel_size_getter(THPCppFunction *self, void *_unused) {
4189:   HANDLE_TH_ERRORS
4190:   auto prop = static_cast<FractionalMaxPool3DBackward0*>(self->cdata.get())->kernel_size;
4191:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4192:   for (auto i : c10::irange(prop.size())) {
4193:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
4194:   }
4195:   return tup;
4196:   END_HANDLE_TH_ERRORS
4197: }
4198: 
4199: static PyObject* THPFractionalMaxPool3DBackward0_output_size_getter(THPCppFunction *self, void *_unused) {
4200:   HANDLE_TH_ERRORS
4201:   auto prop = static_cast<FractionalMaxPool3DBackward0*>(self->cdata.get())->output_size;
4202:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4203:   for (auto i : c10::irange(prop.size())) {
4204:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
4205:   }
4206:   return tup;
4207:   END_HANDLE_TH_ERRORS
4208: }
4209: 
4210: static PyObject* THPFractionalMaxPool3DBackward0_self_getter(THPCppFunction *self, void *_unused) {
4211:   HANDLE_TH_ERRORS
4212:   const auto& prop = static_cast<FractionalMaxPool3DBackward0*>(self->cdata.get())->self_;
4213:   return THPVariable_Wrap(prop.unpack(self->cdata));
4214:   END_HANDLE_TH_ERRORS
4215: }
4216: 
4217: static PyObject* THPFractionalMaxPool3DBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
4218:   HANDLE_TH_ERRORS
4219:   const auto& prop = static_cast<FractionalMaxPool3DBackward0*>(self->cdata.get())->self_;
4220:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4221:   return obj.release().ptr();
4222:   END_HANDLE_TH_ERRORS
4223: }
4224: 
4225: static PyObject* THPFractionalMaxPool3DBackward0_result1_getter(THPCppFunction *self, void *_unused) {
4226:   HANDLE_TH_ERRORS
4227:   const auto& prop = static_cast<FractionalMaxPool3DBackward0*>(self->cdata.get())->result1_;
4228:   return THPVariable_Wrap(prop.unpack(self->cdata));
4229:   END_HANDLE_TH_ERRORS
4230: }
4231: 
4232: static PyObject* THPFractionalMaxPool3DBackward0_result1_raw_getter(THPCppFunction *self, void *_unused) {
4233:   HANDLE_TH_ERRORS
4234:   const auto& prop = static_cast<FractionalMaxPool3DBackward0*>(self->cdata.get())->result1_;
4235:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4236:   return obj.release().ptr();
4237:   END_HANDLE_TH_ERRORS
4238: }
4239: 
4240: static struct PyGetSetDef FractionalMaxPool3DBackward0_properties[] = {
```

- EN: The main execution path in this span is carried by `THPUpsampleNearestExact1DBackward0_self_sym_sizes_getter`, `PyTuple_New`, `PyTuple_SetItem`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPUpsampleNearestExact1DBackward0_self_sym_sizes_getter`, `PyTuple_New`, `PyTuple_SetItem` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 4241-4320

```cpp
4241:   THP_FUNCTION_DEFAULT_PROPERTIES,
4242:   {(char*)"_saved_kernel_size", (getter)THPFractionalMaxPool3DBackward0_kernel_size_getter, nullptr, nullptr, nullptr},
4243:   {(char*)"_saved_output_size", (getter)THPFractionalMaxPool3DBackward0_output_size_getter, nullptr, nullptr, nullptr},
4244:   {(char*)"_saved_self", (getter)THPFractionalMaxPool3DBackward0_self_getter, nullptr, nullptr, nullptr},
4245:   {(char*)"_raw_saved_self", (getter)THPFractionalMaxPool3DBackward0_self_raw_getter, nullptr, nullptr, nullptr},
4246:   {(char*)"_saved_result1", (getter)THPFractionalMaxPool3DBackward0_result1_getter, nullptr, nullptr, nullptr},
4247:   {(char*)"_raw_saved_result1", (getter)THPFractionalMaxPool3DBackward0_result1_raw_getter, nullptr, nullptr, nullptr},
4248:   {nullptr} /* sentinel */
4249: };
4250: 
4251: static PyObject* THPMpsConvolutionBackward0_dilation_getter(THPCppFunction *self, void *_unused) {
4252:   HANDLE_TH_ERRORS
4253:   auto prop = static_cast<MpsConvolutionBackward0*>(self->cdata.get())->dilation;
4254:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4255:   for (auto i : c10::irange(prop.size())) {
4256:       auto si = prop[i];
4257:       if (auto m = si.maybe_as_int()) {
4258:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4259:       } else {
4260:         auto py_symint = py::cast(si).release().ptr();
4261:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4262:       }
4263:   }
4264:   return tup;
4265:   END_HANDLE_TH_ERRORS
4266: }
4267: 
4268: static PyObject* THPMpsConvolutionBackward0_groups_getter(THPCppFunction *self, void *_unused) {
4269:   HANDLE_TH_ERRORS
4270:   auto prop = static_cast<MpsConvolutionBackward0*>(self->cdata.get())->groups;
4271:   if (auto m = prop.maybe_as_int()) {
4272:     return PyLong_FromUnsignedLong(*m);
4273:   } else {
4274:     return py::cast(prop).release().ptr();
4275:   }
4276:   END_HANDLE_TH_ERRORS
4277: }
4278: 
4279: static PyObject* THPMpsConvolutionBackward0_padding_getter(THPCppFunction *self, void *_unused) {
4280:   HANDLE_TH_ERRORS
4281:   auto prop = static_cast<MpsConvolutionBackward0*>(self->cdata.get())->padding;
4282:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4283:   for (auto i : c10::irange(prop.size())) {
4284:       auto si = prop[i];
4285:       if (auto m = si.maybe_as_int()) {
4286:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4287:       } else {
4288:         auto py_symint = py::cast(si).release().ptr();
4289:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4290:       }
4291:   }
4292:   return tup;
4293:   END_HANDLE_TH_ERRORS
4294: }
4295: 
4296: static PyObject* THPMpsConvolutionBackward0_self_getter(THPCppFunction *self, void *_unused) {
4297:   HANDLE_TH_ERRORS
4298:   const auto& prop = static_cast<MpsConvolutionBackward0*>(self->cdata.get())->self_;
4299:   return THPVariable_Wrap(prop.unpack(self->cdata));
4300:   END_HANDLE_TH_ERRORS
4301: }
4302: 
4303: static PyObject* THPMpsConvolutionBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
4304:   HANDLE_TH_ERRORS
4305:   const auto& prop = static_cast<MpsConvolutionBackward0*>(self->cdata.get())->self_;
4306:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4307:   return obj.release().ptr();
4308:   END_HANDLE_TH_ERRORS
4309: }
4310: 
4311: static PyObject* THPMpsConvolutionBackward0_stride_getter(THPCppFunction *self, void *_unused) {
4312:   HANDLE_TH_ERRORS
4313:   auto prop = static_cast<MpsConvolutionBackward0*>(self->cdata.get())->stride;
4314:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4315:   for (auto i : c10::irange(prop.size())) {
4316:       auto si = prop[i];
4317:       if (auto m = si.maybe_as_int()) {
4318:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4319:       } else {
4320:         auto py_symint = py::cast(si).release().ptr();
```

- EN: The main execution path in this span is carried by `THPMpsConvolutionBackward0_dilation_getter`, `PyTuple_New`, `PyTuple_SetItem`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPMpsConvolutionBackward0_dilation_getter`, `PyTuple_New`, `PyTuple_SetItem` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 4321-4400

```cpp
4321:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4322:       }
4323:   }
4324:   return tup;
4325:   END_HANDLE_TH_ERRORS
4326: }
4327: 
4328: static PyObject* THPMpsConvolutionBackward0_weight_getter(THPCppFunction *self, void *_unused) {
4329:   HANDLE_TH_ERRORS
4330:   const auto& prop = static_cast<MpsConvolutionBackward0*>(self->cdata.get())->weight_;
4331:   return THPVariable_Wrap(prop.unpack(self->cdata));
4332:   END_HANDLE_TH_ERRORS
4333: }
4334: 
4335: static PyObject* THPMpsConvolutionBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
4336:   HANDLE_TH_ERRORS
4337:   const auto& prop = static_cast<MpsConvolutionBackward0*>(self->cdata.get())->weight_;
4338:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4339:   return obj.release().ptr();
4340:   END_HANDLE_TH_ERRORS
4341: }
4342: 
4343: static struct PyGetSetDef MpsConvolutionBackward0_properties[] = {
4344:   THP_FUNCTION_DEFAULT_PROPERTIES,
4345:   {(char*)"_saved_dilation", (getter)THPMpsConvolutionBackward0_dilation_getter, nullptr, nullptr, nullptr},
4346:   {(char*)"_saved_groups", (getter)THPMpsConvolutionBackward0_groups_getter, nullptr, nullptr, nullptr},
4347:   {(char*)"_saved_padding", (getter)THPMpsConvolutionBackward0_padding_getter, nullptr, nullptr, nullptr},
4348:   {(char*)"_saved_self", (getter)THPMpsConvolutionBackward0_self_getter, nullptr, nullptr, nullptr},
4349:   {(char*)"_raw_saved_self", (getter)THPMpsConvolutionBackward0_self_raw_getter, nullptr, nullptr, nullptr},
4350:   {(char*)"_saved_stride", (getter)THPMpsConvolutionBackward0_stride_getter, nullptr, nullptr, nullptr},
4351:   {(char*)"_saved_weight", (getter)THPMpsConvolutionBackward0_weight_getter, nullptr, nullptr, nullptr},
4352:   {(char*)"_raw_saved_weight", (getter)THPMpsConvolutionBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
4353:   {nullptr} /* sentinel */
4354: };
4355: 
4356: static PyObject* THPConvDepthwise2DBackward0_bias_sym_sizes_opt_getter(THPCppFunction *self, void *_unused) {
4357:   HANDLE_TH_ERRORS
4358:   auto opt_prop = static_cast<ConvDepthwise2DBackward0*>(self->cdata.get())->bias_sym_sizes_opt;
4359:   if (!opt_prop.list.has_value()) {
4360:     Py_RETURN_NONE;
4361:   }
4362:   auto prop = opt_prop.list.value();
4363:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4364:   for (auto i : c10::irange(prop.size())) {
4365:       auto si = prop[i];
4366:       if (auto m = si.maybe_as_int()) {
4367:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4368:       } else {
4369:         auto py_symint = py::cast(si).release().ptr();
4370:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4371:       }
4372:   }
4373:   return tup;
4374:   END_HANDLE_TH_ERRORS
4375: }
4376: 
4377: static PyObject* THPConvDepthwise2DBackward0_dilation_getter(THPCppFunction *self, void *_unused) {
4378:   HANDLE_TH_ERRORS
4379:   auto prop = static_cast<ConvDepthwise2DBackward0*>(self->cdata.get())->dilation;
4380:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4381:   for (auto i : c10::irange(prop.size())) {
4382:       auto si = prop[i];
4383:       if (auto m = si.maybe_as_int()) {
4384:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4385:       } else {
4386:         auto py_symint = py::cast(si).release().ptr();
4387:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4388:       }
4389:   }
4390:   return tup;
4391:   END_HANDLE_TH_ERRORS
4392: }
4393: 
4394: static PyObject* THPConvDepthwise2DBackward0_padding_getter(THPCppFunction *self, void *_unused) {
4395:   HANDLE_TH_ERRORS
4396:   auto prop = static_cast<ConvDepthwise2DBackward0*>(self->cdata.get())->padding;
4397:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4398:   for (auto i : c10::irange(prop.size())) {
4399:       auto si = prop[i];
4400:       if (auto m = si.maybe_as_int()) {
```

- EN: The main execution path in this span is carried by `PyTuple_SetItem`, `THPMpsConvolutionBackward0_weight_getter`, `THPVariable_Wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyTuple_SetItem`, `THPMpsConvolutionBackward0_weight_getter`, `THPVariable_Wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4401-4480

```cpp
4401:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4402:       } else {
4403:         auto py_symint = py::cast(si).release().ptr();
4404:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4405:       }
4406:   }
4407:   return tup;
4408:   END_HANDLE_TH_ERRORS
4409: }
4410: 
4411: static PyObject* THPConvDepthwise2DBackward0_self_getter(THPCppFunction *self, void *_unused) {
4412:   HANDLE_TH_ERRORS
4413:   const auto& prop = static_cast<ConvDepthwise2DBackward0*>(self->cdata.get())->self_;
4414:   return THPVariable_Wrap(prop.unpack(self->cdata));
4415:   END_HANDLE_TH_ERRORS
4416: }
4417: 
4418: static PyObject* THPConvDepthwise2DBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
4419:   HANDLE_TH_ERRORS
4420:   const auto& prop = static_cast<ConvDepthwise2DBackward0*>(self->cdata.get())->self_;
4421:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4422:   return obj.release().ptr();
4423:   END_HANDLE_TH_ERRORS
4424: }
4425: 
4426: static PyObject* THPConvDepthwise2DBackward0_stride_getter(THPCppFunction *self, void *_unused) {
4427:   HANDLE_TH_ERRORS
4428:   auto prop = static_cast<ConvDepthwise2DBackward0*>(self->cdata.get())->stride;
4429:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4430:   for (auto i : c10::irange(prop.size())) {
4431:       auto si = prop[i];
4432:       if (auto m = si.maybe_as_int()) {
4433:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4434:       } else {
4435:         auto py_symint = py::cast(si).release().ptr();
4436:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4437:       }
4438:   }
4439:   return tup;
4440:   END_HANDLE_TH_ERRORS
4441: }
4442: 
4443: static PyObject* THPConvDepthwise2DBackward0_weight_getter(THPCppFunction *self, void *_unused) {
4444:   HANDLE_TH_ERRORS
4445:   const auto& prop = static_cast<ConvDepthwise2DBackward0*>(self->cdata.get())->weight_;
4446:   return THPVariable_Wrap(prop.unpack(self->cdata));
4447:   END_HANDLE_TH_ERRORS
4448: }
4449: 
4450: static PyObject* THPConvDepthwise2DBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
4451:   HANDLE_TH_ERRORS
4452:   const auto& prop = static_cast<ConvDepthwise2DBackward0*>(self->cdata.get())->weight_;
4453:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4454:   return obj.release().ptr();
4455:   END_HANDLE_TH_ERRORS
4456: }
4457: 
4458: static struct PyGetSetDef ConvDepthwise2DBackward0_properties[] = {
4459:   THP_FUNCTION_DEFAULT_PROPERTIES,
4460:   {(char*)"_saved_bias_sym_sizes_opt", (getter)THPConvDepthwise2DBackward0_bias_sym_sizes_opt_getter, nullptr, nullptr, nullptr},
4461:   {(char*)"_saved_dilation", (getter)THPConvDepthwise2DBackward0_dilation_getter, nullptr, nullptr, nullptr},
4462:   {(char*)"_saved_padding", (getter)THPConvDepthwise2DBackward0_padding_getter, nullptr, nullptr, nullptr},
4463:   {(char*)"_saved_self", (getter)THPConvDepthwise2DBackward0_self_getter, nullptr, nullptr, nullptr},
4464:   {(char*)"_raw_saved_self", (getter)THPConvDepthwise2DBackward0_self_raw_getter, nullptr, nullptr, nullptr},
4465:   {(char*)"_saved_stride", (getter)THPConvDepthwise2DBackward0_stride_getter, nullptr, nullptr, nullptr},
4466:   {(char*)"_saved_weight", (getter)THPConvDepthwise2DBackward0_weight_getter, nullptr, nullptr, nullptr},
4467:   {(char*)"_raw_saved_weight", (getter)THPConvDepthwise2DBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
4468:   {nullptr} /* sentinel */
4469: };
4470: 
4471: static PyObject* THPAdaptiveAvgPool3DBackwardBackward0_grad_output_sym_argsize_minus_1_getter(THPCppFunction *self, void *_unused) {
4472:   HANDLE_TH_ERRORS
4473:   auto prop = static_cast<AdaptiveAvgPool3DBackwardBackward0*>(self->cdata.get())->grad_output_sym_argsize_minus_1;
4474:   if (auto m = prop.maybe_as_int()) {
4475:     return PyLong_FromUnsignedLong(*m);
4476:   } else {
4477:     return py::cast(prop).release().ptr();
4478:   }
4479:   END_HANDLE_TH_ERRORS
4480: }
```

- EN: The main execution path in this span is carried by `PyTuple_SetItem`, `cast`, `THPConvDepthwise2DBackward0_self_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyTuple_SetItem`, `cast`, `THPConvDepthwise2DBackward0_self_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4481-4560

```cpp
4481: 
4482: static PyObject* THPAdaptiveAvgPool3DBackwardBackward0_grad_output_sym_argsize_minus_2_getter(THPCppFunction *self, void *_unused) {
4483:   HANDLE_TH_ERRORS
4484:   auto prop = static_cast<AdaptiveAvgPool3DBackwardBackward0*>(self->cdata.get())->grad_output_sym_argsize_minus_2;
4485:   if (auto m = prop.maybe_as_int()) {
4486:     return PyLong_FromUnsignedLong(*m);
4487:   } else {
4488:     return py::cast(prop).release().ptr();
4489:   }
4490:   END_HANDLE_TH_ERRORS
4491: }
4492: 
4493: static PyObject* THPAdaptiveAvgPool3DBackwardBackward0_grad_output_sym_argsize_minus_3_getter(THPCppFunction *self, void *_unused) {
4494:   HANDLE_TH_ERRORS
4495:   auto prop = static_cast<AdaptiveAvgPool3DBackwardBackward0*>(self->cdata.get())->grad_output_sym_argsize_minus_3;
4496:   if (auto m = prop.maybe_as_int()) {
4497:     return PyLong_FromUnsignedLong(*m);
4498:   } else {
4499:     return py::cast(prop).release().ptr();
4500:   }
4501:   END_HANDLE_TH_ERRORS
4502: }
4503: 
4504: static struct PyGetSetDef AdaptiveAvgPool3DBackwardBackward0_properties[] = {
4505:   THP_FUNCTION_DEFAULT_PROPERTIES,
4506:   {(char*)"_saved_grad_output_sym_argsize_minus_1", (getter)THPAdaptiveAvgPool3DBackwardBackward0_grad_output_sym_argsize_minus_1_getter, nullptr, nullptr, nullptr},
4507:   {(char*)"_saved_grad_output_sym_argsize_minus_2", (getter)THPAdaptiveAvgPool3DBackwardBackward0_grad_output_sym_argsize_minus_2_getter, nullptr, nullptr, nullptr},
4508:   {(char*)"_saved_grad_output_sym_argsize_minus_3", (getter)THPAdaptiveAvgPool3DBackwardBackward0_grad_output_sym_argsize_minus_3_getter, nullptr, nullptr, nullptr},
4509:   {nullptr} /* sentinel */
4510: };
4511: 
4512: static PyObject* THPAdaptiveMaxPool2DBackwardBackward0_indices_getter(THPCppFunction *self, void *_unused) {
4513:   HANDLE_TH_ERRORS
4514:   const auto& prop = static_cast<AdaptiveMaxPool2DBackwardBackward0*>(self->cdata.get())->indices_;
4515:   return THPVariable_Wrap(prop.unpack(self->cdata));
4516:   END_HANDLE_TH_ERRORS
4517: }
4518: 
4519: static PyObject* THPAdaptiveMaxPool2DBackwardBackward0_indices_raw_getter(THPCppFunction *self, void *_unused) {
4520:   HANDLE_TH_ERRORS
4521:   const auto& prop = static_cast<AdaptiveMaxPool2DBackwardBackward0*>(self->cdata.get())->indices_;
4522:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4523:   return obj.release().ptr();
4524:   END_HANDLE_TH_ERRORS
4525: }
4526: 
4527: static struct PyGetSetDef AdaptiveMaxPool2DBackwardBackward0_properties[] = {
4528:   THP_FUNCTION_DEFAULT_PROPERTIES,
4529:   {(char*)"_saved_indices", (getter)THPAdaptiveMaxPool2DBackwardBackward0_indices_getter, nullptr, nullptr, nullptr},
4530:   {(char*)"_raw_saved_indices", (getter)THPAdaptiveMaxPool2DBackwardBackward0_indices_raw_getter, nullptr, nullptr, nullptr},
4531:   {nullptr} /* sentinel */
4532: };
4533: 
4534: static PyObject* THPAdaptiveMaxPool3DBackwardBackward0_indices_getter(THPCppFunction *self, void *_unused) {
4535:   HANDLE_TH_ERRORS
4536:   const auto& prop = static_cast<AdaptiveMaxPool3DBackwardBackward0*>(self->cdata.get())->indices_;
4537:   return THPVariable_Wrap(prop.unpack(self->cdata));
4538:   END_HANDLE_TH_ERRORS
4539: }
4540: 
4541: static PyObject* THPAdaptiveMaxPool3DBackwardBackward0_indices_raw_getter(THPCppFunction *self, void *_unused) {
4542:   HANDLE_TH_ERRORS
4543:   const auto& prop = static_cast<AdaptiveMaxPool3DBackwardBackward0*>(self->cdata.get())->indices_;
4544:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4545:   return obj.release().ptr();
4546:   END_HANDLE_TH_ERRORS
4547: }
4548: 
4549: static struct PyGetSetDef AdaptiveMaxPool3DBackwardBackward0_properties[] = {
4550:   THP_FUNCTION_DEFAULT_PROPERTIES,
4551:   {(char*)"_saved_indices", (getter)THPAdaptiveMaxPool3DBackwardBackward0_indices_getter, nullptr, nullptr, nullptr},
4552:   {(char*)"_raw_saved_indices", (getter)THPAdaptiveMaxPool3DBackwardBackward0_indices_raw_getter, nullptr, nullptr, nullptr},
4553:   {nullptr} /* sentinel */
4554: };
4555: 
4556: static PyObject* THPLogSigmoidBackwardBackward0_buffer_getter(THPCppFunction *self, void *_unused) {
4557:   HANDLE_TH_ERRORS
4558:   const auto& prop = static_cast<LogSigmoidBackwardBackward0*>(self->cdata.get())->buffer_;
4559:   return THPVariable_Wrap(prop.unpack(self->cdata));
4560:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPAdaptiveAvgPool3DBackwardBackward0_grad_output_sym_argsize_minus_2_getter`, `PyLong_FromUnsignedLong`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPAdaptiveAvgPool3DBackwardBackward0_grad_output_sym_argsize_minus_2_getter`, `PyLong_FromUnsignedLong`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4561-4640

```cpp
4561: }
4562: 
4563: static PyObject* THPLogSigmoidBackwardBackward0_buffer_raw_getter(THPCppFunction *self, void *_unused) {
4564:   HANDLE_TH_ERRORS
4565:   const auto& prop = static_cast<LogSigmoidBackwardBackward0*>(self->cdata.get())->buffer_;
4566:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4567:   return obj.release().ptr();
4568:   END_HANDLE_TH_ERRORS
4569: }
4570: 
4571: static PyObject* THPLogSigmoidBackwardBackward0_grad_output_getter(THPCppFunction *self, void *_unused) {
4572:   HANDLE_TH_ERRORS
4573:   const auto& prop = static_cast<LogSigmoidBackwardBackward0*>(self->cdata.get())->grad_output_;
4574:   return THPVariable_Wrap(prop.unpack(self->cdata));
4575:   END_HANDLE_TH_ERRORS
4576: }
4577: 
4578: static PyObject* THPLogSigmoidBackwardBackward0_grad_output_raw_getter(THPCppFunction *self, void *_unused) {
4579:   HANDLE_TH_ERRORS
4580:   const auto& prop = static_cast<LogSigmoidBackwardBackward0*>(self->cdata.get())->grad_output_;
4581:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4582:   return obj.release().ptr();
4583:   END_HANDLE_TH_ERRORS
4584: }
4585: 
4586: static PyObject* THPLogSigmoidBackwardBackward0_self_getter(THPCppFunction *self, void *_unused) {
4587:   HANDLE_TH_ERRORS
4588:   const auto& prop = static_cast<LogSigmoidBackwardBackward0*>(self->cdata.get())->self_;
4589:   return THPVariable_Wrap(prop.unpack(self->cdata));
4590:   END_HANDLE_TH_ERRORS
4591: }
4592: 
4593: static PyObject* THPLogSigmoidBackwardBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
4594:   HANDLE_TH_ERRORS
4595:   const auto& prop = static_cast<LogSigmoidBackwardBackward0*>(self->cdata.get())->self_;
4596:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4597:   return obj.release().ptr();
4598:   END_HANDLE_TH_ERRORS
4599: }
4600: 
4601: static struct PyGetSetDef LogSigmoidBackwardBackward0_properties[] = {
4602:   THP_FUNCTION_DEFAULT_PROPERTIES,
4603:   {(char*)"_saved_buffer", (getter)THPLogSigmoidBackwardBackward0_buffer_getter, nullptr, nullptr, nullptr},
4604:   {(char*)"_raw_saved_buffer", (getter)THPLogSigmoidBackwardBackward0_buffer_raw_getter, nullptr, nullptr, nullptr},
4605:   {(char*)"_saved_grad_output", (getter)THPLogSigmoidBackwardBackward0_grad_output_getter, nullptr, nullptr, nullptr},
4606:   {(char*)"_raw_saved_grad_output", (getter)THPLogSigmoidBackwardBackward0_grad_output_raw_getter, nullptr, nullptr, nullptr},
4607:   {(char*)"_saved_self", (getter)THPLogSigmoidBackwardBackward0_self_getter, nullptr, nullptr, nullptr},
4608:   {(char*)"_raw_saved_self", (getter)THPLogSigmoidBackwardBackward0_self_raw_getter, nullptr, nullptr, nullptr},
4609:   {nullptr} /* sentinel */
4610: };
4611: 
4612: static PyObject* THPLogSoftmaxBackwardDataBackward0_dim_getter(THPCppFunction *self, void *_unused) {
4613:   HANDLE_TH_ERRORS
4614:   auto prop = static_cast<LogSoftmaxBackwardDataBackward0*>(self->cdata.get())->dim;
4615:   return PyLong_FromUnsignedLong((int64_t) prop);
4616:   END_HANDLE_TH_ERRORS
4617: }
4618: 
4619: static PyObject* THPLogSoftmaxBackwardDataBackward0_grad_output_getter(THPCppFunction *self, void *_unused) {
4620:   HANDLE_TH_ERRORS
4621:   const auto& prop = static_cast<LogSoftmaxBackwardDataBackward0*>(self->cdata.get())->grad_output_;
4622:   return THPVariable_Wrap(prop.unpack(self->cdata));
4623:   END_HANDLE_TH_ERRORS
4624: }
4625: 
4626: static PyObject* THPLogSoftmaxBackwardDataBackward0_grad_output_raw_getter(THPCppFunction *self, void *_unused) {
4627:   HANDLE_TH_ERRORS
4628:   const auto& prop = static_cast<LogSoftmaxBackwardDataBackward0*>(self->cdata.get())->grad_output_;
4629:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4630:   return obj.release().ptr();
4631:   END_HANDLE_TH_ERRORS
4632: }
4633: 
4634: static PyObject* THPLogSoftmaxBackwardDataBackward0_output_getter(THPCppFunction *self, void *_unused) {
4635:   HANDLE_TH_ERRORS
4636:   const auto& prop = static_cast<LogSoftmaxBackwardDataBackward0*>(self->cdata.get())->output_;
4637:   return THPVariable_Wrap(prop.unpack(self->cdata));
4638:   END_HANDLE_TH_ERRORS
4639: }
4640: 
```

- EN: The main execution path in this span is carried by `THPLogSigmoidBackwardBackward0_buffer_raw_getter`, `cast`, `THPLogSigmoidBackwardBackward0_grad_output_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPLogSigmoidBackwardBackward0_buffer_raw_getter`, `cast`, `THPLogSigmoidBackwardBackward0_grad_output_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4641-4720

```cpp
4641: static PyObject* THPLogSoftmaxBackwardDataBackward0_output_raw_getter(THPCppFunction *self, void *_unused) {
4642:   HANDLE_TH_ERRORS
4643:   const auto& prop = static_cast<LogSoftmaxBackwardDataBackward0*>(self->cdata.get())->output_;
4644:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4645:   return obj.release().ptr();
4646:   END_HANDLE_TH_ERRORS
4647: }
4648: 
4649: static struct PyGetSetDef LogSoftmaxBackwardDataBackward0_properties[] = {
4650:   THP_FUNCTION_DEFAULT_PROPERTIES,
4651:   {(char*)"_saved_dim", (getter)THPLogSoftmaxBackwardDataBackward0_dim_getter, nullptr, nullptr, nullptr},
4652:   {(char*)"_saved_grad_output", (getter)THPLogSoftmaxBackwardDataBackward0_grad_output_getter, nullptr, nullptr, nullptr},
4653:   {(char*)"_raw_saved_grad_output", (getter)THPLogSoftmaxBackwardDataBackward0_grad_output_raw_getter, nullptr, nullptr, nullptr},
4654:   {(char*)"_saved_output", (getter)THPLogSoftmaxBackwardDataBackward0_output_getter, nullptr, nullptr, nullptr},
4655:   {(char*)"_raw_saved_output", (getter)THPLogSoftmaxBackwardDataBackward0_output_raw_getter, nullptr, nullptr, nullptr},
4656:   {nullptr} /* sentinel */
4657: };
4658: 
4659: static PyObject* THPNllLoss2DBackwardBackward0_ignore_index_getter(THPCppFunction *self, void *_unused) {
4660:   HANDLE_TH_ERRORS
4661:   auto prop = static_cast<NllLoss2DBackwardBackward0*>(self->cdata.get())->ignore_index;
4662:   if (auto m = prop.maybe_as_int()) {
4663:     return PyLong_FromUnsignedLong(*m);
4664:   } else {
4665:     return py::cast(prop).release().ptr();
4666:   }
4667:   END_HANDLE_TH_ERRORS
4668: }
4669: 
4670: static PyObject* THPNllLoss2DBackwardBackward0_reduction_getter(THPCppFunction *self, void *_unused) {
4671:   HANDLE_TH_ERRORS
4672:   auto prop = static_cast<NllLoss2DBackwardBackward0*>(self->cdata.get())->reduction;
4673:   return PyLong_FromUnsignedLong((int64_t) prop);
4674:   END_HANDLE_TH_ERRORS
4675: }
4676: 
4677: static PyObject* THPNllLoss2DBackwardBackward0_target_getter(THPCppFunction *self, void *_unused) {
4678:   HANDLE_TH_ERRORS
4679:   const auto& prop = static_cast<NllLoss2DBackwardBackward0*>(self->cdata.get())->target_;
4680:   return THPVariable_Wrap(prop.unpack(self->cdata));
4681:   END_HANDLE_TH_ERRORS
4682: }
4683: 
4684: static PyObject* THPNllLoss2DBackwardBackward0_target_raw_getter(THPCppFunction *self, void *_unused) {
4685:   HANDLE_TH_ERRORS
4686:   const auto& prop = static_cast<NllLoss2DBackwardBackward0*>(self->cdata.get())->target_;
4687:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4688:   return obj.release().ptr();
4689:   END_HANDLE_TH_ERRORS
4690: }
4691: 
4692: static PyObject* THPNllLoss2DBackwardBackward0_weight_getter(THPCppFunction *self, void *_unused) {
4693:   HANDLE_TH_ERRORS
4694:   const auto& prop = static_cast<NllLoss2DBackwardBackward0*>(self->cdata.get())->weight_;
4695:   return THPVariable_Wrap(prop.unpack(self->cdata));
4696:   END_HANDLE_TH_ERRORS
4697: }
4698: 
4699: static PyObject* THPNllLoss2DBackwardBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
4700:   HANDLE_TH_ERRORS
4701:   const auto& prop = static_cast<NllLoss2DBackwardBackward0*>(self->cdata.get())->weight_;
4702:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4703:   return obj.release().ptr();
4704:   END_HANDLE_TH_ERRORS
4705: }
4706: 
4707: static struct PyGetSetDef NllLoss2DBackwardBackward0_properties[] = {
4708:   THP_FUNCTION_DEFAULT_PROPERTIES,
4709:   {(char*)"_saved_ignore_index", (getter)THPNllLoss2DBackwardBackward0_ignore_index_getter, nullptr, nullptr, nullptr},
4710:   {(char*)"_saved_reduction", (getter)THPNllLoss2DBackwardBackward0_reduction_getter, nullptr, nullptr, nullptr},
4711:   {(char*)"_saved_target", (getter)THPNllLoss2DBackwardBackward0_target_getter, nullptr, nullptr, nullptr},
4712:   {(char*)"_raw_saved_target", (getter)THPNllLoss2DBackwardBackward0_target_raw_getter, nullptr, nullptr, nullptr},
4713:   {(char*)"_saved_weight", (getter)THPNllLoss2DBackwardBackward0_weight_getter, nullptr, nullptr, nullptr},
4714:   {(char*)"_raw_saved_weight", (getter)THPNllLoss2DBackwardBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
4715:   {nullptr} /* sentinel */
4716: };
4717: 
4718: static PyObject* THPSparseSampledAddmmBackward0_alpha_getter(THPCppFunction *self, void *_unused) {
4719:   HANDLE_TH_ERRORS
4720:   auto prop = static_cast<SparseSampledAddmmBackward0*>(self->cdata.get())->alpha;
```

- EN: The main execution path in this span is carried by `THPLogSoftmaxBackwardDataBackward0_output_raw_getter`, `cast`, `THPNllLoss2DBackwardBackward0_ignore_index_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPLogSoftmaxBackwardDataBackward0_output_raw_getter`, `cast`, `THPNllLoss2DBackwardBackward0_ignore_index_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4721-4800

```cpp
4721:   if (prop.isComplex()) {
4722:     auto cprop = prop.to<c10::complex<double>>();
4723:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
4724:   } else if (prop.isFloatingPoint()) {
4725:     return PyFloat_FromDouble(prop.to<double>());
4726:   } else if (prop.isIntegral(/*includeBool=*/false)) {
4727:     return PyLong_FromLong(prop.to<int64_t>());
4728:   } else if (prop.isBoolean()) {
4729:     if (prop.to<bool>()) {
4730:       Py_RETURN_TRUE;
4731:     } else {
4732:       Py_RETURN_FALSE;
4733:     }
4734:   } else {
4735:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
4736:     return nullptr;
4737:   }
4738:   END_HANDLE_TH_ERRORS
4739: }
4740: 
4741: static PyObject* THPSparseSampledAddmmBackward0_beta_getter(THPCppFunction *self, void *_unused) {
4742:   HANDLE_TH_ERRORS
4743:   auto prop = static_cast<SparseSampledAddmmBackward0*>(self->cdata.get())->beta;
4744:   if (prop.isComplex()) {
4745:     auto cprop = prop.to<c10::complex<double>>();
4746:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
4747:   } else if (prop.isFloatingPoint()) {
4748:     return PyFloat_FromDouble(prop.to<double>());
4749:   } else if (prop.isIntegral(/*includeBool=*/false)) {
4750:     return PyLong_FromLong(prop.to<int64_t>());
4751:   } else if (prop.isBoolean()) {
4752:     if (prop.to<bool>()) {
4753:       Py_RETURN_TRUE;
4754:     } else {
4755:       Py_RETURN_FALSE;
4756:     }
4757:   } else {
4758:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
4759:     return nullptr;
4760:   }
4761:   END_HANDLE_TH_ERRORS
4762: }
4763: 
4764: static PyObject* THPSparseSampledAddmmBackward0_mat1_getter(THPCppFunction *self, void *_unused) {
4765:   HANDLE_TH_ERRORS
4766:   const auto& prop = static_cast<SparseSampledAddmmBackward0*>(self->cdata.get())->mat1_;
4767:   return THPVariable_Wrap(prop.unpack(self->cdata));
4768:   END_HANDLE_TH_ERRORS
4769: }
4770: 
4771: static PyObject* THPSparseSampledAddmmBackward0_mat1_raw_getter(THPCppFunction *self, void *_unused) {
4772:   HANDLE_TH_ERRORS
4773:   const auto& prop = static_cast<SparseSampledAddmmBackward0*>(self->cdata.get())->mat1_;
4774:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4775:   return obj.release().ptr();
4776:   END_HANDLE_TH_ERRORS
4777: }
4778: 
4779: static PyObject* THPSparseSampledAddmmBackward0_mat2_getter(THPCppFunction *self, void *_unused) {
4780:   HANDLE_TH_ERRORS
4781:   const auto& prop = static_cast<SparseSampledAddmmBackward0*>(self->cdata.get())->mat2_;
4782:   return THPVariable_Wrap(prop.unpack(self->cdata));
4783:   END_HANDLE_TH_ERRORS
4784: }
4785: 
4786: static PyObject* THPSparseSampledAddmmBackward0_mat2_raw_getter(THPCppFunction *self, void *_unused) {
4787:   HANDLE_TH_ERRORS
4788:   const auto& prop = static_cast<SparseSampledAddmmBackward0*>(self->cdata.get())->mat2_;
4789:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4790:   return obj.release().ptr();
4791:   END_HANDLE_TH_ERRORS
4792: }
4793: 
4794: static PyObject* THPSparseSampledAddmmBackward0_self_getter(THPCppFunction *self, void *_unused) {
4795:   HANDLE_TH_ERRORS
4796:   const auto& prop = static_cast<SparseSampledAddmmBackward0*>(self->cdata.get())->self_;
4797:   return THPVariable_Wrap(prop.unpack(self->cdata));
4798:   END_HANDLE_TH_ERRORS
4799: }
4800: 
```

- EN: The main execution path in this span is carried by `PyComplex_FromDoubles`, `PyFloat_FromDouble`, `PyLong_FromLong`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyComplex_FromDoubles`, `PyFloat_FromDouble`, `PyLong_FromLong` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4801-4880

```cpp
4801: static PyObject* THPSparseSampledAddmmBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
4802:   HANDLE_TH_ERRORS
4803:   const auto& prop = static_cast<SparseSampledAddmmBackward0*>(self->cdata.get())->self_;
4804:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4805:   return obj.release().ptr();
4806:   END_HANDLE_TH_ERRORS
4807: }
4808: 
4809: static struct PyGetSetDef SparseSampledAddmmBackward0_properties[] = {
4810:   THP_FUNCTION_DEFAULT_PROPERTIES,
4811:   {(char*)"_saved_alpha", (getter)THPSparseSampledAddmmBackward0_alpha_getter, nullptr, nullptr, nullptr},
4812:   {(char*)"_saved_beta", (getter)THPSparseSampledAddmmBackward0_beta_getter, nullptr, nullptr, nullptr},
4813:   {(char*)"_saved_mat1", (getter)THPSparseSampledAddmmBackward0_mat1_getter, nullptr, nullptr, nullptr},
4814:   {(char*)"_raw_saved_mat1", (getter)THPSparseSampledAddmmBackward0_mat1_raw_getter, nullptr, nullptr, nullptr},
4815:   {(char*)"_saved_mat2", (getter)THPSparseSampledAddmmBackward0_mat2_getter, nullptr, nullptr, nullptr},
4816:   {(char*)"_raw_saved_mat2", (getter)THPSparseSampledAddmmBackward0_mat2_raw_getter, nullptr, nullptr, nullptr},
4817:   {(char*)"_saved_self", (getter)THPSparseSampledAddmmBackward0_self_getter, nullptr, nullptr, nullptr},
4818:   {(char*)"_raw_saved_self", (getter)THPSparseSampledAddmmBackward0_self_raw_getter, nullptr, nullptr, nullptr},
4819:   {nullptr} /* sentinel */
4820: };
4821: 
4822: static PyObject* THPSoftmaxBackwardDataBackward0_dim_getter(THPCppFunction *self, void *_unused) {
4823:   HANDLE_TH_ERRORS
4824:   auto prop = static_cast<SoftmaxBackwardDataBackward0*>(self->cdata.get())->dim;
4825:   return PyLong_FromUnsignedLong((int64_t) prop);
4826:   END_HANDLE_TH_ERRORS
4827: }
4828: 
4829: static PyObject* THPSoftmaxBackwardDataBackward0_grad_output_getter(THPCppFunction *self, void *_unused) {
4830:   HANDLE_TH_ERRORS
4831:   const auto& prop = static_cast<SoftmaxBackwardDataBackward0*>(self->cdata.get())->grad_output_;
4832:   return THPVariable_Wrap(prop.unpack(self->cdata));
4833:   END_HANDLE_TH_ERRORS
4834: }
4835: 
4836: static PyObject* THPSoftmaxBackwardDataBackward0_grad_output_raw_getter(THPCppFunction *self, void *_unused) {
4837:   HANDLE_TH_ERRORS
4838:   const auto& prop = static_cast<SoftmaxBackwardDataBackward0*>(self->cdata.get())->grad_output_;
4839:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4840:   return obj.release().ptr();
4841:   END_HANDLE_TH_ERRORS
4842: }
4843: 
4844: static PyObject* THPSoftmaxBackwardDataBackward0_output_getter(THPCppFunction *self, void *_unused) {
4845:   HANDLE_TH_ERRORS
4846:   const auto& prop = static_cast<SoftmaxBackwardDataBackward0*>(self->cdata.get())->output_;
4847:   return THPVariable_Wrap(prop.unpack(self->cdata));
4848:   END_HANDLE_TH_ERRORS
4849: }
4850: 
4851: static PyObject* THPSoftmaxBackwardDataBackward0_output_raw_getter(THPCppFunction *self, void *_unused) {
4852:   HANDLE_TH_ERRORS
4853:   const auto& prop = static_cast<SoftmaxBackwardDataBackward0*>(self->cdata.get())->output_;
4854:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4855:   return obj.release().ptr();
4856:   END_HANDLE_TH_ERRORS
4857: }
4858: 
4859: static struct PyGetSetDef SoftmaxBackwardDataBackward0_properties[] = {
4860:   THP_FUNCTION_DEFAULT_PROPERTIES,
4861:   {(char*)"_saved_dim", (getter)THPSoftmaxBackwardDataBackward0_dim_getter, nullptr, nullptr, nullptr},
4862:   {(char*)"_saved_grad_output", (getter)THPSoftmaxBackwardDataBackward0_grad_output_getter, nullptr, nullptr, nullptr},
4863:   {(char*)"_raw_saved_grad_output", (getter)THPSoftmaxBackwardDataBackward0_grad_output_raw_getter, nullptr, nullptr, nullptr},
4864:   {(char*)"_saved_output", (getter)THPSoftmaxBackwardDataBackward0_output_getter, nullptr, nullptr, nullptr},
4865:   {(char*)"_raw_saved_output", (getter)THPSoftmaxBackwardDataBackward0_output_raw_getter, nullptr, nullptr, nullptr},
4866:   {nullptr} /* sentinel */
4867: };
4868: 
4869: static PyObject* THPCudnnGridSamplerBackward0_grid_getter(THPCppFunction *self, void *_unused) {
4870:   HANDLE_TH_ERRORS
4871:   const auto& prop = static_cast<CudnnGridSamplerBackward0*>(self->cdata.get())->grid_;
4872:   return THPVariable_Wrap(prop.unpack(self->cdata));
4873:   END_HANDLE_TH_ERRORS
4874: }
4875: 
4876: static PyObject* THPCudnnGridSamplerBackward0_grid_raw_getter(THPCppFunction *self, void *_unused) {
4877:   HANDLE_TH_ERRORS
4878:   const auto& prop = static_cast<CudnnGridSamplerBackward0*>(self->cdata.get())->grid_;
4879:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4880:   return obj.release().ptr();
```

- EN: The main execution path in this span is carried by `THPSparseSampledAddmmBackward0_self_raw_getter`, `cast`, `THPSoftmaxBackwardDataBackward0_dim_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPSparseSampledAddmmBackward0_self_raw_getter`, `cast`, `THPSoftmaxBackwardDataBackward0_dim_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4881-4960

```cpp
4881:   END_HANDLE_TH_ERRORS
4882: }
4883: 
4884: static PyObject* THPCudnnGridSamplerBackward0_self_getter(THPCppFunction *self, void *_unused) {
4885:   HANDLE_TH_ERRORS
4886:   const auto& prop = static_cast<CudnnGridSamplerBackward0*>(self->cdata.get())->self_;
4887:   return THPVariable_Wrap(prop.unpack(self->cdata));
4888:   END_HANDLE_TH_ERRORS
4889: }
4890: 
4891: static PyObject* THPCudnnGridSamplerBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
4892:   HANDLE_TH_ERRORS
4893:   const auto& prop = static_cast<CudnnGridSamplerBackward0*>(self->cdata.get())->self_;
4894:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4895:   return obj.release().ptr();
4896:   END_HANDLE_TH_ERRORS
4897: }
4898: 
4899: static struct PyGetSetDef CudnnGridSamplerBackward0_properties[] = {
4900:   THP_FUNCTION_DEFAULT_PROPERTIES,
4901:   {(char*)"_saved_grid", (getter)THPCudnnGridSamplerBackward0_grid_getter, nullptr, nullptr, nullptr},
4902:   {(char*)"_raw_saved_grid", (getter)THPCudnnGridSamplerBackward0_grid_raw_getter, nullptr, nullptr, nullptr},
4903:   {(char*)"_saved_self", (getter)THPCudnnGridSamplerBackward0_self_getter, nullptr, nullptr, nullptr},
4904:   {(char*)"_raw_saved_self", (getter)THPCudnnGridSamplerBackward0_self_raw_getter, nullptr, nullptr, nullptr},
4905:   {nullptr} /* sentinel */
4906: };
4907: 
4908: static PyObject* THPNnpackSpatialConvolutionBackward0_bias_sym_sizes_opt_getter(THPCppFunction *self, void *_unused) {
4909:   HANDLE_TH_ERRORS
4910:   auto opt_prop = static_cast<NnpackSpatialConvolutionBackward0*>(self->cdata.get())->bias_sym_sizes_opt;
4911:   if (!opt_prop.list.has_value()) {
4912:     Py_RETURN_NONE;
4913:   }
4914:   auto prop = opt_prop.list.value();
4915:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4916:   for (auto i : c10::irange(prop.size())) {
4917:       auto si = prop[i];
4918:       if (auto m = si.maybe_as_int()) {
4919:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4920:       } else {
4921:         auto py_symint = py::cast(si).release().ptr();
4922:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4923:       }
4924:   }
4925:   return tup;
4926:   END_HANDLE_TH_ERRORS
4927: }
4928: 
4929: static PyObject* THPNnpackSpatialConvolutionBackward0_input_getter(THPCppFunction *self, void *_unused) {
4930:   HANDLE_TH_ERRORS
4931:   const auto& prop = static_cast<NnpackSpatialConvolutionBackward0*>(self->cdata.get())->input_;
4932:   return THPVariable_Wrap(prop.unpack(self->cdata));
4933:   END_HANDLE_TH_ERRORS
4934: }
4935: 
4936: static PyObject* THPNnpackSpatialConvolutionBackward0_input_raw_getter(THPCppFunction *self, void *_unused) {
4937:   HANDLE_TH_ERRORS
4938:   const auto& prop = static_cast<NnpackSpatialConvolutionBackward0*>(self->cdata.get())->input_;
4939:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4940:   return obj.release().ptr();
4941:   END_HANDLE_TH_ERRORS
4942: }
4943: 
4944: static PyObject* THPNnpackSpatialConvolutionBackward0_padding_getter(THPCppFunction *self, void *_unused) {
4945:   HANDLE_TH_ERRORS
4946:   auto prop = static_cast<NnpackSpatialConvolutionBackward0*>(self->cdata.get())->padding;
4947:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4948:   for (auto i : c10::irange(prop.size())) {
4949:       auto si = prop[i];
4950:       if (auto m = si.maybe_as_int()) {
4951:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4952:       } else {
4953:         auto py_symint = py::cast(si).release().ptr();
4954:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4955:       }
4956:   }
4957:   return tup;
4958:   END_HANDLE_TH_ERRORS
4959: }
4960: 
```

- EN: The main execution path in this span is carried by `THPCudnnGridSamplerBackward0_self_getter`, `THPVariable_Wrap`, `THPCudnnGridSamplerBackward0_self_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPCudnnGridSamplerBackward0_self_getter`, `THPVariable_Wrap`, `THPCudnnGridSamplerBackward0_self_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4961-5040

```cpp
4961: static PyObject* THPNnpackSpatialConvolutionBackward0_stride_getter(THPCppFunction *self, void *_unused) {
4962:   HANDLE_TH_ERRORS
4963:   auto prop = static_cast<NnpackSpatialConvolutionBackward0*>(self->cdata.get())->stride;
4964:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4965:   for (auto i : c10::irange(prop.size())) {
4966:       auto si = prop[i];
4967:       if (auto m = si.maybe_as_int()) {
4968:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4969:       } else {
4970:         auto py_symint = py::cast(si).release().ptr();
4971:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4972:       }
4973:   }
4974:   return tup;
4975:   END_HANDLE_TH_ERRORS
4976: }
4977: 
4978: static PyObject* THPNnpackSpatialConvolutionBackward0_weight_getter(THPCppFunction *self, void *_unused) {
4979:   HANDLE_TH_ERRORS
4980:   const auto& prop = static_cast<NnpackSpatialConvolutionBackward0*>(self->cdata.get())->weight_;
4981:   return THPVariable_Wrap(prop.unpack(self->cdata));
4982:   END_HANDLE_TH_ERRORS
4983: }
4984: 
4985: static PyObject* THPNnpackSpatialConvolutionBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
4986:   HANDLE_TH_ERRORS
4987:   const auto& prop = static_cast<NnpackSpatialConvolutionBackward0*>(self->cdata.get())->weight_;
4988:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4989:   return obj.release().ptr();
4990:   END_HANDLE_TH_ERRORS
4991: }
4992: 
4993: static struct PyGetSetDef NnpackSpatialConvolutionBackward0_properties[] = {
4994:   THP_FUNCTION_DEFAULT_PROPERTIES,
4995:   {(char*)"_saved_bias_sym_sizes_opt", (getter)THPNnpackSpatialConvolutionBackward0_bias_sym_sizes_opt_getter, nullptr, nullptr, nullptr},
4996:   {(char*)"_saved_input", (getter)THPNnpackSpatialConvolutionBackward0_input_getter, nullptr, nullptr, nullptr},
4997:   {(char*)"_raw_saved_input", (getter)THPNnpackSpatialConvolutionBackward0_input_raw_getter, nullptr, nullptr, nullptr},
4998:   {(char*)"_saved_padding", (getter)THPNnpackSpatialConvolutionBackward0_padding_getter, nullptr, nullptr, nullptr},
4999:   {(char*)"_saved_stride", (getter)THPNnpackSpatialConvolutionBackward0_stride_getter, nullptr, nullptr, nullptr},
5000:   {(char*)"_saved_weight", (getter)THPNnpackSpatialConvolutionBackward0_weight_getter, nullptr, nullptr, nullptr},
5001:   {(char*)"_raw_saved_weight", (getter)THPNnpackSpatialConvolutionBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
5002:   {nullptr} /* sentinel */
5003: };
5004: 
5005: static PyObject* THPLstmMpsBackward0_batch_first_getter(THPCppFunction *self, void *_unused) {
5006:   HANDLE_TH_ERRORS
5007:   auto prop = static_cast<LstmMpsBackward0*>(self->cdata.get())->batch_first;
5008:   if (prop) {
5009:     Py_RETURN_TRUE;
5010:   } else {
5011:     Py_RETURN_FALSE;
5012:   }
5013:   END_HANDLE_TH_ERRORS
5014: }
5015: 
5016: static PyObject* THPLstmMpsBackward0_bidirectional_getter(THPCppFunction *self, void *_unused) {
5017:   HANDLE_TH_ERRORS
5018:   auto prop = static_cast<LstmMpsBackward0*>(self->cdata.get())->bidirectional;
5019:   if (prop) {
5020:     Py_RETURN_TRUE;
5021:   } else {
5022:     Py_RETURN_FALSE;
5023:   }
5024:   END_HANDLE_TH_ERRORS
5025: }
5026: 
5027: static PyObject* THPLstmMpsBackward0_dropout_getter(THPCppFunction *self, void *_unused) {
5028:   HANDLE_TH_ERRORS
5029:   auto prop = static_cast<LstmMpsBackward0*>(self->cdata.get())->dropout;
5030:   return PyFloat_FromDouble((double) prop);
5031:   END_HANDLE_TH_ERRORS
5032: }
5033: 
5034: static PyObject* THPLstmMpsBackward0_has_biases_getter(THPCppFunction *self, void *_unused) {
5035:   HANDLE_TH_ERRORS
5036:   auto prop = static_cast<LstmMpsBackward0*>(self->cdata.get())->has_biases;
5037:   if (prop) {
5038:     Py_RETURN_TRUE;
5039:   } else {
5040:     Py_RETURN_FALSE;
```

- EN: The main execution path in this span is carried by `THPNnpackSpatialConvolutionBackward0_stride_getter`, `PyTuple_New`, `PyTuple_SetItem`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPNnpackSpatialConvolutionBackward0_stride_getter`, `PyTuple_New`, `PyTuple_SetItem` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5041-5120

```cpp
5041:   }
5042:   END_HANDLE_TH_ERRORS
5043: }
5044: 
5045: static PyObject* THPLstmMpsBackward0_hx_getter(THPCppFunction *self, void *_unused) {
5046:   HANDLE_TH_ERRORS
5047:   const auto *node = static_cast<LstmMpsBackward0*>(self->cdata.get());
5048:   const auto& prop = node->hx_;
5049:   if (node->hx_released_) {
5050:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
5051:     return nullptr;
5052:   }
5053:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5054:   for (auto i: c10::irange(prop.size())) {
5055:     PyTuple_SetItem(tup, (Py_ssize_t) i, THPVariable_Wrap(prop[i].unpack(self->cdata)));
5056:   }
5057:   return tup;
5058:   END_HANDLE_TH_ERRORS
5059: }
5060: 
5061: static PyObject* THPLstmMpsBackward0_hx_raw_getter(THPCppFunction *self, void *_unused) {
5062:   HANDLE_TH_ERRORS
5063:   const auto *node = static_cast<LstmMpsBackward0*>(self->cdata.get());
5064:   const auto& prop = node->hx_;
5065:   if (node->hx_released_) {
5066:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
5067:     return nullptr;
5068:   }
5069:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5070:   for (auto i : c10::irange(prop.size())) {
5071:     pybind11::object obj = pybind11::cast(prop[i], pybind11::return_value_policy::reference);
5072:     PyTuple_SetItem(tup, (Py_ssize_t) i, obj.release().ptr());
5073:   }
5074:   return tup;
5075:   END_HANDLE_TH_ERRORS
5076: }
5077: 
5078: static PyObject* THPLstmMpsBackward0_input_getter(THPCppFunction *self, void *_unused) {
5079:   HANDLE_TH_ERRORS
5080:   const auto& prop = static_cast<LstmMpsBackward0*>(self->cdata.get())->input_;
5081:   return THPVariable_Wrap(prop.unpack(self->cdata));
5082:   END_HANDLE_TH_ERRORS
5083: }
5084: 
5085: static PyObject* THPLstmMpsBackward0_input_raw_getter(THPCppFunction *self, void *_unused) {
5086:   HANDLE_TH_ERRORS
5087:   const auto& prop = static_cast<LstmMpsBackward0*>(self->cdata.get())->input_;
5088:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5089:   return obj.release().ptr();
5090:   END_HANDLE_TH_ERRORS
5091: }
5092: 
5093: static PyObject* THPLstmMpsBackward0_num_layers_getter(THPCppFunction *self, void *_unused) {
5094:   HANDLE_TH_ERRORS
5095:   auto prop = static_cast<LstmMpsBackward0*>(self->cdata.get())->num_layers;
5096:   return PyLong_FromUnsignedLong((int64_t) prop);
5097:   END_HANDLE_TH_ERRORS
5098: }
5099: 
5100: static PyObject* THPLstmMpsBackward0_params_getter(THPCppFunction *self, void *_unused) {
5101:   HANDLE_TH_ERRORS
5102:   const auto *node = static_cast<LstmMpsBackward0*>(self->cdata.get());
5103:   const auto& prop = node->params_;
5104:   if (node->params_released_) {
5105:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
5106:     return nullptr;
5107:   }
5108:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5109:   for (auto i: c10::irange(prop.size())) {
5110:     PyTuple_SetItem(tup, (Py_ssize_t) i, THPVariable_Wrap(prop[i].unpack(self->cdata)));
5111:   }
5112:   return tup;
5113:   END_HANDLE_TH_ERRORS
5114: }
5115: 
5116: static PyObject* THPLstmMpsBackward0_params_raw_getter(THPCppFunction *self, void *_unused) {
5117:   HANDLE_TH_ERRORS
5118:   const auto *node = static_cast<LstmMpsBackward0*>(self->cdata.get());
5119:   const auto& prop = node->params_;
5120:   if (node->params_released_) {
```

- EN: The main execution path in this span is carried by `THPLstmMpsBackward0_hx_getter`, `PyErr_SetString`, `PyTuple_New`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPLstmMpsBackward0_hx_getter`, `PyErr_SetString`, `PyTuple_New` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5121-5200

```cpp
5121:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
5122:     return nullptr;
5123:   }
5124:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5125:   for (auto i : c10::irange(prop.size())) {
5126:     pybind11::object obj = pybind11::cast(prop[i], pybind11::return_value_policy::reference);
5127:     PyTuple_SetItem(tup, (Py_ssize_t) i, obj.release().ptr());
5128:   }
5129:   return tup;
5130:   END_HANDLE_TH_ERRORS
5131: }
5132: 
5133: static PyObject* THPLstmMpsBackward0_train_getter(THPCppFunction *self, void *_unused) {
5134:   HANDLE_TH_ERRORS
5135:   auto prop = static_cast<LstmMpsBackward0*>(self->cdata.get())->train;
5136:   if (prop) {
5137:     Py_RETURN_TRUE;
5138:   } else {
5139:     Py_RETURN_FALSE;
5140:   }
5141:   END_HANDLE_TH_ERRORS
5142: }
5143: 
5144: static PyObject* THPLstmMpsBackward0_result3_getter(THPCppFunction *self, void *_unused) {
5145:   HANDLE_TH_ERRORS
5146:   const auto& prop = static_cast<LstmMpsBackward0*>(self->cdata.get())->result3_;
5147:   return THPVariable_Wrap(prop.unpack(self->cdata));
5148:   END_HANDLE_TH_ERRORS
5149: }
5150: 
5151: static PyObject* THPLstmMpsBackward0_result3_raw_getter(THPCppFunction *self, void *_unused) {
5152:   HANDLE_TH_ERRORS
5153:   const auto& prop = static_cast<LstmMpsBackward0*>(self->cdata.get())->result3_;
5154:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5155:   return obj.release().ptr();
5156:   END_HANDLE_TH_ERRORS
5157: }
5158: 
5159: static PyObject* THPLstmMpsBackward0_result4_getter(THPCppFunction *self, void *_unused) {
5160:   HANDLE_TH_ERRORS
5161:   const auto& prop = static_cast<LstmMpsBackward0*>(self->cdata.get())->result4_;
5162:   return THPVariable_Wrap(prop.unpack(self->cdata));
5163:   END_HANDLE_TH_ERRORS
5164: }
5165: 
5166: static PyObject* THPLstmMpsBackward0_result4_raw_getter(THPCppFunction *self, void *_unused) {
5167:   HANDLE_TH_ERRORS
5168:   const auto& prop = static_cast<LstmMpsBackward0*>(self->cdata.get())->result4_;
5169:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5170:   return obj.release().ptr();
5171:   END_HANDLE_TH_ERRORS
5172: }
5173: 
5174: static PyObject* THPLstmMpsBackward0_result5_getter(THPCppFunction *self, void *_unused) {
5175:   HANDLE_TH_ERRORS
5176:   const auto& prop = static_cast<LstmMpsBackward0*>(self->cdata.get())->result5_;
5177:   return THPVariable_Wrap(prop.unpack(self->cdata));
5178:   END_HANDLE_TH_ERRORS
5179: }
5180: 
5181: static PyObject* THPLstmMpsBackward0_result5_raw_getter(THPCppFunction *self, void *_unused) {
5182:   HANDLE_TH_ERRORS
5183:   const auto& prop = static_cast<LstmMpsBackward0*>(self->cdata.get())->result5_;
5184:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5185:   return obj.release().ptr();
5186:   END_HANDLE_TH_ERRORS
5187: }
5188: 
5189: static struct PyGetSetDef LstmMpsBackward0_properties[] = {
5190:   THP_FUNCTION_DEFAULT_PROPERTIES,
5191:   {(char*)"_saved_batch_first", (getter)THPLstmMpsBackward0_batch_first_getter, nullptr, nullptr, nullptr},
5192:   {(char*)"_saved_bidirectional", (getter)THPLstmMpsBackward0_bidirectional_getter, nullptr, nullptr, nullptr},
5193:   {(char*)"_saved_dropout", (getter)THPLstmMpsBackward0_dropout_getter, nullptr, nullptr, nullptr},
5194:   {(char*)"_saved_has_biases", (getter)THPLstmMpsBackward0_has_biases_getter, nullptr, nullptr, nullptr},
5195:   {(char*)"_saved_hx", (getter)THPLstmMpsBackward0_hx_getter, nullptr, nullptr, nullptr},
5196:   {(char*)"_raw_saved_hx", (getter)THPLstmMpsBackward0_hx_raw_getter, nullptr, nullptr, nullptr},
5197:   {(char*)"_saved_input", (getter)THPLstmMpsBackward0_input_getter, nullptr, nullptr, nullptr},
5198:   {(char*)"_raw_saved_input", (getter)THPLstmMpsBackward0_input_raw_getter, nullptr, nullptr, nullptr},
5199:   {(char*)"_saved_num_layers", (getter)THPLstmMpsBackward0_num_layers_getter, nullptr, nullptr, nullptr},
5200:   {(char*)"_saved_params", (getter)THPLstmMpsBackward0_params_getter, nullptr, nullptr, nullptr},
```

- EN: The main execution path in this span is carried by `PyErr_SetString`, `PyTuple_New`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyErr_SetString`, `PyTuple_New`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5201-5280

```cpp
5201:   {(char*)"_raw_saved_params", (getter)THPLstmMpsBackward0_params_raw_getter, nullptr, nullptr, nullptr},
5202:   {(char*)"_saved_train", (getter)THPLstmMpsBackward0_train_getter, nullptr, nullptr, nullptr},
5203:   {(char*)"_saved_result3", (getter)THPLstmMpsBackward0_result3_getter, nullptr, nullptr, nullptr},
5204:   {(char*)"_raw_saved_result3", (getter)THPLstmMpsBackward0_result3_raw_getter, nullptr, nullptr, nullptr},
5205:   {(char*)"_saved_result4", (getter)THPLstmMpsBackward0_result4_getter, nullptr, nullptr, nullptr},
5206:   {(char*)"_raw_saved_result4", (getter)THPLstmMpsBackward0_result4_raw_getter, nullptr, nullptr, nullptr},
5207:   {(char*)"_saved_result5", (getter)THPLstmMpsBackward0_result5_getter, nullptr, nullptr, nullptr},
5208:   {(char*)"_raw_saved_result5", (getter)THPLstmMpsBackward0_result5_raw_getter, nullptr, nullptr, nullptr},
5209:   {nullptr} /* sentinel */
5210: };
5211: 
5212: static PyObject* THPMiopenDepthwiseConvolutionBackward0_bias_sym_sizes_opt_getter(THPCppFunction *self, void *_unused) {
5213:   HANDLE_TH_ERRORS
5214:   auto opt_prop = static_cast<MiopenDepthwiseConvolutionBackward0*>(self->cdata.get())->bias_sym_sizes_opt;
5215:   if (!opt_prop.list.has_value()) {
5216:     Py_RETURN_NONE;
5217:   }
5218:   auto prop = opt_prop.list.value();
5219:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5220:   for (auto i : c10::irange(prop.size())) {
5221:       auto si = prop[i];
5222:       if (auto m = si.maybe_as_int()) {
5223:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
5224:       } else {
5225:         auto py_symint = py::cast(si).release().ptr();
5226:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
5227:       }
5228:   }
5229:   return tup;
5230:   END_HANDLE_TH_ERRORS
5231: }
5232: 
5233: static PyObject* THPMiopenDepthwiseConvolutionBackward0_dilation_getter(THPCppFunction *self, void *_unused) {
5234:   HANDLE_TH_ERRORS
5235:   auto prop = static_cast<MiopenDepthwiseConvolutionBackward0*>(self->cdata.get())->dilation;
5236:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5237:   for (auto i : c10::irange(prop.size())) {
5238:       auto si = prop[i];
5239:       if (auto m = si.maybe_as_int()) {
5240:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
5241:       } else {
5242:         auto py_symint = py::cast(si).release().ptr();
5243:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
5244:       }
5245:   }
5246:   return tup;
5247:   END_HANDLE_TH_ERRORS
5248: }
5249: 
5250: static PyObject* THPMiopenDepthwiseConvolutionBackward0_groups_getter(THPCppFunction *self, void *_unused) {
5251:   HANDLE_TH_ERRORS
5252:   auto prop = static_cast<MiopenDepthwiseConvolutionBackward0*>(self->cdata.get())->groups;
5253:   if (auto m = prop.maybe_as_int()) {
5254:     return PyLong_FromUnsignedLong(*m);
5255:   } else {
5256:     return py::cast(prop).release().ptr();
5257:   }
5258:   END_HANDLE_TH_ERRORS
5259: }
5260: 
5261: static PyObject* THPMiopenDepthwiseConvolutionBackward0_padding_getter(THPCppFunction *self, void *_unused) {
5262:   HANDLE_TH_ERRORS
5263:   auto prop = static_cast<MiopenDepthwiseConvolutionBackward0*>(self->cdata.get())->padding;
5264:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5265:   for (auto i : c10::irange(prop.size())) {
5266:       auto si = prop[i];
5267:       if (auto m = si.maybe_as_int()) {
5268:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
5269:       } else {
5270:         auto py_symint = py::cast(si).release().ptr();
5271:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
5272:       }
5273:   }
5274:   return tup;
5275:   END_HANDLE_TH_ERRORS
5276: }
5277: 
5278: static PyObject* THPMiopenDepthwiseConvolutionBackward0_self_getter(THPCppFunction *self, void *_unused) {
5279:   HANDLE_TH_ERRORS
5280:   const auto& prop = static_cast<MiopenDepthwiseConvolutionBackward0*>(self->cdata.get())->self_;
```

- EN: The main execution path in this span is carried by `THPMiopenDepthwiseConvolutionBackward0_bias_sym_sizes_opt_getter`, `PyTuple_New`, `PyTuple_SetItem`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPMiopenDepthwiseConvolutionBackward0_bias_sym_sizes_opt_getter`, `PyTuple_New`, `PyTuple_SetItem` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5281-5360

```cpp
5281:   return THPVariable_Wrap(prop.unpack(self->cdata));
5282:   END_HANDLE_TH_ERRORS
5283: }
5284: 
5285: static PyObject* THPMiopenDepthwiseConvolutionBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
5286:   HANDLE_TH_ERRORS
5287:   const auto& prop = static_cast<MiopenDepthwiseConvolutionBackward0*>(self->cdata.get())->self_;
5288:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5289:   return obj.release().ptr();
5290:   END_HANDLE_TH_ERRORS
5291: }
5292: 
5293: static PyObject* THPMiopenDepthwiseConvolutionBackward0_stride_getter(THPCppFunction *self, void *_unused) {
5294:   HANDLE_TH_ERRORS
5295:   auto prop = static_cast<MiopenDepthwiseConvolutionBackward0*>(self->cdata.get())->stride;
5296:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5297:   for (auto i : c10::irange(prop.size())) {
5298:       auto si = prop[i];
5299:       if (auto m = si.maybe_as_int()) {
5300:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
5301:       } else {
5302:         auto py_symint = py::cast(si).release().ptr();
5303:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
5304:       }
5305:   }
5306:   return tup;
5307:   END_HANDLE_TH_ERRORS
5308: }
5309: 
5310: static PyObject* THPMiopenDepthwiseConvolutionBackward0_weight_getter(THPCppFunction *self, void *_unused) {
5311:   HANDLE_TH_ERRORS
5312:   const auto& prop = static_cast<MiopenDepthwiseConvolutionBackward0*>(self->cdata.get())->weight_;
5313:   return THPVariable_Wrap(prop.unpack(self->cdata));
5314:   END_HANDLE_TH_ERRORS
5315: }
5316: 
5317: static PyObject* THPMiopenDepthwiseConvolutionBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
5318:   HANDLE_TH_ERRORS
5319:   const auto& prop = static_cast<MiopenDepthwiseConvolutionBackward0*>(self->cdata.get())->weight_;
5320:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5321:   return obj.release().ptr();
5322:   END_HANDLE_TH_ERRORS
5323: }
5324: 
5325: static struct PyGetSetDef MiopenDepthwiseConvolutionBackward0_properties[] = {
5326:   THP_FUNCTION_DEFAULT_PROPERTIES,
5327:   {(char*)"_saved_bias_sym_sizes_opt", (getter)THPMiopenDepthwiseConvolutionBackward0_bias_sym_sizes_opt_getter, nullptr, nullptr, nullptr},
5328:   {(char*)"_saved_dilation", (getter)THPMiopenDepthwiseConvolutionBackward0_dilation_getter, nullptr, nullptr, nullptr},
5329:   {(char*)"_saved_groups", (getter)THPMiopenDepthwiseConvolutionBackward0_groups_getter, nullptr, nullptr, nullptr},
5330:   {(char*)"_saved_padding", (getter)THPMiopenDepthwiseConvolutionBackward0_padding_getter, nullptr, nullptr, nullptr},
5331:   {(char*)"_saved_self", (getter)THPMiopenDepthwiseConvolutionBackward0_self_getter, nullptr, nullptr, nullptr},
5332:   {(char*)"_raw_saved_self", (getter)THPMiopenDepthwiseConvolutionBackward0_self_raw_getter, nullptr, nullptr, nullptr},
5333:   {(char*)"_saved_stride", (getter)THPMiopenDepthwiseConvolutionBackward0_stride_getter, nullptr, nullptr, nullptr},
5334:   {(char*)"_saved_weight", (getter)THPMiopenDepthwiseConvolutionBackward0_weight_getter, nullptr, nullptr, nullptr},
5335:   {(char*)"_raw_saved_weight", (getter)THPMiopenDepthwiseConvolutionBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
5336:   {nullptr} /* sentinel */
5337: };
5338: 
5339: static PyObject* THPMkldnnMaxPool2DBackward0_ceil_mode_getter(THPCppFunction *self, void *_unused) {
5340:   HANDLE_TH_ERRORS
5341:   auto prop = static_cast<MkldnnMaxPool2DBackward0*>(self->cdata.get())->ceil_mode;
5342:   if (prop) {
5343:     Py_RETURN_TRUE;
5344:   } else {
5345:     Py_RETURN_FALSE;
5346:   }
5347:   END_HANDLE_TH_ERRORS
5348: }
5349: 
5350: static PyObject* THPMkldnnMaxPool2DBackward0_dilation_getter(THPCppFunction *self, void *_unused) {
5351:   HANDLE_TH_ERRORS
5352:   auto prop = static_cast<MkldnnMaxPool2DBackward0*>(self->cdata.get())->dilation;
5353:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5354:   for (auto i : c10::irange(prop.size())) {
5355:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
5356:   }
5357:   return tup;
5358:   END_HANDLE_TH_ERRORS
5359: }
5360: 
```

- EN: The main execution path in this span is carried by `THPVariable_Wrap`, `THPMiopenDepthwiseConvolutionBackward0_self_raw_getter`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_Wrap`, `THPMiopenDepthwiseConvolutionBackward0_self_raw_getter`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5361-5440

```cpp
5361: static PyObject* THPMkldnnMaxPool2DBackward0_kernel_size_getter(THPCppFunction *self, void *_unused) {
5362:   HANDLE_TH_ERRORS
5363:   auto prop = static_cast<MkldnnMaxPool2DBackward0*>(self->cdata.get())->kernel_size;
5364:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5365:   for (auto i : c10::irange(prop.size())) {
5366:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
5367:   }
5368:   return tup;
5369:   END_HANDLE_TH_ERRORS
5370: }
5371: 
5372: static PyObject* THPMkldnnMaxPool2DBackward0_padding_getter(THPCppFunction *self, void *_unused) {
5373:   HANDLE_TH_ERRORS
5374:   auto prop = static_cast<MkldnnMaxPool2DBackward0*>(self->cdata.get())->padding;
5375:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5376:   for (auto i : c10::irange(prop.size())) {
5377:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
5378:   }
5379:   return tup;
5380:   END_HANDLE_TH_ERRORS
5381: }
5382: 
5383: static PyObject* THPMkldnnMaxPool2DBackward0_self_getter(THPCppFunction *self, void *_unused) {
5384:   HANDLE_TH_ERRORS
5385:   const auto& prop = static_cast<MkldnnMaxPool2DBackward0*>(self->cdata.get())->self_;
5386:   return THPVariable_Wrap(prop.unpack(self->cdata));
5387:   END_HANDLE_TH_ERRORS
5388: }
5389: 
5390: static PyObject* THPMkldnnMaxPool2DBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
5391:   HANDLE_TH_ERRORS
5392:   const auto& prop = static_cast<MkldnnMaxPool2DBackward0*>(self->cdata.get())->self_;
5393:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5394:   return obj.release().ptr();
5395:   END_HANDLE_TH_ERRORS
5396: }
5397: 
5398: static PyObject* THPMkldnnMaxPool2DBackward0_stride_getter(THPCppFunction *self, void *_unused) {
5399:   HANDLE_TH_ERRORS
5400:   auto prop = static_cast<MkldnnMaxPool2DBackward0*>(self->cdata.get())->stride;
5401:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5402:   for (auto i : c10::irange(prop.size())) {
5403:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
5404:   }
5405:   return tup;
5406:   END_HANDLE_TH_ERRORS
5407: }
5408: 
5409: static PyObject* THPMkldnnMaxPool2DBackward0_result_getter(THPCppFunction *self, void *_unused) {
5410:   HANDLE_TH_ERRORS
5411:   const auto& prop = static_cast<MkldnnMaxPool2DBackward0*>(self->cdata.get())->result_;
5412:   return THPVariable_Wrap(prop.unpack(self->cdata));
5413:   END_HANDLE_TH_ERRORS
5414: }
5415: 
5416: static PyObject* THPMkldnnMaxPool2DBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
5417:   HANDLE_TH_ERRORS
5418:   const auto& prop = static_cast<MkldnnMaxPool2DBackward0*>(self->cdata.get())->result_;
5419:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5420:   return obj.release().ptr();
5421:   END_HANDLE_TH_ERRORS
5422: }
5423: 
5424: static struct PyGetSetDef MkldnnMaxPool2DBackward0_properties[] = {
5425:   THP_FUNCTION_DEFAULT_PROPERTIES,
5426:   {(char*)"_saved_ceil_mode", (getter)THPMkldnnMaxPool2DBackward0_ceil_mode_getter, nullptr, nullptr, nullptr},
5427:   {(char*)"_saved_dilation", (getter)THPMkldnnMaxPool2DBackward0_dilation_getter, nullptr, nullptr, nullptr},
5428:   {(char*)"_saved_kernel_size", (getter)THPMkldnnMaxPool2DBackward0_kernel_size_getter, nullptr, nullptr, nullptr},
5429:   {(char*)"_saved_padding", (getter)THPMkldnnMaxPool2DBackward0_padding_getter, nullptr, nullptr, nullptr},
5430:   {(char*)"_saved_self", (getter)THPMkldnnMaxPool2DBackward0_self_getter, nullptr, nullptr, nullptr},
5431:   {(char*)"_raw_saved_self", (getter)THPMkldnnMaxPool2DBackward0_self_raw_getter, nullptr, nullptr, nullptr},
5432:   {(char*)"_saved_stride", (getter)THPMkldnnMaxPool2DBackward0_stride_getter, nullptr, nullptr, nullptr},
5433:   {(char*)"_saved_result", (getter)THPMkldnnMaxPool2DBackward0_result_getter, nullptr, nullptr, nullptr},
5434:   {(char*)"_raw_saved_result", (getter)THPMkldnnMaxPool2DBackward0_result_raw_getter, nullptr, nullptr, nullptr},
5435:   {nullptr} /* sentinel */
5436: };
5437: 
5438: static PyObject* THPScaledDotProductFlashAttentionBackward0_dropout_p_getter(THPCppFunction *self, void *_unused) {
5439:   HANDLE_TH_ERRORS
5440:   auto prop = static_cast<ScaledDotProductFlashAttentionBackward0*>(self->cdata.get())->dropout_p;
```

- EN: The main execution path in this span is carried by `THPMkldnnMaxPool2DBackward0_kernel_size_getter`, `PyTuple_New`, `PyTuple_SetItem`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPMkldnnMaxPool2DBackward0_kernel_size_getter`, `PyTuple_New`, `PyTuple_SetItem` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 5441-5520

```cpp
5441:   return PyFloat_FromDouble((double) prop);
5442:   END_HANDLE_TH_ERRORS
5443: }
5444: 
5445: static PyObject* THPScaledDotProductFlashAttentionBackward0_is_causal_getter(THPCppFunction *self, void *_unused) {
5446:   HANDLE_TH_ERRORS
5447:   auto prop = static_cast<ScaledDotProductFlashAttentionBackward0*>(self->cdata.get())->is_causal;
5448:   if (prop) {
5449:     Py_RETURN_TRUE;
5450:   } else {
5451:     Py_RETURN_FALSE;
5452:   }
5453:   END_HANDLE_TH_ERRORS
5454: }
5455: 
5456: static PyObject* THPScaledDotProductFlashAttentionBackward0_key_getter(THPCppFunction *self, void *_unused) {
5457:   HANDLE_TH_ERRORS
5458:   const auto& prop = static_cast<ScaledDotProductFlashAttentionBackward0*>(self->cdata.get())->key_;
5459:   return THPVariable_Wrap(prop.unpack(self->cdata));
5460:   END_HANDLE_TH_ERRORS
5461: }
5462: 
5463: static PyObject* THPScaledDotProductFlashAttentionBackward0_key_raw_getter(THPCppFunction *self, void *_unused) {
5464:   HANDLE_TH_ERRORS
5465:   const auto& prop = static_cast<ScaledDotProductFlashAttentionBackward0*>(self->cdata.get())->key_;
5466:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5467:   return obj.release().ptr();
5468:   END_HANDLE_TH_ERRORS
5469: }
5470: 
5471: static PyObject* THPScaledDotProductFlashAttentionBackward0_query_getter(THPCppFunction *self, void *_unused) {
5472:   HANDLE_TH_ERRORS
5473:   const auto& prop = static_cast<ScaledDotProductFlashAttentionBackward0*>(self->cdata.get())->query_;
5474:   return THPVariable_Wrap(prop.unpack(self->cdata));
5475:   END_HANDLE_TH_ERRORS
5476: }
5477: 
5478: static PyObject* THPScaledDotProductFlashAttentionBackward0_query_raw_getter(THPCppFunction *self, void *_unused) {
5479:   HANDLE_TH_ERRORS
5480:   const auto& prop = static_cast<ScaledDotProductFlashAttentionBackward0*>(self->cdata.get())->query_;
5481:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5482:   return obj.release().ptr();
5483:   END_HANDLE_TH_ERRORS
5484: }
5485: 
5486: static PyObject* THPScaledDotProductFlashAttentionBackward0_scale_getter(THPCppFunction *self, void *_unused) {
5487:   HANDLE_TH_ERRORS
5488:   auto opt_prop = static_cast<ScaledDotProductFlashAttentionBackward0*>(self->cdata.get())->scale;
5489:   if (!opt_prop.has_value()) {
5490:     Py_RETURN_NONE;
5491:   }
5492:   auto prop = opt_prop.value();
5493:   return PyFloat_FromDouble((double) prop);
5494:   END_HANDLE_TH_ERRORS
5495: }
5496: 
5497: static PyObject* THPScaledDotProductFlashAttentionBackward0_value_getter(THPCppFunction *self, void *_unused) {
5498:   HANDLE_TH_ERRORS
5499:   const auto& prop = static_cast<ScaledDotProductFlashAttentionBackward0*>(self->cdata.get())->value_;
5500:   return THPVariable_Wrap(prop.unpack(self->cdata));
5501:   END_HANDLE_TH_ERRORS
5502: }
5503: 
5504: static PyObject* THPScaledDotProductFlashAttentionBackward0_value_raw_getter(THPCppFunction *self, void *_unused) {
5505:   HANDLE_TH_ERRORS
5506:   const auto& prop = static_cast<ScaledDotProductFlashAttentionBackward0*>(self->cdata.get())->value_;
5507:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5508:   return obj.release().ptr();
5509:   END_HANDLE_TH_ERRORS
5510: }
5511: 
5512: static PyObject* THPScaledDotProductFlashAttentionBackward0_cum_seq_k_getter(THPCppFunction *self, void *_unused) {
5513:   HANDLE_TH_ERRORS
5514:   const auto& prop = static_cast<ScaledDotProductFlashAttentionBackward0*>(self->cdata.get())->cum_seq_k_;
5515:   return THPVariable_Wrap(prop.unpack(self->cdata));
5516:   END_HANDLE_TH_ERRORS
5517: }
5518: 
5519: static PyObject* THPScaledDotProductFlashAttentionBackward0_cum_seq_k_raw_getter(THPCppFunction *self, void *_unused) {
5520:   HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `PyFloat_FromDouble`, `THPScaledDotProductFlashAttentionBackward0_is_causal_getter`, `THPScaledDotProductFlashAttentionBackward0_key_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyFloat_FromDouble`, `THPScaledDotProductFlashAttentionBackward0_is_causal_getter`, `THPScaledDotProductFlashAttentionBackward0_key_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5521-5600

```cpp
5521:   const auto& prop = static_cast<ScaledDotProductFlashAttentionBackward0*>(self->cdata.get())->cum_seq_k_;
5522:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5523:   return obj.release().ptr();
5524:   END_HANDLE_TH_ERRORS
5525: }
5526: 
5527: static PyObject* THPScaledDotProductFlashAttentionBackward0_cum_seq_q_getter(THPCppFunction *self, void *_unused) {
5528:   HANDLE_TH_ERRORS
5529:   const auto& prop = static_cast<ScaledDotProductFlashAttentionBackward0*>(self->cdata.get())->cum_seq_q_;
5530:   return THPVariable_Wrap(prop.unpack(self->cdata));
5531:   END_HANDLE_TH_ERRORS
5532: }
5533: 
5534: static PyObject* THPScaledDotProductFlashAttentionBackward0_cum_seq_q_raw_getter(THPCppFunction *self, void *_unused) {
5535:   HANDLE_TH_ERRORS
5536:   const auto& prop = static_cast<ScaledDotProductFlashAttentionBackward0*>(self->cdata.get())->cum_seq_q_;
5537:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5538:   return obj.release().ptr();
5539:   END_HANDLE_TH_ERRORS
5540: }
5541: 
5542: static PyObject* THPScaledDotProductFlashAttentionBackward0_logsumexp_getter(THPCppFunction *self, void *_unused) {
5543:   HANDLE_TH_ERRORS
5544:   const auto& prop = static_cast<ScaledDotProductFlashAttentionBackward0*>(self->cdata.get())->logsumexp_;
5545:   return THPVariable_Wrap(prop.unpack(self->cdata));
5546:   END_HANDLE_TH_ERRORS
5547: }
5548: 
5549: static PyObject* THPScaledDotProductFlashAttentionBackward0_logsumexp_raw_getter(THPCppFunction *self, void *_unused) {
5550:   HANDLE_TH_ERRORS
5551:   const auto& prop = static_cast<ScaledDotProductFlashAttentionBackward0*>(self->cdata.get())->logsumexp_;
5552:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5553:   return obj.release().ptr();
5554:   END_HANDLE_TH_ERRORS
5555: }
5556: 
5557: static PyObject* THPScaledDotProductFlashAttentionBackward0_max_k_getter(THPCppFunction *self, void *_unused) {
5558:   HANDLE_TH_ERRORS
5559:   auto prop = static_cast<ScaledDotProductFlashAttentionBackward0*>(self->cdata.get())->max_k;
5560:   if (auto m = prop.maybe_as_int()) {
5561:     return PyLong_FromUnsignedLong(*m);
5562:   } else {
5563:     return py::cast(prop).release().ptr();
5564:   }
5565:   END_HANDLE_TH_ERRORS
5566: }
5567: 
5568: static PyObject* THPScaledDotProductFlashAttentionBackward0_max_q_getter(THPCppFunction *self, void *_unused) {
5569:   HANDLE_TH_ERRORS
5570:   auto prop = static_cast<ScaledDotProductFlashAttentionBackward0*>(self->cdata.get())->max_q;
5571:   if (auto m = prop.maybe_as_int()) {
5572:     return PyLong_FromUnsignedLong(*m);
5573:   } else {
5574:     return py::cast(prop).release().ptr();
5575:   }
5576:   END_HANDLE_TH_ERRORS
5577: }
5578: 
5579: static PyObject* THPScaledDotProductFlashAttentionBackward0_output_getter(THPCppFunction *self, void *_unused) {
5580:   HANDLE_TH_ERRORS
5581:   const auto& prop = static_cast<ScaledDotProductFlashAttentionBackward0*>(self->cdata.get())->output_;
5582:   return THPVariable_Wrap(prop.unpack(self->cdata));
5583:   END_HANDLE_TH_ERRORS
5584: }
5585: 
5586: static PyObject* THPScaledDotProductFlashAttentionBackward0_output_raw_getter(THPCppFunction *self, void *_unused) {
5587:   HANDLE_TH_ERRORS
5588:   const auto& prop = static_cast<ScaledDotProductFlashAttentionBackward0*>(self->cdata.get())->output_;
5589:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5590:   return obj.release().ptr();
5591:   END_HANDLE_TH_ERRORS
5592: }
5593: 
5594: static PyObject* THPScaledDotProductFlashAttentionBackward0_rng_state_getter(THPCppFunction *self, void *_unused) {
5595:   HANDLE_TH_ERRORS
5596:   const auto& prop = static_cast<ScaledDotProductFlashAttentionBackward0*>(self->cdata.get())->rng_state_;
5597:   return THPVariable_Wrap(prop.unpack(self->cdata));
5598:   END_HANDLE_TH_ERRORS
5599: }
5600: 
```

- EN: The main execution path in this span is carried by `cast`, `THPScaledDotProductFlashAttentionBackward0_cum_seq_q_getter`, `THPVariable_Wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `cast`, `THPScaledDotProductFlashAttentionBackward0_cum_seq_q_getter`, `THPVariable_Wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5601-5680

```cpp
5601: static PyObject* THPScaledDotProductFlashAttentionBackward0_rng_state_raw_getter(THPCppFunction *self, void *_unused) {
5602:   HANDLE_TH_ERRORS
5603:   const auto& prop = static_cast<ScaledDotProductFlashAttentionBackward0*>(self->cdata.get())->rng_state_;
5604:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5605:   return obj.release().ptr();
5606:   END_HANDLE_TH_ERRORS
5607: }
5608: 
5609: static PyObject* THPScaledDotProductFlashAttentionBackward0_unused_getter(THPCppFunction *self, void *_unused) {
5610:   HANDLE_TH_ERRORS
5611:   const auto& prop = static_cast<ScaledDotProductFlashAttentionBackward0*>(self->cdata.get())->unused_;
5612:   return THPVariable_Wrap(prop.unpack(self->cdata));
5613:   END_HANDLE_TH_ERRORS
5614: }
5615: 
5616: static PyObject* THPScaledDotProductFlashAttentionBackward0_unused_raw_getter(THPCppFunction *self, void *_unused) {
5617:   HANDLE_TH_ERRORS
5618:   const auto& prop = static_cast<ScaledDotProductFlashAttentionBackward0*>(self->cdata.get())->unused_;
5619:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5620:   return obj.release().ptr();
5621:   END_HANDLE_TH_ERRORS
5622: }
5623: 
5624: static struct PyGetSetDef ScaledDotProductFlashAttentionBackward0_properties[] = {
5625:   THP_FUNCTION_DEFAULT_PROPERTIES,
5626:   {(char*)"_saved_dropout_p", (getter)THPScaledDotProductFlashAttentionBackward0_dropout_p_getter, nullptr, nullptr, nullptr},
5627:   {(char*)"_saved_is_causal", (getter)THPScaledDotProductFlashAttentionBackward0_is_causal_getter, nullptr, nullptr, nullptr},
5628:   {(char*)"_saved_key", (getter)THPScaledDotProductFlashAttentionBackward0_key_getter, nullptr, nullptr, nullptr},
5629:   {(char*)"_raw_saved_key", (getter)THPScaledDotProductFlashAttentionBackward0_key_raw_getter, nullptr, nullptr, nullptr},
5630:   {(char*)"_saved_query", (getter)THPScaledDotProductFlashAttentionBackward0_query_getter, nullptr, nullptr, nullptr},
5631:   {(char*)"_raw_saved_query", (getter)THPScaledDotProductFlashAttentionBackward0_query_raw_getter, nullptr, nullptr, nullptr},
5632:   {(char*)"_saved_scale", (getter)THPScaledDotProductFlashAttentionBackward0_scale_getter, nullptr, nullptr, nullptr},
5633:   {(char*)"_saved_value", (getter)THPScaledDotProductFlashAttentionBackward0_value_getter, nullptr, nullptr, nullptr},
5634:   {(char*)"_raw_saved_value", (getter)THPScaledDotProductFlashAttentionBackward0_value_raw_getter, nullptr, nullptr, nullptr},
5635:   {(char*)"_saved_cum_seq_k", (getter)THPScaledDotProductFlashAttentionBackward0_cum_seq_k_getter, nullptr, nullptr, nullptr},
5636:   {(char*)"_raw_saved_cum_seq_k", (getter)THPScaledDotProductFlashAttentionBackward0_cum_seq_k_raw_getter, nullptr, nullptr, nullptr},
5637:   {(char*)"_saved_cum_seq_q", (getter)THPScaledDotProductFlashAttentionBackward0_cum_seq_q_getter, nullptr, nullptr, nullptr},
5638:   {(char*)"_raw_saved_cum_seq_q", (getter)THPScaledDotProductFlashAttentionBackward0_cum_seq_q_raw_getter, nullptr, nullptr, nullptr},
5639:   {(char*)"_saved_logsumexp", (getter)THPScaledDotProductFlashAttentionBackward0_logsumexp_getter, nullptr, nullptr, nullptr},
5640:   {(char*)"_raw_saved_logsumexp", (getter)THPScaledDotProductFlashAttentionBackward0_logsumexp_raw_getter, nullptr, nullptr, nullptr},
5641:   {(char*)"_saved_max_k", (getter)THPScaledDotProductFlashAttentionBackward0_max_k_getter, nullptr, nullptr, nullptr},
5642:   {(char*)"_saved_max_q", (getter)THPScaledDotProductFlashAttentionBackward0_max_q_getter, nullptr, nullptr, nullptr},
5643:   {(char*)"_saved_output", (getter)THPScaledDotProductFlashAttentionBackward0_output_getter, nullptr, nullptr, nullptr},
5644:   {(char*)"_raw_saved_output", (getter)THPScaledDotProductFlashAttentionBackward0_output_raw_getter, nullptr, nullptr, nullptr},
5645:   {(char*)"_saved_rng_state", (getter)THPScaledDotProductFlashAttentionBackward0_rng_state_getter, nullptr, nullptr, nullptr},
5646:   {(char*)"_raw_saved_rng_state", (getter)THPScaledDotProductFlashAttentionBackward0_rng_state_raw_getter, nullptr, nullptr, nullptr},
5647:   {(char*)"_saved_unused", (getter)THPScaledDotProductFlashAttentionBackward0_unused_getter, nullptr, nullptr, nullptr},
5648:   {(char*)"_raw_saved_unused", (getter)THPScaledDotProductFlashAttentionBackward0_unused_raw_getter, nullptr, nullptr, nullptr},
5649:   {nullptr} /* sentinel */
5650: };
5651: 
5652: static PyObject* THPScaledDotProductFlashAttentionForCpuBackward0_attn_mask_getter(THPCppFunction *self, void *_unused) {
5653:   HANDLE_TH_ERRORS
5654:   const auto& prop = static_cast<ScaledDotProductFlashAttentionForCpuBackward0*>(self->cdata.get())->attn_mask_;
5655:   return THPVariable_Wrap(prop.unpack(self->cdata));
5656:   END_HANDLE_TH_ERRORS
5657: }
5658: 
5659: static PyObject* THPScaledDotProductFlashAttentionForCpuBackward0_attn_mask_raw_getter(THPCppFunction *self, void *_unused) {
5660:   HANDLE_TH_ERRORS
5661:   const auto& prop = static_cast<ScaledDotProductFlashAttentionForCpuBackward0*>(self->cdata.get())->attn_mask_;
5662:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5663:   return obj.release().ptr();
5664:   END_HANDLE_TH_ERRORS
5665: }
5666: 
5667: static PyObject* THPScaledDotProductFlashAttentionForCpuBackward0_dropout_p_getter(THPCppFunction *self, void *_unused) {
5668:   HANDLE_TH_ERRORS
5669:   auto prop = static_cast<ScaledDotProductFlashAttentionForCpuBackward0*>(self->cdata.get())->dropout_p;
5670:   return PyFloat_FromDouble((double) prop);
5671:   END_HANDLE_TH_ERRORS
5672: }
5673: 
5674: static PyObject* THPScaledDotProductFlashAttentionForCpuBackward0_is_causal_getter(THPCppFunction *self, void *_unused) {
5675:   HANDLE_TH_ERRORS
5676:   auto prop = static_cast<ScaledDotProductFlashAttentionForCpuBackward0*>(self->cdata.get())->is_causal;
5677:   if (prop) {
5678:     Py_RETURN_TRUE;
5679:   } else {
5680:     Py_RETURN_FALSE;
```

- EN: The main execution path in this span is carried by `THPScaledDotProductFlashAttentionBackward0_rng_state_raw_getter`, `cast`, `THPScaledDotProductFlashAttentionBackward0_unused_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPScaledDotProductFlashAttentionBackward0_rng_state_raw_getter`, `cast`, `THPScaledDotProductFlashAttentionBackward0_unused_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5681-5760

```cpp
5681:   }
5682:   END_HANDLE_TH_ERRORS
5683: }
5684: 
5685: static PyObject* THPScaledDotProductFlashAttentionForCpuBackward0_key_getter(THPCppFunction *self, void *_unused) {
5686:   HANDLE_TH_ERRORS
5687:   const auto& prop = static_cast<ScaledDotProductFlashAttentionForCpuBackward0*>(self->cdata.get())->key_;
5688:   return THPVariable_Wrap(prop.unpack(self->cdata));
5689:   END_HANDLE_TH_ERRORS
5690: }
5691: 
5692: static PyObject* THPScaledDotProductFlashAttentionForCpuBackward0_key_raw_getter(THPCppFunction *self, void *_unused) {
5693:   HANDLE_TH_ERRORS
5694:   const auto& prop = static_cast<ScaledDotProductFlashAttentionForCpuBackward0*>(self->cdata.get())->key_;
5695:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5696:   return obj.release().ptr();
5697:   END_HANDLE_TH_ERRORS
5698: }
5699: 
5700: static PyObject* THPScaledDotProductFlashAttentionForCpuBackward0_query_getter(THPCppFunction *self, void *_unused) {
5701:   HANDLE_TH_ERRORS
5702:   const auto& prop = static_cast<ScaledDotProductFlashAttentionForCpuBackward0*>(self->cdata.get())->query_;
5703:   return THPVariable_Wrap(prop.unpack(self->cdata));
5704:   END_HANDLE_TH_ERRORS
5705: }
5706: 
5707: static PyObject* THPScaledDotProductFlashAttentionForCpuBackward0_query_raw_getter(THPCppFunction *self, void *_unused) {
5708:   HANDLE_TH_ERRORS
5709:   const auto& prop = static_cast<ScaledDotProductFlashAttentionForCpuBackward0*>(self->cdata.get())->query_;
5710:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5711:   return obj.release().ptr();
5712:   END_HANDLE_TH_ERRORS
5713: }
5714: 
5715: static PyObject* THPScaledDotProductFlashAttentionForCpuBackward0_scale_getter(THPCppFunction *self, void *_unused) {
5716:   HANDLE_TH_ERRORS
5717:   auto opt_prop = static_cast<ScaledDotProductFlashAttentionForCpuBackward0*>(self->cdata.get())->scale;
5718:   if (!opt_prop.has_value()) {
5719:     Py_RETURN_NONE;
5720:   }
5721:   auto prop = opt_prop.value();
5722:   return PyFloat_FromDouble((double) prop);
5723:   END_HANDLE_TH_ERRORS
5724: }
5725: 
5726: static PyObject* THPScaledDotProductFlashAttentionForCpuBackward0_value_getter(THPCppFunction *self, void *_unused) {
5727:   HANDLE_TH_ERRORS
5728:   const auto& prop = static_cast<ScaledDotProductFlashAttentionForCpuBackward0*>(self->cdata.get())->value_;
5729:   return THPVariable_Wrap(prop.unpack(self->cdata));
5730:   END_HANDLE_TH_ERRORS
5731: }
5732: 
5733: static PyObject* THPScaledDotProductFlashAttentionForCpuBackward0_value_raw_getter(THPCppFunction *self, void *_unused) {
5734:   HANDLE_TH_ERRORS
5735:   const auto& prop = static_cast<ScaledDotProductFlashAttentionForCpuBackward0*>(self->cdata.get())->value_;
5736:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5737:   return obj.release().ptr();
5738:   END_HANDLE_TH_ERRORS
5739: }
5740: 
5741: static PyObject* THPScaledDotProductFlashAttentionForCpuBackward0_logsumexp_getter(THPCppFunction *self, void *_unused) {
5742:   HANDLE_TH_ERRORS
5743:   const auto& prop = static_cast<ScaledDotProductFlashAttentionForCpuBackward0*>(self->cdata.get())->logsumexp_;
5744:   return THPVariable_Wrap(prop.unpack(self->cdata));
5745:   END_HANDLE_TH_ERRORS
5746: }
5747: 
5748: static PyObject* THPScaledDotProductFlashAttentionForCpuBackward0_logsumexp_raw_getter(THPCppFunction *self, void *_unused) {
5749:   HANDLE_TH_ERRORS
5750:   const auto& prop = static_cast<ScaledDotProductFlashAttentionForCpuBackward0*>(self->cdata.get())->logsumexp_;
5751:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5752:   return obj.release().ptr();
5753:   END_HANDLE_TH_ERRORS
5754: }
5755: 
5756: static PyObject* THPScaledDotProductFlashAttentionForCpuBackward0_output_getter(THPCppFunction *self, void *_unused) {
5757:   HANDLE_TH_ERRORS
5758:   const auto& prop = static_cast<ScaledDotProductFlashAttentionForCpuBackward0*>(self->cdata.get())->output_;
5759:   return THPVariable_Wrap(prop.unpack(self->cdata));
5760:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPScaledDotProductFlashAttentionForCpuBackward0_key_getter`, `THPVariable_Wrap`, `THPScaledDotProductFlashAttentionForCpuBackward0_key_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPScaledDotProductFlashAttentionForCpuBackward0_key_getter`, `THPVariable_Wrap`, `THPScaledDotProductFlashAttentionForCpuBackward0_key_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5761-5840

```cpp
5761: }
5762: 
5763: static PyObject* THPScaledDotProductFlashAttentionForCpuBackward0_output_raw_getter(THPCppFunction *self, void *_unused) {
5764:   HANDLE_TH_ERRORS
5765:   const auto& prop = static_cast<ScaledDotProductFlashAttentionForCpuBackward0*>(self->cdata.get())->output_;
5766:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5767:   return obj.release().ptr();
5768:   END_HANDLE_TH_ERRORS
5769: }
5770: 
5771: static struct PyGetSetDef ScaledDotProductFlashAttentionForCpuBackward0_properties[] = {
5772:   THP_FUNCTION_DEFAULT_PROPERTIES,
5773:   {(char*)"_saved_attn_mask", (getter)THPScaledDotProductFlashAttentionForCpuBackward0_attn_mask_getter, nullptr, nullptr, nullptr},
5774:   {(char*)"_raw_saved_attn_mask", (getter)THPScaledDotProductFlashAttentionForCpuBackward0_attn_mask_raw_getter, nullptr, nullptr, nullptr},
5775:   {(char*)"_saved_dropout_p", (getter)THPScaledDotProductFlashAttentionForCpuBackward0_dropout_p_getter, nullptr, nullptr, nullptr},
5776:   {(char*)"_saved_is_causal", (getter)THPScaledDotProductFlashAttentionForCpuBackward0_is_causal_getter, nullptr, nullptr, nullptr},
5777:   {(char*)"_saved_key", (getter)THPScaledDotProductFlashAttentionForCpuBackward0_key_getter, nullptr, nullptr, nullptr},
5778:   {(char*)"_raw_saved_key", (getter)THPScaledDotProductFlashAttentionForCpuBackward0_key_raw_getter, nullptr, nullptr, nullptr},
5779:   {(char*)"_saved_query", (getter)THPScaledDotProductFlashAttentionForCpuBackward0_query_getter, nullptr, nullptr, nullptr},
5780:   {(char*)"_raw_saved_query", (getter)THPScaledDotProductFlashAttentionForCpuBackward0_query_raw_getter, nullptr, nullptr, nullptr},
5781:   {(char*)"_saved_scale", (getter)THPScaledDotProductFlashAttentionForCpuBackward0_scale_getter, nullptr, nullptr, nullptr},
5782:   {(char*)"_saved_value", (getter)THPScaledDotProductFlashAttentionForCpuBackward0_value_getter, nullptr, nullptr, nullptr},
5783:   {(char*)"_raw_saved_value", (getter)THPScaledDotProductFlashAttentionForCpuBackward0_value_raw_getter, nullptr, nullptr, nullptr},
5784:   {(char*)"_saved_logsumexp", (getter)THPScaledDotProductFlashAttentionForCpuBackward0_logsumexp_getter, nullptr, nullptr, nullptr},
5785:   {(char*)"_raw_saved_logsumexp", (getter)THPScaledDotProductFlashAttentionForCpuBackward0_logsumexp_raw_getter, nullptr, nullptr, nullptr},
5786:   {(char*)"_saved_output", (getter)THPScaledDotProductFlashAttentionForCpuBackward0_output_getter, nullptr, nullptr, nullptr},
5787:   {(char*)"_raw_saved_output", (getter)THPScaledDotProductFlashAttentionForCpuBackward0_output_raw_getter, nullptr, nullptr, nullptr},
5788:   {nullptr} /* sentinel */
5789: };
5790: 
5791: static PyObject* THPFftC2CBackward0_dim_getter(THPCppFunction *self, void *_unused) {
5792:   HANDLE_TH_ERRORS
5793:   auto prop = static_cast<FftC2CBackward0*>(self->cdata.get())->dim;
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
5808: static PyObject* THPFftC2CBackward0_forward_getter(THPCppFunction *self, void *_unused) {
5809:   HANDLE_TH_ERRORS
5810:   auto prop = static_cast<FftC2CBackward0*>(self->cdata.get())->forward;
5811:   if (prop) {
5812:     Py_RETURN_TRUE;
5813:   } else {
5814:     Py_RETURN_FALSE;
5815:   }
5816:   END_HANDLE_TH_ERRORS
5817: }
5818: 
5819: static PyObject* THPFftC2CBackward0_normalization_getter(THPCppFunction *self, void *_unused) {
5820:   HANDLE_TH_ERRORS
5821:   auto prop = static_cast<FftC2CBackward0*>(self->cdata.get())->normalization;
5822:   return PyLong_FromUnsignedLong((int64_t) prop);
5823:   END_HANDLE_TH_ERRORS
5824: }
5825: 
5826: static struct PyGetSetDef FftC2CBackward0_properties[] = {
5827:   THP_FUNCTION_DEFAULT_PROPERTIES,
5828:   {(char*)"_saved_dim", (getter)THPFftC2CBackward0_dim_getter, nullptr, nullptr, nullptr},
5829:   {(char*)"_saved_forward", (getter)THPFftC2CBackward0_forward_getter, nullptr, nullptr, nullptr},
5830:   {(char*)"_saved_normalization", (getter)THPFftC2CBackward0_normalization_getter, nullptr, nullptr, nullptr},
5831:   {nullptr} /* sentinel */
5832: };
5833: 
5834: static PyObject* THPStackBackward0_dim_getter(THPCppFunction *self, void *_unused) {
5835:   HANDLE_TH_ERRORS
5836:   auto prop = static_cast<StackBackward0*>(self->cdata.get())->dim;
5837:   return PyLong_FromUnsignedLong((int64_t) prop);
5838:   END_HANDLE_TH_ERRORS
5839: }
5840: 
```

- EN: The main execution path in this span is carried by `THPScaledDotProductFlashAttentionForCpuBackward0_output_raw_getter`, `cast`, `THPFftC2CBackward0_dim_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPScaledDotProductFlashAttentionForCpuBackward0_output_raw_getter`, `cast`, `THPFftC2CBackward0_dim_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5841-5920

```cpp
5841: static struct PyGetSetDef StackBackward0_properties[] = {
5842:   THP_FUNCTION_DEFAULT_PROPERTIES,
5843:   {(char*)"_saved_dim", (getter)THPStackBackward0_dim_getter, nullptr, nullptr, nullptr},
5844:   {nullptr} /* sentinel */
5845: };
5846: 
5847: static PyObject* THPThnnFusedLstmCellBackward0_cx_getter(THPCppFunction *self, void *_unused) {
5848:   HANDLE_TH_ERRORS
5849:   const auto& prop = static_cast<ThnnFusedLstmCellBackward0*>(self->cdata.get())->cx_;
5850:   return THPVariable_Wrap(prop.unpack(self->cdata));
5851:   END_HANDLE_TH_ERRORS
5852: }
5853: 
5854: static PyObject* THPThnnFusedLstmCellBackward0_cx_raw_getter(THPCppFunction *self, void *_unused) {
5855:   HANDLE_TH_ERRORS
5856:   const auto& prop = static_cast<ThnnFusedLstmCellBackward0*>(self->cdata.get())->cx_;
5857:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5858:   return obj.release().ptr();
5859:   END_HANDLE_TH_ERRORS
5860: }
5861: 
5862: static PyObject* THPThnnFusedLstmCellBackward0_hidden_bias_getter(THPCppFunction *self, void *_unused) {
5863:   HANDLE_TH_ERRORS
5864:   const auto& prop = static_cast<ThnnFusedLstmCellBackward0*>(self->cdata.get())->hidden_bias_;
5865:   return THPVariable_Wrap(prop.unpack(self->cdata));
5866:   END_HANDLE_TH_ERRORS
5867: }
5868: 
5869: static PyObject* THPThnnFusedLstmCellBackward0_hidden_bias_raw_getter(THPCppFunction *self, void *_unused) {
5870:   HANDLE_TH_ERRORS
5871:   const auto& prop = static_cast<ThnnFusedLstmCellBackward0*>(self->cdata.get())->hidden_bias_;
5872:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5873:   return obj.release().ptr();
5874:   END_HANDLE_TH_ERRORS
5875: }
5876: 
5877: static PyObject* THPThnnFusedLstmCellBackward0_hidden_gates_getter(THPCppFunction *self, void *_unused) {
5878:   HANDLE_TH_ERRORS
5879:   const auto& prop = static_cast<ThnnFusedLstmCellBackward0*>(self->cdata.get())->hidden_gates_;
5880:   return THPVariable_Wrap(prop.unpack(self->cdata));
5881:   END_HANDLE_TH_ERRORS
5882: }
5883: 
5884: static PyObject* THPThnnFusedLstmCellBackward0_hidden_gates_raw_getter(THPCppFunction *self, void *_unused) {
5885:   HANDLE_TH_ERRORS
5886:   const auto& prop = static_cast<ThnnFusedLstmCellBackward0*>(self->cdata.get())->hidden_gates_;
5887:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5888:   return obj.release().ptr();
5889:   END_HANDLE_TH_ERRORS
5890: }
5891: 
5892: static PyObject* THPThnnFusedLstmCellBackward0_input_bias_getter(THPCppFunction *self, void *_unused) {
5893:   HANDLE_TH_ERRORS
5894:   const auto& prop = static_cast<ThnnFusedLstmCellBackward0*>(self->cdata.get())->input_bias_;
5895:   return THPVariable_Wrap(prop.unpack(self->cdata));
5896:   END_HANDLE_TH_ERRORS
5897: }
5898: 
5899: static PyObject* THPThnnFusedLstmCellBackward0_input_bias_raw_getter(THPCppFunction *self, void *_unused) {
5900:   HANDLE_TH_ERRORS
5901:   const auto& prop = static_cast<ThnnFusedLstmCellBackward0*>(self->cdata.get())->input_bias_;
5902:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5903:   return obj.release().ptr();
5904:   END_HANDLE_TH_ERRORS
5905: }
5906: 
5907: static PyObject* THPThnnFusedLstmCellBackward0_input_gates_getter(THPCppFunction *self, void *_unused) {
5908:   HANDLE_TH_ERRORS
5909:   const auto& prop = static_cast<ThnnFusedLstmCellBackward0*>(self->cdata.get())->input_gates_;
5910:   return THPVariable_Wrap(prop.unpack(self->cdata));
5911:   END_HANDLE_TH_ERRORS
5912: }
5913: 
5914: static PyObject* THPThnnFusedLstmCellBackward0_input_gates_raw_getter(THPCppFunction *self, void *_unused) {
5915:   HANDLE_TH_ERRORS
5916:   const auto& prop = static_cast<ThnnFusedLstmCellBackward0*>(self->cdata.get())->input_gates_;
5917:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5918:   return obj.release().ptr();
5919:   END_HANDLE_TH_ERRORS
5920: }
```

- EN: The main execution path in this span is carried by `THPThnnFusedLstmCellBackward0_cx_getter`, `THPVariable_Wrap`, `THPThnnFusedLstmCellBackward0_cx_raw_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPThnnFusedLstmCellBackward0_cx_getter`, `THPVariable_Wrap`, `THPThnnFusedLstmCellBackward0_cx_raw_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5921-6000

```cpp
5921: 
5922: static PyObject* THPThnnFusedLstmCellBackward0_result1_getter(THPCppFunction *self, void *_unused) {
5923:   HANDLE_TH_ERRORS
5924:   const auto& prop = static_cast<ThnnFusedLstmCellBackward0*>(self->cdata.get())->result1_;
5925:   return THPVariable_Wrap(prop.unpack(self->cdata));
5926:   END_HANDLE_TH_ERRORS
5927: }
5928: 
5929: static PyObject* THPThnnFusedLstmCellBackward0_result1_raw_getter(THPCppFunction *self, void *_unused) {
5930:   HANDLE_TH_ERRORS
5931:   const auto& prop = static_cast<ThnnFusedLstmCellBackward0*>(self->cdata.get())->result1_;
5932:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5933:   return obj.release().ptr();
5934:   END_HANDLE_TH_ERRORS
5935: }
5936: 
5937: static PyObject* THPThnnFusedLstmCellBackward0_result2_getter(THPCppFunction *self, void *_unused) {
5938:   HANDLE_TH_ERRORS
5939:   const auto& prop = static_cast<ThnnFusedLstmCellBackward0*>(self->cdata.get())->result2_;
5940:   return THPVariable_Wrap(prop.unpack(self->cdata));
5941:   END_HANDLE_TH_ERRORS
5942: }
5943: 
5944: static PyObject* THPThnnFusedLstmCellBackward0_result2_raw_getter(THPCppFunction *self, void *_unused) {
5945:   HANDLE_TH_ERRORS
5946:   const auto& prop = static_cast<ThnnFusedLstmCellBackward0*>(self->cdata.get())->result2_;
5947:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5948:   return obj.release().ptr();
5949:   END_HANDLE_TH_ERRORS
5950: }
5951: 
5952: static struct PyGetSetDef ThnnFusedLstmCellBackward0_properties[] = {
5953:   THP_FUNCTION_DEFAULT_PROPERTIES,
5954:   {(char*)"_saved_cx", (getter)THPThnnFusedLstmCellBackward0_cx_getter, nullptr, nullptr, nullptr},
5955:   {(char*)"_raw_saved_cx", (getter)THPThnnFusedLstmCellBackward0_cx_raw_getter, nullptr, nullptr, nullptr},
5956:   {(char*)"_saved_hidden_bias", (getter)THPThnnFusedLstmCellBackward0_hidden_bias_getter, nullptr, nullptr, nullptr},
5957:   {(char*)"_raw_saved_hidden_bias", (getter)THPThnnFusedLstmCellBackward0_hidden_bias_raw_getter, nullptr, nullptr, nullptr},
5958:   {(char*)"_saved_hidden_gates", (getter)THPThnnFusedLstmCellBackward0_hidden_gates_getter, nullptr, nullptr, nullptr},
5959:   {(char*)"_raw_saved_hidden_gates", (getter)THPThnnFusedLstmCellBackward0_hidden_gates_raw_getter, nullptr, nullptr, nullptr},
5960:   {(char*)"_saved_input_bias", (getter)THPThnnFusedLstmCellBackward0_input_bias_getter, nullptr, nullptr, nullptr},
5961:   {(char*)"_raw_saved_input_bias", (getter)THPThnnFusedLstmCellBackward0_input_bias_raw_getter, nullptr, nullptr, nullptr},
5962:   {(char*)"_saved_input_gates", (getter)THPThnnFusedLstmCellBackward0_input_gates_getter, nullptr, nullptr, nullptr},
5963:   {(char*)"_raw_saved_input_gates", (getter)THPThnnFusedLstmCellBackward0_input_gates_raw_getter, nullptr, nullptr, nullptr},
5964:   {(char*)"_saved_result1", (getter)THPThnnFusedLstmCellBackward0_result1_getter, nullptr, nullptr, nullptr},
5965:   {(char*)"_raw_saved_result1", (getter)THPThnnFusedLstmCellBackward0_result1_raw_getter, nullptr, nullptr, nullptr},
5966:   {(char*)"_saved_result2", (getter)THPThnnFusedLstmCellBackward0_result2_getter, nullptr, nullptr, nullptr},
5967:   {(char*)"_raw_saved_result2", (getter)THPThnnFusedLstmCellBackward0_result2_raw_getter, nullptr, nullptr, nullptr},
5968:   {nullptr} /* sentinel */
5969: };
5970: 
5971: static PyObject* THPForeachMaximumBackward0_scalar_getter(THPCppFunction *self, void *_unused) {
5972:   HANDLE_TH_ERRORS
5973:   auto prop = static_cast<ForeachMaximumBackward0*>(self->cdata.get())->scalar;
5974:   if (prop.isComplex()) {
5975:     auto cprop = prop.to<c10::complex<double>>();
5976:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
5977:   } else if (prop.isFloatingPoint()) {
5978:     return PyFloat_FromDouble(prop.to<double>());
5979:   } else if (prop.isIntegral(/*includeBool=*/false)) {
5980:     return PyLong_FromLong(prop.to<int64_t>());
5981:   } else if (prop.isBoolean()) {
5982:     if (prop.to<bool>()) {
5983:       Py_RETURN_TRUE;
5984:     } else {
5985:       Py_RETURN_FALSE;
5986:     }
5987:   } else {
5988:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
5989:     return nullptr;
5990:   }
5991:   END_HANDLE_TH_ERRORS
5992: }
5993: 
5994: static PyObject* THPForeachMaximumBackward0_self_getter(THPCppFunction *self, void *_unused) {
5995:   HANDLE_TH_ERRORS
5996:   const auto *node = static_cast<ForeachMaximumBackward0*>(self->cdata.get());
5997:   const auto& prop = node->self_;
5998:   if (node->self_released_) {
5999:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
6000:     return nullptr;
```

- EN: The main execution path in this span is carried by `THPThnnFusedLstmCellBackward0_result1_getter`, `THPVariable_Wrap`, `THPThnnFusedLstmCellBackward0_result1_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPThnnFusedLstmCellBackward0_result1_getter`, `THPVariable_Wrap`, `THPThnnFusedLstmCellBackward0_result1_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 6001-6080

```cpp
6001:   }
6002:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
6003:   for (auto i: c10::irange(prop.size())) {
6004:     PyTuple_SetItem(tup, (Py_ssize_t) i, THPVariable_Wrap(prop[i].unpack(self->cdata)));
6005:   }
6006:   return tup;
6007:   END_HANDLE_TH_ERRORS
6008: }
6009: 
6010: static PyObject* THPForeachMaximumBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
6011:   HANDLE_TH_ERRORS
6012:   const auto *node = static_cast<ForeachMaximumBackward0*>(self->cdata.get());
6013:   const auto& prop = node->self_;
6014:   if (node->self_released_) {
6015:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
6016:     return nullptr;
6017:   }
6018:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
6019:   for (auto i : c10::irange(prop.size())) {
6020:     pybind11::object obj = pybind11::cast(prop[i], pybind11::return_value_policy::reference);
6021:     PyTuple_SetItem(tup, (Py_ssize_t) i, obj.release().ptr());
6022:   }
6023:   return tup;
6024:   END_HANDLE_TH_ERRORS
6025: }
6026: 
6027: static struct PyGetSetDef ForeachMaximumBackward0_properties[] = {
6028:   THP_FUNCTION_DEFAULT_PROPERTIES,
6029:   {(char*)"_saved_scalar", (getter)THPForeachMaximumBackward0_scalar_getter, nullptr, nullptr, nullptr},
6030:   {(char*)"_saved_self", (getter)THPForeachMaximumBackward0_self_getter, nullptr, nullptr, nullptr},
6031:   {(char*)"_raw_saved_self", (getter)THPForeachMaximumBackward0_self_raw_getter, nullptr, nullptr, nullptr},
6032:   {nullptr} /* sentinel */
6033: };
6034: 
6035: static PyObject* THPForeachMaximumBackward1_scalars_getter(THPCppFunction *self, void *_unused) {
6036:   HANDLE_TH_ERRORS
6037:   const auto *node = static_cast<ForeachMaximumBackward1*>(self->cdata.get());
6038:   const auto& prop = node->scalars;
6039:   if (node->scalars_released_) {
6040:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
6041:     return nullptr;
6042:   }
6043:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
6044:   for (auto i: c10::irange(prop.size())) {
6045:     if (prop[i].isComplex()) {
6046:       auto cprop = prop[i].to<c10::complex<double>>();
6047:       PyTuple_SetItem(tup, (Py_ssize_t) i, PyComplex_FromDoubles(cprop.real(), cprop.imag()));
6048:     } else if (prop[i].isFloatingPoint()) {
6049:       auto double_prop = prop[i].to<double>();
6050:       PyTuple_SetItem(tup, (Py_ssize_t) i, PyFloat_FromDouble(double_prop));
6051:     } else if (prop[i].isIntegral(/*includeBool=*/false)) {
6052:       auto long_prop = prop[i].to<int64_t>();
6053:       PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromLong(long_prop));
6054:     } else if (prop[i].isBoolean()) {
6055:       if (prop[i].to<bool>()) {
6056:         PyTuple_SetItem(tup, (Py_ssize_t) i, Py_True);
6057:       } else {
6058:         PyTuple_SetItem(tup, (Py_ssize_t) i, Py_False);
6059:       }
6060:     } else {
6061:       PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
6062:       return nullptr;
6063:     }
6064:   }
6065:   return tup;
6066:   END_HANDLE_TH_ERRORS
6067: }
6068: 
6069: static PyObject* THPForeachMaximumBackward1_self_getter(THPCppFunction *self, void *_unused) {
6070:   HANDLE_TH_ERRORS
6071:   const auto *node = static_cast<ForeachMaximumBackward1*>(self->cdata.get());
6072:   const auto& prop = node->self_;
6073:   if (node->self_released_) {
6074:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
6075:     return nullptr;
6076:   }
6077:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
6078:   for (auto i: c10::irange(prop.size())) {
6079:     PyTuple_SetItem(tup, (Py_ssize_t) i, THPVariable_Wrap(prop[i].unpack(self->cdata)));
6080:   }
```

- EN: The main execution path in this span is carried by `PyTuple_New`, `PyTuple_SetItem`, `THPForeachMaximumBackward0_self_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyTuple_New`, `PyTuple_SetItem`, `THPForeachMaximumBackward0_self_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 6081-6160

```cpp
6081:   return tup;
6082:   END_HANDLE_TH_ERRORS
6083: }
6084: 
6085: static PyObject* THPForeachMaximumBackward1_self_raw_getter(THPCppFunction *self, void *_unused) {
6086:   HANDLE_TH_ERRORS
6087:   const auto *node = static_cast<ForeachMaximumBackward1*>(self->cdata.get());
6088:   const auto& prop = node->self_;
6089:   if (node->self_released_) {
6090:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
6091:     return nullptr;
6092:   }
6093:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
6094:   for (auto i : c10::irange(prop.size())) {
6095:     pybind11::object obj = pybind11::cast(prop[i], pybind11::return_value_policy::reference);
6096:     PyTuple_SetItem(tup, (Py_ssize_t) i, obj.release().ptr());
6097:   }
6098:   return tup;
6099:   END_HANDLE_TH_ERRORS
6100: }
6101: 
6102: static struct PyGetSetDef ForeachMaximumBackward1_properties[] = {
6103:   THP_FUNCTION_DEFAULT_PROPERTIES,
6104:   {(char*)"_saved_scalars", (getter)THPForeachMaximumBackward1_scalars_getter, nullptr, nullptr, nullptr},
6105:   {(char*)"_saved_self", (getter)THPForeachMaximumBackward1_self_getter, nullptr, nullptr, nullptr},
6106:   {(char*)"_raw_saved_self", (getter)THPForeachMaximumBackward1_self_raw_getter, nullptr, nullptr, nullptr},
6107:   {nullptr} /* sentinel */
6108: };
6109: 
6110: 
6111: 
6112: static struct PyGetSetDef AliasBackward0_copy_properties[] = {
6113:   THP_FUNCTION_DEFAULT_PROPERTIES,
6114: 
6115:   {nullptr} /* sentinel */
6116: };
6117: 
6118: static PyObject* THPSplitBackward0_copy_dim_getter(THPCppFunction *self, void *_unused) {
6119:   HANDLE_TH_ERRORS
6120:   auto prop = static_cast<SplitBackward0_copy*>(self->cdata.get())->dim;
6121:   return PyLong_FromUnsignedLong((int64_t) prop);
6122:   END_HANDLE_TH_ERRORS
6123: }
6124: 
6125: static PyObject* THPSplitBackward0_copy_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
6126:   HANDLE_TH_ERRORS
6127:   auto prop = static_cast<SplitBackward0_copy*>(self->cdata.get())->self_sym_sizes;
6128:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
6129:   for (auto i : c10::irange(prop.size())) {
6130:       auto si = prop[i];
6131:       if (auto m = si.maybe_as_int()) {
6132:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
6133:       } else {
6134:         auto py_symint = py::cast(si).release().ptr();
6135:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
6136:       }
6137:   }
6138:   return tup;
6139:   END_HANDLE_TH_ERRORS
6140: }
6141: 
6142: static PyObject* THPSplitBackward0_copy_split_size_getter(THPCppFunction *self, void *_unused) {
6143:   HANDLE_TH_ERRORS
6144:   auto prop = static_cast<SplitBackward0_copy*>(self->cdata.get())->split_size;
6145:   if (auto m = prop.maybe_as_int()) {
6146:     return PyLong_FromUnsignedLong(*m);
6147:   } else {
6148:     return py::cast(prop).release().ptr();
6149:   }
6150:   END_HANDLE_TH_ERRORS
6151: }
6152: 
6153: static struct PyGetSetDef SplitBackward0_copy_properties[] = {
6154:   THP_FUNCTION_DEFAULT_PROPERTIES,
6155:   {(char*)"_saved_dim", (getter)THPSplitBackward0_copy_dim_getter, nullptr, nullptr, nullptr},
6156:   {(char*)"_saved_self_sym_sizes", (getter)THPSplitBackward0_copy_self_sym_sizes_getter, nullptr, nullptr, nullptr},
6157:   {(char*)"_saved_split_size", (getter)THPSplitBackward0_copy_split_size_getter, nullptr, nullptr, nullptr},
6158:   {nullptr} /* sentinel */
6159: };
6160: 
```

- EN: The main execution path in this span is carried by `THPForeachMaximumBackward1_self_raw_getter`, `PyErr_SetString`, `PyTuple_New`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPForeachMaximumBackward1_self_raw_getter`, `PyErr_SetString`, `PyTuple_New` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 6161-6240

```cpp
6161: static PyObject* THPTransposeBackward0_copy_dim0_getter(THPCppFunction *self, void *_unused) {
6162:   HANDLE_TH_ERRORS
6163:   auto prop = static_cast<TransposeBackward0_copy*>(self->cdata.get())->dim0;
6164:   return PyLong_FromUnsignedLong((int64_t) prop);
6165:   END_HANDLE_TH_ERRORS
6166: }
6167: 
6168: static PyObject* THPTransposeBackward0_copy_dim1_getter(THPCppFunction *self, void *_unused) {
6169:   HANDLE_TH_ERRORS
6170:   auto prop = static_cast<TransposeBackward0_copy*>(self->cdata.get())->dim1;
6171:   return PyLong_FromUnsignedLong((int64_t) prop);
6172:   END_HANDLE_TH_ERRORS
6173: }
6174: 
6175: static struct PyGetSetDef TransposeBackward0_copy_properties[] = {
6176:   THP_FUNCTION_DEFAULT_PROPERTIES,
6177:   {(char*)"_saved_dim0", (getter)THPTransposeBackward0_copy_dim0_getter, nullptr, nullptr, nullptr},
6178:   {(char*)"_saved_dim1", (getter)THPTransposeBackward0_copy_dim1_getter, nullptr, nullptr, nullptr},
6179:   {nullptr} /* sentinel */
6180: };
6181: 
6182: 
6183: 
6184: static struct PyGetSetDef LiftFreshBackward0_copy_properties[] = {
6185:   THP_FUNCTION_DEFAULT_PROPERTIES,
6186: 
6187:   {nullptr} /* sentinel */
6188: };
6189: 
6190: 
6191: 
6192: static struct PyGetSetDef ViewAsRealBackward0_copy_properties[] = {
6193:   THP_FUNCTION_DEFAULT_PROPERTIES,
6194: 
6195:   {nullptr} /* sentinel */
6196: };
6197: 
6198: static PyObject* THPNestedGetValuesBackward0_copy_self_getter(THPCppFunction *self, void *_unused) {
6199:   HANDLE_TH_ERRORS
6200:   const auto& prop = static_cast<NestedGetValuesBackward0_copy*>(self->cdata.get())->self_;
6201:   return THPVariable_Wrap(prop.unpack(self->cdata));
6202:   END_HANDLE_TH_ERRORS
6203: }
6204: 
6205: static PyObject* THPNestedGetValuesBackward0_copy_self_raw_getter(THPCppFunction *self, void *_unused) {
6206:   HANDLE_TH_ERRORS
6207:   const auto& prop = static_cast<NestedGetValuesBackward0_copy*>(self->cdata.get())->self_;
6208:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
6209:   return obj.release().ptr();
6210:   END_HANDLE_TH_ERRORS
6211: }
6212: 
6213: static struct PyGetSetDef NestedGetValuesBackward0_copy_properties[] = {
6214:   THP_FUNCTION_DEFAULT_PROPERTIES,
6215:   {(char*)"_saved_self", (getter)THPNestedGetValuesBackward0_copy_self_getter, nullptr, nullptr, nullptr},
6216:   {(char*)"_raw_saved_self", (getter)THPNestedGetValuesBackward0_copy_self_raw_getter, nullptr, nullptr, nullptr},
6217:   {nullptr} /* sentinel */
6218: };
6219: 
6220: static PyObject* THPUnbindBackward0_copy_dim_getter(THPCppFunction *self, void *_unused) {
6221:   HANDLE_TH_ERRORS
6222:   auto prop = static_cast<UnbindBackward0_copy*>(self->cdata.get())->dim;
6223:   return PyLong_FromUnsignedLong((int64_t) prop);
6224:   END_HANDLE_TH_ERRORS
6225: }
6226: 
6227: static struct PyGetSetDef UnbindBackward0_copy_properties[] = {
6228:   THP_FUNCTION_DEFAULT_PROPERTIES,
6229:   {(char*)"_saved_dim", (getter)THPUnbindBackward0_copy_dim_getter, nullptr, nullptr, nullptr},
6230:   {nullptr} /* sentinel */
6231: };
6232: 
6233: static PyObject* THPUnbindBackwardAutogradNestedTensor0_copy_dim_getter(THPCppFunction *self, void *_unused) {
6234:   HANDLE_TH_ERRORS
6235:   auto prop = static_cast<UnbindBackwardAutogradNestedTensor0_copy*>(self->cdata.get())->dim;
6236:   return PyLong_FromUnsignedLong((int64_t) prop);
6237:   END_HANDLE_TH_ERRORS
6238: }
6239: 
6240: static PyObject* THPUnbindBackwardAutogradNestedTensor0_copy_self_getter(THPCppFunction *self, void *_unused) {
```

- EN: The main execution path in this span is carried by `THPTransposeBackward0_copy_dim0_getter`, `PyLong_FromUnsignedLong`, `THPTransposeBackward0_copy_dim1_getter`. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPTransposeBackward0_copy_dim0_getter`, `PyLong_FromUnsignedLong`, `THPTransposeBackward0_copy_dim1_getter` 等函数/方法承载。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 6241-6320

```cpp
6241:   HANDLE_TH_ERRORS
6242:   const auto& prop = static_cast<UnbindBackwardAutogradNestedTensor0_copy*>(self->cdata.get())->self_;
6243:   return THPVariable_Wrap(prop.unpack(self->cdata));
6244:   END_HANDLE_TH_ERRORS
6245: }
6246: 
6247: static PyObject* THPUnbindBackwardAutogradNestedTensor0_copy_self_raw_getter(THPCppFunction *self, void *_unused) {
6248:   HANDLE_TH_ERRORS
6249:   const auto& prop = static_cast<UnbindBackwardAutogradNestedTensor0_copy*>(self->cdata.get())->self_;
6250:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
6251:   return obj.release().ptr();
6252:   END_HANDLE_TH_ERRORS
6253: }
6254: 
6255: static struct PyGetSetDef UnbindBackwardAutogradNestedTensor0_copy_properties[] = {
6256:   THP_FUNCTION_DEFAULT_PROPERTIES,
6257:   {(char*)"_saved_dim", (getter)THPUnbindBackwardAutogradNestedTensor0_copy_dim_getter, nullptr, nullptr, nullptr},
6258:   {(char*)"_saved_self", (getter)THPUnbindBackwardAutogradNestedTensor0_copy_self_getter, nullptr, nullptr, nullptr},
6259:   {(char*)"_raw_saved_self", (getter)THPUnbindBackwardAutogradNestedTensor0_copy_self_raw_getter, nullptr, nullptr, nullptr},
6260:   {nullptr} /* sentinel */
6261: };
6262: 
6263: void initialize_autogenerated_functions_1(PyObject* module) {
6264:   static PyTypeObject AbsBackward0Class;
6265:   addClass<AbsBackward0>(module, AbsBackward0Class, "AbsBackward0", AbsBackward0_properties);
6266:   static PyTypeObject AddmvBackward0Class;
6267:   addClass<AddmvBackward0>(module, AddmvBackward0Class, "AddmvBackward0", AddmvBackward0_properties);
6268:   static PyTypeObject AddrBackward0Class;
6269:   addClass<AddrBackward0>(module, AddrBackward0Class, "AddrBackward0", AddrBackward0_properties);
6270:   static PyTypeObject AliasBackward0Class;
6271:   addClass<AliasBackward0>(module, AliasBackward0Class, "AliasBackward0", AliasBackward0_properties);
6272:   static PyTypeObject AtanBackward0Class;
6273:   addClass<AtanBackward0>(module, AtanBackward0Class, "AtanBackward0", AtanBackward0_properties);
6274:   static PyTypeObject CeilBackward0Class;
6275:   addClass<CeilBackward0>(module, CeilBackward0Class, "CeilBackward0", CeilBackward0_properties);
6276:   static PyTypeObject LinalgCholeskyExBackward0Class;
6277:   addClass<LinalgCholeskyExBackward0>(module, LinalgCholeskyExBackward0Class, "LinalgCholeskyExBackward0", LinalgCholeskyExBackward0_properties);
6278:   static PyTypeObject CholeskySolveBackward0Class;
6279:   addClass<CholeskySolveBackward0>(module, CholeskySolveBackward0Class, "CholeskySolveBackward0", CholeskySolveBackward0_properties);
6280:   static PyTypeObject CloneBackward0Class;
6281:   addClass<CloneBackward0>(module, CloneBackward0Class, "CloneBackward0", CloneBackward0_properties);
6282:   static PyTypeObject CoshBackward0Class;
6283:   addClass<CoshBackward0>(module, CoshBackward0Class, "CoshBackward0", CoshBackward0_properties);
6284:   static PyTypeObject LinalgCrossBackward0Class;
6285:   addClass<LinalgCrossBackward0>(module, LinalgCrossBackward0Class, "LinalgCrossBackward0", LinalgCrossBackward0_properties);
6286:   static PyTypeObject LogcumsumexpBackward0Class;
6287:   addClass<LogcumsumexpBackward0>(module, LogcumsumexpBackward0Class, "LogcumsumexpBackward0", LogcumsumexpBackward0_properties);
6288:   static PyTypeObject CumminBackward0Class;
6289:   addClass<CumminBackward0>(module, CumminBackward0Class, "CumminBackward0", CumminBackward0_properties);
6290:   static PyTypeObject ConvTbcBackward0Class;
6291:   addClass<ConvTbcBackward0>(module, ConvTbcBackward0Class, "ConvTbcBackward0", ConvTbcBackward0_properties);
6292:   static PyTypeObject DivBackward0Class;
6293:   addClass<DivBackward0>(module, DivBackward0Class, "DivBackward0", DivBackward0_properties);
6294:   static PyTypeObject DivBackward1Class;
6295:   addClass<DivBackward1>(module, DivBackward1Class, "DivBackward1", DivBackward1_properties);
6296:   static PyTypeObject DivBackward2Class;
6297:   addClass<DivBackward2>(module, DivBackward2Class, "DivBackward2", DivBackward2_properties);
6298:   static PyTypeObject DivBackward3Class;
6299:   addClass<DivBackward3>(module, DivBackward3Class, "DivBackward3", DivBackward3_properties);
6300:   static PyTypeObject ErfBackward0Class;
6301:   addClass<ErfBackward0>(module, ErfBackward0Class, "ErfBackward0", ErfBackward0_properties);
6302:   static PyTypeObject FakeQuantizeLearnablePerChannelAffineBackward0Class;
6303:   addClass<FakeQuantizeLearnablePerChannelAffineBackward0>(module, FakeQuantizeLearnablePerChannelAffineBackward0Class, "FakeQuantizeLearnablePerChannelAffineBackward0", FakeQuantizeLearnablePerChannelAffineBackward0_properties);
6304:   static PyTypeObject FillBackward2Class;
6305:   addClass<FillBackward2>(module, FillBackward2Class, "FillBackward2", FillBackward2_properties);
6306:   static PyTypeObject FillBackward3Class;
6307:   addClass<FillBackward3>(module, FillBackward3Class, "FillBackward3", FillBackward3_properties);
6308:   static PyTypeObject GridSampler3DBackward0Class;
6309:   addClass<GridSampler3DBackward0>(module, GridSampler3DBackward0Class, "GridSampler3DBackward0", GridSampler3DBackward0_properties);
6310:   static PyTypeObject GridSampler2DCpuFallbackBackward0Class;
6311:   addClass<GridSampler2DCpuFallbackBackward0>(module, GridSampler2DCpuFallbackBackward0Class, "GridSampler2DCpuFallbackBackward0", GridSampler2DCpuFallbackBackward0_properties);
6312:   static PyTypeObject HardswishBackwardBackward0Class;
6313:   addClass<HardswishBackwardBackward0>(module, HardswishBackwardBackward0Class, "HardswishBackwardBackward0", HardswishBackwardBackward0_properties);
6314:   static PyTypeObject IndexBackward0Class;
6315:   addClass<IndexBackward0>(module, IndexBackward0Class, "IndexBackward0", IndexBackward0_properties);
6316:   static PyTypeObject IndexReduceBackward0Class;
6317:   addClass<IndexReduceBackward0>(module, IndexReduceBackward0Class, "IndexReduceBackward0", IndexReduceBackward0_properties);
6318:   static PyTypeObject IndexFillBackward0Class;
6319:   addClass<IndexFillBackward0>(module, IndexFillBackward0Class, "IndexFillBackward0", IndexFillBackward0_properties);
6320:   static PyTypeObject IndexFillBackward1Class;
```

- EN: The main execution path in this span is carried by `THPVariable_Wrap`, `THPUnbindBackwardAutogradNestedTensor0_copy_self_raw_getter`, `cast`. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_Wrap`, `THPUnbindBackwardAutogradNestedTensor0_copy_self_raw_getter`, `cast` 等函数/方法承载。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 6321-6400

```cpp
6321:   addClass<IndexFillBackward1>(module, IndexFillBackward1Class, "IndexFillBackward1", IndexFillBackward1_properties);
6322:   static PyTypeObject IndexPutImplBackward0Class;
6323:   addClass<IndexPutImplBackward0>(module, IndexPutImplBackward0Class, "IndexPutImplBackward0", IndexPutImplBackward0_properties);
6324:   static PyTypeObject LinalgPinvBackward0Class;
6325:   addClass<LinalgPinvBackward0>(module, LinalgPinvBackward0Class, "LinalgPinvBackward0", LinalgPinvBackward0_properties);
6326:   static PyTypeObject KthvalueBackward0Class;
6327:   addClass<KthvalueBackward0>(module, KthvalueBackward0Class, "KthvalueBackward0", KthvalueBackward0_properties);
6328:   static PyTypeObject DigammaBackward0Class;
6329:   addClass<DigammaBackward0>(module, DigammaBackward0Class, "DigammaBackward0", DigammaBackward0_properties);
6330:   static PyTypeObject PolygammaBackward0Class;
6331:   addClass<PolygammaBackward0>(module, PolygammaBackward0Class, "PolygammaBackward0", PolygammaBackward0_properties);
6332:   static PyTypeObject PolygammaBackward1Class;
6333:   addClass<PolygammaBackward1>(module, PolygammaBackward1Class, "PolygammaBackward1", PolygammaBackward1_properties);
6334:   static PyTypeObject XlogyBackward0Class;
6335:   addClass<XlogyBackward0>(module, XlogyBackward0Class, "XlogyBackward0", XlogyBackward0_properties);
6336:   static PyTypeObject XlogyBackward1Class;
6337:   addClass<XlogyBackward1>(module, XlogyBackward1Class, "XlogyBackward1", XlogyBackward1_properties);
6338:   static PyTypeObject XlogyBackward2Class;
6339:   addClass<XlogyBackward2>(module, XlogyBackward2Class, "XlogyBackward2", XlogyBackward2_properties);
6340:   static PyTypeObject LinalgLuFactorExBackward0Class;
6341:   addClass<LinalgLuFactorExBackward0>(module, LinalgLuFactorExBackward0Class, "LinalgLuFactorExBackward0", LinalgLuFactorExBackward0_properties);
6342:   static PyTypeObject MaximumBackward0Class;
6343:   addClass<MaximumBackward0>(module, MaximumBackward0Class, "MaximumBackward0", MaximumBackward0_properties);
6344:   static PyTypeObject MinimumBackward0Class;
6345:   addClass<MinimumBackward0>(module, MinimumBackward0Class, "MinimumBackward0", MinimumBackward0_properties);
6346:   static PyTypeObject AmaxBackward0Class;
6347:   addClass<AmaxBackward0>(module, AmaxBackward0Class, "AmaxBackward0", AmaxBackward0_properties);
6348:   static PyTypeObject AminBackward0Class;
6349:   addClass<AminBackward0>(module, AminBackward0Class, "AminBackward0", AminBackward0_properties);
6350:   static PyTypeObject GroupedMmBackward0Class;
6351:   addClass<GroupedMmBackward0>(module, GroupedMmBackward0Class, "GroupedMmBackward0", GroupedMmBackward0_properties);
6352:   static PyTypeObject NanToNumBackward0Class;
6353:   addClass<NanToNumBackward0>(module, NanToNumBackward0Class, "NanToNumBackward0", NanToNumBackward0_properties);
6354:   static PyTypeObject NativeBatchNormBackward0Class;
6355:   addClass<NativeBatchNormBackward0>(module, NativeBatchNormBackward0Class, "NativeBatchNormBackward0", NativeBatchNormBackward0_properties);
6356:   static PyTypeObject NativeBatchNormBackwardBackward0Class;
6357:   addClass<NativeBatchNormBackwardBackward0>(module, NativeBatchNormBackwardBackward0Class, "NativeBatchNormBackwardBackward0", NativeBatchNormBackwardBackward0_properties);
6358:   static PyTypeObject NativeLayerNormBackward0Class;
6359:   addClass<NativeLayerNormBackward0>(module, NativeLayerNormBackward0Class, "NativeLayerNormBackward0", NativeLayerNormBackward0_properties);
6360:   static PyTypeObject NeBackward0Class;
6361:   addClass<NeBackward0>(module, NeBackward0Class, "NeBackward0", NeBackward0_properties);
6362:   static PyTypeObject NeBackward1Class;
6363:   addClass<NeBackward1>(module, NeBackward1Class, "NeBackward1", NeBackward1_properties);
6364:   static PyTypeObject BatchNormBackwardBackward0Class;
6365:   addClass<BatchNormBackwardBackward0>(module, BatchNormBackwardBackward0Class, "BatchNormBackwardBackward0", BatchNormBackwardBackward0_properties);
6366:   static PyTypeObject NextafterBackward0Class;
6367:   addClass<NextafterBackward0>(module, NextafterBackward0Class, "NextafterBackward0", NextafterBackward0_properties);
6368:   static PyTypeObject CdistBackward0Class;
6369:   addClass<CdistBackward0>(module, CdistBackward0Class, "CdistBackward0", CdistBackward0_properties);
6370:   static PyTypeObject NormalBackward1Class;
6371:   addClass<NormalBackward1>(module, NormalBackward1Class, "NormalBackward1", NormalBackward1_properties);
6372:   static PyTypeObject NormalBackward2Class;
6373:   addClass<NormalBackward2>(module, NormalBackward2Class, "NormalBackward2", NormalBackward2_properties);
6374:   static PyTypeObject NormalBackward3Class;
6375:   addClass<NormalBackward3>(module, NormalBackward3Class, "NormalBackward3", NormalBackward3_properties);
6376:   static PyTypeObject SpecialLogNdtrBackward0Class;
6377:   addClass<SpecialLogNdtrBackward0>(module, SpecialLogNdtrBackward0Class, "SpecialLogNdtrBackward0", SpecialLogNdtrBackward0_properties);
6378:   static PyTypeObject RoundBackward0Class;
6379:   addClass<RoundBackward0>(module, RoundBackward0Class, "RoundBackward0", RoundBackward0_properties);
6380:   static PyTypeObject RoundBackward1Class;
6381:   addClass<RoundBackward1>(module, RoundBackward1Class, "RoundBackward1", RoundBackward1_properties);
6382:   static PyTypeObject ScatterAddBackward0Class;
6383:   addClass<ScatterAddBackward0>(module, ScatterAddBackward0Class, "ScatterAddBackward0", ScatterAddBackward0_properties);
6384:   static PyTypeObject SelectBackwardBackward0Class;
6385:   addClass<SelectBackwardBackward0>(module, SelectBackwardBackward0Class, "SelectBackwardBackward0", SelectBackwardBackward0_properties);
6386:   static PyTypeObject SigmoidBackward0Class;
6387:   addClass<SigmoidBackward0>(module, SigmoidBackward0Class, "SigmoidBackward0", SigmoidBackward0_properties);
6388:   static PyTypeObject SliceScatterBackward0Class;
6389:   addClass<SliceScatterBackward0>(module, SliceScatterBackward0Class, "SliceScatterBackward0", SliceScatterBackward0_properties);
6390:   static PyTypeObject SplitWithSizesBackward0Class;
6391:   addClass<SplitWithSizesBackward0>(module, SplitWithSizesBackward0Class, "SplitWithSizesBackward0", SplitWithSizesBackward0_properties);
6392:   static PyTypeObject SplitWithSizesBackwardAutogradNestedTensor0Class;
6393:   addClass<SplitWithSizesBackwardAutogradNestedTensor0>(module, SplitWithSizesBackwardAutogradNestedTensor0Class, "SplitWithSizesBackwardAutogradNestedTensor0", SplitWithSizesBackwardAutogradNestedTensor0_properties);
6394:   static PyTypeObject TBackward1Class;
6395:   addClass<TBackward1>(module, TBackward1Class, "TBackward1", TBackward1_properties);
6396:   static PyTypeObject TanhBackward0Class;
6397:   addClass<TanhBackward0>(module, TanhBackward0Class, "TanhBackward0", TanhBackward0_properties);
6398:   static PyTypeObject TopkBackward0Class;
6399:   addClass<TopkBackward0>(module, TopkBackward0Class, "TopkBackward0", TopkBackward0_properties);
6400:   static PyTypeObject ToSparseCscBackward0Class;
```

- EN: Autograd-related metadata or graph connectivity is updated here.
- CN: 这里会更新与自动求导相关的元数据或计算图连接关系。
### Lines 6401-6480

```cpp
6401:   addClass<ToSparseCscBackward0>(module, ToSparseCscBackward0Class, "ToSparseCscBackward0", ToSparseCscBackward0_properties);
6402:   static PyTypeObject ToSparseBscBackward0Class;
6403:   addClass<ToSparseBscBackward0>(module, ToSparseBscBackward0Class, "ToSparseBscBackward0", ToSparseBscBackward0_properties);
6404:   static PyTypeObject UnfoldBackwardBackward0Class;
6405:   addClass<UnfoldBackwardBackward0>(module, UnfoldBackwardBackward0Class, "UnfoldBackwardBackward0", UnfoldBackwardBackward0_properties);
6406:   static PyTypeObject UniformBackward0Class;
6407:   addClass<UniformBackward0>(module, UniformBackward0Class, "UniformBackward0", UniformBackward0_properties);
6408:   static PyTypeObject UniqueDimBackward0Class;
6409:   addClass<UniqueDimBackward0>(module, UniqueDimBackward0Class, "UniqueDimBackward0", UniqueDimBackward0_properties);
6410:   static PyTypeObject UniqueConsecutiveBackward0Class;
6411:   addClass<UniqueConsecutiveBackward0>(module, UniqueConsecutiveBackward0Class, "UniqueConsecutiveBackward0", UniqueConsecutiveBackward0_properties);
6412:   static PyTypeObject BinaryCrossEntropyBackwardBackward0Class;
6413:   addClass<BinaryCrossEntropyBackwardBackward0>(module, BinaryCrossEntropyBackwardBackward0Class, "BinaryCrossEntropyBackwardBackward0", BinaryCrossEntropyBackwardBackward0_properties);
6414:   static PyTypeObject MseLossBackward0Class;
6415:   addClass<MseLossBackward0>(module, MseLossBackward0Class, "MseLossBackward0", MseLossBackward0_properties);
6416:   static PyTypeObject NllLoss2DBackward0Class;
6417:   addClass<NllLoss2DBackward0>(module, NllLoss2DBackward0Class, "NllLoss2DBackward0", NllLoss2DBackward0_properties);
6418:   static PyTypeObject SoftMarginLossBackward0Class;
6419:   addClass<SoftMarginLossBackward0>(module, SoftMarginLossBackward0Class, "SoftMarginLossBackward0", SoftMarginLossBackward0_properties);
6420:   static PyTypeObject MishBackward0Class;
6421:   addClass<MishBackward0>(module, MishBackward0Class, "MishBackward0", MishBackward0_properties);
6422:   static PyTypeObject GeluBackward0Class;
6423:   addClass<GeluBackward0>(module, GeluBackward0Class, "GeluBackward0", GeluBackward0_properties);
6424:   static PyTypeObject GluBackward0Class;
6425:   addClass<GluBackward0>(module, GluBackward0Class, "GluBackward0", GluBackward0_properties);
6426:   static PyTypeObject HardshrinkBackward0Class;
6427:   addClass<HardshrinkBackward0>(module, HardshrinkBackward0Class, "HardshrinkBackward0", HardshrinkBackward0_properties);
6428:   static PyTypeObject HardtanhBackward0Class;
6429:   addClass<HardtanhBackward0>(module, HardtanhBackward0Class, "HardtanhBackward0", HardtanhBackward0_properties);
6430:   static PyTypeObject LeakyReluBackward0Class;
6431:   addClass<LeakyReluBackward0>(module, LeakyReluBackward0Class, "LeakyReluBackward0", LeakyReluBackward0_properties);
6432:   static PyTypeObject LogSigmoidBackward0Class;
6433:   addClass<LogSigmoidBackward0>(module, LogSigmoidBackward0Class, "LogSigmoidBackward0", LogSigmoidBackward0_properties);
6434:   static PyTypeObject RreluWithNoiseBackward1Class;
6435:   addClass<RreluWithNoiseBackward1>(module, RreluWithNoiseBackward1Class, "RreluWithNoiseBackward1", RreluWithNoiseBackward1_properties);
6436:   static PyTypeObject SoftshrinkBackward0Class;
6437:   addClass<SoftshrinkBackward0>(module, SoftshrinkBackward0Class, "SoftshrinkBackward0", SoftshrinkBackward0_properties);
6438:   static PyTypeObject ReflectionPad3DBackward0Class;
6439:   addClass<ReflectionPad3DBackward0>(module, ReflectionPad3DBackward0Class, "ReflectionPad3DBackward0", ReflectionPad3DBackward0_properties);
6440:   static PyTypeObject ReplicationPad1DBackward0Class;
6441:   addClass<ReplicationPad1DBackward0>(module, ReplicationPad1DBackward0Class, "ReplicationPad1DBackward0", ReplicationPad1DBackward0_properties);
6442:   static PyTypeObject ReplicationPad2DBackward0Class;
6443:   addClass<ReplicationPad2DBackward0>(module, ReplicationPad2DBackward0Class, "ReplicationPad2DBackward0", ReplicationPad2DBackward0_properties);
6444:   static PyTypeObject UpsampleNearest1DBackward0Class;
6445:   addClass<UpsampleNearest1DBackward0>(module, UpsampleNearest1DBackward0Class, "UpsampleNearest1DBackward0", UpsampleNearest1DBackward0_properties);
6446:   static PyTypeObject UpsampleNearestExact1DBackward0Class;
6447:   addClass<UpsampleNearestExact1DBackward0>(module, UpsampleNearestExact1DBackward0Class, "UpsampleNearestExact1DBackward0", UpsampleNearestExact1DBackward0_properties);
6448:   static PyTypeObject FractionalMaxPool3DBackward0Class;
6449:   addClass<FractionalMaxPool3DBackward0>(module, FractionalMaxPool3DBackward0Class, "FractionalMaxPool3DBackward0", FractionalMaxPool3DBackward0_properties);
6450:   static PyTypeObject MpsConvolutionBackward0Class;
6451:   addClass<MpsConvolutionBackward0>(module, MpsConvolutionBackward0Class, "MpsConvolutionBackward0", MpsConvolutionBackward0_properties);
6452:   static PyTypeObject ConvDepthwise2DBackward0Class;
6453:   addClass<ConvDepthwise2DBackward0>(module, ConvDepthwise2DBackward0Class, "ConvDepthwise2DBackward0", ConvDepthwise2DBackward0_properties);
6454:   static PyTypeObject AdaptiveAvgPool3DBackwardBackward0Class;
6455:   addClass<AdaptiveAvgPool3DBackwardBackward0>(module, AdaptiveAvgPool3DBackwardBackward0Class, "AdaptiveAvgPool3DBackwardBackward0", AdaptiveAvgPool3DBackwardBackward0_properties);
6456:   static PyTypeObject AdaptiveMaxPool2DBackwardBackward0Class;
6457:   addClass<AdaptiveMaxPool2DBackwardBackward0>(module, AdaptiveMaxPool2DBackwardBackward0Class, "AdaptiveMaxPool2DBackwardBackward0", AdaptiveMaxPool2DBackwardBackward0_properties);
6458:   static PyTypeObject AdaptiveMaxPool3DBackwardBackward0Class;
6459:   addClass<AdaptiveMaxPool3DBackwardBackward0>(module, AdaptiveMaxPool3DBackwardBackward0Class, "AdaptiveMaxPool3DBackwardBackward0", AdaptiveMaxPool3DBackwardBackward0_properties);
6460:   static PyTypeObject LogSigmoidBackwardBackward0Class;
6461:   addClass<LogSigmoidBackwardBackward0>(module, LogSigmoidBackwardBackward0Class, "LogSigmoidBackwardBackward0", LogSigmoidBackwardBackward0_properties);
6462:   static PyTypeObject LogSoftmaxBackwardDataBackward0Class;
6463:   addClass<LogSoftmaxBackwardDataBackward0>(module, LogSoftmaxBackwardDataBackward0Class, "LogSoftmaxBackwardDataBackward0", LogSoftmaxBackwardDataBackward0_properties);
6464:   static PyTypeObject NllLoss2DBackwardBackward0Class;
6465:   addClass<NllLoss2DBackwardBackward0>(module, NllLoss2DBackwardBackward0Class, "NllLoss2DBackwardBackward0", NllLoss2DBackwardBackward0_properties);
6466:   static PyTypeObject SparseSampledAddmmBackward0Class;
6467:   addClass<SparseSampledAddmmBackward0>(module, SparseSampledAddmmBackward0Class, "SparseSampledAddmmBackward0", SparseSampledAddmmBackward0_properties);
6468:   static PyTypeObject SoftmaxBackwardDataBackward0Class;
6469:   addClass<SoftmaxBackwardDataBackward0>(module, SoftmaxBackwardDataBackward0Class, "SoftmaxBackwardDataBackward0", SoftmaxBackwardDataBackward0_properties);
6470:   static PyTypeObject CudnnGridSamplerBackward0Class;
6471:   addClass<CudnnGridSamplerBackward0>(module, CudnnGridSamplerBackward0Class, "CudnnGridSamplerBackward0", CudnnGridSamplerBackward0_properties);
6472:   static PyTypeObject NnpackSpatialConvolutionBackward0Class;
6473:   addClass<NnpackSpatialConvolutionBackward0>(module, NnpackSpatialConvolutionBackward0Class, "NnpackSpatialConvolutionBackward0", NnpackSpatialConvolutionBackward0_properties);
6474:   static PyTypeObject LstmMpsBackward0Class;
6475:   addClass<LstmMpsBackward0>(module, LstmMpsBackward0Class, "LstmMpsBackward0", LstmMpsBackward0_properties);
6476:   static PyTypeObject MiopenDepthwiseConvolutionBackward0Class;
6477:   addClass<MiopenDepthwiseConvolutionBackward0>(module, MiopenDepthwiseConvolutionBackward0Class, "MiopenDepthwiseConvolutionBackward0", MiopenDepthwiseConvolutionBackward0_properties);
6478:   static PyTypeObject MkldnnMaxPool2DBackward0Class;
6479:   addClass<MkldnnMaxPool2DBackward0>(module, MkldnnMaxPool2DBackward0Class, "MkldnnMaxPool2DBackward0", MkldnnMaxPool2DBackward0_properties);
6480:   static PyTypeObject ScaledDotProductFlashAttentionBackward0Class;
```

- EN: This range continues the surrounding implementation through local state updates, helper invocations, and data movement.
- CN: 这一段通过局部状态更新、辅助函数调用与数据传递来延续周围实现。
### Lines 6481-6512

```cpp
6481:   addClass<ScaledDotProductFlashAttentionBackward0>(module, ScaledDotProductFlashAttentionBackward0Class, "ScaledDotProductFlashAttentionBackward0", ScaledDotProductFlashAttentionBackward0_properties);
6482:   static PyTypeObject ScaledDotProductFlashAttentionForCpuBackward0Class;
6483:   addClass<ScaledDotProductFlashAttentionForCpuBackward0>(module, ScaledDotProductFlashAttentionForCpuBackward0Class, "ScaledDotProductFlashAttentionForCpuBackward0", ScaledDotProductFlashAttentionForCpuBackward0_properties);
6484:   static PyTypeObject FftC2CBackward0Class;
6485:   addClass<FftC2CBackward0>(module, FftC2CBackward0Class, "FftC2CBackward0", FftC2CBackward0_properties);
6486:   static PyTypeObject StackBackward0Class;
6487:   addClass<StackBackward0>(module, StackBackward0Class, "StackBackward0", StackBackward0_properties);
6488:   static PyTypeObject ThnnFusedLstmCellBackward0Class;
6489:   addClass<ThnnFusedLstmCellBackward0>(module, ThnnFusedLstmCellBackward0Class, "ThnnFusedLstmCellBackward0", ThnnFusedLstmCellBackward0_properties);
6490:   static PyTypeObject ForeachMaximumBackward0Class;
6491:   addClass<ForeachMaximumBackward0>(module, ForeachMaximumBackward0Class, "ForeachMaximumBackward0", ForeachMaximumBackward0_properties);
6492:   static PyTypeObject ForeachMaximumBackward1Class;
6493:   addClass<ForeachMaximumBackward1>(module, ForeachMaximumBackward1Class, "ForeachMaximumBackward1", ForeachMaximumBackward1_properties);
6494:   static PyTypeObject AliasBackward0_copyClass;
6495:   addClass<AliasBackward0_copy>(module, AliasBackward0_copyClass, "AliasBackward0_copy", AliasBackward0_copy_properties);
6496:   static PyTypeObject SplitBackward0_copyClass;
6497:   addClass<SplitBackward0_copy>(module, SplitBackward0_copyClass, "SplitBackward0_copy", SplitBackward0_copy_properties);
6498:   static PyTypeObject TransposeBackward0_copyClass;
6499:   addClass<TransposeBackward0_copy>(module, TransposeBackward0_copyClass, "TransposeBackward0_copy", TransposeBackward0_copy_properties);
6500:   static PyTypeObject LiftFreshBackward0_copyClass;
6501:   addClass<LiftFreshBackward0_copy>(module, LiftFreshBackward0_copyClass, "LiftFreshBackward0_copy", LiftFreshBackward0_copy_properties);
6502:   static PyTypeObject ViewAsRealBackward0_copyClass;
6503:   addClass<ViewAsRealBackward0_copy>(module, ViewAsRealBackward0_copyClass, "ViewAsRealBackward0_copy", ViewAsRealBackward0_copy_properties);
6504:   static PyTypeObject NestedGetValuesBackward0_copyClass;
6505:   addClass<NestedGetValuesBackward0_copy>(module, NestedGetValuesBackward0_copyClass, "NestedGetValuesBackward0_copy", NestedGetValuesBackward0_copy_properties);
6506:   static PyTypeObject UnbindBackward0_copyClass;
6507:   addClass<UnbindBackward0_copy>(module, UnbindBackward0_copyClass, "UnbindBackward0_copy", UnbindBackward0_copy_properties);
6508:   static PyTypeObject UnbindBackwardAutogradNestedTensor0_copyClass;
6509:   addClass<UnbindBackwardAutogradNestedTensor0_copy>(module, UnbindBackwardAutogradNestedTensor0_copyClass, "UnbindBackwardAutogradNestedTensor0_copy", UnbindBackwardAutogradNestedTensor0_copy_properties);
6510: }
6511: 
6512: } // namespace torch::autograd::generated
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
- Key symbols / 关键符号: `addClass`, `THPAbsBackward0_self_getter`, `THPVariable_Wrap`, `THPAbsBackward0_self_raw_getter`, `THPAddmvBackward0_alpha_getter`, `PyComplex_FromDoubles`, `PyFloat_FromDouble`, `PyLong_FromLong`, `THPAddmvBackward0_beta_getter`, `THPAddmvBackward0_mat_getter`
- Related subsystems / 相关子系统: Autograd / 自动求导, ATen operator layer / ATen 算子层, c10 core utilities / c10 核心工具, Python binding layer / Python 绑定层
