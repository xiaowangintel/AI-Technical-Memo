# check_results.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/dynamo/pr_time_benchmarks/check_results.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60
````python
 1: import copy
 2: import csv
 3: import json
 4: import sys
 5: from dataclasses import dataclass
 6: 
 7: import torch._logging.scribe as scribe
 8: 
 9: 
10: @dataclass
11: class ExpectedFileEntry:
12:     benchmark_name: str
13:     metric_name: str
14:     expected_value: int
15:     noise_margin: float
16: 
17: 
18: @dataclass
19: class ResultFileEntry:
20:     benchmark_name: str
21:     metric_name: str
22:     actual_value: int
23: 
24: 
25: def replace_with_zeros(num):
26:     """
27:     Keeps the first three digits of an integer and replaces the rest with zeros.
28: 
29:     Args:
30:         num (int): The number to modify.
31: 
32:     Returns:
33:         int: The modified number.
34: 
35:     Raises:
36:         ValueError: If the input is not an integer.
37:     """
38:     # Check if input is an integer
39:     if not isinstance(num, int):
40:         raise ValueError("Input must be an integer")
41: 
42:     # Calculate the number of digits to remove
43:     digits_to_remove = len(str(abs(num))) - 4
44: 
45:     # Replace digits with zeros
46:     if digits_to_remove > 0:
47:         modified_num = (num // 10**digits_to_remove) * 10**digits_to_remove
48:     else:
49:         modified_num = num
50: 
51:     return modified_num
52: 
53: 
54: def main():
55:     # Expected file is the file that have the results that we are comparing against.
56:     # Expected has the following format:
57:     # benchmark_name, metric name, expected value, noise margin (as percentage)
58:     # Example:
59:     # add_loop_eager,compile_time_instruction_count,283178305, 0.01 (1% noise margin)
60:     expected_file_path = sys.argv[1]
````
- EN: Handles module imports such as `copy`, `csv`, `json`, `sys`.
- CN: 处理模块导入，例如 `copy`, `csv`, `json`, `sys`。
- EN: Declares or extends types including `ExpectedFileEntry`, `ResultFileEntry`.
- CN: 声明或扩展类型，包括 `ExpectedFileEntry`, `ResultFileEntry`。
- EN: Implements callable logic such as `replace_with_zeros`, `main`.
- CN: 实现可调用逻辑，例如 `replace_with_zeros`, `main`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。

### Lines 61-120
````python
 61: 
 62:     # Result file is the file that have the results of the current run. It has the following format:
 63:     # benchmark_name, metric name, expected value, noise margin (as percentage)
 64:     # Example:
 65:     # add_loop_eager,compile_time_instruction_count,283178305
 66:     result_file_path = sys.argv[2]
 67: 
 68:     # A path where a new expected results file will be written that can be used to replace expected_results.csv
 69:     # in case of failure. In case of no failure the content of this file will match expected_file_path.
 70:     reference_expected_results_path = sys.argv[3]
 71: 
 72:     # Read expected data file.
 73:     expected_data: dict[str, ExpectedFileEntry] = {}
 74: 
 75:     with open(expected_file_path) as f:
 76:         reader = csv.reader(f)
 77:         for row in reader:
 78:             if len(row) == 0:
 79:                 continue
 80:             entry = ExpectedFileEntry(
 81:                 benchmark_name=row[0].strip(),
 82:                 metric_name=row[1].strip(),
 83:                 expected_value=int(row[2]),
 84:                 noise_margin=float(row[3]),
 85:             )
 86:             key = (entry.benchmark_name, entry.metric_name)
 87:             if key in expected_data:
 88:                 raise AssertionError(f"Duplicate entry for {key}")
 89:             expected_data[key] = entry
 90: 
 91:     # Read result data file.
 92:     result_data: dict[str, ResultFileEntry] = {}
 93: 
 94:     with open(result_file_path) as f:
 95:         reader = csv.reader(f)
 96:         for row in reader:
 97:             entry = ResultFileEntry(
 98:                 benchmark_name=row[0].strip(),
 99:                 metric_name=row[1].strip(),
100:                 actual_value=int(row[2]),
101:             )
102: 
103:             key = (entry.benchmark_name, entry.metric_name)
104:             if key in result_data:
105:                 raise AssertionError(f"Duplicate entry for {key}")
106:             result_data[key] = entry
107: 
108:     fail = False
109:     new_expected = copy.deepcopy(expected_data)
110:     for key, entry in expected_data.items():
111:         if key not in result_data:
112:             print(f"Missing entry for {key} in result file")
113:             sys.exit(1)
114: 
115:         low = entry.expected_value - entry.expected_value * entry.noise_margin
116:         high = entry.expected_value + entry.expected_value * entry.noise_margin
117:         result = result_data[key].actual_value
118:         ratio = float(result - entry.expected_value) * 100 / entry.expected_value
119: 
120:         def log(event_name):
````
- EN: Implements callable logic such as `log`.
- CN: 实现可调用逻辑，例如 `log`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 121-180
````python
121:             scribe.open_source_signpost(
122:                 subsystem="pr_time_benchmarks",
123:                 name=event_name,
124:                 parameters=json.dumps(
125:                     {
126:                         "benchmark_name": entry.benchmark_name,
127:                         "metric_name": entry.metric_name,
128:                         "actual_value": result,
129:                         "expected_value": entry.expected_value,
130:                         "noise_margin": entry.noise_margin,
131:                         "change_ratio": ratio,
132:                     }
133:                 ),
134:             )
135: 
136:         new_entry = copy.deepcopy(entry)
137:         # only change if abs(ratio) > entry.noise_margin /5.
138:         new_entry.expected_value = (
139:             replace_with_zeros(result)
140:             if abs(ratio) > entry.noise_margin * 100 / 5
141:             else entry.expected_value
142:         )
143:         new_expected[key] = new_entry
144: 
145:         if result > high:
146:             fail = True
147:             print(
148:                 f"REGRESSION: benchmark {key} failed, actual result {result} "
149:                 f"is {ratio:.2f}% higher than expected {entry.expected_value} ±{entry.noise_margin * 100:+.2f}% "
150:                 f"if this is an expected regression, please update the expected results.\n"
151:             )
152:             print(
153:                 "please update all results that changed significantly, and not only the failed ones"
154:             )
155: 
156:             log("fail_regression")
157: 
158:         elif result < low:
159:             fail = True
160: 
161:             print(
162:                 f"WIN: benchmark {key} failed, actual result {result} is {ratio:+.2f}% lower than "
163:                 f"expected {entry.expected_value} ±{entry.noise_margin * 100:.2f}% "
164:                 f"please OPEN THE TEST RESULTS update ALL BENCHMARKS RESULT with the new printed expected results. ALL ALL ALL\n"
165:             )
166:             print(
167:                 "please update all results that changed significantly, and not only the failed ones"
168:             )
169: 
170:             log("fail_win")
171: 
172:         else:
173:             print(
174:                 f"PASS: benchmark {key} pass, actual result {result} {ratio:+.2f}% is within "
175:                 f"expected {entry.expected_value} ±{entry.noise_margin * 100:.2f}%\n"
176:             )
177: 
178:             log("pass")
179: 
180:     # Log all benchmarks that do not have a regression test enabled for them.
````
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 181-240
````python
181:     for key, entry in result_data.items():
182:         if key not in expected_data:
183:             print(
184:                 f"MISSING REGRESSION TEST: benchmark {key} does not have a regression test enabled for it.\n"
185:             )
186:             scribe.open_source_signpost(
187:                 subsystem="pr_time_benchmarks",
188:                 name="missing_regression_test",
189:                 parameters=json.dumps(
190:                     {
191:                         "benchmark_name": entry.benchmark_name,
192:                         "metric_name": entry.metric_name,
193:                     }
194:                 ),
195:             )
196: 
197:     with open(reference_expected_results_path, "w", newline="") as csvfile:
198:         writer = csv.writer(csvfile)
199:         for entry in new_expected.values():
200:             # Write the data to the CSV file
201:             # print(f"{entry.benchmark_name},{entry.metric_name,},{round(entry.expected_value)},{entry.noise_margin}")
202:             writer.writerow(
203:                 [
204:                     entry.benchmark_name,
205:                     entry.metric_name,
206:                     entry.expected_value,
207:                     entry.noise_margin,
208:                 ]
209:             )
210:             # Three empty rows for merge conflicts.
211:             writer.writerow([])
212:             writer.writerow([])
213:             writer.writerow([])
214: 
215:     print("=" * 80)
216:     print("=" * 80)
217:     print("=" * 80)
218:     print("To update expected results, run the following command:")
219:     print()
220:     print("cat > benchmarks/dynamo/pr_time_benchmarks/expected_results.csv << EOF")
221:     with open(reference_expected_results_path) as f:
222:         print(f.read().rstrip())
223:     print("EOF")
224:     print()
225:     print("=" * 80)
226:     print("=" * 80)
227:     print("=" * 80)
228: 
229:     if fail:
230:         print(
231:             f"There was some failures you can use the new reference expected result stored at path:"
232:             f"{reference_expected_results_path} and printed above\n"
233:         )
234:         print(
235:             "To reproduce locally follow the following instructions, note that absolute instructions count are going "
236:             "to be different than on the CI, hence you might want to run locally with and without your change:\n"
237:             "cd benchmarks/dynamo/pr_time_benchmarks/ \n"
238:             "python benchmarks/BENCHMARK.py result.csv \n"
239:             "note that BENCHMARK.py is the name of the file containing the failing benchmark."
240:         )
````
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 241-247
````python
241:         sys.exit(1)
242:     else:
243:         print("All benchmarks passed")
244: 
245: 
246: if __name__ == "__main__":
247:     main()
````
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `ExpectedFileEntry` / 符号 `ExpectedFileEntry`
- Symbol `ResultFileEntry` / 符号 `ResultFileEntry`
- Symbol `replace_with_zeros` / 符号 `replace_with_zeros`
- Symbol `main` / 符号 `main`

## Dependencies / 依赖关系
- Python imports: `copy`, `csv`, `json`, `sys`, `dataclasses`, `torch._logging.scribe`
- Python 导入: `copy`, `csv`, `json`, `sys`, `dataclasses`, `torch._logging.scribe`
