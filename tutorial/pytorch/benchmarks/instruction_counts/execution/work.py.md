# work.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/instruction_counts/execution/work.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40
````python
 1: """Handle the details of subprocess calls and retries for a given benchmark run."""
 2: 
 3: # mypy: ignore-errors
 4: 
 5: import dataclasses
 6: import json
 7: import os
 8: import pickle
 9: import signal
10: import subprocess
11: import time
12: import uuid
13: from typing import TYPE_CHECKING
14: 
15: from core.api import AutoLabels
16: from core.types import Label
17: from core.utils import get_temp_dir
18: from worker.main import (
19:     WORKER_PATH,
20:     WorkerFailure,
21:     WorkerOutput,
22:     WorkerTimerArgs,
23:     WorkerUnpickler,
24: )
25: 
26: 
27: if TYPE_CHECKING:
28:     PopenType = subprocess.Popen[bytes]
29: else:
30:     PopenType = subprocess.Popen
31: 
32: 
33: # Mitigate https://github.com/pytorch/pytorch/issues/37377
34: _ENV = "MKL_THREADING_LAYER=GNU"
35: _PYTHON = "python"
36: PYTHON_CMD = f"{_ENV} {_PYTHON}"
37: 
38: # We must specify `bash` so that `source activate ...` always works
39: SHELL = "/bin/bash"
40: 
````
- EN: Handles module imports such as `dataclasses`, `json`, `os`, `pickle`.
- CN: 处理模块导入，例如 `dataclasses`, `json`, `os`, `pickle`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 41-80
````python
41: 
42: @dataclasses.dataclass(frozen=True)
43: class WorkOrder:
44:     """Spec to schedule work with the benchmark runner."""
45: 
46:     label: Label
47:     autolabels: AutoLabels
48:     timer_args: WorkerTimerArgs
49:     source_cmd: str | None = None
50:     timeout: float | None = None
51:     retries: int = 0
52: 
53:     def __hash__(self) -> int:
54:         return id(self)
55: 
56:     def __str__(self) -> str:
57:         return json.dumps(
58:             {
59:                 "label": self.label,
60:                 "autolabels": self.autolabels.as_dict,
61:                 "num_threads": self.timer_args.num_threads,
62:             }
63:         )
64: 
65: 
66: class _BenchmarkProcess:
67:     """Wraps subprocess.Popen for a given WorkOrder."""
68: 
69:     _work_order: WorkOrder
70:     _cpu_list: str | None
71:     _proc: PopenType
72: 
73:     # Internal bookkeeping
74:     _communication_file: str
75:     _start_time: float
76:     _end_time: float | None = None
77:     _retcode: int | None
78:     _result: WorkerOutput | WorkerFailure | None = None
79: 
80:     def __init__(self, work_order: WorkOrder, cpu_list: str | None) -> None:
````
- EN: Declares or extends types including `WorkOrder`, `_BenchmarkProcess`.
- CN: 声明或扩展类型，包括 `WorkOrder`, `_BenchmarkProcess`。
- EN: Implements callable logic such as `__hash__`, `__str__`, `__init__`.
- CN: 实现可调用逻辑，例如 `__hash__`, `__str__`, `__init__`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 81-120
````python
 81:         self._work_order = work_order
 82:         self._cpu_list = cpu_list
 83:         self._start_time = time.time()
 84:         self._communication_file = os.path.join(get_temp_dir(), f"{uuid.uuid4()}.pkl")
 85:         with open(self._communication_file, "wb") as f:
 86:             pickle.dump(self._work_order.timer_args, f)
 87: 
 88:         self._proc = subprocess.Popen(
 89:             self.cmd,
 90:             stdout=subprocess.PIPE,
 91:             stderr=subprocess.STDOUT,
 92:             shell=True,
 93:             executable=SHELL,
 94:         )
 95: 
 96:     def clone(self) -> "_BenchmarkProcess":
 97:         return _BenchmarkProcess(self._work_order, self._cpu_list)
 98: 
 99:     @property
100:     def cmd(self) -> str:
101:         cmd: list[str] = []
102:         if self._work_order.source_cmd is not None:
103:             cmd.extend([self._work_order.source_cmd, "&&"])
104: 
105:         cmd.append(_ENV)
106: 
107:         if self._cpu_list is not None:
108:             cmd.extend(
109:                 [
110:                     f"GOMP_CPU_AFFINITY={self._cpu_list}",
111:                     "taskset",
112:                     "--cpu-list",
113:                     self._cpu_list,
114:                 ]
115:             )
116: 
117:         cmd.extend(
118:             [
119:                 _PYTHON,
120:                 WORKER_PATH,
````
- EN: Implements callable logic such as `clone`, `cmd`.
- CN: 实现可调用逻辑，例如 `clone`, `cmd`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-160
````python
121:                 "--communication-file",
122:                 self._communication_file,
123:             ]
124:         )
125:         return " ".join(cmd)
126: 
127:     @property
128:     def duration(self) -> float:
129:         return (self._end_time or time.time()) - self._start_time
130: 
131:     @property
132:     def result(self) -> WorkerOutput | WorkerFailure:
133:         self._maybe_collect()
134:         if self._result is None:
135:             raise AssertionError("result is None after collection")
136:         return self._result
137: 
138:     def poll(self) -> int | None:
139:         self._maybe_collect()
140:         return self._retcode
141: 
142:     def interrupt(self) -> None:
143:         """Soft interrupt. Allows subprocess to cleanup."""
144:         self._proc.send_signal(signal.SIGINT)
145: 
146:     def terminate(self) -> None:
147:         """Hard interrupt. Immediately SIGTERM subprocess."""
148:         self._proc.terminate()
149: 
150:     def _maybe_collect(self) -> None:
151:         if self._result is not None:
152:             # We've already collected the results.
153:             return
154: 
155:         self._retcode = self._proc.poll()
156:         if self._retcode is None:
157:             # `_proc` is still running
158:             return
159: 
160:         with open(self._communication_file, "rb") as f:
````
- EN: Implements callable logic such as `duration`, `result`, `poll`, `interrupt`.
- CN: 实现可调用逻辑，例如 `duration`, `result`, `poll`, `interrupt`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 161-200
````python
161:             result = WorkerUnpickler(f).load_output()
162: 
163:         if isinstance(result, WorkerOutput) and self._retcode:
164:             # Worker managed to complete the designated task, but worker
165:             # process did not finish cleanly.
166:             result = WorkerFailure("Worker failed silently.")
167: 
168:         if isinstance(result, WorkerTimerArgs):
169:             # Worker failed, but did not write a result so we're left with the
170:             # original TimerArgs. Grabbing all of stdout and stderr isn't
171:             # ideal, but we don't have a better way to determine what to keep.
172:             proc_stdout = self._proc.stdout
173:             if proc_stdout is None:
174:                 raise AssertionError("proc_stdout is None")
175:             result = WorkerFailure(failure_trace=proc_stdout.read().decode("utf-8"))
176: 
177:         self._result = result
178:         self._end_time = time.time()
179: 
180:         # Release communication file.
181:         os.remove(self._communication_file)
182: 
183: 
184: class InProgress:
185:     """Used by the benchmark runner to track outstanding jobs.
186:     This class handles bookkeeping and timeout + retry logic.
187:     """
188: 
189:     _proc: _BenchmarkProcess
190:     _timeouts: int = 0
191: 
192:     def __init__(self, work_order: WorkOrder, cpu_list: str | None):
193:         self._work_order = work_order
194:         self._proc = _BenchmarkProcess(work_order, cpu_list)
195: 
196:     @property
197:     def work_order(self) -> WorkOrder:
198:         return self._proc._work_order
199: 
200:     @property
````
- EN: Declares or extends types including `InProgress`, `handles`.
- CN: 声明或扩展类型，包括 `InProgress`, `handles`。
- EN: Implements callable logic such as `__init__`, `work_order`.
- CN: 实现可调用逻辑，例如 `__init__`, `work_order`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 201-239
````python
201:     def cpu_list(self) -> str | None:
202:         return self._proc._cpu_list
203: 
204:     @property
205:     def proc(self) -> _BenchmarkProcess:
206:         # NB: For cleanup only.
207:         return self._proc
208: 
209:     @property
210:     def duration(self) -> float:
211:         return self._proc.duration
212: 
213:     def check_finished(self) -> bool:
214:         if self._proc.poll() is not None:
215:             return True
216: 
217:         timeout = self.work_order.timeout
218:         if timeout is None or self._proc.duration < timeout:
219:             return False
220: 
221:         self._timeouts += 1
222:         max_attempts = (self._work_order.retries or 0) + 1
223:         if self._timeouts < max_attempts:
224:             print(
225:                 f"\nTimeout: {self._work_order.label}, {self._work_order.autolabels} "
226:                 f"(Attempt {self._timeouts} / {max_attempts})"
227:             )
228:             self._proc.interrupt()
229:             self._proc = self._proc.clone()
230:             return False
231: 
232:         raise subprocess.TimeoutExpired(cmd=self._proc.cmd, timeout=timeout)
233: 
234:     @property
235:     def result(self) -> WorkerOutput | WorkerFailure:
236:         return self._proc.result
237: 
238:     def __hash__(self) -> int:
239:         return id(self)
````
- EN: Implements callable logic such as `cpu_list`, `proc`, `duration`, `check_finished`.
- CN: 实现可调用逻辑，例如 `cpu_list`, `proc`, `duration`, `check_finished`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `WorkOrder` / 符号 `WorkOrder`
- Symbol `__hash__` / 符号 `__hash__`
- Symbol `__str__` / 符号 `__str__`
- Symbol `_BenchmarkProcess` / 符号 `_BenchmarkProcess`

## Dependencies / 依赖关系
- Python imports: `dataclasses`, `json`, `os`, `pickle`, `signal`, `subprocess`, `time`, `uuid`, `typing`, `core.api`
- Python 导入: `dataclasses`, `json`, `os`, `pickle`, `signal`, `subprocess`, `time`, `uuid`, `typing`, `core.api`
