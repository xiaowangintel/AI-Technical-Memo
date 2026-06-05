# samplers_benchmark.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/data/samplers_benchmark.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40
````python
 1: #!/usr/bin/env python3
 2: 
 3: import time
 4: from collections.abc import Iterable, Iterator
 5: 
 6: import numpy as np
 7: from tabulate import tabulate
 8: 
 9: from torch.utils.data import BatchSampler, Sampler, SequentialSampler
10: 
11: 
12: class NewBatchSampler(Sampler[list[int]]):
13:     """Alternative implementation of BatchSampler for benchmarking purposes."""
14: 
15:     def __init__(
16:         self,
17:         sampler: Sampler[int] | Iterable[int],
18:         batch_size: int,
19:         drop_last: bool,
20:     ) -> None:
21:         if (
22:             not isinstance(batch_size, int)
23:             or isinstance(batch_size, bool)
24:             or batch_size <= 0
25:         ):
26:             raise ValueError(
27:                 f"batch_size should be a positive integer value, but got batch_size={batch_size}"
28:             )
29:         if not isinstance(drop_last, bool):
30:             raise ValueError(
31:                 f"drop_last should be a boolean value, but got drop_last={drop_last}"
32:             )
33:         self.sampler = sampler
34:         self.batch_size = batch_size
35:         self.drop_last = drop_last
36: 
37:     def __iter__(self) -> Iterator[list[int]]:
38:         if self.drop_last:
39:             sampler_iter = iter(self.sampler)
40:             while True:
````
- EN: Handles module imports such as `time`, `collections.abc`, `numpy`, `tabulate`.
- CN: 处理模块导入，例如 `time`, `collections.abc`, `numpy`, `tabulate`。
- EN: Declares or extends types including `NewBatchSampler`.
- CN: 声明或扩展类型，包括 `NewBatchSampler`。
- EN: Implements callable logic such as `__init__`, `__iter__`.
- CN: 实现可调用逻辑，例如 `__init__`, `__iter__`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。

### Lines 41-80
````python
41:                 try:
42:                     batch = [next(sampler_iter) for _ in range(self.batch_size)]
43:                     yield batch
44:                 except StopIteration:
45:                     break
46:         else:
47:             batch = [0] * self.batch_size
48:             idx_in_batch = 0
49:             for idx in self.sampler:
50:                 batch[idx_in_batch] = idx
51:                 idx_in_batch += 1
52:                 if idx_in_batch == self.batch_size:
53:                     yield batch
54:                     idx_in_batch = 0
55:                     batch = [0] * self.batch_size
56:             if idx_in_batch > 0:
57:                 yield batch[:idx_in_batch]
58: 
59:     def __len__(self) -> int:
60:         # Can only be called if self.sampler has __len__ implemented
61:         if self.drop_last:
62:             return len(self.sampler) // self.batch_size  # type: ignore[arg-type]
63:         else:
64:             return (len(self.sampler) + self.batch_size - 1) // self.batch_size  # type: ignore[arg-type]
65: 
66: 
67: def main():
68:     """Run benchmark with specified parameters."""
69:     DATA_SIZE = 99999
70:     AVG_TIMES = 10
71:     BATCH_SIZES = [4, 8, 64, 640, 6400, 64000]
72:     DROP_LAST_OPTIONS = [True, False]
73: 
74:     results = []
75: 
76:     # Set up samplers here, ensure right args are passed in
77:     baselineSampler = BatchSampler
78:     testSampler = NewBatchSampler
79: 
80:     for batch_size in BATCH_SIZES:
````
- EN: Implements callable logic such as `__len__`, `main`.
- CN: 实现可调用逻辑，例如 `__len__`, `main`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 81-120
````python
 81:         for drop_last in DROP_LAST_OPTIONS:
 82:             print(f"Benchmarking with batch_size={batch_size}, drop_last={drop_last}")
 83: 
 84:             # Benchmark baselineSampler
 85:             original_times = []
 86:             for _ in range(AVG_TIMES):
 87:                 start = time.perf_counter()
 88:                 for _ in baselineSampler(
 89:                     sampler=SequentialSampler(range(DATA_SIZE)),
 90:                     batch_size=batch_size,
 91:                     drop_last=drop_last,
 92:                 ):
 93:                     pass
 94:                 end = time.perf_counter()
 95:                 original_times.append(end - start)
 96:                 time.sleep(0.1)
 97: 
 98:             original_avg = float(np.mean(original_times))
 99: 
100:             # Benchmark testSampler
101:             new_times = []
102:             for _ in range(AVG_TIMES):
103:                 start = time.perf_counter()
104:                 for _ in testSampler(
105:                     sampler=SequentialSampler(range(DATA_SIZE)),
106:                     batch_size=batch_size,
107:                     drop_last=drop_last,
108:                 ):
109:                     pass
110:                 end = time.perf_counter()
111:                 new_times.append(end - start)
112:                 time.sleep(0.1)  # Small delay to reduce system load
113: 
114:             new_avg = float(np.mean(new_times))
115: 
116:             # Calculate speedup
117:             if original_avg > 0 and new_avg > 0:
118:                 speedup = (original_avg - new_avg) / original_avg * 100
119:                 speedup_str = f"{speedup:.2f}%"
120:             else:
````
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 121-142
````python
121:                 speedup_str = "N/A"
122: 
123:             print(f"Speedup: {speedup_str}\n")
124: 
125:             results.append(
126:                 [
127:                     batch_size,
128:                     drop_last,
129:                     f"{original_avg:.4f}",
130:                     f"{new_avg:.4f}",
131:                     speedup_str,
132:                 ]
133:             )
134: 
135:     # Print results in a table
136:     headers = ["Batch Size", "Drop Last", "Original (s)", "New (s)", "Speedup"]
137:     print("\nBenchmark Results:")
138:     print(tabulate(results, headers=headers, tablefmt="grid"))
139: 
140: 
141: if __name__ == "__main__":
142:     main()
````
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `NewBatchSampler` / 符号 `NewBatchSampler`
- Symbol `__init__` / 符号 `__init__`
- Symbol `__iter__` / 符号 `__iter__`
- Symbol `__len__` / 符号 `__len__`

## Dependencies / 依赖关系
- Python imports: `time`, `collections.abc`, `numpy`, `tabulate`, `torch.utils.data`
- Python 导入: `time`, `collections.abc`, `numpy`, `tabulate`, `torch.utils.data`
