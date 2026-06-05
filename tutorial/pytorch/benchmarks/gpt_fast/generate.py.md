# generate.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/gpt_fast/generate.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60
````python
 1: import dataclasses
 2: import itertools
 3: import platform
 4: import time
 5: 
 6: from common import Experiment, register_experiment
 7: from mixtral_moe_model import ConditionalFeedForward, Transformer as MixtralMoE
 8: from mixtral_moe_quantize import (
 9:     ConditionalFeedForwardInt8,
10:     WeightOnlyInt8QuantHandler as MixtralMoEWeightOnlyInt8QuantHandler,
11: )
12: from model import Transformer as LLaMA
13: from quantize import WeightOnlyInt8QuantHandler as LLaMAWeightOnlyInt8QuantHandler
14: 
15: import torch
16: import torch._inductor.config
17: 
18: 
19: torch._inductor.config.coordinate_descent_tuning = True
20: torch._inductor.config.triton.unique_kernel_names = True
21: torch._inductor.config.fx_graph_cache = True  # Experimental feature to reduce compilation times, will be on by default in future
22: torch._inductor.config.assert_indirect_indexing = False
23: 
24: compiled = False
25: 
26: 
27: @dataclasses.dataclass
28: class GPTModelConfig:
29:     name: str
30:     module: type
31:     mode: str | None
32:     quantizer: type
33:     token_per_sec: float
34:     memory_bandwidth: float
35:     compilation_time: float
36:     batch_size: int | None = None
37: 
38: 
39: def device_sync(device):
40:     if "cuda" in device:
41:         torch.cuda.synchronize(device)
42:     elif "cpu" in device:
43:         pass
44:     else:
45:         print(f"device={device} is not yet supported")
46: 
47: 
48: def get_arch_name() -> str:
49:     if torch.cuda.is_available():
50:         return torch.cuda.get_device_name()
51:     else:
52:         # This returns x86_64 or arm64 (for aarch64)
53:         return platform.machine()
54: 
55: 
56: def multinomial_sample_one_no_sync(
57:     probs_sort,
58: ):  # Does multinomial sampling without a cuda synchronization
59:     q = torch.empty_like(probs_sort).exponential_(1)
60:     return torch.argmax(probs_sort / q, dim=-1, keepdim=True).to(dtype=torch.int)
````
- EN: Handles module imports such as `dataclasses`, `itertools`, `platform`, `time`.
- CN: 处理模块导入，例如 `dataclasses`, `itertools`, `platform`, `time`。
- EN: Declares or extends types including `GPTModelConfig`.
- CN: 声明或扩展类型，包括 `GPTModelConfig`。
- EN: Implements callable logic such as `device_sync`, `get_arch_name`, `multinomial_sample_one_no_sync`.
- CN: 实现可调用逻辑，例如 `device_sync`, `get_arch_name`, `multinomial_sample_one_no_sync`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 61-120
````python
 61: 
 62: 
 63: def logits_to_probs(logits, temperature: float = 1.0, top_k: int | None = None):
 64:     logits = logits / max(temperature, 1e-5)
 65: 
 66:     if top_k is not None:
 67:         v, _ = torch.topk(logits, min(top_k, logits.size(-1)))
 68:         pivot = v.select(-1, -1).unsqueeze(-1)
 69:         logits = torch.where(logits < pivot, -float("Inf"), logits)
 70:     probs = torch.nn.functional.softmax(logits, dim=-1)
 71:     return probs
 72: 
 73: 
 74: def sample(logits, temperature: float = 1.0, top_k: int | None = None):
 75:     probs = logits_to_probs(logits[0, -1], temperature, top_k)
 76:     idx_next = multinomial_sample_one_no_sync(probs)
 77:     return idx_next, probs
 78: 
 79: 
 80: def prefill(
 81:     model: torch.nn.Module, x: torch.Tensor, input_pos: torch.Tensor, **sampling_kwargs
 82: ) -> torch.Tensor:
 83:     # input_pos: [B, S]
 84:     logits = model(x, input_pos)
 85:     return sample(logits, **sampling_kwargs)[0]
 86: 
 87: 
 88: def decode_one_token(
 89:     model: torch.nn.Module, x: torch.Tensor, input_pos: torch.Tensor, **sampling_kwargs
 90: ) -> tuple[torch.Tensor, torch.Tensor]:
 91:     # input_pos: [B, 1]
 92:     if input_pos.shape[-1] != 1:
 93:         raise AssertionError(
 94:             f"input_pos.shape[-1] must be 1, but got {input_pos.shape[-1]}"
 95:         )
 96:     logits = model(x, input_pos)
 97:     return sample(logits, **sampling_kwargs)
 98: 
 99: 
100: def decode_n_tokens(
101:     model: torch.nn.Module,
102:     cur_token: torch.Tensor,
103:     input_pos: torch.Tensor,
104:     num_new_tokens: int,
105:     **sampling_kwargs,
106: ):
107:     new_tokens, new_probs = [], []
108:     for i in range(num_new_tokens):
109:         with torch.nn.attention.sdpa_kernel(
110:             torch.nn.attention.SDPBackend.MATH
111:         ):  # Actually better for Inductor to codegen attention here
112:             next_token, next_prob = decode_one_token(
113:                 model, cur_token, input_pos, **sampling_kwargs
114:             )
115:             input_pos += 1
116:             new_tokens.append(next_token.clone())
117:             new_probs.append(next_prob.clone())
118:             cur_token = next_token.view(1, -1)
119: 
120:     return new_tokens, new_probs
````
- EN: Implements callable logic such as `logits_to_probs`, `sample`, `prefill`, `decode_one_token`.
- CN: 实现可调用逻辑，例如 `logits_to_probs`, `sample`, `prefill`, `decode_one_token`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-180
````python
121: 
122: 
123: @torch.no_grad()
124: def generate(
125:     model: torch.nn.Module, prompt: torch.Tensor, max_new_tokens: int, **sampling_kwargs
126: ) -> torch.Tensor:
127:     device, dtype = prompt.device, prompt.dtype
128:     T = prompt.size(0)
129:     T_new = T + max_new_tokens
130:     max_seq_length = min(T_new, model.config.block_size)
131: 
132:     with torch.device(device):
133:         model.setup_caches(max_batch_size=1, max_seq_length=max_seq_length)
134: 
135:     # create an empty tensor of the expected final shape and fill in the current tokens
136:     empty = torch.empty(T_new, dtype=dtype, device=device)
137:     empty[:T] = prompt
138:     seq = empty
139:     input_pos = torch.arange(0, T, device=device)
140: 
141:     next_token = prefill(model, prompt.view(1, -1), input_pos, **sampling_kwargs)
142:     seq[T] = next_token
143: 
144:     input_pos = torch.tensor([T], device=device, dtype=torch.int)
145: 
146:     generated_tokens, _ = decode_n_tokens(
147:         model, next_token.view(1, -1), input_pos, max_new_tokens - 1, **sampling_kwargs
148:     )
149:     seq[T + 1 :] = torch.cat(generated_tokens)
150:     return seq
151: 
152: 
153: def _load_model(x: GPTModelConfig, device="cuda", precision=torch.bfloat16):
154:     with torch.device("meta"):
155:         model = x.module.from_name(x.name)
156:     model = model.to(dtype=precision)
157: 
158:     if x.mode == "int8":
159:         print("Using int8 weight-only quantization!")
160:         model = x.quantizer(model).convert_for_runtime()
161: 
162:     state_dict = model.state_dict()
163:     for k, v in state_dict.items():
164:         state_dict[k] = torch.nn.Parameter(
165:             torch.randn(v.shape, device=device).to(dtype=v.dtype),
166:             requires_grad=v.requires_grad,
167:         )
168:     model.load_state_dict(state_dict, assign=True)
169:     return model.eval()
170: 
171: 
172: # Only count activated parameters and buffers.
173: def _get_model_size(model):
174:     model_size = 0
175:     for name, child in model.named_children():
176:         if not isinstance(child, torch.nn.Embedding):
177:             model_size += sum(
178:                 p.numel() * p.dtype.itemsize
179:                 for p in itertools.chain(child.parameters(), child.buffers())
180:             )
````
- EN: Implements callable logic such as `generate`, `_load_model`, `_get_model_size`.
- CN: 实现可调用逻辑，例如 `generate`, `_load_model`, `_get_model_size`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 181-240
````python
181: 
182:     # Remove the inactivated experts from the model size if this is mixture of experts
183:     # architecture, since only activated experts are loaded.
184:     if hasattr(model.config, "num_experts"):
185:         config = model.config
186:         for submodule in model.modules():
187:             if isinstance(
188:                 submodule, (ConditionalFeedForward, ConditionalFeedForwardInt8)
189:             ):
190:                 model_size -= (
191:                     sum(
192:                         p.numel() * p.dtype.itemsize
193:                         for p in itertools.chain(
194:                             submodule.parameters(), child.buffers()
195:                         )
196:                     )
197:                     * (config.num_experts - config.num_activated_experts)
198:                     / config.num_experts
199:                 )
200: 
201:     return model_size
202: 
203: 
204: def run_experiment(
205:     x: GPTModelConfig,
206:     num_samples: int = 5,
207:     max_new_tokens: int = 200,
208:     top_k: int = 200,
209:     temperature: float = 0.8,
210:     device: str = "cuda",
211: ) -> None:
212:     print(f"Loading model {x.name}")
213:     t0 = time.time()
214:     model = _load_model(x, device=device)
215:     device_sync(device=device)  # MKG
216:     print(f"Time to load model: {time.time() - t0:.02f} seconds")
217: 
218:     prompt = torch.tensor(
219:         [1, 15043, 29892, 590, 1024, 338], device=device, dtype=torch.int32
220:     )
221:     prompt_length = prompt.size(0)
222: 
223:     torch.manual_seed(1234)
224:     model_size = _get_model_size(model)
225: 
226:     aggregate_metrics = {"tokens_per_sec": [], "memory_bandwidth": []}
227:     start = -1
228:     compilation_time = None
229: 
230:     global decode_one_token, prefill, compiled
231:     if not compiled:
232:         compiled = True
233:         decode_one_token = torch.compile(
234:             decode_one_token, mode="reduce-overhead", fullgraph=True
235:         )
236:         prefill = torch.compile(prefill, fullgraph=True)
237: 
238:     for i in range(start, num_samples):
239:         device_sync(device=device)  # MKG
240: 
````
- EN: Implements callable logic such as `run_experiment`.
- CN: 实现可调用逻辑，例如 `run_experiment`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-300
````python
241:         torch.compiler.cudagraph_mark_step_begin()
242:         t0 = time.perf_counter()
243:         y = generate(
244:             model, prompt, max_new_tokens, temperature=temperature, top_k=top_k
245:         )
246: 
247:         if i == -1:
248:             compilation_time = time.perf_counter() - t0
249:             print(f"Compilation time: {compilation_time:.2f} seconds")
250:             continue
251: 
252:         device_sync(device=device)  # MKG
253:         t = time.perf_counter() - t0
254:         tokens_generated = y.size(0) - prompt_length
255:         tokens_sec = tokens_generated / t
256:         aggregate_metrics["tokens_per_sec"].append(tokens_sec)
257:         aggregate_metrics["memory_bandwidth"].append(model_size * tokens_sec / 1e9)
258: 
259:     token_per_sec = torch.mean(torch.tensor(aggregate_metrics["tokens_per_sec"])).item()
260:     memory_bandwidth = torch.mean(
261:         torch.tensor(aggregate_metrics["memory_bandwidth"])
262:     ).item()
263:     print(f"Average tokens/sec: {token_per_sec:.2f} tokens/sec")
264:     print(f"Average bandwidth achieved: {memory_bandwidth:.02f} GB/s")
265:     print(f"Memory used: {torch.cuda.max_memory_reserved() / 1e9:.02f} GB")
266:     return token_per_sec, memory_bandwidth, compilation_time
267: 
268: 
269: # token_per_sec and memory_bandwidth target numbers are for A100-40GB, which are different from the typical A100-80GB.
270: @register_experiment(name="llama2_7b_bf16")
271: def run_llama2_7b_bf16(device: str = "cuda"):
272:     model = GPTModelConfig(
273:         "Llama-2-7b-chat-hf",
274:         LLaMA,
275:         "bfloat16",
276:         LLaMAWeightOnlyInt8QuantHandler,
277:         94,
278:         1253,
279:         133,
280:     )
281:     token_per_sec, memory_bandwidth, compilation_time = run_experiment(
282:         model, device=device
283:     )
284:     return [
285:         Experiment(
286:             model.name,
287:             "token_per_sec",
288:             model.token_per_sec,
289:             f"{token_per_sec:.02f}",
290:             model.mode,
291:             device,
292:             get_arch_name(),
293:             True,
294:         ),
295:         Experiment(
296:             model.name,
297:             "memory_bandwidth(GB/s)",
298:             model.memory_bandwidth,
299:             f"{memory_bandwidth:.02f}",
300:             model.mode,
````
- EN: Implements callable logic such as `run_llama2_7b_bf16`.
- CN: 实现可调用逻辑，例如 `run_llama2_7b_bf16`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 301-360
````python
301:             device,
302:             get_arch_name(),
303:             True,
304:         ),
305:         Experiment(
306:             model.name,
307:             "compilation_time(s)",
308:             model.compilation_time,
309:             f"{compilation_time:.02f}",
310:             model.mode,
311:             device,
312:             get_arch_name(),
313:             True,
314:         ),
315:     ]
316: 
317: 
318: # token_per_sec and memory_bandwidth target numbers are for A100-40GB, which are different from the typical A100-80GB.
319: @register_experiment(name="llama2_7b_int8")
320: def run_llama2_7b_int8(device: str = "cuda"):
321:     model = GPTModelConfig(
322:         "Llama-2-7b-chat-hf",
323:         LLaMA,
324:         "int8",
325:         LLaMAWeightOnlyInt8QuantHandler,
326:         144,
327:         957,
328:         136,
329:     )
330:     token_per_sec, memory_bandwidth, compilation_time = run_experiment(
331:         model, device=device
332:     )
333:     return [
334:         Experiment(
335:             model.name,
336:             "token_per_sec",
337:             model.token_per_sec,
338:             f"{token_per_sec:.02f}",
339:             model.mode,
340:             device,
341:             get_arch_name(),
342:             True,
343:         ),
344:         Experiment(
345:             model.name,
346:             "memory_bandwidth(GB/s)",
347:             model.memory_bandwidth,
348:             f"{memory_bandwidth:.02f}",
349:             model.mode,
350:             device,
351:             get_arch_name(),
352:             True,
353:         ),
354:         Experiment(
355:             model.name,
356:             "compilation_time(s)",
357:             model.compilation_time,
358:             f"{compilation_time:.02f}",
359:             model.mode,
360:             device,
````
- EN: Implements callable logic such as `run_llama2_7b_int8`.
- CN: 实现可调用逻辑，例如 `run_llama2_7b_int8`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 361-414
````python
361:             get_arch_name(),
362:             True,
363:         ),
364:     ]
365: 
366: 
367: # token_per_sec and memory_bandwidth target numbers are for A100-40GB, which are different from the typical A100-80GB.
368: @register_experiment(name="mixtral_8x7b_int8")
369: def run_mixtral_8x7b_int8(device: str = "cuda"):
370:     # We reduced the original number of layers from 32 to 16 to adapt CI memory limitation.
371:     model = GPTModelConfig(
372:         "Mixtral-8x7B-v0.1",
373:         MixtralMoE,
374:         "int8",
375:         MixtralMoEWeightOnlyInt8QuantHandler,
376:         175,
377:         1130,
378:         133,
379:     )
380:     token_per_sec, memory_bandwidth, compilation_time = run_experiment(
381:         model, device=device
382:     )
383:     return [
384:         Experiment(
385:             model.name,
386:             "token_per_sec",
387:             model.token_per_sec,
388:             f"{token_per_sec:.02f}",
389:             model.mode,
390:             device,
391:             get_arch_name(),
392:             True,
393:         ),
394:         Experiment(
395:             model.name,
396:             "memory_bandwidth(GB/s)",
397:             model.memory_bandwidth,
398:             f"{memory_bandwidth:.02f}",
399:             model.mode,
400:             device,
401:             get_arch_name(),
402:             True,
403:         ),
404:         Experiment(
405:             model.name,
406:             "compilation_time(s)",
407:             model.compilation_time,
408:             f"{compilation_time:.02f}",
409:             model.mode,
410:             device,
411:             get_arch_name(),
412:             True,
413:         ),
414:     ]
````
- EN: Implements callable logic such as `run_mixtral_8x7b_int8`.
- CN: 实现可调用逻辑，例如 `run_mixtral_8x7b_int8`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `GPTModelConfig` / 符号 `GPTModelConfig`
- Symbol `device_sync` / 符号 `device_sync`
- Symbol `get_arch_name` / 符号 `get_arch_name`
- Symbol `multinomial_sample_one_no_sync` / 符号 `multinomial_sample_one_no_sync`

## Dependencies / 依赖关系
- Python imports: `dataclasses`, `itertools`, `platform`, `time`, `common`, `mixtral_moe_model`, `mixtral_moe_quantize`, `model`, `quantize`, `torch`
- Python 导入: `dataclasses`, `itertools`, `platform`, `time`, `common`, `mixtral_moe_model`, `mixtral_moe_quantize`, `model`, `quantize`, `torch`
