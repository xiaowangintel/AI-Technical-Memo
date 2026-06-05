# launch_compile_op_numerics.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/dynamo/launch_compile_op_numerics.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-80
````python
 1: import argparse
 2: import asyncio
 3: import datetime
 4: import functools
 5: import hashlib
 6: import itertools
 7: import json
 8: import logging
 9: import os
10: import re
11: import shlex
12: import sys
13: from dataclasses import dataclass, field
14: from pathlib import Path
15: 
16: 
17: log = logging.getLogger(__name__)
18: 
19: try:
20:     import torch
21: except ImportError:
22:     torch = None
23: 
24: PYTORCH_NIGHTLY_CUDA_VERSIONS = ["12.6", "12.8", "13.0"]
25: PYTORCH_CUDA_VERSIONS = {
26:     "2.9.1": ["12.6", "12.8", "13.0"],
27:     "2.9.0": ["12.6", "12.8", "13.0"],
28:     "2.8.0": ["12.6", "12.8", "12.9"],
29:     "2.7.1": ["11.8", "12.6", "12.8"],
30:     "2.7.0": ["11.8", "12.6", "12.8"],
31:     "2.6.0": ["11.8", "12.4", "12.6"],
32:     "2.5.1": ["11.8", "12.1", "12.4"],
33:     "2.5.0": ["11.8", "12.1", "12.4"],
34:     "2.4.1": ["11.8", "12.1", "12.4"],
35:     "2.4.0": ["11.8", "12.1", "12.4"],
36:     "2.3.1": ["11.8", "12.1"],
37:     "2.3.0": ["11.8", "12.1"],
38:     "2.2.2": ["11.8", "12.1"],
39:     "2.2.1": ["11.8", "12.1"],
40:     "2.2.0": ["11.8", "12.1"],
41:     "2.1.2": ["11.8", "12.1"],
42:     "2.1.1": ["11.8", "12.1"],
43:     "2.1.0": ["11.8", "12.1"],
44:     "2.0.1": ["11.8"],
45:     "2.0.0": ["11.8"],
46: }
47: 
48: ENABLED_CONFIGS = [
49:     #    ("git:5f09e6a6c93e0b5bf75b635cddc03b85bbe85938", "12.8"),
50:     ("nightly", "13.0"),
51:     # ("2.9.1", "12.8"),
52:     # ("2.7.1", "12.8"),
53: ]
54: 
55: PYTHON_VERSION = "3.11"
56: 
57: 
58: @dataclass
59: class Mode:
60:     compile: bool
61:     backend: str | None = None
62:     mode: str | None = None
63:     options: dict | None = None
64:     env: dict = field(default_factory=dict)
65: 
66:     def __call__(self, fn):
67:         if self.compile:
68:             kwargs = {}
69:             if self.backend is not None:
70:                 kwargs["backend"] = self.backend
71:             if self.mode is not None:
72:                 kwargs["mode"] = self.mode
73:             if self.options is not None:
74:                 options = {
75:                     k: v
76:                     for k, v in self.options.items()
77:                     if k in torch._inductor.list_options()
78:                 }
79:                 kwargs["options"] = options
80:             return torch.compile(fn, **kwargs)
````
- EN: Handles module imports such as `argparse`, `asyncio`, `datetime`, `functools`.
- CN: 处理模块导入，例如 `argparse`, `asyncio`, `datetime`, `functools`。
- EN: Declares or extends types including `Mode`.
- CN: 声明或扩展类型，包括 `Mode`。
- EN: Implements callable logic such as `__call__`.
- CN: 实现可调用逻辑，例如 `__call__`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 81-160
````python
 81:         assert self.backend is None  # noqa: S101
 82:         assert self.mode is None  # noqa: S101
 83:         assert self.options is None  # noqa: S101
 84:         return fn
 85: 
 86: 
 87: MODES = {
 88:     "eager": Mode(compile=False),
 89:     "decomp": Mode(compile=True, backend="aot_eager_decomp_partition"),
 90:     "compile_numerics": Mode(
 91:         compile=True,
 92:         options={
 93:             "emulate_precision_casts": True,
 94:             "use_fast_math": False,
 95:             "emulate_division_rounding": True,
 96:             "eager_numerics.division_rounding": True,
 97:             "eager_numerics.disable_ftz": True,
 98:         },
 99:     ),
100:     "compile": Mode(compile=True),
101: }
102: 
103: DTYPES = ["float32", "float16", "bfloat16"]
104: 
105: 
106: async def run(command, capture_output=True, capture_stderr=True, stderr=None, **kwargs):
107:     log.info("Running command: %s", command)
108:     proc = await asyncio.create_subprocess_exec(
109:         *shlex.split(command),
110:         stdout=asyncio.subprocess.PIPE if capture_output else None,
111:         stderr=asyncio.subprocess.STDOUT if capture_stderr else stderr,
112:         stdin=asyncio.subprocess.DEVNULL,
113:         **kwargs,
114:     )
115:     if capture_output:
116:         result = await proc.communicate()
117:         result = result[0].decode("utf-8")
118:         log.info("Command result: %s", result)
119:         log.info("Return code: %s", proc.returncode)
120:         return result
121:     else:
122:         await proc.wait()
123:         log.info("Return code: %s", proc.returncode)
124:         return
125: 
126: 
127: async def copy_file_to_remote(hostname, source, destination):
128:     await run(f"scp {source} {hostname}:{destination}")
129: 
130: 
131: async def execute_on_remote(hostname, command):
132:     await run(f"ssh {hostname} {command}", capture_output=False)
133: 
134: 
135: async def copy_results_from_remote(hostname, run_id):
136:     await run(f"mkdir -p results_{run_id}/logs")
137:     await run(f"scp {hostname}:/workspace/result_*.jsonl results_{run_id}/")
138:     await run(
139:         f"scp {hostname}:/workspace/logs.tar.gz results_{run_id}/logs_{hostname}.tar.gz"
140:     )
141:     await run(f"mkdir -p results_{run_id}/logs")
142:     await run(
143:         f"tar xzf results_{run_id}/logs_{hostname}.tar.gz -C results_{run_id}/logs --strip 2"
144:     )
145:     await run(
146:         "bash -c 'for f in *.trace; do cd $f; uvx tlparse --no-browser *.log & cd ..; done; wait'",
147:         cwd=f"results_{run_id}/logs",
148:     )
149:     await run(f"rm results_{run_id}/logs_{hostname}.tar.gz")
150:     await run(f"bash -c 'rm results_{run_id}/logs/*.trace/*.log'")
151:     await run(f"bash -c 'rm results_{run_id}/logs/*.trace/tl_out/raw*'")
152: 
153: 
154: async def do_numerics_test(args, hostname, gpu, run_id):
155:     await copy_file_to_remote(hostname, __file__, "/workspace/run.py")
156:     await execute_on_remote(hostname, f"python3 /workspace/run.py --worker --gpu {gpu}")
157:     await copy_results_from_remote(hostname, run_id)
158: 
159: 
160: async def reserve_gpu(gpu):
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 161-240
````python
161:     stdout = await run(
162:         f"gpu-dev reserve -g 1 -h 24 -t {gpu} --ignore-no-persist --disk none"
163:     )
164:     # the output has a string like this: SSH Command: ssh gpu-dev-1db65ec7
165:     match = re.search(r"SSH Command: ssh (.*)", stdout)
166:     hostname = match.group(1)
167:     log.info(
168:         "Hostname: %s, Reservation ID: %s",
169:         hostname,
170:         hostname.removeprefix("gpu-dev-"),
171:     )
172:     return hostname, hostname.removeprefix("gpu-dev-")
173: 
174: 
175: async def cancel_reservation(reservation_id):
176:     await run(f"gpu-dev cancel {reservation_id}")
177: 
178: 
179: async def run_on_gpu(gpu, command):
180:     log.info("Reserving GPU %s", gpu)
181:     hostname, reservation_id = await reserve_gpu(gpu)
182:     try:
183:         log.info("Running command on %s", hostname)
184:         await command(hostname=hostname)
185:     except Exception:
186:         log.exception("Error running command on %s", gpu)
187:     finally:
188:         log.info("Cancelling reservation %s", reservation_id)
189:         await cancel_reservation(reservation_id)
190: 
191: 
192: async def launcher(args):
193:     run_id = datetime.datetime.now().strftime("%Y-%m-%d_%H-%M-%S")
194:     await asyncio.gather(
195:         *[
196:             run_on_gpu(
197:                 gpu,
198:                 functools.partial(do_numerics_test, args=args, gpu=gpu, run_id=run_id),
199:             )
200:             for gpu in args.gpu
201:         ]
202:     )
203:     await run(f"{sys.executable} {__file__} --html {run_id}")
204: 
205: 
206: def format_results_to_html(run_id):
207:     results_dir = Path(f"results_{run_id}")
208:     for f in results_dir.glob("*.jsonl"):
209:         html_file = f.with_suffix(".html")
210:         lines = f.read_text().splitlines(keepends=True)
211:         data = [json.loads(line) for line in lines]
212:         KEYS = data[0].keys()
213:         print(KEYS)
214:         FILTER_KEYS = [
215:             "gpu",
216:             "pytorch_version",
217:             "cuda_version",
218:             "pytorch_mode",
219:             "is_golden",
220:             "data_type",
221:             "function",
222:             "pass_type",
223:             "category",
224:             "match_full",
225:             "match_normal",
226:         ]
227:         KEY_VALUES = {k: sorted({line[k] for line in data}) for k in FILTER_KEYS}
228:         with html_file.open("w") as html_f:
229:             html_f.write("<html><body style='font-family: monospace;'>\n")
230:             html_f.write(f"<h1>Results for {f.stem}</h1>\n")
231:             for key, values in KEY_VALUES.items():
232:                 html_f.write(
233:                     f"<details id='section-{key}'><summary>{key} ({len(values)})</summary>\n"
234:                 )
235:                 html_f.write(
236:                     f"<a href='#' onclick='document.querySelectorAll(\"#section-{key} input\").forEach(checkbox => checkbox.checked = true);'>Enable all</a>\n"
237:                 )
238:                 html_f.write(
239:                     f"<a href='#' onclick='document.querySelectorAll(\"#section-{key} input\").forEach(checkbox => checkbox.checked = false);'>Disable all</a>\n"
240:                 )
````
- EN: Implements callable logic such as `format_results_to_html`.
- CN: 实现可调用逻辑，例如 `format_results_to_html`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-320
````python
241:                 for value in values:
242:                     safe_value = str(value).replace(".", "_")
243:                     html_f.write(
244:                         f"<label>\n"
245:                         f"<input type='checkbox' id='filter-{key}-{safe_value}' checked>"
246:                         f"{value}</label>\n"
247:                     )
248:                     html_f.write(
249:                         f"<style>body:has(#filter-{key}-{safe_value}:not(:checked)) .visible-{key}-{safe_value} {{ display: none; }}</style>\n"
250:                     )
251:                 html_f.write("</details>")
252:             # one more filter: show "first" mode only
253:             html_f.write("<details><summary>First mode that fails only</summary>\n")
254:             html_f.write(
255:                 "<label><input type='checkbox' id='filter-mode-first' checked>"
256:                 "Exclude items where a previous mode also fails</label>\n"
257:             )
258:             html_f.write("</details>\n")
259:             # column filter:
260:             html_f.write(
261:                 "<details id='section-columns'><summary>Column filter</summary>\n"
262:             )
263:             html_f.write(
264:                 "<a href='#' onclick='document.querySelectorAll(\"#section-columns input\").forEach(checkbox => checkbox.checked = true);'>Enable all</a>\n"
265:             )
266:             html_f.write(
267:                 "<a href='#' onclick='document.querySelectorAll(\"#section-columns input\").forEach(checkbox => checkbox.checked = false);'>Disable all</a>\n"
268:             )
269:             DEFAULT_HIDDEN_KEYS = [
270:                 "gpu",
271:                 "pytorch_version",
272:                 "cuda_version",
273:                 "is_golden",
274:             ]
275:             for key in KEYS:
276:                 html_f.write(
277:                     f"<label><input type='checkbox' id='filter-column-{key}' {'' if key in DEFAULT_HIDDEN_KEYS else 'checked'}>{key}</label>\n"
278:                 )
279:                 html_f.write(
280:                     f"<style>body:has(#filter-column-{key}:not(:checked)) .visible-column-{key} {{ display: none; }}</style>\n"
281:                 )
282:             html_f.write("</details>\n")
283:             html_f.write("<table>\n")
284:             html_f.write("<tr>\n")
285:             for key in KEYS:
286:                 html_f.write(f"<th class='visible-column-{key}'>{key}</th>\n")
287: 
288:             html_f.write("<th>log</th>\n")
289:             html_f.write("<th>tlparse</th>\n")
290:             html_f.write("</tr>\n")
291:             for line in lines:
292:                 data = json.loads(line)
293:                 classes = [
294:                     f"visible-{k}-{str(data[k]).replace('.', '_')}" for k in KEY_VALUES
295:                 ]
296:                 html_f.write(f"<tr class='{' '.join(classes)}'>\n")
297:                 for key in KEYS:
298:                     html_f.write(f"<td class='visible-column-{key}'>\n")
299:                     if key == "mismatch_sample":
300:                         html_f.write(
301:                             f"<input type='checkbox' id='filter-mismatch-sample-{data['identifier']}'>\n"
302:                         )
303:                         html_f.write(
304:                             f"<style>body:has(#filter-mismatch-sample-{data['identifier']}:not(:checked)) .visible-mismatch-sample-{data['identifier']} {{ display: none; }}</style>\n"
305:                         )
306:                     else:
307:                         html_f.write(f"{data[key]}\n")
308:                     html_f.write("</td>\n")
309:                 html_f.write(
310:                     f"<td><a href='logs/{data['identifier']}.log'>log</a></td>"
311:                 )
312:                 html_f.write(
313:                     f"<td><a href='logs/{data['identifier']}.trace/tl_out/index.html'>trace</a></td>\n"
314:                 )
315:                 html_f.write("</tr>\n")
316:                 if data["mismatch_sample"]:
317:                     html_f.write(
318:                         f"<tr class='visible-mismatch-sample-{data['identifier']} {' '.join(classes)}'>\n"
319:                     )
320:                     html_f.write(f"<td colspan='{len(KEYS) + 2}'><pre>\n")
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 321-400
````python
321:                     html_f.write("<table>\n")
322:                     html_f.write("<tr>\n")
323:                     html_f.write("<th>pos</th>\n")
324:                     html_f.write("<th>input</th>\n")
325:                     html_f.write("<th>output</th>\n")
326:                     html_f.write("<th>golden</th>\n")
327:                     html_f.write("<th>rel_err</th>\n")
328:                     html_f.write("</tr>\n")
329:                     for sample in data["mismatch_sample"]:
330:                         html_f.write("<tr>\n")
331:                         html_f.write(f"<td>{sample['pos']}</td>\n")
332:                         html_f.write(
333:                             f"<td>{', '.join(map(str, sample['input']))}</td>\n"
334:                         )
335:                         html_f.write(f"<td>{sample['output']}</td>\n")
336:                         html_f.write(f"<td>{sample['golden']}</td>\n")
337:                         html_f.write(f"<td>{sample['rel_err']}</td>\n")
338:                         html_f.write("</tr>\n")
339:                     html_f.write("</table>\n")
340:                     html_f.write("</td>\n")
341:                     html_f.write("</tr>\n")
342: 
343:             html_f.write("</table>\n")
344:             html_f.write("<style>\n")
345:             # implement the various filters
346:             html_f.write("</style>\n")
347:             html_f.write("</body></html>\n")
348: 
349: 
350: global_utils_lock = asyncio.Lock()
351: cuda_download_lock = asyncio.Lock()
352: venv_creation_lock = asyncio.Lock()
353: pytorch_build_lock = asyncio.Lock()
354: numerics_test_lock = asyncio.Lock()
355: 
356: 
357: async def create_results(args, config):
358:     pytorch_version, cuda_version = config
359:     async with global_utils_lock:
360:         await ensure_global_utils()
361:     async with cuda_download_lock:
362:         await ensure_cuda(args, cuda_version)
363:     async with venv_creation_lock:
364:         venv = await create_venv(args, config)
365:     async with pytorch_build_lock:
366:         await maybe_build_pytorch(args, config, venv)
367:     async with numerics_test_lock:
368:         await run_numerics_test(args, config, venv)
369: 
370: 
371: async def ensure_global_utils():
372:     if os.environ.get("HAS_GLOBAL_UTILS", "0") == "1":
373:         return
374:     await run("mkdir global_bin")
375:     os.environ["PATH"] = f"/workspace/global_bin:{os.environ.get('PATH', '')}"
376: 
377:     os.environ["UV_INSTALL_DIR"] = "/workspace/global_bin"
378:     os.environ["UV_CACHE_DIR"] = "/workspace/.cache/uv"
379:     await run("wget --no-verbose https://astral.sh/uv/install.sh")
380:     await run("bash install.sh")
381:     await run("rm install.sh")
382: 
383:     await run(
384:         "wget --no-verbose https://github.com/seeraven/gitcache/releases/download/v1.0.29/gitcache_v1.0.29_Ubuntu22.04_x86_64"
385:     )
386:     await run("mv gitcache_v1.0.29_Ubuntu22.04_x86_64 global_bin/gitcache")
387:     await run("chmod +x global_bin/gitcache")
388:     which_gitcache = await run("which gitcache")
389:     await run(f"ln -s {which_gitcache.strip()} global_bin/git")
390:     os.environ["GITCACHE_DIR"] = "/workspace/.cache/gitcache"
391: 
392:     await run("sudo apt-get update")
393:     await run("sudo apt-get install -y ccache git-lfs")
394:     os.environ["CMAKE_C_COMPILER_LAUNCHER"] = "ccache"
395:     os.environ["CMAKE_CXX_COMPILER_LAUNCHER"] = "ccache"
396:     os.environ["CMAKE_CUDA_COMPILER_LAUNCHER"] = "ccache"
397:     os.environ["CCACHE_DIR"] = "/workspace/.cache/ccache"
398:     os.environ["CCACHE_NOHASHDIR"] = "1"
399:     os.environ["CCACHE_BASEDIR"] = "/workspace"
400:     os.environ["CMAKE_BUILD_PARALLEL_LEVEL"] = "20"
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 401-480
````python
401:     os.environ["MAX_JOBS"] = "8"
402:     os.environ["MAKEFLAGS"] = "-j20"
403:     os.environ["NINJAFLAGS"] = "-j20"
404: 
405:     os.environ["HAS_GLOBAL_UTILS"] = "1"
406: 
407: 
408: async def ensure_cuda(args, cuda_version):
409:     if os.path.exists(f"/workspace/cuda-{cuda_version}"):
410:         return
411:     await run("sudo apt-get update")
412:     await run(
413:         f"sudo apt-get install -y cuda-toolkit-{cuda_version.split('.')[0]}-{cuda_version.split('.')[1]}"
414:     )
415: 
416: 
417: async def create_venv(args, config):
418:     pytorch_version, cuda_version = config
419:     env = os.environ.copy()
420:     # cuda_home = f"/usr/local/cuda-{cuda_version}"
421:     cuda_home = (
422:         f"/usr/local/cuda-{cuda_version.split('.')[0]}.{cuda_version.split('.')[1]}"
423:     )
424:     env["CUDA_HOME"] = cuda_home
425:     env["CUDA_ROOT"] = cuda_home
426:     env["PATH"] = f"{cuda_home}/bin:{env.get('PATH', '')}"
427:     env["LD_LIBRARY_PATH"] = f"{cuda_home}/lib64:{env.get('LD_LIBRARY_PATH', '')}"
428:     env["CUDA_NVCC_EXECUTABLE"] = f"{cuda_home}/bin/nvcc"
429:     env["PYTORCH_NVCC"] = f"{cuda_home}/bin/nvcc"
430:     venv_dir = f"/workspace/venv-pytorch_{pytorch_version}-cuda_{cuda_version}"
431:     await run(f"mkdir -p {venv_dir}")
432:     await run(f"uv venv -p {PYTHON_VERSION} --managed-python", cwd=venv_dir, env=env)
433:     # update env with venv paths
434:     env["VIRTUAL_ENV"] = f"{venv_dir}/.venv"
435:     env["PATH"] = f"{venv_dir}/.venv/bin:{env.get('PATH', '')}"
436:     await run("uv pip install pip numpy", env=env, cwd=venv_dir)
437: 
438:     index_url = f"https://download.pytorch.org/whl/cu{cuda_version.replace('.', '')}"
439:     if pytorch_version == "nightly":
440:         await run(
441:             f"uv pip install --pre torch --index-url {index_url}", env=env, cwd=venv_dir
442:         )
443:     elif not pytorch_version.startswith("git:"):
444:         await run(
445:             f"uv pip install torch=={pytorch_version} --index-url {index_url}",
446:             env=env,
447:             cwd=venv_dir,
448:         )
449:     return {"env": env, "cwd": venv_dir}
450: 
451: 
452: async def maybe_build_pytorch(args, config, venv):
453:     pytorch_version, cuda_version = config
454:     if pytorch_version.startswith("git:"):
455:         raise NotImplementedError("Building PyTorch from git is not supported yet")
456:     return
457: 
458: 
459: async def run_numerics_test(args, config, venv):
460:     pytorch_version, cuda_version = config
461:     golden_flag = ""
462:     if args.create_golden:
463:         golden_flag = " --create-golden"
464:     for mode in MODES:
465:         cmd = (
466:             f"python {__file__} --runner --gpu {args.gpu[0]}"
467:             f" --pytorch-version {pytorch_version}"
468:             f" --cuda-version {cuda_version}"
469:             f" --mode {mode}"
470:             f" --golden /workspace/golden{golden_flag}"
471:         )
472:         await run(cmd, capture_output=False, **venv)
473:         golden_flag = ""
474: 
475: 
476: def sortable_config_key(config):
477:     pytorch_version, cuda_version = config
478:     cuda_version = tuple(map(int, cuda_version.split(".")))
479: 
480:     if pytorch_version == "nightly":
````
- EN: Implements callable logic such as `sortable_config_key`.
- CN: 实现可调用逻辑，例如 `sortable_config_key`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 481-560
````python
481:         pytorch_version = (-1,)
482:     elif pytorch_version.startswith("git:"):
483:         pytorch_version = (-2,)
484:     else:
485:         pytorch_version = tuple(map(int, pytorch_version.split(".")))
486: 
487:     return (pytorch_version, cuda_version)
488: 
489: 
490: async def worker(args):
491:     assert len(args.gpu) == 1  # noqa: S101
492:     # find golden task
493:     # a config is a tuple (pytorch_version, cuda_version)
494:     # where pytorch_version can either be a version string or nightly or a git hash
495:     golden_config = sorted(ENABLED_CONFIGS, key=sortable_config_key)[-1]
496:     await run("mkdir -p /workspace/logs")
497:     args.create_golden = True
498:     await create_results(args, golden_config)
499:     args.create_golden = False
500:     await asyncio.gather(
501:         *[
502:             create_results(args, config)
503:             for config in ENABLED_CONFIGS
504:             if config != golden_config
505:         ]
506:     )
507:     await run("tar czf /workspace/logs.tar.gz /workspace/logs")
508: 
509: 
510: CATEGORIES = {
511:     "reduction": ["torch.sum", "torch.mean", "torch.softmax"],
512:     "matrix": ["torch.matmul"],
513:     "normalization": [
514:         "torch.nn.functional.layer_norm",
515:         "torch.nn.functional.rms_norm",
516:     ],
517:     "activation": [
518:         "torch.nn.functional.relu",
519:         "torch.nn.functional.sigmoid",
520:         "torch.nn.functional.tanh",
521:         "torch.nn.functional.gelu",
522:         "torch.nn.functional.silu",
523:     ],
524:     "elementary": [
525:         "torch.sin",
526:         "torch.cos",
527:         "torch.tan",
528:         "torch.sigmoid",
529:         "torch.exp",
530:         "torch.exp2",
531:         "torch.log",
532:         "torch.log2",
533:         "torch.sqrt",
534:         "torch.erf",
535:         "torch.reciprocal",
536:         "torch.rsqrt",
537:     ],
538:     "binary": [
539:         "torch.add",
540:         "torch.sub",
541:         "torch.mul",
542:         "torch.div",
543:         "torch.pow",
544:     ],
545: }
546: 
547: PASS_TYPES = {
548:     "reduction": ["fwd", "bwd_0"],
549:     "matrix": ["fwd", "bwd_0", "bwd_1"],
550:     "normalization": ["fwd", "bwd_0", "bwd_1"],
551:     "activation": ["fwd", "bwd_0"],
552:     "elementary": ["fwd", "bwd_0"],
553:     "binary": ["fwd", "bwd_0", "bwd_1"],
554: }
555: 
556: PASSES = {}
557: 
558: 
559: def register_pass(pass_type):
560:     def decorator(func):
````
- EN: Implements callable logic such as `register_pass`, `decorator`.
- CN: 实现可调用逻辑，例如 `register_pass`, `decorator`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 561-640
````python
561:         PASSES[pass_type] = func
562:         return func
563: 
564:     return decorator
565: 
566: 
567: @register_pass("fwd")
568: def pass_fwd(callable):
569:     def wrapper(*args):
570:         return callable(*args)
571: 
572:     return wrapper
573: 
574: 
575: @register_pass("bwd_0")
576: def pass_bwd_0(callable):
577:     def wrapper(*args):
578:         args = [arg.detach().requires_grad_(idx == 0) for idx, arg in enumerate(args)]
579:         output = callable(*args)
580:         output.sum().backward()
581:         return args[0].grad.detach()
582: 
583:     return wrapper
584: 
585: 
586: @register_pass("bwd_1")
587: def pass_bwd_1(callable):
588:     def wrapper(*args):
589:         args = [arg.detach().requires_grad_(idx == 1) for idx, arg in enumerate(args)]
590:         output = callable(*args)
591:         output.sum().backward()
592:         return args[1].grad.detach()
593: 
594:     return wrapper
595: 
596: 
597: def filter_nan(a):
598:     return torch.where(torch.isnan(a), 0, a)
599: 
600: 
601: def mantissa_bits(dtype) -> int:
602:     return {torch.float32: 23, torch.bfloat16: 5, torch.float16: 10}[dtype]
603: 
604: 
605: def float_to_int_type(dtype):
606:     return {
607:         torch.float32: torch.uint32,
608:         torch.bfloat16: torch.uint16,
609:         torch.float16: torch.uint16,
610:     }[dtype]
611: 
612: 
613: def generate_test_tensor(dtype, slow=False):
614:     if dtype in [torch.float16, torch.bfloat16]:
615:         return filter_nan(
616:             torch.arange(0, 2**16, dtype=torch.int32, device="cuda")
617:             .to(torch.uint16)
618:             .view(dtype)
619:         )
620:     if dtype == torch.float32:
621:         if slow:
622:             return filter_nan(
623:                 torch.arange(0, 2**32, dtype=torch.int64, device="cuda")
624:                 .to(torch.uint32)
625:                 .view(dtype)
626:             )
627:         else:
628:             # E8M5, E5M10, E8M23
629:             result = []
630:             for t in [
631:                 generate_test_tensor(torch.bfloat16),
632:                 generate_test_tensor(torch.float16),
633:             ]:
634:                 orig_dtype = t.dtype
635:                 t = t.to(dtype).view(torch.uint32)
636:                 missing_mantissa_bits = mantissa_bits(dtype) - mantissa_bits(orig_dtype)
637:                 r = torch.randint(
638:                     0, 2**missing_mantissa_bits, t.shape, dtype=t.dtype, device="cuda"
639:                 )
640:                 result.append(t.view(dtype))
````
- EN: Implements callable logic such as `pass_fwd`, `wrapper`, `pass_bwd_0`, `pass_bwd_1`.
- CN: 实现可调用逻辑，例如 `pass_fwd`, `wrapper`, `pass_bwd_0`, `pass_bwd_1`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 641-720
````python
641:                 result.append((t.view(torch.int32) | r.view(torch.int32)).view(dtype))
642:             return filter_nan(torch.cat(result))
643: 
644: 
645: def make_input(dtype, category):
646:     dtype = getattr(torch, dtype)
647:     from torch.testing import make_tensor
648: 
649:     if category in ["matrix"]:
650:         return make_tensor(1024, 1024, dtype=dtype, device="cuda"), make_tensor(
651:             1024, 1024, dtype=dtype, device="cuda"
652:         )
653:     if category == "normalization":
654:         return make_tensor(1024, 1024, dtype=dtype, device="cuda"), make_tensor(
655:             1024, dtype=dtype, device="cuda"
656:         )
657:     if category == "reduction":
658:         return (make_tensor(1024, 1024, dtype=dtype, device="cuda"),)
659:     if category == "binary":
660:         test_tensor = generate_test_tensor(dtype)
661:         # randomly permute twice:
662:         return (
663:             test_tensor[torch.randperm(test_tensor.shape[0])],
664:             test_tensor[torch.randperm(test_tensor.shape[0])],
665:         )
666:     return (generate_test_tensor(dtype),)
667: 
668: 
669: def make_function(function, category):
670:     if category == "reduction":
671:         return eval(f"lambda x: {function}(x, dim=-1)")
672:     elif category == "matrix":
673:         return eval(f"lambda x, y: {function}(x, y)")
674:     elif category == "normalization":
675:         return eval(f"lambda x, y: {function}(x, [x.shape[-1]], y, eps=1e-5)")
676:     elif category == "binary":
677:         return eval(f"lambda x, y: {function}(x, y)")
678:     elif category == "activation":
679:         return eval(f"lambda x: {function}(x)")
680:     elif category == "elementary":
681:         return eval(f"lambda x: {function}(x)")
682:     else:
683:         raise ValueError(f"Unknown category: {category}")
684: 
685: 
686: def rel_err_ulp(a, b, dtype):
687:     return (a - b).abs() / (b.abs() * torch.finfo(dtype).eps + torch.finfo(dtype).tiny)
688: 
689: 
690: def evaluate_output(input, output, golden_output):
691:     # todo: how to make sure this handles denormals in the input well?
692:     # for now, only handle it if the inputs have the same shape, otherwise assume there are none
693:     # handle more than one input
694: 
695:     input = [i.flatten().float() for i in input if i.shape == output.shape]
696: 
697:     dtype = golden_output.dtype
698: 
699:     output = output.flatten().float()
700:     golden_output = golden_output.flatten().float()
701: 
702:     # we are checking subnormals separate from the rest of the numbers
703:     # we also need to check NaNs and Infs carefully
704: 
705:     subnormal_mask = golden_output.abs() < torch.finfo(dtype).smallest_normal
706:     for i in input:
707:         subnormal_mask |= i.abs() < torch.finfo(dtype).smallest_normal
708:     nan_mask_golden = torch.isnan(golden_output)
709:     golden_output = torch.where(nan_mask_golden, float("nan"), golden_output)
710:     nan_mask_output = torch.isnan(output)
711:     output = torch.where(nan_mask_output, float("nan"), output)
712: 
713:     equal_mask = (output == golden_output) | (nan_mask_golden & nan_mask_output)
714:     equal_subnormal_mask = (
715:         torch.where(subnormal_mask, 0, output)
716:         == torch.where(subnormal_mask, 0, golden_output)
717:     ) | equal_mask
718:     output_flushed = torch.where(subnormal_mask, 0.0, output)
719:     golden_flushed = torch.where(subnormal_mask, 0.0, golden_output)
720:     output_flushed = torch.where(~torch.isfinite(output_flushed), 0.0, output_flushed)
````
- EN: Handles module imports such as `torch.testing`.
- CN: 处理模块导入，例如 `torch.testing`。
- EN: Implements callable logic such as `make_input`, `make_function`, `rel_err_ulp`, `evaluate_output`.
- CN: 实现可调用逻辑，例如 `make_input`, `make_function`, `rel_err_ulp`, `evaluate_output`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 721-800
````python
721:     golden_flushed = torch.where(~torch.isfinite(golden_flushed), 0.0, golden_flushed)
722: 
723:     num_nonequal = (~equal_mask).sum().item()
724:     num_nonequal_subnormal = (~equal_subnormal_mask).sum().item()
725: 
726:     err = rel_err_ulp(output_flushed, golden_flushed, dtype)
727:     max_ulp_to_golden = err.max().item()
728:     avg_ulp_to_golden = err.mean().item()
729:     mismatch_sample = []
730:     pos = (~equal_mask).nonzero().squeeze(1)
731:     log.info(
732:         "pos.shape: %s, input: %s, golden_output.shape: %s, output.shape: %s",
733:         pos.shape,
734:         [i.shape for i in input],
735:         golden_output.shape,
736:         output.shape,
737:     )
738:     ordered = torch.argsort(err[pos], descending=True)
739:     random = torch.randperm(pos.shape[0])
740:     for sampling in [ordered, random]:
741:         for i in range(min(pos.shape[0], 5)):
742:             sample_idx = pos[sampling[i]]
743:             mismatch_sample.append(
744:                 {
745:                     "pos": sample_idx.item(),
746:                     "input": [inp[sample_idx].item() for inp in input],
747:                     "output": output[sample_idx].item(),
748:                     "golden": golden_output[sample_idx].item(),
749:                     "rel_err": err[sample_idx].item(),
750:                 }
751:             )
752: 
753:     # use hashlib.sha256 to hash the tensors
754:     # this i
755:     return {
756:         "normal_hash": hashlib.sha256(
757:             output_flushed.cpu().numpy().tobytes()
758:         ).hexdigest()[:8],
759:         "full_hash": hashlib.sha256(output.cpu().numpy().tobytes()).hexdigest()[:8],
760:         "max_ulp_to_golden": max_ulp_to_golden,
761:         "avg_ulp_to_golden": avg_ulp_to_golden,
762:         "num_nonequal": num_nonequal,
763:         "num_nonequal_subnormal": num_nonequal_subnormal,
764:         "num_total": output.shape[0],
765:         "match_full": num_nonequal == 0,
766:         "match_normal": num_nonequal_subnormal == 0,
767:         "mismatch_sample": mismatch_sample,
768:     }
769: 
770: 
771: def create_golden(args):
772:     golden_inputs = {
773:         dtype: {category: make_input(dtype, category) for category in CATEGORIES}
774:         for dtype in DTYPES
775:     }
776:     golden_outputs = {
777:         dtype: {
778:             category: {
779:                 function: {
780:                     pass_type: MODES[args.mode](
781:                         PASSES[pass_type](make_function(function, category))
782:                     )(*golden_inputs[dtype][category])
783:                     for pass_type in PASS_TYPES[category]
784:                 }
785:                 for function in CATEGORIES[category]
786:             }
787:             for category in CATEGORIES
788:         }
789:         for dtype in DTYPES
790:     }
791:     return golden_inputs, golden_outputs
792: 
793: 
794: def get_metadata(args, dtype, category, function, pass_type):
795:     metadata = {
796:         "gpu": args.gpu[0],
797:         "pytorch_version": args.pytorch_version,
798:         "cuda_version": args.cuda_version,
799:         "pytorch_mode": args.mode,
800:         "is_golden": args.create_golden,
````
- EN: Implements callable logic such as `create_golden`, `get_metadata`.
- CN: 实现可调用逻辑，例如 `create_golden`, `get_metadata`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 801-880
````python
801:         "data_type": dtype,
802:         "function": function,
803:         "pass_type": pass_type,
804:         "category": category,
805:     }
806:     identifier = hashlib.sha256(json.dumps(metadata).encode()).hexdigest()[:8]
807:     metadata["identifier"] = identifier
808:     return metadata
809: 
810: 
811: def run_test_case(args):
812:     golden = torch.load(args.golden)
813:     golden_inputs, golden_outputs = golden
814:     metadata = get_metadata(
815:         args, args.dtype, args.category, args.function, args.pass_type
816:     )
817:     input = golden_inputs[args.dtype][args.category]
818:     callable = make_function(args.function, args.category)
819:     callable = PASSES[args.pass_type](callable)
820:     callable = MODES[args.mode](callable)
821:     output = callable(*input)
822:     evaluation = evaluate_output(
823:         input,
824:         output,
825:         golden_outputs[args.dtype][args.category][args.function][args.pass_type],
826:     )
827:     data = json.dumps(metadata | evaluation)
828:     log.info(data)
829:     print(data)
830: 
831: 
832: test_concurrency = asyncio.Semaphore(16)
833: 
834: 
835: async def launch_test_case(args, dtype, category, function, pass_type):
836:     async with test_concurrency:
837:         env = os.environ | MODES[args.mode].env
838:         metadata = get_metadata(args, dtype, category, function, pass_type)
839:         identifier = metadata["identifier"]
840:         env["TORCH_TRACE"] = f"/workspace/logs/{identifier}.trace"
841:         Path(env["TORCH_TRACE"]).mkdir(parents=True, exist_ok=True)
842:         env["TORCH_LOGS"] = "+all"
843:         env["TORCHINDUCTOR_FORCE_DISABLE_CACHES"] = "1"
844:         env["TRITON_ALWAYS_COMPILE"] = "1"
845:         cmd_args = [
846:             "--test",
847:             f"--gpu {args.gpu[0]}",
848:             f"--pytorch-version {args.pytorch_version}",
849:             f"--cuda-version {args.cuda_version}",
850:             f"--mode {args.mode}",
851:             f"--golden {args.golden}",
852:             f"--dtype {dtype}",
853:             f"--category {category}",
854:             f"--function {function}",
855:             f"--pass-type {pass_type}",
856:         ]
857:         if args.create_golden:
858:             cmd_args.append("--create-golden")
859:         return await run(
860:             f"{sys.executable} {__file__} {' '.join(cmd_args)}",
861:             env=env,
862:             capture_stderr=False,
863:             stderr=open(f"/workspace/logs/{identifier}.log", "w"),
864:         )
865: 
866: 
867: async def runner(args):
868:     log.info("Runner called! %s", args)
869: 
870:     if args.create_golden:
871:         golden = create_golden(args)
872:         torch.save(golden, args.golden)
873: 
874:     results = await asyncio.gather(
875:         *[
876:             launch_test_case(
877:                 args,
878:                 dtype,
879:                 category,
880:                 function,
````
- EN: Implements callable logic such as `run_test_case`.
- CN: 实现可调用逻辑，例如 `run_test_case`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 881-960
````python
881:                 pass_type,
882:             )
883:             for dtype, category in itertools.product(DTYPES, CATEGORIES)
884:             for function, pass_type in itertools.product(
885:                 CATEGORIES[category], PASS_TYPES[category]
886:             )
887:         ]
888:     )
889: 
890:     with open(
891:         f"/workspace/result_{args.gpu[0]}_{args.pytorch_version}_{args.cuda_version}.jsonl",
892:         "a+",
893:     ) as result:
894:         for r in results:
895:             result.write(r)
896: 
897: 
898: def main():
899:     parser = argparse.ArgumentParser()
900:     parser.add_argument(
901:         "--launcher",
902:         default=False,
903:         action="store_true",
904:         help="launch workers across machines",
905:     )
906:     parser.add_argument(
907:         "--worker",
908:         default=False,
909:         action="store_true",
910:         help="launches jobs on a single machine",
911:     )
912:     parser.add_argument(
913:         "--runner",
914:         default=False,
915:         action="store_true",
916:         help="runs tests in a specific environment",
917:     )
918:     parser.add_argument(
919:         "--test",
920:         action="store_true",
921:         help="runs a single test case",
922:     )
923:     parser.add_argument("--gpu", nargs="+", type=str, default=["t4"])
924:     parser.add_argument("--html", type=str, default=None)
925:     parser.add_argument("--golden", type=str, default=None)
926:     parser.add_argument("--create-golden", action="store_true")
927:     parser.add_argument("--dtype", type=str)
928:     parser.add_argument("--category", type=str)
929:     parser.add_argument("--function", type=str)
930:     parser.add_argument("--pass-type", type=str)
931:     parser.add_argument("--pytorch-version", type=str)
932:     parser.add_argument("--cuda-version", type=str)
933:     parser.add_argument("--mode")
934:     args = parser.parse_args()
935:     if args.runner:
936:         assert torch is not None  # noqa: S101
937:         torch.set_default_device("cuda")
938:         (gpu,) = args.gpu
939:         logging.basicConfig(
940:             level=logging.INFO,
941:             format=f"%(asctime)s - runner:{gpu}/{args.pytorch_version}/{args.cuda_version}/{args.mode} - %(message)s",
942:         )
943:         asyncio.run(runner(args))
944:     if args.worker:
945:         (gpu,) = args.gpu
946:         logging.basicConfig(
947:             level=logging.INFO, format=f"%(asctime)s - worker:{gpu} - %(message)s"
948:         )
949:         asyncio.run(worker(args))
950:     if args.launcher:
951:         logging.basicConfig(
952:             level=logging.INFO, format="%(asctime)s - launcher - %(message)s"
953:         )
954:         try:
955:             asyncio.run(launcher(args))
956:         except KeyboardInterrupt:
957:             log.error("Cancelled by user")
958:     if args.test:
959:         test_id = (
960:             f"{args.gpu[0]}/{args.pytorch_version}/{args.cuda_version}"
````
- EN: Implements callable logic such as `main`.
- CN: 实现可调用逻辑，例如 `main`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 961-974
````python
961:             f"/{args.mode}/{args.dtype}/{args.category}"
962:             f"/{args.function}/{args.pass_type}"
963:         )
964:         logging.basicConfig(
965:             level=logging.INFO,
966:             format=f"%(asctime)s - test:{test_id} - %(message)s",
967:         )
968:         run_test_case(args)
969:     if args.html:
970:         format_results_to_html(args.html)
971: 
972: 
973: if __name__ == "__main__":
974:     main()
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `Mode` / 符号 `Mode`
- Symbol `__call__` / 符号 `__call__`
- Symbol `format_results_to_html` / 符号 `format_results_to_html`
- Symbol `sortable_config_key` / 符号 `sortable_config_key`

## Dependencies / 依赖关系
- Python imports: `argparse`, `asyncio`, `datetime`, `functools`, `hashlib`, `itertools`, `json`, `logging`, `os`, `re`
- Python 导入: `argparse`, `asyncio`, `datetime`, `functools`, `hashlib`, `itertools`, `json`, `logging`, `os`, `re`
