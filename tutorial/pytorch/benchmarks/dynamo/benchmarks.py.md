# benchmarks.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/dynamo/benchmarks.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40
````python
 1: #!/usr/bin/env python3
 2: 
 3: import argparse
 4: import os
 5: import sys
 6: 
 7: 
 8: # Run only this selected group of models, leave this empty to run everything
 9: TORCHBENCH_ONLY_MODELS = [
10:     m.strip() for m in os.getenv("TORCHBENCH_ONLY_MODELS", "").split(",") if m.strip()
11: ]
12: 
13: 
14: # Note - hf and timm have their own version of this, torchbench does not
15: # TODO(voz): Someday, consolidate all the files into one runner instead of a shim like this...
16: def model_names(filename: str) -> set[str]:
17:     names = set()
18:     with open(filename) as fh:
19:         lines = fh.readlines()
20:         lines = [line.rstrip() for line in lines]
21:         for line in lines:
22:             line_parts = line.split(" ")
23:             if len(line_parts) == 1:
24:                 line_parts = line.split(",")
25:             model_name = line_parts[0]
26:             if TORCHBENCH_ONLY_MODELS and model_name not in TORCHBENCH_ONLY_MODELS:
27:                 continue
28:             names.add(model_name)
29:     return names
30: 
31: 
32: TIMM_MODEL_NAMES = model_names(
33:     os.path.join(os.path.dirname(__file__), "timm_models_list.txt")
34: )
35: HF_MODELS_FILE_NAME = model_names(
36:     os.path.join(os.path.dirname(__file__), "huggingface_models_list.txt")
37: )
38: TORCHBENCH_MODELS_FILE_NAME = model_names(
39:     os.path.join(os.path.dirname(__file__), "all_torchbench_models_list.txt")
40: )
````
- EN: Handles module imports such as `argparse`, `os`, `sys`.
- CN: 处理模块导入，例如 `argparse`, `os`, `sys`。
- EN: Implements callable logic such as `model_names`.
- CN: 实现可调用逻辑，例如 `model_names`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 41-80
````python
41: 
42: # timm <> HF disjoint
43: if not TIMM_MODEL_NAMES.isdisjoint(HF_MODELS_FILE_NAME):
44:     raise AssertionError(
45:         f"TIMM and HF model names overlap: {TIMM_MODEL_NAMES & HF_MODELS_FILE_NAME}"
46:     )
47: # timm <> torch disjoint
48: if not TIMM_MODEL_NAMES.isdisjoint(TORCHBENCH_MODELS_FILE_NAME):
49:     raise AssertionError(
50:         f"TIMM and TorchBench model names overlap: {TIMM_MODEL_NAMES & TORCHBENCH_MODELS_FILE_NAME}"
51:     )
52: # torch <> hf disjoint
53: if not TORCHBENCH_MODELS_FILE_NAME.isdisjoint(HF_MODELS_FILE_NAME):
54:     raise AssertionError(
55:         f"TorchBench and HF model names overlap: {TORCHBENCH_MODELS_FILE_NAME & HF_MODELS_FILE_NAME}"
56:     )
57: 
58: 
59: def parse_args(args=None):
60:     parser = argparse.ArgumentParser()
61:     parser.add_argument(
62:         "--only",
63:         help="""Run just one model from whichever model suite it belongs to. Or
64:         specify the path and class name of the model in format like:
65:         --only=path:<MODEL_FILE_PATH>,class:<CLASS_NAME>
66: 
67:         Due to the fact that dynamo changes current working directory,
68:         the path should be an absolute path.
69: 
70:         The class should have a method get_example_inputs to return the inputs
71:         for the model. An example looks like
72:         ```
73:         class LinearModel(nn.Module):
74:             def __init__(self):
75:                 super().__init__()
76:                 self.linear = nn.Linear(10, 10)
77: 
78:             def forward(self, x):
79:                 return self.linear(x)
80: 
````
- EN: Declares or extends types including `name`, `should`, `LinearModel`.
- CN: 声明或扩展类型，包括 `name`, `should`, `LinearModel`。
- EN: Implements callable logic such as `parse_args`, `__init__`, `forward`.
- CN: 实现可调用逻辑，例如 `parse_args`, `__init__`, `forward`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 81-119
````python
 81:             def get_example_inputs(self):
 82:                 return (torch.randn(2, 10),)
 83:         ```
 84:     """,
 85:     )
 86:     return parser.parse_known_args(args)
 87: 
 88: 
 89: if __name__ == "__main__":
 90:     args, unknown = parse_args()
 91:     if args.only:
 92:         name = args.only
 93:         if name in TIMM_MODEL_NAMES:
 94:             import timm_models
 95: 
 96:             timm_models.timm_main()
 97:         elif name in HF_MODELS_FILE_NAME:
 98:             import huggingface
 99: 
100:             huggingface.huggingface_main()
101:         elif name in TORCHBENCH_MODELS_FILE_NAME:
102:             import torchbench
103: 
104:             torchbench.torchbench_main()
105:         else:
106:             print(f"Illegal model name? {name}")
107:             sys.exit(-1)
108:     else:
109:         import torchbench
110: 
111:         torchbench.torchbench_main()
112: 
113:         import huggingface
114: 
115:         huggingface.huggingface_main()
116: 
117:         import timm_models
118: 
119:         timm_models.timm_main()
````
- EN: Handles module imports such as `timm_models`, `huggingface`, `torchbench`.
- CN: 处理模块导入，例如 `timm_models`, `huggingface`, `torchbench`。
- EN: Implements callable logic such as `get_example_inputs`.
- CN: 实现可调用逻辑，例如 `get_example_inputs`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `model_names` / 符号 `model_names`
- Symbol `parse_args` / 符号 `parse_args`
- Symbol `LinearModel` / 符号 `LinearModel`
- Symbol `__init__` / 符号 `__init__`

## Dependencies / 依赖关系
- Python imports: `argparse`, `os`, `sys`, `timm_models`, `huggingface`, `torchbench`
- Python 导入: `argparse`, `os`, `sys`, `timm_models`, `huggingface`, `torchbench`
