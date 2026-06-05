# fuse-nested-loops.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/fuse-nested-loops.mlir`
- **EN:** Test for the MLIR/Triton pipeline `--tritongpu-fuse-nested-loops, -canonicalize, -cse` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `--tritongpu-fuse-nested-loops, -canonicalize, -cse` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s --allow-unregistered-dialect --tritongpu-fuse-nested-loops -canonicalize -cse | FileCheck %s`
- **CN:** RUN 流水线：`// RUN: triton-opt %s --allow-unregistered-dialect --tritongpu-fuse-nested-loops -canonicalize -cse | FileCheck %s`

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
1| // RUN: triton-opt %s --allow-unregistered-dialect --tritongpu-fuse-nested-loops -canonicalize -cse | FileCheck %s
2| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s --allow-unregistered-dialect --tritongpu-fuse-nested-loops -canonicalize -cse | FileCheck %s` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s --allow-unregistered-dialect --tritongpu-fuse-nested-loops -canonicalize -cse | FileCheck %s`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 3-3
```mlir
3| // CHECK-LABEL: @empty_function
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @empty_function anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @empty_function 这样的标签用于锚定匹配范围。

### Lines 4-7
```mlir
4| tt.func @empty_function() {
5|   tt.return
6| }
7| 
```
**EN:** This function-oriented block defines or enters `empty_function`. Within it, the test exercises tt.func, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `empty_function` 为核心。测试在其中演示 tt.func、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 8-8
```mlir
8| // CHECK-LABEL: @no_fusion
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @no_fusion anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @no_fusion 这样的标签用于锚定匹配范围。

### Lines 9-10
```mlir
 9| tt.func @no_fusion(%lb: index, %ub: index, %step: index) -> index {
10|   %c0 = arith.constant 0 : index
```
**EN:** This function-oriented block defines or enters `no_fusion`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `no_fusion` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 11-12
```mlir
11|   // CHECK: before.loop
12|   "before.loop"() : () -> ()
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 13-14
```mlir
13|   // CHECK-NEXT: scf.for
14|   %0 = scf.for %i = %lb to %ub step %step iter_args(%k = %c0) -> index {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环。

### Lines 15-16
```mlir
15|     // CHECK-NEXT: body
16|     %1 = "body"(%i, %k) : (index, index) -> index
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 17-18
```mlir
17|     // CHECK-NEXT: yield
18|     scf.yield %1 : index
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值。

### Lines 19-20
```mlir
19|   // CHECK-NEXT: }
20|   } {"ttg.always-fuse"}
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 21-25
```mlir
21|   // CHECK-NEXT: after.loop
22|   "after.loop"() : () -> ()
23|   tt.return %0 : index
24| }
25| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 26-27
```mlir
26| // CHECK-LABEL: @fuse_one_level_simple
27| // CHECK-SAME: [[LBI:%.*]]: i64, [[UBI:%.*]]: i64, [[STEPI:%.*]]: i64, [[LBJ:%.*]]: i64, [[UBJ:%.*]]: i64, [[STEPJ:%.*]]: i64
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @fuse_one_level_simple anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @fuse_one_level_simple 这样的标签用于锚定匹配范围。

### Lines 28-30
```mlir
28| tt.func @fuse_one_level_simple(%lbi: i64, %ubi: i64, %stepi: i64, %lbj: i64, %ubj: i64, %stepj: i64) {
29|   // len_i = len(range(lbi, ubi, stepi))
30|   //
```
**EN:** This function-oriented block defines or enters `fuse_one_level_simple`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `fuse_one_level_simple` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 31-35
```mlir
31|   // CHECK:      [[DIFF_I:%.*]] = arith.subi [[UBI]], [[LBI]]
32|   // CHECK-NEXT: [[LEN_I:%.*]] = arith.ceildivsi [[DIFF_I]], [[STEPI]]
33| 
34|   // len_j = len(range(lbj0, ubj0, stepj0))
35|   //
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 36-40
```mlir
36|   // CHECK-NEXT: [[DIFF_J:%.*]] = arith.subi [[UBJ]], [[LBJ]]
37|   // CHECK-NEXT: [[LEN_J:%.*]] = arith.ceildivsi [[DIFF_J]], [[STEPJ]]
38| 
39|   // inner_len = max(1, len_j0)
40|   //
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 41-44
```mlir
41|   // CHECK:      [[INNER_LEN:%.*]] = arith.maxsi [[LEN_J]], %c1_i64
42| 
43|   // total_iters = len_i * max(1, inner_len)
44|   //
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 45-51
```mlir
45|   // CHECK: [[TOTAL_ITERS:%.*]] = arith.muli [[LEN_I]], [[INNER_LEN]]
46| 
47|   // T = -1
48|   // i = lbi - stepi
49|   // j = None
50|   // for _ in range(total_iters):
51|   //
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 52-60
```mlir
52|   // CHECK: [[I_INIT:%.*]] = arith.subi [[LBI]], [[STEPI]]
53|   // CHECK: scf.for %{{.*}} = %c0_i64 to [[TOTAL_ITERS]] step %c1_i64 iter_args(
54|   // CHECK-SAME: [[T:%.*]] = %c0_i64, [[I_ARG:%.*]] = [[I_INIT]], [[J_ARG:%.*]] = %c0_i64) -> (i64, i64, i64) : i64 {
55|   scf.for %i = %lbi to %ubi step %stepi : i64 {
56|     // if T == 0:
57|     //   i += stepi
58|     //   prologue(i)
59|     //   j = lbj
60|     //
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环。

### Lines 61-75
```mlir
61|     // CHECK-NEXT: [[PROLOGUE_COND:%.*]] = arith.cmpi eq, [[T]], %c0_i64
62|     // CHECK-NEXT: [[J:%.*]] = arith.select [[PROLOGUE_COND]], [[LBJ]], [[J_ARG]]
63|     // CHECK-NEXT: [[I:%.*]] = scf.if [[PROLOGUE_COND]] -> (i64) {
64|     // CHECK-NEXT:   [[I_INCR:%.*]] = arith.addi [[I_ARG]], [[STEPI]]
65|     // CHECK-NEXT:   "prologue"([[I_INCR]]) : (i64) -> ()
66|     // CHECK-NEXT:   yield [[I_INCR]]
67|     // CHECK-NEXT: } else {
68|     // CHECK-NEXT:   yield [[I_ARG]]
69|     // CHECK-NEXT: }
70|     "prologue"(%i) : (i64) -> ()
71| 
72|     // if T >= 0 and T < len_j:
73|     //   body(i, j)
74|     //   j += stepj
75|     //
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 76-93
```mlir
76|     // CHECK:      [[GE:%.*]] = arith.cmpi sge, [[T]], %c0_i64
77|     // CHECK-NEXT: [[LT:%.*]] = arith.cmpi slt, [[T]], [[LEN_J]]
78|     // CHECK-NEXT: [[COND:%.*]] = arith.andi [[GE]], [[LT]]
79|     // CHECK-NEXT: [[J_NEXT:%.*]] = scf.if [[COND]] -> (i64) {
80|     // CHECK-NEXT:   "body"([[I]], [[J]]) : (i64, i64) -> ()
81|     // CHECK-NEXT:   [[J_INCR:%.*]] = arith.addi [[J]], [[STEPJ]]
82|     // CHECK-NEXT:   yield [[J_INCR]]
83|     // CHECK-NEXT: } else {
84|     // CHECK-NEXT:   yield [[J]]
85|     // CHECK-NEXT: }
86|     scf.for %j = %lbj to %ubj step %stepj : i64 {
87|       "body"(%i, %j) : (i64, i64) -> ()
88|     }
89| 
90|     // if T == max(1, len_j) - 1:
91|     //   epilogue(i)
92|     //   i += stepi
93|     //
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环。

### Lines 94-102
```mlir
 94|     // CHECK:      [[T_END:%.*]] = arith.subi [[INNER_LEN]], %c1_i64
 95|     // CHECK-NEXT: [[EPILOGUE_COND:%.*]] = arith.cmpi eq, [[T]], [[T_END]]
 96|     // CHECK-NEXT: scf.if [[EPILOGUE_COND]] {
 97|     // CHECK-NEXT:   "epilogue"([[I]]) : (i64) -> ()
 98|     // CHECK-NEXT: }
 99|     "epilogue"(%i) : (i64) -> ()
100| 
101|     // T = 0 if T == (inner_len - 1) else T + 1
102|     //
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 103-110
```mlir
103|     // CHECK:      [[T_PLUS_1:%.*]] = arith.addi [[T]], %c1_i64
104|     // CHECK-NEXT: [[T_NEXT:%.*]] = arith.select [[EPILOGUE_COND]], %c0_i64, [[T_PLUS_1]]
105| 
106|     // CHECK-NEXT: yield [[T_NEXT]], [[I]], [[J_NEXT]] : i64, i64, i64
107|   } {"ttg.always-fuse"}
108|   tt.return
109| }
110| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 111-113
```mlir
111| // CHECK-LABEL: @fuse_one_level_inouts
112| // CHECK-SAME: [[LBI:%.*]]: i64, [[UBI:%.*]]: i64, [[STEPI:%.*]]: i64, [[LBJ:%.*]]: i64, [[UBJ:%.*]]: i64, [[STEPJ:%.*]]: i64
113| // CHECK-SAME: [[INOUT:%.*]]: index
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @fuse_one_level_inouts anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @fuse_one_level_inouts 这样的标签用于锚定匹配范围。

### Lines 114-114
```mlir
114| tt.func @fuse_one_level_inouts(%lbi: i64, %ubi: i64, %stepi: i64, %lbj: i64, %ubj: i64, %stepj: i64, %inout: index) -> index {
```
**EN:** This function-oriented block defines or enters `fuse_one_level_inouts`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `fuse_one_level_inouts` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 115-129
```mlir
115|   // CHECK: [[I_INIT:%.*]] = arith.subi [[LBI]], [[STEPI]]
116|   // CHECK: [[OUTER_OUTS:%.*]]:6 = scf.for %{{.*}} = %c0_i64 to [[TOTAL_ITERS:%.*]] step %c1_i64 iter_args(
117|   // CHECK-SAME: [[T:%arg[0-9]+]] = %c0_i64,
118|   // CHECK-SAME: [[I_ARG:%arg[0-9]+]] = [[I_INIT]]
119|   // CHECK-SAME: [[M:%arg[0-9]+]] = [[INOUT]]
120|   // CHECK-SAME: [[J_ARG:%arg[0-9]+]] = %c0_i64
121|   // CHECK-SAME: [[K_ARG:%arg[0-9]+]] = %c0
122|   // CHECK-SAME: [[PROLOGUE_OUT_ARG:%arg[0-9]+]] = %c0
123|   // CHECK-SAME: ) -> (i64, i64, index, i64, index, index) : i64 {
124|   %outer_out = scf.for %i = %lbi to %ubi step %stepi iter_args(%m = %inout) -> index : i64 {
125|     // if T == 0:
126|     //   i += stepi
127|     //   prologue(i)
128|     //   j = lbj
129|     //
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环。

### Lines 130-147
```mlir
130|     // CHECK:      [[PROLOGUE_COND:%.*]] = arith.cmpi eq, [[T]], %c0_i64
131|     // CHECK-NEXT: [[J:%.*]] = arith.select [[PROLOGUE_COND]], [[LBJ]], [[J_ARG]]
132|     // CHECK-NEXT: [[K:%.*]] = arith.select [[PROLOGUE_COND]], [[M]], [[K_ARG]]
133|     // CHECK-NEXT: [[PROLOGUE_OUTS:%.*]]:2 = scf.if [[PROLOGUE_COND]] -> (index, i64) {
134|     // CHECK-NEXT:   [[I:%.*]] = arith.addi [[I_ARG]], [[STEPI]]
135|     // CHECK-NEXT:   [[PROLOGUE_RES:%.*]] = "prologue"([[I]], [[INOUT]], [[M]]) : (i64, index, index) -> index
136|     // CHECK-NEXT:   yield [[PROLOGUE_RES]], [[I]]
137|     // CHECK-NEXT: } else {
138|     // CHECK-NEXT:   yield [[PROLOGUE_OUT_ARG]], [[I_ARG]]
139|     // CHECK-NEXT: }
140|     //
141|     // PROLOGUE_OUT := [[PROLOGUE_OUTS]]#0
142|     // I := [[PROLOGUE_OUTS]]#1
143|     %prologue_out = "prologue"(%i, %inout, %m) : (i64, index, index) -> index
144| 
145|     // if T >= 0 and T < len_j:
146|     //   body(i, j)
147|     //   j += stepj
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 148-148
```mlir
148|     //
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 149-164
```mlir
149|     // CHECK:      [[BODY_OUTS:%.*]]:2 = scf.if {{.*}} -> (i64, index) {
150|     // CHECK-NEXT:   [[BODY_OUT:%.*]] = "body"([[PROLOGUE_OUTS]]#1, [[J]], [[K]], [[PROLOGUE_OUTS]]#0, [[M]]) : (i64, i64, index, index, index) -> index
151|     // CHECK-NEXT:   [[J_INCR:%.*]] = arith.addi [[J]], [[STEPJ]]
152|     // CHECK-NEXT:   yield [[J_INCR]], [[BODY_OUT]]
153|     // CHECK-NEXT: } else {
154|     // CHECK-NEXT:   yield [[J]], [[K_ARG]]
155|     // CHECK-NEXT: }
156|     %inner_out = scf.for %j = %lbj to %ubj step %stepj iter_args(%k = %m) -> index : i64 {
157|       %body_out = "body"(%i, %j, %k, %prologue_out, %m) : (i64, i64, index, index, index) -> index
158|       scf.yield %body_out : index
159|     }
160| 
161|     // if T == max(1, len_j) - 1:
162|     //   epilogue(i)
163|     //   i += stepi
164|     //
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环、循环/分支产出值。

### Lines 165-172
```mlir
165|     // CHECK:      [[EPILOGUE_OUTS:%.*]] = scf.if {{.*}} -> (index) {
166|     // CHECK-NEXT:   [[EPILOGUE_OUT:%.*]] = "epilogue"([[PROLOGUE_OUTS]]#1, [[PROLOGUE_OUTS]]#0, [[BODY_OUTS]]#1, [[M]]) : (i64, index, index, index) -> index
167|     // CHECK-NEXT:   yield [[EPILOGUE_OUT]]
168|     // CHECK-NEXT: } else {
169|     // CHECK-NEXT:   yield [[M]]
170|     // CHECK-NEXT: }
171|     %epilogue_out = "epilogue"(%i, %prologue_out, %inner_out, %m) : (i64, index, index, index) -> index
172| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 173-175
```mlir
173|     // CHECK: yield %{{.*}}, [[PROLOGUE_OUTS]]#1, [[EPILOGUE_OUTS]], [[BODY_OUTS]]#0, [[BODY_OUTS]]#1, [[PROLOGUE_OUTS]]#0 : i64, i64, index, i64, index, index
174|     scf.yield %epilogue_out : index
175|   } {"ttg.always-fuse"}
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值。

### Lines 176-179
```mlir
176|   // CHECK: return [[OUTER_OUTS]]#2
177|   tt.return %outer_out : index
178| }
179| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 180-180
```mlir
180| // CHECK-LABEL: @multiple_loops
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @multiple_loops anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @multiple_loops 这样的标签用于锚定匹配范围。

### Lines 181-181
```mlir
181| tt.func @multiple_loops(
```
**EN:** This function-oriented block defines or enters `multiple_loops`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `multiple_loops` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 182-191
```mlir
182|     // CHECK-SAME: [[LBI:%arg[0-9]+]]: i64, [[UBI:%arg[0-9]+]]: i64, [[STEPI:%arg[0-9]+]]: i64,
183|     // CHECK-SAME: [[LBJ0:%arg[0-9]+]]: i64, [[UBJ0:%arg[0-9]+]]: i64, [[STEPJ0:%arg[0-9]+]]: i64,
184|     // CHECK-SAME: [[LBJ1:%arg[0-9]+]]: i64, [[UBJ1:%arg[0-9]+]]: i64, [[STEPJ1:%arg[0-9]+]]: i64,
185|     // CHECK-SAME: [[LBJ2:%arg[0-9]+]]: i64, [[UBJ2:%arg[0-9]+]]: i64, [[STEPJ2:%arg[0-9]+]]: i64,
186|     // CHECK-SAME: [[M0:%arg[0-9]+]]: f32
187|     %lbi: i64, %ubi: i64, %stepi: i64,
188|     %lbj0: i64, %ubj0: i64, %stepj0: i64,
189|     %lbj1: i64, %ubj1: i64, %stepj1: i64,
190|     %lbj2: i64, %ubj2: i64, %stepj2: i64,
191|     %m0: f32) -> f32 {
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 192-209
```mlir
192|   // CHECK:      [[DIFF_I:%.*]] = arith.subi [[UBI]], [[LBI]]
193|   // CHECK-NEXT: [[LEN_I:%.*]] = arith.ceildivsi [[DIFF_I]], [[STEPI]]
194|   // CHECK-NEXT: [[DIFF_J0:%.*]] = arith.subi [[UBJ0]], [[LBJ0]]
195|   // CHECK-NEXT: [[LEN_J0:%.*]] = arith.ceildivsi [[DIFF_J0]], [[STEPJ0]]
196|   // CHECK-NEXT: [[DIFF_J1:%.*]] = arith.subi [[UBJ1]], [[LBJ1]]
197|   // CHECK-NEXT: [[LEN_J1:%.*]] = arith.ceildivsi [[DIFF_J1]], [[STEPJ1]]
198|   // CHECK-NEXT: [[DIFF_J2:%.*]] = arith.subi [[UBJ2]], [[LBJ2]]
199|   // CHECK-NEXT: [[LEN_J2:%.*]] = arith.ceildivsi [[DIFF_J2]], [[STEPJ2]]
200| 
201|   // CHECK:      [[PLEN1:%.*]] = arith.maxsi [[LEN_J0]], %c1_i64
202|   // CHECK-NEXT: [[LEN_J1_CLAMP:%.*]] = arith.maxsi [[LEN_J1]], %c1_i64
203|   // CHECK-NEXT: [[PLEN2:%.*]] = arith.addi [[PLEN1]], [[LEN_J1_CLAMP]]
204|   // CHECK-NEXT: [[LEN_J2_CLAMP:%.*]] = arith.maxsi [[LEN_J2]], %c1_i64
205|   // CHECK-NEXT: [[PLEN3:%.*]] = arith.addi [[PLEN2]], [[LEN_J2_CLAMP]]
206|   // CHECK:      [[INNER_LEN:%.*]] = arith.subi [[PLEN3]], %c2_i64
207|   // CHECK-NEXT: [[TOTAL_ITERS:%.*]] = arith.muli [[LEN_I]], [[INNER_LEN]]
208| 
209|   // CHECK:      [[I_INIT:%.*]] = arith.subi [[LBI]], [[STEPI]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 210-224
```mlir
210|   // CHECK:      [[OUTS:%.*]]:12 = scf.for %{{.*}} = %c0_i64 to [[TOTAL_ITERS]] step %c1_i64 iter_args(
211|   // CHECK-SAME: [[T:%arg[0-9]+]] = %c0_i64,
212|   // CHECK-SAME: [[I_ARG:%arg[0-9]+]] = [[I_INIT]],
213|   // CHECK-SAME: [[M:%arg[0-9]+]] = [[M0]],
214|   // CHECK-SAME: [[J0_ARG:%arg[0-9]+]] = %c0_i64,
215|   // CHECK-SAME: [[J1_ARG:%arg[0-9]+]] = %c0_i64,
216|   // CHECK-SAME: [[J2_ARG:%arg[0-9]+]] = %c0_i64,
217|   // CHECK-SAME: [[BODY0_ARG:%arg[0-9]+]] = %cst,
218|   // CHECK-SAME: [[BODY1_ARG:%arg[0-9]+]] = %cst,
219|   // CHECK-SAME: [[BODY2_ARG:%arg[0-9]+]] = %cst,
220|   // CHECK-SAME: [[PROLOGUE0_ARG:%arg[0-9]+]] = %cst,
221|   // CHECK-SAME: [[PROLOGUE1_ARG:%arg[0-9]+]] = %cst,
222|   // CHECK-SAME: [[PROLOGUE2_ARG:%arg[0-9]+]] = %cst)
223|   %mN = scf.for %i = %lbi to %ubi step %stepi iter_args(%m = %m0) -> f32 : i64 {
224| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环。

### Lines 225-234
```mlir
225|     // CHECK-NEXT: [[PROLOGUE_COND0:%.*]] = arith.cmpi eq, [[T]], %c0_i64
226|     // CHECK-NEXT: [[J0:%.*]] = arith.select [[PROLOGUE_COND0]], [[LBJ0]], [[J0_ARG]]
227|     // CHECK-NEXT: [[PROLOGUE0_OUTS:%.*]]:3 = scf.if [[PROLOGUE_COND0]]
228|     // CHECK-NEXT:   [[I:%.*]] = arith.addi [[I_ARG]], [[STEPI]]
229|     // CHECK-NEXT:   [[RES:%.*]] = "prologue0"([[I]], [[M]])
230|     // CHECK-NEXT:   yield [[RES]], [[RES]], [[I]]
231|     // CHECK-NEXT: else
232|     // CHECK-NEXT:   yield [[PROLOGUE0_ARG]], [[BODY0_ARG]], [[I_ARG]]
233|     %k00 = "prologue0"(%i, %m) : (i64, f32) -> f32
234| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 235-248
```mlir
235|     // CHECK:      [[GE0:%.*]] = arith.cmpi sge, [[T]], %c0_i64
236|     // CHECK-NEXT: [[LT0:%.*]] = arith.cmpi slt, [[T]], [[LEN_J0]]
237|     // CHECK-NEXT: [[BODY_COND0:%.*]] = arith.andi [[GE0]], [[LT0]]
238|     // CHECK-NEXT: [[BODY0_OUTS:%.*]]:2 = scf.if [[BODY_COND0]]
239|     // CHECK-NEXT:   [[RES:%.*]] = "body0"([[PROLOGUE0_OUTS]]#2, [[J0]], [[PROLOGUE0_OUTS]]#1)
240|     // CHECK-NEXT:   [[NEXT_J0:%.*]] = arith.addi [[J0]], [[STEPJ0]]
241|     // CHECK-NEXT:   yield [[NEXT_J0]], [[RES]]
242|     // CHECK-NEXT: else
243|     // CHECK-NEXT:   yield [[J0]], [[BODY0_ARG]]
244|     %k0N = scf.for %j0 = %lbj0 to %ubj0 step %stepj0 iter_args(%k0 = %k00) -> f32 : i64 {
245|       %res = "body0"(%i, %j0, %k0) : (i64, i64, f32) -> f32
246|       scf.yield %res : f32
247|     }
248| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环、循环/分支产出值。

### Lines 249-258
```mlir
249|     // CHECK:      [[START1:%.*]] = arith.subi [[PLEN1]], %c1_i64
250|     // CHECK-NEXT: [[PROLOGUE_COND1:%.*]] = arith.cmpi eq, [[T]], [[START1]]
251|     // CHECK-NEXT: [[J1:%.*]] = arith.select [[PROLOGUE_COND1]], [[LBJ1]], [[J1_ARG]]
252|     // CHECK-NEXT: [[PROLOGUE1_OUTS:%.*]]:2 = scf.if [[PROLOGUE_COND1]]
253|     // CHECK-NEXT:   [[RES:%.*]] = "prologue1"([[PROLOGUE0_OUTS]]#2, [[BODY0_OUTS]]#1)
254|     // CHECK-NEXT:   yield [[RES]], [[RES]]
255|     // CHECK-NEXT: else
256|     // CHECK-NEXT:   yield [[PROLOGUE1_ARG]], [[BODY1_ARG]]
257|     %k10 = "prologue1"(%i, %k0N) : (i64, f32) -> f32
258| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 259-273
```mlir
259|     // CHECK:      [[END1:%.*]] = arith.addi [[START1]], [[LEN_J1]]
260|     // CHECK-NEXT: [[GE1:%.*]] = arith.cmpi sge, [[T]], [[START1]]
261|     // CHECK-NEXT: [[LT1:%.*]] = arith.cmpi slt, [[T]], [[END1]]
262|     // CHECK-NEXT: [[BODY_COND1:%.*]] = arith.andi [[GE1]], [[LT1]]
263|     // CHECK-NEXT: [[BODY1_OUTS:%.*]]:2 = scf.if [[BODY_COND1]]
264|     // CHECK-NEXT:   [[RES:%.*]] = "body1"([[PROLOGUE0_OUTS]]#2, [[J1]], [[PROLOGUE1_OUTS]]#1)
265|     // CHECK-NEXT:   [[NEXT_J1:%.*]] = arith.addi [[J1]], [[STEPJ1]]
266|     // CHECK-NEXT:   yield [[NEXT_J1]], [[RES]]
267|     // CHECK-NEXT: else
268|     // CHECK-NEXT:   yield [[J1]], [[BODY1_ARG]]
269|     %k1N = scf.for %j1 = %lbj1 to %ubj1 step %stepj1 iter_args(%k1 = %k10) -> f32 : i64 {
270|       %res = "body1"(%i, %j1, %k1) : (i64, i64, f32) -> f32
271|       scf.yield %res : f32
272|     }
273| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环、循环/分支产出值。

### Lines 274-283
```mlir
274|     // CHECK:      [[START2:%.*]] = arith.subi [[PLEN2]], %c2_i64
275|     // CHECK-NEXT: [[PROLOGUE_COND2:%.*]] = arith.cmpi eq, [[T]], [[START2]]
276|     // CHECK-NEXT: [[J2:%.*]] = arith.select [[PROLOGUE_COND2]], [[LBJ2]], [[J2_ARG]]
277|     // CHECK-NEXT: [[PROLOGUE2_OUTS:%.*]]:2 = scf.if [[PROLOGUE_COND2]]
278|     // CHECK-NEXT:   [[RES:%.*]] = "prologue2"([[PROLOGUE0_OUTS]]#2, [[BODY1_OUTS]]#1)
279|     // CHECK-NEXT:   yield [[RES]], [[RES]]
280|     // CHECK-NEXT: else
281|     // CHECK-NEXT:   yield [[PROLOGUE2_ARG]], [[BODY2_ARG]]
282|     %k20 = "prologue2"(%i, %k1N) : (i64, f32) -> f32
283| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 284-298
```mlir
284|     // CHECK:      [[END2:%.*]] = arith.addi [[START2]], [[LEN_J2]]
285|     // CHECK-NEXT: [[GE2:%.*]] = arith.cmpi sge, [[T]], [[START2]]
286|     // CHECK-NEXT: [[LT2:%.*]] = arith.cmpi slt, [[T]], [[END2]]
287|     // CHECK-NEXT: [[BODY_COND2:%.*]] = arith.andi [[GE2]], [[LT2]]
288|     // CHECK-NEXT: [[BODY2_OUTS:%.*]]:2 = scf.if [[BODY_COND2]]
289|     // CHECK-NEXT:   [[RES:%.*]] = "body2"([[PROLOGUE0_OUTS]]#2, [[J2]], [[PROLOGUE2_OUTS]]#1)
290|     // CHECK-NEXT:   [[NEXT_J2:%.*]] = arith.addi [[J2]], [[STEPJ2]]
291|     // CHECK-NEXT:   yield [[NEXT_J2]], [[RES]]
292|     // CHECK-NEXT: else
293|     // CHECK-NEXT:   yield [[J2]], [[BODY2_ARG]]
294|     %k2N = scf.for %j2 = %lbj2 to %ubj2 step %stepj2 iter_args(%k2 = %k20) -> f32 : i64 {
295|       %res = "body2"(%i, %j2, %k2) : (i64, i64, f32) -> f32
296|       scf.yield %res : f32
297|     }
298| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环、循环/分支产出值。

### Lines 299-307
```mlir
299|     // CHECK:      [[T_END:%.*]] = arith.subi [[PLEN3]], %c3_i64
300|     // CHECK-NEXT: [[EPILOGUE_COND:%.*]] = arith.cmpi eq, [[T]], [[T_END]]
301|     // CHECK-NEXT: [[EPILOGUE_OUTS:%.*]] = scf.if [[EPILOGUE_COND]]
302|     // CHECK-NEXT:   [[RES:%.*]] = "epilogue"([[PROLOGUE0_OUTS]]#2, [[BODY2_OUTS]]#1)
303|     // CHECK-NEXT:   yield [[RES]]
304|     // CHECK-NEXT:  else
305|     // CHECK-NEXT:   yield [[M]]
306|     %out = "epilogue"(%i, %k2N) : (i64, f32) -> f32
307| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 308-315
```mlir
308|     // CHECK:      [[T_PLUS_1:%.*]] = arith.addi [[T]], %c1_i64
309|     // CHECK-NEXT: [[T_NEXT:%.*]] = arith.select [[EPILOGUE_COND]], %c0_i64, [[T_PLUS_1]]
310| 
311|     // CHECK:      scf.yield [[T_NEXT]], [[PROLOGUE0_OUTS]]#2, [[EPILOGUE_OUTS]],
312|     // CHECK-SAME:           [[BODY0_OUTS]]#0, [[BODY1_OUTS]]#0, [[BODY2_OUTS]]#0,
313|     // CHECK-SAME:           [[PROLOGUE0_OUTS]]#0, [[PROLOGUE1_OUTS]]#0, [[PROLOGUE2_OUTS]]#0 :
314|     scf.yield %out : f32
315|   } {"ttg.always-fuse"}
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值。

### Lines 316-319
```mlir
316|   // CHECK: return [[OUTS]]#2
317|   tt.return %mN : f32
318| }
319| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 320-320
```mlir
320| // CHECK-LABEL: @two_loop_nests
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @two_loop_nests anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @two_loop_nests 这样的标签用于锚定匹配范围。

### Lines 321-321
```mlir
321| tt.func @two_loop_nests(%lbi: i64, %ubi: i64, %stepi: i64, %lbj: i64, %ubj: i64, %stepj: i64) {
```
**EN:** This function-oriented block defines or enters `two_loop_nests`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `two_loop_nests` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 322-332
```mlir
322|   // CHECK-COUNT-2: scf.for
323|   scf.for %i = %lbi to %ubi step %stepi : i64 {
324|     scf.for %j = %lbj to %ubj step %stepj : i64 {
325|       "body"(%i, %j) : (i64, i64) -> ()
326|     }
327|   } {"ttg.always-fuse"}
328|   scf.for %i = %lbi to %ubi step %stepi : i64 {
329|     scf.for %j = %lbj to %ubj step %stepj : i64 {
330|       "body"(%i, %j) : (i64, i64) -> ()
331|     }
332|   } {"ttg.always-fuse"}
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环。

### Lines 333-337
```mlir
333|   // CHECK-NOT: scf.for
334|   // CHECK: tt.return
335|   tt.return
336| }
337| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 tt.return。

### Lines 338-339
```mlir
338| // CHECK-LABEL: @hoist_loop_bound_computations
339| // CHECK-SAME: [[LBI:%.*]]: i64, [[UBI:%.*]]: i64, [[STEPI:%.*]]: i64
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @hoist_loop_bound_computations anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @hoist_loop_bound_computations 这样的标签用于锚定匹配范围。

### Lines 340-340
```mlir
340| tt.func @hoist_loop_bound_computations(%lbi: i64, %ubi: i64, %stepi: i64) {
```
**EN:** This function-oriented block defines or enters `hoist_loop_bound_computations`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `hoist_loop_bound_computations` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 341-354
```mlir
341|   // CHECK:      [[LBJ:%.*]] = arith.addi [[LBI]], [[STEPI]]
342|   // CHECK-NEXT: [[UBJ:%.*]] = arith.addi [[UBI]], [[STEPI]]
343|   // CHECK-NEXT: [[STEPJ:%.*]] = arith.addi [[STEPI]], [[STEPI]]
344| 
345|   // CHECK-NEXT: [[DIFF_I:%.*]] = arith.subi [[UBI]], [[LBI]]
346|   // CHECK-NEXT: [[LEN_I:%.*]] = arith.ceildivsi [[DIFF_I]], [[STEPI]]
347|   // CHECK-NEXT: [[DIFF_J:%.*]] = arith.subi [[UBJ]], [[LBJ]]
348|   // CHECK-NEXT: [[LEN_J:%.*]] = arith.ceildivsi [[DIFF_J]], [[STEPJ]]
349| 
350|   // CHECK: scf.for
351|   scf.for %i = %lbi to %ubi step %stepi : i64 {
352|     %lbj = arith.addi %lbi, %stepi : i64
353|     %ubj = arith.addi %ubi, %stepi : i64
354|     %stepj = arith.addi %stepi, %stepi : i64
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining integer additions, structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 整数加法、结构化循环。

### Lines 355-367
```mlir
355|     // CHECK: [[J:%.*]] = arith.select %{{.*}}, [[LBJ]], %arg{{[0-9]+}}
356|     // CHECK-NEXT: scf.if
357| 
358|     // CHECK: scf.if
359|     // CHECK-NEXT: "body"
360|     // CHECK-NEXT: arith.addi [[J]], [[STEPJ]]
361|     scf.for %j = %lbj to %ubj step %stepj : i64 {
362|       "body"(%i, %j) : (i64, i64) -> ()
363|     }
364|   } {"ttg.always-fuse"}
365|   tt.return
366| }
367| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环、tt.return。

### Lines 368-369
```mlir
368| // CHECK-LABEL: @dependent_inner_loop
369| // CHECK-SAME: [[LBI:%.*]]: i64, [[UBI:%.*]]: i64, [[STEPI:%.*]]: i64
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @dependent_inner_loop anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @dependent_inner_loop 这样的标签用于锚定匹配范围。

### Lines 370-370
```mlir
370| tt.func @dependent_inner_loop(%lbi: i64, %ubi: i64, %stepi: i64) {
```
**EN:** This function-oriented block defines or enters `dependent_inner_loop`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `dependent_inner_loop` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 371-388
```mlir
371|   // CHECK:      [[TOTAL_ITERS:%.*]] = scf.for [[I:%.*]] = [[LBI]] to [[UBI]] step [[STEPI]] iter_args([[SUM:%.*]] = %c0_i64)
372|   // CHECK-NEXT:   [[LBJ:%.*]] = arith.addi [[LBI]], [[STEPI]]
373|   // CHECK-NEXT:   [[UBJ:%.*]] = arith.addi [[UBI]], [[I]]
374|   // CHECK-NEXT:   [[STEPJ:%.*]] = arith.addi [[STEPI]], [[STEPI]]
375|   // CHECK-NEXT:   [[DIFF_J:%.*]] = arith.subi [[UBJ]], [[LBJ]]
376|   // CHECK-NEXT:   [[LEN_J:%.*]] = arith.ceildivsi [[DIFF_J]], [[STEPJ]]
377|   // CHECK-NEXT:   [[CLAMPED_LEN_J:%.*]] = arith.maxsi [[LEN_J]], %c1_i64
378|   // CHECK-NEXT:   [[ACC:%.*]] = arith.addi [[SUM]], [[CLAMPED_LEN_J]]
379|   // CHECK-NEXT:   yield [[ACC]]
380|   // CHECK-NEXT: }
381| 
382|   // CHECK-NEXT: [[I_INIT:%.*]] = arith.subi [[LBI]], [[STEPI]]
383|   // CHECK-NEXT: [[OUTS:%.*]]:8 = scf.for {{.*}} = %c0_i64 to [[TOTAL_ITERS]] step %c1_i64 iter_args(
384|   // CHECK-SAME: [[T:%arg[0-9]+]] = %c0_i64,
385|   // CHECK-SAME: [[I_ARG:%arg[0-9]+]] = [[I_INIT]],
386|   // CHECK-SAME: [[J_ARG:%arg[0-9]+]] = %c0_i64,
387|   scf.for %i = %lbi to %ubi step %stepi : i64 {
388|     %lbj = arith.addi %lbi, %stepi : i64
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops, integer additions.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环、整数加法。

### Lines 389-399
```mlir
389|     %ubj = arith.addi %ubi, %i : i64
390|     %stepj = arith.addi %stepi, %stepi : i64
391|     "prologue"(%i) : (i64) -> ()
392|     scf.for %j = %lbj to %ubj step %stepj : i64 {
393|       "body"(%i, %j) : (i64, i64) -> ()
394|     }
395|     "epilogue"(%i) : (i64) -> ()
396|   } {"ttg.always-fuse"}
397|   tt.return
398| }
399| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining integer additions, structured loops, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 整数加法、结构化循环、tt.return。

### Lines 400-401
```mlir
400| // CHECK-LABEL: @upcast_i16_to_i32
401| // CHECK-SAME: [[LBI:%.*]]: i32, [[UBI:%.*]]: i32, [[STEPI:%.*]]: i32, [[LBJ:%.*]]: i16, [[UBJ:%.*]]: i16, [[STEPJ:%.*]]: i16
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @upcast_i16_to_i32 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @upcast_i16_to_i32 这样的标签用于锚定匹配范围。

### Lines 402-402
```mlir
402| tt.func @upcast_i16_to_i32(%lbi: i32, %ubi: i32, %stepi: i32, %lbj: i16, %ubj: i16, %stepj: i16) {
```
**EN:** This function-oriented block defines or enters `upcast_i16_to_i32`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `upcast_i16_to_i32` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 403-416
```mlir
403|   // CHECK:      [[DIFF_I:%.*]] = arith.subi [[UBI]], [[LBI]] : i32
404|   // CHECK-NEXT: [[LEN_I:%.*]] = arith.ceildivsi [[DIFF_I]], [[STEPI]] : i32
405|   // CHECK-NEXT: [[DIFF_J:%.*]] = arith.subi [[UBJ]], [[LBJ]] : i16
406|   // CHECK-NEXT: [[LEN_J:%.*]] = arith.ceildivsi [[DIFF_J]], [[STEPJ]] : i16
407| 
408|   // CHECK: arith.extsi [[LEN_J]] : i16 to i32
409|   scf.for %i = %lbi to %ubi step %stepi : i32 {
410|     scf.for %j = %lbj to %ubj step %stepj : i16 {
411|       "body"(%i, %j) : (i32, i16) -> ()
412|     }
413|   } {"ttg.always-fuse"}
414|   tt.return
415| }
416| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环、tt.return。

### Lines 417-418
```mlir
417| // CHECK-LABEL: @upcast_index_to_i64
418| // CHECK-SAME: [[LBI:%.*]]: index, [[UBI:%.*]]: index, [[STEPI:%.*]]: index, [[LBJ:%.*]]: index, [[UBJ:%.*]]: index, [[STEPJ:%.*]]: index
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @upcast_index_to_i64 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @upcast_index_to_i64 这样的标签用于锚定匹配范围。

### Lines 419-419
```mlir
419| tt.func @upcast_index_to_i64(%lbi: index, %ubi: index, %stepi: index, %lbj: index, %ubj: index, %stepj: index) {
```
**EN:** This function-oriented block defines or enters `upcast_index_to_i64`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `upcast_index_to_i64` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 420-434
```mlir
420|   // CHECK:      [[DIFF_I:%.*]] = arith.subi [[UBI]], [[LBI]] : index
421|   // CHECK-NEXT: [[LEN_I:%.*]] = arith.ceildivsi [[DIFF_I]], [[STEPI]] : index
422|   // CHECK-NEXT: [[DIFF_J:%.*]] = arith.subi [[UBJ]], [[LBJ]] : index
423|   // CHECK-NEXT: [[LEN_J:%.*]] = arith.ceildivsi [[DIFF_J]], [[STEPJ]] : index
424| 
425|   // CHECK: arith.index_cast [[LEN_J]] : index to i64
426|   // CHECK: arith.index_cast [[LEN_I]] : index to i64
427|   scf.for %i = %lbi to %ubi step %stepi {
428|     scf.for %j = %lbj to %ubj step %stepj {
429|       "body"(%i, %j) : (index, index) -> ()
430|     }
431|   } {"ttg.always-fuse"}
432|   tt.return
433| }
434| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环、tt.return。

### Lines 435-435
```mlir
435| // CHECK-LABEL: @triple_loop_nest
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @triple_loop_nest anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @triple_loop_nest 这样的标签用于锚定匹配范围。

### Lines 436-439
```mlir
436| tt.func @triple_loop_nest(
437|     %lbi: i64, %ubi: i64, %stepi: i64,
438|     %lbj: i64, %ubj: i64, %stepj: i64,
439|     %lbk: i64, %ubk: i64, %stepk: i64) {
```
**EN:** This function-oriented block defines or enters `triple_loop_nest`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `triple_loop_nest` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 440-447
```mlir
440|  // CHECK-COUNT-1: scf.for
441|  scf.for %i = %lbi to %ubi step %stepi : i64 {
442|    scf.for %j = %lbj to %ubj step %stepj : i64 {
443|       scf.for %k = %lbk to %ubk step %stepk : i64 {
444|         "body"(%i, %j, %k) : (i64, i64, i64) -> ()
445|       }
446|     }
447|   } {"ttg.always-fuse"}
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环。

### Lines 448-452
```mlir
448|   // CHECK-NOT: scf.for
449|   // CHECK: tt.return
450|   tt.return
451| }
452| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 tt.return。

### Lines 453-453
```mlir
453| // CHECK-LABEL: @preserve_stage_count
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @preserve_stage_count anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @preserve_stage_count 这样的标签用于锚定匹配范围。

### Lines 454-456
```mlir
454| tt.func @preserve_stage_count(%lb: i32, %ub: i32) {
455|   %c1_i32 = arith.constant 1 : i32
456| 
```
**EN:** This function-oriented block defines or enters `preserve_stage_count`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `preserve_stage_count` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 457-467
```mlir
457|   // CHECK-COUNT-1: scf.for
458|   scf.for %i = %lb to %ub step %c1_i32 : i32 {
459|     scf.for %j = %lb to %ub step %c1_i32 : i32 {
460|       "body"(%j) : (i32) -> ()
461|       scf.yield
462|     } {tt.num_stages = 4 : i32}
463|     scf.for %j = %lb to %ub step %c1_i32 : i32 {
464|       "body"(%j) : (i32) -> ()
465|       scf.yield
466|     } {tt.num_stages = 5 : i32}
467|   } {"ttg.always-fuse", "tt.disallow_acc_multi_buffer", tt.num_stages = 6 : i32}
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环、循环/分支产出值。

### Lines 468-473
```mlir
468|   // CHECK: tt.disallow_acc_multi_buffer
469|   // CHECK: tt.num_stages = 6 : i32
470|   // CHECK-NOT: scf.for
471|   tt.return
472| }
473| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 tt.return。

### Lines 474-475
```mlir
474| // CHECK-LABEL: @fuse_attr_speculate
475| // CHECK-SAME: [[LB:%.*]]: i32, [[UB:%.*]]: i32
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @fuse_attr_speculate anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @fuse_attr_speculate 这样的标签用于锚定匹配范围。

### Lines 476-478
```mlir
476| tt.func @fuse_attr_speculate(%lb: i32, %ub: i32) {
477|   %c1_i32 = arith.constant 1 : i32
478| 
```
**EN:** This function-oriented block defines or enters `fuse_attr_speculate`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `fuse_attr_speculate` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 479-490
```mlir
479|   // CHECK: [[LEN:%.*]] = arith.subi [[UB]], [[LB]]
480|   // CHECK: [[IS_ZERO:%.*]] = arith.cmpi eq, [[LEN]], %c0_i32
481| 
482|   // CHECK: scf.if [[IS_ZERO]]
483|   // CHECK-NEXT: scf.for %{{.*}} = [[LB]] to [[UB]] step %c1_i32
484|   // CHECK-NEXT:   "prologue"
485|   // CHECK-NXET: } {tt.flatten}
486| 
487|   // CHECK: else
488|   // CHECK-COUNT-1: scf.for
489|   // CHECK-NOT: scf.for
490|   scf.for %i = %lb to %ub step %c1_i32 : i32 {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环。

### Lines 491-494
```mlir
491|     // CHECK: scf.if
492|     // CHECK-NEXT: arith.addi
493|     // CHECK-NEXT: "prologue"
494|     "prologue"(%i) : (i32) -> ()
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 495-498
```mlir
495|     // CHECK: else
496|     // CHECK-NEXT: scf.yield
497|     // CHECK-NEXT: }
498|     scf.for %j = %lb to %ub step %c1_i32 : i32 {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环。

### Lines 499-506
```mlir
499|       // CHECK-NEXT: "body"
500|       "body"(%i, %j) : (i32, i32) -> ()
501|       scf.yield
502|     }
503|   } {tt.flatten, tt.warp_specialize}
504|   tt.return
505| }
506| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、tt.return。

### Lines 507-508
```mlir
507| // CHECK-LABEL: @speculate_hoist
508| // CHECK-SAME: [[LB:%.*]]: i32, [[UB:%.*]]: i32
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @speculate_hoist anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @speculate_hoist 这样的标签用于锚定匹配范围。

### Lines 509-511
```mlir
509| tt.func @speculate_hoist(%lb: i32, %ub: i32) {
510|   %c1_i32 = arith.constant 1 : i32
511| 
```
**EN:** This function-oriented block defines or enters `speculate_hoist`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `speculate_hoist` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 512-525
```mlir
512|   // CHECK: [[IS_ZERO:%.*]] = arith.cmpi eq, [[UB]], %c0_i32
513| 
514|   // CHECK: scf.if [[IS_ZERO]]
515|   scf.for %i = %lb to %ub step %c1_i32 : i32 {
516|     "prologue"(%i) : (i32) -> ()
517|     %ubj = arith.addi %lb, %ub : i32
518|     scf.for %j = %lb to %ubj step %c1_i32 : i32 {
519|       "body"(%i, %j) : (i32, i32) -> ()
520|       scf.yield
521|     }
522|   } {tt.flatten}
523|   tt.return
524| }
525| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops, integer additions, loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环、整数加法、循环/分支产出值、tt.return。

### Lines 526-527
```mlir
526| // CHECK-LABEL: @sink_prologue_to_epilogue
527| // CHECK-SAME: [[UB:%.*]]: i32
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @sink_prologue_to_epilogue anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @sink_prologue_to_epilogue 这样的标签用于锚定匹配范围。

### Lines 528-531
```mlir
528| tt.func @sink_prologue_to_epilogue(%ub: i32) {
529|   %c0_i32 = arith.constant 0 : i32
530|   %c1_i32 = arith.constant 1 : i32
531| 
```
**EN:** This function-oriented block defines or enters `sink_prologue_to_epilogue`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `sink_prologue_to_epilogue` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 532-534
```mlir
532|   // CHECK: else
533|   // CHECK: scf.for
534|   %0 = scf.for %i = %c0_i32 to %ub step %c1_i32 iter_args(%k = %c0_i32) -> i32 : i32 {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环。

### Lines 535-536
```mlir
535|     // CHECK: [[PROLOGUE_OUTS:%.*]] = scf.if
536|     %0 = arith.addi %i, %ub : i32
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining integer additions.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 整数加法。

### Lines 537-544
```mlir
537|     // CHECK: else
538|     // CHECK-NEXT: scf.yield
539|     // CHECK-NEXT: }
540|     // CHECK-NEXT: "body"
541|     scf.for %j = %c0_i32 to %ub step %c1_i32 : i32 {
542|       "body"(%i, %j) : (i32, i32) -> ()
543|       scf.yield
544|     }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环、循环/分支产出值。

### Lines 545-548
```mlir
545|     // CHECK: scf.if
546|     // CHECK-NEXT: [[V0:%.*]] = arith.addi [[PROLOGUE_OUTS]], [[UB]]
547|     // CHECK-NEXT: [[V1:%.*]] = arith.addi [[V0]], [[UB]]
548|     %1 = arith.addi %0, %ub : i32
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining integer additions.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 整数加法。

### Lines 549-556
```mlir
549|     // CHECK-NEXT: "epilogue"([[V1]])
550|     "epilogue"(%1) : (i32) -> ()
551|     scf.yield %0 : i32
552|   } {tt.flatten}
553| 
554|   tt.return
555| }
556| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、tt.return。

### Lines 557-557
```mlir
557| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 558-559
```mlir
558| 
559| // CHECK-LABEL: @prologue_output
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @prologue_output anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @prologue_output 这样的标签用于锚定匹配范围。

### Lines 560-563
```mlir
560| tt.func @prologue_output(%ub: i32) {
561|   %c0_i32 = arith.constant 0 : i32
562|   %c1_i32 = arith.constant 1 : i32
563| 
```
**EN:** This function-oriented block defines or enters `prologue_output`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `prologue_output` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 564-565
```mlir
564|   // CHECK: scf.for
565|   %0 = scf.for %i = %c0_i32 to %ub step %c1_i32 iter_args(%k = %c0_i32) -> i32 : i32 {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环。

### Lines 566-568
```mlir
566|     // CHECK: scf.if
567|     // CHECK: {increment}
568|     %next = arith.addi %k, %c1_i32 {increment} : i32
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining integer additions.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 整数加法。

### Lines 569-570
```mlir
569|     // CHECK: scf.if
570|     scf.for %j = %c0_i32 to %ub step %c1_i32 : i32 {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环。

### Lines 571-573
```mlir
571|       // CHECK-NEXT: "body"
572|       "body"(%i, %j) : (i32, i32) -> ()
573|     }
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 574-576
```mlir
574|     // CHECK: scf.if {{%[0-9]+}} {
575|     // CHECK-NEXT: "epilogue"
576|     "epilogue"(%i) : (i32) -> ()
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 577-582
```mlir
577|     // CHECK-NEXT: }
578|     scf.yield %next : i32
579|   } {"ttg.always-fuse"}
580| 
581|   tt.return
582| }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、tt.return。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `--allow-unregistered-dialect`, `--tritongpu-fuse-nested-loops`, `-canonicalize`, `-cse`
- **CN:** 主要 pass 选项：`--allow-unregistered-dialect`，`--tritongpu-fuse-nested-loops`，`-canonicalize`，`-cse`
- **EN:** Dominant operations include `scf.for`, `tt.func`, `tt.return`, `scf.yield`, `arith.addi`, `arith.constant`.
- **CN:** 主要操作包括 `scf.for`、`tt.func`、`tt.return`、`scf.yield`、`arith.addi`、`arith.constant`。
- **EN:** The file contains 1 independently testable section(s). Check styles used: CHECK-NEXT x161, CHECK x62, CHECK-SAME x40, CHECK-LABEL x16. Important labels include @empty_function, @no_fusion, @fuse_one_level_simple, @fuse_one_level_inouts. It validates both the presence of the desired rewrite and the absence of unwanted residual operations.
- **CN:** 该文件包含 1 个可独立测试的分段。使用的检查类型：CHECK-NEXT ×161，CHECK ×62，CHECK-SAME ×40，CHECK-LABEL ×16。 关键标签包括 @empty_function，@no_fusion，@fuse_one_level_simple，@fuse_one_level_inouts。 它同时验证目标重写已经出现，并且不再残留不需要的操作。

## Dependencies / 依赖关系

- **EN:** This test depends on Triton/MLIR textual IR parsing, the pass pipeline named in the RUN lines, and the dialect/type definitions referenced in the file.
- **CN:** 该测试依赖 Triton/MLIR 文本 IR 解析、RUN 行中指定的 pass 流水线，以及文件中引用的方言/类型定义。
- **EN:** `scf`: Structured control flow for loops and conditionals.
- **CN:** `scf`：用于循环与条件分支的结构化控制流。
- **EN:** `tt`: Triton high-level tensor/pointer operations.
- **CN:** `tt`：Triton 高层张量/指针操作。
- **EN:** `arith`: MLIR arithmetic dialect for scalar/tensor math.
- **CN:** `arith`：用于标量/张量运算的 MLIR arithmetic 方言。
- **EN:** `FileCheck` is used to match required textual patterns after rewriting/lowering.
- **CN:** `FileCheck` 用于在重写/lower 之后匹配必须出现的文本模式。