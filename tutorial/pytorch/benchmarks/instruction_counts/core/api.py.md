# api.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/instruction_counts/core/api.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60
````python
 1: """Key enums and structs used to handle data flow within the benchmark."""
 2: 
 3: # mypy: ignore-errors
 4: 
 5: import dataclasses
 6: import enum
 7: import itertools as it
 8: import re
 9: import textwrap
10: from typing import TYPE_CHECKING
11: 
12: from worker.main import WorkerTimerArgs
13: 
14: 
15: if TYPE_CHECKING:
16:     # Benchmark utils are only partially strict compliant, so MyPy won't follow
17:     # imports using the public namespace. (Due to an exclusion rule in
18:     # mypy-strict.ini)
19:     from torch.utils.benchmark.utils.timer import Language
20: else:
21:     from torch.utils.benchmark import Language
22: 
23: 
24: # Note:
25: #   WorkerTimerArgs is defined in worker.main so that the worker does not
26: #   depend on any files, including core.api. We mirror it with a public symbol
27: #   `TimerArgs` for API consistency.
28: TimerArgs = WorkerTimerArgs
29: 
30: 
31: class RuntimeMode(enum.Enum):
32:     EAGER = "Eager"
33:     JIT = "TorchScript"
34:     EXPLICIT = ""
35: 
36: 
37: class AutogradMode(enum.Enum):
38:     FORWARD = "Forward"
39:     FORWARD_BACKWARD = "Forward + Backward"
40:     EXPLICIT = ""
41: 
42: 
43: @dataclasses.dataclass(frozen=True)
44: class AutoLabels:
45:     """Labels for a TimerArgs instance which are inferred during unpacking."""
46: 
47:     runtime: RuntimeMode
48:     autograd: AutogradMode
49:     language: Language
50: 
51:     @property
52:     def as_dict(self) -> dict[str, str]:
53:         """Dict representation for CI reporting."""
54:         return {
55:             "runtime": self.runtime.value,
56:             "autograd": self.autograd.value,
57:             "language": "Python" if self.language == Language.PYTHON else "C++",
58:         }
59: 
60: 
````
- EN: Handles module imports such as `dataclasses`, `enum`, `itertools`, `re`.
- CN: 处理模块导入，例如 `dataclasses`, `enum`, `itertools`, `re`。
- EN: Declares or extends types including `import`, `RuntimeMode`, `AutogradMode`, `AutoLabels`.
- CN: 声明或扩展类型，包括 `import`, `RuntimeMode`, `AutogradMode`, `AutoLabels`。
- EN: Implements callable logic such as `as_dict`.
- CN: 实现可调用逻辑，例如 `as_dict`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。

### Lines 61-120
````python
 61: @dataclasses.dataclass(frozen=True)
 62: class GroupedSetup:
 63:     py_setup: str = ""
 64:     cpp_setup: str = ""
 65:     global_setup: str = ""
 66: 
 67:     def __post_init__(self) -> None:
 68:         for field in dataclasses.fields(self):
 69:             if field.type is not str:
 70:                 raise AssertionError(
 71:                     f"Expected field {field.name} to be str, but got {field.type}"
 72:                 )
 73:             value: str = getattr(self, field.name)
 74:             object.__setattr__(self, field.name, textwrap.dedent(value))
 75: 
 76: 
 77: @dataclasses.dataclass(frozen=True)
 78: class GroupedBenchmark:
 79:     """Base class for defining groups of benchmarks.
 80: 
 81:     Concrete interfaces:
 82:      - `core.api.GroupedStmts`     (init_from_stmts)
 83:      - `core.api.GroupedModules`   (init_from_model)
 84:      - `core.api.GroupedVariants`  (init_from_variants)
 85: 
 86:     There are a variety of dimensions along which one might wish to measure
 87:     PyTorch performance:
 88:       - Python, C++
 89:       - Eager, TorchScript
 90:       - Single threaded, multi threaded
 91:       - Training, inference
 92: 
 93:     It is useful to define them together, both for clear, concise benchmark
 94:     definition and more intelligent post processing and analysis.
 95: 
 96:     There are also two programming idioms in PyTorch. One is to write free form
 97:     code (so-called "NumPy with gradients"), and the other is to organize code
 98:     using `torch.nn.Module`s. (This is how common neural network layers are
 99:     exposed through the PyTorch API.) To support easy definition two simple
100:     initialization methods are provided:
101:      - `init_from_stmts`
102:      - `init_from_model`
103: 
104:     Those methods will document their unique constructor arguments, however
105:     most are shared and are defined here:
106:         setup: Defines how to initialize a benchmark in both Python and C++.
107:         signature:
108:             A string of the form:
109:             ```
110:                 f(a, b, ...) -> c
111:             ```
112:             For instance, if Python setup is:
113:             ```
114:                 x = torch.ones((2,), requires_grad=True)
115:                 y = torch.ones((2,))
116:             ```
117:             and the corresponding stmt is:
118:             ```
119:                 z = torch.dot(x, y)
120:             ```
````
- EN: Declares or extends types including `GroupedSetup`, `GroupedBenchmark`, `for`.
- CN: 声明或扩展类型，包括 `GroupedSetup`, `GroupedBenchmark`, `for`。
- EN: Implements callable logic such as `__post_init__`.
- CN: 实现可调用逻辑，例如 `__post_init__`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 121-180
````python
121:             Then the signature is `f(x, y) -> z`. `signature` is required any
122:             time we need to generate part of a snippet:
123:              - When calling an opaque model provided by `init_from_models`
124:              - When `torchscript=True`
125:              - When `autograd=True`
126: 
127:             If a return value is not needed (e.g. because of in place mutation)
128:             then `-> None` is valid, but a non-None return must be provided if
129:             `autograd=True`
130: 
131:         torchscript:
132:             If True, also JIT the stmt or model and generate benchmarks which
133:             call the scripted version. Requires that `signature` is defined.
134: 
135:         autograd:
136:             If True, generate both forward and forward + backward benchmarks.
137:             Requires that `signature` is defined, and return value is not None.
138: 
139:         num_threads:
140:             Maps to the Timer arg. If a tuple of ints is provided, benchmarks
141:             will be generated for each value.
142: 
143:     A third method, `init_from_variants`, is provided to define several related
144:     benchmarks at once.
145:     """
146: 
147:     # These are the stmts which are actually executed by Timer. In the case of
148:     # `GroupedStmts` (init_from_stmts) they are passed through from user args.
149:     # In the case of `GroupedModules` (init_from_model) they are generated
150:     # using `signature`. (e.g. `f(x, y) -> z` generates `z = model(x, y)`)
151:     py_fwd_stmt: str | None
152:     cpp_fwd_stmt: str | None
153: 
154:     # Code block used to define a model. `init_from_stmts` will never populate
155:     # `cpp_model_setup`, but if TorchScript is requested it will generate
156:     # `py_model_setup` using `torch.jit.script`.
157:     py_model_setup: str | None
158:     cpp_model_setup: str | None
159: 
160:     # True if this benchmark used `init_from_stmts`, otherwise False.
161:     inferred_model_setup: bool
162: 
163:     # Described above
164:     setup: GroupedSetup
165:     signature_args: tuple[str, ...] | None
166:     signature_output: str | None
167:     torchscript: bool
168:     autograd: bool
169:     num_threads: tuple[int, ...]
170: 
171:     @classmethod
172:     def init_from_stmts(
173:         cls,
174:         py_stmt: str | None = None,
175:         cpp_stmt: str | None = None,
176:         # Generic constructor arguments
177:         setup: GroupedSetup = GroupedSetup(),
178:         signature: str | None = None,
179:         torchscript: bool = False,
180:         autograd: bool = False,
````
- EN: Implements callable logic such as `init_from_stmts`.
- CN: 实现可调用逻辑，例如 `init_from_stmts`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 181-240
````python
181:         num_threads: int | tuple[int, ...] = 1,
182:     ) -> "GroupedBenchmark":
183:         """Create a set of benchmarks from free-form statements.
184: 
185:         This method of benchmark definition is analogous to Timer use, where
186:         we simply execute the provided stmts.
187:         """
188:         if py_stmt is not None:
189:             py_stmt = textwrap.dedent(py_stmt)
190: 
191:         if cpp_stmt is not None:
192:             cpp_stmt = textwrap.dedent(cpp_stmt)
193: 
194:         signature_args, signature_output = cls._parse_signature(signature)
195:         py_model_setup = (
196:             cls._model_from_py_stmt(
197:                 py_stmt=py_stmt,
198:                 signature_args=signature_args,
199:                 signature_output=signature_output,
200:             )
201:             if torchscript
202:             else None
203:         )
204: 
205:         return cls(
206:             py_fwd_stmt=py_stmt,
207:             cpp_fwd_stmt=cpp_stmt,
208:             py_model_setup=py_model_setup,
209:             cpp_model_setup=None,
210:             inferred_model_setup=True,
211:             setup=setup,
212:             signature_args=signature_args,
213:             signature_output=signature_output,
214:             torchscript=torchscript,
215:             autograd=autograd,
216:             num_threads=(num_threads,) if isinstance(num_threads, int) else num_threads,
217:         )
218: 
219:     @classmethod
220:     def init_from_model(
221:         cls,
222:         py_model_setup: str | None = None,
223:         cpp_model_setup: str | None = None,
224:         # Generic constructor arguments
225:         setup: GroupedSetup = GroupedSetup(),
226:         signature: str | None = None,
227:         torchscript: bool = False,
228:         autograd: bool = False,
229:         num_threads: int | tuple[int, ...] = 1,
230:     ) -> "GroupedBenchmark":
231:         """Create a set of benchmarks using torch.nn Modules.
232: 
233:         This method of benchmark creation takes setup code, and then calls
234:         a model rather than a free form block of code. As a result, there are
235:         two additional requirements compared to `init_from_stmts`:
236:           - `signature` must be provided.
237:           - A model (named "model") must be defined, either with `model = ...`
238:             or `def model(...): ...` in Python or `auto model = ...` in C++.
239:         """
240:         signature_args, signature_output = cls._parse_signature(signature)
````
- EN: Implements callable logic such as `init_from_model`.
- CN: 实现可调用逻辑，例如 `init_from_model`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-300
````python
241:         if signature_args is None:
242:             raise ValueError(
243:                 "signature is needed when initializing from model definitions."
244:             )
245: 
246:         return cls(
247:             *cls._make_model_invocation(
248:                 signature_args, signature_output, RuntimeMode.EAGER
249:             ),
250:             py_model_setup=py_model_setup,
251:             cpp_model_setup=cpp_model_setup,
252:             inferred_model_setup=False,
253:             setup=setup,
254:             signature_args=signature_args,
255:             signature_output=signature_output,
256:             torchscript=torchscript,
257:             autograd=autograd,
258:             num_threads=(num_threads,) if isinstance(num_threads, int) else num_threads,
259:         )
260: 
261:     @classmethod
262:     def init_from_variants(
263:         cls,
264:         py_block: str = "",
265:         cpp_block: str = "",
266:         num_threads: int | tuple[int, ...] = 1,
267:     ) -> dict[tuple[str, ...] | str | None, "GroupedBenchmark"]:
268:         py_cases, py_setup, py_global_setup = cls._parse_variants(
269:             py_block, Language.PYTHON
270:         )
271:         cpp_cases, cpp_setup, cpp_global_setup = cls._parse_variants(
272:             cpp_block, Language.CPP
273:         )
274: 
275:         if py_global_setup:
276:             raise AssertionError(
277:                 f"py_global_setup should be empty, but got: {py_global_setup}"
278:             )
279:         setup = GroupedSetup(
280:             py_setup=py_setup,
281:             cpp_setup=cpp_setup,
282:             global_setup=cpp_global_setup,
283:         )
284: 
285:         # NB: The key is actually `Tuple[str, ...]`, however MyPy gets confused
286:         #     and we use the superset `Union[Tuple[str, ...], Optional[str]` to
287:         #     match the expected signature.
288:         variants: dict[tuple[str, ...] | str | None, GroupedBenchmark] = {}
289: 
290:         seen_labels: set[str] = set()
291:         for label in it.chain(py_cases.keys(), cpp_cases.keys()):
292:             if label in seen_labels:
293:                 continue
294:             seen_labels.add(label)
295: 
296:             py_lines = py_cases.get(label, [])
297:             cpp_lines = cpp_cases.get(label, [])
298: 
299:             n_lines = max(len(py_lines), len(cpp_lines))
300:             py_lines += [""] * (n_lines - len(py_lines))
````
- EN: Implements callable logic such as `init_from_variants`.
- CN: 实现可调用逻辑，例如 `init_from_variants`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 301-360
````python
301:             cpp_lines += [""] * (n_lines - len(cpp_lines))
302:             lines = [
303:                 (py_stmt, cpp_stmt)
304:                 for py_stmt, cpp_stmt in zip(py_lines, cpp_lines)
305:                 if py_stmt or cpp_stmt
306:             ]
307: 
308:             for i, (py_stmt, cpp_stmt) in enumerate(lines):
309:                 case = (f"Case: {i:>2}",) if len(lines) > 1 else ()
310:                 variants[(label,) + case] = GroupedBenchmark.init_from_stmts(
311:                     py_stmt=py_stmt or None,
312:                     cpp_stmt=cpp_stmt or None,
313:                     setup=setup,
314:                     num_threads=num_threads,
315:                 )
316: 
317:         return variants
318: 
319:     def __post_init__(self) -> None:
320:         if self.autograd and self.signature_output is None:
321:             raise ValueError(
322:                 "An output variable must be specified when `autograd=True`."
323:             )
324: 
325:         if self.py_model_setup and "model" not in self.py_model_setup:
326:             raise ValueError(
327:                 "`py_model_setup` appears to be missing `model` definition."
328:             )
329: 
330:         if self.cpp_model_setup and "model" not in self.cpp_model_setup:
331:             raise ValueError(
332:                 "`cpp_model_setup` appears to be missing `model` definition."
333:             )
334: 
335:     # =========================================================================
336:     # == String manipulation methods ==========================================
337:     # =========================================================================
338: 
339:     @staticmethod
340:     def _parse_signature(
341:         signature: str | None,
342:     ) -> tuple[tuple[str, ...] | None, str | None]:
343:         if signature is None:
344:             return None, None
345: 
346:         match = re.search(r"^f\((.*)\) -> (.*)$", signature)
347:         if match is None:
348:             raise ValueError(f"Invalid signature: `{signature}`")
349: 
350:         args: tuple[str, ...] = tuple(match.groups()[0].split(", "))
351:         output: str = match.groups()[1].strip()
352: 
353:         if "," in output:
354:             raise ValueError(
355:                 f"Multiple return values are not currently allowed: `{output}`"
356:             )
357: 
358:         if output == "None":
359:             return args, None
360: 
````
- EN: Implements callable logic such as `__post_init__`, `_parse_signature`.
- CN: 实现可调用逻辑，例如 `__post_init__`, `_parse_signature`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 361-420
````python
361:         return args, output
362: 
363:     @staticmethod
364:     def _model_from_py_stmt(
365:         py_stmt: str | None,
366:         signature_args: tuple[str, ...] | None,
367:         signature_output: str | None,
368:     ) -> str:
369:         if py_stmt is None:
370:             raise ValueError("`py_stmt` must be defined in order to derive a model.")
371: 
372:         if signature_args is None:
373:             raise ValueError("signature is needed in order to derive a model.")
374: 
375:         return textwrap.dedent(
376:             f"""\
377:             def model({", ".join(signature_args)}):
378:             {{stmt_str}}
379:                 return {signature_output}
380:         """
381:         ).format(stmt_str=textwrap.indent(py_stmt, " " * 4))
382: 
383:     @staticmethod
384:     def _make_model_invocation(
385:         signature_args: tuple[str, ...],
386:         signature_output: str | None,
387:         runtime: RuntimeMode,
388:     ) -> tuple[str, str]:
389:         py_prefix, cpp_prefix = "", ""
390:         if signature_output is not None:
391:             py_prefix = f"{signature_output} = "
392:             cpp_prefix = f"auto {signature_output} = "
393: 
394:         if runtime == RuntimeMode.EAGER:
395:             model_name = "model"
396:             cpp_invocation = (
397:                 f"{cpp_prefix}{model_name}->forward({', '.join(signature_args)});"
398:             )
399: 
400:         else:
401:             if runtime != RuntimeMode.JIT:
402:                 raise AssertionError(f"Expected RuntimeMode.JIT, but got {runtime}")
403:             model_name = "jit_model"
404:             cpp_invocation = textwrap.dedent(
405:                 f"""\
406:                 std::vector<torch::jit::IValue> ivalue_inputs({{
407:                     {", ".join([f"torch::jit::IValue({a})" for a in signature_args])}
408:                 }});
409:                 {cpp_prefix}{model_name}.forward(ivalue_inputs);
410:             """
411:             )
412: 
413:         # NB:
414:         #   In python we invoke __call__, however C++ doesn't have an analogous
415:         #   method so we invoke `forward` instead. This means that Python
416:         #   is doing extra work (e.g. checking hooks) compared to C++; however
417:         #   because this is the default user experience that's acceptable.
418:         py_invocation = f"{py_prefix}{model_name}({', '.join(signature_args)})"
419: 
420:         return py_invocation, cpp_invocation
````
- EN: Implements callable logic such as `_model_from_py_stmt`, `model`, `_make_model_invocation`.
- CN: 实现可调用逻辑，例如 `_model_from_py_stmt`, `model`, `_make_model_invocation`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 421-454
````python
421: 
422:     @staticmethod
423:     def _parse_variants(
424:         block: str, language: Language
425:     ) -> tuple[dict[str, list[str]], str, str]:
426:         block = textwrap.dedent(block).strip()
427:         comment = "#" if language == Language.PYTHON else "//"
428:         label_pattern = f"{comment} @(.+)$"
429:         label = ""
430: 
431:         lines_by_label: dict[str, list[str]] = {"SETUP": [], "GLOBAL_SETUP": []}
432:         for line in block.splitlines(keepends=False):
433:             match = re.search(label_pattern, line.strip())
434:             if match:
435:                 label = match.groups()[0]
436:                 if label.replace(" ", "_").upper() in ("SETUP", "GLOBAL_SETUP"):
437:                     label = label.replace(" ", "_").upper()
438:                 continue
439: 
440:             lines_by_label.setdefault(label, [])
441:             if line.startswith(comment):
442:                 line = ""
443:             lines_by_label[label].append(line)
444: 
445:         setup = "\n".join(lines_by_label.pop("SETUP"))
446:         global_setup = "\n".join(lines_by_label.pop("GLOBAL_SETUP"))
447: 
448:         return lines_by_label, setup, global_setup
449: 
450: 
451: # These are the user facing APIs.
452: GroupedStmts = GroupedBenchmark.init_from_stmts
453: GroupedModules = GroupedBenchmark.init_from_model
454: GroupedVariants = GroupedBenchmark.init_from_variants
````
- EN: Implements callable logic such as `_parse_variants`.
- CN: 实现可调用逻辑，例如 `_parse_variants`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `RuntimeMode` / 符号 `RuntimeMode`
- Symbol `AutogradMode` / 符号 `AutogradMode`
- Symbol `AutoLabels` / 符号 `AutoLabels`
- Symbol `as_dict` / 符号 `as_dict`

## Dependencies / 依赖关系
- Python imports: `dataclasses`, `enum`, `itertools`, `re`, `textwrap`, `typing`, `worker.main`, `torch.utils.benchmark.utils.timer`, `torch.utils.benchmark`
- Python 导入: `dataclasses`, `enum`, `itertools`, `re`, `textwrap`, `typing`, `worker.main`, `torch.utils.benchmark.utils.timer`, `torch.utils.benchmark`
