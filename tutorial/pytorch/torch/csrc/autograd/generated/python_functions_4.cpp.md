# python_functions_4.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/autograd/generated/python_functions_4.cpp`
- Repository: `/root/xw/pytorch`
- Purpose (EN): Defines generated backward-function classes and helpers used by the autograd engine.
- 目的 (CN): 定义自动求导引擎使用的生成式反向函数类与辅助逻辑。
- Lines: 6253
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
31: static PyObject* THPAcosBackward0_self_getter(THPCppFunction *self, void *_unused) {
32:   HANDLE_TH_ERRORS
33:   const auto& prop = static_cast<AcosBackward0*>(self->cdata.get())->self_;
34:   return THPVariable_Wrap(prop.unpack(self->cdata));
35:   END_HANDLE_TH_ERRORS
36: }
37: 
38: static PyObject* THPAcosBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
39:   HANDLE_TH_ERRORS
40:   const auto& prop = static_cast<AcosBackward0*>(self->cdata.get())->self_;
41:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
42:   return obj.release().ptr();
43:   END_HANDLE_TH_ERRORS
44: }
45: 
46: static struct PyGetSetDef AcosBackward0_properties[] = {
47:   THP_FUNCTION_DEFAULT_PROPERTIES,
48:   {(char*)"_saved_self", (getter)THPAcosBackward0_self_getter, nullptr, nullptr, nullptr},
49:   {(char*)"_raw_saved_self", (getter)THPAcosBackward0_self_raw_getter, nullptr, nullptr, nullptr},
50:   {nullptr} /* sentinel */
51: };
52: 
53: static PyObject* THPAsinBackward0_self_getter(THPCppFunction *self, void *_unused) {
54:   HANDLE_TH_ERRORS
55:   const auto& prop = static_cast<AsinBackward0*>(self->cdata.get())->self_;
56:   return THPVariable_Wrap(prop.unpack(self->cdata));
57:   END_HANDLE_TH_ERRORS
58: }
59: 
60: static PyObject* THPAsinBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
61:   HANDLE_TH_ERRORS
62:   const auto& prop = static_cast<AsinBackward0*>(self->cdata.get())->self_;
63:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
64:   return obj.release().ptr();
65:   END_HANDLE_TH_ERRORS
66: }
67: 
68: static struct PyGetSetDef AsinBackward0_properties[] = {
69:   THP_FUNCTION_DEFAULT_PROPERTIES,
70:   {(char*)"_saved_self", (getter)THPAsinBackward0_self_getter, nullptr, nullptr, nullptr},
71:   {(char*)"_raw_saved_self", (getter)THPAsinBackward0_self_raw_getter, nullptr, nullptr, nullptr},
72:   {nullptr} /* sentinel */
73: };
74: 
75: static PyObject* THPMatmulBackward0_other_getter(THPCppFunction *self, void *_unused) {
76:   HANDLE_TH_ERRORS
77:   const auto& prop = static_cast<MatmulBackward0*>(self->cdata.get())->other_;
78:   return THPVariable_Wrap(prop.unpack(self->cdata));
79:   END_HANDLE_TH_ERRORS
80: }
```

- EN: These lines pull in dependencies such as `torch/csrc/autograd/generated/python_functions.h`, `Python.h`, `ATen/ATen.h`, establishing the headers needed by the implementation. They also open, refine, or close the relevant C++ namespace scope. The main execution path in this span is carried by `addClass`, `_initFunctionPyTypeObject`, `Py_INCREF`. Because this is generated binding code, the span mostly registers or forwards APIs into a mechanically produced Python-facing surface.
- CN: 这些行引入了依赖，例如 `torch/csrc/autograd/generated/python_functions.h`, `Python.h`, `ATen/ATen.h`，为后续实现建立所需的头文件基础。 它们还会打开、细化或关闭相应的 C++ 命名空间作用域。 这一段的主要执行路径由 `addClass`, `_initFunctionPyTypeObject`, `Py_INCREF` 等函数/方法承载。 由于这是生成的绑定代码，这一段主要是在机械化生成的 Python 接口层上完成 API 注册或转发。
### Lines 81-160

```cpp
 81: 
 82: static PyObject* THPMatmulBackward0_other_raw_getter(THPCppFunction *self, void *_unused) {
 83:   HANDLE_TH_ERRORS
 84:   const auto& prop = static_cast<MatmulBackward0*>(self->cdata.get())->other_;
 85:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
 86:   return obj.release().ptr();
 87:   END_HANDLE_TH_ERRORS
 88: }
 89: 
 90: static PyObject* THPMatmulBackward0_self_getter(THPCppFunction *self, void *_unused) {
 91:   HANDLE_TH_ERRORS
 92:   const auto& prop = static_cast<MatmulBackward0*>(self->cdata.get())->self_;
 93:   return THPVariable_Wrap(prop.unpack(self->cdata));
 94:   END_HANDLE_TH_ERRORS
 95: }
 96: 
 97: static PyObject* THPMatmulBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
 98:   HANDLE_TH_ERRORS
 99:   const auto& prop = static_cast<MatmulBackward0*>(self->cdata.get())->self_;
100:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
101:   return obj.release().ptr();
102:   END_HANDLE_TH_ERRORS
103: }
104: 
105: static struct PyGetSetDef MatmulBackward0_properties[] = {
106:   THP_FUNCTION_DEFAULT_PROPERTIES,
107:   {(char*)"_saved_other", (getter)THPMatmulBackward0_other_getter, nullptr, nullptr, nullptr},
108:   {(char*)"_raw_saved_other", (getter)THPMatmulBackward0_other_raw_getter, nullptr, nullptr, nullptr},
109:   {(char*)"_saved_self", (getter)THPMatmulBackward0_self_getter, nullptr, nullptr, nullptr},
110:   {(char*)"_raw_saved_self", (getter)THPMatmulBackward0_self_raw_getter, nullptr, nullptr, nullptr},
111:   {nullptr} /* sentinel */
112: };
113: 
114: static PyObject* THPCopysignBackward0_self_getter(THPCppFunction *self, void *_unused) {
115:   HANDLE_TH_ERRORS
116:   const auto& prop = static_cast<CopysignBackward0*>(self->cdata.get())->self_;
117:   return THPVariable_Wrap(prop.unpack(self->cdata));
118:   END_HANDLE_TH_ERRORS
119: }
120: 
121: static PyObject* THPCopysignBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
122:   HANDLE_TH_ERRORS
123:   const auto& prop = static_cast<CopysignBackward0*>(self->cdata.get())->self_;
124:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
125:   return obj.release().ptr();
126:   END_HANDLE_TH_ERRORS
127: }
128: 
129: static PyObject* THPCopysignBackward0_result_getter(THPCppFunction *self, void *_unused) {
130:   HANDLE_TH_ERRORS
131:   const auto& prop = static_cast<CopysignBackward0*>(self->cdata.get())->result_;
132:   return THPVariable_Wrap(prop.unpack(self->cdata));
133:   END_HANDLE_TH_ERRORS
134: }
135: 
136: static PyObject* THPCopysignBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
137:   HANDLE_TH_ERRORS
138:   const auto& prop = static_cast<CopysignBackward0*>(self->cdata.get())->result_;
139:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
140:   return obj.release().ptr();
141:   END_HANDLE_TH_ERRORS
142: }
143: 
144: static struct PyGetSetDef CopysignBackward0_properties[] = {
145:   THP_FUNCTION_DEFAULT_PROPERTIES,
146:   {(char*)"_saved_self", (getter)THPCopysignBackward0_self_getter, nullptr, nullptr, nullptr},
147:   {(char*)"_raw_saved_self", (getter)THPCopysignBackward0_self_raw_getter, nullptr, nullptr, nullptr},
148:   {(char*)"_saved_result", (getter)THPCopysignBackward0_result_getter, nullptr, nullptr, nullptr},
149:   {(char*)"_raw_saved_result", (getter)THPCopysignBackward0_result_raw_getter, nullptr, nullptr, nullptr},
150:   {nullptr} /* sentinel */
151: };
152: 
153: static PyObject* THPCopysignBackward1_self_getter(THPCppFunction *self, void *_unused) {
154:   HANDLE_TH_ERRORS
155:   const auto& prop = static_cast<CopysignBackward1*>(self->cdata.get())->self_;
156:   return THPVariable_Wrap(prop.unpack(self->cdata));
157:   END_HANDLE_TH_ERRORS
158: }
159: 
160: static PyObject* THPCopysignBackward1_self_raw_getter(THPCppFunction *self, void *_unused) {
```

- EN: The main execution path in this span is carried by `THPMatmulBackward0_other_raw_getter`, `cast`, `THPMatmulBackward0_self_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPMatmulBackward0_other_raw_getter`, `cast`, `THPMatmulBackward0_self_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 161-240

```cpp
161:   HANDLE_TH_ERRORS
162:   const auto& prop = static_cast<CopysignBackward1*>(self->cdata.get())->self_;
163:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
164:   return obj.release().ptr();
165:   END_HANDLE_TH_ERRORS
166: }
167: 
168: static PyObject* THPCopysignBackward1_result_getter(THPCppFunction *self, void *_unused) {
169:   HANDLE_TH_ERRORS
170:   const auto& prop = static_cast<CopysignBackward1*>(self->cdata.get())->result_;
171:   return THPVariable_Wrap(prop.unpack(self->cdata));
172:   END_HANDLE_TH_ERRORS
173: }
174: 
175: static PyObject* THPCopysignBackward1_result_raw_getter(THPCppFunction *self, void *_unused) {
176:   HANDLE_TH_ERRORS
177:   const auto& prop = static_cast<CopysignBackward1*>(self->cdata.get())->result_;
178:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
179:   return obj.release().ptr();
180:   END_HANDLE_TH_ERRORS
181: }
182: 
183: static struct PyGetSetDef CopysignBackward1_properties[] = {
184:   THP_FUNCTION_DEFAULT_PROPERTIES,
185:   {(char*)"_saved_self", (getter)THPCopysignBackward1_self_getter, nullptr, nullptr, nullptr},
186:   {(char*)"_raw_saved_self", (getter)THPCopysignBackward1_self_raw_getter, nullptr, nullptr, nullptr},
187:   {(char*)"_saved_result", (getter)THPCopysignBackward1_result_getter, nullptr, nullptr, nullptr},
188:   {(char*)"_raw_saved_result", (getter)THPCopysignBackward1_result_raw_getter, nullptr, nullptr, nullptr},
189:   {nullptr} /* sentinel */
190: };
191: 
192: 
193: 
194: static struct PyGetSetDef Deg2RadBackward0_properties[] = {
195:   THP_FUNCTION_DEFAULT_PROPERTIES,
196: 
197:   {nullptr} /* sentinel */
198: };
199: 
200: static PyObject* THPLinalgSlogdetBackward0_A_getter(THPCppFunction *self, void *_unused) {
201:   HANDLE_TH_ERRORS
202:   const auto& prop = static_cast<LinalgSlogdetBackward0*>(self->cdata.get())->A_;
203:   return THPVariable_Wrap(prop.unpack(self->cdata));
204:   END_HANDLE_TH_ERRORS
205: }
206: 
207: static PyObject* THPLinalgSlogdetBackward0_A_raw_getter(THPCppFunction *self, void *_unused) {
208:   HANDLE_TH_ERRORS
209:   const auto& prop = static_cast<LinalgSlogdetBackward0*>(self->cdata.get())->A_;
210:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
211:   return obj.release().ptr();
212:   END_HANDLE_TH_ERRORS
213: }
214: 
215: static PyObject* THPLinalgSlogdetBackward0_LU_getter(THPCppFunction *self, void *_unused) {
216:   HANDLE_TH_ERRORS
217:   const auto& prop = static_cast<LinalgSlogdetBackward0*>(self->cdata.get())->LU_;
218:   return THPVariable_Wrap(prop.unpack(self->cdata));
219:   END_HANDLE_TH_ERRORS
220: }
221: 
222: static PyObject* THPLinalgSlogdetBackward0_LU_raw_getter(THPCppFunction *self, void *_unused) {
223:   HANDLE_TH_ERRORS
224:   const auto& prop = static_cast<LinalgSlogdetBackward0*>(self->cdata.get())->LU_;
225:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
226:   return obj.release().ptr();
227:   END_HANDLE_TH_ERRORS
228: }
229: 
230: static PyObject* THPLinalgSlogdetBackward0_pivots_getter(THPCppFunction *self, void *_unused) {
231:   HANDLE_TH_ERRORS
232:   const auto& prop = static_cast<LinalgSlogdetBackward0*>(self->cdata.get())->pivots_;
233:   return THPVariable_Wrap(prop.unpack(self->cdata));
234:   END_HANDLE_TH_ERRORS
235: }
236: 
237: static PyObject* THPLinalgSlogdetBackward0_pivots_raw_getter(THPCppFunction *self, void *_unused) {
238:   HANDLE_TH_ERRORS
239:   const auto& prop = static_cast<LinalgSlogdetBackward0*>(self->cdata.get())->pivots_;
240:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
```

- EN: The main execution path in this span is carried by `cast`, `THPCopysignBackward1_result_getter`, `THPVariable_Wrap`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `cast`, `THPCopysignBackward1_result_getter`, `THPVariable_Wrap` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 241-320

```cpp
241:   return obj.release().ptr();
242:   END_HANDLE_TH_ERRORS
243: }
244: 
245: static PyObject* THPLinalgSlogdetBackward0_sign_getter(THPCppFunction *self, void *_unused) {
246:   HANDLE_TH_ERRORS
247:   const auto& prop = static_cast<LinalgSlogdetBackward0*>(self->cdata.get())->sign_;
248:   return THPVariable_Wrap(prop.unpack(self->cdata));
249:   END_HANDLE_TH_ERRORS
250: }
251: 
252: static PyObject* THPLinalgSlogdetBackward0_sign_raw_getter(THPCppFunction *self, void *_unused) {
253:   HANDLE_TH_ERRORS
254:   const auto& prop = static_cast<LinalgSlogdetBackward0*>(self->cdata.get())->sign_;
255:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
256:   return obj.release().ptr();
257:   END_HANDLE_TH_ERRORS
258: }
259: 
260: static struct PyGetSetDef LinalgSlogdetBackward0_properties[] = {
261:   THP_FUNCTION_DEFAULT_PROPERTIES,
262:   {(char*)"_saved_A", (getter)THPLinalgSlogdetBackward0_A_getter, nullptr, nullptr, nullptr},
263:   {(char*)"_raw_saved_A", (getter)THPLinalgSlogdetBackward0_A_raw_getter, nullptr, nullptr, nullptr},
264:   {(char*)"_saved_LU", (getter)THPLinalgSlogdetBackward0_LU_getter, nullptr, nullptr, nullptr},
265:   {(char*)"_raw_saved_LU", (getter)THPLinalgSlogdetBackward0_LU_raw_getter, nullptr, nullptr, nullptr},
266:   {(char*)"_saved_pivots", (getter)THPLinalgSlogdetBackward0_pivots_getter, nullptr, nullptr, nullptr},
267:   {(char*)"_raw_saved_pivots", (getter)THPLinalgSlogdetBackward0_pivots_raw_getter, nullptr, nullptr, nullptr},
268:   {(char*)"_saved_sign", (getter)THPLinalgSlogdetBackward0_sign_getter, nullptr, nullptr, nullptr},
269:   {(char*)"_raw_saved_sign", (getter)THPLinalgSlogdetBackward0_sign_raw_getter, nullptr, nullptr, nullptr},
270:   {nullptr} /* sentinel */
271: };
272: 
273: static PyObject* THPDistBackward0_other_getter(THPCppFunction *self, void *_unused) {
274:   HANDLE_TH_ERRORS
275:   const auto& prop = static_cast<DistBackward0*>(self->cdata.get())->other_;
276:   return THPVariable_Wrap(prop.unpack(self->cdata));
277:   END_HANDLE_TH_ERRORS
278: }
279: 
280: static PyObject* THPDistBackward0_other_raw_getter(THPCppFunction *self, void *_unused) {
281:   HANDLE_TH_ERRORS
282:   const auto& prop = static_cast<DistBackward0*>(self->cdata.get())->other_;
283:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
284:   return obj.release().ptr();
285:   END_HANDLE_TH_ERRORS
286: }
287: 
288: static PyObject* THPDistBackward0_p_getter(THPCppFunction *self, void *_unused) {
289:   HANDLE_TH_ERRORS
290:   auto prop = static_cast<DistBackward0*>(self->cdata.get())->p;
291:   if (prop.isComplex()) {
292:     auto cprop = prop.to<c10::complex<double>>();
293:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
294:   } else if (prop.isFloatingPoint()) {
295:     return PyFloat_FromDouble(prop.to<double>());
296:   } else if (prop.isIntegral(/*includeBool=*/false)) {
297:     return PyLong_FromLong(prop.to<int64_t>());
298:   } else if (prop.isBoolean()) {
299:     if (prop.to<bool>()) {
300:       Py_RETURN_TRUE;
301:     } else {
302:       Py_RETURN_FALSE;
303:     }
304:   } else {
305:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
306:     return nullptr;
307:   }
308:   END_HANDLE_TH_ERRORS
309: }
310: 
311: static PyObject* THPDistBackward0_self_getter(THPCppFunction *self, void *_unused) {
312:   HANDLE_TH_ERRORS
313:   const auto& prop = static_cast<DistBackward0*>(self->cdata.get())->self_;
314:   return THPVariable_Wrap(prop.unpack(self->cdata));
315:   END_HANDLE_TH_ERRORS
316: }
317: 
318: static PyObject* THPDistBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
319:   HANDLE_TH_ERRORS
320:   const auto& prop = static_cast<DistBackward0*>(self->cdata.get())->self_;
```

- EN: The main execution path in this span is carried by `THPLinalgSlogdetBackward0_sign_getter`, `THPVariable_Wrap`, `THPLinalgSlogdetBackward0_sign_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPLinalgSlogdetBackward0_sign_getter`, `THPVariable_Wrap`, `THPLinalgSlogdetBackward0_sign_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 321-400

```cpp
321:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
322:   return obj.release().ptr();
323:   END_HANDLE_TH_ERRORS
324: }
325: 
326: static PyObject* THPDistBackward0_result_getter(THPCppFunction *self, void *_unused) {
327:   HANDLE_TH_ERRORS
328:   const auto& prop = static_cast<DistBackward0*>(self->cdata.get())->result_;
329:   return THPVariable_Wrap(prop.unpack(self->cdata));
330:   END_HANDLE_TH_ERRORS
331: }
332: 
333: static PyObject* THPDistBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
334:   HANDLE_TH_ERRORS
335:   const auto& prop = static_cast<DistBackward0*>(self->cdata.get())->result_;
336:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
337:   return obj.release().ptr();
338:   END_HANDLE_TH_ERRORS
339: }
340: 
341: static struct PyGetSetDef DistBackward0_properties[] = {
342:   THP_FUNCTION_DEFAULT_PROPERTIES,
343:   {(char*)"_saved_other", (getter)THPDistBackward0_other_getter, nullptr, nullptr, nullptr},
344:   {(char*)"_raw_saved_other", (getter)THPDistBackward0_other_raw_getter, nullptr, nullptr, nullptr},
345:   {(char*)"_saved_p", (getter)THPDistBackward0_p_getter, nullptr, nullptr, nullptr},
346:   {(char*)"_saved_self", (getter)THPDistBackward0_self_getter, nullptr, nullptr, nullptr},
347:   {(char*)"_raw_saved_self", (getter)THPDistBackward0_self_raw_getter, nullptr, nullptr, nullptr},
348:   {(char*)"_saved_result", (getter)THPDistBackward0_result_getter, nullptr, nullptr, nullptr},
349:   {(char*)"_raw_saved_result", (getter)THPDistBackward0_result_raw_getter, nullptr, nullptr, nullptr},
350:   {nullptr} /* sentinel */
351: };
352: 
353: static PyObject* THPErfcBackward0_self_getter(THPCppFunction *self, void *_unused) {
354:   HANDLE_TH_ERRORS
355:   const auto& prop = static_cast<ErfcBackward0*>(self->cdata.get())->self_;
356:   return THPVariable_Wrap(prop.unpack(self->cdata));
357:   END_HANDLE_TH_ERRORS
358: }
359: 
360: static PyObject* THPErfcBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
361:   HANDLE_TH_ERRORS
362:   const auto& prop = static_cast<ErfcBackward0*>(self->cdata.get())->self_;
363:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
364:   return obj.release().ptr();
365:   END_HANDLE_TH_ERRORS
366: }
367: 
368: static struct PyGetSetDef ErfcBackward0_properties[] = {
369:   THP_FUNCTION_DEFAULT_PROPERTIES,
370:   {(char*)"_saved_self", (getter)THPErfcBackward0_self_getter, nullptr, nullptr, nullptr},
371:   {(char*)"_raw_saved_self", (getter)THPErfcBackward0_self_raw_getter, nullptr, nullptr, nullptr},
372:   {nullptr} /* sentinel */
373: };
374: 
375: static PyObject* THPSpecialErfcxBackward0_self_getter(THPCppFunction *self, void *_unused) {
376:   HANDLE_TH_ERRORS
377:   const auto& prop = static_cast<SpecialErfcxBackward0*>(self->cdata.get())->self_;
378:   return THPVariable_Wrap(prop.unpack(self->cdata));
379:   END_HANDLE_TH_ERRORS
380: }
381: 
382: static PyObject* THPSpecialErfcxBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
383:   HANDLE_TH_ERRORS
384:   const auto& prop = static_cast<SpecialErfcxBackward0*>(self->cdata.get())->self_;
385:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
386:   return obj.release().ptr();
387:   END_HANDLE_TH_ERRORS
388: }
389: 
390: static PyObject* THPSpecialErfcxBackward0_result_getter(THPCppFunction *self, void *_unused) {
391:   HANDLE_TH_ERRORS
392:   const auto& prop = static_cast<SpecialErfcxBackward0*>(self->cdata.get())->result_;
393:   return THPVariable_Wrap(prop.unpack(self->cdata));
394:   END_HANDLE_TH_ERRORS
395: }
396: 
397: static PyObject* THPSpecialErfcxBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
398:   HANDLE_TH_ERRORS
399:   const auto& prop = static_cast<SpecialErfcxBackward0*>(self->cdata.get())->result_;
400:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
```

- EN: The main execution path in this span is carried by `cast`, `THPDistBackward0_result_getter`, `THPVariable_Wrap`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `cast`, `THPDistBackward0_result_getter`, `THPVariable_Wrap` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 401-480

```cpp
401:   return obj.release().ptr();
402:   END_HANDLE_TH_ERRORS
403: }
404: 
405: static struct PyGetSetDef SpecialErfcxBackward0_properties[] = {
406:   THP_FUNCTION_DEFAULT_PROPERTIES,
407:   {(char*)"_saved_self", (getter)THPSpecialErfcxBackward0_self_getter, nullptr, nullptr, nullptr},
408:   {(char*)"_raw_saved_self", (getter)THPSpecialErfcxBackward0_self_raw_getter, nullptr, nullptr, nullptr},
409:   {(char*)"_saved_result", (getter)THPSpecialErfcxBackward0_result_getter, nullptr, nullptr, nullptr},
410:   {(char*)"_raw_saved_result", (getter)THPSpecialErfcxBackward0_result_raw_getter, nullptr, nullptr, nullptr},
411:   {nullptr} /* sentinel */
412: };
413: 
414: static PyObject* THPErfinvBackward0_self_getter(THPCppFunction *self, void *_unused) {
415:   HANDLE_TH_ERRORS
416:   const auto& prop = static_cast<ErfinvBackward0*>(self->cdata.get())->self_;
417:   return THPVariable_Wrap(prop.unpack(self->cdata));
418:   END_HANDLE_TH_ERRORS
419: }
420: 
421: static PyObject* THPErfinvBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
422:   HANDLE_TH_ERRORS
423:   const auto& prop = static_cast<ErfinvBackward0*>(self->cdata.get())->self_;
424:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
425:   return obj.release().ptr();
426:   END_HANDLE_TH_ERRORS
427: }
428: 
429: static struct PyGetSetDef ErfinvBackward0_properties[] = {
430:   THP_FUNCTION_DEFAULT_PROPERTIES,
431:   {(char*)"_saved_self", (getter)THPErfinvBackward0_self_getter, nullptr, nullptr, nullptr},
432:   {(char*)"_raw_saved_self", (getter)THPErfinvBackward0_self_raw_getter, nullptr, nullptr, nullptr},
433:   {nullptr} /* sentinel */
434: };
435: 
436: 
437: 
438: static struct PyGetSetDef FmodBackward0_properties[] = {
439:   THP_FUNCTION_DEFAULT_PROPERTIES,
440: 
441:   {nullptr} /* sentinel */
442: };
443: 
444: static PyObject* THPFmodBackward1_other_getter(THPCppFunction *self, void *_unused) {
445:   HANDLE_TH_ERRORS
446:   const auto& prop = static_cast<FmodBackward1*>(self->cdata.get())->other_;
447:   return THPVariable_Wrap(prop.unpack(self->cdata));
448:   END_HANDLE_TH_ERRORS
449: }
450: 
451: static PyObject* THPFmodBackward1_other_raw_getter(THPCppFunction *self, void *_unused) {
452:   HANDLE_TH_ERRORS
453:   const auto& prop = static_cast<FmodBackward1*>(self->cdata.get())->other_;
454:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
455:   return obj.release().ptr();
456:   END_HANDLE_TH_ERRORS
457: }
458: 
459: static PyObject* THPFmodBackward1_self_getter(THPCppFunction *self, void *_unused) {
460:   HANDLE_TH_ERRORS
461:   const auto& prop = static_cast<FmodBackward1*>(self->cdata.get())->self_;
462:   return THPVariable_Wrap(prop.unpack(self->cdata));
463:   END_HANDLE_TH_ERRORS
464: }
465: 
466: static PyObject* THPFmodBackward1_self_raw_getter(THPCppFunction *self, void *_unused) {
467:   HANDLE_TH_ERRORS
468:   const auto& prop = static_cast<FmodBackward1*>(self->cdata.get())->self_;
469:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
470:   return obj.release().ptr();
471:   END_HANDLE_TH_ERRORS
472: }
473: 
474: static struct PyGetSetDef FmodBackward1_properties[] = {
475:   THP_FUNCTION_DEFAULT_PROPERTIES,
476:   {(char*)"_saved_other", (getter)THPFmodBackward1_other_getter, nullptr, nullptr, nullptr},
477:   {(char*)"_raw_saved_other", (getter)THPFmodBackward1_other_raw_getter, nullptr, nullptr, nullptr},
478:   {(char*)"_saved_self", (getter)THPFmodBackward1_self_getter, nullptr, nullptr, nullptr},
479:   {(char*)"_raw_saved_self", (getter)THPFmodBackward1_self_raw_getter, nullptr, nullptr, nullptr},
480:   {nullptr} /* sentinel */
```

- EN: The main execution path in this span is carried by `THPErfinvBackward0_self_getter`, `THPVariable_Wrap`, `THPErfinvBackward0_self_raw_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPErfinvBackward0_self_getter`, `THPVariable_Wrap`, `THPErfinvBackward0_self_raw_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 481-560

```cpp
481: };
482: 
483: 
484: 
485: static struct PyGetSetDef GeqrfBackward0_properties[] = {
486:   THP_FUNCTION_DEFAULT_PROPERTIES,
487: 
488:   {nullptr} /* sentinel */
489: };
490: 
491: static PyObject* THPGridSampler2DBackward0_align_corners_getter(THPCppFunction *self, void *_unused) {
492:   HANDLE_TH_ERRORS
493:   auto prop = static_cast<GridSampler2DBackward0*>(self->cdata.get())->align_corners;
494:   if (prop) {
495:     Py_RETURN_TRUE;
496:   } else {
497:     Py_RETURN_FALSE;
498:   }
499:   END_HANDLE_TH_ERRORS
500: }
501: 
502: static PyObject* THPGridSampler2DBackward0_grid_getter(THPCppFunction *self, void *_unused) {
503:   HANDLE_TH_ERRORS
504:   const auto& prop = static_cast<GridSampler2DBackward0*>(self->cdata.get())->grid_;
505:   return THPVariable_Wrap(prop.unpack(self->cdata));
506:   END_HANDLE_TH_ERRORS
507: }
508: 
509: static PyObject* THPGridSampler2DBackward0_grid_raw_getter(THPCppFunction *self, void *_unused) {
510:   HANDLE_TH_ERRORS
511:   const auto& prop = static_cast<GridSampler2DBackward0*>(self->cdata.get())->grid_;
512:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
513:   return obj.release().ptr();
514:   END_HANDLE_TH_ERRORS
515: }
516: 
517: static PyObject* THPGridSampler2DBackward0_input_getter(THPCppFunction *self, void *_unused) {
518:   HANDLE_TH_ERRORS
519:   const auto& prop = static_cast<GridSampler2DBackward0*>(self->cdata.get())->input_;
520:   return THPVariable_Wrap(prop.unpack(self->cdata));
521:   END_HANDLE_TH_ERRORS
522: }
523: 
524: static PyObject* THPGridSampler2DBackward0_input_raw_getter(THPCppFunction *self, void *_unused) {
525:   HANDLE_TH_ERRORS
526:   const auto& prop = static_cast<GridSampler2DBackward0*>(self->cdata.get())->input_;
527:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
528:   return obj.release().ptr();
529:   END_HANDLE_TH_ERRORS
530: }
531: 
532: static PyObject* THPGridSampler2DBackward0_interpolation_mode_getter(THPCppFunction *self, void *_unused) {
533:   HANDLE_TH_ERRORS
534:   auto prop = static_cast<GridSampler2DBackward0*>(self->cdata.get())->interpolation_mode;
535:   return PyLong_FromUnsignedLong((int64_t) prop);
536:   END_HANDLE_TH_ERRORS
537: }
538: 
539: static PyObject* THPGridSampler2DBackward0_padding_mode_getter(THPCppFunction *self, void *_unused) {
540:   HANDLE_TH_ERRORS
541:   auto prop = static_cast<GridSampler2DBackward0*>(self->cdata.get())->padding_mode;
542:   return PyLong_FromUnsignedLong((int64_t) prop);
543:   END_HANDLE_TH_ERRORS
544: }
545: 
546: static struct PyGetSetDef GridSampler2DBackward0_properties[] = {
547:   THP_FUNCTION_DEFAULT_PROPERTIES,
548:   {(char*)"_saved_align_corners", (getter)THPGridSampler2DBackward0_align_corners_getter, nullptr, nullptr, nullptr},
549:   {(char*)"_saved_grid", (getter)THPGridSampler2DBackward0_grid_getter, nullptr, nullptr, nullptr},
550:   {(char*)"_raw_saved_grid", (getter)THPGridSampler2DBackward0_grid_raw_getter, nullptr, nullptr, nullptr},
551:   {(char*)"_saved_input", (getter)THPGridSampler2DBackward0_input_getter, nullptr, nullptr, nullptr},
552:   {(char*)"_raw_saved_input", (getter)THPGridSampler2DBackward0_input_raw_getter, nullptr, nullptr, nullptr},
553:   {(char*)"_saved_interpolation_mode", (getter)THPGridSampler2DBackward0_interpolation_mode_getter, nullptr, nullptr, nullptr},
554:   {(char*)"_saved_padding_mode", (getter)THPGridSampler2DBackward0_padding_mode_getter, nullptr, nullptr, nullptr},
555:   {nullptr} /* sentinel */
556: };
557: 
558: static PyObject* THPGridSampler2DCpuFallbackBackwardBackward0_align_corners_getter(THPCppFunction *self, void *_unused) {
559:   HANDLE_TH_ERRORS
560:   auto prop = static_cast<GridSampler2DCpuFallbackBackwardBackward0*>(self->cdata.get())->align_corners;
```

- EN: The main execution path in this span is carried by `THPGridSampler2DBackward0_align_corners_getter`, `THPGridSampler2DBackward0_grid_getter`, `THPVariable_Wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPGridSampler2DBackward0_align_corners_getter`, `THPGridSampler2DBackward0_grid_getter`, `THPVariable_Wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 561-640

```cpp
561:   if (prop) {
562:     Py_RETURN_TRUE;
563:   } else {
564:     Py_RETURN_FALSE;
565:   }
566:   END_HANDLE_TH_ERRORS
567: }
568: 
569: static PyObject* THPGridSampler2DCpuFallbackBackwardBackward0_grad_output_getter(THPCppFunction *self, void *_unused) {
570:   HANDLE_TH_ERRORS
571:   const auto& prop = static_cast<GridSampler2DCpuFallbackBackwardBackward0*>(self->cdata.get())->grad_output_;
572:   return THPVariable_Wrap(prop.unpack(self->cdata));
573:   END_HANDLE_TH_ERRORS
574: }
575: 
576: static PyObject* THPGridSampler2DCpuFallbackBackwardBackward0_grad_output_raw_getter(THPCppFunction *self, void *_unused) {
577:   HANDLE_TH_ERRORS
578:   const auto& prop = static_cast<GridSampler2DCpuFallbackBackwardBackward0*>(self->cdata.get())->grad_output_;
579:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
580:   return obj.release().ptr();
581:   END_HANDLE_TH_ERRORS
582: }
583: 
584: static PyObject* THPGridSampler2DCpuFallbackBackwardBackward0_grid_getter(THPCppFunction *self, void *_unused) {
585:   HANDLE_TH_ERRORS
586:   const auto& prop = static_cast<GridSampler2DCpuFallbackBackwardBackward0*>(self->cdata.get())->grid_;
587:   return THPVariable_Wrap(prop.unpack(self->cdata));
588:   END_HANDLE_TH_ERRORS
589: }
590: 
591: static PyObject* THPGridSampler2DCpuFallbackBackwardBackward0_grid_raw_getter(THPCppFunction *self, void *_unused) {
592:   HANDLE_TH_ERRORS
593:   const auto& prop = static_cast<GridSampler2DCpuFallbackBackwardBackward0*>(self->cdata.get())->grid_;
594:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
595:   return obj.release().ptr();
596:   END_HANDLE_TH_ERRORS
597: }
598: 
599: static PyObject* THPGridSampler2DCpuFallbackBackwardBackward0_input_getter(THPCppFunction *self, void *_unused) {
600:   HANDLE_TH_ERRORS
601:   const auto& prop = static_cast<GridSampler2DCpuFallbackBackwardBackward0*>(self->cdata.get())->input_;
602:   return THPVariable_Wrap(prop.unpack(self->cdata));
603:   END_HANDLE_TH_ERRORS
604: }
605: 
606: static PyObject* THPGridSampler2DCpuFallbackBackwardBackward0_input_raw_getter(THPCppFunction *self, void *_unused) {
607:   HANDLE_TH_ERRORS
608:   const auto& prop = static_cast<GridSampler2DCpuFallbackBackwardBackward0*>(self->cdata.get())->input_;
609:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
610:   return obj.release().ptr();
611:   END_HANDLE_TH_ERRORS
612: }
613: 
614: static PyObject* THPGridSampler2DCpuFallbackBackwardBackward0_interpolation_mode_getter(THPCppFunction *self, void *_unused) {
615:   HANDLE_TH_ERRORS
616:   auto prop = static_cast<GridSampler2DCpuFallbackBackwardBackward0*>(self->cdata.get())->interpolation_mode;
617:   return PyLong_FromUnsignedLong((int64_t) prop);
618:   END_HANDLE_TH_ERRORS
619: }
620: 
621: static PyObject* THPGridSampler2DCpuFallbackBackwardBackward0_padding_mode_getter(THPCppFunction *self, void *_unused) {
622:   HANDLE_TH_ERRORS
623:   auto prop = static_cast<GridSampler2DCpuFallbackBackwardBackward0*>(self->cdata.get())->padding_mode;
624:   return PyLong_FromUnsignedLong((int64_t) prop);
625:   END_HANDLE_TH_ERRORS
626: }
627: 
628: static struct PyGetSetDef GridSampler2DCpuFallbackBackwardBackward0_properties[] = {
629:   THP_FUNCTION_DEFAULT_PROPERTIES,
630:   {(char*)"_saved_align_corners", (getter)THPGridSampler2DCpuFallbackBackwardBackward0_align_corners_getter, nullptr, nullptr, nullptr},
631:   {(char*)"_saved_grad_output", (getter)THPGridSampler2DCpuFallbackBackwardBackward0_grad_output_getter, nullptr, nullptr, nullptr},
632:   {(char*)"_raw_saved_grad_output", (getter)THPGridSampler2DCpuFallbackBackwardBackward0_grad_output_raw_getter, nullptr, nullptr, nullptr},
633:   {(char*)"_saved_grid", (getter)THPGridSampler2DCpuFallbackBackwardBackward0_grid_getter, nullptr, nullptr, nullptr},
634:   {(char*)"_raw_saved_grid", (getter)THPGridSampler2DCpuFallbackBackwardBackward0_grid_raw_getter, nullptr, nullptr, nullptr},
635:   {(char*)"_saved_input", (getter)THPGridSampler2DCpuFallbackBackwardBackward0_input_getter, nullptr, nullptr, nullptr},
636:   {(char*)"_raw_saved_input", (getter)THPGridSampler2DCpuFallbackBackwardBackward0_input_raw_getter, nullptr, nullptr, nullptr},
637:   {(char*)"_saved_interpolation_mode", (getter)THPGridSampler2DCpuFallbackBackwardBackward0_interpolation_mode_getter, nullptr, nullptr, nullptr},
638:   {(char*)"_saved_padding_mode", (getter)THPGridSampler2DCpuFallbackBackwardBackward0_padding_mode_getter, nullptr, nullptr, nullptr},
639:   {nullptr} /* sentinel */
640: };
```

- EN: The main execution path in this span is carried by `THPGridSampler2DCpuFallbackBackwardBackward0_grad_output_getter`, `THPVariable_Wrap`, `THPGridSampler2DCpuFallbackBackwardBackward0_grad_output_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPGridSampler2DCpuFallbackBackwardBackward0_grad_output_getter`, `THPVariable_Wrap`, `THPGridSampler2DCpuFallbackBackwardBackward0_grad_output_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 641-720

```cpp
641: 
642: 
643: 
644: static struct PyGetSetDef GtBackward0_properties[] = {
645:   THP_FUNCTION_DEFAULT_PROPERTIES,
646: 
647:   {nullptr} /* sentinel */
648: };
649: 
650: 
651: 
652: static struct PyGetSetDef GtBackward1_properties[] = {
653:   THP_FUNCTION_DEFAULT_PROPERTIES,
654: 
655:   {nullptr} /* sentinel */
656: };
657: 
658: static PyObject* THPSpecialI0EBackward0_self_getter(THPCppFunction *self, void *_unused) {
659:   HANDLE_TH_ERRORS
660:   const auto& prop = static_cast<SpecialI0EBackward0*>(self->cdata.get())->self_;
661:   return THPVariable_Wrap(prop.unpack(self->cdata));
662:   END_HANDLE_TH_ERRORS
663: }
664: 
665: static PyObject* THPSpecialI0EBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
666:   HANDLE_TH_ERRORS
667:   const auto& prop = static_cast<SpecialI0EBackward0*>(self->cdata.get())->self_;
668:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
669:   return obj.release().ptr();
670:   END_HANDLE_TH_ERRORS
671: }
672: 
673: static PyObject* THPSpecialI0EBackward0_result_getter(THPCppFunction *self, void *_unused) {
674:   HANDLE_TH_ERRORS
675:   const auto& prop = static_cast<SpecialI0EBackward0*>(self->cdata.get())->result_;
676:   return THPVariable_Wrap(prop.unpack(self->cdata));
677:   END_HANDLE_TH_ERRORS
678: }
679: 
680: static PyObject* THPSpecialI0EBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
681:   HANDLE_TH_ERRORS
682:   const auto& prop = static_cast<SpecialI0EBackward0*>(self->cdata.get())->result_;
683:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
684:   return obj.release().ptr();
685:   END_HANDLE_TH_ERRORS
686: }
687: 
688: static struct PyGetSetDef SpecialI0EBackward0_properties[] = {
689:   THP_FUNCTION_DEFAULT_PROPERTIES,
690:   {(char*)"_saved_self", (getter)THPSpecialI0EBackward0_self_getter, nullptr, nullptr, nullptr},
691:   {(char*)"_raw_saved_self", (getter)THPSpecialI0EBackward0_self_raw_getter, nullptr, nullptr, nullptr},
692:   {(char*)"_saved_result", (getter)THPSpecialI0EBackward0_result_getter, nullptr, nullptr, nullptr},
693:   {(char*)"_raw_saved_result", (getter)THPSpecialI0EBackward0_result_raw_getter, nullptr, nullptr, nullptr},
694:   {nullptr} /* sentinel */
695: };
696: 
697: static PyObject* THPSpecialI1Backward0_self_getter(THPCppFunction *self, void *_unused) {
698:   HANDLE_TH_ERRORS
699:   const auto& prop = static_cast<SpecialI1Backward0*>(self->cdata.get())->self_;
700:   return THPVariable_Wrap(prop.unpack(self->cdata));
701:   END_HANDLE_TH_ERRORS
702: }
703: 
704: static PyObject* THPSpecialI1Backward0_self_raw_getter(THPCppFunction *self, void *_unused) {
705:   HANDLE_TH_ERRORS
706:   const auto& prop = static_cast<SpecialI1Backward0*>(self->cdata.get())->self_;
707:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
708:   return obj.release().ptr();
709:   END_HANDLE_TH_ERRORS
710: }
711: 
712: static PyObject* THPSpecialI1Backward0_result_getter(THPCppFunction *self, void *_unused) {
713:   HANDLE_TH_ERRORS
714:   const auto& prop = static_cast<SpecialI1Backward0*>(self->cdata.get())->result_;
715:   return THPVariable_Wrap(prop.unpack(self->cdata));
716:   END_HANDLE_TH_ERRORS
717: }
718: 
719: static PyObject* THPSpecialI1Backward0_result_raw_getter(THPCppFunction *self, void *_unused) {
720:   HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPSpecialI0EBackward0_self_getter`, `THPVariable_Wrap`, `THPSpecialI0EBackward0_self_raw_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPSpecialI0EBackward0_self_getter`, `THPVariable_Wrap`, `THPSpecialI0EBackward0_self_raw_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 721-800

```cpp
721:   const auto& prop = static_cast<SpecialI1Backward0*>(self->cdata.get())->result_;
722:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
723:   return obj.release().ptr();
724:   END_HANDLE_TH_ERRORS
725: }
726: 
727: static struct PyGetSetDef SpecialI1Backward0_properties[] = {
728:   THP_FUNCTION_DEFAULT_PROPERTIES,
729:   {(char*)"_saved_self", (getter)THPSpecialI1Backward0_self_getter, nullptr, nullptr, nullptr},
730:   {(char*)"_raw_saved_self", (getter)THPSpecialI1Backward0_self_raw_getter, nullptr, nullptr, nullptr},
731:   {(char*)"_saved_result", (getter)THPSpecialI1Backward0_result_getter, nullptr, nullptr, nullptr},
732:   {(char*)"_raw_saved_result", (getter)THPSpecialI1Backward0_result_raw_getter, nullptr, nullptr, nullptr},
733:   {nullptr} /* sentinel */
734: };
735: 
736: static PyObject* THPUnsafeMaskedIndexPutAccumulateBackward0_indices_getter(THPCppFunction *self, void *_unused) {
737:   HANDLE_TH_ERRORS
738:   const auto *node = static_cast<UnsafeMaskedIndexPutAccumulateBackward0*>(self->cdata.get());
739:   const auto& prop = node->indices_;
740:   if (node->indices_released_) {
741:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
742:     return nullptr;
743:   }
744:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
745:   for (auto i: c10::irange(prop.size())) {
746:     PyTuple_SetItem(tup, (Py_ssize_t) i, THPVariable_Wrap(prop[i].unpack(self->cdata)));
747:   }
748:   return tup;
749:   END_HANDLE_TH_ERRORS
750: }
751: 
752: static PyObject* THPUnsafeMaskedIndexPutAccumulateBackward0_indices_raw_getter(THPCppFunction *self, void *_unused) {
753:   HANDLE_TH_ERRORS
754:   const auto *node = static_cast<UnsafeMaskedIndexPutAccumulateBackward0*>(self->cdata.get());
755:   const auto& prop = node->indices_;
756:   if (node->indices_released_) {
757:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
758:     return nullptr;
759:   }
760:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
761:   for (auto i : c10::irange(prop.size())) {
762:     pybind11::object obj = pybind11::cast(prop[i], pybind11::return_value_policy::reference);
763:     PyTuple_SetItem(tup, (Py_ssize_t) i, obj.release().ptr());
764:   }
765:   return tup;
766:   END_HANDLE_TH_ERRORS
767: }
768: 
769: static PyObject* THPUnsafeMaskedIndexPutAccumulateBackward0_mask_getter(THPCppFunction *self, void *_unused) {
770:   HANDLE_TH_ERRORS
771:   const auto& prop = static_cast<UnsafeMaskedIndexPutAccumulateBackward0*>(self->cdata.get())->mask_;
772:   return THPVariable_Wrap(prop.unpack(self->cdata));
773:   END_HANDLE_TH_ERRORS
774: }
775: 
776: static PyObject* THPUnsafeMaskedIndexPutAccumulateBackward0_mask_raw_getter(THPCppFunction *self, void *_unused) {
777:   HANDLE_TH_ERRORS
778:   const auto& prop = static_cast<UnsafeMaskedIndexPutAccumulateBackward0*>(self->cdata.get())->mask_;
779:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
780:   return obj.release().ptr();
781:   END_HANDLE_TH_ERRORS
782: }
783: 
784: static struct PyGetSetDef UnsafeMaskedIndexPutAccumulateBackward0_properties[] = {
785:   THP_FUNCTION_DEFAULT_PROPERTIES,
786:   {(char*)"_saved_indices", (getter)THPUnsafeMaskedIndexPutAccumulateBackward0_indices_getter, nullptr, nullptr, nullptr},
787:   {(char*)"_raw_saved_indices", (getter)THPUnsafeMaskedIndexPutAccumulateBackward0_indices_raw_getter, nullptr, nullptr, nullptr},
788:   {(char*)"_saved_mask", (getter)THPUnsafeMaskedIndexPutAccumulateBackward0_mask_getter, nullptr, nullptr, nullptr},
789:   {(char*)"_raw_saved_mask", (getter)THPUnsafeMaskedIndexPutAccumulateBackward0_mask_raw_getter, nullptr, nullptr, nullptr},
790:   {nullptr} /* sentinel */
791: };
792: 
793: static PyObject* THPIndexAddBackward0_alpha_getter(THPCppFunction *self, void *_unused) {
794:   HANDLE_TH_ERRORS
795:   auto prop = static_cast<IndexAddBackward0*>(self->cdata.get())->alpha;
796:   if (prop.isComplex()) {
797:     auto cprop = prop.to<c10::complex<double>>();
798:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
799:   } else if (prop.isFloatingPoint()) {
800:     return PyFloat_FromDouble(prop.to<double>());
```

- EN: The main execution path in this span is carried by `cast`, `THPUnsafeMaskedIndexPutAccumulateBackward0_indices_getter`, `PyErr_SetString`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `cast`, `THPUnsafeMaskedIndexPutAccumulateBackward0_indices_getter`, `PyErr_SetString` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 801-880

```cpp
801:   } else if (prop.isIntegral(/*includeBool=*/false)) {
802:     return PyLong_FromLong(prop.to<int64_t>());
803:   } else if (prop.isBoolean()) {
804:     if (prop.to<bool>()) {
805:       Py_RETURN_TRUE;
806:     } else {
807:       Py_RETURN_FALSE;
808:     }
809:   } else {
810:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
811:     return nullptr;
812:   }
813:   END_HANDLE_TH_ERRORS
814: }
815: 
816: static PyObject* THPIndexAddBackward0_dim_getter(THPCppFunction *self, void *_unused) {
817:   HANDLE_TH_ERRORS
818:   auto prop = static_cast<IndexAddBackward0*>(self->cdata.get())->dim;
819:   return PyLong_FromUnsignedLong((int64_t) prop);
820:   END_HANDLE_TH_ERRORS
821: }
822: 
823: static PyObject* THPIndexAddBackward0_index_getter(THPCppFunction *self, void *_unused) {
824:   HANDLE_TH_ERRORS
825:   const auto& prop = static_cast<IndexAddBackward0*>(self->cdata.get())->index_;
826:   return THPVariable_Wrap(prop.unpack(self->cdata));
827:   END_HANDLE_TH_ERRORS
828: }
829: 
830: static PyObject* THPIndexAddBackward0_index_raw_getter(THPCppFunction *self, void *_unused) {
831:   HANDLE_TH_ERRORS
832:   const auto& prop = static_cast<IndexAddBackward0*>(self->cdata.get())->index_;
833:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
834:   return obj.release().ptr();
835:   END_HANDLE_TH_ERRORS
836: }
837: 
838: static PyObject* THPIndexAddBackward0_source_getter(THPCppFunction *self, void *_unused) {
839:   HANDLE_TH_ERRORS
840:   const auto& prop = static_cast<IndexAddBackward0*>(self->cdata.get())->source_;
841:   return THPVariable_Wrap(prop.unpack(self->cdata));
842:   END_HANDLE_TH_ERRORS
843: }
844: 
845: static PyObject* THPIndexAddBackward0_source_raw_getter(THPCppFunction *self, void *_unused) {
846:   HANDLE_TH_ERRORS
847:   const auto& prop = static_cast<IndexAddBackward0*>(self->cdata.get())->source_;
848:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
849:   return obj.release().ptr();
850:   END_HANDLE_TH_ERRORS
851: }
852: 
853: static PyObject* THPIndexAddBackward0_source_dim_getter(THPCppFunction *self, void *_unused) {
854:   HANDLE_TH_ERRORS
855:   auto prop = static_cast<IndexAddBackward0*>(self->cdata.get())->source_dim;
856:   return PyLong_FromUnsignedLong((int64_t) prop);
857:   END_HANDLE_TH_ERRORS
858: }
859: 
860: static struct PyGetSetDef IndexAddBackward0_properties[] = {
861:   THP_FUNCTION_DEFAULT_PROPERTIES,
862:   {(char*)"_saved_alpha", (getter)THPIndexAddBackward0_alpha_getter, nullptr, nullptr, nullptr},
863:   {(char*)"_saved_dim", (getter)THPIndexAddBackward0_dim_getter, nullptr, nullptr, nullptr},
864:   {(char*)"_saved_index", (getter)THPIndexAddBackward0_index_getter, nullptr, nullptr, nullptr},
865:   {(char*)"_raw_saved_index", (getter)THPIndexAddBackward0_index_raw_getter, nullptr, nullptr, nullptr},
866:   {(char*)"_saved_source", (getter)THPIndexAddBackward0_source_getter, nullptr, nullptr, nullptr},
867:   {(char*)"_raw_saved_source", (getter)THPIndexAddBackward0_source_raw_getter, nullptr, nullptr, nullptr},
868:   {(char*)"_saved_source_dim", (getter)THPIndexAddBackward0_source_dim_getter, nullptr, nullptr, nullptr},
869:   {nullptr} /* sentinel */
870: };
871: 
872: static PyObject* THPIndexPutBackward0_accumulate_getter(THPCppFunction *self, void *_unused) {
873:   HANDLE_TH_ERRORS
874:   auto prop = static_cast<IndexPutBackward0*>(self->cdata.get())->accumulate;
875:   if (prop) {
876:     Py_RETURN_TRUE;
877:   } else {
878:     Py_RETURN_FALSE;
879:   }
880:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `PyLong_FromLong`, `PyErr_SetString`, `THPIndexAddBackward0_dim_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyLong_FromLong`, `PyErr_SetString`, `THPIndexAddBackward0_dim_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 881-960

```cpp
881: }
882: 
883: static PyObject* THPIndexPutBackward0_indices_getter(THPCppFunction *self, void *_unused) {
884:   HANDLE_TH_ERRORS
885:   const auto *node = static_cast<IndexPutBackward0*>(self->cdata.get());
886:   const auto& prop = node->indices_;
887:   if (node->indices_released_) {
888:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
889:     return nullptr;
890:   }
891:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
892:   for (auto i: c10::irange(prop.size())) {
893:     PyTuple_SetItem(tup, (Py_ssize_t) i, THPVariable_Wrap(prop[i].unpack(self->cdata)));
894:   }
895:   return tup;
896:   END_HANDLE_TH_ERRORS
897: }
898: 
899: static PyObject* THPIndexPutBackward0_indices_raw_getter(THPCppFunction *self, void *_unused) {
900:   HANDLE_TH_ERRORS
901:   const auto *node = static_cast<IndexPutBackward0*>(self->cdata.get());
902:   const auto& prop = node->indices_;
903:   if (node->indices_released_) {
904:     PyErr_SetString(PyExc_RuntimeError, ERR_BACKWARD_TWICE);
905:     return nullptr;
906:   }
907:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
908:   for (auto i : c10::irange(prop.size())) {
909:     pybind11::object obj = pybind11::cast(prop[i], pybind11::return_value_policy::reference);
910:     PyTuple_SetItem(tup, (Py_ssize_t) i, obj.release().ptr());
911:   }
912:   return tup;
913:   END_HANDLE_TH_ERRORS
914: }
915: 
916: static struct PyGetSetDef IndexPutBackward0_properties[] = {
917:   THP_FUNCTION_DEFAULT_PROPERTIES,
918:   {(char*)"_saved_accumulate", (getter)THPIndexPutBackward0_accumulate_getter, nullptr, nullptr, nullptr},
919:   {(char*)"_saved_indices", (getter)THPIndexPutBackward0_indices_getter, nullptr, nullptr, nullptr},
920:   {(char*)"_raw_saved_indices", (getter)THPIndexPutBackward0_indices_raw_getter, nullptr, nullptr, nullptr},
921:   {nullptr} /* sentinel */
922: };
923: 
924: 
925: 
926: static struct PyGetSetDef LogNormalBackward0_properties[] = {
927:   THP_FUNCTION_DEFAULT_PROPERTIES,
928: 
929:   {nullptr} /* sentinel */
930: };
931: 
932: static PyObject* THPLinalgLuSolveBackward0_LU_getter(THPCppFunction *self, void *_unused) {
933:   HANDLE_TH_ERRORS
934:   const auto& prop = static_cast<LinalgLuSolveBackward0*>(self->cdata.get())->LU_;
935:   return THPVariable_Wrap(prop.unpack(self->cdata));
936:   END_HANDLE_TH_ERRORS
937: }
938: 
939: static PyObject* THPLinalgLuSolveBackward0_LU_raw_getter(THPCppFunction *self, void *_unused) {
940:   HANDLE_TH_ERRORS
941:   const auto& prop = static_cast<LinalgLuSolveBackward0*>(self->cdata.get())->LU_;
942:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
943:   return obj.release().ptr();
944:   END_HANDLE_TH_ERRORS
945: }
946: 
947: static PyObject* THPLinalgLuSolveBackward0_adjoint_getter(THPCppFunction *self, void *_unused) {
948:   HANDLE_TH_ERRORS
949:   auto prop = static_cast<LinalgLuSolveBackward0*>(self->cdata.get())->adjoint;
950:   if (prop) {
951:     Py_RETURN_TRUE;
952:   } else {
953:     Py_RETURN_FALSE;
954:   }
955:   END_HANDLE_TH_ERRORS
956: }
957: 
958: static PyObject* THPLinalgLuSolveBackward0_left_getter(THPCppFunction *self, void *_unused) {
959:   HANDLE_TH_ERRORS
960:   auto prop = static_cast<LinalgLuSolveBackward0*>(self->cdata.get())->left;
```

- EN: The main execution path in this span is carried by `THPIndexPutBackward0_indices_getter`, `PyErr_SetString`, `PyTuple_New`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPIndexPutBackward0_indices_getter`, `PyErr_SetString`, `PyTuple_New` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 961-1040

```cpp
 961:   if (prop) {
 962:     Py_RETURN_TRUE;
 963:   } else {
 964:     Py_RETURN_FALSE;
 965:   }
 966:   END_HANDLE_TH_ERRORS
 967: }
 968: 
 969: static PyObject* THPLinalgLuSolveBackward0_pivots_getter(THPCppFunction *self, void *_unused) {
 970:   HANDLE_TH_ERRORS
 971:   const auto& prop = static_cast<LinalgLuSolveBackward0*>(self->cdata.get())->pivots_;
 972:   return THPVariable_Wrap(prop.unpack(self->cdata));
 973:   END_HANDLE_TH_ERRORS
 974: }
 975: 
 976: static PyObject* THPLinalgLuSolveBackward0_pivots_raw_getter(THPCppFunction *self, void *_unused) {
 977:   HANDLE_TH_ERRORS
 978:   const auto& prop = static_cast<LinalgLuSolveBackward0*>(self->cdata.get())->pivots_;
 979:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
 980:   return obj.release().ptr();
 981:   END_HANDLE_TH_ERRORS
 982: }
 983: 
 984: static PyObject* THPLinalgLuSolveBackward0_result_getter(THPCppFunction *self, void *_unused) {
 985:   HANDLE_TH_ERRORS
 986:   const auto& prop = static_cast<LinalgLuSolveBackward0*>(self->cdata.get())->result_;
 987:   return THPVariable_Wrap(prop.unpack(self->cdata));
 988:   END_HANDLE_TH_ERRORS
 989: }
 990: 
 991: static PyObject* THPLinalgLuSolveBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
 992:   HANDLE_TH_ERRORS
 993:   const auto& prop = static_cast<LinalgLuSolveBackward0*>(self->cdata.get())->result_;
 994:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
 995:   return obj.release().ptr();
 996:   END_HANDLE_TH_ERRORS
 997: }
 998: 
 999: static struct PyGetSetDef LinalgLuSolveBackward0_properties[] = {
1000:   THP_FUNCTION_DEFAULT_PROPERTIES,
1001:   {(char*)"_saved_LU", (getter)THPLinalgLuSolveBackward0_LU_getter, nullptr, nullptr, nullptr},
1002:   {(char*)"_raw_saved_LU", (getter)THPLinalgLuSolveBackward0_LU_raw_getter, nullptr, nullptr, nullptr},
1003:   {(char*)"_saved_adjoint", (getter)THPLinalgLuSolveBackward0_adjoint_getter, nullptr, nullptr, nullptr},
1004:   {(char*)"_saved_left", (getter)THPLinalgLuSolveBackward0_left_getter, nullptr, nullptr, nullptr},
1005:   {(char*)"_saved_pivots", (getter)THPLinalgLuSolveBackward0_pivots_getter, nullptr, nullptr, nullptr},
1006:   {(char*)"_raw_saved_pivots", (getter)THPLinalgLuSolveBackward0_pivots_raw_getter, nullptr, nullptr, nullptr},
1007:   {(char*)"_saved_result", (getter)THPLinalgLuSolveBackward0_result_getter, nullptr, nullptr, nullptr},
1008:   {(char*)"_raw_saved_result", (getter)THPLinalgLuSolveBackward0_result_raw_getter, nullptr, nullptr, nullptr},
1009:   {nullptr} /* sentinel */
1010: };
1011: 
1012: static PyObject* THPMaskedFillBackward0_mask_getter(THPCppFunction *self, void *_unused) {
1013:   HANDLE_TH_ERRORS
1014:   const auto& prop = static_cast<MaskedFillBackward0*>(self->cdata.get())->mask_;
1015:   return THPVariable_Wrap(prop.unpack(self->cdata));
1016:   END_HANDLE_TH_ERRORS
1017: }
1018: 
1019: static PyObject* THPMaskedFillBackward0_mask_raw_getter(THPCppFunction *self, void *_unused) {
1020:   HANDLE_TH_ERRORS
1021:   const auto& prop = static_cast<MaskedFillBackward0*>(self->cdata.get())->mask_;
1022:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1023:   return obj.release().ptr();
1024:   END_HANDLE_TH_ERRORS
1025: }
1026: 
1027: static struct PyGetSetDef MaskedFillBackward0_properties[] = {
1028:   THP_FUNCTION_DEFAULT_PROPERTIES,
1029:   {(char*)"_saved_mask", (getter)THPMaskedFillBackward0_mask_getter, nullptr, nullptr, nullptr},
1030:   {(char*)"_raw_saved_mask", (getter)THPMaskedFillBackward0_mask_raw_getter, nullptr, nullptr, nullptr},
1031:   {nullptr} /* sentinel */
1032: };
1033: 
1034: static PyObject* THPMaskedFillBackward1_mask_getter(THPCppFunction *self, void *_unused) {
1035:   HANDLE_TH_ERRORS
1036:   const auto& prop = static_cast<MaskedFillBackward1*>(self->cdata.get())->mask_;
1037:   return THPVariable_Wrap(prop.unpack(self->cdata));
1038:   END_HANDLE_TH_ERRORS
1039: }
1040: 
```

- EN: The main execution path in this span is carried by `THPLinalgLuSolveBackward0_pivots_getter`, `THPVariable_Wrap`, `THPLinalgLuSolveBackward0_pivots_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPLinalgLuSolveBackward0_pivots_getter`, `THPVariable_Wrap`, `THPLinalgLuSolveBackward0_pivots_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1041-1120

```cpp
1041: static PyObject* THPMaskedFillBackward1_mask_raw_getter(THPCppFunction *self, void *_unused) {
1042:   HANDLE_TH_ERRORS
1043:   const auto& prop = static_cast<MaskedFillBackward1*>(self->cdata.get())->mask_;
1044:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1045:   return obj.release().ptr();
1046:   END_HANDLE_TH_ERRORS
1047: }
1048: 
1049: static struct PyGetSetDef MaskedFillBackward1_properties[] = {
1050:   THP_FUNCTION_DEFAULT_PROPERTIES,
1051:   {(char*)"_saved_mask", (getter)THPMaskedFillBackward1_mask_getter, nullptr, nullptr, nullptr},
1052:   {(char*)"_raw_saved_mask", (getter)THPMaskedFillBackward1_mask_raw_getter, nullptr, nullptr, nullptr},
1053:   {nullptr} /* sentinel */
1054: };
1055: 
1056: static PyObject* THPMaskedScatterBackward0_mask_getter(THPCppFunction *self, void *_unused) {
1057:   HANDLE_TH_ERRORS
1058:   const auto& prop = static_cast<MaskedScatterBackward0*>(self->cdata.get())->mask_;
1059:   return THPVariable_Wrap(prop.unpack(self->cdata));
1060:   END_HANDLE_TH_ERRORS
1061: }
1062: 
1063: static PyObject* THPMaskedScatterBackward0_mask_raw_getter(THPCppFunction *self, void *_unused) {
1064:   HANDLE_TH_ERRORS
1065:   const auto& prop = static_cast<MaskedScatterBackward0*>(self->cdata.get())->mask_;
1066:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1067:   return obj.release().ptr();
1068:   END_HANDLE_TH_ERRORS
1069: }
1070: 
1071: static PyObject* THPMaskedScatterBackward0_source_sym_sizes_getter(THPCppFunction *self, void *_unused) {
1072:   HANDLE_TH_ERRORS
1073:   auto prop = static_cast<MaskedScatterBackward0*>(self->cdata.get())->source_sym_sizes;
1074:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
1075:   for (auto i : c10::irange(prop.size())) {
1076:       auto si = prop[i];
1077:       if (auto m = si.maybe_as_int()) {
1078:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
1079:       } else {
1080:         auto py_symint = py::cast(si).release().ptr();
1081:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
1082:       }
1083:   }
1084:   return tup;
1085:   END_HANDLE_TH_ERRORS
1086: }
1087: 
1088: static struct PyGetSetDef MaskedScatterBackward0_properties[] = {
1089:   THP_FUNCTION_DEFAULT_PROPERTIES,
1090:   {(char*)"_saved_mask", (getter)THPMaskedScatterBackward0_mask_getter, nullptr, nullptr, nullptr},
1091:   {(char*)"_raw_saved_mask", (getter)THPMaskedScatterBackward0_mask_raw_getter, nullptr, nullptr, nullptr},
1092:   {(char*)"_saved_source_sym_sizes", (getter)THPMaskedScatterBackward0_source_sym_sizes_getter, nullptr, nullptr, nullptr},
1093:   {nullptr} /* sentinel */
1094: };
1095: 
1096: static PyObject* THPMedianBackward0_self_getter(THPCppFunction *self, void *_unused) {
1097:   HANDLE_TH_ERRORS
1098:   const auto& prop = static_cast<MedianBackward0*>(self->cdata.get())->self_;
1099:   return THPVariable_Wrap(prop.unpack(self->cdata));
1100:   END_HANDLE_TH_ERRORS
1101: }
1102: 
1103: static PyObject* THPMedianBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
1104:   HANDLE_TH_ERRORS
1105:   const auto& prop = static_cast<MedianBackward0*>(self->cdata.get())->self_;
1106:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1107:   return obj.release().ptr();
1108:   END_HANDLE_TH_ERRORS
1109: }
1110: 
1111: static PyObject* THPMedianBackward0_result_getter(THPCppFunction *self, void *_unused) {
1112:   HANDLE_TH_ERRORS
1113:   const auto& prop = static_cast<MedianBackward0*>(self->cdata.get())->result_;
1114:   return THPVariable_Wrap(prop.unpack(self->cdata));
1115:   END_HANDLE_TH_ERRORS
1116: }
1117: 
1118: static PyObject* THPMedianBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
1119:   HANDLE_TH_ERRORS
1120:   const auto& prop = static_cast<MedianBackward0*>(self->cdata.get())->result_;
```

- EN: The main execution path in this span is carried by `THPMaskedFillBackward1_mask_raw_getter`, `cast`, `THPMaskedScatterBackward0_mask_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPMaskedFillBackward1_mask_raw_getter`, `cast`, `THPMaskedScatterBackward0_mask_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1121-1200

```cpp
1121:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1122:   return obj.release().ptr();
1123:   END_HANDLE_TH_ERRORS
1124: }
1125: 
1126: static struct PyGetSetDef MedianBackward0_properties[] = {
1127:   THP_FUNCTION_DEFAULT_PROPERTIES,
1128:   {(char*)"_saved_self", (getter)THPMedianBackward0_self_getter, nullptr, nullptr, nullptr},
1129:   {(char*)"_raw_saved_self", (getter)THPMedianBackward0_self_raw_getter, nullptr, nullptr, nullptr},
1130:   {(char*)"_saved_result", (getter)THPMedianBackward0_result_getter, nullptr, nullptr, nullptr},
1131:   {(char*)"_raw_saved_result", (getter)THPMedianBackward0_result_raw_getter, nullptr, nullptr, nullptr},
1132:   {nullptr} /* sentinel */
1133: };
1134: 
1135: static PyObject* THPNanmedianBackward0_self_getter(THPCppFunction *self, void *_unused) {
1136:   HANDLE_TH_ERRORS
1137:   const auto& prop = static_cast<NanmedianBackward0*>(self->cdata.get())->self_;
1138:   return THPVariable_Wrap(prop.unpack(self->cdata));
1139:   END_HANDLE_TH_ERRORS
1140: }
1141: 
1142: static PyObject* THPNanmedianBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
1143:   HANDLE_TH_ERRORS
1144:   const auto& prop = static_cast<NanmedianBackward0*>(self->cdata.get())->self_;
1145:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1146:   return obj.release().ptr();
1147:   END_HANDLE_TH_ERRORS
1148: }
1149: 
1150: static PyObject* THPNanmedianBackward0_result_getter(THPCppFunction *self, void *_unused) {
1151:   HANDLE_TH_ERRORS
1152:   const auto& prop = static_cast<NanmedianBackward0*>(self->cdata.get())->result_;
1153:   return THPVariable_Wrap(prop.unpack(self->cdata));
1154:   END_HANDLE_TH_ERRORS
1155: }
1156: 
1157: static PyObject* THPNanmedianBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
1158:   HANDLE_TH_ERRORS
1159:   const auto& prop = static_cast<NanmedianBackward0*>(self->cdata.get())->result_;
1160:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1161:   return obj.release().ptr();
1162:   END_HANDLE_TH_ERRORS
1163: }
1164: 
1165: static struct PyGetSetDef NanmedianBackward0_properties[] = {
1166:   THP_FUNCTION_DEFAULT_PROPERTIES,
1167:   {(char*)"_saved_self", (getter)THPNanmedianBackward0_self_getter, nullptr, nullptr, nullptr},
1168:   {(char*)"_raw_saved_self", (getter)THPNanmedianBackward0_self_raw_getter, nullptr, nullptr, nullptr},
1169:   {(char*)"_saved_result", (getter)THPNanmedianBackward0_result_getter, nullptr, nullptr, nullptr},
1170:   {(char*)"_raw_saved_result", (getter)THPNanmedianBackward0_result_raw_getter, nullptr, nullptr, nullptr},
1171:   {nullptr} /* sentinel */
1172: };
1173: 
1174: static PyObject* THPMedianBackward1_dim_getter(THPCppFunction *self, void *_unused) {
1175:   HANDLE_TH_ERRORS
1176:   auto prop = static_cast<MedianBackward1*>(self->cdata.get())->dim;
1177:   return PyLong_FromUnsignedLong((int64_t) prop);
1178:   END_HANDLE_TH_ERRORS
1179: }
1180: 
1181: static PyObject* THPMedianBackward1_keepdim_getter(THPCppFunction *self, void *_unused) {
1182:   HANDLE_TH_ERRORS
1183:   auto prop = static_cast<MedianBackward1*>(self->cdata.get())->keepdim;
1184:   if (prop) {
1185:     Py_RETURN_TRUE;
1186:   } else {
1187:     Py_RETURN_FALSE;
1188:   }
1189:   END_HANDLE_TH_ERRORS
1190: }
1191: 
1192: static PyObject* THPMedianBackward1_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
1193:   HANDLE_TH_ERRORS
1194:   auto prop = static_cast<MedianBackward1*>(self->cdata.get())->self_sym_sizes;
1195:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
1196:   for (auto i : c10::irange(prop.size())) {
1197:       auto si = prop[i];
1198:       if (auto m = si.maybe_as_int()) {
1199:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
1200:       } else {
```

- EN: The main execution path in this span is carried by `cast`, `THPNanmedianBackward0_self_getter`, `THPVariable_Wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `cast`, `THPNanmedianBackward0_self_getter`, `THPVariable_Wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1201-1280

```cpp
1201:         auto py_symint = py::cast(si).release().ptr();
1202:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
1203:       }
1204:   }
1205:   return tup;
1206:   END_HANDLE_TH_ERRORS
1207: }
1208: 
1209: static PyObject* THPMedianBackward1_indices_getter(THPCppFunction *self, void *_unused) {
1210:   HANDLE_TH_ERRORS
1211:   const auto& prop = static_cast<MedianBackward1*>(self->cdata.get())->indices_;
1212:   return THPVariable_Wrap(prop.unpack(self->cdata));
1213:   END_HANDLE_TH_ERRORS
1214: }
1215: 
1216: static PyObject* THPMedianBackward1_indices_raw_getter(THPCppFunction *self, void *_unused) {
1217:   HANDLE_TH_ERRORS
1218:   const auto& prop = static_cast<MedianBackward1*>(self->cdata.get())->indices_;
1219:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1220:   return obj.release().ptr();
1221:   END_HANDLE_TH_ERRORS
1222: }
1223: 
1224: static struct PyGetSetDef MedianBackward1_properties[] = {
1225:   THP_FUNCTION_DEFAULT_PROPERTIES,
1226:   {(char*)"_saved_dim", (getter)THPMedianBackward1_dim_getter, nullptr, nullptr, nullptr},
1227:   {(char*)"_saved_keepdim", (getter)THPMedianBackward1_keepdim_getter, nullptr, nullptr, nullptr},
1228:   {(char*)"_saved_self_sym_sizes", (getter)THPMedianBackward1_self_sym_sizes_getter, nullptr, nullptr, nullptr},
1229:   {(char*)"_saved_indices", (getter)THPMedianBackward1_indices_getter, nullptr, nullptr, nullptr},
1230:   {(char*)"_raw_saved_indices", (getter)THPMedianBackward1_indices_raw_getter, nullptr, nullptr, nullptr},
1231:   {nullptr} /* sentinel */
1232: };
1233: 
1234: static PyObject* THPNanmedianBackward1_dim_getter(THPCppFunction *self, void *_unused) {
1235:   HANDLE_TH_ERRORS
1236:   auto prop = static_cast<NanmedianBackward1*>(self->cdata.get())->dim;
1237:   return PyLong_FromUnsignedLong((int64_t) prop);
1238:   END_HANDLE_TH_ERRORS
1239: }
1240: 
1241: static PyObject* THPNanmedianBackward1_keepdim_getter(THPCppFunction *self, void *_unused) {
1242:   HANDLE_TH_ERRORS
1243:   auto prop = static_cast<NanmedianBackward1*>(self->cdata.get())->keepdim;
1244:   if (prop) {
1245:     Py_RETURN_TRUE;
1246:   } else {
1247:     Py_RETURN_FALSE;
1248:   }
1249:   END_HANDLE_TH_ERRORS
1250: }
1251: 
1252: static PyObject* THPNanmedianBackward1_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
1253:   HANDLE_TH_ERRORS
1254:   auto prop = static_cast<NanmedianBackward1*>(self->cdata.get())->self_sym_sizes;
1255:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
1256:   for (auto i : c10::irange(prop.size())) {
1257:       auto si = prop[i];
1258:       if (auto m = si.maybe_as_int()) {
1259:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
1260:       } else {
1261:         auto py_symint = py::cast(si).release().ptr();
1262:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
1263:       }
1264:   }
1265:   return tup;
1266:   END_HANDLE_TH_ERRORS
1267: }
1268: 
1269: static PyObject* THPNanmedianBackward1_indices_getter(THPCppFunction *self, void *_unused) {
1270:   HANDLE_TH_ERRORS
1271:   const auto& prop = static_cast<NanmedianBackward1*>(self->cdata.get())->indices_;
1272:   return THPVariable_Wrap(prop.unpack(self->cdata));
1273:   END_HANDLE_TH_ERRORS
1274: }
1275: 
1276: static PyObject* THPNanmedianBackward1_indices_raw_getter(THPCppFunction *self, void *_unused) {
1277:   HANDLE_TH_ERRORS
1278:   const auto& prop = static_cast<NanmedianBackward1*>(self->cdata.get())->indices_;
1279:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1280:   return obj.release().ptr();
```

- EN: The main execution path in this span is carried by `cast`, `PyTuple_SetItem`, `THPMedianBackward1_indices_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `cast`, `PyTuple_SetItem`, `THPMedianBackward1_indices_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1281-1360

```cpp
1281:   END_HANDLE_TH_ERRORS
1282: }
1283: 
1284: static struct PyGetSetDef NanmedianBackward1_properties[] = {
1285:   THP_FUNCTION_DEFAULT_PROPERTIES,
1286:   {(char*)"_saved_dim", (getter)THPNanmedianBackward1_dim_getter, nullptr, nullptr, nullptr},
1287:   {(char*)"_saved_keepdim", (getter)THPNanmedianBackward1_keepdim_getter, nullptr, nullptr, nullptr},
1288:   {(char*)"_saved_self_sym_sizes", (getter)THPNanmedianBackward1_self_sym_sizes_getter, nullptr, nullptr, nullptr},
1289:   {(char*)"_saved_indices", (getter)THPNanmedianBackward1_indices_getter, nullptr, nullptr, nullptr},
1290:   {(char*)"_raw_saved_indices", (getter)THPNanmedianBackward1_indices_raw_getter, nullptr, nullptr, nullptr},
1291:   {nullptr} /* sentinel */
1292: };
1293: 
1294: static PyObject* THPFusedRmsNormBackward0_input_getter(THPCppFunction *self, void *_unused) {
1295:   HANDLE_TH_ERRORS
1296:   const auto& prop = static_cast<FusedRmsNormBackward0*>(self->cdata.get())->input_;
1297:   return THPVariable_Wrap(prop.unpack(self->cdata));
1298:   END_HANDLE_TH_ERRORS
1299: }
1300: 
1301: static PyObject* THPFusedRmsNormBackward0_input_raw_getter(THPCppFunction *self, void *_unused) {
1302:   HANDLE_TH_ERRORS
1303:   const auto& prop = static_cast<FusedRmsNormBackward0*>(self->cdata.get())->input_;
1304:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1305:   return obj.release().ptr();
1306:   END_HANDLE_TH_ERRORS
1307: }
1308: 
1309: static PyObject* THPFusedRmsNormBackward0_normalized_shape_getter(THPCppFunction *self, void *_unused) {
1310:   HANDLE_TH_ERRORS
1311:   auto prop = static_cast<FusedRmsNormBackward0*>(self->cdata.get())->normalized_shape;
1312:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
1313:   for (auto i : c10::irange(prop.size())) {
1314:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
1315:   }
1316:   return tup;
1317:   END_HANDLE_TH_ERRORS
1318: }
1319: 
1320: static PyObject* THPFusedRmsNormBackward0_weight_getter(THPCppFunction *self, void *_unused) {
1321:   HANDLE_TH_ERRORS
1322:   const auto& prop = static_cast<FusedRmsNormBackward0*>(self->cdata.get())->weight_;
1323:   return THPVariable_Wrap(prop.unpack(self->cdata));
1324:   END_HANDLE_TH_ERRORS
1325: }
1326: 
1327: static PyObject* THPFusedRmsNormBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
1328:   HANDLE_TH_ERRORS
1329:   const auto& prop = static_cast<FusedRmsNormBackward0*>(self->cdata.get())->weight_;
1330:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1331:   return obj.release().ptr();
1332:   END_HANDLE_TH_ERRORS
1333: }
1334: 
1335: static PyObject* THPFusedRmsNormBackward0_result1_getter(THPCppFunction *self, void *_unused) {
1336:   HANDLE_TH_ERRORS
1337:   const auto& prop = static_cast<FusedRmsNormBackward0*>(self->cdata.get())->result1_;
1338:   return THPVariable_Wrap(prop.unpack(self->cdata));
1339:   END_HANDLE_TH_ERRORS
1340: }
1341: 
1342: static PyObject* THPFusedRmsNormBackward0_result1_raw_getter(THPCppFunction *self, void *_unused) {
1343:   HANDLE_TH_ERRORS
1344:   const auto& prop = static_cast<FusedRmsNormBackward0*>(self->cdata.get())->result1_;
1345:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1346:   return obj.release().ptr();
1347:   END_HANDLE_TH_ERRORS
1348: }
1349: 
1350: static struct PyGetSetDef FusedRmsNormBackward0_properties[] = {
1351:   THP_FUNCTION_DEFAULT_PROPERTIES,
1352:   {(char*)"_saved_input", (getter)THPFusedRmsNormBackward0_input_getter, nullptr, nullptr, nullptr},
1353:   {(char*)"_raw_saved_input", (getter)THPFusedRmsNormBackward0_input_raw_getter, nullptr, nullptr, nullptr},
1354:   {(char*)"_saved_normalized_shape", (getter)THPFusedRmsNormBackward0_normalized_shape_getter, nullptr, nullptr, nullptr},
1355:   {(char*)"_saved_weight", (getter)THPFusedRmsNormBackward0_weight_getter, nullptr, nullptr, nullptr},
1356:   {(char*)"_raw_saved_weight", (getter)THPFusedRmsNormBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
1357:   {(char*)"_saved_result1", (getter)THPFusedRmsNormBackward0_result1_getter, nullptr, nullptr, nullptr},
1358:   {(char*)"_raw_saved_result1", (getter)THPFusedRmsNormBackward0_result1_raw_getter, nullptr, nullptr, nullptr},
1359:   {nullptr} /* sentinel */
1360: };
```

- EN: The main execution path in this span is carried by `THPFusedRmsNormBackward0_input_getter`, `THPVariable_Wrap`, `THPFusedRmsNormBackward0_input_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPFusedRmsNormBackward0_input_getter`, `THPVariable_Wrap`, `THPFusedRmsNormBackward0_input_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1361-1440

```cpp
1361: 
1362: static PyObject* THPBatchNormNoUpdateBackward0_eps_getter(THPCppFunction *self, void *_unused) {
1363:   HANDLE_TH_ERRORS
1364:   auto prop = static_cast<BatchNormNoUpdateBackward0*>(self->cdata.get())->eps;
1365:   return PyFloat_FromDouble((double) prop);
1366:   END_HANDLE_TH_ERRORS
1367: }
1368: 
1369: static PyObject* THPBatchNormNoUpdateBackward0_input_getter(THPCppFunction *self, void *_unused) {
1370:   HANDLE_TH_ERRORS
1371:   const auto& prop = static_cast<BatchNormNoUpdateBackward0*>(self->cdata.get())->input_;
1372:   return THPVariable_Wrap(prop.unpack(self->cdata));
1373:   END_HANDLE_TH_ERRORS
1374: }
1375: 
1376: static PyObject* THPBatchNormNoUpdateBackward0_input_raw_getter(THPCppFunction *self, void *_unused) {
1377:   HANDLE_TH_ERRORS
1378:   const auto& prop = static_cast<BatchNormNoUpdateBackward0*>(self->cdata.get())->input_;
1379:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1380:   return obj.release().ptr();
1381:   END_HANDLE_TH_ERRORS
1382: }
1383: 
1384: static PyObject* THPBatchNormNoUpdateBackward0_running_mean_getter(THPCppFunction *self, void *_unused) {
1385:   HANDLE_TH_ERRORS
1386:   const auto& prop = static_cast<BatchNormNoUpdateBackward0*>(self->cdata.get())->running_mean_;
1387:   return THPVariable_Wrap(prop.unpack(self->cdata));
1388:   END_HANDLE_TH_ERRORS
1389: }
1390: 
1391: static PyObject* THPBatchNormNoUpdateBackward0_running_mean_raw_getter(THPCppFunction *self, void *_unused) {
1392:   HANDLE_TH_ERRORS
1393:   const auto& prop = static_cast<BatchNormNoUpdateBackward0*>(self->cdata.get())->running_mean_;
1394:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1395:   return obj.release().ptr();
1396:   END_HANDLE_TH_ERRORS
1397: }
1398: 
1399: static PyObject* THPBatchNormNoUpdateBackward0_running_var_getter(THPCppFunction *self, void *_unused) {
1400:   HANDLE_TH_ERRORS
1401:   const auto& prop = static_cast<BatchNormNoUpdateBackward0*>(self->cdata.get())->running_var_;
1402:   return THPVariable_Wrap(prop.unpack(self->cdata));
1403:   END_HANDLE_TH_ERRORS
1404: }
1405: 
1406: static PyObject* THPBatchNormNoUpdateBackward0_running_var_raw_getter(THPCppFunction *self, void *_unused) {
1407:   HANDLE_TH_ERRORS
1408:   const auto& prop = static_cast<BatchNormNoUpdateBackward0*>(self->cdata.get())->running_var_;
1409:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1410:   return obj.release().ptr();
1411:   END_HANDLE_TH_ERRORS
1412: }
1413: 
1414: static PyObject* THPBatchNormNoUpdateBackward0_weight_getter(THPCppFunction *self, void *_unused) {
1415:   HANDLE_TH_ERRORS
1416:   const auto& prop = static_cast<BatchNormNoUpdateBackward0*>(self->cdata.get())->weight_;
1417:   return THPVariable_Wrap(prop.unpack(self->cdata));
1418:   END_HANDLE_TH_ERRORS
1419: }
1420: 
1421: static PyObject* THPBatchNormNoUpdateBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
1422:   HANDLE_TH_ERRORS
1423:   const auto& prop = static_cast<BatchNormNoUpdateBackward0*>(self->cdata.get())->weight_;
1424:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1425:   return obj.release().ptr();
1426:   END_HANDLE_TH_ERRORS
1427: }
1428: 
1429: static PyObject* THPBatchNormNoUpdateBackward0_result1_getter(THPCppFunction *self, void *_unused) {
1430:   HANDLE_TH_ERRORS
1431:   const auto& prop = static_cast<BatchNormNoUpdateBackward0*>(self->cdata.get())->result1_;
1432:   return THPVariable_Wrap(prop.unpack(self->cdata));
1433:   END_HANDLE_TH_ERRORS
1434: }
1435: 
1436: static PyObject* THPBatchNormNoUpdateBackward0_result1_raw_getter(THPCppFunction *self, void *_unused) {
1437:   HANDLE_TH_ERRORS
1438:   const auto& prop = static_cast<BatchNormNoUpdateBackward0*>(self->cdata.get())->result1_;
1439:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1440:   return obj.release().ptr();
```

- EN: The main execution path in this span is carried by `THPBatchNormNoUpdateBackward0_eps_getter`, `PyFloat_FromDouble`, `THPBatchNormNoUpdateBackward0_input_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPBatchNormNoUpdateBackward0_eps_getter`, `PyFloat_FromDouble`, `THPBatchNormNoUpdateBackward0_input_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1441-1520

```cpp
1441:   END_HANDLE_TH_ERRORS
1442: }
1443: 
1444: static PyObject* THPBatchNormNoUpdateBackward0_result2_getter(THPCppFunction *self, void *_unused) {
1445:   HANDLE_TH_ERRORS
1446:   const auto& prop = static_cast<BatchNormNoUpdateBackward0*>(self->cdata.get())->result2_;
1447:   return THPVariable_Wrap(prop.unpack(self->cdata));
1448:   END_HANDLE_TH_ERRORS
1449: }
1450: 
1451: static PyObject* THPBatchNormNoUpdateBackward0_result2_raw_getter(THPCppFunction *self, void *_unused) {
1452:   HANDLE_TH_ERRORS
1453:   const auto& prop = static_cast<BatchNormNoUpdateBackward0*>(self->cdata.get())->result2_;
1454:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1455:   return obj.release().ptr();
1456:   END_HANDLE_TH_ERRORS
1457: }
1458: 
1459: static PyObject* THPBatchNormNoUpdateBackward0_result3_getter(THPCppFunction *self, void *_unused) {
1460:   HANDLE_TH_ERRORS
1461:   const auto& prop = static_cast<BatchNormNoUpdateBackward0*>(self->cdata.get())->result3_;
1462:   return THPVariable_Wrap(prop.unpack(self->cdata));
1463:   END_HANDLE_TH_ERRORS
1464: }
1465: 
1466: static PyObject* THPBatchNormNoUpdateBackward0_result3_raw_getter(THPCppFunction *self, void *_unused) {
1467:   HANDLE_TH_ERRORS
1468:   const auto& prop = static_cast<BatchNormNoUpdateBackward0*>(self->cdata.get())->result3_;
1469:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1470:   return obj.release().ptr();
1471:   END_HANDLE_TH_ERRORS
1472: }
1473: 
1474: static struct PyGetSetDef BatchNormNoUpdateBackward0_properties[] = {
1475:   THP_FUNCTION_DEFAULT_PROPERTIES,
1476:   {(char*)"_saved_eps", (getter)THPBatchNormNoUpdateBackward0_eps_getter, nullptr, nullptr, nullptr},
1477:   {(char*)"_saved_input", (getter)THPBatchNormNoUpdateBackward0_input_getter, nullptr, nullptr, nullptr},
1478:   {(char*)"_raw_saved_input", (getter)THPBatchNormNoUpdateBackward0_input_raw_getter, nullptr, nullptr, nullptr},
1479:   {(char*)"_saved_running_mean", (getter)THPBatchNormNoUpdateBackward0_running_mean_getter, nullptr, nullptr, nullptr},
1480:   {(char*)"_raw_saved_running_mean", (getter)THPBatchNormNoUpdateBackward0_running_mean_raw_getter, nullptr, nullptr, nullptr},
1481:   {(char*)"_saved_running_var", (getter)THPBatchNormNoUpdateBackward0_running_var_getter, nullptr, nullptr, nullptr},
1482:   {(char*)"_raw_saved_running_var", (getter)THPBatchNormNoUpdateBackward0_running_var_raw_getter, nullptr, nullptr, nullptr},
1483:   {(char*)"_saved_weight", (getter)THPBatchNormNoUpdateBackward0_weight_getter, nullptr, nullptr, nullptr},
1484:   {(char*)"_raw_saved_weight", (getter)THPBatchNormNoUpdateBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
1485:   {(char*)"_saved_result1", (getter)THPBatchNormNoUpdateBackward0_result1_getter, nullptr, nullptr, nullptr},
1486:   {(char*)"_raw_saved_result1", (getter)THPBatchNormNoUpdateBackward0_result1_raw_getter, nullptr, nullptr, nullptr},
1487:   {(char*)"_saved_result2", (getter)THPBatchNormNoUpdateBackward0_result2_getter, nullptr, nullptr, nullptr},
1488:   {(char*)"_raw_saved_result2", (getter)THPBatchNormNoUpdateBackward0_result2_raw_getter, nullptr, nullptr, nullptr},
1489:   {(char*)"_saved_result3", (getter)THPBatchNormNoUpdateBackward0_result3_getter, nullptr, nullptr, nullptr},
1490:   {(char*)"_raw_saved_result3", (getter)THPBatchNormNoUpdateBackward0_result3_raw_getter, nullptr, nullptr, nullptr},
1491:   {nullptr} /* sentinel */
1492: };
1493: 
1494: 
1495: 
1496: static struct PyGetSetDef PoissonBackward0_properties[] = {
1497:   THP_FUNCTION_DEFAULT_PROPERTIES,
1498: 
1499:   {nullptr} /* sentinel */
1500: };
1501: 
1502: static PyObject* THPProdBackward0_self_getter(THPCppFunction *self, void *_unused) {
1503:   HANDLE_TH_ERRORS
1504:   const auto& prop = static_cast<ProdBackward0*>(self->cdata.get())->self_;
1505:   return THPVariable_Wrap(prop.unpack(self->cdata));
1506:   END_HANDLE_TH_ERRORS
1507: }
1508: 
1509: static PyObject* THPProdBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
1510:   HANDLE_TH_ERRORS
1511:   const auto& prop = static_cast<ProdBackward0*>(self->cdata.get())->self_;
1512:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1513:   return obj.release().ptr();
1514:   END_HANDLE_TH_ERRORS
1515: }
1516: 
1517: static PyObject* THPProdBackward0_result_getter(THPCppFunction *self, void *_unused) {
1518:   HANDLE_TH_ERRORS
1519:   const auto& prop = static_cast<ProdBackward0*>(self->cdata.get())->result_;
1520:   return THPVariable_Wrap(prop.unpack(self->cdata));
```

- EN: The main execution path in this span is carried by `THPBatchNormNoUpdateBackward0_result2_getter`, `THPVariable_Wrap`, `THPBatchNormNoUpdateBackward0_result2_raw_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPBatchNormNoUpdateBackward0_result2_getter`, `THPVariable_Wrap`, `THPBatchNormNoUpdateBackward0_result2_raw_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1521-1600

```cpp
1521:   END_HANDLE_TH_ERRORS
1522: }
1523: 
1524: static PyObject* THPProdBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
1525:   HANDLE_TH_ERRORS
1526:   const auto& prop = static_cast<ProdBackward0*>(self->cdata.get())->result_;
1527:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1528:   return obj.release().ptr();
1529:   END_HANDLE_TH_ERRORS
1530: }
1531: 
1532: static struct PyGetSetDef ProdBackward0_properties[] = {
1533:   THP_FUNCTION_DEFAULT_PROPERTIES,
1534:   {(char*)"_saved_self", (getter)THPProdBackward0_self_getter, nullptr, nullptr, nullptr},
1535:   {(char*)"_raw_saved_self", (getter)THPProdBackward0_self_raw_getter, nullptr, nullptr, nullptr},
1536:   {(char*)"_saved_result", (getter)THPProdBackward0_result_getter, nullptr, nullptr, nullptr},
1537:   {(char*)"_raw_saved_result", (getter)THPProdBackward0_result_raw_getter, nullptr, nullptr, nullptr},
1538:   {nullptr} /* sentinel */
1539: };
1540: 
1541: static PyObject* THPProdBackward1_dim_getter(THPCppFunction *self, void *_unused) {
1542:   HANDLE_TH_ERRORS
1543:   auto prop = static_cast<ProdBackward1*>(self->cdata.get())->dim;
1544:   return PyLong_FromUnsignedLong((int64_t) prop);
1545:   END_HANDLE_TH_ERRORS
1546: }
1547: 
1548: static PyObject* THPProdBackward1_keepdim_getter(THPCppFunction *self, void *_unused) {
1549:   HANDLE_TH_ERRORS
1550:   auto prop = static_cast<ProdBackward1*>(self->cdata.get())->keepdim;
1551:   if (prop) {
1552:     Py_RETURN_TRUE;
1553:   } else {
1554:     Py_RETURN_FALSE;
1555:   }
1556:   END_HANDLE_TH_ERRORS
1557: }
1558: 
1559: static PyObject* THPProdBackward1_self_getter(THPCppFunction *self, void *_unused) {
1560:   HANDLE_TH_ERRORS
1561:   const auto& prop = static_cast<ProdBackward1*>(self->cdata.get())->self_;
1562:   return THPVariable_Wrap(prop.unpack(self->cdata));
1563:   END_HANDLE_TH_ERRORS
1564: }
1565: 
1566: static PyObject* THPProdBackward1_self_raw_getter(THPCppFunction *self, void *_unused) {
1567:   HANDLE_TH_ERRORS
1568:   const auto& prop = static_cast<ProdBackward1*>(self->cdata.get())->self_;
1569:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1570:   return obj.release().ptr();
1571:   END_HANDLE_TH_ERRORS
1572: }
1573: 
1574: static PyObject* THPProdBackward1_result_getter(THPCppFunction *self, void *_unused) {
1575:   HANDLE_TH_ERRORS
1576:   const auto& prop = static_cast<ProdBackward1*>(self->cdata.get())->result_;
1577:   return THPVariable_Wrap(prop.unpack(self->cdata));
1578:   END_HANDLE_TH_ERRORS
1579: }
1580: 
1581: static PyObject* THPProdBackward1_result_raw_getter(THPCppFunction *self, void *_unused) {
1582:   HANDLE_TH_ERRORS
1583:   const auto& prop = static_cast<ProdBackward1*>(self->cdata.get())->result_;
1584:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1585:   return obj.release().ptr();
1586:   END_HANDLE_TH_ERRORS
1587: }
1588: 
1589: static struct PyGetSetDef ProdBackward1_properties[] = {
1590:   THP_FUNCTION_DEFAULT_PROPERTIES,
1591:   {(char*)"_saved_dim", (getter)THPProdBackward1_dim_getter, nullptr, nullptr, nullptr},
1592:   {(char*)"_saved_keepdim", (getter)THPProdBackward1_keepdim_getter, nullptr, nullptr, nullptr},
1593:   {(char*)"_saved_self", (getter)THPProdBackward1_self_getter, nullptr, nullptr, nullptr},
1594:   {(char*)"_raw_saved_self", (getter)THPProdBackward1_self_raw_getter, nullptr, nullptr, nullptr},
1595:   {(char*)"_saved_result", (getter)THPProdBackward1_result_getter, nullptr, nullptr, nullptr},
1596:   {(char*)"_raw_saved_result", (getter)THPProdBackward1_result_raw_getter, nullptr, nullptr, nullptr},
1597:   {nullptr} /* sentinel */
1598: };
1599: 
1600: static PyObject* THPLinalgQrBackward0_mode_getter(THPCppFunction *self, void *_unused) {
```

- EN: The main execution path in this span is carried by `THPProdBackward0_result_raw_getter`, `cast`, `THPProdBackward1_dim_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPProdBackward0_result_raw_getter`, `cast`, `THPProdBackward1_dim_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1601-1680

```cpp
1601:   HANDLE_TH_ERRORS
1602:   auto prop = static_cast<LinalgQrBackward0*>(self->cdata.get())->mode;
1603:   return PyUnicode_FromStringAndSize(prop.data(), prop.size());
1604:   END_HANDLE_TH_ERRORS
1605: }
1606: 
1607: static PyObject* THPLinalgQrBackward0_Q_getter(THPCppFunction *self, void *_unused) {
1608:   HANDLE_TH_ERRORS
1609:   const auto& prop = static_cast<LinalgQrBackward0*>(self->cdata.get())->Q_;
1610:   return THPVariable_Wrap(prop.unpack(self->cdata));
1611:   END_HANDLE_TH_ERRORS
1612: }
1613: 
1614: static PyObject* THPLinalgQrBackward0_Q_raw_getter(THPCppFunction *self, void *_unused) {
1615:   HANDLE_TH_ERRORS
1616:   const auto& prop = static_cast<LinalgQrBackward0*>(self->cdata.get())->Q_;
1617:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1618:   return obj.release().ptr();
1619:   END_HANDLE_TH_ERRORS
1620: }
1621: 
1622: static PyObject* THPLinalgQrBackward0_R_getter(THPCppFunction *self, void *_unused) {
1623:   HANDLE_TH_ERRORS
1624:   const auto& prop = static_cast<LinalgQrBackward0*>(self->cdata.get())->R_;
1625:   return THPVariable_Wrap(prop.unpack(self->cdata));
1626:   END_HANDLE_TH_ERRORS
1627: }
1628: 
1629: static PyObject* THPLinalgQrBackward0_R_raw_getter(THPCppFunction *self, void *_unused) {
1630:   HANDLE_TH_ERRORS
1631:   const auto& prop = static_cast<LinalgQrBackward0*>(self->cdata.get())->R_;
1632:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1633:   return obj.release().ptr();
1634:   END_HANDLE_TH_ERRORS
1635: }
1636: 
1637: static struct PyGetSetDef LinalgQrBackward0_properties[] = {
1638:   THP_FUNCTION_DEFAULT_PROPERTIES,
1639:   {(char*)"_saved_mode", (getter)THPLinalgQrBackward0_mode_getter, nullptr, nullptr, nullptr},
1640:   {(char*)"_saved_Q", (getter)THPLinalgQrBackward0_Q_getter, nullptr, nullptr, nullptr},
1641:   {(char*)"_raw_saved_Q", (getter)THPLinalgQrBackward0_Q_raw_getter, nullptr, nullptr, nullptr},
1642:   {(char*)"_saved_R", (getter)THPLinalgQrBackward0_R_getter, nullptr, nullptr, nullptr},
1643:   {(char*)"_raw_saved_R", (getter)THPLinalgQrBackward0_R_raw_getter, nullptr, nullptr, nullptr},
1644:   {nullptr} /* sentinel */
1645: };
1646: 
1647: 
1648: 
1649: static struct PyGetSetDef RemainderBackward0_properties[] = {
1650:   THP_FUNCTION_DEFAULT_PROPERTIES,
1651: 
1652:   {nullptr} /* sentinel */
1653: };
1654: 
1655: static PyObject* THPRemainderBackward1_other_getter(THPCppFunction *self, void *_unused) {
1656:   HANDLE_TH_ERRORS
1657:   const auto& prop = static_cast<RemainderBackward1*>(self->cdata.get())->other_;
1658:   return THPVariable_Wrap(prop.unpack(self->cdata));
1659:   END_HANDLE_TH_ERRORS
1660: }
1661: 
1662: static PyObject* THPRemainderBackward1_other_raw_getter(THPCppFunction *self, void *_unused) {
1663:   HANDLE_TH_ERRORS
1664:   const auto& prop = static_cast<RemainderBackward1*>(self->cdata.get())->other_;
1665:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1666:   return obj.release().ptr();
1667:   END_HANDLE_TH_ERRORS
1668: }
1669: 
1670: static PyObject* THPRemainderBackward1_self_getter(THPCppFunction *self, void *_unused) {
1671:   HANDLE_TH_ERRORS
1672:   const auto& prop = static_cast<RemainderBackward1*>(self->cdata.get())->self_;
1673:   return THPVariable_Wrap(prop.unpack(self->cdata));
1674:   END_HANDLE_TH_ERRORS
1675: }
1676: 
1677: static PyObject* THPRemainderBackward1_self_raw_getter(THPCppFunction *self, void *_unused) {
1678:   HANDLE_TH_ERRORS
1679:   const auto& prop = static_cast<RemainderBackward1*>(self->cdata.get())->self_;
1680:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
```

- EN: The main execution path in this span is carried by `PyUnicode_FromStringAndSize`, `THPLinalgQrBackward0_Q_getter`, `THPVariable_Wrap`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyUnicode_FromStringAndSize`, `THPLinalgQrBackward0_Q_getter`, `THPVariable_Wrap` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1681-1760

```cpp
1681:   return obj.release().ptr();
1682:   END_HANDLE_TH_ERRORS
1683: }
1684: 
1685: static struct PyGetSetDef RemainderBackward1_properties[] = {
1686:   THP_FUNCTION_DEFAULT_PROPERTIES,
1687:   {(char*)"_saved_other", (getter)THPRemainderBackward1_other_getter, nullptr, nullptr, nullptr},
1688:   {(char*)"_raw_saved_other", (getter)THPRemainderBackward1_other_raw_getter, nullptr, nullptr, nullptr},
1689:   {(char*)"_saved_self", (getter)THPRemainderBackward1_self_getter, nullptr, nullptr, nullptr},
1690:   {(char*)"_raw_saved_self", (getter)THPRemainderBackward1_self_raw_getter, nullptr, nullptr, nullptr},
1691:   {nullptr} /* sentinel */
1692: };
1693: 
1694: static PyObject* THPRenormBackward0_dim_getter(THPCppFunction *self, void *_unused) {
1695:   HANDLE_TH_ERRORS
1696:   auto prop = static_cast<RenormBackward0*>(self->cdata.get())->dim;
1697:   return PyLong_FromUnsignedLong((int64_t) prop);
1698:   END_HANDLE_TH_ERRORS
1699: }
1700: 
1701: static PyObject* THPRenormBackward0_maxnorm_getter(THPCppFunction *self, void *_unused) {
1702:   HANDLE_TH_ERRORS
1703:   auto prop = static_cast<RenormBackward0*>(self->cdata.get())->maxnorm;
1704:   if (prop.isComplex()) {
1705:     auto cprop = prop.to<c10::complex<double>>();
1706:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
1707:   } else if (prop.isFloatingPoint()) {
1708:     return PyFloat_FromDouble(prop.to<double>());
1709:   } else if (prop.isIntegral(/*includeBool=*/false)) {
1710:     return PyLong_FromLong(prop.to<int64_t>());
1711:   } else if (prop.isBoolean()) {
1712:     if (prop.to<bool>()) {
1713:       Py_RETURN_TRUE;
1714:     } else {
1715:       Py_RETURN_FALSE;
1716:     }
1717:   } else {
1718:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
1719:     return nullptr;
1720:   }
1721:   END_HANDLE_TH_ERRORS
1722: }
1723: 
1724: static PyObject* THPRenormBackward0_p_getter(THPCppFunction *self, void *_unused) {
1725:   HANDLE_TH_ERRORS
1726:   auto prop = static_cast<RenormBackward0*>(self->cdata.get())->p;
1727:   if (prop.isComplex()) {
1728:     auto cprop = prop.to<c10::complex<double>>();
1729:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
1730:   } else if (prop.isFloatingPoint()) {
1731:     return PyFloat_FromDouble(prop.to<double>());
1732:   } else if (prop.isIntegral(/*includeBool=*/false)) {
1733:     return PyLong_FromLong(prop.to<int64_t>());
1734:   } else if (prop.isBoolean()) {
1735:     if (prop.to<bool>()) {
1736:       Py_RETURN_TRUE;
1737:     } else {
1738:       Py_RETURN_FALSE;
1739:     }
1740:   } else {
1741:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
1742:     return nullptr;
1743:   }
1744:   END_HANDLE_TH_ERRORS
1745: }
1746: 
1747: static PyObject* THPRenormBackward0_self_getter(THPCppFunction *self, void *_unused) {
1748:   HANDLE_TH_ERRORS
1749:   const auto& prop = static_cast<RenormBackward0*>(self->cdata.get())->self_;
1750:   return THPVariable_Wrap(prop.unpack(self->cdata));
1751:   END_HANDLE_TH_ERRORS
1752: }
1753: 
1754: static PyObject* THPRenormBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
1755:   HANDLE_TH_ERRORS
1756:   const auto& prop = static_cast<RenormBackward0*>(self->cdata.get())->self_;
1757:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1758:   return obj.release().ptr();
1759:   END_HANDLE_TH_ERRORS
1760: }
```

- EN: The main execution path in this span is carried by `THPRenormBackward0_dim_getter`, `PyLong_FromUnsignedLong`, `THPRenormBackward0_maxnorm_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPRenormBackward0_dim_getter`, `PyLong_FromUnsignedLong`, `THPRenormBackward0_maxnorm_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1761-1840

```cpp
1761: 
1762: static struct PyGetSetDef RenormBackward0_properties[] = {
1763:   THP_FUNCTION_DEFAULT_PROPERTIES,
1764:   {(char*)"_saved_dim", (getter)THPRenormBackward0_dim_getter, nullptr, nullptr, nullptr},
1765:   {(char*)"_saved_maxnorm", (getter)THPRenormBackward0_maxnorm_getter, nullptr, nullptr, nullptr},
1766:   {(char*)"_saved_p", (getter)THPRenormBackward0_p_getter, nullptr, nullptr, nullptr},
1767:   {(char*)"_saved_self", (getter)THPRenormBackward0_self_getter, nullptr, nullptr, nullptr},
1768:   {(char*)"_raw_saved_self", (getter)THPRenormBackward0_self_raw_getter, nullptr, nullptr, nullptr},
1769:   {nullptr} /* sentinel */
1770: };
1771: 
1772: static PyObject* THPSincBackward0_self_getter(THPCppFunction *self, void *_unused) {
1773:   HANDLE_TH_ERRORS
1774:   const auto& prop = static_cast<SincBackward0*>(self->cdata.get())->self_;
1775:   return THPVariable_Wrap(prop.unpack(self->cdata));
1776:   END_HANDLE_TH_ERRORS
1777: }
1778: 
1779: static PyObject* THPSincBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
1780:   HANDLE_TH_ERRORS
1781:   const auto& prop = static_cast<SincBackward0*>(self->cdata.get())->self_;
1782:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1783:   return obj.release().ptr();
1784:   END_HANDLE_TH_ERRORS
1785: }
1786: 
1787: static struct PyGetSetDef SincBackward0_properties[] = {
1788:   THP_FUNCTION_DEFAULT_PROPERTIES,
1789:   {(char*)"_saved_self", (getter)THPSincBackward0_self_getter, nullptr, nullptr, nullptr},
1790:   {(char*)"_raw_saved_self", (getter)THPSincBackward0_self_raw_getter, nullptr, nullptr, nullptr},
1791:   {nullptr} /* sentinel */
1792: };
1793: 
1794: static PyObject* THPSliceBackward0_dim_getter(THPCppFunction *self, void *_unused) {
1795:   HANDLE_TH_ERRORS
1796:   auto prop = static_cast<SliceBackward0*>(self->cdata.get())->dim;
1797:   return PyLong_FromUnsignedLong((int64_t) prop);
1798:   END_HANDLE_TH_ERRORS
1799: }
1800: 
1801: static PyObject* THPSliceBackward0_end_getter(THPCppFunction *self, void *_unused) {
1802:   HANDLE_TH_ERRORS
1803:   auto opt_prop = static_cast<SliceBackward0*>(self->cdata.get())->end;
1804:   if (!opt_prop.has_value()) {
1805:     Py_RETURN_NONE;
1806:   }
1807:   auto prop = opt_prop.value();
1808:   if (auto m = prop.maybe_as_int()) {
1809:     return PyLong_FromUnsignedLong(*m);
1810:   } else {
1811:     return py::cast(prop).release().ptr();
1812:   }
1813:   END_HANDLE_TH_ERRORS
1814: }
1815: 
1816: static PyObject* THPSliceBackward0_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
1817:   HANDLE_TH_ERRORS
1818:   auto prop = static_cast<SliceBackward0*>(self->cdata.get())->self_sym_sizes;
1819:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
1820:   for (auto i : c10::irange(prop.size())) {
1821:       auto si = prop[i];
1822:       if (auto m = si.maybe_as_int()) {
1823:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
1824:       } else {
1825:         auto py_symint = py::cast(si).release().ptr();
1826:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
1827:       }
1828:   }
1829:   return tup;
1830:   END_HANDLE_TH_ERRORS
1831: }
1832: 
1833: static PyObject* THPSliceBackward0_start_getter(THPCppFunction *self, void *_unused) {
1834:   HANDLE_TH_ERRORS
1835:   auto opt_prop = static_cast<SliceBackward0*>(self->cdata.get())->start;
1836:   if (!opt_prop.has_value()) {
1837:     Py_RETURN_NONE;
1838:   }
1839:   auto prop = opt_prop.value();
1840:   if (auto m = prop.maybe_as_int()) {
```

- EN: The main execution path in this span is carried by `THPSincBackward0_self_getter`, `THPVariable_Wrap`, `THPSincBackward0_self_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPSincBackward0_self_getter`, `THPVariable_Wrap`, `THPSincBackward0_self_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1841-1920

```cpp
1841:     return PyLong_FromUnsignedLong(*m);
1842:   } else {
1843:     return py::cast(prop).release().ptr();
1844:   }
1845:   END_HANDLE_TH_ERRORS
1846: }
1847: 
1848: static PyObject* THPSliceBackward0_step_getter(THPCppFunction *self, void *_unused) {
1849:   HANDLE_TH_ERRORS
1850:   auto prop = static_cast<SliceBackward0*>(self->cdata.get())->step;
1851:   if (auto m = prop.maybe_as_int()) {
1852:     return PyLong_FromUnsignedLong(*m);
1853:   } else {
1854:     return py::cast(prop).release().ptr();
1855:   }
1856:   END_HANDLE_TH_ERRORS
1857: }
1858: 
1859: static struct PyGetSetDef SliceBackward0_properties[] = {
1860:   THP_FUNCTION_DEFAULT_PROPERTIES,
1861:   {(char*)"_saved_dim", (getter)THPSliceBackward0_dim_getter, nullptr, nullptr, nullptr},
1862:   {(char*)"_saved_end", (getter)THPSliceBackward0_end_getter, nullptr, nullptr, nullptr},
1863:   {(char*)"_saved_self_sym_sizes", (getter)THPSliceBackward0_self_sym_sizes_getter, nullptr, nullptr, nullptr},
1864:   {(char*)"_saved_start", (getter)THPSliceBackward0_start_getter, nullptr, nullptr, nullptr},
1865:   {(char*)"_saved_step", (getter)THPSliceBackward0_step_getter, nullptr, nullptr, nullptr},
1866:   {nullptr} /* sentinel */
1867: };
1868: 
1869: static PyObject* THPSliceInverseBackward0_dim_getter(THPCppFunction *self, void *_unused) {
1870:   HANDLE_TH_ERRORS
1871:   auto prop = static_cast<SliceInverseBackward0*>(self->cdata.get())->dim;
1872:   return PyLong_FromUnsignedLong((int64_t) prop);
1873:   END_HANDLE_TH_ERRORS
1874: }
1875: 
1876: static PyObject* THPSliceInverseBackward0_end_getter(THPCppFunction *self, void *_unused) {
1877:   HANDLE_TH_ERRORS
1878:   auto opt_prop = static_cast<SliceInverseBackward0*>(self->cdata.get())->end;
1879:   if (!opt_prop.has_value()) {
1880:     Py_RETURN_NONE;
1881:   }
1882:   auto prop = opt_prop.value();
1883:   if (auto m = prop.maybe_as_int()) {
1884:     return PyLong_FromUnsignedLong(*m);
1885:   } else {
1886:     return py::cast(prop).release().ptr();
1887:   }
1888:   END_HANDLE_TH_ERRORS
1889: }
1890: 
1891: static PyObject* THPSliceInverseBackward0_start_getter(THPCppFunction *self, void *_unused) {
1892:   HANDLE_TH_ERRORS
1893:   auto opt_prop = static_cast<SliceInverseBackward0*>(self->cdata.get())->start;
1894:   if (!opt_prop.has_value()) {
1895:     Py_RETURN_NONE;
1896:   }
1897:   auto prop = opt_prop.value();
1898:   if (auto m = prop.maybe_as_int()) {
1899:     return PyLong_FromUnsignedLong(*m);
1900:   } else {
1901:     return py::cast(prop).release().ptr();
1902:   }
1903:   END_HANDLE_TH_ERRORS
1904: }
1905: 
1906: static PyObject* THPSliceInverseBackward0_step_getter(THPCppFunction *self, void *_unused) {
1907:   HANDLE_TH_ERRORS
1908:   auto prop = static_cast<SliceInverseBackward0*>(self->cdata.get())->step;
1909:   if (auto m = prop.maybe_as_int()) {
1910:     return PyLong_FromUnsignedLong(*m);
1911:   } else {
1912:     return py::cast(prop).release().ptr();
1913:   }
1914:   END_HANDLE_TH_ERRORS
1915: }
1916: 
1917: static struct PyGetSetDef SliceInverseBackward0_properties[] = {
1918:   THP_FUNCTION_DEFAULT_PROPERTIES,
1919:   {(char*)"_saved_dim", (getter)THPSliceInverseBackward0_dim_getter, nullptr, nullptr, nullptr},
1920:   {(char*)"_saved_end", (getter)THPSliceInverseBackward0_end_getter, nullptr, nullptr, nullptr},
```

- EN: The main execution path in this span is carried by `PyLong_FromUnsignedLong`, `cast`, `THPSliceBackward0_step_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyLong_FromUnsignedLong`, `cast`, `THPSliceBackward0_step_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1921-2000

```cpp
1921:   {(char*)"_saved_start", (getter)THPSliceInverseBackward0_start_getter, nullptr, nullptr, nullptr},
1922:   {(char*)"_saved_step", (getter)THPSliceInverseBackward0_step_getter, nullptr, nullptr, nullptr},
1923:   {nullptr} /* sentinel */
1924: };
1925: 
1926: static PyObject* THPSelectScatterBackward0_dim_getter(THPCppFunction *self, void *_unused) {
1927:   HANDLE_TH_ERRORS
1928:   auto prop = static_cast<SelectScatterBackward0*>(self->cdata.get())->dim;
1929:   return PyLong_FromUnsignedLong((int64_t) prop);
1930:   END_HANDLE_TH_ERRORS
1931: }
1932: 
1933: static PyObject* THPSelectScatterBackward0_index_getter(THPCppFunction *self, void *_unused) {
1934:   HANDLE_TH_ERRORS
1935:   auto prop = static_cast<SelectScatterBackward0*>(self->cdata.get())->index;
1936:   if (auto m = prop.maybe_as_int()) {
1937:     return PyLong_FromUnsignedLong(*m);
1938:   } else {
1939:     return py::cast(prop).release().ptr();
1940:   }
1941:   END_HANDLE_TH_ERRORS
1942: }
1943: 
1944: static struct PyGetSetDef SelectScatterBackward0_properties[] = {
1945:   THP_FUNCTION_DEFAULT_PROPERTIES,
1946:   {(char*)"_saved_dim", (getter)THPSelectScatterBackward0_dim_getter, nullptr, nullptr, nullptr},
1947:   {(char*)"_saved_index", (getter)THPSelectScatterBackward0_index_getter, nullptr, nullptr, nullptr},
1948:   {nullptr} /* sentinel */
1949: };
1950: 
1951: static PyObject* THPLinalgSolveExBackward0_A_getter(THPCppFunction *self, void *_unused) {
1952:   HANDLE_TH_ERRORS
1953:   const auto& prop = static_cast<LinalgSolveExBackward0*>(self->cdata.get())->A_;
1954:   return THPVariable_Wrap(prop.unpack(self->cdata));
1955:   END_HANDLE_TH_ERRORS
1956: }
1957: 
1958: static PyObject* THPLinalgSolveExBackward0_A_raw_getter(THPCppFunction *self, void *_unused) {
1959:   HANDLE_TH_ERRORS
1960:   const auto& prop = static_cast<LinalgSolveExBackward0*>(self->cdata.get())->A_;
1961:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1962:   return obj.release().ptr();
1963:   END_HANDLE_TH_ERRORS
1964: }
1965: 
1966: static PyObject* THPLinalgSolveExBackward0_left_getter(THPCppFunction *self, void *_unused) {
1967:   HANDLE_TH_ERRORS
1968:   auto prop = static_cast<LinalgSolveExBackward0*>(self->cdata.get())->left;
1969:   if (prop) {
1970:     Py_RETURN_TRUE;
1971:   } else {
1972:     Py_RETURN_FALSE;
1973:   }
1974:   END_HANDLE_TH_ERRORS
1975: }
1976: 
1977: static PyObject* THPLinalgSolveExBackward0_LU_getter(THPCppFunction *self, void *_unused) {
1978:   HANDLE_TH_ERRORS
1979:   const auto& prop = static_cast<LinalgSolveExBackward0*>(self->cdata.get())->LU_;
1980:   return THPVariable_Wrap(prop.unpack(self->cdata));
1981:   END_HANDLE_TH_ERRORS
1982: }
1983: 
1984: static PyObject* THPLinalgSolveExBackward0_LU_raw_getter(THPCppFunction *self, void *_unused) {
1985:   HANDLE_TH_ERRORS
1986:   const auto& prop = static_cast<LinalgSolveExBackward0*>(self->cdata.get())->LU_;
1987:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
1988:   return obj.release().ptr();
1989:   END_HANDLE_TH_ERRORS
1990: }
1991: 
1992: static PyObject* THPLinalgSolveExBackward0_pivots_getter(THPCppFunction *self, void *_unused) {
1993:   HANDLE_TH_ERRORS
1994:   const auto& prop = static_cast<LinalgSolveExBackward0*>(self->cdata.get())->pivots_;
1995:   return THPVariable_Wrap(prop.unpack(self->cdata));
1996:   END_HANDLE_TH_ERRORS
1997: }
1998: 
1999: static PyObject* THPLinalgSolveExBackward0_pivots_raw_getter(THPCppFunction *self, void *_unused) {
2000:   HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPSelectScatterBackward0_dim_getter`, `PyLong_FromUnsignedLong`, `THPSelectScatterBackward0_index_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPSelectScatterBackward0_dim_getter`, `PyLong_FromUnsignedLong`, `THPSelectScatterBackward0_index_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2001-2080

```cpp
2001:   const auto& prop = static_cast<LinalgSolveExBackward0*>(self->cdata.get())->pivots_;
2002:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2003:   return obj.release().ptr();
2004:   END_HANDLE_TH_ERRORS
2005: }
2006: 
2007: static PyObject* THPLinalgSolveExBackward0_result_getter(THPCppFunction *self, void *_unused) {
2008:   HANDLE_TH_ERRORS
2009:   const auto& prop = static_cast<LinalgSolveExBackward0*>(self->cdata.get())->result_;
2010:   return THPVariable_Wrap(prop.unpack(self->cdata));
2011:   END_HANDLE_TH_ERRORS
2012: }
2013: 
2014: static PyObject* THPLinalgSolveExBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
2015:   HANDLE_TH_ERRORS
2016:   const auto& prop = static_cast<LinalgSolveExBackward0*>(self->cdata.get())->result_;
2017:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2018:   return obj.release().ptr();
2019:   END_HANDLE_TH_ERRORS
2020: }
2021: 
2022: static struct PyGetSetDef LinalgSolveExBackward0_properties[] = {
2023:   THP_FUNCTION_DEFAULT_PROPERTIES,
2024:   {(char*)"_saved_A", (getter)THPLinalgSolveExBackward0_A_getter, nullptr, nullptr, nullptr},
2025:   {(char*)"_raw_saved_A", (getter)THPLinalgSolveExBackward0_A_raw_getter, nullptr, nullptr, nullptr},
2026:   {(char*)"_saved_left", (getter)THPLinalgSolveExBackward0_left_getter, nullptr, nullptr, nullptr},
2027:   {(char*)"_saved_LU", (getter)THPLinalgSolveExBackward0_LU_getter, nullptr, nullptr, nullptr},
2028:   {(char*)"_raw_saved_LU", (getter)THPLinalgSolveExBackward0_LU_raw_getter, nullptr, nullptr, nullptr},
2029:   {(char*)"_saved_pivots", (getter)THPLinalgSolveExBackward0_pivots_getter, nullptr, nullptr, nullptr},
2030:   {(char*)"_raw_saved_pivots", (getter)THPLinalgSolveExBackward0_pivots_raw_getter, nullptr, nullptr, nullptr},
2031:   {(char*)"_saved_result", (getter)THPLinalgSolveExBackward0_result_getter, nullptr, nullptr, nullptr},
2032:   {(char*)"_raw_saved_result", (getter)THPLinalgSolveExBackward0_result_raw_getter, nullptr, nullptr, nullptr},
2033:   {nullptr} /* sentinel */
2034: };
2035: 
2036: static PyObject* THPUnsafeSplitWithSizesBackward0_dim_getter(THPCppFunction *self, void *_unused) {
2037:   HANDLE_TH_ERRORS
2038:   auto prop = static_cast<UnsafeSplitWithSizesBackward0*>(self->cdata.get())->dim;
2039:   return PyLong_FromUnsignedLong((int64_t) prop);
2040:   END_HANDLE_TH_ERRORS
2041: }
2042: 
2043: static PyObject* THPUnsafeSplitWithSizesBackward0_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
2044:   HANDLE_TH_ERRORS
2045:   auto prop = static_cast<UnsafeSplitWithSizesBackward0*>(self->cdata.get())->self_sym_sizes;
2046:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
2047:   for (auto i : c10::irange(prop.size())) {
2048:       auto si = prop[i];
2049:       if (auto m = si.maybe_as_int()) {
2050:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
2051:       } else {
2052:         auto py_symint = py::cast(si).release().ptr();
2053:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
2054:       }
2055:   }
2056:   return tup;
2057:   END_HANDLE_TH_ERRORS
2058: }
2059: 
2060: static PyObject* THPUnsafeSplitWithSizesBackward0_split_sizes_getter(THPCppFunction *self, void *_unused) {
2061:   HANDLE_TH_ERRORS
2062:   auto prop = static_cast<UnsafeSplitWithSizesBackward0*>(self->cdata.get())->split_sizes;
2063:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
2064:   for (auto i : c10::irange(prop.size())) {
2065:       auto si = prop[i];
2066:       if (auto m = si.maybe_as_int()) {
2067:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
2068:       } else {
2069:         auto py_symint = py::cast(si).release().ptr();
2070:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
2071:       }
2072:   }
2073:   return tup;
2074:   END_HANDLE_TH_ERRORS
2075: }
2076: 
2077: static struct PyGetSetDef UnsafeSplitWithSizesBackward0_properties[] = {
2078:   THP_FUNCTION_DEFAULT_PROPERTIES,
2079:   {(char*)"_saved_dim", (getter)THPUnsafeSplitWithSizesBackward0_dim_getter, nullptr, nullptr, nullptr},
2080:   {(char*)"_saved_self_sym_sizes", (getter)THPUnsafeSplitWithSizesBackward0_self_sym_sizes_getter, nullptr, nullptr, nullptr},
```

- EN: The main execution path in this span is carried by `cast`, `THPLinalgSolveExBackward0_result_getter`, `THPVariable_Wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `cast`, `THPLinalgSolveExBackward0_result_getter`, `THPVariable_Wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2081-2160

```cpp
2081:   {(char*)"_saved_split_sizes", (getter)THPUnsafeSplitWithSizesBackward0_split_sizes_getter, nullptr, nullptr, nullptr},
2082:   {nullptr} /* sentinel */
2083: };
2084: 
2085: static PyObject* THPSubBackward0_alpha_getter(THPCppFunction *self, void *_unused) {
2086:   HANDLE_TH_ERRORS
2087:   auto prop = static_cast<SubBackward0*>(self->cdata.get())->alpha;
2088:   if (prop.isComplex()) {
2089:     auto cprop = prop.to<c10::complex<double>>();
2090:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
2091:   } else if (prop.isFloatingPoint()) {
2092:     return PyFloat_FromDouble(prop.to<double>());
2093:   } else if (prop.isIntegral(/*includeBool=*/false)) {
2094:     return PyLong_FromLong(prop.to<int64_t>());
2095:   } else if (prop.isBoolean()) {
2096:     if (prop.to<bool>()) {
2097:       Py_RETURN_TRUE;
2098:     } else {
2099:       Py_RETURN_FALSE;
2100:     }
2101:   } else {
2102:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
2103:     return nullptr;
2104:   }
2105:   END_HANDLE_TH_ERRORS
2106: }
2107: 
2108: static struct PyGetSetDef SubBackward0_properties[] = {
2109:   THP_FUNCTION_DEFAULT_PROPERTIES,
2110:   {(char*)"_saved_alpha", (getter)THPSubBackward0_alpha_getter, nullptr, nullptr, nullptr},
2111:   {nullptr} /* sentinel */
2112: };
2113: 
2114: 
2115: 
2116: static struct PyGetSetDef SubBackward1_properties[] = {
2117:   THP_FUNCTION_DEFAULT_PROPERTIES,
2118: 
2119:   {nullptr} /* sentinel */
2120: };
2121: 
2122: static PyObject* THPLinalgSvdBackward0_full_matrices_getter(THPCppFunction *self, void *_unused) {
2123:   HANDLE_TH_ERRORS
2124:   auto prop = static_cast<LinalgSvdBackward0*>(self->cdata.get())->full_matrices;
2125:   if (prop) {
2126:     Py_RETURN_TRUE;
2127:   } else {
2128:     Py_RETURN_FALSE;
2129:   }
2130:   END_HANDLE_TH_ERRORS
2131: }
2132: 
2133: static PyObject* THPLinalgSvdBackward0_S_getter(THPCppFunction *self, void *_unused) {
2134:   HANDLE_TH_ERRORS
2135:   const auto& prop = static_cast<LinalgSvdBackward0*>(self->cdata.get())->S_;
2136:   return THPVariable_Wrap(prop.unpack(self->cdata));
2137:   END_HANDLE_TH_ERRORS
2138: }
2139: 
2140: static PyObject* THPLinalgSvdBackward0_S_raw_getter(THPCppFunction *self, void *_unused) {
2141:   HANDLE_TH_ERRORS
2142:   const auto& prop = static_cast<LinalgSvdBackward0*>(self->cdata.get())->S_;
2143:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2144:   return obj.release().ptr();
2145:   END_HANDLE_TH_ERRORS
2146: }
2147: 
2148: static PyObject* THPLinalgSvdBackward0_S_sym_argsize_minus_1_getter(THPCppFunction *self, void *_unused) {
2149:   HANDLE_TH_ERRORS
2150:   auto prop = static_cast<LinalgSvdBackward0*>(self->cdata.get())->S_sym_argsize_minus_1;
2151:   if (auto m = prop.maybe_as_int()) {
2152:     return PyLong_FromUnsignedLong(*m);
2153:   } else {
2154:     return py::cast(prop).release().ptr();
2155:   }
2156:   END_HANDLE_TH_ERRORS
2157: }
2158: 
2159: static PyObject* THPLinalgSvdBackward0_U_getter(THPCppFunction *self, void *_unused) {
2160:   HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPSubBackward0_alpha_getter`, `PyComplex_FromDoubles`, `PyFloat_FromDouble`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPSubBackward0_alpha_getter`, `PyComplex_FromDoubles`, `PyFloat_FromDouble` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2161-2240

```cpp
2161:   const auto& prop = static_cast<LinalgSvdBackward0*>(self->cdata.get())->U_;
2162:   return THPVariable_Wrap(prop.unpack(self->cdata));
2163:   END_HANDLE_TH_ERRORS
2164: }
2165: 
2166: static PyObject* THPLinalgSvdBackward0_U_raw_getter(THPCppFunction *self, void *_unused) {
2167:   HANDLE_TH_ERRORS
2168:   const auto& prop = static_cast<LinalgSvdBackward0*>(self->cdata.get())->U_;
2169:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2170:   return obj.release().ptr();
2171:   END_HANDLE_TH_ERRORS
2172: }
2173: 
2174: static PyObject* THPLinalgSvdBackward0_Vh_getter(THPCppFunction *self, void *_unused) {
2175:   HANDLE_TH_ERRORS
2176:   const auto& prop = static_cast<LinalgSvdBackward0*>(self->cdata.get())->Vh_;
2177:   return THPVariable_Wrap(prop.unpack(self->cdata));
2178:   END_HANDLE_TH_ERRORS
2179: }
2180: 
2181: static PyObject* THPLinalgSvdBackward0_Vh_raw_getter(THPCppFunction *self, void *_unused) {
2182:   HANDLE_TH_ERRORS
2183:   const auto& prop = static_cast<LinalgSvdBackward0*>(self->cdata.get())->Vh_;
2184:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2185:   return obj.release().ptr();
2186:   END_HANDLE_TH_ERRORS
2187: }
2188: 
2189: static struct PyGetSetDef LinalgSvdBackward0_properties[] = {
2190:   THP_FUNCTION_DEFAULT_PROPERTIES,
2191:   {(char*)"_saved_full_matrices", (getter)THPLinalgSvdBackward0_full_matrices_getter, nullptr, nullptr, nullptr},
2192:   {(char*)"_saved_S", (getter)THPLinalgSvdBackward0_S_getter, nullptr, nullptr, nullptr},
2193:   {(char*)"_raw_saved_S", (getter)THPLinalgSvdBackward0_S_raw_getter, nullptr, nullptr, nullptr},
2194:   {(char*)"_saved_S_sym_argsize_minus_1", (getter)THPLinalgSvdBackward0_S_sym_argsize_minus_1_getter, nullptr, nullptr, nullptr},
2195:   {(char*)"_saved_U", (getter)THPLinalgSvdBackward0_U_getter, nullptr, nullptr, nullptr},
2196:   {(char*)"_raw_saved_U", (getter)THPLinalgSvdBackward0_U_raw_getter, nullptr, nullptr, nullptr},
2197:   {(char*)"_saved_Vh", (getter)THPLinalgSvdBackward0_Vh_getter, nullptr, nullptr, nullptr},
2198:   {(char*)"_raw_saved_Vh", (getter)THPLinalgSvdBackward0_Vh_raw_getter, nullptr, nullptr, nullptr},
2199:   {nullptr} /* sentinel */
2200: };
2201: 
2202: static PyObject* THPTakeBackward0_index_getter(THPCppFunction *self, void *_unused) {
2203:   HANDLE_TH_ERRORS
2204:   const auto& prop = static_cast<TakeBackward0*>(self->cdata.get())->index_;
2205:   return THPVariable_Wrap(prop.unpack(self->cdata));
2206:   END_HANDLE_TH_ERRORS
2207: }
2208: 
2209: static PyObject* THPTakeBackward0_index_raw_getter(THPCppFunction *self, void *_unused) {
2210:   HANDLE_TH_ERRORS
2211:   const auto& prop = static_cast<TakeBackward0*>(self->cdata.get())->index_;
2212:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2213:   return obj.release().ptr();
2214:   END_HANDLE_TH_ERRORS
2215: }
2216: 
2217: static PyObject* THPTakeBackward0_self_getter(THPCppFunction *self, void *_unused) {
2218:   HANDLE_TH_ERRORS
2219:   const auto& prop = static_cast<TakeBackward0*>(self->cdata.get())->self_;
2220:   return THPVariable_Wrap(prop.unpack(self->cdata));
2221:   END_HANDLE_TH_ERRORS
2222: }
2223: 
2224: static PyObject* THPTakeBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
2225:   HANDLE_TH_ERRORS
2226:   const auto& prop = static_cast<TakeBackward0*>(self->cdata.get())->self_;
2227:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2228:   return obj.release().ptr();
2229:   END_HANDLE_TH_ERRORS
2230: }
2231: 
2232: static struct PyGetSetDef TakeBackward0_properties[] = {
2233:   THP_FUNCTION_DEFAULT_PROPERTIES,
2234:   {(char*)"_saved_index", (getter)THPTakeBackward0_index_getter, nullptr, nullptr, nullptr},
2235:   {(char*)"_raw_saved_index", (getter)THPTakeBackward0_index_raw_getter, nullptr, nullptr, nullptr},
2236:   {(char*)"_saved_self", (getter)THPTakeBackward0_self_getter, nullptr, nullptr, nullptr},
2237:   {(char*)"_raw_saved_self", (getter)THPTakeBackward0_self_raw_getter, nullptr, nullptr, nullptr},
2238:   {nullptr} /* sentinel */
2239: };
2240: 
```

- EN: The main execution path in this span is carried by `THPVariable_Wrap`, `THPLinalgSvdBackward0_U_raw_getter`, `cast`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_Wrap`, `THPLinalgSvdBackward0_U_raw_getter`, `cast` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2241-2320

```cpp
2241: static PyObject* THPTanBackward0_result_getter(THPCppFunction *self, void *_unused) {
2242:   HANDLE_TH_ERRORS
2243:   const auto& prop = static_cast<TanBackward0*>(self->cdata.get())->result_;
2244:   return THPVariable_Wrap(prop.unpack(self->cdata));
2245:   END_HANDLE_TH_ERRORS
2246: }
2247: 
2248: static PyObject* THPTanBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
2249:   HANDLE_TH_ERRORS
2250:   const auto& prop = static_cast<TanBackward0*>(self->cdata.get())->result_;
2251:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2252:   return obj.release().ptr();
2253:   END_HANDLE_TH_ERRORS
2254: }
2255: 
2256: static struct PyGetSetDef TanBackward0_properties[] = {
2257:   THP_FUNCTION_DEFAULT_PROPERTIES,
2258:   {(char*)"_saved_result", (getter)THPTanBackward0_result_getter, nullptr, nullptr, nullptr},
2259:   {(char*)"_raw_saved_result", (getter)THPTanBackward0_result_raw_getter, nullptr, nullptr, nullptr},
2260:   {nullptr} /* sentinel */
2261: };
2262: 
2263: static PyObject* THPTraceBackward0_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
2264:   HANDLE_TH_ERRORS
2265:   auto prop = static_cast<TraceBackward0*>(self->cdata.get())->self_sym_sizes;
2266:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
2267:   for (auto i : c10::irange(prop.size())) {
2268:       auto si = prop[i];
2269:       if (auto m = si.maybe_as_int()) {
2270:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
2271:       } else {
2272:         auto py_symint = py::cast(si).release().ptr();
2273:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
2274:       }
2275:   }
2276:   return tup;
2277:   END_HANDLE_TH_ERRORS
2278: }
2279: 
2280: static struct PyGetSetDef TraceBackward0_properties[] = {
2281:   THP_FUNCTION_DEFAULT_PROPERTIES,
2282:   {(char*)"_saved_self_sym_sizes", (getter)THPTraceBackward0_self_sym_sizes_getter, nullptr, nullptr, nullptr},
2283:   {nullptr} /* sentinel */
2284: };
2285: 
2286: static PyObject* THPToSparseCsrBackward0_self_self_sym_blocksize_opt_getter(THPCppFunction *self, void *_unused) {
2287:   HANDLE_TH_ERRORS
2288:   auto opt_prop = static_cast<ToSparseCsrBackward0*>(self->cdata.get())->self_self_sym_blocksize_opt;
2289:   if (!opt_prop.list.has_value()) {
2290:     Py_RETURN_NONE;
2291:   }
2292:   auto prop = opt_prop.list.value();
2293:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
2294:   for (auto i : c10::irange(prop.size())) {
2295:       auto si = prop[i];
2296:       if (auto m = si.maybe_as_int()) {
2297:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
2298:       } else {
2299:         auto py_symint = py::cast(si).release().ptr();
2300:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
2301:       }
2302:   }
2303:   return tup;
2304:   END_HANDLE_TH_ERRORS
2305: }
2306: 
2307: static struct PyGetSetDef ToSparseCsrBackward0_properties[] = {
2308:   THP_FUNCTION_DEFAULT_PROPERTIES,
2309:   {(char*)"_saved_self_self_sym_blocksize_opt", (getter)THPToSparseCsrBackward0_self_self_sym_blocksize_opt_getter, nullptr, nullptr, nullptr},
2310:   {nullptr} /* sentinel */
2311: };
2312: 
2313: static PyObject* THPToSparseBsrBackward0_self_self_sym_blocksize_opt_getter(THPCppFunction *self, void *_unused) {
2314:   HANDLE_TH_ERRORS
2315:   auto opt_prop = static_cast<ToSparseBsrBackward0*>(self->cdata.get())->self_self_sym_blocksize_opt;
2316:   if (!opt_prop.list.has_value()) {
2317:     Py_RETURN_NONE;
2318:   }
2319:   auto prop = opt_prop.list.value();
2320:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
```

- EN: The main execution path in this span is carried by `THPTanBackward0_result_getter`, `THPVariable_Wrap`, `THPTanBackward0_result_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPTanBackward0_result_getter`, `THPVariable_Wrap`, `THPTanBackward0_result_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2321-2400

```cpp
2321:   for (auto i : c10::irange(prop.size())) {
2322:       auto si = prop[i];
2323:       if (auto m = si.maybe_as_int()) {
2324:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
2325:       } else {
2326:         auto py_symint = py::cast(si).release().ptr();
2327:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
2328:       }
2329:   }
2330:   return tup;
2331:   END_HANDLE_TH_ERRORS
2332: }
2333: 
2334: static struct PyGetSetDef ToSparseBsrBackward0_properties[] = {
2335:   THP_FUNCTION_DEFAULT_PROPERTIES,
2336:   {(char*)"_saved_self_self_sym_blocksize_opt", (getter)THPToSparseBsrBackward0_self_self_sym_blocksize_opt_getter, nullptr, nullptr, nullptr},
2337:   {nullptr} /* sentinel */
2338: };
2339: 
2340: static PyObject* THPUnfoldBackward0_dimension_getter(THPCppFunction *self, void *_unused) {
2341:   HANDLE_TH_ERRORS
2342:   auto prop = static_cast<UnfoldBackward0*>(self->cdata.get())->dimension;
2343:   return PyLong_FromUnsignedLong((int64_t) prop);
2344:   END_HANDLE_TH_ERRORS
2345: }
2346: 
2347: static PyObject* THPUnfoldBackward0_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
2348:   HANDLE_TH_ERRORS
2349:   auto prop = static_cast<UnfoldBackward0*>(self->cdata.get())->self_sym_sizes;
2350:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
2351:   for (auto i : c10::irange(prop.size())) {
2352:       auto si = prop[i];
2353:       if (auto m = si.maybe_as_int()) {
2354:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
2355:       } else {
2356:         auto py_symint = py::cast(si).release().ptr();
2357:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
2358:       }
2359:   }
2360:   return tup;
2361:   END_HANDLE_TH_ERRORS
2362: }
2363: 
2364: static PyObject* THPUnfoldBackward0_size_getter(THPCppFunction *self, void *_unused) {
2365:   HANDLE_TH_ERRORS
2366:   auto prop = static_cast<UnfoldBackward0*>(self->cdata.get())->size;
2367:   return PyLong_FromUnsignedLong((int64_t) prop);
2368:   END_HANDLE_TH_ERRORS
2369: }
2370: 
2371: static PyObject* THPUnfoldBackward0_step_getter(THPCppFunction *self, void *_unused) {
2372:   HANDLE_TH_ERRORS
2373:   auto prop = static_cast<UnfoldBackward0*>(self->cdata.get())->step;
2374:   return PyLong_FromUnsignedLong((int64_t) prop);
2375:   END_HANDLE_TH_ERRORS
2376: }
2377: 
2378: static struct PyGetSetDef UnfoldBackward0_properties[] = {
2379:   THP_FUNCTION_DEFAULT_PROPERTIES,
2380:   {(char*)"_saved_dimension", (getter)THPUnfoldBackward0_dimension_getter, nullptr, nullptr, nullptr},
2381:   {(char*)"_saved_self_sym_sizes", (getter)THPUnfoldBackward0_self_sym_sizes_getter, nullptr, nullptr, nullptr},
2382:   {(char*)"_saved_size", (getter)THPUnfoldBackward0_size_getter, nullptr, nullptr, nullptr},
2383:   {(char*)"_saved_step", (getter)THPUnfoldBackward0_step_getter, nullptr, nullptr, nullptr},
2384:   {nullptr} /* sentinel */
2385: };
2386: 
2387: 
2388: 
2389: static struct PyGetSetDef UniqueDimConsecutiveBackward0_properties[] = {
2390:   THP_FUNCTION_DEFAULT_PROPERTIES,
2391: 
2392:   {nullptr} /* sentinel */
2393: };
2394: 
2395: static PyObject* THPUnsafeViewBackward0_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
2396:   HANDLE_TH_ERRORS
2397:   auto prop = static_cast<UnsafeViewBackward0*>(self->cdata.get())->self_sym_sizes;
2398:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
2399:   for (auto i : c10::irange(prop.size())) {
2400:       auto si = prop[i];
```

- EN: The main execution path in this span is carried by `PyTuple_SetItem`, `cast`, `THPUnfoldBackward0_dimension_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyTuple_SetItem`, `cast`, `THPUnfoldBackward0_dimension_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2401-2480

```cpp
2401:       if (auto m = si.maybe_as_int()) {
2402:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
2403:       } else {
2404:         auto py_symint = py::cast(si).release().ptr();
2405:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
2406:       }
2407:   }
2408:   return tup;
2409:   END_HANDLE_TH_ERRORS
2410: }
2411: 
2412: static struct PyGetSetDef UnsafeViewBackward0_properties[] = {
2413:   THP_FUNCTION_DEFAULT_PROPERTIES,
2414:   {(char*)"_saved_self_sym_sizes", (getter)THPUnsafeViewBackward0_self_sym_sizes_getter, nullptr, nullptr, nullptr},
2415:   {nullptr} /* sentinel */
2416: };
2417: 
2418: static PyObject* THPUnsqueezeBackward0_dim_getter(THPCppFunction *self, void *_unused) {
2419:   HANDLE_TH_ERRORS
2420:   auto prop = static_cast<UnsqueezeBackward0*>(self->cdata.get())->dim;
2421:   return PyLong_FromUnsignedLong((int64_t) prop);
2422:   END_HANDLE_TH_ERRORS
2423: }
2424: 
2425: static struct PyGetSetDef UnsqueezeBackward0_properties[] = {
2426:   THP_FUNCTION_DEFAULT_PROPERTIES,
2427:   {(char*)"_saved_dim", (getter)THPUnsqueezeBackward0_dim_getter, nullptr, nullptr, nullptr},
2428:   {nullptr} /* sentinel */
2429: };
2430: 
2431: static PyObject* THPUnsqueezeBackward1_dim_getter(THPCppFunction *self, void *_unused) {
2432:   HANDLE_TH_ERRORS
2433:   auto prop = static_cast<UnsqueezeBackward1*>(self->cdata.get())->dim;
2434:   return PyLong_FromUnsignedLong((int64_t) prop);
2435:   END_HANDLE_TH_ERRORS
2436: }
2437: 
2438: static struct PyGetSetDef UnsqueezeBackward1_properties[] = {
2439:   THP_FUNCTION_DEFAULT_PROPERTIES,
2440:   {(char*)"_saved_dim", (getter)THPUnsqueezeBackward1_dim_getter, nullptr, nullptr, nullptr},
2441:   {nullptr} /* sentinel */
2442: };
2443: 
2444: 
2445: 
2446: static struct PyGetSetDef ViewAsRealBackward0_properties[] = {
2447:   THP_FUNCTION_DEFAULT_PROPERTIES,
2448: 
2449:   {nullptr} /* sentinel */
2450: };
2451: 
2452: 
2453: 
2454: static struct PyGetSetDef ViewAsComplexBackward0_properties[] = {
2455:   THP_FUNCTION_DEFAULT_PROPERTIES,
2456: 
2457:   {nullptr} /* sentinel */
2458: };
2459: 
2460: static PyObject* THPSparseCompressedTensorBackward0_values_getter(THPCppFunction *self, void *_unused) {
2461:   HANDLE_TH_ERRORS
2462:   const auto& prop = static_cast<SparseCompressedTensorBackward0*>(self->cdata.get())->values_;
2463:   return THPVariable_Wrap(prop.unpack(self->cdata));
2464:   END_HANDLE_TH_ERRORS
2465: }
2466: 
2467: static PyObject* THPSparseCompressedTensorBackward0_values_raw_getter(THPCppFunction *self, void *_unused) {
2468:   HANDLE_TH_ERRORS
2469:   const auto& prop = static_cast<SparseCompressedTensorBackward0*>(self->cdata.get())->values_;
2470:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2471:   return obj.release().ptr();
2472:   END_HANDLE_TH_ERRORS
2473: }
2474: 
2475: static PyObject* THPSparseCompressedTensorBackward0_result_getter(THPCppFunction *self, void *_unused) {
2476:   HANDLE_TH_ERRORS
2477:   const auto& prop = static_cast<SparseCompressedTensorBackward0*>(self->cdata.get())->result_;
2478:   return THPVariable_Wrap(prop.unpack(self->cdata));
2479:   END_HANDLE_TH_ERRORS
2480: }
```

- EN: The main execution path in this span is carried by `PyTuple_SetItem`, `cast`, `THPUnsqueezeBackward0_dim_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyTuple_SetItem`, `cast`, `THPUnsqueezeBackward0_dim_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2481-2560

```cpp
2481: 
2482: static PyObject* THPSparseCompressedTensorBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
2483:   HANDLE_TH_ERRORS
2484:   const auto& prop = static_cast<SparseCompressedTensorBackward0*>(self->cdata.get())->result_;
2485:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2486:   return obj.release().ptr();
2487:   END_HANDLE_TH_ERRORS
2488: }
2489: 
2490: static struct PyGetSetDef SparseCompressedTensorBackward0_properties[] = {
2491:   THP_FUNCTION_DEFAULT_PROPERTIES,
2492:   {(char*)"_saved_values", (getter)THPSparseCompressedTensorBackward0_values_getter, nullptr, nullptr, nullptr},
2493:   {(char*)"_raw_saved_values", (getter)THPSparseCompressedTensorBackward0_values_raw_getter, nullptr, nullptr, nullptr},
2494:   {(char*)"_saved_result", (getter)THPSparseCompressedTensorBackward0_result_getter, nullptr, nullptr, nullptr},
2495:   {(char*)"_raw_saved_result", (getter)THPSparseCompressedTensorBackward0_result_raw_getter, nullptr, nullptr, nullptr},
2496:   {nullptr} /* sentinel */
2497: };
2498: 
2499: static PyObject* THPConstantPadNdBackward0_pad_getter(THPCppFunction *self, void *_unused) {
2500:   HANDLE_TH_ERRORS
2501:   auto prop = static_cast<ConstantPadNdBackward0*>(self->cdata.get())->pad;
2502:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
2503:   for (auto i : c10::irange(prop.size())) {
2504:       auto si = prop[i];
2505:       if (auto m = si.maybe_as_int()) {
2506:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
2507:       } else {
2508:         auto py_symint = py::cast(si).release().ptr();
2509:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
2510:       }
2511:   }
2512:   return tup;
2513:   END_HANDLE_TH_ERRORS
2514: }
2515: 
2516: static struct PyGetSetDef ConstantPadNdBackward0_properties[] = {
2517:   THP_FUNCTION_DEFAULT_PROPERTIES,
2518:   {(char*)"_saved_pad", (getter)THPConstantPadNdBackward0_pad_getter, nullptr, nullptr, nullptr},
2519:   {nullptr} /* sentinel */
2520: };
2521: 
2522: static PyObject* THPEmbeddingBackward0_indices_getter(THPCppFunction *self, void *_unused) {
2523:   HANDLE_TH_ERRORS
2524:   const auto& prop = static_cast<EmbeddingBackward0*>(self->cdata.get())->indices_;
2525:   return THPVariable_Wrap(prop.unpack(self->cdata));
2526:   END_HANDLE_TH_ERRORS
2527: }
2528: 
2529: static PyObject* THPEmbeddingBackward0_indices_raw_getter(THPCppFunction *self, void *_unused) {
2530:   HANDLE_TH_ERRORS
2531:   const auto& prop = static_cast<EmbeddingBackward0*>(self->cdata.get())->indices_;
2532:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2533:   return obj.release().ptr();
2534:   END_HANDLE_TH_ERRORS
2535: }
2536: 
2537: static PyObject* THPEmbeddingBackward0_padding_idx_getter(THPCppFunction *self, void *_unused) {
2538:   HANDLE_TH_ERRORS
2539:   auto prop = static_cast<EmbeddingBackward0*>(self->cdata.get())->padding_idx;
2540:   if (auto m = prop.maybe_as_int()) {
2541:     return PyLong_FromUnsignedLong(*m);
2542:   } else {
2543:     return py::cast(prop).release().ptr();
2544:   }
2545:   END_HANDLE_TH_ERRORS
2546: }
2547: 
2548: static PyObject* THPEmbeddingBackward0_scale_grad_by_freq_getter(THPCppFunction *self, void *_unused) {
2549:   HANDLE_TH_ERRORS
2550:   auto prop = static_cast<EmbeddingBackward0*>(self->cdata.get())->scale_grad_by_freq;
2551:   if (prop) {
2552:     Py_RETURN_TRUE;
2553:   } else {
2554:     Py_RETURN_FALSE;
2555:   }
2556:   END_HANDLE_TH_ERRORS
2557: }
2558: 
2559: static PyObject* THPEmbeddingBackward0_sparse_getter(THPCppFunction *self, void *_unused) {
2560:   HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPSparseCompressedTensorBackward0_result_raw_getter`, `cast`, `THPConstantPadNdBackward0_pad_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPSparseCompressedTensorBackward0_result_raw_getter`, `cast`, `THPConstantPadNdBackward0_pad_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2561-2640

```cpp
2561:   auto prop = static_cast<EmbeddingBackward0*>(self->cdata.get())->sparse;
2562:   if (prop) {
2563:     Py_RETURN_TRUE;
2564:   } else {
2565:     Py_RETURN_FALSE;
2566:   }
2567:   END_HANDLE_TH_ERRORS
2568: }
2569: 
2570: static PyObject* THPEmbeddingBackward0_weight_sym_argsize_0_getter(THPCppFunction *self, void *_unused) {
2571:   HANDLE_TH_ERRORS
2572:   auto prop = static_cast<EmbeddingBackward0*>(self->cdata.get())->weight_sym_argsize_0;
2573:   if (auto m = prop.maybe_as_int()) {
2574:     return PyLong_FromUnsignedLong(*m);
2575:   } else {
2576:     return py::cast(prop).release().ptr();
2577:   }
2578:   END_HANDLE_TH_ERRORS
2579: }
2580: 
2581: static struct PyGetSetDef EmbeddingBackward0_properties[] = {
2582:   THP_FUNCTION_DEFAULT_PROPERTIES,
2583:   {(char*)"_saved_indices", (getter)THPEmbeddingBackward0_indices_getter, nullptr, nullptr, nullptr},
2584:   {(char*)"_raw_saved_indices", (getter)THPEmbeddingBackward0_indices_raw_getter, nullptr, nullptr, nullptr},
2585:   {(char*)"_saved_padding_idx", (getter)THPEmbeddingBackward0_padding_idx_getter, nullptr, nullptr, nullptr},
2586:   {(char*)"_saved_scale_grad_by_freq", (getter)THPEmbeddingBackward0_scale_grad_by_freq_getter, nullptr, nullptr, nullptr},
2587:   {(char*)"_saved_sparse", (getter)THPEmbeddingBackward0_sparse_getter, nullptr, nullptr, nullptr},
2588:   {(char*)"_saved_weight_sym_argsize_0", (getter)THPEmbeddingBackward0_weight_sym_argsize_0_getter, nullptr, nullptr, nullptr},
2589:   {nullptr} /* sentinel */
2590: };
2591: 
2592: 
2593: 
2594: static struct PyGetSetDef EmbeddingBagBackwardBackward0_properties[] = {
2595:   THP_FUNCTION_DEFAULT_PROPERTIES,
2596: 
2597:   {nullptr} /* sentinel */
2598: };
2599: 
2600: 
2601: 
2602: static struct PyGetSetDef EmbeddingBagDenseBackwardBackward0_properties[] = {
2603:   THP_FUNCTION_DEFAULT_PROPERTIES,
2604: 
2605:   {nullptr} /* sentinel */
2606: };
2607: 
2608: static PyObject* THPSmoothL1LossBackward0_beta_getter(THPCppFunction *self, void *_unused) {
2609:   HANDLE_TH_ERRORS
2610:   auto prop = static_cast<SmoothL1LossBackward0*>(self->cdata.get())->beta;
2611:   return PyFloat_FromDouble((double) prop);
2612:   END_HANDLE_TH_ERRORS
2613: }
2614: 
2615: static PyObject* THPSmoothL1LossBackward0_reduction_getter(THPCppFunction *self, void *_unused) {
2616:   HANDLE_TH_ERRORS
2617:   auto prop = static_cast<SmoothL1LossBackward0*>(self->cdata.get())->reduction;
2618:   return PyLong_FromUnsignedLong((int64_t) prop);
2619:   END_HANDLE_TH_ERRORS
2620: }
2621: 
2622: static PyObject* THPSmoothL1LossBackward0_self_getter(THPCppFunction *self, void *_unused) {
2623:   HANDLE_TH_ERRORS
2624:   const auto& prop = static_cast<SmoothL1LossBackward0*>(self->cdata.get())->self_;
2625:   return THPVariable_Wrap(prop.unpack(self->cdata));
2626:   END_HANDLE_TH_ERRORS
2627: }
2628: 
2629: static PyObject* THPSmoothL1LossBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
2630:   HANDLE_TH_ERRORS
2631:   const auto& prop = static_cast<SmoothL1LossBackward0*>(self->cdata.get())->self_;
2632:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2633:   return obj.release().ptr();
2634:   END_HANDLE_TH_ERRORS
2635: }
2636: 
2637: static PyObject* THPSmoothL1LossBackward0_target_getter(THPCppFunction *self, void *_unused) {
2638:   HANDLE_TH_ERRORS
2639:   const auto& prop = static_cast<SmoothL1LossBackward0*>(self->cdata.get())->target_;
2640:   return THPVariable_Wrap(prop.unpack(self->cdata));
```

- EN: The main execution path in this span is carried by `THPEmbeddingBackward0_weight_sym_argsize_0_getter`, `PyLong_FromUnsignedLong`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPEmbeddingBackward0_weight_sym_argsize_0_getter`, `PyLong_FromUnsignedLong`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2641-2720

```cpp
2641:   END_HANDLE_TH_ERRORS
2642: }
2643: 
2644: static PyObject* THPSmoothL1LossBackward0_target_raw_getter(THPCppFunction *self, void *_unused) {
2645:   HANDLE_TH_ERRORS
2646:   const auto& prop = static_cast<SmoothL1LossBackward0*>(self->cdata.get())->target_;
2647:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2648:   return obj.release().ptr();
2649:   END_HANDLE_TH_ERRORS
2650: }
2651: 
2652: static struct PyGetSetDef SmoothL1LossBackward0_properties[] = {
2653:   THP_FUNCTION_DEFAULT_PROPERTIES,
2654:   {(char*)"_saved_beta", (getter)THPSmoothL1LossBackward0_beta_getter, nullptr, nullptr, nullptr},
2655:   {(char*)"_saved_reduction", (getter)THPSmoothL1LossBackward0_reduction_getter, nullptr, nullptr, nullptr},
2656:   {(char*)"_saved_self", (getter)THPSmoothL1LossBackward0_self_getter, nullptr, nullptr, nullptr},
2657:   {(char*)"_raw_saved_self", (getter)THPSmoothL1LossBackward0_self_raw_getter, nullptr, nullptr, nullptr},
2658:   {(char*)"_saved_target", (getter)THPSmoothL1LossBackward0_target_getter, nullptr, nullptr, nullptr},
2659:   {(char*)"_raw_saved_target", (getter)THPSmoothL1LossBackward0_target_raw_getter, nullptr, nullptr, nullptr},
2660:   {nullptr} /* sentinel */
2661: };
2662: 
2663: static PyObject* THPReluBackward0_result_getter(THPCppFunction *self, void *_unused) {
2664:   HANDLE_TH_ERRORS
2665:   const auto& prop = static_cast<ReluBackward0*>(self->cdata.get())->result_;
2666:   return THPVariable_Wrap(prop.unpack(self->cdata));
2667:   END_HANDLE_TH_ERRORS
2668: }
2669: 
2670: static PyObject* THPReluBackward0_result_raw_getter(THPCppFunction *self, void *_unused) {
2671:   HANDLE_TH_ERRORS
2672:   const auto& prop = static_cast<ReluBackward0*>(self->cdata.get())->result_;
2673:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2674:   return obj.release().ptr();
2675:   END_HANDLE_TH_ERRORS
2676: }
2677: 
2678: static struct PyGetSetDef ReluBackward0_properties[] = {
2679:   THP_FUNCTION_DEFAULT_PROPERTIES,
2680:   {(char*)"_saved_result", (getter)THPReluBackward0_result_getter, nullptr, nullptr, nullptr},
2681:   {(char*)"_raw_saved_result", (getter)THPReluBackward0_result_raw_getter, nullptr, nullptr, nullptr},
2682:   {nullptr} /* sentinel */
2683: };
2684: 
2685: static PyObject* THPEluBackward0_alpha_getter(THPCppFunction *self, void *_unused) {
2686:   HANDLE_TH_ERRORS
2687:   auto prop = static_cast<EluBackward0*>(self->cdata.get())->alpha;
2688:   if (prop.isComplex()) {
2689:     auto cprop = prop.to<c10::complex<double>>();
2690:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
2691:   } else if (prop.isFloatingPoint()) {
2692:     return PyFloat_FromDouble(prop.to<double>());
2693:   } else if (prop.isIntegral(/*includeBool=*/false)) {
2694:     return PyLong_FromLong(prop.to<int64_t>());
2695:   } else if (prop.isBoolean()) {
2696:     if (prop.to<bool>()) {
2697:       Py_RETURN_TRUE;
2698:     } else {
2699:       Py_RETURN_FALSE;
2700:     }
2701:   } else {
2702:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
2703:     return nullptr;
2704:   }
2705:   END_HANDLE_TH_ERRORS
2706: }
2707: 
2708: static PyObject* THPEluBackward0_input_scale_getter(THPCppFunction *self, void *_unused) {
2709:   HANDLE_TH_ERRORS
2710:   auto prop = static_cast<EluBackward0*>(self->cdata.get())->input_scale;
2711:   if (prop.isComplex()) {
2712:     auto cprop = prop.to<c10::complex<double>>();
2713:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
2714:   } else if (prop.isFloatingPoint()) {
2715:     return PyFloat_FromDouble(prop.to<double>());
2716:   } else if (prop.isIntegral(/*includeBool=*/false)) {
2717:     return PyLong_FromLong(prop.to<int64_t>());
2718:   } else if (prop.isBoolean()) {
2719:     if (prop.to<bool>()) {
2720:       Py_RETURN_TRUE;
```

- EN: The main execution path in this span is carried by `THPSmoothL1LossBackward0_target_raw_getter`, `cast`, `THPReluBackward0_result_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPSmoothL1LossBackward0_target_raw_getter`, `cast`, `THPReluBackward0_result_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2721-2800

```cpp
2721:     } else {
2722:       Py_RETURN_FALSE;
2723:     }
2724:   } else {
2725:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
2726:     return nullptr;
2727:   }
2728:   END_HANDLE_TH_ERRORS
2729: }
2730: 
2731: static PyObject* THPEluBackward0_scale_getter(THPCppFunction *self, void *_unused) {
2732:   HANDLE_TH_ERRORS
2733:   auto prop = static_cast<EluBackward0*>(self->cdata.get())->scale;
2734:   if (prop.isComplex()) {
2735:     auto cprop = prop.to<c10::complex<double>>();
2736:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
2737:   } else if (prop.isFloatingPoint()) {
2738:     return PyFloat_FromDouble(prop.to<double>());
2739:   } else if (prop.isIntegral(/*includeBool=*/false)) {
2740:     return PyLong_FromLong(prop.to<int64_t>());
2741:   } else if (prop.isBoolean()) {
2742:     if (prop.to<bool>()) {
2743:       Py_RETURN_TRUE;
2744:     } else {
2745:       Py_RETURN_FALSE;
2746:     }
2747:   } else {
2748:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
2749:     return nullptr;
2750:   }
2751:   END_HANDLE_TH_ERRORS
2752: }
2753: 
2754: static PyObject* THPEluBackward0_self_getter(THPCppFunction *self, void *_unused) {
2755:   HANDLE_TH_ERRORS
2756:   const auto& prop = static_cast<EluBackward0*>(self->cdata.get())->self_;
2757:   return THPVariable_Wrap(prop.unpack(self->cdata));
2758:   END_HANDLE_TH_ERRORS
2759: }
2760: 
2761: static PyObject* THPEluBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
2762:   HANDLE_TH_ERRORS
2763:   const auto& prop = static_cast<EluBackward0*>(self->cdata.get())->self_;
2764:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2765:   return obj.release().ptr();
2766:   END_HANDLE_TH_ERRORS
2767: }
2768: 
2769: static struct PyGetSetDef EluBackward0_properties[] = {
2770:   THP_FUNCTION_DEFAULT_PROPERTIES,
2771:   {(char*)"_saved_alpha", (getter)THPEluBackward0_alpha_getter, nullptr, nullptr, nullptr},
2772:   {(char*)"_saved_input_scale", (getter)THPEluBackward0_input_scale_getter, nullptr, nullptr, nullptr},
2773:   {(char*)"_saved_scale", (getter)THPEluBackward0_scale_getter, nullptr, nullptr, nullptr},
2774:   {(char*)"_saved_self", (getter)THPEluBackward0_self_getter, nullptr, nullptr, nullptr},
2775:   {(char*)"_raw_saved_self", (getter)THPEluBackward0_self_raw_getter, nullptr, nullptr, nullptr},
2776:   {nullptr} /* sentinel */
2777: };
2778: 
2779: static PyObject* THPEluBackward1_alpha_getter(THPCppFunction *self, void *_unused) {
2780:   HANDLE_TH_ERRORS
2781:   auto prop = static_cast<EluBackward1*>(self->cdata.get())->alpha;
2782:   if (prop.isComplex()) {
2783:     auto cprop = prop.to<c10::complex<double>>();
2784:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
2785:   } else if (prop.isFloatingPoint()) {
2786:     return PyFloat_FromDouble(prop.to<double>());
2787:   } else if (prop.isIntegral(/*includeBool=*/false)) {
2788:     return PyLong_FromLong(prop.to<int64_t>());
2789:   } else if (prop.isBoolean()) {
2790:     if (prop.to<bool>()) {
2791:       Py_RETURN_TRUE;
2792:     } else {
2793:       Py_RETURN_FALSE;
2794:     }
2795:   } else {
2796:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
2797:     return nullptr;
2798:   }
2799:   END_HANDLE_TH_ERRORS
2800: }
```

- EN: The main execution path in this span is carried by `PyErr_SetString`, `THPEluBackward0_scale_getter`, `PyComplex_FromDoubles`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyErr_SetString`, `THPEluBackward0_scale_getter`, `PyComplex_FromDoubles` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2801-2880

```cpp
2801: 
2802: static PyObject* THPEluBackward1_input_scale_getter(THPCppFunction *self, void *_unused) {
2803:   HANDLE_TH_ERRORS
2804:   auto prop = static_cast<EluBackward1*>(self->cdata.get())->input_scale;
2805:   if (prop.isComplex()) {
2806:     auto cprop = prop.to<c10::complex<double>>();
2807:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
2808:   } else if (prop.isFloatingPoint()) {
2809:     return PyFloat_FromDouble(prop.to<double>());
2810:   } else if (prop.isIntegral(/*includeBool=*/false)) {
2811:     return PyLong_FromLong(prop.to<int64_t>());
2812:   } else if (prop.isBoolean()) {
2813:     if (prop.to<bool>()) {
2814:       Py_RETURN_TRUE;
2815:     } else {
2816:       Py_RETURN_FALSE;
2817:     }
2818:   } else {
2819:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
2820:     return nullptr;
2821:   }
2822:   END_HANDLE_TH_ERRORS
2823: }
2824: 
2825: static PyObject* THPEluBackward1_scale_getter(THPCppFunction *self, void *_unused) {
2826:   HANDLE_TH_ERRORS
2827:   auto prop = static_cast<EluBackward1*>(self->cdata.get())->scale;
2828:   if (prop.isComplex()) {
2829:     auto cprop = prop.to<c10::complex<double>>();
2830:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
2831:   } else if (prop.isFloatingPoint()) {
2832:     return PyFloat_FromDouble(prop.to<double>());
2833:   } else if (prop.isIntegral(/*includeBool=*/false)) {
2834:     return PyLong_FromLong(prop.to<int64_t>());
2835:   } else if (prop.isBoolean()) {
2836:     if (prop.to<bool>()) {
2837:       Py_RETURN_TRUE;
2838:     } else {
2839:       Py_RETURN_FALSE;
2840:     }
2841:   } else {
2842:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
2843:     return nullptr;
2844:   }
2845:   END_HANDLE_TH_ERRORS
2846: }
2847: 
2848: static PyObject* THPEluBackward1_result_getter(THPCppFunction *self, void *_unused) {
2849:   HANDLE_TH_ERRORS
2850:   const auto& prop = static_cast<EluBackward1*>(self->cdata.get())->result_;
2851:   return THPVariable_Wrap(prop.unpack(self->cdata));
2852:   END_HANDLE_TH_ERRORS
2853: }
2854: 
2855: static PyObject* THPEluBackward1_result_raw_getter(THPCppFunction *self, void *_unused) {
2856:   HANDLE_TH_ERRORS
2857:   const auto& prop = static_cast<EluBackward1*>(self->cdata.get())->result_;
2858:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2859:   return obj.release().ptr();
2860:   END_HANDLE_TH_ERRORS
2861: }
2862: 
2863: static struct PyGetSetDef EluBackward1_properties[] = {
2864:   THP_FUNCTION_DEFAULT_PROPERTIES,
2865:   {(char*)"_saved_alpha", (getter)THPEluBackward1_alpha_getter, nullptr, nullptr, nullptr},
2866:   {(char*)"_saved_input_scale", (getter)THPEluBackward1_input_scale_getter, nullptr, nullptr, nullptr},
2867:   {(char*)"_saved_scale", (getter)THPEluBackward1_scale_getter, nullptr, nullptr, nullptr},
2868:   {(char*)"_saved_result", (getter)THPEluBackward1_result_getter, nullptr, nullptr, nullptr},
2869:   {(char*)"_raw_saved_result", (getter)THPEluBackward1_result_raw_getter, nullptr, nullptr, nullptr},
2870:   {nullptr} /* sentinel */
2871: };
2872: 
2873: static PyObject* THPHardshrinkBackwardBackward0_lambd_getter(THPCppFunction *self, void *_unused) {
2874:   HANDLE_TH_ERRORS
2875:   auto prop = static_cast<HardshrinkBackwardBackward0*>(self->cdata.get())->lambd;
2876:   if (prop.isComplex()) {
2877:     auto cprop = prop.to<c10::complex<double>>();
2878:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
2879:   } else if (prop.isFloatingPoint()) {
2880:     return PyFloat_FromDouble(prop.to<double>());
```

- EN: The main execution path in this span is carried by `THPEluBackward1_input_scale_getter`, `PyComplex_FromDoubles`, `PyFloat_FromDouble`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPEluBackward1_input_scale_getter`, `PyComplex_FromDoubles`, `PyFloat_FromDouble` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2881-2960

```cpp
2881:   } else if (prop.isIntegral(/*includeBool=*/false)) {
2882:     return PyLong_FromLong(prop.to<int64_t>());
2883:   } else if (prop.isBoolean()) {
2884:     if (prop.to<bool>()) {
2885:       Py_RETURN_TRUE;
2886:     } else {
2887:       Py_RETURN_FALSE;
2888:     }
2889:   } else {
2890:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
2891:     return nullptr;
2892:   }
2893:   END_HANDLE_TH_ERRORS
2894: }
2895: 
2896: static PyObject* THPHardshrinkBackwardBackward0_self_getter(THPCppFunction *self, void *_unused) {
2897:   HANDLE_TH_ERRORS
2898:   const auto& prop = static_cast<HardshrinkBackwardBackward0*>(self->cdata.get())->self_;
2899:   return THPVariable_Wrap(prop.unpack(self->cdata));
2900:   END_HANDLE_TH_ERRORS
2901: }
2902: 
2903: static PyObject* THPHardshrinkBackwardBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
2904:   HANDLE_TH_ERRORS
2905:   const auto& prop = static_cast<HardshrinkBackwardBackward0*>(self->cdata.get())->self_;
2906:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2907:   return obj.release().ptr();
2908:   END_HANDLE_TH_ERRORS
2909: }
2910: 
2911: static struct PyGetSetDef HardshrinkBackwardBackward0_properties[] = {
2912:   THP_FUNCTION_DEFAULT_PROPERTIES,
2913:   {(char*)"_saved_lambd", (getter)THPHardshrinkBackwardBackward0_lambd_getter, nullptr, nullptr, nullptr},
2914:   {(char*)"_saved_self", (getter)THPHardshrinkBackwardBackward0_self_getter, nullptr, nullptr, nullptr},
2915:   {(char*)"_raw_saved_self", (getter)THPHardshrinkBackwardBackward0_self_raw_getter, nullptr, nullptr, nullptr},
2916:   {nullptr} /* sentinel */
2917: };
2918: 
2919: static PyObject* THPLeakyReluBackward1_negative_slope_getter(THPCppFunction *self, void *_unused) {
2920:   HANDLE_TH_ERRORS
2921:   auto prop = static_cast<LeakyReluBackward1*>(self->cdata.get())->negative_slope;
2922:   if (prop.isComplex()) {
2923:     auto cprop = prop.to<c10::complex<double>>();
2924:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
2925:   } else if (prop.isFloatingPoint()) {
2926:     return PyFloat_FromDouble(prop.to<double>());
2927:   } else if (prop.isIntegral(/*includeBool=*/false)) {
2928:     return PyLong_FromLong(prop.to<int64_t>());
2929:   } else if (prop.isBoolean()) {
2930:     if (prop.to<bool>()) {
2931:       Py_RETURN_TRUE;
2932:     } else {
2933:       Py_RETURN_FALSE;
2934:     }
2935:   } else {
2936:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
2937:     return nullptr;
2938:   }
2939:   END_HANDLE_TH_ERRORS
2940: }
2941: 
2942: static PyObject* THPLeakyReluBackward1_result_getter(THPCppFunction *self, void *_unused) {
2943:   HANDLE_TH_ERRORS
2944:   const auto& prop = static_cast<LeakyReluBackward1*>(self->cdata.get())->result_;
2945:   return THPVariable_Wrap(prop.unpack(self->cdata));
2946:   END_HANDLE_TH_ERRORS
2947: }
2948: 
2949: static PyObject* THPLeakyReluBackward1_result_raw_getter(THPCppFunction *self, void *_unused) {
2950:   HANDLE_TH_ERRORS
2951:   const auto& prop = static_cast<LeakyReluBackward1*>(self->cdata.get())->result_;
2952:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2953:   return obj.release().ptr();
2954:   END_HANDLE_TH_ERRORS
2955: }
2956: 
2957: static struct PyGetSetDef LeakyReluBackward1_properties[] = {
2958:   THP_FUNCTION_DEFAULT_PROPERTIES,
2959:   {(char*)"_saved_negative_slope", (getter)THPLeakyReluBackward1_negative_slope_getter, nullptr, nullptr, nullptr},
2960:   {(char*)"_saved_result", (getter)THPLeakyReluBackward1_result_getter, nullptr, nullptr, nullptr},
```

- EN: The main execution path in this span is carried by `PyLong_FromLong`, `PyErr_SetString`, `THPHardshrinkBackwardBackward0_self_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyLong_FromLong`, `PyErr_SetString`, `THPHardshrinkBackwardBackward0_self_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2961-3040

```cpp
2961:   {(char*)"_raw_saved_result", (getter)THPLeakyReluBackward1_result_raw_getter, nullptr, nullptr, nullptr},
2962:   {nullptr} /* sentinel */
2963: };
2964: 
2965: static PyObject* THPRreluWithNoiseFunctionalBackward0_lower_getter(THPCppFunction *self, void *_unused) {
2966:   HANDLE_TH_ERRORS
2967:   auto prop = static_cast<RreluWithNoiseFunctionalBackward0*>(self->cdata.get())->lower;
2968:   if (prop.isComplex()) {
2969:     auto cprop = prop.to<c10::complex<double>>();
2970:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
2971:   } else if (prop.isFloatingPoint()) {
2972:     return PyFloat_FromDouble(prop.to<double>());
2973:   } else if (prop.isIntegral(/*includeBool=*/false)) {
2974:     return PyLong_FromLong(prop.to<int64_t>());
2975:   } else if (prop.isBoolean()) {
2976:     if (prop.to<bool>()) {
2977:       Py_RETURN_TRUE;
2978:     } else {
2979:       Py_RETURN_FALSE;
2980:     }
2981:   } else {
2982:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
2983:     return nullptr;
2984:   }
2985:   END_HANDLE_TH_ERRORS
2986: }
2987: 
2988: static PyObject* THPRreluWithNoiseFunctionalBackward0_noise_getter(THPCppFunction *self, void *_unused) {
2989:   HANDLE_TH_ERRORS
2990:   const auto& prop = static_cast<RreluWithNoiseFunctionalBackward0*>(self->cdata.get())->noise_;
2991:   return THPVariable_Wrap(prop.unpack(self->cdata));
2992:   END_HANDLE_TH_ERRORS
2993: }
2994: 
2995: static PyObject* THPRreluWithNoiseFunctionalBackward0_noise_raw_getter(THPCppFunction *self, void *_unused) {
2996:   HANDLE_TH_ERRORS
2997:   const auto& prop = static_cast<RreluWithNoiseFunctionalBackward0*>(self->cdata.get())->noise_;
2998:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
2999:   return obj.release().ptr();
3000:   END_HANDLE_TH_ERRORS
3001: }
3002: 
3003: static PyObject* THPRreluWithNoiseFunctionalBackward0_self_getter(THPCppFunction *self, void *_unused) {
3004:   HANDLE_TH_ERRORS
3005:   const auto& prop = static_cast<RreluWithNoiseFunctionalBackward0*>(self->cdata.get())->self_;
3006:   return THPVariable_Wrap(prop.unpack(self->cdata));
3007:   END_HANDLE_TH_ERRORS
3008: }
3009: 
3010: static PyObject* THPRreluWithNoiseFunctionalBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
3011:   HANDLE_TH_ERRORS
3012:   const auto& prop = static_cast<RreluWithNoiseFunctionalBackward0*>(self->cdata.get())->self_;
3013:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3014:   return obj.release().ptr();
3015:   END_HANDLE_TH_ERRORS
3016: }
3017: 
3018: static PyObject* THPRreluWithNoiseFunctionalBackward0_training_getter(THPCppFunction *self, void *_unused) {
3019:   HANDLE_TH_ERRORS
3020:   auto prop = static_cast<RreluWithNoiseFunctionalBackward0*>(self->cdata.get())->training;
3021:   if (prop) {
3022:     Py_RETURN_TRUE;
3023:   } else {
3024:     Py_RETURN_FALSE;
3025:   }
3026:   END_HANDLE_TH_ERRORS
3027: }
3028: 
3029: static PyObject* THPRreluWithNoiseFunctionalBackward0_upper_getter(THPCppFunction *self, void *_unused) {
3030:   HANDLE_TH_ERRORS
3031:   auto prop = static_cast<RreluWithNoiseFunctionalBackward0*>(self->cdata.get())->upper;
3032:   if (prop.isComplex()) {
3033:     auto cprop = prop.to<c10::complex<double>>();
3034:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
3035:   } else if (prop.isFloatingPoint()) {
3036:     return PyFloat_FromDouble(prop.to<double>());
3037:   } else if (prop.isIntegral(/*includeBool=*/false)) {
3038:     return PyLong_FromLong(prop.to<int64_t>());
3039:   } else if (prop.isBoolean()) {
3040:     if (prop.to<bool>()) {
```

- EN: The main execution path in this span is carried by `THPRreluWithNoiseFunctionalBackward0_lower_getter`, `PyComplex_FromDoubles`, `PyFloat_FromDouble`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPRreluWithNoiseFunctionalBackward0_lower_getter`, `PyComplex_FromDoubles`, `PyFloat_FromDouble` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3041-3120

```cpp
3041:       Py_RETURN_TRUE;
3042:     } else {
3043:       Py_RETURN_FALSE;
3044:     }
3045:   } else {
3046:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
3047:     return nullptr;
3048:   }
3049:   END_HANDLE_TH_ERRORS
3050: }
3051: 
3052: static struct PyGetSetDef RreluWithNoiseFunctionalBackward0_properties[] = {
3053:   THP_FUNCTION_DEFAULT_PROPERTIES,
3054:   {(char*)"_saved_lower", (getter)THPRreluWithNoiseFunctionalBackward0_lower_getter, nullptr, nullptr, nullptr},
3055:   {(char*)"_saved_noise", (getter)THPRreluWithNoiseFunctionalBackward0_noise_getter, nullptr, nullptr, nullptr},
3056:   {(char*)"_raw_saved_noise", (getter)THPRreluWithNoiseFunctionalBackward0_noise_raw_getter, nullptr, nullptr, nullptr},
3057:   {(char*)"_saved_self", (getter)THPRreluWithNoiseFunctionalBackward0_self_getter, nullptr, nullptr, nullptr},
3058:   {(char*)"_raw_saved_self", (getter)THPRreluWithNoiseFunctionalBackward0_self_raw_getter, nullptr, nullptr, nullptr},
3059:   {(char*)"_saved_training", (getter)THPRreluWithNoiseFunctionalBackward0_training_getter, nullptr, nullptr, nullptr},
3060:   {(char*)"_saved_upper", (getter)THPRreluWithNoiseFunctionalBackward0_upper_getter, nullptr, nullptr, nullptr},
3061:   {nullptr} /* sentinel */
3062: };
3063: 
3064: static PyObject* THPSoftplusBackward0_beta_getter(THPCppFunction *self, void *_unused) {
3065:   HANDLE_TH_ERRORS
3066:   auto prop = static_cast<SoftplusBackward0*>(self->cdata.get())->beta;
3067:   if (prop.isComplex()) {
3068:     auto cprop = prop.to<c10::complex<double>>();
3069:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
3070:   } else if (prop.isFloatingPoint()) {
3071:     return PyFloat_FromDouble(prop.to<double>());
3072:   } else if (prop.isIntegral(/*includeBool=*/false)) {
3073:     return PyLong_FromLong(prop.to<int64_t>());
3074:   } else if (prop.isBoolean()) {
3075:     if (prop.to<bool>()) {
3076:       Py_RETURN_TRUE;
3077:     } else {
3078:       Py_RETURN_FALSE;
3079:     }
3080:   } else {
3081:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
3082:     return nullptr;
3083:   }
3084:   END_HANDLE_TH_ERRORS
3085: }
3086: 
3087: static PyObject* THPSoftplusBackward0_self_getter(THPCppFunction *self, void *_unused) {
3088:   HANDLE_TH_ERRORS
3089:   const auto& prop = static_cast<SoftplusBackward0*>(self->cdata.get())->self_;
3090:   return THPVariable_Wrap(prop.unpack(self->cdata));
3091:   END_HANDLE_TH_ERRORS
3092: }
3093: 
3094: static PyObject* THPSoftplusBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
3095:   HANDLE_TH_ERRORS
3096:   const auto& prop = static_cast<SoftplusBackward0*>(self->cdata.get())->self_;
3097:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3098:   return obj.release().ptr();
3099:   END_HANDLE_TH_ERRORS
3100: }
3101: 
3102: static PyObject* THPSoftplusBackward0_threshold_getter(THPCppFunction *self, void *_unused) {
3103:   HANDLE_TH_ERRORS
3104:   auto prop = static_cast<SoftplusBackward0*>(self->cdata.get())->threshold;
3105:   if (prop.isComplex()) {
3106:     auto cprop = prop.to<c10::complex<double>>();
3107:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
3108:   } else if (prop.isFloatingPoint()) {
3109:     return PyFloat_FromDouble(prop.to<double>());
3110:   } else if (prop.isIntegral(/*includeBool=*/false)) {
3111:     return PyLong_FromLong(prop.to<int64_t>());
3112:   } else if (prop.isBoolean()) {
3113:     if (prop.to<bool>()) {
3114:       Py_RETURN_TRUE;
3115:     } else {
3116:       Py_RETURN_FALSE;
3117:     }
3118:   } else {
3119:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
3120:     return nullptr;
```

- EN: The main execution path in this span is carried by `PyErr_SetString`, `THPSoftplusBackward0_beta_getter`, `PyComplex_FromDoubles`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyErr_SetString`, `THPSoftplusBackward0_beta_getter`, `PyComplex_FromDoubles` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3121-3200

```cpp
3121:   }
3122:   END_HANDLE_TH_ERRORS
3123: }
3124: 
3125: static struct PyGetSetDef SoftplusBackward0_properties[] = {
3126:   THP_FUNCTION_DEFAULT_PROPERTIES,
3127:   {(char*)"_saved_beta", (getter)THPSoftplusBackward0_beta_getter, nullptr, nullptr, nullptr},
3128:   {(char*)"_saved_self", (getter)THPSoftplusBackward0_self_getter, nullptr, nullptr, nullptr},
3129:   {(char*)"_raw_saved_self", (getter)THPSoftplusBackward0_self_raw_getter, nullptr, nullptr, nullptr},
3130:   {(char*)"_saved_threshold", (getter)THPSoftplusBackward0_threshold_getter, nullptr, nullptr, nullptr},
3131:   {nullptr} /* sentinel */
3132: };
3133: 
3134: static PyObject* THPReflectionPad2DBackward0_padding_getter(THPCppFunction *self, void *_unused) {
3135:   HANDLE_TH_ERRORS
3136:   auto prop = static_cast<ReflectionPad2DBackward0*>(self->cdata.get())->padding;
3137:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3138:   for (auto i : c10::irange(prop.size())) {
3139:       auto si = prop[i];
3140:       if (auto m = si.maybe_as_int()) {
3141:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3142:       } else {
3143:         auto py_symint = py::cast(si).release().ptr();
3144:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3145:       }
3146:   }
3147:   return tup;
3148:   END_HANDLE_TH_ERRORS
3149: }
3150: 
3151: static PyObject* THPReflectionPad2DBackward0_self_getter(THPCppFunction *self, void *_unused) {
3152:   HANDLE_TH_ERRORS
3153:   const auto& prop = static_cast<ReflectionPad2DBackward0*>(self->cdata.get())->self_;
3154:   return THPVariable_Wrap(prop.unpack(self->cdata));
3155:   END_HANDLE_TH_ERRORS
3156: }
3157: 
3158: static PyObject* THPReflectionPad2DBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
3159:   HANDLE_TH_ERRORS
3160:   const auto& prop = static_cast<ReflectionPad2DBackward0*>(self->cdata.get())->self_;
3161:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3162:   return obj.release().ptr();
3163:   END_HANDLE_TH_ERRORS
3164: }
3165: 
3166: static struct PyGetSetDef ReflectionPad2DBackward0_properties[] = {
3167:   THP_FUNCTION_DEFAULT_PROPERTIES,
3168:   {(char*)"_saved_padding", (getter)THPReflectionPad2DBackward0_padding_getter, nullptr, nullptr, nullptr},
3169:   {(char*)"_saved_self", (getter)THPReflectionPad2DBackward0_self_getter, nullptr, nullptr, nullptr},
3170:   {(char*)"_raw_saved_self", (getter)THPReflectionPad2DBackward0_self_raw_getter, nullptr, nullptr, nullptr},
3171:   {nullptr} /* sentinel */
3172: };
3173: 
3174: static PyObject* THPUpsampleBilinear2DAaBackward0_align_corners_getter(THPCppFunction *self, void *_unused) {
3175:   HANDLE_TH_ERRORS
3176:   auto prop = static_cast<UpsampleBilinear2DAaBackward0*>(self->cdata.get())->align_corners;
3177:   if (prop) {
3178:     Py_RETURN_TRUE;
3179:   } else {
3180:     Py_RETURN_FALSE;
3181:   }
3182:   END_HANDLE_TH_ERRORS
3183: }
3184: 
3185: static PyObject* THPUpsampleBilinear2DAaBackward0_output_size_getter(THPCppFunction *self, void *_unused) {
3186:   HANDLE_TH_ERRORS
3187:   auto prop = static_cast<UpsampleBilinear2DAaBackward0*>(self->cdata.get())->output_size;
3188:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3189:   for (auto i : c10::irange(prop.size())) {
3190:       auto si = prop[i];
3191:       if (auto m = si.maybe_as_int()) {
3192:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3193:       } else {
3194:         auto py_symint = py::cast(si).release().ptr();
3195:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3196:       }
3197:   }
3198:   return tup;
3199:   END_HANDLE_TH_ERRORS
3200: }
```

- EN: The main execution path in this span is carried by `THPReflectionPad2DBackward0_padding_getter`, `PyTuple_New`, `PyTuple_SetItem`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPReflectionPad2DBackward0_padding_getter`, `PyTuple_New`, `PyTuple_SetItem` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3201-3280

```cpp
3201: 
3202: static PyObject* THPUpsampleBilinear2DAaBackward0_scales_h_getter(THPCppFunction *self, void *_unused) {
3203:   HANDLE_TH_ERRORS
3204:   auto opt_prop = static_cast<UpsampleBilinear2DAaBackward0*>(self->cdata.get())->scales_h;
3205:   if (!opt_prop.has_value()) {
3206:     Py_RETURN_NONE;
3207:   }
3208:   auto prop = opt_prop.value();
3209:   return PyFloat_FromDouble((double) prop);
3210:   END_HANDLE_TH_ERRORS
3211: }
3212: 
3213: static PyObject* THPUpsampleBilinear2DAaBackward0_scales_w_getter(THPCppFunction *self, void *_unused) {
3214:   HANDLE_TH_ERRORS
3215:   auto opt_prop = static_cast<UpsampleBilinear2DAaBackward0*>(self->cdata.get())->scales_w;
3216:   if (!opt_prop.has_value()) {
3217:     Py_RETURN_NONE;
3218:   }
3219:   auto prop = opt_prop.value();
3220:   return PyFloat_FromDouble((double) prop);
3221:   END_HANDLE_TH_ERRORS
3222: }
3223: 
3224: static PyObject* THPUpsampleBilinear2DAaBackward0_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
3225:   HANDLE_TH_ERRORS
3226:   auto prop = static_cast<UpsampleBilinear2DAaBackward0*>(self->cdata.get())->self_sym_sizes;
3227:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3228:   for (auto i : c10::irange(prop.size())) {
3229:       auto si = prop[i];
3230:       if (auto m = si.maybe_as_int()) {
3231:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3232:       } else {
3233:         auto py_symint = py::cast(si).release().ptr();
3234:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3235:       }
3236:   }
3237:   return tup;
3238:   END_HANDLE_TH_ERRORS
3239: }
3240: 
3241: static struct PyGetSetDef UpsampleBilinear2DAaBackward0_properties[] = {
3242:   THP_FUNCTION_DEFAULT_PROPERTIES,
3243:   {(char*)"_saved_align_corners", (getter)THPUpsampleBilinear2DAaBackward0_align_corners_getter, nullptr, nullptr, nullptr},
3244:   {(char*)"_saved_output_size", (getter)THPUpsampleBilinear2DAaBackward0_output_size_getter, nullptr, nullptr, nullptr},
3245:   {(char*)"_saved_scales_h", (getter)THPUpsampleBilinear2DAaBackward0_scales_h_getter, nullptr, nullptr, nullptr},
3246:   {(char*)"_saved_scales_w", (getter)THPUpsampleBilinear2DAaBackward0_scales_w_getter, nullptr, nullptr, nullptr},
3247:   {(char*)"_saved_self_sym_sizes", (getter)THPUpsampleBilinear2DAaBackward0_self_sym_sizes_getter, nullptr, nullptr, nullptr},
3248:   {nullptr} /* sentinel */
3249: };
3250: 
3251: static PyObject* THPChannelShuffleBackward0_groups_getter(THPCppFunction *self, void *_unused) {
3252:   HANDLE_TH_ERRORS
3253:   auto prop = static_cast<ChannelShuffleBackward0*>(self->cdata.get())->groups;
3254:   if (auto m = prop.maybe_as_int()) {
3255:     return PyLong_FromUnsignedLong(*m);
3256:   } else {
3257:     return py::cast(prop).release().ptr();
3258:   }
3259:   END_HANDLE_TH_ERRORS
3260: }
3261: 
3262: static struct PyGetSetDef ChannelShuffleBackward0_properties[] = {
3263:   THP_FUNCTION_DEFAULT_PROPERTIES,
3264:   {(char*)"_saved_groups", (getter)THPChannelShuffleBackward0_groups_getter, nullptr, nullptr, nullptr},
3265:   {nullptr} /* sentinel */
3266: };
3267: 
3268: static PyObject* THPAdaptiveAvgPool2DBackward0_self_getter(THPCppFunction *self, void *_unused) {
3269:   HANDLE_TH_ERRORS
3270:   const auto& prop = static_cast<AdaptiveAvgPool2DBackward0*>(self->cdata.get())->self_;
3271:   return THPVariable_Wrap(prop.unpack(self->cdata));
3272:   END_HANDLE_TH_ERRORS
3273: }
3274: 
3275: static PyObject* THPAdaptiveAvgPool2DBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
3276:   HANDLE_TH_ERRORS
3277:   const auto& prop = static_cast<AdaptiveAvgPool2DBackward0*>(self->cdata.get())->self_;
3278:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3279:   return obj.release().ptr();
3280:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPUpsampleBilinear2DAaBackward0_scales_h_getter`, `PyFloat_FromDouble`, `THPUpsampleBilinear2DAaBackward0_scales_w_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPUpsampleBilinear2DAaBackward0_scales_h_getter`, `PyFloat_FromDouble`, `THPUpsampleBilinear2DAaBackward0_scales_w_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3281-3360

```cpp
3281: }
3282: 
3283: static struct PyGetSetDef AdaptiveAvgPool2DBackward0_properties[] = {
3284:   THP_FUNCTION_DEFAULT_PROPERTIES,
3285:   {(char*)"_saved_self", (getter)THPAdaptiveAvgPool2DBackward0_self_getter, nullptr, nullptr, nullptr},
3286:   {(char*)"_raw_saved_self", (getter)THPAdaptiveAvgPool2DBackward0_self_raw_getter, nullptr, nullptr, nullptr},
3287:   {nullptr} /* sentinel */
3288: };
3289: 
3290: static PyObject* THPAvgPool3DBackward0_ceil_mode_getter(THPCppFunction *self, void *_unused) {
3291:   HANDLE_TH_ERRORS
3292:   auto prop = static_cast<AvgPool3DBackward0*>(self->cdata.get())->ceil_mode;
3293:   if (prop) {
3294:     Py_RETURN_TRUE;
3295:   } else {
3296:     Py_RETURN_FALSE;
3297:   }
3298:   END_HANDLE_TH_ERRORS
3299: }
3300: 
3301: static PyObject* THPAvgPool3DBackward0_count_include_pad_getter(THPCppFunction *self, void *_unused) {
3302:   HANDLE_TH_ERRORS
3303:   auto prop = static_cast<AvgPool3DBackward0*>(self->cdata.get())->count_include_pad;
3304:   if (prop) {
3305:     Py_RETURN_TRUE;
3306:   } else {
3307:     Py_RETURN_FALSE;
3308:   }
3309:   END_HANDLE_TH_ERRORS
3310: }
3311: 
3312: static PyObject* THPAvgPool3DBackward0_divisor_override_getter(THPCppFunction *self, void *_unused) {
3313:   HANDLE_TH_ERRORS
3314:   auto opt_prop = static_cast<AvgPool3DBackward0*>(self->cdata.get())->divisor_override;
3315:   if (!opt_prop.has_value()) {
3316:     Py_RETURN_NONE;
3317:   }
3318:   auto prop = opt_prop.value();
3319:   return PyLong_FromUnsignedLong((int64_t) prop);
3320:   END_HANDLE_TH_ERRORS
3321: }
3322: 
3323: static PyObject* THPAvgPool3DBackward0_kernel_size_getter(THPCppFunction *self, void *_unused) {
3324:   HANDLE_TH_ERRORS
3325:   auto prop = static_cast<AvgPool3DBackward0*>(self->cdata.get())->kernel_size;
3326:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3327:   for (auto i : c10::irange(prop.size())) {
3328:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
3329:   }
3330:   return tup;
3331:   END_HANDLE_TH_ERRORS
3332: }
3333: 
3334: static PyObject* THPAvgPool3DBackward0_padding_getter(THPCppFunction *self, void *_unused) {
3335:   HANDLE_TH_ERRORS
3336:   auto prop = static_cast<AvgPool3DBackward0*>(self->cdata.get())->padding;
3337:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3338:   for (auto i : c10::irange(prop.size())) {
3339:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
3340:   }
3341:   return tup;
3342:   END_HANDLE_TH_ERRORS
3343: }
3344: 
3345: static PyObject* THPAvgPool3DBackward0_self_getter(THPCppFunction *self, void *_unused) {
3346:   HANDLE_TH_ERRORS
3347:   const auto& prop = static_cast<AvgPool3DBackward0*>(self->cdata.get())->self_;
3348:   return THPVariable_Wrap(prop.unpack(self->cdata));
3349:   END_HANDLE_TH_ERRORS
3350: }
3351: 
3352: static PyObject* THPAvgPool3DBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
3353:   HANDLE_TH_ERRORS
3354:   const auto& prop = static_cast<AvgPool3DBackward0*>(self->cdata.get())->self_;
3355:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3356:   return obj.release().ptr();
3357:   END_HANDLE_TH_ERRORS
3358: }
3359: 
3360: static PyObject* THPAvgPool3DBackward0_stride_getter(THPCppFunction *self, void *_unused) {
```

- EN: The main execution path in this span is carried by `THPAvgPool3DBackward0_ceil_mode_getter`, `THPAvgPool3DBackward0_count_include_pad_getter`, `THPAvgPool3DBackward0_divisor_override_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPAvgPool3DBackward0_ceil_mode_getter`, `THPAvgPool3DBackward0_count_include_pad_getter`, `THPAvgPool3DBackward0_divisor_override_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 3361-3440

```cpp
3361:   HANDLE_TH_ERRORS
3362:   auto prop = static_cast<AvgPool3DBackward0*>(self->cdata.get())->stride;
3363:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3364:   for (auto i : c10::irange(prop.size())) {
3365:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
3366:   }
3367:   return tup;
3368:   END_HANDLE_TH_ERRORS
3369: }
3370: 
3371: static struct PyGetSetDef AvgPool3DBackward0_properties[] = {
3372:   THP_FUNCTION_DEFAULT_PROPERTIES,
3373:   {(char*)"_saved_ceil_mode", (getter)THPAvgPool3DBackward0_ceil_mode_getter, nullptr, nullptr, nullptr},
3374:   {(char*)"_saved_count_include_pad", (getter)THPAvgPool3DBackward0_count_include_pad_getter, nullptr, nullptr, nullptr},
3375:   {(char*)"_saved_divisor_override", (getter)THPAvgPool3DBackward0_divisor_override_getter, nullptr, nullptr, nullptr},
3376:   {(char*)"_saved_kernel_size", (getter)THPAvgPool3DBackward0_kernel_size_getter, nullptr, nullptr, nullptr},
3377:   {(char*)"_saved_padding", (getter)THPAvgPool3DBackward0_padding_getter, nullptr, nullptr, nullptr},
3378:   {(char*)"_saved_self", (getter)THPAvgPool3DBackward0_self_getter, nullptr, nullptr, nullptr},
3379:   {(char*)"_raw_saved_self", (getter)THPAvgPool3DBackward0_self_raw_getter, nullptr, nullptr, nullptr},
3380:   {(char*)"_saved_stride", (getter)THPAvgPool3DBackward0_stride_getter, nullptr, nullptr, nullptr},
3381:   {nullptr} /* sentinel */
3382: };
3383: 
3384: static PyObject* THPMaxPool3DWithIndicesBackward0_ceil_mode_getter(THPCppFunction *self, void *_unused) {
3385:   HANDLE_TH_ERRORS
3386:   auto prop = static_cast<MaxPool3DWithIndicesBackward0*>(self->cdata.get())->ceil_mode;
3387:   if (prop) {
3388:     Py_RETURN_TRUE;
3389:   } else {
3390:     Py_RETURN_FALSE;
3391:   }
3392:   END_HANDLE_TH_ERRORS
3393: }
3394: 
3395: static PyObject* THPMaxPool3DWithIndicesBackward0_dilation_getter(THPCppFunction *self, void *_unused) {
3396:   HANDLE_TH_ERRORS
3397:   auto prop = static_cast<MaxPool3DWithIndicesBackward0*>(self->cdata.get())->dilation;
3398:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3399:   for (auto i : c10::irange(prop.size())) {
3400:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
3401:   }
3402:   return tup;
3403:   END_HANDLE_TH_ERRORS
3404: }
3405: 
3406: static PyObject* THPMaxPool3DWithIndicesBackward0_kernel_size_getter(THPCppFunction *self, void *_unused) {
3407:   HANDLE_TH_ERRORS
3408:   auto prop = static_cast<MaxPool3DWithIndicesBackward0*>(self->cdata.get())->kernel_size;
3409:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3410:   for (auto i : c10::irange(prop.size())) {
3411:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
3412:   }
3413:   return tup;
3414:   END_HANDLE_TH_ERRORS
3415: }
3416: 
3417: static PyObject* THPMaxPool3DWithIndicesBackward0_padding_getter(THPCppFunction *self, void *_unused) {
3418:   HANDLE_TH_ERRORS
3419:   auto prop = static_cast<MaxPool3DWithIndicesBackward0*>(self->cdata.get())->padding;
3420:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3421:   for (auto i : c10::irange(prop.size())) {
3422:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
3423:   }
3424:   return tup;
3425:   END_HANDLE_TH_ERRORS
3426: }
3427: 
3428: static PyObject* THPMaxPool3DWithIndicesBackward0_self_getter(THPCppFunction *self, void *_unused) {
3429:   HANDLE_TH_ERRORS
3430:   const auto& prop = static_cast<MaxPool3DWithIndicesBackward0*>(self->cdata.get())->self_;
3431:   return THPVariable_Wrap(prop.unpack(self->cdata));
3432:   END_HANDLE_TH_ERRORS
3433: }
3434: 
3435: static PyObject* THPMaxPool3DWithIndicesBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
3436:   HANDLE_TH_ERRORS
3437:   const auto& prop = static_cast<MaxPool3DWithIndicesBackward0*>(self->cdata.get())->self_;
3438:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3439:   return obj.release().ptr();
3440:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `PyTuple_New`, `PyTuple_SetItem`, `THPMaxPool3DWithIndicesBackward0_ceil_mode_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `PyTuple_New`, `PyTuple_SetItem`, `THPMaxPool3DWithIndicesBackward0_ceil_mode_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 3441-3520

```cpp
3441: }
3442: 
3443: static PyObject* THPMaxPool3DWithIndicesBackward0_stride_getter(THPCppFunction *self, void *_unused) {
3444:   HANDLE_TH_ERRORS
3445:   auto prop = static_cast<MaxPool3DWithIndicesBackward0*>(self->cdata.get())->stride;
3446:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3447:   for (auto i : c10::irange(prop.size())) {
3448:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
3449:   }
3450:   return tup;
3451:   END_HANDLE_TH_ERRORS
3452: }
3453: 
3454: static PyObject* THPMaxPool3DWithIndicesBackward0_result1_getter(THPCppFunction *self, void *_unused) {
3455:   HANDLE_TH_ERRORS
3456:   const auto& prop = static_cast<MaxPool3DWithIndicesBackward0*>(self->cdata.get())->result1_;
3457:   return THPVariable_Wrap(prop.unpack(self->cdata));
3458:   END_HANDLE_TH_ERRORS
3459: }
3460: 
3461: static PyObject* THPMaxPool3DWithIndicesBackward0_result1_raw_getter(THPCppFunction *self, void *_unused) {
3462:   HANDLE_TH_ERRORS
3463:   const auto& prop = static_cast<MaxPool3DWithIndicesBackward0*>(self->cdata.get())->result1_;
3464:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3465:   return obj.release().ptr();
3466:   END_HANDLE_TH_ERRORS
3467: }
3468: 
3469: static struct PyGetSetDef MaxPool3DWithIndicesBackward0_properties[] = {
3470:   THP_FUNCTION_DEFAULT_PROPERTIES,
3471:   {(char*)"_saved_ceil_mode", (getter)THPMaxPool3DWithIndicesBackward0_ceil_mode_getter, nullptr, nullptr, nullptr},
3472:   {(char*)"_saved_dilation", (getter)THPMaxPool3DWithIndicesBackward0_dilation_getter, nullptr, nullptr, nullptr},
3473:   {(char*)"_saved_kernel_size", (getter)THPMaxPool3DWithIndicesBackward0_kernel_size_getter, nullptr, nullptr, nullptr},
3474:   {(char*)"_saved_padding", (getter)THPMaxPool3DWithIndicesBackward0_padding_getter, nullptr, nullptr, nullptr},
3475:   {(char*)"_saved_self", (getter)THPMaxPool3DWithIndicesBackward0_self_getter, nullptr, nullptr, nullptr},
3476:   {(char*)"_raw_saved_self", (getter)THPMaxPool3DWithIndicesBackward0_self_raw_getter, nullptr, nullptr, nullptr},
3477:   {(char*)"_saved_stride", (getter)THPMaxPool3DWithIndicesBackward0_stride_getter, nullptr, nullptr, nullptr},
3478:   {(char*)"_saved_result1", (getter)THPMaxPool3DWithIndicesBackward0_result1_getter, nullptr, nullptr, nullptr},
3479:   {(char*)"_raw_saved_result1", (getter)THPMaxPool3DWithIndicesBackward0_result1_raw_getter, nullptr, nullptr, nullptr},
3480:   {nullptr} /* sentinel */
3481: };
3482: 
3483: static PyObject* THPConvolutionBackwardBackward0_dilation_getter(THPCppFunction *self, void *_unused) {
3484:   HANDLE_TH_ERRORS
3485:   auto prop = static_cast<ConvolutionBackwardBackward0*>(self->cdata.get())->dilation;
3486:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3487:   for (auto i : c10::irange(prop.size())) {
3488:       auto si = prop[i];
3489:       if (auto m = si.maybe_as_int()) {
3490:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3491:       } else {
3492:         auto py_symint = py::cast(si).release().ptr();
3493:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3494:       }
3495:   }
3496:   return tup;
3497:   END_HANDLE_TH_ERRORS
3498: }
3499: 
3500: static PyObject* THPConvolutionBackwardBackward0_grad_output_getter(THPCppFunction *self, void *_unused) {
3501:   HANDLE_TH_ERRORS
3502:   const auto& prop = static_cast<ConvolutionBackwardBackward0*>(self->cdata.get())->grad_output_;
3503:   return THPVariable_Wrap(prop.unpack(self->cdata));
3504:   END_HANDLE_TH_ERRORS
3505: }
3506: 
3507: static PyObject* THPConvolutionBackwardBackward0_grad_output_raw_getter(THPCppFunction *self, void *_unused) {
3508:   HANDLE_TH_ERRORS
3509:   const auto& prop = static_cast<ConvolutionBackwardBackward0*>(self->cdata.get())->grad_output_;
3510:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3511:   return obj.release().ptr();
3512:   END_HANDLE_TH_ERRORS
3513: }
3514: 
3515: static PyObject* THPConvolutionBackwardBackward0_groups_getter(THPCppFunction *self, void *_unused) {
3516:   HANDLE_TH_ERRORS
3517:   auto prop = static_cast<ConvolutionBackwardBackward0*>(self->cdata.get())->groups;
3518:   if (auto m = prop.maybe_as_int()) {
3519:     return PyLong_FromUnsignedLong(*m);
3520:   } else {
```

- EN: The main execution path in this span is carried by `THPMaxPool3DWithIndicesBackward0_stride_getter`, `PyTuple_New`, `PyTuple_SetItem`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPMaxPool3DWithIndicesBackward0_stride_getter`, `PyTuple_New`, `PyTuple_SetItem` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 3521-3600

```cpp
3521:     return py::cast(prop).release().ptr();
3522:   }
3523:   END_HANDLE_TH_ERRORS
3524: }
3525: 
3526: static PyObject* THPConvolutionBackwardBackward0_input_getter(THPCppFunction *self, void *_unused) {
3527:   HANDLE_TH_ERRORS
3528:   const auto& prop = static_cast<ConvolutionBackwardBackward0*>(self->cdata.get())->input_;
3529:   return THPVariable_Wrap(prop.unpack(self->cdata));
3530:   END_HANDLE_TH_ERRORS
3531: }
3532: 
3533: static PyObject* THPConvolutionBackwardBackward0_input_raw_getter(THPCppFunction *self, void *_unused) {
3534:   HANDLE_TH_ERRORS
3535:   const auto& prop = static_cast<ConvolutionBackwardBackward0*>(self->cdata.get())->input_;
3536:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3537:   return obj.release().ptr();
3538:   END_HANDLE_TH_ERRORS
3539: }
3540: 
3541: static PyObject* THPConvolutionBackwardBackward0_output_padding_getter(THPCppFunction *self, void *_unused) {
3542:   HANDLE_TH_ERRORS
3543:   auto prop = static_cast<ConvolutionBackwardBackward0*>(self->cdata.get())->output_padding;
3544:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3545:   for (auto i : c10::irange(prop.size())) {
3546:       auto si = prop[i];
3547:       if (auto m = si.maybe_as_int()) {
3548:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3549:       } else {
3550:         auto py_symint = py::cast(si).release().ptr();
3551:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3552:       }
3553:   }
3554:   return tup;
3555:   END_HANDLE_TH_ERRORS
3556: }
3557: 
3558: static PyObject* THPConvolutionBackwardBackward0_padding_getter(THPCppFunction *self, void *_unused) {
3559:   HANDLE_TH_ERRORS
3560:   auto prop = static_cast<ConvolutionBackwardBackward0*>(self->cdata.get())->padding;
3561:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3562:   for (auto i : c10::irange(prop.size())) {
3563:       auto si = prop[i];
3564:       if (auto m = si.maybe_as_int()) {
3565:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3566:       } else {
3567:         auto py_symint = py::cast(si).release().ptr();
3568:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3569:       }
3570:   }
3571:   return tup;
3572:   END_HANDLE_TH_ERRORS
3573: }
3574: 
3575: static PyObject* THPConvolutionBackwardBackward0_stride_getter(THPCppFunction *self, void *_unused) {
3576:   HANDLE_TH_ERRORS
3577:   auto prop = static_cast<ConvolutionBackwardBackward0*>(self->cdata.get())->stride;
3578:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3579:   for (auto i : c10::irange(prop.size())) {
3580:       auto si = prop[i];
3581:       if (auto m = si.maybe_as_int()) {
3582:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3583:       } else {
3584:         auto py_symint = py::cast(si).release().ptr();
3585:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3586:       }
3587:   }
3588:   return tup;
3589:   END_HANDLE_TH_ERRORS
3590: }
3591: 
3592: static PyObject* THPConvolutionBackwardBackward0_transposed_getter(THPCppFunction *self, void *_unused) {
3593:   HANDLE_TH_ERRORS
3594:   auto prop = static_cast<ConvolutionBackwardBackward0*>(self->cdata.get())->transposed;
3595:   if (prop) {
3596:     Py_RETURN_TRUE;
3597:   } else {
3598:     Py_RETURN_FALSE;
3599:   }
3600:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `cast`, `THPConvolutionBackwardBackward0_input_getter`, `THPVariable_Wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `cast`, `THPConvolutionBackwardBackward0_input_getter`, `THPVariable_Wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3601-3680

```cpp
3601: }
3602: 
3603: static PyObject* THPConvolutionBackwardBackward0_weight_getter(THPCppFunction *self, void *_unused) {
3604:   HANDLE_TH_ERRORS
3605:   const auto& prop = static_cast<ConvolutionBackwardBackward0*>(self->cdata.get())->weight_;
3606:   return THPVariable_Wrap(prop.unpack(self->cdata));
3607:   END_HANDLE_TH_ERRORS
3608: }
3609: 
3610: static PyObject* THPConvolutionBackwardBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
3611:   HANDLE_TH_ERRORS
3612:   const auto& prop = static_cast<ConvolutionBackwardBackward0*>(self->cdata.get())->weight_;
3613:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3614:   return obj.release().ptr();
3615:   END_HANDLE_TH_ERRORS
3616: }
3617: 
3618: static struct PyGetSetDef ConvolutionBackwardBackward0_properties[] = {
3619:   THP_FUNCTION_DEFAULT_PROPERTIES,
3620:   {(char*)"_saved_dilation", (getter)THPConvolutionBackwardBackward0_dilation_getter, nullptr, nullptr, nullptr},
3621:   {(char*)"_saved_grad_output", (getter)THPConvolutionBackwardBackward0_grad_output_getter, nullptr, nullptr, nullptr},
3622:   {(char*)"_raw_saved_grad_output", (getter)THPConvolutionBackwardBackward0_grad_output_raw_getter, nullptr, nullptr, nullptr},
3623:   {(char*)"_saved_groups", (getter)THPConvolutionBackwardBackward0_groups_getter, nullptr, nullptr, nullptr},
3624:   {(char*)"_saved_input", (getter)THPConvolutionBackwardBackward0_input_getter, nullptr, nullptr, nullptr},
3625:   {(char*)"_raw_saved_input", (getter)THPConvolutionBackwardBackward0_input_raw_getter, nullptr, nullptr, nullptr},
3626:   {(char*)"_saved_output_padding", (getter)THPConvolutionBackwardBackward0_output_padding_getter, nullptr, nullptr, nullptr},
3627:   {(char*)"_saved_padding", (getter)THPConvolutionBackwardBackward0_padding_getter, nullptr, nullptr, nullptr},
3628:   {(char*)"_saved_stride", (getter)THPConvolutionBackwardBackward0_stride_getter, nullptr, nullptr, nullptr},
3629:   {(char*)"_saved_transposed", (getter)THPConvolutionBackwardBackward0_transposed_getter, nullptr, nullptr, nullptr},
3630:   {(char*)"_saved_weight", (getter)THPConvolutionBackwardBackward0_weight_getter, nullptr, nullptr, nullptr},
3631:   {(char*)"_raw_saved_weight", (getter)THPConvolutionBackwardBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
3632:   {nullptr} /* sentinel */
3633: };
3634: 
3635: static PyObject* THPConvolutionOverrideableBackward0_dilation_getter(THPCppFunction *self, void *_unused) {
3636:   HANDLE_TH_ERRORS
3637:   auto prop = static_cast<ConvolutionOverrideableBackward0*>(self->cdata.get())->dilation;
3638:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3639:   for (auto i : c10::irange(prop.size())) {
3640:       auto si = prop[i];
3641:       if (auto m = si.maybe_as_int()) {
3642:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3643:       } else {
3644:         auto py_symint = py::cast(si).release().ptr();
3645:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3646:       }
3647:   }
3648:   return tup;
3649:   END_HANDLE_TH_ERRORS
3650: }
3651: 
3652: static PyObject* THPConvolutionOverrideableBackward0_groups_getter(THPCppFunction *self, void *_unused) {
3653:   HANDLE_TH_ERRORS
3654:   auto prop = static_cast<ConvolutionOverrideableBackward0*>(self->cdata.get())->groups;
3655:   if (auto m = prop.maybe_as_int()) {
3656:     return PyLong_FromUnsignedLong(*m);
3657:   } else {
3658:     return py::cast(prop).release().ptr();
3659:   }
3660:   END_HANDLE_TH_ERRORS
3661: }
3662: 
3663: static PyObject* THPConvolutionOverrideableBackward0_input_getter(THPCppFunction *self, void *_unused) {
3664:   HANDLE_TH_ERRORS
3665:   const auto& prop = static_cast<ConvolutionOverrideableBackward0*>(self->cdata.get())->input_;
3666:   return THPVariable_Wrap(prop.unpack(self->cdata));
3667:   END_HANDLE_TH_ERRORS
3668: }
3669: 
3670: static PyObject* THPConvolutionOverrideableBackward0_input_raw_getter(THPCppFunction *self, void *_unused) {
3671:   HANDLE_TH_ERRORS
3672:   const auto& prop = static_cast<ConvolutionOverrideableBackward0*>(self->cdata.get())->input_;
3673:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3674:   return obj.release().ptr();
3675:   END_HANDLE_TH_ERRORS
3676: }
3677: 
3678: static PyObject* THPConvolutionOverrideableBackward0_output_padding_getter(THPCppFunction *self, void *_unused) {
3679:   HANDLE_TH_ERRORS
3680:   auto prop = static_cast<ConvolutionOverrideableBackward0*>(self->cdata.get())->output_padding;
```

- EN: The main execution path in this span is carried by `THPConvolutionBackwardBackward0_weight_getter`, `THPVariable_Wrap`, `THPConvolutionBackwardBackward0_weight_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPConvolutionBackwardBackward0_weight_getter`, `THPVariable_Wrap`, `THPConvolutionBackwardBackward0_weight_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3681-3760

```cpp
3681:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3682:   for (auto i : c10::irange(prop.size())) {
3683:       auto si = prop[i];
3684:       if (auto m = si.maybe_as_int()) {
3685:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3686:       } else {
3687:         auto py_symint = py::cast(si).release().ptr();
3688:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3689:       }
3690:   }
3691:   return tup;
3692:   END_HANDLE_TH_ERRORS
3693: }
3694: 
3695: static PyObject* THPConvolutionOverrideableBackward0_padding_getter(THPCppFunction *self, void *_unused) {
3696:   HANDLE_TH_ERRORS
3697:   auto prop = static_cast<ConvolutionOverrideableBackward0*>(self->cdata.get())->padding;
3698:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3699:   for (auto i : c10::irange(prop.size())) {
3700:       auto si = prop[i];
3701:       if (auto m = si.maybe_as_int()) {
3702:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3703:       } else {
3704:         auto py_symint = py::cast(si).release().ptr();
3705:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3706:       }
3707:   }
3708:   return tup;
3709:   END_HANDLE_TH_ERRORS
3710: }
3711: 
3712: static PyObject* THPConvolutionOverrideableBackward0_stride_getter(THPCppFunction *self, void *_unused) {
3713:   HANDLE_TH_ERRORS
3714:   auto prop = static_cast<ConvolutionOverrideableBackward0*>(self->cdata.get())->stride;
3715:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3716:   for (auto i : c10::irange(prop.size())) {
3717:       auto si = prop[i];
3718:       if (auto m = si.maybe_as_int()) {
3719:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3720:       } else {
3721:         auto py_symint = py::cast(si).release().ptr();
3722:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3723:       }
3724:   }
3725:   return tup;
3726:   END_HANDLE_TH_ERRORS
3727: }
3728: 
3729: static PyObject* THPConvolutionOverrideableBackward0_transposed_getter(THPCppFunction *self, void *_unused) {
3730:   HANDLE_TH_ERRORS
3731:   auto prop = static_cast<ConvolutionOverrideableBackward0*>(self->cdata.get())->transposed;
3732:   if (prop) {
3733:     Py_RETURN_TRUE;
3734:   } else {
3735:     Py_RETURN_FALSE;
3736:   }
3737:   END_HANDLE_TH_ERRORS
3738: }
3739: 
3740: static PyObject* THPConvolutionOverrideableBackward0_weight_getter(THPCppFunction *self, void *_unused) {
3741:   HANDLE_TH_ERRORS
3742:   const auto& prop = static_cast<ConvolutionOverrideableBackward0*>(self->cdata.get())->weight_;
3743:   return THPVariable_Wrap(prop.unpack(self->cdata));
3744:   END_HANDLE_TH_ERRORS
3745: }
3746: 
3747: static PyObject* THPConvolutionOverrideableBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
3748:   HANDLE_TH_ERRORS
3749:   const auto& prop = static_cast<ConvolutionOverrideableBackward0*>(self->cdata.get())->weight_;
3750:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3751:   return obj.release().ptr();
3752:   END_HANDLE_TH_ERRORS
3753: }
3754: 
3755: static struct PyGetSetDef ConvolutionOverrideableBackward0_properties[] = {
3756:   THP_FUNCTION_DEFAULT_PROPERTIES,
3757:   {(char*)"_saved_dilation", (getter)THPConvolutionOverrideableBackward0_dilation_getter, nullptr, nullptr, nullptr},
3758:   {(char*)"_saved_groups", (getter)THPConvolutionOverrideableBackward0_groups_getter, nullptr, nullptr, nullptr},
3759:   {(char*)"_saved_input", (getter)THPConvolutionOverrideableBackward0_input_getter, nullptr, nullptr, nullptr},
3760:   {(char*)"_raw_saved_input", (getter)THPConvolutionOverrideableBackward0_input_raw_getter, nullptr, nullptr, nullptr},
```

- EN: The main execution path in this span is carried by `PyTuple_New`, `PyTuple_SetItem`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyTuple_New`, `PyTuple_SetItem`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3761-3840

```cpp
3761:   {(char*)"_saved_output_padding", (getter)THPConvolutionOverrideableBackward0_output_padding_getter, nullptr, nullptr, nullptr},
3762:   {(char*)"_saved_padding", (getter)THPConvolutionOverrideableBackward0_padding_getter, nullptr, nullptr, nullptr},
3763:   {(char*)"_saved_stride", (getter)THPConvolutionOverrideableBackward0_stride_getter, nullptr, nullptr, nullptr},
3764:   {(char*)"_saved_transposed", (getter)THPConvolutionOverrideableBackward0_transposed_getter, nullptr, nullptr, nullptr},
3765:   {(char*)"_saved_weight", (getter)THPConvolutionOverrideableBackward0_weight_getter, nullptr, nullptr, nullptr},
3766:   {(char*)"_raw_saved_weight", (getter)THPConvolutionOverrideableBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
3767:   {nullptr} /* sentinel */
3768: };
3769: 
3770: static PyObject* THPSlowConvTranspose3DBackward0_bias_sym_sizes_opt_getter(THPCppFunction *self, void *_unused) {
3771:   HANDLE_TH_ERRORS
3772:   auto opt_prop = static_cast<SlowConvTranspose3DBackward0*>(self->cdata.get())->bias_sym_sizes_opt;
3773:   if (!opt_prop.list.has_value()) {
3774:     Py_RETURN_NONE;
3775:   }
3776:   auto prop = opt_prop.list.value();
3777:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3778:   for (auto i : c10::irange(prop.size())) {
3779:       auto si = prop[i];
3780:       if (auto m = si.maybe_as_int()) {
3781:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3782:       } else {
3783:         auto py_symint = py::cast(si).release().ptr();
3784:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3785:       }
3786:   }
3787:   return tup;
3788:   END_HANDLE_TH_ERRORS
3789: }
3790: 
3791: static PyObject* THPSlowConvTranspose3DBackward0_dilation_getter(THPCppFunction *self, void *_unused) {
3792:   HANDLE_TH_ERRORS
3793:   auto prop = static_cast<SlowConvTranspose3DBackward0*>(self->cdata.get())->dilation;
3794:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3795:   for (auto i : c10::irange(prop.size())) {
3796:       auto si = prop[i];
3797:       if (auto m = si.maybe_as_int()) {
3798:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3799:       } else {
3800:         auto py_symint = py::cast(si).release().ptr();
3801:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3802:       }
3803:   }
3804:   return tup;
3805:   END_HANDLE_TH_ERRORS
3806: }
3807: 
3808: static PyObject* THPSlowConvTranspose3DBackward0_output_padding_getter(THPCppFunction *self, void *_unused) {
3809:   HANDLE_TH_ERRORS
3810:   auto prop = static_cast<SlowConvTranspose3DBackward0*>(self->cdata.get())->output_padding;
3811:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3812:   for (auto i : c10::irange(prop.size())) {
3813:       auto si = prop[i];
3814:       if (auto m = si.maybe_as_int()) {
3815:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3816:       } else {
3817:         auto py_symint = py::cast(si).release().ptr();
3818:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3819:       }
3820:   }
3821:   return tup;
3822:   END_HANDLE_TH_ERRORS
3823: }
3824: 
3825: static PyObject* THPSlowConvTranspose3DBackward0_padding_getter(THPCppFunction *self, void *_unused) {
3826:   HANDLE_TH_ERRORS
3827:   auto prop = static_cast<SlowConvTranspose3DBackward0*>(self->cdata.get())->padding;
3828:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3829:   for (auto i : c10::irange(prop.size())) {
3830:       auto si = prop[i];
3831:       if (auto m = si.maybe_as_int()) {
3832:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3833:       } else {
3834:         auto py_symint = py::cast(si).release().ptr();
3835:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3836:       }
3837:   }
3838:   return tup;
3839:   END_HANDLE_TH_ERRORS
3840: }
```

- EN: The main execution path in this span is carried by `THPSlowConvTranspose3DBackward0_bias_sym_sizes_opt_getter`, `PyTuple_New`, `PyTuple_SetItem`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPSlowConvTranspose3DBackward0_bias_sym_sizes_opt_getter`, `PyTuple_New`, `PyTuple_SetItem` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3841-3920

```cpp
3841: 
3842: static PyObject* THPSlowConvTranspose3DBackward0_self_getter(THPCppFunction *self, void *_unused) {
3843:   HANDLE_TH_ERRORS
3844:   const auto& prop = static_cast<SlowConvTranspose3DBackward0*>(self->cdata.get())->self_;
3845:   return THPVariable_Wrap(prop.unpack(self->cdata));
3846:   END_HANDLE_TH_ERRORS
3847: }
3848: 
3849: static PyObject* THPSlowConvTranspose3DBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
3850:   HANDLE_TH_ERRORS
3851:   const auto& prop = static_cast<SlowConvTranspose3DBackward0*>(self->cdata.get())->self_;
3852:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3853:   return obj.release().ptr();
3854:   END_HANDLE_TH_ERRORS
3855: }
3856: 
3857: static PyObject* THPSlowConvTranspose3DBackward0_stride_getter(THPCppFunction *self, void *_unused) {
3858:   HANDLE_TH_ERRORS
3859:   auto prop = static_cast<SlowConvTranspose3DBackward0*>(self->cdata.get())->stride;
3860:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3861:   for (auto i : c10::irange(prop.size())) {
3862:       auto si = prop[i];
3863:       if (auto m = si.maybe_as_int()) {
3864:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3865:       } else {
3866:         auto py_symint = py::cast(si).release().ptr();
3867:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3868:       }
3869:   }
3870:   return tup;
3871:   END_HANDLE_TH_ERRORS
3872: }
3873: 
3874: static PyObject* THPSlowConvTranspose3DBackward0_weight_getter(THPCppFunction *self, void *_unused) {
3875:   HANDLE_TH_ERRORS
3876:   const auto& prop = static_cast<SlowConvTranspose3DBackward0*>(self->cdata.get())->weight_;
3877:   return THPVariable_Wrap(prop.unpack(self->cdata));
3878:   END_HANDLE_TH_ERRORS
3879: }
3880: 
3881: static PyObject* THPSlowConvTranspose3DBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
3882:   HANDLE_TH_ERRORS
3883:   const auto& prop = static_cast<SlowConvTranspose3DBackward0*>(self->cdata.get())->weight_;
3884:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3885:   return obj.release().ptr();
3886:   END_HANDLE_TH_ERRORS
3887: }
3888: 
3889: static struct PyGetSetDef SlowConvTranspose3DBackward0_properties[] = {
3890:   THP_FUNCTION_DEFAULT_PROPERTIES,
3891:   {(char*)"_saved_bias_sym_sizes_opt", (getter)THPSlowConvTranspose3DBackward0_bias_sym_sizes_opt_getter, nullptr, nullptr, nullptr},
3892:   {(char*)"_saved_dilation", (getter)THPSlowConvTranspose3DBackward0_dilation_getter, nullptr, nullptr, nullptr},
3893:   {(char*)"_saved_output_padding", (getter)THPSlowConvTranspose3DBackward0_output_padding_getter, nullptr, nullptr, nullptr},
3894:   {(char*)"_saved_padding", (getter)THPSlowConvTranspose3DBackward0_padding_getter, nullptr, nullptr, nullptr},
3895:   {(char*)"_saved_self", (getter)THPSlowConvTranspose3DBackward0_self_getter, nullptr, nullptr, nullptr},
3896:   {(char*)"_raw_saved_self", (getter)THPSlowConvTranspose3DBackward0_self_raw_getter, nullptr, nullptr, nullptr},
3897:   {(char*)"_saved_stride", (getter)THPSlowConvTranspose3DBackward0_stride_getter, nullptr, nullptr, nullptr},
3898:   {(char*)"_saved_weight", (getter)THPSlowConvTranspose3DBackward0_weight_getter, nullptr, nullptr, nullptr},
3899:   {(char*)"_raw_saved_weight", (getter)THPSlowConvTranspose3DBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
3900:   {nullptr} /* sentinel */
3901: };
3902: 
3903: static PyObject* THPSlowConv3DBackward0_bias_sym_sizes_opt_getter(THPCppFunction *self, void *_unused) {
3904:   HANDLE_TH_ERRORS
3905:   auto opt_prop = static_cast<SlowConv3DBackward0*>(self->cdata.get())->bias_sym_sizes_opt;
3906:   if (!opt_prop.list.has_value()) {
3907:     Py_RETURN_NONE;
3908:   }
3909:   auto prop = opt_prop.list.value();
3910:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3911:   for (auto i : c10::irange(prop.size())) {
3912:       auto si = prop[i];
3913:       if (auto m = si.maybe_as_int()) {
3914:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3915:       } else {
3916:         auto py_symint = py::cast(si).release().ptr();
3917:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3918:       }
3919:   }
3920:   return tup;
```

- EN: The main execution path in this span is carried by `THPSlowConvTranspose3DBackward0_self_getter`, `THPVariable_Wrap`, `THPSlowConvTranspose3DBackward0_self_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPSlowConvTranspose3DBackward0_self_getter`, `THPVariable_Wrap`, `THPSlowConvTranspose3DBackward0_self_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3921-4000

```cpp
3921:   END_HANDLE_TH_ERRORS
3922: }
3923: 
3924: static PyObject* THPSlowConv3DBackward0_padding_getter(THPCppFunction *self, void *_unused) {
3925:   HANDLE_TH_ERRORS
3926:   auto prop = static_cast<SlowConv3DBackward0*>(self->cdata.get())->padding;
3927:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3928:   for (auto i : c10::irange(prop.size())) {
3929:       auto si = prop[i];
3930:       if (auto m = si.maybe_as_int()) {
3931:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3932:       } else {
3933:         auto py_symint = py::cast(si).release().ptr();
3934:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3935:       }
3936:   }
3937:   return tup;
3938:   END_HANDLE_TH_ERRORS
3939: }
3940: 
3941: static PyObject* THPSlowConv3DBackward0_self_getter(THPCppFunction *self, void *_unused) {
3942:   HANDLE_TH_ERRORS
3943:   const auto& prop = static_cast<SlowConv3DBackward0*>(self->cdata.get())->self_;
3944:   return THPVariable_Wrap(prop.unpack(self->cdata));
3945:   END_HANDLE_TH_ERRORS
3946: }
3947: 
3948: static PyObject* THPSlowConv3DBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
3949:   HANDLE_TH_ERRORS
3950:   const auto& prop = static_cast<SlowConv3DBackward0*>(self->cdata.get())->self_;
3951:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3952:   return obj.release().ptr();
3953:   END_HANDLE_TH_ERRORS
3954: }
3955: 
3956: static PyObject* THPSlowConv3DBackward0_stride_getter(THPCppFunction *self, void *_unused) {
3957:   HANDLE_TH_ERRORS
3958:   auto prop = static_cast<SlowConv3DBackward0*>(self->cdata.get())->stride;
3959:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
3960:   for (auto i : c10::irange(prop.size())) {
3961:       auto si = prop[i];
3962:       if (auto m = si.maybe_as_int()) {
3963:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
3964:       } else {
3965:         auto py_symint = py::cast(si).release().ptr();
3966:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
3967:       }
3968:   }
3969:   return tup;
3970:   END_HANDLE_TH_ERRORS
3971: }
3972: 
3973: static PyObject* THPSlowConv3DBackward0_weight_getter(THPCppFunction *self, void *_unused) {
3974:   HANDLE_TH_ERRORS
3975:   const auto& prop = static_cast<SlowConv3DBackward0*>(self->cdata.get())->weight_;
3976:   return THPVariable_Wrap(prop.unpack(self->cdata));
3977:   END_HANDLE_TH_ERRORS
3978: }
3979: 
3980: static PyObject* THPSlowConv3DBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
3981:   HANDLE_TH_ERRORS
3982:   const auto& prop = static_cast<SlowConv3DBackward0*>(self->cdata.get())->weight_;
3983:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
3984:   return obj.release().ptr();
3985:   END_HANDLE_TH_ERRORS
3986: }
3987: 
3988: static struct PyGetSetDef SlowConv3DBackward0_properties[] = {
3989:   THP_FUNCTION_DEFAULT_PROPERTIES,
3990:   {(char*)"_saved_bias_sym_sizes_opt", (getter)THPSlowConv3DBackward0_bias_sym_sizes_opt_getter, nullptr, nullptr, nullptr},
3991:   {(char*)"_saved_padding", (getter)THPSlowConv3DBackward0_padding_getter, nullptr, nullptr, nullptr},
3992:   {(char*)"_saved_self", (getter)THPSlowConv3DBackward0_self_getter, nullptr, nullptr, nullptr},
3993:   {(char*)"_raw_saved_self", (getter)THPSlowConv3DBackward0_self_raw_getter, nullptr, nullptr, nullptr},
3994:   {(char*)"_saved_stride", (getter)THPSlowConv3DBackward0_stride_getter, nullptr, nullptr, nullptr},
3995:   {(char*)"_saved_weight", (getter)THPSlowConv3DBackward0_weight_getter, nullptr, nullptr, nullptr},
3996:   {(char*)"_raw_saved_weight", (getter)THPSlowConv3DBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
3997:   {nullptr} /* sentinel */
3998: };
3999: 
4000: static PyObject* THPIm2ColBackward0_dilation_getter(THPCppFunction *self, void *_unused) {
```

- EN: The main execution path in this span is carried by `THPSlowConv3DBackward0_padding_getter`, `PyTuple_New`, `PyTuple_SetItem`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPSlowConv3DBackward0_padding_getter`, `PyTuple_New`, `PyTuple_SetItem` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4001-4080

```cpp
4001:   HANDLE_TH_ERRORS
4002:   auto prop = static_cast<Im2ColBackward0*>(self->cdata.get())->dilation;
4003:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4004:   for (auto i : c10::irange(prop.size())) {
4005:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
4006:   }
4007:   return tup;
4008:   END_HANDLE_TH_ERRORS
4009: }
4010: 
4011: static PyObject* THPIm2ColBackward0_kernel_size_getter(THPCppFunction *self, void *_unused) {
4012:   HANDLE_TH_ERRORS
4013:   auto prop = static_cast<Im2ColBackward0*>(self->cdata.get())->kernel_size;
4014:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4015:   for (auto i : c10::irange(prop.size())) {
4016:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
4017:   }
4018:   return tup;
4019:   END_HANDLE_TH_ERRORS
4020: }
4021: 
4022: static PyObject* THPIm2ColBackward0_padding_getter(THPCppFunction *self, void *_unused) {
4023:   HANDLE_TH_ERRORS
4024:   auto prop = static_cast<Im2ColBackward0*>(self->cdata.get())->padding;
4025:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4026:   for (auto i : c10::irange(prop.size())) {
4027:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
4028:   }
4029:   return tup;
4030:   END_HANDLE_TH_ERRORS
4031: }
4032: 
4033: static PyObject* THPIm2ColBackward0_self_sym_argsize_minus_1_getter(THPCppFunction *self, void *_unused) {
4034:   HANDLE_TH_ERRORS
4035:   auto prop = static_cast<Im2ColBackward0*>(self->cdata.get())->self_sym_argsize_minus_1;
4036:   if (auto m = prop.maybe_as_int()) {
4037:     return PyLong_FromUnsignedLong(*m);
4038:   } else {
4039:     return py::cast(prop).release().ptr();
4040:   }
4041:   END_HANDLE_TH_ERRORS
4042: }
4043: 
4044: static PyObject* THPIm2ColBackward0_self_sym_argsize_minus_2_getter(THPCppFunction *self, void *_unused) {
4045:   HANDLE_TH_ERRORS
4046:   auto prop = static_cast<Im2ColBackward0*>(self->cdata.get())->self_sym_argsize_minus_2;
4047:   if (auto m = prop.maybe_as_int()) {
4048:     return PyLong_FromUnsignedLong(*m);
4049:   } else {
4050:     return py::cast(prop).release().ptr();
4051:   }
4052:   END_HANDLE_TH_ERRORS
4053: }
4054: 
4055: static PyObject* THPIm2ColBackward0_stride_getter(THPCppFunction *self, void *_unused) {
4056:   HANDLE_TH_ERRORS
4057:   auto prop = static_cast<Im2ColBackward0*>(self->cdata.get())->stride;
4058:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4059:   for (auto i : c10::irange(prop.size())) {
4060:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
4061:   }
4062:   return tup;
4063:   END_HANDLE_TH_ERRORS
4064: }
4065: 
4066: static struct PyGetSetDef Im2ColBackward0_properties[] = {
4067:   THP_FUNCTION_DEFAULT_PROPERTIES,
4068:   {(char*)"_saved_dilation", (getter)THPIm2ColBackward0_dilation_getter, nullptr, nullptr, nullptr},
4069:   {(char*)"_saved_kernel_size", (getter)THPIm2ColBackward0_kernel_size_getter, nullptr, nullptr, nullptr},
4070:   {(char*)"_saved_padding", (getter)THPIm2ColBackward0_padding_getter, nullptr, nullptr, nullptr},
4071:   {(char*)"_saved_self_sym_argsize_minus_1", (getter)THPIm2ColBackward0_self_sym_argsize_minus_1_getter, nullptr, nullptr, nullptr},
4072:   {(char*)"_saved_self_sym_argsize_minus_2", (getter)THPIm2ColBackward0_self_sym_argsize_minus_2_getter, nullptr, nullptr, nullptr},
4073:   {(char*)"_saved_stride", (getter)THPIm2ColBackward0_stride_getter, nullptr, nullptr, nullptr},
4074:   {nullptr} /* sentinel */
4075: };
4076: 
4077: static PyObject* THPAvgPool3DBackwardBackward0_ceil_mode_getter(THPCppFunction *self, void *_unused) {
4078:   HANDLE_TH_ERRORS
4079:   auto prop = static_cast<AvgPool3DBackwardBackward0*>(self->cdata.get())->ceil_mode;
4080:   if (prop) {
```

- EN: The main execution path in this span is carried by `PyTuple_New`, `PyTuple_SetItem`, `THPIm2ColBackward0_kernel_size_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `PyTuple_New`, `PyTuple_SetItem`, `THPIm2ColBackward0_kernel_size_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 4081-4160

```cpp
4081:     Py_RETURN_TRUE;
4082:   } else {
4083:     Py_RETURN_FALSE;
4084:   }
4085:   END_HANDLE_TH_ERRORS
4086: }
4087: 
4088: static PyObject* THPAvgPool3DBackwardBackward0_count_include_pad_getter(THPCppFunction *self, void *_unused) {
4089:   HANDLE_TH_ERRORS
4090:   auto prop = static_cast<AvgPool3DBackwardBackward0*>(self->cdata.get())->count_include_pad;
4091:   if (prop) {
4092:     Py_RETURN_TRUE;
4093:   } else {
4094:     Py_RETURN_FALSE;
4095:   }
4096:   END_HANDLE_TH_ERRORS
4097: }
4098: 
4099: static PyObject* THPAvgPool3DBackwardBackward0_divisor_override_getter(THPCppFunction *self, void *_unused) {
4100:   HANDLE_TH_ERRORS
4101:   auto opt_prop = static_cast<AvgPool3DBackwardBackward0*>(self->cdata.get())->divisor_override;
4102:   if (!opt_prop.has_value()) {
4103:     Py_RETURN_NONE;
4104:   }
4105:   auto prop = opt_prop.value();
4106:   return PyLong_FromUnsignedLong((int64_t) prop);
4107:   END_HANDLE_TH_ERRORS
4108: }
4109: 
4110: static PyObject* THPAvgPool3DBackwardBackward0_kernel_size_getter(THPCppFunction *self, void *_unused) {
4111:   HANDLE_TH_ERRORS
4112:   auto prop = static_cast<AvgPool3DBackwardBackward0*>(self->cdata.get())->kernel_size;
4113:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4114:   for (auto i : c10::irange(prop.size())) {
4115:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
4116:   }
4117:   return tup;
4118:   END_HANDLE_TH_ERRORS
4119: }
4120: 
4121: static PyObject* THPAvgPool3DBackwardBackward0_padding_getter(THPCppFunction *self, void *_unused) {
4122:   HANDLE_TH_ERRORS
4123:   auto prop = static_cast<AvgPool3DBackwardBackward0*>(self->cdata.get())->padding;
4124:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4125:   for (auto i : c10::irange(prop.size())) {
4126:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
4127:   }
4128:   return tup;
4129:   END_HANDLE_TH_ERRORS
4130: }
4131: 
4132: static PyObject* THPAvgPool3DBackwardBackward0_stride_getter(THPCppFunction *self, void *_unused) {
4133:   HANDLE_TH_ERRORS
4134:   auto prop = static_cast<AvgPool3DBackwardBackward0*>(self->cdata.get())->stride;
4135:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4136:   for (auto i : c10::irange(prop.size())) {
4137:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
4138:   }
4139:   return tup;
4140:   END_HANDLE_TH_ERRORS
4141: }
4142: 
4143: static struct PyGetSetDef AvgPool3DBackwardBackward0_properties[] = {
4144:   THP_FUNCTION_DEFAULT_PROPERTIES,
4145:   {(char*)"_saved_ceil_mode", (getter)THPAvgPool3DBackwardBackward0_ceil_mode_getter, nullptr, nullptr, nullptr},
4146:   {(char*)"_saved_count_include_pad", (getter)THPAvgPool3DBackwardBackward0_count_include_pad_getter, nullptr, nullptr, nullptr},
4147:   {(char*)"_saved_divisor_override", (getter)THPAvgPool3DBackwardBackward0_divisor_override_getter, nullptr, nullptr, nullptr},
4148:   {(char*)"_saved_kernel_size", (getter)THPAvgPool3DBackwardBackward0_kernel_size_getter, nullptr, nullptr, nullptr},
4149:   {(char*)"_saved_padding", (getter)THPAvgPool3DBackwardBackward0_padding_getter, nullptr, nullptr, nullptr},
4150:   {(char*)"_saved_stride", (getter)THPAvgPool3DBackwardBackward0_stride_getter, nullptr, nullptr, nullptr},
4151:   {nullptr} /* sentinel */
4152: };
4153: 
4154: static PyObject* THPGluBackwardBackward0_dim_getter(THPCppFunction *self, void *_unused) {
4155:   HANDLE_TH_ERRORS
4156:   auto prop = static_cast<GluBackwardBackward0*>(self->cdata.get())->dim;
4157:   return PyLong_FromUnsignedLong((int64_t) prop);
4158:   END_HANDLE_TH_ERRORS
4159: }
4160: 
```

- EN: The main execution path in this span is carried by `THPAvgPool3DBackwardBackward0_count_include_pad_getter`, `THPAvgPool3DBackwardBackward0_divisor_override_getter`, `PyLong_FromUnsignedLong`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPAvgPool3DBackwardBackward0_count_include_pad_getter`, `THPAvgPool3DBackwardBackward0_divisor_override_getter`, `PyLong_FromUnsignedLong` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 4161-4240

```cpp
4161: static PyObject* THPGluBackwardBackward0_grad_output_getter(THPCppFunction *self, void *_unused) {
4162:   HANDLE_TH_ERRORS
4163:   const auto& prop = static_cast<GluBackwardBackward0*>(self->cdata.get())->grad_output_;
4164:   return THPVariable_Wrap(prop.unpack(self->cdata));
4165:   END_HANDLE_TH_ERRORS
4166: }
4167: 
4168: static PyObject* THPGluBackwardBackward0_grad_output_raw_getter(THPCppFunction *self, void *_unused) {
4169:   HANDLE_TH_ERRORS
4170:   const auto& prop = static_cast<GluBackwardBackward0*>(self->cdata.get())->grad_output_;
4171:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4172:   return obj.release().ptr();
4173:   END_HANDLE_TH_ERRORS
4174: }
4175: 
4176: static PyObject* THPGluBackwardBackward0_self_getter(THPCppFunction *self, void *_unused) {
4177:   HANDLE_TH_ERRORS
4178:   const auto& prop = static_cast<GluBackwardBackward0*>(self->cdata.get())->self_;
4179:   return THPVariable_Wrap(prop.unpack(self->cdata));
4180:   END_HANDLE_TH_ERRORS
4181: }
4182: 
4183: static PyObject* THPGluBackwardBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
4184:   HANDLE_TH_ERRORS
4185:   const auto& prop = static_cast<GluBackwardBackward0*>(self->cdata.get())->self_;
4186:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4187:   return obj.release().ptr();
4188:   END_HANDLE_TH_ERRORS
4189: }
4190: 
4191: static struct PyGetSetDef GluBackwardBackward0_properties[] = {
4192:   THP_FUNCTION_DEFAULT_PROPERTIES,
4193:   {(char*)"_saved_dim", (getter)THPGluBackwardBackward0_dim_getter, nullptr, nullptr, nullptr},
4194:   {(char*)"_saved_grad_output", (getter)THPGluBackwardBackward0_grad_output_getter, nullptr, nullptr, nullptr},
4195:   {(char*)"_raw_saved_grad_output", (getter)THPGluBackwardBackward0_grad_output_raw_getter, nullptr, nullptr, nullptr},
4196:   {(char*)"_saved_self", (getter)THPGluBackwardBackward0_self_getter, nullptr, nullptr, nullptr},
4197:   {(char*)"_raw_saved_self", (getter)THPGluBackwardBackward0_self_raw_getter, nullptr, nullptr, nullptr},
4198:   {nullptr} /* sentinel */
4199: };
4200: 
4201: static PyObject* THPHardtanhBackwardBackward0_max_val_getter(THPCppFunction *self, void *_unused) {
4202:   HANDLE_TH_ERRORS
4203:   auto prop = static_cast<HardtanhBackwardBackward0*>(self->cdata.get())->max_val;
4204:   if (prop.isComplex()) {
4205:     auto cprop = prop.to<c10::complex<double>>();
4206:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
4207:   } else if (prop.isFloatingPoint()) {
4208:     return PyFloat_FromDouble(prop.to<double>());
4209:   } else if (prop.isIntegral(/*includeBool=*/false)) {
4210:     return PyLong_FromLong(prop.to<int64_t>());
4211:   } else if (prop.isBoolean()) {
4212:     if (prop.to<bool>()) {
4213:       Py_RETURN_TRUE;
4214:     } else {
4215:       Py_RETURN_FALSE;
4216:     }
4217:   } else {
4218:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
4219:     return nullptr;
4220:   }
4221:   END_HANDLE_TH_ERRORS
4222: }
4223: 
4224: static PyObject* THPHardtanhBackwardBackward0_min_val_getter(THPCppFunction *self, void *_unused) {
4225:   HANDLE_TH_ERRORS
4226:   auto prop = static_cast<HardtanhBackwardBackward0*>(self->cdata.get())->min_val;
4227:   if (prop.isComplex()) {
4228:     auto cprop = prop.to<c10::complex<double>>();
4229:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
4230:   } else if (prop.isFloatingPoint()) {
4231:     return PyFloat_FromDouble(prop.to<double>());
4232:   } else if (prop.isIntegral(/*includeBool=*/false)) {
4233:     return PyLong_FromLong(prop.to<int64_t>());
4234:   } else if (prop.isBoolean()) {
4235:     if (prop.to<bool>()) {
4236:       Py_RETURN_TRUE;
4237:     } else {
4238:       Py_RETURN_FALSE;
4239:     }
4240:   } else {
```

- EN: The main execution path in this span is carried by `THPGluBackwardBackward0_grad_output_getter`, `THPVariable_Wrap`, `THPGluBackwardBackward0_grad_output_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPGluBackwardBackward0_grad_output_getter`, `THPVariable_Wrap`, `THPGluBackwardBackward0_grad_output_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4241-4320

```cpp
4241:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
4242:     return nullptr;
4243:   }
4244:   END_HANDLE_TH_ERRORS
4245: }
4246: 
4247: static PyObject* THPHardtanhBackwardBackward0_self_getter(THPCppFunction *self, void *_unused) {
4248:   HANDLE_TH_ERRORS
4249:   const auto& prop = static_cast<HardtanhBackwardBackward0*>(self->cdata.get())->self_;
4250:   return THPVariable_Wrap(prop.unpack(self->cdata));
4251:   END_HANDLE_TH_ERRORS
4252: }
4253: 
4254: static PyObject* THPHardtanhBackwardBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
4255:   HANDLE_TH_ERRORS
4256:   const auto& prop = static_cast<HardtanhBackwardBackward0*>(self->cdata.get())->self_;
4257:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4258:   return obj.release().ptr();
4259:   END_HANDLE_TH_ERRORS
4260: }
4261: 
4262: static struct PyGetSetDef HardtanhBackwardBackward0_properties[] = {
4263:   THP_FUNCTION_DEFAULT_PROPERTIES,
4264:   {(char*)"_saved_max_val", (getter)THPHardtanhBackwardBackward0_max_val_getter, nullptr, nullptr, nullptr},
4265:   {(char*)"_saved_min_val", (getter)THPHardtanhBackwardBackward0_min_val_getter, nullptr, nullptr, nullptr},
4266:   {(char*)"_saved_self", (getter)THPHardtanhBackwardBackward0_self_getter, nullptr, nullptr, nullptr},
4267:   {(char*)"_raw_saved_self", (getter)THPHardtanhBackwardBackward0_self_raw_getter, nullptr, nullptr, nullptr},
4268:   {nullptr} /* sentinel */
4269: };
4270: 
4271: static PyObject* THPLeakyReluBackwardBackward0_negative_slope_getter(THPCppFunction *self, void *_unused) {
4272:   HANDLE_TH_ERRORS
4273:   auto prop = static_cast<LeakyReluBackwardBackward0*>(self->cdata.get())->negative_slope;
4274:   if (prop.isComplex()) {
4275:     auto cprop = prop.to<c10::complex<double>>();
4276:     return PyComplex_FromDoubles(cprop.real(), cprop.imag());
4277:   } else if (prop.isFloatingPoint()) {
4278:     return PyFloat_FromDouble(prop.to<double>());
4279:   } else if (prop.isIntegral(/*includeBool=*/false)) {
4280:     return PyLong_FromLong(prop.to<int64_t>());
4281:   } else if (prop.isBoolean()) {
4282:     if (prop.to<bool>()) {
4283:       Py_RETURN_TRUE;
4284:     } else {
4285:       Py_RETURN_FALSE;
4286:     }
4287:   } else {
4288:     PyErr_SetString(PyExc_RuntimeError, "Unknown scalar type");
4289:     return nullptr;
4290:   }
4291:   END_HANDLE_TH_ERRORS
4292: }
4293: 
4294: static PyObject* THPLeakyReluBackwardBackward0_self_getter(THPCppFunction *self, void *_unused) {
4295:   HANDLE_TH_ERRORS
4296:   const auto& prop = static_cast<LeakyReluBackwardBackward0*>(self->cdata.get())->self_;
4297:   return THPVariable_Wrap(prop.unpack(self->cdata));
4298:   END_HANDLE_TH_ERRORS
4299: }
4300: 
4301: static PyObject* THPLeakyReluBackwardBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
4302:   HANDLE_TH_ERRORS
4303:   const auto& prop = static_cast<LeakyReluBackwardBackward0*>(self->cdata.get())->self_;
4304:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4305:   return obj.release().ptr();
4306:   END_HANDLE_TH_ERRORS
4307: }
4308: 
4309: static struct PyGetSetDef LeakyReluBackwardBackward0_properties[] = {
4310:   THP_FUNCTION_DEFAULT_PROPERTIES,
4311:   {(char*)"_saved_negative_slope", (getter)THPLeakyReluBackwardBackward0_negative_slope_getter, nullptr, nullptr, nullptr},
4312:   {(char*)"_saved_self", (getter)THPLeakyReluBackwardBackward0_self_getter, nullptr, nullptr, nullptr},
4313:   {(char*)"_raw_saved_self", (getter)THPLeakyReluBackwardBackward0_self_raw_getter, nullptr, nullptr, nullptr},
4314:   {nullptr} /* sentinel */
4315: };
4316: 
4317: 
4318: 
4319: static struct PyGetSetDef MaxPool2DBackwardBackward0_properties[] = {
4320:   THP_FUNCTION_DEFAULT_PROPERTIES,
```

- EN: The main execution path in this span is carried by `PyErr_SetString`, `THPHardtanhBackwardBackward0_self_getter`, `THPVariable_Wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyErr_SetString`, `THPHardtanhBackwardBackward0_self_getter`, `THPVariable_Wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4321-4400

```cpp
4321: 
4322:   {nullptr} /* sentinel */
4323: };
4324: 
4325: static PyObject* THPMaxPool2DWithIndicesBackwardBackward0_indices_getter(THPCppFunction *self, void *_unused) {
4326:   HANDLE_TH_ERRORS
4327:   const auto& prop = static_cast<MaxPool2DWithIndicesBackwardBackward0*>(self->cdata.get())->indices_;
4328:   return THPVariable_Wrap(prop.unpack(self->cdata));
4329:   END_HANDLE_TH_ERRORS
4330: }
4331: 
4332: static PyObject* THPMaxPool2DWithIndicesBackwardBackward0_indices_raw_getter(THPCppFunction *self, void *_unused) {
4333:   HANDLE_TH_ERRORS
4334:   const auto& prop = static_cast<MaxPool2DWithIndicesBackwardBackward0*>(self->cdata.get())->indices_;
4335:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4336:   return obj.release().ptr();
4337:   END_HANDLE_TH_ERRORS
4338: }
4339: 
4340: static struct PyGetSetDef MaxPool2DWithIndicesBackwardBackward0_properties[] = {
4341:   THP_FUNCTION_DEFAULT_PROPERTIES,
4342:   {(char*)"_saved_indices", (getter)THPMaxPool2DWithIndicesBackwardBackward0_indices_getter, nullptr, nullptr, nullptr},
4343:   {(char*)"_raw_saved_indices", (getter)THPMaxPool2DWithIndicesBackwardBackward0_indices_raw_getter, nullptr, nullptr, nullptr},
4344:   {nullptr} /* sentinel */
4345: };
4346: 
4347: static PyObject* THPUpsampleLinear1DBackwardBackward0_align_corners_getter(THPCppFunction *self, void *_unused) {
4348:   HANDLE_TH_ERRORS
4349:   auto prop = static_cast<UpsampleLinear1DBackwardBackward0*>(self->cdata.get())->align_corners;
4350:   if (prop) {
4351:     Py_RETURN_TRUE;
4352:   } else {
4353:     Py_RETURN_FALSE;
4354:   }
4355:   END_HANDLE_TH_ERRORS
4356: }
4357: 
4358: static PyObject* THPUpsampleLinear1DBackwardBackward0_output_size_getter(THPCppFunction *self, void *_unused) {
4359:   HANDLE_TH_ERRORS
4360:   auto prop = static_cast<UpsampleLinear1DBackwardBackward0*>(self->cdata.get())->output_size;
4361:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4362:   for (auto i : c10::irange(prop.size())) {
4363:       auto si = prop[i];
4364:       if (auto m = si.maybe_as_int()) {
4365:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4366:       } else {
4367:         auto py_symint = py::cast(si).release().ptr();
4368:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4369:       }
4370:   }
4371:   return tup;
4372:   END_HANDLE_TH_ERRORS
4373: }
4374: 
4375: static PyObject* THPUpsampleLinear1DBackwardBackward0_scales_getter(THPCppFunction *self, void *_unused) {
4376:   HANDLE_TH_ERRORS
4377:   auto opt_prop = static_cast<UpsampleLinear1DBackwardBackward0*>(self->cdata.get())->scales;
4378:   if (!opt_prop.has_value()) {
4379:     Py_RETURN_NONE;
4380:   }
4381:   auto prop = opt_prop.value();
4382:   return PyFloat_FromDouble((double) prop);
4383:   END_HANDLE_TH_ERRORS
4384: }
4385: 
4386: static struct PyGetSetDef UpsampleLinear1DBackwardBackward0_properties[] = {
4387:   THP_FUNCTION_DEFAULT_PROPERTIES,
4388:   {(char*)"_saved_align_corners", (getter)THPUpsampleLinear1DBackwardBackward0_align_corners_getter, nullptr, nullptr, nullptr},
4389:   {(char*)"_saved_output_size", (getter)THPUpsampleLinear1DBackwardBackward0_output_size_getter, nullptr, nullptr, nullptr},
4390:   {(char*)"_saved_scales", (getter)THPUpsampleLinear1DBackwardBackward0_scales_getter, nullptr, nullptr, nullptr},
4391:   {nullptr} /* sentinel */
4392: };
4393: 
4394: static PyObject* THPUpsampleNearest1DBackwardBackward0_output_size_getter(THPCppFunction *self, void *_unused) {
4395:   HANDLE_TH_ERRORS
4396:   auto prop = static_cast<UpsampleNearest1DBackwardBackward0*>(self->cdata.get())->output_size;
4397:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4398:   for (auto i : c10::irange(prop.size())) {
4399:       auto si = prop[i];
4400:       if (auto m = si.maybe_as_int()) {
```

- EN: The main execution path in this span is carried by `THPMaxPool2DWithIndicesBackwardBackward0_indices_getter`, `THPVariable_Wrap`, `THPMaxPool2DWithIndicesBackwardBackward0_indices_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPMaxPool2DWithIndicesBackwardBackward0_indices_getter`, `THPVariable_Wrap`, `THPMaxPool2DWithIndicesBackwardBackward0_indices_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
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
4411: static PyObject* THPUpsampleNearest1DBackwardBackward0_scales_getter(THPCppFunction *self, void *_unused) {
4412:   HANDLE_TH_ERRORS
4413:   auto opt_prop = static_cast<UpsampleNearest1DBackwardBackward0*>(self->cdata.get())->scales;
4414:   if (!opt_prop.has_value()) {
4415:     Py_RETURN_NONE;
4416:   }
4417:   auto prop = opt_prop.value();
4418:   return PyFloat_FromDouble((double) prop);
4419:   END_HANDLE_TH_ERRORS
4420: }
4421: 
4422: static struct PyGetSetDef UpsampleNearest1DBackwardBackward0_properties[] = {
4423:   THP_FUNCTION_DEFAULT_PROPERTIES,
4424:   {(char*)"_saved_output_size", (getter)THPUpsampleNearest1DBackwardBackward0_output_size_getter, nullptr, nullptr, nullptr},
4425:   {(char*)"_saved_scales", (getter)THPUpsampleNearest1DBackwardBackward0_scales_getter, nullptr, nullptr, nullptr},
4426:   {nullptr} /* sentinel */
4427: };
4428: 
4429: static PyObject* THPUpsampleNearest2DBackwardBackward0_output_size_getter(THPCppFunction *self, void *_unused) {
4430:   HANDLE_TH_ERRORS
4431:   auto prop = static_cast<UpsampleNearest2DBackwardBackward0*>(self->cdata.get())->output_size;
4432:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4433:   for (auto i : c10::irange(prop.size())) {
4434:       auto si = prop[i];
4435:       if (auto m = si.maybe_as_int()) {
4436:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4437:       } else {
4438:         auto py_symint = py::cast(si).release().ptr();
4439:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4440:       }
4441:   }
4442:   return tup;
4443:   END_HANDLE_TH_ERRORS
4444: }
4445: 
4446: static PyObject* THPUpsampleNearest2DBackwardBackward0_scales_h_getter(THPCppFunction *self, void *_unused) {
4447:   HANDLE_TH_ERRORS
4448:   auto opt_prop = static_cast<UpsampleNearest2DBackwardBackward0*>(self->cdata.get())->scales_h;
4449:   if (!opt_prop.has_value()) {
4450:     Py_RETURN_NONE;
4451:   }
4452:   auto prop = opt_prop.value();
4453:   return PyFloat_FromDouble((double) prop);
4454:   END_HANDLE_TH_ERRORS
4455: }
4456: 
4457: static PyObject* THPUpsampleNearest2DBackwardBackward0_scales_w_getter(THPCppFunction *self, void *_unused) {
4458:   HANDLE_TH_ERRORS
4459:   auto opt_prop = static_cast<UpsampleNearest2DBackwardBackward0*>(self->cdata.get())->scales_w;
4460:   if (!opt_prop.has_value()) {
4461:     Py_RETURN_NONE;
4462:   }
4463:   auto prop = opt_prop.value();
4464:   return PyFloat_FromDouble((double) prop);
4465:   END_HANDLE_TH_ERRORS
4466: }
4467: 
4468: static struct PyGetSetDef UpsampleNearest2DBackwardBackward0_properties[] = {
4469:   THP_FUNCTION_DEFAULT_PROPERTIES,
4470:   {(char*)"_saved_output_size", (getter)THPUpsampleNearest2DBackwardBackward0_output_size_getter, nullptr, nullptr, nullptr},
4471:   {(char*)"_saved_scales_h", (getter)THPUpsampleNearest2DBackwardBackward0_scales_h_getter, nullptr, nullptr, nullptr},
4472:   {(char*)"_saved_scales_w", (getter)THPUpsampleNearest2DBackwardBackward0_scales_w_getter, nullptr, nullptr, nullptr},
4473:   {nullptr} /* sentinel */
4474: };
4475: 
4476: static PyObject* THPMpsConvolutionTransposeBackward0_dilation_getter(THPCppFunction *self, void *_unused) {
4477:   HANDLE_TH_ERRORS
4478:   auto prop = static_cast<MpsConvolutionTransposeBackward0*>(self->cdata.get())->dilation;
4479:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4480:   for (auto i : c10::irange(prop.size())) {
```

- EN: The main execution path in this span is carried by `PyTuple_SetItem`, `cast`, `THPUpsampleNearest1DBackwardBackward0_scales_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyTuple_SetItem`, `cast`, `THPUpsampleNearest1DBackwardBackward0_scales_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4481-4560

```cpp
4481:       auto si = prop[i];
4482:       if (auto m = si.maybe_as_int()) {
4483:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4484:       } else {
4485:         auto py_symint = py::cast(si).release().ptr();
4486:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4487:       }
4488:   }
4489:   return tup;
4490:   END_HANDLE_TH_ERRORS
4491: }
4492: 
4493: static PyObject* THPMpsConvolutionTransposeBackward0_groups_getter(THPCppFunction *self, void *_unused) {
4494:   HANDLE_TH_ERRORS
4495:   auto prop = static_cast<MpsConvolutionTransposeBackward0*>(self->cdata.get())->groups;
4496:   if (auto m = prop.maybe_as_int()) {
4497:     return PyLong_FromUnsignedLong(*m);
4498:   } else {
4499:     return py::cast(prop).release().ptr();
4500:   }
4501:   END_HANDLE_TH_ERRORS
4502: }
4503: 
4504: static PyObject* THPMpsConvolutionTransposeBackward0_output_padding_getter(THPCppFunction *self, void *_unused) {
4505:   HANDLE_TH_ERRORS
4506:   auto prop = static_cast<MpsConvolutionTransposeBackward0*>(self->cdata.get())->output_padding;
4507:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4508:   for (auto i : c10::irange(prop.size())) {
4509:       auto si = prop[i];
4510:       if (auto m = si.maybe_as_int()) {
4511:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4512:       } else {
4513:         auto py_symint = py::cast(si).release().ptr();
4514:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4515:       }
4516:   }
4517:   return tup;
4518:   END_HANDLE_TH_ERRORS
4519: }
4520: 
4521: static PyObject* THPMpsConvolutionTransposeBackward0_padding_getter(THPCppFunction *self, void *_unused) {
4522:   HANDLE_TH_ERRORS
4523:   auto prop = static_cast<MpsConvolutionTransposeBackward0*>(self->cdata.get())->padding;
4524:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4525:   for (auto i : c10::irange(prop.size())) {
4526:       auto si = prop[i];
4527:       if (auto m = si.maybe_as_int()) {
4528:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4529:       } else {
4530:         auto py_symint = py::cast(si).release().ptr();
4531:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4532:       }
4533:   }
4534:   return tup;
4535:   END_HANDLE_TH_ERRORS
4536: }
4537: 
4538: static PyObject* THPMpsConvolutionTransposeBackward0_self_getter(THPCppFunction *self, void *_unused) {
4539:   HANDLE_TH_ERRORS
4540:   const auto& prop = static_cast<MpsConvolutionTransposeBackward0*>(self->cdata.get())->self_;
4541:   return THPVariable_Wrap(prop.unpack(self->cdata));
4542:   END_HANDLE_TH_ERRORS
4543: }
4544: 
4545: static PyObject* THPMpsConvolutionTransposeBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
4546:   HANDLE_TH_ERRORS
4547:   const auto& prop = static_cast<MpsConvolutionTransposeBackward0*>(self->cdata.get())->self_;
4548:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4549:   return obj.release().ptr();
4550:   END_HANDLE_TH_ERRORS
4551: }
4552: 
4553: static PyObject* THPMpsConvolutionTransposeBackward0_stride_getter(THPCppFunction *self, void *_unused) {
4554:   HANDLE_TH_ERRORS
4555:   auto prop = static_cast<MpsConvolutionTransposeBackward0*>(self->cdata.get())->stride;
4556:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4557:   for (auto i : c10::irange(prop.size())) {
4558:       auto si = prop[i];
4559:       if (auto m = si.maybe_as_int()) {
4560:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
```

- EN: The main execution path in this span is carried by `PyTuple_SetItem`, `cast`, `THPMpsConvolutionTransposeBackward0_groups_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyTuple_SetItem`, `cast`, `THPMpsConvolutionTransposeBackward0_groups_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4561-4640

```cpp
4561:       } else {
4562:         auto py_symint = py::cast(si).release().ptr();
4563:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4564:       }
4565:   }
4566:   return tup;
4567:   END_HANDLE_TH_ERRORS
4568: }
4569: 
4570: static PyObject* THPMpsConvolutionTransposeBackward0_weight_getter(THPCppFunction *self, void *_unused) {
4571:   HANDLE_TH_ERRORS
4572:   const auto& prop = static_cast<MpsConvolutionTransposeBackward0*>(self->cdata.get())->weight_;
4573:   return THPVariable_Wrap(prop.unpack(self->cdata));
4574:   END_HANDLE_TH_ERRORS
4575: }
4576: 
4577: static PyObject* THPMpsConvolutionTransposeBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
4578:   HANDLE_TH_ERRORS
4579:   const auto& prop = static_cast<MpsConvolutionTransposeBackward0*>(self->cdata.get())->weight_;
4580:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4581:   return obj.release().ptr();
4582:   END_HANDLE_TH_ERRORS
4583: }
4584: 
4585: static struct PyGetSetDef MpsConvolutionTransposeBackward0_properties[] = {
4586:   THP_FUNCTION_DEFAULT_PROPERTIES,
4587:   {(char*)"_saved_dilation", (getter)THPMpsConvolutionTransposeBackward0_dilation_getter, nullptr, nullptr, nullptr},
4588:   {(char*)"_saved_groups", (getter)THPMpsConvolutionTransposeBackward0_groups_getter, nullptr, nullptr, nullptr},
4589:   {(char*)"_saved_output_padding", (getter)THPMpsConvolutionTransposeBackward0_output_padding_getter, nullptr, nullptr, nullptr},
4590:   {(char*)"_saved_padding", (getter)THPMpsConvolutionTransposeBackward0_padding_getter, nullptr, nullptr, nullptr},
4591:   {(char*)"_saved_self", (getter)THPMpsConvolutionTransposeBackward0_self_getter, nullptr, nullptr, nullptr},
4592:   {(char*)"_raw_saved_self", (getter)THPMpsConvolutionTransposeBackward0_self_raw_getter, nullptr, nullptr, nullptr},
4593:   {(char*)"_saved_stride", (getter)THPMpsConvolutionTransposeBackward0_stride_getter, nullptr, nullptr, nullptr},
4594:   {(char*)"_saved_weight", (getter)THPMpsConvolutionTransposeBackward0_weight_getter, nullptr, nullptr, nullptr},
4595:   {(char*)"_raw_saved_weight", (getter)THPMpsConvolutionTransposeBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
4596:   {nullptr} /* sentinel */
4597: };
4598: 
4599: static PyObject* THPCudnnBatchNormBackward0_epsilon_getter(THPCppFunction *self, void *_unused) {
4600:   HANDLE_TH_ERRORS
4601:   auto prop = static_cast<CudnnBatchNormBackward0*>(self->cdata.get())->epsilon;
4602:   return PyFloat_FromDouble((double) prop);
4603:   END_HANDLE_TH_ERRORS
4604: }
4605: 
4606: static PyObject* THPCudnnBatchNormBackward0_input_getter(THPCppFunction *self, void *_unused) {
4607:   HANDLE_TH_ERRORS
4608:   const auto& prop = static_cast<CudnnBatchNormBackward0*>(self->cdata.get())->input_;
4609:   return THPVariable_Wrap(prop.unpack(self->cdata));
4610:   END_HANDLE_TH_ERRORS
4611: }
4612: 
4613: static PyObject* THPCudnnBatchNormBackward0_input_raw_getter(THPCppFunction *self, void *_unused) {
4614:   HANDLE_TH_ERRORS
4615:   const auto& prop = static_cast<CudnnBatchNormBackward0*>(self->cdata.get())->input_;
4616:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4617:   return obj.release().ptr();
4618:   END_HANDLE_TH_ERRORS
4619: }
4620: 
4621: static PyObject* THPCudnnBatchNormBackward0_running_mean_getter(THPCppFunction *self, void *_unused) {
4622:   HANDLE_TH_ERRORS
4623:   const auto& prop = static_cast<CudnnBatchNormBackward0*>(self->cdata.get())->running_mean_;
4624:   return THPVariable_Wrap(prop.unpack(self->cdata));
4625:   END_HANDLE_TH_ERRORS
4626: }
4627: 
4628: static PyObject* THPCudnnBatchNormBackward0_running_mean_raw_getter(THPCppFunction *self, void *_unused) {
4629:   HANDLE_TH_ERRORS
4630:   const auto& prop = static_cast<CudnnBatchNormBackward0*>(self->cdata.get())->running_mean_;
4631:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4632:   return obj.release().ptr();
4633:   END_HANDLE_TH_ERRORS
4634: }
4635: 
4636: static PyObject* THPCudnnBatchNormBackward0_running_var_getter(THPCppFunction *self, void *_unused) {
4637:   HANDLE_TH_ERRORS
4638:   const auto& prop = static_cast<CudnnBatchNormBackward0*>(self->cdata.get())->running_var_;
4639:   return THPVariable_Wrap(prop.unpack(self->cdata));
4640:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `cast`, `PyTuple_SetItem`, `THPMpsConvolutionTransposeBackward0_weight_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `cast`, `PyTuple_SetItem`, `THPMpsConvolutionTransposeBackward0_weight_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4641-4720

```cpp
4641: }
4642: 
4643: static PyObject* THPCudnnBatchNormBackward0_running_var_raw_getter(THPCppFunction *self, void *_unused) {
4644:   HANDLE_TH_ERRORS
4645:   const auto& prop = static_cast<CudnnBatchNormBackward0*>(self->cdata.get())->running_var_;
4646:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4647:   return obj.release().ptr();
4648:   END_HANDLE_TH_ERRORS
4649: }
4650: 
4651: static PyObject* THPCudnnBatchNormBackward0_training_getter(THPCppFunction *self, void *_unused) {
4652:   HANDLE_TH_ERRORS
4653:   auto prop = static_cast<CudnnBatchNormBackward0*>(self->cdata.get())->training;
4654:   if (prop) {
4655:     Py_RETURN_TRUE;
4656:   } else {
4657:     Py_RETURN_FALSE;
4658:   }
4659:   END_HANDLE_TH_ERRORS
4660: }
4661: 
4662: static PyObject* THPCudnnBatchNormBackward0_weight_getter(THPCppFunction *self, void *_unused) {
4663:   HANDLE_TH_ERRORS
4664:   const auto& prop = static_cast<CudnnBatchNormBackward0*>(self->cdata.get())->weight_;
4665:   return THPVariable_Wrap(prop.unpack(self->cdata));
4666:   END_HANDLE_TH_ERRORS
4667: }
4668: 
4669: static PyObject* THPCudnnBatchNormBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
4670:   HANDLE_TH_ERRORS
4671:   const auto& prop = static_cast<CudnnBatchNormBackward0*>(self->cdata.get())->weight_;
4672:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4673:   return obj.release().ptr();
4674:   END_HANDLE_TH_ERRORS
4675: }
4676: 
4677: static PyObject* THPCudnnBatchNormBackward0_result1_getter(THPCppFunction *self, void *_unused) {
4678:   HANDLE_TH_ERRORS
4679:   const auto& prop = static_cast<CudnnBatchNormBackward0*>(self->cdata.get())->result1_;
4680:   return THPVariable_Wrap(prop.unpack(self->cdata));
4681:   END_HANDLE_TH_ERRORS
4682: }
4683: 
4684: static PyObject* THPCudnnBatchNormBackward0_result1_raw_getter(THPCppFunction *self, void *_unused) {
4685:   HANDLE_TH_ERRORS
4686:   const auto& prop = static_cast<CudnnBatchNormBackward0*>(self->cdata.get())->result1_;
4687:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4688:   return obj.release().ptr();
4689:   END_HANDLE_TH_ERRORS
4690: }
4691: 
4692: static PyObject* THPCudnnBatchNormBackward0_result2_getter(THPCppFunction *self, void *_unused) {
4693:   HANDLE_TH_ERRORS
4694:   const auto& prop = static_cast<CudnnBatchNormBackward0*>(self->cdata.get())->result2_;
4695:   return THPVariable_Wrap(prop.unpack(self->cdata));
4696:   END_HANDLE_TH_ERRORS
4697: }
4698: 
4699: static PyObject* THPCudnnBatchNormBackward0_result2_raw_getter(THPCppFunction *self, void *_unused) {
4700:   HANDLE_TH_ERRORS
4701:   const auto& prop = static_cast<CudnnBatchNormBackward0*>(self->cdata.get())->result2_;
4702:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4703:   return obj.release().ptr();
4704:   END_HANDLE_TH_ERRORS
4705: }
4706: 
4707: static PyObject* THPCudnnBatchNormBackward0_result3_getter(THPCppFunction *self, void *_unused) {
4708:   HANDLE_TH_ERRORS
4709:   const auto& prop = static_cast<CudnnBatchNormBackward0*>(self->cdata.get())->result3_;
4710:   return THPVariable_Wrap(prop.unpack(self->cdata));
4711:   END_HANDLE_TH_ERRORS
4712: }
4713: 
4714: static PyObject* THPCudnnBatchNormBackward0_result3_raw_getter(THPCppFunction *self, void *_unused) {
4715:   HANDLE_TH_ERRORS
4716:   const auto& prop = static_cast<CudnnBatchNormBackward0*>(self->cdata.get())->result3_;
4717:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4718:   return obj.release().ptr();
4719:   END_HANDLE_TH_ERRORS
4720: }
```

- EN: The main execution path in this span is carried by `THPCudnnBatchNormBackward0_running_var_raw_getter`, `cast`, `THPCudnnBatchNormBackward0_training_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPCudnnBatchNormBackward0_running_var_raw_getter`, `cast`, `THPCudnnBatchNormBackward0_training_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4721-4800

```cpp
4721: 
4722: static struct PyGetSetDef CudnnBatchNormBackward0_properties[] = {
4723:   THP_FUNCTION_DEFAULT_PROPERTIES,
4724:   {(char*)"_saved_epsilon", (getter)THPCudnnBatchNormBackward0_epsilon_getter, nullptr, nullptr, nullptr},
4725:   {(char*)"_saved_input", (getter)THPCudnnBatchNormBackward0_input_getter, nullptr, nullptr, nullptr},
4726:   {(char*)"_raw_saved_input", (getter)THPCudnnBatchNormBackward0_input_raw_getter, nullptr, nullptr, nullptr},
4727:   {(char*)"_saved_running_mean", (getter)THPCudnnBatchNormBackward0_running_mean_getter, nullptr, nullptr, nullptr},
4728:   {(char*)"_raw_saved_running_mean", (getter)THPCudnnBatchNormBackward0_running_mean_raw_getter, nullptr, nullptr, nullptr},
4729:   {(char*)"_saved_running_var", (getter)THPCudnnBatchNormBackward0_running_var_getter, nullptr, nullptr, nullptr},
4730:   {(char*)"_raw_saved_running_var", (getter)THPCudnnBatchNormBackward0_running_var_raw_getter, nullptr, nullptr, nullptr},
4731:   {(char*)"_saved_training", (getter)THPCudnnBatchNormBackward0_training_getter, nullptr, nullptr, nullptr},
4732:   {(char*)"_saved_weight", (getter)THPCudnnBatchNormBackward0_weight_getter, nullptr, nullptr, nullptr},
4733:   {(char*)"_raw_saved_weight", (getter)THPCudnnBatchNormBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
4734:   {(char*)"_saved_result1", (getter)THPCudnnBatchNormBackward0_result1_getter, nullptr, nullptr, nullptr},
4735:   {(char*)"_raw_saved_result1", (getter)THPCudnnBatchNormBackward0_result1_raw_getter, nullptr, nullptr, nullptr},
4736:   {(char*)"_saved_result2", (getter)THPCudnnBatchNormBackward0_result2_getter, nullptr, nullptr, nullptr},
4737:   {(char*)"_raw_saved_result2", (getter)THPCudnnBatchNormBackward0_result2_raw_getter, nullptr, nullptr, nullptr},
4738:   {(char*)"_saved_result3", (getter)THPCudnnBatchNormBackward0_result3_getter, nullptr, nullptr, nullptr},
4739:   {(char*)"_raw_saved_result3", (getter)THPCudnnBatchNormBackward0_result3_raw_getter, nullptr, nullptr, nullptr},
4740:   {nullptr} /* sentinel */
4741: };
4742: 
4743: static PyObject* THPMiopenConvolutionTransposeBackward0_bias_sym_sizes_opt_getter(THPCppFunction *self, void *_unused) {
4744:   HANDLE_TH_ERRORS
4745:   auto opt_prop = static_cast<MiopenConvolutionTransposeBackward0*>(self->cdata.get())->bias_sym_sizes_opt;
4746:   if (!opt_prop.list.has_value()) {
4747:     Py_RETURN_NONE;
4748:   }
4749:   auto prop = opt_prop.list.value();
4750:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4751:   for (auto i : c10::irange(prop.size())) {
4752:       auto si = prop[i];
4753:       if (auto m = si.maybe_as_int()) {
4754:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4755:       } else {
4756:         auto py_symint = py::cast(si).release().ptr();
4757:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4758:       }
4759:   }
4760:   return tup;
4761:   END_HANDLE_TH_ERRORS
4762: }
4763: 
4764: static PyObject* THPMiopenConvolutionTransposeBackward0_dilation_getter(THPCppFunction *self, void *_unused) {
4765:   HANDLE_TH_ERRORS
4766:   auto prop = static_cast<MiopenConvolutionTransposeBackward0*>(self->cdata.get())->dilation;
4767:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4768:   for (auto i : c10::irange(prop.size())) {
4769:       auto si = prop[i];
4770:       if (auto m = si.maybe_as_int()) {
4771:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4772:       } else {
4773:         auto py_symint = py::cast(si).release().ptr();
4774:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4775:       }
4776:   }
4777:   return tup;
4778:   END_HANDLE_TH_ERRORS
4779: }
4780: 
4781: static PyObject* THPMiopenConvolutionTransposeBackward0_groups_getter(THPCppFunction *self, void *_unused) {
4782:   HANDLE_TH_ERRORS
4783:   auto prop = static_cast<MiopenConvolutionTransposeBackward0*>(self->cdata.get())->groups;
4784:   if (auto m = prop.maybe_as_int()) {
4785:     return PyLong_FromUnsignedLong(*m);
4786:   } else {
4787:     return py::cast(prop).release().ptr();
4788:   }
4789:   END_HANDLE_TH_ERRORS
4790: }
4791: 
4792: static PyObject* THPMiopenConvolutionTransposeBackward0_output_padding_getter(THPCppFunction *self, void *_unused) {
4793:   HANDLE_TH_ERRORS
4794:   auto prop = static_cast<MiopenConvolutionTransposeBackward0*>(self->cdata.get())->output_padding;
4795:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4796:   for (auto i : c10::irange(prop.size())) {
4797:       auto si = prop[i];
4798:       if (auto m = si.maybe_as_int()) {
4799:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4800:       } else {
```

- EN: The main execution path in this span is carried by `THPMiopenConvolutionTransposeBackward0_bias_sym_sizes_opt_getter`, `PyTuple_New`, `PyTuple_SetItem`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPMiopenConvolutionTransposeBackward0_bias_sym_sizes_opt_getter`, `PyTuple_New`, `PyTuple_SetItem` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4801-4880

```cpp
4801:         auto py_symint = py::cast(si).release().ptr();
4802:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4803:       }
4804:   }
4805:   return tup;
4806:   END_HANDLE_TH_ERRORS
4807: }
4808: 
4809: static PyObject* THPMiopenConvolutionTransposeBackward0_padding_getter(THPCppFunction *self, void *_unused) {
4810:   HANDLE_TH_ERRORS
4811:   auto prop = static_cast<MiopenConvolutionTransposeBackward0*>(self->cdata.get())->padding;
4812:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4813:   for (auto i : c10::irange(prop.size())) {
4814:       auto si = prop[i];
4815:       if (auto m = si.maybe_as_int()) {
4816:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4817:       } else {
4818:         auto py_symint = py::cast(si).release().ptr();
4819:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4820:       }
4821:   }
4822:   return tup;
4823:   END_HANDLE_TH_ERRORS
4824: }
4825: 
4826: static PyObject* THPMiopenConvolutionTransposeBackward0_self_getter(THPCppFunction *self, void *_unused) {
4827:   HANDLE_TH_ERRORS
4828:   const auto& prop = static_cast<MiopenConvolutionTransposeBackward0*>(self->cdata.get())->self_;
4829:   return THPVariable_Wrap(prop.unpack(self->cdata));
4830:   END_HANDLE_TH_ERRORS
4831: }
4832: 
4833: static PyObject* THPMiopenConvolutionTransposeBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
4834:   HANDLE_TH_ERRORS
4835:   const auto& prop = static_cast<MiopenConvolutionTransposeBackward0*>(self->cdata.get())->self_;
4836:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4837:   return obj.release().ptr();
4838:   END_HANDLE_TH_ERRORS
4839: }
4840: 
4841: static PyObject* THPMiopenConvolutionTransposeBackward0_stride_getter(THPCppFunction *self, void *_unused) {
4842:   HANDLE_TH_ERRORS
4843:   auto prop = static_cast<MiopenConvolutionTransposeBackward0*>(self->cdata.get())->stride;
4844:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
4845:   for (auto i : c10::irange(prop.size())) {
4846:       auto si = prop[i];
4847:       if (auto m = si.maybe_as_int()) {
4848:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
4849:       } else {
4850:         auto py_symint = py::cast(si).release().ptr();
4851:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
4852:       }
4853:   }
4854:   return tup;
4855:   END_HANDLE_TH_ERRORS
4856: }
4857: 
4858: static PyObject* THPMiopenConvolutionTransposeBackward0_weight_getter(THPCppFunction *self, void *_unused) {
4859:   HANDLE_TH_ERRORS
4860:   const auto& prop = static_cast<MiopenConvolutionTransposeBackward0*>(self->cdata.get())->weight_;
4861:   return THPVariable_Wrap(prop.unpack(self->cdata));
4862:   END_HANDLE_TH_ERRORS
4863: }
4864: 
4865: static PyObject* THPMiopenConvolutionTransposeBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
4866:   HANDLE_TH_ERRORS
4867:   const auto& prop = static_cast<MiopenConvolutionTransposeBackward0*>(self->cdata.get())->weight_;
4868:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4869:   return obj.release().ptr();
4870:   END_HANDLE_TH_ERRORS
4871: }
4872: 
4873: static struct PyGetSetDef MiopenConvolutionTransposeBackward0_properties[] = {
4874:   THP_FUNCTION_DEFAULT_PROPERTIES,
4875:   {(char*)"_saved_bias_sym_sizes_opt", (getter)THPMiopenConvolutionTransposeBackward0_bias_sym_sizes_opt_getter, nullptr, nullptr, nullptr},
4876:   {(char*)"_saved_dilation", (getter)THPMiopenConvolutionTransposeBackward0_dilation_getter, nullptr, nullptr, nullptr},
4877:   {(char*)"_saved_groups", (getter)THPMiopenConvolutionTransposeBackward0_groups_getter, nullptr, nullptr, nullptr},
4878:   {(char*)"_saved_output_padding", (getter)THPMiopenConvolutionTransposeBackward0_output_padding_getter, nullptr, nullptr, nullptr},
4879:   {(char*)"_saved_padding", (getter)THPMiopenConvolutionTransposeBackward0_padding_getter, nullptr, nullptr, nullptr},
4880:   {(char*)"_saved_self", (getter)THPMiopenConvolutionTransposeBackward0_self_getter, nullptr, nullptr, nullptr},
```

- EN: The main execution path in this span is carried by `cast`, `PyTuple_SetItem`, `THPMiopenConvolutionTransposeBackward0_padding_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `cast`, `PyTuple_SetItem`, `THPMiopenConvolutionTransposeBackward0_padding_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4881-4960

```cpp
4881:   {(char*)"_raw_saved_self", (getter)THPMiopenConvolutionTransposeBackward0_self_raw_getter, nullptr, nullptr, nullptr},
4882:   {(char*)"_saved_stride", (getter)THPMiopenConvolutionTransposeBackward0_stride_getter, nullptr, nullptr, nullptr},
4883:   {(char*)"_saved_weight", (getter)THPMiopenConvolutionTransposeBackward0_weight_getter, nullptr, nullptr, nullptr},
4884:   {(char*)"_raw_saved_weight", (getter)THPMiopenConvolutionTransposeBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
4885:   {nullptr} /* sentinel */
4886: };
4887: 
4888: static PyObject* THPMiopenBatchNormBackward0_epsilon_getter(THPCppFunction *self, void *_unused) {
4889:   HANDLE_TH_ERRORS
4890:   auto prop = static_cast<MiopenBatchNormBackward0*>(self->cdata.get())->epsilon;
4891:   return PyFloat_FromDouble((double) prop);
4892:   END_HANDLE_TH_ERRORS
4893: }
4894: 
4895: static PyObject* THPMiopenBatchNormBackward0_input_getter(THPCppFunction *self, void *_unused) {
4896:   HANDLE_TH_ERRORS
4897:   const auto& prop = static_cast<MiopenBatchNormBackward0*>(self->cdata.get())->input_;
4898:   return THPVariable_Wrap(prop.unpack(self->cdata));
4899:   END_HANDLE_TH_ERRORS
4900: }
4901: 
4902: static PyObject* THPMiopenBatchNormBackward0_input_raw_getter(THPCppFunction *self, void *_unused) {
4903:   HANDLE_TH_ERRORS
4904:   const auto& prop = static_cast<MiopenBatchNormBackward0*>(self->cdata.get())->input_;
4905:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4906:   return obj.release().ptr();
4907:   END_HANDLE_TH_ERRORS
4908: }
4909: 
4910: static PyObject* THPMiopenBatchNormBackward0_running_mean_getter(THPCppFunction *self, void *_unused) {
4911:   HANDLE_TH_ERRORS
4912:   const auto& prop = static_cast<MiopenBatchNormBackward0*>(self->cdata.get())->running_mean_;
4913:   return THPVariable_Wrap(prop.unpack(self->cdata));
4914:   END_HANDLE_TH_ERRORS
4915: }
4916: 
4917: static PyObject* THPMiopenBatchNormBackward0_running_mean_raw_getter(THPCppFunction *self, void *_unused) {
4918:   HANDLE_TH_ERRORS
4919:   const auto& prop = static_cast<MiopenBatchNormBackward0*>(self->cdata.get())->running_mean_;
4920:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4921:   return obj.release().ptr();
4922:   END_HANDLE_TH_ERRORS
4923: }
4924: 
4925: static PyObject* THPMiopenBatchNormBackward0_running_var_getter(THPCppFunction *self, void *_unused) {
4926:   HANDLE_TH_ERRORS
4927:   const auto& prop = static_cast<MiopenBatchNormBackward0*>(self->cdata.get())->running_var_;
4928:   return THPVariable_Wrap(prop.unpack(self->cdata));
4929:   END_HANDLE_TH_ERRORS
4930: }
4931: 
4932: static PyObject* THPMiopenBatchNormBackward0_running_var_raw_getter(THPCppFunction *self, void *_unused) {
4933:   HANDLE_TH_ERRORS
4934:   const auto& prop = static_cast<MiopenBatchNormBackward0*>(self->cdata.get())->running_var_;
4935:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4936:   return obj.release().ptr();
4937:   END_HANDLE_TH_ERRORS
4938: }
4939: 
4940: static PyObject* THPMiopenBatchNormBackward0_training_getter(THPCppFunction *self, void *_unused) {
4941:   HANDLE_TH_ERRORS
4942:   auto prop = static_cast<MiopenBatchNormBackward0*>(self->cdata.get())->training;
4943:   if (prop) {
4944:     Py_RETURN_TRUE;
4945:   } else {
4946:     Py_RETURN_FALSE;
4947:   }
4948:   END_HANDLE_TH_ERRORS
4949: }
4950: 
4951: static PyObject* THPMiopenBatchNormBackward0_weight_getter(THPCppFunction *self, void *_unused) {
4952:   HANDLE_TH_ERRORS
4953:   const auto& prop = static_cast<MiopenBatchNormBackward0*>(self->cdata.get())->weight_;
4954:   return THPVariable_Wrap(prop.unpack(self->cdata));
4955:   END_HANDLE_TH_ERRORS
4956: }
4957: 
4958: static PyObject* THPMiopenBatchNormBackward0_weight_raw_getter(THPCppFunction *self, void *_unused) {
4959:   HANDLE_TH_ERRORS
4960:   const auto& prop = static_cast<MiopenBatchNormBackward0*>(self->cdata.get())->weight_;
```

- EN: The main execution path in this span is carried by `THPMiopenBatchNormBackward0_epsilon_getter`, `PyFloat_FromDouble`, `THPMiopenBatchNormBackward0_input_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPMiopenBatchNormBackward0_epsilon_getter`, `PyFloat_FromDouble`, `THPMiopenBatchNormBackward0_input_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4961-5040

```cpp
4961:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4962:   return obj.release().ptr();
4963:   END_HANDLE_TH_ERRORS
4964: }
4965: 
4966: static PyObject* THPMiopenBatchNormBackward0_result1_getter(THPCppFunction *self, void *_unused) {
4967:   HANDLE_TH_ERRORS
4968:   const auto& prop = static_cast<MiopenBatchNormBackward0*>(self->cdata.get())->result1_;
4969:   return THPVariable_Wrap(prop.unpack(self->cdata));
4970:   END_HANDLE_TH_ERRORS
4971: }
4972: 
4973: static PyObject* THPMiopenBatchNormBackward0_result1_raw_getter(THPCppFunction *self, void *_unused) {
4974:   HANDLE_TH_ERRORS
4975:   const auto& prop = static_cast<MiopenBatchNormBackward0*>(self->cdata.get())->result1_;
4976:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4977:   return obj.release().ptr();
4978:   END_HANDLE_TH_ERRORS
4979: }
4980: 
4981: static PyObject* THPMiopenBatchNormBackward0_result2_getter(THPCppFunction *self, void *_unused) {
4982:   HANDLE_TH_ERRORS
4983:   const auto& prop = static_cast<MiopenBatchNormBackward0*>(self->cdata.get())->result2_;
4984:   return THPVariable_Wrap(prop.unpack(self->cdata));
4985:   END_HANDLE_TH_ERRORS
4986: }
4987: 
4988: static PyObject* THPMiopenBatchNormBackward0_result2_raw_getter(THPCppFunction *self, void *_unused) {
4989:   HANDLE_TH_ERRORS
4990:   const auto& prop = static_cast<MiopenBatchNormBackward0*>(self->cdata.get())->result2_;
4991:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
4992:   return obj.release().ptr();
4993:   END_HANDLE_TH_ERRORS
4994: }
4995: 
4996: static struct PyGetSetDef MiopenBatchNormBackward0_properties[] = {
4997:   THP_FUNCTION_DEFAULT_PROPERTIES,
4998:   {(char*)"_saved_epsilon", (getter)THPMiopenBatchNormBackward0_epsilon_getter, nullptr, nullptr, nullptr},
4999:   {(char*)"_saved_input", (getter)THPMiopenBatchNormBackward0_input_getter, nullptr, nullptr, nullptr},
5000:   {(char*)"_raw_saved_input", (getter)THPMiopenBatchNormBackward0_input_raw_getter, nullptr, nullptr, nullptr},
5001:   {(char*)"_saved_running_mean", (getter)THPMiopenBatchNormBackward0_running_mean_getter, nullptr, nullptr, nullptr},
5002:   {(char*)"_raw_saved_running_mean", (getter)THPMiopenBatchNormBackward0_running_mean_raw_getter, nullptr, nullptr, nullptr},
5003:   {(char*)"_saved_running_var", (getter)THPMiopenBatchNormBackward0_running_var_getter, nullptr, nullptr, nullptr},
5004:   {(char*)"_raw_saved_running_var", (getter)THPMiopenBatchNormBackward0_running_var_raw_getter, nullptr, nullptr, nullptr},
5005:   {(char*)"_saved_training", (getter)THPMiopenBatchNormBackward0_training_getter, nullptr, nullptr, nullptr},
5006:   {(char*)"_saved_weight", (getter)THPMiopenBatchNormBackward0_weight_getter, nullptr, nullptr, nullptr},
5007:   {(char*)"_raw_saved_weight", (getter)THPMiopenBatchNormBackward0_weight_raw_getter, nullptr, nullptr, nullptr},
5008:   {(char*)"_saved_result1", (getter)THPMiopenBatchNormBackward0_result1_getter, nullptr, nullptr, nullptr},
5009:   {(char*)"_raw_saved_result1", (getter)THPMiopenBatchNormBackward0_result1_raw_getter, nullptr, nullptr, nullptr},
5010:   {(char*)"_saved_result2", (getter)THPMiopenBatchNormBackward0_result2_getter, nullptr, nullptr, nullptr},
5011:   {(char*)"_raw_saved_result2", (getter)THPMiopenBatchNormBackward0_result2_raw_getter, nullptr, nullptr, nullptr},
5012:   {nullptr} /* sentinel */
5013: };
5014: 
5015: static PyObject* THPMkldnnAdaptiveAvgPool2DBackward0_self_getter(THPCppFunction *self, void *_unused) {
5016:   HANDLE_TH_ERRORS
5017:   const auto& prop = static_cast<MkldnnAdaptiveAvgPool2DBackward0*>(self->cdata.get())->self_;
5018:   return THPVariable_Wrap(prop.unpack(self->cdata));
5019:   END_HANDLE_TH_ERRORS
5020: }
5021: 
5022: static PyObject* THPMkldnnAdaptiveAvgPool2DBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
5023:   HANDLE_TH_ERRORS
5024:   const auto& prop = static_cast<MkldnnAdaptiveAvgPool2DBackward0*>(self->cdata.get())->self_;
5025:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5026:   return obj.release().ptr();
5027:   END_HANDLE_TH_ERRORS
5028: }
5029: 
5030: static struct PyGetSetDef MkldnnAdaptiveAvgPool2DBackward0_properties[] = {
5031:   THP_FUNCTION_DEFAULT_PROPERTIES,
5032:   {(char*)"_saved_self", (getter)THPMkldnnAdaptiveAvgPool2DBackward0_self_getter, nullptr, nullptr, nullptr},
5033:   {(char*)"_raw_saved_self", (getter)THPMkldnnAdaptiveAvgPool2DBackward0_self_raw_getter, nullptr, nullptr, nullptr},
5034:   {nullptr} /* sentinel */
5035: };
5036: 
5037: static PyObject* THPNestedTensorFromMaskBackward0_t_sym_sizes_getter(THPCppFunction *self, void *_unused) {
5038:   HANDLE_TH_ERRORS
5039:   auto prop = static_cast<NestedTensorFromMaskBackward0*>(self->cdata.get())->t_sym_sizes;
5040:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
```

- EN: The main execution path in this span is carried by `cast`, `THPMiopenBatchNormBackward0_result1_getter`, `THPVariable_Wrap`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `cast`, `THPMiopenBatchNormBackward0_result1_getter`, `THPVariable_Wrap` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5041-5120

```cpp
5041:   for (auto i : c10::irange(prop.size())) {
5042:       auto si = prop[i];
5043:       if (auto m = si.maybe_as_int()) {
5044:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
5045:       } else {
5046:         auto py_symint = py::cast(si).release().ptr();
5047:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
5048:       }
5049:   }
5050:   return tup;
5051:   END_HANDLE_TH_ERRORS
5052: }
5053: 
5054: static struct PyGetSetDef NestedTensorFromMaskBackward0_properties[] = {
5055:   THP_FUNCTION_DEFAULT_PROPERTIES,
5056:   {(char*)"_saved_t_sym_sizes", (getter)THPNestedTensorFromMaskBackward0_t_sym_sizes_getter, nullptr, nullptr, nullptr},
5057:   {nullptr} /* sentinel */
5058: };
5059: 
5060: static PyObject* THPNestedFromPaddedTensorBackward0_padded_sym_sizes_getter(THPCppFunction *self, void *_unused) {
5061:   HANDLE_TH_ERRORS
5062:   auto prop = static_cast<NestedFromPaddedTensorBackward0*>(self->cdata.get())->padded_sym_sizes;
5063:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5064:   for (auto i : c10::irange(prop.size())) {
5065:       auto si = prop[i];
5066:       if (auto m = si.maybe_as_int()) {
5067:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
5068:       } else {
5069:         auto py_symint = py::cast(si).release().ptr();
5070:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
5071:       }
5072:   }
5073:   return tup;
5074:   END_HANDLE_TH_ERRORS
5075: }
5076: 
5077: static struct PyGetSetDef NestedFromPaddedTensorBackward0_properties[] = {
5078:   THP_FUNCTION_DEFAULT_PROPERTIES,
5079:   {(char*)"_saved_padded_sym_sizes", (getter)THPNestedFromPaddedTensorBackward0_padded_sym_sizes_getter, nullptr, nullptr, nullptr},
5080:   {nullptr} /* sentinel */
5081: };
5082: 
5083: static PyObject* THPNestedGetValuesBackward0_self_getter(THPCppFunction *self, void *_unused) {
5084:   HANDLE_TH_ERRORS
5085:   const auto& prop = static_cast<NestedGetValuesBackward0*>(self->cdata.get())->self_;
5086:   return THPVariable_Wrap(prop.unpack(self->cdata));
5087:   END_HANDLE_TH_ERRORS
5088: }
5089: 
5090: static PyObject* THPNestedGetValuesBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
5091:   HANDLE_TH_ERRORS
5092:   const auto& prop = static_cast<NestedGetValuesBackward0*>(self->cdata.get())->self_;
5093:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5094:   return obj.release().ptr();
5095:   END_HANDLE_TH_ERRORS
5096: }
5097: 
5098: static struct PyGetSetDef NestedGetValuesBackward0_properties[] = {
5099:   THP_FUNCTION_DEFAULT_PROPERTIES,
5100:   {(char*)"_saved_self", (getter)THPNestedGetValuesBackward0_self_getter, nullptr, nullptr, nullptr},
5101:   {(char*)"_raw_saved_self", (getter)THPNestedGetValuesBackward0_self_raw_getter, nullptr, nullptr, nullptr},
5102:   {nullptr} /* sentinel */
5103: };
5104: 
5105: static PyObject* THPCudnnAttentionBackward0_attn_bias_getter(THPCppFunction *self, void *_unused) {
5106:   HANDLE_TH_ERRORS
5107:   const auto& prop = static_cast<CudnnAttentionBackward0*>(self->cdata.get())->attn_bias_;
5108:   return THPVariable_Wrap(prop.unpack(self->cdata));
5109:   END_HANDLE_TH_ERRORS
5110: }
5111: 
5112: static PyObject* THPCudnnAttentionBackward0_attn_bias_raw_getter(THPCppFunction *self, void *_unused) {
5113:   HANDLE_TH_ERRORS
5114:   const auto& prop = static_cast<CudnnAttentionBackward0*>(self->cdata.get())->attn_bias_;
5115:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5116:   return obj.release().ptr();
5117:   END_HANDLE_TH_ERRORS
5118: }
5119: 
5120: static PyObject* THPCudnnAttentionBackward0_cum_seq_k_getter(THPCppFunction *self, void *_unused) {
```

- EN: The main execution path in this span is carried by `PyTuple_SetItem`, `cast`, `THPNestedFromPaddedTensorBackward0_padded_sym_sizes_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyTuple_SetItem`, `cast`, `THPNestedFromPaddedTensorBackward0_padded_sym_sizes_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5121-5200

```cpp
5121:   HANDLE_TH_ERRORS
5122:   const auto& prop = static_cast<CudnnAttentionBackward0*>(self->cdata.get())->cum_seq_k_;
5123:   return THPVariable_Wrap(prop.unpack(self->cdata));
5124:   END_HANDLE_TH_ERRORS
5125: }
5126: 
5127: static PyObject* THPCudnnAttentionBackward0_cum_seq_k_raw_getter(THPCppFunction *self, void *_unused) {
5128:   HANDLE_TH_ERRORS
5129:   const auto& prop = static_cast<CudnnAttentionBackward0*>(self->cdata.get())->cum_seq_k_;
5130:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5131:   return obj.release().ptr();
5132:   END_HANDLE_TH_ERRORS
5133: }
5134: 
5135: static PyObject* THPCudnnAttentionBackward0_cum_seq_q_getter(THPCppFunction *self, void *_unused) {
5136:   HANDLE_TH_ERRORS
5137:   const auto& prop = static_cast<CudnnAttentionBackward0*>(self->cdata.get())->cum_seq_q_;
5138:   return THPVariable_Wrap(prop.unpack(self->cdata));
5139:   END_HANDLE_TH_ERRORS
5140: }
5141: 
5142: static PyObject* THPCudnnAttentionBackward0_cum_seq_q_raw_getter(THPCppFunction *self, void *_unused) {
5143:   HANDLE_TH_ERRORS
5144:   const auto& prop = static_cast<CudnnAttentionBackward0*>(self->cdata.get())->cum_seq_q_;
5145:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5146:   return obj.release().ptr();
5147:   END_HANDLE_TH_ERRORS
5148: }
5149: 
5150: static PyObject* THPCudnnAttentionBackward0_dropout_p_getter(THPCppFunction *self, void *_unused) {
5151:   HANDLE_TH_ERRORS
5152:   auto prop = static_cast<CudnnAttentionBackward0*>(self->cdata.get())->dropout_p;
5153:   return PyFloat_FromDouble((double) prop);
5154:   END_HANDLE_TH_ERRORS
5155: }
5156: 
5157: static PyObject* THPCudnnAttentionBackward0_is_causal_getter(THPCppFunction *self, void *_unused) {
5158:   HANDLE_TH_ERRORS
5159:   auto prop = static_cast<CudnnAttentionBackward0*>(self->cdata.get())->is_causal;
5160:   if (prop) {
5161:     Py_RETURN_TRUE;
5162:   } else {
5163:     Py_RETURN_FALSE;
5164:   }
5165:   END_HANDLE_TH_ERRORS
5166: }
5167: 
5168: static PyObject* THPCudnnAttentionBackward0_key_getter(THPCppFunction *self, void *_unused) {
5169:   HANDLE_TH_ERRORS
5170:   const auto& prop = static_cast<CudnnAttentionBackward0*>(self->cdata.get())->key_;
5171:   return THPVariable_Wrap(prop.unpack(self->cdata));
5172:   END_HANDLE_TH_ERRORS
5173: }
5174: 
5175: static PyObject* THPCudnnAttentionBackward0_key_raw_getter(THPCppFunction *self, void *_unused) {
5176:   HANDLE_TH_ERRORS
5177:   const auto& prop = static_cast<CudnnAttentionBackward0*>(self->cdata.get())->key_;
5178:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5179:   return obj.release().ptr();
5180:   END_HANDLE_TH_ERRORS
5181: }
5182: 
5183: static PyObject* THPCudnnAttentionBackward0_max_k_getter(THPCppFunction *self, void *_unused) {
5184:   HANDLE_TH_ERRORS
5185:   auto prop = static_cast<CudnnAttentionBackward0*>(self->cdata.get())->max_k;
5186:   if (auto m = prop.maybe_as_int()) {
5187:     return PyLong_FromUnsignedLong(*m);
5188:   } else {
5189:     return py::cast(prop).release().ptr();
5190:   }
5191:   END_HANDLE_TH_ERRORS
5192: }
5193: 
5194: static PyObject* THPCudnnAttentionBackward0_max_q_getter(THPCppFunction *self, void *_unused) {
5195:   HANDLE_TH_ERRORS
5196:   auto prop = static_cast<CudnnAttentionBackward0*>(self->cdata.get())->max_q;
5197:   if (auto m = prop.maybe_as_int()) {
5198:     return PyLong_FromUnsignedLong(*m);
5199:   } else {
5200:     return py::cast(prop).release().ptr();
```

- EN: The main execution path in this span is carried by `THPVariable_Wrap`, `THPCudnnAttentionBackward0_cum_seq_k_raw_getter`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_Wrap`, `THPCudnnAttentionBackward0_cum_seq_k_raw_getter`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5201-5280

```cpp
5201:   }
5202:   END_HANDLE_TH_ERRORS
5203: }
5204: 
5205: static PyObject* THPCudnnAttentionBackward0_query_getter(THPCppFunction *self, void *_unused) {
5206:   HANDLE_TH_ERRORS
5207:   const auto& prop = static_cast<CudnnAttentionBackward0*>(self->cdata.get())->query_;
5208:   return THPVariable_Wrap(prop.unpack(self->cdata));
5209:   END_HANDLE_TH_ERRORS
5210: }
5211: 
5212: static PyObject* THPCudnnAttentionBackward0_query_raw_getter(THPCppFunction *self, void *_unused) {
5213:   HANDLE_TH_ERRORS
5214:   const auto& prop = static_cast<CudnnAttentionBackward0*>(self->cdata.get())->query_;
5215:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5216:   return obj.release().ptr();
5217:   END_HANDLE_TH_ERRORS
5218: }
5219: 
5220: static PyObject* THPCudnnAttentionBackward0_scale_getter(THPCppFunction *self, void *_unused) {
5221:   HANDLE_TH_ERRORS
5222:   auto opt_prop = static_cast<CudnnAttentionBackward0*>(self->cdata.get())->scale;
5223:   if (!opt_prop.has_value()) {
5224:     Py_RETURN_NONE;
5225:   }
5226:   auto prop = opt_prop.value();
5227:   return PyFloat_FromDouble((double) prop);
5228:   END_HANDLE_TH_ERRORS
5229: }
5230: 
5231: static PyObject* THPCudnnAttentionBackward0_value_getter(THPCppFunction *self, void *_unused) {
5232:   HANDLE_TH_ERRORS
5233:   const auto& prop = static_cast<CudnnAttentionBackward0*>(self->cdata.get())->value_;
5234:   return THPVariable_Wrap(prop.unpack(self->cdata));
5235:   END_HANDLE_TH_ERRORS
5236: }
5237: 
5238: static PyObject* THPCudnnAttentionBackward0_value_raw_getter(THPCppFunction *self, void *_unused) {
5239:   HANDLE_TH_ERRORS
5240:   const auto& prop = static_cast<CudnnAttentionBackward0*>(self->cdata.get())->value_;
5241:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5242:   return obj.release().ptr();
5243:   END_HANDLE_TH_ERRORS
5244: }
5245: 
5246: static PyObject* THPCudnnAttentionBackward0_logsumexp_getter(THPCppFunction *self, void *_unused) {
5247:   HANDLE_TH_ERRORS
5248:   const auto& prop = static_cast<CudnnAttentionBackward0*>(self->cdata.get())->logsumexp_;
5249:   return THPVariable_Wrap(prop.unpack(self->cdata));
5250:   END_HANDLE_TH_ERRORS
5251: }
5252: 
5253: static PyObject* THPCudnnAttentionBackward0_logsumexp_raw_getter(THPCppFunction *self, void *_unused) {
5254:   HANDLE_TH_ERRORS
5255:   const auto& prop = static_cast<CudnnAttentionBackward0*>(self->cdata.get())->logsumexp_;
5256:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5257:   return obj.release().ptr();
5258:   END_HANDLE_TH_ERRORS
5259: }
5260: 
5261: static PyObject* THPCudnnAttentionBackward0_output_getter(THPCppFunction *self, void *_unused) {
5262:   HANDLE_TH_ERRORS
5263:   const auto& prop = static_cast<CudnnAttentionBackward0*>(self->cdata.get())->output_;
5264:   return THPVariable_Wrap(prop.unpack(self->cdata));
5265:   END_HANDLE_TH_ERRORS
5266: }
5267: 
5268: static PyObject* THPCudnnAttentionBackward0_output_raw_getter(THPCppFunction *self, void *_unused) {
5269:   HANDLE_TH_ERRORS
5270:   const auto& prop = static_cast<CudnnAttentionBackward0*>(self->cdata.get())->output_;
5271:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5272:   return obj.release().ptr();
5273:   END_HANDLE_TH_ERRORS
5274: }
5275: 
5276: static PyObject* THPCudnnAttentionBackward0_philox_offset_getter(THPCppFunction *self, void *_unused) {
5277:   HANDLE_TH_ERRORS
5278:   const auto& prop = static_cast<CudnnAttentionBackward0*>(self->cdata.get())->philox_offset_;
5279:   return THPVariable_Wrap(prop.unpack(self->cdata));
5280:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPCudnnAttentionBackward0_query_getter`, `THPVariable_Wrap`, `THPCudnnAttentionBackward0_query_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPCudnnAttentionBackward0_query_getter`, `THPVariable_Wrap`, `THPCudnnAttentionBackward0_query_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5281-5360

```cpp
5281: }
5282: 
5283: static PyObject* THPCudnnAttentionBackward0_philox_offset_raw_getter(THPCppFunction *self, void *_unused) {
5284:   HANDLE_TH_ERRORS
5285:   const auto& prop = static_cast<CudnnAttentionBackward0*>(self->cdata.get())->philox_offset_;
5286:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5287:   return obj.release().ptr();
5288:   END_HANDLE_TH_ERRORS
5289: }
5290: 
5291: static PyObject* THPCudnnAttentionBackward0_philox_seed_getter(THPCppFunction *self, void *_unused) {
5292:   HANDLE_TH_ERRORS
5293:   const auto& prop = static_cast<CudnnAttentionBackward0*>(self->cdata.get())->philox_seed_;
5294:   return THPVariable_Wrap(prop.unpack(self->cdata));
5295:   END_HANDLE_TH_ERRORS
5296: }
5297: 
5298: static PyObject* THPCudnnAttentionBackward0_philox_seed_raw_getter(THPCppFunction *self, void *_unused) {
5299:   HANDLE_TH_ERRORS
5300:   const auto& prop = static_cast<CudnnAttentionBackward0*>(self->cdata.get())->philox_seed_;
5301:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5302:   return obj.release().ptr();
5303:   END_HANDLE_TH_ERRORS
5304: }
5305: 
5306: static struct PyGetSetDef CudnnAttentionBackward0_properties[] = {
5307:   THP_FUNCTION_DEFAULT_PROPERTIES,
5308:   {(char*)"_saved_attn_bias", (getter)THPCudnnAttentionBackward0_attn_bias_getter, nullptr, nullptr, nullptr},
5309:   {(char*)"_raw_saved_attn_bias", (getter)THPCudnnAttentionBackward0_attn_bias_raw_getter, nullptr, nullptr, nullptr},
5310:   {(char*)"_saved_cum_seq_k", (getter)THPCudnnAttentionBackward0_cum_seq_k_getter, nullptr, nullptr, nullptr},
5311:   {(char*)"_raw_saved_cum_seq_k", (getter)THPCudnnAttentionBackward0_cum_seq_k_raw_getter, nullptr, nullptr, nullptr},
5312:   {(char*)"_saved_cum_seq_q", (getter)THPCudnnAttentionBackward0_cum_seq_q_getter, nullptr, nullptr, nullptr},
5313:   {(char*)"_raw_saved_cum_seq_q", (getter)THPCudnnAttentionBackward0_cum_seq_q_raw_getter, nullptr, nullptr, nullptr},
5314:   {(char*)"_saved_dropout_p", (getter)THPCudnnAttentionBackward0_dropout_p_getter, nullptr, nullptr, nullptr},
5315:   {(char*)"_saved_is_causal", (getter)THPCudnnAttentionBackward0_is_causal_getter, nullptr, nullptr, nullptr},
5316:   {(char*)"_saved_key", (getter)THPCudnnAttentionBackward0_key_getter, nullptr, nullptr, nullptr},
5317:   {(char*)"_raw_saved_key", (getter)THPCudnnAttentionBackward0_key_raw_getter, nullptr, nullptr, nullptr},
5318:   {(char*)"_saved_max_k", (getter)THPCudnnAttentionBackward0_max_k_getter, nullptr, nullptr, nullptr},
5319:   {(char*)"_saved_max_q", (getter)THPCudnnAttentionBackward0_max_q_getter, nullptr, nullptr, nullptr},
5320:   {(char*)"_saved_query", (getter)THPCudnnAttentionBackward0_query_getter, nullptr, nullptr, nullptr},
5321:   {(char*)"_raw_saved_query", (getter)THPCudnnAttentionBackward0_query_raw_getter, nullptr, nullptr, nullptr},
5322:   {(char*)"_saved_scale", (getter)THPCudnnAttentionBackward0_scale_getter, nullptr, nullptr, nullptr},
5323:   {(char*)"_saved_value", (getter)THPCudnnAttentionBackward0_value_getter, nullptr, nullptr, nullptr},
5324:   {(char*)"_raw_saved_value", (getter)THPCudnnAttentionBackward0_value_raw_getter, nullptr, nullptr, nullptr},
5325:   {(char*)"_saved_logsumexp", (getter)THPCudnnAttentionBackward0_logsumexp_getter, nullptr, nullptr, nullptr},
5326:   {(char*)"_raw_saved_logsumexp", (getter)THPCudnnAttentionBackward0_logsumexp_raw_getter, nullptr, nullptr, nullptr},
5327:   {(char*)"_saved_output", (getter)THPCudnnAttentionBackward0_output_getter, nullptr, nullptr, nullptr},
5328:   {(char*)"_raw_saved_output", (getter)THPCudnnAttentionBackward0_output_raw_getter, nullptr, nullptr, nullptr},
5329:   {(char*)"_saved_philox_offset", (getter)THPCudnnAttentionBackward0_philox_offset_getter, nullptr, nullptr, nullptr},
5330:   {(char*)"_raw_saved_philox_offset", (getter)THPCudnnAttentionBackward0_philox_offset_raw_getter, nullptr, nullptr, nullptr},
5331:   {(char*)"_saved_philox_seed", (getter)THPCudnnAttentionBackward0_philox_seed_getter, nullptr, nullptr, nullptr},
5332:   {(char*)"_raw_saved_philox_seed", (getter)THPCudnnAttentionBackward0_philox_seed_raw_getter, nullptr, nullptr, nullptr},
5333:   {nullptr} /* sentinel */
5334: };
5335: 
5336: static PyObject* THPScaledDotProductFusedAttentionOverrideableBackward0_attn_bias_getter(THPCppFunction *self, void *_unused) {
5337:   HANDLE_TH_ERRORS
5338:   const auto& prop = static_cast<ScaledDotProductFusedAttentionOverrideableBackward0*>(self->cdata.get())->attn_bias_;
5339:   return THPVariable_Wrap(prop.unpack(self->cdata));
5340:   END_HANDLE_TH_ERRORS
5341: }
5342: 
5343: static PyObject* THPScaledDotProductFusedAttentionOverrideableBackward0_attn_bias_raw_getter(THPCppFunction *self, void *_unused) {
5344:   HANDLE_TH_ERRORS
5345:   const auto& prop = static_cast<ScaledDotProductFusedAttentionOverrideableBackward0*>(self->cdata.get())->attn_bias_;
5346:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5347:   return obj.release().ptr();
5348:   END_HANDLE_TH_ERRORS
5349: }
5350: 
5351: static PyObject* THPScaledDotProductFusedAttentionOverrideableBackward0_dropout_p_getter(THPCppFunction *self, void *_unused) {
5352:   HANDLE_TH_ERRORS
5353:   auto prop = static_cast<ScaledDotProductFusedAttentionOverrideableBackward0*>(self->cdata.get())->dropout_p;
5354:   return PyFloat_FromDouble((double) prop);
5355:   END_HANDLE_TH_ERRORS
5356: }
5357: 
5358: static PyObject* THPScaledDotProductFusedAttentionOverrideableBackward0_is_causal_getter(THPCppFunction *self, void *_unused) {
5359:   HANDLE_TH_ERRORS
5360:   auto prop = static_cast<ScaledDotProductFusedAttentionOverrideableBackward0*>(self->cdata.get())->is_causal;
```

- EN: The main execution path in this span is carried by `THPCudnnAttentionBackward0_philox_offset_raw_getter`, `cast`, `THPCudnnAttentionBackward0_philox_seed_getter`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPCudnnAttentionBackward0_philox_offset_raw_getter`, `cast`, `THPCudnnAttentionBackward0_philox_seed_getter` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5361-5440

```cpp
5361:   if (prop) {
5362:     Py_RETURN_TRUE;
5363:   } else {
5364:     Py_RETURN_FALSE;
5365:   }
5366:   END_HANDLE_TH_ERRORS
5367: }
5368: 
5369: static PyObject* THPScaledDotProductFusedAttentionOverrideableBackward0_key_getter(THPCppFunction *self, void *_unused) {
5370:   HANDLE_TH_ERRORS
5371:   const auto& prop = static_cast<ScaledDotProductFusedAttentionOverrideableBackward0*>(self->cdata.get())->key_;
5372:   return THPVariable_Wrap(prop.unpack(self->cdata));
5373:   END_HANDLE_TH_ERRORS
5374: }
5375: 
5376: static PyObject* THPScaledDotProductFusedAttentionOverrideableBackward0_key_raw_getter(THPCppFunction *self, void *_unused) {
5377:   HANDLE_TH_ERRORS
5378:   const auto& prop = static_cast<ScaledDotProductFusedAttentionOverrideableBackward0*>(self->cdata.get())->key_;
5379:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5380:   return obj.release().ptr();
5381:   END_HANDLE_TH_ERRORS
5382: }
5383: 
5384: static PyObject* THPScaledDotProductFusedAttentionOverrideableBackward0_query_getter(THPCppFunction *self, void *_unused) {
5385:   HANDLE_TH_ERRORS
5386:   const auto& prop = static_cast<ScaledDotProductFusedAttentionOverrideableBackward0*>(self->cdata.get())->query_;
5387:   return THPVariable_Wrap(prop.unpack(self->cdata));
5388:   END_HANDLE_TH_ERRORS
5389: }
5390: 
5391: static PyObject* THPScaledDotProductFusedAttentionOverrideableBackward0_query_raw_getter(THPCppFunction *self, void *_unused) {
5392:   HANDLE_TH_ERRORS
5393:   const auto& prop = static_cast<ScaledDotProductFusedAttentionOverrideableBackward0*>(self->cdata.get())->query_;
5394:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5395:   return obj.release().ptr();
5396:   END_HANDLE_TH_ERRORS
5397: }
5398: 
5399: static PyObject* THPScaledDotProductFusedAttentionOverrideableBackward0_scale_getter(THPCppFunction *self, void *_unused) {
5400:   HANDLE_TH_ERRORS
5401:   auto opt_prop = static_cast<ScaledDotProductFusedAttentionOverrideableBackward0*>(self->cdata.get())->scale;
5402:   if (!opt_prop.has_value()) {
5403:     Py_RETURN_NONE;
5404:   }
5405:   auto prop = opt_prop.value();
5406:   return PyFloat_FromDouble((double) prop);
5407:   END_HANDLE_TH_ERRORS
5408: }
5409: 
5410: static PyObject* THPScaledDotProductFusedAttentionOverrideableBackward0_value_getter(THPCppFunction *self, void *_unused) {
5411:   HANDLE_TH_ERRORS
5412:   const auto& prop = static_cast<ScaledDotProductFusedAttentionOverrideableBackward0*>(self->cdata.get())->value_;
5413:   return THPVariable_Wrap(prop.unpack(self->cdata));
5414:   END_HANDLE_TH_ERRORS
5415: }
5416: 
5417: static PyObject* THPScaledDotProductFusedAttentionOverrideableBackward0_value_raw_getter(THPCppFunction *self, void *_unused) {
5418:   HANDLE_TH_ERRORS
5419:   const auto& prop = static_cast<ScaledDotProductFusedAttentionOverrideableBackward0*>(self->cdata.get())->value_;
5420:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5421:   return obj.release().ptr();
5422:   END_HANDLE_TH_ERRORS
5423: }
5424: 
5425: static PyObject* THPScaledDotProductFusedAttentionOverrideableBackward0_cum_seq_k_getter(THPCppFunction *self, void *_unused) {
5426:   HANDLE_TH_ERRORS
5427:   const auto& prop = static_cast<ScaledDotProductFusedAttentionOverrideableBackward0*>(self->cdata.get())->cum_seq_k_;
5428:   return THPVariable_Wrap(prop.unpack(self->cdata));
5429:   END_HANDLE_TH_ERRORS
5430: }
5431: 
5432: static PyObject* THPScaledDotProductFusedAttentionOverrideableBackward0_cum_seq_k_raw_getter(THPCppFunction *self, void *_unused) {
5433:   HANDLE_TH_ERRORS
5434:   const auto& prop = static_cast<ScaledDotProductFusedAttentionOverrideableBackward0*>(self->cdata.get())->cum_seq_k_;
5435:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5436:   return obj.release().ptr();
5437:   END_HANDLE_TH_ERRORS
5438: }
5439: 
5440: static PyObject* THPScaledDotProductFusedAttentionOverrideableBackward0_cum_seq_q_getter(THPCppFunction *self, void *_unused) {
```

- EN: The main execution path in this span is carried by `THPScaledDotProductFusedAttentionOverrideableBackward0_key_getter`, `THPVariable_Wrap`, `THPScaledDotProductFusedAttentionOverrideableBackward0_key_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPScaledDotProductFusedAttentionOverrideableBackward0_key_getter`, `THPVariable_Wrap`, `THPScaledDotProductFusedAttentionOverrideableBackward0_key_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5441-5520

```cpp
5441:   HANDLE_TH_ERRORS
5442:   const auto& prop = static_cast<ScaledDotProductFusedAttentionOverrideableBackward0*>(self->cdata.get())->cum_seq_q_;
5443:   return THPVariable_Wrap(prop.unpack(self->cdata));
5444:   END_HANDLE_TH_ERRORS
5445: }
5446: 
5447: static PyObject* THPScaledDotProductFusedAttentionOverrideableBackward0_cum_seq_q_raw_getter(THPCppFunction *self, void *_unused) {
5448:   HANDLE_TH_ERRORS
5449:   const auto& prop = static_cast<ScaledDotProductFusedAttentionOverrideableBackward0*>(self->cdata.get())->cum_seq_q_;
5450:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5451:   return obj.release().ptr();
5452:   END_HANDLE_TH_ERRORS
5453: }
5454: 
5455: static PyObject* THPScaledDotProductFusedAttentionOverrideableBackward0_logsumexp_getter(THPCppFunction *self, void *_unused) {
5456:   HANDLE_TH_ERRORS
5457:   const auto& prop = static_cast<ScaledDotProductFusedAttentionOverrideableBackward0*>(self->cdata.get())->logsumexp_;
5458:   return THPVariable_Wrap(prop.unpack(self->cdata));
5459:   END_HANDLE_TH_ERRORS
5460: }
5461: 
5462: static PyObject* THPScaledDotProductFusedAttentionOverrideableBackward0_logsumexp_raw_getter(THPCppFunction *self, void *_unused) {
5463:   HANDLE_TH_ERRORS
5464:   const auto& prop = static_cast<ScaledDotProductFusedAttentionOverrideableBackward0*>(self->cdata.get())->logsumexp_;
5465:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5466:   return obj.release().ptr();
5467:   END_HANDLE_TH_ERRORS
5468: }
5469: 
5470: static PyObject* THPScaledDotProductFusedAttentionOverrideableBackward0_max_k_getter(THPCppFunction *self, void *_unused) {
5471:   HANDLE_TH_ERRORS
5472:   auto prop = static_cast<ScaledDotProductFusedAttentionOverrideableBackward0*>(self->cdata.get())->max_k;
5473:   if (auto m = prop.maybe_as_int()) {
5474:     return PyLong_FromUnsignedLong(*m);
5475:   } else {
5476:     return py::cast(prop).release().ptr();
5477:   }
5478:   END_HANDLE_TH_ERRORS
5479: }
5480: 
5481: static PyObject* THPScaledDotProductFusedAttentionOverrideableBackward0_max_q_getter(THPCppFunction *self, void *_unused) {
5482:   HANDLE_TH_ERRORS
5483:   auto prop = static_cast<ScaledDotProductFusedAttentionOverrideableBackward0*>(self->cdata.get())->max_q;
5484:   if (auto m = prop.maybe_as_int()) {
5485:     return PyLong_FromUnsignedLong(*m);
5486:   } else {
5487:     return py::cast(prop).release().ptr();
5488:   }
5489:   END_HANDLE_TH_ERRORS
5490: }
5491: 
5492: static PyObject* THPScaledDotProductFusedAttentionOverrideableBackward0_output_getter(THPCppFunction *self, void *_unused) {
5493:   HANDLE_TH_ERRORS
5494:   const auto& prop = static_cast<ScaledDotProductFusedAttentionOverrideableBackward0*>(self->cdata.get())->output_;
5495:   return THPVariable_Wrap(prop.unpack(self->cdata));
5496:   END_HANDLE_TH_ERRORS
5497: }
5498: 
5499: static PyObject* THPScaledDotProductFusedAttentionOverrideableBackward0_output_raw_getter(THPCppFunction *self, void *_unused) {
5500:   HANDLE_TH_ERRORS
5501:   const auto& prop = static_cast<ScaledDotProductFusedAttentionOverrideableBackward0*>(self->cdata.get())->output_;
5502:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5503:   return obj.release().ptr();
5504:   END_HANDLE_TH_ERRORS
5505: }
5506: 
5507: static PyObject* THPScaledDotProductFusedAttentionOverrideableBackward0_philox_offset_getter(THPCppFunction *self, void *_unused) {
5508:   HANDLE_TH_ERRORS
5509:   const auto& prop = static_cast<ScaledDotProductFusedAttentionOverrideableBackward0*>(self->cdata.get())->philox_offset_;
5510:   return THPVariable_Wrap(prop.unpack(self->cdata));
5511:   END_HANDLE_TH_ERRORS
5512: }
5513: 
5514: static PyObject* THPScaledDotProductFusedAttentionOverrideableBackward0_philox_offset_raw_getter(THPCppFunction *self, void *_unused) {
5515:   HANDLE_TH_ERRORS
5516:   const auto& prop = static_cast<ScaledDotProductFusedAttentionOverrideableBackward0*>(self->cdata.get())->philox_offset_;
5517:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5518:   return obj.release().ptr();
5519:   END_HANDLE_TH_ERRORS
5520: }
```

- EN: The main execution path in this span is carried by `THPVariable_Wrap`, `THPScaledDotProductFusedAttentionOverrideableBackward0_cum_seq_q_raw_getter`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_Wrap`, `THPScaledDotProductFusedAttentionOverrideableBackward0_cum_seq_q_raw_getter`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5521-5600

```cpp
5521: 
5522: static PyObject* THPScaledDotProductFusedAttentionOverrideableBackward0_philox_seed_getter(THPCppFunction *self, void *_unused) {
5523:   HANDLE_TH_ERRORS
5524:   const auto& prop = static_cast<ScaledDotProductFusedAttentionOverrideableBackward0*>(self->cdata.get())->philox_seed_;
5525:   return THPVariable_Wrap(prop.unpack(self->cdata));
5526:   END_HANDLE_TH_ERRORS
5527: }
5528: 
5529: static PyObject* THPScaledDotProductFusedAttentionOverrideableBackward0_philox_seed_raw_getter(THPCppFunction *self, void *_unused) {
5530:   HANDLE_TH_ERRORS
5531:   const auto& prop = static_cast<ScaledDotProductFusedAttentionOverrideableBackward0*>(self->cdata.get())->philox_seed_;
5532:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5533:   return obj.release().ptr();
5534:   END_HANDLE_TH_ERRORS
5535: }
5536: 
5537: static struct PyGetSetDef ScaledDotProductFusedAttentionOverrideableBackward0_properties[] = {
5538:   THP_FUNCTION_DEFAULT_PROPERTIES,
5539:   {(char*)"_saved_attn_bias", (getter)THPScaledDotProductFusedAttentionOverrideableBackward0_attn_bias_getter, nullptr, nullptr, nullptr},
5540:   {(char*)"_raw_saved_attn_bias", (getter)THPScaledDotProductFusedAttentionOverrideableBackward0_attn_bias_raw_getter, nullptr, nullptr, nullptr},
5541:   {(char*)"_saved_dropout_p", (getter)THPScaledDotProductFusedAttentionOverrideableBackward0_dropout_p_getter, nullptr, nullptr, nullptr},
5542:   {(char*)"_saved_is_causal", (getter)THPScaledDotProductFusedAttentionOverrideableBackward0_is_causal_getter, nullptr, nullptr, nullptr},
5543:   {(char*)"_saved_key", (getter)THPScaledDotProductFusedAttentionOverrideableBackward0_key_getter, nullptr, nullptr, nullptr},
5544:   {(char*)"_raw_saved_key", (getter)THPScaledDotProductFusedAttentionOverrideableBackward0_key_raw_getter, nullptr, nullptr, nullptr},
5545:   {(char*)"_saved_query", (getter)THPScaledDotProductFusedAttentionOverrideableBackward0_query_getter, nullptr, nullptr, nullptr},
5546:   {(char*)"_raw_saved_query", (getter)THPScaledDotProductFusedAttentionOverrideableBackward0_query_raw_getter, nullptr, nullptr, nullptr},
5547:   {(char*)"_saved_scale", (getter)THPScaledDotProductFusedAttentionOverrideableBackward0_scale_getter, nullptr, nullptr, nullptr},
5548:   {(char*)"_saved_value", (getter)THPScaledDotProductFusedAttentionOverrideableBackward0_value_getter, nullptr, nullptr, nullptr},
5549:   {(char*)"_raw_saved_value", (getter)THPScaledDotProductFusedAttentionOverrideableBackward0_value_raw_getter, nullptr, nullptr, nullptr},
5550:   {(char*)"_saved_cum_seq_k", (getter)THPScaledDotProductFusedAttentionOverrideableBackward0_cum_seq_k_getter, nullptr, nullptr, nullptr},
5551:   {(char*)"_raw_saved_cum_seq_k", (getter)THPScaledDotProductFusedAttentionOverrideableBackward0_cum_seq_k_raw_getter, nullptr, nullptr, nullptr},
5552:   {(char*)"_saved_cum_seq_q", (getter)THPScaledDotProductFusedAttentionOverrideableBackward0_cum_seq_q_getter, nullptr, nullptr, nullptr},
5553:   {(char*)"_raw_saved_cum_seq_q", (getter)THPScaledDotProductFusedAttentionOverrideableBackward0_cum_seq_q_raw_getter, nullptr, nullptr, nullptr},
5554:   {(char*)"_saved_logsumexp", (getter)THPScaledDotProductFusedAttentionOverrideableBackward0_logsumexp_getter, nullptr, nullptr, nullptr},
5555:   {(char*)"_raw_saved_logsumexp", (getter)THPScaledDotProductFusedAttentionOverrideableBackward0_logsumexp_raw_getter, nullptr, nullptr, nullptr},
5556:   {(char*)"_saved_max_k", (getter)THPScaledDotProductFusedAttentionOverrideableBackward0_max_k_getter, nullptr, nullptr, nullptr},
5557:   {(char*)"_saved_max_q", (getter)THPScaledDotProductFusedAttentionOverrideableBackward0_max_q_getter, nullptr, nullptr, nullptr},
5558:   {(char*)"_saved_output", (getter)THPScaledDotProductFusedAttentionOverrideableBackward0_output_getter, nullptr, nullptr, nullptr},
5559:   {(char*)"_raw_saved_output", (getter)THPScaledDotProductFusedAttentionOverrideableBackward0_output_raw_getter, nullptr, nullptr, nullptr},
5560:   {(char*)"_saved_philox_offset", (getter)THPScaledDotProductFusedAttentionOverrideableBackward0_philox_offset_getter, nullptr, nullptr, nullptr},
5561:   {(char*)"_raw_saved_philox_offset", (getter)THPScaledDotProductFusedAttentionOverrideableBackward0_philox_offset_raw_getter, nullptr, nullptr, nullptr},
5562:   {(char*)"_saved_philox_seed", (getter)THPScaledDotProductFusedAttentionOverrideableBackward0_philox_seed_getter, nullptr, nullptr, nullptr},
5563:   {(char*)"_raw_saved_philox_seed", (getter)THPScaledDotProductFusedAttentionOverrideableBackward0_philox_seed_raw_getter, nullptr, nullptr, nullptr},
5564:   {nullptr} /* sentinel */
5565: };
5566: 
5567: static PyObject* THPFftC2RBackward0_dim_getter(THPCppFunction *self, void *_unused) {
5568:   HANDLE_TH_ERRORS
5569:   auto prop = static_cast<FftC2RBackward0*>(self->cdata.get())->dim;
5570:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5571:   for (auto i : c10::irange(prop.size())) {
5572:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
5573:   }
5574:   return tup;
5575:   END_HANDLE_TH_ERRORS
5576: }
5577: 
5578: static PyObject* THPFftC2RBackward0_normalization_getter(THPCppFunction *self, void *_unused) {
5579:   HANDLE_TH_ERRORS
5580:   auto prop = static_cast<FftC2RBackward0*>(self->cdata.get())->normalization;
5581:   return PyLong_FromUnsignedLong((int64_t) prop);
5582:   END_HANDLE_TH_ERRORS
5583: }
5584: 
5585: static struct PyGetSetDef FftC2RBackward0_properties[] = {
5586:   THP_FUNCTION_DEFAULT_PROPERTIES,
5587:   {(char*)"_saved_dim", (getter)THPFftC2RBackward0_dim_getter, nullptr, nullptr, nullptr},
5588:   {(char*)"_saved_normalization", (getter)THPFftC2RBackward0_normalization_getter, nullptr, nullptr, nullptr},
5589:   {nullptr} /* sentinel */
5590: };
5591: 
5592: 
5593: 
5594: static struct PyGetSetDef TestWarnInAutogradBackward0_properties[] = {
5595:   THP_FUNCTION_DEFAULT_PROPERTIES,
5596: 
5597:   {nullptr} /* sentinel */
5598: };
5599: 
5600: static PyObject* THPTestAutogradMultipleDispatchViewBackward0_self_getter(THPCppFunction *self, void *_unused) {
```

- EN: The main execution path in this span is carried by `THPScaledDotProductFusedAttentionOverrideableBackward0_philox_seed_getter`, `THPVariable_Wrap`, `THPScaledDotProductFusedAttentionOverrideableBackward0_philox_seed_raw_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPScaledDotProductFusedAttentionOverrideableBackward0_philox_seed_getter`, `THPVariable_Wrap`, `THPScaledDotProductFusedAttentionOverrideableBackward0_philox_seed_raw_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 5601-5680

```cpp
5601:   HANDLE_TH_ERRORS
5602:   const auto& prop = static_cast<TestAutogradMultipleDispatchViewBackward0*>(self->cdata.get())->self_;
5603:   return THPVariable_Wrap(prop.unpack(self->cdata));
5604:   END_HANDLE_TH_ERRORS
5605: }
5606: 
5607: static PyObject* THPTestAutogradMultipleDispatchViewBackward0_self_raw_getter(THPCppFunction *self, void *_unused) {
5608:   HANDLE_TH_ERRORS
5609:   const auto& prop = static_cast<TestAutogradMultipleDispatchViewBackward0*>(self->cdata.get())->self_;
5610:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5611:   return obj.release().ptr();
5612:   END_HANDLE_TH_ERRORS
5613: }
5614: 
5615: static struct PyGetSetDef TestAutogradMultipleDispatchViewBackward0_properties[] = {
5616:   THP_FUNCTION_DEFAULT_PROPERTIES,
5617:   {(char*)"_saved_self", (getter)THPTestAutogradMultipleDispatchViewBackward0_self_getter, nullptr, nullptr, nullptr},
5618:   {(char*)"_raw_saved_self", (getter)THPTestAutogradMultipleDispatchViewBackward0_self_raw_getter, nullptr, nullptr, nullptr},
5619:   {nullptr} /* sentinel */
5620: };
5621: 
5622: static PyObject* THPTestAutogradMultipleDispatchViewBackwardAutogradCUDA0_self_getter(THPCppFunction *self, void *_unused) {
5623:   HANDLE_TH_ERRORS
5624:   const auto& prop = static_cast<TestAutogradMultipleDispatchViewBackwardAutogradCUDA0*>(self->cdata.get())->self_;
5625:   return THPVariable_Wrap(prop.unpack(self->cdata));
5626:   END_HANDLE_TH_ERRORS
5627: }
5628: 
5629: static PyObject* THPTestAutogradMultipleDispatchViewBackwardAutogradCUDA0_self_raw_getter(THPCppFunction *self, void *_unused) {
5630:   HANDLE_TH_ERRORS
5631:   const auto& prop = static_cast<TestAutogradMultipleDispatchViewBackwardAutogradCUDA0*>(self->cdata.get())->self_;
5632:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5633:   return obj.release().ptr();
5634:   END_HANDLE_TH_ERRORS
5635: }
5636: 
5637: static struct PyGetSetDef TestAutogradMultipleDispatchViewBackwardAutogradCUDA0_properties[] = {
5638:   THP_FUNCTION_DEFAULT_PROPERTIES,
5639:   {(char*)"_saved_self", (getter)THPTestAutogradMultipleDispatchViewBackwardAutogradCUDA0_self_getter, nullptr, nullptr, nullptr},
5640:   {(char*)"_raw_saved_self", (getter)THPTestAutogradMultipleDispatchViewBackwardAutogradCUDA0_self_raw_getter, nullptr, nullptr, nullptr},
5641:   {nullptr} /* sentinel */
5642: };
5643: 
5644: static PyObject* THPReshapeCopyBackward0_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
5645:   HANDLE_TH_ERRORS
5646:   auto prop = static_cast<ReshapeCopyBackward0*>(self->cdata.get())->self_sym_sizes;
5647:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5648:   for (auto i : c10::irange(prop.size())) {
5649:       auto si = prop[i];
5650:       if (auto m = si.maybe_as_int()) {
5651:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
5652:       } else {
5653:         auto py_symint = py::cast(si).release().ptr();
5654:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
5655:       }
5656:   }
5657:   return tup;
5658:   END_HANDLE_TH_ERRORS
5659: }
5660: 
5661: static struct PyGetSetDef ReshapeCopyBackward0_properties[] = {
5662:   THP_FUNCTION_DEFAULT_PROPERTIES,
5663:   {(char*)"_saved_self_sym_sizes", (getter)THPReshapeCopyBackward0_self_sym_sizes_getter, nullptr, nullptr, nullptr},
5664:   {nullptr} /* sentinel */
5665: };
5666: 
5667: static PyObject* THPSliceBackward0_copy_dim_getter(THPCppFunction *self, void *_unused) {
5668:   HANDLE_TH_ERRORS
5669:   auto prop = static_cast<SliceBackward0_copy*>(self->cdata.get())->dim;
5670:   return PyLong_FromUnsignedLong((int64_t) prop);
5671:   END_HANDLE_TH_ERRORS
5672: }
5673: 
5674: static PyObject* THPSliceBackward0_copy_end_getter(THPCppFunction *self, void *_unused) {
5675:   HANDLE_TH_ERRORS
5676:   auto opt_prop = static_cast<SliceBackward0_copy*>(self->cdata.get())->end;
5677:   if (!opt_prop.has_value()) {
5678:     Py_RETURN_NONE;
5679:   }
5680:   auto prop = opt_prop.value();
```

- EN: The main execution path in this span is carried by `THPVariable_Wrap`, `THPTestAutogradMultipleDispatchViewBackward0_self_raw_getter`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_Wrap`, `THPTestAutogradMultipleDispatchViewBackward0_self_raw_getter`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 5681-5760

```cpp
5681:   if (auto m = prop.maybe_as_int()) {
5682:     return PyLong_FromUnsignedLong(*m);
5683:   } else {
5684:     return py::cast(prop).release().ptr();
5685:   }
5686:   END_HANDLE_TH_ERRORS
5687: }
5688: 
5689: static PyObject* THPSliceBackward0_copy_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
5690:   HANDLE_TH_ERRORS
5691:   auto prop = static_cast<SliceBackward0_copy*>(self->cdata.get())->self_sym_sizes;
5692:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5693:   for (auto i : c10::irange(prop.size())) {
5694:       auto si = prop[i];
5695:       if (auto m = si.maybe_as_int()) {
5696:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
5697:       } else {
5698:         auto py_symint = py::cast(si).release().ptr();
5699:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
5700:       }
5701:   }
5702:   return tup;
5703:   END_HANDLE_TH_ERRORS
5704: }
5705: 
5706: static PyObject* THPSliceBackward0_copy_start_getter(THPCppFunction *self, void *_unused) {
5707:   HANDLE_TH_ERRORS
5708:   auto opt_prop = static_cast<SliceBackward0_copy*>(self->cdata.get())->start;
5709:   if (!opt_prop.has_value()) {
5710:     Py_RETURN_NONE;
5711:   }
5712:   auto prop = opt_prop.value();
5713:   if (auto m = prop.maybe_as_int()) {
5714:     return PyLong_FromUnsignedLong(*m);
5715:   } else {
5716:     return py::cast(prop).release().ptr();
5717:   }
5718:   END_HANDLE_TH_ERRORS
5719: }
5720: 
5721: static PyObject* THPSliceBackward0_copy_step_getter(THPCppFunction *self, void *_unused) {
5722:   HANDLE_TH_ERRORS
5723:   auto prop = static_cast<SliceBackward0_copy*>(self->cdata.get())->step;
5724:   if (auto m = prop.maybe_as_int()) {
5725:     return PyLong_FromUnsignedLong(*m);
5726:   } else {
5727:     return py::cast(prop).release().ptr();
5728:   }
5729:   END_HANDLE_TH_ERRORS
5730: }
5731: 
5732: static struct PyGetSetDef SliceBackward0_copy_properties[] = {
5733:   THP_FUNCTION_DEFAULT_PROPERTIES,
5734:   {(char*)"_saved_dim", (getter)THPSliceBackward0_copy_dim_getter, nullptr, nullptr, nullptr},
5735:   {(char*)"_saved_end", (getter)THPSliceBackward0_copy_end_getter, nullptr, nullptr, nullptr},
5736:   {(char*)"_saved_self_sym_sizes", (getter)THPSliceBackward0_copy_self_sym_sizes_getter, nullptr, nullptr, nullptr},
5737:   {(char*)"_saved_start", (getter)THPSliceBackward0_copy_start_getter, nullptr, nullptr, nullptr},
5738:   {(char*)"_saved_step", (getter)THPSliceBackward0_copy_step_getter, nullptr, nullptr, nullptr},
5739:   {nullptr} /* sentinel */
5740: };
5741: 
5742: static PyObject* THPSplitWithSizesBackward0_copy_dim_getter(THPCppFunction *self, void *_unused) {
5743:   HANDLE_TH_ERRORS
5744:   auto prop = static_cast<SplitWithSizesBackward0_copy*>(self->cdata.get())->dim;
5745:   return PyLong_FromUnsignedLong((int64_t) prop);
5746:   END_HANDLE_TH_ERRORS
5747: }
5748: 
5749: static PyObject* THPSplitWithSizesBackward0_copy_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
5750:   HANDLE_TH_ERRORS
5751:   auto prop = static_cast<SplitWithSizesBackward0_copy*>(self->cdata.get())->self_sym_sizes;
5752:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5753:   for (auto i : c10::irange(prop.size())) {
5754:       auto si = prop[i];
5755:       if (auto m = si.maybe_as_int()) {
5756:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
5757:       } else {
5758:         auto py_symint = py::cast(si).release().ptr();
5759:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
5760:       }
```

- EN: The main execution path in this span is carried by `PyLong_FromUnsignedLong`, `cast`, `THPSliceBackward0_copy_self_sym_sizes_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyLong_FromUnsignedLong`, `cast`, `THPSliceBackward0_copy_self_sym_sizes_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5761-5840

```cpp
5761:   }
5762:   return tup;
5763:   END_HANDLE_TH_ERRORS
5764: }
5765: 
5766: static PyObject* THPSplitWithSizesBackward0_copy_split_sizes_getter(THPCppFunction *self, void *_unused) {
5767:   HANDLE_TH_ERRORS
5768:   auto prop = static_cast<SplitWithSizesBackward0_copy*>(self->cdata.get())->split_sizes;
5769:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5770:   for (auto i : c10::irange(prop.size())) {
5771:       auto si = prop[i];
5772:       if (auto m = si.maybe_as_int()) {
5773:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
5774:       } else {
5775:         auto py_symint = py::cast(si).release().ptr();
5776:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
5777:       }
5778:   }
5779:   return tup;
5780:   END_HANDLE_TH_ERRORS
5781: }
5782: 
5783: static struct PyGetSetDef SplitWithSizesBackward0_copy_properties[] = {
5784:   THP_FUNCTION_DEFAULT_PROPERTIES,
5785:   {(char*)"_saved_dim", (getter)THPSplitWithSizesBackward0_copy_dim_getter, nullptr, nullptr, nullptr},
5786:   {(char*)"_saved_self_sym_sizes", (getter)THPSplitWithSizesBackward0_copy_self_sym_sizes_getter, nullptr, nullptr, nullptr},
5787:   {(char*)"_saved_split_sizes", (getter)THPSplitWithSizesBackward0_copy_split_sizes_getter, nullptr, nullptr, nullptr},
5788:   {nullptr} /* sentinel */
5789: };
5790: 
5791: static PyObject* THPSplitWithSizesBackwardAutogradNestedTensor0_copy_dim_getter(THPCppFunction *self, void *_unused) {
5792:   HANDLE_TH_ERRORS
5793:   auto prop = static_cast<SplitWithSizesBackwardAutogradNestedTensor0_copy*>(self->cdata.get())->dim;
5794:   return PyLong_FromUnsignedLong((int64_t) prop);
5795:   END_HANDLE_TH_ERRORS
5796: }
5797: 
5798: static PyObject* THPSplitWithSizesBackwardAutogradNestedTensor0_copy_self_getter(THPCppFunction *self, void *_unused) {
5799:   HANDLE_TH_ERRORS
5800:   const auto& prop = static_cast<SplitWithSizesBackwardAutogradNestedTensor0_copy*>(self->cdata.get())->self_;
5801:   return THPVariable_Wrap(prop.unpack(self->cdata));
5802:   END_HANDLE_TH_ERRORS
5803: }
5804: 
5805: static PyObject* THPSplitWithSizesBackwardAutogradNestedTensor0_copy_self_raw_getter(THPCppFunction *self, void *_unused) {
5806:   HANDLE_TH_ERRORS
5807:   const auto& prop = static_cast<SplitWithSizesBackwardAutogradNestedTensor0_copy*>(self->cdata.get())->self_;
5808:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5809:   return obj.release().ptr();
5810:   END_HANDLE_TH_ERRORS
5811: }
5812: 
5813: static PyObject* THPSplitWithSizesBackwardAutogradNestedTensor0_copy_split_sizes_getter(THPCppFunction *self, void *_unused) {
5814:   HANDLE_TH_ERRORS
5815:   auto prop = static_cast<SplitWithSizesBackwardAutogradNestedTensor0_copy*>(self->cdata.get())->split_sizes;
5816:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5817:   for (auto i : c10::irange(prop.size())) {
5818:       auto si = prop[i];
5819:       if (auto m = si.maybe_as_int()) {
5820:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
5821:       } else {
5822:         auto py_symint = py::cast(si).release().ptr();
5823:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
5824:       }
5825:   }
5826:   return tup;
5827:   END_HANDLE_TH_ERRORS
5828: }
5829: 
5830: static struct PyGetSetDef SplitWithSizesBackwardAutogradNestedTensor0_copy_properties[] = {
5831:   THP_FUNCTION_DEFAULT_PROPERTIES,
5832:   {(char*)"_saved_dim", (getter)THPSplitWithSizesBackwardAutogradNestedTensor0_copy_dim_getter, nullptr, nullptr, nullptr},
5833:   {(char*)"_saved_self", (getter)THPSplitWithSizesBackwardAutogradNestedTensor0_copy_self_getter, nullptr, nullptr, nullptr},
5834:   {(char*)"_raw_saved_self", (getter)THPSplitWithSizesBackwardAutogradNestedTensor0_copy_self_raw_getter, nullptr, nullptr, nullptr},
5835:   {(char*)"_saved_split_sizes", (getter)THPSplitWithSizesBackwardAutogradNestedTensor0_copy_split_sizes_getter, nullptr, nullptr, nullptr},
5836:   {nullptr} /* sentinel */
5837: };
5838: 
5839: static PyObject* THPSqueezeBackward0_copy_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
5840:   HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPSplitWithSizesBackward0_copy_split_sizes_getter`, `PyTuple_New`, `PyTuple_SetItem`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPSplitWithSizesBackward0_copy_split_sizes_getter`, `PyTuple_New`, `PyTuple_SetItem` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 5841-5920

```cpp
5841:   auto prop = static_cast<SqueezeBackward0_copy*>(self->cdata.get())->self_sym_sizes;
5842:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5843:   for (auto i : c10::irange(prop.size())) {
5844:       auto si = prop[i];
5845:       if (auto m = si.maybe_as_int()) {
5846:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
5847:       } else {
5848:         auto py_symint = py::cast(si).release().ptr();
5849:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
5850:       }
5851:   }
5852:   return tup;
5853:   END_HANDLE_TH_ERRORS
5854: }
5855: 
5856: static struct PyGetSetDef SqueezeBackward0_copy_properties[] = {
5857:   THP_FUNCTION_DEFAULT_PROPERTIES,
5858:   {(char*)"_saved_self_sym_sizes", (getter)THPSqueezeBackward0_copy_self_sym_sizes_getter, nullptr, nullptr, nullptr},
5859:   {nullptr} /* sentinel */
5860: };
5861: 
5862: static PyObject* THPSqueezeBackward1_copy_dim_getter(THPCppFunction *self, void *_unused) {
5863:   HANDLE_TH_ERRORS
5864:   auto prop = static_cast<SqueezeBackward1_copy*>(self->cdata.get())->dim;
5865:   return PyLong_FromUnsignedLong((int64_t) prop);
5866:   END_HANDLE_TH_ERRORS
5867: }
5868: 
5869: static PyObject* THPSqueezeBackward1_copy_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
5870:   HANDLE_TH_ERRORS
5871:   auto prop = static_cast<SqueezeBackward1_copy*>(self->cdata.get())->self_sym_sizes;
5872:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5873:   for (auto i : c10::irange(prop.size())) {
5874:       auto si = prop[i];
5875:       if (auto m = si.maybe_as_int()) {
5876:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
5877:       } else {
5878:         auto py_symint = py::cast(si).release().ptr();
5879:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
5880:       }
5881:   }
5882:   return tup;
5883:   END_HANDLE_TH_ERRORS
5884: }
5885: 
5886: static struct PyGetSetDef SqueezeBackward1_copy_properties[] = {
5887:   THP_FUNCTION_DEFAULT_PROPERTIES,
5888:   {(char*)"_saved_dim", (getter)THPSqueezeBackward1_copy_dim_getter, nullptr, nullptr, nullptr},
5889:   {(char*)"_saved_self_sym_sizes", (getter)THPSqueezeBackward1_copy_self_sym_sizes_getter, nullptr, nullptr, nullptr},
5890:   {nullptr} /* sentinel */
5891: };
5892: 
5893: static PyObject* THPSqueezeBackwardAutogradNestedTensor0_copy_dim_getter(THPCppFunction *self, void *_unused) {
5894:   HANDLE_TH_ERRORS
5895:   auto prop = static_cast<SqueezeBackwardAutogradNestedTensor0_copy*>(self->cdata.get())->dim;
5896:   return PyLong_FromUnsignedLong((int64_t) prop);
5897:   END_HANDLE_TH_ERRORS
5898: }
5899: 
5900: static struct PyGetSetDef SqueezeBackwardAutogradNestedTensor0_copy_properties[] = {
5901:   THP_FUNCTION_DEFAULT_PROPERTIES,
5902:   {(char*)"_saved_dim", (getter)THPSqueezeBackwardAutogradNestedTensor0_copy_dim_getter, nullptr, nullptr, nullptr},
5903:   {nullptr} /* sentinel */
5904: };
5905: 
5906: static PyObject* THPSqueezeBackward2_copy_dim_getter(THPCppFunction *self, void *_unused) {
5907:   HANDLE_TH_ERRORS
5908:   auto prop = static_cast<SqueezeBackward2_copy*>(self->cdata.get())->dim;
5909:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5910:   for (auto i : c10::irange(prop.size())) {
5911:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
5912:   }
5913:   return tup;
5914:   END_HANDLE_TH_ERRORS
5915: }
5916: 
5917: static PyObject* THPSqueezeBackward2_copy_self_sym_sizes_getter(THPCppFunction *self, void *_unused) {
5918:   HANDLE_TH_ERRORS
5919:   auto prop = static_cast<SqueezeBackward2_copy*>(self->cdata.get())->self_sym_sizes;
5920:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
```

- EN: The main execution path in this span is carried by `PyTuple_New`, `PyTuple_SetItem`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `PyTuple_New`, `PyTuple_SetItem`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 5921-6000

```cpp
5921:   for (auto i : c10::irange(prop.size())) {
5922:       auto si = prop[i];
5923:       if (auto m = si.maybe_as_int()) {
5924:         PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong(*m));
5925:       } else {
5926:         auto py_symint = py::cast(si).release().ptr();
5927:         PyTuple_SetItem(tup, (Py_ssize_t) i, py_symint);
5928:       }
5929:   }
5930:   return tup;
5931:   END_HANDLE_TH_ERRORS
5932: }
5933: 
5934: static struct PyGetSetDef SqueezeBackward2_copy_properties[] = {
5935:   THP_FUNCTION_DEFAULT_PROPERTIES,
5936:   {(char*)"_saved_dim", (getter)THPSqueezeBackward2_copy_dim_getter, nullptr, nullptr, nullptr},
5937:   {(char*)"_saved_self_sym_sizes", (getter)THPSqueezeBackward2_copy_self_sym_sizes_getter, nullptr, nullptr, nullptr},
5938:   {nullptr} /* sentinel */
5939: };
5940: 
5941: static PyObject* THPSqueezeBackwardAutogradNestedTensor1_copy_dim_getter(THPCppFunction *self, void *_unused) {
5942:   HANDLE_TH_ERRORS
5943:   auto prop = static_cast<SqueezeBackwardAutogradNestedTensor1_copy*>(self->cdata.get())->dim;
5944:   PyObject* tup = PyTuple_New((Py_ssize_t) prop.size());
5945:   for (auto i : c10::irange(prop.size())) {
5946:     PyTuple_SetItem(tup, (Py_ssize_t) i, PyLong_FromUnsignedLong((uint64_t) prop[i]));
5947:   }
5948:   return tup;
5949:   END_HANDLE_TH_ERRORS
5950: }
5951: 
5952: static PyObject* THPSqueezeBackwardAutogradNestedTensor1_copy_self_dim_getter(THPCppFunction *self, void *_unused) {
5953:   HANDLE_TH_ERRORS
5954:   auto prop = static_cast<SqueezeBackwardAutogradNestedTensor1_copy*>(self->cdata.get())->self_dim;
5955:   return PyLong_FromUnsignedLong((int64_t) prop);
5956:   END_HANDLE_TH_ERRORS
5957: }
5958: 
5959: static struct PyGetSetDef SqueezeBackwardAutogradNestedTensor1_copy_properties[] = {
5960:   THP_FUNCTION_DEFAULT_PROPERTIES,
5961:   {(char*)"_saved_dim", (getter)THPSqueezeBackwardAutogradNestedTensor1_copy_dim_getter, nullptr, nullptr, nullptr},
5962:   {(char*)"_saved_self_dim", (getter)THPSqueezeBackwardAutogradNestedTensor1_copy_self_dim_getter, nullptr, nullptr, nullptr},
5963:   {nullptr} /* sentinel */
5964: };
5965: 
5966: static PyObject* THPValuesBackward0_copy_self_getter(THPCppFunction *self, void *_unused) {
5967:   HANDLE_TH_ERRORS
5968:   const auto& prop = static_cast<ValuesBackward0_copy*>(self->cdata.get())->self_;
5969:   return THPVariable_Wrap(prop.unpack(self->cdata));
5970:   END_HANDLE_TH_ERRORS
5971: }
5972: 
5973: static PyObject* THPValuesBackward0_copy_self_raw_getter(THPCppFunction *self, void *_unused) {
5974:   HANDLE_TH_ERRORS
5975:   const auto& prop = static_cast<ValuesBackward0_copy*>(self->cdata.get())->self_;
5976:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5977:   return obj.release().ptr();
5978:   END_HANDLE_TH_ERRORS
5979: }
5980: 
5981: static struct PyGetSetDef ValuesBackward0_copy_properties[] = {
5982:   THP_FUNCTION_DEFAULT_PROPERTIES,
5983:   {(char*)"_saved_self", (getter)THPValuesBackward0_copy_self_getter, nullptr, nullptr, nullptr},
5984:   {(char*)"_raw_saved_self", (getter)THPValuesBackward0_copy_self_raw_getter, nullptr, nullptr, nullptr},
5985:   {nullptr} /* sentinel */
5986: };
5987: 
5988: static PyObject* THPValuesBackwardAutogradNestedTensor0_copy_self_getter(THPCppFunction *self, void *_unused) {
5989:   HANDLE_TH_ERRORS
5990:   const auto& prop = static_cast<ValuesBackwardAutogradNestedTensor0_copy*>(self->cdata.get())->self_;
5991:   return THPVariable_Wrap(prop.unpack(self->cdata));
5992:   END_HANDLE_TH_ERRORS
5993: }
5994: 
5995: static PyObject* THPValuesBackwardAutogradNestedTensor0_copy_self_raw_getter(THPCppFunction *self, void *_unused) {
5996:   HANDLE_TH_ERRORS
5997:   const auto& prop = static_cast<ValuesBackwardAutogradNestedTensor0_copy*>(self->cdata.get())->self_;
5998:   pybind11::object obj = pybind11::cast(prop, pybind11::return_value_policy::reference);
5999:   return obj.release().ptr();
6000:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `PyTuple_SetItem`, `cast`, `THPSqueezeBackwardAutogradNestedTensor1_copy_dim_getter`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `PyTuple_SetItem`, `cast`, `THPSqueezeBackwardAutogradNestedTensor1_copy_dim_getter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 6001-6080

```cpp
6001: }
6002: 
6003: static struct PyGetSetDef ValuesBackwardAutogradNestedTensor0_copy_properties[] = {
6004:   THP_FUNCTION_DEFAULT_PROPERTIES,
6005:   {(char*)"_saved_self", (getter)THPValuesBackwardAutogradNestedTensor0_copy_self_getter, nullptr, nullptr, nullptr},
6006:   {(char*)"_raw_saved_self", (getter)THPValuesBackwardAutogradNestedTensor0_copy_self_raw_getter, nullptr, nullptr, nullptr},
6007:   {nullptr} /* sentinel */
6008: };
6009: 
6010: void initialize_autogenerated_functions_4(PyObject* module) {
6011:   static PyTypeObject AcosBackward0Class;
6012:   addClass<AcosBackward0>(module, AcosBackward0Class, "AcosBackward0", AcosBackward0_properties);
6013:   static PyTypeObject AsinBackward0Class;
6014:   addClass<AsinBackward0>(module, AsinBackward0Class, "AsinBackward0", AsinBackward0_properties);
6015:   static PyTypeObject MatmulBackward0Class;
6016:   addClass<MatmulBackward0>(module, MatmulBackward0Class, "MatmulBackward0", MatmulBackward0_properties);
6017:   static PyTypeObject CopysignBackward0Class;
6018:   addClass<CopysignBackward0>(module, CopysignBackward0Class, "CopysignBackward0", CopysignBackward0_properties);
6019:   static PyTypeObject CopysignBackward1Class;
6020:   addClass<CopysignBackward1>(module, CopysignBackward1Class, "CopysignBackward1", CopysignBackward1_properties);
6021:   static PyTypeObject Deg2RadBackward0Class;
6022:   addClass<Deg2RadBackward0>(module, Deg2RadBackward0Class, "Deg2RadBackward0", Deg2RadBackward0_properties);
6023:   static PyTypeObject LinalgSlogdetBackward0Class;
6024:   addClass<LinalgSlogdetBackward0>(module, LinalgSlogdetBackward0Class, "LinalgSlogdetBackward0", LinalgSlogdetBackward0_properties);
6025:   static PyTypeObject DistBackward0Class;
6026:   addClass<DistBackward0>(module, DistBackward0Class, "DistBackward0", DistBackward0_properties);
6027:   static PyTypeObject ErfcBackward0Class;
6028:   addClass<ErfcBackward0>(module, ErfcBackward0Class, "ErfcBackward0", ErfcBackward0_properties);
6029:   static PyTypeObject SpecialErfcxBackward0Class;
6030:   addClass<SpecialErfcxBackward0>(module, SpecialErfcxBackward0Class, "SpecialErfcxBackward0", SpecialErfcxBackward0_properties);
6031:   static PyTypeObject ErfinvBackward0Class;
6032:   addClass<ErfinvBackward0>(module, ErfinvBackward0Class, "ErfinvBackward0", ErfinvBackward0_properties);
6033:   static PyTypeObject FmodBackward0Class;
6034:   addClass<FmodBackward0>(module, FmodBackward0Class, "FmodBackward0", FmodBackward0_properties);
6035:   static PyTypeObject FmodBackward1Class;
6036:   addClass<FmodBackward1>(module, FmodBackward1Class, "FmodBackward1", FmodBackward1_properties);
6037:   static PyTypeObject GeqrfBackward0Class;
6038:   addClass<GeqrfBackward0>(module, GeqrfBackward0Class, "GeqrfBackward0", GeqrfBackward0_properties);
6039:   static PyTypeObject GridSampler2DBackward0Class;
6040:   addClass<GridSampler2DBackward0>(module, GridSampler2DBackward0Class, "GridSampler2DBackward0", GridSampler2DBackward0_properties);
6041:   static PyTypeObject GridSampler2DCpuFallbackBackwardBackward0Class;
6042:   addClass<GridSampler2DCpuFallbackBackwardBackward0>(module, GridSampler2DCpuFallbackBackwardBackward0Class, "GridSampler2DCpuFallbackBackwardBackward0", GridSampler2DCpuFallbackBackwardBackward0_properties);
6043:   static PyTypeObject GtBackward0Class;
6044:   addClass<GtBackward0>(module, GtBackward0Class, "GtBackward0", GtBackward0_properties);
6045:   static PyTypeObject GtBackward1Class;
6046:   addClass<GtBackward1>(module, GtBackward1Class, "GtBackward1", GtBackward1_properties);
6047:   static PyTypeObject SpecialI0EBackward0Class;
6048:   addClass<SpecialI0EBackward0>(module, SpecialI0EBackward0Class, "SpecialI0EBackward0", SpecialI0EBackward0_properties);
6049:   static PyTypeObject SpecialI1Backward0Class;
6050:   addClass<SpecialI1Backward0>(module, SpecialI1Backward0Class, "SpecialI1Backward0", SpecialI1Backward0_properties);
6051:   static PyTypeObject UnsafeMaskedIndexPutAccumulateBackward0Class;
6052:   addClass<UnsafeMaskedIndexPutAccumulateBackward0>(module, UnsafeMaskedIndexPutAccumulateBackward0Class, "UnsafeMaskedIndexPutAccumulateBackward0", UnsafeMaskedIndexPutAccumulateBackward0_properties);
6053:   static PyTypeObject IndexAddBackward0Class;
6054:   addClass<IndexAddBackward0>(module, IndexAddBackward0Class, "IndexAddBackward0", IndexAddBackward0_properties);
6055:   static PyTypeObject IndexPutBackward0Class;
6056:   addClass<IndexPutBackward0>(module, IndexPutBackward0Class, "IndexPutBackward0", IndexPutBackward0_properties);
6057:   static PyTypeObject LogNormalBackward0Class;
6058:   addClass<LogNormalBackward0>(module, LogNormalBackward0Class, "LogNormalBackward0", LogNormalBackward0_properties);
6059:   static PyTypeObject LinalgLuSolveBackward0Class;
6060:   addClass<LinalgLuSolveBackward0>(module, LinalgLuSolveBackward0Class, "LinalgLuSolveBackward0", LinalgLuSolveBackward0_properties);
6061:   static PyTypeObject MaskedFillBackward0Class;
6062:   addClass<MaskedFillBackward0>(module, MaskedFillBackward0Class, "MaskedFillBackward0", MaskedFillBackward0_properties);
6063:   static PyTypeObject MaskedFillBackward1Class;
6064:   addClass<MaskedFillBackward1>(module, MaskedFillBackward1Class, "MaskedFillBackward1", MaskedFillBackward1_properties);
6065:   static PyTypeObject MaskedScatterBackward0Class;
6066:   addClass<MaskedScatterBackward0>(module, MaskedScatterBackward0Class, "MaskedScatterBackward0", MaskedScatterBackward0_properties);
6067:   static PyTypeObject MedianBackward0Class;
6068:   addClass<MedianBackward0>(module, MedianBackward0Class, "MedianBackward0", MedianBackward0_properties);
6069:   static PyTypeObject NanmedianBackward0Class;
6070:   addClass<NanmedianBackward0>(module, NanmedianBackward0Class, "NanmedianBackward0", NanmedianBackward0_properties);
6071:   static PyTypeObject MedianBackward1Class;
6072:   addClass<MedianBackward1>(module, MedianBackward1Class, "MedianBackward1", MedianBackward1_properties);
6073:   static PyTypeObject NanmedianBackward1Class;
6074:   addClass<NanmedianBackward1>(module, NanmedianBackward1Class, "NanmedianBackward1", NanmedianBackward1_properties);
6075:   static PyTypeObject FusedRmsNormBackward0Class;
6076:   addClass<FusedRmsNormBackward0>(module, FusedRmsNormBackward0Class, "FusedRmsNormBackward0", FusedRmsNormBackward0_properties);
6077:   static PyTypeObject BatchNormNoUpdateBackward0Class;
6078:   addClass<BatchNormNoUpdateBackward0>(module, BatchNormNoUpdateBackward0Class, "BatchNormNoUpdateBackward0", BatchNormNoUpdateBackward0_properties);
6079:   static PyTypeObject PoissonBackward0Class;
6080:   addClass<PoissonBackward0>(module, PoissonBackward0Class, "PoissonBackward0", PoissonBackward0_properties);
```

- EN: The main execution path in this span is carried by `initialize_autogenerated_functions_4`. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `initialize_autogenerated_functions_4` 等函数/方法承载。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 6081-6160

```cpp
6081:   static PyTypeObject ProdBackward0Class;
6082:   addClass<ProdBackward0>(module, ProdBackward0Class, "ProdBackward0", ProdBackward0_properties);
6083:   static PyTypeObject ProdBackward1Class;
6084:   addClass<ProdBackward1>(module, ProdBackward1Class, "ProdBackward1", ProdBackward1_properties);
6085:   static PyTypeObject LinalgQrBackward0Class;
6086:   addClass<LinalgQrBackward0>(module, LinalgQrBackward0Class, "LinalgQrBackward0", LinalgQrBackward0_properties);
6087:   static PyTypeObject RemainderBackward0Class;
6088:   addClass<RemainderBackward0>(module, RemainderBackward0Class, "RemainderBackward0", RemainderBackward0_properties);
6089:   static PyTypeObject RemainderBackward1Class;
6090:   addClass<RemainderBackward1>(module, RemainderBackward1Class, "RemainderBackward1", RemainderBackward1_properties);
6091:   static PyTypeObject RenormBackward0Class;
6092:   addClass<RenormBackward0>(module, RenormBackward0Class, "RenormBackward0", RenormBackward0_properties);
6093:   static PyTypeObject SincBackward0Class;
6094:   addClass<SincBackward0>(module, SincBackward0Class, "SincBackward0", SincBackward0_properties);
6095:   static PyTypeObject SliceBackward0Class;
6096:   addClass<SliceBackward0>(module, SliceBackward0Class, "SliceBackward0", SliceBackward0_properties);
6097:   static PyTypeObject SliceInverseBackward0Class;
6098:   addClass<SliceInverseBackward0>(module, SliceInverseBackward0Class, "SliceInverseBackward0", SliceInverseBackward0_properties);
6099:   static PyTypeObject SelectScatterBackward0Class;
6100:   addClass<SelectScatterBackward0>(module, SelectScatterBackward0Class, "SelectScatterBackward0", SelectScatterBackward0_properties);
6101:   static PyTypeObject LinalgSolveExBackward0Class;
6102:   addClass<LinalgSolveExBackward0>(module, LinalgSolveExBackward0Class, "LinalgSolveExBackward0", LinalgSolveExBackward0_properties);
6103:   static PyTypeObject UnsafeSplitWithSizesBackward0Class;
6104:   addClass<UnsafeSplitWithSizesBackward0>(module, UnsafeSplitWithSizesBackward0Class, "UnsafeSplitWithSizesBackward0", UnsafeSplitWithSizesBackward0_properties);
6105:   static PyTypeObject SubBackward0Class;
6106:   addClass<SubBackward0>(module, SubBackward0Class, "SubBackward0", SubBackward0_properties);
6107:   static PyTypeObject SubBackward1Class;
6108:   addClass<SubBackward1>(module, SubBackward1Class, "SubBackward1", SubBackward1_properties);
6109:   static PyTypeObject LinalgSvdBackward0Class;
6110:   addClass<LinalgSvdBackward0>(module, LinalgSvdBackward0Class, "LinalgSvdBackward0", LinalgSvdBackward0_properties);
6111:   static PyTypeObject TakeBackward0Class;
6112:   addClass<TakeBackward0>(module, TakeBackward0Class, "TakeBackward0", TakeBackward0_properties);
6113:   static PyTypeObject TanBackward0Class;
6114:   addClass<TanBackward0>(module, TanBackward0Class, "TanBackward0", TanBackward0_properties);
6115:   static PyTypeObject TraceBackward0Class;
6116:   addClass<TraceBackward0>(module, TraceBackward0Class, "TraceBackward0", TraceBackward0_properties);
6117:   static PyTypeObject ToSparseCsrBackward0Class;
6118:   addClass<ToSparseCsrBackward0>(module, ToSparseCsrBackward0Class, "ToSparseCsrBackward0", ToSparseCsrBackward0_properties);
6119:   static PyTypeObject ToSparseBsrBackward0Class;
6120:   addClass<ToSparseBsrBackward0>(module, ToSparseBsrBackward0Class, "ToSparseBsrBackward0", ToSparseBsrBackward0_properties);
6121:   static PyTypeObject UnfoldBackward0Class;
6122:   addClass<UnfoldBackward0>(module, UnfoldBackward0Class, "UnfoldBackward0", UnfoldBackward0_properties);
6123:   static PyTypeObject UniqueDimConsecutiveBackward0Class;
6124:   addClass<UniqueDimConsecutiveBackward0>(module, UniqueDimConsecutiveBackward0Class, "UniqueDimConsecutiveBackward0", UniqueDimConsecutiveBackward0_properties);
6125:   static PyTypeObject UnsafeViewBackward0Class;
6126:   addClass<UnsafeViewBackward0>(module, UnsafeViewBackward0Class, "UnsafeViewBackward0", UnsafeViewBackward0_properties);
6127:   static PyTypeObject UnsqueezeBackward0Class;
6128:   addClass<UnsqueezeBackward0>(module, UnsqueezeBackward0Class, "UnsqueezeBackward0", UnsqueezeBackward0_properties);
6129:   static PyTypeObject UnsqueezeBackward1Class;
6130:   addClass<UnsqueezeBackward1>(module, UnsqueezeBackward1Class, "UnsqueezeBackward1", UnsqueezeBackward1_properties);
6131:   static PyTypeObject ViewAsRealBackward0Class;
6132:   addClass<ViewAsRealBackward0>(module, ViewAsRealBackward0Class, "ViewAsRealBackward0", ViewAsRealBackward0_properties);
6133:   static PyTypeObject ViewAsComplexBackward0Class;
6134:   addClass<ViewAsComplexBackward0>(module, ViewAsComplexBackward0Class, "ViewAsComplexBackward0", ViewAsComplexBackward0_properties);
6135:   static PyTypeObject SparseCompressedTensorBackward0Class;
6136:   addClass<SparseCompressedTensorBackward0>(module, SparseCompressedTensorBackward0Class, "SparseCompressedTensorBackward0", SparseCompressedTensorBackward0_properties);
6137:   static PyTypeObject ConstantPadNdBackward0Class;
6138:   addClass<ConstantPadNdBackward0>(module, ConstantPadNdBackward0Class, "ConstantPadNdBackward0", ConstantPadNdBackward0_properties);
6139:   static PyTypeObject EmbeddingBackward0Class;
6140:   addClass<EmbeddingBackward0>(module, EmbeddingBackward0Class, "EmbeddingBackward0", EmbeddingBackward0_properties);
6141:   static PyTypeObject EmbeddingBagBackwardBackward0Class;
6142:   addClass<EmbeddingBagBackwardBackward0>(module, EmbeddingBagBackwardBackward0Class, "EmbeddingBagBackwardBackward0", EmbeddingBagBackwardBackward0_properties);
6143:   static PyTypeObject EmbeddingBagDenseBackwardBackward0Class;
6144:   addClass<EmbeddingBagDenseBackwardBackward0>(module, EmbeddingBagDenseBackwardBackward0Class, "EmbeddingBagDenseBackwardBackward0", EmbeddingBagDenseBackwardBackward0_properties);
6145:   static PyTypeObject SmoothL1LossBackward0Class;
6146:   addClass<SmoothL1LossBackward0>(module, SmoothL1LossBackward0Class, "SmoothL1LossBackward0", SmoothL1LossBackward0_properties);
6147:   static PyTypeObject ReluBackward0Class;
6148:   addClass<ReluBackward0>(module, ReluBackward0Class, "ReluBackward0", ReluBackward0_properties);
6149:   static PyTypeObject EluBackward0Class;
6150:   addClass<EluBackward0>(module, EluBackward0Class, "EluBackward0", EluBackward0_properties);
6151:   static PyTypeObject EluBackward1Class;
6152:   addClass<EluBackward1>(module, EluBackward1Class, "EluBackward1", EluBackward1_properties);
6153:   static PyTypeObject HardshrinkBackwardBackward0Class;
6154:   addClass<HardshrinkBackwardBackward0>(module, HardshrinkBackwardBackward0Class, "HardshrinkBackwardBackward0", HardshrinkBackwardBackward0_properties);
6155:   static PyTypeObject LeakyReluBackward1Class;
6156:   addClass<LeakyReluBackward1>(module, LeakyReluBackward1Class, "LeakyReluBackward1", LeakyReluBackward1_properties);
6157:   static PyTypeObject RreluWithNoiseFunctionalBackward0Class;
6158:   addClass<RreluWithNoiseFunctionalBackward0>(module, RreluWithNoiseFunctionalBackward0Class, "RreluWithNoiseFunctionalBackward0", RreluWithNoiseFunctionalBackward0_properties);
6159:   static PyTypeObject SoftplusBackward0Class;
6160:   addClass<SoftplusBackward0>(module, SoftplusBackward0Class, "SoftplusBackward0", SoftplusBackward0_properties);
```

- EN: This range continues the surrounding implementation through local state updates, helper invocations, and data movement.
- CN: 这一段通过局部状态更新、辅助函数调用与数据传递来延续周围实现。
### Lines 6161-6240

```cpp
6161:   static PyTypeObject ReflectionPad2DBackward0Class;
6162:   addClass<ReflectionPad2DBackward0>(module, ReflectionPad2DBackward0Class, "ReflectionPad2DBackward0", ReflectionPad2DBackward0_properties);
6163:   static PyTypeObject UpsampleBilinear2DAaBackward0Class;
6164:   addClass<UpsampleBilinear2DAaBackward0>(module, UpsampleBilinear2DAaBackward0Class, "UpsampleBilinear2DAaBackward0", UpsampleBilinear2DAaBackward0_properties);
6165:   static PyTypeObject ChannelShuffleBackward0Class;
6166:   addClass<ChannelShuffleBackward0>(module, ChannelShuffleBackward0Class, "ChannelShuffleBackward0", ChannelShuffleBackward0_properties);
6167:   static PyTypeObject AdaptiveAvgPool2DBackward0Class;
6168:   addClass<AdaptiveAvgPool2DBackward0>(module, AdaptiveAvgPool2DBackward0Class, "AdaptiveAvgPool2DBackward0", AdaptiveAvgPool2DBackward0_properties);
6169:   static PyTypeObject AvgPool3DBackward0Class;
6170:   addClass<AvgPool3DBackward0>(module, AvgPool3DBackward0Class, "AvgPool3DBackward0", AvgPool3DBackward0_properties);
6171:   static PyTypeObject MaxPool3DWithIndicesBackward0Class;
6172:   addClass<MaxPool3DWithIndicesBackward0>(module, MaxPool3DWithIndicesBackward0Class, "MaxPool3DWithIndicesBackward0", MaxPool3DWithIndicesBackward0_properties);
6173:   static PyTypeObject ConvolutionBackwardBackward0Class;
6174:   addClass<ConvolutionBackwardBackward0>(module, ConvolutionBackwardBackward0Class, "ConvolutionBackwardBackward0", ConvolutionBackwardBackward0_properties);
6175:   static PyTypeObject ConvolutionOverrideableBackward0Class;
6176:   addClass<ConvolutionOverrideableBackward0>(module, ConvolutionOverrideableBackward0Class, "ConvolutionOverrideableBackward0", ConvolutionOverrideableBackward0_properties);
6177:   static PyTypeObject SlowConvTranspose3DBackward0Class;
6178:   addClass<SlowConvTranspose3DBackward0>(module, SlowConvTranspose3DBackward0Class, "SlowConvTranspose3DBackward0", SlowConvTranspose3DBackward0_properties);
6179:   static PyTypeObject SlowConv3DBackward0Class;
6180:   addClass<SlowConv3DBackward0>(module, SlowConv3DBackward0Class, "SlowConv3DBackward0", SlowConv3DBackward0_properties);
6181:   static PyTypeObject Im2ColBackward0Class;
6182:   addClass<Im2ColBackward0>(module, Im2ColBackward0Class, "Im2ColBackward0", Im2ColBackward0_properties);
6183:   static PyTypeObject AvgPool3DBackwardBackward0Class;
6184:   addClass<AvgPool3DBackwardBackward0>(module, AvgPool3DBackwardBackward0Class, "AvgPool3DBackwardBackward0", AvgPool3DBackwardBackward0_properties);
6185:   static PyTypeObject GluBackwardBackward0Class;
6186:   addClass<GluBackwardBackward0>(module, GluBackwardBackward0Class, "GluBackwardBackward0", GluBackwardBackward0_properties);
6187:   static PyTypeObject HardtanhBackwardBackward0Class;
6188:   addClass<HardtanhBackwardBackward0>(module, HardtanhBackwardBackward0Class, "HardtanhBackwardBackward0", HardtanhBackwardBackward0_properties);
6189:   static PyTypeObject LeakyReluBackwardBackward0Class;
6190:   addClass<LeakyReluBackwardBackward0>(module, LeakyReluBackwardBackward0Class, "LeakyReluBackwardBackward0", LeakyReluBackwardBackward0_properties);
6191:   static PyTypeObject MaxPool2DBackwardBackward0Class;
6192:   addClass<MaxPool2DBackwardBackward0>(module, MaxPool2DBackwardBackward0Class, "MaxPool2DBackwardBackward0", MaxPool2DBackwardBackward0_properties);
6193:   static PyTypeObject MaxPool2DWithIndicesBackwardBackward0Class;
6194:   addClass<MaxPool2DWithIndicesBackwardBackward0>(module, MaxPool2DWithIndicesBackwardBackward0Class, "MaxPool2DWithIndicesBackwardBackward0", MaxPool2DWithIndicesBackwardBackward0_properties);
6195:   static PyTypeObject UpsampleLinear1DBackwardBackward0Class;
6196:   addClass<UpsampleLinear1DBackwardBackward0>(module, UpsampleLinear1DBackwardBackward0Class, "UpsampleLinear1DBackwardBackward0", UpsampleLinear1DBackwardBackward0_properties);
6197:   static PyTypeObject UpsampleNearest1DBackwardBackward0Class;
6198:   addClass<UpsampleNearest1DBackwardBackward0>(module, UpsampleNearest1DBackwardBackward0Class, "UpsampleNearest1DBackwardBackward0", UpsampleNearest1DBackwardBackward0_properties);
6199:   static PyTypeObject UpsampleNearest2DBackwardBackward0Class;
6200:   addClass<UpsampleNearest2DBackwardBackward0>(module, UpsampleNearest2DBackwardBackward0Class, "UpsampleNearest2DBackwardBackward0", UpsampleNearest2DBackwardBackward0_properties);
6201:   static PyTypeObject MpsConvolutionTransposeBackward0Class;
6202:   addClass<MpsConvolutionTransposeBackward0>(module, MpsConvolutionTransposeBackward0Class, "MpsConvolutionTransposeBackward0", MpsConvolutionTransposeBackward0_properties);
6203:   static PyTypeObject CudnnBatchNormBackward0Class;
6204:   addClass<CudnnBatchNormBackward0>(module, CudnnBatchNormBackward0Class, "CudnnBatchNormBackward0", CudnnBatchNormBackward0_properties);
6205:   static PyTypeObject MiopenConvolutionTransposeBackward0Class;
6206:   addClass<MiopenConvolutionTransposeBackward0>(module, MiopenConvolutionTransposeBackward0Class, "MiopenConvolutionTransposeBackward0", MiopenConvolutionTransposeBackward0_properties);
6207:   static PyTypeObject MiopenBatchNormBackward0Class;
6208:   addClass<MiopenBatchNormBackward0>(module, MiopenBatchNormBackward0Class, "MiopenBatchNormBackward0", MiopenBatchNormBackward0_properties);
6209:   static PyTypeObject MkldnnAdaptiveAvgPool2DBackward0Class;
6210:   addClass<MkldnnAdaptiveAvgPool2DBackward0>(module, MkldnnAdaptiveAvgPool2DBackward0Class, "MkldnnAdaptiveAvgPool2DBackward0", MkldnnAdaptiveAvgPool2DBackward0_properties);
6211:   static PyTypeObject NestedTensorFromMaskBackward0Class;
6212:   addClass<NestedTensorFromMaskBackward0>(module, NestedTensorFromMaskBackward0Class, "NestedTensorFromMaskBackward0", NestedTensorFromMaskBackward0_properties);
6213:   static PyTypeObject NestedFromPaddedTensorBackward0Class;
6214:   addClass<NestedFromPaddedTensorBackward0>(module, NestedFromPaddedTensorBackward0Class, "NestedFromPaddedTensorBackward0", NestedFromPaddedTensorBackward0_properties);
6215:   static PyTypeObject NestedGetValuesBackward0Class;
6216:   addClass<NestedGetValuesBackward0>(module, NestedGetValuesBackward0Class, "NestedGetValuesBackward0", NestedGetValuesBackward0_properties);
6217:   static PyTypeObject CudnnAttentionBackward0Class;
6218:   addClass<CudnnAttentionBackward0>(module, CudnnAttentionBackward0Class, "CudnnAttentionBackward0", CudnnAttentionBackward0_properties);
6219:   static PyTypeObject ScaledDotProductFusedAttentionOverrideableBackward0Class;
6220:   addClass<ScaledDotProductFusedAttentionOverrideableBackward0>(module, ScaledDotProductFusedAttentionOverrideableBackward0Class, "ScaledDotProductFusedAttentionOverrideableBackward0", ScaledDotProductFusedAttentionOverrideableBackward0_properties);
6221:   static PyTypeObject FftC2RBackward0Class;
6222:   addClass<FftC2RBackward0>(module, FftC2RBackward0Class, "FftC2RBackward0", FftC2RBackward0_properties);
6223:   static PyTypeObject TestWarnInAutogradBackward0Class;
6224:   addClass<TestWarnInAutogradBackward0>(module, TestWarnInAutogradBackward0Class, "TestWarnInAutogradBackward0", TestWarnInAutogradBackward0_properties);
6225:   static PyTypeObject TestAutogradMultipleDispatchViewBackward0Class;
6226:   addClass<TestAutogradMultipleDispatchViewBackward0>(module, TestAutogradMultipleDispatchViewBackward0Class, "TestAutogradMultipleDispatchViewBackward0", TestAutogradMultipleDispatchViewBackward0_properties);
6227:   static PyTypeObject TestAutogradMultipleDispatchViewBackwardAutogradCUDA0Class;
6228:   addClass<TestAutogradMultipleDispatchViewBackwardAutogradCUDA0>(module, TestAutogradMultipleDispatchViewBackwardAutogradCUDA0Class, "TestAutogradMultipleDispatchViewBackwardAutogradCUDA0", TestAutogradMultipleDispatchViewBackwardAutogradCUDA0_properties);
6229:   static PyTypeObject ReshapeCopyBackward0Class;
6230:   addClass<ReshapeCopyBackward0>(module, ReshapeCopyBackward0Class, "ReshapeCopyBackward0", ReshapeCopyBackward0_properties);
6231:   static PyTypeObject SliceBackward0_copyClass;
6232:   addClass<SliceBackward0_copy>(module, SliceBackward0_copyClass, "SliceBackward0_copy", SliceBackward0_copy_properties);
6233:   static PyTypeObject SplitWithSizesBackward0_copyClass;
6234:   addClass<SplitWithSizesBackward0_copy>(module, SplitWithSizesBackward0_copyClass, "SplitWithSizesBackward0_copy", SplitWithSizesBackward0_copy_properties);
6235:   static PyTypeObject SplitWithSizesBackwardAutogradNestedTensor0_copyClass;
6236:   addClass<SplitWithSizesBackwardAutogradNestedTensor0_copy>(module, SplitWithSizesBackwardAutogradNestedTensor0_copyClass, "SplitWithSizesBackwardAutogradNestedTensor0_copy", SplitWithSizesBackwardAutogradNestedTensor0_copy_properties);
6237:   static PyTypeObject SqueezeBackward0_copyClass;
6238:   addClass<SqueezeBackward0_copy>(module, SqueezeBackward0_copyClass, "SqueezeBackward0_copy", SqueezeBackward0_copy_properties);
6239:   static PyTypeObject SqueezeBackward1_copyClass;
6240:   addClass<SqueezeBackward1_copy>(module, SqueezeBackward1_copyClass, "SqueezeBackward1_copy", SqueezeBackward1_copy_properties);
```

- EN: Autograd-related metadata or graph connectivity is updated here.
- CN: 这里会更新与自动求导相关的元数据或计算图连接关系。
### Lines 6241-6253

```cpp
6241:   static PyTypeObject SqueezeBackwardAutogradNestedTensor0_copyClass;
6242:   addClass<SqueezeBackwardAutogradNestedTensor0_copy>(module, SqueezeBackwardAutogradNestedTensor0_copyClass, "SqueezeBackwardAutogradNestedTensor0_copy", SqueezeBackwardAutogradNestedTensor0_copy_properties);
6243:   static PyTypeObject SqueezeBackward2_copyClass;
6244:   addClass<SqueezeBackward2_copy>(module, SqueezeBackward2_copyClass, "SqueezeBackward2_copy", SqueezeBackward2_copy_properties);
6245:   static PyTypeObject SqueezeBackwardAutogradNestedTensor1_copyClass;
6246:   addClass<SqueezeBackwardAutogradNestedTensor1_copy>(module, SqueezeBackwardAutogradNestedTensor1_copyClass, "SqueezeBackwardAutogradNestedTensor1_copy", SqueezeBackwardAutogradNestedTensor1_copy_properties);
6247:   static PyTypeObject ValuesBackward0_copyClass;
6248:   addClass<ValuesBackward0_copy>(module, ValuesBackward0_copyClass, "ValuesBackward0_copy", ValuesBackward0_copy_properties);
6249:   static PyTypeObject ValuesBackwardAutogradNestedTensor0_copyClass;
6250:   addClass<ValuesBackwardAutogradNestedTensor0_copy>(module, ValuesBackwardAutogradNestedTensor0_copyClass, "ValuesBackwardAutogradNestedTensor0_copy", ValuesBackwardAutogradNestedTensor0_copy_properties);
6251: }
6252: 
6253: } // namespace torch::autograd::generated
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
- Key symbols / 关键符号: `addClass`, `THPAcosBackward0_self_getter`, `THPVariable_Wrap`, `THPAcosBackward0_self_raw_getter`, `THPAsinBackward0_self_getter`, `THPAsinBackward0_self_raw_getter`, `THPMatmulBackward0_other_getter`, `THPMatmulBackward0_other_raw_getter`, `THPMatmulBackward0_self_getter`, `THPMatmulBackward0_self_raw_getter`
- Related subsystems / 相关子系统: Autograd / 自动求导, ATen operator layer / ATen 算子层, c10 core utilities / c10 核心工具, Python binding layer / Python 绑定层
