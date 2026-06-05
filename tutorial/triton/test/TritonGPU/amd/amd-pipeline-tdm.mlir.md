# amd-pipeline-tdm.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/amd/amd-pipeline-tdm.mlir`
- **EN:** Test for the MLIR/Triton pipeline `-tritonamdgpu-optimize-descriptor-encoding, -tritonamdgpu-schedule-loops="num_stages=2", -tritonamdgpu-pipeline="use_async_copy=1", -canonicalize` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `-tritonamdgpu-optimize-descriptor-encoding, -tritonamdgpu-schedule-loops="num_stages=2", -tritonamdgpu-pipeline="use_async_copy=1", -canonicalize` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -split-input-file -tritonamdgpu-optimize-descriptor-encoding  -tritonamdgpu-schedule-loops="num_stages=2" -tritonamdgpu-pipeline="use_async_copy=1" -canonicalize | FileCheck %s`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -split-input-file -tritonamdgpu-optimize-descriptor-encoding  -tritonamdgpu-schedule-loops="num_stages=2" -tritonamdgpu-pipeline="use_async_copy=1" -canonicalize | FileCheck %s`

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
1| // RUN: triton-opt %s -split-input-file -tritonamdgpu-optimize-descriptor-encoding  -tritonamdgpu-schedule-loops="num_stages=2" -tritonamdgpu-pipeline="use_async_copy=1" -canonicalize | FileCheck %s
2| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -split-input-file -tritonamdgpu-optimize-descriptor-encoding  -tritonamdgpu-schedule-loops="num_stages=2" -tritonamdgpu-pipeline="use_async_copy=1" -canonicalize | FileCheck %s` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -split-input-file -tritonamdgpu-optimize-descriptor-encoding  -tritonamdgpu-schedule-loops="num_stages=2" -tritonamdgpu-pipeline="use_async_copy=1" -canonicalize | FileCheck %s`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 3-5
```mlir
3| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 4], warpsPerCTA = [8, 1], order = [1, 0]}>
4| #blocked1 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 8], warpsPerCTA = [8, 1], order = [1, 0]}>
5| #mma = #ttg.amd_wmma<{version = 3, isTranspose = true, ctaLayout = {warp = [[1, 0], [2, 0], [4, 0]]}, instrShape = [16, 16, 32]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 6-6
```mlir
6| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 7-24
```mlir
 7|   tt.func @matmul_kernel_make_tensor_descriptor(%a_ptr: !tt.ptr<f16> {tt.divisibility = 16 : i32},
 8|     %b_ptr: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %c_ptr: !tt.ptr<f16> {tt.divisibility = 16 : i32},
 9|     %M: i32 {tt.divisibility = 16 : i32}, %N: i32 {tt.divisibility = 16 : i32}, %K: i32 {tt.divisibility = 16 : i32}) {
10|     %c512_i32 = arith.constant 512 : i32
11|     %c64_i32 = arith.constant 64 : i32
12|     %c0_i32 = arith.constant 0 : i32
13|     %c1_i64 = arith.constant 1 : i64
14|     %c32_i32 = arith.constant 32 : i32
15|     %c1_i32 = arith.constant 1 : i32
16|     %c31_i32 = arith.constant 31 : i32
17|     %cst = arith.constant dense<0.000000e+00> : tensor<512x64xf32, #mma>
18|     %0 = tt.get_program_id x : i32
19|     %1 = tt.get_program_id y : i32
20|     %2 = arith.muli %0, %c512_i32 : i32
21|     %3 = arith.muli %1, %c64_i32 : i32
22|     %4 = arith.extsi %K : i32 to i64
23|     %5 = tt.make_tensor_descriptor %a_ptr, [%M, %K], [%4, %c1_i64] : <f16>, <512x32xf16>
24|     %6 = arith.extsi %N : i32 to i64
```
**EN:** This function-oriented block defines or enters `matmul_kernel_make_tensor_descriptor`. Within it, the test exercises constants, tt.func, program IDs, integer multiplications, arith.extsi, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `matmul_kernel_make_tensor_descriptor` 为核心。测试在其中演示 常量、tt.func、程序 ID、整数乘法、arith.extsi，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 25-42
```mlir
25|     %7 = tt.make_tensor_descriptor %b_ptr, [%K, %N], [%6, %c1_i64] : <f16>, <32x64xf16>
26|     %8 = tt.make_tensor_descriptor %c_ptr, [%M, %N], [%6, %c1_i64] : <f16>, <512x64xf16>
27|     %9 = arith.addi %K, %c31_i32 : i32
28|     %10 = arith.divsi %9, %c32_i32 : i32
29|     %accumulator:2 = scf.for %accumulator_0 = %c0_i32 to %10 step %c1_i32 iter_args(%arg7 = %c0_i32, %arg8 = %cst) -> (i32, tensor<512x64xf32, #mma>)  : i32 {
30|       %13 = tt.descriptor_load %5[%2, %arg7] : !tt.tensordesc<512x32xf16> -> tensor<512x32xf16, #blocked>
31|       %14 = tt.descriptor_load %7[%arg7, %3] : !tt.tensordesc<32x64xf16> -> tensor<32x64xf16, #blocked1>
32|       %15 = ttg.convert_layout %13 : tensor<512x32xf16, #blocked> -> tensor<512x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>>
33|       %16 = ttg.convert_layout %14 : tensor<32x64xf16, #blocked1> -> tensor<32x64xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>>
34|       %17 = tt.dot %15, %16, %arg8 : tensor<512x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>> * tensor<32x64xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>> -> tensor<512x64xf32, #mma>
35|       %18 = arith.addi %arg7, %c32_i32 : i32
36|       scf.yield %18, %17 : i32, tensor<512x64xf32, #mma>
37|     }
38|     %11 = arith.truncf %accumulator#1 : tensor<512x64xf32, #mma> to tensor<512x64xf16, #mma>
39|     %12 = ttg.convert_layout %11 : tensor<512x64xf16, #mma> -> tensor<512x64xf16, #blocked1>
40|     tt.descriptor_store %8[%2, %3], %12 : !tt.tensordesc<512x64xf16>, tensor<512x64xf16, #blocked1>
41|     tt.return
42|   }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining layout conversions, tt.make_tensor_descriptor, integer additions, tt.descriptor_load, arith.divsi.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 布局转换、tt.make_tensor_descriptor、整数加法、tt.descriptor_load、arith.divsi。

### Lines 43-55
```mlir
43| }
44| 
45| // Operand A (opIdx=0, order=[1,0]): loadTransposed = (1 != 1) = false → non-transposed
46| //   padAmount = min(kWidth=8, 128/16) = min(8, 8) = 8
47| //   innerDimLength = shape[order[0]] = shape[1] = 32 (K dim)
48| //   → padded_shared<[32:+8]>
49| //
50| // Operand B (opIdx=1, order=[1,0]): loadTransposed = (1 != 0) = true → transposed
51| //   queryLDSTransLoadParams(16) → instBitWidth=128, padAmount = 2*128/16 = 16
52| //   innerDimLength = shape[order[0]] = shape[1] = 64 (N dim)
53| //   → padded_shared<[64:+16]>
54| //
55| // tt.desciptor_store -> padded_shared<[64:+8]>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 56-63
```mlir
56| // CHECK: #[[$PADDED_A:.*]] = #ttg.padded_shared<[128:+8] {order = [1, 0], shape = [512, 32]}>
57| // CHECK: #[[$PADDED_B:.*]] = #ttg.padded_shared<[128:+16] {order = [1, 0], shape = [32, 64]}>
58| // CHECK: #[[$PADDED_C:.*]] = #ttg.padded_shared<[64:+8] {order = [1, 0], shape = [512, 64]}>
59| // CHECK-NOT: #ttg.padded_shared
60| 
61| // The loop body and epilogue each emit two adjacent amdg.async_tdm_wait ops
62| // (one per descriptor_load) which combineRedundantWaitOps folds into a single
63| // wait taking both tokens; the matched two-operand wait below proves the fold.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 64-80
```mlir
64| // CHECK-LABEL: tt.func @matmul_kernel_make_tensor_descriptor
65| // CHECK: async_tdm_copy_global_to_local {{.*}} : !tt.tensordesc<512x32xf16, #[[$PADDED_A]]> -> !ttg.memdesc<512x32xf16, #[[$PADDED_A]], #smem, mutable>
66| // CHECK-NOT: ttg.async_commit_group
67| // CHECK: async_tdm_copy_global_to_local {{.*}} : !tt.tensordesc<32x64xf16, #[[$PADDED_B]]> -> !ttg.memdesc<32x64xf16, #[[$PADDED_B]], #smem, mutable>
68| // CHECK-NOT: ttg.async_commit_group
69| // CHECK: scf.for
70| // CHECK: amdg.async_tdm_wait %{{[^,]+}}, %{{[^,]+}} {num = 0 : i32}
71| // CHECK-NOT: amdg.async_tdm_wait
72| // CHECK: async_tdm_copy_global_to_local {{.*}} : !tt.tensordesc<512x32xf16, #[[$PADDED_A]]> -> !ttg.memdesc<512x32xf16, #[[$PADDED_A]], #smem, mutable>
73| // CHECK-NOT: ttg.async_commit_group
74| // CHECK: async_tdm_copy_global_to_local {{.*}} : !tt.tensordesc<32x64xf16, #[[$PADDED_B]]> -> !ttg.memdesc<32x64xf16, #[[$PADDED_B]], #smem, mutable>
75| // CHECK-NOT: ttg.async_commit_group
76| // CHECK: }
77| // CHECK: amdg.async_tdm_wait %{{[^,]+}}, %{{[^,]+}} {num = 0 : i32}
78| // CHECK-NOT: amdg.async_tdm_wait
79| // CHECK: tt.descriptor_store {{.*}} : !tt.tensordesc<512x64xf16, #[[$PADDED_C]]>
80| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: tt.func @matmul_kernel_make_tensor_descriptor anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: tt.func @matmul_kernel_make_tensor_descriptor 这样的标签用于锚定匹配范围。

### Lines 81-81
```mlir
81| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 82-96
```mlir
82| 
83| // Test TDM padding for fp8 (f8E5M2) matmul on gfx1250.
84| //
85| // Operand A (opIdx=0, order=[1,0]): loadTransposed = (1 != 1) = false → non-transposed
86| //   padAmount = 128/8 = 16 (sub-dword: no min with vecWidth, full 4-dword
87| //   stride separation needed for ds_load_2addr_b64 cross-address conflicts)
88| //   innerDimLength = shape[1] = 64 (K dim)
89| //   → padded_shared<[64:+16]>
90| //
91| // Operand B (opIdx=1, order=[1,0]): loadTransposed = (1 != 0) = true → transposed
92| //   queryLDSTransLoadParams(8) → instBitWidth=64, padAmount = 2*64/8 = 16
93| //   innerDimLength = shape[1] = 64 (N dim)
94| //   → padded_shared<[64:+16]>
95| //
96| // tt.descriptor_store -> padded_shared<[64:+8]>
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 97-101
```mlir
 97| // CHECK: #[[$PADDED_A:.*]] = #ttg.padded_shared<[256:+16] {order = [1, 0], shape = [256, 64]}>
 98| // CHECK: #[[$PADDED_B:.*]] = #ttg.padded_shared<[256:+16] {order = [1, 0], shape = [64, 64]}>
 99| // CHECK: #[[$PADDED_C:.*]] = #ttg.padded_shared<[64:+8] {order = [1, 0], shape = [256, 64]}>
100| // CHECK-NOT: #ttg.padded_shared
101| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 102-104
```mlir
102| #blocked = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [8, 4], warpsPerCTA = [8, 1], order = [1, 0]}>
103| #blocked1 = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [4, 8], warpsPerCTA = [8, 1], order = [1, 0]}>
104| #mma = #ttg.amd_wmma<{version = 3, isTranspose = true, ctaLayout = {warp = [[1, 0], [2, 0], [4, 0]]}, instrShape = [16, 16, 64]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 105-105
```mlir
105| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 106-123
```mlir
106|   tt.func @tdm_padding_fp8(%a_ptr: !tt.ptr<f8E5M2> {tt.divisibility = 16 : i32},
107|     %b_ptr: !tt.ptr<f8E5M2> {tt.divisibility = 16 : i32}, %c_ptr: !tt.ptr<f16> {tt.divisibility = 16 : i32},
108|     %M: i32 {tt.divisibility = 16 : i32}, %N: i32 {tt.divisibility = 16 : i32}, %K: i32 {tt.divisibility = 16 : i32}) {
109|     %c256_i32 = arith.constant 256 : i32
110|     %c64_i32 = arith.constant 64 : i32
111|     %c0_i32 = arith.constant 0 : i32
112|     %c1_i64 = arith.constant 1 : i64
113|     %c1_i32 = arith.constant 1 : i32
114|     %c63_i32 = arith.constant 63 : i32
115|     %cst = arith.constant dense<0.000000e+00> : tensor<256x64xf32, #mma>
116|     %0 = tt.get_program_id x : i32
117|     %1 = tt.get_program_id y : i32
118|     %2 = arith.muli %0, %c256_i32 : i32
119|     %3 = arith.muli %1, %c64_i32 : i32
120|     %4 = arith.extsi %K : i32 to i64
121|     %5 = tt.make_tensor_descriptor %a_ptr, [%M, %K], [%4, %c1_i64] : <f8E5M2>, <256x64xf8E5M2>
122|     %6 = arith.extsi %N : i32 to i64
123|     %7 = tt.make_tensor_descriptor %b_ptr, [%K, %N], [%6, %c1_i64] : <f8E5M2>, <64x64xf8E5M2>
```
**EN:** This function-oriented block defines or enters `tdm_padding_fp8`. Within it, the test exercises constants, tt.func, program IDs, integer multiplications, arith.extsi, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tdm_padding_fp8` 为核心。测试在其中演示 常量、tt.func、程序 ID、整数乘法、arith.extsi，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 124-141
```mlir
124|     %8 = tt.make_tensor_descriptor %c_ptr, [%M, %N], [%6, %c1_i64] : <f16>, <256x64xf16>
125|     %9 = arith.addi %K, %c63_i32 : i32
126|     %10 = arith.divsi %9, %c64_i32 : i32
127|     %accumulator:2 = scf.for %iv = %c0_i32 to %10 step %c1_i32 iter_args(%k_off = %c0_i32, %acc = %cst) -> (i32, tensor<256x64xf32, #mma>)  : i32 {
128|       %a = tt.descriptor_load %5[%2, %k_off] : !tt.tensordesc<256x64xf8E5M2> -> tensor<256x64xf8E5M2, #blocked>
129|       %b = tt.descriptor_load %7[%k_off, %3] : !tt.tensordesc<64x64xf8E5M2> -> tensor<64x64xf8E5M2, #blocked1>
130|       %a_dot = ttg.convert_layout %a : tensor<256x64xf8E5M2, #blocked> -> tensor<256x64xf8E5M2, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>>
131|       %b_dot = ttg.convert_layout %b : tensor<64x64xf8E5M2, #blocked1> -> tensor<64x64xf8E5M2, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>>
132|       %d = tt.dot %a_dot, %b_dot, %acc : tensor<256x64xf8E5M2, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>> * tensor<64x64xf8E5M2, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>> -> tensor<256x64xf32, #mma>
133|       %next_k = arith.addi %k_off, %c64_i32 : i32
134|       scf.yield %next_k, %d : i32, tensor<256x64xf32, #mma>
135|     }
136|     %out = arith.truncf %accumulator#1 : tensor<256x64xf32, #mma> to tensor<256x64xf16, #mma>
137|     %out_blocked = ttg.convert_layout %out : tensor<256x64xf16, #mma> -> tensor<256x64xf16, #blocked1>
138|     tt.descriptor_store %8[%2, %3], %out_blocked : !tt.tensordesc<256x64xf16>, tensor<256x64xf16, #blocked1>
139|     tt.return
140|   }
141| }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining layout conversions, integer additions, tt.descriptor_load, tt.make_tensor_descriptor, arith.divsi.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 布局转换、整数加法、tt.descriptor_load、tt.make_tensor_descriptor、arith.divsi。

### Lines 142-155
```mlir
142| 
143| // CHECK-LABEL: tt.func @tdm_padding_fp8
144| // CHECK: async_tdm_copy_global_to_local {{.*}} : !tt.tensordesc<256x64xf8E5M2, #[[$PADDED_A]]> -> !ttg.memdesc<256x64xf8E5M2, #[[$PADDED_A]], #smem, mutable>
145| // CHECK-NOT: ttg.async_commit_group
146| // CHECK: async_tdm_copy_global_to_local {{.*}} : !tt.tensordesc<64x64xf8E5M2, #[[$PADDED_B]]> -> !ttg.memdesc<64x64xf8E5M2, #[[$PADDED_B]], #smem, mutable>
147| // CHECK-NOT: ttg.async_commit_group
148| // CHECK: scf.for
149| // CHECK: async_tdm_copy_global_to_local {{.*}} : !tt.tensordesc<256x64xf8E5M2, #[[$PADDED_A]]> -> !ttg.memdesc<256x64xf8E5M2, #[[$PADDED_A]], #smem, mutable>
150| // CHECK-NOT: ttg.async_commit_group
151| // CHECK: async_tdm_copy_global_to_local {{.*}} : !tt.tensordesc<64x64xf8E5M2, #[[$PADDED_B]]> -> !ttg.memdesc<64x64xf8E5M2, #[[$PADDED_B]], #smem, mutable>
152| // CHECK-NOT: ttg.async_commit_group
153| // CHECK: }
154| // CHECK: tt.descriptor_store {{.*}} : !tt.tensordesc<256x64xf16, #[[$PADDED_C]]>
155| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: tt.func @tdm_padding_fp8 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: tt.func @tdm_padding_fp8 这样的标签用于锚定匹配范围。

### Lines 156-156
```mlir
156| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 157-170
```mlir
157| 
158| // Test TDM padding for f32 matmul on gfx1250.
159| //
160| // Operand A (opIdx=0, order=[1,0]): loadTransposed = (1 != 1) = false → non-transposed
161| //   padAmount = min(kWidth=8, 128/32) = min(8, 4) = 4
162| //   innerDimLength = shape[1] = 16 (K dim)
163| //   → padded_shared<[16:+4]>
164| //
165| // Operand B (opIdx=1, order=[1,0]): loadTransposed = (1 != 0) = true → transposed
166| //   queryLDSTransLoadParams(32) → empty, falls back to padAmount = 128/32 = 4
167| //   innerDimLength = shape[1] = 64 (N dim)
168| //   → padded_shared<[64:+4]>
169| //
170| // tt.desciptor_store -> padded_shared<[64:+4]>
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 171-175
```mlir
171| // CHECK: #[[$PADDED_A:.*]] = #ttg.padded_shared<[64:+4] {order = [1, 0], shape = [256, 16]}>
172| // CHECK: #[[$PADDED_B:.*]] = #ttg.padded_shared<[64:+4] {order = [1, 0], shape = [16, 64]}>
173| // CHECK: #[[$PADDED_C:.*]] = #ttg.padded_shared<[64:+4] {order = [1, 0], shape = [256, 64]}>
174| // CHECK-NOT: #ttg.padded_shared
175| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 176-178
```mlir
176| #blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [8, 4], warpsPerCTA = [8, 1], order = [1, 0]}>
177| #blocked1 = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [4, 8], warpsPerCTA = [8, 1], order = [1, 0]}>
178| #mma = #ttg.amd_wmma<{version = 3, isTranspose = true, ctaLayout = {warp = [[1, 0], [2, 0], [4, 0]]}, instrShape = [16, 16, 4]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 179-179
```mlir
179| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 180-197
```mlir
180|   tt.func @tdm_padding_f32(%a_ptr: !tt.ptr<f32> {tt.divisibility = 16 : i32},
181|     %b_ptr: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %c_ptr: !tt.ptr<f32> {tt.divisibility = 16 : i32},
182|     %M: i32 {tt.divisibility = 16 : i32}, %N: i32 {tt.divisibility = 16 : i32}, %K: i32 {tt.divisibility = 16 : i32}) {
183|     %c256_i32 = arith.constant 256 : i32
184|     %c64_i32 = arith.constant 64 : i32
185|     %c0_i32 = arith.constant 0 : i32
186|     %c1_i64 = arith.constant 1 : i64
187|     %c16_i32 = arith.constant 16 : i32
188|     %c1_i32 = arith.constant 1 : i32
189|     %c15_i32 = arith.constant 15 : i32
190|     %cst = arith.constant dense<0.000000e+00> : tensor<256x64xf32, #mma>
191|     %0 = tt.get_program_id x : i32
192|     %1 = tt.get_program_id y : i32
193|     %2 = arith.muli %0, %c256_i32 : i32
194|     %3 = arith.muli %1, %c64_i32 : i32
195|     %4 = arith.extsi %K : i32 to i64
196|     %5 = tt.make_tensor_descriptor %a_ptr, [%M, %K], [%4, %c1_i64] : <f32>, <256x16xf32>
197|     %6 = arith.extsi %N : i32 to i64
```
**EN:** This function-oriented block defines or enters `tdm_padding_f32`. Within it, the test exercises constants, tt.func, program IDs, integer multiplications, arith.extsi, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tdm_padding_f32` 为核心。测试在其中演示 常量、tt.func、程序 ID、整数乘法、arith.extsi，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 198-215
```mlir
198|     %7 = tt.make_tensor_descriptor %b_ptr, [%K, %N], [%6, %c1_i64] : <f32>, <16x64xf32>
199|     %8 = tt.make_tensor_descriptor %c_ptr, [%M, %N], [%6, %c1_i64] : <f32>, <256x64xf32>
200|     %9 = arith.addi %K, %c15_i32 : i32
201|     %10 = arith.divsi %9, %c16_i32 : i32
202|     %accumulator:2 = scf.for %iv = %c0_i32 to %10 step %c1_i32 iter_args(%k_off = %c0_i32, %acc = %cst) -> (i32, tensor<256x64xf32, #mma>)  : i32 {
203|       %a = tt.descriptor_load %5[%2, %k_off] : !tt.tensordesc<256x16xf32> -> tensor<256x16xf32, #blocked>
204|       %b = tt.descriptor_load %7[%k_off, %3] : !tt.tensordesc<16x64xf32> -> tensor<16x64xf32, #blocked1>
205|       %a_dot = ttg.convert_layout %a : tensor<256x16xf32, #blocked> -> tensor<256x16xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>>
206|       %b_dot = ttg.convert_layout %b : tensor<16x64xf32, #blocked1> -> tensor<16x64xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>>
207|       %d = tt.dot %a_dot, %b_dot, %acc, inputPrecision = tf32 : tensor<256x16xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>> * tensor<16x64xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>> -> tensor<256x64xf32, #mma>
208|       %next_k = arith.addi %k_off, %c16_i32 : i32
209|       scf.yield %next_k, %d : i32, tensor<256x64xf32, #mma>
210|     }
211|     %out_blocked = ttg.convert_layout %accumulator#1 : tensor<256x64xf32, #mma> -> tensor<256x64xf32, #blocked1>
212|     tt.descriptor_store %8[%2, %3], %out_blocked : !tt.tensordesc<256x64xf32>, tensor<256x64xf32, #blocked1>
213|     tt.return
214|   }
215| }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining layout conversions, tt.make_tensor_descriptor, integer additions, tt.descriptor_load, arith.divsi.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 布局转换、tt.make_tensor_descriptor、整数加法、tt.descriptor_load、arith.divsi。

### Lines 216-229
```mlir
216| 
217| // CHECK-LABEL: tt.func @tdm_padding_f32
218| // CHECK: async_tdm_copy_global_to_local {{.*}} : !tt.tensordesc<256x16xf32, #[[$PADDED_A]]> -> !ttg.memdesc<256x16xf32, #[[$PADDED_A]], #smem, mutable>
219| // CHECK-NOT: ttg.async_commit_group
220| // CHECK: async_tdm_copy_global_to_local {{.*}} : !tt.tensordesc<16x64xf32, #[[$PADDED_B]]> -> !ttg.memdesc<16x64xf32, #[[$PADDED_B]], #smem, mutable>
221| // CHECK-NOT: ttg.async_commit_group
222| // CHECK: scf.for
223| // CHECK: async_tdm_copy_global_to_local {{.*}} : !tt.tensordesc<256x16xf32, #[[$PADDED_A]]> -> !ttg.memdesc<256x16xf32, #[[$PADDED_A]], #smem, mutable>
224| // CHECK-NOT: ttg.async_commit_group
225| // CHECK: async_tdm_copy_global_to_local {{.*}} : !tt.tensordesc<16x64xf32, #[[$PADDED_B]]> -> !ttg.memdesc<16x64xf32, #[[$PADDED_B]], #smem, mutable>
226| // CHECK-NOT: ttg.async_commit_group
227| // CHECK: }
228| // CHECK: tt.descriptor_store {{.*}} : !tt.tensordesc<256x64xf32, #[[$PADDED_C]]>
229| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: tt.func @tdm_padding_f32 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: tt.func @tdm_padding_f32 这样的标签用于锚定匹配范围。

### Lines 230-230
```mlir
230| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 231-235
```mlir
231| 
232| // Test TDM pipeline for gather + dot on gfx1250.
233| // Two gathers (A and B) inside a loop feed into a dot. The pipeline pass
234| // should convert them to async_tdm_gather ops and software-pipeline the loop.
235| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 236-242
```mlir
236| #blocked_ga = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 4], warpsPerCTA = [8, 1], order = [1, 0]}>
237| #blocked_gb = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [4, 8], warpsPerCTA = [8, 1], order = [1, 0]}>
238| #mma_g = #ttg.amd_wmma<{version = 3, isTranspose = true, ctaLayout = {warp = [[1, 0], [2, 0], [4, 0]]}, instrShape = [16, 16, 32]}>
239| #padded_ga = #ttg.padded_shared<[32:+8] {order = [1, 0], shape = [1, 32]}>
240| #padded_gb = #ttg.padded_shared<[16:+16] {order = [1, 0], shape = [1, 16]}>
241| #padded_gc = #ttg.padded_shared<[16:+8] {order = [1, 0], shape = [16, 16]}>
242| #idx_enc = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [32, 1], warpsPerCTA = [1, 8], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 243-243
```mlir
243| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 244-261
```mlir
244|   tt.func @gather_dot_pipeline(
245|       %a_ptr: !tt.ptr<f16> {tt.divisibility = 16 : i32},
246|       %b_ptr: !tt.ptr<f16> {tt.divisibility = 16 : i32},
247|       %c_ptr: !tt.ptr<f16> {tt.divisibility = 16 : i32},
248|       %M: i32 {tt.divisibility = 16 : i32},
249|       %N: i32 {tt.divisibility = 16 : i32},
250|       %K: i32 {tt.divisibility = 16 : i32}) {
251|     %c16_i32 = arith.constant 16 : i32
252|     %c0_i32 = arith.constant 0 : i32
253|     %c1_i64 = arith.constant 1 : i64
254|     %c32_i32 = arith.constant 32 : i32
255|     %c1_i32 = arith.constant 1 : i32
256|     %c31_i32 = arith.constant 31 : i32
257|     %cst = arith.constant dense<0.000000e+00> : tensor<16x16xf32, #mma_g>
258|     %4 = arith.extsi %K : i32 to i64
259|     %a_desc = tt.make_tensor_descriptor %a_ptr, [%M, %K], [%4, %c1_i64] : <f16>, <1x32xf16, #padded_ga>
260|     %6 = arith.extsi %N : i32 to i64
261|     %b_desc = tt.make_tensor_descriptor %b_ptr, [%K, %N], [%6, %c1_i64] : <f16>, <1x16xf16, #padded_gb>
```
**EN:** This function-oriented block defines or enters `gather_dot_pipeline`. Within it, the test exercises constants, tt.func, arith.extsi, tt.make_tensor_descriptor, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `gather_dot_pipeline` 为核心。测试在其中演示 常量、tt.func、arith.extsi、tt.make_tensor_descriptor，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 262-279
```mlir
262|     %c_desc = tt.make_tensor_descriptor %c_ptr, [%M, %N], [%6, %c1_i64] : <f16>, <16x16xf16, #padded_gc>
263|     %a_indices = tt.make_range {start = 0 : i32, end = 16 : i32} : tensor<16xi32, #ttg.slice<{dim = 0, parent = #idx_enc}>>
264|     %b_indices = tt.make_range {start = 0 : i32, end = 32 : i32} : tensor<32xi32, #ttg.slice<{dim = 0, parent = #idx_enc}>>
265|     %9 = arith.addi %K, %c31_i32 : i32
266|     %10 = arith.divsi %9, %c32_i32 : i32
267|     %accumulator:2 = scf.for %iv = %c0_i32 to %10 step %c1_i32 iter_args(%k_off = %c0_i32, %acc = %cst) -> (i32, tensor<16x16xf32, #mma_g>)  : i32 {
268|       %a = tt.descriptor_gather %a_desc[%a_indices, %k_off] : (!tt.tensordesc<1x32xf16, #padded_ga>, tensor<16xi32, #ttg.slice<{dim = 0, parent = #idx_enc}>>, i32) -> tensor<16x32xf16, #blocked_ga>
269|       %b = tt.descriptor_gather %b_desc[%b_indices, %c0_i32] : (!tt.tensordesc<1x16xf16, #padded_gb>, tensor<32xi32, #ttg.slice<{dim = 0, parent = #idx_enc}>>, i32) -> tensor<32x16xf16, #blocked_gb>
270|       %a_dot = ttg.convert_layout %a : tensor<16x32xf16, #blocked_ga> -> tensor<16x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma_g, kWidth = 8}>>
271|       %b_dot = ttg.convert_layout %b : tensor<32x16xf16, #blocked_gb> -> tensor<32x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma_g, kWidth = 8}>>
272|       %d = tt.dot %a_dot, %b_dot, %acc : tensor<16x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma_g, kWidth = 8}>> * tensor<32x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma_g, kWidth = 8}>> -> tensor<16x16xf32, #mma_g>
273|       %next_k = arith.addi %k_off, %c32_i32 : i32
274|       scf.yield %next_k, %d : i32, tensor<16x16xf32, #mma_g>
275|     }
276|     %out = arith.truncf %accumulator#1 : tensor<16x16xf32, #mma_g> to tensor<16x16xf16, #mma_g>
277|     %out_blocked = ttg.convert_layout %out : tensor<16x16xf16, #mma_g> -> tensor<16x16xf16, #blocked_gb>
278|     tt.descriptor_store %c_desc[%c0_i32, %c0_i32], %out_blocked : !tt.tensordesc<16x16xf16, #padded_gc>, tensor<16x16xf16, #blocked_gb>
279|     tt.return
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining layout conversions, lane/block index ranges, integer additions, tt.descriptor_gather, tt.make_tensor_descriptor.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 布局转换、lane/block 索引范围、整数加法、tt.descriptor_gather、tt.make_tensor_descriptor。

### Lines 280-282
```mlir
280|   }
281| }
282| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 283-284
```mlir
283| // CHECK-LABEL: tt.func @gather_dot_pipeline
284| // Prologue: two async_tdm_gather ops before the loop
```
**EN:** This function-oriented block defines or enters `gather_dot_pipeline`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `gather_dot_pipeline` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 285-289
```mlir
285| // CHECK: amdg.async_tdm_gather
286| // CHECK-NOT: ttg.async_commit_group
287| // CHECK: amdg.async_tdm_gather
288| // CHECK-NOT: ttg.async_commit_group
289| // Loop body: two more async_tdm_gather ops (pipelined next iteration)
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 290-296
```mlir
290| // CHECK: scf.for
291| // CHECK: amdg.async_tdm_gather
292| // CHECK-NOT: ttg.async_commit_group
293| // CHECK: amdg.async_tdm_gather
294| // CHECK-NOT: ttg.async_commit_group
295| // CHECK: }
296| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 297-297
```mlir
297| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 298-302
```mlir
298| 
299| // A/B descriptor loads are converted to TDM copies, but a descriptor load used
300| // as the dot accumulator operand remains a raw tt.descriptor_load. Dynamic loop
301| // predication must guard that raw load directly.
302| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 303-305
```mlir
303| #blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [8, 4], warpsPerCTA = [8, 1], order = [1, 0]}>
304| #blocked1 = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [4, 8], warpsPerCTA = [8, 1], order = [1, 0]}>
305| #mma = #ttg.amd_wmma<{version = 3, isTranspose = true, ctaLayout = {warp = [[1, 0], [2, 0], [4, 0]]}, instrShape = [16, 16, 4]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 306-306
```mlir
306| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 307-324
```mlir
307|   tt.func @descriptor_load_accumulator_predicated(
308|       %a_ptr: !tt.ptr<f32> {tt.divisibility = 16 : i32},
309|       %b_ptr: !tt.ptr<f32> {tt.divisibility = 16 : i32},
310|       %acc_ptr: !tt.ptr<f32> {tt.divisibility = 16 : i32},
311|       %c_ptr: !tt.ptr<f32> {tt.divisibility = 16 : i32},
312|       %M: i32 {tt.divisibility = 16 : i32},
313|       %N: i32 {tt.divisibility = 16 : i32},
314|       %K: i32 {tt.divisibility = 16 : i32}) {
315|     %c256_i32 = arith.constant 256 : i32
316|     %c64_i32 = arith.constant 64 : i32
317|     %c0_i32 = arith.constant 0 : i32
318|     %c1_i64 = arith.constant 1 : i64
319|     %c16_i32 = arith.constant 16 : i32
320|     %c1_i32 = arith.constant 1 : i32
321|     %c15_i32 = arith.constant 15 : i32
322|     %zero = arith.constant dense<0.000000e+00> : tensor<256x64xf32, #mma>
323|     %pid_m = tt.get_program_id x : i32
324|     %pid_n = tt.get_program_id y : i32
```
**EN:** This function-oriented block defines or enters `descriptor_load_accumulator_predicated`. Within it, the test exercises constants, tt.func, program IDs, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `descriptor_load_accumulator_predicated` 为核心。测试在其中演示 常量、tt.func、程序 ID，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 325-342
```mlir
325|     %m = arith.muli %pid_m, %c256_i32 : i32
326|     %n = arith.muli %pid_n, %c64_i32 : i32
327|     %k_stride = arith.extsi %K : i32 to i64
328|     %a_desc = tt.make_tensor_descriptor %a_ptr, [%M, %K], [%k_stride, %c1_i64] : <f32>, <256x16xf32>
329|     %n_stride = arith.extsi %N : i32 to i64
330|     %b_desc = tt.make_tensor_descriptor %b_ptr, [%K, %N], [%n_stride, %c1_i64] : <f32>, <16x64xf32>
331|     %acc_desc = tt.make_tensor_descriptor %acc_ptr, [%M, %N], [%n_stride, %c1_i64] : <f32>, <256x64xf32>
332|     %c_desc = tt.make_tensor_descriptor %c_ptr, [%M, %N], [%n_stride, %c1_i64] : <f32>, <256x64xf32>
333|     %k_plus = arith.addi %K, %c15_i32 : i32
334|     %num_k = arith.divsi %k_plus, %c16_i32 : i32
335|     %accumulator:2 = scf.for %iv = %c0_i32 to %num_k step %c1_i32 iter_args(%k_off = %c0_i32, %acc = %zero) -> (i32, tensor<256x64xf32, #mma>)  : i32 {
336|       %a = tt.descriptor_load %a_desc[%m, %k_off] : !tt.tensordesc<256x16xf32> -> tensor<256x16xf32, #blocked>
337|       %b = tt.descriptor_load %b_desc[%k_off, %n] : !tt.tensordesc<16x64xf32> -> tensor<16x64xf32, #blocked1>
338|       %acc_load = tt.descriptor_load %acc_desc[%m, %n] : !tt.tensordesc<256x64xf32> -> tensor<256x64xf32, #mma>
339|       %a_dot = ttg.convert_layout %a : tensor<256x16xf32, #blocked> -> tensor<256x16xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>>
340|       %b_dot = ttg.convert_layout %b : tensor<16x64xf32, #blocked1> -> tensor<16x64xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>>
341|       %d = tt.dot %a_dot, %b_dot, %acc_load, inputPrecision = tf32 : tensor<256x16xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>> * tensor<16x64xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>> -> tensor<256x64xf32, #mma>
342|       %next_k = arith.addi %k_off, %c16_i32 : i32
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining tt.make_tensor_descriptor, tt.descriptor_load, integer multiplications, arith.extsi, integer additions.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 tt.make_tensor_descriptor、tt.descriptor_load、整数乘法、arith.extsi、整数加法。

### Lines 343-350
```mlir
343|       scf.yield %next_k, %d : i32, tensor<256x64xf32, #mma>
344|     }
345|     %out = ttg.convert_layout %accumulator#1 : tensor<256x64xf32, #mma> -> tensor<256x64xf32, #blocked1>
346|     tt.descriptor_store %c_desc[%m, %n], %out : !tt.tensordesc<256x64xf32>, tensor<256x64xf32, #blocked1>
347|     tt.return
348|   }
349| }
350| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, layout conversions, tt.descriptor_store, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、布局转换、tt.descriptor_store、tt.return。

### Lines 351-358
```mlir
351| // CHECK-LABEL: tt.func @descriptor_load_accumulator_predicated
352| // CHECK: scf.if {{.*}} -> (tensor<256x64xf32
353| // CHECK-NEXT: tt.descriptor_load
354| // CHECK-NEXT: tt.dot
355| // CHECK-NEXT: scf.yield
356| // CHECK-NEXT: } else {
357| // CHECK-NEXT: scf.yield
358| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: tt.func @descriptor_load_accumulator_predicated anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: tt.func @descriptor_load_accumulator_predicated 这样的标签用于锚定匹配范围。

### Lines 359-359
```mlir
359| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 360-366
```mlir
360| 
361| #blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [4, 2], order = [1, 0]}>
362| #blocked3 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [2, 16], warpsPerCTA = [8, 1], order = [1, 0]}>
363| #blocked5 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [8, 1], order = [1, 0]}>
364| #blocked6 = #ttg.blocked<{sizePerThread = [1, 1, 1, 1], threadsPerWarp = [1, 1, 1, 32], warpsPerCTA = [1, 2, 4, 1], order = [3, 2, 1, 0]}>
365| #blocked8 = #ttg.blocked<{sizePerThread = [2, 2], threadsPerWarp = [1, 32], warpsPerCTA = [8, 1], order = [1, 0]}>
366| #blocked11 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [2, 4], order = [0, 1]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 367-367
```mlir
367| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 368-385
```mlir
368|   tt.func @descriptor_store_predicate(%arg0: !tt.tensordesc<32x16xbf16>, %arg1: !tt.tensordesc<1x16x4x32xbf16>, %arg2: tensor<16x64xbf16, #ttg.dot_op<{opIdx = 1, parent = #blocked8}>>, %ub: i32) {
369|     %c0_i32 = arith.constant 0 : i32
370|     %c32_i32 = arith.constant 32 : i32
371|     %cst = arith.constant dense<0.000000e+00> : tensor<32x64xf32, #blocked8>
372|     %0 = scf.for %iv = %c0_i32 to %ub step %c32_i32 iter_args(%idx = %c0_i32) -> (i32) : i32 {
373|       %load = tt.descriptor_load %arg0[%idx, %c0_i32] : !tt.tensordesc<32x16xbf16> -> tensor<32x16xbf16, #blocked3>
374|       %lhs = ttg.convert_layout %load : tensor<32x16xbf16, #blocked3> -> tensor<32x16xbf16, #ttg.dot_op<{opIdx = 0, parent = #blocked8}>>
375|       %dot = tt.dot %lhs, %arg2, %cst : tensor<32x16xbf16, #ttg.dot_op<{opIdx = 0, parent = #blocked8}>> * tensor<16x64xbf16, #ttg.dot_op<{opIdx = 1, parent = #blocked8}>> -> tensor<32x64xf32, #blocked8>
376|       %dot_blocked = ttg.convert_layout %dot : tensor<32x64xf32, #blocked8> -> tensor<32x64xf32, #blocked1>
377|       %trans = tt.trans %dot_blocked {order = array<i32: 1, 0>} : tensor<32x64xf32, #blocked1> -> tensor<64x32xf32, #blocked11>
378|       %store_layout = ttg.convert_layout %trans : tensor<64x32xf32, #blocked11> -> tensor<64x32xf32, #blocked5>
379|       %reshaped = tt.reshape %store_layout : tensor<64x32xf32, #blocked5> -> tensor<1x16x4x32xf32, #blocked6>
380|       %out = arith.truncf %reshaped : tensor<1x16x4x32xf32, #blocked6> to tensor<1x16x4x32xbf16, #blocked6>
381|       tt.descriptor_store %arg1[%c0_i32, %c0_i32, %c0_i32, %idx], %out : !tt.tensordesc<1x16x4x32xbf16>, tensor<1x16x4x32xbf16, #blocked6>
382|       %next = arith.addi %idx, %c32_i32 : i32
383|       scf.yield %next : i32
384|     }
385|     tt.return
```
**EN:** This function-oriented block defines or enters `descriptor_store_predicate`. Within it, the test exercises constants, layout conversions, tt.func, structured loops, tt.descriptor_load, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `descriptor_store_predicate` 为核心。测试在其中演示 常量、布局转换、tt.func、结构化循环、tt.descriptor_load，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 386-388
```mlir
386|   }
387| }
388| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 389-393
```mlir
389| // CHECK-LABEL: tt.func @descriptor_store_predicate
390| // CHECK-NOT: ttg.mask
391| // CHECK: scf.if %{{[0-9]+}} {
392| // CHECK-NEXT: tt.descriptor_store
393| // CHECK-NEXT: }
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: tt.func @descriptor_store_predicate anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: tt.func @descriptor_store_predicate 这样的标签用于锚定匹配范围。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `-tritonamdgpu-optimize-descriptor-encoding`, `-tritonamdgpu-schedule-loops="num_stages=2"`, `-tritonamdgpu-pipeline="use_async_copy=1"`, `-canonicalize`
- **CN:** 主要 pass 选项：`-split-input-file`，`-tritonamdgpu-optimize-descriptor-encoding`，`-tritonamdgpu-schedule-loops="num_stages=2"`，`-tritonamdgpu-pipeline="use_async_copy=1"`，`-canonicalize`
- **EN:** Dominant operations include `arith.constant`, `ttg.convert_layout`, `tt.make_tensor_descriptor`, `tt.func`, `arith.addi`, `arith.extsi`, `tt.descriptor_load`, `tt.get_program_id`, `arith.muli`, `module`.
- **CN:** 主要操作包括 `arith.constant`、`ttg.convert_layout`、`tt.make_tensor_descriptor`、`tt.func`、`arith.addi`、`arith.extsi`、`tt.descriptor_load`、`tt.get_program_id`、`arith.muli`、`module`。
- **EN:** The file contains 5 independently testable section(s). Check styles used: CHECK x40, CHECK-NOT x22, CHECK-NEXT x7, CHECK-LABEL x6. Important labels include tt.func @matmul_kernel_make_tensor_descriptor, tt.func @tdm_padding_fp8, tt.func @tdm_padding_f32, tt.func @gather_dot_pipeline. It validates both the presence of the desired rewrite and the absence of unwanted residual operations.
- **CN:** 该文件包含 5 个可独立测试的分段。使用的检查类型：CHECK ×40，CHECK-NOT ×22，CHECK-NEXT ×7，CHECK-LABEL ×6。 关键标签包括 tt.func @matmul_kernel_make_tensor_descriptor，tt.func @tdm_padding_fp8，tt.func @tdm_padding_f32，tt.func @gather_dot_pipeline。 它同时验证目标重写已经出现，并且不再残留不需要的操作。

## Dependencies / 依赖关系

- **EN:** This test depends on Triton/MLIR textual IR parsing, the pass pipeline named in the RUN lines, and the dialect/type definitions referenced in the file.
- **CN:** 该测试依赖 Triton/MLIR 文本 IR 解析、RUN 行中指定的 pass 流水线，以及文件中引用的方言/类型定义。
- **EN:** `tt`: Triton high-level tensor/pointer operations.
- **CN:** `tt`：Triton 高层张量/指针操作。
- **EN:** `arith`: MLIR arithmetic dialect for scalar/tensor math.
- **CN:** `arith`：用于标量/张量运算的 MLIR arithmetic 方言。
- **EN:** `ttg`: TritonGPU layout, memory, and GPU-specific operations.
- **CN:** `ttg`：TritonGPU 布局、内存和 GPU 专用操作。
- **EN:** `scf`: Structured control flow for loops and conditionals.
- **CN:** `scf`：用于循环与条件分支的结构化控制流。
- **EN:** `FileCheck` is used to match required textual patterns after rewriting/lowering.
- **CN:** `FileCheck` 用于在重写/lower 之后匹配必须出现的文本模式。