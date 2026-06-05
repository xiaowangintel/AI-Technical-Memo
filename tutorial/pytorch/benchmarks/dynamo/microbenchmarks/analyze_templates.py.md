# analyze_templates.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/dynamo/microbenchmarks/analyze_templates.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40
````python
 1: """
 2: This script uses linear programming to analyze outputs of triton mm config tuning.
 3: To generate output that can be fed into this script set the env varTORCHINDUCTOR_MM_LOGGING_FILE.
 4: 
 5: That file can be fed into this script to generate the minimizes total, weighted matmul time as a function of allowed templates.
 6: """
 7: 
 8: import json
 9: 
10: import click
11: import pulp
12: 
13: 
14: def parse_log_file(file_path):
15:     with open(file_path) as f:
16:         logs = json.load(f)
17: 
18:     occurrence_count = {}
19:     benchmark_logs = {}
20: 
21:     # Parse the logs
22:     for entry in logs:
23:         if "invoke" in entry:
24:             shape = entry["invoke"]
25:             if shape not in occurrence_count:
26:                 occurrence_count[shape] = 0
27:             occurrence_count[shape] += 1
28:         else:
29:             for shape, timings in entry.items():
30:                 if shape not in benchmark_logs:
31:                     benchmark_logs[shape] = []
32:                 benchmark_logs[shape].extend(timings)
33: 
34:     return occurrence_count, benchmark_logs
35: 
36: 
37: def optimize_templates(N, occurrence_count, benchmark_logs, verbose=False):
38:     # Set of all possible Triton templates keyed by their attributes
39:     triton_templates = set()
40:     for timings in benchmark_logs.values():
````
- EN: Handles module imports such as `json`, `click`, `pulp`.
- CN: 处理模块导入，例如 `json`, `click`, `pulp`。
- EN: Implements callable logic such as `parse_log_file`, `optimize_templates`.
- CN: 实现可调用逻辑，例如 `parse_log_file`, `optimize_templates`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 41-80
````python
41:         for timing in timings:
42:             if timing["type"] == "triton":
43:                 triton_templates.add(
44:                     (
45:                         timing["BLOCK_M"],
46:                         timing["BLOCK_N"],
47:                         timing["BLOCK_K"],
48:                         timing["num_stages"],
49:                         timing["num_warps"],
50:                     )
51:                 )
52: 
53:     # Print the initial data
54:     if verbose:
55:         print("Occurrence Count:", occurrence_count)
56:         print("Triton Templates:", triton_templates)
57: 
58:     # Create a dictionary to store template selection variables
59:     template_vars = {
60:         template: pulp.LpVariable(f"Template_{template}", 0, 1, pulp.LpBinary)
61:         for template in triton_templates
62:     }
63: 
64:     # Variables to select specific timing option for each shape
65:     selection_vars = {
66:         (shape, "cublas"): pulp.LpVariable(
67:             f"Select_{shape}_cublas", 0, 1, pulp.LpBinary
68:         )
69:         for shape in occurrence_count
70:     }
71:     for shape in occurrence_count:
72:         for template in triton_templates:
73:             selection_vars[(shape, template)] = pulp.LpVariable(
74:                 f"Select_{shape}_{template}", 0, 1, pulp.LpBinary
75:             )
76: 
77:     # Variables for the total time for each shape
78:     min_time_vars = pulp.LpVariable.dicts(
79:         "MinTime", occurrence_count.keys(), 0, None, pulp.LpContinuous
80:     )
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 81-120
````python
 81: 
 82:     # Define the problem
 83:     prob = pulp.LpProblem("MatrixMultiplicationOptimization", pulp.LpMinimize)
 84: 
 85:     # Objective: Minimize the weighted total time
 86:     prob += pulp.lpSum(
 87:         [occurrence_count[shape] * min_time_vars[shape] for shape in occurrence_count]
 88:     )
 89: 
 90:     # Constraints to select exactly N templates
 91:     prob += pulp.lpSum([template_vars[template] for template in triton_templates]) == N
 92: 
 93:     # Store triton options per shape for debugging
 94:     triton_options_per_shape = {}
 95: 
 96:     # Constraints for the total time for each shape
 97:     for shape in occurrence_count:
 98:         # Get cuBLAS time
 99:         cublas_times = [
100:             timing["time"]
101:             for timing in benchmark_logs[shape]
102:             if timing["type"] == "cublas"
103:         ]
104:         min_cublas_time = min(cublas_times)
105: 
106:         # Collect Triton options
107:         triton_options = []
108:         for template in triton_templates:
109:             triton_times = [
110:                 timing["time"]
111:                 for timing in benchmark_logs[shape]
112:                 if timing["type"] == "triton"
113:                 and (
114:                     timing["BLOCK_M"],
115:                     timing["BLOCK_N"],
116:                     timing["BLOCK_K"],
117:                     timing["num_stages"],
118:                     timing["num_warps"],
119:                 )
120:                 == template
````
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 121-160
````python
121:             ]
122:             if triton_times:
123:                 min_triton_time = min(triton_times)
124:                 triton_options.append((min_triton_time, template))
125: 
126:         # Save triton options for debugging
127:         triton_options_per_shape[shape] = triton_options
128: 
129:         # Ensure exactly one timing option is selected for each shape
130:         prob += (
131:             pulp.lpSum(
132:                 [selection_vars[(shape, "cublas")]]
133:                 + [
134:                     selection_vars[(shape, template)]
135:                     for triton_time, template in triton_options
136:                 ]
137:             )
138:             == 1
139:         )
140: 
141:         # Ensure min_time_vars[shape] matches the selected timing option
142:         prob += min_time_vars[shape] == (
143:             selection_vars[(shape, "cublas")] * min_cublas_time
144:             + pulp.lpSum(
145:                 [
146:                     selection_vars[(shape, template)] * triton_time
147:                     for triton_time, template in triton_options
148:                 ]
149:             )
150:         )
151: 
152:         # Ensure Triton templates can only be selected if they are included in the N allowed templates
153:         for triton_time, template in triton_options:
154:             prob += selection_vars[(shape, template)] <= template_vars[template]
155: 
156:     # Print the constraints
157:     if verbose:
158:         print("Constraints:")
159:         for constraint in prob.constraints.values():
160:             print(constraint)
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 161-200
````python
161: 
162:     # Solve the problem with suppressed output
163:     prob.solve(pulp.PULP_CBC_CMD(msg=False))
164: 
165:     # Output the selected templates and their configurations
166:     selected_templates = [
167:         template
168:         for template in triton_templates
169:         if pulp.value(template_vars[template]) == 1
170:     ]
171:     total_time = sum(
172:         pulp.value(min_time_vars[shape]) * occurrence_count[shape]
173:         for shape in occurrence_count
174:     )
175: 
176:     # Print the values of the decision variables after solving
177:     if verbose:
178:         print("Decision Variable Values:")
179:         for var in prob.variables():
180:             print(f"{var.name} = {var.varValue}")
181: 
182:     # # Debugging information
183:     if verbose:
184:         for shape in occurrence_count:
185:             print(f"Shape: {shape}")
186:             print(f"  Min Time: {pulp.value(min_time_vars[shape])}")
187:             print(f"  Occurrences: {occurrence_count[shape]}")
188:             print(
189:                 f"  Min CuBLAS Time: {min_cublas_time} Selected: {pulp.value(selection_vars[(shape, 'cublas')])}"
190:             )
191:             for triton_time, template in triton_options_per_shape[shape]:
192:                 print(
193:                     f"  Triton Template: {template} Time: {triton_time} Selected: {pulp.value(selection_vars[(shape, template)])}"
194:                 )
195: 
196:     return selected_templates, total_time
197: 
198: 
199: # Main code to parse the log file and optimize templates
200: @click.command()
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 201-220
````python
201: @click.argument("filename")
202: @click.option("--min-templates", default=0, help="Minimum number of templates.")
203: @click.option("--max-templates", default=10, help="Maximum number of templates.")
204: @click.option("--verbose", is_flag=True, help="Enable verbose output.")
205: def main(filename, min_templates, max_templates, verbose):
206:     occurrence_count, benchmark_logs = parse_log_file(filename)
207:     times = []
208:     for N in range(min_templates, max_templates + 1):
209:         selected_templates, total_time = optimize_templates(
210:             N, occurrence_count, benchmark_logs, verbose
211:         )
212:         print(f"N = {N}")
213:         print(f"Selected Templates: {selected_templates}")
214:         print(f"Total Weighted Time: {total_time}")
215:         times.append(total_time)
216:     print(times)
217: 
218: 
219: if __name__ == "__main__":
220:     main()
````
- EN: Implements callable logic such as `main`.
- CN: 实现可调用逻辑，例如 `main`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `parse_log_file` / 符号 `parse_log_file`
- Symbol `optimize_templates` / 符号 `optimize_templates`
- Symbol `main` / 符号 `main`

## Dependencies / 依赖关系
- Python imports: `json`, `click`, `pulp`
- Python 导入: `json`, `click`, `pulp`
