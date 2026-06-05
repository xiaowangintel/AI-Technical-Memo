# parse_logs.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/dynamo/parse_logs.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40
````python
 1: import csv
 2: import os
 3: import re
 4: import sys
 5: from pathlib import Path
 6: 
 7: 
 8: # This script takes the logs produced by the benchmark scripts (e.g.,
 9: # torchbench.py) and parses it into a CSV file that summarizes what
10: # is failing and why.  It is kept separate from the benchmark script
11: # emitting a more structured output as it is often more convenient
12: # to iterate quickly on log files offline instead of having to make
13: # a change to the benchmark script and then do a full sweep to see
14: # the updates.
15: #
16: # This script is not very well written, feel free to rewrite it as necessary
17: 
18: if len(sys.argv) != 2:
19:     raise AssertionError(f"Expected 1 argument, but got {len(sys.argv) - 1}")
20: full_log = Path(sys.argv[1]).read_text()
21: 
22: # If the log contains a gist URL, extract it so we can include it in the CSV
23: gist_url = ""
24: m = re.search(r"https://gist.github.com/[a-f0-9]+", full_log)
25: if m is not None:
26:     gist_url = m.group(0)
27: 
28: # Split the log into an entry per benchmark
29: entries = re.split(
30:     r"(?:cuda (?:train|eval) +([^ ]+)|WARNING:root:([^ ]+) failed to load)", full_log
31: )[1:]
32: # Entries schema example:
33: # `['hf_Bert', None, '
34: #  PASS\nTIMING: entire_frame_compile:1.80925 backend_compile:6e-05\nDynamo produced 1 graph(s) covering 367 ops\n']`
35: 
36: 
37: def chunker(seq, size):
38:     return (seq[pos : pos + size] for pos in range(0, len(seq), size))
39: 
40: 
````
- EN: Handles module imports such as `csv`, `os`, `re`, `sys`.
- CN: 处理模块导入，例如 `csv`, `os`, `re`, `sys`。
- EN: Implements callable logic such as `chunker`.
- CN: 实现可调用逻辑，例如 `chunker`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 41-80
````python
41: c = 0
42: i = 0
43: 
44: out = csv.DictWriter(
45:     sys.stdout,
46:     [
47:         "bench",
48:         "name",
49:         "result",
50:         "component",
51:         "context",
52:         "explain",
53:         "frame_time",
54:         "backend_time",
55:         "graph_count",
56:         "op_count",
57:         "graph_breaks",
58:         "unique_graph_breaks",
59:     ],
60:     dialect="excel",
61: )
62: out.writeheader()
63: out.writerow({"explain": gist_url})
64: 
65: 
66: # Sometimes backtraces will be in third party code, which results
67: # in very long file names.  Delete the absolute path in this case.
68: def normalize_file(f):
69:     if "site-packages/" in f:
70:         return f.split("site-packages/", 2)[1]
71:     else:
72:         return os.path.relpath(f)
73: 
74: 
75: # Assume we run torchbench, huggingface, timm_models in that order
76: # (as output doesn't say which suite the benchmark is part of)
77: # TODO: make this more robust
78: 
79: bench = "torchbench"
80: 
````
- EN: Implements callable logic such as `normalize_file`.
- CN: 实现可调用逻辑，例如 `normalize_file`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 81-120
````python
 81: # 3 = 1 + number of matches in the entries split regex
 82: for name, name2, log in chunker(entries, 3):
 83:     if name is None:
 84:         name = name2
 85:     if name.startswith("Albert"):
 86:         bench = "huggingface"
 87:     elif name.startswith("adv_inc"):
 88:         bench = "timm_models"
 89: 
 90:     # Payload that will go into the csv
 91:     r = "UNKNOWN"
 92:     explain = ""
 93:     component = ""
 94:     context = ""
 95: 
 96:     if "PASS" in log:
 97:         r = "PASS"
 98:     if "TIMEOUT" in log:
 99:         r = "FAIL TIMEOUT"
100:     if "Accuracy failed" in log:
101:         r = "FAIL ACCURACY"
102: 
103:     # Attempt to extract out useful information from the traceback
104: 
105:     log = log.split(
106:         "The above exception was the direct cause of the following exception"
107:     )[0]
108:     split = log.split("Traceback (most recent call last)", maxsplit=1)
109:     if len(split) == 2:
110:         log = split[1]
111:     log = log.split("Original traceback:")[0]
112:     m = re.search(
113:         r'File "([^"]+)", line ([0-9]+), in .+\n +(.+)\n([A-Za-z]+(?:Error|Exception|NotImplementedError): ?.*)',
114:         log,
115:     )
116: 
117:     if m is not None:
118:         r = "FAIL"
119:         component = f"{normalize_file(m.group(1))}:{m.group(2)}"
120:         context = m.group(3)
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 121-160
````python
121:         explain = f"{m.group(4)}"
122:     else:
123:         m = re.search(
124:             r'File "([^"]+)", line ([0-9]+), in .+\n +(.+)\nAssertionError', log
125:         )
126:         if m is not None:
127:             r = "FAIL"
128:             component = f"{normalize_file(m.group(1))}:{m.group(2)}"
129:             context = m.group(3)
130:             explain = "AssertionError"
131: 
132:     # Sometimes, the benchmark will say FAIL without any useful info
133:     # See https://github.com/pytorch/torchdynamo/issues/1910
134:     if "FAIL" in log:
135:         r = "FAIL"
136: 
137:     if r == "UNKNOWN":
138:         c += 1
139: 
140:     backend_time = None
141:     frame_time = None
142:     if "TIMING:" in log:
143:         result = re.search("TIMING:(.*)\n", log).group(1)
144:         split_str = result.split("backend_compile:")
145:         if len(split_str) == 2:
146:             backend_time = float(split_str[1])
147:             frame_time = float(split_str[0].split("entire_frame_compile:")[1])
148: 
149:     if "STATS:" in log:
150:         result = re.search("STATS:(.*)\n", log).group(1)
151:         # call_* op count: 970 | FakeTensor.__torch_dispatch__:35285 | ProxyTorchDispatchMode.__torch_dispatch__:13339
152:         split_all = result.split("|")
153:         # TODO: rewrite this to work with arbitrarily many stats
154: 
155:     graph_count = None
156:     op_count = None
157:     graph_breaks = None
158:     unique_graph_breaks = None
159:     if m := re.search(
160:         r"Dynamo produced (\d+) graphs covering (\d+) ops with (\d+) graph breaks \((\d+) unique\)",
````
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 161-199
````python
161:         log,
162:     ):
163:         graph_count = m.group(1)
164:         op_count = m.group(2)
165:         graph_breaks = m.group(3)
166:         unique_graph_breaks = m.group(4)
167: 
168:     # If the context string is too long, don't put it in the CSV.
169:     # This is a hack to try to make it more likely that Google Sheets will
170:     # offer to split columns
171:     if len(context) > 78:
172:         context = ""
173: 
174:     # Temporary file names are meaningless, report it's generated code in this
175:     # case
176:     if "/tmp/" in component:
177:         component = "generated code"
178:         context = ""
179: 
180:     out.writerow(
181:         {
182:             "bench": bench,
183:             "name": name,
184:             "result": r,
185:             "component": component,
186:             "context": context,
187:             "explain": explain,
188:             "frame_time": frame_time,
189:             "backend_time": backend_time,
190:             "graph_count": graph_count,
191:             "op_count": op_count,
192:             "graph_breaks": graph_breaks,
193:             "unique_graph_breaks": unique_graph_breaks,
194:         }
195:     )
196:     i += 1  # noqa: SIM113
197: 
198: if c:
199:     print(f"failed to classify {c} entries", file=sys.stderr)
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `chunker` / 符号 `chunker`
- Symbol `normalize_file` / 符号 `normalize_file`

## Dependencies / 依赖关系
- Python imports: `csv`, `os`, `re`, `sys`, `pathlib`
- Python 导入: `csv`, `os`, `re`, `sys`, `pathlib`
