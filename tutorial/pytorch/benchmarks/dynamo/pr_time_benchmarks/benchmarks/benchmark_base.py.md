# benchmark_base.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/dynamo/pr_time_benchmarks/benchmarks/benchmark_base.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60
````python
 1: import csv
 2: import gc
 3: import json
 4: import os
 5: from abc import ABC, abstractmethod
 6: from typing_extensions import Self
 7: 
 8: import torch._C._instruction_counter as i_counter
 9: import torch._dynamo.config as config
10: from torch._dynamo.utils import CompileTimeInstructionCounter
11: 
12: 
13: log_to_scuba = os.getenv("CI", "false") == "true"
14: if log_to_scuba:
15:     from fbscribelogger import make_scribe_logger
16: 
17:     scribe_log_torch_benchmark_compile_time = make_scribe_logger(
18:         "TorchBenchmarkCompileTime",
19:         """
20:         struct TorchBenchmarkCompileTimeLogEntry {
21: 
22:         # The commit SHA that triggered the workflow, e.g., 02a6b1d30f338206a71d0b75bfa09d85fac0028a. Derived from GITHUB_SHA.
23:         4: optional string commit_sha;
24: 
25:         # The unit timestamp in second for the Scuba Time Column override
26:         6: optional i64 time;
27:         7: optional i64 instruction_count; # Instruction count of compilation step
28:         8: optional string name; # Benchmark name
29: 
30:         # Commit date (not author date) of the commit in commit_sha as timestamp, e.g., 1724208105.  Increasing if merge bot is used, though not monotonic; duplicates occur when stack is landed.
31:         16: optional i64 commit_date;
32: 
33:         # A unique number for each workflow run within a repository, e.g., 19471190684. Derived from GITHUB_RUN_ID.
34:         17: optional string github_run_id;
35: 
36:         # A unique number for each attempt of a particular workflow run in a repository, e.g., 1. Derived from GITHUB_RUN_ATTEMPT.
37:         18: optional string github_run_attempt;
38: 
39:         # Indicates if branch protections or rulesets are configured for the ref that triggered the workflow run. Derived from GITHUB_REF_PROTECTED.
40:         20: optional bool github_ref_protected;
41: 
42:         # The fully-formed ref of the branch or tag that triggered the workflow run, e.g., refs/pull/133891/merge or refs/heads/main. Derived from GITHUB_REF.
43:         21: optional string github_ref;
44: 
45:         # The weight of the record according to current sampling rate
46:         25: optional i64 weight;
47: 
48:         # The name of the current job. Derived from JOB_NAME, e.g., linux-jammy-py3.8-gcc11 / test (default, 3, 4, linux.2xlarge).
49:         26: optional string github_job;
50: 
51:         # The GitHub user who triggered the job.  Derived from GITHUB_TRIGGERING_ACTOR.
52:         27: optional string github_triggering_actor;
53: 
54:         # A unique number for each run of a particular workflow in a repository, e.g., 238742. Derived from GITHUB_RUN_NUMBER.
55:         28: optional string github_run_number_str;
56:         }
57:         """,
58:     )
59: 
60: 
````
- EN: Handles module imports such as `csv`, `gc`, `json`, `os`.
- CN: 处理模块导入，例如 `csv`, `gc`, `json`, `os`。
- EN: Declares or extends types including `TorchBenchmarkCompileTimeLogEntry`.
- CN: 声明或扩展类型，包括 `TorchBenchmarkCompileTimeLogEntry`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 61-120
````python
 61: class BenchmarkBase(ABC):
 62:     # Measure total number of instruction spent in _work.
 63:     # Garbage collection is NOT disabled during _work().
 64:     _enable_instruction_count = False
 65: 
 66:     # Measure total number of instruction spent in convert_frame.compile_inner
 67:     # Garbage collection is disabled during _work() to avoid noise.
 68:     _enable_compile_time_instruction_count = False
 69: 
 70:     # number of iterations used to run when collecting instruction_count or compile_time_instruction_count.
 71:     _num_iterations = 5
 72: 
 73:     def __init__(
 74:         self,
 75:         category: str,
 76:         device: str,
 77:         backend: str = "",
 78:         mode: str = "",
 79:         dynamic=None,
 80:     ) -> None:
 81:         # These individual attributes are used to support different filters on the
 82:         # dashboard later
 83:         self._category = category
 84:         self._device = device
 85:         self._backend = backend
 86:         self._mode = mode  # Training or inference
 87:         self._dynamic = dynamic
 88: 
 89:     def with_iterations(self, value: int) -> Self:
 90:         self._num_iterations = value
 91:         return self
 92: 
 93:     def enable_instruction_count(self) -> Self:
 94:         self._enable_instruction_count = True
 95:         return self
 96: 
 97:     def enable_compile_time_instruction_count(self) -> Self:
 98:         self._enable_compile_time_instruction_count = True
 99:         return self
100: 
101:     def name(self) -> str:
102:         return ""
103: 
104:     def backend(self) -> str:
105:         return self._backend
106: 
107:     def mode(self) -> str:
108:         return self._mode
109: 
110:     def category(self) -> str:
111:         return self._category
112: 
113:     def device(self) -> str:
114:         return self._device
115: 
116:     def is_dynamic(self) -> bool | None:
117:         return self._dynamic
118: 
119:     def description(self) -> str:
120:         return ""
````
- EN: Declares or extends types including `BenchmarkBase`.
- CN: 声明或扩展类型，包括 `BenchmarkBase`。
- EN: Implements callable logic such as `__init__`, `with_iterations`, `enable_instruction_count`, `enable_compile_time_instruction_count`.
- CN: 实现可调用逻辑，例如 `__init__`, `with_iterations`, `enable_instruction_count`, `enable_compile_time_instruction_count`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-180
````python
121: 
122:     @abstractmethod
123:     def _prepare(self) -> None:
124:         pass
125: 
126:     @abstractmethod
127:     def _work(self) -> None:
128:         pass
129: 
130:     def _prepare_once(self) -> None:  # noqa: B027
131:         pass
132: 
133:     def _count_instructions(self) -> int:
134:         print(f"collecting instruction count for {self.name()}")
135:         results = []
136:         for i in range(self._num_iterations):
137:             self._prepare()
138:             id = i_counter.start()
139:             self._work()
140:             count = i_counter.end(id)
141:             print(f"instruction count for iteration {i} is {count}")
142:             results.append(count)
143:         return min(results)
144: 
145:     def _count_compile_time_instructions(self) -> int:
146:         gc.disable()
147: 
148:         try:
149:             print(f"collecting compile time instruction count for {self.name()}")
150:             config.record_compile_time_instruction_count = True
151: 
152:             results = []
153:             for i in range(self._num_iterations):
154:                 self._prepare()
155:                 gc.collect()
156:                 # CompileTimeInstructionCounter.record is only called on convert_frame._compile_inner
157:                 # hence this will only count instruction count spent in compile_inner.
158:                 CompileTimeInstructionCounter.clear()
159:                 self._work()
160:                 count = CompileTimeInstructionCounter.value()
161:                 if count == 0:
162:                     raise RuntimeError(
163:                         "compile time instruction count is 0, please check your benchmarks"
164:                     )
165:                 print(f"compile time instruction count for iteration {i} is {count}")
166:                 results.append(count)
167: 
168:             config.record_compile_time_instruction_count = False
169:             return min(results)
170:         finally:
171:             gc.enable()
172: 
173:     def _write_to_json(self, output_dir: str) -> None:
174:         """
175:         Write the result into JSON format, so that it can be uploaded to the benchmark database
176:         to be displayed on OSS dashboard. The JSON format is defined at
177:         https://github.com/pytorch/pytorch/wiki/How-to-integrate-with-PyTorch-OSS-benchmark-database
178:         """
179:         records = []
180:         for entry in self.results:
````
- EN: Implements callable logic such as `_prepare`, `_work`, `_prepare_once`, `_count_instructions`.
- CN: 实现可调用逻辑，例如 `_prepare`, `_work`, `_prepare_once`, `_count_instructions`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 181-240
````python
181:             metric_name = entry[1]
182:             value = entry[2]
183: 
184:             if not metric_name or value is None:
185:                 continue
186: 
187:             records.append(
188:                 {
189:                     "benchmark": {
190:                         "name": "pr_time_benchmarks",
191:                         "mode": self.mode(),
192:                         "extra_info": {
193:                             "is_dynamic": self.is_dynamic(),
194:                             "device": self.device(),
195:                             "description": self.description(),
196:                         },
197:                     },
198:                     "model": {
199:                         "name": self.name(),
200:                         "type": self.category(),
201:                         "backend": self.backend(),
202:                     },
203:                     "metric": {
204:                         "name": metric_name,
205:                         "benchmark_values": [value],
206:                     },
207:                 }
208:             )
209: 
210:         with open(os.path.join(output_dir, f"{self.name()}.json"), "w") as f:
211:             json.dump(records, f)
212: 
213:     def append_results(self, path: str) -> None:
214:         with open(path, "a", newline="") as csvfile:
215:             # Create a writer object
216:             writer = csv.writer(csvfile)
217:             # Write the data to the CSV file
218:             for entry in self.results:
219:                 writer.writerow(entry)
220: 
221:         # TODO (huydhn) This requires the path to write to, so it needs to be in the same place
222:         # as the CSV writer for now
223:         self._write_to_json(os.path.dirname(os.path.abspath(path)))
224: 
225:     def print(self) -> None:
226:         for entry in self.results:
227:             print(f"{entry[0]},{entry[1]},{entry[2]}")
228: 
229:     def collect_all(self) -> Self:
230:         self._prepare_once()
231:         self.results = []
232:         if (
233:             self._enable_instruction_count
234:             and self._enable_compile_time_instruction_count
235:         ):
236:             raise RuntimeError(
237:                 "not supported until we update the logger, both logs to the same field now"
238:             )
239: 
240:         if self._enable_instruction_count:
````
- EN: Implements callable logic such as `append_results`, `print`, `collect_all`.
- CN: 实现可调用逻辑，例如 `append_results`, `print`, `collect_all`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 241-267
````python
241:             r = self._count_instructions()
242:             self.results.append((self.name(), "instruction_count", r))
243:             if log_to_scuba:
244:                 scribe_log_torch_benchmark_compile_time(
245:                     name=self.name(),
246:                     instruction_count=r,
247:                 )
248:         if self._enable_compile_time_instruction_count:
249:             # enable_cpp_symbolic_shape_guards has impact on these benchmarks
250:             # Keep using False value for consistency.
251:             with config.patch("enable_cpp_symbolic_shape_guards", False):
252:                 r = self._count_compile_time_instructions()
253: 
254:             self.results.append(
255:                 (
256:                     self.name(),
257:                     "compile_time_instruction_count",
258:                     r,
259:                 )
260:             )
261:             if log_to_scuba:
262:                 # TODO add a new field compile_time_instruction_count to the logger.
263:                 scribe_log_torch_benchmark_compile_time(
264:                     name=self.name(),
265:                     instruction_count=r,
266:                 )
267:         return self
````
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `BenchmarkBase` / 符号 `BenchmarkBase`
- Symbol `__init__` / 符号 `__init__`
- Symbol `with_iterations` / 符号 `with_iterations`
- Symbol `enable_instruction_count` / 符号 `enable_instruction_count`

## Dependencies / 依赖关系
- Python imports: `csv`, `gc`, `json`, `os`, `abc`, `typing_extensions`, `torch._C._instruction_counter`, `torch._dynamo.config`, `torch._dynamo.utils`, `fbscribelogger`
- Python 导入: `csv`, `gc`, `json`, `os`, `abc`, `typing_extensions`, `torch._C._instruction_counter`, `torch._dynamo.config`, `torch._dynamo.utils`, `fbscribelogger`
