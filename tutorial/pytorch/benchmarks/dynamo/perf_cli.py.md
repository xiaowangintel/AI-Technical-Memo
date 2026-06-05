# perf_cli.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/dynamo/perf_cli.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-80
````python
 1: #!/usr/bin/env python3
 2: """
 3: CLI tool for launching, summarizing, and reproducing inductor perf regression runs.
 4: 
 5: Usage:
 6:     python benchmarks/dynamo/perf_cli.py launch [--device a100 h100 ...] [--ref BRANCH] [--wait]
 7:     python benchmarks/dynamo/perf_cli.py summary <run-id|branch> [--top 5] [--config PATTERN]
 8:     python benchmarks/dynamo/perf_cli.py repro <run-id> [--model MODEL] [--suite SUITE] [--print-only]
 9: 
10: Requires: gh CLI (authenticated), internet access to S3 (gha-artifacts bucket).
11: """
12: 
13: from __future__ import annotations
14: 
15: import argparse
16: import csv
17: import io
18: import json
19: import os
20: import re
21: import shutil
22: import subprocess
23: import sys
24: import time
25: import urllib.request
26: import zipfile
27: from collections import defaultdict
28: from dataclasses import dataclass, field
29: from datetime import datetime
30: from math import exp, log
31: from pathlib import Path
32: 
33: 
34: WORKFLOWS = {
35:     "a100": {
36:         "name": "inductor-A100-perf-nightly",
37:         "id": 42513231,
38:     },
39:     "a100-compare": {
40:         "name": "inductor-A100-perf-compare",
41:         "id": 50531883,
42:     },
43:     "h100": {
44:         "name": "inductor-perf-nightly-h100",
45:         "id": 144201955,
46:     },
47:     "b200": {
48:         "name": "inductor-perf-b200",
49:         "id": 173716622,
50:     },
51:     "rocm-mi300": {
52:         "name": "inductor-perf-nightly-rocm-mi300",
53:         "id": 197925166,
54:     },
55:     "rocm-mi355": {
56:         "name": "inductor-perf-nightly-rocm-mi355",
57:         "id": 197925165,
58:     },
59:     "x86": {
60:         "name": "inductor-perf-nightly-x86",
61:         "id": 108782874,
62:     },
63:     "x86-zen": {
64:         "name": "inductor-perf-nightly-x86-zen",
65:         "id": 167573808,
66:     },
67:     "aarch64": {
68:         "name": "inductor-perf-nightly-aarch64",
69:         "id": 109196799,
70:     },
71:     "macos": {
72:         "name": "inductor-perf-nightly-macos",
73:         "id": 117199085,
74:     },
75:     "xpu": {
76:         "name": "inductor-perf-nightly-xpu",
77:         "id": 201149053,
78:     },
79: }
80: 
````
- EN: Handles module imports such as `__future__`, `argparse`, `csv`, `io`.
- CN: 处理模块导入，例如 `__future__`, `argparse`, `csv`, `io`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 81-160
````python
 81: DEVICE_CHOICES = sorted(k for k in WORKFLOWS if k != "a100-compare")
 82: 
 83: S3_BUCKET = "gha-artifacts"
 84: S3_URL = f"https://{S3_BUCKET}.s3.amazonaws.com"
 85: REPO = "pytorch/pytorch"
 86: 
 87: # Regex to parse test job names like:
 88: # "cuda13.0-py3.10-gcc11-sm80 / test (inductor_huggingface_perf, 1, 5, linux.aws.a100)"
 89: JOB_RE = re.compile(
 90:     r"test \((?P<config>[^,]+),\s*(?P<shard>\d+),\s*(?P<num_shards>\d+),\s*(?P<runner>[^)]+)\)"
 91: )
 92: 
 93: PERF_CONFIGS = re.compile(r"inductor_(huggingface|timm|torchbench)_perf")
 94: 
 95: SUITE_ALIASES = {
 96:     "hf": "huggingface",
 97:     "huggingface": "huggingface",
 98:     "timm": "timm_models",
 99:     "timm_models": "timm_models",
100:     "tb": "torchbench",
101:     "torchbench": "torchbench",
102: }
103: 
104: 
105: # ---------------------------------------------------------------------------
106: # Helpers
107: # ---------------------------------------------------------------------------
108: 
109: 
110: def gh(*args: str, json_output: bool = False) -> str | dict | list:
111:     cmd = ["gh"] + list(args)
112:     result = subprocess.run(cmd, capture_output=True, text=True)
113:     if result.returncode != 0:
114:         print(f"gh error: {result.stderr.strip()}", file=sys.stderr)
115:         sys.exit(1)
116:     if json_output:
117:         return json.loads(result.stdout)
118:     return result.stdout.strip()
119: 
120: 
121: def git(*args: str) -> str:
122:     result = subprocess.run(["git"] + list(args), capture_output=True, text=True)
123:     return result.stdout.strip()
124: 
125: 
126: def gmean(values: list[float]) -> float:
127:     if not values:
128:         return 0.0
129:     return exp(sum(log(v) for v in values if v > 0) / max(len(values), 1))
130: 
131: 
132: @dataclass
133: class Metric:
134:     name: str
135:     field: str  # attribute on ModelResult
136:     unit: str  # display suffix
137:     higher_is_better: bool
138:     aggregate: str  # "gmean" or "mean"
139: 
140: 
141: METRICS = {
142:     "speedup": Metric("speedup", "speedup", "x", True, "gmean"),
143:     "compilation_latency": Metric(
144:         "compilation latency", "compilation_latency", "s", False, "mean"
145:     ),
146:     "compression_ratio": Metric(
147:         "memory compression", "compression_ratio", "x", True, "gmean"
148:     ),
149:     "abs_latency": Metric("absolute latency", "abs_latency", "ms", False, "mean"),
150: }
151: 
152: METRIC_CHOICES = list(METRICS.keys())
153: 
154: # HUD uses 5% relative threshold for flagging regressions
155: RELATIVE_THRESHOLD = 0.05
156: 
157: WORKFLOW_NAME_TO_NIGHTLY_ID = {
158:     v["name"]: v["id"] for k, v in WORKFLOWS.items() if k != "a100-compare"
159: }
160: # compare's baseline is A100 nightly
````
- EN: Declares or extends types including `Metric`.
- CN: 声明或扩展类型，包括 `Metric`。
- EN: Implements callable logic such as `gh`, `git`, `gmean`.
- CN: 实现可调用逻辑，例如 `gh`, `git`, `gmean`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 161-240
````python
161: WORKFLOW_NAME_TO_NIGHTLY_ID["inductor-A100-perf-compare"] = WORKFLOWS["a100"]["id"]
162: 
163: 
164: def _short_config(config: str, device: str = "") -> str:
165:     """Compact label: e.g. 'a100 cudagraphs huggingface training'."""
166:     c = config
167:     backend = c
168:     for s in ("_huggingface_", "_timm_models_", "_torchbench_"):
169:         if s in c:
170:             backend = c.split(s)[0]
171:             break
172:     backend = backend.removeprefix("inductor_")
173:     mode = "training" if "training" in c else "inference" if "inference" in c else ""
174:     suite = ""
175:     for s in ("huggingface", "timm_models", "torchbench"):
176:         if s in c:
177:             suite = s
178:             break
179:     parts = [p for p in (device, backend, suite, mode) if p]
180:     return " ".join(parts)
181: 
182: 
183: @dataclass
184: class ModelResult:
185:     name: str
186:     speedup: float
187:     abs_latency: float = 0.0
188:     compilation_latency: float = 0.0
189:     compression_ratio: float = 0.0
190:     eager_peak_mem: float = 0.0
191:     dynamo_peak_mem: float = 0.0
192:     config: str = ""
193:     device: str = ""
194: 
195:     @property
196:     def short_config(self) -> str:
197:         return _short_config(self.config, self.device)
198: 
199: 
200: @dataclass
201: class PerfData:
202:     config: str  # e.g. "inductor_with_cudagraphs_huggingface_amp_training_cuda"
203:     models: list[ModelResult] = field(default_factory=list)
204:     device: str = ""
205: 
206:     @property
207:     def suite(self) -> str:
208:         for s in ("huggingface", "timm_models", "torchbench"):
209:             if s in self.config:
210:                 return s
211:         return "unknown"
212: 
213:     @property
214:     def mode(self) -> str:
215:         if "training" in self.config:
216:             return "training"
217:         if "inference" in self.config:
218:             return "inference"
219:         return "unknown"
220: 
221:     @property
222:     def dtype(self) -> str:
223:         # Config format: {backend}_{suite}_{dtype}_{mode}_{device}
224:         # e.g. inductor_with_cudagraphs_huggingface_amp_training_cuda
225:         for s in ("_huggingface_", "_timm_models_", "_torchbench_"):
226:             if s in self.config:
227:                 tail = self.config.split(s, 1)[1]
228:                 # tail is e.g. "amp_training_cuda"
229:                 parts = tail.split("_")
230:                 if parts:
231:                     return parts[0]
232:         return "unknown"
233: 
234:     @property
235:     def short_name(self) -> str:
236:         name = self.config
237:         for s in ("_huggingface_", "_timm_models_", "_torchbench_"):
238:             if s in name:
239:                 name = name.split(s)[0]
240:                 break
````
- EN: Declares or extends types including `ModelResult`, `PerfData`.
- CN: 声明或扩展类型，包括 `ModelResult`, `PerfData`。
- EN: Implements callable logic such as `_short_config`, `short_config`, `suite`, `mode`.
- CN: 实现可调用逻辑，例如 `_short_config`, `short_config`, `suite`, `mode`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-320
````python
241:         return name
242: 
243:     @property
244:     def runtime(self) -> str:
245:         # Last token in config: e.g. "..._training_cuda" → "cuda"
246:         parts = self.config.rsplit("_", 1)
247:         if len(parts) == 2:
248:             return parts[1]
249:         return "unknown"
250: 
251:     @property
252:     def qualified_config(self) -> str:
253:         if self.device:
254:             return f"{self.device}/{self.config}"
255:         return self.config
256: 
257:     def gmean_speedup(self) -> float:
258:         vals = [m.speedup for m in self.models if m.speedup > 0]
259:         return gmean(vals)
260: 
261:     def aggregate_metric(self, metric: Metric) -> float:
262:         vals = [
263:             getattr(m, metric.field)
264:             for m in self.models
265:             if getattr(m, metric.field) > 0
266:         ]
267:         if not vals:
268:             return 0.0
269:         if metric.aggregate == "gmean":
270:             return gmean(vals)
271:         return sum(vals) / len(vals)
272: 
273: 
274: # ---------------------------------------------------------------------------
275: # Artifact downloading
276: # ---------------------------------------------------------------------------
277: 
278: 
279: def get_run_jobs(run_id: int) -> list[dict]:
280:     data = gh(
281:         "run",
282:         "view",
283:         str(run_id),
284:         "--repo",
285:         REPO,
286:         "--json",
287:         "jobs",
288:         json_output=True,
289:     )
290:     return data["jobs"]
291: 
292: 
293: def get_perf_jobs(jobs: list[dict]) -> list[dict]:
294:     perf_jobs = []
295:     for job in jobs:
296:         m = JOB_RE.search(job["name"])
297:         if not m:
298:             continue
299:         config = m.group("config")
300:         if not PERF_CONFIGS.match(config):
301:             continue
302:         if job.get("conclusion") != "success":
303:             continue
304:         perf_jobs.append(
305:             {
306:                 "config": config,
307:                 "shard": m.group("shard"),
308:                 "num_shards": m.group("num_shards"),
309:                 "runner": m.group("runner"),
310:                 "job_id": job["databaseId"],
311:                 "name": job["name"],
312:             }
313:         )
314:     return perf_jobs
315: 
316: 
317: def s3_artifact_url(run_id: int, attempt: int, job: dict) -> str:
318:     config = job["config"]
319:     shard = job["shard"]
320:     num_shards = job["num_shards"]
````
- EN: Implements callable logic such as `runtime`, `qualified_config`, `gmean_speedup`, `aggregate_metric`.
- CN: 实现可调用逻辑，例如 `runtime`, `qualified_config`, `gmean_speedup`, `aggregate_metric`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 321-400
````python
321:     runner = job["runner"]
322:     job_id = job["job_id"]
323:     filename = f"test-reports-test-{config}-{shard}-{num_shards}-{runner}_{job_id}.zip"
324:     return f"{S3_URL}/{REPO}/{run_id}/{attempt}/artifact/{filename}"
325: 
326: 
327: CACHE_DIR = Path.home() / ".cache" / "perf_cli"
328: 
329: 
330: def get_cache_dir(run_id: int, attempt: int) -> Path:
331:     d = CACHE_DIR / f"{run_id}" / f"{attempt}"
332:     d.mkdir(parents=True, exist_ok=True)
333:     return d
334: 
335: 
336: def download_and_extract_csvs(
337:     run_id: int,
338:     jobs: list[dict],
339:     attempt: int = 1,
340:     no_cache: bool = False,
341: ) -> list[tuple[str, str]]:
342:     """Download artifacts and return list of (csv_filename, csv_content) pairs."""
343:     cache = get_cache_dir(run_id, attempt)
344:     results = []
345:     fetched = 0
346: 
347:     for job in jobs:
348:         # Check cache first
349:         cache_key = (
350:             f"{job['config']}-{job['shard']}-{job['num_shards']}-{job['job_id']}"
351:         )
352:         cache_marker = cache / f"{cache_key}.done"
353: 
354:         if not no_cache and cache_marker.exists():
355:             # Read cached CSVs
356:             for csv_file in cache.glob(f"{cache_key}__*.csv"):
357:                 csv_name = csv_file.name.split("__", 1)[1]
358:                 results.append((csv_name, csv_file.read_text()))
359:             continue
360: 
361:         url = s3_artifact_url(run_id, attempt, job)
362:         zip_path = cache / f"{cache_key}.zip"
363:         try:
364:             urllib.request.urlretrieve(url, str(zip_path))
365:             fetched += 1
366:         except urllib.error.HTTPError as e:
367:             print(
368:                 f"  warning: failed to download shard {job['config']} "
369:                 f"shard {job['shard']}: {e}",
370:                 file=sys.stderr,
371:             )
372:             continue
373: 
374:         with zipfile.ZipFile(zip_path) as zf:
375:             for name in zf.namelist():
376:                 if name.endswith("_performance.csv"):
377:                     csv_name = os.path.basename(name)
378:                     with zf.open(name) as f:
379:                         content = f.read().decode("utf-8")
380:                         results.append((csv_name, content))
381:                         # Write to cache
382:                         (cache / f"{cache_key}__{csv_name}").write_text(content)
383: 
384:         # Mark this shard as cached and remove the zip
385:         cache_marker.touch()
386:         zip_path.unlink(missing_ok=True)
387: 
388:     if fetched > 0:
389:         print(f"  downloaded {fetched} shards (cached at {cache})")
390:     elif results:
391:         print(f"  using cached data from {cache}")
392: 
393:     return results
394: 
395: 
396: def parse_csvs(csv_pairs: list[tuple[str, str]], device: str = "") -> list[PerfData]:
397:     grouped: dict[str, list[ModelResult]] = defaultdict(list)
398: 
399:     for csv_name, content in csv_pairs:
400:         # csv_name like: inductor_with_cudagraphs_huggingface_amp_training_cuda_performance.csv
````
- EN: Implements callable logic such as `get_cache_dir`, `download_and_extract_csvs`, `parse_csvs`.
- CN: 实现可调用逻辑，例如 `get_cache_dir`, `download_and_extract_csvs`, `parse_csvs`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 401-480
````python
401:         config = csv_name.replace("_performance.csv", "")
402:         reader = csv.DictReader(io.StringIO(content))
403:         for row in reader:
404:             try:
405:                 speedup = float(row.get("speedup", 0))
406:             except (ValueError, TypeError):
407:                 continue
408:             grouped[config].append(
409:                 ModelResult(
410:                     name=row.get("name", "?"),
411:                     speedup=speedup,
412:                     abs_latency=float(row.get("abs_latency", 0) or 0),
413:                     compilation_latency=float(row.get("compilation_latency", 0) or 0),
414:                     compression_ratio=float(row.get("compression_ratio", 0) or 0),
415:                     eager_peak_mem=float(row.get("eager_peak_mem", 0) or 0),
416:                     dynamo_peak_mem=float(row.get("dynamo_peak_mem", 0) or 0),
417:                     config=config,
418:                     device=device,
419:                 )
420:             )
421: 
422:     return [
423:         PerfData(config=k, models=v, device=device) for k, v in sorted(grouped.items())
424:     ]
425: 
426: 
427: # ---------------------------------------------------------------------------
428: # S-curve rendering
429: # ---------------------------------------------------------------------------
430: 
431: 
432: def subsample(items: list, max_rows: int) -> list:
433:     """Evenly subsample a sorted list, always keeping first and last."""
434:     n = len(items)
435:     if n <= max_rows:
436:         return items
437:     # Always include first and last; evenly space the rest
438:     indices = {0, n - 1}
439:     for i in range(1, max_rows - 1):
440:         indices.add(round(i * (n - 1) / (max_rows - 1)))
441:     return [items[i] for i in sorted(indices)]
442: 
443: 
444: def render_scurve(
445:     perf: PerfData,
446:     metric: Metric,
447:     top_n: int = 5,
448:     term_width: int | None = None,
449:     term_height: int | None = None,
450: ):
451:     if not perf.models:
452:         return
453: 
454:     if term_width is None or term_height is None:
455:         sz = shutil.get_terminal_size((100, 50))
456:         term_width = term_width or sz.columns
457:         term_height = term_height or sz.lines
458: 
459:     live = [m for m in perf.models if getattr(m, metric.field) > 0]
460:     if not live:
461:         return
462: 
463:     sorted_models = sorted(live, key=lambda m: getattr(m, metric.field))
464:     agg = perf.aggregate_metric(metric)
465:     n = len(sorted_models)
466: 
467:     # Reserve lines for header (2) + axis label (1) + padding (2)
468:     max_rows = max(term_height - 5, 15)
469:     display = subsample(sorted_models, max_rows)
470:     skipped = n - len(display)
471: 
472:     agg_label = metric.aggregate
473:     header = f"{perf.config} ({n} data points, {agg_label}={agg:.2f}{metric.unit})"
474:     if skipped > 0:
475:         header += f" [showing {len(display)}/{n}]"
476:     print(f"\n  {header}")
477:     print(f"  {'─' * min(len(header), term_width - 4)}")
478: 
479:     def fmt_val(v: float) -> str:
480:         if metric.unit == "s" or metric.unit == "ms":
````
- EN: Implements callable logic such as `subsample`, `render_scurve`, `fmt_val`.
- CN: 实现可调用逻辑，例如 `subsample`, `render_scurve`, `fmt_val`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 481-560
````python
481:             return f"{v:7.1f}{metric.unit}"
482:         return f"{v:5.2f}{metric.unit}"
483: 
484:     # Layout: "  {name:<max_name}  {val:>8}  {dots}"
485:     sample_val = fmt_val(display[0] and getattr(display[0], metric.field))
486:     max_name = min(max(len(m.name) for m in display), 30)
487:     val_width = len(sample_val)
488:     prefix_len = 2 + max_name + 2 + val_width + 2
489:     plot_width = max(term_width - prefix_len - 1, 20)
490: 
491:     def get_val(m):
492:         return getattr(m, metric.field)
493: 
494:     min_val = get_val(sorted_models[0])
495:     p95_idx = max(0, int(n * 0.95) - 1)
496:     p95_val = get_val(sorted_models[p95_idx])
497: 
498:     # For ratio metrics (speedup, compression_ratio), anchor at 1.0
499:     # For absolute metrics (latency), anchor at 0
500:     if metric.unit == "x":
501:         plot_min = min(min_val, 0.5)
502:         plot_max = max(p95_val * 1.1, 1.5)
503:         marker_val = 1.0
504:         marker_label = "1.0x"
505:     else:
506:         plot_min = 0
507:         plot_max = p95_val * 1.1
508:         marker_val = None
509:         marker_label = None
510: 
511:     span = plot_max - plot_min
512:     if span == 0:
513:         span = 1
514: 
515:     def val_to_col(v: float) -> int:
516:         return max(
517:             0, min(plot_width - 1, int((v - plot_min) / span * (plot_width - 1)))
518:         )
519: 
520:     marker_col = val_to_col(marker_val) if marker_val is not None else None
521: 
522:     for m in display:
523:         name = m.name[:max_name].ljust(max_name)
524:         v = get_val(m)
525:         col = val_to_col(v)
526:         bar = [" "] * plot_width
527:         if marker_col is not None:
528:             bar[marker_col] = "|"
529:         for i in range(col + 1):
530:             if marker_col is not None and i == marker_col:
531:                 bar[i] = "|"
532:             else:
533:                 bar[i] = "·"
534:         print(f"  {name}  {fmt_val(v)}  {''.join(bar)}")
535: 
536:     pad = " " * prefix_len
537:     if marker_label and marker_col is not None:
538:         print(f"{pad}{' ' * marker_col}{marker_label}")
539:     else:
540:         print()
541: 
542: 
543: def print_worst_offenders(perf: PerfData, metric: Metric, top_n: int = 5):
544:     def get_val(m):
545:         return getattr(m, metric.field)
546: 
547:     live = [m for m in perf.models if get_val(m) > 0]
548:     if not live:
549:         return
550:     # "worst" depends on metric direction
551:     if metric.higher_is_better:
552:         worst = sorted(live, key=get_val)[:top_n]
553:     else:
554:         worst = sorted(live, key=get_val, reverse=True)[:top_n]
555:     print(f"\n  Worst offenders ({metric.name}):")
556:     for i, m in enumerate(worst, 1):
557:         v = get_val(m)
558:         parts = [f"{v:.3f}{metric.unit}"]
559:         if m.config:
560:             parts.append(m.short_config)
````
- EN: Implements callable logic such as `get_val`, `val_to_col`, `print_worst_offenders`.
- CN: 实现可调用逻辑，例如 `get_val`, `val_to_col`, `print_worst_offenders`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 561-640
````python
561:         detail = "  ".join(parts)
562:         print(f"    {i}. {m.name:<30}  {detail}")
563: 
564: 
565: # ---------------------------------------------------------------------------
566: # Subcommands
567: # ---------------------------------------------------------------------------
568: 
569: 
570: def build_dispatch_inputs(args) -> list[str]:
571:     """Build -f flags for workflow dispatch inputs from CLI args."""
572:     flags = []
573:     bool_inputs = [
574:         "training",
575:         "inference",
576:         "default",
577:         "dynamic",
578:         "cppwrapper",
579:         "cudagraphs",
580:         "freezing_cudagraphs",
581:         "aotinductor",
582:         "maxautotune",
583:     ]
584:     for name in bool_inputs:
585:         val = getattr(args, name, None)
586:         if val is not None:
587:             flags.extend(["-f", f"{name}={'true' if val else 'false'}"])
588:     if args.benchmark_configs:
589:         flags.extend(["-f", f"benchmark_configs={args.benchmark_configs}"])
590:     return flags
591: 
592: 
593: def dispatch_one(device: str, ref: str, extra_flags: list[str]) -> int | None:
594:     wf = WORKFLOWS[device]
595:     print(f"\nLaunching {wf['name']} on ref: {ref}")
596: 
597:     dispatch_args = [
598:         "workflow",
599:         "run",
600:         str(wf["id"]),
601:         "--repo",
602:         REPO,
603:         "--ref",
604:         ref,
605:     ] + extra_flags
606: 
607:     gh(*dispatch_args)
608:     print("Dispatched. Waiting a few seconds for the run to appear...")
609:     time.sleep(5)
610: 
611:     runs = gh(
612:         "run",
613:         "list",
614:         "--repo",
615:         REPO,
616:         "--workflow",
617:         str(wf["id"]),
618:         "--branch",
619:         ref,
620:         "--limit",
621:         "1",
622:         "--json",
623:         "databaseId,status,url,createdAt",
624:         json_output=True,
625:     )
626:     if not runs:
627:         print("Could not find the dispatched run. Check the Actions tab manually.")
628:         return None
629: 
630:     run = runs[0]
631:     run_id = run["databaseId"]
632:     url = f"https://github.com/{REPO}/actions/runs/{run_id}"
633:     print(f"Run ID:  {run_id}")
634:     print(f"URL:     {url}")
635:     print(f"Status:  {run['status']}")
636:     return run_id
637: 
638: 
639: def wait_for_runs(pending: dict[str, int]) -> dict[str, int]:
640:     """Poll all runs until they complete. Returns dict of successful runs."""
````
- EN: Implements callable logic such as `build_dispatch_inputs`, `dispatch_one`, `wait_for_runs`.
- CN: 实现可调用逻辑，例如 `build_dispatch_inputs`, `dispatch_one`, `wait_for_runs`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 641-720
````python
641:     print(f"\nWaiting for {len(pending)} run(s)...", flush=True)
642:     remaining = dict(pending)
643:     succeeded: dict[str, int] = {}
644:     while remaining:
645:         time.sleep(30)
646:         done = []
647:         for device, run_id in remaining.items():
648:             data = gh(
649:                 "run",
650:                 "view",
651:                 str(run_id),
652:                 "--repo",
653:                 REPO,
654:                 "--json",
655:                 "status,conclusion",
656:                 json_output=True,
657:             )
658:             status = data.get("status", "unknown")
659:             if status == "completed":
660:                 conclusion = data.get("conclusion", "unknown")
661:                 print(f"  {device} (run {run_id}): {conclusion}")
662:                 if conclusion == "success":
663:                     succeeded[device] = run_id
664:                 done.append(device)
665:         for d in done:
666:             del remaining[d]
667:         if remaining:
668:             ts = datetime.now().strftime("%H:%M:%S")
669:             print(f"  [{ts}] still waiting: {', '.join(remaining)}...", flush=True)
670:     return succeeded
671: 
672: 
673: def cmd_launch(args):
674:     ref = args.ref or git("rev-parse", "--abbrev-ref", "HEAD")
675:     if ref == "HEAD":
676:         ref = git("rev-parse", "HEAD")
677: 
678:     extra_flags = build_dispatch_inputs(args)
679:     launched: dict[str, int] = {}
680:     for device in args.device:
681:         if device not in WORKFLOWS:
682:             print(f"Unknown device: {device}", file=sys.stderr)
683:             sys.exit(1)
684:         run_id = dispatch_one(device, ref, extra_flags)
685:         if run_id:
686:             launched[device] = run_id
687: 
688:     if (args.wait or args.wait_and_summarize) and launched:
689:         succeeded = wait_for_runs(launched)
690:         if args.wait_and_summarize and succeeded:
691:             # Use the first device's run ID as the positional arg; pass all
692:             # device→run_id pairs via _run_ids so cmd_summary skips resolution.
693:             first_run = next(iter(succeeded.values()))
694:             summary_args = argparse.Namespace(
695:                 run_id=str(first_run),
696:                 device=list(succeeded.keys()),
697:                 _run_ids=succeeded,
698:                 baseline="latest",
699:                 metric="speedup",
700:                 top=5,
701:                 config=None,
702:                 suite=None,
703:                 mode=None,
704:                 group_by=None,
705:                 attempt=1,
706:                 no_cache=False,
707:             )
708:             print(f"\n{'=' * 70}")
709:             print("Summary")
710:             print(f"{'=' * 70}")
711:             cmd_summary(summary_args)
712: 
713: 
714: def filter_perf(all_perf: list[PerfData], args) -> list[PerfData]:
715:     result = all_perf
716:     if getattr(args, "config", None):
717:         pattern = re.compile(args.config, re.IGNORECASE)
718:         result = [p for p in result if pattern.search(p.config)]
719:     if getattr(args, "suite", None):
720:         suite = SUITE_ALIASES.get(args.suite, args.suite)
````
- EN: Implements callable logic such as `cmd_launch`, `filter_perf`.
- CN: 实现可调用逻辑，例如 `cmd_launch`, `filter_perf`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 721-800
````python
721:         result = [p for p in result if p.suite == suite]
722:     if getattr(args, "mode", None):
723:         result = [p for p in result if p.mode == args.mode]
724:     if getattr(args, "backend", None):
725:         pattern = re.compile(args.backend, re.IGNORECASE)
726:         result = [p for p in result if pattern.search(p.short_name)]
727:     if getattr(args, "dtype", None):
728:         result = [p for p in result if p.dtype == args.dtype]
729:     if getattr(args, "runtime", None):
730:         result = [p for p in result if p.runtime == args.runtime]
731:     return result
732: 
733: 
734: GROUP_KEY_FNS: dict[str, callable] = {
735:     "config": lambda p: p.qualified_config,
736:     "suite": lambda p: p.suite,
737:     "mode": lambda p: p.mode,
738:     "backend": lambda p: p.short_name,
739:     "device": lambda p: p.device or "unknown",
740:     "dtype": lambda p: p.dtype,
741:     "runtime": lambda p: p.runtime,
742: }
743: 
744: GROUP_CHOICES = sorted(GROUP_KEY_FNS.keys())
745: 
746: 
747: def group_perf(all_perf: list[PerfData], group_by: list[str] | None) -> list[PerfData]:
748:     if not group_by:
749:         all_models = []
750:         for p in all_perf:
751:             all_models.extend(p.models)
752:         return [PerfData(config="all", models=all_models)]
753: 
754:     fns = []
755:     for key in group_by:
756:         fn = GROUP_KEY_FNS.get(key)
757:         if fn is None:
758:             print(f"Unknown group-by: {key}", file=sys.stderr)
759:             sys.exit(1)
760:         fns.append(fn)
761: 
762:     def composite_key(p: PerfData) -> str:
763:         return " / ".join(fn(p) for fn in fns)
764: 
765:     groups: dict[str, list[ModelResult]] = defaultdict(list)
766:     for p in all_perf:
767:         groups[composite_key(p)].extend(p.models)
768: 
769:     return [PerfData(config=k, models=v) for k, v in sorted(groups.items())]
770: 
771: 
772: # ---------------------------------------------------------------------------
773: # Baseline comparison
774: # ---------------------------------------------------------------------------
775: 
776: 
777: NIGHTLY_WORKFLOW_IDS = {k: v["id"] for k, v in WORKFLOWS.items() if k != "a100-compare"}
778: 
779: 
780: def _find_latest_run(
781:     branch: str,
782:     device: str,
783: ) -> dict | None:
784:     """Find the latest successful perf nightly run for a branch + device.
785: 
786:     Returns {databaseId, createdAt, headSha} or None.
787:     """
788:     wf_id = NIGHTLY_WORKFLOW_IDS.get(device)
789:     if wf_id is None:
790:         return None
791: 
792:     runs = gh(
793:         "run",
794:         "list",
795:         "--repo",
796:         REPO,
797:         "--workflow",
798:         str(wf_id),
799:         "--branch",
800:         branch,
````
- EN: Implements callable logic such as `group_perf`, `composite_key`, `_find_latest_run`.
- CN: 实现可调用逻辑，例如 `group_perf`, `composite_key`, `_find_latest_run`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 801-880
````python
801:         "--status",
802:         "success",
803:         "--limit",
804:         "1",
805:         "--json",
806:         "databaseId,createdAt,headSha",
807:         json_output=True,
808:     )
809:     if not runs:
810:         return None
811:     return runs[0]
812: 
813: 
814: def resolve_run(branch: str, device: str) -> int:
815:     """Find the latest successful perf nightly run for a branch + device."""
816:     run = _find_latest_run(branch, device)
817:     if not run:
818:         print(
819:             f"No successful perf run found for branch '{branch}' on {device}.",
820:             file=sys.stderr,
821:         )
822:         sys.exit(1)
823:     run_id = run["databaseId"]
824:     created = run["createdAt"][:10]
825:     print(f"Resolved '{branch}' → run {run_id} ({device}, {created})")
826:     return run_id
827: 
828: 
829: def discover_runs(branch: str) -> dict[str, int]:
830:     """Auto-discover all devices with successful runs for a branch.
831: 
832:     Finds the latest commit SHA that has runs, then returns all runs matching
833:     that commit.
834:     """
835:     candidates: list[tuple[str, dict]] = []
836:     for device in DEVICE_CHOICES:
837:         run = _find_latest_run(branch, device)
838:         if run:
839:             candidates.append((device, run))
840: 
841:     if not candidates:
842:         print(
843:             f"No successful perf runs found for branch '{branch}' on any device.",
844:             file=sys.stderr,
845:         )
846:         sys.exit(1)
847: 
848:     # Pick the most recent commit (by createdAt) and collect all runs on it
849:     candidates.sort(key=lambda x: x[1]["createdAt"], reverse=True)
850:     target_sha = candidates[0][1]["headSha"]
851: 
852:     result = {}
853:     for device, run in candidates:
854:         if run["headSha"] == target_sha:
855:             run_id = run["databaseId"]
856:             created = run["createdAt"][:10]
857:             print(
858:                 f"Discovered '{branch}' → run {run_id} ({device}, {created}, {target_sha[:10]})"
859:             )
860:             result[device] = run_id
861: 
862:     return result
863: 
864: 
865: def resolve_runs(branch: str, devices: list[str]) -> dict[str, int]:
866:     """Resolve the latest successful run for each device. Returns {device: run_id}."""
867:     result = {}
868:     for device in devices:
869:         result[device] = resolve_run(branch, device)
870:     return result
871: 
872: 
873: def resolve_baseline(head_run_id: int) -> int:
874:     """Find the latest successful nightly on main, skipping the head run itself."""
875:     run_data = gh(
876:         "run",
877:         "view",
878:         str(head_run_id),
879:         "--repo",
880:         REPO,
````
- EN: Implements callable logic such as `resolve_run`, `discover_runs`, `resolve_runs`, `resolve_baseline`.
- CN: 实现可调用逻辑，例如 `resolve_run`, `discover_runs`, `resolve_runs`, `resolve_baseline`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 881-960
````python
881:         "--json",
882:         "workflowName",
883:         json_output=True,
884:     )
885:     wf_name = run_data.get("workflowName", "")
886:     nightly_id = WORKFLOW_NAME_TO_NIGHTLY_ID.get(wf_name)
887:     if nightly_id is None:
888:         print(
889:             f"Don't know which nightly corresponds to workflow '{wf_name}'",
890:             file=sys.stderr,
891:         )
892:         sys.exit(1)
893: 
894:     runs = gh(
895:         "run",
896:         "list",
897:         "--repo",
898:         REPO,
899:         "--workflow",
900:         str(nightly_id),
901:         "--branch",
902:         "main",
903:         "--status",
904:         "success",
905:         "--limit",
906:         "5",
907:         "--json",
908:         "databaseId,createdAt,headBranch",
909:         json_output=True,
910:     )
911:     # Skip the head run itself to avoid comparing a run to itself
912:     for run in runs:
913:         if run["databaseId"] != head_run_id:
914:             baseline_id = run["databaseId"]
915:             created = run["createdAt"][:10]
916:             print(f"Baseline: run {baseline_id} (main, {created})")
917:             return baseline_id
918: 
919:     print("No suitable baseline nightly found on main.", file=sys.stderr)
920:     sys.exit(1)
921: 
922: 
923: def device_for_workflow(workflow_name: str) -> str:
924:     """Reverse-lookup device key from workflow name."""
925:     for k, v in WORKFLOWS.items():
926:         if v["name"] == workflow_name:
927:             return k
928:     return ""
929: 
930: 
931: @dataclass
932: class RunMeta:
933:     run_id: int
934:     head_sha: str
935:     head_branch: str
936:     workflow_name: str
937:     created_at: str
938:     event: str
939: 
940:     @property
941:     def short_sha(self) -> str:
942:         return self.head_sha[:10]
943: 
944:     @property
945:     def date(self) -> str:
946:         return self.created_at[:10]
947: 
948: 
949: def fetch_run_meta(run_id: int) -> RunMeta:
950:     data = gh(
951:         "api",
952:         f"repos/{REPO}/actions/runs/{run_id}",
953:         "-q",
954:         "{headSha: .head_sha, headBranch: .head_branch, workflowName: .name, createdAt: .created_at, event: .event}",
955:         json_output=True,
956:     )
957:     return RunMeta(
958:         run_id=run_id,
959:         head_sha=data.get("headSha", "unknown"),
960:         head_branch=data.get("headBranch", "unknown"),
````
- EN: Declares or extends types including `RunMeta`.
- CN: 声明或扩展类型，包括 `RunMeta`。
- EN: Implements callable logic such as `device_for_workflow`, `short_sha`, `date`, `fetch_run_meta`.
- CN: 实现可调用逻辑，例如 `device_for_workflow`, `short_sha`, `date`, `fetch_run_meta`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 961-1040
````python
 961:         workflow_name=data.get("workflowName", "unknown"),
 962:         created_at=data.get("createdAt", "unknown"),
 963:         event=data.get("event", "unknown"),
 964:     )
 965: 
 966: 
 967: def print_run_header(
 968:     label: str,
 969:     metas: list[RunMeta],
 970:     configs: list[str] | None = None,
 971: ):
 972:     print(f"\n  {label}")
 973:     print(f"  {'─' * len(label)}")
 974:     if len(metas) == 1:
 975:         m = metas[0]
 976:         print(f"  Run:      {m.run_id}  ({m.workflow_name})")
 977:         print(f"  Commit:   {m.short_sha}  ({m.head_branch}, {m.date})")
 978:     else:
 979:         # Show commit from first (should all match for multi-device)
 980:         print(
 981:             f"  Commit:   {metas[0].short_sha}  ({metas[0].head_branch}, {metas[0].date})"
 982:         )
 983:         print("  Runs:")
 984:         for m in metas:
 985:             print(f"    {m.run_id}  ({m.workflow_name})")
 986:     if configs:
 987:         print(f"  Configs:  {len(configs)} — {', '.join(sorted(configs))}")
 988: 
 989: 
 990: def fetch_run_perf(
 991:     run_id: int,
 992:     attempt: int,
 993:     no_cache: bool,
 994:     device: str = "",
 995:     allow_empty: bool = False,
 996: ) -> list[PerfData]:
 997:     """Fetch and parse perf data for a run."""
 998:     jobs = get_run_jobs(run_id)
 999:     perf_jobs = get_perf_jobs(jobs)
1000:     if not perf_jobs:
1001:         if allow_empty:
1002:             print(f"  no perf jobs in run {run_id}, skipping", file=sys.stderr)
1003:             return []
1004:         print(f"No successful perf jobs in run {run_id}.", file=sys.stderr)
1005:         sys.exit(1)
1006:     csv_pairs = download_and_extract_csvs(run_id, perf_jobs, attempt, no_cache=no_cache)
1007:     if not csv_pairs:
1008:         if allow_empty:
1009:             print(f"  no CSVs in run {run_id}, skipping", file=sys.stderr)
1010:             return []
1011:         print(f"No performance CSVs in run {run_id}.", file=sys.stderr)
1012:         sys.exit(1)
1013:     return parse_csvs(csv_pairs, device=device)
1014: 
1015: 
1016: @dataclass
1017: class ModelDelta:
1018:     name: str
1019:     base_val: float
1020:     head_val: float
1021:     config: str = ""
1022:     device: str = ""
1023: 
1024:     @property
1025:     def delta_pct(self) -> float:
1026:         if self.base_val == 0:
1027:             return 0.0
1028:         return (self.head_val - self.base_val) / self.base_val * 100
1029: 
1030:     @property
1031:     def short_config(self) -> str:
1032:         return _short_config(self.config, self.device)
1033: 
1034: 
1035: @dataclass
1036: class ConfigAgg:
1037:     base_agg: float
1038:     base_count: int
1039:     head_agg: float
1040:     head_count: int
````
- EN: Declares or extends types including `ModelDelta`, `ConfigAgg`.
- CN: 声明或扩展类型，包括 `ModelDelta`, `ConfigAgg`。
- EN: Implements callable logic such as `print_run_header`, `fetch_run_perf`, `delta_pct`, `short_config`.
- CN: 实现可调用逻辑，例如 `print_run_header`, `fetch_run_perf`, `delta_pct`, `short_config`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 1041-1120
````python
1041:     paired_ratio: float  # gmean(head_val / base_val) over paired models
1042:     paired_count: int
1043: 
1044: 
1045: def compute_deltas(
1046:     head_perf: list[PerfData], base_perf: list[PerfData], metric: Metric
1047: ) -> tuple[list[ModelDelta], dict[str, ConfigAgg]]:
1048:     """Join head and base on (device, config, model_name) and compute deltas.
1049: 
1050:     Returns (per_model_deltas, per_config_aggregates).
1051:     per_config_aggregates maps qualified_config -> ConfigAgg.
1052:     """
1053:     # Build base lookup: (device, config, model_name) -> metric value
1054:     base_lookup: dict[tuple[str, str, str], float] = {}
1055:     for perf in base_perf:
1056:         for m in perf.models:
1057:             v = getattr(m, metric.field)
1058:             if v > 0:
1059:                 base_lookup[(perf.device, perf.config, m.name)] = v
1060: 
1061:     deltas = []
1062:     for perf in head_perf:
1063:         for m in perf.models:
1064:             head_val = getattr(m, metric.field)
1065:             if head_val <= 0:
1066:                 continue
1067:             key = (perf.device, perf.config, m.name)
1068:             if key not in base_lookup:
1069:                 continue
1070:             base_val = base_lookup[key]
1071:             deltas.append(
1072:                 ModelDelta(
1073:                     name=m.name,
1074:                     base_val=base_val,
1075:                     head_val=head_val,
1076:                     config=perf.config,
1077:                     device=perf.device,
1078:                 )
1079:             )
1080: 
1081:     # Group deltas by qualified_config for paired aggregates
1082:     deltas_by_qconfig: dict[str, list[ModelDelta]] = defaultdict(list)
1083:     for d in deltas:
1084:         qc = f"{d.device}/{d.config}" if d.device else d.config
1085:         deltas_by_qconfig[qc].append(d)
1086: 
1087:     # Per-config aggregates (keyed by qualified_config for display)
1088:     config_aggs: dict[str, ConfigAgg] = {}
1089:     base_by_qconfig: dict[str, PerfData] = {p.qualified_config: p for p in base_perf}
1090:     for perf in head_perf:
1091:         qc = perf.qualified_config
1092:         if qc not in base_by_qconfig:
1093:             continue
1094:         base_perf_data = base_by_qconfig[qc]
1095:         head_agg = perf.aggregate_metric(metric)
1096:         base_agg = base_perf_data.aggregate_metric(metric)
1097:         head_count = len([m for m in perf.models if getattr(m, metric.field) > 0])
1098:         base_count = len(
1099:             [m for m in base_perf_data.models if getattr(m, metric.field) > 0]
1100:         )
1101: 
1102:         paired = deltas_by_qconfig.get(qc, [])
1103:         ratios = [d.head_val / d.base_val for d in paired if d.base_val > 0]
1104:         config_aggs[qc] = ConfigAgg(
1105:             base_agg=base_agg,
1106:             base_count=base_count,
1107:             head_agg=head_agg,
1108:             head_count=head_count,
1109:             paired_ratio=gmean(ratios) if ratios else 0.0,
1110:             paired_count=len(ratios),
1111:         )
1112: 
1113:     return deltas, config_aggs
1114: 
1115: 
1116: def print_comparison_table(config_aggs: dict[str, ConfigAgg], metric: Metric):
1117:     u = metric.unit
1118:     print(f"\n{'Config':<55} {'base':>16} {'new':>16} {'head/base':>18}")
1119:     print("─" * 108)
1120:     for config in sorted(config_aggs):
````
- EN: Implements callable logic such as `compute_deltas`, `print_comparison_table`.
- CN: 实现可调用逻辑，例如 `compute_deltas`, `print_comparison_table`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 1121-1200
````python
1121:         agg = config_aggs[config]
1122:         flag = ""
1123:         if agg.paired_ratio > 0:
1124:             delta_pct = (agg.paired_ratio - 1.0) * 100
1125:             if abs(delta_pct) > RELATIVE_THRESHOLD * 100:
1126:                 if metric.higher_is_better:
1127:                     flag = " !!" if delta_pct < 0 else " ++"
1128:                 else:
1129:                     flag = " !!" if delta_pct > 0 else " ++"
1130:         print(
1131:             f"  {config:<53} "
1132:             f"{agg.base_agg:>5.2f}{u} (n={agg.base_count}) "
1133:             f"{agg.head_agg:>5.2f}{u} (n={agg.head_count}) "
1134:             f"{agg.paired_ratio:>5.3f}x (n={agg.paired_count}){flag}"
1135:         )
1136: 
1137: 
1138: def print_regressions(deltas: list[ModelDelta], metric: Metric, top_n: int):
1139:     # For higher_is_better metrics, regression = negative delta
1140:     # For lower_is_better metrics, regression = positive delta
1141:     if metric.higher_is_better:
1142:         bad = [d for d in deltas if d.delta_pct < -RELATIVE_THRESHOLD * 100]
1143:         bad.sort(key=lambda d: d.delta_pct)
1144:     else:
1145:         bad = [d for d in deltas if d.delta_pct > RELATIVE_THRESHOLD * 100]
1146:         bad.sort(key=lambda d: d.delta_pct, reverse=True)
1147: 
1148:     if not bad:
1149:         print(f"\n  No regressions (>{RELATIVE_THRESHOLD * 100:.0f}% change).")
1150:         return
1151: 
1152:     print(f"\n  Regressions ({len(bad)} models, showing top {min(top_n, len(bad))}):")
1153:     for i, d in enumerate(bad[:top_n], 1):
1154:         print(
1155:             f"    {i}. {d.name:<30}  "
1156:             f"{d.base_val:.2f}{metric.unit} → {d.head_val:.2f}{metric.unit}  "
1157:             f"{d.delta_pct:>+6.1f}%  {d.short_config}"
1158:         )
1159: 
1160: 
1161: def render_delta_scurve(
1162:     deltas: list[ModelDelta],
1163:     metric: Metric,
1164:     term_width: int | None = None,
1165:     term_height: int | None = None,
1166: ):
1167:     if not deltas:
1168:         return
1169: 
1170:     if term_width is None or term_height is None:
1171:         sz = shutil.get_terminal_size((100, 50))
1172:         term_width = term_width or sz.columns
1173:         term_height = term_height or sz.lines
1174: 
1175:     sorted_deltas = sorted(deltas, key=lambda d: d.delta_pct)
1176:     n = len(sorted_deltas)
1177:     max_rows = max(term_height - 5, 15)
1178:     display = subsample(sorted_deltas, max_rows)
1179:     skipped = n - len(display)
1180: 
1181:     header = f"Delta S-curve ({n} models)"
1182:     if skipped > 0:
1183:         header += f" [showing {len(display)}/{n}]"
1184:     print(f"\n  {header}")
1185:     print(f"  {'─' * min(len(header), term_width - 4)}")
1186: 
1187:     max_name = min(max(len(d.name) for d in display), 28)
1188:     # "  name  +12.3%  {bar}"
1189:     prefix_len = 2 + max_name + 2 + 7 + 2
1190:     plot_width = max(term_width - prefix_len - 1, 20)
1191: 
1192:     # Range: cap at p5/p95 to avoid outlier squishing
1193:     p5_idx = max(0, int(n * 0.05))
1194:     p95_idx = min(n - 1, int(n * 0.95))
1195:     range_lo = min(sorted_deltas[p5_idx].delta_pct, -10)
1196:     range_hi = max(sorted_deltas[p95_idx].delta_pct, 10)
1197:     # Ensure symmetric-ish around 0
1198:     abs_max = max(abs(range_lo), abs(range_hi))
1199:     range_lo = -abs_max
1200:     range_hi = abs_max
````
- EN: Implements callable logic such as `print_regressions`, `render_delta_scurve`.
- CN: 实现可调用逻辑，例如 `print_regressions`, `render_delta_scurve`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 1201-1280
````python
1201:     span = range_hi - range_lo
1202:     if span == 0:
1203:         span = 1
1204: 
1205:     def pct_to_col(pct: float) -> int:
1206:         return max(
1207:             0, min(plot_width - 1, int((pct - range_lo) / span * (plot_width - 1)))
1208:         )
1209: 
1210:     zero_col = pct_to_col(0)
1211: 
1212:     for d in display:
1213:         name = d.name[:max_name].ljust(max_name)
1214:         col = pct_to_col(d.delta_pct)
1215:         bar = [" "] * plot_width
1216:         bar[zero_col] = "|"
1217:         if col <= zero_col:
1218:             for i in range(col, zero_col):
1219:                 bar[i] = "·"
1220:             bar[zero_col] = "|"
1221:         else:
1222:             bar[zero_col] = "|"
1223:             for i in range(zero_col + 1, col + 1):
1224:                 bar[i] = "·"
1225:         print(f"  {name}  {d.delta_pct:>+6.1f}%  {''.join(bar)}")
1226: 
1227:     pad = " " * prefix_len
1228:     print(f"{pad}{' ' * zero_col}0%")
1229: 
1230: 
1231: def print_summary_table(all_perf: list[PerfData], metric: Metric):
1232:     agg_label = metric.aggregate
1233:     print(f"\n{'Config':<65} {'Models':>6} {agg_label:>10}")
1234:     print("─" * 85)
1235:     for perf in all_perf:
1236:         n = len([m for m in perf.models if getattr(m, metric.field) > 0])
1237:         agg = perf.aggregate_metric(metric)
1238:         print(f"  {perf.qualified_config:<63} {n:>6} {agg:>8.2f}{metric.unit}")
1239: 
1240: 
1241: def _resolve_head_runs(args) -> dict[str, int]:
1242:     """Parse run_id arg into {device: run_id} mapping.
1243: 
1244:     run_id can be:
1245:       - A single numeric run ID (device inferred from workflow)
1246:       - A branch name (resolved across all --device values, or auto-discovered)
1247:     Pre-resolved IDs can be passed via args._run_ids.
1248:     """
1249:     # Pre-resolved (from --wait-and-summarize)
1250:     if hasattr(args, "_run_ids") and args._run_ids:
1251:         return args._run_ids
1252: 
1253:     raw = args.run_id
1254:     try:
1255:         run_id = int(raw)
1256:         # Single run ID — infer device from the workflow
1257:         meta = fetch_run_meta(run_id)
1258:         device = device_for_workflow(meta.workflow_name)
1259:         return {device: run_id}
1260:     except ValueError:
1261:         # Branch name
1262:         if args.device:
1263:             return resolve_runs(raw, args.device)
1264:         return discover_runs(raw)
1265: 
1266: 
1267: def cmd_summary(args):
1268:     attempt = args.attempt
1269:     metric = METRICS[args.metric]
1270: 
1271:     head_run_ids = _resolve_head_runs(args)
1272: 
1273:     # Fetch head runs
1274:     auto_discovered = not hasattr(args, "_run_ids") and not args.device
1275:     head_metas: list[RunMeta] = []
1276:     head_perf: list[PerfData] = []
1277:     for device, run_id in list(head_run_ids.items()):
1278:         print(f"Fetching head run {run_id} ({device})...")
1279:         perf = fetch_run_perf(
1280:             run_id,
````
- EN: Implements callable logic such as `pct_to_col`, `print_summary_table`, `_resolve_head_runs`, `cmd_summary`.
- CN: 实现可调用逻辑，例如 `pct_to_col`, `print_summary_table`, `_resolve_head_runs`, `cmd_summary`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 1281-1360
````python
1281:             attempt,
1282:             args.no_cache,
1283:             device=device,
1284:             allow_empty=auto_discovered,
1285:         )
1286:         if not perf:
1287:             del head_run_ids[device]
1288:             continue
1289:         head_metas.append(fetch_run_meta(run_id))
1290:         head_perf.extend(perf)
1291: 
1292:     head_perf = filter_perf(head_perf, args)
1293:     if not head_perf:
1294:         print("No configs matched filters.")
1295:         sys.exit(1)
1296: 
1297:     head_configs = [p.qualified_config for p in head_perf]
1298: 
1299:     # Baseline comparison mode
1300:     if args.baseline and args.baseline.lower() != "none":
1301:         base_metas: list[RunMeta] = []
1302:         base_perf: list[PerfData] = []
1303: 
1304:         for device, head_run_id in head_run_ids.items():
1305:             if args.baseline == "latest":
1306:                 baseline_id = resolve_baseline(head_run_id)
1307:             else:
1308:                 try:
1309:                     baseline_id = int(args.baseline)
1310:                 except ValueError:
1311:                     baseline_id = resolve_run(args.baseline, device)
1312: 
1313:             print(f"Fetching baseline run {baseline_id} ({device})...")
1314:             base_data = fetch_run_perf(
1315:                 baseline_id,
1316:                 attempt,
1317:                 args.no_cache,
1318:                 device=device,
1319:                 allow_empty=auto_discovered,
1320:             )
1321:             if not base_data:
1322:                 continue
1323:             base_metas.append(fetch_run_meta(baseline_id))
1324:             base_perf.extend(base_data)
1325: 
1326:         base_perf = filter_perf(base_perf, args)
1327:         if not base_perf:
1328:             print("No baseline configs matched filters.")
1329:             sys.exit(1)
1330: 
1331:         print_run_header("HEAD", head_metas, head_configs)
1332:         print_run_header("BASE", base_metas, [p.qualified_config for p in base_perf])
1333:         print()
1334: 
1335:         deltas, config_aggs = compute_deltas(head_perf, base_perf, metric)
1336:         if not deltas:
1337:             print("No matching models between head and baseline.")
1338:             sys.exit(1)
1339: 
1340:         print_comparison_table(config_aggs, metric)
1341:         print_regressions(deltas, metric, args.top)
1342:         render_delta_scurve(deltas, metric)
1343:         return
1344: 
1345:     # Absolute mode (no baseline)
1346:     print_run_header("RUN", head_metas, head_configs)
1347:     print()
1348: 
1349:     print_summary_table(head_perf, metric)
1350:     grouped = group_perf(head_perf, args.group_by)
1351:     for perf in grouped:
1352:         print_worst_offenders(perf, metric, args.top)
1353:         render_scurve(perf, metric, args.top)
1354: 
1355: 
1356: CONFIG_RE = re.compile(
1357:     r"(?P<backend>inductor_[a-z_]+?)_"
1358:     r"(?P<suite>huggingface|timm_models|torchbench)_"
1359:     r"(?P<dtype>\w+)_"
1360:     r"(?P<mode>training|inference)_"
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 1361-1440
````python
1361:     r"(?P<device>\w+)"
1362: )
1363: 
1364: 
1365: def config_to_command(
1366:     config: str,
1367:     suite: str,
1368:     model: str | None = None,
1369: ) -> str | None:
1370:     """Turn a config name into a runnable benchmark command."""
1371:     m = CONFIG_RE.match(config)
1372:     if not m:
1373:         return None
1374: 
1375:     backend_variant = m.group("backend")
1376:     dtype = m.group("dtype")
1377:     mode = m.group("mode")
1378:     runtime = m.group("device")
1379: 
1380:     # Runtime → --device flag (strip platform suffix like _x86_zen)
1381:     device_flag = runtime.split("_")[0]  # "cpu_x86_zen" → "cpu"
1382: 
1383:     cmd_parts = [
1384:         "python",
1385:         f"benchmarks/dynamo/{suite}.py",
1386:         f"--{mode}",
1387:         f"--{dtype}",
1388:         "--backend",
1389:         "inductor",
1390:         "--device",
1391:         device_flag,
1392:     ]
1393: 
1394:     if "no_cudagraphs" in backend_variant:
1395:         cmd_parts.append("--disable-cudagraphs")
1396:     if "dynamic" in backend_variant:
1397:         cmd_parts.extend(["--dynamic-shapes", "--dynamic-batch-only"])
1398:     if "cpp_wrapper" in backend_variant:
1399:         cmd_parts.insert(0, "TORCHINDUCTOR_CPP_WRAPPER=1")
1400:         cmd_parts.append("--disable-cudagraphs")
1401:     if "freezing" in backend_variant:
1402:         cmd_parts.append("--freezing")
1403:     if "max_autotune" in backend_variant:
1404:         cmd_parts.insert(0, "TORCHINDUCTOR_MAX_AUTOTUNE=1")
1405:     if "aot_inductor" in backend_variant:
1406:         cmd_parts.append("--export-aot-inductor")
1407:         cmd_parts.append("--disable-cudagraphs")
1408: 
1409:     cmd_parts.extend(["--performance", "--cold-start-latency"])
1410: 
1411:     if model:
1412:         cmd_parts.extend(["--only", model])
1413: 
1414:     cmd_parts.extend(["--output", f"{config}_performance.csv"])
1415: 
1416:     return " ".join(cmd_parts)
1417: 
1418: 
1419: def cmd_repro(args):
1420:     run_ids = _resolve_head_runs(args)
1421: 
1422:     # Fetch perf data to discover configs
1423:     auto_discovered = not hasattr(args, "_run_ids") and not args.device
1424:     all_perf: list[PerfData] = []
1425:     metas: list[RunMeta] = []
1426:     for device, run_id in list(run_ids.items()):
1427:         print(f"Fetching run {run_id} ({device})...")
1428:         metas.append(fetch_run_meta(run_id))
1429:         perf = fetch_run_perf(
1430:             run_id,
1431:             args.attempt,
1432:             no_cache=False,
1433:             device=device,
1434:             allow_empty=auto_discovered,
1435:         )
1436:         all_perf.extend(perf)
1437: 
1438:     all_perf = filter_perf(all_perf, args)
1439:     if not all_perf:
1440:         print("No configs matched filters.")
````
- EN: Implements callable logic such as `config_to_command`, `cmd_repro`.
- CN: 实现可调用逻辑，例如 `config_to_command`, `cmd_repro`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 1441-1520
````python
1441:         sys.exit(1)
1442: 
1443:     print_run_header("REPRO", metas)
1444: 
1445:     configs_seen: dict[str, str] = {}  # config_name → suite
1446:     for perf in all_perf:
1447:         configs_seen[perf.config] = perf.suite
1448: 
1449:     count = 0
1450:     commands: list[str] = []
1451:     for config in sorted(configs_seen):
1452:         suite = configs_seen[config]
1453:         cmd = config_to_command(config, suite, args.model)
1454:         if not cmd:
1455:             continue
1456:         commands.append(f"# {config}\n{cmd}")
1457:         count += 1
1458: 
1459:     print(f"\nReproducible commands ({count} configs):\n")
1460:     for cmd in commands:
1461:         print(f"{cmd}\n")
1462: 
1463: 
1464: PIN_DIR = Path(".ci/docker/ci_commit_pins")
1465: 
1466: 
1467: def read_pin(name: str) -> str:
1468:     """Read a pinned commit or requirements file."""
1469:     path = PIN_DIR / name
1470:     if not path.exists():
1471:         return f"<{name} not found>"
1472:     return path.read_text().strip()
1473: 
1474: 
1475: def cmd_prepare_repro(args):
1476:     suites = set()
1477:     if args.suite:
1478:         suite = SUITE_ALIASES.get(args.suite, args.suite)
1479:         suites = {suite}
1480:     else:
1481:         suites = {"huggingface", "timm_models", "torchbench"}
1482: 
1483:     torchbench_pin = read_pin("torchbench.txt")
1484:     timm_pin = read_pin("timm.txt")
1485:     hf_reqs = read_pin("huggingface-requirements.txt")
1486: 
1487:     print("# Setup commands for inductor perf benchmark suites")
1488:     print("# These mirror what CI does in the inductor-benchmarks Docker image.")
1489:     print("#")
1490:     print("# Pinned versions (commits, package versions) are read live from")
1491:     print("#   .ci/docker/ci_commit_pins/")
1492:     print("# Install steps are based on:")
1493:     print("#   .ci/docker/common/install_inductor_benchmark_deps.sh  (build-time)")
1494:     print("#   .ci/pytorch/test.sh                                   (runtime)")
1495:     print("# If the setup process changes, check those files.")
1496:     print()
1497: 
1498:     if "huggingface" in suites:
1499:         print("# ── HuggingFace ──")
1500:         for line in hf_reqs.splitlines():
1501:             line = line.strip()
1502:             if line and not line.startswith("#"):
1503:                 print(f"pip install {line}")
1504:         print()
1505: 
1506:     if "timm_models" in suites:
1507:         print("# ── Timm ──")
1508:         print(
1509:             f"pip install git+https://github.com/huggingface/pytorch-image-models@{timm_pin}"
1510:         )
1511:         print()
1512: 
1513:     if "torchbench" in suites:
1514:         print("# ── TorchBench ──")
1515:         print("git clone https://github.com/pytorch/benchmark torchbench")
1516:         print(f"cd torchbench && git checkout {torchbench_pin}")
1517:         print("python install.py --continue_on_fail")
1518:         print("cd ..")
1519:         print()
1520:         print("# Set PYTHONPATH so benchmark scripts find torchbench")
````
- EN: Implements callable logic such as `read_pin`, `cmd_prepare_repro`.
- CN: 实现可调用逻辑，例如 `read_pin`, `cmd_prepare_repro`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 1521-1600
````python
1521:         print("export PYTHONPATH=$(pwd)/torchbench")
1522:         print()
1523: 
1524:     print("# ── Runtime dependencies ──")
1525:     print("pip install torchvision torchaudio")
1526:     if "torchbench" in suites:
1527:         print("pip install opencv-python==4.8.0.74")
1528:     print()
1529: 
1530:     print("# ── Environment variables ──")
1531:     print("export TORCHINDUCTOR_FX_GRAPH_CACHE=True")
1532:     print("export TORCHINDUCTOR_AUTOGRAD_CACHE=True")
1533:     print()
1534: 
1535:     if not args.no_repro:
1536:         # Also print repro commands if we have a run_id
1537:         if args.run_id:
1538:             # Reuse the repro logic
1539:             repro_args = argparse.Namespace(
1540:                 run_id=args.run_id,
1541:                 device=args.device,
1542:                 model=args.model,
1543:                 suite=args.suite,
1544:                 mode=args.mode,
1545:                 backend=None,
1546:                 dtype=None,
1547:                 runtime=None,
1548:                 attempt=args.attempt,
1549:             )
1550:             print("# ── Benchmark commands ──")
1551:             cmd_repro(repro_args)
1552: 
1553: 
1554: # ---------------------------------------------------------------------------
1555: # Main
1556: # ---------------------------------------------------------------------------
1557: 
1558: 
1559: def main():
1560:     parser = argparse.ArgumentParser(
1561:         description="CLI tool for inductor perf regression runs",
1562:         formatter_class=argparse.RawDescriptionHelpFormatter,
1563:         epilog="""
1564: Examples:
1565:   # Launch a perf run on your current branch (A100)
1566:   python benchmarks/dynamo/perf_cli.py launch
1567: 
1568:   # Launch on both A100 and H100
1569:   python benchmarks/dynamo/perf_cli.py launch --device a100 h100
1570: 
1571:   # Launch on H100 and wait for completion
1572:   python benchmarks/dynamo/perf_cli.py launch --device h100 --wait
1573: 
1574:   # Launch with only inference, no training
1575:   python benchmarks/dynamo/perf_cli.py launch --no-training --inference
1576: 
1577:   # Launch with dynamic shapes enabled
1578:   python benchmarks/dynamo/perf_cli.py launch --dynamic
1579: 
1580:   # Launch on ROCm MI300
1581:   python benchmarks/dynamo/perf_cli.py launch --device rocm-mi300
1582: 
1583:   # Check your branch against latest main nightly
1584:   python benchmarks/dynamo/perf_cli.py summary my-feature-branch
1585: 
1586:   # Same but on H100
1587:   python benchmarks/dynamo/perf_cli.py summary my-feature-branch --device h100
1588: 
1589:   # Compare across A100 and H100 in one summary
1590:   python benchmarks/dynamo/perf_cli.py summary my-feature-branch --device a100 h100
1591: 
1592:   # Use a specific run ID instead of branch name
1593:   python benchmarks/dynamo/perf_cli.py summary 22842783236
1594: 
1595:   # Compare against a specific baseline (run ID or branch)
1596:   python benchmarks/dynamo/perf_cli.py summary 22842783236 --baseline 22816292132
1597: 
1598:   # Absolute metrics only (no comparison)
1599:   python benchmarks/dynamo/perf_cli.py summary 22842783236 --baseline none
1600: 
````
- EN: Implements callable logic such as `main`.
- CN: 实现可调用逻辑，例如 `main`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 1601-1680
````python
1601:   # Filter to cudagraphs training
1602:   python benchmarks/dynamo/perf_cli.py summary main --config cudagraphs --mode training
1603: 
1604:   # Show commands to reproduce a run locally for a single model
1605:   python benchmarks/dynamo/perf_cli.py repro 22842783236 --model BERT_pytorch --suite tb
1606: """,
1607:     )
1608:     sub = parser.add_subparsers(dest="command", required=True)
1609: 
1610:     # -- launch --
1611:     p_launch = sub.add_parser(
1612:         "launch", help="Launch a perf regression run on your branch"
1613:     )
1614:     p_launch.add_argument(
1615:         "--device",
1616:         nargs="+",
1617:         default=["a100"],
1618:         choices=DEVICE_CHOICES,
1619:         metavar="DEVICE",
1620:         help=f"Devices to launch (default: a100). Choices: {', '.join(DEVICE_CHOICES)}",
1621:     )
1622:     p_launch.add_argument(
1623:         "--ref",
1624:         type=str,
1625:         default=None,
1626:         help="Git ref to benchmark (default: current branch)",
1627:     )
1628:     p_launch.add_argument(
1629:         "--wait", action="store_true", help="Wait for all runs to complete"
1630:     )
1631:     p_launch.add_argument(
1632:         "--wait-and-summarize",
1633:         dest="wait_and_summarize",
1634:         action="store_true",
1635:         help="Wait for all runs, then print summary vs. latest main nightly",
1636:     )
1637:     # Workflow dispatch inputs
1638:     launch_opts = p_launch.add_argument_group("workflow options")
1639:     launch_opts.add_argument(
1640:         "--training",
1641:         action="store_true",
1642:         default=None,
1643:         help="Enable training benchmarks",
1644:     )
1645:     launch_opts.add_argument(
1646:         "--no-training",
1647:         dest="training",
1648:         action="store_false",
1649:         help="Disable training benchmarks",
1650:     )
1651:     launch_opts.add_argument(
1652:         "--inference",
1653:         action="store_true",
1654:         default=None,
1655:         help="Enable inference benchmarks",
1656:     )
1657:     launch_opts.add_argument(
1658:         "--no-inference",
1659:         dest="inference",
1660:         action="store_false",
1661:         help="Disable inference benchmarks",
1662:     )
1663:     launch_opts.add_argument(
1664:         "--cudagraphs", action="store_true", default=None, help="Enable cudagraphs"
1665:     )
1666:     launch_opts.add_argument(
1667:         "--no-cudagraphs",
1668:         dest="cudagraphs",
1669:         action="store_false",
1670:         help="Disable cudagraphs",
1671:     )
1672:     launch_opts.add_argument(
1673:         "--dynamic", action="store_true", default=None, help="Enable dynamic shapes"
1674:     )
1675:     launch_opts.add_argument(
1676:         "--no-dynamic",
1677:         dest="dynamic",
1678:         action="store_false",
1679:         help="Disable dynamic shapes",
1680:     )
````
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 1681-1760
````python
1681:     launch_opts.add_argument(
1682:         "--cppwrapper", action="store_true", default=None, help="Enable cpp wrapper"
1683:     )
1684:     launch_opts.add_argument(
1685:         "--freezing-cudagraphs",
1686:         dest="freezing_cudagraphs",
1687:         action="store_true",
1688:         default=None,
1689:     )
1690:     launch_opts.add_argument("--aotinductor", action="store_true", default=None)
1691:     launch_opts.add_argument("--maxautotune", action="store_true", default=None)
1692:     launch_opts.add_argument(
1693:         "--default", dest="default", action="store_true", default=None
1694:     )
1695:     launch_opts.add_argument(
1696:         "--benchmark-configs",
1697:         dest="benchmark_configs",
1698:         type=str,
1699:         default=None,
1700:         help="Override benchmark_configs input",
1701:     )
1702: 
1703:     # -- summary --
1704:     p_summary = sub.add_parser("summary", help="Summarize results of a perf run")
1705:     p_summary.add_argument(
1706:         "run_id", type=str, help="GitHub Actions run ID or branch name"
1707:     )
1708:     p_summary.add_argument(
1709:         "--device",
1710:         nargs="+",
1711:         default=None,
1712:         choices=DEVICE_CHOICES,
1713:         metavar="DEVICE",
1714:         help=f"Device(s) to summarize (default: auto-discover from branch). Choices: {', '.join(DEVICE_CHOICES)}",
1715:     )
1716:     p_summary.add_argument(
1717:         "--baseline",
1718:         type=str,
1719:         default="latest",
1720:         help="Baseline run ID, 'latest' for most recent main nightly (default), or 'none' to disable",
1721:     )
1722:     p_summary.add_argument(
1723:         "--metric",
1724:         type=str,
1725:         default="speedup",
1726:         choices=METRIC_CHOICES,
1727:         help="Metric to display (default: speedup)",
1728:     )
1729:     p_summary.add_argument(
1730:         "--top",
1731:         type=int,
1732:         default=5,
1733:         help="Number of worst offenders to show (default: 5)",
1734:     )
1735:     # Filters
1736:     filters = p_summary.add_argument_group("filters")
1737:     filters.add_argument(
1738:         "--config",
1739:         type=str,
1740:         default=None,
1741:         help="Regex to filter config names (e.g. 'cudagraphs', 'dynamic')",
1742:     )
1743:     filters.add_argument(
1744:         "--suite", type=str, default=None, help="Filter to suite: hf, timm, tb"
1745:     )
1746:     filters.add_argument(
1747:         "--mode",
1748:         type=str,
1749:         choices=["training", "inference"],
1750:         default=None,
1751:         help="Filter to training or inference",
1752:     )
1753:     filters.add_argument(
1754:         "--backend",
1755:         type=str,
1756:         default=None,
1757:         help="Regex to filter backend (e.g. 'cudagraphs', 'dynamic')",
1758:     )
1759:     filters.add_argument(
1760:         "--dtype",
````
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 1761-1840
````python
1761:         type=str,
1762:         default=None,
1763:         help="Filter to dtype (e.g. amp, float16, bfloat16)",
1764:     )
1765:     filters.add_argument(
1766:         "--runtime",
1767:         type=str,
1768:         default=None,
1769:         help="Filter to runtime (e.g. cuda, cpu, xpu)",
1770:     )
1771:     p_summary.add_argument(
1772:         "--group-by",
1773:         dest="group_by",
1774:         nargs="+",
1775:         default=None,
1776:         choices=GROUP_CHOICES,
1777:         metavar="KEY",
1778:         help=f"Group S-curves by key(s) (default: single combined). Choices: {', '.join(GROUP_CHOICES)}",
1779:     )
1780:     p_summary.add_argument(
1781:         "--attempt", type=int, default=1, help="Run attempt number (default: 1)"
1782:     )
1783:     p_summary.add_argument(
1784:         "--no-cache",
1785:         dest="no_cache",
1786:         action="store_true",
1787:         default=False,
1788:         help="Re-download artifacts even if cached",
1789:     )
1790: 
1791:     # -- repro --
1792:     p_repro = sub.add_parser("repro", help="Reproduce a remote perf run locally")
1793:     p_repro.add_argument(
1794:         "run_id", type=str, help="GitHub Actions run ID or branch name"
1795:     )
1796:     p_repro.add_argument(
1797:         "--device",
1798:         nargs="+",
1799:         default=None,
1800:         choices=DEVICE_CHOICES,
1801:         metavar="DEVICE",
1802:         help="Device(s) (default: auto-discover from branch)",
1803:     )
1804:     p_repro.add_argument(
1805:         "--model",
1806:         type=str,
1807:         default=None,
1808:         help="Run only this model (e.g. BERT_pytorch)",
1809:     )
1810:     p_repro.add_argument(
1811:         "--suite", type=str, default=None, help="Filter to suite (hf, timm, tb)"
1812:     )
1813:     p_repro.add_argument(
1814:         "--mode",
1815:         type=str,
1816:         choices=["training", "inference"],
1817:         default=None,
1818:         help="Filter to mode",
1819:     )
1820:     p_repro.add_argument(
1821:         "--backend",
1822:         type=str,
1823:         default=None,
1824:         help="Regex to filter backend (e.g. 'cudagraphs', 'dynamic')",
1825:     )
1826:     p_repro.add_argument(
1827:         "--dtype", type=str, default=None, help="Filter to dtype (e.g. amp, bfloat16)"
1828:     )
1829:     p_repro.add_argument(
1830:         "--runtime", type=str, default=None, help="Filter to runtime (e.g. cuda, cpu)"
1831:     )
1832:     p_repro.add_argument(
1833:         "--attempt", type=int, default=1, help="Run attempt number (default: 1)"
1834:     )
1835: 
1836:     # -- prepare-repro --
1837:     p_prep = sub.add_parser(
1838:         "prepare-repro", help="Show setup commands to prepare benchmark suites locally"
1839:     )
1840:     p_prep.add_argument(
````
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 1841-1894
````python
1841:         "run_id",
1842:         nargs="?",
1843:         type=str,
1844:         default=None,
1845:         help="Optional: run ID or branch to also show benchmark commands",
1846:     )
1847:     p_prep.add_argument(
1848:         "--device",
1849:         nargs="+",
1850:         default=None,
1851:         choices=DEVICE_CHOICES,
1852:         metavar="DEVICE",
1853:         help="Device(s) for benchmark commands",
1854:     )
1855:     p_prep.add_argument(
1856:         "--suite",
1857:         type=str,
1858:         default=None,
1859:         help="Only show setup for this suite (hf, timm, tb)",
1860:     )
1861:     p_prep.add_argument(
1862:         "--mode",
1863:         type=str,
1864:         choices=["training", "inference"],
1865:         default=None,
1866:         help="Filter benchmark commands to mode",
1867:     )
1868:     p_prep.add_argument(
1869:         "--model", type=str, default=None, help="Filter benchmark commands to model"
1870:     )
1871:     p_prep.add_argument(
1872:         "--no-repro",
1873:         dest="no_repro",
1874:         action="store_true",
1875:         help="Only show setup, skip benchmark commands",
1876:     )
1877:     p_prep.add_argument(
1878:         "--attempt", type=int, default=1, help="Run attempt number (default: 1)"
1879:     )
1880: 
1881:     args = parser.parse_args()
1882: 
1883:     if args.command == "launch":
1884:         cmd_launch(args)
1885:     elif args.command == "summary":
1886:         cmd_summary(args)
1887:     elif args.command == "repro":
1888:         cmd_repro(args)
1889:     elif args.command == "prepare-repro":
1890:         cmd_prepare_repro(args)
1891: 
1892: 
1893: if __name__ == "__main__":
1894:     main()
````
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `gh` / 符号 `gh`
- Symbol `git` / 符号 `git`
- Symbol `gmean` / 符号 `gmean`
- Symbol `Metric` / 符号 `Metric`

## Dependencies / 依赖关系
- Python imports: `__future__`, `argparse`, `csv`, `io`, `json`, `os`, `re`, `shutil`, `subprocess`, `sys`
- Python 导入: `__future__`, `argparse`, `csv`, `io`, `json`, `os`, `re`, `shutil`, `subprocess`, `sys`
