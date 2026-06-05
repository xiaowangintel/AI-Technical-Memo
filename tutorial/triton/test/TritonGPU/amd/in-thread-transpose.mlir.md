# in-thread-transpose.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/amd/in-thread-transpose.mlir`
- **EN:** Test for the MLIR/Triton pipeline `-tritonamdgpu-in-thread-transpose` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `-tritonamdgpu-in-thread-transpose` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -split-input-file -tritonamdgpu-in-thread-transpose | FileCheck %s`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -split-input-file -tritonamdgpu-in-thread-transpose | FileCheck %s`

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
1| // RUN: triton-opt %s -split-input-file -tritonamdgpu-in-thread-transpose | FileCheck %s
2| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -split-input-file -tritonamdgpu-in-thread-transpose | FileCheck %s` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -split-input-file -tritonamdgpu-in-thread-transpose | FileCheck %s`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 3-8
```mlir
3| #blocked = #ttg.blocked<{sizePerThread = [8, 1], threadsPerWarp = [8, 8], warpsPerCTA = [8, 1], order = [0, 1]}>
4| #blocked1 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 8], warpsPerCTA = [1, 8], order = [1, 0]}>
5| #shared = #ttg.swizzled_shared<{vec = 4, perPhase = 2, maxPhase = 4, order = [0, 1]}>
6| #smem = #ttg.shared_memory
7| #mma = #ttg.amd_mfma<{version = 3, warpsPerCTA = [4, 2], instrShape = [32, 32, 8], isTransposed = true}>
8| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 9-10
```mlir
 9| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
10| 
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 11-28
```mlir
11| // CHECK-DAG: [[$OLD_LAYOUT1:#.*]] = #ttg.blocked<{sizePerThread = [8, 1], threadsPerWarp = [8, 8], warpsPerCTA = [8, 1], order = [0, 1]}>
12| // CHECK-DAG: [[$OLD_LAYOUT2:#.*]] = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 8], warpsPerCTA = [1, 8], order = [1, 0]}>
13| // CHECK-DAG: [[$TRANSPOSABLE_LAYOUT1:#.*]] = #ttg.blocked<{sizePerThread = [8, 4], threadsPerWarp = [32, 2], warpsPerCTA = [1, 8], order = [0, 1]}>
14| // CHECK-DAG: [[$TRANSPOSABLE_LAYOUT2:#.*]] = #ttg.blocked<{sizePerThread = [4, 8], threadsPerWarp = [4, 16], warpsPerCTA = [8, 1], order = [1, 0]}>
15| // CHECK-DAG: [[$LINEAR1:#.*]] = #ttg.linear<{register = {{\[\[}}0, 1], [0, 2], [1, 0], [2, 0], [4, 0{{]]}}, lane = {{\[\[}}8, 0], [16, 0], [32, 0], [64, 0], [128, 0], [0, 4{{]]}}, warp = {{\[\[}}0, 8], [0, 16], [0, 0{{]]}}, block = []}>
16| // CHECK-DAG: [[$LINEAR2:#.*]] = #ttg.linear<{register = {{\[\[}}1, 0], [2, 0], [0, 1], [0, 2], [0, 4{{]]}}, lane = {{\[\[}}0, 8], [0, 16], [0, 32], [0, 64], [4, 0], [8, 0{{]]}}, warp = {{\[\[}}16, 0], [0, 0], [0, 0{{]]}}, block = []}>
17| // CHECK-DAG: [[$SHARED1:#.*]] = #ttg.amd_rotating_shared<{vec = 4, perPhase = 2, maxPhase = 8, order = [1, 0]}>
18| // CHECK-DAG: [[$SHARED2:#.*]] = #ttg.amd_rotating_shared<{vec = 4, perPhase = 2, maxPhase = 8, order = [0, 1]}>
19| 
20| // CHECK-LABEL: inThreadTranspose_simple
21| 
22| // CHECK-DAG: [[LOAD_VAL1:%.*]] = tt.load {{.*}} : tensor<256x32x!tt.ptr<f16>, [[$TRANSPOSABLE_LAYOUT1]]>
23| // CHECK-DAG: [[LOAD_VAL2:%.*]] = tt.load {{.*}} : tensor<32x128x!tt.ptr<f16>, [[$TRANSPOSABLE_LAYOUT2]]>
24| 
25| // CHECK-DAG: [[TMP1_VAL1:%.*]] = ttg.convert_layout [[LOAD_VAL1]] : tensor<256x32xf16, [[$TRANSPOSABLE_LAYOUT1]]> -> tensor<256x32xf16, [[$OLD_LAYOUT1]]>
26| // CHECK-DAG: [[TMP2_VAL1:%.*]] = ttg.convert_layout [[TMP1_VAL1]] : tensor<256x32xf16, [[$OLD_LAYOUT1]]> -> tensor<256x32xf16, [[$TRANSPOSABLE_LAYOUT1]]>
27| // CHECK-DAG: [[TRANSPOSED_VAL1:%.*]] = amdg.in_thread_transpose [[TMP2_VAL1]] : tensor<256x32xf16, [[$TRANSPOSABLE_LAYOUT1]]> -> tensor<256x32xf16, [[$LINEAR1]]>
28| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: inThreadTranspose_simple anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: inThreadTranspose_simple 这样的标签用于锚定匹配范围。

### Lines 29-36
```mlir
29| // CHECK-DAG: [[TMP1_VAL2:%.*]] = ttg.convert_layout [[LOAD_VAL2]] : tensor<32x128xf16, [[$TRANSPOSABLE_LAYOUT2]]> -> tensor<32x128xf16, [[$OLD_LAYOUT2]]>
30| // CHECK-DAG: [[TMP2_VAL2:%.*]] = ttg.convert_layout [[TMP1_VAL2]] : tensor<32x128xf16, [[$OLD_LAYOUT2]]> -> tensor<32x128xf16, [[$TRANSPOSABLE_LAYOUT2]]>
31| // CHECK-DAG: [[TRANSPOSED_VAL2:%.*]] = amdg.in_thread_transpose [[TMP2_VAL2]] : tensor<32x128xf16, [[$TRANSPOSABLE_LAYOUT2]]> -> tensor<32x128xf16, [[$LINEAR2]]>
32| 
33| // CHECK-DAG: [[ALLOC1:%.*]] = ttg.local_alloc [[TRANSPOSED_VAL1]] : (tensor<256x32xf16, [[$LINEAR1]]>) -> !ttg.memdesc<256x32xf16, [[$SHARED1]], #smem>
34| // CHECK-DAG: [[ALLOC2:%.*]] = ttg.local_alloc [[TRANSPOSED_VAL2]] : (tensor<32x128xf16, [[$LINEAR2]]>) -> !ttg.memdesc<32x128xf16, [[$SHARED2]], #smem>
35| // CHECK-DAG: ttg.local_load [[ALLOC1]] : !ttg.memdesc<256x32xf16, [[$SHARED1]], #smem> -> tensor<256x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>
36| // CHECK-DAG: ttg.local_load [[ALLOC2]] : !ttg.memdesc<32x128xf16, [[$SHARED2]], #smem> -> tensor<32x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 37-54
```mlir
37|   tt.func public @inThreadTranspose_simple(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
38|     %cst_0 = arith.constant dense<0.000000e+00> : tensor<256x128xf32, #mma>
39|     %0 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<256x32x!tt.ptr<f16>, #blocked>
40|     %1 = tt.splat %arg1 : !tt.ptr<f16> -> tensor<32x128x!tt.ptr<f16>, #blocked1>
41|     %2 = tt.load %0 : tensor<256x32x!tt.ptr<f16>, #blocked>
42|     %3 = tt.load %1 : tensor<32x128x!tt.ptr<f16>, #blocked1>
43| 
44|     %4 = ttg.local_alloc %2 : (tensor<256x32xf16, #blocked>) -> !ttg.memdesc<256x32xf16, #shared, #smem>
45|     %5 = ttg.local_load %4 : !ttg.memdesc<256x32xf16, #shared, #smem> -> tensor<256x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>
46| 
47|     %6 = ttg.local_alloc %3 : (tensor<32x128xf16, #blocked1>) -> !ttg.memdesc<32x128xf16, #shared, #smem>
48|     %7 = ttg.local_load %6 : !ttg.memdesc<32x128xf16, #shared, #smem> -> tensor<32x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>
49| 
50|     %8 = tt.dot %5, %7, %cst_0 : tensor<256x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>> * tensor<32x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>> -> tensor<256x128xf32, #mma>
51|     tt.return
52|   }
53| }
54| 
```
**EN:** This function-oriented block defines or enters `inThreadTranspose_simple`. Within it, the test exercises tt.func, broadcasted scalars or pointers, masked or vectorized loads, shared/local memory allocation, local/shared memory loads, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `inThreadTranspose_simple` 为核心。测试在其中演示 tt.func、广播后的标量或指针、带掩码或向量化的加载、共享/本地内存分配、本地/共享内存加载，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 55-55
```mlir
55| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 56-62
```mlir
56| 
57| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [16, 4], warpsPerCTA = [8, 1], order = [1, 0]}>
58| #blocked1 = #ttg.blocked<{sizePerThread = [8, 1], threadsPerWarp = [4, 16], warpsPerCTA = [1, 8], order = [0, 1]}>
59| #shared = #ttg.swizzled_shared<{vec = 4, perPhase = 2, maxPhase = 4, order = [0, 1]}>
60| #smem = #ttg.shared_memory
61| #mma = #ttg.amd_mfma<{version = 3, warpsPerCTA = [4, 2], instrShape = [32, 32, 8], isTransposed = true}>
62| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 63-64
```mlir
63| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
64| 
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 65-73
```mlir
65| // CHECK-NOT: #ttg.amd_rotating_shared
66| // CHECK-NOT: #ttg.linear
67| // CHECK-DAG: [[$BLOCKED1:#.*]] = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [16, 4], warpsPerCTA = [8, 1], order = [1, 0]}>
68| // CHECK-DAG: [[$BLOCKED2:#.*]] = #ttg.blocked<{sizePerThread = [8, 1], threadsPerWarp = [4, 16], warpsPerCTA = [1, 8], order = [0, 1]}>
69| // CHECK-NOT: #ttg.amd_rotating_shared
70| // CHECK-NOT: #ttg.linear
71| // CHECK-LABEL: inThreadTranspose_k_fast_neg
72| // CHECK-DAG: tt.load {{.*}} : tensor<256x32x!tt.ptr<f16>, [[$BLOCKED1]]>
73| // CHECK-DAG: tt.load {{.*}} : tensor<32x128x!tt.ptr<f16>, [[$BLOCKED2]]>
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: inThreadTranspose_k_fast_neg anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: inThreadTranspose_k_fast_neg 这样的标签用于锚定匹配范围。

### Lines 74-91
```mlir
74|   tt.func public @inThreadTranspose_k_fast_neg(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
75|     %cst_0 = arith.constant dense<0.000000e+00> : tensor<256x128xf32, #mma>
76|     %0 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<256x32x!tt.ptr<f16>, #blocked>
77|     %1 = tt.splat %arg1 : !tt.ptr<f16> -> tensor<32x128x!tt.ptr<f16>, #blocked1>
78|     %2 = tt.load %0 : tensor<256x32x!tt.ptr<f16>, #blocked>
79|     %3 = tt.load %1 : tensor<32x128x!tt.ptr<f16>, #blocked1>
80| 
81|     %4 = ttg.local_alloc %2 : (tensor<256x32xf16, #blocked>) -> !ttg.memdesc<256x32xf16, #shared, #smem>
82|     %5 = ttg.local_load %4 : !ttg.memdesc<256x32xf16, #shared, #smem> -> tensor<256x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>
83| 
84|     %6 = ttg.local_alloc %3 : (tensor<32x128xf16, #blocked1>) -> !ttg.memdesc<32x128xf16, #shared, #smem>
85|     %7 = ttg.local_load %6 : !ttg.memdesc<32x128xf16, #shared, #smem> -> tensor<32x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>
86| 
87|     %8 = tt.dot %5, %7, %cst_0 : tensor<256x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>> * tensor<32x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>> -> tensor<256x128xf32, #mma>
88|     tt.return
89|   }
90| }
91| 
```
**EN:** This function-oriented block defines or enters `inThreadTranspose_k_fast_neg`. Within it, the test exercises tt.func, broadcasted scalars or pointers, masked or vectorized loads, shared/local memory allocation, local/shared memory loads, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `inThreadTranspose_k_fast_neg` 为核心。测试在其中演示 tt.func、广播后的标量或指针、带掩码或向量化的加载、共享/本地内存分配、本地/共享内存加载，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 92-92
```mlir
92| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 93-99
```mlir
93| 
94| #blocked = #ttg.blocked<{sizePerThread = [8, 1], threadsPerWarp = [16, 4], warpsPerCTA = [1, 8], order = [0, 1]}>
95| #blocked1 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 16], warpsPerCTA = [8, 1], order = [1, 0]}>
96| #shared = #ttg.swizzled_shared<{vec = 4, perPhase = 2, maxPhase = 4, order = [0, 1]}>
97| #smem = #ttg.shared_memory
98| #mma = #ttg.amd_mfma<{version = 3, warpsPerCTA = [4, 2], instrShape = [32, 32, 8], isTransposed = true}>
99| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 100-101
```mlir
100| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
101| 
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 102-110
```mlir
102| // CHECK-NOT: #ttg.amd_rotating_shared
103| // CHECK-NOT: #ttg.linear
104| // CHECK-DAG: [[$BLOCKED1:#.*]] = #ttg.blocked<{sizePerThread = [8, 1], threadsPerWarp = [16, 4], warpsPerCTA = [1, 8], order = [0, 1]}>
105| // CHECK-DAG: [[$BLOCKED2:#.*]] = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 16], warpsPerCTA = [8, 1], order = [1, 0]}>
106| // CHECK-NOT: #ttg.amd_rotating_shared
107| // CHECK-NOT: #ttg.linear
108| // CHECK-LABEL: inThreadTranspose_small_k_neg
109| // CHECK-DAG: tt.load {{.*}} : tensor<256x32x!tt.ptr<f16>, [[$BLOCKED1]]>
110| // CHECK-DAG: tt.load {{.*}} : tensor<32x128x!tt.ptr<f16>, [[$BLOCKED2]]>
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: inThreadTranspose_small_k_neg anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: inThreadTranspose_small_k_neg 这样的标签用于锚定匹配范围。

### Lines 111-128
```mlir
111|   tt.func public @inThreadTranspose_small_k_neg(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
112|     %cst_0 = arith.constant dense<0.000000e+00> : tensor<256x128xf32, #mma>
113|     %0 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<256x32x!tt.ptr<f16>, #blocked>
114|     %1 = tt.splat %arg1 : !tt.ptr<f16> -> tensor<32x128x!tt.ptr<f16>, #blocked1>
115|     %2 = tt.load %0 : tensor<256x32x!tt.ptr<f16>, #blocked>
116|     %3 = tt.load %1 : tensor<32x128x!tt.ptr<f16>, #blocked1>
117| 
118|     %4 = ttg.local_alloc %2 : (tensor<256x32xf16, #blocked>) -> !ttg.memdesc<256x32xf16, #shared, #smem>
119|     %5 = ttg.local_load %4 : !ttg.memdesc<256x32xf16, #shared, #smem> -> tensor<256x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>
120| 
121|     %6 = ttg.local_alloc %3 : (tensor<32x128xf16, #blocked1>) -> !ttg.memdesc<32x128xf16, #shared, #smem>
122|     %7 = ttg.local_load %6 : !ttg.memdesc<32x128xf16, #shared, #smem> -> tensor<32x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>
123| 
124|     %8 = tt.dot %5, %7, %cst_0 : tensor<256x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>> * tensor<32x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>> -> tensor<256x128xf32, #mma>
125|     tt.return
126|   }
127| }
128| 
```
**EN:** This function-oriented block defines or enters `inThreadTranspose_small_k_neg`. Within it, the test exercises tt.func, broadcasted scalars or pointers, masked or vectorized loads, shared/local memory allocation, local/shared memory loads, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `inThreadTranspose_small_k_neg` 为核心。测试在其中演示 tt.func、广播后的标量或指针、带掩码或向量化的加载、共享/本地内存分配、本地/共享内存加载，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 129-129
```mlir
129| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 130-147
```mlir
130| 
131| // CHECK-DAG: [[$OLD_LAYOUT:#.*]] = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 8], warpsPerCTA = [1, 1], order = [1, 0]}>
132| // CHECK-DAG: [[$TRANSPOSABLE_LAYOUT:#.*]] = #ttg.blocked<{sizePerThread = [4, 8], threadsPerWarp = [8, 8], warpsPerCTA = [1, 1], order = [1, 0]}>
133| // CHECK-DAG: [[$LINEAR:#.*]] = #ttg.linear<{register = {{\[\[}}1, 0], [2, 0], [0, 1], [0, 2], [0, 4], [32, 0{{]]}}, lane = {{\[\[}}0, 8], [0, 16], [0, 32], [4, 0], [8, 0], [16, 0{{]]}}, warp = [], block = []}>
134| // CHECK-DAG: [[$SHARED:#.*]] = #ttg.amd_rotating_shared<{vec = 4, perPhase = 1, maxPhase = 16, order = [0, 1]}>
135| 
136| // CHECK-LABEL: inThreadTranspose_with_cfg
137| 
138| // CHECK-DAG: ttg.local_alloc : () -> !ttg.memdesc<1x64x64xf16, [[$SHARED]], #smem, mutable>
139| // CHECK-DAG: [[LOAD_VAL_preloop:%.*]] = tt.load {{.*}} : tensor<64x64x!tt.ptr<f16>, [[$TRANSPOSABLE_LAYOUT]]>
140| 
141| // CHECK-DAG: [[TMP1_VAL_preloop:%.*]] = ttg.convert_layout [[LOAD_VAL_preloop]] : tensor<64x64xf16, [[$TRANSPOSABLE_LAYOUT]]> -> tensor<64x64xf16, [[$OLD_LAYOUT]]>
142| // CHECK-DAG: [[TMP2_VAL_preloop:%.*]] = ttg.convert_layout [[TMP1_VAL_preloop]] : tensor<64x64xf16, [[$OLD_LAYOUT]]> -> tensor<64x64xf16, [[$TRANSPOSABLE_LAYOUT]]>
143| // CHECK-DAG: [[TRANSPOSED_VAL_preloop:%.*]] = amdg.in_thread_transpose [[TMP2_VAL_preloop]] : tensor<64x64xf16, [[$TRANSPOSABLE_LAYOUT]]> -> tensor<64x64xf16, [[$LINEAR]]>
144| 
145| // CHECK-DAG: ttg.local_store [[TRANSPOSED_VAL_preloop]], {{.*}} : tensor<64x64xf16, [[$LINEAR]]> -> !ttg.memdesc<64x64xf16, [[$SHARED]], #smem, mutable>
146| // CHECK: scf.for
147| // CHECK-DAG: [[LOAD_VAL_loop:%.*]] = tt.load {{.*}} : tensor<64x64x!tt.ptr<f16>, [[$TRANSPOSABLE_LAYOUT]]>
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: inThreadTranspose_with_cfg anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: inThreadTranspose_with_cfg 这样的标签用于锚定匹配范围。

### Lines 148-154
```mlir
148| // CHECK-DAG: ttg.local_load {{.*}} : !ttg.memdesc<64x64xf16, [[$SHARED]], #smem, mutable> -> tensor<64x64xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>
149| 
150| // CHECK-DAG: [[TMP1_VAL_loop:%.*]] = ttg.convert_layout [[LOAD_VAL_loop]] : tensor<64x64xf16, [[$TRANSPOSABLE_LAYOUT]]> -> tensor<64x64xf16, [[$OLD_LAYOUT]]>
151| // CHECK-DAG: [[TMP2_VAL_loop:%.*]] = ttg.convert_layout [[TMP1_VAL_loop]] : tensor<64x64xf16, [[$OLD_LAYOUT]]> -> tensor<64x64xf16, [[$TRANSPOSABLE_LAYOUT]]>
152| // CHECK-DAG: [[TRANSPOSED_VAL_loop:%.*]] = amdg.in_thread_transpose [[TMP2_VAL_loop]] : tensor<64x64xf16, [[$TRANSPOSABLE_LAYOUT]]> -> tensor<64x64xf16, [[$LINEAR]]>
153| 
154| // CHECK: ttg.local_store [[TRANSPOSED_VAL_loop]], {{.*}} : tensor<64x64xf16, [[$LINEAR]]> -> !ttg.memdesc<64x64xf16, [[$SHARED]], #smem, mutable>
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 155-159
```mlir
155| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 8], warpsPerCTA = [1, 1], order = [1, 0]}>
156| #mma = #ttg.amd_mfma<{version = 3, warpsPerCTA = [1, 1], instrShape = [16, 16, 16], isTransposed = true}>
157| #shared = #ttg.swizzled_shared<{vec = 4, perPhase = 1, maxPhase = 16, order = [1, 0]}>
158| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
159| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 160-160
```mlir
160| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 161-178
```mlir
161|   tt.func public @inThreadTranspose_with_cfg(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg2: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg3: i32 {tt.divisibility = 16 : i32}, %arg4: i32 {tt.divisibility = 16 : i32}, %arg5: i32 {tt.divisibility = 16 : i32}, %arg6: i32 {tt.divisibility = 16 : i32}, %arg7: i32 {tt.divisibility = 16 : i32}, %arg8: i32 {tt.divisibility = 16 : i32}) {
162|     %cst = arith.constant dense<64> : tensor<64x64xi32, #blocked>
163|     %c64_i32 = arith.constant 64 : i32
164|     %c1_i32 = arith.constant 1 : i32
165|     %c0_i32 = arith.constant 0 : i32
166|     %c63_i32 = arith.constant 63 : i32
167|     %cst_0 = arith.constant dense<0.000000e+00> : tensor<64x64xf32, #mma>
168|     %cst_1 = arith.constant dense<true> : tensor<64x64xi1, #blocked>
169|     %cst_2 = arith.constant dense<true> : tensor<64x64xi1, #mma>
170|     %0 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<64x64x!tt.ptr<f16>, #blocked>
171|     %1 = tt.splat %arg1 : !tt.ptr<f16> -> tensor<64x64x!tt.ptr<f16>, #blocked>
172|     %2 = arith.addi %arg5, %c63_i32 : i32
173|     %3 = arith.divsi %2, %c64_i32 : i32
174|     %4 = arith.muli %arg7, %c64_i32 : i32
175|     %5 = tt.splat %4 : i32 -> tensor<64x64xi32, #blocked>
176|     %6 = ttg.local_alloc  : () -> !ttg.memdesc<1x64x64xf16, #shared, #smem, mutable>
177|     %7 = ttg.local_alloc  : () -> !ttg.memdesc<1x64x64xf16, #shared1, #smem, mutable>
178|     %8 = tt.load %0, %cst_1 : tensor<64x64x!tt.ptr<f16>, #blocked>
```
**EN:** This function-oriented block defines or enters `inThreadTranspose_with_cfg`. Within it, the test exercises constants, broadcasted scalars or pointers, tt.func, shared/local memory allocation, integer additions, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `inThreadTranspose_with_cfg` 为核心。测试在其中演示 常量、广播后的标量或指针、tt.func、共享/本地内存分配、整数加法，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 179-196
```mlir
179|     %9 = tt.load %1, %cst_1 : tensor<64x64x!tt.ptr<f16>, #blocked>
180|     %10 = ttg.memdesc_index %6[%c0_i32] : !ttg.memdesc<1x64x64xf16, #shared, #smem, mutable> -> !ttg.memdesc<64x64xf16, #shared, #smem, mutable>
181|     ttg.local_store %8, %10 : tensor<64x64xf16, #blocked> -> !ttg.memdesc<64x64xf16, #shared, #smem, mutable>
182|     %11 = ttg.memdesc_index %7[%c0_i32] : !ttg.memdesc<1x64x64xf16, #shared1, #smem, mutable> -> !ttg.memdesc<64x64xf16, #shared1, #smem, mutable>
183|     ttg.local_store %9, %11 : tensor<64x64xf16, #blocked> -> !ttg.memdesc<64x64xf16, #shared1, #smem, mutable>
184|     %12 = arith.subi %3, %c1_i32 : i32
185|     %13:6 = scf.for %arg9 = %c0_i32 to %12 step %c1_i32 iter_args(%arg10 = %cst_0, %arg11 = %0, %arg12 = %1, %arg13 = %c0_i32, %arg14 = %10, %arg15 = %11) -> (tensor<64x64xf32, #mma>, tensor<64x64x!tt.ptr<f16>, #blocked>, tensor<64x64x!tt.ptr<f16>, #blocked>, i32, !ttg.memdesc<64x64xf16, #shared, #smem, mutable>, !ttg.memdesc<64x64xf16, #shared1, #smem, mutable>)  : i32 {
186|       %21 = tt.addptr %arg11, %cst : tensor<64x64x!tt.ptr<f16>, #blocked>, tensor<64x64xi32, #blocked>
187|       %22 = tt.addptr %arg12, %5 : tensor<64x64x!tt.ptr<f16>, #blocked>, tensor<64x64xi32, #blocked>
188|       %23 = tt.load %21 : tensor<64x64x!tt.ptr<f16>, #blocked>
189|       %24 = ttg.local_load %arg14 : !ttg.memdesc<64x64xf16, #shared, #smem, mutable> -> tensor<64x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>
190|       %25 = tt.load %22 : tensor<64x64x!tt.ptr<f16>, #blocked>
191|       %26 = ttg.local_load %arg15 : !ttg.memdesc<64x64xf16, #shared1, #smem, mutable> -> tensor<64x64xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>
192|       %27 = tt.dot %24, %26, %arg10, inputPrecision = tf32 : tensor<64x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>> * tensor<64x64xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>> -> tensor<64x64xf32, #mma>
193|       %28 = arith.addi %arg13, %c1_i32 : i32
194|       %29 = arith.cmpi slt, %28, %c1_i32 : i32
195|       %30 = arith.select %29, %28, %c0_i32 : i32
196|       %31 = ttg.memdesc_index %6[%30] : !ttg.memdesc<1x64x64xf16, #shared, #smem, mutable> -> !ttg.memdesc<64x64xf16, #shared, #smem, mutable>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining masked or vectorized loads, ttg.memdesc_index, local/shared memory stores, pointer arithmetic, local/shared memory loads.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 带掩码或向量化的加载、ttg.memdesc_index、本地/共享内存存储、指针算术、本地/共享内存加载。

### Lines 197-214
```mlir
197|       ttg.local_store %23, %31 : tensor<64x64xf16, #blocked> -> !ttg.memdesc<64x64xf16, #shared, #smem, mutable>
198|       %32 = ttg.memdesc_index %7[%30] : !ttg.memdesc<1x64x64xf16, #shared1, #smem, mutable> -> !ttg.memdesc<64x64xf16, #shared1, #smem, mutable>
199|       ttg.local_store %25, %32 : tensor<64x64xf16, #blocked> -> !ttg.memdesc<64x64xf16, #shared1, #smem, mutable>
200|       scf.yield %27, %21, %22, %30, %31, %32 : tensor<64x64xf32, #mma>, tensor<64x64x!tt.ptr<f16>, #blocked>, tensor<64x64x!tt.ptr<f16>, #blocked>, i32, !ttg.memdesc<64x64xf16, #shared, #smem, mutable>, !ttg.memdesc<64x64xf16, #shared1, #smem, mutable>
201|     }
202|     %14 = arith.cmpi sge, %3, %c1_i32 : i32
203|     %15 = ttg.local_load %13#4 : !ttg.memdesc<64x64xf16, #shared, #smem, mutable> -> tensor<64x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>
204|     %16 = ttg.local_load %13#5 : !ttg.memdesc<64x64xf16, #shared1, #smem, mutable> -> tensor<64x64xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>
205|     %17 = scf.if %14 -> (tensor<64x64xf32, #mma>) {
206|       %21 = tt.dot %15, %16, %13#0, inputPrecision = tf32 : tensor<64x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>> * tensor<64x64xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>> -> tensor<64x64xf32, #mma>
207|       scf.yield %21 : tensor<64x64xf32, #mma>
208|     } else {
209|       scf.yield %13#0 : tensor<64x64xf32, #mma>
210|     }
211|     %18 = arith.select %14, %17, %13#0 : tensor<64x64xf32, #mma>
212|     ttg.local_dealloc %6 : !ttg.memdesc<1x64x64xf16, #shared, #smem, mutable>
213|     ttg.local_dealloc %7 : !ttg.memdesc<1x64x64xf16, #shared1, #smem, mutable>
214|     %19 = arith.truncf %18 : tensor<64x64xf32, #mma> to tensor<64x64xf16, #mma>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, local/shared memory stores, local/shared memory loads, ttg.local_dealloc, ttg.memdesc_index.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、本地/共享内存存储、本地/共享内存加载、ttg.local_dealloc、ttg.memdesc_index。

### Lines 215-220
```mlir
215|     %20 = tt.splat %arg2 : !tt.ptr<f16> -> tensor<64x64x!tt.ptr<f16>, #mma>
216|     tt.store %20, %19, %cst_2 : tensor<64x64x!tt.ptr<f16>, #mma>
217|     tt.return
218|   }
219| }
220| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on broadcasted scalars or pointers, masked or vectorized stores, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 广播后的标量或指针、带掩码或向量化的存储、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 221-221
```mlir
221| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 222-239
```mlir
222| 
223| // CHECK-LABEL: inThreadTranspose_multiple_local_loads
224| 
225| // CHECK: [[LOAD_ADDR:%.*]] = tt.splat
226| // CHECK: [[IF:%.*]] = scf.if
227| // CHECK-DAG: [[LOAD_ADDR_CVT1:%.*]] = ttg.convert_layout [[LOAD_ADDR]]
228| // CHECK-DAG: [[LOAD_VAL1:%.*]] = tt.load [[LOAD_ADDR_CVT1]]
229| // CHECK-DAG: [[LOAD_VAL1_CVT1:%.*]] = ttg.convert_layout [[LOAD_VAL1]]
230| // CHECK-DAG: [[LOAD_VAL1_CVT2:%.*]] = ttg.convert_layout [[LOAD_VAL1_CVT1:%.*]]
231| // CHECK-DAG: [[TRANSPOSED_IN_REG1:%.*]] = amdg.in_thread_transpose [[LOAD_VAL1_CVT2]]
232| // CHECK-DAG: [[LOCAL_ALLOC1:%.*]] = ttg.local_alloc [[TRANSPOSED_IN_REG1]]
233| // CHECK-DAG: [[LOCAL_LOAD1:%.*]] = ttg.local_load [[LOCAL_ALLOC1]]
234| // CHECK-DAG: scf.yield [[LOCAL_LOAD1]]
235| // CHECK: } else {
236| // CHECK-DAG: [[LOAD_ADDR_CVT2:%.*]] = ttg.convert_layout [[LOAD_ADDR]]
237| // CHECK-DAG: [[LOAD_VAL2:%.*]] = tt.load [[LOAD_ADDR_CVT2]]
238| // CHECK-DAG: [[LOAD_VAL2_CVT1:%.*]] = ttg.convert_layout [[LOAD_VAL2]]
239| // CHECK-DAG: [[LOAD_VAL2_CVT2:%.*]] = ttg.convert_layout [[LOAD_VAL2_CVT1:%.*]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: inThreadTranspose_multiple_local_loads anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: inThreadTranspose_multiple_local_loads 这样的标签用于锚定匹配范围。

### Lines 240-244
```mlir
240| // CHECK-DAG: [[TRANSPOSED_IN_REG2:%.*]] = amdg.in_thread_transpose [[LOAD_VAL2_CVT2]]
241| // CHECK-DAG: [[LOCAL_ALLOC2:%.*]] = ttg.local_alloc [[TRANSPOSED_IN_REG2]]
242| // CHECK-DAG: [[LOCAL_LOAD2:%.*]] = ttg.local_load [[LOCAL_ALLOC2]]
243| // CHECK-DAG: scf.yield [[LOCAL_LOAD2]]
244| // CHECK: tt.dot {{.*}}, [[IF]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 245-250
```mlir
245| #blocked = #ttg.blocked<{sizePerThread = [8, 1], threadsPerWarp = [8, 8], warpsPerCTA = [8, 1], order = [0, 1]}>
246| #blocked1 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 8], warpsPerCTA = [1, 8], order = [1, 0]}>
247| #shared = #ttg.swizzled_shared<{vec = 4, perPhase = 2, maxPhase = 4, order = [0, 1]}>
248| #smem = #ttg.shared_memory
249| #mma = #ttg.amd_mfma<{version = 3, warpsPerCTA = [4, 2], instrShape = [32, 32, 8], isTransposed = true}>
250| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 251-252
```mlir
251| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
252| 
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 253-270
```mlir
253|   tt.func public @inThreadTranspose_multiple_local_loads(%arg0: tensor<256x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>, %arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg2: i1) {
254|     %cst_0 = arith.constant dense<0.000000e+00> : tensor<256x128xf32, #mma>
255|     %0 = tt.splat %arg1 : !tt.ptr<f16> -> tensor<32x128x!tt.ptr<f16>, #blocked1>
256|     %7 = scf.if %arg2 -> (tensor<32x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>) {
257|       %1 = tt.load %0 : tensor<32x128x!tt.ptr<f16>, #blocked1>
258|       %3 = ttg.local_alloc %1 : (tensor<32x128xf16, #blocked1>) -> !ttg.memdesc<32x128xf16, #shared, #smem>
259|       %4 = ttg.local_load %3 : !ttg.memdesc<32x128xf16, #shared, #smem> -> tensor<32x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>
260|       scf.yield %4 : tensor<32x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>
261|     } else {
262|       %2 = tt.load %0 : tensor<32x128x!tt.ptr<f16>, #blocked1>
263|       %5 = ttg.local_alloc %2 : (tensor<32x128xf16, #blocked1>) -> !ttg.memdesc<32x128xf16, #shared, #smem>
264|       %6 = ttg.local_load %5 : !ttg.memdesc<32x128xf16, #shared, #smem> -> tensor<32x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>
265|       scf.yield %6 : tensor<32x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>
266|     }
267| 
268|     %8 = tt.dot %arg0, %7, %cst_0 : tensor<256x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>> * tensor<32x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>> -> tensor<256x128xf32, #mma>
269|     tt.return
270|   }
```
**EN:** This function-oriented block defines or enters `inThreadTranspose_multiple_local_loads`. Within it, the test exercises tt.func, masked or vectorized loads, shared/local memory allocation, local/shared memory loads, loop/if yielded values, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `inThreadTranspose_multiple_local_loads` 为核心。测试在其中演示 tt.func、带掩码或向量化的加载、共享/本地内存分配、本地/共享内存加载、循环/分支产出值，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 271-272
```mlir
271| }
272| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 273-273
```mlir
273| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 274-275
```mlir
274| 
275| // Test that backward SCF traversal correctly process nested CF structures
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 276-290
```mlir
276| // CHECK-LABEL: inThreadTranspose_nested_scf_traversal_regression
277| 
278| // CHECK: [[IF:%.*]] = scf.if {{.*}} -> (!ttg.memdesc<32x128xf16, #shared, #smem>) {
279| // CHECK:   scf.if {{.*}} -> (tensor<32x128xf16, #blocked>) {
280| // CHECK:   } else {
281| // CHECK:   }
282| // CHECK:   [[TRANS1:%.*]] = amdg.in_thread_transpose {{.*}} : tensor<32x128xf16
283| // CHECK:   [[ALLOC1:%.*]] = ttg.local_alloc [[TRANS1]] : {{.*}} !ttg.memdesc<32x128xf16
284| // CHECK:   scf.yield [[ALLOC1]] : !ttg.memdesc<32x128xf16, #shared, #smem>
285| // CHECK: } else {
286| // CHECK:   [[TRANS2:%.*]] = amdg.in_thread_transpose {{.*}} : tensor<32x128xf16
287| // CHECK:   [[ALLOC2:%.*]] = ttg.local_alloc [[TRANS2]] : {{.*}} -> !ttg.memdesc<32x128xf16
288| // CHECK:   scf.yield [[ALLOC2]] : !ttg.memdesc<32x128xf16
289| // CHECK: }
290| // CHECK: ttg.local_load [[IF]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: inThreadTranspose_nested_scf_traversal_regression anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: inThreadTranspose_nested_scf_traversal_regression 这样的标签用于锚定匹配范围。

### Lines 291-294
```mlir
291| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 8], warpsPerCTA = [1, 8], order = [1, 0]}>
292| #shared = #ttg.swizzled_shared<{vec = 4, perPhase = 2, maxPhase = 4, order = [0, 1]}>
293| #smem = #ttg.shared_memory
294| #mma = #ttg.amd_mfma<{version = 3, warpsPerCTA = [4, 2], instrShape = [32, 32, 8], isTransposed = true}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 295-295
```mlir
295| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 296-313
```mlir
296|   tt.func public @inThreadTranspose_nested_scf_traversal_regression(%arg0: tensor<256x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>, %arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg2: i1) {
297|     %cst_0 = arith.constant dense<0.000000e+00> : tensor<256x128xf32, #mma>
298|     %0 = tt.splat %arg1 : !tt.ptr<f16> -> tensor<32x128x!tt.ptr<f16>, #blocked>
299|     %5 = scf.if %arg2 -> (!ttg.memdesc<32x128xf16, #shared, #smem>) {
300|       %10 = scf.if %arg2 -> (tensor<32x128xf16, #blocked>) {
301|         %11 = tt.load %0 : tensor<32x128x!tt.ptr<f16>, #blocked>
302|         scf.yield %11 : tensor<32x128xf16, #blocked>
303|       } else {
304|         %cst_1 = arith.constant dense<0.000000e+00> : tensor<32x128xf16, #blocked>
305|         scf.yield %cst_1 : tensor<32x128xf16, #blocked>
306|       }
307|       %2 = ttg.local_alloc %10 : (tensor<32x128xf16, #blocked>) -> !ttg.memdesc<32x128xf16, #shared, #smem>
308|       scf.yield %2 : !ttg.memdesc<32x128xf16, #shared, #smem>
309|     } else {
310|       %3 = tt.load %0 : tensor<32x128x!tt.ptr<f16>, #blocked>
311|       %4 = ttg.local_alloc %3 : (tensor<32x128xf16, #blocked>) -> !ttg.memdesc<32x128xf16, #shared, #smem>
312|       scf.yield %4 : !ttg.memdesc<32x128xf16, #shared, #smem>
313|     }
```
**EN:** This function-oriented block defines or enters `inThreadTranspose_nested_scf_traversal_regression`. Within it, the test exercises loop/if yielded values, tt.func, constants, structured conditionals, masked or vectorized loads, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `inThreadTranspose_nested_scf_traversal_regression` 为核心。测试在其中演示 循环/分支产出值、tt.func、常量、结构化条件分支、带掩码或向量化的加载，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 314-319
```mlir
314|     %6 = ttg.local_load %5 : !ttg.memdesc<32x128xf16, #shared, #smem> -> tensor<32x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>
315|     %7 = tt.dot %arg0, %6, %cst_0 : tensor<256x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>> * tensor<32x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>> -> tensor<256x128xf32, #mma>
316|     tt.return
317|   }
318| }
319| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on local/shared memory loads, dot-product or MMA-style math, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 本地/共享内存加载、点积或 MMA 风格计算、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 320-320
```mlir
320| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 321-328
```mlir
321| 
322| // Test that ITT does not crash on following Data flow:
323| //
324| // %v = define mem ref
325| // while (%arg = %v) {
326| //   use %arg
327| // }
328| //
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 329-336
```mlir
329| // CHECK-LABEL: inThreadTranspose_inbound_df_while_regression
330| // CHECK: [[TRANS1:%.*]] = amdg.in_thread_transpose
331| // CHECK: ttg.local_alloc [[TRANS1]] : (tensor<32x128xf16
332| // CHECK: scf.while
333| // CHECK: } do {
334| // CHECK:  [[TRANS2:%.*]] = amdg.in_thread_transpose
335| // CHECK:  ttg.local_store [[TRANS2]], {{.*}} : tensor<32x128xf16
336| // CHECK: }
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: inThreadTranspose_inbound_df_while_regression anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: inThreadTranspose_inbound_df_while_regression 这样的标签用于锚定匹配范围。

### Lines 337-340
```mlir
337| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 8], warpsPerCTA = [1, 8], order = [1, 0]}>
338| #shared = #ttg.swizzled_shared<{vec = 4, perPhase = 2, maxPhase = 4, order = [0, 1]}>
339| #smem = #ttg.shared_memory
340| #mma = #ttg.amd_mfma<{version = 3, warpsPerCTA = [4, 2], instrShape = [32, 32, 8], isTransposed = true}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 341-341
```mlir
341| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 342-359
```mlir
342|   tt.func public @inThreadTranspose_inbound_df_while_regression(%arg0: tensor<256x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>, %arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg2: i1) {
343|     %cst_0 = arith.constant dense<0.000000e+00> : tensor<256x128xf32, #mma>
344|     %0 = tt.splat %arg1 : !tt.ptr<f16> -> tensor<32x128x!tt.ptr<f16>, #blocked>
345|     %1 = tt.load %0 : tensor<32x128x!tt.ptr<f16>, #blocked>
346|     %2 = ttg.local_alloc %1 : (tensor<32x128xf16, #blocked>) -> !ttg.memdesc<32x128xf16, #shared, #smem, mutable>
347|     %3:1 = scf.while (%arg10 = %2, %arg11 = %arg2) : (!ttg.memdesc<32x128xf16, #shared, #smem, mutable>, i1) -> (!ttg.memdesc<32x128xf16, #shared, #smem, mutable>) {
348|       scf.condition(%arg11) %arg10 : !ttg.memdesc<32x128xf16, #shared, #smem, mutable>
349|     } do {
350|     ^bb0(%arg20: !ttg.memdesc<32x128xf16, #shared, #smem, mutable>):
351|       %10 = tt.load %0 : tensor<32x128x!tt.ptr<f16>, #blocked>
352|       %11 = ttg.local_load %arg20 : !ttg.memdesc<32x128xf16, #shared, #smem, mutable> -> tensor<32x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>
353|       ttg.local_store %10, %arg20 : tensor<32x128xf16, #blocked> -> !ttg.memdesc<32x128xf16, #shared, #smem, mutable>
354|       %12 = tt.dot %arg0, %11, %cst_0 : tensor<256x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>> * tensor<32x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>> -> tensor<256x128xf32, #mma>
355|       scf.yield %arg20, %arg2 : !ttg.memdesc<32x128xf16, #shared, #smem, mutable>, i1
356|     }
357|     tt.return
358|   }
359| }
```
**EN:** This function-oriented block defines or enters `inThreadTranspose_inbound_df_while_regression`. Within it, the test exercises tt.func, masked or vectorized loads, constants, broadcasted scalars or pointers, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `inThreadTranspose_inbound_df_while_regression` 为核心。测试在其中演示 tt.func、带掩码或向量化的加载、常量、广播后的标量或指针、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 361-362
```mlir
361| 
362| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 362-370
```mlir
362| 
363| // Test that ITT does not crash on following Data flow:
364| //
365| // %w = while () {
366| //   %v = define mem ref
367| //   yield %v
368| // }
369| // use %w
370| //
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 371-378
```mlir
371| // CHECK-LABEL: inThreadTranspose_outbound_df_while_regression
372| // CHECK: [[TRANS1:%.*]] = amdg.in_thread_transpose
373| // CHECK: ttg.local_alloc [[TRANS1]] : (tensor<32x128xf16
374| // CHECK: scf.while
375| // CHECK: } do {
376| // CHECK: }
377| // CHECK: [[TRANS2:%.*]] = amdg.in_thread_transpose
378| // CHECK: ttg.local_store [[TRANS2]], {{.*}} : tensor<32x128xf16
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: inThreadTranspose_outbound_df_while_regression anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: inThreadTranspose_outbound_df_while_regression 这样的标签用于锚定匹配范围。

### Lines 379-382
```mlir
379| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 8], warpsPerCTA = [1, 8], order = [1, 0]}>
380| #shared = #ttg.swizzled_shared<{vec = 4, perPhase = 2, maxPhase = 4, order = [0, 1]}>
381| #smem = #ttg.shared_memory
382| #mma = #ttg.amd_mfma<{version = 3, warpsPerCTA = [4, 2], instrShape = [32, 32, 8], isTransposed = true}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 383-383
```mlir
383| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 384-401
```mlir
384|   tt.func public @inThreadTranspose_outbound_df_while_regression(%arg0: tensor<256x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>, %arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg2: i1) {
385|     %cst_0 = arith.constant dense<0.000000e+00> : tensor<256x128xf32, #mma>
386|     %0 = tt.splat %arg1 : !tt.ptr<f16> -> tensor<32x128x!tt.ptr<f16>, #blocked>
387|     %1 = tt.load %0 : tensor<32x128x!tt.ptr<f16>, #blocked>
388|     %2 = ttg.local_alloc %1 : (tensor<32x128xf16, #blocked>) -> !ttg.memdesc<32x128xf16, #shared, #smem, mutable>
389|     %3:1 = scf.while (%arg10 = %2, %arg11 = %arg2) : (!ttg.memdesc<32x128xf16, #shared, #smem, mutable>, i1) -> (!ttg.memdesc<32x128xf16, #shared, #smem, mutable>) {
390|       scf.condition(%arg11) %arg10 : !ttg.memdesc<32x128xf16, #shared, #smem, mutable>
391|     } do {
392|     ^bb0(%arg20: !ttg.memdesc<32x128xf16, #shared, #smem, mutable>):
393|       scf.yield %arg20, %arg2 : !ttg.memdesc<32x128xf16, #shared, #smem, mutable>, i1
394|     }
395|     ttg.local_store %1, %3#0 : tensor<32x128xf16, #blocked> -> !ttg.memdesc<32x128xf16, #shared, #smem, mutable>
396|     %4 = ttg.local_load %3#0 : !ttg.memdesc<32x128xf16, #shared, #smem, mutable> -> tensor<32x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>
397|     %5 = tt.dot %arg0, %4, %cst_0 : tensor<256x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>> * tensor<32x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>> -> tensor<256x128xf32, #mma>
398|     tt.return
399|   }
400| }
401| 
```
**EN:** This function-oriented block defines or enters `inThreadTranspose_outbound_df_while_regression`. Within it, the test exercises tt.func, constants, broadcasted scalars or pointers, masked or vectorized loads, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `inThreadTranspose_outbound_df_while_regression` 为核心。测试在其中演示 tt.func、常量、广播后的标量或指针、带掩码或向量化的加载、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 402-402
```mlir
402| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 403-410
```mlir
403| 
404| // Test that ITT does not crash on following Data flow:
405| //
406| // %v = define mem ref
407| // for (%arg = %v) {
408| //   use %arg
409| // }
410| //
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 411-417
```mlir
411| // CHECK-LABEL: inThreadTranspose_inbound_df_for_regression
412| // CHECK: [[TRANS1:%.*]] = amdg.in_thread_transpose
413| // CHECK: ttg.local_alloc [[TRANS1]] : (tensor<32x128xf16
414| // CHECK: scf.for
415| // CHECK:   [[TRANS2:%.*]] = amdg.in_thread_transpose
416| // CHECK:   ttg.local_store [[TRANS2]], {{.*}} : tensor<32x128xf16
417| // CHECK: }
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: inThreadTranspose_inbound_df_for_regression anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: inThreadTranspose_inbound_df_for_regression 这样的标签用于锚定匹配范围。

### Lines 418-421
```mlir
418| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 8], warpsPerCTA = [1, 8], order = [1, 0]}>
419| #shared = #ttg.swizzled_shared<{vec = 4, perPhase = 2, maxPhase = 4, order = [0, 1]}>
420| #smem = #ttg.shared_memory
421| #mma = #ttg.amd_mfma<{version = 3, warpsPerCTA = [4, 2], instrShape = [32, 32, 8], isTransposed = true}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 422-422
```mlir
422| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 423-439
```mlir
423|   tt.func public @inThreadTranspose_inbound_df_for_regression(%arg0: tensor<256x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>, %arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg2: i1) {
424|     %c0_i32 = arith.constant 0 : i32
425|     %c1_i32 = arith.constant 0 : i32
426|     %c10_i32 = arith.constant 10 : i32
427|     %0 = tt.splat %arg1 : !tt.ptr<f16> -> tensor<32x128x!tt.ptr<f16>, #blocked>
428|     %1 = tt.load %0 : tensor<32x128x!tt.ptr<f16>, #blocked>
429|     %2 = ttg.local_alloc %1 : (tensor<32x128xf16, #blocked>) -> !ttg.memdesc<32x128xf16, #shared, #smem, mutable>
430|     %3:1 = scf.for %arg10 = %c0_i32 to %c10_i32 step %c1_i32 iter_args(%arg11 = %2) -> (!ttg.memdesc<32x128xf16, #shared, #smem, mutable>) : i32 {
431|       %10 = tt.load %0 : tensor<32x128x!tt.ptr<f16>, #blocked>
432|       %11 = ttg.local_load %arg11 : !ttg.memdesc<32x128xf16, #shared, #smem, mutable> -> tensor<32x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>
433|       ttg.local_store %10, %arg11 : tensor<32x128xf16, #blocked> -> !ttg.memdesc<32x128xf16, #shared, #smem, mutable>
434|       scf.yield %arg11 : !ttg.memdesc<32x128xf16, #shared, #smem, mutable>
435|     }
436|     tt.return
437|   }
438| }
439| 
```
**EN:** This function-oriented block defines or enters `inThreadTranspose_inbound_df_for_regression`. Within it, the test exercises constants, tt.func, masked or vectorized loads, broadcasted scalars or pointers, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `inThreadTranspose_inbound_df_for_regression` 为核心。测试在其中演示 常量、tt.func、带掩码或向量化的加载、广播后的标量或指针、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 440-440
```mlir
440| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 441-449
```mlir
441| 
442| // Test that ITT does not crash on following Data flow:
443| //
444| // %f = for () {
445| //   %v = define mem ref
446| //   yield %v
447| // }
448| // use %f
449| //
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 450-455
```mlir
450| // CHECK-LABEL: inThreadTranspose_outbound_df_for_regression
451| // CHECK: scf.for
452| // CHECK:   [[TRANS:%.*]] = amdg.in_thread_transpose
453| // CHECK:   ttg.local_store [[TRANS]], {{.*}} : tensor<32x128xf16
454| // CHECK: }
455| // CHECK: ttg.local_load
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: inThreadTranspose_outbound_df_for_regression anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: inThreadTranspose_outbound_df_for_regression 这样的标签用于锚定匹配范围。

### Lines 456-459
```mlir
456| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 8], warpsPerCTA = [1, 8], order = [1, 0]}>
457| #shared = #ttg.swizzled_shared<{vec = 4, perPhase = 2, maxPhase = 4, order = [0, 1]}>
458| #smem = #ttg.shared_memory
459| #mma = #ttg.amd_mfma<{version = 3, warpsPerCTA = [4, 2], instrShape = [32, 32, 8], isTransposed = true}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 460-460
```mlir
460| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 461-477
```mlir
461|   tt.func public @inThreadTranspose_outbound_df_for_regression(%arg0: tensor<256x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>, %arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg2: i1) {
462|     %c0_i32 = arith.constant 0 : i32
463|     %c1_i32 = arith.constant 0 : i32
464|     %c10_i32 = arith.constant 10 : i32
465|     %0 = tt.splat %arg1 : !tt.ptr<f16> -> tensor<32x128x!tt.ptr<f16>, #blocked>
466|     %1 = ttg.local_alloc  : () -> !ttg.memdesc<32x128xf16, #shared, #smem, mutable>
467|     %2:1 = scf.for %arg10 = %c0_i32 to %c10_i32 step %c1_i32 iter_args(%arg11 = %1) -> (!ttg.memdesc<32x128xf16, #shared, #smem, mutable>) : i32 {
468|       %10 = tt.load %0 : tensor<32x128x!tt.ptr<f16>, #blocked>
469|       %11 = ttg.local_load %arg11 : !ttg.memdesc<32x128xf16, #shared, #smem, mutable> -> tensor<32x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>
470|       ttg.local_store %10, %arg11 : tensor<32x128xf16, #blocked> -> !ttg.memdesc<32x128xf16, #shared, #smem, mutable>
471|       scf.yield %arg11 : !ttg.memdesc<32x128xf16, #shared, #smem, mutable>
472|     }
473|     %3 = ttg.local_load %2#0 : !ttg.memdesc<32x128xf16, #shared, #smem, mutable> -> tensor<32x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>
474|     tt.return
475|   }
476| }
477| 
```
**EN:** This function-oriented block defines or enters `inThreadTranspose_outbound_df_for_regression`. Within it, the test exercises constants, tt.func, local/shared memory loads, broadcasted scalars or pointers, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `inThreadTranspose_outbound_df_for_regression` 为核心。测试在其中演示 常量、tt.func、本地/共享内存加载、广播后的标量或指针、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 478-478
```mlir
478| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 479-490
```mlir
479| 
480| // Test that ITT does not crash on following Data flow:
481| //
482| // %i = if () {
483| //   %v1 = define mem ref
484| //   yield %v1
485| // } else {
486| //   %v2 = define mem ref
487| //   yield %v2
488| // }
489| // use %i
490| //
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 491-501
```mlir
491| // CHECK-LABEL: inThreadTranspose_outbound_df_for_regression
492| // CHECK: [[IF:%.*]] = scf.if
493| // CHECK:   [[ALLOC1:%.*]] = ttg.local_alloc : () -> !ttg.memdesc<32x128xf16
494| // CHECK:   scf.yield [[ALLOC1]]
495| // CHECK: } else {
496| // CHECK:   [[ALLOC2:%.*]] = ttg.local_alloc : () -> !ttg.memdesc<32x128xf16
497| // CHECK:   scf.yield [[ALLOC2]]
498| // CHECK: }
499| // CHECK: [[TRANS:%.*]] = amdg.in_thread_transpose
500| // CHECK: ttg.local_store [[TRANS]], [[IF]] : tensor<32x128xf16
501| // CHECK: ttg.local_load [[IF]] : !ttg.memdesc<32x128xf16
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: inThreadTranspose_outbound_df_for_regression anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: inThreadTranspose_outbound_df_for_regression 这样的标签用于锚定匹配范围。

### Lines 502-505
```mlir
502| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 8], warpsPerCTA = [1, 8], order = [1, 0]}>
503| #shared = #ttg.swizzled_shared<{vec = 4, perPhase = 2, maxPhase = 4, order = [0, 1]}>
504| #smem = #ttg.shared_memory
505| #mma = #ttg.amd_mfma<{version = 3, warpsPerCTA = [4, 2], instrShape = [32, 32, 8], isTransposed = true}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 506-506
```mlir
506| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 507-522
```mlir
507|   tt.func public @inThreadTranspose_outbound_df_for_regression(%arg0: tensor<256x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>, %arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg2: i1) {
508|     %0 = scf.if %arg2 -> (!ttg.memdesc<32x128xf16, #shared, #smem, mutable>) {
509|       %1 = ttg.local_alloc  : () -> !ttg.memdesc<32x128xf16, #shared, #smem, mutable>
510|       scf.yield %1 : !ttg.memdesc<32x128xf16, #shared, #smem, mutable>
511|     } else {
512|       %2 = ttg.local_alloc  : () -> !ttg.memdesc<32x128xf16, #shared, #smem, mutable>
513|       scf.yield %2 : !ttg.memdesc<32x128xf16, #shared, #smem, mutable>
514|     }
515|     %3 = tt.splat %arg1 : !tt.ptr<f16> -> tensor<32x128x!tt.ptr<f16>, #blocked>
516|     %4 = tt.load %3: tensor<32x128x!tt.ptr<f16>, #blocked>
517|     ttg.local_store %4, %0 : tensor<32x128xf16, #blocked> -> !ttg.memdesc<32x128xf16, #shared, #smem, mutable>
518|     %5 = ttg.local_load %0 : !ttg.memdesc<32x128xf16, #shared, #smem, mutable> -> tensor<32x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>
519|     tt.return
520|   }
521| }
522| 
```
**EN:** This function-oriented block defines or enters `inThreadTranspose_outbound_df_for_regression`. Within it, the test exercises tt.func, shared/local memory allocation, loop/if yielded values, structured conditionals, broadcasted scalars or pointers, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `inThreadTranspose_outbound_df_for_regression` 为核心。测试在其中演示 tt.func、共享/本地内存分配、循环/分支产出值、结构化条件分支、广播后的标量或指针，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 523-523
```mlir
523| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 524-524
```mlir
524| // Test that ITT is not used for direct-to-lds loads
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 525-528
```mlir
525| // CHECK-LABEL: inThreadTranspose_async_copy
526| // CHECK-NOT: amdg.in_thread_transpose
527| // CHECK: tt.return
528| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: inThreadTranspose_async_copy anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: inThreadTranspose_async_copy 这样的标签用于锚定匹配范围。

### Lines 529-533
```mlir
529| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 8], warpsPerCTA = [1, 8], order = [1, 0]}>
530| #blocked1 = #ttg.blocked<{sizePerThread = [8, 1], threadsPerWarp = [8, 8], warpsPerCTA = [8, 1], order = [0, 1]}>
531| #mma = #ttg.amd_mfma<{version = 3, warpsPerCTA = [4, 2], instrShape = [32, 32, 8], isTransposed = true}>
532| #shared = #ttg.swizzled_shared<{vec = 4, perPhase = 2, maxPhase = 4, order = [0, 1]}>
533| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 534-534
```mlir
534| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 535-548
```mlir
535|   tt.func public @inThreadTranspose_async_copy(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
536|     %cst = arith.constant dense<0.000000e+00> : tensor<256x128xf32, #mma>
537|     %cst_0 = arith.constant dense<0> : tensor<32x128xi32, #blocked>
538|     %0 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<256x32x!tt.ptr<f16>, #blocked1>
539|     %1 = ttg.local_alloc : () -> !ttg.memdesc<256x32xf16, #shared, #smem, mutable>
540|     %2 = ttg.async_copy_global_to_local %0, %1 : tensor<256x32x!tt.ptr<f16>, #blocked1> -> <256x32xf16, #shared, #smem, mutable>
541|     %3 = ttg.local_load %1 : !ttg.memdesc<256x32xf16, #shared, #smem, mutable> -> tensor<256x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>
542|     %4 = ttg.local_alloc : () -> !ttg.memdesc<32x128xf16, #shared, #smem, mutable>
543|     %5 = amdg.buffer_load_to_local %arg1[%cst_0] into %4 : <f16>[tensor<32x128xi32, #blocked>]  -> <32x128xf16, #shared, #smem, mutable>
544|     %6 = ttg.local_load %4 : !ttg.memdesc<32x128xf16, #shared, #smem, mutable> -> tensor<32x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>
545|     %7 = tt.dot %3, %6, %cst : tensor<256x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>> * tensor<32x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>> -> tensor<256x128xf32, #mma>
546|     tt.return
547|   }
548| }
```
**EN:** This function-oriented block defines or enters `inThreadTranspose_async_copy`. Within it, the test exercises tt.func, constants, shared/local memory allocation, local/shared memory loads, broadcasted scalars or pointers, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `inThreadTranspose_async_copy` 为核心。测试在其中演示 tt.func、常量、共享/本地内存分配、本地/共享内存加载、广播后的标量或指针，这些操作构成了被优化、lower 或验证的主要数据流。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `-tritonamdgpu-in-thread-transpose`
- **CN:** 主要 pass 选项：`-split-input-file`，`-tritonamdgpu-in-thread-transpose`
- **EN:** Dominant operations include `tt.func`, `arith.constant`, `tt.load`, `ttg.local_load`, `ttg.local_alloc`, `tt.splat`, `scf.yield`, `module`, `tt.return`, `tt.dot`.
- **CN:** 主要操作包括 `tt.func`、`arith.constant`、`tt.load`、`ttg.local_load`、`ttg.local_alloc`、`tt.splat`、`scf.yield`、`module`、`tt.return`、`tt.dot`。
- **EN:** The file contains 11 independently testable section(s). Check styles used: CHECK-DAG x59, CHECK x55, CHECK-LABEL x12, CHECK-NOT x9. Important labels include inThreadTranspose_simple, inThreadTranspose_k_fast_neg, inThreadTranspose_small_k_neg, inThreadTranspose_with_cfg. It validates both the presence of the desired rewrite and the absence of unwanted residual operations.
- **CN:** 该文件包含 11 个可独立测试的分段。使用的检查类型：CHECK-DAG ×59，CHECK ×55，CHECK-LABEL ×12，CHECK-NOT ×9。 关键标签包括 inThreadTranspose_simple，inThreadTranspose_k_fast_neg，inThreadTranspose_small_k_neg，inThreadTranspose_with_cfg。 它同时验证目标重写已经出现，并且不再残留不需要的操作。

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