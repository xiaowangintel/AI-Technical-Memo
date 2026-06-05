# functional_autograd_benchmark.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/functional_autograd_benchmark/functional_autograd_benchmark.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60
````python
 1: import time
 2: from argparse import ArgumentParser
 3: from collections import defaultdict
 4: from collections.abc import Callable
 5: from typing import Any, NamedTuple
 6: 
 7: import torch
 8: from torch.autograd import functional
 9: 
10: 
11: try:
12:     import functorch as ft
13: 
14:     has_functorch = True
15:     print(f"Found functorch: {ft.__version__}")
16: except ImportError:
17:     has_functorch = False
18: 
19: import audio_text_models
20: import ppl_models
21: import vision_models
22: 
23: from utils import GetterType, InputsType, TimingResultType, to_markdown_table, VType
24: 
25: 
26: def get_task_func(task: str) -> Callable:
27:     def hessian_fwdrev(model, inp, strict=None):
28:         return functional.hessian(
29:             model,
30:             inp,
31:             strict=False,
32:             vectorize=True,
33:             outer_jacobian_strategy="forward-mode",
34:         )
35: 
36:     def hessian_revrev(model, inp, strict=None):
37:         return functional.hessian(model, inp, strict=False, vectorize=True)
38: 
39:     def jacfwd(model, inp, strict=None):
40:         return functional.jacobian(
41:             model, inp, strict=False, vectorize=True, strategy="forward-mode"
42:         )
43: 
44:     def jacrev(model, inp, strict=None):
45:         return functional.jacobian(model, inp, strict=False, vectorize=True)
46: 
47:     if task == "hessian_fwdrev":
48:         return hessian_fwdrev
49:     elif task == "hessian_revrev":
50:         return hessian_revrev
51:     elif task == "jacfwd":
52:         return jacfwd
53:     elif task == "jacrev":
54:         return jacrev
55:     else:
56:         return getattr(functional, task)
57: 
58: 
59: def get_task_functorch(task: str) -> Callable:
60:     @torch.no_grad()
````
- EN: Handles module imports such as `time`, `argparse`, `collections`, `collections.abc`.
- CN: 处理模块导入，例如 `time`, `argparse`, `collections`, `collections.abc`。
- EN: Implements callable logic such as `get_task_func`, `hessian_fwdrev`, `hessian_revrev`, `jacfwd`.
- CN: 实现可调用逻辑，例如 `get_task_func`, `hessian_fwdrev`, `hessian_revrev`, `jacfwd`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 61-120
````python
 61:     def vjp(model, inp, v=None, strict=None):
 62:         if v is None:
 63:             raise AssertionError("v must not be None for vjp")
 64:         out, vjpfunc = ft.vjp(model, *inp)
 65:         return out, vjpfunc(v)
 66: 
 67:     @torch.no_grad()
 68:     def jvp(model, inp, v=None, strict=None):
 69:         if v is None:
 70:             raise AssertionError("v must not be None for jvp")
 71:         return ft.jvp(model, inp, v)
 72: 
 73:     @torch.no_grad()
 74:     def vhp(model, inp, v=None, strict=None):
 75:         if v is None:
 76:             raise AssertionError("v must not be None for vhp")
 77:         argnums = tuple(range(len(inp)))
 78:         _, vjpfunc, aux = ft.vjp(ft.grad_and_value(model, argnums), *inp, has_aux=True)
 79:         return aux, vjpfunc(v)
 80: 
 81:     @torch.no_grad()
 82:     def hvp(model, inp, v=None, strict=None):
 83:         if v is None:
 84:             raise AssertionError("v must not be None for hvp")
 85:         argnums = tuple(range(len(inp)))
 86:         _, hvp_out, aux = ft.jvp(
 87:             ft.grad_and_value(model, argnums), inp, v, has_aux=True
 88:         )
 89:         return aux, hvp_out
 90: 
 91:     @torch.no_grad()
 92:     def jacfwd(model, inp, v=None, strict=None):
 93:         argnums = tuple(range(len(inp)))
 94:         return ft.jacfwd(model, argnums)(*inp)
 95: 
 96:     @torch.no_grad()
 97:     def jacrev(model, inp, v=None, strict=None):
 98:         argnums = tuple(range(len(inp)))
 99:         return ft.jacrev(model, argnums)(*inp)
100: 
101:     @torch.no_grad()
102:     def hessian(model, inp, v=None, strict=None):
103:         argnums = tuple(range(len(inp)))
104:         return ft.hessian(model, argnums=argnums)(*inp)
105: 
106:     @torch.no_grad()
107:     def hessian_fwdrev(model, inp, v=None, strict=None):
108:         argnums = tuple(range(len(inp)))
109:         return ft.jacfwd(ft.jacrev(model, argnums=argnums), argnums=argnums)(*inp)
110: 
111:     @torch.no_grad()
112:     def hessian_revrev(model, inp, v=None, strict=None):
113:         argnums = tuple(range(len(inp)))
114:         return ft.jacrev(ft.jacrev(model, argnums=argnums), argnums=argnums)(*inp)
115: 
116:     if task in locals():
117:         return locals()[task]
118:     elif task == "jacobian":
119:         raise RuntimeError(
120:             "functorch has no equivalent of autograd.functional.jacobian with vectorize=False yet"
````
- EN: Implements callable logic such as `vjp`, `jvp`, `vhp`, `hvp`.
- CN: 实现可调用逻辑，例如 `vjp`, `jvp`, `vhp`, `hvp`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-180
````python
121:         )
122:     else:
123:         raise RuntimeError(f"Unsupported task: {task}")
124: 
125: 
126: # Listing of the different tasks
127: FAST_TASKS_NO_DOUBLE_BACK = [
128:     "vjp",
129: ]
130: 
131: FAST_TASKS = FAST_TASKS_NO_DOUBLE_BACK + [
132:     "vhp",
133:     "jvp",
134: ]
135: 
136: ALL_TASKS_NON_VECTORIZED = FAST_TASKS + ["hvp", "jacobian", "hessian"]
137: 
138: DOUBLE_BACKWARD_TASKS = ["jvp", "hvp", "vhp", "hessian"]
139: 
140: VECTORIZED_TASKS = ["hessian_fwdrev", "hessian_revrev", "jacfwd", "jacrev"]
141: 
142: ALL_TASKS = ALL_TASKS_NON_VECTORIZED + VECTORIZED_TASKS
143: 
144: 
145: # Model definition which contains:
146: # - name: a string with the model name.
147: # - getter: a function to get the model. It takes as input the device on which the model
148: #     will run. It should return the forward function and the parameters (Tensors) used as
149: #     input for the forward function. Note that the forward must *not* have any side effect.
150: # - tasks: the list of recommended tasks that can run in a reasonable amount of time with this model.
151: # - unsupported: the list of tasks that this model cannot run.
152: class ModelDef(NamedTuple):
153:     name: str
154:     getter: GetterType
155:     tasks: list[str]
156:     unsupported: list[str]
157: 
158: 
159: MODELS = [
160:     ModelDef("resnet18", vision_models.get_resnet18, FAST_TASKS, []),
161:     ModelDef("fcn_resnet", vision_models.get_fcn_resnet, FAST_TASKS, []),
162:     ModelDef("detr", vision_models.get_detr, FAST_TASKS, []),
163:     ModelDef("ppl_simple_reg", ppl_models.get_simple_regression, ALL_TASKS, []),
164:     ModelDef("ppl_robust_reg", ppl_models.get_robust_regression, ALL_TASKS, []),
165:     ModelDef("wav2letter", audio_text_models.get_wav2letter, FAST_TASKS, []),
166:     ModelDef(
167:         "deepspeech",
168:         audio_text_models.get_deepspeech,
169:         FAST_TASKS_NO_DOUBLE_BACK,
170:         DOUBLE_BACKWARD_TASKS,
171:     ),
172:     ModelDef("transformer", audio_text_models.get_transformer, FAST_TASKS, []),
173:     ModelDef("multiheadattn", audio_text_models.get_multiheadattn, FAST_TASKS, []),
174: ]
175: 
176: 
177: def get_v_for(model: Callable, inp: InputsType, task: str) -> VType:
178:     v: VType
179: 
180:     if task in ["vjp"]:
````
- EN: Declares or extends types including `ModelDef`.
- CN: 声明或扩展类型，包括 `ModelDef`。
- EN: Implements callable logic such as `get_v_for`.
- CN: 实现可调用逻辑，例如 `get_v_for`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 181-240
````python
181:         out = model(*inp)
182:         v = torch.rand_like(out)
183:     elif task in ["jvp", "hvp", "vhp"]:
184:         if isinstance(inp, tuple):
185:             v = tuple(torch.rand_like(i) for i in inp)
186:         else:
187:             v = torch.rand_like(inp)
188:     else:
189:         v = None
190: 
191:     return v
192: 
193: 
194: def run_once(model: Callable, inp: InputsType, task: str, v: VType, **kwargs) -> None:
195:     func = get_task_func(task)
196: 
197:     if v is not None:
198:         func(model, inp, v=v, strict=True)
199:     else:
200:         func(model, inp, strict=True)
201: 
202: 
203: def run_once_functorch(
204:     model: Callable, inp: InputsType, task: str, v: VType, maybe_check_consistency=False
205: ) -> None:
206:     func = get_task_functorch(task)
207: 
208:     if v is not None:
209:         res = func(model, inp, v=v, strict=True)
210:     else:
211:         res = func(model, inp, strict=True)
212: 
213:     if maybe_check_consistency:
214:         af_func = get_task_func(task)
215:         if v is not None:
216:             expected = af_func(model, inp, v=v, strict=True)
217:         else:
218:             expected = af_func(model, inp, strict=True)
219:         atol = 1e-2 if task == "vhp" else 5e-3
220:         torch.testing.assert_close(
221:             res,
222:             expected,
223:             rtol=1e-5,
224:             atol=atol,
225:             msg=f"Consistency fail for task '{task}'",
226:         )
227: 
228: 
229: def run_model(
230:     model_getter: GetterType, args: Any, task: str, run_once_fn: Callable = run_once
231: ) -> list[float]:
232:     if args.gpu == -1:
233:         device = torch.device("cpu")
234: 
235:         def noop():
236:             pass
237: 
238:         do_sync = noop
239:     else:
240:         device = torch.device(f"cuda:{args.gpu}")
````
- EN: Implements callable logic such as `run_once`, `run_once_functorch`, `run_model`, `noop`.
- CN: 实现可调用逻辑，例如 `run_once`, `run_once_functorch`, `run_model`, `noop`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-300
````python
241:         do_sync = torch.cuda.synchronize
242: 
243:     model, inp = model_getter(device)
244: 
245:     v = get_v_for(model, inp, task)
246: 
247:     # Warmup
248:     # maybe_check_consistency=True checks for consistency between
249:     # functorch vs autograd.functional and is done in run_once_functorch only
250:     run_once_fn(model, inp, task, v, maybe_check_consistency=True)
251: 
252:     elapsed = []
253:     for it in range(args.num_iters):
254:         do_sync()
255:         start = time.time()
256:         run_once_fn(model, inp, task, v)
257:         do_sync()
258:         elapsed.append(time.time() - start)
259: 
260:     return elapsed
261: 
262: 
263: def main():
264:     parser = ArgumentParser("Main script to benchmark functional API of the autograd.")
265:     parser.add_argument(
266:         "--output", type=str, default="", help="Text file where to write the output"
267:     )
268:     parser.add_argument("--num-iters", type=int, default=10)
269:     parser.add_argument(
270:         "--gpu",
271:         type=int,
272:         default=-2,
273:         help="GPU to use, -1 for CPU and -2 for auto-detect",
274:     )
275:     parser.add_argument(
276:         "--run-slow-tasks", action="store_true", help="Run even the slow tasks"
277:     )
278:     parser.add_argument(
279:         "--model-filter",
280:         type=str,
281:         default="",
282:         help="Only run the models in this filter",
283:     )
284:     parser.add_argument(
285:         "--task-filter", type=str, default="", help="Only run the tasks in this filter"
286:     )
287:     parser.add_argument(
288:         "--num-threads",
289:         type=int,
290:         default=10,
291:         help="Number of concurrent threads to use when running on cpu",
292:     )
293:     parser.add_argument("--seed", type=int, default=0, help="The random seed to use.")
294:     args = parser.parse_args()
295: 
296:     results: TimingResultType = defaultdict(defaultdict)
297:     torch.set_num_threads(args.num_threads)
298:     torch.set_num_interop_threads(args.num_threads)
299: 
300:     # This automatically seed cuda if it is available
````
- EN: Implements callable logic such as `main`.
- CN: 实现可调用逻辑，例如 `main`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 301-347
````python
301:     torch.manual_seed(args.seed)
302: 
303:     if args.gpu == -2:
304:         args.gpu = 0 if torch.cuda.is_available() else -1
305: 
306:     for name, model_getter, recommended_tasks, unsupported_tasks in MODELS:
307:         if args.model_filter and name not in args.model_filter:
308:             continue
309:         tasks = ALL_TASKS if args.run_slow_tasks else recommended_tasks
310:         for task in tasks:
311:             if task in unsupported_tasks:
312:                 continue
313:             if args.task_filter and task not in args.task_filter:
314:                 continue
315:             runtimes = run_model(model_getter, args, task)
316: 
317:             runtimes = torch.tensor(runtimes)
318:             mean, var = runtimes.mean(), runtimes.var()
319:             results[name][task] = (mean.item(), var.item())
320:             print(f"Results for model {name} on task {task}: {mean}s (var: {var})")
321: 
322:             if has_functorch:
323:                 try:
324:                     runtimes = run_model(
325:                         model_getter, args, task, run_once_fn=run_once_functorch
326:                     )
327:                 except RuntimeError as e:
328:                     print(
329:                         f"Failed model using Functorch: {name}, task: {task}, Error message: \n\t",
330:                         e,
331:                     )
332:                     continue
333: 
334:                 runtimes = torch.tensor(runtimes)
335:                 mean, var = runtimes.mean(), runtimes.var()
336:                 results[name][f"functorch {task}"] = (mean.item(), var.item())
337:                 print(
338:                     f"Results for model {name} on task {task} using Functorch: {mean}s (var: {var})"
339:                 )
340: 
341:     if args.output:
342:         with open(args.output, "w") as f:
343:             f.write(to_markdown_table(results))
344: 
345: 
346: if __name__ == "__main__":
347:     main()
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `get_task_func` / 符号 `get_task_func`
- Symbol `hessian_fwdrev` / 符号 `hessian_fwdrev`
- Symbol `hessian_revrev` / 符号 `hessian_revrev`
- Symbol `jacfwd` / 符号 `jacfwd`

## Dependencies / 依赖关系
- Python imports: `time`, `argparse`, `collections`, `collections.abc`, `typing`, `torch`, `torch.autograd`, `functorch`, `audio_text_models`, `ppl_models`
- Python 导入: `time`, `argparse`, `collections`, `collections.abc`, `typing`, `torch`, `torch.autograd`, `functorch`, `audio_text_models`, `ppl_models`
