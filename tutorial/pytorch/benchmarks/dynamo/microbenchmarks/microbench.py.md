# microbench.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/dynamo/microbenchmarks/microbench.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40
````python
 1: #!/usr/bin/env python3
 2: 
 3: import argparse
 4: import inspect
 5: import sys
 6: 
 7: import numpy as np
 8: import tabulate
 9: 
10: import torch
11: import torch._inductor
12: from torch._dynamo.backends.cudagraphs import cudagraphs_inner
13: from torch._dynamo.testing import same
14: from torch._inductor.compile_fx import compile_fx
15: from torch._inductor.utils import timed
16: 
17: 
18: aten = torch.ops.aten
19: 
20: try:
21:     import test.test_torchinductor as tti
22: except ImportError:
23:     tti = None
24: 
25: 
26: def compute_speedups(args, models, example_inputs):
27:     expected = models[0](*example_inputs)
28:     for model in models[1:]:
29:         actual = model(*example_inputs)
30:         if not same(actual, expected):
31:             raise AssertionError(f"Output mismatch: diff={expected[0] - actual[0]}")
32: 
33:     timings = np.zeros((args.repeat, len(models)), np.float64)
34:     for rep in range(args.repeat):
35:         # interleave the runs to handle frequency scaling and load changes
36:         for m, model in enumerate(models):
37:             timings[rep, m] = timed(model, example_inputs)
38:     median = np.median(timings, axis=0)
39:     return (median[0] / median[1:]).tolist()
40: 
````
- EN: Handles module imports such as `argparse`, `inspect`, `sys`, `numpy`.
- CN: 处理模块导入，例如 `argparse`, `inspect`, `sys`, `numpy`。
- EN: Implements callable logic such as `compute_speedups`.
- CN: 实现可调用逻辑，例如 `compute_speedups`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 41-80
````python
41: 
42: def microbenchmark(args, model, example_inputs):
43:     compiled_fn = compile_fx(torch.fx.symbolic_trace(model), example_inputs)
44:     cudagraphs_eager = cudagraphs_inner(model, example_inputs, copy_outputs=False)
45:     cudagraphs_jit = cudagraphs_inner(
46:         torch.jit.trace(model, example_inputs), example_inputs, copy_outputs=False
47:     )
48:     return compute_speedups(
49:         args,
50:         [cudagraphs_eager, cudagraphs_jit, compiled_fn],
51:         example_inputs,
52:     )
53: 
54: 
55: class MyModel1(torch.nn.Module):
56:     def __init__(self):
57:         super().__init__()
58:         self.model = torch.nn.Sequential(
59:             torch.nn.Linear(1024, 1024),
60:             torch.nn.ReLU(),
61:         )
62: 
63:     def forward(self, input):
64:         # return (self.model(input) + 1,)
65:         return (self.model(input),)
66: 
67: 
68: class MyModel2(torch.nn.Module):
69:     def forward(self, x, y):
70:         # return x / (torch.abs(x) + 1.0),
71:         return (x + y,)
72: 
73: 
74: class MicroBenchmarks:
75:     @staticmethod
76:     def add(a, b):
77:         return (a + b,)
78: 
79:     @staticmethod
80:     def scale(x, m, d):
````
- EN: Declares or extends types including `MyModel1`, `MyModel2`, `MicroBenchmarks`.
- CN: 声明或扩展类型，包括 `MyModel1`, `MyModel2`, `MicroBenchmarks`。
- EN: Implements callable logic such as `microbenchmark`, `__init__`, `forward`, `add`.
- CN: 实现可调用逻辑，例如 `microbenchmark`, `__init__`, `forward`, `add`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 81-120
````python
 81:         return ((x - m) / torch.clip(d, 1e-4),)
 82: 
 83:     @staticmethod
 84:     def abs_norm(x):
 85:         return (x / (torch.abs(x) + 1),)
 86: 
 87:     @staticmethod
 88:     def add_relu_softmax(x, a):
 89:         return (torch.softmax(torch.relu(x + a), -1),)
 90: 
 91:     @staticmethod
 92:     def sum(a, b):
 93:         return ((a + b).sum(),)
 94: 
 95:     @staticmethod
 96:     def view(x):
 97:         return (aten.alias(x),)
 98: 
 99: 
100: def main():
101:     parser = argparse.ArgumentParser()
102:     parser.add_argument(
103:         "--filter", "-k", action="append", help="filter benchmarks with regexp"
104:     )
105:     parser.add_argument(
106:         "--exclude", "-x", action="append", help="filter benchmarks with regexp"
107:     )
108:     parser.add_argument("--devices", "-d", action="append", help="cpu or cuda")
109:     parser.add_argument("--size", "-s", action="append", help="cpu or cuda")
110:     parser.add_argument(
111:         "--repeat", "-n", type=int, default=30, help="number of timing runs"
112:     )
113:     parser.add_argument(
114:         "--threads", "-t", type=int, help="number of threads to use for eager"
115:     )
116:     parser.add_argument(
117:         "--verbose", "-v", action="store_true", help="enable verbose debug printouts"
118:     )
119:     parser.add_argument(
120:         "--nvfuser", action="store_true", help="enable nvfuser globally"
````
- EN: Implements callable logic such as `abs_norm`, `add_relu_softmax`, `sum`, `view`.
- CN: 实现可调用逻辑，例如 `abs_norm`, `add_relu_softmax`, `sum`, `view`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-160
````python
121:     )
122:     parser.add_argument("--transpose", action="store_true", help="transpose one input")
123:     parser.add_argument("--broadcast", action="store_true", help="broadcast one input")
124:     args = parser.parse_args()
125: 
126:     # defaults
127:     args.devices = args.devices or ["cpu", "cuda"]
128:     args.filter = args.filter or [r"."]
129:     args.exclude = args.exclude or [r"^$"]
130:     args.size = args.size or [64, 256, 1024, 4096, 8192]
131: 
132:     if args.nvfuser:
133:         torch._C._jit_override_can_fuse_on_cpu(False)
134:         torch._C._jit_override_can_fuse_on_gpu(False)
135:         torch._C._jit_set_texpr_fuser_enabled(False)
136:         torch._C._jit_set_nvfuser_enabled(True)
137:     else:
138:         torch._C._jit_override_can_fuse_on_cpu(torch._C._llvm_enabled())
139:         torch._C._jit_override_can_fuse_on_gpu(True)
140:         torch._C._jit_set_texpr_fuser_enabled(True)
141:         if torch.cuda.is_available():
142:             torch._C._jit_set_nvfuser_enabled(False)
143: 
144:     if args.threads:
145:         torch.set_num_threads(args.threads)
146:         torch._inductor.config.cpp.threads = args.threads
147: 
148:     if args.verbose:
149:         torch._inductor.config.debug = True
150: 
151:     torch._inductor.config.triton.autotune_pointwise = True
152: 
153:     rows = []
154:     for model in (MicroBenchmarks.sum, MicroBenchmarks.view):
155:         nargs = len(inspect.signature(model).parameters)
156:         for device in args.devices:
157:             for n in args.size:
158:                 n = int(n)
159:                 sys.stdout.write(f"{model.__name__:10} {device:4} {n:5} ")
160:                 sys.stdout.flush()
````
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 161-185
````python
161:                 inputs = [torch.rand((n, n), device=device) for _ in range(nargs)]
162:                 if args.broadcast:
163:                     inputs[-1] = torch.rand((1, n), device=device)
164:                 if args.transpose:
165:                     inputs[-1] = inputs[-1].transpose(0, 1)
166:                 result = microbenchmark(args, model, inputs)
167:                 rows.append([model.__name__, device, str(n)] + result)
168:                 print(" ".join(f"{v:.2f}x" for v in result))
169: 
170:     print(
171:         tabulate.tabulate(
172:             rows,
173:             headers=[
174:                 "model",
175:                 "dev",
176:                 "n",
177:                 "ts",
178:                 "inductor",
179:             ],
180:         )
181:     )
182: 
183: 
184: if __name__ == "__main__":
185:     main()
````
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `compute_speedups` / 符号 `compute_speedups`
- Symbol `microbenchmark` / 符号 `microbenchmark`
- Symbol `MyModel1` / 符号 `MyModel1`
- Symbol `__init__` / 符号 `__init__`

## Dependencies / 依赖关系
- Python imports: `argparse`, `inspect`, `sys`, `numpy`, `tabulate`, `torch`, `torch._inductor`, `torch._dynamo.backends.cudagraphs`, `torch._dynamo.testing`, `torch._inductor.compile_fx`
- Python 导入: `argparse`, `inspect`, `sys`, `numpy`, `tabulate`, `torch`, `torch._inductor`, `torch._dynamo.backends.cudagraphs`, `torch._dynamo.testing`, `torch._inductor.compile_fx`
