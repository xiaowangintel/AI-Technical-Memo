# common.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/dynamo/common.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-80
````python
 1: #!/usr/bin/env python3
 2: 
 3: from __future__ import annotations
 4: 
 5: import argparse
 6: import collections
 7: import contextlib
 8: import copy
 9: import csv
10: import dataclasses
11: import functools
12: import gc
13: import importlib
14: import itertools
15: import json
16: import logging
17: import os
18: import platform
19: import random
20: import shutil
21: import signal
22: import subprocess
23: import sys
24: import tempfile
25: import time
26: import weakref
27: from contextlib import contextmanager
28: from typing import Any, NamedTuple, overload, TYPE_CHECKING, TypeVar
29: from unittest.mock import MagicMock
30: 
31: import numpy as np
32: import pandas as pd
33: import psutil
34: import yaml
35: from scipy.stats import gmean, ttest_ind
36: from tqdm.auto import tqdm, trange
37: 
38: import torch
39: import torch._dynamo
40: import torch._dynamo.utils
41: import torch._export
42: import torch.distributed
43: import torch.multiprocessing as mp
44: from torch._C import _has_cuda as HAS_CUDA, _has_xpu as HAS_XPU
45: from torch._C._nativert import PyModelRunner
46: from torch._dynamo.profiler import fx_insert_profiling, Profiler
47: from torch._dynamo.testing import (
48:     dummy_fx_compile,
49:     format_speedup,
50:     reset_rng_state,
51:     same,
52: )
53: from torch._dynamo.utils import bitwise_same
54: from torch._logging.scribe import open_source_signpost
55: 
56: 
57: try:
58:     from torch._dynamo.utils import (
59:         clone_inputs,
60:         copy_dynamo_tensor_attributes,
61:         graph_break_reasons,
62:     )
63:     from torch._inductor.utils import fresh_cache
64: except ImportError:
65:     from _dynamo.utils import (
66:         clone_inputs,
67:         copy_dynamo_tensor_attributes,
68:         graph_break_reasons,
69:     )
70:     from _inductor.utils import fresh_cache
71: 
72: import torch._functorch.config
73: from torch._functorch.aot_autograd import set_model_name
74: from torch._inductor import config as inductor_config, metrics
75: from torch._subclasses.fake_tensor import FakeTensorMode
76: from torch.utils import _pytree as pytree
77: from torch.utils._pytree import tree_map, tree_map_only
78: 
79: 
80: try:
````
- EN: Handles module imports such as `__future__`, `argparse`, `collections`, `contextlib`.
- CN: 处理模块导入，例如 `__future__`, `argparse`, `collections`, `contextlib`。

### Lines 81-160
````python
 81:     import torch_xla
 82:     import torch_xla.core.xla_model as xm
 83: 
 84:     # This is to workaround the backward issue https://github.com/pytorch/xla/issues/4174
 85:     torch_xla._XLAC._init_computation_client()
 86: except ImportError:
 87:     # ignore the error if torch_xla is not installed
 88:     pass
 89: 
 90: 
 91: if TYPE_CHECKING:
 92:     from collections.abc import Sequence
 93: 
 94: _D = TypeVar("_D", bound=dict[str, Any])
 95: _T = TypeVar("_T")
 96: 
 97: 
 98: log = logging.getLogger(__name__)
 99: 
100: # We are primarily interested in TF32
101: torch.backends.cuda.matmul.allow_tf32 = True
102: torch.backends.cuda.allow_fp16_bf16_reduction_math_sdp(True)
103: 
104: # Suppress torch.profiler spam
105: os.environ["KINETO_LOG_LEVEL"] = "5"
106: 
107: current_name = ""
108: current_device = ""
109: current_backend = ""
110: current_mode = ""
111: current_dtype = ""
112: current_quantization = ""
113: current_settings = None
114: current_batch_size = None
115: output_filename = None
116: disable_output = False
117: 
118: MAX_DOWNLOAD_ATTEMPTS = 5
119: 
120: 
121: class CI(NamedTuple):
122:     backend: str  # aot_eager or inductor
123:     training: bool
124:     dynamic: bool = False
125:     device: str = "cuda"
126: 
127: 
128: CI_SKIP_OPTIMIZER = {
129:     # HF
130:     "MobileBertForMaskedLM",  # Stack issue in fx
131: }
132: 
133: try:
134:     from .fb.common import INTERNAL_CI_SKIP_DYNAMIC_BATCH_ONLY
135: except ImportError:
136:     INTERNAL_CI_SKIP_DYNAMIC_BATCH_ONLY = set()
137: 
138: try:
139:     from pytorch.benchmark.fb.run_utils import trace_handler
140: except ImportError:
141:     trace_handler = None
142: 
143: 
144: CI_SKIP_DYNAMIC_BATCH_ONLY = {
145:     "sam",
146:     # See https://github.com/mindee/doctr/blob/f2114758d529ed8d3d0030581638f0520b6b98d8/doctr/models/detection/core.py#L89
147:     # It iterates over the batch, which is dynamic, and dynamo chokes
148:     # We should be able to graphbreak there.
149:     "doctr_det_predictor",
150:     "dlrm",
151:     "pyhpc_isoneutral_mixing",
152:     "pyhpc_equation_of_state",
153:     "pyhpc_turbulent_kinetic_energy",
154:     "detectron2_fcos_r_50_fpn",
155:     "detectron2_fasterrcnn_r_101_c4",
156:     "detectron2_fasterrcnn_r_101_dc5",
157:     "detectron2_fasterrcnn_r_101_fpn",
158:     "detectron2_fasterrcnn_r_50_c4",
159:     "detectron2_fasterrcnn_r_50_dc5",
160:     "detectron2_fasterrcnn_r_50_fpn",
````
- EN: Handles module imports such as `torch_xla`, `torch_xla.core.xla_model`, `collections.abc`, `.fb.common`.
- CN: 处理模块导入，例如 `torch_xla`, `torch_xla.core.xla_model`, `collections.abc`, `.fb.common`。
- EN: Declares or extends types including `CI`.
- CN: 声明或扩展类型，包括 `CI`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 161-240
````python
161:     "Reformer",
162:     "llama",
163: }.union(INTERNAL_CI_SKIP_DYNAMIC_BATCH_ONLY)
164: 
165: # These models currently fail accuracy with eager Adam optimizer
166: # so we use SGD when running the full benchmarks
167: # https://github.com/pytorch/pytorch/issues/115966
168: BENCHMARK_USE_SGD = {
169:     # TorchBench
170:     "BERT_pytorch",
171:     "LearningToPaint",
172:     "alexnet",
173:     "dcgan",
174:     "demucs",
175:     "densenet121",
176:     "dlrm",
177:     "fastNLP_Bert",
178:     "mobilenet_v2",
179:     "phlippe_densenet",
180:     "phlippe_resnet",
181:     "pytorch_stargan",
182:     "resnet18",
183:     "shufflenet_v2_x1_0",
184:     "speech_transformer",
185:     "squeezenet1_1",
186:     "stable_diffusion_text_encoder",
187:     "vgg16",
188:     # HF
189:     "AlbertForMaskedLM",
190:     "BartForCausalLM",
191:     "ElectraForCausalLM",
192:     "M2M100ForConditionalGeneration",
193:     "MBartForCausalLM",
194:     "OPTForCausalLM",
195:     "PLBartForCausalLM",
196:     "PegasusForCausalLM",
197:     "TrOCRForCausalLM",
198:     "XGLMForCausalLM",
199:     # TIMM
200:     "adv_inception_v3",
201:     "tf_efficientnet_b0",
202:     "ghostnet_100",
203: }
204: 
205: # These models OOM in CI
206: # due to the extra memory of Adam optimizer states,
207: # so we fall back to SGD in CI
208: CI_USE_SGD = {
209:     "torchrec_dlrm",
210:     "demucs",
211:     "detectron2_fasterrcnn_r_101_c4",
212:     "detectron2_fasterrcnn_r_101_dc5",
213:     "detectron2_fasterrcnn_r_101_fpn",
214:     "detectron2_fasterrcnn_r_50_c4",
215:     "detectron2_fasterrcnn_r_50_dc5",
216:     "detectron2_fasterrcnn_r_50_fpn",
217:     "detectron2_maskrcnn_r_101_c4",
218:     "detectron2_maskrcnn_r_101_fpn",
219:     "detectron2_maskrcnn_r_50_c4",
220:     "detectron2_maskrcnn_r_50_fpn",
221:     "llama_v2_7b_16h",
222:     "mobilenet_v2_quantized_qat",
223:     "phi_1_5 resnet50_quantized_qat",
224:     "BlenderbotForCausalLM",
225:     "DALLE2_pytorch",
226:     "moco",
227:     "timm_efficientdet",
228:     "ghostnet_100",
229:     "inception_v3",
230:     "mobilevit_s",
231:     "pytorch_CycleGAN_and_pix2pix",
232:     "vision_maskrcnn",
233:     "dlrm",
234:     "resnet50",
235:     "dm_nfnet_f0",
236: }
237: 
238: 
239: DO_NOT_CAST_INPUTS = {"stable_diffusion"}
240: 
````
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。

### Lines 241-320
````python
241: 
242: # Maps a benchmark model name to a list of status codes. For any listed entry, we'll
243: # capture TORCH_COMPILE_DEBUG logs in CI runs and preserve them (i.e., for upload) if
244: # the result status matches one listed.
245: CI_PRESERVE_COMPILE_DEBUG = {
246:     # For example:
247:     # "mnasnet1_0": ["fail_accuracy"],
248: }
249: 
250: 
251: @functools.lru_cache(maxsize=1)
252: def load_yaml_file(filename):
253:     filepath = os.path.join(os.path.dirname(__file__), filename)
254: 
255:     with open(filepath) as f:
256:         data = yaml.safe_load(f)
257: 
258:     internal_file_path = os.path.join(os.path.dirname(__file__), "fb", filename)
259:     if os.path.exists(internal_file_path):
260:         with open(internal_file_path) as f:
261:             internal_data = yaml.safe_load(f)
262:             data.update(internal_data)
263: 
264:     def flatten(lst):
265:         for item in lst:
266:             if isinstance(item, list):
267:                 yield from flatten(item)
268:             else:
269:                 yield item
270: 
271:     def maybe_list_to_set(obj):
272:         if isinstance(obj, dict):
273:             return {k: maybe_list_to_set(v) for k, v in obj.items()}
274:         if isinstance(obj, list):
275:             return set(flatten(obj))
276:         return obj
277: 
278:     return maybe_list_to_set(data)
279: 
280: 
281: def model_specified_by_path(path_and_class_str):
282:     return ":" in path_and_class_str
283: 
284: 
285: def load_model_from_path(path_and_class_str):
286:     configs = {}
287:     for kvstr in path_and_class_str.split(","):
288:         k, v = kvstr.split(":")
289:         configs[k] = v
290: 
291:     for name in ["path", "class"]:
292:         if name not in configs:
293:             raise RuntimeError(
294:                 "Invalid --only arguments. Check help message for the correct format"
295:             )
296: 
297:     path = configs["path"]
298:     class_name = configs["class"]
299: 
300:     if path[:1] != "/":
301:         raise RuntimeError(
302:             "Use absolute path since dynamo may change the current working directory which makes using relative path tricky"
303:         )
304: 
305:     spec = importlib.util.spec_from_file_location("module_name", path)
306:     module = importlib.util.module_from_spec(spec)
307:     spec.loader.exec_module(module)
308: 
309:     model_class = getattr(module, class_name)
310:     if not issubclass(model_class, torch.nn.Module):
311:         raise AssertionError(
312:             f"expected {class_name} to be a subclass of torch.nn.Module, got {model_class}"
313:         )
314:     model = model_class()
315:     if not hasattr(model, "get_example_inputs"):
316:         raise AssertionError(
317:             f"expected model {class_name} to have get_example_inputs method"
318:         )
319:     inputs = model.get_example_inputs()
320:     return model, inputs
````
- EN: Implements callable logic such as `load_yaml_file`, `flatten`, `maybe_list_to_set`, `model_specified_by_path`.
- CN: 实现可调用逻辑，例如 `load_yaml_file`, `flatten`, `maybe_list_to_set`, `model_specified_by_path`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 321-400
````python
321: 
322: 
323: def write_outputs(filename, headers, row, upload_to_benchmark_db: bool = True):
324:     """
325:     Write both CSV and JSON outputs using the original CSV output interface
326:     """
327:     global disable_output
328:     if disable_output:
329:         return
330: 
331:     output_csv(filename, headers, row)
332:     if upload_to_benchmark_db:
333:         output_json(filename, headers, row)
334: 
335: 
336: def output_csv(filename, headers, row):
337:     if os.path.exists(filename):
338:         with open(filename) as fd:
339:             lines = list(csv.reader(fd)) or [[]]
340:             if headers and len(headers) > len(lines[0]):
341:                 # if prior results failed the header might not be filled in yet
342:                 lines[0] = headers
343:             else:
344:                 headers = lines[0]
345:     else:
346:         lines = [headers]
347:     lines.append([(f"{x:.6f}" if isinstance(x, float) else x) for x in row])
348:     with open(filename, "w") as fd:
349:         writer = csv.writer(fd, lineterminator="\n")
350:         for line in lines:
351:             writer.writerow(list(line) + ["0"] * (len(headers) - len(line)))
352: 
353: 
354: def output_json(filename, headers, row):
355:     """
356:     Write the result into JSON format, so that it can be uploaded to the benchmark database
357:     to be displayed on OSS dashboard. The JSON format is defined at
358:     https://github.com/pytorch/pytorch/wiki/How-to-integrate-with-PyTorch-OSS-benchmark-database
359:     """
360:     origin = ""
361:     if "torchbench" in filename:
362:         origin = "torchbench"
363:     elif "huggingface" in filename:
364:         origin = "huggingface"
365:     elif "timm_models" in filename:
366:         origin = "timm_models"
367: 
368:     extra_info = {
369:         "device": current_device,
370:         "quantization": current_quantization,
371:         "batch_size": current_batch_size,
372:     }
373:     if current_settings:
374:         extra_info.update(current_settings)
375: 
376:     mapping_headers = {headers[i]: v for i, v in enumerate(row)}
377:     with open(f"{os.path.splitext(filename)[0]}.json", "a") as f:
378:         for header, value in mapping_headers.items():
379:             # These headers are not metric names
380:             if header in ("dev", "name", "batch_size"):
381:                 continue
382: 
383:             # Make sure that the record is valid
384:             if not current_name:
385:                 continue
386: 
387:             record = {
388:                 "benchmark": {
389:                     "name": "TorchInductor",
390:                     "mode": current_mode,
391:                     "dtype": current_dtype,
392:                     "extra_info": extra_info,
393:                 },
394:                 "model": {
395:                     "name": current_name,
396:                     "type": "OSS model",
397:                     "backend": current_backend,
398:                     "origins": [origin],
399:                 },
400:             }
````
- EN: Implements callable logic such as `write_outputs`, `output_csv`, `output_json`.
- CN: 实现可调用逻辑，例如 `write_outputs`, `output_csv`, `output_json`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 401-480
````python
401: 
402:             # NB: When the metric is accuracy, its value is actually a string, i.e. pass, and
403:             # not a number. ClickHouse doesn't support mix types atm. It has a Variant type
404:             # https://clickhouse.com/docs/en/sql-reference/data-types/variant, but this isn't
405:             # recommended by CH team themselves. The workaround here is to store that value
406:             # in the extra_info field instead.
407:             if isinstance(value, str):
408:                 record["metric"] = {
409:                     "name": header,
410:                     "extra_info": {"benchmark_values": [value]},
411:                 }
412:             else:
413:                 record["metric"] = {
414:                     "name": header,
415:                     "benchmark_values": [value],
416:                 }
417: 
418:             print(json.dumps(record), file=f)
419: 
420: 
421: def get_suite_from_model_iter_fn(model_iter_fn):
422:     # TODO: This is a bit of a hack
423:     suite = None
424:     if (runner := getattr(model_iter_fn, "__self__", None)) and hasattr(
425:         runner, "suite_name"
426:     ):
427:         suite = runner.suite_name
428:     return suite
429: 
430: 
431: def output_signpost(data, args, suite, error=None):
432:     from torch.utils._stats import simple_call_counter
433: 
434:     data = data.copy()
435: 
436:     if "name" not in data:
437:         data["name"] = current_name
438: 
439:     if "dev" not in data:
440:         data["dev"] = current_device
441: 
442:     filtered_args = vars(args).copy()
443:     # I generated this list by reading through all the configs and dropping
444:     # ones that looked irrelevant or redundant
445:     for k in [
446:         "filter",
447:         "exclude",
448:         "exclude_exact",
449:         "dump_raw_metrics",
450:         "log_operator_inputs",
451:         "distributed_master_port",
452:         "skip_accuracy_check",
453:         "generate_aot_autograd_stats",
454:         "output",
455:         "output_directory",
456:         "disable_output",
457:         "export_profiler_trace",
458:         "profiler_trace_name",
459:         "explain",
460:         "stats",
461:         "print_memory",
462:         "print_compilation_time",
463:         "print_dataframe_summary",
464:         "print_graph_breaks",
465:         "log_graph_breaks",
466:         "timing",
467:         "progress",
468:         "timeout",
469:         "per_process_memory_fraction",
470:         "minify",
471:         "verbose",
472:         "quiet",
473:         "print_fx",
474:         "print_aten_ops",
475:         "log_conv_args",
476:         "recompile_profiler",
477:         "find_batch_sizes",
478:         # Redundant
479:         "batch_size",
480:         "batch_size_file",
````
- EN: Handles module imports such as `torch.utils._stats`.
- CN: 处理模块导入，例如 `torch.utils._stats`。
- EN: Implements callable logic such as `get_suite_from_model_iter_fn`, `output_signpost`.
- CN: 实现可调用逻辑，例如 `get_suite_from_model_iter_fn`, `output_signpost`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 481-560
````python
481:         "only",
482:         "diff_branch",
483:         "tag",
484:         "coverage",
485:         "overhead",
486:         "speedup_dynamo_ts",
487:         "speedup_fx2trt",
488:         "speedup_fx2trt_fp16",
489:         "accuracy",
490:         "performance",
491:         "tolerance",
492:     ]:
493:         del filtered_args[k]
494: 
495:     event_name = "unknown"
496:     if args.accuracy:
497:         event_name = "accuracy"
498:     elif args.quantization:
499:         event_name = "quantization"
500:     elif args.performance:
501:         event_name = "performance"
502: 
503:     from torch._dynamo.utils import calculate_time_spent, compilation_time_metrics
504: 
505:     wall_time_by_phase = calculate_time_spent()
506: 
507:     open_source_signpost(
508:         subsystem="dynamo_benchmark",
509:         name=event_name,
510:         parameters=json.dumps(
511:             {
512:                 **data,
513:                 # TODO: Arguably the rest of these should be in the CSV too
514:                 "suite": suite,
515:                 # Better than using compile_times utils directly
516:                 # NB: Externally, compilation_metrics colloquially refers to
517:                 # the coarse-grained phase timings, even though internally
518:                 # they are called something else
519:                 "compilation_metrics": wall_time_by_phase,
520:                 "agg_compilation_metrics": {
521:                     k: sum(v) for k, v in compilation_time_metrics.items()
522:                 },
523:                 "detailed_compilation_metrics": compilation_time_metrics,
524:                 "simple_call_counter": simple_call_counter,
525:                 # NB: args has training vs inference
526:                 "args": filtered_args,
527:                 "error": error,
528:             }
529:         ),
530:     )
531: 
532:     return wall_time_by_phase["total_wall_time"]
533: 
534: 
535: def nothing(f):
536:     return f
537: 
538: 
539: @functools.cache
540: def patch_torch_manual_seed():
541:     """Make torch manual seed deterministic. Helps with accuracy testing."""
542: 
543:     def deterministic_torch_manual_seed(*args, **kwargs):
544:         from torch._C import default_generator
545: 
546:         seed = 1337
547:         if HAS_CUDA:
548:             import torch.cuda
549: 
550:             if not torch.cuda._is_in_bad_fork():
551:                 torch.cuda.manual_seed_all(seed)
552:         if HAS_XPU:
553:             import torch.xpu
554: 
555:             if not torch.xpu._is_in_bad_fork():
556:                 torch.xpu.manual_seed_all(seed)
557:         return default_generator.manual_seed(seed)
558: 
559:     torch.manual_seed = deterministic_torch_manual_seed
560: 
````
- EN: Handles module imports such as `torch._dynamo.utils`, `torch._C`, `torch.cuda`, `torch.xpu`.
- CN: 处理模块导入，例如 `torch._dynamo.utils`, `torch._C`, `torch.cuda`, `torch.xpu`。
- EN: Implements callable logic such as `nothing`, `patch_torch_manual_seed`, `deterministic_torch_manual_seed`.
- CN: 实现可调用逻辑，例如 `nothing`, `patch_torch_manual_seed`, `deterministic_torch_manual_seed`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 561-640
````python
561: 
562: def empty_gpu_cache(device):
563:     """
564:     Explicitly empty gpu cache to avoid OOM in subsequent run.
565:     """
566: 
567:     if device not in ["cuda", "xpu", "mps"]:
568:         log.warning(
569:             "Trying to call the empty_gpu_cache for device: %s, which is not in list [cuda, xpu]",
570:             device,
571:         )
572:         return
573: 
574:     getattr(torch, device).empty_cache()
575: 
576: 
577: def synchronize():
578:     pass
579: 
580: 
581: def summarize_graph_break(filename):
582:     """
583:     Sorts and de-dupes the graphs breaks on the reason string. Note that this
584:     function is just a best effort to reduce the logging information. We could
585:     miss some graph breaks because of de-duping. We can further refine this
586:     function as need arises.
587:     """
588:     log_file = f"{filename.rstrip('.csv')}_graph_breaks.csv"
589:     if os.path.exists(log_file):
590:         df = pd.read_csv(log_file)
591:         df = df.sort_values("reason").drop_duplicates(subset="reason")
592: 
593:         # Specialize for multi tensor sgd as reason is not identical
594:         multi_tensor_sgd_row = df.loc[df["reason"].str.contains("_multi_tensor_sgd")]
595:         if len(multi_tensor_sgd_row):
596:             df = df[
597:                 ~df["reason"].str.contains("_multi_tensor_sgd")
598:             ]  # Drop all sgd rows
599:             df = pd.concat(
600:                 [df, pd.DataFrame([multi_tensor_sgd_row.iloc[0]])], axis=0
601:             )  # Add back a single row
602:         df.to_csv(f"{log_file.rstrip('.csv')}_deduped.csv", index=False)
603: 
604: 
605: def print_summary(filename, print_dataframe=False):
606:     if not (filename and os.path.exists(filename)):
607:         return
608:     data = pd.read_csv(filename)
609:     if "tag" in data.columns:
610:         for tag in data.tag.unique():
611:             if tag == "0.0000":
612:                 continue  # This happens for failed runs
613:             print(f"\nSummary for tag={tag}:")
614:             print_summary_table(data[data.tag == tag], print_dataframe=print_dataframe)
615:     else:
616:         print_summary_table(data, print_dataframe=print_dataframe)
617:     summarize_graph_break(filename)
618: 
619: 
620: def print_summary_table(data, print_dataframe=False):
621:     if print_dataframe:
622:         pd.options.display.max_rows = 1000
623:         pd.options.display.max_columns = 1000
624:         pd.options.display.width = 2000
625:         print(data)
626:     width = max(map(len, data.columns))
627:     for col in data.columns:
628:         try:
629:             if col in ("dev", "name", "batch_size", "tag"):
630:                 continue
631:             elif col in ("pct_ops", "pct_time"):
632:                 print(col.ljust(width), f"{data[col].mean():.3%}")
633:             elif col in ("graphs", "graph_calls", "captured_ops", "total_ops"):
634:                 print(col.ljust(width), f"{data[col].mean():.3f}")
635:             elif col in ("compilation_latency"):
636:                 print(col.ljust(width), f"mean={data[col].mean():.3f} seconds")
637:             elif col in ("compression_ratio"):
638:                 print(col.ljust(width), f"mean={data[col].mean():.3f}x")
639:             elif col in ("accuracy"):
640:                 pass_rate = (data[col] == "pass").mean()
````
- EN: Implements callable logic such as `empty_gpu_cache`, `synchronize`, `summarize_graph_break`, `print_summary`.
- CN: 实现可调用逻辑，例如 `empty_gpu_cache`, `synchronize`, `summarize_graph_break`, `print_summary`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 641-720
````python
641:                 print(col.ljust(width), f"pass_rate={100 * pass_rate:.2f}%")
642:             else:
643:                 cdata = data[col]
644:                 print(
645:                     col.ljust(width),
646:                     f"gmean={gmean(cdata):.2f}x mean={cdata.mean():.3f}x",
647:                 )
648:         except Exception:
649:             pass
650: 
651: 
652: def tensor_is_on_xla(tensors):
653:     def visit(x: torch.Tensor):
654:         nonlocal result
655:         if x.device.type == "xla":
656:             result = True
657: 
658:     result = False
659:     tree_map_only(torch.Tensor, visit, tensors)
660:     return result
661: 
662: 
663: def timed(
664:     model,
665:     model_iter_fn,
666:     example_inputs,
667:     times=1,
668:     return_result=False,
669:     collect_outputs=False,
670:     batch_size=None,
671: ):
672:     use_xla = tensor_is_on_xla(example_inputs)
673:     synchronize()
674: 
675:     if batch_size:
676:         patch_torch_manual_seed()
677: 
678:     if use_xla:
679:         xm.mark_step()
680:         xm.wait_device_ops()
681: 
682:     def vary_batch(t: torch.Tensor, new_batch_size) -> torch.Tensor:
683:         for i, s in enumerate(t.size()):
684:             if s == batch_size:
685:                 # If new batch is smaller, we truncate
686:                 if new_batch_size < batch_size:
687:                     indexer = [slice(None)] * t.ndim
688:                     indexer[i] = slice(0, new_batch_size)
689:                     t = t[tuple(indexer)]
690:                 # If new batch is greater, we just duplicate the last row
691:                 # over and over until we hit the desired batch size
692:                 elif new_batch_size > batch_size:
693:                     indexer = [slice(None)] * t.ndim
694:                     indexer[i] = -1
695:                     last_slice = t[tuple(indexer)].unsqueeze(i)
696:                     repeat_shape = list(t.shape)
697:                     repeat_shape[i] = new_batch_size - batch_size
698:                     padding = last_slice.expand(*repeat_shape)
699:                     t = torch.cat([t, padding], dim=i)
700:                 break
701:         return t
702: 
703:     time_total = 0
704:     # Dont collect outputs to correctly measure timing
705:     for i in range(times):
706:         # If batch_size is 1, it too often collides with other non batch size
707:         # dimensions resulting in errors.
708:         if batch_size and batch_size > 1:
709:             # Calculate new batch size by varying the original batch size by up to 20%
710:             # Ensure it's at least greater than 1
711:             variation = random.uniform(0.8, 1.2)
712:             new_batch_size = max(2, int(batch_size * variation))
713:             example_inputs = tree_map_only(
714:                 torch.Tensor, lambda x: vary_batch(x, new_batch_size), example_inputs
715:             )
716:         # Put this call inside the loop to reset the seed for each iteration.
717:         # Don't include reset_rng_state() to correctly measure timing
718:         reset_rng_state(use_xla)
719:         t_iter_begin = time.perf_counter()
720:         result = model_iter_fn(model, example_inputs, collect_outputs=collect_outputs)
````
- EN: Implements callable logic such as `tensor_is_on_xla`, `visit`, `timed`, `vary_batch`.
- CN: 实现可调用逻辑，例如 `tensor_is_on_xla`, `visit`, `timed`, `vary_batch`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 721-800
````python
721: 
722:         # instead of calling sync on result_list, we should call mark_step.
723:         # In training case, result_list may be empty, but we want to
724:         # send all the pending graphs for compilation.
725:         if use_xla:
726:             # For the model running on regular torchxla (baseline), we need the
727:             # mark step to send the accumulated graph for compilation.
728:             #
729:             # For the model running with dynamo/torchxla bridge, in training case,
730:             # we need the mark step to send the optimizer graph out for
731:             # compilation.
732:             xm.mark_step()
733:         t_iter_end = time.perf_counter()
734:         time_total += t_iter_end - t_iter_begin
735: 
736:     t_0 = time.perf_counter()
737:     if use_xla:
738:         xm.wait_device_ops()
739:     synchronize()
740:     t_1 = time.perf_counter()
741:     time_total += t_1 - t_0
742:     return (time_total, result) if return_result else time_total
743: 
744: 
745: @overload
746: def _normalize_bench_inputs(example_inputs: _D) -> tuple[tuple[()], _D]: ...
747: 
748: 
749: @overload
750: def _normalize_bench_inputs(
751:     example_inputs: Sequence[_T],
752: ) -> tuple[tuple[_T, ...], dict[str, Any]]: ...
753: 
754: 
755: def _normalize_bench_inputs(example_inputs):
756:     # NOTE(bowbao): For huggingface benchmark, example_inputs are formatted as dictionary,
757:     # and consumed like `model(**example_inputs)`.
758:     # For other benchmarks, example_inputs are formatted as tuple and consumed
759:     # like `model(*example_inputs)`.
760:     if isinstance(example_inputs, dict):
761:         return (), example_inputs
762:     else:
763:         return tuple(example_inputs), {}
764: 
765: 
766: def _register_dataclass_output_as_pytree(example_outputs) -> None:
767:     # NOTE(angelayi): For huggingface benchmark, some example outputs are
768:     # formatted as a dataclass which pytree cannot consume. So we want
769:     # to register the pytree implementation here
770:     example_outputs_flat = pytree.tree_leaves(example_outputs)
771:     output_dataclass_types = [
772:         type(out) for out in example_outputs_flat if dataclasses.is_dataclass(type(out))
773:     ]
774:     for output_type in output_dataclass_types:
775:         from torch._export.utils import register_dataclass_as_pytree_node
776: 
777:         register_dataclass_as_pytree_node(
778:             output_type,
779:             serialized_type_name=f"{output_type.__module__}.{output_type.__name__}",
780:         )
781: 
782: 
783: class Stats:
784:     totals = collections.defaultdict(collections.Counter)
785: 
786:     @classmethod
787:     def reset_counters(cls):
788:         for k, v in torch._dynamo.utils.counters.items():
789:             cls.totals[k].update(v)
790:         ok = torch._dynamo.utils.counters["frames"]["ok"]
791:         total = torch._dynamo.utils.counters["frames"]["total"]
792:         torch._dynamo.utils.counters.clear()
793:         return ok, total
794: 
795:     @classmethod
796:     def print_summary(cls):
797:         for k, v in sorted(cls.totals.items()):
798:             lines = "\n  ".join(map(str, v.most_common(50)))
799:             print(f"STATS {k}\n  {lines}")
800: 
````
- EN: Handles module imports such as `torch._export.utils`.
- CN: 处理模块导入，例如 `torch._export.utils`。
- EN: Declares or extends types including `Stats`.
- CN: 声明或扩展类型，包括 `Stats`。
- EN: Implements callable logic such as `_normalize_bench_inputs`, `_register_dataclass_output_as_pytree`, `reset_counters`, `print_summary`.
- CN: 实现可调用逻辑，例如 `_normalize_bench_inputs`, `_register_dataclass_output_as_pytree`, `reset_counters`, `print_summary`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。

### Lines 801-880
````python
801:     @classmethod
802:     def aot_summary(cls):
803:         return [cls.totals["aot_autograd"]["total"], cls.totals["aot_autograd"]["ok"]]
804: 
805: 
806: def coverage_experiment(args, model_iter_fn, model, example_inputs, **kwargs):
807:     """
808:     Test operator/model coverage of TorchDynamo and record statistics
809:     taken from a profiler.  This target is mainly intended to check
810:     correctness.
811: 
812:     Writes to ./coverage.csv
813:     """
814:     profiler = Profiler()
815:     frozen_model_iter_fn = torch._dynamo.run(model_iter_fn)
816:     with profiler.prof:
817:         frozen_model_iter_fn(model, example_inputs)
818:     coverage_result = profiler.results()
819:     write_outputs(
820:         output_filename,
821:         (
822:             "dev",
823:             "name",
824:             "batch_size",
825:             "graphs",
826:             "graph_calls",
827:             "captured_ops",
828:             "total_ops",
829:             "pct_ops",
830:             "pct_time",
831:         ),
832:         [
833:             current_device,
834:             current_name,
835:             current_batch_size,
836:         ]
837:         + coverage_result.tocsv(),
838:     )
839:     return coverage_result
840: 
841: 
842: def speedup_experiment_fx2trt(args, model_iter_fn, model, example_inputs):
843:     """
844:     Measure speedups over eager using the trt inference backend. TRT backend is based fx graph
845:     generated by torch._dynamo.
846:     Writes to ./speedups_fx2trt.csv
847:     """
848:     return speedup_experiment(args, model_iter_fn, model, example_inputs)
849: 
850: 
851: # TODO: CompilerProfiler is deprecated, remove this
852: def recompile_profiler_experiment(args, model_iter_fn, model, example_inputs):
853:     prof = torch._dynamo.utils.CompilerProfiler()
854:     opt_model_iter_fn = torch._dynamo.optimize(prof, nopython=args.nopython)(
855:         model_iter_fn
856:     )
857:     opt_model_iter_fn(model, example_inputs)
858:     write_outputs(
859:         output_filename, ["model", "profiler report"], [current_name, prof.report()]
860:     )
861:     met = prof.get_metrics()
862:     guard_failures = len(met["guard_failures"])
863:     return [guard_failures]
864: 
865: 
866: def randomize_input(inputs):
867:     if isinstance(inputs, (list, tuple)):
868:         return type(inputs)([randomize_input(x) for x in inputs])
869:     elif isinstance(inputs, torch.Tensor):
870:         if inputs.dtype in (torch.float32, torch.float64):
871:             torch._dynamo.utils.counters["randomize_input"]["times"] += 1
872:             return torch.randn_like(inputs)
873:         elif inputs.dtype == torch.int64:
874:             # Note: we can not simply tune integer tensors as follows
875:             #   `return torch.randint_like(inputs, high=inputs.max().item())`
876:             # This may break some invariants between tensors.
877:             # E.g. in embedding lookup case, one tensor is the length
878:             # and another is an indices tensor.
879:             return inputs
880:         else:
````
- EN: Implements callable logic such as `aot_summary`, `coverage_experiment`, `speedup_experiment_fx2trt`, `recompile_profiler_experiment`.
- CN: 实现可调用逻辑，例如 `aot_summary`, `coverage_experiment`, `speedup_experiment_fx2trt`, `recompile_profiler_experiment`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 881-960
````python
881:             raise RuntimeError(
882:                 f"randomize_input need support tensor of type {inputs.dtype}"
883:             )
884:     else:
885:         raise RuntimeError(
886:             f"randomize_input can not handle input of type {type(inputs)}"
887:         )
888: 
889: 
890: def maybe_mark_step(args):
891:     if args.trace_on_xla:
892:         xm.mark_step()
893: 
894: 
895: def latency_experiment(args, model_iter_fn, model, example_inputs, mark, **kwargs):
896:     """
897:     Measure latency on a specific backend.
898:     """
899: 
900:     timings = np.zeros((args.repeat,), np.float64)
901:     # if we randomize the input, we should also check the result is correct
902:     should_randomize_input = args.randomize_input
903: 
904:     import contextlib
905: 
906:     from torch._inductor.utils import maybe_profile
907: 
908:     @contextlib.contextmanager
909:     def maybe_mark_profile(*args, **kwargs):
910:         prof: torch.profiler.profile = kwargs.pop("p", None)
911:         mark = kwargs.pop("mark", None)
912:         if prof:
913:             with torch.profiler.record_function(mark):
914:                 yield
915:         else:
916:             yield
917: 
918:     times = args.iterations_per_run
919: 
920:     with maybe_profile(args.export_profiler_trace, **args.profile_details) as p:
921:         for rep in trange(args.repeat, desc="running benchmark"):
922:             inputs = (
923:                 randomize_input(copy.deepcopy(example_inputs))
924:                 if should_randomize_input
925:                 else example_inputs
926:             )
927:             # need call mark_step to perform the computation
928:             # on randomize_input. Otherwise the first call using the
929:             # inputs will incur high penalty then the next one.
930:             maybe_mark_step(args)
931: 
932:             with maybe_mark_profile(p=p, mark=mark):
933:                 timings[rep], actual_output = timed(
934:                     model,
935:                     model_iter_fn,
936:                     inputs,
937:                     return_result=True,
938:                     times=times,
939:                     collect_outputs=args.collect_outputs,
940:                 )
941: 
942:     if args.export_profiler_trace:
943:         name = args.profiler_trace_name + "_" + model.name
944:         if hasattr(args, "rank"):
945:             name += f"_rank_{args.rank}"
946:         name += ".json"
947:         name = os.path.join(torch._dynamo.config.base_dir, name)
948:         p.export_chrome_trace(name)
949:     return timings
950: 
951: 
952: # TODO: This seems to be specifically triggered by torchao testing
953: def latency_experiment_summary(suite_name, args, model, timings, **kwargs):
954:     median = np.median(timings, axis=0)
955:     speedup = median[0] / median[1]
956:     if args.dump_raw_metrics:
957:         np.save(
958:             f"{output_filename[:-4]}-raw_timings-{current_name}-{current_device}.npy",
959:             timings,
960:         )
````
- EN: Handles module imports such as `contextlib`, `torch._inductor.utils`.
- CN: 处理模块导入，例如 `contextlib`, `torch._inductor.utils`。
- EN: Implements callable logic such as `maybe_mark_step`, `latency_experiment`, `maybe_mark_profile`, `latency_experiment_summary`.
- CN: 实现可调用逻辑，例如 `maybe_mark_step`, `latency_experiment`, `maybe_mark_profile`, `latency_experiment_summary`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 961-1040
````python
 961: 
 962:     first_headers = ["dev", "name", "batch_size"]
 963:     first_fields = [current_device, current_name, current_batch_size]
 964:     if "tag" in kwargs:
 965:         first_headers.append("tag")
 966:         first_fields.append(kwargs["tag"])
 967:     headers = first_headers + ["speedup", "abs_latency"]
 968:     row = first_fields + [float(speedup), median[1] * 1000]
 969:     msg = f"{speedup:.3f}x"
 970:     if getattr(args, "_print_latency_ms", False):
 971:         msg = f"{median[0] * 1000:.4f} ms, {median[1] * 1000:.4f} ms, {msg}"
 972:     if args.baseline:
 973:         headers.extend(
 974:             [
 975:                 "baseline",
 976:                 "speedup_vs_baseline",
 977:             ]
 978:         )
 979:         df = pd.read_csv(args.baseline)
 980:         try:
 981:             baseline_speedup = df[df["name"] == current_name]["speedup"].item()
 982:             row.extend([baseline_speedup, speedup / baseline_speedup])
 983:             msg = f"{baseline_speedup:.3f}x -> {speedup:.3f}x [{speedup / baseline_speedup:.3f}x]"
 984:         except (KeyError, ZeroDivisionError):
 985:             row.extend(
 986:                 [
 987:                     0.0,
 988:                     0.0,
 989:                 ]
 990:             )
 991:     if "compilation_latency" in kwargs:
 992:         headers += [
 993:             "compilation_latency",
 994:             "compression_ratio",
 995:             "eager_peak_mem",
 996:             "dynamo_peak_mem",
 997:         ]
 998:         row.append(kwargs["compilation_latency"])
 999:         row.append(kwargs["compression_ratio"])
1000:         row.append(kwargs["eager_peak_mem"])
1001:         row.append(kwargs["dynamo_peak_mem"])
1002: 
1003:     if "cache_lookup_latency" in kwargs:
1004:         headers.append("cache_lookup_latency")
1005:         row.append(kwargs["cache_lookup_latency"])
1006: 
1007:     if "dynamo_stats" in kwargs:
1008:         for k, v in kwargs["dynamo_stats"].items():
1009:             headers.append(k)
1010:             row.append(v)
1011:     write_outputs(
1012:         output_filename,
1013:         headers,
1014:         row,
1015:     )
1016:     c_headers, c_data = torch._dynamo.utils.compile_times(repr="csv", aggregate=True)
1017:     if output_filename.find(".csv") <= 0:
1018:         raise AssertionError(
1019:             f"expected output_filename to be a .csv, but got {output_filename}"
1020:         )
1021:     write_outputs(
1022:         output_filename[:-4] + "_compilation_metrics.csv",
1023:         first_headers + c_headers,
1024:         first_fields + c_data,
1025:     )
1026: 
1027:     # Hypothetically you can use this from other places, but it's currently
1028:     # inaccessible, and when this assert fails you need to update the
1029:     # event_name here to account for the other cases you are using this
1030:     if not any([args.quantization, args.optimus]):
1031:         raise AssertionError("expected args.quantization or args.optimus to be set")
1032:     output_signpost(
1033:         dict(zip(headers, row)),
1034:         args,
1035:         suite_name,
1036:     )
1037: 
1038:     return msg
1039: 
1040: 
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 1041-1120
````python
1041: def speedup_experiment(args, model_iter_fn, model, example_inputs, **kwargs):
1042:     """
1043:     Measure speedups over eager.
1044: 
1045:     Writes to ./speedups.csv
1046:     """
1047:     timings = np.zeros((args.repeat, 2), np.float64)
1048:     # if we randomize the input, we should also check the result is correct
1049:     should_randomize_input = args.randomize_input
1050: 
1051:     import contextlib
1052: 
1053:     from torch._inductor.utils import maybe_profile
1054: 
1055:     @contextlib.contextmanager
1056:     def maybe_mark_profile(*args, **kwargs):
1057:         prof: torch.profiler.profile = kwargs.pop("p", None)
1058:         mark = kwargs.pop("mark", None)
1059:         if prof:
1060:             with torch.profiler.record_function(mark):
1061:                 yield
1062:         else:
1063:             yield
1064: 
1065:     times = args.iterations_per_run
1066: 
1067:     # Use higher tolerance for XLA since XLA cause numerical instability when
1068:     # graph size changes
1069:     tolerance = args.xla_tolerance if args.trace_on_xla else 1e-4
1070:     torch._dynamo.config.repro_tolerance = tolerance
1071: 
1072:     with maybe_profile(args.export_profiler_trace, **args.profile_details) as p:
1073:         if args.export_aot_inductor:
1074:             frozen_model_iter_fn = export_aot_inductor(
1075:                 model, example_inputs, args.inductor_compile_mode
1076:             )
1077:         elif args.export_nativert:
1078:             frozen_model_iter_fn = export_nativert(model, example_inputs)
1079:         elif args.torchscript_jit_trace:
1080:             frozen_model_iter_fn = torchscript_jit_trace(model, example_inputs)
1081:         elif args.aot_precompile:
1082:             frozen_model_iter_fn = aot_precompile(model, example_inputs)
1083:         else:
1084:             if kwargs["hf_llm"]:
1085:                 # If it's an llm, we want to optimize model.forward, and use
1086:                 # the generate function
1087:                 model.forward = torch._dynamo.run(model)
1088:                 frozen_model_iter_fn = model_iter_fn
1089:             else:
1090:                 frozen_model_iter_fn = torch._dynamo.run(model_iter_fn)
1091: 
1092:         for rep in trange(args.repeat, desc="running benchmark"):
1093:             inputs = (
1094:                 randomize_input(copy.deepcopy(example_inputs))
1095:                 if should_randomize_input
1096:                 else example_inputs
1097:             )
1098:             # need call mark_step to perform the computation
1099:             # on randomize_input. Otherwise the first call using the
1100:             # inputs will incur high penalty then the next one.
1101:             maybe_mark_step(args)
1102: 
1103:             # interleave the runs to handle frequency scaling and load changes
1104:             with (
1105:                 maybe_mark_profile(p=p, mark="expected"),
1106:                 torch.compiler.set_stance("force_eager"),
1107:             ):
1108:                 timings[rep, 0], expected_output = timed(
1109:                     model,
1110:                     model_iter_fn,
1111:                     inputs,
1112:                     return_result=True,
1113:                     times=times,
1114:                     collect_outputs=args.collect_outputs,
1115:                     batch_size=kwargs.get("batch_size"),
1116:                 )
1117: 
1118:             # call mark_step between the 2 calls to make the comparison fair.
1119:             maybe_mark_step(args)
1120: 
````
- EN: Handles module imports such as `contextlib`, `torch._inductor.utils`.
- CN: 处理模块导入，例如 `contextlib`, `torch._inductor.utils`。
- EN: Implements callable logic such as `speedup_experiment`, `maybe_mark_profile`.
- CN: 实现可调用逻辑，例如 `speedup_experiment`, `maybe_mark_profile`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 1121-1200
````python
1121:             with maybe_mark_profile(p=p, mark="actual"):
1122:                 timings[rep, 1], actual_output = timed(
1123:                     model,
1124:                     frozen_model_iter_fn,
1125:                     inputs,
1126:                     return_result=True,
1127:                     times=times,
1128:                     collect_outputs=args.collect_outputs,
1129:                 )
1130: 
1131:     if args.export_profiler_trace:
1132:         name = args.profiler_trace_name + "_" + model.name
1133:         if hasattr(args, "rank"):
1134:             name += f"_rank_{args.rank}"
1135:         if args.export_perfdoctor and trace_handler:
1136:             trace_handler(name, p)
1137:         else:
1138:             name += ".json"
1139:             name = os.path.join(torch._dynamo.config.base_dir, name)
1140:             p.export_chrome_trace(name)
1141: 
1142:     median = np.median(timings, axis=0)
1143:     speedup = median[0] / median[1]
1144:     if args.dump_raw_metrics:
1145:         np.save(
1146:             f"{output_filename[:-4]}-raw_timings-{current_name}-{current_device}.npy",
1147:             timings,
1148:         )
1149: 
1150:     first_headers = ["dev", "name", "batch_size"]
1151:     first_fields = [current_device, current_name, current_batch_size]
1152:     if "tag" in kwargs:
1153:         first_headers.append("tag")
1154:         first_fields.append(kwargs["tag"])
1155:     headers = first_headers + ["speedup", "abs_latency"]
1156:     row = first_fields + [float(speedup), median[1] * 1000]
1157:     msg = f"{speedup:.3f}x"
1158:     if getattr(args, "_print_latency_ms", False):
1159:         msg = f"{median[0] * 1000:.4f} ms, {median[1] * 1000:.4f} ms, {msg}"
1160:     if args.baseline:
1161:         headers.extend(
1162:             [
1163:                 "baseline",
1164:                 "speedup_vs_baseline",
1165:             ]
1166:         )
1167:         df = pd.read_csv(args.baseline)
1168:         try:
1169:             baseline_speedup = df[df["name"] == current_name]["speedup"].item()
1170:             row.extend([baseline_speedup, speedup / baseline_speedup])
1171:             msg = f"{baseline_speedup:.3f}x -> {speedup:.3f}x [{speedup / baseline_speedup:.3f}x]"
1172:         except (KeyError, ZeroDivisionError):
1173:             row.extend(
1174:                 [
1175:                     0.0,
1176:                     0.0,
1177:                 ]
1178:             )
1179:     if "compilation_latency" in kwargs:
1180:         headers += [
1181:             "compilation_latency",
1182:             "compression_ratio",
1183:             "eager_peak_mem",
1184:             "dynamo_peak_mem",
1185:         ]
1186:         row.append(kwargs["compilation_latency"])
1187:         row.append(kwargs["compression_ratio"])
1188:         row.append(kwargs["eager_peak_mem"])
1189:         row.append(kwargs["dynamo_peak_mem"])
1190: 
1191:     if "cache_lookup_latency" in kwargs:
1192:         headers.append("cache_lookup_latency")
1193:         row.append(kwargs["cache_lookup_latency"])
1194: 
1195:     if "dynamo_stats" in kwargs:
1196:         for k, v in kwargs["dynamo_stats"].items():
1197:             headers.append(k)
1198:             row.append(v)
1199:     write_outputs(
1200:         output_filename,
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 1201-1280
````python
1201:         headers,
1202:         row,
1203:     )
1204:     c_headers, c_data = torch._dynamo.utils.compile_times(repr="csv", aggregate=True)
1205:     if output_filename.find(".csv") <= 0:
1206:         raise AssertionError(
1207:             f"expected output_filename to be a .csv, but got {output_filename}"
1208:         )
1209:     write_outputs(
1210:         output_filename[:-4] + "_compilation_metrics.csv",
1211:         first_headers + c_headers,
1212:         first_fields + c_data,
1213:     )
1214: 
1215:     output_signpost(
1216:         dict(zip(headers, row)),
1217:         args,
1218:         get_suite_from_model_iter_fn(model_iter_fn),
1219:     )
1220: 
1221:     return msg
1222: 
1223: 
1224: def overhead_experiment(*args, model_iter_fn):
1225:     """
1226:     Measure overheads of TorchDynamo by running with no backend (only
1227:     eager+FX), and reporting speedup/slowdown over eager.
1228: 
1229:     Writes to ./overheads.csv
1230:     """
1231:     return speedup_experiment(*args, model_iter_fn)
1232: 
1233: 
1234: def print_fx(gm, example_inputs):
1235:     print(gm.graph)
1236:     return gm
1237: 
1238: 
1239: def print_aten_ops(gm, example_inputs):
1240:     from functorch.compile import aot_module
1241: 
1242:     def trace_printer(gm, _):
1243:         print(gm.graph)
1244:         return gm
1245: 
1246:     return aot_module(gm, fw_compiler=trace_printer, bw_compiler=trace_printer)
1247: 
1248: 
1249: def baselines(models, model_iter_fn, example_inputs, args):
1250:     """
1251:     Common measurement code across all baseline experiments.
1252:     """
1253:     models = list(models)
1254:     for idx, (name, model) in enumerate(models):
1255:         if idx == 0:
1256:             result0 = model_iter_fn(model, example_inputs)
1257:         elif model is not None:
1258:             try:
1259:                 result = model_iter_fn(model, example_inputs)
1260:                 if same(result0, result):
1261:                     continue
1262:                 print(name, "is INCORRECT")
1263:             except Exception:
1264:                 log.exception("error checking %s", name)
1265:             models[idx] = (name, None)
1266:     timings = np.zeros((args.repeat, len(models)), np.float64)
1267:     timings.fill(1.0e10)
1268:     for rep in range(args.repeat):
1269:         for idx, (name, model) in enumerate(models):
1270:             if model is not None:
1271:                 try:
1272:                     timings[rep, idx] = timed(model, model_iter_fn, example_inputs)
1273:                 except Exception:
1274:                     pass
1275:     pvalue = [
1276:         ttest_ind(timings[:, 0], timings[:, i]).pvalue
1277:         for i in range(1, timings.shape[1])
1278:     ]
1279:     median = np.median(timings, axis=0)
1280:     speedup = median[0] / median[1:]
````
- EN: Handles module imports such as `functorch.compile`.
- CN: 处理模块导入，例如 `functorch.compile`。
- EN: Implements callable logic such as `overhead_experiment`, `print_fx`, `print_aten_ops`, `trace_printer`.
- CN: 实现可调用逻辑，例如 `overhead_experiment`, `print_fx`, `print_aten_ops`, `trace_printer`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 1281-1360
````python
1281:     for idx, (name, model) in enumerate(models[1:]):
1282:         if model is None:
1283:             speedup[idx] = 0.0
1284:     result = " ".join(
1285:         [
1286:             format_speedup(s, p, m is not None)
1287:             for s, p, m in zip(speedup, pvalue, [m for n, m in models[1:]])
1288:         ]
1289:     )
1290:     write_outputs(
1291:         output_filename,
1292:         ("dev", "name", "batch_size") + tuple(n for n, m in models[1:]),
1293:         [current_device, current_name, current_batch_size]
1294:         + [f"{x:.4f}" for x in speedup],
1295:     )
1296:     return result
1297: 
1298: 
1299: def xla(args, model_iter_fn, model, example_inputs):
1300:     xla_dev = xm.xla_device(devkind=current_device)
1301:     model_xla = copy.deepcopy(model).to("cpu").to(device=xla_dev)
1302:     example_inputs_xla = tree_map_only(
1303:         torch.Tensor, lambda x: x.to("cpu").to(device=xla_dev), example_inputs
1304:     )
1305:     for _ in range(3):  # warmup
1306:         timed(model, model_iter_fn, example_inputs)
1307:         timed(model_xla, model_iter_fn, example_inputs_xla)
1308:     timings = np.zeros((args.repeat, 2), np.float64)
1309:     timings.fill(1.0e10)
1310:     for rep in range(args.repeat):
1311:         timings[rep, 0] = timed(model, model_iter_fn, example_inputs)
1312:         timings[rep, 1] = timed(model_xla, model_iter_fn, example_inputs_xla)
1313: 
1314:     pvalue = ttest_ind(timings[:, 0], timings[:, 1]).pvalue
1315:     time_baseline, time_xla = np.median(timings, axis=0)
1316:     speedup = time_baseline / time_xla
1317:     write_outputs(
1318:         output_filename,
1319:         ("dev", "name", "batch_size", "speedup", "time_baseline", "time_xla"),
1320:         [
1321:             current_device,
1322:             current_name,
1323:             current_batch_size,
1324:             speedup,
1325:             time_baseline,
1326:             time_xla,
1327:         ],
1328:     )
1329:     return format_speedup(speedup, pvalue)
1330: 
1331: 
1332: def try_script(model, example_inputs):
1333:     try:
1334:         return torch.jit.script(model)
1335:     except Exception:
1336:         return None
1337: 
1338: 
1339: def _produce_dynamic_shapes_for_export(path, x):
1340:     # mark_dynamic() is ignored for export.
1341:     # use this to produce dynamic_shapes spec instead.
1342:     from torch.export.dynamic_shapes import Dim
1343: 
1344:     if not isinstance(x, torch.Tensor):
1345:         return None
1346:     return dict.fromkeys(getattr(x, "_dynamo_dynamic_indices", {}), Dim.AUTO)
1347: 
1348: 
1349: class AOTInductorModelCache:
1350:     cache: dict[weakref.ref, tuple[Any, float]] = {}
1351: 
1352:     @classmethod
1353:     def load(cls, model, example_inputs, mode):
1354:         import torch._inductor
1355:         from torch.export.dynamic_shapes import _combine_args, _tree_map_with_path
1356: 
1357:         key = weakref.ref(model)
1358:         if key not in cls.cache:
1359:             # Register the output dataclass to pytree
1360:             example_args, example_kwargs = _normalize_bench_inputs(example_inputs)
````
- EN: Handles module imports such as `torch.export.dynamic_shapes`, `torch._inductor`.
- CN: 处理模块导入，例如 `torch.export.dynamic_shapes`, `torch._inductor`。
- EN: Declares or extends types including `AOTInductorModelCache`.
- CN: 声明或扩展类型，包括 `AOTInductorModelCache`。
- EN: Implements callable logic such as `xla`, `try_script`, `_produce_dynamic_shapes_for_export`, `load`.
- CN: 实现可调用逻辑，例如 `xla`, `try_script`, `_produce_dynamic_shapes_for_export`, `load`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 1361-1440
````python
1361:             with torch.no_grad():
1362:                 # copy.deepcopy is required to prevent any surprising side-effect,
1363:                 # see https://github.com/pytorch/pytorch/issues/113029
1364:                 # This will cause memory stats to be overshadowed by this eager run.
1365:                 # To fix that, memory stats will be reset later.
1366:                 example_outputs = copy.deepcopy(model)(*example_args, **example_kwargs)
1367: 
1368:             if pytree.is_namedtuple_instance(example_outputs):
1369:                 typ = type(example_outputs)
1370:                 pytree._register_namedtuple(
1371:                     typ,
1372:                     serialized_type_name=f"{typ.__module__}.{typ.__name__}",
1373:                 )
1374:             else:
1375:                 _register_dataclass_output_as_pytree(example_outputs)
1376: 
1377:             combined_args = _combine_args(model, example_args, example_kwargs)
1378:             dynamic_shapes = _tree_map_with_path(
1379:                 _produce_dynamic_shapes_for_export, combined_args
1380:             )
1381: 
1382:             # delete example_outputs and reset memory stats here
1383:             del example_outputs
1384:             if current_device == "cuda":
1385:                 empty_gpu_cache(current_device)
1386:                 torch.cuda.reset_peak_memory_stats()
1387:                 pre_clone_memory_used = torch.cuda.max_memory_allocated()
1388:             elif current_device == "hpu":
1389:                 torch.hpu.reset_peak_memory_stats()
1390:                 pre_clone_memory_used = torch.hpu.max_memory_allocated()
1391: 
1392:             # Clone the model pre-exporting.  This prevents scenarios observed in a few
1393:             # models, where the forward pass modifies model state while exporting, and
1394:             # FakeTensors are thus saved as model data members.  This invalidates model
1395:             # reuse in eager mode, so it's safest to export a model clone.
1396:             model_clone = copy.deepcopy(model)
1397: 
1398:             # Since CPU doesn't monitor max memory allocation, anything measuring peak
1399:             # memory will miss our transient model clone on CPU anyway.
1400:             #
1401:             # The justification for tracking this value (in order to remove it from the
1402:             # AOTInductor memory measurements) is that normal usage of AOTInductor would
1403:             # not clone the model, since the eager model would be unused post-export.
1404:             clone_memory_used = 0.0
1405:             if current_device == "cuda":
1406:                 clone_memory_used = (
1407:                     torch.cuda.max_memory_allocated() - pre_clone_memory_used
1408:                 ) / 1e9
1409:             elif current_device == "hpu":
1410:                 clone_memory_used = (
1411:                     torch.hpu.max_memory_allocated() - pre_clone_memory_used
1412:                 ) / 1e9
1413: 
1414:             inductor_configs = {}
1415:             if mode == "max-autotune":
1416:                 inductor_configs["max_autotune"] = True
1417:             ep = torch.export.export(
1418:                 model_clone,
1419:                 example_args,
1420:                 example_kwargs,
1421:                 dynamic_shapes=dynamic_shapes,
1422:                 strict=False,
1423:             )
1424:             with torch.no_grad():
1425:                 package_path = torch._inductor.aoti_compile_and_package(
1426:                     ep, inductor_configs=inductor_configs
1427:                 )  # type: ignore[arg-type]
1428: 
1429:             cls.cache[key] = (
1430:                 torch._inductor.aoti_load_package(package_path),
1431:                 clone_memory_used,
1432:             )
1433: 
1434:         return cls.cache[key][0]
1435: 
1436:     @classmethod
1437:     def get_excess_memory(cls, model) -> float:
1438:         return cls.cache.get(weakref.ref(model), (None, 0.0))[1]
1439: 
1440: 
````
- EN: Implements callable logic such as `get_excess_memory`.
- CN: 实现可调用逻辑，例如 `get_excess_memory`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 1441-1520
````python
1441: class NativeRTCache:
1442:     cache: dict[weakref.ref, Any] = {}
1443: 
1444:     @classmethod
1445:     def load(cls, model, example_inputs):
1446:         from torch.export.dynamic_shapes import _combine_args, _tree_map_with_path
1447: 
1448:         key = weakref.ref(model)
1449:         if key not in cls.cache:
1450:             example_args, example_kwargs = _normalize_bench_inputs(example_inputs)
1451:             example_outputs = model(*example_args, **example_kwargs)
1452:             _register_dataclass_output_as_pytree(example_outputs)
1453: 
1454:             combined_args = _combine_args(model, example_args, example_kwargs)
1455:             dynamic_shapes = _tree_map_with_path(
1456:                 _produce_dynamic_shapes_for_export, combined_args
1457:             )
1458: 
1459:             ep = torch.export.export(
1460:                 model, example_args, example_kwargs, dynamic_shapes=dynamic_shapes
1461:             )
1462:             ep = ep.run_decompositions({})
1463:             with tempfile.NamedTemporaryFile(delete=False) as f:
1464:                 torch.export.pt2_archive._package.package_pt2(
1465:                     f, exported_programs={"forward": ep}
1466:                 )
1467:                 filename = f.name
1468:             cls.cache[key] = PyModelRunner(filename, "forward")
1469: 
1470:         return cls.cache[key]
1471: 
1472: 
1473: class JitTracedCache:
1474:     cache: dict[weakref.ref, Any] = {}
1475: 
1476:     @classmethod
1477:     def load(cls, model, example_inputs):
1478:         key = weakref.ref(model)
1479:         if key not in cls.cache:
1480:             example_args, example_kwargs = _normalize_bench_inputs(example_inputs)
1481:             if example_args:
1482:                 jit_traced_module = torch.jit.trace(
1483:                     model, example_inputs=example_args, strict=False
1484:                 )
1485:             else:
1486:                 jit_traced_module = torch.jit.trace(
1487:                     model, example_kwarg_inputs=example_kwargs, strict=False
1488:                 )
1489: 
1490:             cls.cache[key] = jit_traced_module
1491: 
1492:         return cls.cache[key]
1493: 
1494: 
1495: def export(model, example_inputs):
1496:     from torch.export.dynamic_shapes import _combine_args, _tree_map_with_path
1497: 
1498:     example_args, example_kwargs = _normalize_bench_inputs(example_inputs)
1499:     example_outputs = model(*example_args, **example_kwargs)
1500:     _register_dataclass_output_as_pytree(example_outputs)
1501: 
1502:     combined_args = _combine_args(model, example_args, example_kwargs)
1503:     dynamic_shapes = _tree_map_with_path(
1504:         _produce_dynamic_shapes_for_export, combined_args
1505:     )
1506: 
1507:     # NOTE: if args.export is ever enabled for --performance mode (rather than solely
1508:     # --accuracy), we'll need to clone the model and subtract out extra memory usage, as
1509:     # done in AOTInductorModelCache.
1510:     ep = torch.export.export(
1511:         model, example_args, example_kwargs, dynamic_shapes=dynamic_shapes, strict=True
1512:     )
1513: 
1514:     def opt_export(_, example_inputs):
1515:         example_args, example_kwargs = _normalize_bench_inputs(example_inputs)
1516:         return ep.module()(*example_args, **example_kwargs)
1517: 
1518:     return opt_export
1519: 
1520: 
````
- EN: Handles module imports such as `torch.export.dynamic_shapes`.
- CN: 处理模块导入，例如 `torch.export.dynamic_shapes`。
- EN: Declares or extends types including `NativeRTCache`, `JitTracedCache`.
- CN: 声明或扩展类型，包括 `NativeRTCache`, `JitTracedCache`。
- EN: Implements callable logic such as `load`, `export`, `opt_export`.
- CN: 实现可调用逻辑，例如 `load`, `export`, `opt_export`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 1521-1600
````python
1521: def aot_precompile(model, example_inputs):
1522:     example_args, example_kwargs = _normalize_bench_inputs(example_inputs)
1523: 
1524:     with tempfile.NamedTemporaryFile(suffix=".pt", delete=False) as f:
1525:         save_path = f.name
1526: 
1527:     with fresh_cache(), torch._dynamo.config.patch("enable_aot_compile", True):
1528:         compiled_fn = torch.compile(
1529:             model,
1530:             fullgraph=True,
1531:             options={"guard_filter_fn": lambda guards: [False for _ in guards]},
1532:         ).forward.aot_compile((example_args, example_kwargs))
1533: 
1534:         compiled_fn.save_compiled_function(save_path)
1535: 
1536:         torch._dynamo.reset()
1537:         with open(save_path, "rb") as f:
1538:             load_start_time = time.perf_counter()
1539:             loaded_fn = torch.compiler.load_compiled_function(f)
1540:             load_end_time = time.perf_counter()
1541:             print(
1542:                 f"AOT Precompile loading time: {load_end_time - load_start_time} seconds"
1543:             )
1544: 
1545:             def opt_aot_precompile(_, example_inputs, collect_outputs=False):
1546:                 example_args, example_kwargs = _normalize_bench_inputs(example_inputs)
1547:                 return loaded_fn(model, *example_args, **example_kwargs)
1548: 
1549:             return opt_aot_precompile
1550: 
1551: 
1552: def export_nativert(model, example_inputs):
1553:     optimized = NativeRTCache.load(model, example_inputs)
1554: 
1555:     def opt_nativert(_, example_inputs, collect_outputs=False):
1556:         example_args, example_kwargs = _normalize_bench_inputs(example_inputs)
1557:         return optimized.run(*example_args, **example_kwargs)
1558: 
1559:     return opt_nativert
1560: 
1561: 
1562: def export_aot_inductor(model, example_inputs, mode):
1563:     optimized = AOTInductorModelCache.load(model, example_inputs, mode)
1564: 
1565:     def opt_aot_inductor(_, example_inputs, collect_outputs=False):
1566:         example_args, example_kwargs = _normalize_bench_inputs(example_inputs)
1567:         return optimized(*example_args, **example_kwargs)
1568: 
1569:     return opt_aot_inductor
1570: 
1571: 
1572: def torchscript_jit_trace(model, example_inputs):
1573:     optimized = JitTracedCache.load(model, example_inputs)
1574: 
1575:     def opt_jit_trace(_, example_inputs, collect_outputs=False):
1576:         example_args, example_kwargs = _normalize_bench_inputs(example_inputs)
1577:         return optimized(*example_args, **example_kwargs)
1578: 
1579:     return opt_jit_trace
1580: 
1581: 
1582: def download_retry_decorator(download_fn):
1583:     """
1584:     Decorator function for applying retry logic to a download function.
1585: 
1586:     The wrapped function will be called up to 5 times and raises an exception if the function fails each time.
1587:     After each unsuccessful attempt, there is a delay before the next attempt, which is increased linearly with the number of tries.
1588: 
1589:     Usage:
1590:     @download_retry_decorator
1591:     def download_function(model_name: str):
1592:         # download logic goes here
1593:     """
1594: 
1595:     @functools.wraps(download_fn)
1596:     def wrapper(self, *args, **kwargs) -> Any:
1597:         tries = 0
1598:         total_allowed_tries = MAX_DOWNLOAD_ATTEMPTS
1599:         while tries <= total_allowed_tries:
1600:             try:
````
- EN: Implements callable logic such as `aot_precompile`, `opt_aot_precompile`, `export_nativert`, `opt_nativert`.
- CN: 实现可调用逻辑，例如 `aot_precompile`, `opt_aot_precompile`, `export_nativert`, `opt_nativert`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 1601-1680
````python
1601:                 model = download_fn(self, *args, **kwargs)
1602:                 return model
1603:             except Exception as e:
1604:                 tries += 1
1605:                 if tries <= total_allowed_tries:
1606:                     wait = tries * 30
1607:                     print(
1608:                         f"Failed to load model: {e}. Trying again ({tries}/{total_allowed_tries}) after {wait}s"
1609:                     )
1610:                     time.sleep(wait)
1611:                 else:
1612:                     raise RuntimeError(  # noqa: B904
1613:                         f"Failed to load model '{args}' with following error(s): {str(e)}."
1614:                     )
1615: 
1616:     return wrapper
1617: 
1618: 
1619: def read_batch_size_from_file(args, filename, model_name):
1620:     batch_size = None
1621:     if os.path.exists("benchmarks"):
1622:         filename = os.path.join("benchmarks", filename)
1623:     if not os.path.exists(filename):
1624:         raise AssertionError(f"file not found: {filename}")
1625:     with open(filename) as f:
1626:         lines = f.readlines()
1627:         lines = [i.split(",") for i in lines if len(i.strip()) > 0]
1628:         for val in lines:
1629:             cur_name, b = val
1630:             if model_name == cur_name:
1631:                 batch_size = int(b)
1632:     if batch_size is None:
1633:         log.warning("Could not find batch size for %s", model_name)
1634:     elif batch_size == -1:
1635:         raise RuntimeError(
1636:             f"Batch size is unset for {model_name} in {args.batch_size_file}"
1637:         )
1638:     print(f"batch size: {batch_size}")
1639:     return batch_size
1640: 
1641: 
1642: class TimeOutException(Exception):
1643:     pass
1644: 
1645: 
1646: def alarm_handler(signum, frame):
1647:     raise TimeOutException
1648: 
1649: 
1650: def exit_after(s):
1651:     """
1652:     Decorator to raise TimeoutException if the fn is taking more than s seconds
1653:     to run.
1654:     """
1655: 
1656:     def outer(fn):
1657:         def inner(*args, **kwargs):
1658:             signal.signal(signal.SIGALRM, alarm_handler)
1659:             signal.alarm(s)
1660:             try:
1661:                 result = fn(*args, **kwargs)
1662:             finally:
1663:                 signal.alarm(0)
1664:             return result
1665: 
1666:         return inner
1667: 
1668:     return outer
1669: 
1670: 
1671: def get_peak_memory():
1672:     return torch.cuda.max_memory_allocated() / 10**9
1673: 
1674: 
1675: def null_experiment(args, model_iter_fn, model, example_inputs):
1676:     """
1677:     A no-op experiment useful for making sure TorchBenchark alone works properly.
1678:     """
1679: 
1680:     return []
````
- EN: Declares or extends types including `TimeOutException`.
- CN: 声明或扩展类型，包括 `TimeOutException`。
- EN: Implements callable logic such as `read_batch_size_from_file`, `alarm_handler`, `exit_after`, `outer`.
- CN: 实现可调用逻辑，例如 `read_batch_size_from_file`, `alarm_handler`, `exit_after`, `outer`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 1681-1760
````python
1681: 
1682: 
1683: def cast_to(dtype, model, inputs):
1684:     # cast model and inputs to fp16
1685:     if dtype == torch.float16:
1686:         model = model.half()
1687:     else:
1688:         model = model.to(dtype)
1689: 
1690:     def cast_and_preserve_markings(x):
1691:         if not isinstance(x, torch.Tensor) or not x.is_floating_point():
1692:             return x
1693:         y = x.to(dtype)
1694:         # Preserve dynamic/unbacked markings
1695:         copy_dynamo_tensor_attributes(x, y)
1696:         return y
1697: 
1698:     inputs = tree_map(cast_and_preserve_markings, inputs)
1699:     return model, inputs
1700: 
1701: 
1702: def cast_to_bf16(model, inputs):
1703:     return cast_to(torch.bfloat16, model, inputs)
1704: 
1705: 
1706: def cast_to_fp16(model, inputs):
1707:     return cast_to(torch.float16, model, inputs)
1708: 
1709: 
1710: def cast_to_fp64(model, inputs):
1711:     return cast_to(torch.float64, model, inputs)
1712: 
1713: 
1714: def cast_to_fp32(model, inputs):
1715:     return cast_to(torch.float32, model, inputs)
1716: 
1717: 
1718: class DummyGradScaler:
1719:     def scale(self, loss):
1720:         return loss
1721: 
1722: 
1723: def get_dynamo_stats():
1724:     # TODO: consider deepcopy'ing the entire counters struct and
1725:     # adding a helper to do subtraction on it
1726:     return collections.Counter(
1727:         {
1728:             "calls_captured": torch._dynamo.utils.counters["stats"]["calls_captured"],
1729:             "unique_graphs": torch._dynamo.utils.counters["stats"]["unique_graphs"],
1730:             "graph_breaks": sum(torch._dynamo.utils.counters["graph_break"].values()),
1731:             # NB: The plus removes zero counts
1732:             "unique_graph_breaks": len(+torch._dynamo.utils.counters["graph_break"]),
1733:             "autograd_captures": torch._dynamo.utils.counters["compiled_autograd"][
1734:                 "captures"
1735:             ],
1736:             "autograd_compiles": torch._dynamo.utils.counters["compiled_autograd"][
1737:                 "compiles"
1738:             ],
1739:             "cudagraph_skips": torch._dynamo.utils.counters["inductor"][
1740:                 "cudagraph_skips"
1741:             ],
1742:         }
1743:     )
1744: 
1745: 
1746: @contextmanager
1747: def maybe_init_distributed(should_init_distributed, rank, world_size, port="6789"):
1748:     try:
1749:         if should_init_distributed:
1750:             torch.cuda.set_device(rank)
1751:             os.environ["MASTER_ADDR"] = "localhost"
1752:             os.environ["MASTER_PORT"] = port
1753:             torch.distributed.init_process_group(
1754:                 "nccl", rank=rank, world_size=world_size
1755:             )
1756:         yield
1757:     finally:
1758:         if should_init_distributed:
1759:             torch.distributed.destroy_process_group()
1760: 
````
- EN: Declares or extends types including `DummyGradScaler`, `and`.
- CN: 声明或扩展类型，包括 `DummyGradScaler`, `and`。
- EN: Implements callable logic such as `cast_to`, `cast_and_preserve_markings`, `cast_to_bf16`, `cast_to_fp16`.
- CN: 实现可调用逻辑，例如 `cast_to`, `cast_and_preserve_markings`, `cast_to_bf16`, `cast_to_fp16`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 1761-1840
````python
1761: 
1762: @contextmanager
1763: def maybe_snapshot_memory(should_snapshot_memory, suffix):
1764:     # Enables Memory Snapshot tool for memory deep dives:
1765:     # https://pytorch.org/blog/understanding-gpu-memory-1/
1766:     try:
1767:         if should_snapshot_memory:
1768:             torch.cuda.memory._record_memory_history(max_entries=100000)
1769:         yield
1770:     finally:
1771:         if should_snapshot_memory:
1772:             try:
1773:                 torch.cuda.memory._dump_snapshot(
1774:                     os.path.join(
1775:                         torch._dynamo.config.base_dir,
1776:                         f"{output_filename.rstrip('.csv')}_{suffix}.pickle",
1777:                     )
1778:                 )
1779:             except Exception:
1780:                 log.exception("Failed to save memory snapshot")
1781: 
1782:             torch.cuda.memory._record_memory_history(enabled=None)
1783: 
1784: 
1785: class BenchmarkRunner:
1786:     def __init__(self):
1787:         self.model_iter_fn = None
1788:         self.grad_scaler = DummyGradScaler()
1789:         self.autocast = contextlib.nullcontext
1790:         self.autocast_arg = {}
1791:         self.optimizer: torch.optim.Optimizer | None = None
1792:         self._args = None
1793: 
1794:     def setup_amp(self, current_device=None):
1795:         if self.args.only in self.fp32_only_models:
1796:             return
1797: 
1798:         devices = [current_device] if current_device else self.args.devices
1799:         if self.args.amp:
1800:             # AMP training can lead to small loss values which can underflow
1801:             # gradient values returning in zero gradients. To solve this
1802:             # problem, PyTorch introduces GradScaler. GradScaler is a stateful
1803:             # structure, that scales the loss values to prevent underflow. Loss
1804:             # values are big at the beginning of training (therefore not
1805:             # requiring scaling), while loss value tends to be small as network
1806:             # starts getting better (requiring scaling). GradScaler manages all
1807:             # of this fine tuning, checking the gradients are turning to inf,
1808:             # discarding such batches.
1809: 
1810:             # Since we are not running a long iteration, default value of
1811:             # init_scale 65536 is going to turn all gradients to inf. Therefore,
1812:             # we just use a init_scale of 2.0 for benchmarking purpose.
1813: 
1814:             # Disabling Gradscaler because
1815:             #  1) Benchmark setup runs 2 iterations of fwd-bwd. So, not useful.
1816:             #  2) Current setup shares grad_scaler for eager and dynamo model,
1817:             #  which is bad as Gradscaler has state and can adjust the scaling
1818:             #  factor between eager and dynamo run, making accuracy check
1819:             #  harder.
1820:             # self.grad_scaler = torch.amp.GradScaler(device="cuda", init_scale=2.0)
1821:             self.autocast = functools.partial(
1822:                 torch.amp.autocast, device_type=devices[0]
1823:             )
1824:             if self.args.amp_dtype is None:
1825:                 if self.args.only in self.amp_dtype_bfloat16:
1826:                     self.autocast_arg["dtype"] = torch.bfloat16
1827:             else:
1828:                 amp_dtype = (
1829:                     torch.float16
1830:                     if self.args.amp_dtype == "float16"
1831:                     else torch.bfloat16
1832:                 )
1833:                 self.autocast_arg["dtype"] = amp_dtype
1834: 
1835:     def init_optimizer(self, name, device, params):
1836:         if device == "cuda" and self.args.training and name not in CI_SKIP_OPTIMIZER:
1837:             if (name in CI_USE_SGD and self.args.ci) or name in BENCHMARK_USE_SGD:
1838:                 self.optimizer = torch.optim.SGD(params, lr=0.01, foreach=True)
1839:                 # Disable multi_tensor_sgd for benchmarking, there isn't a large performance benefit (~1%) to compiling
1840:                 # this optimizer because it is a single foreach add, and increases compile time.
````
- EN: Declares or extends types including `BenchmarkRunner`.
- CN: 声明或扩展类型，包括 `BenchmarkRunner`。
- EN: Implements callable logic such as `maybe_snapshot_memory`, `__init__`, `setup_amp`, `init_optimizer`.
- CN: 实现可调用逻辑，例如 `maybe_snapshot_memory`, `__init__`, `setup_amp`, `init_optimizer`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 1841-1920
````python
1841:                 # After autotuning and fake tensor caching lands, we can enable, because the compile time impact will be lower.
1842:                 # Fake Tensor caching: https://github.com/pytorch/pytorch/pull/113873
1843:                 # Autotuning: https://github.com/pytorch/pytorch/issues/117447
1844:                 self.optimizer.step = torch._dynamo.disable(self.optimizer.step)
1845:             else:
1846:                 self.optimizer = torch.optim.Adam(
1847:                     params, lr=0.01, capturable=True, foreach=True
1848:                 )
1849:         else:
1850:             self.optimizer = None
1851: 
1852:     @property
1853:     def args(self):
1854:         return self._args
1855: 
1856:     @args.setter
1857:     def args(self, args):
1858:         self._args = args
1859: 
1860:     @property
1861:     def skip_models(self):
1862:         return set()
1863: 
1864:     @property
1865:     def skip_models_for_cuda(self):
1866:         return set()
1867: 
1868:     @property
1869:     def skip_models_for_xpu(self):
1870:         return set()
1871: 
1872:     @property
1873:     def skip_models_for_cpu(self):
1874:         return set()
1875: 
1876:     @property
1877:     def skip_models_for_cpu_aarch64(self):
1878:         return set()
1879: 
1880:     @property
1881:     def skip_models_for_freezing_cpu(self):
1882:         return set()
1883: 
1884:     @property
1885:     def skip_models_for_freezing_cuda(self):
1886:         return set()
1887: 
1888:     @property
1889:     def slow_models(self):
1890:         return set()
1891: 
1892:     @property
1893:     def very_slow_models(self):
1894:         return set()
1895: 
1896:     @property
1897:     def non_deterministic_models(self):
1898:         return set()
1899: 
1900:     @property
1901:     def fp32_only_models(self):
1902:         return set()
1903: 
1904:     @property
1905:     def force_amp_for_fp16_bf16_models(self):
1906:         return set()
1907: 
1908:     @property
1909:     def force_fp16_for_bf16_models(self):
1910:         return set()
1911: 
1912:     @property
1913:     def amp_dtype_bfloat16(self):
1914:         return set()
1915: 
1916:     @property
1917:     def skip_not_suitable_for_training_models(self):
1918:         return set()
1919: 
1920:     @property
````
- EN: Implements callable logic such as `args`, `skip_models`, `skip_models_for_cuda`, `skip_models_for_xpu`.
- CN: 实现可调用逻辑，例如 `args`, `skip_models`, `skip_models_for_cuda`, `skip_models_for_xpu`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 1921-2000
````python
1921:     def failing_torchinductor_models(self):
1922:         return set()
1923: 
1924:     @property
1925:     def failing_fx2trt_models(self):
1926:         return set()
1927: 
1928:     @property
1929:     def skip_accuracy_checks_large_models_dashboard(self):
1930:         return set()
1931: 
1932:     @property
1933:     def skip_accuracy_check_as_eager_non_deterministic(self):
1934:         return set()
1935: 
1936:     @property
1937:     def skip_multiprocess_models(self):
1938:         return set()
1939: 
1940:     @property
1941:     def skip_models_due_to_control_flow(self):
1942:         return set()
1943: 
1944:     @property
1945:     def skip_models_due_to_export_not_supported(self):
1946:         return set()
1947: 
1948:     @property
1949:     def disable_cudagraph_models(self):
1950:         return set()
1951: 
1952:     @property
1953:     def guard_on_nn_module_models(self):
1954:         return set()
1955: 
1956:     def get_tolerance_and_cosine_flag(self, is_training, current_device, name):
1957:         raise NotImplementedError
1958: 
1959:     @property
1960:     def equal_nan(self):
1961:         equal_nan = True
1962:         if self.args.float32:
1963:             equal_nan = False
1964:         return equal_nan
1965: 
1966:     def use_larger_multiplier_for_smaller_tensor(self, name):
1967:         return False
1968: 
1969:     def use_iou_for_bool_accuracy(self, name):
1970:         return False
1971: 
1972:     def get_iou_threshold(self, name):
1973:         return 0.99
1974: 
1975:     def get_accuracy_check_runs(self, name):
1976:         return 1
1977: 
1978:     def iter_models(self, args):
1979:         for model_name in self.iter_model_names(args):
1980:             for device in args.devices:
1981:                 try:
1982:                     yield self.load_model(
1983:                         device,
1984:                         model_name,
1985:                         batch_size=args.batch_size,
1986:                     )
1987:                 except NotImplementedError:
1988:                     continue  # bad benchmark implementation
1989: 
1990:     def deepcopy_model(self, model):
1991:         return copy.deepcopy(model)
1992: 
1993:     def cast_based_on_args(self, model, example_inputs):
1994:         if self.args.float32 or self.args.only in self.fp32_only_models:
1995:             if not self.args.float32:
1996:                 log.warning("Model %s supports float32 only", self.args.only)
1997:             model, example_inputs = cast_to_fp32(model, example_inputs)
1998:         elif self.args.float16:
1999:             if self.args.only in self.force_amp_for_fp16_bf16_models:
2000:                 log.warning(
````
- EN: Implements callable logic such as `failing_torchinductor_models`, `failing_fx2trt_models`, `skip_accuracy_checks_large_models_dashboard`, `skip_accuracy_check_as_eager_non_deterministic`.
- CN: 实现可调用逻辑，例如 `failing_torchinductor_models`, `failing_fx2trt_models`, `skip_accuracy_checks_large_models_dashboard`, `skip_accuracy_check_as_eager_non_deterministic`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 2001-2080
````python
2001:                     "Model %s does not support float16, running with amp instead",
2002:                     self.args.only,
2003:                 )
2004:                 self.args.amp = True
2005:                 self.setup_amp()
2006:             else:
2007:                 model, example_inputs = cast_to_fp16(model, example_inputs)
2008:         elif self.args.bfloat16:
2009:             if self.args.only in self.force_amp_for_fp16_bf16_models:
2010:                 log.warning(
2011:                     "Model %s does not support bfloat16, running with amp instead",
2012:                     self.args.only,
2013:                 )
2014:                 self.args.amp = True
2015:                 self.setup_amp()
2016:             elif self.args.only in self.force_fp16_for_bf16_models:
2017:                 log.warning(
2018:                     "Model %s does not support bfloat16, running with float16 instead",
2019:                     self.args.only,
2020:                 )
2021:                 model, example_inputs = cast_to_fp16(model, example_inputs)
2022:             else:
2023:                 model, example_inputs = cast_to_bf16(model, example_inputs)
2024: 
2025:         return model, example_inputs
2026: 
2027:     def validate_model(self, name, model, example_inputs):
2028:         """
2029:         Runs the eager model with example inputs to ensure that eager passes.
2030:         """
2031:         model = self.deepcopy_model(model)
2032:         example_inputs = clone_inputs(example_inputs)
2033:         model, example_inputs = self.cast_based_on_args(model, example_inputs)
2034:         with self.pick_grad(name, self.args.training):
2035:             try:
2036:                 self.model_iter_fn(model, example_inputs)
2037:             except Exception as e:
2038:                 raise RuntimeError("Eager run failed") from e
2039: 
2040:     def maybe_cast(self, model, example_inputs):
2041:         model, example_inputs = self.cast_based_on_args(model, example_inputs)
2042:         return model, example_inputs
2043: 
2044:     def decay_batch_exp(self, batch_size, factor=0.5, divisor=2):
2045:         out_batch_size = batch_size * factor
2046:         if out_batch_size > divisor:
2047:             out_batch_size = (out_batch_size + 1) // divisor * divisor
2048:         else:
2049:             out_batch_size = batch_size - 1
2050:         return max(0, int(out_batch_size))
2051: 
2052:     def batch_size_finder(self, device, model_name, initial_batch_size=1024):
2053:         batch_size = initial_batch_size
2054:         while batch_size >= 1:
2055:             empty_gpu_cache(current_device)
2056:             try:
2057:                 device, name, model, example_inputs, _ = self.load_model(
2058:                     device,
2059:                     model_name,
2060:                     batch_size,
2061:                 )
2062:                 self.model_iter_fn(model, example_inputs)
2063:                 return batch_size
2064:             except RuntimeError as e:
2065:                 error_str = str(e)
2066:                 if "channels_last" in error_str:
2067:                     break
2068:             batch_size = self.decay_batch_exp(batch_size)
2069:         return 1
2070: 
2071:     def run_n_iterations(self, mod, inputs, model_iter_fn):
2072:         n = self.args.iterations
2073:         for _ in range(n - 1):
2074:             model_iter_fn(mod, inputs, collect_outputs=False)
2075:         return model_iter_fn(mod, inputs, collect_outputs=True)
2076: 
2077:     @torch._disable_dynamo(recursive=True)
2078:     def optimizer_zero_grad(self, mod):
2079:         if self.optimizer is not None:
2080:             self.optimizer.zero_grad(True)
````
- EN: Implements callable logic such as `validate_model`, `maybe_cast`, `decay_batch_exp`, `batch_size_finder`.
- CN: 实现可调用逻辑，例如 `validate_model`, `maybe_cast`, `decay_batch_exp`, `batch_size_finder`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 2081-2160
````python
2081:         else:
2082:             mod.zero_grad(True)
2083: 
2084:     def optimizer_step(self):
2085:         if self.optimizer is not None:
2086:             self.optimizer.step()
2087: 
2088:     def get_benchmark_indices(self, length):
2089:         start = self._args.partition_id * (length // self._args.total_partitions)
2090:         end = (
2091:             (self._args.partition_id + 1) * (length // self._args.total_partitions)
2092:             if self._args.partition_id < self._args.total_partitions - 1
2093:             else length
2094:         )
2095:         return start, end
2096: 
2097:     def get_fsdp_auto_wrap_policy(self, model_name: str):
2098:         from diffusers.models.transformer_2d import Transformer2DModel
2099:         from torchbenchmark.models.nanogpt.model import Block
2100:         from transformers.models.llama.modeling_llama import LlamaDecoderLayer
2101: 
2102:         from torch.distributed.fsdp.wrap import (
2103:             ModuleWrapPolicy,
2104:             size_based_auto_wrap_policy,
2105:         )
2106: 
2107:         # handcrafted wrap policy
2108:         MODEL_FSDP_WRAP = {
2109:             "stable_diffusion_unet": (Transformer2DModel,),
2110:             "llama_v2_7b_16h": (LlamaDecoderLayer,),
2111:             "nanogpt": (Block,),
2112:         }
2113: 
2114:         if model_name not in MODEL_FSDP_WRAP:
2115:             # default to using wrap policy based on module size
2116:             return functools.partial(
2117:                 size_based_auto_wrap_policy, recurse=True, min_num_params=int(1e5)
2118:             )
2119: 
2120:         return ModuleWrapPolicy(MODEL_FSDP_WRAP[model_name])
2121: 
2122:     def deepcopy_and_maybe_parallelize(self, model):
2123:         model = self.deepcopy_model(model)
2124:         if self.args.ddp:
2125:             if not torch.distributed.is_available():
2126:                 raise AssertionError(
2127:                     "Can't use DDP without a distributed enabled build"
2128:                 )
2129:             from torch.nn.parallel import DistributedDataParallel as DDP
2130: 
2131:             model = DDP(model, find_unused_parameters=True)
2132:         elif self.args.fsdp:
2133:             if not torch.distributed.is_available():
2134:                 raise AssertionError(
2135:                     "Can't use FSDP without a distributed enabled build"
2136:                 )
2137:             from torch.distributed.fsdp import (
2138:                 FullyShardedDataParallel as FSDP,
2139:                 MixedPrecision,
2140:             )
2141: 
2142:             if self.args.float16:
2143:                 dtype = torch.float16
2144:             elif self.args.bfloat16:
2145:                 dtype = torch.bfloat16
2146:             else:
2147:                 dtype = torch.float32
2148: 
2149:             mp_policy = MixedPrecision(
2150:                 param_dtype=dtype,
2151:                 # Gradient communication precision.
2152:                 reduce_dtype=dtype,
2153:                 # Buffer precision.
2154:                 buffer_dtype=dtype,
2155:             )
2156: 
2157:             model = FSDP(
2158:                 model,
2159:                 use_orig_params=True,
2160:                 device_id=torch.cuda.current_device()
````
- EN: Handles module imports such as `diffusers.models.transformer_2d`, `torchbenchmark.models.nanogpt.model`, `transformers.models.llama.modeling_llama`, `torch.distributed.fsdp.wrap`.
- CN: 处理模块导入，例如 `diffusers.models.transformer_2d`, `torchbenchmark.models.nanogpt.model`, `transformers.models.llama.modeling_llama`, `torch.distributed.fsdp.wrap`。
- EN: Implements callable logic such as `optimizer_step`, `get_benchmark_indices`, `get_fsdp_auto_wrap_policy`, `deepcopy_and_maybe_parallelize`.
- CN: 实现可调用逻辑，例如 `optimizer_step`, `get_benchmark_indices`, `get_fsdp_auto_wrap_policy`, `deepcopy_and_maybe_parallelize`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 2161-2240
````python
2161:                 if self.args.devices[-1] == "cuda"
2162:                 else None,
2163:                 mixed_precision=mp_policy,
2164:                 limit_all_gathers=True,
2165:                 auto_wrap_policy=self.get_fsdp_auto_wrap_policy(self.args.only),
2166:             )
2167:         return model
2168: 
2169:     def _write_accuracy_row(self, status, dynamo_start_stats, tag):
2170:         """
2171:         Shared CSV + signpost writer for accuracy checks.
2172:         """
2173:         headers = ["dev", "name", "batch_size", "accuracy"]
2174:         fields = [current_device, current_name, current_batch_size, status]
2175: 
2176:         if tag is not None:
2177:             headers.insert(3, "tag")
2178:             fields.insert(3, tag)
2179: 
2180:         o_headers = list(headers)
2181:         o_fields = list(fields)
2182: 
2183:         dynamo_stats = get_dynamo_stats()
2184:         dynamo_stats.subtract(dynamo_start_stats)
2185:         for k, v in dynamo_stats.items():
2186:             headers.append(k)
2187:             fields.append(v)
2188: 
2189:         total_wall_time = output_signpost(
2190:             dict(zip(o_headers, o_fields)),
2191:             self.args,
2192:             self.suite_name,
2193:         )
2194:         headers.append("compilation_latency")
2195:         fields.append(total_wall_time)
2196:         write_outputs(output_filename, headers, fields)
2197: 
2198:         if self.args.print_compilation_time:
2199:             print(f"Compilation time (from dynamo_timed): {total_wall_time}")
2200: 
2201:     def check_accuracy(
2202:         self, name, model, example_inputs, optimize_ctx, experiment, tag
2203:     ):
2204:         """
2205:         Checks accuracy.
2206:         1) Collect the outputs with fp64 datatype. This is useful for error checking.
2207:         2) Checks if eager itself has variations.
2208:         """
2209:         start_stats = get_dynamo_stats()
2210: 
2211:         def record_status(accuracy_status, dynamo_start_stats):
2212:             """
2213:             Records the status in the csv file
2214:             """
2215:             if current_name in self.non_deterministic_models:
2216:                 if accuracy_status in (
2217:                     "pass",
2218:                     "eager_two_runs_differ",
2219:                     "fail_accuracy",
2220:                 ):
2221:                     accuracy_status = "pass"
2222: 
2223:             self._write_accuracy_row(accuracy_status, dynamo_start_stats, tag)
2224:             return accuracy_status
2225: 
2226:         if name in self.skip_accuracy_checks_large_models_dashboard:
2227:             return record_status("pass_due_to_skip", dynamo_start_stats=start_stats)
2228: 
2229:         # Skip all accuracy check for the torchao backend
2230:         if self.args.backend == "torchao":
2231:             return record_status("pass_due_to_skip", dynamo_start_stats=start_stats)
2232: 
2233:         with self.pick_grad(name, self.args.training):
2234:             # Collect the fp64 reference outputs to be used later for accuracy checking.
2235:             fp64_outputs = None
2236:             model_fp64 = None
2237:             inputs_fp64 = None
2238:             try:
2239:                 model_fp64, inputs_fp64 = cast_to_fp64(
2240:                     self.deepcopy_and_maybe_parallelize(model),
````
- EN: Implements callable logic such as `_write_accuracy_row`, `check_accuracy`, `record_status`.
- CN: 实现可调用逻辑，例如 `_write_accuracy_row`, `check_accuracy`, `record_status`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 2241-2320
````python
2241:                     clone_inputs(example_inputs),
2242:                 )
2243:                 self.init_optimizer(name, current_device, model_fp64.parameters())
2244:                 fp64_outputs = self.run_n_iterations(
2245:                     model_fp64, inputs_fp64, self.model_iter_fn
2246:                 )
2247:                 fp64_outputs = tree_map(
2248:                     lambda x: x.to(torch.float64)
2249:                     if isinstance(x, torch.Tensor) and x.is_floating_point()
2250:                     else x,
2251:                     fp64_outputs,
2252:                 )
2253:             except Exception:
2254:                 log.warning(
2255:                     "fp64 golden ref were not generated for %s. Setting accuracy check to cosine",
2256:                     name,
2257:                     exc_info=True,
2258:                 )
2259:                 self.args.cosine = True
2260:                 fp64_outputs = None
2261:             finally:
2262:                 del model_fp64, inputs_fp64
2263:                 empty_gpu_cache(current_device)
2264: 
2265:             tolerance, cos_similarity = self.get_tolerance_and_cosine_flag(
2266:                 self.args.training, current_device, name
2267:             )
2268: 
2269:             # Cast the model to float16/float32 as necessary
2270:             model, example_inputs = self.maybe_cast(model, example_inputs)
2271:             accuracy_status = "pass"
2272: 
2273:             # Get results of native pytorch
2274:             reset_rng_state()
2275:             model_copy = None
2276:             try:
2277:                 with torch.compiler.set_stance("force_eager"):
2278:                     model_copy = self.deepcopy_and_maybe_parallelize(model)
2279:                     self.init_optimizer(name, current_device, model_copy.parameters())
2280:                     correct_result = self.run_n_iterations(
2281:                         model_copy, clone_inputs(example_inputs), self.model_iter_fn
2282:                     )
2283:             except Exception as e:
2284:                 accuracy_status = (
2285:                     "eager_1st_run_OOM"
2286:                     if isinstance(e, torch.cuda.OutOfMemoryError)
2287:                     else "eager_1st_run_fail"
2288:                 )
2289:                 log.exception("")
2290:                 return record_status(accuracy_status, dynamo_start_stats=start_stats)
2291:             finally:
2292:                 del model_copy
2293:                 empty_gpu_cache(current_device)
2294: 
2295:             # Rerun native pytorch
2296:             reset_rng_state()
2297:             model_copy = None
2298:             try:
2299:                 with torch.compiler.set_stance("force_eager"):
2300:                     model_copy = self.deepcopy_and_maybe_parallelize(model)
2301:                     self.init_optimizer(name, current_device, model_copy.parameters())
2302:                     correct_rerun_result = self.run_n_iterations(
2303:                         model_copy, clone_inputs(example_inputs), self.model_iter_fn
2304:                     )
2305:             except Exception as e:
2306:                 accuracy_status = (
2307:                     "eager_2nd_run_OOM"
2308:                     if isinstance(e, torch.cuda.OutOfMemoryError)
2309:                     else "eager_2nd_run_fail"
2310:                 )
2311:                 log.exception("")
2312:                 return record_status(accuracy_status, dynamo_start_stats=start_stats)
2313:             finally:
2314:                 del model_copy
2315:                 empty_gpu_cache(current_device)
2316: 
2317:             # Two eager runs should have exactly same result, within tolerance.
2318:             # TODO If we want the above to be true, then deterministic should be set.
2319:             # For example, MIOpen convolutions could be implemented with non-deterministic algos.
2320:             is_same = True
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 2321-2400
````python
2321:             try:
2322:                 if (
2323:                     name not in self.skip_accuracy_check_as_eager_non_deterministic
2324:                     and not same(
2325:                         correct_result,
2326:                         correct_rerun_result,
2327:                         fp64_ref=None,
2328:                         cos_similarity=False,
2329:                         tol=tolerance if torch.version.hip else 0,
2330:                         equal_nan=self.equal_nan,
2331:                         use_larger_multiplier_for_smaller_tensor=self.use_larger_multiplier_for_smaller_tensor(
2332:                             name
2333:                         ),
2334:                     )
2335:                 ):
2336:                     is_same = False
2337:             except Exception:
2338:                 # Sometimes torch.allclose may throw RuntimeError
2339:                 is_same = False
2340: 
2341:             if not is_same:
2342:                 accuracy_status = "eager_two_runs_differ"
2343:                 return record_status(accuracy_status, dynamo_start_stats=start_stats)
2344: 
2345:             correct_rerun_result = None
2346: 
2347:             # Support multiple accuracy check runs for flaky models
2348:             accuracy_check_runs = self.get_accuracy_check_runs(name)
2349:             pass_count = 0
2350: 
2351:             for run_idx in range(accuracy_check_runs):
2352:                 # Run with Dynamo
2353:                 reset_rng_state()
2354:                 torch._dynamo.reset()
2355:                 torch._dynamo.utils.counters.clear()
2356:                 model_copy = None
2357:                 run_passed = True
2358: 
2359:                 try:
2360:                     model_copy = self.deepcopy_and_maybe_parallelize(model)
2361:                     self.init_optimizer(name, current_device, model_copy.parameters())
2362:                     if (
2363:                         self.args.export
2364:                         or self.args.export_aot_inductor
2365:                         or self.args.export_nativert
2366:                         or self.args.torchscript_jit_trace
2367:                         or self.args.aot_precompile
2368:                     ):
2369:                         # apply export on module directly
2370:                         # no need for n iterations
2371:                         # the logic should be the same to self.model_iter_fn (forward_pass)
2372:                         with self.autocast(**self.autocast_arg):
2373:                             optimized_model_iter_fn = optimize_ctx(
2374:                                 model_copy, example_inputs
2375:                             )
2376:                             new_result = optimized_model_iter_fn(
2377:                                 model_copy, example_inputs
2378:                             )
2379:                     else:
2380:                         optimized_model_iter_fn = optimize_ctx(self.model_iter_fn)
2381:                         new_result = self.run_n_iterations(
2382:                             model_copy, example_inputs, optimized_model_iter_fn
2383:                         )
2384:                 except Exception as e:
2385:                     log.exception("")
2386:                     print(
2387:                         "TorchDynamo optimized model failed to run because of following error"
2388:                     )
2389:                     accuracy_status = (
2390:                         "OOM"
2391:                         if isinstance(e, torch.cuda.OutOfMemoryError)
2392:                         else "fail_to_run"
2393:                     )
2394:                     return record_status(
2395:                         accuracy_status, dynamo_start_stats=start_stats
2396:                     )
2397:                 finally:
2398:                     del model_copy
2399: 
2400:                 if name in self.skip_accuracy_check_as_eager_non_deterministic:
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 2401-2480
````python
2401:                     return record_status(
2402:                         "pass_due_to_skip", dynamo_start_stats=start_stats
2403:                     )
2404: 
2405:                 force_max_multiplier = False
2406:                 if (
2407:                     self.args.freezing
2408:                     and self.args.bfloat16
2409:                     and torch._dynamo.utils.counters["inductor"]["binary_folding_conv"]
2410:                     > 0
2411:                 ):
2412:                     force_max_multiplier = True
2413: 
2414:                 try:
2415:                     if self.args.training and self.args.amp:
2416:                         if process_fn := self.get_output_amp_train_process_func.get(
2417:                             name, None
2418:                         ):
2419:                             correct_result = process_fn(correct_result)
2420:                             new_result = process_fn(new_result)
2421:                             fp64_outputs = process_fn(fp64_outputs)
2422: 
2423:                     if (
2424:                         self.args.save_model_outputs_to
2425:                         and self.args.compare_model_outputs_with
2426:                         and self.args.save_model_outputs_to
2427:                         == self.args.compare_model_outputs_with
2428:                     ):
2429:                         log.warning(
2430:                             "args.save_model_outputs_to and args.compare_model_outputs_with points to the same path."
2431:                             "Result will be undefined."
2432:                         )
2433: 
2434:                     if self.args.save_model_outputs_to:
2435:                         print(
2436:                             f"Save model outputs to: {self.args.save_model_outputs_to}"
2437:                         )
2438:                         torch.save(new_result, self.args.save_model_outputs_to)
2439: 
2440:                     if self.args.compare_model_outputs_with:
2441:                         print(
2442:                             f"Load model outputs from {self.args.compare_model_outputs_with} to compare"
2443:                         )
2444:                         saved_result = torch.load(
2445:                             self.args.compare_model_outputs_with, weights_only=False
2446:                         )
2447:                         is_bitwise_same = bitwise_same(saved_result, new_result)
2448:                         if not is_bitwise_same:
2449:                             print(
2450:                                 "The result is not bitwise equivalent to the previously saved result"
2451:                             )
2452:                             return record_status(
2453:                                 "not_bitwise_equivalent",
2454:                                 dynamo_start_stats=start_stats,
2455:                             )
2456: 
2457:                         print(
2458:                             "The result is bitwise equivalent to the previously saved result"
2459:                         )
2460:                         del saved_result
2461: 
2462:                     if not same(
2463:                         correct_result,
2464:                         new_result,
2465:                         fp64_outputs,
2466:                         equal_nan=self.equal_nan,
2467:                         use_larger_multiplier_for_smaller_tensor=self.use_larger_multiplier_for_smaller_tensor(
2468:                             name
2469:                         ),
2470:                         cos_similarity=cos_similarity,
2471:                         tol=tolerance,
2472:                         force_max_multiplier=force_max_multiplier,
2473:                         use_iou_for_bool=self.use_iou_for_bool_accuracy(name),
2474:                         iou_threshold=self.get_iou_threshold(name),
2475:                     ):
2476:                         run_passed = False
2477:                 except Exception:
2478:                     # Sometimes torch.allclose may throw RuntimeError
2479:                     run_passed = False
2480: 
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 2481-2560
````python
2481:                 if run_passed:
2482:                     pass_count += 1
2483: 
2484:                 if accuracy_check_runs > 1:
2485:                     log.info(
2486:                         "Accuracy check run %d/%d: %s",
2487:                         run_idx + 1,
2488:                         accuracy_check_runs,
2489:                         "passed" if run_passed else "failed",
2490:                     )
2491: 
2492:             # Pass if majority of runs pass (more than half)
2493:             is_same = pass_count > accuracy_check_runs // 2
2494: 
2495:             if accuracy_check_runs > 1:
2496:                 log.info(
2497:                     "Accuracy check summary: %d/%d runs passed, %s",
2498:                     pass_count,
2499:                     accuracy_check_runs,
2500:                     "PASS" if is_same else "FAIL",
2501:                 )
2502: 
2503:             if not is_same:
2504:                 if self.args.skip_accuracy_check:
2505:                     accuracy_status = "pass_due_to_skip"
2506:                 else:
2507:                     accuracy_status = "fail_accuracy"
2508:                 return record_status(accuracy_status, dynamo_start_stats=start_stats)
2509: 
2510:         return record_status(accuracy_status, dynamo_start_stats=start_stats)
2511: 
2512:     def check_batch_invariance(
2513:         self, name, model, example_inputs, optimize_ctx, experiment, tag
2514:     ):
2515:         """
2516:         Batch invariance check: run the compiled forward at N, N/2, ..., 1 and
2517:         verify each output matches the reference sliced to that range bitwise.
2518: 
2519:         Always exercises forward-only, even under --training: batch invariance
2520:         is a property of the forward pass; backward and optimizer step
2521:         aggregate over the batch and are not batch-invariant by construction.
2522:         Models with batch-dependent forward ops (e.g. BatchNorm in train mode)
2523:         will still fail here -- that's inherent, not a harness bug.
2524:         """
2525:         start_stats = get_dynamo_stats()
2526: 
2527:         def record_status(status, dynamo_start_stats):
2528:             self._write_accuracy_row(status, dynamo_start_stats, tag)
2529:             return status
2530: 
2531:         if name in self.skip_accuracy_checks_large_models_dashboard:
2532:             return record_status("pass_due_to_skip", dynamo_start_stats=start_stats)
2533: 
2534:         if (
2535:             name in self.skip_accuracy_check_as_eager_non_deterministic
2536:             or name in self.non_deterministic_models
2537:         ):
2538:             return record_status("pass_due_to_skip", dynamo_start_stats=start_stats)
2539: 
2540:         full_batch = current_batch_size
2541:         if full_batch is None or full_batch < 2:
2542:             return record_status("pass_due_to_skip", dynamo_start_stats=start_stats)
2543: 
2544:         # If no input tensor has batch as its first dim, the slicer below is a
2545:         # no-op and the comparison would trivially pass without actually
2546:         # exercising batch invariance. Skip rather than report a misleading pass.
2547:         if not any(
2548:             isinstance(x, torch.Tensor) and x.dim() > 0 and x.shape[0] == full_batch
2549:             for x in pytree.tree_leaves(example_inputs)
2550:         ):
2551:             return record_status("pass_due_to_skip", dynamo_start_stats=start_stats)
2552: 
2553:         def make_slicer(target):
2554:             def slicer(x):
2555:                 if x.dim() > 0 and x.shape[0] == full_batch:
2556:                     return x[:target].contiguous()
2557:                 return x
2558: 
2559:             return slicer
2560: 
````
- EN: Implements callable logic such as `check_batch_invariance`, `record_status`, `make_slicer`, `slicer`.
- CN: 实现可调用逻辑，例如 `check_batch_invariance`, `record_status`, `make_slicer`, `slicer`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 2561-2640
````python
2561:         def run_fresh(inputs):
2562:             # Rebuild model for every run so parameter-mutating side effects
2563:             # (BN running stats, caches, etc.) from a prior run don't bleed
2564:             # into the next comparison. Force eval mode regardless of
2565:             # --training: dropout and train-mode BN are batch-size-dependent
2566:             # by construction. Forward-only: backward/optimizer aggregate
2567:             # over the batch and are not batch-invariant by construction.
2568:             reset_rng_state()
2569:             torch._dynamo.reset()
2570:             torch._dynamo.utils.counters.clear()
2571:             model_copy = self.deepcopy_and_maybe_parallelize(model)
2572:             model_copy.eval()
2573:             try:
2574:                 optimized_iter_fn = optimize_ctx(self.forward_pass)
2575:                 return self.run_n_iterations(model_copy, inputs, optimized_iter_fn)
2576:             finally:
2577:                 del model_copy
2578:                 empty_gpu_cache(current_device)
2579: 
2580:         with self.pick_grad(name, self.args.training):
2581:             model, example_inputs = self.maybe_cast(model, example_inputs)
2582: 
2583:             try:
2584:                 reference = run_fresh(clone_inputs(example_inputs))
2585:             except Exception as e:
2586:                 log.exception("")
2587:                 status = (
2588:                     "OOM"
2589:                     if isinstance(e, torch.cuda.OutOfMemoryError)
2590:                     else "fail_to_run"
2591:                 )
2592:                 return record_status(status, dynamo_start_stats=start_stats)
2593: 
2594:             size = full_batch // 2
2595:             while size >= 1:
2596:                 slicer = make_slicer(size)
2597:                 sliced_inputs = tree_map_only(
2598:                     torch.Tensor, slicer, clone_inputs(example_inputs)
2599:                 )
2600: 
2601:                 try:
2602:                     out = run_fresh(sliced_inputs)
2603:                 except Exception as e:
2604:                     log.exception("")
2605:                     status = (
2606:                         "OOM"
2607:                         if isinstance(e, torch.cuda.OutOfMemoryError)
2608:                         else f"fail_to_run_at_batch_{size}"
2609:                     )
2610:                     return record_status(status, dynamo_start_stats=start_stats)
2611: 
2612:                 reference_sliced = tree_map_only(torch.Tensor, slicer, reference)
2613: 
2614:                 # Only compare batch-first output tensors. Aggregated outputs
2615:                 # (e.g. HuggingFace's MaskedLMOutput.loss) don't have a batch
2616:                 # dim and legitimately differ between batch sizes; comparing
2617:                 # them would produce misleading failures.
2618:                 def keep_batch_first(x):
2619:                     return x if x.dim() > 0 and x.shape[0] == size else None
2620: 
2621:                 ref_for_cmp = tree_map_only(
2622:                     torch.Tensor, keep_batch_first, reference_sliced
2623:                 )
2624:                 out_for_cmp = tree_map_only(torch.Tensor, keep_batch_first, out)
2625: 
2626:                 try:
2627:                     is_same = bitwise_same(
2628:                         ref_for_cmp, out_for_cmp, equal_nan=self.equal_nan
2629:                     )
2630:                 except Exception:
2631:                     is_same = False
2632: 
2633:                 if not is_same:
2634:                     if self.args.skip_accuracy_check:
2635:                         return record_status(
2636:                             "pass_due_to_skip", dynamo_start_stats=start_stats
2637:                         )
2638:                     return record_status(
2639:                         f"fail_batch_invariance_at_{size}",
2640:                         dynamo_start_stats=start_stats,
````
- EN: Implements callable logic such as `run_fresh`, `keep_batch_first`.
- CN: 实现可调用逻辑，例如 `run_fresh`, `keep_batch_first`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 2641-2720
````python
2641:                     )
2642: 
2643:                 size //= 2
2644: 
2645:         return record_status("pass", dynamo_start_stats=start_stats)
2646: 
2647:     def check_tolerance(
2648:         self, name, model, example_inputs, optimize_ctx, base_device="cpu"
2649:     ):
2650:         """
2651:         Checks tolerance based on https://pytorch.org/docs/stable/generated/torch.allclose.html.
2652:         """
2653:         tolerance_status = "pass"
2654:         if name in self.skip_accuracy_checks_large_models_dashboard:
2655:             tolerance_status = "pass_due_to_skip"
2656:             return tolerance_status
2657:         # Cast the model to float16/float32 as necessary
2658:         model, example_inputs = self.maybe_cast(model, example_inputs)
2659: 
2660:         with self.pick_grad(name, self.args.training):
2661:             # Get results of native pytorch
2662:             reset_rng_state()
2663:             model_copy = copy.deepcopy(model)
2664:             model_copy = model_copy.to(base_device)
2665:             example_inputs_copy = copy.deepcopy(example_inputs)
2666:             example_inputs_copy = tree_map(
2667:                 lambda x: x.to(base_device), example_inputs_copy
2668:             )
2669:             self.init_optimizer(name, base_device, model_copy.parameters())
2670:             correct_result = self.run_n_iterations(
2671:                 model_copy, example_inputs_copy, self.model_iter_fn
2672:             )
2673: 
2674:             # Run with Dynamo
2675:             # Sometime CI fails with random triton compilation failure which will be skipped for now
2676:             # TODO: revisit this after switching to new Triton runtime
2677:             reset_rng_state()
2678:             torch._dynamo.reset()
2679:             try:
2680:                 self.init_optimizer(name, current_device, model.parameters())
2681:                 optimized_model_iter_fn = optimize_ctx(self.model_iter_fn)
2682:                 new_result = self.run_n_iterations(
2683:                     model_copy, example_inputs, optimized_model_iter_fn
2684:                 )
2685:             except Exception:
2686:                 log.exception("")
2687:                 print(
2688:                     "TorchDynamo optimized model failed to run because of following error"
2689:                 )
2690:                 return "fail_to_run"
2691: 
2692:             def dump_max_mean_values(tol, ref, res):
2693:                 if isinstance(ref, (list, tuple, torch.nn.ParameterList, torch.Size)):
2694:                     for refi, resi in zip(ref, res):
2695:                         dump_max_mean_values(tol, refi, resi)
2696:                 elif isinstance(ref, dict):
2697:                     for k in ref:
2698:                         dump_max_mean_values(tol, ref[k], res[k])
2699:                 elif isinstance(ref, torch.Tensor):
2700:                     res = res.to(base_device)
2701:                     t = torch.abs(ref - res) / (1 + torch.abs(ref))
2702:                     tol.append(t.flatten().to(torch.float32))
2703:                 return tol
2704: 
2705:             tol = []
2706:             dump_max_mean_values(tol, correct_result, new_result)
2707:             tol = torch.cat(tol)
2708:             tol = torch.tensor(tol)
2709:             max = torch.max(tol)
2710:             mean = torch.mean(tol)
2711:             div = torch.std(tol)
2712:             headers = ["dev", "name", "batch_size", "max", "mean", "std"]
2713:             fields = [
2714:                 current_device,
2715:                 current_name,
2716:                 current_batch_size,
2717:                 max.item(),
2718:                 mean.item(),
2719:                 div.item(),
2720:             ]
````
- EN: Implements callable logic such as `check_tolerance`, `dump_max_mean_values`.
- CN: 实现可调用逻辑，例如 `check_tolerance`, `dump_max_mean_values`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 2721-2800
````python
2721:             write_outputs(output_filename, headers, fields)
2722:         return tolerance_status
2723: 
2724:     def run_performance_test_non_alternate(
2725:         self, name, model, example_inputs, optimize_ctx, experiment, tag=None
2726:     ):
2727:         "Run performance test in non-alternately."
2728:         if experiment.func is not latency_experiment:
2729:             raise AssertionError(
2730:                 f"Must run with latency_experiment, got {experiment.func}"
2731:             )
2732: 
2733:         def warmup(fn, model, example_inputs, mode, niters=10):
2734:             gc.collect()
2735:             peak_mem = 0
2736:             start_stats = get_dynamo_stats()
2737:             try:
2738:                 if current_device == "cuda":
2739:                     torch.cuda.reset_peak_memory_stats()
2740:                     empty_gpu_cache(current_device)
2741:                 elif current_device == "hpu":
2742:                     torch.hpu.reset_peak_memory_stats()
2743:                 t0 = time.perf_counter()
2744:                 for _ in range(niters):
2745:                     fn(model, example_inputs)
2746:                 t1 = time.perf_counter()
2747:                 latency = t1 - t0
2748:                 if current_device == "cuda":
2749:                     peak_mem = get_peak_memory()
2750:                 elif current_device == "hpu":
2751:                     peak_mem = torch.hpu.max_memory_allocated() / 10**9
2752:                 elif current_device == "cpu":
2753:                     total = psutil.virtual_memory().total
2754:                     percentage = psutil.Process(os.getpid()).memory_percent()
2755:                     peak_mem = percentage * total / 10**9
2756:             except Exception:
2757:                 log.exception("Backend %s failed in warmup()", mode)
2758:                 write_csv_when_exception(
2759:                     self.args, current_name, "warmup_failed", current_device
2760:                 )
2761:                 output_signpost({}, self.args, self.suite_name, error="warmup_failed")
2762:                 return sys.exit(-1)
2763:             dynamo_stats = get_dynamo_stats()
2764:             dynamo_stats.subtract(start_stats)
2765:             return latency, peak_mem, dynamo_stats
2766: 
2767:         # Cast the model to float16/float32 as necessary
2768:         model, example_inputs = self.maybe_cast(model, example_inputs)
2769: 
2770:         # Use distributed wrapping as necessary
2771:         model = self.deepcopy_and_maybe_parallelize(model)
2772: 
2773:         if not hasattr(model, name):
2774:             model.name = name
2775:         self.init_optimizer(name, current_device, model.parameters())
2776: 
2777:         # The self.autocast context is needed for the model we export with aot_compile,
2778:         # similar to what we do in the check_accuracy function
2779:         ctx = (
2780:             self.autocast(**self.autocast_arg)
2781:             if self.args.export_aot_inductor
2782:             else contextlib.nullcontext()
2783:         )
2784: 
2785:         with self.pick_grad(name, self.args.training), ctx:
2786:             ok, total = Stats.reset_counters()
2787:             experiment_kwargs = {}
2788:             if tag is not None:
2789:                 experiment_kwargs["tag"] = tag
2790:             results = []
2791: 
2792:             with maybe_snapshot_memory(
2793:                 self.args.snapshot_memory, f"eager_{self.args.only}"
2794:             ):
2795:                 eager_latency, eager_peak_mem, _ = warmup(
2796:                     self.model_iter_fn, model, example_inputs, "eager"
2797:                 )
2798:                 if self.args.use_warm_peak_memory:
2799:                     _, eager_peak_mem, _ = warmup(
2800:                         self.model_iter_fn, model, example_inputs, "eager", niters=1
````
- EN: Implements callable logic such as `run_performance_test_non_alternate`, `warmup`.
- CN: 实现可调用逻辑，例如 `run_performance_test_non_alternate`, `warmup`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 2801-2880
````python
2801:                     )
2802: 
2803:             baseline_timings = experiment(
2804:                 self.model_iter_fn,
2805:                 model,
2806:                 example_inputs,
2807:                 mark="expected",
2808:                 **experiment_kwargs,
2809:             )
2810: 
2811:             # reset dynamo
2812:             torch._dynamo.reset()
2813: 
2814:             if self.args.export_aot_inductor:
2815:                 optimized_model_iter_fn = optimize_ctx
2816:             else:
2817:                 optimized_model_iter_fn = optimize_ctx(self.model_iter_fn)
2818: 
2819:             with maybe_snapshot_memory(
2820:                 self.args.snapshot_memory, f"compiled_{self.args.only}"
2821:             ):
2822:                 dynamo_latency, dynamo_peak_mem, dynamo_stats = warmup(
2823:                     optimized_model_iter_fn, model, example_inputs, "dynamo"
2824:                 )
2825:                 if self.args.use_warm_peak_memory:
2826:                     _, dynamo_peak_mem, _ = warmup(
2827:                         optimized_model_iter_fn,
2828:                         model,
2829:                         example_inputs,
2830:                         "dynamo",
2831:                         niters=1,
2832:                     )
2833:                 # If we use warm peak memory, the AOT model loading transient memory
2834:                 # won't be present on the warm measurement.  We only have to account for
2835:                 # it when using cold memory.
2836:                 elif self.args.export_aot_inductor:
2837:                     dynamo_peak_mem -= AOTInductorModelCache.get_excess_memory(model)
2838: 
2839:             if self.args.profile_dynamo_cache_lookup:
2840:                 with torch.profiler.profile(
2841:                     activities=[torch.profiler.ProfilerActivity.CPU]
2842:                 ) as prof:
2843:                     warmup(optimized_model_iter_fn, model, example_inputs, "dynamo")
2844: 
2845:                 events = list(
2846:                     filter(
2847:                         lambda event: "TorchDynamo Cache Lookup" in event.key,
2848:                         prof.key_averages(),
2849:                     )
2850:                 )
2851:                 dynamo_cache_lookup_latency = events[0].self_cpu_time_total
2852: 
2853:             compilation_time = dynamo_latency - eager_latency
2854:             compression_ratio = (
2855:                 eager_peak_mem / dynamo_peak_mem if dynamo_peak_mem else 0.0
2856:             )
2857:             if self.args.print_memory:
2858:                 print(
2859:                     f"memory: eager: {eager_peak_mem:.2f} GB, "
2860:                     f"dynamo: {dynamo_peak_mem:.2f} GB, "
2861:                     f"ratio: {compression_ratio:.2f}"
2862:                 )
2863: 
2864:             if self.args.print_compilation_time:
2865:                 print(f"Compilation time: {compilation_time:.2f}")
2866: 
2867:             if experiment.func is speedup_experiment:
2868:                 experiment_kwargs["compilation_latency"] = compilation_time
2869:                 experiment_kwargs["compression_ratio"] = compression_ratio
2870:                 experiment_kwargs["eager_peak_mem"] = eager_peak_mem
2871:                 experiment_kwargs["dynamo_peak_mem"] = dynamo_peak_mem
2872:                 experiment_kwargs["dynamo_stats"] = dynamo_stats
2873:                 if self.args.profile_dynamo_cache_lookup:
2874:                     experiment_kwargs["cache_lookup_latency"] = (
2875:                         dynamo_cache_lookup_latency
2876:                     )
2877: 
2878:             backend_timings = experiment(
2879:                 self.model_iter_fn,
2880:                 model,
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 2881-2960
````python
2881:                 example_inputs,
2882:                 mark="expected",
2883:                 **experiment_kwargs,
2884:             )
2885:             timings = np.stack((baseline_timings, backend_timings), axis=1)
2886:             result_summary = latency_experiment_summary(
2887:                 self.suite_name, self.args, model, timings, **experiment_kwargs
2888:             )
2889:             results.append(result_summary)
2890:             return " ".join(map(str, results))
2891: 
2892:     def run_performance_test(
2893:         self,
2894:         name,
2895:         model,
2896:         example_inputs,
2897:         optimize_ctx,
2898:         experiment,
2899:         tag=None,
2900:         batch_size=None,
2901:     ):
2902:         niters = 5
2903:         if getattr(self, "hf_llm", False):
2904:             # If we're benchmarking an llm, we want to use the generate function
2905:             self.model_iter_fn = self.generate
2906:             niters = 1
2907: 
2908:         if self.args.xla:
2909:             with self.pick_grad(name, self.args.training):
2910:                 return experiment(
2911:                     self.model_iter_fn, *self.maybe_cast(model, example_inputs)
2912:                 )
2913: 
2914:         def warmup(fn, model, example_inputs, mode, niters=5):
2915:             gc.collect()
2916:             peak_mem = 0
2917:             start_stats = get_dynamo_stats()
2918:             try:
2919:                 if current_device == "cuda":
2920:                     torch.cuda.reset_peak_memory_stats()
2921:                     empty_gpu_cache(current_device)
2922:                 elif current_device == "hpu":
2923:                     torch.hpu.reset_peak_memory_stats()
2924:                 t0 = time.perf_counter()
2925:                 for _ in range(niters):
2926:                     fn(model, example_inputs)
2927:                 t1 = time.perf_counter()
2928:                 latency = t1 - t0
2929:                 if current_device == "cuda":
2930:                     peak_mem = get_peak_memory()
2931:                 elif current_device == "hpu":
2932:                     peak_mem = torch.hpu.max_memory_allocated() / 10**9
2933:                 elif current_device == "cpu":
2934:                     total = psutil.virtual_memory().total
2935:                     percentage = psutil.Process(os.getpid()).memory_percent()
2936:                     peak_mem = percentage * total / 10**9
2937:             except Exception:
2938:                 log.exception("Backend %s failed in warmup()", mode)
2939:                 write_csv_when_exception(
2940:                     self.args, current_name, "warmup_failed", current_device
2941:                 )
2942:                 output_signpost({}, self.args, self.suite_name, error="warmup_failed")
2943:                 return sys.exit(-1)
2944:             dynamo_stats = get_dynamo_stats()
2945:             dynamo_stats.subtract(start_stats)
2946:             return latency, peak_mem, dynamo_stats
2947: 
2948:         # Cast the model to float16/float32 as necessary
2949:         model, example_inputs = self.maybe_cast(model, example_inputs)
2950: 
2951:         # Use distributed wrapping as necessary
2952:         model = self.deepcopy_and_maybe_parallelize(model)
2953: 
2954:         if not hasattr(model, name):
2955:             model.name = name
2956: 
2957:         self.init_optimizer(name, current_device, model.parameters())
2958: 
2959:         # The self.autocast context is needed for the model we export with aot_compile,
2960:         # similar to what we do in the check_accuracy function
````
- EN: Implements callable logic such as `run_performance_test`, `warmup`.
- CN: 实现可调用逻辑，例如 `run_performance_test`, `warmup`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 2961-3040
````python
2961:         ctx = (
2962:             self.autocast(**self.autocast_arg)
2963:             if self.args.export_aot_inductor
2964:             else contextlib.nullcontext()
2965:         )
2966: 
2967:         with self.pick_grad(name, self.args.training), ctx:
2968:             ok, total = Stats.reset_counters()
2969:             experiment_kwargs = {}
2970:             experiment_kwargs["batch_size"] = batch_size
2971:             if tag is not None:
2972:                 experiment_kwargs["tag"] = tag
2973:             results = []
2974:             with maybe_snapshot_memory(
2975:                 self.args.snapshot_memory, f"eager_{self.args.only}"
2976:             ):
2977:                 with torch.compiler.set_stance("force_eager"):
2978:                     eager_latency, eager_peak_mem, _ = warmup(
2979:                         self.model_iter_fn,
2980:                         copy.deepcopy(model),
2981:                         example_inputs,
2982:                         "eager",
2983:                         niters=niters,
2984:                     )
2985:                     if self.args.use_warm_peak_memory:
2986:                         _, eager_peak_mem, _ = warmup(
2987:                             self.model_iter_fn,
2988:                             copy.deepcopy(model),
2989:                             example_inputs,
2990:                             "eager",
2991:                             niters=1,
2992:                         )
2993: 
2994:             if (
2995:                 self.args.export_aot_inductor
2996:                 or self.args.export_nativert
2997:                 or self.args.torchscript_jit_trace
2998:                 or self.args.aot_precompile
2999:             ):
3000:                 optimized_model_iter_fn = optimize_ctx
3001:             else:
3002:                 if getattr(self, "hf_llm", False):
3003:                     # If it's an llm, we want to optimize model.forward, and use
3004:                     # the generate function
3005:                     model = optimize_ctx(model)
3006:                     optimized_model_iter_fn = self.model_iter_fn
3007:                 else:
3008:                     optimized_model_iter_fn = optimize_ctx(self.model_iter_fn)
3009: 
3010:             with maybe_snapshot_memory(
3011:                 self.args.snapshot_memory, f"compiled_{self.args.only}"
3012:             ):
3013:                 dynamo_latency, dynamo_peak_mem, dynamo_stats = warmup(
3014:                     optimized_model_iter_fn, model, example_inputs, "dynamo"
3015:                 )
3016:                 if self.args.use_warm_peak_memory:
3017:                     _, dynamo_peak_mem, _ = warmup(
3018:                         optimized_model_iter_fn,
3019:                         model,
3020:                         example_inputs,
3021:                         "dynamo",
3022:                         niters=1,
3023:                     )
3024:                 # If we use warm peak memory, the AOT model loading transient memory
3025:                 # won't be present on the warm measurement.  We only have to account for
3026:                 # it when using cold memory.
3027:                 elif self.args.export_aot_inductor:
3028:                     dynamo_peak_mem -= AOTInductorModelCache.get_excess_memory(model)
3029: 
3030:             if self.args.profile_dynamo_cache_lookup:
3031:                 with torch.profiler.profile(
3032:                     activities=[torch.profiler.ProfilerActivity.CPU]
3033:                 ) as prof:
3034:                     warmup(optimized_model_iter_fn, model, example_inputs, "dynamo")
3035: 
3036:                 events = list(
3037:                     filter(
3038:                         lambda event: "TorchDynamo Cache Lookup" in event.key,
3039:                         prof.key_averages(),
3040:                     )
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 3041-3120
````python
3041:                 )
3042:                 dynamo_cache_lookup_latency = events[0].self_cpu_time_total
3043: 
3044:             compilation_time = dynamo_latency - eager_latency
3045:             compression_ratio = (
3046:                 eager_peak_mem / dynamo_peak_mem if dynamo_peak_mem else 0.0
3047:             )
3048:             if self.args.print_memory:
3049:                 print(
3050:                     f"memory: eager: {eager_peak_mem:.2f} GB, "
3051:                     f"dynamo: {dynamo_peak_mem:.2f} GB, "
3052:                     f"ratio: {compression_ratio:.2f}"
3053:                 )
3054: 
3055:             if self.args.print_compilation_time:
3056:                 print(f"Compilation time: {compilation_time:.2f}")
3057: 
3058:             if experiment.func is speedup_experiment:
3059:                 experiment_kwargs["compilation_latency"] = compilation_time
3060:                 experiment_kwargs["compression_ratio"] = compression_ratio
3061:                 experiment_kwargs["eager_peak_mem"] = eager_peak_mem
3062:                 experiment_kwargs["dynamo_peak_mem"] = dynamo_peak_mem
3063:                 experiment_kwargs["dynamo_stats"] = dynamo_stats
3064:                 if self.args.profile_dynamo_cache_lookup:
3065:                     experiment_kwargs["cache_lookup_latency"] = (
3066:                         dynamo_cache_lookup_latency
3067:                     )
3068: 
3069:             if experiment.func is coverage_experiment:
3070:                 ok, total = Stats.reset_counters()
3071:                 results = []
3072:                 # run with torch._dynamo few times to populate the cache
3073:                 for _ in range(3):
3074:                     optimized_model_iter_fn(model, example_inputs)
3075:                 _, frames_second_pass = Stats.reset_counters()  # should be 0
3076:                 if frames_second_pass > 0:
3077:                     optimized_model_iter_fn(model, example_inputs)
3078:                     _, frames_third_pass = Stats.reset_counters()  # should be 0
3079:                 else:
3080:                     frames_third_pass = 0
3081: 
3082:                 results.append(
3083:                     f"{ok:3}/{total:3} +{frames_third_pass} frames {compilation_time:3.0f}s"
3084:                 )
3085: 
3086:             experiment_kwargs["hf_llm"] = getattr(self, "hf_llm", False)
3087: 
3088:             results.append(
3089:                 experiment(
3090:                     self.model_iter_fn, model, example_inputs, **experiment_kwargs
3091:                 )
3092:             )
3093:             return " ".join(map(str, results))
3094: 
3095:     def minify_model(
3096:         self,
3097:         name,
3098:         model,
3099:         example_inputs,
3100:         optimize_ctx,
3101:         experiment,
3102:         tag,
3103:     ):
3104:         log.info("Minifying %s...", name)
3105:         os.environ["TORCH_COMPILE_DEBUG"] = "1"
3106:         os.environ["TORCHDYNAMO_REPRO_AFTER"] = "dynamo"
3107:         os.environ["TORCHDYNAMO_REPRO_LEVEL"] = "4"
3108: 
3109:         self.check_accuracy(name, model, example_inputs, optimize_ctx, experiment, tag)
3110: 
3111:         if self.args.output_directory:
3112:             repro_dir = self.args.output_directory
3113:         else:
3114:             repro_dir = torch._dynamo.config.base_dir
3115: 
3116:         try:
3117:             shutil.move("repro.py", f"{repro_dir}/{name}_repro.py")
3118:         except OSError:
3119:             log.error("Could not find repro script for model %s", name)
3120:         else:
````
- EN: Implements callable logic such as `minify_model`.
- CN: 实现可调用逻辑，例如 `minify_model`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 3121-3200
````python
3121:             log.info(
3122:                 "Repro script for model %s with minified graph saved to %s",
3123:                 name,
3124:                 repro_dir,
3125:             )
3126: 
3127:     def maybe_preserve_compile_debug(self, name, status):
3128:         if (
3129:             name in CI_PRESERVE_COMPILE_DEBUG
3130:             and status in CI_PRESERVE_COMPILE_DEBUG[name]
3131:         ):
3132:             src_dir = torch._dynamo.utils.get_debug_dir()
3133:             if os.path.isdir(src_dir):
3134:                 dbg_dir = os.path.join(
3135:                     os.getcwd(), "test", "debug", "torch_compile_debug"
3136:                 )
3137:                 dst_dir = os.path.join(dbg_dir, os.path.basename(src_dir))
3138:                 try:
3139:                     os.makedirs(dbg_dir, exist_ok=True)
3140:                     os.rename(src_dir, dst_dir)
3141:                     log.warning("Moved %s to %s", src_dir, dst_dir)
3142:                 except OSError:
3143:                     log.exception("Failed to preserve %s", src_dir)
3144: 
3145:     def run_one_model(
3146:         self,
3147:         name,
3148:         model,
3149:         example_inputs,
3150:         optimize_ctx,
3151:         experiment,
3152:         explain=False,
3153:         tag=None,
3154:         batch_size=None,
3155:     ):
3156:         mode = "train" if self.args.training else "eval"
3157:         msg = f"{current_device:4} {mode:5} {current_name:34} "
3158:         if tag:
3159:             msg += f" {tag:26}"
3160:         print(msg, flush=True)
3161: 
3162:         start_stats = get_dynamo_stats()
3163: 
3164:         if self.args.accuracy:
3165:             if self.args.batch_invariant:
3166:                 status = self.check_batch_invariance(
3167:                     name, model, example_inputs, optimize_ctx, experiment, tag
3168:                 )
3169:             else:
3170:                 status = self.check_accuracy(
3171:                     name, model, example_inputs, optimize_ctx, experiment, tag
3172:                 )
3173:             print(status)
3174:             if status == "fail_accuracy" and self.args.minify:
3175:                 self.minify_model(
3176:                     name, model, example_inputs, optimize_ctx, experiment, tag
3177:                 )
3178:         elif self.args.tolerance:
3179:             status = self.check_tolerance(name, model, example_inputs, optimize_ctx)
3180:             print(status)
3181:         elif self.args.performance:
3182:             if self.args.backend in ["torchao", "optimus"]:
3183:                 status = self.run_performance_test_non_alternate(
3184:                     name, model, example_inputs, optimize_ctx, experiment, tag
3185:                 )
3186:             else:
3187:                 status = self.run_performance_test(
3188:                     name,
3189:                     model,
3190:                     example_inputs,
3191:                     optimize_ctx,
3192:                     experiment,
3193:                     tag,
3194:                     batch_size=batch_size,
3195:                 )
3196:             print(status)
3197:         empty_gpu_cache(current_device)
3198: 
3199:         self.maybe_preserve_compile_debug(name, status)
3200: 
````
- EN: Implements callable logic such as `maybe_preserve_compile_debug`, `run_one_model`.
- CN: 实现可调用逻辑，例如 `maybe_preserve_compile_debug`, `run_one_model`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 3201-3280
````python
3201:         if self.args.timing:
3202:             from torch._dynamo.utils import op_count, print_time_report
3203:             from torch.utils._stats import simple_call_counter
3204: 
3205:             print_time_report()
3206:             stats = "STATS: "
3207:             stats = stats + " | ".join(
3208:                 itertools.chain(
3209:                     [f"call_* op count: {op_count}"],
3210:                     (f"{key}:{value}" for key, value in simple_call_counter.items()),
3211:                 )
3212:             )
3213:             print(stats)
3214:         stats = get_dynamo_stats()
3215:         stats.subtract(start_stats)
3216: 
3217:         if explain:
3218:             print(
3219:                 f"Dynamo produced {stats['unique_graphs']} graphs "
3220:                 f"covering {stats['calls_captured']} ops with "
3221:                 f"{stats['graph_breaks']} graph breaks ({stats['unique_graph_breaks']} unique)"
3222:             )
3223: 
3224:         if explain or self.args.log_graph_breaks or self.args.print_graph_breaks:
3225:             filename = f"{output_filename.rstrip('.csv')}_graph_breaks.csv"
3226: 
3227:             def add_double_quotes(x):
3228:                 # Delimiter because reason could have comma
3229:                 return f'"{x}"'
3230: 
3231:             for graph_break in graph_break_reasons:
3232:                 reason = add_double_quotes(graph_break.reason)
3233:                 user_stack = add_double_quotes(
3234:                     ", ".join([str(x) for x in graph_break.user_stack])
3235:                 )
3236: 
3237:                 # NB: Don't upload them to the benchmark database as they are debugging
3238:                 # information. There are also around a million records a day which is
3239:                 # wasteful to store
3240:                 write_outputs(
3241:                     filename,
3242:                     ["model", "reason", "user_stack"],
3243:                     [current_name, reason, user_stack],
3244:                     False,
3245:                 )
3246: 
3247:         if self.args.stats:
3248:             Stats.print_summary()
3249: 
3250: 
3251: def help(fn):
3252:     return fn.__doc__
3253: 
3254: 
3255: diff_branch_default = "DIFF-BRANCH-DEFAULT"
3256: 
3257: 
3258: def should_diff_branch(args):
3259:     return args.diff_branch != diff_branch_default
3260: 
3261: 
3262: def parse_args(args=None):
3263:     parser = argparse.ArgumentParser()
3264:     parser.add_argument(
3265:         "--filter", "-k", action="append", help="filter benchmarks with regexp"
3266:     )
3267:     parser.add_argument(
3268:         "--exclude", "-x", action="append", help="filter benchmarks with regexp"
3269:     )
3270:     parser.add_argument(
3271:         "--exclude-exact", action="append", help="filter benchmarks with exact match"
3272:     )
3273:     parser.add_argument(
3274:         "--total-partitions",
3275:         type=int,
3276:         default=1,
3277:         choices=range(1, 16),
3278:         help="Total number of partitions we want to divide the benchmark suite into",
3279:     )
3280:     parser.add_argument(
````
- EN: Handles module imports such as `torch._dynamo.utils`, `torch.utils._stats`.
- CN: 处理模块导入，例如 `torch._dynamo.utils`, `torch.utils._stats`。
- EN: Implements callable logic such as `add_double_quotes`, `help`, `should_diff_branch`, `parse_args`.
- CN: 实现可调用逻辑，例如 `add_double_quotes`, `help`, `should_diff_branch`, `parse_args`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 3281-3360
````python
3281:         "--partition-id",
3282:         type=int,
3283:         default=0,
3284:         help="ID of the benchmark suite partition to be run. Used to divide CI tasks",
3285:     )
3286:     parser.add_argument(
3287:         "--devices", "--device", "-d", action="append", help="cpu, cuda or hpu"
3288:     )
3289:     parser.add_argument("--device-index", help="CUDA device index")
3290:     parser.add_argument(
3291:         "--repeat", "-n", type=int, default=30, help="number of timing runs"
3292:     )
3293:     iterations_per_run_help = """
3294:         Run this may iterations for each time measurement. This is mainly used for
3295:         XLA training. We want to run multiple iterations per measurement so the
3296:         tracing and computation for different iterations can overlap with each
3297:         other. This makes sure we have an accurate xla baseline.
3298:     """
3299:     parser.add_argument(
3300:         "--iterations-per-run", type=int, default=1, help=iterations_per_run_help
3301:     )
3302:     parser.add_argument(
3303:         "--randomize-input",
3304:         action="store_true",
3305:         help="Whether to randomize the input values. Dimensions will be kept the same.",
3306:     )
3307:     parser.add_argument(
3308:         "--threads",
3309:         "-t",
3310:         type=int,
3311:         help="number of threads to use for eager and inductor",
3312:     )
3313:     parser.add_argument(
3314:         "--nopython", action="store_true", help="Turn graph breaks into errors"
3315:     )
3316:     parser.add_argument(
3317:         "--no-skip",
3318:         action="store_true",
3319:         help="run models that are in the global SKIP list",
3320:     )
3321:     parser.add_argument(
3322:         "--prims-nvfuser", action="store_true", help="user prims + nvfuser backend"
3323:     )
3324:     parser.add_argument(
3325:         "--dump-raw-metrics",
3326:         action="store_true",
3327:         help="dump raw timing metrics from speedup experiment",
3328:     )
3329:     parser.add_argument(
3330:         "--log-operator-inputs",
3331:         action="store_true",
3332:         default=False,
3333:     )
3334:     parser.add_argument(
3335:         "--channels-last",
3336:         action="store_true",
3337:         default=False,
3338:         help="use channels last format",
3339:     )
3340:     parser.add_argument(
3341:         "--batch-size", "--batch_size", type=int, help="batch size for benchmarking"
3342:     )
3343:     parser.add_argument(
3344:         "--iterations", type=int, default=2, help="how many iterations to run"
3345:     )
3346:     parser.add_argument(
3347:         "--batch-size-file", type=str, help="String to load batch size from"
3348:     )
3349:     parser.add_argument("--cosine", action="store_true", help="use cosine similarity")
3350:     parser.add_argument(
3351:         "--freezing", action="store_true", help="turn on freezing", default=False
3352:     )
3353:     parser.add_argument(
3354:         "--deterministic",
3355:         action="store_true",
3356:         help="Enable deterministic mode (torch.use_deterministic_algorithms, cudnn.deterministic, etc.)",
3357:     )
3358:     parser.add_argument(
3359:         "--batch-invariant",
3360:         action="store_true",
````
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 3361-3440
````python
3361:         help="Check batch invariance: compare compiled forward outputs at full vs half batch "
3362:         "size and verify they match bitwise. Only valid with --accuracy.",
3363:     )
3364:     parser.add_argument(
3365:         "--inductor-config",
3366:         "-c",
3367:         action="append",
3368:         help="key=value in torch._inductor.config",
3369:     )
3370:     parser.add_argument(
3371:         "--ci", action="store_true", help="Flag to tell that its a CI run"
3372:     )
3373:     parser.add_argument(
3374:         "--dashboard", action="store_true", help="Flag to tell that its a Dashboard run"
3375:     )
3376:     parser.add_argument(
3377:         "--skip-fp64-check", action="store_true", help="skip accuracy check using fp64"
3378:     )
3379:     parser.add_argument(
3380:         "--fast", "-f", action="store_true", help="skip slow benchmarks"
3381:     )
3382:     parser.add_argument(
3383:         "--only",
3384:         help="""Run just one model from torchbench. Or
3385:         specify the path and class name of the model in format like:
3386:         --only=path:<MODEL_FILE_PATH>,class:<CLASS_NAME>
3387: 
3388:         Due to the fact that dynamo changes current working directory,
3389:         the path should be an absolute path.
3390: 
3391:         The class should have a method get_example_inputs to return the inputs
3392:         for the model. An example looks like
3393:         ```
3394:         class LinearModel(nn.Module):
3395:             def __init__(self):
3396:                 super().__init__()
3397:                 self.linear = nn.Linear(10, 10)
3398: 
3399:             def forward(self, x):
3400:                 return self.linear(x)
3401: 
3402:             def get_example_inputs(self):
3403:                 return (torch.randn(2, 10),)
3404:         ```
3405:     """,
3406:     )
3407:     parser.add_argument(
3408:         "--multiprocess",
3409:         action="store_true",
3410:         help="Create n processes based on the number of devices (distributed use case).",
3411:     )
3412:     parser.add_argument(
3413:         "--ddp",
3414:         action="store_true",
3415:         help="Wraps model in DDP before running it, and uses dynamo DDPOptmizer (graph breaks) by default.",
3416:     )
3417:     parser.add_argument(
3418:         "--fsdp",
3419:         action="store_true",
3420:         help="""Wraps model in FSDP before running it.
3421:         Doesn't recursively wrap, mainly useful for checking dynamo UnspecNNModule compatibility
3422:     """,
3423:     )
3424:     parser.add_argument(
3425:         "--optimize-ddp-mode",
3426:         type=str,
3427:         default="ddp_optimizer",
3428:         help="Specify the DDP optimization mode -- the value of torch._dynamo.config.optimize_ddp.",
3429:     )
3430:     parser.add_argument(
3431:         "--distributed-master-port",
3432:         default="6789",
3433:         help="Port to bind for for torch.distributed.  Use the default unless it's conflicting with another user",
3434:     )
3435:     parser.add_argument(
3436:         "--dynamic-shapes",
3437:         action="store_true",
3438:         help="Runs a dynamic shapes version of the benchmark, if available.",
3439:     )
3440:     parser.add_argument(
````
- EN: Declares or extends types including `name`, `should`, `LinearModel`.
- CN: 声明或扩展类型，包括 `name`, `should`, `LinearModel`。
- EN: Implements callable logic such as `__init__`, `forward`, `get_example_inputs`.
- CN: 实现可调用逻辑，例如 `__init__`, `forward`, `get_example_inputs`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 3441-3520
````python
3441:         "--propagate-real-tensors",
3442:         action="store_true",
3443:         help="Capture as much data dependent as you can by unsoundly propagating real tensors",
3444:     )
3445:     parser.add_argument(
3446:         "--dynamic-batch-only",
3447:         action="store_true",
3448:         help="Only assume batch dimension is dynamic.  Implies --dynamic-shapes",
3449:     )
3450:     parser.add_argument(
3451:         "--unbacked-batch-only",
3452:         action="store_true",
3453:         help="Mark batch dimension as unbacked using mark_unbacked. Implies --dynamic-shapes",
3454:     )
3455:     parser.add_argument(
3456:         "--compare-backed-unbacked",
3457:         action="store_true",
3458:         help="Run both dynamic-batch-only (backed) and unbacked-batch-only, then compare results side by side",
3459:     )
3460:     parser.add_argument(
3461:         "--_print-latency-ms",
3462:         action="store_true",
3463:         help=argparse.SUPPRESS,
3464:     )
3465:     parser.add_argument(
3466:         "--specialize-int", action="store_true", help="Run with specialize_int=True."
3467:     )
3468:     parser.add_argument(
3469:         "--use-eval-mode",
3470:         action="store_true",
3471:         help="sets model.eval() to reduce randomness",
3472:     )
3473:     parser.add_argument(
3474:         "--skip-accuracy-check",
3475:         action="store_true",
3476:         help="keeps running even when accuracy fails",
3477:     )
3478:     parser.add_argument(
3479:         "--generate-aot-autograd-stats",
3480:         action="store_true",
3481:         help="Generates AOT Autograd stats like how many graphs are sent to AOT",
3482:     )
3483:     parser.add_argument(
3484:         "--inductor-settings",
3485:         action="store_true",
3486:         help="Use same settings as --inductor for baseline comparisons",
3487:     )
3488:     parser.add_argument(
3489:         "--suppress-errors",
3490:         action="store_true",
3491:         help="Suppress errors instead of raising them",
3492:     )
3493:     parser.add_argument(
3494:         "--output",
3495:         help="Overrides the output filename",
3496:     )
3497:     parser.add_argument(
3498:         "--output-directory",
3499:         help="Overrides the directory to place output files.",
3500:     )
3501:     parser.add_argument(
3502:         "--disable-output",
3503:         action="store_true",
3504:         help="Disable writing of output files, e.g., for warm-up runs",
3505:     )
3506:     parser.add_argument(
3507:         "--baseline",
3508:         help="Compare with a prior --output",
3509:     )
3510:     parser.add_argument(
3511:         "--part",
3512:         default=None,
3513:         help="Specify the part of the model to run.",
3514:     )
3515:     parser.add_argument(
3516:         "--export-profiler-trace",
3517:         action="store_true",
3518:         help="exports trace of kineto profiler",
3519:     )
3520:     parser.add_argument(
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 3521-3600
````python
3521:         "--profiler-trace-name",
3522:         "--profiler_trace_name",
3523:         help="Overwrites exported trace name",
3524:     )
3525:     parser.add_argument(
3526:         "--profile-details", action="store_true", help="More detailed profiler trace."
3527:     )
3528:     parser.add_argument(
3529:         "--export-perfdoctor",
3530:         action="store_true",
3531:         help="Export Chrome trace to perf doctor. (internal only)",
3532:     )
3533:     parser.add_argument(
3534:         "--diff-branch",
3535:         default=diff_branch_default,
3536:         help="delta current branch against given branch.",
3537:     )
3538:     parser.add_argument(
3539:         "--tag", default=None, help="Specify a tag to be included in csv files."
3540:     )
3541:     parser.add_argument(
3542:         "--explain",
3543:         action="store_true",
3544:         help="print some graph/op statistics during the run, similar to .explain()",
3545:     )
3546:     parser.add_argument(
3547:         "--stats",
3548:         action="store_true",
3549:         help="print graph counter stats",
3550:     )
3551:     parser.add_argument(
3552:         "--use-warm-peak-memory",
3553:         "--use_warm_peak_memory",
3554:         action="store_true",
3555:         help="Measure peak memory using a warm run to reduce autotuning noise",
3556:     )
3557:     parser.add_argument(
3558:         "--print-memory",
3559:         action="store_true",
3560:         help="print extra memory statistics",
3561:     )
3562:     parser.add_argument(
3563:         "--print-compilation-time",
3564:         action="store_true",
3565:         help="print compilation latency",
3566:     )
3567:     parser.add_argument(
3568:         "--print-dataframe-summary",
3569:         action="store_true",
3570:         help="print dataframe result used for calculating accuracy",
3571:     )
3572:     parser.add_argument(
3573:         "--disable-cudagraphs",
3574:         action="store_true",
3575:         help="Disables cudagraphs for Inductor",
3576:     )
3577:     parser.add_argument(
3578:         "--disable-split-reductions",
3579:         action="store_true",
3580:         help="Disables split reductions for Inductor",
3581:     )
3582:     parser.add_argument(
3583:         "--disable-persistent-reductions",
3584:         action="store_true",
3585:         help="Disables split reductions for Inductor",
3586:     )
3587:     parser.add_argument(
3588:         "--disable-divisible-by-16",
3589:         action="store_true",
3590:         help="Disables divisible by 16 hint to Triton for Inductor",
3591:     )
3592:     parser.add_argument(
3593:         "--inductor-compile-mode",
3594:         default=None,
3595:         help="torch.compile mode argument for inductor runs.",
3596:     )
3597:     parser.add_argument(
3598:         "--print-graph-breaks",
3599:         action="store_true",
3600:         help="Show a warning whenever graph break",
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 3601-3680
````python
3601:     )
3602:     parser.add_argument(
3603:         "--log-graph-breaks",
3604:         action="store_true",
3605:         help="log graph breaks in a file",
3606:     )
3607:     parser.add_argument(
3608:         "--trace-on-xla",
3609:         action="store_true",
3610:         help="Whether to trace the model on XLA or on eager device",
3611:     )
3612:     parser.add_argument(
3613:         "--xla-tolerance",
3614:         type=float,
3615:         default=1e-2,
3616:         help="XLA needs a loose tolerance to pass the correctness check",
3617:     )
3618:     parser.add_argument(
3619:         "--collect-outputs",
3620:         action="store_true",
3621:         help="""Whether to collect outputs for training. Set this to true if we
3622:         want to verify the numerical correctness of graidents. But that may
3623:         cause time measurement not accurate""",
3624:     )
3625:     parser.add_argument(
3626:         "--enable-activation-checkpointing",
3627:         action="store_true",
3628:         help="Enables activation checkpointing for HF models",
3629:     )
3630:     parser.add_argument("--timing", action="store_true", help="Emits phase timing")
3631: 
3632:     parser.add_argument(
3633:         "--progress",
3634:         action="store_true",
3635:         help="Print n/k models message between each model run.",
3636:     )
3637: 
3638:     parser.add_argument(
3639:         "--timeout",
3640:         type=int,
3641:         default=2000,
3642:         help="timeout (second) for benchmarking.",
3643:     )
3644: 
3645:     parser.add_argument(
3646:         "--per_process_memory_fraction",
3647:         type=float,
3648:         default=1,
3649:         help="Set per-process GPU memory fraction (limit) for reducing usable size and reproducing OOMs",
3650:     )
3651: 
3652:     parser.add_argument(
3653:         "--no-translation-validation",
3654:         action="store_true",
3655:         help="Disable translation validation for accuracy builds.",
3656:     )
3657: 
3658:     parser.add_argument(
3659:         "--minify",
3660:         action="store_true",
3661:         help="Enable minification when failure is below tolerance. Save repro script for each model.",
3662:     )
3663: 
3664:     parser.add_argument(
3665:         "--compiled-autograd",
3666:         action="store_true",
3667:         help="Enables compiled autograd on compiled benchmark",
3668:     )
3669: 
3670:     parser.add_argument(
3671:         "--profile_dynamo_cache_lookup",
3672:         "--profile-dynamo-cache-lookup",
3673:         action="store_true",
3674:         help="profiles TorchDynamo cache lookup",
3675:     )
3676: 
3677:     parser.add_argument(
3678:         "--snapshot-memory",
3679:         "--snapshot_memory",
3680:         action="store_true",
````
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 3681-3760
````python
3681:         help="Enables Memory Snapshot tool for memory deep dives: https://pytorch.org/blog/understanding-gpu-memory-1/",
3682:     )
3683: 
3684:     parser.add_argument(
3685:         "--retain-output",
3686:         action="store_true",
3687:         help="Enables appending to the already existing output file if it exists \
3688:             instead of deleting it and creating a new one.",
3689:     )
3690: 
3691:     parser.add_argument(
3692:         "--caching-precompile",
3693:         action="store_true",
3694:         help="Enables caching precompile, serializing artifacts to DynamoCache between runs",
3695:     )
3696: 
3697:     parser.add_argument(
3698:         "--save-model-outputs-to",
3699:         default="",
3700:         help="Specify the path to save model output to so we can load later for comparison",
3701:     )
3702:     parser.add_argument(
3703:         "--compare-model-outputs-with",
3704:         default="",
3705:         help="Specify the path for the saved model outputs to compare against",
3706:     )
3707: 
3708:     group_latency = parser.add_mutually_exclusive_group()
3709:     group_latency.add_argument(
3710:         "--cold-start-latency",
3711:         "--cold_start_latency",
3712:         action="store_true",
3713:         help="Use a fresh triton cachedir when running each model, to force cold-start compile.",
3714:     )
3715:     group_latency.add_argument(
3716:         "--warm-start-latency",
3717:         "--warm_start_latency",
3718:         action="store_true",
3719:         help="Run model(s) twice and preserve caches in between to enable a 'warm start' on the 2nd run",
3720:     )
3721: 
3722:     group_fuser = parser.add_mutually_exclusive_group()
3723:     # --nvfuser is now the default, keep the option to not break scripts
3724:     group_fuser.add_argument("--nvfuser", action="store_true", help=argparse.SUPPRESS)
3725:     group_fuser.add_argument("--nnc", action="store_true", help="enable NNC for GPUs")
3726: 
3727:     group_prec = parser.add_mutually_exclusive_group()
3728:     group_prec.add_argument("--float16", action="store_true", help="cast model to fp16")
3729:     group_prec.add_argument(
3730:         "--bfloat16", action="store_true", help="cast model to bf16"
3731:     )
3732:     group_prec.add_argument("--float32", action="store_true", help="cast model to fp32")
3733:     group_prec.add_argument(
3734:         "--amp", action="store_true", help="use automatic mixed precision"
3735:     )
3736:     parser.add_argument(
3737:         "--amp-dtype",
3738:         choices=("bfloat16", "float16"),
3739:         help="the data type used with automatic mixed precision",
3740:     )
3741:     group_printout = parser.add_mutually_exclusive_group()
3742:     group_printout.add_argument(
3743:         "--verbose", "-v", action="store_true", help="enable verbose debug printouts"
3744:     )
3745:     group_printout.add_argument(
3746:         "--quiet", "-q", action="store_true", help="suppress debug printouts"
3747:     )
3748: 
3749:     group = parser.add_mutually_exclusive_group()
3750:     group.add_argument(
3751:         "--coverage", action="store_true", help="(default) " + help(coverage_experiment)
3752:     )
3753:     group.add_argument(
3754:         "--overhead", action="store_true", help=help(overhead_experiment)
3755:     )
3756:     group.add_argument(
3757:         "--speedup-dynamo-ts",
3758:         action="store_true",
3759:         help="TorchDynamo frontend with torchscript backend",
3760:     )
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 3761-3840
````python
3761:     group.add_argument(
3762:         "--speedup-fx2trt", action="store_true", help=help(speedup_experiment_fx2trt)
3763:     )
3764:     group.add_argument(
3765:         "--speedup-fx2trt-fp16",
3766:         action="store_true",
3767:         help=help(speedup_experiment_fx2trt),
3768:     )
3769:     group.add_argument(
3770:         "--print-fx",
3771:         action="store_true",
3772:         help="Print fx traces captured from model",
3773:     )
3774:     group.add_argument(
3775:         "--print-aten-ops",
3776:         action="store_true",
3777:         help="Print traces of aten ops captured by AOT autograd",
3778:     )
3779:     group.add_argument(
3780:         "--inductor",
3781:         action="store_true",
3782:         help="Measure speedup with TorchInductor",
3783:     )
3784:     group.add_argument(
3785:         "--optimus",
3786:         choices=["vertical_opt", "horizontal_opt", "all"],
3787:         default=None,
3788:         help="Measure speedup of Optimus with TorchInductor baseline",
3789:     )
3790:     group.add_argument(
3791:         "--quantization",
3792:         choices=[
3793:             "int8dynamic",
3794:             "int8weightonly",
3795:             "int4weightonly",
3796:             "noquant",
3797:         ],
3798:         default=None,
3799:         help="Measure speedup of torchao quantization with TorchInductor baseline",
3800:     )
3801:     group.add_argument(
3802:         "--export",
3803:         action="store_true",
3804:         help="Measure pass rate with export",
3805:     )
3806:     group.add_argument(
3807:         "--export-aot-inductor",
3808:         action="store_true",
3809:         help="Measure pass rate with Export+AOTInductor",
3810:     )
3811:     group.add_argument(
3812:         "--aot-precompile",
3813:         action="store_true",
3814:         help="Measure pass rate with AOT Precompile",
3815:     )
3816:     group.add_argument(
3817:         "--export-nativert",
3818:         action="store_true",
3819:         help="Measure pass rate with Export+NativeRT",
3820:     )
3821:     group.add_argument(
3822:         "--torchscript-jit-trace",
3823:         action="store_true",
3824:         help="Measure pass rate with TorchScript jit.trace",
3825:     )
3826:     group.add_argument(
3827:         "--xla", action="store_true", help="Compare TorchXLA to eager PyTorch"
3828:     )
3829:     group.add_argument(
3830:         "--backend",
3831:         choices=torch._dynamo.list_backends(exclude_tags=None),
3832:         help="measure speedup with a given backend",
3833:     )
3834:     group.add_argument("--nothing", action="store_true", help=help(null_experiment))
3835:     group.add_argument(
3836:         "--log-conv-args",
3837:         action="store_true",
3838:         help="Dump convolution input/weight/bias's shape/stride/dtype and other options to json",
3839:     )
3840:     group.add_argument(
````
- EN: This range contributes implementation details for the file goal: Implements benchmark definitions, helpers, or runners for performance measurement.
- CN: 该范围为文件目标提供实现细节：实现用于性能测量的基准定义、辅助工具或运行器。

### Lines 3841-3920
````python
3841:         "--recompile-profiler",
3842:         "--recompile_profiler",
3843:         action="store_true",
3844:         help="Run the dynamo recompilation profiler on each model.",
3845:     )
3846:     group.add_argument(
3847:         "--find-batch-sizes",
3848:         action="store_true",
3849:         help="finds the largest batch size that could fit on GPUs",
3850:     )
3851: 
3852:     mode_group = parser.add_mutually_exclusive_group(required=True)
3853:     mode_group.add_argument(
3854:         "--accuracy",
3855:         action="store_true",
3856:         help="Checks accuracy with small batch size and eval mode",
3857:     )
3858:     mode_group.add_argument(
3859:         "--performance", action="store_true", help="Measures performance speedup"
3860:     )
3861:     mode_group.add_argument(
3862:         "--tolerance",
3863:         action="store_true",
3864:         help="extracts the tolerance for each model with small batch size and eval mode",
3865:     )
3866:     run_mode_group = parser.add_mutually_exclusive_group(required=True)
3867:     run_mode_group.add_argument(
3868:         "--training",
3869:         action="store_true",
3870:         help="Performs training",
3871:     )
3872:     run_mode_group.add_argument(
3873:         "--inference", action="store_true", help="Performs inference"
3874:     )
3875:     parsed = parser.parse_args(args)
3876:     if parsed.batch_invariant and not parsed.accuracy:
3877:         parser.error("--batch-invariant requires --accuracy")
3878:     return parsed
3879: 
3880: 
3881: def process_caching_precompile():
3882:     """
3883:     After every process_entry, save precompile artifacts to DynamoCache
3884:     """
3885:     if not torch._dynamo.config.caching_precompile:
3886:         raise AssertionError(
3887:             "Caching precompile should be enabled with --caching-precompile"
3888:         )
3889:     from torch._dynamo.precompile_context import PrecompileContext
3890: 
3891:     debug_info = PrecompileContext.save_to_dynamo_cache()
3892:     print(
3893:         f"Saved {len(debug_info['dynamo'])} precompile artifacts with {len(debug_info['backends'])} backends"
3894:     )
3895: 
3896: 
3897: def process_entry(rank, runner, original_dir, args):
3898:     args.rank = rank
3899:     with maybe_init_distributed(
3900:         args.init_distributed,
3901:         rank=rank,
3902:         world_size=args.world_size,
3903:         port=args.distributed_master_port,
3904:     ):
3905:         result = run(runner, args, original_dir)
3906:         if args.caching_precompile:
3907:             process_caching_precompile()
3908:         return result
3909: 
3910: 
3911: def maybe_fresh_cache(args):
3912:     cache_dir_assigned = "TORCHINDUCTOR_CACHE_DIR" in os.environ
3913:     if not cache_dir_assigned and (
3914:         args.cold_start_latency or args.warm_start_latency or args.ci
3915:     ):
3916:         return fresh_cache()
3917:     else:
3918:         return contextlib.nullcontext()
3919: 
3920: 
````
- EN: Handles module imports such as `torch._dynamo.precompile_context`.
- CN: 处理模块导入，例如 `torch._dynamo.precompile_context`。
- EN: Implements callable logic such as `process_caching_precompile`, `process_entry`, `maybe_fresh_cache`.
- CN: 实现可调用逻辑，例如 `process_caching_precompile`, `process_entry`, `maybe_fresh_cache`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 3921-4000
````python
3921: def main(runner, original_dir=None, args=None):
3922:     if original_dir:
3923:         os.chdir(original_dir)
3924:     args = parse_args() if not args else parse_args(args)
3925:     if args.baseline:
3926:         args.baseline = os.path.abspath(args.baseline)
3927: 
3928:     if should_diff_branch(args):
3929:         import git
3930: 
3931:         # We do this here so we error out earlier if there's an issue
3932:         repo = git.Repo()
3933:         if repo.is_dirty():
3934:             raise RuntimeError(
3935:                 "--diff-branch called on dirty branch. Commit, stash, or reset."
3936:             )
3937:         main_branch = repo.active_branch.name
3938:         if main_branch == args.diff_branch:
3939:             raise RuntimeError(
3940:                 f"--diff-branch: current branch is same as {args.diff_branch} branch, what are you diffing?"
3941:             )
3942: 
3943:     with maybe_fresh_cache(args):
3944:         if args.caching_precompile:
3945:             os.environ["TORCH_CACHING_PRECOMPILE"] = "1"
3946:             torch._dynamo.config.caching_precompile = True
3947: 
3948:         args.init_distributed = args.only and args.multiprocess
3949:         if args.init_distributed:
3950:             # NB: Do NOT query device count before CUDA initialization; we're
3951:             # going to overwrite CUDA_VISIBLE_DEVICES and this will result in
3952:             # https://github.com/pytorch/pytorch/issues/107300
3953:             device_count = torch.cuda.device_count()
3954:             if device_count <= 1:
3955:                 log.warning(
3956:                     "The use multiprocess flag is set but there are <= 1 devices available."
3957:                 )
3958:             # multiprocess path
3959:             args.world_size = device_count
3960:             mp.spawn(
3961:                 process_entry, args=(runner, original_dir, args), nprocs=device_count
3962:             )
3963:         elif args.only and args.warm_start_latency:
3964:             # Warm start mode. Enable FX graph caching and perform back-to-back runs in
3965:             # separate processes (but ensure the inductor cache is preserved across runs).
3966:             env = os.environ.copy()
3967:             env["TORCHINDUCTOR_FX_GRAPH_CACHE"] = "1"
3968:             cmd = [sys.executable] + sys.argv
3969:             cmd.remove("--warm-start-latency")
3970: 
3971:             print(f"Performing cold-start run for {args.only}")
3972:             warmup_cmd = cmd + ["--repeat=1", "--disable-output"]
3973:             subprocess.check_call(warmup_cmd, timeout=args.timeout, env=env)
3974: 
3975:             print(f"Performing warm-start run for {args.only}")
3976:             subprocess.check_call(cmd, timeout=args.timeout, env=env)
3977:         else:
3978:             # single process path just uses the main process
3979:             args.world_size = 1
3980:             if args.compare_backed_unbacked:
3981:                 _run_compare_backed_unbacked(runner, args)
3982:             else:
3983:                 process_entry(0, runner, original_dir, args)
3984: 
3985: 
3986: def _run_compare_backed_unbacked(runner, args):
3987:     """Run backed and unbacked per-model, alternating, and compare speedup."""
3988:     import re
3989:     import subprocess
3990: 
3991:     def print_comparison(all_results):
3992:         print(f"\n{'=' * 80}", flush=True)
3993:         print("COMPARISON", flush=True)
3994:         print(f"{'=' * 80}", flush=True)
3995:         print(
3996:             f"  {'model':<40s} {'backed_ms':>10s} {'unbacked_ms':>11s} {'diff':>8s}",
3997:             flush=True,
3998:         )
3999:         print(f"  {'-' * 40} {'-' * 10} {'-' * 11} {'-' * 8}", flush=True)
4000:         for name, modes in all_results.items():
````
- EN: Handles module imports such as `git`, `re`, `subprocess`.
- CN: 处理模块导入，例如 `git`, `re`, `subprocess`。
- EN: Implements callable logic such as `main`, `_run_compare_backed_unbacked`, `print_comparison`.
- CN: 实现可调用逻辑，例如 `main`, `_run_compare_backed_unbacked`, `print_comparison`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 4001-4080
````python
4001:             b_ms = modes.get("backed_ms")
4002:             u_ms = modes.get("unbacked_ms")
4003:             if b_ms is not None and u_ms is not None:
4004:                 ms_diff_pct = (u_ms - b_ms) / b_ms * 100
4005:                 print(
4006:                     f"  {name:<40s} {b_ms:>10.3f} {u_ms:>11.3f} {ms_diff_pct:>+7.1f}%",
4007:                     flush=True,
4008:                 )
4009:             elif b_ms is not None:
4010:                 print(
4011:                     f"  {name:<40s} {b_ms:>10.3f} {'N/A':>11s} {'N/A':>8s}", flush=True
4012:                 )
4013:             elif u_ms is not None:
4014:                 print(
4015:                     f"  {name:<40s} {'N/A':>10s} {u_ms:>11.3f} {'N/A':>8s}", flush=True
4016:                 )
4017:             else:
4018:                 backed = (
4019:                     "FAILED" if "backed" not in modes else f"{modes['backed']:.3f}x"
4020:                 )
4021:                 unbacked = (
4022:                     "FAILED" if "unbacked" not in modes else f"{modes['unbacked']:.3f}x"
4023:                 )
4024:                 print(
4025:                     f"  {name:<40s} {backed:>10s} {unbacked:>11s} {'N/A':>8s}",
4026:                     flush=True,
4027:                 )
4028:         print(f"{'=' * 80}", flush=True)
4029: 
4030:     # Build base command, stripping --compare-backed-unbacked, --only, --filter and their values
4031:     # Handles both space-separated (--filter VALUE) and equals-separated (--filter=VALUE) forms
4032:     filtered = []
4033:     skip_next = False
4034:     for a in sys.argv:
4035:         if a == "--compare-backed-unbacked":
4036:             continue
4037:         if skip_next:
4038:             skip_next = False
4039:             continue
4040:         if a == "--only" or a.startswith("--only="):
4041:             if "=" not in a:
4042:                 skip_next = True
4043:             continue
4044:         if a == "--filter" or a.startswith("--filter="):
4045:             if "=" not in a:
4046:                 skip_next = True
4047:             continue
4048:         filtered.append(a)
4049:     base_cmd = [sys.executable, "-B"] + filtered
4050: 
4051:     # Get model list from runner
4052:     runner.args = args
4053:     args.filter = args.filter or [r"."]
4054:     args.exclude = args.exclude or [r"^$"]
4055:     args.exclude_exact = args.exclude_exact or []
4056:     models = list(runner.iter_model_names(args))
4057: 
4058:     if args.only:
4059:         models = [args.only]
4060: 
4061:     all_results = {}
4062:     for model in models:
4063:         print(f"\n--- {model} ---", flush=True)
4064:         for mode, flag in [
4065:             ("backed", "--dynamic-batch-only"),
4066:             ("unbacked", "--unbacked-batch-only"),
4067:         ]:
4068:             cmd = base_cmd + ["--only", model, flag, "--_print-latency-ms"]
4069:             print(f"  {mode}...", end=" ", flush=True)
4070:             try:
4071:                 proc = subprocess.Popen(
4072:                     cmd, stdout=subprocess.PIPE, stderr=subprocess.PIPE, text=True
4073:                 )
4074:                 stdout, stderr = proc.communicate(timeout=600)
4075:             except subprocess.TimeoutExpired:
4076:                 proc.kill()
4077:                 print("TIMEOUT", flush=True)
4078:                 continue
4079:             except Exception as e:
4080:                 print(f"ERROR ({e})", flush=True)
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 4081-4160
````python
4081:                 continue
4082: 
4083:             speedup_match = re.search(r"(\d+\.\d+)x", stdout)
4084:             latency_match = re.search(r"([\d.]+) ms, ([\d.]+) ms,", stdout)
4085:             if speedup_match:
4086:                 speedup = float(speedup_match.group(1))
4087:                 eager_ms = float(latency_match.group(1)) if latency_match else None
4088:                 compiled_ms = float(latency_match.group(2)) if latency_match else None
4089:                 extra = ""
4090:                 if eager_ms and compiled_ms:
4091:                     extra = f" (eager={eager_ms:.3f} ms, compiled={compiled_ms:.3f} ms)"
4092:                 print(f"{speedup:.3f}x{extra}", flush=True)
4093:                 if model not in all_results:
4094:                     all_results[model] = {}
4095:                 all_results[model][mode] = speedup
4096:                 if eager_ms is not None:
4097:                     all_results[model][f"{mode}_eager_ms"] = eager_ms
4098:                 if compiled_ms is not None:
4099:                     all_results[model][f"{mode}_ms"] = compiled_ms
4100:             else:
4101:                 err_match = re.search(
4102:                     r"(Error|Exception|Traceback).*", stdout + stderr, re.IGNORECASE
4103:                 )
4104:                 if err_match:
4105:                     print("FAILED", flush=True)
4106:                 else:
4107:                     print("SKIP", flush=True)
4108: 
4109:         # Print running diff for this model
4110:         if (
4111:             model in all_results
4112:             and "backed_ms" in all_results[model]
4113:             and "unbacked_ms" in all_results[model]
4114:         ):
4115:             b_ms = all_results[model]["backed_ms"]
4116:             u_ms = all_results[model]["unbacked_ms"]
4117:             ms_diff_pct = (u_ms - b_ms) / b_ms * 100
4118:             print(
4119:                 f"  => diff: {ms_diff_pct:+.1f}% ({b_ms:.3f} ms vs {u_ms:.3f} ms)",
4120:                 flush=True,
4121:             )
4122:         elif (
4123:             model in all_results
4124:             and "backed" in all_results[model]
4125:             and "unbacked" in all_results[model]
4126:         ):
4127:             b = all_results[model]["backed"]
4128:             u = all_results[model]["unbacked"]
4129:             diff_pct = (u - b) / b * 100
4130:             print(f"  => diff: {diff_pct:+.1f}% (ratio-based, no ms data)", flush=True)
4131: 
4132:     print_comparison(all_results)
4133: 
4134: 
4135: def write_csv_when_exception(args, name: str, status: str, device=None):
4136:     print(status)
4137:     placeholder_batch_size = 0
4138:     devices = [device] if device is not None else args.devices
4139:     if args.accuracy:
4140:         headers = ["dev", "name", "batch_size", "accuracy"]
4141:         rows = [[device, name, placeholder_batch_size, status] for device in devices]
4142:     elif args.performance:
4143:         headers = ["dev", "name", "batch_size", "speedup", "abs_latency"]
4144:         rows = [[device, name, placeholder_batch_size, 0.0, 0.0] for device in devices]
4145:     else:
4146:         headers = []
4147:         rows = [[device, name, placeholder_batch_size, 0.0] for device in devices]
4148: 
4149:     for row in rows:
4150:         write_outputs(output_filename, headers, row)
4151: 
4152: 
4153: def setup_determinism(args):
4154:     if args.only is not None and args.only not in {
4155:         "alexnet",
4156:         "Background_Matting",
4157:         "pytorch_CycleGAN_and_pix2pix",
4158:         "pytorch_unet",
4159:         "Super_SloMo",
4160:         "vgg16",
````
- EN: Implements callable logic such as `write_csv_when_exception`, `setup_determinism`.
- CN: 实现可调用逻辑，例如 `write_csv_when_exception`, `setup_determinism`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 4161-4240
````python
4161:         # https://github.com/pytorch/pytorch/issues/96724
4162:         "Wav2Vec2ForCTC",
4163:         "Wav2Vec2ForPreTraining",
4164:         "sam",
4165:         "sam_fast",
4166:         "resnet50_quantized_qat",
4167:         "mobilenet_v2_quantized_qat",
4168:         "detectron2_maskrcnn",
4169:         "detectron2_maskrcnn_r_101_c4",
4170:         "detectron2_maskrcnn_r_101_fpn",
4171:         "detectron2_maskrcnn_r_50_c4",
4172:         "detectron2_maskrcnn_r_50_fpn",
4173:         "detectron2_fasterrcnn_r_101_c4",
4174:         "detectron2_fasterrcnn_r_101_dc5",
4175:         "detectron2_fasterrcnn_r_101_fpn",
4176:         "detectron2_fasterrcnn_r_50_c4",
4177:         "detectron2_fasterrcnn_r_50_dc5",
4178:         "detectron2_fasterrcnn_r_50_fpn",
4179:     }:
4180:         # some of the models do not support use_deterministic_algorithms
4181:         torch.use_deterministic_algorithms(True)
4182: 
4183:     if args.devices == ["rocm"]:
4184:         torch.use_deterministic_algorithms(True, warn_only=True)
4185: 
4186:     torch.backends.cudnn.deterministic = True
4187:     torch.backends.cudnn.benchmark = False
4188:     torch.backends.mkldnn.deterministic = True
4189:     os.environ["CUBLAS_WORKSPACE_CONFIG"] = ":4096:8"
4190:     torch.backends.cudnn.allow_tf32 = False
4191:     torch.backends.cuda.matmul.allow_tf32 = False
4192:     torch.backends.cuda.allow_fp16_bf16_reduction_math_sdp(False)
4193:     patch_torch_manual_seed()
4194: 
4195: 
4196: def setup_batch_invariant(args):
4197:     if not torch.cuda.is_available():
4198:         return
4199:     setup_determinism(args)
4200:     inductor_config.triton.cudagraphs = False
4201:     torch.backends.cuda.preferred_blas_library("cublaslt")
4202:     torch.backends.cuda.matmul.allow_fp16_reduced_precision_reduction = (False, False)
4203:     torch.backends.cuda.matmul.allow_bf16_reduced_precision_reduction = (False, False)
4204: 
4205: 
4206: def run(runner, args, original_dir=None):
4207:     # Pass the parsed args object to benchmark runner object
4208:     torch._dynamo.reset()
4209:     runner.args = args
4210: 
4211:     args.filter = args.filter or [r"."]
4212:     args.exclude = args.exclude or [r"^$"]
4213:     args.exclude_exact = args.exclude_exact or []
4214: 
4215:     if args.inductor:
4216:         if args.backend is not None:
4217:             raise AssertionError(f"--inductor conflicts with --backend={args.backend}")
4218:         args.backend = "inductor"
4219:     if args.optimus:
4220:         if args.backend is not None:
4221:             raise AssertionError(f"--optimus conflicts with --backend={args.backend}")
4222:         args.backend = "optimus"
4223:     if args.quantization:
4224:         if args.backend is not None:
4225:             raise AssertionError(
4226:                 f"--quantization conflicts with --backend={args.backend}"
4227:             )
4228:         args.backend = "torchao"
4229:     if args.dynamic_batch_only:
4230:         args.dynamic_shapes = True
4231:         torch._dynamo.config.assume_static_by_default = True
4232:     if args.unbacked_batch_only:
4233:         args.dynamic_shapes = True
4234:         torch._dynamo.config.assume_static_by_default = True
4235:     if args.dynamic_shapes:
4236:         if not args.dynamic_batch_only and not args.unbacked_batch_only:
4237:             torch._dynamo.config.assume_static_by_default = False
4238:     if args.compiled_autograd:
4239:         torch._dynamo.config.compiled_autograd = True
4240:     if args.propagate_real_tensors:
````
- EN: Implements callable logic such as `setup_batch_invariant`, `run`.
- CN: 实现可调用逻辑，例如 `setup_batch_invariant`, `run`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 4241-4320
````python
4241:         # TODO: Separate flag for data dependent
4242:         torch._dynamo.config.capture_scalar_outputs = True
4243:         torch._dynamo.config.capture_dynamic_output_shape_ops = True
4244:         torch._functorch.config.fake_tensor_propagate_real_tensors = True
4245:     if args.specialize_int:
4246:         torch._dynamo.config.specialize_int = True
4247:     if args.ci:
4248:         if args.accuracy:
4249:             # Run fewer iterations when checking accuracy
4250:             args.repeat = min(args.repeat, 2)
4251: 
4252:             # Set translation validation on by default on CI accuracy runs.
4253:             torch.fx.experimental._config.translation_validation = True
4254: 
4255:     if args.ddp:
4256:         if not args.training:
4257:             raise AssertionError("DDP benchmark requires --training mode")
4258:         torch._dynamo.config.optimize_ddp = args.optimize_ddp_mode
4259:         if args.only == "dlrm":
4260:             log.error(
4261:                 "DLRM+DDP is unsupported as it requires sharding the embedding layer separately from DDP"
4262:             )
4263:             return sys.exit(-1)
4264:     if args.deterministic and not args.accuracy:
4265:         setup_determinism(args)
4266: 
4267:     if args.accuracy:
4268:         # Use small batch size. We use >1 batch size to ensure we test
4269:         # batch_norm type of operators that work on batch dims.
4270:         # TODO - Go through the failures for batch size = 2
4271:         if args.batch_size is None:
4272:             if args.batch_invariant:
4273:                 if runner.suite_name == "huggingface":
4274:                     args.batch_size = 8
4275:                 elif runner.suite_name == "torchbench":
4276:                     args.batch_size = 8
4277:                 else:
4278:                     args.batch_size = 16
4279:             elif runner.suite_name == "huggingface":
4280:                 args.batch_size = 1
4281:             elif runner.suite_name == "torchbench":
4282:                 args.batch_size = 4
4283:             else:
4284:                 # Larger batch size of TIMM models to have stable batch_norm
4285:                 if runner.suite_name != "timm_models":
4286:                     raise AssertionError(
4287:                         f"expected runner.suite_name to be 'timm_models', got {runner.suite_name}"
4288:                     )
4289:                 args.batch_size = 8
4290: 
4291:         # Remove sources of randomness
4292:         if runner.suite_name not in ("timm_models", "huggingface"):
4293:             # TODO - Using train mode for timm_models and HF models. Move to train mode for Torchbench as well.
4294:             args.use_eval_mode = True
4295:         inductor_config.fallback_random = True
4296: 
4297:         setup_determinism(args)
4298:         if args.batch_invariant:
4299:             setup_batch_invariant(args)
4300: 
4301:         if args.only is not None and args.only in {
4302:             "nvidia_deeprecommender",
4303:         }:
4304:             # These seem unhappy with numerics of larger cuBLASLt workspace
4305:             torch.backends.cuda.matmul.allow_bf16_reduced_precision_reduction = False
4306:             torch.backends.cuda.matmul.allow_fp16_reduced_precision_reduction = False
4307: 
4308:         # Some models e.g. yolov3 assert batch size on n_gpus
4309:         if "CUDA_VISIBLE_DEVICES" not in os.environ and not args.multiprocess:
4310:             args.device_index = "0"
4311: 
4312:         # Stricter check to disable fallbacks
4313:         args.suppress_errors = False
4314: 
4315:         if not args.disable_cudagraphs:
4316:             runner.skip_models.update(
4317:                 {
4318:                     # xfail: https://github.com/pytorch/pytorch/issues/145773
4319:                     "llama",
4320:                     "cm3leon_generate",
````
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 4321-4400
````python
4321:                     "modded_nanogpt",
4322:                 }
4323:             )
4324: 
4325:     if args.device_index is not None:
4326:         if args.multiprocess:
4327:             print("Cannot specify both --device_index and --multiprocess")
4328:             return sys.exit(-1)
4329:         os.environ["CUDA_VISIBLE_DEVICES"] = args.device_index
4330: 
4331:     elif args.performance:
4332:         # Ensure that we test on real scenarios
4333:         args.use_eval_mode = False
4334: 
4335:     if args.partition_id > args.total_partitions or args.partition_id < 0:
4336:         print("Invalid partition id")
4337:         return sys.exit(-1)
4338: 
4339:     if not args.devices:
4340:         if torch.cuda.is_available():
4341:             args.devices = ["cuda"]
4342:         else:
4343:             log.warning("torch.cuda.is_available() == False, using CPU")
4344:             args.devices = ["cpu"]
4345: 
4346:     if args.devices != ["cpu"] and (HAS_CUDA or HAS_XPU):
4347:         global synchronize
4348:         synchronize = torch.cuda.synchronize if HAS_CUDA else torch.xpu.synchronize
4349: 
4350:     if args.nnc:
4351:         torch._C._jit_override_can_fuse_on_cpu(True)
4352:         torch._C._jit_override_can_fuse_on_gpu(True)
4353:         torch._C._jit_set_texpr_fuser_enabled(True)
4354:         torch._C._jit_set_nvfuser_enabled(False)
4355: 
4356:     if args.threads:
4357:         torch.set_num_threads(args.threads)
4358: 
4359:     if args.verbose:
4360:         torch._logging.set_logs(dynamo=logging.DEBUG)
4361: 
4362:     if args.print_graph_breaks:
4363:         torch._logging.set_logs(graph_breaks=True)
4364: 
4365:     if args.quiet:
4366:         torch._logging.set_logs(dynamo=logging.ERROR)
4367: 
4368:     torch._dynamo.config.suppress_errors = args.suppress_errors
4369: 
4370:     if args.training:
4371:         runner.model_iter_fn = runner.forward_and_backward_pass
4372:         runner.skip_models.update(runner.skip_not_suitable_for_training_models)
4373:     else:
4374:         runner.model_iter_fn = runner.forward_pass
4375: 
4376:     if args.fast:
4377:         runner.skip_models.update(runner.slow_models)
4378: 
4379:     if args.devices == ["cpu"]:
4380:         arch = platform.machine()
4381:         runner.skip_models.update(runner.skip_models_for_cpu)
4382:         if arch == "aarch64":
4383:             runner.skip_models.update(runner.skip_models_for_cpu_aarch64)
4384:     elif args.devices == ["cuda"]:
4385:         runner.skip_models.update(runner.skip_models_for_cuda)
4386:     elif args.devices == ["xpu"]:
4387:         runner.skip_models.update(runner.skip_models_for_xpu)
4388: 
4389:     if not args.multiprocess:
4390:         runner.skip_models.update(runner.skip_multiprocess_models)
4391: 
4392:     if args.freezing:
4393:         if args.devices == ["cpu"]:
4394:             runner.skip_models.update(runner.skip_models_for_freezing_cpu)
4395:         elif args.devices == ["cuda"]:
4396:             runner.skip_models.update(runner.skip_models_for_freezing_cuda)
4397: 
4398:     if args.no_skip:
4399:         runner.skip_models.clear()
4400: 
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 4401-4480
````python
4401:     experiment = null_experiment
4402:     global \
4403:         current_name, \
4404:         current_device, \
4405:         current_batch_size, \
4406:         current_backend, \
4407:         current_mode, \
4408:         current_dtype, \
4409:         current_quantization, \
4410:         current_settings, \
4411:         output_filename, \
4412:         disable_output, \
4413:         optimize_ctx
4414:     optimize_ctx = contextlib.nullcontext()
4415: 
4416:     if args.disable_output:
4417:         disable_output = True
4418: 
4419:     if args.overhead:
4420:         optimize_ctx = torch._dynamo.optimize(dummy_fx_compile, nopython=args.nopython)
4421:         experiment = speedup_experiment
4422:         output_filename = "overheads.csv"
4423:     elif args.inductor:
4424:         inductor_config.debug = args.verbose
4425:         if args.threads:
4426:             inductor_config.cpp.threads = args.threads
4427: 
4428:         optimize_ctx = functools.partial(
4429:             torch.compile,
4430:             backend="inductor",
4431:             fullgraph=args.nopython,
4432:             mode=args.inductor_compile_mode,
4433:         )
4434:         experiment = speedup_experiment
4435:         output_filename = "inductor.csv"
4436:     elif args.export:
4437:         optimize_ctx = export
4438:         experiment = speedup_experiment
4439:         output_filename = "export.csv"
4440:     elif args.aot_precompile:
4441:         optimize_ctx = aot_precompile
4442:         experiment = speedup_experiment
4443:         output_filename = "aot_precompile.csv"
4444:     elif args.export_nativert:
4445:         optimize_ctx = export_nativert
4446:         experiment = speedup_experiment
4447:         output_filename = "export_nativert.csv"
4448:     elif args.torchscript_jit_trace:
4449:         optimize_ctx = torchscript_jit_trace
4450:         experiment = speedup_experiment
4451:         output_filename = "torchscript_jit_trace.csv"
4452:     elif args.xla:
4453:         (dev,) = args.devices
4454:         os.environ["PJRT_DEVICE"] = {"cuda": "GPU", "cpu": "CPU"}[dev]
4455:         torch._dynamo.mark_dynamic = MagicMock()
4456:         experiment = xla
4457:         output_filename = "xla.csv"
4458:     elif args.speedup_dynamo_ts:
4459:         optimize_ctx = torch._dynamo.optimize("ts", nopython=args.nopython)
4460:         experiment = speedup_experiment
4461:         output_filename = "speedup_dynamo_ts.csv"
4462:     elif args.prims_nvfuser:
4463:         optimize_ctx = torch._dynamo.optimize("prims_nvfuser", nopython=args.nopython)
4464:         experiment = speedup_experiment
4465:         backend_str = "prims_nvfuser"
4466:         output_filename = f"accuracy_aot_{backend_str}.csv"
4467:     elif args.print_fx:
4468:         optimize_ctx = torch._dynamo.optimize(
4469:             print_fx,
4470:             nopython=args.nopython,
4471:         )
4472:     elif args.print_aten_ops:
4473:         optimize_ctx = torch._dynamo.optimize(
4474:             print_aten_ops,
4475:             nopython=args.nopython,
4476:         )
4477:     elif args.nothing:
4478:         optimize_ctx = nothing
4479:         experiment = speedup_experiment
4480:         output_filename = "nothing.csv"
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 4481-4560
````python
4481:     elif args.backend or args.export_aot_inductor:
4482:         if args.export_aot_inductor:
4483:             if args.training:
4484:                 raise AssertionError("AOTInductor only supports inference")
4485:             optimize_ctx = functools.partial(
4486:                 export_aot_inductor, mode=args.inductor_compile_mode
4487:             )
4488: 
4489:             # AOTInductor doesn't support control flow yet
4490:             runner.skip_models.update(runner.skip_models_due_to_control_flow)
4491:             runner.skip_models.update(runner.skip_models_due_to_export_not_supported)
4492:         elif args.backend == "torchao":
4493:             if "cuda" not in args.devices:
4494:                 raise AssertionError(
4495:                     f"Quantization requires CUDA device, got devices={args.devices}"
4496:                 )
4497:             if not args.bfloat16:
4498:                 raise AssertionError("Quantization requires dtype bfloat16")
4499:             try:
4500:                 from torchao_backend import setup_baseline, torchao_optimize_ctx
4501:             except ImportError:
4502:                 try:
4503:                     from .torchao_backend import setup_baseline, torchao_optimize_ctx
4504:                 except ImportError:
4505:                     from userbenchmark.dynamo.dynamobench.torchao_backend import (
4506:                         setup_baseline,
4507:                         torchao_optimize_ctx,
4508:                     )
4509: 
4510:             setup_baseline()
4511:             baseline_ctx = functools.partial(
4512:                 torch.compile,
4513:                 backend="inductor",
4514:                 fullgraph=args.nopython,
4515:                 mode=args.inductor_compile_mode,
4516:             )
4517:             model_iter_fn = baseline_ctx(runner.model_iter_fn)
4518: 
4519:             # needed to avoid CUDAGraph fast-path warning / inconsistent timing when prior
4520:             # outputs still require backward (see torch._inductor.cudagraph_trees)
4521:             def model_iter_fn_and_mark_step(*args, **kwargs):
4522:                 torch.compiler.cudagraph_mark_step_begin()
4523:                 model_iter_fn(*args, **kwargs)
4524: 
4525:             runner.model_iter_fn = model_iter_fn_and_mark_step
4526:             optimize_ctx = torchao_optimize_ctx(args.quantization)
4527:         elif args.backend == "optimus":
4528:             from .optimus import get_baseline_ctx, get_optimus_optimize_ctx
4529: 
4530:             baseline_ctx = get_baseline_ctx(
4531:                 nopython=args.nopython, inductor_compile_mode=args.inductor_compile_mode
4532:             )
4533:             runner.model_iter_fn = baseline_ctx(runner.model_iter_fn)
4534:             optimize_ctx = get_optimus_optimize_ctx(
4535:                 args.optimus, args.nopython, args.inductor_compile_mode
4536:             )
4537:         else:
4538:             optimize_ctx = torch._dynamo.optimize(args.backend, nopython=args.nopython)
4539:         experiment = (
4540:             speedup_experiment
4541:             if args.backend not in ["torchao", "optimus"]
4542:             else latency_experiment
4543:         )
4544:         if args.accuracy:
4545:             output_filename = f"accuracy_{args.backend}.csv"
4546:         elif args.tolerance:
4547:             output_filename = f"tolerance_{args.backend}.csv"
4548:         else:
4549:             output_filename = f"speedup_{args.backend}.csv"
4550:     elif args.recompile_profiler:
4551:         output_filename = "recompile_profiler_log.csv"
4552:         experiment = recompile_profiler_experiment
4553:     else:
4554:         optimize_ctx = torch._dynamo.optimize(
4555:             fx_insert_profiling, nopython=args.nopython
4556:         )
4557:         experiment = coverage_experiment
4558:         output_filename = "coverage.csv"
4559: 
4560:     if args.only in runner.disable_cudagraph_models:
````
- EN: Handles module imports such as `torchao_backend`, `.torchao_backend`, `userbenchmark.dynamo.dynamobench.torchao_backend`, `.optimus`.
- CN: 处理模块导入，例如 `torchao_backend`, `.torchao_backend`, `userbenchmark.dynamo.dynamobench.torchao_backend`, `.optimus`。
- EN: Implements callable logic such as `model_iter_fn_and_mark_step`.
- CN: 实现可调用逻辑，例如 `model_iter_fn_and_mark_step`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 4561-4640
````python
4561:         args.disable_cudagraphs = True
4562: 
4563:     if (
4564:         args.inductor
4565:         or args.backend == "inductor"
4566:         or args.export_aot_inductor
4567:         or args.backend == "optimus"
4568:     ):
4569:         inductor_config.triton.cudagraphs = not args.disable_cudagraphs
4570:         inductor_config.triton.persistent_reductions = (
4571:             not args.disable_persistent_reductions
4572:         )
4573:         inductor_config.split_reductions = not args.disable_split_reductions
4574:         inductor_config.triton.divisible_by_16 = not args.disable_divisible_by_16
4575:         if args.inference:
4576:             inductor_config.freezing = args.freezing
4577:         if args.inductor_config:
4578:             for config in args.inductor_config:
4579:                 key, value = config.split("=")
4580:                 typ = type(inductor_config.__getattr__(key))
4581:                 if issubclass(typ, bool):
4582:                     if value not in ("0", "1", "True", "False"):
4583:                         raise AssertionError(
4584:                             f"expected bool value for {key}, got {value}"
4585:                         )
4586:                     value = value in ("1", "True")
4587:                 elif issubclass(typ, (str, int, float)):
4588:                     value = typ(value)
4589:                 else:
4590:                     raise NotImplementedError(typ)
4591:                 inductor_config.__setattr__(key, value)
4592: 
4593:     runner.setup_amp()
4594: 
4595:     if args.output:
4596:         output_filename = args.output
4597: 
4598:     if output_filename:
4599:         if args.output_directory:
4600:             output_filename = os.path.join(args.output_directory, output_filename)
4601:         else:
4602:             output_filename = os.path.join(
4603:                 torch._dynamo.config.base_dir, output_filename
4604:             )
4605: 
4606:     if args.find_batch_sizes and args.only:
4607:         for device in args.devices:
4608:             batch_size = runner.batch_size_finder(device, args.only)
4609:             print(args.only, batch_size)
4610:             write_outputs(output_filename, [], [args.only, batch_size])
4611:         return
4612: 
4613:     should_profile_details = args.profile_details
4614:     args.profile_details = {}
4615:     if args.export_profiler_trace:
4616:         if should_profile_details:
4617:             args.profile_details = {
4618:                 "record_shapes": True,
4619:                 "profile_memory": True,
4620:                 "with_stack": True,
4621:                 "with_modules": True,
4622:                 "activities": [
4623:                     torch.profiler.ProfilerActivity.CPU,
4624:                     torch.profiler.ProfilerActivity.CUDA,
4625:                 ],
4626:             }
4627: 
4628:         if args.profiler_trace_name is None:
4629:             if args.backend:
4630:                 args.profiler_trace_name = args.backend
4631:             elif args.inductor:
4632:                 args.profiler_trace_name = "inductor"
4633:             else:
4634:                 args.profiler_trace_name = "profile"
4635:         else:
4636:             args.profiler_trace_name = args.profiler_trace_name
4637: 
4638:     if args.no_translation_validation:
4639:         # Overwrite 'translation_validation' config, if specified.
4640:         torch.fx.experimental._config.translation_validation = False
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 4641-4720
````python
4641: 
4642:     experiment = functools.partial(experiment, args)
4643: 
4644:     if args.only and should_diff_branch(args):
4645:         import git
4646: 
4647:         repo = git.Repo()
4648:         main_branch = repo.active_branch.name
4649:         try:
4650:             # Adding diff-branch again to the args will override previous value
4651:             call_args = (
4652:                 [sys.executable] + sys.argv + [f"--diff-branch={diff_branch_default}"]
4653:             )
4654:             # Run for main branch
4655:             subprocess.check_call(call_args + [f"--tag={main_branch}"])
4656:             # Run for comparison branch
4657:             repo.git.checkout(args.diff_branch)
4658:             subprocess.check_call(call_args + [f"--tag={args.diff_branch}"])
4659:         finally:
4660:             # Go back to main branch
4661:             repo.git.checkout(main_branch)
4662:     elif args.only:
4663:         model_name = args.only
4664:         for device in args.devices:
4665:             batch_size = args.batch_size
4666:             if args.batch_size_file:
4667:                 batch_size = read_batch_size_from_file(
4668:                     args, args.batch_size_file, model_name
4669:                 )
4670:             if model_specified_by_path(args.only):
4671:                 model, example_inputs = load_model_from_path(args.only)
4672:                 name = model.__class__.__name__
4673:                 model = model.to(device=device)
4674:                 example_inputs = tree_map_only(
4675:                     torch.Tensor, lambda x: x.to(device=device), example_inputs
4676:                 )
4677:             else:
4678:                 name = model_name
4679:                 try:
4680:                     with tqdm(desc="loading model"):
4681:                         extra_args = []
4682:                         if hasattr(args, "rank") and hasattr(args, "world_size"):
4683:                             extra_args += [
4684:                                 "--rank",
4685:                                 str(args.rank),
4686:                                 "--world_size",
4687:                                 str(args.world_size),
4688:                             ]
4689: 
4690:                         if args.part:
4691:                             (
4692:                                 device,
4693:                                 name,
4694:                                 model,
4695:                                 example_inputs,
4696:                                 batch_size,
4697:                             ) = runner.load_model(
4698:                                 device,
4699:                                 model_name,
4700:                                 batch_size=batch_size,
4701:                                 part=args.part,
4702:                                 extra_args=extra_args,
4703:                             )
4704:                         else:
4705:                             if args.fsdp:
4706:                                 # Always load model on cpu for fsdp
4707:                                 # When initializing FSDP, we will use the cuda device if args.cuda is set
4708:                                 (
4709:                                     _,
4710:                                     name,
4711:                                     model,
4712:                                     example_inputs,
4713:                                     batch_size,
4714:                                 ) = runner.load_model(
4715:                                     "cpu",
4716:                                     model_name,
4717:                                     batch_size=batch_size,
4718:                                     extra_args=extra_args,
4719:                                 )
4720:                             else:
````
- EN: Handles module imports such as `git`.
- CN: 处理模块导入，例如 `git`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 4721-4800
````python
4721:                                 (
4722:                                     device,
4723:                                     name,
4724:                                     model,
4725:                                     example_inputs,
4726:                                     batch_size,
4727:                                 ) = runner.load_model(
4728:                                     device,
4729:                                     model_name,
4730:                                     batch_size=batch_size,
4731:                                     extra_args=extra_args,
4732:                                 )
4733:                 except Exception as e:
4734:                     import traceback
4735: 
4736:                     mode = "train" if args.training else "eval"
4737:                     print(f"{device:4} {mode:5} {name:34} ")
4738:                     print(traceback.format_exc())
4739:                     status = (
4740:                         "model_fail_to_load"
4741:                         if isinstance(e, NotImplementedError)
4742:                         else "eager_fail_to_run"
4743:                     )
4744:                     write_csv_when_exception(args, name, status, device)
4745:                     # NB: current_name/current_device not set, so pass
4746:                     # explicitly
4747:                     output_signpost(
4748:                         {"name": name, "dev": device},
4749:                         args,
4750:                         runner.suite_name,
4751:                         error=status,
4752:                     )
4753:                     continue  # bad benchmark implementation
4754: 
4755:             if args.trace_on_xla:
4756:                 xla_dev = xm.xla_device()
4757:                 model = model.to(device=xla_dev)
4758:                 example_inputs = tree_map_only(
4759:                     torch.Tensor, lambda x: x.to(device=xla_dev), example_inputs
4760:                 )
4761: 
4762:             current_name = name
4763:             current_device = device
4764:             current_batch_size = batch_size
4765:             current_backend = args.backend
4766:             current_mode = (
4767:                 "training" if args.training else "inference" if args.inference else ""
4768:             )
4769:             if args.float16:
4770:                 current_dtype = "float16"
4771:             elif args.bfloat16:
4772:                 current_dtype = "bfloat16"
4773:             elif args.float32:
4774:                 current_dtype = "float32"
4775:             elif args.amp:
4776:                 current_dtype = "amp"
4777:             else:
4778:                 current_dtype = ""
4779:             current_quantization = args.quantization
4780:             # Keep the remaining of the settings
4781:             current_settings = vars(args)
4782:             set_model_name(name)
4783: 
4784:             # Look for stuff that looks like batch size, and mark it dynamic.
4785:             # Better integration would integrate directly with benchmark suite
4786:             # but cannot conveniently do this
4787:             # NB: This must be done late enough so that we don't do more
4788:             # conversions on the inputs
4789:             # NB: Assumes only the first batch-y like dimension is the batch
4790:             marked = False
4791: 
4792:             def detect_and_mark_batch(t, use_unbacked=False):
4793:                 nonlocal marked
4794:                 for i, s in enumerate(t.size()):
4795:                     if s == batch_size:
4796:                         if use_unbacked:
4797:                             # Use duck_shape_id="batch" so all batch dimensions
4798:                             # share the same unbacked symbol
4799:                             torch._dynamo.decorators.mark_unbacked(
4800:                                 t, i, shape_id="batch", hint_override=batch_size, min=1
````
- EN: Handles module imports such as `traceback`.
- CN: 处理模块导入，例如 `traceback`。
- EN: Implements callable logic such as `detect_and_mark_batch`.
- CN: 实现可调用逻辑，例如 `detect_and_mark_batch`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 4801-4880
````python
4801:                             )
4802:                         else:
4803:                             torch._dynamo.maybe_mark_dynamic(t, i)
4804:                         marked = True
4805:                         break
4806: 
4807:             if (
4808:                 (args.dynamic_batch_only or args.unbacked_batch_only)
4809:                 and batch_size > 1
4810:                 and model_name not in CI_SKIP_DYNAMIC_BATCH_ONLY
4811:             ):
4812:                 mark_fn = functools.partial(
4813:                     detect_and_mark_batch, use_unbacked=args.unbacked_batch_only
4814:                 )
4815:                 tree_map_only(torch.Tensor, mark_fn, example_inputs)
4816:                 if not marked:
4817:                     raise AssertionError(
4818:                         f"nothing in example_inputs had a dim with {batch_size}"
4819:                     )
4820: 
4821:             if args.log_operator_inputs:
4822:                 log_operator_inputs(
4823:                     model, example_inputs, runner.model_iter_fn, name, args
4824:                 )
4825:                 continue
4826: 
4827:             if args.per_process_memory_fraction != 1:
4828:                 torch.cuda.set_per_process_memory_fraction(
4829:                     args.per_process_memory_fraction
4830:                 )
4831:             if model_name in DO_NOT_CAST_INPUTS:
4832:                 model, _ = runner.cast_based_on_args(model, example_inputs)
4833: 
4834:             else:
4835:                 model, example_inputs = runner.cast_based_on_args(model, example_inputs)
4836:             runner.setup_amp(current_device)
4837:             guard_ctx = contextlib.nullcontext()
4838:             if name in runner.guard_on_nn_module_models:
4839:                 guard_ctx = torch._dynamo.config.patch(guard_nn_modules=True)
4840: 
4841:             with guard_ctx:
4842:                 runner.run_one_model(
4843:                     name,
4844:                     model,
4845:                     example_inputs,
4846:                     optimize_ctx,
4847:                     experiment,
4848:                     explain=args.explain,
4849:                     tag=args.tag,
4850:                     batch_size=batch_size if args.dynamic_batch_only else None,
4851:                 )
4852:         if args.generate_aot_autograd_stats:
4853:             stats_file = output_filename.split(".csv")[0] + "_stats.csv"
4854:             write_outputs(
4855:                 stats_file,
4856:                 ("dev", "name", "batch_size", "total_aot_graphs", "ok_aot_graphs"),
4857:                 [
4858:                     current_device,
4859:                     current_name,
4860:                     current_batch_size,
4861:                     *Stats.aot_summary(),
4862:                 ],
4863:             )
4864:     else:
4865:         metrics.purge_old_log_files()
4866:         if (
4867:             output_filename
4868:             and os.path.exists(output_filename)
4869:             and not args.retain_output
4870:         ):
4871:             os.unlink(output_filename)
4872:         if original_dir:
4873:             os.chdir(original_dir)
4874:         model_names = list(runner.iter_model_names(args))
4875:         nmodels = len(model_names)
4876:         for i, name in enumerate(model_names):
4877:             current_name = name
4878:             if args.progress:
4879:                 print(f"Running model {i + 1}/{nmodels}", flush=True)
4880: 
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 4881-4948
````python
4881:             try:
4882:                 timeout = args.timeout
4883:                 if should_diff_branch(args):
4884:                     timeout *= 2
4885:                 env = os.environ.copy()
4886:                 if args.ci and name in CI_PRESERVE_COMPILE_DEBUG:
4887:                     env["TORCH_COMPILE_DEBUG"] = "1"
4888:                 subprocess.check_call(
4889:                     [sys.executable] + sys.argv + [f"--only={name}"],
4890:                     timeout=timeout,
4891:                     env=env,
4892:                 )
4893:             except subprocess.TimeoutExpired:
4894:                 write_csv_when_exception(args, name, "timeout")
4895:                 # NB: device is potentially multiple here, though we should
4896:                 # try our best to report in anyway TODO
4897:                 output_signpost(
4898:                     {"name": name}, args, runner.suite_name, error="timeout"
4899:                 )
4900:             except subprocess.CalledProcessError as e:
4901:                 print("Run failed with return code: ", e.returncode, file=sys.stderr)
4902:                 print("Output: ", e.output, file=sys.stderr)
4903:                 print("Error: ", e.stderr, file=sys.stderr)
4904:         print_summary(output_filename, print_dataframe=args.print_dataframe_summary)
4905: 
4906: 
4907: def log_operator_inputs(model, example_inputs, model_iter_fn, name, args):
4908:     mode = "training" if args.training else "eval"
4909:     output = os.path.join(os.path.dirname(args.output), f"{name}_{mode}.txt")
4910: 
4911:     # TODO - add option for coalescing inputs over multiple runs
4912:     if os.path.exists(output):
4913:         print(f"Skipping {name}, {output} already exists")
4914:         return
4915: 
4916:     print(f"Running {name}")
4917:     try:
4918:         from .microbenchmarks.operator_inp_utils import OperatorInputsMode
4919:     except ImportError:
4920:         from microbenchmarks.operator_inp_utils import OperatorInputsMode
4921: 
4922:     operator_mode = OperatorInputsMode()
4923:     fake_tensor_mode = FakeTensorMode()
4924: 
4925:     with torch._subclasses.fake_tensor.FakeCopyMode(fake_tensor_mode):
4926:         model_fake = copy.deepcopy(model)
4927:         example_inputs_fake = copy.deepcopy(example_inputs)
4928:     try:
4929:         with fake_tensor_mode, operator_mode:
4930:             model_iter_fn(model_fake, example_inputs_fake, collect_outputs=False)
4931:     except Exception as e:
4932:         print(f"{name} failed to run with fake tensors, trying real. Exception: {e}")
4933:         operator_mode = OperatorInputsMode()
4934:         try:
4935:             with operator_mode:
4936:                 model_iter_fn(model, example_inputs, collect_outputs=False)
4937:         except Exception as e2:
4938:             print(f"{name} failed to run with real. Exception: {e2}")
4939:             raise
4940: 
4941:     print(f"Writing output to {output}")
4942:     operator_mode.log_to_file(output)
4943: 
4944: 
4945: if __name__ == "__main__":
4946:     raise RuntimeError(
4947:         f"You shouldn't run {sys.argv[0]} directly, instead try timm_model.py, torchbench.py or huggingface.py"
4948:     )
````
- EN: Handles module imports such as `.microbenchmarks.operator_inp_utils`, `microbenchmarks.operator_inp_utils`.
- CN: 处理模块导入，例如 `.microbenchmarks.operator_inp_utils`, `microbenchmarks.operator_inp_utils`。
- EN: Implements callable logic such as `log_operator_inputs`.
- CN: 实现可调用逻辑，例如 `log_operator_inputs`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `CI` / 符号 `CI`
- Symbol `load_yaml_file` / 符号 `load_yaml_file`
- Symbol `flatten` / 符号 `flatten`
- Symbol `maybe_list_to_set` / 符号 `maybe_list_to_set`

## Dependencies / 依赖关系
- Python imports: `__future__`, `argparse`, `collections`, `contextlib`, `copy`, `csv`, `dataclasses`, `functools`, `gc`, `importlib`
- Python 导入: `__future__`, `argparse`, `collections`, `contextlib`, `copy`, `csv`, `dataclasses`, `functools`, `gc`, `importlib`
