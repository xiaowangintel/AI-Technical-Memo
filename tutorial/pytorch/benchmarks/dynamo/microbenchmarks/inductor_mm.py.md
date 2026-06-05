# inductor_mm.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/dynamo/microbenchmarks/inductor_mm.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40
````python
 1: import triton
 2: from benchmark_helper import time_with_torch_timer
 3: 
 4: import torch
 5: import torch._dynamo
 6: import torch._dynamo.config
 7: import torch._inductor.config as config
 8: from torch._inductor.runtime.benchmarking import benchmarker
 9: 
10: 
11: # The flag below controls whether to allow TF32 on matmul. This flag defaults to True.
12: torch.backends.cuda.matmul.allow_tf32 = True
13: # The flag below controls whether to allow TF32 on cuDNN. This flag defaults to True.
14: torch.backends.cudnn.allow_tf32 = True
15: 
16: 
17: @torch._dynamo.optimize("inductor", nopython=True)
18: def inductor_aten_mm(a, b):
19:     return torch.mm(a, b)
20: 
21: 
22: @torch._dynamo.optimize("inductor", nopython=True)
23: def inductor_triton_mm(a, b):
24:     return torch.mm(a, b)
25: 
26: 
27: def torch_mm(a, b):
28:     return torch.mm(a, b)
29: 
30: 
31: def triton_mm(a, b):
32:     return triton.ops.matmul(a, b)
33: 
34: 
35: def test_total_time(shapes):
36:     print("shape; torch mm; triton mm; inductor aten mm; inductor triton mm")
37:     for i in range(len(shapes)):
38:         a_shape, b_shape = shapes[i]
39:         print(a_shape, "x", b_shape, end="; ")
40:         a = torch.randn(a_shape, device="cuda", dtype=torch.float16)
````
- EN: Handles module imports such as `triton`, `benchmark_helper`, `torch`, `torch._dynamo`.
- CN: 处理模块导入，例如 `triton`, `benchmark_helper`, `torch`, `torch._dynamo`。
- EN: Implements callable logic such as `inductor_aten_mm`, `inductor_triton_mm`, `torch_mm`, `triton_mm`.
- CN: 实现可调用逻辑，例如 `inductor_aten_mm`, `inductor_triton_mm`, `torch_mm`, `triton_mm`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 41-80
````python
41:         b = torch.randn(b_shape, device="cuda", dtype=a.dtype)
42: 
43:         config.triton.mm = "aten"
44:         inductor_aten_mm(a, b)
45: 
46:         config.triton.mm = "triton"
47:         inductor_triton_mm(a, b)
48: 
49:         torch_ms = time_with_torch_timer(torch_mm, (a, b)).mean * 1000
50: 
51:         triton_ms = time_with_torch_timer(triton_mm, (a, b)).mean * 1000
52: 
53:         config.triton.mm = "aten"
54:         ind_aten_ms = time_with_torch_timer(inductor_aten_mm, (a, b)).mean * 1000
55: 
56:         config.triton.mm = "triton"
57:         ind_triton_ms = time_with_torch_timer(inductor_triton_mm, (a, b)).mean * 1000
58: 
59:         print(torch_ms, triton_ms, ind_aten_ms, ind_triton_ms, sep="; ")
60: 
61:         torch._dynamo.reset()
62: 
63: 
64: def test_GPU_time(shapes):
65:     print("shape; torch mm; triton mm; inductor aten mm; inductor triton mm")
66:     for i in range(len(shapes)):
67:         a_shape, b_shape = shapes[i]
68:         print(a_shape, "x", b_shape, end="; ")
69:         a = torch.randn(a_shape, device="cuda", dtype=torch.float16)
70:         b = torch.randn(b_shape, device="cuda", dtype=a.dtype)
71: 
72:         config.triton.mm = "aten"
73:         inductor_aten_mm(a, b)
74: 
75:         config.triton.mm = "triton"
76:         inductor_triton_mm(a, b)
77: 
78:         torch_ms, _, _ = benchmarker.benchmark_gpu(lambda: torch_mm(a, b))
79:         triton_ms, _, _ = benchmarker.benchmark_gpu(lambda: triton_mm(a, b))
80:         ind_aten_ms, _, _ = benchmarker.benchmark_gpu(lambda: inductor_aten_mm(a, b))
````
- EN: Implements callable logic such as `test_GPU_time`.
- CN: 实现可调用逻辑，例如 `test_GPU_time`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 81-120
````python
 81:         ind_triton_ms, _, _ = benchmarker.benchmark_gpu(
 82:             lambda: inductor_triton_mm(a, b)
 83:         )
 84:         print(torch_ms, triton_ms, ind_aten_ms, ind_triton_ms, sep="; ")
 85: 
 86:         torch._dynamo.reset()
 87: 
 88: 
 89: if __name__ == "__main__":
 90:     shapes = [
 91:         # alexnet
 92:         ([128, 9216], [9216, 4096]),
 93:         ([128, 4096], [4096, 4096]),
 94:         ([128, 4096], [4096, 1000]),
 95:         # BERT
 96:         ([2048, 768], [768, 768]),
 97:         ([2048, 768], [768, 3072]),
 98:         ([2048, 3072], [3072, 768]),
 99:         # hf_GPT2
100:         ([1024, 768], [768, 768]),
101:         ([1024, 768], [768, 3072]),
102:         ([1024, 3072], [3072, 768]),
103:         ([1024, 768], [768, 2304]),
104:     ]
105:     print("test total time")
106:     test_total_time(shapes)
107: 
108:     print("test GPU time")
109:     test_GPU_time(shapes)
110: 
111: 
112: # Results Preview on AWS AI cluster
113: """
114: test total time
115: shape; torch mm; triton mm; inductor aten mm; inductor triton mm
116: [128, 9216] x [9216, 4096]; 0.07240759208798409; 0.10885953903198242; 0.20063146017491817; 0.20054904278367758
117: [128, 4096] x [4096, 4096]; 0.03640300128608942; 0.10960095096379519; 0.09948539081960917; 0.0996188772842288
118: [128, 4096] x [4096, 1000]; 0.02215010579675436; 0.12592008337378502; 0.031120930798351765; 0.0370654184371233
119: [2048, 768] x [768, 768]; 0.023501068353652954; 0.10804693214595318; 0.03004650119692087; 0.0276932492852211
120: [2048, 768] x [768, 3072]; 0.045639658346772194; 0.10883208829909563; 0.062736920081079; 0.06480381824076176
````
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 121-138
````python
121: [2048, 3072] x [3072, 768]; 0.054093082435429096; 0.10804777964949608; 0.08744294755160809; 0.07766005117446184
122: [1024, 768] x [768, 768]; 0.021525858901441097; 0.10909941978752613; 0.02656651195138693; 0.02683836966753006
123: [1024, 768] x [768, 3072]; 0.027319076471030712; 0.10825308971107006; 0.040118801407516; 0.039282338693737984
124: [1024, 3072] x [3072, 768]; 0.034132059663534164; 0.10594133753329515; 0.05069758277386427; 0.04572632722556591
125: [1024, 768] x [768, 2304]; 0.02529360819607973; 0.10486091021448374; 0.03724239766597748; 0.036449190229177475
126: test GPU time
127: shape; torch mm; triton mm; inductor aten mm; inductor triton mm
128: [128, 9216] x [9216, 4096]; 0.09113600105047226; 0.09011200070381165; 0.21606400609016418; 0.21606400609016418
129: [128, 4096] x [4096, 4096]; 0.053247999399900436; 0.05222399905323982; 0.1157120019197464; 0.1157120019197464
130: [128, 4096] x [4096, 1000]; 0.026623999699950218; 0.02969600073993206; 0.04710400104522705; 0.05222399905323982
131: [2048, 768] x [768, 768]; 0.02457600086927414; 0.020479999482631683; 0.04095999896526337; 0.03993599861860275
132: [2048, 768] x [768, 3072]; 0.05119999870657921; 0.05222399905323982; 0.07475200295448303; 0.07577600330114365
133: [2048, 3072] x [3072, 768]; 0.05939200147986412; 0.05222399905323982; 0.09830400347709656; 0.0870399996638298
134: [1024, 768] x [768, 768]; 0.01945599913597107; 0.016383999958634377; 0.03276799991726875; 0.03276799991726875
135: [1024, 768] x [768, 3072]; 0.03174399957060814; 0.03276799991726875; 0.053247999399900436; 0.053247999399900436
136: [1024, 3072] x [3072, 768]; 0.04403200000524521; 0.03379200026392937; 0.06860800087451935; 0.062463998794555664
137: [1024, 768] x [768, 2304]; 0.02969600073993206; 0.02969600073993206; 0.04915200173854828; 0.048128001391887665
138: """
````
- EN: This range contributes implementation details for the file goal: Implements benchmark definitions, helpers, or runners for performance measurement.
- CN: 该范围为文件目标提供实现细节：实现用于性能测量的基准定义、辅助工具或运行器。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `inductor_aten_mm` / 符号 `inductor_aten_mm`
- Symbol `inductor_triton_mm` / 符号 `inductor_triton_mm`
- Symbol `torch_mm` / 符号 `torch_mm`
- Symbol `triton_mm` / 符号 `triton_mm`

## Dependencies / 依赖关系
- Python imports: `triton`, `benchmark_helper`, `torch`, `torch._dynamo`, `torch._dynamo.config`, `torch._inductor.config`, `torch._inductor.runtime.benchmarking`
- Python 导入: `triton`, `benchmark_helper`, `torch`, `torch._dynamo`, `torch._dynamo.config`, `torch._inductor.config`, `torch._inductor.runtime.benchmarking`
