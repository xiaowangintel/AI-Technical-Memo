# loop-invariant-code-motion.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/Triton/loop-invariant-code-motion.mlir`
- **EN:** Test for the MLIR/Triton pipeline `-triton-licm` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `-triton-licm` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt --split-input-file %s -triton-licm | FileCheck %s`
- **CN:** RUN 流水线：`// RUN: triton-opt --split-input-file %s -triton-licm | FileCheck %s`

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
1| // RUN: triton-opt --split-input-file %s -triton-licm | FileCheck %s
2| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt --split-input-file %s -triton-licm | FileCheck %s` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt --split-input-file %s -triton-licm | FileCheck %s`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 3-6
```mlir
3| tt.func @hoist_load_without_mask(%arg0: tensor<1024x!tt.ptr<f32>>, %arg1: tensor<1024xi32>, %arg2: tensor<1024xi32>, %arg3: i32, %arg4 : i32, %arg5: tensor<1024x!tt.ptr<f32>>) {
4|   %cst = arith.constant dense<0.000000e+00> : tensor<1024xf32>
5|   %c1_i32 = arith.constant 1 : i32
6|   // Check if the load is hoisted
```
**EN:** This function-oriented block defines or enters `hoist_load_without_mask`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `hoist_load_without_mask` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 7-23
```mlir
 7|   // CHECK-LABEL: hoist_load_without_mask
 8|   // CHECK: %[[TRIP_COUNT_CMP:.*]] = arith.cmpi slt, %[[LB:.*]], %[[UB:.*]]
 9|   // CHECK: %[[SPLAT:.*]] = tt.splat %[[TRIP_COUNT_CMP]]
10|   // CHECK: %[[LOAD:.*]] = tt.load %[[_:.*]], %[[SPLAT]]
11|   // CHECK: arith.addf %[[LOAD]], %[[LOAD]]
12|   // CHECK: scf.for
13|   // CHECK-NOT: tt.load
14|   %1 = scf.for %arg7 = %arg3 to %arg4 step %c1_i32 iter_args(%arg6 = %cst) -> (tensor<1024xf32>)  : i32 {
15|     %2 = tt.load %arg0 : tensor<1024x!tt.ptr<f32>>
16|     %3 = arith.addf %2, %2 : tensor<1024xf32>
17|     %4 = arith.addf %arg6, %3 : tensor<1024xf32>
18|     scf.yield %4 : tensor<1024xf32>
19|   }
20|   tt.store %arg5, %1 : tensor<1024x!tt.ptr<f32>>
21|   tt.return
22| }
23| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining floating-point additions, structured loops, masked or vectorized loads, loop/if yielded values, masked or vectorized stores.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 浮点加法、结构化循环、带掩码或向量化的加载、循环/分支产出值、带掩码或向量化的存储。

### Lines 24-24
```mlir
24| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 25-28
```mlir
25| 
26| tt.func @hoist_two_loads_without_mask(%arg0: tensor<1024x!tt.ptr<f32>>, %arg1: tensor<1024xi32>, %arg2: tensor<1024xi32>, %arg3: i32, %arg4 : i32, %arg5: tensor<1024x!tt.ptr<f32>>, %arg6: tensor<1024x!tt.ptr<f32>>) {
27|   %cst = arith.constant dense<0.000000e+00> : tensor<1024xf32>
28|   %c1_i32 = arith.constant 1 : i32
```
**EN:** This function-oriented block defines or enters `hoist_two_loads_without_mask`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `hoist_two_loads_without_mask` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 29-46
```mlir
29|   // CHECK-LABEL: hoist_two_loads_without_mask
30|   // CHECK: %[[TRIP_COUNT_CMP_1:.*]] = arith.cmpi slt, %[[LB:.*]], %[[UB:.*]]
31|   // CHECK: %[[SPLAT_1:.*]] = tt.splat %[[TRIP_COUNT_CMP_1]]
32|   // CHECK: %[[LOAD_1:.*]] = tt.load %[[_:.*]], %[[SPLAT_1]]
33|   // CHECK: %[[TRIP_COUNT_CMP_2:.*]] = arith.cmpi slt, %[[LB]], %[[UB]]
34|   // CHECK: %[[SPLAT_2:.*]] = tt.splat %[[TRIP_COUNT_CMP_2]]
35|   // CHECK: %[[LOAD_2:.*]] = tt.load %[[_:.*]], %[[SPLAT_2]]
36|   // CHECK: arith.addf %[[LOAD_1]], %[[LOAD_2]]
37|   // CHECK: scf.for
38|   // CHECK-NOT: tt.load
39|   %1 = scf.for %arg8 = %arg3 to %arg4 step %c1_i32 iter_args(%arg7 = %cst) -> (tensor<1024xf32>)  : i32 {
40|     %2 = tt.load %arg0 : tensor<1024x!tt.ptr<f32>>
41|     %3 = tt.load %arg6 : tensor<1024x!tt.ptr<f32>>
42|     %4 = arith.addf %2, %3 : tensor<1024xf32>
43|     %5 = arith.addf %arg7, %4 : tensor<1024xf32>
44|     scf.yield %5 : tensor<1024xf32>
45|   }
46|   tt.store %arg5, %1 : tensor<1024x!tt.ptr<f32>>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining masked or vectorized loads, floating-point additions, structured loops, loop/if yielded values, masked or vectorized stores.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 带掩码或向量化的加载、浮点加法、结构化循环、循环/分支产出值、带掩码或向量化的存储。

### Lines 47-49
```mlir
47|   tt.return
48| }
49| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 50-50
```mlir
50| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 51-55
```mlir
51| 
52| tt.func @hoist_load_with_mask(%arg0: tensor<1024x!tt.ptr<f32>>, %arg1: tensor<1024xi32>, %arg2: tensor<1024xi32>, %arg3: i32, %arg4 : i32, %arg5: tensor<1024x!tt.ptr<f32>>) {
53|   %cst = arith.constant dense<0.000000e+00> : tensor<1024xf32>
54|   %c1_i32 = arith.constant 1 : i32
55|   // Check if the load is hoisted
```
**EN:** This function-oriented block defines or enters `hoist_load_with_mask`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `hoist_load_with_mask` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 56-73
```mlir
56|   // CHECK-LABEL: hoist_load_with_mask
57|   // CHECK: %[[MASK:.*]] = arith.cmpi
58|   // CHECK: %[[TRIP_COUNT_CMP:.*]] = arith.cmpi slt, %[[LB:.*]], %[[UB:.*]]
59|   // CHECK: %[[SPLAT:.*]] = tt.splat %[[TRIP_COUNT_CMP]]
60|   // CHECK: %[[AND:.*]] = arith.andi %[[SPLAT]], %[[MASK]]
61|   // CHECK: %[[LOAD:.*]] = tt.load %[[_:.*]], %[[AND]]
62|   // CHECK: arith.addf %[[LOAD]], %[[LOAD]]
63|   // CHECK: scf.for
64|   // CHECK-NOT: tt.load
65|   %0 = arith.cmpi slt, %arg1, %arg2 : tensor<1024xi32>
66|   %1 = scf.for %arg7 = %arg3 to %arg4 step %c1_i32 iter_args(%arg6 = %cst) -> (tensor<1024xf32>)  : i32 {
67|     %2 = tt.load %arg0, %0 : tensor<1024x!tt.ptr<f32>>
68|     %3 = arith.addf %2, %2 : tensor<1024xf32>
69|     %4 = arith.addf %arg6, %3 : tensor<1024xf32>
70|     scf.yield %4 : tensor<1024xf32>
71|   }
72|   tt.store %arg5, %1, %0 : tensor<1024x!tt.ptr<f32>>
73|   tt.return
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining floating-point additions, integer comparisons, structured loops, masked or vectorized loads, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 浮点加法、整数比较、结构化循环、带掩码或向量化的加载、循环/分支产出值。

### Lines 74-75
```mlir
74| }
75| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 76-76
```mlir
76| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 77-80
```mlir
77| 
78| tt.func @cannot_hoist_with_print_in_loop(%arg0: tensor<1024x!tt.ptr<f32>>, %arg1: tensor<1024xi32>, %arg2: tensor<1024xi32>, %arg3: i32, %arg4 : i32, %arg5: tensor<1024x!tt.ptr<f32>>) {
79|   %cst = arith.constant dense<0.000000e+00> : tensor<1024xf32>
80|   %c1_i32 = arith.constant 1 : i32
```
**EN:** This function-oriented block defines or enters `cannot_hoist_with_print_in_loop`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `cannot_hoist_with_print_in_loop` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 81-97
```mlir
81|   // CHECK-NOT: tt.load
82|   // CHECK: scf.for
83|   // CHECK: tt.load
84|   // CHECK: arith.addf
85|   // CHECK: arith.addf
86|   %0 = arith.cmpi slt, %arg1, %arg2 : tensor<1024xi32>
87|   %1 = scf.for %arg7 = %arg3 to %arg4 step %c1_i32 iter_args(%arg6 = %cst) -> (tensor<1024xf32>)  : i32 {
88|     %2 = tt.load %arg0, %0 : tensor<1024x!tt.ptr<f32>>
89|     %3 = arith.addf %2, %2 : tensor<1024xf32>
90|     %4 = arith.addf %arg6, %3 : tensor<1024xf32>
91|     tt.print " x: " {hex = false, isSigned = array<i32: 0>} : %4 : tensor<1024xf32>
92|     scf.yield %4 : tensor<1024xf32>
93|   }
94|   tt.store %arg5, %1, %0 : tensor<1024x!tt.ptr<f32>>
95|   tt.return
96| }
97| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining floating-point additions, integer comparisons, structured loops, masked or vectorized loads, tt.print.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 浮点加法、整数比较、结构化循环、带掩码或向量化的加载、tt.print。

### Lines 98-98
```mlir
98| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 99-102
```mlir
 99| 
100| tt.func @cannot_hoist_with_assert_in_loop(%arg0: tensor<1024x!tt.ptr<f32>>, %arg1: tensor<1024xi32>, %arg2: tensor<1024xi32>, %arg3: i32, %arg4 : i32, %arg5: tensor<1024x!tt.ptr<f32>>) {
101|   %cst = arith.constant dense<0.000000e+00> : tensor<1024xf32>
102|   %c1_i32 = arith.constant 1 : i32
```
**EN:** This function-oriented block defines or enters `cannot_hoist_with_assert_in_loop`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `cannot_hoist_with_assert_in_loop` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 103-120
```mlir
103|   // CHECK-NOT: tt.load
104|   // CHECK: scf.for
105|   // CHECK: tt.load
106|   // CHECK: arith.addf
107|   // CHECK: arith.addf
108|   %0 = arith.cmpi slt, %arg1, %arg2 : tensor<1024xi32>
109|   %cmp = arith.cmpi sge, %arg4, %arg3 : i32
110|   %1 = scf.for %arg7 = %arg3 to %arg4 step %c1_i32 iter_args(%arg6 = %cst) -> (tensor<1024xf32>)  : i32 {
111|     tt.assert %cmp, "cond must be true " : i1
112|     %2 = tt.load %arg0, %0 : tensor<1024x!tt.ptr<f32>>
113|     %3 = arith.addf %2, %2 : tensor<1024xf32>
114|     %4 = arith.addf %arg6, %3 : tensor<1024xf32>
115|     scf.yield %4 : tensor<1024xf32>
116|   }
117|   tt.store %arg5, %1, %0 : tensor<1024x!tt.ptr<f32>>
118|   tt.return
119| }
120| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining integer comparisons, floating-point additions, structured loops, tt.assert, masked or vectorized loads.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 整数比较、浮点加法、结构化循环、tt.assert、带掩码或向量化的加载。

### Lines 121-121
```mlir
121| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 122-125
```mlir
122| 
123| tt.func @cannot_hoist_with_store_in_loop(%arg0: tensor<1024x!tt.ptr<f32>>, %arg1: tensor<1024xi32>, %arg2: tensor<1024xi32>, %arg3: i32, %arg4 : i32, %arg5: tensor<1024x!tt.ptr<f32>>, %tmp: tensor<1024x!tt.ptr<f32>>) {
124|   %cst = arith.constant dense<0.000000e+00> : tensor<1024xf32>
125|   %c1_i32 = arith.constant 1 : i32
```
**EN:** This function-oriented block defines or enters `cannot_hoist_with_store_in_loop`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `cannot_hoist_with_store_in_loop` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 126-142
```mlir
126|   // CHECK-NOT: tt.load
127|   // CHECK: scf.for
128|   // CHECK: tt.load
129|   // CHECK: arith.addf
130|   // CHECK: arith.addf
131|   %0 = arith.cmpi slt, %arg1, %arg2 : tensor<1024xi32>
132|   %1 = scf.for %arg7 = %arg3 to %arg4 step %c1_i32 iter_args(%arg6 = %cst) -> (tensor<1024xf32>)  : i32 {
133|     %2 = tt.load %arg0, %0 : tensor<1024x!tt.ptr<f32>>
134|     %3 = arith.addf %2, %2 : tensor<1024xf32>
135|     %4 = arith.addf %arg6, %3 : tensor<1024xf32>
136|     tt.store %tmp, %4, %0 : tensor<1024x!tt.ptr<f32>>
137|     scf.yield %4 : tensor<1024xf32>
138|   }
139|   tt.store %arg5, %1, %0 : tensor<1024x!tt.ptr<f32>>
140|   tt.return
141| }
142| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining floating-point additions, masked or vectorized stores, integer comparisons, structured loops, masked or vectorized loads.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 浮点加法、带掩码或向量化的存储、整数比较、结构化循环、带掩码或向量化的加载。

### Lines 143-143
```mlir
143| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 144-146
```mlir
144| 
145| tt.func @hoist_cond_no_hoist_load_from_scf_while(%ptr: tensor<1024x!tt.ptr<f32>>, %arg1: i32, %arg2 : i32) {
146|   %cst = arith.constant dense<0.000000e+00> : tensor<1024xf32>
```
**EN:** This function-oriented block defines or enters `hoist_cond_no_hoist_load_from_scf_while`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `hoist_cond_no_hoist_load_from_scf_while` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 147-164
```mlir
147|   // CHECK-LABEL: hoist_cond_no_hoist_load_from_scf_while
148|   // CHECK: %[[CST42:.*]] = arith.constant 42
149|   // CHECK: %[[ADD:.*]] = arith.addi %[[_:.*]], %[[CST42]]
150|   // CHECK: %[[COND:.*]] = arith.cmpi slt, %[[ADD]], %[[_:.*]]
151|   // CHECK: scf.while
152|   // CHECK: do
153|   // CHECK: tt.load
154|   // CHECK: arith.addf
155|   // CHECK: scf.yield
156|   %1 = scf.while (%arg0 = %cst) : (tensor<1024xf32>) -> (tensor<1024xf32>) {
157|     %cst_42 = arith.constant 42 : i32
158|     %add_42 = arith.addi %arg1, %cst_42 : i32
159|     %2 = arith.cmpi slt, %add_42, %arg2 : i32
160|     scf.condition(%2) %arg0 : tensor<1024xf32>
161|   } do {
162|   ^bb0(%arg0: tensor<1024xf32>):
163|     %3 = tt.load %ptr : tensor<1024x!tt.ptr<f32>>
164|     %4 = arith.addf %3, %3 : tensor<1024xf32>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining scf.while, constants, integer additions, integer comparisons, scf.condition.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 scf.while、常量、整数加法、整数比较、scf.condition。

### Lines 165-169
```mlir
165|     scf.yield %4 : tensor<1024xf32>
166|   }
167|   tt.store %ptr, %1 : tensor<1024x!tt.ptr<f32>>
168|   tt.return
169| }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, masked or vectorized stores, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、带掩码或向量化的存储、tt.return。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `--split-input-file`, `-triton-licm`
- **CN:** 主要 pass 选项：`--split-input-file`，`-triton-licm`
- **EN:** Dominant operations include `tt.func`, `arith.constant`, `arith.addf`, `tt.load`, `tt.store`, `scf.yield`, `tt.return`, `scf.for`, `arith.cmpi`, `tt.print`.
- **CN:** 主要操作包括 `tt.func`、`arith.constant`、`arith.addf`、`tt.load`、`tt.store`、`scf.yield`、`tt.return`、`scf.for`、`arith.cmpi`、`tt.print`。
- **EN:** The file contains 6 independently testable section(s). Check styles used: CHECK x40, CHECK-NOT x6, CHECK-LABEL x4. Important labels include hoist_load_without_mask, hoist_two_loads_without_mask, hoist_load_with_mask, hoist_cond_no_hoist_load_from_scf_while. It validates both the presence of the desired rewrite and the absence of unwanted residual operations.
- **CN:** 该文件包含 6 个可独立测试的分段。使用的检查类型：CHECK ×40，CHECK-NOT ×6，CHECK-LABEL ×4。 关键标签包括 hoist_load_without_mask，hoist_two_loads_without_mask，hoist_load_with_mask，hoist_cond_no_hoist_load_from_scf_while。 它同时验证目标重写已经出现，并且不再残留不需要的操作。

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