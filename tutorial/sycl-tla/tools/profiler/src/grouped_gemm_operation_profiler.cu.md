# grouped_gemm_operation_profiler.cu — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `tools/profiler/src/grouped_gemm_operation_profiler.cu`
- **Purpose (EN):** This file implements grouped GEMM for the CUTLASS profiler executable.
- **目的 (CN):** 该文件实现了面向CUTLASS profiler 可执行工具的分组 GEMM逻辑。
- **Brief / 简述:** Execution environment

## Line-by-Line Analysis / 逐行分析
### Lines 1-20
```cpp
1: /***************************************************************************************************
2:  * Copyright (c) 2025 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
3:  * SPDX-License-Identifier: BSD-3-Clause
4:  *
5:  * Redistribution and use in source and binary forms, with or without
6:  * modification, are permitted provided that the following conditions are met:
7:  *
8:  * 1. Redistributions of source code must retain the above copyright notice, this
9:  * list of conditions and the following disclaimer.
10:  *
11:  * 2. Redistributions in binary form must reproduce the above copyright notice,
12:  * this list of conditions and the following disclaimer in the documentation
13:  * and/or other materials provided with the distribution.
14:  *
15:  * 3. Neither the name of the copyright holder nor the names of its
16:  * contributors may be used to endorse or promote products derived from
17:  * this software without specific prior written permission.
18:  *
19:  * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
20:  * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
```
- **EN:** License header and copyright terms for the file.
- **CN:** 文件的许可证头和版权条款。

### Lines 21-30
```cpp
21:  * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
22:  * DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
23:  * FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
24:  * DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
25:  * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
26:  * CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
27:  * OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
28:  * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
29:  *
30:  **************************************************************************************************/
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 31-33
```cpp
31: /* \file
32:    \brief Execution environment
33: */
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 35-43
```cpp
35: #include <bitset>
36: #include <cstdint>
37: #include <iostream>
38: #include <memory>
39: #include <optional>
40: #include <stdexcept>
41: #include <string>
42: #include <vector>
43: #include <regex>
```
- **EN:** Includes the main dependencies required by this module; notable headers are `bitset`, `cstdint`, `iostream`, `memory`, `optional`.
- **CN:** 引入该模块所需的主要依赖；较重要的头文件有 `bitset`, `cstdint`, `iostream`, `memory`, `optional`。

### Lines 45-45
```cpp
45: #include <cuda_runtime_api.h>
```
- **EN:** Includes the main dependencies required by this module; notable headers are `cuda_runtime_api.h`.
- **CN:** 引入该模块所需的主要依赖；较重要的头文件有 `cuda_runtime_api.h`。

### Lines 47-52
```cpp
47: #include "cutlass/cutlass.h"
48: #include "cutlass/profiler/grouped_gemm_operation_profiler.h"
49: #include "cutlass/library/handle.h"
50: #include "cutlass/library/library.h"
51: #include "cutlass/library/operation_table.h"
52: #include "cutlass/library/singleton.h"
```
- **EN:** Includes the main dependencies required by this module; notable headers are `cutlass/cutlass.h`, `cutlass/profiler/grouped_gemm_operation_profiler.h`, `cutlass/library/handle.h`, `cutlass/library/library.h`, `cutlass/library/operation_table.h`.
- **CN:** 引入该模块所需的主要依赖；较重要的头文件有 `cutlass/cutlass.h`, `cutlass/profiler/grouped_gemm_operation_profiler.h`, `cutlass/library/handle.h`, `cutlass/library/library.h`, `cutlass/library/operation_table.h`。

### Lines 53-53
```cpp
53: /////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 55-56
```cpp
55: namespace {
56: std::vector<std::vector<int>> parseProblemSizes(std::string const& input) {
```
- **EN:** Implements `parseProblemSizes` for this file's main component.
- **CN:** 为该文件的核心组件实现 `parseProblemSizes`。

### Lines 57-58
```cpp
57:   // input must be of the form:
58:   //  `[m0xn0xk0][m1xn1xk1]` where 0, 1 are the group indexes
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 59-72
```cpp
59:   std::stringstream ss(input);
60:   std::string token;
61:   std::vector<std::vector<int>> result;
62:   while (std::getline(ss, token, ']')) {
63:     std::stringstream ss(token);
64:     std::string token;
65:     ss.get(); // discard '['
66:     std::getline(ss, token, 'x');
67:     auto m = std::stoi(token);
68:     std::getline(ss, token, 'x');
69:     auto n = std::stoi(token);
70:     std::getline(ss, token);
71:     auto k = std::stoi(token);
72:     result.push_back({m, n, k});
```
- **EN:** Implements `ss` and coordinates helper calls such as `getline`, `get`, `stoi`.
- **CN:** 实现 `ss`，并协调调用 `getline`, `get`, `stoi` 等辅助逻辑。

### Lines 73-76
```cpp
73:   }
74:   return result;
75: }
76: } // namespace
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 78-79
```cpp
78: namespace cutlass {
79: namespace profiler {
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 81-94
```cpp
81: GroupedGemmOperationProfiler::GroupedGemmOperationProfiler(Options const& options)
82:     : OperationProfiler(
83:         options,
84:         library::OperationKind::kGroupedGemm,
85:         {{ArgumentTypeID::kEnumerated,
86:           {"gemm_kind"},
87:           "Variant of GEMM (universal, gemm, planar_complex, planar_complex_array)"},
88:          {ArgumentTypeID::kInteger,
89:           {"m", "problem-size::m"},
90:           "M dimension of the GEMM problem space (for all groups)"},
91:          {ArgumentTypeID::kInteger,
92:           {"n", "problem-size::n"},
93:           "N dimension of the GEMM problem space (for all groups)"},
94:          {ArgumentTypeID::kInteger,
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 95-108
```cpp
95:           {"k", "problem-size::k"},
96:           "K dimension of the GEMM problem space (for all groups)"},
97:          {ArgumentTypeID::kInteger,
98:           {"num_groups"},
99:           "If m,n,k are specified, run a grouped GEMM with this number of groups, where each GEMM "
100:           "uses the same m,n,k values."},
101:          {ArgumentTypeID::kTensor, {"A"}, "Tensor storing the A operand"},
102:          {ArgumentTypeID::kTensor, {"B"}, "Tensor storing the B operand"},
103:          {ArgumentTypeID::kTensor, {"C"}, "Tensor storing the C operand"},
104:          {ArgumentTypeID::kTensor, {"D"}, "Tensor storing the D output"},
105:          {ArgumentTypeID::kScalar,
106:           {"alpha", "epilogue::alpha"},
107:           "Epilogue scalar alpha (applied to all GEMMs in group)."},
108:          {ArgumentTypeID::kScalar,
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 109-122
```cpp
109:           {"beta", "epilogue::beta"},
110:           "Epilogue scalar beta (applied to all GEMMs in group)."},
111:          {ArgumentTypeID::kEnumerated, {"runtime_input_datatype_a", "runtime-input-datatype::a"},
112:           "Runtime datatype (e4m3, e5m2, e3m2, e2m3, e2m1)"}, 
113:          {ArgumentTypeID::kEnumerated, {"runtime_input_datatype_b", "runtime-input-datatype::b"},
114:           "Runtime datatype (e4m3, e5m2, e3m2, e2m3, e2m1)"}, 
115:          {ArgumentTypeID::kEnumerated, {"raster_order", "raster-order"},
116:           "Raster order (heuristic, along_n, along_m)"},
117:          {ArgumentTypeID::kInteger, {"swizzle_size", "swizzle-size"}, "Size to swizzle"},
118:          {ArgumentTypeID::kEnumerated, {"use_pdl", "use_pdl"}, "Use PDL (true, false)"},
119:          {ArgumentTypeID::kScalar,
120:           {"problem-sizes"},
121:           "MxNxK Problem sizes for the grouped GEMM, where a group is enclosed by `[]`. E.g. "
122:           "--problem-sizes='[m1xn1xk1][m2xn2xk2]'"},
```
- **EN:** Declares or updates local/member state such as `sizes`.
- **CN:** 声明或更新局部/成员状态，例如 `sizes`。

### Lines 123-127
```cpp
123:          {ArgumentTypeID::kScalar,
124:           {"problem-sizes-file"},
125:           "File containing grouped GEMM problem sizes, where each line represents a group whose "
126:           "GEMM dimensions are 'mxnxk'."}},
127:         {library::Provider::kReferenceDevice}) {
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 129-131
```cpp
129:   description_ = "      Grouped matrix-matrix product. D[g] = alpha[g] * A[g] * B[g] + beta[g] * "
130:                  "C[g] for g in [0, num_groups)";
131: }
```
- **EN:** Declares or updates local/member state such as `description_`.
- **CN:** 声明或更新局部/成员状态，例如 `description_`。

### Lines 133-133
```cpp
133: GroupedGemmOperationProfiler::~GroupedGemmOperationProfiler() {}
```
- **EN:** Implements `~GroupedGemmOperationProfiler` and coordinates helper calls such as `GroupedGemmOperationProfiler`.
- **CN:** 实现 `~GroupedGemmOperationProfiler`，并协调调用 `GroupedGemmOperationProfiler` 等辅助逻辑。

### Lines 135-137
```cpp
135: void GroupedGemmOperationProfiler::print_usage(std::ostream& out) const {
136:   OperationProfiler::print_usage(out);
137: }
```
- **EN:** Formats human-readable information about command-line usage or current options.
- **CN:** 格式化命令行用法或当前选项的人类可读信息。

### Lines 139-139
```cpp
139: void GroupedGemmOperationProfiler::print_examples(std::ostream& out) const {
```
- **EN:** Implements `print_examples` for this file's main component.
- **CN:** 为该文件的核心组件实现 `print_examples`。

### Lines 141-144
```cpp
141:   out
142:     << "\nExamples:\n\n"
143:     << "Profile a particular problem size (explicit shapes):\n"
144:     << "  $ cutlass_profiler --operation=GroupedGemm --problem-sizes='[1024x1024x128][16x8x8]'\n\n"
```
- **EN:** Declares or updates local/member state such as `operation`, `sizes`.
- **CN:** 声明或更新局部/成员状态，例如 `operation`, `sizes`。

### Lines 146-147
```cpp
146:     << "Profile a particular problem size (same M, N, K for all groups):\n"
147:     << "  $ cutlass_profiler --operation=GroupedGemm --m=16 --n=32 --k=64 --num_groups=8'\n\n"
```
- **EN:** Declares or updates local/member state such as `operation`, `m`, `n`, `k`.
- **CN:** 声明或更新局部/成员状态，例如 `operation`, `m`, `n`, `k`。

### Lines 149-150
```cpp
149:     << "Profile a particular problem size from a file:\n"
150:     << "  $ cutlass_profiler --operation=GroupedGemm --problem-sizes-file=shapes.txt\n\n"
```
- **EN:** Declares or updates local/member state such as `operation`, `file`.
- **CN:** 声明或更新局部/成员状态，例如 `operation`, `file`。

### Lines 152-154
```cpp
152:     << "Schmoo over problem size and beta:\n"
153:     << "  $ cutlass_profiler --operation=GroupedGemm --problem-sizes='[8x8x8],[16x8x16][32x32x32]' "
154:        "--beta=0,1,2.5\n\n"
```
- **EN:** Declares or updates local/member state such as `operation`, `sizes`, `beta`.
- **CN:** 声明或更新局部/成员状态，例如 `operation`, `sizes`, `beta`。

### Lines 156-157
```cpp
156:     << "Schmoo over accumulator types:\n"
157:     << "  $ cutlass_profiler --operation=GroupedGemm --accumulator-type=f16,f32\n\n"
```
- **EN:** Declares or updates local/member state such as `operation`, `type`.
- **CN:** 声明或更新局部/成员状态，例如 `operation`, `type`。

### Lines 159-161
```cpp
159:     << "Run when A is f16 with column-major and B is any datatype with row-major (For column "
160:        "major, use column, col, or n. For row major use, row or t):\n"
161:     << "  $ cutlass_profiler --operation=GroupedGemm --A=f16:column --B=*:row\n\n"
```
- **EN:** Declares or updates local/member state such as `operation`, `A`, `B`.
- **CN:** 声明或更新局部/成员状态，例如 `operation`, `A`, `B`。

### Lines 163-166
```cpp
163:     << "Using various input value distribution:\n"
164:     << "  $ cutlass_profiler --operation=GroupedGemm --dist=uniform,min:0,max:3\n"
165:     << "  $ cutlass_profiler --operation=GroupedGemm --dist=gaussian,mean:0,stddev:3\n"
166:     << "  $ cutlass_profiler --operation=GroupedGemm --dist=sequential,start:0,delta:1\n\n"
```
- **EN:** Declares or updates local/member state such as `operation`, `dist`.
- **CN:** 声明或更新局部/成员状态，例如 `operation`, `dist`。

### Lines 168-174
```cpp
168:     << "Test your changes to gemm kernels with a quick functional test and save results in "
169:        "functional-test.csv:\n"
170:     << " $ cutlass_profiler  --operation=Gemm \\ \n"
171:     << "   --problem-sizes='[8x8x8][5x10x5],[16x8x16][32x32x32]' \\ \n"
172:     << "   --beta=0,1,2 --profiling-iterations=1 \\ \n"
173:     << "   --providers=cutlass --output=functional-test.csv\n\n";
174: }
```
- **EN:** Declares or updates local/member state such as `operation`, `sizes`, `beta`, `iterations`.
- **CN:** 声明或更新局部/成员状态，例如 `operation`, `sizes`, `beta`, `iterations`。

### Lines 176-179
```cpp
176: Status GroupedGemmOperationProfiler::GroupedGemmProblem::parse(
177:   library::GroupedGemmDescription const& operation_desc,
178:   ProblemSpace const& problem_space,
179:   ProblemSpace::Problem const& problem) {
```
- **EN:** Implements `parse` for this file's main component.
- **CN:** 为该文件的核心组件实现 `parse`。

### Lines 181-181
```cpp
181:   this->mode = library::GemmUniversalMode::kGrouped;
```
- **EN:** Declares or updates local/member state such as `mode`, `kGrouped`.
- **CN:** 声明或更新局部/成员状态，例如 `mode`, `kGrouped`。

### Lines 183-191
```cpp
183:   std::bitset<3> args_exist;
184:   std::string problem_sizes_str;
185:   args_exist[0] = arg_as_string(problem_sizes_str, "problem-sizes", problem_space, problem);
186:   int m, n, k;
187:   args_exist[1] = arg_as_int(m, "m", problem_space, problem) &&
188:                   arg_as_int(n, "n", problem_space, problem) &&
189:                   arg_as_int(k, "k", problem_space, problem);
190:   std::string problem_file;
191:   args_exist[2] = arg_as_string(problem_file, "problem-sizes-file", problem_space, problem);
```
- **EN:** Implements `arg_as_string` and coordinates helper calls such as `arg_as_int`.
- **CN:** 实现 `arg_as_string`，并协调调用 `arg_as_int` 等辅助逻辑。

### Lines 193-206
```cpp
193:   if (args_exist.count() == 0) {
194:     int num_groups = 8;
195:     problem_sizes.resize(num_groups);
196:     problem_sizes_3x.resize(num_groups);
197:     int m0 = 16;
198:     int n0 = 32;
199:     int k0 = 64;
200:     for (int i = 0; i < num_groups; i++) {
201:       auto m = m0 * (i + 1);
202:       auto n = n0 * (i + 1);
203:       auto k = k0 * (i + 1);
204:       problem_sizes[i] = {m, n, k};
205:       problem_sizes_3x[i] = {m, n, k};
206:     }
```
- **EN:** Declares or updates local/member state such as `num_groups`, `m0`, `n0`, `k0`.
- **CN:** 声明或更新局部/成员状态，例如 `num_groups`, `m0`, `n0`, `k0`。

### Lines 207-212
```cpp
207:   }
208:   else if (args_exist.count() > 1) {
209:     std::cerr
210:       << "Exactly one of --problem-sizes, --problem-sizes-file, or --m --n --k may be specified.\n";
211:     return Status::kErrorInvalidProblem;
212:   }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 213-213
```cpp
213:   // --problem-sizes path
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 214-226
```cpp
214:   else if (args_exist[0]) {
215:     auto problems = parseProblemSizes(problem_sizes_str);
216:     auto num_groups = problems.size();
217:     problem_sizes.resize(num_groups);
218:     problem_sizes_3x.resize(num_groups);
219:     for (size_t i = 0; i < num_groups; i++) {
220:       auto m = problems[i][0];
221:       auto n = problems[i][1];
222:       auto k = problems[i][2];
223:       problem_sizes[i] = {m, n, k};
224:       problem_sizes_3x[i] = {m, n, k};
225:     }
226:   }
```
- **EN:** Declares or updates local/member state such as `problems`, `num_groups`, `i`, `m`.
- **CN:** 声明或更新局部/成员状态，例如 `problems`, `num_groups`, `i`, `m`。

### Lines 227-227
```cpp
227:   // m, n, k path
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 228-240
```cpp
228:   else if (args_exist[1]) {
229:     int num_groups;
230:     if (!arg_as_int(num_groups, "num_groups", problem_space, problem)) {
231:       std::cerr << "num_groups must be specified if --m --n and --k are set.\n";
232:       return Status::kErrorInvalidProblem;
233:     }
234:     problem_sizes.resize(num_groups);
235:     problem_sizes_3x.resize(num_groups);
236:     for (int i = 0; i < num_groups; i++) {
237:       problem_sizes[i] = {m, n, k};
238:       problem_sizes_3x[i] = {m, n, k};
239:     }
240:   }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 241-241
```cpp
241:   // --problem-sizes-file path
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 242-246
```cpp
242:   else if (args_exist[2]) {
243:     std::ifstream file(problem_file);
244:     if (!file.good()) {
245:       throw std::runtime_error("Failed to open file: " + problem_file);
246:     }
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 247-247
```cpp
247:     // clear the problem sizes and 3x problem sizes from previous operation
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 248-249
```cpp
248:     problem_sizes.clear();
249:     problem_sizes_3x.clear();
```
- **EN:** Implements `clear` for this file's main component.
- **CN:** 为该文件的核心组件实现 `clear`。

### Lines 251-252
```cpp
251:     for (std::string line; std::getline(file, line);) {
252:       std::istringstream iss(line);
```
- **EN:** Declares or updates local/member state such as `line`.
- **CN:** 声明或更新局部/成员状态，例如 `line`。

### Lines 254-256
```cpp
254:       int m, n, k;
255:       char sep1, sep2;
256:       std::string remaining;
```
- **EN:** Declares or updates local/member state such as `k`, `sep2`, `remaining`.
- **CN:** 声明或更新局部/成员状态，例如 `k`, `sep2`, `remaining`。

### Lines 258-267
```cpp
258:       if (iss >> m >> sep1 >> n >> sep2 >> k && sep1 == 'x' && sep2 == 'x' && !(iss >> remaining)) {
259:         problem_sizes.emplace_back(m, n, k);
260:         problem_sizes_3x.emplace_back(m, n, k);
261:       }
262:       else {
263:         throw std::runtime_error(
264:           "Invalid format in line: " + line + ". Each line in file expected to be 'mxnxk'.");
265:       }
266:     }
267:   }
```
- **EN:** Declares or updates local/member state such as `sep1`, `sep2`.
- **CN:** 声明或更新局部/成员状态，例如 `sep1`, `sep2`。

### Lines 269-269
```cpp
269:   if (!arg_as_int(this->cluster_m, "cluster_m", problem_space, problem)) {
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 270-270
```cpp
270:     // default value
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 271-272
```cpp
271:     this->cluster_m = std::string(operation_desc.gemm.name).find("_2sm") != std::string::npos ? 2 : 1;
272:   }
```
- **EN:** Declares or updates local/member state such as `cluster_m`.
- **CN:** 声明或更新局部/成员状态，例如 `cluster_m`。

### Lines 274-274
```cpp
274:   if (!arg_as_int(this->cluster_n, "cluster_n", problem_space, problem)) {
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 275-275
```cpp
275:     // default value
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 276-277
```cpp
276:     this->cluster_n = 1;
277:   }
```
- **EN:** Declares or updates local/member state such as `cluster_n`.
- **CN:** 声明或更新局部/成员状态，例如 `cluster_n`。

### Lines 279-279
```cpp
279:   if (!arg_as_int(this->cluster_k, "cluster_k", problem_space, problem)) {
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 280-280
```cpp
280:     // default value
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 281-282
```cpp
281:     this->cluster_k = 1;
282:   }
```
- **EN:** Declares or updates local/member state such as `cluster_k`.
- **CN:** 声明或更新局部/成员状态，例如 `cluster_k`。

### Lines 284-284
```cpp
284:   if (!arg_as_int(this->cluster_m_fallback, "cluster_m_fallback", problem_space, problem)) {
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 285-285
```cpp
285:     // default value
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 286-287
```cpp
286:     this->cluster_m_fallback = (this->cluster_m % 2 == 0) ? 2 : 1;
287:   }
```
- **EN:** Declares or updates local/member state such as `cluster_m_fallback`.
- **CN:** 声明或更新局部/成员状态，例如 `cluster_m_fallback`。

### Lines 289-289
```cpp
289:   if (!arg_as_int(this->cluster_n_fallback, "cluster_n_fallback", problem_space, problem)) {
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 290-290
```cpp
290:     // default value
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 291-292
```cpp
291:     this->cluster_n_fallback = 1;
292:   }
```
- **EN:** Declares or updates local/member state such as `cluster_n_fallback`.
- **CN:** 声明或更新局部/成员状态，例如 `cluster_n_fallback`。

### Lines 294-294
```cpp
294:   if (!arg_as_int(this->cluster_k_fallback, "cluster_k_fallback", problem_space, problem)) {
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 295-295
```cpp
295:     // default value
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 296-297
```cpp
296:     this->cluster_k_fallback = 1;
297:   }
```
- **EN:** Declares or updates local/member state such as `cluster_k_fallback`.
- **CN:** 声明或更新局部/成员状态，例如 `cluster_k_fallback`。

### Lines 299-299
```cpp
299:   this->mode = library::GemmUniversalMode::kGrouped;
```
- **EN:** Declares or updates local/member state such as `mode`, `kGrouped`.
- **CN:** 声明或更新局部/成员状态，例如 `mode`, `kGrouped`。

### Lines 301-303
```cpp
301:   if (!tensor_description_satisfies(operation_desc.gemm.A, "A", problem_space, problem)) {
302:     return Status::kErrorInvalidProblem;
303:   }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 305-307
```cpp
305:   if (!tensor_description_satisfies(operation_desc.gemm.B, "B", problem_space, problem)) {
306:     return Status::kErrorInvalidProblem;
307:   }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 309-311
```cpp
309:   if (!tensor_description_satisfies(operation_desc.gemm.C, "C", problem_space, problem)) {
310:     return Status::kErrorInvalidProblem;
311:   }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 313-315
```cpp
313:   if (!tensor_description_satisfies(operation_desc.gemm.D, "D", problem_space, problem)) {
314:     return Status::kErrorInvalidProblem;
315:   }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 317-317
```cpp
317:   if (!arg_as_bool(this->use_pdl, "use_pdl", problem_space, problem)) {
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 318-318
```cpp
318:     // default value
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 319-320
```cpp
319:     this->use_pdl = false;
320:   }
```
- **EN:** Declares or updates local/member state such as `use_pdl`, `false`.
- **CN:** 声明或更新局部/成员状态，例如 `use_pdl`, `false`。

### Lines 322-322
```cpp
322:   if (!arg_as_RuntimeDatatype(this->runtime_input_datatype_a, "runtime_input_datatype_a", problem_space, problem)) {
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 323-323
```cpp
323:     // default value
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 324-325
```cpp
324:     this->runtime_input_datatype_a = cutlass::library::RuntimeDatatype::kStatic;
325:   }
```
- **EN:** Declares or updates local/member state such as `runtime_input_datatype_a`, `kStatic`.
- **CN:** 声明或更新局部/成员状态，例如 `runtime_input_datatype_a`, `kStatic`。

### Lines 327-327
```cpp
327:   if (!arg_as_RuntimeDatatype(this->runtime_input_datatype_b, "runtime_input_datatype_b", problem_space, problem)) {
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 328-328
```cpp
328:     // default value
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 329-330
```cpp
329:     this->runtime_input_datatype_b = cutlass::library::RuntimeDatatype::kStatic;
330:   }
```
- **EN:** Declares or updates local/member state such as `runtime_input_datatype_b`, `kStatic`.
- **CN:** 声明或更新局部/成员状态，例如 `runtime_input_datatype_b`, `kStatic`。

### Lines 332-332
```cpp
332:   if (!arg_as_int(this->swizzle_size, "swizzle_size", problem_space, problem)) {
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 333-333
```cpp
333:     // default value
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 334-335
```cpp
334:     this->swizzle_size = 1;
335:   }
```
- **EN:** Declares or updates local/member state such as `swizzle_size`.
- **CN:** 声明或更新局部/成员状态，例如 `swizzle_size`。

### Lines 337-337
```cpp
337:   if (!arg_as_RasterOrder(this->raster_order, "raster_order", problem_space, problem)) {
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 338-338
```cpp
338:     // default value
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 339-340
```cpp
339:     this->raster_order = library::RasterOrder::kHeuristic;
340:   }
```
- **EN:** Declares or updates local/member state such as `raster_order`, `kHeuristic`.
- **CN:** 声明或更新局部/成员状态，例如 `raster_order`, `kHeuristic`。

### Lines 342-347
```cpp
342:   if (!arg_as_scalar(
343:         this->alpha,
344:         operation_desc.gemm.element_epilogue,
345:         "alpha",
346:         problem_space,
347:         problem)) {
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 349-352
```cpp
349:     if (!cast_from_double(this->alpha, operation_desc.gemm.element_epilogue, 1)) {
350:       return Status::kErrorInternal;
351:     }
352:   }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 354-359
```cpp
354:   if (!arg_as_scalar(
355:         this->beta,
356:         operation_desc.gemm.element_epilogue,
357:         "beta",
358:         problem_space,
359:         problem)) {
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 361-364
```cpp
361:     if (!cast_from_double(this->beta, operation_desc.gemm.element_epilogue, 0)) {
362:       return Status::kErrorInternal;
363:     }
364:   }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 366-374
```cpp
366:   auto num_groups = problem_sizes.size();
367:   this->lda.resize(num_groups);
368:   this->ldb.resize(num_groups);
369:   this->ldc.resize(num_groups);
370:   for (size_t group_idx = 0; group_idx < num_groups; group_idx++) {
371:     this->lda[group_idx] = DeviceAllocation::get_packed_layout(
372:                              operation_desc.gemm.A.layout,
373:                              {int(this->m(group_idx)), int(this->k(group_idx))})
374:                              .front();
```
- **EN:** Implements `size` and coordinates helper calls such as `resize`, `get_packed_layout`, `int`.
- **CN:** 实现 `size`，并协调调用 `resize`, `get_packed_layout`, `int` 等辅助逻辑。

### Lines 376-379
```cpp
376:     this->ldb[group_idx] = DeviceAllocation::get_packed_layout(
377:                              operation_desc.gemm.B.layout,
378:                              {int(this->k(group_idx)), int(this->n(group_idx))})
379:                              .front();
```
- **EN:** Implements `front` and coordinates helper calls such as `get_packed_layout`, `int`, `k`.
- **CN:** 实现 `front`，并协调调用 `get_packed_layout`, `int`, `k` 等辅助逻辑。

### Lines 381-385
```cpp
381:     this->ldc[group_idx] = DeviceAllocation::get_packed_layout(
382:                              operation_desc.gemm.C.layout,
383:                              {int(this->m(group_idx)), int(this->n(group_idx))})
384:                              .front();
385:   }
```
- **EN:** Implements `front` and coordinates helper calls such as `get_packed_layout`, `int`, `m`.
- **CN:** 实现 `front`，并协调调用 `get_packed_layout`, `int`, `m` 等辅助逻辑。

### Lines 387-387
```cpp
387:   // instantiation for exploration profiling
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 388-398
```cpp
388:   this->raster_orders = {
389:     cutlass::library::RasterOrder::kAlongN,
390:     cutlass::library::RasterOrder::kAlongM
391:   };
392:   this->swizzle_sizes = {1, 2, 4, 8};
393:   this->preferred_clusters = {
394:     {1, 1, 1}, {2, 1, 1}, {2, 2, 1}, {4, 1, 1}, {4, 2, 1}, {4, 4, 1}, {8, 2, 1}
395:   };
396:   this->fallback_clusters = {
397:     {1, 1, 1}, {2, 1, 1}, {2, 2, 1}
398:   };
```
- **EN:** Declares or updates local/member state such as `raster_orders`, `swizzle_sizes`, `preferred_clusters`, `fallback_clusters`.
- **CN:** 声明或更新局部/成员状态，例如 `raster_orders`, `swizzle_sizes`, `preferred_clusters`, `fallback_clusters`。

### Lines 400-401
```cpp
400:   return Status::kSuccess;
401: }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 403-403
```cpp
403: /// Total number of bytes loaded
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 404-405
```cpp
404: int64_t GroupedGemmOperationProfiler::GroupedGemmProblem::bytes(
405:   library::GroupedGemmDescription const& operation_desc) const {
```
- **EN:** Implements `bytes` for this file's main component.
- **CN:** 为该文件的核心组件实现 `bytes`。

### Lines 406-406
```cpp
406:   // Input bytes read and Output bytes written for the gemm problem
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 407-409
```cpp
407:   int64_t bytes = 0;
408:   for (size_t group_idx = 0, num_groups = problem_sizes.size(); group_idx < num_groups;
409:        group_idx++) {
```
- **EN:** Declares or updates local/member state such as `bytes`, `group_idx`, `num_groups`.
- **CN:** 声明或更新局部/成员状态，例如 `bytes`, `group_idx`, `num_groups`。

### Lines 411-411
```cpp
411:     // If M = 0 or N = 0, no tiles are scheduled and no bytes are loaded for the group
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 412-414
```cpp
412:     if (m(group_idx) * n(group_idx) == 0) {
413:       continue;
414:     }
```
- **EN:** Declares or updates local/member state such as `continue`.
- **CN:** 声明或更新局部/成员状态，例如 `continue`。

### Lines 416-419
```cpp
416:     bytes +=
417:       int64_t(library::sizeof_bits(operation_desc.gemm.A.element) * m(group_idx) / 8) * k(group_idx) +
418:       int64_t(library::sizeof_bits(operation_desc.gemm.B.element) * n(group_idx) / 8) * k(group_idx) +
419:       int64_t(library::sizeof_bits(operation_desc.gemm.C.element) * m(group_idx) / 8) * n(group_idx);
```
- **EN:** Implements `int64_t` and coordinates helper calls such as `sizeof_bits`, `m`, `k`.
- **CN:** 实现 `int64_t`，并协调调用 `sizeof_bits`, `m`, `k` 等辅助逻辑。

### Lines 421-421
```cpp
421:     // Set is_beta_zero true if beta is zero
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 422-422
```cpp
422:     bool is_beta_zero = std::all_of(beta.begin(), beta.end(), [](uint8_t i) { return i == 0; });
```
- **EN:** Implements `all_of` and coordinates helper calls such as `begin`, `end`.
- **CN:** 实现 `all_of`，并协调调用 `begin`, `end` 等辅助逻辑。

### Lines 423-423
```cpp
423:     // Output bytes read for the gemm problem for non-zero beta values
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 424-428
```cpp
424:     if (!is_beta_zero) {
425:       bytes +=
426:         int64_t(library::sizeof_bits(operation_desc.gemm.C.element) * m(group_idx) / 8) * n(group_idx);
427:     }
428:   }
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 430-431
```cpp
430:   return bytes;
431: }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 433-433
```cpp
433: /// Total number of flops computed
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 434-441
```cpp
434: int64_t GroupedGemmOperationProfiler::GroupedGemmProblem::flops(
435:   library::GroupedGemmDescription const& operation_desc) const {
436:   int64_t flops_ = 0;
437:   for (size_t group_idx = 0, num_groups = problem_sizes.size(); group_idx < num_groups;
438:        group_idx++) {
439:     flops_ +=
440:       (int64_t(m(group_idx)) * n(group_idx) * k(group_idx) + m(group_idx) * n(group_idx)) * 2;
441:   }
```
- **EN:** Implements `flops` and coordinates helper calls such as `size`, `int64_t`, `m`.
- **CN:** 实现 `flops`，并协调调用 `size`, `int64_t`, `m` 等辅助逻辑。

### Lines 443-443
```cpp
443:   // complex-valued support
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 444-451
```cpp
444:   switch (operation_desc.gemm.tile_description.math_instruction.math_operation) {
445:   case library::MathOperationID::kMultiplyAddComplex:
446:   case library::MathOperationID::kMultiplyAddComplexFastF32:
447:     flops_ *= 4;
448:     break;
449:   case library::MathOperationID::kMultiplyAddGaussianComplex:
450:     flops_ *= 3;
451:     break;
```
- **EN:** Declares or updates local/member state such as `break`.
- **CN:** 声明或更新局部/成员状态，例如 `break`。

### Lines 453-455
```cpp
453:   default:
454:     break;
455:   }
```
- **EN:** Declares or updates local/member state such as `break`.
- **CN:** 声明或更新局部/成员状态，例如 `break`。

### Lines 457-458
```cpp
457:   return flops_;
458: }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 460-460
```cpp
460: /// Initializes a performance result
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 461-464
```cpp
461: void GroupedGemmOperationProfiler::GroupedGemmProblem::initialize_result(
462:   PerformanceResult& result,
463:   library::GroupedGemmDescription const& operation_desc,
464:   ProblemSpace const& problem_space) {
```
- **EN:** Initializes or registers grouped GEMM components for later lookup or execution.
- **CN:** 初始化或注册分组 GEMM组件，以便后续查找或执行。

### Lines 466-466
```cpp
466:   result.arguments.resize(problem_space.rank());
```
- **EN:** Implements `resize` and coordinates helper calls such as `rank`.
- **CN:** 实现 `resize`，并协调调用 `rank` 等辅助逻辑。

### Lines 468-472
```cpp
468:   set_argument(
469:     result,
470:     "gemm_kind",
471:     problem_space,
472:     library::to_string(operation_desc.gemm.gemm_kind));
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 474-479
```cpp
474:   set_argument(
475:     result,
476:     "A",
477:     problem_space,
478:     std::string(library::to_string(operation_desc.gemm.A.element)) + ":" +
479:       library::to_string(operation_desc.gemm.A.layout));
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 481-486
```cpp
481:   set_argument(
482:     result,
483:     "B",
484:     problem_space,
485:     std::string(library::to_string(operation_desc.gemm.B.element)) + ":" +
486:       library::to_string(operation_desc.gemm.B.layout));
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 488-493
```cpp
488:   set_argument(
489:     result,
490:     "C",
491:     problem_space,
492:     std::string(library::to_string(operation_desc.gemm.C.element)) + ":" +
493:       library::to_string(operation_desc.gemm.C.layout));
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 495-500
```cpp
495:   set_argument(
496:     result,
497:     "D",
498:     problem_space,
499:     std::string(library::to_string(operation_desc.gemm.D.element)) + ":" +
500:       library::to_string(operation_desc.gemm.D.layout));
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 502-515
```cpp
502:   {
503:     std::stringstream ss;
504:     ss << "'";
505:     for (auto const& problem_size : problem_sizes) {
506:       ss << "[";
507:       auto m = problem_size[0];
508:       auto n = problem_size[1];
509:       auto k = problem_size[2];
510:       ss << m << "x" << n << "x" << k;
511:       ss << "]";
512:     }
513:     ss << "'";
514:     set_argument(result, "problem-sizes", problem_space, ss.str());
515:   }
```
- **EN:** Declares or updates local/member state such as `ss`, `m`, `n`, `k`.
- **CN:** 声明或更新局部/成员状态，例如 `ss`, `m`, `n`, `k`。

### Lines 517-524
```cpp
517:   auto cluster_shape = operation_desc.gemm.tile_description.cluster_shape;
518:   auto is_dynamic = cluster_shape.m() == 0 || cluster_shape.n() == 0 || cluster_shape.k() == 0;
519:   set_argument(result, "cluster_m", problem_space, is_dynamic ? this->cluster_m : cluster_shape.m());
520:   set_argument(result, "cluster_n", problem_space, is_dynamic ? this->cluster_n : cluster_shape.n());
521:   set_argument(result, "cluster_k", problem_space, is_dynamic ? this->cluster_k : cluster_shape.k());
522:   set_argument(result, "cluster_m_fallback", problem_space, cluster_m_fallback);
523:   set_argument(result, "cluster_n_fallback", problem_space, cluster_n_fallback);
524:   set_argument(result, "cluster_k_fallback", problem_space, cluster_k_fallback);
```
- **EN:** Implements `set_argument` and coordinates helper calls such as `m`, `n`, `k`.
- **CN:** 实现 `set_argument`，并协调调用 `m`, `n`, `k` 等辅助逻辑。

### Lines 526-528
```cpp
526:   set_argument(result, "raster_order", problem_space, library::to_string(raster_order));
527:   set_argument(result, "swizzle_size", problem_space, swizzle_size);
528:   set_argument(result, "use_pdl", problem_space, library::to_string(use_pdl));
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 530-531
```cpp
530:   set_argument(result, "runtime_input_datatype_a", problem_space, library::to_string(runtime_input_datatype_a));
531:   set_argument(result, "runtime_input_datatype_b", problem_space, library::to_string(runtime_input_datatype_b));
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 533-537
```cpp
533:   set_argument(
534:     result,
535:     "alpha",
536:     problem_space,
537:     library::lexical_cast(alpha, operation_desc.gemm.element_epilogue));
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 539-544
```cpp
539:   set_argument(
540:     result,
541:     "beta",
542:     problem_space,
543:     library::lexical_cast(beta, operation_desc.gemm.element_epilogue));
544: }
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 546-555
```cpp
546: void GroupedGemmOperationProfiler::update_workspace_and_result_(
547:   GroupedGemmWorkspace &gemm_workspace,
548:   PerformanceResult &result,
549:   ProblemSpace const &problem_space,
550:   cutlass::library::RasterOrder const &raster_order,
551:   std::array<int64_t, 3> const &preferred_cluster,
552:   std::array<int64_t, 3> const &fallback_cluster,
553:   int swizzle_size,
554:   bool is_dynamic_cluster_enabled
555: ) {
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 557-558
```cpp
557:   gemm_workspace.arguments.swizzle_size = swizzle_size;
558:   gemm_workspace.arguments.raster_order = raster_order;
```
- **EN:** Declares or updates local/member state such as `swizzle_size`, `raster_order`.
- **CN:** 声明或更新局部/成员状态，例如 `swizzle_size`, `raster_order`。

### Lines 560-561
```cpp
560:   set_argument(result, "raster_order", problem_space, library::to_string(raster_order));
561:   set_argument(result, "swizzle_size", problem_space, swizzle_size);
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 563-573
```cpp
563:   if (is_dynamic_cluster_enabled) {
564:     gemm_workspace.arguments.cluster_shape = {int(preferred_cluster[0]), int(preferred_cluster[1]), int(preferred_cluster[2])};
565:     gemm_workspace.arguments.cluster_shape_fallback = {int(fallback_cluster[0]), int(fallback_cluster[1]), int(fallback_cluster[2])};
566:     set_argument(result, "cluster_m", problem_space, preferred_cluster[0]);
567:     set_argument(result, "cluster_n", problem_space, preferred_cluster[1]);
568:     set_argument(result, "cluster_k", problem_space, preferred_cluster[2]);
569:     set_argument(result, "cluster_m_fallback", problem_space, fallback_cluster[0]);
570:     set_argument(result, "cluster_n_fallback", problem_space, fallback_cluster[1]);
571:     set_argument(result, "cluster_k_fallback", problem_space, fallback_cluster[2]);
572:   }
573: }
```
- **EN:** Declares or updates local/member state such as `cluster_shape`, `cluster_shape_fallback`.
- **CN:** 声明或更新局部/成员状态，例如 `cluster_shape`, `cluster_shape_fallback`。

### Lines 575-575
```cpp
575: /////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 577-577
```cpp
577: /// Extracts the problem dimensions
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 578-584
```cpp
578: Status GroupedGemmOperationProfiler::initialize_configuration(
579:   Options const& options,
580:   PerformanceReport& report,
581:   DeviceContext& device_context,
582:   library::Operation const* operation,
583:   ProblemSpace const& problem_space,
584:   ProblemSpace::Problem const& problem) {
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 586-587
```cpp
586:   library::GroupedGemmDescription const& operation_desc =
587:     static_cast<library::GroupedGemmDescription const&>(operation->description());
```
- **EN:** Implements `description` for this file's main component.
- **CN:** 为该文件的核心组件实现 `description`。

### Lines 589-592
```cpp
589:   // We want to share the same operation profiler for any grouped gemm operation.
590:   // We distinguish between block scaled and non-block scaled operations by looking at the kernel
591:   // name, which tells us what reference kernel to use, which arguments to pass to the operation
592:   // etc. This avoids creating yet another OperationProfiler with a lot of boilerplate in it.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 594-598
```cpp
594:   std::string sf_tuple = "\\d+x\\d+";
595:   std::string datatypes_regex = "\\w?f\\d+|e\\dm\\d"; // bf16 | f16 | f32 | e4m3 | ...
596:   std::string blockwise_regex_string = sf_tuple + "(" +  datatypes_regex + ")x(" + 
597:                                        datatypes_regex + ")_" + sf_tuple + "(" + 
598:                                        datatypes_regex + ")x(" + datatypes_regex + ")";
```
- **EN:** Declares or updates local/member state such as `sf_tuple`, `datatypes_regex`, `blockwise_regex_string`.
- **CN:** 声明或更新局部/成员状态，例如 `sf_tuple`, `datatypes_regex`, `blockwise_regex_string`。

### Lines 600-611
```cpp
600:   if (std::string(operation_desc.gemm.name).find("bstensor") != std::string::npos) {
601:     is_block_scaled = true;
602:     gemm_workspace_.block_scales = BlockScalingWorkspace{};
603:   }
604:   else if (std::regex_search(operation_desc.gemm.name, std::regex(blockwise_regex_string))) {
605:     is_blockwise = true;
606:     gemm_workspace_.block_scales = BlockScalingWorkspace{};
607:   }
608:   else {
609:     is_block_scaled = false;
610:     gemm_workspace_.block_scales = std::nullopt;
611:   }
```
- **EN:** Declares or updates local/member state such as `is_block_scaled`, `true`, `block_scales`, `is_blockwise`.
- **CN:** 声明或更新局部/成员状态，例如 `is_block_scaled`, `true`, `block_scales`, `is_blockwise`。

### Lines 613-615
```cpp
613:   if (operation_desc.gemm.gemm_kind != library::GemmKind::kGrouped) {
614:     return Status::kErrorInvalidProblem;
615:   }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 617-620
```cpp
617:   Status status = problem_.parse(operation_desc, problem_space, problem);
618:   if (status != Status::kSuccess) {
619:     return status;
620:   }
```
- **EN:** Implements `parse` for this file's main component.
- **CN:** 为该文件的核心组件实现 `parse`。

### Lines 622-628
```cpp
622:   auto num_groups = problem_.problem_sizes.size();
623:   auto& config = gemm_workspace_.configuration;
624:   config.problem_count = num_groups;
625:   config.lda = problem_.lda.data();
626:   config.ldb = problem_.ldb.data();
627:   config.ldc = problem_.ldc.data();
628:   config.problem_sizes_3x_host = problem_.problem_sizes_3x.data();
```
- **EN:** Implements `size` and coordinates helper calls such as `data`.
- **CN:** 实现 `size`，并协调调用 `data` 等辅助逻辑。

### Lines 630-631
```cpp
630:   gemm_workspace_.arguments.swizzle_size = problem_.swizzle_size;
631:   gemm_workspace_.arguments.raster_order = problem_.raster_order;
```
- **EN:** Declares or updates local/member state such as `swizzle_size`, `raster_order`.
- **CN:** 声明或更新局部/成员状态，例如 `swizzle_size`, `raster_order`。

### Lines 633-634
```cpp
633:   gemm_workspace_.arguments.runtime_input_datatype_a = problem_.runtime_input_datatype_a;
634:   gemm_workspace_.arguments.runtime_input_datatype_b = problem_.runtime_input_datatype_b;
```
- **EN:** Declares or updates local/member state such as `runtime_input_datatype_a`, `runtime_input_datatype_b`.
- **CN:** 声明或更新局部/成员状态，例如 `runtime_input_datatype_a`, `runtime_input_datatype_b`。

### Lines 636-636
```cpp
636:   gemm_workspace_.arguments.use_pdl = problem_.use_pdl;
```
- **EN:** Declares or updates local/member state such as `use_pdl`.
- **CN:** 声明或更新局部/成员状态，例如 `use_pdl`。

### Lines 638-638
```cpp
638:   cudaStreamCreateWithFlags(&gemm_workspace_.stream, cudaStreamNonBlocking);
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 640-640
```cpp
640:   initialize_result_(this->model_result_, options, operation_desc, problem_space);
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 642-643
```cpp
642:   return status;
643: }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 645-645
```cpp
645: /// Initializes the performance result
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 646-650
```cpp
646: void GroupedGemmOperationProfiler::initialize_result_(
647:   PerformanceResult& result,
648:   Options const& options,
649:   library::GroupedGemmDescription const& operation_desc,
650:   ProblemSpace const& problem_space) {
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 652-655
```cpp
652:   result.provider = library::Provider::kCUTLASS;
653:   result.disposition = Disposition::kNotRun;
654:   result.status = Status::kSuccess;
655:   result.operation_name = operation_desc.gemm.name;
```
- **EN:** Declares or updates local/member state such as `provider`, `kCUTLASS`, `disposition`, `kNotRun`.
- **CN:** 声明或更新局部/成员状态，例如 `provider`, `kCUTLASS`, `disposition`, `kNotRun`。

### Lines 657-657
```cpp
657:   problem_.initialize_result(result, operation_desc, problem_space);
```
- **EN:** Initializes or registers grouped GEMM components for later lookup or execution.
- **CN:** 初始化或注册分组 GEMM组件，以便后续查找或执行。

### Lines 659-659
```cpp
659:   OperationProfiler::initialize_result_(result, operation_desc, problem_space);
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 661-664
```cpp
661:   result.bytes = problem_.bytes(operation_desc);
662:   result.flops = problem_.flops(operation_desc);
663:   result.runtime = 0;
664:   result.runtime_vector.resize(options.device.devices.size(), 0);
```
- **EN:** Implements `bytes` and coordinates helper calls such as `flops`, `resize`, `size`.
- **CN:** 实现 `bytes`，并协调调用 `flops`, `resize`, `size` 等辅助逻辑。

### Lines 666-666
```cpp
666: }
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 668-668
```cpp
668: /// Initializes workspace
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 669-675
```cpp
669: Status GroupedGemmOperationProfiler::initialize_workspace(
670:   Options const& options,
671:   PerformanceReport& report,
672:   DeviceContext& device_context,
673:   library::Operation const* operation,
674:   ProblemSpace const& problem_space,
675:   ProblemSpace::Problem const& problem) {
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 677-680
```cpp
677:   if (options.device.devices.size() != 1) {
678:     throw std::runtime_error("This operation profiler only supports a single "
679:                              "device.");
680:   }
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 682-686
```cpp
682:   cudaError_t result;
683:   result = cudaSetDevice(options.device.device_id(0));
684:   if (result != cudaSuccess) {
685:     throw std::runtime_error("cudaSetDevice() failed.");
686:   }
```
- **EN:** Implements `cudaSetDevice` and coordinates helper calls such as `device_id`, `runtime_error`.
- **CN:** 实现 `cudaSetDevice`，并协调调用 `device_id`, `runtime_error` 等辅助逻辑。

### Lines 688-690
```cpp
688:   library::Operation const* underlying_operation = operation;
689:   library::GroupedGemmDescription const& operation_desc =
690:     static_cast<library::GroupedGemmDescription const&>(operation->description());
```
- **EN:** Implements `description` for this file's main component.
- **CN:** 为该文件的核心组件实现 `description`。

### Lines 692-692
```cpp
692:   // Compute the number of copies of the problem to avoid L2 camping.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 693-705
```cpp
693:   if (!options.profiling.workspace_count) {
694:     int64_t bytes = problem_.bytes(operation_desc);
695:     if (bytes < 3 * int64_t(options.device.properties[0].l2CacheSize)) {
696:       gemm_workspace_.problem_count =
697:         1 + int((3 * int64_t(options.device.properties[0].l2CacheSize)) / bytes);
698:     }
699:     else {
700:       gemm_workspace_.problem_count = 1;
701:     }
702:   }
703:   else {
704:     gemm_workspace_.problem_count = options.profiling.workspace_count;
705:   }
```
- **EN:** Declares or updates local/member state such as `bytes`, `problem_count`, `workspace_count`.
- **CN:** 声明或更新局部/成员状态，例如 `bytes`, `problem_count`, `workspace_count`。

### Lines 707-709
```cpp
707:   bool allocate_device_tensors = options.execution_mode != ExecutionMode::kDryRun;
708:   if (allocate_device_tensors) {
709:     size_t num_groups = problem_.problem_sizes.size();
```
- **EN:** Declares or updates local/member state such as `allocate_device_tensors`, `kDryRun`, `num_groups`.
- **CN:** 声明或更新局部/成员状态，例如 `allocate_device_tensors`, `kDryRun`, `num_groups`。

### Lines 710-710
```cpp
710:     // input data
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 711-724
```cpp
711:     gemm_workspace_.A_ptr_array_host.resize(num_groups);
712:     gemm_workspace_.B_ptr_array_host.resize(num_groups);
713:     gemm_workspace_.C_ptr_array_host.resize(num_groups);
714:     gemm_workspace_.D_ptr_array_host.resize(num_groups);
715:     if (is_block_scaled) {
716:       auto& block_scaling_ws = gemm_workspace_.block_scales.value();
717:       block_scaling_ws.SFA_ptr_array_host.resize(num_groups);
718:       block_scaling_ws.SFB_ptr_array_host.resize(num_groups);
719:       block_scaling_ws.SFC_ptr_array_host.resize(num_groups);
720:       block_scaling_ws.SFD_ptr_array_host.resize(num_groups);
721:       block_scaling_ws.SFD_reference_ptr_array_host.resize(num_groups);
722:     }
723:     else if (is_blockwise) {
724:       auto& block_scaling_ws = gemm_workspace_.block_scales.value();
```
- **EN:** Implements `resize` and coordinates helper calls such as `value`.
- **CN:** 实现 `resize`，并协调调用 `value` 等辅助逻辑。

### Lines 725-728
```cpp
725:       block_scaling_ws.SFA_ptr_array_host.resize(num_groups);
726:       block_scaling_ws.SFB_ptr_array_host.resize(num_groups);
727:       block_scaling_ws.SFC_ptr_array_host.resize(num_groups);
728:     }
```
- **EN:** Implements `resize` for this file's main component.
- **CN:** 为该文件的核心组件实现 `resize`。

### Lines 729-729
```cpp
729:     static_assert(sizeof(void*) == 8); // allocating blocks for pointers, so verify pointer size
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 730-730
```cpp
730:     // ldx
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 731-742
```cpp
731:     gemm_workspace_.lda_array_device =
732:       device_context
733:         .allocate_block(options, "lda_array", library::NumericTypeID::kS64, num_groups, 0);
734:     gemm_workspace_.ldb_array_device =
735:       device_context
736:         .allocate_block(options, "ldb_array", library::NumericTypeID::kS64, num_groups, 0);
737:     gemm_workspace_.ldc_array_device =
738:       device_context
739:         .allocate_block(options, "ldc_array", library::NumericTypeID::kS64, num_groups, 0);
740:     gemm_workspace_.lda_array_device->copy_from_host(problem_.lda.data());
741:     gemm_workspace_.ldb_array_device->copy_from_host(problem_.ldb.data());
742:     gemm_workspace_.ldc_array_device->copy_from_host(problem_.ldc.data());
```
- **EN:** Implements `allocate_block` and coordinates helper calls such as `copy_from_host`, `data`.
- **CN:** 实现 `allocate_block`，并协调调用 `copy_from_host`, `data` 等辅助逻辑。

### Lines 743-743
```cpp
743:     // problem sizes
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 744-750
```cpp
744:     gemm_workspace_.problem_sizes_array_device = device_context.allocate_block(
745:       options,
746:       "problem_sizes_array",
747:       library::NumericTypeID::kU8,
748:       num_groups * sizeof(gemm::GemmCoord),
749:       0);
750:     gemm_workspace_.problem_sizes_array_device->copy_from_host(problem_.problem_sizes.data());
```
- **EN:** Declares or updates local/member state such as `problem_sizes_array_device`.
- **CN:** 声明或更新局部/成员状态，例如 `problem_sizes_array_device`。

### Lines 752-758
```cpp
752:     gemm_workspace_.problem_sizes_3x_array_device = device_context.allocate_block(
753:       options,
754:       "problem_sizes_array_3x",
755:       library::NumericTypeID::kU8,
756:       num_groups * sizeof(cute::Shape<int, int, int>),
757:       0);
758:     gemm_workspace_.problem_sizes_3x_array_device->copy_from_host(problem_.problem_sizes_3x.data());
```
- **EN:** Declares or updates local/member state such as `problem_sizes_3x_array_device`.
- **CN:** 声明或更新局部/成员状态，例如 `problem_sizes_3x_array_device`。

### Lines 760-760
```cpp
760:     // reference
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 761-761
```cpp
761:     gemm_workspace_.reference_ptr_array_host.resize(num_groups);
```
- **EN:** Implements `resize` for this file's main component.
- **CN:** 为该文件的核心组件实现 `resize`。

### Lines 763-776
```cpp
763:     int seed_shift = 0;
764:     for (size_t group_idx = 0; group_idx < num_groups; group_idx++) {
765:       auto group_str = std::to_string(group_idx);
766:       gemm_workspace_.A_ptr_array_host[group_idx] = device_context.allocate_and_initialize_tensor(
767:         options,
768:         "A_" + group_str,
769:         operation_desc.gemm.A.element,
770:         operation_desc.gemm.A.layout,
771:         {int(problem_.m(group_idx)), int(problem_.k(group_idx))},
772:         {int(problem_.lda[group_idx])},
773:         gemm_workspace_.problem_count,
774:         seed_shift++,
775:         0);
776:       gemm_workspace_.B_ptr_array_host[group_idx] = device_context.allocate_and_initialize_tensor(
```
- **EN:** Implements `to_string` and coordinates helper calls such as `allocate_and_initialize_tensor`, `int`, `m`.
- **CN:** 实现 `to_string`，并协调调用 `allocate_and_initialize_tensor`, `int`, `m` 等辅助逻辑。

### Lines 777-790
```cpp
777:         options,
778:         "B_" + group_str,
779:         operation_desc.gemm.B.element,
780:         operation_desc.gemm.B.layout,
781:         {int(problem_.k(group_idx)), int(problem_.n(group_idx))},
782:         {int(problem_.ldb[group_idx])},
783:         gemm_workspace_.problem_count,
784:         seed_shift++,
785:         0);
786:       gemm_workspace_.C_ptr_array_host[group_idx] = device_context.allocate_and_initialize_tensor(
787:         options,
788:         "C_" + group_str,
789:         operation_desc.gemm.C.element,
790:         operation_desc.gemm.C.layout,
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 791-804
```cpp
791:         {int(problem_.m(group_idx)), int(problem_.n(group_idx))},
792:         {int(problem_.ldc[group_idx])},
793:         gemm_workspace_.problem_count,
794:         seed_shift++,
795:         0);
796:       gemm_workspace_.D_ptr_array_host[group_idx] = device_context.allocate_tensor(
797:         options,
798:         "D_" + group_str,
799:         operation_desc.gemm.D.element,
800:         operation_desc.gemm.D.layout,
801:         {int(problem_.m(group_idx)), int(problem_.n(group_idx))},
802:         {int(problem_.ldc[group_idx])},
803:         gemm_workspace_.problem_count,
804:         0);
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 806-814
```cpp
806:       gemm_workspace_.reference_ptr_array_host[group_idx] = device_context.allocate_tensor(
807:         options,
808:         "Reference_" + group_str,
809:         operation_desc.gemm.D.element,
810:         operation_desc.gemm.D.layout,
811:         {int(problem_.m(group_idx)), int(problem_.n(group_idx))},
812:         {int(problem_.ldc[group_idx])},
813:         1,
814:         0);
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 816-822
```cpp
816:       if (is_block_scaled) {
817:         auto const block_scale_desc = operation_desc.block_scales.value();
818:         auto& block_scale_ws = gemm_workspace_.block_scales.value();
819:         int sfa_m = round_up(int(problem_.m(group_idx)), 128);
820:         int sfb_n = round_up(int(problem_.n(group_idx)), 128);
821:         int sfa_sfb_k =
822:           round_up(ceil_div(int(problem_.k(group_idx)), block_scale_desc.SFKVecSize), 4);
```
- **EN:** Declares or updates local/member state such as `block_scale_desc`, `block_scale_ws`, `sfa_m`, `sfb_n`.
- **CN:** 声明或更新局部/成员状态，例如 `block_scale_desc`, `block_scale_ws`, `sfa_m`, `sfb_n`。

### Lines 824-831
```cpp
824:         int sfd_m =
825:           block_scale_desc.SFD.layout == cutlass::library::LayoutTypeID::kRowMajor
826:             ? sfa_m
827:             : round_up(ceil_div(int(problem_.m(group_idx)), block_scale_desc.EpilogueSFVecSize), 4);
828:         int sfd_n =
829:           block_scale_desc.SFD.layout == cutlass::library::LayoutTypeID::kRowMajor
830:             ? round_up(ceil_div(int(problem_.n(group_idx)), block_scale_desc.EpilogueSFVecSize), 4)
831:             : sfb_n;
```
- **EN:** Implements `round_up` and coordinates helper calls such as `ceil_div`, `int`, `m`.
- **CN:** 实现 `round_up`，并协调调用 `ceil_div`, `int`, `m` 等辅助逻辑。

### Lines 833-843
```cpp
833:         block_scale_ws.SFA_ptr_array_host[group_idx] =
834:           device_context.allocate_and_initialize_tensor(
835:             options,
836:             "SFA",
837:             block_scale_desc.SFA.element,
838:             block_scale_desc.SFA.layout,
839:             {sfa_m, sfa_sfb_k},
840:             {sfa_sfb_k},
841:             gemm_workspace_.problem_count,
842:             seed_shift++,
843:             0);
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 845-855
```cpp
845:         block_scale_ws.SFB_ptr_array_host[group_idx] =
846:           device_context.allocate_and_initialize_tensor(
847:             options,
848:             "SFB",
849:             block_scale_desc.SFB.element,
850:             block_scale_desc.SFB.layout,
851:             {sfb_n, sfa_sfb_k},
852:             {sfa_sfb_k},
853:             gemm_workspace_.problem_count,
854:             seed_shift++,
855:             0);
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 857-865
```cpp
857:         block_scale_ws.SFD_ptr_array_host[group_idx] = device_context.allocate_tensor(
858:           options,
859:           "SFD",
860:           block_scale_desc.SFD.element,
861:           block_scale_desc.SFD.layout,
862:           {sfd_m, sfd_n},
863:           {sfd_n},
864:           gemm_workspace_.problem_count,
865:           0);
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 867-875
```cpp
867:         block_scale_ws.SFD_reference_ptr_array_host[group_idx] = device_context.allocate_tensor(
868:           options,
869:           "Reference_SFD",
870:           block_scale_desc.SFD.element,
871:           block_scale_desc.SFD.layout,
872:           {sfd_m, sfd_n},
873:           {sfd_n},
874:           gemm_workspace_.problem_count,
875:           0);
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 877-878
```cpp
877:         // ScaleFactor tensor results may have some holes and will not be touched by the kernel.
878:         // If we randomly fill the two tensors, these holes may encounter refcheck errors.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 879-889
```cpp
879:         if (block_scale_ws.SFD_ptr_array_host[group_idx]->type() != library::NumericTypeID::kVoid) {
880:           block_scale_ws.SFD_reference_ptr_array_host[group_idx]->fill_device(0);
881:           block_scale_ws.SFD_ptr_array_host[group_idx]->fill_device(0);
882:         }
883:       }
884:       else if (is_blockwise) {
885:         auto const block_scale_desc = operation_desc.block_scales.value();
886:         auto& block_scale_ws = gemm_workspace_.block_scales.value();
887:         int sfa_m     = ceil_div(int(problem_.m(group_idx)), block_scale_desc.SFMVecSize);
888:         int sfb_n     = ceil_div(int(problem_.n(group_idx)), block_scale_desc.SFNVecSize);
889:         int sfa_sfb_k = ceil_div(int(problem_.k(group_idx)), block_scale_desc.SFKVecSize);
```
- **EN:** Declares or updates local/member state such as `block_scale_desc`, `block_scale_ws`, `sfa_m`, `sfb_n`.
- **CN:** 声明或更新局部/成员状态，例如 `block_scale_desc`, `block_scale_ws`, `sfa_m`, `sfb_n`。

### Lines 891-901
```cpp
891:         block_scale_ws.SFA_ptr_array_host[group_idx] =
892:           device_context.allocate_and_initialize_tensor(
893:             options,
894:             "SFA_" + std::to_string(group_idx),
895:             block_scale_desc.SFA.element,
896:             block_scale_desc.SFA.layout,
897:             {sfa_m, sfa_sfb_k},
898:             {sfa_m},
899:             gemm_workspace_.problem_count,
900:             seed_shift++,
901:             0);
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 903-915
```cpp
903:         block_scale_ws.SFB_ptr_array_host[group_idx] =
904:           device_context.allocate_and_initialize_tensor(
905:             options,
906:             "SFB_" + std::to_string(group_idx),
907:             block_scale_desc.SFB.element,
908:             block_scale_desc.SFB.layout,
909:             {sfa_sfb_k, sfb_n},
910:             {sfb_n},
911:             gemm_workspace_.problem_count,
912:             seed_shift++,
913:             0);
914:       }
915:     }
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 917-917
```cpp
917:     // takes the allocated tensors and initializes an array of pointers per problem in the workspace
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 918-923
```cpp
918:     auto create_dev_ptr_array_all_workspace = [&](
919:                                                 std::vector<DeviceAllocation*>& dev_ptr_arrays,
920:                                                 std::vector<DeviceAllocation*> const& input,
921:                                                 std::string const& id) {
922:       auto num_workspaces = gemm_workspace_.problem_count;
923:       dev_ptr_arrays.resize(num_workspaces);
```
- **EN:** Declares or updates local/member state such as `create_dev_ptr_array_all_workspace`, `num_workspaces`, `problem_count`.
- **CN:** 声明或更新局部/成员状态，例如 `create_dev_ptr_array_all_workspace`, `num_workspaces`, `problem_count`。

### Lines 924-924
```cpp
924:       // note "problem_count" here refers to input/output count for L2 cycling
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 925-938
```cpp
925:       for (int i = 0; i < gemm_workspace_.problem_count; i++) {
926:         std::string name = id + "_ptr_array_workspace" + std::to_string(i);
927:         dev_ptr_arrays[i] =
928:           device_context.allocate_block(options, name, library::NumericTypeID::kU64, num_groups, 0);
929:         std::vector<void*> group_ptrs(num_groups);
930:         for (size_t group_idx = 0; group_idx < num_groups; group_idx++) {
931:           group_ptrs[group_idx] = input[group_idx]->batch_data(i);
932:         }
933:         dev_ptr_arrays[i]->copy_from_host(group_ptrs.data());
934:       }
935:     };
936:     create_dev_ptr_array_all_workspace(
937:       gemm_workspace_.A_ptr_array_device,
938:       gemm_workspace_.A_ptr_array_host,
```
- **EN:** Declares or updates local/member state such as `i`, `problem_count`, `name`, `group_idx`.
- **CN:** 声明或更新局部/成员状态，例如 `i`, `problem_count`, `name`, `group_idx`。

### Lines 939-951
```cpp
939:       "A");
940:     create_dev_ptr_array_all_workspace(
941:       gemm_workspace_.B_ptr_array_device,
942:       gemm_workspace_.B_ptr_array_host,
943:       "B");
944:     create_dev_ptr_array_all_workspace(
945:       gemm_workspace_.C_ptr_array_device,
946:       gemm_workspace_.C_ptr_array_host,
947:       "C");
948:     create_dev_ptr_array_all_workspace(
949:       gemm_workspace_.D_ptr_array_device,
950:       gemm_workspace_.D_ptr_array_host,
951:       "D");
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 953-966
```cpp
953:     if (is_block_scaled) {
954:       auto& block_scale_ws = gemm_workspace_.block_scales.value();
955:       create_dev_ptr_array_all_workspace(
956:         block_scale_ws.SFA_ptr_array_device,
957:         block_scale_ws.SFA_ptr_array_host,
958:         "SFA");
959:       create_dev_ptr_array_all_workspace(
960:         block_scale_ws.SFB_ptr_array_device,
961:         block_scale_ws.SFB_ptr_array_host,
962:         "SFB");
963:       create_dev_ptr_array_all_workspace(
964:         block_scale_ws.SFD_ptr_array_device,
965:         block_scale_ws.SFD_ptr_array_host,
966:         "SFD");
```
- **EN:** Declares or updates local/member state such as `block_scale_ws`.
- **CN:** 声明或更新局部/成员状态，例如 `block_scale_ws`。

### Lines 968-981
```cpp
968:       block_scale_ws.norm_constant = device_context.allocate_and_initialize_tensor(
969:         options,
970:         "norm_constant",
971:         operation_desc.gemm.element_epilogue,
972:         operation_desc.gemm.A.layout, // copied, but should this be D layout?
973:         {1, 1},
974:         {1},
975:         1,
976:         seed_shift++,
977:         0 // device_index
978:       );
979:     }
980:     else if (is_blockwise) {
981:       auto& block_scale_ws = gemm_workspace_.block_scales.value();
```
- **EN:** Declares or updates local/member state such as `norm_constant`, `block_scale_ws`.
- **CN:** 声明或更新局部/成员状态，例如 `norm_constant`, `block_scale_ws`。

### Lines 982-990
```cpp
982:       create_dev_ptr_array_all_workspace(
983:         block_scale_ws.SFA_ptr_array_device,
984:         block_scale_ws.SFA_ptr_array_host,
985:         "SFA");
986:       create_dev_ptr_array_all_workspace(
987:         block_scale_ws.SFB_ptr_array_device,
988:         block_scale_ws.SFB_ptr_array_host,
989:         "SFB");
990:     }
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 992-993
```cpp
992:     init_arguments(options);
993:   }
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 995-997
```cpp
995:   //
996:   // Initialize the CUTLASS operation
997:   //
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 998-1003
```cpp
998:   Status status = Status::kSuccess;
999:   if (options.profiling.provider_enabled(library::Provider::kCUTLASS)) {
1000:     if (options.execution_mode != ExecutionMode::kDryRun) {
1001:       uint64_t workspace_size =
1002:         underlying_operation->get_host_workspace_size(&gemm_workspace_.configuration);
1003:       gemm_workspace_.host_workspace.resize(workspace_size, 0);
```
- **EN:** Declares or updates local/member state such as `status`, `kSuccess`, `workspace_size`.
- **CN:** 声明或更新局部/成员状态，例如 `status`, `kSuccess`, `workspace_size`。

### Lines 1005-1008
```cpp
1005:       workspace_size = underlying_operation->get_device_workspace_size(
1006:         &gemm_workspace_.configuration,
1007:         &gemm_workspace_.arguments);
1008:       gemm_workspace_.device_workspace.reset(library::NumericTypeID::kU8, workspace_size);
```
- **EN:** Computes a thread-local view or helper object from previously constructed copy metadata.
- **CN:** 基于先前构造的拷贝元数据，计算线程局部视图或辅助对象。

### Lines 1010-1016
```cpp
1010:       status = underlying_operation->initialize(
1011:         &gemm_workspace_.configuration,
1012:         gemm_workspace_.host_workspace.data(),
1013:         gemm_workspace_.device_workspace.data());
1014:       if (status != Status::kSuccess) {
1015:         return status;
1016:       }
```
- **EN:** Initializes or registers grouped GEMM components for later lookup or execution.
- **CN:** 初始化或注册分组 GEMM组件，以便后续查找或执行。

### Lines 1018-1024
```cpp
1018:       status = underlying_operation->can_implement(
1019:         &gemm_workspace_.configuration,
1020:         &gemm_workspace_.arguments);
1021:       if (status != Status::kSuccess) {
1022:         return status;
1023:       }
1024:     }
```
- **EN:** Implements `can_implement` for this file's main component.
- **CN:** 为该文件的核心组件实现 `can_implement`。

### Lines 1026-1028
```cpp
1026:     //
1027:     // If CUTLASS is enabled, generate a result for it
1028:     //
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1029-1032
```cpp
1029:     results_.push_back(model_result_);
1030:     results_.back().provider = library::Provider::kCUTLASS;
1031:     results_.back().op_kind = library::OperationKind::kGroupedGemm;
1032:     results_.back().disposition = Disposition::kNotRun;
```
- **EN:** Implements `push_back` and coordinates helper calls such as `back`.
- **CN:** 实现 `push_back`，并协调调用 `back` 等辅助逻辑。

### Lines 1034-1039
```cpp
1034:     for (auto provider : verification_providers_) {
1035:       results_.back().verification_map[provider] = Disposition::kNotRun;
1036:     }
1037:   }
1038:   return status;
1039: }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 1041-1041
```cpp
1041: /////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1043-1043
```cpp
1043: /// Verifies CUTLASS against references
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1044-1050
```cpp
1044: bool GroupedGemmOperationProfiler::verify_cutlass(
1045:   Options const& options,
1046:   PerformanceReport& report,
1047:   DeviceContext& device_context,
1048:   library::Operation const* operation,
1049:   ProblemSpace const& problem_space,
1050:   ProblemSpace::Problem const& problem) {
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 1052-1054
```cpp
1052:   if (!options.profiling.provider_enabled(library::Provider::kCUTLASS)) {
1053:     return true;
1054:   }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 1056-1058
```cpp
1056:   if (options.execution_mode == ExecutionMode::kDryRun) {
1057:     return true;
1058:   }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 1060-1060
```cpp
1060:   init_arguments(options);
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 1062-1066
```cpp
1062:   library::Operation const* underlying_operation = operation;
1063:   results_.back().status = underlying_operation->initialize_with_arguments(&gemm_workspace_.arguments);
1064:   if (results_.back().status != Status::kSuccess) {
1065:     return false;
1066:   }
```
- **EN:** Implements `back` and coordinates helper calls such as `initialize_with_arguments`.
- **CN:** 实现 `back`，并协调调用 `initialize_with_arguments` 等辅助逻辑。

### Lines 1068-1071
```cpp
1068:   results_.back().status = underlying_operation->run(
1069:     &gemm_workspace_.arguments,
1070:     gemm_workspace_.host_workspace.data(),
1071:     gemm_workspace_.device_workspace.data());
```
- **EN:** Implements `back` and coordinates helper calls such as `run`, `data`.
- **CN:** 实现 `back`，并协调调用 `run`, `data` 等辅助逻辑。

### Lines 1073-1076
```cpp
1073:   if (results_.back().status != Status::kSuccess) {
1074:     results_.back().disposition = Disposition::kFailed;
1075:     return false;
1076:   }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 1078-1082
```cpp
1078:   cudaError_t result = cudaDeviceSynchronize();
1079:   if (result != cudaSuccess) {
1080:     results_.back().disposition = Disposition::kFailed;
1081:     return false;
1082:   }
```
- **EN:** Implements `cudaDeviceSynchronize` and coordinates helper calls such as `back`.
- **CN:** 实现 `cudaDeviceSynchronize`，并协调调用 `back` 等辅助逻辑。

### Lines 1084-1084
```cpp
1084:   // CUTLASS op ran the but not yet verified against any verification provider
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1085-1085
```cpp
1085:   results_.back().disposition = Disposition::kNotVerified;
```
- **EN:** Declares or updates local/member state such as `disposition`, `kNotVerified`.
- **CN:** 声明或更新局部/成员状态，例如 `disposition`, `kNotVerified`。

### Lines 1087-1089
```cpp
1087:   //
1088:   // Run verification providers
1089:   //
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1091-1091
```cpp
1091:   if (options.verification.enabled) {
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 1093-1093
```cpp
1093: #if CUTLASS_ENABLE_CUBLAS
```
- **EN:** Conditional-compilation or macro block keyed on `CUTLASS_ENABLE_CUBLAS`.
- **CN:** 以 `CUTLASS_ENABLE_CUBLAS` 为条件的条件编译或宏定义代码块。

### Lines 1094-1094
```cpp
1094:     if (options.verification.provider_enabled(library::Provider::kCUBLAS)) {
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 1095-1095
```cpp
1095:       // set verification map for cublas to not supported
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1096-1097
```cpp
1096:       results_.back().verification_map[library::Provider::kCUBLAS] = Disposition::kNotSupported;
1097:     }
```
- **EN:** Declares or updates local/member state such as `kNotSupported`.
- **CN:** 声明或更新局部/成员状态，例如 `kNotSupported`。

### Lines 1098-1098
```cpp
1098: #endif // #if CUTLASS_ENABLE_CUBLAS
```
- **EN:** Conditional-compilation or macro block keyed on `CUTLASS_ENABLE_CUBLAS`.
- **CN:** 以 `CUTLASS_ENABLE_CUBLAS` 为条件的条件编译或宏定义代码块。

### Lines 1100-1101
```cpp
1100:     auto const& desc =
1101:       static_cast<library::GroupedGemmDescription const&>(operation->description());
```
- **EN:** Implements `description` for this file's main component.
- **CN:** 为该文件的核心组件实现 `description`。

### Lines 1103-1104
```cpp
1103:     cutlass::library::RuntimeDatatype runtime_datatype_a = gemm_workspace_.arguments.runtime_input_datatype_a;
1104:     cutlass::library::RuntimeDatatype runtime_datatype_b = gemm_workspace_.arguments.runtime_input_datatype_b;
```
- **EN:** Declares or updates local/member state such as `runtime_datatype_a`, `runtime_input_datatype_a`, `runtime_datatype_b`, `runtime_input_datatype_b`.
- **CN:** 声明或更新局部/成员状态，例如 `runtime_datatype_a`, `runtime_input_datatype_a`, `runtime_datatype_b`, `runtime_input_datatype_b`。

### Lines 1106-1107
```cpp
1106:     bool is_runtime_datatype_a = runtime_datatype_a != cutlass::library::RuntimeDatatype::kStatic;
1107:     bool is_runtime_datatype_b = runtime_datatype_b != cutlass::library::RuntimeDatatype::kStatic;
```
- **EN:** Declares or updates local/member state such as `is_runtime_datatype_a`, `kStatic`, `is_runtime_datatype_b`.
- **CN:** 声明或更新局部/成员状态，例如 `is_runtime_datatype_a`, `kStatic`, `is_runtime_datatype_b`。

### Lines 1109-1109
```cpp
1109:     assert(is_runtime_datatype_a == is_runtime_datatype_b && "runtime datatype should be both dynamic or static.");
```
- **EN:** Declares or updates local/member state such as `is_runtime_datatype_a`.
- **CN:** 声明或更新局部/成员状态，例如 `is_runtime_datatype_a`。

### Lines 1111-1112
```cpp
1111:     cutlass::library::NumericTypeID element_A = desc.gemm.A.element;
1112:     cutlass::library::NumericTypeID element_B = desc.gemm.B.element;
```
- **EN:** Declares or updates local/member state such as `element_A`, `element`, `element_B`.
- **CN:** 声明或更新局部/成员状态，例如 `element_A`, `element`, `element_B`。

### Lines 1114-1116
```cpp
1114:     if (is_runtime_datatype_a) {
1115:       element_A = cutlass::library::dynamic_datatype_to_id(runtime_datatype_a);
1116:     }
```
- **EN:** Declares or updates local/member state such as `element_A`.
- **CN:** 声明或更新局部/成员状态，例如 `element_A`。

### Lines 1118-1120
```cpp
1118:     if (is_runtime_datatype_b) {
1119:       element_B = cutlass::library::dynamic_datatype_to_id(runtime_datatype_b);
1120:     }
```
- **EN:** Declares or updates local/member state such as `element_B`.
- **CN:** 声明或更新局部/成员状态，例如 `element_B`。

### Lines 1122-1130
```cpp
1122:     bool verification_status = verify_with_reference_(
1123:       options,
1124:       report,
1125:       device_context,
1126:       operation,
1127:       problem_space,
1128:       problem,
1129:       element_A,
1130:       element_B);
```
- **EN:** Declares or updates local/member state such as `verification_status`.
- **CN:** 声明或更新局部/成员状态，例如 `verification_status`。

### Lines 1132-1133
```cpp
1132:     // Update disposition to worst case verification outcome among all
1133:     // verification providers which are supported
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1134-1143
```cpp
1134:     bool is_any_verification_run_passed = false;
1135:     for (auto& m : results_.back().verification_map) {
1136:       if (m.second == Disposition::kFailed || m.second == Disposition::kIncorrect) {
1137:         results_.back().disposition = m.second;
1138:         return true;
1139:       }
1140:       if (!is_any_verification_run_passed && m.second == Disposition::kPassed) {
1141:         is_any_verification_run_passed = true;
1142:       }
1143:     }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 1145-1148
```cpp
1145:     if (is_any_verification_run_passed) {
1146:       results_.back().disposition = Disposition::kPassed;
1147:     }
1148:   }
```
- **EN:** Declares or updates local/member state such as `disposition`, `kPassed`.
- **CN:** 声明或更新局部/成员状态，例如 `disposition`, `kPassed`。

### Lines 1150-1150
```cpp
1150:   // if verification.required is set, then return success iff at least one ref-check was run
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1151-1156
```cpp
1151:   if (options.verification.required) {
1152:     bool did_any_verification_run = false;
1153:     for (auto provider : options.verification.providers) {
1154:       did_any_verification_run |=
1155:         (Disposition::kNotRun != results_.back().verification_map[provider]);
1156:     }
```
- **EN:** Declares or updates local/member state such as `did_any_verification_run`, `false`.
- **CN:** 声明或更新局部/成员状态，例如 `did_any_verification_run`, `false`。

### Lines 1158-1162
```cpp
1158:     if (not did_any_verification_run) {
1159:       results_.back().status = Status::kErrorNotSupported;
1160:       return false;
1161:     }
1162:   }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 1164-1164
```cpp
1164:   // Return true means continue profiling
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1165-1166
```cpp
1165:   return true;
1166: }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 1168-1168
```cpp
1168: /// Verifies CUTLASS against host and device references
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1169-1179
```cpp
1169: bool GroupedGemmOperationProfiler::verify_with_reference_(
1170:   Options const& options,
1171:   PerformanceReport& report,
1172:   DeviceContext& device_context,
1173:   library::Operation const* operation,
1174:   ProblemSpace const& problem_space,
1175:   ProblemSpace::Problem const& problem,
1176:   cutlass::library::NumericTypeID element_A,
1177:   cutlass::library::NumericTypeID element_B) {
1178:   library::GroupedGemmDescription const& desc =
1179:     static_cast<library::GroupedGemmDescription const&>(operation->description());
```
- **EN:** Declares or updates local/member state such as `desc`.
- **CN:** 声明或更新局部/成员状态，例如 `desc`。

### Lines 1181-1181
```cpp
1181:   for (auto provider : options.verification.providers) {
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 1183-1183
```cpp
1183:     // Skip providers that are not enabled
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1184-1186
```cpp
1184:     if (!options.verification.provider_enabled(provider)) {
1185:       continue;
1186:     }
```
- **EN:** Declares or updates local/member state such as `continue`.
- **CN:** 声明或更新局部/成员状态，例如 `continue`。

### Lines 1188-1188
```cpp
1188:     // we only have a block scaled reference kernel implemented on the host
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1189-1191
```cpp
1189:     if ((is_block_scaled || is_blockwise) && provider != library::Provider::kReferenceHost) {
1190:       continue;
1191:     }
```
- **EN:** Declares or updates local/member state such as `continue`.
- **CN:** 声明或更新局部/成员状态，例如 `continue`。

### Lines 1193-1194
```cpp
1193:     auto status = Status::kSuccess;
1194:     auto disposition = Disposition::kFailed;
```
- **EN:** Declares or updates local/member state such as `status`, `kSuccess`, `disposition`, `kFailed`.
- **CN:** 声明或更新局部/成员状态，例如 `status`, `kSuccess`, `disposition`, `kFailed`。

### Lines 1195-1196
```cpp
1195:     // we don't have grouped GEMM reference kernels so we loop over the groups and perform
1196:     // a regular GEMM for each group
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1197-1202
```cpp
1197:     for (size_t group_idx = 0, num_groups = problem_.problem_sizes.size(); group_idx < num_groups;
1198:          group_idx++) {
1199:       void* ptr_A = gemm_workspace_.A_ptr_array_host[group_idx]->data();
1200:       void* ptr_B = gemm_workspace_.B_ptr_array_host[group_idx]->data();
1201:       void* ptr_C = gemm_workspace_.C_ptr_array_host[group_idx]->data();
1202:       void* ptr_D = gemm_workspace_.reference_ptr_array_host[group_idx]->data();
```
- **EN:** Declares or updates local/member state such as `group_idx`, `num_groups`, `ptr_A`, `ptr_B`.
- **CN:** 声明或更新局部/成员状态，例如 `group_idx`, `num_groups`, `ptr_A`, `ptr_B`。

### Lines 1204-1205
```cpp
1204:       // To support the host-side reference, conditionally allocate and
1205:       // copy tensors to host memory.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1206-1214
```cpp
1206:       std::vector<uint8_t> host_data_A;
1207:       std::vector<uint8_t> host_data_B;
1208:       std::vector<uint8_t> host_data_C;
1209:       std::vector<uint8_t> host_data_D;
1210:       std::vector<uint8_t> host_data_SFA;
1211:       std::vector<uint8_t> host_data_SFB;
1212:       std::vector<uint8_t> host_data_SFC;
1213:       std::vector<uint8_t> host_data_SFD;
1214:       std::vector<uint8_t> host_data_norm_constant;
```
- **EN:** Declares or updates local/member state such as `host_data_A`, `host_data_B`, `host_data_C`, `host_data_D`.
- **CN:** 声明或更新局部/成员状态，例如 `host_data_A`, `host_data_B`, `host_data_C`, `host_data_D`。

### Lines 1216-1219
```cpp
1216:       void* ptr_SFA{nullptr};
1217:       void* ptr_SFB{nullptr};
1218:       void* ptr_SFD{nullptr};
1219:       void* ptr_norm_constant{nullptr};
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 1221-1225
```cpp
1221:       if (provider == library::Provider::kReferenceHost) {
1222:         host_data_A.resize(gemm_workspace_.A_ptr_array_host[group_idx]->bytes());
1223:         ptr_A = host_data_A.data();
1224:         gemm_workspace_.A_ptr_array_host[group_idx]->copy_to_host(
1225:           ptr_A); // this is copying all the data for L2 busting as well
```
- **EN:** Declares or updates local/member state such as `provider`, `ptr_A`.
- **CN:** 声明或更新局部/成员状态，例如 `provider`, `ptr_A`。

### Lines 1227-1229
```cpp
1227:         host_data_B.resize(gemm_workspace_.B_ptr_array_host[group_idx]->bytes());
1228:         ptr_B = host_data_B.data();
1229:         gemm_workspace_.B_ptr_array_host[group_idx]->copy_to_host(ptr_B);
```
- **EN:** Implements `resize` and coordinates helper calls such as `bytes`, `data`, `copy_to_host`.
- **CN:** 实现 `resize`，并协调调用 `bytes`, `data`, `copy_to_host` 等辅助逻辑。

### Lines 1231-1233
```cpp
1231:         host_data_C.resize(gemm_workspace_.C_ptr_array_host[group_idx]->bytes());
1232:         ptr_C = host_data_C.data();
1233:         gemm_workspace_.C_ptr_array_host[group_idx]->copy_to_host(ptr_C);
```
- **EN:** Implements `resize` and coordinates helper calls such as `bytes`, `data`, `copy_to_host`.
- **CN:** 实现 `resize`，并协调调用 `bytes`, `data`, `copy_to_host` 等辅助逻辑。

### Lines 1235-1236
```cpp
1235:         host_data_D.resize(gemm_workspace_.reference_ptr_array_host[group_idx]->bytes());
1236:         ptr_D = host_data_D.data();
```
- **EN:** Implements `resize` and coordinates helper calls such as `bytes`, `data`.
- **CN:** 实现 `resize`，并协调调用 `bytes`, `data` 等辅助逻辑。

### Lines 1238-1239
```cpp
1238:         if (is_block_scaled) {
1239:           auto const& ws = gemm_workspace_.block_scales.value();
```
- **EN:** Declares or updates local/member state such as `ws`.
- **CN:** 声明或更新局部/成员状态，例如 `ws`。

### Lines 1241-1246
```cpp
1241:           host_data_SFA.resize(ws.SFA_ptr_array_host[group_idx]->bytes());
1242:           ptr_SFA = host_data_SFA.data();
1243:           ws.SFA_ptr_array_host[group_idx]->copy_to_host(ptr_SFA);
1244:           host_data_SFB.resize(ws.SFB_ptr_array_host[group_idx]->bytes());
1245:           ptr_SFB = host_data_SFB.data();
1246:           ws.SFB_ptr_array_host[group_idx]->copy_to_host(ptr_SFB);
```
- **EN:** Implements `resize` and coordinates helper calls such as `bytes`, `data`, `copy_to_host`.
- **CN:** 实现 `resize`，并协调调用 `bytes`, `data`, `copy_to_host` 等辅助逻辑。

### Lines 1248-1249
```cpp
1248:           host_data_SFD.resize(ws.SFD_reference_ptr_array_host[group_idx]->bytes());
1249:           ptr_SFD = host_data_SFD.data();
```
- **EN:** Implements `resize` and coordinates helper calls such as `bytes`, `data`.
- **CN:** 实现 `resize`，并协调调用 `bytes`, `data` 等辅助逻辑。

### Lines 1251-1256
```cpp
1251:           host_data_norm_constant.resize(ws.norm_constant->bytes());
1252:           ptr_norm_constant = host_data_norm_constant.data();
1253:           ws.norm_constant->copy_to_host(ptr_norm_constant);
1254:         }
1255:         else if (is_blockwise) {
1256:           auto const& ws = gemm_workspace_.block_scales.value();
```
- **EN:** Implements `resize` and coordinates helper calls such as `bytes`, `data`, `copy_to_host`.
- **CN:** 实现 `resize`，并协调调用 `bytes`, `data`, `copy_to_host` 等辅助逻辑。

### Lines 1258-1265
```cpp
1258:           host_data_SFA.resize(ws.SFA_ptr_array_host[group_idx]->bytes());
1259:           ptr_SFA = host_data_SFA.data();
1260:           ws.SFA_ptr_array_host[group_idx]->copy_to_host(ptr_SFA);
1261:           host_data_SFB.resize(ws.SFB_ptr_array_host[group_idx]->bytes());
1262:           ptr_SFB = host_data_SFB.data();
1263:           ws.SFB_ptr_array_host[group_idx]->copy_to_host(ptr_SFB);
1264:         }
1265:       }
```
- **EN:** Implements `resize` and coordinates helper calls such as `bytes`, `data`, `copy_to_host`.
- **CN:** 实现 `resize`，并协调调用 `bytes`, `data`, `copy_to_host` 等辅助逻辑。

### Lines 1267-1268
```cpp
1267:       const auto &desc = static_cast<library::GroupedGemmDescription const &>(operation->description());
1268:       const auto& gemm_desc = desc.gemm;
```
- **EN:** Implements `description` for this file's main component.
- **CN:** 为该文件的核心组件实现 `description`。

### Lines 1270-1272
```cpp
1270:       if (!is_block_scaled and !is_blockwise) {
1271:         library::Handle handle;
1272:         handle.set_provider(provider);
```
- **EN:** Declares or updates local/member state such as `handle`.
- **CN:** 声明或更新局部/成员状态，例如 `handle`。

### Lines 1274-1287
```cpp
1274:         status = handle.gemm_universal(
1275:           library::GemmUniversalMode::kGemm,
1276:           problem_.m(group_idx),
1277:           problem_.n(group_idx),
1278:           problem_.k(group_idx),
1279:           problem_.cluster_m,
1280:           problem_.cluster_n,
1281:           problem_.cluster_k,
1282:           problem_.cluster_m_fallback,
1283:           problem_.cluster_n_fallback,
1284:           problem_.cluster_k_fallback,
1285:           desc.gemm.tile_description.math_instruction.element_accumulator,
1286:           desc.gemm.element_epilogue,
1287:           problem_.alpha.data(),
```
- **EN:** Declares or updates local/member state such as `status`.
- **CN:** 声明或更新局部/成员状态，例如 `status`。

### Lines 1288-1301
```cpp
1288:           element_A,
1289:           desc.gemm.A.layout,
1290:           desc.gemm.transform_A,
1291:           ptr_A,
1292:           int(problem_.lda[group_idx]),
1293:           element_B,
1294:           desc.gemm.B.layout,
1295:           desc.gemm.transform_B,
1296:           ptr_B,
1297:           int(problem_.ldb[group_idx]),
1298:           problem_.beta.data(),
1299:           desc.gemm.C.element,
1300:           desc.gemm.C.layout,
1301:           ptr_C,
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 1302-1312
```cpp
1302:           int(problem_.ldc[group_idx]),
1303:           desc.gemm.D.element,
1304:           desc.gemm.D.layout,
1305:           ptr_D,
1306:           int(problem_.ldc[group_idx]),
1307:           1,
1308:           gemm_workspace_.A_ptr_array_host[group_idx]->batch_stride(),
1309:           gemm_workspace_.B_ptr_array_host[group_idx]->batch_stride(),
1310:           gemm_workspace_.C_ptr_array_host[group_idx]->batch_stride(),
1311:           gemm_workspace_.reference_ptr_array_host[group_idx]->batch_stride());
1312:       }
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 1313-1315
```cpp
1313:       else if (is_block_scaled) {
1314:         auto const& block_scale_desc = desc.block_scales.value();
1315:         auto& block_scale_ws = gemm_workspace_.block_scales.value();
```
- **EN:** Declares or updates local/member state such as `block_scale_desc`, `block_scale_ws`.
- **CN:** 声明或更新局部/成员状态，例如 `block_scale_desc`, `block_scale_ws`。

### Lines 1317-1330
```cpp
1317:         library::BlockScaledGemmFunctionalKey blockScaledGemm_key(
1318:           library::Provider::kReferenceHost,
1319:           library::GemmKind::kUniversal,
1320:           library::OperationKind::kBlockScaledGemm,
1321:           gemm_desc.tile_description.math_instruction.element_accumulator,
1322:           gemm_desc.element_epilogue,
1323:           element_A,
1324:           gemm_desc.A.layout,
1325:           block_scale_desc.SFA.element,
1326:           element_B,
1327:           gemm_desc.B.layout,
1328:           block_scale_desc.SFB.element,
1329:           gemm_desc.C.element,
1330:           gemm_desc.C.layout,
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 1331-1336
```cpp
1331:           gemm_desc.D.element,
1332:           gemm_desc.D.layout,
1333:           block_scale_desc.SFD.element,
1334:           block_scale_desc.SFD.layout,
1335:           block_scale_desc.SFKVecSize,
1336:           block_scale_desc.EpilogueSFVecSize);
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 1338-1346
```cpp
1338:         auto operators_it =
1339:           library::Singleton::get().operation_table.block_scaled_gemm_operations.find(
1340:             blockScaledGemm_key);
1341:         if (
1342:           operators_it ==
1343:           library::Singleton::get().operation_table.block_scaled_gemm_operations.end()) {
1344:           disposition = Disposition::kNotSupported;
1345:           break;
1346:         }
```
- **EN:** Implements `get` and coordinates helper calls such as `find`, `end`.
- **CN:** 实现 `get`，并协调调用 `find`, `end` 等辅助逻辑。

### Lines 1348-1351
```cpp
1348:         if (operators_it->second.empty()) {
1349:           disposition = Disposition::kNotSupported;
1350:           break;
1351:         }
```
- **EN:** Declares or updates local/member state such as `disposition`, `kNotSupported`, `break`.
- **CN:** 声明或更新局部/成员状态，例如 `disposition`, `kNotSupported`, `break`。

### Lines 1353-1357
```cpp
1353:         auto cc_it = operators_it->second.begin();
1354:         if (cc_it == operators_it->second.end()) {
1355:           disposition = Disposition::kNotSupported;
1356:           break;
1357:         }
```
- **EN:** Provides a small helper for managing container lifetime or iteration.
- **CN:** 提供用于管理容器生命周期或迭代的小型辅助函数。

### Lines 1359-1359
```cpp
1359:         // host reference has only one instances in BlockScaledOperationVectorMap
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1360-1373
```cpp
1360:         library::Operation const* reference_op = cc_it->second[0];
1361:         library::BlockScaledGemmArguments arguments{
1362:           {int(problem_.m(group_idx)), int(problem_.n(group_idx)), int(problem_.k(group_idx))},
1363:           {int(problem_.cluster_m), int(problem_.cluster_n), int(problem_.cluster_k)},
1364:           {int(problem_.cluster_m_fallback), int(problem_.cluster_n_fallback), int(problem_.cluster_k_fallback)},
1365:           1, // batch count
1366:           ptr_A,
1367:           ptr_B,
1368:           ptr_SFA,
1369:           ptr_SFB,
1370:           ptr_C,
1371:           ptr_D,
1372:           ptr_SFD,
1373:           problem_.alpha.data(),
```
- **EN:** Declares or updates local/member state such as `reference_op`.
- **CN:** 声明或更新局部/成员状态，例如 `reference_op`。

### Lines 1374-1384
```cpp
1374:           problem_.beta.data(),
1375:           library::ScalarPointerMode::kHost,
1376:           problem_.lda[group_idx],
1377:           problem_.ldb[group_idx],
1378:           problem_.ldc[group_idx],
1379:           problem_.ldc[group_idx],
1380:           gemm_workspace_.A_ptr_array_host[group_idx]->batch_stride(),
1381:           gemm_workspace_.B_ptr_array_host[group_idx]->batch_stride(),
1382:           gemm_workspace_.C_ptr_array_host[group_idx]->batch_stride(),
1383:           gemm_workspace_.reference_ptr_array_host[group_idx]->batch_stride(),
1384:           ptr_norm_constant};
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 1386-1399
```cpp
1386:         library::GemmUniversalConfiguration configuration{
1387:           library::GemmUniversalMode::kGemm,
1388:           problem_.problem_sizes[group_idx],
1389:           {problem_.cluster_m, problem_.cluster_n, problem_.cluster_k},
1390:           {problem_.cluster_m_fallback, problem_.cluster_n_fallback, problem_.cluster_k_fallback},
1391:           1,
1392:           problem_.lda[group_idx],
1393:           problem_.ldb[group_idx],
1394:           problem_.ldc[group_idx],
1395:           problem_.ldc[group_idx],
1396:           1,
1397:         };
1398:         uint64_t host_workspace_size_needed = reference_op->get_host_workspace_size(&gemm_workspace_.configuration);
1399:         std::vector<char> host_workspace(host_workspace_size_needed);
```
- **EN:** Declares or updates local/member state such as `host_workspace_size_needed`.
- **CN:** 声明或更新局部/成员状态，例如 `host_workspace_size_needed`。

### Lines 1400-1403
```cpp
1400:         status = reference_op->initialize(&configuration, host_workspace.data());
1401:         if (status != Status::kSuccess) {
1402:           break;
1403:         }
```
- **EN:** Initializes or registers grouped GEMM components for later lookup or execution.
- **CN:** 初始化或注册分组 GEMM组件，以便后续查找或执行。

### Lines 1405-1405
```cpp
1405:         status = reference_op->run(&arguments, host_workspace.data());
```
- **EN:** Implements `run` and coordinates helper calls such as `data`.
- **CN:** 实现 `run`，并协调调用 `data` 等辅助逻辑。

### Lines 1407-1409
```cpp
1407:         block_scale_ws.SFD_reference_ptr_array_host[group_idx]->copy_from_host(ptr_SFD);
1408:       }
1409:       else {
```
- **EN:** Implements `copy_from_host` for this file's main component.
- **CN:** 为该文件的核心组件实现 `copy_from_host`。

### Lines 1410-1410
```cpp
1410:         // Blockwise
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1411-1412
```cpp
1411:         auto const& block_scale_desc = desc.block_scales.value();
1412:         auto& block_scale_ws = gemm_workspace_.block_scales.value();
```
- **EN:** Implements `value` for this file's main component.
- **CN:** 为该文件的核心组件实现 `value`。

### Lines 1414-1427
```cpp
1414:         library::BlockwiseGemmFunctionalKey blockwiseGemm_key(
1415:           library::Provider::kReferenceHost,
1416:           library::GemmKind::kUniversal,
1417:           library::OperationKind::kBlockwiseGemm,
1418:           gemm_desc.tile_description.math_instruction.element_accumulator,
1419:           gemm_desc.element_epilogue,
1420:           element_A,
1421:           gemm_desc.A.layout,
1422:           block_scale_desc.SFA.element,
1423:           element_B,
1424:           gemm_desc.B.layout,
1425:           block_scale_desc.SFB.element,
1426:           gemm_desc.C.element,
1427:           gemm_desc.C.layout,
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 1428-1433
```cpp
1428:           gemm_desc.D.element,
1429:           gemm_desc.D.layout,
1430:           block_scale_desc.SFMVecSize,
1431:           block_scale_desc.SFNVecSize,
1432:           block_scale_desc.SFKVecSize
1433:         );
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 1435-1441
```cpp
1435:         auto operators_it = library::Singleton::get().operation_table.blockwise_gemm_operations.find(blockwiseGemm_key);
1436:         if (
1437:           operators_it ==
1438:           library::Singleton::get().operation_table.blockwise_gemm_operations.end()) {
1439:           disposition = Disposition::kNotSupported;
1440:           break;
1441:         }
```
- **EN:** Implements `get` and coordinates helper calls such as `find`, `end`.
- **CN:** 实现 `get`，并协调调用 `find`, `end` 等辅助逻辑。

### Lines 1443-1446
```cpp
1443:         if (operators_it->second.empty()) {
1444:           disposition = Disposition::kNotSupported;
1445:           break;
1446:         }
```
- **EN:** Declares or updates local/member state such as `disposition`, `kNotSupported`, `break`.
- **CN:** 声明或更新局部/成员状态，例如 `disposition`, `kNotSupported`, `break`。

### Lines 1448-1452
```cpp
1448:         auto cc_it = operators_it->second.begin();
1449:         if (cc_it == operators_it->second.end()) {
1450:           disposition = Disposition::kNotSupported;
1451:           break;
1452:         }
```
- **EN:** Provides a small helper for managing container lifetime or iteration.
- **CN:** 提供用于管理容器生命周期或迭代的小型辅助函数。

### Lines 1454-1454
```cpp
1454:         // host reference has only one instances in BlockScaledOperationVectorMap
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1455-1455
```cpp
1455:         library::Operation const* reference_op = cc_it->second[0];
```
- **EN:** Declares or updates local/member state such as `reference_op`.
- **CN:** 声明或更新局部/成员状态，例如 `reference_op`。

### Lines 1457-1470
```cpp
1457:         library::BlockwiseGemmArguments arguments {
1458:           {int(problem_.m(group_idx)), int(problem_.n(group_idx)), int(problem_.k(group_idx))},
1459:           {int(problem_.cluster_m), int(problem_.cluster_n), int(problem_.cluster_k)},
1460:           {int(problem_.cluster_m_fallback), int(problem_.cluster_n_fallback), int(problem_.cluster_k_fallback)},
1461:           1, // batch_count
1462:           ptr_A,
1463:           ptr_B,
1464:           ptr_SFA,
1465:           ptr_SFB,
1466:           ptr_C,
1467:           ptr_D,
1468:           problem_.alpha.data(),
1469:           problem_.beta.data(),
1470:           library::ScalarPointerMode::kHost,
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 1471-1479
```cpp
1471:           problem_.lda[group_idx],
1472:           problem_.ldb[group_idx],
1473:           problem_.ldc[group_idx],
1474:           problem_.ldc[group_idx],
1475:           gemm_workspace_.A_ptr_array_host[group_idx]->batch_stride(),
1476:           gemm_workspace_.B_ptr_array_host[group_idx]->batch_stride(),
1477:           gemm_workspace_.C_ptr_array_host[group_idx]->batch_stride(),
1478:           gemm_workspace_.reference_ptr_array_host[group_idx]->batch_stride(),
1479:         };
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 1481-1494
```cpp
1481:         library::GemmUniversalConfiguration configuration{
1482:           library::GemmUniversalMode::kGemm,
1483:           problem_.problem_sizes[group_idx],
1484:           {problem_.cluster_m, problem_.cluster_n, problem_.cluster_k},
1485:           {problem_.cluster_m_fallback, problem_.cluster_n_fallback, problem_.cluster_k_fallback},
1486:           1,
1487:           problem_.lda[group_idx],
1488:           problem_.ldb[group_idx],
1489:           problem_.ldc[group_idx],
1490:           problem_.ldc[group_idx],
1491:           1,
1492:         };
1493:         uint64_t host_workspace_size_needed = reference_op->get_host_workspace_size(&gemm_workspace_.configuration);
1494:         std::vector<char> host_workspace(host_workspace_size_needed);
```
- **EN:** Declares or updates local/member state such as `host_workspace_size_needed`.
- **CN:** 声明或更新局部/成员状态，例如 `host_workspace_size_needed`。

### Lines 1495-1498
```cpp
1495:         status = reference_op->initialize(&configuration, host_workspace.data());
1496:         if (status != Status::kSuccess) {
1497:           break;
1498:         }
```
- **EN:** Initializes or registers grouped GEMM components for later lookup or execution.
- **CN:** 初始化或注册分组 GEMM组件，以便后续查找或执行。

### Lines 1500-1501
```cpp
1500:         status = reference_op->run(&arguments, host_workspace.data());
1501:       }
```
- **EN:** Implements `run` and coordinates helper calls such as `data`.
- **CN:** 实现 `run`，并协调调用 `data` 等辅助逻辑。

### Lines 1503-1505
```cpp
1503:       if (status != Status::kSuccess) {
1504:         break;
1505:       }
```
- **EN:** Declares or updates local/member state such as `break`.
- **CN:** 声明或更新局部/成员状态，例如 `break`。

### Lines 1507-1509
```cpp
1507:       if (provider == library::Provider::kReferenceHost) {
1508:         gemm_workspace_.reference_ptr_array_host[group_idx]->copy_from_host(ptr_D);
1509:       }
```
- **EN:** Declares or updates local/member state such as `provider`.
- **CN:** 声明或更新局部/成员状态，例如 `provider`。

### Lines 1511-1518
```cpp
1511:       disposition = compare_tensors(
1512:         options,
1513:         *gemm_workspace_.D_ptr_array_host[group_idx],
1514:         *gemm_workspace_.reference_ptr_array_host[group_idx],
1515:         gemm_workspace_.D_ptr_array_host[group_idx]->batch_stride());
1516:       if (disposition != Disposition::kPassed) {
1517:         break;
1518:       }
```
- **EN:** Declares or updates local/member state such as `disposition`, `break`.
- **CN:** 声明或更新局部/成员状态，例如 `disposition`, `break`。

### Lines 1520-1533
```cpp
1520:       if (is_block_scaled) {
1521:         auto& ws = gemm_workspace_.block_scales.value();
1522:         auto const& block_scale_desc = desc.block_scales.value();
1523:         if (block_scale_desc.SFD.element != library::NumericTypeID::kVoid) {
1524:           disposition = compare_tensors(
1525:             options,
1526:             *ws.SFD_ptr_array_host[group_idx],
1527:             *ws.SFD_reference_ptr_array_host[group_idx],
1528:             ws.SFD_ptr_array_host[group_idx]->batch_stride());
1529:           if (disposition != Disposition::kPassed) {
1530:             break;
1531:           }
1532:         }
1533:       }
```
- **EN:** Declares or updates local/member state such as `ws`, `block_scale_desc`, `disposition`, `break`.
- **CN:** 声明或更新局部/成员状态，例如 `ws`, `block_scale_desc`, `disposition`, `break`。

### Lines 1534-1538
```cpp
1534:     }
1535:     if (status != Status::kSuccess) {
1536:       results_.back().verification_map[provider] = Disposition::kNotVerified;
1537:       continue;
1538:     }
```
- **EN:** Declares or updates local/member state such as `kNotVerified`, `continue`.
- **CN:** 声明或更新局部/成员状态，例如 `kNotVerified`, `continue`。

### Lines 1539-1540
```cpp
1539:     results_.back().status = status;
1540:     results_.back().verification_map[provider] = disposition;
```
- **EN:** Declares or updates local/member state such as `status`, `disposition`.
- **CN:** 声明或更新局部/成员状态，例如 `status`, `disposition`。

### Lines 1542-1547
```cpp
1542:     if (
1543:       options.verification.save_workspace == SaveWorkspace::kIncorrect &&
1544:       results_.back().verification_map[provider] == Disposition::kIncorrect) {
1545:       save_workspace(device_context, options, desc, library::Provider::kCUTLASS, provider);
1546:     }
1547:   }
```
- **EN:** Declares or updates local/member state such as `save_workspace`.
- **CN:** 声明或更新局部/成员状态，例如 `save_workspace`。

### Lines 1549-1550
```cpp
1549:   return true; // continue profiling
1550: }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 1552-1552
```cpp
1552: /////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1554-1554
```cpp
1554: /// Measures performance results
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1555-1561
```cpp
1555: bool GroupedGemmOperationProfiler::profile(
1556:   Options const& options,
1557:   PerformanceReport& report,
1558:   DeviceContext& device_context,
1559:   library::Operation const* operation,
1560:   ProblemSpace const& problem_space,
1561:   ProblemSpace::Problem const& problem) {
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 1563-1576
```cpp
1563:   if (options.profiling.provider_enabled(library::Provider::kCUTLASS)) {
1564:     if (options.profiling.enable_kernel_performance_search) {
1565:       std::cerr << "Exhaustive performance search is not available for Grouped GEMMs. " 
1566:                 << "Please use --enable-best-kernel-for-fixed-shape to profile a specific problem size "
1567:                 << "with --problem-sizes or --problem-sizes-file.\n";
1568:     }
1569:     else if (options.profiling.enable_best_kernel_for_fixed_shape) {
1570:       return profile_cutlass_for_fixed_shape_(options, operation, problem_space);
1571:     }
1572:     else {
1573:       results_.back().status = profile_cutlass_(
1574:         results_.back(),
1575:         options,
1576:         operation,
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 1577-1580
```cpp
1577:         &gemm_workspace_.arguments,
1578:         gemm_workspace_.host_workspace.data(),
1579:         gemm_workspace_.device_workspace.data());
1580:     }
```
- **EN:** Implements `data` for this file's main component.
- **CN:** 为该文件的核心组件实现 `data`。

### Lines 1581-1583
```cpp
1581:   }
1582:   return true;
1583: }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 1585-1585
```cpp
1585: /////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1587-1587
```cpp
1587: /// Method to profile a CUTLASS Operation
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1588-1599
```cpp
1588: Status GroupedGemmOperationProfiler::profile_cutlass_(
1589:   PerformanceResult& result,
1590:   Options const& options,
1591:   library::Operation const* operation,
1592:   void* arguments,
1593:   void* host_workspace,
1594:   void* device_workspace) {
1595:   library::Operation const* underlying_operation = operation;
1596:   result.status = underlying_operation->initialize_with_arguments(&gemm_workspace_.arguments);
1597:   if (result.status != Status::kSuccess) {
1598:     return result.status;
1599:   }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 1601-1601
```cpp
1601:   auto func = [&](cudaStream_t stream, int iteration) {
```
- **EN:** Declares or updates local/member state such as `func`.
- **CN:** 声明或更新局部/成员状态，例如 `func`。

### Lines 1602-1602
```cpp
1602:     // Iterate over copies of the problem in memory
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1603-1604
```cpp
1603:     int workspace_idx = options.profiling.warmup_iterations + iteration;
1604:     int problem_idx = (workspace_idx % gemm_workspace_.problem_count);
```
- **EN:** Declares or updates local/member state such as `workspace_idx`, `iteration`, `problem_idx`.
- **CN:** 声明或更新局部/成员状态，例如 `workspace_idx`, `iteration`, `problem_idx`。

### Lines 1606-1609
```cpp
1606:     gemm_workspace_.arguments.ptr_A = gemm_workspace_.A_ptr_array_device[problem_idx]->data();
1607:     gemm_workspace_.arguments.ptr_B = gemm_workspace_.B_ptr_array_device[problem_idx]->data();
1608:     gemm_workspace_.arguments.ptr_C = gemm_workspace_.C_ptr_array_device[problem_idx]->data();
1609:     gemm_workspace_.arguments.ptr_D = gemm_workspace_.D_ptr_array_device[problem_idx]->data();
```
- **EN:** Implements `data` for this file's main component.
- **CN:** 为该文件的核心组件实现 `data`。

### Lines 1611-1614
```cpp
1611:     return underlying_operation->run(arguments, host_workspace, device_workspace, stream);
1612:   };
1613:   return profile_kernel_(result, options, func, gemm_workspace_.stream);
1614: }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 1616-1616
```cpp
1616: /////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1618-1618
```cpp
1618: /// Method to profile a CUTLASS Operation for the best configuration for a fixed shape
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1619-1624
```cpp
1619: bool GroupedGemmOperationProfiler::profile_cutlass_for_fixed_shape_(
1620:   Options const& options,
1621:   library::Operation const* operation,
1622:   ProblemSpace const& problem_space) {
1623:   library::GroupedGemmDescription const &operation_desc =
1624:     static_cast<library::GroupedGemmDescription const &>(operation->description());
```
- **EN:** Implements `profile_cutlass_for_fixed_shape_` and coordinates helper calls such as `description`.
- **CN:** 实现 `profile_cutlass_for_fixed_shape_`，并协调调用 `description` 等辅助逻辑。

### Lines 1626-1627
```cpp
1626:   auto cluster_shape = operation_desc.tile_description.cluster_shape;
1627:   bool is_dynamic_cluster_enabled = cluster_shape.m() == 0 || cluster_shape.n() == 0 || cluster_shape.k() == 0;
```
- **EN:** Declares or updates local/member state such as `cluster_shape`, `is_dynamic_cluster_enabled`.
- **CN:** 声明或更新局部/成员状态，例如 `cluster_shape`, `is_dynamic_cluster_enabled`。

### Lines 1629-1629
```cpp
1629:   // Helper function to test validity of fallback cluster shapes and preferred cluster shapes.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1630-1637
```cpp
1630:   auto is_valid_dynamic_cluster_shape = [](const std::array<int64_t, 3>& preferred_cluster, const std::array<int64_t, 3>& fallback_cluster) {
1631:     for (size_t i = 0; i < 3; ++i) {
1632:       if (preferred_cluster[i] % fallback_cluster[i] != 0) {
1633:         return false;
1634:       }
1635:     }
1636:     return true;
1637:   };
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 1639-1639
```cpp
1639:   // Helper function to select the best performance number among a list.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1640-1650
```cpp
1640:   auto select_best_candidate = [&](std::vector<PerformanceResult> &candidates) {
1641:     assert(!candidates.empty() && "Candidates vector should not be empty");
1642:     auto best_iter = std::max_element(
1643:       candidates.begin(), candidates.end(),
1644:       [](PerformanceResult const &a, PerformanceResult const &b) {
1645:         return a.gflops_per_sec() < b.gflops_per_sec();
1646:       }
1647:     );
1648:     assert(best_iter != candidates.end() && "No candidate found despite non-empty candidates vector");
1649:     results_.push_back(std::move(*best_iter));
1650:   };
```
- **EN:** Implements `assert` and coordinates helper calls such as `empty`, `max_element`, `begin`.
- **CN:** 实现 `assert`，并协调调用 `empty`, `max_element`, `begin` 等辅助逻辑。

### Lines 1652-1654
```cpp
1652:   std::vector<PerformanceResult> candidates;
1653:   PerformanceResult result_base = results_.back();
1654:   results_.pop_back();
```
- **EN:** Implements `back` and coordinates helper calls such as `pop_back`.
- **CN:** 实现 `back`，并协调调用 `pop_back` 等辅助逻辑。

### Lines 1656-1657
```cpp
1656:   std::vector<std::array<int64_t, 3>> preferred_clusters;
1657:   std::vector<std::array<int64_t, 3>> fallback_clusters;
```
- **EN:** Declares or updates local/member state such as `preferred_clusters`, `fallback_clusters`.
- **CN:** 声明或更新局部/成员状态，例如 `preferred_clusters`, `fallback_clusters`。

### Lines 1659-1660
```cpp
1659:   // Only loop over built-in cluster shape lists for dynamic cluster kernels
1660:   // and for kernels that can leverage the dynamic cluster feature.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1661-1668
```cpp
1661:   if (is_dynamic_cluster_enabled) {
1662:     preferred_clusters = this->problem_.preferred_clusters;
1663:     fallback_clusters = this->problem_.fallback_clusters;
1664:   }
1665:   else {
1666:     preferred_clusters = {{int(problem_.cluster_m), int(problem_.cluster_n), int(problem_.cluster_k)}};
1667:     fallback_clusters = {{int(problem_.cluster_m_fallback), int(problem_.cluster_n_fallback), int(problem_.cluster_k_fallback)}};
1668:   }
```
- **EN:** Declares or updates local/member state such as `preferred_clusters`, `fallback_clusters`.
- **CN:** 声明或更新局部/成员状态，例如 `preferred_clusters`, `fallback_clusters`。

### Lines 1670-1683
```cpp
1670:   for (auto preferred_cluster : preferred_clusters) {
1671:     for (auto fallback_cluster : fallback_clusters) {
1672:       if (is_dynamic_cluster_enabled && !is_valid_dynamic_cluster_shape(preferred_cluster, fallback_cluster)) {
1673:         continue;
1674:       }
1675:       for (auto swizzle_size : this->problem_.swizzle_sizes) {
1676:         for (auto raster_order : this->problem_.raster_orders) {
1677:           PerformanceResult curr_result(result_base);
1678:           update_workspace_and_result_(gemm_workspace_, curr_result, problem_space, raster_order, preferred_cluster, fallback_cluster, swizzle_size, is_dynamic_cluster_enabled);
1679:           curr_result.status  = profile_cutlass_(
1680:             curr_result,
1681:             options,
1682:             operation,
1683:             &gemm_workspace_.arguments,
```
- **EN:** Declares or updates local/member state such as `continue`, `status`.
- **CN:** 声明或更新局部/成员状态，例如 `continue`, `status`。

### Lines 1684-1689
```cpp
1684:             gemm_workspace_.host_workspace.data(),
1685:             gemm_workspace_.device_workspace.data()
1686:           );
1687:           if (curr_result.status == Status::kSuccess) {  // Only add valid results
1688:             candidates.push_back(curr_result);
1689:           }
```
- **EN:** Implements `data` and coordinates helper calls such as `push_back`.
- **CN:** 实现 `data`，并协调调用 `push_back` 等辅助逻辑。

### Lines 1690-1693
```cpp
1690:         }// for raster_order
1691:       }// for swizzle_size
1692:     }// for fallback_cluster
1693:   }// for preferred_clusters
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 1695-1700
```cpp
1695:   if (candidates.empty()) {
1696:     return false;
1697:   }
1698:   select_best_candidate(candidates);
1699:   return true;
1700: }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 1702-1702
```cpp
1702: /////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1704-1705
```cpp
1704: } // namespace profiler
1705: } // namespace cutlass
```
- **EN:** Supporting logic for the grouped GEMM implementation.
- **CN:** 分组 GEMM实现的辅助逻辑。

### Lines 1707-1707
```cpp
1707: /////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

## Key Concepts / 关键概念
- **Conditional compilation / 条件编译**
- **CUDA ecosystem integration / CUDA 生态集成**
- **Profiling workflow / 性能分析流程**
- **Reference implementation / 参考实现**
- **Tensor manipulation / 张量处理**
- **GEMM specialization / GEMM 特化**

## Dependencies / 依赖关系
- **Internal headers / 内部头文件:** `cutlass/cutlass.h`, `cutlass/profiler/grouped_gemm_operation_profiler.h`, `cutlass/library/handle.h`, `cutlass/library/library.h`, `cutlass/library/operation_table.h`, `cutlass/library/singleton.h`
- **External headers / 外部头文件:** `bitset`, `cstdint`, `iostream`, `memory`, `optional`, `stdexcept`, `string`, `vector`
- **Runtime/backends / 运行时与后端:** `CUDA`, `cuBLAS`, `CuTe`
