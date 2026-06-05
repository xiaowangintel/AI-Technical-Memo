# loop-pipeline-indirect-load.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/loop-pipeline-indirect-load.mlir`
- **EN:** Test for the MLIR/Triton pipeline `-tritongpu-assign-latencies=num-stages=2, -tritongpu-schedule-loops, -tritongpu-pipeline=num-stages=2` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `-tritongpu-assign-latencies=num-stages=2, -tritongpu-schedule-loops, -tritongpu-pipeline=num-stages=2` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -tritongpu-assign-latencies=num-stages=2 -tritongpu-schedule-loops -tritongpu-pipeline=num-stages=2 | FileCheck %s`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -tritongpu-assign-latencies=num-stages=2 -tritongpu-schedule-loops -tritongpu-pipeline=num-stages=2 | FileCheck %s`

## Line-by-Line Analysis / 逐行分析

### Lines 1-1
```mlir
1| // RUN: triton-opt %s -tritongpu-assign-latencies=num-stages=2 -tritongpu-schedule-loops -tritongpu-pipeline=num-stages=2 | FileCheck %s
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -tritongpu-assign-latencies=num-stages=2 -tritongpu-schedule-loops -tritongpu-pipeline=num-stages=2 | FileCheck %s` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -tritongpu-assign-latencies=num-stages=2 -tritongpu-schedule-loops -tritongpu-pipeline=num-stages=2 | FileCheck %s`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 2-8
```mlir
2| // CHECK-LABEL: @indirect_load_two_stages
3| // CHECK: scf.for
4| // CHECK: tt.dot
5| // CHECK: tt.load
6| // CHECK: async_copy_global_to_local
7| // CHECK: async_copy_global_to_local
8| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @indirect_load_two_stages anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @indirect_load_two_stages 这样的标签用于锚定匹配范围。

### Lines 9-12
```mlir
 9| #blocked = #ttg.blocked<{sizePerThread = [4, 4], threadsPerWarp = [1, 32], warpsPerCTA = [2, 1], order = [1, 0]}>
10| #blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [16, 2], warpsPerCTA = [1, 2], order = [0, 1]}>
11| #blocked3 = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [1, 32], warpsPerCTA = [2, 1], order = [1, 0]}>
12| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 13-13
```mlir
13| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 2 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 14-31
```mlir
14|   tt.func public @indirect_load_two_stages(%arg2: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg3: !tt.ptr<i64> {tt.divisibility = 16 : i32}, %arg4: !tt.ptr<i64> {tt.divisibility = 16 : i32}, %arg5: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg7: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg12: i32 {tt.divisibility = 16 : i32}, %arg17: i32 {tt.divisibility = 16 : i32}, %arg18: i32, %arg19: i32) {
15|     %c32_i32 = arith.constant 32 : i32
16|     %c16_i32 = arith.constant 16 : i32
17|     %cst = arith.constant dense<0.000000e+00> : tensor<16x128xf32, #blocked>
18| 
19|     %0 = tt.get_program_id y : i32
20|     %1 = tt.addptr %arg3, %0 : !tt.ptr<i64>, i32
21|     %2 = tt.load %1 : !tt.ptr<i64>
22| 
23|     %7 = tt.get_program_id x : i32
24|     %8 = arith.muli %7, %c16_i32 : i32
25|     %10 = tt.make_range {end = 16 : i32, start = 0 : i32} : tensor<16xi32, #ttg.slice<{dim = 1, parent = #blocked3}>>
26|     %15 = tt.splat %8 : i32 -> tensor<16xi32, #ttg.slice<{dim = 1, parent = #blocked3}>>
27|     %18 = arith.addi %15, %10 : tensor<16xi32, #ttg.slice<{dim = 1, parent = #blocked3}>>
28| 
29|     %20 = tt.make_range {end = 32 : i32, start = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>
30|     %22 = tt.make_range {end = 32 : i32, start = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 1, parent = #blocked3}>>
31|     %34 = arith.extsi %arg12 : i32 to i64
```
**EN:** This function-oriented block defines or enters `indirect_load_two_stages`. Within it, the test exercises constants, lane/block index ranges, tt.func, program IDs, pointer arithmetic, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `indirect_load_two_stages` 为核心。测试在其中演示 常量、lane/block 索引范围、tt.func、程序 ID、指针算术，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 32-49
```mlir
32|     %35 = arith.muli %2, %34 : i64
33|     %36 = tt.addptr %arg2, %35 : !tt.ptr<f32>, i64
34| 
35|     %47 = tt.splat %arg4 : !tt.ptr<i64> -> tensor<32x!tt.ptr<i64>, #ttg.slice<{dim = 0, parent = #blocked1}>>
36|     %48 = tt.addptr %47, %20 : tensor<32x!tt.ptr<i64>, #ttg.slice<{dim = 0, parent = #blocked1}>>, tensor<32xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>
37| 
38|     %59 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked3}>>
39|     %61 = arith.extsi %59 : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked3}>> to tensor<128xi64, #ttg.slice<{dim = 0, parent = #blocked3}>>
40|     %63 = tt.expand_dims %61 {axis = 0 : i32} : tensor<128xi64, #ttg.slice<{dim = 0, parent = #blocked3}>> -> tensor<1x128xi64, #blocked3>
41| 
42|     %85 = arith.extsi %22 : tensor<32xi32, #ttg.slice<{dim = 1, parent = #blocked3}>> to tensor<32xi64, #ttg.slice<{dim = 1, parent = #blocked3}>>
43|     %107 = tt.splat %36 : !tt.ptr<f32> -> tensor<32x128x!tt.ptr<f32>, #blocked3>
44|     %108 = tt.splat %34 : i64 -> tensor<32x1xi64, #blocked3>
45|     %109 = tt.broadcast %63 : tensor<1x128xi64, #blocked3> -> tensor<32x128xi64, #blocked3>
46| 
47|     %101 = tt.splat %arg5 : !tt.ptr<f32> -> tensor<16x32x!tt.ptr<f32>, #blocked1>
48|     %111:1 = scf.for %arg28 = %arg18 to %arg19 step %c32_i32 iter_args(%arg29 = %cst) -> (tensor<16x128xf32, #blocked>)  : i32 {
49|       %129 = tt.splat %arg28 : i32 -> tensor<32xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining broadcasted scalars or pointers, pointer arithmetic, arith.extsi, integer multiplications, lane/block index ranges.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 广播后的标量或指针、指针算术、arith.extsi、整数乘法、lane/block 索引范围。

### Lines 50-67
```mlir
50|       %160 = tt.addptr %48, %129 : tensor<32x!tt.ptr<i64>, #ttg.slice<{dim = 0, parent = #blocked1}>>, tensor<32xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>
51|       %161 = tt.load %160 : tensor<32x!tt.ptr<i64>, #ttg.slice<{dim = 0, parent = #blocked1}>>
52|       %162 = tt.expand_dims %161 {axis = 0 : i32} : tensor<32xi64, #ttg.slice<{dim = 0, parent = #blocked1}>> -> tensor<1x32xi64, #blocked1>
53|       %163 = tt.broadcast %162 : tensor<1x32xi64, #blocked1> -> tensor<16x32xi64, #blocked1>
54|       %182 = tt.addptr %101, %163 : tensor<16x32x!tt.ptr<f32>, #blocked1>, tensor<16x32xi64, #blocked1>
55|       %183 = tt.load %182 : tensor<16x32x!tt.ptr<f32>, #blocked1>
56| 
57|       %197 = arith.extsi %arg28 : i32 to i64
58|       %198 = tt.splat %197 : i64 -> tensor<32xi64, #ttg.slice<{dim = 1, parent = #blocked3}>>
59|       %199 = arith.addi %198, %85 : tensor<32xi64, #ttg.slice<{dim = 1, parent = #blocked3}>>
60|       %200 = tt.expand_dims %199 {axis = 1 : i32} : tensor<32xi64, #ttg.slice<{dim = 1, parent = #blocked3}>> -> tensor<32x1xi64, #blocked3>
61|       %201 = arith.muli %200, %108 : tensor<32x1xi64, #blocked3>
62|       %202 = tt.broadcast %201 : tensor<32x1xi64, #blocked3> -> tensor<32x128xi64, #blocked3>
63|       %203 = arith.addi %202, %109 : tensor<32x128xi64, #blocked3>
64|       %204 = tt.addptr %107, %203 : tensor<32x128x!tt.ptr<f32>, #blocked3>, tensor<32x128xi64, #blocked3>
65|       %209 = tt.load %204 : tensor<32x128x!tt.ptr<f32>, #blocked3>
66| 
67|       %210 = ttg.convert_layout %183 : tensor<16x32xf32, #blocked1> -> tensor<16x32xf32, #ttg.dot_op<{opIdx = 0, parent = #blocked}>>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on pointer arithmetic, masked or vectorized loads, shape expansion, tensor broadcasting, integer additions. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 指针算术、带掩码或向量化的加载、形状扩展、张量广播、整数加法。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 68-85
```mlir
68|       %211 = ttg.convert_layout %209 : tensor<32x128xf32, #blocked3> -> tensor<32x128xf32, #ttg.dot_op<{opIdx = 1, parent = #blocked}>>
69|       %212 = tt.dot %210, %211, %arg29 : tensor<16x32xf32, #ttg.dot_op<{opIdx = 0, parent = #blocked}>> * tensor<32x128xf32, #ttg.dot_op<{opIdx = 1, parent = #blocked}>> -> tensor<16x128xf32, #blocked>
70|       scf.yield %212 : tensor<16x128xf32, #blocked>
71|     }
72|     %112 = tt.expand_dims %18 {axis = 1 : i32} : tensor<16xi32, #ttg.slice<{dim = 1, parent = #blocked3}>> -> tensor<16x1xi32, #blocked3>
73|     %113 = tt.splat %2 : i64 -> tensor<16x1xi64, #blocked3>
74|     %114 = arith.extsi %112 : tensor<16x1xi32, #blocked3> to tensor<16x1xi64, #blocked3>
75|     %115 = arith.addi %113, %114 : tensor<16x1xi64, #blocked3>
76|     %116 = arith.extsi %arg17 : i32 to i64
77|     %117 = tt.splat %116 : i64 -> tensor<16x1xi64, #blocked3>
78|     %118 = arith.muli %115, %117 : tensor<16x1xi64, #blocked3>
79|     %119 = tt.expand_dims %59 {axis = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked3}>> -> tensor<1x128xi32, #blocked3>
80|     %120 = tt.broadcast %118 : tensor<16x1xi64, #blocked3> -> tensor<16x128xi64, #blocked3>
81|     %121 = arith.extsi %119 : tensor<1x128xi32, #blocked3> to tensor<1x128xi64, #blocked3>
82|     %122 = tt.broadcast %121 : tensor<1x128xi64, #blocked3> -> tensor<16x128xi64, #blocked3>
83|     %123 = arith.addi %120, %122 : tensor<16x128xi64, #blocked3>
84|     %124 = tt.splat %arg7 : !tt.ptr<f32> -> tensor<16x128x!tt.ptr<f32>, #blocked3>
85|     %125 = tt.addptr %124, %123 : tensor<16x128x!tt.ptr<f32>, #blocked3>, tensor<16x128xi64, #blocked3>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining broadcasted scalars or pointers, arith.extsi, shape expansion, integer additions, tensor broadcasting.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 广播后的标量或指针、arith.extsi、形状扩展、整数加法、张量广播。

### Lines 86-90
```mlir
86|     %128 = ttg.convert_layout %111#0 : tensor<16x128xf32, #blocked> -> tensor<16x128xf32, #blocked3>
87|     tt.store %125, %128 : tensor<16x128x!tt.ptr<f32>, #blocked3>
88|     tt.return
89|   }
90| }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, masked or vectorized stores, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、带掩码或向量化的存储、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-tritongpu-assign-latencies=num-stages=2`, `-tritongpu-schedule-loops`, `-tritongpu-pipeline=num-stages=2`
- **CN:** 主要 pass 选项：`-tritongpu-assign-latencies=num-stages=2`，`-tritongpu-schedule-loops`，`-tritongpu-pipeline=num-stages=2`
- **EN:** Dominant operations include `tt.splat`, `tt.addptr`, `arith.extsi`, `arith.addi`, `tt.expand_dims`, `tt.broadcast`, `tt.load`, `arith.muli`, `tt.make_range`, `arith.constant`.
- **CN:** 主要操作包括 `tt.splat`、`tt.addptr`、`arith.extsi`、`arith.addi`、`tt.expand_dims`、`tt.broadcast`、`tt.load`、`arith.muli`、`tt.make_range`、`arith.constant`。
- **EN:** The file contains 1 independently testable section(s). Check styles used: CHECK x5, CHECK-LABEL x1. Important labels include @indirect_load_two_stages. It validates that the selected pass pipeline rewrites each labeled test case into the checked form.
- **CN:** 该文件包含 1 个可独立测试的分段。使用的检查类型：CHECK ×5，CHECK-LABEL ×1。 关键标签包括 @indirect_load_two_stages。 它验证所选 pass 流水线会把每个带标签的测试用例重写成检查所要求的形式。

## Dependencies / 依赖关系

- **EN:** This test depends on Triton/MLIR textual IR parsing, the pass pipeline named in the RUN lines, and the dialect/type definitions referenced in the file.
- **CN:** 该测试依赖 Triton/MLIR 文本 IR 解析、RUN 行中指定的 pass 流水线，以及文件中引用的方言/类型定义。
- **EN:** `tt`: Triton high-level tensor/pointer operations.
- **CN:** `tt`：Triton 高层张量/指针操作。
- **EN:** `ttg`: TritonGPU layout, memory, and GPU-specific operations.
- **CN:** `ttg`：TritonGPU 布局、内存和 GPU 专用操作。
- **EN:** `arith`: MLIR arithmetic dialect for scalar/tensor math.
- **CN:** `arith`：用于标量/张量运算的 MLIR arithmetic 方言。
- **EN:** `scf`: Structured control flow for loops and conditionals.
- **CN:** `scf`：用于循环与条件分支的结构化控制流。
- **EN:** `FileCheck` is used to match required textual patterns after rewriting/lowering.
- **CN:** `FileCheck` 用于在重写/lower 之后匹配必须出现的文本模式。