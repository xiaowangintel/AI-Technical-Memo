# bench.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/fastrnns/bench.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60
````python
 1: import argparse
 2: import copy
 3: import gc
 4: import json
 5: import sys
 6: import time
 7: from collections import namedtuple
 8: 
 9: import torch
10: from torch.autograd.profiler import record_function
11: 
12: from .fuser import set_fuser
13: from .runner import get_nn_runners
14: 
15: 
16: BenchResult = namedtuple(
17:     "BenchResult",
18:     [
19:         "name",
20:         "avg_fwd",
21:         "std_fwd",
22:         "info_fwd",
23:         "avg_bwd",
24:         "std_bwd",
25:         "info_bwd",
26:     ],
27: )
28: 
29: 
30: def fit_str(string, colwidth=16):
31:     if len(string) < colwidth:
32:         return (colwidth - len(string)) * " " + string
33:     else:
34:         return string[:colwidth]
35: 
36: 
37: def to_str(item):
38:     if isinstance(item, float):
39:         return f"{item:.4g}"
40:     return str(item)
41: 
42: 
43: def print_header(colwidth=16, sep=" "):
44:     items = []
45:     for item in BenchResult._fields:
46:         items.append(fit_str(item))
47:     return sep.join(items)
48: 
49: 
50: def pretty_print(benchresult, colwidth=16, sep=" "):
51:     items = []
52:     for thing in benchresult:
53:         items.append(fit_str(to_str(thing)))
54:     return sep.join(items)
55: 
56: 
57: # shim for torch.cuda.Event when running on cpu
58: class Event:
59:     def __init__(self, enable_timing):
60:         pass
````
- EN: Handles module imports such as `argparse`, `copy`, `gc`, `json`.
- CN: 处理模块导入，例如 `argparse`, `copy`, `gc`, `json`。
- EN: Declares or extends types including `Event`.
- CN: 声明或扩展类型，包括 `Event`。
- EN: Implements callable logic such as `fit_str`, `to_str`, `print_header`, `pretty_print`.
- CN: 实现可调用逻辑，例如 `fit_str`, `to_str`, `print_header`, `pretty_print`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 61-120
````python
 61: 
 62:     def record(self):
 63:         self.time = time.perf_counter()
 64: 
 65:     def elapsed_time(self, end_event):
 66:         if not isinstance(end_event, Event):
 67:             raise AssertionError(f"Expected Event, but got {type(end_event)}")
 68:         return end_event.time - self.time
 69: 
 70: 
 71: def trainbench(
 72:     name,
 73:     rnn_creator,
 74:     nloops=100,
 75:     warmup=10,
 76:     seqLength=100,
 77:     numLayers=1,
 78:     inputSize=512,
 79:     hiddenSize=512,
 80:     miniBatch=64,
 81:     device="cuda",
 82:     seed=None,
 83: ):
 84:     def train_batch(modeldef):
 85:         # CUDA events for timing
 86:         if device == "cuda":
 87:             timer_class = torch.cuda.Event
 88:         else:
 89:             timer_class = Event
 90: 
 91:         fwd_start_event = timer_class(enable_timing=True)
 92:         fwd_end_event = timer_class(enable_timing=True)
 93:         bwd_start_event = timer_class(enable_timing=True)
 94:         bwd_end_event = timer_class(enable_timing=True)
 95: 
 96:         gc.collect()
 97: 
 98:         fwd_start_event.record()
 99:         with record_function("## forward ##"):
100:             forward_output = modeldef.forward(*modeldef.inputs)
101:         fwd_end_event.record()
102: 
103:         # XXX: Use if need to print something
104:         # print(modeldef.forward.graph_for(*modeldef.inputs))
105: 
106:         if modeldef.backward_setup is not None:
107:             backward_input = modeldef.backward_setup(forward_output)
108:         else:
109:             backward_input = forward_output
110: 
111:         gc.collect()
112: 
113:         bwd_start_event.record()
114:         if modeldef.backward is not None:
115:             modeldef.backward(*backward_input)
116:         bwd_end_event.record()
117: 
118:         if modeldef.backward is not None:
119:             with torch.no_grad():
120:                 for param in modeldef.params:
````
- EN: Implements callable logic such as `record`, `elapsed_time`, `trainbench`, `train_batch`.
- CN: 实现可调用逻辑，例如 `record`, `elapsed_time`, `trainbench`, `train_batch`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-180
````python
121:                     if param.grad is None:
122:                         raise AssertionError("Parameter gradient must not be None")
123:                     param.grad.zero_()
124: 
125:         if device == "cuda":
126:             torch.cuda.synchronize()
127: 
128:         fwd_time = fwd_start_event.elapsed_time(fwd_end_event)
129:         bwd_time = bwd_start_event.elapsed_time(bwd_end_event)
130:         return fwd_time, bwd_time
131: 
132:     creator_args = {
133:         "seqLength": seqLength,
134:         "numLayers": numLayers,
135:         "inputSize": inputSize,
136:         "hiddenSize": hiddenSize,
137:         "miniBatch": miniBatch,
138:         "device": device,
139:         "seed": seed,
140:     }
141: 
142:     modeldef = rnn_creator(**creator_args)
143: 
144:     [train_batch(modeldef) for _ in range(warmup)]
145: 
146:     results = [train_batch(modeldef) for _ in range(nloops)]
147:     fwd_times, bwd_times = zip(*results)
148: 
149:     fwd_times = torch.tensor(fwd_times)
150:     bwd_times = torch.tensor(bwd_times)
151:     return BenchResult(
152:         name=name,
153:         avg_fwd=fwd_times.mean().item(),
154:         std_fwd=fwd_times.std().item(),
155:         info_fwd=fwd_times,
156:         avg_bwd=bwd_times.mean().item(),
157:         std_bwd=bwd_times.std().item(),
158:         info_bwd=bwd_times,
159:     )
160: 
161: 
162: def print_stderr(*args, **kwargs):
163:     kwargs["file"] = sys.stderr
164:     return print(*args, **kwargs)
165: 
166: 
167: def print_json_oss_format(results):
168:     oss_results = {}
169:     for group_name, group_val in results.items():
170:         oss_results[group_name] = {}
171:         for model_name, run_time in group_val.items():
172:             # Output for OSS
173:             oss_results[group_name][model_name] = run_time["avg"]
174: 
175:     print(json.dumps(oss_results))
176: 
177: 
178: def print_json_pep_format(results):
179:     # print the AI-PEP format json string for each model
180:     for group_name, group_val in results.items():
````
- EN: Implements callable logic such as `print_stderr`, `print_json_oss_format`, `print_json_pep_format`.
- CN: 实现可调用逻辑，例如 `print_stderr`, `print_json_oss_format`, `print_json_pep_format`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 181-240
````python
181:         for model_name, run_time in group_val.items():
182:             # Output for AI-PEP
183:             num_iters = len(run_time["info"])
184:             info = run_time["info"].tolist()
185:             for i in range(num_iters):
186:                 print(
187:                     "Caffe2Observer "
188:                     + json.dumps(
189:                         {
190:                             "type": "NET",
191:                             "metric": group_name + "-" + model_name,
192:                             "unit": "ms",
193:                             "value": str(info[i]),
194:                         }
195:                     )
196:                 )
197: 
198: 
199: def bench(rnn_runners, group_name, print_json=False, sep=" ", **params):
200:     print_stderr(print_header(sep=sep))
201:     results = {}
202:     for name, creator, context in rnn_runners:
203:         with context():
204:             try:
205:                 result = trainbench(name, creator, **params)
206:                 # Replace the value of info_fwd and info_bwd to None
207:                 result_with_no_info = result._replace(info_fwd="None", info_bwd="None")
208:                 print_stderr(pretty_print(result_with_no_info, sep=sep))
209:                 results[name] = result
210:             except Exception:
211:                 if not print_json:
212:                     raise
213: 
214:     return {
215:         group_name: {
216:             k: {"avg": v.avg_fwd, "std": v.std_fwd, "info": v.info_fwd}
217:             for k, v in results.items()
218:         },
219:         f"{group_name}-backward": {
220:             k: {"avg": v.avg_bwd, "std": v.std_bwd, "info": v.info_bwd}
221:             for k, v in results.items()
222:         },
223:     }
224: 
225: 
226: def bench_group(model_list, bench_name, bench_group, bench_args):
227:     print_stderr(f"Benchmarking {bench_name}s...")
228:     nn_results = bench(get_nn_runners(*model_list), bench_group, **bench_args)
229:     print_stderr("")
230:     return nn_results
231: 
232: 
233: if __name__ == "__main__":
234:     parser = argparse.ArgumentParser(description="Profile RNNs")
235: 
236:     # groups help control which test group you want to run
237:     # if you only want to run one/two benchmark, run it with
238:     # e.g: python -m fastrnns.bench --rnns jit and --group rnns
239:     default_groups = ["cnns", "rnns"]
240: 
````
- EN: Implements callable logic such as `bench`, `bench_group`.
- CN: 实现可调用逻辑，例如 `bench`, `bench_group`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-300
````python
241:     parser.add_argument("--seqLength", default="100", type=int)
242:     parser.add_argument("--numLayers", default="1", type=int)
243:     parser.add_argument("--inputSize", default="512", type=int)
244:     parser.add_argument("--hiddenSize", default="512", type=int)
245:     parser.add_argument("--miniBatch", default="64", type=int)
246:     parser.add_argument("--warmup", default="10", type=int)
247:     parser.add_argument("--nloops", default="100", type=int)
248:     parser.add_argument("--device", default="cuda", type=str)
249:     parser.add_argument(
250:         "--variable-lstms",
251:         "--variable_lstms",
252:         action="store_true",
253:         help="Also benchmark variable sequence length lstms "
254:         "Note that some of these run really slowly "
255:         "and that the `seqLength` flag will be ignored.",
256:     )
257:     parser.add_argument("--sep", default=" ", type=str)
258:     parser.add_argument("--print-json", nargs="?", default=None, const="oss")
259:     parser.add_argument("--rnns", nargs="*", help="What to run. cudnn, aten, jit, etc")
260:     parser.add_argument(
261:         "--cnns", nargs="*", help="What to run. resnet18, resnet18_jit, resnet50, etc"
262:     )
263:     parser.add_argument(
264:         "--group",
265:         nargs="*",
266:         default=default_groups,
267:         help="Which group to run. cnns, rnns, etc.",
268:     )
269:     parser.add_argument(
270:         "--fuser",
271:         default="te",
272:         type=str,
273:         help="The fuser backend to use. One of: te, old, or none",
274:     )
275:     parser.add_argument(
276:         "--executor",
277:         default=None,
278:         type=str,
279:         help="The executor to use. One of: legacy, simple, profiling",
280:     )
281:     parser.add_argument(
282:         "--cuda-pointwise-loop-level",
283:         "--cuda_pointwise_loop_level",
284:         default=None,
285:         type=int,
286:     )
287:     parser.add_argument(
288:         "--cuda-pointwise-block-count",
289:         "--cuda_pointwise_block_count",
290:         default=None,
291:         type=int,
292:     )
293:     parser.add_argument(
294:         "--cuda-pointwise-block-size",
295:         "--cuda_pointwise_block_size",
296:         default=None,
297:         type=int,
298:     )
299: 
300:     args = parser.parse_args()
````
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。

### Lines 301-360
````python
301:     set_fuser(args.fuser, args.executor)
302: 
303:     if args.cuda_pointwise_loop_level:
304:         torch._C._jit_set_te_cuda_pointwise_loop_levels(args.cuda_pointwise_loop_level)
305:     if args.cuda_pointwise_block_count:
306:         torch._C._jit_set_te_cuda_pointwise_block_count(args.cuda_pointwise_block_count)
307:     if args.cuda_pointwise_block_size:
308:         torch._C._jit_set_te_cuda_pointwise_block_size(args.cuda_pointwise_block_size)
309: 
310:     rnns = args.rnns or [
311:         "cudnn",
312:         "aten",
313:         "jit",
314:         "jit_premul",
315:         "jit_premul_bias",
316:         "jit_simple",
317:         "jit_multilayer",
318:         "py",
319:     ]
320:     cnns = args.cnns or ["resnet18", "resnet18_jit", "resnet50", "resnet50_jit"]
321:     # TODO: Maybe add a separate section for the layernorm/dropout lstms
322:     # 'cudnn_layernorm', jit_layernorm', 'jit_layernom_decom',
323:     # 'jit', 'jit_dropout', 'cudnn_dropout'
324:     vlrnns = ["vl_cudnn", "vl_jit", "vl_py"]
325: 
326:     if args.print_json:
327:         print_stderr = lambda *args, **kwargs: None  # noqa: E731
328:     print_stderr(args)
329: 
330:     bench_args = copy.deepcopy(vars(args))
331:     should_bench_varlen_lstms = args.variable_lstms
332:     del bench_args["group"]
333:     del bench_args["rnns"]
334:     del bench_args["cnns"]
335:     del bench_args["variable_lstms"]
336:     del bench_args["fuser"]
337:     del bench_args["executor"]
338:     del bench_args["cuda_pointwise_loop_level"]
339:     del bench_args["cuda_pointwise_block_count"]
340:     del bench_args["cuda_pointwise_block_size"]
341: 
342:     results = {}
343:     if should_bench_varlen_lstms:
344:         if args.nloops + args.warmup > 30:
345:             print_stderr(
346:                 "WARNING: some of the variable sequence length lstms are "
347:                 "very unoptimized and therefore take forever to run."
348:             )
349:         results.update(
350:             bench_group(vlrnns, "variable-length sequence LSTM", "vl_lstm", bench_args)
351:         )
352: 
353:     if "rnns" in args.group:
354:         results.update(bench_group(rnns, "LSTM", "lstm", bench_args))
355:     if "cnns" in args.group:
356:         results.update(bench_group(cnns, "ResNet", "resnet", bench_args))
357: 
358:     if args.print_json == "oss":
359:         print_json_oss_format(results)
360:     elif args.print_json == "pep":
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 361-361
````python
361:         print_json_pep_format(results)
````
- EN: This range contributes implementation details for the file goal: Implements benchmark definitions, helpers, or runners for performance measurement.
- CN: 该范围为文件目标提供实现细节：实现用于性能测量的基准定义、辅助工具或运行器。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `fit_str` / 符号 `fit_str`
- Symbol `to_str` / 符号 `to_str`
- Symbol `print_header` / 符号 `print_header`
- Symbol `pretty_print` / 符号 `pretty_print`

## Dependencies / 依赖关系
- Python imports: `argparse`, `copy`, `gc`, `json`, `sys`, `time`, `collections`, `torch`, `torch.autograd.profiler`, `.fuser`
- Python 导入: `argparse`, `copy`, `gc`, `json`, `sys`, `time`, `collections`, `torch`, `torch.autograd.profiler`, `.fuser`
