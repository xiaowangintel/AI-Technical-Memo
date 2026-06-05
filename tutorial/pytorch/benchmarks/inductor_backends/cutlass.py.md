# cutlass.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/inductor_backends/cutlass.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60
````python
 1: import os
 2: import sys
 3: 
 4: 
 5: os.environ["TORCH_LOGS"] = "inductor"
 6: 
 7: import itertools
 8: import logging
 9: import time
10: from abc import abstractmethod
11: from collections import defaultdict
12: from collections.abc import Callable
13: from dataclasses import asdict, dataclass, field
14: from typing import Any
15: 
16: from tabulate import tabulate
17: from tqdm import tqdm
18: from triton.testing import do_bench
19: 
20: import torch
21: from torch._inductor import config as inductor_config
22: from torch.testing._internal.inductor_utils import _quantize_rowwise
23: 
24: 
25: log: logging.Logger = logging.getLogger(__name__)
26: 
27: 
28: inductor_config.autotune_num_choices_displayed = None
29: # force autotuning, but reuse compilation artifacts
30: inductor_config.autotune_local_cache = False
31: # uncomment for better debugging
32: # inductor_config.force_disable_caches = True
33: 
34: USE_FAST_ACCUM = True
35: 
36: UNITS = {
37:     "name": "",
38:     "forward_time": " (us)",
39:     "teraflops": " (TFLOPS)",
40:     "compilation_time": " (s)",
41: }
42: PERF_OVER_ATEN_STR: str = "perf_over_aten (%)"
43: 
44: OP_NAMES = [
45:     "mm",
46:     # "addmm",
47:     # "bmm",
48:     # "_scaled_mm",
49: ]
50: 
51: SHAPES = [
52:     # M, N, K
53:     (1024, 1024, 1024),
54:     (2048, 2048, 2048),
55:     (8192, 8192, 8192),
56: ]
57: 
58: BATCH_SIZES = [
59:     # For non-bmm testing, still need to specify something
60:     8,
````
- EN: Handles module imports such as `os`, `sys`, `itertools`, `logging`.
- CN: 处理模块导入，例如 `os`, `sys`, `itertools`, `logging`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 61-120
````python
 61: ]
 62: 
 63: DTYPES = [
 64:     torch.float16,
 65:     torch.bfloat16,
 66:     # torch.float8_e4m3fn,
 67: ]
 68: 
 69: # triton knobs
 70: ENABLE_PERSISTENT_TMA_MATMULS = [
 71:     False,
 72:     True,
 73: ]
 74: 
 75: # cutlass knobs
 76: CUTLASS_INSTANTIATION_LEVELS = [
 77:     "0",
 78:     # "1111",
 79:     # "2222",
 80:     "3332",
 81:     # "9992",
 82: ]
 83: 
 84: 
 85: def benchmark_torch_function_in_microseconds(func: Callable, *args, **kwargs) -> float:
 86:     return do_bench(lambda: func(*args, **kwargs), warmup=100, rep=10000) * 1e3
 87: 
 88: 
 89: @dataclass(frozen=True, kw_only=True)
 90: class ExperimentConfig:
 91:     max_autotune: bool = True
 92:     coordinate_descent_tuning: bool = True
 93:     max_autotune_gemm_backends: str = "ATEN"
 94: 
 95:     @abstractmethod
 96:     def name(self) -> str:
 97:         pass
 98: 
 99:     def to_options(self) -> dict[str, Any]:
100:         return {
101:             "max_autotune": self.max_autotune,
102:             "coordinate_descent_tuning": self.coordinate_descent_tuning,
103:             "max_autotune_gemm_backends": self.max_autotune_gemm_backends,
104:         }
105: 
106: 
107: @dataclass(frozen=True, kw_only=True)
108: class AtenExperimentConfig(ExperimentConfig):
109:     def name(self) -> str:
110:         return "aten"
111: 
112: 
113: @dataclass(frozen=True, kw_only=True)
114: class CutlassExperimentConfig(ExperimentConfig):
115:     cutlass_instantiation_level: str
116: 
117:     def name(self) -> str:
118:         level_name = (
119:             self.cutlass_instantiation_level
120:             if self.cutlass_instantiation_level != "0"
````
- EN: Declares or extends types including `ExperimentConfig`, `AtenExperimentConfig`, `CutlassExperimentConfig`.
- CN: 声明或扩展类型，包括 `ExperimentConfig`, `AtenExperimentConfig`, `CutlassExperimentConfig`。
- EN: Implements callable logic such as `benchmark_torch_function_in_microseconds`, `name`, `to_options`.
- CN: 实现可调用逻辑，例如 `benchmark_torch_function_in_microseconds`, `name`, `to_options`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 121-180
````python
121:             else "default"
122:         )
123:         return f"cutlass_lvl_{level_name}"
124: 
125:     def to_options(self) -> dict[str, Any]:
126:         return {
127:             **super().to_options(),
128:             "cutlass.cutlass_instantiation_level": self.cutlass_instantiation_level,
129:         }
130: 
131: 
132: @dataclass(frozen=True, kw_only=True)
133: class TritonExperimentConfig(ExperimentConfig):
134:     enable_persistent_tma_matmul: bool = False
135: 
136:     def name(self) -> str:
137:         if self.enable_persistent_tma_matmul:
138:             return "triton_persistent_tma"
139:         else:
140:             return "triton"
141: 
142:     def to_options(self) -> dict[str, Any]:
143:         return {
144:             **super().to_options(),
145:             "triton.enable_persistent_tma_matmul": self.enable_persistent_tma_matmul,
146:         }
147: 
148: 
149: @dataclass(frozen=True, kw_only=True)
150: class ExperimentGroupConfig:
151:     op_name: str
152:     shape: tuple[int, int, int]
153:     dtype: torch.dtype
154:     batch_size: int
155: 
156:     experiments: list[ExperimentConfig] = field(default_factory=list)
157: 
158:     def name(self) -> str:
159:         M, N, K = self.shape
160:         B = self.batch_size
161:         sizes = (
162:             f"(BS: {B}, {M}x{K}, {K}x{N})"
163:             if self.op_name == "bmm"
164:             else f"({M}x{K}, {K}x{N})"
165:         )
166:         return f"{self.op_name} {sizes} {self.dtype}"
167: 
168: 
169: @dataclass(frozen=True, kw_only=True)
170: class ExperimentResults:
171:     name: str
172:     forward_time: float
173:     teraflops: float
174:     compilation_time: float
175: 
176:     def asdict(self):
177:         return asdict(self)
178: 
179: 
180: @dataclass(frozen=True, kw_only=True)
````
- EN: Declares or extends types including `TritonExperimentConfig`, `ExperimentGroupConfig`, `ExperimentResults`.
- CN: 声明或扩展类型，包括 `TritonExperimentConfig`, `ExperimentGroupConfig`, `ExperimentResults`。
- EN: Implements callable logic such as `to_options`, `name`, `asdict`.
- CN: 实现可调用逻辑，例如 `to_options`, `name`, `asdict`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 181-240
````python
181: class ExperimentGroup:
182:     config: ExperimentGroupConfig
183:     results: list[ExperimentResults] = field(default_factory=list)
184: 
185: 
186: def get_inputs(
187:     config: ExperimentGroupConfig,
188: ) -> tuple[torch.Tensor, ...]:
189:     op_name = config.op_name
190:     M, N, K = config.shape
191:     batch_size = config.batch_size
192:     dtype = config.dtype
193:     device = torch.device("cuda")
194: 
195:     if op_name == "mm":
196:         A = torch.randn(M, K, dtype=dtype, device=device)
197:         B = torch.randn(N, K, dtype=dtype, device=device).t()
198:         return A, B
199:     elif op_name == "addmm":
200:         A = torch.randn(M, K, dtype=dtype, device=device)
201:         B = torch.randn(N, K, dtype=dtype, device=device).t()
202:         C = torch.randn(N, dtype=dtype, device=device)
203:         return C, A, B
204:     elif op_name == "bmm":
205:         A = torch.randn(batch_size, M, K, dtype=dtype, device=device)
206:         B = torch.randn(batch_size, N, K, dtype=dtype, device=device).permute(0, 2, 1)
207:         return A, B
208:     elif op_name == "_scaled_mm":
209:         # For _scaled_mm, we only support fp8e4m3 with rowwise scaling
210:         if dtype != torch.float8_e4m3fn:
211:             raise ValueError(f"_scaled_mm only supports fp8e4m3, got {dtype}")
212: 
213:         # Create input tensors in bfloat16 first, then quantize to fp8
214:         input_dtype = torch.bfloat16
215:         x = torch.randn(M, K, dtype=input_dtype, device=device)
216:         w = torch.randn(N, K, dtype=input_dtype, device=device)
217: 
218:         # Quantize using rowwise scaling
219:         w_fp8, w_inverse_scale = _quantize_rowwise(w, dtype)
220:         w_t_fp8 = w_fp8.t()
221:         w_inverse_scale = w_inverse_scale.t()  # scale_b should be (1, N)
222: 
223:         x_fp8, x_inverse_scale = _quantize_rowwise(x, dtype)
224: 
225:         # Return inputs for _scaled_mm: (input, weight_t, scale_a, scale_b, bias, out, out_dtype, use_fast_accum)
226:         return (
227:             x_fp8,
228:             w_t_fp8,
229:             x_inverse_scale,
230:             w_inverse_scale,
231:             None,
232:             None,
233:             torch.bfloat16,
234:             USE_FAST_ACCUM,
235:         )
236:     else:
237:         raise ValueError(f"Unknown op {op_name}")
238: 
239: 
240: def run_single_experiment_group(
````
- EN: Declares or extends types including `ExperimentGroup`.
- CN: 声明或扩展类型，包括 `ExperimentGroup`。
- EN: Implements callable logic such as `get_inputs`, `run_single_experiment_group`.
- CN: 实现可调用逻辑，例如 `get_inputs`, `run_single_experiment_group`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-300
````python
241:     group_config: ExperimentGroupConfig,
242: ) -> list[ExperimentResults]:
243:     inputs = get_inputs(group_config)
244:     op = getattr(torch, group_config.op_name)
245: 
246:     results = []
247: 
248:     for config in group_config.experiments:
249:         torch._dynamo.reset()
250:         torch._inductor.utils.clear_caches()
251:         compiled_op = torch.compile(
252:             op,
253:             options=config.to_options(),
254:         )
255: 
256:         start_time = time.perf_counter()
257:         try:
258:             _ = compiled_op(*inputs)
259:         except Exception as e:
260:             import traceback
261: 
262:             log.warning(
263:                 f"Benchmark config {config.name()} failed: {e}, "  # noqa: G004
264:                 f"traceback: {traceback.format_exc()}"
265:             )
266:             results.append(
267:                 ExperimentResults(
268:                     name=config.name(),
269:                     forward_time=float("inf"),
270:                     teraflops=0.0,
271:                     compilation_time=float("inf"),
272:                 )
273:             )
274:             continue
275:         compilation_time = time.perf_counter() - start_time
276: 
277:         forward_time = benchmark_torch_function_in_microseconds(
278:             compiled_op,
279:             *inputs,
280:         )
281: 
282:         flops = calculate_flops(
283:             group_config.op_name,
284:             group_config.shape,
285:             group_config.batch_size,
286:         )
287:         teraflops = flops / (forward_time * 1e-6) / 1e12
288: 
289:         results.append(
290:             ExperimentResults(
291:                 name=config.name(),
292:                 forward_time=forward_time,
293:                 teraflops=teraflops,
294:                 compilation_time=compilation_time,
295:             )
296:         )
297: 
298:     return results
299: 
300: 
````
- EN: Handles module imports such as `traceback`.
- CN: 处理模块导入，例如 `traceback`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 301-360
````python
301: def generate_experiment_groups(
302:     op_names: list[str],
303:     shapes: list[tuple[int, int, int]],
304:     dtypes: list[torch.dtype],
305:     enable_persistent_tma_matmuls: list[bool],
306:     cutlass_instantiation_levels: list[str],
307:     batch_sizes: list[int],
308: ) -> list[ExperimentGroupConfig]:
309:     groups = []
310:     for (
311:         op_name,
312:         shape,
313:         dtype,
314:         batch_size,
315:     ) in itertools.product(op_names, shapes, dtypes, batch_sizes):
316:         group = ExperimentGroupConfig(
317:             op_name=op_name,
318:             shape=shape,
319:             dtype=dtype,
320:             batch_size=batch_size,
321:         )
322:         experiments = generate_experiment_configs(
323:             enable_persistent_tma_matmuls, cutlass_instantiation_levels
324:         )
325:         group.experiments.extend(experiments)
326:         groups.append(group)
327: 
328:     return groups
329: 
330: 
331: def generate_experiment_configs(
332:     enable_persistent_tma_matmuls: list[bool], cutlass_instantiation_levels: list[str]
333: ) -> list[ExperimentConfig]:
334:     configs = []
335: 
336:     # add aten configs
337:     configs.append(
338:         AtenExperimentConfig(
339:             max_autotune_gemm_backends="ATEN",
340:         )
341:     )
342: 
343:     # add triton configs
344:     for enable_persistent_tma_matmul in enable_persistent_tma_matmuls:
345:         configs.append(
346:             TritonExperimentConfig(
347:                 max_autotune_gemm_backends="TRITON",
348:                 enable_persistent_tma_matmul=enable_persistent_tma_matmul,
349:             )
350:         )
351: 
352:     # add cutlass configs
353:     for cutlass_instantiation_level in cutlass_instantiation_levels:
354:         configs.append(
355:             CutlassExperimentConfig(
356:                 max_autotune_gemm_backends="CUTLASS",
357:                 cutlass_instantiation_level=cutlass_instantiation_level,
358:             )
359:         )
360: 
````
- EN: Implements callable logic such as `generate_experiment_groups`, `generate_experiment_configs`.
- CN: 实现可调用逻辑，例如 `generate_experiment_groups`, `generate_experiment_configs`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 361-420
````python
361:     return configs
362: 
363: 
364: def calculate_table_data(results: list[ExperimentResults]) -> dict:
365:     table_data = defaultdict(list)
366:     aten_perf: float | None = None
367: 
368:     for experiment_result in results:
369:         for key, value in experiment_result.asdict().items():
370:             if key not in UNITS:
371:                 raise AssertionError(
372:                     f"Unknown key '{key}'. Expected one of: {list(UNITS.keys())}"
373:                 )
374:             table_data[key + UNITS[key]].append(value)
375: 
376:         if experiment_result.name == "aten":
377:             aten_perf = experiment_result.forward_time
378:             table_data[PERF_OVER_ATEN_STR].append("NA")
379:         elif aten_perf is not None:
380:             perf_over_aten = (
381:                 (experiment_result.forward_time - aten_perf) / aten_perf * 100
382:             )
383:             table_data[PERF_OVER_ATEN_STR].append(perf_over_aten)
384:         else:
385:             # fallback in case aten is not in experiment group
386:             table_data[PERF_OVER_ATEN_STR].append("NA")
387: 
388:     return table_data
389: 
390: 
391: def calculate_flops(op_name: str, shape: tuple[int, int, int], batch_size: int) -> int:
392:     """
393:     Calculate the number of floating point operations based on operation type and shape.
394:     """
395:     M, N, K = shape
396: 
397:     if op_name == "bmm":
398:         return 2 * batch_size * M * N * K
399:     elif op_name == "addmm":
400:         return 2 * M * N * K + M * N
401:     elif op_name == "_scaled_mm":
402:         return 2 * M * N * K
403:     else:
404:         return 2 * M * N * K
405: 
406: 
407: def get_printable_results(experiment_groups: list[ExperimentGroup]) -> list[str]:
408:     edge_over_aten = defaultdict(list)
409:     output = []
410: 
411:     for experiment_group in experiment_groups:
412:         group_config_name = experiment_group.config.name()
413:         output.append(f"\nExperiment group: {group_config_name}")
414: 
415:         table_data = calculate_table_data(experiment_group.results)
416:         for name, edge in zip(table_data["name"], table_data[PERF_OVER_ATEN_STR]):
417:             edge_over_aten[name].append(edge)
418:         output.append(
419:             tabulate(table_data, headers="keys", tablefmt="pretty", floatfmt=".3f")
420:         )
````
- EN: Implements callable logic such as `calculate_table_data`, `calculate_flops`, `get_printable_results`.
- CN: 实现可调用逻辑，例如 `calculate_table_data`, `calculate_flops`, `get_printable_results`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 421-470
````python
421: 
422:     if "aten" in edge_over_aten:
423:         output.append("\nAverage edge over aten (max(-edge, 0), higher is better):")
424:         for name in edge_over_aten:
425:             if name != "aten":
426:                 values = [
427:                     max(-v, 0.0)
428:                     for v in edge_over_aten[name]
429:                     if v != float("inf") and v != "NA"
430:                 ]
431:                 valid_count = len(values)
432:                 average_edge = sum(values) / valid_count if values else "No valid data"
433:                 output.append(
434:                     f"{name}: {average_edge} (from {valid_count} valid values)"
435:                 )
436:         output.append("\n")
437: 
438:     return "\n".join(output)
439: 
440: 
441: def main():
442:     seed = 123
443:     torch.manual_seed(seed)
444:     results = []
445:     log.info("Starting benchmarking...")
446:     configs = list(
447:         generate_experiment_groups(
448:             OP_NAMES,
449:             SHAPES,
450:             DTYPES,
451:             ENABLE_PERSISTENT_TMA_MATMULS,
452:             CUTLASS_INSTANTIATION_LEVELS,
453:             BATCH_SIZES,
454:         )
455:     )
456:     for i, group_config in enumerate(tqdm(configs)):
457:         group_results = run_single_experiment_group(group_config)
458:         results.append(
459:             ExperimentGroup(config=group_config, results=group_results),
460:         )
461:         sys.stderr.write(
462:             f"\nINTERMEDIATE results: {i + 1}/{len(configs)} \n"
463:             + get_printable_results(results)
464:         )
465:     print("\nFINAL results...")
466:     print(get_printable_results(results))
467: 
468: 
469: if __name__ == "__main__":
470:     main()
````
- EN: Implements callable logic such as `main`.
- CN: 实现可调用逻辑，例如 `main`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `benchmark_torch_function_in_microseconds` / 符号 `benchmark_torch_function_in_microseconds`
- Symbol `ExperimentConfig` / 符号 `ExperimentConfig`
- Symbol `name` / 符号 `name`
- Symbol `to_options` / 符号 `to_options`

## Dependencies / 依赖关系
- Python imports: `os`, `sys`, `itertools`, `logging`, `time`, `abc`, `collections`, `collections.abc`, `dataclasses`, `typing`
- Python 导入: `os`, `sys`, `itertools`, `logging`, `time`, `abc`, `collections`, `collections.abc`, `dataclasses`, `typing`
