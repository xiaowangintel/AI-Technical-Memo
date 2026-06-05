# dist_engine.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `torch/csrc/distributed/autograd/engine/dist_engine.cpp`
- Repository: `pytorch` (`/root/xw/pytorch`)
- Purpose (EN): This file provides implementation logic for dist engine in the distributed autograd execution engine. Key types include `DistAccumulateGradCaptureHook`.
- 用途 (CN): 该文件在分布式自动求导执行引擎中提供dist engine 的实现逻辑。 关键类型包括 `DistAccumulateGradCaptureHook`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

```cpp
1: #include <queue>
2: 
3: #include <ATen/Parallel.h>
4: #include <c10/core/Event.h>
5: #include <c10/util/DeadlockDetection.h>
6: #include <c10/util/irange.h>
7: #include <c10/util/thread_name.h>
8: #include <torch/csrc/autograd/functions/accumulate_grad.h>
9: #include <torch/csrc/autograd/input_buffer.h>
10: #include <torch/csrc/distributed/autograd/context/container.h>
11: #include <torch/csrc/distributed/autograd/engine/dist_engine.h>
12: 
13: namespace torch::distributed::autograd {
14: 
15: using torch::autograd::AccumulateGrad;
16: using torch::autograd::edge_list;
17: using torch::autograd::Engine;
18: using torch::autograd::GraphRoot;
19: using torch::autograd::GraphTask;
20: using torch::autograd::GraphTaskGuard;
```

- EN: Lines 1-20 pulls in the headers required by this translation unit or interface; opens or closes namespaces to place the code in the correct distributed component.
- CN: 第 1-20 行引入该实现单元或接口所需的头文件；打开或关闭命名空间，以便将代码放入正确的分布式组件中。

### Lines 21-40 / 第 21-40 行

```cpp
21: using torch::autograd::InputBuffer;
22: using torch::autograd::Node;
23: using torch::autograd::NodeTask;
24: using torch::autograd::ReadyQueue;
25: using torch::autograd::validate_outputs;
26: using torch::autograd::variable_list;
27: 
28: static constexpr const char* kNumBackwardPasses = "num_current_backward_passes";
29: static constexpr const char* kNumAutogradContexts = "num_autograd_contexts";
30: 
31: // This hook does 3 things:
32: //   1. Call pre hooks of the original AccumulateGrad to modify the input grad.
33: //   2. Accumurate the guard to RPC context.
34: //   3. Call post hooks of the original AccumulateGrad.
35: class DistAccumulateGradCaptureHook
36:     : public GraphTask::ExecInfo::Capture::GradCaptureHook {
37:  public:
38:   DistAccumulateGradCaptureHook(
39:       c10::intrusive_ptr<AccumulateGrad> accumulateGrad,
40:       ContextPtr autogradContext)
```

- EN: Lines 21-40 declares or defines types such as `DistAccumulateGradCaptureHook`.
- CN: 第 21-40 行声明或定义了 `DistAccumulateGradCaptureHook` 等类型。

### Lines 41-60 / 第 41-60 行

```cpp
41:       : accumulateGrad_(std::move(accumulateGrad)),
42:         autogradContext_(std::move(autogradContext)) {}
43: 
44:   at::Tensor operator()(const at::Tensor& grad) override {
45:     ThreadLocalDistAutogradContext contextGuard{ContextPtr(autogradContext_)};
46:     variable_list inputGrads = {grad};
47:     // It's intended that pre/post hooks are still called even if the grad is
48:     // undefined here.
49:     for (const auto& hook : accumulateGrad_->pre_hooks()) {
50:       inputGrads = (*hook)(inputGrads);
51:     }
52:     // It is possible that the grad is not defined since a separate
53:     // invocation of the autograd engine on the same node might actually
54:     // compute this gradient.
55:     if (inputGrads[0].defined()) {
56:       // There are 3 internal references to 'inputGrads[0]' at this moment:
57:       //   1. 'inputGrads[0]' in this function.
58:       //   2. 'graph_task->captured_vars_' on the callsite in the local engine.
59:       //   3. 'InputBuffer& inputs' on the callsite as the inputs of the
60:       //   function node.
```

- EN: Lines 41-60 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 41-60 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 61-80 / 第 61-80 行

```cpp
61:       autogradContext_->accumulateGrad(
62:           accumulateGrad_->variable, inputGrads[0], 3 /* num_expected_refs */);
63:     }
64:     const variable_list kEmptyOutput;
65:     for (const auto& hook : accumulateGrad_->post_hooks()) {
66:       (*hook)(kEmptyOutput, inputGrads);
67:     }
68:     return std::move(inputGrads[0]);
69:   }
70: 
71:  private:
72:   c10::intrusive_ptr<AccumulateGrad> accumulateGrad_;
73:   ContextPtr autogradContext_;
74: };
75: 
76: void DistEngine::globalCpuThread(
77:     const std::shared_ptr<ReadyQueue>& ready_queue) {
78:   c10::setThreadName("pt_dist_engine");
79:   while (true) {
80:     NodeTask task = ready_queue->pop();
```

- EN: Lines 61-80 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 61-80 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 81-100 / 第 81-100 行

```cpp
81:     if (task.isShutdownTask_) {
82:       // Need to shutdown this thread.
83:       C10_LOG_API_USAGE_ONCE("torch.autograd.thread_shutdown");
84:       break;
85:     }
86: 
87:     auto graphTask = task.base_.lock();
88:     if (graphTask == nullptr) {
89:       // GraphTask has expired, ignore and continue processing.
90:       continue;
91:     }
92: 
93:     // Launch the execution on a JIT thread.
94:     at::launch([this,
95:                 graphTask,
96:                 graphRoot = task.fn_,
97:                 variables =
98:                     InputBuffer::variables(std::move(task.inputs_))]() mutable {
99:       InputBuffer inputs(variables.size());
100:       for (const auto i : c10::irange(variables.size())) {
```

- EN: Lines 81-100 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 81-100 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 101-120 / 第 101-120 行

```cpp
101:         inputs.add(
102:             i,
103:             std::move(variables[i]),
104:             std::nullopt,
105:             std::nullopt,
106:             graphRoot.get());
107:       }
108:       execute_graph_task_until_ready_queue_empty(
109:           /*node_task*/ NodeTask(graphTask, graphRoot, std::move(inputs)),
110:           /*incrementOutstandingTasks*/ false);
111:     });
112:   }
113: }
114: 
115: DistEngine::DistEngine()
116:     : engine_(Engine::get_default_engine()),
117:       global_cpu_ready_queue_(std::make_shared<ReadyQueue>()),
118:       global_cpu_thread_(
119:           &DistEngine::globalCpuThread,
120:           this,
```

- EN: Lines 101-120 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 101-120 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 121-140 / 第 121-140 行

```cpp
121:           global_cpu_ready_queue_) {
122:   // Note [GPU to CPU continuations]
123:   // ~~~~~~~~~~~~~~~~~~~~~~~~~~
124:   // Initialize a single CPU thread to execute continuations from GPU
125:   // tasks. The multithreaded structure for the distributed engine works
126:   // well only for CPU tasks. If we have an order of tasks like
127:   // CPU->GPU->CPU, distributed autograd has no thread to execute the last
128:   // CPU task on. To fix this, we introduce a global CPU thread to handle
129:   // such situations and it will be responsible for executing these CPU
130:   // tasks. The CPU thread has its own ready_queue which is used as the
131:   // cpu_ready_queue for all GraphTasks for DistEngine. This ensures all GPU
132:   // to CPU continuations are enqueued on this thread. The global CPU thread
133:   // simply dequeues tasks from the global queue and calls
134:   // "execute_graph_task_until_ready_queue_empty" on a JIT thread to execute the
135:   // appropriate task.
136:   global_cpu_thread_.detach();
137: }
138: 
139: DistEngine::~DistEngine() {
140:   // Ensure we shutdown the CPU thread.
```

- EN: Lines 121-140 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 121-140 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 141-160 / 第 141-160 行

```cpp
141:   TORCH_ASSERT_NO_GIL_WITHOUT_PYTHON_DEP();
142:   global_cpu_ready_queue_->pushShutdownTask();
143:   global_cpu_thread_.join();
144: }
145: 
146: DistEngine& DistEngine::getInstance() {
147:   // Leaky singleton to avoid module destructor race.
148:   static DistEngine* engine = new DistEngine();
149:   return *engine;
150: }
151: 
152: void DistEngine::validateRootsAndRetrieveEdges(
153:     const variable_list& roots,
154:     edge_list& rootEdges,
155:     variable_list& grads) {
156:   TORCH_CHECK(!roots.empty(), "No tensors provided for gradient computation.");
157:   TORCH_INTERNAL_ASSERT(rootEdges.empty());
158:   TORCH_INTERNAL_ASSERT(grads.empty());
159: 
160:   // Verify roots are all scalar and require gradients.
```

- EN: Lines 141-160 introduces executable logic in routines such as `TORCH_ASSERT_NO_GIL_WITHOUT_PYTHON_DEP`; performs validation and error handling to keep distributed state consistent.
- CN: 第 141-160 行在 `TORCH_ASSERT_NO_GIL_WITHOUT_PYTHON_DEP` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 161-180 / 第 161-180 行

```cpp
161:   for (const auto& root : roots) {
162:     TORCH_CHECK(root.requires_grad(), "requires_grad not set on root");
163:     TORCH_CHECK(
164:         root.numel() == 1,
165:         root.name(),
166:         " is not a scalar, all roots need to be scalar");
167:     TORCH_CHECK(
168:         root.grad_fn(),
169:         root.name(),
170:         " does not have a valid gradient function.");
171: 
172:     // Compute the root edges and generate the appropriate gradients.
173:     rootEdges.push_back(torch::autograd::impl::gradient_edge(root));
174:     grads.push_back(at::ones_like(root, LEGACY_CONTIGUOUS_MEMORY_FORMAT));
175:   }
176: 
177:   // Validate rootEdges and grads.
178:   validate_outputs(
179:       rootEdges, grads, [](const std::string& msg) { return msg; });
180: }
```

- EN: Lines 161-180 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 161-180 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 181-200 / 第 181-200 行

```cpp
181: 
182: void DistEngine::computeDependencies(
183:     const ContextPtr& autogradContext,
184:     const edge_list& rootEdges,
185:     const variable_list& grads,
186:     const c10::intrusive_ptr<Node>& graphRoot,
187:     edge_list& outputEdges,
188:     bool retainGraph) {
189:   TORCH_INTERNAL_ASSERT(graphRoot, "graphRoot is null!");
190: 
191:   // Store root nodes so we can traverse through the graph later
192:   // e.g., for get_current_graph_task_execution_order
193:   c10::SmallVector<Node*, 4> temp_roots{rootEdges.size()};
194:   for (const auto i : c10::irange(rootEdges.size())) {
195:     temp_roots[i] = rootEdges[i].function.get();
196:   }
197: 
198:   // Build the graph task and graph root.
199:   // NOTE: we don't need to build and pass a cpu_ready_queue to GraphTask
200:   // as we use execute_graph_task_until_ready_queue_empty, which will build
```

- EN: Lines 181-200 introduces executable logic in routines such as `TORCH_INTERNAL_ASSERT`.
- CN: 第 181-200 行在 `TORCH_INTERNAL_ASSERT` 等例程中引入具体执行逻辑。

### Lines 201-220 / 第 201-220 行

```cpp
201:   // a separate ReadyQueue for each call.
202:   auto graphTask = std::make_shared<GraphTask>(
203:       /* keep_graph */ retainGraph,
204:       /* create_graph */ false,
205:       /* depth */ 0,
206:       /* cpu_ready_queue */ global_cpu_ready_queue_,
207:       /* graph_roots */ temp_roots,
208:       /* exit_on_error */ true);
209: 
210:   // Run BFS to traverse the graph locally. The roots of the graph are
211:   // GraphRoot and all send functions for this autograd context.
212:   std::unordered_set<Node*> seen;
213:   std::queue<Node*> queue;
214:   queue.push(static_cast<Node*>(graphRoot.get()));
215: 
216:   auto sendFunctions = autogradContext->sendFunctions();
217: 
218:   // Add all the send functions to the queue as roots.
219:   for (const auto& mapEntry : sendFunctions) {
220:     // Increment 'outstanding_tasks_' for GraphTask for each send_function
```

- EN: Lines 201-220 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 201-220 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 221-240 / 第 221-240 行

```cpp
221:     // since we want the local autograd engine to wait for all of them.
222:     graphTask->outstanding_tasks_++;
223:     queue.push(mapEntry.second.get());
224:   }
225: 
226:   bool will_use_accelerator = false;
227: 
228:   edge_list recvBackwardEdges;
229:   // Traverse the graph.
230:   auto& dependencies = graphTask->dependencies_;
231:   while (!queue.empty()) {
232:     auto fn = queue.front();
233:     queue.pop();
234: 
235:     if (!will_use_accelerator) {
236:       will_use_accelerator = fn->stream().has_value();
237:     }
238: 
239:     for (const auto& edge : fn->next_edges()) {
240:       if (auto nextFn = edge.function.get()) {
```

- EN: Lines 221-240 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 221-240 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 241-260 / 第 241-260 行

```cpp
241:         dependencies[nextFn] += 1;
242:         const bool wasInserted = seen.insert(nextFn).second;
243:         if (wasInserted) {
244:           // Seeing this function for the first time.
245:           queue.push(nextFn);
246: 
247:           if (nextFn->next_edges().empty()) {
248:             TORCH_INTERNAL_ASSERT(
249:                 dynamic_cast<AccumulateGrad*>(nextFn) ||
250:                 dynamic_cast<RecvRpcBackward*>(nextFn));
251:             // We have found a leaf node which should be either AccumulateGrad
252:             // or RecvRpcBackward. Record the function
253:             // to ensure we don't execute it and instead accumulate the grads on
254:             // the autograd context. These functions would be passed in as the
255:             // 'outputs' parameter of the vanilla autograd engine.
256: 
257:             // We don't accumulate any grads in the context for RecvRpcBackward.
258:             // RecvRpcBackward is added as an output edge to indicate it is a
259:             // leaf node and this helps in properly computing dependencies for
260:             // the local autograd graph. Putting RecvRpcBackward in
```

- EN: Lines 241-260 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 241-260 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 261-280 / 第 261-280 行

```cpp
261:             // 'outputEdges' means that this function needs to be executed
262:             // (inline with our assumption for FAST mode that all send/recv
263:             // functions are valid in the backward pass), and as a result all of
264:             //  its ancestors need to be executed as well.
265:             if (dynamic_cast<RecvRpcBackward*>(nextFn)) {
266:               recvBackwardEdges.emplace_back(edge);
267:             }
268:             outputEdges.emplace_back(edge);
269:           }
270:         }
271:       }
272:     }
273:   }
274: 
275:   if (will_use_accelerator) {
276:     // Collects current streams for CUDA/ROCM devices where this process has a
277:     // context, so graphTask::exec_post_processing can sync them with
278:     // leaf_streams.
279:     graphTask->stash_current_streams();
280:   }
```

- EN: Lines 261-280 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 261-280 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 281-300 / 第 281-300 行

```cpp
281: 
282:   // Now lets compute which functions need to be executed. The algorithm is as
283:   // follows:
284:   // 1. Create a dummy GraphRoot which points to all 'send' functions for this
285:   //    context and the original graphRoot. Run 'init_to_execute' with the
286:   //    outputEdges and the dummy GraphRoot. This ensures we mark
287:   //    appropriate functions as needed if they are reachable only from a
288:   //    specific 'send' function locally and not necessarily from the provided
289:   //    roots.
290:   // 2. For all edges in 'outputEdges' which point to 'RecvRpcBackward', mark
291:   //    those functions as needed for execution. The reason for this is that
292:   //    'init_to_execute', will mark these as not needed. But 'RecvRpcBackward'
293:   //    is unique in the sense that we use it as a leaf node in graph to compute
294:   //    needed execution accurately, but unlike AccumulateGrad, we do need to
295:   //    execute this function.
296:   if (!outputEdges.empty()) {
297:     // Compute 'needed execution' starting from all 'send' functions and the
298:     // original graphRoot.
299:     edge_list edges;
300:     // Create some dummy edges (input_nr not important for init_to_execute).
```

- EN: Lines 281-300 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 281-300 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 301-320 / 第 301-320 行

```cpp
301:     for (const auto& mapEntry : sendFunctions) {
302:       edges.emplace_back(mapEntry.second, 0);
303:     }
304: 
305:     // Add the original graphRoot as an edge.
306:     edges.emplace_back(graphRoot, 0);
307: 
308:     // Create a dummy GraphRoot and run init_to_execute with it.
309:     GraphRoot dummyRoot(edges, {});
310:     graphTask->init_to_execute(
311:         dummyRoot, outputEdges, /*accumulate_grad=*/false, /*min_topo_nr=*/0);
312:     for (auto& mapEntry : graphTask->exec_info_) {
313:       auto& execInfo = mapEntry.second;
314:       if (!execInfo.captures_) {
315:         continue;
316:       }
317:       auto fn = mapEntry.first;
318:       // There may be nodes other than 'AccumulateGrad', e.g. RecvRPCBackward,
319:       // to be captured.
320:       if (auto accumulateGradFn = dynamic_cast<AccumulateGrad*>(fn)) {
```

- EN: Lines 301-320 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 301-320 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 321-340 / 第 321-340 行

```cpp
321:         for (auto& capture : *execInfo.captures_) {
322:           // Capture hooks are technically deprecated, but as an exception below
323:           // is the single and only instance of capture hooks usage that we
324:           // support. See NOTE [Deprecated capture hooks] for more context.
325:           capture.DO_NOT_USE_DEPRECATED_register_capture_hook(
326:               std::make_unique<DistAccumulateGradCaptureHook>(
327:                   c10::static_intrusive_pointer_cast<AccumulateGrad>(
328:                       accumulateGradFn->getptr()),
329:                   autogradContext));
330:         }
331:       }
332:     }
333: 
334:     // Mark all 'RecvRPCBackward' as needing execution.
335:     for (const auto& recvBackwardEdge : recvBackwardEdges) {
336:       graphTask->exec_info_[recvBackwardEdge.function.get()].needed_ = true;
337:     }
338:   }
339: 
340:   // Set graph task owner in a single thread since concurrent access to
```

- EN: Lines 321-340 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 321-340 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 341-360 / 第 341-360 行

```cpp
341:   // 'owner_' field is not permitted.
342:   graphTask->owner_ = torch::autograd::CPU_DEVICE;
343: 
344:   // Let autograd context take ownership of the GraphTask.
345:   autogradContext->setGraphTask(std::move(graphTask));
346: }
347: 
348: void DistEngine::execute_graph_task_until_ready_queue_empty(
349:     NodeTask&& node_task,
350:     bool incrementOutstandingTasks) {
351:   engine_.initialize_device_threads_pool();
352:   // Create a ready queue per call to traverse the graph_task from
353:   // root_to_execute This allow concurrent execution of the same GraphTask from
354:   // different threads
355:   std::shared_ptr<ReadyQueue> cpu_ready_queue = std::make_shared<ReadyQueue>();
356:   auto graph_task = node_task.base_.lock();
357:   if (graph_task == nullptr) {
358:     LOG(ERROR) << "GraphTask has expired for NodeTask: "
359:                << node_task.fn_->name() << ", skipping execution.";
360:     return;
```

- EN: Lines 341-360 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 341-360 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 361-380 / 第 361-380 行

```cpp
361:   }
362: 
363:   cpu_ready_queue->push(std::move(node_task), incrementOutstandingTasks);
364: 
365:   torch::autograd::set_device(torch::autograd::CPU_DEVICE);
366:   while (!cpu_ready_queue->empty()) {
367:     std::shared_ptr<GraphTask> local_graph_task;
368:     {
369:       // Scope this block of execution since NodeTask is not needed after this
370:       // block and can be deallocated (release any references to grad tensors
371:       // as part of inputs_)
372:       NodeTask task = cpu_ready_queue->pop();
373:       local_graph_task = task.base_.lock();
374:       if (!local_graph_task) {
375:         continue;
376:       }
377:       if (task.fn_ && !local_graph_task->has_error_.load()) {
378:         at::ThreadLocalStateGuard tls_guard(local_graph_task->thread_locals_);
379:         try {
380:           GraphTaskGuard guard(local_graph_task);
```

- EN: Lines 361-380 introduces executable logic in routines such as `tls_guard`.
- CN: 第 361-380 行在 `tls_guard` 等例程中引入具体执行逻辑。

### Lines 381-400 / 第 381-400 行

```cpp
381:           engine_.evaluate_function(
382:               local_graph_task, task.fn_.get(), task.inputs_, cpu_ready_queue);
383:         } catch (std::exception& e) {
384:           engine_.thread_on_exception(local_graph_task, task.fn_, e);
385:           // break the loop in error so that we immediately stop the execution
386:           // of this GraphTask, mark it completed if necessary and return the
387:           // future with proper ErrorMessage
388:           break;
389:         }
390:       }
391:     }
392:     // Decrement the outstanding task.
393:     --local_graph_task->outstanding_tasks_;
394:   }
395:   // Check if we've completed execution.
396:   if (graph_task->completed()) {
397:     // We don't need to explicitly notify the owner thread, since
398:     // 'mark_as_completed_and_run_post_processing' would mark the Future as
399:     // completed and this would notify the owner thread that the task has been
400:     // completed.
```

- EN: Lines 381-400 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 381-400 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 401-420 / 第 401-420 行

```cpp
401:     graph_task->mark_as_completed_and_run_post_processing();
402:   }
403: }
404: 
405: c10::intrusive_ptr<c10::ivalue::Future> DistEngine::
406:     runEngineAndAccumulateGradients(
407:         const ContextPtr& autogradContext,
408:         const c10::intrusive_ptr<Node>& graphRoot,
409:         const edge_list& outputEdges,
410:         bool incrementOutstandingTasks) {
411:   // Cleanup previous state for outstanding RPCs. Outstanding RPCs could be
412:   // lingering if we're running backward multiple times and some of the
413:   // passes ran into errors.
414:   autogradContext->clearOutstandingRpcs();
415:   auto graphTask = autogradContext->retrieveGraphTask();
416:   at::launch([this, graphTask, graphRoot, incrementOutstandingTasks]() {
417:     execute_graph_task_until_ready_queue_empty(
418:         /*node_task*/ NodeTask(graphTask, graphRoot, InputBuffer(0)),
419:         /*incrementOutstandingTasks*/ incrementOutstandingTasks);
420:   });
```

- EN: Lines 401-420 introduces executable logic in routines such as `runEngineAndAccumulateGradients`.
- CN: 第 401-420 行在 `runEngineAndAccumulateGradients` 等例程中引入具体执行逻辑。

### Lines 421-440 / 第 421-440 行

```cpp
421:   // Use a reference here to avoid refcount bump on futureGrads.
422:   auto& futureGrads = graphTask->future_result_;
423: 
424:   // Build a future that waits for the callbacks to execute (since callbacks
425:   // execute after the original future is completed). This ensures we return a
426:   // future that waits for all gradient accumulation to finish.
427:   auto accumulateGradFuture =
428:       c10::make_intrusive<c10::ivalue::Future>(c10::NoneType::get());
429: 
430:   futureGrads->addCallback([autogradContext, outputEdges, accumulateGradFuture](
431:                                c10::ivalue::Future& futureGrads) {
432:     if (futureGrads.hasError()) {
433:       // Don't accumulate gradients if we receive an error.
434:       // We must add the node information here since DistEngine::execute
435:       // waits on accumulateGradFuture and will throw an exception once we
436:       // set the error below.
437:       std::string errorMsg = c10::str(
438:           "Error on Node ",
439:           DistAutogradContainer::getInstance().getWorkerId(),
440:           ": ",
```

- EN: Lines 421-440 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 421-440 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 441-460 / 第 441-460 行

```cpp
441:           futureGrads.tryRetrieveErrorMessage());
442:       accumulateGradFuture->setError(std::make_exception_ptr(
443:           c10::ivalue::Future::FutureError(std::move(errorMsg))));
444:       return;
445:     }
446: 
447:     try {
448:       const variable_list& grads = futureGrads.constValue().toTensorVector();
449:       TORCH_INTERNAL_ASSERT(grads.size() == outputEdges.size());
450:       accumulateGradFuture->markCompleted(c10::IValue());
451:     } catch (std::exception&) {
452:       accumulateGradFuture->setErrorIfNeeded(std::current_exception());
453:     }
454:   });
455: 
456:   return accumulateGradFuture;
457: }
458: 
459: c10::intrusive_ptr<c10::ivalue::Future> DistEngine::executeSendFunctionAsync(
460:     const ContextPtr& autogradContext,
```

- EN: Lines 441-460 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 441-460 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 461-480 / 第 461-480 行

```cpp
461:     const c10::intrusive_ptr<SendRpcBackward>& sendFunction,
462:     bool retainGraph) {
463:   // Typically the local autograd engine ensures stream synchronizations between
464:   // nodes in the graph. However, for distributed autograd the sendFunction
465:   // inputs might have been retrieved over the wire on a separate stream and the
466:   // sendFunction itself runs on a different stream. As a result, we need to
467:   // manually synchronize those two streams here.
468:   const auto& send_backward_stream = sendFunction->stream();
469:   if (send_backward_stream) {
470:     for (const auto& grad : sendFunction->getGrads()) {
471:       const auto guard = c10::impl::VirtualGuardImpl{c10::DeviceType::CUDA};
472:       const auto default_stream = guard.getStream(grad.device());
473:       if (send_backward_stream != default_stream) {
474:         auto event = c10::Event{c10::DeviceType::CUDA};
475:         event.record(default_stream);
476:         send_backward_stream->wait(event);
477:       }
478:     }
479:   }
480: 
```

- EN: Lines 461-480 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 461-480 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 481-500 / 第 481-500 行

```cpp
481:   std::unique_lock<std::mutex> lock(initializedContextIdsLock_);
482:   if (initializedContextIds_.find(autogradContext->contextId()) ==
483:       initializedContextIds_.end()) {
484:     edge_list outputEdges;
485:     // Pass in a dummy graphRoot since all send functions are the roots.
486:     auto dummyRoot =
487:         c10::make_intrusive<GraphRoot>(edge_list(), variable_list());
488:     computeDependencies(
489:         autogradContext, {}, {}, dummyRoot, outputEdges, retainGraph);
490: 
491:     // Mark the autograd context id as initialized and unlock.
492:     initializedContextIds_.insert(autogradContext->contextId());
493:     lock.unlock();
494: 
495:     // Enqueue the current send function.
496:     auto graphTask = autogradContext->retrieveGraphTask();
497:     // Run the autograd engine.
498:     auto accumulateGradFuture = runEngineAndAccumulateGradients(
499:         autogradContext,
500:         sendFunction,
```

- EN: Lines 481-500 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 481-500 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 501-520 / 第 501-520 行

```cpp
501:         outputEdges,
502:         /*incrementOutstandingTasks=*/false);
503: 
504:     // Build the 'uber' future that waits for everything.
505:     auto callbackFuture =
506:         c10::make_intrusive<c10::ivalue::Future>(c10::NoneType::get());
507: 
508:     accumulateGradFuture->addCallback(
509:         [autogradContext,
510:          callbackFuture](c10::ivalue::Future& accumulateGradFuture) {
511:           try {
512:             if (accumulateGradFuture.hasError()) {
513:               // Perform cleanup at the end of the backward pass (before we mark
514:               // the future as completed).
515:               DistEngine::getInstance().cleanupBackwardPass(autogradContext);
516: 
517:               // Skip any further processing on errors.
518:               callbackFuture->setError(accumulateGradFuture.exception_ptr());
519:               return;
520:             }
```

- EN: Lines 501-520 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 501-520 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 521-540 / 第 521-540 行

```cpp
521: 
522:             // Wait for all RPCs after the autograd engine is done.
523:             auto rpcFuture =
524:                 autogradContext->clearAndWaitForOutstandingRpcsAsync();
525:             rpcFuture->addCallback([callbackFuture, autogradContext](
526:                                        c10::ivalue::Future& rpcFuture) {
527:               try {
528:                 // Perform cleanup at the end of the backward pass (before
529:                 // we mark the future as completed).
530:                 DistEngine::getInstance().cleanupBackwardPass(autogradContext);
531:               } catch (std::exception&) {
532:                 callbackFuture->setErrorIfNeeded(std::current_exception());
533:                 return;
534:               }
535: 
536:               // Finally mark the 'uber' future as completed.
537:               if (!rpcFuture.hasError()) {
538:                 callbackFuture->markCompleted(c10::IValue());
539:               } else {
540:                 callbackFuture->setError(rpcFuture.exception_ptr());
```

- EN: Lines 521-540 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 521-540 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 541-560 / 第 541-560 行

```cpp
541:               }
542:             });
543:           } catch (std::exception&) {
544:             callbackFuture->setErrorIfNeeded(std::current_exception());
545:           }
546:         });
547: 
548:     // Return the future which waits for all async processing to be done.
549:     return callbackFuture;
550:   } else {
551:     lock.unlock();
552:     auto graphTask = autogradContext->retrieveGraphTask();
553:     at::launch([this, graphTask, sendFunction]() {
554:       execute_graph_task_until_ready_queue_empty(
555:           /*node_task*/ NodeTask(graphTask, sendFunction, InputBuffer(0)),
556:           /*incrementOutstandingTasks*/ false);
557:     });
558:     auto fut = c10::make_intrusive<c10::ivalue::Future>(c10::NoneType::get());
559:     fut->markCompleted(c10::IValue());
560:     return fut;
```

- EN: Lines 541-560 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 541-560 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 561-580 / 第 561-580 行

```cpp
561:   }
562: }
563: 
564: void DistEngine::execute(
565:     int64_t contextId,
566:     const variable_list& roots,
567:     bool retainGraph) {
568:   // Retrieve the context for the given context_id. This will throw if the
569:   // context_id is invalid.
570:   auto autogradContext =
571:       DistAutogradContainer::getInstance().retrieveContext(contextId);
572: 
573:   // Perform initial pre-processing.
574:   edge_list rootEdges;
575:   variable_list grads;
576:   validateRootsAndRetrieveEdges(roots, rootEdges, grads);
577: 
578:   c10::intrusive_ptr<Node> graphRoot =
579:       c10::make_intrusive<GraphRoot>(rootEdges, grads);
580:   edge_list outputEdges;
```

- EN: Lines 561-580 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 561-580 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 581-600 / 第 581-600 行

```cpp
581:   // Compute dependencies locally, starting from all roots and all 'send'
582:   // functions.
583:   {
584:     std::lock_guard<std::mutex> guard(initializedContextIdsLock_);
585:     // Context should not have been initialized already.
586:     TORCH_INTERNAL_ASSERT(
587:         initializedContextIds_.find(autogradContext->contextId()) ==
588:         initializedContextIds_.end());
589: 
590:     computeDependencies(
591:         autogradContext, rootEdges, grads, graphRoot, outputEdges, retainGraph);
592: 
593:     // Mark the autograd context id as initialized.
594:     initializedContextIds_.insert(autogradContext->contextId());
595:   }
596: 
597:   BackwardPassCleanupGuard guard(autogradContext);
598: 
599:   // This needs to be blocking and as a result we wait for the future to
600:   // complete.
```

- EN: Lines 581-600 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 581-600 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 601-620 / 第 601-620 行

```cpp
601:   runEngineAndAccumulateGradients(autogradContext, graphRoot, outputEdges)
602:       ->waitAndThrow();
603: 
604:   // Wait for all of the outstanding rpcs to complete.
605:   autogradContext->clearAndWaitForOutstandingRpcsAsync()->waitAndThrow();
606: }
607: 
608: void DistEngine::cleanupBackwardPass(const ContextPtr& autogradContext) {
609:   // Validate only the GraphTask is holding a reference to the Future
610:   // which holds gradients for the backward pass. This ensures that
611:   // after 'resetGraphTask' is called below, there are no remaining
612:   // references left to the gradients for the backward pass.
613:   //
614:   // This ensures our 'use_count' checks in
615:   // AccumulateGrad::accumulateGrad are correct and we're
616:   // not leaking any references to the gradients anywhere else.
617:   const auto& futureGrads =
618:       autogradContext->retrieveGraphTask()->future_result_;
619:   TORCH_INTERNAL_ASSERT(futureGrads.use_count() == 1);
620: 
```

- EN: Lines 601-620 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 601-620 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 621-640 / 第 621-640 行

```cpp
621:   // Reset the graph task once we're done with all processing.
622:   autogradContext->resetGraphTask();
623: 
624:   // Clear any outstanding rpcs.
625:   autogradContext->clearOutstandingRpcs();
626: 
627:   // Clear the context id once we're done with the autograd engine
628:   // processing.
629:   std::lock_guard<std::mutex> guard(initializedContextIdsLock_);
630:   initializedContextIds_.erase(autogradContext->contextId());
631: }
632: 
633: size_t DistEngine::numBackwardPasses() const {
634:   std::lock_guard<std::mutex> guard(initializedContextIdsLock_);
635:   return initializedContextIds_.size();
636: }
637: 
638: std::unordered_map<std::string, int64_t> DistEngine::getDebugInfo() const {
639:   std::unordered_map<std::string, int64_t> debugInfo;
640:   debugInfo[kNumBackwardPasses] = static_cast<int64_t>(numBackwardPasses());
```

- EN: Lines 621-640 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 621-640 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 641-646 / 第 641-646 行

```cpp
641:   debugInfo[kNumAutogradContexts] = static_cast<int64_t>(
642:       DistAutogradContainer::getInstance().numAutogradContexts());
643:   return debugInfo;
644: }
645: 
646: } // namespace torch::distributed::autograd
```

- EN: Lines 641-646 opens or closes namespaces to place the code in the correct distributed component; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 641-646 行打开或关闭命名空间，以便将代码放入正确的分布式组件中；为本段代码补充可调用 API、操作符或辅助例程。

## Key Concepts / 关键概念

- EN: Subsystem: distributed autograd execution engine.
- CN: 子系统：分布式自动求导执行引擎。
- EN: File role: implementation/registration.
- CN: 文件角色：实现/注册。
- EN: Primary symbols: `DistAccumulateGradCaptureHook`
- CN: 核心符号：`DistAccumulateGradCaptureHook`
- EN: Notable themes: store/state coordination, distributed autograd.
- CN: 值得关注的主题：存储/状态协调、分布式自动求导。

## Dependencies / 依赖关系

- Internal distributed headers / 内部分布式头文件: `torch/csrc/distributed/autograd/context/container.h`, `torch/csrc/distributed/autograd/engine/dist_engine.h`
- PyTorch/ATen/C10 headers / PyTorch/ATen/C10 头文件: `ATen/Parallel.h`, `c10/core/Event.h`, `c10/util/DeadlockDetection.h`, `c10/util/irange.h`, `c10/util/thread_name.h`, `torch/csrc/autograd/functions/accumulate_grad.h`, `torch/csrc/autograd/input_buffer.h`
- External or system headers / 外部或系统头文件: `queue`
- Local symbols / 本地符号: `DistAccumulateGradCaptureHook`