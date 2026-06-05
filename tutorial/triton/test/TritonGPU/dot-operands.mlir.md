# dot-operands.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/dot-operands.mlir`
- **EN:** Test for the MLIR/Triton pipeline `-tritongpu-optimize-dot-operands, -canonicalize` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `-tritongpu-optimize-dot-operands, -canonicalize` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -split-input-file -tritongpu-optimize-dot-operands -canonicalize | FileCheck %s`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -split-input-file -tritongpu-optimize-dot-operands -canonicalize | FileCheck %s`

## Line-by-Line Analysis / 逐行分析

### Lines 1-3
```mlir
1| // RUN: triton-opt %s -split-input-file -tritongpu-optimize-dot-operands -canonicalize | FileCheck %s
2| 
3| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -split-input-file -tritongpu-optimize-dot-operands -canonicalize | FileCheck %s` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -split-input-file -tritongpu-optimize-dot-operands -canonicalize | FileCheck %s`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 4-7
```mlir
4| #blockedA = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
5| #blockedB = #ttg.blocked<{sizePerThread = [2, 1], threadsPerWarp = [8, 4], warpsPerCTA = [1, 4], order = [0, 1]}>
6| #mma = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [1, 4], instrShape = [16, 8]}>
7| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 8-8
```mlir
8| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 9-12
```mlir
 9| 
10| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
11| #mma = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [2, 2], instrShape = [16, 8]}>
12| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 13-13
```mlir
13| module attributes {"ttg.target" = "cuda:80", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 14-15
```mlir
14| // CHECK: tt.func @a_impl
15| // CHECK-NOT: %[[SELECT:.*]] = arith.select {{.*}} : tensor<128x128xi1, #ttg.dot_op<{{.*}}>, tensor<128x128xf16, #ttg.dot_op<{{.*}}>
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 16-32
```mlir
16|   tt.func @a_impl(%pa: tensor<128x128x!tt.ptr<f16>, #blocked>) -> tensor<128x128xf32, #mma> {
17|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #mma>
18|     %cst_0 = arith.constant dense<0.000000e+00> : tensor<128x128xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
19|     %cst_3 = arith.constant dense<5> : tensor<128x1xi32, #blocked>
20|     %cst_4 = arith.constant dense<0.000000e+00> : tensor<128x128xf16, #blocked>
21|     %tl = tt.load %pa : tensor<128x128x!tt.ptr<f16>, #blocked>
22|     %tr = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
23|     %te = tt.expand_dims %tr {axis = 1 : i32} : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<128x1xi32, #blocked>
24|     %tc = arith.cmpi slt, %te, %cst_3 : tensor<128x1xi32, #blocked>
25|     %tb = tt.broadcast %tc : tensor<128x1xi1, #blocked> -> tensor<128x128xi1, #blocked>
26|     %ts = arith.select %tb, %tl, %cst_4 : tensor<128x128xi1, #blocked>, tensor<128x128xf16, #blocked>
27|     %conv = ttg.convert_layout %ts : tensor<128x128xf16, #blocked> -> tensor<128x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>
28|     %td = tt.dot %cst_0, %conv, %cst : tensor<128x128xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * tensor<128x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>> -> tensor<128x128xf32, #mma>
29|     tt.return %td : tensor<128x128xf32, #mma>
30|   }
31| }
32| 
```
**EN:** This function-oriented block defines or enters `a_impl`. Within it, the test exercises constants, tt.func, masked or vectorized loads, lane/block index ranges, shape expansion, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `a_impl` 为核心。测试在其中演示 常量、tt.func、带掩码或向量化的加载、lane/block 索引范围、形状扩展，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 33-33
```mlir
33| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 34-40
```mlir
34| 
35| #blocked = #ttg.blocked<{sizePerThread = [16, 1], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
36| #blocked1 = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [0, 1]}>
37| #mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 64, 16]}>
38| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
39| #shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
40| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 41-41
```mlir
41| module attributes {"ttg.target" = "cuda:90", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 42-45
```mlir
42| // CHECK-LABEL: mma_reorder_transpose
43| // CHECK: ttg.local_alloc
44| // CHECK: ttg.memdesc_trans
45| // CHECK: ttng.warp_group_dot
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: mma_reorder_transpose anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: mma_reorder_transpose 这样的标签用于锚定匹配范围。

### Lines 46-53
```mlir
46|   tt.func @mma_reorder_transpose(%t: tensor<64x128xf16, #blocked1>, %dotb: !ttg.memdesc<64x64xf16, #shared, #smem>, %dotc: tensor<128x64xf32, #mma>) -> tensor<128x64xf32, #mma>{
47|     %a = tt.trans %t {order = array<i32: 1, 0>} : tensor<64x128xf16, #blocked1> -> tensor<128x64xf16, #blocked>
48|     %dota = ttg.local_alloc %a: (tensor<128x64xf16, #blocked>) -> !ttg.memdesc<128x64xf16, #shared1, #smem>
49|     %r = ttng.warp_group_dot %dota, %dotb, %dotc : !ttg.memdesc<128x64xf16, #shared1, #smem> * !ttg.memdesc<64x64xf16, #shared, #smem> -> tensor<128x64xf32, #mma>
50|     tt.return %r : tensor<128x64xf32, #mma>
51|   }
52| }
53| 
```
**EN:** This function-oriented block defines or enters `mma_reorder_transpose`. Within it, the test exercises tt.func, transpose-like layout changes, shared/local memory allocation, ttng.warp_group_dot, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `mma_reorder_transpose` 为核心。测试在其中演示 tt.func、转置类布局变换、共享/本地内存分配、ttng.warp_group_dot、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 54-54
```mlir
54| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 55-62
```mlir
55| 
56| #blocked = #ttg.blocked<{sizePerThread = [16, 1], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
57| #blocked1 = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [0, 1]}>
58| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
59| #shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = true, elementBitWidth = 16}>
60| #smem = #ttg.shared_memory
61| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
62| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 63-63
```mlir
63| // CHECK: #[[$SHARED:.*]] = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 64-64
```mlir
64| module attributes {"ttg.target" = "cuda:90", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 65-65
```mlir
65|   // CHECK-LABEL: mma_reorder_transpose_mmav5
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: mma_reorder_transpose_mmav5 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: mma_reorder_transpose_mmav5 这样的标签用于锚定匹配范围。

### Lines 66-68
```mlir
66|   tt.func @mma_reorder_transpose_mmav5(%t: tensor<64x256xf8E4M3FN, #blocked1>, %dotb: !ttg.memdesc<64x128xf8E4M3FN, #shared1, #smem>, %dotc: !ttg.memdesc<256x128xf32, #tmem, #ttng.tensor_memory>) {
67|     %true = arith.constant true
68|     %a = tt.trans %t {order = array<i32: 1, 0>} : tensor<64x256xf8E4M3FN, #blocked1> -> tensor<256x64xf8E4M3FN, #blocked>
```
**EN:** This function-oriented block defines or enters `mma_reorder_transpose_mmav5`. Within it, the test exercises tt.func, constants, transpose-like layout changes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `mma_reorder_transpose_mmav5` 为核心。测试在其中演示 tt.func、常量、转置类布局变换，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 69-77
```mlir
69|     // CHECK: %[[A:.+]] = ttg.local_alloc {{.*}} -> !ttg.memdesc<64x256xf8E4M3FN, #[[$SHARED]], #smem>
70|     // CHECK: %[[T:.+]] = ttg.memdesc_trans %[[A]] {order = array<i32: 1, 0>}
71|     // CHECK: ttng.tc_gen5_mma %[[T]]
72|     %dota = ttg.local_alloc %a: (tensor<256x64xf8E4M3FN, #blocked>) -> !ttg.memdesc<256x64xf8E4M3FN, #shared1, #smem>
73|     ttng.tc_gen5_mma %dota, %dotb, %dotc, %true, %true : !ttg.memdesc<256x64xf8E4M3FN, #shared1, #smem>, !ttg.memdesc<64x128xf8E4M3FN, #shared1, #smem>, !ttg.memdesc<256x128xf32, #tmem, #ttng.tensor_memory>
74|     tt.return
75|   }
76| }
77| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, Gen5 tensor-core MMA ops, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、Gen5 张量核 MMA 操作、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 78-78
```mlir
78| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 79-83
```mlir
79| 
80| #blocked = #ttg.blocked<{sizePerThread = [16, 1], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
81| #blocked1 = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [0, 1]}>
82| #mma = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 8]}>
83| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 84-84
```mlir
84| module attributes {"ttg.target" = "cuda:80", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 85-91
```mlir
85| // CHECK-LABEL: mmav2_reorder_transpose
86| // CHECK: ttg.local_alloc
87| // CHECK: ttg.memdesc_trans
88| // CHECK: %[[T0:.+]] = ttg.local_load
89| // CHECK: %[[T1:.*]] = tt.trans
90| // CHECK: tt.dot %[[T0]]
91| // CHECK: arith.extf %[[T1]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: mmav2_reorder_transpose anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: mmav2_reorder_transpose 这样的标签用于锚定匹配范围。

### Lines 92-100
```mlir
 92|   tt.func @mmav2_reorder_transpose(%t: tensor<32x128xf16, #blocked1>, %dotb: tensor<32x64xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>, %dotc: tensor<128x64xf32, #mma>) -> (tensor<128x64xf32, #mma>, tensor<128x32xf32, #blocked>){
 93|     %a = tt.trans %t {order = array<i32: 1, 0>} : tensor<32x128xf16, #blocked1> -> tensor<128x32xf16, #blocked>
 94|     %cv = ttg.convert_layout %a : tensor<128x32xf16, #blocked> -> tensor<128x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
 95|     %r = tt.dot %cv, %dotb, %dotc, inputPrecision = tf32 : tensor<128x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * tensor<32x64xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>> -> tensor<128x64xf32, #mma>
 96|     %trans_use = arith.extf %a : tensor<128x32xf16, #blocked> to tensor<128x32xf32, #blocked>
 97|     tt.return %r, %trans_use : tensor<128x64xf32, #mma>, tensor<128x32xf32, #blocked>
 98|   }
 99| }
100| 
```
**EN:** This function-oriented block defines or enters `mmav2_reorder_transpose`. Within it, the test exercises tt.func, transpose-like layout changes, layout conversions, dot-product or MMA-style math, arith.extf, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `mmav2_reorder_transpose` 为核心。测试在其中演示 tt.func、转置类布局变换、布局转换、点积或 MMA 风格计算、arith.extf，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 101-101
```mlir
101| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 102-106
```mlir
102| 
103| #blocked = #ttg.blocked<{sizePerThread = [16, 1], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
104| #blocked1 = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [0, 1]}>
105| #mma = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 8]}>
106| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 107-107
```mlir
107| module attributes {"ttg.target" = "cuda:90", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 108-112
```mlir
108| // CHECK-LABEL: mmav2_transpose_indirect
109| // CHECK: tt.trans
110| // CHECK: ttg.convert_layout
111| // CHECK: arith.addf
112| // CHECK: tt.dot
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: mmav2_transpose_indirect anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: mmav2_transpose_indirect 这样的标签用于锚定匹配范围。

### Lines 113-122
```mlir
113|   tt.func @mmav2_transpose_indirect(%t: tensor<32x128xf16, #blocked1>, %dotb: tensor<32x64xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>, %dotc: tensor<128x64xf32, #mma>) -> tensor<128x64xf32, #mma>{
114|     %cst = arith.constant dense<0.000000e+00> : tensor<128x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
115|     %a = tt.trans %t {order = array<i32: 1, 0>} : tensor<32x128xf16, #blocked1> -> tensor<128x32xf16, #blocked>
116|     %cv = ttg.convert_layout %a : tensor<128x32xf16, #blocked> -> tensor<128x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
117|     %add = arith.addf %cv, %cst : tensor<128x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
118|     %r = tt.dot %add, %dotb, %dotc, inputPrecision = tf32 : tensor<128x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * tensor<32x64xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>> -> tensor<128x64xf32, #mma>
119|     tt.return %r : tensor<128x64xf32, #mma>
120|   }
121| }
122| 
```
**EN:** This function-oriented block defines or enters `mmav2_transpose_indirect`. Within it, the test exercises tt.func, constants, transpose-like layout changes, layout conversions, floating-point additions, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `mmav2_transpose_indirect` 为核心。测试在其中演示 tt.func、常量、转置类布局变换、布局转换、浮点加法，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 123-123
```mlir
123| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 124-134
```mlir
124| 
125| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
126| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
127| #smem = #ttg.shared_memory
128| #blocked4 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [1, 32], warpsPerCTA = [2, 2], order = [1, 0]}>
129| #blocked8 = #ttg.blocked<{sizePerThread = [1, 1, 1, 2, 4], threadsPerWarp = [1, 1, 16, 2, 1], warpsPerCTA = [2, 1, 2, 1, 1], order = [4, 3, 2, 1, 0]}>
130| #blocked9 = #ttg.blocked<{sizePerThread = [1, 2, 1, 1, 4], threadsPerWarp = [1, 2, 16, 1, 1], warpsPerCTA = [2, 1, 2, 1, 1], order = [4, 1, 2, 3, 0]}>
131| #blocked10 = #ttg.blocked<{sizePerThread = [1, 1, 1, 1, 4], threadsPerWarp = [1, 1, 32, 1, 1], warpsPerCTA = [1, 1, 1, 1, 4], order = [4, 3, 2, 1, 0]}>
132| #blocked11 = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [32, 1], warpsPerCTA = [1, 4], order = [1, 0]}>
133| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
134| #tmem_scales = #ttng.tensor_memory_scales_encoding<>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 135-135
```mlir
135| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 136-147
```mlir
136|   // CHECK-LABEL: @scales_in_shmem
137|   // CHECK: %[[A_LA:.*]] = ttg.local_alloc
138|   // CHECK: %[[B_LA:.*]] = ttg.local_alloc
139|   // CHECK: %[[A_RS:.*]] = ttg.memdesc_reshape %[[A_LA]]
140|   // CHECK: %[[A_TR:.*]] = ttg.memdesc_trans %[[A_RS]]
141|   // CHECK: %[[A_FINAL:.*]] = ttg.memdesc_reshape %[[A_TR]]
142|   // CHECK: %[[B_RS:.*]] = ttg.memdesc_reshape %[[B_LA]]
143|   // CHECK: %[[B_TR:.*]] = ttg.memdesc_trans %[[B_RS]]
144|   // CHECK: %[[B_FINAL:.*]] = ttg.memdesc_reshape %[[B_TR]]
145|   // CHECK-NOT: ttg.local_load
146|   // CHECK: ttng.tc_gen5_mma_scaled {{.*}}, %[[A_FINAL]], %[[B_FINAL]],
147| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @scales_in_shmem anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @scales_in_shmem 这样的标签用于锚定匹配范围。

### Lines 148-165
```mlir
148|   tt.func public @scales_in_shmem(
149|     %scale: tensor<2x512x!tt.ptr<i8>, #blocked4> {tt.contiguity = 16 : i32, tt.divisibility = 16 : i32},
150|     %A_sh: !ttg.memdesc<128x128xf8E5M2, #shared, #ttg.shared_memory>,
151|     %B_sh: !ttg.memdesc<128x128xf8E5M2, #shared, #ttg.shared_memory>,
152|     %acc_tm: !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory>
153|     ) {
154|       %true = arith.constant true
155|       %A_la = ttg.local_alloc : () -> !ttg.memdesc<2x512xi8, #shared1, #smem, mutable>
156|       %B_la = ttg.local_alloc : () -> !ttg.memdesc<2x512xi8, #shared1, #smem, mutable>
157|       %A_ll = ttg.local_load %A_la : !ttg.memdesc<2x512xi8, #shared1, #smem, mutable> -> tensor<2x512xi8, #blocked4>
158|       %B_ll = ttg.local_load %B_la : !ttg.memdesc<2x512xi8, #shared1, #smem, mutable> -> tensor<2x512xi8, #blocked4>
159|       %A_r = tt.reshape %A_ll : tensor<2x512xi8, #blocked4> -> tensor<2x1x32x4x4xi8, #blocked8>
160|       %B_r = tt.reshape %B_ll : tensor<2x512xi8, #blocked4> -> tensor<2x1x32x4x4xi8, #blocked8>
161|       %A_tr = tt.trans %A_r {order = array<i32: 0, 3, 2, 1, 4>} : tensor<2x1x32x4x4xi8, #blocked8> -> tensor<2x4x32x1x4xi8, #blocked9>
162|       %B_tr = tt.trans %B_r {order = array<i32: 0, 3, 2, 1, 4>} : tensor<2x1x32x4x4xi8, #blocked8> -> tensor<2x4x32x1x4xi8, #blocked9>
163|       %A_cv = ttg.convert_layout %A_tr : tensor<2x4x32x1x4xi8, #blocked9> -> tensor<2x4x32x1x4xi8, #blocked10>
164|       %B_cv = ttg.convert_layout %B_tr : tensor<2x4x32x1x4xi8, #blocked9> -> tensor<2x4x32x1x4xi8, #blocked10>
165|       %A_r2 = tt.reshape %A_cv : tensor<2x4x32x1x4xi8, #blocked10> -> tensor<256x4xi8, #blocked11>
```
**EN:** This function-oriented block defines or enters `scales_in_shmem`. Within it, the test exercises tensor reshaping, tt.func, shared/local memory allocation, local/shared memory loads, transpose-like layout changes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `scales_in_shmem` 为核心。测试在其中演示 张量重塑、tt.func、共享/本地内存分配、本地/共享内存加载、转置类布局变换，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 166-173
```mlir
166|       %B_r2 = tt.reshape %B_cv : tensor<2x4x32x1x4xi8, #blocked10> -> tensor<256x4xi8, #blocked11>
167|       %A_tm = ttng.tmem_alloc %A_r2 : (tensor<256x4xi8, #blocked11>) -> !ttg.memdesc<256x4xi8, #tmem_scales, #ttng.tensor_memory>
168|       %B_tm = ttng.tmem_alloc %B_r2 : (tensor<256x4xi8, #blocked11>) -> !ttg.memdesc<256x4xi8, #tmem_scales, #ttng.tensor_memory>
169|       ttng.tc_gen5_mma_scaled %A_sh, %B_sh, %acc_tm, %A_tm, %B_tm, %true, %true lhs = e5m2 rhs = e5m2 {loop.cluster = 0 : i32, loop.stage = 2 : i32} : !ttg.memdesc<128x128xf8E5M2, #shared, #ttg.shared_memory>, !ttg.memdesc<128x128xf8E5M2, #shared, #ttg.shared_memory>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory>, !ttg.memdesc<256x4xi8, #tmem_scales, #ttng.tensor_memory>, !ttg.memdesc<256x4xi8, #tmem_scales, #ttng.tensor_memory>
170|       tt.return
171| }
172| }
173| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tensor-memory allocation, tensor reshaping, scaled tensor-core MMA ops, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 张量内存分配、张量重塑、带缩放的张量核 MMA 操作、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 174-174
```mlir
174| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 175-181
```mlir
175| 
176| #blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [2, 2], order = [1, 0], CGALayout = [[1, 0]]}>
177| #blocked2 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [2, 2], order = [0, 1], CGALayout = [[0, 1]]}>
178| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16, CGALayout = [[1, 0]]}>
179| #shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16, CGALayout = [[0, 1]]}>
180| #smem = #ttg.shared_memory
181| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1, CGALayout = [[1, 0]], twoCTAs = true>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 182-182
```mlir
182| module attributes {"ttg.target" = "cuda:100", "ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32, "ttng.two-ctas" = true} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 183-188
```mlir
183|   // CHECK-DAG: #[[BLOCKED:.*]] = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [2, 2], order = [1, 0], CGALayout = {{\[\[1, 0\]\]}}}>
184|   // CHECK-DAG: #[[SHARED:.*]] = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16, CGALayout = {{\[\[0, 1\]\]}}}>
185|   // CHECK-DAG: #[[SHARED_TRANS:.*]] = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = true, elementBitWidth = 16, CGALayout = {{\[\[1, 0\]\]}}}>
186|   // CHECK: %[[ALLOC:.*]] = ttg.local_alloc %arg0 : (tensor<128x64xf8E4M3FN, #[[BLOCKED]]>) -> !ttg.memdesc<128x64xf8E4M3FN, #[[SHARED_TRANS]], #smem>
187|   // CHECK: %[[TRANS:.*]] = ttg.memdesc_trans %[[ALLOC]] {order = array<i32: 1, 0>} : !ttg.memdesc<128x64xf8E4M3FN, #[[SHARED_TRANS]], #smem> -> !ttg.memdesc<64x128xf8E4M3FN, #[[SHARED]], #smem>
188|   // CHECK: ttng.tc_gen5_mma %arg1, %[[TRANS]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 189-197
```mlir
189|   tt.func @mmav5_reorder_transpose_2cta(%b_trans: tensor<128x64xf8E4M3FN, #blocked1>, %dota: !ttg.memdesc<256x64xf8E4M3FN, #shared, #smem>, %dotc: !ttg.memdesc<256x128xf32, #tmem, #ttng.tensor_memory>) {
190|     %true = arith.constant true
191|     %trans = tt.trans %b_trans {order = array<i32: 1, 0>} : tensor<128x64xf8E4M3FN, #blocked1> -> tensor<64x128xf8E4M3FN, #blocked2>
192|     %dotb = ttg.local_alloc %trans : (tensor<64x128xf8E4M3FN, #blocked2>) -> !ttg.memdesc<64x128xf8E4M3FN, #shared1, #smem>
193|     ttng.tc_gen5_mma %dota, %dotb, %dotc, %true, %true {two_ctas}: !ttg.memdesc<256x64xf8E4M3FN, #shared, #smem>, !ttg.memdesc<64x128xf8E4M3FN, #shared1, #smem>, !ttg.memdesc<256x128xf32, #tmem, #ttng.tensor_memory>
194|     tt.return
195|   }
196| }
197| 
```
**EN:** This function-oriented block defines or enters `mmav5_reorder_transpose_2cta`. Within it, the test exercises tt.func, constants, transpose-like layout changes, shared/local memory allocation, Gen5 tensor-core MMA ops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `mmav5_reorder_transpose_2cta` 为核心。测试在其中演示 tt.func、常量、转置类布局变换、共享/本地内存分配、Gen5 张量核 MMA 操作，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 198-198
```mlir
198| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 199-212
```mlir
199| 
200| #blocked1 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 2], order = [0, 1]}>
201| #blocked5 = #ttg.blocked<{sizePerThread = [1, 1, 1, 1, 1], threadsPerWarp = [1, 1, 1, 2, 16], warpsPerCTA = [1, 1, 1, 8, 1], order = [4, 3, 2, 1, 0]}>
202| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 8}>
203| #shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = true, elementBitWidth = 8, fp4Padded = true}>
204| #shared2 = #ttg.nvmma_shared<{swizzlingByteWidth = 0, transposed = false, elementBitWidth = 8, rank = 5}>
205| #smem = #ttg.shared_memory
206| #linear = #ttg.linear<{register = [[0, 1], [0, 2], [32, 0]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [16, 0]], warp = [[0, 0], [0, 0], [64, 0]], block = []}>
207| #linear1 = #ttg.linear<{register = [[0, 0, 0, 0, 1], [0, 0, 0, 0, 2], [0, 0, 0, 0, 4], [0, 0, 0, 0, 8]], lane = [[0, 0, 0, 1, 0], [0, 0, 0, 2, 0], [0, 0, 0, 4, 0], [0, 0, 0, 8, 0], [0, 0, 0, 16, 0]], warp = [[0, 0, 0, 0, 0], [0, 0, 0, 0, 0], [0, 1, 0, 0, 0]], block = []}>
208| #linear2 = #ttg.linear<{register = [[0, 0, 0, 0, 1], [0, 0, 0, 0, 2], [0, 0, 0, 1, 0], [0, 0, 0, 2, 0]], lane = [[0, 0, 1, 0, 0], [0, 0, 2, 0, 0], [0, 0, 4, 0, 0], [0, 0, 8, 0, 0], [0, 0, 16, 0, 0]], warp = [[0, 0, 0, 0, 0], [0, 0, 0, 0, 0], [1, 0, 0, 0, 0]], block = []}>
209| #linear3 = #ttg.linear<{register = [[0, 0, 0, 0, 1], [0, 0, 0, 0, 2], [0, 1, 0, 0, 0], [0, 2, 0, 0, 0]], lane = [[0, 0, 1, 0, 0], [0, 0, 2, 0, 0], [0, 0, 4, 0, 0], [0, 0, 8, 0, 0], [0, 0, 16, 0, 0]], warp = [[0, 0, 0, 0, 0], [0, 0, 0, 0, 0], [1, 0, 0, 0, 0]], block = []}>
210| #linear4 = #ttg.linear<{register = [[0, 1], [0, 2], [32, 0], [64, 0]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [16, 0]], warp = [[0, 0], [0, 0], [128, 0]], block = []}>
211| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 256, colStride = 1>
212| #tmem_scales = #ttng.tensor_memory_scales_encoding<>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 213-213
```mlir
213| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 214-216
```mlir
214|   // CHECK-DAG: #[[BLOCKED5:.*]] = #ttg.blocked<{sizePerThread = [1, 1, 1, 1, 1], threadsPerWarp = [1, 1, 1, 2, 16], warpsPerCTA = [1, 1, 1, 8, 1], order = [4, 3, 2, 1, 0]}>
215|   // CHECK-DAG: #[[SHARED2:.*]] = #ttg.nvmma_shared<{swizzlingByteWidth = 0, transposed = false, elementBitWidth = 8, rank = 5}>
216|   // CHECK-DAG: #[[SMEM:.*]] = #ttg.shared_memory
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 217-234
```mlir
217|   tt.func public @descriptor_load_scales_in_shmem(
218|       %scale_desc_ptr: !tt.ptr<i8>,
219|       %shmemA: !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem>,
220|       %shmemB: !ttg.memdesc<64x256xi8, #shared1, #smem>,
221|       %acc: tensor<128x256xf32, #blocked1>
222|     ) {
223|     %c0_i32 = arith.constant 0 : i32
224|     %c1_i32 = arith.constant 1 : i32
225|     %c2_i32 = arith.constant 2 : i32
226|     %c16_i32 = arith.constant 16 : i32
227|     %c32_i32 = arith.constant 32 : i32
228|     %c1_i64 = arith.constant 1 : i64
229|     %c16_i64 = arith.constant 16 : i64
230|     %c512_i64 = arith.constant 512 : i64
231|     %c1024_i64 = arith.constant 1024 : i64
232|     %cst_scales = arith.constant dense<127> : tensor<128x4xi8, #linear>
233|     %true = arith.constant true
234| 
```
**EN:** This function-oriented block defines or enters `descriptor_load_scales_in_shmem`. Within it, the test exercises constants, tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `descriptor_load_scales_in_shmem` 为核心。测试在其中演示 常量、tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 235-235
```mlir
235|     %desc = tt.make_tensor_descriptor %scale_desc_ptr, [%c1_i32, %c2_i32, %c1_i32, %c32_i32, %c16_i32], [%c1024_i64, %c512_i64, %c512_i64, %c16_i64, %c1_i64] : !tt.ptr<i8>, <1x2x1x32x16xi8>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.make_tensor_descriptor. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.make_tensor_descriptor。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 236-237
```mlir
236|     // CHECK: %[[DESC_LOAD:.*]] = tt.descriptor_load {{.*}} !tt.tensordesc<1x2x1x32x16xi8> -> tensor<1x2x1x32x16xi8, #[[BLOCKED5]]>
237|     %83 = tt.descriptor_load %desc[%c0_i32, %c0_i32, %c0_i32, %c0_i32, %c0_i32] : !tt.tensordesc<1x2x1x32x16xi8> -> tensor<1x2x1x32x16xi8, #blocked5>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.descriptor_load. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.descriptor_load。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 238-239
```mlir
238|     // CHECK: %[[DESC_LA:.*]] = ttg.local_alloc %[[DESC_LOAD]] : (tensor<1x2x1x32x16xi8, #[[BLOCKED5]]>) -> !ttg.memdesc<1x2x1x32x16xi8, #[[SHARED2]], #[[SMEM]]>
239|     %84 = ttg.local_alloc %83 : (tensor<1x2x1x32x16xi8, #blocked5>) -> !ttg.memdesc<1x2x1x32x16xi8, #shared2, #smem>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 240-241
```mlir
240|     // CHECK-NOT: ttg.local_load
241|     %85 = ttg.local_load %84 : !ttg.memdesc<1x2x1x32x16xi8, #shared2, #smem> -> tensor<1x2x1x32x16xi8, #linear1>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on local/shared memory loads. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 本地/共享内存加载。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 242-243
```mlir
242|     // CHECK-NOT: tt.reshape
243|     %86 = tt.reshape %85 : tensor<1x2x1x32x16xi8, #linear1> -> tensor<2x1x32x4x4xi8, #linear2>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tensor reshaping. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 张量重塑。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 244-245
```mlir
244|     // CHECK-NOT: tt.trans
245|     %87 = tt.trans %86 {order = array<i32: 0, 3, 2, 1, 4>} : tensor<2x1x32x4x4xi8, #linear2> -> tensor<2x4x32x1x4xi8, #linear3>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on transpose-like layout changes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 转置类布局变换。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 246-250
```mlir
246|     // CHECK-NOT: tt.reshape
247|     %88 = tt.reshape %87 : tensor<2x4x32x1x4xi8, #linear3> -> tensor<256x4xi8, #linear4>
248|     %89 = ttng.tmem_alloc %acc : (tensor<128x256xf32, #blocked1>) -> !ttg.memdesc<128x256xf32, #tmem, #ttng.tensor_memory, mutable>
249|     %90 = ttng.tmem_alloc %cst_scales : (tensor<128x4xi8, #linear>) -> !ttg.memdesc<128x4xi8, #tmem_scales, #ttng.tensor_memory>
250|     %91 = ttng.tmem_alloc %88 : (tensor<256x4xi8, #linear4>) -> !ttg.memdesc<256x4xi8, #tmem_scales, #ttng.tensor_memory>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tensor-memory allocation, tensor reshaping. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 张量内存分配、张量重塑。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 251-259
```mlir
251|     // CHECK: %[[DESC_RS:.*]] = ttg.memdesc_reshape %[[DESC_LA]] : !ttg.memdesc<1x2x1x32x16xi8, #[[SHARED2]], #[[SMEM]]> -> !ttg.memdesc<2x1x32x4x4xi8, {{.*}}, #smem>
252|     // CHECK: %[[DESC_TR:.*]] = ttg.memdesc_trans %[[DESC_RS]]
253|     // CHECK: %[[SCALE_ALLOC:.*]] = ttg.memdesc_reshape %[[DESC_TR]] : !ttg.memdesc<2x4x32x1x4xi8, {{.*}}, #smem> -> !ttg.memdesc<256x4xi8, {{.*}}, #smem>
254|     // CHECK: ttng.tc_gen5_mma_scaled {{.*}}, {{.*}}, {{.*}}, {{.*}}, %[[SCALE_ALLOC]], {{.*}}
255|     ttng.tc_gen5_mma_scaled %shmemA, %shmemB, %89, %90, %91, %true, %true lhs = e4m3 rhs = e2m1 : !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem>, !ttg.memdesc<64x256xi8, #shared1, #smem>, !ttg.memdesc<128x256xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.memdesc<128x4xi8, #tmem_scales, #ttng.tensor_memory>, !ttg.memdesc<256x4xi8, #tmem_scales, #ttng.tensor_memory>
256|     tt.return
257|   }
258| }
259| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on scaled tensor-core MMA ops, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 带缩放的张量核 MMA 操作、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 260-260
```mlir
260| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 261-265
```mlir
261| 
262| #blocked = #ttg.blocked<{sizePerThread = [1, 1, 1, 1], threadsPerWarp = [1, 1, 1, 32], warpsPerCTA = [1, 1, 4, 1], order = [3, 2, 1, 0]}>
263| #blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
264| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = false, elementBitWidth = 16}>
265| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 266-267
```mlir
266| // CHECK-DAG: #[[$SHARED:.+]] = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = false, elementBitWidth = 16}>
267| // CHECK-DAG: #[[$SHARED1:.+]] = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = false, elementBitWidth = 16, rank = 4}>
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 268-268
```mlir
268| module attributes {"ttg.target" = "cuda:100", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 269-269
```mlir
269|   // CHECK-LABEL: @reshape_memedesc
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @reshape_memedesc anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @reshape_memedesc 这样的标签用于锚定匹配范围。

### Lines 270-270
```mlir
270|   tt.func @reshape_memedesc(%arg: tensor<32x1x4x64xf16, #blocked>) -> !ttg.memdesc<128x64xf16, #shared, #smem> {
```
**EN:** This function-oriented block defines or enters `reshape_memedesc`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `reshape_memedesc` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 271-272
```mlir
271|     // CHECK: [[A:.+]] = ttg.local_alloc %{{.+}} : (tensor<32x1x4x64xf16, #{{.*}}>) -> !ttg.memdesc<32x1x4x64xf16, #[[$SHARED1]], #smem>
272|     %r = tt.reshape %arg : tensor<32x1x4x64xf16, #blocked> -> tensor<128x64xf16, #blocked1>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tensor reshaping. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 张量重塑。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 273-274
```mlir
273|     // CHECK: %[[R:.+]] = ttg.memdesc_reshape %[[A:.+]] : !ttg.memdesc<32x1x4x64xf16, #[[$SHARED1]], #smem> -> !ttg.memdesc<128x64xf16, #[[$SHARED]], #smem>
274|     %a = ttg.local_alloc %r : (tensor<128x64xf16, #blocked1>) -> !ttg.memdesc<128x64xf16, #shared, #smem>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 275-279
```mlir
275|     // CHECK: tt.return %[[R]]
276|     tt.return %a: !ttg.memdesc<128x64xf16, #shared, #smem>
277|   }
278| }
279| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 280-280
```mlir
280| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 281-285
```mlir
281| 
282| #blocked = #ttg.blocked<{sizePerThread = [1, 1, 1, 1], threadsPerWarp = [1, 1, 1, 32], warpsPerCTA = [1, 2, 2, 1], order = [3, 2, 1, 0]}>
283| #blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
284| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = true, elementBitWidth = 32}>
285| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 286-286
```mlir
286| module attributes {"ttg.target" = "cuda:100", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 287-287
```mlir
287|   // CHECK-LABEL: @reshape_memedesc_negative
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @reshape_memedesc_negative anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @reshape_memedesc_negative 这样的标签用于锚定匹配范围。

### Lines 288-289
```mlir
288|   tt.func @reshape_memedesc_negative(%arg: tensor<1x32x2x64xf32, #blocked>) -> !ttg.memdesc<64x64xf32, #shared, #smem> {
289|     %r = tt.reshape %arg : tensor<1x32x2x64xf32, #blocked> -> tensor<64x64xf32, #blocked1>
```
**EN:** This function-oriented block defines or enters `reshape_memedesc_negative`. Within it, the test exercises tt.func, tensor reshaping, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `reshape_memedesc_negative` 为核心。测试在其中演示 tt.func、张量重塑，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 290-291
```mlir
290|     // CHECK-NOT: ttg.memdesc_reshape
291|     %a = ttg.local_alloc %r : (tensor<64x64xf32, #blocked1>) -> !ttg.memdesc<64x64xf32, #shared, #smem>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 292-296
```mlir
292|     // CHECK: tt.return
293|     tt.return %a: !ttg.memdesc<64x64xf32, #shared, #smem>
294|   }
295| }
296| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 297-297
```mlir
297| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 298-308
```mlir
298| 
299| #blocked0 = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
300| #blocked1 = #ttg.blocked<{sizePerThread = [1, 1, 1, 1, 16], threadsPerWarp = [1, 1, 4, 8, 1], warpsPerCTA = [1, 1, 4, 1, 1], order = [4, 3, 2, 1, 0]}>
301| #blocked2 = #ttg.blocked<{sizePerThread = [1, 1, 1, 16], threadsPerWarp = [1, 4, 8, 1], warpsPerCTA = [1, 4, 1, 1], order = [3, 2, 1, 0]}>
302| #blocked3 = #ttg.blocked<{sizePerThread = [1, 1, 1, 16], threadsPerWarp = [4, 8, 1, 1], warpsPerCTA = [4, 1, 1, 1], order = [3, 1, 0, 2]}>
303| #linear0 = #ttg.linear<{register = [[0, 1], [0, 2], [0, 4], [0, 8], [128, 0], [0, 16], [0, 32], [0, 64]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [16, 0]], warp = [[32, 0], [64, 0]], block = []}>
304| #linear2 = #ttg.linear<{register = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 128], [16, 0], [32, 0], [64, 0]], lane = [[0, 1], [0, 2], [0, 4], [0, 8], [0, 16]], warp = [[0, 32], [0, 64]], block = []}>
305| #shared0 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 8}>
306| #shared5 = #ttg.shared_linear<{offset = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 1], [0, 2], [0, 4], [0, 8], [0, 16], [0, 32], [0, 64], [0, 128], [16, 0], [32, 0], [64, 0]]}, alignment = 128>
307| #smem = #ttg.shared_memory
308| #tmem0 = #ttng.tensor_memory_encoding<blockM = 128, blockN = 256, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 309-310
```mlir
309| // CHECK-DAG: #{{.*}} = #ttg.shared_linear<{{.*}}alignment = 128>
310| // CHECK-DAG: #{{.*}} = #ttg.shared_linear<{{.*}}alignment = 128>
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 311-311
```mlir
311| module attributes {"ttg.target" = "cuda:100", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 312-324
```mlir
312|   // CHECK-LABEL: @swizzle0_operand_views
313|   // CHECK-DAG: %[[A_DESC:.*]] = tt.descriptor_load {{.*}} : !tt.tensordesc<128x128xf8E4M3FN> -> tensor<128x128xf8E4M3FN, #{{.*}}>
314|   // CHECK-DAG: %[[A_BASE:.*]] = ttg.local_alloc %[[A_DESC]] : (tensor<128x128xf8E4M3FN, #{{.*}}>) -> !ttg.memdesc<128x128xf8E4M3FN, #{{.*}}, #smem{{.*}}>
315|   // CHECK-DAG: %[[B_DESC:.*]] = tt.descriptor_load {{.*}} : !tt.tensordesc<1x1x256x8x16xf8E4M3FN> -> tensor<1x1x256x8x16xf8E4M3FN, #{{.*}}>
316|   // CHECK-DAG: %[[B_BASE:.*]] = ttg.local_alloc %[[B_DESC]] : (tensor<1x1x256x8x16xf8E4M3FN, #{{.*}}>) -> !ttg.memdesc<1x1x256x8x16xf8E4M3FN, #{{.*}}, #smem{{.*}}>
317|   // CHECK-DAG: %[[B_RS0:.*]] = ttg.memdesc_reshape %[[B_BASE]] : !ttg.memdesc<1x1x256x8x16xf8E4M3FN, #{{.*}}, #smem{{.*}}> -> !ttg.memdesc<8x32x8x16xf8E4M3FN, #{{.*}}, #smem{{.*}}>
318|   // CHECK-DAG: %[[B_TR0:.*]] = ttg.memdesc_trans %[[B_RS0]] {order = array<i32: 1, 2, 0, 3>} : !ttg.memdesc<8x32x8x16xf8E4M3FN, #{{.*}}, #smem{{.*}}> -> !ttg.memdesc<32x8x8x16xf8E4M3FN, #{{.*}}, #smem{{.*}}>
319|   // CHECK-DAG: %[[B_RS1:.*]] = ttg.memdesc_reshape %[[B_TR0]] : !ttg.memdesc<32x8x8x16xf8E4M3FN, #{{.*}}, #smem{{.*}}> -> !ttg.memdesc<256x128xf8E4M3FN, #{{.*}}, #smem{{.*}}>
320|   // CHECK-DAG: %[[B_TR1:.*]] = ttg.memdesc_trans %[[B_RS1]] {order = array<i32: 1, 0>} : !ttg.memdesc<256x128xf8E4M3FN, #{{.*}}, #smem{{.*}}> -> !ttg.memdesc<128x256xf8E4M3FN, #{{.*}}, #smem{{.*}}>
321|   // CHECK-NOT: tt.reshape
322|   // CHECK-NOT: tt.trans
323|   // CHECK-NOT: ttg.local_load
324|   // CHECK: ttng.tc_gen5_mma %[[A_BASE]], %[[B_TR1]], %arg2, %true, %true
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @swizzle0_operand_views anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @swizzle0_operand_views 这样的标签用于锚定匹配范围。

### Lines 325-342
```mlir
325|   tt.func @swizzle0_operand_views(
326|       %a_desc: !tt.tensordesc<128x128xf8E4M3FN>,
327|       %b_desc: !tt.tensordesc<1x1x256x8x16xf8E4M3FN>,
328|       %acc: !ttg.memdesc<128x256xf32, #tmem0, #ttng.tensor_memory>) {
329|     %true = arith.constant true
330|     %c0_i32 = arith.constant 0 : i32
331|     %a = tt.descriptor_load %a_desc[%c0_i32, %c0_i32] : !tt.tensordesc<128x128xf8E4M3FN> -> tensor<128x128xf8E4M3FN, #blocked0>
332|     %b = tt.descriptor_load %b_desc[%c0_i32, %c0_i32, %c0_i32, %c0_i32, %c0_i32] : !tt.tensordesc<1x1x256x8x16xf8E4M3FN> -> tensor<1x1x256x8x16xf8E4M3FN, #blocked1>
333|     %b1 = tt.reshape %b : tensor<1x1x256x8x16xf8E4M3FN, #blocked1> -> tensor<8x32x8x16xf8E4M3FN, #blocked2>
334|     %b2 = tt.trans %b1 {order = array<i32: 1, 2, 0, 3>} : tensor<8x32x8x16xf8E4M3FN, #blocked2> -> tensor<32x8x8x16xf8E4M3FN, #blocked3>
335|     %b3 = tt.reshape %b2 : tensor<32x8x8x16xf8E4M3FN, #blocked3> -> tensor<256x128xf8E4M3FN, #linear0>
336|     %b4 = tt.trans %b3 {order = array<i32: 1, 0>} : tensor<256x128xf8E4M3FN, #linear0> -> tensor<128x256xf8E4M3FN, #linear2>
337|     %a_s = ttg.local_alloc %a : (tensor<128x128xf8E4M3FN, #blocked0>) -> !ttg.memdesc<128x128xf8E4M3FN, #shared0, #smem>
338|     %b_s = ttg.local_alloc %b4 : (tensor<128x256xf8E4M3FN, #linear2>) -> !ttg.memdesc<128x256xf8E4M3FN, #shared5, #smem>
339|     ttng.tc_gen5_mma %a_s, %b_s, %acc, %true, %true : !ttg.memdesc<128x128xf8E4M3FN, #shared0, #smem>, !ttg.memdesc<128x256xf8E4M3FN, #shared5, #smem>, !ttg.memdesc<128x256xf32, #tmem0, #ttng.tensor_memory>
340|     tt.return
341|   }
342| }
```
**EN:** This function-oriented block defines or enters `swizzle0_operand_views`. Within it, the test exercises tt.func, constants, tt.descriptor_load, tensor reshaping, transpose-like layout changes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `swizzle0_operand_views` 为核心。测试在其中演示 tt.func、常量、tt.descriptor_load、张量重塑、转置类布局变换，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 344-345
```mlir
344| 
345| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 345-351
```mlir
345| 
346| #blocked_a_load = #ttg.blocked<{sizePerThread = [1, 1, 1, 2], threadsPerWarp = [1, 4, 1, 8], warpsPerCTA = [1, 4, 1, 1], order = [3, 2, 1, 0]}>
347| #blocked_a_mat = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
348| #shared_a_nvmma = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = true, elementBitWidth = 32}>
349| #shared_rhs_linear = #ttg.shared_linear<{offset = [[0, 1], [0, 2], [0, 4], [0, 8], [1, 0], [2, 4], [4, 8], [8, 0], [16, 0], [32, 0]]}, alignment = 512>
350| #tmem_dbg = #ttng.tensor_memory_encoding<blockM = 64, blockN = 16, colStride = 1>
351| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 352-352
```mlir
352| module attributes {"ttg.target" = "cuda:100", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 353-358
```mlir
353|   // CHECK-LABEL: @nvmma_operand_views_not_rewritten
354|   // CHECK: %[[DESC:.*]] = tt.descriptor_load %{{.*}}[%{{.*}}] : !tt.tensordesc<1x16x1x16xf32> -> tensor<1x16x1x16xf32, #{{.*}}>
355|   // CHECK: %[[RESHAPE:.*]] = tt.reshape %[[DESC]] : tensor<1x16x1x16xf32, #{{.*}}> -> tensor<16x16xf32, #{{.*}}>
356|   // CHECK: %[[ALLOC:.*]] = ttg.local_alloc %[[RESHAPE]] : (tensor<16x16xf32, #{{.*}}>) -> !ttg.memdesc<16x16xf32, #{{.*}}, #smem>
357|   // CHECK-NOT: ttg.memdesc_reshape
358|   // CHECK: ttng.tc_gen5_mma %{{.*}}, %[[ALLOC]], %{{.*}}, %true, %true : !ttg.memdesc<64x16xf32, #{{.*}}, #smem>, !ttg.memdesc<16x16xf32, #{{.*}}, #smem>, !ttg.memdesc<64x16xf32, #{{.*}}, #ttng.tensor_memory>
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @nvmma_operand_views_not_rewritten anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @nvmma_operand_views_not_rewritten 这样的标签用于锚定匹配范围。

### Lines 359-372
```mlir
359|   tt.func @nvmma_operand_views_not_rewritten(
360|       %lhs: !tt.tensordesc<1x16x1x16xf32>,
361|       %rhs: !ttg.memdesc<64x16xf32, #shared_rhs_linear, #smem>,
362|       %acc: !ttg.memdesc<64x16xf32, #tmem_dbg, #ttng.tensor_memory>) {
363|     %true = arith.constant true
364|     %c0 = arith.constant 0 : i32
365|     %a = tt.descriptor_load %lhs[%c0, %c0, %c0, %c0] : !tt.tensordesc<1x16x1x16xf32> -> tensor<1x16x1x16xf32, #blocked_a_load>
366|     %a_rs = tt.reshape %a : tensor<1x16x1x16xf32, #blocked_a_load> -> tensor<16x16xf32, #blocked_a_mat>
367|     %a_s = ttg.local_alloc %a_rs : (tensor<16x16xf32, #blocked_a_mat>) -> !ttg.memdesc<16x16xf32, #shared_a_nvmma, #smem>
368|     ttng.tc_gen5_mma %rhs, %a_s, %acc, %true, %true : !ttg.memdesc<64x16xf32, #shared_rhs_linear, #smem>, !ttg.memdesc<16x16xf32, #shared_a_nvmma, #smem>, !ttg.memdesc<64x16xf32, #tmem_dbg, #ttng.tensor_memory>
369|     tt.return
370|   }
371| }
372| 
```
**EN:** This function-oriented block defines or enters `nvmma_operand_views_not_rewritten`. Within it, the test exercises tt.func, constants, tt.descriptor_load, tensor reshaping, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `nvmma_operand_views_not_rewritten` 为核心。测试在其中演示 tt.func、常量、tt.descriptor_load、张量重塑、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 373-373
```mlir
373| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 374-384
```mlir
374| 
375| #blockedA_desc = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
376| #blockedB_desc = #ttg.blocked<{sizePerThread = [1, 1, 1, 1, 16], threadsPerWarp = [1, 1, 4, 8, 1], warpsPerCTA = [1, 1, 4, 1, 1], order = [4, 3, 2, 1, 0]}>
377| #blockedB0_desc = #ttg.blocked<{sizePerThread = [1, 1, 1, 16], threadsPerWarp = [1, 4, 8, 1], warpsPerCTA = [1, 4, 1, 1], order = [3, 2, 1, 0]}>
378| #blockedB1_desc = #ttg.blocked<{sizePerThread = [1, 1, 1, 16], threadsPerWarp = [4, 8, 1, 1], warpsPerCTA = [4, 1, 1, 1], order = [3, 1, 0, 2]}>
379| #linear_desc = #ttg.linear<{register = [[0, 1], [0, 2], [0, 4], [0, 8], [128, 0], [0, 16], [0, 32], [0, 64]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [16, 0]], warp = [[32, 0], [64, 0]], block = []}>
380| #mma_desc = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 256, 32]}>
381| #sharedA_desc = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 8}>
382| #sharedB3_desc = #ttg.shared_linear<{offset = [[0, 1], [0, 2], [0, 4], [0, 8], [1, 0], [2, 0], [4, 0], [8, 0], [16, 0], [32, 0], [64, 0], [128, 0], [0, 16], [0, 32], [0, 64]]}, alignment = 128>
383| #sharedB4_desc = #ttg.shared_linear<{offset = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 1], [0, 2], [0, 4], [0, 8], [0, 16], [0, 32], [0, 64], [0, 128], [16, 0], [32, 0], [64, 0]]}, alignment = 128>
384| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 385-386
```mlir
385| // CHECK-DAG: #{{.*}} = #ttg.shared_linear<{{.*}}alignment = 128>
386| // CHECK-DAG: #{{.*}} = #ttg.shared_linear<{{.*}}alignment = 128>
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 387-387
```mlir
387| module attributes {"ttg.target" = "cuda:90", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 388-398
```mlir
388|   // CHECK-LABEL: @swizzle0_operand_views_warp_group_dot
389|   // CHECK-DAG: %[[B_DESC_LOAD:.*]] = tt.descriptor_load %arg1[%{{.*}}] : !tt.tensordesc<1x1x256x8x16xf8E4M3FN> -> tensor<1x1x256x8x16xf8E4M3FN, #{{.*}}>
390|   // CHECK-DAG: %[[B_BASE:.*]] = ttg.local_alloc %[[B_DESC_LOAD]] : (tensor<1x1x256x8x16xf8E4M3FN, {{.*}}>) -> !ttg.memdesc<1x1x256x8x16xf8E4M3FN, #{{.*}}, #smem{{.*}}>
391|   // CHECK-DAG: %[[B_RS0:.*]] = ttg.memdesc_reshape %[[B_BASE]] : !ttg.memdesc<1x1x256x8x16xf8E4M3FN, #{{.*}}, #smem{{.*}}> -> !ttg.memdesc<8x32x8x16xf8E4M3FN, {{.*}}, #smem{{.*}}>
392|   // CHECK-DAG: %[[B_TR:.*]] = ttg.memdesc_trans %[[B_RS0]] {order = array<i32: 1, 2, 0, 3>} : !ttg.memdesc<8x32x8x16xf8E4M3FN, {{.*}}, #smem{{.*}}> -> !ttg.memdesc<32x8x8x16xf8E4M3FN, {{.*}}, #smem{{.*}}>
393|   // CHECK-DAG: %[[B_RS1:.*]] = ttg.memdesc_reshape %[[B_TR]] : !ttg.memdesc<32x8x8x16xf8E4M3FN, {{.*}}, #smem{{.*}}> -> !ttg.memdesc<256x128xf8E4M3FN, {{.*}}, #smem{{.*}}>
394|   // CHECK-DAG: %[[B_T:.*]] = ttg.memdesc_trans %[[B_RS1]] {order = array<i32: 1, 0>} : !ttg.memdesc<256x128xf8E4M3FN, {{.*}}, #smem{{.*}}> -> !ttg.memdesc<128x256xf8E4M3FN, #{{.*}}, #smem{{.*}}>
395|   // CHECK-NOT: tt.reshape
396|   // CHECK-NOT: tt.trans
397|   // CHECK: %[[DOT:.*]] = ttng.warp_group_dot %{{.*}}, %[[B_T]], %arg2
398|   // CHECK: %[[WAIT:.*]]:3 = ttng.warp_group_dot_wait %[[DOT]], {{.*}}, %[[B_T]] {pendings = 0 : i32}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @swizzle0_operand_views_warp_group_dot anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @swizzle0_operand_views_warp_group_dot 这样的标签用于锚定匹配范围。

### Lines 399-416
```mlir
399|   tt.func @swizzle0_operand_views_warp_group_dot(
400|       %a_desc: !tt.tensordesc<128x128xf8E4M3FN, #sharedA_desc>,
401|       %b_desc: !tt.tensordesc<1x1x256x8x16xf8E4M3FN>,
402|       %acc: tensor<128x256xf32, #mma_desc>) -> tensor<128x256xf32, #mma_desc> {
403|     %c0 = arith.constant 0 : i32
404|     %a = tt.descriptor_load %a_desc[%c0, %c0] : !tt.tensordesc<128x128xf8E4M3FN, #sharedA_desc> -> tensor<128x128xf8E4M3FN, #blockedA_desc>
405|     %a_s = ttg.local_alloc %a : (tensor<128x128xf8E4M3FN, #blockedA_desc>) -> !ttg.memdesc<128x128xf8E4M3FN, #sharedA_desc, #smem>
406|     %b_cm = tt.descriptor_load %b_desc[%c0, %c0, %c0, %c0, %c0] : !tt.tensordesc<1x1x256x8x16xf8E4M3FN> -> tensor<1x1x256x8x16xf8E4M3FN, #blockedB_desc>
407|     %b0 = tt.reshape %b_cm : tensor<1x1x256x8x16xf8E4M3FN, #blockedB_desc> -> tensor<8x32x8x16xf8E4M3FN, #blockedB0_desc>
408|     %b1 = tt.trans %b0 {order = array<i32: 1, 2, 0, 3>} : tensor<8x32x8x16xf8E4M3FN, #blockedB0_desc> -> tensor<32x8x8x16xf8E4M3FN, #blockedB1_desc>
409|     %b2 = tt.reshape %b1 : tensor<32x8x8x16xf8E4M3FN, #blockedB1_desc> -> tensor<256x128xf8E4M3FN, #linear_desc>
410|     %b_s = ttg.local_alloc %b2 : (tensor<256x128xf8E4M3FN, #linear_desc>) -> !ttg.memdesc<256x128xf8E4M3FN, #sharedB3_desc, #smem>
411|     %b_t = ttg.memdesc_trans %b_s {order = array<i32: 1, 0>} : !ttg.memdesc<256x128xf8E4M3FN, #sharedB3_desc, #smem> -> !ttg.memdesc<128x256xf8E4M3FN, #sharedB4_desc, #smem>
412|     %r = ttng.warp_group_dot %a_s, %b_t, %acc {inputPrecision = 0 : i32, isAsync = true, maxNumImpreciseAcc = 1073741824 : i32}
413|       : !ttg.memdesc<128x128xf8E4M3FN, #sharedA_desc, #smem> * !ttg.memdesc<128x256xf8E4M3FN, #sharedB4_desc, #smem> -> tensor<128x256xf32, #mma_desc>
414|     %w:3 = ttng.warp_group_dot_wait %r, %a_s, %b_t {pendings = 0 : i32} : tensor<128x256xf32, #mma_desc>, !ttg.memdesc<128x128xf8E4M3FN, #sharedA_desc, #smem>, !ttg.memdesc<128x256xf8E4M3FN, #sharedB4_desc, #smem>
415|     tt.return %w#0 : tensor<128x256xf32, #mma_desc>
416|   }
```
**EN:** This function-oriented block defines or enters `swizzle0_operand_views_warp_group_dot`. Within it, the test exercises tt.func, tt.descriptor_load, shared/local memory allocation, tensor reshaping, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `swizzle0_operand_views_warp_group_dot` 为核心。测试在其中演示 tt.func、tt.descriptor_load、共享/本地内存分配、张量重塑、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 417-417
```mlir
417| }
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `-tritongpu-optimize-dot-operands`, `-canonicalize`
- **CN:** 主要 pass 选项：`-split-input-file`，`-tritongpu-optimize-dot-operands`，`-canonicalize`
- **EN:** Dominant operations include `tt.func`, `arith.constant`, `module`, `tt.return`, `ttg.local_alloc`, `tt.reshape`, `tt.trans`, `tt.descriptor_load`, `ttg.convert_layout`, `ttng.tmem_alloc`.
- **CN:** 主要操作包括 `tt.func`、`arith.constant`、`module`、`tt.return`、`ttg.local_alloc`、`tt.reshape`、`tt.trans`、`tt.descriptor_load`、`ttg.convert_layout`、`ttng.tmem_alloc`。
- **EN:** The file contains 13 independently testable section(s). Check styles used: CHECK x47, CHECK-DAG x26, CHECK-NOT x13, CHECK-LABEL x10. Important labels include mma_reorder_transpose, mma_reorder_transpose_mmav5, mmav2_reorder_transpose, mmav2_transpose_indirect. It validates both the presence of the desired rewrite and the absence of unwanted residual operations.
- **CN:** 该文件包含 13 个可独立测试的分段。使用的检查类型：CHECK ×47，CHECK-DAG ×26，CHECK-NOT ×13，CHECK-LABEL ×10。 关键标签包括 mma_reorder_transpose，mma_reorder_transpose_mmav5，mmav2_reorder_transpose，mmav2_transpose_indirect。 它同时验证目标重写已经出现，并且不再残留不需要的操作。

## Dependencies / 依赖关系

- **EN:** This test depends on Triton/MLIR textual IR parsing, the pass pipeline named in the RUN lines, and the dialect/type definitions referenced in the file.
- **CN:** 该测试依赖 Triton/MLIR 文本 IR 解析、RUN 行中指定的 pass 流水线，以及文件中引用的方言/类型定义。
- **EN:** `ttg`: TritonGPU layout, memory, and GPU-specific operations.
- **CN:** `ttg`：TritonGPU 布局、内存和 GPU 专用操作。
- **EN:** `tt`: Triton high-level tensor/pointer operations.
- **CN:** `tt`：Triton 高层张量/指针操作。
- **EN:** `ttng`: Triton Nvidia GPU tensor-memory and MMA operations.
- **CN:** `ttng`：Triton Nvidia GPU 张量内存与 MMA 操作。
- **EN:** `arith`: MLIR arithmetic dialect for scalar/tensor math.
- **CN:** `arith`：用于标量/张量运算的 MLIR arithmetic 方言。
- **EN:** `FileCheck` is used to match required textual patterns after rewriting/lowering.
- **CN:** `FileCheck` 用于在重写/lower 之后匹配必须出现的文本模式。