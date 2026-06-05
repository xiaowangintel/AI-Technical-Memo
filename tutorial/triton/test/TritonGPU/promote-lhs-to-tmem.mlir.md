# promote-lhs-to-tmem.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/promote-lhs-to-tmem.mlir`
- **EN:** Test for the MLIR/Triton pipeline `-tritongpu-promote-lhs-to-tmem` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `-tritongpu-promote-lhs-to-tmem` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -tritongpu-promote-lhs-to-tmem | FileCheck --dump-input-context=50 %s`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -tritongpu-promote-lhs-to-tmem | FileCheck --dump-input-context=50 %s`

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
1| // RUN: triton-opt %s -tritongpu-promote-lhs-to-tmem | FileCheck --dump-input-context=50 %s
2| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -tritongpu-promote-lhs-to-tmem | FileCheck --dump-input-context=50 %s` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -tritongpu-promote-lhs-to-tmem | FileCheck --dump-input-context=50 %s`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 3-9
```mlir
3| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
4| #blocked1 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
5| #blocked2 = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [32, 1], warpsPerCTA = [1, 4], order = [1, 0]}>
6| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
7| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
8| #tmem_scales = #ttng.tensor_memory_scales_encoding<>
9| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 10-10
```mlir
10| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 11-15
```mlir
11|   // CHECK-LABEL: @promote_lhs
12|   // CHECK: scf.for
13|   // CHECK: %[[A:.+]] = tt.load
14|   // CHECK: %[[A_TMEM:.+]] = ttng.tmem_alloc %[[A]]
15|   // CHECK: ttng.tc_gen5_mma %[[A_TMEM]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @promote_lhs anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @promote_lhs 这样的标签用于锚定匹配范围。

### Lines 16-33
```mlir
16|   tt.func public @promote_lhs(%A_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1>, %B_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1>, %arg3: i32) -> tensor<128x128xf16, #blocked1> {
17|     %true = arith.constant true
18|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked1>
19|     %c0_i32 = arith.constant 0 : i32
20|     %c1_i32 = arith.constant 1 : i32
21|     %B_multibuf = ttg.local_alloc : () -> !ttg.memdesc<1x128x128xf16, #shared, #ttg.shared_memory, mutable>
22|     %res = scf.for %i = %c0_i32 to %arg3 step %c1_i32 iter_args(%acc = %cst) -> (tensor<128x128xf32, #blocked1>)  : i32 {
23|       %A = tt.load %A_ptr : tensor<128x128x!tt.ptr<f16>, #blocked1>
24|       %A_sh = ttg.local_alloc %A : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>
25|       %B_sh = ttg.memdesc_index %B_multibuf[%c0_i32] : !ttg.memdesc<1x128x128xf16, #shared, #ttg.shared_memory, mutable> -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>
26|       %acc_tm = ttng.tmem_alloc %acc : (tensor<128x128xf32, #blocked1>) -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
27|       ttng.tc_gen5_mma %A_sh, %B_sh, %acc_tm, %true, %true : !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>, !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
28|       %acc_res = ttng.tmem_load %acc_tm : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked1>
29|       scf.yield %acc_res : tensor<128x128xf32, #blocked1>
30|     }
31|     ttg.local_dealloc %B_multibuf : !ttg.memdesc<1x128x128xf16, #shared, #ttg.shared_memory, mutable>
32|     %res_f16 = arith.truncf %res : tensor<128x128xf32, #blocked1> to tensor<128x128xf16, #blocked1>
33|     tt.return %res_f16 : tensor<128x128xf16, #blocked1>
```
**EN:** This function-oriented block defines or enters `promote_lhs`. Within it, the test exercises constants, tt.func, shared/local memory allocation, structured loops, masked or vectorized loads, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `promote_lhs` 为核心。测试在其中演示 常量、tt.func、共享/本地内存分配、结构化循环、带掩码或向量化的加载，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 34-35
```mlir
34|   }
35| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 36-40
```mlir
36|   // CHECK-LABEL: @promote_lhs_mxfp
37|   // CHECK: scf.for
38|   // CHECK: %[[A:.+]] = tt.load
39|   // CHECK: %[[A_TMEM:.+]] = ttng.tmem_alloc %[[A]]
40|   // CHECK: ttng.tc_gen5_mma_scaled %[[A_TMEM]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @promote_lhs_mxfp anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @promote_lhs_mxfp 这样的标签用于锚定匹配范围。

### Lines 41-58
```mlir
41|   tt.func public @promote_lhs_mxfp(%A_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1>, %B_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1>, %arg3: i32, %a_scale: tensor<128x1xi8, #blocked2>, %b_scale: tensor<64x1xi8, #blocked2>) -> tensor<128x128xf16, #blocked1> {
42|     %true = arith.constant true
43|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked1>
44|     %c0_i32 = arith.constant 0 : i32
45|     %c1_i32 = arith.constant 1 : i32
46|     %B_multibuf = ttg.local_alloc : () -> !ttg.memdesc<1x128x128xf16, #shared, #ttg.shared_memory, mutable>
47|     %res = scf.for %i = %c0_i32 to %arg3 step %c1_i32 iter_args(%acc = %cst) -> (tensor<128x128xf32, #blocked1>)  : i32 {
48|       %A = tt.load %A_ptr : tensor<128x128x!tt.ptr<f16>, #blocked1>
49|       %A_sh = ttg.local_alloc %A : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>
50|       %B_sh = ttg.memdesc_index %B_multibuf[%c0_i32] : !ttg.memdesc<1x128x128xf16, #shared, #ttg.shared_memory, mutable> -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>
51|       %acc_tm = ttng.tmem_alloc %acc : (tensor<128x128xf32, #blocked1>) -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
52|       %a_scale_tm = ttng.tmem_alloc %a_scale : (tensor<128x1xi8, #blocked2>) -> !ttg.memdesc<128x1xi8, #tmem_scales, #ttng.tensor_memory>
53|       %b_scale_tm = ttng.tmem_alloc %b_scale : (tensor<64x1xi8, #blocked2>) -> !ttg.memdesc<64x1xi8, #tmem_scales, #ttng.tensor_memory>
54|       ttng.tc_gen5_mma_scaled %A_sh, %B_sh, %acc_tm, %a_scale_tm, %b_scale_tm, %true, %true lhs = e5m2 rhs = e5m2 : !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>, !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.memdesc<128x1xi8, #tmem_scales, #ttng.tensor_memory>, !ttg.memdesc<64x1xi8, #tmem_scales, #ttng.tensor_memory>
55|       %acc_res = ttng.tmem_load %acc_tm : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked1>
56|       scf.yield %acc_res : tensor<128x128xf32, #blocked1>
57|     }
58|     ttg.local_dealloc %B_multibuf : !ttg.memdesc<1x128x128xf16, #shared, #ttg.shared_memory, mutable>
```
**EN:** This function-oriented block defines or enters `promote_lhs_mxfp`. Within it, the test exercises constants, tensor-memory allocation, tt.func, shared/local memory allocation, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `promote_lhs_mxfp` 为核心。测试在其中演示 常量、张量内存分配、tt.func、共享/本地内存分配、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 59-62
```mlir
59|     %res_f16 = arith.truncf %res : tensor<128x128xf32, #blocked1> to tensor<128x128xf16, #blocked1>
60|     tt.return %res_f16 : tensor<128x128xf16, #blocked1>
61|   }
62| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 63-67
```mlir
63|   // CHECK-LABEL: @dont_promote_rhs
64|   // CHECK: scf.for
65|   // CHECK: %[[B:.+]] = tt.load
66|   // CHECK: %[[B_TMEM:.+]] = ttg.local_alloc %[[B]]
67|   // CHECK: ttng.tc_gen5_mma %{{.+}}, %[[B_TMEM]], %{{.+}}, {{.+}}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @dont_promote_rhs anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @dont_promote_rhs 这样的标签用于锚定匹配范围。

### Lines 68-85
```mlir
68|   tt.func public @dont_promote_rhs(%A_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1>, %B_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1>, %arg3: i32) -> tensor<128x128xf16, #blocked1> {
69|     %true = arith.constant true
70|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked1>
71|     %c0_i32 = arith.constant 0 : i32
72|     %c1_i32 = arith.constant 1 : i32
73|     %A_multibuf = ttg.local_alloc : () -> !ttg.memdesc<1x128x128xf16, #shared, #ttg.shared_memory, mutable>
74|     %res = scf.for %i = %c0_i32 to %arg3 step %c1_i32 iter_args(%acc = %cst) -> (tensor<128x128xf32, #blocked1>)  : i32 {
75|       %B = tt.load %A_ptr : tensor<128x128x!tt.ptr<f16>, #blocked1>
76|       %B_sh = ttg.local_alloc %B : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>
77|       %A_sh = ttg.memdesc_index %A_multibuf[%c0_i32] : !ttg.memdesc<1x128x128xf16, #shared, #ttg.shared_memory, mutable> -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>
78|       %acc_tm = ttng.tmem_alloc %acc : (tensor<128x128xf32, #blocked1>) -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
79|       ttng.tc_gen5_mma %A_sh, %B_sh, %acc_tm, %true, %true : !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>, !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
80|       %acc_res = ttng.tmem_load %acc_tm : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked1>
81|       scf.yield %acc_res : tensor<128x128xf32, #blocked1>
82|     }
83|     ttg.local_dealloc %A_multibuf : !ttg.memdesc<1x128x128xf16, #shared, #ttg.shared_memory, mutable>
84|     %res_f16 = arith.truncf %res : tensor<128x128xf32, #blocked1> to tensor<128x128xf16, #blocked1>
85|     tt.return %res_f16 : tensor<128x128xf16, #blocked1>
```
**EN:** This function-oriented block defines or enters `dont_promote_rhs`. Within it, the test exercises constants, tt.func, shared/local memory allocation, structured loops, masked or vectorized loads, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `dont_promote_rhs` 为核心。测试在其中演示 常量、tt.func、共享/本地内存分配、结构化循环、带掩码或向量化的加载，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 86-87
```mlir
86|   }
87| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 88-92
```mlir
88|   // CHECK-LABEL: @dont_promote_long_lr
89|   // CHECK: %[[A:.+]] = tt.load
90|   // CHECK: %[[A_SMEM:.+]] = ttg.local_alloc %[[A]]
91|   // CHECK: scf.for
92|   // CHECK: ttng.tc_gen5_mma %[[A_SMEM]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @dont_promote_long_lr anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @dont_promote_long_lr 这样的标签用于锚定匹配范围。

### Lines 93-110
```mlir
 93|   tt.func public @dont_promote_long_lr(%A_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1>, %B_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1>, %arg3: i32) -> tensor<128x128xf16, #blocked1> {
 94|     %true = arith.constant true
 95|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked1>
 96|     %c0_i32 = arith.constant 0 : i32
 97|     %c1_i32 = arith.constant 1 : i32
 98|     %B_multibuf = ttg.local_alloc : () -> !ttg.memdesc<1x128x128xf16, #shared, #ttg.shared_memory, mutable>
 99|     %A = tt.load %A_ptr : tensor<128x128x!tt.ptr<f16>, #blocked1>
100|     %A_sh = ttg.local_alloc %A : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>
101|     %res = scf.for %i = %c0_i32 to %arg3 step %c1_i32 iter_args(%acc = %cst) -> (tensor<128x128xf32, #blocked1>)  : i32 {
102|       %B_sh = ttg.memdesc_index %B_multibuf[%c0_i32] : !ttg.memdesc<1x128x128xf16, #shared, #ttg.shared_memory, mutable> -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>
103|       %acc_tm = ttng.tmem_alloc %acc : (tensor<128x128xf32, #blocked1>) -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
104|       ttng.tc_gen5_mma %A_sh, %B_sh, %acc_tm, %true, %true : !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>, !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
105|       %acc_res = ttng.tmem_load %acc_tm : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked1>
106|       scf.yield %acc_res : tensor<128x128xf32, #blocked1>
107|     }
108|     ttg.local_dealloc %B_multibuf : !ttg.memdesc<1x128x128xf16, #shared, #ttg.shared_memory, mutable>
109|     %res_f16 = arith.truncf %res : tensor<128x128xf32, #blocked1> to tensor<128x128xf16, #blocked1>
110|     tt.return %res_f16 : tensor<128x128xf16, #blocked1>
```
**EN:** This function-oriented block defines or enters `dont_promote_long_lr`. Within it, the test exercises constants, tt.func, shared/local memory allocation, masked or vectorized loads, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `dont_promote_long_lr` 为核心。测试在其中演示 常量、tt.func、共享/本地内存分配、带掩码或向量化的加载、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 111-112
```mlir
111|   }
112| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 113-117
```mlir
113|   // CHECK-LABEL: @dont_convert_layout
114|   // CHECK: scf.for
115|   // CHECK: %[[A:.+]] = tt.load
116|   // CHECK: %[[A_SMEM:.+]] = ttg.local_alloc %[[A]]
117|   // CHECK: ttng.tc_gen5_mma %[[A_SMEM]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @dont_convert_layout anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @dont_convert_layout 这样的标签用于锚定匹配范围。

### Lines 118-135
```mlir
118|   tt.func public @dont_convert_layout(%A_ptr: tensor<128x128x!tt.ptr<f16>, #blocked2>, %B_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1>, %arg3: i32) -> tensor<128x128xf16, #blocked1> {
119|     %true = arith.constant true
120|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked1>
121|     %c0_i32 = arith.constant 0 : i32
122|     %c1_i32 = arith.constant 1 : i32
123|     %B_multibuf = ttg.local_alloc : () -> !ttg.memdesc<1x128x128xf16, #shared, #ttg.shared_memory, mutable>
124|     %res = scf.for %i = %c0_i32 to %arg3 step %c1_i32 iter_args(%acc = %cst) -> (tensor<128x128xf32, #blocked1>)  : i32 {
125|       %A = tt.load %A_ptr : tensor<128x128x!tt.ptr<f16>, #blocked2>
126|       %A_sh = ttg.local_alloc %A : (tensor<128x128xf16, #blocked2>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>
127|       %B_sh = ttg.memdesc_index %B_multibuf[%c0_i32] : !ttg.memdesc<1x128x128xf16, #shared, #ttg.shared_memory, mutable> -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>
128|       %acc_tm = ttng.tmem_alloc %acc : (tensor<128x128xf32, #blocked1>) -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
129|       ttng.tc_gen5_mma %A_sh, %B_sh, %acc_tm, %true, %true : !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>, !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
130|       %acc_res = ttng.tmem_load %acc_tm : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked1>
131|       scf.yield %acc_res : tensor<128x128xf32, #blocked1>
132|     }
133|     ttg.local_dealloc %B_multibuf : !ttg.memdesc<1x128x128xf16, #shared, #ttg.shared_memory, mutable>
134|     %res_f16 = arith.truncf %res : tensor<128x128xf32, #blocked1> to tensor<128x128xf16, #blocked1>
135|     tt.return %res_f16 : tensor<128x128xf16, #blocked1>
```
**EN:** This function-oriented block defines or enters `dont_convert_layout`. Within it, the test exercises constants, tt.func, shared/local memory allocation, structured loops, masked or vectorized loads, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `dont_convert_layout` 为核心。测试在其中演示 常量、tt.func、共享/本地内存分配、结构化循环、带掩码或向量化的加载，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 136-137
```mlir
136|   }
137| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 138-138
```mlir
138|   // CHECK-LABEL: @promote_lhs_arith
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @promote_lhs_arith anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @promote_lhs_arith 这样的标签用于锚定匹配范围。

### Lines 139-153
```mlir
139|   tt.func public @promote_lhs_arith(%A_ptr: tensor<128x128x!tt.ptr<f32>, #blocked2>, %B_sh: !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>, %arg3: i32) {
140|     %true = arith.constant true
141|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked1>
142|     // %[[A:.+]] = arith.truncf
143|     // %[[C:.+]] = ttg.convert_layout %[[A]]
144|     // %[[D:.+]] = ttng.tmem_alloc %[[C]]
145|     // ttng.tc_gen5_mma %[[D]]
146|     %A = tt.load %A_ptr : tensor<128x128x!tt.ptr<f32>, #blocked2>
147|     %A_f16 = arith.truncf %A : tensor<128x128xf32, #blocked2> to tensor<128x128xf16, #blocked2>
148|     %A_sh = ttg.local_alloc %A_f16 : (tensor<128x128xf16, #blocked2>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>
149|     %acc_tm = ttng.tmem_alloc %cst : (tensor<128x128xf32, #blocked1>) -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
150|     ttng.tc_gen5_mma %A_sh, %B_sh, %acc_tm, %true, %true : !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>, !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
151|     tt.return
152|   }
153| }
```
**EN:** This function-oriented block defines or enters `promote_lhs_arith`. Within it, the test exercises tt.func, constants, masked or vectorized loads, arith.truncf, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `promote_lhs_arith` 为核心。测试在其中演示 tt.func、常量、带掩码或向量化的加载、arith.truncf、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-tritongpu-promote-lhs-to-tmem`
- **CN:** 主要 pass 选项：`-tritongpu-promote-lhs-to-tmem`
- **EN:** Dominant operations include `arith.constant`, `tt.func`, `ttg.local_alloc`, `ttng.tmem_alloc`, `tt.load`, `arith.truncf`, `tt.return`, `scf.for`, `ttg.memdesc_index`, `ttng.tc_gen5_mma`.
- **CN:** 主要操作包括 `arith.constant`、`tt.func`、`ttg.local_alloc`、`ttng.tmem_alloc`、`tt.load`、`arith.truncf`、`tt.return`、`scf.for`、`ttg.memdesc_index`、`ttng.tc_gen5_mma`。
- **EN:** The file contains 1 independently testable section(s). Check styles used: CHECK x20, CHECK-LABEL x6. Important labels include @promote_lhs, @promote_lhs_mxfp, @dont_promote_rhs, @dont_promote_long_lr. It validates that the selected pass pipeline rewrites each labeled test case into the checked form.
- **CN:** 该文件包含 1 个可独立测试的分段。使用的检查类型：CHECK ×20，CHECK-LABEL ×6。 关键标签包括 @promote_lhs，@promote_lhs_mxfp，@dont_promote_rhs，@dont_promote_long_lr。 它验证所选 pass 流水线会把每个带标签的测试用例重写成检查所要求的形式。

## Dependencies / 依赖关系

- **EN:** This test depends on Triton/MLIR textual IR parsing, the pass pipeline named in the RUN lines, and the dialect/type definitions referenced in the file.
- **CN:** 该测试依赖 Triton/MLIR 文本 IR 解析、RUN 行中指定的 pass 流水线，以及文件中引用的方言/类型定义。
- **EN:** `ttg`: TritonGPU layout, memory, and GPU-specific operations.
- **CN:** `ttg`：TritonGPU 布局、内存和 GPU 专用操作。
- **EN:** `ttng`: Triton Nvidia GPU tensor-memory and MMA operations.
- **CN:** `ttng`：Triton Nvidia GPU 张量内存与 MMA 操作。
- **EN:** `tt`: Triton high-level tensor/pointer operations.
- **CN:** `tt`：Triton 高层张量/指针操作。
- **EN:** `arith`: MLIR arithmetic dialect for scalar/tensor math.
- **CN:** `arith`：用于标量/张量运算的 MLIR arithmetic 方言。
- **EN:** `scf`: Structured control flow for loops and conditionals.
- **CN:** `scf`：用于循环与条件分支的结构化控制流。
- **EN:** `FileCheck` is used to match required textual patterns after rewriting/lowering.
- **CN:** `FileCheck` 用于在重写/lower 之后匹配必须出现的文本模式。