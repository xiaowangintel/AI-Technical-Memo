# partition-loops.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/partition-loops.mlir`
- **EN:** Mixed regression test for `-tritongpu-partition-loops, -verify-diagnostics, -canonicalize`: most sections are checked with FileCheck, while some sections intentionally trigger diagnostics.
- **CN:** 这是针对 `-tritongpu-partition-loops, -verify-diagnostics, -canonicalize` 的混合回归测试：大部分分段用 FileCheck 验证，部分分段则故意触发诊断。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -split-input-file -allow-unregistered-dialect -tritongpu-partition-loops -verify-diagnostics -canonicalize | FileCheck %s`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -split-input-file -allow-unregistered-dialect -tritongpu-partition-loops -verify-diagnostics -canonicalize | FileCheck %s`

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
1| // RUN: triton-opt %s -split-input-file -allow-unregistered-dialect -tritongpu-partition-loops -verify-diagnostics -canonicalize | FileCheck %s
2| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -split-input-file -allow-unregistered-dialect -tritongpu-partition-loops -verify-diagnostics -canonicalize | FileCheck %s` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -split-input-file -allow-unregistered-dialect -tritongpu-partition-loops -verify-diagnostics -canonicalize | FileCheck %s`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 3-5
```mlir
3| #blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
4| !ty = tensor<1xi32, #blocked>
5| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 6-7
```mlir
6| module attributes {"ttg.num-warps" = 4 : i32} {
7| 
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 8-8
```mlir
8| // CHECK-LABEL: @one_partition
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @one_partition anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @one_partition 这样的标签用于锚定匹配范围。

### Lines 9-9
```mlir
9| tt.func @one_partition(%lb: i32, %ub: i32, %step: i32) {
```
**EN:** This function-oriented block defines or enters `one_partition`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `one_partition` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 10-11
```mlir
10|   // CHECK-NEXT: scf.for
11|   scf.for %i = %lb to %ub step %step : i32 {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环。

### Lines 12-17
```mlir
12|     // CHECK-NEXT: op_a
13|     "op_a"() {ttg.partition = array<i32: 0>} : () -> ()
14|   } {ttg.partition.stages = [0], ttg.warp_specialize.tag = 0 : i32, ttg.partition = array<i32: 0>}
15|   tt.return
16| }
17| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 18-18
```mlir
18| // CHECK-LABEL: @two_empty_partitions
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @two_empty_partitions anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @two_empty_partitions 这样的标签用于锚定匹配范围。

### Lines 19-19
```mlir
19| tt.func @two_empty_partitions(%lb: i32, %ub: i32, %step: i32) {
```
**EN:** This function-oriented block defines or enters `two_empty_partitions`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `two_empty_partitions` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 20-37
```mlir
20|   // CHECK-NEXT: nvws.warp_group
21|   // CHECK-NEXT: partition0 num_warps(4)
22|   // CHECK-NEXT:   scf.for [[I:%.*]] = %arg0 to %arg1 step %arg2
23|   // CHECK-NEXT:     "op_a"([[I]])
24|   // CHECK-NEXT:   }
25|   // CHECK-NEXT:   nvws.warp_group.yield
26|   // CHECK-NEXT: }
27|   // CHECK-NEXT: partition1 num_warps(4)
28|   // CHECK-NEXT:   scf.for [[I:%.*]] = %arg0 to %arg1 step %arg2
29|   // CHECK-NEXT:     "op_a"([[I]])
30|   // CHECK-NEXT:   }
31|   // CHECK-NEXT:   nvws.warp_group.return
32|   scf.for %i = %lb to %ub step %step : i32 {
33|     "op_a"(%i) {ttg.partition = array<i32: 0, 1>} : (i32) -> ()
34|   } {ttg.partition.stages = [0, 0], ttg.warp_specialize.tag = 0 : i32, ttg.partition = array<i32: 0, 1>}
35|   tt.return
36| }
37| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环、tt.return。

### Lines 38-38
```mlir
38| // CHECK-LABEL: @empty_partition_fwd_root
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @empty_partition_fwd_root anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @empty_partition_fwd_root 这样的标签用于锚定匹配范围。

### Lines 39-39
```mlir
39| tt.func @empty_partition_fwd_root(%lb: i32, %ub: i32, %step: i32) {
```
**EN:** This function-oriented block defines or enters `empty_partition_fwd_root`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `empty_partition_fwd_root` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 40-41
```mlir
40|   // CHECK-NEXT: [[C0:%.*]] = arith.constant 0
41|   %c0_i32 = arith.constant 0 : i32
```
**EN:** This operational block carries the concrete IR steps for the test, centered on constants. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 常量。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 42-51
```mlir
42|   // CHECK: partition0
43|   // CHECK-NEXT: scf.for [[I:%.*]] = {{.*}} iter_args([[K:%.*]] = [[C0]])
44|   // CHECK-NEXT:   "op_a"([[I]], [[K]])
45|   scf.for %i = %lb to %ub step %step iter_args(%k = %c0_i32) -> i32 : i32 {
46|     %0 = "op_a"(%i, %k) {ttg.partition = array<i32: 0, 1>} : (i32, i32) -> i32
47|     scf.yield {ttg.partition = array<i32: 0, 1>} %0 : i32
48|   } {ttg.partition.stages = [0, 0], ttg.warp_specialize.tag = 0 : i32, ttg.partition = array<i32: 0, 1>, ttg.partition.outputs = [array<i32: 0, 1>]}
49|   tt.return
50| }
51| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops, loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环、循环/分支产出值、tt.return。

### Lines 52-52
```mlir
52| // CHECK-LABEL: @multiple_partitions
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @multiple_partitions anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @multiple_partitions 这样的标签用于锚定匹配范围。

### Lines 53-53
```mlir
53| tt.func @multiple_partitions(%lb: i32, %ub: i32, %step: i32) {
```
**EN:** This function-oriented block defines or enters `multiple_partitions`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `multiple_partitions` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 54-71
```mlir
54|   // CHECK: partition0 num_warps(4)
55|   // CHECK-NEXT: scf.for
56|   // CHECK-NEXT:   [[X:%.*]] = "op_a"
57|   // CHECK-NEXT:   "op_b"([[X]])
58|   // CHECK-NEXT:   "op_b"([[X]])
59|   // CHECK-NEXT: }
60| 
61|   // CHECK: partition1
62|   // CHECK-NEXT: scf.for [[I:%arg[0-9]+]]
63|   // CHECK-NEXT:   [[Y:%.*]] = arith.addi [[I]], [[I]]
64|   // CHECK-NEXT:   [[X:%.*]] = "op_a"([[Y]])
65|   // CHECK-NEXT:   "op_b"([[X]])
66|   // CHECK-NEXT:   "op_b"([[X]])
67|   // CHECK-NEXT: }
68| 
69|   // CHECK: partition2
70|   // CHECK-NEXT: scf.for [[I:%arg[0-9]+]]
71|   // CHECK-NEXT:   [[Y:%.*]] = arith.addi [[I]], [[I]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 72-89
```mlir
72|   // CHECK-NEXT:   [[Z:%.*]] = arith.addi [[I]], [[Y]]
73|   // CHECK-NEXT:   [[X:%.*]] = "op_a"([[Z]])
74|   // CHECK-NEXT:   "op_b"([[X]])
75|   // CHECK-NEXT:   "op_b"([[X]])
76|   // CHECK-NEXT: }
77| 
78|   scf.for %i = %lb to %ub step %step : i32 {
79|     %a = arith.addi %i, %i {ttg.partition = array<i32: 1, 2>} : i32
80|     %b = arith.addi %i, %a {ttg.partition = array<i32: 1, 2>}: i32
81| 
82|     %0 = "op_a"(%i) {ttg.partition = array<i32: 0>} : (i32) -> i32
83|     "op_b"(%0) {ttg.partition = array<i32: 0>} : (i32) -> ()
84|     "op_b"(%0) {ttg.partition = array<i32: 0>} : (i32) -> ()
85| 
86|     %1 = "op_a"(%a) {ttg.partition = array<i32: 1>} : (i32) -> i32
87|     "op_b"(%1) {ttg.partition = array<i32: 1>} : (i32) -> ()
88|     "op_b"(%1) {ttg.partition = array<i32: 1>} : (i32) -> ()
89| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining integer additions, structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 整数加法、结构化循环。

### Lines 90-96
```mlir
90|     %2 = "op_a"(%b) {ttg.partition = array<i32: 2>} : (i32) -> i32
91|     "op_b"(%2) {ttg.partition = array<i32: 2>} : (i32) -> ()
92|     "op_b"(%2) {ttg.partition = array<i32: 2>} : (i32) -> ()
93|   } {ttg.partition.stages = [0, 0, 0], ttg.warp_specialize.tag = 0 : i32, ttg.partition = array<i32: 0, 1, 2>}
94|   tt.return
95| }
96| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 97-97
```mlir
97| // CHECK-LABEL: @multiple_partitions_two_loops
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @multiple_partitions_two_loops anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @multiple_partitions_two_loops 这样的标签用于锚定匹配范围。

### Lines 98-99
```mlir
98| tt.func @multiple_partitions_two_loops(%lb: i32, %ub: i32, %step: i32,
99|                                        %c0 : i32, %c1 : i32, %c2 : i32) {
```
**EN:** This function-oriented block defines or enters `multiple_partitions_two_loops`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `multiple_partitions_two_loops` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 100-117
```mlir
100|   // CHECK: "op_b"
101|   // CHECK-NEXT: nvws.warp_group
102|   // CHECK-NEXT: partition0 num_warps(4)
103|   // CHECK-NEXT: op_00b
104|   // CHECK-NEXT: [[RET:%.*]]:3 = scf.for [[I:%.*]] = [[LB:%.*]] to [[UB:%.*]] step [[STEP:%.*]] iter_args([[ARG0:%.*]] = {{.*}}, [[ARG1:%.*]] = {{.*}}, [[ARG2:%.*]] = {{.*}}) -> (i32, i32, i32) : i32 {
105|   // CHECK-NEXT:   [[X:%.*]] = "op_a"
106|   // CHECK-NEXT:   "op_b"([[ARG0]])
107|   // CHECK-NEXT:   "op_b"([[X]])
108|   // CHECK-NEXT:   arith.addi
109|   // CHECK-NEXT:   arith.addi
110|   // CHECK-NEXT:   arith.addi
111|   // CHECK-NEXT:   scf.yield
112|   // CHECK-NEXT: }
113|   // CHECK-NEXT: "op_00e"([[RET]]#0)
114| 
115|   // CHECK: partition1
116|   // CHECK-NEXT: op_01b
117|   // CHECK-NEXT: [[RET:%.*]] = scf.for [[I:%.*]] = [[LB:%.*]] to [[UB:%.*]] step [[STEP:%.*]] iter_args([[ARG1:%.*]] = {{.*}}) -> (i32) : i32 {
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 118-135
```mlir
118|   // CHECK-NEXT:   [[Y:%.*]] = arith.addi [[I]], [[I]]
119|   // CHECK-NEXT:   [[X:%.*]] = "op_a"([[Y]])
120|   // CHECK-NEXT:   "op_b"([[ARG1]])
121|   // CHECK-NEXT:   "op_b"([[X]])
122|   // CHECK-NEXT:   arith.addi
123|   // CHECK-NEXT:   scf.yield
124|   // CHECK-NEXT: }
125|   // CHECK-NEXT: "op_01e"([[RET]])
126| 
127|   // CHECK: partition2
128|   // CHECK-NEXT: op_02b
129|   // CHECK-NEXT: [[RET:%.*]] = scf.for [[I:%.*]] = [[LB:%.*]] to [[UB:%.*]] step [[STEP:%.*]] iter_args([[ARG2:%.*]] = {{.*}}) -> (i32) : i32 {
130|   // CHECK-NEXT:   [[Y:%.*]] = arith.addi [[I]], [[I]]
131|   // CHECK-NEXT:   [[Z:%.*]] = arith.addi [[I]], [[Y]]
132|   // CHECK-NEXT:   [[X:%.*]] = "op_a"([[Z]])
133|   // CHECK-NEXT:   "op_b"([[ARG2]])
134|   // CHECK-NEXT:   "op_b"([[X]])
135|   // CHECK-NEXT:   arith.addi
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 136-153
```mlir
136|   // CHECK-NEXT:   scf.yield
137|   // CHECK-NEXT: }
138|   // CHECK-NEXT: "op_02e"([[RET]])
139|   // CHECK: nvws.warp_group.return
140|   // CHECK-NEXT: }
141|   // CHECK-NEXT: "op_e"
142| 
143|   "op_00b"() {ttg.partition = array<i32: 0>, ttg.warp_specialize.tag = 0} : () -> ()
144|   "op_01b"() {ttg.partition = array<i32: 1>, ttg.warp_specialize.tag = 0} : () -> ()
145|   "op_b"() : () -> ()
146|   "op_02b"() {ttg.partition = array<i32: 2>, ttg.warp_specialize.tag = 0} : () -> ()
147|   %ret:3 = scf.for %i = %lb to %ub step %step iter_args(%arg0 = %c0, %arg1 = %c1, %arg2 = %c2) -> (i32, i32, i32) : i32 {
148|     %a = arith.addi %i, %i {ttg.partition = array<i32: 1, 2>} : i32
149|     %b = arith.addi %i, %a {ttg.partition = array<i32: 1, 2>} : i32
150| 
151|     %0 = "op_a"(%i) {ttg.partition = array<i32: 0>} : (i32) -> i32
152|     "op_b"(%arg0) {ttg.partition = array<i32: 0>} : (i32) -> ()
153|     "op_b"(%0) {ttg.partition = array<i32: 0>} : (i32) -> ()
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining integer additions, structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 整数加法、结构化循环。

### Lines 154-171
```mlir
154| 
155|     %1 = "op_a"(%a) {ttg.partition = array<i32: 1>} : (i32) -> i32
156|     "op_b"(%arg1) {ttg.partition = array<i32: 1>} : (i32) -> ()
157|     "op_b"(%1) {ttg.partition = array<i32: 1>} : (i32) -> ()
158| 
159|     %2 = "op_a"(%b) {ttg.partition = array<i32: 2>} : (i32) -> i32
160|     "op_b"(%arg2) {ttg.partition = array<i32: 2>} : (i32) -> ()
161|     "op_b"(%2) {ttg.partition = array<i32: 2>} : (i32) -> ()
162| 
163|     %v0 = arith.addi %arg0, %arg0 {ttg.partition = array<i32: 0>} : i32
164|     %v1 = arith.addi %arg1, %arg1 {ttg.partition = array<i32: 0, 1>} : i32
165|     %v2 = arith.addi %arg2, %arg2 {ttg.partition = array<i32: 0, 2>}: i32
166|     scf.yield {ttg.partition = array<i32: 0, 1, 2>} %v0, %v1, %v2: i32, i32, i32
167|   } {ttg.partition.stages = [0, 0, 0], ttg.warp_specialize.tag = 0 : i32, ttg.partition = array<i32: 0, 1, 2>, ttg.partition.outputs = [array<i32: 0>, array<i32: 0, 1>, array<i32: 0, 2>]}
168|   "op_00e"(%ret#0) {ttg.partition = array<i32: 0>, ttg.warp_specialize.tag = 0} : (i32) -> ()
169|   "op_01e"(%ret#1) {ttg.partition = array<i32: 1>, ttg.warp_specialize.tag = 0} : (i32) -> ()
170|   "op_e"() : () -> ()
171|   "op_02e"(%ret#2) {ttg.partition = array<i32: 2>, ttg.warp_specialize.tag = 0} : (i32) -> ()
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining integer additions, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 整数加法、循环/分支产出值。

### Lines 172-189
```mlir
172| 
173|   // CHECK: partition0 num_warps(4)
174|   // CHECK-NEXT: op_10b
175|   // CHECK-NEXT: scf.for
176|   // CHECK: } {ttg.warp_specialize.tag = 1
177|   // CHECK-NEXT: op_10e
178| 
179|   // CHECK: partition1
180|   // CHECK-NEXT: op_11b
181|   // CHECK-NEXT: scf.for
182|   // CHECK: } {ttg.warp_specialize.tag = 1
183|   // CHECK-NEXT: op_11e
184| 
185|   // CHECK: partition2
186|   // CHECK-NEXT: op_12b
187|   // CHECK-NEXT: scf.for
188|   // CHECK: } {ttg.warp_specialize.tag = 1
189|   // CHECK-NEXT: op_12e
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 190-207
```mlir
190|   "op_10b"() {ttg.partition = array<i32: 0>, ttg.warp_specialize.tag = 1} : () -> ()
191|   "op_11b"() {ttg.partition = array<i32: 1>, ttg.warp_specialize.tag = 1} : () -> ()
192|   "op_12b"() {ttg.partition = array<i32: 2>, ttg.warp_specialize.tag = 1} : () -> ()
193|   scf.for %i = %lb to %ub step %step : i32 {
194|     %a = arith.addi %i, %i {ttg.partition = array<i32: 1, 2>} : i32
195|     %b = arith.addi %i, %a {ttg.partition = array<i32: 1, 2>} : i32
196| 
197|     %0 = "op_a"(%i) {ttg.partition = array<i32: 0>} : (i32) -> i32
198|     "op_b"(%0) {ttg.partition = array<i32: 0>} : (i32) -> ()
199|     "op_b"(%0) {ttg.partition = array<i32: 0>} : (i32) -> ()
200| 
201|     %1 = "op_a"(%a) {ttg.partition = array<i32: 1>} : (i32) -> i32
202|     "op_b"(%1) {ttg.partition = array<i32: 1>} : (i32) -> ()
203|     "op_b"(%1) {ttg.partition = array<i32: 1>} : (i32) -> ()
204| 
205|     %2 = "op_a"(%b) {ttg.partition = array<i32: 2>} : (i32) -> i32
206|     "op_b"(%2) {ttg.partition = array<i32: 2>} : (i32) -> ()
207|     "op_b"(%2) {ttg.partition = array<i32: 2>} : (i32) -> ()
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining integer additions, structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 整数加法、结构化循环。

### Lines 208-214
```mlir
208|   } {ttg.partition.stages = [0, 0, 0], ttg.warp_specialize.tag = 1 : i32, ttg.partition = array<i32: 0, 1, 2>}
209|   "op_10e"() {ttg.partition = array<i32: 0>, ttg.warp_specialize.tag = 1} : () -> ()
210|   "op_11e"() {ttg.partition = array<i32: 1>, ttg.warp_specialize.tag = 1} : () -> ()
211|   "op_12e"() {ttg.partition = array<i32: 2>, ttg.warp_specialize.tag = 1} : () -> ()
212|   tt.return
213| }
214| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 215-215
```mlir
215| // CHECK-LABEL: @split_block_arguments
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @split_block_arguments anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @split_block_arguments 这样的标签用于锚定匹配范围。

### Lines 216-216
```mlir
216| tt.func @split_block_arguments(%lb: i32, %ub: i32, %step: i32) {
```
**EN:** This function-oriented block defines or enters `split_block_arguments`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `split_block_arguments` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 217-220
```mlir
217|   // CHECK-NEXT: [[C0:%.*]] = arith.constant 0
218|   // CHECK-NEXT: [[C1:%.*]] = arith.constant 1
219|   %c0_i32 = arith.constant 0 : i32
220|   %c1_i32 = arith.constant 1 : i32
```
**EN:** This operational block carries the concrete IR steps for the test, centered on constants. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 常量。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 221-237
```mlir
221|   // CHECK:      partition0
222|   // CHECK-NEXT:   scf.for {{.*}} iter_args([[A:%.*]] = [[C0]])
223|   // CHECK-NEXT:     [[X:%.*]] = "op_a"([[A]])
224|   // CHECK-NEXT:     yield [[X]] : i32
225| 
226|   // CHECK:      partition1
227|   // CHECK-NEXT:   scf.for {{.*}} iter_args([[B:%.*]] = [[C1]])
228|   // CHECK-NEXT:     [[X:%.*]] = "op_b"([[B]])
229|   // CHECK-NEXT:     yield [[X]] : i32
230|   scf.for %i = %lb to %ub step %step iter_args(%a = %c0_i32, %b = %c1_i32) -> (i32, i32) : i32 {
231|     %0 = "op_a"(%a) {ttg.partition = array<i32: 0>} : (i32) -> i32
232|     %1 = "op_b"(%b) {ttg.partition = array<i32: 1>} : (i32) -> i32
233|     scf.yield {ttg.partition = array<i32: 0, 1>} %0, %1 : i32, i32
234|   } {ttg.partition.stages = [0, 0], ttg.warp_specialize.tag = 0 : i32, ttg.partition = array<i32: 0, 1>, ttg.partition.outputs = [array<i32: 0>, array<i32: 1>]}
235|   tt.return
236| }
237| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops, loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环、循环/分支产出值、tt.return。

### Lines 238-238
```mlir
238| // CHECK-LABEL: @partition_outputs
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @partition_outputs anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @partition_outputs 这样的标签用于锚定匹配范围。

### Lines 239-239
```mlir
239| tt.func @partition_outputs(%lb: i32, %ub: i32, %step: i32) -> (!ty, !ty, !ty) {
```
**EN:** This function-oriented block defines or enters `partition_outputs`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `partition_outputs` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 240-246
```mlir
240|   // CHECK-NEXT: [[CST0:%.*]] = arith.constant dense<0>
241|   // CHECK-NEXT: [[CST1:%.*]] = arith.constant dense<1>
242|   // CHECK-NEXT: [[CST2:%.*]] = arith.constant dense<2>
243|   %cst0 = arith.constant dense<0> : !ty
244|   %cst1 = arith.constant dense<1> : !ty
245|   %cst2 = arith.constant dense<2> : !ty
246| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on constants. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 常量。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 247-264
```mlir
247|   // CHECK-NEXT: [[B_BUF:%.*]] = ttg.local_alloc
248|   // CHECK-NEXT: [[C_BUF:%.*]] = ttg.local_alloc
249|   // CHECK-NEXT: [[A_OUT:%.*]] = nvws.warp_group
250| 
251|   // CHECK-NEXT: partition0
252|   // CHECK-NEXT: [[OUT:%.*]] = scf.for [[I:%arg[0-9]+]] {{.*}} iter_args([[A:%.*]] = [[CST0]])
253|   // CHECK-NEXT:   [[X:%.*]] = "op_a"([[I]], [[A]])
254|   // CHECK-NEXT:   yield [[X]]
255|   // CHECK-NEXT: }
256|   // CHECK-NEXT: nvws.warp_group.yield [[OUT]]
257| 
258|   // CHECK:      partition1 num_warps(4)
259|   // CHECK-NEXT: [[OUT:%.*]] = scf.for [[I:%arg[0-9]+]] {{.*}} iter_args([[B:%.*]] = [[CST1]])
260|   // CHECK-NEXT:   [[X:%.*]] = "op_b"([[I]], [[B]])
261|   // CHECK-NEXT:   yield [[X]]
262|   // CHECK-NEXT: }
263|   // CHECK-NEXT: local_store [[OUT]], [[B_BUF]]
264| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 265-278
```mlir
265|   // CHECK:      partition2 num_warps(4)
266|   // CHECK-NEXT: [[OUT:%.*]] = scf.for [[I:%arg[0-9]+]] {{.*}} iter_args([[C:%.*]] = [[CST2]])
267|   // CHECK-NEXT:   [[X:%.*]] = "op_c"([[I]], [[C]])
268|   // CHECK-NEXT:   yield [[X]]
269|   // CHECK-NEXT: }
270|   // CHECK-NEXT: local_store [[OUT]], [[C_BUF]]
271| 
272|   %outs:3 = scf.for %i = %lb to %ub step %step iter_args(%a = %cst0, %b = %cst1, %c = %cst2) -> (!ty, !ty, !ty) : i32 {
273|     %0 = "op_a"(%i, %a) {ttg.partition = array<i32: 0>} : (i32, !ty) -> !ty
274|     %1 = "op_b"(%i, %b) {ttg.partition = array<i32: 1>} : (i32, !ty) -> !ty
275|     %2 = "op_c"(%i, %c) {ttg.partition = array<i32: 2>} : (i32, !ty) -> !ty
276|     scf.yield {ttg.partition = array<i32: 0, 1, 2>} %0, %1, %2 : !ty, !ty, !ty
277|   } {ttg.partition.stages = [0, 0, 0], ttg.warp_specialize.tag = 0 : i32, ttg.partition = array<i32: 0, 1, 2>, ttg.partition.outputs = [array<i32: 0>, array<i32: 1>, array<i32: 2>]}
278| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环、循环/分支产出值。

### Lines 279-287
```mlir
279|   // CHECK: [[B_OUT:%.*]] = ttg.local_load [[B_BUF]]
280|   // CHECK-NEXT: local_dealloc [[B_BUF]]
281|   // CHECK-NEXT: [[C_OUT:%.*]] = ttg.local_load [[C_BUF]]
282|   // CHECK-NEXT: local_dealloc [[C_BUF]]
283| 
284|   // CHECK-NEXT: tt.return [[A_OUT]], [[B_OUT]], [[C_OUT]]
285|   tt.return %outs#0, %outs#1, %outs#2 : !ty, !ty, !ty
286| }
287| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 288-288
```mlir
288| // CHECK-LABEL: @trivial_tensor_captures
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @trivial_tensor_captures anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @trivial_tensor_captures 这样的标签用于锚定匹配范围。

### Lines 289-291
```mlir
289| tt.func @trivial_tensor_captures(%arg0: f16, %lb: i32, %ub: i32, %step: i32) {
290|   %0 = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32>
291|   %1 = tt.splat %arg0 : f16 -> tensor<32xf16>
```
**EN:** This function-oriented block defines or enters `trivial_tensor_captures`. Within it, the test exercises tt.func, lane/block index ranges, broadcasted scalars or pointers, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `trivial_tensor_captures` 为核心。测试在其中演示 tt.func、lane/block 索引范围、广播后的标量或指针，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 292-295
```mlir
292|   // CHECK: [[RANGE:%.*]] = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32>
293|   // CHECK-NEXT: [[SPLAT:%.*]] = tt.splat %arg0 : f16 -> tensor<32xf16>
294|   // CHECK-NEXT: nvws.warp_group
295|   scf.for %i = %lb to %ub step %step : i32 {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环。

### Lines 296-303
```mlir
296|     // CHECK: partition1 num_warps(4)
297|     // CHECK-NEXT: scf.for
298|     // CHECK-NEXT: "use"([[RANGE]], [[SPLAT]])
299|     "use"(%0, %1) {ttg.partition = array<i32: 1>} : (tensor<256xi32>, tensor<32xf16>) -> ()
300|   } {ttg.partition.stages = [0, 0], ttg.warp_specialize.tag = 0 : i32, ttg.partition = array<i32: 0, 1>}
301|   tt.return
302| }
303| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 tt.return。

### Lines 304-304
```mlir
304| // CHECK-LABEL: @tensor_captures_over_smem
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tensor_captures_over_smem anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tensor_captures_over_smem 这样的标签用于锚定匹配范围。

### Lines 305-305
```mlir
305| tt.func @tensor_captures_over_smem(%lb: i32, %ub: i32, %step: i32) {
```
**EN:** This function-oriented block defines or enters `tensor_captures_over_smem`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tensor_captures_over_smem` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 306-307
```mlir
306|   // CHECK: [[VALUE:%.*]] = "value"()
307|   %0 = "value"() : () -> tensor<32xf16, #blocked>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 308-309
```mlir
308|   // CHECK: nvws.warp_group
309|   scf.for %i = %lb to %ub step %step : i32 {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环。

### Lines 310-317
```mlir
310|     // CHECK: partition1
311|     // CHECK-NEXT: scf.for
312|     // CHECK-NEXT: "use"([[VALUE]])
313|     "use"(%0) {ttg.partition = array<i32: 1>} : (tensor<32xf16, #blocked>) -> ()
314|   } {ttg.partition.stages = [0, 0], ttg.warp_specialize.tag = 0 : i32, ttg.partition = array<i32: 0, 1>}
315|   tt.return
316| }
317| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 tt.return。

### Lines 318-318
```mlir
318| // CHECK-LABEL: @dce_before_warp_allocation
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @dce_before_warp_allocation anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @dce_before_warp_allocation 这样的标签用于锚定匹配范围。

### Lines 319-320
```mlir
319| tt.func @dce_before_warp_allocation(%lb: i32, %ub: i32, %step: i32) {
320|   %cst = arith.constant dense<0> : tensor<128xi32, #blocked>
```
**EN:** This function-oriented block defines or enters `dce_before_warp_allocation`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `dce_before_warp_allocation` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 321-338
```mlir
321|   // CHECK: nvws.warp_group
322|   // CHECK: partition1 num_warps(4)
323|   // CHECK: partition2 num_warps(4)
324|   scf.for %i = %lb to %ub step %step iter_args(%idxs = %cst) -> tensor<128xi32, #blocked> : i32 {
325|     %do_prologue = "prologue_cond"(%i) {ttg.partition = array<i32: 0, 1, 2>} : (i32) -> i1
326|     %0 = scf.if %do_prologue -> tensor<128xi32, #blocked> {
327|       %1 = tt.splat %i {ttg.partition = array<i32: 0, 1, 2>} : i32 -> tensor<128xi32, #blocked>
328|       %2 = arith.addi %1, %idxs {ttg.partition = array<i32: 0, 1, 2>} : tensor<128xi32, #blocked>
329|       scf.yield {ttg.partition = array<i32: 0, 1, 2>} %2 : tensor<128xi32, #blocked>
330|     } else {
331|       scf.yield {ttg.partition = array<i32: 0, 1, 2>} %idxs : tensor<128xi32, #blocked>
332|     } {ttg.partition = array<i32: 0, 1, 2>, ttg.partition.outputs = [array<i32: 0, 1, 2>]}
333|     "op_a"(%0) {ttg.partition = array<i32: 0>} : (tensor<128xi32, #blocked>) -> ()
334|     "op_b"(%i) {ttg.partition = array<i32: 1>} : (i32) -> ()
335|     "op_c"(%0) {ttg.partition = array<i32: 2>} : (tensor<128xi32, #blocked>) -> ()
336|     scf.yield {ttg.partition = array<i32: 0, 1, 2>} %0 : tensor<128xi32, #blocked>
337|   } {ttg.partition.stages = [0, 0, 0], ttg.warp_specialize.tag = 0 : i32, ttg.partition = array<i32: 0, 1, 2>, ttg.partition.outputs = [array<i32: 0, 1, 2>]}
338|   tt.return
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, structured loops, structured conditionals, broadcasted scalars or pointers, integer additions.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、结构化循环、结构化条件分支、广播后的标量或指针、整数加法。

### Lines 339-340
```mlir
339| }
340| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 341-341
```mlir
341| // CHECK-LABEL: @capture_order
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @capture_order anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @capture_order 这样的标签用于锚定匹配范围。

### Lines 342-346
```mlir
342| tt.func @capture_order(%arg0: i32) {
343|   %c0_i32 = arith.constant 0 : i32
344|   %c1_i32 = arith.constant 1 : i32
345|   %0 = tt.make_range {end = 4 : i32, start = 0 : i32} : tensor<4xi32, #blocked>
346|   %1 = arith.extsi %0 : tensor<4xi32, #blocked> to tensor<4xi64, #blocked>
```
**EN:** This function-oriented block defines or enters `capture_order`. Within it, the test exercises tt.func, constants, lane/block index ranges, arith.extsi, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `capture_order` 为核心。测试在其中演示 tt.func、常量、lane/block 索引范围、arith.extsi，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 347-352
```mlir
347|   // CHECK: [[VALUE:%.*]] = tt.make_range
348|   // CHECK-NEXT: [[EXT:%.*]] = arith.extsi [[VALUE]]
349|   // CHECK: nvws.warp_group
350|   // CHECK: partition1
351|   // CHECK-NEXT: scf.for
352|   scf.for %arg1 = %c0_i32 to %arg0 step %c1_i32  : i32 {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环。

### Lines 353-354
```mlir
353|     // CHECK-NEXT: "use"([[VALUE]])
354|     "use"(%0) {ttg.partition = array<i32: 0, 1>} : (tensor<4xi32, #blocked>) -> ()
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 355-360
```mlir
355|     // CHECK-NEXT: "use"([[EXT]])
356|     "use"(%1) {ttg.partition = array<i32: 0, 1>} : (tensor<4xi64, #blocked>) -> ()
357|   } {ttg.partition.stages = [1 : i32, 0 : i32], ttg.warp_specialize.tag = 0 : i32, ttg.partition = array<i32: 0, 1>}
358|   tt.return
359| }
360| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 361-361
```mlir
361| // CHECK-LABEL: @clone_then_capture
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @clone_then_capture anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @clone_then_capture 这样的标签用于锚定匹配范围。

### Lines 362-365
```mlir
362| tt.func @clone_then_capture(%arg0: i32) {
363|   %c0_i32 = arith.constant 0 : i32
364|   %c1_i32 = arith.constant 1 : i32
365| 
```
**EN:** This function-oriented block defines or enters `clone_then_capture`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `clone_then_capture` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 366-369
```mlir
366|   // CHECK: [[TT:%.*]] = "tensor_op"()
367|   // CHECK: [[V:%.*]] = arith.addi [[TT]], [[TT]]
368|   %0 = "tensor_op"() : () -> tensor<4xi32, #blocked>
369|   %1 = arith.addi %0, %0 : tensor<4xi32, #blocked>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on integer additions. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 整数加法。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 370-372
```mlir
370|   // CHECK: partition1
371|   // CHECK: scf.for
372|   scf.for %arg1 = %c0_i32 to %arg0 step %c1_i32  : i32 {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环。

### Lines 373-378
```mlir
373|     // CHECK: "use"([[V]])
374|     "use"(%1) {ttg.partition = array<i32: 1>} : (tensor<4xi32, #blocked>) -> ()
375|   } {ttg.partition.stages = [0 : i32, 1 : i32], ttg.warp_specialize.tag = 0 : i32, ttg.partition = array<i32: 0, 1>}
376|   tt.return
377| }
378| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 379-379
```mlir
379| // CHECK-LABEL: @if_stmt_split
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @if_stmt_split anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @if_stmt_split 这样的标签用于锚定匹配范围。

### Lines 380-382
```mlir
380| tt.func @if_stmt_split(%arg1: !ty, %ub: i32, %lb: i32, %step: i32) {
381|   %out:2 = scf.for %i = %lb to %ub step %step iter_args(%a = %arg1, %b = %arg1) -> (!ty, !ty) : i32 {
382|     %cond = "cond"(%i) {ttg.partition = array<i32: 0, 1>} : (i32) -> i1
```
**EN:** This function-oriented block defines or enters `if_stmt_split`. Within it, the test exercises tt.func, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `if_stmt_split` 为核心。测试在其中演示 tt.func、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 383-400
```mlir
383|     // CHECK: nvws.warp_group
384|     // CHECK-NEXT: partition0
385|     // CHECK-NEXT: scf.for
386|     // CHECK-NEXT: "cond"
387|     // CHECK-NEXT: [[C:%.*]] = scf.if
388|     // CHECK-NEXT: [[A:%.*]] = "use1"
389|     // CHECK-NEXT: scf.yield [[A]]
390|     // CHECK-NEXT: } else {
391|     // CHECK-NEXT: [[B:%.*]] = "use3"
392|     // CHECK-NEXT: scf.yield [[B]]
393|     // CHECK-NEXT: }
394|     // CHECK-NEXT: scf.yield [[C]]
395| 
396|     // CHECK: partition1
397|     // CHECK-NEXT: scf.for
398|     // CHECK-NEXT: "cond"
399|     // CHECK-NEXT: [[C:%.*]] = scf.if
400|     // CHECK-NEXT: [[A:%.*]] = "use2"
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 401-418
```mlir
401|     // CHECK-NEXT: scf.yield [[A]]
402|     // CHECK-NEXT: } else {
403|     // CHECK-NEXT: [[B:%.*]] = "use4"
404|     // CHECK-NEXT: scf.yield [[B]]
405|     // CHECK-NEXT: }
406|     // CHECK-NEXT: scf.yield [[C]]
407|     %ret:2 = scf.if %cond -> (!ty, !ty) {
408|       %1 = "use1"(%a) {ttg.partition = array<i32: 0>} : (!ty) -> !ty
409|       %2 = "use2"(%b) {ttg.partition = array<i32: 1>} : (!ty) -> !ty
410|       scf.yield {ttg.partition = array<i32: 0, 1>} %1, %2 : !ty, !ty
411|     }  else {
412|        %3 = "use3"(%a) {ttg.partition = array<i32: 0>} : (!ty) -> !ty
413|        %4 = "use4"(%b) {ttg.partition = array<i32: 1>} : (!ty) -> !ty
414|        scf.yield {ttg.partition = array<i32: 0, 1>} %3, %4 : !ty, !ty
415|     } {ttg.partition = array<i32: 0, 1>, ttg.partition.outputs = [array<i32: 0>, array<i32: 1>]}
416|     scf.yield {ttg.partition = array<i32: 0, 1>} %ret#0, %ret#1 : !ty, !ty
417|   } {ttg.partition.stages = [0, 0], ttg.warp_specialize.tag = 0 : i32, ttg.partition = array<i32: 0, 1>, ttg.partition.outputs = [array<i32: 0>, array<i32: 1>]}
418|   tt.return
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, structured conditionals, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、结构化条件分支、tt.return。

### Lines 419-422
```mlir
419| }
420| 
421| }
422| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 423-423
```mlir
423| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 424-427
```mlir
424| 
425| #blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
426| !ty = tensor<1xi32, #blocked>
427| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 428-429
```mlir
428| module attributes {"ttg.num-warps" = 4 : i32} {
429| 
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 430-431
```mlir
430| tt.func @still_has_ssa_deps(%lb: i32, %ub: i32, %step: i32) {
431|   scf.for %i = %lb to %ub step %step : i32 {
```
**EN:** This function-oriented block defines or enters `still_has_ssa_deps`. Within it, the test exercises tt.func, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `still_has_ssa_deps` 为核心。测试在其中演示 tt.func、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 432-433
```mlir
432|     // expected-warning @below {{non-root partition #0 has direct SSA consumer}}
433|     %0 = "op_a"() {ttg.partition = array<i32: 0>} : () -> !ty
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 434-440
```mlir
434|     // expected-note @below {{use at distance 0 in partition #1 here}}
435|     "op_b"(%0) {ttg.partition = array<i32: 1>} : (!ty) -> ()
436|   } {ttg.partition.stages = [0, 1], ttg.warp_specialize.tag = 0 : i32, ttg.partition = array<i32: 0, 1>}
437|   tt.return
438| }
439| 
440| }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `-allow-unregistered-dialect`, `-tritongpu-partition-loops`, `-verify-diagnostics`, `-canonicalize`
- **CN:** 主要 pass 选项：`-split-input-file`，`-allow-unregistered-dialect`，`-tritongpu-partition-loops`，`-verify-diagnostics`，`-canonicalize`
- **EN:** Dominant operations include `tt.func`, `scf.for`, `tt.return`, `arith.constant`, `arith.addi`, `scf.yield`, `module`, `tt.make_range`, `tt.splat`, `scf.if`.
- **CN:** 主要操作包括 `tt.func`、`scf.for`、`tt.return`、`arith.constant`、`arith.addi`、`scf.yield`、`module`、`tt.make_range`、`tt.splat`、`scf.if`。
- **EN:** The file contains 1 independently testable section(s). Check styles used: CHECK-NEXT x145, CHECK x37, CHECK-LABEL x13. Important labels include @one_partition, @two_empty_partitions, @empty_partition_fwd_root, @multiple_partitions. Expected diagnostic comments specify the exact verifier/pass failures. It validates that the selected pass pipeline rewrites each labeled test case into the checked form.
- **CN:** 该文件包含 1 个可独立测试的分段。使用的检查类型：CHECK-NEXT ×145，CHECK ×37，CHECK-LABEL ×13。 关键标签包括 @one_partition，@two_empty_partitions，@empty_partition_fwd_root，@multiple_partitions。 expected 诊断注释给出了 verifier/pass 需要触发的精确报错。 它验证所选 pass 流水线会把每个带标签的测试用例重写成检查所要求的形式。

## Dependencies / 依赖关系

- **EN:** This test depends on Triton/MLIR textual IR parsing, the pass pipeline named in the RUN lines, and the dialect/type definitions referenced in the file.
- **CN:** 该测试依赖 Triton/MLIR 文本 IR 解析、RUN 行中指定的 pass 流水线，以及文件中引用的方言/类型定义。
- **EN:** `tt`: Triton high-level tensor/pointer operations.
- **CN:** `tt`：Triton 高层张量/指针操作。
- **EN:** `scf`: Structured control flow for loops and conditionals.
- **CN:** `scf`：用于循环与条件分支的结构化控制流。
- **EN:** `arith`: MLIR arithmetic dialect for scalar/tensor math.
- **CN:** `arith`：用于标量/张量运算的 MLIR arithmetic 方言。
- **EN:** `ttg`: TritonGPU layout, memory, and GPU-specific operations.
- **CN:** `ttg`：TritonGPU 布局、内存和 GPU 专用操作。
- **EN:** `FileCheck` is used to match required textual patterns after rewriting/lowering.
- **CN:** `FileCheck` 用于在重写/lower 之后匹配必须出现的文本模式。
- **EN:** Diagnostic verification is part of the contract, so parser/verifier error strings are also dependencies of the test.
- **CN:** 诊断验证也是该测试契约的一部分，因此 parser/verifier 的报错文本同样是依赖项。