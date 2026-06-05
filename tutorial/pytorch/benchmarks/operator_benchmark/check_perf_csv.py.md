# check_perf_csv.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/operator_benchmark/check_perf_csv.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40
````python
 1: import argparse
 2: import sys
 3: import textwrap
 4: 
 5: import pandas as pd
 6: 
 7: 
 8: SKIP_TEST_LISTS = [
 9:     # https://github.com/pytorch/pytorch/issues/143852
10:     "channel_shuffle_batch_size4_channels_per_group64_height64_width64_groups4_channel_lastTrue",
11:     "batchnorm_N3136_C256_cpu_trainingTrue_cudnnFalse",
12:     "index_add__M256_N512_K1_dim1_cpu_dtypetorch.float32",
13:     "interpolate_input_size(1,3,600,400)_output_size(240,240)_channels_lastTrue_modelinear",
14:     "original_kernel_tensor_N1_C3_H512_W512_zero_point_dtypetorch.int32_nbits4_cpu",
15:     "original_kernel_tensor_N1_C3_H512_W512_zero_point_dtypetorch.int32_nbits8_cpu",
16: ]
17: 
18: 
19: def get_field(csv, case: str, field: str):
20:     try:
21:         return csv.loc[csv["Case Name"] == case][field].item()
22:     except Exception:
23:         return None
24: 
25: 
26: def check_perf(actual_csv, expected_csv, expected_filename, threshold):
27:     failed = []
28:     improved = []
29:     baseline_not_found = []
30: 
31:     actual_csv = actual_csv[~actual_csv["Case Name"].isin(set(SKIP_TEST_LISTS))]
32: 
33:     for case in actual_csv["Case Name"]:
34:         perf = get_field(actual_csv, case, "Execution Time")
35:         expected_perf = get_field(expected_csv, case, "Execution Time")
36: 
37:         if expected_perf is None:
38:             status = "Baseline Not Found"
39:             print(f"{case:34}  {status}")
40:             baseline_not_found.append(case)
````
- EN: Handles module imports such as `argparse`, `sys`, `textwrap`, `pandas`.
- CN: 处理模块导入，例如 `argparse`, `sys`, `textwrap`, `pandas`。
- EN: Implements callable logic such as `get_field`, `check_perf`.
- CN: 实现可调用逻辑，例如 `get_field`, `check_perf`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 41-80
````python
41:             continue
42: 
43:         speed_up = expected_perf / perf
44: 
45:         if (1 - threshold) <= speed_up < (1 + threshold):
46:             status = "PASS"
47:             print(f"{case:34}  {status}")
48:             continue
49:         elif speed_up >= 1 + threshold:
50:             status = "IMPROVED:"
51:             improved.append(case)
52:         else:
53:             status = "FAILED:"
54:             failed.append(case)
55:         print(f"{case:34}  {status:9} perf={perf}, expected={expected_perf}")
56: 
57:     msg = ""
58:     if failed or improved or baseline_not_found:
59:         if failed:
60:             msg += textwrap.dedent(
61:                 f"""
62:             Error: {len(failed)} models have performance status regressed:
63:                 {" ".join(failed)}
64: 
65:             """
66:             )
67:         if improved:
68:             msg += textwrap.dedent(
69:                 f"""
70:             Improvement: {len(improved)} models have performance status improved:
71:                 {" ".join(improved)}
72: 
73:             """
74:             )
75: 
76:         if baseline_not_found:
77:             msg += textwrap.dedent(
78:                 f"""
79:             Baseline Not Found: {len(baseline_not_found)} models don't have the baseline data:
80:                 {" ".join(baseline_not_found)}
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 81-116
````python
 81: 
 82:             """
 83:             )
 84: 
 85:         msg += textwrap.dedent(
 86:             f"""
 87:         If this change is expected, you can update `{expected_filename}` to reflect the new baseline.
 88:         """
 89:         )
 90:     return failed or improved or baseline_not_found, msg
 91: 
 92: 
 93: def main():
 94:     parser = argparse.ArgumentParser()
 95:     parser.add_argument("--actual", type=str, required=True)
 96:     parser.add_argument("--expected", type=str, required=True)
 97:     parser.add_argument(
 98:         "--threshold",
 99:         type=float,
100:         default=0.5,
101:         help="threshold to define regression/improvement",
102:     )
103:     args = parser.parse_args()
104: 
105:     actual = pd.read_csv(args.actual)
106:     actual.drop_duplicates(subset=["Case Name"], keep="first", inplace=True)
107:     expected = pd.read_csv(args.expected)
108: 
109:     failed, msg = check_perf(actual, expected, args.expected, args.threshold)
110:     if failed:
111:         print(msg)
112:         sys.exit(1)
113: 
114: 
115: if __name__ == "__main__":
116:     main()
````
- EN: Implements callable logic such as `main`.
- CN: 实现可调用逻辑，例如 `main`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `get_field` / 符号 `get_field`
- Symbol `check_perf` / 符号 `check_perf`
- Symbol `main` / 符号 `main`

## Dependencies / 依赖关系
- Python imports: `argparse`, `sys`, `textwrap`, `pandas`
- Python 导入: `argparse`, `sys`, `textwrap`, `pandas`
