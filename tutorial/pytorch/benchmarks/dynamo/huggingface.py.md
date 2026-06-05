# huggingface.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/dynamo/huggingface.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60
````python
 1: #!/usr/bin/env python3
 2: # flake8: noqa: F821
 3: 
 4: import importlib
 5: import logging
 6: import os
 7: import re
 8: import subprocess
 9: import sys
10: import types
11: import warnings
12: 
13: 
14: try:
15:     from .common import (
16:         BenchmarkRunner,
17:         download_retry_decorator,
18:         load_yaml_file,
19:         main,
20:         reset_rng_state,
21:     )
22: except ImportError:
23:     from common import (
24:         BenchmarkRunner,
25:         download_retry_decorator,
26:         load_yaml_file,
27:         main,
28:         reset_rng_state,
29:     )
30: 
31: import torch
32: from torch._dynamo.testing import collect_results
33: from torch._dynamo.utils import clone_inputs
34: 
35: 
36: log = logging.getLogger(__name__)
37: 
38: # Enable FX graph caching
39: if "TORCHINDUCTOR_FX_GRAPH_CACHE" not in os.environ:
40:     torch._inductor.config.fx_graph_cache = True
41: 
42: # Enable Autograd caching
43: if "TORCHINDUCTOR_AUTOGRAD_CACHE" not in os.environ:
44:     torch._functorch.config.enable_autograd_cache = True
45: 
46: 
47: def pip_install(package):
48:     subprocess.check_call([sys.executable, "-m", "pip", "install", package])
49: 
50: 
51: # Disable the flake warnings for the imports. Flake8 does not provide a way to
52: # disable just warning for the entire file. Disabling flake8 entirely.
53: imports = [
54:     "AlbertForPreTraining",
55:     "AutoConfig",
56:     "AutoModelForCausalLM",
57:     "AutoModelForMaskedLM",
58:     "AutoModelForSeq2SeqLM",
59:     "BigBirdConfig",
60:     "BlenderbotForConditionalGeneration",
````
- EN: Handles module imports such as `importlib`, `logging`, `os`, `re`.
- CN: 处理模块导入，例如 `importlib`, `logging`, `os`, `re`。
- EN: Implements callable logic such as `pip_install`.
- CN: 实现可调用逻辑，例如 `pip_install`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 61-120
````python
 61:     "BlenderbotModel",
 62:     "BlenderbotSmallModel",
 63:     "CLIPModel",
 64:     "CLIPVisionModel",
 65:     "ElectraForPreTraining",
 66:     "GPT2ForSequenceClassification",
 67:     "GPTJForSequenceClassification",
 68:     "GPTNeoForSequenceClassification",
 69:     "HubertForSequenceClassification",
 70:     "LxmertForPreTraining",
 71:     "LxmertForQuestionAnswering",
 72:     "MarianForCausalLM",
 73:     "MarianModel",
 74:     "MarianMTModel",
 75:     "PegasusModel",
 76:     "ReformerConfig",
 77:     "ViTForImageClassification",
 78:     "ViTForMaskedImageModeling",
 79:     "ViTModel",
 80: ]
 81: 
 82: 
 83: def process_hf_reformer_output(out):
 84:     if not isinstance(out, list):
 85:         raise AssertionError(f"expected out to be a list, got {type(out)}")
 86:     # second output is unstable
 87:     return [elem for i, elem in enumerate(out) if i != 1]
 88: 
 89: 
 90: try:
 91:     mod = importlib.import_module("transformers")
 92:     for cls in imports:
 93:         if not hasattr(mod, cls):
 94:             raise ModuleNotFoundError
 95: except ModuleNotFoundError:
 96:     print("Installing HuggingFace Transformers...")
 97:     pip_install("git+https://github.com/huggingface/transformers.git#egg=transformers")
 98: finally:
 99:     for cls in imports:
100:         exec(f"from transformers import {cls}")
101: 
102: 
103: # These models contain the models present in huggingface_models_list. It is a
104: # combination of models supported by HF Fx parser and some manually supplied
105: # models. For these models, we already know the largest batch size that can fit
106: # on A100 GPUs - 40 GB.
107: BATCH_SIZE_KNOWN_MODELS = {}
108: 
109: # Run only this selected group of models, leave this empty to run everything
110: TORCHBENCH_ONLY_MODELS = [
111:     m.strip() for m in os.getenv("TORCHBENCH_ONLY_MODELS", "").split(",") if m.strip()
112: ]
113: 
114: 
115: # TODO(sdym): use batch-size-file parameter of common.main, like torchbench.py
116: # Get the list of models and their batch sizes
117: MODELS_FILENAME = os.path.join(os.path.dirname(__file__), "huggingface_models_list.txt")
118: if not os.path.exists(MODELS_FILENAME):
119:     raise AssertionError(f"models file not found: {MODELS_FILENAME}")
120: with open(MODELS_FILENAME) as fh:
````
- EN: Implements callable logic such as `process_hf_reformer_output`.
- CN: 实现可调用逻辑，例如 `process_hf_reformer_output`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-180
````python
121:     lines = fh.readlines()
122:     lines = [line.rstrip() for line in lines]
123:     for line in lines:
124:         model_name, batch_size = line.split(",")
125:         if TORCHBENCH_ONLY_MODELS and model_name not in TORCHBENCH_ONLY_MODELS:
126:             continue
127:         batch_size = int(batch_size)
128:         BATCH_SIZE_KNOWN_MODELS[model_name] = batch_size
129: if not BATCH_SIZE_KNOWN_MODELS:
130:     raise AssertionError("BATCH_SIZE_KNOWN_MODELS is empty")
131: 
132: 
133: try:
134:     from .huggingface_llm_models import HF_LLM_MODELS
135: except ImportError:
136:     from huggingface_llm_models import HF_LLM_MODELS
137: 
138: 
139: def get_module_cls_by_model_name(model_cls_name):
140:     _module_by_model_name = {
141:         "Speech2Text2Decoder": "transformers.models.speech_to_text_2.modeling_speech_to_text_2",
142:         "TrOCRDecoder": "transformers.models.trocr.modeling_trocr",
143:     }
144:     module_name = _module_by_model_name.get(model_cls_name, "transformers")
145:     module = importlib.import_module(module_name)
146:     return getattr(module, model_cls_name)
147: 
148: 
149: def get_sequence_length(model_cls, model_name):
150:     if model_name.startswith(("Blenderbot",)):
151:         seq_length = 128
152:     elif model_name.startswith(("GPT2", "Bart", "T5", "PLBart", "MBart")):
153:         seq_length = 1024
154:     elif model_name in ("AllenaiLongformerBase", "BigBird"):
155:         seq_length = 1024
156:     elif model_name.startswith("OPT"):
157:         seq_length = 2048
158:     elif "Reformer" in model_name:
159:         seq_length = 4096
160:     elif model_name.startswith(
161:         (
162:             "Albert",
163:             "Deberta",
164:             "Layout",
165:             "Electra",
166:             "XLNet",
167:             "MegatronBert",
168:             "Bert",
169:             "Roberta",
170:         )
171:     ) or model_name in ("DistillGPT2", "GoogleFnet", "YituTechConvBert"):
172:         seq_length = 512
173:     elif model_name in ("TrOCRForCausalLM"):
174:         seq_length = 256
175:     elif model_name.startswith("MobileBert"):
176:         seq_length = 128
177:     elif model_name.startswith("Wav2Vec2"):
178:         # If too short, will fail with something like
179:         # ValueError: `mask_length` has to be smaller than `sequence_length`,
180:         # but got `mask_length`: 10 and `sequence_length`: 9`
````
- EN: Handles module imports such as `.huggingface_llm_models`, `huggingface_llm_models`.
- CN: 处理模块导入，例如 `.huggingface_llm_models`, `huggingface_llm_models`。
- EN: Implements callable logic such as `get_module_cls_by_model_name`, `get_sequence_length`.
- CN: 实现可调用逻辑，例如 `get_module_cls_by_model_name`, `get_sequence_length`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 181-240
````python
181:         seq_length = 10000  # NB: a more realistic size is 155136
182:     else:
183:         log.info(
184:             f"Sequence Length not defined for {model_name}. Choosing 128 arbitrarily"  # noqa: G004
185:         )
186:         seq_length = 128
187:     return seq_length
188: 
189: 
190: def generate_inputs_for_model(
191:     model_cls, model, model_name, bs, device, include_loss_args=False
192: ):
193:     # TODO - Check if following values are representative
194:     num_choices = 3
195:     num_visual_features = 42
196:     seq_length = get_sequence_length(model_cls, model_name)
197:     vocab_size = model.config.vocab_size
198: 
199:     if model_name.startswith("Wav2Vec2"):
200:         # TODO: If we add more input_values style models, try to work this
201:         # into the overall control flow
202:         target_length = 100
203:         return {
204:             "input_values": torch.randn((bs, seq_length), device=device),
205:             # Added because that's what the example training script has
206:             "attention_mask": rand_int_tensor(device, 0, 2, (bs, seq_length)),
207:             "labels": rand_int_tensor(device, 0, vocab_size, (bs, target_length)),
208:         }
209: 
210:     if model_name.endswith("MultipleChoice"):
211:         input = rand_int_tensor(device, 0, vocab_size, (bs, num_choices, seq_length))
212:     elif model_name.startswith("Roberta"):
213:         input = rand_int_tensor(device, 0, 1, (bs, seq_length))
214:     else:
215:         input = rand_int_tensor(device, 0, vocab_size, (bs, seq_length))
216: 
217:     if "Bart" in model_name:
218:         input[:, -1] = model.config.eos_token_id
219: 
220:     input_dict = {"input_ids": input}
221: 
222:     if model_name.startswith(("T5", "M2M100", "MT5")) or model_cls in [
223:         BlenderbotModel,
224:         BlenderbotSmallModel,
225:         BlenderbotForConditionalGeneration,
226:         PegasusModel,
227:         MarianModel,
228:         MarianMTModel,
229:     ]:
230:         input_dict["decoder_input_ids"] = input
231: 
232:     if model_name.startswith("Lxmert"):
233:         visual_feat_dim, visual_pos_dim = (
234:             model.config.visual_feat_dim,
235:             model.config.visual_pos_dim,
236:         )
237:         input_dict["visual_feats"] = torch.randn(
238:             bs, num_visual_features, visual_feat_dim
239:         )
240:         input_dict["visual_pos"] = torch.randn(bs, num_visual_features, visual_pos_dim)
````
- EN: Implements callable logic such as `generate_inputs_for_model`.
- CN: 实现可调用逻辑，例如 `generate_inputs_for_model`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-300
````python
241: 
242:     if include_loss_args:
243:         if model_name.endswith("PreTraining"):
244:             if model_cls in [ElectraForPreTraining, LxmertForPreTraining]:
245:                 input_dict["labels"] = rand_int_tensor(device, 0, 1, (bs, seq_length))
246:             else:
247:                 label_name = (
248:                     "sentence_order_label"
249:                     if model_cls in [AlbertForPreTraining]
250:                     else "next_sentence_label"
251:                 )
252:                 input_dict["labels"] = (
253:                     rand_int_tensor(device, 0, vocab_size, (bs, seq_length)),
254:                 )
255:                 input_dict[label_name] = rand_int_tensor(device, 0, 1, (bs,))
256:         elif model_name.endswith("QuestionAnswering"):
257:             input_dict["start_positions"] = rand_int_tensor(
258:                 device, 0, seq_length, (bs,)
259:             )
260:             input_dict["end_positions"] = rand_int_tensor(device, 0, seq_length, (bs,))
261:         elif model_name.endswith(
262:             ("MaskedLM", "HeadModel", "CausalLM", "DoubleHeadsModel")
263:         ):
264:             input_dict["labels"] = rand_int_tensor(
265:                 device, 0, vocab_size, (bs, seq_length)
266:             )
267:         elif model_name.endswith("TokenClassification"):
268:             input_dict["labels"] = rand_int_tensor(
269:                 device, 0, model.config.num_labels - 1, (bs, seq_length)
270:             )
271:         elif model_name.endswith("MultipleChoice"):
272:             input_dict["labels"] = rand_int_tensor(device, 0, num_choices, (bs,))
273:         elif model_name.endswith("SequenceClassification"):
274:             input_dict["labels"] = rand_int_tensor(
275:                 device, 0, model.config.num_labels - 1, (bs,)
276:             )
277:         elif model_name.endswith("NextSentencePrediction"):
278:             input_dict["labels"] = rand_int_tensor(device, 0, 1, (bs,))
279:         elif model_name.endswith("ForConditionalGeneration"):
280:             input_dict["labels"] = rand_int_tensor(
281:                 device, 0, vocab_size - 1, (bs, seq_length)
282:             )
283:         elif model_name in EXTRA_MODELS:
284:             input_dict["labels"] = rand_int_tensor(
285:                 device, 0, vocab_size, (bs, seq_length)
286:             )
287:         else:
288:             raise NotImplementedError(
289:                 f"Class {model_name} unsupported for training test "
290:             )
291: 
292:     return input_dict
293: 
294: 
295: def rand_int_tensor(device, low, high, shape):
296:     return torch.randint(
297:         low,
298:         high,
299:         shape,
300:         device=device,
````
- EN: Implements callable logic such as `rand_int_tensor`.
- CN: 实现可调用逻辑，例如 `rand_int_tensor`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 301-360
````python
301:         dtype=torch.int64,
302:         requires_grad=False,
303:     )
304: 
305: 
306: EXTRA_MODELS = {
307:     "AllenaiLongformerBase": (
308:         AutoConfig.from_pretrained("allenai/longformer-base-4096"),
309:         AutoModelForMaskedLM,
310:     ),
311:     "Reformer": (
312:         ReformerConfig(),
313:         AutoModelForMaskedLM,
314:     ),
315:     "T5Small": (
316:         AutoConfig.from_pretrained("t5-small"),
317:         AutoModelForSeq2SeqLM,
318:     ),
319:     # "BigBird": (
320:     #     BigBirdConfig(attention_type="block_sparse"),
321:     #     AutoModelForMaskedLM,
322:     # ),
323:     "DistillGPT2": (
324:         AutoConfig.from_pretrained("distilgpt2"),
325:         AutoModelForCausalLM,
326:     ),
327:     "GoogleFnet": (
328:         AutoConfig.from_pretrained("google/fnet-base"),
329:         AutoModelForMaskedLM,
330:     ),
331:     "YituTechConvBert": (
332:         AutoConfig.from_pretrained("YituTech/conv-bert-base"),
333:         AutoModelForMaskedLM,
334:     ),
335: }
336: 
337: 
338: class HuggingfaceRunner(BenchmarkRunner):
339:     def __init__(self):
340:         super().__init__()
341:         self.suite_name = "huggingface"
342: 
343:     @property
344:     def _config(self):
345:         return load_yaml_file("huggingface.yaml")
346: 
347:     @property
348:     def _skip(self):
349:         return self._config["skip"]
350: 
351:     @property
352:     def _accuracy(self):
353:         return self._config["accuracy"]
354: 
355:     @property
356:     def skip_models(self):
357:         return self._skip["all"]
358: 
359:     @property
360:     def skip_models_for_cpu(self):
````
- EN: Declares or extends types including `HuggingfaceRunner`.
- CN: 声明或扩展类型，包括 `HuggingfaceRunner`。
- EN: Implements callable logic such as `__init__`, `_config`, `_skip`, `_accuracy`.
- CN: 实现可调用逻辑，例如 `__init__`, `_config`, `_skip`, `_accuracy`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 361-420
````python
361:         return self._skip["device"]["cpu"]
362: 
363:     @property
364:     def fp32_only_models(self):
365:         return self._config["only_fp32"]
366: 
367:     @property
368:     def skip_models_due_to_control_flow(self):
369:         return self._skip["control_flow"]
370: 
371:     def use_larger_multiplier_for_smaller_tensor(self, name):
372:         return name in [
373:             "GPT2ForSequenceClassification",
374:         ]
375: 
376:     def _get_model_cls_and_config(self, model_name):
377:         if model_name not in EXTRA_MODELS:
378:             model_cls = get_module_cls_by_model_name(model_name)
379:             config_cls = model_cls.config_class
380:             config = config_cls()
381: 
382:             # NB: some models need a pad token defined to handle BS > 1
383:             if (
384:                 model_cls
385:                 in [
386:                     GPT2ForSequenceClassification,
387:                     GPTNeoForSequenceClassification,
388:                     GPTJForSequenceClassification,
389:                 ]
390:                 or model_cls.__name__.startswith("Roberta")
391:                 or model_cls.__name__.startswith("Marian")
392:             ):
393:                 config.pad_token_id = 0
394: 
395:         else:
396:             config, model_cls = EXTRA_MODELS[model_name]
397: 
398:         return model_cls, config
399: 
400:     @download_retry_decorator
401:     def _download_model(self, model_name):
402:         model_cls, config = self._get_model_cls_and_config(model_name)
403:         if "auto" in model_cls.__module__:
404:             # Handle auto classes
405:             model = model_cls.from_config(config)
406:         else:
407:             model = model_cls(config)
408:         return model
409: 
410:     def load_model(
411:         self,
412:         device,
413:         model_name,
414:         batch_size=None,
415:         extra_args=None,
416:     ):
417:         is_training = self.args.training
418:         use_eval_mode = self.args.use_eval_mode
419:         dtype = torch.float32
420:         reset_rng_state()
````
- EN: Implements callable logic such as `fp32_only_models`, `skip_models_due_to_control_flow`, `use_larger_multiplier_for_smaller_tensor`, `_get_model_cls_and_config`.
- CN: 实现可调用逻辑，例如 `fp32_only_models`, `skip_models_due_to_control_flow`, `use_larger_multiplier_for_smaller_tensor`, `_get_model_cls_and_config`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 421-480
````python
421: 
422:         # Get batch size
423:         if model_name in BATCH_SIZE_KNOWN_MODELS:
424:             batch_size_default = BATCH_SIZE_KNOWN_MODELS[model_name]
425:         elif batch_size is None:
426:             batch_size_default = 16
427:             log.info(
428:                 f"Batch size not specified for {model_name}. Setting batch_size=16"  # noqa: G004
429:             )
430: 
431:         if batch_size is None:
432:             batch_size = batch_size_default
433:             batch_size_divisors = self._config["batch_size"]["divisors"]
434:             if model_name in batch_size_divisors:
435:                 batch_size = max(int(batch_size / batch_size_divisors[model_name]), 1)
436:                 log.info(
437:                     f"Running smaller batch size={batch_size} for {model_name}, orig batch_size={batch_size_default}"  # noqa: G004
438:                 )
439: 
440:         # Get model and example inputs
441:         if model_name in HF_LLM_MODELS:
442:             benchmark_cls = HF_LLM_MODELS[model_name]
443:             model, example_inputs = benchmark_cls.get_model_and_inputs(
444:                 model_name, device
445:             )
446: 
447:             # Set this flag so that when we test for speedup, we use
448:             # model.generate instead of using model.forward
449:             self.hf_llm = True
450: 
451:             def generate(self, _, example_inputs, collect_outputs=True):
452:                 return model.generate(**example_inputs)
453: 
454:             self.generate = types.MethodType(generate, self)
455: 
456:         else:
457:             self.hf_llm = False
458: 
459:             model_cls, config = self._get_model_cls_and_config(model_name)
460:             model = self._download_model(model_name)
461:             model = model.to(device, dtype=dtype)
462: 
463:             example_inputs = generate_inputs_for_model(
464:                 model_cls, model, model_name, batch_size, device, include_loss_args=True
465:             )
466: 
467:             # So we can check for correct gradients without eliminating the dropout computation
468:             for attr in dir(config):
469:                 if "drop" in attr and isinstance(getattr(config, attr), float):
470:                     setattr(config, attr, 1e-30)
471: 
472:             # Turning off kv cache for torchbench models. This is not the right
473:             # thing to do, but the pt2 dashboard is outdated. Real transformers
474:             # benchmarks will be added soon using a different infra.
475:             if hasattr(model, "config") and hasattr(model.config, "use_cache"):
476:                 model.config.use_cache = False
477: 
478:         if self.args.enable_activation_checkpointing:
479:             model.gradient_checkpointing_enable()
480: 
````
- EN: Implements callable logic such as `generate`.
- CN: 实现可调用逻辑，例如 `generate`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 481-540
````python
481:         if (
482:             is_training
483:             and not use_eval_mode
484:             and not (
485:                 self.args.accuracy and model_name in self._config["only_inference"]
486:             )
487:         ):
488:             model.train()
489:         else:
490:             model.eval()
491: 
492:         self.validate_model(model_name, model, example_inputs)
493:         return device, model_name, model, example_inputs, batch_size
494: 
495:     def iter_model_names(self, args):
496:         model_names = list(BATCH_SIZE_KNOWN_MODELS.keys()) + list(EXTRA_MODELS.keys())
497:         model_names = set(model_names)
498:         model_names = sorted(model_names)
499: 
500:         start, end = self.get_benchmark_indices(len(model_names))
501:         for index, model_name in enumerate(model_names):
502:             if index < start or index >= end:
503:                 continue
504:             if (
505:                 not re.search("|".join(args.filter), model_name, re.IGNORECASE)
506:                 or re.search("|".join(args.exclude), model_name, re.IGNORECASE)
507:                 or model_name in args.exclude_exact
508:                 or model_name in self.skip_models
509:             ):
510:                 continue
511:             yield model_name
512: 
513:     @property
514:     def skip_accuracy_checks_large_models_dashboard(self):
515:         if self.args.dashboard or self.args.accuracy:
516:             return self._accuracy["skip"]["large_models"]
517:         return set()
518: 
519:     @property
520:     def get_output_amp_train_process_func(self):
521:         return {}
522: 
523:     def pick_grad(self, name, is_training):
524:         if is_training:
525:             return torch.enable_grad()
526:         else:
527:             return torch.no_grad()
528: 
529:     def get_tolerance_and_cosine_flag(self, is_training, current_device, name):
530:         cosine = self.args.cosine
531:         if is_training:
532:             from torch._inductor import config as inductor_config
533: 
534:             if (name in self._config["tolerance"]["higher_training"]) or (
535:                 inductor_config.max_autotune
536:                 and name in self._config["tolerance"]["higher_max_autotune_training"]
537:             ):
538:                 return 2e-2, cosine
539:             else:
540:                 return 1e-2, cosine
````
- EN: Handles module imports such as `torch._inductor`.
- CN: 处理模块导入，例如 `torch._inductor`。
- EN: Implements callable logic such as `iter_model_names`, `skip_accuracy_checks_large_models_dashboard`, `get_output_amp_train_process_func`, `pick_grad`.
- CN: 实现可调用逻辑，例如 `iter_model_names`, `skip_accuracy_checks_large_models_dashboard`, `get_output_amp_train_process_func`, `pick_grad`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 541-600
````python
541:         else:
542:             if (
543:                 current_device == "cpu"
544:                 and name in self._config["tolerance"]["higher_inference_cpu"]
545:             ):
546:                 return 5e-3, cosine
547:             if name in self._config["tolerance"]["higher_inference"]:
548:                 return 4e-3, cosine
549:         return 1e-3, cosine
550: 
551:     def compute_loss(self, pred):
552:         return pred[0]
553: 
554:     def forward_pass(self, mod, inputs, collect_outputs=True):
555:         with torch.no_grad(), self.autocast(**self.autocast_arg):
556:             res = mod(**inputs)
557:         return res.logits if self.hf_llm else res
558: 
559:     def forward_and_backward_pass(self, mod, inputs, collect_outputs=True):
560:         cloned_inputs = clone_inputs(inputs)
561:         self.optimizer_zero_grad(mod)
562:         with self.autocast(**self.autocast_arg):
563:             pred = mod(**cloned_inputs)
564:             loss = self.compute_loss(pred)
565:         self.grad_scaler.scale(loss).backward()
566:         self.optimizer_step()
567:         if collect_outputs:
568:             return collect_results(mod, None, loss, cloned_inputs)
569:         return None
570: 
571: 
572: def refresh_model_names_and_batch_sizes():
573:     """
574:     This function reads the HF Fx tracer supported models and finds the largest
575:     batch size that could fit on the GPU with PyTorch eager.
576: 
577:     The resulting data is written in huggingface_models_list.txt.
578: 
579:     Note - We only need to run this function if we believe that HF Fx tracer now
580:     supports more models.
581:     """
582:     import transformers.utils.fx as hf_fx
583: 
584:     family = {}
585:     lm_seen = set()
586:     family_seen = set()
587:     for cls_name in hf_fx._SUPPORTED_MODELS:
588:         if "For" not in cls_name:
589:             continue
590: 
591:         model_cls = get_module_cls_by_model_name(cls_name)
592: 
593:         # TODO: AttributeError: '*Config' object has no attribute 'vocab_size'
594:         if model_cls in [
595:             CLIPModel,
596:             CLIPVisionModel,
597:             # SwinForImageClassification,
598:             # SwinForImageClassification,
599:             # SwinForMaskedImageModeling,
600:             # SwinModel,
````
- EN: Handles module imports such as `transformers.utils.fx`.
- CN: 处理模块导入，例如 `transformers.utils.fx`。
- EN: Implements callable logic such as `compute_loss`, `forward_pass`, `forward_and_backward_pass`, `refresh_model_names_and_batch_sizes`.
- CN: 实现可调用逻辑，例如 `compute_loss`, `forward_pass`, `forward_and_backward_pass`, `refresh_model_names_and_batch_sizes`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 601-660
````python
601:             ViTForImageClassification,
602:             ViTForMaskedImageModeling,
603:             ViTModel,
604:         ]:
605:             continue
606: 
607:         # TODO: AssertionError: Padding_idx must be within num_embeddings
608:         if model_cls in [MarianForCausalLM, MarianMTModel, MarianModel]:
609:             continue
610: 
611:         # TODO: "model is not supported yet" from HFTracer
612:         if model_cls in [HubertForSequenceClassification]:
613:             continue
614: 
615:         # TODO: shape mismatch in loss calculation
616:         if model_cls in [LxmertForQuestionAnswering]:
617:             continue
618: 
619:         family_name = cls_name.split("For")[0]
620:         if family_name not in family:
621:             family[family_name] = []
622:         if cls_name.endswith(("MaskedLM", "CausalLM")) and family_name not in lm_seen:
623:             family[family_name].append(cls_name)
624:             lm_seen.add(family_name)
625:         elif (
626:             cls_name.endswith(
627:                 ("SequenceClassification", "ConditionalGeneration", "QuestionAnswering")
628:             )
629:             and family_name not in family_seen
630:         ):
631:             family[family_name].append(cls_name)
632:             family_seen.add(family_name)
633:         elif cls_name.endswith("ImageClassification"):
634:             family[family_name].append(cls_name)
635: 
636:     chosen_models = set()
637:     for members in family.values():
638:         chosen_models.update(set(members))
639: 
640:     # Add the EXTRA_MODELS
641:     chosen_models.update(set(EXTRA_MODELS.keys()))
642: 
643:     for model_name in sorted(chosen_models):
644:         try:
645:             subprocess.check_call(
646:                 [sys.executable]
647:                 + sys.argv
648:                 + ["--find-batch-sizes"]
649:                 + [f"--only={model_name}"]
650:                 + [f"--output={MODELS_FILENAME}"]
651:             )
652:         except subprocess.SubprocessError:
653:             log.warning(f"Failed to find suitable batch size for {model_name}")  # noqa: G004
654: 
655: 
656: def huggingface_main():
657:     # Code to refresh model names and batch sizes
658:     # if "--find-batch-sizes" not in sys.argv:
659:     #     refresh_model_names_and_batch_sizes()
660:     logging.basicConfig(level=logging.WARNING)
````
- EN: Implements callable logic such as `huggingface_main`.
- CN: 实现可调用逻辑，例如 `huggingface_main`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 661-666
````python
661:     warnings.filterwarnings("ignore")
662:     main(HuggingfaceRunner())
663: 
664: 
665: if __name__ == "__main__":
666:     huggingface_main()
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `pip_install` / 符号 `pip_install`
- Symbol `process_hf_reformer_output` / 符号 `process_hf_reformer_output`
- Symbol `get_module_cls_by_model_name` / 符号 `get_module_cls_by_model_name`
- Symbol `get_sequence_length` / 符号 `get_sequence_length`

## Dependencies / 依赖关系
- Python imports: `importlib`, `logging`, `os`, `re`, `subprocess`, `sys`, `types`, `warnings`, `.common`, `common`
- Python 导入: `importlib`, `logging`, `os`, `re`, `subprocess`, `sys`, `types`, `warnings`, `.common`, `common`
