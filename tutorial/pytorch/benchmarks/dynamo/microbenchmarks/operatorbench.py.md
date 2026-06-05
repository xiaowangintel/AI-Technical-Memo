# operatorbench.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/dynamo/microbenchmarks/operatorbench.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60
````python
 1: #!/usr/bin/env python3
 2: import csv
 3: import itertools
 4: import sys
 5: import time
 6: import warnings
 7: from contextlib import nullcontext
 8: 
 9: import click
10: import numpy as np
11: from operator_inp_utils import OperatorInputsLoader
12: from tqdm import tqdm
13: 
14: import torch
15: from torch._dynamo.backends.cudagraphs import cudagraphs_inner
16: from torch._dynamo.testing import same
17: from torch._inductor.compile_fx import compile_fx
18: from torch._inductor.decomposition import decompositions
19: from torch._inductor.lowering import lowerings
20: from torch._inductor.runtime.benchmarking import benchmarker
21: from torch._inductor.utils import gen_gm_and_inputs
22: from torch.utils._pytree import tree_map_only
23: 
24: 
25: aten = torch.ops.aten
26: profile_enabled = False
27: inductor_config_options = {
28:     "halide": {"cpu_backend": "halide", "cuda_backend": "halide"},
29:     "autotune": {
30:         "max_autotune_pointwise": True,
31:         "max_autotune": True,
32:         "max_autotune_gemm": True,
33:         "coordinate_descent_tuning": True,
34:     },
35: }
36: 
37: 
38: def maybe_record_function(name):
39:     return torch.profiler.record_function(name) if profile_enabled else nullcontext()
40: 
41: 
42: def compute_speedups(
43:     operator, models, example_inputs, repeats, accuracy_checking=False, device="cuda"
44: ):
45:     expected = models[0](*example_inputs)
46:     if accuracy_checking:
47:         for model in models[1:]:
48:             actual = model(*example_inputs)
49:             # change to assert later
50:             try:
51:                 same(actual, expected, cos_similarity=True, equal_nan=True)
52:             except AssertionError as e:
53:                 print(e)
54:                 print(f"Accuracy check failed: {operator}")
55:                 print((expected[0] - actual[0]).abs().max())
56: 
57:     timings = np.zeros((repeats, len(models)), np.float64)
58:     for rep in range(repeats):
59:         with maybe_record_function(f"rep_{rep}"):
60:             # interleave the runs to handle frequency scaling and load changes
````
- EN: Handles module imports such as `csv`, `itertools`, `sys`, `time`.
- CN: 处理模块导入，例如 `csv`, `itertools`, `sys`, `time`。
- EN: Implements callable logic such as `maybe_record_function`, `compute_speedups`.
- CN: 实现可调用逻辑，例如 `maybe_record_function`, `compute_speedups`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 61-120
````python
 61:             for m, model in enumerate(models):
 62:                 with maybe_record_function(f"model_{m}"):
 63:                     if device == "cuda":
 64:                         model(*example_inputs)
 65: 
 66:                         # benchmarker.benchmark_gpu() clears L2 cache to hide the latency of CPU launch time
 67:                         # along with cuda synchronization
 68:                         timings[rep, m] = benchmarker.benchmark_gpu(
 69:                             lambda: model(*example_inputs)
 70:                         )
 71:                     else:
 72:                         from torch._inductor.utils import timed
 73: 
 74:                         timings[rep, m] = timed(model, example_inputs)
 75:     return np.median(timings, axis=0)
 76: 
 77: 
 78: def strip_overloads(gm):
 79:     """
 80:     Modifies the target of graph nodes in :attr:`gm` to strip overloads.
 81:     Args:
 82:         gm(fx.GraphModule): The input Fx graph module to be modified
 83:     """
 84:     for node in gm.graph.nodes:
 85:         if isinstance(node.target, torch._ops.OpOverload):
 86:             node.target = node.target.overloadpacket
 87:     gm.recompile()
 88: 
 89: 
 90: def convert_to_jit(gm, gm_args):
 91:     strip_overloads(gm)
 92:     try:
 93:         return torch.jit.script(gm)
 94:     except Exception:
 95:         pass
 96:     return torch.jit.trace(gm, gm_args)
 97: 
 98: 
 99: def to_channels_last(ten):
100:     return ten if ten.ndim != 4 else ten.to(memory_format=torch.channels_last)
101: 
102: 
103: def microbenchmark(
104:     operator,
105:     args,
106:     kwargs,
107:     accuracy_checking,
108:     repeats,
109:     inductor_configs,
110:     measure_nvfuser,
111:     device,
112: ):
113:     gm, gm_args = gen_gm_and_inputs(operator, args, kwargs)
114:     torch.jit._builtins._register_builtin(
115:         torch.ops.aten.convolution_backward.default, "aten::convolution_backward"
116:     )
117:     compiled = [gm]
118:     for config in inductor_configs:
119:         t = -time.perf_counter()
120:         compiled.append(compile_fx(gm, gm_args, config_patches=config))
````
- EN: Handles module imports such as `torch._inductor.utils`.
- CN: 处理模块导入，例如 `torch._inductor.utils`。
- EN: Implements callable logic such as `strip_overloads`, `convert_to_jit`, `to_channels_last`, `microbenchmark`.
- CN: 实现可调用逻辑，例如 `strip_overloads`, `convert_to_jit`, `to_channels_last`, `microbenchmark`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 121-180
````python
121:         t += time.perf_counter()
122:         if t > 10:
123:             print(f"slow compile inductor {t:.1f}s {config}")
124: 
125:     if measure_nvfuser:
126:         g = convert_to_jit(gm, gm_args)
127:         cudagraphs_jit = cudagraphs_inner(
128:             g, gm_args, copy_outputs=False, copy_inputs=False
129:         )
130:         compiled += [cudagraphs_jit]
131:     if accuracy_checking:
132:         repeats = 1
133: 
134:     medians = compute_speedups(
135:         operator, compiled, gm_args, repeats, accuracy_checking, device
136:     )
137:     return medians
138: 
139: 
140: quantiles_thresholds = (0.2, 0.5, 0.8)
141: 
142: 
143: def quantiles(timings):
144:     return np.quantile(timings, quantiles_thresholds).tolist()
145: 
146: 
147: def skip_operator(operator):
148:     nyi_strings = (
149:         "aten.gather.default",
150:         "nll_loss",
151:         "aten.index",
152:         "aten.scatter_",
153:         "masked_fill_.Scalar",
154:     )
155: 
156:     if any(nyi_string in str(operator) for nyi_string in nyi_strings):
157:         # maybe disable aten.native_layer_norm.default
158:         # TODO - inputs cannot be randomly initialized, causes cyda failures
159:         print(f"Skipping {operator}, input generator nyi")
160:         return True
161: 
162:     # not covered by other non-compute operator heuristics
163:     if operator == torch.ops.aten._unsafe_view.default:
164:         print(f"Skipping {operator}, non compute operator")
165:         return True
166: 
167:     # some of inductor registered to the OpOverload, some registered to OpOverloadPacket
168:     op_impls = [operator]
169:     if isinstance(operator, torch._ops.OpOverload):
170:         op_impls.append(operator.overloadpacket)
171: 
172:     # TODO - skip benchmarking fallbacks. for some ops we have both lowerings and fallbacks
173:     # so its not clear just from operator what will be lowered.
174: 
175:     if all(op not in decompositions and op not in lowerings for op in op_impls):
176:         print(f"Skipping {operator}, no inductor impl")
177:         return True
178: 
179:     if "convolution" in str(operator):
180:         return True
````
- EN: Implements callable logic such as `quantiles`, `skip_operator`.
- CN: 实现可调用逻辑，例如 `quantiles`, `skip_operator`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 181-240
````python
181: 
182:     return False
183: 
184: 
185: @click.command()
186: @click.option(
187:     "--suite",
188:     help="suite to load inps from: options: timm, huggingface, torchbench",
189:     default="torchbench",
190: )
191: @click.option("--op", help="operator overload to benchmark", default="all")
192: @click.option("--dtype", help="dtype to benchmark", default="float32")
193: @click.option("--max-samples", help="max samples per op", default=15)
194: @click.option("--accuracy-checking", help="check accuracy", default=False)
195: @click.option(
196:     "--repeats", help="how many times to repeat for perf measurement", default=3
197: )
198: @click.option(
199:     "--inductor-config",
200:     multiple=True,
201:     help="Custom inductor config, options: " + ", ".join(inductor_config_options),
202: )
203: @click.option(
204:     "--measure-nvfuser/--no-measure-nvfuser",
205:     help="default we only measure inductor",
206:     default=False,
207: )
208: @click.option("--device", help="cpu or cuda", default="cuda")
209: @click.option("--inp-file", help="use custom input file instead of suite", default=None)
210: @click.option("--start-idx", help="specify start index of samples", default=0)
211: @click.option(
212:     "--channels-last", help="force inputs to channels last", is_flag=True, default=False
213: )
214: @click.option("--profile", help="profile the benchmark", is_flag=True, default=False)
215: def benchmark(
216:     suite,
217:     op,
218:     dtype,
219:     max_samples,
220:     accuracy_checking,
221:     repeats,
222:     inductor_config,
223:     measure_nvfuser,
224:     device,
225:     inp_file,
226:     start_idx,
227:     channels_last,
228:     profile,
229: ):
230:     warnings.filterwarnings("ignore", module="torch.jit._check")
231:     torch.set_float32_matmul_precision("high")
232:     global profile_enabled
233: 
234:     if inp_file is not None:
235:         loader = OperatorInputsLoader(inp_file)
236:     else:
237:         if suite not in ("timm", "huggingface", "torchbench"):
238:             raise AssertionError(
239:                 f"suite must be one of 'timm', 'huggingface', 'torchbench', but got '{suite}'"
240:             )
````
- EN: Implements callable logic such as `benchmark`.
- CN: 实现可调用逻辑，例如 `benchmark`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-300
````python
241:         if suite == "timm":
242:             loader = OperatorInputsLoader.get_timm_loader()
243:         elif suite == "huggingface":
244:             loader = OperatorInputsLoader.get_huggingface_loader()
245:         else:
246:             loader = OperatorInputsLoader.get_torchbench_loader()
247: 
248:     if dtype not in ("float16", "float32"):
249:         raise AssertionError(f"dtype must be 'float16' or 'float32', but got '{dtype}'")
250: 
251:     inductor_configs = [{}]
252:     backend_names = ["inductor"]
253:     for name in inductor_config or ():
254:         backend_names.append(name)
255:         inductor_configs.append(inductor_config_options[name])
256:     if measure_nvfuser:
257:         backend_names.append("nvfuser")
258: 
259:     compare2 = len(backend_names) == 2
260:     if compare2:
261:         a, b = backend_names
262:         backend_names.append(f"{a}/{b}")
263: 
264:     output_fd = None
265:     output_csv = None
266:     if op == "all":
267:         filename = f"operatorbench_{suite}_{dtype}.csv"
268:         with open(filename, "w") as output_fd:
269:             output_csv = csv.writer(output_fd)
270:             output_csv.writerow(
271:                 [
272:                     "operator",
273:                     *[
274:                         f"{a} {b}"
275:                         for a, b in itertools.product(
276:                             backend_names,
277:                             [f"{x * 100:.0f}th" for x in quantiles_thresholds],
278:                         )
279:                     ],
280:                     "elapsed",
281:                     *map("{} abs".format, ["eager", *backend_names]),
282:                 ]
283:             )
284: 
285:     dtype = torch.float16 if dtype == "float16" else torch.float32
286: 
287:     if op == "all":
288:         ops = loader.get_all_ops()
289:     else:
290:         ops = [eval(op)]
291: 
292:     max_samples = max_samples + start_idx
293:     profile_enabled = profile
294: 
295:     for operator in ops:
296:         if skip_operator(operator):
297:             continue
298:         start = time.perf_counter()
299:         inp_gen = loader.get_inputs_for_operator(operator, dtype=dtype, device=device)
300:         timings = []
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 301-360
````python
301:         inputs_list = []
302:         for _ in range(min(max_samples, 1000000)):
303:             try:
304:                 inps = next(inp_gen)
305:                 inputs_list.append(inps)
306:             except StopIteration:
307:                 break
308: 
309:         profiler_context = (
310:             torch.profiler.profile(
311:                 activities=[
312:                     torch.profiler.ProfilerActivity.CPU,
313:                     torch.profiler.ProfilerActivity.CUDA,
314:                 ],
315:                 record_shapes=False,
316:                 profile_memory=False,
317:                 on_trace_ready=torch.profiler.tensorboard_trace_handler(
318:                     f"./log/operator_{operator}", use_gzip=True
319:                 ),
320:             )
321:             if profile_enabled
322:             else nullcontext()
323:         )
324:         with profiler_context:
325:             for i, inps in enumerate(tqdm(inputs_list[start_idx:], desc=str(operator))):
326:                 if inps is None:
327:                     break
328:                 args, kwargs = inps
329:                 if channels_last:
330:                     args, kwargs = tree_map_only(
331:                         torch.Tensor, to_channels_last, (args, kwargs)
332:                     )
333:                 try:
334:                     with maybe_record_function(f"iter_{i}"):
335:                         # aten, nvfuser, inductor
336:                         timings.append(
337:                             microbenchmark(
338:                                 operator,
339:                                 args,
340:                                 kwargs,
341:                                 accuracy_checking,
342:                                 repeats,
343:                                 inductor_configs,
344:                                 measure_nvfuser,
345:                                 device,
346:                             )
347:                         )
348:                 except Exception as e:
349:                     print(f"error {operator} input {i}: {type(e).__name__}: {e}")
350:                     # comment out this line to avoid blocking other tests
351:                     # raise e
352: 
353:         if not timings:
354:             continue
355: 
356:         timings = np.stack(timings)
357:         speedups = [
358:             quantiles(timings[:, 0] / timings[:, x]) for x in range(1, timings.shape[1])
359:         ]
360:         if compare2:
````
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 361-387
````python
361:             speedups.append(quantiles(timings[:, 1] / timings[:, 2]))
362:         if len(backend_names) != len(speedups):
363:             raise AssertionError(
364:                 f"Expected {len(backend_names)} speedups for {len(backend_names)} backends, but got {len(speedups)}"
365:             )
366: 
367:         row = [f"{operator}"]
368:         sys.stdout.write(f"{operator}: ")
369:         for backend, (low, mid, high) in zip(backend_names, speedups):
370:             sys.stdout.write(f"{backend}={mid:.4f}x ({low:.4f}-{high:.4f}) ")
371:             row.extend(map("{:.6f}".format, [low, mid, high]))
372:         elapsed = time.perf_counter() - start
373:         row.append(f"{elapsed:1f}")
374:         row.extend(map("{:.8f}".format, np.mean(timings, axis=0).tolist()))
375:         sys.stdout.write(f"took {elapsed:.0f}s\n")
376:         sys.stdout.flush()
377:         if output_csv:
378:             output_csv.writerow(row)
379:             output_fd.flush()
380: 
381:     if output_fd:
382:         print(f"Wrote {filename}")
383:         output_fd.close()
384: 
385: 
386: if __name__ == "__main__":
387:     benchmark()
````
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `maybe_record_function` / 符号 `maybe_record_function`
- Symbol `compute_speedups` / 符号 `compute_speedups`
- Symbol `strip_overloads` / 符号 `strip_overloads`
- Symbol `convert_to_jit` / 符号 `convert_to_jit`

## Dependencies / 依赖关系
- Python imports: `csv`, `itertools`, `sys`, `time`, `warnings`, `contextlib`, `click`, `numpy`, `operator_inp_utils`, `tqdm`
- Python 导入: `csv`, `itertools`, `sys`, `time`, `warnings`, `contextlib`, `click`, `numpy`, `operator_inp_utils`, `tqdm`
