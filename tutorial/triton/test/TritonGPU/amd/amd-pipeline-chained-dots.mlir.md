# amd-pipeline-chained-dots.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/amd/amd-pipeline-chained-dots.mlir`
- **EN:** Test for the MLIR/Triton pipeline `-tritonamdgpu-schedule-loops="num_stages=4", -tritonamdgpu-pipeline="use_async_copy=1", -canonicalize` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `-tritonamdgpu-schedule-loops="num_stages=4", -tritonamdgpu-pipeline="use_async_copy=1", -canonicalize` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -split-input-file -tritonamdgpu-schedule-loops="num_stages=4" -tritonamdgpu-pipeline="use_async_copy=1" -canonicalize | FileCheck %s`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -split-input-file -tritonamdgpu-schedule-loops="num_stages=4" -tritonamdgpu-pipeline="use_async_copy=1" -canonicalize | FileCheck %s`

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
1| // RUN: triton-opt %s -split-input-file -tritonamdgpu-schedule-loops="num_stages=4" -tritonamdgpu-pipeline="use_async_copy=1" -canonicalize | FileCheck %s
2| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -split-input-file -tritonamdgpu-schedule-loops="num_stages=4" -tritonamdgpu-pipeline="use_async_copy=1" -canonicalize | FileCheck %s` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -split-input-file -tritonamdgpu-schedule-loops="num_stages=4" -tritonamdgpu-pipeline="use_async_copy=1" -canonicalize | FileCheck %s`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 3-4
```mlir
3| #blocked = #ttg.blocked<{sizePerThread = [8, 1], threadsPerWarp = [8, 8], warpsPerCTA = [1, 4], order = [0, 1]}>
4| #mma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [4, 1], instrShape = [32, 32, 16], isTransposed = true}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 5-5
```mlir
5| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 6-8
```mlir
6|   // CHECK-LABEL: tt.func @direct_chained_dots
7| 
8|   // We have no ops between the dots so we just check that dot and memory ops are in the correct order and check if basic pipelining (prologue, epilogue) is working correctly.
```
**EN:** This function-oriented block defines or enters `direct_chained_dots`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `direct_chained_dots` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 9-19
```mlir
 9|   // CHECK-COUNT-2: ttg.local_load
10|   // CHECK: scf.for
11|   // CHECK: tt.dot
12|   // CHECK: ttg.async_copy_global_to_local
13|   // CHECK: tt.dot
14|   // CHECK: ttg.async_wait
15|   // CHECK: ttg.local_load
16|   // CHECK: scf.yield
17|   // CHECK: ttg.async_wait
18|   // CHECK: ttg.local_load
19| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 20-37
```mlir
20|   tt.func @direct_chained_dots(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg2: tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>, %arg3: i32, %arg4: i32) -> tensor<128x16xf32, #mma> {
21|     %c0_i32 = arith.constant 0 : i32
22|     %cst = arith.constant dense<0.000000e+00> : tensor<128x16xf32, #mma>
23|     %0 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<1x16x!tt.ptr<f16>, #blocked>
24|     %1 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
25|     %2 = tt.expand_dims %1 {axis = 1 : i32} : tensor<64xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<64x1xi32, #blocked>
26|     %3 = tt.broadcast %0 : tensor<1x16x!tt.ptr<f16>, #blocked> -> tensor<64x16x!tt.ptr<f16>, #blocked>
27|     %4 = tt.broadcast %2 : tensor<64x1xi32, #blocked> -> tensor<64x16xi32, #blocked>
28|     %5 = tt.addptr %3, %4 : tensor<64x16x!tt.ptr<f16>, #blocked>, tensor<64x16xi32, #blocked>
29|     %6 = scf.for %arg6 = %c0_i32 to %arg3 step %arg4 iter_args(%arg5 = %cst) -> (tensor<128x16xf32, #mma>)  : i32 {
30|       %7 = tt.load %5 : tensor<64x16x!tt.ptr<f16>, #blocked>
31|       %8 = ttg.convert_layout %7 : tensor<64x16xf16, #blocked> -> tensor<64x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>
32|       %9 = tt.dot %arg2, %8, %cst : tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * tensor<64x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>> -> tensor<128x16xf32, #mma>
33|       %10 = tt.dot %arg2, %8, %9 : tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * tensor<64x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>> -> tensor<128x16xf32, #mma>
34|       scf.yield %10 : tensor<128x16xf32, #mma>
35|     }
36|     tt.return %6 : tensor<128x16xf32, #mma>
37|   }
```
**EN:** This function-oriented block defines or enters `direct_chained_dots`. Within it, the test exercises tt.func, constants, tensor broadcasting, dot-product or MMA-style math, broadcasted scalars or pointers, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `direct_chained_dots` 为核心。测试在其中演示 tt.func、常量、张量广播、点积或 MMA 风格计算、广播后的标量或指针，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 38-39
```mlir
38| }
39| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 40-40
```mlir
40| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 41-43
```mlir
41| 
42| #blocked = #ttg.blocked<{sizePerThread = [8, 1], threadsPerWarp = [8, 8], warpsPerCTA = [1, 4], order = [0, 1]}>
43| #mma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [4, 1], instrShape = [32, 32, 16], isTransposed = true}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 44-44
```mlir
44| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 45-50
```mlir
45|   // CHECK-LABEL: tt.func @chained_dots_with_ops_in_between
46| 
47|   // Ops between dots
48|   // dot1 -> reduce -> addf %dot1, %reduce1 -> add -> exp2 -> add -> dot2
49|   // We expect to split after the reduce because the result is used twice
50| 
```
**EN:** This function-oriented block defines or enters `chained_dots_with_ops_in_between`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `chained_dots_with_ops_in_between` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 51-68
```mlir
51|   // CHECK: scf.for
52| 
53|   // CHECK: tt.dot
54|   // CHECK: arith.addf
55|   // CHECK: math.exp2
56|   // CHECK: arith.addf
57| 
58|   // CHECK: ttg.async_wait
59|   // CHECK: ttg.local_load
60|   // CHECK: ttg.async_copy_global_to_local
61| 
62|   // CHECK: tt.dot
63|   // CHECK: tt.reduce
64| 
65|   // CHECK: ttg.async_wait
66|   // CHECK: ttg.local_load
67|   // CHECK: ttg.async_copy_global_to_local
68| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 69-70
```mlir
69|   // CHECK: scf.yield
70| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 71-88
```mlir
71|   tt.func @chained_dots_with_ops_in_between(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg1: tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>, %arg2: i32, %arg3: i32) -> tensor<128x16xf32, #mma> {
72|     %c0_i32 = arith.constant 0 : i32
73|     %cst = arith.constant dense<0.000000e+00> : tensor<128x16xf32, #mma>
74|     %0 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<1x16x!tt.ptr<f16>, #blocked>
75|     %1 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
76|     %2 = tt.expand_dims %1 {axis = 1 : i32} : tensor<64xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<64x1xi32, #blocked>
77|     %3 = tt.broadcast %0 : tensor<1x16x!tt.ptr<f16>, #blocked> -> tensor<64x16x!tt.ptr<f16>, #blocked>
78|     %4 = tt.broadcast %2 : tensor<64x1xi32, #blocked> -> tensor<64x16xi32, #blocked>
79|     %5 = tt.addptr %3, %4 : tensor<64x16x!tt.ptr<f16>, #blocked>, tensor<64x16xi32, #blocked>
80|     %6 = scf.for %arg5 = %c0_i32 to %arg2 step %arg3 iter_args(%arg6 = %cst) -> (tensor<128x16xf32, #mma>)  : i32 {
81|       %7 = tt.load %5 : tensor<64x16x!tt.ptr<f16>, #blocked>
82|       %8 = ttg.convert_layout %7 : tensor<64x16xf16, #blocked> -> tensor<64x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>
83|       %9 = tt.load %5 : tensor<64x16x!tt.ptr<f16>, #blocked>
84|       %10 = ttg.convert_layout %9 : tensor<64x16xf16, #blocked> -> tensor<64x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>
85|       %11 = tt.dot %arg1, %8, %cst : tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * tensor<64x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>> -> tensor<128x16xf32, #mma>
86|       %12 = "tt.reduce"(%11) <{axis = 1 : i32}> ({
87|       ^bb0(%arg8: f32, %arg9: f32):
88|         %20 = arith.maxnumf %arg8, %arg9 : f32
```
**EN:** This function-oriented block defines or enters `chained_dots_with_ops_in_between`. Within it, the test exercises tt.func, constants, tensor broadcasting, masked or vectorized loads, layout conversions, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `chained_dots_with_ops_in_between` 为核心。测试在其中演示 tt.func、常量、张量广播、带掩码或向量化的加载、布局转换，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 89-103
```mlir
 89|         tt.reduce.return %20 : f32
 90|       }) : (tensor<128x16xf32, #mma>) -> tensor<128xf32, #ttg.slice<{dim = 1, parent = #mma}>>
 91|       %14 = tt.expand_dims %12 {axis = 1 : i32} : tensor<128xf32, #ttg.slice<{dim = 1, parent = #mma}>> -> tensor<128x1xf32, #mma>
 92|       %15 = tt.broadcast %14 : tensor<128x1xf32, #mma> -> tensor<128x16xf32, #mma>
 93|       // Split here since %15 is used twice
 94|       %16 = arith.addf %11, %15 : tensor<128x16xf32, #mma>
 95|       %17 = math.exp2 %15 : tensor<128x16xf32, #mma>
 96|       %18 = arith.addf %16, %17 : tensor<128x16xf32, #mma>
 97|       %19 = tt.dot %arg1, %10, %18 : tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * tensor<64x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>> -> tensor<128x16xf32, #mma>
 98|       scf.yield %19 : tensor<128x16xf32, #mma>
 99|     }
100|     tt.return %6#0 : tensor<128x16xf32, #mma>
101|   }
102| }
103| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining floating-point additions, reductions, shape expansion, tensor broadcasting, math.exp2.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 浮点加法、归约、形状扩展、张量广播、math.exp2。

### Lines 104-104
```mlir
104| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 105-107
```mlir
105| 
106| #blocked = #ttg.blocked<{sizePerThread = [8, 1], threadsPerWarp = [8, 8], warpsPerCTA = [1, 4], order = [0, 1]}>
107| #mma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [4, 1], instrShape = [32, 32, 16], isTransposed = true}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 108-108
```mlir
108| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 109-112
```mlir
109|   // CHECK-LABEL: tt.func @chained_dots_with_loop_carried_partial_result
110| 
111|   // Similar to the previous test but we take the max of the reduce over all iterations (loop carried) so expect a split after the maximum
112| 
```
**EN:** This function-oriented block defines or enters `chained_dots_with_loop_carried_partial_result`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `chained_dots_with_loop_carried_partial_result` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 113-130
```mlir
113|   // CHECK: scf.for
114| 
115|   // CHECK: tt.dot
116|   // CHECK: arith.mulf
117| 
118|   // CHECK: ttg.async_wait
119|   // CHECK: ttg.local_load
120|   // CHECK: ttg.async_copy_global_to_local
121| 
122|   // CHECK: tt.dot
123|   // CHECK: tt.reduce
124|   // CHECK: arith.maxnumf
125| 
126|   // CHECK: ttg.async_wait
127|   // CHECK: ttg.local_load
128|   // CHECK: ttg.async_copy_global_to_local
129| 
130|   // CHECK: scf.yield
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 131-148
```mlir
131| 
132|   tt.func @chained_dots_with_loop_carried_partial_result(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg1: tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>, %arg2: i32, %arg3: i32, %arg101: tensor<128xf32, #ttg.slice<{dim = 1, parent = #mma}>>) -> tensor<128x16xf32, #mma> {
133|     %c0_i32 = arith.constant 0 : i32
134|     %cst = arith.constant dense<0.000000e+00> : tensor<128x16xf32, #mma>
135|     %0 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<1x16x!tt.ptr<f16>, #blocked>
136|     %1 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
137|     %2 = tt.expand_dims %1 {axis = 1 : i32} : tensor<64xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<64x1xi32, #blocked>
138|     %3 = tt.broadcast %0 : tensor<1x16x!tt.ptr<f16>, #blocked> -> tensor<64x16x!tt.ptr<f16>, #blocked>
139|     %4 = tt.broadcast %2 : tensor<64x1xi32, #blocked> -> tensor<64x16xi32, #blocked>
140|     %5 = tt.addptr %3, %4 : tensor<64x16x!tt.ptr<f16>, #blocked>, tensor<64x16xi32, #blocked>
141|     %6:2 = scf.for %arg4 = %c0_i32 to %arg2 step %arg3 iter_args(%arg5 = %cst, %arg100 = %arg101) -> (tensor<128x16xf32, #mma>, tensor<128xf32, #ttg.slice<{dim = 1, parent = #mma}>>)  : i32 {
142|       %7 = tt.load %5 : tensor<64x16x!tt.ptr<f16>, #blocked>
143|       %8 = ttg.convert_layout %7 : tensor<64x16xf16, #blocked> -> tensor<64x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>
144|       %9 = tt.load %5 : tensor<64x16x!tt.ptr<f16>, #blocked>
145|       %10 = ttg.convert_layout %9 : tensor<64x16xf16, #blocked> -> tensor<64x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>
146|       %11 = tt.dot %arg1, %8, %cst : tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * tensor<64x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>> -> tensor<128x16xf32, #mma>
147|       %12 = "tt.reduce"(%11) <{axis = 1 : i32}> ({
148|       ^bb0(%arg6: f32, %arg7: f32):
```
**EN:** This function-oriented block defines or enters `chained_dots_with_loop_carried_partial_result`. Within it, the test exercises tt.func, constants, tensor broadcasting, masked or vectorized loads, layout conversions, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `chained_dots_with_loop_carried_partial_result` 为核心。测试在其中演示 tt.func、常量、张量广播、带掩码或向量化的加载、布局转换，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 149-163
```mlir
149|         %21 = arith.maxnumf %arg6, %arg7 : f32
150|         tt.reduce.return %21 : f32
151|       }) : (tensor<128x16xf32, #mma>) -> tensor<128xf32, #ttg.slice<{dim = 1, parent = #mma}>>
152|       %24 = arith.maxnumf %12, %arg100 :tensor<128xf32, #ttg.slice<{dim = 1, parent = #mma}>>
153|       // Split here since %24 is used twice
154|       %13 = tt.expand_dims %24 {axis = 1 : i32} : tensor<128xf32, #ttg.slice<{dim = 1, parent = #mma}>> -> tensor<128x1xf32, #mma>
155|       %14 = tt.broadcast %13 : tensor<128x1xf32, #mma> -> tensor<128x16xf32, #mma>
156|       %15 = arith.mulf %14, %11 : tensor<128x16xf32, #mma>
157|       %18 = tt.dot %arg1, %10, %15 : tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * tensor<64x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>> -> tensor<128x16xf32, #mma>
158|       scf.yield %18, %24 : tensor<128x16xf32, #mma>, tensor<128xf32, #ttg.slice<{dim = 1, parent = #mma}>>
159|     }
160|     tt.return %6 : tensor<128x16xf32, #mma>
161|   }
162| }
163| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining arith.maxnumf, reductions, shape expansion, tensor broadcasting, arith.mulf.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 arith.maxnumf、归约、形状扩展、张量广播、arith.mulf。

### Lines 164-164
```mlir
164| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 165-167
```mlir
165| 
166| #blocked = #ttg.blocked<{sizePerThread = [8, 1], threadsPerWarp = [8, 8], warpsPerCTA = [1, 8], order = [0, 1]}>
167| #mma = #ttg.amd_mfma<{version = 3, warpsPerCTA = [8, 1], instrShape = [16, 16, 16], isTransposed = true}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 168-168
```mlir
168| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 169-173
```mlir
169|   // CHECK-LABEL: tt.func @chained_dots_with_load_bias_in_between
170| 
171|   // Similar to the previous test but load bias tensor bewteen 2 dots
172|   // We expect the unstreamable load can be kept after pipelining
173| 
```
**EN:** This function-oriented block defines or enters `chained_dots_with_load_bias_in_between`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `chained_dots_with_load_bias_in_between` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 174-182
```mlir
174|   // CHECK: scf.for
175|   // CHECK: tt.dot
176|   // CHECK: ttg.async_copy_global_to_local
177|   // CHECK: tt.dot
178|   // CHECK: ttg.async_wait
179|   // CHECK: ttg.local_load
180|   // CHECK: tt.load
181|   // CHECK: scf.yield
182| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 183-200
```mlir
183|   tt.func @chained_dots_with_load_bias_in_between(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg1: tensor<256x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>, %arg2: i64 {tt.divisibility = 16 : i32}, %arg3: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg4: i32) -> tensor<256x64xf32, #mma> {
184|     %c0_i32 = arith.constant 0 : i32
185|     %c1_i32 = arith.constant 1 : i32
186|     %c64_i32 = arith.constant 64 : i32
187|     %cst = arith.constant dense<0.000000e+00> : tensor<256x64xf32, #mma>
188|     %0 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
189|     %1 = tt.expand_dims %0 {axis = 1 : i32} : tensor<64xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<64x1xi32, #blocked>
190|     %2 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<64x64x!tt.ptr<f16>, #blocked>
191|     %3 = tt.broadcast %1 : tensor<64x1xi32, #blocked> -> tensor<64x64xi32, #blocked>
192|     %4 = tt.addptr %2, %3 : tensor<64x64x!tt.ptr<f16>, #blocked>, tensor<64x64xi32, #blocked>
193|     %5 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
194|     %6 = tt.splat %arg3 : !tt.ptr<f16> -> tensor<256x64x!tt.ptr<f16>, #blocked>
195|     %7 = scf.for %arg5 = %c0_i32 to %arg4 step %c1_i32 iter_args(%arg6 = %cst) -> (tensor<256x64xf32, #mma>)  : i32 {
196|       %8 = tt.load %4 : tensor<64x64x!tt.ptr<f16>, #blocked>
197|       %9 = ttg.convert_layout %8 : tensor<64x64xf16, #blocked> -> tensor<64x64xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>
198|       %10 = tt.dot %arg1, %9, %cst : tensor<256x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>> * tensor<64x64xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>> -> tensor<256x64xf32, #mma>
199|       %11 = arith.muli %arg5, %c64_i32 : i32
200|       %12 = tt.splat %11 : i32 -> tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
```
**EN:** This function-oriented block defines or enters `chained_dots_with_load_bias_in_between`. Within it, the test exercises constants, broadcasted scalars or pointers, tt.func, lane/block index ranges, shape expansion, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `chained_dots_with_load_bias_in_between` 为核心。测试在其中演示 常量、广播后的标量或指针、tt.func、lane/block 索引范围、形状扩展，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 201-216
```mlir
201|       %13 = arith.addi %12, %5 : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
202|       %14 = tt.expand_dims %13 {axis = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x64xi32, #blocked>
203|       %15 = tt.broadcast %14 : tensor<1x64xi32, #blocked> -> tensor<256x64xi32, #blocked>
204|       %bias_ptr = tt.addptr %6, %15 : tensor<256x64x!tt.ptr<f16>, #blocked>, tensor<256x64xi32, #blocked>
205|       %bias = tt.load %bias_ptr : tensor<256x64x!tt.ptr<f16>, #blocked>
206|       %bias_mma = ttg.convert_layout %bias : tensor<256x64xf16, #blocked> -> tensor<256x64xf16, #mma>
207|       %bias_f32 = arith.extf %bias_mma : tensor<256x64xf16, #mma> to tensor<256x64xf32, #mma>
208|       %dot_bias = arith.addf %10, %bias_f32 : tensor<256x64xf32, #mma>
209|       %21 = arith.truncf %dot_bias : tensor<256x64xf32, #mma> to tensor<256x64xf16, #mma>
210|       %22 = ttg.convert_layout %21 : tensor<256x64xf16, #mma> -> tensor<256x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>
211|       %23 = tt.dot %22, %9, %arg6 : tensor<256x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>> * tensor<64x64xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>> -> tensor<256x64xf32, #mma>
212|       scf.yield %23 : tensor<256x64xf32, #mma>
213|     }
214|     tt.return %7 : tensor<256x64xf32, #mma>
215|   }
216| }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining layout conversions, integer additions, shape expansion, tensor broadcasting, pointer arithmetic.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 布局转换、整数加法、形状扩展、张量广播、指针算术。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `-tritonamdgpu-schedule-loops="num_stages=4"`, `-tritonamdgpu-pipeline="use_async_copy=1"`, `-canonicalize`
- **CN:** 主要 pass 选项：`-split-input-file`，`-tritonamdgpu-schedule-loops="num_stages=4"`，`-tritonamdgpu-pipeline="use_async_copy=1"`，`-canonicalize`
- **EN:** Dominant operations include `arith.constant`, `tt.broadcast`, `tt.func`, `ttg.convert_layout`, `tt.dot`, `tt.expand_dims`, `tt.load`, `tt.splat`, `tt.make_range`, `tt.addptr`.
- **CN:** 主要操作包括 `arith.constant`、`tt.broadcast`、`tt.func`、`ttg.convert_layout`、`tt.dot`、`tt.expand_dims`、`tt.load`、`tt.splat`、`tt.make_range`、`tt.addptr`。
- **EN:** The file contains 3 independently testable section(s). Check styles used: CHECK x44, CHECK-LABEL x4, CHECK-COUNT x1. Important labels include tt.func @direct_chained_dots, tt.func @chained_dots_with_ops_in_between, tt.func @chained_dots_with_loop_carried_partial_result, tt.func @chained_dots_with_load_bias_in_between. It validates that the selected pass pipeline rewrites each labeled test case into the checked form.
- **CN:** 该文件包含 3 个可独立测试的分段。使用的检查类型：CHECK ×44，CHECK-LABEL ×4，CHECK-COUNT ×1。 关键标签包括 tt.func @direct_chained_dots，tt.func @chained_dots_with_ops_in_between，tt.func @chained_dots_with_loop_carried_partial_result，tt.func @chained_dots_with_load_bias_in_between。 它验证所选 pass 流水线会把每个带标签的测试用例重写成检查所要求的形式。

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