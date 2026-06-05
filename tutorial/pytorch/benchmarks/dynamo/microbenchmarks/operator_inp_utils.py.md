# operator_inp_utils.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/dynamo/microbenchmarks/operator_inp_utils.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60
````python
 1: import functools
 2: import logging
 3: import math
 4: import os
 5: from collections import Counter, defaultdict
 6: from collections.abc import Generator, Iterable
 7: from functools import partial
 8: from typing import Any
 9: 
10: import torch
11: from torch.testing import make_tensor
12: from torch.utils import _pytree as pytree
13: from torch.utils._python_dispatch import TorchDispatchMode
14: from torch.utils._pytree import tree_map
15: 
16: 
17: log = logging.getLogger(__name__)
18: 
19: OP_INP_DIRECTORY = os.path.join(os.path.dirname(__file__), "operator_inp_logs")
20: 
21: TIMM_DIR = os.path.join(OP_INP_DIRECTORY, "timm_train")
22: HF_DIR = os.path.join(OP_INP_DIRECTORY, "hf_train")
23: TORCHBENCH_DIR = os.path.join(OP_INP_DIRECTORY, "torchbench_train")
24: 
25: aten = torch.ops.aten
26: tensor_type = torch._C.TensorType.get()
27: 
28: dtype_abbrs = {
29:     torch.bfloat16: "bf16",
30:     torch.float64: "f64",
31:     torch.float32: "f32",
32:     torch.float16: "f16",
33:     torch.complex32: "c32",
34:     torch.complex64: "c64",
35:     torch.complex128: "c128",
36:     torch.int8: "i8",
37:     torch.int16: "i16",
38:     torch.int32: "i32",
39:     torch.int64: "i64",
40:     torch.bool: "b8",
41:     torch.uint8: "u8",
42: }
43: 
44: dtype_abbrs_parsing = {value: key for key, value in dtype_abbrs.items()}
45: 
46: 
47: def truncate_inp(arg):
48:     if arg in dtype_abbrs:
49:         return dtype_abbrs[arg]
50:     elif isinstance(arg, torch.device):
51:         return arg.type
52:     else:
53:         return arg
54: 
55: 
56: # Serialize Function Call
57: class FuncCallWrapper:
58:     def __init__(self, call, *args, **kwargs):
59:         self.call = call
60:         self.args = tree_map(truncate_inp, args)
````
- EN: Handles module imports such as `functools`, `logging`, `math`, `os`.
- CN: 处理模块导入，例如 `functools`, `logging`, `math`, `os`。
- EN: Declares or extends types including `FuncCallWrapper`.
- CN: 声明或扩展类型，包括 `FuncCallWrapper`。
- EN: Implements callable logic such as `truncate_inp`, `__init__`.
- CN: 实现可调用逻辑，例如 `truncate_inp`, `__init__`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 61-120
````python
 61:         self.kwargs = tree_map(truncate_inp, kwargs) if kwargs is not None else {}
 62: 
 63:     def __repr__(self):
 64:         args = ", ".join([repr(arg) for arg in self.args])
 65:         kwargs = "".join(
 66:             [f", {str(key)}={value}" for key, value in self.kwargs.items()]
 67:         )
 68:         out = f"{self.call}({args}{kwargs})".strip('"')
 69:         # f strings introduce quotations we dont want
 70:         for key in dtype_abbrs_parsing:
 71:             out = out.replace(f"'{key}'", key)
 72:         return out
 73: 
 74: 
 75: def serialize_sparse_tensor(e):
 76:     if isinstance(e, torch._subclasses.FakeTensor):
 77:         return FuncCallWrapper("ST", list(e.shape), e.dtype, e.layout, e.is_coalesced())
 78:     else:
 79:         return FuncCallWrapper(
 80:             "ST", list(e.shape), e.dtype, e.layout, e.is_coalesced(), e._nnz()
 81:         )
 82: 
 83: 
 84: def deserialize_sparse_tensor(size, dtype, layout, is_coalesced, nnz=None):
 85:     raise NotImplementedError
 86: 
 87: 
 88: def deserialize_tensor(size, dtype, stride=None):
 89:     if stride is not None:
 90:         out = torch.empty_strided(size, stride, dtype=dtype)
 91:     else:
 92:         out = torch.empty(size, dtype=dtype)
 93:     try:
 94:         out.copy_(make_tensor(size, dtype=dtype, device="cpu"))
 95:     except Exception as e:
 96:         print(e)
 97:         return out
 98:     return out
 99: 
100: 
101: def serialize_tensor(e):
102:     if not e.is_contiguous():
103:         return FuncCallWrapper("T", list(e.shape), e.dtype, stride=e.stride())
104:     else:
105:         return FuncCallWrapper("T", list(e.shape), e.dtype)
106: 
107: 
108: def serialize_torch_args(e):
109:     if isinstance(e, torch.Tensor):
110:         if e.is_sparse:
111:             return serialize_sparse_tensor(e)
112:         return serialize_tensor(e)
113:     else:
114:         return truncate_inp(e)
115: 
116: 
117: def contains_tensor(elems):
118:     for elem in pytree.tree_leaves(elems):
119:         if isinstance(elem, torch.Tensor):
120:             return True
````
- EN: Implements callable logic such as `__repr__`, `serialize_sparse_tensor`, `deserialize_sparse_tensor`, `deserialize_tensor`.
- CN: 实现可调用逻辑，例如 `__repr__`, `serialize_sparse_tensor`, `deserialize_sparse_tensor`, `deserialize_tensor`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-180
````python
121:     return False
122: 
123: 
124: def skip_args(elems):
125:     for i in pytree.tree_leaves(elems):
126:         # only shows up in constructors and ops like that
127:         if isinstance(i, (torch.memory_format, torch.storage.UntypedStorage)):
128:             return True
129:     return False
130: 
131: 
132: def contains_tensor_types(type):
133:     return type.isSubtypeOf(tensor_type) or any(
134:         contains_tensor_types(e) for e in type.containedTypes()
135:     )
136: 
137: 
138: @functools.cache
139: def non_compute_operator(op):
140:     schema = op._schema
141: 
142:     # skip constructors
143:     if not any(contains_tensor_types(arg.type) for arg in schema.arguments):
144:         return True
145:     if "_like" in op.name():
146:         return True
147: 
148:     # allow in place writes
149:     if schema.is_mutable:
150:         return False
151: 
152:     tensor_inps = [arg for arg in schema.arguments if arg.type is tensor_type]
153:     tensor_outputs = [ret for ret in schema.returns if ret.type is tensor_type]
154: 
155:     # skip aliasing unless there are multiple outputs
156:     if len(tensor_outputs) != 1:
157:         return False
158: 
159:     for inp in tensor_inps:
160:         if inp.alias_info and tensor_outputs[0].alias_info:
161:             if inp.alias_info.before_set.intersection(
162:                 tensor_outputs[0].alias_info.after_set
163:             ):
164:                 return True
165: 
166:     return False
167: 
168: 
169: class OperatorInputsMode(TorchDispatchMode):
170:     def __init__(self, func_db=None):
171:         self.func_db = defaultdict(Counter) if func_db is None else func_db
172: 
173:     def __torch_dispatch__(self, func_overload, types, args=(), kwargs=None):
174:         kwargs = kwargs if kwargs else {}
175:         arg_meta, kwarg_meta = tree_map(serialize_torch_args, (args, kwargs))
176: 
177:         out = func_overload(*args, **kwargs)
178: 
179:         inps = (args, kwargs)
180:         if contains_tensor(inps) and not skip_args(inps) and contains_tensor(out):
````
- EN: Declares or extends types including `OperatorInputsMode`.
- CN: 声明或扩展类型，包括 `OperatorInputsMode`。
- EN: Implements callable logic such as `skip_args`, `contains_tensor_types`, `non_compute_operator`, `__init__`.
- CN: 实现可调用逻辑，例如 `skip_args`, `contains_tensor_types`, `non_compute_operator`, `__init__`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 181-240
````python
181:             serialized_str = repr((arg_meta, kwarg_meta))
182:             self.func_db[str(func_overload)][serialized_str] += 1
183: 
184:         return out
185: 
186:     def log_to_file(self, output_filename, *, skip_non_compute_operators=True):
187:         sorted_operators = sorted(self.func_db.keys())
188:         with open(output_filename, "w") as f:
189:             for operator in sorted_operators:
190:                 if skip_non_compute_operators and non_compute_operator(eval(operator)):
191:                     continue
192:                 f.write(f"Operator: {operator}\n")
193:                 operator_inputs = self.func_db[operator]
194:                 for inps, count in operator_inputs.items():
195:                     f.write(f"cnt: {count}, ")
196:                     # repr will add quotation marks around the dtype strings
197:                     for dtype_abbr in dtype_abbrs.values():
198:                         inps = inps.replace("'" + dtype_abbr + "'", dtype_abbr)
199:                     f.write(inps)
200:                     f.write("\n")
201: 
202: 
203: def map_to_device(e, device):
204:     if isinstance(e, torch.Tensor):
205:         return e.to(device)
206:     elif isinstance(e, torch.device):
207:         return device
208:     elif isinstance(e, str):
209:         if e == "cuda" or e == "cpu":
210:             return device.type
211:     else:
212:         return e
213: 
214: 
215: def map_to_dtype(e, dtype):
216:     if isinstance(e, torch.Tensor) and e.is_floating_point():
217:         return e.to(dtype)
218:     elif isinstance(e, torch.dtype):
219:         return dtype
220:     else:
221:         return e
222: 
223: 
224: def deserialize_args(inps):
225:     inps = inps.strip().strip("'")
226:     global_vals = {
227:         "T": deserialize_tensor,
228:         "ST": deserialize_sparse_tensor,
229:         "th": torch,
230:         "inf": math.inf,
231:         "torch": torch,
232:         **dtype_abbrs_parsing,
233:     }
234:     # f strings introduce quotations we dont want
235:     for key in dtype_abbrs_parsing:
236:         inps = inps.replace(f"'{key}'", key)
237:     return eval(inps.strip().strip("'").strip('"'), global_vals)
238: 
239: 
240: class OperatorInputsLoader:
````
- EN: Declares or extends types including `OperatorInputsLoader`.
- CN: 声明或扩展类型，包括 `OperatorInputsLoader`。
- EN: Implements callable logic such as `log_to_file`, `map_to_device`, `map_to_dtype`, `deserialize_args`.
- CN: 实现可调用逻辑，例如 `log_to_file`, `map_to_device`, `map_to_dtype`, `deserialize_args`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-300
````python
241:     def __init__(self, json_file_path):
242:         self.operator_db = defaultdict(Counter)
243: 
244:         with open(json_file_path) as f:
245:             lines = f.readlines()
246: 
247:         i = 0
248:         while i < len(lines):
249:             op_line = lines[i].strip("\n")
250:             if "Operator: " not in op_line:
251:                 raise AssertionError(
252:                     f"Expected 'Operator: ' in line, but got: {op_line}"
253:                 )
254:             operator = op_line[len("Operator: ") :]
255:             operator = (
256:                 operator if operator != "aten.sum.SymInt" else "aten.sum.dim_IntList"
257:             )
258:             op_inps = Counter()
259:             i += 1
260:             while i < len(lines) and "Operator: " not in lines[i]:
261:                 line = lines[i]
262:                 cnt = eval(line[len("cnt: ") : line.find(",")])
263:                 inps = line[line.find(",") + 2 :].strip("'")
264:                 op_inps[inps] += cnt
265:                 i += 1
266:             self.operator_db[operator] = op_inps
267: 
268:     def get_inputs_for_operator(
269:         self, operator, dtype=None, device="cuda"
270:     ) -> Generator[tuple[Iterable[Any], dict[str, Any]], None, None]:
271:         if str(operator) not in self.operator_db:
272:             raise AssertionError(f"Could not find {operator}, must provide overload")
273: 
274:         if "embedding" in str(operator):
275:             log.warning("Embedding inputs NYI, input data cannot be randomized")
276:             yield
277:             return
278: 
279:         # line[1] represents number of times these inputs occurred, ignored for now
280:         for line in self.operator_db[str(operator)].items():
281:             inps = line[0]
282: 
283:             args, kwargs = deserialize_args(inps)
284: 
285:             # Backwards require some inputs to be float16 and some to be float32
286:             # So we record on half and upcast to float when specified
287:             if dtype and dtype != torch.float16:
288:                 to_dtype = partial(map_to_dtype, dtype=dtype)
289:                 args, kwargs = tree_map(to_dtype, (args, kwargs))
290: 
291:             if device:
292:                 to_device = partial(map_to_device, device=torch.device(device))
293:                 args, kwargs = tree_map(to_device, (args, kwargs))
294: 
295:             yield args, kwargs
296: 
297:     def get_all_ops(self):
298:         for key in self.operator_db:
299:             try:
300:                 op = eval(key)
````
- EN: Implements callable logic such as `__init__`, `get_inputs_for_operator`, `get_all_ops`.
- CN: 实现可调用逻辑，例如 `__init__`, `get_inputs_for_operator`, `get_all_ops`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 301-345
````python
301:             except AttributeError:
302:                 log.warning("Evaluating an op name into an OpOverload", exc_info=True)
303:                 continue
304:             yield op
305: 
306:     def get_call_frequency(self, op):
307:         if str(op) not in self.operator_db:
308:             raise AssertionError(f"Could not find {op}, must provide overload")
309: 
310:         count = 0
311:         for counter in self.operator_db[str(op)].values():
312:             count += counter
313:         return count
314: 
315:     def merge(self, other):
316:         for operator, counter_dict in other.operator_db.items():
317:             for inps, cnt in counter_dict.items():
318:                 self.operator_db[operator][inps] += cnt
319: 
320:     @staticmethod
321:     def get_timm_loader():
322:         return OperatorInputsLoader._load_directory(TIMM_DIR)
323: 
324:     @staticmethod
325:     def get_huggingface_loader():
326:         return OperatorInputsLoader._load_directory(HF_DIR)
327: 
328:     @staticmethod
329:     def get_torchbench_loader():
330:         return OperatorInputsLoader._load_directory(TORCHBENCH_DIR)
331: 
332:     @staticmethod
333:     def _load_directory(inp_dir):
334:         if not os.path.isdir(inp_dir):
335:             raise AssertionError(f"Directory does not exist: {inp_dir}")
336:         union = None
337:         for inp in os.listdir(inp_dir):
338:             if inp[-4:] != ".txt":
339:                 continue
340:             path = os.path.join(inp_dir, inp)
341:             if union is None:
342:                 union = OperatorInputsLoader(path)
343:             else:
344:                 union.merge(OperatorInputsLoader(path))
345:         return union
````
- EN: Implements callable logic such as `get_call_frequency`, `merge`, `get_timm_loader`, `get_huggingface_loader`.
- CN: 实现可调用逻辑，例如 `get_call_frequency`, `merge`, `get_timm_loader`, `get_huggingface_loader`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `truncate_inp` / 符号 `truncate_inp`
- Symbol `FuncCallWrapper` / 符号 `FuncCallWrapper`
- Symbol `__init__` / 符号 `__init__`
- Symbol `__repr__` / 符号 `__repr__`

## Dependencies / 依赖关系
- Python imports: `functools`, `logging`, `math`, `os`, `collections`, `collections.abc`, `typing`, `torch`, `torch.testing`, `torch.utils`
- Python 导入: `functools`, `logging`, `math`, `os`, `collections`, `collections.abc`, `typing`, `torch`, `torch.testing`, `torch.utils`
