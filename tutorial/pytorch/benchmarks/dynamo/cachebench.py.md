# cachebench.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/dynamo/cachebench.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60
````python
 1: import argparse
 2: import dataclasses
 3: import json
 4: import logging
 5: import os
 6: import subprocess
 7: import sys
 8: import tempfile
 9: from collections.abc import Callable
10: 
11: from torch._inductor.utils import fresh_cache
12: 
13: 
14: logger: logging.Logger = logging.getLogger(__name__)
15: 
16: TIMEOUT: int = 2000
17: 
18: 
19: # Keep in sync with .ci/pytorch/test.sh
20: TORCHBENCH_MODELS: list[str] = [
21:     "nanogpt",
22:     "BERT_pytorch",
23:     "resnet50",
24:     "moco",
25:     "llama",
26: ]
27: HUGGINGFACE_MODELS: list[str] = [
28:     "AllenaiLongformerBase",
29:     "BertForMaskedLM",
30:     "GPT2ForSequenceClassification",
31: ]
32: 
33: 
34: @dataclasses.dataclass
35: class RunResult:
36:     model: str
37:     mode: str  # inference or training
38:     benchmark: str
39:     dynamic: bool
40:     device: str  # cuda or cpu
41:     cold_compile_s: list[float]
42:     warm_compile_s: list[float]
43:     speedup_pct: float
44: 
45: 
46: def get_compile_time(file: tempfile._TemporaryFileWrapper) -> float:
47:     lines = file.readlines()
48:     # Decode from byte string, remove new lines, parse csv
49:     lines = [line.decode("utf-8").strip().split(",") for line in lines]
50:     compilation_time_idx = lines[0].index("compilation_latency")
51:     compilation_time = lines[1][compilation_time_idx]
52:     return float(compilation_time)
53: 
54: 
55: def _run_torchbench_from_args(
56:     cmd_args: argparse.Namespace,
57:     model: str,
58:     args: list[str],
59: ) -> tuple[list[float], list[float]]:
60:     cold_compile_time: list[float] = []
````
- EN: Handles module imports such as `argparse`, `dataclasses`, `json`, `logging`.
- CN: 处理模块导入，例如 `argparse`, `dataclasses`, `json`, `logging`。
- EN: Declares or extends types including `RunResult`.
- CN: 声明或扩展类型，包括 `RunResult`。
- EN: Implements callable logic such as `get_compile_time`, `_run_torchbench_from_args`.
- CN: 实现可调用逻辑，例如 `get_compile_time`, `_run_torchbench_from_args`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。

### Lines 61-120
````python
 61:     warm_compile_time: list[float] = []
 62: 
 63:     for _ in range(cmd_args.repeat):
 64:         with fresh_cache():
 65:             env = os.environ.copy()
 66:             with tempfile.NamedTemporaryFile(suffix=".csv") as file:
 67:                 args.append("--output=" + file.name)
 68:                 logger.info(f"Performing cold-start run for {model}")  # noqa: G004
 69:                 subprocess.check_call(args, timeout=TIMEOUT, env=env)
 70:                 cold_compile_time.append(get_compile_time(file))
 71: 
 72:             args.pop()
 73:             with tempfile.NamedTemporaryFile(suffix=".csv") as file:
 74:                 args.append("--output=" + file.name)
 75:                 logger.info(f"Performing warm-start run for {model}")  # noqa: G004
 76:                 subprocess.check_call(args, timeout=TIMEOUT, env=env)
 77:                 warm_compile_time.append(get_compile_time(file))
 78: 
 79:     return cold_compile_time, warm_compile_time
 80: 
 81: 
 82: MODE_ARGS_DICT = {
 83:     "inference": ["--inference", "--bfloat16"],
 84:     "training": ["--training", "--amp"],
 85: }
 86: 
 87: 
 88: BENCHMARK_FILE = {
 89:     "torchbench": "torchbench.py",
 90:     "huggingface": "huggingface.py",
 91: }
 92: 
 93: 
 94: def _run_torchbench_model(
 95:     cmd_args: argparse.Namespace,
 96:     results: list[RunResult],
 97:     model: str,
 98: ) -> None:
 99:     cur_file = os.path.abspath(__file__)
100:     torchbench_file = os.path.join(
101:         os.path.dirname(cur_file), BENCHMARK_FILE[cmd_args.benchmark]
102:     )
103:     if not os.path.exists(torchbench_file):
104:         raise AssertionError(f"Torchbench does not exist at {torchbench_file}")
105: 
106:     dynamic = cmd_args.dynamic
107:     dynamic_args = ["--dynamic-shapes", "--dynamic-batch-only"] if dynamic else []
108: 
109:     args = (
110:         [
111:             sys.executable,
112:             torchbench_file,
113:             f"--only={model}",
114:             "--repeat=1",
115:             "--performance",
116:             "--backend=inductor",
117:             f"--device={cmd_args.device}",
118:         ]
119:         + MODE_ARGS_DICT[cmd_args.mode]
120:         + dynamic_args
````
- EN: Implements callable logic such as `_run_torchbench_model`.
- CN: 实现可调用逻辑，例如 `_run_torchbench_model`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-180
````python
121:     )
122: 
123:     logger.info(f"Command: {args}")  # noqa: G004
124:     try:
125:         cold_compile_t, warm_compile_t = _run_torchbench_from_args(
126:             cmd_args, model, args
127:         )
128:         speedup_pct = (1 - (sum(warm_compile_t) / sum(cold_compile_t))) * 100
129:         results.append(
130:             RunResult(
131:                 model=model,
132:                 mode=cmd_args.mode,
133:                 benchmark=cmd_args.benchmark,
134:                 dynamic=dynamic,
135:                 device=cmd_args.device,
136:                 cold_compile_s=cold_compile_t,
137:                 warm_compile_s=warm_compile_t,
138:                 speedup_pct=speedup_pct,
139:             )
140:         )
141:     except Exception:
142:         logger.info("fail", exc_info=True)
143:         return None
144: 
145: 
146: def _write_results_to_json(
147:     cmd_args: argparse.Namespace,
148:     results: list[RunResult],
149: ) -> None:
150:     if len(results) == 0:
151:         # do not write empty results
152:         return
153: 
154:     records = []
155:     for result in results:
156:         for metric_name, value in [
157:             ("Cold compile time (s)", result.cold_compile_s),
158:             ("Warm compile time (s)", result.warm_compile_s),
159:             ("Speedup (%)", [result.speedup_pct]),
160:         ]:
161:             records.append(
162:                 {
163:                     "benchmark": {
164:                         "name": "TorchCache Benchmark",
165:                         "mode": result.mode,
166:                         "extra_info": {
167:                             "is_dynamic": result.dynamic,
168:                             "device": result.device,
169:                         },
170:                     },
171:                     "model": {
172:                         "name": result.model,
173:                         "backend": "inductor",
174:                         "origins": [result.benchmark],
175:                     },
176:                     "metric": {
177:                         "name": metric_name,
178:                         "type": "OSS model",
179:                         "benchmark_values": value,
180:                     },
````
- EN: Implements callable logic such as `_write_results_to_json`.
- CN: 实现可调用逻辑，例如 `_write_results_to_json`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 181-240
````python
181:                 }
182:             )
183:     with open(cmd_args.output, "w") as f:
184:         json.dump(records, f)
185: 
186: 
187: def parse_cmd_args() -> argparse.Namespace:
188:     parser = argparse.ArgumentParser(description="Run a TorchCache benchmark.")
189:     parser.add_argument(
190:         "-m",
191:         "--model",
192:         help="Name of the model to run",
193:     )
194:     parser.add_argument(
195:         "--dynamic",
196:         action="store_true",
197:         help="Whether to run with dynamic enabled",
198:     )
199:     parser.add_argument(
200:         "--benchmark",
201:         choices=("torchbench", "huggingface"),
202:         required=True,
203:         help="Name of benchmark suite to run",
204:     )
205:     parser.add_argument(
206:         "--mode",
207:         choices=("inference", "training"),
208:         default="training",
209:     )
210:     parser.add_argument(
211:         "--device",
212:         default="cuda",
213:         choices=("cuda", "cpu"),
214:     )
215:     parser.add_argument(
216:         "--output",
217:         required=True,
218:         help="The output filename (json)",
219:     )
220:     parser.add_argument(
221:         "--repeat",
222:         type=int,
223:         default=1,
224:         choices=range(1, 10),
225:         help="Number of times to repeat the compilation (reduce noise)",
226:     )
227:     args, _ = parser.parse_known_args()
228:     return args
229: 
230: 
231: Dispatch_fn_t = Callable[[argparse.Namespace, list[RunResult], str], None]
232: 
233: 
234: def main() -> None:
235:     cmd_args = parse_cmd_args()
236: 
237:     dispatcher: dict[str, tuple[Dispatch_fn_t, list[str]]] = {
238:         "torchbench": (_run_torchbench_model, TORCHBENCH_MODELS),
239:         "huggingface": (_run_torchbench_model, HUGGINGFACE_MODELS),
240:     }
````
- EN: Implements callable logic such as `parse_cmd_args`, `main`.
- CN: 实现可调用逻辑，例如 `parse_cmd_args`, `main`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-253
````python
241:     fn, models = dispatcher[cmd_args.benchmark]
242:     if cmd_args.model is not None:
243:         models = [cmd_args.model]
244: 
245:     results: list[RunResult] = []
246:     for model in models:
247:         fn(cmd_args, results, model)
248: 
249:     _write_results_to_json(cmd_args, results)
250: 
251: 
252: if __name__ == "__main__":
253:     main()
````
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `RunResult` / 符号 `RunResult`
- Symbol `get_compile_time` / 符号 `get_compile_time`
- Symbol `_run_torchbench_from_args` / 符号 `_run_torchbench_from_args`
- Symbol `_run_torchbench_model` / 符号 `_run_torchbench_model`

## Dependencies / 依赖关系
- Python imports: `argparse`, `dataclasses`, `json`, `logging`, `os`, `subprocess`, `sys`, `tempfile`, `collections.abc`, `torch._inductor.utils`
- Python 导入: `argparse`, `dataclasses`, `json`, `logging`, `os`, `subprocess`, `sys`, `tempfile`, `collections.abc`, `torch._inductor.utils`
