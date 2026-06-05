# timm_models.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/dynamo/timm_models.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60
````python
 1: #!/usr/bin/env python3
 2: 
 3: import importlib
 4: import logging
 5: import os
 6: import re
 7: import subprocess
 8: import sys
 9: import warnings
10: 
11: 
12: try:
13:     from .common import BenchmarkRunner, download_retry_decorator, load_yaml_file, main
14: except ImportError:
15:     from common import BenchmarkRunner, download_retry_decorator, load_yaml_file, main
16: 
17: import torch
18: from torch._dynamo.testing import collect_results, reduce_to_scalar_loss
19: from torch._dynamo.utils import clone_inputs
20: 
21: 
22: # Enable FX graph caching
23: if "TORCHINDUCTOR_FX_GRAPH_CACHE" not in os.environ:
24:     torch._inductor.config.fx_graph_cache = True
25: 
26: 
27: def pip_install(package):
28:     subprocess.check_call([sys.executable, "-m", "pip", "install", package])
29: 
30: 
31: try:
32:     importlib.import_module("timm")
33: except ModuleNotFoundError:
34:     print("Installing PyTorch Image Models...")
35:     pip_install("git+https://github.com/rwightman/pytorch-image-models")
36: finally:
37:     from timm import __version__ as timmversion
38:     from timm.data import resolve_data_config
39:     from timm.models import create_model
40: 
41: TIMM_MODELS = {}
42: 
43: # Run only this selected group of models, leave this empty to run everything
44: TORCHBENCH_ONLY_MODELS = [
45:     m.strip() for m in os.getenv("TORCHBENCH_ONLY_MODELS", "").split(",") if m.strip()
46: ]
47: 
48: filename = os.path.join(os.path.dirname(__file__), "timm_models_list.txt")
49: with open(filename) as fh:
50:     lines = fh.readlines()
51:     lines = [line.rstrip() for line in lines]
52:     for line in lines:
53:         model_name, batch_size = line.split(" ")
54:         if TORCHBENCH_ONLY_MODELS and model_name not in TORCHBENCH_ONLY_MODELS:
55:             continue
56:         TIMM_MODELS[model_name] = int(batch_size)
57: 
58: 
59: def refresh_model_names():
60:     import glob
````
- EN: Handles module imports such as `importlib`, `logging`, `os`, `re`.
- CN: 处理模块导入，例如 `importlib`, `logging`, `os`, `re`。
- EN: Implements callable logic such as `pip_install`, `refresh_model_names`.
- CN: 实现可调用逻辑，例如 `pip_install`, `refresh_model_names`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 61-120
````python
 61: 
 62:     from timm.models import list_models
 63: 
 64:     def read_models_from_docs():
 65:         models = set()
 66:         # TODO - set the path to pytorch-image-models repo
 67:         for fn in glob.glob("../pytorch-image-models/docs/models/*.md"):
 68:             with open(fn) as f:
 69:                 while True:
 70:                     line = f.readline()
 71:                     if not line:
 72:                         break
 73:                     if not line.startswith("model = timm.create_model("):
 74:                         continue
 75: 
 76:                     model = line.split("'")[1]
 77:                     # print(model)
 78:                     models.add(model)
 79:         return models
 80: 
 81:     def get_family_name(name):
 82:         known_families = [
 83:             "darknet",
 84:             "densenet",
 85:             "dla",
 86:             "dpn",
 87:             "ecaresnet",
 88:             "halo",
 89:             "regnet",
 90:             "efficientnet",
 91:             "deit",
 92:             "mobilevit",
 93:             "mnasnet",
 94:             "convnext",
 95:             "resnet",
 96:             "resnest",
 97:             "resnext",
 98:             "selecsls",
 99:             "vgg",
100:             "xception",
101:         ]
102: 
103:         for known_family in known_families:
104:             if known_family in name:
105:                 return known_family
106: 
107:         if name.startswith("gluon_"):
108:             return "gluon_" + name.split("_")[1]
109:         return name.split("_")[0]
110: 
111:     def populate_family(models):
112:         family = {}
113:         for model_name in models:
114:             family_name = get_family_name(model_name)
115:             if family_name not in family:
116:                 family[family_name] = []
117:             family[family_name].append(model_name)
118:         return family
119: 
120:     docs_models = read_models_from_docs()
````
- EN: Handles module imports such as `timm.models`.
- CN: 处理模块导入，例如 `timm.models`。
- EN: Implements callable logic such as `read_models_from_docs`, `get_family_name`, `populate_family`.
- CN: 实现可调用逻辑，例如 `read_models_from_docs`, `get_family_name`, `populate_family`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-180
````python
121:     all_models = list_models(pretrained=True, exclude_filters=["*in21k"])
122: 
123:     all_models_family = populate_family(all_models)
124:     docs_models_family = populate_family(docs_models)
125: 
126:     for key in docs_models_family:
127:         del all_models_family[key]
128: 
129:     chosen_models = set()
130:     chosen_models.update(value[0] for value in docs_models_family.values())
131: 
132:     chosen_models.update(value[0] for key, value in all_models_family.items())
133: 
134:     filename = "timm_models_list.txt"
135:     if os.path.exists("benchmarks"):
136:         filename = "benchmarks/" + filename
137:     with open(filename, "w") as fw:
138:         for model_name in sorted(chosen_models):
139:             fw.write(model_name + "\n")
140: 
141: 
142: class TimmRunner(BenchmarkRunner):
143:     def __init__(self):
144:         super().__init__()
145:         self.suite_name = "timm_models"
146: 
147:     @property
148:     def _config(self):
149:         return load_yaml_file("timm_models.yaml")
150: 
151:     @property
152:     def _skip(self):
153:         return self._config["skip"]
154: 
155:     @property
156:     def _batch_size(self):
157:         return self._config["batch_size"]
158: 
159:     @property
160:     def _tolerance(self):
161:         return self._config["tolerance"]
162: 
163:     @property
164:     def _accuracy(self):
165:         return self._config["accuracy"]
166: 
167:     @property
168:     def _require_larger_multiplier_for_smaller_tensor(self):
169:         return self._config["require_larger_multiplier_for_smaller_tensor"]
170: 
171:     @property
172:     def skip_models_for_cpu(self):
173:         return self._skip["device"]["cpu"]
174: 
175:     @property
176:     def skip_models_for_cpu_aarch64(self):
177:         return self._skip["device"]["cpu_aarch64"]
178: 
179:     @property
180:     def skip_models(self):
````
- EN: Declares or extends types including `TimmRunner`.
- CN: 声明或扩展类型，包括 `TimmRunner`。
- EN: Implements callable logic such as `__init__`, `_config`, `_skip`, `_batch_size`.
- CN: 实现可调用逻辑，例如 `__init__`, `_config`, `_skip`, `_batch_size`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 181-240
````python
181:         return self._skip["all"]
182: 
183:     @property
184:     def force_amp_for_fp16_bf16_models(self):
185:         return self._config["dtype"]["force_amp_for_fp16_bf16_models"]
186: 
187:     @property
188:     def force_fp16_for_bf16_models(self):
189:         return set()
190: 
191:     @property
192:     def get_output_amp_train_process_func(self):
193:         return {}
194: 
195:     @property
196:     def skip_accuracy_check_as_eager_non_deterministic(self):
197:         if self.args.accuracy and self.args.training:
198:             return self._accuracy["skip"]["eager_not_deterministic"]
199:         return set()
200: 
201:     @property
202:     def guard_on_nn_module_models(self):
203:         return {}
204: 
205:     @download_retry_decorator
206:     def _download_model(self, model_name):
207:         model = create_model(
208:             model_name,
209:             in_chans=3,
210:             scriptable=False,
211:             num_classes=None,
212:             drop_rate=0.0,
213:             drop_path_rate=None,
214:             drop_block_rate=None,
215:             pretrained=True,
216:         )
217:         return model
218: 
219:     def load_model(
220:         self,
221:         device,
222:         model_name,
223:         batch_size=None,
224:         extra_args=None,
225:     ):
226:         if self.args.enable_activation_checkpointing:
227:             raise NotImplementedError(
228:                 "Activation checkpointing not implemented for Timm models"
229:             )
230: 
231:         is_training = self.args.training
232:         use_eval_mode = self.args.use_eval_mode
233: 
234:         channels_last = self._args.channels_last
235:         model = self._download_model(model_name)
236: 
237:         if model is None:
238:             raise RuntimeError(f"Failed to load model '{model_name}'")
239:         model.to(
240:             device=device,
````
- EN: Implements callable logic such as `force_amp_for_fp16_bf16_models`, `force_fp16_for_bf16_models`, `get_output_amp_train_process_func`, `skip_accuracy_check_as_eager_non_deterministic`.
- CN: 实现可调用逻辑，例如 `force_amp_for_fp16_bf16_models`, `force_fp16_for_bf16_models`, `get_output_amp_train_process_func`, `skip_accuracy_check_as_eager_non_deterministic`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-300
````python
241:             memory_format=torch.channels_last if channels_last else None,
242:         )
243: 
244:         data_config = resolve_data_config(
245:             vars(self._args) if timmversion >= "0.8.0" else self._args,
246:             model=model,
247:             use_test_size=not is_training,
248:         )
249:         input_size = data_config["input_size"]
250:         recorded_batch_size = TIMM_MODELS[model_name]
251: 
252:         batch_size_divisors = self._batch_size["divisors"]
253:         if model_name in batch_size_divisors:
254:             recorded_batch_size = max(
255:                 int(recorded_batch_size / batch_size_divisors[model_name]), 1
256:             )
257:         batch_size = batch_size or recorded_batch_size
258: 
259:         torch.manual_seed(1337)
260:         input_tensor = torch.randint(
261:             256, size=(batch_size,) + input_size, device=device
262:         ).to(dtype=torch.float32)
263:         mean = torch.mean(input_tensor)
264:         std_dev = torch.std(input_tensor)
265:         example_inputs = (input_tensor - mean) / std_dev
266: 
267:         if channels_last:
268:             example_inputs = example_inputs.contiguous(
269:                 memory_format=torch.channels_last
270:             )
271:         example_inputs = [
272:             example_inputs,
273:         ]
274: 
275:         self.loss = torch.nn.CrossEntropyLoss().to(device)
276: 
277:         if model_name in self._config["scaled_compute_loss"]:
278:             self.compute_loss = self.scaled_compute_loss
279: 
280:         if is_training and not use_eval_mode:
281:             model.train()
282:         else:
283:             model.eval()
284: 
285:         self.validate_model(model_name, model, example_inputs)
286: 
287:         return device, model_name, model, example_inputs, batch_size
288: 
289:     def iter_model_names(self, args):
290:         # for model_name in list_models(pretrained=True, exclude_filters=["*in21k"]):
291:         model_names = sorted(TIMM_MODELS.keys())
292:         start, end = self.get_benchmark_indices(len(model_names))
293:         for index, model_name in enumerate(model_names):
294:             if index < start or index >= end:
295:                 continue
296:             if (
297:                 not re.search("|".join(args.filter), model_name, re.IGNORECASE)
298:                 or re.search("|".join(args.exclude), model_name, re.IGNORECASE)
299:                 or model_name in args.exclude_exact
300:                 or model_name in self.skip_models
````
- EN: Implements callable logic such as `iter_model_names`.
- CN: 实现可调用逻辑，例如 `iter_model_names`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 301-360
````python
301:             ):
302:                 continue
303: 
304:             yield model_name
305: 
306:     def pick_grad(self, name, is_training):
307:         if is_training:
308:             return torch.enable_grad()
309:         else:
310:             return torch.no_grad()
311: 
312:     def use_larger_multiplier_for_smaller_tensor(self, name):
313:         return name in self._require_larger_multiplier_for_smaller_tensor
314: 
315:     def get_tolerance_and_cosine_flag(self, is_training, current_device, name):
316:         cosine = self.args.cosine
317:         tolerance = 1e-3
318: 
319:         if self.args.freezing and name in self._tolerance["freezing"]:
320:             # the conv-batchnorm fusion used under freezing may cause relatively
321:             # large numerical difference. We need a larger tolerance.
322:             # Check https://github.com/pytorch/pytorch/issues/120545 for context
323:             tolerance = 8 * 1e-2
324: 
325:         if is_training:
326:             from torch._inductor import config as inductor_config
327: 
328:             if name in self._tolerance["highest_training"]:
329:                 tolerance = 16 * 1e-2
330:             elif name in self._tolerance["even_higher"] or (
331:                 inductor_config.max_autotune
332:                 and name in self._tolerance["even_higher_max_autotune"]
333:             ):
334:                 tolerance = 8 * 1e-2
335:             elif name in self._tolerance["higher_training"] or (
336:                 self.args.amp and name in self._tolerance["higher_amp"]
337:             ):
338:                 tolerance = 4 * 1e-2
339:             elif (
340:                 name in self._tolerance["higher_fp16_xpu"]
341:                 and self.args.float16
342:                 and current_device == "xpu"
343:             ):
344:                 tolerance = 4 * 1e-2
345:             else:
346:                 tolerance = 1e-2
347:         return tolerance, cosine
348: 
349:     def compute_loss(self, pred):
350:         # High loss values make gradient checking harder, as small changes in
351:         # accumulation order upsets accuracy checks.
352:         return reduce_to_scalar_loss(pred)
353: 
354:     def scaled_compute_loss(self, pred):
355:         # Loss values need zoom out further.
356:         return reduce_to_scalar_loss(pred) / 1000.0
357: 
358:     def forward_pass(self, mod, inputs, collect_outputs=True):
359:         with self.autocast(**self.autocast_arg):
360:             return mod(*inputs)
````
- EN: Handles module imports such as `torch._inductor`.
- CN: 处理模块导入，例如 `torch._inductor`。
- EN: Implements callable logic such as `pick_grad`, `use_larger_multiplier_for_smaller_tensor`, `get_tolerance_and_cosine_flag`, `compute_loss`.
- CN: 实现可调用逻辑，例如 `pick_grad`, `use_larger_multiplier_for_smaller_tensor`, `get_tolerance_and_cosine_flag`, `compute_loss`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 361-384
````python
361: 
362:     def forward_and_backward_pass(self, mod, inputs, collect_outputs=True):
363:         cloned_inputs = clone_inputs(inputs)
364:         self.optimizer_zero_grad(mod)
365:         with self.autocast(**self.autocast_arg):
366:             pred = mod(*cloned_inputs)
367:             if isinstance(pred, tuple):
368:                 pred = pred[0]
369:             loss = self.compute_loss(pred)
370:         self.grad_scaler.scale(loss).backward()
371:         self.optimizer_step()
372:         if collect_outputs:
373:             return collect_results(mod, None, loss, cloned_inputs)
374:         return None
375: 
376: 
377: def timm_main():
378:     logging.basicConfig(level=logging.WARNING)
379:     warnings.filterwarnings("ignore")
380:     main(TimmRunner())
381: 
382: 
383: if __name__ == "__main__":
384:     timm_main()
````
- EN: Implements callable logic such as `forward_and_backward_pass`, `timm_main`.
- CN: 实现可调用逻辑，例如 `forward_and_backward_pass`, `timm_main`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `pip_install` / 符号 `pip_install`
- Symbol `refresh_model_names` / 符号 `refresh_model_names`
- Symbol `read_models_from_docs` / 符号 `read_models_from_docs`
- Symbol `get_family_name` / 符号 `get_family_name`

## Dependencies / 依赖关系
- Python imports: `importlib`, `logging`, `os`, `re`, `subprocess`, `sys`, `warnings`, `.common`, `common`, `torch`
- Python 导入: `importlib`, `logging`, `os`, `re`, `subprocess`, `sys`, `warnings`, `.common`, `common`, `torch`
