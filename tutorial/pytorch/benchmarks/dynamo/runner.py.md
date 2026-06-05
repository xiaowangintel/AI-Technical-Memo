# runner.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/dynamo/runner.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-80
````python
 1: #!/usr/bin/env python3
 2: 
 3: """
 4: A wrapper over the benchmark infrastructure to generate commonly used commands,
 5: parse results and generate csv/graphs.
 6: 
 7: The script works on manually written TABLE (see below). We can add more commands
 8: in the future.
 9: 
10: One example usage is
11: -> python benchmarks/runner.py --suites=torchbench --inference
12: This command will generate the commands for the default compilers (see DEFAULTS
13: below) for inference, run them and visualize the logs.
14: 
15: If you want to just print the commands, you could use the following command
16: -> python benchmarks/runner.py --print-run-commands --suites=torchbench --inference
17: 
18: Similarly, if you want to just visualize the already finished logs
19: -> python benchmarks/runner.py --visualize-logs --suites=torchbench --inference
20: 
21: If you want to test float16
22: -> python benchmarks/runner.py --suites=torchbench --inference --dtypes=float16
23: 
24: """
25: 
26: import argparse
27: import dataclasses
28: import functools
29: import glob
30: import importlib
31: import io
32: import itertools
33: import logging
34: import os
35: import platform
36: import re
37: import shutil
38: import subprocess
39: import sys
40: import tempfile
41: from collections import defaultdict
42: from datetime import datetime, timedelta, timezone
43: from os.path import abspath, exists
44: from random import randint
45: 
46: import matplotlib.pyplot as plt
47: import numpy as np
48: import pandas as pd
49: from matplotlib import rcParams
50: from scipy.stats import gmean
51: from tabulate import tabulate
52: 
53: import torch
54: import torch._dynamo
55: 
56: 
57: rcParams.update({"figure.autolayout": True})
58: plt.rc("axes", axisbelow=True)
59: 
60: DEFAULT_OUTPUT_DIR = "benchmark_logs"
61: 
62: 
63: log = logging.getLogger(__name__)
64: 
65: TABLE = {
66:     "training": {
67:         "ts_nnc": "--training --speedup-ts ",
68:         "ts_nvfuser": "--training --nvfuser --speedup-dynamo-ts ",
69:         "eager": "--training --backend=eager ",
70:         "aot_eager": "--training --backend=aot_eager ",
71:         "cudagraphs": "--training --backend=cudagraphs ",
72:         "aot_nvfuser": "--training --nvfuser --backend=aot_ts_nvfuser ",
73:         "nvprims_nvfuser": "--training --backend=nvprims_nvfuser ",
74:         "inductor": "--training --inductor ",
75:         "inductor_no_cudagraphs": "--training --inductor --disable-cudagraphs ",
76:         "inductor_max_autotune": "--training --inductor --inductor-compile-mode max-autotune ",
77:         "inductor_max_autotune_no_cudagraphs": (
78:             "--training --inductor --inductor-compile-mode max-autotune-no-cudagraphs --disable-cudagraphs "
79:         ),
80:     },
````
- EN: Handles module imports such as `argparse`, `dataclasses`, `functools`, `glob`.
- CN: 处理模块导入，例如 `argparse`, `dataclasses`, `functools`, `glob`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 81-160
````python
 81:     "inference": {
 82:         "aot_eager": "--inference --backend=aot_eager ",
 83:         "eager": "--inference --backend=eager ",
 84:         "ts_nnc": "--inference --speedup-ts ",
 85:         "ts_nvfuser": "--inference -n100 --speedup-ts --nvfuser ",
 86:         "trt": "--inference -n100 --speedup-trt ",
 87:         "ts_nvfuser_cudagraphs": "--inference --backend=cudagraphs_ts ",
 88:         "inductor": "--inference -n50 --inductor ",
 89:         "inductor_no_cudagraphs": "--inference -n50 --inductor --disable-cudagraphs ",
 90:         "inductor_max_autotune": "--inference -n50 --inductor --inductor-compile-mode max-autotune ",
 91:         "inductor_max_autotune_no_cudagraphs": (
 92:             "--inference -n50 --inductor --inductor-compile-mode max-autotune-no-cudagraphs --disable-cudagraphs "
 93:         ),
 94:     },
 95: }
 96: 
 97: INFERENCE_COMPILERS = tuple(TABLE["inference"].keys())
 98: TRAINING_COMPILERS = tuple(TABLE["training"].keys())
 99: 
100: DEFAULTS = {
101:     "training": [
102:         "eager",
103:         "aot_eager",
104:         "inductor",
105:         "inductor_no_cudagraphs",
106:     ],
107:     "inference": [
108:         "eager",
109:         "aot_eager",
110:         "inductor",
111:         "inductor_no_cudagraphs",
112:     ],
113:     "flag_compilers": {
114:         "training": ["inductor", "inductor_no_cudagraphs"],
115:         "inference": ["inductor", "inductor_no_cudagraphs"],
116:     },
117:     "dtypes": [
118:         "float32",
119:     ],
120:     "suites": ["torchbench", "huggingface", "timm_models"],
121:     "devices": [
122:         "cuda",
123:     ],
124:     "quick": {
125:         "torchbench": '-k "resnet..$"',
126:         "huggingface": "-k Albert",
127:         "timm_models": ' -k "^resnet" -k "^inception"',
128:     },
129: }
130: 
131: 
132: DASHBOARD_DEFAULTS = {
133:     "dashboard_image_uploader": "/fsx/users/anijain/bin/imgur.sh",
134:     "dashboard_archive_path": "/data/home/anijain/cluster/cron_logs",
135:     "dashboard_gh_cli_path": "/data/home/anijain/miniconda/bin/gh",
136: }
137: 
138: 
139: def flag_speedup(x):
140:     return x < 0.95
141: 
142: 
143: def flag_compilation_latency(x):
144:     return x > 120
145: 
146: 
147: def flag_compression_ratio(x):
148:     return x < 0.9
149: 
150: 
151: def flag_accuracy(x):
152:     return "pass" not in x
153: 
154: 
155: FLAG_FNS = {
156:     "speedup": flag_speedup,
157:     "compilation_latency": flag_compilation_latency,
158:     "compression_ratio": flag_compression_ratio,
159:     "accuracy": flag_accuracy,
160: }
````
- EN: Implements callable logic such as `flag_speedup`, `flag_compilation_latency`, `flag_compression_ratio`, `flag_accuracy`.
- CN: 实现可调用逻辑，例如 `flag_speedup`, `flag_compilation_latency`, `flag_compression_ratio`, `flag_accuracy`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 161-240
````python
161: 
162: 
163: def percentage(part, whole, decimals=2):
164:     if whole == 0:
165:         return 0
166:     return round(100 * float(part) / float(whole), decimals)
167: 
168: 
169: def parse_args():
170:     parser = argparse.ArgumentParser()
171:     parser.add_argument("--devices", action="append", help="cpu or cuda")
172:     parser.add_argument("--dtypes", action="append", help="float16/float32/amp")
173:     parser.add_argument("--suites", action="append", help="huggingface/torchbench/timm")
174:     parser.add_argument(
175:         "--compilers",
176:         action="append",
177:         help=f"For --inference, options are {INFERENCE_COMPILERS}. For --training, options are {TRAINING_COMPILERS}",
178:     )
179: 
180:     parser.add_argument(
181:         "--flag-compilers",
182:         action="append",
183:         help="List of compilers to flag issues. Same format as --compilers.",
184:     )
185:     parser.add_argument(
186:         "--quick", action="store_true", help="Just runs one model. Helps in debugging"
187:     )
188:     parser.add_argument(
189:         "--output-dir",
190:         help="Choose the output directory to save the logs",
191:         default=DEFAULT_OUTPUT_DIR,
192:     )
193:     parser.add_argument(
194:         "--keep-output-dir",
195:         action="store_true",
196:         help="Do not cleanup the output directory before running",
197:     )
198: 
199:     # Choose either generation of commands, pretty parsing or e2e runs
200:     group = parser.add_mutually_exclusive_group(required=False)
201:     group.add_argument(
202:         "--print-run-commands",
203:         "--print_run_commands",
204:         action="store_true",
205:         help="Generate commands and saves them to run.sh",
206:     )
207:     group.add_argument(
208:         "--visualize-logs",
209:         "--visualize_logs",
210:         action="store_true",
211:         help="Pretty print the log files and draw graphs",
212:     )
213:     group.add_argument(
214:         "--run",
215:         action="store_true",
216:         default=True,
217:         help="Generate commands, run and parses the files",
218:     )
219: 
220:     parser.add_argument(
221:         "--log-operator-inputs",
222:         action="store_true",
223:         default=False,
224:         help="Log operator inputs",
225:     )
226:     parser.add_argument(
227:         "--include-slowdowns",
228:         "--include_slowdowns",
229:         action="store_true",
230:         default=False,
231:         help="Include slowdowns in geomean performance speedup report. By default, slowdowns are ignored. "
232:         "This is because one can always use eager if compile is not speeding things up",
233:     )
234: 
235:     parser.add_argument(
236:         "--extra-args", default="", help="Append commandline with these args"
237:     )
238: 
239:     # Choose either inference or training
240:     group_mode = parser.add_mutually_exclusive_group(required=True)
````
- EN: Implements callable logic such as `percentage`, `parse_args`.
- CN: 实现可调用逻辑，例如 `percentage`, `parse_args`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-320
````python
241:     group_mode.add_argument(
242:         "--inference", action="store_true", help="Only run inference related tasks"
243:     )
244:     group_mode.add_argument(
245:         "--training", action="store_true", help="Only run training related tasks"
246:     )
247: 
248:     parser.add_argument(
249:         "--base-sha",
250:         help="commit id for the tested pytorch",
251:     )
252:     parser.add_argument(
253:         "--total-partitions",
254:         type=int,
255:         help="Total number of partitions, to be passed to the actual benchmark script",
256:     )
257:     parser.add_argument(
258:         "--partition-id",
259:         type=int,
260:         help="ID of partition, to be passed to the actual benchmark script",
261:     )
262: 
263:     parser.add_argument(
264:         "--update-dashboard",
265:         action="store_true",
266:         default=False,
267:         help="Updates to dashboard",
268:     )
269:     parser.add_argument(
270:         "--no-graphs",
271:         action="store_true",
272:         default=False,
273:         help="Do not generate and upload metric graphs",
274:     )
275:     parser.add_argument(
276:         "--no-update-archive",
277:         action="store_true",
278:         default=False,
279:         help="Do not update lookup.csv or the log archive",
280:     )
281:     parser.add_argument(
282:         "--no-gh-comment",
283:         action="store_true",
284:         default=False,
285:         help="Do not write a comment to github",
286:     )
287:     parser.add_argument(
288:         "--no-detect-regressions",
289:         action="store_true",
290:         default=False,
291:         help="Do not compare to previous runs for regressions or metric graphs.",
292:     )
293:     parser.add_argument(
294:         "--update-dashboard-test",
295:         action="store_true",
296:         default=False,
297:         help="does all of --no-graphs, --no-update-archive, and --no-gh-comment",
298:     )
299:     parser.add_argument(
300:         "--dashboard-image-uploader",
301:         default=DASHBOARD_DEFAULTS["dashboard_image_uploader"],
302:         help="Image uploader command",
303:     )
304:     parser.add_argument(
305:         "--dashboard-archive-path",
306:         default=DASHBOARD_DEFAULTS["dashboard_archive_path"],
307:         help="Archived directory path",
308:     )
309:     parser.add_argument(
310:         "--archive-name",
311:         help="Directory name under dashboard-archive-path to copy output-dir to. "
312:         "If not provided, a generated name is used.",
313:     )
314:     parser.add_argument(
315:         "--dashboard-gh-cli-path",
316:         default=DASHBOARD_DEFAULTS["dashboard_gh_cli_path"],
317:         help="Github CLI path",
318:     )
319:     parser.add_argument(
320:         "--batch-size",
````
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 321-400
````python
321:         "--batch_size",
322:         type=int,
323:         default=None,
324:         help="batch size for benchmarking",
325:     )
326:     parser.add_argument(
327:         "--threads",
328:         "-t",
329:         type=int,
330:         default=None,
331:         help="number of threads to use for eager and inductor.",
332:     )
333:     launcher_group = parser.add_argument_group("CPU Launcher Parameters")
334:     launcher_group.add_argument(
335:         "--enable-cpu-launcher",
336:         "--enable_cpu_launcher",
337:         action="store_true",
338:         default=False,
339:         help="Use torch.backends.xeon.run_cpu to get the peak performance on Intel(R) Xeon(R) Scalable Processors.",
340:     )
341:     launcher_group.add_argument(
342:         "--cpu-launcher-args",
343:         "--cpu_launcher_args",
344:         type=str,
345:         default="",
346:         help="Provide the args of torch.backends.xeon.run_cpu. "
347:         "To look up what optional arguments this launcher offers: python -m torch.backends.xeon.run_cpu --help",
348:     )
349:     parser.add_argument(
350:         "--no-cold-start-latency",
351:         action="store_true",
352:         default=False,
353:         help="Do not include --cold-start-latency on inductor benchmarks",
354:     )
355:     parser.add_argument(
356:         "--inductor-compile-mode",
357:         default=None,
358:         help="torch.compile mode argument for inductor runs.",
359:     )
360:     args = parser.parse_args()
361:     return args
362: 
363: 
364: def get_mode(args):
365:     if args.inference:
366:         return "inference"
367:     return "training"
368: 
369: 
370: def get_skip_tests(suite, device, is_training: bool):
371:     """
372:     Generate -x separated string to skip the unusual setup training tests
373:     """
374:     skip_tests = set()
375:     original_dir = abspath(os.getcwd())
376:     module = importlib.import_module(suite)
377:     os.chdir(original_dir)
378:     arch = platform.machine()
379: 
380:     if suite == "torchbench":
381:         skip_tests.update(module.TorchBenchmarkRunner().skip_models)
382:         if is_training:
383:             skip_tests.update(
384:                 module.TorchBenchmarkRunner().skip_not_suitable_for_training_models
385:             )
386:         if device == "cpu":
387:             skip_tests.update(module.TorchBenchmarkRunner().skip_models_for_cpu)
388:             if arch == "aarch64":
389:                 skip_tests.update(
390:                     module.TorchBenchmarkRunner().skip_models_for_cpu_aarch64
391:                 )
392:         elif device == "cuda":
393:             skip_tests.update(module.TorchBenchmarkRunner().skip_models_for_cuda)
394: 
395:     skip_tests = (f"-x {name}" for name in skip_tests)
396:     skip_str = " ".join(skip_tests)
397:     return skip_str
398: 
399: 
400: def generate_csv_name(args, dtype, suite, device, compiler, testing):
````
- EN: Implements callable logic such as `get_mode`, `get_skip_tests`, `generate_csv_name`.
- CN: 实现可调用逻辑，例如 `get_mode`, `get_skip_tests`, `generate_csv_name`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 401-480
````python
401:     mode = get_mode(args)
402:     return f"{compiler}_{suite}_{dtype}_{mode}_{device}_{testing}.csv"
403: 
404: 
405: def generate_commands(args, dtypes, suites, devices, compilers, output_dir):
406:     mode = get_mode(args)
407:     suites_str = "_".join(suites)
408:     devices_str = "_".join(devices)
409:     dtypes_str = "_".join(dtypes)
410:     compilers_str = "_".join(compilers)
411:     generated_file = (
412:         f"run_{mode}_{devices_str}_{dtypes_str}_{suites_str}_{compilers_str}.sh"
413:     )
414:     with open(generated_file, "w") as runfile:
415:         lines = []
416: 
417:         lines.append("#!/bin/bash")
418:         lines.append("set -x")
419:         lines.append("# Setup the output directory")
420:         if not args.keep_output_dir:
421:             lines.append(f"rm -rf {output_dir}")
422:         # It's ok if the output directory already exists
423:         lines.append(f"mkdir -p {output_dir}")
424:         lines.append("")
425: 
426:         for testing in ["performance", "accuracy"]:
427:             for iter in itertools.product(suites, devices, dtypes):
428:                 suite, device, dtype = iter
429:                 lines.append(
430:                     f"# Commands for {suite} for device={device}, dtype={dtype} for {mode} and for {testing} testing"
431:                 )
432:                 info = TABLE[mode]
433:                 for compiler in compilers:
434:                     base_cmd = info[compiler]
435:                     output_filename = f"{output_dir}/{generate_csv_name(args, dtype, suite, device, compiler, testing)}"
436:                     launcher_cmd = "python"
437:                     if args.enable_cpu_launcher:
438:                         launcher_cmd = f"python -m torch.backends.xeon.run_cpu {args.cpu_launcher_args}"
439:                     cmd = f"{launcher_cmd} benchmarks/dynamo/{suite}.py --{testing} --{dtype} -d{device} --output={output_filename}"
440:                     cmd = f"{cmd} {base_cmd} {args.extra_args} --dashboard"
441:                     skip_tests_str = get_skip_tests(suite, device, args.training)
442:                     cmd = f"{cmd} {skip_tests_str}"
443: 
444:                     if args.log_operator_inputs:
445:                         cmd = f"{cmd} --log-operator-inputs"
446: 
447:                     if args.quick:
448:                         filters = DEFAULTS["quick"][suite]
449:                         cmd = f"{cmd} {filters}"
450: 
451:                     if (
452:                         compiler
453:                         in (
454:                             "inductor",
455:                             "inductor_no_cudagraphs",
456:                         )
457:                         and not args.no_cold_start_latency
458:                     ):
459:                         cmd = f"{cmd} --cold-start-latency"
460: 
461:                     if args.batch_size is not None:
462:                         cmd = f"{cmd} --batch-size {args.batch_size}"
463: 
464:                     if args.threads is not None:
465:                         cmd = f"{cmd} --threads {args.threads}"
466: 
467:                     if args.total_partitions is not None:
468:                         cmd = f"{cmd} --total-partitions {args.total_partitions}"
469: 
470:                     if args.partition_id is not None:
471:                         cmd = f"{cmd} --partition-id {args.partition_id}"
472: 
473:                     if args.inductor_compile_mode is not None:
474:                         cmd = f"{cmd} --inductor-compile-mode {args.inductor_compile_mode}"
475:                     lines.append(cmd)
476:                 lines.append("")
477:         runfile.writelines([line + "\n" for line in lines])
478:     return generated_file
479: 
480: 
````
- EN: Implements callable logic such as `generate_commands`.
- CN: 实现可调用逻辑，例如 `generate_commands`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 481-560
````python
481: def generate_dropdown_comment(title, body):
482:     str_io = io.StringIO()
483:     str_io.write(f"{title}\n")
484:     str_io.write("<details>\n")
485:     str_io.write("<summary>see more</summary>\n")
486:     str_io.write(f"{body}")
487:     str_io.write("\n")
488:     str_io.write("</details>\n\n")
489:     return str_io.getvalue()
490: 
491: 
492: def build_summary(args):
493:     out_io = io.StringIO()
494: 
495:     def print_commit_hash(path, name):
496:         if args.base_sha is not None:
497:             if name == "pytorch":
498:                 out_io.write(f"{name} commit: {args.base_sha}\n")
499:         elif exists(path):
500:             import git
501: 
502:             repo = git.Repo(path, search_parent_directories=True)
503:             sha = repo.head.object.hexsha
504:             date = repo.head.object.committed_datetime
505:             out_io.write(f"{name} commit: {sha}\n")
506:             out_io.write(f"{name} commit date: {date}\n")
507:         else:
508:             out_io.write(f"{name} Absent\n")
509: 
510:     def env_var(name):
511:         if name in os.environ:
512:             out_io.write(f"{name} = {os.environ[name]}\n")
513:         else:
514:             out_io.write(f"{name} = {None}\n")
515: 
516:     out_io.write("\n")
517:     out_io.write("### Run name ###\n")
518:     out_io.write(get_archive_name(args, args.dtypes[0]))
519:     out_io.write("\n")
520: 
521:     out_io.write("\n")
522:     out_io.write("### Commit hashes ###\n")
523:     print_commit_hash("../pytorch", "pytorch")
524:     print_commit_hash("../torchbenchmark", "torchbench")
525: 
526:     out_io.write("\n")
527:     out_io.write("### TorchDynamo config flags ###\n")
528:     for key in dir(torch._dynamo.config):
529:         val = getattr(torch._dynamo.config, key)
530:         if not key.startswith("__") and isinstance(val, bool):
531:             out_io.write(f"torch._dynamo.config.{key} = {val}\n")
532: 
533:     out_io.write("\n")
534:     out_io.write("### Torch version ###\n")
535:     out_io.write(f"torch: {torch.__version__}\n")
536: 
537:     out_io.write("\n")
538:     out_io.write("### Environment variables ###\n")
539:     env_var("TORCH_CUDA_ARCH_LIST")
540:     env_var("CUDA_HOME")
541:     env_var("USE_LLVM")
542: 
543:     if "cuda" in args.devices:
544:         out_io.write("\n")
545:         out_io.write("### GPU details ###\n")
546:         out_io.write(f"CUDNN VERSION: {torch.backends.cudnn.version()}\n")
547:         out_io.write(f"Number CUDA Devices: {torch.cuda.device_count()}\n")
548:         out_io.write(f"Device Name: {torch.cuda.get_device_name(0)}\n")
549:         out_io.write(
550:             f"Device Memory [GB]: {torch.cuda.get_device_properties(0).total_memory / 1e9}\n"
551:         )
552: 
553:     title = "## Build Summary"
554:     comment = generate_dropdown_comment(title, out_io.getvalue())
555:     with open(f"{output_dir}/gh_build_summary.txt", "w") as gh_fh:
556:         gh_fh.write(comment)
557: 
558: 
559: @functools.cache
560: def archive_data(archive_name):
````
- EN: Handles module imports such as `git`.
- CN: 处理模块导入，例如 `git`。
- EN: Implements callable logic such as `generate_dropdown_comment`, `build_summary`, `print_commit_hash`, `env_var`.
- CN: 实现可调用逻辑，例如 `generate_dropdown_comment`, `build_summary`, `print_commit_hash`, `env_var`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 561-640
````python
561:     if archive_name is not None:
562:         prefix_match = re.search(r"\w+(?=_performance)", archive_name)
563:         if prefix_match is not None:
564:             prefix = prefix_match.group(0)
565:         else:
566:             prefix = ""
567:         day_match = re.search(r"day_(\d+)_", archive_name)
568:         if day_match is not None:
569:             day = day_match.group(1)
570:         else:
571:             day = "000"
572:     else:
573:         now = datetime.now(tz=timezone(timedelta(hours=-8)))
574:         day = now.strftime("%j")
575:         prefix = now.strftime(f"day_{day}_%d_%m_%y")
576:     return day, prefix
577: 
578: 
579: @functools.cache
580: def default_archive_name(dtype):
581:     _, prefix = archive_data(None)
582:     return f"{prefix}_performance_{dtype}_{randint(100, 999)}"
583: 
584: 
585: def get_archive_name(args, dtype):
586:     return (
587:         default_archive_name(dtype) if args.archive_name is None else args.archive_name
588:     )
589: 
590: 
591: def archive(src_dir, dest_dir_prefix, archive_name, dtype):
592:     if archive_name is None:
593:         archive_name = default_archive_name(dtype)
594:     # Copy the folder to archived location
595:     dest = os.path.join(dest_dir_prefix, archive_name)
596:     shutil.copytree(src_dir, dest, dirs_exist_ok=True)
597:     print(f"copied contents of {src_dir} to {dest}")
598: 
599: 
600: def get_metric_title(metric):
601:     if metric == "speedup":
602:         return "Performance speedup"
603:     elif metric == "accuracy":
604:         return "Accuracy"
605:     elif metric == "compilation_latency":
606:         return "Compilation latency (sec)"
607:     elif metric == "compression_ratio":
608:         return "Peak Memory Compression Ratio"
609:     elif metric == "abs_latency":
610:         return "Absolute latency (ms)"
611:     raise RuntimeError("unknown metric")
612: 
613: 
614: class Parser:
615:     def __init__(
616:         self, suites, devices, dtypes, compilers, flag_compilers, mode, output_dir
617:     ):
618:         self.suites = suites
619:         self.devices = devices
620:         self.dtypes = dtypes
621:         self.compilers = compilers
622:         self.flag_compilers = flag_compilers
623:         self.output_dir = output_dir
624:         self.mode = mode
625: 
626:     def has_header(self, output_filename):
627:         header_present = False
628:         with open(output_filename) as f:
629:             line = f.readline()
630:             if "dev" in line:
631:                 header_present = True
632:         return header_present
633: 
634: 
635: class ParsePerformanceLogs(Parser):
636:     def __init__(
637:         self,
638:         suites,
639:         devices,
640:         dtypes,
````
- EN: Declares or extends types including `Parser`, `ParsePerformanceLogs`.
- CN: 声明或扩展类型，包括 `Parser`, `ParsePerformanceLogs`。
- EN: Implements callable logic such as `default_archive_name`, `get_archive_name`, `archive`, `get_metric_title`.
- CN: 实现可调用逻辑，例如 `default_archive_name`, `get_archive_name`, `archive`, `get_metric_title`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 641-720
````python
641:         compilers,
642:         flag_compilers,
643:         mode,
644:         output_dir,
645:         include_slowdowns=False,
646:     ):
647:         super().__init__(
648:             suites,
649:             devices,
650:             dtypes,
651:             compilers,
652:             flag_compilers,
653:             mode,
654:             output_dir,
655:         )
656:         self.parsed_frames = defaultdict(lambda: defaultdict(None))
657:         self.untouched_parsed_frames = defaultdict(lambda: defaultdict(None))
658:         self.metrics = [
659:             "speedup",
660:             "abs_latency",
661:             "compilation_latency",
662:             "compression_ratio",
663:         ]
664:         self.bottom_k = 50
665:         self.parse()
666:         self.include_slowdowns = include_slowdowns
667: 
668:     def plot_graph(self, df, title):
669:         labels = df.columns.values.tolist()
670:         labels = labels[3:]
671:         df.plot(
672:             x="name",
673:             y=labels,
674:             kind="bar",
675:             width=0.65,
676:             title=title,
677:             ylabel="Speedup over eager",
678:             xlabel="",
679:             grid=True,
680:             figsize=(max(len(df.index) / 4, 5), 10),
681:             edgecolor="black",
682:         )
683:         plt.tight_layout()
684:         plt.savefig(f"{self.output_dir}/{title}.png")
685: 
686:     def read_csv(self, output_filename):
687:         if self.has_header(output_filename):
688:             return pd.read_csv(output_filename)
689:         else:
690:             return pd.read_csv(
691:                 output_filename,
692:                 names=[
693:                     "dev",
694:                     "name",
695:                     "batch_size",
696:                     "speedup",
697:                     "abs_latency",
698:                     "compilation_latency",
699:                     "compression_ratio",
700:                 ],
701:                 header=None,
702:                 engine="python",
703:             )
704: 
705:     def parse(self):
706:         self.extract_df("accuracy", "accuracy")
707:         for metric in self.metrics:
708:             self.extract_df(metric, "performance")
709: 
710:     def clean_batch_sizes(self, frames):
711:         # Clean up batch sizes when its 0
712:         if len(frames) == 1:
713:             return frames
714:         batch_sizes = frames[0]["batch_size"].to_list()
715:         for frame in frames[1:]:
716:             frame_batch_sizes = frame["batch_size"].to_list()
717:             for idx, (batch_a, batch_b) in enumerate(
718:                 zip(batch_sizes, frame_batch_sizes)
719:             ):
720:                 if not (batch_a == batch_b or batch_a == 0 or batch_b == 0):
````
- EN: Implements callable logic such as `plot_graph`, `read_csv`, `parse`, `clean_batch_sizes`.
- CN: 实现可调用逻辑，例如 `plot_graph`, `read_csv`, `parse`, `clean_batch_sizes`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 721-800
````python
721:                     raise AssertionError(
722:                         f"batch size mismatch: a={batch_a}, b={batch_b}"
723:                     )
724:                 batch_sizes[idx] = max(batch_a, batch_b)
725:         for frame in frames:
726:             frame["batch_size"] = batch_sizes
727:         return frames
728: 
729:     def extract_df(self, metric, testing):
730:         for iter in itertools.product(self.suites, self.devices, self.dtypes):
731:             suite, device, dtype = iter
732:             frames = []
733:             for compiler in self.compilers:
734:                 output_filename = f"{self.output_dir}/{compiler}_{suite}_{dtype}_{self.mode}_{device}_{testing}.csv"
735:                 df = self.read_csv(output_filename)
736:                 if metric not in df:
737:                     df.insert(len(df.columns), metric, np.nan)
738:                 df = df[["dev", "name", "batch_size", metric]]
739:                 df.rename(columns={metric: compiler}, inplace=True)
740:                 df["batch_size"] = df["batch_size"].astype(int)
741:                 frames.append(df)
742: 
743:             # Merge the results
744:             frames = self.clean_batch_sizes(frames)
745:             if len(self.compilers) == 1:
746:                 df = frames[0]
747:             else:
748:                 # Merge data frames
749:                 df = pd.merge(frames[0], frames[1], on=["dev", "name", "batch_size"])
750:                 for idx in range(2, len(frames)):
751:                     df = pd.merge(df, frames[idx], on=["dev", "name", "batch_size"])
752: 
753:             if testing == "performance":
754:                 for compiler in self.compilers:
755:                     df[compiler] = pd.to_numeric(df[compiler], errors="coerce").fillna(
756:                         0
757:                     )
758: 
759:             df_copy = df.copy()
760:             df_copy = df_copy.sort_values(
761:                 by=list(reversed(self.compilers)), ascending=False
762:             )
763:             if "inductor" in self.compilers:
764:                 df_copy = df_copy.sort_values(by="inductor", ascending=False)
765:             self.untouched_parsed_frames[suite][metric] = df_copy
766: 
767:             if testing == "performance":
768:                 df_accuracy = self.parsed_frames[suite]["accuracy"]
769:                 perf_rows = []
770:                 for model_name in df["name"]:
771:                     perf_row = df[df["name"] == model_name].copy()
772:                     acc_row = df_accuracy[df_accuracy["name"] == model_name]
773:                     for compiler in self.compilers:
774:                         if not perf_row.empty:
775:                             if acc_row.empty:
776:                                 perf_row[compiler] = 0.0
777:                             elif acc_row[compiler].iloc[0] in (
778:                                 "model_fail_to_load",
779:                                 "eager_fail_to_run",
780:                             ):
781:                                 perf_row = pd.DataFrame()
782:                             elif acc_row[compiler].iloc[0] not in (
783:                                 "pass",
784:                                 "pass_due_to_skip",
785:                             ):
786:                                 perf_row[compiler] = 0.0
787:                     if not perf_row.empty:
788:                         perf_rows.append(perf_row)
789:                 df = pd.concat(perf_rows)
790:             df = df.sort_values(by=list(reversed(self.compilers)), ascending=False)
791: 
792:             if "inductor" in self.compilers:
793:                 df = df.sort_values(by="inductor", ascending=False)
794:             self.parsed_frames[suite][metric] = df
795: 
796:     def get_passing_entries(self, compiler, df):
797:         return df[compiler][df[compiler] > 0]
798: 
799:     def comp_time(self, compiler, df):
800:         df = self.get_passing_entries(compiler, df)
````
- EN: Implements callable logic such as `extract_df`, `get_passing_entries`, `comp_time`.
- CN: 实现可调用逻辑，例如 `extract_df`, `get_passing_entries`, `comp_time`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 801-880
````python
801:         # df = df.sort_values(by=compiler, ascending=False)[compiler][: self.bottom_k]
802:         if df.empty:
803:             return "0.0"
804: 
805:         return f"{df.mean():.2f}"
806: 
807:     def geomean(self, compiler, df):
808:         cleaned_df = self.get_passing_entries(compiler, df)
809:         if not self.include_slowdowns:
810:             cleaned_df = cleaned_df.clip(1)
811:         if cleaned_df.empty:
812:             return "0.0x"
813:         return f"{gmean(cleaned_df):.2f}x"
814: 
815:     def passrate(self, compiler, df):
816:         total = len(df.index)
817:         passing = df[df[compiler] > 0.0][compiler].count()
818:         perc = int(percentage(passing, total, decimals=0))
819:         return f"{perc}%, {passing}/{total}"
820: 
821:     def memory(self, compiler, df):
822:         df = self.get_passing_entries(compiler, df)
823:         df = df.fillna(0)
824:         df = df[df > 0]
825:         if df.empty:
826:             return "0.0x"
827:         return f"{df.mean():.2f}x"
828: 
829:     def exec_summary_df(self, fn, metric):
830:         """
831:         Generate a table with passrate and geomean perf
832:         """
833:         cols = {}
834:         cols["Compiler"] = self.compilers
835:         for suite in self.suites:
836:             df = self.parsed_frames[suite][metric]
837:             # speedups = [self.geomean(compiler, df) for compiler in self.compilers]
838:             speedups = [fn(compiler, df) for compiler in self.compilers]
839:             col = pd.Series(data=speedups, index=self.compilers)
840:             cols[suite] = col
841:         df = pd.DataFrame(cols)
842:         df = df.fillna(0)
843:         df.to_csv(os.path.join(self.output_dir, f"{fn.__name__}.csv"))
844:         return df
845: 
846:     def exec_summary_text(self, caption, fn, metric):
847:         df = self.exec_summary_df(fn, metric)
848:         tabform = tabulate(df, headers="keys", tablefmt="pretty", showindex="never")
849: 
850:         str_io = io.StringIO()
851:         str_io.write(f"{caption}")
852:         str_io.write("~~~\n")
853:         str_io.write(f"{tabform}\n")
854:         str_io.write("~~~\n")
855:         return str_io.getvalue()
856: 
857:     def generate_executive_summary(self):
858:         machine = "A100 GPUs"
859:         if "cpu" in self.devices:
860:             get_machine_cmd = "lscpu| grep 'Model name' | awk -F':' '{print $2}'"
861:             machine = subprocess.getstatusoutput(get_machine_cmd)[1].strip()
862:         description = (
863:             "We evaluate different backends "
864:             "across three benchmark suites - torchbench, huggingface and timm. We run "
865:             "these experiments on "
866:             + machine
867:             + ". Each experiment runs one iteration of forward pass "
868:             "and backward pass for training and forward pass only for inference. "
869:             "For accuracy, we check the numerical correctness of forward pass outputs and gradients "
870:             "by comparing with native pytorch. We measure speedup "
871:             "by normalizing against the performance of native pytorch. We report mean "
872:             "compilation latency numbers and peak memory footprint reduction ratio. \n\n"
873:             "Caveats\n"
874:             "1) Batch size has been reduced to workaround OOM errors. Work is in progress to "
875:             "reduce peak memory footprint.\n"
876:             "2) Experiments do not cover dynamic shapes.\n"
877:             "3) Experimental setup does not have optimizer.\n\n"
878:         )
879:         comment = generate_dropdown_comment("", description)
880:         str_io = io.StringIO()
````
- EN: Implements callable logic such as `geomean`, `passrate`, `memory`, `exec_summary_df`.
- CN: 实现可调用逻辑，例如 `geomean`, `passrate`, `memory`, `exec_summary_df`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 881-960
````python
881:         str_io.write("\n")
882:         str_io.write("## Executive Summary ##\n")
883:         str_io.write(comment)
884: 
885:         speedup_caption = "Geometric mean speedup \n"
886:         speedup_summary = self.exec_summary_text(
887:             speedup_caption, self.geomean, "speedup"
888:         )
889: 
890:         passrate_caption = "Passrate\n"
891:         passrate_summary = self.exec_summary_text(
892:             passrate_caption, self.passrate, "speedup"
893:         )
894: 
895:         comp_time_caption = "Mean compilation time (seconds)\n"
896:         comp_time_summary = self.exec_summary_text(
897:             comp_time_caption, self.comp_time, "compilation_latency"
898:         )
899: 
900:         peak_memory_caption = (
901:             "Peak memory footprint compression ratio (higher is better)\n"
902:         )
903:         peak_memory_summary = self.exec_summary_text(
904:             peak_memory_caption, self.memory, "compression_ratio"
905:         )
906: 
907:         str_io.write(
908:             "To measure performance, compilation latency and memory footprint reduction, "
909:             "we remove the models that fail accuracy checks.\n\n"
910:         )
911:         str_io.write(passrate_summary)
912:         str_io.write(speedup_summary)
913:         str_io.write(comp_time_summary)
914:         str_io.write(peak_memory_summary)
915:         self.executive_summary = str_io.getvalue()
916: 
917:     def flag_bad_entries(self, suite, metric, flag_fn):
918:         df = self.untouched_parsed_frames[suite][metric]
919:         df = df.drop("dev", axis=1)
920:         df = df.rename(columns={"batch_size": "bs"})
921:         # apply flag_fn elementwise to flag_compilers columns,
922:         # if one element fails, the entire row is flagged
923:         flag = np.logical_or.reduce(
924:             df[self.flag_compilers].applymap(flag_fn),
925:             axis=1,
926:         )
927:         df = df[flag]
928:         df = df.assign(suite=suite)
929:         return df.reindex(columns=["suite", "name"] + self.flag_compilers)
930: 
931:     def generate_warnings(self):
932:         title = "## Warnings ##"
933:         body = (
934:             "We flag models where:\n\n"
935:             " - accuracy fails\n"
936:             " - speedup < 0.95x (NOTE: 0.0 speedup typically signifies a failure in the performance test)\n"
937:             " - compilation latency > 120 sec.\n"
938:             " - compression ratio < 0.9\n"
939:             "\n"
940:         )
941:         for metric in [
942:             "accuracy",
943:             "speedup",
944:             "compilation_latency",
945:             "compression_ratio",
946:         ]:
947:             dfs = []
948:             for suite in self.suites:
949:                 dfs.append(self.flag_bad_entries(suite, metric, FLAG_FNS[metric]))
950:             df = pd.concat(dfs, axis=0)
951:             if df.empty:
952:                 continue
953:             tabform = tabulate(df, headers="keys", tablefmt="pretty", showindex="never")
954:             str_io = io.StringIO()
955:             str_io.write("\n")
956:             str_io.write(get_metric_title(metric) + " warnings\n")
957:             str_io.write("~~~\n")
958:             str_io.write(f"{tabform}\n")
959:             str_io.write("~~~\n")
960:             body += str_io.getvalue()
````
- EN: Implements callable logic such as `flag_bad_entries`, `generate_warnings`.
- CN: 实现可调用逻辑，例如 `flag_bad_entries`, `generate_warnings`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 961-1040
````python
 961: 
 962:         comment = generate_dropdown_comment(title, body)
 963:         return comment
 964: 
 965:     def prepare_message(self, suite):
 966:         title = f"## {suite} suite with {self.dtypes[0]} precision ##"
 967:         body = ""
 968:         for metric in [
 969:             "speedup",
 970:             "accuracy",
 971:             "compilation_latency",
 972:             "compression_ratio",
 973:             "abs_latency",
 974:         ]:
 975:             df = self.untouched_parsed_frames[suite][metric]
 976:             df = df.drop("dev", axis=1)
 977:             df = df.rename(columns={"batch_size": "bs"})
 978:             tabform = tabulate(df, headers="keys", tablefmt="pretty", showindex="never")
 979:             str_io = io.StringIO()
 980:             str_io.write("\n")
 981:             str_io.write(get_metric_title(metric) + "\n")
 982:             str_io.write("~~~\n")
 983:             str_io.write(f"{tabform}\n")
 984:             str_io.write("~~~\n")
 985:             body += str_io.getvalue()
 986: 
 987:         comment = generate_dropdown_comment(title, body)
 988:         return comment
 989: 
 990:     def gen_summary_files(self):
 991:         self.generate_executive_summary()
 992:         for suite in self.suites:
 993:             self.plot_graph(
 994:                 self.untouched_parsed_frames[suite]["speedup"],
 995:                 f"{suite}_{self.dtypes[0]}",
 996:             )
 997: 
 998:         with open(f"{self.output_dir}/gh_title.txt", "w") as gh_fh:
 999:             str_io = io.StringIO()
1000:             str_io.write("\n")
1001:             str_io.write(f"# Performance Dashboard for {self.dtypes[0]} precision ##\n")
1002:             str_io.write("\n")
1003:             gh_fh.write(str_io.getvalue())
1004: 
1005:         with open(f"{self.output_dir}/gh_executive_summary.txt", "w") as gh_fh:
1006:             gh_fh.write(self.executive_summary)
1007: 
1008:         with open(f"{self.output_dir}/gh_warnings.txt", "w") as gh_fh:
1009:             warnings_body = self.generate_warnings()
1010:             gh_fh.write(warnings_body)
1011: 
1012:         str_io = io.StringIO()
1013:         for suite in self.suites:
1014:             str_io.write(self.prepare_message(suite))
1015:         str_io.write("\n")
1016:         with open(f"{self.output_dir}/gh_{self.mode}.txt", "w") as gh_fh:
1017:             gh_fh.write(str_io.getvalue())
1018: 
1019: 
1020: def parse_logs(args, dtypes, suites, devices, compilers, flag_compilers, output_dir):
1021:     mode = get_mode(args)
1022:     build_summary(args)
1023:     include_slowdowns = args.include_slowdowns
1024: 
1025:     parser_class = ParsePerformanceLogs
1026:     parser = parser_class(
1027:         suites,
1028:         devices,
1029:         dtypes,
1030:         compilers,
1031:         flag_compilers,
1032:         mode,
1033:         output_dir,
1034:         include_slowdowns,
1035:     )
1036:     parser.gen_summary_files()
1037:     return
1038: 
1039: 
1040: @dataclasses.dataclass
````
- EN: Implements callable logic such as `prepare_message`, `gen_summary_files`, `parse_logs`.
- CN: 实现可调用逻辑，例如 `prepare_message`, `gen_summary_files`, `parse_logs`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 1041-1120
````python
1041: class LogInfo:
1042:     # Day of the year this log was generated
1043:     day: str
1044: 
1045:     # Directory path where all logs are present
1046:     dir_path: str
1047: 
1048: 
1049: def get_date(log_info):
1050:     return datetime.strptime(f"{log_info.day}", "%j").strftime("%m-%d")
1051: 
1052: 
1053: def find_last_2_with_filenames(lookup_file, dashboard_archive_path, dtype, filenames):
1054:     df = pd.read_csv(lookup_file, names=("day", "mode", "prec", "path"))
1055:     df = df[df["mode"] == "performance"]
1056:     df = df[df["prec"] == dtype]
1057:     df = df[::-1]
1058:     last2 = []
1059:     for path in df["path"]:
1060:         output_dir = os.path.join(dashboard_archive_path, path)
1061:         fullpaths = [
1062:             os.path.join(dashboard_archive_path, path, name) for name in filenames
1063:         ]
1064:         if all(os.path.exists(fullpath) for fullpath in fullpaths):
1065:             last2.append(output_dir)
1066:         if len(last2) >= 2:
1067:             return last2
1068:     return None
1069: 
1070: 
1071: class SummaryStatDiffer:
1072:     def __init__(self, args):
1073:         self.args = args
1074:         self.lookup_file = os.path.join(self.args.dashboard_archive_path, "lookup.csv")
1075:         if not os.path.exists(self.lookup_file):
1076:             raise AssertionError(f"lookup file not found: {self.lookup_file}")
1077: 
1078:     def generate_diff(self, last2, filename, caption):
1079:         df_cur, df_prev = (pd.read_csv(os.path.join(path, filename)) for path in last2)
1080:         df_merge = df_cur.merge(df_prev, on="Compiler", suffixes=("_cur", "_prev"))
1081:         data = {col: [] for col in ("compiler", "suite", "prev_value", "cur_value")}
1082:         for _, row in df_merge.iterrows():
1083:             if row["Compiler"] in self.args.flag_compilers:
1084:                 for suite in self.args.suites:
1085:                     if suite + "_prev" not in row or suite + "_cur" not in row:
1086:                         continue
1087:                     data["compiler"].append(row["Compiler"])
1088:                     data["suite"].append(suite)
1089:                     data["prev_value"].append(row[suite + "_prev"])
1090:                     data["cur_value"].append(row[suite + "_cur"])
1091: 
1092:         df = pd.DataFrame(data)
1093:         tabform = tabulate(df, headers="keys", tablefmt="pretty", showindex="never")
1094:         str_io = io.StringIO()
1095:         str_io.write("\n")
1096:         str_io.write(f"{caption}\n")
1097:         str_io.write("~~~\n")
1098:         str_io.write(f"{tabform}\n")
1099:         str_io.write("~~~\n")
1100:         return str_io.getvalue()
1101: 
1102:     def generate_comment(self):
1103:         title = "## Summary Statistics Diff ##\n"
1104:         body = (
1105:             "For each relevant compiler, we compare the summary statistics "
1106:             "for the most 2 recent reports that actually run the compiler.\n\n"
1107:         )
1108:         dtype = self.args.dtypes[0]
1109:         last2 = find_last_2_with_filenames(
1110:             self.lookup_file,
1111:             self.args.dashboard_archive_path,
1112:             dtype,
1113:             ["geomean.csv", "passrate.csv"],
1114:         )
1115: 
1116:         if last2 is None:
1117:             body += "Could not find most 2 recent reports.\n\n"
1118:         else:
1119:             for state, path in zip(("Current", "Previous"), last2):
1120:                 body += f"{state} report name: {path}\n\n"
````
- EN: Declares or extends types including `LogInfo`, `SummaryStatDiffer`.
- CN: 声明或扩展类型，包括 `LogInfo`, `SummaryStatDiffer`。
- EN: Implements callable logic such as `get_date`, `find_last_2_with_filenames`, `__init__`, `generate_diff`.
- CN: 实现可调用逻辑，例如 `get_date`, `find_last_2_with_filenames`, `__init__`, `generate_diff`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 1121-1200
````python
1121:             body += self.generate_diff(last2, "passrate.csv", "Passrate diff")
1122:             body += self.generate_diff(
1123:                 last2, "geomean.csv", "Geometric mean speedup diff"
1124:             )
1125: 
1126:         comment = generate_dropdown_comment(title, body)
1127: 
1128:         with open(f"{self.args.output_dir}/gh_summary_diff.txt", "w") as gh_fh:
1129:             gh_fh.write(comment)
1130: 
1131: 
1132: class RegressionDetector:
1133:     """
1134:     Compares the most recent 2 benchmarks to find previously unflagged models
1135:     that are now flagged.
1136:     """
1137: 
1138:     def __init__(self, args):
1139:         self.args = args
1140:         self.lookup_file = os.path.join(self.args.dashboard_archive_path, "lookup.csv")
1141:         if not os.path.exists(self.lookup_file):
1142:             raise AssertionError(f"lookup file not found: {self.lookup_file}")
1143: 
1144:     def generate_comment(self):
1145:         title = "## Recent Regressions ##\n"
1146:         body = (
1147:             "For each relevant compiler, we compare the most recent 2 reports "
1148:             "(that actually run the compiler) to find previously unflagged "
1149:             "models that are now flagged as problematic (according to the "
1150:             "'Warnings' section).\n\n"
1151:         )
1152:         dtype = self.args.dtypes[0]
1153:         device = self.args.devices[0]
1154:         for suite in self.args.suites:
1155:             body += f"### Regressions for {suite} ###\n"
1156:             last2 = {}
1157: 
1158:             for compiler in self.args.flag_compilers:
1159:                 filenames = [
1160:                     generate_csv_name(
1161:                         self.args, dtype, suite, device, compiler, testing
1162:                     )
1163:                     for testing in ["performance", "accuracy"]
1164:                 ]
1165:                 compiler_last2 = find_last_2_with_filenames(
1166:                     self.lookup_file, self.args.dashboard_archive_path, dtype, filenames
1167:                 )
1168:                 if compiler_last2 is not None:
1169:                     last2[compiler] = [
1170:                         ParsePerformanceLogs(
1171:                             [suite],
1172:                             [device],
1173:                             [dtype],
1174:                             [compiler],
1175:                             [compiler],
1176:                             get_mode(self.args),
1177:                             output_dir,
1178:                         )
1179:                         for output_dir in compiler_last2
1180:                     ]
1181:                     for state, path in zip(("Current", "Previous"), compiler_last2):
1182:                         body += (
1183:                             f"{state} report name (compiler: {compiler}, "
1184:                             f"suite: {suite}): {path}\n\n"
1185:                         )
1186: 
1187:             regressions_present = False
1188:             for metric in [
1189:                 "accuracy",
1190:                 "speedup",
1191:                 "compilation_latency",
1192:                 "compression_ratio",
1193:             ]:
1194:                 dfs = []
1195:                 for compiler in self.args.flag_compilers:
1196:                     if last2[compiler] is None:
1197:                         continue
1198: 
1199:                     df_cur, df_prev = (
1200:                         last2[compiler][i].untouched_parsed_frames[suite][metric]
````
- EN: Declares or extends types including `RegressionDetector`.
- CN: 声明或扩展类型，包括 `RegressionDetector`。
- EN: Implements callable logic such as `__init__`, `generate_comment`.
- CN: 实现可调用逻辑，例如 `__init__`, `generate_comment`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 1201-1280
````python
1201:                         for i in (0, 1)
1202:                     )
1203:                     df_merge = df_cur.merge(
1204:                         df_prev, on="name", suffixes=("_cur", "_prev")
1205:                     )
1206:                     flag_fn = FLAG_FNS[metric]
1207:                     flag = np.logical_and(
1208:                         df_merge[compiler + "_prev"].apply(
1209:                             lambda x: not pd.isna(x) and not flag_fn(x)
1210:                         ),
1211:                         df_merge[compiler + "_cur"].apply(
1212:                             lambda x: not pd.isna(x) and flag_fn(x)
1213:                         ),
1214:                     )
1215:                     df_bad = df_merge[flag]
1216:                     dfs.append(
1217:                         pd.DataFrame(
1218:                             data={
1219:                                 "compiler": compiler,
1220:                                 "name": df_bad["name"],
1221:                                 "prev_status": df_bad[compiler + "_prev"],
1222:                                 "cur_status": df_bad[compiler + "_cur"],
1223:                             }
1224:                         )
1225:                     )
1226: 
1227:                 if not dfs:
1228:                     continue
1229:                 df = pd.concat(dfs, axis=0)
1230:                 if df.empty:
1231:                     continue
1232:                 regressions_present = True
1233:                 tabform = tabulate(
1234:                     df, headers="keys", tablefmt="pretty", showindex="never"
1235:                 )
1236:                 str_io = io.StringIO()
1237:                 str_io.write("\n")
1238:                 str_io.write(f"{get_metric_title(metric)} regressions\n")
1239:                 str_io.write("~~~\n")
1240:                 str_io.write(f"{tabform}\n")
1241:                 str_io.write("~~~\n")
1242:                 body += str_io.getvalue()
1243: 
1244:             if not regressions_present:
1245:                 body += "No regressions found.\n"
1246: 
1247:         comment = generate_dropdown_comment(title, body)
1248: 
1249:         with open(f"{self.args.output_dir}/gh_metric_regression.txt", "w") as gh_fh:
1250:             gh_fh.write(comment)
1251: 
1252: 
1253: class RegressionTracker:
1254:     """
1255:     Plots progress of different metrics over time to detect regressions.
1256:     """
1257: 
1258:     def __init__(self, args):
1259:         self.args = args
1260:         self.suites = self.args.suites
1261:         self.lookup_file = os.path.join(self.args.dashboard_archive_path, "lookup.csv")
1262:         if not os.path.exists(self.lookup_file):
1263:             raise AssertionError(f"lookup file not found: {self.lookup_file}")
1264:         self.k = 10
1265: 
1266:     def find_last_k(self):
1267:         """
1268:         Find the last k pairs of (day number, log_path)
1269:         """
1270:         dtype = self.args.dtypes[0]
1271:         df = pd.read_csv(self.lookup_file, names=("day", "mode", "prec", "path"))
1272:         df = df[df["mode"] == "performance"]
1273:         df = df[df["prec"] == dtype]
1274:         log_infos = []
1275:         for day, path in zip(df["day"], df["path"]):
1276:             log_infos.append(LogInfo(day, path))
1277: 
1278:         if len(log_infos) < self.k:
1279:             raise AssertionError(
1280:                 f"expected at least {self.k} log entries, got {len(log_infos)}"
````
- EN: Declares or extends types including `RegressionTracker`.
- CN: 声明或扩展类型，包括 `RegressionTracker`。
- EN: Implements callable logic such as `__init__`, `find_last_k`.
- CN: 实现可调用逻辑，例如 `__init__`, `find_last_k`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 1281-1360
````python
1281:             )
1282:         log_infos = log_infos[len(log_infos) - self.k :]
1283:         return log_infos
1284: 
1285:     def generate_comment(self):
1286:         title = "## Metrics over time ##\n"
1287:         str_io = io.StringIO()
1288:         if not self.args.update_dashboard_test and not self.args.no_graphs:
1289:             for name in glob.glob(self.args.output_dir + "/*over_time.png"):
1290:                 output = (
1291:                     subprocess.check_output([self.args.dashboard_image_uploader, name])
1292:                     .decode("ascii")
1293:                     .rstrip()
1294:                 )
1295:                 str_io.write(f"\n{name} : ![]({output})\n")
1296:         comment = generate_dropdown_comment(title, str_io.getvalue())
1297: 
1298:         with open(f"{self.args.output_dir}/gh_regression.txt", "w") as gh_fh:
1299:             gh_fh.write(comment)
1300: 
1301:     def diff(self):
1302:         log_infos = self.find_last_k()
1303: 
1304:         for metric in ["geomean", "passrate", "comp_time", "memory"]:
1305:             fig, axes = plt.subplots(nrows=1, ncols=3, figsize=(15, 5))
1306:             for idx, suite in enumerate(self.suites):
1307:                 dfs = []
1308:                 for log_info in log_infos:
1309:                     dir_path = os.path.join(
1310:                         self.args.dashboard_archive_path, log_info.dir_path
1311:                     )
1312:                     if not os.path.exists(dir_path):
1313:                         raise AssertionError(f"directory not found: {dir_path}")
1314:                     gmean_filename = os.path.join(dir_path, f"{metric}.csv")
1315:                     if not os.path.exists(gmean_filename):
1316:                         continue
1317:                     df = pd.read_csv(gmean_filename)
1318:                     if suite not in df:
1319:                         continue
1320:                     if metric == "geomean" or metric == "memory":
1321:                         df[suite] = df[suite].str.replace("x", "").astype(float)
1322:                     elif metric == "passrate":
1323:                         df[suite] = df[suite].str.split("%").str[0].astype(float)
1324:                     df.insert(0, "day", get_date(log_info))
1325:                     df = df.pivot(index="day", columns="Compiler", values=suite)
1326: 
1327:                     # Interim stage when both inductor_cudagraphs and inductor exist
1328:                     df = df.rename(columns={"inductor_cudagraphs": "inductor"})
1329:                     for col_name in df.columns:
1330:                         if col_name not in self.args.compilers:
1331:                             df = df.drop(columns=[col_name])
1332:                     dfs.append(df)
1333: 
1334:                 df = pd.concat(dfs)
1335:                 df = df.interpolate(method="linear")
1336:                 ax = df.plot(
1337:                     ax=axes[idx],
1338:                     kind="line",
1339:                     ylabel=metric,
1340:                     xlabel="Date",
1341:                     grid=True,
1342:                     ylim=0 if metric == "passrate" else 0.8,
1343:                     title=suite,
1344:                     style=".-",
1345:                     legend=False,
1346:                 )
1347:                 ax.legend(loc="lower right", ncol=2)
1348: 
1349:             plt.tight_layout()
1350:             plt.savefig(os.path.join(output_dir, f"{metric}_over_time.png"))
1351: 
1352:         self.generate_comment()
1353: 
1354: 
1355: class DashboardUpdater:
1356:     """
1357:     Aggregates the information and makes a comment to Performance Dashboard.
1358:     https://github.com/pytorch/torchdynamo/issues/681
1359:     """
1360: 
````
- EN: Declares or extends types including `DashboardUpdater`.
- CN: 声明或扩展类型，包括 `DashboardUpdater`。
- EN: Implements callable logic such as `generate_comment`, `diff`.
- CN: 实现可调用逻辑，例如 `generate_comment`, `diff`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 1361-1440
````python
1361:     def __init__(self, args):
1362:         self.args = args
1363:         self.output_dir = args.output_dir
1364:         self.lookup_file = os.path.join(self.args.dashboard_archive_path, "lookup.csv")
1365:         if not os.path.exists(self.lookup_file):
1366:             raise AssertionError(f"lookup file not found: {self.lookup_file}")
1367:         try:
1368:             if not self.args.update_dashboard_test and not self.args.no_update_archive:
1369:                 self.update_lookup_file()
1370:         except subprocess.CalledProcessError:
1371:             sys.stderr.write("failed to update lookup file\n")
1372: 
1373:     def update_lookup_file(self):
1374:         dtype = self.args.dtypes[0]
1375:         day, _ = archive_data(self.args.archive_name)
1376:         target_dir = get_archive_name(self.args, dtype)
1377:         # Update lookup csv the folder to archived logs
1378:         subprocess.check_call(
1379:             f'echo "{day},performance,{dtype},{target_dir}" >> {self.lookup_file}',
1380:             shell=True,
1381:         )
1382: 
1383:     def archive(self):
1384:         dtype = self.args.dtypes[0]
1385:         # Copy the folder to archived location
1386:         archive(
1387:             self.output_dir,
1388:             self.args.dashboard_archive_path,
1389:             self.args.archive_name,
1390:             dtype,
1391:         )
1392: 
1393:     def upload_graphs(self):
1394:         title = "## Performance graphs ##\n"
1395:         str_io = io.StringIO()
1396:         if not self.args.update_dashboard_test and not self.args.no_graphs:
1397:             for name in glob.glob(self.output_dir + "/*png"):
1398:                 if "over_time" not in name:
1399:                     output = (
1400:                         subprocess.check_output(
1401:                             [self.args.dashboard_image_uploader, name]
1402:                         )
1403:                         .decode("ascii")
1404:                         .rstrip()
1405:                     )
1406:                     str_io.write(f"\n{name} : ![]({output})\n")
1407:         comment = generate_dropdown_comment(title, str_io.getvalue())
1408: 
1409:         with open(f"{self.output_dir}/gh_graphs.txt", "w") as gh_fh:
1410:             gh_fh.write(comment)
1411: 
1412:     def gen_comment(self):
1413:         files = [
1414:             "gh_title.txt",
1415:             "gh_executive_summary.txt",
1416:             "gh_summary_diff.txt",
1417:             "gh_warnings.txt",
1418:             "gh_regression.txt",
1419:             "gh_metric_regression.txt",
1420:             "gh_training.txt" if self.args.training else "gh_inference.txt",
1421:             "gh_graphs.txt",
1422:             "gh_build_summary.txt",
1423:         ]
1424:         all_lines = []
1425:         for f in files:
1426:             try:
1427:                 with open(os.path.join(self.output_dir, f)) as fh:
1428:                     all_lines.extend(fh.readlines())
1429:             except FileNotFoundError:
1430:                 pass
1431: 
1432:         return "\n".join([x.rstrip() for x in all_lines])
1433: 
1434:     def comment_on_gh(self, comment):
1435:         """
1436:         Send a comment to dashboard
1437:         """
1438:         with tempfile.NamedTemporaryFile(mode="w", delete=False) as f:
1439:             f.write(comment)
1440:             filename = f.name
````
- EN: Implements callable logic such as `__init__`, `update_lookup_file`, `archive`, `upload_graphs`.
- CN: 实现可调用逻辑，例如 `__init__`, `update_lookup_file`, `archive`, `upload_graphs`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 1441-1520
````python
1441: 
1442:         issue_number = "93794"
1443:         if self.args.dtypes[0] == "float32":
1444:             issue_number = "93518"
1445: 
1446:         subprocess.check_call(
1447:             [
1448:                 self.args.dashboard_gh_cli_path,
1449:                 "issue",
1450:                 "comment",
1451:                 "--repo=https://github.com/pytorch/pytorch.git",
1452:                 issue_number,
1453:                 "-F",
1454:                 filename,
1455:             ]
1456:         )
1457: 
1458:         os.remove(filename)
1459: 
1460:     def update(self):
1461:         self.upload_graphs()
1462:         if not self.args.no_detect_regressions:
1463:             SummaryStatDiffer(self.args).generate_comment()
1464:             RegressionDetector(self.args).generate_comment()
1465:             try:
1466:                 RegressionTracker(self.args).diff()
1467:             except Exception:
1468:                 log.exception("")
1469:                 with open(f"{self.args.output_dir}/gh_regression.txt", "w") as gh_fh:
1470:                     gh_fh.write("")
1471: 
1472:         comment = self.gen_comment()
1473:         print(comment)
1474: 
1475:         if not self.args.update_dashboard_test:
1476:             if not self.args.no_gh_comment:
1477:                 self.comment_on_gh(comment)
1478:             if not self.args.no_update_archive:
1479:                 self.archive()
1480: 
1481: 
1482: if __name__ == "__main__":
1483:     args = parse_args()
1484: 
1485:     def extract(key):
1486:         return DEFAULTS[key] if getattr(args, key, None) is None else getattr(args, key)
1487: 
1488:     dtypes = extract("dtypes")
1489:     suites = extract("suites")
1490:     devices = extract("devices")
1491: 
1492:     if args.inference:
1493:         compilers = DEFAULTS["inference"] if args.compilers is None else args.compilers
1494:         flag_compilers = (
1495:             DEFAULTS["flag_compilers"]["inference"]
1496:             if args.flag_compilers is None
1497:             else args.flag_compilers
1498:         )
1499:     else:
1500:         if not args.training:
1501:             raise AssertionError("expected args.training to be True")
1502:         compilers = DEFAULTS["training"] if args.compilers is None else args.compilers
1503:         flag_compilers = (
1504:             DEFAULTS["flag_compilers"]["training"]
1505:             if args.flag_compilers is None
1506:             else args.flag_compilers
1507:         )
1508: 
1509:     output_dir = args.output_dir
1510:     args.compilers = compilers
1511:     args.devices = devices
1512:     args.dtypes = dtypes
1513:     flag_compilers = list(set(flag_compilers) & set(compilers))
1514:     args.flag_compilers = flag_compilers
1515:     args.suites = suites
1516: 
1517:     if args.print_run_commands:
1518:         generated_file = generate_commands(
1519:             args, dtypes, suites, devices, compilers, output_dir
1520:         )
````
- EN: Implements callable logic such as `update`, `extract`.
- CN: 实现可调用逻辑，例如 `update`, `extract`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 1521-1561
````python
1521:         print(
1522:             f"Running commands are generated in file {generated_file}. Please run (bash {generated_file})."
1523:         )
1524:     elif args.visualize_logs:
1525:         parse_logs(args, dtypes, suites, devices, compilers, flag_compilers, output_dir)
1526:     elif args.run:
1527:         generated_file = generate_commands(
1528:             args, dtypes, suites, devices, compilers, output_dir
1529:         )
1530:         # generate memoized archive name now so that the date is reflective
1531:         # of when the run started
1532:         get_archive_name(args, dtypes[0])
1533:         # TODO - Do we need to worry about segfaults
1534:         try:
1535:             os.system(f"bash {generated_file}")
1536:         except Exception as e:
1537:             print(
1538:                 f"Running commands failed. Please run manually (bash {generated_file}) and inspect the errors."
1539:             )
1540:             raise e
1541:         if not args.log_operator_inputs:
1542:             if not args.no_update_archive:
1543:                 archive(
1544:                     output_dir,
1545:                     args.dashboard_archive_path,
1546:                     args.archive_name,
1547:                     dtypes[0],
1548:                 )
1549:             parse_logs(
1550:                 args, dtypes, suites, devices, compilers, flag_compilers, output_dir
1551:             )
1552:             if not args.no_update_archive:
1553:                 archive(
1554:                     output_dir,
1555:                     args.dashboard_archive_path,
1556:                     args.archive_name,
1557:                     dtypes[0],
1558:                 )
1559: 
1560:     if args.update_dashboard:
1561:         DashboardUpdater(args).update()
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `flag_speedup` / 符号 `flag_speedup`
- Symbol `flag_compilation_latency` / 符号 `flag_compilation_latency`
- Symbol `flag_compression_ratio` / 符号 `flag_compression_ratio`
- Symbol `flag_accuracy` / 符号 `flag_accuracy`

## Dependencies / 依赖关系
- Python imports: `argparse`, `dataclasses`, `functools`, `glob`, `importlib`, `io`, `itertools`, `logging`, `os`, `platform`
- Python 导入: `argparse`, `dataclasses`, `functools`, `glob`, `importlib`, `io`, `itertools`, `logging`, `os`, `platform`
