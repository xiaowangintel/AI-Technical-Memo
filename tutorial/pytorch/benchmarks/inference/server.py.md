# server.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/inference/server.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60
````python
 1: import argparse
 2: import asyncio
 3: import os.path
 4: import subprocess
 5: import threading
 6: import time
 7: from concurrent.futures import ThreadPoolExecutor
 8: from queue import Empty
 9: 
10: import numpy as np
11: import pandas as pd
12: 
13: import torch
14: import torch.multiprocessing as mp
15: 
16: 
17: class FrontendWorker(mp.Process):
18:     """
19:     This worker will send requests to a backend process, and measure the
20:     throughput and latency of those requests as well as GPU utilization.
21:     """
22: 
23:     def __init__(
24:         self,
25:         metrics_dict,
26:         request_queue,
27:         response_queue,
28:         read_requests_event,
29:         batch_size,
30:         num_iters=10,
31:     ):
32:         super().__init__()
33:         self.metrics_dict = metrics_dict
34:         self.request_queue = request_queue
35:         self.response_queue = response_queue
36:         self.read_requests_event = read_requests_event
37:         self.warmup_event = mp.Event()
38:         self.batch_size = batch_size
39:         self.num_iters = num_iters
40:         self.poll_gpu = True
41:         self.start_send_time = None
42:         self.end_recv_time = None
43: 
44:     def _run_metrics(self, metrics_lock):
45:         """
46:         This function will poll the response queue until it has received all
47:         responses. It records the startup latency, the average, max, min latency
48:         as well as throughput of requests.
49:         """
50:         warmup_response_time = None
51:         response_times = []
52: 
53:         for i in range(self.num_iters + 1):
54:             response, request_time = self.response_queue.get()
55:             if warmup_response_time is None:
56:                 self.warmup_event.set()
57:                 warmup_response_time = time.time() - request_time
58:             else:
59:                 response_times.append(time.time() - request_time)
60: 
````
- EN: Handles module imports such as `argparse`, `asyncio`, `os.path`, `subprocess`.
- CN: 处理模块导入，例如 `argparse`, `asyncio`, `os.path`, `subprocess`。
- EN: Declares or extends types including `FrontendWorker`.
- CN: 声明或扩展类型，包括 `FrontendWorker`。
- EN: Implements callable logic such as `__init__`, `_run_metrics`.
- CN: 实现可调用逻辑，例如 `__init__`, `_run_metrics`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 61-120
````python
 61:         self.end_recv_time = time.time()
 62:         self.poll_gpu = False
 63: 
 64:         response_times = np.array(response_times)
 65:         with metrics_lock:
 66:             self.metrics_dict["warmup_latency"] = warmup_response_time
 67:             self.metrics_dict["average_latency"] = response_times.mean()
 68:             self.metrics_dict["max_latency"] = response_times.max()
 69:             self.metrics_dict["min_latency"] = response_times.min()
 70:             self.metrics_dict["throughput"] = (self.num_iters * self.batch_size) / (
 71:                 self.end_recv_time - self.start_send_time
 72:             )
 73: 
 74:     def _run_gpu_utilization(self, metrics_lock):
 75:         """
 76:         This function will poll nvidia-smi for GPU utilization every 100ms to
 77:         record the average GPU utilization.
 78:         """
 79: 
 80:         def get_gpu_utilization():
 81:             try:
 82:                 nvidia_smi_output = subprocess.check_output(
 83:                     [
 84:                         "nvidia-smi",
 85:                         "--query-gpu=utilization.gpu",
 86:                         "--id=0",
 87:                         "--format=csv,noheader,nounits",
 88:                     ]
 89:                 )
 90:                 gpu_utilization = nvidia_smi_output.decode().strip()
 91:                 return gpu_utilization
 92:             except subprocess.CalledProcessError:
 93:                 return "N/A"
 94: 
 95:         gpu_utilizations = []
 96: 
 97:         while self.poll_gpu:
 98:             gpu_utilization = get_gpu_utilization()
 99:             if gpu_utilization != "N/A":
100:                 gpu_utilizations.append(float(gpu_utilization))
101: 
102:         with metrics_lock:
103:             self.metrics_dict["gpu_util"] = torch.tensor(gpu_utilizations).mean().item()
104: 
105:     def _send_requests(self):
106:         """
107:         This function will send one warmup request, and then num_iters requests
108:         to the backend process.
109:         """
110: 
111:         fake_data = torch.randn(self.batch_size, 3, 250, 250, requires_grad=False)
112:         other_data = [
113:             torch.randn(self.batch_size, 3, 250, 250, requires_grad=False)
114:             for i in range(self.num_iters)
115:         ]
116: 
117:         # Send one batch of warmup data
118:         self.request_queue.put((fake_data, time.time()))
119:         # Tell backend to poll queue for warmup request
120:         self.read_requests_event.set()
````
- EN: Implements callable logic such as `_run_gpu_utilization`, `get_gpu_utilization`, `_send_requests`.
- CN: 实现可调用逻辑，例如 `_run_gpu_utilization`, `get_gpu_utilization`, `_send_requests`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-180
````python
121:         self.warmup_event.wait()
122:         # Tell backend to poll queue for rest of requests
123:         self.read_requests_event.set()
124: 
125:         # Send fake data
126:         self.start_send_time = time.time()
127:         for i in range(self.num_iters):
128:             self.request_queue.put((other_data[i], time.time()))
129: 
130:     def run(self):
131:         # Lock for writing to metrics_dict
132:         metrics_lock = threading.Lock()
133:         requests_thread = threading.Thread(target=self._send_requests)
134:         metrics_thread = threading.Thread(
135:             target=self._run_metrics, args=(metrics_lock,)
136:         )
137:         gpu_utilization_thread = threading.Thread(
138:             target=self._run_gpu_utilization, args=(metrics_lock,)
139:         )
140: 
141:         requests_thread.start()
142:         metrics_thread.start()
143: 
144:         # only start polling GPU utilization after the warmup request is complete
145:         self.warmup_event.wait()
146:         gpu_utilization_thread.start()
147: 
148:         requests_thread.join()
149:         metrics_thread.join()
150:         gpu_utilization_thread.join()
151: 
152: 
153: class BackendWorker:
154:     """
155:     This worker will take tensors from the request queue, do some computation,
156:     and then return the result back in the response queue.
157:     """
158: 
159:     def __init__(
160:         self,
161:         metrics_dict,
162:         request_queue,
163:         response_queue,
164:         read_requests_event,
165:         batch_size,
166:         num_workers,
167:         model_dir=".",
168:         compile_model=True,
169:     ):
170:         super().__init__()
171:         self.device = "cuda:0"
172:         self.metrics_dict = metrics_dict
173:         self.request_queue = request_queue
174:         self.response_queue = response_queue
175:         self.read_requests_event = read_requests_event
176:         self.batch_size = batch_size
177:         self.num_workers = num_workers
178:         self.model_dir = model_dir
179:         self.compile_model = compile_model
180:         self._setup_complete = False
````
- EN: Declares or extends types including `BackendWorker`.
- CN: 声明或扩展类型，包括 `BackendWorker`。
- EN: Implements callable logic such as `run`, `__init__`.
- CN: 实现可调用逻辑，例如 `run`, `__init__`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 181-240
````python
181:         self.h2d_stream = torch.cuda.Stream()
182:         self.d2h_stream = torch.cuda.Stream()
183:         # maps thread_id to the cuda.Stream associated with that worker thread
184:         self.stream_map = {}
185: 
186:     def _setup(self):
187:         import time
188: 
189:         from torchvision.models.resnet import BasicBlock, ResNet
190: 
191:         import torch
192: 
193:         # Create ResNet18 on meta device
194:         with torch.device("meta"):
195:             m = ResNet(BasicBlock, [2, 2, 2, 2])
196: 
197:         # Load pretrained weights
198:         start_load_time = time.time()
199:         state_dict = torch.load(
200:             f"{self.model_dir}/resnet18-f37072fd.pth",
201:             mmap=True,
202:             map_location=self.device,
203:         )
204:         self.metrics_dict["torch_load_time"] = time.time() - start_load_time
205:         m.load_state_dict(state_dict, assign=True)
206:         m.eval()
207: 
208:         if self.compile_model:
209:             start_compile_time = time.time()
210:             m.compile()
211:             end_compile_time = time.time()
212:             self.metrics_dict["m_compile_time"] = end_compile_time - start_compile_time
213:         return m
214: 
215:     def model_predict(
216:         self,
217:         model,
218:         input_buffer,
219:         copy_event,
220:         compute_event,
221:         copy_sem,
222:         compute_sem,
223:         response_list,
224:         request_time,
225:     ):
226:         # copy_sem makes sure copy_event has been recorded in the data copying thread
227:         copy_sem.acquire()
228:         self.stream_map[threading.get_native_id()].wait_event(copy_event)
229:         with torch.cuda.stream(self.stream_map[threading.get_native_id()]):
230:             with torch.no_grad():
231:                 response_list.append(model(input_buffer))
232:                 compute_event.record()
233:                 compute_sem.release()
234:         del input_buffer
235: 
236:     def copy_data(self, input_buffer, data, copy_event, copy_sem):
237:         data = data.pin_memory()
238:         with torch.cuda.stream(self.h2d_stream):
239:             input_buffer.copy_(data, non_blocking=True)
240:             copy_event.record()
````
- EN: Handles module imports such as `time`, `torchvision.models.resnet`, `torch`.
- CN: 处理模块导入，例如 `time`, `torchvision.models.resnet`, `torch`。
- EN: Implements callable logic such as `_setup`, `model_predict`, `copy_data`.
- CN: 实现可调用逻辑，例如 `_setup`, `model_predict`, `copy_data`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-300
````python
241:             copy_sem.release()
242: 
243:     def respond(self, compute_event, compute_sem, response_list, request_time):
244:         # compute_sem makes sure compute_event has been recorded in the model_predict thread
245:         compute_sem.acquire()
246:         self.d2h_stream.wait_event(compute_event)
247:         with torch.cuda.stream(self.d2h_stream):
248:             self.response_queue.put((response_list[0].cpu(), request_time))
249: 
250:     async def run(self):
251:         def worker_initializer():
252:             self.stream_map[threading.get_native_id()] = torch.cuda.Stream()
253: 
254:         worker_pool = ThreadPoolExecutor(
255:             max_workers=self.num_workers, initializer=worker_initializer
256:         )
257:         h2d_pool = ThreadPoolExecutor(max_workers=1)
258:         d2h_pool = ThreadPoolExecutor(max_workers=1)
259: 
260:         self.read_requests_event.wait()
261:         # Clear as we will wait for this event again before continuing to
262:         # poll the request_queue for the non-warmup requests
263:         self.read_requests_event.clear()
264:         while True:
265:             try:
266:                 data, request_time = self.request_queue.get(timeout=5)
267:             except Empty:
268:                 break
269: 
270:             if not self._setup_complete:
271:                 model = self._setup()
272: 
273:             copy_sem = threading.Semaphore(0)
274:             compute_sem = threading.Semaphore(0)
275:             copy_event = torch.cuda.Event()
276:             compute_event = torch.cuda.Event()
277:             response_list = []
278:             input_buffer = torch.empty(
279:                 [self.batch_size, 3, 250, 250], dtype=torch.float32, device="cuda"
280:             )
281:             asyncio.get_running_loop().run_in_executor(
282:                 h2d_pool,
283:                 self.copy_data,
284:                 input_buffer,
285:                 data,
286:                 copy_event,
287:                 copy_sem,
288:             )
289:             asyncio.get_running_loop().run_in_executor(
290:                 worker_pool,
291:                 self.model_predict,
292:                 model,
293:                 input_buffer,
294:                 copy_event,
295:                 compute_event,
296:                 copy_sem,
297:                 compute_sem,
298:                 response_list,
299:                 request_time,
300:             )
````
- EN: Implements callable logic such as `respond`, `worker_initializer`.
- CN: 实现可调用逻辑，例如 `respond`, `worker_initializer`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 301-360
````python
301:             asyncio.get_running_loop().run_in_executor(
302:                 d2h_pool,
303:                 self.respond,
304:                 compute_event,
305:                 compute_sem,
306:                 response_list,
307:                 request_time,
308:             )
309: 
310:             if not self._setup_complete:
311:                 self.read_requests_event.wait()
312:                 self._setup_complete = True
313: 
314: 
315: if __name__ == "__main__":
316:     parser = argparse.ArgumentParser()
317:     parser.add_argument("--num_iters", type=int, default=100)
318:     parser.add_argument("--batch_size", type=int, default=32)
319:     parser.add_argument("--model_dir", type=str, default=".")
320:     parser.add_argument(
321:         "--compile", default=True, action=argparse.BooleanOptionalAction
322:     )
323:     parser.add_argument("--output_file", type=str, default="output.csv")
324:     parser.add_argument(
325:         "--profile", default=False, action=argparse.BooleanOptionalAction
326:     )
327:     parser.add_argument("--num_workers", type=int, default=4)
328:     args = parser.parse_args()
329: 
330:     downloaded_checkpoint = False
331:     if not os.path.isfile(f"{args.model_dir}/resnet18-f37072fd.pth"):
332:         p = subprocess.run(
333:             [
334:                 "wget",
335:                 "https://download.pytorch.org/models/resnet18-f37072fd.pth",
336:             ]
337:         )
338:         if p.returncode == 0:
339:             downloaded_checkpoint = True
340:         else:
341:             raise RuntimeError("Failed to download checkpoint")
342: 
343:     try:
344:         mp.set_start_method("forkserver")
345:         request_queue = mp.Queue()
346:         response_queue = mp.Queue()
347:         read_requests_event = mp.Event()
348: 
349:         manager = mp.Manager()
350:         metrics_dict = manager.dict()
351:         metrics_dict["batch_size"] = args.batch_size
352:         metrics_dict["compile"] = args.compile
353: 
354:         frontend = FrontendWorker(
355:             metrics_dict,
356:             request_queue,
357:             response_queue,
358:             read_requests_event,
359:             args.batch_size,
360:             num_iters=args.num_iters,
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 361-398
````python
361:         )
362:         backend = BackendWorker(
363:             metrics_dict,
364:             request_queue,
365:             response_queue,
366:             read_requests_event,
367:             args.batch_size,
368:             args.num_workers,
369:             args.model_dir,
370:             args.compile,
371:         )
372: 
373:         frontend.start()
374: 
375:         if args.profile:
376: 
377:             def trace_handler(prof):
378:                 prof.export_chrome_trace("trace.json")
379: 
380:             with torch.profiler.profile(on_trace_ready=trace_handler) as prof:
381:                 asyncio.run(backend.run())
382:         else:
383:             asyncio.run(backend.run())
384: 
385:         frontend.join()
386: 
387:         metrics_dict = {k: [v] for k, v in metrics_dict._getvalue().items()}
388:         output = pd.DataFrame.from_dict(metrics_dict, orient="columns")
389:         output_file = "./results/" + args.output_file
390:         is_empty = not os.path.isfile(output_file)
391: 
392:         with open(output_file, "a+", newline="") as file:
393:             output.to_csv(file, header=is_empty, index=False)
394: 
395:     finally:
396:         # Cleanup checkpoint file if we downloaded it
397:         if downloaded_checkpoint:
398:             os.remove(f"{args.model_dir}/resnet18-f37072fd.pth")
````
- EN: Implements callable logic such as `trace_handler`.
- CN: 实现可调用逻辑，例如 `trace_handler`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `FrontendWorker` / 符号 `FrontendWorker`
- Symbol `__init__` / 符号 `__init__`
- Symbol `_run_metrics` / 符号 `_run_metrics`
- Symbol `_run_gpu_utilization` / 符号 `_run_gpu_utilization`

## Dependencies / 依赖关系
- Python imports: `argparse`, `asyncio`, `os.path`, `subprocess`, `threading`, `time`, `concurrent.futures`, `queue`, `numpy`, `pandas`
- Python 导入: `argparse`, `asyncio`, `os.path`, `subprocess`, `threading`, `time`, `concurrent.futures`, `queue`, `numpy`, `pandas`
