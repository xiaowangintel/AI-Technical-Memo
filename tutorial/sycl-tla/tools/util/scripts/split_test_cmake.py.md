# split_test_cmake.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `tools/util/scripts/split_test_cmake.py`
- **Purpose (EN):** This file implements test-file splitting for tooling/build scripts.
- **目的 (CN):** 该文件实现了面向工具/构建脚本的测试文件拆分逻辑。
- **Brief / 简述:** Given a set of test files to be included in a CMake target, this script extracts

## Line-by-Line Analysis / 逐行分析
### Lines 1-16
```python
1: #################################################################################################
2: #
3: # Copyright (c) 2017 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
4: # SPDX-License-Identifier: BSD-3-Clause
5: #
6: # Redistribution and use in source and binary forms, with or without
7: # modification, are permitted provided that the following conditions are met:
8: #
9: # 1. Redistributions of source code must retain the above copyright notice, this
10: # list of conditions and the following disclaimer.
11: #
12: # 2. Redistributions in binary form must reproduce the above copyright notice,
13: # this list of conditions and the following disclaimer in the documentation
14: # and/or other materials provided with the distribution.
15: #
16: # 3. Neither the name of the copyright holder nor the names of its
```
- **EN:** License header and copyright terms for the file.
- **CN:** 文件的许可证头和版权条款。

### Lines 17-31
```python
17: # contributors may be used to endorse or promote products derived from
18: # this software without specific prior written permission.
19: #
20: # THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
21: # AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
22: # IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
23: # DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
24: # FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
25: # DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
26: # SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
27: # CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
28: # OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
29: # OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
30: #
31: #################################################################################################
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 34-57
```python
34: """
35: Given a set of test files to be included in a CMake target, this script extracts
36: the TEST definitions from each file, writes them into new files, and prints the names
37: of the new files so that they can be processed as part of a new CMake target.
38: 
39: For example, given a set of --src_files test_a.cu test_b.cu containing 3 and 2 TEST
40: definitions, respectively, this script would produce:
41:     test_a_000.cu
42:     test_a_001.cu
43:     test_a_002.cu
44:     test_b_000.cu
45:     test_b_001.cu
46: 
47: The splitting follows a fairly rudimentary algorithm that does not support all valid C++ programs.
48: We walk through a given input test file line by line. Any lines that are not within a TEST definition is added to a running
49: "filler" text. When a TEST definition is encountered, the current filler text becomes the prefix
50: for that test. All subsequent lines are considered to be part of the TEST definition until the
51: number of starting function braces ('{') match the number of closing function braces ('}'). When
52: these counts are equal, the TEST definition is considered to be completed. At this point, we return
53: to adding lines to the "filler" text until a new TEST definition is encountered. Any "filler" text
54: following a TEST definition is added to the suffix of that TEST definition (this is useful for finishing
55: off #if statements, as is common in unit tests.).
56: 
57: A state machine illustrating this algorithm at a high level is provided in the source below.
```
- **EN:** Module or file-level documentation describing purpose, algorithm, assumptions, and examples.
- **CN:** 模块或文件级文档，描述目的、算法、假设和示例。

### Lines 58-81
```python
58: 
59: Example: Suppose an input test `test.cu` has the following source:
60:     // COPYRIGHT
61:     #include <iostream>
62: 
63:     #if defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)
64: 
65:     // Test #1
66:     TEST(SM90_a, 256x128x64_2x2x1) {
67:         std::cout << "Test #1" << std::endl;
68:     }
69: 
70:     // Test #2
71:     TEST(SM90_b, 256x128x64_1x1x1) {
72:         std::cout << "Test #2" << std::endl;
73:     }
74: 
75:     #endif defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)
76: 
77: The contents of the two resulting test files will be:
78:   $ cat test_000.cu
79:     // COPYRIGHT
80:     #include <iostream>
81: 
```
- **EN:** Module or file-level documentation describing purpose, algorithm, assumptions, and examples.
- **CN:** 模块或文件级文档，描述目的、算法、假设和示例。

### Lines 82-105
```python
82:     #if defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)
83: 
84:     // Test #1
85:     TEST(SM90_a, 256x128x64_2x2x1) {
86:         std::cout << "Test #1" << std::endl;
87:     }
88: 
89:     // Test #2
90: 
91:     #endif defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)
92:   $ cat test_001.cu
93:     // COPYRIGHT
94:     #include <iostream>
95: 
96:     #if defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)
97: 
98:     // Test #1
99: 
100:     // Test #2
101:     TEST(SM90_b, 256x128x64_1x1x1) {
102:         std::cout << "Test #2" << std::endl;
103:     }
104: 
105:     #endif defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)
```
- **EN:** Module or file-level documentation describing purpose, algorithm, assumptions, and examples.
- **CN:** 模块或文件级文档，描述目的、算法、假设和示例。

### Lines 106-129
```python
106: 
107: Notice that each of test_000.cu and test_001.cu contain comments that appear outside
108: the TEST definitions not included in each file. This is by design, as these
109: would be considered "filler" text.
110: 
111: As expected, some cases can't be handled. Below is a non-exhaustive list:
112:     1. New TEST following the closing '}' of a TEST case on the same line:
113:         TEST(x, y) {
114:             // Do stuff
115:         } TEST(a, b) {
116: 
117:         In this case, "TEST(a, b) {" will be ignored
118: 
119:     2. Preprocessor macros that occur midway through a test case and extend
120:        beyond the conclusion of a testcase
121: 
122:        Example:
123:             TEST(a, b) {
124:                 // Do stuff
125:         #if X
126:                 // Do more stuff
127:             }
128:         #else
129:                 // Do other stuff
```
- **EN:** Module or file-level documentation describing purpose, algorithm, assumptions, and examples.
- **CN:** 模块或文件级文档，描述目的、算法、假设和示例。

### Lines 130-132
```python
130:             }
131:         #endif
132: """
```
- **EN:** Module or file-level documentation describing purpose, algorithm, assumptions, and examples.
- **CN:** 模块或文件级文档，描述目的、算法、假设和示例。

### Lines 135-137
```python
135: import argparse
136: import enum
137: import os
```
- **EN:** Imports Python modules used by the script for argument parsing, enums, and filesystem work.
- **CN:** 导入脚本所需的 Python 模块，用于参数解析、枚举和文件系统处理。

### Lines 140-151
```python
140: parser = argparse.ArgumentParser()
141: parser.add_argument("cmake_target", type=str,
142:                     help="Name of the CMake target being generated.")
143: parser.add_argument("src_dir", type=str,
144:                     help="Path to the directory containing test files.")
145: parser.add_argument("--src_files", nargs='+',
146:                     help="Files containing TEST instances to split.")
147: parser.add_argument("--max_tests_per_file", type=int, default=1,
148:                     help="Maximum number of TEST instances per file.")
149: parser.add_argument("--dst_dir", type=str,
150:                     help="Path to the directory to which to write new test files. If not set, uses src_dir.")
151: args = parser.parse_args()
```
- **EN:** Defines command-line arguments that control input files, output paths, and script behavior.
- **CN:** 定义控制输入文件、输出路径和脚本行为的命令行参数。

### Lines 154-155
```python
154: if args.dst_dir == None:
155:     args.dst_dir = args.src_dir
```
- **EN:** Declares or updates local/member state such as `dst_dir`.
- **CN:** 声明或更新局部/成员状态，例如 `dst_dir`。

### Lines 158-158
```python
158: class Testcase:
```
- **EN:** Introduces `Testcase`, state tracked while splitting source tests.
- **CN:** 引入 `Testcase`，即拆分测试源码时跟踪的状态。

### Lines 159-161
```python
159:     """
160:     Lightweight tracker of test-case processing status
161:     """
```
- **EN:** Module or file-level documentation describing purpose, algorithm, assumptions, and examples.
- **CN:** 模块或文件级文档，描述目的、算法、假设和示例。

### Lines 162-162
```python
162:     def __init__(self, prefix_text):
```
- **EN:** Supporting logic for the test-file splitting implementation.
- **CN:** 测试文件拆分实现的辅助逻辑。

### Lines 163-164
```python
163:         # Any text that preceded the TEST definition that was
164:         # not part of another TEST definition
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 165-165
```python
165:         self.prefix = prefix_text
```
- **EN:** Declares or updates local/member state such as `prefix`.
- **CN:** 声明或更新局部/成员状态，例如 `prefix`。

### Lines 167-167
```python
167:         # Any text within the TEST definition
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 168-168
```python
168:         self.test = ""
```
- **EN:** Declares or updates local/member state such as `test`.
- **CN:** 声明或更新局部/成员状态，例如 `test`。

### Lines 170-171
```python
170:         # Any text that follows the completion of the TEST definition
171:         # and is not included in other TEST definitions
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 172-172
```python
172:         self.suffix = ""
```
- **EN:** Declares or updates local/member state such as `suffix`.
- **CN:** 声明或更新局部/成员状态，例如 `suffix`。

### Lines 174-174
```python
174:         # Whether the test's definition has concluded
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 175-175
```python
175:         self.completed = False
```
- **EN:** Declares or updates local/member state such as `completed`.
- **CN:** 声明或更新局部/成员状态，例如 `completed`。

### Lines 177-180
```python
177:         # Current balance of opening and closing curly brackets in
178:         # the TEST definition. '{' increments the count and '}' decrements it.
179:         # A value of 0 (when self.completed == False) indicates that the test
180:         # has completed.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 181-181
```python
181:         self.curly_bracket_balance = 0
```
- **EN:** Declares or updates local/member state such as `curly_bracket_balance`.
- **CN:** 声明或更新局部/成员状态，例如 `curly_bracket_balance`。

### Lines 184-184
```python
184: class ParseState(enum.Enum):
```
- **EN:** Introduces `ParseState`, a type used to support test-file splitting.
- **CN:** 引入 `ParseState`，即一个用于支持测试文件拆分的类型。

### Lines 185-207
```python
185:     """
186:       State machine for processing.
187:       Transitions occur on each line encountered in the soruce file
188: 
189: 
190:       Line does not contain 'TEST('
191:                  +----+
192:                  |    |
193:                  |    v          'TEST('
194:                +--------+      encountered         +--------------------------+
195:         ------>| Filler | -----------------------> | TestDeclaredWaitingStart |
196:                +--------+                          +--------------------------+
197:                    ^                                         |
198:  Number of '{'     |                                         | First '{' encountered
199:  equals number of  |           +--------+                    |
200:  '}' encountered   +-----------| InTest | <------------------+
201:                                +--------+
202:                                  |    ^
203:                                  |    |
204:                                  +----+
205:                       Number of '{' encountered
206:                       exceeds number of '}' encountered
207:     """
```
- **EN:** Module or file-level documentation describing purpose, algorithm, assumptions, and examples.
- **CN:** 模块或文件级文档，描述目的、算法、假设和示例。

### Lines 210-210
```python
210:     # Any text that is not part of a TEST case
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 211-211
```python
211:     Filler = 0
```
- **EN:** Declares or updates local/member state such as `Filler`.
- **CN:** 声明或更新局部/成员状态，例如 `Filler`。

### Lines 213-214
```python
213:     # Processing text within the first { of the TEST case
214:     # and before the en of the final } of the TEST case
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 215-215
```python
215:     InTest = 1
```
- **EN:** Declares or updates local/member state such as `InTest`.
- **CN:** 声明或更新局部/成员状态，例如 `InTest`。

### Lines 217-219
```python
217:     # Processing text from the start of the TEST definition
218:     # but before the first {. This could occur if the opening {
219:     # occurs on a separate line than the TEST definition.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 220-220
```python
220:     TestDeclaredWaitingStart = 2
```
- **EN:** Declares or updates local/member state such as `TestDeclaredWaitingStart`.
- **CN:** 声明或更新局部/成员状态，例如 `TestDeclaredWaitingStart`。

### Lines 223-225
```python
223: cmake_src_list = []
224: for filename in args.src_files:
225:     if '.' not in filename:
```
- **EN:** Declares or updates local/member state such as `cmake_src_list`.
- **CN:** 声明或更新局部/成员状态，例如 `cmake_src_list`。

### Lines 226-226
```python
226:         # Add any non-filename arguments to the command list by default
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 227-228
```python
227:         cmake_src_list.append(filename)
228:         continue
```
- **EN:** Supporting logic for the test-file splitting implementation.
- **CN:** 测试文件拆分实现的辅助逻辑。

### Lines 230-234
```python
230:     if '/' in filename:
231:         raise Exception(
232:             f"Source files passed to {__file__} must be within the same directory "
233:             "as the CMakeLists defining the target using the files. "
234:             f"Provided path {filename} is in a different directory.")
```
- **EN:** Supporting logic for the test-file splitting implementation.
- **CN:** 测试文件拆分实现的辅助逻辑。

### Lines 236-238
```python
236:     full_filename = os.path.join(args.src_dir, filename)
237:     with open(full_filename, 'r') as infile:
238:         lines = infile.readlines()
```
- **EN:** Implements `join` and coordinates helper calls such as `open`, `readlines`.
- **CN:** 实现 `join`，并协调调用 `open`, `readlines` 等辅助逻辑。

### Lines 240-240
```python
240:     # Find the number of instances of "TEST("
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 241-241
```python
241:     ntest = sum([1 for line in lines if "TEST(" in line])
```
- **EN:** Implements `sum` for this file's main component.
- **CN:** 为该文件的核心组件实现 `sum`。

### Lines 243-243
```python
243:     if ntest <= args.max_tests_per_file:
```
- **EN:** Supporting logic for the test-file splitting implementation.
- **CN:** 测试文件拆分实现的辅助逻辑。

### Lines 244-245
```python
244:         # File contains fewer than max_tests_per_file TEST instances. It does
245:         # not need to be split
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 246-247
```python
246:         cmake_src_list.append(filename)
247:         continue
```
- **EN:** Supporting logic for the test-file splitting implementation.
- **CN:** 测试文件拆分实现的辅助逻辑。

### Lines 249-249
```python
249:     # Current state of the parsing state machine. We start with filler text
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 250-250
```python
250:     state = ParseState.Filler
```
- **EN:** Declares or updates local/member state such as `state`.
- **CN:** 声明或更新局部/成员状态，例如 `state`。

### Lines 252-252
```python
252:     # List of individual TESTs found
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 253-253
```python
253:     tests = []
```
- **EN:** Declares or updates local/member state such as `tests`.
- **CN:** 声明或更新局部/成员状态，例如 `tests`。

### Lines 255-256
```python
255:     # Ongoing text that is not included in a TEST definition. This will serve
256:     # as the prefix for any yet-to-be encountered TEST definitions.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 257-257
```python
257:     filler_text = ""
```
- **EN:** Declares or updates local/member state such as `filler_text`.
- **CN:** 声明或更新局部/成员状态，例如 `filler_text`。

### Lines 259-260
```python
259:     def add_filler_text(text):
260:         global filler_text
```
- **EN:** Supporting logic for the test-file splitting implementation.
- **CN:** 测试文件拆分实现的辅助逻辑。

### Lines 261-262
```python
261:         # Add new text to the ongoing filler text and to the suffixes of
262:         # any completed tests
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 263-266
```python
263:         filler_text += text
264:         for i in range(len(tests)):
265:             if tests[i].completed:
266:                 tests[i].suffix += text
```
- **EN:** Supporting logic for the test-file splitting implementation.
- **CN:** 测试文件拆分实现的辅助逻辑。

### Lines 268-269
```python
268:     for line in lines:
269:         if state == ParseState.Filler:
```
- **EN:** Declares or updates local/member state such as `state`.
- **CN:** 声明或更新局部/成员状态，例如 `state`。

### Lines 270-270
```python
270:             # We are not currently within a TEST definition.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 272-272
```python
272:             if 'TEST(' in line:
```
- **EN:** Contains or manipulates unit-test definitions that will later be built or split.
- **CN:** 包含或处理后续会被构建或拆分的单元测试定义。

### Lines 273-277
```python
273:                 # We have encountered a new TEST( case. Any text preceding this
274:                 # must be added to the filler text (e.g., if we have a line of the form:
275:                 #   "static constexpr int Val = 4; TEST(blah) {"
276:                 #   then "static constexpr int Val = 4;" needs to be included in filler
277:                 #   text, as it could be used by subsequent tests.)
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 278-278
```python
278:                 splits = line.split('TEST')
```
- **EN:** Implements `split` for this file's main component.
- **CN:** 为该文件的核心组件实现 `split`。

### Lines 280-280
```python
280:                 # There should not be more than one TEST definition on a given line
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 281-281
```python
281:                 assert len(splits) <= 2
```
- **EN:** Supporting logic for the test-file splitting implementation.
- **CN:** 测试文件拆分实现的辅助逻辑。

### Lines 283-284
```python
283:                 if len(splits) > 1:
284:                     if not splits[0].isspace():
```
- **EN:** Supporting logic for the test-file splitting implementation.
- **CN:** 测试文件拆分实现的辅助逻辑。

### Lines 285-286
```python
285:                         # Only add text to filler if there are non-whitespace charcters
286:                         # preceding the TEST definition in the line
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 287-287
```python
287:                         filler_text += splits[0]
```
- **EN:** Supporting logic for the test-file splitting implementation.
- **CN:** 测试文件拆分实现的辅助逻辑。

### Lines 289-289
```python
289:                         # The new line is just the TEST-related line
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 290-290
```python
290:                         line = 'TEST' + splits[-1]
```
- **EN:** Declares or updates local/member state such as `line`.
- **CN:** 声明或更新局部/成员状态，例如 `line`。

### Lines 292-295
```python
292:                 # Add tests and transtion to TestDeclaredWaitingStart state.
293:                 # Do not add the line to the test text of the new test case; this
294:                 # will be done in either the TestDeclaredWaitingStart state processing
295:                 # below or in the InTest state processing below.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 296-298
```python
296:                 tests.append(Testcase(filler_text))
297:                 state = ParseState.TestDeclaredWaitingStart
298:             else:
```
- **EN:** Declares or updates local/member state such as `state`.
- **CN:** 声明或更新局部/成员状态，例如 `state`。

### Lines 299-301
```python
299:                 # Any remaining filler text is added to the running filler_text
300:                 # which will be used as the prefix for any new tests, and to the
301:                 # suffix of any completed tests
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 302-302
```python
302:                 add_filler_text(line)
```
- **EN:** Supporting logic for the test-file splitting implementation.
- **CN:** 测试文件拆分实现的辅助逻辑。

### Lines 304-304
```python
304:         if state == ParseState.TestDeclaredWaitingStart:
```
- **EN:** Declares or updates local/member state such as `state`.
- **CN:** 声明或更新局部/成员状态，例如 `state`。

### Lines 305-305
```python
305:             # We have seen a TEST definition but have not yet seen its opening {.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 307-307
```python
307:             if '{' in line:
```
- **EN:** Supporting logic for the test-file splitting implementation.
- **CN:** 测试文件拆分实现的辅助逻辑。

### Lines 308-311
```python
308:                 # The first curly bracket for the TEST definition has been found.
309:                 # Advance to state InTests. Do not add the line to the test's text
310:                 # or change the curly-brace balance of the test; these will be done
311:                 # when processing the state == ParseState.InTest condition below.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 312-314
```python
312:                 state = ParseState.InTest
313:             else:
314:                 tests[-1].test += line
```
- **EN:** Declares or updates local/member state such as `state`.
- **CN:** 声明或更新局部/成员状态，例如 `state`。

### Lines 316-316
```python
316:         if state == ParseState.InTest:
```
- **EN:** Declares or updates local/member state such as `state`.
- **CN:** 声明或更新局部/成员状态，例如 `state`。

### Lines 317-320
```python
317:             # We are currently within a TEST definition.
318:             # Process lines character-by-character looking for opening and closing
319:             # braces. If we reach parity between opening and closing braces, the
320:             # test is considered done.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 321-332
```python
321:             filler_text_to_add = ""
322:             for char in line:
323:                 if not tests[-1].completed:
324:                     tests[-1].test += char
325:                     if char == '{':
326:                         tests[-1].curly_bracket_balance += 1
327:                     elif char == '}':
328:                         tests[-1].curly_bracket_balance -= 1
329:                         if tests[-1].curly_bracket_balance == 0:
330:                             tests[-1].completed = True
331:                 else:
332:                     filler_text_to_add += char
```
- **EN:** Declares or updates local/member state such as `filler_text_to_add`, `char`, `curly_bracket_balance`, `completed`.
- **CN:** 声明或更新局部/成员状态，例如 `filler_text_to_add`, `char`, `curly_bracket_balance`, `completed`。

### Lines 334-335
```python
334:             if filler_text_to_add != "" and (not filler_text_to_add.isspace() or '\n' in filler_text_to_add):
335:                 add_filler_text('\n' + filler_text_to_add)
```
- **EN:** Supporting logic for the test-file splitting implementation.
- **CN:** 测试文件拆分实现的辅助逻辑。

### Lines 337-338
```python
337:             if tests[-1].completed:
338:                 state = ParseState.Filler
```
- **EN:** Declares or updates local/member state such as `state`.
- **CN:** 声明或更新局部/成员状态，例如 `state`。

### Lines 340-340
```python
340:     # Write out the new files for tests
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 341-345
```python
341:     filename_prefix, filename_suffix = filename.split('.')
342:     for i, test in enumerate(tests):
343:         assert test.completed
344:         new_filename = filename_prefix + '_' + str(i).zfill(3) + '.' + filename_suffix
345:         full_new_filename = os.path.join(args.dst_dir, new_filename)
```
- **EN:** Declares or updates local/member state such as `filename_suffix`, `new_filename`, `full_new_filename`.
- **CN:** 声明或更新局部/成员状态，例如 `filename_suffix`, `new_filename`, `full_new_filename`。

### Lines 347-347
```python
347:         # Replace any '\' with '/'. CMake doesn't like '\'.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 348-348
```python
348:         full_new_filename = full_new_filename.replace('\\', '/')
```
- **EN:** Implements `replace` for this file's main component.
- **CN:** 为该文件的核心组件实现 `replace`。

### Lines 350-352
```python
350:         with open(full_new_filename, 'w') as outfile:
351:             outfile.write(test.prefix + test.test + test.suffix)
352:         cmake_src_list.append(full_new_filename)
```
- **EN:** Implements `open` and coordinates helper calls such as `write`, `append`.
- **CN:** 实现 `open`，并协调调用 `write`, `append` 等辅助逻辑。

### Lines 355-356
```python
355: for cmake_file in cmake_src_list:
356:     print(cmake_file)
```
- **EN:** Emits debug or diagnostic output so intermediate state can be inspected.
- **CN:** 输出调试或诊断信息，便于检查中间状态。

## Key Concepts / 关键概念
- **Strongly typed enums / 强类型枚举**
- **Structured type design / 结构化类型设计**
- **Conditional compilation / 条件编译**
- **CUDA ecosystem integration / CUDA 生态集成**
- **Build/test automation / 构建/测试自动化**

## Dependencies / 依赖关系
- **External headers / 外部头文件:** `iostream`, `iostream`, `iostream`
- **Python modules / Python 模块:** `argparse`, `enum`, `os`
