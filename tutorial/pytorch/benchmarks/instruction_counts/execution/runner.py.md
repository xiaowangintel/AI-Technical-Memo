# runner.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/instruction_counts/execution/runner.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60
````python
 1: """Run benchmarks while handling parallelism, isolation, and fault tolerance."""
 2: 
 3: # mypy: ignore-errors
 4: 
 5: import math
 6: import multiprocessing
 7: import subprocess
 8: import textwrap
 9: import threading
10: import time
11: 
12: from worker.main import WorkerFailure, WorkerOutput
13: 
14: from execution.work import InProgress, PYTHON_CMD, SHELL, WorkOrder
15: 
16: 
17: CPU_COUNT: int = multiprocessing.cpu_count()
18: 
19: 
20: class WorkerFailed(Exception):
21:     """Raised in the main process when a worker failure is detected."""
22: 
23:     def __init__(self, cmd: str, wrapped_trace: str | None = None) -> None:
24:         self.cmd: str = cmd
25:         self.wrapped_trace: str | None = wrapped_trace
26:         super().__init__()
27: 
28: 
29: class CorePool:
30:     """Allocator style helper class to assign individual tasks to a core range.
31: 
32:     Pinning tasks to separate cores (or core ranges if `num_threads` > 1)
33:     serves two purposes. First, it prevents the machine from being overloaded,
34:     which can result in OOMs or Callgrind crashes. Second, it helps reduce
35:     noise in the wall times, which are collected as a secondary metric. For
36:     multi-threaded workloads, adjacency is important. Often pairs of cores
37:     share silicon (e.g. cache), while far away cores may lie on separate NUMA
38:     nodes. For this reason, CorePool will only allocate contiguous core ranges.
39:     This falls short of full architecture awareness, and instead tries to find
40:     a balance between rigor and engineering complexity.
41:     """
42: 
43:     def __init__(self, min_core_id: int, max_core_id: int) -> None:
44:         if min_core_id < 0:
45:             raise AssertionError(f"min_core_id must be >= 0, got {min_core_id}")
46:         if max_core_id < min_core_id:
47:             raise AssertionError(
48:                 f"max_core_id ({max_core_id}) must be >= min_core_id ({min_core_id})"
49:             )
50:         if max_core_id >= CPU_COUNT:
51:             raise AssertionError(
52:                 f"max_core_id ({max_core_id}) must be < CPU_COUNT ({CPU_COUNT})"
53:             )
54: 
55:         self._min_core_id: int = min_core_id
56:         self._max_core_id: int = max_core_id
57:         self._num_cores = max_core_id - min_core_id + 1
58:         print(f"Core pool created: cores {self._min_core_id}-{self._max_core_id}")
59: 
60:         self._available: list[bool] = [
````
- EN: Handles module imports such as `math`, `multiprocessing`, `subprocess`, `textwrap`.
- CN: 处理模块导入，例如 `math`, `multiprocessing`, `subprocess`, `textwrap`。
- EN: Declares or extends types including `WorkerFailed`, `CorePool`, `to`.
- CN: 声明或扩展类型，包括 `WorkerFailed`, `CorePool`, `to`。
- EN: Implements callable logic such as `__init__`.
- CN: 实现可调用逻辑，例如 `__init__`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。

### Lines 61-120
````python
 61:             True for _ in range(min_core_id, min_core_id + self._num_cores)
 62:         ]
 63: 
 64:         self._reservations: dict[str, tuple[int, ...]] = {}
 65:         self._lock = threading.Lock()
 66: 
 67:     def reserve(self, n: int) -> str | None:
 68:         """Simple first-fit policy.
 69: 
 70:         If successful, return a string for `taskset`. Otherwise, return None.
 71:         """
 72:         with self._lock:
 73:             for lower_index in range(self._num_cores - n + 1):
 74:                 indices = tuple(range(lower_index, lower_index + n))
 75:                 if all(self._available[i] for i in indices):
 76:                     for i in indices:
 77:                         self._available[i] = False
 78: 
 79:                     lower_core = indices[0] + self._min_core_id
 80:                     upper_core = indices[-1] + self._min_core_id
 81:                     key = f"{lower_core}-{upper_core}" if n > 1 else f"{lower_core}"
 82:                     self._reservations[key] = indices
 83:                     return key
 84:         return None
 85: 
 86:     def release(self, key: str) -> None:
 87:         with self._lock:
 88:             for i in self._reservations[key]:
 89:                 self._available[i] = True
 90:             self._reservations.pop(key)
 91: 
 92: 
 93: class Runner:
 94:     def __init__(
 95:         self,
 96:         work_items: tuple[WorkOrder, ...],
 97:         core_pool: CorePool | None = None,
 98:         cadence: float = 1.0,
 99:     ) -> None:
100:         self._work_items: tuple[WorkOrder, ...] = work_items
101:         self._core_pool: CorePool = core_pool or CorePool(0, CPU_COUNT - 4)
102:         self._cadence: float = cadence
103: 
104:         # Working state.
105:         self._work_queue: list[WorkOrder] = list(work_items)
106:         self._active_jobs: list[InProgress] = []
107:         self._results: dict[WorkOrder, WorkerOutput] = {}
108: 
109:         # Debug information for ETA and error messages.
110:         self._start_time: float = -1
111:         self._durations: dict[WorkOrder, float] = {}
112:         self._currently_processed: WorkOrder | None = None
113: 
114:         if len(work_items) != len(set(work_items)):
115:             raise ValueError("Duplicate work items.")
116: 
117:     def run(self) -> dict[WorkOrder, WorkerOutput]:
118:         try:
119:             return self._run()
120: 
````
- EN: Declares or extends types including `Runner`.
- CN: 声明或扩展类型，包括 `Runner`。
- EN: Implements callable logic such as `reserve`, `release`, `__init__`, `run`.
- CN: 实现可调用逻辑，例如 `reserve`, `release`, `__init__`, `run`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-180
````python
121:         except KeyboardInterrupt:
122:             print("\n\nKeyboardInterrupt (ctrl-c) detected. Shutting down children.")
123:             self._force_shutdown(verbose=False)
124:             raise
125: 
126:         except subprocess.TimeoutExpired:
127:             print("\n\nJob timed out. Shutting down children.")
128:             self._force_shutdown(verbose=True)
129:             raise
130: 
131:         except WorkerFailed as e:
132:             print("Shutting down all outstanding jobs before re-raising.")
133:             self._force_shutdown(verbose=True)
134:             print(f"Cmd: {e.cmd}")
135:             if e.wrapped_trace:
136:                 print(e.wrapped_trace)
137:             else:
138:                 print("Unknown failure. (Worker did not report exception contents.)")
139:             raise
140: 
141:         except BaseException:
142:             print("\n\nUnknown exception. Shutting down jobs before re-raising.")
143:             self._force_shutdown(verbose=True)
144:             raise
145: 
146:     def _run(self) -> dict[WorkOrder, WorkerOutput]:
147:         self._start_time = time.time()
148:         self._canary_import()
149:         while self._work_queue or self._active_jobs:
150:             t0 = time.time()
151:             self._update_active_jobs()
152:             self._enqueue_new_jobs()
153:             self._print_progress()
154:             time.sleep(max(self._cadence - (time.time() - t0), 0.0))
155:         print(f"\nTotal time: {time.time() - self._start_time:.0f} seconds")
156:         return self._results.copy()
157: 
158:     def _update_active_jobs(self) -> None:
159:         active_jobs: list[InProgress] = []
160:         for job in self._active_jobs:
161:             self._currently_processed = job.work_order
162:             if not job.check_finished():
163:                 active_jobs.append(job)
164:                 continue
165: 
166:             result: WorkerOutput | WorkerFailure = job.result
167:             if isinstance(result, WorkerOutput):
168:                 self._results[job.work_order] = result
169:                 if job.cpu_list is None:
170:                     raise AssertionError("job.cpu_list must not be None")
171:                 self._core_pool.release(job.cpu_list)
172:                 self._durations[job.work_order] = job.duration
173: 
174:             else:
175:                 if not isinstance(result, WorkerFailure):
176:                     raise AssertionError(f"expected WorkerFailure, got {type(result)}")
177:                 raise WorkerFailed(cmd=job.proc.cmd, wrapped_trace=result.failure_trace)
178:         self._currently_processed = None
179:         self._active_jobs.clear()
180:         self._active_jobs.extend(active_jobs)
````
- EN: Implements callable logic such as `_run`, `_update_active_jobs`.
- CN: 实现可调用逻辑，例如 `_run`, `_update_active_jobs`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 181-240
````python
181: 
182:     def _enqueue_new_jobs(self) -> None:
183:         work_queue: list[WorkOrder] = []
184:         for i, work_order in enumerate(self._work_queue):
185:             self._currently_processed = work_order
186:             cpu_list = self._core_pool.reserve(work_order.timer_args.num_threads)
187: 
188:             if cpu_list is None:
189:                 work_queue.append(work_order)
190:             else:
191:                 self._active_jobs.append(InProgress(work_order, cpu_list))
192: 
193:                 # Stagger creation. This helps with contention.
194:                 time.sleep(0.5)
195:         self._currently_processed = None
196:         self._work_queue.clear()
197:         self._work_queue.extend(work_queue)
198: 
199:     def _print_progress(self) -> None:
200:         fraction = f"{len(self._results)} / {len(self._work_items)}"
201:         elapsed = f"{time.time() - self._start_time:.0f} seconds"
202:         if len(self._results) < 5:
203:             eta = "Unknown"
204:         else:
205:             remaining = len(self._work_items) - len(self._results)
206:             iters_remaining = math.ceil(remaining / self._core_pool._num_cores)
207:             mean_time = sum(self._durations.values()) / len(self._durations)
208:             eta_minutes = math.ceil(iters_remaining * mean_time / 60)
209:             eta = f"~{eta_minutes:.0f} minute{'s' if eta_minutes > 1 else ''}"
210:         print(f"\r{fraction} ({elapsed}), ETA: {eta}", end="")
211: 
212:     def _force_shutdown(self, verbose: bool = False) -> None:
213:         """Try to interrupt jobs, and kill if need be.
214:         We would prefer to softly terminate jobs so that they have a chance to
215:         clean up before shutting down.
216:         """
217:         for job in self._active_jobs:
218:             job.proc.interrupt()
219: 
220:         if verbose and self._currently_processed is not None:
221:             print(
222:                 textwrap.dedent(
223:                     f"""
224:                 Failed when processing the following Job:
225:                   Label:      {self._currently_processed.label}
226:                   AutoLabels: {self._currently_processed.autolabels}
227:                   Source cmd: {self._currently_processed.source_cmd}
228:             """
229:                 ).strip()
230:                 + "\n"
231:             )
232: 
233:         if self._active_jobs:
234:             time.sleep(0.5)
235: 
236:         remaining_jobs = [j for j in self._active_jobs if j.proc.poll() is None]
237:         if remaining_jobs:
238:             print(
239:                 f"SIGINT sent to {len(self._active_jobs)} jobs, "
240:                 f"{len(remaining_jobs)} have not yet exited.\n"
````
- EN: Implements callable logic such as `_enqueue_new_jobs`, `_print_progress`, `_force_shutdown`.
- CN: 实现可调用逻辑，例如 `_enqueue_new_jobs`, `_print_progress`, `_force_shutdown`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 241-279
````python
241:                 "Entering short cleanup loop, after which stragglers will "
242:                 "be forcibly terminated."
243:             )
244: 
245:             for _ in range(5):
246:                 time.sleep(2.0)
247:                 remaining_jobs = [j for j in remaining_jobs if j.proc.poll() is None]
248:                 if remaining_jobs:
249:                     print(f"{len(remaining_jobs)} still remain.")
250:                 else:
251:                     print("All remaining jobs have gracefully terminated.")
252:                     return
253: 
254:             print(f"{len(remaining_jobs)} jobs refused to exit. Forcibly terminating.")
255:             for j in remaining_jobs:
256:                 j.proc.terminate()
257: 
258:     def _canary_import(self) -> None:
259:         """Make sure we can import torch before launching a slew of workers."""
260:         source_cmds: set[str] = set()
261:         for w in self._work_items:
262:             if w.source_cmd is not None:
263:                 source_cmds.add(f"{w.source_cmd} && ")
264: 
265:         for source_cmd in source_cmds or {""}:
266:             cmd = f'{source_cmd}{PYTHON_CMD} -c "import torch"'
267:             proc = subprocess.run(
268:                 cmd,
269:                 shell=True,
270:                 stdout=subprocess.PIPE,
271:                 stderr=subprocess.STDOUT,
272:                 encoding="utf-8",
273:                 executable=SHELL,
274:             )
275: 
276:             if proc.returncode:
277:                 raise ImportError(
278:                     f"Failed to import torch in subprocess: {cmd}\n{proc.stdout}"
279:                 )
````
- EN: Implements callable logic such as `_canary_import`.
- CN: 实现可调用逻辑，例如 `_canary_import`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `WorkerFailed` / 符号 `WorkerFailed`
- Symbol `__init__` / 符号 `__init__`
- Symbol `CorePool` / 符号 `CorePool`
- Symbol `reserve` / 符号 `reserve`

## Dependencies / 依赖关系
- Python imports: `math`, `multiprocessing`, `subprocess`, `textwrap`, `threading`, `time`, `worker.main`, `execution.work`
- Python 导入: `math`, `multiprocessing`, `subprocess`, `textwrap`, `threading`, `time`, `worker.main`, `execution.work`
