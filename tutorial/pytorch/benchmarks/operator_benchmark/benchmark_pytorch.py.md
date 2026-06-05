# benchmark_pytorch.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/operator_benchmark/benchmark_pytorch.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60
````python
 1: import json
 2: import time
 3: 
 4: import torch
 5: 
 6: 
 7: # Import the C++ extension to register the _consume operator
 8: try:
 9:     import benchmark_cpp_extension  # noqa: F401
10: except ImportError as err:
11:     # If the extension isn't built, the script must raise an error
12:     raise ImportError(
13:         "Failed to import C++ extension, please build it using \ncd pt_extension \npython -m pip install ."
14:     ) from err
15: 
16: """PyTorch performance microbenchmarks.
17: 
18: This module contains PyTorch-specific functionalities for performance
19: microbenchmarks.
20: """
21: 
22: 
23: class TorchBenchmarkBase(torch.nn.Module):
24:     """This is a base class used to create Pytorch operator benchmark.
25:     module_name is the name of the operator being benchmarked.
26:     test_name is the name (it's created by concatenating all the
27:     inputs) of a specific test
28:     """
29: 
30:     def __init__(self):
31:         super().__init__()
32:         self.user_given_name = None
33:         self._pass_count = 0
34:         self._num_inputs_require_grads = 0
35: 
36:     def _set_backward_test(self, is_backward):
37:         self._is_backward = is_backward
38: 
39:     def auto_set(self):
40:         """This is used to automatically set the require_grad for the backward patch.
41:         It is implemented based on two counters. One counter to save the number of
42:         times init has been called. The other counter to save the number of times
43:         this function itself has been called. In the very first time init is called,
44:         this function counts how many inputs require gradient. In each of the
45:         following init calls, this function will return only one true value.
46:         Here is an example:
47:             ...
48:             self.v1 = torch.rand(M, N, K, requires_grad=self.auto_set())
49:             self.v2 = torch.rand(M, N, K, requires_grad=self.auto_set())
50:             ...
51:         """
52:         if not self._is_backward:
53:             return False
54: 
55:         if self._pass_count == 0:
56:             self._num_inputs_require_grads += 1
57:             return True
58:         else:
59:             self._auto_set_counter += 1
60:             return self._pass_count == self._auto_set_counter
````
- EN: Handles module imports such as `json`, `time`, `torch`, `benchmark_cpp_extension`.
- CN: 处理模块导入，例如 `json`, `time`, `torch`, `benchmark_cpp_extension`。
- EN: Declares or extends types including `TorchBenchmarkBase`, `used`.
- CN: 声明或扩展类型，包括 `TorchBenchmarkBase`, `used`。
- EN: Implements callable logic such as `__init__`, `_set_backward_test`, `auto_set`.
- CN: 实现可调用逻辑，例如 `__init__`, `_set_backward_test`, `auto_set`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。

### Lines 61-120
````python
 61: 
 62:     def extract_inputs_tuple(self):
 63:         self.inputs_tuple = tuple(self.inputs.values())
 64: 
 65:     @torch.jit.export
 66:     def get_inputs(self):
 67:         # Need to convert the inputs to tuple outside of JIT so that
 68:         # JIT can infer the size of the inputs.
 69:         return self.inputs_tuple
 70: 
 71:     @torch.jit.export
 72:     def forward_impl(self):
 73:         # This is to supply the inputs to the forward function which
 74:         # will be called in both the eager and JIT mode of local runs
 75:         return self.forward(*self.get_inputs())
 76: 
 77:     @torch.jit.export
 78:     def forward_consume(self, iters: int):
 79:         #  _consume is used to avoid the dead-code-elimination optimization
 80:         for _ in range(iters):
 81:             torch.ops.operator_benchmark._consume(self.forward_impl())
 82: 
 83:     def forward_impl_eager(self):
 84:         # This is to supply the inputs to the forward function which
 85:         # will be called in both the eager and compile mode of local runs
 86:         return self.forward(*self.get_inputs())
 87: 
 88:     def forward_consume_eager(self, iters: int):
 89:         # Eager version of forward_consume without decorators (compilation handled by torch.compile)
 90:         for _ in range(iters):
 91:             torch.ops.operator_benchmark._consume(self.forward_impl_eager())
 92: 
 93:     def module_name(self):
 94:         """this is used to label the operator being benchmarked"""
 95:         if self.user_given_name:
 96:             return self.user_given_name
 97:         return self.__class__.__name__
 98: 
 99:     def set_module_name(self, name):
100:         self.user_given_name = name
101: 
102:     def test_name(self, **kargs):
103:         """this is a globally unique name which can be used to
104:         label a specific test
105:         """
106: 
107:         # This is a list of attributes which will not be included
108:         # in the test name.
109:         skip_key_list = ["device"]
110: 
111:         test_name_str = []
112:         for key in kargs:
113:             value = kargs[key]
114:             test_name_str.append(
115:                 ("" if key in skip_key_list else key)
116:                 + str(value if type(value) is not bool else int(value))
117:             )
118:         name = (self.module_name() + "_" + "_".join(test_name_str)).replace(" ", "")
119:         return name
120: 
````
- EN: Implements callable logic such as `extract_inputs_tuple`, `get_inputs`, `forward_impl`, `forward_consume`.
- CN: 实现可调用逻辑，例如 `extract_inputs_tuple`, `get_inputs`, `forward_impl`, `forward_consume`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-180
````python
121:     def get_memory_traffic_bytes(self):
122:         """Return the number of bytes read/written by this operator.
123: 
124:         Override this method in subclasses for operations with non-standard memory patterns
125:         (e.g., matmul which is compute-bound rather than memory-bound).
126: 
127:         The framework will use this value along with execution time to compute
128:         and report memory bandwidth in GB/s.
129: 
130:         Default implementation assumes a pointwise-like operation:
131:         - Reads: all input tensors
132:         - Writes: output tensor (estimated as size of largest input)
133: 
134:         This default works correctly for:
135:         - Element-wise operations (add, mul, relu, etc.)
136:         - Activations (gelu, sigmoid, etc.)
137:         - Optimizers (SGD, Adam, etc.)
138:         - Reductions (sum, mean, etc. - may underestimate writes)
139: 
140:         Returns:
141:             int or None: Total bytes transferred (reads + writes), or None if not applicable
142:         """
143:         if not hasattr(self, "inputs") or not self.inputs:
144:             return None
145: 
146:         input_tensors = [v for v in self.inputs.values() if isinstance(v, torch.Tensor)]
147:         if not input_tensors:
148:             return None
149: 
150:         # Calculate total bytes read from all inputs
151:         bytes_read = sum(t.numel() * t.element_size() for t in input_tensors)
152: 
153:         # Estimate output size as the largest input (common for pointwise ops)
154:         largest_input = max(input_tensors, key=lambda t: t.numel())
155:         bytes_written = largest_input.numel() * largest_input.element_size()
156: 
157:         return bytes_read + bytes_written
158: 
159: 
160: class PyTorchOperatorTestCase:
161:     """This class includes all the information needed to benchmark an operator.
162:     op_bench: it's a user-defined class (child of TorchBenchmarkBase)
163:     which includes input and operator, .etc
164:     test_config: a namedtuple includes test_name, input_shape, tag, run_backward.
165:     When run_backward is false, the run_forward method will be executed,
166:     When run_backward is true, run_forward_eager and _output_mean will be
167:     executed to generate output. Then, run_backward will be executed.
168:     """
169: 
170:     def __init__(self, op_bench, test_config):
171:         self.test_config = test_config
172:         self.op_bench = op_bench
173:         self.place_holder_tensor = torch.ones(1)
174:         self.framework = "PyTorch"
175:         self.time_series = []
176:         self._jit_forward_graph = None
177:         self._compile_forward_graph = None
178: 
179:     def _generate_jit_forward_graph(self):
180:         """generate a graph for the forward function via scripting"""
````
- EN: Declares or extends types including `PyTorchOperatorTestCase`, `includes`.
- CN: 声明或扩展类型，包括 `PyTorchOperatorTestCase`, `includes`。
- EN: Implements callable logic such as `get_memory_traffic_bytes`, `__init__`, `_generate_jit_forward_graph`.
- CN: 实现可调用逻辑，例如 `get_memory_traffic_bytes`, `__init__`, `_generate_jit_forward_graph`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 181-240
````python
181:         scripted_op_bench = torch.jit.script(self.op_bench)
182:         return scripted_op_bench.forward_consume
183: 
184:     def _generate_compile_forward_graph(self):
185:         """generate a compiled graph for the forward function via torch.compile"""
186:         compiled_forward_consume = torch.compile(
187:             self.op_bench.forward_consume_eager, backend="inductor"
188:         )
189:         return compiled_forward_consume
190: 
191:     def run_jit_forward(self, num_runs, print_per_iter=False, cuda_sync=False):
192:         """Run the forward path of an op with JIT mode"""
193:         if self._jit_forward_graph is None:
194:             self._jit_forward_graph = self._generate_jit_forward_graph()
195:         self._jit_forward_graph(num_runs)
196: 
197:     def run_compile_forward(self, num_runs, print_per_iter=False, cuda_sync=False):
198:         """Run the forward path of an op with compile mode"""
199:         if self._compile_forward_graph is None:
200:             self._compile_forward_graph = self._generate_compile_forward_graph()
201:         self._compile_forward_graph(num_runs)
202:         if cuda_sync:
203:             torch.cuda.synchronize(torch.cuda.current_device())
204: 
205:     def _print_per_iter(self):
206:         # print last 50 values
207:         length = min(len(self.time_series), 50)
208:         for i in range(length):
209:             print(
210:                 "PyTorchObserver "
211:                 + json.dumps(
212:                     {
213:                         "type": self.test_config.test_name,
214:                         "metric": "latency",
215:                         "unit": "ms",
216:                         "value": str(self.time_series[length - i - 1]),
217:                     }
218:                 )
219:             )
220: 
221:     def run_forward(self, num_runs, print_per_iter, cuda_sync):
222:         """Run the forward path of an op with eager mode"""
223:         if print_per_iter:
224:             for _ in range(num_runs):
225:                 start_time = time.time()
226:                 self.output = self.op_bench.forward_impl_eager()
227:                 if cuda_sync:
228:                     torch.cuda.synchronize(torch.cuda.current_device())
229:                 end_time = time.time()
230:                 self.time_series.append((end_time - start_time) * 1e3)
231:         else:
232:             for _ in range(num_runs):
233:                 self.output = self.op_bench.forward_impl_eager()
234:             if cuda_sync:
235:                 torch.cuda.synchronize(torch.cuda.current_device())
236: 
237:     def _output_mean(self):
238:         """TODO (mingzhe): it is not necessary to sum up everything by myself,
239:         torch.autograd.backward do take a gradient tensor. By default, it
240:         is the same shape as your output tensor, with all 1s.
````
- EN: Implements callable logic such as `_generate_compile_forward_graph`, `run_jit_forward`, `run_compile_forward`, `_print_per_iter`.
- CN: 实现可调用逻辑，例如 `_generate_compile_forward_graph`, `run_jit_forward`, `run_compile_forward`, `_print_per_iter`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-269
````python
241:         Mathematically, it is the same as if the output is summed together.
242:         So we should be able to get ride of this method.
243:         dummy function for gradient calculation
244:         """
245:         self.mean = self.output.mean()
246: 
247:     def run_backward(self, num_runs, print_per_iter=False):
248:         """Run the backward path of an op in many iterations"""
249:         # TODO: can we use JIT here to reduce python overhead?
250:         for _ in range(num_runs):
251:             self.mean.backward(retain_graph=True)
252: 
253: 
254: def create_pytorch_op_test_case(op_bench, test_config):
255:     """This method is used to generate est. func_name is a global unique
256:     string. For PyTorch add operator with M=8, N=2, K=1, tag = long, here
257:     are the values for the members in test_case:
258:     op.module_name: add
259:     framework: PyTorch
260:     test_config: TestConfig(test_name='add_M8_N2_K1', input_config='M: 8, N: 2, K: 1',
261:         tag='long', run_backward=False)
262:     func_name: addPyTorchTestConfig(test_name='add_M8_N2_K1', input_config='M: 8, N: 2, K: 1',
263:                                     tag='long', run_backward=False)
264:     """
265:     test_case = PyTorchOperatorTestCase(op_bench, test_config)
266:     test_config = test_case.test_config
267:     op = test_case.op_bench
268:     func_name = f"{op.module_name()}{test_case.framework}{str(test_config)}"
269:     return (func_name, test_case)
````
- EN: Implements callable logic such as `run_backward`, `create_pytorch_op_test_case`.
- CN: 实现可调用逻辑，例如 `run_backward`, `create_pytorch_op_test_case`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `TorchBenchmarkBase` / 符号 `TorchBenchmarkBase`
- Symbol `__init__` / 符号 `__init__`
- Symbol `_set_backward_test` / 符号 `_set_backward_test`
- Symbol `auto_set` / 符号 `auto_set`

## Dependencies / 依赖关系
- Python imports: `json`, `time`, `torch`, `benchmark_cpp_extension`
- Python 导入: `json`, `time`, `torch`, `benchmark_cpp_extension`
