# prefetch.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/prefetch.mlir`
- **EN:** Test for the MLIR/Triton pipeline `-tritongpu-prefetch, -canonicalize` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `-tritongpu-prefetch, -canonicalize` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -split-input-file -tritongpu-prefetch -canonicalize | FileCheck %s --dump-input-context=50`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -split-input-file -tritongpu-prefetch -canonicalize | FileCheck %s --dump-input-context=50`

## Line-by-Line Analysis / 逐行分析

### Lines 1-4
```mlir
1| // RUN: triton-opt %s -split-input-file -tritongpu-prefetch -canonicalize | FileCheck %s --dump-input-context=50
2| 
3| // 4 warps
4| // matmul: 128x32 @ 32x128 -> 128x128
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -split-input-file -tritongpu-prefetch -canonicalize | FileCheck %s --dump-input-context=50` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -split-input-file -tritongpu-prefetch -canonicalize | FileCheck %s --dump-input-context=50`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 5-13
```mlir
 5| #AL = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
 6| #BL = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
 7| #A = #ttg.swizzled_shared<{vec = 2, perPhase = 2, maxPhase = 4, order = [1, 0]}>
 8| #B = #ttg.swizzled_shared<{vec = 2, perPhase = 2, maxPhase = 4, order = [1, 0]}>
 9| #C = #ttg.nvidia_mma<{versionMajor = 2, warpsPerCTA = [4, 1], instrShape = [16, 8]}>
10| #A_OP = #ttg.dot_op<{opIdx = 0, parent = #C, kWidth = 2}>
11| #B_OP = #ttg.dot_op<{opIdx = 1, parent = #C, kWidth = 2}>
12| #smem = #ttg.shared_memory
13| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 14-31
```mlir
14| // CHECK: tt.func @matmul_loop_mixed
15| // CHECK-DAG: %[[A0_PREFETCH_SMEM:.*]] = ttg.memdesc_subslice %[[A0:.*]][0, 0]
16| // CHECK-DAG: %[[A0_PREFETCH:.*]] = ttg.local_load %[[A0_PREFETCH_SMEM]]
17| // CHECK-DAG: %[[A0_CVT:.*]] = tt.fp_to_fp %[[A0_PREFETCH]]
18| // CHECK-DAG: %[[B0_PREFETCH_SMEM:.*]] = ttg.memdesc_subslice %[[B0:.*]][0, 0]
19| // CHECK-DAG: %[[B0_PREFETCH:.*]] = ttg.local_load %[[B0_PREFETCH_SMEM]]
20| // CHECK:     scf.for {{.*}} iter_args({{.*}}, {{.*}}, %[[arg_a0:.*]] = %[[A0]], %[[arg_b0:.*]] = %[[B0]], {{.*}}, %[[a0_prefetch:.*]] = %[[A0_CVT]], %[[b0_prefetch:.*]] = %[[B0_PREFETCH]]
21| // CHECK-DAG:   %[[A_REM_SMEM:.*]] = ttg.memdesc_subslice %[[arg_a0]][0, 16]
22| // CHECK-DAG:   %[[A_REM:.*]] = ttg.local_load %[[A_REM_SMEM]]
23| // CHECK-DAG:   %[[A_REM_CVT:.*]] = tt.fp_to_fp %[[A_REM]]
24| // CHECK-DAG:   %[[B_REM_SMEM:.*]] = ttg.memdesc_subslice %[[arg_b0]][16, 0]
25| // CHECK-DAG:   %[[B_REM:.*]] = ttg.local_load %[[B_REM_SMEM]]
26| // CHECK:       %[[D_FIRST:.*]] = tt.dot %[[a0_prefetch]], %[[b0_prefetch:.*]], {{.*}}
27| // CHECK-DAG:   %[[NEXT_A_PREFETCH_SMEM:.*]] = ttg.memdesc_subslice {{.*}}[0, 0]
28| // CHECK-DAG:   %[[NEXT_A_PREFETCH:.*]] = ttg.local_load %[[NEXT_A_PREFETCH_SMEM]]
29| // CHECK-DAG:   %[[NEXT_A_PREFETCH_CVT:.*]] = tt.fp_to_fp %[[NEXT_A_PREFETCH]]
30| // CHECK-DAG:   %[[NEXT_B_PREFETCH_SMEM:.*]] = ttg.memdesc_subslice {{.*}}[0, 0]
31| // CHECK-DAG:   %[[NEXT_B_PREFETCH:.*]] = ttg.local_load %[[NEXT_B_PREFETCH_SMEM]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 32-33
```mlir
32| // CHECK:       tt.dot %[[A_REM_CVT]], %[[B_REM]], %[[D_FIRST:.*]]
33| // CHECK:     scf.yield {{.*}}, {{.*}}, {{.*}}, {{.*}}, {{.*}}, %[[NEXT_A_PREFETCH_CVT]], %[[NEXT_B_PREFETCH]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 34-34
```mlir
34| module attributes { "ttg.num-warps" = 4 : i32 } {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 35-52
```mlir
35| tt.func @matmul_loop_mixed(%lb : index, %ub : index, %step : index, %A : !tt.ptr<f8E5M2>, %B : !tt.ptr<f16>) -> tensor<128x128xf32, #C>{
36|   %a_ptr_init = tt.splat %A : !tt.ptr<f8E5M2> -> tensor<128x32x!tt.ptr<f8E5M2>, #AL>
37|   %b_ptr_init = tt.splat %B : !tt.ptr<f16> -> tensor<32x128x!tt.ptr<f16>, #BL>
38| 
39|   %a_mask = arith.constant dense<true> : tensor<128x32xi1, #AL>
40|   %a_other = arith.constant dense<0.00e+00> : tensor<128x32xf8E5M2, #AL>
41|   %b_mask = arith.constant dense<true> : tensor<32x128xi1, #BL>
42|   %b_other = arith.constant dense<0.00e+00> : tensor<32x128xf16, #BL>
43|   %c_init = arith.constant dense<0.00e+00> : tensor<128x128xf32, #C>
44| 
45|   %a_off = arith.constant dense<4> : tensor<128x32xi32, #AL>
46|   %b_off = arith.constant dense<4> : tensor<32x128xi32, #BL>
47| 
48|   %a_ = tt.load %a_ptr_init, %a_mask, %a_other : tensor<128x32x!tt.ptr<f8E5M2>, #AL>
49|   %a_init = ttg.local_alloc %a_ : (tensor<128x32xf8E5M2, #AL>) -> !ttg.memdesc<128x32xf8E5M2, #A, #smem>
50|   %b_ = tt.load %b_ptr_init, %b_mask, %b_other : tensor<32x128x!tt.ptr<f16>, #BL>
51|   %b_init = ttg.local_alloc %b_ : (tensor<32x128xf16, #BL>) -> !ttg.memdesc<32x128xf16, #B, #smem>
52| 
```
**EN:** This function-oriented block defines or enters `matmul_loop_mixed`. Within it, the test exercises constants, tt.func, broadcasted scalars or pointers, masked or vectorized loads, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `matmul_loop_mixed` 为核心。测试在其中演示 常量、tt.func、广播后的标量或指针、带掩码或向量化的加载、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 53-70
```mlir
53|   %loop:5 = scf.for %iv = %lb to %ub step %step iter_args(%a_ptr = %a_ptr_init, %b_ptr = %b_ptr_init, %a = %a_init, %b = %b_init, %prev_c = %c_init) -> (tensor<128x32x!tt.ptr<f8E5M2>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, !ttg.memdesc<128x32xf8E5M2, #A, #smem>, !ttg.memdesc<32x128xf16, #B, #smem>, tensor<128x128xf32, #C>) {
54|     %a_op_ = ttg.local_load %a : !ttg.memdesc<128x32xf8E5M2, #A, #smem> -> tensor<128x32xf8E5M2, #A_OP>
55|     %a_op = tt.fp_to_fp %a_op_ : tensor<128x32xf8E5M2, #A_OP> -> tensor<128x32xf16, #A_OP>
56|     %b_op = ttg.local_load %b : !ttg.memdesc<32x128xf16, #B, #smem> -> tensor<32x128xf16, #B_OP>
57|     %c = tt.dot %a_op, %b_op, %prev_c : tensor<128x32xf16, #A_OP> * tensor<32x128xf16, #B_OP> -> tensor<128x128xf32, #C>
58| 
59|     %next_a_ptr = tt.addptr %a_ptr, %a_off : tensor<128x32x!tt.ptr<f8E5M2>, #AL>, tensor<128x32xi32, #AL>
60|     %next_b_ptr = tt.addptr %b_ptr, %b_off : tensor<32x128x!tt.ptr<f16>, #BL>, tensor<32x128xi32, #BL>
61|     %next_a_ = tt.load %next_a_ptr, %a_mask, %a_other : tensor<128x32x!tt.ptr<f8E5M2>, #AL>
62|     %next_a = ttg.local_alloc %next_a_ : (tensor<128x32xf8E5M2, #AL>) -> !ttg.memdesc<128x32xf8E5M2, #A, #smem>
63|     %next_b_ = tt.load %next_b_ptr, %b_mask, %b_other : tensor<32x128x!tt.ptr<f16>, #BL>
64|     %next_b = ttg.local_alloc %b_ : (tensor<32x128xf16, #BL>) -> !ttg.memdesc<32x128xf16, #B, #smem>
65| 
66|     scf.yield %next_a_ptr, %next_b_ptr, %next_a, %next_b, %c : tensor<128x32x!tt.ptr<f8E5M2>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, !ttg.memdesc<128x32xf8E5M2, #A, #smem>, !ttg.memdesc<32x128xf16, #B, #smem>, tensor<128x128xf32, #C>
67|   }
68|   tt.return %loop#4 : tensor<128x128xf32, #C>
69| }
70| }  // end module
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining local/shared memory loads, pointer arithmetic, masked or vectorized loads, shared/local memory allocation, structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 本地/共享内存加载、指针算术、带掩码或向量化的加载、共享/本地内存分配、结构化循环。

### Lines 71-73
```mlir
71| 
72| // 4 warps
73| // matmul: 128x16 @ 16x128 -> 128x128
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 74-87
```mlir
74| // CHECK: tt.func @matmul_loop_mixed_4warps
75| // CHECK-DAG: %[[A0_PREFETCH_SMEM:.*]] = ttg.memdesc_subslice %[[A0:.*]][0, 0]
76| // CHECK-DAG: %[[A0_PREFETCH:.*]] = ttg.local_load %[[A0_PREFETCH_SMEM]]
77| // CHECK-DAG: %[[A0_CVT:.*]] = tt.fp_to_fp %[[A0_PREFETCH]]
78| // CHECK-DAG: %[[B0_PREFETCH_SMEM:.*]] = ttg.memdesc_subslice %[[B0:.*]][0, 0]
79| // CHECK-DAG: %[[B0_PREFETCH:.*]] = ttg.local_load %[[B0_PREFETCH_SMEM]]
80| // CHECK:     scf.for {{.*}} iter_args({{.*}}, {{.*}}, {{.*}}, %[[a0_prefetch:.*]] = %[[A0_CVT]], %[[b0_prefetch:.*]] = %[[B0_PREFETCH]]
81| // CHECK-DAG:   %[[NEXT_A_PREFETCH_SMEM:.*]] = ttg.memdesc_subslice {{.*}}[0, 0]
82| // CHECK-DAG:   %[[NEXT_A_PREFETCH:.*]] = ttg.local_load %[[NEXT_A_PREFETCH_SMEM]]
83| // CHECK-DAG:   %[[NEXT_A_PREFETCH_CVT:.*]] = tt.fp_to_fp %[[NEXT_A_PREFETCH]]
84| // CHECK-DAG:   %[[NEXT_B_PREFETCH_SMEM:.*]] = ttg.memdesc_subslice {{.*}}[0, 0]
85| // CHECK-DAG:   %[[NEXT_B_PREFETCH:.*]] = ttg.local_load %[[NEXT_B_PREFETCH_SMEM]]
86| // CHECK:       tt.dot %[[a0_prefetch]], %[[b0_prefetch]], {{.*}}
87| // CHECK:     scf.yield {{.*}}, {{.*}}, {{.*}}, %[[NEXT_A_PREFETCH_CVT]], %[[NEXT_B_PREFETCH]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 88-88
```mlir
88| module attributes { "ttg.num-warps" = 4 : i32 } {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 89-106
```mlir
 89| tt.func @matmul_loop_mixed_4warps(%lb : index, %ub : index, %step : index, %A : !tt.ptr<f8E5M2>, %B : !tt.ptr<f16>) -> tensor<128x128xf32, #C>{
 90|   %a_ptr_init = tt.splat %A : !tt.ptr<f8E5M2> -> tensor<128x16x!tt.ptr<f8E5M2>, #AL>
 91|   %b_ptr_init = tt.splat %B : !tt.ptr<f16> -> tensor<16x128x!tt.ptr<f16>, #BL>
 92| 
 93|   %a_mask = arith.constant dense<true> : tensor<128x16xi1, #AL>
 94|   %a_other = arith.constant dense<0.00e+00> : tensor<128x16xf8E5M2, #AL>
 95|   %b_mask = arith.constant dense<true> : tensor<16x128xi1, #BL>
 96|   %b_other = arith.constant dense<0.00e+00> : tensor<16x128xf16, #BL>
 97|   %c_init = arith.constant dense<0.00e+00> : tensor<128x128xf32, #C>
 98| 
 99|   %a_off = arith.constant dense<4> : tensor<128x16xi32, #AL>
100|   %b_off = arith.constant dense<4> : tensor<16x128xi32, #BL>
101| 
102|   %a_ = tt.load %a_ptr_init, %a_mask, %a_other : tensor<128x16x!tt.ptr<f8E5M2>, #AL>
103|   %a_init = ttg.local_alloc %a_ : (tensor<128x16xf8E5M2, #AL>) -> !ttg.memdesc<128x16xf8E5M2, #A, #smem>
104|   %b_ = tt.load %b_ptr_init, %b_mask, %b_other : tensor<16x128x!tt.ptr<f16>, #BL>
105|   %b_init = ttg.local_alloc %b_ : (tensor<16x128xf16, #BL>) -> !ttg.memdesc<16x128xf16, #B, #smem>
106| 
```
**EN:** This function-oriented block defines or enters `matmul_loop_mixed_4warps`. Within it, the test exercises constants, tt.func, broadcasted scalars or pointers, masked or vectorized loads, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `matmul_loop_mixed_4warps` 为核心。测试在其中演示 常量、tt.func、广播后的标量或指针、带掩码或向量化的加载、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 107-124
```mlir
107|   %loop:5 = scf.for %iv = %lb to %ub step %step iter_args(%a_ptr = %a_ptr_init, %b_ptr = %b_ptr_init, %a = %a_init, %b = %b_init, %prev_c = %c_init) -> (tensor<128x16x!tt.ptr<f8E5M2>, #AL>, tensor<16x128x!tt.ptr<f16>, #BL>, !ttg.memdesc<128x16xf8E5M2, #A, #smem>, !ttg.memdesc<16x128xf16, #B, #smem>, tensor<128x128xf32, #C>) {
108|     %a_op_ = ttg.local_load %a : !ttg.memdesc<128x16xf8E5M2, #A, #smem> -> tensor<128x16xf8E5M2, #A_OP>
109|     %a_op = tt.fp_to_fp %a_op_ : tensor<128x16xf8E5M2, #A_OP> -> tensor<128x16xf16, #A_OP>
110|     %b_op = ttg.local_load %b : !ttg.memdesc<16x128xf16, #B, #smem> -> tensor<16x128xf16, #B_OP>
111|     %c = tt.dot %a_op, %b_op, %prev_c : tensor<128x16xf16, #A_OP> * tensor<16x128xf16, #B_OP> -> tensor<128x128xf32, #C>
112| 
113|     %next_a_ptr = tt.addptr %a_ptr, %a_off : tensor<128x16x!tt.ptr<f8E5M2>, #AL>, tensor<128x16xi32, #AL>
114|     %next_b_ptr = tt.addptr %b_ptr, %b_off : tensor<16x128x!tt.ptr<f16>, #BL>, tensor<16x128xi32, #BL>
115|     %next_a_ = tt.load %next_a_ptr, %a_mask, %a_other : tensor<128x16x!tt.ptr<f8E5M2>, #AL>
116|     %next_a = ttg.local_alloc %next_a_ : (tensor<128x16xf8E5M2, #AL>) -> !ttg.memdesc<128x16xf8E5M2, #A, #smem>
117|     %next_b_ = tt.load %next_b_ptr, %b_mask, %b_other : tensor<16x128x!tt.ptr<f16>, #BL>
118|     %next_b = ttg.local_alloc %b_ : (tensor<16x128xf16, #BL>) -> !ttg.memdesc<16x128xf16, #B, #smem>
119| 
120|     scf.yield %next_a_ptr, %next_b_ptr, %next_a, %next_b, %c : tensor<128x16x!tt.ptr<f8E5M2>, #AL>, tensor<16x128x!tt.ptr<f16>, #BL>, !ttg.memdesc<128x16xf8E5M2, #A, #smem>, !ttg.memdesc<16x128xf16, #B, #smem>, tensor<128x128xf32, #C>
121|   }
122|   tt.return %loop#4 : tensor<128x128xf32, #C>
123| }
124| }  // end module
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining local/shared memory loads, pointer arithmetic, masked or vectorized loads, shared/local memory allocation, structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 本地/共享内存加载、指针算术、带掩码或向量化的加载、共享/本地内存分配、结构化循环。

### Lines 125-134
```mlir
125| 
126| #AL_3D = #ttg.blocked<{sizePerThread = [1, 1, 4], threadsPerWarp = [2, 4, 4], warpsPerCTA = [1, 4, 1], order = [2, 0, 1]}>
127| #BL_3D = #ttg.blocked<{sizePerThread = [1, 1, 4], threadsPerWarp = [2, 4, 4], warpsPerCTA = [1, 4, 1], order = [2, 0, 1]}>
128| #A_3D = #ttg.swizzled_shared<{vec = 2, perPhase = 2, maxPhase = 4, order = [2, 0, 1]}>
129| #B_3D = #ttg.swizzled_shared<{vec = 2, perPhase = 2, maxPhase = 4, order = [2, 0, 1]}>
130| #C_3D = #ttg.nvidia_mma<{versionMajor = 2, warpsPerCTA = [1, 4, 1], instrShape = [1, 16, 8]}>
131| #A_OP_3D = #ttg.dot_op<{opIdx = 0, parent = #C_3D, kWidth = 2}>
132| #B_OP_3D = #ttg.dot_op<{opIdx = 1, parent = #C_3D, kWidth = 2}>
133| 
134| // matmul: 8x128x16 @ 8x16x128 -> 8x128x128
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 135-148
```mlir
135| // CHECK: tt.func @matmul_3D_loop_mixed
136| // CHECK-DAG: %[[A0_PREFETCH_SMEM:.*]] = ttg.memdesc_subslice %[[A0:.*]][0, 0, 0]
137| // CHECK-DAG: %[[A0_PREFETCH:.*]] = ttg.local_load %[[A0_PREFETCH_SMEM]]
138| // CHECK-DAG: %[[A0_CVT:.*]] = tt.fp_to_fp %[[A0_PREFETCH]]
139| // CHECK-DAG: %[[B0_PREFETCH_SMEM:.*]] = ttg.memdesc_subslice %[[B0:.*]][0, 0, 0]
140| // CHECK-DAG: %[[B0_PREFETCH:.*]] = ttg.local_load %[[B0_PREFETCH_SMEM]]
141| // CHECK:     scf.for {{.*}} iter_args({{.*}}, {{.*}}, {{.*}}, %[[a0_prefetch:.*]] = %[[A0_CVT]], %[[b0_prefetch:.*]] = %[[B0_PREFETCH]]
142| // CHECK-DAG:   %[[NEXT_A_PREFETCH_SMEM:.*]] = ttg.memdesc_subslice {{.*}}[0, 0, 0]
143| // CHECK-DAG:   %[[NEXT_A_PREFETCH:.*]] = ttg.local_load %[[NEXT_A_PREFETCH_SMEM]]
144| // CHECK-DAG:   %[[NEXT_A_PREFETCH_CVT:.*]] = tt.fp_to_fp %[[NEXT_A_PREFETCH]]
145| // CHECK-DAG:   %[[NEXT_B_PREFETCH_SMEM:.*]] = ttg.memdesc_subslice {{.*}}[0, 0, 0]
146| // CHECK-DAG:   %[[NEXT_B_PREFETCH:.*]] = ttg.local_load %[[NEXT_B_PREFETCH_SMEM]]
147| // CHECK:       tt.dot %[[a0_prefetch]], %[[b0_prefetch]], {{.*}}
148| // CHECK:     scf.yield {{.*}}, {{.*}}, {{.*}}, %[[NEXT_A_PREFETCH_CVT]], %[[NEXT_B_PREFETCH]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 149-149
```mlir
149| module attributes { "ttg.num-warps" = 4 : i32 } {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 150-167
```mlir
150| tt.func @matmul_3D_loop_mixed(%lb : index, %ub : index, %step : index, %A : !tt.ptr<f8E5M2>, %B : !tt.ptr<f16>) -> tensor<8x128x128xf32, #C_3D>{
151|   %a_ptr_init = tt.splat %A : !tt.ptr<f8E5M2> -> tensor<8x128x16x!tt.ptr<f8E5M2>, #AL_3D>
152|   %b_ptr_init = tt.splat %B : !tt.ptr<f16> -> tensor<8x16x128x!tt.ptr<f16>, #BL_3D>
153| 
154|   %a_mask = arith.constant dense<true> : tensor<8x128x16xi1, #AL_3D>
155|   %a_other = arith.constant dense<0.00e+00> : tensor<8x128x16xf8E5M2, #AL_3D>
156|   %b_mask = arith.constant dense<true> : tensor<8x16x128xi1, #BL_3D>
157|   %b_other = arith.constant dense<0.00e+00> : tensor<8x16x128xf16, #BL_3D>
158|   %c_init = arith.constant dense<0.00e+00> : tensor<8x128x128xf32, #C_3D>
159| 
160|   %a_off = arith.constant dense<4> : tensor<8x128x16xi32, #AL_3D>
161|   %b_off = arith.constant dense<4> : tensor<8x16x128xi32, #BL_3D>
162| 
163|   %a_ = tt.load %a_ptr_init, %a_mask, %a_other : tensor<8x128x16x!tt.ptr<f8E5M2>, #AL_3D>
164|   %a_init = ttg.local_alloc %a_ : (tensor<8x128x16xf8E5M2, #AL_3D>) -> !ttg.memdesc<8x128x16xf8E5M2, #A_3D, #smem>
165|   %b_ = tt.load %b_ptr_init, %b_mask, %b_other : tensor<8x16x128x!tt.ptr<f16>, #BL_3D>
166|   %b_init = ttg.local_alloc %b_ : (tensor<8x16x128xf16, #BL_3D>) -> !ttg.memdesc<8x16x128xf16, #B_3D, #smem>
167| 
```
**EN:** This function-oriented block defines or enters `matmul_3D_loop_mixed`. Within it, the test exercises constants, tt.func, broadcasted scalars or pointers, masked or vectorized loads, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `matmul_3D_loop_mixed` 为核心。测试在其中演示 常量、tt.func、广播后的标量或指针、带掩码或向量化的加载、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 168-185
```mlir
168|   %loop:5 = scf.for %iv = %lb to %ub step %step iter_args(%a_ptr = %a_ptr_init, %b_ptr = %b_ptr_init, %a = %a_init, %b = %b_init, %prev_c = %c_init) -> (tensor<8x128x16x!tt.ptr<f8E5M2>, #AL_3D>, tensor<8x16x128x!tt.ptr<f16>, #BL_3D>, !ttg.memdesc<8x128x16xf8E5M2, #A_3D, #smem>, !ttg.memdesc<8x16x128xf16, #B_3D, #smem>, tensor<8x128x128xf32, #C_3D>) {
169|     %a_op_ = ttg.local_load %a : !ttg.memdesc<8x128x16xf8E5M2, #A_3D, #smem> -> tensor<8x128x16xf8E5M2, #A_OP_3D>
170|     %a_op = tt.fp_to_fp %a_op_ : tensor<8x128x16xf8E5M2, #A_OP_3D> -> tensor<8x128x16xf16, #A_OP_3D>
171|     %b_op = ttg.local_load %b : !ttg.memdesc<8x16x128xf16, #B_3D, #smem> -> tensor<8x16x128xf16, #B_OP_3D>
172|     %c = tt.dot %a_op, %b_op, %prev_c : tensor<8x128x16xf16, #A_OP_3D> * tensor<8x16x128xf16, #B_OP_3D> -> tensor<8x128x128xf32, #C_3D>
173| 
174|     %next_a_ptr = tt.addptr %a_ptr, %a_off : tensor<8x128x16x!tt.ptr<f8E5M2>, #AL_3D>, tensor<8x128x16xi32, #AL_3D>
175|     %next_b_ptr = tt.addptr %b_ptr, %b_off : tensor<8x16x128x!tt.ptr<f16>, #BL_3D>, tensor<8x16x128xi32, #BL_3D>
176|     %next_a_ = tt.load %next_a_ptr, %a_mask, %a_other : tensor<8x128x16x!tt.ptr<f8E5M2>, #AL_3D>
177|     %next_a = ttg.local_alloc %next_a_ : (tensor<8x128x16xf8E5M2, #AL_3D>) -> !ttg.memdesc<8x128x16xf8E5M2, #A_3D, #smem>
178|     %next_b_ = tt.load %next_b_ptr, %b_mask, %b_other : tensor<8x16x128x!tt.ptr<f16>, #BL_3D>
179|     %next_b = ttg.local_alloc %b_ : (tensor<8x16x128xf16, #BL_3D>) -> !ttg.memdesc<8x16x128xf16, #B_3D, #smem>
180| 
181|     scf.yield %next_a_ptr, %next_b_ptr, %next_a, %next_b, %c : tensor<8x128x16x!tt.ptr<f8E5M2>, #AL_3D>, tensor<8x16x128x!tt.ptr<f16>, #BL_3D>, !ttg.memdesc<8x128x16xf8E5M2, #A_3D, #smem>, !ttg.memdesc<8x16x128xf16, #B_3D, #smem>, tensor<8x128x128xf32, #C_3D>
182|   }
183|   tt.return %loop#4 : tensor<8x128x128xf32, #C_3D>
184| }
185| }  // end module
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining local/shared memory loads, pointer arithmetic, masked or vectorized loads, shared/local memory allocation, structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 本地/共享内存加载、指针算术、带掩码或向量化的加载、共享/本地内存分配、结构化循环。

### Lines 186-187
```mlir
186| 
187| // matmul: 8x128x32 @ 8x32x128 -> 8x128x128
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 188-205
```mlir
188| // CHECK: tt.func @matmul_3D_loop_mixed2
189| // CHECK-DAG: %[[A0_PREFETCH_SMEM:.*]] = ttg.memdesc_subslice %[[A0:.*]][0, 0, 0]
190| // CHECK-DAG: %[[A0_PREFETCH:.*]] = ttg.local_load %[[A0_PREFETCH_SMEM]]
191| // CHECK-DAG: %[[A0_CVT:.*]] = tt.fp_to_fp %[[A0_PREFETCH]]
192| // CHECK-DAG: %[[B0_PREFETCH_SMEM:.*]] = ttg.memdesc_subslice %[[B0:.*]][0, 0, 0]
193| // CHECK-DAG: %[[B0_PREFETCH:.*]] = ttg.local_load %[[B0_PREFETCH_SMEM]]
194| // CHECK:     scf.for {{.*}} iter_args({{.*}}, {{.*}}, %[[arg_a0:.*]] = %[[A0]], %[[arg_b0:.*]] = %[[B0]], {{.*}}, %[[a0_prefetch:.*]] = %[[A0_CVT]], %[[b0_prefetch:.*]] = %[[B0_PREFETCH]]
195| // CHECK-DAG:   %[[A_REM_SMEM:.*]] = ttg.memdesc_subslice %[[arg_a0]][0, 0, 16]
196| // CHECK-DAG:   %[[A_REM:.*]] = ttg.local_load %[[A_REM_SMEM]]
197| // CHECK-DAG:   %[[A_REM_CVT:.*]] = tt.fp_to_fp %[[A_REM]]
198| // CHECK-DAG:   %[[B_REM_SMEM:.*]] = ttg.memdesc_subslice %[[arg_b0]][0, 16, 0]
199| // CHECK-DAG:   %[[B_REM:.*]] = ttg.local_load %[[B_REM_SMEM]]
200| // CHECK:       %[[D_FIRST:.*]] = tt.dot %[[a0_prefetch]], %[[b0_prefetch:.*]], {{.*}}
201| // CHECK-DAG:   %[[NEXT_A_PREFETCH_SMEM:.*]] = ttg.memdesc_subslice {{.*}}[0, 0, 0]
202| // CHECK-DAG:   %[[NEXT_A_PREFETCH:.*]] = ttg.local_load %[[NEXT_A_PREFETCH_SMEM]]
203| // CHECK-DAG:   %[[NEXT_A_PREFETCH_CVT:.*]] = tt.fp_to_fp %[[NEXT_A_PREFETCH]]
204| // CHECK-DAG:   %[[NEXT_B_PREFETCH_SMEM:.*]] = ttg.memdesc_subslice {{.*}}[0, 0, 0]
205| // CHECK-DAG:   %[[NEXT_B_PREFETCH:.*]] = ttg.local_load %[[NEXT_B_PREFETCH_SMEM]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 206-207
```mlir
206| // CHECK:       tt.dot %[[A_REM_CVT]], %[[B_REM]], %[[D_FIRST:.*]]
207| // CHECK:     scf.yield {{.*}}, {{.*}}, {{.*}}, {{.*}}, {{.*}}, %[[NEXT_A_PREFETCH_CVT]], %[[NEXT_B_PREFETCH]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 208-208
```mlir
208| module attributes { "ttg.num-warps" = 4 : i32 } {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 209-226
```mlir
209| tt.func @matmul_3D_loop_mixed2(%lb : index, %ub : index, %step : index, %A : !tt.ptr<f8E5M2>, %B : !tt.ptr<f16>) -> tensor<8x128x128xf32, #C_3D>{
210|   %a_ptr_init = tt.splat %A : !tt.ptr<f8E5M2> -> tensor<8x128x32x!tt.ptr<f8E5M2>, #AL_3D>
211|   %b_ptr_init = tt.splat %B : !tt.ptr<f16> -> tensor<8x32x128x!tt.ptr<f16>, #BL_3D>
212| 
213|   %a_mask = arith.constant dense<true> : tensor<8x128x32xi1, #AL_3D>
214|   %a_other = arith.constant dense<0.00e+00> : tensor<8x128x32xf8E5M2, #AL_3D>
215|   %b_mask = arith.constant dense<true> : tensor<8x32x128xi1, #BL_3D>
216|   %b_other = arith.constant dense<0.00e+00> : tensor<8x32x128xf16, #BL_3D>
217|   %c_init = arith.constant dense<0.00e+00> : tensor<8x128x128xf32, #C_3D>
218| 
219|   %a_off = arith.constant dense<4> : tensor<8x128x32xi32, #AL_3D>
220|   %b_off = arith.constant dense<4> : tensor<8x32x128xi32, #BL_3D>
221| 
222|   %a_ = tt.load %a_ptr_init, %a_mask, %a_other : tensor<8x128x32x!tt.ptr<f8E5M2>, #AL_3D>
223|   %a_init = ttg.local_alloc %a_ : (tensor<8x128x32xf8E5M2, #AL_3D>) -> !ttg.memdesc<8x128x32xf8E5M2, #A_3D, #smem>
224|   %b_ = tt.load %b_ptr_init, %b_mask, %b_other : tensor<8x32x128x!tt.ptr<f16>, #BL_3D>
225|   %b_init = ttg.local_alloc %b_ : (tensor<8x32x128xf16, #BL_3D>) -> !ttg.memdesc<8x32x128xf16, #B_3D, #smem>
226| 
```
**EN:** This function-oriented block defines or enters `matmul_3D_loop_mixed2`. Within it, the test exercises constants, tt.func, broadcasted scalars or pointers, masked or vectorized loads, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `matmul_3D_loop_mixed2` 为核心。测试在其中演示 常量、tt.func、广播后的标量或指针、带掩码或向量化的加载、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 227-244
```mlir
227|   %loop:5 = scf.for %iv = %lb to %ub step %step iter_args(%a_ptr = %a_ptr_init, %b_ptr = %b_ptr_init, %a = %a_init, %b = %b_init, %prev_c = %c_init) -> (tensor<8x128x32x!tt.ptr<f8E5M2>, #AL_3D>, tensor<8x32x128x!tt.ptr<f16>, #BL_3D>, !ttg.memdesc<8x128x32xf8E5M2, #A_3D, #smem>, !ttg.memdesc<8x32x128xf16, #B_3D, #smem>, tensor<8x128x128xf32, #C_3D>) {
228|     %a_op_ = ttg.local_load %a : !ttg.memdesc<8x128x32xf8E5M2, #A_3D, #smem> -> tensor<8x128x32xf8E5M2, #A_OP_3D>
229|     %a_op = tt.fp_to_fp %a_op_ : tensor<8x128x32xf8E5M2, #A_OP_3D> -> tensor<8x128x32xf16, #A_OP_3D>
230|     %b_op = ttg.local_load %b : !ttg.memdesc<8x32x128xf16, #B_3D, #smem> -> tensor<8x32x128xf16, #B_OP_3D>
231|     %c = tt.dot %a_op, %b_op, %prev_c : tensor<8x128x32xf16, #A_OP_3D> * tensor<8x32x128xf16, #B_OP_3D> -> tensor<8x128x128xf32, #C_3D>
232| 
233|     %next_a_ptr = tt.addptr %a_ptr, %a_off : tensor<8x128x32x!tt.ptr<f8E5M2>, #AL_3D>, tensor<8x128x32xi32, #AL_3D>
234|     %next_b_ptr = tt.addptr %b_ptr, %b_off : tensor<8x32x128x!tt.ptr<f16>, #BL_3D>, tensor<8x32x128xi32, #BL_3D>
235|     %next_a_ = tt.load %next_a_ptr, %a_mask, %a_other : tensor<8x128x32x!tt.ptr<f8E5M2>, #AL_3D>
236|     %next_a = ttg.local_alloc %next_a_ : (tensor<8x128x32xf8E5M2, #AL_3D>) -> !ttg.memdesc<8x128x32xf8E5M2, #A_3D, #smem>
237|     %next_b_ = tt.load %next_b_ptr, %b_mask, %b_other : tensor<8x32x128x!tt.ptr<f16>, #BL_3D>
238|     %next_b = ttg.local_alloc %b_ : (tensor<8x32x128xf16, #BL_3D>) -> !ttg.memdesc<8x32x128xf16, #B_3D, #smem>
239| 
240|     scf.yield %next_a_ptr, %next_b_ptr, %next_a, %next_b, %c : tensor<8x128x32x!tt.ptr<f8E5M2>, #AL_3D>, tensor<8x32x128x!tt.ptr<f16>, #BL_3D>, !ttg.memdesc<8x128x32xf8E5M2, #A_3D, #smem>, !ttg.memdesc<8x32x128xf16, #B_3D, #smem>, tensor<8x128x128xf32, #C_3D>
241|   }
242|   tt.return %loop#4 : tensor<8x128x128xf32, #C_3D>
243| }
244| }  // end module
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining local/shared memory loads, pointer arithmetic, masked or vectorized loads, shared/local memory allocation, structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 本地/共享内存加载、指针算术、带掩码或向量化的加载、共享/本地内存分配、结构化循环。

### Lines 245-250
```mlir
245| 
246| // CHECK: tt.func @matmul_loop_yield_no_operand
247| // CHECK: scf.for
248| // CHECK: scf.if
249| // CHECK: tt.store
250| // CHECK-NOT: scf.yield
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 251-253
```mlir
251| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
252| #blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
253| #mma = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [2, 2], instrShape = [16, 8]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 254-254
```mlir
254| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:86", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:86`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:86`。

### Lines 255-272
```mlir
255|   tt.func @matmul_loop_yield_no_operand(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg2: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg3: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg4: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg5: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg6: i32 {tt.divisibility = 16 : i32}, %arg7: i32, %arg8: i32 {tt.divisibility = 16 : i32}, %arg9: i32, %arg10: i32) {
256|     %cst = arith.constant dense<0.000000e+00> : tensor<32x32xf32, #mma>
257|     %c32_i32 = arith.constant 32 : i32
258|     %c31_i32 = arith.constant 31 : i32
259|     %c0_i32 = arith.constant 0 : i32
260|     %0 = arith.muli %arg9, %arg10 : i32
261|     %1 = arith.addi %arg8, %c31_i32 : i32
262|     %2 = arith.divsi %1, %c32_i32 : i32
263|     %3 = arith.addi %0, %c31_i32 : i32
264|     %4 = arith.divsi %3, %c32_i32 : i32
265|     %5 = arith.muli %1, %4 : i32
266|     %6 = tt.get_program_id x : i32
267|     %7 = tt.get_num_programs x : i32
268|     %8 = tt.splat %arg3 : !tt.ptr<f16> -> tensor<32x32x!tt.ptr<f16>, #blocked>
269|     scf.for %arg11 = %6 to %5 step %7  : i32 {
270|       %9 = arith.divsi %arg11, %4 : i32
271|       %10 = arith.remsi %9, %2 : i32
272|       %11 = tt.load %8 : tensor<32x32x!tt.ptr<f16>, #blocked>
```
**EN:** This function-oriented block defines or enters `matmul_loop_yield_no_operand`. Within it, the test exercises constants, arith.divsi, tt.func, integer multiplications, integer additions, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `matmul_loop_yield_no_operand` 为核心。测试在其中演示 常量、arith.divsi、tt.func、整数乘法、整数加法，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 273-290
```mlir
273|       %12 = tt.load %8 : tensor<32x32x!tt.ptr<f16>, #blocked>
274|       %13 = ttg.convert_layout %12 : tensor<32x32xf16, #blocked> -> tensor<32x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
275|       %14 = ttg.convert_layout %11 : tensor<32x32xf16, #blocked> -> tensor<32x32xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>
276|       %15 = tt.dot %13, %14, %cst, inputPrecision = tf32 : tensor<32x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * tensor<32x32xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>> -> tensor<32x32xf32, #mma>
277|       %16 = arith.cmpi sgt, %10, %c0_i32 : i32
278|       %17 = scf.if %16 -> (tensor<32x32xf32, #mma>) {
279|         %21 = tt.dot %13, %14, %15, inputPrecision = tf32 : tensor<32x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * tensor<32x32xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>> -> tensor<32x32xf32, #mma>
280|         scf.yield %21 : tensor<32x32xf32, #mma>
281|       } else {
282|         scf.yield %15 : tensor<32x32xf32, #mma>
283|       }
284|       %18 = tt.splat %arg5 : !tt.ptr<f16> -> tensor<32x32x!tt.ptr<f16>, #blocked1>
285|       %19 = arith.truncf %17 : tensor<32x32xf32, #mma> to tensor<32x32xf16, #mma>
286|       %20 = ttg.convert_layout %19 : tensor<32x32xf16, #mma> -> tensor<32x32xf16, #blocked1>
287|       tt.store %18, %20 : tensor<32x32x!tt.ptr<f16>, #blocked1>
288|     }
289|     tt.return
290|   }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining layout conversions, dot-product or MMA-style math, loop/if yielded values, masked or vectorized loads, integer comparisons.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 布局转换、点积或 MMA 风格计算、循环/分支产出值、带掩码或向量化的加载、整数比较。

### Lines 291-292
```mlir
291| }
292| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 293-293
```mlir
293| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 294-301
```mlir
294| 
295| #A_RING = #ttg.swizzled_shared<{vec = 2, perPhase = 2, maxPhase = 4, order = [1, 0]}>
296| #B_RING = #ttg.swizzled_shared<{vec = 2, perPhase = 2, maxPhase = 4, order = [1, 0]}>
297| #C_RING = #ttg.nvidia_mma<{versionMajor = 2, warpsPerCTA = [4, 1], instrShape = [16, 8]}>
298| #A_RING_OP = #ttg.dot_op<{opIdx = 0, parent = #C_RING, kWidth = 2}>
299| #B_RING_OP = #ttg.dot_op<{opIdx = 1, parent = #C_RING, kWidth = 2}>
300| #smem = #ttg.shared_memory
301| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 302-319
```mlir
302| // CHECK-LABEL: tt.func @split_pipelined_mmav2_loads
303| // CHECK-DAG: %[[A_VIEW:.+]] = ttg.memdesc_index %[[A_BUF:.+]][
304| // CHECK-DAG: %[[WAIT:.+]] = ttg.async_wait
305| // CHECK-DAG: %[[B_VIEW:.+]] = ttg.memdesc_index %[[B_BUF:.+]][
306| // CHECK-DAG: %[[A0_SMEM:.+]] = ttg.memdesc_subslice %[[A_VIEW]][0, 0]
307| // CHECK-DAG: %[[A0:.+]] = ttg.local_load %[[A0_SMEM]] token %[[WAIT]]
308| // CHECK-DAG: %[[B0_SMEM:.+]] = ttg.memdesc_subslice %[[B_VIEW]][0, 0]
309| // CHECK-DAG: %[[B0:.+]] = ttg.local_load %[[B0_SMEM]] token %[[WAIT]]
310| // CHECK: %[[LOOP:.+]]:8 = scf.for {{.+}} iter_args(%[[IDX_ARG:.+]] = %[[C0:.+]], %[[ACC_ARG:.+]] = %{{.+}}, %[[A_VIEW_ARG:.+]] = %[[A_VIEW]], %[[B_VIEW_ARG:.+]] = %[[B_VIEW]], %[[A_WAIT_ARG:.+]] = %[[WAIT]], %[[B_WAIT_ARG:.+]] = %[[WAIT]], %[[A0_ARG:.+]] = %[[A0]], %[[B0_ARG:.+]] = %[[B0]])
311| // CHECK-DAG: %[[A1_SMEM:.+]] = ttg.memdesc_subslice %[[A_VIEW_ARG]][0, 16]
312| // CHECK-DAG: %[[A1:.+]] = ttg.local_load %[[A1_SMEM]] token %[[A_WAIT_ARG]]
313| // CHECK-DAG: %[[B1_SMEM:.+]] = ttg.memdesc_subslice %[[B_VIEW_ARG]][16, 0]
314| // CHECK-DAG: %[[B1:.+]] = ttg.local_load %[[B1_SMEM]] token %[[B_WAIT_ARG]]
315| // CHECK: %[[DOT0:.+]] = tt.dot %[[A0_ARG]], %[[B0_ARG]], %[[ACC_ARG]]
316| // CHECK: ttg.memdesc_index %[[A_BUF]][
317| // CHECK: ttg.memdesc_index %[[B_BUF]][
318| // CHECK: %[[NEXT_A_HEAD_SMEM:.+]] = ttg.memdesc_subslice %{{.+}}[0, 0] : !ttg.memdesc<128x32xf16, #shared, #smem, mutable> -> !ttg.memdesc<128x16xf16, #shared, #smem, mutable, 128x32>
319| // CHECK: ttg.local_load %[[NEXT_A_HEAD_SMEM]] token %{{.+}} : !ttg.memdesc<128x16xf16, #shared, #smem, mutable, 128x32> -> tensor<128x16xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: tt.func @split_pipelined_mmav2_loads anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: tt.func @split_pipelined_mmav2_loads 这样的标签用于锚定匹配范围。

### Lines 320-322
```mlir
320| // CHECK: %[[NEXT_B_HEAD_SMEM:.+]] = ttg.memdesc_subslice %{{.+}}[0, 0] : !ttg.memdesc<32x128xf16, #shared, #smem, mutable> -> !ttg.memdesc<16x128xf16, #shared, #smem, mutable, 32x128>
321| // CHECK: ttg.local_load %[[NEXT_B_HEAD_SMEM]] token %{{.+}} : !ttg.memdesc<16x128xf16, #shared, #smem, mutable, 32x128> -> tensor<16x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>
322| // CHECK: tt.dot %[[A1]], %[[B1]], %[[DOT0]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 323-323
```mlir
323| module attributes { "ttg.num-warps" = 4 : i32 } {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 324-341
```mlir
324| tt.func @split_pipelined_mmav2_loads(%lb : index, %ub : index, %step : index, %tok0 : !ttg.async.token, %tok1 : !ttg.async.token) -> tensor<128x128xf32, #C_RING> {
325|   %c0_i32 = arith.constant 0 : i32
326|   %c1_i32 = arith.constant 1 : i32
327|   %c3_i32 = arith.constant 3 : i32
328|   %cst = arith.constant dense<0.00e+00> : tensor<128x128xf32, #C_RING>
329|   %a = ttg.local_alloc : () -> !ttg.memdesc<3x128x32xf16, #A_RING, #smem, mutable>
330|   %b = ttg.local_alloc : () -> !ttg.memdesc<3x32x128xf16, #B_RING, #smem, mutable>
331|   %loop:2 = scf.for %iv = %lb to %ub step %step iter_args(%idx = %c0_i32, %acc = %cst) -> (i32, tensor<128x128xf32, #C_RING>) {
332|     %idx_p1 = arith.addi %idx, %c1_i32 : i32
333|     %idx_cmp = arith.cmpi sge, %idx_p1, %c3_i32 : i32
334|     %idx_next = arith.select %idx_cmp, %c0_i32, %idx_p1 : i32
335|     %wait = ttg.async_wait %tok0, %tok1 {num = 4 : i32}
336|     %a_view = ttg.memdesc_index %a[%idx_next] : !ttg.memdesc<3x128x32xf16, #A_RING, #smem, mutable> -> !ttg.memdesc<128x32xf16, #A_RING, #smem, mutable>
337|     %a_val = ttg.local_load %a_view token %wait : !ttg.memdesc<128x32xf16, #A_RING, #smem, mutable> -> tensor<128x32xf16, #A_RING_OP>
338|     %b_view = ttg.memdesc_index %b[%idx_next] : !ttg.memdesc<3x32x128xf16, #B_RING, #smem, mutable> -> !ttg.memdesc<32x128xf16, #B_RING, #smem, mutable>
339|     %b_val = ttg.local_load %b_view token %wait : !ttg.memdesc<32x128xf16, #B_RING, #smem, mutable> -> tensor<32x128xf16, #B_RING_OP>
340|     %acc_next = tt.dot %a_val, %b_val, %acc : tensor<128x32xf16, #A_RING_OP> * tensor<32x128xf16, #B_RING_OP> -> tensor<128x128xf32, #C_RING>
341|     scf.yield %idx_next, %acc_next : i32, tensor<128x128xf32, #C_RING>
```
**EN:** This function-oriented block defines or enters `split_pipelined_mmav2_loads`. Within it, the test exercises constants, tt.func, shared/local memory allocation, ttg.memdesc_index, local/shared memory loads, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `split_pipelined_mmav2_loads` 为核心。测试在其中演示 常量、tt.func、共享/本地内存分配、ttg.memdesc_index、本地/共享内存加载，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 342-346
```mlir
342|   }
343|   tt.return %loop#1 : tensor<128x128xf32, #C_RING>
344| }
345| }  // end module
346| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 347-363
```mlir
347| // CHECK-LABEL: tt.func @prefetch_pipelined_loop_args
348| // CHECK-DAG: %[[INIT_WAIT:.+]] = ttg.async_wait %arg3, %arg4 {num = 4 : i32}
349| // CHECK-DAG: %[[A0_SMEM:.+]] = ttg.memdesc_subslice %[[A0:.+]][0, 0]
350| // CHECK-DAG: %[[A0_PREFETCH:.+]] = ttg.local_load %[[A0_SMEM]] token %[[INIT_WAIT]]
351| // CHECK-DAG: %[[B0_SMEM:.+]] = ttg.memdesc_subslice %[[B0:.+]][0, 0]
352| // CHECK-DAG: %[[B0_PREFETCH:.+]] = ttg.local_load %[[B0_SMEM]] token %[[INIT_WAIT]]
353| // CHECK: %[[LOOP:.+]]:7 = scf.for {{.+}} iter_args(%[[IDX_ARG:.+]] = %[[C0:.+]], %[[A_ARG:.+]] = %[[A0]], %[[B_ARG:.+]] = %[[B0]], %[[WAIT_ARG:.+]] = %[[INIT_WAIT]], %[[ACC_ARG:.+]] = %{{.+}}, %[[A_PREFETCH_ARG:.+]] = %[[A0_PREFETCH]], %[[B_PREFETCH_ARG:.+]] = %[[B0_PREFETCH]])
354| // CHECK: %[[WAIT_NEXT:.+]] = ttg.async_wait %arg3, %arg4 {num = 4 : i32}
355| // CHECK-DAG: %[[A1_SMEM:.+]] = ttg.memdesc_subslice %[[A_ARG]][0, 16]
356| // CHECK-DAG: %[[A1:.+]] = ttg.local_load %[[A1_SMEM]] token %[[WAIT_ARG]]
357| // CHECK-DAG: %[[B1_SMEM:.+]] = ttg.memdesc_subslice %[[B_ARG]][16, 0]
358| // CHECK-DAG: %[[B1:.+]] = ttg.local_load %[[B1_SMEM]] token %[[WAIT_ARG]]
359| // CHECK: %{{.+}} = tt.dot %[[A_PREFETCH_ARG]], %[[B_PREFETCH_ARG]], %[[ACC_ARG]]
360| // CHECK-DAG: %[[NEXT_A_SMEM:.+]] = ttg.memdesc_subslice %{{.+}}[0, 0]
361| // CHECK-DAG: %[[NEXT_A_PREFETCH:.+]] = ttg.local_load %[[NEXT_A_SMEM]] token %[[WAIT_NEXT]]
362| // CHECK-DAG: %[[NEXT_B_SMEM:.+]] = ttg.memdesc_subslice %{{.+}}[0, 0]
363| // CHECK-DAG: %[[NEXT_B_PREFETCH:.+]] = ttg.local_load %[[NEXT_B_SMEM]] token %[[WAIT_NEXT]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: tt.func @prefetch_pipelined_loop_args anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: tt.func @prefetch_pipelined_loop_args 这样的标签用于锚定匹配范围。

### Lines 364-364
```mlir
364| module attributes { "ttg.num-warps" = 4 : i32 } {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 365-382
```mlir
365| tt.func @prefetch_pipelined_loop_args(%lb : index, %ub : index, %step : index, %tok0 : !ttg.async.token, %tok1 : !ttg.async.token) -> tensor<128x128xf32, #C_RING> {
366|   %c0_i32 = arith.constant 0 : i32
367|   %c1_i32 = arith.constant 1 : i32
368|   %c3_i32 = arith.constant 3 : i32
369|   %cst = arith.constant dense<0.00e+00> : tensor<128x128xf32, #C_RING>
370|   %a = ttg.local_alloc : () -> !ttg.memdesc<3x128x32xf16, #A_RING, #smem, mutable>
371|   %b = ttg.local_alloc : () -> !ttg.memdesc<3x32x128xf16, #B_RING, #smem, mutable>
372|   %wait0 = ttg.async_wait %tok0, %tok1 {num = 4 : i32}
373|   %a0 = ttg.memdesc_index %a[%c0_i32] : !ttg.memdesc<3x128x32xf16, #A_RING, #smem, mutable> -> !ttg.memdesc<128x32xf16, #A_RING, #smem, mutable>
374|   %b0 = ttg.memdesc_index %b[%c0_i32] : !ttg.memdesc<3x32x128xf16, #B_RING, #smem, mutable> -> !ttg.memdesc<32x128xf16, #B_RING, #smem, mutable>
375|   %loop:7 = scf.for %iv = %lb to %ub step %step iter_args(%idx = %c0_i32, %a_view = %a0, %b_view = %b0, %wait = %wait0, %tok_a = %tok0, %tok_b = %tok1, %acc = %cst) -> (i32, !ttg.memdesc<128x32xf16, #A_RING, #smem, mutable>, !ttg.memdesc<32x128xf16, #B_RING, #smem, mutable>, !ttg.async.token, !ttg.async.token, !ttg.async.token, tensor<128x128xf32, #C_RING>) {
376|     %a_val = ttg.local_load %a_view token %wait : !ttg.memdesc<128x32xf16, #A_RING, #smem, mutable> -> tensor<128x32xf16, #A_RING_OP>
377|     %b_val = ttg.local_load %b_view token %wait : !ttg.memdesc<32x128xf16, #B_RING, #smem, mutable> -> tensor<32x128xf16, #B_RING_OP>
378|     %idx_p1 = arith.addi %idx, %c1_i32 : i32
379|     %idx_cmp = arith.cmpi sge, %idx_p1, %c3_i32 : i32
380|     %idx_next = arith.select %idx_cmp, %c0_i32, %idx_p1 : i32
381|     %wait_next = ttg.async_wait %tok_a, %tok_b {num = 4 : i32}
382|     %a_next = ttg.memdesc_index %a[%idx_next] : !ttg.memdesc<3x128x32xf16, #A_RING, #smem, mutable> -> !ttg.memdesc<128x32xf16, #A_RING, #smem, mutable>
```
**EN:** This function-oriented block defines or enters `prefetch_pipelined_loop_args`. Within it, the test exercises constants, ttg.memdesc_index, tt.func, shared/local memory allocation, async wait synchronization, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `prefetch_pipelined_loop_args` 为核心。测试在其中演示 常量、ttg.memdesc_index、tt.func、共享/本地内存分配、异步等待同步，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 383-390
```mlir
383|     %b_next = ttg.memdesc_index %b[%idx_next] : !ttg.memdesc<3x32x128xf16, #B_RING, #smem, mutable> -> !ttg.memdesc<32x128xf16, #B_RING, #smem, mutable>
384|     %acc_next = tt.dot %a_val, %b_val, %acc : tensor<128x32xf16, #A_RING_OP> * tensor<32x128xf16, #B_RING_OP> -> tensor<128x128xf32, #C_RING>
385|     scf.yield %idx_next, %a_next, %b_next, %wait_next, %tok_a, %tok_b, %acc_next : i32, !ttg.memdesc<128x32xf16, #A_RING, #smem, mutable>, !ttg.memdesc<32x128xf16, #B_RING, #smem, mutable>, !ttg.async.token, !ttg.async.token, !ttg.async.token, tensor<128x128xf32, #C_RING>
386|   }
387|   tt.return %loop#6 : tensor<128x128xf32, #C_RING>
388| }
389| }  // end module
390| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining ttg.memdesc_index, dot-product or MMA-style math, loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 ttg.memdesc_index、点积或 MMA 风格计算、循环/分支产出值、tt.return。

### Lines 391-408
```mlir
391| // CHECK-LABEL: tt.func @prefetch_induction_var_source
392| // CHECK-DAG: %[[LB_I32:.+]] = arith.index_cast %arg0 : index to i32
393| // CHECK-DAG: %[[INIT_WAIT:.+]] = ttg.async_wait %arg3, %arg4 {num = 4 : i32}
394| // CHECK-DAG: %[[A0_VIEW:.+]] = ttg.memdesc_index %{{.+}}[%[[LB_I32]]]
395| // CHECK-DAG: %[[A0_HEAD_SMEM:.+]] = ttg.memdesc_subslice %[[A0_VIEW]][0, 0]
396| // CHECK-DAG: %[[A0_HEAD:.+]] = ttg.local_load %[[A0_HEAD_SMEM]] token %[[INIT_WAIT]]
397| // CHECK-DAG: %[[B0_VIEW:.+]] = ttg.memdesc_index %{{.+}}[%[[LB_I32]]]
398| // CHECK-DAG: %[[B0_HEAD_SMEM:.+]] = ttg.memdesc_subslice %[[B0_VIEW]][0, 0]
399| // CHECK-DAG: %[[B0_HEAD:.+]] = ttg.local_load %[[B0_HEAD_SMEM]] token %[[INIT_WAIT]]
400| // CHECK: %[[LOOP:.+]]:{{[0-9]+}} = scf.for %[[IV:.+]] = %arg0 to %arg1 step %arg2 iter_args({{.*}}%[[A_PREFETCH_ARG:.+]] = %[[A0_HEAD]], %[[B_PREFETCH_ARG:.+]] = %[[B0_HEAD]])
401| // CHECK: %[[WAIT:.+]] = ttg.async_wait %arg3, %arg4 {num = 4 : i32}
402| // CHECK-DAG: %[[IV_NEXT:.+]] = arith.addi %[[IV]], %arg2 : index
403| // CHECK-DAG: %[[IV_NEXT_I32:.+]] = arith.index_cast %[[IV_NEXT]] : index to i32
404| // CHECK: %[[A1_HEAD_SMEM:.+]] = ttg.memdesc_subslice %{{.+}}[0, 0] : !ttg.memdesc<128x32xf16, #shared, #smem, mutable> -> !ttg.memdesc<128x16xf16, #shared, #smem, mutable, 128x32>
405| // CHECK: %[[A1_HEAD:.+]] = ttg.local_load %[[A1_HEAD_SMEM]] token %{{.+}} : !ttg.memdesc<128x16xf16, #shared, #smem, mutable, 128x32> -> tensor<128x16xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
406| // CHECK: %[[B1_HEAD_SMEM:.+]] = ttg.memdesc_subslice %{{.+}}[0, 0] : !ttg.memdesc<32x128xf16, #shared, #smem, mutable> -> !ttg.memdesc<16x128xf16, #shared, #smem, mutable, 32x128>
407| // CHECK: %[[B1_HEAD:.+]] = ttg.local_load %[[B1_HEAD_SMEM]] token %{{.+}} : !ttg.memdesc<16x128xf16, #shared, #smem, mutable, 32x128> -> tensor<16x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>
408| // CHECK: %{{.+}} = tt.dot %{{.+}}, %{{.+}}, %{{.+}}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: tt.func @prefetch_induction_var_source anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: tt.func @prefetch_induction_var_source 这样的标签用于锚定匹配范围。

### Lines 409-409
```mlir
409| module attributes { "ttg.num-warps" = 4 : i32 } {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 410-427
```mlir
410| tt.func @prefetch_induction_var_source(%lb : index, %ub : index, %step : index, %tok0 : !ttg.async.token, %tok1 : !ttg.async.token) -> tensor<128x128xf32, #C_RING> {
411|   %cst = arith.constant dense<0.00e+00> : tensor<128x128xf32, #C_RING>
412|   %a = ttg.local_alloc : () -> !ttg.memdesc<3x128x32xf16, #A_RING, #smem, mutable>
413|   %b = ttg.local_alloc : () -> !ttg.memdesc<3x32x128xf16, #B_RING, #smem, mutable>
414|   %loop = scf.for %iv = %lb to %ub step %step iter_args(%acc = %cst) -> (tensor<128x128xf32, #C_RING>) {
415|     %iv_i32 = arith.index_cast %iv : index to i32
416|     %wait = ttg.async_wait %tok0, %tok1 {num = 4 : i32}
417|     %a_view = ttg.memdesc_index %a[%iv_i32] : !ttg.memdesc<3x128x32xf16, #A_RING, #smem, mutable> -> !ttg.memdesc<128x32xf16, #A_RING, #smem, mutable>
418|     %a_val = ttg.local_load %a_view token %wait : !ttg.memdesc<128x32xf16, #A_RING, #smem, mutable> -> tensor<128x32xf16, #A_RING_OP>
419|     %b_view = ttg.memdesc_index %b[%iv_i32] : !ttg.memdesc<3x32x128xf16, #B_RING, #smem, mutable> -> !ttg.memdesc<32x128xf16, #B_RING, #smem, mutable>
420|     %b_val = ttg.local_load %b_view token %wait : !ttg.memdesc<32x128xf16, #B_RING, #smem, mutable> -> tensor<32x128xf16, #B_RING_OP>
421|     %acc_next = tt.dot %a_val, %b_val, %acc : tensor<128x32xf16, #A_RING_OP> * tensor<32x128xf16, #B_RING_OP> -> tensor<128x128xf32, #C_RING>
422|     scf.yield %acc_next : tensor<128x128xf32, #C_RING>
423|   }
424|   tt.return %loop : tensor<128x128xf32, #C_RING>
425| }
426| }  // end module
427| 
```
**EN:** This function-oriented block defines or enters `prefetch_induction_var_source`. Within it, the test exercises tt.func, shared/local memory allocation, ttg.memdesc_index, local/shared memory loads, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `prefetch_induction_var_source` 为核心。测试在其中演示 tt.func、共享/本地内存分配、ttg.memdesc_index、本地/共享内存加载、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 428-428
```mlir
428| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 429-436
```mlir
429| 
430| #shared_f64 = #ttg.swizzled_shared<{vec = 4, perPhase = 1, maxPhase = 4, order = [1, 0]}>
431| #shared1_f64 = #ttg.swizzled_shared<{vec = 8, perPhase = 1, maxPhase = 2, order = [1, 0]}>
432| #mma_f64 = #ttg.nvidia_mma<{versionMajor = 2, warpsPerCTA = [1, 1], instrShape = [16, 8]}>
433| #a_f64_op = #ttg.dot_op<{opIdx = 0, parent = #mma_f64, kWidth = 1}>
434| #b_f64_op = #ttg.dot_op<{opIdx = 1, parent = #mma_f64, kWidth = 1}>
435| #smem = #ttg.shared_memory
436| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 437-451
```mlir
437| // CHECK-LABEL: tt.func @split_pipelined_mmav2_loads_f64
438| // CHECK-DAG: %[[A_VIEW:.+]] = ttg.memdesc_index %[[A_BUF:.+]][
439| // CHECK-DAG: %[[WAIT:.+]] = ttg.async_wait
440| // CHECK-DAG: %[[B_VIEW:.+]] = ttg.memdesc_index %[[B_BUF:.+]][
441| // CHECK-DAG: %[[A0_SMEM:.+]] = ttg.memdesc_subslice %[[A_VIEW]][0, 0]
442| // CHECK-DAG: %[[A0:.+]] = ttg.local_load %[[A0_SMEM]] token %[[WAIT]]
443| // CHECK-DAG: %[[B0_SMEM:.+]] = ttg.memdesc_subslice %[[B_VIEW]][0, 0]
444| // CHECK-DAG: %[[B0:.+]] = ttg.local_load %[[B0_SMEM]] token %[[WAIT]]
445| // CHECK: %[[LOOP:.+]]:8 = scf.for {{.+}} iter_args(%[[IDX_ARG:.+]] = %[[C0:.+]], %[[ACC_ARG:.+]] = %{{.+}}, %[[A_VIEW_ARG:.+]] = %[[A_VIEW]], %[[B_VIEW_ARG:.+]] = %[[B_VIEW]], %[[A_WAIT_ARG:.+]] = %[[WAIT]], %[[B_WAIT_ARG:.+]] = %[[WAIT]], %[[A0_ARG:.+]] = %[[A0]], %[[B0_ARG:.+]] = %[[B0]])
446| // CHECK-DAG: %[[A1_SMEM:.+]] = ttg.memdesc_subslice %[[A_VIEW_ARG]][0, 8]
447| // CHECK-DAG: %[[A1:.+]] = ttg.local_load %[[A1_SMEM]] token %[[A_WAIT_ARG]]
448| // CHECK-DAG: %[[B1_SMEM:.+]] = ttg.memdesc_subslice %[[B_VIEW_ARG]][8, 0]
449| // CHECK-DAG: %[[B1:.+]] = ttg.local_load %[[B1_SMEM]] token %[[B_WAIT_ARG]]
450| // CHECK: %[[DOT0:.+]] = tt.dot %[[A0_ARG]], %[[B0_ARG]], %[[ACC_ARG]]
451| // CHECK: tt.dot %[[A1]], %[[B1]], %[[DOT0]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: tt.func @split_pipelined_mmav2_loads_f64 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: tt.func @split_pipelined_mmav2_loads_f64 这样的标签用于锚定匹配范围。

### Lines 452-452
```mlir
452| module attributes {ttg.target = "cuda:90", "ttg.num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 453-470
```mlir
453| tt.func @split_pipelined_mmav2_loads_f64(%lb : index, %ub : index, %step : index, %tok0 : !ttg.async.token, %tok1 : !ttg.async.token) -> tensor<16x16xf64, #mma_f64> {
454|   %c0_i32 = arith.constant 0 : i32
455|   %c1_i32 = arith.constant 1 : i32
456|   %c3_i32 = arith.constant 3 : i32
457|   %cst = arith.constant dense<0.00e+00> : tensor<16x16xf64, #mma_f64>
458|   %a = ttg.local_alloc : () -> !ttg.memdesc<3x16x16xf64, #shared_f64, #smem, mutable>
459|   %b = ttg.local_alloc : () -> !ttg.memdesc<3x16x16xf64, #shared1_f64, #smem, mutable>
460|   %loop:2 = scf.for %iv = %lb to %ub step %step iter_args(%idx = %c0_i32, %acc = %cst) -> (i32, tensor<16x16xf64, #mma_f64>) {
461|     %idx_p1 = arith.addi %idx, %c1_i32 : i32
462|     %idx_cmp = arith.cmpi sge, %idx_p1, %c3_i32 : i32
463|     %idx_next = arith.select %idx_cmp, %c0_i32, %idx_p1 : i32
464|     %wait = ttg.async_wait %tok0, %tok1 {num = 4 : i32}
465|     %a_view = ttg.memdesc_index %a[%idx_next] : !ttg.memdesc<3x16x16xf64, #shared_f64, #smem, mutable> -> !ttg.memdesc<16x16xf64, #shared_f64, #smem, mutable>
466|     %a_val = ttg.local_load %a_view token %wait : !ttg.memdesc<16x16xf64, #shared_f64, #smem, mutable> -> tensor<16x16xf64, #a_f64_op>
467|     %b_view = ttg.memdesc_index %b[%idx_next] : !ttg.memdesc<3x16x16xf64, #shared1_f64, #smem, mutable> -> !ttg.memdesc<16x16xf64, #shared1_f64, #smem, mutable>
468|     %b_val = ttg.local_load %b_view token %wait : !ttg.memdesc<16x16xf64, #shared1_f64, #smem, mutable> -> tensor<16x16xf64, #b_f64_op>
469|     %acc_next = tt.dot %a_val, %b_val, %acc, inputPrecision = tf32 : tensor<16x16xf64, #a_f64_op> * tensor<16x16xf64, #b_f64_op> -> tensor<16x16xf64, #mma_f64>
470|     scf.yield %idx_next, %acc_next : i32, tensor<16x16xf64, #mma_f64>
```
**EN:** This function-oriented block defines or enters `split_pipelined_mmav2_loads_f64`. Within it, the test exercises constants, tt.func, shared/local memory allocation, ttg.memdesc_index, local/shared memory loads, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `split_pipelined_mmav2_loads_f64` 为核心。测试在其中演示 常量、tt.func、共享/本地内存分配、ttg.memdesc_index、本地/共享内存加载，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 471-475
```mlir
471|   }
472|   tt.return %loop#1 : tensor<16x16xf64, #mma_f64>
473| }
474| }  // end module
475| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 476-476
```mlir
476| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 477-486
```mlir
477| 
478| #AL = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [8, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
479| #BL = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [1, 64], warpsPerCTA = [4, 1], order = [1, 0]}>
480| #A = #ttg.swizzled_shared<{vec = 2, perPhase = 2, maxPhase = 4, order = [1, 0]}>
481| #B = #ttg.swizzled_shared<{vec = 2, perPhase = 2, maxPhase = 4, order = [1, 0]}>
482| #C = #ttg.amd_mfma<{version = 3, warpsPerCTA = [1, 4], instrShape = [32, 32, 8], isTransposed = false}>
483| #A_OP = #ttg.dot_op<{opIdx = 0, parent = #C, kWidth = 2}>
484| #B_OP = #ttg.dot_op<{opIdx = 1, parent = #C, kWidth = 2}>
485| #smem = #ttg.shared_memory
486| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 487-504
```mlir
487| // CHECK-LABEL: tt.func @matmul_loop_mixed_amd
488| // CHECK-DAG: %[[A0_PREFETCH_SMEM:.*]] = ttg.memdesc_subslice %[[A0:.*]][0, 0]
489| // CHECK-DAG: %[[A0_PREFETCH:.*]] = ttg.local_load %[[A0_PREFETCH_SMEM]]
490| // CHECK-DAG: %[[A0_CVT:.*]] = tt.fp_to_fp %[[A0_PREFETCH]]
491| // CHECK-DAG: %[[B0_PREFETCH_SMEM:.*]] = ttg.memdesc_subslice %[[B0:.*]][0, 0]
492| // CHECK-DAG: %[[B0_PREFETCH:.*]] = ttg.local_load %[[B0_PREFETCH_SMEM]]
493| // CHECK:     scf.for {{.*}} iter_args({{.*}}, {{.*}}, %[[arg_a0:.*]] = %[[A0]], %[[arg_b0:.*]] = %[[B0]], {{.*}}, %[[a0_prefetch:.*]] = %[[A0_CVT]], %[[b0_prefetch:.*]] = %[[B0_PREFETCH]]
494| // CHECK-DAG:   %[[A_REM_SMEM:.*]] = ttg.memdesc_subslice %[[arg_a0]][0, 16]
495| // CHECK-DAG:   %[[A_REM:.*]] = ttg.local_load %[[A_REM_SMEM]]
496| // CHECK-DAG:   %[[A_REM_CVT:.*]] = tt.fp_to_fp %[[A_REM]]
497| // CHECK-DAG:   %[[B_REM_SMEM:.*]] = ttg.memdesc_subslice %[[arg_b0]][16, 0]
498| // CHECK-DAG:   %[[B_REM:.*]] = ttg.local_load %[[B_REM_SMEM]]
499| // CHECK:       %[[D_FIRST:.*]] = tt.dot %[[a0_prefetch]], %[[b0_prefetch:.*]], {{.*}}
500| // CHECK-DAG:   %[[NEXT_A_PREFETCH_SMEM:.*]] = ttg.memdesc_subslice {{.*}}[0, 0]
501| // CHECK-DAG:   %[[NEXT_A_PREFETCH:.*]] = ttg.local_load %[[NEXT_A_PREFETCH_SMEM]]
502| // CHECK-DAG:   %[[NEXT_A_PREFETCH_CVT:.*]] = tt.fp_to_fp %[[NEXT_A_PREFETCH]]
503| // CHECK-DAG:   %[[NEXT_B_PREFETCH_SMEM:.*]] = ttg.memdesc_subslice {{.*}}[0, 0]
504| // CHECK-DAG:   %[[NEXT_B_PREFETCH:.*]] = ttg.local_load %[[NEXT_B_PREFETCH_SMEM]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: tt.func @matmul_loop_mixed_amd anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: tt.func @matmul_loop_mixed_amd 这样的标签用于锚定匹配范围。

### Lines 505-506
```mlir
505| // CHECK:       tt.dot %[[A_REM_CVT]], %[[B_REM]], %[[D_FIRST:.*]]
506| // CHECK:     scf.yield {{.*}}, {{.*}}, {{.*}}, {{.*}}, {{.*}}, %[[NEXT_A_PREFETCH_CVT]], %[[NEXT_B_PREFETCH]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 507-507
```mlir
507| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 508-525
```mlir
508| tt.func @matmul_loop_mixed_amd(%lb : index, %ub : index, %step : index, %A : !tt.ptr<f8E5M2>, %B : !tt.ptr<f16>) -> tensor<128x128xf32, #C>{
509|   %a_ptr_init = tt.splat %A : !tt.ptr<f8E5M2> -> tensor<128x32x!tt.ptr<f8E5M2>, #AL>
510|   %b_ptr_init = tt.splat %B : !tt.ptr<f16> -> tensor<32x128x!tt.ptr<f16>, #BL>
511| 
512|   %a_mask = arith.constant dense<true> : tensor<128x32xi1, #AL>
513|   %a_other = arith.constant dense<0.00e+00> : tensor<128x32xf8E5M2, #AL>
514|   %b_mask = arith.constant dense<true> : tensor<32x128xi1, #BL>
515|   %b_other = arith.constant dense<0.00e+00> : tensor<32x128xf16, #BL>
516|   %c_init = arith.constant dense<0.00e+00> : tensor<128x128xf32, #C>
517| 
518|   %a_off = arith.constant dense<4> : tensor<128x32xi32, #AL>
519|   %b_off = arith.constant dense<4> : tensor<32x128xi32, #BL>
520| 
521|   %a_ = tt.load %a_ptr_init, %a_mask, %a_other : tensor<128x32x!tt.ptr<f8E5M2>, #AL>
522|   %a_init = ttg.local_alloc %a_ : (tensor<128x32xf8E5M2, #AL>) -> !ttg.memdesc<128x32xf8E5M2, #A, #smem>
523|   %b_ = tt.load %b_ptr_init, %b_mask, %b_other : tensor<32x128x!tt.ptr<f16>, #BL>
524|   %b_init = ttg.local_alloc %b_ : (tensor<32x128xf16, #BL>) -> !ttg.memdesc<32x128xf16, #B, #smem>
525| 
```
**EN:** This function-oriented block defines or enters `matmul_loop_mixed_amd`. Within it, the test exercises constants, tt.func, broadcasted scalars or pointers, masked or vectorized loads, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `matmul_loop_mixed_amd` 为核心。测试在其中演示 常量、tt.func、广播后的标量或指针、带掩码或向量化的加载、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 526-543
```mlir
526|   %loop:5 = scf.for %iv = %lb to %ub step %step iter_args(%a_ptr = %a_ptr_init, %b_ptr = %b_ptr_init, %a = %a_init, %b = %b_init, %prev_c = %c_init) -> (tensor<128x32x!tt.ptr<f8E5M2>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, !ttg.memdesc<128x32xf8E5M2, #A, #smem>, !ttg.memdesc<32x128xf16, #B, #smem>, tensor<128x128xf32, #C>) {
527|     %a_op_ = ttg.local_load %a : !ttg.memdesc<128x32xf8E5M2, #A, #smem> -> tensor<128x32xf8E5M2, #A_OP>
528|     %a_op = tt.fp_to_fp %a_op_ : tensor<128x32xf8E5M2, #A_OP> -> tensor<128x32xf16, #A_OP>
529|     %b_op = ttg.local_load %b : !ttg.memdesc<32x128xf16, #B, #smem> -> tensor<32x128xf16, #B_OP>
530|     %c = tt.dot %a_op, %b_op, %prev_c : tensor<128x32xf16, #A_OP> * tensor<32x128xf16, #B_OP> -> tensor<128x128xf32, #C>
531| 
532|     %next_a_ptr = tt.addptr %a_ptr, %a_off : tensor<128x32x!tt.ptr<f8E5M2>, #AL>, tensor<128x32xi32, #AL>
533|     %next_b_ptr = tt.addptr %b_ptr, %b_off : tensor<32x128x!tt.ptr<f16>, #BL>, tensor<32x128xi32, #BL>
534|     %next_a_ = tt.load %next_a_ptr, %a_mask, %a_other : tensor<128x32x!tt.ptr<f8E5M2>, #AL>
535|     %next_a = ttg.local_alloc %next_a_ : (tensor<128x32xf8E5M2, #AL>) -> !ttg.memdesc<128x32xf8E5M2, #A, #smem>
536|     %next_b_ = tt.load %next_b_ptr, %b_mask, %b_other : tensor<32x128x!tt.ptr<f16>, #BL>
537|     %next_b = ttg.local_alloc %b_ : (tensor<32x128xf16, #BL>) -> !ttg.memdesc<32x128xf16, #B, #smem>
538| 
539|     scf.yield %next_a_ptr, %next_b_ptr, %next_a, %next_b, %c : tensor<128x32x!tt.ptr<f8E5M2>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, !ttg.memdesc<128x32xf8E5M2, #A, #smem>, !ttg.memdesc<32x128xf16, #B, #smem>, tensor<128x128xf32, #C>
540|   }
541|   tt.return %loop#4 : tensor<128x128xf32, #C>
542| }
543| }  // end module
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining local/shared memory loads, pointer arithmetic, masked or vectorized loads, shared/local memory allocation, structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 本地/共享内存加载、指针算术、带掩码或向量化的加载、共享/本地内存分配、结构化循环。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `-tritongpu-prefetch`, `-canonicalize`
- **CN:** 主要 pass 选项：`-split-input-file`，`-tritongpu-prefetch`，`-canonicalize`
- **EN:** Dominant operations include `arith.constant`, `ttg.local_alloc`, `tt.load`, `tt.func`, `ttg.local_load`, `tt.splat`, `tt.dot`, `scf.yield`, `module`, `scf.for`.
- **CN:** 主要操作包括 `arith.constant`、`ttg.local_alloc`、`tt.load`、`tt.func`、`ttg.local_load`、`tt.splat`、`tt.dot`、`scf.yield`、`module`、`scf.for`。
- **EN:** The file contains 3 independently testable section(s). Check styles used: CHECK-DAG x110, CHECK x48, CHECK-LABEL x5, CHECK-NOT x1. Important labels include tt.func @split_pipelined_mmav2_loads, tt.func @prefetch_pipelined_loop_args, tt.func @prefetch_induction_var_source, tt.func @split_pipelined_mmav2_loads_f64. It validates both the presence of the desired rewrite and the absence of unwanted residual operations.
- **CN:** 该文件包含 3 个可独立测试的分段。使用的检查类型：CHECK-DAG ×110，CHECK ×48，CHECK-LABEL ×5，CHECK-NOT ×1。 关键标签包括 tt.func @split_pipelined_mmav2_loads，tt.func @prefetch_pipelined_loop_args，tt.func @prefetch_induction_var_source，tt.func @split_pipelined_mmav2_loads_f64。 它同时验证目标重写已经出现，并且不再残留不需要的操作。

## Dependencies / 依赖关系

- **EN:** This test depends on Triton/MLIR textual IR parsing, the pass pipeline named in the RUN lines, and the dialect/type definitions referenced in the file.
- **CN:** 该测试依赖 Triton/MLIR 文本 IR 解析、RUN 行中指定的 pass 流水线，以及文件中引用的方言/类型定义。
- **EN:** `ttg`: TritonGPU layout, memory, and GPU-specific operations.
- **CN:** `ttg`：TritonGPU 布局、内存和 GPU 专用操作。
- **EN:** `tt`: Triton high-level tensor/pointer operations.
- **CN:** `tt`：Triton 高层张量/指针操作。
- **EN:** `arith`: MLIR arithmetic dialect for scalar/tensor math.
- **CN:** `arith`：用于标量/张量运算的 MLIR arithmetic 方言。
- **EN:** `scf`: Structured control flow for loops and conditionals.
- **CN:** `scf`：用于循环与条件分支的结构化控制流。
- **EN:** `FileCheck` is used to match required textual patterns after rewriting/lowering.
- **CN:** `FileCheck` 用于在重写/lower 之后匹配必须出现的文本模式。