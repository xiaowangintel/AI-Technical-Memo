# verbosity.py — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/utils/lit/tests/verbosity.py` | `llvm/utils/lit/tests/verbosity.py` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | Test various combinations of options controlling lit stdout and stderr output. | 实现 LLVM 的 lit 测试基础设施、运行器、辅助工具或测试夹具。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-28

````python
# Test various combinations of options controlling lit stdout and stderr output

# RUN: mkdir -p %t

### Test default

# RUN: not %{lit} %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt
# RUN: FileCheck %s --check-prefix NO-ARGS < %t/stdout.txt
# RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt

# NO-ARGS:      -- Testing: 5 tests, 1 workers --
# NO-ARGS-NEXT: FAIL: verbosity :: fail.txt (1 of 5)
# NO-ARGS-NEXT: PASS: verbosity :: pass.txt (2 of 5)
# NO-ARGS-NEXT: {{UN}}SUPPORTED: verbosity :: unsupported.txt (3 of 5)
# NO-ARGS-NEXT: {{X}}FAIL: verbosity :: xfail.txt (4 of 5)
# NO-ARGS-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)
# NO-ARGS-NEXT: ********************
# NO-ARGS-NEXT: Failed Tests (1):
# NO-ARGS-NEXT:   verbosity :: fail.txt
# NO-ARGS-EMPTY:
# NO-ARGS-NEXT: ********************
# NO-ARGS-NEXT: Unexpectedly Passed Tests (1):
# NO-ARGS-NEXT:   verbosity :: xpass.txt
# NO-ARGS-EMPTY:
# NO-ARGS-EMPTY:
# NO-ARGS-NEXT: Testing Time: {{.*}}s
# NO-ARGS-EMPTY:
# NO-ARGS-NEXT: Total Discovered Tests: 5
````
- **L1 EN**: Comment documents nearby script behavior: `Test various combinations of options controlling lit stdout and stderr output`.
  **L1 CN**: 注释说明了附近脚本逻辑：`Test various combinations of options controlling lit stdout and stderr output`。
- **L2 EN**: Blank line separates nearby declarations or logic blocks.
  **L2 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3 EN**: Comment documents nearby script behavior: `RUN: mkdir -p %t`.
  **L3 CN**: 注释说明了附近脚本逻辑：`RUN: mkdir -p %t`。
- **L4 EN**: Blank line separates nearby declarations or logic blocks.
  **L4 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L5 EN**: Comment documents nearby script behavior: `## Test default`.
  **L5 CN**: 注释说明了附近脚本逻辑：`## Test default`。
- **L6 EN**: Blank line separates nearby declarations or logic blocks.
  **L6 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L7 EN**: Comment documents nearby script behavior: `RUN: not %{lit} %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt`.
  **L7 CN**: 注释说明了附近脚本逻辑：`RUN: not %{lit} %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt`。
- **L8 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix NO-ARGS < %t/stdout.txt`.
  **L8 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix NO-ARGS < %t/stdout.txt`。
- **L9 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt`.
  **L9 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt`。
- **L10 EN**: Blank line separates nearby declarations or logic blocks.
  **L10 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L11 EN**: Comment documents nearby script behavior: `NO-ARGS: -- Testing: 5 tests, 1 workers`.
  **L11 CN**: 注释说明了附近脚本逻辑：`NO-ARGS: -- Testing: 5 tests, 1 workers`。
- **L12 EN**: Comment documents nearby script behavior: `NO-ARGS-NEXT: FAIL: verbosity :: fail.txt (1 of 5)`.
  **L12 CN**: 注释说明了附近脚本逻辑：`NO-ARGS-NEXT: FAIL: verbosity :: fail.txt (1 of 5)`。
- **L13 EN**: Comment documents nearby script behavior: `NO-ARGS-NEXT: PASS: verbosity :: pass.txt (2 of 5)`.
  **L13 CN**: 注释说明了附近脚本逻辑：`NO-ARGS-NEXT: PASS: verbosity :: pass.txt (2 of 5)`。
- **L14 EN**: Comment documents nearby script behavior: `NO-ARGS-NEXT: {{UN}}SUPPORTED: verbosity :: unsupported.txt (3 of 5)`.
  **L14 CN**: 注释说明了附近脚本逻辑：`NO-ARGS-NEXT: {{UN}}SUPPORTED: verbosity :: unsupported.txt (3 of 5)`。
- **L15 EN**: Comment documents nearby script behavior: `NO-ARGS-NEXT: {{X}}FAIL: verbosity :: xfail.txt (4 of 5)`.
  **L15 CN**: 注释说明了附近脚本逻辑：`NO-ARGS-NEXT: {{X}}FAIL: verbosity :: xfail.txt (4 of 5)`。
- **L16 EN**: Comment documents nearby script behavior: `NO-ARGS-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)`.
  **L16 CN**: 注释说明了附近脚本逻辑：`NO-ARGS-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)`。
- **L17 EN**: Comment documents nearby script behavior: `NO-ARGS-NEXT: ********************`.
  **L17 CN**: 注释说明了附近脚本逻辑：`NO-ARGS-NEXT: ********************`。
- **L18 EN**: Comment documents nearby script behavior: `NO-ARGS-NEXT: Failed Tests (1):`.
  **L18 CN**: 注释说明了附近脚本逻辑：`NO-ARGS-NEXT: Failed Tests (1):`。
- **L19 EN**: Comment documents nearby script behavior: `NO-ARGS-NEXT: verbosity :: fail.txt`.
  **L19 CN**: 注释说明了附近脚本逻辑：`NO-ARGS-NEXT: verbosity :: fail.txt`。
- **L20 EN**: Comment documents nearby script behavior: `NO-ARGS-EMPTY:`.
  **L20 CN**: 注释说明了附近脚本逻辑：`NO-ARGS-EMPTY:`。
- **L21 EN**: Comment documents nearby script behavior: `NO-ARGS-NEXT: ********************`.
  **L21 CN**: 注释说明了附近脚本逻辑：`NO-ARGS-NEXT: ********************`。
- **L22 EN**: Comment documents nearby script behavior: `NO-ARGS-NEXT: Unexpectedly Passed Tests (1):`.
  **L22 CN**: 注释说明了附近脚本逻辑：`NO-ARGS-NEXT: Unexpectedly Passed Tests (1):`。
- **L23 EN**: Comment documents nearby script behavior: `NO-ARGS-NEXT: verbosity :: xpass.txt`.
  **L23 CN**: 注释说明了附近脚本逻辑：`NO-ARGS-NEXT: verbosity :: xpass.txt`。
- **L24 EN**: Comment documents nearby script behavior: `NO-ARGS-EMPTY:`.
  **L24 CN**: 注释说明了附近脚本逻辑：`NO-ARGS-EMPTY:`。
- **L25 EN**: Comment documents nearby script behavior: `NO-ARGS-EMPTY:`.
  **L25 CN**: 注释说明了附近脚本逻辑：`NO-ARGS-EMPTY:`。
- **L26 EN**: Comment documents nearby script behavior: `NO-ARGS-NEXT: Testing Time: {{.*}}s`.
  **L26 CN**: 注释说明了附近脚本逻辑：`NO-ARGS-NEXT: Testing Time: {{.*}}s`。
- **L27 EN**: Comment documents nearby script behavior: `NO-ARGS-EMPTY:`.
  **L27 CN**: 注释说明了附近脚本逻辑：`NO-ARGS-EMPTY:`。
- **L28 EN**: Comment documents nearby script behavior: `NO-ARGS-NEXT: Total Discovered Tests: 5`.
  **L28 CN**: 注释说明了附近脚本逻辑：`NO-ARGS-NEXT: Total Discovered Tests: 5`。

### Lines 29-42

````python
# NO-ARGS-NEXT:   Unsupported        : 1 (20.00%)
# NO-ARGS-NEXT:   Passed             : 1 (20.00%)
# NO-ARGS-NEXT:   Expectedly Failed  : 1 (20.00%)
# NO-ARGS-NEXT:   Failed             : 1 (20.00%)
# NO-ARGS-NEXT:   Unexpectedly Passed: 1 (20.00%)

# NO-ARGS-ERR: lit.py: {{.*}}lit.cfg:{{[0-9]+}}: note: this is a note
# NO-ARGS-ERR-NEXT: lit.py: {{.*}}lit.cfg:{{[0-9]+}}: warning: this is a warning
# NO-ARGS-ERR-EMPTY:
# NO-ARGS-ERR-NEXT: 1 warning(s) in tests


### Test aliases

````
- **L29 EN**: Comment documents nearby script behavior: `NO-ARGS-NEXT: Unsupported : 1 (20.00%)`.
  **L29 CN**: 注释说明了附近脚本逻辑：`NO-ARGS-NEXT: Unsupported : 1 (20.00%)`。
- **L30 EN**: Comment documents nearby script behavior: `NO-ARGS-NEXT: Passed : 1 (20.00%)`.
  **L30 CN**: 注释说明了附近脚本逻辑：`NO-ARGS-NEXT: Passed : 1 (20.00%)`。
- **L31 EN**: Comment documents nearby script behavior: `NO-ARGS-NEXT: Expectedly Failed : 1 (20.00%)`.
  **L31 CN**: 注释说明了附近脚本逻辑：`NO-ARGS-NEXT: Expectedly Failed : 1 (20.00%)`。
- **L32 EN**: Comment documents nearby script behavior: `NO-ARGS-NEXT: Failed : 1 (20.00%)`.
  **L32 CN**: 注释说明了附近脚本逻辑：`NO-ARGS-NEXT: Failed : 1 (20.00%)`。
- **L33 EN**: Comment documents nearby script behavior: `NO-ARGS-NEXT: Unexpectedly Passed: 1 (20.00%)`.
  **L33 CN**: 注释说明了附近脚本逻辑：`NO-ARGS-NEXT: Unexpectedly Passed: 1 (20.00%)`。
- **L34 EN**: Blank line separates nearby declarations or logic blocks.
  **L34 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L35 EN**: Comment documents nearby script behavior: `NO-ARGS-ERR: lit.py: {{.*}}lit.cfg:{{[0-9]+}}: note: this is a note`.
  **L35 CN**: 注释说明了附近脚本逻辑：`NO-ARGS-ERR: lit.py: {{.*}}lit.cfg:{{[0-9]+}}: note: this is a note`。
- **L36 EN**: Comment documents nearby script behavior: `NO-ARGS-ERR-NEXT: lit.py: {{.*}}lit.cfg:{{[0-9]+}}: warning: this is a warning`.
  **L36 CN**: 注释说明了附近脚本逻辑：`NO-ARGS-ERR-NEXT: lit.py: {{.*}}lit.cfg:{{[0-9]+}}: warning: this is a warning`。
- **L37 EN**: Comment documents nearby script behavior: `NO-ARGS-ERR-EMPTY:`.
  **L37 CN**: 注释说明了附近脚本逻辑：`NO-ARGS-ERR-EMPTY:`。
- **L38 EN**: Comment documents nearby script behavior: `NO-ARGS-ERR-NEXT: 1 warning(s) in tests`.
  **L38 CN**: 注释说明了附近脚本逻辑：`NO-ARGS-ERR-NEXT: 1 warning(s) in tests`。
- **L39 EN**: Blank line separates nearby declarations or logic blocks.
  **L39 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L40 EN**: Blank line separates nearby declarations or logic blocks.
  **L40 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L41 EN**: Comment documents nearby script behavior: `## Test aliases`.
  **L41 CN**: 注释说明了附近脚本逻辑：`## Test aliases`。
- **L42 EN**: Blank line separates nearby declarations or logic blocks.
  **L42 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 43-70

````python
# RUN: not %{lit} --succinct %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt
# RUN: FileCheck %s --check-prefix SUCCINCT < %t/stdout.txt
# RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt

# SUCCINCT:      -- Testing: 5 tests, 1 workers --
# SUCCINCT-NEXT: Testing:
# SUCCINCT-NEXT: FAIL: verbosity :: fail.txt (1 of 5)
# SUCCINCT-NEXT: Testing:
# SUCCINCT-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)
# SUCCINCT-NEXT: Testing:
# SUCCINCT-NEXT: ********************
# SUCCINCT-NEXT: Failed Tests (1):
# SUCCINCT-NEXT:   verbosity :: fail.txt
# SUCCINCT-EMPTY:
# SUCCINCT-NEXT: ********************
# SUCCINCT-NEXT: Unexpectedly Passed Tests (1):
# SUCCINCT-NEXT:   verbosity :: xpass.txt
# SUCCINCT-EMPTY:
# SUCCINCT-EMPTY:
# SUCCINCT-NEXT: Testing Time: {{.*}}s
# SUCCINCT-EMPTY:
# SUCCINCT-NEXT: Total Discovered Tests: 5
# SUCCINCT-NEXT:   Unsupported        : 1 (20.00%)
# SUCCINCT-NEXT:   Passed             : 1 (20.00%)
# SUCCINCT-NEXT:   Expectedly Failed  : 1 (20.00%)
# SUCCINCT-NEXT:   Failed             : 1 (20.00%)
# SUCCINCT-NEXT:   Unexpectedly Passed: 1 (20.00%)

````
- **L43 EN**: Comment documents nearby script behavior: `RUN: not %{lit} --succinct %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt`.
  **L43 CN**: 注释说明了附近脚本逻辑：`RUN: not %{lit} --succinct %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt`。
- **L44 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix SUCCINCT < %t/stdout.txt`.
  **L44 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix SUCCINCT < %t/stdout.txt`。
- **L45 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt`.
  **L45 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt`。
- **L46 EN**: Blank line separates nearby declarations or logic blocks.
  **L46 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L47 EN**: Comment documents nearby script behavior: `SUCCINCT: -- Testing: 5 tests, 1 workers`.
  **L47 CN**: 注释说明了附近脚本逻辑：`SUCCINCT: -- Testing: 5 tests, 1 workers`。
- **L48 EN**: Comment documents nearby script behavior: `SUCCINCT-NEXT: Testing:`.
  **L48 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-NEXT: Testing:`。
- **L49 EN**: Comment documents nearby script behavior: `SUCCINCT-NEXT: FAIL: verbosity :: fail.txt (1 of 5)`.
  **L49 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-NEXT: FAIL: verbosity :: fail.txt (1 of 5)`。
- **L50 EN**: Comment documents nearby script behavior: `SUCCINCT-NEXT: Testing:`.
  **L50 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-NEXT: Testing:`。
- **L51 EN**: Comment documents nearby script behavior: `SUCCINCT-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)`.
  **L51 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)`。
- **L52 EN**: Comment documents nearby script behavior: `SUCCINCT-NEXT: Testing:`.
  **L52 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-NEXT: Testing:`。
- **L53 EN**: Comment documents nearby script behavior: `SUCCINCT-NEXT: ********************`.
  **L53 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-NEXT: ********************`。
- **L54 EN**: Comment documents nearby script behavior: `SUCCINCT-NEXT: Failed Tests (1):`.
  **L54 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-NEXT: Failed Tests (1):`。
- **L55 EN**: Comment documents nearby script behavior: `SUCCINCT-NEXT: verbosity :: fail.txt`.
  **L55 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-NEXT: verbosity :: fail.txt`。
- **L56 EN**: Comment documents nearby script behavior: `SUCCINCT-EMPTY:`.
  **L56 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-EMPTY:`。
- **L57 EN**: Comment documents nearby script behavior: `SUCCINCT-NEXT: ********************`.
  **L57 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-NEXT: ********************`。
- **L58 EN**: Comment documents nearby script behavior: `SUCCINCT-NEXT: Unexpectedly Passed Tests (1):`.
  **L58 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-NEXT: Unexpectedly Passed Tests (1):`。
- **L59 EN**: Comment documents nearby script behavior: `SUCCINCT-NEXT: verbosity :: xpass.txt`.
  **L59 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-NEXT: verbosity :: xpass.txt`。
- **L60 EN**: Comment documents nearby script behavior: `SUCCINCT-EMPTY:`.
  **L60 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-EMPTY:`。
- **L61 EN**: Comment documents nearby script behavior: `SUCCINCT-EMPTY:`.
  **L61 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-EMPTY:`。
- **L62 EN**: Comment documents nearby script behavior: `SUCCINCT-NEXT: Testing Time: {{.*}}s`.
  **L62 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-NEXT: Testing Time: {{.*}}s`。
- **L63 EN**: Comment documents nearby script behavior: `SUCCINCT-EMPTY:`.
  **L63 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-EMPTY:`。
- **L64 EN**: Comment documents nearby script behavior: `SUCCINCT-NEXT: Total Discovered Tests: 5`.
  **L64 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-NEXT: Total Discovered Tests: 5`。
- **L65 EN**: Comment documents nearby script behavior: `SUCCINCT-NEXT: Unsupported : 1 (20.00%)`.
  **L65 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-NEXT: Unsupported : 1 (20.00%)`。
- **L66 EN**: Comment documents nearby script behavior: `SUCCINCT-NEXT: Passed : 1 (20.00%)`.
  **L66 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-NEXT: Passed : 1 (20.00%)`。
- **L67 EN**: Comment documents nearby script behavior: `SUCCINCT-NEXT: Expectedly Failed : 1 (20.00%)`.
  **L67 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-NEXT: Expectedly Failed : 1 (20.00%)`。
- **L68 EN**: Comment documents nearby script behavior: `SUCCINCT-NEXT: Failed : 1 (20.00%)`.
  **L68 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-NEXT: Failed : 1 (20.00%)`。
- **L69 EN**: Comment documents nearby script behavior: `SUCCINCT-NEXT: Unexpectedly Passed: 1 (20.00%)`.
  **L69 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-NEXT: Unexpectedly Passed: 1 (20.00%)`。
- **L70 EN**: Blank line separates nearby declarations or logic blocks.
  **L70 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 71-98

````python
# RUN: not %{lit} --verbose %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt
# RUN: FileCheck %s --check-prefix VERBOSE < %t/stdout.txt
# RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt

# VERBOSE:      -- Testing: 5 tests, 1 workers --
# VERBOSE-NEXT: FAIL: verbosity :: fail.txt (1 of 5)
# VERBOSE-NEXT: ******************** TEST 'verbosity :: fail.txt' FAILED ********************
# VERBOSE-NEXT: Exit Code: 127
# VERBOSE-EMPTY:
# VERBOSE-NEXT: Command Output (stdout):
# VERBOSE-NEXT: --
# VERBOSE-NEXT: # {{R}}UN: at line 1
# VERBOSE-NEXT: echo "fail test output"
# VERBOSE-NEXT: # executed command: echo 'fail test output'
# VERBOSE-NEXT: # .---command stdout------------
# VERBOSE-NEXT: # | fail test output
# VERBOSE-NEXT: # `-----------------------------
# VERBOSE-NEXT: # {{R}}UN: at line 2
# VERBOSE-NEXT: fail
# VERBOSE-NEXT: # executed command: fail
# VERBOSE-NEXT: # .---command stderr------------
# VERBOSE-NEXT: # | 'fail': command not found
# VERBOSE-NEXT: # `-----------------------------
# VERBOSE-NEXT: # error: command failed with exit status: 127
# VERBOSE-EMPTY:
# VERBOSE-NEXT: --
# VERBOSE-EMPTY:
# VERBOSE-NEXT: ********************
````
- **L71 EN**: Comment documents nearby script behavior: `RUN: not %{lit} --verbose %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt`.
  **L71 CN**: 注释说明了附近脚本逻辑：`RUN: not %{lit} --verbose %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt`。
- **L72 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix VERBOSE < %t/stdout.txt`.
  **L72 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix VERBOSE < %t/stdout.txt`。
- **L73 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt`.
  **L73 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt`。
- **L74 EN**: Blank line separates nearby declarations or logic blocks.
  **L74 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L75 EN**: Comment documents nearby script behavior: `VERBOSE: -- Testing: 5 tests, 1 workers`.
  **L75 CN**: 注释说明了附近脚本逻辑：`VERBOSE: -- Testing: 5 tests, 1 workers`。
- **L76 EN**: Comment documents nearby script behavior: `VERBOSE-NEXT: FAIL: verbosity :: fail.txt (1 of 5)`.
  **L76 CN**: 注释说明了附近脚本逻辑：`VERBOSE-NEXT: FAIL: verbosity :: fail.txt (1 of 5)`。
- **L77 EN**: Comment documents nearby script behavior: `VERBOSE-NEXT: ******************** TEST 'verbosity :: fail.txt' FAILED ****************...`.
  **L77 CN**: 注释说明了附近脚本逻辑：`VERBOSE-NEXT: ******************** TEST 'verbosity :: fail.txt' FAILED ****************...`。
- **L78 EN**: Comment documents nearby script behavior: `VERBOSE-NEXT: Exit Code: 127`.
  **L78 CN**: 注释说明了附近脚本逻辑：`VERBOSE-NEXT: Exit Code: 127`。
- **L79 EN**: Comment documents nearby script behavior: `VERBOSE-EMPTY:`.
  **L79 CN**: 注释说明了附近脚本逻辑：`VERBOSE-EMPTY:`。
- **L80 EN**: Comment documents nearby script behavior: `VERBOSE-NEXT: Command Output (stdout):`.
  **L80 CN**: 注释说明了附近脚本逻辑：`VERBOSE-NEXT: Command Output (stdout):`。
- **L81 EN**: Comment documents nearby script behavior: `VERBOSE-NEXT:`.
  **L81 CN**: 注释说明了附近脚本逻辑：`VERBOSE-NEXT:`。
- **L82 EN**: Comment documents nearby script behavior: `VERBOSE-NEXT: # {{R}}UN: at line 1`.
  **L82 CN**: 注释说明了附近脚本逻辑：`VERBOSE-NEXT: # {{R}}UN: at line 1`。
- **L83 EN**: Comment documents nearby script behavior: `VERBOSE-NEXT: echo "fail test output"`.
  **L83 CN**: 注释说明了附近脚本逻辑：`VERBOSE-NEXT: echo "fail test output"`。
- **L84 EN**: Comment documents nearby script behavior: `VERBOSE-NEXT: # executed command: echo 'fail test output'`.
  **L84 CN**: 注释说明了附近脚本逻辑：`VERBOSE-NEXT: # executed command: echo 'fail test output'`。
- **L85 EN**: Comment documents nearby script behavior: `VERBOSE-NEXT: # .---command stdout`.
  **L85 CN**: 注释说明了附近脚本逻辑：`VERBOSE-NEXT: # .---command stdout`。
- **L86 EN**: Comment documents nearby script behavior: `VERBOSE-NEXT: # | fail test output`.
  **L86 CN**: 注释说明了附近脚本逻辑：`VERBOSE-NEXT: # | fail test output`。
- **L87 EN**: Comment documents nearby script behavior: `VERBOSE-NEXT: # \``.
  **L87 CN**: 注释说明了附近脚本逻辑：`VERBOSE-NEXT: # \``。
- **L88 EN**: Comment documents nearby script behavior: `VERBOSE-NEXT: # {{R}}UN: at line 2`.
  **L88 CN**: 注释说明了附近脚本逻辑：`VERBOSE-NEXT: # {{R}}UN: at line 2`。
- **L89 EN**: Comment documents nearby script behavior: `VERBOSE-NEXT: fail`.
  **L89 CN**: 注释说明了附近脚本逻辑：`VERBOSE-NEXT: fail`。
- **L90 EN**: Comment documents nearby script behavior: `VERBOSE-NEXT: # executed command: fail`.
  **L90 CN**: 注释说明了附近脚本逻辑：`VERBOSE-NEXT: # executed command: fail`。
- **L91 EN**: Comment documents nearby script behavior: `VERBOSE-NEXT: # .---command stderr`.
  **L91 CN**: 注释说明了附近脚本逻辑：`VERBOSE-NEXT: # .---command stderr`。
- **L92 EN**: Comment documents nearby script behavior: `VERBOSE-NEXT: # | 'fail': command not found`.
  **L92 CN**: 注释说明了附近脚本逻辑：`VERBOSE-NEXT: # | 'fail': command not found`。
- **L93 EN**: Comment documents nearby script behavior: `VERBOSE-NEXT: # \``.
  **L93 CN**: 注释说明了附近脚本逻辑：`VERBOSE-NEXT: # \``。
- **L94 EN**: Comment documents nearby script behavior: `VERBOSE-NEXT: # error: command failed with exit status: 127`.
  **L94 CN**: 注释说明了附近脚本逻辑：`VERBOSE-NEXT: # error: command failed with exit status: 127`。
- **L95 EN**: Comment documents nearby script behavior: `VERBOSE-EMPTY:`.
  **L95 CN**: 注释说明了附近脚本逻辑：`VERBOSE-EMPTY:`。
- **L96 EN**: Comment documents nearby script behavior: `VERBOSE-NEXT:`.
  **L96 CN**: 注释说明了附近脚本逻辑：`VERBOSE-NEXT:`。
- **L97 EN**: Comment documents nearby script behavior: `VERBOSE-EMPTY:`.
  **L97 CN**: 注释说明了附近脚本逻辑：`VERBOSE-EMPTY:`。
- **L98 EN**: Comment documents nearby script behavior: `VERBOSE-NEXT: ********************`.
  **L98 CN**: 注释说明了附近脚本逻辑：`VERBOSE-NEXT: ********************`。

### Lines 99-126

````python
# VERBOSE-NEXT: PASS: verbosity :: pass.txt (2 of 5)
# VERBOSE-NEXT: {{UN}}SUPPORTED: verbosity :: unsupported.txt (3 of 5)
# VERBOSE-NEXT: {{X}}FAIL: verbosity :: xfail.txt (4 of 5)
# VERBOSE-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)
# VERBOSE-NEXT: ******************** TEST 'verbosity :: xpass.txt' FAILED ********************
# VERBOSE-NEXT: Exit Code: 0
# VERBOSE-EMPTY:
# VERBOSE-NEXT: Command Output (stdout):
# VERBOSE-NEXT: --
# VERBOSE-NEXT: # {{R}}UN: at line 2
# VERBOSE-NEXT: echo "xpass test output"
# VERBOSE-NEXT: # executed command: echo 'xpass test output'
# VERBOSE-NEXT: # .---command stdout------------
# VERBOSE-NEXT: # | xpass test output
# VERBOSE-NEXT: # `-----------------------------
# VERBOSE-EMPTY:
# VERBOSE-NEXT: --
# VERBOSE-EMPTY:
# VERBOSE-NEXT: ********************
# VERBOSE-NEXT: ********************
# VERBOSE-NEXT: Failed Tests (1):
# VERBOSE-NEXT:   verbosity :: fail.txt
# VERBOSE-EMPTY:
# VERBOSE-NEXT: ********************
# VERBOSE-NEXT: Unexpectedly Passed Tests (1):
# VERBOSE-NEXT:   verbosity :: xpass.txt
# VERBOSE-EMPTY:
# VERBOSE-EMPTY:
````
- **L99 EN**: Comment documents nearby script behavior: `VERBOSE-NEXT: PASS: verbosity :: pass.txt (2 of 5)`.
  **L99 CN**: 注释说明了附近脚本逻辑：`VERBOSE-NEXT: PASS: verbosity :: pass.txt (2 of 5)`。
- **L100 EN**: Comment documents nearby script behavior: `VERBOSE-NEXT: {{UN}}SUPPORTED: verbosity :: unsupported.txt (3 of 5)`.
  **L100 CN**: 注释说明了附近脚本逻辑：`VERBOSE-NEXT: {{UN}}SUPPORTED: verbosity :: unsupported.txt (3 of 5)`。
- **L101 EN**: Comment documents nearby script behavior: `VERBOSE-NEXT: {{X}}FAIL: verbosity :: xfail.txt (4 of 5)`.
  **L101 CN**: 注释说明了附近脚本逻辑：`VERBOSE-NEXT: {{X}}FAIL: verbosity :: xfail.txt (4 of 5)`。
- **L102 EN**: Comment documents nearby script behavior: `VERBOSE-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)`.
  **L102 CN**: 注释说明了附近脚本逻辑：`VERBOSE-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)`。
- **L103 EN**: Comment documents nearby script behavior: `VERBOSE-NEXT: ******************** TEST 'verbosity :: xpass.txt' FAILED ***************...`.
  **L103 CN**: 注释说明了附近脚本逻辑：`VERBOSE-NEXT: ******************** TEST 'verbosity :: xpass.txt' FAILED ***************...`。
- **L104 EN**: Comment documents nearby script behavior: `VERBOSE-NEXT: Exit Code: 0`.
  **L104 CN**: 注释说明了附近脚本逻辑：`VERBOSE-NEXT: Exit Code: 0`。
- **L105 EN**: Comment documents nearby script behavior: `VERBOSE-EMPTY:`.
  **L105 CN**: 注释说明了附近脚本逻辑：`VERBOSE-EMPTY:`。
- **L106 EN**: Comment documents nearby script behavior: `VERBOSE-NEXT: Command Output (stdout):`.
  **L106 CN**: 注释说明了附近脚本逻辑：`VERBOSE-NEXT: Command Output (stdout):`。
- **L107 EN**: Comment documents nearby script behavior: `VERBOSE-NEXT:`.
  **L107 CN**: 注释说明了附近脚本逻辑：`VERBOSE-NEXT:`。
- **L108 EN**: Comment documents nearby script behavior: `VERBOSE-NEXT: # {{R}}UN: at line 2`.
  **L108 CN**: 注释说明了附近脚本逻辑：`VERBOSE-NEXT: # {{R}}UN: at line 2`。
- **L109 EN**: Comment documents nearby script behavior: `VERBOSE-NEXT: echo "xpass test output"`.
  **L109 CN**: 注释说明了附近脚本逻辑：`VERBOSE-NEXT: echo "xpass test output"`。
- **L110 EN**: Comment documents nearby script behavior: `VERBOSE-NEXT: # executed command: echo 'xpass test output'`.
  **L110 CN**: 注释说明了附近脚本逻辑：`VERBOSE-NEXT: # executed command: echo 'xpass test output'`。
- **L111 EN**: Comment documents nearby script behavior: `VERBOSE-NEXT: # .---command stdout`.
  **L111 CN**: 注释说明了附近脚本逻辑：`VERBOSE-NEXT: # .---command stdout`。
- **L112 EN**: Comment documents nearby script behavior: `VERBOSE-NEXT: # | xpass test output`.
  **L112 CN**: 注释说明了附近脚本逻辑：`VERBOSE-NEXT: # | xpass test output`。
- **L113 EN**: Comment documents nearby script behavior: `VERBOSE-NEXT: # \``.
  **L113 CN**: 注释说明了附近脚本逻辑：`VERBOSE-NEXT: # \``。
- **L114 EN**: Comment documents nearby script behavior: `VERBOSE-EMPTY:`.
  **L114 CN**: 注释说明了附近脚本逻辑：`VERBOSE-EMPTY:`。
- **L115 EN**: Comment documents nearby script behavior: `VERBOSE-NEXT:`.
  **L115 CN**: 注释说明了附近脚本逻辑：`VERBOSE-NEXT:`。
- **L116 EN**: Comment documents nearby script behavior: `VERBOSE-EMPTY:`.
  **L116 CN**: 注释说明了附近脚本逻辑：`VERBOSE-EMPTY:`。
- **L117 EN**: Comment documents nearby script behavior: `VERBOSE-NEXT: ********************`.
  **L117 CN**: 注释说明了附近脚本逻辑：`VERBOSE-NEXT: ********************`。
- **L118 EN**: Comment documents nearby script behavior: `VERBOSE-NEXT: ********************`.
  **L118 CN**: 注释说明了附近脚本逻辑：`VERBOSE-NEXT: ********************`。
- **L119 EN**: Comment documents nearby script behavior: `VERBOSE-NEXT: Failed Tests (1):`.
  **L119 CN**: 注释说明了附近脚本逻辑：`VERBOSE-NEXT: Failed Tests (1):`。
- **L120 EN**: Comment documents nearby script behavior: `VERBOSE-NEXT: verbosity :: fail.txt`.
  **L120 CN**: 注释说明了附近脚本逻辑：`VERBOSE-NEXT: verbosity :: fail.txt`。
- **L121 EN**: Comment documents nearby script behavior: `VERBOSE-EMPTY:`.
  **L121 CN**: 注释说明了附近脚本逻辑：`VERBOSE-EMPTY:`。
- **L122 EN**: Comment documents nearby script behavior: `VERBOSE-NEXT: ********************`.
  **L122 CN**: 注释说明了附近脚本逻辑：`VERBOSE-NEXT: ********************`。
- **L123 EN**: Comment documents nearby script behavior: `VERBOSE-NEXT: Unexpectedly Passed Tests (1):`.
  **L123 CN**: 注释说明了附近脚本逻辑：`VERBOSE-NEXT: Unexpectedly Passed Tests (1):`。
- **L124 EN**: Comment documents nearby script behavior: `VERBOSE-NEXT: verbosity :: xpass.txt`.
  **L124 CN**: 注释说明了附近脚本逻辑：`VERBOSE-NEXT: verbosity :: xpass.txt`。
- **L125 EN**: Comment documents nearby script behavior: `VERBOSE-EMPTY:`.
  **L125 CN**: 注释说明了附近脚本逻辑：`VERBOSE-EMPTY:`。
- **L126 EN**: Comment documents nearby script behavior: `VERBOSE-EMPTY:`.
  **L126 CN**: 注释说明了附近脚本逻辑：`VERBOSE-EMPTY:`。

### Lines 127-154

````python
# VERBOSE-NEXT: Testing Time: {{.*}}s
# VERBOSE-EMPTY:
# VERBOSE-NEXT: Total Discovered Tests: 5
# VERBOSE-NEXT:   Unsupported        : 1 (20.00%)
# VERBOSE-NEXT:   Passed             : 1 (20.00%)
# VERBOSE-NEXT:   Expectedly Failed  : 1 (20.00%)
# VERBOSE-NEXT:   Failed             : 1 (20.00%)
# VERBOSE-NEXT:   Unexpectedly Passed: 1 (20.00%)

# RUN: not %{lit} --show-all %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt
# RUN: FileCheck %s --check-prefix SHOW-ALL < %t/stdout.txt
# RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt

# SHOW-ALL:      -- Testing: 5 tests, 1 workers --
# SHOW-ALL-NEXT: FAIL: verbosity :: fail.txt (1 of 5)
# SHOW-ALL-NEXT: ******************** TEST 'verbosity :: fail.txt' FAILED ********************
# SHOW-ALL-NEXT: Exit Code: 127
# SHOW-ALL-EMPTY:
# SHOW-ALL-NEXT: Command Output (stdout):
# SHOW-ALL-NEXT: --
# SHOW-ALL-NEXT: # {{R}}UN: at line 1
# SHOW-ALL-NEXT: echo "fail test output"
# SHOW-ALL-NEXT: # executed command: echo 'fail test output'
# SHOW-ALL-NEXT: # .---command stdout------------
# SHOW-ALL-NEXT: # | fail test output
# SHOW-ALL-NEXT: # `-----------------------------
# SHOW-ALL-NEXT: # {{R}}UN: at line 2
# SHOW-ALL-NEXT: fail
````
- **L127 EN**: Comment documents nearby script behavior: `VERBOSE-NEXT: Testing Time: {{.*}}s`.
  **L127 CN**: 注释说明了附近脚本逻辑：`VERBOSE-NEXT: Testing Time: {{.*}}s`。
- **L128 EN**: Comment documents nearby script behavior: `VERBOSE-EMPTY:`.
  **L128 CN**: 注释说明了附近脚本逻辑：`VERBOSE-EMPTY:`。
- **L129 EN**: Comment documents nearby script behavior: `VERBOSE-NEXT: Total Discovered Tests: 5`.
  **L129 CN**: 注释说明了附近脚本逻辑：`VERBOSE-NEXT: Total Discovered Tests: 5`。
- **L130 EN**: Comment documents nearby script behavior: `VERBOSE-NEXT: Unsupported : 1 (20.00%)`.
  **L130 CN**: 注释说明了附近脚本逻辑：`VERBOSE-NEXT: Unsupported : 1 (20.00%)`。
- **L131 EN**: Comment documents nearby script behavior: `VERBOSE-NEXT: Passed : 1 (20.00%)`.
  **L131 CN**: 注释说明了附近脚本逻辑：`VERBOSE-NEXT: Passed : 1 (20.00%)`。
- **L132 EN**: Comment documents nearby script behavior: `VERBOSE-NEXT: Expectedly Failed : 1 (20.00%)`.
  **L132 CN**: 注释说明了附近脚本逻辑：`VERBOSE-NEXT: Expectedly Failed : 1 (20.00%)`。
- **L133 EN**: Comment documents nearby script behavior: `VERBOSE-NEXT: Failed : 1 (20.00%)`.
  **L133 CN**: 注释说明了附近脚本逻辑：`VERBOSE-NEXT: Failed : 1 (20.00%)`。
- **L134 EN**: Comment documents nearby script behavior: `VERBOSE-NEXT: Unexpectedly Passed: 1 (20.00%)`.
  **L134 CN**: 注释说明了附近脚本逻辑：`VERBOSE-NEXT: Unexpectedly Passed: 1 (20.00%)`。
- **L135 EN**: Blank line separates nearby declarations or logic blocks.
  **L135 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L136 EN**: Comment documents nearby script behavior: `RUN: not %{lit} --show-all %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt`.
  **L136 CN**: 注释说明了附近脚本逻辑：`RUN: not %{lit} --show-all %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt`。
- **L137 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix SHOW-ALL < %t/stdout.txt`.
  **L137 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix SHOW-ALL < %t/stdout.txt`。
- **L138 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt`.
  **L138 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt`。
- **L139 EN**: Blank line separates nearby declarations or logic blocks.
  **L139 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L140 EN**: Comment documents nearby script behavior: `SHOW-ALL: -- Testing: 5 tests, 1 workers`.
  **L140 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL: -- Testing: 5 tests, 1 workers`。
- **L141 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: FAIL: verbosity :: fail.txt (1 of 5)`.
  **L141 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: FAIL: verbosity :: fail.txt (1 of 5)`。
- **L142 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: ******************** TEST 'verbosity :: fail.txt' FAILED ***************...`.
  **L142 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: ******************** TEST 'verbosity :: fail.txt' FAILED ***************...`。
- **L143 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: Exit Code: 127`.
  **L143 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: Exit Code: 127`。
- **L144 EN**: Comment documents nearby script behavior: `SHOW-ALL-EMPTY:`.
  **L144 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-EMPTY:`。
- **L145 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: Command Output (stdout):`.
  **L145 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: Command Output (stdout):`。
- **L146 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT:`.
  **L146 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT:`。
- **L147 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: # {{R}}UN: at line 1`.
  **L147 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: # {{R}}UN: at line 1`。
- **L148 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: echo "fail test output"`.
  **L148 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: echo "fail test output"`。
- **L149 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: # executed command: echo 'fail test output'`.
  **L149 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: # executed command: echo 'fail test output'`。
- **L150 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: # .---command stdout`.
  **L150 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: # .---command stdout`。
- **L151 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: # | fail test output`.
  **L151 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: # | fail test output`。
- **L152 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: # \``.
  **L152 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: # \``。
- **L153 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: # {{R}}UN: at line 2`.
  **L153 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: # {{R}}UN: at line 2`。
- **L154 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: fail`.
  **L154 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: fail`。

### Lines 155-182

````python
# SHOW-ALL-NEXT: # executed command: fail
# SHOW-ALL-NEXT: # .---command stderr------------
# SHOW-ALL-NEXT: # | 'fail': command not found
# SHOW-ALL-NEXT: # `-----------------------------
# SHOW-ALL-NEXT: # error: command failed with exit status: 127
# SHOW-ALL-EMPTY:
# SHOW-ALL-NEXT: --
# SHOW-ALL-EMPTY:
# SHOW-ALL-NEXT: ********************
# SHOW-ALL-NEXT: PASS: verbosity :: pass.txt (2 of 5)
# SHOW-ALL-NEXT: Exit Code: 0
# SHOW-ALL-EMPTY:
# SHOW-ALL-NEXT: Command Output (stdout):
# SHOW-ALL-NEXT: --
# SHOW-ALL-NEXT: # {{R}}UN: at line 1
# SHOW-ALL-NEXT: echo "pass test output"
# SHOW-ALL-NEXT: # executed command: echo 'pass test output'
# SHOW-ALL-NEXT: # .---command stdout------------
# SHOW-ALL-NEXT: # | pass test output
# SHOW-ALL-NEXT: # `-----------------------------
# SHOW-ALL-EMPTY:
# SHOW-ALL-NEXT: --
# SHOW-ALL-EMPTY:
# SHOW-ALL-NEXT: ********************
# SHOW-ALL-NEXT: {{UN}}SUPPORTED: verbosity :: unsupported.txt (3 of 5)
# SHOW-ALL-NEXT: Test requires the following unavailable features: asdf
# SHOW-ALL-NEXT: ********************
# SHOW-ALL-NEXT: {{X}}FAIL: verbosity :: xfail.txt (4 of 5)
````
- **L155 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: # executed command: fail`.
  **L155 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: # executed command: fail`。
- **L156 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: # .---command stderr`.
  **L156 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: # .---command stderr`。
- **L157 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: # | 'fail': command not found`.
  **L157 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: # | 'fail': command not found`。
- **L158 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: # \``.
  **L158 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: # \``。
- **L159 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: # error: command failed with exit status: 127`.
  **L159 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: # error: command failed with exit status: 127`。
- **L160 EN**: Comment documents nearby script behavior: `SHOW-ALL-EMPTY:`.
  **L160 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-EMPTY:`。
- **L161 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT:`.
  **L161 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT:`。
- **L162 EN**: Comment documents nearby script behavior: `SHOW-ALL-EMPTY:`.
  **L162 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-EMPTY:`。
- **L163 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: ********************`.
  **L163 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: ********************`。
- **L164 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: PASS: verbosity :: pass.txt (2 of 5)`.
  **L164 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: PASS: verbosity :: pass.txt (2 of 5)`。
- **L165 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: Exit Code: 0`.
  **L165 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: Exit Code: 0`。
- **L166 EN**: Comment documents nearby script behavior: `SHOW-ALL-EMPTY:`.
  **L166 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-EMPTY:`。
- **L167 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: Command Output (stdout):`.
  **L167 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: Command Output (stdout):`。
- **L168 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT:`.
  **L168 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT:`。
- **L169 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: # {{R}}UN: at line 1`.
  **L169 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: # {{R}}UN: at line 1`。
- **L170 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: echo "pass test output"`.
  **L170 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: echo "pass test output"`。
- **L171 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: # executed command: echo 'pass test output'`.
  **L171 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: # executed command: echo 'pass test output'`。
- **L172 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: # .---command stdout`.
  **L172 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: # .---command stdout`。
- **L173 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: # | pass test output`.
  **L173 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: # | pass test output`。
- **L174 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: # \``.
  **L174 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: # \``。
- **L175 EN**: Comment documents nearby script behavior: `SHOW-ALL-EMPTY:`.
  **L175 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-EMPTY:`。
- **L176 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT:`.
  **L176 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT:`。
- **L177 EN**: Comment documents nearby script behavior: `SHOW-ALL-EMPTY:`.
  **L177 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-EMPTY:`。
- **L178 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: ********************`.
  **L178 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: ********************`。
- **L179 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: {{UN}}SUPPORTED: verbosity :: unsupported.txt (3 of 5)`.
  **L179 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: {{UN}}SUPPORTED: verbosity :: unsupported.txt (3 of 5)`。
- **L180 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: Test requires the following unavailable features: asdf`.
  **L180 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: Test requires the following unavailable features: asdf`。
- **L181 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: ********************`.
  **L181 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: ********************`。
- **L182 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: {{X}}FAIL: verbosity :: xfail.txt (4 of 5)`.
  **L182 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: {{X}}FAIL: verbosity :: xfail.txt (4 of 5)`。

### Lines 183-210

````python
# SHOW-ALL-NEXT: Exit Code: 1
# SHOW-ALL-EMPTY:
# SHOW-ALL-NEXT: Command Output (stdout):
# SHOW-ALL-NEXT: --
# SHOW-ALL-NEXT: # {{R}}UN: at line 2
# SHOW-ALL-NEXT: not echo "xfail test output"
# SHOW-ALL-NEXT: # executed command: not echo 'xfail test output'
# SHOW-ALL-NEXT: # .---command stdout------------
# SHOW-ALL-NEXT: # | xfail test output
# SHOW-ALL-NEXT: # `-----------------------------
# SHOW-ALL-NEXT: # error: command failed with exit status: 1
# SHOW-ALL-EMPTY:
# SHOW-ALL-NEXT: --
# SHOW-ALL-EMPTY:
# SHOW-ALL-NEXT: ********************
# SHOW-ALL-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)
# SHOW-ALL-NEXT: ******************** TEST 'verbosity :: xpass.txt' FAILED ********************
# SHOW-ALL-NEXT: Exit Code: 0
# SHOW-ALL-EMPTY:
# SHOW-ALL-NEXT: Command Output (stdout):
# SHOW-ALL-NEXT: --
# SHOW-ALL-NEXT: # {{R}}UN: at line 2
# SHOW-ALL-NEXT: echo "xpass test output"
# SHOW-ALL-NEXT: # executed command: echo 'xpass test output'
# SHOW-ALL-NEXT: # .---command stdout------------
# SHOW-ALL-NEXT: # | xpass test output
# SHOW-ALL-NEXT: # `-----------------------------
# SHOW-ALL-EMPTY:
````
- **L183 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: Exit Code: 1`.
  **L183 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: Exit Code: 1`。
- **L184 EN**: Comment documents nearby script behavior: `SHOW-ALL-EMPTY:`.
  **L184 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-EMPTY:`。
- **L185 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: Command Output (stdout):`.
  **L185 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: Command Output (stdout):`。
- **L186 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT:`.
  **L186 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT:`。
- **L187 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: # {{R}}UN: at line 2`.
  **L187 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: # {{R}}UN: at line 2`。
- **L188 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: not echo "xfail test output"`.
  **L188 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: not echo "xfail test output"`。
- **L189 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: # executed command: not echo 'xfail test output'`.
  **L189 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: # executed command: not echo 'xfail test output'`。
- **L190 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: # .---command stdout`.
  **L190 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: # .---command stdout`。
- **L191 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: # | xfail test output`.
  **L191 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: # | xfail test output`。
- **L192 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: # \``.
  **L192 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: # \``。
- **L193 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: # error: command failed with exit status: 1`.
  **L193 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: # error: command failed with exit status: 1`。
- **L194 EN**: Comment documents nearby script behavior: `SHOW-ALL-EMPTY:`.
  **L194 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-EMPTY:`。
- **L195 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT:`.
  **L195 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT:`。
- **L196 EN**: Comment documents nearby script behavior: `SHOW-ALL-EMPTY:`.
  **L196 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-EMPTY:`。
- **L197 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: ********************`.
  **L197 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: ********************`。
- **L198 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)`.
  **L198 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)`。
- **L199 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: ******************** TEST 'verbosity :: xpass.txt' FAILED **************...`.
  **L199 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: ******************** TEST 'verbosity :: xpass.txt' FAILED **************...`。
- **L200 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: Exit Code: 0`.
  **L200 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: Exit Code: 0`。
- **L201 EN**: Comment documents nearby script behavior: `SHOW-ALL-EMPTY:`.
  **L201 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-EMPTY:`。
- **L202 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: Command Output (stdout):`.
  **L202 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: Command Output (stdout):`。
- **L203 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT:`.
  **L203 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT:`。
- **L204 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: # {{R}}UN: at line 2`.
  **L204 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: # {{R}}UN: at line 2`。
- **L205 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: echo "xpass test output"`.
  **L205 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: echo "xpass test output"`。
- **L206 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: # executed command: echo 'xpass test output'`.
  **L206 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: # executed command: echo 'xpass test output'`。
- **L207 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: # .---command stdout`.
  **L207 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: # .---command stdout`。
- **L208 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: # | xpass test output`.
  **L208 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: # | xpass test output`。
- **L209 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: # \``.
  **L209 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: # \``。
- **L210 EN**: Comment documents nearby script behavior: `SHOW-ALL-EMPTY:`.
  **L210 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-EMPTY:`。

### Lines 211-231

````python
# SHOW-ALL-NEXT: --
# SHOW-ALL-EMPTY:
# SHOW-ALL-NEXT: ********************
# SHOW-ALL-NEXT: ********************
# SHOW-ALL-NEXT: Failed Tests (1):
# SHOW-ALL-NEXT:   verbosity :: fail.txt
# SHOW-ALL-EMPTY:
# SHOW-ALL-NEXT: ********************
# SHOW-ALL-NEXT: Unexpectedly Passed Tests (1):
# SHOW-ALL-NEXT:   verbosity :: xpass.txt
# SHOW-ALL-EMPTY:
# SHOW-ALL-EMPTY:
# SHOW-ALL-NEXT: Testing Time: {{.*}}s
# SHOW-ALL-EMPTY:
# SHOW-ALL-NEXT: Total Discovered Tests: 5
# SHOW-ALL-NEXT:   Unsupported        : 1 (20.00%)
# SHOW-ALL-NEXT:   Passed             : 1 (20.00%)
# SHOW-ALL-NEXT:   Expectedly Failed  : 1 (20.00%)
# SHOW-ALL-NEXT:   Failed             : 1 (20.00%)
# SHOW-ALL-NEXT:   Unexpectedly Passed: 1 (20.00%)

````
- **L211 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT:`.
  **L211 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT:`。
- **L212 EN**: Comment documents nearby script behavior: `SHOW-ALL-EMPTY:`.
  **L212 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-EMPTY:`。
- **L213 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: ********************`.
  **L213 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: ********************`。
- **L214 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: ********************`.
  **L214 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: ********************`。
- **L215 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: Failed Tests (1):`.
  **L215 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: Failed Tests (1):`。
- **L216 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: verbosity :: fail.txt`.
  **L216 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: verbosity :: fail.txt`。
- **L217 EN**: Comment documents nearby script behavior: `SHOW-ALL-EMPTY:`.
  **L217 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-EMPTY:`。
- **L218 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: ********************`.
  **L218 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: ********************`。
- **L219 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: Unexpectedly Passed Tests (1):`.
  **L219 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: Unexpectedly Passed Tests (1):`。
- **L220 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: verbosity :: xpass.txt`.
  **L220 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: verbosity :: xpass.txt`。
- **L221 EN**: Comment documents nearby script behavior: `SHOW-ALL-EMPTY:`.
  **L221 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-EMPTY:`。
- **L222 EN**: Comment documents nearby script behavior: `SHOW-ALL-EMPTY:`.
  **L222 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-EMPTY:`。
- **L223 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: Testing Time: {{.*}}s`.
  **L223 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: Testing Time: {{.*}}s`。
- **L224 EN**: Comment documents nearby script behavior: `SHOW-ALL-EMPTY:`.
  **L224 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-EMPTY:`。
- **L225 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: Total Discovered Tests: 5`.
  **L225 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: Total Discovered Tests: 5`。
- **L226 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: Unsupported : 1 (20.00%)`.
  **L226 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: Unsupported : 1 (20.00%)`。
- **L227 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: Passed : 1 (20.00%)`.
  **L227 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: Passed : 1 (20.00%)`。
- **L228 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: Expectedly Failed : 1 (20.00%)`.
  **L228 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: Expectedly Failed : 1 (20.00%)`。
- **L229 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: Failed : 1 (20.00%)`.
  **L229 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: Failed : 1 (20.00%)`。
- **L230 EN**: Comment documents nearby script behavior: `SHOW-ALL-NEXT: Unexpectedly Passed: 1 (20.00%)`.
  **L230 CN**: 注释说明了附近脚本逻辑：`SHOW-ALL-NEXT: Unexpectedly Passed: 1 (20.00%)`。
- **L231 EN**: Blank line separates nearby declarations or logic blocks.
  **L231 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 232-251

````python
# RUN: not %{lit} --quiet %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt
# RUN: FileCheck %s --check-prefix QUIET < %t/stdout.txt
# RUN: FileCheck %s --check-prefix QUIET-ERR --implicit-check-not lit < %t/stderr.txt

# QUIET:      -- Testing: 5 tests, 1 workers --
# QUIET-NEXT: FAIL: verbosity :: fail.txt (1 of 5)
# QUIET-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)
# QUIET-NEXT: ********************
# QUIET-NEXT: Failed Tests (1):
# QUIET-NEXT:   verbosity :: fail.txt
# QUIET-EMPTY:
# QUIET-NEXT: ********************
# QUIET-NEXT: Unexpectedly Passed Tests (1):
# QUIET-NEXT:   verbosity :: xpass.txt
# QUIET-EMPTY:
# QUIET-EMPTY:
# QUIET-NEXT: Total Discovered Tests: 5
# QUIET-NEXT:   Failed             : 1 (20.00%)
# QUIET-NEXT:   Unexpectedly Passed: 1 (20.00%)

````
- **L232 EN**: Comment documents nearby script behavior: `RUN: not %{lit} --quiet %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt`.
  **L232 CN**: 注释说明了附近脚本逻辑：`RUN: not %{lit} --quiet %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt`。
- **L233 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix QUIET < %t/stdout.txt`.
  **L233 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix QUIET < %t/stdout.txt`。
- **L234 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix QUIET-ERR --implicit-check-not lit < %t/stderr.txt`.
  **L234 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix QUIET-ERR --implicit-check-not lit < %t/stderr.txt`。
- **L235 EN**: Blank line separates nearby declarations or logic blocks.
  **L235 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L236 EN**: Comment documents nearby script behavior: `QUIET: -- Testing: 5 tests, 1 workers`.
  **L236 CN**: 注释说明了附近脚本逻辑：`QUIET: -- Testing: 5 tests, 1 workers`。
- **L237 EN**: Comment documents nearby script behavior: `QUIET-NEXT: FAIL: verbosity :: fail.txt (1 of 5)`.
  **L237 CN**: 注释说明了附近脚本逻辑：`QUIET-NEXT: FAIL: verbosity :: fail.txt (1 of 5)`。
- **L238 EN**: Comment documents nearby script behavior: `QUIET-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)`.
  **L238 CN**: 注释说明了附近脚本逻辑：`QUIET-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)`。
- **L239 EN**: Comment documents nearby script behavior: `QUIET-NEXT: ********************`.
  **L239 CN**: 注释说明了附近脚本逻辑：`QUIET-NEXT: ********************`。
- **L240 EN**: Comment documents nearby script behavior: `QUIET-NEXT: Failed Tests (1):`.
  **L240 CN**: 注释说明了附近脚本逻辑：`QUIET-NEXT: Failed Tests (1):`。
- **L241 EN**: Comment documents nearby script behavior: `QUIET-NEXT: verbosity :: fail.txt`.
  **L241 CN**: 注释说明了附近脚本逻辑：`QUIET-NEXT: verbosity :: fail.txt`。
- **L242 EN**: Comment documents nearby script behavior: `QUIET-EMPTY:`.
  **L242 CN**: 注释说明了附近脚本逻辑：`QUIET-EMPTY:`。
- **L243 EN**: Comment documents nearby script behavior: `QUIET-NEXT: ********************`.
  **L243 CN**: 注释说明了附近脚本逻辑：`QUIET-NEXT: ********************`。
- **L244 EN**: Comment documents nearby script behavior: `QUIET-NEXT: Unexpectedly Passed Tests (1):`.
  **L244 CN**: 注释说明了附近脚本逻辑：`QUIET-NEXT: Unexpectedly Passed Tests (1):`。
- **L245 EN**: Comment documents nearby script behavior: `QUIET-NEXT: verbosity :: xpass.txt`.
  **L245 CN**: 注释说明了附近脚本逻辑：`QUIET-NEXT: verbosity :: xpass.txt`。
- **L246 EN**: Comment documents nearby script behavior: `QUIET-EMPTY:`.
  **L246 CN**: 注释说明了附近脚本逻辑：`QUIET-EMPTY:`。
- **L247 EN**: Comment documents nearby script behavior: `QUIET-EMPTY:`.
  **L247 CN**: 注释说明了附近脚本逻辑：`QUIET-EMPTY:`。
- **L248 EN**: Comment documents nearby script behavior: `QUIET-NEXT: Total Discovered Tests: 5`.
  **L248 CN**: 注释说明了附近脚本逻辑：`QUIET-NEXT: Total Discovered Tests: 5`。
- **L249 EN**: Comment documents nearby script behavior: `QUIET-NEXT: Failed : 1 (20.00%)`.
  **L249 CN**: 注释说明了附近脚本逻辑：`QUIET-NEXT: Failed : 1 (20.00%)`。
- **L250 EN**: Comment documents nearby script behavior: `QUIET-NEXT: Unexpectedly Passed: 1 (20.00%)`.
  **L250 CN**: 注释说明了附近脚本逻辑：`QUIET-NEXT: Unexpectedly Passed: 1 (20.00%)`。
- **L251 EN**: Blank line separates nearby declarations or logic blocks.
  **L251 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 252-279

````python
# QUIET-ERR: 1 warning(s) in tests


### Test log output

# RUN: not %{lit} --debug %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt
# RUN: FileCheck %s --check-prefix DEBUG < %t/stdout.txt
# RUN: FileCheck %s --check-prefix DEBUG-ERR --implicit-check-not lit < %t/stderr.txt

# DEBUG:      -- Testing: 5 tests, 1 workers --
# DEBUG-NEXT: FAIL: verbosity :: fail.txt (1 of 5)
# DEBUG-NEXT: PASS: verbosity :: pass.txt (2 of 5)
# DEBUG-NEXT: {{UN}}SUPPORTED: verbosity :: unsupported.txt (3 of 5)
# DEBUG-NEXT: {{X}}FAIL: verbosity :: xfail.txt (4 of 5)
# DEBUG-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)
# DEBUG-NEXT: ********************
# DEBUG-NEXT: Failed Tests (1):
# DEBUG-NEXT:   verbosity :: fail.txt
# DEBUG-EMPTY:
# DEBUG-NEXT: ********************
# DEBUG-NEXT: Unexpectedly Passed Tests (1):
# DEBUG-NEXT:   verbosity :: xpass.txt
# DEBUG-EMPTY:
# DEBUG-EMPTY:
# DEBUG-NEXT: Testing Time: {{.*}}s
# DEBUG-EMPTY:
# DEBUG-NEXT: Total Discovered Tests: 5
# DEBUG-NEXT:   Unsupported        : 1 (20.00%)
````
- **L252 EN**: Comment documents nearby script behavior: `QUIET-ERR: 1 warning(s) in tests`.
  **L252 CN**: 注释说明了附近脚本逻辑：`QUIET-ERR: 1 warning(s) in tests`。
- **L253 EN**: Blank line separates nearby declarations or logic blocks.
  **L253 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L254 EN**: Blank line separates nearby declarations or logic blocks.
  **L254 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L255 EN**: Comment documents nearby script behavior: `## Test log output`.
  **L255 CN**: 注释说明了附近脚本逻辑：`## Test log output`。
- **L256 EN**: Blank line separates nearby declarations or logic blocks.
  **L256 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L257 EN**: Comment documents nearby script behavior: `RUN: not %{lit} --debug %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt`.
  **L257 CN**: 注释说明了附近脚本逻辑：`RUN: not %{lit} --debug %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt`。
- **L258 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix DEBUG < %t/stdout.txt`.
  **L258 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix DEBUG < %t/stdout.txt`。
- **L259 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix DEBUG-ERR --implicit-check-not lit < %t/stderr.txt`.
  **L259 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix DEBUG-ERR --implicit-check-not lit < %t/stderr.txt`。
- **L260 EN**: Blank line separates nearby declarations or logic blocks.
  **L260 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L261 EN**: Comment documents nearby script behavior: `DEBUG: -- Testing: 5 tests, 1 workers`.
  **L261 CN**: 注释说明了附近脚本逻辑：`DEBUG: -- Testing: 5 tests, 1 workers`。
- **L262 EN**: Comment documents nearby script behavior: `DEBUG-NEXT: FAIL: verbosity :: fail.txt (1 of 5)`.
  **L262 CN**: 注释说明了附近脚本逻辑：`DEBUG-NEXT: FAIL: verbosity :: fail.txt (1 of 5)`。
- **L263 EN**: Comment documents nearby script behavior: `DEBUG-NEXT: PASS: verbosity :: pass.txt (2 of 5)`.
  **L263 CN**: 注释说明了附近脚本逻辑：`DEBUG-NEXT: PASS: verbosity :: pass.txt (2 of 5)`。
- **L264 EN**: Comment documents nearby script behavior: `DEBUG-NEXT: {{UN}}SUPPORTED: verbosity :: unsupported.txt (3 of 5)`.
  **L264 CN**: 注释说明了附近脚本逻辑：`DEBUG-NEXT: {{UN}}SUPPORTED: verbosity :: unsupported.txt (3 of 5)`。
- **L265 EN**: Comment documents nearby script behavior: `DEBUG-NEXT: {{X}}FAIL: verbosity :: xfail.txt (4 of 5)`.
  **L265 CN**: 注释说明了附近脚本逻辑：`DEBUG-NEXT: {{X}}FAIL: verbosity :: xfail.txt (4 of 5)`。
- **L266 EN**: Comment documents nearby script behavior: `DEBUG-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)`.
  **L266 CN**: 注释说明了附近脚本逻辑：`DEBUG-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)`。
- **L267 EN**: Comment documents nearby script behavior: `DEBUG-NEXT: ********************`.
  **L267 CN**: 注释说明了附近脚本逻辑：`DEBUG-NEXT: ********************`。
- **L268 EN**: Comment documents nearby script behavior: `DEBUG-NEXT: Failed Tests (1):`.
  **L268 CN**: 注释说明了附近脚本逻辑：`DEBUG-NEXT: Failed Tests (1):`。
- **L269 EN**: Comment documents nearby script behavior: `DEBUG-NEXT: verbosity :: fail.txt`.
  **L269 CN**: 注释说明了附近脚本逻辑：`DEBUG-NEXT: verbosity :: fail.txt`。
- **L270 EN**: Comment documents nearby script behavior: `DEBUG-EMPTY:`.
  **L270 CN**: 注释说明了附近脚本逻辑：`DEBUG-EMPTY:`。
- **L271 EN**: Comment documents nearby script behavior: `DEBUG-NEXT: ********************`.
  **L271 CN**: 注释说明了附近脚本逻辑：`DEBUG-NEXT: ********************`。
- **L272 EN**: Comment documents nearby script behavior: `DEBUG-NEXT: Unexpectedly Passed Tests (1):`.
  **L272 CN**: 注释说明了附近脚本逻辑：`DEBUG-NEXT: Unexpectedly Passed Tests (1):`。
- **L273 EN**: Comment documents nearby script behavior: `DEBUG-NEXT: verbosity :: xpass.txt`.
  **L273 CN**: 注释说明了附近脚本逻辑：`DEBUG-NEXT: verbosity :: xpass.txt`。
- **L274 EN**: Comment documents nearby script behavior: `DEBUG-EMPTY:`.
  **L274 CN**: 注释说明了附近脚本逻辑：`DEBUG-EMPTY:`。
- **L275 EN**: Comment documents nearby script behavior: `DEBUG-EMPTY:`.
  **L275 CN**: 注释说明了附近脚本逻辑：`DEBUG-EMPTY:`。
- **L276 EN**: Comment documents nearby script behavior: `DEBUG-NEXT: Testing Time: {{.*}}s`.
  **L276 CN**: 注释说明了附近脚本逻辑：`DEBUG-NEXT: Testing Time: {{.*}}s`。
- **L277 EN**: Comment documents nearby script behavior: `DEBUG-EMPTY:`.
  **L277 CN**: 注释说明了附近脚本逻辑：`DEBUG-EMPTY:`。
- **L278 EN**: Comment documents nearby script behavior: `DEBUG-NEXT: Total Discovered Tests: 5`.
  **L278 CN**: 注释说明了附近脚本逻辑：`DEBUG-NEXT: Total Discovered Tests: 5`。
- **L279 EN**: Comment documents nearby script behavior: `DEBUG-NEXT: Unsupported : 1 (20.00%)`.
  **L279 CN**: 注释说明了附近脚本逻辑：`DEBUG-NEXT: Unsupported : 1 (20.00%)`。

### Lines 280-293

````python
# DEBUG-NEXT:   Passed             : 1 (20.00%)
# DEBUG-NEXT:   Expectedly Failed  : 1 (20.00%)
# DEBUG-NEXT:   Failed             : 1 (20.00%)
# DEBUG-NEXT:   Unexpectedly Passed: 1 (20.00%)

# DEBUG-ERR:      lit.py: {{.*}}discovery.py:{{[0-9]+}}: debug: loading suite config '{{.*}}lit.cfg'
# DEBUG-ERR-NEXT: lit.py: {{.*}}lit.cfg:{{[0-9]+}}: debug: this is a debug log
# DEBUG-ERR-NEXT: lit.py: {{.*}}lit.cfg:{{[0-9]+}}: note: this is a note
# DEBUG-ERR-NEXT: lit.py: {{.*}}lit.cfg:{{[0-9]+}}: warning: this is a warning
# DEBUG-ERR-NEXT: lit.py: {{.*}}TestingConfig.py:{{[0-9]+}}: debug: ... loaded config '{{.*}}lit.cfg'
# DEBUG-ERR-NEXT: lit.py: {{.*}}discovery.py:{{[0-9]+}}: debug: resolved input '{{.*}}verbosity' to 'verbosity'::()
# DEBUG-ERR-EMPTY:
# DEBUG-ERR-NEXT: 1 warning(s) in tests

````
- **L280 EN**: Comment documents nearby script behavior: `DEBUG-NEXT: Passed : 1 (20.00%)`.
  **L280 CN**: 注释说明了附近脚本逻辑：`DEBUG-NEXT: Passed : 1 (20.00%)`。
- **L281 EN**: Comment documents nearby script behavior: `DEBUG-NEXT: Expectedly Failed : 1 (20.00%)`.
  **L281 CN**: 注释说明了附近脚本逻辑：`DEBUG-NEXT: Expectedly Failed : 1 (20.00%)`。
- **L282 EN**: Comment documents nearby script behavior: `DEBUG-NEXT: Failed : 1 (20.00%)`.
  **L282 CN**: 注释说明了附近脚本逻辑：`DEBUG-NEXT: Failed : 1 (20.00%)`。
- **L283 EN**: Comment documents nearby script behavior: `DEBUG-NEXT: Unexpectedly Passed: 1 (20.00%)`.
  **L283 CN**: 注释说明了附近脚本逻辑：`DEBUG-NEXT: Unexpectedly Passed: 1 (20.00%)`。
- **L284 EN**: Blank line separates nearby declarations or logic blocks.
  **L284 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L285 EN**: Comment documents nearby script behavior: `DEBUG-ERR: lit.py: {{.*}}discovery.py:{{[0-9]+}}: debug: loading suite config '{{.*}}li...`.
  **L285 CN**: 注释说明了附近脚本逻辑：`DEBUG-ERR: lit.py: {{.*}}discovery.py:{{[0-9]+}}: debug: loading suite config '{{.*}}li...`。
- **L286 EN**: Comment documents nearby script behavior: `DEBUG-ERR-NEXT: lit.py: {{.*}}lit.cfg:{{[0-9]+}}: debug: this is a debug log`.
  **L286 CN**: 注释说明了附近脚本逻辑：`DEBUG-ERR-NEXT: lit.py: {{.*}}lit.cfg:{{[0-9]+}}: debug: this is a debug log`。
- **L287 EN**: Comment documents nearby script behavior: `DEBUG-ERR-NEXT: lit.py: {{.*}}lit.cfg:{{[0-9]+}}: note: this is a note`.
  **L287 CN**: 注释说明了附近脚本逻辑：`DEBUG-ERR-NEXT: lit.py: {{.*}}lit.cfg:{{[0-9]+}}: note: this is a note`。
- **L288 EN**: Comment documents nearby script behavior: `DEBUG-ERR-NEXT: lit.py: {{.*}}lit.cfg:{{[0-9]+}}: warning: this is a warning`.
  **L288 CN**: 注释说明了附近脚本逻辑：`DEBUG-ERR-NEXT: lit.py: {{.*}}lit.cfg:{{[0-9]+}}: warning: this is a warning`。
- **L289 EN**: Comment documents nearby script behavior: `DEBUG-ERR-NEXT: lit.py: {{.*}}TestingConfig.py:{{[0-9]+}}: debug: ... loaded config '{{...`.
  **L289 CN**: 注释说明了附近脚本逻辑：`DEBUG-ERR-NEXT: lit.py: {{.*}}TestingConfig.py:{{[0-9]+}}: debug: ... loaded config '{{...`。
- **L290 EN**: Comment documents nearby script behavior: `DEBUG-ERR-NEXT: lit.py: {{.*}}discovery.py:{{[0-9]+}}: debug: resolved input '{{.*}}ver...`.
  **L290 CN**: 注释说明了附近脚本逻辑：`DEBUG-ERR-NEXT: lit.py: {{.*}}discovery.py:{{[0-9]+}}: debug: resolved input '{{.*}}ver...`。
- **L291 EN**: Comment documents nearby script behavior: `DEBUG-ERR-EMPTY:`.
  **L291 CN**: 注释说明了附近脚本逻辑：`DEBUG-ERR-EMPTY:`。
- **L292 EN**: Comment documents nearby script behavior: `DEBUG-ERR-NEXT: 1 warning(s) in tests`.
  **L292 CN**: 注释说明了附近脚本逻辑：`DEBUG-ERR-NEXT: 1 warning(s) in tests`。
- **L293 EN**: Blank line separates nearby declarations or logic blocks.
  **L293 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 294-310

````python

# RUN: not %{lit} --diagnostic-level note %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt
# RUN: FileCheck %s --check-prefix NO-ARGS < %t/stdout.txt
# RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt

# RUN: not %{lit} --diagnostic-level warning %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt
# RUN: FileCheck %s --check-prefix NO-ARGS < %t/stdout.txt
# RUN: FileCheck %s --check-prefix WARNING-ERR --implicit-check-not lit < %t/stderr.txt

# WARNING-ERR: lit.py: {{.*}}lit.cfg:{{[0-9]+}}: warning: this is a warning
# WARNING-ERR-EMPTY:
# WARNING-ERR-NEXT: 1 warning(s) in tests

# RUN: not %{lit} --diagnostic-level error %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt
# RUN: FileCheck %s --check-prefix NO-ARGS < %t/stdout.txt
# RUN: FileCheck %s --check-prefix ERROR-ERR --implicit-check-not lit < %t/stderr.txt

````
- **L294 EN**: Blank line separates nearby declarations or logic blocks.
  **L294 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L295 EN**: Comment documents nearby script behavior: `RUN: not %{lit} --diagnostic-level note %{inputs}/verbosity 2> %t/stderr.txt > %t/stdou...`.
  **L295 CN**: 注释说明了附近脚本逻辑：`RUN: not %{lit} --diagnostic-level note %{inputs}/verbosity 2> %t/stderr.txt > %t/stdou...`。
- **L296 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix NO-ARGS < %t/stdout.txt`.
  **L296 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix NO-ARGS < %t/stdout.txt`。
- **L297 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt`.
  **L297 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt`。
- **L298 EN**: Blank line separates nearby declarations or logic blocks.
  **L298 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L299 EN**: Comment documents nearby script behavior: `RUN: not %{lit} --diagnostic-level warning %{inputs}/verbosity 2> %t/stderr.txt > %t/st...`.
  **L299 CN**: 注释说明了附近脚本逻辑：`RUN: not %{lit} --diagnostic-level warning %{inputs}/verbosity 2> %t/stderr.txt > %t/st...`。
- **L300 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix NO-ARGS < %t/stdout.txt`.
  **L300 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix NO-ARGS < %t/stdout.txt`。
- **L301 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix WARNING-ERR --implicit-check-not lit < %t/stderr.txt`.
  **L301 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix WARNING-ERR --implicit-check-not lit < %t/stderr.txt`。
- **L302 EN**: Blank line separates nearby declarations or logic blocks.
  **L302 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L303 EN**: Comment documents nearby script behavior: `WARNING-ERR: lit.py: {{.*}}lit.cfg:{{[0-9]+}}: warning: this is a warning`.
  **L303 CN**: 注释说明了附近脚本逻辑：`WARNING-ERR: lit.py: {{.*}}lit.cfg:{{[0-9]+}}: warning: this is a warning`。
- **L304 EN**: Comment documents nearby script behavior: `WARNING-ERR-EMPTY:`.
  **L304 CN**: 注释说明了附近脚本逻辑：`WARNING-ERR-EMPTY:`。
- **L305 EN**: Comment documents nearby script behavior: `WARNING-ERR-NEXT: 1 warning(s) in tests`.
  **L305 CN**: 注释说明了附近脚本逻辑：`WARNING-ERR-NEXT: 1 warning(s) in tests`。
- **L306 EN**: Blank line separates nearby declarations or logic blocks.
  **L306 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L307 EN**: Comment documents nearby script behavior: `RUN: not %{lit} --diagnostic-level error %{inputs}/verbosity 2> %t/stderr.txt > %t/stdo...`.
  **L307 CN**: 注释说明了附近脚本逻辑：`RUN: not %{lit} --diagnostic-level error %{inputs}/verbosity 2> %t/stderr.txt > %t/stdo...`。
- **L308 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix NO-ARGS < %t/stdout.txt`.
  **L308 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix NO-ARGS < %t/stdout.txt`。
- **L309 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix ERROR-ERR --implicit-check-not lit < %t/stderr.txt`.
  **L309 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix ERROR-ERR --implicit-check-not lit < %t/stderr.txt`。
- **L310 EN**: Blank line separates nearby declarations or logic blocks.
  **L310 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 311-338

````python
# ERROR-ERR: 1 warning(s) in tests


### Test --test-output

# RUN: not %{lit} --test-output off  %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt
# RUN: FileCheck %s --check-prefix NO-ARGS < %t/stdout.txt
# RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt

# RUN: not %{lit} --test-output failed  %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt
# RUN: FileCheck %s --check-prefix VERBOSE < %t/stdout.txt
# RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt

# TEST-OUTPUT-OFF:      -- Testing: 5 tests, 1 workers --
# TEST-OUTPUT-OFF-NEXT: FAIL: verbosity :: fail.txt (1 of 5)
# TEST-OUTPUT-OFF-NEXT: PASS: verbosity :: pass.txt (2 of 5)
# TEST-OUTPUT-OFF-NEXT: {{UN}}SUPPORTED: verbosity :: unsupported.txt (3 of 5)
# TEST-OUTPUT-OFF-NEXT: {{X}}FAIL: verbosity :: xfail.txt (4 of 5)
# TEST-OUTPUT-OFF-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)
# TEST-OUTPUT-OFF-NEXT: ********************
# TEST-OUTPUT-OFF-NEXT: Failed Tests (1):
# TEST-OUTPUT-OFF-NEXT:   verbosity :: fail.txt
# TEST-OUTPUT-OFF-EMPTY:
# TEST-OUTPUT-OFF-NEXT: ********************
# TEST-OUTPUT-OFF-NEXT: Unexpectedly Passed Tests (1):
# TEST-OUTPUT-OFF-NEXT:   verbosity :: xpass.txt
# TEST-OUTPUT-OFF-EMPTY:
# TEST-OUTPUT-OFF-EMPTY:
````
- **L311 EN**: Comment documents nearby script behavior: `ERROR-ERR: 1 warning(s) in tests`.
  **L311 CN**: 注释说明了附近脚本逻辑：`ERROR-ERR: 1 warning(s) in tests`。
- **L312 EN**: Blank line separates nearby declarations or logic blocks.
  **L312 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L313 EN**: Blank line separates nearby declarations or logic blocks.
  **L313 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L314 EN**: Comment documents nearby script behavior: `## Test --test-output`.
  **L314 CN**: 注释说明了附近脚本逻辑：`## Test --test-output`。
- **L315 EN**: Blank line separates nearby declarations or logic blocks.
  **L315 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L316 EN**: Comment documents nearby script behavior: `RUN: not %{lit} --test-output off %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt`.
  **L316 CN**: 注释说明了附近脚本逻辑：`RUN: not %{lit} --test-output off %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt`。
- **L317 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix NO-ARGS < %t/stdout.txt`.
  **L317 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix NO-ARGS < %t/stdout.txt`。
- **L318 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt`.
  **L318 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt`。
- **L319 EN**: Blank line separates nearby declarations or logic blocks.
  **L319 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L320 EN**: Comment documents nearby script behavior: `RUN: not %{lit} --test-output failed %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt`.
  **L320 CN**: 注释说明了附近脚本逻辑：`RUN: not %{lit} --test-output failed %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt`。
- **L321 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix VERBOSE < %t/stdout.txt`.
  **L321 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix VERBOSE < %t/stdout.txt`。
- **L322 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt`.
  **L322 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt`。
- **L323 EN**: Blank line separates nearby declarations or logic blocks.
  **L323 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L324 EN**: Comment documents nearby script behavior: `TEST-OUTPUT-OFF: -- Testing: 5 tests, 1 workers`.
  **L324 CN**: 注释说明了附近脚本逻辑：`TEST-OUTPUT-OFF: -- Testing: 5 tests, 1 workers`。
- **L325 EN**: Comment documents nearby script behavior: `TEST-OUTPUT-OFF-NEXT: FAIL: verbosity :: fail.txt (1 of 5)`.
  **L325 CN**: 注释说明了附近脚本逻辑：`TEST-OUTPUT-OFF-NEXT: FAIL: verbosity :: fail.txt (1 of 5)`。
- **L326 EN**: Comment documents nearby script behavior: `TEST-OUTPUT-OFF-NEXT: PASS: verbosity :: pass.txt (2 of 5)`.
  **L326 CN**: 注释说明了附近脚本逻辑：`TEST-OUTPUT-OFF-NEXT: PASS: verbosity :: pass.txt (2 of 5)`。
- **L327 EN**: Comment documents nearby script behavior: `TEST-OUTPUT-OFF-NEXT: {{UN}}SUPPORTED: verbosity :: unsupported.txt (3 of 5)`.
  **L327 CN**: 注释说明了附近脚本逻辑：`TEST-OUTPUT-OFF-NEXT: {{UN}}SUPPORTED: verbosity :: unsupported.txt (3 of 5)`。
- **L328 EN**: Comment documents nearby script behavior: `TEST-OUTPUT-OFF-NEXT: {{X}}FAIL: verbosity :: xfail.txt (4 of 5)`.
  **L328 CN**: 注释说明了附近脚本逻辑：`TEST-OUTPUT-OFF-NEXT: {{X}}FAIL: verbosity :: xfail.txt (4 of 5)`。
- **L329 EN**: Comment documents nearby script behavior: `TEST-OUTPUT-OFF-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)`.
  **L329 CN**: 注释说明了附近脚本逻辑：`TEST-OUTPUT-OFF-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)`。
- **L330 EN**: Comment documents nearby script behavior: `TEST-OUTPUT-OFF-NEXT: ********************`.
  **L330 CN**: 注释说明了附近脚本逻辑：`TEST-OUTPUT-OFF-NEXT: ********************`。
- **L331 EN**: Comment documents nearby script behavior: `TEST-OUTPUT-OFF-NEXT: Failed Tests (1):`.
  **L331 CN**: 注释说明了附近脚本逻辑：`TEST-OUTPUT-OFF-NEXT: Failed Tests (1):`。
- **L332 EN**: Comment documents nearby script behavior: `TEST-OUTPUT-OFF-NEXT: verbosity :: fail.txt`.
  **L332 CN**: 注释说明了附近脚本逻辑：`TEST-OUTPUT-OFF-NEXT: verbosity :: fail.txt`。
- **L333 EN**: Comment documents nearby script behavior: `TEST-OUTPUT-OFF-EMPTY:`.
  **L333 CN**: 注释说明了附近脚本逻辑：`TEST-OUTPUT-OFF-EMPTY:`。
- **L334 EN**: Comment documents nearby script behavior: `TEST-OUTPUT-OFF-NEXT: ********************`.
  **L334 CN**: 注释说明了附近脚本逻辑：`TEST-OUTPUT-OFF-NEXT: ********************`。
- **L335 EN**: Comment documents nearby script behavior: `TEST-OUTPUT-OFF-NEXT: Unexpectedly Passed Tests (1):`.
  **L335 CN**: 注释说明了附近脚本逻辑：`TEST-OUTPUT-OFF-NEXT: Unexpectedly Passed Tests (1):`。
- **L336 EN**: Comment documents nearby script behavior: `TEST-OUTPUT-OFF-NEXT: verbosity :: xpass.txt`.
  **L336 CN**: 注释说明了附近脚本逻辑：`TEST-OUTPUT-OFF-NEXT: verbosity :: xpass.txt`。
- **L337 EN**: Comment documents nearby script behavior: `TEST-OUTPUT-OFF-EMPTY:`.
  **L337 CN**: 注释说明了附近脚本逻辑：`TEST-OUTPUT-OFF-EMPTY:`。
- **L338 EN**: Comment documents nearby script behavior: `TEST-OUTPUT-OFF-EMPTY:`.
  **L338 CN**: 注释说明了附近脚本逻辑：`TEST-OUTPUT-OFF-EMPTY:`。

### Lines 339-352

````python
# TEST-OUTPUT-OFF-NEXT: Testing Time: {{.*}}s
# TEST-OUTPUT-OFF-EMPTY:
# TEST-OUTPUT-OFF-NEXT: Total Discovered Tests: 5
# TEST-OUTPUT-OFF-NEXT:   Unsupported        : 1 (20.00%)
# TEST-OUTPUT-OFF-NEXT:   Passed             : 1 (20.00%)
# TEST-OUTPUT-OFF-NEXT:   Expectedly Failed  : 1 (20.00%)
# TEST-OUTPUT-OFF-NEXT:   Failed             : 1 (20.00%)
# TEST-OUTPUT-OFF-NEXT:   Unexpectedly Passed: 1 (20.00%)

# RUN: not %{lit} --test-output all  %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt
# RUN: FileCheck %s --check-prefix SHOW-ALL < %t/stdout.txt
# RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt


````
- **L339 EN**: Comment documents nearby script behavior: `TEST-OUTPUT-OFF-NEXT: Testing Time: {{.*}}s`.
  **L339 CN**: 注释说明了附近脚本逻辑：`TEST-OUTPUT-OFF-NEXT: Testing Time: {{.*}}s`。
- **L340 EN**: Comment documents nearby script behavior: `TEST-OUTPUT-OFF-EMPTY:`.
  **L340 CN**: 注释说明了附近脚本逻辑：`TEST-OUTPUT-OFF-EMPTY:`。
- **L341 EN**: Comment documents nearby script behavior: `TEST-OUTPUT-OFF-NEXT: Total Discovered Tests: 5`.
  **L341 CN**: 注释说明了附近脚本逻辑：`TEST-OUTPUT-OFF-NEXT: Total Discovered Tests: 5`。
- **L342 EN**: Comment documents nearby script behavior: `TEST-OUTPUT-OFF-NEXT: Unsupported : 1 (20.00%)`.
  **L342 CN**: 注释说明了附近脚本逻辑：`TEST-OUTPUT-OFF-NEXT: Unsupported : 1 (20.00%)`。
- **L343 EN**: Comment documents nearby script behavior: `TEST-OUTPUT-OFF-NEXT: Passed : 1 (20.00%)`.
  **L343 CN**: 注释说明了附近脚本逻辑：`TEST-OUTPUT-OFF-NEXT: Passed : 1 (20.00%)`。
- **L344 EN**: Comment documents nearby script behavior: `TEST-OUTPUT-OFF-NEXT: Expectedly Failed : 1 (20.00%)`.
  **L344 CN**: 注释说明了附近脚本逻辑：`TEST-OUTPUT-OFF-NEXT: Expectedly Failed : 1 (20.00%)`。
- **L345 EN**: Comment documents nearby script behavior: `TEST-OUTPUT-OFF-NEXT: Failed : 1 (20.00%)`.
  **L345 CN**: 注释说明了附近脚本逻辑：`TEST-OUTPUT-OFF-NEXT: Failed : 1 (20.00%)`。
- **L346 EN**: Comment documents nearby script behavior: `TEST-OUTPUT-OFF-NEXT: Unexpectedly Passed: 1 (20.00%)`.
  **L346 CN**: 注释说明了附近脚本逻辑：`TEST-OUTPUT-OFF-NEXT: Unexpectedly Passed: 1 (20.00%)`。
- **L347 EN**: Blank line separates nearby declarations or logic blocks.
  **L347 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L348 EN**: Comment documents nearby script behavior: `RUN: not %{lit} --test-output all %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt`.
  **L348 CN**: 注释说明了附近脚本逻辑：`RUN: not %{lit} --test-output all %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt`。
- **L349 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix SHOW-ALL < %t/stdout.txt`.
  **L349 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix SHOW-ALL < %t/stdout.txt`。
- **L350 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt`.
  **L350 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt`。
- **L351 EN**: Blank line separates nearby declarations or logic blocks.
  **L351 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L352 EN**: Blank line separates nearby declarations or logic blocks.
  **L352 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 353-376

````python
### Test --print-result-after

# RUN: not %{lit} --print-result-after off  %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt
# RUN: FileCheck %s --check-prefix RESULT-OFF < %t/stdout.txt
# RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt

# RESULT-OFF:      ********************
# RESULT-OFF-NEXT: Failed Tests (1):
# RESULT-OFF-NEXT:   verbosity :: fail.txt
# RESULT-OFF-EMPTY:
# RESULT-OFF-NEXT: ********************
# RESULT-OFF-NEXT: Unexpectedly Passed Tests (1):
# RESULT-OFF-NEXT:   verbosity :: xpass.txt
# RESULT-OFF-EMPTY:
# RESULT-OFF-EMPTY:
# RESULT-OFF-NEXT: Testing Time: {{.*}}s
# RESULT-OFF-EMPTY:
# RESULT-OFF-NEXT: Total Discovered Tests: 5
# RESULT-OFF-NEXT:   Unsupported        : 1 (20.00%)
# RESULT-OFF-NEXT:   Passed             : 1 (20.00%)
# RESULT-OFF-NEXT:   Expectedly Failed  : 1 (20.00%)
# RESULT-OFF-NEXT:   Failed             : 1 (20.00%)
# RESULT-OFF-NEXT:   Unexpectedly Passed: 1 (20.00%)

````
- **L353 EN**: Comment documents nearby script behavior: `## Test --print-result-after`.
  **L353 CN**: 注释说明了附近脚本逻辑：`## Test --print-result-after`。
- **L354 EN**: Blank line separates nearby declarations or logic blocks.
  **L354 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L355 EN**: Comment documents nearby script behavior: `RUN: not %{lit} --print-result-after off %{inputs}/verbosity 2> %t/stderr.txt > %t/stdo...`.
  **L355 CN**: 注释说明了附近脚本逻辑：`RUN: not %{lit} --print-result-after off %{inputs}/verbosity 2> %t/stderr.txt > %t/stdo...`。
- **L356 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix RESULT-OFF < %t/stdout.txt`.
  **L356 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix RESULT-OFF < %t/stdout.txt`。
- **L357 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt`.
  **L357 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt`。
- **L358 EN**: Blank line separates nearby declarations or logic blocks.
  **L358 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L359 EN**: Comment documents nearby script behavior: `RESULT-OFF: ********************`.
  **L359 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF: ********************`。
- **L360 EN**: Comment documents nearby script behavior: `RESULT-OFF-NEXT: Failed Tests (1):`.
  **L360 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-NEXT: Failed Tests (1):`。
- **L361 EN**: Comment documents nearby script behavior: `RESULT-OFF-NEXT: verbosity :: fail.txt`.
  **L361 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-NEXT: verbosity :: fail.txt`。
- **L362 EN**: Comment documents nearby script behavior: `RESULT-OFF-EMPTY:`.
  **L362 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-EMPTY:`。
- **L363 EN**: Comment documents nearby script behavior: `RESULT-OFF-NEXT: ********************`.
  **L363 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-NEXT: ********************`。
- **L364 EN**: Comment documents nearby script behavior: `RESULT-OFF-NEXT: Unexpectedly Passed Tests (1):`.
  **L364 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-NEXT: Unexpectedly Passed Tests (1):`。
- **L365 EN**: Comment documents nearby script behavior: `RESULT-OFF-NEXT: verbosity :: xpass.txt`.
  **L365 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-NEXT: verbosity :: xpass.txt`。
- **L366 EN**: Comment documents nearby script behavior: `RESULT-OFF-EMPTY:`.
  **L366 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-EMPTY:`。
- **L367 EN**: Comment documents nearby script behavior: `RESULT-OFF-EMPTY:`.
  **L367 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-EMPTY:`。
- **L368 EN**: Comment documents nearby script behavior: `RESULT-OFF-NEXT: Testing Time: {{.*}}s`.
  **L368 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-NEXT: Testing Time: {{.*}}s`。
- **L369 EN**: Comment documents nearby script behavior: `RESULT-OFF-EMPTY:`.
  **L369 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-EMPTY:`。
- **L370 EN**: Comment documents nearby script behavior: `RESULT-OFF-NEXT: Total Discovered Tests: 5`.
  **L370 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-NEXT: Total Discovered Tests: 5`。
- **L371 EN**: Comment documents nearby script behavior: `RESULT-OFF-NEXT: Unsupported : 1 (20.00%)`.
  **L371 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-NEXT: Unsupported : 1 (20.00%)`。
- **L372 EN**: Comment documents nearby script behavior: `RESULT-OFF-NEXT: Passed : 1 (20.00%)`.
  **L372 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-NEXT: Passed : 1 (20.00%)`。
- **L373 EN**: Comment documents nearby script behavior: `RESULT-OFF-NEXT: Expectedly Failed : 1 (20.00%)`.
  **L373 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-NEXT: Expectedly Failed : 1 (20.00%)`。
- **L374 EN**: Comment documents nearby script behavior: `RESULT-OFF-NEXT: Failed : 1 (20.00%)`.
  **L374 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-NEXT: Failed : 1 (20.00%)`。
- **L375 EN**: Comment documents nearby script behavior: `RESULT-OFF-NEXT: Unexpectedly Passed: 1 (20.00%)`.
  **L375 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-NEXT: Unexpectedly Passed: 1 (20.00%)`。
- **L376 EN**: Blank line separates nearby declarations or logic blocks.
  **L376 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 377-402

````python

# RUN: not %{lit} --print-result-after failed  %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt
# RUN: FileCheck %s --check-prefix RESULT-FAILED < %t/stdout.txt
# RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt

# RESULT-FAILED:      -- Testing: 5 tests, 1 workers --
# RESULT-FAILED-NEXT: FAIL: verbosity :: fail.txt (1 of 5)
# RESULT-FAILED-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)
# RESULT-FAILED-NEXT: ********************
# RESULT-FAILED-NEXT: Failed Tests (1):
# RESULT-FAILED-NEXT:   verbosity :: fail.txt
# RESULT-FAILED-EMPTY:
# RESULT-FAILED-NEXT: ********************
# RESULT-FAILED-NEXT: Unexpectedly Passed Tests (1):
# RESULT-FAILED-NEXT:   verbosity :: xpass.txt
# RESULT-FAILED-EMPTY:
# RESULT-FAILED-EMPTY:
# RESULT-FAILED-NEXT: Testing Time: {{.*}}s
# RESULT-FAILED-EMPTY:
# RESULT-FAILED-NEXT: Total Discovered Tests: 5
# RESULT-FAILED-NEXT:   Unsupported        : 1 (20.00%)
# RESULT-FAILED-NEXT:   Passed             : 1 (20.00%)
# RESULT-FAILED-NEXT:   Expectedly Failed  : 1 (20.00%)
# RESULT-FAILED-NEXT:   Failed             : 1 (20.00%)
# RESULT-FAILED-NEXT:   Unexpectedly Passed: 1 (20.00%)

````
- **L377 EN**: Blank line separates nearby declarations or logic blocks.
  **L377 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L378 EN**: Comment documents nearby script behavior: `RUN: not %{lit} --print-result-after failed %{inputs}/verbosity 2> %t/stderr.txt > %t/s...`.
  **L378 CN**: 注释说明了附近脚本逻辑：`RUN: not %{lit} --print-result-after failed %{inputs}/verbosity 2> %t/stderr.txt > %t/s...`。
- **L379 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix RESULT-FAILED < %t/stdout.txt`.
  **L379 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix RESULT-FAILED < %t/stdout.txt`。
- **L380 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt`.
  **L380 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt`。
- **L381 EN**: Blank line separates nearby declarations or logic blocks.
  **L381 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L382 EN**: Comment documents nearby script behavior: `RESULT-FAILED: -- Testing: 5 tests, 1 workers`.
  **L382 CN**: 注释说明了附近脚本逻辑：`RESULT-FAILED: -- Testing: 5 tests, 1 workers`。
- **L383 EN**: Comment documents nearby script behavior: `RESULT-FAILED-NEXT: FAIL: verbosity :: fail.txt (1 of 5)`.
  **L383 CN**: 注释说明了附近脚本逻辑：`RESULT-FAILED-NEXT: FAIL: verbosity :: fail.txt (1 of 5)`。
- **L384 EN**: Comment documents nearby script behavior: `RESULT-FAILED-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)`.
  **L384 CN**: 注释说明了附近脚本逻辑：`RESULT-FAILED-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)`。
- **L385 EN**: Comment documents nearby script behavior: `RESULT-FAILED-NEXT: ********************`.
  **L385 CN**: 注释说明了附近脚本逻辑：`RESULT-FAILED-NEXT: ********************`。
- **L386 EN**: Comment documents nearby script behavior: `RESULT-FAILED-NEXT: Failed Tests (1):`.
  **L386 CN**: 注释说明了附近脚本逻辑：`RESULT-FAILED-NEXT: Failed Tests (1):`。
- **L387 EN**: Comment documents nearby script behavior: `RESULT-FAILED-NEXT: verbosity :: fail.txt`.
  **L387 CN**: 注释说明了附近脚本逻辑：`RESULT-FAILED-NEXT: verbosity :: fail.txt`。
- **L388 EN**: Comment documents nearby script behavior: `RESULT-FAILED-EMPTY:`.
  **L388 CN**: 注释说明了附近脚本逻辑：`RESULT-FAILED-EMPTY:`。
- **L389 EN**: Comment documents nearby script behavior: `RESULT-FAILED-NEXT: ********************`.
  **L389 CN**: 注释说明了附近脚本逻辑：`RESULT-FAILED-NEXT: ********************`。
- **L390 EN**: Comment documents nearby script behavior: `RESULT-FAILED-NEXT: Unexpectedly Passed Tests (1):`.
  **L390 CN**: 注释说明了附近脚本逻辑：`RESULT-FAILED-NEXT: Unexpectedly Passed Tests (1):`。
- **L391 EN**: Comment documents nearby script behavior: `RESULT-FAILED-NEXT: verbosity :: xpass.txt`.
  **L391 CN**: 注释说明了附近脚本逻辑：`RESULT-FAILED-NEXT: verbosity :: xpass.txt`。
- **L392 EN**: Comment documents nearby script behavior: `RESULT-FAILED-EMPTY:`.
  **L392 CN**: 注释说明了附近脚本逻辑：`RESULT-FAILED-EMPTY:`。
- **L393 EN**: Comment documents nearby script behavior: `RESULT-FAILED-EMPTY:`.
  **L393 CN**: 注释说明了附近脚本逻辑：`RESULT-FAILED-EMPTY:`。
- **L394 EN**: Comment documents nearby script behavior: `RESULT-FAILED-NEXT: Testing Time: {{.*}}s`.
  **L394 CN**: 注释说明了附近脚本逻辑：`RESULT-FAILED-NEXT: Testing Time: {{.*}}s`。
- **L395 EN**: Comment documents nearby script behavior: `RESULT-FAILED-EMPTY:`.
  **L395 CN**: 注释说明了附近脚本逻辑：`RESULT-FAILED-EMPTY:`。
- **L396 EN**: Comment documents nearby script behavior: `RESULT-FAILED-NEXT: Total Discovered Tests: 5`.
  **L396 CN**: 注释说明了附近脚本逻辑：`RESULT-FAILED-NEXT: Total Discovered Tests: 5`。
- **L397 EN**: Comment documents nearby script behavior: `RESULT-FAILED-NEXT: Unsupported : 1 (20.00%)`.
  **L397 CN**: 注释说明了附近脚本逻辑：`RESULT-FAILED-NEXT: Unsupported : 1 (20.00%)`。
- **L398 EN**: Comment documents nearby script behavior: `RESULT-FAILED-NEXT: Passed : 1 (20.00%)`.
  **L398 CN**: 注释说明了附近脚本逻辑：`RESULT-FAILED-NEXT: Passed : 1 (20.00%)`。
- **L399 EN**: Comment documents nearby script behavior: `RESULT-FAILED-NEXT: Expectedly Failed : 1 (20.00%)`.
  **L399 CN**: 注释说明了附近脚本逻辑：`RESULT-FAILED-NEXT: Expectedly Failed : 1 (20.00%)`。
- **L400 EN**: Comment documents nearby script behavior: `RESULT-FAILED-NEXT: Failed : 1 (20.00%)`.
  **L400 CN**: 注释说明了附近脚本逻辑：`RESULT-FAILED-NEXT: Failed : 1 (20.00%)`。
- **L401 EN**: Comment documents nearby script behavior: `RESULT-FAILED-NEXT: Unexpectedly Passed: 1 (20.00%)`.
  **L401 CN**: 注释说明了附近脚本逻辑：`RESULT-FAILED-NEXT: Unexpectedly Passed: 1 (20.00%)`。
- **L402 EN**: Blank line separates nearby declarations or logic blocks.
  **L402 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 403-430

````python

# RUN: not %{lit} --print-result-after all  %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt
# RUN: FileCheck %s --check-prefix NO-ARGS < %t/stdout.txt
# RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt


### Test combinations of --print-result-after followed by --test-output

# RUN: not %{lit} --print-result-after off --test-output failed %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt
# RUN: FileCheck %s --check-prefix RESULT-OFF-OUTPUT-FAILED < %t/stdout.txt
# RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt

# RESULT-OFF-OUTPUT-FAILED:      -- Testing: 5 tests, 1 workers --
# RESULT-OFF-OUTPUT-FAILED-NEXT: FAIL: verbosity :: fail.txt (1 of 5)
# RESULT-OFF-OUTPUT-FAILED-NEXT: ******************** TEST 'verbosity :: fail.txt' FAILED ********************
# RESULT-OFF-OUTPUT-FAILED-NEXT: Exit Code: 127
# RESULT-OFF-OUTPUT-FAILED-EMPTY:
# RESULT-OFF-OUTPUT-FAILED-NEXT: Command Output (stdout):
# RESULT-OFF-OUTPUT-FAILED-NEXT: --
# RESULT-OFF-OUTPUT-FAILED-NEXT: # {{R}}UN: at line 1
# RESULT-OFF-OUTPUT-FAILED-NEXT: echo "fail test output"
# RESULT-OFF-OUTPUT-FAILED-NEXT: # executed command: echo 'fail test output'
# RESULT-OFF-OUTPUT-FAILED-NEXT: # .---command stdout------------
# RESULT-OFF-OUTPUT-FAILED-NEXT: # | fail test output
# RESULT-OFF-OUTPUT-FAILED-NEXT: # `-----------------------------
# RESULT-OFF-OUTPUT-FAILED-NEXT: # {{R}}UN: at line 2
# RESULT-OFF-OUTPUT-FAILED-NEXT: fail
# RESULT-OFF-OUTPUT-FAILED-NEXT: # executed command: fail
````
- **L403 EN**: Blank line separates nearby declarations or logic blocks.
  **L403 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L404 EN**: Comment documents nearby script behavior: `RUN: not %{lit} --print-result-after all %{inputs}/verbosity 2> %t/stderr.txt > %t/stdo...`.
  **L404 CN**: 注释说明了附近脚本逻辑：`RUN: not %{lit} --print-result-after all %{inputs}/verbosity 2> %t/stderr.txt > %t/stdo...`。
- **L405 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix NO-ARGS < %t/stdout.txt`.
  **L405 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix NO-ARGS < %t/stdout.txt`。
- **L406 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt`.
  **L406 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt`。
- **L407 EN**: Blank line separates nearby declarations or logic blocks.
  **L407 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L408 EN**: Blank line separates nearby declarations or logic blocks.
  **L408 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L409 EN**: Comment documents nearby script behavior: `## Test combinations of --print-result-after followed by --test-output`.
  **L409 CN**: 注释说明了附近脚本逻辑：`## Test combinations of --print-result-after followed by --test-output`。
- **L410 EN**: Blank line separates nearby declarations or logic blocks.
  **L410 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L411 EN**: Comment documents nearby script behavior: `RUN: not %{lit} --print-result-after off --test-output failed %{inputs}/verbosity 2> %t...`.
  **L411 CN**: 注释说明了附近脚本逻辑：`RUN: not %{lit} --print-result-after off --test-output failed %{inputs}/verbosity 2> %t...`。
- **L412 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix RESULT-OFF-OUTPUT-FAILED < %t/stdout.txt`.
  **L412 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix RESULT-OFF-OUTPUT-FAILED < %t/stdout.txt`。
- **L413 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt`.
  **L413 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt`。
- **L414 EN**: Blank line separates nearby declarations or logic blocks.
  **L414 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L415 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED: -- Testing: 5 tests, 1 workers`.
  **L415 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED: -- Testing: 5 tests, 1 workers`。
- **L416 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-NEXT: FAIL: verbosity :: fail.txt (1 of 5)`.
  **L416 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-NEXT: FAIL: verbosity :: fail.txt (1 of 5)`。
- **L417 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-NEXT: ******************** TEST 'verbosity :: fail.txt' FAILED...`.
  **L417 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-NEXT: ******************** TEST 'verbosity :: fail.txt' FAILED...`。
- **L418 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-NEXT: Exit Code: 127`.
  **L418 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-NEXT: Exit Code: 127`。
- **L419 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-EMPTY:`.
  **L419 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-EMPTY:`。
- **L420 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-NEXT: Command Output (stdout):`.
  **L420 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-NEXT: Command Output (stdout):`。
- **L421 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-NEXT:`.
  **L421 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-NEXT:`。
- **L422 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-NEXT: # {{R}}UN: at line 1`.
  **L422 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-NEXT: # {{R}}UN: at line 1`。
- **L423 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-NEXT: echo "fail test output"`.
  **L423 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-NEXT: echo "fail test output"`。
- **L424 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-NEXT: # executed command: echo 'fail test output'`.
  **L424 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-NEXT: # executed command: echo 'fail test output'`。
- **L425 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-NEXT: # .---command stdout`.
  **L425 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-NEXT: # .---command stdout`。
- **L426 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-NEXT: # | fail test output`.
  **L426 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-NEXT: # | fail test output`。
- **L427 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-NEXT: # \``.
  **L427 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-NEXT: # \``。
- **L428 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-NEXT: # {{R}}UN: at line 2`.
  **L428 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-NEXT: # {{R}}UN: at line 2`。
- **L429 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-NEXT: fail`.
  **L429 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-NEXT: fail`。
- **L430 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-NEXT: # executed command: fail`.
  **L430 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-NEXT: # executed command: fail`。

### Lines 431-458

````python
# RESULT-OFF-OUTPUT-FAILED-NEXT: # .---command stderr------------
# RESULT-OFF-OUTPUT-FAILED-NEXT: # | 'fail': command not found
# RESULT-OFF-OUTPUT-FAILED-NEXT: # `-----------------------------
# RESULT-OFF-OUTPUT-FAILED-NEXT: # error: command failed with exit status: 127
# RESULT-OFF-OUTPUT-FAILED-EMPTY:
# RESULT-OFF-OUTPUT-FAILED-NEXT: --
# RESULT-OFF-OUTPUT-FAILED-EMPTY:
# RESULT-OFF-OUTPUT-FAILED-NEXT: ********************
# RESULT-OFF-OUTPUT-FAILED-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)
# RESULT-OFF-OUTPUT-FAILED-NEXT: ******************** TEST 'verbosity :: xpass.txt' FAILED ********************
# RESULT-OFF-OUTPUT-FAILED-NEXT: Exit Code: 0
# RESULT-OFF-OUTPUT-FAILED-EMPTY:
# RESULT-OFF-OUTPUT-FAILED-NEXT: Command Output (stdout):
# RESULT-OFF-OUTPUT-FAILED-NEXT: --
# RESULT-OFF-OUTPUT-FAILED-NEXT: # {{R}}UN: at line 2
# RESULT-OFF-OUTPUT-FAILED-NEXT: echo "xpass test output"
# RESULT-OFF-OUTPUT-FAILED-NEXT: # executed command: echo 'xpass test output'
# RESULT-OFF-OUTPUT-FAILED-NEXT: # .---command stdout------------
# RESULT-OFF-OUTPUT-FAILED-NEXT: # | xpass test output
# RESULT-OFF-OUTPUT-FAILED-NEXT: # `-----------------------------
# RESULT-OFF-OUTPUT-FAILED-EMPTY:
# RESULT-OFF-OUTPUT-FAILED-NEXT: --
# RESULT-OFF-OUTPUT-FAILED-EMPTY:
# RESULT-OFF-OUTPUT-FAILED-NEXT: ********************
# RESULT-OFF-OUTPUT-FAILED-NEXT: ********************
# RESULT-OFF-OUTPUT-FAILED-NEXT: Failed Tests (1):
# RESULT-OFF-OUTPUT-FAILED-NEXT:   verbosity :: fail.txt
# RESULT-OFF-OUTPUT-FAILED-EMPTY:
````
- **L431 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-NEXT: # .---command stderr`.
  **L431 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-NEXT: # .---command stderr`。
- **L432 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-NEXT: # | 'fail': command not found`.
  **L432 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-NEXT: # | 'fail': command not found`。
- **L433 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-NEXT: # \``.
  **L433 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-NEXT: # \``。
- **L434 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-NEXT: # error: command failed with exit status: 127`.
  **L434 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-NEXT: # error: command failed with exit status: 127`。
- **L435 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-EMPTY:`.
  **L435 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-EMPTY:`。
- **L436 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-NEXT:`.
  **L436 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-NEXT:`。
- **L437 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-EMPTY:`.
  **L437 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-EMPTY:`。
- **L438 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-NEXT: ********************`.
  **L438 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-NEXT: ********************`。
- **L439 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)`.
  **L439 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)`。
- **L440 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-NEXT: ******************** TEST 'verbosity :: xpass.txt' FAILE...`.
  **L440 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-NEXT: ******************** TEST 'verbosity :: xpass.txt' FAILE...`。
- **L441 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-NEXT: Exit Code: 0`.
  **L441 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-NEXT: Exit Code: 0`。
- **L442 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-EMPTY:`.
  **L442 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-EMPTY:`。
- **L443 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-NEXT: Command Output (stdout):`.
  **L443 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-NEXT: Command Output (stdout):`。
- **L444 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-NEXT:`.
  **L444 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-NEXT:`。
- **L445 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-NEXT: # {{R}}UN: at line 2`.
  **L445 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-NEXT: # {{R}}UN: at line 2`。
- **L446 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-NEXT: echo "xpass test output"`.
  **L446 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-NEXT: echo "xpass test output"`。
- **L447 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-NEXT: # executed command: echo 'xpass test output'`.
  **L447 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-NEXT: # executed command: echo 'xpass test output'`。
- **L448 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-NEXT: # .---command stdout`.
  **L448 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-NEXT: # .---command stdout`。
- **L449 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-NEXT: # | xpass test output`.
  **L449 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-NEXT: # | xpass test output`。
- **L450 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-NEXT: # \``.
  **L450 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-NEXT: # \``。
- **L451 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-EMPTY:`.
  **L451 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-EMPTY:`。
- **L452 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-NEXT:`.
  **L452 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-NEXT:`。
- **L453 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-EMPTY:`.
  **L453 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-EMPTY:`。
- **L454 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-NEXT: ********************`.
  **L454 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-NEXT: ********************`。
- **L455 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-NEXT: ********************`.
  **L455 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-NEXT: ********************`。
- **L456 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-NEXT: Failed Tests (1):`.
  **L456 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-NEXT: Failed Tests (1):`。
- **L457 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-NEXT: verbosity :: fail.txt`.
  **L457 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-NEXT: verbosity :: fail.txt`。
- **L458 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-EMPTY:`.
  **L458 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-EMPTY:`。

### Lines 459-472

````python
# RESULT-OFF-OUTPUT-FAILED-NEXT: ********************
# RESULT-OFF-OUTPUT-FAILED-NEXT: Unexpectedly Passed Tests (1):
# RESULT-OFF-OUTPUT-FAILED-NEXT:   verbosity :: xpass.txt
# RESULT-OFF-OUTPUT-FAILED-EMPTY:
# RESULT-OFF-OUTPUT-FAILED-EMPTY:
# RESULT-OFF-OUTPUT-FAILED-NEXT: Testing Time: {{.*}}s
# RESULT-OFF-OUTPUT-FAILED-EMPTY:
# RESULT-OFF-OUTPUT-FAILED-NEXT: Total Discovered Tests: 5
# RESULT-OFF-OUTPUT-FAILED-NEXT:   Unsupported        : 1 (20.00%)
# RESULT-OFF-OUTPUT-FAILED-NEXT:   Passed             : 1 (20.00%)
# RESULT-OFF-OUTPUT-FAILED-NEXT:   Expectedly Failed  : 1 (20.00%)
# RESULT-OFF-OUTPUT-FAILED-NEXT:   Failed             : 1 (20.00%)
# RESULT-OFF-OUTPUT-FAILED-NEXT:   Unexpectedly Passed: 1 (20.00%)

````
- **L459 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-NEXT: ********************`.
  **L459 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-NEXT: ********************`。
- **L460 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-NEXT: Unexpectedly Passed Tests (1):`.
  **L460 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-NEXT: Unexpectedly Passed Tests (1):`。
- **L461 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-NEXT: verbosity :: xpass.txt`.
  **L461 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-NEXT: verbosity :: xpass.txt`。
- **L462 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-EMPTY:`.
  **L462 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-EMPTY:`。
- **L463 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-EMPTY:`.
  **L463 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-EMPTY:`。
- **L464 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-NEXT: Testing Time: {{.*}}s`.
  **L464 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-NEXT: Testing Time: {{.*}}s`。
- **L465 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-EMPTY:`.
  **L465 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-EMPTY:`。
- **L466 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-NEXT: Total Discovered Tests: 5`.
  **L466 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-NEXT: Total Discovered Tests: 5`。
- **L467 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-NEXT: Unsupported : 1 (20.00%)`.
  **L467 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-NEXT: Unsupported : 1 (20.00%)`。
- **L468 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-NEXT: Passed : 1 (20.00%)`.
  **L468 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-NEXT: Passed : 1 (20.00%)`。
- **L469 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-NEXT: Expectedly Failed : 1 (20.00%)`.
  **L469 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-NEXT: Expectedly Failed : 1 (20.00%)`。
- **L470 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-NEXT: Failed : 1 (20.00%)`.
  **L470 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-NEXT: Failed : 1 (20.00%)`。
- **L471 EN**: Comment documents nearby script behavior: `RESULT-OFF-OUTPUT-FAILED-NEXT: Unexpectedly Passed: 1 (20.00%)`.
  **L471 CN**: 注释说明了附近脚本逻辑：`RESULT-OFF-OUTPUT-FAILED-NEXT: Unexpectedly Passed: 1 (20.00%)`。
- **L472 EN**: Blank line separates nearby declarations or logic blocks.
  **L472 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 473-487

````python
# RUN: not %{lit} --print-result-after all --test-output off %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt
# RUN: FileCheck %s --check-prefix NO-ARGS < %t/stdout.txt
# RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt

# RUN: not %{lit} --print-result-after failed --test-output all %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt
# RUN: FileCheck %s --check-prefix SHOW-ALL < %t/stdout.txt
# RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt


### Test combinations of --test-output followed by --print-result-after

# RUN: not %{lit} --test-output failed --print-result-after off %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt
# RUN: FileCheck %s --check-prefix RESULT-OFF < %t/stdout.txt
# RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt

````
- **L473 EN**: Comment documents nearby script behavior: `RUN: not %{lit} --print-result-after all --test-output off %{inputs}/verbosity 2> %t/st...`.
  **L473 CN**: 注释说明了附近脚本逻辑：`RUN: not %{lit} --print-result-after all --test-output off %{inputs}/verbosity 2> %t/st...`。
- **L474 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix NO-ARGS < %t/stdout.txt`.
  **L474 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix NO-ARGS < %t/stdout.txt`。
- **L475 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt`.
  **L475 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt`。
- **L476 EN**: Blank line separates nearby declarations or logic blocks.
  **L476 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L477 EN**: Comment documents nearby script behavior: `RUN: not %{lit} --print-result-after failed --test-output all %{inputs}/verbosity 2> %t...`.
  **L477 CN**: 注释说明了附近脚本逻辑：`RUN: not %{lit} --print-result-after failed --test-output all %{inputs}/verbosity 2> %t...`。
- **L478 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix SHOW-ALL < %t/stdout.txt`.
  **L478 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix SHOW-ALL < %t/stdout.txt`。
- **L479 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt`.
  **L479 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt`。
- **L480 EN**: Blank line separates nearby declarations or logic blocks.
  **L480 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L481 EN**: Blank line separates nearby declarations or logic blocks.
  **L481 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L482 EN**: Comment documents nearby script behavior: `## Test combinations of --test-output followed by --print-result-after`.
  **L482 CN**: 注释说明了附近脚本逻辑：`## Test combinations of --test-output followed by --print-result-after`。
- **L483 EN**: Blank line separates nearby declarations or logic blocks.
  **L483 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L484 EN**: Comment documents nearby script behavior: `RUN: not %{lit} --test-output failed --print-result-after off %{inputs}/verbosity 2> %t...`.
  **L484 CN**: 注释说明了附近脚本逻辑：`RUN: not %{lit} --test-output failed --print-result-after off %{inputs}/verbosity 2> %t...`。
- **L485 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix RESULT-OFF < %t/stdout.txt`.
  **L485 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix RESULT-OFF < %t/stdout.txt`。
- **L486 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt`.
  **L486 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt`。
- **L487 EN**: Blank line separates nearby declarations or logic blocks.
  **L487 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 488-515

````python
# RUN: not %{lit} --test-output off --print-result-after all %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt
# RUN: FileCheck %s --check-prefix NO-ARGS < %t/stdout.txt
# RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt

# RUN: not %{lit} --test-output all --print-result-after failed %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt
# RUN: FileCheck %s --check-prefix OUTPUT-ALL-RESULT-FAILED < %t/stdout.txt
# RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt

# OUTPUT-ALL-RESULT-FAILED:      -- Testing: 5 tests, 1 workers --
# OUTPUT-ALL-RESULT-FAILED-NEXT: FAIL: verbosity :: fail.txt (1 of 5)
# OUTPUT-ALL-RESULT-FAILED-NEXT: ******************** TEST 'verbosity :: fail.txt' FAILED ********************
# OUTPUT-ALL-RESULT-FAILED-NEXT: Exit Code: 127
# OUTPUT-ALL-RESULT-FAILED-EMPTY:
# OUTPUT-ALL-RESULT-FAILED-NEXT: Command Output (stdout):
# OUTPUT-ALL-RESULT-FAILED-NEXT: --
# OUTPUT-ALL-RESULT-FAILED-NEXT: # {{R}}UN: at line 1
# OUTPUT-ALL-RESULT-FAILED-NEXT: echo "fail test output"
# OUTPUT-ALL-RESULT-FAILED-NEXT: # executed command: echo 'fail test output'
# OUTPUT-ALL-RESULT-FAILED-NEXT: # .---command stdout------------
# OUTPUT-ALL-RESULT-FAILED-NEXT: # | fail test output
# OUTPUT-ALL-RESULT-FAILED-NEXT: # `-----------------------------
# OUTPUT-ALL-RESULT-FAILED-NEXT: # {{R}}UN: at line 2
# OUTPUT-ALL-RESULT-FAILED-NEXT: fail
# OUTPUT-ALL-RESULT-FAILED-NEXT: # executed command: fail
# OUTPUT-ALL-RESULT-FAILED-NEXT: # .---command stderr------------
# OUTPUT-ALL-RESULT-FAILED-NEXT: # | 'fail': command not found
# OUTPUT-ALL-RESULT-FAILED-NEXT: # `-----------------------------
# OUTPUT-ALL-RESULT-FAILED-NEXT: # error: command failed with exit status: 127
````
- **L488 EN**: Comment documents nearby script behavior: `RUN: not %{lit} --test-output off --print-result-after all %{inputs}/verbosity 2> %t/st...`.
  **L488 CN**: 注释说明了附近脚本逻辑：`RUN: not %{lit} --test-output off --print-result-after all %{inputs}/verbosity 2> %t/st...`。
- **L489 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix NO-ARGS < %t/stdout.txt`.
  **L489 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix NO-ARGS < %t/stdout.txt`。
- **L490 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt`.
  **L490 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt`。
- **L491 EN**: Blank line separates nearby declarations or logic blocks.
  **L491 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L492 EN**: Comment documents nearby script behavior: `RUN: not %{lit} --test-output all --print-result-after failed %{inputs}/verbosity 2> %t...`.
  **L492 CN**: 注释说明了附近脚本逻辑：`RUN: not %{lit} --test-output all --print-result-after failed %{inputs}/verbosity 2> %t...`。
- **L493 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix OUTPUT-ALL-RESULT-FAILED < %t/stdout.txt`.
  **L493 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix OUTPUT-ALL-RESULT-FAILED < %t/stdout.txt`。
- **L494 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt`.
  **L494 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt`。
- **L495 EN**: Blank line separates nearby declarations or logic blocks.
  **L495 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L496 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED: -- Testing: 5 tests, 1 workers`.
  **L496 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED: -- Testing: 5 tests, 1 workers`。
- **L497 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-NEXT: FAIL: verbosity :: fail.txt (1 of 5)`.
  **L497 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-NEXT: FAIL: verbosity :: fail.txt (1 of 5)`。
- **L498 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-NEXT: ******************** TEST 'verbosity :: fail.txt' FAILED...`.
  **L498 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-NEXT: ******************** TEST 'verbosity :: fail.txt' FAILED...`。
- **L499 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-NEXT: Exit Code: 127`.
  **L499 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-NEXT: Exit Code: 127`。
- **L500 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-EMPTY:`.
  **L500 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-EMPTY:`。
- **L501 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-NEXT: Command Output (stdout):`.
  **L501 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-NEXT: Command Output (stdout):`。
- **L502 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-NEXT:`.
  **L502 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-NEXT:`。
- **L503 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-NEXT: # {{R}}UN: at line 1`.
  **L503 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-NEXT: # {{R}}UN: at line 1`。
- **L504 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-NEXT: echo "fail test output"`.
  **L504 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-NEXT: echo "fail test output"`。
- **L505 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-NEXT: # executed command: echo 'fail test output'`.
  **L505 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-NEXT: # executed command: echo 'fail test output'`。
- **L506 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-NEXT: # .---command stdout`.
  **L506 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-NEXT: # .---command stdout`。
- **L507 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-NEXT: # | fail test output`.
  **L507 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-NEXT: # | fail test output`。
- **L508 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-NEXT: # \``.
  **L508 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-NEXT: # \``。
- **L509 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-NEXT: # {{R}}UN: at line 2`.
  **L509 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-NEXT: # {{R}}UN: at line 2`。
- **L510 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-NEXT: fail`.
  **L510 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-NEXT: fail`。
- **L511 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-NEXT: # executed command: fail`.
  **L511 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-NEXT: # executed command: fail`。
- **L512 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-NEXT: # .---command stderr`.
  **L512 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-NEXT: # .---command stderr`。
- **L513 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-NEXT: # | 'fail': command not found`.
  **L513 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-NEXT: # | 'fail': command not found`。
- **L514 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-NEXT: # \``.
  **L514 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-NEXT: # \``。
- **L515 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-NEXT: # error: command failed with exit status: 127`.
  **L515 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-NEXT: # error: command failed with exit status: 127`。

### Lines 516-543

````python
# OUTPUT-ALL-RESULT-FAILED-EMPTY:
# OUTPUT-ALL-RESULT-FAILED-NEXT: --
# OUTPUT-ALL-RESULT-FAILED-EMPTY:
# OUTPUT-ALL-RESULT-FAILED-NEXT: ********************
# OUTPUT-ALL-RESULT-FAILED-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)
# OUTPUT-ALL-RESULT-FAILED-NEXT: ******************** TEST 'verbosity :: xpass.txt' FAILED ********************
# OUTPUT-ALL-RESULT-FAILED-NEXT: Exit Code: 0
# OUTPUT-ALL-RESULT-FAILED-EMPTY:
# OUTPUT-ALL-RESULT-FAILED-NEXT: Command Output (stdout):
# OUTPUT-ALL-RESULT-FAILED-NEXT: --
# OUTPUT-ALL-RESULT-FAILED-NEXT: # {{R}}UN: at line 2
# OUTPUT-ALL-RESULT-FAILED-NEXT: echo "xpass test output"
# OUTPUT-ALL-RESULT-FAILED-NEXT: # executed command: echo 'xpass test output'
# OUTPUT-ALL-RESULT-FAILED-NEXT: # .---command stdout------------
# OUTPUT-ALL-RESULT-FAILED-NEXT: # | xpass test output
# OUTPUT-ALL-RESULT-FAILED-NEXT: # `-----------------------------
# OUTPUT-ALL-RESULT-FAILED-EMPTY:
# OUTPUT-ALL-RESULT-FAILED-NEXT: --
# OUTPUT-ALL-RESULT-FAILED-EMPTY:
# OUTPUT-ALL-RESULT-FAILED-NEXT: ********************
# OUTPUT-ALL-RESULT-FAILED-NEXT: ********************
# OUTPUT-ALL-RESULT-FAILED-NEXT: Failed Tests (1):
# OUTPUT-ALL-RESULT-FAILED-NEXT:   verbosity :: fail.txt
# OUTPUT-ALL-RESULT-FAILED-EMPTY:
# OUTPUT-ALL-RESULT-FAILED-NEXT: ********************
# OUTPUT-ALL-RESULT-FAILED-NEXT: Unexpectedly Passed Tests (1):
# OUTPUT-ALL-RESULT-FAILED-NEXT:   verbosity :: xpass.txt
# OUTPUT-ALL-RESULT-FAILED-EMPTY:
````
- **L516 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-EMPTY:`.
  **L516 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-EMPTY:`。
- **L517 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-NEXT:`.
  **L517 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-NEXT:`。
- **L518 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-EMPTY:`.
  **L518 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-EMPTY:`。
- **L519 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-NEXT: ********************`.
  **L519 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-NEXT: ********************`。
- **L520 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)`.
  **L520 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)`。
- **L521 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-NEXT: ******************** TEST 'verbosity :: xpass.txt' FAILE...`.
  **L521 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-NEXT: ******************** TEST 'verbosity :: xpass.txt' FAILE...`。
- **L522 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-NEXT: Exit Code: 0`.
  **L522 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-NEXT: Exit Code: 0`。
- **L523 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-EMPTY:`.
  **L523 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-EMPTY:`。
- **L524 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-NEXT: Command Output (stdout):`.
  **L524 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-NEXT: Command Output (stdout):`。
- **L525 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-NEXT:`.
  **L525 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-NEXT:`。
- **L526 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-NEXT: # {{R}}UN: at line 2`.
  **L526 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-NEXT: # {{R}}UN: at line 2`。
- **L527 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-NEXT: echo "xpass test output"`.
  **L527 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-NEXT: echo "xpass test output"`。
- **L528 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-NEXT: # executed command: echo 'xpass test output'`.
  **L528 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-NEXT: # executed command: echo 'xpass test output'`。
- **L529 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-NEXT: # .---command stdout`.
  **L529 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-NEXT: # .---command stdout`。
- **L530 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-NEXT: # | xpass test output`.
  **L530 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-NEXT: # | xpass test output`。
- **L531 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-NEXT: # \``.
  **L531 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-NEXT: # \``。
- **L532 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-EMPTY:`.
  **L532 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-EMPTY:`。
- **L533 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-NEXT:`.
  **L533 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-NEXT:`。
- **L534 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-EMPTY:`.
  **L534 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-EMPTY:`。
- **L535 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-NEXT: ********************`.
  **L535 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-NEXT: ********************`。
- **L536 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-NEXT: ********************`.
  **L536 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-NEXT: ********************`。
- **L537 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-NEXT: Failed Tests (1):`.
  **L537 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-NEXT: Failed Tests (1):`。
- **L538 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-NEXT: verbosity :: fail.txt`.
  **L538 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-NEXT: verbosity :: fail.txt`。
- **L539 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-EMPTY:`.
  **L539 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-EMPTY:`。
- **L540 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-NEXT: ********************`.
  **L540 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-NEXT: ********************`。
- **L541 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-NEXT: Unexpectedly Passed Tests (1):`.
  **L541 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-NEXT: Unexpectedly Passed Tests (1):`。
- **L542 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-NEXT: verbosity :: xpass.txt`.
  **L542 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-NEXT: verbosity :: xpass.txt`。
- **L543 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-EMPTY:`.
  **L543 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-EMPTY:`。

### Lines 544-560

````python
# OUTPUT-ALL-RESULT-FAILED-EMPTY:
# OUTPUT-ALL-RESULT-FAILED-NEXT: Testing Time: {{.*}}
# OUTPUT-ALL-RESULT-FAILED-EMPTY:
# OUTPUT-ALL-RESULT-FAILED-NEXT: Total Discovered Tests: 5
# OUTPUT-ALL-RESULT-FAILED-NEXT:   Unsupported        : 1 (20.00%)
# OUTPUT-ALL-RESULT-FAILED-NEXT:   Passed             : 1 (20.00%)
# OUTPUT-ALL-RESULT-FAILED-NEXT:   Expectedly Failed  : 1 (20.00%)
# OUTPUT-ALL-RESULT-FAILED-NEXT:   Failed             : 1 (20.00%)
# OUTPUT-ALL-RESULT-FAILED-NEXT:   Unexpectedly Passed: 1 (20.00%)


### Test progress bar and terse summary in isolation

# RUN: not %{lit} --progress-bar %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt
# RUN: FileCheck %s --check-prefix PROGRESS < %t/stdout.txt
# RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt

````
- **L544 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-EMPTY:`.
  **L544 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-EMPTY:`。
- **L545 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-NEXT: Testing Time: {{.*}}`.
  **L545 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-NEXT: Testing Time: {{.*}}`。
- **L546 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-EMPTY:`.
  **L546 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-EMPTY:`。
- **L547 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-NEXT: Total Discovered Tests: 5`.
  **L547 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-NEXT: Total Discovered Tests: 5`。
- **L548 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-NEXT: Unsupported : 1 (20.00%)`.
  **L548 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-NEXT: Unsupported : 1 (20.00%)`。
- **L549 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-NEXT: Passed : 1 (20.00%)`.
  **L549 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-NEXT: Passed : 1 (20.00%)`。
- **L550 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-NEXT: Expectedly Failed : 1 (20.00%)`.
  **L550 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-NEXT: Expectedly Failed : 1 (20.00%)`。
- **L551 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-NEXT: Failed : 1 (20.00%)`.
  **L551 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-NEXT: Failed : 1 (20.00%)`。
- **L552 EN**: Comment documents nearby script behavior: `OUTPUT-ALL-RESULT-FAILED-NEXT: Unexpectedly Passed: 1 (20.00%)`.
  **L552 CN**: 注释说明了附近脚本逻辑：`OUTPUT-ALL-RESULT-FAILED-NEXT: Unexpectedly Passed: 1 (20.00%)`。
- **L553 EN**: Blank line separates nearby declarations or logic blocks.
  **L553 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L554 EN**: Blank line separates nearby declarations or logic blocks.
  **L554 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L555 EN**: Comment documents nearby script behavior: `## Test progress bar and terse summary in isolation`.
  **L555 CN**: 注释说明了附近脚本逻辑：`## Test progress bar and terse summary in isolation`。
- **L556 EN**: Blank line separates nearby declarations or logic blocks.
  **L556 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L557 EN**: Comment documents nearby script behavior: `RUN: not %{lit} --progress-bar %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt`.
  **L557 CN**: 注释说明了附近脚本逻辑：`RUN: not %{lit} --progress-bar %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt`。
- **L558 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix PROGRESS < %t/stdout.txt`.
  **L558 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix PROGRESS < %t/stdout.txt`。
- **L559 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt`.
  **L559 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt`。
- **L560 EN**: Blank line separates nearby declarations or logic blocks.
  **L560 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 561-588

````python
# PROGRESS:      -- Testing: 5 tests, 1 workers --
# PROGRESS-NEXT: Testing:
# PROGRESS-NEXT: FAIL: verbosity :: fail.txt (1 of 5)
# PROGRESS-NEXT: Testing:
# PROGRESS-NEXT: PASS: verbosity :: pass.txt (2 of 5)
# PROGRESS-NEXT: Testing:
# PROGRESS-NEXT: {{UN}}SUPPORTED: verbosity :: unsupported.txt (3 of 5)
# PROGRESS-NEXT: Testing:
# PROGRESS-NEXT: {{X}}FAIL: verbosity :: xfail.txt (4 of 5)
# PROGRESS-NEXT: Testing:
# PROGRESS-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)
# PROGRESS-NEXT: Testing:
# PROGRESS-NEXT: ********************
# PROGRESS-NEXT: Failed Tests (1):
# PROGRESS-NEXT:   verbosity :: fail.txt
# PROGRESS-EMPTY:
# PROGRESS-NEXT: ********************
# PROGRESS-NEXT: Unexpectedly Passed Tests (1):
# PROGRESS-NEXT:   verbosity :: xpass.txt
# PROGRESS-EMPTY:
# PROGRESS-EMPTY:
# PROGRESS-NEXT: Testing Time: {{.*}}s
# PROGRESS-EMPTY:
# PROGRESS-NEXT: Total Discovered Tests: 5
# PROGRESS-NEXT:   Unsupported        : 1 (20.00%)
# PROGRESS-NEXT:   Passed             : 1 (20.00%)
# PROGRESS-NEXT:   Expectedly Failed  : 1 (20.00%)
# PROGRESS-NEXT:   Failed             : 1 (20.00%)
````
- **L561 EN**: Comment documents nearby script behavior: `PROGRESS: -- Testing: 5 tests, 1 workers`.
  **L561 CN**: 注释说明了附近脚本逻辑：`PROGRESS: -- Testing: 5 tests, 1 workers`。
- **L562 EN**: Comment documents nearby script behavior: `PROGRESS-NEXT: Testing:`.
  **L562 CN**: 注释说明了附近脚本逻辑：`PROGRESS-NEXT: Testing:`。
- **L563 EN**: Comment documents nearby script behavior: `PROGRESS-NEXT: FAIL: verbosity :: fail.txt (1 of 5)`.
  **L563 CN**: 注释说明了附近脚本逻辑：`PROGRESS-NEXT: FAIL: verbosity :: fail.txt (1 of 5)`。
- **L564 EN**: Comment documents nearby script behavior: `PROGRESS-NEXT: Testing:`.
  **L564 CN**: 注释说明了附近脚本逻辑：`PROGRESS-NEXT: Testing:`。
- **L565 EN**: Comment documents nearby script behavior: `PROGRESS-NEXT: PASS: verbosity :: pass.txt (2 of 5)`.
  **L565 CN**: 注释说明了附近脚本逻辑：`PROGRESS-NEXT: PASS: verbosity :: pass.txt (2 of 5)`。
- **L566 EN**: Comment documents nearby script behavior: `PROGRESS-NEXT: Testing:`.
  **L566 CN**: 注释说明了附近脚本逻辑：`PROGRESS-NEXT: Testing:`。
- **L567 EN**: Comment documents nearby script behavior: `PROGRESS-NEXT: {{UN}}SUPPORTED: verbosity :: unsupported.txt (3 of 5)`.
  **L567 CN**: 注释说明了附近脚本逻辑：`PROGRESS-NEXT: {{UN}}SUPPORTED: verbosity :: unsupported.txt (3 of 5)`。
- **L568 EN**: Comment documents nearby script behavior: `PROGRESS-NEXT: Testing:`.
  **L568 CN**: 注释说明了附近脚本逻辑：`PROGRESS-NEXT: Testing:`。
- **L569 EN**: Comment documents nearby script behavior: `PROGRESS-NEXT: {{X}}FAIL: verbosity :: xfail.txt (4 of 5)`.
  **L569 CN**: 注释说明了附近脚本逻辑：`PROGRESS-NEXT: {{X}}FAIL: verbosity :: xfail.txt (4 of 5)`。
- **L570 EN**: Comment documents nearby script behavior: `PROGRESS-NEXT: Testing:`.
  **L570 CN**: 注释说明了附近脚本逻辑：`PROGRESS-NEXT: Testing:`。
- **L571 EN**: Comment documents nearby script behavior: `PROGRESS-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)`.
  **L571 CN**: 注释说明了附近脚本逻辑：`PROGRESS-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)`。
- **L572 EN**: Comment documents nearby script behavior: `PROGRESS-NEXT: Testing:`.
  **L572 CN**: 注释说明了附近脚本逻辑：`PROGRESS-NEXT: Testing:`。
- **L573 EN**: Comment documents nearby script behavior: `PROGRESS-NEXT: ********************`.
  **L573 CN**: 注释说明了附近脚本逻辑：`PROGRESS-NEXT: ********************`。
- **L574 EN**: Comment documents nearby script behavior: `PROGRESS-NEXT: Failed Tests (1):`.
  **L574 CN**: 注释说明了附近脚本逻辑：`PROGRESS-NEXT: Failed Tests (1):`。
- **L575 EN**: Comment documents nearby script behavior: `PROGRESS-NEXT: verbosity :: fail.txt`.
  **L575 CN**: 注释说明了附近脚本逻辑：`PROGRESS-NEXT: verbosity :: fail.txt`。
- **L576 EN**: Comment documents nearby script behavior: `PROGRESS-EMPTY:`.
  **L576 CN**: 注释说明了附近脚本逻辑：`PROGRESS-EMPTY:`。
- **L577 EN**: Comment documents nearby script behavior: `PROGRESS-NEXT: ********************`.
  **L577 CN**: 注释说明了附近脚本逻辑：`PROGRESS-NEXT: ********************`。
- **L578 EN**: Comment documents nearby script behavior: `PROGRESS-NEXT: Unexpectedly Passed Tests (1):`.
  **L578 CN**: 注释说明了附近脚本逻辑：`PROGRESS-NEXT: Unexpectedly Passed Tests (1):`。
- **L579 EN**: Comment documents nearby script behavior: `PROGRESS-NEXT: verbosity :: xpass.txt`.
  **L579 CN**: 注释说明了附近脚本逻辑：`PROGRESS-NEXT: verbosity :: xpass.txt`。
- **L580 EN**: Comment documents nearby script behavior: `PROGRESS-EMPTY:`.
  **L580 CN**: 注释说明了附近脚本逻辑：`PROGRESS-EMPTY:`。
- **L581 EN**: Comment documents nearby script behavior: `PROGRESS-EMPTY:`.
  **L581 CN**: 注释说明了附近脚本逻辑：`PROGRESS-EMPTY:`。
- **L582 EN**: Comment documents nearby script behavior: `PROGRESS-NEXT: Testing Time: {{.*}}s`.
  **L582 CN**: 注释说明了附近脚本逻辑：`PROGRESS-NEXT: Testing Time: {{.*}}s`。
- **L583 EN**: Comment documents nearby script behavior: `PROGRESS-EMPTY:`.
  **L583 CN**: 注释说明了附近脚本逻辑：`PROGRESS-EMPTY:`。
- **L584 EN**: Comment documents nearby script behavior: `PROGRESS-NEXT: Total Discovered Tests: 5`.
  **L584 CN**: 注释说明了附近脚本逻辑：`PROGRESS-NEXT: Total Discovered Tests: 5`。
- **L585 EN**: Comment documents nearby script behavior: `PROGRESS-NEXT: Unsupported : 1 (20.00%)`.
  **L585 CN**: 注释说明了附近脚本逻辑：`PROGRESS-NEXT: Unsupported : 1 (20.00%)`。
- **L586 EN**: Comment documents nearby script behavior: `PROGRESS-NEXT: Passed : 1 (20.00%)`.
  **L586 CN**: 注释说明了附近脚本逻辑：`PROGRESS-NEXT: Passed : 1 (20.00%)`。
- **L587 EN**: Comment documents nearby script behavior: `PROGRESS-NEXT: Expectedly Failed : 1 (20.00%)`.
  **L587 CN**: 注释说明了附近脚本逻辑：`PROGRESS-NEXT: Expectedly Failed : 1 (20.00%)`。
- **L588 EN**: Comment documents nearby script behavior: `PROGRESS-NEXT: Failed : 1 (20.00%)`.
  **L588 CN**: 注释说明了附近脚本逻辑：`PROGRESS-NEXT: Failed : 1 (20.00%)`。

### Lines 589-613

````python
# PROGRESS-NEXT:   Unexpectedly Passed: 1 (20.00%)

# RUN: not %{lit} --terse-summary %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt
# RUN: FileCheck %s --check-prefix TERSE < %t/stdout.txt
# RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt

# TERSE:      -- Testing: 5 tests, 1 workers --
# TERSE-NEXT: FAIL: verbosity :: fail.txt (1 of 5)
# TERSE-NEXT: PASS: verbosity :: pass.txt (2 of 5)
# TERSE-NEXT: {{UN}}SUPPORTED: verbosity :: unsupported.txt (3 of 5)
# TERSE-NEXT: {{X}}FAIL: verbosity :: xfail.txt (4 of 5)
# TERSE-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)
# TERSE-NEXT: ********************
# TERSE-NEXT: Failed Tests (1):
# TERSE-NEXT:   verbosity :: fail.txt
# TERSE-EMPTY:
# TERSE-NEXT: ********************
# TERSE-NEXT: Unexpectedly Passed Tests (1):
# TERSE-NEXT:   verbosity :: xpass.txt
# TERSE-EMPTY:
# TERSE-EMPTY:
# TERSE-NEXT: Total Discovered Tests: 5
# TERSE-NEXT:   Failed             : 1 (20.00%)
# TERSE-NEXT:   Unexpectedly Passed: 1 (20.00%)

````
- **L589 EN**: Comment documents nearby script behavior: `PROGRESS-NEXT: Unexpectedly Passed: 1 (20.00%)`.
  **L589 CN**: 注释说明了附近脚本逻辑：`PROGRESS-NEXT: Unexpectedly Passed: 1 (20.00%)`。
- **L590 EN**: Blank line separates nearby declarations or logic blocks.
  **L590 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L591 EN**: Comment documents nearby script behavior: `RUN: not %{lit} --terse-summary %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt`.
  **L591 CN**: 注释说明了附近脚本逻辑：`RUN: not %{lit} --terse-summary %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt`。
- **L592 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix TERSE < %t/stdout.txt`.
  **L592 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix TERSE < %t/stdout.txt`。
- **L593 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt`.
  **L593 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt`。
- **L594 EN**: Blank line separates nearby declarations or logic blocks.
  **L594 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L595 EN**: Comment documents nearby script behavior: `TERSE: -- Testing: 5 tests, 1 workers`.
  **L595 CN**: 注释说明了附近脚本逻辑：`TERSE: -- Testing: 5 tests, 1 workers`。
- **L596 EN**: Comment documents nearby script behavior: `TERSE-NEXT: FAIL: verbosity :: fail.txt (1 of 5)`.
  **L596 CN**: 注释说明了附近脚本逻辑：`TERSE-NEXT: FAIL: verbosity :: fail.txt (1 of 5)`。
- **L597 EN**: Comment documents nearby script behavior: `TERSE-NEXT: PASS: verbosity :: pass.txt (2 of 5)`.
  **L597 CN**: 注释说明了附近脚本逻辑：`TERSE-NEXT: PASS: verbosity :: pass.txt (2 of 5)`。
- **L598 EN**: Comment documents nearby script behavior: `TERSE-NEXT: {{UN}}SUPPORTED: verbosity :: unsupported.txt (3 of 5)`.
  **L598 CN**: 注释说明了附近脚本逻辑：`TERSE-NEXT: {{UN}}SUPPORTED: verbosity :: unsupported.txt (3 of 5)`。
- **L599 EN**: Comment documents nearby script behavior: `TERSE-NEXT: {{X}}FAIL: verbosity :: xfail.txt (4 of 5)`.
  **L599 CN**: 注释说明了附近脚本逻辑：`TERSE-NEXT: {{X}}FAIL: verbosity :: xfail.txt (4 of 5)`。
- **L600 EN**: Comment documents nearby script behavior: `TERSE-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)`.
  **L600 CN**: 注释说明了附近脚本逻辑：`TERSE-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)`。
- **L601 EN**: Comment documents nearby script behavior: `TERSE-NEXT: ********************`.
  **L601 CN**: 注释说明了附近脚本逻辑：`TERSE-NEXT: ********************`。
- **L602 EN**: Comment documents nearby script behavior: `TERSE-NEXT: Failed Tests (1):`.
  **L602 CN**: 注释说明了附近脚本逻辑：`TERSE-NEXT: Failed Tests (1):`。
- **L603 EN**: Comment documents nearby script behavior: `TERSE-NEXT: verbosity :: fail.txt`.
  **L603 CN**: 注释说明了附近脚本逻辑：`TERSE-NEXT: verbosity :: fail.txt`。
- **L604 EN**: Comment documents nearby script behavior: `TERSE-EMPTY:`.
  **L604 CN**: 注释说明了附近脚本逻辑：`TERSE-EMPTY:`。
- **L605 EN**: Comment documents nearby script behavior: `TERSE-NEXT: ********************`.
  **L605 CN**: 注释说明了附近脚本逻辑：`TERSE-NEXT: ********************`。
- **L606 EN**: Comment documents nearby script behavior: `TERSE-NEXT: Unexpectedly Passed Tests (1):`.
  **L606 CN**: 注释说明了附近脚本逻辑：`TERSE-NEXT: Unexpectedly Passed Tests (1):`。
- **L607 EN**: Comment documents nearby script behavior: `TERSE-NEXT: verbosity :: xpass.txt`.
  **L607 CN**: 注释说明了附近脚本逻辑：`TERSE-NEXT: verbosity :: xpass.txt`。
- **L608 EN**: Comment documents nearby script behavior: `TERSE-EMPTY:`.
  **L608 CN**: 注释说明了附近脚本逻辑：`TERSE-EMPTY:`。
- **L609 EN**: Comment documents nearby script behavior: `TERSE-EMPTY:`.
  **L609 CN**: 注释说明了附近脚本逻辑：`TERSE-EMPTY:`。
- **L610 EN**: Comment documents nearby script behavior: `TERSE-NEXT: Total Discovered Tests: 5`.
  **L610 CN**: 注释说明了附近脚本逻辑：`TERSE-NEXT: Total Discovered Tests: 5`。
- **L611 EN**: Comment documents nearby script behavior: `TERSE-NEXT: Failed : 1 (20.00%)`.
  **L611 CN**: 注释说明了附近脚本逻辑：`TERSE-NEXT: Failed : 1 (20.00%)`。
- **L612 EN**: Comment documents nearby script behavior: `TERSE-NEXT: Unexpectedly Passed: 1 (20.00%)`.
  **L612 CN**: 注释说明了附近脚本逻辑：`TERSE-NEXT: Unexpectedly Passed: 1 (20.00%)`。
- **L613 EN**: Blank line separates nearby declarations or logic blocks.
  **L613 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 614-641

````python

### Aliases in combination

# RUN: not %{lit} -a -s %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt
# RUN: FileCheck %s --check-prefix AS < %t/stdout.txt
# RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt

# AS:      -- Testing: 5 tests, 1 workers --
# AS-NEXT: Testing:
# AS-NEXT: FAIL: verbosity :: fail.txt (1 of 5)
# AS-NEXT: ******************** TEST 'verbosity :: fail.txt' FAILED ********************
# AS-NEXT: Exit Code: 127
# AS-EMPTY:
# AS-NEXT: Command Output (stdout):
# AS-NEXT: --
# AS-NEXT: # {{R}}UN: at line 1
# AS-NEXT: echo "fail test output"
# AS-NEXT: # executed command: echo 'fail test output'
# AS-NEXT: # .---command stdout------------
# AS-NEXT: # | fail test output
# AS-NEXT: # `-----------------------------
# AS-NEXT: # {{R}}UN: at line 2
# AS-NEXT: fail
# AS-NEXT: # executed command: fail
# AS-NEXT: # .---command stderr------------
# AS-NEXT: # | 'fail': command not found
# AS-NEXT: # `-----------------------------
# AS-NEXT: # error: command failed with exit status: 127
````
- **L614 EN**: Blank line separates nearby declarations or logic blocks.
  **L614 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L615 EN**: Comment documents nearby script behavior: `## Aliases in combination`.
  **L615 CN**: 注释说明了附近脚本逻辑：`## Aliases in combination`。
- **L616 EN**: Blank line separates nearby declarations or logic blocks.
  **L616 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L617 EN**: Comment documents nearby script behavior: `RUN: not %{lit} -a -s %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt`.
  **L617 CN**: 注释说明了附近脚本逻辑：`RUN: not %{lit} -a -s %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt`。
- **L618 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix AS < %t/stdout.txt`.
  **L618 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix AS < %t/stdout.txt`。
- **L619 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt`.
  **L619 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt`。
- **L620 EN**: Blank line separates nearby declarations or logic blocks.
  **L620 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L621 EN**: Comment documents nearby script behavior: `AS: -- Testing: 5 tests, 1 workers`.
  **L621 CN**: 注释说明了附近脚本逻辑：`AS: -- Testing: 5 tests, 1 workers`。
- **L622 EN**: Comment documents nearby script behavior: `AS-NEXT: Testing:`.
  **L622 CN**: 注释说明了附近脚本逻辑：`AS-NEXT: Testing:`。
- **L623 EN**: Comment documents nearby script behavior: `AS-NEXT: FAIL: verbosity :: fail.txt (1 of 5)`.
  **L623 CN**: 注释说明了附近脚本逻辑：`AS-NEXT: FAIL: verbosity :: fail.txt (1 of 5)`。
- **L624 EN**: Comment documents nearby script behavior: `AS-NEXT: ******************** TEST 'verbosity :: fail.txt' FAILED ********************`.
  **L624 CN**: 注释说明了附近脚本逻辑：`AS-NEXT: ******************** TEST 'verbosity :: fail.txt' FAILED ********************`。
- **L625 EN**: Comment documents nearby script behavior: `AS-NEXT: Exit Code: 127`.
  **L625 CN**: 注释说明了附近脚本逻辑：`AS-NEXT: Exit Code: 127`。
- **L626 EN**: Comment documents nearby script behavior: `AS-EMPTY:`.
  **L626 CN**: 注释说明了附近脚本逻辑：`AS-EMPTY:`。
- **L627 EN**: Comment documents nearby script behavior: `AS-NEXT: Command Output (stdout):`.
  **L627 CN**: 注释说明了附近脚本逻辑：`AS-NEXT: Command Output (stdout):`。
- **L628 EN**: Comment documents nearby script behavior: `AS-NEXT:`.
  **L628 CN**: 注释说明了附近脚本逻辑：`AS-NEXT:`。
- **L629 EN**: Comment documents nearby script behavior: `AS-NEXT: # {{R}}UN: at line 1`.
  **L629 CN**: 注释说明了附近脚本逻辑：`AS-NEXT: # {{R}}UN: at line 1`。
- **L630 EN**: Comment documents nearby script behavior: `AS-NEXT: echo "fail test output"`.
  **L630 CN**: 注释说明了附近脚本逻辑：`AS-NEXT: echo "fail test output"`。
- **L631 EN**: Comment documents nearby script behavior: `AS-NEXT: # executed command: echo 'fail test output'`.
  **L631 CN**: 注释说明了附近脚本逻辑：`AS-NEXT: # executed command: echo 'fail test output'`。
- **L632 EN**: Comment documents nearby script behavior: `AS-NEXT: # .---command stdout`.
  **L632 CN**: 注释说明了附近脚本逻辑：`AS-NEXT: # .---command stdout`。
- **L633 EN**: Comment documents nearby script behavior: `AS-NEXT: # | fail test output`.
  **L633 CN**: 注释说明了附近脚本逻辑：`AS-NEXT: # | fail test output`。
- **L634 EN**: Comment documents nearby script behavior: `AS-NEXT: # \``.
  **L634 CN**: 注释说明了附近脚本逻辑：`AS-NEXT: # \``。
- **L635 EN**: Comment documents nearby script behavior: `AS-NEXT: # {{R}}UN: at line 2`.
  **L635 CN**: 注释说明了附近脚本逻辑：`AS-NEXT: # {{R}}UN: at line 2`。
- **L636 EN**: Comment documents nearby script behavior: `AS-NEXT: fail`.
  **L636 CN**: 注释说明了附近脚本逻辑：`AS-NEXT: fail`。
- **L637 EN**: Comment documents nearby script behavior: `AS-NEXT: # executed command: fail`.
  **L637 CN**: 注释说明了附近脚本逻辑：`AS-NEXT: # executed command: fail`。
- **L638 EN**: Comment documents nearby script behavior: `AS-NEXT: # .---command stderr`.
  **L638 CN**: 注释说明了附近脚本逻辑：`AS-NEXT: # .---command stderr`。
- **L639 EN**: Comment documents nearby script behavior: `AS-NEXT: # | 'fail': command not found`.
  **L639 CN**: 注释说明了附近脚本逻辑：`AS-NEXT: # | 'fail': command not found`。
- **L640 EN**: Comment documents nearby script behavior: `AS-NEXT: # \``.
  **L640 CN**: 注释说明了附近脚本逻辑：`AS-NEXT: # \``。
- **L641 EN**: Comment documents nearby script behavior: `AS-NEXT: # error: command failed with exit status: 127`.
  **L641 CN**: 注释说明了附近脚本逻辑：`AS-NEXT: # error: command failed with exit status: 127`。

### Lines 642-669

````python
# AS-EMPTY:
# AS-NEXT: --
# AS-EMPTY:
# AS-NEXT: ********************
# AS-NEXT: Testing:
# AS-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)
# AS-NEXT: ******************** TEST 'verbosity :: xpass.txt' FAILED ********************
# AS-NEXT: Exit Code: 0
# AS-EMPTY:
# AS-NEXT: Command Output (stdout):
# AS-NEXT: --
# AS-NEXT: # {{R}}UN: at line 2
# AS-NEXT: echo "xpass test output"
# AS-NEXT: # executed command: echo 'xpass test output'
# AS-NEXT: # .---command stdout------------
# AS-NEXT: # | xpass test output
# AS-NEXT: # `-----------------------------
# AS-EMPTY:
# AS-NEXT: --
# AS-EMPTY:
# AS-NEXT: ********************
# AS-NEXT: Testing:
# AS-NEXT: ********************
# AS-NEXT: Failed Tests (1):
# AS-NEXT:   verbosity :: fail.txt
# AS-EMPTY:
# AS-NEXT: ********************
# AS-NEXT: Unexpectedly Passed Tests (1):
````
- **L642 EN**: Comment documents nearby script behavior: `AS-EMPTY:`.
  **L642 CN**: 注释说明了附近脚本逻辑：`AS-EMPTY:`。
- **L643 EN**: Comment documents nearby script behavior: `AS-NEXT:`.
  **L643 CN**: 注释说明了附近脚本逻辑：`AS-NEXT:`。
- **L644 EN**: Comment documents nearby script behavior: `AS-EMPTY:`.
  **L644 CN**: 注释说明了附近脚本逻辑：`AS-EMPTY:`。
- **L645 EN**: Comment documents nearby script behavior: `AS-NEXT: ********************`.
  **L645 CN**: 注释说明了附近脚本逻辑：`AS-NEXT: ********************`。
- **L646 EN**: Comment documents nearby script behavior: `AS-NEXT: Testing:`.
  **L646 CN**: 注释说明了附近脚本逻辑：`AS-NEXT: Testing:`。
- **L647 EN**: Comment documents nearby script behavior: `AS-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)`.
  **L647 CN**: 注释说明了附近脚本逻辑：`AS-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)`。
- **L648 EN**: Comment documents nearby script behavior: `AS-NEXT: ******************** TEST 'verbosity :: xpass.txt' FAILED ********************`.
  **L648 CN**: 注释说明了附近脚本逻辑：`AS-NEXT: ******************** TEST 'verbosity :: xpass.txt' FAILED ********************`。
- **L649 EN**: Comment documents nearby script behavior: `AS-NEXT: Exit Code: 0`.
  **L649 CN**: 注释说明了附近脚本逻辑：`AS-NEXT: Exit Code: 0`。
- **L650 EN**: Comment documents nearby script behavior: `AS-EMPTY:`.
  **L650 CN**: 注释说明了附近脚本逻辑：`AS-EMPTY:`。
- **L651 EN**: Comment documents nearby script behavior: `AS-NEXT: Command Output (stdout):`.
  **L651 CN**: 注释说明了附近脚本逻辑：`AS-NEXT: Command Output (stdout):`。
- **L652 EN**: Comment documents nearby script behavior: `AS-NEXT:`.
  **L652 CN**: 注释说明了附近脚本逻辑：`AS-NEXT:`。
- **L653 EN**: Comment documents nearby script behavior: `AS-NEXT: # {{R}}UN: at line 2`.
  **L653 CN**: 注释说明了附近脚本逻辑：`AS-NEXT: # {{R}}UN: at line 2`。
- **L654 EN**: Comment documents nearby script behavior: `AS-NEXT: echo "xpass test output"`.
  **L654 CN**: 注释说明了附近脚本逻辑：`AS-NEXT: echo "xpass test output"`。
- **L655 EN**: Comment documents nearby script behavior: `AS-NEXT: # executed command: echo 'xpass test output'`.
  **L655 CN**: 注释说明了附近脚本逻辑：`AS-NEXT: # executed command: echo 'xpass test output'`。
- **L656 EN**: Comment documents nearby script behavior: `AS-NEXT: # .---command stdout`.
  **L656 CN**: 注释说明了附近脚本逻辑：`AS-NEXT: # .---command stdout`。
- **L657 EN**: Comment documents nearby script behavior: `AS-NEXT: # | xpass test output`.
  **L657 CN**: 注释说明了附近脚本逻辑：`AS-NEXT: # | xpass test output`。
- **L658 EN**: Comment documents nearby script behavior: `AS-NEXT: # \``.
  **L658 CN**: 注释说明了附近脚本逻辑：`AS-NEXT: # \``。
- **L659 EN**: Comment documents nearby script behavior: `AS-EMPTY:`.
  **L659 CN**: 注释说明了附近脚本逻辑：`AS-EMPTY:`。
- **L660 EN**: Comment documents nearby script behavior: `AS-NEXT:`.
  **L660 CN**: 注释说明了附近脚本逻辑：`AS-NEXT:`。
- **L661 EN**: Comment documents nearby script behavior: `AS-EMPTY:`.
  **L661 CN**: 注释说明了附近脚本逻辑：`AS-EMPTY:`。
- **L662 EN**: Comment documents nearby script behavior: `AS-NEXT: ********************`.
  **L662 CN**: 注释说明了附近脚本逻辑：`AS-NEXT: ********************`。
- **L663 EN**: Comment documents nearby script behavior: `AS-NEXT: Testing:`.
  **L663 CN**: 注释说明了附近脚本逻辑：`AS-NEXT: Testing:`。
- **L664 EN**: Comment documents nearby script behavior: `AS-NEXT: ********************`.
  **L664 CN**: 注释说明了附近脚本逻辑：`AS-NEXT: ********************`。
- **L665 EN**: Comment documents nearby script behavior: `AS-NEXT: Failed Tests (1):`.
  **L665 CN**: 注释说明了附近脚本逻辑：`AS-NEXT: Failed Tests (1):`。
- **L666 EN**: Comment documents nearby script behavior: `AS-NEXT: verbosity :: fail.txt`.
  **L666 CN**: 注释说明了附近脚本逻辑：`AS-NEXT: verbosity :: fail.txt`。
- **L667 EN**: Comment documents nearby script behavior: `AS-EMPTY:`.
  **L667 CN**: 注释说明了附近脚本逻辑：`AS-EMPTY:`。
- **L668 EN**: Comment documents nearby script behavior: `AS-NEXT: ********************`.
  **L668 CN**: 注释说明了附近脚本逻辑：`AS-NEXT: ********************`。
- **L669 EN**: Comment documents nearby script behavior: `AS-NEXT: Unexpectedly Passed Tests (1):`.
  **L669 CN**: 注释说明了附近脚本逻辑：`AS-NEXT: Unexpectedly Passed Tests (1):`。

### Lines 670-686

````python
# AS-NEXT:   verbosity :: xpass.txt
# AS-EMPTY:
# AS-EMPTY:
# AS-NEXT: Testing Time: {{.*}}s
# AS-EMPTY:
# AS-NEXT: Total Discovered Tests: 5
# AS-NEXT:   Unsupported        : 1 (20.00%)
# AS-NEXT:   Passed             : 1 (20.00%)
# AS-NEXT:   Expectedly Failed  : 1 (20.00%)
# AS-NEXT:   Failed             : 1 (20.00%)
# AS-NEXT:   Unexpectedly Passed: 1 (20.00%)


# RUN: not %{lit} -s -a %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt
# RUN: FileCheck %s --check-prefix SA < %t/stdout.txt
# RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt

````
- **L670 EN**: Comment documents nearby script behavior: `AS-NEXT: verbosity :: xpass.txt`.
  **L670 CN**: 注释说明了附近脚本逻辑：`AS-NEXT: verbosity :: xpass.txt`。
- **L671 EN**: Comment documents nearby script behavior: `AS-EMPTY:`.
  **L671 CN**: 注释说明了附近脚本逻辑：`AS-EMPTY:`。
- **L672 EN**: Comment documents nearby script behavior: `AS-EMPTY:`.
  **L672 CN**: 注释说明了附近脚本逻辑：`AS-EMPTY:`。
- **L673 EN**: Comment documents nearby script behavior: `AS-NEXT: Testing Time: {{.*}}s`.
  **L673 CN**: 注释说明了附近脚本逻辑：`AS-NEXT: Testing Time: {{.*}}s`。
- **L674 EN**: Comment documents nearby script behavior: `AS-EMPTY:`.
  **L674 CN**: 注释说明了附近脚本逻辑：`AS-EMPTY:`。
- **L675 EN**: Comment documents nearby script behavior: `AS-NEXT: Total Discovered Tests: 5`.
  **L675 CN**: 注释说明了附近脚本逻辑：`AS-NEXT: Total Discovered Tests: 5`。
- **L676 EN**: Comment documents nearby script behavior: `AS-NEXT: Unsupported : 1 (20.00%)`.
  **L676 CN**: 注释说明了附近脚本逻辑：`AS-NEXT: Unsupported : 1 (20.00%)`。
- **L677 EN**: Comment documents nearby script behavior: `AS-NEXT: Passed : 1 (20.00%)`.
  **L677 CN**: 注释说明了附近脚本逻辑：`AS-NEXT: Passed : 1 (20.00%)`。
- **L678 EN**: Comment documents nearby script behavior: `AS-NEXT: Expectedly Failed : 1 (20.00%)`.
  **L678 CN**: 注释说明了附近脚本逻辑：`AS-NEXT: Expectedly Failed : 1 (20.00%)`。
- **L679 EN**: Comment documents nearby script behavior: `AS-NEXT: Failed : 1 (20.00%)`.
  **L679 CN**: 注释说明了附近脚本逻辑：`AS-NEXT: Failed : 1 (20.00%)`。
- **L680 EN**: Comment documents nearby script behavior: `AS-NEXT: Unexpectedly Passed: 1 (20.00%)`.
  **L680 CN**: 注释说明了附近脚本逻辑：`AS-NEXT: Unexpectedly Passed: 1 (20.00%)`。
- **L681 EN**: Blank line separates nearby declarations or logic blocks.
  **L681 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L682 EN**: Blank line separates nearby declarations or logic blocks.
  **L682 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L683 EN**: Comment documents nearby script behavior: `RUN: not %{lit} -s -a %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt`.
  **L683 CN**: 注释说明了附近脚本逻辑：`RUN: not %{lit} -s -a %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt`。
- **L684 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix SA < %t/stdout.txt`.
  **L684 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix SA < %t/stdout.txt`。
- **L685 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt`.
  **L685 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt`。
- **L686 EN**: Blank line separates nearby declarations or logic blocks.
  **L686 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 687-714

````python
# SA:      -- Testing: 5 tests, 1 workers --
# SA-NEXT: Testing:
# SA-NEXT: FAIL: verbosity :: fail.txt (1 of 5)
# SA-NEXT: ******************** TEST 'verbosity :: fail.txt' FAILED ********************
# SA-NEXT: Exit Code: 127
# SA-EMPTY:
# SA-NEXT: Command Output (stdout):
# SA-NEXT: --
# SA-NEXT: # {{R}}UN: at line 1
# SA-NEXT: echo "fail test output"
# SA-NEXT: # executed command: echo 'fail test output'
# SA-NEXT: # .---command stdout------------
# SA-NEXT: # | fail test output
# SA-NEXT: # `-----------------------------
# SA-NEXT: # {{R}}UN: at line 2
# SA-NEXT: fail
# SA-NEXT: # executed command: fail
# SA-NEXT: # .---command stderr------------
# SA-NEXT: # | 'fail': command not found
# SA-NEXT: # `-----------------------------
# SA-NEXT: # error: command failed with exit status: 127
# SA-EMPTY:
# SA-NEXT: --
# SA-EMPTY:
# SA-NEXT: ********************
# SA-NEXT: Testing:
# SA-NEXT: PASS: verbosity :: pass.txt (2 of 5)
# SA-NEXT: Exit Code: 0
````
- **L687 EN**: Comment documents nearby script behavior: `SA: -- Testing: 5 tests, 1 workers`.
  **L687 CN**: 注释说明了附近脚本逻辑：`SA: -- Testing: 5 tests, 1 workers`。
- **L688 EN**: Comment documents nearby script behavior: `SA-NEXT: Testing:`.
  **L688 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: Testing:`。
- **L689 EN**: Comment documents nearby script behavior: `SA-NEXT: FAIL: verbosity :: fail.txt (1 of 5)`.
  **L689 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: FAIL: verbosity :: fail.txt (1 of 5)`。
- **L690 EN**: Comment documents nearby script behavior: `SA-NEXT: ******************** TEST 'verbosity :: fail.txt' FAILED ********************`.
  **L690 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: ******************** TEST 'verbosity :: fail.txt' FAILED ********************`。
- **L691 EN**: Comment documents nearby script behavior: `SA-NEXT: Exit Code: 127`.
  **L691 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: Exit Code: 127`。
- **L692 EN**: Comment documents nearby script behavior: `SA-EMPTY:`.
  **L692 CN**: 注释说明了附近脚本逻辑：`SA-EMPTY:`。
- **L693 EN**: Comment documents nearby script behavior: `SA-NEXT: Command Output (stdout):`.
  **L693 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: Command Output (stdout):`。
- **L694 EN**: Comment documents nearby script behavior: `SA-NEXT:`.
  **L694 CN**: 注释说明了附近脚本逻辑：`SA-NEXT:`。
- **L695 EN**: Comment documents nearby script behavior: `SA-NEXT: # {{R}}UN: at line 1`.
  **L695 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: # {{R}}UN: at line 1`。
- **L696 EN**: Comment documents nearby script behavior: `SA-NEXT: echo "fail test output"`.
  **L696 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: echo "fail test output"`。
- **L697 EN**: Comment documents nearby script behavior: `SA-NEXT: # executed command: echo 'fail test output'`.
  **L697 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: # executed command: echo 'fail test output'`。
- **L698 EN**: Comment documents nearby script behavior: `SA-NEXT: # .---command stdout`.
  **L698 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: # .---command stdout`。
- **L699 EN**: Comment documents nearby script behavior: `SA-NEXT: # | fail test output`.
  **L699 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: # | fail test output`。
- **L700 EN**: Comment documents nearby script behavior: `SA-NEXT: # \``.
  **L700 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: # \``。
- **L701 EN**: Comment documents nearby script behavior: `SA-NEXT: # {{R}}UN: at line 2`.
  **L701 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: # {{R}}UN: at line 2`。
- **L702 EN**: Comment documents nearby script behavior: `SA-NEXT: fail`.
  **L702 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: fail`。
- **L703 EN**: Comment documents nearby script behavior: `SA-NEXT: # executed command: fail`.
  **L703 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: # executed command: fail`。
- **L704 EN**: Comment documents nearby script behavior: `SA-NEXT: # .---command stderr`.
  **L704 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: # .---command stderr`。
- **L705 EN**: Comment documents nearby script behavior: `SA-NEXT: # | 'fail': command not found`.
  **L705 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: # | 'fail': command not found`。
- **L706 EN**: Comment documents nearby script behavior: `SA-NEXT: # \``.
  **L706 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: # \``。
- **L707 EN**: Comment documents nearby script behavior: `SA-NEXT: # error: command failed with exit status: 127`.
  **L707 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: # error: command failed with exit status: 127`。
- **L708 EN**: Comment documents nearby script behavior: `SA-EMPTY:`.
  **L708 CN**: 注释说明了附近脚本逻辑：`SA-EMPTY:`。
- **L709 EN**: Comment documents nearby script behavior: `SA-NEXT:`.
  **L709 CN**: 注释说明了附近脚本逻辑：`SA-NEXT:`。
- **L710 EN**: Comment documents nearby script behavior: `SA-EMPTY:`.
  **L710 CN**: 注释说明了附近脚本逻辑：`SA-EMPTY:`。
- **L711 EN**: Comment documents nearby script behavior: `SA-NEXT: ********************`.
  **L711 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: ********************`。
- **L712 EN**: Comment documents nearby script behavior: `SA-NEXT: Testing:`.
  **L712 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: Testing:`。
- **L713 EN**: Comment documents nearby script behavior: `SA-NEXT: PASS: verbosity :: pass.txt (2 of 5)`.
  **L713 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: PASS: verbosity :: pass.txt (2 of 5)`。
- **L714 EN**: Comment documents nearby script behavior: `SA-NEXT: Exit Code: 0`.
  **L714 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: Exit Code: 0`。

### Lines 715-742

````python
# SA-EMPTY:
# SA-NEXT: Command Output (stdout):
# SA-NEXT: --
# SA-NEXT: # {{R}}UN: at line 1
# SA-NEXT: echo "pass test output"
# SA-NEXT: # executed command: echo 'pass test output'
# SA-NEXT: # .---command stdout------------
# SA-NEXT: # | pass test output
# SA-NEXT: # `-----------------------------
# SA-EMPTY:
# SA-NEXT: --
# SA-EMPTY:
# SA-NEXT: ********************
# SA-NEXT: Testing:
# SA-NEXT: {{UN}}SUPPORTED: verbosity :: unsupported.txt (3 of 5)
# SA-NEXT: Test requires the following unavailable features: asdf
# SA-NEXT: ********************
# SA-NEXT: Testing:
# SA-NEXT: {{X}}FAIL: verbosity :: xfail.txt (4 of 5)
# SA-NEXT: Exit Code: 1
# SA-EMPTY:
# SA-NEXT: Command Output (stdout):
# SA-NEXT: --
# SA-NEXT: # {{R}}UN: at line 2
# SA-NEXT: not echo "xfail test output"
# SA-NEXT: # executed command: not echo 'xfail test output'
# SA-NEXT: # .---command stdout------------
# SA-NEXT: # | xfail test output
````
- **L715 EN**: Comment documents nearby script behavior: `SA-EMPTY:`.
  **L715 CN**: 注释说明了附近脚本逻辑：`SA-EMPTY:`。
- **L716 EN**: Comment documents nearby script behavior: `SA-NEXT: Command Output (stdout):`.
  **L716 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: Command Output (stdout):`。
- **L717 EN**: Comment documents nearby script behavior: `SA-NEXT:`.
  **L717 CN**: 注释说明了附近脚本逻辑：`SA-NEXT:`。
- **L718 EN**: Comment documents nearby script behavior: `SA-NEXT: # {{R}}UN: at line 1`.
  **L718 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: # {{R}}UN: at line 1`。
- **L719 EN**: Comment documents nearby script behavior: `SA-NEXT: echo "pass test output"`.
  **L719 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: echo "pass test output"`。
- **L720 EN**: Comment documents nearby script behavior: `SA-NEXT: # executed command: echo 'pass test output'`.
  **L720 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: # executed command: echo 'pass test output'`。
- **L721 EN**: Comment documents nearby script behavior: `SA-NEXT: # .---command stdout`.
  **L721 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: # .---command stdout`。
- **L722 EN**: Comment documents nearby script behavior: `SA-NEXT: # | pass test output`.
  **L722 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: # | pass test output`。
- **L723 EN**: Comment documents nearby script behavior: `SA-NEXT: # \``.
  **L723 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: # \``。
- **L724 EN**: Comment documents nearby script behavior: `SA-EMPTY:`.
  **L724 CN**: 注释说明了附近脚本逻辑：`SA-EMPTY:`。
- **L725 EN**: Comment documents nearby script behavior: `SA-NEXT:`.
  **L725 CN**: 注释说明了附近脚本逻辑：`SA-NEXT:`。
- **L726 EN**: Comment documents nearby script behavior: `SA-EMPTY:`.
  **L726 CN**: 注释说明了附近脚本逻辑：`SA-EMPTY:`。
- **L727 EN**: Comment documents nearby script behavior: `SA-NEXT: ********************`.
  **L727 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: ********************`。
- **L728 EN**: Comment documents nearby script behavior: `SA-NEXT: Testing:`.
  **L728 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: Testing:`。
- **L729 EN**: Comment documents nearby script behavior: `SA-NEXT: {{UN}}SUPPORTED: verbosity :: unsupported.txt (3 of 5)`.
  **L729 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: {{UN}}SUPPORTED: verbosity :: unsupported.txt (3 of 5)`。
- **L730 EN**: Comment documents nearby script behavior: `SA-NEXT: Test requires the following unavailable features: asdf`.
  **L730 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: Test requires the following unavailable features: asdf`。
- **L731 EN**: Comment documents nearby script behavior: `SA-NEXT: ********************`.
  **L731 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: ********************`。
- **L732 EN**: Comment documents nearby script behavior: `SA-NEXT: Testing:`.
  **L732 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: Testing:`。
- **L733 EN**: Comment documents nearby script behavior: `SA-NEXT: {{X}}FAIL: verbosity :: xfail.txt (4 of 5)`.
  **L733 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: {{X}}FAIL: verbosity :: xfail.txt (4 of 5)`。
- **L734 EN**: Comment documents nearby script behavior: `SA-NEXT: Exit Code: 1`.
  **L734 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: Exit Code: 1`。
- **L735 EN**: Comment documents nearby script behavior: `SA-EMPTY:`.
  **L735 CN**: 注释说明了附近脚本逻辑：`SA-EMPTY:`。
- **L736 EN**: Comment documents nearby script behavior: `SA-NEXT: Command Output (stdout):`.
  **L736 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: Command Output (stdout):`。
- **L737 EN**: Comment documents nearby script behavior: `SA-NEXT:`.
  **L737 CN**: 注释说明了附近脚本逻辑：`SA-NEXT:`。
- **L738 EN**: Comment documents nearby script behavior: `SA-NEXT: # {{R}}UN: at line 2`.
  **L738 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: # {{R}}UN: at line 2`。
- **L739 EN**: Comment documents nearby script behavior: `SA-NEXT: not echo "xfail test output"`.
  **L739 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: not echo "xfail test output"`。
- **L740 EN**: Comment documents nearby script behavior: `SA-NEXT: # executed command: not echo 'xfail test output'`.
  **L740 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: # executed command: not echo 'xfail test output'`。
- **L741 EN**: Comment documents nearby script behavior: `SA-NEXT: # .---command stdout`.
  **L741 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: # .---command stdout`。
- **L742 EN**: Comment documents nearby script behavior: `SA-NEXT: # | xfail test output`.
  **L742 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: # | xfail test output`。

### Lines 743-770

````python
# SA-NEXT: # `-----------------------------
# SA-NEXT: # error: command failed with exit status: 1
# SA-EMPTY:
# SA-NEXT: --
# SA-EMPTY:
# SA-NEXT: ********************
# SA-NEXT: Testing:
# SA-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)
# SA-NEXT: ******************** TEST 'verbosity :: xpass.txt' FAILED ********************
# SA-NEXT: Exit Code: 0
# SA-EMPTY:
# SA-NEXT: Command Output (stdout):
# SA-NEXT: --
# SA-NEXT: # {{R}}UN: at line 2
# SA-NEXT: echo "xpass test output"
# SA-NEXT: # executed command: echo 'xpass test output'
# SA-NEXT: # .---command stdout------------
# SA-NEXT: # | xpass test output
# SA-NEXT: # `-----------------------------
# SA-EMPTY:
# SA-NEXT: --
# SA-EMPTY:
# SA-NEXT: ********************
# SA-NEXT: Testing:
# SA-NEXT: ********************
# SA-NEXT: Failed Tests (1):
# SA-NEXT:   verbosity :: fail.txt
# SA-EMPTY:
````
- **L743 EN**: Comment documents nearby script behavior: `SA-NEXT: # \``.
  **L743 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: # \``。
- **L744 EN**: Comment documents nearby script behavior: `SA-NEXT: # error: command failed with exit status: 1`.
  **L744 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: # error: command failed with exit status: 1`。
- **L745 EN**: Comment documents nearby script behavior: `SA-EMPTY:`.
  **L745 CN**: 注释说明了附近脚本逻辑：`SA-EMPTY:`。
- **L746 EN**: Comment documents nearby script behavior: `SA-NEXT:`.
  **L746 CN**: 注释说明了附近脚本逻辑：`SA-NEXT:`。
- **L747 EN**: Comment documents nearby script behavior: `SA-EMPTY:`.
  **L747 CN**: 注释说明了附近脚本逻辑：`SA-EMPTY:`。
- **L748 EN**: Comment documents nearby script behavior: `SA-NEXT: ********************`.
  **L748 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: ********************`。
- **L749 EN**: Comment documents nearby script behavior: `SA-NEXT: Testing:`.
  **L749 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: Testing:`。
- **L750 EN**: Comment documents nearby script behavior: `SA-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)`.
  **L750 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)`。
- **L751 EN**: Comment documents nearby script behavior: `SA-NEXT: ******************** TEST 'verbosity :: xpass.txt' FAILED ********************`.
  **L751 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: ******************** TEST 'verbosity :: xpass.txt' FAILED ********************`。
- **L752 EN**: Comment documents nearby script behavior: `SA-NEXT: Exit Code: 0`.
  **L752 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: Exit Code: 0`。
- **L753 EN**: Comment documents nearby script behavior: `SA-EMPTY:`.
  **L753 CN**: 注释说明了附近脚本逻辑：`SA-EMPTY:`。
- **L754 EN**: Comment documents nearby script behavior: `SA-NEXT: Command Output (stdout):`.
  **L754 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: Command Output (stdout):`。
- **L755 EN**: Comment documents nearby script behavior: `SA-NEXT:`.
  **L755 CN**: 注释说明了附近脚本逻辑：`SA-NEXT:`。
- **L756 EN**: Comment documents nearby script behavior: `SA-NEXT: # {{R}}UN: at line 2`.
  **L756 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: # {{R}}UN: at line 2`。
- **L757 EN**: Comment documents nearby script behavior: `SA-NEXT: echo "xpass test output"`.
  **L757 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: echo "xpass test output"`。
- **L758 EN**: Comment documents nearby script behavior: `SA-NEXT: # executed command: echo 'xpass test output'`.
  **L758 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: # executed command: echo 'xpass test output'`。
- **L759 EN**: Comment documents nearby script behavior: `SA-NEXT: # .---command stdout`.
  **L759 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: # .---command stdout`。
- **L760 EN**: Comment documents nearby script behavior: `SA-NEXT: # | xpass test output`.
  **L760 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: # | xpass test output`。
- **L761 EN**: Comment documents nearby script behavior: `SA-NEXT: # \``.
  **L761 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: # \``。
- **L762 EN**: Comment documents nearby script behavior: `SA-EMPTY:`.
  **L762 CN**: 注释说明了附近脚本逻辑：`SA-EMPTY:`。
- **L763 EN**: Comment documents nearby script behavior: `SA-NEXT:`.
  **L763 CN**: 注释说明了附近脚本逻辑：`SA-NEXT:`。
- **L764 EN**: Comment documents nearby script behavior: `SA-EMPTY:`.
  **L764 CN**: 注释说明了附近脚本逻辑：`SA-EMPTY:`。
- **L765 EN**: Comment documents nearby script behavior: `SA-NEXT: ********************`.
  **L765 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: ********************`。
- **L766 EN**: Comment documents nearby script behavior: `SA-NEXT: Testing:`.
  **L766 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: Testing:`。
- **L767 EN**: Comment documents nearby script behavior: `SA-NEXT: ********************`.
  **L767 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: ********************`。
- **L768 EN**: Comment documents nearby script behavior: `SA-NEXT: Failed Tests (1):`.
  **L768 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: Failed Tests (1):`。
- **L769 EN**: Comment documents nearby script behavior: `SA-NEXT: verbosity :: fail.txt`.
  **L769 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: verbosity :: fail.txt`。
- **L770 EN**: Comment documents nearby script behavior: `SA-EMPTY:`.
  **L770 CN**: 注释说明了附近脚本逻辑：`SA-EMPTY:`。

### Lines 771-784

````python
# SA-NEXT: ********************
# SA-NEXT: Unexpectedly Passed Tests (1):
# SA-NEXT:   verbosity :: xpass.txt
# SA-EMPTY:
# SA-EMPTY:
# SA-NEXT: Testing Time: {{.*}}s
# SA-EMPTY:
# SA-NEXT: Total Discovered Tests: 5
# SA-NEXT:   Unsupported        : 1 (20.00%)
# SA-NEXT:   Passed             : 1 (20.00%)
# SA-NEXT:   Expectedly Failed  : 1 (20.00%)
# SA-NEXT:   Failed             : 1 (20.00%)
# SA-NEXT:   Unexpectedly Passed: 1 (20.00%)

````
- **L771 EN**: Comment documents nearby script behavior: `SA-NEXT: ********************`.
  **L771 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: ********************`。
- **L772 EN**: Comment documents nearby script behavior: `SA-NEXT: Unexpectedly Passed Tests (1):`.
  **L772 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: Unexpectedly Passed Tests (1):`。
- **L773 EN**: Comment documents nearby script behavior: `SA-NEXT: verbosity :: xpass.txt`.
  **L773 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: verbosity :: xpass.txt`。
- **L774 EN**: Comment documents nearby script behavior: `SA-EMPTY:`.
  **L774 CN**: 注释说明了附近脚本逻辑：`SA-EMPTY:`。
- **L775 EN**: Comment documents nearby script behavior: `SA-EMPTY:`.
  **L775 CN**: 注释说明了附近脚本逻辑：`SA-EMPTY:`。
- **L776 EN**: Comment documents nearby script behavior: `SA-NEXT: Testing Time: {{.*}}s`.
  **L776 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: Testing Time: {{.*}}s`。
- **L777 EN**: Comment documents nearby script behavior: `SA-EMPTY:`.
  **L777 CN**: 注释说明了附近脚本逻辑：`SA-EMPTY:`。
- **L778 EN**: Comment documents nearby script behavior: `SA-NEXT: Total Discovered Tests: 5`.
  **L778 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: Total Discovered Tests: 5`。
- **L779 EN**: Comment documents nearby script behavior: `SA-NEXT: Unsupported : 1 (20.00%)`.
  **L779 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: Unsupported : 1 (20.00%)`。
- **L780 EN**: Comment documents nearby script behavior: `SA-NEXT: Passed : 1 (20.00%)`.
  **L780 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: Passed : 1 (20.00%)`。
- **L781 EN**: Comment documents nearby script behavior: `SA-NEXT: Expectedly Failed : 1 (20.00%)`.
  **L781 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: Expectedly Failed : 1 (20.00%)`。
- **L782 EN**: Comment documents nearby script behavior: `SA-NEXT: Failed : 1 (20.00%)`.
  **L782 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: Failed : 1 (20.00%)`。
- **L783 EN**: Comment documents nearby script behavior: `SA-NEXT: Unexpectedly Passed: 1 (20.00%)`.
  **L783 CN**: 注释说明了附近脚本逻辑：`SA-NEXT: Unexpectedly Passed: 1 (20.00%)`。
- **L784 EN**: Blank line separates nearby declarations or logic blocks.
  **L784 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 785-812

````python

# RUN: not %{lit} -q -a %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt
# RUN: FileCheck %s --check-prefix QA < %t/stdout.txt
# RUN: FileCheck %s --check-prefix QUIET-ERR --implicit-check-not lit < %t/stderr.txt

# QA:      -- Testing: 5 tests, 1 workers --
# QA-NEXT: FAIL: verbosity :: fail.txt (1 of 5)
# QA-NEXT: ******************** TEST 'verbosity :: fail.txt' FAILED ********************
# QA-NEXT: Exit Code: 127
# QA-EMPTY:
# QA-NEXT: Command Output (stdout):
# QA-NEXT: --
# QA-NEXT: # {{R}}UN: at line 1
# QA-NEXT: echo "fail test output"
# QA-NEXT: # executed command: echo 'fail test output'
# QA-NEXT: # .---command stdout------------
# QA-NEXT: # | fail test output
# QA-NEXT: # `-----------------------------
# QA-NEXT: # {{R}}UN: at line 2
# QA-NEXT: fail
# QA-NEXT: # executed command: fail
# QA-NEXT: # .---command stderr------------
# QA-NEXT: # | 'fail': command not found
# QA-NEXT: # `-----------------------------
# QA-NEXT: # error: command failed with exit status: 127
# QA-EMPTY:
# QA-NEXT: --
# QA-EMPTY:
````
- **L785 EN**: Blank line separates nearby declarations or logic blocks.
  **L785 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L786 EN**: Comment documents nearby script behavior: `RUN: not %{lit} -q -a %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt`.
  **L786 CN**: 注释说明了附近脚本逻辑：`RUN: not %{lit} -q -a %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt`。
- **L787 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix QA < %t/stdout.txt`.
  **L787 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix QA < %t/stdout.txt`。
- **L788 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix QUIET-ERR --implicit-check-not lit < %t/stderr.txt`.
  **L788 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix QUIET-ERR --implicit-check-not lit < %t/stderr.txt`。
- **L789 EN**: Blank line separates nearby declarations or logic blocks.
  **L789 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L790 EN**: Comment documents nearby script behavior: `QA: -- Testing: 5 tests, 1 workers`.
  **L790 CN**: 注释说明了附近脚本逻辑：`QA: -- Testing: 5 tests, 1 workers`。
- **L791 EN**: Comment documents nearby script behavior: `QA-NEXT: FAIL: verbosity :: fail.txt (1 of 5)`.
  **L791 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: FAIL: verbosity :: fail.txt (1 of 5)`。
- **L792 EN**: Comment documents nearby script behavior: `QA-NEXT: ******************** TEST 'verbosity :: fail.txt' FAILED ********************`.
  **L792 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: ******************** TEST 'verbosity :: fail.txt' FAILED ********************`。
- **L793 EN**: Comment documents nearby script behavior: `QA-NEXT: Exit Code: 127`.
  **L793 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: Exit Code: 127`。
- **L794 EN**: Comment documents nearby script behavior: `QA-EMPTY:`.
  **L794 CN**: 注释说明了附近脚本逻辑：`QA-EMPTY:`。
- **L795 EN**: Comment documents nearby script behavior: `QA-NEXT: Command Output (stdout):`.
  **L795 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: Command Output (stdout):`。
- **L796 EN**: Comment documents nearby script behavior: `QA-NEXT:`.
  **L796 CN**: 注释说明了附近脚本逻辑：`QA-NEXT:`。
- **L797 EN**: Comment documents nearby script behavior: `QA-NEXT: # {{R}}UN: at line 1`.
  **L797 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: # {{R}}UN: at line 1`。
- **L798 EN**: Comment documents nearby script behavior: `QA-NEXT: echo "fail test output"`.
  **L798 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: echo "fail test output"`。
- **L799 EN**: Comment documents nearby script behavior: `QA-NEXT: # executed command: echo 'fail test output'`.
  **L799 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: # executed command: echo 'fail test output'`。
- **L800 EN**: Comment documents nearby script behavior: `QA-NEXT: # .---command stdout`.
  **L800 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: # .---command stdout`。
- **L801 EN**: Comment documents nearby script behavior: `QA-NEXT: # | fail test output`.
  **L801 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: # | fail test output`。
- **L802 EN**: Comment documents nearby script behavior: `QA-NEXT: # \``.
  **L802 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: # \``。
- **L803 EN**: Comment documents nearby script behavior: `QA-NEXT: # {{R}}UN: at line 2`.
  **L803 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: # {{R}}UN: at line 2`。
- **L804 EN**: Comment documents nearby script behavior: `QA-NEXT: fail`.
  **L804 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: fail`。
- **L805 EN**: Comment documents nearby script behavior: `QA-NEXT: # executed command: fail`.
  **L805 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: # executed command: fail`。
- **L806 EN**: Comment documents nearby script behavior: `QA-NEXT: # .---command stderr`.
  **L806 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: # .---command stderr`。
- **L807 EN**: Comment documents nearby script behavior: `QA-NEXT: # | 'fail': command not found`.
  **L807 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: # | 'fail': command not found`。
- **L808 EN**: Comment documents nearby script behavior: `QA-NEXT: # \``.
  **L808 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: # \``。
- **L809 EN**: Comment documents nearby script behavior: `QA-NEXT: # error: command failed with exit status: 127`.
  **L809 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: # error: command failed with exit status: 127`。
- **L810 EN**: Comment documents nearby script behavior: `QA-EMPTY:`.
  **L810 CN**: 注释说明了附近脚本逻辑：`QA-EMPTY:`。
- **L811 EN**: Comment documents nearby script behavior: `QA-NEXT:`.
  **L811 CN**: 注释说明了附近脚本逻辑：`QA-NEXT:`。
- **L812 EN**: Comment documents nearby script behavior: `QA-EMPTY:`.
  **L812 CN**: 注释说明了附近脚本逻辑：`QA-EMPTY:`。

### Lines 813-840

````python
# QA-NEXT: ********************
# QA-NEXT: PASS: verbosity :: pass.txt (2 of 5)
# QA-NEXT: Exit Code: 0
# QA-EMPTY:
# QA-NEXT: Command Output (stdout):
# QA-NEXT: --
# QA-NEXT: # {{R}}UN: at line 1
# QA-NEXT: echo "pass test output"
# QA-NEXT: # executed command: echo 'pass test output'
# QA-NEXT: # .---command stdout------------
# QA-NEXT: # | pass test output
# QA-NEXT: # `-----------------------------
# QA-EMPTY:
# QA-NEXT: --
# QA-EMPTY:
# QA-NEXT: ********************
# QA-NEXT: {{UN}}SUPPORTED: verbosity :: unsupported.txt (3 of 5)
# QA-NEXT: Test requires the following unavailable features: asdf
# QA-NEXT: ********************
# QA-NEXT: {{X}}FAIL: verbosity :: xfail.txt (4 of 5)
# QA-NEXT: Exit Code: 1
# QA-EMPTY:
# QA-NEXT: Command Output (stdout):
# QA-NEXT: --
# QA-NEXT: # {{R}}UN: at line 2
# QA-NEXT: not echo "xfail test output"
# QA-NEXT: # executed command: not echo 'xfail test output'
# QA-NEXT: # .---command stdout------------
````
- **L813 EN**: Comment documents nearby script behavior: `QA-NEXT: ********************`.
  **L813 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: ********************`。
- **L814 EN**: Comment documents nearby script behavior: `QA-NEXT: PASS: verbosity :: pass.txt (2 of 5)`.
  **L814 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: PASS: verbosity :: pass.txt (2 of 5)`。
- **L815 EN**: Comment documents nearby script behavior: `QA-NEXT: Exit Code: 0`.
  **L815 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: Exit Code: 0`。
- **L816 EN**: Comment documents nearby script behavior: `QA-EMPTY:`.
  **L816 CN**: 注释说明了附近脚本逻辑：`QA-EMPTY:`。
- **L817 EN**: Comment documents nearby script behavior: `QA-NEXT: Command Output (stdout):`.
  **L817 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: Command Output (stdout):`。
- **L818 EN**: Comment documents nearby script behavior: `QA-NEXT:`.
  **L818 CN**: 注释说明了附近脚本逻辑：`QA-NEXT:`。
- **L819 EN**: Comment documents nearby script behavior: `QA-NEXT: # {{R}}UN: at line 1`.
  **L819 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: # {{R}}UN: at line 1`。
- **L820 EN**: Comment documents nearby script behavior: `QA-NEXT: echo "pass test output"`.
  **L820 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: echo "pass test output"`。
- **L821 EN**: Comment documents nearby script behavior: `QA-NEXT: # executed command: echo 'pass test output'`.
  **L821 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: # executed command: echo 'pass test output'`。
- **L822 EN**: Comment documents nearby script behavior: `QA-NEXT: # .---command stdout`.
  **L822 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: # .---command stdout`。
- **L823 EN**: Comment documents nearby script behavior: `QA-NEXT: # | pass test output`.
  **L823 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: # | pass test output`。
- **L824 EN**: Comment documents nearby script behavior: `QA-NEXT: # \``.
  **L824 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: # \``。
- **L825 EN**: Comment documents nearby script behavior: `QA-EMPTY:`.
  **L825 CN**: 注释说明了附近脚本逻辑：`QA-EMPTY:`。
- **L826 EN**: Comment documents nearby script behavior: `QA-NEXT:`.
  **L826 CN**: 注释说明了附近脚本逻辑：`QA-NEXT:`。
- **L827 EN**: Comment documents nearby script behavior: `QA-EMPTY:`.
  **L827 CN**: 注释说明了附近脚本逻辑：`QA-EMPTY:`。
- **L828 EN**: Comment documents nearby script behavior: `QA-NEXT: ********************`.
  **L828 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: ********************`。
- **L829 EN**: Comment documents nearby script behavior: `QA-NEXT: {{UN}}SUPPORTED: verbosity :: unsupported.txt (3 of 5)`.
  **L829 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: {{UN}}SUPPORTED: verbosity :: unsupported.txt (3 of 5)`。
- **L830 EN**: Comment documents nearby script behavior: `QA-NEXT: Test requires the following unavailable features: asdf`.
  **L830 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: Test requires the following unavailable features: asdf`。
- **L831 EN**: Comment documents nearby script behavior: `QA-NEXT: ********************`.
  **L831 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: ********************`。
- **L832 EN**: Comment documents nearby script behavior: `QA-NEXT: {{X}}FAIL: verbosity :: xfail.txt (4 of 5)`.
  **L832 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: {{X}}FAIL: verbosity :: xfail.txt (4 of 5)`。
- **L833 EN**: Comment documents nearby script behavior: `QA-NEXT: Exit Code: 1`.
  **L833 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: Exit Code: 1`。
- **L834 EN**: Comment documents nearby script behavior: `QA-EMPTY:`.
  **L834 CN**: 注释说明了附近脚本逻辑：`QA-EMPTY:`。
- **L835 EN**: Comment documents nearby script behavior: `QA-NEXT: Command Output (stdout):`.
  **L835 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: Command Output (stdout):`。
- **L836 EN**: Comment documents nearby script behavior: `QA-NEXT:`.
  **L836 CN**: 注释说明了附近脚本逻辑：`QA-NEXT:`。
- **L837 EN**: Comment documents nearby script behavior: `QA-NEXT: # {{R}}UN: at line 2`.
  **L837 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: # {{R}}UN: at line 2`。
- **L838 EN**: Comment documents nearby script behavior: `QA-NEXT: not echo "xfail test output"`.
  **L838 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: not echo "xfail test output"`。
- **L839 EN**: Comment documents nearby script behavior: `QA-NEXT: # executed command: not echo 'xfail test output'`.
  **L839 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: # executed command: not echo 'xfail test output'`。
- **L840 EN**: Comment documents nearby script behavior: `QA-NEXT: # .---command stdout`.
  **L840 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: # .---command stdout`。

### Lines 841-868

````python
# QA-NEXT: # | xfail test output
# QA-NEXT: # `-----------------------------
# QA-NEXT: # error: command failed with exit status: 1
# QA-EMPTY:
# QA-NEXT: --
# QA-EMPTY:
# QA-NEXT: ********************
# QA-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)
# QA-NEXT: ******************** TEST 'verbosity :: xpass.txt' FAILED ********************
# QA-NEXT: Exit Code: 0
# QA-EMPTY:
# QA-NEXT: Command Output (stdout):
# QA-NEXT: --
# QA-NEXT: # {{R}}UN: at line 2
# QA-NEXT: echo "xpass test output"
# QA-NEXT: # executed command: echo 'xpass test output'
# QA-NEXT: # .---command stdout------------
# QA-NEXT: # | xpass test output
# QA-NEXT: # `-----------------------------
# QA-EMPTY:
# QA-NEXT: --
# QA-EMPTY:
# QA-NEXT: ********************
# QA-NEXT: ********************
# QA-NEXT: Failed Tests (1):
# QA-NEXT:   verbosity :: fail.txt
# QA-EMPTY:
# QA-NEXT: ********************
````
- **L841 EN**: Comment documents nearby script behavior: `QA-NEXT: # | xfail test output`.
  **L841 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: # | xfail test output`。
- **L842 EN**: Comment documents nearby script behavior: `QA-NEXT: # \``.
  **L842 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: # \``。
- **L843 EN**: Comment documents nearby script behavior: `QA-NEXT: # error: command failed with exit status: 1`.
  **L843 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: # error: command failed with exit status: 1`。
- **L844 EN**: Comment documents nearby script behavior: `QA-EMPTY:`.
  **L844 CN**: 注释说明了附近脚本逻辑：`QA-EMPTY:`。
- **L845 EN**: Comment documents nearby script behavior: `QA-NEXT:`.
  **L845 CN**: 注释说明了附近脚本逻辑：`QA-NEXT:`。
- **L846 EN**: Comment documents nearby script behavior: `QA-EMPTY:`.
  **L846 CN**: 注释说明了附近脚本逻辑：`QA-EMPTY:`。
- **L847 EN**: Comment documents nearby script behavior: `QA-NEXT: ********************`.
  **L847 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: ********************`。
- **L848 EN**: Comment documents nearby script behavior: `QA-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)`.
  **L848 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)`。
- **L849 EN**: Comment documents nearby script behavior: `QA-NEXT: ******************** TEST 'verbosity :: xpass.txt' FAILED ********************`.
  **L849 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: ******************** TEST 'verbosity :: xpass.txt' FAILED ********************`。
- **L850 EN**: Comment documents nearby script behavior: `QA-NEXT: Exit Code: 0`.
  **L850 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: Exit Code: 0`。
- **L851 EN**: Comment documents nearby script behavior: `QA-EMPTY:`.
  **L851 CN**: 注释说明了附近脚本逻辑：`QA-EMPTY:`。
- **L852 EN**: Comment documents nearby script behavior: `QA-NEXT: Command Output (stdout):`.
  **L852 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: Command Output (stdout):`。
- **L853 EN**: Comment documents nearby script behavior: `QA-NEXT:`.
  **L853 CN**: 注释说明了附近脚本逻辑：`QA-NEXT:`。
- **L854 EN**: Comment documents nearby script behavior: `QA-NEXT: # {{R}}UN: at line 2`.
  **L854 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: # {{R}}UN: at line 2`。
- **L855 EN**: Comment documents nearby script behavior: `QA-NEXT: echo "xpass test output"`.
  **L855 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: echo "xpass test output"`。
- **L856 EN**: Comment documents nearby script behavior: `QA-NEXT: # executed command: echo 'xpass test output'`.
  **L856 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: # executed command: echo 'xpass test output'`。
- **L857 EN**: Comment documents nearby script behavior: `QA-NEXT: # .---command stdout`.
  **L857 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: # .---command stdout`。
- **L858 EN**: Comment documents nearby script behavior: `QA-NEXT: # | xpass test output`.
  **L858 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: # | xpass test output`。
- **L859 EN**: Comment documents nearby script behavior: `QA-NEXT: # \``.
  **L859 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: # \``。
- **L860 EN**: Comment documents nearby script behavior: `QA-EMPTY:`.
  **L860 CN**: 注释说明了附近脚本逻辑：`QA-EMPTY:`。
- **L861 EN**: Comment documents nearby script behavior: `QA-NEXT:`.
  **L861 CN**: 注释说明了附近脚本逻辑：`QA-NEXT:`。
- **L862 EN**: Comment documents nearby script behavior: `QA-EMPTY:`.
  **L862 CN**: 注释说明了附近脚本逻辑：`QA-EMPTY:`。
- **L863 EN**: Comment documents nearby script behavior: `QA-NEXT: ********************`.
  **L863 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: ********************`。
- **L864 EN**: Comment documents nearby script behavior: `QA-NEXT: ********************`.
  **L864 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: ********************`。
- **L865 EN**: Comment documents nearby script behavior: `QA-NEXT: Failed Tests (1):`.
  **L865 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: Failed Tests (1):`。
- **L866 EN**: Comment documents nearby script behavior: `QA-NEXT: verbosity :: fail.txt`.
  **L866 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: verbosity :: fail.txt`。
- **L867 EN**: Comment documents nearby script behavior: `QA-EMPTY:`.
  **L867 CN**: 注释说明了附近脚本逻辑：`QA-EMPTY:`。
- **L868 EN**: Comment documents nearby script behavior: `QA-NEXT: ********************`.
  **L868 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: ********************`。

### Lines 869-884

````python
# QA-NEXT: Unexpectedly Passed Tests (1):
# QA-NEXT:   verbosity :: xpass.txt
# QA-EMPTY:
# QA-EMPTY:
# QA-NEXT: Total Discovered Tests: 5
# QA-NEXT:   Failed             : 1 (20.00%)
# QA-NEXT:   Unexpectedly Passed: 1 (20.00%)

# RUN: not %{lit} -a -q %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt
# RUN: FileCheck %s --check-prefix QUIET < %t/stdout.txt
# RUN: FileCheck %s --check-prefix QUIET-ERR --implicit-check-not lit < %t/stderr.txt

# RUN: not %{lit} -sqav %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt
# RUN: FileCheck %s --check-prefix SQAV < %t/stdout.txt
# RUN: FileCheck %s --check-prefix QUIET-ERR --implicit-check-not lit < %t/stderr.txt

````
- **L869 EN**: Comment documents nearby script behavior: `QA-NEXT: Unexpectedly Passed Tests (1):`.
  **L869 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: Unexpectedly Passed Tests (1):`。
- **L870 EN**: Comment documents nearby script behavior: `QA-NEXT: verbosity :: xpass.txt`.
  **L870 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: verbosity :: xpass.txt`。
- **L871 EN**: Comment documents nearby script behavior: `QA-EMPTY:`.
  **L871 CN**: 注释说明了附近脚本逻辑：`QA-EMPTY:`。
- **L872 EN**: Comment documents nearby script behavior: `QA-EMPTY:`.
  **L872 CN**: 注释说明了附近脚本逻辑：`QA-EMPTY:`。
- **L873 EN**: Comment documents nearby script behavior: `QA-NEXT: Total Discovered Tests: 5`.
  **L873 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: Total Discovered Tests: 5`。
- **L874 EN**: Comment documents nearby script behavior: `QA-NEXT: Failed : 1 (20.00%)`.
  **L874 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: Failed : 1 (20.00%)`。
- **L875 EN**: Comment documents nearby script behavior: `QA-NEXT: Unexpectedly Passed: 1 (20.00%)`.
  **L875 CN**: 注释说明了附近脚本逻辑：`QA-NEXT: Unexpectedly Passed: 1 (20.00%)`。
- **L876 EN**: Blank line separates nearby declarations or logic blocks.
  **L876 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L877 EN**: Comment documents nearby script behavior: `RUN: not %{lit} -a -q %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt`.
  **L877 CN**: 注释说明了附近脚本逻辑：`RUN: not %{lit} -a -q %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt`。
- **L878 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix QUIET < %t/stdout.txt`.
  **L878 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix QUIET < %t/stdout.txt`。
- **L879 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix QUIET-ERR --implicit-check-not lit < %t/stderr.txt`.
  **L879 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix QUIET-ERR --implicit-check-not lit < %t/stderr.txt`。
- **L880 EN**: Blank line separates nearby declarations or logic blocks.
  **L880 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L881 EN**: Comment documents nearby script behavior: `RUN: not %{lit} -sqav %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt`.
  **L881 CN**: 注释说明了附近脚本逻辑：`RUN: not %{lit} -sqav %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt`。
- **L882 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix SQAV < %t/stdout.txt`.
  **L882 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix SQAV < %t/stdout.txt`。
- **L883 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix QUIET-ERR --implicit-check-not lit < %t/stderr.txt`.
  **L883 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix QUIET-ERR --implicit-check-not lit < %t/stderr.txt`。
- **L884 EN**: Blank line separates nearby declarations or logic blocks.
  **L884 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 885-912

````python
# SQAV:      -- Testing: 5 tests, 1 workers --
# SQAV-NEXT: Testing:
# SQAV-NEXT: FAIL: verbosity :: fail.txt (1 of 5)
# SQAV-NEXT: ******************** TEST 'verbosity :: fail.txt' FAILED ********************
# SQAV-NEXT: Exit Code: 127
# SQAV-EMPTY:
# SQAV-NEXT: Command Output (stdout):
# SQAV-NEXT: --
# SQAV-NEXT: # {{R}}UN: at line 1
# SQAV-NEXT: echo "fail test output"
# SQAV-NEXT: # executed command: echo 'fail test output'
# SQAV-NEXT: # .---command stdout------------
# SQAV-NEXT: # | fail test output
# SQAV-NEXT: # `-----------------------------
# SQAV-NEXT: # {{R}}UN: at line 2
# SQAV-NEXT: fail
# SQAV-NEXT: # executed command: fail
# SQAV-NEXT: # .---command stderr------------
# SQAV-NEXT: # | 'fail': command not found
# SQAV-NEXT: # `-----------------------------
# SQAV-NEXT: # error: command failed with exit status: 127
# SQAV-EMPTY:
# SQAV-NEXT: --
# SQAV-EMPTY:
# SQAV-NEXT: ********************
# SQAV-NEXT: Testing:
# SQAV-NEXT: PASS: verbosity :: pass.txt (2 of 5)
# SQAV-NEXT: Testing:
````
- **L885 EN**: Comment documents nearby script behavior: `SQAV: -- Testing: 5 tests, 1 workers`.
  **L885 CN**: 注释说明了附近脚本逻辑：`SQAV: -- Testing: 5 tests, 1 workers`。
- **L886 EN**: Comment documents nearby script behavior: `SQAV-NEXT: Testing:`.
  **L886 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT: Testing:`。
- **L887 EN**: Comment documents nearby script behavior: `SQAV-NEXT: FAIL: verbosity :: fail.txt (1 of 5)`.
  **L887 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT: FAIL: verbosity :: fail.txt (1 of 5)`。
- **L888 EN**: Comment documents nearby script behavior: `SQAV-NEXT: ******************** TEST 'verbosity :: fail.txt' FAILED ********************`.
  **L888 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT: ******************** TEST 'verbosity :: fail.txt' FAILED ********************`。
- **L889 EN**: Comment documents nearby script behavior: `SQAV-NEXT: Exit Code: 127`.
  **L889 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT: Exit Code: 127`。
- **L890 EN**: Comment documents nearby script behavior: `SQAV-EMPTY:`.
  **L890 CN**: 注释说明了附近脚本逻辑：`SQAV-EMPTY:`。
- **L891 EN**: Comment documents nearby script behavior: `SQAV-NEXT: Command Output (stdout):`.
  **L891 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT: Command Output (stdout):`。
- **L892 EN**: Comment documents nearby script behavior: `SQAV-NEXT:`.
  **L892 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT:`。
- **L893 EN**: Comment documents nearby script behavior: `SQAV-NEXT: # {{R}}UN: at line 1`.
  **L893 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT: # {{R}}UN: at line 1`。
- **L894 EN**: Comment documents nearby script behavior: `SQAV-NEXT: echo "fail test output"`.
  **L894 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT: echo "fail test output"`。
- **L895 EN**: Comment documents nearby script behavior: `SQAV-NEXT: # executed command: echo 'fail test output'`.
  **L895 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT: # executed command: echo 'fail test output'`。
- **L896 EN**: Comment documents nearby script behavior: `SQAV-NEXT: # .---command stdout`.
  **L896 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT: # .---command stdout`。
- **L897 EN**: Comment documents nearby script behavior: `SQAV-NEXT: # | fail test output`.
  **L897 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT: # | fail test output`。
- **L898 EN**: Comment documents nearby script behavior: `SQAV-NEXT: # \``.
  **L898 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT: # \``。
- **L899 EN**: Comment documents nearby script behavior: `SQAV-NEXT: # {{R}}UN: at line 2`.
  **L899 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT: # {{R}}UN: at line 2`。
- **L900 EN**: Comment documents nearby script behavior: `SQAV-NEXT: fail`.
  **L900 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT: fail`。
- **L901 EN**: Comment documents nearby script behavior: `SQAV-NEXT: # executed command: fail`.
  **L901 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT: # executed command: fail`。
- **L902 EN**: Comment documents nearby script behavior: `SQAV-NEXT: # .---command stderr`.
  **L902 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT: # .---command stderr`。
- **L903 EN**: Comment documents nearby script behavior: `SQAV-NEXT: # | 'fail': command not found`.
  **L903 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT: # | 'fail': command not found`。
- **L904 EN**: Comment documents nearby script behavior: `SQAV-NEXT: # \``.
  **L904 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT: # \``。
- **L905 EN**: Comment documents nearby script behavior: `SQAV-NEXT: # error: command failed with exit status: 127`.
  **L905 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT: # error: command failed with exit status: 127`。
- **L906 EN**: Comment documents nearby script behavior: `SQAV-EMPTY:`.
  **L906 CN**: 注释说明了附近脚本逻辑：`SQAV-EMPTY:`。
- **L907 EN**: Comment documents nearby script behavior: `SQAV-NEXT:`.
  **L907 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT:`。
- **L908 EN**: Comment documents nearby script behavior: `SQAV-EMPTY:`.
  **L908 CN**: 注释说明了附近脚本逻辑：`SQAV-EMPTY:`。
- **L909 EN**: Comment documents nearby script behavior: `SQAV-NEXT: ********************`.
  **L909 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT: ********************`。
- **L910 EN**: Comment documents nearby script behavior: `SQAV-NEXT: Testing:`.
  **L910 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT: Testing:`。
- **L911 EN**: Comment documents nearby script behavior: `SQAV-NEXT: PASS: verbosity :: pass.txt (2 of 5)`.
  **L911 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT: PASS: verbosity :: pass.txt (2 of 5)`。
- **L912 EN**: Comment documents nearby script behavior: `SQAV-NEXT: Testing:`.
  **L912 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT: Testing:`。

### Lines 913-940

````python
# SQAV-NEXT: {{UN}}SUPPORTED: verbosity :: unsupported.txt (3 of 5)
# SQAV-NEXT: Testing:
# SQAV-NEXT: {{X}}FAIL: verbosity :: xfail.txt (4 of 5)
# SQAV-NEXT: Testing:
# SQAV-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)
# SQAV-NEXT: ******************** TEST 'verbosity :: xpass.txt' FAILED ********************
# SQAV-NEXT: Exit Code: 0
# SQAV-EMPTY:
# SQAV-NEXT: Command Output (stdout):
# SQAV-NEXT: --
# SQAV-NEXT: # {{R}}UN: at line 2
# SQAV-NEXT: echo "xpass test output"
# SQAV-NEXT: # executed command: echo 'xpass test output'
# SQAV-NEXT: # .---command stdout------------
# SQAV-NEXT: # | xpass test output
# SQAV-NEXT: # `-----------------------------
# SQAV-EMPTY:
# SQAV-NEXT: --
# SQAV-EMPTY:
# SQAV-NEXT: ********************
# SQAV-NEXT: Testing:
# SQAV-NEXT: ********************
# SQAV-NEXT: Failed Tests (1):
# SQAV-NEXT:   verbosity :: fail.txt
# SQAV-EMPTY:
# SQAV-NEXT: ********************
# SQAV-NEXT: Unexpectedly Passed Tests (1):
# SQAV-NEXT:   verbosity :: xpass.txt
````
- **L913 EN**: Comment documents nearby script behavior: `SQAV-NEXT: {{UN}}SUPPORTED: verbosity :: unsupported.txt (3 of 5)`.
  **L913 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT: {{UN}}SUPPORTED: verbosity :: unsupported.txt (3 of 5)`。
- **L914 EN**: Comment documents nearby script behavior: `SQAV-NEXT: Testing:`.
  **L914 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT: Testing:`。
- **L915 EN**: Comment documents nearby script behavior: `SQAV-NEXT: {{X}}FAIL: verbosity :: xfail.txt (4 of 5)`.
  **L915 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT: {{X}}FAIL: verbosity :: xfail.txt (4 of 5)`。
- **L916 EN**: Comment documents nearby script behavior: `SQAV-NEXT: Testing:`.
  **L916 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT: Testing:`。
- **L917 EN**: Comment documents nearby script behavior: `SQAV-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)`.
  **L917 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)`。
- **L918 EN**: Comment documents nearby script behavior: `SQAV-NEXT: ******************** TEST 'verbosity :: xpass.txt' FAILED ********************`.
  **L918 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT: ******************** TEST 'verbosity :: xpass.txt' FAILED ********************`。
- **L919 EN**: Comment documents nearby script behavior: `SQAV-NEXT: Exit Code: 0`.
  **L919 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT: Exit Code: 0`。
- **L920 EN**: Comment documents nearby script behavior: `SQAV-EMPTY:`.
  **L920 CN**: 注释说明了附近脚本逻辑：`SQAV-EMPTY:`。
- **L921 EN**: Comment documents nearby script behavior: `SQAV-NEXT: Command Output (stdout):`.
  **L921 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT: Command Output (stdout):`。
- **L922 EN**: Comment documents nearby script behavior: `SQAV-NEXT:`.
  **L922 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT:`。
- **L923 EN**: Comment documents nearby script behavior: `SQAV-NEXT: # {{R}}UN: at line 2`.
  **L923 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT: # {{R}}UN: at line 2`。
- **L924 EN**: Comment documents nearby script behavior: `SQAV-NEXT: echo "xpass test output"`.
  **L924 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT: echo "xpass test output"`。
- **L925 EN**: Comment documents nearby script behavior: `SQAV-NEXT: # executed command: echo 'xpass test output'`.
  **L925 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT: # executed command: echo 'xpass test output'`。
- **L926 EN**: Comment documents nearby script behavior: `SQAV-NEXT: # .---command stdout`.
  **L926 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT: # .---command stdout`。
- **L927 EN**: Comment documents nearby script behavior: `SQAV-NEXT: # | xpass test output`.
  **L927 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT: # | xpass test output`。
- **L928 EN**: Comment documents nearby script behavior: `SQAV-NEXT: # \``.
  **L928 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT: # \``。
- **L929 EN**: Comment documents nearby script behavior: `SQAV-EMPTY:`.
  **L929 CN**: 注释说明了附近脚本逻辑：`SQAV-EMPTY:`。
- **L930 EN**: Comment documents nearby script behavior: `SQAV-NEXT:`.
  **L930 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT:`。
- **L931 EN**: Comment documents nearby script behavior: `SQAV-EMPTY:`.
  **L931 CN**: 注释说明了附近脚本逻辑：`SQAV-EMPTY:`。
- **L932 EN**: Comment documents nearby script behavior: `SQAV-NEXT: ********************`.
  **L932 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT: ********************`。
- **L933 EN**: Comment documents nearby script behavior: `SQAV-NEXT: Testing:`.
  **L933 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT: Testing:`。
- **L934 EN**: Comment documents nearby script behavior: `SQAV-NEXT: ********************`.
  **L934 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT: ********************`。
- **L935 EN**: Comment documents nearby script behavior: `SQAV-NEXT: Failed Tests (1):`.
  **L935 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT: Failed Tests (1):`。
- **L936 EN**: Comment documents nearby script behavior: `SQAV-NEXT: verbosity :: fail.txt`.
  **L936 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT: verbosity :: fail.txt`。
- **L937 EN**: Comment documents nearby script behavior: `SQAV-EMPTY:`.
  **L937 CN**: 注释说明了附近脚本逻辑：`SQAV-EMPTY:`。
- **L938 EN**: Comment documents nearby script behavior: `SQAV-NEXT: ********************`.
  **L938 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT: ********************`。
- **L939 EN**: Comment documents nearby script behavior: `SQAV-NEXT: Unexpectedly Passed Tests (1):`.
  **L939 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT: Unexpectedly Passed Tests (1):`。
- **L940 EN**: Comment documents nearby script behavior: `SQAV-NEXT: verbosity :: xpass.txt`.
  **L940 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT: verbosity :: xpass.txt`。

### Lines 941-968

````python
# SQAV-EMPTY:
# SQAV-EMPTY:
# SQAV-NEXT: Total Discovered Tests: 5
# SQAV-NEXT:   Failed             : 1 (20.00%)
# SQAV-NEXT:   Unexpectedly Passed: 1 (20.00%)


### Aliases with specific overrides

# RUN: not %{lit} --quiet --no-terse-summary %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt
# RUN: FileCheck %s --check-prefix QUIET-W-SUMMARY < %t/stdout.txt
# RUN: FileCheck %s --check-prefix QUIET-ERR --implicit-check-not lit < %t/stderr.txt

# QUIET-W-SUMMARY:      -- Testing: 5 tests, 1 workers --
# QUIET-W-SUMMARY-NEXT: FAIL: verbosity :: fail.txt (1 of 5)
# QUIET-W-SUMMARY-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)
# QUIET-W-SUMMARY-NEXT: ********************
# QUIET-W-SUMMARY-NEXT: Failed Tests (1):
# QUIET-W-SUMMARY-NEXT:   verbosity :: fail.txt
# QUIET-W-SUMMARY-EMPTY:
# QUIET-W-SUMMARY-NEXT: ********************
# QUIET-W-SUMMARY-NEXT: Unexpectedly Passed Tests (1):
# QUIET-W-SUMMARY-NEXT:   verbosity :: xpass.txt
# QUIET-W-SUMMARY-EMPTY:
# QUIET-W-SUMMARY-EMPTY:
# QUIET-W-SUMMARY-NEXT: Testing Time: {{.*}}s
# QUIET-W-SUMMARY-EMPTY:
# QUIET-W-SUMMARY-NEXT: Total Discovered Tests: 5
````
- **L941 EN**: Comment documents nearby script behavior: `SQAV-EMPTY:`.
  **L941 CN**: 注释说明了附近脚本逻辑：`SQAV-EMPTY:`。
- **L942 EN**: Comment documents nearby script behavior: `SQAV-EMPTY:`.
  **L942 CN**: 注释说明了附近脚本逻辑：`SQAV-EMPTY:`。
- **L943 EN**: Comment documents nearby script behavior: `SQAV-NEXT: Total Discovered Tests: 5`.
  **L943 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT: Total Discovered Tests: 5`。
- **L944 EN**: Comment documents nearby script behavior: `SQAV-NEXT: Failed : 1 (20.00%)`.
  **L944 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT: Failed : 1 (20.00%)`。
- **L945 EN**: Comment documents nearby script behavior: `SQAV-NEXT: Unexpectedly Passed: 1 (20.00%)`.
  **L945 CN**: 注释说明了附近脚本逻辑：`SQAV-NEXT: Unexpectedly Passed: 1 (20.00%)`。
- **L946 EN**: Blank line separates nearby declarations or logic blocks.
  **L946 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L947 EN**: Blank line separates nearby declarations or logic blocks.
  **L947 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L948 EN**: Comment documents nearby script behavior: `## Aliases with specific overrides`.
  **L948 CN**: 注释说明了附近脚本逻辑：`## Aliases with specific overrides`。
- **L949 EN**: Blank line separates nearby declarations or logic blocks.
  **L949 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L950 EN**: Comment documents nearby script behavior: `RUN: not %{lit} --quiet --no-terse-summary %{inputs}/verbosity 2> %t/stderr.txt > %t/st...`.
  **L950 CN**: 注释说明了附近脚本逻辑：`RUN: not %{lit} --quiet --no-terse-summary %{inputs}/verbosity 2> %t/stderr.txt > %t/st...`。
- **L951 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix QUIET-W-SUMMARY < %t/stdout.txt`.
  **L951 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix QUIET-W-SUMMARY < %t/stdout.txt`。
- **L952 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix QUIET-ERR --implicit-check-not lit < %t/stderr.txt`.
  **L952 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix QUIET-ERR --implicit-check-not lit < %t/stderr.txt`。
- **L953 EN**: Blank line separates nearby declarations or logic blocks.
  **L953 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L954 EN**: Comment documents nearby script behavior: `QUIET-W-SUMMARY: -- Testing: 5 tests, 1 workers`.
  **L954 CN**: 注释说明了附近脚本逻辑：`QUIET-W-SUMMARY: -- Testing: 5 tests, 1 workers`。
- **L955 EN**: Comment documents nearby script behavior: `QUIET-W-SUMMARY-NEXT: FAIL: verbosity :: fail.txt (1 of 5)`.
  **L955 CN**: 注释说明了附近脚本逻辑：`QUIET-W-SUMMARY-NEXT: FAIL: verbosity :: fail.txt (1 of 5)`。
- **L956 EN**: Comment documents nearby script behavior: `QUIET-W-SUMMARY-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)`.
  **L956 CN**: 注释说明了附近脚本逻辑：`QUIET-W-SUMMARY-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)`。
- **L957 EN**: Comment documents nearby script behavior: `QUIET-W-SUMMARY-NEXT: ********************`.
  **L957 CN**: 注释说明了附近脚本逻辑：`QUIET-W-SUMMARY-NEXT: ********************`。
- **L958 EN**: Comment documents nearby script behavior: `QUIET-W-SUMMARY-NEXT: Failed Tests (1):`.
  **L958 CN**: 注释说明了附近脚本逻辑：`QUIET-W-SUMMARY-NEXT: Failed Tests (1):`。
- **L959 EN**: Comment documents nearby script behavior: `QUIET-W-SUMMARY-NEXT: verbosity :: fail.txt`.
  **L959 CN**: 注释说明了附近脚本逻辑：`QUIET-W-SUMMARY-NEXT: verbosity :: fail.txt`。
- **L960 EN**: Comment documents nearby script behavior: `QUIET-W-SUMMARY-EMPTY:`.
  **L960 CN**: 注释说明了附近脚本逻辑：`QUIET-W-SUMMARY-EMPTY:`。
- **L961 EN**: Comment documents nearby script behavior: `QUIET-W-SUMMARY-NEXT: ********************`.
  **L961 CN**: 注释说明了附近脚本逻辑：`QUIET-W-SUMMARY-NEXT: ********************`。
- **L962 EN**: Comment documents nearby script behavior: `QUIET-W-SUMMARY-NEXT: Unexpectedly Passed Tests (1):`.
  **L962 CN**: 注释说明了附近脚本逻辑：`QUIET-W-SUMMARY-NEXT: Unexpectedly Passed Tests (1):`。
- **L963 EN**: Comment documents nearby script behavior: `QUIET-W-SUMMARY-NEXT: verbosity :: xpass.txt`.
  **L963 CN**: 注释说明了附近脚本逻辑：`QUIET-W-SUMMARY-NEXT: verbosity :: xpass.txt`。
- **L964 EN**: Comment documents nearby script behavior: `QUIET-W-SUMMARY-EMPTY:`.
  **L964 CN**: 注释说明了附近脚本逻辑：`QUIET-W-SUMMARY-EMPTY:`。
- **L965 EN**: Comment documents nearby script behavior: `QUIET-W-SUMMARY-EMPTY:`.
  **L965 CN**: 注释说明了附近脚本逻辑：`QUIET-W-SUMMARY-EMPTY:`。
- **L966 EN**: Comment documents nearby script behavior: `QUIET-W-SUMMARY-NEXT: Testing Time: {{.*}}s`.
  **L966 CN**: 注释说明了附近脚本逻辑：`QUIET-W-SUMMARY-NEXT: Testing Time: {{.*}}s`。
- **L967 EN**: Comment documents nearby script behavior: `QUIET-W-SUMMARY-EMPTY:`.
  **L967 CN**: 注释说明了附近脚本逻辑：`QUIET-W-SUMMARY-EMPTY:`。
- **L968 EN**: Comment documents nearby script behavior: `QUIET-W-SUMMARY-NEXT: Total Discovered Tests: 5`.
  **L968 CN**: 注释说明了附近脚本逻辑：`QUIET-W-SUMMARY-NEXT: Total Discovered Tests: 5`。

### Lines 969-996

````python
# QUIET-W-SUMMARY-NEXT:   Unsupported        : 1 (20.00%)
# QUIET-W-SUMMARY-NEXT:   Passed             : 1 (20.00%)
# QUIET-W-SUMMARY-NEXT:   Expectedly Failed  : 1 (20.00%)
# QUIET-W-SUMMARY-NEXT:   Failed             : 1 (20.00%)
# QUIET-W-SUMMARY-NEXT:   Unexpectedly Passed: 1 (20.00%)


# RUN: not %{lit} --quiet --progress-bar %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt
# RUN: FileCheck %s --check-prefix QUIET-W-PROGRESS < %t/stdout.txt
# RUN: FileCheck %s --check-prefix QUIET-ERR --implicit-check-not lit < %t/stderr.txt

# QUIET-W-PROGRESS: -- Testing: 5 tests, 1 workers --
# QUIET-W-PROGRESS-NEXT: Testing:
# QUIET-W-PROGRESS-NEXT: FAIL: verbosity :: fail.txt (1 of 5)
# QUIET-W-PROGRESS-NEXT: Testing:
# QUIET-W-PROGRESS-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)
# QUIET-W-PROGRESS-NEXT: Testing:
# QUIET-W-PROGRESS-NEXT: ********************
# QUIET-W-PROGRESS-NEXT: Failed Tests (1):
# QUIET-W-PROGRESS-NEXT:   verbosity :: fail.txt
# QUIET-W-PROGRESS-EMPTY:
# QUIET-W-PROGRESS-NEXT: ********************
# QUIET-W-PROGRESS-NEXT: Unexpectedly Passed Tests (1):
# QUIET-W-PROGRESS-NEXT:   verbosity :: xpass.txt
# QUIET-W-PROGRESS-EMPTY:
# QUIET-W-PROGRESS-EMPTY:
# QUIET-W-PROGRESS-NEXT: Total Discovered Tests: 5
# QUIET-W-PROGRESS-NEXT:   Failed             : 1 (20.00%)
````
- **L969 EN**: Comment documents nearby script behavior: `QUIET-W-SUMMARY-NEXT: Unsupported : 1 (20.00%)`.
  **L969 CN**: 注释说明了附近脚本逻辑：`QUIET-W-SUMMARY-NEXT: Unsupported : 1 (20.00%)`。
- **L970 EN**: Comment documents nearby script behavior: `QUIET-W-SUMMARY-NEXT: Passed : 1 (20.00%)`.
  **L970 CN**: 注释说明了附近脚本逻辑：`QUIET-W-SUMMARY-NEXT: Passed : 1 (20.00%)`。
- **L971 EN**: Comment documents nearby script behavior: `QUIET-W-SUMMARY-NEXT: Expectedly Failed : 1 (20.00%)`.
  **L971 CN**: 注释说明了附近脚本逻辑：`QUIET-W-SUMMARY-NEXT: Expectedly Failed : 1 (20.00%)`。
- **L972 EN**: Comment documents nearby script behavior: `QUIET-W-SUMMARY-NEXT: Failed : 1 (20.00%)`.
  **L972 CN**: 注释说明了附近脚本逻辑：`QUIET-W-SUMMARY-NEXT: Failed : 1 (20.00%)`。
- **L973 EN**: Comment documents nearby script behavior: `QUIET-W-SUMMARY-NEXT: Unexpectedly Passed: 1 (20.00%)`.
  **L973 CN**: 注释说明了附近脚本逻辑：`QUIET-W-SUMMARY-NEXT: Unexpectedly Passed: 1 (20.00%)`。
- **L974 EN**: Blank line separates nearby declarations or logic blocks.
  **L974 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L975 EN**: Blank line separates nearby declarations or logic blocks.
  **L975 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L976 EN**: Comment documents nearby script behavior: `RUN: not %{lit} --quiet --progress-bar %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout...`.
  **L976 CN**: 注释说明了附近脚本逻辑：`RUN: not %{lit} --quiet --progress-bar %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout...`。
- **L977 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix QUIET-W-PROGRESS < %t/stdout.txt`.
  **L977 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix QUIET-W-PROGRESS < %t/stdout.txt`。
- **L978 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix QUIET-ERR --implicit-check-not lit < %t/stderr.txt`.
  **L978 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix QUIET-ERR --implicit-check-not lit < %t/stderr.txt`。
- **L979 EN**: Blank line separates nearby declarations or logic blocks.
  **L979 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L980 EN**: Comment documents nearby script behavior: `QUIET-W-PROGRESS: -- Testing: 5 tests, 1 workers`.
  **L980 CN**: 注释说明了附近脚本逻辑：`QUIET-W-PROGRESS: -- Testing: 5 tests, 1 workers`。
- **L981 EN**: Comment documents nearby script behavior: `QUIET-W-PROGRESS-NEXT: Testing:`.
  **L981 CN**: 注释说明了附近脚本逻辑：`QUIET-W-PROGRESS-NEXT: Testing:`。
- **L982 EN**: Comment documents nearby script behavior: `QUIET-W-PROGRESS-NEXT: FAIL: verbosity :: fail.txt (1 of 5)`.
  **L982 CN**: 注释说明了附近脚本逻辑：`QUIET-W-PROGRESS-NEXT: FAIL: verbosity :: fail.txt (1 of 5)`。
- **L983 EN**: Comment documents nearby script behavior: `QUIET-W-PROGRESS-NEXT: Testing:`.
  **L983 CN**: 注释说明了附近脚本逻辑：`QUIET-W-PROGRESS-NEXT: Testing:`。
- **L984 EN**: Comment documents nearby script behavior: `QUIET-W-PROGRESS-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)`.
  **L984 CN**: 注释说明了附近脚本逻辑：`QUIET-W-PROGRESS-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)`。
- **L985 EN**: Comment documents nearby script behavior: `QUIET-W-PROGRESS-NEXT: Testing:`.
  **L985 CN**: 注释说明了附近脚本逻辑：`QUIET-W-PROGRESS-NEXT: Testing:`。
- **L986 EN**: Comment documents nearby script behavior: `QUIET-W-PROGRESS-NEXT: ********************`.
  **L986 CN**: 注释说明了附近脚本逻辑：`QUIET-W-PROGRESS-NEXT: ********************`。
- **L987 EN**: Comment documents nearby script behavior: `QUIET-W-PROGRESS-NEXT: Failed Tests (1):`.
  **L987 CN**: 注释说明了附近脚本逻辑：`QUIET-W-PROGRESS-NEXT: Failed Tests (1):`。
- **L988 EN**: Comment documents nearby script behavior: `QUIET-W-PROGRESS-NEXT: verbosity :: fail.txt`.
  **L988 CN**: 注释说明了附近脚本逻辑：`QUIET-W-PROGRESS-NEXT: verbosity :: fail.txt`。
- **L989 EN**: Comment documents nearby script behavior: `QUIET-W-PROGRESS-EMPTY:`.
  **L989 CN**: 注释说明了附近脚本逻辑：`QUIET-W-PROGRESS-EMPTY:`。
- **L990 EN**: Comment documents nearby script behavior: `QUIET-W-PROGRESS-NEXT: ********************`.
  **L990 CN**: 注释说明了附近脚本逻辑：`QUIET-W-PROGRESS-NEXT: ********************`。
- **L991 EN**: Comment documents nearby script behavior: `QUIET-W-PROGRESS-NEXT: Unexpectedly Passed Tests (1):`.
  **L991 CN**: 注释说明了附近脚本逻辑：`QUIET-W-PROGRESS-NEXT: Unexpectedly Passed Tests (1):`。
- **L992 EN**: Comment documents nearby script behavior: `QUIET-W-PROGRESS-NEXT: verbosity :: xpass.txt`.
  **L992 CN**: 注释说明了附近脚本逻辑：`QUIET-W-PROGRESS-NEXT: verbosity :: xpass.txt`。
- **L993 EN**: Comment documents nearby script behavior: `QUIET-W-PROGRESS-EMPTY:`.
  **L993 CN**: 注释说明了附近脚本逻辑：`QUIET-W-PROGRESS-EMPTY:`。
- **L994 EN**: Comment documents nearby script behavior: `QUIET-W-PROGRESS-EMPTY:`.
  **L994 CN**: 注释说明了附近脚本逻辑：`QUIET-W-PROGRESS-EMPTY:`。
- **L995 EN**: Comment documents nearby script behavior: `QUIET-W-PROGRESS-NEXT: Total Discovered Tests: 5`.
  **L995 CN**: 注释说明了附近脚本逻辑：`QUIET-W-PROGRESS-NEXT: Total Discovered Tests: 5`。
- **L996 EN**: Comment documents nearby script behavior: `QUIET-W-PROGRESS-NEXT: Failed : 1 (20.00%)`.
  **L996 CN**: 注释说明了附近脚本逻辑：`QUIET-W-PROGRESS-NEXT: Failed : 1 (20.00%)`。

### Lines 997-1024

````python
# QUIET-W-PROGRESS-NEXT:   Unexpectedly Passed: 1 (20.00%)

# RUN: not %{lit} --show-all --terse-summary %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt
# RUN: FileCheck %s --check-prefix ALL-TERSE < %t/stdout.txt
# RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt

# ALL-TERSE: -- Testing: 5 tests, 1 workers --
# ALL-TERSE-NEXT: FAIL: verbosity :: fail.txt (1 of 5)
# ALL-TERSE-NEXT: ******************** TEST 'verbosity :: fail.txt' FAILED ********************
# ALL-TERSE-NEXT: Exit Code: 127
# ALL-TERSE-EMPTY:
# ALL-TERSE-NEXT: Command Output (stdout):
# ALL-TERSE-NEXT: --
# ALL-TERSE-NEXT: # {{R}}UN: at line 1
# ALL-TERSE-NEXT: echo "fail test output"
# ALL-TERSE-NEXT: # executed command: echo 'fail test output'
# ALL-TERSE-NEXT: # .---command stdout------------
# ALL-TERSE-NEXT: # | fail test output
# ALL-TERSE-NEXT: # `-----------------------------
# ALL-TERSE-NEXT: # {{R}}UN: at line 2
# ALL-TERSE-NEXT: fail
# ALL-TERSE-NEXT: # executed command: fail
# ALL-TERSE-NEXT: # .---command stderr------------
# ALL-TERSE-NEXT: # | 'fail': command not found
# ALL-TERSE-NEXT: # `-----------------------------
# ALL-TERSE-NEXT: # error: command failed with exit status: 127
# ALL-TERSE-EMPTY:
# ALL-TERSE-NEXT: --
````
- **L997 EN**: Comment documents nearby script behavior: `QUIET-W-PROGRESS-NEXT: Unexpectedly Passed: 1 (20.00%)`.
  **L997 CN**: 注释说明了附近脚本逻辑：`QUIET-W-PROGRESS-NEXT: Unexpectedly Passed: 1 (20.00%)`。
- **L998 EN**: Blank line separates nearby declarations or logic blocks.
  **L998 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L999 EN**: Comment documents nearby script behavior: `RUN: not %{lit} --show-all --terse-summary %{inputs}/verbosity 2> %t/stderr.txt > %t/st...`.
  **L999 CN**: 注释说明了附近脚本逻辑：`RUN: not %{lit} --show-all --terse-summary %{inputs}/verbosity 2> %t/stderr.txt > %t/st...`。
- **L1000 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix ALL-TERSE < %t/stdout.txt`.
  **L1000 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix ALL-TERSE < %t/stdout.txt`。
- **L1001 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt`.
  **L1001 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt`。
- **L1002 EN**: Blank line separates nearby declarations or logic blocks.
  **L1002 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1003 EN**: Comment documents nearby script behavior: `ALL-TERSE: -- Testing: 5 tests, 1 workers`.
  **L1003 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE: -- Testing: 5 tests, 1 workers`。
- **L1004 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: FAIL: verbosity :: fail.txt (1 of 5)`.
  **L1004 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: FAIL: verbosity :: fail.txt (1 of 5)`。
- **L1005 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: ******************** TEST 'verbosity :: fail.txt' FAILED **************...`.
  **L1005 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: ******************** TEST 'verbosity :: fail.txt' FAILED **************...`。
- **L1006 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: Exit Code: 127`.
  **L1006 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: Exit Code: 127`。
- **L1007 EN**: Comment documents nearby script behavior: `ALL-TERSE-EMPTY:`.
  **L1007 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-EMPTY:`。
- **L1008 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: Command Output (stdout):`.
  **L1008 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: Command Output (stdout):`。
- **L1009 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT:`.
  **L1009 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT:`。
- **L1010 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: # {{R}}UN: at line 1`.
  **L1010 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: # {{R}}UN: at line 1`。
- **L1011 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: echo "fail test output"`.
  **L1011 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: echo "fail test output"`。
- **L1012 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: # executed command: echo 'fail test output'`.
  **L1012 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: # executed command: echo 'fail test output'`。
- **L1013 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: # .---command stdout`.
  **L1013 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: # .---command stdout`。
- **L1014 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: # | fail test output`.
  **L1014 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: # | fail test output`。
- **L1015 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: # \``.
  **L1015 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: # \``。
- **L1016 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: # {{R}}UN: at line 2`.
  **L1016 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: # {{R}}UN: at line 2`。
- **L1017 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: fail`.
  **L1017 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: fail`。
- **L1018 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: # executed command: fail`.
  **L1018 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: # executed command: fail`。
- **L1019 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: # .---command stderr`.
  **L1019 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: # .---command stderr`。
- **L1020 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: # | 'fail': command not found`.
  **L1020 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: # | 'fail': command not found`。
- **L1021 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: # \``.
  **L1021 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: # \``。
- **L1022 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: # error: command failed with exit status: 127`.
  **L1022 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: # error: command failed with exit status: 127`。
- **L1023 EN**: Comment documents nearby script behavior: `ALL-TERSE-EMPTY:`.
  **L1023 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-EMPTY:`。
- **L1024 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT:`.
  **L1024 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT:`。

### Lines 1025-1052

````python
# ALL-TERSE-EMPTY:
# ALL-TERSE-NEXT: ********************
# ALL-TERSE-NEXT: PASS: verbosity :: pass.txt (2 of 5)
# ALL-TERSE-NEXT: Exit Code: 0
# ALL-TERSE-EMPTY:
# ALL-TERSE-NEXT: Command Output (stdout):
# ALL-TERSE-NEXT: --
# ALL-TERSE-NEXT: # {{R}}UN: at line 1
# ALL-TERSE-NEXT: echo "pass test output"
# ALL-TERSE-NEXT: # executed command: echo 'pass test output'
# ALL-TERSE-NEXT: # .---command stdout------------
# ALL-TERSE-NEXT: # | pass test output
# ALL-TERSE-NEXT: # `-----------------------------
# ALL-TERSE-EMPTY:
# ALL-TERSE-NEXT: --
# ALL-TERSE-EMPTY:
# ALL-TERSE-NEXT: ********************
# ALL-TERSE-NEXT: {{UN}}SUPPORTED: verbosity :: unsupported.txt (3 of 5)
# ALL-TERSE-NEXT: Test requires the following unavailable features: asdf
# ALL-TERSE-NEXT: ********************
# ALL-TERSE-NEXT: {{X}}FAIL: verbosity :: xfail.txt (4 of 5)
# ALL-TERSE-NEXT: Exit Code: 1
# ALL-TERSE-EMPTY:
# ALL-TERSE-NEXT: Command Output (stdout):
# ALL-TERSE-NEXT: --
# ALL-TERSE-NEXT: # {{R}}UN: at line 2
# ALL-TERSE-NEXT: not echo "xfail test output"
# ALL-TERSE-NEXT: # executed command: not echo 'xfail test output'
````
- **L1025 EN**: Comment documents nearby script behavior: `ALL-TERSE-EMPTY:`.
  **L1025 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-EMPTY:`。
- **L1026 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: ********************`.
  **L1026 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: ********************`。
- **L1027 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: PASS: verbosity :: pass.txt (2 of 5)`.
  **L1027 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: PASS: verbosity :: pass.txt (2 of 5)`。
- **L1028 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: Exit Code: 0`.
  **L1028 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: Exit Code: 0`。
- **L1029 EN**: Comment documents nearby script behavior: `ALL-TERSE-EMPTY:`.
  **L1029 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-EMPTY:`。
- **L1030 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: Command Output (stdout):`.
  **L1030 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: Command Output (stdout):`。
- **L1031 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT:`.
  **L1031 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT:`。
- **L1032 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: # {{R}}UN: at line 1`.
  **L1032 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: # {{R}}UN: at line 1`。
- **L1033 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: echo "pass test output"`.
  **L1033 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: echo "pass test output"`。
- **L1034 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: # executed command: echo 'pass test output'`.
  **L1034 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: # executed command: echo 'pass test output'`。
- **L1035 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: # .---command stdout`.
  **L1035 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: # .---command stdout`。
- **L1036 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: # | pass test output`.
  **L1036 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: # | pass test output`。
- **L1037 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: # \``.
  **L1037 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: # \``。
- **L1038 EN**: Comment documents nearby script behavior: `ALL-TERSE-EMPTY:`.
  **L1038 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-EMPTY:`。
- **L1039 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT:`.
  **L1039 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT:`。
- **L1040 EN**: Comment documents nearby script behavior: `ALL-TERSE-EMPTY:`.
  **L1040 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-EMPTY:`。
- **L1041 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: ********************`.
  **L1041 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: ********************`。
- **L1042 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: {{UN}}SUPPORTED: verbosity :: unsupported.txt (3 of 5)`.
  **L1042 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: {{UN}}SUPPORTED: verbosity :: unsupported.txt (3 of 5)`。
- **L1043 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: Test requires the following unavailable features: asdf`.
  **L1043 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: Test requires the following unavailable features: asdf`。
- **L1044 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: ********************`.
  **L1044 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: ********************`。
- **L1045 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: {{X}}FAIL: verbosity :: xfail.txt (4 of 5)`.
  **L1045 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: {{X}}FAIL: verbosity :: xfail.txt (4 of 5)`。
- **L1046 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: Exit Code: 1`.
  **L1046 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: Exit Code: 1`。
- **L1047 EN**: Comment documents nearby script behavior: `ALL-TERSE-EMPTY:`.
  **L1047 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-EMPTY:`。
- **L1048 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: Command Output (stdout):`.
  **L1048 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: Command Output (stdout):`。
- **L1049 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT:`.
  **L1049 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT:`。
- **L1050 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: # {{R}}UN: at line 2`.
  **L1050 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: # {{R}}UN: at line 2`。
- **L1051 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: not echo "xfail test output"`.
  **L1051 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: not echo "xfail test output"`。
- **L1052 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: # executed command: not echo 'xfail test output'`.
  **L1052 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: # executed command: not echo 'xfail test output'`。

### Lines 1053-1080

````python
# ALL-TERSE-NEXT: # .---command stdout------------
# ALL-TERSE-NEXT: # | xfail test output
# ALL-TERSE-NEXT: # `-----------------------------
# ALL-TERSE-NEXT: # error: command failed with exit status: 1
# ALL-TERSE-EMPTY:
# ALL-TERSE-NEXT: --
# ALL-TERSE-EMPTY:
# ALL-TERSE-NEXT: ********************
# ALL-TERSE-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)
# ALL-TERSE-NEXT: ******************** TEST 'verbosity :: xpass.txt' FAILED ********************
# ALL-TERSE-NEXT: Exit Code: 0
# ALL-TERSE-EMPTY:
# ALL-TERSE-NEXT: Command Output (stdout):
# ALL-TERSE-NEXT: --
# ALL-TERSE-NEXT: # {{R}}UN: at line 2
# ALL-TERSE-NEXT: echo "xpass test output"
# ALL-TERSE-NEXT: # executed command: echo 'xpass test output'
# ALL-TERSE-NEXT: # .---command stdout------------
# ALL-TERSE-NEXT: # | xpass test output
# ALL-TERSE-NEXT: # `-----------------------------
# ALL-TERSE-EMPTY:
# ALL-TERSE-NEXT: --
# ALL-TERSE-EMPTY:
# ALL-TERSE-NEXT: ********************
# ALL-TERSE-NEXT: ********************
# ALL-TERSE-NEXT: Failed Tests (1):
# ALL-TERSE-NEXT:   verbosity :: fail.txt
# ALL-TERSE-EMPTY:
````
- **L1053 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: # .---command stdout`.
  **L1053 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: # .---command stdout`。
- **L1054 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: # | xfail test output`.
  **L1054 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: # | xfail test output`。
- **L1055 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: # \``.
  **L1055 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: # \``。
- **L1056 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: # error: command failed with exit status: 1`.
  **L1056 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: # error: command failed with exit status: 1`。
- **L1057 EN**: Comment documents nearby script behavior: `ALL-TERSE-EMPTY:`.
  **L1057 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-EMPTY:`。
- **L1058 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT:`.
  **L1058 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT:`。
- **L1059 EN**: Comment documents nearby script behavior: `ALL-TERSE-EMPTY:`.
  **L1059 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-EMPTY:`。
- **L1060 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: ********************`.
  **L1060 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: ********************`。
- **L1061 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)`.
  **L1061 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)`。
- **L1062 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: ******************** TEST 'verbosity :: xpass.txt' FAILED *************...`.
  **L1062 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: ******************** TEST 'verbosity :: xpass.txt' FAILED *************...`。
- **L1063 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: Exit Code: 0`.
  **L1063 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: Exit Code: 0`。
- **L1064 EN**: Comment documents nearby script behavior: `ALL-TERSE-EMPTY:`.
  **L1064 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-EMPTY:`。
- **L1065 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: Command Output (stdout):`.
  **L1065 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: Command Output (stdout):`。
- **L1066 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT:`.
  **L1066 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT:`。
- **L1067 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: # {{R}}UN: at line 2`.
  **L1067 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: # {{R}}UN: at line 2`。
- **L1068 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: echo "xpass test output"`.
  **L1068 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: echo "xpass test output"`。
- **L1069 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: # executed command: echo 'xpass test output'`.
  **L1069 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: # executed command: echo 'xpass test output'`。
- **L1070 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: # .---command stdout`.
  **L1070 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: # .---command stdout`。
- **L1071 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: # | xpass test output`.
  **L1071 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: # | xpass test output`。
- **L1072 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: # \``.
  **L1072 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: # \``。
- **L1073 EN**: Comment documents nearby script behavior: `ALL-TERSE-EMPTY:`.
  **L1073 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-EMPTY:`。
- **L1074 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT:`.
  **L1074 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT:`。
- **L1075 EN**: Comment documents nearby script behavior: `ALL-TERSE-EMPTY:`.
  **L1075 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-EMPTY:`。
- **L1076 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: ********************`.
  **L1076 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: ********************`。
- **L1077 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: ********************`.
  **L1077 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: ********************`。
- **L1078 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: Failed Tests (1):`.
  **L1078 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: Failed Tests (1):`。
- **L1079 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: verbosity :: fail.txt`.
  **L1079 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: verbosity :: fail.txt`。
- **L1080 EN**: Comment documents nearby script behavior: `ALL-TERSE-EMPTY:`.
  **L1080 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-EMPTY:`。

### Lines 1081-1097

````python
# ALL-TERSE-NEXT: ********************
# ALL-TERSE-NEXT: Unexpectedly Passed Tests (1):
# ALL-TERSE-NEXT:   verbosity :: xpass.txt
# ALL-TERSE-EMPTY:
# ALL-TERSE-EMPTY:
# ALL-TERSE-NEXT: Total Discovered Tests: 5
# ALL-TERSE-NEXT:   Failed             : 1 (20.00%)
# ALL-TERSE-NEXT:   Unexpectedly Passed: 1 (20.00%)

# RUN: not %{lit} --show-all --diagnostic-level error %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt
# RUN: FileCheck %s --check-prefix SHOW-ALL < %t/stdout.txt
# RUN: FileCheck %s --check-prefix QUIET-ERR --implicit-check-not lit < %t/stderr.txt

# RUN: not %{lit} --show-all --test-output off %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt
# RUN: FileCheck %s --check-prefix NO-ARGS < %t/stdout.txt
# RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt

````
- **L1081 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: ********************`.
  **L1081 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: ********************`。
- **L1082 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: Unexpectedly Passed Tests (1):`.
  **L1082 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: Unexpectedly Passed Tests (1):`。
- **L1083 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: verbosity :: xpass.txt`.
  **L1083 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: verbosity :: xpass.txt`。
- **L1084 EN**: Comment documents nearby script behavior: `ALL-TERSE-EMPTY:`.
  **L1084 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-EMPTY:`。
- **L1085 EN**: Comment documents nearby script behavior: `ALL-TERSE-EMPTY:`.
  **L1085 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-EMPTY:`。
- **L1086 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: Total Discovered Tests: 5`.
  **L1086 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: Total Discovered Tests: 5`。
- **L1087 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: Failed : 1 (20.00%)`.
  **L1087 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: Failed : 1 (20.00%)`。
- **L1088 EN**: Comment documents nearby script behavior: `ALL-TERSE-NEXT: Unexpectedly Passed: 1 (20.00%)`.
  **L1088 CN**: 注释说明了附近脚本逻辑：`ALL-TERSE-NEXT: Unexpectedly Passed: 1 (20.00%)`。
- **L1089 EN**: Blank line separates nearby declarations or logic blocks.
  **L1089 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1090 EN**: Comment documents nearby script behavior: `RUN: not %{lit} --show-all --diagnostic-level error %{inputs}/verbosity 2> %t/stderr.tx...`.
  **L1090 CN**: 注释说明了附近脚本逻辑：`RUN: not %{lit} --show-all --diagnostic-level error %{inputs}/verbosity 2> %t/stderr.tx...`。
- **L1091 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix SHOW-ALL < %t/stdout.txt`.
  **L1091 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix SHOW-ALL < %t/stdout.txt`。
- **L1092 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix QUIET-ERR --implicit-check-not lit < %t/stderr.txt`.
  **L1092 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix QUIET-ERR --implicit-check-not lit < %t/stderr.txt`。
- **L1093 EN**: Blank line separates nearby declarations or logic blocks.
  **L1093 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1094 EN**: Comment documents nearby script behavior: `RUN: not %{lit} --show-all --test-output off %{inputs}/verbosity 2> %t/stderr.txt > %t/...`.
  **L1094 CN**: 注释说明了附近脚本逻辑：`RUN: not %{lit} --show-all --test-output off %{inputs}/verbosity 2> %t/stderr.txt > %t/...`。
- **L1095 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix NO-ARGS < %t/stdout.txt`.
  **L1095 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix NO-ARGS < %t/stdout.txt`。
- **L1096 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt`.
  **L1096 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt`。
- **L1097 EN**: Blank line separates nearby declarations or logic blocks.
  **L1097 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1098-1125

````python
# RUN: not %{lit} --succinct --print-result-after all %{inputs}/verbosity 2> %t/stderr.txt > %t/stdout.txt
# RUN: FileCheck %s --check-prefix SUCCINCT-RESULT-ALL < %t/stdout.txt
# RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt

# SUCCINCT-RESULT-ALL:      -- Testing: 5 tests, 1 workers --
# SUCCINCT-RESULT-ALL-NEXT: Testing:
# SUCCINCT-RESULT-ALL-NEXT: FAIL: verbosity :: fail.txt (1 of 5)
# SUCCINCT-RESULT-ALL-NEXT: Testing:
# SUCCINCT-RESULT-ALL-NEXT: PASS: verbosity :: pass.txt (2 of 5)
# SUCCINCT-RESULT-ALL-NEXT: Testing:
# SUCCINCT-RESULT-ALL-NEXT: {{UN}}SUPPORTED: verbosity :: unsupported.txt (3 of 5)
# SUCCINCT-RESULT-ALL-NEXT: Testing:
# SUCCINCT-RESULT-ALL-NEXT: {{X}}FAIL: verbosity :: xfail.txt (4 of 5)
# SUCCINCT-RESULT-ALL-NEXT: Testing:
# SUCCINCT-RESULT-ALL-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)
# SUCCINCT-RESULT-ALL-NEXT: Testing:
# SUCCINCT-RESULT-ALL-NEXT: ********************
# SUCCINCT-RESULT-ALL-NEXT: Failed Tests (1):
# SUCCINCT-RESULT-ALL-NEXT:   verbosity :: fail.txt
# SUCCINCT-RESULT-ALL-EMPTY:
# SUCCINCT-RESULT-ALL-NEXT: ********************
# SUCCINCT-RESULT-ALL-NEXT: Unexpectedly Passed Tests (1):
# SUCCINCT-RESULT-ALL-NEXT:   verbosity :: xpass.txt
# SUCCINCT-RESULT-ALL-EMPTY:
# SUCCINCT-RESULT-ALL-EMPTY:
# SUCCINCT-RESULT-ALL-NEXT: Testing Time: {{.*}}s
# SUCCINCT-RESULT-ALL-EMPTY:
# SUCCINCT-RESULT-ALL-NEXT: Total Discovered Tests: 5
````
- **L1098 EN**: Comment documents nearby script behavior: `RUN: not %{lit} --succinct --print-result-after all %{inputs}/verbosity 2> %t/stderr.tx...`.
  **L1098 CN**: 注释说明了附近脚本逻辑：`RUN: not %{lit} --succinct --print-result-after all %{inputs}/verbosity 2> %t/stderr.tx...`。
- **L1099 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix SUCCINCT-RESULT-ALL < %t/stdout.txt`.
  **L1099 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix SUCCINCT-RESULT-ALL < %t/stdout.txt`。
- **L1100 EN**: Comment documents nearby script behavior: `RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt`.
  **L1100 CN**: 注释说明了附近脚本逻辑：`RUN: FileCheck %s --check-prefix NO-ARGS-ERR --implicit-check-not lit < %t/stderr.txt`。
- **L1101 EN**: Blank line separates nearby declarations or logic blocks.
  **L1101 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1102 EN**: Comment documents nearby script behavior: `SUCCINCT-RESULT-ALL: -- Testing: 5 tests, 1 workers`.
  **L1102 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-RESULT-ALL: -- Testing: 5 tests, 1 workers`。
- **L1103 EN**: Comment documents nearby script behavior: `SUCCINCT-RESULT-ALL-NEXT: Testing:`.
  **L1103 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-RESULT-ALL-NEXT: Testing:`。
- **L1104 EN**: Comment documents nearby script behavior: `SUCCINCT-RESULT-ALL-NEXT: FAIL: verbosity :: fail.txt (1 of 5)`.
  **L1104 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-RESULT-ALL-NEXT: FAIL: verbosity :: fail.txt (1 of 5)`。
- **L1105 EN**: Comment documents nearby script behavior: `SUCCINCT-RESULT-ALL-NEXT: Testing:`.
  **L1105 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-RESULT-ALL-NEXT: Testing:`。
- **L1106 EN**: Comment documents nearby script behavior: `SUCCINCT-RESULT-ALL-NEXT: PASS: verbosity :: pass.txt (2 of 5)`.
  **L1106 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-RESULT-ALL-NEXT: PASS: verbosity :: pass.txt (2 of 5)`。
- **L1107 EN**: Comment documents nearby script behavior: `SUCCINCT-RESULT-ALL-NEXT: Testing:`.
  **L1107 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-RESULT-ALL-NEXT: Testing:`。
- **L1108 EN**: Comment documents nearby script behavior: `SUCCINCT-RESULT-ALL-NEXT: {{UN}}SUPPORTED: verbosity :: unsupported.txt (3 of 5)`.
  **L1108 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-RESULT-ALL-NEXT: {{UN}}SUPPORTED: verbosity :: unsupported.txt (3 of 5)`。
- **L1109 EN**: Comment documents nearby script behavior: `SUCCINCT-RESULT-ALL-NEXT: Testing:`.
  **L1109 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-RESULT-ALL-NEXT: Testing:`。
- **L1110 EN**: Comment documents nearby script behavior: `SUCCINCT-RESULT-ALL-NEXT: {{X}}FAIL: verbosity :: xfail.txt (4 of 5)`.
  **L1110 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-RESULT-ALL-NEXT: {{X}}FAIL: verbosity :: xfail.txt (4 of 5)`。
- **L1111 EN**: Comment documents nearby script behavior: `SUCCINCT-RESULT-ALL-NEXT: Testing:`.
  **L1111 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-RESULT-ALL-NEXT: Testing:`。
- **L1112 EN**: Comment documents nearby script behavior: `SUCCINCT-RESULT-ALL-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)`.
  **L1112 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-RESULT-ALL-NEXT: XPASS: verbosity :: xpass.txt (5 of 5)`。
- **L1113 EN**: Comment documents nearby script behavior: `SUCCINCT-RESULT-ALL-NEXT: Testing:`.
  **L1113 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-RESULT-ALL-NEXT: Testing:`。
- **L1114 EN**: Comment documents nearby script behavior: `SUCCINCT-RESULT-ALL-NEXT: ********************`.
  **L1114 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-RESULT-ALL-NEXT: ********************`。
- **L1115 EN**: Comment documents nearby script behavior: `SUCCINCT-RESULT-ALL-NEXT: Failed Tests (1):`.
  **L1115 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-RESULT-ALL-NEXT: Failed Tests (1):`。
- **L1116 EN**: Comment documents nearby script behavior: `SUCCINCT-RESULT-ALL-NEXT: verbosity :: fail.txt`.
  **L1116 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-RESULT-ALL-NEXT: verbosity :: fail.txt`。
- **L1117 EN**: Comment documents nearby script behavior: `SUCCINCT-RESULT-ALL-EMPTY:`.
  **L1117 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-RESULT-ALL-EMPTY:`。
- **L1118 EN**: Comment documents nearby script behavior: `SUCCINCT-RESULT-ALL-NEXT: ********************`.
  **L1118 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-RESULT-ALL-NEXT: ********************`。
- **L1119 EN**: Comment documents nearby script behavior: `SUCCINCT-RESULT-ALL-NEXT: Unexpectedly Passed Tests (1):`.
  **L1119 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-RESULT-ALL-NEXT: Unexpectedly Passed Tests (1):`。
- **L1120 EN**: Comment documents nearby script behavior: `SUCCINCT-RESULT-ALL-NEXT: verbosity :: xpass.txt`.
  **L1120 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-RESULT-ALL-NEXT: verbosity :: xpass.txt`。
- **L1121 EN**: Comment documents nearby script behavior: `SUCCINCT-RESULT-ALL-EMPTY:`.
  **L1121 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-RESULT-ALL-EMPTY:`。
- **L1122 EN**: Comment documents nearby script behavior: `SUCCINCT-RESULT-ALL-EMPTY:`.
  **L1122 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-RESULT-ALL-EMPTY:`。
- **L1123 EN**: Comment documents nearby script behavior: `SUCCINCT-RESULT-ALL-NEXT: Testing Time: {{.*}}s`.
  **L1123 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-RESULT-ALL-NEXT: Testing Time: {{.*}}s`。
- **L1124 EN**: Comment documents nearby script behavior: `SUCCINCT-RESULT-ALL-EMPTY:`.
  **L1124 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-RESULT-ALL-EMPTY:`。
- **L1125 EN**: Comment documents nearby script behavior: `SUCCINCT-RESULT-ALL-NEXT: Total Discovered Tests: 5`.
  **L1125 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-RESULT-ALL-NEXT: Total Discovered Tests: 5`。

### Lines 1126-1130

````python
# SUCCINCT-RESULT-ALL-NEXT:   Unsupported        : 1 (20.00%)
# SUCCINCT-RESULT-ALL-NEXT:   Passed             : 1 (20.00%)
# SUCCINCT-RESULT-ALL-NEXT:   Expectedly Failed  : 1 (20.00%)
# SUCCINCT-RESULT-ALL-NEXT:   Failed             : 1 (20.00%)
# SUCCINCT-RESULT-ALL-NEXT:   Unexpectedly Passed: 1 (20.00%)
````
- **L1126 EN**: Comment documents nearby script behavior: `SUCCINCT-RESULT-ALL-NEXT: Unsupported : 1 (20.00%)`.
  **L1126 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-RESULT-ALL-NEXT: Unsupported : 1 (20.00%)`。
- **L1127 EN**: Comment documents nearby script behavior: `SUCCINCT-RESULT-ALL-NEXT: Passed : 1 (20.00%)`.
  **L1127 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-RESULT-ALL-NEXT: Passed : 1 (20.00%)`。
- **L1128 EN**: Comment documents nearby script behavior: `SUCCINCT-RESULT-ALL-NEXT: Expectedly Failed : 1 (20.00%)`.
  **L1128 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-RESULT-ALL-NEXT: Expectedly Failed : 1 (20.00%)`。
- **L1129 EN**: Comment documents nearby script behavior: `SUCCINCT-RESULT-ALL-NEXT: Failed : 1 (20.00%)`.
  **L1129 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-RESULT-ALL-NEXT: Failed : 1 (20.00%)`。
- **L1130 EN**: Comment documents nearby script behavior: `SUCCINCT-RESULT-ALL-NEXT: Unexpectedly Passed: 1 (20.00%)`.
  **L1130 CN**: 注释说明了附近脚本逻辑：`SUCCINCT-RESULT-ALL-NEXT: Unexpectedly Passed: 1 (20.00%)`。

## Key Concepts / 关键概念

- EN: lit test runner infrastructure
  - CN: lit 测试运行基础设施
- EN: pattern-based output verification
  - CN: 基于模式的输出校验
- EN: test harness behavior
  - CN: 测试框架行为
- EN: worker execution model
  - CN: 工作线程/进程执行模型
- EN: Python utility scripting
  - CN: Python 工具脚本

## Dependencies / 依赖关系

- EN: No direct include/import dependencies are declared in this file.
  - CN: 该文件中没有声明直接的 include/import 依赖。
