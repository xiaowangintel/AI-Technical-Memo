# optimize-locality.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/optimize-locality.mlir`
- **EN:** Test for the MLIR/Triton pipeline `-tritongpu-optimize-thread-locality, -canonicalize` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `-tritongpu-optimize-thread-locality, -canonicalize` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -split-input-file -tritongpu-optimize-thread-locality -canonicalize | FileCheck %s`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -split-input-file -tritongpu-optimize-thread-locality -canonicalize | FileCheck %s`

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
1| // RUN: triton-opt %s -split-input-file -tritongpu-optimize-thread-locality -canonicalize | FileCheck %s
2| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -split-input-file -tritongpu-optimize-thread-locality -canonicalize | FileCheck %s` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -split-input-file -tritongpu-optimize-thread-locality -canonicalize | FileCheck %s`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 3-14
```mlir
 3| // CHECK-LABEL: negative_zero_accumulator
 4| // CHECK: %[[INIT_ARG:.*]] = arith.constant dense<0.000000e+00>
 5| // CHECK: %[[LOOP_OUTPUT:.*]] = scf.for {{.*}} iter_args(%[[FOR_ARG:.*]] = %[[INIT_ARG]]) -> {{.*}}
 6| // CHECK: %[[LOAD:.*]] = tt.load
 7| // CHECK: tt.reshape %[[LOAD]] allow_reorder efficient_layout : {{.*}} -> tensor<{{32x32x4xf32.*}}
 8| // CHECK-NEXT: %[[REDUCE:.*]] = "tt.reduce"({{%.*}}) <{axis = 2 : i32}>
 9| // CHECK: arith.addf
10| // CHECK: arith.addf %[[FOR_ARG]], %[[REDUCE]]
11| // CHECK-NEXT: scf.yield
12| // CHECK: %[[FINAL_REDUCE:.*]] = "tt.reduce"(%[[LOOP_OUTPUT]]) <{axis = 1 : i32}>
13| // CHECK: %[[CVT_OUTPUT:.*]] = ttg.convert_layout %[[FINAL_REDUCE]]
14| // CHECK: tt.store {{%.*}}, %[[CVT_OUTPUT]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: negative_zero_accumulator anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: negative_zero_accumulator 这样的标签用于锚定匹配范围。

### Lines 15-16
```mlir
15| #blocked = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
16| #blocked1 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 17-17
```mlir
17| module attributes {"ttg.target" = "cuda:80", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 18-35
```mlir
18|   tt.func public @negative_zero_accumulator(
19|     %arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32},
20|     %arg1: !tt.ptr<f32> {tt.divisibility = 16 : i32},
21|     %arg2: i32 {tt.divisibility = 16 : i32, tt.max_divisibility = 8 : i32},
22|     %18: tensor<32x128x!tt.ptr<f32>, #blocked> {tt.divisibility = 16 : i32},
23|     %11: i32 {tt.divisibility = 16 : i32},
24|     %25: tensor<32x!tt.ptr<f32>, #blocked1> {tt.divisibility = 16 : i32}
25|     ) {
26|     %cst = arith.constant dense<-0.000000e+00> : tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
27|     %c128_i32 = arith.constant 128 : i32
28|     %1 = tt.get_program_id y : i32
29|     %2 = tt.get_num_programs y : i32
30|     %12 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
31|     %19 = scf.for %arg3 = %1 to %11 step %2 iter_args(%arg4 = %cst) -> (tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>>)  : i32 {
32|       %27 = arith.muli %arg3, %c128_i32 : i32
33|       %28 = tt.splat %27 : i32 -> tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
34|       %29 = arith.addi %28, %12 : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
35|       %30 = tt.expand_dims %29 {axis = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x128xi32, #blocked>
```
**EN:** This function-oriented block defines or enters `negative_zero_accumulator`. Within it, the test exercises tt.func, constants, program IDs, tt.get_num_programs, lane/block index ranges, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `negative_zero_accumulator` 为核心。测试在其中演示 tt.func、常量、程序 ID、tt.get_num_programs、lane/block 索引范围，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 36-52
```mlir
36|       %31 = tt.broadcast %30 : tensor<1x128xi32, #blocked> -> tensor<32x128xi32, #blocked>
37|       %32 = tt.addptr %18, %31 : tensor<32x128x!tt.ptr<f32>, #blocked>, tensor<32x128xi32, #blocked>
38|       %33 = tt.load %32 : tensor<32x128x!tt.ptr<f32>, #blocked>
39|       %34 = "tt.reduce"(%33) <{axis = 1 : i32}> ({
40|       ^bb0(%arg5: f32, %arg6: f32):
41|         %36 = arith.addf %arg5, %arg6 : f32
42|         tt.reduce.return %36 : f32
43|       }) : (tensor<32x128xf32, #blocked>) -> tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
44|       %35 = arith.addf %arg4, %34 : tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
45|       scf.yield %35 : tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
46|     }
47|     %26 = ttg.convert_layout %19 : tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<32xf32, #blocked1>
48|     tt.store %25, %26 : tensor<32x!tt.ptr<f32>, #blocked1>
49|     tt.return
50|   }
51| }
52| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining floating-point additions, tensor broadcasting, pointer arithmetic, masked or vectorized loads, reductions.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 浮点加法、张量广播、指针算术、带掩码或向量化的加载、归约。

### Lines 53-53
```mlir
53| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 54-67
```mlir
54| 
55| // CHECK-LABEL: positive_zero_accumulator
56| // CHECK: %[[CST:.*]] = arith.constant dense<0.000000e+00>
57| // CHECK-NEXT: %[[CST1:.*]] = arith.constant dense<0.000000e+00>
58| // CHECK: %[[LOOP_OUTPUT:.*]] = scf.for {{.*}} iter_args(%[[FOR_ARG:.*]] = %[[CST1]]) -> {{.*}}
59| // CHECK: tt.load
60| // CHECK: tt.reshape
61| // CHECK-NEXT: %[[REDUCE:.*]] = "tt.reduce"({{%.*}}) <{axis = 2 : i32}>
62| // CHECK: arith.addf
63| // CHECK: arith.addf %[[FOR_ARG]], %[[REDUCE]]
64| // CHECK-NEXT: scf.yield
65| // CHECK: %[[FINAL_REDUCE:.*]] = "tt.reduce"(%[[LOOP_OUTPUT]]) <{axis = 1 : i32}>
66| // CHECK: %[[CVT_OUTPUT:.*]] = ttg.convert_layout %[[FINAL_REDUCE]]
67| // CHECK: arith.addf %[[CVT_OUTPUT]], %[[CST]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: positive_zero_accumulator anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: positive_zero_accumulator 这样的标签用于锚定匹配范围。

### Lines 68-69
```mlir
68| #blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
69| #blocked1 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 70-70
```mlir
70| module attributes {"ttg.target" = "cuda:80", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 71-88
```mlir
71|   tt.func public @positive_zero_accumulator(
72|     %arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32},
73|     %arg1: !tt.ptr<f32> {tt.divisibility = 16 : i32},
74|     %arg2: i32 {tt.divisibility = 16 : i32, tt.max_divisibility = 8 : i32},
75|     %18: tensor<32x128x!tt.ptr<f32>, #blocked> {tt.divisibility = 16 : i32},
76|     %11: i32 {tt.divisibility = 16 : i32},
77|     %25: tensor<32x!tt.ptr<f32>, #blocked1> {tt.divisibility = 16 : i32}
78|     ) {
79|     %cst = arith.constant dense<0.000000e+00> : tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
80|     %c128_i32 = arith.constant 128 : i32
81|     %1 = tt.get_program_id y : i32
82|     %2 = tt.get_num_programs y : i32
83|     %12 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
84|     %19 = scf.for %arg3 = %1 to %11 step %2 iter_args(%arg4 = %cst) -> (tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>>)  : i32 {
85|       %27 = arith.muli %arg3, %c128_i32 : i32
86|       %28 = tt.splat %27 : i32 -> tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
87|       %29 = arith.addi %28, %12 : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
88|       %30 = tt.expand_dims %29 {axis = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x128xi32, #blocked>
```
**EN:** This function-oriented block defines or enters `positive_zero_accumulator`. Within it, the test exercises tt.func, constants, program IDs, tt.get_num_programs, lane/block index ranges, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `positive_zero_accumulator` 为核心。测试在其中演示 tt.func、常量、程序 ID、tt.get_num_programs、lane/block 索引范围，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 89-105
```mlir
 89|       %31 = tt.broadcast %30 : tensor<1x128xi32, #blocked> -> tensor<32x128xi32, #blocked>
 90|       %32 = tt.addptr %18, %31 : tensor<32x128x!tt.ptr<f32>, #blocked>, tensor<32x128xi32, #blocked>
 91|       %33 = tt.load %32 : tensor<32x128x!tt.ptr<f32>, #blocked>
 92|       %34 = "tt.reduce"(%33) <{axis = 1 : i32}> ({
 93|       ^bb0(%arg5: f32, %arg6: f32):
 94|         %36 = arith.addf %arg5, %arg6 : f32
 95|         tt.reduce.return %36 : f32
 96|       }) : (tensor<32x128xf32, #blocked>) -> tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
 97|       %35 = arith.addf %arg4, %34 : tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
 98|       scf.yield %35 : tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
 99|     }
100|     %26 = ttg.convert_layout %19 : tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<32xf32, #blocked1>
101|     tt.store %25, %26 : tensor<32x!tt.ptr<f32>, #blocked1>
102|     tt.return
103|   }
104| }
105| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining floating-point additions, tensor broadcasting, pointer arithmetic, masked or vectorized loads, reductions.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 浮点加法、张量广播、指针算术、带掩码或向量化的加载、归约。

### Lines 106-106
```mlir
106| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 107-115
```mlir
107| 
108| // CHECK-LABEL: slice_layout
109| // CHECK: %[[LOOP_OUTPUT:.*]] = scf.for
110| // CHECK: %[[LOAD:.*]] = tt.load
111| // CHECK-NEXT: "tt.reduce"(%[[LOAD]]) <{axis = 1 : i32}>
112| // CHECK: arith.addf
113| // CHECK: arith.addf
114| // CHECK-NEXT: scf.yield
115| // CHECK: %[[CVT_OUTPUT:.*]] = ttg.convert_layout %[[LOOP_OUTPUT]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: slice_layout anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: slice_layout 这样的标签用于锚定匹配范围。

### Lines 116-118
```mlir
116| #blocked3d = #ttg.blocked<{sizePerThread = [1, 4, 1], threadsPerWarp = [1, 32, 1], warpsPerCTA = [4, 1, 1], order = [2, 1, 0]}>
117| #slice2d = #ttg.slice<{dim = 2, parent = #blocked3d}>
118| #blocked1 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 119-119
```mlir
119| module attributes {"ttg.target" = "cuda:80", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 120-137
```mlir
120|   tt.func public @slice_layout(
121|     %arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32},
122|     %arg1: !tt.ptr<f32> {tt.divisibility = 16 : i32},
123|     %arg2: i32 {tt.divisibility = 16 : i32, tt.max_divisibility = 8 : i32},
124|     %18: tensor<32x128x!tt.ptr<f32>, #slice2d> {tt.divisibility = 16 : i32},
125|     %11: i32 {tt.divisibility = 16 : i32},
126|     %25: tensor<32x!tt.ptr<f32>, #blocked1> {tt.divisibility = 16 : i32}
127|     ) {
128|     %cst = arith.constant dense<0.000000e+00> : tensor<32xf32, #ttg.slice<{dim = 1, parent = #slice2d}>>
129|     %c128_i32 = arith.constant 128 : i32
130|     %1 = tt.get_program_id y : i32
131|     %2 = tt.get_num_programs y : i32
132|     %12 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 0, parent = #slice2d}>>
133|     %19 = scf.for %arg3 = %1 to %11 step %2 iter_args(%arg4 = %cst) -> (tensor<32xf32, #ttg.slice<{dim = 1, parent = #slice2d}>>)  : i32 {
134|       %27 = arith.muli %arg3, %c128_i32 : i32
135|       %28 = tt.splat %27 : i32 -> tensor<128xi32, #ttg.slice<{dim = 0, parent = #slice2d}>>
136|       %29 = arith.addi %28, %12 : tensor<128xi32, #ttg.slice<{dim = 0, parent = #slice2d}>>
137|       %30 = tt.expand_dims %29 {axis = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 0, parent = #slice2d}>> -> tensor<1x128xi32, #slice2d>
```
**EN:** This function-oriented block defines or enters `slice_layout`. Within it, the test exercises tt.func, constants, program IDs, tt.get_num_programs, lane/block index ranges, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `slice_layout` 为核心。测试在其中演示 tt.func、常量、程序 ID、tt.get_num_programs、lane/block 索引范围，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 138-154
```mlir
138|       %31 = tt.broadcast %30 : tensor<1x128xi32, #slice2d> -> tensor<32x128xi32, #slice2d>
139|       %32 = tt.addptr %18, %31 : tensor<32x128x!tt.ptr<f32>, #slice2d>, tensor<32x128xi32, #slice2d>
140|       %33 = tt.load %32 : tensor<32x128x!tt.ptr<f32>, #slice2d>
141|       %34 = "tt.reduce"(%33) <{axis = 1 : i32}> ({
142|       ^bb0(%arg5: f32, %arg6: f32):
143|         %36 = arith.addf %arg5, %arg6 : f32
144|         tt.reduce.return %36 : f32
145|       }) : (tensor<32x128xf32, #slice2d>) -> tensor<32xf32, #ttg.slice<{dim = 1, parent = #slice2d}>>
146|       %35 = arith.addf %arg4, %34 : tensor<32xf32, #ttg.slice<{dim = 1, parent = #slice2d}>>
147|       scf.yield %35 : tensor<32xf32, #ttg.slice<{dim = 1, parent = #slice2d}>>
148|     }
149|     %26 = ttg.convert_layout %19 : tensor<32xf32, #ttg.slice<{dim = 1, parent = #slice2d}>> -> tensor<32xf32, #blocked1>
150|     tt.store %25, %26 : tensor<32x!tt.ptr<f32>, #blocked1>
151|     tt.return
152|   }
153| }
154| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining floating-point additions, tensor broadcasting, pointer arithmetic, masked or vectorized loads, reductions.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 浮点加法、张量广播、指针算术、带掩码或向量化的加载、归约。

### Lines 155-155
```mlir
155| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 156-164
```mlir
156| 
157| // CHECK-LABEL: mma_layout
158| // CHECK: %[[LOOP_OUTPUT:.*]] = scf.for
159| // CHECK: %[[LOAD:.*]] = tt.load
160| // CHECK-NEXT: "tt.reduce"(%[[LOAD]]) <{axis = 1 : i32}>
161| // CHECK: arith.addf
162| // CHECK: arith.addf
163| // CHECK-NEXT: scf.yield
164| // CHECK: %[[CVT_OUTPUT:.*]] = ttg.convert_layout %[[LOOP_OUTPUT]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: mma_layout anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: mma_layout 这样的标签用于锚定匹配范围。

### Lines 165-167
```mlir
165| #blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
166| #mma = #ttg.nvidia_mma<{versionMajor = 2, warpsPerCTA = [4, 1], instrShape = [16, 8]}>
167| #blocked1 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 168-168
```mlir
168| module attributes {"ttg.target" = "cuda:80", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 169-186
```mlir
169|   tt.func public @mma_layout(
170|     %arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32},
171|     %arg1: !tt.ptr<f32> {tt.divisibility = 16 : i32},
172|     %arg2: i32 {tt.divisibility = 16 : i32, tt.max_divisibility = 8 : i32},
173|     %18: tensor<32x128x!tt.ptr<f32>, #mma> {tt.divisibility = 16 : i32},
174|     %11: i32 {tt.divisibility = 16 : i32},
175|     %25: tensor<32x!tt.ptr<f32>, #blocked1> {tt.divisibility = 16 : i32}
176|     ) {
177|     %cst = arith.constant dense<0.000000e+00> : tensor<32xf32, #ttg.slice<{dim = 1, parent = #mma}>>
178|     %c128_i32 = arith.constant 128 : i32
179|     %1 = tt.get_program_id y : i32
180|     %2 = tt.get_num_programs y : i32
181|     %12 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 0, parent = #mma}>>
182|     %19 = scf.for %arg3 = %1 to %11 step %2 iter_args(%arg4 = %cst) -> (tensor<32xf32, #ttg.slice<{dim = 1, parent = #mma}>>)  : i32 {
183|       %27 = arith.muli %arg3, %c128_i32 : i32
184|       %28 = tt.splat %27 : i32 -> tensor<128xi32, #ttg.slice<{dim = 0, parent = #mma}>>
185|       %29 = arith.addi %28, %12 : tensor<128xi32, #ttg.slice<{dim = 0, parent = #mma}>>
186|       %30 = tt.expand_dims %29 {axis = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 0, parent = #mma}>> -> tensor<1x128xi32, #mma>
```
**EN:** This function-oriented block defines or enters `mma_layout`. Within it, the test exercises tt.func, constants, program IDs, tt.get_num_programs, lane/block index ranges, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `mma_layout` 为核心。测试在其中演示 tt.func、常量、程序 ID、tt.get_num_programs、lane/block 索引范围，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 187-203
```mlir
187|       %31 = tt.broadcast %30 : tensor<1x128xi32, #mma> -> tensor<32x128xi32, #mma>
188|       %32 = tt.addptr %18, %31 : tensor<32x128x!tt.ptr<f32>, #mma>, tensor<32x128xi32, #mma>
189|       %33 = tt.load %32 : tensor<32x128x!tt.ptr<f32>, #mma>
190|       %34 = "tt.reduce"(%33) <{axis = 1 : i32}> ({
191|       ^bb0(%arg5: f32, %arg6: f32):
192|         %36 = arith.addf %arg5, %arg6 : f32
193|         tt.reduce.return %36 : f32
194|       }) : (tensor<32x128xf32, #mma>) -> tensor<32xf32, #ttg.slice<{dim = 1, parent = #mma}>>
195|       %35 = arith.addf %arg4, %34 : tensor<32xf32, #ttg.slice<{dim = 1, parent = #mma}>>
196|       scf.yield %35 : tensor<32xf32, #ttg.slice<{dim = 1, parent = #mma}>>
197|     }
198|     %26 = ttg.convert_layout %19 : tensor<32xf32, #ttg.slice<{dim = 1, parent = #mma}>> -> tensor<32xf32, #blocked1>
199|     tt.store %25, %26 : tensor<32x!tt.ptr<f32>, #blocked1>
200|     tt.return
201|   }
202| }
203| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining floating-point additions, tensor broadcasting, pointer arithmetic, masked or vectorized loads, reductions.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 浮点加法、张量广播、指针算术、带掩码或向量化的加载、归约。

### Lines 204-204
```mlir
204| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 205-218
```mlir
205| 
206| // CHECK-LABEL: max_reduce
207| // CHECK: %[[INIT_ARG:.*]] = arith.constant dense<0xFF800000>
208| // CHECK: %[[LOOP_OUTPUT:.*]] = scf.for {{.*}} iter_args(%[[FOR_ARG:.*]] = %[[INIT_ARG]]) -> {{.*}}
209| // CHECK: %[[LOAD:.*]] = tt.load
210| // CHECK: tt.reshape %[[LOAD]] allow_reorder efficient_layout : {{.*}} -> tensor<{{32x32x4xf32.*}}
211| // CHECK-NEXT: %[[REDUCE:.*]] = "tt.reduce"({{%.*}}) <{axis = 2 : i32}>
212| // CHECK: arith.maximumf
213| // CHECK: arith.maximumf %[[FOR_ARG]], %[[REDUCE]]
214| // CHECK-NEXT: scf.yield
215| // CHECK: %[[FINAL_REDUCE:.*]] = "tt.reduce"(%[[LOOP_OUTPUT]]) <{axis = 1 : i32}>
216| // CHECK: arith.maximumf
217| // CHECK: %[[CVT_OUTPUT:.*]] = ttg.convert_layout %[[FINAL_REDUCE]]
218| // CHECK: tt.store {{%.*}}, %[[CVT_OUTPUT]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: max_reduce anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: max_reduce 这样的标签用于锚定匹配范围。

### Lines 219-220
```mlir
219| #blocked = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
220| #blocked1 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 221-221
```mlir
221| module attributes {"ttg.target" = "cuda:80", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 222-239
```mlir
222|   tt.func public @max_reduce(
223|     %arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32},
224|     %arg1: !tt.ptr<f32> {tt.divisibility = 16 : i32},
225|     %arg2: i32 {tt.divisibility = 16 : i32, tt.max_divisibility = 8 : i32},
226|     %18: tensor<32x128x!tt.ptr<f32>, #blocked> {tt.divisibility = 16 : i32},
227|     %11: i32 {tt.divisibility = 16 : i32},
228|     %25: tensor<32x!tt.ptr<f32>, #blocked1> {tt.divisibility = 16 : i32}
229|     ) {
230|     %cst = arith.constant dense<0xFF800000> : tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
231|     %c128_i32 = arith.constant 128 : i32
232|     %1 = tt.get_program_id y : i32
233|     %2 = tt.get_num_programs y : i32
234|     %12 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
235|     %19 = scf.for %arg3 = %1 to %11 step %2 iter_args(%arg4 = %cst) -> (tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>>)  : i32 {
236|       %27 = arith.muli %arg3, %c128_i32 : i32
237|       %28 = tt.splat %27 : i32 -> tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
238|       %29 = arith.addi %28, %12 : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
239|       %30 = tt.expand_dims %29 {axis = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x128xi32, #blocked>
```
**EN:** This function-oriented block defines or enters `max_reduce`. Within it, the test exercises tt.func, constants, program IDs, tt.get_num_programs, lane/block index ranges, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `max_reduce` 为核心。测试在其中演示 tt.func、常量、程序 ID、tt.get_num_programs、lane/block 索引范围，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 240-256
```mlir
240|       %31 = tt.broadcast %30 : tensor<1x128xi32, #blocked> -> tensor<32x128xi32, #blocked>
241|       %32 = tt.addptr %18, %31 : tensor<32x128x!tt.ptr<f32>, #blocked>, tensor<32x128xi32, #blocked>
242|       %33 = tt.load %32 : tensor<32x128x!tt.ptr<f32>, #blocked>
243|       %34 = "tt.reduce"(%33) <{axis = 1 : i32}> ({
244|       ^bb0(%arg5: f32, %arg6: f32):
245|         %36 = arith.maximumf %arg5, %arg6 : f32
246|         tt.reduce.return %36 : f32
247|       }) : (tensor<32x128xf32, #blocked>) -> tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
248|       %35 = arith.maximumf %arg4, %34 : tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
249|       scf.yield %35 : tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
250|     }
251|     %26 = ttg.convert_layout %19 : tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<32xf32, #blocked1>
252|     tt.store %25, %26 : tensor<32x!tt.ptr<f32>, #blocked1>
253|     tt.return
254|   }
255| }
256| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining arith.maximumf, tensor broadcasting, pointer arithmetic, masked or vectorized loads, reductions.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 arith.maximumf、张量广播、指针算术、带掩码或向量化的加载、归约。

### Lines 257-257
```mlir
257| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 258-272
```mlir
258| 
259| // CHECK-LABEL: max_reduce_zero_int_accumulator
260| // CHECK: %[[CST:.*]] = arith.constant dense<0.000000e+00>
261| // CHECK-NEXT: %[[CST1:.*]] = arith.constant dense<0xFF800000>
262| // CHECK: %[[LOOP_OUTPUT:.*]] = scf.for {{.*}} iter_args(%[[FOR_ARG:.*]] = %[[CST1]]) -> {{.*}}
263| // CHECK: tt.load
264| // CHECK: tt.reshape
265| // CHECK-NEXT: %[[REDUCE:.*]] = "tt.reduce"({{%.*}}) <{axis = 2 : i32}>
266| // CHECK: arith.maximumf
267| // CHECK: arith.maximumf %[[FOR_ARG]], %[[REDUCE]]
268| // CHECK-NEXT: scf.yield
269| // CHECK: %[[FINAL_REDUCE:.*]] = "tt.reduce"(%[[LOOP_OUTPUT]]) <{axis = 1 : i32}>
270| // CHECK: arith.maximumf
271| // CHECK: %[[CVT_OUTPUT:.*]] = ttg.convert_layout %[[FINAL_REDUCE]]
272| // CHECK: arith.maximumf %[[CVT_OUTPUT]], %[[CST]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: max_reduce_zero_int_accumulator anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: max_reduce_zero_int_accumulator 这样的标签用于锚定匹配范围。

### Lines 273-274
```mlir
273| #blocked = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
274| #blocked1 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 275-275
```mlir
275| module attributes {"ttg.target" = "cuda:80", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 276-293
```mlir
276|   tt.func public @max_reduce_zero_int_accumulator(
277|     %arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32},
278|     %arg1: !tt.ptr<f32> {tt.divisibility = 16 : i32},
279|     %arg2: i32 {tt.divisibility = 16 : i32, tt.max_divisibility = 8 : i32},
280|     %18: tensor<32x128x!tt.ptr<f32>, #blocked> {tt.divisibility = 16 : i32},
281|     %11: i32 {tt.divisibility = 16 : i32},
282|     %25: tensor<32x!tt.ptr<f32>, #blocked1> {tt.divisibility = 16 : i32}
283|     ) {
284|     %cst = arith.constant dense<0.000000e+00> : tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
285|     %c128_i32 = arith.constant 128 : i32
286|     %1 = tt.get_program_id y : i32
287|     %2 = tt.get_num_programs y : i32
288|     %12 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
289|     %19 = scf.for %arg3 = %1 to %11 step %2 iter_args(%arg4 = %cst) -> (tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>>)  : i32 {
290|       %27 = arith.muli %arg3, %c128_i32 : i32
291|       %28 = tt.splat %27 : i32 -> tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
292|       %29 = arith.addi %28, %12 : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
293|       %30 = tt.expand_dims %29 {axis = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x128xi32, #blocked>
```
**EN:** This function-oriented block defines or enters `max_reduce_zero_int_accumulator`. Within it, the test exercises tt.func, constants, program IDs, tt.get_num_programs, lane/block index ranges, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `max_reduce_zero_int_accumulator` 为核心。测试在其中演示 tt.func、常量、程序 ID、tt.get_num_programs、lane/block 索引范围，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 294-310
```mlir
294|       %31 = tt.broadcast %30 : tensor<1x128xi32, #blocked> -> tensor<32x128xi32, #blocked>
295|       %32 = tt.addptr %18, %31 : tensor<32x128x!tt.ptr<f32>, #blocked>, tensor<32x128xi32, #blocked>
296|       %33 = tt.load %32 : tensor<32x128x!tt.ptr<f32>, #blocked>
297|       %34 = "tt.reduce"(%33) <{axis = 1 : i32}> ({
298|       ^bb0(%arg5: f32, %arg6: f32):
299|         %36 = arith.maximumf %arg5, %arg6 : f32
300|         tt.reduce.return %36 : f32
301|       }) : (tensor<32x128xf32, #blocked>) -> tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
302|       %35 = arith.maximumf %arg4, %34 : tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
303|       scf.yield %35 : tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
304|     }
305|     %26 = ttg.convert_layout %19 : tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<32xf32, #blocked1>
306|     tt.store %25, %26 : tensor<32x!tt.ptr<f32>, #blocked1>
307|     tt.return
308|   }
309| }
310| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining arith.maximumf, tensor broadcasting, pointer arithmetic, masked or vectorized loads, reductions.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 arith.maximumf、张量广播、指针算术、带掩码或向量化的加载、归约。

### Lines 311-311
```mlir
311| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 312-325
```mlir
312| 
313| // CHECK-LABEL: min_reduce
314| // CHECK: %[[CST:.*]] = arith.constant dense<0x7F800000>
315| // CHECK: %[[LOOP_OUTPUT:.*]] = scf.for {{.*}} iter_args(%[[FOR_ARG:.*]] = %[[CST]]) -> {{.*}}
316| // CHECK: %[[LOAD:.*]] = tt.load
317| // CHECK: tt.reshape %[[LOAD]] allow_reorder efficient_layout : {{.*}} -> tensor<{{32x32x4xf32.*}}
318| // CHECK-NEXT: %[[REDUCE:.*]] = "tt.reduce"({{%.*}}) <{axis = 2 : i32}>
319| // CHECK: arith.minimumf
320| // CHECK: arith.minimumf %[[FOR_ARG]], %[[REDUCE]]
321| // CHECK-NEXT: scf.yield
322| // CHECK: %[[FINAL_REDUCE:.*]] = "tt.reduce"(%[[LOOP_OUTPUT]]) <{axis = 1 : i32}>
323| // CHECK: arith.minimumf
324| // CHECK: %[[CVT_OUTPUT:.*]] = ttg.convert_layout %[[FINAL_REDUCE]]
325| // CHECK: tt.store {{%.*}}, %[[CVT_OUTPUT]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: min_reduce anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: min_reduce 这样的标签用于锚定匹配范围。

### Lines 326-327
```mlir
326| #blocked = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
327| #blocked1 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 328-328
```mlir
328| module attributes {"ttg.target" = "cuda:80", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 329-346
```mlir
329|   tt.func public @min_reduce(
330|     %arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32},
331|     %arg1: !tt.ptr<f32> {tt.divisibility = 16 : i32},
332|     %arg2: i32 {tt.divisibility = 16 : i32, tt.max_divisibility = 8 : i32},
333|     %18: tensor<32x128x!tt.ptr<f32>, #blocked> {tt.divisibility = 16 : i32},
334|     %11: i32 {tt.divisibility = 16 : i32},
335|     %25: tensor<32x!tt.ptr<f32>, #blocked1> {tt.divisibility = 16 : i32}
336|     ) {
337|     %cst = arith.constant dense<0x7F800000> : tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
338|     %c128_i32 = arith.constant 128 : i32
339|     %1 = tt.get_program_id y : i32
340|     %2 = tt.get_num_programs y : i32
341|     %12 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
342|     %19 = scf.for %arg3 = %1 to %11 step %2 iter_args(%arg4 = %cst) -> (tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>>)  : i32 {
343|       %27 = arith.muli %arg3, %c128_i32 : i32
344|       %28 = tt.splat %27 : i32 -> tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
345|       %29 = arith.addi %28, %12 : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
346|       %30 = tt.expand_dims %29 {axis = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x128xi32, #blocked>
```
**EN:** This function-oriented block defines or enters `min_reduce`. Within it, the test exercises tt.func, constants, program IDs, tt.get_num_programs, lane/block index ranges, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `min_reduce` 为核心。测试在其中演示 tt.func、常量、程序 ID、tt.get_num_programs、lane/block 索引范围，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 347-363
```mlir
347|       %31 = tt.broadcast %30 : tensor<1x128xi32, #blocked> -> tensor<32x128xi32, #blocked>
348|       %32 = tt.addptr %18, %31 : tensor<32x128x!tt.ptr<f32>, #blocked>, tensor<32x128xi32, #blocked>
349|       %33 = tt.load %32 : tensor<32x128x!tt.ptr<f32>, #blocked>
350|       %34 = "tt.reduce"(%33) <{axis = 1 : i32}> ({
351|       ^bb0(%arg5: f32, %arg6: f32):
352|         %36 = arith.minimumf %arg5, %arg6 : f32
353|         tt.reduce.return %36 : f32
354|       }) : (tensor<32x128xf32, #blocked>) -> tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
355|       %35 = arith.minimumf %arg4, %34 : tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
356|       scf.yield %35 : tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
357|     }
358|     %26 = ttg.convert_layout %19 : tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<32xf32, #blocked1>
359|     tt.store %25, %26 : tensor<32x!tt.ptr<f32>, #blocked1>
360|     tt.return
361|   }
362| }
363| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining arith.minimumf, tensor broadcasting, pointer arithmetic, masked or vectorized loads, reductions.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 arith.minimumf、张量广播、指针算术、带掩码或向量化的加载、归约。

### Lines 364-364
```mlir
364| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 365-379
```mlir
365| 
366| // CHECK-LABEL: min_reduce_zero_int_accumulator
367| // CHECK: %[[CST:.*]] = arith.constant dense<0.000000e+00>
368| // CHECK-NEXT: %[[CST1:.*]] = arith.constant dense<0x7F800000>
369| // CHECK: %[[LOOP_OUTPUT:.*]] = scf.for {{.*}} iter_args(%[[FOR_ARG:.*]] = %[[CST1]]) -> {{.*}}
370| // CHECK: tt.load
371| // CHECK: tt.reshape
372| // CHECK-NEXT: %[[REDUCE:.*]] = "tt.reduce"({{%.*}}) <{axis = 2 : i32}>
373| // CHECK: arith.minimumf
374| // CHECK: arith.minimumf %[[FOR_ARG]], %[[REDUCE]]
375| // CHECK-NEXT: scf.yield
376| // CHECK: %[[FINAL_REDUCE:.*]] = "tt.reduce"(%[[LOOP_OUTPUT]]) <{axis = 1 : i32}>
377| // CHECK: arith.minimumf
378| // CHECK: %[[CVT_OUTPUT:.*]] = ttg.convert_layout %[[FINAL_REDUCE]]
379| // CHECK: arith.minimumf %[[CVT_OUTPUT]], %[[CST]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: min_reduce_zero_int_accumulator anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: min_reduce_zero_int_accumulator 这样的标签用于锚定匹配范围。

### Lines 380-381
```mlir
380| #blocked = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
381| #blocked1 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 382-382
```mlir
382| module attributes {"ttg.target" = "cuda:80", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 383-400
```mlir
383|   tt.func public @min_reduce_zero_int_accumulator(
384|     %arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32},
385|     %arg1: !tt.ptr<f32> {tt.divisibility = 16 : i32},
386|     %arg2: i32 {tt.divisibility = 16 : i32, tt.max_divisibility = 8 : i32},
387|     %18: tensor<32x128x!tt.ptr<f32>, #blocked> {tt.divisibility = 16 : i32},
388|     %11: i32 {tt.divisibility = 16 : i32},
389|     %25: tensor<32x!tt.ptr<f32>, #blocked1> {tt.divisibility = 16 : i32}
390|     ) {
391|     %cst = arith.constant dense<0.000000e+00> : tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
392|     %c128_i32 = arith.constant 128 : i32
393|     %1 = tt.get_program_id y : i32
394|     %2 = tt.get_num_programs y : i32
395|     %12 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
396|     %19 = scf.for %arg3 = %1 to %11 step %2 iter_args(%arg4 = %cst) -> (tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>>)  : i32 {
397|       %27 = arith.muli %arg3, %c128_i32 : i32
398|       %28 = tt.splat %27 : i32 -> tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
399|       %29 = arith.addi %28, %12 : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
400|       %30 = tt.expand_dims %29 {axis = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x128xi32, #blocked>
```
**EN:** This function-oriented block defines or enters `min_reduce_zero_int_accumulator`. Within it, the test exercises tt.func, constants, program IDs, tt.get_num_programs, lane/block index ranges, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `min_reduce_zero_int_accumulator` 为核心。测试在其中演示 tt.func、常量、程序 ID、tt.get_num_programs、lane/block 索引范围，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 401-417
```mlir
401|       %31 = tt.broadcast %30 : tensor<1x128xi32, #blocked> -> tensor<32x128xi32, #blocked>
402|       %32 = tt.addptr %18, %31 : tensor<32x128x!tt.ptr<f32>, #blocked>, tensor<32x128xi32, #blocked>
403|       %33 = tt.load %32 : tensor<32x128x!tt.ptr<f32>, #blocked>
404|       %34 = "tt.reduce"(%33) <{axis = 1 : i32}> ({
405|       ^bb0(%arg5: f32, %arg6: f32):
406|         %36 = arith.minimumf %arg5, %arg6 : f32
407|         tt.reduce.return %36 : f32
408|       }) : (tensor<32x128xf32, #blocked>) -> tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
409|       %35 = arith.minimumf %arg4, %34 : tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
410|       scf.yield %35 : tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
411|     }
412|     %26 = ttg.convert_layout %19 : tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<32xf32, #blocked1>
413|     tt.store %25, %26 : tensor<32x!tt.ptr<f32>, #blocked1>
414|     tt.return
415|   }
416| }
417| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining arith.minimumf, tensor broadcasting, pointer arithmetic, masked or vectorized loads, reductions.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 arith.minimumf、张量广播、指针算术、带掩码或向量化的加载、归约。

### Lines 418-418
```mlir
418| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 419-432
```mlir
419| 
420| // CHECK-LABEL: mul_reduce
421| // CHECK: %[[CST:.*]] = arith.constant dense<1.000000e+00>
422| // CHECK: %[[LOOP_OUTPUT:.*]] = scf.for {{.*}} iter_args(%[[FOR_ARG:.*]] = %[[CST]]) -> {{.*}}
423| // CHECK: %[[LOAD:.*]] = tt.load
424| // CHECK: tt.reshape %[[LOAD]] allow_reorder efficient_layout : {{.*}} -> tensor<{{32x32x4xf32.*}}
425| // CHECK-NEXT: %[[REDUCE:.*]] = "tt.reduce"({{%.*}}) <{axis = 2 : i32}>
426| // CHECK: arith.mulf
427| // CHECK: arith.mulf %[[FOR_ARG]], %[[REDUCE]]
428| // CHECK-NEXT: scf.yield
429| // CHECK: %[[FINAL_REDUCE:.*]] = "tt.reduce"(%[[LOOP_OUTPUT]]) <{axis = 1 : i32}>
430| // CHECK: arith.mulf
431| // CHECK: %[[CVT_OUTPUT:.*]] = ttg.convert_layout %[[FINAL_REDUCE]]
432| // CHECK: tt.store {{%.*}}, %[[CVT_OUTPUT]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: mul_reduce anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: mul_reduce 这样的标签用于锚定匹配范围。

### Lines 433-434
```mlir
433| #blocked = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
434| #blocked1 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 435-435
```mlir
435| module attributes {"ttg.target" = "cuda:80", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 436-453
```mlir
436|   tt.func public @mul_reduce(
437|     %arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32},
438|     %arg1: !tt.ptr<f32> {tt.divisibility = 16 : i32},
439|     %arg2: i32 {tt.divisibility = 16 : i32, tt.max_divisibility = 8 : i32},
440|     %18: tensor<32x128x!tt.ptr<f32>, #blocked> {tt.divisibility = 16 : i32},
441|     %11: i32 {tt.divisibility = 16 : i32},
442|     %25: tensor<32x!tt.ptr<f32>, #blocked1> {tt.divisibility = 16 : i32}
443|     ) {
444|     %cst = arith.constant dense<1.000000e+00> : tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
445|     %c128_i32 = arith.constant 128 : i32
446|     %1 = tt.get_program_id y : i32
447|     %2 = tt.get_num_programs y : i32
448|     %12 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
449|     %19 = scf.for %arg3 = %1 to %11 step %2 iter_args(%arg4 = %cst) -> (tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>>)  : i32 {
450|       %27 = arith.muli %arg3, %c128_i32 : i32
451|       %28 = tt.splat %27 : i32 -> tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
452|       %29 = arith.addi %28, %12 : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
453|       %30 = tt.expand_dims %29 {axis = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x128xi32, #blocked>
```
**EN:** This function-oriented block defines or enters `mul_reduce`. Within it, the test exercises tt.func, constants, program IDs, tt.get_num_programs, lane/block index ranges, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `mul_reduce` 为核心。测试在其中演示 tt.func、常量、程序 ID、tt.get_num_programs、lane/block 索引范围，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 454-470
```mlir
454|       %31 = tt.broadcast %30 : tensor<1x128xi32, #blocked> -> tensor<32x128xi32, #blocked>
455|       %32 = tt.addptr %18, %31 : tensor<32x128x!tt.ptr<f32>, #blocked>, tensor<32x128xi32, #blocked>
456|       %33 = tt.load %32 : tensor<32x128x!tt.ptr<f32>, #blocked>
457|       %34 = "tt.reduce"(%33) <{axis = 1 : i32}> ({
458|       ^bb0(%arg5: f32, %arg6: f32):
459|         %36 = arith.mulf %arg5, %arg6 : f32
460|         tt.reduce.return %36 : f32
461|       }) : (tensor<32x128xf32, #blocked>) -> tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
462|       %35 = arith.mulf %arg4, %34 : tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
463|       scf.yield %35 : tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
464|     }
465|     %26 = ttg.convert_layout %19 : tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<32xf32, #blocked1>
466|     tt.store %25, %26 : tensor<32x!tt.ptr<f32>, #blocked1>
467|     tt.return
468|   }
469| }
470| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining arith.mulf, tensor broadcasting, pointer arithmetic, masked or vectorized loads, reductions.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 arith.mulf、张量广播、指针算术、带掩码或向量化的加载、归约。

### Lines 471-471
```mlir
471| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 472-486
```mlir
472| 
473| // CHECK-LABEL: mul_reduce_zero_int_accumulator
474| // CHECK: %[[CST:.*]] = arith.constant dense
475| // CHECK-NEXT: %[[CST1:.*]] = arith.constant dense<1.000000e+00>
476| // CHECK: %[[LOOP_OUTPUT:.*]] = scf.for {{.*}} iter_args(%[[FOR_ARG:.*]] = %[[CST1]]) -> {{.*}}
477| // CHECK: tt.load
478| // CHECK: tt.reshape
479| // CHECK-NEXT: %[[REDUCE:.*]] = "tt.reduce"({{%.*}}) <{axis = 2 : i32}>
480| // CHECK: arith.mulf
481| // CHECK: arith.mulf %[[FOR_ARG]], %[[REDUCE]]
482| // CHECK-NEXT: scf.yield
483| // CHECK: %[[FINAL_REDUCE:.*]] = "tt.reduce"(%[[LOOP_OUTPUT]]) <{axis = 1 : i32}>
484| // CHECK: arith.mulf
485| // CHECK: %[[CVT_OUTPUT:.*]] = ttg.convert_layout %[[FINAL_REDUCE]]
486| // CHECK: arith.mulf %[[CVT_OUTPUT]], %[[CST]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: mul_reduce_zero_int_accumulator anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: mul_reduce_zero_int_accumulator 这样的标签用于锚定匹配范围。

### Lines 487-488
```mlir
487| #blocked = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
488| #blocked1 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 489-489
```mlir
489| module attributes {"ttg.target" = "cuda:80", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 490-507
```mlir
490|   tt.func public @mul_reduce_zero_int_accumulator(
491|     %arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32},
492|     %arg1: !tt.ptr<f32> {tt.divisibility = 16 : i32},
493|     %arg2: i32 {tt.divisibility = 16 : i32, tt.max_divisibility = 8 : i32},
494|     %18: tensor<32x128x!tt.ptr<f32>, #blocked> {tt.divisibility = 16 : i32},
495|     %11: i32 {tt.divisibility = 16 : i32},
496|     %25: tensor<32x!tt.ptr<f32>, #blocked1> {tt.divisibility = 16 : i32}
497|     ) {
498|     %cst = arith.constant dense<0.000000e+00> : tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
499|     %c128_i32 = arith.constant 128 : i32
500|     %1 = tt.get_program_id y : i32
501|     %2 = tt.get_num_programs y : i32
502|     %12 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
503|     %19 = scf.for %arg3 = %1 to %11 step %2 iter_args(%arg4 = %cst) -> (tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>>)  : i32 {
504|       %27 = arith.muli %arg3, %c128_i32 : i32
505|       %28 = tt.splat %27 : i32 -> tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
506|       %29 = arith.addi %28, %12 : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
507|       %30 = tt.expand_dims %29 {axis = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x128xi32, #blocked>
```
**EN:** This function-oriented block defines or enters `mul_reduce_zero_int_accumulator`. Within it, the test exercises tt.func, constants, program IDs, tt.get_num_programs, lane/block index ranges, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `mul_reduce_zero_int_accumulator` 为核心。测试在其中演示 tt.func、常量、程序 ID、tt.get_num_programs、lane/block 索引范围，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 508-525
```mlir
508|       %31 = tt.broadcast %30 : tensor<1x128xi32, #blocked> -> tensor<32x128xi32, #blocked>
509|       %32 = tt.addptr %18, %31 : tensor<32x128x!tt.ptr<f32>, #blocked>, tensor<32x128xi32, #blocked>
510|       %33 = tt.load %32 : tensor<32x128x!tt.ptr<f32>, #blocked>
511|       %34 = "tt.reduce"(%33) <{axis = 1 : i32}> ({
512|       ^bb0(%arg5: f32, %arg6: f32):
513|         %36 = arith.mulf %arg5, %arg6 : f32
514|         tt.reduce.return %36 : f32
515|       }) : (tensor<32x128xf32, #blocked>) -> tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
516|       %35 = arith.mulf %arg4, %34 : tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
517|       scf.yield %35 : tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
518|     }
519|     %26 = ttg.convert_layout %19 : tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<32xf32, #blocked1>
520|     tt.store %25, %26 : tensor<32x!tt.ptr<f32>, #blocked1>
521|     tt.return
522|   }
523| }
524| 
525| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining arith.mulf, tensor broadcasting, pointer arithmetic, masked or vectorized loads, reductions.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 arith.mulf、张量广播、指针算术、带掩码或向量化的加载、归约。

### Lines 526-526
```mlir
526| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 527-536
```mlir
527| 
528| // CHECK-LABEL: remains_unchanged
529| // CHECK: %[[CST:.*]] = arith.constant dense
530| // CHECK: %[[LOOP_OUTPUT:.*]] = scf.for {{.*}} iter_args(%[[FOR_ARG:.*]] = %[[CST]]) -> {{.*}}
531| // CHECK: %[[LOAD:.*]] = tt.load
532| // CHECK: %[[MULF:.*]] = arith.mulf %[[LOAD]], %[[LOAD]]
533| // CHECK-NEXT: %[[REDUCE:.*]] = "tt.reduce"(%[[MULF]]) <{axis = 1 : i32}>
534| // CHECK: arith.maximumf
535| // CHECK: arith.maximumf %[[FOR_ARG]], %[[REDUCE]]
536| // CHECK-NEXT: scf.yield
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: remains_unchanged anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: remains_unchanged 这样的标签用于锚定匹配范围。

### Lines 537-538
```mlir
537| #blocked = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
538| #blocked1 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 539-539
```mlir
539| module attributes {"ttg.target" = "cuda:80", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 540-557
```mlir
540|   tt.func public @remains_unchanged(
541|     %arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32},
542|     %arg1: !tt.ptr<f32> {tt.divisibility = 16 : i32},
543|     %arg2: i32 {tt.divisibility = 16 : i32, tt.max_divisibility = 8 : i32},
544|     %18: tensor<32x128x!tt.ptr<f32>, #blocked> {tt.divisibility = 16 : i32},
545|     %11: i32 {tt.divisibility = 16 : i32},
546|     %25: tensor<32x!tt.ptr<f32>, #blocked1> {tt.divisibility = 16 : i32}
547|     ) {
548|     %cst = arith.constant dense<0.000000e+00> : tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
549|     %c128_i32 = arith.constant 128 : i32
550|     %1 = tt.get_program_id y : i32
551|     %2 = tt.get_num_programs y : i32
552|     %12 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
553|     %19 = scf.for %arg3 = %1 to %11 step %2 iter_args(%arg4 = %cst) -> (tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>>)  : i32 {
554|       %27 = arith.muli %arg3, %c128_i32 : i32
555|       %28 = tt.splat %27 : i32 -> tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
556|       %29 = arith.addi %28, %12 : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
557|       %30 = tt.expand_dims %29 {axis = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x128xi32, #blocked>
```
**EN:** This function-oriented block defines or enters `remains_unchanged`. Within it, the test exercises tt.func, constants, program IDs, tt.get_num_programs, lane/block index ranges, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `remains_unchanged` 为核心。测试在其中演示 tt.func、常量、程序 ID、tt.get_num_programs、lane/block 索引范围，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 558-575
```mlir
558|       %31 = tt.broadcast %30 : tensor<1x128xi32, #blocked> -> tensor<32x128xi32, #blocked>
559|       %32 = tt.addptr %18, %31 : tensor<32x128x!tt.ptr<f32>, #blocked>, tensor<32x128xi32, #blocked>
560|       %33 = tt.load %32 : tensor<32x128x!tt.ptr<f32>, #blocked>
561|       %333 = arith.mulf %33, %33: tensor<32x128xf32, #blocked>
562|       %34 = "tt.reduce"(%333) <{axis = 1 : i32}> ({
563|       ^bb0(%arg5: f32, %arg6: f32):
564|         %36 = arith.maximumf %arg5, %arg6 : f32
565|         tt.reduce.return %36 : f32
566|       }) : (tensor<32x128xf32, #blocked>) -> tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
567|       %35 = arith.maximumf %arg4, %34 : tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
568|       scf.yield %35 : tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
569|     }
570|     %26 = ttg.convert_layout %19 : tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<32xf32, #blocked1>
571|     tt.store %25, %26 : tensor<32x!tt.ptr<f32>, #blocked1>
572|     tt.return
573|   }
574| }
575| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining arith.maximumf, tensor broadcasting, pointer arithmetic, masked or vectorized loads, arith.mulf.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 arith.maximumf、张量广播、指针算术、带掩码或向量化的加载、arith.mulf。

### Lines 576-576
```mlir
576| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 577-584
```mlir
577| 
578| // CHECK-DAG: #[[$BLOCK0:.+]] = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [4, 8], warpsPerCTA = [2, 1], order = [1, 0]}>
579| // CHECK-DAG: #[[$BLOCK1:.+]] = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [2, 16], warpsPerCTA = [2, 1], order = [1, 0]}>
580| // CHECK-DAG: #[[$BLOCK2:.+]] = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [2, 1], order = [0, 1]}>
581| // CHECK-LABEL: optimize_view_layout
582| // CHECK: %[[R:.+]] = tt.reshape {{.*}} allow_reorder efficient_layout : tensor<8x128xf32, #[[$BLOCK0]]> -> tensor<64x16xf32, #[[$BLOCK2]]>
583| // CHECK: %[[C:.+]] = ttg.convert_layout %[[R]] : tensor<64x16xf32, #[[$BLOCK2]]> -> tensor<64x16xf32, #[[$BLOCK1]]>
584| // CHECK:  "tt.reduce"(%[[C]])
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: optimize_view_layout anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: optimize_view_layout 这样的标签用于锚定匹配范围。

### Lines 585-586
```mlir
585| #blocked = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [4, 8], warpsPerCTA = [2, 1], order = [1, 0]}>
586| #blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [2, 16], warpsPerCTA = [2, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 587-587
```mlir
587| module attributes {"ttg.target" = "cuda:80", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 2 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 588-598
```mlir
588|   tt.func public @optimize_view_layout(%arg0: tensor<8x128xf32, #blocked>) -> tensor<64xf32, #ttg.slice<{dim = 1, parent = #blocked1}>> {
589|     %0 = tt.reshape %arg0 allow_reorder : tensor<8x128xf32, #blocked> -> tensor<64x16xf32, #blocked1>
590|     %1 = "tt.reduce"(%0) <{axis = 1 : i32}> ({
591|     ^bb0(%arg1: f32, %arg2: f32):
592|       %2 = arith.maximumf %arg1, %arg2 : f32
593|       tt.reduce.return %2 : f32
594|     }) : (tensor<64x16xf32, #blocked1>) -> tensor<64xf32, #ttg.slice<{dim = 1, parent = #blocked1}>>
595|     tt.return %1 : tensor<64xf32, #ttg.slice<{dim = 1, parent = #blocked1}>>
596|   }
597| }
598| 
```
**EN:** This function-oriented block defines or enters `optimize_view_layout`. Within it, the test exercises tt.func, tensor reshaping, arith.maximumf, reductions, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `optimize_view_layout` 为核心。测试在其中演示 tt.func、张量重塑、arith.maximumf、归约、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 599-599
```mlir
599| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 600-607
```mlir
600| 
601| 
602| // CHECK-DAG: #[[$BLOCK0:.+]] = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [2, 16], warpsPerCTA = [2, 1], order = [1, 0]}>
603| // CHECK-DAG: #[[$BLOCK1:.+]] = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [2, 1], order = [0, 1]}>
604| // CHECK-LABEL: optimize_view_layout_same_shape
605| // CHECK: %[[R:.+]] = tt.reshape {{.*}} allow_reorder efficient_layout : tensor<64x16xf32, #[[$BLOCK0]]> -> tensor<64x16xf32, #[[$BLOCK1]]>
606| // CHECK: %[[C:.+]] = ttg.convert_layout %[[R]] : tensor<64x16xf32, #[[$BLOCK1]]> -> tensor<64x16xf32, #[[$BLOCK0]]>
607| // CHECK:  "tt.reduce"(%[[C]])
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: optimize_view_layout_same_shape anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: optimize_view_layout_same_shape 这样的标签用于锚定匹配范围。

### Lines 608-608
```mlir
608| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [2, 16], warpsPerCTA = [2, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 609-609
```mlir
609| module attributes {"ttg.target" = "cuda:80", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 2 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 610-620
```mlir
610|   tt.func public @optimize_view_layout_same_shape(%arg0: tensor<64x16xf32, #blocked>) -> tensor<64xf32, #ttg.slice<{dim = 1, parent = #blocked}>> {
611|     %0 = tt.reshape %arg0 allow_reorder : tensor<64x16xf32, #blocked> -> tensor<64x16xf32, #blocked>
612|     %1 = "tt.reduce"(%0) <{axis = 1 : i32}> ({
613|     ^bb0(%arg1: f32, %arg2: f32):
614|       %2 = arith.maximumf %arg1, %arg2 : f32
615|       tt.reduce.return %2 : f32
616|     }) : (tensor<64x16xf32, #blocked>) -> tensor<64xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
617|     tt.return %1 : tensor<64xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
618|   }
619| }
620| 
```
**EN:** This function-oriented block defines or enters `optimize_view_layout_same_shape`. Within it, the test exercises tt.func, tensor reshaping, arith.maximumf, reductions, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `optimize_view_layout_same_shape` 为核心。测试在其中演示 tt.func、张量重塑、arith.maximumf、归约、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 621-621
```mlir
621| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 622-624
```mlir
622| #blocked = #ttg.blocked<{sizePerThread = [8, 1], threadsPerWarp = [32, 1], warpsPerCTA = [1, 1], order = [1, 0]}>
623| #blocked1 = #ttg.blocked<{sizePerThread = [8], threadsPerWarp = [32], warpsPerCTA = [1], order = [0]}>
624| #slice = #ttg.slice<{dim = 1, parent = #blocked}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 625-625
```mlir
625| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 626-643
```mlir
626|   tt.func public @reduce_for_arg(%arg: tensor<64x128xf32, #blocked>, %arg1: !tt.ptr<f32>) {
627|     %c0_i32 = arith.constant 0 : i32
628|     %c128_i32 = arith.constant 128 : i32
629|     %c4096_i32 = arith.constant 4096 : i32
630|     %cst_1 = arith.constant dense<1.000000e+00> : tensor<64x128xf32, #blocked>
631|     %64:1 = scf.for %arg22 = %c0_i32 to %c4096_i32 step %c128_i32 iter_args(%arg29 = %arg) -> (tensor<64x128xf32, #blocked>)  : i32 {
632|       %129 = "tt.reduce"(%arg29) <{axis = 1 : i32}> ({
633|       ^bb0(%arg31: f32, %arg32: f32):
634|         %160 = arith.maxnumf %arg31, %arg32 : f32
635|         tt.reduce.return %160 : f32
636|       }) : (tensor<64x128xf32, #blocked>) -> tensor<64xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
637|       %75 = ttg.convert_layout %129 : tensor<64xf32, #slice> -> tensor<64xf32, #blocked1>
638|       %79 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #blocked1>
639|       %80 = tt.splat %arg1 : !tt.ptr<f32> -> tensor<64x!tt.ptr<f32>, #blocked1>
640|       %81 = tt.addptr %80, %79 : tensor<64x!tt.ptr<f32>, #blocked1>, tensor<64xi32, #blocked1>
641|       tt.store %81, %75 : tensor<64x!tt.ptr<f32>, #blocked1>
642|       %141 = arith.addf %arg29, %cst_1 : tensor<64x128xf32, #blocked>
643|       scf.yield %141 : tensor<64x128xf32, #blocked>
```
**EN:** This function-oriented block defines or enters `reduce_for_arg`. Within it, the test exercises constants, tt.func, structured loops, arith.maxnumf, reductions, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `reduce_for_arg` 为核心。测试在其中演示 常量、tt.func、结构化循环、arith.maxnumf、归约，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 644-648
```mlir
644|     }
645|     tt.return
646|   }
647| }
648| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 649-649
```mlir
649| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 650-652
```mlir
650| 
651| #blocked = #ttg.blocked<{sizePerThread = [2, 2], threadsPerWarp = [16, 2], warpsPerCTA = [2, 2], order = [1, 0]}>
652| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 653-654
```mlir
653| // CHECK: [[LAYOUT:#.*]] = #ttg.blocked<{sizePerThread = [2, 1], threadsPerWarp = [32, 1], warpsPerCTA = [1, 4], order = [0, 1]}>
654| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 655-656
```mlir
655| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
656| 
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 657-657
```mlir
657| // CHECK: set_warp_shuffle_layout_square_axis_0
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 658-658
```mlir
658| tt.func @set_warp_shuffle_layout_square_axis_0(%arg0: tensor<64x64xf32, #blocked>, %arg1: tensor<64x64xi32, #blocked>) -> tensor<64x64xf32, #blocked> {
```
**EN:** This function-oriented block defines or enters `set_warp_shuffle_layout_square_axis_0`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `set_warp_shuffle_layout_square_axis_0` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 659-662
```mlir
659|   // CHECK-NEXT: [[SRC:%.*]] = ttg.convert_layout %arg0
660|   // CHECK-NEXT: [[IDX:%.*]] = ttg.convert_layout %arg1
661|   // CHECK-NEXT: [[OUT:%.*]] = tt.gather [[SRC]][[[IDX]]] {axis = 0 : i32, efficient_layout} : (tensor<64x64xf32, [[LAYOUT]]>, tensor<64x64xi32, [[LAYOUT]]>) -> tensor<64x64xf32, [[LAYOUT]]>
662|   %0 = tt.gather %arg0[%arg1] {axis = 0 : i32} : (tensor<64x64xf32, #blocked>, tensor<64x64xi32, #blocked>) -> tensor<64x64xf32, #blocked>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.gather. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.gather。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 663-669
```mlir
663|   // CHECK-NEXT: [[RES:%.*]] = ttg.convert_layout [[OUT]]
664|   // CHECK-NEXT: return [[RES]]
665|   tt.return %0 : tensor<64x64xf32, #blocked>
666| }
667| 
668| }
669| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 670-670
```mlir
670| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 671-673
```mlir
671| 
672| #blocked = #ttg.blocked<{sizePerThread = [2, 2], threadsPerWarp = [16, 2], warpsPerCTA = [2, 2], order = [1, 0]}>
673| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 674-675
```mlir
674| // CHECK: [[LAYOUT:#.*]] = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
675| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 676-677
```mlir
676| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
677| 
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 678-678
```mlir
678| // CHECK: set_warp_shuffle_layout_square_axis_1
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 679-679
```mlir
679| tt.func @set_warp_shuffle_layout_square_axis_1(%arg0: tensor<64x64xf32, #blocked>, %arg1: tensor<64x64xi32, #blocked>) -> tensor<64x64xf32, #blocked> {
```
**EN:** This function-oriented block defines or enters `set_warp_shuffle_layout_square_axis_1`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `set_warp_shuffle_layout_square_axis_1` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 680-686
```mlir
680|   // CHECK: tt.gather {{.*}} (tensor<64x64xf32, [[LAYOUT]]>, tensor<64x64xi32, [[LAYOUT]]>) -> tensor<64x64xf32, [[LAYOUT]]>
681|   %0 = tt.gather %arg0[%arg1] {axis = 1 : i32} : (tensor<64x64xf32, #blocked>, tensor<64x64xi32, #blocked>) -> tensor<64x64xf32, #blocked>
682|   tt.return %0 : tensor<64x64xf32, #blocked>
683| }
684| 
685| }
686| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.gather, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.gather、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 687-687
```mlir
687| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 688-690
```mlir
688| 
689| #blocked = #ttg.blocked<{sizePerThread = [2, 2], threadsPerWarp = [16, 2], warpsPerCTA = [2, 2], order = [1, 0]}>
690| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 691-692
```mlir
691| // CHECK: [[LAYOUT:#.*]] = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
692| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 693-694
```mlir
693| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
694| 
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 695-695
```mlir
695| // CHECK: set_warp_shuffle_layout_warp_broadcast
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 696-696
```mlir
696| tt.func @set_warp_shuffle_layout_warp_broadcast(%arg0: tensor<64x64xf32, #blocked>, %arg1: tensor<64x1xi32, #blocked>) -> tensor<64x1xf32, #blocked> {
```
**EN:** This function-oriented block defines or enters `set_warp_shuffle_layout_warp_broadcast`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `set_warp_shuffle_layout_warp_broadcast` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 697-703
```mlir
697|   // CHECK: tt.gather {{.*}} [[LAYOUT]]>
698|   %0 = tt.gather %arg0[%arg1] {axis = 1 : i32} : (tensor<64x64xf32, #blocked>, tensor<64x1xi32, #blocked>) -> tensor<64x1xf32, #blocked>
699|   tt.return %0 : tensor<64x1xf32, #blocked>
700| }
701| 
702| }
703| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.gather, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.gather、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 704-704
```mlir
704| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 705-707
```mlir
705| 
706| #blocked = #ttg.blocked<{sizePerThread = [2, 2, 1], threadsPerWarp = [16, 2, 1], warpsPerCTA = [2, 1, 2], order = [1, 0, 2]}>
707| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 708-709
```mlir
708| // CHECK: [[LAYOUT:#.*]] = #ttg.blocked<{sizePerThread = [1, 1, 1], threadsPerWarp = [1, 1, 32], warpsPerCTA = [2, 2, 1], order = [2, 0, 1]}>
709| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 710-711
```mlir
710| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
711| 
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 712-712
```mlir
712| // CHECK: set_warp_shuffle_layout_3d_warp
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 713-713
```mlir
713| tt.func @set_warp_shuffle_layout_3d_warp(%arg0: tensor<32x2x32xf32, #blocked>, %arg1: tensor<32x2x2xi32, #blocked>) -> tensor<32x2x2xf32, #blocked> {
```
**EN:** This function-oriented block defines or enters `set_warp_shuffle_layout_3d_warp`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `set_warp_shuffle_layout_3d_warp` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 714-720
```mlir
714|   // CHECK: tt.gather {{.*}} [[LAYOUT]]>
715|     %0 = tt.gather %arg0[%arg1] {axis = 2 : i32} : (tensor<32x2x32xf32, #blocked>, tensor<32x2x2xi32, #blocked>) -> tensor<32x2x2xf32, #blocked>
716|     tt.return %0 : tensor<32x2x2xf32, #blocked>
717| }
718| 
719| }
720| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.gather, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.gather、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 721-721
```mlir
721| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 722-724
```mlir
722| 
723| #blocked = #ttg.blocked<{sizePerThread = [2, 2, 1], threadsPerWarp = [16, 2, 1], warpsPerCTA = [2, 1, 2], order = [1, 0, 2]}>
724| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 725-726
```mlir
725| // CHECK: [[LAYOUT:#.*]] = #ttg.blocked<{sizePerThread = [1, 1, 1], threadsPerWarp = [1, 2, 16], warpsPerCTA = [2, 2, 1], order = [2, 1, 0]}>
726| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 727-728
```mlir
727| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
728| 
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 729-729
```mlir
729| // CHECK: set_warp_shuffle_layout_3d_warp_thread_split
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 730-730
```mlir
730| tt.func @set_warp_shuffle_layout_3d_warp_thread_split(%arg0: tensor<32x4x16xf32, #blocked>, %arg1: tensor<32x4x2xi32, #blocked>) -> tensor<32x4x2xf32, #blocked> {
```
**EN:** This function-oriented block defines or enters `set_warp_shuffle_layout_3d_warp_thread_split`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `set_warp_shuffle_layout_3d_warp_thread_split` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 731-738
```mlir
731|   // CHECK: tt.gather {{.*}} [[LAYOUT]]>
732|     %0 = tt.gather %arg0[%arg1] {axis = 2 : i32} : (tensor<32x4x16xf32, #blocked>, tensor<32x4x2xi32, #blocked>) -> tensor<32x4x2xf32, #blocked>
733|     tt.return %0 : tensor<32x4x2xf32, #blocked>
734| }
735| 
736| }
737| 
738| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.gather, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.gather、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 739-739
```mlir
739| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 740-742
```mlir
740| 
741| #blocked = #ttg.blocked<{sizePerThread = [2, 2], threadsPerWarp = [16, 2], warpsPerCTA = [2, 2], order = [1, 0]}>
742| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 743-744
```mlir
743| // CHECK: [[LAYOUT:#.*]] = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
744| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 745-746
```mlir
745| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
746| 
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 747-747
```mlir
747| // CHECK: set_warp_shuffle_layout_thread_broadcast
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 748-748
```mlir
748| tt.func @set_warp_shuffle_layout_thread_broadcast(%arg0: tensor<16x64xf32, #blocked>, %arg1: tensor<16x1xi32, #blocked>) -> tensor<16x1xf32, #blocked> {
```
**EN:** This function-oriented block defines or enters `set_warp_shuffle_layout_thread_broadcast`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `set_warp_shuffle_layout_thread_broadcast` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 749-755
```mlir
749|   // CHECK: tt.gather {{.*}} [[LAYOUT]]>
750|   %0 = tt.gather %arg0[%arg1] {axis = 1 : i32} : (tensor<16x64xf32, #blocked>, tensor<16x1xi32, #blocked>) -> tensor<16x1xf32, #blocked>
751|   tt.return %0 : tensor<16x1xf32, #blocked>
752| }
753| 
754| }
755| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.gather, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.gather、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 756-756
```mlir
756| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 757-759
```mlir
757| 
758| #blocked = #ttg.blocked<{sizePerThread = [2, 2], threadsPerWarp = [16, 2], warpsPerCTA = [2, 2], order = [1, 0]}>
759| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 760-761
```mlir
760| // CHECK: [[LAYOUT:#.*]] = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
761| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 762-763
```mlir
762| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
763| 
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 764-764
```mlir
764| // CHECK: set_warp_shuffle_layout_large_source
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 765-765
```mlir
765| tt.func @set_warp_shuffle_layout_large_source(%arg0: tensor<256x256xf32, #blocked>, %arg1: tensor<256x8xi32, #blocked>) -> tensor<256x8xf32, #blocked> {
```
**EN:** This function-oriented block defines or enters `set_warp_shuffle_layout_large_source`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `set_warp_shuffle_layout_large_source` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 766-773
```mlir
766|   // CHECK: tt.gather {{.*}} [[LAYOUT]]>
767|   %0 = tt.gather %arg0[%arg1] {axis = 1 : i32} : (tensor<256x256xf32, #blocked>, tensor<256x8xi32, #blocked>) -> tensor<256x8xf32, #blocked>
768|   tt.return %0 : tensor<256x8xf32, #blocked>
769| }
770| 
771| }
772| 
773| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.gather, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.gather、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 774-774
```mlir
774| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 775-777
```mlir
775| 
776| #blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
777| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 778-779
```mlir
778| // CHECK: [[LAYOUT:#.*]] = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
779| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 780-781
```mlir
780| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
781| 
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 782-782
```mlir
782| // CHECK: skip_optimize_on_1d_tensor
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 783-783
```mlir
783| tt.func @skip_optimize_on_1d_tensor(%arg0: tensor<256xf32, #blocked>, %arg1: tensor<8xi32, #blocked>) -> tensor<8xf32, #blocked> {
```
**EN:** This function-oriented block defines or enters `skip_optimize_on_1d_tensor`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `skip_optimize_on_1d_tensor` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 784-789
```mlir
784|   // CHECK: tt.gather {{.*}} [[LAYOUT]]>
785|   %0 = tt.gather %arg0[%arg1] {axis = 0 : i32} : (tensor<256xf32, #blocked>, tensor<8xi32, #blocked>) -> tensor<8xf32, #blocked>
786|   tt.return %0 : tensor<8xf32, #blocked>
787| }
788| 
789| }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.gather, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.gather、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `-tritongpu-optimize-thread-locality`, `-canonicalize`
- **CN:** 主要 pass 选项：`-split-input-file`，`-tritongpu-optimize-thread-locality`，`-canonicalize`
- **EN:** Dominant operations include `tt.func`, `arith.constant`, `module`, `tt.return`, `tt.reduce`, `tt.make_range`, `scf.for`, `tt.splat`, `tt.addptr`, `scf.yield`.
- **CN:** 主要操作包括 `tt.func`、`arith.constant`、`module`、`tt.return`、`tt.reduce`、`tt.make_range`、`scf.for`、`tt.splat`、`tt.addptr`、`scf.yield`。
- **EN:** The file contains 21 independently testable section(s). Check styles used: CHECK x123, CHECK-NEXT x31, CHECK-LABEL x13, CHECK-DAG x5. Important labels include negative_zero_accumulator, positive_zero_accumulator, slice_layout, mma_layout. It validates that the selected pass pipeline rewrites each labeled test case into the checked form.
- **CN:** 该文件包含 21 个可独立测试的分段。使用的检查类型：CHECK ×123，CHECK-NEXT ×31，CHECK-LABEL ×13，CHECK-DAG ×5。 关键标签包括 negative_zero_accumulator，positive_zero_accumulator，slice_layout，mma_layout。 它验证所选 pass 流水线会把每个带标签的测试用例重写成检查所要求的形式。

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