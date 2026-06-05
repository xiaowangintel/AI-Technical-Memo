# benchmark.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/dynamo/genai_layers/benchmark.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40
````python
 1: #!/usr/bin/env python3
 2: """
 3: Benchmark runner for various kernel implementations.
 4: 
 5: This script provides a command-line interface to run benchmarks for different
 6: kernel implementations including CrossEntropy, Softmax, RMSNorm, and LayerNorm
 7: kernels in both forward and backward directions.
 8: """
 9: 
10: import argparse
11: import sys
12: 
13: from kernels import (
14:     BenchmarkKernel,
15:     CrossEntropyBackward,
16:     CrossEntropyForward,
17:     LayerNormBackward,
18:     LayerNormForward,
19:     RMSNormBackward,
20:     RMSNormForward,
21:     SoftmaxBackward,
22:     SoftmaxForward,
23: )
24: 
25: import torch
26: 
27: 
28: torch._dynamo.config.automatic_dynamic_shapes = False
29: # Needed since changing args to function causes recompiles
30: torch._dynamo.config.recompile_limit = 1000000
31: 
32: 
33: # Registry of all available benchmarks
34: BENCHMARK_REGISTRY: dict[str, type[BenchmarkKernel]] = {
35:     "cross_entropy_forward": CrossEntropyForward,
36:     "cross_entropy_backward": CrossEntropyBackward,
37:     "softmax_forward": SoftmaxForward,
38:     "softmax_backward": SoftmaxBackward,
39:     "rmsnorm_forward": RMSNormForward,
40:     "rmsnorm_backward": RMSNormBackward,
````
- EN: Handles module imports such as `argparse`, `sys`, `kernels`, `torch`.
- CN: 处理模块导入，例如 `argparse`, `sys`, `kernels`, `torch`。
- EN: Declares or extends types including `to`.
- CN: 声明或扩展类型，包括 `to`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 41-80
````python
41:     "layernorm_forward": LayerNormForward,
42:     "layernorm_backward": LayerNormBackward,
43: }
44: 
45: 
46: def show_environment_info():
47:     """Show environment information."""
48:     print("Environment information:")
49:     print(f"  Python version: {sys.version}")
50:     print(f"  PyTorch version: {torch.__version__}")
51:     print(f"  CUDA version: {torch.version.cuda}")
52: 
53: 
54: def list_benchmarks():
55:     """List all available benchmarks."""
56:     print(f"Available benchmarks: {list(BENCHMARK_REGISTRY.keys())}")
57: 
58: 
59: def _run_benchmark(
60:     benchmark_cls,
61:     script_args,
62: ):
63:     benchmark = benchmark_cls(script_args)
64:     benchmark.benchmark()
65:     benchmark.report_geomean_speedup()
66:     if script_args.print_benchmark_result:
67:         print(f"Benchmarking results {benchmark.name}:")
68:         print(benchmark.profiling_results)
69:     if script_args.visualize:
70:         benchmark.visualize()
71: 
72: 
73: def run_benchmark(
74:     benchmark_name: str,
75:     script_args,
76: ):
77:     """Run a specific benchmark."""
78:     if benchmark_name not in BENCHMARK_REGISTRY:
79:         print(f"Error: Unknown benchmark '{benchmark_name}'")
80:         print("Use --list to see available benchmarks")
````
- EN: Implements callable logic such as `show_environment_info`, `list_benchmarks`, `_run_benchmark`, `run_benchmark`.
- CN: 实现可调用逻辑，例如 `show_environment_info`, `list_benchmarks`, `_run_benchmark`, `run_benchmark`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 81-120
````python
 81:         return False
 82: 
 83:     print(f"Running benchmark: {benchmark_name}")
 84:     print(f"Torch compile mode: {script_args.compile_mode}")
 85:     print("=" * 60)
 86: 
 87:     benchmark_class = BENCHMARK_REGISTRY[benchmark_name]
 88:     _run_benchmark(benchmark_class, script_args)
 89: 
 90:     return True
 91: 
 92: 
 93: def run_all_benchmarks(script_args):
 94:     """Run all available benchmarks."""
 95:     print("Running all benchmarks...")
 96:     print(f"Torch compile mode: {script_args.compile_mode}")
 97:     print("=" * 60)
 98: 
 99:     for name, cls in BENCHMARK_REGISTRY.items():
100:         print(f"\n{'=' * 20} {name.upper()} {'=' * 20}")
101:         _run_benchmark(cls, script_args)
102:         print()
103: 
104: 
105: def main():
106:     show_environment_info()
107: 
108:     parser = argparse.ArgumentParser(
109:         description="Benchmark runner for kernel implementations",
110:         formatter_class=argparse.RawDescriptionHelpFormatter,
111:         epilog="""
112: Examples:
113:   python benchmark.py --list                    # List all available benchmarks
114:   python benchmark.py --all                     # Run all benchmarks
115:   python benchmark.py cross_entropy_forward     # Run specific benchmark
116:   python benchmark.py softmax_forward softmax_backward  # Run multiple benchmarks
117:         """,
118:     )
119: 
120:     parser.add_argument(
````
- EN: Implements callable logic such as `run_all_benchmarks`, `main`.
- CN: 实现可调用逻辑，例如 `run_all_benchmarks`, `main`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-160
````python
121:         "benchmarks",
122:         nargs="*",
123:         help="Names of benchmarks to run (use --list to see available options)",
124:     )
125: 
126:     parser.add_argument(
127:         "--list", action="store_true", help="List all available benchmarks"
128:     )
129: 
130:     parser.add_argument(
131:         "--all", action="store_true", help="Run all available benchmarks"
132:     )
133: 
134:     parser.add_argument(
135:         "--visualize",
136:         action="store_true",
137:         help="Visualize results after running benchmarks",
138:     )
139: 
140:     parser.add_argument(
141:         "--compile-mode",
142:         choices=["default", "max-autotune-no-cudagraphs"],
143:         default="max-autotune-no-cudagraphs",
144:         help="Torch compile mode to use (default: default)",
145:     )
146: 
147:     parser.add_argument(
148:         "--tolerance",
149:         type=float,
150:         default=None,
151:         help="Tolerance for the accuracy check",
152:     )
153: 
154:     parser.add_argument(
155:         "--exit-on-accuracy-failure",
156:         action="store_true",
157:         help="Whether to exit with an error message for accuracy failure",
158:     )
159: 
160:     parser.add_argument(
````
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 161-200
````python
161:         "--print-benchmark-result",
162:         action="store_true",
163:         help="Whether to print the raw benchmarking result. Easier to quickly check the benchmark results on a server without GUI",
164:     )
165: 
166:     parser.add_argument(
167:         "--custom-compile-name",
168:         type=str,
169:         default=None,
170:         help="Name for the curve with customized compilation options",
171:     )
172: 
173:     parser.add_argument(
174:         "--custom-compile-options",
175:         type=str,
176:         default=None,
177:         help="Json string for the custom compile options.",
178:     )
179: 
180:     args = parser.parse_args()
181: 
182:     if args.custom_compile_options:
183:         import json
184: 
185:         try:
186:             args.custom_compile_options = json.loads(args.custom_compile_options)
187:         except json.decoder.JSONDecodeError as e:
188:             raise RuntimeError(
189:                 f"Invalid json string for --custom-compile-options: {args.custom_compile_options}"
190:             ) from e
191: 
192:         if not args.custom_compile_options:
193:             raise RuntimeError("Found no options for --custom-compile-options")
194:         if not args.custom_compile_name:
195:             raise RuntimeError("Missing label name for the custom compilation")
196: 
197:     # Handle list option
198:     if args.list:
199:         list_benchmarks()
200:         return
````
- EN: Handles module imports such as `json`.
- CN: 处理模块导入，例如 `json`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 201-220
````python
201: 
202:     # Handle all option
203:     if args.all:
204:         run_all_benchmarks(args)
205:         return
206: 
207:     # Handle specific benchmarks
208:     if not args.benchmarks:
209:         print("Error: No benchmarks specified")
210:         print("Use --list to see available benchmarks or --all to run all benchmarks")
211:         parser.print_help()
212:         sys.exit(1)
213: 
214:     for benchmark_name in args.benchmarks:
215:         run_benchmark(benchmark_name, args)
216:         print()  # Add spacing between benchmarks
217: 
218: 
219: if __name__ == "__main__":
220:     main()
````
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `show_environment_info` / 符号 `show_environment_info`
- Symbol `list_benchmarks` / 符号 `list_benchmarks`
- Symbol `_run_benchmark` / 符号 `_run_benchmark`
- Symbol `run_benchmark` / 符号 `run_benchmark`

## Dependencies / 依赖关系
- Python imports: `argparse`, `sys`, `kernels`, `torch`, `json`
- Python 导入: `argparse`, `sys`, `kernels`, `torch`, `json`
