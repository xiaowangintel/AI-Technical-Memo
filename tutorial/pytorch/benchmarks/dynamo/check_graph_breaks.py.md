# check_graph_breaks.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/dynamo/check_graph_breaks.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40
````python
 1: import argparse
 2: import os
 3: import sys
 4: import textwrap
 5: 
 6: import pandas as pd
 7: 
 8: 
 9: # Hack to have something similar to DISABLED_TEST. These models are flaky.
10: 
11: flaky_models = {
12:     "yolov3",
13:     "detectron2_maskrcnn_r_101_c4",
14:     "XGLMForCausalLM",  # discovered in https://github.com/pytorch/pytorch/pull/128148
15:     "detectron2_fcos_r_50_fpn",
16: }
17: 
18: 
19: def get_field(csv, model_name: str, field: str):
20:     try:
21:         return csv.loc[csv["name"] == model_name][field].item()
22:     except Exception:
23:         return None
24: 
25: 
26: def check_graph_breaks(actual_csv, expected_csv, expected_filename):
27:     failed = []
28:     improved = []
29: 
30:     if "rocm" in expected_filename:
31:         flaky_models.update(
32:             {
33:                 "alexnet",
34:                 "demucs",
35:                 "densenet121",
36:                 "detectron2_fcos_r_50_fpn",
37:                 "doctr_det_predictor",
38:                 "doctr_reco_predictor",
39:                 "levit_128",
40:                 "llava",
````
- EN: Handles module imports such as `argparse`, `os`, `sys`, `textwrap`.
- CN: 处理模块导入，例如 `argparse`, `os`, `sys`, `textwrap`。
- EN: Implements callable logic such as `get_field`, `check_graph_breaks`.
- CN: 实现可调用逻辑，例如 `get_field`, `check_graph_breaks`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 41-80
````python
41:                 "microbench_unbacked_tolist_sum",
42:                 "resnet50",
43:                 "resnet152",
44:                 "sam",
45:                 "sam_fast",
46:                 "stable_diffusion_text_encoder",
47:                 "stable_diffusion_unet",
48:                 "timm_efficientdet",
49:                 "torchrec_dlrm",
50:                 "vgg16",
51:                 # LLM
52:                 "meta-llama/Llama-3.2-1B",
53:                 "google/gemma-2-2b",
54:                 "google/gemma-3-4b-it",
55:                 "openai/whisper-tiny",
56:                 "Qwen/Qwen3-0.6B",
57:                 "mistralai/Mistral-7B-Instruct-v0.3",
58:                 "openai/gpt-oss-20b",
59:                 # Discovered after gfx950 CI enablement and rocm 7.2
60:                 "mobilenet_v3_large",
61:                 "mnasnet1_0",
62:             }
63:         )
64: 
65:     for model in actual_csv["name"]:
66:         graph_breaks = get_field(actual_csv, model, "graph_breaks")
67:         expected_graph_breaks = get_field(expected_csv, model, "graph_breaks")
68:         flaky = model in flaky_models
69: 
70:         if expected_graph_breaks is None:
71:             status = "MISSING:"
72:             improved.append(model)
73:         elif graph_breaks == expected_graph_breaks:
74:             status = "PASS_BUT_FLAKY" if flaky else "PASS"
75:             print(f"{model:34}  {status}")
76:             continue
77:         elif graph_breaks > expected_graph_breaks:
78:             if flaky:
79:                 status = "FAIL_BUT_FLAKY:"
80:             else:
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 81-120
````python
 81:                 status = "FAIL:"
 82:                 failed.append(model)
 83:         elif graph_breaks < expected_graph_breaks:
 84:             if flaky:
 85:                 status = "IMPROVED_BUT_FLAKY:"
 86:             else:
 87:                 status = "IMPROVED:"
 88:                 improved.append(model)
 89:         print(
 90:             f"{model:34}  {status:19} graph_breaks={graph_breaks}, expected={expected_graph_breaks}"
 91:         )
 92: 
 93:     msg = ""
 94:     if failed or improved:
 95:         if failed:
 96:             msg += textwrap.dedent(
 97:                 f"""
 98:             Error: {len(failed)} models have new dynamo graph breaks:
 99:                 {" ".join(failed)}
100: 
101:             """
102:             )
103:         if improved:
104:             msg += textwrap.dedent(
105:                 f"""
106:             Improvement: {len(improved)} models have fixed dynamo graph breaks:
107:                 {" ".join(improved)}
108: 
109:             """
110:             )
111:         sha = os.getenv("SHA1", "{your CI commit sha}")
112:         msg += textwrap.dedent(
113:             f"""
114:         If this change is expected, you can update `{expected_filename}` to reflect the new baseline.
115:         from pytorch/pytorch root, run
116:         `python benchmarks/dynamo/ci_expected_accuracy/update_expected.py {sha}`
117:         and then `git add` the resulting local changes to expected CSVs to your commit.
118:         """
119:         )
120:     return failed or improved, msg
````
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-139
````python
121: 
122: 
123: def main():
124:     parser = argparse.ArgumentParser()
125:     parser.add_argument("--actual", type=str, required=True)
126:     parser.add_argument("--expected", type=str, required=True)
127:     args = parser.parse_args()
128: 
129:     actual = pd.read_csv(args.actual)
130:     expected = pd.read_csv(args.expected)
131: 
132:     failed, msg = check_graph_breaks(actual, expected, args.expected)
133:     if failed:
134:         print(msg)
135:         sys.exit(1)
136: 
137: 
138: if __name__ == "__main__":
139:     main()
````
- EN: Implements callable logic such as `main`.
- CN: 实现可调用逻辑，例如 `main`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `get_field` / 符号 `get_field`
- Symbol `check_graph_breaks` / 符号 `check_graph_breaks`
- Symbol `main` / 符号 `main`

## Dependencies / 依赖关系
- Python imports: `argparse`, `os`, `sys`, `textwrap`, `pandas`
- Python 导入: `argparse`, `os`, `sys`, `textwrap`, `pandas`
