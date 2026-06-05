# bench_mm_fusion.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/dynamo/microbenchmarks/bench_mm_fusion.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40
````python
 1: from prettytable import PrettyTable
 2: 
 3: import torch
 4: import torch._dynamo
 5: import torch._inductor.config
 6: from torch._inductor.runtime.benchmarking import benchmarker
 7: 
 8: 
 9: # torch._inductor.config.debug = True
10: torch._inductor.config.triton.dense_indexing = True
11: torch.manual_seed(0)
12: 
13: 
14: # The flag below controls whether to allow TF32 on matmul.
15: torch.backends.cuda.matmul.allow_tf32 = True
16: 
17: 
18: class Func:
19:     # mm
20:     @torch._dynamo.optimize("inductor")
21:     def mm(a, b, bias):
22:         y = torch.mm(a, b)
23:         return y
24: 
25:     # mm+bias
26:     @torch._dynamo.optimize("inductor")
27:     def mm_add(a, b, bias):
28:         y = torch.mm(a, b)
29:         return y + bias
30: 
31:     # relu(mm)
32:     @torch._dynamo.optimize("inductor")
33:     def mm_relu(a, b, bias):
34:         y = torch.mm(a, b)
35:         return torch.relu(y)
36: 
37:     # relu(mm+bias)
38:     @torch._dynamo.optimize("inductor")
39:     def mm_add_relu(a, b, bias):
40:         y = torch.mm(a, b)
````
- EN: Handles module imports such as `prettytable`, `torch`, `torch._dynamo`, `torch._inductor.config`.
- CN: 处理模块导入，例如 `prettytable`, `torch`, `torch._dynamo`, `torch._inductor.config`。
- EN: Declares or extends types including `Func`.
- CN: 声明或扩展类型，包括 `Func`。
- EN: Implements callable logic such as `mm`, `mm_add`, `mm_relu`, `mm_add_relu`.
- CN: 实现可调用逻辑，例如 `mm`, `mm_add`, `mm_relu`, `mm_add_relu`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。

### Lines 41-80
````python
41:         y += bias
42:         return torch.relu(y)
43: 
44: 
45: def bench(shape, layer_id, p, fusion_types=None):
46:     torch._logging.set_logs(inductor_metrics=True)
47:     if fusion_types is None:
48:         fusion_types = [""]
49:     dtype = torch.float16
50:     M, K = shape[0]
51:     _, N = shape[1]
52:     torch.manual_seed(0)
53:     # allocate inputs
54:     a = torch.randn(shape[0], device="cuda", dtype=dtype)
55:     b = torch.randn(shape[1], device="cuda", dtype=dtype)
56: 
57:     def tflops(ms):
58:         return M * K * N / ms * 1e-9
59: 
60:     row = [layer_id]
61:     for fusion_type in fusion_types:
62:         if fusion_type == "":
63:             fn_mm = Func.mm
64:         else:
65:             fn_mm = getattr(Func, f"mm_{fusion_type}")
66: 
67:         if "add" in fusion_type:
68:             bias = torch.randn((M, N), dtype=dtype, device="cuda")
69:         else:
70:             bias = None
71: 
72:         args = (a, b, bias)
73: 
74:         def fn():
75:             return fn_mm(*args)
76: 
77:         torch._inductor.config.triton.mm = "aten"
78:         torch_mm_ms, _, _ = benchmarker.benchmark_gpu(fn)
79:         torch._inductor.config.triton.mm = "triton"
80:         # reset to force code gen new python code
````
- EN: Implements callable logic such as `bench`, `tflops`, `fn`.
- CN: 实现可调用逻辑，例如 `bench`, `tflops`, `fn`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 81-120
````python
 81:         torch._dynamo.reset()
 82:         torch._inductor.metrics.reset()
 83:         triton_mm_ms, _, _ = benchmarker.benchmark_gpu(fn)
 84:         if torch._inductor.metrics.generated_kernel_count != 1:
 85:             raise AssertionError(
 86:                 f"Expected 1 generated kernel, but got {torch._inductor.metrics.generated_kernel_count}"
 87:             )
 88:         row.extend([tflops(torch_mm_ms), tflops(triton_mm_ms)])
 89: 
 90:     p.add_row(row)
 91:     torch._logging.set_logs()
 92: 
 93: 
 94: fusion_types = ["", "add", "relu", "add_relu"]
 95: shapes = [
 96:     # alexnet
 97:     ([128, 9216], [9216, 4096]),
 98:     ([128, 4096], [4096, 4096]),
 99:     ([128, 4096], [4096, 1000]),
100:     # BERT
101:     ([2048, 768], [768, 768]),
102:     ([2048, 768], [768, 3072]),
103:     ([2048, 3072], [3072, 768]),
104:     # hf_GPT2
105:     ([1024, 768], [768, 768]),
106:     ([1024, 768], [768, 3072]),
107:     ([1024, 3072], [3072, 768]),
108:     ([1024, 768], [768, 2304]),
109: ]
110: p = PrettyTable()
111: field_names = ["layer"]
112: for fusion_type in fusion_types:
113:     if fusion_type == "":
114:         field_names.append("torch mm")
115:         field_names.append("triton mm")
116:     else:
117:         field_names.append(f"torch mm+{fusion_type}")
118:         field_names.append(f"triton mm+{fusion_type}")
119: 
120: p.field_names = field_names
````
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 121-125
````python
121: p.float_format = ".3"
122: for id, shape in enumerate(shapes):
123:     bench(shape, id, p, fusion_types)
124: 
125: print(p)
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `Func` / 符号 `Func`
- Symbol `mm` / 符号 `mm`
- Symbol `mm_add` / 符号 `mm_add`
- Symbol `mm_relu` / 符号 `mm_relu`

## Dependencies / 依赖关系
- Python imports: `prettytable`, `torch`, `torch._dynamo`, `torch._inductor.config`, `torch._inductor.runtime.benchmarking`
- Python 导入: `prettytable`, `torch`, `torch._dynamo`, `torch._inductor.config`, `torch._inductor.runtime.benchmarking`
