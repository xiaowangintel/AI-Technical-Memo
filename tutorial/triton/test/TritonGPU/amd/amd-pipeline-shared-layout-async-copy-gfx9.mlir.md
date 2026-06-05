# amd-pipeline-shared-layout-async-copy-gfx9.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/amd/amd-pipeline-shared-layout-async-copy-gfx9.mlir`
- **EN:** Test for the MLIR/Triton pipeline `-tritonamdgpu-schedule-loops="num_stages=2", -tritonamdgpu-pipeline="use_async_copy=1", -canonicalize` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `-tritonamdgpu-schedule-loops="num_stages=2", -tritonamdgpu-pipeline="use_async_copy=1", -canonicalize` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -split-input-file -tritonamdgpu-schedule-loops="num_stages=2" -tritonamdgpu-pipeline="use_async_copy=1" -canonicalize | FileCheck %s`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -split-input-file -tritonamdgpu-schedule-loops="num_stages=2" -tritonamdgpu-pipeline="use_async_copy=1" -canonicalize | FileCheck %s`

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
1| // RUN: triton-opt %s -split-input-file -tritonamdgpu-schedule-loops="num_stages=2" -tritonamdgpu-pipeline="use_async_copy=1" -canonicalize | FileCheck %s
2| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -split-input-file -tritonamdgpu-schedule-loops="num_stages=2" -tritonamdgpu-pipeline="use_async_copy=1" -canonicalize | FileCheck %s` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -split-input-file -tritonamdgpu-schedule-loops="num_stages=2" -tritonamdgpu-pipeline="use_async_copy=1" -canonicalize | FileCheck %s`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 3-3
```mlir
3| #blocked1 = #ttg.blocked<{sizePerThread = [2, 1], threadsPerWarp = [4, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 4-4
```mlir
4| // CHECK: #shared = {{.*}}vec = 1, {{.*}} order = [1, 0]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 5-5
```mlir
5| #mma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [4, 1], instrShape = [16, 16, 4], isTransposed = true}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 6-6
```mlir
6| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 7-7
```mlir
7|   // CHECK-LABEL: async_copy_shared_vec2_clamp_to_vec1
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: async_copy_shared_vec2_clamp_to_vec1 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: async_copy_shared_vec2_clamp_to_vec1 这样的标签用于锚定匹配范围。

### Lines 8-10
```mlir
 8|   tt.func @async_copy_shared_vec2_clamp_to_vec1(%arg0: tensor<16x32x!tt.ptr<f32>, #blocked1> {tt.contiguity = dense<[1, 2]> : tensor<2xi32>, tt.divisibility = dense<16> : tensor<2xi32>},
 9|                 %arg1: tensor<32x32xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 1}>>,
10|                 %lb: i32, %ub: i32, %step: i32) -> tensor<16x32xf32, #mma> {
```
**EN:** This function-oriented block defines or enters `async_copy_shared_vec2_clamp_to_vec1`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_copy_shared_vec2_clamp_to_vec1` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 11-23
```mlir
11|     // CHECK: ttg.async_copy_global_to_local {{.*}} -> <16x32xf32, #shared, #smem, mutable>
12|     %cst = arith.constant dense<32> : tensor<16x32xi32, #blocked1>
13|     %cst_acc = arith.constant dense<0.000000e+00> : tensor<16x32xf32, #mma>
14|     %result = scf.for %iv = %lb to %ub step %step iter_args(%acc = %cst_acc) -> (tensor<16x32xf32, #mma>) : i32 {
15|       %a = tt.load %arg0 : tensor<16x32x!tt.ptr<f32>, #blocked1>
16|       %a_dot = ttg.convert_layout %a : tensor<16x32xf32, #blocked1> -> tensor<16x32xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 1}>>
17|       %c = tt.dot %a_dot, %arg1, %acc : tensor<16x32xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 1}>> * tensor<32x32xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 1}>> -> tensor<16x32xf32, #mma>
18|       scf.yield %c : tensor<16x32xf32, #mma>
19|     }
20|     tt.return %result : tensor<16x32xf32, #mma>
21|   }
22| }
23| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining constants, structured loops, masked or vectorized loads, layout conversions, dot-product or MMA-style math.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 常量、结构化循环、带掩码或向量化的加载、布局转换、点积或 MMA 风格计算。

### Lines 24-24
```mlir
24| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 25-26
```mlir
25| 
26| // Test with #blocked layout (sizePerThread = [1, 1]) for the 32x32 load
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 27-27
```mlir
27| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [2, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 28-28
```mlir
28| // CHECK: #shared = {{.*}} order = [1, 0]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 29-29
```mlir
29| #mma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [4, 1], instrShape = [16, 16, 4], isTransposed = false}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 30-30
```mlir
30| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 31-31
```mlir
31|   // CHECK-LABEL: async_copy_shared_layout_vec1_order
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: async_copy_shared_layout_vec1_order anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: async_copy_shared_layout_vec1_order 这样的标签用于锚定匹配范围。

### Lines 32-34
```mlir
32|   tt.func @async_copy_shared_layout_vec1_order(%arg0: tensor<32x32x!tt.ptr<f32>, #blocked> {tt.contiguity = dense<[1, 1]> : tensor<2xi32>, tt.divisibility = dense<16> : tensor<2xi32>},
33|                 %arg1: tensor<16x32xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 1}>>,
34|                 %lb: i32, %ub: i32, %step: i32) -> tensor<16x32xf32, #mma> {
```
**EN:** This function-oriented block defines or enters `async_copy_shared_layout_vec1_order`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_copy_shared_layout_vec1_order` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 35-46
```mlir
35|     // CHECK: ttg.async_copy_global_to_local {{.*}} -> <32x32xf32, #shared, #smem, mutable>
36|     %cst_acc = arith.constant dense<0.000000e+00> : tensor<16x32xf32, #mma>
37|     %result = scf.for %iv = %lb to %ub step %step iter_args(%acc = %cst_acc) -> (tensor<16x32xf32, #mma>) : i32 {
38|       %b = tt.load %arg0 : tensor<32x32x!tt.ptr<f32>, #blocked>
39|       %b_dot = ttg.convert_layout %b : tensor<32x32xf32, #blocked> -> tensor<32x32xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 1}>>
40|       %c = tt.dot %arg1, %b_dot, %acc : tensor<16x32xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 1}>> * tensor<32x32xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 1}>> -> tensor<16x32xf32, #mma>
41|       scf.yield %c : tensor<16x32xf32, #mma>
42|     }
43|     tt.return %result : tensor<16x32xf32, #mma>
44|   }
45| }
46| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining constants, structured loops, masked or vectorized loads, layout conversions, dot-product or MMA-style math.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 常量、结构化循环、带掩码或向量化的加载、布局转换、点积或 MMA 风格计算。

### Lines 47-47
```mlir
47| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 48-51
```mlir
48| 
49| // For sizePerThrad=[1, 1] threadsPerWarp=[1, 64] order=[1, 0] and dim1=64 the registers will be contigious along dim0 which is the *non* contig dimension.
50| // Check that we correctly follow the memory order which will be the lane order instead of the register order.
51| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 52-52
```mlir
52| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 64], warpsPerCTA = [1, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 53-53
```mlir
53| // CHECK: #shared = {{.*}} order = [1, 0]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 54-54
```mlir
54| #mma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [1, 1], instrShape = [16, 16, 4], isTransposed = true}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 55-55
```mlir
55| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 56-56
```mlir
56|   // CHECK-LABEL: async_copy_lanes_cover_contig_dim
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: async_copy_lanes_cover_contig_dim anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: async_copy_lanes_cover_contig_dim 这样的标签用于锚定匹配范围。

### Lines 57-60
```mlir
57|   tt.func @async_copy_lanes_cover_contig_dim(
58|               %arg0: tensor<16x64x!tt.ptr<f32>, #blocked> {tt.contiguity = dense<[1, 1]> : tensor<2xi32>, tt.divisibility = dense<16> : tensor<2xi32>},
59|               %arg1: tensor<64x16xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 1}>>,
60|               %lb: i32, %ub: i32, %step: i32) -> tensor<16x16xf32, #mma> {
```
**EN:** This function-oriented block defines or enters `async_copy_lanes_cover_contig_dim`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_copy_lanes_cover_contig_dim` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 61-72
```mlir
61|     // CHECK: ttg.async_copy_global_to_local {{.*}} -> <16x64xf32, #shared, #smem, mutable>
62|     %cst_acc = arith.constant dense<0.000000e+00> : tensor<16x16xf32, #mma>
63|     %result = scf.for %iv = %lb to %ub step %step iter_args(%acc = %cst_acc) -> (tensor<16x16xf32, #mma>) : i32 {
64|       %a = tt.load %arg0 : tensor<16x64x!tt.ptr<f32>, #blocked>
65|       %a_dot = ttg.convert_layout %a : tensor<16x64xf32, #blocked> -> tensor<16x64xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 1}>>
66|       %c = tt.dot %a_dot, %arg1, %acc : tensor<16x64xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 1}>> * tensor<64x16xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 1}>> -> tensor<16x16xf32, #mma>
67|       scf.yield %c : tensor<16x16xf32, #mma>
68|     } {tt.scheduled_max_stage = 1 : i32}
69|     tt.return %result : tensor<16x16xf32, #mma>
70|   }
71| }
72| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining constants, structured loops, masked or vectorized loads, layout conversions, dot-product or MMA-style math.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 常量、结构化循环、带掩码或向量化的加载、布局转换、点积或 MMA 风格计算。

### Lines 73-73
```mlir
73| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 74-75
```mlir
74| 
75| // If sizePerThread is > 1 in the non contig dim we still need to choose the actual memory order.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 76-76
```mlir
76| #blocked = #ttg.blocked<{sizePerThread = [4, 1], threadsPerWarp = [1, 64], warpsPerCTA = [1, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 77-77
```mlir
77| // CHECK: #shared = {{.*}} order = [1, 0]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 78-78
```mlir
78| #mma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [1, 1], instrShape = [16, 16, 4], isTransposed = true}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 79-79
```mlir
79| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 80-80
```mlir
80|   // CHECK-LABEL: async_copy_sizeperthread_in_noncontig_dim_not_vectorized
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: async_copy_sizeperthread_in_noncontig_dim_not_vectorized anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: async_copy_sizeperthread_in_noncontig_dim_not_vectorized 这样的标签用于锚定匹配范围。

### Lines 81-84
```mlir
81|   tt.func @async_copy_sizeperthread_in_noncontig_dim_not_vectorized(
82|               %arg0: tensor<16x64x!tt.ptr<f32>, #blocked> {tt.contiguity = dense<[1, 1]> : tensor<2xi32>, tt.divisibility = dense<16> : tensor<2xi32>},
83|               %arg1: tensor<64x16xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 1}>>,
84|               %lb: i32, %ub: i32, %step: i32) -> tensor<16x16xf32, #mma> {
```
**EN:** This function-oriented block defines or enters `async_copy_sizeperthread_in_noncontig_dim_not_vectorized`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_copy_sizeperthread_in_noncontig_dim_not_vectorized` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 85-95
```mlir
85|     // CHECK: ttg.async_copy_global_to_local {{.*}} -> <16x64xf32, #shared, #smem, mutable>
86|     %cst_acc = arith.constant dense<0.000000e+00> : tensor<16x16xf32, #mma>
87|     %result = scf.for %iv = %lb to %ub step %step iter_args(%acc = %cst_acc) -> (tensor<16x16xf32, #mma>) : i32 {
88|       %a = tt.load %arg0 : tensor<16x64x!tt.ptr<f32>, #blocked>
89|       %a_dot = ttg.convert_layout %a : tensor<16x64xf32, #blocked> -> tensor<16x64xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 1}>>
90|       %c = tt.dot %a_dot, %arg1, %acc : tensor<16x64xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 1}>> * tensor<64x16xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 1}>> -> tensor<16x16xf32, #mma>
91|       scf.yield %c : tensor<16x16xf32, #mma>
92|     } {tt.scheduled_max_stage = 1 : i32}
93|     tt.return %result : tensor<16x16xf32, #mma>
94|   }
95| }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining constants, structured loops, masked or vectorized loads, layout conversions, dot-product or MMA-style math.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 常量、结构化循环、带掩码或向量化的加载、布局转换、点积或 MMA 风格计算。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `-tritonamdgpu-schedule-loops="num_stages=2"`, `-tritonamdgpu-pipeline="use_async_copy=1"`, `-canonicalize`
- **CN:** 主要 pass 选项：`-split-input-file`，`-tritonamdgpu-schedule-loops="num_stages=2"`，`-tritonamdgpu-pipeline="use_async_copy=1"`，`-canonicalize`
- **EN:** Dominant operations include `tt.func`, `arith.constant`, `module`, `scf.for`, `tt.load`, `ttg.convert_layout`, `tt.dot`, `scf.yield`, `tt.return`.
- **CN:** 主要操作包括 `tt.func`、`arith.constant`、`module`、`scf.for`、`tt.load`、`ttg.convert_layout`、`tt.dot`、`scf.yield`、`tt.return`。
- **EN:** The file contains 3 independently testable section(s). Check styles used: CHECK x8, CHECK-LABEL x4. Important labels include async_copy_shared_vec2_clamp_to_vec1, async_copy_shared_layout_vec1_order, async_copy_lanes_cover_contig_dim, async_copy_sizeperthread_in_noncontig_dim_not_vectorized. It validates that the selected pass pipeline rewrites each labeled test case into the checked form.
- **CN:** 该文件包含 3 个可独立测试的分段。使用的检查类型：CHECK ×8，CHECK-LABEL ×4。 关键标签包括 async_copy_shared_vec2_clamp_to_vec1，async_copy_shared_layout_vec1_order，async_copy_lanes_cover_contig_dim，async_copy_sizeperthread_in_noncontig_dim_not_vectorized。 它验证所选 pass 流水线会把每个带标签的测试用例重写成检查所要求的形式。

## Dependencies / 依赖关系

- **EN:** This test depends on Triton/MLIR textual IR parsing, the pass pipeline named in the RUN lines, and the dialect/type definitions referenced in the file.
- **CN:** 该测试依赖 Triton/MLIR 文本 IR 解析、RUN 行中指定的 pass 流水线，以及文件中引用的方言/类型定义。
- **EN:** `ttg`: TritonGPU layout, memory, and GPU-specific operations.
- **CN:** `ttg`：TritonGPU 布局、内存和 GPU 专用操作。
- **EN:** `tt`: Triton high-level tensor/pointer operations.
- **CN:** `tt`：Triton 高层张量/指针操作。
- **EN:** `scf`: Structured control flow for loops and conditionals.
- **CN:** `scf`：用于循环与条件分支的结构化控制流。
- **EN:** `arith`: MLIR arithmetic dialect for scalar/tensor math.
- **CN:** `arith`：用于标量/张量运算的 MLIR arithmetic 方言。
- **EN:** `FileCheck` is used to match required textual patterns after rewriting/lowering.
- **CN:** `FileCheck` 用于在重写/lower 之后匹配必须出现的文本模式。