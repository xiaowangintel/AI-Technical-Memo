# python_function.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/autograd/python_function.cpp`
- Repository: `/root/xw/pytorch`
- Purpose (EN): Implements Python bindings that connect PyTorch C++ autograd/inductor internals to CPython.
- 目的 (CN): 实现 Python 绑定，把 PyTorch C++ 自动求导/inductor 内部能力连接到 CPython。
- Lines: 1985
- Language: C++ / C++

## Line-by-Line Analysis / 逐行分析

### Lines 1-32

```cpp
 1: #include <torch/csrc/autograd/python_function.h>
 2: 
 3: #include <atomic>
 4: 
 5: #include <ATen/ATen.h>
 6: #include <ATen/SequenceNumber.h>
 7: #include <c10/util/irange.h>
 8: #include <pybind11/pybind11.h>
 9: #include <structmember.h>
10: #include <torch/csrc/PyInterpreter.h>
11: #include <torch/csrc/python_headers.h>
12: #include <torch/csrc/utils/pybind.h>
13: 
14: #include <ATen/FuncTorchTLS.h>
15: #include <ATen/functorch/DynamicLayer.h>
16: #include <torch/csrc/DynamicTypes.h>
17: #include <torch/csrc/Exceptions.h>
18: #include <torch/csrc/THP.h>
19: #include <torch/csrc/autograd/functions/accumulate_grad.h>
20: #include <torch/csrc/autograd/functions/basic_ops.h>
21: #include <torch/csrc/autograd/functions/utils.h>
22: #include <torch/csrc/autograd/grad_mode.h>
23: #include <torch/csrc/autograd/graph_task.h>
24: #include <torch/csrc/autograd/python_anomaly_mode.h>
25: #include <torch/csrc/autograd/python_cpp_function.h>
26: #include <torch/csrc/autograd/python_hook.h>
27: #include <torch/csrc/autograd/saved_variable.h>
28: #include <torch/csrc/autograd/utils/wrap_outputs.h>
29: #include <torch/csrc/dynamo/compiled_autograd.h>
30: #include <torch/csrc/jit/frontend/tracer.h>
31: #include <torch/csrc/jit/ir/ir.h>
32: #include <torch/csrc/jit/python/pybind_utils.h>
```

- EN: These lines pull in dependencies such as `torch/csrc/autograd/python_function.h`, `atomic`, `ATen/ATen.h`, establishing the headers needed by the implementation.
- CN: 这些行引入了依赖，例如 `torch/csrc/autograd/python_function.h`, `atomic`, `ATen/ATen.h`，为后续实现建立所需的头文件基础。
### Lines 33-64

```cpp
33: #include <torch/csrc/jit/python/python_tracer.h>
34: #include <torch/csrc/profiler/api.h>
35: #include <torch/csrc/utils/python_numbers.h>
36: #include <torch/csrc/utils/python_strings.h>
37: #include <torch/csrc/utils/tensor_dtypes.h>
38: 
39: #include <torch/csrc/autograd/function.h>
40: #include <functional>
41: #include <memory>
42: #include <stdexcept>
43: #include <string>
44: #include <unordered_map>
45: #include <unordered_set>
46: #include <utility>
47: #include <vector>
48: 
49: using namespace torch;
50: using namespace torch::autograd;
51: using at::Tensor;
52: 
53: PyObject* THPFunctionClass = nullptr;
54: PyObject* THPGradientEdgeClass = nullptr;
55: 
56: #define THPFunction_assert(condition, ...) \
57:   if (!(condition)) {                      \
58:     THPUtils_setError(__VA_ARGS__);        \
59:     throw python_error();                  \
60:   }
61: 
62: // Anonymous namespace for helpful functions used in this file
63: namespace {
64: 
```

- EN: These lines pull in dependencies such as `torch/csrc/jit/python/python_tracer.h`, `torch/csrc/profiler/api.h`, `torch/csrc/utils/python_numbers.h`, establishing the headers needed by the implementation. They also open, refine, or close the relevant C++ namespace scope. The main execution path in this span is carried by `THPUtils_setError`, `python_error`. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这些行引入了依赖，例如 `torch/csrc/jit/python/python_tracer.h`, `torch/csrc/profiler/api.h`, `torch/csrc/utils/python_numbers.h`，为后续实现建立所需的头文件基础。 它们还会打开、细化或关闭相应的 C++ 命名空间作用域。 这一段的主要执行路径由 `THPUtils_setError`, `python_error` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 65-96

```cpp
65: inline void check_legacy_fn_attr_access(
66:     const c10::intrusive_ptr<torch::autograd::Node>& cdata,
67:     const char* attr) {
68:   TORCH_CHECK(
69:       cdata,
70:       "Attribute '",
71:       attr,
72:       "' is invalid for this instance of _C._FunctionBase. "
73:       "Accessing this attribute directly on an instance of autograd.Function "
74:       "is a legacy access pattern that is no longer supported. For examples "
75:       "on how to use new‑style autograd functions, see "
76:       "https://pytorch.org/docs/stable/autograd.html#torch.autograd.Function ");
77: }
78: 
79: // TODO: We shouldn't need to call this function because the engine
80: // can already persist the errors for us. This still seems to be
81: // needed for the DistEngine however.
82: //
83: // python test/distributed/rpc/test_tensorpipe_agent.py -k
84: // test_backward_autograd_engine_error
85: //
86: // See Note [ Persisting PyErr state across autograd engine threads ]
87: void throw_python_error() {
88:   python_error err;
89:   err.persist();
90:   throw std::move(err);
91: }
92: 
93: static PyObject* unpack_saved_variables(
94:     THPFunction* self,
95:     const std::function<PyObject*(const Variable&)>& unpack_fn) {
96:   HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `check_legacy_fn_attr_access`, `TORCH_CHECK`, `throw_python_error`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here.
- CN: 这一段的主要执行路径由 `check_legacy_fn_attr_access`, `TORCH_CHECK`, `throw_python_error` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。
### Lines 97-128

```cpp
 97:   TORCH_CHECK(!self->has_freed_buffers, ERR_BACKWARD_TWICE);
 98:   auto& saved_variables = self->saved_variables;
 99:   if (saved_variables.empty())
100:     return PyTuple_New(0);
101: 
102:   auto num_saved = saved_variables.size();
103:   THPObjectPtr saved(PyTuple_New(static_cast<Py_ssize_t>(num_saved)));
104:   if (!saved)
105:     return nullptr;
106:   auto saved_for = self->cdata.lock();
107:   // This is really a true assert, because we've already tested for the
108:   // self->has_freed_buffers case at the beginning of this function:
109:   // buffers are freed when PyNode dies; if the buffers are not freed,
110:   // PyNode must be live.  (Note that the buffers could be freed
111:   // even though the PyNode is live, but that doesn't matter here
112:   // because we will never hit this line of code if the buffers are freed--
113:   // and in any case saved_for will be non-NULL.)
114:   TORCH_INTERNAL_ASSERT(saved_for);
115:   for (const auto i : c10::irange(num_saved)) {
116:     auto unpacked_var = saved_variables[i].unpack(saved_for);
117:     THPObjectPtr value;
118:     if (!unpacked_var.defined()) {
119:       Py_INCREF(Py_None);
120:       value = Py_None;
121:     } else {
122:       value = unpack_fn(unpacked_var);
123:     }
124:     PyTuple_SET_ITEM(saved.get(), i, value.release());
125:   }
126:   return saved.release();
127:   END_HANDLE_TH_ERRORS
128: }
```

- EN: The main execution path in this span is carried by `TORCH_CHECK`, `PyTuple_New`, `saved`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here.
- CN: 这一段的主要执行路径由 `TORCH_CHECK`, `PyTuple_New`, `saved` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。
### Lines 129-160

```cpp
129: 
130: PyObject* to_py_size(const std::vector<c10::SymInt>& size) {
131:   c10::SymIntArrayRef sym_sizes(size);
132: 
133:   auto ret = THPObjectPtr(THPSizeType.tp_alloc(
134:       &THPSizeType, static_cast<Py_ssize_t>(sym_sizes.size())));
135:   if (!ret)
136:     throw python_error();
137: 
138:   for (auto i : c10::irange(sym_sizes.size())) {
139:     auto symint = sym_sizes[i];
140:     if (auto maybe_int = symint.maybe_as_int(); maybe_int.has_value()) {
141:       PyTuple_SET_ITEM(ret.get(), i, THPUtils_packInt64(*maybe_int));
142:     } else {
143:       auto py_symint = py::cast(symint).release().ptr();
144:       PyTuple_SET_ITEM(ret.get(), i, py_symint);
145:     }
146:   }
147:   return ret.release();
148: }
149: 
150: } // namespace
151: 
152: namespace torch::autograd {
153: 
154: // NOTE: this function is written in a way that assumes it's only called for
155: // backward; it's used by engine.cpp.  This is responsible for forwarding a call
156: // from C++'s Node::apply to a Python method "apply".
157: // NOLINTNEXTLINE(*-rvalue-reference*)
158: auto PyNode::apply(variable_list&& inputs) -> variable_list {
159:   pybind11::gil_scoped_acquire gil;
160:   at::OptionalDeviceGuard _device_guard;
```

- EN: They also open, refine, or close the relevant C++ namespace scope. The main execution path in this span is carried by `to_py_size`, `sym_sizes`, `THPObjectPtr`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here.
- CN: 它们还会打开、细化或关闭相应的 C++ 命名空间作用域。 这一段的主要执行路径由 `to_py_size`, `sym_sizes`, `THPObjectPtr` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。
### Lines 161-192

```cpp
161:   THPFunction* py_fn = (THPFunction*)obj;
162: 
163:   // Massage a C++ variable_list into a Python arguments tuple
164:   THPObjectPtr pyInputs(to_py_args(inputs, &_device_guard));
165:   inputs.clear();
166: 
167:   THPObjectPtr r;
168:   if (py_fn->boxed_grads_call) {
169:     // Move grad tensors from the immutable args tuple into a plain list
170:     // and call apply_boxed instead of apply. This lets backward pop/clear
171:     // individual grads to free memory mid-execution, because the mutable
172:     // list (not the C++ tuple) is the only container holding grad refs.
173:     auto num_inputs = PyTuple_GET_SIZE(pyInputs.get());
174:     THPObjectPtr gradsList(PyList_New(num_inputs));
175:     if (!gradsList)
176:       throw_python_error();
177:     for (Py_ssize_t i = 0; i < num_inputs; i++) {
178:       PyObject* item = PyTuple_GET_ITEM(pyInputs.get(), i);
179:       Py_INCREF(item);
180:       PyList_SET_ITEM(gradsList.get(), i, item);
181:     }
182:     // Release the tuple so its refs to individual grads are dropped
183:     pyInputs = nullptr;
184: 
185:     THPObjectPtr boxedArgs(PyTuple_New(1));
186:     if (!boxedArgs)
187:       throw_python_error();
188:     PyTuple_SET_ITEM(boxedArgs.get(), 0, gradsList.release());
189: 
190:     THPObjectPtr apply_fn(PyObject_GetAttrString(obj, "apply_boxed"));
191:     if (!apply_fn)
192:       throw_python_error();
```

- EN: The main execution path in this span is carried by `pyInputs`, `list`, `PyTuple_GET_SIZE`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `pyInputs`, `list`, `PyTuple_GET_SIZE` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 193-224

```cpp
193:     r = THPObjectPtr(PyObject_CallObject(apply_fn, boxedArgs.get()));
194:   } else {
195:     THPObjectPtr apply_fn(PyObject_GetAttrString(obj, "apply"));
196:     if (!apply_fn)
197:       throw_python_error();
198:     r = THPObjectPtr(PyObject_CallObject(apply_fn, pyInputs.get()));
199:   }
200:   pyInputs = nullptr;
201:   if (!r)
202:     throw_python_error();
203:   ensure_tuple(r);
204: 
205:   auto& is_variable_input = py_fn->is_variable_input;
206:   auto num_outputs = PyTuple_GET_SIZE(r.get());
207:   auto num_forward_inputs = static_cast<Py_ssize_t>(is_variable_input.size());
208:   // Returning too many results is ok, but only as long as they're all None.
209:   // Truncate the result tuple in that case.
210:   if (num_outputs > num_forward_inputs) {
211:     bool all_none = true;
212:     for (const auto i : c10::irange(num_forward_inputs, num_outputs)) {
213:       all_none &= Py_IsNone(PyTuple_GET_ITEM(r.get(), i));
214:     }
215:     if (all_none) {
216:       num_outputs = num_forward_inputs;
217:       r = PyTuple_GetSlice(r.get(), 0, num_forward_inputs);
218:       if (!r)
219:         throw_python_error();
220:     }
221:   }
222: 
223:   // Now the number of gradients should match
224:   TORCH_CHECK(
```

- EN: The main execution path in this span is carried by `THPObjectPtr`, `apply_fn`, `throw_python_error`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPObjectPtr`, `apply_fn`, `throw_python_error` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 225-256

```cpp
225:       num_outputs == num_forward_inputs,
226:       "function ",
227:       name(),
228:       " returned an incorrect number of gradients (expected ",
229:       num_forward_inputs,
230:       ", got ",
231:       num_outputs,
232:       ")");
233: 
234:   // Massage the Python results tuple back into a C++ variable_list
235:   return to_variable_list(r.get(), is_variable_input);
236: }
237: 
238: auto PyNode::apply_with_saved_impl(
239:     const variable_list& inputs,
240:     const SwapSavedVariables& saved) -> variable_list {
241:   pybind11::gil_scoped_acquire gil;
242:   at::OptionalDeviceGuard _device_guard;
243:   THPFunction* py_fn = (THPFunction*)obj;
244: 
245:   // Massage a C++ variable_list into a Python arguments tuple
246:   THPObjectPtr pyInputs(to_py_args(inputs, &_device_guard));
247: 
248:   const auto& is_variable_input = py_fn->is_variable_input;
249:   const auto& input_infos = py_fn->input_info;
250:   // input_info only contains info from variable inputs and should be a subset
251:   TORCH_INTERNAL_ASSERT(is_variable_input.size() >= input_infos.size());
252: 
253:   // The gradients returned in the backwards need to match the number of inputs
254:   // to the forward, and their metadata, so we pass the fwdInputs
255:   THPObjectPtr fwdInputMetadatas(
256:       PyTuple_New(static_cast<Py_ssize_t>(is_variable_input.size())));
```

- EN: The main execution path in this span is carried by `name`, `gradients`, `to_variable_list`. The logic emits runtime diagnostics or assertions to guard assumptions. Autograd-related metadata or graph connectivity is updated here. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `name`, `gradients`, `to_variable_list` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这里会更新与自动求导相关的元数据或计算图连接关系。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 257-288

```cpp
257:   if (!fwdInputMetadatas)
258:     throw python_error();
259: 
260:   int offset = 0;
261:   for (const auto i : c10::irange(is_variable_input.size())) {
262:     if (!is_variable_input[i]) {
263:       // input at i is not a variable, skip index
264:       PyTuple_SET_ITEM(fwdInputMetadatas.get(), i, Py_None);
265:       offset++;
266:       continue;
267:     }
268: 
269:     const auto& input_info = input_infos[i - offset];
270: 
271:     PyObject* device(THPDevice_New(input_info.device));
272:     if (!device)
273:       throw_python_error();
274:     // Metadata is a tuple of 4 elements: (layout, device, dtype, size)
275:     PyObject* fwdInputMetadata = PyTuple_Pack(
276:         4,
277:         autograd::utils::wrap(input_info.layout),
278:         device,
279:         autograd::utils::wrap(input_info.scalar_type),
280:         to_py_size(input_info.size));
281:     if (!fwdInputMetadata)
282:       throw python_error();
283: 
284:     PyTuple_SET_ITEM(fwdInputMetadatas.get(), i, fwdInputMetadata);
285:   }
286:   THPObjectPtr saved_tensors(unpack_saved_variables(
287:       py_fn, [](const Variable& var) { return THPVariable_Wrap(var); }));
288: 
```

- EN: The main execution path in this span is carried by `python_error`, `PyTuple_SET_ITEM`, `device`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `python_error`, `PyTuple_SET_ITEM`, `device` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 289-320

```cpp
289:   auto [bwd_idx, maybe_bwd_state_idx, opaque_obj_indices] =
290:       saved.retrieve_pynode_objs(this);
291: 
292:   PyObject* backward_state_idx = Py_None;
293:   if (maybe_bwd_state_idx.has_value()) {
294:     backward_state_idx = THPUtils_packUInt64(maybe_bwd_state_idx.value());
295:     // this might be simplifiable now that we no longer inline
296:     Py_CLEAR(py_fn->compiled_autograd_backward_state);
297:   }
298: 
299:   THPObjectPtr opaque_indices_list(
300:       PyList_New(static_cast<Py_ssize_t>(opaque_obj_indices.size())));
301:   if (!opaque_indices_list) {
302:     throw_python_error();
303:   }
304:   for (size_t i = 0; i < opaque_obj_indices.size(); i += 1) {
305:     PyList_SET_ITEM(
306:         opaque_indices_list.get(),
307:         i,
308:         THPUtils_packUInt64(opaque_obj_indices[i]));
309:   }
310: 
311:   THPObjectPtr r(PyObject_CallMethod(
312:       saved.get_py_compiler(),
313:       "proxy_call_backward",
314:       "OOOiOOO",
315:       pyInputs.get(),
316:       fwdInputMetadatas.get(),
317:       saved_tensors.get(),
318:       bwd_idx,
319:       obj,
320:       backward_state_idx,
```

- EN: The main execution path in this span is carried by `THPUtils_packUInt64`, `Py_CLEAR`, `opaque_indices_list`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPUtils_packUInt64`, `Py_CLEAR`, `opaque_indices_list` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 321-352

```cpp
321:       opaque_indices_list.get()));
322: 
323:   if (!r)
324:     throw_python_error();
325:   ensure_tuple(r);
326: 
327:   // Massage the Python results tuple back into a C++ variable_list
328:   return to_variable_list(r.get(), is_variable_input);
329: }
330: 
331: auto PyNode::is_traceable() -> bool {
332:   pybind11::gil_scoped_acquire gil;
333:   THPObjectPtr forward_class{PyObject_GetAttrString(obj, "_forward_cls")};
334:   if (!forward_class)
335:     throw_python_error();
336:   THPObjectPtr traceable_py_bool{
337:       PyObject_GetAttrString(forward_class, "is_traceable")};
338:   if (!traceable_py_bool)
339:     throw_python_error();
340:   return Py_IsTrue(traceable_py_bool);
341: }
342: 
343: auto PyNode::release_variables() -> void {
344:   // This function is called as part of the Node destructor!
345:   // Since this object might be kept alive by C++, it is possible
346:   // that the python interpreter is already dead here. In that case
347:   // we just leak the saved objects.
348:   if (Py_IsInitialized()) {
349:     pybind11::gil_scoped_acquire gil;
350:     auto f = (THPFunction*)obj;
351:     f->saved_variables.clear();
352:     f->has_freed_buffers = 1;
```

- EN: The main execution path in this span is carried by `throw_python_error`, `ensure_tuple`, `to_variable_list`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `throw_python_error`, `ensure_tuple`, `to_variable_list` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 353-384

```cpp
353:   }
354: }
355: 
356: void PyNode::release_resources() {
357:   // NB: Node::release_resources calls into release_variables(), which
358:   // accesses the Python object so it must be called first.
359:   Node::release_resources();
360: 
361:   // Release the Python object so that it can be freed when the C++ Node
362:   // outlives all strong references (weak_intrusive_ptr may keep the
363:   // allocation alive, but shouldn't prevent the PyObject from being freed).
364:   if (Py_IsInitialized()) {
365:     pybind11::gil_scoped_acquire gil;
366:     Py_CLEAR(obj);
367:   }
368: }
369: 
370: auto PyNode::name() const -> std::string {
371:   pybind11::gil_scoped_acquire gil;
372:   auto f = (THPFunction*)obj;
373:   auto name = std::string(Py_TYPE(f)->tp_name);
374:   return name;
375: }
376: 
377: bool PyNode::is_aot_backward() const {
378:   py::handle handle(obj);
379:   return py::hasattr(py::getattr(handle, "_forward_cls"), "_aot_id");
380: }
381: 
382: void PyNode::compiled_args(CompiledNodeArgs& args) const {
383:   static PyObject* method_name =
384:       PyUnicode_InternFromString("_compiled_autograd_key");
```

- EN: The main execution path in this span is carried by `release_resources`, `release_variables`, `references`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `release_resources`, `release_variables`, `references` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 385-416

```cpp
385:   THPObjectPtr pykey(PyObject_CallMethodObjArgs(obj, method_name, nullptr));
386:   if (!pykey)
387:     throw_python_error();
388:   TORCH_CHECK(
389:       PyTuple_CheckExact(pykey.get()),
390:       "_compiled_autograd_key should return tuple of ints");
391:   auto size = PyTuple_GET_SIZE(pykey.get());
392:   TORCH_INTERNAL_ASSERT(size > 0);
393:   // first value is unique id managed by AUTOGRAD_FUNCTION_COUNTER
394:   auto key = PyLong_AsSsize_t(PyTuple_GET_ITEM(pykey.get(), 0));
395:   if (C10_UNLIKELY(key < 0)) {
396:     TORCH_CHECK(PyErr_Occurred(), "key must be positive");
397:     throw_python_error();
398:   }
399:   args.collect_size(static_cast<size_t>(key));
400:   args.collect_size(static_cast<size_t>(size));
401: 
402:   auto f = (THPFunction*)obj;
403:   f->compiled_autograd_symints.clear();
404:   f->compiled_autograd_symints.reserve(size - 1);
405:   for (const auto i : c10::irange(1, size)) {
406:     auto val = PyLong_AsSsize_t(PyTuple_GET_ITEM(pykey.get(), i));
407:     if (C10_UNLIKELY(val == -1 && PyErr_Occurred()))
408:       throw_python_error();
409:     f->compiled_autograd_symints.emplace_back(val);
410:   }
411: 
412:   // AotAutograd symints are all dynamic
413:   auto prior =
414:       args.set_default_dyn_type(torch::dynamo::autograd::SizeInput::DYNAMIC);
415:   args.collect(f->compiled_autograd_symints);
416:   args.set_default_dyn_type(prior);
```

- EN: The main execution path in this span is carried by `pykey`, `throw_python_error`, `TORCH_CHECK`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here.
- CN: 这一段的主要执行路径由 `pykey`, `throw_python_error`, `TORCH_CHECK` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。
### Lines 417-448

```cpp
417: 
418:   args.collect(f->saved_variables, true); // always unpacked as output in eager
419:   args.collect(f->materialize_grads);
420:   args.collect(f->is_variable_input);
421:   args.collect(f->needs_input_grad);
422:   args.collect(f->materialize_non_diff_grads);
423:   args.collect(f->output_info);
424:   args.collect(f->input_info);
425: 
426:   Py_INCREF(obj);
427:   c10::SafePyObject backward_obj(obj, getPyInterpreter());
428:   std::optional<c10::SafePyObject> backward_state_obj;
429:   PyObject* bw_state = f->compiled_autograd_backward_state;
430:   if (args.cond(bw_state != nullptr)) {
431:     Py_INCREF(bw_state);
432:     backward_state_obj = c10::SafePyObject(bw_state, getPyInterpreter());
433:   }
434: 
435:   std::vector<c10::SafePyObject> opaque_objs;
436:   if (THPObjectPtr opaque_objs_ptr =
437:           THPObjectPtr(PyObject_GetAttrString(obj, "opaque_objects"))) {
438:     Py_ssize_t size = PySequence_Size(opaque_objs_ptr.get());
439:     if (size > 0) {
440:       opaque_objs.reserve(static_cast<size_t>(size));
441:       for (Py_ssize_t i = 0; i < size; i += 1) {
442:         opaque_objs.emplace_back(c10::SafePyObject(
443:             PySequence_GetItem(opaque_objs_ptr.get(), i), getPyInterpreter()));
444:       }
445:     } else if (size < 0) {
446:       PyErr_Clear();
447:     }
448:   } else {
```

- EN: The main execution path in this span is carried by `Py_INCREF`, `backward_obj`, `SafePyObject`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `Py_INCREF`, `backward_obj`, `SafePyObject` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 449-480

```cpp
449:     PyErr_Clear();
450:   }
451: 
452:   args.collect_pynode_objs(
453:       this,
454:       std::move(backward_obj),
455:       std::move(backward_state_obj),
456:       std::move(opaque_objs));
457: }
458: 
459: variable_list PyNode::apply_with_saved(
460:     const variable_list& inputs,
461:     SwapSavedVariables& saved) {
462:   auto f = (THPFunction*)obj;
463:   saved.before(f->compiled_autograd_symints);
464:   saved.before(f->saved_variables);
465:   saved.before(f->needs_input_grad);
466:   saved.before(f->materialize_non_diff_grads);
467:   saved.before(f->output_info);
468:   saved.before(f->input_info);
469:   variable_list result = apply_with_saved_impl(variable_list(inputs), saved);
470:   saved.after(f->compiled_autograd_symints);
471:   saved.after(f->saved_variables);
472:   saved.after(f->needs_input_grad);
473:   saved.after(f->materialize_non_diff_grads);
474:   saved.after(f->output_info);
475:   saved.after(f->input_info);
476:   return result;
477: }
478: 
479: PyObject* PyNode::to_py_args(
480:     const variable_list& inputs,
```

- EN: The main execution path in this span is carried by `PyErr_Clear`, `move`, `apply_with_saved`. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyErr_Clear`, `move`, `apply_with_saved` 等函数/方法承载。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 481-512

```cpp
481:     at::OptionalDeviceGuard* device_guard) {
482:   THPFunction* py_fn = (THPFunction*)obj;
483: 
484:   auto zeros_without_gil = [](const VariableInfo& variable,
485:                               at::OptionalDeviceGuard& dg) {
486:     pybind11::gil_scoped_release gil;
487:     return variable.zeros(dg);
488:   };
489: 
490:   auto num_inputs = inputs.size();
491:   PyObject* pyInputs = PyTuple_New(static_cast<Py_ssize_t>(num_inputs));
492:   if (!pyInputs)
493:     throw_python_error();
494:   auto& output_info = py_fn->output_info;
495:   for (const auto i : c10::irange(num_inputs)) {
496:     PyObject* input = nullptr;
497:     if (inputs[i].defined() || !py_fn->materialize_grads ||
498:         (input_metadata(i).was_default_constructed() &&
499:          !py_fn->materialize_non_diff_grads)) {
500:       input = THPVariable_Wrap(inputs[i]);
501:     } else {
502:       input =
503:           THPVariable_Wrap(zeros_without_gil(output_info[i], *device_guard));
504:     }
505:     if (!input)
506:       throw_python_error();
507:     PyTuple_SET_ITEM(pyInputs, i, input);
508:   }
509: 
510:   return pyInputs;
511: }
512: 
```

- EN: The main execution path in this span is carried by `PyTuple_New`, `throw_python_error`, `THPVariable_Wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyTuple_New`, `throw_python_error`, `THPVariable_Wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 513-544

```cpp
513: variable_list PyNode::to_variable_list(
514:     const PyObject* outputs,
515:     const std::vector<bool>& is_variable_input) {
516:   auto num_outputs = PyTuple_GET_SIZE(outputs);
517:   variable_list results;
518:   results.reserve(num_outputs);
519:   for (int i = 0; i != num_outputs; ++i) {
520:     PyObject* output = PyTuple_GET_ITEM(outputs, i);
521:     bool was_variable = is_variable_input[i];
522:     if (!was_variable) {
523:       TORCH_CHECK(
524:           Py_IsNone(output),
525:           "function ",
526:           name(),
527:           " returned a gradient different than None at position ",
528:           i + 1,
529:           ", but the corresponding forward input was not a Variable");
530:       continue;
531:     }
532:     if (Py_IsNone(output)) {
533:       results.emplace_back();
534:     } else {
535:       TORCH_CHECK(
536:           THPVariable_Check(output),
537:           "expected Variable or None (got ",
538:           THPUtils_typename(output),
539:           ")");
540: 
541:       results.emplace_back(THPVariable_Unpack(output));
542:     }
543:   }
544: 
```

- EN: The main execution path in this span is carried by `to_variable_list`, `PyTuple_GET_SIZE`, `PyTuple_GET_ITEM`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here.
- CN: 这一段的主要执行路径由 `to_variable_list`, `PyTuple_GET_SIZE`, `PyTuple_GET_ITEM` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。
### Lines 545-576

```cpp
545:   return results;
546: }
547: 
548: } // namespace torch::autograd
549: 
550: // Traverse and clear are required for supporting Python's GC cycle handling.
551: static int THPFunction_traverse(THPFunction* self, visitproc visit, void* arg) {
552:   // NB: We should not traverse PyObbject stored on PyNode, since we only hold
553:   // as weak reference to the PyNode.
554:   Py_VISIT(self->to_save);
555:   Py_VISIT(self->non_differentiable);
556:   Py_VISIT(self->dirty_tensors);
557:   Py_VISIT(self->compiled_autograd_backward_state);
558:   Py_VISIT(self->saved_for_forward);
559:   return 0;
560: }
561: 
562: static int THPFunction_clear(THPFunction* self) {
563:   // Note that the cdata might not be expired yet in the case where this
564:   // object is part of a cycle and the GC happens to tp_clear this PyObject
565:   // before the other ones that trigger the de-allocation of the cdata
566: 
567:   Py_CLEAR(self->needs_input_grad);
568: 
569:   Py_CLEAR(self->to_save);
570:   Py_CLEAR(self->non_differentiable);
571:   Py_CLEAR(self->dirty_tensors);
572:   Py_CLEAR(self->compiled_autograd_backward_state);
573:   Py_CLEAR(self->saved_for_forward);
574: 
575:   self->output_info.clear();
576:   self->input_info.clear();
```

- EN: They also open, refine, or close the relevant C++ namespace scope. The main execution path in this span is carried by `THPFunction_traverse`, `Py_VISIT`, `THPFunction_clear`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here.
- CN: 它们还会打开、细化或关闭相应的 C++ 命名空间作用域。 这一段的主要执行路径由 `THPFunction_traverse`, `Py_VISIT`, `THPFunction_clear` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。
### Lines 577-608

```cpp
577:   self->saved_variables.clear();
578:   self->is_variable_input.clear();
579: 
580:   return 0;
581: }
582: 
583: static void THPFunction_dealloc(THPFunction* self) {
584:   // Why is this guaranteed to be true?  Suppose that self->cdata is non-null
585:   // (otherwise the condition is trivially true).  Then there is a PyNode
586:   // which contains an owning reference to this object.  But we are only
587:   // allowed to clear if all owning references are gone!  Contradiction.
588:   //
589:   // However, note that THPFunction_clear is typically called in the shared_ptr
590:   // destructor of PyNode; in that case, per
591:   // https://cplusplus.github.io/LWG/lwg-active.html#2751 it's not currently
592:   // specified in the standard that this is guaranteed.  If you see this
593:   // assert triggering in the wild, feel free to comment it out.  They're
594:   // likely to standardize that you ARE guaranteed to see the weak pointers
595:   // as expired in the destructor in the future, so we'll keep this for now.
596:   TORCH_INTERNAL_ASSERT(self->cdata.expired());
597: 
598:   PyObject_GC_UnTrack(self);
599:   THPFunction_clear(self);
600:   self->cdata.~weak_intrusive_ptr();
601:   self->output_info.~vector();
602:   self->input_info.~vector();
603:   self->saved_variables.~vector();
604:   self->is_variable_input.~vector();
605:   Py_TYPE(self)->tp_free((PyObject*)self);
606: }
607: 
608: static PyObject* THPFunction_new(
```

- EN: The main execution path in this span is carried by `THPFunction_dealloc`, `TORCH_INTERNAL_ASSERT`, `PyObject_GC_UnTrack`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here.
- CN: 这一段的主要执行路径由 `THPFunction_dealloc`, `TORCH_INTERNAL_ASSERT`, `PyObject_GC_UnTrack` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。
### Lines 609-640

```cpp
609:     PyTypeObject* type,
610:     PyObject* args,
611:     PyObject* kwargs) {
612:   PyObject* obj = type->tp_alloc(type, 0);
613:   if (!obj)
614:     return nullptr;
615:   // Python zero-initializes the object memory, so there's no need to initialize
616:   // most fields
617:   THPFunction* self = (THPFunction*)obj;
618:   // Setup the PyNode later; we can't keep it live here
619:   new (&self->cdata)
620:       c10::weak_intrusive_ptr<PyNode>(c10::intrusive_ptr<PyNode>());
621:   new (&self->output_info) std::vector<VariableInfo>();
622:   new (&self->input_info) std::vector<VariableInfo>();
623:   new (&self->saved_variables) std::vector<SavedVariable>();
624:   new (&self->is_variable_input) std::vector<bool>();
625:   self->materialize_grads = true;
626:   self->pure_view = false;
627:   self->materialize_non_diff_grads = true;
628:   self->clear_saved_tensors_on_access = false;
629:   self->saved_tensors_accessed_and_cleared = false;
630:   return obj;
631: }
632: 
633: ////////////////////////////////////////////////////////////////////////////////
634: // Forward
635: ////////////////////////////////////////////////////////////////////////////////
636: 
637: // Bump the counters of all recorded dirty input tensors, adding each of them
638: // into dirty_inputs.  Also does some sanity checking.
639: static std::unordered_set<at::TensorImpl*> _mark_dirty(THPFunction* self) {
640:   // Increase versions of modified tensors
```

- EN: The main execution path in this span is carried by `new`, `_mark_dirty`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `new`, `_mark_dirty` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 641-672

```cpp
641:   std::unordered_set<at::TensorImpl*> dirty_inputs;
642:   if (!self->dirty_tensors)
643:     return dirty_inputs;
644: 
645:   THPFunction_assert(
646:       PyTuple_Check(self->dirty_tensors),
647:       "autograd "
648:       "internal error: dirty_tensors attribute is expected to be a tuple "
649:       "but is ",
650:       THPUtils_typename(self->dirty_tensors));
651:   Py_ssize_t num_dirty = PyTuple_GET_SIZE(self->dirty_tensors);
652:   dirty_inputs.reserve(num_dirty);
653:   for (const auto i : c10::irange(num_dirty)) {
654:     PyObject* obj = PyTuple_GET_ITEM(self->dirty_tensors, i);
655:     THPFunction_assert(
656:         THPVariable_Check(obj),
657:         "mark_dirty can "
658:         "only accept variables, but argument ",
659:         i,
660:         " is of type ",
661:         THPUtils_typename(obj));
662: 
663:     const auto& tensor = THPVariable_Unpack(obj);
664:     dirty_inputs.insert(tensor.unsafeGetTensorImpl());
665:     torch::autograd::impl::bump_version(tensor);
666:   }
667:   // We're not going to ever need this so let's remove references now
668:   Py_CLEAR(self->dirty_tensors);
669:   return dirty_inputs;
670: }
671: 
672: static std::unordered_set<at::TensorImpl*> _parse_non_differentiable(
```

- EN: The main execution path in this span is carried by `THPFunction_assert`, `PyTuple_Check`, `THPUtils_typename`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPFunction_assert`, `PyTuple_Check`, `THPUtils_typename` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 673-704

```cpp
673:     THPFunction* self);
674: 
675: // Given a Python tuple of raw output tensors (raw_output), set each of
676: // the corresponding entries in a different Python tuple (outputs) with
677: // these tensors wrapped with variables.  We save the gradient function (self)
678: // to the variable if the output requires grad.
679: //
680: // There is a considerable amount of complexity to handle if the operation
681: // that produced these output tensors is inplace.  A mapping of *input*
682: // tensors to variables (t2var) is used to test if this occurred, and
683: // the set of dirty tensors (dirty_inputs) is used to figure out what to
684: // do in this case.  After this method is run, t2var is extended with
685: // mappings for output tensors as well.
686: static void _wrap_outputs(
687:     const c10::intrusive_ptr<PyNode>& cdata,
688:     THPFunction* self,
689:     const variable_list& input_vars,
690:     PyObject* raw_output,
691:     PyObject* outputs,
692:     bool is_executable,
693:     const std::unordered_set<at::TensorImpl*>& to_save_if_setup_context) {
694:   auto cdata_if_executable = is_executable ? cdata : nullptr;
695:   Py_ssize_t num_outputs = PyTuple_GET_SIZE(raw_output);
696:   if (is_executable) {
697:     self->output_info.clear();
698:     self->output_info.reserve(num_outputs);
699:   }
700: 
701:   auto non_differentiable = _parse_non_differentiable(self);
702:   auto dirty_inputs = _mark_dirty(self);
703: 
704:   std::vector<std::optional<Variable>> raw_output_vars;
```

- EN: The main execution path in this span is carried by `tensors`, `tuple`, `function`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `tensors`, `tuple`, `function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 705-736

```cpp
705:   raw_output_vars.reserve(num_outputs);
706:   for (const auto i : c10::irange(num_outputs)) {
707:     PyObject* obj = PyTuple_GET_ITEM(raw_output, i);
708:     // Only process tensors as outputs for autograd purposes.
709:     if (THPVariable_Check(obj)) {
710:       raw_output_vars.emplace_back(THPVariable_Unpack(obj));
711:     } else {
712:       raw_output_vars.emplace_back();
713:     }
714:   }
715: 
716:   _jvp_fn_t jvp_user_function = [self](
717:                                     variable_list inputs,
718:                                     variable_list grad_inputs) {
719:     pybind11::gil_scoped_acquire gil;
720: 
721:     // Massage a C++ variable_list into a Python arguments tuple
722:     // Making sure to introduce the proper None for non-Tensor inputs
723:     auto num_inputs = self->is_variable_input.size();
724:     THPObjectPtr pyInputs(PyTuple_New(static_cast<Py_ssize_t>(num_inputs)));
725:     if (!pyInputs)
726:       throw_python_error();
727:     int64_t variable_idx = 0;
728:     for (const auto i : c10::irange(num_inputs)) {
729:       PyObject* input = nullptr;
730:       if (self->is_variable_input[i]) {
731:         if (grad_inputs[variable_idx].defined() || !self->materialize_grads ||
732:             !isDifferentiableType(inputs[variable_idx].scalar_type())) {
733:           input = THPVariable_Wrap(grad_inputs[variable_idx]);
734:         } else {
735:           input = THPVariable_Wrap(at::zeros_like(inputs[variable_idx]));
736:         }
```

- EN: The main execution path in this span is carried by `PyTuple_GET_ITEM`, `pyInputs`, `throw_python_error`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `PyTuple_GET_ITEM`, `pyInputs`, `throw_python_error` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 737-768

```cpp
737:         if (!input) {
738:           throw_python_error();
739:         }
740:         variable_idx++;
741:       } else {
742:         Py_INCREF(Py_None);
743:         input = Py_None;
744:       }
745:       PyTuple_SET_ITEM(pyInputs.get(), i, input);
746:     }
747: 
748:     THPObjectPtr apply_jvp_fn(
749:         PyObject_GetAttrString((PyObject*)self, "apply_jvp"));
750:     if (!apply_jvp_fn)
751:       throw_python_error();
752:     THPObjectPtr r(PyObject_CallObject(apply_jvp_fn, pyInputs.get()));
753:     if (!r)
754:       throw_python_error();
755:     ensure_tuple(r);
756: 
757:     // Massage the Python results tuple back into a C++ variable_list
758:     // Don't do any check on the number of results here as
759:     // it is handled by the caller
760:     const int num_outputs = PyTuple_GET_SIZE(r.get());
761:     variable_list results;
762:     results.reserve(num_outputs);
763:     for (const auto i : c10::irange(num_outputs)) {
764:       PyObject* output = PyTuple_GET_ITEM(r.get(), i);
765:       if (Py_IsNone(output)) {
766:         results.emplace_back();
767:       } else {
768:         TORCH_CHECK(
```

- EN: The main execution path in this span is carried by `throw_python_error`, `Py_INCREF`, `PyTuple_SET_ITEM`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `throw_python_error`, `Py_INCREF`, `PyTuple_SET_ITEM` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 769-800

```cpp
769:             THPVariable_Check(output),
770:             "expected Variable or None (got ",
771:             THPUtils_typename(output),
772:             ") for grad output ",
773:             i,
774:             ".")
775:         results.emplace_back(THPVariable_Unpack(output));
776:       }
777:     }
778: 
779:     return results;
780:   };
781: 
782:   auto view_as_self_fn = [](const at::Tensor& x) -> at::Tensor {
783:     pybind11::gil_scoped_acquire gil;
784:     THPObjectPtr py_x(THPVariable_Wrap(x));
785:     THPObjectPtr py_view_as_method(PyObject_GetAttrString(py_x, "view_as"));
786:     if (!py_view_as_method)
787:       throw python_error();
788:     THPObjectPtr args(PyTuple_Pack(1, py_x.get()));
789:     if (!args)
790:       throw python_error();
791:     THPObjectPtr result(PyObject_CallObject(py_view_as_method, args));
792:     if (!result)
793:       throw python_error();
794:     return THPVariable_Unpack(result);
795:   };
796: 
797:   // Wrap only the tensor outputs.
798:   auto wrapped_outputs = _wrap_outputs(
799:       input_vars,
800:       non_differentiable,
```

- EN: The main execution path in this span is carried by `THPVariable_Check`, `None`, `THPUtils_typename`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_Check`, `None`, `THPUtils_typename` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 801-832

```cpp
801:       dirty_inputs,
802:       raw_output_vars,
803:       cdata_if_executable,
804:       jvp_user_function,
805:       to_save_if_setup_context,
806:       view_as_self_fn,
807:       self->pure_view);
808: 
809:   for (const auto i : c10::irange(num_outputs)) {
810:     PyObject* obj = PyTuple_GetItem(raw_output, i);
811:     const auto& wrapped_output = wrapped_outputs[i];
812:     // Keep the non-tensor outputs as is.
813:     if (!THPVariable_Check(obj) || !wrapped_output.has_value()) {
814:       if (is_executable) {
815:         self->output_info.emplace_back();
816:       }
817:       Py_INCREF(obj);
818:       PyTuple_SetItem(outputs, i, obj);
819:     } else {
820:       if (is_executable) {
821:         // If one of the grad outputs is undefined, a correctly-shaped zeros
822:         // should be used instead. To construct these for NJT, zeros_like() must
823:         // be used until we have factory function support.
824:         bool is_differentiable =
825:             (non_differentiable.count(wrapped_output->unsafeGetTensorImpl()) ==
826:                  0 &&
827:              isDifferentiableType(wrapped_output->scalar_type()));
828:         bool use_zeros_like =
829:             is_differentiable && num_outputs > 1 && wrapped_output->is_nested();
830:         self->output_info.emplace_back(wrapped_output.value(), use_zeros_like);
831:       }
832:       PyTuple_SetItem(outputs, i, THPVariable_Wrap(wrapped_output.value()));
```

- EN: The main execution path in this span is carried by `PyTuple_GetItem`, `Py_INCREF`, `PyTuple_SetItem`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `PyTuple_GetItem`, `Py_INCREF`, `PyTuple_SetItem` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 833-864

```cpp
833:     }
834:   }
835: }
836: 
837: static void _get_tensors_to_save(
838:     THPFunction* self,
839:     std::unordered_set<at::TensorImpl*>& to_save_if_setup_context,
840:     std::vector<std::optional<at::Tensor>>& tensors_to_save,
841:     bool overridden_setup_context,
842:     bool is_executable) {
843:   if (self->saved_for_forward && overridden_setup_context) {
844:     // We look at saved_for_forward here purely for the purpose of populating
845:     // to_save_if_setup_context, the actual saving is not done here.
846:     THPFunction_assert(
847:         PyTuple_Check(self->saved_for_forward),
848:         "autograd internal "
849:         "error: saved_for_forward attribute is expected to be a tuple but is ",
850:         THPUtils_typename(self->saved_for_forward));
851:     Py_ssize_t num_saved_for_forward =
852:         PyTuple_GET_SIZE(self->saved_for_forward);
853:     for (const auto i : c10::irange(num_saved_for_forward)) {
854:       PyObject* obj = PyTuple_GET_ITEM(self->saved_for_forward, i);
855:       if (THPVariable_Check(obj)) {
856:         const auto& tensor = THPVariable_Unpack(obj);
857:         to_save_if_setup_context.insert(tensor.unsafeGetTensorImpl());
858:       }
859:     }
860:   }
861:   if (self->to_save) {
862:     THPFunction_assert(
863:         PyTuple_Check(self->to_save),
864:         "autograd internal "
```

- EN: The main execution path in this span is carried by `_get_tensors_to_save`, `THPFunction_assert`, `PyTuple_Check`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `_get_tensors_to_save`, `THPFunction_assert`, `PyTuple_Check` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 865-896

```cpp
865:         "error: to_save attribute is expected to be a tuple but is ",
866:         THPUtils_typename(self->to_save));
867: 
868:     Py_ssize_t num_saved = PyTuple_GET_SIZE(self->to_save);
869:     for (const auto i : c10::irange(num_saved)) {
870:       PyObject* obj = PyTuple_GET_ITEM(self->to_save, i);
871:       if (Py_IsNone(obj)) {
872:         tensors_to_save.emplace_back(std::nullopt);
873:         continue;
874:       } else if (THPVariable_Check(obj)) {
875:         const auto& tensor = THPVariable_Unpack(obj);
876:         if (overridden_setup_context) {
877:           to_save_if_setup_context.insert(tensor.unsafeGetTensorImpl());
878:         }
879:         if (is_executable) {
880:           tensors_to_save.emplace_back(tensor);
881:         }
882:       } else {
883:         if (is_executable) {
884:           // TODO: We should really just ALWAYS throw an error here, but
885:           // doing so will break some internal tests. We should fix those.
886:           TORCH_CHECK_TYPE(
887:               false,
888:               fmt::format(
889:                   "save_for_backward can only save variables, but argument {} is of "
890:                   "type {}",
891:                   i,
892:                   Py_TYPE(obj)->tp_name));
893:         }
894:       }
895:     }
896:     Py_CLEAR(self->to_save);
```

- EN: The main execution path in this span is carried by `THPUtils_typename`, `PyTuple_GET_SIZE`, `PyTuple_GET_ITEM`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPUtils_typename`, `PyTuple_GET_SIZE`, `PyTuple_GET_ITEM` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 897-928

```cpp
897:   }
898: }
899: // Save any variables that requested by to_save
900: static void _save_variables(
901:     const std::vector<std::optional<at::Tensor>>& tensors_to_save,
902:     const c10::intrusive_ptr<PyNode>& cdata_ptr,
903:     THPFunction* self,
904:     PyObject* outputs,
905:     int64_t num_outputs) {
906:   if (tensors_to_save.empty())
907:     return;
908:   size_t num_saved = tensors_to_save.size();
909:   self->saved_variables.clear();
910:   self->saved_variables.reserve(num_saved);
911: 
912:   std::unordered_set<at::TensorImpl*> output_impls{};
913:   output_impls.reserve(num_outputs);
914:   for (const auto i : c10::irange(num_outputs)) {
915:     PyObject* obj = PyTuple_GET_ITEM(outputs, i);
916:     if (THPVariable_Check(obj)) {
917:       const auto& tensor = THPVariable_Unpack(obj);
918:       output_impls.insert(tensor.unsafeGetTensorImpl());
919:     }
920:   }
921: 
922:   for (const auto& opt_tensor : tensors_to_save) {
923:     if (!opt_tensor.has_value()) {
924:       self->saved_variables.emplace_back();
925:     } else {
926:       bool is_output =
927:           output_impls.count(opt_tensor.value().unsafeGetTensorImpl()) > 0;
928:       self->saved_variables.emplace_back(opt_tensor.value(), is_output);
```

- EN: The main execution path in this span is carried by `_save_variables`, `PyTuple_GET_ITEM`, `THPVariable_Unpack`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `_save_variables`, `PyTuple_GET_ITEM`, `THPVariable_Unpack` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 929-960

```cpp
929:     }
930:   }
931: }
932: 
933: // Mark requires_grad = 0 on non-differentiable variables (as per
934: // non_differentiable)
935: static std::unordered_set<at::TensorImpl*> _parse_non_differentiable(
936:     THPFunction* self) {
937:   std::unordered_set<at::TensorImpl*> set;
938:   if (!self->non_differentiable)
939:     return set;
940: 
941:   THPFunction_assert(
942:       PyTuple_Check(self->non_differentiable),
943:       "autograd "
944:       "internal error: non_differentiable attribute is expected to be a "
945:       "tuple but is ",
946:       THPUtils_typename(self->non_differentiable));
947:   Py_ssize_t num_nondiff = PyTuple_GET_SIZE(self->non_differentiable);
948:   set.reserve(num_nondiff);
949:   for (const auto i : c10::irange(num_nondiff)) {
950:     PyObject* t = PyTuple_GET_ITEM(self->non_differentiable, i);
951:     THPFunction_assert(
952:         THPVariable_Check(t),
953:         "mark_non_differentiable "
954:         "only accepts variable arguments, but got ",
955:         THPUtils_typename(t));
956:     set.insert(THPVariable_Unpack(t).unsafeGetTensorImpl());
957:   }
958:   Py_CLEAR(self->non_differentiable);
959:   return set;
960: }
```

- EN: The main execution path in this span is carried by `variables`, `_parse_non_differentiable`, `THPFunction_assert`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `variables`, `_parse_non_differentiable`, `THPFunction_assert` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 961-992

```cpp
961: 
962: struct UnpackedInput {
963:   THPObjectPtr input_tuple;
964:   variable_list input_vars;
965:   // record_function_inputs is for RECORD_FUNCTION only
966:   std::vector<c10::IValue> record_function_inputs;
967: };
968: 
969: struct InputFlags {
970:   bool is_executable = false;
971:   edge_list next_edges;
972:   THPObjectPtr needs_input_grad;
973:   std::vector<bool> is_variable_input;
974: };
975: 
976: namespace {
977: template <bool enforce_variables>
978: std::pair<UnpackedInput, InputFlags> unpack_input(PyObject* args) {
979:   UnpackedInput unpacked;
980:   InputFlags flags;
981: 
982:   auto num_args = PyTuple_GET_SIZE(args);
983:   unpacked.input_tuple = PyTuple_New(num_args);
984:   flags.needs_input_grad = PyTuple_New(num_args);
985:   bool profiler_need_input = torch::autograd::profiler::profilerEnabled() &&
986:       torch::autograd::profiler::getProfilerConfig().report_input_shapes;
987: 
988:   for (const auto i : c10::irange(num_args)) {
989:     PyObject* arg = PyTuple_GET_ITEM(args, i);
990: 
991:     bool is_variable = THPVariable_Check(arg);
992:     flags.is_variable_input.push_back(is_variable);
```

- EN: They also open, refine, or close the relevant C++ namespace scope. This range declares or shapes types such as `UnpackedInput`, `InputFlags`. The main execution path in this span is carried by `unpack_input`, `PyTuple_GET_SIZE`, `PyTuple_New`. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 它们还会打开、细化或关闭相应的 C++ 命名空间作用域。 这一段声明或塑造了 ``UnpackedInput`, `InputFlags`` 等类型。 这一段的主要执行路径由 `unpack_input`, `PyTuple_GET_SIZE`, `PyTuple_New` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 993-1024

```cpp
 993:     if (!is_variable) {
 994:       // TODO: remove this code path once Variable and Tensor are merged in
 995:       // Python
 996:       if (enforce_variables) {
 997:         THPUtils_setError(
 998:             "expected a Tensor argument, but got ", THPUtils_typename(arg));
 999:         throw python_error();
1000:       }
1001:       Py_INCREF(Py_False);
1002:       PyTuple_SET_ITEM(flags.needs_input_grad.get(), i, Py_False);
1003: 
1004:       if (profiler_need_input) {
1005:         // The following conversion from PyObject to IValue is expensive
1006:         // Only do it if profiler is enabled and needs input shapes
1007:         auto match = torch::jit::tryToInferPrimitiveType(arg);
1008:         if (match.success()) {
1009:           unpacked.record_function_inputs.push_back(
1010:               torch::jit::toIValue(arg, match.type()));
1011:         }
1012:       }
1013:     } else {
1014:       const auto& tensor = THPVariable_Unpack(arg);
1015:       unpacked.input_vars.push_back(tensor);
1016:       PyObject* needs_grad = tensor.requires_grad() ? Py_True : Py_False;
1017:       Py_INCREF(needs_grad);
1018:       PyTuple_SET_ITEM(flags.needs_input_grad.get(), i, needs_grad);
1019:       unpacked.record_function_inputs.emplace_back(tensor);
1020:     }
1021:     Py_INCREF(arg);
1022:     PyTuple_SET_ITEM(unpacked.input_tuple.get(), i, arg);
1023:   }
1024: 
```

- EN: The main execution path in this span is carried by `THPUtils_setError`, `THPUtils_typename`, `python_error`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPUtils_setError`, `THPUtils_typename`, `python_error` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 1025-1056

```cpp
1025:   flags.is_executable =
1026:       GradMode::is_enabled() && any_variable_requires_grad(unpacked.input_vars);
1027:   flags.next_edges =
1028:       (flags.is_executable ? collect_next_edges(unpacked.input_vars)
1029:                            : edge_list());
1030:   return std::make_pair(std::move(unpacked), std::move(flags));
1031: }
1032: 
1033: // Given a prim::PythonOp node, _append_subgraph creates a subgraph such that:
1034: // (1) It has the same inputs as the prim::PythonOp node
1035: // (2) The intermediate nodes used in the PythonOp are cloned and stored in the
1036: // subgraph (3) trace_outputs stores the Value* objects, before a new trace
1037: // value is assigned by the prim::PythonOp node and helps to eventually route
1038: // the outputs of the subgraph correctly This newly created subgraph is then
1039: // added to the prim::PythonOp node as a subgraph attribute
1040: void _append_subgraph(
1041:     torch::jit::Node* node,
1042:     torch::jit::Graph* graph,
1043:     std::vector<torch::jit::Value*> trace_outputs,
1044:     bool unpack_output) {
1045:   using Value = torch::jit::Value;
1046:   node->g_(
1047:       torch::jit::attr::Subgraph,
1048:       std::make_shared<torch::jit::Graph>(graph->current_scope()));
1049:   auto subgraph = node->g(torch::jit::attr::Subgraph);
1050: 
1051:   std::unordered_map<Value*, Value*> value_map;
1052:   auto value_map_func = [&](Value* v) { return value_map.at(v); };
1053:   for (size_t i = 0; i < node->inputs().size(); ++i) {
1054:     auto subgraph_input = subgraph->addInput();
1055:     subgraph_input->copyMetadata(node->inputs().at(i));
1056:     value_map[node->inputs().at(i)] = subgraph_input;
```

- EN: The main execution path in this span is carried by `is_enabled`, `edge_list`, `make_pair`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `is_enabled`, `edge_list`, `make_pair` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1057-1088

```cpp
1057:   }
1058:   // Find node position in owning block, all subsequent nodes after are added to
1059:   // subgraph
1060:   auto owning_block = node->owningBlock();
1061:   auto it = std::find(
1062:       owning_block->nodes().begin(), owning_block->nodes().end(), node);
1063:   // Skip TupleUnpack node if created
1064:   if (!unpack_output) {
1065:     it++;
1066:   }
1067:   for (it++; it != owning_block->nodes().end(); ++it) {
1068:     torch::jit::Node* node = *it;
1069:     auto* clone_node =
1070:         subgraph->insertNode(subgraph->createClone(node, value_map_func));
1071:     for (size_t i = 0; i < node->outputs().size(); ++i) {
1072:       value_map[node->outputs()[i]] = clone_node->outputs()[i];
1073:       auto trace_it = std::find(
1074:           trace_outputs.begin(), trace_outputs.end(), node->outputs()[i]);
1075:       if (trace_it != trace_outputs.end()) {
1076:         subgraph->registerOutput(clone_node->outputs()[i]);
1077:       }
1078:     }
1079:   }
1080: }
1081: 
1082: torch::jit::Node* _trace_pre_record(
1083:     PyObject* op_obj,
1084:     PyObject* input_objects,
1085:     const variable_list& input_vars) {
1086:   if (!jit::tracer::isTracing()) {
1087:     return nullptr;
1088:   }
```

- EN: The main execution path in this span is carried by `find`, `_trace_pre_record`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `find`, `_trace_pre_record` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 1089-1120

```cpp
1089: 
1090:   // Save scalar args and the calling convention
1091:   auto num_args = PyTuple_GET_SIZE(input_objects);
1092:   pyobj_list scalar_args;
1093:   std::string arg_types;
1094:   arg_types.reserve(num_args);
1095:   scalar_args.reserve(num_args);
1096:   for (const auto i : c10::irange(num_args)) {
1097:     PyObject* arg_object = PyTuple_GET_ITEM(input_objects, i);
1098:     if (THPVariable_Check(arg_object)) {
1099:       arg_types.push_back('d');
1100:     } else {
1101:       arg_types.push_back('c');
1102:       Py_INCREF(arg_object);
1103:       scalar_args.emplace_back(arg_object);
1104:     }
1105:   }
1106: 
1107:   Py_INCREF(op_obj);
1108:   auto pyobj = THPObjectPtr(op_obj);
1109:   return jit::tracer::preRecordPythonTrace(
1110:       std::move(pyobj), arg_types, input_vars, std::move(scalar_args));
1111: }
1112: 
1113: void _trace_post_record(
1114:     torch::jit::Node* node,
1115:     PyObject* op_obj,
1116:     const variable_list& input_vars,
1117:     PyObject* output_objects,
1118:     bool is_inplace,
1119:     bool unpack_output) {
1120:   if (!jit::tracer::isTracing()) {
```

- EN: The main execution path in this span is carried by `PyTuple_GET_SIZE`, `PyTuple_GET_ITEM`, `Py_INCREF`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `PyTuple_GET_SIZE`, `PyTuple_GET_ITEM`, `Py_INCREF` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 1121-1152

```cpp
1121:     return;
1122:   }
1123: 
1124:   node->i_(jit::attr::inplace, is_inplace);
1125:   if (PyObject* module_name = PyDict_GetItemString(
1126:           ((PyTypeObject*)op_obj)->tp_dict, "__module__")) {
1127:     if (auto ptr = PyUnicode_AsUTF8(module_name)) {
1128:       node->s_(jit::attr::module, std::string(ptr));
1129:     }
1130:   }
1131: 
1132:   // Isolate C variable ptrs in a vector
1133:   int num_outputs = PyTuple_GET_SIZE(output_objects);
1134:   auto graph = node->owningGraph();
1135:   node->addOutput();
1136:   auto old_node = node;
1137:   if (!unpack_output) {
1138:     std::vector<at::TypePtr> tuple_values(num_outputs, at::TensorType::get());
1139:     auto tuple_type = at::TupleType::create(std::move(tuple_values));
1140:     // Original type is tuple of tensors "without" element type and shape.
1141:     // The missed parts will be added below.
1142:     node->output()->setType(std::move(tuple_type));
1143:     auto unpacked = graph->createTupleUnpack(node->output())->insertAfter(node);
1144:     node = unpacked;
1145:   }
1146: 
1147:   std::vector<torch::jit::Value*> trace_outputs;
1148:   trace_outputs.reserve(static_cast<size_t>(std::max(0, num_outputs)));
1149:   for (const auto i : c10::irange(num_outputs)) {
1150:     PyObject* obj = PyTuple_GET_ITEM(output_objects, i);
1151:     if (THPVariable_Check(obj)) {
1152:       auto value = node->outputs()[i];
```

- EN: The main execution path in this span is carried by `PyTuple_GET_SIZE`, `tuple_values`, `create`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `PyTuple_GET_SIZE`, `tuple_values`, `create` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 1153-1184

```cpp
1153:       const auto& tensor = THPVariable_Unpack(obj);
1154:       if (tensor.defined()) {
1155:         value->inferTypeFrom(tensor);
1156:         trace_outputs.push_back(jit::tracer::getValueTrace(tensor));
1157:         jit::tracer::setValueTrace(tensor, value);
1158:       }
1159:     }
1160:   }
1161:   py::object onnx_globals =
1162:       py::module::import("torch.onnx._internal.torchscript_exporter._globals");
1163:   py::bool_ is_in_onnx_export =
1164:       py::module::import("torch.onnx.__init__").attr("is_in_onnx_export");
1165:   py::bool_ is_autograd_inlining_enabled =
1166:       py::cast<bool>(onnx_globals.attr("GLOBALS").attr("autograd_inlining"));
1167: 
1168:   if (py::cast<bool>(is_in_onnx_export) &&
1169:       py::cast<bool>(is_autograd_inlining_enabled)) {
1170:     _append_subgraph(old_node, graph, std::move(trace_outputs), unpack_output);
1171:   }
1172: 
1173:   // If TupleUnpack operator is created, we copy its output type back
1174:   // to the original tuple type.
1175:   if (!unpack_output) {
1176:     std::vector<at::TypePtr> new_tuple_values;
1177:     new_tuple_values.reserve(num_outputs);
1178:     for (const auto i : c10::irange(num_outputs)) {
1179:       auto ptr = node->outputs()[i]->type();
1180:       new_tuple_values.push_back(std::move(ptr));
1181:     }
1182:     auto tuple_type = at::TupleType::create(std::move(new_tuple_values));
1183:     // The i-th tuple element receives a new tensor type with element type and
1184:     // shape.
```

- EN: The main execution path in this span is carried by `THPVariable_Unpack`, `setValueTrace`, `import`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_Unpack`, `setValueTrace`, `import` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 1185-1216

```cpp
1185:     old_node->output()->setType(std::move(tuple_type));
1186:   }
1187: }
1188: 
1189: PyObject* process_outputs(
1190:     PyObject* op_obj,
1191:     const c10::intrusive_ptr<PyNode>& cdata,
1192:     THPFunction* grad_fn,
1193:     const UnpackedInput& unpacked,
1194:     PyObject* inputs,
1195:     // NOLINTNEXTLINE(cppcoreguidelines-rvalue-reference-param-not-moved)
1196:     THPObjectPtr&& raw_output,
1197:     bool is_executable,
1198:     torch::jit::Node* node,
1199:     bool overridden_setup_context) {
1200:   bool unpack_output = ensure_tuple(raw_output);
1201: 
1202:   auto num_outputs = PyTuple_GET_SIZE(raw_output.get());
1203: 
1204:   THPObjectPtr outputs(PyTuple_New(num_outputs));
1205:   if (!outputs)
1206:     throw python_error();
1207: 
1208:   cdata->clear_input_metadata();
1209: 
1210:   // Record type, device, and size information about inputs
1211:   if (is_executable) {
1212:     grad_fn->input_info.clear();
1213:     grad_fn->input_info.reserve(unpacked.input_vars.size());
1214:     for (auto& var : unpacked.input_vars) {
1215:       grad_fn->input_info.emplace_back(var);
1216:     }
```

- EN: The main execution path in this span is carried by `process_outputs`, `NOLINTNEXTLINE`, `ensure_tuple`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `process_outputs`, `NOLINTNEXTLINE`, `ensure_tuple` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 1217-1248

```cpp
1217:   }
1218: 
1219:   std::unordered_set<at::TensorImpl*> to_save_if_setup_context{};
1220:   std::vector<std::optional<at::Tensor>> tensors_to_save{};
1221:   _get_tensors_to_save(
1222:       grad_fn,
1223:       to_save_if_setup_context,
1224:       tensors_to_save,
1225:       overridden_setup_context,
1226:       is_executable);
1227: 
1228:   bool is_inplace = static_cast<bool>(grad_fn->dirty_tensors);
1229:   _wrap_outputs(
1230:       cdata,
1231:       grad_fn,
1232:       unpacked.input_vars,
1233:       raw_output,
1234:       outputs,
1235:       is_executable,
1236:       to_save_if_setup_context);
1237:   _trace_post_record(
1238:       node, op_obj, unpacked.input_vars, outputs, is_inplace, unpack_output);
1239: 
1240:   // It is important that creating the SavedVariables happen after the output
1241:   // wrapping as the outputs must have their grad_fn/fw_grad properly set before
1242:   // we save them.
1243:   if (is_executable) {
1244:     _save_variables(
1245:         tensors_to_save, cdata, grad_fn, outputs.get(), num_outputs);
1246:   } else {
1247:     // Remove unnecessary attributes
1248:     Py_CLEAR(grad_fn->to_save);
```

- EN: The main execution path in this span is carried by `_get_tensors_to_save`, `_wrap_outputs`, `_trace_post_record`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here.
- CN: 这一段的主要执行路径由 `_get_tensors_to_save`, `_wrap_outputs`, `_trace_post_record` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。
### Lines 1249-1280

```cpp
1249:     Py_CLEAR(grad_fn->non_differentiable);
1250:   }
1251: 
1252:   Py_CLEAR(grad_fn->saved_for_forward);
1253: 
1254:   // Unpack the output, unless .forward() returned a tuple
1255:   if (unpack_output) {
1256:     PyObject* output = PyTuple_GET_ITEM(outputs.get(), 0);
1257:     Py_INCREF(output);
1258:     return output;
1259:   }
1260: 
1261:   return outputs.release();
1262: }
1263: 
1264: PyObject* THPFunction_name(PyObject* self, PyObject* noargs) {
1265:   HANDLE_TH_ERRORS
1266:   auto cdata = ((THPFunction*)self)->cdata.lock();
1267:   check_legacy_fn_attr_access(cdata, "name");
1268:   return THPUtils_packString(cdata->name());
1269:   END_HANDLE_TH_ERRORS
1270: }
1271: 
1272: PyObject* THPFunction_sequence_nr(PyObject* self, PyObject* noargs) {
1273:   HANDLE_TH_ERRORS;
1274:   auto cdata = ((THPFunction*)self)->cdata.lock();
1275:   check_legacy_fn_attr_access(cdata, "_sequence_nr");
1276:   return THPUtils_packUInt64(cdata->sequence_nr());
1277:   END_HANDLE_TH_ERRORS
1278: }
1279: 
1280: PyObject* THPFunction_set_sequence_nr(PyObject* self, PyObject* sequence_nr) {
```

- EN: The main execution path in this span is carried by `Py_CLEAR`, `PyTuple_GET_ITEM`, `Py_INCREF`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `Py_CLEAR`, `PyTuple_GET_ITEM`, `Py_INCREF` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 1281-1312

```cpp
1281:   HANDLE_TH_ERRORS;
1282:   auto cdata = ((THPFunction*)self)->cdata.lock();
1283:   check_legacy_fn_attr_access(cdata, "_set_sequence_nr");
1284:   cdata->set_sequence_nr(THPUtils_unpackUInt64(sequence_nr));
1285:   Py_RETURN_NONE;
1286:   END_HANDLE_TH_ERRORS
1287: }
1288: 
1289: PyObject* THPFunction_input_metadata(PyObject* self, void* unused) {
1290:   HANDLE_TH_ERRORS;
1291:   auto cdata = ((THPFunction*)self)->cdata.lock();
1292:   check_legacy_fn_attr_access(cdata, "_input_metadata");
1293:   const auto num_inputs = cdata->num_inputs();
1294:   THPObjectPtr list(PyTuple_New(num_inputs));
1295:   if (!list) {
1296:     return nullptr;
1297:   }
1298:   for (size_t i = 0; i < num_inputs; ++i) {
1299:     const auto& metadata = cdata->input_metadata(i);
1300:     THPObjectPtr item(py::cast(metadata).release().ptr());
1301:     if (!item) {
1302:       return nullptr;
1303:     }
1304:     PyTuple_SET_ITEM(list.get(), i, item.release());
1305:   }
1306:   return list.release();
1307:   END_HANDLE_TH_ERRORS
1308: }
1309: 
1310: PyObject* THPFunction_maybe_clear_saved_tensors(
1311:     PyObject* self,
1312:     PyObject* noargs) {
```

- EN: The main execution path in this span is carried by `check_legacy_fn_attr_access`, `THPFunction_input_metadata`, `list`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `check_legacy_fn_attr_access`, `THPFunction_input_metadata`, `list` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1313-1344

```cpp
1313:   HANDLE_TH_ERRORS;
1314:   auto cdata = ((THPFunction*)self)->cdata.lock();
1315:   if (!get_current_graph_task_keep_graph()) {
1316:     cdata->release_variables();
1317:   }
1318:   Py_RETURN_NONE;
1319:   END_HANDLE_TH_ERRORS
1320: }
1321: 
1322: THPObjectPtr make_ctx_input_tuple(
1323:     THPFunction* ctx,
1324:     const UnpackedInput& unpacked_input,
1325:     int64_t num_args) {
1326:   THPObjectPtr ctx_input_tuple(PyTuple_New(num_args + 1));
1327:   if (!ctx_input_tuple)
1328:     return {};
1329:   Py_INCREF(ctx);
1330:   PyTuple_SET_ITEM(ctx_input_tuple.get(), 0, (PyObject*)ctx);
1331:   for (const auto i : c10::irange(num_args)) {
1332:     PyObject* arg = PyTuple_GET_ITEM(unpacked_input.input_tuple.get(), i);
1333:     Py_INCREF(arg);
1334:     PyTuple_SET_ITEM(ctx_input_tuple.get(), i + 1, arg);
1335:   }
1336:   return ctx_input_tuple;
1337: }
1338: 
1339: THPObjectPtr make_ctx_input_output_tuple(
1340:     THPFunction* ctx,
1341:     UnpackedInput& unpacked_input,
1342:     PyObject* output) {
1343:   THPObjectPtr result(PyTuple_New(3));
1344:   if (!result)
```

- EN: The main execution path in this span is carried by `make_ctx_input_tuple`, `ctx_input_tuple`, `Py_INCREF`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `make_ctx_input_tuple`, `ctx_input_tuple`, `Py_INCREF` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1345-1376

```cpp
1345:     return {};
1346:   Py_INCREF(ctx);
1347:   Py_INCREF(unpacked_input.input_tuple.get());
1348:   Py_INCREF(output);
1349:   PyTuple_SET_ITEM(result.get(), 0, (PyObject*)ctx);
1350:   PyTuple_SET_ITEM(result.get(), 1, unpacked_input.input_tuple.get());
1351:   PyTuple_SET_ITEM(result.get(), 2, output);
1352:   return result;
1353: }
1354: 
1355: static PyObject* get_base_setup_context() {
1356:   // NOTE: THPFunction_setup_context is intentionally leaked and never freed.
1357:   static std::atomic<PyObject*> THPFunction_setup_context = nullptr;
1358: 
1359:   PyObject* setup_context =
1360:       THPFunction_setup_context.load(std::memory_order_acquire);
1361:   if (setup_context != nullptr) {
1362:     return setup_context;
1363:   }
1364: 
1365:   auto module = THPObjectPtr(PyImport_ImportModule("torch.autograd.function"));
1366:   if (!module)
1367:     return nullptr;
1368: 
1369:   auto function =
1370:       THPObjectPtr(PyObject_GetAttrString(module, "_SingleLevelFunction"));
1371:   if (!function)
1372:     return nullptr;
1373: 
1374:   // setup_context gets "leaked" - we return a new reference and hold onto it
1375:   // forever.
1376:   setup_context = PyObject_GetAttrString(function, "setup_context");
```

- EN: The main execution path in this span is carried by `Py_INCREF`, `PyTuple_SET_ITEM`, `get_base_setup_context`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `Py_INCREF`, `PyTuple_SET_ITEM`, `get_base_setup_context` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1377-1408

```cpp
1377:   if (!setup_context)
1378:     return nullptr;
1379: 
1380:   PyObject* expected = nullptr;
1381:   if (!THPFunction_setup_context.compare_exchange_strong(
1382:           expected, setup_context, std::memory_order_acq_rel)) {
1383:     Py_DECREF(setup_context);
1384:     return expected;
1385:   }
1386:   return setup_context;
1387: }
1388: 
1389: PyObject* THPFunction_apply(PyObject* cls, PyObject* inputs) {
1390:   HANDLE_TH_ERRORS
1391: 
1392:   // save a local copy of seq_id before it gets incremented
1393:   auto seq_id = at::sequence_number::peek();
1394:   auto info_pair = unpack_input<false>(inputs);
1395:   UnpackedInput& unpacked_input = info_pair.first;
1396:   InputFlags& input_info = info_pair.second;
1397: 
1398:   // Call record function after all the inputs have been decoded, but
1399:   // before context has been allocated.
1400:   RECORD_FUNCTION(
1401:       ((PyTypeObject*)cls)->tp_name,
1402:       unpacked_input.record_function_inputs,
1403:       seq_id);
1404: 
1405:   const auto& functorch_tls = at::functorch::functorchTLSAccessor();
1406:   if (functorch_tls) {
1407:     // autograd.Function support for functorch is handled in Python.
1408:     // If we have gotten here, then either we are dealing with a
```

- EN: The main execution path in this span is carried by `Py_DECREF`, `THPFunction_apply`, `peek`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `Py_DECREF`, `THPFunction_apply`, `peek` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1409-1440

```cpp
1409:     // torch.autograd.function._SingleLevelFunction, or something in
1410:     // the implementation went wrong.
1411:     // The following code is useful for debugging when something goes wrong
1412:     // because it'll raise a loud error (instead of being silently incorrect).
1413:     functorch_tls->checkSupportsSingleLevelAutogradFunction();
1414:   }
1415: 
1416:   THPObjectPtr backward_cls(PyObject_GetAttrString(cls, "_backward_cls"));
1417:   if (!backward_cls)
1418:     return nullptr;
1419:   THPObjectPtr ctx_obj(PyObject_CallFunctionObjArgs(backward_cls, nullptr));
1420:   if (!ctx_obj)
1421:     return nullptr;
1422:   THPFunction* ctx = (THPFunction*)ctx_obj.get();
1423: 
1424:   auto cdata = c10::make_intrusive<PyNode>(std::move(ctx_obj));
1425:   ctx->cdata = cdata;
1426: 
1427:   // Record input nodes if tracing
1428:   auto* node = _trace_pre_record(cls, inputs, unpacked_input.input_vars);
1429: 
1430:   // Initialize backward function (and ctx)
1431:   bool is_executable = input_info.is_executable;
1432:   cdata->set_next_edges(std::move(input_info.next_edges));
1433:   ctx->needs_input_grad = input_info.needs_input_grad.release();
1434:   ctx->is_variable_input = std::move(input_info.is_variable_input);
1435: 
1436:   // Get clear_saved_tensors_on_access from the Function class
1437:   THPObjectPtr clear_attr(
1438:       PyObject_GetAttrString(cls, "clear_saved_tensors_on_access"));
1439:   TORCH_CHECK(
1440:       clear_attr,
```

- EN: The main execution path in this span is carried by `error`, `backward_cls`, `ctx_obj`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here.
- CN: 这一段的主要执行路径由 `error`, `backward_cls`, `ctx_obj` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。
### Lines 1441-1472

```cpp
1441:       "autograd.Function is missing clear_saved_tensors_on_access attribute");
1442:   TORCH_CHECK(
1443:       PyBool_Check(clear_attr.get()),
1444:       "clear_saved_tensors_on_access must be a bool, got ",
1445:       Py_TYPE(clear_attr.get())->tp_name);
1446:   ctx->clear_saved_tensors_on_access = Py_IsTrue(clear_attr.get());
1447: 
1448:   // Get boxed_grads_call from the Function class
1449:   THPObjectPtr boxed_attr(PyObject_GetAttrString(cls, "boxed_grads_call"));
1450:   TORCH_CHECK(
1451:       boxed_attr, "autograd.Function is missing boxed_grads_call attribute");
1452:   TORCH_CHECK(
1453:       PyBool_Check(boxed_attr.get()),
1454:       "boxed_grads_call must be a bool, got ",
1455:       Py_TYPE(boxed_attr.get())->tp_name);
1456:   ctx->boxed_grads_call = Py_IsTrue(boxed_attr.get());
1457: 
1458:   // autograd.Function may optionally override a setup_context staticmethod.
1459:   // In this case, autograd.Function.forward does NOT accept a ctx object.
1460:   // Determine if this is the case.
1461:   auto cls_setup_context =
1462:       THPObjectPtr(PyObject_GetAttrString(cls, "setup_context"));
1463:   if (!cls_setup_context) {
1464:     return nullptr;
1465:   }
1466:   auto orig_setup_context = get_base_setup_context();
1467:   if (!orig_setup_context) {
1468:     return nullptr;
1469:   }
1470:   auto overridden_setup_context = cls_setup_context.get() != orig_setup_context;
1471: 
1472:   auto num_args = PyTuple_GET_SIZE(inputs);
```

- EN: The main execution path in this span is carried by `TORCH_CHECK`, `PyBool_Check`, `Py_TYPE`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `TORCH_CHECK`, `PyBool_Check`, `Py_TYPE` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 1473-1504

```cpp
1473: 
1474:   // Call forward
1475:   THPObjectPtr output;
1476:   {
1477:     AutoGradMode grad_mode(false);
1478:     at::AutoFwGradMode fw_grad_mode(false);
1479:     THPObjectPtr forward_fn(PyObject_GetAttrString(cls, "forward"));
1480:     if (!forward_fn)
1481:       return nullptr;
1482:     if (overridden_setup_context) {
1483:       // call forward followed by setup_context
1484:       output = PyObject_CallObject(forward_fn, unpacked_input.input_tuple);
1485:       if (!output) {
1486:         return nullptr;
1487:       }
1488:       // signature is setup_context(ctx, inputs, output)
1489:       auto ctx_input_output_tuple =
1490:           make_ctx_input_output_tuple(ctx, unpacked_input, output);
1491:       if (!ctx_input_output_tuple) {
1492:         return nullptr;
1493:       }
1494:       THPObjectPtr setup_context_fn(
1495:           PyObject_GetAttrString(cls, "setup_context"));
1496:       auto result =
1497:           PyObject_CallObject(setup_context_fn, ctx_input_output_tuple);
1498:       if (!result) {
1499:         return nullptr;
1500:       }
1501:     } else {
1502:       // call forward
1503:       auto ctx_input_tuple =
1504:           make_ctx_input_tuple(ctx, unpacked_input, num_args);
```

- EN: The main execution path in this span is carried by `grad_mode`, `fw_grad_mode`, `forward_fn`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `grad_mode`, `fw_grad_mode`, `forward_fn` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1505-1536

```cpp
1505:       if (!ctx_input_tuple) {
1506:         return nullptr;
1507:       }
1508:       output = PyObject_CallObject(forward_fn, ctx_input_tuple);
1509:     }
1510:     if (!output)
1511:       return nullptr;
1512:   }
1513: 
1514:   return process_outputs(
1515:       cls,
1516:       cdata,
1517:       ctx,
1518:       unpacked_input,
1519:       inputs,
1520:       std::move(output),
1521:       is_executable,
1522:       node,
1523:       overridden_setup_context);
1524:   END_HANDLE_TH_ERRORS
1525: }
1526: 
1527: ////////////////////////////////////////////////////////////////////////////////
1528: // Other methods / attributes
1529: ////////////////////////////////////////////////////////////////////////////////
1530: 
1531: PyObject* THPFunction__register_hook_dict(PyObject* _self, PyObject* _var) {
1532:   HANDLE_TH_ERRORS
1533:   TORCH_CHECK(THPVariable_Check(_var), "_register_hook_dict expected a Tensor");
1534:   THPVariable* var = reinterpret_cast<THPVariable*>(_var);
1535:   const auto& tensor = THPVariable_Unpack(var);
1536:   std::unique_ptr<FunctionPreHook> hook(
```

- EN: The main execution path in this span is carried by `PyObject_CallObject`, `process_outputs`, `move`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `PyObject_CallObject`, `process_outputs`, `move` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 1537-1568

```cpp
1537:       new PyFunctionTensorPreHook(var->backward_hooks, tensor.output_nr()));
1538:   auto self = (THPFunction*)_self;
1539:   auto cdata = self->cdata.lock();
1540:   check_legacy_fn_attr_access(cdata, "_register_hook_dict");
1541:   cdata->add_tensor_pre_hook(std::move(hook));
1542:   Py_RETURN_NONE;
1543:   END_HANDLE_TH_ERRORS
1544: }
1545: 
1546: PyObject* THPFunction_register_hook(PyObject* _self, PyObject* hook) {
1547:   HANDLE_TH_ERRORS
1548:   auto self = (THPFunction*)_self;
1549:   auto cdata = self->cdata.lock();
1550:   check_legacy_fn_attr_access(cdata, "register_hook");
1551:   return torch::autograd::registerFunctionHook(*cdata, hook);
1552:   END_HANDLE_TH_ERRORS
1553: }
1554: 
1555: PyObject* THPFunction_register_prehook(PyObject* _self, PyObject* hook) {
1556:   HANDLE_TH_ERRORS
1557:   auto self = (THPFunction*)_self;
1558:   auto cdata = self->cdata.lock();
1559:   check_legacy_fn_attr_access(cdata, "register_prehook");
1560:   return torch::autograd::registerFunctionPreHook(*cdata, hook);
1561:   END_HANDLE_TH_ERRORS
1562: }
1563: 
1564: int THPFunction_set_materialize_grads(
1565:     THPFunction* self,
1566:     PyObject* value,
1567:     void* unused) {
1568:   HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `PyFunctionTensorPreHook`, `check_legacy_fn_attr_access`, `THPFunction_register_hook`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyFunctionTensorPreHook`, `check_legacy_fn_attr_access`, `THPFunction_register_hook` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1569-1600

```cpp
1569:   if (!PyBool_Check(value)) {
1570:     THPUtils_invalidArguments(
1571:         value, nullptr, "set_materialize_grads", 1, "(bool)");
1572:     return -1;
1573:   }
1574:   self->materialize_grads = (Py_IsTrue(value));
1575:   return 0;
1576:   END_HANDLE_TH_ERRORS_RET(-1)
1577: }
1578: 
1579: int THPFunction_set_pure_view(
1580:     THPFunction* self,
1581:     PyObject* value,
1582:     void* unused) {
1583:   HANDLE_TH_ERRORS
1584:   if (!PyBool_Check(value)) {
1585:     THPUtils_invalidArguments(value, nullptr, "set_pure_view", 1, "(bool)");
1586:     return -1;
1587:   }
1588:   self->pure_view = (Py_IsTrue(value));
1589:   return 0;
1590:   END_HANDLE_TH_ERRORS_RET(-1)
1591: }
1592: 
1593: PyObject* THPFunction_get_materialize_non_diff_grads(
1594:     THPFunction* self,
1595:     void* _unused) {
1596:   HANDLE_TH_ERRORS
1597:   if (self->materialize_non_diff_grads) {
1598:     Py_RETURN_TRUE;
1599:   } else {
1600:     Py_RETURN_FALSE;
```

- EN: The main execution path in this span is carried by `THPUtils_invalidArguments`, `END_HANDLE_TH_ERRORS_RET`, `THPFunction_set_pure_view`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPUtils_invalidArguments`, `END_HANDLE_TH_ERRORS_RET`, `THPFunction_set_pure_view` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1601-1632

```cpp
1601:   }
1602:   END_HANDLE_TH_ERRORS
1603: }
1604: 
1605: int THPFunction_set_materialize_non_diff_grads(
1606:     THPFunction* self,
1607:     PyObject* value,
1608:     void* unused) {
1609:   HANDLE_TH_ERRORS
1610:   if (!PyBool_Check(value)) {
1611:     THPUtils_invalidArguments(
1612:         value, nullptr, "set_materialize_non_diff_grads", 1, "(bool)");
1613:     return -1;
1614:   }
1615:   self->materialize_non_diff_grads = (Py_IsTrue(value));
1616:   return 0;
1617:   END_HANDLE_TH_ERRORS_RET(-1)
1618: }
1619: 
1620: PyObject* THPFunction_saved_tensors(THPFunction* self, void* _unused) {
1621:   HANDLE_TH_ERRORS
1622:   TORCH_CHECK(
1623:       !self->saved_tensors_accessed_and_cleared,
1624:       "saved_tensors can only be accessed once when "
1625:       "clear_saved_tensors_on_access=True is set on the autograd.Function. "
1626:       "Either access saved_tensors only once, or set "
1627:       "clear_saved_tensors_on_access=False.");
1628:   if (self->saved_for_forward) {
1629:     Py_INCREF(self->saved_for_forward);
1630:     return self->saved_for_forward;
1631:   } else {
1632:     PyObject* result = unpack_saved_variables(
```

- EN: The main execution path in this span is carried by `THPFunction_set_materialize_non_diff_grads`, `THPUtils_invalidArguments`, `END_HANDLE_TH_ERRORS_RET`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPFunction_set_materialize_non_diff_grads`, `THPUtils_invalidArguments`, `END_HANDLE_TH_ERRORS_RET` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 1633-1664

```cpp
1633:         self, [](const Variable& var) { return THPVariable_Wrap(var); });
1634:     if (!result) {
1635:       return nullptr;
1636:     }
1637: 
1638:     if (self->clear_saved_tensors_on_access) {
1639:       self->saved_variables.clear();
1640:       self->saved_tensors_accessed_and_cleared = true;
1641:     }
1642: 
1643:     return result;
1644:   }
1645:   END_HANDLE_TH_ERRORS
1646: }
1647: 
1648: PyObject* THPFunction_saved_variables(THPFunction* self, void* _unused) {
1649:   HANDLE_TH_ERRORS
1650:   auto r = PyErr_WarnEx(
1651:       PyExc_DeprecationWarning,
1652:       "'saved_variables' is deprecated; use 'saved_tensors'",
1653:       0);
1654:   if (r != 0)
1655:     throw python_error();
1656:   TORCH_CHECK(
1657:       !self->saved_tensors_accessed_and_cleared,
1658:       "saved_tensors can only be accessed once when "
1659:       "clear_saved_tensors_on_access=True is set on the autograd.Function. "
1660:       "Either access saved_tensors only once, or set "
1661:       "clear_saved_tensors_on_access=False.");
1662:   PyObject* result = unpack_saved_variables(
1663:       self, [](const Variable& var) { return THPVariable_Wrap(var); });
1664:   if (!result) {
```

- EN: The main execution path in this span is carried by `THPFunction_saved_variables`, `PyErr_WarnEx`, `python_error`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPFunction_saved_variables`, `PyErr_WarnEx`, `python_error` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 1665-1696

```cpp
1665:     return nullptr;
1666:   }
1667: 
1668:   if (self->clear_saved_tensors_on_access) {
1669:     self->saved_variables.clear();
1670:     self->saved_tensors_accessed_and_cleared = true;
1671:   }
1672: 
1673:   return result;
1674:   END_HANDLE_TH_ERRORS
1675: }
1676: 
1677: PyObject* THPFunction_get_compiled_autograd_symints(
1678:     PyObject* _self,
1679:     PyObject* _unused) {
1680:   HANDLE_TH_ERRORS
1681:   auto self = (THPFunction*)_self;
1682:   auto size = self->compiled_autograd_symints.size();
1683:   PyObject* result = PyTuple_New(static_cast<Py_ssize_t>(size));
1684:   if (!result) {
1685:     throw python_error();
1686:   }
1687:   for (const auto i : c10::irange(size)) {
1688:     PyTuple_SET_ITEM(
1689:         result,
1690:         i,
1691:         py::cast(self->compiled_autograd_symints[i]).release().ptr());
1692:   }
1693:   return result;
1694:   END_HANDLE_TH_ERRORS
1695: }
1696: 
```

- EN: The main execution path in this span is carried by `THPFunction_get_compiled_autograd_symints`, `PyTuple_New`, `python_error`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPFunction_get_compiled_autograd_symints`, `PyTuple_New`, `python_error` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1697-1728

```cpp
1697: PyObject* THPFunction_get_compiled_autograd_backward_state(
1698:     PyObject* _self,
1699:     void* _unused) {
1700:   HANDLE_TH_ERRORS
1701:   auto self = (THPFunction*)_self;
1702:   PyObject* bw_state = self->compiled_autograd_backward_state;
1703:   if (bw_state == nullptr) {
1704:     bw_state = Py_None;
1705:   }
1706:   Py_INCREF(bw_state);
1707:   return bw_state;
1708:   END_HANDLE_TH_ERRORS
1709: }
1710: 
1711: int THPFunction_set_compiled_autograd_backward_state(
1712:     PyObject* _self,
1713:     PyObject* bw_state,
1714:     void* _unused) {
1715:   HANDLE_TH_ERRORS
1716:   auto self = (THPFunction*)_self;
1717:   TORCH_INTERNAL_ASSERT(self->compiled_autograd_backward_state == nullptr);
1718:   Py_INCREF(bw_state);
1719:   self->compiled_autograd_backward_state = bw_state;
1720:   return 0;
1721:   END_HANDLE_TH_ERRORS_RET(-1)
1722: }
1723: 
1724: PyObject* THPFunction_raw_saved_tensors(THPFunction* self, void* _unused) {
1725:   HANDLE_TH_ERRORS
1726:   // User tries to access saved variables after they have been freed
1727:   TORCH_CHECK(!self->has_freed_buffers, ERR_BACKWARD_TWICE);
1728:   const auto& saved_variables = self->saved_variables;
```

- EN: The main execution path in this span is carried by `THPFunction_get_compiled_autograd_backward_state`, `Py_INCREF`, `THPFunction_set_compiled_autograd_backward_state`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPFunction_get_compiled_autograd_backward_state`, `Py_INCREF`, `THPFunction_set_compiled_autograd_backward_state` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 1729-1760

```cpp
1729:   if (saved_variables.empty())
1730:     return PyTuple_New(0);
1731:   size_t num_saved = saved_variables.size();
1732:   THPObjectPtr saved(PyTuple_New(static_cast<Py_ssize_t>(num_saved)));
1733:   if (!saved) {
1734:     return nullptr;
1735:   }
1736:   for (const auto i : c10::irange(num_saved)) {
1737:     py::object obj =
1738:         py::cast(saved_variables[i], py::return_value_policy::reference);
1739:     PyTuple_SET_ITEM(saved.get(), i, obj.release().ptr());
1740:   }
1741:   return saved.release();
1742:   END_HANDLE_TH_ERRORS
1743: }
1744: 
1745: PyObject* THPFunction_next_functions(THPFunction* self, void* _unused) {
1746:   HANDLE_TH_ERRORS
1747:   auto cdata = self->cdata.lock();
1748:   check_legacy_fn_attr_access(cdata, "next_functions");
1749:   const auto num_outputs = cdata->num_outputs();
1750:   THPObjectPtr result(PyTuple_New(num_outputs));
1751:   if (!result)
1752:     return nullptr;
1753:   for (const auto i : c10::irange(num_outputs)) {
1754:     THPObjectPtr fn_tuple(PyTuple_New(2));
1755:     if (!fn_tuple)
1756:       return nullptr;
1757:     const auto& edge = cdata->next_edge(i);
1758:     PyObject* fn = functionToPyObject(edge.function);
1759:     if (!fn)
1760:       return nullptr;
```

- EN: The main execution path in this span is carried by `PyTuple_New`, `saved`, `cast`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `PyTuple_New`, `saved`, `cast` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1761-1792

```cpp
1761:     PyTuple_SET_ITEM(fn_tuple.get(), 0, fn);
1762:     PyTuple_SET_ITEM(fn_tuple.get(), 1, THPUtils_packInt64(edge.input_nr));
1763:     PyTuple_SET_ITEM(result.get(), i, fn_tuple.release());
1764:   }
1765:   return result.release();
1766:   END_HANDLE_TH_ERRORS
1767: }
1768: 
1769: PyObject* THPFunction_metadata(THPFunction* self, void* _unused) {
1770:   HANDLE_TH_ERRORS
1771:   auto cdata = self->cdata.lock();
1772:   // The correct way to solve this problem is to stop exposing grad_fn
1773:   // of PyFunctions as THPFunction; instead, we should use THPCppFunction
1774:   // like everyone else.  But this is a BC-breaking change as it would
1775:   // mean that you no longer get the property that grad_fn is a subclass
1776:   // of the autograd function class that you defined in the custom case,
1777:   // so I didn't fix it here.
1778:   TORCH_CHECK(
1779:       cdata,
1780:       "You attempted to access the anomaly metadata of a custom autograd function "
1781:       "but the underlying PyNode has already been deallocated.  The most likely "
1782:       "reason this occurred is because you assigned x.grad_fn to a local variable "
1783:       "and then let the original variable get deallocated.  Don't do that!  If "
1784:       "you really have no way of restructuring your code so this is the case, "
1785:       "please file an issue reporting that you are affected by this.");
1786:   auto metadata = static_cast<PyAnomalyMetadata*>(cdata->metadata())->dict();
1787: 
1788:   Py_INCREF(metadata);
1789:   return metadata;
1790:   END_HANDLE_TH_ERRORS
1791: }
1792: } // namespace
```

- EN: The main execution path in this span is carried by `PyTuple_SET_ITEM`, `THPFunction_metadata`, `TORCH_CHECK`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here.
- CN: 这一段的主要执行路径由 `PyTuple_SET_ITEM`, `THPFunction_metadata`, `TORCH_CHECK` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。
### Lines 1793-1824

```cpp
1793: 
1794: using getter = PyObject* (*)(PyObject*, void*);
1795: using setter = int (*)(PyObject*, PyObject*, void*);
1796: 
1797: namespace {
1798: 
1799: template <PyObject* THPFunction::* ptr>
1800: PyObject* getObject(PyObject* obj, void* _unused) {
1801:   auto self = (THPFunction*)obj;
1802:   PyObject* value = self->*ptr;
1803:   if (!value) {
1804:     Py_RETURN_NONE;
1805:   }
1806:   Py_INCREF(value);
1807:   return value;
1808: }
1809: 
1810: template <PyObject* THPFunction::* ptr>
1811: int setObject(PyObject* obj, PyObject* value, void* _unused) {
1812:   auto self = (THPFunction*)obj;
1813:   if (Py_IsNone(value)) {
1814:     value = nullptr;
1815:   }
1816:   Py_XDECREF((self->*ptr));
1817:   Py_XINCREF(value);
1818:   self->*ptr = value;
1819:   return 0;
1820: }
1821: 
1822: template <typename M, M THPFunction::* ptr, PyObject* (*Convert)(long)>
1823: PyObject* getMember(PyObject* obj, void* _unused) {
1824:   auto self = (THPFunction*)obj;
```

- EN: They also open, refine, or close the relevant C++ namespace scope. The main execution path in this span is carried by `int`, `getObject`, `Py_INCREF`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 它们还会打开、细化或关闭相应的 C++ 命名空间作用域。 这一段的主要执行路径由 `int`, `getObject`, `Py_INCREF` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 1825-1856

```cpp
1825:   return Convert(self->*ptr);
1826: }
1827: 
1828: template <typename M, M autograd::Node::* ptr, PyObject* (*Convert)(long)>
1829: PyObject* getImplMember(PyObject* obj, void* _unused) {
1830:   auto self = (THPFunction*)obj;
1831:   return Convert(self->cdata.*ptr);
1832: }
1833: 
1834: PyObject* getRequiresGrad(PyObject* obj, void* _unused) {
1835:   Py_RETURN_TRUE;
1836: }
1837: 
1838: } // namespace
1839: 
1840: // NOLINTNEXTLINE(modernize-avoid-c-arrays,cppcoreguidelines-avoid-c-arrays,cppcoreguidelines-avoid-non-const-global-variables)
1841: static struct PyGetSetDef THPFunction_properties[] = {
1842:     {"saved_tensors",
1843:      (getter)THPFunction_saved_tensors,
1844:      nullptr,
1845:      nullptr,
1846:      nullptr},
1847:     {"saved_variables",
1848:      (getter)THPFunction_saved_variables,
1849:      nullptr,
1850:      nullptr,
1851:      nullptr},
1852:     {"_raw_saved_tensors",
1853:      (getter)THPFunction_raw_saved_tensors,
1854:      nullptr,
1855:      nullptr,
1856:      nullptr},
```

- EN: The main execution path in this span is carried by `Convert`, `getImplMember`, `getRequiresGrad`. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `Convert`, `getImplMember`, `getRequiresGrad` 等函数/方法承载。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1857-1888

```cpp
1857:     {"next_functions",
1858:      (getter)THPFunction_next_functions,
1859:      nullptr,
1860:      nullptr,
1861:      nullptr},
1862:     {"to_save",
1863:      &getObject<&THPFunction::to_save>,
1864:      &setObject<&THPFunction::to_save>,
1865:      nullptr,
1866:      nullptr},
1867:     {"non_differentiable",
1868:      &getObject<&THPFunction::non_differentiable>,
1869:      &setObject<&THPFunction::non_differentiable>,
1870:      nullptr,
1871:      nullptr},
1872:     {"dirty_tensors",
1873:      &getObject<&THPFunction::dirty_tensors>,
1874:      &setObject<&THPFunction::dirty_tensors>,
1875:      nullptr,
1876:      nullptr},
1877:     {"saved_for_forward",
1878:      &getObject<&THPFunction::saved_for_forward>,
1879:      &setObject<&THPFunction::saved_for_forward>,
1880:      nullptr,
1881:      nullptr},
1882:     {"needs_input_grad",
1883:      &getObject<&THPFunction::needs_input_grad>,
1884:      &setObject<&THPFunction::needs_input_grad>,
1885:      nullptr,
1886:      nullptr},
1887:     {"requires_grad", getRequiresGrad, nullptr, nullptr, nullptr},
1888:     {"metadata", (getter)THPFunction_metadata, nullptr, nullptr, nullptr},
```

- EN: Autograd-related metadata or graph connectivity is updated here.
- CN: 这里会更新与自动求导相关的元数据或计算图连接关系。
### Lines 1889-1920

```cpp
1889:     {"_input_metadata",
1890:      (getter)THPFunction_input_metadata,
1891:      nullptr,
1892:      nullptr,
1893:      nullptr},
1894:     {"materialize_grads",
1895:      nullptr,
1896:      (setter)THPFunction_set_materialize_grads,
1897:      nullptr,
1898:      nullptr},
1899:     {"_is_pure_view",
1900:      nullptr,
1901:      (setter)THPFunction_set_pure_view,
1902:      nullptr,
1903:      nullptr},
1904:     {"_materialize_non_diff_grads",
1905:      (getter)THPFunction_get_materialize_non_diff_grads,
1906:      (setter)THPFunction_set_materialize_non_diff_grads,
1907:      nullptr,
1908:      nullptr},
1909:     {"_compiled_autograd_backward_state",
1910:      (getter)THPFunction_get_compiled_autograd_backward_state,
1911:      (setter)THPFunction_set_compiled_autograd_backward_state,
1912:      nullptr,
1913:      nullptr},
1914:     {nullptr}};
1915: 
1916: // NOLINTNEXTLINE(modernize-avoid-c-arrays,cppcoreguidelines-avoid-c-arrays,cppcoreguidelines-avoid-non-const-global-variables)
1917: static struct PyMethodDef THPFunction_methods[] = {
1918:     {(char*)"name", THPFunction_name, METH_NOARGS, nullptr},
1919:     {(char*)"_sequence_nr", THPFunction_sequence_nr, METH_NOARGS, nullptr},
1920:     {(char*)"_set_sequence_nr", THPFunction_set_sequence_nr, METH_O, nullptr},
```

- EN: The main execution path in this span is carried by `NOLINTNEXTLINE`.
- CN: 这一段的主要执行路径由 `NOLINTNEXTLINE` 等函数/方法承载。
### Lines 1921-1952

```cpp
1921:     {(char*)"maybe_clear_saved_tensors",
1922:      THPFunction_maybe_clear_saved_tensors,
1923:      METH_NOARGS,
1924:      nullptr},
1925:     {(char*)"apply", THPFunction_apply, METH_CLASS | METH_VARARGS, nullptr},
1926:     {(char*)"_register_hook_dict",
1927:      THPFunction__register_hook_dict,
1928:      METH_O,
1929:      nullptr},
1930:     {(char*)"register_hook", THPFunction_register_hook, METH_O, nullptr},
1931:     {(char*)"register_prehook", THPFunction_register_prehook, METH_O, nullptr},
1932:     {(char*)"_get_compiled_autograd_symints",
1933:      THPFunction_get_compiled_autograd_symints,
1934:      METH_NOARGS,
1935:      nullptr},
1936:     {nullptr}};
1937: 
1938: PyTypeObject THPFunctionType = {
1939:     PyVarObject_HEAD_INIT(nullptr, 0)
1940:     "torch._C._FunctionBase", /* tp_name */
1941:     sizeof(THPFunction), /* tp_basicsize */
1942:     0, /* tp_itemsize */
1943:     (destructor)THPFunction_dealloc, /* tp_dealloc */
1944:     0, /* tp_vectorcall_offset */
1945:     nullptr, /* tp_getattr */
1946:     nullptr, /* tp_setattr */
1947:     nullptr, /* tp_reserved */
1948:     nullptr, /* tp_repr */
1949:     nullptr, /* tp_as_number */
1950:     nullptr, /* tp_as_sequence */
1951:     nullptr, /* tp_as_mapping */
1952:     nullptr, /* tp_hash  */
```

- EN: The main execution path in this span is carried by `PyVarObject_HEAD_INIT`.
- CN: 这一段的主要执行路径由 `PyVarObject_HEAD_INIT` 等函数/方法承载。
### Lines 1953-1984

```cpp
1953:     nullptr, /* tp_call */
1954:     nullptr, /* tp_str */
1955:     nullptr, /* tp_getattro */
1956:     nullptr, /* tp_setattro */
1957:     nullptr, /* tp_as_buffer */
1958:     // NOLINTNEXTLINE(misc-redundant-expression)
1959:     Py_TPFLAGS_DEFAULT | Py_TPFLAGS_BASETYPE |
1960:         Py_TPFLAGS_HAVE_GC, /* tp_flags */
1961:     nullptr, /* tp_doc */
1962:     (traverseproc)THPFunction_traverse, /* tp_traverse */
1963:     (inquiry)THPFunction_clear, /* tp_clear */
1964:     nullptr, /* tp_richcompare */
1965:     0, /* tp_weaklistoffset */
1966:     nullptr, /* tp_iter */
1967:     nullptr, /* tp_iternext */
1968:     THPFunction_methods, /* tp_methods */
1969:     nullptr, /* tp_members */
1970:     THPFunction_properties, /* tp_getset */
1971:     nullptr, /* tp_base */
1972:     nullptr, /* tp_dict */
1973:     nullptr, /* tp_descr_get */
1974:     nullptr, /* tp_descr_set */
1975:     0, /* tp_dictoffset */
1976:     nullptr, /* tp_init */
1977:     nullptr, /* tp_alloc */
1978:     THPFunction_new /* tp_new */
1979: };
1980: 
1981: bool THPFunction_initModule(PyObject* module) {
1982:   if (PyModule_AddType(module, &THPFunctionType) < 0)
1983:     return false;
1984:   return true;
```

- EN: The main execution path in this span is carried by `NOLINTNEXTLINE`, `THPFunction_initModule`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `NOLINTNEXTLINE`, `THPFunction_initModule` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1985-1985

```cpp
1985: }
```

- EN: This range continues the surrounding implementation through local state updates, helper invocations, and data movement.
- CN: 这一段通过局部状态更新、辅助函数调用与数据传递来延续周围实现。

## Key Concepts / 关键概念
- Autograd graph and gradient metadata / 自动求导图与梯度元数据
- Gradient bookkeeping and autograd history / 梯度记录与自动求导历史
- Python/C++ binding boundary / Python/C++ 绑定边界
- Tensor/value representation management / 张量/数值表示管理
- Runtime validation and diagnostics / 运行时校验与诊断
- State coordination and lifetime management / 状态协调与生命周期管理

## Dependencies / 依赖关系
- Direct includes / 直接包含: `torch/csrc/autograd/python_function.h`, `atomic`, `ATen/ATen.h`, `ATen/SequenceNumber.h`, `c10/util/irange.h`, `pybind11/pybind11.h`, `structmember.h`, `torch/csrc/PyInterpreter.h`, `torch/csrc/python_headers.h`, `torch/csrc/utils/pybind.h`
- Include roots / 头文件根模块: `ATen`, `c10`, `pybind11`, `torch`
- Key symbols / 关键符号: `UnpackedInput`, `InputFlags`, `python_error`, `check_legacy_fn_attr_access`, `throw_python_error`, `unpack_saved_variables`, `TORCH_CHECK`, `PyTuple_New`, `saved`, `to_py_size`
- Related subsystems / 相关子系统: Autograd / 自动求导, ATen operator layer / ATen 算子层, c10 core utilities / c10 核心工具, Python binding layer / Python 绑定层, JIT/tracing integration / JIT 与追踪集成
