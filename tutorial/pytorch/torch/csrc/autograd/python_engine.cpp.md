# python_engine.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/autograd/python_engine.cpp`
- Repository: `/root/xw/pytorch`
- Purpose (EN): Implements the core autograd execution engine and the scheduling logic behind backward passes.
- 目的 (CN): 实现核心自动求导执行引擎以及反向传播背后的调度逻辑。
- Lines: 513
- Language: C++ / C++

## Line-by-Line Analysis / 逐行分析

### Lines 1-16

```cpp
 1: #include <torch/csrc/autograd/python_engine.h>
 2: 
 3: #include <ATen/LegacyBatchedTensorImpl.h>
 4: #include <ATen/LegacyVmapMode.h>
 5: #include <c10/util/irange.h>
 6: #include <pybind11/pybind11.h>
 7: #include <torch/csrc/DynamicTypes.h>
 8: #include <torch/csrc/THP.h>
 9: #include <torch/csrc/autograd/edge.h>
10: #include <torch/csrc/autograd/engine.h>
11: #include <torch/csrc/autograd/function.h>
12: #include <torch/csrc/autograd/functions/basic_ops.h>
13: #include <torch/csrc/autograd/python_anomaly_mode.h>
14: #include <torch/csrc/autograd/python_cpp_function.h>
15: #include <torch/csrc/autograd/python_function.h>
16: #include <torch/csrc/autograd/python_saved_variable_hooks.h>
```

- EN: These lines pull in dependencies such as `torch/csrc/autograd/python_engine.h`, `ATen/LegacyBatchedTensorImpl.h`, `ATen/LegacyVmapMode.h`, establishing the headers needed by the implementation.
- CN: 这些行引入了依赖，例如 `torch/csrc/autograd/python_engine.h`, `ATen/LegacyBatchedTensorImpl.h`, `ATen/LegacyVmapMode.h`，为后续实现建立所需的头文件基础。
### Lines 17-32

```cpp
17: #include <torch/csrc/utils/pybind.h>
18: #include <torch/csrc/utils/pycfunction_helpers.h>
19: 
20: #ifndef _WIN32
21: #include <pthread.h>
22: #endif
23: 
24: #include <memory> // for unique_ptr
25: #include <utility>
26: 
27: using namespace torch::autograd;
28: 
29: struct THPEngine {
30:   PyObject_HEAD
31: };
32: 
```

- EN: These lines pull in dependencies such as `torch/csrc/utils/pybind.h`, `torch/csrc/utils/pycfunction_helpers.h`, `pthread.h`, establishing the headers needed by the implementation. They also open, refine, or close the relevant C++ namespace scope. This range declares or shapes types such as `THPEngine`. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这些行引入了依赖，例如 `torch/csrc/utils/pybind.h`, `torch/csrc/utils/pycfunction_helpers.h`, `pthread.h`，为后续实现建立所需的头文件基础。 它们还会打开、细化或关闭相应的 C++ 命名空间作用域。 这一段声明或塑造了 ``THPEngine`` 等类型。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 33-48

```cpp
33: static bool _reinitialize_engine = false;
34: 
35: namespace torch::autograd::python {
36: 
37: PythonEngine::PythonEngine() = default;
38: 
39: Engine& PythonEngine::get_python_engine() {
40:   static PythonEngine engine;
41:   // This is "probably" thread-safe because the flag is set in a fork handler
42:   // before any threads are created, and this function is only called with the
43:   // GIL held. However, using fork + threads is playing with fire so this is
44:   // more of a "best effort" thing. For example, if the fork occurs while the
45:   // backwards threads hold a lock, we'll probably deadlock in the engine
46:   // destructor.
47:   if (_reinitialize_engine) {
48:     engine.release_workers();
```

- EN: They also open, refine, or close the relevant C++ namespace scope. The main execution path in this span is carried by `PythonEngine`, `get_python_engine`. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 它们还会打开、细化或关闭相应的 C++ 命名空间作用域。 这一段的主要执行路径由 `PythonEngine`, `get_python_engine` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 49-64

```cpp
49:     engine.~PythonEngine();
50:     new (&engine) torch::autograd::python::PythonEngine();
51:     _reinitialize_engine = false;
52:   }
53:   return engine;
54: }
55: 
56: PythonEngine::~PythonEngine() {
57:   Engine::stop();
58: }
59: 
60: void PythonEngine::thread_init(
61:     int device,
62:     const std::shared_ptr<ReadyQueue>& ready_queue,
63:     bool should_increment) {
64:   // Increment thread usage count before acquiring the GIL
```

- EN: The main execution path in this span is carried by `new`, `PythonEngine`, `stop`. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `new`, `PythonEngine`, `stop` 等函数/方法承载。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 65-80

```cpp
65:   if (should_increment) {
66:     increment_non_reentrant_thread_count();
67:   }
68:   // Create a PyThreadState, but release the GIL. This lets
69:   // pybind11::gil_scoped_acquire calls inside thread_main acquire the GIL
70:   // without having to create a new PyThreadState each time.
71:   auto gil = std::make_unique<pybind11::gil_scoped_acquire>();
72:   pybind11::gil_scoped_release no_gil;
73:   Engine::thread_init(device, ready_queue, false);
74: 
75:   if (should_increment) {
76:     // Decrement the count during shutdown if we incremented earlier.
77:     decrement_non_reentrant_thread_count();
78:   }
79: 
80:   // Do not call PyEval_RestoreThread, PyThreadState_[Clear|DeleteCurrent] if
```

- EN: The main execution path in this span is carried by `increment_non_reentrant_thread_count`, `thread_init`, `decrement_non_reentrant_thread_count`. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这一段的主要执行路径由 `increment_non_reentrant_thread_count`, `thread_init`, `decrement_non_reentrant_thread_count` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 81-96

```cpp
81:   // runtime is finalizing
82:   if (!Py_IsInitialized()) {
83:     no_gil.disarm();
84:     // TODO: call disarm once PyThreadState_Clear can safely be called from
85:     // finalize NOTE: deploy.cpp calls `PyInterpreterState_Delete` to destruct
86:     // PyThreadState, so avoid use-after-free here.
87:     auto ptr = gil.release();
88:     operator delete(ptr);
89:   }
90: }
91: 
92: void PythonEngine::thread_on_exception(
93:     const std::shared_ptr<GraphTask>& graph_task,
94:     const c10::intrusive_ptr<Node>& fn,
95:     std::exception& e) {
96:   // See Note [ Persisting PyErr state across autograd engine threads ]
```

- EN: The main execution path in this span is carried by `delete`, `thread_on_exception`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here.
- CN: 这一段的主要执行路径由 `delete`, `thread_on_exception` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。
### Lines 97-112

```cpp
 97:   auto python_err = dynamic_cast<python_error*>(&e);
 98:   if (python_err) {
 99:     python_err->persist();
100:   }
101:   Engine::thread_on_exception(graph_task, fn, e);
102: }
103: 
104: std::unique_ptr<AnomalyMetadata> PythonEngine::make_anomaly_metadata() {
105:   return std::make_unique<PyAnomalyMetadata>();
106: }
107: 
108: std::unique_ptr<SavedVariableHooks> PythonEngine::
109:     get_default_saved_variable_hooks() {
110:   return PyDefaultSavedVariableHooks::get_hooks();
111: }
112: 
```

- EN: The main execution path in this span is carried by `thread_on_exception`, `make_anomaly_metadata`, `get_default_saved_variable_hooks`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `thread_on_exception`, `make_anomaly_metadata`, `get_default_saved_variable_hooks` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 113-128

```cpp
113: variable_list PythonEngine::execute(
114:     const edge_list& roots,
115:     const variable_list& inputs,
116:     bool keep_graph,
117:     bool create_graph,
118:     bool accumulate_grad,
119:     const edge_list& outputs) {
120:   TORCH_CHECK(
121:       !PyGILState_Check(),
122:       "The autograd engine was called while holding the GIL. If you are using the C++ "
123:       "API, the autograd engine is an expensive operation that does not require the "
124:       "GIL to be held so you should release it with 'pybind11::gil_scoped_release no_gil;'"
125:       ". If you are not using the C++ API, please report a bug to the pytorch team.")
126:   try {
127:     return Engine::execute(
128:         roots, inputs, keep_graph, create_graph, accumulate_grad, outputs);
```

- EN: The main execution path in this span is carried by `execute`, `TORCH_CHECK`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `execute`, `TORCH_CHECK` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 129-144

```cpp
129:   } catch (python_error& e) {
130:     e.restore();
131:     throw;
132:   }
133: }
134: 
135: c10::intrusive_ptr<at::ivalue::Future> PythonEngine::execute_with_graph_task(
136:     const std::shared_ptr<GraphTask>& graph_task,
137:     c10::intrusive_ptr<Node> graph_root,
138:     InputBuffer&& input_buffer) {
139:   try {
140:     return Engine::execute_with_graph_task(
141:         graph_task, std::move(graph_root), std::move(input_buffer));
142:   } catch (python_error& e) {
143:     pybind11::gil_scoped_acquire gil;
144:     if (!PyErr_Occurred()) {
```

- EN: The main execution path in this span is carried by `execute_with_graph_task`, `move`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `execute_with_graph_task`, `move` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 145-160

```cpp
145:       // Set the error indicator only if it is not set already.
146:       e.restore();
147:     }
148:     throw;
149:   }
150: }
151: } // namespace torch::autograd::python
152: 
153: static Edge parseGradientEdge(PyObject* obj, int64_t index) {
154:   PyObject* grad_fn = PyTuple_GetItem(obj, 0);
155:   auto output_nr = THPUtils_unpackLong(PyTuple_GetItem(obj, 1));
156:   c10::intrusive_ptr<torch::autograd::Node> grad_fn_sp;
157:   if (THPFunction_Check(grad_fn)) {
158:     grad_fn_sp = ((THPFunction*)grad_fn)->cdata.lock();
159:   } else if (THPCppFunction_Check(grad_fn)) {
160:     grad_fn_sp = ((THPCppFunction*)grad_fn)->cdata;
```

- EN: They also open, refine, or close the relevant C++ namespace scope. The main execution path in this span is carried by `parseGradientEdge`, `PyTuple_GetItem`, `THPUtils_unpackLong`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here.
- CN: 它们还会打开、细化或关闭相应的 C++ 命名空间作用域。 这一段的主要执行路径由 `parseGradientEdge`, `PyTuple_GetItem`, `THPUtils_unpackLong` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。
### Lines 161-176

```cpp
161:   } else {
162:     TORCH_CHECK(
163:         false,
164:         "GradientEdge's first object must be an autograd.graph.Node "
165:         "but got ",
166:         THPUtils_typename(grad_fn));
167:   }
168:   return Edge(grad_fn_sp, output_nr);
169: }
170: 
171: // Implementation of torch._C._EngineBase.run_backward
172: static PyObject* THPEngine_run_backward(
173:     PyObject* self,
174:     PyObject* args,
175:     PyObject* kwargs) {
176:   HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `TORCH_CHECK`, `THPUtils_typename`, `Edge`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here.
- CN: 这一段的主要执行路径由 `TORCH_CHECK`, `THPUtils_typename`, `Edge` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。
### Lines 177-192

```cpp
177:   PyObject* tensors = nullptr;
178:   PyObject* grad_tensors = nullptr;
179:   unsigned char keep_graph = 0;
180:   unsigned char create_graph = 0;
181:   PyObject* inputs = nullptr;
182:   unsigned char allow_unreachable = 0;
183:   unsigned char accumulate_grad =
184:       0; // Indicate whether to accumulate grad into leaf Tensors or capture
185:   constexpr const char* accepted_kwargs[] = {
186:       "tensors",
187:       "grad_tensors",
188:       "keep_graph",
189:       "create_graph",
190:       "inputs",
191:       "allow_unreachable",
192:       "accumulate_grad",
```

- EN: This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 193-208

```cpp
193:       nullptr};
194:   if (!PyArg_ParseTupleAndKeywords(
195:           args,
196:           kwargs,
197:           "OObb|Obb",
198:           // NOLINTNEXTLINE(cppcoreguidelines-pro-type-const-cast,-warnings-as-errors)
199:           const_cast<char**>(accepted_kwargs),
200:           &tensors,
201:           &grad_tensors,
202:           &keep_graph,
203:           &create_graph,
204:           &inputs,
205:           &allow_unreachable,
206:           &accumulate_grad))
207:     return nullptr;
208:   TORCH_CHECK(
```

- EN: The main execution path in this span is carried by `NOLINTNEXTLINE`, `TORCH_CHECK`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `NOLINTNEXTLINE`, `TORCH_CHECK` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 209-224

```cpp
209:       PyTuple_Check(tensors),
210:       "tensors argument is expected to "
211:       "be a tuple, but got ",
212:       THPUtils_typename(tensors));
213:   TORCH_CHECK(
214:       PyTuple_Check(grad_tensors),
215:       "grad_tensors argument is "
216:       "expected to be a tuple, but got ",
217:       THPUtils_typename(grad_tensors));
218: 
219:   Py_ssize_t num_tensors = PyTuple_GET_SIZE(tensors);
220:   Py_ssize_t num_gradients = PyTuple_GET_SIZE(grad_tensors);
221:   TORCH_CHECK(
222:       num_tensors == num_gradients,
223:       "got ",
224:       num_tensors,
```

- EN: The main execution path in this span is carried by `PyTuple_Check`, `THPUtils_typename`, `TORCH_CHECK`. The logic emits runtime diagnostics or assertions to guard assumptions.
- CN: 这一段的主要执行路径由 `PyTuple_Check`, `THPUtils_typename`, `TORCH_CHECK` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。
### Lines 225-240

```cpp
225:       " tensors and ",
226:       num_gradients,
227:       " gradients");
228: 
229:   // The user either called autograd.backward(...) or autograd.grad(...) to get
230:   // here
231:   bool backward_api_called = accumulate_grad;
232:   TORCH_CHECK(
233:       !backward_api_called || at::impl::VmapMode::current_vmap_level() == 0,
234:       "backward() called inside torch.vmap. This is not supported, "
235:       "please call backward() outside torch.vmap or instead use "
236:       "torch.autograd.grad inside torch.vmap");
237: 
238:   edge_list roots;
239:   roots.reserve(num_tensors);
240:   variable_list grads;
```

- EN: The main execution path in this span is carried by `TORCH_CHECK`, `current_vmap_level`, `backward`. The logic emits runtime diagnostics or assertions to guard assumptions.
- CN: 这一段的主要执行路径由 `TORCH_CHECK`, `current_vmap_level`, `backward` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。
### Lines 241-256

```cpp
241:   grads.reserve(num_tensors);
242:   for (const auto i : c10::irange(num_tensors)) {
243:     PyObject* _tensor = PyTuple_GET_ITEM(tensors, i);
244:     Edge gradient_edge; // Temporary variable to hold the gradient edge
245:     std::optional<at::Tensor> mb_output;
246:     if (THPVariable_Check(_tensor)) {
247:       mb_output = THPVariable_Unpack(_tensor);
248:       TORCH_CHECK(
249:           !isBatchedTensor(mb_output.value()),
250:           "torch.autograd.grad(outputs, inputs, grad_outputs) called inside ",
251:           "torch.vmap. We do not support the case where any outputs are ",
252:           "vmapped tensors (output ",
253:           i,
254:           " is being vmapped over). Please "
255:           "call autograd.grad() outside torch.vmap or file a bug report "
256:           "with your use case.");
```

- EN: The main execution path in this span is carried by `PyTuple_GET_ITEM`, `THPVariable_Unpack`, `TORCH_CHECK`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `PyTuple_GET_ITEM`, `THPVariable_Unpack`, `TORCH_CHECK` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 257-272

```cpp
257:       gradient_edge = torch::autograd::impl::gradient_edge(mb_output.value());
258:     } else if (PyObject_IsInstance(_tensor, THPGradientEdgeClass)) {
259:       gradient_edge = parseGradientEdge(_tensor, i);
260:     } else {
261:       TORCH_CHECK(
262:           false,
263:           "element ",
264:           i,
265:           " of tensors tuple is neither a Tensor nor a GradientEdge");
266:     }
267:     TORCH_CHECK(
268:         gradient_edge.function,
269:         "element ",
270:         i,
271:         " of tensors does not require grad and does not have a grad_fn");
272:     roots.push_back(std::move(gradient_edge));
```

- EN: The main execution path in this span is carried by `gradient_edge`, `parseGradientEdge`, `TORCH_CHECK`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here.
- CN: 这一段的主要执行路径由 `gradient_edge`, `parseGradientEdge`, `TORCH_CHECK` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。
### Lines 273-288

```cpp
273: 
274:     PyObject* grad = PyTuple_GET_ITEM(grad_tensors, i);
275:     if (THPVariable_Check(grad)) {
276:       const Variable& grad_var = THPVariable_Unpack(grad);
277:       if (grad_var.has_names()) {
278:         TORCH_WARN(
279:             "Autograd was passed a named grad tensor with dims ",
280:             grad_var.names(),
281:             ". Autograd does not yet support named tensor semantics, so all names ",
282:             "will be ignored. In practice all computed gradients will still be correct "
283:             "according to regular tensor semantics.");
284:       }
285:       grads.push_back(grad_var);
286:     } else {
287:       TORCH_CHECK(
288:           Py_IsNone(grad),
```

- EN: The main execution path in this span is carried by `PyTuple_GET_ITEM`, `THPVariable_Unpack`, `TORCH_WARN`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here.
- CN: 这一段的主要执行路径由 `PyTuple_GET_ITEM`, `THPVariable_Unpack`, `TORCH_WARN` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。
### Lines 289-304

```cpp
289:           "element ",
290:           i,
291:           " of gradients tuple is not a Tensor or None");
292:       TORCH_CHECK(
293:           mb_output.has_value(),
294:           "element ",
295:           i,
296:           " of gradients tuple is None, but the corresponding output is a GradientEdge."
297:           "This is not supported.");
298:       TORCH_CHECK(
299:           !mb_output.value().requires_grad(),
300:           "element ",
301:           i,
302:           " of gradients tuple is None, but the corresponding Tensor requires grad");
303:     }
304:   }
```

- EN: The main execution path in this span is carried by `TORCH_CHECK`. The logic emits runtime diagnostics or assertions to guard assumptions. Autograd-related metadata or graph connectivity is updated here.
- CN: 这一段的主要执行路径由 `TORCH_CHECK` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这里会更新与自动求导相关的元数据或计算图连接关系。
### Lines 305-320

```cpp
305: 
306:   std::vector<Edge> output_edges;
307:   if (inputs != nullptr) {
308:     TORCH_CHECK(
309:         PyTuple_CheckExact(inputs), "inputs to run_backward must be a tuple");
310:     int num_inputs = PyTuple_GET_SIZE(inputs);
311:     output_edges.reserve(num_inputs);
312:     for (const auto i : c10::irange(num_inputs)) {
313:       PyObject* input = PyTuple_GET_ITEM(inputs, i);
314:       if (THPVariable_Check(input)) {
315:         const auto& tensor = THPVariable_Unpack(input);
316:         TORCH_CHECK(
317:             !isBatchedTensor(tensor),
318:             "torch.autograd.grad(outputs, inputs, grad_outputs) called inside ",
319:             "torch.vmap. We do not support the case where any inputs are ",
320:             "vmapped tensors (input ",
```

- EN: The main execution path in this span is carried by `TORCH_CHECK`, `PyTuple_CheckExact`, `PyTuple_GET_SIZE`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `TORCH_CHECK`, `PyTuple_CheckExact`, `PyTuple_GET_SIZE` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 321-336

```cpp
321:             i,
322:             " is being vmapped over). Please "
323:             "call autograd.grad() outside torch.vmap or file a bug report "
324:             "with your use case.")
325:         const auto output_nr = tensor.output_nr();
326:         auto grad_fn = tensor.grad_fn();
327:         if (!grad_fn) {
328:           grad_fn = torch::autograd::impl::try_get_grad_accumulator(tensor);
329:         }
330:         if (accumulate_grad) {
331:           tensor.retain_grad();
332:         }
333:         TORCH_CHECK(
334:             tensor.requires_grad(),
335:             "One of the differentiated Tensors does not require grad");
336:         if (!grad_fn) {
```

- EN: The main execution path in this span is carried by `try_get_grad_accumulator`, `TORCH_CHECK`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here.
- CN: 这一段的主要执行路径由 `try_get_grad_accumulator`, `TORCH_CHECK` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。
### Lines 337-352

```cpp
337:           // NOTE [ Autograd Unreachable Input ]
338:           // Since input has no grad_accumulator, its guaranteed to be
339:           // unreachable. We initialize an edge pointing to a non-nullptr Node
340:           // so nodes in the graph (e.g., mul when an operand is scalar) that
341:           // have edges pointing to nullptr don't get erroneously assigned
342:           // `needed = True` in exec_info.
343:           output_edges.emplace_back(c10::make_intrusive<Identity>(), 0);
344:         } else {
345:           output_edges.emplace_back(grad_fn, output_nr);
346:         }
347:       } else if (PyObject_IsInstance(input, THPGradientEdgeClass)) {
348:         output_edges.emplace_back(parseGradientEdge(input, i));
349:       } else {
350:         TORCH_CHECK(
351:             false,
352:             "all inputs have to be Tensors or GradientEdges, but got ",
```

- EN: The main execution path in this span is carried by `graph`, `TORCH_CHECK`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here.
- CN: 这一段的主要执行路径由 `graph`, `TORCH_CHECK` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。
### Lines 353-368

```cpp
353:             THPUtils_typename(input));
354:       }
355:     }
356:   }
357: 
358:   variable_list outputs;
359:   {
360:     pybind11::gil_scoped_release no_gil;
361:     auto& engine = python::PythonEngine::get_python_engine();
362:     outputs = engine.execute(
363:         roots, grads, keep_graph, create_graph, accumulate_grad, output_edges);
364:   }
365: 
366:   if (!backward_api_called && inputs != nullptr) {
367:     int num_inputs = PyTuple_GET_SIZE(inputs);
368:     THPObjectPtr py_outputs{PyTuple_New(num_inputs)};
```

- EN: The main execution path in this span is carried by `THPUtils_typename`, `get_python_engine`, `PyTuple_GET_SIZE`. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这一段的主要执行路径由 `THPUtils_typename`, `get_python_engine`, `PyTuple_GET_SIZE` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 369-384

```cpp
369:     if (!py_outputs)
370:       return nullptr;
371:     for (const auto i : c10::irange(num_inputs)) {
372:       TORCH_CHECK(
373:           allow_unreachable || outputs[i].defined(),
374:           "One of the "
375:           "differentiated Tensors appears to not have been used "
376:           "in the graph. Set allow_unused=True if this is the "
377:           "desired behavior.");
378:       PyTuple_SET_ITEM(py_outputs.get(), i, THPVariable_Wrap(outputs[i]));
379:     }
380:     return py_outputs.release();
381:   } else {
382:     Py_RETURN_NONE;
383:   }
384:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `TORCH_CHECK`, `PyTuple_SET_ITEM`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `TORCH_CHECK`, `PyTuple_SET_ITEM` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 385-400

```cpp
385: }
386: 
387: static PyObject* THPEngine_queue_callback(PyObject* self, PyObject* _callback) {
388:   HANDLE_TH_ERRORS
389:   auto& engine = python::PythonEngine::get_python_engine();
390:   std::shared_ptr<PyObject> callback(_callback, [](PyObject* obj) {
391:     pybind11::gil_scoped_acquire gil;
392:     Py_DECREF(obj);
393:   });
394:   Py_INCREF(_callback);
395:   engine.queue_callback([callback]() {
396:     pybind11::gil_scoped_acquire gil;
397:     THPObjectPtr result{PyObject_CallFunctionObjArgs(callback.get(), nullptr)};
398:     if (!result) {
399:       // Note [ Persisting PyErr state across autograd engine threads ]
400:       //
```

- EN: The main execution path in this span is carried by `THPEngine_queue_callback`, `get_python_engine`, `callback`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPEngine_queue_callback`, `get_python_engine`, `callback` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 401-416

```cpp
401:       // Since the autograd engine is multi-threaded, and Python error state is
402:       // local to each thread, it must preserve the python error from the worker
403:       // thread and rethrow it as-is in the calling thread. This is done via
404:       // persisting the error in the two places that can encounter Python
405:       // errors: (1) evaluate function and (2) queued callbacks.
406:       //
407:       // TODO: the engine is not actually responsible for persisting the error
408:       // in the custom autograd Function case today! See the note above
409:       // `raise_python_error()` function in python_function.cpp and
410:       // python_hooks.cpp for more details. Persisting an extra time in the
411:       // engine is fine because doing so is a no-op when the python_error has
412:       // already been persisted.
413:       python_error err;
414:       err.persist();
415:       throw std::move(err);
416:     }
```

- EN: The main execution path in this span is carried by `move`. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这一段的主要执行路径由 `move` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 417-432

```cpp
417:   });
418:   Py_RETURN_NONE;
419:   END_HANDLE_TH_ERRORS
420: }
421: 
422: static PyObject* THPEngine_is_checkpoint_valid(
423:     PyObject* self,
424:     PyObject* noargs) {
425:   HANDLE_TH_ERRORS
426:   auto& engine = python::PythonEngine::get_python_engine();
427:   if (engine.is_checkpoint_valid()) {
428:     Py_RETURN_TRUE;
429:   } else {
430:     Py_RETURN_FALSE;
431:   }
432:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPEngine_is_checkpoint_valid`, `get_python_engine`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPEngine_is_checkpoint_valid`, `get_python_engine` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 433-448

```cpp
433: }
434: 
435: static PyObject* THPEngine_new(
436:     PyTypeObject* type,
437:     PyObject* args,
438:     PyObject* kwargs) {
439:   return type->tp_alloc(type, 0);
440: }
441: 
442: // NOLINTNEXTLINE(cppcoreguidelines-avoid-c-arrays,modernize-avoid-c-arrays,cppcoreguidelines-avoid-non-const-global-variables)
443: static struct PyMethodDef THPEngine_methods[] = {
444:     {"run_backward",
445:      castPyCFunctionWithKeywords(THPEngine_run_backward),
446:      METH_VARARGS | METH_KEYWORDS,
447:      nullptr},
448:     {"queue_callback", THPEngine_queue_callback, METH_O, nullptr},
```

- EN: The main execution path in this span is carried by `THPEngine_new`, `NOLINTNEXTLINE`, `castPyCFunctionWithKeywords`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPEngine_new`, `NOLINTNEXTLINE`, `castPyCFunctionWithKeywords` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 449-464

```cpp
449:     {"is_checkpoint_valid",
450:      THPEngine_is_checkpoint_valid,
451:      METH_NOARGS,
452:      nullptr},
453:     {nullptr}};
454: 
455: static PyTypeObject THPEngineType = {
456:     PyVarObject_HEAD_INIT(nullptr, 0)
457:     "torch._C._EngineBase", /* tp_name */
458:     sizeof(THPEngine), /* tp_basicsize */
459:     0, /* tp_itemsize */
460:     nullptr, /* tp_dealloc */
461:     0, /* tp_vectorcall_offset */
462:     nullptr, /* tp_getattr */
463:     nullptr, /* tp_setattr */
464:     nullptr, /* tp_reserved */
```

- EN: The main execution path in this span is carried by `PyVarObject_HEAD_INIT`.
- CN: 这一段的主要执行路径由 `PyVarObject_HEAD_INIT` 等函数/方法承载。
### Lines 465-480

```cpp
465:     nullptr, /* tp_repr */
466:     nullptr, /* tp_as_number */
467:     nullptr, /* tp_as_sequence */
468:     nullptr, /* tp_as_mapping */
469:     nullptr, /* tp_hash  */
470:     nullptr, /* tp_call */
471:     nullptr, /* tp_str */
472:     nullptr, /* tp_getattro */
473:     nullptr, /* tp_setattro */
474:     nullptr, /* tp_as_buffer */
475:     // NOLINTNEXTLINE(misc-redundant-expression)
476:     Py_TPFLAGS_DEFAULT | Py_TPFLAGS_BASETYPE, /* tp_flags */
477:     nullptr, /* tp_doc */
478:     nullptr, /* tp_traverse */
479:     nullptr, /* tp_clear */
480:     nullptr, /* tp_richcompare */
```

- EN: This range is comment-only; it documents the surrounding implementation, generated provenance, or usage notes.
- CN: 这一段仅包含注释，用于说明周围实现、生成来源或使用说明。
### Lines 481-496

```cpp
481:     0, /* tp_weaklistoffset */
482:     nullptr, /* tp_iter */
483:     nullptr, /* tp_iternext */
484:     THPEngine_methods, /* tp_methods */
485:     nullptr, /* tp_members */
486:     nullptr, /* tp_getset */
487:     nullptr, /* tp_base */
488:     nullptr, /* tp_dict */
489:     nullptr, /* tp_descr_get */
490:     nullptr, /* tp_descr_set */
491:     0, /* tp_dictoffset */
492:     nullptr, /* tp_init */
493:     nullptr, /* tp_alloc */
494:     THPEngine_new /* tp_new */
495: };
496: 
```

- EN: This range continues the surrounding implementation through local state updates, helper invocations, and data movement.
- CN: 这一段通过局部状态更新、辅助函数调用与数据传递来延续周围实现。
### Lines 497-512

```cpp
497: static void child_atfork() {
498:   _reinitialize_engine = true;
499: }
500: 
501: bool THPEngine_initModule(PyObject* module) {
502: #ifndef _WIN32
503:   TORCH_CHECK(
504:       pthread_atfork(nullptr, nullptr, child_atfork) == 0,
505:       "unable to set pthread_atfork handler");
506: #endif
507:   if (PyType_Ready(&THPEngineType) < 0)
508:     return false;
509:   Py_INCREF(&THPEngineType);
510:   PyModule_AddObject(module, "_ImperativeEngine", (PyObject*)&THPEngineType);
511:   set_default_engine_stub(python::PythonEngine::get_python_engine);
512:   return true;
```

- EN: The main execution path in this span is carried by `child_atfork`, `THPEngine_initModule`, `TORCH_CHECK`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `child_atfork`, `THPEngine_initModule`, `TORCH_CHECK` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 513-513

```cpp
513: }
```

- EN: This range continues the surrounding implementation through local state updates, helper invocations, and data movement.
- CN: 这一段通过局部状态更新、辅助函数调用与数据传递来延续周围实现。

## Key Concepts / 关键概念
- Autograd graph and gradient metadata / 自动求导图与梯度元数据
- Gradient bookkeeping and autograd history / 梯度记录与自动求导历史
- Python/C++ binding boundary / Python/C++ 绑定边界
- Tensor/value representation management / 张量/数值表示管理
- Runtime validation and diagnostics / 运行时校验与诊断
- Primary symbol `THPEngine` / 核心符号 `THPEngine`

## Dependencies / 依赖关系
- Direct includes / 直接包含: `torch/csrc/autograd/python_engine.h`, `ATen/LegacyBatchedTensorImpl.h`, `ATen/LegacyVmapMode.h`, `c10/util/irange.h`, `pybind11/pybind11.h`, `torch/csrc/DynamicTypes.h`, `torch/csrc/THP.h`, `torch/csrc/autograd/edge.h`, `torch/csrc/autograd/engine.h`, `torch/csrc/autograd/function.h`
- Include roots / 头文件根模块: `ATen`, `c10`, `pybind11`, `torch`
- Key symbols / 关键符号: `THPEngine`, `delete`, `get_default_saved_variable_hooks`, `parseGradientEdge`, `Edge`, `THPEngine_run_backward`, `THPEngine_queue_callback`, `callback`, `THPEngine_is_checkpoint_valid`, `THPEngine_new`
- Related subsystems / 相关子系统: Autograd / 自动求导, ATen operator layer / ATen 算子层, c10 core utilities / c10 核心工具, Python binding layer / Python 绑定层
