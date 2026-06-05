# torchbench.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/dynamo/torchbench.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60
````python
 1: #!/usr/bin/env python3
 2: 
 3: import gc
 4: import importlib
 5: import logging
 6: import os
 7: import re
 8: import sys
 9: import tempfile
10: import warnings
11: from collections import namedtuple
12: from functools import cached_property
13: from os.path import abspath, exists
14: 
15: import torch
16: 
17: 
18: try:
19:     from .common import BenchmarkRunner, load_yaml_file, main
20: except ImportError:
21:     from common import BenchmarkRunner, load_yaml_file, main
22: 
23: from torch._dynamo.testing import collect_results, reduce_to_scalar_loss
24: from torch._dynamo.utils import clone_inputs
25: 
26: 
27: # We are primarily interested in tf32 datatype
28: torch.backends.cuda.matmul.allow_tf32 = True
29: 
30: # Enable FX graph caching
31: if "TORCHINDUCTOR_FX_GRAPH_CACHE" not in os.environ:
32:     torch._inductor.config.fx_graph_cache = True
33: 
34: # Enable Autograd caching
35: if "TORCHINDUCTOR_AUTOGRAD_CACHE" not in os.environ:
36:     torch._functorch.config.enable_autograd_cache = True
37: 
38: 
39: def _reassign_parameters(model):
40:     # torch_geometric models register parameter as tensors due to
41:     # https://github.com/pyg-team/pytorch_geometric/blob/master/torch_geometric/nn/dense/linear.py#L158-L168
42:     # Since it is unusual thing to do, we just reassign them to parameters
43:     def state_dict_hook(module, destination, prefix, local_metadata):
44:         for name, param in module.named_parameters():
45:             if isinstance(destination[name], torch.Tensor) and not isinstance(
46:                 destination[name], torch.nn.Parameter
47:             ):
48:                 destination[name] = torch.nn.Parameter(destination[name])
49: 
50:     model._register_state_dict_hook(state_dict_hook)
51: 
52: 
53: def setup_torchbench_cwd():
54:     original_dir = abspath(os.getcwd())
55: 
56:     os.environ["KALDI_ROOT"] = tempfile.gettempdir()  # avoids some spam
57:     for torchbench_dir in (
58:         "./torchbenchmark",
59:         "../torchbenchmark",
60:         "../torchbench",
````
- EN: Handles module imports such as `gc`, `importlib`, `logging`, `os`.
- CN: 处理模块导入，例如 `gc`, `importlib`, `logging`, `os`。
- EN: Implements callable logic such as `_reassign_parameters`, `state_dict_hook`, `setup_torchbench_cwd`.
- CN: 实现可调用逻辑，例如 `_reassign_parameters`, `state_dict_hook`, `setup_torchbench_cwd`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 61-120
````python
 61:         "../benchmark",
 62:         "../../torchbenchmark",
 63:         "../../torchbench",
 64:         "../../benchmark",
 65:         "../../../torchbenchmark",
 66:         "../../../torchbench",
 67:         "../../../benchmark",
 68:     ):
 69:         if exists(torchbench_dir):
 70:             break
 71: 
 72:     if exists(torchbench_dir):
 73:         torchbench_dir = abspath(torchbench_dir)
 74:         os.chdir(torchbench_dir)
 75:         sys.path.append(torchbench_dir)
 76: 
 77:     return original_dir
 78: 
 79: 
 80: process_train_model_output = {}
 81: 
 82: 
 83: class TorchBenchmarkRunner(BenchmarkRunner):
 84:     def __init__(self):
 85:         super().__init__()
 86:         self.suite_name = "torchbench"
 87:         self.optimizer = None
 88: 
 89:     @property
 90:     def _config(self):
 91:         return load_yaml_file("torchbench.yaml")
 92: 
 93:     @property
 94:     def _skip(self):
 95:         return self._config["skip"]
 96: 
 97:     @property
 98:     def _batch_size(self):
 99:         return self._config["batch_size"]
100: 
101:     @property
102:     def _tolerance(self):
103:         return self._config["tolerance"]
104: 
105:     @property
106:     def _require_larger_multiplier_for_smaller_tensor(self):
107:         return self._config["require_larger_multiplier_for_smaller_tensor"]
108: 
109:     @property
110:     def _accuracy(self):
111:         return self._config["accuracy"]
112: 
113:     @property
114:     def skip_models(self):
115:         return self._skip["all"]
116: 
117:     @property
118:     def skip_models_for_cpu(self):
119:         return self._skip["device"]["cpu"]
120: 
````
- EN: Declares or extends types including `TorchBenchmarkRunner`.
- CN: 声明或扩展类型，包括 `TorchBenchmarkRunner`。
- EN: Implements callable logic such as `__init__`, `_config`, `_skip`, `_batch_size`.
- CN: 实现可调用逻辑，例如 `__init__`, `_config`, `_skip`, `_batch_size`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 121-180
````python
121:     @property
122:     def skip_models_for_cpu_aarch64(self):
123:         return self._skip["device"]["cpu_aarch64"]
124: 
125:     @property
126:     def skip_models_for_cuda(self):
127:         return self._skip["device"]["cuda"]
128: 
129:     @property
130:     def skip_models_for_xpu(self):
131:         return self._skip["device"]["xpu"]
132: 
133:     @property
134:     def skip_models_for_freezing_cuda(self):
135:         return self._skip["freezing"]["cuda"]
136: 
137:     @property
138:     def disable_cudagraph_models(self):
139:         return self._config["disable_cudagraph"]
140: 
141:     @property
142:     def skip_models_for_freezing_cpu(self):
143:         return self._skip["freezing"]["cpu"]
144: 
145:     @property
146:     def slow_models(self):
147:         return self._config["slow"]
148: 
149:     @property
150:     def very_slow_models(self):
151:         return self._config["very_slow"]
152: 
153:     @property
154:     def non_deterministic_models(self):
155:         return self._config["non_deterministic"]
156: 
157:     @property
158:     def get_output_amp_train_process_func(self):
159:         return process_train_model_output
160: 
161:     @property
162:     def skip_not_suitable_for_training_models(self):
163:         return self._skip["test"]["training"]
164: 
165:     @property
166:     def failing_fx2trt_models(self):
167:         return self._config["trt_not_yet_working"]
168: 
169:     @property
170:     def force_amp_for_fp16_bf16_models(self):
171:         return self._config["dtype"]["force_amp_for_fp16_bf16_models"]
172: 
173:     @property
174:     def force_fp16_for_bf16_models(self):
175:         return self._config["dtype"]["force_fp16_for_bf16_models"]
176: 
177:     @property
178:     def amp_dtype_bfloat16(self):
179:         return self._config["dtype"]["amp_dtype_bfloat16"]
180: 
````
- EN: Implements callable logic such as `skip_models_for_cpu_aarch64`, `skip_models_for_cuda`, `skip_models_for_xpu`, `skip_models_for_freezing_cuda`.
- CN: 实现可调用逻辑，例如 `skip_models_for_cpu_aarch64`, `skip_models_for_cuda`, `skip_models_for_xpu`, `skip_models_for_freezing_cuda`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 181-240
````python
181:     @property
182:     def skip_accuracy_checks_large_models_dashboard(self):
183:         if self.args.dashboard or self.args.accuracy:
184:             return self._accuracy["skip"]["large_models"]
185:         return set()
186: 
187:     @property
188:     def skip_accuracy_check_as_eager_non_deterministic(self):
189:         if self.args.accuracy and self.args.training:
190:             return self._accuracy["skip"]["eager_not_deterministic"]
191:         return set()
192: 
193:     @property
194:     def skip_multiprocess_models(self):
195:         return self._skip["multiprocess"]
196: 
197:     @property
198:     def skip_models_due_to_control_flow(self):
199:         return self._skip["control_flow"]
200: 
201:     @property
202:     def skip_models_due_to_export_not_supported(self):
203:         return self._skip["export_not_supported"]
204: 
205:     @property
206:     def guard_on_nn_module_models(self):
207:         return {
208:             "vision_maskrcnn",
209:         }
210: 
211:     @cached_property
212:     def _fb_models_available(self):
213:         """This property exists because importing IS_FBCODE causes some models to be
214:         frozen out of setting certain config flags."""
215:         return importlib.util.find_spec("torchbenchmark.models.fb") is not None
216: 
217:     def load_model(
218:         self,
219:         device,
220:         model_name,
221:         batch_size=None,
222:         part=None,
223:         extra_args=None,
224:     ):
225:         if self.args.enable_activation_checkpointing:
226:             raise NotImplementedError(
227:                 "Activation checkpointing not implemented for Torchbench models"
228:             )
229:         is_training = self.args.training
230:         use_eval_mode = self.args.use_eval_mode
231: 
232:         candidates = [
233:             f"torchbenchmark.models.{model_name}",
234:             f"torchbenchmark.canary_models.{model_name}",
235:         ]
236:         if self._fb_models_available:
237:             candidates.append(f"torchbenchmark.models.fb.{model_name}")
238: 
239:         for c in candidates:
240:             try:
````
- EN: Implements callable logic such as `skip_accuracy_checks_large_models_dashboard`, `skip_accuracy_check_as_eager_non_deterministic`, `skip_multiprocess_models`, `skip_models_due_to_control_flow`.
- CN: 实现可调用逻辑，例如 `skip_accuracy_checks_large_models_dashboard`, `skip_accuracy_check_as_eager_non_deterministic`, `skip_multiprocess_models`, `skip_models_due_to_control_flow`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-300
````python
241:                 module = importlib.import_module(c)
242:                 break
243:             except ModuleNotFoundError as e:
244:                 if e.name != c:
245:                     raise
246:         else:
247:             raise ImportError(f"could not import any of {candidates}")
248:         benchmark_cls = getattr(module, "Model", None)
249:         if benchmark_cls is None:
250:             raise NotImplementedError(f"{model_name}.Model is None")
251: 
252:         if not hasattr(benchmark_cls, "name"):
253:             benchmark_cls.name = model_name
254: 
255:         cant_change_batch_size = (
256:             not getattr(benchmark_cls, "ALLOW_CUSTOMIZE_BSIZE", True)
257:             or model_name in self._config["dont_change_batch_size"]
258:         )
259:         if cant_change_batch_size:
260:             batch_size = None
261:         if (
262:             batch_size is None
263:             and is_training
264:             and model_name in self._batch_size["training"]
265:         ):
266:             batch_size = self._batch_size["training"][model_name]
267:         elif (
268:             batch_size is None
269:             and not is_training
270:             and model_name in self._batch_size["inference"]
271:         ):
272:             batch_size = self._batch_size["inference"][model_name]
273: 
274:         # Control the memory footprint for few models
275:         if self.args.accuracy and model_name in self._accuracy["max_batch_size"]:
276:             batch_size = min(batch_size, self._accuracy["max_batch_size"][model_name])
277: 
278:         # workaround "RuntimeError: not allowed to set torch.backends.cudnn flags"
279:         torch.backends.__allow_nonbracketed_mutation_flag = True
280:         if extra_args is None:
281:             extra_args = []
282:         if part:
283:             extra_args += ["--part", part]
284: 
285:         # sam_fast only runs with amp
286:         if model_name == "sam_fast":
287:             self.args.amp = True
288:             self.setup_amp()
289: 
290:         if model_name == "vision_maskrcnn" and is_training:
291:             # Output of vision_maskrcnn model is a list of bounding boxes,
292:             # sorted on the basis of their scores. This makes accuracy
293:             # comparison hard with torch.compile. torch.compile can cause minor
294:             # divergences in the output because of how fusion works for amp in
295:             # TorchInductor compared to eager.  Therefore, instead of looking at
296:             # all the bounding boxes, we compare only top 4.
297:             model_kwargs = {"box_detections_per_img": 4}
298:             benchmark = benchmark_cls(
299:                 test="train",
300:                 device=device,
````
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 301-360
````python
301:                 batch_size=batch_size,
302:                 extra_args=extra_args,
303:                 model_kwargs=model_kwargs,
304:             )
305:             use_eval_mode = True
306:         elif is_training:
307:             benchmark = benchmark_cls(
308:                 test="train",
309:                 device=device,
310:                 batch_size=batch_size,
311:                 extra_args=extra_args,
312:             )
313:         else:
314:             benchmark = benchmark_cls(
315:                 test="eval",
316:                 device=device,
317:                 batch_size=batch_size,
318:                 extra_args=extra_args,
319:             )
320:         model, example_inputs = benchmark.get_module()
321:         if model_name in [
322:             "basic_gnn_edgecnn",
323:             "basic_gnn_gcn",
324:             "basic_gnn_sage",
325:             "basic_gnn_gin",
326:         ]:
327:             _reassign_parameters(model)
328: 
329:         # Models that must be in train mode while training
330:         if is_training and (
331:             not use_eval_mode or model_name in self._config["only_training"]
332:         ):
333:             model.train()
334:         else:
335:             model.eval()
336:         gc.collect()
337:         batch_size = benchmark.batch_size
338:         if model_name == "torchrec_dlrm":
339:             batch_namedtuple = namedtuple(
340:                 "Batch", "dense_features sparse_features labels"
341:             )
342:             example_inputs = tuple(
343:                 batch_namedtuple(
344:                     dense_features=batch.dense_features,
345:                     sparse_features=batch.sparse_features,
346:                     labels=batch.labels,
347:                 )
348:                 for batch in example_inputs
349:             )
350:         # Torchbench has quite different setup for yolov3, so directly passing
351:         # the right example_inputs
352:         if model_name == "yolov3":
353:             example_inputs = (torch.rand(batch_size, 3, 384, 512).to(device),)
354:         # See https://github.com/pytorch/benchmark/issues/1561
355:         if model_name == "maml_omniglot":
356:             batch_size = 5
357:             if example_inputs[0].shape[0] != batch_size:
358:                 raise AssertionError(
359:                     f"Expected batch size {batch_size}, but got {example_inputs[0].shape[0]}"
360:                 )
````
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 361-420
````python
361:         if model_name == "vision_maskrcnn":
362:             batch_size = 1
363:         # global current_name, current_device
364:         # current_device = device
365:         # current_name = benchmark.name
366: 
367:         if self.args.trace_on_xla:
368:             # work around for: https://github.com/pytorch/xla/issues/4174
369:             import torch_xla  # noqa: F401
370: 
371:         # Turning off kv cache for torchbench models. This is not the right
372:         # thing to do, but the torchbench models are way outdated, and since we
373:         # are using torchbench pt2 dashboard to track regressions (rather than
374:         # improving performance), we are just setting the kv cache to false.
375:         # Real transformers benchmarks will be added soon using a different
376:         # infra.
377:         if (
378:             model_name.startswith("hf")
379:             and hasattr(model, "config")
380:             and hasattr(model.config, "use_cache")
381:         ):
382:             model.config.use_cache = False
383: 
384:         self.validate_model(benchmark.name, model, example_inputs)
385:         return device, benchmark.name, model, example_inputs, batch_size
386: 
387:     def iter_model_names(self, args):
388:         from torchbenchmark import _list_canary_model_paths, _list_model_paths
389: 
390:         models = _list_model_paths()
391:         models += [
392:             f
393:             for f in _list_canary_model_paths()
394:             if os.path.basename(f) in self._config["canary_models"]
395:         ]
396:         models.sort()
397: 
398:         start, end = self.get_benchmark_indices(len(models))
399:         for index, model_path in enumerate(models):
400:             if index < start or index >= end:
401:                 continue
402: 
403:             model_name = os.path.basename(model_path)
404:             if (
405:                 not re.search("|".join(args.filter), model_name, re.IGNORECASE)
406:                 or re.search("|".join(args.exclude), model_name, re.IGNORECASE)
407:                 or model_name in args.exclude_exact
408:                 or model_name in self.skip_models
409:             ):
410:                 continue
411: 
412:             yield model_name
413: 
414:     def pick_grad(self, name, is_training):
415:         if is_training or name == "maml":
416:             return torch.enable_grad()
417:         else:
418:             return torch.no_grad()
419: 
420:     def use_larger_multiplier_for_smaller_tensor(self, name):
````
- EN: Handles module imports such as `torch_xla`, `torchbenchmark`.
- CN: 处理模块导入，例如 `torch_xla`, `torchbenchmark`。
- EN: Implements callable logic such as `iter_model_names`, `pick_grad`, `use_larger_multiplier_for_smaller_tensor`.
- CN: 实现可调用逻辑，例如 `iter_model_names`, `pick_grad`, `use_larger_multiplier_for_smaller_tensor`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 421-480
````python
421:         return name in self._require_larger_multiplier_for_smaller_tensor
422: 
423:     def use_iou_for_bool_accuracy(self, name):
424:         iou_models = self._tolerance.get("use_iou_for_bool_masks", [])
425:         return name in iou_models
426: 
427:     def get_iou_threshold(self, name):
428:         iou_thresholds = self._tolerance.get("iou_thresholds", {})
429:         return iou_thresholds.get(name, 0.99)
430: 
431:     def get_accuracy_check_runs(self, name):
432:         accuracy_check_runs = self._tolerance.get("accuracy_check_runs", {})
433:         return accuracy_check_runs.get(name, 1)
434: 
435:     def get_tolerance_and_cosine_flag(self, is_training, current_device, name):
436:         tolerance = 1e-4
437:         cosine = self.args.cosine
438:         # Increase the tolerance for torch allclose
439:         if self.args.float16 or self.args.amp:
440:             if self.args.freezing and (freezing := self._tolerance["freezing"]):
441:                 higher_fp16 = freezing.get("higher_fp16", None)
442:                 even_higher = freezing.get("even_higher", None)
443:                 if higher_fp16 and name in higher_fp16:
444:                     return 1e-2, cosine
445:                 elif even_higher and name in even_higher:
446:                     return 8 * 1e-2, cosine
447:             if name in self._tolerance["higher_fp16"]:
448:                 return 1e-2, cosine
449:             elif name in self._tolerance["even_higher"]:
450:                 return 8 * 1e-2, cosine
451:             return 1e-3, cosine
452: 
453:         if self.args.bfloat16:
454:             if name in self._tolerance["higher_bf16"]:
455:                 return 1e-2, cosine
456:             elif current_device == "xpu" and name in self._tolerance["higher_bf16_xpu"]:
457:                 return 8 * 1e-2, cosine
458: 
459:         if is_training and (current_device == "cuda" or current_device == "xpu"):
460:             tolerance = 1e-3
461:             if name in self._tolerance["cosine"]:
462:                 cosine = True
463:             elif name in self._tolerance["higher"]:
464:                 tolerance = 1e-3
465:             elif name in self._tolerance["even_higher"]:
466:                 tolerance = 8 * 1e-2
467:         return tolerance, cosine
468: 
469:     def compute_loss(self, pred):
470:         return reduce_to_scalar_loss(pred)
471: 
472:     def forward_pass(self, mod, inputs, collect_outputs=True):
473:         with self.autocast(**self.autocast_arg):
474:             if isinstance(inputs, dict):
475:                 return mod(**inputs)
476:             else:
477:                 return mod(*inputs)
478: 
479:     def forward_and_backward_pass(self, mod, inputs, collect_outputs=True):
480:         cloned_inputs = clone_inputs(inputs)
````
- EN: Implements callable logic such as `use_iou_for_bool_accuracy`, `get_iou_threshold`, `get_accuracy_check_runs`, `get_tolerance_and_cosine_flag`.
- CN: 实现可调用逻辑，例如 `use_iou_for_bool_accuracy`, `get_iou_threshold`, `get_accuracy_check_runs`, `get_tolerance_and_cosine_flag`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 481-503
````python
481:         self.optimizer_zero_grad(mod)
482:         with self.autocast(**self.autocast_arg):
483:             if isinstance(cloned_inputs, dict):
484:                 pred = mod(**cloned_inputs)
485:             else:
486:                 pred = mod(*cloned_inputs)
487:             loss = self.compute_loss(pred)
488:         self.grad_scaler.scale(loss).backward()
489:         self.optimizer_step()
490:         if collect_outputs:
491:             return collect_results(mod, None, loss, cloned_inputs)
492:         return None
493: 
494: 
495: def torchbench_main():
496:     original_dir = setup_torchbench_cwd()
497:     logging.basicConfig(level=logging.WARNING)
498:     warnings.filterwarnings("ignore")
499:     main(TorchBenchmarkRunner(), original_dir)
500: 
501: 
502: if __name__ == "__main__":
503:     torchbench_main()
````
- EN: Implements callable logic such as `torchbench_main`.
- CN: 实现可调用逻辑，例如 `torchbench_main`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `_reassign_parameters` / 符号 `_reassign_parameters`
- Symbol `state_dict_hook` / 符号 `state_dict_hook`
- Symbol `setup_torchbench_cwd` / 符号 `setup_torchbench_cwd`
- Symbol `TorchBenchmarkRunner` / 符号 `TorchBenchmarkRunner`

## Dependencies / 依赖关系
- Python imports: `gc`, `importlib`, `logging`, `os`, `re`, `sys`, `tempfile`, `warnings`, `collections`, `functools`
- Python 导入: `gc`, `importlib`, `logging`, `os`, `re`, `sys`, `tempfile`, `warnings`, `collections`, `functools`
