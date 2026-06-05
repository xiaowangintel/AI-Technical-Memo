# benchmark.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/gpt_fast/benchmark.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60
````python
 1: import argparse
 2: import csv
 3: import dataclasses
 4: import json
 5: import os
 6: 
 7: from common import all_experiments, Experiment, register_experiment
 8: from generate import get_arch_name
 9: 
10: import torch
11: import torch.nn as nn
12: from torch._inductor.runtime.benchmarking import benchmarker
13: from torch.utils.flop_counter import FlopCounterMode
14: 
15: 
16: WARMUP_ITER = 5
17: 
18: A100_40G_BF16_TFLOPS = 312
19: 
20: 
21: class SimpleMLP(nn.Module):
22:     def __init__(self, input_dim, hidden_dim, output_dim, dtype):
23:         super().__init__()
24:         self.layers = nn.ModuleList(
25:             [
26:                 nn.Linear(input_dim, hidden_dim, dtype=dtype),
27:                 nn.LayerNorm(hidden_dim, dtype=dtype),
28:                 nn.Linear(hidden_dim, output_dim, dtype=dtype),
29:                 nn.LayerNorm(output_dim, dtype=dtype),
30:             ]
31:         )
32: 
33:     def forward(self, x):
34:         for layer in self.layers:
35:             x = layer(x)
36:         return x
37: 
38: 
39: @register_experiment(name="mlp_layer_norm_gelu")
40: def run_mlp_layer_norm_gelu(device: str = "cuda"):
41:     dtype_flops_utilization_map = {
42:         torch.bfloat16: "0.8",
43:     }
44:     input_shapes = [1024, 4096, 8192, 16384]
45:     intermediate_size = 14336
46:     results = []
47:     for dtype, expected_flops_utilization in dtype_flops_utilization_map.items():
48:         flops_utilization = 0
49:         for D in input_shapes:
50:             mod = SimpleMLP(
51:                 input_dim=D, hidden_dim=intermediate_size, output_dim=D, dtype=dtype
52:             ).to(device)
53: 
54:             x = torch.randn(D, device=device, dtype=torch.bfloat16)
55: 
56:             with FlopCounterMode(display=False) as mode:
57:                 mod(x)
58: 
59:             flops = mode.get_total_flops()
60: 
````
- EN: Handles module imports such as `argparse`, `csv`, `dataclasses`, `json`.
- CN: 处理模块导入，例如 `argparse`, `csv`, `dataclasses`, `json`。
- EN: Declares or extends types including `SimpleMLP`.
- CN: 声明或扩展类型，包括 `SimpleMLP`。
- EN: Implements callable logic such as `__init__`, `forward`, `run_mlp_layer_norm_gelu`.
- CN: 实现可调用逻辑，例如 `__init__`, `forward`, `run_mlp_layer_norm_gelu`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。

### Lines 61-120
````python
 61:             compiled_mod = torch.compile(mod, dynamic=False)
 62: 
 63:             for _ in range(WARMUP_ITER):
 64:                 compiled_mod(x)
 65: 
 66:             us_per_iter = benchmarker.benchmark(compiled_mod, (x,), {}) * 1000
 67:             flops_utilization += us_per_iter * flops / 1e9 / A100_40G_BF16_TFLOPS
 68: 
 69:         flops_utilization = flops_utilization / len(input_shapes)
 70:         dtype_str = str(dtype).replace("torch.", "")
 71:         results.append(
 72:             Experiment(
 73:                 "mlp_layer_norm_gelu",
 74:                 "flops_utilization",
 75:                 expected_flops_utilization,
 76:                 f"{flops_utilization:.02f}",
 77:                 dtype_str,
 78:                 device,
 79:                 get_arch_name(),
 80:             )
 81:         )
 82:     return results
 83: 
 84: 
 85: @register_experiment(name="layer_norm")
 86: def run_layer_norm(device: str = "cuda"):
 87:     dtype_memory_bandwidth_map = {
 88:         torch.bfloat16: "950",
 89:     }
 90:     input_shapes = [1024, 4096, 8192, 16384]
 91:     BS = 4096
 92:     results = []
 93:     for dtype, expected_memory_bandwidth in dtype_memory_bandwidth_map.items():
 94:         memory_bandwidth = 0
 95:         for D in input_shapes:
 96:             mod = nn.LayerNorm(D).to(device)
 97: 
 98:             x = torch.randn(BS, D, device=device, dtype=dtype)
 99: 
100:             compiled_mod = torch.compile(mod, dynamic=False)
101: 
102:             for _ in range(WARMUP_ITER):
103:                 compiled_mod(x)
104: 
105:             us_per_iter = benchmarker.benchmark(compiled_mod, (x,), {}) * 1000
106:             memory_bandwidth += (1e6 / us_per_iter) * 2 * BS * D * dtype.itemsize / 1e9
107: 
108:         memory_bandwidth = memory_bandwidth / len(input_shapes)
109:         dtype_str = str(dtype).replace("torch.", "")
110:         results.append(
111:             Experiment(
112:                 "layer_norm",
113:                 "memory_bandwidth(GB/s)",
114:                 expected_memory_bandwidth,
115:                 f"{memory_bandwidth:.02f}",
116:                 dtype_str,
117:                 device,
118:                 get_arch_name(),
119:             )
120:         )
````
- EN: Implements callable logic such as `run_layer_norm`.
- CN: 实现可调用逻辑，例如 `run_layer_norm`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-180
````python
121:     return results
122: 
123: 
124: @register_experiment(name="gather_gemv")
125: @torch._inductor.config.patch(coordinate_descent_tuning=True)
126: def run_gather_gemv(device: str = "cuda"):
127:     E = 8
128:     dtype_memory_bandwidth_map = {
129:         torch.int8: "990",
130:         torch.bfloat16: "1060",
131:     }
132:     input_shapes = [1024, 4096, 8192, 16384]
133:     results = []
134:     for dtype, expected_memory_bandwidth in dtype_memory_bandwidth_map.items():
135:         memory_bandwidth = 0
136:         for D in input_shapes:
137: 
138:             def gather_gemv(W, score_idxs, x):
139:                 return W[score_idxs].to(x.dtype) @ x
140: 
141:             W = torch.randn(E, D, D, device=device).to(dtype=dtype)
142:             x = torch.randn(D, device=device, dtype=torch.bfloat16)
143:             score_idxs = torch.tensor([3, 5], device=device)
144: 
145:             compiled_fn = torch.compile(gather_gemv, dynamic=False)
146: 
147:             for _ in range(WARMUP_ITER):
148:                 compiled_fn(W, score_idxs, x)
149: 
150:             us_per_iter = (
151:                 benchmarker.benchmark(
152:                     compiled_fn,
153:                     (
154:                         W,
155:                         score_idxs,
156:                         x,
157:                     ),
158:                     {},
159:                 )
160:                 * 1000
161:             )
162:             memory_bandwidth += (1e6 / us_per_iter) * 2 * D * D * dtype.itemsize / 1e9
163: 
164:         memory_bandwidth = memory_bandwidth / len(input_shapes)
165:         dtype_str = str(dtype).replace("torch.", "")
166:         results.append(
167:             Experiment(
168:                 "gather_gemv",
169:                 "memory_bandwidth(GB/s)",
170:                 expected_memory_bandwidth,
171:                 f"{memory_bandwidth:.02f}",
172:                 dtype_str,
173:                 device,
174:                 get_arch_name(),
175:             )
176:         )
177:     return results
178: 
179: 
180: @register_experiment(name="gemv")
````
- EN: Implements callable logic such as `run_gather_gemv`, `gather_gemv`.
- CN: 实现可调用逻辑，例如 `run_gather_gemv`, `gather_gemv`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 181-240
````python
181: @torch._inductor.config.patch(coordinate_descent_tuning=True)
182: def run_gemv(device: str = "cuda"):
183:     dtype_memory_bandwidth_map = {
184:         torch.int8: "870",
185:         torch.bfloat16: "990",
186:     }
187:     input_shapes = [1024, 4096, 8192, 16384]
188:     results = []
189:     for dtype, expected_memory_bandwidth in dtype_memory_bandwidth_map.items():
190:         memory_bandwidth = 0
191:         for D in input_shapes:
192: 
193:             def gemv(W, x):
194:                 return W.to(x.dtype) @ x
195: 
196:             W = torch.randn(D, D, device=device).to(dtype=dtype)
197:             x = torch.randn(D, device=device, dtype=torch.bfloat16)
198: 
199:             compiled_fn = torch.compile(gemv, dynamic=False)
200: 
201:             for _ in range(WARMUP_ITER):
202:                 compiled_fn(W, x)
203: 
204:             us_per_iter = (
205:                 benchmarker.benchmark(
206:                     compiled_fn,
207:                     (
208:                         W,
209:                         x,
210:                     ),
211:                     {},
212:                 )
213:                 * 1000
214:             )
215:             memory_bandwidth += (1e6 / us_per_iter) * D * D * dtype.itemsize / 1e9
216: 
217:         memory_bandwidth = memory_bandwidth / len(input_shapes)
218:         dtype_str = str(dtype).replace("torch.", "")
219:         results.append(
220:             Experiment(
221:                 "gemv",
222:                 "memory_bandwidth(GB/s)",
223:                 expected_memory_bandwidth,
224:                 f"{memory_bandwidth:.02f}",
225:                 dtype_str,
226:                 device,
227:                 get_arch_name(),
228:             )
229:         )
230:     return results
231: 
232: 
233: def output_csv(output_file, headers, row):
234:     if os.path.exists(output_file):
235:         with open(output_file) as fd:
236:             lines = list(csv.reader(fd)) or [[]]
237:             if headers and len(headers) > len(lines[0]):
238:                 # if prior results failed the header might not be filled in yet
239:                 lines[0] = headers
240:             else:
````
- EN: Implements callable logic such as `run_gemv`, `gemv`, `output_csv`.
- CN: 实现可调用逻辑，例如 `run_gemv`, `gemv`, `output_csv`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-300
````python
241:                 headers = lines[0]
242:     else:
243:         lines = [headers]
244: 
245:     if output_file != DEFAULT_OUTPUT_FILE:
246:         os.makedirs(os.path.dirname(output_file), exist_ok=True)
247:     lines.append([(f"{x:.6f}" if isinstance(x, float) else x) for x in row])
248:     with open(output_file, "w") as fd:
249:         writer = csv.writer(fd, lineterminator="\n")
250:         for line in lines:
251:             writer.writerow(list(line) + ["0"] * (len(headers) - len(line)))
252: 
253: 
254: def output_json(output_file, headers, row):
255:     """
256:     Write the result into JSON format, so that it can be uploaded to the benchmark database
257:     to be displayed on OSS dashboard. The JSON format is defined at
258:     https://github.com/pytorch/pytorch/wiki/How-to-integrate-with-PyTorch-OSS-benchmark-database
259:     """
260:     mapping_headers = {headers[i]: v for i, v in enumerate(row)}
261:     record = {
262:         "benchmark": {
263:             "name": "PyTorch gpt-fast benchmark",
264:             "mode": "inference",
265:             "dtype": mapping_headers["dtype"],
266:             "extra_info": {
267:                 "device": mapping_headers["device"],
268:                 "arch": mapping_headers["arch"],
269:             },
270:         },
271:         "model": {
272:             "name": mapping_headers["name"],
273:             "type": "OSS model" if mapping_headers["is_model"] else "micro-benchmark",
274:             "origins": ["pytorch"],
275:         },
276:         "metric": {
277:             "name": mapping_headers["metric"],
278:             "benchmark_values": [mapping_headers["actual"]],
279:             "target_value": mapping_headers["target"],
280:         },
281:     }
282: 
283:     with open(f"{os.path.splitext(output_file)[0]}.json", "a") as f:
284:         print(json.dumps(record), file=f)
285: 
286: 
287: DEFAULT_OUTPUT_FILE = "gpt_fast_benchmark.csv"
288: 
289: 
290: def main(output_file=DEFAULT_OUTPUT_FILE, only_model=None):
291:     results = []
292: 
293:     if not only_model:
294:         experiments = all_experiments.values()
295:     else:
296:         if only_model not in all_experiments:
297:             print(
298:                 f"Unknown model: {only_model}, all available models: {all_experiments.keys()}"
299:             )
300:         # only run the specified model
````
- EN: Implements callable logic such as `output_json`, `main`.
- CN: 实现可调用逻辑，例如 `output_json`, `main`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 301-335
````python
301:         experiments = [all_experiments[only_model]]
302:     for func in experiments:
303:         try:
304:             device = "cuda" if torch.cuda.is_available() else "cpu"
305:         except AssertionError:
306:             # This happens when torch is compiled with CUDA turning off completely
307:             device = "cpu"
308: 
309:         torch.compiler.cudagraph_mark_step_begin()
310:         lst = func(device)
311:         for x in lst:
312:             results.append(dataclasses.astuple(x))
313: 
314:     headers = [field.name for field in dataclasses.fields(Experiment)]
315: 
316:     for row in results:
317:         output_csv(output_file, headers, row)
318:         # Also write the output in JSON format so that it can be ingested into the OSS benchmark database
319:         output_json(output_file, headers, row)
320: 
321: 
322: if __name__ == "__main__":
323:     parser = argparse.ArgumentParser(description="Run experiments.")
324:     parser.add_argument(
325:         "--output",
326:         default=DEFAULT_OUTPUT_FILE,
327:         help="Set the output CSV file to save the benchmark results",
328:     )
329:     parser.add_argument(
330:         "--only",
331:         help="Specify a model or micro-benchmark name to run exclusively",
332:     )
333:     args = parser.parse_args()
334: 
335:     main(output_file=args.output, only_model=args.only)
````
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `SimpleMLP` / 符号 `SimpleMLP`
- Symbol `__init__` / 符号 `__init__`
- Symbol `forward` / 符号 `forward`
- Symbol `run_mlp_layer_norm_gelu` / 符号 `run_mlp_layer_norm_gelu`

## Dependencies / 依赖关系
- Python imports: `argparse`, `csv`, `dataclasses`, `json`, `os`, `common`, `generate`, `torch`, `torch.nn`, `torch._inductor.runtime.benchmarking`
- Python 导入: `argparse`, `csv`, `dataclasses`, `json`, `os`, `common`, `generate`, `torch`, `torch.nn`, `torch._inductor.runtime.benchmarking`
