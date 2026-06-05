# bench_nvshmem_tile_reduce.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/distributed/bench_nvshmem_tile_reduce.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40
````python
 1: #!/usr/bin/env python3
 2: """
 3: Benchmark for NVSHMEM tile reduce operations.
 4: 
 5: Usage:
 6: python benchmarks/distributed/bench_nvshmem_tile_reduce.py
 7: 
 8: This benchmark measures the performance of tile reduce operations across different
 9: matrix sizes and tile configurations.
10: """
11: 
12: import time
13: 
14: import torch
15: import torch.distributed as dist
16: import torch.distributed._symmetric_memory as symm_mem
17: from torch.testing._internal.common_distributed import MultiProcContinuousTest
18: from torch.testing._internal.common_utils import (
19:     requires_cuda_p2p_access,
20:     skip_but_pass_in_sandcastle_if,
21:     skipIfRocm,
22: )
23: 
24: 
25: # Decorator
26: def requires_nvshmem():
27:     return skip_but_pass_in_sandcastle_if(
28:         not symm_mem.is_nvshmem_available(),
29:         "bench_nvshmem_tile_reduce requires NVSHMEM, skipping benchmark",
30:     )
31: 
32: 
33: # So that benchmarks are written in device-agnostic way
34: device_type = "cuda"
35: device_module = torch.get_device_module(device_type)
36: 
37: 
38: @requires_nvshmem()
39: @requires_cuda_p2p_access()
40: class NVSHMEMTileReduceBenchmark(MultiProcContinuousTest):
````
- EN: Handles module imports such as `time`, `torch`, `torch.distributed`, `torch.distributed._symmetric_memory`.
- CN: 处理模块导入，例如 `time`, `torch`, `torch.distributed`, `torch.distributed._symmetric_memory`。
- EN: Declares or extends types including `NVSHMEMTileReduceBenchmark`.
- CN: 声明或扩展类型，包括 `NVSHMEMTileReduceBenchmark`。
- EN: Implements callable logic such as `requires_nvshmem`.
- CN: 实现可调用逻辑，例如 `requires_nvshmem`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。

### Lines 41-80
````python
41:     def _init_device(self) -> None:
42:         # TODO: relieve this (seems to hang if without)
43:         device_module.set_device(self.device)
44:         # Set NVSHMEM as SymmMem backend
45:         symm_mem.set_backend("NVSHMEM")
46: 
47:     @property
48:     def device(self) -> torch.device:
49:         return torch.device(device_type, self.rank)
50: 
51:     def _benchmark_tile_reduce_single(
52:         self,
53:         full_size: int,
54:         tile_size: int,
55:         warmup_iters: int = 5,
56:         bench_iters: int = 10,
57:     ) -> dict:
58:         """
59:         Benchmark a single configuration of tile reduce.
60: 
61:         Args:
62:             full_size: Size of the full matrix (full_size x full_size)
63:             warmup_iters: Number of warmup iterations
64:             bench_iters: Number of benchmark iterations
65: 
66:         Returns:
67:             Dictionary with benchmark results
68:         """
69:         self._init_device()
70:         group_name = dist.group.WORLD.group_name
71: 
72:         dtype = torch.float
73: 
74:         # Allocate full matrices
75:         full_inp = symm_mem.empty(
76:             full_size, full_size, dtype=dtype, device=self.device
77:         ).fill_(self.rank)
78:         full_out = symm_mem.empty(
79:             full_size, full_size, dtype=dtype, device=self.device
80:         ).fill_(0)
````
- EN: Implements callable logic such as `_init_device`, `device`, `_benchmark_tile_reduce_single`.
- CN: 实现可调用逻辑，例如 `_init_device`, `device`, `_benchmark_tile_reduce_single`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 81-120
````python
 81: 
 82:         slice_ut = slice(0, tile_size)
 83:         inp_tile = full_inp[slice_ut, slice_ut]
 84:         out_tile = full_out[slice_ut, slice_ut]
 85: 
 86:         root = 0
 87: 
 88:         # Warmup iterations
 89:         for _ in range(warmup_iters):
 90:             torch.ops.symm_mem.tile_reduce(inp_tile, out_tile, root, group_name)
 91:             torch.cuda.synchronize(self.device)
 92: 
 93:         # Benchmark iterations
 94:         times = []
 95: 
 96:         dist.barrier()
 97:         torch.cuda.synchronize(self.device)
 98:         start_time = time.perf_counter()
 99: 
100:         for _ in range(bench_iters):
101:             torch.ops.symm_mem.tile_reduce(inp_tile, out_tile, root, group_name)
102: 
103:         torch.cuda.synchronize(self.device)
104:         end_time = time.perf_counter()
105:         times.append((end_time - start_time) / bench_iters)
106: 
107:         # Calculate statistics
108:         times = torch.tensor(times, dtype=torch.float64)
109:         tile_elements = tile_size * tile_size
110:         tile_bytes = (
111:             tile_elements * dtype.itemsize
112:             if hasattr(dtype, "itemsize")
113:             else tile_elements * 4
114:         )
115: 
116:         results = {
117:             "full_size": full_size,
118:             "tile_size": tile_size,
119:             "tile_elements": tile_elements,
120:             "tile_bytes": tile_bytes,
````
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 121-160
````python
121:             "world_size": self.world_size,
122:             "mean_time_ms": times.mean().item() * 1000,
123:             "std_time_ms": times.std().item() * 1000,
124:             "min_time_ms": times.min().item() * 1000,
125:             "max_time_ms": times.max().item() * 1000,
126:             "throughput_gb_s": tile_bytes / (times.mean().item() * 1e9),
127:             "elements_per_sec": tile_elements / times.mean().item(),
128:         }
129: 
130:         return results
131: 
132:     @skipIfRocm
133:     def test_benchmark_tile_reduce_various_sizes(self) -> None:
134:         """
135:         Benchmark tile reduce across various matrix sizes.
136:         """
137:         # Test various matrix sizes
138:         tile_sizes = [512, 1024, 2048, 4096, 8192, 16384]
139:         full_size = tile_sizes[-1]
140:         warmup_iters = 5
141:         bench_iters = 20
142: 
143:         results = []
144: 
145:         for tile_size in tile_sizes:
146:             try:
147:                 result = self._benchmark_tile_reduce_single(
148:                     full_size, tile_size, warmup_iters, bench_iters
149:                 )
150:                 results.append(result)
151: 
152:                 if self.rank == 0:
153:                     print(
154:                         f"Matrix Size: {full_size}x{full_size}, Tile Size: {tile_size}x{tile_size}"
155:                     )
156:                     print(
157:                         f"  Mean Time: {result['mean_time_ms']:.3f} ± {result['std_time_ms']:.3f} ms"
158:                     )
159:                     print(f"  Throughput: {result['throughput_gb_s']:.2f} GB/s")
160:                     print(f"  Bytes: {result['tile_bytes']:.0f}")
````
- EN: Implements callable logic such as `test_benchmark_tile_reduce_various_sizes`.
- CN: 实现可调用逻辑，例如 `test_benchmark_tile_reduce_various_sizes`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 161-190
````python
161:                     print()
162: 
163:             except Exception as e:
164:                 if self.rank == 0:
165:                     print(f"Failed to benchmark matrix size {full_size}: {e}")
166: 
167:         # Print summary
168:         if self.rank == 0 and results:
169:             print("=== BENCHMARK SUMMARY ===")
170:             print(
171:                 f"{'Matrix Size':<12} {'Tile Size':<10} {'Time (ms)':<12} {'Throughput (GB/s)':<18} {'Bytes':<15}"
172:             )
173:             print("-" * 70)
174: 
175:             for result in results:
176:                 print(
177:                     f"{result['full_size']}x{result['full_size']:<7} "
178:                     f"{result['tile_size']}x{result['tile_size']:<5} "
179:                     f"{result['mean_time_ms']:<12.3f} "
180:                     f"{result['throughput_gb_s']:<18.2f} "
181:                     f"{result['tile_bytes']:<15.0f}"
182:                 )
183: 
184: 
185: if __name__ == "__main__":
186:     # For standalone usage, you'd need to set up distributed environment
187:     # For now, this is meant to be run via the PyTorch test framework
188:     from torch.testing._internal.common_utils import run_tests
189: 
190:     run_tests()
````
- EN: Handles module imports such as `torch.testing._internal.common_utils`.
- CN: 处理模块导入，例如 `torch.testing._internal.common_utils`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `requires_nvshmem` / 符号 `requires_nvshmem`
- Symbol `NVSHMEMTileReduceBenchmark` / 符号 `NVSHMEMTileReduceBenchmark`
- Symbol `_init_device` / 符号 `_init_device`
- Symbol `device` / 符号 `device`

## Dependencies / 依赖关系
- Python imports: `time`, `torch`, `torch.distributed`, `torch.distributed._symmetric_memory`, `torch.testing._internal.common_distributed`, `torch.testing._internal.common_utils`
- Python 导入: `time`, `torch`, `torch.distributed`, `torch.distributed._symmetric_memory`, `torch.testing._internal.common_distributed`, `torch.testing._internal.common_utils`
