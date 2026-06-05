# python_functions.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `torch/csrc/distributed/rpc/python_functions.cpp`
- Repository: `pytorch` (`/root/xw/pytorch`)
- Purpose (EN): This file provides Python interoperability logic in the distributed RPC layer. Representative routines include `toPyIValue`, `TORCH_CHECK`, `matchBuiltinOp`, `sendPythonRemoteCall`, `toPyJitFuture`. It also exposes native functionality to Python.
- 用途 (CN): 该文件在分布式 RPC 层中提供Python 互操作逻辑。 代表性例程包括 `toPyIValue`、`TORCH_CHECK`、`matchBuiltinOp`、`sendPythonRemoteCall`、`toPyJitFuture`。 同时它还会向 Python 暴露原生能力。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16 / 第 1-16 行

```cpp
1: #include <ATen/ThreadLocalState.h>
2: #include <torch/csrc/distributed/autograd/utils.h>
3: #include <torch/csrc/distributed/rpc/message.h>
4: #include <torch/csrc/distributed/rpc/python_call.h>
5: #include <torch/csrc/distributed/rpc/python_functions.h>
6: #include <torch/csrc/distributed/rpc/python_remote_call.h>
7: #include <torch/csrc/distributed/rpc/python_resp.h>
8: #include <torch/csrc/distributed/rpc/python_rpc_handler.h>
9: #include <torch/csrc/distributed/rpc/rref_context.h>
10: #include <torch/csrc/distributed/rpc/script_call.h>
11: #include <torch/csrc/distributed/rpc/script_remote_call.h>
12: #include <torch/csrc/distributed/rpc/script_resp.h>
13: #include <torch/csrc/distributed/rpc/torchscript_functions.h>
14: #include <torch/csrc/distributed/rpc/utils.h>
15: #include <torch/csrc/jit/runtime/operator.h>
16: #include <exception>
```

- EN: Lines 1-16 pulls in the headers required by this translation unit or interface.
- CN: 第 1-16 行引入该实现单元或接口所需的头文件。

### Lines 17-32 / 第 17-32 行

```cpp
17: 
18: namespace torch::distributed::rpc {
19: 
20: namespace {
21: 
22: IValue toPyIValue(const Message& message) {
23:   MessageType msgType = message.type();
24:   auto response = deserializeResponse(message, msgType);
25:   switch (msgType) {
26:     case MessageType::SCRIPT_RET: {
27:       auto& ret = static_cast<ScriptResp&>(*response);
28:       Stack stack;
29:       stack.push_back(ret.value());
30:       // Need GIL to guard createPyObjectForStack() and its returned
31:       // py::object
32:       py::gil_scoped_acquire acquire;
```

- EN: Lines 17-32 opens or closes namespaces to place the code in the correct distributed component; introduces executable logic in routines such as `toPyIValue`; binds C++ behavior into Python-facing APIs with pybind or CPython helpers.
- CN: 第 17-32 行打开或关闭命名空间，以便将代码放入正确的分布式组件中；在 `toPyIValue` 等例程中引入具体执行逻辑；通过 pybind 或 CPython 辅助接口把 C++ 行为绑定到 Python API。

### Lines 33-48 / 第 33-48 行

```cpp
33:       return jit::toIValue(
34:           torch::jit::createPyObjectForStack(std::move(stack)),
35:           PyObjectType::get());
36:     }
37:     case MessageType::PYTHON_RET: {
38:       // TODO: Try to avoid a copy here.
39:       auto& resp = static_cast<PythonResp&>(*response);
40:       auto& pythonRpcHandler = PythonRpcHandler::getInstance();
41:       // Need GIL to destruct the py::object returned by deserialize()
42:       py::gil_scoped_acquire acquire;
43:       py::object value = pythonRpcHandler.deserialize(resp.serializedPyObj());
44:       pythonRpcHandler.handleException(value);
45:       return jit::toIValue(value, PyObjectType::get());
46:     }
47:     default: {
48:       TORCH_CHECK(false, "Unrecognized response message type ", msgType);
```

- EN: Lines 33-48 introduces executable logic in routines such as `TORCH_CHECK`; binds C++ behavior into Python-facing APIs with pybind or CPython helpers; performs validation and error handling to keep distributed state consistent.
- CN: 第 33-48 行在 `TORCH_CHECK` 等例程中引入具体执行逻辑；通过 pybind 或 CPython 辅助接口把 C++ 行为绑定到 Python API；执行校验与错误处理，以保持分布式状态一致。

### Lines 49-64 / 第 49-64 行

```cpp
49:     }
50:   }
51: }
52: 
53: std::shared_ptr<Operator> matchBuiltinOp(
54:     const std::string& opName,
55:     const py::args& args,
56:     const py::kwargs& kwargs,
57:     Stack& stack) {
58:   Symbol symbol = Symbol::fromQualString(opName);
59: 
60:   std::shared_ptr<jit::Operator> matchedOperator;
61:   if (symbol.is_aten()) {
62:     // Prefer C10 ops so that they go through C10 dispatch. We expect the
63:     // total # of possible overloaded ops (i.e. size of below ops list) to be
64:     // small (i.e. it is 10 for torch.add) so a worst-case linear search should
```

- EN: Lines 49-64 introduces executable logic in routines such as `matchBuiltinOp`; binds C++ behavior into Python-facing APIs with pybind or CPython helpers.
- CN: 第 49-64 行在 `matchBuiltinOp` 等例程中引入具体执行逻辑；通过 pybind 或 CPython 辅助接口把 C++ 行为绑定到 Python API。

### Lines 65-80 / 第 65-80 行

```cpp
65:     // not incur significant extra overhead.
66:     auto ops = torch::jit::getAllOperatorsFor(symbol);
67:     std::vector<std::shared_ptr<torch::jit::Operator>> c10OpsForSymbol;
68:     for (auto it = ops.begin(); it != ops.end();) {
69:       std::shared_ptr<jit::Operator> op = *it;
70:       if (op->isC10Op()) {
71:         c10OpsForSymbol.emplace_back(std::move(op));
72:         it = ops.erase(it);
73:       } else {
74:         ++it;
75:       }
76:     }
77: 
78:     // Don't throw on failures in this call, since we are not examining on all
79:     // operators here, and the matched operator may indeed not be a c10 op.
80:     std::pair<std::shared_ptr<torch::jit::Operator>, torch::jit::Stack>
```

- EN: Lines 65-80 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 65-80 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 81-96 / 第 81-96 行

```cpp
81:         opWithStack;
82:     try {
83:       opWithStack = torch::jit::getOpWithStack(c10OpsForSymbol, args, kwargs);
84:     } catch (const std::runtime_error&) {
85:       opWithStack = torch::jit::getOpWithStack(ops, args, kwargs);
86:     }
87:     matchedOperator = std::move(std::get<0>(opWithStack));
88:     stack = std::move(std::get<1>(opWithStack));
89:   }
90: 
91:   // We should never hit this path, since if !matchedOperator, then the last
92:   // call to getOpWithStack should have thrown.
93:   TORCH_CHECK(
94:       matchedOperator != nullptr,
95:       "Failed to match operator name ",
96:       opName,
```

- EN: Lines 81-96 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 81-96 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 97-112 / 第 97-112 行

```cpp
97:       " and arguments "
98:       "(args: ",
99:       args,
100:       ", kwargs: ",
101:       kwargs,
102:       ") to a builtin operator");
103: 
104:   return matchedOperator;
105: }
106: 
107: c10::intrusive_ptr<JitFuture> sendPythonRemoteCall(
108:     const WorkerInfo& dst,
109:     SerializedPyObj serializedPyObj,
110:     const IValue& rrefId,
111:     const IValue& forkId,
112:     const float rpcTimeoutSeconds,
```

- EN: Lines 97-112 returns computed state or forwards results to the surrounding caller.
- CN: 第 97-112 行返回计算结果，或将结果继续传递给外围调用方。

### Lines 113-128 / 第 113-128 行

```cpp
113:     const bool isAsyncExecution) {
114:   auto pythonRemoteCall = std::make_unique<PythonRemoteCall>(
115:       std::move(serializedPyObj), rrefId, forkId, isAsyncExecution);
116: 
117:   // set forceGradRecording to true as even if the args does not contain any
118:   // tensor, the return value might still contain tensors.
119:   auto agent = RpcAgent::getCurrentRpcAgent();
120:   return torch::distributed::autograd::sendMessageWithAutograd(
121:       *agent,
122:       dst,
123:       std::move(*pythonRemoteCall).toMessage(),
124:       true /*forceGradRecording*/,
125:       rpcTimeoutSeconds);
126: }
127: 
128: } // namespace
```

- EN: Lines 113-128 opens or closes namespaces to place the code in the correct distributed component; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 113-128 行打开或关闭命名空间，以便将代码放入正确的分布式组件中；为本段代码补充可调用 API、操作符或辅助例程。

### Lines 129-144 / 第 129-144 行

```cpp
129: 
130: using namespace torch::distributed::autograd;
131: 
132: c10::intrusive_ptr<JitFuture> toPyJitFuture(
133:     const c10::intrusive_ptr<JitFuture>& messageJitFuture,
134:     bool hasValue) {
135:   if (hasValue) {
136:     auto child = messageJitFuture->createInstance(PyObjectType::get());
137:     messageJitFuture->addCallback(
138:         at::wrapPropagateTLSState([child](JitFuture& future) {
139:           if (future.hasError()) {
140:             child->setError(future.exception_ptr());
141:           } else {
142:             const Message& message = *future.value().toCustomClass<Message>();
143: 
144:             // toPyIValue might throw and we need to record the appropriate
```

- EN: Lines 129-144 introduces executable logic in routines such as `toPyJitFuture`; binds C++ behavior into Python-facing APIs with pybind or CPython helpers; performs validation and error handling to keep distributed state consistent.
- CN: 第 129-144 行在 `toPyJitFuture` 等例程中引入具体执行逻辑；通过 pybind 或 CPython 辅助接口把 C++ 行为绑定到 Python API；执行校验与错误处理，以保持分布式状态一致。

### Lines 145-160 / 第 145-160 行

```cpp
145:             // exception.
146:             IValue ivalue;
147:             try {
148:               ivalue = toPyIValue(message);
149:             } catch (py::error_already_set& e) {
150:               py::gil_scoped_acquire acquire;
151:               // FIXME: this is a temporary solution to add a special-case for
152:               // ValueError and TypeError, as those are already used in our
153:               // tests. We should have a more comprehensive coverage for other
154:               // types of exceptions as well.
155:               if (e.matches(PyExc_ValueError)) {
156:                 child->setErrorIfNeeded(
157:                     std::make_exception_ptr(pybind11::value_error(e.what())));
158:               } else if (e.matches(PyExc_TypeError)) {
159:                 child->setErrorIfNeeded(
160:                     std::make_exception_ptr(pybind11::type_error(e.what())));
```

- EN: Lines 145-160 adds callable APIs, operators, or helper routines for this part of the file; binds C++ behavior into Python-facing APIs with pybind or CPython helpers.
- CN: 第 145-160 行为本段代码补充可调用 API、操作符或辅助例程；通过 pybind 或 CPython 辅助接口把 C++ 行为绑定到 Python API。

### Lines 161-176 / 第 161-176 行

```cpp
161:               } else {
162:                 // py::error_already_set requires GIL to destruct, take special
163:                 // care.
164:                 child->setErrorIfNeeded(
165:                     std::make_exception_ptr(std::runtime_error(e.what())));
166:               }
167:               e.restore();
168:               PyErr_Clear();
169:               return;
170:             } catch (std::exception&) {
171:               child->setErrorIfNeeded(std::current_exception());
172:               return;
173:             }
174: 
175:             child->markCompleted(ivalue, future.storages());
176:           }
```

- EN: Lines 161-176 introduces executable logic in routines such as `PyErr_Clear`; binds C++ behavior into Python-facing APIs with pybind or CPython helpers.
- CN: 第 161-176 行在 `PyErr_Clear` 等例程中引入具体执行逻辑；通过 pybind 或 CPython 辅助接口把 C++ 行为绑定到 Python API。

### Lines 177-192 / 第 177-192 行

```cpp
177:         }));
178:     return child;
179:   } else {
180:     return messageJitFuture->then(
181:         at::wrapPropagateTLSState([](JitFuture& future) {
182:           if (future.hasError()) {
183:             std::rethrow_exception(future.exception_ptr());
184:           } else {
185:             return IValue();
186:           }
187:         }),
188:         NoneType::get());
189:   }
190: }
191: 
192: c10::intrusive_ptr<JitFuture> pyRpcBuiltin(
```

- EN: Lines 177-192 introduces executable logic in routines such as `IValue`; returns computed state or forwards results to the surrounding caller.
- CN: 第 177-192 行在 `IValue` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 193-208 / 第 193-208 行

```cpp
193:     const WorkerInfo& dst,
194:     const std::string& opName,
195:     const py::args& args,
196:     const py::kwargs& kwargs,
197:     const float rpcTimeoutSeconds) {
198:   DCHECK(PyGILState_Check());
199:   Stack stack;
200:   auto op = matchBuiltinOp(opName, args, kwargs, stack);
201:   // Release GIL since args and kwargs processing is done.
202:   py::gil_scoped_release release;
203:   auto scriptCall = std::make_unique<ScriptCall>(op, std::move(stack));
204:   auto agent = RpcAgent::getCurrentRpcAgent();
205:   return toPyJitFuture(sendMessageWithAutograd(
206:       *agent,
207:       dst,
208:       std::move(*scriptCall).toMessage(),
```

- EN: Lines 193-208 adds callable APIs, operators, or helper routines for this part of the file; binds C++ behavior into Python-facing APIs with pybind or CPython helpers.
- CN: 第 193-208 行为本段代码补充可调用 API、操作符或辅助例程；通过 pybind 或 CPython 辅助接口把 C++ 行为绑定到 Python API。

### Lines 209-224 / 第 209-224 行

```cpp
209:       false,
210:       rpcTimeoutSeconds));
211: }
212: 
213: c10::intrusive_ptr<JitFuture> pyRpcPythonUdf(
214:     const WorkerInfo& dst,
215:     std::string& pickledPythonUDF,
216:     std::vector<torch::Tensor>& tensors,
217:     const float rpcTimeoutSeconds,
218:     const bool isAsyncExecution) {
219:   DCHECK(!PyGILState_Check());
220:   auto serializedPyObj =
221:       SerializedPyObj(std::move(pickledPythonUDF), std::move(tensors));
222:   auto pythonCall = std::make_unique<PythonCall>(
223:       std::move(serializedPyObj), isAsyncExecution);
224: 
```

- EN: Lines 209-224 introduces executable logic in routines such as `pyRpcPythonUdf`.
- CN: 第 209-224 行在 `pyRpcPythonUdf` 等例程中引入具体执行逻辑。

### Lines 225-240 / 第 225-240 行

```cpp
225:   auto agent = RpcAgent::getCurrentRpcAgent();
226:   return toPyJitFuture(sendMessageWithAutograd(
227:       *agent,
228:       dst,
229:       std::move(*pythonCall).toMessage(),
230:       true /*forceGradRecording*/,
231:       rpcTimeoutSeconds));
232: }
233: 
234: c10::intrusive_ptr<JitFuture> pyRpcTorchscript(
235:     const std::string& dstWorkerName,
236:     const std::string& qualifiedNameStr,
237:     const py::tuple& argsTuple,
238:     const py::dict& kwargsDict,
239:     const float rpcTimeoutSeconds,
240:     const bool isAsyncExecution) {
```

- EN: Lines 225-240 introduces executable logic in routines such as `pyRpcTorchscript`; binds C++ behavior into Python-facing APIs with pybind or CPython helpers.
- CN: 第 225-240 行在 `pyRpcTorchscript` 等例程中引入具体执行逻辑；通过 pybind 或 CPython 辅助接口把 C++ 行为绑定到 Python API。

### Lines 241-256 / 第 241-256 行

```cpp
241:   // No need to catch exception here, if function can not be found,
242:   // exception will be thrown in get_function() call; if args do not match
243:   // with function schema, exception will be thrown in
244:   // createStackForSchema() call.
245:   DCHECK(!PyGILState_Check());
246:   const c10::QualifiedName qualifiedName(qualifiedNameStr);
247:   auto functionSchema = PythonRpcHandler::getInstance()
248:                             .jitCompilationUnit()
249:                             ->get_function(qualifiedName)
250:                             .getSchema();
251:   Stack stack;
252:   {
253:     // Acquire GIL for py::args and py::kwargs processing.
254:     py::gil_scoped_acquire acquire;
255:     stack = torch::jit::createStackForSchema(
256:         functionSchema,
```

- EN: Lines 241-256 adds callable APIs, operators, or helper routines for this part of the file; binds C++ behavior into Python-facing APIs with pybind or CPython helpers.
- CN: 第 241-256 行为本段代码补充可调用 API、操作符或辅助例程；通过 pybind 或 CPython 辅助接口把 C++ 行为绑定到 Python API。

### Lines 257-272 / 第 257-272 行

```cpp
257:         argsTuple.cast<py::args>(),
258:         kwargsDict.cast<py::kwargs>(),
259:         std::nullopt);
260:   }
261:   DCHECK(!PyGILState_Check());
262:   c10::intrusive_ptr<c10::ivalue::Future> fut = rpcTorchscript(
263:       dstWorkerName,
264:       qualifiedName,
265:       functionSchema,
266:       std::move(stack),
267:       rpcTimeoutSeconds,
268:       isAsyncExecution);
269:   return fut;
270: }
271: 
272: PyRRef pyRemoteBuiltin(
```

- EN: Lines 257-272 adds callable APIs, operators, or helper routines for this part of the file; binds C++ behavior into Python-facing APIs with pybind or CPython helpers.
- CN: 第 257-272 行为本段代码补充可调用 API、操作符或辅助例程；通过 pybind 或 CPython 辅助接口把 C++ 行为绑定到 Python API。

### Lines 273-288 / 第 273-288 行

```cpp
273:     const WorkerInfo& dst,
274:     const std::string& opName,
275:     const float rpcTimeoutSeconds,
276:     const py::args& args,
277:     const py::kwargs& kwargs) {
278:   DCHECK(PyGILState_Check());
279:   Stack stack;
280:   auto op = matchBuiltinOp(opName, args, kwargs, stack);
281:   // Release GIL since args and kwargs processing is done.
282:   py::gil_scoped_release release;
283:   TypePtr returnType = op->schema().returns()[0].type();
284: 
285:   auto& ctx = RRefContext::getInstance();
286:   auto agent = RpcAgent::getCurrentRpcAgent();
287: 
288:   if (ctx.getWorkerId() != dst.id_) {
```

- EN: Lines 273-288 adds callable APIs, operators, or helper routines for this part of the file; binds C++ behavior into Python-facing APIs with pybind or CPython helpers.
- CN: 第 273-288 行为本段代码补充可调用 API、操作符或辅助例程；通过 pybind 或 CPython 辅助接口把 C++ 行为绑定到 Python API。

### Lines 289-304 / 第 289-304 行

```cpp
289:     auto userRRef = ctx.createUserRRef(dst.id_, returnType);
290: 
291:     auto scriptRemoteCall = std::make_unique<ScriptRemoteCall>(
292:         op, std::move(stack), userRRef->rrefId(), userRRef->forkId());
293: 
294:     auto jitFuture = sendMessageWithAutograd(
295:         *agent,
296:         dst,
297:         std::move(*scriptRemoteCall).toMessage(),
298:         /*forceGradRecord */ false,
299:         /* timeout */ rpcTimeoutSeconds);
300: 
301:     userRRef->registerOwnerCreationFuture(jitFuture);
302:     ctx.addPendingUser(userRRef->forkId(), userRRef);
303:     jitFuture->addCallback(at::wrapPropagateTLSState(
304:         [forkId{userRRef->forkId()}](JitFuture& future) {
```

- EN: Lines 289-304 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 289-304 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 305-320 / 第 305-320 行

```cpp
305:           callback::confirmPendingUser(future, forkId);
306:         }));
307:     return PyRRef(userRRef);
308:   } else {
309:     auto ownerRRef = ctx.createOwnerRRef(returnType);
310:     // prevent this owner RRef being deleted due to other forks
311:     ctx.addSelfAsFork(ownerRRef);
312: 
313:     auto scriptRemoteCall = std::make_unique<ScriptRemoteCall>(
314:         op, std::move(stack), ownerRRef->rrefId(), ownerRRef->rrefId());
315:     auto jitFuture = sendMessageWithAutograd(
316:         *agent,
317:         dst,
318:         std::move(*scriptRemoteCall).toMessage(),
319:         /* forceGradRecord */ false,
320:         /* timeout */ rpcTimeoutSeconds);
```

- EN: Lines 305-320 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 305-320 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 321-336 / 第 321-336 行

```cpp
321: 
322:     ownerRRef->registerOwnerCreationFuture(jitFuture);
323:     // Builtin operators does not return py::object, and hence does not require
324:     // GIL for destructing the potentially deleted OwerRRef.
325:     jitFuture->addCallback(at::wrapPropagateTLSState(
326:         [ownerRRefId = ownerRRef->rrefId()](JitFuture& future) {
327:           callback::finishCreatingOwnerRRef(future, ownerRRefId);
328:         }));
329:     return PyRRef(ownerRRef);
330:   }
331: }
332: 
333: PyRRef pyRemotePythonUdf(
334:     const WorkerInfo& dst,
335:     std::string& pickledPythonUDF,
336:     std::vector<torch::Tensor>& tensors,
```

- EN: Lines 321-336 adds callable APIs, operators, or helper routines for this part of the file; binds C++ behavior into Python-facing APIs with pybind or CPython helpers.
- CN: 第 321-336 行为本段代码补充可调用 API、操作符或辅助例程；通过 pybind 或 CPython 辅助接口把 C++ 行为绑定到 Python API。

### Lines 337-352 / 第 337-352 行

```cpp
337:     const float rpcTimeoutSeconds,
338:     const bool isAsyncExecution) {
339:   DCHECK(!PyGILState_Check());
340:   auto& ctx = RRefContext::getInstance();
341:   auto serializedPyObj =
342:       SerializedPyObj(std::move(pickledPythonUDF), std::move(tensors));
343: 
344:   if (ctx.getWorkerId() != dst.id_) {
345:     auto userRRef = ctx.createUserRRef(dst.id_, PyObjectType::get());
346:     auto jitFuture = sendPythonRemoteCall(
347:         dst,
348:         std::move(serializedPyObj),
349:         userRRef->rrefId().toIValue(),
350:         userRRef->forkId().toIValue(),
351:         rpcTimeoutSeconds,
352:         isAsyncExecution);
```

- EN: Lines 337-352 adds callable APIs, operators, or helper routines for this part of the file; binds C++ behavior into Python-facing APIs with pybind or CPython helpers.
- CN: 第 337-352 行为本段代码补充可调用 API、操作符或辅助例程；通过 pybind 或 CPython 辅助接口把 C++ 行为绑定到 Python API。

### Lines 353-368 / 第 353-368 行

```cpp
353: 
354:     userRRef->registerOwnerCreationFuture(jitFuture);
355:     ctx.addPendingUser(userRRef->forkId(), userRRef);
356:     jitFuture->addCallback(at::wrapPropagateTLSState(
357:         [forkId{userRRef->forkId()}](JitFuture& future) {
358:           callback::confirmPendingUser(future, forkId);
359:         }));
360:     return PyRRef(userRRef);
361:   } else {
362:     // Sending remote message to self
363:     auto ownerRRef = ctx.createOwnerRRef(PyObjectType::get());
364:     // prevent this owner RRef being deleted due to other forks
365:     ctx.addSelfAsFork(ownerRRef);
366:     auto jitFuture = sendPythonRemoteCall(
367:         dst,
368:         std::move(serializedPyObj),
```

- EN: Lines 353-368 adds callable APIs, operators, or helper routines for this part of the file; binds C++ behavior into Python-facing APIs with pybind or CPython helpers.
- CN: 第 353-368 行为本段代码补充可调用 API、操作符或辅助例程；通过 pybind 或 CPython 辅助接口把 C++ 行为绑定到 Python API。

### Lines 369-384 / 第 369-384 行

```cpp
369:         ownerRRef->rrefId().toIValue(),
370:         ownerRRef->rrefId().toIValue(),
371:         rpcTimeoutSeconds,
372:         isAsyncExecution);
373: 
374:     ownerRRef->registerOwnerCreationFuture(jitFuture);
375:     jitFuture->addCallback(at::wrapPropagateTLSState(
376:         [ownerRRefId = ownerRRef->rrefId()](JitFuture& future) {
377:           auto deletedRRef =
378:               callback::finishCreatingOwnerRRef(future, ownerRRefId);
379:           if (deletedRRef && deletedRRef->isPyObj()) {
380:             py::gil_scoped_acquire ag;
381:             deletedRRef.reset();
382:           }
383:         }));
384:     return PyRRef(ownerRRef);
```

- EN: Lines 369-384 adds callable APIs, operators, or helper routines for this part of the file; binds C++ behavior into Python-facing APIs with pybind or CPython helpers.
- CN: 第 369-384 行为本段代码补充可调用 API、操作符或辅助例程；通过 pybind 或 CPython 辅助接口把 C++ 行为绑定到 Python API。

### Lines 385-400 / 第 385-400 行

```cpp
385:   }
386: }
387: 
388: PyRRef pyRemoteTorchscript(
389:     const std::string& dstWorkerName,
390:     const std::string& qualifiedNameStr,
391:     const float rpcTimeoutSeconds,
392:     const bool isAsyncExecution,
393:     const py::args& args,
394:     const py::kwargs& kwargs) {
395:   DCHECK(!PyGILState_Check());
396:   auto qualifiedName = c10::QualifiedName(qualifiedNameStr);
397:   auto functionSchema = PythonRpcHandler::getInstance()
398:                             .jitCompilationUnit()
399:                             ->get_function(qualifiedName)
400:                             .getSchema();
```

- EN: Lines 385-400 introduces executable logic in routines such as `pyRemoteTorchscript`; binds C++ behavior into Python-facing APIs with pybind or CPython helpers.
- CN: 第 385-400 行在 `pyRemoteTorchscript` 等例程中引入具体执行逻辑；通过 pybind 或 CPython 辅助接口把 C++ 行为绑定到 Python API。

### Lines 401-416 / 第 401-416 行

```cpp
401:   Stack stack;
402:   {
403:     // Acquire GIL for py::args and py::kwargs processing.
404:     py::gil_scoped_acquire ag;
405:     stack = torch::jit::createStackForSchema(
406:         functionSchema, args, kwargs, std::nullopt);
407:   }
408:   DCHECK(!PyGILState_Check());
409:   auto rrefPtr = remoteTorchscript(
410:       dstWorkerName,
411:       qualifiedName,
412:       functionSchema,
413:       stack,
414:       rpcTimeoutSeconds,
415:       isAsyncExecution);
416:   return PyRRef(rrefPtr);
```

- EN: Lines 401-416 adds callable APIs, operators, or helper routines for this part of the file; binds C++ behavior into Python-facing APIs with pybind or CPython helpers.
- CN: 第 401-416 行为本段代码补充可调用 API、操作符或辅助例程；通过 pybind 或 CPython 辅助接口把 C++ 行为绑定到 Python API。

### Lines 417-419 / 第 417-419 行

```cpp
417: }
418: 
419: } // namespace torch::distributed::rpc
```

- EN: Lines 417-419 opens or closes namespaces to place the code in the correct distributed component.
- CN: 第 417-419 行打开或关闭命名空间，以便将代码放入正确的分布式组件中。

## Key Concepts / 关键概念

- EN: Subsystem: distributed RPC layer.
- CN: 子系统：分布式 RPC 层。
- EN: File role: implementation/registration.
- CN: 文件角色：实现/注册。
- EN: Primary symbols: `toPyIValue`, `TORCH_CHECK`, `matchBuiltinOp`, `sendPythonRemoteCall`, `toPyJitFuture`, `PyErr_Clear`
- CN: 核心符号：`toPyIValue`、`TORCH_CHECK`、`matchBuiltinOp`、`sendPythonRemoteCall`、`toPyJitFuture`、`PyErr_Clear`
- EN: Notable themes: Python bindings, RPC/RRef semantics.
- CN: 值得关注的主题：Python 绑定、RPC/RRef 语义。

## Dependencies / 依赖关系

- Internal distributed headers / 内部分布式头文件: `torch/csrc/distributed/autograd/utils.h`, `torch/csrc/distributed/rpc/message.h`, `torch/csrc/distributed/rpc/python_call.h`, `torch/csrc/distributed/rpc/python_functions.h`, `torch/csrc/distributed/rpc/python_remote_call.h`, `torch/csrc/distributed/rpc/python_resp.h`, `torch/csrc/distributed/rpc/python_rpc_handler.h`, `torch/csrc/distributed/rpc/rref_context.h`, `torch/csrc/distributed/rpc/script_call.h`, `torch/csrc/distributed/rpc/script_remote_call.h`, `torch/csrc/distributed/rpc/script_resp.h`, `torch/csrc/distributed/rpc/torchscript_functions.h`, ... (+1 more)
- PyTorch/ATen/C10 headers / PyTorch/ATen/C10 头文件: `ATen/ThreadLocalState.h`, `torch/csrc/jit/runtime/operator.h`
- External or system headers / 外部或系统头文件: `exception`
- Local symbols / 本地符号: `toPyIValue`, `TORCH_CHECK`, `matchBuiltinOp`, `sendPythonRemoteCall`, `toPyJitFuture`, `PyErr_Clear`, `IValue`, `pyRpcBuiltin`