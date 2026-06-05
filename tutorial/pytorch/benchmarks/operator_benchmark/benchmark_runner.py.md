# benchmark_runner.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/operator_benchmark/benchmark_runner.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40
````python
 1: import argparse
 2: 
 3: import benchmark_core
 4: 
 5: import benchmark_utils
 6: 
 7: import torch
 8: 
 9: 
10: """Performance microbenchmarks's main binary.
11: 
12: This is the main function for running performance microbenchmark tests.
13: It also registers existing benchmark tests via Python module imports.
14: """
15: parser = argparse.ArgumentParser(
16:     description="Run microbenchmarks.",
17:     formatter_class=argparse.ArgumentDefaultsHelpFormatter,
18:     conflict_handler="resolve",
19:     allow_abbrev=False,
20: )
21: 
22: 
23: def parse_args():
24:     parser.add_argument(
25:         "--tag-filter",
26:         "--tag_filter",
27:         help="tag_filter can be used to run the shapes which matches the tag. (all is used to run all the shapes)",
28:         default="short",
29:     )
30: 
31:     # This option is used to filter test cases to run.
32:     parser.add_argument(
33:         "--operators",
34:         help="Filter tests based on comma-delimited list of operators to test",
35:         default=None,
36:     )
37: 
38:     parser.add_argument(
39:         "--operator-range",
40:         "--operator_range",
````
- EN: Handles module imports such as `argparse`, `benchmark_core`, `benchmark_utils`, `torch`.
- CN: 处理模块导入，例如 `argparse`, `benchmark_core`, `benchmark_utils`, `torch`。
- EN: Implements callable logic such as `parse_args`.
- CN: 实现可调用逻辑，例如 `parse_args`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 41-80
````python
41:         help="Filter tests based on operator_range(e.g. a-c or b,c-d)",
42:         default=None,
43:     )
44: 
45:     parser.add_argument(
46:         "--test-name",
47:         "--test_name",
48:         help="Run tests that have the provided test_name",
49:         default=None,
50:     )
51: 
52:     parser.add_argument(
53:         "--list-ops",
54:         "--list_ops",
55:         help="List operators without running them",
56:         action="store_true",
57:     )
58: 
59:     parser.add_argument(
60:         "--output-json",
61:         "--output_json",
62:         help="JSON file path to write the results to",
63:         default=None,
64:     )
65: 
66:     parser.add_argument(
67:         "--benchmark-name",
68:         "--benchmark_name",
69:         help="Name of the benchmark to store results to",
70:         default="PyTorch operator benchmark",
71:     )
72: 
73:     parser.add_argument(
74:         "--list-tests",
75:         "--list_tests",
76:         help="List all test cases without running them",
77:         action="store_true",
78:     )
79: 
80:     parser.add_argument(
````
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。

### Lines 81-120
````python
 81:         "--iterations",
 82:         help="Repeat each operator for the number of iterations",
 83:         type=int,
 84:     )
 85: 
 86:     parser.add_argument(
 87:         "--num-runs",
 88:         "--num_runs",
 89:         help="Run each test for num_runs. Each run executes an operator for number of <--iterations>",
 90:         type=int,
 91:         default=1,
 92:     )
 93: 
 94:     parser.add_argument(
 95:         "--min-time-per-test",
 96:         "--min_time_per_test",
 97:         help="Set the minimum time (unit: seconds) to run each test",
 98:         type=int,
 99:         default=0,
100:     )
101: 
102:     parser.add_argument(
103:         "--warmup-iterations",
104:         "--warmup_iterations",
105:         help="Number of iterations to ignore before measuring performance",
106:         default=100,
107:         type=int,
108:     )
109: 
110:     parser.add_argument(
111:         "--omp-num-threads",
112:         "--omp_num_threads",
113:         help="Number of OpenMP threads used in PyTorch runtime",
114:         default=None,
115:         type=int,
116:     )
117: 
118:     parser.add_argument(
119:         "--mkl-num-threads",
120:         "--mkl_num_threads",
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 121-160
````python
121:         help="Number of MKL threads used in PyTorch runtime",
122:         default=None,
123:         type=int,
124:     )
125: 
126:     parser.add_argument(
127:         "--report-aibench",
128:         "--report_aibench",
129:         type=benchmark_utils.str2bool,
130:         nargs="?",
131:         const=True,
132:         default=False,
133:         help="Print result when running on AIBench",
134:     )
135: 
136:     parser.add_argument(
137:         "--use-jit",
138:         "--use_jit",
139:         type=benchmark_utils.str2bool,
140:         nargs="?",
141:         const=True,
142:         default=False,
143:         help="Run operators with PyTorch JIT mode",
144:     )
145: 
146:     parser.add_argument(
147:         "--use-compile",
148:         "--use_compile",
149:         type=benchmark_utils.str2bool,
150:         nargs="?",
151:         const=True,
152:         default=False,
153:         help="Run operators with PyTorch Compile mode",
154:     )
155: 
156:     parser.add_argument(
157:         "--forward-only",
158:         "--forward_only",
159:         type=benchmark_utils.str2bool,
160:         nargs="?",
````
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。

### Lines 161-200
````python
161:         const=True,
162:         default=False,
163:         help="Only run the forward path of operators",
164:     )
165: 
166:     parser.add_argument(
167:         "--device",
168:         help="Run tests on the provided architecture (cpu, cuda)",
169:         default="None",
170:     )
171: 
172:     parser.add_argument(
173:         "--output-csv",
174:         "--output_csv",
175:         help="CSV file path to store the results",
176:         default="benchmark_logs",
177:     )
178: 
179:     parser.add_argument(
180:         "--output-json-for-dashboard",
181:         "--output_json_for_dashboard",
182:         help="Save results in JSON format for display on the OSS dashboard",
183:         default="benchmark-results.json",
184:     )
185: 
186:     args, _ = parser.parse_known_args()
187: 
188:     if args.omp_num_threads:
189:         # benchmark_utils.set_omp_threads sets the env variable OMP_NUM_THREADS
190:         # which doesn't have any impact as C2 init logic has already been called
191:         # before setting the env var.
192: 
193:         # In general, OMP_NUM_THREADS (and other OMP env variables) needs to be set
194:         # before the program is started.
195:         # From Chapter 4 in OMP standard: https://www.openmp.org/wp-content/uploads/openmp-4.5.pdf
196:         # "Modifications to the environment variables after the program has started,
197:         # even if modified by the program itself, are ignored by the OpenMP implementation"
198:         benchmark_utils.set_omp_threads(args.omp_num_threads)
199:         torch.set_num_threads(args.omp_num_threads)
200:     if args.mkl_num_threads:
````
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 201-212
````python
201:         benchmark_utils.set_mkl_threads(args.mkl_num_threads)
202: 
203:     return args
204: 
205: 
206: def main():
207:     args = parse_args()
208:     benchmark_core.BenchmarkRunner(args).run()
209: 
210: 
211: if __name__ == "__main__":
212:     main()
````
- EN: Implements callable logic such as `main`.
- CN: 实现可调用逻辑，例如 `main`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `parse_args` / 符号 `parse_args`
- Symbol `main` / 符号 `main`

## Dependencies / 依赖关系
- Python imports: `argparse`, `benchmark_core`, `benchmark_utils`, `torch`
- Python 导入: `argparse`, `benchmark_core`, `benchmark_utils`, `torch`
