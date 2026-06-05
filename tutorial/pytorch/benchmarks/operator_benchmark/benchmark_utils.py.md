# benchmark_utils.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/operator_benchmark/benchmark_utils.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60
````python
 1: import argparse
 2: import bisect
 3: import itertools
 4: import os
 5: import random
 6: 
 7: import numpy as np
 8: 
 9: 
10: """Performance microbenchmarks's utils.
11: 
12: This module contains utilities for writing microbenchmark tests.
13: """
14: 
15: # Here are the reserved keywords in the benchmark suite
16: _reserved_keywords = {"probs", "total_samples", "tags"}
17: _supported_devices = {"cpu", "cuda"}
18: 
19: 
20: def shape_to_string(shape):
21:     return ", ".join([str(x) for x in shape])
22: 
23: 
24: def str2bool(v):
25:     if isinstance(v, bool):
26:         return v
27:     if v.lower() in ("yes", "true", "t", "y", "1"):
28:         return True
29:     elif v.lower() in ("no", "false", "f", "n", "0"):
30:         return False
31:     else:
32:         raise argparse.ArgumentTypeError("Boolean value expected.")
33: 
34: 
35: def numpy_random(dtype, *shapes):
36:     """Return a random numpy tensor of the provided dtype.
37:     Args:
38:         shapes: int or a sequence of ints to defining the shapes of the tensor
39:         dtype: use the dtypes from numpy
40:             (https://numpy.org/doc/stable/user/basics.types.html)
41:     Return:
42:         numpy tensor of dtype
43:     """
44:     # TODO: consider more complex/custom dynamic ranges for
45:     # comprehensive test coverage.
46:     return np.random.rand(*shapes).astype(dtype)
47: 
48: 
49: def set_omp_threads(num_threads):
50:     existing_value = os.environ.get("OMP_NUM_THREADS", "")
51:     if existing_value != "":
52:         print(
53:             f"Overwriting existing OMP_NUM_THREADS value: {existing_value}; Setting it to {num_threads}."
54:         )
55:     os.environ["OMP_NUM_THREADS"] = str(num_threads)
56: 
57: 
58: def set_mkl_threads(num_threads):
59:     existing_value = os.environ.get("MKL_NUM_THREADS", "")
60:     if existing_value != "":
````
- EN: Handles module imports such as `argparse`, `bisect`, `itertools`, `os`.
- CN: 处理模块导入，例如 `argparse`, `bisect`, `itertools`, `os`。
- EN: Implements callable logic such as `shape_to_string`, `str2bool`, `numpy_random`, `set_omp_threads`.
- CN: 实现可调用逻辑，例如 `shape_to_string`, `str2bool`, `numpy_random`, `set_omp_threads`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 61-120
````python
 61:         print(
 62:             f"Overwriting existing MKL_NUM_THREADS value: {existing_value}; Setting it to {num_threads}."
 63:         )
 64:     os.environ["MKL_NUM_THREADS"] = str(num_threads)
 65: 
 66: 
 67: def cross_product(*inputs):
 68:     """
 69:     Return a list of cartesian product of input iterables.
 70:     For example, cross_product(A, B) returns ((x,y) for x in A for y in B).
 71:     """
 72:     return list(itertools.product(*inputs))
 73: 
 74: 
 75: def get_n_rand_nums(min_val, max_val, n):
 76:     random.seed((1 << 32) - 1)
 77:     return random.sample(range(min_val, max_val), n)
 78: 
 79: 
 80: def generate_configs(**configs):
 81:     """
 82:     Given configs from users, we want to generate different combinations of
 83:     those configs
 84:     For example, given M = ((1, 2), N = (4, 5)) and sample_func being cross_product,
 85:     we will generate (({'M': 1}, {'N' : 4}),
 86:                       ({'M': 1}, {'N' : 5}),
 87:                       ({'M': 2}, {'N' : 4}),
 88:                       ({'M': 2}, {'N' : 5}))
 89:     """
 90:     if "sample_func" not in configs:
 91:         raise AssertionError("Missing sample_func to generate configs")
 92:     result = []
 93:     for key, values in configs.items():
 94:         if key == "sample_func":
 95:             continue
 96:         tmp_result = []
 97:         for value in values:
 98:             tmp_result.append({key: value})
 99:         result.append(tmp_result)
100: 
101:     results = configs["sample_func"](*result)
102:     return results
103: 
104: 
105: def cross_product_configs(**configs):
106:     """
107:     Given configs from users, we want to generate different combinations of
108:     those configs
109:     For example, given M = ((1, 2), N = (4, 5)),
110:     we will generate (({'M': 1}, {'N' : 4}),
111:                       ({'M': 1}, {'N' : 5}),
112:                       ({'M': 2}, {'N' : 4}),
113:                       ({'M': 2}, {'N' : 5}))
114:     """
115:     _validate(configs)
116:     configs_attrs_list = []
117:     for key, values in configs.items():
118:         tmp_results = [{key: value} for value in values]
119:         configs_attrs_list.append(tmp_results)
120: 
````
- EN: Implements callable logic such as `cross_product`, `get_n_rand_nums`, `generate_configs`, `cross_product_configs`.
- CN: 实现可调用逻辑，例如 `cross_product`, `get_n_rand_nums`, `generate_configs`, `cross_product_configs`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-180
````python
121:     # TODO(mingzhe0908) remove the conversion to list.
122:     # itertools.product produces an iterator that produces element on the fly
123:     # while converting to a list produces everything at the same time.
124:     generated_configs = list(itertools.product(*configs_attrs_list))
125:     return generated_configs
126: 
127: 
128: def _validate(configs):
129:     """Validate inputs from users."""
130:     if "device" in configs:
131:         for v in configs["device"]:
132:             if v not in _supported_devices:
133:                 raise AssertionError(
134:                     f"Device must be one of {_supported_devices}, got {v}"
135:                 )
136: 
137: 
138: def config_list(**configs):
139:     """Generate configs based on the list of input shapes.
140:     This function will take input shapes specified in a list from user. Besides
141:     that, all other parameters will be cross produced first and each of the
142:     generated list will be merged with the input shapes list.
143: 
144:     Reserved Args:
145:         attr_names(reserved): a list of names for input shapes.
146:         attrs(reserved): a list of values for each input shape.
147:         corss_product: a dictionary of attributes which will be
148:                        cross produced with the input shapes.
149:         tags(reserved): a tag used to filter inputs.
150: 
151:     Here is an example:
152:     attrs = [
153:         [1, 2],
154:         [4, 5],
155:     ],
156:     attr_names = ['M', 'N'],
157:     cross_product_configs={
158:         'device': ['cpu', 'cuda'],
159:     },
160: 
161:     we will generate [[{'M': 1}, {'N' : 2}, {'device' : 'cpu'}],
162:                       [{'M': 1}, {'N' : 2}, {'device' : 'cuda'}],
163:                       [{'M': 4}, {'N' : 5}, {'device' : 'cpu'}],
164:                       [{'M': 4}, {'N' : 5}, {'device' : 'cuda'}]]
165:     """
166:     generated_configs = []
167:     reserved_names = ["attrs", "attr_names", "tags"]
168:     if any(attr not in configs for attr in reserved_names):
169:         raise ValueError("Missing attrs in configs")
170: 
171:     _validate(configs)
172: 
173:     cross_configs = None
174:     if "cross_product_configs" in configs:
175:         cross_configs = cross_product_configs(**configs["cross_product_configs"])
176: 
177:     for inputs in configs["attrs"]:
178:         tmp_result = [
179:             {configs["attr_names"][i]: input_value}
180:             for i, input_value in enumerate(inputs)
````
- EN: Implements callable logic such as `_validate`, `config_list`.
- CN: 实现可调用逻辑，例如 `_validate`, `config_list`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 181-240
````python
181:         ]
182:         # TODO(mingzhe0908):
183:         # If multiple 'tags' were provided, do they get concat?
184:         # If a config has both ['short', 'medium'], it should match
185:         # both 'short' and 'medium' tag-filter?
186:         tmp_result.append({"tags": "_".join(configs["tags"])})
187:         if cross_configs:
188:             generated_configs += [tmp_result + list(config) for config in cross_configs]
189:         else:
190:             generated_configs.append(tmp_result)
191: 
192:     return generated_configs
193: 
194: 
195: def attr_probs(**probs):
196:     """return the inputs in a dictionary"""
197:     return probs
198: 
199: 
200: class RandomSample:
201:     def __init__(self, configs):
202:         self.saved_cum_distribution = {}
203:         self.configs = configs
204: 
205:     def _distribution_func(self, key, weights):
206:         """this is a cumulative distribution function used for random sampling inputs"""
207:         if key in self.saved_cum_distribution:
208:             return self.saved_cum_distribution[key]
209: 
210:         total = sum(weights)
211:         result = []
212:         cumsum = 0
213:         for w in weights:
214:             cumsum += w
215:             result.append(cumsum / total)
216:         self.saved_cum_distribution[key] = result
217:         return result
218: 
219:     def _random_sample(self, key, values, weights):
220:         """given values and weights, this function randomly sample values based their weights"""
221:         # TODO(mingzhe09088): cache the results to avoid recalculation overhead
222:         if len(values) != len(weights):
223:             raise AssertionError(
224:                 f"values and weights must have same length, got {len(values)} and {len(weights)}"
225:             )
226:         _distribution_func_vals = self._distribution_func(key, weights)
227:         x = random.random()
228:         idx = bisect.bisect(_distribution_func_vals, x)
229: 
230:         if idx > len(values):
231:             raise AssertionError(
232:                 f"Wrong index value is returned: idx={idx}, len(values)={len(values)}"
233:             )
234:         # Due to numerical property, the last value in cumsum could be slightly
235:         # smaller than 1, and lead to the (index == len(values)).
236:         if idx == len(values):
237:             idx -= 1
238:         return values[idx]
239: 
240:     def get_one_set_of_inputs(self):
````
- EN: Declares or extends types including `RandomSample`.
- CN: 声明或扩展类型，包括 `RandomSample`。
- EN: Implements callable logic such as `attr_probs`, `__init__`, `_distribution_func`, `_random_sample`.
- CN: 实现可调用逻辑，例如 `attr_probs`, `__init__`, `_distribution_func`, `_random_sample`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-300
````python
241:         tmp_attr_list = []
242:         for key, values in self.configs.items():
243:             if key in _reserved_keywords:
244:                 continue
245:             value = self._random_sample(key, values, self.configs["probs"][str(key)])
246:             tmp_results = {key: value}
247:             tmp_attr_list.append(tmp_results)
248:         return tmp_attr_list
249: 
250: 
251: def random_sample_configs(**configs):
252:     """
253:     This function randomly sample <total_samples> values from the given inputs based on
254:     their weights.
255:     Here is an example showing what are the expected inputs and outputs from this function:
256:     M = [1, 2],
257:     N = [4, 5],
258:     K = [7, 8],
259:     probs = attr_probs(
260:         M = [0.7, 0.2],
261:         N = [0.5, 0.2],
262:         K = [0.6, 0.2],
263:     ),
264:     total_samples=10,
265:     this function will generate
266:     [
267:         [{'K': 7}, {'M': 1}, {'N': 4}],
268:         [{'K': 7}, {'M': 2}, {'N': 5}],
269:         [{'K': 8}, {'M': 2}, {'N': 4}],
270:         ...
271:     ]
272:     Note:
273:     The probs is optional. Without them, it implies everything is 1. The probs doesn't
274:     have to reflect the actual normalized probability, the implementation will
275:     normalize it.
276:     TODO (mingzhe09088):
277:     (1):  a lambda that accepts or rejects a config as a sample. For example: for matmul
278:     with M, N, and K, this function could get rid of (M * N * K > 1e8) to filter out
279:     very slow benchmarks.
280:     (2): Make sure each sample is unique. If the number of samples are larger than the
281:     total combinations, just return the cross product. Otherwise, if the number of samples
282:     is close to the number of cross-products, it is numerical safer to generate the list
283:     that you don't want, and remove them.
284:     """
285:     if "probs" not in configs:
286:         raise ValueError(
287:             "probs is missing. Consider adding probs or using other config functions"
288:         )
289: 
290:     configs_attrs_list = []
291:     randomsample = RandomSample(configs)
292:     for i in range(configs["total_samples"]):
293:         tmp_attr_list = randomsample.get_one_set_of_inputs()
294:         tmp_attr_list.append({"tags": "_".join(configs["tags"])})
295:         configs_attrs_list.append(tmp_attr_list)
296:     return configs_attrs_list
297: 
298: 
299: def op_list(**configs):
300:     """Generate a list of ops organized in a specific format.
````
- EN: Implements callable logic such as `random_sample_configs`, `op_list`.
- CN: 实现可调用逻辑，例如 `random_sample_configs`, `op_list`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 301-360
````python
301:     It takes two parameters which are "attr_names" and "attr".
302:     attrs stores the name and function of operators.
303:     Args:
304:         configs: key-value pairs including the name and function of
305:         operators. attrs and attr_names must be present in configs.
306:     Return:
307:         a sequence of dictionaries which stores the name and function
308:         of ops in a specifal format
309:     Example:
310:     attrs = [
311:         ["abs", torch.abs],
312:         ["abs_", torch.abs_],
313:     ]
314:     attr_names = ["op_name", "op"].
315: 
316:     With those two examples,
317:     we will generate (({"op_name": "abs"}, {"op" : torch.abs}),
318:                       ({"op_name": "abs_"}, {"op" : torch.abs_}))
319:     """
320:     generated_configs = []
321:     if "attrs" not in configs:
322:         raise ValueError("Missing attrs in configs")
323:     for inputs in configs["attrs"]:
324:         tmp_result = {
325:             configs["attr_names"][i]: input_value
326:             for i, input_value in enumerate(inputs)
327:         }
328:         generated_configs.append(tmp_result)
329:     return generated_configs
330: 
331: 
332: def get_operator_range(chars_range):
333:     """Generates the characters from chars_range inclusive."""
334:     if chars_range == "None" or chars_range is None:
335:         return None
336: 
337:     if all(item not in chars_range for item in [",", "-"]):
338:         raise ValueError(
339:             "The correct format for operator_range is "
340:             "<start>-<end>, or <point>, <start>-<end>"
341:         )
342: 
343:     ops_start_chars_set = set()
344:     ranges = chars_range.split(",")
345:     for item in ranges:
346:         if len(item) == 1:
347:             ops_start_chars_set.add(item.lower())
348:             continue
349:         start, end = item.split("-")
350:         ops_start_chars_set.update(
351:             chr(c).lower() for c in range(ord(start), ord(end) + 1)
352:         )
353:     return ops_start_chars_set
354: 
355: 
356: def process_arg_list(arg_list):
357:     if arg_list == "None":
358:         return None
359: 
360:     return [fr.strip() for fr in arg_list.split(",") if len(fr.strip()) > 0]
````
- EN: Implements callable logic such as `get_operator_range`, `process_arg_list`.
- CN: 实现可调用逻辑，例如 `get_operator_range`, `process_arg_list`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `shape_to_string` / 符号 `shape_to_string`
- Symbol `str2bool` / 符号 `str2bool`
- Symbol `numpy_random` / 符号 `numpy_random`
- Symbol `set_omp_threads` / 符号 `set_omp_threads`

## Dependencies / 依赖关系
- Python imports: `argparse`, `bisect`, `itertools`, `os`, `random`, `numpy`
- Python 导入: `argparse`, `bisect`, `itertools`, `os`, `random`, `numpy`
