# loop-peeling.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/Triton/loop-peeling.mlir`
- **EN:** Test for the MLIR/Triton pipeline `-triton-test-loop-peeling, -canonicalize` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `-triton-test-loop-peeling, -canonicalize` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -split-input-file -allow-unregistered-dialect -triton-test-loop-peeling -canonicalize | FileCheck %s`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -split-input-file -allow-unregistered-dialect -triton-test-loop-peeling -canonicalize | FileCheck %s`

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
1| // RUN: triton-opt %s -split-input-file -allow-unregistered-dialect -triton-test-loop-peeling -canonicalize | FileCheck %s
2| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -split-input-file -allow-unregistered-dialect -triton-test-loop-peeling -canonicalize | FileCheck %s` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -split-input-file -allow-unregistered-dialect -triton-test-loop-peeling -canonicalize | FileCheck %s`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 3-3
```mlir
3| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 4-21
```mlir
 4| // CHECK-LABEL: @simple_loop_i32
 5| // CHECK: (%[[LB:.*]]: i32, %[[UB:.*]]: i32, %[[STEP:.*]]: i32) -> f32
 6| // CHECK-DAG: %[[CST:.*]] = arith.constant 0.000000e+00 : f32
 7| // CHECK-DAG: %[[ONE:.*]] = arith.constant 1 : i32
 8| // CHECK: %[[NUB:.*]] = arith.subi %[[UB]], %[[STEP]]
 9| // CHECK: %[[FOR:.*]] = scf.for %[[IV:.*]] = %[[LB]] to %[[NUB]] step %[[STEP]]
10| // CHECK: scf.yield
11| // CHECK: %[[RANGE:.*]] = arith.subi %[[UB]], %[[LB]]
12| // CHECK: %[[RANGE_M1:.*]] = arith.subi %[[RANGE]], %[[ONE]]
13| // CHECK: %[[ITERS_M1:.*]] = arith.divsi %[[RANGE_M1]], %[[STEP]]
14| // CHECK: %[[DELTA:.*]] = arith.muli %[[ITERS_M1]], %[[STEP]]
15| // CHECK: %[[LAST_IV:.*]] = arith.addi %[[DELTA]], %[[LB]]
16| // CHECK: %[[COND:.*]] = arith.cmpi slt, %[[LB]], %[[UB]]
17| // CHECK: %[[IF:.*]] = scf.if %[[COND]]
18| // CHECK:   %[[DEF:.*]] = "def"(%[[LAST_IV]]) : (i32) -> f32
19| // CHECK:   %[[RES:.*]] = arith.addf %[[FOR]], %[[DEF]] : f32
20| // CHECK:   scf.yield %[[RES]] : f32
21| // CHECK: else
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @simple_loop_i32 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @simple_loop_i32 这样的标签用于锚定匹配范围。

### Lines 22-23
```mlir
22| // CHECK:   scf.yield %[[FOR]] : f32
23| // CHECK: tt.return %[[IF]] : f32
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 24-35
```mlir
24| tt.func @simple_loop_i32(%lb : i32, %ub : i32, %step : i32) -> f32 {
25|   %init = arith.constant 0.00e+00 : f32
26|   %loop = scf.for %iv = %lb to %ub step %step iter_args(%acc = %init) -> (f32) : i32 {
27|     %a = "def"(%iv) : (i32) -> f32
28|     %res = arith.addf %acc, %a : f32
29|     scf.yield %res : f32
30|   } {__test_peel_epilogue}
31| 
32|   tt.return %loop#0 : f32
33| }
34| }
35| 
```
**EN:** This function-oriented block defines or enters `simple_loop_i32`. Within it, the test exercises tt.func, constants, structured loops, floating-point additions, loop/if yielded values, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `simple_loop_i32` 为核心。测试在其中演示 tt.func、常量、结构化循环、浮点加法、循环/分支产出值，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 36-36
```mlir
36| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 37-38
```mlir
37| 
38| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 39-56
```mlir
39| // CHECK-LABEL: @simple_loop_i32
40| // CHECK: (%[[LB:.*]]: i32, %[[UB:.*]]: i32, %[[STEP:.*]]: i32) -> f32
41| // CHECK-DAG: %[[CST:.*]] = arith.constant 0.000000e+00 : f32
42| // CHECK-DAG: %[[ONE:.*]] = arith.constant 1 : i32
43| // CHECK: %[[NUB:.*]] = arith.subi %[[UB]], %[[STEP]]
44| // CHECK: %[[FOR:.*]] = scf.for %[[IV:.*]] = %[[LB]] to %[[NUB]] step %[[STEP]]
45| // CHECK: scf.yield
46| // CHECK: %[[RANGE:.*]] = arith.subi %[[UB]], %[[LB]]
47| // CHECK: %[[RANGE_M1:.*]] = arith.subi %[[RANGE]], %[[ONE]]
48| // CHECK: %[[ITERS_M1:.*]] = arith.divsi %[[RANGE_M1]], %[[STEP]]
49| // CHECK: %[[DELTA:.*]] = arith.muli %[[ITERS_M1]], %[[STEP]]
50| // CHECK: %[[LAST_IV:.*]] = arith.addi %[[DELTA]], %[[LB]]
51| // CHECK: %[[COND:.*]] = arith.cmpi slt, %[[LB]], %[[UB]]
52| // CHECK: %[[IF:.*]] = scf.if %[[COND]]
53| // CHECK:   %[[DEF:.*]] = "def"(%[[LAST_IV]]) : (i32) -> f32
54| // CHECK:   %[[RES:.*]] = arith.addf %[[FOR]], %[[DEF]] : f32
55| // CHECK:   scf.yield %[[RES]] : f32
56| // CHECK: else
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @simple_loop_i32 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @simple_loop_i32 这样的标签用于锚定匹配范围。

### Lines 57-58
```mlir
57| // CHECK:   scf.yield %[[FOR]] : f32
58| // CHECK: tt.return %[[IF]] : f32
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 59-69
```mlir
59| tt.func @simple_loop_i32(%lb : i32, %ub : i32, %step : i32) -> f32 {
60|   %init = arith.constant 0.00e+00 : f32
61|   %loop = scf.for %iv = %lb to %ub step %step iter_args(%acc = %init) -> (f32) : i32 {
62|     %a = "def"(%iv) : (i32) -> f32
63|     %res = arith.addf %acc, %a : f32
64|     scf.yield %res : f32
65|   } {__test_peel_epilogue}
66| 
67|   tt.return %loop#0 : f32
68| }
69| }
```
**EN:** This function-oriented block defines or enters `simple_loop_i32`. Within it, the test exercises tt.func, constants, structured loops, floating-point additions, loop/if yielded values, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `simple_loop_i32` 为核心。测试在其中演示 tt.func、常量、结构化循环、浮点加法、循环/分支产出值，这些操作构成了被优化、lower 或验证的主要数据流。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `-allow-unregistered-dialect`, `-triton-test-loop-peeling`, `-canonicalize`
- **CN:** 主要 pass 选项：`-split-input-file`，`-allow-unregistered-dialect`，`-triton-test-loop-peeling`，`-canonicalize`
- **EN:** Dominant operations include `tt.func`, `module`, `arith.constant`, `scf.for`, `arith.addf`, `scf.yield`, `tt.return`.
- **CN:** 主要操作包括 `tt.func`、`module`、`arith.constant`、`scf.for`、`arith.addf`、`scf.yield`、`tt.return`。
- **EN:** The file contains 1 independently testable section(s). Check styles used: CHECK x34, CHECK-DAG x4, CHECK-LABEL x2. Important labels include @simple_loop_i32, @simple_loop_i32. It validates that the selected pass pipeline rewrites each labeled test case into the checked form.
- **CN:** 该文件包含 1 个可独立测试的分段。使用的检查类型：CHECK ×34，CHECK-DAG ×4，CHECK-LABEL ×2。 关键标签包括 @simple_loop_i32，@simple_loop_i32。 它验证所选 pass 流水线会把每个带标签的测试用例重写成检查所要求的形式。

## Dependencies / 依赖关系

- **EN:** This test depends on Triton/MLIR textual IR parsing, the pass pipeline named in the RUN lines, and the dialect/type definitions referenced in the file.
- **CN:** 该测试依赖 Triton/MLIR 文本 IR 解析、RUN 行中指定的 pass 流水线，以及文件中引用的方言/类型定义。
- **EN:** `tt`: Triton high-level tensor/pointer operations.
- **CN:** `tt`：Triton 高层张量/指针操作。
- **EN:** `arith`: MLIR arithmetic dialect for scalar/tensor math.
- **CN:** `arith`：用于标量/张量运算的 MLIR arithmetic 方言。
- **EN:** `scf`: Structured control flow for loops and conditionals.
- **CN:** `scf`：用于循环与条件分支的结构化控制流。
- **EN:** `FileCheck` is used to match required textual patterns after rewriting/lowering.
- **CN:** `FileCheck` 用于在重写/lower 之后匹配必须出现的文本模式。