# request_callback_no_python.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `torch/csrc/distributed/rpc/request_callback_no_python.cpp`
- Repository: `pytorch` (`/root/xw/pytorch`)
- Purpose (EN): This file provides implementation logic for request callback no python in the distributed RPC layer. Key types include `DistAutogradContextGuard`.
- 用途 (CN): 该文件在分布式 RPC 层中提供request callback no python 的实现逻辑。 关键类型包括 `DistAutogradContextGuard`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

```cpp
1: #include <torch/csrc/distributed/rpc/request_callback_no_python.h>
2: 
3: #include <c10/core/StreamGuard.h>
4: #include <torch/csrc/distributed/autograd/context/container.h>
5: #include <torch/csrc/distributed/autograd/engine/dist_engine.h>
6: #include <torch/csrc/distributed/autograd/rpc_messages/cleanup_autograd_context_req.h>
7: #include <torch/csrc/distributed/autograd/rpc_messages/cleanup_autograd_context_resp.h>
8: #include <torch/csrc/distributed/autograd/rpc_messages/propagate_gradients_req.h>
9: #include <torch/csrc/distributed/autograd/rpc_messages/propagate_gradients_resp.h>
10: #include <torch/csrc/distributed/autograd/rpc_messages/rpc_with_autograd.h>
11: #include <torch/csrc/distributed/autograd/utils.h>
12: #include <torch/csrc/distributed/rpc/profiler/server_process_global_profiler.h>
13: #include <torch/csrc/distributed/rpc/rpc_agent.h>
14: #include <torch/csrc/distributed/rpc/rref_context.h>
15: #include <torch/csrc/distributed/rpc/rref_proto.h>
16: #include <torch/csrc/distributed/rpc/script_resp.h>
17: #include <torch/csrc/distributed/rpc/utils.h>
18: 
19: #include <utility>
20: 
```

- EN: Lines 1-20 pulls in the headers required by this translation unit or interface.
- CN: 第 1-20 行引入该实现单元或接口所需的头文件。

### Lines 21-40 / 第 21-40 行

```cpp
21: namespace torch::distributed::rpc {
22: 
23: using namespace torch::distributed::autograd;
24: using namespace torch::autograd::profiler;
25: 
26: // When request message has autograd info, processMessage() will set up valid
27: // current context id properly. This struct is used to clean up current context
28: // id after processMessage() is done.
29: struct DistAutogradContextGuard {
30:   explicit DistAutogradContextGuard(int64_t ctxId) {
31:     auto& container = DistAutogradContainer::getInstance();
32:     prevCtxId_ = container.currentContextId();
33:     container.forceCurrentContextId(ctxId);
34:   }
35:   DistAutogradContextGuard(const DistAutogradContextGuard&) = delete;
36:   DistAutogradContextGuard(DistAutogradContextGuard&&) = delete;
37:   DistAutogradContextGuard& operator=(const DistAutogradContextGuard&) = delete;
38:   DistAutogradContextGuard& operator=(DistAutogradContextGuard&&) = delete;
39:   ~DistAutogradContextGuard() {
40:     auto& container = DistAutogradContainer::getInstance();
```

- EN: Lines 21-40 opens or closes namespaces to place the code in the correct distributed component; declares or defines types such as `DistAutogradContextGuard`; introduces executable logic in routines such as `DistAutogradContextGuard`, `~DistAutogradContextGuard`.
- CN: 第 21-40 行打开或关闭命名空间，以便将代码放入正确的分布式组件中；声明或定义了 `DistAutogradContextGuard` 等类型；在 `DistAutogradContextGuard`、`~DistAutogradContextGuard` 等例程中引入具体执行逻辑。

### Lines 41-60 / 第 41-60 行

```cpp
41:     container.forceCurrentContextId(prevCtxId_);
42:   }
43: 
44:   int64_t prevCtxId_;
45: };
46: 
47: std::unique_ptr<RpcCommandBase> RequestCallbackNoPython::
48:     deserializePythonRpcCommand(
49:         std::unique_ptr<RpcCommandBase> rpc,
50:         const MessageType& messageType) const {
51:   TORCH_CHECK(
52:       messageType != MessageType::PYTHON_CALL &&
53:           messageType != MessageType::PYTHON_REMOTE_CALL,
54:       "Python calls are not supported!");
55:   return rpc;
56: }
57: 
58: c10::intrusive_ptr<JitFuture> RequestCallbackNoPython::processMessage(
59:     Message& request,
60:     std::vector<c10::Stream> streams) const {
```

- EN: Lines 41-60 introduces executable logic in routines such as `deserializePythonRpcCommand`, `TORCH_CHECK`; performs validation and error handling to keep distributed state consistent.
- CN: 第 41-60 行在 `deserializePythonRpcCommand`、`TORCH_CHECK` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 61-80 / 第 61-80 行

```cpp
61:   // We need two futures here because it could pause twice when processing a
62:   // RPC message:
63:   //  1) waiting for all RRefs in the arguments to become confirmed;
64:   //  2) waiting for processRpc to finish.
65:   auto& rrefContext = RRefContext::getInstance();
66:   try {
67:     rrefContext.recordThreadLocalPendingRRefs();
68:     // Deserialize PythonUDF here to trigger RRef unpickling
69:     std::unique_ptr<RpcCommandBase> rpc = deserializePythonRpcCommand(
70:         deserializeRequest(request), request.type());
71:     auto rrefsReadyFuture = rrefContext.waitForThreadLocalPendingRRefs();
72: 
73:     auto retFuture = rrefsReadyFuture->thenAsync(
74:         [this,
75:          // std::function must be copyable, hence has to cast the unique_ptr to
76:          // a shared_ptr here.
77:          rpc = std::shared_ptr<RpcCommandBase>(std::move(rpc)),
78:          messageType = request.type(),
79:          streams = std::move(streams)](JitFuture& /* unused */) mutable {
80:           // The cost of pre-request check is minimal thanks to
```

- EN: Lines 61-80 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 61-80 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 81-100 / 第 81-100 行

```cpp
81:           // std::shared_lock. The cost is in magnitude
82:           // of 10us.
83:           auto serverProcessGlobalProfilerStateStackEntryPtr =
84:               profiler::processglobal::StateStackEntry::current();
85:           // If server global profiler is enabled, we further pay the
86:           // cost of thread local profiler state initialization.
87:           if (serverProcessGlobalProfilerStateStackEntryPtr) {
88:             // Initialize thread-local profiler state from process-global
89:             // profiler state.
90:             enableProfilerLegacy(
91:                 serverProcessGlobalProfilerStateStackEntryPtr->statePtr()
92:                     ->config());
93:           }
94: 
95:           auto retFuture = processRpcWithErrors(*rpc, messageType, streams);
96: 
97:           // Response message has been sent at this moment, this post-response
98:           // work doesn't affect RPC trip time.
99:           if (serverProcessGlobalProfilerStateStackEntryPtr) {
100:             // Restore thread-local profiler state.
```

- EN: Lines 81-100 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 81-100 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 101-120 / 第 101-120 行

```cpp
101:             thread_event_lists event_lists = disableProfilerLegacy();
102:             // Put thread_local event_lists into the process-global profiler
103:             // state.
104:             profiler::processglobal::pushResultRecursive(
105:                 serverProcessGlobalProfilerStateStackEntryPtr, event_lists);
106:           }
107: 
108:           return retFuture;
109:         },
110:         c10::getCustomClassType<c10::intrusive_ptr<Message>>());
111: 
112:     auto retFutureWithMessageId = retFuture->then(
113:         [id = request.id()](JitFuture& future) {
114:           c10::intrusive_ptr<Message> message =
115:               future.value().toCustomClass<Message>();
116:           message->setId(id);
117:           return withStorages(message);
118:         },
119:         c10::getCustomClassType<c10::intrusive_ptr<Message>>());
120: 
```

- EN: Lines 101-120 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 101-120 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 121-140 / 第 121-140 行

```cpp
121:     return retFutureWithMessageId;
122:   } catch (std::exception& e) {
123:     rrefContext.clearRecordedPendingRRefsOnError();
124:     return asFuture(handleError(e, request.type(), request.id()));
125:   }
126: }
127: 
128: c10::intrusive_ptr<JitFuture> RequestCallbackNoPython::processRpcWithErrors(
129:     RpcCommandBase& rpc,
130:     const MessageType& messageType,
131:     const std::vector<c10::Stream>& streams) const {
132:   try {
133:     return processRpc(rpc, messageType, streams);
134:   } catch (std::exception& e) {
135:     // Pass a dummy message ID since it will be overwritten anyways.
136:     return asFuture(handleError(e, messageType, -1));
137:   }
138: }
139: 
140: c10::intrusive_ptr<JitFuture> RequestCallbackNoPython::processScriptCall(
```

- EN: Lines 121-140 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 121-140 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 141-160 / 第 141-160 行

```cpp
141:     RpcCommandBase& rpc,
142:     const std::vector<c10::Stream>& streams) const {
143:   auto& scriptCall = static_cast<ScriptCall&>(rpc);
144: 
145:   TORCH_CHECK(
146:       scriptCall.hasOp(), "Only supports the case where ScriptCall has an op");
147:   auto future =
148:       runJitOperator(*scriptCall.op(), scriptCall.stackRef(), streams);
149: 
150:   return future->then(
151:       [](JitFuture& future) {
152:         return withStorages(ScriptResp(future.value()).toMessage());
153:       },
154:       c10::getCustomClassType<c10::intrusive_ptr<Message>>());
155: }
156: 
157: c10::intrusive_ptr<JitFuture> RequestCallbackNoPython::processPythonCall(
158:     RpcCommandBase& rpc,
159:     const std::vector<c10::Stream>& /* unused */) const {
160:   C10_THROW_ERROR(Error, "Python call not supported!");
```

- EN: Lines 141-160 introduces executable logic in routines such as `C10_THROW_ERROR`; performs validation and error handling to keep distributed state consistent.
- CN: 第 141-160 行在 `C10_THROW_ERROR` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 161-180 / 第 161-180 行

```cpp
161: }
162: 
163: c10::intrusive_ptr<JitFuture> RequestCallbackNoPython::processPythonRemoteCall(
164:     RpcCommandBase& rpc,
165:     const std::vector<c10::Stream>& /* unused */) const {
166:   C10_THROW_ERROR(Error, "Python call not supported!");
167: }
168: 
169: c10::intrusive_ptr<JitFuture> RequestCallbackNoPython::assignOwnerRRef(
170:     const RRefId& rrefId,
171:     const RRefId& forkId,
172:     const c10::intrusive_ptr<JitFuture>& valueFuture) const {
173:   auto& ctx = RRefContext::getInstance();
174: 
175:   c10::intrusive_ptr<OwnerRRef> ownerRRef;
176:   if (rrefId == forkId) {
177:     // Creating an owner RRef on self, should already exist in owners map
178:     ownerRRef =
179:         fromRRefInterface(ctx.getOwnerRRef(rrefId, /* forceCreated */ true)
180:                               ->constValue()
```

- EN: Lines 161-180 introduces executable logic in routines such as `C10_THROW_ERROR`; performs validation and error handling to keep distributed state consistent.
- CN: 第 161-180 行在 `C10_THROW_ERROR` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 181-200 / 第 181-200 行

```cpp
181:                               .toRRef());
182:   } else {
183:     ownerRRef = ctx.getOrCreateOwnerRRef(rrefId, valueFuture->elementType());
184:     // Caller is a user and callee is the owner, add fork
185:     //
186:     // NB: rrefId == forkId is true if and only if calling remote to self.
187:     // In that case both the caller and the callee will access the
188:     // OwnerRRef. Hence, on the callee side (here), it should not call
189:     // addForkOfOwner as it is not a fork. To allow callee to distinguish
190:     // when this request is sent to self, the caller will set forkId using
191:     // rrefId (OwnerRRef does not have a forkId anyway).
192:     ctx.addForkOfOwner(rrefId, forkId);
193:   }
194: 
195:   return valueFuture->then(
196:       [ownerRRef, rrefId, forkId](JitFuture& future) {
197:         if (future.hasError()) {
198:           ownerRRef->setError(future.exception_ptr());
199:         } else {
200:           ownerRRef->setValue(future.value());
```

- EN: Lines 181-200 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 181-200 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 201-220 / 第 201-220 行

```cpp
201:         }
202:         return withStorages(RemoteRet(rrefId, forkId).toMessage());
203:       },
204:       c10::getCustomClassType<c10::intrusive_ptr<Message>>());
205: }
206: 
207: c10::intrusive_ptr<JitFuture> RequestCallbackNoPython::processScriptRemoteCall(
208:     RpcCommandBase& rpc,
209:     const std::vector<c10::Stream>& streams) const {
210:   auto& scriptRemoteCall = static_cast<ScriptRemoteCall&>(rpc);
211: 
212:   TORCH_CHECK(
213:       scriptRemoteCall.hasOp(), "ScriptRemoteCall needs to have an op!");
214:   auto future = runJitOperator(
215:       *scriptRemoteCall.op(), scriptRemoteCall.stackRef(), streams);
216: 
217:   return assignOwnerRRef(
218:       scriptRemoteCall.retRRefId(), scriptRemoteCall.retForkId(), future);
219: }
220: 
```

- EN: Lines 201-220 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 201-220 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 221-240 / 第 221-240 行

```cpp
221: c10::intrusive_ptr<JitFuture> RequestCallbackNoPython::retrieveOwnerRRef(
222:     const RRefId& rrefId) const {
223:   auto& ctx = RRefContext::getInstance();
224: 
225:   auto rrefFuture = ctx.getOwnerRRef(rrefId);
226: 
227:   at::TypePtr type = rrefFuture->elementType();
228:   TORCH_INTERNAL_ASSERT(type->kind() == at::RRefType::Kind);
229:   return rrefFuture->thenAsync(
230:       [](JitFuture& rrefFuture) {
231:         c10::intrusive_ptr<OwnerRRef> rref =
232:             fromRRefInterface(rrefFuture.value().toRRef());
233:         return rref->getFuture();
234:       },
235:       type->cast<at::RRefType>()->getElementType());
236: }
237: 
238: c10::intrusive_ptr<JitFuture> RequestCallbackNoPython::
239:     processScriptRRefFetchCall(RpcCommandBase& rpc) const {
240:   auto& srf = static_cast<ScriptRRefFetchCall&>(rpc);
```

- EN: Lines 221-240 introduces executable logic in routines such as `processScriptRRefFetchCall`; returns computed state or forwards results to the surrounding caller.
- CN: 第 221-240 行在 `processScriptRRefFetchCall` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 241-260 / 第 241-260 行

```cpp
241: 
242:   auto future = retrieveOwnerRRef(srf.rrefId());
243: 
244:   return future->then(
245:       [](JitFuture& future) {
246:         return withStorages(ScriptRRefFetchRet({future.value()}).toMessage());
247:       },
248:       c10::getCustomClassType<c10::intrusive_ptr<Message>>());
249: }
250: 
251: c10::intrusive_ptr<JitFuture> RequestCallbackNoPython::
252:     processPythonRRefFetchCall(RpcCommandBase& rpc) const {
253:   C10_THROW_ERROR(Error, "Python call not supported!");
254: }
255: 
256: c10::intrusive_ptr<JitFuture> RequestCallbackNoPython::processRRefUserDelete(
257:     RpcCommandBase& rpc) const {
258:   auto& rud = static_cast<RRefUserDelete&>(rpc);
259:   auto& ctx = RRefContext::getInstance();
260:   auto deletedRRef = ctx.delForkOfOwner(rud.rrefId(), rud.forkId());
```

- EN: Lines 241-260 introduces executable logic in routines such as `processPythonRRefFetchCall`, `C10_THROW_ERROR`; performs validation and error handling to keep distributed state consistent.
- CN: 第 241-260 行在 `processPythonRRefFetchCall`、`C10_THROW_ERROR` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 261-280 / 第 261-280 行

```cpp
261:   handleRRefDelete(deletedRRef);
262:   return asFuture(RRefAck().toMessage());
263: }
264: 
265: void RequestCallbackNoPython::handleRRefDelete(
266:     c10::intrusive_ptr<RRef>& rref) const {
267:   TORCH_CHECK(!rref->isPyObj(), "RRefs with python objects not supported!");
268: }
269: 
270: c10::intrusive_ptr<JitFuture> RequestCallbackNoPython::processRRefChildAccept(
271:     RpcCommandBase& rpc) const {
272:   auto& rca = static_cast<RRefChildAccept&>(rpc);
273:   auto& ctx = RRefContext::getInstance();
274:   ctx.delPendingChild(rca.forkId());
275:   return asFuture(RRefAck().toMessage());
276: }
277: 
278: c10::intrusive_ptr<JitFuture> RequestCallbackNoPython::processRRefForkRequest(
279:     RpcCommandBase& rpc) const {
280:   auto& rfr = static_cast<RRefForkRequest&>(rpc);
```

- EN: Lines 261-280 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 261-280 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 281-300 / 第 281-300 行

```cpp
281:   auto& ctx = RRefContext::getInstance();
282:   ctx.addForkOfOwnerIfNotPresent(rfr.rrefId(), rfr.forkId());
283:   return asFuture(RRefAck().toMessage());
284: }
285: 
286: c10::intrusive_ptr<JitFuture> RequestCallbackNoPython::
287:     processForwardAutogradReq(
288:         RpcCommandBase& rpc,
289:         const std::vector<c10::Stream>& streams) const {
290:   auto& rpcWithAutograd = static_cast<RpcWithAutograd&>(rpc);
291: 
292:   // Need to reverse the device map for the backward pass of distributed
293:   // autograd.
294:   DeviceMap reverseDeviceMap;
295:   for (const auto& mapEntry : rpcWithAutograd.deviceMap()) {
296:     reverseDeviceMap.insert({mapEntry.second, mapEntry.first});
297:   }
298: 
299:   // Attach 'recv' autograd function.
300:   auto autogradContext = addRecvRpcBackward(
```

- EN: Lines 281-300 introduces executable logic in routines such as `processForwardAutogradReq`; returns computed state or forwards results to the surrounding caller.
- CN: 第 281-300 行在 `processForwardAutogradReq` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 301-320 / 第 301-320 行

```cpp
301:       rpcWithAutograd.autogradMetadata(),
302:       rpcWithAutograd.tensors(),
303:       rpcWithAutograd.fromWorkerId(),
304:       reverseDeviceMap);
305:   // For this recv thread on server side, before processRpc(),
306:   // set current_context_id_ to be context_id passed from client.
307:   // In this way, if there is nested rpc call in python rpc call, original
308:   // context_id from client can be passed in the chain calls.
309:   TORCH_INTERNAL_ASSERT(
310:       autogradContext != nullptr,
311:       "autogradContext is nullptr, FORWARD_AUTOGRAD_REQ should always get "
312:       "or create valid autogradContext in addRecvRpcBackward.");
313: 
314:   DistAutogradContextGuard ctxGuard(autogradContext->contextId());
315: 
316:   // Process the original RPC.
317:   auto wrappedMessageType = rpcWithAutograd.wrappedMessageType();
318:   // Kick off processing for the nested RPC command.
319:   // wrappedRpcResponseFuture will be a Future<T> to the result.
320:   auto wrappedRpcResponseFuture =
```

- EN: Lines 301-320 introduces executable logic in routines such as `TORCH_INTERNAL_ASSERT`.
- CN: 第 301-320 行在 `TORCH_INTERNAL_ASSERT` 等例程中引入具体执行逻辑。

### Lines 321-340 / 第 321-340 行

```cpp
321:       processRpc(rpcWithAutograd.wrappedRpc(), wrappedMessageType, streams);
322: 
323:   auto fromWorkerId = rpcWithAutograd.fromWorkerId();
324:   // The original future needs to be marked as completed when the wrapped
325:   // one completes, with the autograd context information wrapped.
326:   auto responseFuture = wrappedRpcResponseFuture->then(
327:       [fromWorkerId, ctxId = autogradContext->contextId()](
328:           JitFuture& wrappedRpcResponseFuture) {
329:         // As this callback can be invoked by a different thread, we have to
330:         // make sure that the thread_local states in the previous thread is
331:         // correctly propagated.
332:         // NB: The execution of TorchScript functions can also run on a
333:         // different thread, which is addressed by
334:         // https://github.com/pytorch/pytorch/pull/36395
335:         // NB: when adding async UDF support, we should also propagate
336:         // thread_local states there.
337:         // TODO: Land on a general solution for RPC ThreadLocalState. See
338:         // https://github.com/pytorch/pytorch/issues/38510
339:         DistAutogradContextGuard cbCtxGuard(ctxId);
340: 
```

- EN: Lines 321-340 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 321-340 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 341-360 / 第 341-360 行

```cpp
341:         if (wrappedRpcResponseFuture.hasError()) {
342:           // Propagate error to responseFuture if we had one.
343:           std::rethrow_exception(wrappedRpcResponseFuture.exception_ptr());
344:         } else {
345:           auto msg = getMessageWithAutograd(
346:               fromWorkerId,
347:               wrappedRpcResponseFuture.value().toCustomClass<Message>(),
348:               MessageType::FORWARD_AUTOGRAD_RESP);
349:           return withStorages(std::move(msg));
350:         }
351:       },
352:       c10::getCustomClassType<c10::intrusive_ptr<Message>>());
353: 
354:   return responseFuture;
355: }
356: 
357: c10::intrusive_ptr<JitFuture> RequestCallbackNoPython::
358:     processBackwardAutogradReq(
359:         RpcCommandBase& rpc,
360:         const std::vector<c10::Stream>& streams) const {
```

- EN: Lines 341-360 introduces executable logic in routines such as `processBackwardAutogradReq`; returns computed state or forwards results to the surrounding caller.
- CN: 第 341-360 行在 `processBackwardAutogradReq` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 361-380 / 第 361-380 行

```cpp
361:   c10::MultiStreamGuard guard(streams);
362:   auto& gradientsCall = static_cast<PropagateGradientsReq&>(rpc);
363:   const auto& autogradMetadata = gradientsCall.getAutogradMetadata();
364: 
365:   // Retrieve the appropriate autograd context.
366:   auto autogradContext = DistAutogradContainer::getInstance().retrieveContext(
367:       autogradMetadata.autogradContextId);
368: 
369:   // Lookup the appropriate 'send' function to enqueue.
370:   c10::intrusive_ptr<SendRpcBackward> sendFunction =
371:       autogradContext->retrieveSendFunction(autogradMetadata.autogradMessageId);
372: 
373:   // Attach the gradients to the send function.
374:   sendFunction->setGrads(gradientsCall.getGrads());
375: 
376:   // Now execute the autograd graph using the "distributed engine."
377:   auto execFuture = DistEngine::getInstance().executeSendFunctionAsync(
378:       autogradContext, sendFunction, gradientsCall.retainGraph());
379: 
380:   // Our response is satisfied when the rpcs come back.
```

- EN: Lines 361-380 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 361-380 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 381-400 / 第 381-400 行

```cpp
381:   return execFuture->then(
382:       [](JitFuture& execFuture) {
383:         if (execFuture.hasError()) {
384:           std::rethrow_exception(execFuture.exception_ptr());
385:         } else {
386:           return withStorages(PropagateGradientsResp().toMessage());
387:         }
388:       },
389:       c10::getCustomClassType<c10::intrusive_ptr<Message>>());
390: }
391: 
392: c10::intrusive_ptr<JitFuture> RequestCallbackNoPython::
393:     processCleanupAutogradContextReq(RpcCommandBase& rpc) const {
394:   auto& cleanupContextReq = static_cast<CleanupAutogradContextReq&>(rpc);
395:   auto cleanupContextId = cleanupContextReq.getContextId();
396:   // release the context if it still exists on this thread. We need to
397:   // check if it exists since it may have been deleted by an in-flight
398:   // RPC. This can create nested RPCs if there are other nodes that get
399:   // notified to clean up their context.
400:   DistAutogradContainer::getInstance().releaseContextIfPresent(
```

- EN: Lines 381-400 introduces executable logic in routines such as `processCleanupAutogradContextReq`; returns computed state or forwards results to the surrounding caller.
- CN: 第 381-400 行在 `processCleanupAutogradContextReq` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 401-420 / 第 401-420 行

```cpp
401:       cleanupContextId);
402:   return asFuture(CleanupAutogradContextResp().toMessage());
403: }
404: 
405: c10::intrusive_ptr<JitFuture> RequestCallbackNoPython::
406:     processRunWithProfilingReq(RpcCommandBase& rpc) const {
407:   auto& rpcWithProfilingReq = static_cast<RpcWithProfilingReq&>(rpc);
408:   auto wrappedMsgType = rpcWithProfilingReq.wrappedMessageType();
409:   auto profilingConfig = rpcWithProfilingReq.getProfilingConfig();
410: 
411:   if (profilingConfig.state == ProfilerState::KINETO ||
412:       profilingConfig.state == ProfilerState::KINETO_GPU_FALLBACK ||
413:       profilingConfig.state == ProfilerState::KINETO_PRIVATEUSE1_FALLBACK) {
414:     profilingConfig = ProfilerConfig(
415:         ProfilerState::CPU,
416:         profilingConfig.report_input_shapes,
417:         profilingConfig.profile_memory);
418:   }
419: 
420:   // If requested with CUDA from caller but CUDA is not available on this
```

- EN: Lines 401-420 introduces executable logic in routines such as `processRunWithProfilingReq`; returns computed state or forwards results to the surrounding caller.
- CN: 第 401-420 行在 `processRunWithProfilingReq` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 421-440 / 第 421-440 行

```cpp
421:   // machine, fallback to CPU and log a warning instead of crashing.
422:   if (profilingConfig.state == ProfilerState::CUDA && !this->cudaAvailable()) {
423:     profilingConfig = ProfilerConfig(
424:         ProfilerState::CPU,
425:         profilingConfig.report_input_shapes,
426:         profilingConfig.profile_memory);
427: 
428:     LOG(WARNING) << "Profiler was requested to be enabled with CUDA on this "
429:                     "node, but CUDA is not available. "
430:                  << "Falling back to CPU profiling only.";
431:   }
432:   TORCH_INTERNAL_ASSERT(
433:       profilingConfig.state != ProfilerState::CUDA || this->cudaAvailable(),
434:       "Profiler state set to CUDA but CUDA not available.");
435:   const auto profilingKeyId = rpcWithProfilingReq.getProfilingId();
436:   // Enable the profiler with the config from the sender.
437:   // When enabling on the main thread, ensure profiler states are cleaned
438:   // up, but defer consolidation of all profiled events to the continuation
439:   // below.
440:   ProfilerDisableOptions requestThreadOptions(
```

- EN: Lines 421-440 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 421-440 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 441-460 / 第 441-460 行

```cpp
441:       true /* cleanup TLS state */, false /* consolidate events */);
442:   {
443:     TLSLegacyProfilerGuard g(
444:         profilingConfig, std::nullopt, requestThreadOptions);
445:     TORCH_INTERNAL_ASSERT(
446:         profilerEnabled(), "Expected profiler to be enabled!");
447:     // Kick off processing for nested work and get Future<T> result in
448:     // wrappedRpcResponseFuture
449:     auto wrappedRpcResponseFuture = processRpc(
450:         rpcWithProfilingReq.wrappedRpc(),
451:         wrappedMsgType,
452:         {}); // TODO: https://github.com/pytorch/pytorch/issues/55757
453: 
454:     auto responseFuture = wrappedRpcResponseFuture->then(
455:         at::wrapPropagateTLSState([profilingKeyId, profilingConfig](
456:                                       JitFuture& wrappedRpcResponseFuture) {
457:           std::vector<LegacyEvent> profiledEvents;
458:           // Defer consolidation of profiler events until async work has
459:           // completed (such as async UDF)
460: 
```

- EN: Lines 441-460 introduces executable logic in routines such as `g`.
- CN: 第 441-460 行在 `g` 等例程中引入具体执行逻辑。

### Lines 461-480 / 第 461-480 行

```cpp
461:           TORCH_INTERNAL_ASSERT(
462:               profilerEnabled(), "Expected profiler to be enabled!");
463: 
464:           // On continuation thread, don't clean up profiler states, since
465:           // they will be cleaned up by main thread, and consolidate all
466:           // events so we obtain asynchronously run events.
467:           ProfilerDisableOptions opts(false, true);
468:           auto event_lists = disableProfilerLegacy(opts);
469:           if (wrappedRpcResponseFuture.hasError()) {
470:             // Propagate error
471:             // No need to propagate remote events in the case of an error.
472:             std::rethrow_exception(wrappedRpcResponseFuture.exception_ptr());
473:           } else {
474:             populateRemoteProfiledEvents(
475:                 profiledEvents, profilingConfig, event_lists);
476:             auto rpcWithProfilingResp = std::make_unique<RpcWithProfilingResp>(
477:                 MessageType::RUN_WITH_PROFILING_RESP,
478:                 wrappedRpcResponseFuture.value().toCustomClass<Message>(),
479:                 profiledEvents,
480:                 profilingKeyId);
```

- EN: Lines 461-480 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 461-480 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 481-500 / 第 481-500 行

```cpp
481:             return withStorages(std::move(*rpcWithProfilingResp).toMessage());
482:           }
483:         }),
484:         c10::getCustomClassType<c10::intrusive_ptr<Message>>());
485: 
486:     return responseFuture;
487:     // Exiting the scope will disable the profiler on this thread with the
488:     // options specified above.
489:   }
490: }
491: 
492: c10::intrusive_ptr<JitFuture> RequestCallbackNoPython::processRRefBackward(
493:     RpcCommandBase& rpc) const {
494:   C10_THROW_ERROR(Error, "Python call not supported!");
495: }
496: 
497: c10::intrusive_ptr<JitFuture> RequestCallbackNoPython::processRpc(
498:     RpcCommandBase& rpc,
499:     const MessageType& messageType,
500:     const std::vector<c10::Stream>& streams) const {
```

- EN: Lines 481-500 introduces executable logic in routines such as `C10_THROW_ERROR`; performs validation and error handling to keep distributed state consistent.
- CN: 第 481-500 行在 `C10_THROW_ERROR` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 501-520 / 第 501-520 行

```cpp
501:   // TODO: RpcCommandBase should have an abstract execute() method that we can
502:   // call here instead of having another switch statement here. Even better we
503:   // could have abstract classes RpcRequest and RpcResp which inherit from
504:   // RpcCommandBase and RpcRequest declares the abstract method execute() that
505:   // we can call here. RpcResponse could have an abstract method to convert it
506:   // to a python object.
507:   switch (messageType) {
508:     case MessageType::SCRIPT_CALL: {
509:       return processScriptCall(rpc, streams);
510:     }
511:     case MessageType::PYTHON_CALL: {
512:       return processPythonCall(rpc, streams);
513:     }
514:     case MessageType::SCRIPT_REMOTE_CALL: {
515:       return processScriptRemoteCall(rpc, streams);
516:     }
517:     case MessageType::PYTHON_REMOTE_CALL: {
518:       return processPythonRemoteCall(rpc, streams);
519:     }
520:     case MessageType::SCRIPT_RREF_FETCH_CALL: {
```

- EN: Lines 501-520 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 501-520 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 521-540 / 第 521-540 行

```cpp
521:       return processScriptRRefFetchCall(rpc);
522:     }
523:     case MessageType::PYTHON_RREF_FETCH_CALL: {
524:       return processPythonRRefFetchCall(rpc);
525:     }
526:     case MessageType::RREF_USER_DELETE: {
527:       return processRRefUserDelete(rpc);
528:     }
529:     case MessageType::RREF_CHILD_ACCEPT: {
530:       return processRRefChildAccept(rpc);
531:     }
532:     case MessageType::RREF_FORK_REQUEST: {
533:       return processRRefForkRequest(rpc);
534:     }
535:     case MessageType::FORWARD_AUTOGRAD_REQ: {
536:       return processForwardAutogradReq(rpc, streams);
537:     }
538:     case MessageType::BACKWARD_AUTOGRAD_REQ: {
539:       return processBackwardAutogradReq(rpc, streams);
540:     };
```

- EN: Lines 521-540 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 521-540 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 541-560 / 第 541-560 行

```cpp
541:     case MessageType::CLEANUP_AUTOGRAD_CONTEXT_REQ: {
542:       return processCleanupAutogradContextReq(rpc);
543:     }
544:     case MessageType::RUN_WITH_PROFILING_REQ: {
545:       return processRunWithProfilingReq(rpc);
546:     }
547:     case MessageType::RREF_BACKWARD_REQ: {
548:       return processRRefBackward(rpc);
549:     }
550:     default: {
551:       TORCH_INTERNAL_ASSERT(
552:           false, "Request type ", messageType, " not supported.");
553:     }
554:   }
555: }
556: 
557: c10::intrusive_ptr<Message> RequestCallbackNoPython::handleError(
558:     const std::exception& e,
559:     const MessageType messageType,
560:     int64_t messageId) const {
```

- EN: Lines 541-560 introduces executable logic in routines such as `TORCH_INTERNAL_ASSERT`; returns computed state or forwards results to the surrounding caller.
- CN: 第 541-560 行在 `TORCH_INTERNAL_ASSERT` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 561-580 / 第 561-580 行

```cpp
561:   LOG(ERROR) << "Received error while processing request type " << messageType
562:              << ": " << e.what();
563:   // Adding node information to the error here since all processed RPC
564:   // requests should be going through this function.
565:   std::string errorMsg = c10::str(
566:       "Error on Node ",
567:       DistAutogradContainer::getInstance().getWorkerId(),
568:       ": ",
569:       e.what());
570:   return createExceptionResponse(errorMsg, messageId);
571: }
572: 
573: bool RequestCallbackNoPython::cudaAvailable() const {
574: #ifdef USE_CUDA
575:   return true;
576: #else
577:   return false;
578: #endif
579: }
580: 
```

- EN: Lines 561-580 uses conditional compilation to adapt to feature flags, platforms, or optional backends; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 561-580 行使用条件编译来适配特性开关、平台或可选后端；为本段代码补充可调用 API、操作符或辅助例程。

### Lines 581-600 / 第 581-600 行

```cpp
581: c10::intrusive_ptr<JitFuture> RequestCallbackNoPython::runJitOperator(
582:     const jit::Operator& op,
583:     std::vector<at::IValue>& stack,
584:     const std::vector<c10::Stream>& streams) const {
585:   c10::MultiStreamGuard guard(streams);
586:   try {
587:     op.getOperation()(stack);
588:   } catch (const std::exception&) {
589:     return asFuture(std::current_exception());
590:   }
591:   TORCH_INTERNAL_ASSERT(
592:       stack.size() == 1,
593:       "Return value of a builtin operator or a TorchScript function should be "
594:       "a single IValue, got a vector of size ",
595:       stack.size());
596:   TypePtr type = stack.front().type();
597:   return asFuture(std::move(stack.front()), std::move(type));
598: }
599: 
600: c10::intrusive_ptr<JitFuture> RequestCallbackNoPython::asFuture(
```

- EN: Lines 581-600 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 581-600 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 601-620 / 第 601-620 行

```cpp
601:     IValue value,
602:     TypePtr type) const {
603:   auto future = c10::make_intrusive<JitFuture>(
604:       std::move(type), RpcAgent::getCurrentRpcAgent()->getDevices());
605:   future->markCompleted(std::move(value));
606:   return future;
607: }
608: 
609: c10::intrusive_ptr<JitFuture> RequestCallbackNoPython::asFuture(
610:     c10::intrusive_ptr<Message> message) const {
611:   auto future = c10::make_intrusive<JitFuture>(
612:       at::getCustomClassType<c10::intrusive_ptr<Message>>(),
613:       RpcAgent::getCurrentRpcAgent()->getDevices());
614:   std::vector<c10::weak_intrusive_ptr<c10::StorageImpl>> storages =
615:       message->getStorages();
616:   future->markCompleted(std::move(message), std::move(storages));
617:   return future;
618: }
619: 
620: c10::intrusive_ptr<JitFuture> RequestCallbackNoPython::asFuture(
```

- EN: Lines 601-620 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 601-620 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 621-628 / 第 621-628 行

```cpp
621:     std::exception_ptr err) const {
622:   auto future = c10::make_intrusive<JitFuture>(
623:       at::NoneType::get(), RpcAgent::getCurrentRpcAgent()->getDevices());
624:   future->setError(std::move(err));
625:   return future;
626: }
627: 
628: } // namespace torch::distributed::rpc
```

- EN: Lines 621-628 opens or closes namespaces to place the code in the correct distributed component; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 621-628 行打开或关闭命名空间，以便将代码放入正确的分布式组件中；为本段代码补充可调用 API、操作符或辅助例程。

## Key Concepts / 关键概念

- EN: Subsystem: distributed RPC layer.
- CN: 子系统：分布式 RPC 层。
- EN: File role: implementation/registration.
- CN: 文件角色：实现/注册。
- EN: Primary symbols: `DistAutogradContextGuard`
- CN: 核心符号：`DistAutogradContextGuard`
- EN: Notable themes: RPC/RRef semantics.
- CN: 值得关注的主题：RPC/RRef 语义。

## Dependencies / 依赖关系

- Internal distributed headers / 内部分布式头文件: `torch/csrc/distributed/rpc/request_callback_no_python.h`, `torch/csrc/distributed/autograd/context/container.h`, `torch/csrc/distributed/autograd/engine/dist_engine.h`, `torch/csrc/distributed/autograd/rpc_messages/cleanup_autograd_context_req.h`, `torch/csrc/distributed/autograd/rpc_messages/cleanup_autograd_context_resp.h`, `torch/csrc/distributed/autograd/rpc_messages/propagate_gradients_req.h`, `torch/csrc/distributed/autograd/rpc_messages/propagate_gradients_resp.h`, `torch/csrc/distributed/autograd/rpc_messages/rpc_with_autograd.h`, `torch/csrc/distributed/autograd/utils.h`, `torch/csrc/distributed/rpc/profiler/server_process_global_profiler.h`, `torch/csrc/distributed/rpc/rpc_agent.h`, `torch/csrc/distributed/rpc/rref_context.h`, ... (+3 more)
- PyTorch/ATen/C10 headers / PyTorch/ATen/C10 头文件: `c10/core/StreamGuard.h`
- External or system headers / 外部或系统头文件: `utility`
- Local symbols / 本地符号: `DistAutogradContextGuard`