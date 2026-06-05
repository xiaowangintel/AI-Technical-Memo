# utils.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/dynamo/genai_layers/utils.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60
````python
 1: import os
 2: import sys
 3: from collections import defaultdict
 4: from collections.abc import Callable
 5: from dataclasses import dataclass
 6: from typing import Any
 7: 
 8: import matplotlib.pyplot as plt
 9: from scipy.stats import gmean
10: 
11: import torch
12: from torch._inductor.runtime.benchmarking import benchmarker
13: 
14: 
15: def benchmark_kernel_in_milliseconds(func: Callable, *args, **kwargs) -> float:
16:     # warmup
17:     for _ in range(5):
18:         func(*args, **kwargs)
19:     with torch.compiler.set_stance("fail_on_recompile"):
20:         return benchmarker.benchmark_gpu(lambda: func(*args, **kwargs))
21: 
22: 
23: @dataclass
24: class Performance:
25:     # Benchmark setting usually the shape of the input tensor
26:     setting: str
27: 
28:     # Latency in milliseconds
29:     latency: float
30: 
31:     # Number of  memory access in bytes
32:     memory_bytes: float
33: 
34:     # Memory bandwidth in GB/s
35:     memory_bandwidth: float = 0.0
36: 
37:     # Compute intensity in FLOPs/byte
38:     compute_intensity: float = 0.0
39: 
40:     def __post_init__(self):
41:         self.memory_bandwidth = self.memory_bytes / (self.latency / 1000) / 1e9
42: 
43:     def __str__(self):
44:         return f"setting: {self.setting}, latency: {self.latency} ms, memory bandwidth: {self.memory_bandwidth} GB/s"
45: 
46: 
47: class BenchmarkKernel:
48:     def __init__(self, script_args):
49:         self.script_args = script_args
50:         self.name = self.__class__.__name__
51:         self.available_backends: list[str] = []
52:         self.compile_mode: str = script_args.compile_mode
53: 
54:         # mapping from backend to list of performance results
55:         self.profiling_results: defaultdict[str, list[Performance]] = defaultdict(list)
56: 
57:     def get_memory_bytes(self, args, kwargs) -> int:
58:         # Get the necessary memory access in bytes for the kernelßß
59:         raise NotImplementedError
60: 
````
- EN: Handles module imports such as `os`, `sys`, `collections`, `collections.abc`.
- CN: 处理模块导入，例如 `os`, `sys`, `collections`, `collections.abc`。
- EN: Declares or extends types including `Performance`, `BenchmarkKernel`.
- CN: 声明或扩展类型，包括 `Performance`, `BenchmarkKernel`。
- EN: Implements callable logic such as `benchmark_kernel_in_milliseconds`, `__post_init__`, `__str__`, `__init__`.
- CN: 实现可调用逻辑，例如 `benchmark_kernel_in_milliseconds`, `__post_init__`, `__str__`, `__init__`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。

### Lines 61-120
````python
 61:     def get_shapes(self) -> tuple[tuple[int, ...], ...]:
 62:         # Get a list of input shapes to benchmark the kernel
 63:         raise NotImplementedError
 64: 
 65:     def eager(self, args, kwargs) -> Any:
 66:         raise NotImplementedError
 67: 
 68:     def compiled(self, args, kwargs) -> Any:
 69:         raise NotImplementedError
 70: 
 71:     def helion(self, args, kwargs) -> Any:
 72:         raise NotImplementedError
 73: 
 74:     def quack(self, args, kwargs) -> Any:
 75:         raise NotImplementedError
 76: 
 77:     def liger(self, args, kwargs) -> Any:
 78:         raise NotImplementedError
 79: 
 80:     def triton(self, args, kwargs) -> Any:
 81:         raise NotImplementedError
 82: 
 83:     def benchmark(self):
 84:         raise NotImplementedError
 85: 
 86:     def clone_inputs(self, args, kwargs) -> Any:
 87:         args_ref = [
 88:             arg.clone().detach().requires_grad_(arg.requires_grad) for arg in args
 89:         ]
 90: 
 91:         kwargs_ref = (
 92:             {
 93:                 k: (
 94:                     v.clone().detach().requires_grad_(v.requires_grad)
 95:                     if isinstance(v, torch.Tensor)
 96:                     else v
 97:                 )
 98:                 for k, v in kwargs.items()
 99:             }
100:             if kwargs
101:             else kwargs
102:         )
103: 
104:         return args_ref, kwargs_ref
105: 
106:     def check_accuracy(self, args, kwargs) -> None:
107:         res = {}
108:         for backend in self.available_backends:
109:             args_ref, kwargs_ref = self.clone_inputs(args, kwargs)
110:             res[backend] = getattr(self, backend)(args_ref, kwargs_ref)()
111: 
112:         if (
113:             "compiled" in self.available_backends
114:             and self.script_args.custom_compile_options
115:         ):
116:             torch._dynamo.reset()  # cause recompile
117:             with torch._inductor.config.patch(self.script_args.custom_compile_options):
118:                 args_ref, kwargs_ref = self.clone_inputs(args, kwargs)
119:                 res[self.script_args.custom_compile_name] = self.compiled(
120:                     args_ref, kwargs_ref
````
- EN: Implements callable logic such as `get_shapes`, `eager`, `compiled`, `helion`.
- CN: 实现可调用逻辑，例如 `get_shapes`, `eager`, `compiled`, `helion`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-180
````python
121:                 )()
122: 
123:         gold = res["eager"]
124: 
125:         tol = {}
126:         if self.script_args.tolerance:
127:             tol = {
128:                 "atol": self.script_args.tolerance,
129:                 "rtol": self.script_args.tolerance,
130:             }
131:         for backend in res:
132:             if backend == "eager":
133:                 continue
134:             try:
135:                 torch.testing.assert_close(res[backend], gold, **tol)
136:                 for t, gold_t in zip(res[backend], gold):
137:                     if t.requires_grad:
138:                         torch.testing.assert_close(t.grad, gold_t.grad, **tol)
139:                 print(
140:                     f"Accuracy check \033[92m✓ succeed\033[0m for {backend} backend on {self.name} kernel"
141:                 )
142:             except Exception as e:
143:                 print(
144:                     f"Accuracy check \033[91m✗ failed\033[0m for {backend} backend on {self.name} kernel. Error {e}"
145:                 )
146:                 if self.script_args.exit_on_accuracy_failure:
147:                     print("Exit right away since --exit-on-accuracy-failure is set")
148:                     sys.exit(1)
149: 
150:     def benchmark_single_shape_for_backend(
151:         self, backend, args, kwargs, setting, fn=None
152:     ) -> bool:
153:         if fn is None:
154:             fn = getattr(self, backend)
155:         args_ref, kwargs_ref = self.clone_inputs(args, kwargs)
156:         try:
157:             avg_time = benchmark_kernel_in_milliseconds(fn(args_ref, kwargs_ref))
158:         except Exception as e:
159:             print(
160:                 f"Failed to run {backend} backend on {self.name} kernel for {setting} due to {e}"
161:             )
162:             self.available_backends.remove(backend)  # noqa: B909
163:             return False
164:         mem_bytes = self.get_memory_bytes(args_ref, kwargs_ref)
165:         perf = Performance(setting, avg_time, mem_bytes)
166:         print(f"{self.name} kernel on {backend} backend. {perf}")
167:         self.profiling_results[backend].append(perf)
168:         return True
169: 
170:     def benchmark_single_shape(
171:         self, args, kwargs=None, should_check_accuracy=True, setting: str = ""
172:     ):
173:         for backend in self.available_backends:
174:             self.benchmark_single_shape_for_backend(backend, args, kwargs, setting)
175:         if (
176:             "compiled" in self.available_backends
177:             and self.script_args.custom_compile_options
178:         ):
179:             torch._dynamo.reset()  # cause recompile
180:             with torch._inductor.config.patch(self.script_args.custom_compile_options):
````
- EN: Implements callable logic such as `benchmark_single_shape_for_backend`, `benchmark_single_shape`.
- CN: 实现可调用逻辑，例如 `benchmark_single_shape_for_backend`, `benchmark_single_shape`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 181-240
````python
181:                 status = self.benchmark_single_shape_for_backend(
182:                     self.script_args.custom_compile_name,
183:                     args,
184:                     kwargs,
185:                     setting,
186:                     fn=self.compiled,
187:                 )
188:             if not status:
189:                 self.script_args.custom_compile_options = (
190:                     None  # once fail, don't run again
191:                 )
192: 
193:         if should_check_accuracy:
194:             self.check_accuracy(args, kwargs)
195: 
196:     def visualize(self) -> None:
197:         device_name = torch.cuda.get_device_name(0)
198:         visualize_comparison(
199:             self.profiling_results,
200:             title=f"{self.name} ({device_name})",
201:             output_path=f"{self.name}_bench",
202:         )
203:         return
204: 
205:     def report_geomean_speedup(self) -> None:
206:         print(f"Geomean speedup for benchmark {self.name}")
207:         eager_result = {
208:             result.setting: result for result in self.profiling_results["eager"]
209:         }
210:         print(f"  eager {len(eager_result)} data points")
211:         for backend, backend_result in self.profiling_results.items():
212:             if backend == "eager":
213:                 continue
214:             speeduplist = []
215:             for result in backend_result:
216:                 eager_latency = eager_result[result.setting].latency
217:                 backend_latency = result.latency
218:                 speeduplist.append(
219:                     eager_latency / backend_latency if backend_latency != 0 else 0.0
220:                 )
221: 
222:             if len(speeduplist) > 0:
223:                 print(
224:                     f"  {backend} {len(speeduplist)} data points, {gmean(speeduplist):.2f}x speedup"
225:                 )
226: 
227: 
228: def get_backend_colors() -> dict[str, str]:
229:     """Get consistent color scheme for different backends."""
230:     return {
231:         "eager": "#1f77b4",  # blue
232:         "compiled": "#ff7f0e",  # orange
233:         "quack": "#2ca02c",  # green
234:         "liger": "#d62728",  # red
235:         "helion": "#9467bd",  # purple
236:         "triton": "#8c564b",  # brown
237:         "cutlass": "#e377c2",  # pink
238:         "flash_attn": "#7f7f7f",  # gray
239:         "default": "#000000",  # black
240:     }
````
- EN: Implements callable logic such as `visualize`, `report_geomean_speedup`, `get_backend_colors`.
- CN: 实现可调用逻辑，例如 `visualize`, `report_geomean_speedup`, `get_backend_colors`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-300
````python
241: 
242: 
243: def visualize_comparison(
244:     profiling_results: dict[str, list[Performance]],
245:     title: str | None = None,
246:     output_path: str | None = None,
247: ) -> None:
248:     """
249:     Create a single memory_bandwidth comparison plot from profiling results.
250: 
251:     Args:
252:         profiling_results: Dict mapping backend names to lists of Performance objects
253:         output_path: Path to save the plot (optional)
254:     """
255:     # Get backend colors
256:     backend_colors = get_backend_colors()
257: 
258:     # Extract settings from eager backend which runs all settings
259:     all_settings = []
260:     for perf in profiling_results["eager"]:
261:         all_settings.append(perf.setting)
262: 
263:     # Create single plot
264:     fig, ax = plt.subplots(1, 1, figsize=(12, 8))
265: 
266:     for backend in profiling_results:
267:         backend_perfs = profiling_results[backend]
268:         perf_dict = {perf.setting: perf for perf in backend_perfs}
269: 
270:         x_vals = []
271:         y_vals = []
272:         for i, setting in enumerate(all_settings):
273:             if setting in perf_dict:
274:                 x_vals.append(i)
275:                 y_vals.append(perf_dict[setting].memory_bandwidth)
276: 
277:         if x_vals:  # Only plot if we have data
278:             color = backend_colors.get(backend, backend_colors["default"])
279:             ax.plot(
280:                 x_vals,
281:                 y_vals,
282:                 "o-",
283:                 label=backend,
284:                 color=color,
285:                 linewidth=2,
286:                 markersize=8,
287:                 alpha=0.8,
288:             )
289: 
290:     # Configure the plot
291:     ax.set_title(title or "Memory Bandwidth Comparison", fontsize=16)
292:     ax.set_xlabel("Shape", fontsize=12)
293:     ax.set_ylabel("memory bandwidth (GB/s)", fontsize=12)
294:     ax.set_xticks(range(len(all_settings)))
295:     ax.set_xticklabels(
296:         [
297:             s.replace("shape: ", "").replace("[", "").replace("]", "")
298:             for s in all_settings
299:         ],
300:         rotation=45,
````
- EN: Implements callable logic such as `visualize_comparison`.
- CN: 实现可调用逻辑，例如 `visualize_comparison`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 301-316
````python
301:         ha="right",
302:     )
303:     ax.legend(fontsize=10)
304:     ax.grid(True, alpha=0.3)
305: 
306:     plt.tight_layout()
307: 
308:     # Save the plot if output path is provided
309:     if output_path:
310:         # Save as PNG
311:         os.makedirs("pics", exist_ok=True)
312:         full_path = os.path.join("pics", output_path + ".png")
313:         plt.savefig(full_path, dpi=300, bbox_inches="tight", facecolor="white")
314:         print(f"Chart saved to {full_path}")
315: 
316:     plt.close()
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `benchmark_kernel_in_milliseconds` / 符号 `benchmark_kernel_in_milliseconds`
- Symbol `Performance` / 符号 `Performance`
- Symbol `__post_init__` / 符号 `__post_init__`
- Symbol `__str__` / 符号 `__str__`

## Dependencies / 依赖关系
- Python imports: `os`, `sys`, `collections`, `collections.abc`, `dataclasses`, `typing`, `matplotlib.pyplot`, `scipy.stats`, `torch`, `torch._inductor.runtime.benchmarking`
- Python 导入: `os`, `sys`, `collections`, `collections.abc`, `dataclasses`, `typing`, `matplotlib.pyplot`, `scipy.stats`, `torch`, `torch._inductor.runtime.benchmarking`
