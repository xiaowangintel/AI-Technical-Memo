# benchmark.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/distributed/ddp/benchmark.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60
````python
 1: #!/usr/bin/env python3
 2: #
 3: # Measure distributed training iteration time.
 4: #
 5: # This program performs a sweep over a) a number of model architectures, and
 6: # b) an increasing number of processes. This produces a 1-GPU baseline,
 7: # an 8-GPU baseline (if applicable), as well as measurements for however
 8: # many processes can participate in training.
 9: #
10: 
11: import argparse
12: import itertools
13: import json
14: import os
15: import shlex
16: import subprocess
17: import sys
18: import time
19: 
20: import numpy as np
21: import torchvision
22: 
23: import torch
24: import torch.distributed as dist
25: import torch.nn as nn
26: import torch.optim as optim
27: 
28: 
29: def allgather_object(obj):
30:     out = [None for _ in range(dist.get_world_size())]
31:     dist.all_gather_object(out, obj)
32:     return out
33: 
34: 
35: def allgather_run(cmd):
36:     proc = subprocess.run(shlex.split(cmd), capture_output=True)
37:     if proc.returncode != 0:
38:         raise AssertionError(
39:             f"Command '{cmd}' failed with return code {proc.returncode}: {proc.stderr.decode('utf-8')}"
40:         )
41:     return allgather_object(proc.stdout.decode("utf-8"))
42: 
43: 
44: def allequal(iterator):
45:     iterator = iter(iterator)
46:     try:
47:         first = next(iterator)
48:     except StopIteration:
49:         return True
50:     return all(first == rest for rest in iterator)
51: 
52: 
53: def benchmark_process_group(pg, benchmark, use_ddp_for_single_rank=True):
54:     torch.manual_seed(pg.rank())
55:     torch.cuda.manual_seed(pg.rank())
56: 
57:     model = benchmark.create_model()
58:     data = [(benchmark.generate_inputs(), benchmark.generate_target())]
59:     criterion = nn.CrossEntropyLoss()
60:     optimizer = optim.SGD(model.parameters(), 0.001, momentum=0.9, weight_decay=1e-4)
````
- EN: Handles module imports such as `argparse`, `itertools`, `json`, `os`.
- CN: 处理模块导入，例如 `argparse`, `itertools`, `json`, `os`。
- EN: Implements callable logic such as `allgather_object`, `allgather_run`, `allequal`, `benchmark_process_group`.
- CN: 实现可调用逻辑，例如 `allgather_object`, `allgather_run`, `allequal`, `benchmark_process_group`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 61-120
````python
 61:     if use_ddp_for_single_rank or pg.size() > 1:
 62:         model = torch.nn.parallel.DistributedDataParallel(
 63:             model,
 64:             device_ids=[torch.cuda.current_device()],
 65:             broadcast_buffers=False,
 66:             process_group=pg,
 67:             bucket_cap_mb=benchmark.bucket_size,
 68:         )
 69: 
 70:     measurements = []
 71:     warmup_iterations = 5
 72:     measured_iterations = 10
 73:     for inputs, target in data * (warmup_iterations + measured_iterations):
 74:         start = time.time()
 75:         output = model(*inputs)
 76:         loss = criterion(output, target)
 77:         loss.backward()
 78:         optimizer.step()
 79:         torch.cuda.synchronize()
 80:         measurements.append(time.time() - start)
 81: 
 82:     # Throw away measurements for warmup iterations
 83:     return measurements[warmup_iterations:]
 84: 
 85: 
 86: def run_benchmark(benchmark, ranks, opts):
 87:     group = dist.new_group(ranks=ranks, backend=benchmark.distributed_backend)
 88:     measurements = []
 89:     if dist.get_rank() in set(ranks):
 90:         if not opts:
 91:             opts = {}
 92:         measurements = benchmark_process_group(group, benchmark, **opts)
 93:     dist.destroy_process_group(group)
 94:     dist.barrier()
 95: 
 96:     # Aggregate measurements for better estimation of percentiles
 97:     return list(itertools.chain(*allgather_object(measurements)))
 98: 
 99: 
100: def sweep(benchmark):
101:     # Synthesize the set of benchmarks to run.
102:     # This list contain tuples for ("string prefix", [rank...]).
103:     benchmarks = []
104: 
105:     def append_benchmark(prefix, ranks, opts=None):
106:         prefix = f"{len(ranks):4} GPUs -- {prefix}"
107:         benchmarks.append((prefix, ranks, opts))
108: 
109:     def local_print(msg):
110:         if dist.get_rank() == 0:
111:             print(msg, end="", flush=True)
112: 
113:     def print_header():
114:         local_print("\n")
115:         local_print(" " * 22)
116:         for _ in [50, 75, 90, 95]:
117:             local_print(f"{'sec/iter':14s}{'ex/sec':10s}")
118:         local_print("\n")
119: 
120:     def print_measurements(prefix, nelem, measurements):
````
- EN: Implements callable logic such as `run_benchmark`, `sweep`, `append_benchmark`, `local_print`.
- CN: 实现可调用逻辑，例如 `run_benchmark`, `sweep`, `append_benchmark`, `local_print`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-180
````python
121:         measurements = sorted(measurements)
122:         local_print(f"{prefix:8s}:")
123:         for p in [50, 75, 90, 95]:
124:             v = np.percentile(measurements, p)
125:             local_print(f"  p{p:02d}:  {v:1.3f}s  {nelem / v:6d}/s")
126:         local_print("\n")
127: 
128:     # Every process runs once by themselves to warm up (CUDA init, etc).
129:     append_benchmark("  warmup", [dist.get_rank()], {"use_ddp_for_single_rank": False})
130: 
131:     # Single machine baselines
132:     append_benchmark("  no ddp", range(1), {"use_ddp_for_single_rank": False})
133:     append_benchmark("   1M/1G", range(1))
134:     append_benchmark("   1M/2G", range(2))
135:     append_benchmark("   1M/4G", range(4))
136: 
137:     # Multi-machine benchmarks
138:     for i in range(1, (dist.get_world_size() // 8) + 1):
139:         append_benchmark(f"   {i:d}M/8G", range(i * 8))
140: 
141:     # Run benchmarks in order of increasing number of GPUs
142:     print_header()
143:     results = []
144:     for prefix, ranks, opts in sorted(benchmarks, key=lambda tup: len(tup[1])):
145:         # Turn range into materialized list.
146:         ranks = list(ranks)
147:         measurements = run_benchmark(benchmark, ranks, opts)
148:         if "warmup" not in prefix:
149:             print_measurements(prefix, benchmark.batch_size, measurements)
150:             results.append({"ranks": ranks, "measurements": measurements})
151: 
152:     return results
153: 
154: 
155: class Benchmark:
156:     def __init__(self, device, distributed_backend, bucket_size):
157:         self.device = device
158:         self.batch_size = 32
159:         self.distributed_backend = distributed_backend
160:         self.bucket_size = bucket_size
161: 
162:     def __str__(self):
163:         raise NotImplementedError
164: 
165:     def create_model(self):
166:         raise NotImplementedError
167: 
168:     def generate_inputs(self):
169:         raise NotImplementedError
170: 
171:     def generate_target(self):
172:         raise NotImplementedError
173: 
174: 
175: class TorchvisionBenchmark(Benchmark):
176:     def __init__(self, device, distributed_backend, bucket_size, model):
177:         super().__init__(
178:             device,
179:             distributed_backend,
180:             bucket_size,
````
- EN: Declares or extends types including `Benchmark`, `TorchvisionBenchmark`.
- CN: 声明或扩展类型，包括 `Benchmark`, `TorchvisionBenchmark`。
- EN: Implements callable logic such as `__init__`, `__str__`, `create_model`, `generate_inputs`.
- CN: 实现可调用逻辑，例如 `__init__`, `__str__`, `create_model`, `generate_inputs`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 181-240
````python
181:         )
182:         self.model = model
183: 
184:     def __str__(self):
185:         return f"{self.model} with batch size {self.batch_size}"
186: 
187:     def create_model(self):
188:         return torchvision.models.__dict__[self.model]().to(self.device)
189: 
190:     def generate_inputs(self):
191:         return [torch.rand([self.batch_size, 3, 224, 224], device=self.device)]
192: 
193:     def generate_target(self):
194:         return torch.tensor([1] * self.batch_size, dtype=torch.long, device=self.device)
195: 
196: 
197: def main():
198:     parser = argparse.ArgumentParser(description="PyTorch distributed benchmark suite")
199:     parser.add_argument("--rank", type=int, default=os.environ["RANK"])
200:     parser.add_argument("--world-size", type=int, required=True)
201:     parser.add_argument("--distributed-backend", type=str, default="nccl")
202:     parser.add_argument("--bucket-size", type=int, default=25)
203:     parser.add_argument("--master-addr", type=str, required=True)
204:     parser.add_argument("--master-port", type=str, required=True)
205:     parser.add_argument("--model", type=str)
206:     parser.add_argument(
207:         "--json", type=str, metavar="PATH", help="Write file with benchmark results"
208:     )
209:     args = parser.parse_args()
210: 
211:     num_gpus_per_node = torch.cuda.device_count()
212:     if num_gpus_per_node != 8:
213:         raise AssertionError(
214:             f"Expected 8 GPUs per machine, but found {num_gpus_per_node}"
215:         )
216: 
217:     # The global process group used only for communicating benchmark
218:     # metadata, like measurements. Not for benchmarking itself.
219:     dist.init_process_group(
220:         backend="gloo",
221:         init_method=f"tcp://{args.master_addr}:{args.master_port}",
222:         rank=args.rank,
223:         world_size=args.world_size,
224:     )
225: 
226:     output = allgather_run("nvidia-smi topo -m")
227:     if not allequal(output):
228:         print('Output of "nvidia-smi topo -m" differs between machines')
229:         sys.exit(1)
230: 
231:     if args.rank == 0:
232:         print("-----------------------------------")
233:         print("PyTorch distributed benchmark suite")
234:         print("-----------------------------------")
235:         print()
236:         print(f"* PyTorch version: {torch.__version__}")
237:         print(f"* CUDA version: {torch.version.cuda}")
238:         print(f"* Distributed backend: {args.distributed_backend}")
239:         print(f"* Maximum bucket size: {args.bucket_size}MB")
240:         print()
````
- EN: Implements callable logic such as `__str__`, `create_model`, `generate_inputs`, `generate_target`.
- CN: 实现可调用逻辑，例如 `__str__`, `create_model`, `generate_inputs`, `generate_target`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-298
````python
241:         print("--- nvidia-smi topo -m ---")
242:         print()
243:         print(output[0])
244:         print("--------------------------")
245:         print()
246: 
247:     torch.cuda.set_device(dist.get_rank() % 8)
248:     device = torch.device(f"cuda:{dist.get_rank() % 8:d}")
249: 
250:     benchmarks = []
251:     if args.model:
252:         benchmarks.append(
253:             TorchvisionBenchmark(
254:                 device=device,
255:                 distributed_backend=args.distributed_backend,
256:                 bucket_size=args.bucket_size,
257:                 model=args.model,
258:             )
259:         )
260:     else:
261:         for model in ["resnet50", "resnet101", "resnext50_32x4d", "resnext101_32x8d"]:
262:             benchmarks.append(
263:                 TorchvisionBenchmark(
264:                     device=device,
265:                     distributed_backend=args.distributed_backend,
266:                     bucket_size=args.bucket_size,
267:                     model=model,
268:                 )
269:             )
270: 
271:     benchmark_results = []
272:     for benchmark in benchmarks:
273:         if args.rank == 0:
274:             print(f"\nBenchmark: {str(benchmark)}")
275:         result = sweep(benchmark)
276:         benchmark_results.append(
277:             {
278:                 "model": benchmark.model,
279:                 "batch_size": benchmark.batch_size,
280:                 "result": result,
281:             }
282:         )
283: 
284:     # Write file with benchmark results if applicable
285:     if args.rank == 0 and args.json:
286:         report = {
287:             "pytorch_version": torch.__version__,
288:             "cuda_version": torch.version.cuda,
289:             "distributed_backend": args.distributed_backend,
290:             "bucket_size": args.bucket_size,
291:             "benchmark_results": benchmark_results,
292:         }
293:         with open(args.json, "w") as f:
294:             json.dump(report, f)
295: 
296: 
297: if __name__ == "__main__":
298:     main()
````
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `allgather_object` / 符号 `allgather_object`
- Symbol `allgather_run` / 符号 `allgather_run`
- Symbol `allequal` / 符号 `allequal`
- Symbol `benchmark_process_group` / 符号 `benchmark_process_group`

## Dependencies / 依赖关系
- Python imports: `argparse`, `itertools`, `json`, `os`, `shlex`, `subprocess`, `sys`, `time`, `numpy`, `torchvision`
- Python 导入: `argparse`, `itertools`, `json`, `os`, `shlex`, `subprocess`, `sys`, `time`, `numpy`, `torchvision`
