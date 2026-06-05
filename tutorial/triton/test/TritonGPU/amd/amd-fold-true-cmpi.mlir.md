# amd-fold-true-cmpi.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/amd/amd-fold-true-cmpi.mlir`
- **EN:** Test for the MLIR/Triton pipeline `-tritonamdgpu-fold-true-cmpi, -canonicalize` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `-tritonamdgpu-fold-true-cmpi, -canonicalize` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -split-input-file -allow-unregistered-dialect -tritonamdgpu-fold-true-cmpi -canonicalize | FileCheck %s`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -split-input-file -allow-unregistered-dialect -tritonamdgpu-fold-true-cmpi -canonicalize | FileCheck %s`

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
1| // RUN: triton-opt %s -split-input-file -allow-unregistered-dialect -tritonamdgpu-fold-true-cmpi -canonicalize | FileCheck %s
2| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -split-input-file -allow-unregistered-dialect -tritonamdgpu-fold-true-cmpi -canonicalize | FileCheck %s` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -split-input-file -allow-unregistered-dialect -tritonamdgpu-fold-true-cmpi -canonicalize | FileCheck %s`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 3-3
```mlir
3| module attributes {"ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 4-11
```mlir
 4|   tt.func @cmpsle(%arg0: !tt.ptr<f32>) -> i1 {
 5|     %c0 = arith.constant 0 : i32
 6|     %c1024_i32 = arith.constant 1024 : i32
 7|     %cmpsle = arith.cmpi sle, %c0, %c1024_i32 : i32
 8|     tt.return %cmpsle: i1
 9|   }
10| }
11| 
```
**EN:** This function-oriented block defines or enters `cmpsle`. Within it, the test exercises tt.func, constants, integer comparisons, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `cmpsle` 为核心。测试在其中演示 tt.func、常量、整数比较、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 12-17
```mlir
12| // CHECK-LABEL:   tt.func @cmpsle(
13| // CHECK-SAME:                       %[[VAL_0:.*]]: !tt.ptr<f32>) -> i1 {
14| // CHECK:           %[[VAL_1:.*]] = arith.constant true
15| // CHECK:           tt.return %[[VAL_1]] : i1
16| // CHECK:         }
17| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL:   tt.func @cmpsle( anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL:   tt.func @cmpsle( 这样的标签用于锚定匹配范围。

### Lines 18-18
```mlir
18| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 19-20
```mlir
19| 
20| module attributes {"ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 21-36
```mlir
21|   tt.func @assumepid(%arg0: !tt.ptr<f32>) -> tensor<1024xf32> {
22|     %c0 = arith.constant 0 : i32
23|     %c1024_i32 = arith.constant 1024 : i32
24|     %pid = tt.get_program_id x : i32
25|     %cmpsle = arith.cmpi sle, %pid, %c1024_i32 : i32
26|     llvm.intr.assume %cmpsle : i1
27|     %cmpsge = arith.cmpi sge, %pid, %c0 : i32
28|     llvm.intr.assume %cmpsge : i1
29|     %1 = arith.muli %pid, %c1024_i32 : i32
30|     %2 = tt.addptr %arg0, %1 : !tt.ptr<f32>, i32
31|     %3 = tt.splat %2 : !tt.ptr<f32> -> tensor<1024x!tt.ptr<f32>>
32|     %4 = tt.load %3 : tensor<1024x!tt.ptr<f32>>
33|     tt.return %4 : tensor<1024xf32>
34|   }
35| }
36| 
```
**EN:** This function-oriented block defines or enters `assumepid`. Within it, the test exercises tt.func, constants, integer comparisons, llvm.intr, program IDs, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `assumepid` 为核心。测试在其中演示 tt.func、常量、整数比较、llvm.intr、程序 ID，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 37-50
```mlir
37| // CHECK-LABEL:   tt.func @assumepid(
38| // CHECK-SAME:                       %[[VAL_0:.*]]: !tt.ptr<f32>) -> tensor<1024xf32> {
39| // CHECK:           %[[VAL_1:.*]] = arith.constant true
40| // CHECK:           %[[VAL_2:.*]] = arith.constant 1024 : i32
41| // CHECK:           %[[VAL_3:.*]] = tt.get_program_id x : i32
42| // CHECK:           llvm.intr.assume %[[VAL_1]] : i1
43| // CHECK:           llvm.intr.assume %[[VAL_1]] : i1
44| // CHECK:           %[[VAL_4:.*]] = arith.muli %[[VAL_3]], %[[VAL_2]] : i32
45| // CHECK:           %[[VAL_5:.*]] = tt.addptr %[[VAL_0]], %[[VAL_4]] : !tt.ptr<f32>, i32
46| // CHECK:           %[[VAL_6:.*]] = tt.splat %[[VAL_5]] : !tt.ptr<f32> -> tensor<1024x!tt.ptr<f32>>
47| // CHECK:           %[[VAL_7:.*]] = tt.load %[[VAL_6]] : tensor<1024x!tt.ptr<f32>>
48| // CHECK:           tt.return %[[VAL_7]] : tensor<1024xf32>
49| // CHECK:         }
50| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL:   tt.func @assumepid( anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL:   tt.func @assumepid( 这样的标签用于锚定匹配范围。

### Lines 51-51
```mlir
51| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 52-58
```mlir
52| 
53| #blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
54| #blocked1 = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
55| #mma = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 8]}>
56| #shared = #ttg.swizzled_shared<{vec = 8, perPhase = 2, maxPhase = 4, order = [1, 0]}>
57| #shared1 = #ttg.swizzled_shared<{vec = 8, perPhase = 1, maxPhase = 8, order = [1, 0]}>
58| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 59-59
```mlir
59| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 60-77
```mlir
60|   tt.func @assume_matmul(%arg0: index, %arg1: index, %arg2: index, %arg3: !tt.ptr<f16>, %arg4: !tt.ptr<f16>) -> tensor<128x128xf32, #mma> {
61|     %c-1 = arith.constant -1 : index
62|     %c1 = arith.constant 1 : index
63|     %c0 = arith.constant 0 : index
64|     %c1_i32 = arith.constant 1 : i32
65|     %c0_i32 = arith.constant 0 : i32
66|     %true = arith.constant true
67|     %cst = arith.constant dense<4.000000e+00> : tensor<32x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>
68|     %cst_0 = arith.constant dense<4> : tensor<32x128xi32, #blocked>
69|     %cst_1 = arith.constant dense<4> : tensor<128x32xi32, #blocked1>
70|     %cst_2 = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #mma>
71|     %cst_3 = arith.constant dense<0.000000e+00> : tensor<32x128xf16, #blocked>
72|     %0 = tt.splat %arg3 : !tt.ptr<f16> -> tensor<128x32x!tt.ptr<f16>, #blocked1>
73|     %1 = tt.make_range {end = 32 : i32, start = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>
74|     %2 = tt.expand_dims %1 {axis = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 0, parent = #blocked1}>> -> tensor<1x32xi32, #blocked1>
75|     %3 = tt.broadcast %2 : tensor<1x32xi32, #blocked1> -> tensor<128x32xi32, #blocked1>
76|     %4 = tt.addptr %0, %3 : tensor<128x32x!tt.ptr<f16>, #blocked1>, tensor<128x32xi32, #blocked1>
77|     %5 = tt.splat %arg4 : !tt.ptr<f16> -> tensor<32x128x!tt.ptr<f16>, #blocked>
```
**EN:** This function-oriented block defines or enters `assume_matmul`. Within it, the test exercises constants, tt.func, broadcasted scalars or pointers, lane/block index ranges, shape expansion, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `assume_matmul` 为核心。测试在其中演示 常量、tt.func、广播后的标量或指针、lane/block 索引范围、形状扩展，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 78-95
```mlir
78|     %6 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
79|     %7 = tt.expand_dims %6 {axis = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x128xi32, #blocked>
80|     %8 = tt.broadcast %7 : tensor<1x128xi32, #blocked> -> tensor<32x128xi32, #blocked>
81|     %9 = tt.addptr %5, %8 : tensor<32x128x!tt.ptr<f16>, #blocked>, tensor<32x128xi32, #blocked>
82|     %10 = ttg.local_alloc : () -> !ttg.memdesc<1x128x32xf16, #shared, #smem, mutable>
83|     %11 = ttg.local_alloc : () -> !ttg.memdesc<1x32x128xf16, #shared1, #smem, mutable>
84|     %12 = arith.cmpi slt, %arg0, %arg1 : index
85|     %13 = tt.splat %12 : i1 -> tensor<128x32xi1, #blocked1>
86|     %14 = tt.load %4, %13 : tensor<128x32x!tt.ptr<f16>, #blocked1>
87|     %15 = tt.splat %12 : i1 -> tensor<32x128xi1, #blocked>
88|     %16 = tt.load %9, %15, %cst_3 : tensor<32x128x!tt.ptr<f16>, #blocked>
89|     %17 = ttg.memdesc_index %10[%c0_i32] : !ttg.memdesc<1x128x32xf16, #shared, #smem, mutable> -> !ttg.memdesc<128x32xf16, #shared, #smem, mutable>
90|     ttg.local_store %14, %17 : tensor<128x32xf16, #blocked1> -> !ttg.memdesc<128x32xf16, #shared, #smem, mutable>
91|     %18 = ttg.memdesc_index %11[%c0_i32] : !ttg.memdesc<1x32x128xf16, #shared1, #smem, mutable> -> !ttg.memdesc<32x128xf16, #shared1, #smem, mutable>
92|     ttg.local_store %16, %18 : tensor<32x128xf16, #blocked> -> !ttg.memdesc<32x128xf16, #shared1, #smem, mutable>
93|     %19 = arith.subi %arg1, %arg2 : index
94|     %20:6 = scf.for %arg5 = %arg0 to %19 step %arg2 iter_args(%arg6 = %4, %arg7 = %9, %arg8 = %cst_2, %arg9 = %c0_i32, %arg10 = %17, %arg11 = %18) -> (tensor<128x32x!tt.ptr<f16>, #blocked1>, tensor<32x128x!tt.ptr<f16>, #blocked>, tensor<128x128xf32, #mma>, i32, !ttg.memdesc<128x32xf16, #shared, #smem, mutable>, !ttg.memdesc<32x128xf16, #shared1, #smem, mutable>) {
95|       %33 = tt.addptr %arg6, %cst_1 : tensor<128x32x!tt.ptr<f16>, #blocked1>, tensor<128x32xi32, #blocked1>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining pointer arithmetic, shared/local memory allocation, broadcasted scalars or pointers, masked or vectorized loads, ttg.memdesc_index.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 指针算术、共享/本地内存分配、广播后的标量或指针、带掩码或向量化的加载、ttg.memdesc_index。

### Lines 96-113
```mlir
 96|       %34 = tt.addptr %arg7, %cst_0 : tensor<32x128x!tt.ptr<f16>, #blocked>, tensor<32x128xi32, #blocked>
 97|       llvm.intr.assume %true : i1
 98|       %35 = tt.load %33 : tensor<128x32x!tt.ptr<f16>, #blocked1>
 99|       %36 = ttg.local_load %arg10 : !ttg.memdesc<128x32xf16, #shared, #smem, mutable> -> tensor<128x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
100|       %37 = tt.load %34 : tensor<32x128x!tt.ptr<f16>, #blocked>
101|       %38 = ttg.local_load %arg11 : !ttg.memdesc<32x128xf16, #shared1, #smem, mutable> -> tensor<32x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>
102|       %39 = arith.mulf %38, %cst : tensor<32x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>
103|       %40 = tt.dot %36, %39, %arg8 : tensor<128x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * tensor<32x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>> -> tensor<128x128xf32, #mma>
104|       %41 = arith.addi %arg9, %c1_i32 : i32
105|       %42 = arith.cmpi slt, %41, %c1_i32 : i32
106|       %43 = arith.select %42, %41, %c0_i32 : i32
107|       %44 = ttg.memdesc_index %10[%43] : !ttg.memdesc<1x128x32xf16, #shared, #smem, mutable> -> !ttg.memdesc<128x32xf16, #shared, #smem, mutable>
108|       ttg.local_store %35, %44 : tensor<128x32xf16, #blocked1> -> !ttg.memdesc<128x32xf16, #shared, #smem, mutable>
109|       %45 = ttg.memdesc_index %11[%43] : !ttg.memdesc<1x32x128xf16, #shared1, #smem, mutable> -> !ttg.memdesc<32x128xf16, #shared1, #smem, mutable>
110|       ttg.local_store %37, %45 : tensor<32x128xf16, #blocked> -> !ttg.memdesc<32x128xf16, #shared1, #smem, mutable>
111|       scf.yield %33, %34, %40, %43, %44, %45 : tensor<128x32x!tt.ptr<f16>, #blocked1>, tensor<32x128x!tt.ptr<f16>, #blocked>, tensor<128x128xf32, #mma>, i32, !ttg.memdesc<128x32xf16, #shared, #smem, mutable>, !ttg.memdesc<32x128xf16, #shared1, #smem, mutable>
112|     }
113|     %21 = arith.cmpi slt, %arg2, %c0 : index
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining masked or vectorized loads, local/shared memory loads, integer comparisons, ttg.memdesc_index, local/shared memory stores.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 带掩码或向量化的加载、本地/共享内存加载、整数比较、ttg.memdesc_index、本地/共享内存存储。

### Lines 114-131
```mlir
114|     %22 = arith.select %21, %c1, %c-1 : index
115|     %23 = arith.subi %arg1, %arg0 : index
116|     %24 = arith.addi %23, %arg2 : index
117|     %25 = arith.addi %24, %22 : index
118|     %26 = arith.divsi %25, %arg2 : index
119|     %28 = ttg.local_load %20#4 : !ttg.memdesc<128x32xf16, #shared, #smem, mutable> -> tensor<128x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
120|     %29 = ttg.local_load %20#5 : !ttg.memdesc<32x128xf16, #shared1, #smem, mutable> -> tensor<32x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>
121|     %30 = arith.mulf %29, %cst : tensor<32x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>
122|     %27 = arith.cmpi sge, %26, %c1 : index
123|     llvm.intr.assume %27 : i1
124|     %31 = scf.if %27 -> (tensor<128x128xf32, #mma>) {
125|       %33 = tt.dot %28, %30, %20#2 : tensor<128x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * tensor<32x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>> -> tensor<128x128xf32, #mma>
126|       scf.yield %33 : tensor<128x128xf32, #mma>
127|     } else {
128|       scf.yield %20#2 : tensor<128x128xf32, #mma>
129|     }
130|     %32 = arith.select %27, %31, %20#2 : tensor<128x128xf32, #mma>
131|     ttg.local_dealloc %10 : !ttg.memdesc<1x128x32xf16, #shared, #smem, mutable>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining arith.select, integer additions, local/shared memory loads, loop/if yielded values, arith.subi.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 arith.select、整数加法、本地/共享内存加载、循环/分支产出值、arith.subi。

### Lines 132-136
```mlir
132|     ttg.local_dealloc %11 : !ttg.memdesc<1x32x128xf16, #shared1, #smem, mutable>
133|     tt.return %32 : tensor<128x128xf32, #mma>
134|   }
135| }
136| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.local_dealloc, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.local_dealloc、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 137-141
```mlir
137| // CHECK: #[[$ATTR_2:.+]] = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 8]}>
138| // CHECK: #[[$ATTR_3:.+]] = #ttg.swizzled_shared<{vec = 8, perPhase = 2, maxPhase = 4, order = [1, 0]}>
139| // CHECK: #[[$ATTR_4:.+]] = #ttg.swizzled_shared<{vec = 8, perPhase = 1, maxPhase = 8, order = [1, 0]}>
140| // CHECK: #[[$ATTR_5:.+]] = #ttg.shared_memory
141| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 142-159
```mlir
142| // CHECK-LABEL:   tt.func @assume_matmul(
143| // CHECK:           %[[VAL_7:.*]] = arith.constant true
144| // CHECK:           %[[VAL_8:.*]] = arith.constant dense<4.000000e+00> : tensor<32x128xf16, #ttg.dot_op<{opIdx = 1, parent = #[[$ATTR_2]], kWidth = 2}>>
145| // CHECK:           %[[VAL_23:.*]] = ttg.local_alloc : () -> !ttg.memdesc<1x128x32xf16, #[[$ATTR_3]], #[[$ATTR_5]], mutable>
146| // CHECK:           %[[VAL_24:.*]] = ttg.local_alloc : () -> !ttg.memdesc<1x32x128xf16, #[[$ATTR_4]], #[[$ATTR_5]], mutable>
147| // CHECK:           %[[VAL_33:.*]]:6 = scf.for
148| // CHECK:             scf.yield
149| // CHECK:           }
150| // CHECK-NEXT:      %[[VAL_54:.*]] = ttg.local_load %[[VAL_55:.*]]#4 : !ttg.memdesc<128x32xf16, #[[$ATTR_3]], #[[$ATTR_5]], mutable> -> tensor<128x32xf16, #ttg.dot_op<{opIdx = 0, parent = #[[$ATTR_2]], kWidth = 2}>>
151| // CHECK-NEXT:      %[[VAL_56:.*]] = ttg.local_load %[[VAL_55]]#5 : !ttg.memdesc<32x128xf16, #[[$ATTR_4]], #[[$ATTR_5]], mutable> -> tensor<32x128xf16, #ttg.dot_op<{opIdx = 1, parent = #[[$ATTR_2]], kWidth = 2}>>
152| // CHECK-NEXT:      %[[VAL_57:.*]] = arith.mulf %[[VAL_56]], %[[VAL_8]] : tensor<32x128xf16, #ttg.dot_op<{opIdx = 1, parent = #[[$ATTR_2]], kWidth = 2}>>
153| // CHECK-NEXT:      llvm.intr.assume %[[VAL_7]] : i1
154| // CHECK-NEXT:      %[[VAL_58:.*]] = tt.dot %[[VAL_54]], %[[VAL_57]], %[[VAL_55]]#2 : tensor<128x32xf16, #ttg.dot_op<{opIdx = 0, parent = #[[$ATTR_2]], kWidth = 2}>> * tensor<32x128xf16, #ttg.dot_op<{opIdx = 1, parent = #[[$ATTR_2]], kWidth = 2}>> -> tensor<128x128xf32, #[[$ATTR_2]]>
155| // CHECK-NEXT:      ttg.local_dealloc %[[VAL_23]] : !ttg.memdesc<1x128x32xf16, #[[$ATTR_3]], #[[$ATTR_5]], mutable>
156| // CHECK-NEXT:      ttg.local_dealloc %[[VAL_24]] : !ttg.memdesc<1x32x128xf16, #[[$ATTR_4]], #[[$ATTR_5]], mutable>
157| // CHECK-NEXT:      tt.return %[[VAL_58]] : tensor<128x128xf32, #[[$ATTR_2]]>
158| // CHECK-NEXT:      }
159| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL:   tt.func @assume_matmul( anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL:   tt.func @assume_matmul( 这样的标签用于锚定匹配范围。

### Lines 160-160
```mlir
160| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 161-164
```mlir
161| 
162| // Tensor-typed cmpi that is statically true should be folded to dense<true>.
163| // make_range(129, 257) elements are [129..256], make_range(0, 128) elements
164| // are [0..127]. Since min(t1)=129 > max(t0)=127, sgt is always true.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 165-165
```mlir
165| module attributes {"ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 166-173
```mlir
166|   tt.func @foldtensorcmpi() -> tensor<128xi1> {
167|     %t0 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32>
168|     %t1 = tt.make_range {end = 257 : i32, start = 129 : i32} : tensor<128xi32>
169|     %cmp = arith.cmpi sgt, %t1, %t0 : tensor<128xi32>
170|     tt.return %cmp: tensor<128xi1>
171|   }
172| }
173| 
```
**EN:** This function-oriented block defines or enters `foldtensorcmpi`. Within it, the test exercises tt.func, lane/block index ranges, integer comparisons, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `foldtensorcmpi` 为核心。测试在其中演示 tt.func、lane/block 索引范围、整数比较、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 174-178
```mlir
174| // CHECK-LABEL:   tt.func @foldtensorcmpi
175| // CHECK:           %[[TRUE:.*]] = arith.constant dense<true> : tensor<128xi1>
176| // CHECK:           tt.return %[[TRUE]] : tensor<128xi1>
177| // CHECK:         }
178| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL:   tt.func @foldtensorcmpi anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL:   tt.func @foldtensorcmpi 这样的标签用于锚定匹配范围。

### Lines 179-179
```mlir
179| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 180-183
```mlir
180| 
181| // Tensor-typed cmpi that is statically false should be folded to dense<false>.
182| // make_range(0, 128) elements are [0..127], make_range(129, 257) elements
183| // are [129..256]. Since max(t0)=127 < min(t1)=129, sgt is always false.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 184-184
```mlir
184| module attributes {"ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 185-192
```mlir
185|   tt.func @foldtensorcmpifalse() -> tensor<128xi1> {
186|     %t0 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32>
187|     %t1 = tt.make_range {end = 257 : i32, start = 129 : i32} : tensor<128xi32>
188|     %cmp = arith.cmpi sgt, %t0, %t1 : tensor<128xi32>
189|     tt.return %cmp: tensor<128xi1>
190|   }
191| }
192| 
```
**EN:** This function-oriented block defines or enters `foldtensorcmpifalse`. Within it, the test exercises tt.func, lane/block index ranges, integer comparisons, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `foldtensorcmpifalse` 为核心。测试在其中演示 tt.func、lane/block 索引范围、整数比较、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 193-197
```mlir
193| // CHECK-LABEL:   tt.func @foldtensorcmpifalse
194| // CHECK:           %[[FALSE:.*]] = arith.constant dense<false> : tensor<128xi1>
195| // CHECK:           tt.return %[[FALSE]] : tensor<128xi1>
196| // CHECK:         }
197| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL:   tt.func @foldtensorcmpifalse anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL:   tt.func @foldtensorcmpifalse 这样的标签用于锚定匹配范围。

### Lines 198-198
```mlir
198| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 199-200
```mlir
199| 
200| // Scalar cmpi that is statically false should be folded.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 201-201
```mlir
201| module attributes {"ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 202-209
```mlir
202|   tt.func @cmpsle_false(%arg0: !tt.ptr<f32>) -> i1 {
203|     %c1024 = arith.constant 1024 : i32
204|     %c0 = arith.constant 0 : i32
205|     %cmpsle = arith.cmpi sle, %c1024, %c0 : i32
206|     tt.return %cmpsle: i1
207|   }
208| }
209| 
```
**EN:** This function-oriented block defines or enters `cmpsle_false`. Within it, the test exercises tt.func, constants, integer comparisons, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `cmpsle_false` 为核心。测试在其中演示 tt.func、常量、整数比较、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 210-214
```mlir
210| // CHECK-LABEL:   tt.func @cmpsle_false(
211| // CHECK:           %[[FALSE:.*]] = arith.constant false
212| // CHECK:           tt.return %[[FALSE]] : i1
213| // CHECK:         }
214| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL:   tt.func @cmpsle_false( anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL:   tt.func @cmpsle_false( 这样的标签用于锚定匹配范围。

### Lines 215-215
```mlir
215| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 216-219
```mlir
216| 
217| // Tensor-typed cmpi that is NOT statically determinable should NOT be folded.
218| // make_range(0, 128) elements [0..127] vs make_range(64, 192) elements
219| // [64..191]. Ranges overlap, so slt is not always true or always false.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 220-220
```mlir
220| module attributes {"ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 221-228
```mlir
221|   tt.func @dontfoldtensorcmpi() -> tensor<128xi1> {
222|     %t0 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32>
223|     %t1 = tt.make_range {end = 192 : i32, start = 64 : i32} : tensor<128xi32>
224|     %cmp = arith.cmpi slt, %t0, %t1 : tensor<128xi32>
225|     tt.return %cmp: tensor<128xi1>
226|   }
227| }
228| 
```
**EN:** This function-oriented block defines or enters `dontfoldtensorcmpi`. Within it, the test exercises tt.func, lane/block index ranges, integer comparisons, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `dontfoldtensorcmpi` 为核心。测试在其中演示 tt.func、lane/block 索引范围、整数比较、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 229-234
```mlir
229| // CHECK-LABEL:   tt.func @dontfoldtensorcmpi
230| // CHECK-NOT:       arith.constant dense<true>
231| // CHECK-NOT:       arith.constant dense<false>
232| // CHECK:           arith.cmpi slt
233| // CHECK:         }
234| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL:   tt.func @dontfoldtensorcmpi anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL:   tt.func @dontfoldtensorcmpi 这样的标签用于锚定匹配范围。

### Lines 235-235
```mlir
235| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 236-237
```mlir
236| 
237| // Tensor-typed cmpi with splat constant: [0..31] < 1024 is always true.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 238-238
```mlir
238| module attributes {"ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 239-246
```mlir
239|   tt.func @foldtensorsplatcmpi() -> tensor<32xi1> {
240|     %t0 = tt.make_range {end = 32 : i32, start = 0 : i32} : tensor<32xi32>
241|     %c1024 = arith.constant dense<1024> : tensor<32xi32>
242|     %cmp = arith.cmpi slt, %t0, %c1024 : tensor<32xi32>
243|     tt.return %cmp: tensor<32xi1>
244|   }
245| }
246| 
```
**EN:** This function-oriented block defines or enters `foldtensorsplatcmpi`. Within it, the test exercises tt.func, lane/block index ranges, constants, integer comparisons, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `foldtensorsplatcmpi` 为核心。测试在其中演示 tt.func、lane/block 索引范围、常量、整数比较、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 247-250
```mlir
247| // CHECK-LABEL:   tt.func @foldtensorsplatcmpi
248| // CHECK:           %[[TRUE:.*]] = arith.constant dense<true> : tensor<32xi1>
249| // CHECK:           tt.return %[[TRUE]] : tensor<32xi1>
250| // CHECK:         }
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL:   tt.func @foldtensorsplatcmpi anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL:   tt.func @foldtensorsplatcmpi 这样的标签用于锚定匹配范围。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `-allow-unregistered-dialect`, `-tritonamdgpu-fold-true-cmpi`, `-canonicalize`
- **CN:** 主要 pass 选项：`-split-input-file`，`-allow-unregistered-dialect`，`-tritonamdgpu-fold-true-cmpi`，`-canonicalize`
- **EN:** Dominant operations include `arith.constant`, `tt.func`, `arith.cmpi`, `tt.make_range`, `module`, `tt.return`, `tt.addptr`, `tt.splat`, `tt.load`, `llvm.intr`.
- **CN:** 主要操作包括 `arith.constant`、`tt.func`、`arith.cmpi`、`tt.make_range`、`module`、`tt.return`、`tt.addptr`、`tt.splat`、`tt.load`、`llvm.intr`。
- **EN:** The file contains 7 independently testable section(s). Check styles used: CHECK x39, CHECK-NEXT x9, CHECK-LABEL x8, CHECK-SAME x2. Important labels include tt.func @cmpsle(, tt.func @assumepid(, tt.func @assume_matmul(, tt.func @foldtensorcmpi. It validates both the presence of the desired rewrite and the absence of unwanted residual operations.
- **CN:** 该文件包含 7 个可独立测试的分段。使用的检查类型：CHECK ×39，CHECK-NEXT ×9，CHECK-LABEL ×8，CHECK-SAME ×2。 关键标签包括 tt.func @cmpsle(，tt.func @assumepid(，tt.func @assume_matmul(，tt.func @foldtensorcmpi。 它同时验证目标重写已经出现，并且不再残留不需要的操作。

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
- **EN:** `llvm`: LLVM-style low-level operations and types.
- **CN:** `llvm`：LLVM 风格底层操作与类型。
- **EN:** `FileCheck` is used to match required textual patterns after rewriting/lowering.
- **CN:** `FileCheck` 用于在重写/lower 之后匹配必须出现的文本模式。