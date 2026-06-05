# block_scaled_gemm_operation_profiler.cu — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `tools/profiler/src/block_scaled_gemm_operation_profiler.cu`
- **Purpose (EN):** This file implements block-scaled GEMM for the CUTLASS profiler executable.
- **目的 (CN):** 该文件实现了面向CUTLASS profiler 可执行工具的块缩放 GEMM逻辑。
- **Brief / 简述:** Execution environment

## Line-by-Line Analysis / 逐行分析
### Lines 1-20
```cpp
1: /***************************************************************************************************
2:  * Copyright (c) 2017 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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

### Lines 37-41
```cpp
37: #include <iostream>
38: #include <stdexcept>
39: #include <iomanip>
40: #include <ios>
41: #include <vector>
```
- **EN:** Includes the main dependencies required by this module; notable headers are `iostream`, `stdexcept`, `iomanip`, `ios`, `vector`.
- **CN:** 引入该模块所需的主要依赖；较重要的头文件有 `iostream`, `stdexcept`, `iomanip`, `ios`, `vector`。

### Lines 43-43
```cpp
43: #include "cutlass/core_io.h"
```
- **EN:** Includes the main dependencies required by this module; notable headers are `cutlass/core_io.h`.
- **CN:** 引入该模块所需的主要依赖；较重要的头文件有 `cutlass/core_io.h`。

### Lines 45-50
```cpp
45: #include "cutlass/profiler/cublas_helpers.h"
46: #include "cutlass/profiler/block_scaled_gemm_operation_profiler.h"
47: #include "cutlass/profiler/gpu_timer.h"
48: #include "cutlass/library/singleton.h"
49: #include "cutlass/library/library.h"
50: #include "cutlass/library/handle.h"
```
- **EN:** Includes the main dependencies required by this module; notable headers are `cutlass/profiler/cublas_helpers.h`, `cutlass/profiler/block_scaled_gemm_operation_profiler.h`, `cutlass/profiler/gpu_timer.h`, `cutlass/library/singleton.h`, `cutlass/library/library.h`.
- **CN:** 引入该模块所需的主要依赖；较重要的头文件有 `cutlass/profiler/cublas_helpers.h`, `cutlass/profiler/block_scaled_gemm_operation_profiler.h`, `cutlass/profiler/gpu_timer.h`, `cutlass/library/singleton.h`, `cutlass/library/library.h`。

### Lines 52-52
```cpp
52: #include "cutlass/util/reference/host/gett.hpp"
```
- **EN:** Includes the main dependencies required by this module; notable headers are `cutlass/util/reference/host/gett.hpp`.
- **CN:** 引入该模块所需的主要依赖；较重要的头文件有 `cutlass/util/reference/host/gett.hpp`。

### Lines 54-54
```cpp
54: /////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 56-57
```cpp
56: namespace cutlass {
57: namespace profiler {
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 60-60
```cpp
60: /////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 62-62
```cpp
62: /// Ctor
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 63-76
```cpp
63: BlockScaledGemmOperationProfiler::BlockScaledGemmOperationProfiler(Options const &options):
64:   OperationProfiler(
65:     options,
66:     library::OperationKind::kBlockScaledGemm,
67:     {
68:       {ArgumentTypeID::kEnumerated, {"gemm_kind"}, "Variant of GEMM (universal, gemm, planar_complex, planar_complex_array)"},
69:       {ArgumentTypeID::kInteger, {"m", "problem-size::m"}, "M dimension of the GEMM problem space"},
70:       {ArgumentTypeID::kInteger, {"n", "problem-size::n"}, "N dimension of the GEMM problem space"},
71:       {ArgumentTypeID::kInteger, {"k", "problem-size::k"}, "K dimension of the GEMM problem space"},
72:       {ArgumentTypeID::kTensor, {"A"}, "Tensor storing the A operand"},
73:       {ArgumentTypeID::kTensor, {"B"}, "Tensor storing the B operand"},
74:       {ArgumentTypeID::kTensor, {"C"}, "Tensor storing the C operand"},
75:       {ArgumentTypeID::kTensor, {"D"}, "Tensor storing the D output"},
76:       {ArgumentTypeID::kScalar, {"alpha", "epilogue::alpha"}, "Epilogue scalar alpha"},
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 77-88
```cpp
77:       {ArgumentTypeID::kScalar, {"beta", "epilogue::beta"}, "Epilogue scalar beta"},
78:       {ArgumentTypeID::kEnumerated, {"split_k_mode", "split-k-mode"}, "Variant of split K mode(serial, parallel)"},
79:       {ArgumentTypeID::kInteger, {"split_k_slices", "split-k-slices"}, "Number of partitions of K dimension"},
80:       {ArgumentTypeID::kInteger, {"batch_count", "batch-count"}, "Number of GEMMs computed in one batch"},
81:       {ArgumentTypeID::kEnumerated, {"runtime_input_datatype_a", "runtime-input-datatype::a"}, "Runtime datatype (e4m3, e5m2, e3m2, e2m3, e2m1)"}, 
82:       {ArgumentTypeID::kEnumerated, {"runtime_input_datatype_b", "runtime-input-datatype::b"}, "Runtime datatype (e4m3, e5m2, e3m2, e2m3, e2m1)"}, 
83:       {ArgumentTypeID::kEnumerated, {"raster_order", "raster-order"}, "Raster order (heuristic, along_n, along_m)"},
84:       {ArgumentTypeID::kInteger, {"swizzle_size", "swizzle-size"}, "Size to swizzle"},
85:       {ArgumentTypeID::kEnumerated, {"use_pdl", "use_pdl"}, "Use PDL (true, false)"},
86:     },
87:     { library::Provider::kCUBLAS}
88:   ) {
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 90-91
```cpp
90:   description_ = "      General matrix-matrix product. D = alpha * A*B + beta * C";
91: }
```
- **EN:** Declares or updates local/member state such as `description_`, `D`.
- **CN:** 声明或更新局部/成员状态，例如 `description_`, `D`。

### Lines 93-93
```cpp
93: /// Destructor
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 94-94
```cpp
94: BlockScaledGemmOperationProfiler::~BlockScaledGemmOperationProfiler() {
```
- **EN:** Implements `~BlockScaledGemmOperationProfiler` and coordinates helper calls such as `BlockScaledGemmOperationProfiler`.
- **CN:** 实现 `~BlockScaledGemmOperationProfiler`，并协调调用 `BlockScaledGemmOperationProfiler` 等辅助逻辑。

### Lines 96-96
```cpp
96: }
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 98-98
```cpp
98: /// Prints usage statement for the math function
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 99-100
```cpp
99: void BlockScaledGemmOperationProfiler::print_usage(std::ostream &out) const {
100:   out << "Block Scaled GEMM" << "\n\n";
```
- **EN:** Formats human-readable information about command-line usage or current options.
- **CN:** 格式化命令行用法或当前选项的人类可读信息。

### Lines 102-103
```cpp
102:   OperationProfiler::print_usage(out);
103: }
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 105-105
```cpp
105: /// Prints examples
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 106-106
```cpp
106: void BlockScaledGemmOperationProfiler::print_examples(std::ostream &out) const {
```
- **EN:** Implements `print_examples` for this file's main component.
- **CN:** 为该文件的核心组件实现 `print_examples`。

### Lines 108-110
```cpp
108:   out << "\nExamples:\n\n"
109:     << "Profile a particular problem size:\n"
110:     << "  $ cutlass_profiler --operation=block_scaled_gemm --m=1024 --n=1024 --k=128\n\n"
```
- **EN:** Declares or updates local/member state such as `operation`, `m`, `n`, `k`.
- **CN:** 声明或更新局部/成员状态，例如 `operation`, `m`, `n`, `k`。

### Lines 112-113
```cpp
112:     << "Schmoo over problem size and beta:\n"
113:     << "  $ cutlass_profiler --operation=block_scaled_gemm --m=1024:4096:256 --n=1024:4096:256 --k=128:8192:128 --beta=0,1,2.5\n\n"
```
- **EN:** Declares or updates local/member state such as `operation`, `m`, `n`, `k`.
- **CN:** 声明或更新局部/成员状态，例如 `operation`, `m`, `n`, `k`。

### Lines 115-116
```cpp
115:     << "For column major, use column, col, or n. For row major use, row or t:\n"
116:     << "  $ cutlass_profiler --operation=Gemm --A=f16:column --B=*:row\n\n"
```
- **EN:** Declares or updates local/member state such as `operation`, `A`, `B`.
- **CN:** 声明或更新局部/成员状态，例如 `operation`, `A`, `B`。

### Lines 118-119
```cpp
118:     << "Profile a particular problem size with split K and parallel reduction:\n"
119:     << "  $ cutlass_profiler --operation=Gemm --split_k_mode=parallel --split_k_slices=2 --m=1024 --n=1024 --k=128\n\n"
```
- **EN:** Declares or updates local/member state such as `operation`, `split_k_mode`, `split_k_slices`, `m`.
- **CN:** 声明或更新局部/成员状态，例如 `operation`, `split_k_mode`, `split_k_slices`, `m`。

### Lines 121-124
```cpp
121:     << "Using various input value distribution:\n"
122:     << "  $ cutlass_profiler --operation=Gemm --dist=uniform,min:0,max:3\n"
123:     << "  $ cutlass_profiler --operation=Gemm --dist=gaussian,mean:0,stddev:3\n"
124:     << "  $ cutlass_profiler --operation=Gemm --dist=sequential,start:0,delta:1\n\n"
```
- **EN:** Declares or updates local/member state such as `operation`, `dist`.
- **CN:** 声明或更新局部/成员状态，例如 `operation`, `dist`。

### Lines 126-127
```cpp
126:     << "Run a kernel with cta tile size of 256x128x32 and save workspace if results are incorrect (note that --cta-tile::k=32 is default cta-tile size):\n"
127:     << " $ cutlass_profiler --operation=Gemm --cta_m=256 --cta_n=128  --cta_k=32 --save-workspace=incorrect\n\n"
```
- **EN:** Declares or updates local/member state such as `k`, `operation`, `cta_m`, `cta_n`.
- **CN:** 声明或更新局部/成员状态，例如 `k`, `operation`, `cta_m`, `cta_n`。

### Lines 129-136
```cpp
129:     << "Test your changes to gemm kernels with a quick functional test and save results in functional-test.csv:\n"
130:     << " $ cutlass_profiler  --operation=Gemm \\ \n"
131:     << "   --m=8,56,120,136,256,264,512,520,1024,1032,4096,8192,16384 \\ \n"
132:     << "   --n=8,56,120,136,256,264,512,520,1024,1032,4096,8192,16384 \\ \n"
133:     << "   --k=8,16,32,64,128,256,288,384,504,512,520 \\ \n"
134:     << "   --beta=0,1,2 --profiling-iterations=1 \\ \n"
135:     << "   --providers=cutlass --output=functional-test.csv\n\n";
136: }
```
- **EN:** Declares or updates local/member state such as `operation`, `m`, `n`, `k`.
- **CN:** 声明或更新局部/成员状态，例如 `operation`, `m`, `n`, `k`。

### Lines 138-138
```cpp
138: /////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 140-140
```cpp
140: #if 0
```
- **EN:** Conditional-compilation or macro block keyed on `0`.
- **CN:** 以 `0` 为条件的条件编译或宏定义代码块。

### Lines 141-141
```cpp
141: // used this for debugging
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 142-143
```cpp
142: static std::string byte_string(std::vector<uint8_t> const &bytes) {
143:   std::stringstream ss;
```
- **EN:** Implements `byte_string` for this file's main component.
- **CN:** 为该文件的核心组件实现 `byte_string`。

### Lines 145-145
```cpp
145:   ss << "0x";
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 147-149
```cpp
147:   for (size_t idx = bytes.size(); idx > 0; --idx) {
148:     ss << std::hex << std::setw(2) << std::setfill('0') << uint32_t(bytes.at(idx - 1));
149:   }
```
- **EN:** Declares or updates local/member state such as `idx`.
- **CN:** 声明或更新局部/成员状态，例如 `idx`。

### Lines 151-152
```cpp
151:   return ss.str();
152: }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 153-153
```cpp
153: #endif
```
- **EN:** Preprocessor directives configure compilation behavior for this source file.
- **CN:** 预处理指令用于配置该源文件的编译行为。

### Lines 155-158
```cpp
155: Status BlockScaledGemmOperationProfiler::GemmProblem::parse(
156:   library::BlockScaledGemmDescription const &operation_desc,
157:   ProblemSpace const &problem_space,
158:   ProblemSpace::Problem const &problem) {
```
- **EN:** Implements `parse` for this file's main component.
- **CN:** 为该文件的核心组件实现 `parse`。

### Lines 160-160
```cpp
160:   this->mode = library::GemmUniversalMode::kGemm;
```
- **EN:** Declares or updates local/member state such as `mode`, `kGemm`.
- **CN:** 声明或更新局部/成员状态，例如 `mode`, `kGemm`。

### Lines 162-162
```cpp
162:   if (!arg_as_int(this->m, "m", problem_space, problem)) {
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 163-163
```cpp
163:     // default value
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 164-165
```cpp
164:     this->m = 1024;
165:   }
```
- **EN:** Declares or updates local/member state such as `m`.
- **CN:** 声明或更新局部/成员状态，例如 `m`。

### Lines 167-167
```cpp
167:   if (!arg_as_int(this->n, "n", problem_space, problem)) {
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 168-168
```cpp
168:     // default value
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 169-170
```cpp
169:     this->n = 1024;
170:   }
```
- **EN:** Declares or updates local/member state such as `n`.
- **CN:** 声明或更新局部/成员状态，例如 `n`。

### Lines 172-172
```cpp
172:   if (!arg_as_int(this->k, "k", problem_space, problem)) {
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 173-173
```cpp
173:     // default value
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 174-175
```cpp
174:     this->k = 1024;
175:   }
```
- **EN:** Declares or updates local/member state such as `k`.
- **CN:** 声明或更新局部/成员状态，例如 `k`。

### Lines 177-177
```cpp
177:   if (!arg_as_int(this->cluster_m, "cluster_m", problem_space, problem)) {
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 178-178
```cpp
178:     // default value
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 179-180
```cpp
179:     this->cluster_m = std::string(operation_desc.name).find("_2sm") != std::string::npos ? 2 : 1;
180:   }
```
- **EN:** Declares or updates local/member state such as `cluster_m`.
- **CN:** 声明或更新局部/成员状态，例如 `cluster_m`。

### Lines 182-182
```cpp
182:   if (!arg_as_int(this->cluster_n, "cluster_n", problem_space, problem)) {
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 183-183
```cpp
183:     // default value
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 184-185
```cpp
184:     this->cluster_n = 1;
185:   }
```
- **EN:** Declares or updates local/member state such as `cluster_n`.
- **CN:** 声明或更新局部/成员状态，例如 `cluster_n`。

### Lines 187-187
```cpp
187:   if (!arg_as_int(this->cluster_k, "cluster_k", problem_space, problem)) {
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 188-188
```cpp
188:     // default value
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 189-190
```cpp
189:     this->cluster_k = 1;
190:   }
```
- **EN:** Declares or updates local/member state such as `cluster_k`.
- **CN:** 声明或更新局部/成员状态，例如 `cluster_k`。

### Lines 192-192
```cpp
192:   if (!arg_as_int(this->cluster_m_fallback, "cluster_m_fallback", problem_space, problem)) {
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 193-193
```cpp
193:     // default value
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 194-195
```cpp
194:     this->cluster_m_fallback = (this->cluster_m % 2 == 0) ? 2 : 1;
195:   }
```
- **EN:** Declares or updates local/member state such as `cluster_m_fallback`.
- **CN:** 声明或更新局部/成员状态，例如 `cluster_m_fallback`。

### Lines 197-197
```cpp
197:   if (!arg_as_int(this->cluster_n_fallback, "cluster_n_fallback", problem_space, problem)) {
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 198-198
```cpp
198:     // default value
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 199-200
```cpp
199:     this->cluster_n_fallback = 1;
200:   }
```
- **EN:** Declares or updates local/member state such as `cluster_n_fallback`.
- **CN:** 声明或更新局部/成员状态，例如 `cluster_n_fallback`。

### Lines 202-202
```cpp
202:   if (!arg_as_int(this->cluster_k_fallback, "cluster_k_fallback", problem_space, problem)) {
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 203-203
```cpp
203:     // default value
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 204-205
```cpp
204:     this->cluster_k_fallback = 1;
205:   }
```
- **EN:** Declares or updates local/member state such as `cluster_k_fallback`.
- **CN:** 声明或更新局部/成员状态，例如 `cluster_k_fallback`。

### Lines 207-207
```cpp
207:   if (!arg_as_SplitKModeID(this->split_k_mode, "split_k_mode", problem_space, problem)) {
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 208-208
```cpp
208:     // default value
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 209-210
```cpp
209:     this->split_k_mode = library::SplitKMode::kSerial;
210:   }
```
- **EN:** Declares or updates local/member state such as `split_k_mode`, `kSerial`.
- **CN:** 声明或更新局部/成员状态，例如 `split_k_mode`, `kSerial`。

### Lines 212-215
```cpp
212:   this->mode = library::GemmUniversalMode::kGemm;
213:   if (this->split_k_mode == library::SplitKMode::kParallel) {
214:     this->mode = library::GemmUniversalMode::kGemmSplitKParallel;
215:   }
```
- **EN:** Declares or updates local/member state such as `mode`, `kGemm`, `split_k_mode`, `kGemmSplitKParallel`.
- **CN:** 声明或更新局部/成员状态，例如 `mode`, `kGemm`, `split_k_mode`, `kGemmSplitKParallel`。

### Lines 217-217
```cpp
217:   if (!arg_as_int(this->split_k_slices, "split_k_slices", problem_space, problem)) {
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 218-218
```cpp
218:     // default value
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 219-220
```cpp
219:     this->split_k_slices = 1;
220:   }
```
- **EN:** Declares or updates local/member state such as `split_k_slices`.
- **CN:** 声明或更新局部/成员状态，例如 `split_k_slices`。

### Lines 222-225
```cpp
222:   if (this->split_k_mode != library::SplitKMode::kSerial) {
223:     std::cout<<"SplitK/StreamK feature is not supported yet!";
224:     return Status::kErrorInvalidProblem;
225:   }
```
- **EN:** Emits debug or diagnostic output so intermediate state can be inspected.
- **CN:** 输出调试或诊断信息，便于检查中间状态。

### Lines 227-227
```cpp
227:   if (!arg_as_bool(this->use_pdl, "use_pdl", problem_space, problem)) {
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 228-228
```cpp
228:     // default value
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 229-230
```cpp
229:     this->use_pdl = false;
230:   }
```
- **EN:** Declares or updates local/member state such as `use_pdl`, `false`.
- **CN:** 声明或更新局部/成员状态，例如 `use_pdl`, `false`。

### Lines 233-233
```cpp
233:   if (!arg_as_RuntimeDatatype(this->runtime_input_datatype_a, "runtime_input_datatype_a", problem_space, problem)) {
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 234-234
```cpp
234:     // default value
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 235-236
```cpp
235:     this->runtime_input_datatype_a = cutlass::library::RuntimeDatatype::kStatic;
236:   }
```
- **EN:** Declares or updates local/member state such as `runtime_input_datatype_a`, `kStatic`.
- **CN:** 声明或更新局部/成员状态，例如 `runtime_input_datatype_a`, `kStatic`。

### Lines 238-238
```cpp
238:   if (!arg_as_RuntimeDatatype(this->runtime_input_datatype_b, "runtime_input_datatype_b", problem_space, problem)) {
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 239-239
```cpp
239:     // default value
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 240-241
```cpp
240:     this->runtime_input_datatype_b = cutlass::library::RuntimeDatatype::kStatic;
241:   }
```
- **EN:** Declares or updates local/member state such as `runtime_input_datatype_b`, `kStatic`.
- **CN:** 声明或更新局部/成员状态，例如 `runtime_input_datatype_b`, `kStatic`。

### Lines 244-244
```cpp
244:   if (!arg_as_int(this->batch_count, "batch_count", problem_space, problem)) {
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 245-245
```cpp
245:     // default value
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 246-249
```cpp
246:     this->batch_count = 1;
247:   } else if (this->batch_count > 1) {
248:     this->mode = library::GemmUniversalMode::kBatched;
249:   }
```
- **EN:** Declares or updates local/member state such as `batch_count`, `mode`, `kBatched`.
- **CN:** 声明或更新局部/成员状态，例如 `batch_count`, `mode`, `kBatched`。

### Lines 251-251
```cpp
251:   if (!arg_as_int(this->swizzle_size, "swizzle_size", problem_space, problem)) {
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 252-252
```cpp
252:     // default value
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 253-254
```cpp
253:     this->swizzle_size = 1;
254:   }
```
- **EN:** Declares or updates local/member state such as `swizzle_size`.
- **CN:** 声明或更新局部/成员状态，例如 `swizzle_size`。

### Lines 256-256
```cpp
256:   if (!arg_as_RasterOrder(this->raster_order, "raster_order", problem_space, problem)) {
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 257-257
```cpp
257:     // default value
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 258-259
```cpp
258:     this->raster_order = library::RasterOrder::kHeuristic;
259:   }
```
- **EN:** Declares or updates local/member state such as `raster_order`, `kHeuristic`.
- **CN:** 声明或更新局部/成员状态，例如 `raster_order`, `kHeuristic`。

### Lines 261-261
```cpp
261:   if (this->split_k_slices > 1 && this->batch_count > 1) {
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 262-262
```cpp
262:     // At least one of these must be one
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 263-264
```cpp
263:     return Status::kErrorInvalidProblem;
264:   }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 266-268
```cpp
266:   if (!tensor_description_satisfies(operation_desc.A, "A", problem_space, problem)) {
267:     return Status::kErrorInvalidProblem;
268:   }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 270-272
```cpp
270:   if (!tensor_description_satisfies(operation_desc.B, "B", problem_space, problem)) {
271:     return Status::kErrorInvalidProblem;
272:   }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 274-276
```cpp
274:   if (!tensor_description_satisfies(operation_desc.C, "C", problem_space, problem)) {
275:     return Status::kErrorInvalidProblem;
276:   }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 278-280
```cpp
278:   if (!tensor_description_satisfies(operation_desc.D, "D", problem_space, problem)) {
279:     return Status::kErrorInvalidProblem;
280:   }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 282-287
```cpp
282:   if (!arg_as_scalar(
283:     this->alpha,
284:     operation_desc.element_epilogue,
285:     "alpha",
286:     problem_space,
287:     problem)) {
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 289-292
```cpp
289:     if (!cast_from_double(this->alpha, operation_desc.element_epilogue, 1)) {
290:       return Status::kErrorInternal;
291:     }
292:   }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 294-299
```cpp
294:   if (!arg_as_scalar(
295:     this->beta,
296:     operation_desc.element_epilogue,
297:     "beta",
298:     problem_space,
299:     problem)) {
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 301-304
```cpp
301:     if (!cast_from_double(this->beta, operation_desc.element_epilogue, 0)) {
302:       return Status::kErrorInternal;
303:     }
304:   }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 306-307
```cpp
306:   this->lda = DeviceAllocation::get_packed_layout(
307:     operation_desc.A.layout, {int(this->m), int(this->k)}).front();
```
- **EN:** Implements `front` and coordinates helper calls such as `get_packed_layout`, `int`.
- **CN:** 实现 `front`，并协调调用 `get_packed_layout`, `int` 等辅助逻辑。

### Lines 309-310
```cpp
309:   this->ldb = DeviceAllocation::get_packed_layout(
310:     operation_desc.B.layout, {int(this->k), int(this->n)}).front();
```
- **EN:** Implements `front` and coordinates helper calls such as `get_packed_layout`, `int`.
- **CN:** 实现 `front`，并协调调用 `get_packed_layout`, `int` 等辅助逻辑。

### Lines 312-313
```cpp
312:   this->ldc = DeviceAllocation::get_packed_layout(
313:     operation_desc.C.layout, {int(this->m), int(this->n)}).front();
```
- **EN:** Implements `front` and coordinates helper calls such as `get_packed_layout`, `int`.
- **CN:** 实现 `front`，并协调调用 `get_packed_layout`, `int` 等辅助逻辑。

### Lines 315-315
```cpp
315:   // instantiation for deep profiling
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 316-318
```cpp
316:   int num_sizes = 8;
317:   this->problem_sizes.resize(num_sizes);
318:   this->leading_dims.resize(num_sizes, {0, 0, 0});
```
- **EN:** Implements `resize` for this file's main component.
- **CN:** 为该文件的核心组件实现 `resize`。

### Lines 320-332
```cpp
320:   int m0 = 1024;
321:   int n0 = 1024;
322:   int k0 = 1024;
323:   for (int i = 0; i < num_sizes; i++) {
324:     auto m = m0 * (i + 3);
325:     auto n = n0 * (i + 3);
326:     auto k = k0 * (i + 3);
327:     this->problem_sizes[i] = {m, n, k};
328:     this->leading_dims[i] = {
329:       DeviceAllocation::get_packed_layout(operation_desc.A.layout, {int(m), int(k)}).front(),
330:       DeviceAllocation::get_packed_layout(operation_desc.B.layout, {int(k), int(n)}).front(),
331:       DeviceAllocation::get_packed_layout(operation_desc.C.layout, {int(m), int(n)}).front()
332:     };
```
- **EN:** Declares or updates local/member state such as `m0`, `n0`, `k0`, `i`.
- **CN:** 声明或更新局部/成员状态，例如 `m0`, `n0`, `k0`, `i`。

### Lines 334-334
```cpp
334:   }
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 336-339
```cpp
336:   this->raster_orders = {
337:     cutlass::library::RasterOrder::kAlongN,
338:     cutlass::library::RasterOrder::kAlongM
339:   };
```
- **EN:** Declares or updates local/member state such as `raster_orders`.
- **CN:** 声明或更新局部/成员状态，例如 `raster_orders`。

### Lines 341-341
```cpp
341:   this->swizzle_sizes = {1, 2, 4, 8};
```
- **EN:** Declares or updates local/member state such as `swizzle_sizes`.
- **CN:** 声明或更新局部/成员状态，例如 `swizzle_sizes`。

### Lines 343-345
```cpp
343:   this->preferred_clusters = {
344:     {1, 1, 1}, {2, 1, 1}, {2, 2, 1}, {4, 1, 1}, {4, 2, 1}, {4, 4, 1}, {8, 2, 1}
345:   };
```
- **EN:** Declares or updates local/member state such as `preferred_clusters`.
- **CN:** 声明或更新局部/成员状态，例如 `preferred_clusters`。

### Lines 347-349
```cpp
347:   this->fallback_clusters = {
348:     {1, 1, 1}, {2, 1, 1}, {2, 2, 1}
349:   };
```
- **EN:** Declares or updates local/member state such as `fallback_clusters`.
- **CN:** 声明或更新局部/成员状态，例如 `fallback_clusters`。

### Lines 351-352
```cpp
351:   return Status::kSuccess;
352: }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 354-354
```cpp
354: /// Total number of bytes loaded
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 355-357
```cpp
355: int64_t BlockScaledGemmOperationProfiler::GemmProblem::bytes_with_problem_shape(
356:   library::BlockScaledGemmDescription const &operation_desc,
357:   gemm::GemmCoord const &problem_shape) const {
```
- **EN:** Implements `bytes_with_problem_shape` for this file's main component.
- **CN:** 为该文件的核心组件实现 `bytes_with_problem_shape`。

### Lines 358-358
```cpp
358:   // Input bytes read and Output bytes written for the gemm problem
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 359-362
```cpp
359:   int64_t bytes =
360:     int64_t(library::sizeof_bits(operation_desc.A.element) * problem_shape.m() / 8) * problem_shape.k() +
361:     int64_t(library::sizeof_bits(operation_desc.B.element) * problem_shape.n() / 8) * problem_shape.k() +
362:     int64_t(library::sizeof_bits(operation_desc.C.element) * problem_shape.m() / 8) * problem_shape.n();
```
- **EN:** Implements `int64_t` and coordinates helper calls such as `sizeof_bits`, `m`, `k`.
- **CN:** 实现 `int64_t`，并协调调用 `sizeof_bits`, `m`, `k` 等辅助逻辑。

### Lines 364-364
```cpp
364:   // Set is_beta_zero true if beta is zero
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 365-365
```cpp
365:   bool is_beta_zero = std::all_of(beta.begin(), beta.end(), [](uint8_t i) { return i==0; });
```
- **EN:** Implements `all_of` and coordinates helper calls such as `begin`, `end`.
- **CN:** 实现 `all_of`，并协调调用 `begin`, `end` 等辅助逻辑。

### Lines 367-367
```cpp
367:   // Output bytes read for the gemm problem for non-zero beta values
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 368-370
```cpp
368:   if (!is_beta_zero) {
369:     bytes += int64_t(library::sizeof_bits(operation_desc.C.element) * problem_shape.m() / 8) * problem_shape.n();
370:   }
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 372-372
```cpp
372:   bytes *= batch_count;
```
- **EN:** Declares or updates local/member state such as `batch_count`.
- **CN:** 声明或更新局部/成员状态，例如 `batch_count`。

### Lines 374-375
```cpp
374:   return bytes;
375: }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 377-380
```cpp
377: int64_t BlockScaledGemmOperationProfiler::GemmProblem::bytes(library::BlockScaledGemmDescription const &operation_desc) const {
378:   gemm::GemmCoord problem_shape({int(m), int(n), int(k)});
379:   return bytes_with_problem_shape(operation_desc, problem_shape);
380: }
```
- **EN:** Implements `bytes` and coordinates helper calls such as `problem_shape`, `int`, `bytes_with_problem_shape`.
- **CN:** 实现 `bytes`，并协调调用 `problem_shape`, `int`, `bytes_with_problem_shape` 等辅助逻辑。

### Lines 382-382
```cpp
382: /// Total number of flops computed
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 383-386
```cpp
383: int64_t BlockScaledGemmOperationProfiler::GemmProblem::flops_with_problem_shape(
384:   library::BlockScaledGemmDescription const &operation_desc,
385:   gemm::GemmCoord const &problem_shape) const {
386:   int64_t flops_ = (int64_t(problem_shape.m()) * problem_shape.n() * problem_shape.k() + problem_shape.m() * problem_shape.n()) * 2 * batch_count;
```
- **EN:** Implements `flops_with_problem_shape` and coordinates helper calls such as `int64_t`, `m`, `n`.
- **CN:** 实现 `flops_with_problem_shape`，并协调调用 `int64_t`, `m`, `n` 等辅助逻辑。

### Lines 388-388
```cpp
388:   // complex-valued support
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 389-392
```cpp
389:   switch (operation_desc.tile_description.math_instruction.math_operation) {
390:   case library::MathOperationID::kMultiplyAddComplex:
391:     flops_ *= 4;
392:     break;
```
- **EN:** Declares or updates local/member state such as `break`.
- **CN:** 声明或更新局部/成员状态，例如 `break`。

### Lines 394-396
```cpp
394:   case library::MathOperationID::kMultiplyAddComplexFastF32:
395:     flops_ *= 4;
396:     break;
```
- **EN:** Declares or updates local/member state such as `break`.
- **CN:** 声明或更新局部/成员状态，例如 `break`。

### Lines 398-400
```cpp
398:   case library::MathOperationID::kMultiplyAddGaussianComplex:
399:     flops_ *= 3;
400:     break;
```
- **EN:** Declares or updates local/member state such as `break`.
- **CN:** 声明或更新局部/成员状态，例如 `break`。

### Lines 402-403
```cpp
402:   default: break;
403:   }
```
- **EN:** Declares or updates local/member state such as `break`.
- **CN:** 声明或更新局部/成员状态，例如 `break`。

### Lines 405-406
```cpp
405:   return flops_;
406: }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 408-411
```cpp
408: int64_t BlockScaledGemmOperationProfiler::GemmProblem::flops(library::BlockScaledGemmDescription const &operation_desc) const {
409:   gemm::GemmCoord problem_shape({int(m), int(n), int(k)});
410:   return flops_with_problem_shape(operation_desc, problem_shape);
411: }
```
- **EN:** Implements `flops` and coordinates helper calls such as `problem_shape`, `int`, `flops_with_problem_shape`.
- **CN:** 实现 `flops`，并协调调用 `problem_shape`, `int`, `flops_with_problem_shape` 等辅助逻辑。

### Lines 413-413
```cpp
413: /// Initializes a performance result
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 414-417
```cpp
414: void BlockScaledGemmOperationProfiler::GemmProblem::initialize_result(
415:   PerformanceResult &result,
416:   library::BlockScaledGemmDescription const &operation_desc,
417:   ProblemSpace const &problem_space) {
```
- **EN:** Initializes or registers block-scaled GEMM components for later lookup or execution.
- **CN:** 初始化或注册块缩放 GEMM组件，以便后续查找或执行。

### Lines 419-419
```cpp
419:   result.arguments.resize(problem_space.rank());
```
- **EN:** Implements `resize` and coordinates helper calls such as `rank`.
- **CN:** 实现 `resize`，并协调调用 `rank` 等辅助逻辑。

### Lines 421-421
```cpp
421:   set_argument(result, "gemm_kind", problem_space, library::to_string(operation_desc.gemm_kind));
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 423-424
```cpp
423:   set_argument(result, "A", problem_space,
424:     std::string(library::to_string(operation_desc.A.element)) + ":" + library::to_string(operation_desc.A.layout));
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 426-427
```cpp
426:   set_argument(result, "B", problem_space,
427:     std::string(library::to_string(operation_desc.B.element)) + ":" + library::to_string(operation_desc.B.layout));
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 429-430
```cpp
429:   set_argument(result, "C", problem_space,
430:     std::string(library::to_string(operation_desc.C.element)) + ":" + library::to_string(operation_desc.C.layout));
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 432-433
```cpp
432:   set_argument(result, "D", problem_space,
433:     std::string(library::to_string(operation_desc.D.element)) + ":" + library::to_string(operation_desc.D.layout));
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 435-437
```cpp
435:   set_argument(result, "m", problem_space, m);
436:   set_argument(result, "n", problem_space, n);
437:   set_argument(result, "k", problem_space, k);
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 440-447
```cpp
440:   auto cluster_shape = operation_desc.tile_description.cluster_shape;
441:   auto is_dynamic = cluster_shape.m() == 0 || cluster_shape.n() == 0 || cluster_shape.k() == 0;
442:   set_argument(result, "cluster_m", problem_space, is_dynamic ? this->cluster_m : cluster_shape.m());
443:   set_argument(result, "cluster_n", problem_space, is_dynamic ? this->cluster_n : cluster_shape.n());
444:   set_argument(result, "cluster_k", problem_space, is_dynamic ? this->cluster_k : cluster_shape.k());
445:   set_argument(result, "cluster_m_fallback", problem_space, cluster_m_fallback);
446:   set_argument(result, "cluster_n_fallback", problem_space, cluster_n_fallback);
447:   set_argument(result, "cluster_k_fallback", problem_space, cluster_k_fallback);
```
- **EN:** Implements `set_argument` and coordinates helper calls such as `m`, `n`, `k`.
- **CN:** 实现 `set_argument`，并协调调用 `m`, `n`, `k` 等辅助逻辑。

### Lines 450-455
```cpp
450:   set_argument(result, "split_k_mode", problem_space, library::to_string(split_k_mode));
451:   set_argument(result, "split_k_slices", problem_space, split_k_slices);
452:   set_argument(result, "batch_count", problem_space, batch_count);
453:   set_argument(result, "raster_order", problem_space, library::to_string(raster_order));
454:   set_argument(result, "swizzle_size", problem_space, swizzle_size);
455:   set_argument(result, "use_pdl", problem_space, library::to_string(use_pdl));
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 458-459
```cpp
458:   set_argument(result, "runtime_input_datatype_a", problem_space, library::to_string(runtime_input_datatype_a));
459:   set_argument(result, "runtime_input_datatype_b", problem_space, library::to_string(runtime_input_datatype_b));
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 462-463
```cpp
462:   set_argument(result, "alpha", problem_space,
463:     library::lexical_cast(alpha, operation_desc.element_epilogue));
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 465-467
```cpp
465:   set_argument(result, "beta", problem_space,
466:     library::lexical_cast(beta, operation_desc.element_epilogue));
467: }
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 469-469
```cpp
469: /////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 471-471
```cpp
471: /// Extracts the problem dimensions
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 472-478
```cpp
472: Status BlockScaledGemmOperationProfiler::initialize_configuration(
473:     Options const &options,
474:     PerformanceReport &report,
475:     DeviceContext &device_context,
476:     library::Operation const *operation,
477:     ProblemSpace const &problem_space,
478:     ProblemSpace::Problem const &problem) {
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 480-481
```cpp
480:   library::BlockScaledGemmDescription const &operation_desc =
481:     static_cast<library::BlockScaledGemmDescription const &>(operation->description());
```
- **EN:** Implements `description` for this file's main component.
- **CN:** 为该文件的核心组件实现 `description`。

### Lines 483-485
```cpp
483:   if (operation_desc.gemm_kind != library::GemmKind::kUniversal) {
484:     return Status::kErrorInvalidProblem;
485:   }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 487-487
```cpp
487:   Status status = problem_.parse(operation_desc, problem_space, problem);
```
- **EN:** Implements `parse` for this file's main component.
- **CN:** 为该文件的核心组件实现 `parse`。

### Lines 489-491
```cpp
489:   if (status != Status::kSuccess) {
490:     return status;
491:   }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 493-496
```cpp
493:   gemm_workspace_.configuration.mode = problem_.mode;
494:   gemm_workspace_.configuration.problem_size.m() = int(problem_.m);
495:   gemm_workspace_.configuration.problem_size.n() = int(problem_.n);
496:   gemm_workspace_.configuration.problem_size.k() = int(problem_.k);
```
- **EN:** Implements `m` and coordinates helper calls such as `int`, `n`, `k`.
- **CN:** 实现 `m`，并协调调用 `int`, `n`, `k` 等辅助逻辑。

### Lines 498-503
```cpp
498:   gemm_workspace_.configuration.cluster_shape.m() = int(problem_.cluster_m);
499:   gemm_workspace_.configuration.cluster_shape.n() = int(problem_.cluster_n);
500:   gemm_workspace_.configuration.cluster_shape.k() = int(problem_.cluster_k);
501:   gemm_workspace_.configuration.cluster_shape_fallback.m() = int(problem_.cluster_m_fallback);
502:   gemm_workspace_.configuration.cluster_shape_fallback.n() = int(problem_.cluster_n_fallback);
503:   gemm_workspace_.configuration.cluster_shape_fallback.k() = int(problem_.cluster_k_fallback);
```
- **EN:** Implements `m` and coordinates helper calls such as `int`, `n`, `k`.
- **CN:** 实现 `m`，并协调调用 `int`, `n`, `k` 等辅助逻辑。

### Lines 505-508
```cpp
505:   gemm_workspace_.configuration.lda = problem_.lda;
506:   gemm_workspace_.configuration.ldb = problem_.ldb;
507:   gemm_workspace_.configuration.ldc = problem_.ldc;
508:   gemm_workspace_.configuration.ldd = problem_.ldc;
```
- **EN:** Declares or updates local/member state such as `lda`, `ldb`, `ldc`, `ldd`.
- **CN:** 声明或更新局部/成员状态，例如 `lda`, `ldb`, `ldc`, `ldd`。

### Lines 510-515
```cpp
510:   if (problem_.mode == library::GemmUniversalMode::kBatched) {
511:     gemm_workspace_.configuration.batch_count = problem_.batch_count;
512:   }
513:   else {
514:     gemm_workspace_.configuration.batch_count = problem_.split_k_slices;
515:   }
```
- **EN:** Declares or updates local/member state such as `mode`, `batch_count`, `split_k_slices`.
- **CN:** 声明或更新局部/成员状态，例如 `mode`, `batch_count`, `split_k_slices`。

### Lines 517-520
```cpp
517:   gemm_workspace_.arguments.problem_size.m() = int(problem_.m);
518:   gemm_workspace_.arguments.problem_size.n() = int(problem_.n);
519:   gemm_workspace_.arguments.problem_size.k() = int(problem_.k);
520:   gemm_workspace_.arguments.batch_count = problem_.batch_count;
```
- **EN:** Implements `m` and coordinates helper calls such as `int`, `n`, `k`.
- **CN:** 实现 `m`，并协调调用 `int`, `n`, `k` 等辅助逻辑。

### Lines 522-534
```cpp
522:   gemm_workspace_.arguments.A = nullptr;
523:   gemm_workspace_.arguments.B = nullptr;
524:   gemm_workspace_.arguments.C = nullptr;
525:   gemm_workspace_.arguments.D = nullptr;
526:   gemm_workspace_.arguments.alpha = problem_.alpha.data();
527:   gemm_workspace_.arguments.beta = problem_.beta.data();
528:   gemm_workspace_.arguments.pointer_mode = library::ScalarPointerMode::kHost;
529:   gemm_workspace_.arguments.swizzle_size = problem_.swizzle_size;
530:   gemm_workspace_.arguments.raster_order = problem_.raster_order;
531:   gemm_workspace_.arguments.norm_constant = 0;                    
532:   gemm_workspace_.arguments.cluster_shape = {int(problem_.cluster_m), int(problem_.cluster_n), int(problem_.cluster_k)};
533:   gemm_workspace_.arguments.cluster_shape_fallback = {int(problem_.cluster_m_fallback), int(problem_.cluster_n_fallback), int(problem_.cluster_k_fallback)};
534:   gemm_workspace_.arguments.split_k_slices = problem_.split_k_slices;
```
- **EN:** Declares or updates local/member state such as `A`, `nullptr`, `B`, `C`.
- **CN:** 声明或更新局部/成员状态，例如 `A`, `nullptr`, `B`, `C`。

### Lines 537-538
```cpp
537:   gemm_workspace_.arguments.runtime_input_datatype_a = problem_.runtime_input_datatype_a;
538:   gemm_workspace_.arguments.runtime_input_datatype_b = problem_.runtime_input_datatype_b;
```
- **EN:** Declares or updates local/member state such as `runtime_input_datatype_a`, `runtime_input_datatype_b`.
- **CN:** 声明或更新局部/成员状态，例如 `runtime_input_datatype_a`, `runtime_input_datatype_b`。

### Lines 541-541
```cpp
541:   gemm_workspace_.arguments.use_pdl = problem_.use_pdl;
```
- **EN:** Declares or updates local/member state such as `use_pdl`.
- **CN:** 声明或更新局部/成员状态，例如 `use_pdl`。

### Lines 543-543
```cpp
543:   cudaStreamCreateWithFlags(&gemm_workspace_.stream, cudaStreamNonBlocking);
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 545-545
```cpp
545:   // initialize reduction operation for parallel splitKMode
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 546-550
```cpp
546:   if (problem_.split_k_mode == library::SplitKMode::kParallel) {
547:     if (!initialize_reduction_configuration_(operation, problem)) {
548:       return Status::kErrorInternal;
549:     }
550:   }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 552-553
```cpp
552:   initialize_result_(this->model_result_, options, operation_desc, problem_space);
553:   return operation->can_implement(&gemm_workspace_.configuration, &gemm_workspace_.arguments);
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 555-555
```cpp
555: }
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 557-566
```cpp
557: void BlockScaledGemmOperationProfiler::update_workspace_(
558:   GemmWorkspace &gemm_workspace,
559:   gemm::GemmCoord const &problem_shape,
560:   std::array<int64_t, 3> const &leading_dim,
561:   std::array<int64_t, 3> const &preferred_cluster,
562:   std::array<int64_t, 3> const &fallback_cluster,
563:   cutlass::library::RasterOrder const &raster_order,
564:   int swizzle_size,
565:   bool is_dynamic_cluster_enabled
566: ) {
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 568-570
```cpp
568:   gemm_workspace.arguments.problem_size.m() = problem_shape.m();
569:   gemm_workspace.arguments.problem_size.n() = problem_shape.n();
570:   gemm_workspace.arguments.problem_size.k() = problem_shape.k();
```
- **EN:** Implements `m` and coordinates helper calls such as `n`, `k`.
- **CN:** 实现 `m`，并协调调用 `n`, `k` 等辅助逻辑。

### Lines 572-574
```cpp
572:   gemm_workspace.arguments.lda = leading_dim[0];
573:   gemm_workspace.arguments.ldb = leading_dim[1];
574:   gemm_workspace.arguments.ldc = leading_dim[2];
```
- **EN:** Declares or updates local/member state such as `lda`, `ldb`, `ldc`.
- **CN:** 声明或更新局部/成员状态，例如 `lda`, `ldb`, `ldc`。

### Lines 576-577
```cpp
576:   gemm_workspace.arguments.swizzle_size = swizzle_size;
577:   gemm_workspace.arguments.raster_order = raster_order;
```
- **EN:** Declares or updates local/member state such as `swizzle_size`, `raster_order`.
- **CN:** 声明或更新局部/成员状态，例如 `swizzle_size`, `raster_order`。

### Lines 579-584
```cpp
579:   if (is_dynamic_cluster_enabled) {
580:     gemm_workspace.arguments.cluster_shape = {int(preferred_cluster[0]), int(preferred_cluster[1]), int(preferred_cluster[2])};
581:     gemm_workspace.arguments.cluster_shape_fallback = {int(fallback_cluster[0]), int(fallback_cluster[1]), int(fallback_cluster[2])};
582:     gemm_workspace.configuration.cluster_shape = {int(preferred_cluster[0]), int(preferred_cluster[1]), int(preferred_cluster[2])};
583:     gemm_workspace.configuration.cluster_shape_fallback = {int(fallback_cluster[0]), int(fallback_cluster[1]), int(fallback_cluster[2])};
584:   }
```
- **EN:** Declares or updates local/member state such as `cluster_shape`, `cluster_shape_fallback`.
- **CN:** 声明或更新局部/成员状态，例如 `cluster_shape`, `cluster_shape_fallback`。

### Lines 586-588
```cpp
586:   gemm_workspace.configuration.problem_size.m() = problem_shape.m();
587:   gemm_workspace.configuration.problem_size.n() = problem_shape.n();
588:   gemm_workspace.configuration.problem_size.k() = problem_shape.k();
```
- **EN:** Implements `m` and coordinates helper calls such as `n`, `k`.
- **CN:** 实现 `m`，并协调调用 `n`, `k` 等辅助逻辑。

### Lines 591-593
```cpp
591:   gemm_workspace.configuration.lda = leading_dim[0];
592:   gemm_workspace.configuration.ldb = leading_dim[1];
593:   gemm_workspace.configuration.ldc = leading_dim[2];
```
- **EN:** Declares or updates local/member state such as `lda`, `ldb`, `ldc`.
- **CN:** 声明或更新局部/成员状态，例如 `lda`, `ldb`, `ldc`。

### Lines 595-595
```cpp
595: }
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 598-611
```cpp
598: void BlockScaledGemmOperationProfiler::update_result_(
599:   PerformanceResult &result,
600:   library::BlockScaledGemmDescription const &operation_desc,
601:   ProblemSpace const &problem_space,
602:   gemm::GemmCoord const &problem_shape,
603:   cutlass::library::RasterOrder const &raster_order,
604:   std::array<int64_t, 3> const &preferred_cluster,
605:   std::array<int64_t, 3> const &fallback_cluster,
606:   int swizzle_size,
607:   bool is_dynamic_cluster_enabled
608: ) {
609:   result.bytes = problem_.bytes_with_problem_shape(operation_desc, problem_shape);
610:   result.flops = problem_.flops_with_problem_shape(operation_desc, problem_shape);
611:   set_argument(result, "m", problem_space, problem_shape.m());
```
- **EN:** Declares or updates local/member state such as `bytes`, `flops`.
- **CN:** 声明或更新局部/成员状态，例如 `bytes`, `flops`。

### Lines 612-613
```cpp
612:   set_argument(result, "n", problem_space, problem_shape.n());
613:   set_argument(result, "k", problem_space, problem_shape.k());
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 615-616
```cpp
615:   set_argument(result, "raster_order", problem_space, library::to_string(raster_order));
616:   set_argument(result, "swizzle_size", problem_space, swizzle_size);
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 618-625
```cpp
618:   if (is_dynamic_cluster_enabled) {
619:     set_argument(result, "cluster_m", problem_space, preferred_cluster[0]);
620:     set_argument(result, "cluster_n", problem_space, preferred_cluster[1]);
621:     set_argument(result, "cluster_k", problem_space, preferred_cluster[2]);
622:     set_argument(result, "cluster_m_fallback", problem_space, fallback_cluster[0]);
623:     set_argument(result, "cluster_n_fallback", problem_space, fallback_cluster[1]);
624:     set_argument(result, "cluster_k_fallback", problem_space, fallback_cluster[2]);
625:   }
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 627-627
```cpp
627: }
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 629-629
```cpp
629: /// Initializes the performance result
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 630-634
```cpp
630: void BlockScaledGemmOperationProfiler::initialize_result_(
631:     PerformanceResult &result,
632:     Options const &options,
633:     library::BlockScaledGemmDescription const &operation_desc,
634:     ProblemSpace const &problem_space) {
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 636-639
```cpp
636:   result.provider = library::Provider::kCUTLASS;
637:   result.disposition = Disposition::kNotRun;
638:   result.status = Status::kSuccess;
639:   result.operation_name = operation_desc.name;
```
- **EN:** Declares or updates local/member state such as `provider`, `kCUTLASS`, `disposition`, `kNotRun`.
- **CN:** 声明或更新局部/成员状态，例如 `provider`, `kCUTLASS`, `disposition`, `kNotRun`。

### Lines 641-641
```cpp
641:   problem_.initialize_result(result, operation_desc, problem_space);
```
- **EN:** Initializes or registers block-scaled GEMM components for later lookup or execution.
- **CN:** 初始化或注册块缩放 GEMM组件，以便后续查找或执行。

### Lines 643-643
```cpp
643:   OperationProfiler::initialize_result_(result, operation_desc, problem_space);
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 645-648
```cpp
645:   result.bytes = problem_.bytes(operation_desc);
646:   result.flops = problem_.flops(operation_desc);
647:   result.runtime = 0;
648:   result.runtime_vector.resize(options.device.devices.size(), 0);
```
- **EN:** Implements `bytes` and coordinates helper calls such as `flops`, `resize`, `size`.
- **CN:** 实现 `bytes`，并协调调用 `flops`, `resize`, `size` 等辅助逻辑。

### Lines 650-650
```cpp
650: }
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 652-652
```cpp
652: /// Initialize reduction problem dimensions and library::Operation
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 653-655
```cpp
653: bool BlockScaledGemmOperationProfiler::initialize_reduction_configuration_(
654:   library::Operation const *operation,
655:   ProblemSpace::Problem const &problem) {
```
- **EN:** Initializes or registers block-scaled GEMM components for later lookup or execution.
- **CN:** 初始化或注册块缩放 GEMM组件，以便后续查找或执行。

### Lines 657-658
```cpp
657:   library::BlockScaledGemmDescription const &gemm_desc =
658:     static_cast<library::BlockScaledGemmDescription const&>(operation->description());
```
- **EN:** Implements `description` for this file's main component.
- **CN:** 为该文件的核心组件实现 `description`。

### Lines 660-662
```cpp
660:   if (!cast_from_double(problem_.alpha_one, gemm_desc.element_epilogue, 1)) {
661:     return false;
662:   }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 664-666
```cpp
664:   if (!cast_from_double(problem_.beta_zero, gemm_desc.element_epilogue, 0)) {
665:     return false;
666:   }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 668-668
```cpp
668:   /// initialize library::ReductionConfiguration
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 669-674
```cpp
669:   gemm_workspace_.reduction_configuration.problem_size      = gemm::GemmCoord(int(problem_.n), int(problem_.m), int(problem_.k)).mn();
670:   gemm_workspace_.reduction_configuration.partitions        = int(problem_.split_k_slices);
671:   gemm_workspace_.reduction_configuration.partition_stride  = gemm::GemmCoord(int(problem_.n), int(problem_.m), int(problem_.k)).mn().product();
672:   gemm_workspace_.reduction_configuration.ldw               = problem_.ldc;
673:   gemm_workspace_.reduction_configuration.lds               = problem_.ldc;
674:   gemm_workspace_.reduction_configuration.ldd               = problem_.ldc;
```
- **EN:** Implements `GemmCoord` and coordinates helper calls such as `int`, `mn`, `product`.
- **CN:** 实现 `GemmCoord`，并协调调用 `int`, `mn`, `product` 等辅助逻辑。

### Lines 676-676
```cpp
676:   // find reduction operation
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 677-683
```cpp
677:   library::ReductionFunctionalKey reduction_key(
678:     library::Provider::kCUTLASS,
679:     gemm_desc.tile_description.math_instruction.element_accumulator,    // element workspace
680:     gemm_desc.tile_description.math_instruction.element_accumulator,    // element accumulator
681:     gemm_desc.D.element,                                                // element output
682:     gemm_desc.element_epilogue                                          // element compute
683:   );
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 685-685
```cpp
685:   auto reduction_it = library::Singleton::get().operation_table.reduction_operations.find(reduction_key);
```
- **EN:** Implements `get` and coordinates helper calls such as `find`.
- **CN:** 实现 `get`，并协调调用 `find` 等辅助逻辑。

### Lines 687-689
```cpp
687:   if (reduction_it == library::Singleton::get().operation_table.reduction_operations.end()) {
688:     return false;
689:   }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 691-691
```cpp
691:   // initialize reduction operation required for parallel split-k operator
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 692-692
```cpp
692:   reduction_op_ = reduction_it->second;
```
- **EN:** Declares or updates local/member state such as `reduction_op_`, `second`.
- **CN:** 声明或更新局部/成员状态，例如 `reduction_op_`, `second`。

### Lines 694-694
```cpp
694:   // reduction operation found and initialized
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 695-696
```cpp
695:   return true;
696: }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 698-698
```cpp
698: /// Initializes workspace
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 699-705
```cpp
699: Status BlockScaledGemmOperationProfiler::initialize_workspace(
700:   Options const &options,
701:   PerformanceReport &report,
702:   DeviceContext &device_context,
703:   library::Operation const *operation,
704:   ProblemSpace const &problem_space,
705:   ProblemSpace::Problem const &problem) {
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 707-710
```cpp
707:   if (options.device.devices.size() != 1) {
708:     throw std::runtime_error("This operation profiler only supports a single "
709:                              "device.");
710:   }
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 712-716
```cpp
712:   cudaError_t result;
713:   result = cudaSetDevice(options.device.device_id(0));
714:   if (result != cudaSuccess) {
715:     throw std::runtime_error("cudaSetDevice() failed.");
716:   }
```
- **EN:** Implements `cudaSetDevice` and coordinates helper calls such as `device_id`, `runtime_error`.
- **CN:** 实现 `cudaSetDevice`，并协调调用 `device_id`, `runtime_error` 等辅助逻辑。

### Lines 718-718
```cpp
718:   library::Operation const* underlying_operation = operation;
```
- **EN:** Declares or updates local/member state such as `underlying_operation`, `operation`.
- **CN:** 声明或更新局部/成员状态，例如 `underlying_operation`, `operation`。

### Lines 720-724
```cpp
720:   if (problem_.split_k_mode == library::SplitKMode::kParallel) {
721:     if (!(underlying_operation = library::find_gemm_operation_for_parallel_reduction(operation))) {
722:       return Status::kErrorNotSupported;
723:     }
724:   }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 726-727
```cpp
726:   library::BlockScaledGemmDescription const &operation_desc =
727:     static_cast<library::BlockScaledGemmDescription const &>(operation->description());
```
- **EN:** Implements `description` for this file's main component.
- **CN:** 为该文件的核心组件实现 `description`。

### Lines 729-729
```cpp
729:   // Compute the number of copies of the problem to avoid L2 camping.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 730-742
```cpp
730:   if (!options.profiling.workspace_count) {
731:     int64_t bytes = problem_.bytes(operation_desc);
732:     if (bytes < 3 * int64_t(options.device.properties[0].l2CacheSize)) {
733:       gemm_workspace_.problem_count =
734:         1 + int((3 * int64_t(options.device.properties[0].l2CacheSize)) / bytes);
735:     }
736:     else {
737:       gemm_workspace_.problem_count = 1;
738:     }
739:   }
740:   else {
741:     gemm_workspace_.problem_count = options.profiling.workspace_count;
742:   }
```
- **EN:** Declares or updates local/member state such as `bytes`, `problem_count`, `workspace_count`.
- **CN:** 声明或更新局部/成员状态，例如 `bytes`, `problem_count`, `workspace_count`。

### Lines 744-747
```cpp
744:   bool allocate_device_tensors = options.execution_mode != ExecutionMode::kDryRun;
745:   if (allocate_device_tensors) {
746:     bool enable_deep_profiling = options.profiling.enable_kernel_performance_search;
747:     int seed_shift = 0;
```
- **EN:** Declares or updates local/member state such as `allocate_device_tensors`, `kDryRun`, `enable_deep_profiling`, `enable_kernel_performance_search`.
- **CN:** 声明或更新局部/成员状态，例如 `allocate_device_tensors`, `kDryRun`, `enable_deep_profiling`, `enable_kernel_performance_search`。

### Lines 749-750
```cpp
749:     // When exhaustive performance search (deep profiling) option is enabled, device buffers are initialized to the largest problem shape
750:     // so that later performance search can re-use those buffers.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 751-756
```cpp
751:     int init_m = enable_deep_profiling ? std::max(int(problem_.m),  problem_.problem_sizes.back().m()) : int(problem_.m);
752:     int init_n = enable_deep_profiling ? std::max(int(problem_.n),  problem_.problem_sizes.back().n()) : int(problem_.n);
753:     int init_k = enable_deep_profiling ? std::max(int(problem_.k),  problem_.problem_sizes.back().k()) : int(problem_.k);
754:     int init_lda = enable_deep_profiling ? int(std::max(problem_.lda,  problem_.leading_dims.back()[0])) : int(problem_.lda);
755:     int init_ldb = enable_deep_profiling ? int(std::max(problem_.ldb,  problem_.leading_dims.back()[1])) : int(problem_.ldb);
756:     int init_ldc = enable_deep_profiling ? int(std::max(problem_.ldc,  problem_.leading_dims.back()[2])) : int(problem_.ldc);
```
- **EN:** Implements `max` and coordinates helper calls such as `int`, `back`, `m`.
- **CN:** 实现 `max`，并协调调用 `int`, `back`, `m` 等辅助逻辑。

### Lines 758-768
```cpp
758:     gemm_workspace_.A = device_context.allocate_and_initialize_tensor(
759:       options,
760:       "A",
761:       operation_desc.A.element,
762:       operation_desc.A.layout,
763:       {init_m, init_k},
764:       {init_lda},
765:       problem_.batch_count * gemm_workspace_.problem_count,
766:       seed_shift++,
767:       0 // device_index
768:     );
```
- **EN:** Declares or updates local/member state such as `A`.
- **CN:** 声明或更新局部/成员状态，例如 `A`。

### Lines 770-772
```cpp
770:     int sfa_m     = round_up(init_m, 128);
771:     int sfb_n     = round_up(init_n, 128);
772:     int sfa_sfb_k = round_up(ceil_div(init_k, operation_desc.SFVecSize), 4);
```
- **EN:** Implements `round_up` and coordinates helper calls such as `ceil_div`.
- **CN:** 实现 `round_up`，并协调调用 `ceil_div` 等辅助逻辑。

### Lines 774-777
```cpp
774:     int sfd_m     = operation_desc.SFD.layout == cutlass::library::LayoutTypeID::kRowMajor ?
775:                       sfa_m : round_up(ceil_div(init_m, operation_desc.EpilogueSFVecSize), 4);
776:     int sfd_n     = operation_desc.SFD.layout == cutlass::library::LayoutTypeID::kRowMajor ?
777:                       round_up(ceil_div(init_n, operation_desc.EpilogueSFVecSize), 4) : sfb_n;
```
- **EN:** Implements `round_up` and coordinates helper calls such as `ceil_div`.
- **CN:** 实现 `round_up`，并协调调用 `ceil_div` 等辅助逻辑。

### Lines 780-790
```cpp
780:     gemm_workspace_.SFA = device_context.allocate_and_initialize_tensor(
781:       options,
782:       "SFA",
783:       operation_desc.SFA.element,
784:       operation_desc.SFA.layout,
785:       {sfa_m, sfa_sfb_k},
786:       {sfa_sfb_k},
787:       problem_.batch_count * gemm_workspace_.problem_count,
788:       seed_shift++,
789:       0 // device_index
790:     );
```
- **EN:** Declares or updates local/member state such as `SFA`.
- **CN:** 声明或更新局部/成员状态，例如 `SFA`。

### Lines 792-802
```cpp
792:     gemm_workspace_.SFB = device_context.allocate_and_initialize_tensor(
793:       options,
794:       "SFB",
795:       operation_desc.SFB.element,
796:       operation_desc.SFB.layout,
797:       {sfb_n, sfa_sfb_k},
798:       {sfa_sfb_k},
799:       problem_.batch_count * gemm_workspace_.problem_count,
800:       seed_shift++,
801:       0 // device_index
802:     );
```
- **EN:** Declares or updates local/member state such as `SFB`.
- **CN:** 声明或更新局部/成员状态，例如 `SFB`。

### Lines 804-814
```cpp
804:     gemm_workspace_.B = device_context.allocate_and_initialize_tensor(
805:       options,
806:       "B",
807:       operation_desc.B.element,
808:       operation_desc.B.layout,
809:       {init_k, init_n},
810:       {init_ldb},
811:       problem_.batch_count * gemm_workspace_.problem_count,
812:       seed_shift++,
813:       0 // device_index
814:     );
```
- **EN:** Declares or updates local/member state such as `B`.
- **CN:** 声明或更新局部/成员状态，例如 `B`。

### Lines 816-826
```cpp
816:     gemm_workspace_.C = device_context.allocate_and_initialize_tensor(
817:       options,
818:       "C",
819:       operation_desc.C.element,
820:       operation_desc.C.layout,
821:       {init_m, init_n},
822:       {init_ldc},
823:       problem_.batch_count * gemm_workspace_.problem_count,
824:       seed_shift++,
825:       0 // device_index
826:     );
```
- **EN:** Declares or updates local/member state such as `C`.
- **CN:** 声明或更新局部/成员状态，例如 `C`。

### Lines 828-837
```cpp
828:     gemm_workspace_.Computed = device_context.allocate_tensor(
829:       options,
830:       "D",
831:       operation_desc.D.element,
832:       operation_desc.D.layout,
833:       {init_m, init_n},
834:       {init_ldc},
835:       problem_.batch_count * gemm_workspace_.problem_count,
836:       0 // device_index
837:     );
```
- **EN:** Declares or updates local/member state such as `Computed`.
- **CN:** 声明或更新局部/成员状态，例如 `Computed`。

### Lines 839-848
```cpp
839:     gemm_workspace_.Reference = device_context.allocate_tensor(
840:       options,
841:       "Reference",
842:       operation_desc.D.element,
843:       operation_desc.D.layout,
844:       {init_m, init_n},
845:       {init_ldc},
846:       problem_.batch_count * gemm_workspace_.problem_count,
847:       0 // device_index
848:     );
```
- **EN:** Declares or updates local/member state such as `Reference`.
- **CN:** 声明或更新局部/成员状态，例如 `Reference`。

### Lines 850-859
```cpp
850:     gemm_workspace_.Computed_SFD = device_context.allocate_tensor(
851:       options,
852:       "SFD",
853:       operation_desc.SFD.element,
854:       operation_desc.SFD.layout,
855:       {sfd_m, sfd_n},
856:       {sfd_n},
857:       problem_.batch_count * gemm_workspace_.problem_count,
858:       0 // device_index
859:     );
```
- **EN:** Declares or updates local/member state such as `Computed_SFD`.
- **CN:** 声明或更新局部/成员状态，例如 `Computed_SFD`。

### Lines 861-870
```cpp
861:     gemm_workspace_.Reference_SFD = device_context.allocate_tensor(
862:       options,
863:       "Reference_SFD",
864:       operation_desc.SFD.element,
865:       operation_desc.SFD.layout,
866:       {sfd_m, sfd_n},
867:       {sfd_n},
868:       problem_.batch_count * gemm_workspace_.problem_count,
869:       0 // device_index
870:     );
```
- **EN:** Declares or updates local/member state such as `Reference_SFD`.
- **CN:** 声明或更新局部/成员状态，例如 `Reference_SFD`。

### Lines 872-882
```cpp
872:     gemm_workspace_.Norm_constant = device_context.allocate_and_initialize_tensor(
873:       options,
874:       "Norm_constant",
875:       operation_desc.element_epilogue,
876:       operation_desc.A.layout,
877:       {1, 1},
878:       {1},
879:       1,
880:       seed_shift++,
881:       0 // device_index
882:     );
```
- **EN:** Declares or updates local/member state such as `Norm_constant`.
- **CN:** 声明或更新局部/成员状态，例如 `Norm_constant`。

### Lines 884-884
```cpp
884:   }
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 886-886
```cpp
886:   if (options.execution_mode != ExecutionMode::kDryRun) {
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 888-889
```cpp
888:     // ScaleFactor tensor results may have some holes and will not be touched by the kernel.
889:     // If we randomly fill the two tensors, these holes may encounter refcheck errors.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 890-899
```cpp
890:     if (gemm_workspace_.Computed_SFD->type() != library::NumericTypeID::kVoid) {
891:       if (options.initialization.provider == library::Provider::kReferenceHost) {
892:         gemm_workspace_.Reference_SFD->fill_host(0);
893:         gemm_workspace_.Computed_SFD->fill_host(0);
894:       }
895:       else {
896:         gemm_workspace_.Reference_SFD->fill_device(0);
897:         gemm_workspace_.Computed_SFD->fill_device(0);
898:       }
899:     }
```
- **EN:** Declares or updates local/member state such as `provider`.
- **CN:** 声明或更新局部/成员状态，例如 `provider`。

### Lines 902-902
```cpp
902:     // NOTE: the leading non-batch strides are duplicated here for 3.0 API kernels
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 903-916
```cpp
903:     gemm_workspace_.arguments.problem_size = {int(problem_.m), int(problem_.n), int(problem_.k)};
904:     gemm_workspace_.arguments.cluster_shape = {int(problem_.cluster_m), int(problem_.cluster_n), int(problem_.cluster_k)};
905:     gemm_workspace_.arguments.cluster_shape_fallback = {int(problem_.cluster_m_fallback), int(problem_.cluster_n_fallback), int(problem_.cluster_k_fallback)};
906:     gemm_workspace_.arguments.split_k_slices = problem_.split_k_slices;
907:     gemm_workspace_.arguments.batch_count = problem_.batch_count;
908:     gemm_workspace_.arguments.lda = problem_.lda;
909:     gemm_workspace_.arguments.ldb = problem_.ldb;
910:     gemm_workspace_.arguments.ldc = problem_.ldc;
911:     gemm_workspace_.arguments.ldd = problem_.ldc;
912:     gemm_workspace_.arguments.batch_stride_A = gemm_workspace_.A->batch_stride();
913:     gemm_workspace_.arguments.batch_stride_B = gemm_workspace_.B->batch_stride();
914:     gemm_workspace_.arguments.batch_stride_C = gemm_workspace_.C->batch_stride();
915:     gemm_workspace_.arguments.batch_stride_D = gemm_workspace_.Computed->batch_stride();
916:     gemm_workspace_.arguments.use_pdl = problem_.use_pdl;
```
- **EN:** Declares or updates local/member state such as `problem_size`, `cluster_shape`, `cluster_shape_fallback`, `split_k_slices`.
- **CN:** 声明或更新局部/成员状态，例如 `problem_size`, `cluster_shape`, `cluster_shape_fallback`, `split_k_slices`。

### Lines 918-918
```cpp
918:     /* Query device SM count to pass onto the kernel as an argument, where needed */
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 919-920
```cpp
919:     gemm_workspace_.arguments.sm_count = options.device.get_sm_count(0);
920:   }
```
- **EN:** Computes a thread-local view or helper object from previously constructed copy metadata.
- **CN:** 基于先前构造的拷贝元数据，计算线程局部视图或辅助对象。

### Lines 922-924
```cpp
922:   //
923:   // Initialize the CUTLASS operation
924:   //
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 925-925
```cpp
925:   Status status = Status::kSuccess;
```
- **EN:** Declares or updates local/member state such as `status`, `kSuccess`.
- **CN:** 声明或更新局部/成员状态，例如 `status`, `kSuccess`。

### Lines 927-927
```cpp
927:   if (options.profiling.provider_enabled(library::Provider::kCUTLASS)) {
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 929-931
```cpp
929:     if (options.execution_mode != ExecutionMode::kDryRun) {
930:       uint64_t workspace_size = underlying_operation->get_host_workspace_size(&gemm_workspace_.configuration);
931:       gemm_workspace_.host_workspace.resize(workspace_size, 0);
```
- **EN:** Declares or updates local/member state such as `workspace_size`.
- **CN:** 声明或更新局部/成员状态，例如 `workspace_size`。

### Lines 933-935
```cpp
933:       workspace_size = underlying_operation->get_device_workspace_size(&gemm_workspace_.configuration,
934:                                                             &gemm_workspace_.arguments);
935:       gemm_workspace_.device_workspace.reset(library::NumericTypeID::kU8, workspace_size);
```
- **EN:** Computes a thread-local view or helper object from previously constructed copy metadata.
- **CN:** 基于先前构造的拷贝元数据，计算线程局部视图或辅助对象。

### Lines 937-943
```cpp
937:       status = underlying_operation->initialize(
938:         &gemm_workspace_.configuration,
939:         gemm_workspace_.host_workspace.data(),
940:         gemm_workspace_.device_workspace.data());
941:       if (status != Status::kSuccess) {
942:         return status;
943:       }
```
- **EN:** Initializes or registers block-scaled GEMM components for later lookup or execution.
- **CN:** 初始化或注册块缩放 GEMM组件，以便后续查找或执行。

### Lines 945-947
```cpp
945:       if (problem_.split_k_mode == library::SplitKMode::kParallel) {
946:         workspace_size = reduction_op_->get_host_workspace_size(&gemm_workspace_.reduction_configuration);
947:         gemm_workspace_.reduction_host_workspace.resize(workspace_size, 0);
```
- **EN:** Declares or updates local/member state such as `split_k_mode`, `workspace_size`.
- **CN:** 声明或更新局部/成员状态，例如 `split_k_mode`, `workspace_size`。

### Lines 949-952
```cpp
949:         status = reduction_op_->initialize(
950:           &gemm_workspace_.reduction_configuration,
951:           gemm_workspace_.reduction_host_workspace.data(),
952:           nullptr);
```
- **EN:** Initializes or registers block-scaled GEMM components for later lookup or execution.
- **CN:** 初始化或注册块缩放 GEMM组件，以便后续查找或执行。

### Lines 954-958
```cpp
954:         if (status != Status::kSuccess) {
955:           return status;
956:         }
957:       }
958:     }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 960-962
```cpp
960:     //
961:     // If CUTLASS is enabled, generate a result for it
962:     //
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 963-966
```cpp
963:     results_.push_back(model_result_);
964:     results_.back().provider = library::Provider::kCUTLASS;
965:     results_.back().op_kind = library::OperationKind::kGemm;
966:     results_.back().disposition = Disposition::kNotRun;
```
- **EN:** Implements `push_back` and coordinates helper calls such as `back`.
- **CN:** 实现 `push_back`，并协调调用 `back` 等辅助逻辑。

### Lines 968-973
```cpp
968:     for (auto provider : verification_providers_) {
969:       results_.back().verification_map[provider] = Disposition::kNotRun;
970:     }
971:   }
972:   return status;
973: }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 975-975
```cpp
975: /////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 977-977
```cpp
977: /// Verifies CUTLASS against references
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 978-984
```cpp
978: bool BlockScaledGemmOperationProfiler::verify_cutlass(
979:   Options const &options,
980:   PerformanceReport &report,
981:   DeviceContext &device_context,
982:   library::Operation const *operation,
983:   ProblemSpace const &problem_space,
984:   ProblemSpace::Problem const &problem) {
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 986-988
```cpp
986:   if (!options.profiling.provider_enabled(library::Provider::kCUTLASS)) {
987:     return true;
988:   }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 990-992
```cpp
990:   if (options.execution_mode == ExecutionMode::kDryRun) {
991:     return true;
992:   }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 994-994
```cpp
994:   // Initialize structure containing GEMM arguments
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 995-1008
```cpp
995:   gemm_workspace_.arguments.A = gemm_workspace_.A->data();
996:   gemm_workspace_.arguments.B = gemm_workspace_.B->data();
997:   gemm_workspace_.arguments.SFA = gemm_workspace_.SFA->data();
998:   gemm_workspace_.arguments.SFB = gemm_workspace_.SFB->data();
999:   gemm_workspace_.arguments.C = gemm_workspace_.C->data();
1000:   gemm_workspace_.arguments.D = gemm_workspace_.Computed->data();
1001:   gemm_workspace_.arguments.SFD = gemm_workspace_.Computed_SFD->data();
1002:   gemm_workspace_.arguments.alpha = problem_.alpha.data();
1003:   gemm_workspace_.arguments.beta = problem_.beta.data();
1004:   gemm_workspace_.arguments.norm_constant = gemm_workspace_.Norm_constant->data();
1005:   gemm_workspace_.arguments.pointer_mode = library::ScalarPointerMode::kHost;
1006:   gemm_workspace_.arguments.batch_stride_A = gemm_workspace_.A->batch_stride();
1007:   gemm_workspace_.arguments.batch_stride_B = gemm_workspace_.B->batch_stride();
1008:   gemm_workspace_.arguments.batch_stride_C = gemm_workspace_.C->batch_stride();
```
- **EN:** Implements `data` and coordinates helper calls such as `batch_stride`.
- **CN:** 实现 `data`，并协调调用 `batch_stride` 等辅助逻辑。

### Lines 1009-1009
```cpp
1009:   gemm_workspace_.arguments.batch_stride_D = gemm_workspace_.Computed->batch_stride();
```
- **EN:** Implements `batch_stride` for this file's main component.
- **CN:** 为该文件的核心组件实现 `batch_stride`。

### Lines 1011-1014
```cpp
1011:   if (problem_.split_k_mode == library::SplitKMode::kParallel) {
1012:     gemm_workspace_.arguments.D                       = gemm_workspace_.device_workspace.data();
1013:     gemm_workspace_.arguments.alpha                   = problem_.alpha_one.data();
1014:     gemm_workspace_.arguments.beta                    = problem_.beta_zero.data();
```
- **EN:** Declares or updates local/member state such as `split_k_mode`, `D`, `alpha`, `beta`.
- **CN:** 声明或更新局部/成员状态，例如 `split_k_mode`, `D`, `alpha`, `beta`。

### Lines 1016-1022
```cpp
1016:     gemm_workspace_.reduction_arguments.workspace     = gemm_workspace_.device_workspace.data();
1017:     gemm_workspace_.reduction_arguments.source        = gemm_workspace_.C->data();
1018:     gemm_workspace_.reduction_arguments.destination   = gemm_workspace_.Computed->data();
1019:     gemm_workspace_.reduction_arguments.alpha         = problem_.alpha.data();
1020:     gemm_workspace_.reduction_arguments.beta          = problem_.beta.data();
1021:     gemm_workspace_.reduction_arguments.pointer_mode  = library::ScalarPointerMode::kHost;
1022:   }
```
- **EN:** Implements `data` for this file's main component.
- **CN:** 为该文件的核心组件实现 `data`。

### Lines 1024-1026
```cpp
1024:   //
1025:   // Run the CUTLASS operation
1026:   //
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1028-1028
```cpp
1028:   // initialize gemm underlying operation to handle parallel reduction
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1029-1029
```cpp
1029:   library::Operation const * underlying_operation = operation;
```
- **EN:** Declares or updates local/member state such as `underlying_operation`, `operation`.
- **CN:** 声明或更新局部/成员状态，例如 `underlying_operation`, `operation`。

### Lines 1031-1036
```cpp
1031:   if (problem_.split_k_mode == library::SplitKMode::kParallel) {
1032:     if (!(underlying_operation = library::find_gemm_operation_for_parallel_reduction(operation))) {
1033:       results_.back().disposition = Disposition::kFailed;
1034:       return false;
1035:     }
1036:   }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 1038-1042
```cpp
1038:   results_.back().status = underlying_operation->run(
1039:     &gemm_workspace_.arguments,
1040:     gemm_workspace_.host_workspace.data(),
1041:     gemm_workspace_.device_workspace.data(),
1042:     nullptr);
```
- **EN:** Declares or updates local/member state such as `status`.
- **CN:** 声明或更新局部/成员状态，例如 `status`。

### Lines 1044-1047
```cpp
1044:   if (results_.back().status != Status::kSuccess) {
1045:     results_.back().disposition = Disposition::kFailed;
1046:     return false;
1047:   }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 1049-1049
```cpp
1049:   // Run parallel reduction kernel for parallel split_k_mode
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1050-1055
```cpp
1050:   if (problem_.split_k_mode == library::SplitKMode::kParallel) {
1051:     results_.back().status = reduction_op_->run(
1052:       &gemm_workspace_.reduction_arguments,
1053:       gemm_workspace_.reduction_host_workspace.data(),
1054:       nullptr,
1055:       nullptr);
```
- **EN:** Declares or updates local/member state such as `split_k_mode`, `status`.
- **CN:** 声明或更新局部/成员状态，例如 `split_k_mode`, `status`。

### Lines 1057-1061
```cpp
1057:     if (results_.back().status != Status::kSuccess) {
1058:       results_.back().disposition = Disposition::kFailed;
1059:       return false;
1060:     }
1061:   }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 1063-1067
```cpp
1063:   cudaError_t result = cudaDeviceSynchronize();
1064:   if (result != cudaSuccess) {
1065:     results_.back().disposition = Disposition::kFailed;
1066:     return false;
1067:   }
```
- **EN:** Implements `cudaDeviceSynchronize` and coordinates helper calls such as `back`.
- **CN:** 实现 `cudaDeviceSynchronize`，并协调调用 `back` 等辅助逻辑。

### Lines 1069-1069
```cpp
1069:   // CUTLASS op ran the but not yet verified against any verification provider
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1070-1070
```cpp
1070:   results_.back().disposition = Disposition::kNotVerified;
```
- **EN:** Declares or updates local/member state such as `disposition`, `kNotVerified`.
- **CN:** 声明或更新局部/成员状态，例如 `disposition`, `kNotVerified`。

### Lines 1072-1074
```cpp
1072:   //
1073:   // Run verification providers
1074:   //
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1076-1076
```cpp
1076:   if (options.verification.enabled) {
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 1078-1078
```cpp
1078: #if CUTLASS_ENABLE_CUBLAS
```
- **EN:** Conditional-compilation or macro block keyed on `CUTLASS_ENABLE_CUBLAS`.
- **CN:** 以 `CUTLASS_ENABLE_CUBLAS` 为条件的条件编译或宏定义代码块。

### Lines 1079-1079
```cpp
1079:     if (options.verification.provider_enabled(library::Provider::kCUBLAS)) {
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 1080-1080
```cpp
1080:       // set verification map for cublas to not supported
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1081-1082
```cpp
1081:       results_.back().verification_map[library::Provider::kCUBLAS] = Disposition::kNotSupported;
1082:     }
```
- **EN:** Declares or updates local/member state such as `kNotSupported`.
- **CN:** 声明或更新局部/成员状态，例如 `kNotSupported`。

### Lines 1083-1083
```cpp
1083: #endif // #if CUTLASS_ENABLE_CUBLAS
```
- **EN:** Conditional-compilation or macro block keyed on `CUTLASS_ENABLE_CUBLAS`.
- **CN:** 以 `CUTLASS_ENABLE_CUBLAS` 为条件的条件编译或宏定义代码块。

### Lines 1086-1087
```cpp
1086:     cutlass::library::RuntimeDatatype runtime_datatype_a = gemm_workspace_.arguments.runtime_input_datatype_a;
1087:     cutlass::library::RuntimeDatatype runtime_datatype_b = gemm_workspace_.arguments.runtime_input_datatype_b;
```
- **EN:** Declares or updates local/member state such as `runtime_datatype_a`, `runtime_input_datatype_a`, `runtime_datatype_b`, `runtime_input_datatype_b`.
- **CN:** 声明或更新局部/成员状态，例如 `runtime_datatype_a`, `runtime_input_datatype_a`, `runtime_datatype_b`, `runtime_input_datatype_b`。

### Lines 1089-1090
```cpp
1089:     bool is_runtime_datatype_a = runtime_datatype_a != cutlass::library::RuntimeDatatype::kStatic;
1090:     bool is_runtime_datatype_b = runtime_datatype_b != cutlass::library::RuntimeDatatype::kStatic;
```
- **EN:** Declares or updates local/member state such as `is_runtime_datatype_a`, `kStatic`, `is_runtime_datatype_b`.
- **CN:** 声明或更新局部/成员状态，例如 `is_runtime_datatype_a`, `kStatic`, `is_runtime_datatype_b`。

### Lines 1092-1092
```cpp
1092:     assert(is_runtime_datatype_a == is_runtime_datatype_b && "runtime datatype should be both dynamic or static.");
```
- **EN:** Declares or updates local/member state such as `is_runtime_datatype_a`.
- **CN:** 声明或更新局部/成员状态，例如 `is_runtime_datatype_a`。

### Lines 1094-1095
```cpp
1094:     library::OperationDescription const &desc = operation->description();
1095:     auto &gemm_desc = static_cast<library::BlockScaledGemmDescription const &>(desc);
```
- **EN:** Implements `description` for this file's main component.
- **CN:** 为该文件的核心组件实现 `description`。

### Lines 1097-1098
```cpp
1097:     cutlass::library::NumericTypeID element_A = gemm_desc.A.element;
1098:     cutlass::library::NumericTypeID element_B = gemm_desc.B.element;
```
- **EN:** Declares or updates local/member state such as `element_A`, `element`, `element_B`.
- **CN:** 声明或更新局部/成员状态，例如 `element_A`, `element`, `element_B`。

### Lines 1100-1102
```cpp
1100:     if (is_runtime_datatype_a) {
1101:       element_A = cutlass::library::dynamic_datatype_to_id(runtime_datatype_a);
1102:     }
```
- **EN:** Declares or updates local/member state such as `element_A`.
- **CN:** 声明或更新局部/成员状态，例如 `element_A`。

### Lines 1104-1106
```cpp
1104:     if (is_runtime_datatype_b) {
1105:       element_B = cutlass::library::dynamic_datatype_to_id(runtime_datatype_b);
1106:     }
```
- **EN:** Declares or updates local/member state such as `element_B`.
- **CN:** 声明或更新局部/成员状态，例如 `element_B`。

### Lines 1109-1109
```cpp
1109:     bool verification_status = verify_with_reference_(options, report, device_context, operation, problem_space, problem, element_A, element_B);
```
- **EN:** Implements `verify_with_reference_` for this file's main component.
- **CN:** 为该文件的核心组件实现 `verify_with_reference_`。

### Lines 1111-1112
```cpp
1111:     // Update disposition to worst case verification outcome among all
1112:     // verification providers which are supported
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1113-1122
```cpp
1113:     bool is_any_verification_run_passed = false;
1114:     for (auto &m : results_.back().verification_map) {
1115:       if (m.second == Disposition::kFailed || m.second == Disposition::kIncorrect) {
1116:         results_.back().disposition = m.second;
1117:         return true;
1118:       }
1119:       if (!is_any_verification_run_passed && m.second == Disposition::kPassed) {
1120:         is_any_verification_run_passed = true;
1121:       }
1122:     }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 1124-1127
```cpp
1124:     if (is_any_verification_run_passed) {
1125:       results_.back().disposition = Disposition::kPassed;
1126:     }
1127:   }
```
- **EN:** Declares or updates local/member state such as `disposition`, `kPassed`.
- **CN:** 声明或更新局部/成员状态，例如 `disposition`, `kPassed`。

### Lines 1129-1129
```cpp
1129:   // if verification.required is set, then return success iff at least one ref-check was run
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1130-1134
```cpp
1130:   if (options.verification.required) {
1131:     bool did_any_verification_run = false;
1132:     for (auto provider : options.verification.providers) {
1133:       did_any_verification_run |= (Disposition::kNotRun != results_.back().verification_map[provider]);
1134:     }
```
- **EN:** Declares or updates local/member state such as `did_any_verification_run`, `false`.
- **CN:** 声明或更新局部/成员状态，例如 `did_any_verification_run`, `false`。

### Lines 1136-1140
```cpp
1136:     if (not did_any_verification_run) {
1137:       results_.back().status = Status::kErrorNotSupported;
1138:       return false;
1139:     }
1140:   }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 1142-1142
```cpp
1142:   // Return true means continue profiling
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1143-1144
```cpp
1143:   return true;
1144: }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 1146-1146
```cpp
1146: ///////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1148-1148
```cpp
1148: /// Verifies CUTLASS against references
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1149-1155
```cpp
1149: bool BlockScaledGemmOperationProfiler::verify_with_cublas_(
1150:   Options const &options,
1151:   PerformanceReport &report,
1152:   DeviceContext &device_context,
1153:   library::Operation const *operation,
1154:   ProblemSpace const &problem_space,
1155:   ProblemSpace::Problem const &problem) {
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 1157-1157
```cpp
1157: #if CUTLASS_ENABLE_CUBLAS
```
- **EN:** Conditional-compilation or macro block keyed on `CUTLASS_ENABLE_CUBLAS`.
- **CN:** 以 `CUTLASS_ENABLE_CUBLAS` 为条件的条件编译或宏定义代码块。

### Lines 1158-1158
```cpp
1158:   std::cerr << "cuBLAS is not supported" << std::endl;
```
- **EN:** Declares or updates local/member state such as `endl`.
- **CN:** 声明或更新局部/成员状态，例如 `endl`。

### Lines 1159-1159
```cpp
1159: #endif
```
- **EN:** Preprocessor directives configure compilation behavior for this source file.
- **CN:** 预处理指令用于配置该源文件的编译行为。

### Lines 1161-1161
```cpp
1161:   // Return true means continue profiling
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1162-1163
```cpp
1162:   return true;
1163: }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 1165-1165
```cpp
1165: /////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1167-1167
```cpp
1167: /// Verifies CUTLASS against host and device references
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1168-1176
```cpp
1168: bool BlockScaledGemmOperationProfiler::verify_with_reference_(
1169:   Options const &options,
1170:   PerformanceReport &report,
1171:   DeviceContext &device_context,
1172:   library::Operation const *operation,
1173:   ProblemSpace const &problem_space,
1174:   ProblemSpace::Problem const &problem,
1175:   cutlass::library::NumericTypeID element_A,
1176:   cutlass::library::NumericTypeID element_B) {
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 1178-1178
```cpp
1178:   /// Verifies CUTLASS against host reference
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1180-1182
```cpp
1180:   //
1181:   // Find host reference operation using conv2d functional description key
1182:   //
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1183-1183
```cpp
1183:   library::OperationDescription const &desc = operation->description();
```
- **EN:** Implements `description` for this file's main component.
- **CN:** 为该文件的核心组件实现 `description`。

### Lines 1185-1185
```cpp
1185:   auto &gemm_desc = static_cast<library::BlockScaledGemmDescription const &>(desc);
```
- **EN:** Declares or updates local/member state such as `gemm_desc`.
- **CN:** 声明或更新局部/成员状态，例如 `gemm_desc`。

### Lines 1187-1200
```cpp
1187:   library::BlockScaledGemmFunctionalKey blockScaledGemm_key(
1188:     library::Provider::kReferenceHost,
1189:     gemm_desc.gemm_kind,
1190:     gemm_desc.kind,
1191:     gemm_desc.tile_description.math_instruction.element_accumulator,
1192:     gemm_desc.element_epilogue,
1193:     element_A,
1194:     gemm_desc.A.layout,
1195:     gemm_desc.SFA.element,
1196:     element_B,
1197:     gemm_desc.B.layout,
1198:     gemm_desc.SFB.element,
1199:     gemm_desc.C.element,
1200:     gemm_desc.C.layout,
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 1201-1207
```cpp
1201:     gemm_desc.D.element,
1202:     gemm_desc.D.layout,
1203:     gemm_desc.SFD.element,
1204:     gemm_desc.SFD.layout,
1205:     gemm_desc.SFVecSize
1206:     , gemm_desc.EpilogueSFVecSize
1207:   );
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 1209-1209
```cpp
1209:   auto operators_it = library::Singleton::get().operation_table.block_scaled_gemm_operations.find(blockScaledGemm_key);
```
- **EN:** Implements `get` and coordinates helper calls such as `find`.
- **CN:** 实现 `get`，并协调调用 `find` 等辅助逻辑。

### Lines 1211-1213
```cpp
1211:   if (operators_it == library::Singleton::get().operation_table.block_scaled_gemm_operations.end()) {
1212:     return true;
1213:   }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 1215-1217
```cpp
1215:   if (operators_it->second.empty()) {
1216:     return true;
1217:   }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 1219-1219
```cpp
1219:   // Not use preference to filter the reference kernel.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1220-1220
```cpp
1220:   auto cc_it = operators_it->second.begin();
```
- **EN:** Provides a small helper for managing container lifetime or iteration.
- **CN:** 提供用于管理容器生命周期或迭代的小型辅助函数。

### Lines 1222-1226
```cpp
1222:   if(cc_it == operators_it->second.end()) {
1223:     std::cout<< "not find any reference kernel" << std::endl;
1224:     results_.back().verification_map[library::Provider::kReferenceHost] = Disposition::kNotRun;
1225:     return true;
1226:   }
```
- **EN:** Emits debug or diagnostic output so intermediate state can be inspected.
- **CN:** 输出调试或诊断信息，便于检查中间状态。

### Lines 1228-1228
```cpp
1228:   // host reference has only one instances in BlockScaledOperationVectorMap
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1229-1229
```cpp
1229:   library::Operation const *reference_op = cc_it->second[0];
```
- **EN:** Declares or updates local/member state such as `reference_op`.
- **CN:** 声明或更新局部/成员状态，例如 `reference_op`。

### Lines 1231-1232
```cpp
1231:   // To support the host-side reference, conditionally allocate and
1232:   // copy tensors to host memory.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1233-1240
```cpp
1233:   std::vector<uint8_t> host_data_A;
1234:   std::vector<uint8_t> host_data_SFA;
1235:   std::vector<uint8_t> host_data_B;
1236:   std::vector<uint8_t> host_data_SFB;
1237:   std::vector<uint8_t> host_data_C;
1238:   std::vector<uint8_t> host_data_D;
1239:   std::vector<uint8_t> host_data_SFD;
1240:   std::vector<uint8_t> host_data_Norm_constant;
```
- **EN:** Declares or updates local/member state such as `host_data_A`, `host_data_SFA`, `host_data_B`, `host_data_SFB`.
- **CN:** 声明或更新局部/成员状态，例如 `host_data_A`, `host_data_SFA`, `host_data_B`, `host_data_SFB`。

### Lines 1242-1244
```cpp
1242:   //
1243:   // Copy input tensors A, B, and C from device to host buffers
1244:   //
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1246-1248
```cpp
1246:   host_data_A.resize(gemm_workspace_.A->bytes());
1247:   void * ptr_A = host_data_A.data();
1248:   gemm_workspace_.A->copy_to_host(ptr_A);
```
- **EN:** Implements `resize` and coordinates helper calls such as `bytes`, `data`, `copy_to_host`.
- **CN:** 实现 `resize`，并协调调用 `bytes`, `data`, `copy_to_host` 等辅助逻辑。

### Lines 1250-1252
```cpp
1250:   host_data_SFA.resize(gemm_workspace_.SFA->bytes());
1251:   void * ptr_SFA = host_data_SFA.data();
1252:   gemm_workspace_.SFA->copy_to_host(ptr_SFA);
```
- **EN:** Implements `resize` and coordinates helper calls such as `bytes`, `data`, `copy_to_host`.
- **CN:** 实现 `resize`，并协调调用 `bytes`, `data`, `copy_to_host` 等辅助逻辑。

### Lines 1254-1256
```cpp
1254:   host_data_B.resize(gemm_workspace_.B->bytes());
1255:   void * ptr_B = host_data_B.data();
1256:   gemm_workspace_.B->copy_to_host(ptr_B);
```
- **EN:** Implements `resize` and coordinates helper calls such as `bytes`, `data`, `copy_to_host`.
- **CN:** 实现 `resize`，并协调调用 `bytes`, `data`, `copy_to_host` 等辅助逻辑。

### Lines 1258-1260
```cpp
1258:   host_data_SFB.resize(gemm_workspace_.SFB->bytes());
1259:   void * ptr_SFB = host_data_SFB.data();
1260:   gemm_workspace_.SFB->copy_to_host(ptr_SFB);
```
- **EN:** Implements `resize` and coordinates helper calls such as `bytes`, `data`, `copy_to_host`.
- **CN:** 实现 `resize`，并协调调用 `bytes`, `data`, `copy_to_host` 等辅助逻辑。

### Lines 1262-1264
```cpp
1262:   host_data_C.resize(gemm_workspace_.C->bytes());
1263:   void * ptr_C = host_data_C.data();
1264:   gemm_workspace_.C->copy_to_host(ptr_C);
```
- **EN:** Implements `resize` and coordinates helper calls such as `bytes`, `data`, `copy_to_host`.
- **CN:** 实现 `resize`，并协调调用 `bytes`, `data`, `copy_to_host` 等辅助逻辑。

### Lines 1266-1268
```cpp
1266:   host_data_Norm_constant.resize(gemm_workspace_.Norm_constant->bytes());
1267:   void * ptr_Norm_constant = host_data_Norm_constant.data();
1268:   gemm_workspace_.Norm_constant->copy_to_host(ptr_Norm_constant);
```
- **EN:** Implements `resize` and coordinates helper calls such as `bytes`, `data`, `copy_to_host`.
- **CN:** 实现 `resize`，并协调调用 `bytes`, `data`, `copy_to_host` 等辅助逻辑。

### Lines 1270-1271
```cpp
1270:   host_data_D.resize(gemm_workspace_.Reference->bytes());
1271:   void * ptr_D = host_data_D.data();
```
- **EN:** Implements `resize` and coordinates helper calls such as `bytes`, `data`.
- **CN:** 实现 `resize`，并协调调用 `bytes`, `data` 等辅助逻辑。

### Lines 1273-1274
```cpp
1273:   host_data_SFD.resize(gemm_workspace_.Reference_SFD->bytes());
1274:   void * ptr_SFD = host_data_SFD.data();
```
- **EN:** Implements `resize` and coordinates helper calls such as `bytes`, `data`.
- **CN:** 实现 `resize`，并协调调用 `bytes`, `data` 等辅助逻辑。

### Lines 1276-1276
```cpp
1276:   /// Set reference kernel Arguments
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1278-1291
```cpp
1278:   library::BlockScaledGemmArguments arguments {
1279:     {int(problem_.m), int(problem_.n), int(problem_.k)},
1280:     {int(problem_.cluster_m), int(problem_.cluster_n), int(problem_.cluster_k)},
1281:     {int(problem_.cluster_m_fallback), int(problem_.cluster_n_fallback), int(problem_.cluster_k_fallback)},
1282:     gemm_workspace_.configuration.batch_count,
1283:     ptr_A,
1284:     ptr_B,
1285:     ptr_SFA,
1286:     ptr_SFB,
1287:     ptr_C,
1288:     ptr_D,
1289:     ptr_SFD,
1290:     problem_.alpha.data(),
1291:     problem_.beta.data(),
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 1292-1302
```cpp
1292:     library::ScalarPointerMode::kHost,
1293:     int(gemm_workspace_.configuration.lda),
1294:     int(gemm_workspace_.configuration.ldb),
1295:     int(gemm_workspace_.configuration.ldc),
1296:     int(gemm_workspace_.configuration.ldd),
1297:     gemm_workspace_.A->batch_stride(),
1298:     gemm_workspace_.B->batch_stride(),
1299:     gemm_workspace_.C->batch_stride(),
1300:     gemm_workspace_.Reference->batch_stride()
1301:     , ptr_Norm_constant
1302:   };
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 1304-1304
```cpp
1304:   // Query host work space size
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1305-1305
```cpp
1305:   uint64_t host_workspace_size_needed = reference_op->get_host_workspace_size(&gemm_workspace_.configuration);
```
- **EN:** Computes a thread-local view or helper object from previously constructed copy metadata.
- **CN:** 基于先前构造的拷贝元数据，计算线程局部视图或辅助对象。

### Lines 1307-1307
```cpp
1307:   std::vector<char> host_workspace(host_workspace_size_needed);
```
- **EN:** Implements `host_workspace` for this file's main component.
- **CN:** 为该文件的核心组件实现 `host_workspace`。

### Lines 1309-1309
```cpp
1309:   // Query device workspace size
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1310-1310
```cpp
1310:   uint64_t device_workspace_size_needed = reference_op->get_device_workspace_size(&gemm_workspace_.configuration);
```
- **EN:** Computes a thread-local view or helper object from previously constructed copy metadata.
- **CN:** 基于先前构造的拷贝元数据，计算线程局部视图或辅助对象。

### Lines 1311-1311
```cpp
1311:   // Initialize host and device workspaces
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1312-1315
```cpp
1312:   Status status = reference_op->initialize(
1313:     &gemm_workspace_.configuration,
1314:     host_workspace.data()
1315:   );
```
- **EN:** Initializes or registers block-scaled GEMM components for later lookup or execution.
- **CN:** 初始化或注册块缩放 GEMM组件，以便后续查找或执行。

### Lines 1317-1320
```cpp
1317:   if (status != cutlass::Status::kSuccess) {
1318:     results_.back().verification_map[library::Provider::kReferenceHost] = Disposition::kNotRun;
1319:     return true;
1320:   }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 1322-1322
```cpp
1322:   // Run the operator
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1323-1323
```cpp
1323:   status = reference_op->run(&arguments, host_workspace.data());
```
- **EN:** Implements `run` and coordinates helper calls such as `data`.
- **CN:** 实现 `run`，并协调调用 `data` 等辅助逻辑。

### Lines 1325-1325
```cpp
1325:   results_.back().status = status;
```
- **EN:** Declares or updates local/member state such as `status`.
- **CN:** 声明或更新局部/成员状态，例如 `status`。

### Lines 1327-1328
```cpp
1327:   gemm_workspace_.Reference->copy_from_host(ptr_D);
1328:   gemm_workspace_.Reference_SFD->copy_from_host(ptr_SFD);
```
- **EN:** Implements `copy_from_host` for this file's main component.
- **CN:** 为该文件的核心组件实现 `copy_from_host`。

### Lines 1330-1332
```cpp
1330:   //
1331:   // Verify results
1332:   //
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1333-1338
```cpp
1333:   auto resultD = compare_tensors(
1334:     options,
1335:     *gemm_workspace_.Computed,
1336:     *gemm_workspace_.Reference,
1337:     gemm_workspace_.Computed->batch_stride()
1338:   );
```
- **EN:** Declares or updates local/member state such as `resultD`.
- **CN:** 声明或更新局部/成员状态，例如 `resultD`。

### Lines 1340-1348
```cpp
1340:   auto resultSFD = Disposition::kPassed;
1341:   if (gemm_desc.SFD.element != library::NumericTypeID::kVoid) {
1342:     resultSFD = compare_tensors(
1343:       options,
1344:       *gemm_workspace_.Computed_SFD,
1345:       *gemm_workspace_.Reference_SFD,
1346:       gemm_workspace_.Computed_SFD->batch_stride()
1347:     );
1348:   }
```
- **EN:** Declares or updates local/member state such as `resultSFD`, `kPassed`.
- **CN:** 声明或更新局部/成员状态，例如 `resultSFD`, `kPassed`。

### Lines 1350-1350
```cpp
1350:   results_.back().verification_map[library::Provider::kReferenceHost] = resultD;
```
- **EN:** Declares or updates local/member state such as `resultD`.
- **CN:** 声明或更新局部/成员状态，例如 `resultD`。

### Lines 1352-1354
```cpp
1352:   if (resultSFD != Disposition::kPassed) {
1353:       results_.back().verification_map[library::Provider::kReferenceHost] = resultSFD;
1354:   }
```
- **EN:** Declares or updates local/member state such as `resultSFD`.
- **CN:** 声明或更新局部/成员状态，例如 `resultSFD`。

### Lines 1357-1357
```cpp
1357:   // Save workspace if incorrect
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1358-1366
```cpp
1358:   if (options.verification.save_workspace == SaveWorkspace::kIncorrect &&
1359:     results_.back().verification_map[library::Provider::kReferenceHost] == Disposition::kIncorrect) {
1360:     save_workspace(
1361:       device_context,
1362:       options,
1363:       gemm_desc,
1364:       library::Provider::kCUTLASS,
1365:       library::Provider::kReferenceHost);
1366:   }
```
- **EN:** Declares or updates local/member state such as `save_workspace`.
- **CN:** 声明或更新局部/成员状态，例如 `save_workspace`。

### Lines 1368-1368
```cpp
1368:   // Return true means continue profiling
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1369-1370
```cpp
1369:   return true;
1370: }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 1372-1372
```cpp
1372: /////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1374-1374
```cpp
1374: /// Measures performance results
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1375-1381
```cpp
1375: bool BlockScaledGemmOperationProfiler::profile(
1376:   Options const &options,
1377:   PerformanceReport &report,
1378:   DeviceContext &device_context,
1379:   library::Operation const *operation,
1380:   ProblemSpace const &problem_space,
1381:   ProblemSpace::Problem const &problem) {
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 1383-1383
```cpp
1383:   if (options.profiling.provider_enabled(library::Provider::kCUTLASS)) {
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 1385-1387
```cpp
1385:     if (options.profiling.enable_kernel_performance_search || options.profiling.enable_best_kernel_for_fixed_shape) {
1386:       library::BlockScaledGemmDescription const &operation_desc =
1387:         static_cast<library::BlockScaledGemmDescription const &>(operation->description());
```
- **EN:** Declares or updates local/member state such as `operation_desc`.
- **CN:** 声明或更新局部/成员状态，例如 `operation_desc`。

### Lines 1389-1390
```cpp
1389:       auto cluster_shape = operation_desc.tile_description.cluster_shape;
1390:       bool is_dynamic_cluster_enabled = cluster_shape.m() == 0 || cluster_shape.n() == 0 || cluster_shape.k() == 0;
```
- **EN:** Declares or updates local/member state such as `cluster_shape`, `is_dynamic_cluster_enabled`.
- **CN:** 声明或更新局部/成员状态，例如 `cluster_shape`, `is_dynamic_cluster_enabled`。

### Lines 1392-1392
```cpp
1392:       // Helper function wrapping up performance test with flexible parameters.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1393-1400
```cpp
1393:       auto initialize_and_profile = [&](
1394:         PerformanceResult const &result,
1395:         gemm::GemmCoord const &problem_shape,
1396:         std::array<int64_t, 3> const &leading_dim,
1397:         std::array<int64_t, 3> const &preferred_cluster,
1398:         std::array<int64_t, 3> const &fallback_cluster,
1399:         cutlass::library::RasterOrder const &raster_order,
1400:         int swizzle_size) -> std::optional<PerformanceResult> {
```
- **EN:** Declares or updates local/member state such as `initialize_and_profile`.
- **CN:** 声明或更新局部/成员状态，例如 `initialize_and_profile`。

### Lines 1402-1402
```cpp
1402:         // Initialize structure containing GEMM arguments
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1403-1415
```cpp
1403:         gemm_workspace_.arguments.A = gemm_workspace_.A->data();
1404:         gemm_workspace_.arguments.B = gemm_workspace_.B->data();
1405:         gemm_workspace_.arguments.SFA = gemm_workspace_.SFA->data();
1406:         gemm_workspace_.arguments.SFB = gemm_workspace_.SFB->data();
1407:         gemm_workspace_.arguments.C = gemm_workspace_.C->data();
1408:         gemm_workspace_.arguments.D = gemm_workspace_.Computed->data();
1409:         gemm_workspace_.arguments.alpha = problem_.alpha.data();
1410:         gemm_workspace_.arguments.beta = problem_.beta.data();
1411:         gemm_workspace_.arguments.pointer_mode = library::ScalarPointerMode::kHost;
1412:         gemm_workspace_.arguments.batch_stride_A = gemm_workspace_.A->batch_stride();
1413:         gemm_workspace_.arguments.batch_stride_B = gemm_workspace_.B->batch_stride();
1414:         gemm_workspace_.arguments.batch_stride_C = gemm_workspace_.C->batch_stride();
1415:         gemm_workspace_.arguments.batch_stride_D = gemm_workspace_.Computed->batch_stride();
```
- **EN:** Implements `data` and coordinates helper calls such as `batch_stride`.
- **CN:** 实现 `data`，并协调调用 `batch_stride` 等辅助逻辑。

### Lines 1417-1420
```cpp
1417:         if (problem_.split_k_mode == library::SplitKMode::kParallel) {
1418:           gemm_workspace_.arguments.D                       = gemm_workspace_.device_workspace.data();
1419:           gemm_workspace_.arguments.alpha                   = problem_.alpha_one.data();
1420:           gemm_workspace_.arguments.beta                    = problem_.beta_zero.data();
```
- **EN:** Declares or updates local/member state such as `split_k_mode`, `D`, `alpha`, `beta`.
- **CN:** 声明或更新局部/成员状态，例如 `split_k_mode`, `D`, `alpha`, `beta`。

### Lines 1422-1428
```cpp
1422:           gemm_workspace_.reduction_arguments.workspace     = gemm_workspace_.device_workspace.data();
1423:           gemm_workspace_.reduction_arguments.source        = gemm_workspace_.C->data();
1424:           gemm_workspace_.reduction_arguments.destination   = gemm_workspace_.Computed->data();
1425:           gemm_workspace_.reduction_arguments.alpha         = problem_.alpha.data();
1426:           gemm_workspace_.reduction_arguments.beta          = problem_.beta.data();
1427:           gemm_workspace_.reduction_arguments.pointer_mode  = library::ScalarPointerMode::kHost;
1428:         }
```
- **EN:** Implements `data` for this file's main component.
- **CN:** 为该文件的核心组件实现 `data`。

### Lines 1430-1430
```cpp
1430:         update_workspace_(gemm_workspace_, problem_shape, leading_dim, preferred_cluster, fallback_cluster, raster_order, swizzle_size, is_dynamic_cluster_enabled);
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 1432-1436
```cpp
1432:         const auto can_implement = operation->can_implement(&gemm_workspace_.configuration, &gemm_workspace_.arguments);
1433:         if (can_implement != Status::kSuccess) {
1434:           return std::nullopt;  // Return nullopt to indicate failure
1435:         }
1436:         library::Operation const* underlying_operation = operation;
```
- **EN:** Implements `can_implement` for this file's main component.
- **CN:** 为该文件的核心组件实现 `can_implement`。

### Lines 1438-1439
```cpp
1438:         uint64_t workspace_size = underlying_operation->get_host_workspace_size(&gemm_workspace_.configuration);
1439:         gemm_workspace_.host_workspace.resize(workspace_size, 0);
```
- **EN:** Computes a thread-local view or helper object from previously constructed copy metadata.
- **CN:** 基于先前构造的拷贝元数据，计算线程局部视图或辅助对象。

### Lines 1441-1442
```cpp
1441:         workspace_size = underlying_operation->get_device_workspace_size(&gemm_workspace_.configuration,
1442:                                                               &gemm_workspace_.arguments);
```
- **EN:** Computes a thread-local view or helper object from previously constructed copy metadata.
- **CN:** 基于先前构造的拷贝元数据，计算线程局部视图或辅助对象。

### Lines 1444-1444
```cpp
1444:         gemm_workspace_.device_workspace.reset(library::NumericTypeID::kU8, workspace_size);
```
- **EN:** Implements `reset` for this file's main component.
- **CN:** 为该文件的核心组件实现 `reset`。

### Lines 1446-1446
```cpp
1446:         Status status = Status::kSuccess;
```
- **EN:** Declares or updates local/member state such as `status`, `kSuccess`.
- **CN:** 声明或更新局部/成员状态，例如 `status`, `kSuccess`。

### Lines 1448-1451
```cpp
1448:         status = underlying_operation->initialize(
1449:           &gemm_workspace_.configuration,
1450:           gemm_workspace_.host_workspace.data(),
1451:           gemm_workspace_.device_workspace.data());
```
- **EN:** Initializes or registers block-scaled GEMM components for later lookup or execution.
- **CN:** 初始化或注册块缩放 GEMM组件，以便后续查找或执行。

### Lines 1453-1455
```cpp
1453:         if (status != Status::kSuccess) {
1454:           return std::nullopt;  // Return nullopt to indicate failure
1455:         }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 1457-1458
```cpp
1457:         PerformanceResult curr_result(result);
1458:         update_result_(curr_result, operation_desc, problem_space, problem_shape, raster_order, preferred_cluster, fallback_cluster, swizzle_size, is_dynamic_cluster_enabled);
```
- **EN:** Implements `curr_result` and coordinates helper calls such as `update_result_`.
- **CN:** 实现 `curr_result`，并协调调用 `update_result_` 等辅助逻辑。

### Lines 1460-1467
```cpp
1460:         curr_result.status = profile_cutlass_(
1461:           curr_result,
1462:           options,
1463:           operation,
1464:           &gemm_workspace_.arguments,
1465:           gemm_workspace_.host_workspace.data(),
1466:           gemm_workspace_.device_workspace.data()
1467:         );
```
- **EN:** Declares or updates local/member state such as `status`.
- **CN:** 声明或更新局部/成员状态，例如 `status`。

### Lines 1469-1470
```cpp
1469:         return curr_result;
1470:       };
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 1473-1473
```cpp
1473:       // Helper function to test validity of fallback cluster shapes and preferred cluster shapes.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1474-1481
```cpp
1474:       auto is_valid_dynamic_cluster_shape = [](const std::array<int64_t, 3>& preferred_cluster, const std::array<int64_t, 3>& fallback_cluster) {
1475:         for (size_t i = 0; i < 3; ++i) {
1476:           if (preferred_cluster[i] % fallback_cluster[i] != 0) {
1477:             return false;
1478:           }
1479:         }
1480:         return true;
1481:       };
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 1484-1484
```cpp
1484:       // Helper function to select the best performance number among a list.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1485-1495
```cpp
1485:       auto select_best_candidate = [&](std::vector<PerformanceResult> &candidates) {
1486:         assert(!candidates.empty() && "Candidates vector should not be empty");
1487:         auto best_iter = std::max_element(
1488:           candidates.begin(), candidates.end(),
1489:           [](PerformanceResult const &a, PerformanceResult const &b) {
1490:             return a.gflops_per_sec() < b.gflops_per_sec();
1491:           }
1492:         );
1493:         assert(best_iter != candidates.end() && "No candidate found despite non-empty candidates vector");
1494:         results_.push_back(std::move(*best_iter));
1495:       };
```
- **EN:** Implements `assert` and coordinates helper calls such as `empty`, `max_element`, `begin`.
- **CN:** 实现 `assert`，并协调调用 `empty`, `max_element`, `begin` 等辅助逻辑。

### Lines 1497-1499
```cpp
1497:       std::vector<PerformanceResult> candidates;
1498:       PerformanceResult result_base = results_.back();
1499:       results_.pop_back();
```
- **EN:** Implements `back` and coordinates helper calls such as `pop_back`.
- **CN:** 实现 `back`，并协调调用 `pop_back` 等辅助逻辑。

### Lines 1501-1502
```cpp
1501:       std::vector<std::array<int64_t, 3>> preferred_clusters;
1502:       std::vector<std::array<int64_t, 3>> fallback_clusters;
```
- **EN:** Declares or updates local/member state such as `preferred_clusters`, `fallback_clusters`.
- **CN:** 声明或更新局部/成员状态，例如 `preferred_clusters`, `fallback_clusters`。

### Lines 1504-1505
```cpp
1504:       // Only loop over built-in cluster shape lists for dynamic cluster kernels
1505:       // and for kernels that can leverage the dynamic cluster feature.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1506-1513
```cpp
1506:       if (is_dynamic_cluster_enabled) {
1507:         preferred_clusters = this->problem_.preferred_clusters;
1508:         fallback_clusters = this->problem_.fallback_clusters;
1509:       } 
1510:       else {
1511:         preferred_clusters = {{int(problem_.cluster_m), int(problem_.cluster_n), int(problem_.cluster_k)}};
1512:         fallback_clusters = {{int(problem_.cluster_m_fallback), int(problem_.cluster_n_fallback), int(problem_.cluster_k_fallback)}};
1513:       }
```
- **EN:** Declares or updates local/member state such as `preferred_clusters`, `fallback_clusters`.
- **CN:** 声明或更新局部/成员状态，例如 `preferred_clusters`, `fallback_clusters`。

### Lines 1515-1521
```cpp
1515:       for (auto preferred_cluster : preferred_clusters) {
1516:         for (auto fallback_cluster : fallback_clusters) {
1517:           if (is_dynamic_cluster_enabled && !is_valid_dynamic_cluster_shape(preferred_cluster, fallback_cluster)) {
1518:             continue;
1519:           }
1520:           for (auto swizzle_size : this->problem_.swizzle_sizes) {
1521:             for (auto raster_order : this->problem_.raster_orders) {
```
- **EN:** Declares or updates local/member state such as `continue`.
- **CN:** 声明或更新局部/成员状态，例如 `continue`。

### Lines 1522-1522
```cpp
1522:               // With the fixed shape option turned on, only a specific problem shape is tested.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1523-1526
```cpp
1523:               if (options.profiling.enable_best_kernel_for_fixed_shape) {
1524:                 this->problem_.problem_sizes = {{int(this->problem_.m), int(this->problem_.n), int(this->problem_.k)}};
1525:                 this->problem_.leading_dims = {{this->problem_.lda, this->problem_.ldb, this->problem_.ldc}};
1526:               }
```
- **EN:** Declares or updates local/member state such as `problem_sizes`, `leading_dims`.
- **CN:** 声明或更新局部/成员状态，例如 `problem_sizes`, `leading_dims`。

### Lines 1528-1531
```cpp
1528:               for (int i = 0; i < int(this->problem_.problem_sizes.size()); i++) {
1529:                 gemm::GemmCoord problem_shape = problem_.problem_sizes[i];
1530:                 std::array<int64_t, 3> leading_dim = problem_.leading_dims[i];
1531:                 auto result_opt = initialize_and_profile(result_base, problem_shape, leading_dim, preferred_cluster, fallback_cluster, raster_order, swizzle_size);
```
- **EN:** Declares or updates local/member state such as `i`, `problem_shape`, `leading_dim`, `result_opt`.
- **CN:** 声明或更新局部/成员状态，例如 `i`, `problem_shape`, `leading_dim`, `result_opt`。

### Lines 1533-1535
```cpp
1533:                 if (result_opt) {  // Only add valid results
1534:                   candidates.push_back(*result_opt);
1535:                 }
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 1537-1541
```cpp
1537:               }
1538:             }// for raster_order
1539:           }// for swizzle_size
1540:         }// for fallback_cluster
1541:       }// for swizzle_size
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 1543-1546
```cpp
1543:       if (candidates.empty()) {
1544:         return false;
1545:       }
1546:       select_best_candidate(candidates);
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 1548-1558
```cpp
1548:     }
1549:     else {
1550:       results_.back().status = profile_cutlass_(
1551:         results_.back(),
1552:         options,
1553:         operation,
1554:         &gemm_workspace_.arguments,
1555:         gemm_workspace_.host_workspace.data(),
1556:         gemm_workspace_.device_workspace.data()
1557:       );
1558:     }
```
- **EN:** Declares or updates local/member state such as `status`.
- **CN:** 声明或更新局部/成员状态，例如 `status`。

### Lines 1560-1562
```cpp
1560:   }
1561:   return true;
1562: }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 1564-1564
```cpp
1564: /////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1566-1566
```cpp
1566: /// Method to profile a CUTLASS Operation
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1567-1573
```cpp
1567: Status BlockScaledGemmOperationProfiler::profile_cutlass_(
1568:   PerformanceResult &result,
1569:   Options const &options,
1570:   library::Operation const *operation,
1571:   void *arguments,
1572:   void *host_workspace,
1573:   void *device_workspace) {
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 1575-1575
```cpp
1575:   // initialize gemm underlying operation to handle parallel reduction
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1576-1576
```cpp
1576:   library::Operation const * underlying_operation = operation;
```
- **EN:** Declares or updates local/member state such as `underlying_operation`, `operation`.
- **CN:** 声明或更新局部/成员状态，例如 `underlying_operation`, `operation`。

### Lines 1578-1582
```cpp
1578:   if (problem_.split_k_mode == library::SplitKMode::kParallel) {
1579:     if (!(underlying_operation = library::find_gemm_operation_for_parallel_reduction(operation))) {
1580:       return Status::kErrorNotSupported;
1581:     }
1582:   }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 1584-1584
```cpp
1584:   auto func = [&](cudaStream_t stream, int iteration) {
```
- **EN:** Declares or updates local/member state such as `func`.
- **CN:** 声明或更新局部/成员状态，例如 `func`。

### Lines 1585-1585
```cpp
1585:     // Iterate over copies of the problem in memory
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1586-1586
```cpp
1586:     int problem_idx = (iteration % gemm_workspace_.problem_count) * problem_.batch_count;
```
- **EN:** Declares or updates local/member state such as `problem_idx`, `batch_count`.
- **CN:** 声明或更新局部/成员状态，例如 `problem_idx`, `batch_count`。

### Lines 1588-1591
```cpp
1588:     gemm_workspace_.arguments.A = gemm_workspace_.A->batch_data(problem_idx);
1589:     gemm_workspace_.arguments.B = gemm_workspace_.B->batch_data(problem_idx);
1590:     gemm_workspace_.arguments.C = gemm_workspace_.C->batch_data(problem_idx);
1591:     gemm_workspace_.arguments.D = gemm_workspace_.Computed->batch_data(problem_idx);
```
- **EN:** Implements `batch_data` for this file's main component.
- **CN:** 为该文件的核心组件实现 `batch_data`。

### Lines 1593-1594
```cpp
1593:     if (problem_.split_k_mode == library::SplitKMode::kParallel) {
1594:       gemm_workspace_.arguments.D                     = gemm_workspace_.device_workspace.data();
```
- **EN:** Declares or updates local/member state such as `split_k_mode`, `D`.
- **CN:** 声明或更新局部/成员状态，例如 `split_k_mode`, `D`。

### Lines 1596-1599
```cpp
1596:       gemm_workspace_.reduction_arguments.workspace   = gemm_workspace_.device_workspace.data();
1597:       gemm_workspace_.reduction_arguments.source      = gemm_workspace_.C->batch_data(problem_idx);
1598:       gemm_workspace_.reduction_arguments.destination = gemm_workspace_.Computed->batch_data(problem_idx);
1599:     }
```
- **EN:** Implements `data` and coordinates helper calls such as `batch_data`.
- **CN:** 实现 `data`，并协调调用 `batch_data` 等辅助逻辑。

### Lines 1601-1605
```cpp
1601:     Status status = underlying_operation->run(
1602:       arguments,
1603:       host_workspace,
1604:       device_workspace,
1605:       stream);
```
- **EN:** Declares or updates local/member state such as `status`.
- **CN:** 声明或更新局部/成员状态，例如 `status`。

### Lines 1607-1609
```cpp
1607:     if (status != Status::kSuccess) {
1608:       return status;
1609:     }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 1611-1611
```cpp
1611:     // Run parallel reduction kernel for parallel split_k_mode
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1612-1617
```cpp
1612:     if (problem_.split_k_mode == library::SplitKMode::kParallel) {
1613:       status = reduction_op_->run(
1614:         &gemm_workspace_.reduction_arguments,
1615:         gemm_workspace_.reduction_host_workspace.data(),
1616:         nullptr,
1617:         stream);
```
- **EN:** Declares or updates local/member state such as `split_k_mode`, `status`.
- **CN:** 声明或更新局部/成员状态，例如 `split_k_mode`, `status`。

### Lines 1619-1622
```cpp
1619:       if (status != Status::kSuccess) {
1620:         return status;
1621:       }
1622:     }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 1624-1625
```cpp
1624:     return status;
1625:   };
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 1627-1628
```cpp
1627:   return profile_kernel_(result, options, func, gemm_workspace_.stream);
1628: }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 1630-1630
```cpp
1630: /////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 1632-1633
```cpp
1632: } // namespace profiler
1633: } // namespace cutlass
```
- **EN:** Supporting logic for the block-scaled GEMM implementation.
- **CN:** 块缩放 GEMM实现的辅助逻辑。

### Lines 1635-1635
```cpp
1635: /////////////////////////////////////////////////////////////////////////////////////////////////
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
- **Internal headers / 内部头文件:** `cutlass/core_io.h`, `cutlass/profiler/cublas_helpers.h`, `cutlass/profiler/block_scaled_gemm_operation_profiler.h`, `cutlass/profiler/gpu_timer.h`, `cutlass/library/singleton.h`, `cutlass/library/library.h`, `cutlass/library/handle.h`, `cutlass/util/reference/host/gett.hpp`
- **External headers / 外部头文件:** `iostream`, `stdexcept`, `iomanip`, `ios`, `vector`
- **Runtime/backends / 运行时与后端:** `CUDA`, `cuBLAS`
