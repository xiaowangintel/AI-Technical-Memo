# benchmark_core.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/operator_benchmark/benchmark_core.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60
````python
 1: import ast
 2: import copy
 3: import csv
 4: import functools
 5: import json
 6: import os
 7: import platform
 8: import timeit
 9: from collections import namedtuple
10: from dataclasses import asdict, dataclass
11: from typing import Any
12: 
13: import benchmark_utils
14: 
15: import numpy as np
16: 
17: import torch
18: 
19: # needs to be imported after torch
20: import torch.utils.cpp_extension as cpp_extension  # noqa: F401
21: from torch.utils.benchmark import Timer
22: 
23: 
24: """Performance microbenchmarks.
25: 
26: This module contains core functionalities for performance microbenchmark tests.
27: """
28: 
29: """
30: This is used to store configs of tests
31: An example input is:
32: TestConfig(test_name='add_M8_N2_K1', input_config='M: 8, N: 2, K: 1',
33:     tag='long', run_backward=False)
34: """
35: TestConfig = namedtuple("TestConfig", "test_name input_config tag run_backward")
36: 
37: 
38: BENCHMARK_TESTER = []
39: 
40: SKIP_OP_LISTS = ["weight_norm_sparsifier_step"]
41: 
42: 
43: def _register_test(*test_metainfo):
44:     """save the metainfo needed to create a test. Currently test_metainfo
45:     takes two different inputs:
46:     1) This input when adds single op to the benchmark
47:      _register_test(configs, pt_bench_op, create_pytorch_op_test_case,
48:                       run_backward=True)
49:     2) This input when adds a list of ops to the benchmark
50:     _register_test(configs, pt_bench_op, create_pytorch_op_test_case,
51:                       run_backward=False,
52:                       op_name_function=op)
53:     """
54:     BENCHMARK_TESTER.append(test_metainfo)
55: 
56: 
57: def _create_test(
58:     bench_op_obj, orig_test_attrs, tags, OperatorTestCase, run_backward, bwd_input
59: ):
60:     """Create tests with the benchmark backend.
````
- EN: Handles module imports such as `ast`, `copy`, `csv`, `functools`.
- CN: 处理模块导入，例如 `ast`, `copy`, `csv`, `functools`。
- EN: Implements callable logic such as `_register_test`, `_create_test`.
- CN: 实现可调用逻辑，例如 `_register_test`, `_create_test`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 61-120
````python
 61:     Args:
 62:         bench_op_obj: an object which instantiated from a subclass of
 63:             TorchBenchmarkBase which includes tensor
 64:             creation and operator execution.
 65:         orig_test_attrs: a dictionary includes test configs.
 66:         tags: a attribute in test config to filter inputs
 67:         OperatorTestCase: a named tuple to save the metadata of an test
 68:         run_backward: a bool parameter indicating backward path
 69:     """
 70:     test_attrs = copy.deepcopy(orig_test_attrs)
 71:     test_attrs = {k: str(v) for k, v in test_attrs.items()}
 72:     ascii_test_attrs = ast.literal_eval(json.dumps(test_attrs))
 73:     input_config = str(ascii_test_attrs)[1:-1].replace("'", "")
 74:     if bwd_input:
 75:         # When auto_set is used, the test name needs to include input.
 76:         test_attrs.update({"bwd": bwd_input})
 77:     test_name = bench_op_obj.test_name(**test_attrs)
 78:     test_config = TestConfig(test_name, input_config, tags, run_backward)
 79:     return OperatorTestCase(bench_op_obj, test_config)
 80: 
 81: 
 82: def _build_test(
 83:     configs, bench_op, OperatorTestCase, run_backward, op_name_function=None
 84: ):
 85:     """Generate PyTorch/Caffe2 tests of operators with different inputs.
 86:     Args:
 87:         configs: a dictionary that has the input shapes
 88:         bench_op: a subclass of TorchBenchmarkBase which includes tensor
 89:             creation and operator execution
 90:         OperatorTestCase: a named tuple to save the metadata of an test
 91:         run_backward: a bool parameter indicating backward path
 92:         op_name_function: a dictionary includes operator name and function
 93:     """
 94:     for config in configs:
 95:         test_attrs = {}
 96:         tags = None
 97:         keep_config = True
 98:         for attr in config:
 99:             # tags is only used in our benchmark backend to filter tests and
100:             # it will be removed from config which is then passed to the init function
101:             # an example of config and atrr is:
102:             # config: [{'M': 16}, {'N': 16}, {'K': 64}, {'tags': 'short'}]
103:             # attr: {'tags': 'short'}
104:             if "tags" in attr:
105:                 tags = attr["tags"]
106:                 continue
107: 
108:             # if 'cuda' is specified in input shape but the testing machines doesn't
109:             # support, we will skip this input
110:             if "cuda" in attr.values():
111:                 if not torch.cuda.is_available():
112:                     keep_config = False
113:                     break
114: 
115:             test_attrs.update(attr)
116: 
117:         if not keep_config:
118:             continue
119: 
120:         if tags is None:
````
- EN: Implements callable logic such as `_build_test`.
- CN: 实现可调用逻辑，例如 `_build_test`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-180
````python
121:             raise ValueError("Missing tags in configs")
122: 
123:         op = bench_op()
124:         if op is None:
125:             raise AssertionError("Can't create test: bench_op() returned None")
126:         # op_name_function is a dictionary which has op_name and op_function.
127:         # an example of op_name_function is:
128:         # {'op_name' : 'abs', 'op_function' : torch.abs}
129:         # op_function is concatenated with the input dict then passed to the init function
130:         # op_name is passed to the set_module_name function
131:         init_dict = copy.deepcopy(test_attrs)
132:         if op_name_function is not None:
133:             op_name = op_name_function["op_name"]
134:             init_dict.update({"op_func": op_name_function["op_func"]})
135:             op.set_module_name(op_name)
136: 
137:         op._set_backward_test(run_backward)
138:         op.init(**init_dict)
139:         op.extract_inputs_tuple()
140: 
141:         if not run_backward:
142:             for attr in vars(op).values():
143:                 if isinstance(attr, torch.nn.Module):
144:                     for param in attr.parameters():
145:                         param.requires_grad = False
146: 
147:         input_name = None
148: 
149:         # _num_inputs_require_grads is used to track the number of tensors
150:         # which use auto_set().
151:         if op._num_inputs_require_grads > 0:
152:             input_name = "all"
153:         yield _create_test(
154:             op, test_attrs, tags, OperatorTestCase, run_backward, input_name
155:         )
156: 
157:         # This for loop is only used when auto_set is used.
158:         # _pass_count counts how many times init has been called.
159:         # _auto_set_counter is reset after init is called.
160:         for i in range(op._num_inputs_require_grads):
161:             op._pass_count += 1
162:             op._auto_set_counter = 0
163: 
164:             # TODO(mingzhe09088): remove this deepcopy when we encounter
165:             # performance issue.
166:             new_op = copy.deepcopy(op)
167:             new_op.init(**init_dict)
168:             # Input name index will start from input1
169:             input_name = i + 1
170:             yield _create_test(
171:                 new_op, test_attrs, tags, OperatorTestCase, run_backward, input_name
172:             )
173: 
174: 
175: class BenchmarkRunner:
176:     """BenchmarkRunner is responsible for benchmarking all the registered
177:     benchmark test groups.
178: 
179:     Attributes:
180:         tag_filter (str): control the benchmarks which matches the tag.
````
- EN: Declares or extends types including `BenchmarkRunner`.
- CN: 声明或扩展类型，包括 `BenchmarkRunner`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 181-240
````python
181:         operator (str): only run benchmark test cases that contains
182:     this filter string in the test case's id.
183:         test_name (str): only run benchmark test cases that matches this filter,
184:         this is a case-sensitive substring match and it happens in
185:         the _keep_test method.
186:     """
187: 
188:     def __init__(self, args):
189:         # TODO: consider time-bound constraints as well.
190:         self.args = args
191:         self.iters = 100
192:         self.has_explicit_iteration_count = False
193:         self.multiplier = 2
194:         self.predefined_minimum_secs = 1
195:         self.max_iters = 1e6
196:         self.use_jit = args.use_jit
197:         self.use_compile = args.use_compile
198:         if self.use_jit and self.use_compile:
199:             raise ValueError(
200:                 "use_jit and use_compile are mutually exclusive, please specify one."
201:             )
202:         self.num_runs = args.num_runs
203:         self.print_per_iter = False
204:         self.output_csv = args.output_csv
205:         self.operator_range = benchmark_utils.get_operator_range(args.operator_range)
206:         # 100 is the default warmup iterations
207:         if self.args.warmup_iterations == -1:
208:             self.args.warmup_iterations = 100
209:         if self.args.iterations and self.args.iterations != -1:
210:             self.has_explicit_iteration_count = True
211:             self.iters = self.args.iterations
212:         # when a specific test is selected by a user, we don't need
213:         # to match the tag anymore
214:         if self.args.test_name is not None:
215:             self.args.tag_filter = None
216: 
217:     def _print_header(self):
218:         DASH_LINE = "-" * 40
219:         print(
220:             f"# {DASH_LINE}\n"
221:             "# PyTorch/Caffe2 Operator Micro-benchmarks\n"
222:             f"# {DASH_LINE}\n"
223:             f"# Tag : {self.args.tag_filter}\n"
224:         )
225:         if self.args.list_tests:
226:             print("# List of tests:")
227:         elif self.args.list_ops:
228:             print("# List of Operators to run:")
229:             self.printed_ops_list = set()
230:             if self.args.operators:
231:                 print(f"# {self.args.operators}")
232: 
233:     def _print_perf_result(self, results, test_case):
234:         if self.args.report_aibench:
235:             # Output for AIBench
236:             # Print out per iteration execution time instead of avg time
237:             return
238:             test_name = "_".join([test_case.framework, test_case.test_config.test_name])
239:             for run in range(self.num_runs):
240:                 print(
````
- EN: Implements callable logic such as `__init__`, `_print_header`, `_print_perf_result`.
- CN: 实现可调用逻辑，例如 `__init__`, `_print_header`, `_print_perf_result`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-300
````python
241:                     f"{test_case.framework}Observer "
242:                     + json.dumps(
243:                         {
244:                             "type": test_name,
245:                             "metric": "latency",
246:                             "unit": "us",
247:                             "value": str(results["reported_run_time_us"[run]]),
248:                         }
249:                     )
250:                 )
251:         else:
252:             print(
253:                 f"# Mode: {'JIT' if self.use_jit else 'Compile' if self.use_compile else 'Eager'}"
254:             )
255:             print(
256:                 f"# Name: {test_case.test_config.test_name}\n# Input: {test_case.test_config.input_config}"
257:             )
258: 
259:             mode = "Backward" if test_case.test_config.run_backward else "Forward"
260:             if self.num_runs > 1:
261:                 for run in range(self.num_runs):
262:                     print(
263:                         f"Run: {run}, {mode} Execution Time (us) : {results['reported_run_time_us'][run]:.3f}"
264:                     )
265:                 print()
266:             else:
267:                 print(
268:                     f"{mode} Execution Time (us) : {results['reported_run_time_us'][0]:.3f}"
269:                 )
270:                 print(f"Peak Memory (KB) : {results['peak_memory']}")
271:                 # Calculate and print memory bandwidth if operator provides memory traffic
272:                 if results.get("memory_bandwidth_gb_s") is not None:
273:                     print(
274:                         f"Memory Bandwidth (GB/s) : {results['memory_bandwidth_gb_s']:.2f}"
275:                     )
276:                 print()
277: 
278:     def _perf_result_to_dict(self, results, test_case):
279:         """This function is the parallel of _print_perf_result, which instead of
280:         writing information to terminal, returns a dictionary.
281:         """
282:         if self.args.report_aibench:
283:             return {}
284: 
285:         out = {
286:             "test_name": test_case.test_config.test_name,
287:             "input_config": test_case.test_config.input_config,
288:             "runtime": (
289:                 "JIT" if self.use_jit else "Compile" if self.use_compile else "Eager"
290:             ),
291:             "run": "Backward" if test_case.test_config.run_backward else "Forward",
292:             "latency": round(results["reported_run_time_us"][0], 3),
293:             "latency unit": "us",
294:             "peak memory": results["peak_memory"],
295:             "memory unit": "KB",
296:             "memory bandwidth": results.get("memory_bandwidth_gb_s"),
297:             "memory bandwidth unit": "GB/s",
298:         }
299: 
300:         # parsing test_case.test_config.input_config, adding it as entries to the 'out' dictionary
````
- EN: Implements callable logic such as `_perf_result_to_dict`.
- CN: 实现可调用逻辑，例如 `_perf_result_to_dict`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 301-360
````python
301:         # input: 'M: 1, N: 1, K: 1, device: cpu'
302:         # output: {'M':'1', 'N':'1', 'K':'1', 'device': 'cpu'}
303:         # splitting the string on unnested commas
304:         def split(s):
305:             open_to_close = {"{": "}", "(": ")", "[": "]"}
306:             break_idxs = [-1]
307:             curr_brackets = []
308:             for i, c in enumerate(s):
309:                 if c in open_to_close:
310:                     curr_brackets.append(c)
311:                 elif c in open_to_close.values():
312:                     if not curr_brackets or open_to_close[curr_brackets[-1]] != c:
313:                         raise AssertionError(
314:                             f"ERROR: not able to parse the string! Mismatched bracket '{c}'"
315:                         )
316:                     curr_brackets.pop()
317:                 elif c == "," and (not curr_brackets):
318:                     break_idxs.append(i)
319:             break_idxs.append(len(s))
320:             out = []
321:             for i in range(len(break_idxs) - 1):
322:                 start, end = break_idxs[i], break_idxs[i + 1]
323:                 out.append(s[start + 1 : end])
324:             return out
325: 
326:         key_vals = split(
327:             test_case.test_config.input_config
328:         )  # 'M: [(32, 16), (64, 32)], ZPB: 2' -> ['M: [(32, 16), (64, 32)]', 'ZPB: 2']
329:         key_vals = [
330:             (key.strip(), value.strip())
331:             for key, value in map(lambda str: str.split(":"), key_vals)  # noqa: C417
332:         ]  # ['M: (32, 16)', 'ZPB: 2'] -> [('M', '(32, 16)'), ('ZPB', '2')]
333:         out.update(key_vals)
334: 
335:         return out
336: 
337:     def _predict_num_iter_needed(self, i):
338:         return i * self.multiplier
339: 
340:     def _iteration_result_is_significant(
341:         self, iters, run_time_sec, curr_test_total_time, has_explicit_iteration_count
342:     ):
343:         """This function decides whether the measured time can be reported based on the
344:         following conditions: 1) the number of iterations is larger than the max_iters.
345:         2) the execution time is larger than the predefined minimum_time
346:         3) the execution time is larger than user defined minimum_time
347:         """
348:         return (
349:             iters > self.max_iters
350:             or run_time_sec > self.predefined_minimum_secs
351:             or has_explicit_iteration_count
352:         ) and curr_test_total_time > self.args.min_time_per_test
353: 
354:     def _launch_forward(self, test_case, iters, print_per_iter):
355:         """Use Python's timeit module to measure execution time (unit: second)."""
356:         cuda_sync = "cuda" in test_case.test_config.test_name
357:         func = test_case.run_forward
358:         if self.use_jit:
359:             func = test_case.run_jit_forward
360:         if self.use_compile:
````
- EN: Implements callable logic such as `split`, `_predict_num_iter_needed`, `_iteration_result_is_significant`, `_launch_forward`.
- CN: 实现可调用逻辑，例如 `split`, `_predict_num_iter_needed`, `_iteration_result_is_significant`, `_launch_forward`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 361-420
````python
361:             func = test_case.run_compile_forward
362: 
363:         if not cuda_sync:
364:             forward_time = timeit.timeit(
365:                 functools.partial(func, iters, print_per_iter, cuda_sync), number=1
366:             )
367:             return forward_time
368:         # Stable timing with Timer
369:         timer = Timer(
370:             stmt="func(iters, print_per_iter, cuda_sync)",
371:             globals={
372:                 "func": func,
373:                 "iters": iters,
374:                 "print_per_iter": print_per_iter,
375:                 "cuda_sync": cuda_sync,
376:             },
377:         )
378:         result = timer.adaptive_autorange(min_run_time=0.0001)
379:         return result.median * iters
380: 
381:     def _launch_backward(self, test_case, iters, print_per_iter=False):
382:         """This function runs forward path of an op to get an output. Then the backward path is executed
383:         and the execution time is reported
384:         """
385:         test_case.run_forward(num_runs=1, print_per_iter=False, cuda_sync=False)
386:         test_case._output_mean()
387:         backward_time = timeit.timeit(
388:             functools.partial(test_case.run_backward, iters, print_per_iter), number=1
389:         )
390:         return backward_time
391: 
392:     def _measure_metrics(self, launch_test, test_case, iters, print_per_iter):
393:         """
394:         This function execute the operator for <iters> iterations then look at the time.
395:         If it's not significant, the number of iterations will be increased before rerun.
396:         The execution stops when the time becomes significant.
397:         """
398:         curr_test_total_time = 0
399:         time_trace = []
400:         peak_memory = 0
401:         input_values = test_case.op_bench.inputs.values()
402:         device, device_module = None, None
403:         if input_values and isinstance(next(iter(input_values)), torch.Tensor):
404:             # The device and device module information are crucial for memory metric calculation,
405:             # In case of ops where inputs are integers (not tensor), memory metrics need not be calculated.
406:             sample_input = next(iter(input_values))
407:             device = sample_input.device
408:             device_module = torch.get_device_module(device.type)
409:         # TODO: add support for cpu memory measurement
410:         while True:
411:             if hasattr(device_module, "reset_peak_memory_stats"):
412:                 device_module.reset_peak_memory_stats(device)
413:             run_time_sec = launch_test(test_case, iters, print_per_iter)
414:             if hasattr(device_module, "synchronize"):
415:                 device_module.synchronize(device)
416:             # Memory measurement process
417:             if hasattr(device_module, "max_memory_allocated"):
418:                 peak_memory = device_module.max_memory_allocated(device)
419:             curr_test_total_time += run_time_sec
420:             # Analyze time after each run to decide if the result is stable
````
- EN: Implements callable logic such as `_launch_backward`, `_measure_metrics`.
- CN: 实现可调用逻辑，例如 `_launch_backward`, `_measure_metrics`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 421-480
````python
421:             results_are_significant = self._iteration_result_is_significant(
422:                 iters,
423:                 run_time_sec,
424:                 curr_test_total_time,
425:                 self.has_explicit_iteration_count,
426:             )
427: 
428:             report_run_time = 1e6 * run_time_sec / iters
429:             time_trace.append(report_run_time)
430:             # Print out the time spent in each epoch in ms
431:             if self.args.report_aibench:
432:                 mode = (
433:                     "JIT"
434:                     if self.use_jit
435:                     else "Compile"
436:                     if self.use_compile
437:                     else "Eager"
438:                 )
439:                 test_name = "_".join(
440:                     [test_case.framework, test_case.test_config.test_name, mode]
441:                 )
442:                 print(
443:                     "PyTorchObserver "
444:                     + json.dumps(
445:                         {
446:                             "type": test_name,
447:                             "metric": "latency",
448:                             "unit": "ms",
449:                             "value": str(report_run_time / 1e3),
450:                         },
451:                     )
452:                 )
453:             if results_are_significant:
454:                 break
455: 
456:             # Re-estimate the hopefully-sufficient
457:             # iteration count, and run the benchmark again...
458:             iters = self._predict_num_iter_needed(iters)
459:         reported_run_time_us = np.percentile(np.array(time_trace), 50)
460:         return reported_run_time_us, peak_memory / 1024
461: 
462:     def _check_keep(self, test_flag, cmd_flag):
463:         return cmd_flag is None or test_flag == cmd_flag
464: 
465:     def _check_operator_first_char(self, test_flag, cmd_flag):
466:         return cmd_flag is None or test_flag[:1].lower() in cmd_flag
467: 
468:     def _check_keep_list(self, test_flag, cmd_flag_list):
469:         return cmd_flag_list is None or any(
470:             test_flag == cmd_flag for cmd_flag in cmd_flag_list
471:         )
472: 
473:     def _check_skip(self, test_module, cmd_flag):
474:         return cmd_flag is None or (test_module not in cmd_flag)
475: 
476:     def _keep_test(self, test_case):
477:         # TODO: consider regex matching for test filtering.
478:         # Currently, this is a sub-string matching.
479:         op_test_config = test_case.test_config
480: 
````
- EN: Implements callable logic such as `_check_keep`, `_check_operator_first_char`, `_check_keep_list`, `_check_skip`.
- CN: 实现可调用逻辑，例如 `_check_keep`, `_check_operator_first_char`, `_check_keep_list`, `_check_skip`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 481-540
````python
481:         operators = (
482:             benchmark_utils.process_arg_list(self.args.operators)
483:             if self.args.operators
484:             else None
485:         )
486: 
487:         # Filter framework, operator, test_name, tag, forward_only
488:         return (
489:             self._check_keep(op_test_config.test_name, self.args.test_name)
490:             and self._check_keep_list(test_case.op_bench.module_name(), operators)
491:             and self._check_skip(test_case.op_bench.module_name(), SKIP_OP_LISTS)
492:             and self._check_operator_first_char(
493:                 test_case.op_bench.module_name(), self.operator_range
494:             )
495:             and (
496:                 self.args.tag_filter == "all"
497:                 or self._check_keep(op_test_config.tag, self.args.tag_filter)
498:             )
499:             and (
500:                 not self.args.forward_only
501:                 or op_test_config.run_backward != self.args.forward_only
502:             )
503:             and (
504:                 self.args.device == "None"
505:                 or "device" not in test_case.test_config.input_config
506:                 or self.args.device in op_test_config.test_name
507:             )
508:         )
509: 
510:     def _print_test_case_info(self, test_case):
511:         # Print out the test name and skip the real execution
512:         if self.args.list_tests:
513:             print(f"# {test_case.test_config.test_name}")
514:             return True
515:         elif self.args.list_ops:
516:             if self.args.operators is None:
517:                 op_name = test_case.op_bench.module_name()
518: 
519:                 if op_name not in self.printed_ops_list:
520:                     print(f"# {op_name}")
521:                     self.printed_ops_list.add(op_name)
522:             return True
523: 
524:         return False
525: 
526:     def _output_csv(self, filename, headers, row):
527:         if os.path.exists(filename):
528:             with open(filename) as fd:
529:                 lines = list(csv.reader(fd)) or [[]]
530:                 if headers and len(headers) > len(lines[0]):
531:                     # if prior results failed the header might not be filled in yet
532:                     lines[0] = headers
533:                 else:
534:                     headers = lines[0]
535:         else:
536:             lines = [headers]
537:         lines.append([(f"{x:.6f}" if isinstance(x, float) else x) for x in row])
538:         with open(filename, "w") as fd:
539:             writer = csv.writer(fd, lineterminator="\n")
540:             for line in lines:
````
- EN: Implements callable logic such as `_print_test_case_info`, `_output_csv`.
- CN: 实现可调用逻辑，例如 `_print_test_case_info`, `_output_csv`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 541-600
````python
541:                 writer.writerow(list(line) + ["0"] * (len(headers) - len(line)))
542: 
543:     def _output_json(
544:         self,
545:         perf_list,
546:         output_file,
547:         benchmark_name="PyTorch operator benchmark",
548:     ):
549:         """
550:         Write the result into JSON format, so that it can be uploaded to the benchmark database
551:         to be displayed on OSS dashboard. The JSON format is defined at
552:         https://github.com/pytorch/pytorch/wiki/How-to-integrate-with-PyTorch-OSS-benchmark-database
553:         """
554:         if not perf_list:
555:             return
556: 
557:         # Prepare headers and records for JSON output
558:         records = []
559:         for perf_item in perf_list:
560:             # Extract data from perf_item
561:             test_name = perf_item.get("test_name", "unknown")
562:             input_config = perf_item.get("input_config", "")
563:             run_type = perf_item.get("run")
564:             latency = perf_item.get("latency", 0)
565:             peak_memory = perf_item.get("peak memory", 0)
566:             memory_bandwidth = perf_item.get("memory bandwidth", 0)
567:             device = perf_item.get("device", "unknown")
568:             dtype = perf_item.get("dtype", "torch.float").split(".")[1]
569:             runtime = perf_item.get("runtime", None)
570: 
571:             # Extract mode based on run_type
572:             mode = None
573:             if run_type == "Forward":
574:                 mode = "inference"
575:             elif run_type == "Backward":
576:                 mode = "training"
577: 
578:             # Extract use_compile from it
579:             if runtime == "Compile":
580:                 use_compile = True
581:             elif runtime == "Eager":
582:                 use_compile = False
583:             else:
584:                 use_compile = None
585: 
586:             device_arch = (
587:                 torch.cuda.get_device_name(0)
588:                 if device == "cuda"
589:                 else platform.processor()
590:                 if device == "cpu"
591:                 else "unknown"
592:             )
593: 
594:             # Extract operator name from test_name
595:             operator_name = test_name.split("_")[0]
596: 
597:             # Create the record
598:             @dataclass
599:             class BenchmarkInfo:
600:                 name: str
````
- EN: Declares or extends types including `BenchmarkInfo`.
- CN: 声明或扩展类型，包括 `BenchmarkInfo`。
- EN: Implements callable logic such as `_output_json`.
- CN: 实现可调用逻辑，例如 `_output_json`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 601-660
````python
601:                 mode: str | None
602:                 dtype: str
603:                 extra_info: dict[str, Any]
604: 
605:             @dataclass
606:             class ModelInfo:
607:                 name: str
608:                 type: str
609:                 origins: list[str]
610:                 extra_info: dict[str, Any]
611: 
612:             @dataclass
613:             class MetricInfo:
614:                 name: str
615:                 unit: str
616:                 benchmark_values: list[float]
617:                 target_value: float | None
618: 
619:             @dataclass
620:             class BenchmarkRecord:
621:                 benchmark: BenchmarkInfo
622:                 model: ModelInfo
623:                 metric: MetricInfo
624: 
625:             # Add record for latency
626:             record_latency = BenchmarkRecord(
627:                 benchmark=BenchmarkInfo(
628:                     name=benchmark_name,
629:                     mode=mode,
630:                     dtype=dtype,
631:                     extra_info={
632:                         "input_config": input_config,
633:                         "device": device,
634:                         "arch": device_arch,
635:                         "use_compile": use_compile,
636:                         "operator_name": operator_name,
637:                     },
638:                 ),
639:                 model=ModelInfo(
640:                     name=test_name,
641:                     type="micro-benchmark",
642:                     origins=["pytorch"],
643:                     extra_info={"operator_name": operator_name},
644:                 ),
645:                 metric=MetricInfo(
646:                     name="latency",
647:                     unit="us",
648:                     benchmark_values=[latency],
649:                     target_value=None,
650:                 ),
651:             )
652:             records.append(asdict(record_latency))
653: 
654:             # Add record for peak memory
655:             record_memory = copy.deepcopy(record_latency)
656:             record_memory.metric = MetricInfo(
657:                 name="peak memory",
658:                 unit="KB",
659:                 benchmark_values=[peak_memory],
660:                 target_value=None,
````
- EN: Declares or extends types including `ModelInfo`, `MetricInfo`, `BenchmarkRecord`.
- CN: 声明或扩展类型，包括 `ModelInfo`, `MetricInfo`, `BenchmarkRecord`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 661-720
````python
661:             )
662:             records.append(asdict(record_memory))
663: 
664:             # Add record for memory bandwidth
665:             record_memory_bandwidth = copy.deepcopy(record_latency)
666:             record_memory_bandwidth.metric = MetricInfo(
667:                 name="memory bandwidth",
668:                 unit="GB/s",
669:                 benchmark_values=[memory_bandwidth],
670:                 target_value=None,
671:             )
672:             records.append(asdict(record_memory_bandwidth))
673: 
674:         # Write all records to the output file
675:         with open(output_file, "w", encoding="utf-8") as f:
676:             json.dump(records, f, indent=2)
677: 
678:     def run(self):
679:         self._print_header()
680:         output_csv_filename = self.args.output_csv
681:         headers = [
682:             "Benchmarking Framework",
683:             "Benchmarking Module Name",
684:             "Case Name",
685:             "tag",
686:             "run_backward",
687:             "Execution Time",
688:             "Peak Memory (KB)",
689:             "Memory Bandwidth (GB/s)",
690:         ]
691: 
692:         if self.args.output_json or self.args.output_json_for_dashboard:
693:             perf_list = []
694: 
695:         for test_metainfo in BENCHMARK_TESTER:
696:             for test in _build_test(*test_metainfo):
697:                 full_test_id, test_case = test
698:                 op_test_config = test_case.test_config
699: 
700:                 if self._print_test_case_info(test_case):
701:                     continue
702: 
703:                 if not self._keep_test(test_case):
704:                     continue
705: 
706:                 # To reduce variance, fix a numpy randseed to the test case,
707:                 # so that the randomly generated input tensors remain the
708:                 # same for each test case.
709:                 # The random seed is limited to 32-bit because of numpy
710:                 # requirement.
711:                 np.random.seed(seed=hash(full_test_id) & ((1 << 32) - 1))
712: 
713:                 print(
714:                     f"# Benchmarking {test_case.framework}: {test_case.op_bench.module_name()}"
715:                 )
716: 
717:                 if op_test_config.run_backward:
718:                     launch_func = self._launch_backward
719:                 else:
720:                     launch_func = self._launch_forward
````
- EN: Implements callable logic such as `run`.
- CN: 实现可调用逻辑，例如 `run`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 721-778
````python
721: 
722:                 # Warmup
723:                 launch_func(
724:                     test_case, self.args.warmup_iterations, print_per_iter=False
725:                 )
726:                 # Actual Execution
727:                 results = [
728:                     self._measure_metrics(
729:                         launch_func, test_case, self.iters, self.print_per_iter
730:                     )
731:                     for _ in range(self.num_runs)
732:                 ]
733:                 result_dict = dict()
734:                 result_dict["reported_run_time_us"] = [r[0] for r in results]
735:                 result_dict["peak_memory"] = results[0][1]
736: 
737:                 # Calculate memory bandwidth if operator provides memory traffic
738:                 memory_traffic_bytes = test_case.op_bench.get_memory_traffic_bytes()
739:                 if memory_traffic_bytes is not None:
740:                     execution_time_s = result_dict["reported_run_time_us"][0] / 1e6
741:                     result_dict["memory_bandwidth_gb_s"] = (
742:                         memory_traffic_bytes / execution_time_s / 1e9
743:                     )
744:                 else:
745:                     result_dict["memory_bandwidth_gb_s"] = None
746: 
747:                 self._print_perf_result(results=result_dict, test_case=test_case)
748: 
749:                 # output results to csv
750:                 self._output_csv(
751:                     output_csv_filename,
752:                     headers,
753:                     [
754:                         test_case.framework,
755:                         test_case.op_bench.module_name(),
756:                         (
757:                             test_case.test_config.test_name + "_BACKWARD"
758:                             if test_case.test_config.run_backward is True
759:                             else test_case.test_config.test_name
760:                         ),
761:                         test_case.test_config.tag,
762:                         test_case.test_config.run_backward,
763:                         result_dict["reported_run_time_us"][0],
764:                         result_dict["peak_memory"],
765:                         result_dict["memory_bandwidth_gb_s"],
766:                     ],
767:                 )
768:                 if self.args.output_json or self.args.output_json_for_dashboard:
769:                     perf_list.append(self._perf_result_to_dict(result_dict, test_case))
770: 
771:         if self.args.output_json_for_dashboard:
772:             self._output_json(
773:                 perf_list, self.args.output_json_for_dashboard, self.args.benchmark_name
774:             )
775: 
776:         if self.args.output_json:
777:             with open(self.args.output_json, "w") as f:
778:                 json.dump(perf_list, f)
````
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `_register_test` / 符号 `_register_test`
- Symbol `_create_test` / 符号 `_create_test`
- Symbol `_build_test` / 符号 `_build_test`
- Symbol `BenchmarkRunner` / 符号 `BenchmarkRunner`

## Dependencies / 依赖关系
- Python imports: `ast`, `copy`, `csv`, `functools`, `json`, `os`, `platform`, `timeit`, `collections`, `dataclasses`
- Python 导入: `ast`, `copy`, `csv`, `functools`, `json`, `os`, `platform`, `timeit`, `collections`, `dataclasses`
