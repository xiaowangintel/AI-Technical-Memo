# expand.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/instruction_counts/core/expand.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60
````python
 1: """Logic for converting human-readable benchmarks into executable form.
 2: 
 3: This is mostly string manipulation, with just a bit of importlib magic.
 4: """
 5: 
 6: # mypy: ignore-errors
 7: 
 8: import importlib.abc
 9: import importlib.util
10: import itertools as it
11: import os
12: import re
13: import textwrap
14: import uuid
15: from typing import TYPE_CHECKING
16: 
17: import torch
18: 
19: 
20: if TYPE_CHECKING:
21:     # See the note in api.py for why this is necessary.
22:     from torch.utils.benchmark.utils.timer import Language
23: else:
24:     from torch.utils.benchmark import Language
25: 
26: from core.api import AutogradMode, AutoLabels, GroupedBenchmark, RuntimeMode, TimerArgs
27: from core.types import FlatDefinition, FlatIntermediateDefinition, Label
28: from core.utils import get_temp_dir
29: 
30: 
31: _ALL_MODES = tuple(
32:     it.product(
33:         RuntimeMode,
34:         AutogradMode,
35:         Language,
36:     )
37: )
38: 
39: 
40: def _generate_torchscript_file(model_src: str, name: str) -> str | None:
41:     """Returns the path a saved model if one can be constructed from `spec`.
42: 
43:     Because TorchScript requires actual source code in order to script a
44:     model, we can't simply `eval` an appropriate model string. Instead, we
45:     must write the correct source to a temporary Python file and then import
46:     the TorchScript model from that temporary file.
47: 
48:     `model_src` must contain `jit_model = ...`, which `materialize` will supply.
49:     """
50:     # Double check.
51:     if "jit_model = " not in model_src:
52:         raise AssertionError(f"Missing jit_model definition:\n{model_src}")
53: 
54:     # `torch.utils.benchmark.Timer` will automatically import torch, so we
55:     # need to match that convention.
56:     model_src = f"import torch\n{model_src}"
57: 
58:     model_root = os.path.join(get_temp_dir(), "TorchScript_models")
59:     os.makedirs(model_root, exist_ok=True)
60:     module_path = os.path.join(model_root, f"torchscript_{name}.py")
````
- EN: Handles module imports such as `importlib.abc`, `importlib.util`, `itertools`, `os`.
- CN: 处理模块导入，例如 `importlib.abc`, `importlib.util`, `itertools`, `os`。
- EN: Implements callable logic such as `_generate_torchscript_file`.
- CN: 实现可调用逻辑，例如 `_generate_torchscript_file`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 61-120
````python
 61:     artifact_path = os.path.join(model_root, f"torchscript_{name}.pt")
 62: 
 63:     if os.path.exists(module_path):
 64:         # The uuid in `name` should protect against this, but it doesn't hurt
 65:         # to confirm.
 66:         raise ValueError(f"File {module_path} already exists.")
 67: 
 68:     with open(module_path, "w") as f:
 69:         f.write(model_src)
 70: 
 71:     # Import magic to actually load our function.
 72:     module_spec = importlib.util.spec_from_file_location(
 73:         f"torchscript__{name}", module_path
 74:     )
 75:     if module_spec is None:
 76:         raise AssertionError(f"Failed to create module spec for {module_path}")
 77:     module = importlib.util.module_from_spec(module_spec)
 78:     loader = module_spec.loader
 79:     if loader is None:
 80:         raise AssertionError(f"Module spec has no loader for {module_path}")
 81: 
 82:     loader.exec_module(module)
 83: 
 84:     # And again, the type checker has no way of knowing that this line is valid.
 85:     jit_model = module.jit_model  # type: ignore[attr-defined]
 86:     if not isinstance(jit_model, (torch.jit.ScriptFunction, torch.jit.ScriptModule)):
 87:         raise AssertionError(
 88:             f"Expected ScriptFunction or ScriptModule, got: {type(jit_model)}"
 89:         )
 90:     jit_model.save(artifact_path)  # type: ignore[call-arg]
 91: 
 92:     # Cleanup now that we have the actual serialized model.
 93:     os.remove(module_path)
 94:     return artifact_path
 95: 
 96: 
 97: def _get_stmt(
 98:     benchmark: GroupedBenchmark,
 99:     runtime: RuntimeMode,
100:     autograd: AutogradMode,
101:     language: Language,
102: ) -> str | None:
103:     """Specialize a GroupedBenchmark for a particular configuration."""
104:     is_python = language == Language.PYTHON
105: 
106:     # During GroupedBenchmark construction, py_fwd_stmt and cpp_fwd_stmt are
107:     # set to the eager invocation. So in the RuntimeMode.EAGER case we can
108:     # simply reuse them. For the RuntimeMode.JIT case, we need to generate
109:     # an appropriate `jit_model(...)` invocation.
110:     if runtime == RuntimeMode.EAGER:
111:         stmts = (benchmark.py_fwd_stmt, benchmark.cpp_fwd_stmt)
112: 
113:     else:
114:         if runtime != RuntimeMode.JIT:
115:             raise AssertionError(f"Expected RuntimeMode.JIT, but got {runtime}")
116:         if benchmark.signature_args is None:
117:             raise AssertionError(
118:                 "benchmark.signature_args must not be None for JIT mode"
119:             )
120:         stmts = GroupedBenchmark._make_model_invocation(
````
- EN: Implements callable logic such as `_get_stmt`.
- CN: 实现可调用逻辑，例如 `_get_stmt`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-180
````python
121:             benchmark.signature_args, benchmark.signature_output, RuntimeMode.JIT
122:         )
123: 
124:     stmt = stmts[0 if is_python else 1]
125: 
126:     if autograd == AutogradMode.FORWARD_BACKWARD and stmt is not None:
127:         if benchmark.signature_output is None:
128:             raise AssertionError(
129:                 "benchmark.signature_output must not be None for FORWARD_BACKWARD mode"
130:             )
131:         backward = (
132:             f"{benchmark.signature_output}"
133:             # In C++ we have to get the Tensor out of the IValue to call `.backward()`
134:             f"{'.toTensor()' if runtime == RuntimeMode.JIT and language == Language.CPP else ''}"
135:             f".backward(){';' if language == Language.CPP else ''}"
136:         )
137:         stmt = f"{stmt}\n{backward}"
138:     return stmt
139: 
140: 
141: def _get_setup(
142:     benchmark: GroupedBenchmark,
143:     runtime: RuntimeMode,
144:     language: Language,
145:     stmt: str,
146:     model_path: str | None,
147: ) -> str:
148:     """Specialize a GroupedBenchmark for a particular configuration.
149: 
150:     Setup requires two extra pieces of information:
151:       1) The benchmark stmt. This is needed to warm up the model and avoid
152:          measuring lazy initialization.
153:       2) The model path so we can load it during the benchmark.
154: 
155:     These are only used when `runtime == RuntimeMode.JIT`.
156:     """
157: 
158:     # By the time we get here, details about how to set up a model have already
159:     # been determined by GroupedBenchmark. (Or set to None if appropriate.) We
160:     # simply need to collect and package the code blocks.
161:     if language == Language.PYTHON:
162:         setup = benchmark.setup.py_setup
163:         model_setup = benchmark.py_model_setup
164:     else:
165:         if language != Language.CPP:
166:             raise AssertionError(f"Expected Language.CPP, but got {language}")
167:         setup = benchmark.setup.cpp_setup
168:         model_setup = benchmark.cpp_model_setup
169: 
170:     if runtime == RuntimeMode.EAGER:
171:         return "\n".join([setup, model_setup or ""])
172: 
173:     if runtime != RuntimeMode.JIT:
174:         raise AssertionError(f"Expected RuntimeMode.JIT, but got {runtime}")
175:     if model_path is None:
176:         raise AssertionError("model_path must not be None for JIT mode")
177: 
178:     # We template `"{model_path}"`, so quotes would break model loading. The
179:     # model path is generated within the benchmark, so this is just an
180:     # abundance of caution rather than something that is expected in practice.
````
- EN: Implements callable logic such as `_get_setup`.
- CN: 实现可调用逻辑，例如 `_get_setup`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 181-240
````python
181:     if '"' in model_path:
182:         raise AssertionError(f"model_path contains quotes: {model_path}")
183: 
184:     # `stmt` may contain newlines, so we can't use f-strings. Instead we need
185:     # to generate templates so that dedent works properly.
186:     if language == Language.PYTHON:
187:         setup_template: str = textwrap.dedent(
188:             f"""
189:             jit_model = torch.jit.load("{model_path}")
190: 
191:             # Warmup `jit_model`
192:             for _ in range(3):
193:             {{stmt}}
194:         """
195:         )
196: 
197:     else:
198:         if language != Language.CPP:
199:             raise AssertionError(f"Expected Language.CPP, but got {language}")
200:         setup_template = textwrap.dedent(
201:             f"""
202:             const std::string fpath = "{model_path}";
203:             auto jit_model = torch::jit::load(fpath);
204: 
205:             // Warmup `jit_model`
206:             for (int i = 0; i < 3; i++) {{{{
207:             {{stmt}}
208:             }}}}
209:         """
210:         )
211: 
212:     model_load = setup_template.format(stmt=textwrap.indent(stmt, " " * 4))
213:     return "\n".join([setup, model_load])
214: 
215: 
216: def materialize(benchmarks: FlatIntermediateDefinition) -> FlatDefinition:
217:     """Convert a heterogeneous benchmark into an executable state.
218: 
219:     This entails generation of TorchScript model artifacts, splitting
220:     GroupedBenchmarks into multiple TimerArgs, and tagging the results with
221:     AutoLabels.
222:     """
223:     results: list[tuple[Label, AutoLabels, TimerArgs]] = []
224: 
225:     for label, args in benchmarks.items():
226:         if isinstance(args, TimerArgs):
227:             # User provided an explicit TimerArgs, so no processing is necessary.
228:             auto_labels = AutoLabels(
229:                 RuntimeMode.EXPLICIT, AutogradMode.EXPLICIT, args.language
230:             )
231:             results.append((label, auto_labels, args))
232: 
233:         else:
234:             if not isinstance(args, GroupedBenchmark):
235:                 raise AssertionError(f"Expected GroupedBenchmark, but got {type(args)}")
236: 
237:             model_path: str | None = None
238:             if args.py_model_setup and args.torchscript:
239:                 model_setup = (
240:                     f"{args.py_model_setup}\njit_model = torch.jit.script(model)"
````
- EN: Implements callable logic such as `materialize`.
- CN: 实现可调用逻辑，例如 `materialize`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-289
````python
241:                 )
242: 
243:                 # This is just for debugging. We just need a unique name for the
244:                 # model, but embedding the label makes debugging easier.
245:                 name: str = re.sub(r"[^a-z0-9_]", "_", "_".join(label).lower())
246:                 name = f"{name}_{uuid.uuid4()}"
247: 
248:                 model_path = _generate_torchscript_file(model_setup, name=name)
249: 
250:             for (runtime, autograd, language), num_threads in it.product(
251:                 _ALL_MODES, args.num_threads
252:             ):
253:                 if runtime == RuntimeMode.EXPLICIT or autograd == AutogradMode.EXPLICIT:
254:                     continue
255: 
256:                 if runtime == RuntimeMode.JIT and not args.torchscript:
257:                     continue
258: 
259:                 if autograd == AutogradMode.FORWARD_BACKWARD and not args.autograd:
260:                     continue
261: 
262:                 stmt = _get_stmt(args, runtime, autograd, language)
263:                 if stmt is None:
264:                     continue
265: 
266:                 setup = _get_setup(args, runtime, language, stmt, model_path)
267: 
268:                 global_setup: str = ""
269:                 if language == Language.CPP and runtime == RuntimeMode.JIT:
270:                     global_setup = textwrap.dedent(
271:                         """
272:                         #include <string>
273:                         #include <vector>
274:                         #include <torch/script.h>
275:                     """
276:                     )
277: 
278:                 autolabels = AutoLabels(runtime, autograd, language)
279:                 timer_args = TimerArgs(
280:                     stmt=stmt,
281:                     setup=setup,
282:                     global_setup=global_setup,
283:                     num_threads=num_threads,
284:                     language=language,
285:                 )
286: 
287:                 results.append((label, autolabels, timer_args))
288: 
289:     return tuple(results)
````
- EN: Pulls in native headers such as `string`, `vector`, `torch/script.h`.
- CN: 引入原生头文件，例如 `string`, `vector`, `torch/script.h`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `_generate_torchscript_file` / 符号 `_generate_torchscript_file`
- Symbol `_get_stmt` / 符号 `_get_stmt`
- Symbol `_get_setup` / 符号 `_get_setup`
- Symbol `materialize` / 符号 `materialize`

## Dependencies / 依赖关系
- Python imports: `importlib.abc`, `importlib.util`, `itertools`, `os`, `re`, `textwrap`, `uuid`, `typing`, `torch`, `torch.utils.benchmark.utils.timer`
- Python 导入: `importlib.abc`, `importlib.util`, `itertools`, `os`, `re`, `textwrap`, `uuid`, `typing`, `torch`, `torch.utils.benchmark.utils.timer`
