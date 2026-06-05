# check_accuracy.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/dynamo/check_accuracy.py`
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
15:     "moondream",  # discovered in https://github.com/pytorch/pytorch/pull/159291
16:     # discovered in https://github.com/pytorch/pytorch/issues/161419. Its not flaky but really hard to repro, so skipping it
17:     "mobilenetv3_large_100",
18:     # https://github.com/pytorch/pytorch/issues/163670
19:     "vision_maskrcnn",
20: }
21: 
22: 
23: def get_field(csv, model_name: str, field: str):
24:     try:
25:         return csv.loc[csv["name"] == model_name][field].item()
26:     except Exception:
27:         return None
28: 
29: 
30: def check_accuracy(actual_csv, expected_csv, expected_filename):
31:     failed = []
32:     improved = []
33: 
34:     if "rocm" in expected_filename:
35:         flaky_models.update(
36:             {
37:                 "Background_Matting",
38:                 "mnasnet1_0",
39:                 "llava",
40:                 "repvgg_a2",
````
- EN: Handles module imports such as `argparse`, `os`, `sys`, `textwrap`.
- CN: 处理模块导入，例如 `argparse`, `os`, `sys`, `textwrap`。
- EN: Implements callable logic such as `get_field`, `check_accuracy`.
- CN: 实现可调用逻辑，例如 `get_field`, `check_accuracy`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 41-80
````python
41:                 "resnet152",
42:                 "resnet18",
43:                 "resnet50",
44:                 "stable_diffusion_unet",
45:                 "torchrec_dlrm",
46:                 "shufflenet_v2_x1_0",
47:                 "vgg16",
48:                 "BERT_pytorch",
49:                 # LLM
50:                 "google/gemma-2-2b",
51:                 "tts_angular",  # RuntimeError: Cannot access data pointer of Tensor
52:                 # Discovered on gfx950 CI after ROCm 7.2 upgrade, eager mode non determinism
53:                 "alexnet",
54:                 "demucs",
55:             }
56:         )
57: 
58:     for model in actual_csv["name"]:
59:         accuracy = get_field(actual_csv, model, "accuracy")
60:         expected_accuracy = get_field(expected_csv, model, "accuracy")
61: 
62:         if accuracy is None:
63:             status = "MISSING_ACCURACY:"
64:             failed.append(model)
65:         elif expected_accuracy is None:
66:             status = "MISSING_EXPECTED:"
67:             failed.append(model)
68:         elif accuracy == expected_accuracy:
69:             status = "PASS" if expected_accuracy == "pass" else "XFAIL"
70:             print(f"{model:34}  {status}")
71:             continue
72:         elif model in flaky_models:
73:             if accuracy == "pass":
74:                 # model passed but marked xfailed
75:                 status = "PASS_BUT_FLAKY:"
76:             else:
77:                 # model failed but marked passe
78:                 status = "FAIL_BUT_FLAKY:"
79:         elif accuracy != "pass":
80:             status = "FAIL:"
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 81-120
````python
 81:             failed.append(model)
 82:         else:
 83:             status = "IMPROVED:"
 84:             improved.append(model)
 85:         print(
 86:             f"{model:34}  {status:9} accuracy={accuracy}, expected={expected_accuracy}"
 87:         )
 88: 
 89:     msg = ""
 90:     if failed or improved:
 91:         if failed:
 92:             msg += textwrap.dedent(
 93:                 f"""
 94:             Error: {len(failed)} models have accuracy status regressed:
 95:                 {" ".join(failed)}
 96: 
 97:             """
 98:             )
 99:         if improved:
100:             msg += textwrap.dedent(
101:                 f"""
102:             Improvement: {len(improved)} models have accuracy status improved:
103:                 {" ".join(improved)}
104: 
105:             """
106:             )
107:         sha = os.getenv("SHA1", "{your CI commit sha}")
108:         msg += textwrap.dedent(
109:             f"""
110:         If this change is expected, you can update `{expected_filename}` to reflect the new baseline.
111:         from pytorch/pytorch root, run
112:         `python benchmarks/dynamo/ci_expected_accuracy/update_expected.py {sha}`
113:         and then `git add` the resulting local changes to expected CSVs to your commit.
114:         """
115:         )
116:     return failed or improved, msg
117: 
118: 
119: def main():
120:     parser = argparse.ArgumentParser()
````
- EN: Implements callable logic such as `main`.
- CN: 实现可调用逻辑，例如 `main`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-135
````python
121:     parser.add_argument("--actual", type=str, required=True)
122:     parser.add_argument("--expected", type=str, required=True)
123:     args = parser.parse_args()
124: 
125:     actual = pd.read_csv(args.actual)
126:     expected = pd.read_csv(args.expected)
127: 
128:     failed, msg = check_accuracy(actual, expected, args.expected)
129:     if failed:
130:         print(msg)
131:         sys.exit(1)
132: 
133: 
134: if __name__ == "__main__":
135:     main()
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `get_field` / 符号 `get_field`
- Symbol `check_accuracy` / 符号 `check_accuracy`
- Symbol `main` / 符号 `main`

## Dependencies / 依赖关系
- Python imports: `argparse`, `os`, `sys`, `textwrap`, `pandas`
- Python 导入: `argparse`, `os`, `sys`, `textwrap`, `pandas`
