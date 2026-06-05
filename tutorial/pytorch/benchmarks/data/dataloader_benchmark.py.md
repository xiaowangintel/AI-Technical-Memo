# dataloader_benchmark.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/data/dataloader_benchmark.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60
````python
 1: #!/usr/bin/env python3
 2: """
 3: Benchmark script for PyTorch DataLoader with different worker methods.
 4: 
 5: This script measures:
 6: 1. Dataloader initialization time
 7: 2. Dataloading speed (time per batch)
 8: 3. CPU memory utilization
 9: 
10: Usage:
11:     python dataloader_benchmark.py --data_path /path/to/dataset --batch_size 32 --num_workers 4
12: """
13: 
14: import argparse
15: import copy
16: import gc
17: import time
18: 
19: import psutil
20: import torchvision
21: import torchvision.transforms as transforms
22: from torchvision.models import resnet18
23: 
24: import torch
25: import torch.nn as nn
26: import torch.optim as optim
27: from torch.utils.data import DataLoader
28: from torch.utils.data.dataset import ConcatDataset
29: 
30: 
31: def get_memory_usage():
32:     """
33:     Get current memory usage in MB. This includes all child processes.
34: 
35:     Returns:
36:         Total memory usage in MB
37:     """
38:     process = psutil.Process()
39: 
40:     main_memory = process.memory_full_info().pss
41: 
42:     # Add memory usage of all child processes
43:     for child in process.children(recursive=True):
44:         try:
45:             child_mem = child.memory_full_info().pss
46:             main_memory += child_mem
47:         except (psutil.NoSuchProcess, psutil.AccessDenied, AttributeError):
48:             # Process might have terminated or doesn't support PSS, fall back to USS
49:             print(f"Failed to get PSS for {child}, falling back to USS")
50:             child_mem = child.memory_info().uss
51:             main_memory += child_mem
52: 
53:     return main_memory / (1024 * 1024)
54: 
55: 
56: def print_detailed_memory():
57:     """Print detailed memory information."""
58:     process = psutil.Process()
59:     print("\nDetailed memory information:")
60:     try:
````
- EN: Handles module imports such as `argparse`, `copy`, `gc`, `time`.
- CN: 处理模块导入，例如 `argparse`, `copy`, `gc`, `time`。
- EN: Implements callable logic such as `get_memory_usage`, `print_detailed_memory`.
- CN: 实现可调用逻辑，例如 `get_memory_usage`, `print_detailed_memory`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 61-120
````python
 61:         print(
 62:             f"  USS (Unique Set Size): {process.memory_full_info().uss / (1024 * 1024):.2f} MB"
 63:         )
 64:         print(
 65:             f"  PSS (Proportional Set Size): {process.memory_full_info().pss / (1024 * 1024):.2f} MB"
 66:         )
 67:         print(
 68:             f"  RSS (Resident Set Size): {process.memory_info().rss / (1024 * 1024):.2f} MB"
 69:         )
 70:     except Exception:
 71:         print("  Detailed memory info not available")
 72: 
 73: 
 74: def create_model():
 75:     """Create a simple model for benchmarking."""
 76:     model = resnet18()
 77:     return model
 78: 
 79: 
 80: def benchmark_dataloader(
 81:     dataset,
 82:     batch_size,
 83:     num_workers,
 84:     num_epochs=1,
 85:     max_batches=10,
 86:     multiprocessing_context=None,
 87:     logging_freq=10,
 88: ):
 89:     """Benchmark a dataloader with specific configuration."""
 90:     print("\n--- Benchmarking DataLoader ---")
 91: 
 92:     # Clear memory before starting
 93:     gc.collect()
 94:     torch.cuda.empty_cache()
 95: 
 96:     # Create model
 97:     model = create_model()
 98: 
 99:     # Measure memory before dataloader creation
100:     memory_before = get_memory_usage()
101:     print(f"Memory before DataLoader creation: {memory_before:.2f} MB")
102:     print_detailed_memory()
103: 
104:     # Measure dataloader initialization time
105:     start = time.perf_counter()
106:     dataloader = DataLoader(
107:         dataset,
108:         batch_size=batch_size,
109:         shuffle=True,
110:         num_workers=num_workers,
111:         pin_memory=torch.cuda.is_available(),
112:         prefetch_factor=2 if num_workers > 0 else None,
113:         multiprocessing_context=multiprocessing_context,
114:     )
115:     it = iter(dataloader)
116:     dataloader_init_time = time.perf_counter() - start
117: 
118:     # Measure memory after dataloader creation
119:     memory_after = get_memory_usage()
120:     print(f"Memory after DataLoader creation: {memory_after:.2f} MB")
````
- EN: Implements callable logic such as `create_model`, `benchmark_dataloader`.
- CN: 实现可调用逻辑，例如 `create_model`, `benchmark_dataloader`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-180
````python
121:     print(f"Memory increase: {memory_after - memory_before:.2f} MB")
122: 
123:     # Create model and optimizer
124:     device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
125:     model = model.to(device)
126:     criterion = nn.CrossEntropyLoss()
127:     optimizer = optim.SGD(model.parameters(), lr=0.01, momentum=0.9)
128: 
129:     # Benchmark dataloading speed
130:     model.train()
131:     total_batches = 0
132:     total_samples = 0
133:     total_time = 0
134:     total_data_load_time = 0
135: 
136:     # Measure peak memory during training
137:     peak_memory = memory_after
138: 
139:     print(
140:         f"\nStarting training loop with {num_epochs} epochs (max {max_batches} batches per epoch)"
141:     )
142: 
143:     for epoch in range(num_epochs):
144:         while total_batches < max_batches:
145:             batch_start = time.perf_counter()
146: 
147:             try:
148:                 inputs, labels = next(it)
149:             except StopIteration:
150:                 break
151: 
152:             # Move data to device
153:             inputs = inputs.to(device)
154:             labels = labels.to(device)
155: 
156:             # Capture data fetch time (including sending to device)
157:             data_load_time = time.perf_counter() - batch_start
158: 
159:             # Forward pass
160:             outputs = model(inputs)
161:             loss = criterion(outputs, labels)
162: 
163:             # Backward and optimize
164:             optimizer.zero_grad()
165:             loss.backward()
166:             optimizer.step()
167: 
168:             # Capture batch time
169:             batch_time = time.perf_counter() - batch_start
170: 
171:             total_batches += 1
172:             total_samples += inputs.size(0)
173:             total_data_load_time += data_load_time
174:             total_time += batch_time
175: 
176:             # Update peak memory and log memory usage periodically
177:             if total_batches % 5 == 0:
178:                 # Force garbage collection before measuring memory
179:                 gc.collect()
180:                 current_memory = get_memory_usage()
````
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 181-240
````python
181: 
182:                 if current_memory > peak_memory:
183:                     peak_memory = current_memory
184: 
185:             if total_batches % logging_freq == 0:
186:                 print(
187:                     f"Epoch {epoch + 1}, Batch {total_batches}, "
188:                     f"Time: {batch_time:.4f}s, "
189:                     f"Memory: {current_memory:.2f} MB"
190:                 )
191: 
192:     # Calculate statistics
193:     avg_data_load_time = (
194:         total_data_load_time / total_batches if total_batches > 0 else 0
195:     )
196:     avg_batch_time = total_time / total_batches if total_batches > 0 else 0
197:     samples_per_second = total_samples / total_time if total_time > 0 else 0
198: 
199:     results = {
200:         "dataloader_init_time": dataloader_init_time,
201:         "num_workers": num_workers,
202:         "batch_size": batch_size,
203:         "total_batches": total_batches,
204:         "avg_batch_time": avg_batch_time,
205:         "avg_data_load_time": avg_data_load_time,
206:         "samples_per_second": samples_per_second,
207:         "peak_memory_mb": peak_memory,
208:         "memory_increase_mb": peak_memory - memory_before,
209:     }
210: 
211:     print("\nResults:")
212:     print(f"  DataLoader init time: {dataloader_init_time:.4f} seconds")
213:     print(f"  Average data loading time: {avg_data_load_time:.4f} seconds")
214:     print(f"  Average batch time: {avg_batch_time:.4f} seconds")
215:     print(f"  Samples per second: {samples_per_second:.2f}")
216:     print(f"  Peak memory usage: {peak_memory:.2f} MB")
217:     print(f"  Memory increase: {peak_memory - memory_before:.2f} MB")
218: 
219:     # Clean up
220:     del model, optimizer
221:     del dataloader
222: 
223:     # Force garbage collection
224:     gc.collect()
225:     torch.cuda.empty_cache()
226: 
227:     return results
228: 
229: 
230: def main():
231:     parser = argparse.ArgumentParser(
232:         description="Benchmark PyTorch DataLoader with different worker methods"
233:     )
234:     parser.add_argument("--data_path", required=True, help="Path to dataset")
235:     parser.add_argument("--batch_size", type=int, default=32, help="Batch size")
236:     parser.add_argument("--num_workers", type=int, default=4, help="Number of workers")
237:     parser.add_argument(
238:         "--max_batches",
239:         type=int,
240:         default=100,
````
- EN: Implements callable logic such as `main`.
- CN: 实现可调用逻辑，例如 `main`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-300
````python
241:         help="Maximum number of batches per epoch",
242:     )
243:     parser.add_argument("--num_epochs", type=int, default=1, help="Number of epochs")
244:     parser.add_argument(
245:         "--multiprocessing_context",
246:         choices=["fork", "spawn", "forkserver"],
247:         default="forkserver",
248:         help="Multiprocessing context to use (fork, spawn, forkserver)",
249:     )
250:     parser.add_argument(
251:         "--dataset_copies",
252:         type=int,
253:         default=1,
254:         help="Number of copies of the dataset to concatenate (for testing memory usage)",
255:     )
256:     parser.add_argument(
257:         "--logging_freq",
258:         type=int,
259:         default=10,
260:         help="Frequency of logging memory usage during training",
261:     )
262:     args = parser.parse_args()
263: 
264:     # Print system info
265:     print("System Information:")
266:     # The following are handy for debugging if building from source worked correctly
267:     print(f"  PyTorch version: {torch.__version__}")
268:     print(f"  PyTorch location: {torch.__file__}")
269:     print(f"  Torchvision version: {torchvision.__version__}")
270:     print(f"  Torchvision location: {torchvision.__file__}")
271:     print(f"  CUDA available: {torch.cuda.is_available()}")
272:     if torch.cuda.is_available():
273:         print(f"  CUDA device: {torch.cuda.get_device_name(0)}")
274:     print(f"  CPU count: {psutil.cpu_count(logical=True)}")
275:     print(f"  Physical CPU cores: {psutil.cpu_count(logical=False)}")
276:     print(f"  Total system memory: {psutil.virtual_memory().total / (1024**3):.2f} GB")
277: 
278:     # Define transforms
279:     transform = transforms.Compose(
280:         [
281:             transforms.Resize(256),
282:             transforms.CenterCrop(224),
283:             transforms.ToTensor(),
284:             transforms.Normalize(mean=[0.485, 0.456, 0.406], std=[0.229, 0.224, 0.225]),
285:         ]
286:     )
287: 
288:     # Load dataset
289:     print(f"\nLoading dataset from {args.data_path} ({args.dataset_copies} copies)")
290: 
291:     # Try to load as ImageFolder
292:     datasets = []
293:     for _ in range(args.dataset_copies):
294:         base_dataset = torchvision.datasets.ImageFolder(
295:             args.data_path, transform=transform
296:         )
297:         datasets.append(copy.deepcopy(base_dataset))
298:         del base_dataset
299:     dataset = ConcatDataset(datasets)
300: 
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 301-316
````python
301:     print(f"Dataset size: {len(dataset)}")
302: 
303:     # Run benchmark with specified worker method
304:     benchmark_dataloader(
305:         dataset,
306:         batch_size=args.batch_size,
307:         num_workers=args.num_workers,
308:         multiprocessing_context=args.multiprocessing_context,
309:         num_epochs=args.num_epochs,
310:         max_batches=args.max_batches,
311:         logging_freq=args.logging_freq,
312:     )
313: 
314: 
315: if __name__ == "__main__":
316:     main()
````
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `get_memory_usage` / 符号 `get_memory_usage`
- Symbol `print_detailed_memory` / 符号 `print_detailed_memory`
- Symbol `create_model` / 符号 `create_model`
- Symbol `benchmark_dataloader` / 符号 `benchmark_dataloader`

## Dependencies / 依赖关系
- Python imports: `argparse`, `copy`, `gc`, `time`, `psutil`, `torchvision`, `torchvision.transforms`, `torchvision.models`, `torch`, `torch.nn`
- Python 导入: `argparse`, `copy`, `gc`, `time`, `psutil`, `torchvision`, `torchvision.transforms`, `torchvision.models`, `torch`, `torch.nn`
