# kernels.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/dynamo/genai_layers/kernels.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60
````python
 1: from typing import Any
 2: 
 3: import cutlass
 4: import cutlass.torch as cutlass_torch
 5: from utils import BenchmarkKernel
 6: 
 7: import torch
 8: import torch.nn.functional as F
 9: 
10: 
11: # more important shapes used by internal models
12: extra_shapes_for_norm = (
13:     (1152 * 500, 384),
14:     (1152 * 500, 512),
15:     (1152 * 1000, 384),
16:     (1152 * 1000, 512),
17: )
18: 
19: 
20: class CrossEntropyForward(BenchmarkKernel):
21:     def __init__(self, script_args):
22:         super().__init__(script_args)
23:         self.available_backends = ["eager", "compiled", "quack", "liger"]
24: 
25:     def get_shapes(self) -> tuple[tuple[int, ...], ...]:
26:         return (
27:             (32768, 256),
28:             (32768, 512),
29:             (32768, 1024),
30:             (32768, 2048),
31:             (32768, 4096),
32:             (32768, 8192),
33:             (32768, 16384),
34:             (32768, 32768),
35:             (32768, 65536),
36:             (16384, 131072),
37:             (8192, 262144),
38:         )
39: 
40:     def get_memory_bytes(self, args, kwargs) -> int:
41:         # Read x (M*N elements) + read target (M elements) + write loss (M elements)
42:         x, target = args
43:         M, N = x.shape
44:         dtype = x.dtype
45:         return (M * N + M + M) * dtype.itemsize
46: 
47:     def eager(self, args, kwargs=None) -> Any:
48:         if kwargs is not None:
49:             raise AssertionError(f"Expected kwargs to be None, but got {kwargs}")
50:         x, target = args
51:         return lambda: F.cross_entropy(x, target, reduction="none")
52: 
53:     def compiled(self, args, kwargs=None) -> Any:
54:         if kwargs is not None:
55:             raise AssertionError(f"Expected kwargs to be None, but got {kwargs}")
56:         x, target = args
57: 
58:         # Mark batch size as dynamic for realistic workload
59:         torch._dynamo.mark_dynamic(x, 0)
60:         torch._dynamo.mark_dynamic(target, 0)
````
- EN: Handles module imports such as `typing`, `cutlass`, `cutlass.torch`, `utils`.
- CN: 处理模块导入，例如 `typing`, `cutlass`, `cutlass.torch`, `utils`。
- EN: Declares or extends types including `CrossEntropyForward`.
- CN: 声明或扩展类型，包括 `CrossEntropyForward`。
- EN: Implements callable logic such as `__init__`, `get_shapes`, `get_memory_bytes`, `eager`.
- CN: 实现可调用逻辑，例如 `__init__`, `get_shapes`, `get_memory_bytes`, `eager`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。

### Lines 61-120
````python
 61: 
 62:         # Need `lambda` otherwise torch.compile will not trace the function.
 63:         # More discussion: https://github.com/pytorch/pytorch/issues/158455
 64:         compiled_cross_entropy = torch.compile(
 65:             lambda x, target: F.cross_entropy(x, target, reduction="none"),
 66:             mode=self.compile_mode,
 67:             fullgraph=True,
 68:         )
 69:         return lambda: compiled_cross_entropy(x, target)
 70: 
 71:     def quack(self, args, kwargs=None) -> Any:
 72:         if kwargs is not None:
 73:             raise AssertionError(f"Expected kwargs to be None, but got {kwargs}")
 74:         x, target = args
 75:         from quack.cross_entropy import _cross_entropy
 76: 
 77:         return lambda: _cross_entropy(x, target)
 78: 
 79:     def liger(self, args, kwargs=None) -> Any:
 80:         if kwargs is not None:
 81:             raise AssertionError(f"Expected kwargs to be None, but got {kwargs}")
 82:         from liger_kernel.transformers.cross_entropy import LigerCrossEntropyLoss
 83: 
 84:         x, target = args
 85:         cross_entropy = LigerCrossEntropyLoss(reduction="none")
 86:         return lambda: cross_entropy(x, target)
 87: 
 88:     def benchmark(self):
 89:         for M, N in self.get_shapes():
 90:             print(f"\n Tensor dimensions: [{M}, {N}]")
 91:             # quack requires cutlass dtype
 92:             torch_dtype = cutlass_torch.dtype(cutlass.BFloat16)
 93:             x = 0.1 * torch.randn(M, N, device="cuda", dtype=torch_dtype)
 94:             target = torch.randint(0, N, (M,), device="cuda", dtype=torch.int64)
 95:             self.benchmark_single_shape((x, target), setting=f"shape: [{M}, {N}]")
 96: 
 97:     def check_accuracy(self, args, kwargs) -> None:
 98:         res = {}
 99:         for backend in self.available_backends:
100:             args_ref, kwargs_ref = self.clone_inputs(args, kwargs)
101:             res[backend] = getattr(self, backend)(args_ref, kwargs_ref)()
102:         gold = res["eager"]
103:         for backend in self.available_backends:
104:             if backend == "eager":
105:                 continue
106:             if backend == "quack":
107:                 # quack's cross_entropy only returns float32 loss output.
108:                 # Need to convert it to the same dtype as gold for comparison.
109:                 res[backend] = res[backend].to(gold.dtype)
110:             try:
111:                 torch.testing.assert_close(res[backend], gold)
112:                 print(
113:                     f"Accuracy check \033[92m✓ succeed\033[0m for {backend} backend on {self.name} kernel"
114:                 )
115:             except Exception as e:
116:                 print(
117:                     f"Accuracy check \033[91m✗ failed\033[0m for {backend} backend on {self.name} kernel. Error {e}"
118:                 )
119: 
120: 
````
- EN: Handles module imports such as `quack.cross_entropy`, `liger_kernel.transformers.cross_entropy`.
- CN: 处理模块导入，例如 `quack.cross_entropy`, `liger_kernel.transformers.cross_entropy`。
- EN: Implements callable logic such as `quack`, `liger`, `benchmark`, `check_accuracy`.
- CN: 实现可调用逻辑，例如 `quack`, `liger`, `benchmark`, `check_accuracy`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 121-180
````python
121: class CrossEntropyBackward(BenchmarkKernel):
122:     def __init__(self, script_args):
123:         super().__init__(script_args)
124:         self.available_backends = ["eager", "compiled", "quack", "liger"]
125: 
126:     def get_shapes(self) -> tuple[tuple[int, ...], ...]:
127:         return (
128:             (32768, 256),
129:             (32768, 512),
130:             (32768, 1024),
131:             (32768, 2048),
132:             (32768, 4096),
133:             (32768, 8192),
134:             (32768, 16384),
135:             (32768, 32768),
136:             (32768, 65536),
137:             (16384, 131072),
138:             (8192, 262144),
139:         )
140: 
141:     def get_memory_bytes(self, args, kwargs) -> int:
142:         # Read x (M*N elements) + read target (M elements) + read dloss (M elements) + write grad(M*N elements)
143:         x, target, dloss = args
144:         # Memory ba
145:         M, N = x.shape
146:         return (
147:             2 * M * N * x.dtype.itemsize
148:             + M * target.dtype.itemsize
149:             + M * dloss.dtype.itemsize
150:         )
151: 
152:     def eager(self, args, kwargs=None) -> Any:
153:         if kwargs is not None:
154:             raise AssertionError(f"Expected kwargs to be None, but got {kwargs}")
155:         x, target, dloss = args
156:         loss = F.cross_entropy(x, target, reduction="none")
157:         return lambda: torch.autograd.grad(
158:             loss, x, grad_outputs=dloss, retain_graph=True
159:         )
160: 
161:     def compiled(self, args, kwargs=None) -> Any:
162:         if kwargs is not None:
163:             raise AssertionError(f"Expected kwargs to be None, but got {kwargs}")
164:         x, target, dloss = args
165: 
166:         compiled_cross_entropy = torch.compile(
167:             lambda x, target: F.cross_entropy(x, target, reduction="none"),
168:             mode=self.compile_mode,
169:             fullgraph=True,
170:         )
171:         loss = compiled_cross_entropy(x, target)
172:         return lambda: torch.autograd.grad(
173:             loss, x, grad_outputs=dloss, retain_graph=True
174:         )
175: 
176:     def quack(self, args, kwargs=None) -> Any:
177:         from quack.cross_entropy import cross_entropy
178: 
179:         if kwargs is not None:
180:             raise AssertionError(f"Expected kwargs to be None, but got {kwargs}")
````
- EN: Handles module imports such as `quack.cross_entropy`.
- CN: 处理模块导入，例如 `quack.cross_entropy`。
- EN: Declares or extends types including `CrossEntropyBackward`.
- CN: 声明或扩展类型，包括 `CrossEntropyBackward`。
- EN: Implements callable logic such as `__init__`, `get_shapes`, `get_memory_bytes`, `eager`.
- CN: 实现可调用逻辑，例如 `__init__`, `get_shapes`, `get_memory_bytes`, `eager`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。

### Lines 181-240
````python
181:         x, target, dloss = args
182:         loss = cross_entropy(x, target)
183:         return lambda: torch.autograd.grad(
184:             loss, x, grad_outputs=dloss, retain_graph=True
185:         )
186: 
187:     def liger(self, args, kwargs=None) -> Any:
188:         if kwargs is not None:
189:             raise AssertionError(f"Expected kwargs to be None, but got {kwargs}")
190:         from liger_kernel.transformers.cross_entropy import LigerCrossEntropyLoss
191: 
192:         x, target, dloss = args
193:         cross_entropy = LigerCrossEntropyLoss(reduction="none")
194:         loss = cross_entropy(x, target)
195:         return lambda: torch.autograd.grad(
196:             loss, x, grad_outputs=dloss, retain_graph=True
197:         )
198: 
199:     def benchmark(self):
200:         for M, N in self.get_shapes():
201:             print(f"Tensor dimensions: [{M}, {N}]")
202:             torch_dtype = cutlass_torch.dtype(cutlass.BFloat16)
203:             x = 0.1 * torch.randn(
204:                 M, N, device="cuda", dtype=torch_dtype, requires_grad=True
205:             )
206:             target = torch.randint(0, N, (M,), device="cuda", dtype=torch.int64)
207:             dloss = torch.randn(M, device="cuda", dtype=torch.float32)
208:             self.benchmark_single_shape(
209:                 (x, target, dloss), setting=f"shape: [{M}, {N}]"
210:             )
211: 
212: 
213: class SoftmaxForward(BenchmarkKernel):
214:     def __init__(self, script_args):
215:         super().__init__(script_args)
216:         self.available_backends = ["eager", "compiled", "quack", "liger"]
217: 
218:     def get_shapes(self) -> tuple[tuple[int, ...], ...]:
219:         return (
220:             (32768, 256),
221:             (32768, 512),
222:             (32768, 1024),
223:             (32768, 2048),
224:             (32768, 4096),
225:             (32768, 8192),
226:             (32768, 16384),
227:             (32768, 32768),
228:             (32768, 65536),
229:             (16384, 131072),
230:             (8192, 262144),
231:         )
232: 
233:     def get_memory_bytes(self, args, kwargs) -> int:
234:         (x,) = args
235:         M, N = x.shape
236:         return 2 * M * N * x.dtype.itemsize
237: 
238:     def eager(self, args, kwargs=None) -> Any:
239:         if kwargs is not None:
240:             raise AssertionError(f"Expected kwargs to be None, but got {kwargs}")
````
- EN: Handles module imports such as `liger_kernel.transformers.cross_entropy`.
- CN: 处理模块导入，例如 `liger_kernel.transformers.cross_entropy`。
- EN: Declares or extends types including `SoftmaxForward`.
- CN: 声明或扩展类型，包括 `SoftmaxForward`。
- EN: Implements callable logic such as `liger`, `benchmark`, `__init__`, `get_shapes`.
- CN: 实现可调用逻辑，例如 `liger`, `benchmark`, `__init__`, `get_shapes`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。

### Lines 241-300
````python
241:         (x,) = args
242:         return lambda: F.softmax(x, dim=-1)
243: 
244:     def compiled(self, args, kwargs=None) -> Any:
245:         if kwargs is not None:
246:             raise AssertionError(f"Expected kwargs to be None, but got {kwargs}")
247:         (x,) = args
248: 
249:         # Mark batch size as dynamic for realistic workload
250:         torch._dynamo.mark_dynamic(x, 0)
251: 
252:         compiled_softmax = torch.compile(
253:             lambda x: F.softmax(x, dim=-1), mode=self.compile_mode, fullgraph=True
254:         )
255:         return lambda: compiled_softmax(x)
256: 
257:     def quack(self, args, kwargs=None) -> Any:
258:         from quack.softmax import softmax
259: 
260:         if kwargs is not None:
261:             raise AssertionError(f"Expected kwargs to be None, but got {kwargs}")
262:         (x,) = args
263:         return lambda: softmax(x)
264: 
265:     def liger(self, args, kwargs=None) -> Any:
266:         from liger_kernel.transformers.softmax import LigerSoftmax
267: 
268:         if kwargs is not None:
269:             raise AssertionError(f"Expected kwargs to be None, but got {kwargs}")
270:         (x,) = args
271:         softmax = LigerSoftmax().to("cuda")
272:         return lambda: softmax(x)
273: 
274:     def benchmark(self):
275:         for M, N in self.get_shapes():
276:             print(f"Tensor dimensions: [{M}, {N}]")
277:             torch_dtype = cutlass_torch.dtype(cutlass.BFloat16)
278:             x = 0.1 * torch.randn(M, N, device="cuda", dtype=torch_dtype)
279:             self.benchmark_single_shape((x,), setting=f"shape: [{M}, {N}]")
280: 
281: 
282: class SoftmaxBackward(BenchmarkKernel):
283:     def __init__(self, script_args):
284:         super().__init__(script_args)
285:         self.available_backends = ["eager", "compiled", "quack", "liger"]
286: 
287:     def get_shapes(self) -> tuple[tuple[int, ...], ...]:
288:         return (
289:             (32768, 256),
290:             (32768, 512),
291:             (32768, 1024),
292:             (32768, 2048),
293:             (32768, 4096),
294:             (32768, 8192),
295:             (32768, 16384),
296:             (32768, 32768),
297:             (32768, 65536),
298:             (16384, 131072),
299:             (8192, 262144),
300:         )
````
- EN: Handles module imports such as `quack.softmax`, `liger_kernel.transformers.softmax`.
- CN: 处理模块导入，例如 `quack.softmax`, `liger_kernel.transformers.softmax`。
- EN: Declares or extends types including `SoftmaxBackward`.
- CN: 声明或扩展类型，包括 `SoftmaxBackward`。
- EN: Implements callable logic such as `compiled`, `quack`, `liger`, `benchmark`.
- CN: 实现可调用逻辑，例如 `compiled`, `quack`, `liger`, `benchmark`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。

### Lines 301-360
````python
301: 
302:     def get_memory_bytes(self, args, kwargs) -> int:
303:         # Memory: read dy and y, write ax backward
304:         x, dy = args
305:         M, N = x.shape
306:         return 3 * M * N * x.dtype.itemsize
307: 
308:     def eager(self, args, kwargs=None) -> Any:
309:         if kwargs is not None:
310:             raise AssertionError(f"Expected kwargs to be None, but got {kwargs}")
311:         x, dy = args
312:         y = F.softmax(x, dim=-1)
313:         return lambda: torch.autograd.grad(y, x, grad_outputs=dy, retain_graph=True)
314: 
315:     def compiled(self, args, kwargs=None) -> Any:
316:         if kwargs is not None:
317:             raise AssertionError(f"Expected kwargs to be None, but got {kwargs}")
318:         x, dy = args
319:         compiled_softmax = torch.compile(
320:             lambda x: F.softmax(x, dim=-1), mode=self.compile_mode, fullgraph=True
321:         )
322:         y = compiled_softmax(x)
323:         return lambda: torch.autograd.grad(y, x, grad_outputs=dy, retain_graph=True)
324: 
325:     def quack(self, args, kwargs=None) -> Any:
326:         from quack.softmax import softmax
327: 
328:         if kwargs is not None:
329:             raise AssertionError(f"Expected kwargs to be None, but got {kwargs}")
330:         x, dy = args
331: 
332:         y = softmax(x)
333:         return lambda: torch.autograd.grad(y, x, grad_outputs=dy, retain_graph=True)
334: 
335:     def liger(self, args, kwargs=None) -> Any:
336:         from liger_kernel.transformers.softmax import LigerSoftmax
337: 
338:         if kwargs is not None:
339:             raise AssertionError(f"Expected kwargs to be None, but got {kwargs}")
340:         x, dy = args
341:         softmax = LigerSoftmax().to("cuda")
342:         y = softmax(x)
343:         return lambda: torch.autograd.grad(y, x, grad_outputs=dy, retain_graph=True)
344: 
345:     def benchmark(self):
346:         for M, N in self.get_shapes():
347:             print(f"Tensor dimensions: [{M}, {N}]")
348:             torch_dtype = cutlass_torch.dtype(cutlass.BFloat16)
349:             x = 0.1 * torch.randn(
350:                 M, N, device="cuda", dtype=torch_dtype, requires_grad=True
351:             )
352:             dy = torch.randn(M, N, device="cuda", dtype=torch_dtype)
353:             self.benchmark_single_shape((x, dy), setting=f"shape: [{M}, {N}]")
354: 
355: 
356: class RMSNormForward(BenchmarkKernel):
357:     def __init__(self, script_args):
358:         super().__init__(script_args)
359:         self.available_backends = ["eager", "compiled", "quack", "liger"]
360: 
````
- EN: Handles module imports such as `quack.softmax`, `liger_kernel.transformers.softmax`.
- CN: 处理模块导入，例如 `quack.softmax`, `liger_kernel.transformers.softmax`。
- EN: Declares or extends types including `RMSNormForward`.
- CN: 声明或扩展类型，包括 `RMSNormForward`。
- EN: Implements callable logic such as `get_memory_bytes`, `eager`, `compiled`, `quack`.
- CN: 实现可调用逻辑，例如 `get_memory_bytes`, `eager`, `compiled`, `quack`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。

### Lines 361-420
````python
361:     def get_shapes(self) -> tuple[tuple[int, ...], ...]:
362:         return (
363:             (32768, 256),
364:             (32768, 512),
365:             (32768, 1024),
366:             (32768, 2048),
367:             (32768, 4096),
368:             (32768, 8192),
369:             (32768, 16384),
370:             (32768, 32768),
371:             (32768, 65536),
372:             (16384, 131072),
373:             (8192, 262144),
374:         ) + extra_shapes_for_norm
375: 
376:     def get_memory_bytes(self, args, kwargs) -> int:
377:         x, w = args
378:         M, N = x.shape
379:         return 2 * M * N * x.dtype.itemsize + N * w.dtype.itemsize
380: 
381:     def rms_norm_ref(self, x, w):
382:         x_f32 = x.float()
383:         return (
384:             x_f32
385:             * torch.rsqrt(torch.mean(x_f32.square(), dim=-1, keepdim=True) + 1e-6)
386:             * w
387:         ).to(x.dtype)
388: 
389:     def eager(self, args, kwargs=None) -> Any:
390:         if kwargs is not None:
391:             raise AssertionError(f"Expected kwargs to be None, but got {kwargs}")
392:         x, w = args
393:         return lambda: self.rms_norm_ref(x, w)
394: 
395:     def compiled(self, args, kwargs=None) -> Any:
396:         if kwargs is not None:
397:             raise AssertionError(f"Expected kwargs to be None, but got {kwargs}")
398:         x, w = args
399: 
400:         # Mark batch size as dynamic for realistic workload
401:         torch._dynamo.mark_dynamic(x, 0)
402: 
403:         compiled_rms_norm = torch.compile(
404:             self.rms_norm_ref, mode=self.compile_mode, fullgraph=True
405:         )
406:         return lambda: compiled_rms_norm(x, w)
407: 
408:     def quack(self, args, kwargs=None) -> Any:
409:         # Note: only supper weight with float32 dtype
410:         from quack.rmsnorm import _rmsnorm_fwd
411: 
412:         x, w = args
413:         y = torch.empty_like(x)
414: 
415:         def quack_fwd():
416:             _rmsnorm_fwd(
417:                 x,
418:                 w,
419:                 out=y,
420:                 bias=None,
````
- EN: Handles module imports such as `quack.rmsnorm`.
- CN: 处理模块导入，例如 `quack.rmsnorm`。
- EN: Implements callable logic such as `get_shapes`, `get_memory_bytes`, `rms_norm_ref`, `eager`.
- CN: 实现可调用逻辑，例如 `get_shapes`, `get_memory_bytes`, `rms_norm_ref`, `eager`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 421-480
````python
421:                 rstd=None,
422:                 residual=None,
423:                 residual_out=None,
424:                 eps=1e-6,
425:             )
426:             return y
427: 
428:         return quack_fwd
429: 
430:     def liger(self, args, kwargs) -> Any:
431:         from liger_kernel.transformers.rms_norm import LigerRMSNorm
432: 
433:         x, w = args
434:         M, N = x.shape
435:         liger_rmsnorm = LigerRMSNorm(hidden_size=N, eps=1e-6).cuda()
436:         liger_rmsnorm.weight.data.copy_(w)
437:         return lambda: liger_rmsnorm(x)
438: 
439:     def benchmark(self):
440:         for M, N in self.get_shapes():
441:             print(f"Tensor dimensions: [{M}, {N}]")
442:             torch_dtype = cutlass_torch.dtype(cutlass.BFloat16)
443:             x = torch.randn(M, N, device="cuda", dtype=torch_dtype)
444:             w = torch.randn(N, device="cuda", dtype=torch.float32)
445:             self.benchmark_single_shape((x, w), setting=f"shape: [{M}, {N}]")
446: 
447: 
448: class RMSNormBackward(BenchmarkKernel):
449:     def __init__(self, script_args):
450:         super().__init__(script_args)
451:         self.available_backends = [
452:             "eager",
453:             "compiled",
454:             "quack",
455:             "liger",
456:         ]
457: 
458:     def get_shapes(self) -> tuple[tuple[int, ...], ...]:
459:         # TODO: OOM for (32768, 65536) on h100
460:         return (
461:             (32768, 256),
462:             (32768, 512),
463:             (32768, 1024),
464:             (32768, 2048),
465:             (32768, 4096),
466:             (32768, 8192),
467:             (32768, 16384),
468:         ) + extra_shapes_for_norm
469: 
470:     def get_memory_bytes(self, args, kwargs) -> int:
471:         x, w, dy = args
472:         # x, dy: [M, N], w: [N]
473:         M, N = x.shape
474:         # Read x, w, dy, write dx, dw
475:         return 3 * M * N * x.dtype.itemsize + 2 * N * w.dtype.itemsize
476: 
477:     def rms_norm_ref(self, x, w):
478:         x_f32 = x.float()
479:         return (
480:             x_f32
````
- EN: Handles module imports such as `liger_kernel.transformers.rms_norm`.
- CN: 处理模块导入，例如 `liger_kernel.transformers.rms_norm`。
- EN: Declares or extends types including `RMSNormBackward`.
- CN: 声明或扩展类型，包括 `RMSNormBackward`。
- EN: Implements callable logic such as `liger`, `benchmark`, `__init__`, `get_shapes`.
- CN: 实现可调用逻辑，例如 `liger`, `benchmark`, `__init__`, `get_shapes`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。

### Lines 481-540
````python
481:             * torch.rsqrt(torch.mean(x_f32.square(), dim=-1, keepdim=True) + 1e-6)
482:             * w
483:         ).to(x.dtype)
484: 
485:     def eager(self, args, kwargs=None) -> Any:
486:         if kwargs is not None:
487:             raise AssertionError(f"Expected kwargs to be None, but got {kwargs}")
488:         x, w, dy = args
489:         y = self.rms_norm_ref(x, w)
490:         return lambda: torch.autograd.grad(
491:             y, [x, w], grad_outputs=dy, retain_graph=True
492:         )
493: 
494:     def compiled(self, args, kwargs=None) -> Any:
495:         if kwargs is not None:
496:             raise AssertionError(f"Expected kwargs to be None, but got {kwargs}")
497:         x, w, dy = args
498:         y = torch.compile(self.rms_norm_ref, mode=self.compile_mode, fullgraph=True)(
499:             x, w
500:         )
501:         return lambda: torch.autograd.grad(
502:             y, [x, w], grad_outputs=dy, retain_graph=True
503:         )
504: 
505:     def compute_rstd(self, x, eps):
506:         return torch.rsqrt(torch.mean(x.float().square(), dim=-1, keepdim=True) + eps)
507: 
508:     def quack(self, args, kwargs=None) -> Any:
509:         if kwargs is not None:
510:             raise AssertionError(f"Expected kwargs to be None, but got {kwargs}")
511:         from quack.rmsnorm import _get_sm_count, _rmsnorm_bwd
512: 
513:         (
514:             x,
515:             w,
516:             dy,
517:         ) = args
518:         M, N = x.shape
519: 
520:         rstd = self.compute_rstd(x, eps=1e-6)
521:         dx = torch.empty_like(x)
522:         sm_count = _get_sm_count(x.size(1), x.device)
523:         dw_partial = torch.empty(
524:             sm_count, x.size(1), device=x.device, dtype=torch.float32
525:         )
526: 
527:         def quack_bwd():
528:             _rmsnorm_bwd(
529:                 x,
530:                 w,
531:                 dy,
532:                 rstd,
533:                 dx,
534:                 dw_partial,
535:                 db_partial=None,
536:                 dresidual_out=None,
537:                 dresidual=None,
538:                 sm_count=sm_count,
539:             )
540:             dw = dw_partial.sum(dim=0).to(w.dtype)
````
- EN: Handles module imports such as `quack.rmsnorm`.
- CN: 处理模块导入，例如 `quack.rmsnorm`。
- EN: Implements callable logic such as `eager`, `compiled`, `compute_rstd`, `quack`.
- CN: 实现可调用逻辑，例如 `eager`, `compiled`, `compute_rstd`, `quack`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 541-600
````python
541:             return dx, dw
542: 
543:         return quack_bwd
544: 
545:     def liger(self, args, kwargs=None) -> Any:
546:         if kwargs is not None:
547:             raise AssertionError(f"Expected kwargs to be None, but got {kwargs}")
548:         from liger_kernel.transformers.rms_norm import LigerRMSNorm
549: 
550:         x, w, dy = args
551:         M, N = x.shape
552:         liger_rmsnorm = LigerRMSNorm(
553:             hidden_size=N, eps=1e-6, casting_mode="gemma"
554:         ).cuda()
555:         liger_rmsnorm.weight.data.copy_(w)
556:         y = liger_rmsnorm(x)
557:         return lambda: torch.autograd.grad(
558:             y, [x, liger_rmsnorm.weight], grad_outputs=dy, retain_graph=True
559:         )
560: 
561:     def benchmark(self):
562:         for M, N in self.get_shapes():
563:             print(f"Tensor dimensions: [{M}, {N}]")
564:             torch_dtype = cutlass_torch.dtype(cutlass.BFloat16)
565:             x = torch.randn(M, N, device="cuda", dtype=torch_dtype, requires_grad=True)
566:             w = torch.randn(N, device="cuda", dtype=torch.float32, requires_grad=True)
567:             dy = torch.randn(M, N, device="cuda", dtype=torch_dtype)
568:             self.benchmark_single_shape((x, w, dy), setting=f"shape: [{M}, {N}]")
569: 
570: 
571: class LayerNormForward(BenchmarkKernel):
572:     def __init__(self, script_args):
573:         super().__init__(script_args)
574:         self.available_backends = ["eager", "compiled", "quack", "liger"]
575: 
576:     def get_shapes(self) -> tuple[tuple[int, ...], ...]:
577:         # OOM for (16384, 131072) on h100
578:         return (
579:             (32768, 256),
580:             (32768, 512),
581:             (32768, 1024),
582:             (32768, 2048),
583:             (32768, 4096),
584:             (32768, 8192),
585:             (32768, 16384),
586:             (32768, 32768),
587:             (32768, 65536),
588:         ) + extra_shapes_for_norm
589: 
590:     def get_memory_bytes(self, args, kwargs) -> int:
591:         x, w = args
592:         M, N = x.shape
593:         # Read x ([M, N]), w ([N]), write y ([M, N])
594:         return 2 * M * N * x.dtype.itemsize + N * w.dtype.itemsize
595: 
596:     def layernorm_ref(self, x: torch.Tensor, w: torch.Tensor, eps: float = 1e-6):
597:         x_f32 = x.float()
598:         return F.layer_norm(x_f32, w.shape, w, None, eps).to(x.dtype)
599: 
600:     def eager(self, args, kwargs=None) -> Any:
````
- EN: Handles module imports such as `liger_kernel.transformers.rms_norm`.
- CN: 处理模块导入，例如 `liger_kernel.transformers.rms_norm`。
- EN: Declares or extends types including `LayerNormForward`.
- CN: 声明或扩展类型，包括 `LayerNormForward`。
- EN: Implements callable logic such as `liger`, `benchmark`, `__init__`, `get_shapes`.
- CN: 实现可调用逻辑，例如 `liger`, `benchmark`, `__init__`, `get_shapes`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。

### Lines 601-660
````python
601:         if kwargs is not None:
602:             raise AssertionError(f"Expected kwargs to be None, but got {kwargs}")
603:         x, w = args
604:         return lambda: self.layernorm_ref(x, w)
605: 
606:     def compiled(self, args, kwargs=None) -> Any:
607:         if kwargs is not None:
608:             raise AssertionError(f"Expected kwargs to be None, but got {kwargs}")
609:         x, w = args
610: 
611:         # Mark batch size as dynamic for realistic workload
612:         torch._dynamo.mark_dynamic(x, 0)
613: 
614:         compiled_layernorm = torch.compile(
615:             self.layernorm_ref, mode=self.compile_mode, fullgraph=True
616:         )
617:         return lambda: compiled_layernorm(x, w, eps=1e-6)
618: 
619:     def quack(self, args, kwargs) -> Any:
620:         # Note: quack layernorm does not support bias
621:         from quack.layernorm import layernorm
622: 
623:         x, w = args
624:         return lambda: layernorm(x, w, eps=1e-6)
625: 
626:     def liger(self, args, kwargs) -> Any:
627:         from liger_kernel.transformers.layer_norm import LigerLayerNorm
628: 
629:         x, w = args
630:         M, N = x.shape
631:         liger_layernorm = LigerLayerNorm(hidden_size=N, eps=1e-6).cuda()
632:         liger_layernorm.weight.data.copy_(w)
633:         liger_layernorm.bias.data.copy_(
634:             torch.zeros(N, device="cuda", dtype=torch.float32)
635:         )
636:         return lambda: liger_layernorm(x)
637: 
638:     def benchmark(self):
639:         for M, N in self.get_shapes():
640:             print(f"Tensor dimensions: [{M}, {N}]")
641:             torch_dtype = cutlass_torch.dtype(cutlass.BFloat16)
642:             x = torch.randn(M, N, device="cuda", dtype=torch_dtype)
643:             w = torch.randn(N, device="cuda", dtype=torch.float32)
644:             self.benchmark_single_shape((x, w), setting=f"shape: [{M}, {N}]")
645: 
646: 
647: class LayerNormBackward(BenchmarkKernel):
648:     def __init__(self, script_args):
649:         super().__init__(script_args)
650:         self.available_backends = ["eager", "compiled", "liger"]
651: 
652:     def get_shapes(self) -> tuple[tuple[int, ...], ...]:
653:         # OOM for (16384, 131072), (8192, 262144)
654:         return (
655:             (32768, 256),
656:             (32768, 512),
657:             (32768, 1024),
658:             (32768, 2048),
659:             (32768, 4096),
660:             (32768, 8192),
````
- EN: Handles module imports such as `quack.layernorm`, `liger_kernel.transformers.layer_norm`.
- CN: 处理模块导入，例如 `quack.layernorm`, `liger_kernel.transformers.layer_norm`。
- EN: Declares or extends types including `LayerNormBackward`.
- CN: 声明或扩展类型，包括 `LayerNormBackward`。
- EN: Implements callable logic such as `compiled`, `quack`, `liger`, `benchmark`.
- CN: 实现可调用逻辑，例如 `compiled`, `quack`, `liger`, `benchmark`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。

### Lines 661-720
````python
661:             (32768, 16384),
662:             (32768, 32768),
663:             (32768, 65536),
664:         ) + extra_shapes_for_norm
665: 
666:     def get_memory_bytes(self, args, kwargs) -> int:
667:         x, w, dy = args
668:         M, N = x.shape
669:         # Read x ([M, N]), w ([N]), dy ([M, N]), write dx ([M, N]), dw ([N])
670:         return (
671:             2 * M * N * x.dtype.itemsize
672:             + 2 * N * w.dtype.itemsize
673:             + M * N * dy.dtype.itemsize
674:         )
675: 
676:     def layernorm_ref(self, x: torch.Tensor, w: torch.Tensor, eps: float = 1e-6):
677:         x_f32 = x.float()
678:         return F.layer_norm(x_f32, w.shape, w, None, eps).to(x.dtype)
679: 
680:     def eager(self, args, kwargs=None) -> Any:
681:         if kwargs is not None:
682:             raise AssertionError(f"Expected kwargs to be None, but got {kwargs}")
683:         x, w, dy = args
684:         y = self.layernorm_ref(x, w)
685:         return lambda: torch.autograd.grad(
686:             y, [x, w], grad_outputs=dy, retain_graph=True
687:         )
688: 
689:     def compiled(self, args, kwargs=None) -> Any:
690:         if kwargs is not None:
691:             raise AssertionError(f"Expected kwargs to be None, but got {kwargs}")
692:         x, w, dy = args
693:         compiled_layernorm = torch.compile(
694:             self.layernorm_ref, mode=self.compile_mode, fullgraph=True
695:         )
696:         y = compiled_layernorm(x, w)
697:         return lambda: torch.autograd.grad(
698:             y, [x, w], grad_outputs=dy, retain_graph=True
699:         )
700: 
701:     def compute_mean_rstd(self, x, eps):
702:         x = x.float()
703: 
704:         var, mean = torch.var_mean(x, dim=-1, keepdim=True, correction=0)
705:         rstd = torch.rsqrt(var + eps)
706:         return mean, rstd
707: 
708:     def liger(self, args, kwargs) -> Any:
709:         """
710:         Call layer_norm_backward directly rather than calling
711:         liger_kernel.transformers.layer_norm.LigerLayerNorm and
712:         torch.autograd.grad.
713: 
714:         The latter fashion saves mean/rstd in x.dtype which can fail
715:         accuracy test. We call layer_norm_backward with fp32 mean and
716:         rstd.
717:         """
718:         from liger_kernel.ops.layer_norm import layer_norm_backward
719: 
720:         x, w, dy = args
````
- EN: Handles module imports such as `liger_kernel.ops.layer_norm`.
- CN: 处理模块导入，例如 `liger_kernel.ops.layer_norm`。
- EN: Implements callable logic such as `get_memory_bytes`, `layernorm_ref`, `eager`, `compiled`.
- CN: 实现可调用逻辑，例如 `get_memory_bytes`, `layernorm_ref`, `eager`, `compiled`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 721-734
````python
721:         eps = 1e-6
722:         mean, rstd = self.compute_mean_rstd(x, eps)
723:         M, N = x.shape
724: 
725:         return lambda: layer_norm_backward(dy, x, w, None, mean, rstd)[0:2]
726: 
727:     def benchmark(self):
728:         for M, N in self.get_shapes():
729:             print(f"Tensor dimensions: [{M}, {N}]")
730:             torch_dtype = cutlass_torch.dtype(cutlass.BFloat16)
731:             x = torch.randn(M, N, device="cuda", dtype=torch_dtype, requires_grad=True)
732:             w = torch.randn(N, device="cuda", dtype=torch.float32, requires_grad=True)
733:             dy = torch.randn(M, N, device="cuda", dtype=torch_dtype)
734:             self.benchmark_single_shape((x, w, dy), setting=f"shape: [{M}, {N}]")
````
- EN: Implements callable logic such as `benchmark`.
- CN: 实现可调用逻辑，例如 `benchmark`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `CrossEntropyForward` / 符号 `CrossEntropyForward`
- Symbol `__init__` / 符号 `__init__`
- Symbol `get_shapes` / 符号 `get_shapes`
- Symbol `get_memory_bytes` / 符号 `get_memory_bytes`

## Dependencies / 依赖关系
- Python imports: `typing`, `cutlass`, `cutlass.torch`, `utils`, `torch`, `torch.nn.functional`, `quack.cross_entropy`, `liger_kernel.transformers.cross_entropy`, `quack.softmax`, `liger_kernel.transformers.softmax`
- Python 导入: `typing`, `cutlass`, `cutlass.torch`, `utils`, `torch`, `torch.nn.functional`, `quack.cross_entropy`, `liger_kernel.transformers.cross_entropy`, `quack.softmax`, `liger_kernel.transformers.softmax`
