# run_benchmarks.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/fuser/run_benchmarks.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60
````python
 1: import inspect
 2: import itertools
 3: import sys
 4: import time
 5: 
 6: import click
 7: 
 8: import torch
 9: 
10: 
11: torch.set_num_threads(1)
12: torch._C._debug_set_fusion_group_inlining(False)
13: 
14: 
15: def rand(*shape):
16:     return torch.rand(*shape).mul(16).add(1)
17: 
18: 
19: # ------------------------------------------------------------------------------
20: # Shape test cases
21: # ------------------------------------------------------------------------------
22: def scalar():
23:     return (rand(1), rand(1))
24: 
25: 
26: def small():
27:     return (rand(32), rand(32))
28: 
29: 
30: def small_2d():
31:     return (rand(1, 32), rand(1, 32))
32: 
33: 
34: def small_broadcast():
35:     return (rand(4, 32), rand(32))
36: 
37: 
38: def medium():
39:     return (rand(32, 12, 64, 64), rand(32, 12, 64, 64))
40: 
41: 
42: def medium_sliced():
43:     return (rand(32, 12, 64, 64)[..., ::2], rand(32, 12, 64, 64)[..., ::2])
44: 
45: 
46: def medium_transpose():
47:     return (
48:         rand(32, 12, 64, 64).transpose(-1, -2),
49:         rand(32, 12, 64, 64).transpose(-1, -2),
50:     )
51: 
52: 
53: def medium2():
54:     return (rand(32, 3, 224, 224), rand(32, 3, 224, 224))
55: 
56: 
57: def medium3d():
58:     return (rand(16, 32, 64), rand(16, 32, 64))
59: 
60: 
````
- EN: Handles module imports such as `inspect`, `itertools`, `sys`, `time`.
- CN: 处理模块导入，例如 `inspect`, `itertools`, `sys`, `time`。
- EN: Implements callable logic such as `rand`, `scalar`, `small`, `small_2d`.
- CN: 实现可调用逻辑，例如 `rand`, `scalar`, `small`, `small_2d`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 61-120
````python
 61: def medium_channels_last():
 62:     return (
 63:         rand(32, 3, 224, 224).to(memory_format=torch.channels_last),
 64:         rand(32, 3, 224, 224).to(memory_format=torch.channels_last),
 65:     )
 66: 
 67: 
 68: def medium_broadcast():
 69:     return (rand(32, 12, 64, 64), rand(64))
 70: 
 71: 
 72: def medium_broadcast_channels_last():
 73:     return (rand(32, 3, 223, 223).to(memory_format=torch.channels_last), rand(3, 1, 1))
 74: 
 75: 
 76: def large():
 77:     return (rand(8192, 8192), rand(8192, 8192))
 78: 
 79: 
 80: def large_transpose():
 81:     return (rand(8192, 8192).transpose(0, 1), rand(8192, 8192).transpose(0, 1))
 82: 
 83: 
 84: def large_channels_last():
 85:     return (
 86:         rand(32, 32, 256, 256).to(memory_format=torch.channels_last),
 87:         rand(32, 32, 256, 256).to(memory_format=torch.channels_last),
 88:     )
 89: 
 90: 
 91: def broadcast_narrow_57611():
 92:     return (rand(1, 32, 32, 2), rand(1024, 1, 1, 2))
 93: 
 94: 
 95: def large_broadcast_66816():
 96:     return (rand(64, 8, 256, 162), rand(256, 162))
 97: 
 98: 
 99: # ------------------------------------------------------------------------------
100: # Operator test cases
101: # ------------------------------------------------------------------------------
102: def add(a, b):
103:     return 3 * a + b
104: 
105: 
106: def sub(a, b):
107:     return 3 * a - b
108: 
109: 
110: def mul(a, b):
111:     return 3 * a * b
112: 
113: 
114: def div(a, b):
115:     return 3 * a / b
116: 
117: 
118: def relu(a):
119:     return (3 * a).relu()
120: 
````
- EN: Implements callable logic such as `medium_channels_last`, `medium_broadcast`, `medium_broadcast_channels_last`, `large`.
- CN: 实现可调用逻辑，例如 `medium_channels_last`, `medium_broadcast`, `medium_broadcast_channels_last`, `large`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-180
````python
121: 
122: def sigmoid(a):
123:     return (3 * a).sigmoid()
124: 
125: 
126: def tanh(a):
127:     return (3 * a).tanh()
128: 
129: 
130: def log(a):
131:     return (3 * a).log()
132: 
133: 
134: def exp(a):
135:     return (3 * a).exp()
136: 
137: 
138: def square(a):
139:     return (3 * a) ** 2
140: 
141: 
142: def fma(a, b):
143:     return a * b + b
144: 
145: 
146: def mul_mul_add_66816(a, b, c):
147:     return (a * b) + (a * c)
148: 
149: 
150: def hardswish_int(a):
151:     return a * (a + 3).clamp(0, 6) / 6
152: 
153: 
154: def hardswish(a):
155:     return a * (a + 3).clamp(0.0, 6.0) / 6
156: 
157: 
158: def native_hardswish(a):
159:     return torch._C._nn.hardswish(a * 3)
160: 
161: 
162: def softplus(a):
163:     return (a * 1.0).exp().log1p() / 1.0
164: 
165: 
166: def mish(a):
167:     return a * ((a * 1.0).exp().log1p() / 1.0).tanh()
168: 
169: 
170: SHAPES = [
171:     scalar,
172:     small,
173:     small_2d,
174:     small_broadcast,
175:     medium,
176:     medium2,
177:     medium3d,
178:     medium_sliced,
179:     medium_transpose,
180:     medium_channels_last,
````
- EN: Implements callable logic such as `sigmoid`, `tanh`, `log`, `exp`.
- CN: 实现可调用逻辑，例如 `sigmoid`, `tanh`, `log`, `exp`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 181-240
````python
181:     medium_broadcast,
182:     medium_broadcast_channels_last,
183:     large,
184:     large_transpose,
185:     large_channels_last,
186:     broadcast_narrow_57611,
187:     large_broadcast_66816,
188: ]
189: 
190: OPERATORS = [
191:     add,
192:     sub,
193:     mul,
194:     div,
195:     relu,
196:     sigmoid,
197:     tanh,
198:     log,
199:     exp,
200:     square,
201:     fma,
202:     mul_mul_add_66816,
203:     hardswish_int,
204:     hardswish,
205:     native_hardswish,
206:     softplus,
207:     mish,
208: ]
209: 
210: 
211: def time_cpu(fn, args, iters):
212:     s = time.perf_counter()
213:     for _ in range(iters):
214:         fn(*args)
215:     e = time.perf_counter()
216:     return e - s
217: 
218: 
219: def time_cuda(fn, args, iters):
220:     start = torch.cuda.Event(enable_timing=True)
221:     end = torch.cuda.Event(enable_timing=True)
222:     start.record()
223:     for _ in range(iters):
224:         fn(*args)
225:     end.record()
226:     torch.cuda.synchronize()
227:     return start.elapsed_time(end) / 1e3
228: 
229: 
230: def benchmark_with_timer(fn, args, timer):
231:     timer(fn, args, 3)
232:     calibration = timer(fn, args, 1)
233:     iters = int(1.0 / calibration)
234:     return timer(fn, args, iters) / iters
235: 
236: 
237: def benchmark(fn, args):
238:     timer = time_cpu if args[0].device.type == "cpu" else time_cuda
239:     return benchmark_with_timer(fn, args, timer)
240: 
````
- EN: Implements callable logic such as `time_cpu`, `time_cuda`, `benchmark_with_timer`, `benchmark`.
- CN: 实现可调用逻辑，例如 `time_cpu`, `time_cuda`, `benchmark_with_timer`, `benchmark`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-300
````python
241: 
242: def micros(s):
243:     return f"{s * 1e6:.1f}"
244: 
245: 
246: def with_nvfuser():
247:     torch._C._jit_override_can_fuse_on_cpu(False)
248:     torch._C._jit_override_can_fuse_on_gpu(False)
249:     torch._C._jit_set_texpr_fuser_enabled(False)
250:     torch._C._jit_set_nvfuser_enabled(True)
251:     torch._C._jit_set_profiling_executor(True)
252:     torch._C._jit_set_profiling_mode(True)
253: 
254: 
255: def with_nnc():
256:     torch._C._jit_override_can_fuse_on_cpu(True)
257:     torch._C._jit_override_can_fuse_on_gpu(True)
258:     torch._C._jit_set_texpr_fuser_enabled(True)
259:     torch._C._jit_set_nvfuser_enabled(False)
260:     torch._C._jit_set_profiling_executor(True)
261:     torch._C._jit_set_profiling_mode(True)
262: 
263: 
264: def with_legacy():
265:     torch._C._jit_override_can_fuse_on_cpu(True)
266:     torch._C._jit_override_can_fuse_on_gpu(True)
267:     torch._C._jit_set_texpr_fuser_enabled(False)
268:     torch._C._jit_set_nvfuser_enabled(False)
269:     torch._C._jit_set_profiling_executor(False)
270:     torch._C._jit_set_profiling_mode(False)
271: 
272: 
273: @click.command()
274: @click.option("--operators", default=None)
275: @click.option("--shapes", default=None)
276: def run_benchmarks(operators, shapes):
277:     if operators is None:
278:         operators = OPERATORS
279:     else:
280:         operators = [globals()[k] for k in operators.split(",")]
281:     if shapes is None:
282:         shapes = SHAPES
283:     else:
284:         shapes = [globals()[k] for k in shapes.split(",")]
285: 
286:     print("fuser,device,operator,shape,time")
287:     for shape, operator in itertools.product(shapes, operators):
288:         nargs = len(inspect.signature(operator).parameters)
289:         args = shape()
290:         if nargs > len(args):
291:             args = list(args)
292:             args += [args[-1]] * (nargs - len(args))
293:         args = args[:nargs]
294:         args = [arg.to("cuda") for arg in args]
295: 
296:         result = benchmark(operator, args)
297:         print(
298:             ",".join(
299:                 [
300:                     "eager",
````
- EN: Implements callable logic such as `micros`, `with_nvfuser`, `with_nnc`, `with_legacy`.
- CN: 实现可调用逻辑，例如 `micros`, `with_nvfuser`, `with_nnc`, `with_legacy`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 301-334
````python
301:                     args[0].device.type,
302:                     operator.__name__,
303:                     shape.__name__,
304:                     micros(result),
305:                 ]
306:             )
307:         )
308: 
309:         def bench(name):
310:             nnc_op = torch.jit.trace(operator, args)
311:             result = benchmark(nnc_op, args)
312:             print(
313:                 ",".join(
314:                     [
315:                         name,
316:                         args[0].device.type,
317:                         operator.__name__,
318:                         shape.__name__,
319:                         micros(result),
320:                     ]
321:                 )
322:             )
323:             sys.stdout.flush()
324: 
325:         with_nnc()
326:         bench("nnc")
327:         with_nvfuser()
328:         bench("nvfuser")
329:         with_legacy()
330:         bench("legacy")
331: 
332: 
333: if __name__ == "__main__":
334:     run_benchmarks()
````
- EN: Implements callable logic such as `bench`.
- CN: 实现可调用逻辑，例如 `bench`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `rand` / 符号 `rand`
- Symbol `scalar` / 符号 `scalar`
- Symbol `small` / 符号 `small`
- Symbol `small_2d` / 符号 `small_2d`

## Dependencies / 依赖关系
- Python imports: `inspect`, `itertools`, `sys`, `time`, `click`, `torch`
- Python 导入: `inspect`, `itertools`, `sys`, `time`, `click`, `torch`
