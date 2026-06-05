# loop-pipeline-hopper.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/loop-pipeline-hopper.mlir`
- **EN:** Test for the MLIR/Triton pipeline `-tritongpu-assign-latencies, -tritongpu-schedule-loops, -tritongpu-pipeline, -canonicalize` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `-tritongpu-assign-latencies, -tritongpu-schedule-loops, -tritongpu-pipeline, -canonicalize` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -split-input-file -tritongpu-assign-latencies -tritongpu-schedule-loops -tritongpu-pipeline -canonicalize | FileCheck --dump-input-context=50 %s`; `// RUN: triton-opt %s -split-input-file -tritongpu-assign-latencies -tritongpu-schedule-loops -tritongpu-pipeline | FileCheck %s --check-prefix=CHECK-NOCANON`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -split-input-file -tritongpu-assign-latencies -tritongpu-schedule-loops -tritongpu-pipeline -canonicalize | FileCheck --dump-input-context=50 %s`；`// RUN: triton-opt %s -split-input-file -tritongpu-assign-latencies -tritongpu-schedule-loops -tritongpu-pipeline | FileCheck %s --check-prefix=CHECK-NOCANON`

## Line-by-Line Analysis / 逐行分析

### Lines 1-5
```mlir
1| // RUN: triton-opt %s -split-input-file -tritongpu-assign-latencies -tritongpu-schedule-loops -tritongpu-pipeline -canonicalize | FileCheck --dump-input-context=50 %s
2| // RUN: triton-opt %s -split-input-file -tritongpu-assign-latencies -tritongpu-schedule-loops -tritongpu-pipeline | FileCheck %s --check-prefix=CHECK-NOCANON
3| 
4| // 4 warps
5| // matmul: 128x32 @ 32x128 -> 128x128
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -split-input-file -tritongpu-assign-latencies -tritongpu-schedule-loops -tritongpu-pipeline -canonicalize | FileCheck --dump-input-context=50 %s` ; ` triton-opt %s -split-input-file -tritongpu-assign-latencies -tritongpu-schedule-loops -tritongpu-pipeline | FileCheck %s --check-prefix=CHECK-NOCANON` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -split-input-file -tritongpu-assign-latencies -tritongpu-schedule-loops -tritongpu-pipeline -canonicalize | FileCheck --dump-input-context=50 %s`；` triton-opt %s -split-input-file -tritongpu-assign-latencies -tritongpu-schedule-loops -tritongpu-pipeline | FileCheck %s --check-prefix=CHECK-NOCANON`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 6-14
```mlir
 6| #AL = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
 7| #BL = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
 8| #ALs0 = #ttg.slice<{parent=#AL, dim=0}>
 9| #BLs0 = #ttg.slice<{parent=#BL, dim=0}>
10| #C = #ttg.nvidia_mma<{versionMajor = 2, warpsPerCTA = [4, 1], instrShape = [16, 8]}>
11| #A = #ttg.dot_op<{opIdx = 0, parent = #C, kWidth=2}>
12| #B = #ttg.dot_op<{opIdx = 1, parent = #C, kWidth=2}>
13| #smem = #ttg.shared_memory
14| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 15-32
```mlir
15| // CHECK-LABEL: tt.func @matmul_loop
16| // CHECK-DAG: %[[CONSTANT_NEG1:.*]] = arith.constant -1 : i32
17| // CHECK-DAG: %[[CONSTANT_0:.*]] = arith.constant 0 : i32
18| // CHECK-DAG: %[[CONSTANT_1:.*]] = arith.constant 1 : i32
19| // CHECK-DAG: %[[CONSTANT_2:.*]] = arith.constant 2 : i32
20| // CHECK: %[[ABUFFER:.*]] = ttg.local_alloc
21| // CHECK: %[[BBUFFER:.*]] = ttg.local_alloc
22| // CHECK-DAG: %[[LOOP_COND_0:.*]] = arith.cmpi slt, %[[LB:.*]], %[[UB:.*]]
23| // CHECK-DAG: %[[LOOP_COND_0_SPLAT_A:.*]] = tt.splat %[[LOOP_COND_0]]
24| // CHECK-DAG: %[[ASUB:.*]] = ttg.memdesc_index %[[ABUFFER]]{{\[}}%[[CONSTANT_0]]{{\]}} : !ttg.memdesc<2x128x32xf16, #shared, #smem, mutable> -> !ttg.memdesc<128x32xf16, #shared, #smem, mutable>
25| // CHECK: %[[T_A0:.*]] = ttg.async_copy_global_to_local %{{.*}}, %[[ASUB]] mask %[[LOOP_COND_0_SPLAT_A]] {contiguity = 4 : i32} : tensor<128x32x!tt.ptr<f16>, #blocked1> -> <128x32xf16, #shared, #smem, mutable>
26| // CHECK-DAG: %[[LOOP_COND_0_SPLAT_B:.*]] = tt.splat %[[LOOP_COND_0]]
27| // CHECK-DAG: %[[BSUB:.*]] = ttg.memdesc_index %[[BBUFFER]]{{\[}}%[[CONSTANT_0]]{{\]}}
28| // CHECK: %[[T_B0:.*]] = ttg.async_copy_global_to_local %{{.*}}, %[[BSUB]] mask %[[LOOP_COND_0_SPLAT_B]] other %{{.*}} {contiguity = 4 : i32} : tensor<32x128x!tt.ptr<f16>, #blocked> -> <32x128xf16, #shared1, #smem, mutable>
29| // CHECK-DAG: %[[IV_1:.*]] = arith.addi %[[LB]], %[[STEP:.*]]
30| // CHECK-DAG: %[[LOOP_COND_1:.*]] = arith.cmpi slt, %[[IV_1]], %[[UB]]
31| // CHECK-DAG: %[[LOOP_COND_1_SPLAT_A:.*]] = tt.splat %[[LOOP_COND_1]]
32| // CHECK-DAG: %[[ASUB1:.*]] = ttg.memdesc_index %[[ABUFFER]]{{\[}}%[[CONSTANT_1]]{{\]}}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: tt.func @matmul_loop anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: tt.func @matmul_loop 这样的标签用于锚定匹配范围。

### Lines 33-50
```mlir
33| // CHECK: %[[T_A1:.*]] = ttg.async_copy_global_to_local %{{.*}}, %[[ASUB1]] mask %[[LOOP_COND_1_SPLAT_A]]
34| // CHECK-DAG: %[[LOOP_COND_1_SPLAT_B:.*]] = tt.splat %[[LOOP_COND_1]]
35| // CHECK-DAG: %[[BSUB1:.*]] = ttg.memdesc_index %[[BBUFFER]]{{\[}}%[[CONSTANT_1]]{{\]}}
36| // CHECK: %[[T_B1:.*]] = ttg.async_copy_global_to_local %{{.*}}, %[[BSUB1]] mask %[[LOOP_COND_1_SPLAT_B]]
37| // CHECK: scf.for {{.*}} iter_args({{.*}}, %[[INS_IDX:.*]] = %[[CONSTANT_1]], %[[EXT_IDX:.*]] = %[[CONSTANT_NEG1]]
38| // CHECK:   %[[EXT_IDX_2:.*]] = arith.addi %[[EXT_IDX]], %[[CONSTANT_1]] : i32
39| // CHECK:   %[[CMP_EXT:.*]] = arith.cmpi sge, %[[EXT_IDX_2]], %[[CONSTANT_2]]
40| // CHECK:   %[[EXT_IDX_3:.*]] = arith.select %[[CMP_EXT]], %[[CONSTANT_0]], %[[EXT_IDX_2]]
41| // CHECK:   ttg.async_wait {{.*}} {num = 2 : i32}
42| // CHECK:   %[[A:.*]] = ttg.memdesc_index %[[ABUFFER]]{{\[}}%[[EXT_IDX_3]]{{\]}}
43| // CHECK:   %[[arg_a0_dot_op:.*]] = ttg.local_load %[[A]]
44| // CHECK:   %[[B:.*]] = ttg.memdesc_index %[[BBUFFER]]{{\[}}%[[EXT_IDX_3]]{{\]}}
45| // CHECK:   %[[arg_b0_dot_op_0:.*]] = ttg.local_load %[[B]]
46| // CHECK:   tt.dot %[[arg_a0_dot_op]], %[[arg_b0_dot_op_0]], {{.*}}
47| // CHECK-DAG: %[[INS_IDX_2:.*]] = arith.addi %[[INS_IDX]], %[[CONSTANT_1]] : i32
48| // CHECK-DAG: %[[CMP_INS:.*]] = arith.cmpi sge, %[[INS_IDX_2]], %[[CONSTANT_2]]
49| // CHECK-DAG: %[[INS_IDX_3:.*]] = arith.select %[[CMP_INS]], %[[CONSTANT_0]], %[[INS_IDX_2]]
50| // CHECK:   %[[ASUB3:.*]] = ttg.memdesc_index %[[ABUFFER]]{{\[}}%[[INS_IDX_3]]{{\]}}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 51-54
```mlir
51| // CHECK:   %[[NEXT_A_BUFFER:.*]] = ttg.async_copy_global_to_local {{.*}}, %[[ASUB3]]
52| // CHECK:   %[[BSUB3:.*]] = ttg.memdesc_index %[[BBUFFER]]{{\[}}%[[INS_IDX_3]]{{\]}}
53| // CHECK:   %[[NEXT_B_BUFFER:.*]] = ttg.async_copy_global_to_local {{.*}}, %[[BSUB3]]
54| // CHECK:   scf.yield {{.*}}, %[[INS_IDX_3]], %[[EXT_IDX_3]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 55-55
```mlir
55| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 56-73
```mlir
56| tt.func @matmul_loop(%lb : index, %ub : index, %step : index,
57|                        %A : !tt.ptr<f16> {tt.divisibility = 16 : i32},
58|                        %B : !tt.ptr<f16> {tt.divisibility = 16 : i32}) {
59|   // A ptrs
60|   %a_ptr_splat = tt.splat %A : !tt.ptr<f16> -> tensor<128x32x!tt.ptr<f16>, #AL>
61|   %a_tmp0 = tt.make_range {end = 32: i32, start = 0: i32} : tensor<32xi32, #ALs0>
62|   %a_tmp1 = tt.expand_dims %a_tmp0 {axis = 0 : i32} : tensor<32xi32, #ALs0> -> tensor<1x32xi32, #AL>
63|   %a_offs = tt.broadcast %a_tmp1 : tensor<1x32xi32, #AL> -> tensor<128x32xi32, #AL>
64|   %a_ptr_init = tt.addptr %a_ptr_splat, %a_offs : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<128x32xi32, #AL>
65|   // B ptrs
66|   %b_ptr_splat = tt.splat %B : !tt.ptr<f16> -> tensor<32x128x!tt.ptr<f16>, #BL>
67|   %b_tmp0 = tt.make_range {end = 128: i32, start = 0: i32} : tensor<128xi32, #BLs0>
68|   %b_tmp1 = tt.expand_dims %b_tmp0 {axis = 0 : i32} : tensor<128xi32, #BLs0> -> tensor<1x128xi32, #BL>
69|   %b_offs = tt.broadcast %b_tmp1 : tensor<1x128xi32, #BL> -> tensor<32x128xi32, #BL>
70|   %b_ptr_init = tt.addptr %b_ptr_splat, %b_offs : tensor<32x128x!tt.ptr<f16>, #BL>, tensor<32x128xi32, #BL>
71| 
72| 
73|   %a_mask = arith.constant dense<true> : tensor<128x32xi1, #AL>
```
**EN:** This function-oriented block defines or enters `matmul_loop`. Within it, the test exercises tt.func, broadcasted scalars or pointers, lane/block index ranges, shape expansion, tensor broadcasting, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `matmul_loop` 为核心。测试在其中演示 tt.func、广播后的标量或指针、lane/block 索引范围、形状扩展、张量广播，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 74-91
```mlir
74|   %a_other = arith.constant dense<0.00e+00> : tensor<128x32xf16, #AL>
75|   %b_mask = arith.constant dense<true> : tensor<32x128xi1, #BL>
76|   %b_other = arith.constant dense<0.00e+00> : tensor<32x128xf16, #BL>
77|   %c_init = arith.constant dense<0.00e+00> : tensor<128x128xf32, #C>
78| 
79|   %a_off = arith.constant dense<4> : tensor<128x32xi32, #AL>
80|   %b_off = arith.constant dense<4> : tensor<32x128xi32, #BL>
81| 
82|   scf.for %iv = %lb to %ub step %step iter_args(%a_ptr = %a_ptr_init, %b_ptr = %b_ptr_init, %prev_c = %c_init) -> (tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #C>) {
83|     %a_ = tt.load %a_ptr : tensor<128x32x!tt.ptr<f16>, #AL>
84|     %a = ttg.convert_layout %a_ : tensor<128x32xf16, #AL> -> tensor<128x32xf16, #A>
85|     %b_ = tt.load %b_ptr, %b_mask, %b_other : tensor<32x128x!tt.ptr<f16>, #BL>
86|     %b = ttg.convert_layout %b_ : tensor<32x128xf16, #BL> -> tensor<32x128xf16, #B>
87| 
88|     %c = tt.dot %a, %b, %prev_c : tensor<128x32xf16, #A> * tensor<32x128xf16, #B> -> tensor<128x128xf32, #C>
89| 
90|     %next_a_ptr = tt.addptr %a_ptr, %a_off : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<128x32xi32, #AL>
91|     %next_b_ptr = tt.addptr %b_ptr, %b_off : tensor<32x128x!tt.ptr<f16>, #BL>, tensor<32x128xi32, #BL>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining constants, masked or vectorized loads, layout conversions, pointer arithmetic, structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 常量、带掩码或向量化的加载、布局转换、指针算术、结构化循环。

### Lines 92-97
```mlir
92|     scf.yield %next_a_ptr, %next_b_ptr, %c : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #C>
93|   }
94|   tt.return
95| }
96| }
97| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、tt.return。

### Lines 98-98
```mlir
98| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 99-106
```mlir
 99| 
100| #blocked = #ttg.blocked<{sizePerThread = [8, 1], threadsPerWarp = [8, 4], warpsPerCTA = [1, 4], order = [0, 1]}>
101| #blocked1 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
102| #mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 64, 16]}>
103| #mma1 = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 16, 16]}>
104| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
105| #shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = true, elementBitWidth = 16}>
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

### Lines 108-108
```mlir
108| // CHECK-LABEL: dot_chained_single_load
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: dot_chained_single_load anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: dot_chained_single_load 这样的标签用于锚定匹配范围。

### Lines 109-126
```mlir
109|   tt.func @dot_chained_single_load(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32}) -> tensor<128x64xf32, #mma> {
110|     %cst = arith.constant dense<0> : tensor<64x16xi32, #blocked>
111|     %c0_i32 = arith.constant 0 : i32
112|     %cst_0 = arith.constant dense<0> : tensor<1x16xi32, #blocked>
113|     %cst_1 = arith.constant dense<0> : tensor<128x1xi32, #blocked1>
114|     %c0_i64 = arith.constant 0 : i64
115|     %cst_2 = arith.constant dense<0.000000e+00> : tensor<128x16xf32, #mma1>
116|     %cst_3 = arith.constant dense<0.000000e+00> : tensor<128x64xf32, #mma>
117|     %c1_i32 = arith.constant 1 : i32
118|     %c8_i32 = arith.constant 8 : i32
119|     %0 = tt.addptr %arg0, %c0_i64 : !tt.ptr<f16>, i64
120|     %1 = tt.addptr %arg1, %c0_i64 : !tt.ptr<f16>, i64
121|     %2 = tt.splat %1 : !tt.ptr<f16> -> tensor<128x1x!tt.ptr<f16>, #blocked1>
122|     %3 = tt.addptr %2, %cst_1 : tensor<128x1x!tt.ptr<f16>, #blocked1>, tensor<128x1xi32, #blocked1>
123|     %4 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>
124|     %5 = tt.expand_dims %4 {axis = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked1}>> -> tensor<1x64xi32, #blocked1>
125|     %6 = tt.broadcast %3 : tensor<128x1x!tt.ptr<f16>, #blocked1> -> tensor<128x64x!tt.ptr<f16>, #blocked1>
126|     %7 = tt.broadcast %5 : tensor<1x64xi32, #blocked1> -> tensor<128x64xi32, #blocked1>
```
**EN:** This function-oriented block defines or enters `dot_chained_single_load`. Within it, the test exercises constants, pointer arithmetic, tt.func, tensor broadcasting, broadcasted scalars or pointers, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `dot_chained_single_load` 为核心。测试在其中演示 常量、指针算术、tt.func、张量广播、广播后的标量或指针，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 127-135
```mlir
127|     %8 = tt.addptr %6, %7 : tensor<128x64x!tt.ptr<f16>, #blocked1>, tensor<128x64xi32, #blocked1>
128|     %9 = tt.load %8 : tensor<128x64x!tt.ptr<f16>, #blocked1>
129|     %10 = tt.splat %0 : !tt.ptr<f16> -> tensor<1x16x!tt.ptr<f16>, #blocked>
130|     %11 = tt.addptr %10, %cst_0 : tensor<1x16x!tt.ptr<f16>, #blocked>, tensor<1x16xi32, #blocked>
131|     %12 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
132|     %13 = tt.expand_dims %12 {axis = 1 : i32} : tensor<64xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<64x1xi32, #blocked>
133|     %14 = tt.broadcast %11 : tensor<1x16x!tt.ptr<f16>, #blocked> -> tensor<64x16x!tt.ptr<f16>, #blocked>
134|     %15 = tt.broadcast %13 : tensor<64x1xi32, #blocked> -> tensor<64x16xi32, #blocked>
135|     %16 = tt.addptr %14, %15 : tensor<64x16x!tt.ptr<f16>, #blocked>, tensor<64x16xi32, #blocked>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on pointer arithmetic, tensor broadcasting, masked or vectorized loads, broadcasted scalars or pointers, lane/block index ranges. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 指针算术、张量广播、带掩码或向量化的加载、广播后的标量或指针、lane/block 索引范围。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 136-153
```mlir
136|     // CHECK: scf.for
137|     // CHECK:   ttg.async_wait {{.*}} {num = 1 : i32}
138|     // CHECK:   ttng.warp_group_dot
139|     // CHECK-NEXT: ttng.warp_group_dot_wait {{.*}} {pendings = 0 : i32}
140|     // CHECK:   ttng.warp_group_dot
141|     // CHECK:   ttg.async_copy_global_to_local
142|     // CHECK:   ttg.async_commit_group
143|     // CHECK:   scf.yield
144|     %17:2 = scf.for %arg3 = %c0_i32 to %c8_i32 step %c1_i32 iter_args(%arg4 = %cst_3, %arg5 = %16) -> (tensor<128x64xf32, #mma>, tensor<64x16x!tt.ptr<f16>, #blocked>)  : i32 {
145|       %18 = tt.load %arg5 : tensor<64x16x!tt.ptr<f16>, #blocked>
146|       %19 = ttg.local_alloc %9 : (tensor<128x64xf16, #blocked1>) -> !ttg.memdesc<128x64xf16, #shared, #smem>
147|       %20 = ttg.local_alloc %18 : (tensor<64x16xf16, #blocked>) -> !ttg.memdesc<64x16xf16, #shared1, #smem>
148|       %21 = ttng.warp_group_dot %19, %20, %cst_2 : !ttg.memdesc<128x64xf16, #shared, #smem> * !ttg.memdesc<64x16xf16, #shared1, #smem> -> tensor<128x16xf32, #mma1>
149|       %22 = arith.truncf %21 : tensor<128x16xf32, #mma1> to tensor<128x16xf16, #mma1>
150|       %23 = ttg.memdesc_trans %20 {order=array<i32: 1,0>} : !ttg.memdesc<64x16xf16, #shared1, #smem> -> !ttg.memdesc<16x64xf16, #shared, #smem>
151|       %24 = ttg.convert_layout %22 : tensor<128x16xf16, #mma1> -> tensor<128x16xf16, #ttg.dot_op<{opIdx = 0, parent = #mma1, kWidth = 2}>>
152|       %25 = ttng.warp_group_dot %24, %23, %arg4 : tensor<128x16xf16, #ttg.dot_op<{opIdx = 0, parent = #mma1, kWidth = 2}>> * !ttg.memdesc<16x64xf16, #shared, #smem> -> tensor<128x64xf32, #mma>
153|       %26 = tt.addptr %arg5, %cst : tensor<64x16x!tt.ptr<f16>, #blocked>, tensor<64x16xi32, #blocked>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining shared/local memory allocation, ttng.warp_group_dot, structured loops, masked or vectorized loads, arith.truncf.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 共享/本地内存分配、ttng.warp_group_dot、结构化循环、带掩码或向量化的加载、arith.truncf。

### Lines 154-159
```mlir
154|       scf.yield %25, %26 : tensor<128x64xf32, #mma>, tensor<64x16x!tt.ptr<f16>, #blocked>
155|     }
156|     tt.return %17#0 : tensor<128x64xf32, #mma>
157|   }
158| 
159|   // Check that we are able to perform WGMMA pipelining if the accumulator is conditionally being modified
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、tt.return。

### Lines 160-160
```mlir
160|   // CHECK-LABEL: dot_acc_cond_modified
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: dot_acc_cond_modified anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: dot_acc_cond_modified 这样的标签用于锚定匹配范围。

### Lines 161-178
```mlir
161|   tt.func @dot_acc_cond_modified(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %ext : i32) -> tensor<128x16xf32, #mma1> {
162|     %cst = arith.constant dense<0> : tensor<64x16xi32, #blocked>
163|     %cst2 = arith.constant dense<0> : tensor<128x64xi32, #blocked1>
164|     %c0_i32 = arith.constant 0 : i32
165|     %cst_0 = arith.constant dense<0> : tensor<1x16xi32, #blocked>
166|     %cst_1 = arith.constant dense<0> : tensor<128x1xi32, #blocked1>
167|     %c0_i64 = arith.constant 0 : i64
168|     %cst_2 = arith.constant dense<0.000000e+00> : tensor<128x16xf32, #mma1>
169|     %c1_i32 = arith.constant 1 : i32
170|     %c8_i32 = arith.constant 8 : i32
171|     %2 = tt.splat %arg1 : !tt.ptr<f16> -> tensor<128x1x!tt.ptr<f16>, #blocked1>
172|     %4 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>
173|     %5 = tt.expand_dims %4 {axis = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked1}>> -> tensor<1x64xi32, #blocked1>
174|     %6 = tt.broadcast %2 : tensor<128x1x!tt.ptr<f16>, #blocked1> -> tensor<128x64x!tt.ptr<f16>, #blocked1>
175|     %7 = tt.broadcast %5 : tensor<1x64xi32, #blocked1> -> tensor<128x64xi32, #blocked1>
176|     %8 = tt.addptr %6, %7 : tensor<128x64x!tt.ptr<f16>, #blocked1>, tensor<128x64xi32, #blocked1>
177|     %10 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<1x16x!tt.ptr<f16>, #blocked>
178|     %12 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
```
**EN:** This function-oriented block defines or enters `dot_acc_cond_modified`. Within it, the test exercises constants, tt.func, broadcasted scalars or pointers, lane/block index ranges, tensor broadcasting, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `dot_acc_cond_modified` 为核心。测试在其中演示 常量、tt.func、广播后的标量或指针、lane/block 索引范围、张量广播，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 179-182
```mlir
179|     %13 = tt.expand_dims %12 {axis = 1 : i32} : tensor<64xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<64x1xi32, #blocked>
180|     %14 = tt.broadcast %10 : tensor<1x16x!tt.ptr<f16>, #blocked> -> tensor<64x16x!tt.ptr<f16>, #blocked>
181|     %15 = tt.broadcast %13 : tensor<64x1xi32, #blocked> -> tensor<64x16xi32, #blocked>
182|     %16 = tt.addptr %14, %15 : tensor<64x16x!tt.ptr<f16>, #blocked>, tensor<64x16xi32, #blocked>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tensor broadcasting, shape expansion, pointer arithmetic. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 张量广播、形状扩展、指针算术。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 183-200
```mlir
183|     // CHECK: scf.for
184|     // CHECK:   ttg.async_wait {{.*}} {num = 2 : i32}
185|     // CHECK:   ttng.warp_group_dot
186|     // CHECK-NEXT: ttng.warp_group_dot_wait {{.*}} {pendings = 1 : i32}
187|     // CHECK:   ttg.async_copy_global_to_local
188|     // CHECK:   ttg.async_commit_group
189|     // CHECK:   scf.if
190|     // CHECK:     ttng.warp_group_dot_wait {{.*}} {pendings = 0 : i32}
191|     // CHECK:     arith.mulf
192|     // CHECK:     scf.yield
193|     // CHECK:   scf.yield
194|     // CHECK:   ttng.warp_group_dot_wait {{.*}} {pendings = 0 : i32}
195|     %17:3 = scf.for %arg3 = %c0_i32 to %c8_i32 step %c1_i32 iter_args(%arg4 = %cst_2, %arg5 = %16, %arg6 = %8) -> (tensor<128x16xf32, #mma1>, tensor<64x16x!tt.ptr<f16>, #blocked>, tensor<128x64x!tt.ptr<f16>, #blocked1>)  : i32 {
196|       %9 = tt.load %arg6 : tensor<128x64x!tt.ptr<f16>, #blocked1>
197|       %18 = tt.load %arg5 : tensor<64x16x!tt.ptr<f16>, #blocked>
198|       %19 = ttg.local_alloc %9 : (tensor<128x64xf16, #blocked1>) -> !ttg.memdesc<128x64xf16, #shared, #smem>
199|       %20 = ttg.local_alloc %18 : (tensor<64x16xf16, #blocked>) -> !ttg.memdesc<64x16xf16, #shared1, #smem>
200|       %acc = ttng.warp_group_dot %19, %20, %arg4 : !ttg.memdesc<128x64xf16, #shared, #smem> * !ttg.memdesc<64x16xf16, #shared1, #smem> -> tensor<128x16xf32, #mma1>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining masked or vectorized loads, shared/local memory allocation, structured loops, ttng.warp_group_dot.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 带掩码或向量化的加载、共享/本地内存分配、结构化循环、ttng.warp_group_dot。

### Lines 201-215
```mlir
201|       %cnd = arith.cmpi slt, %arg3, %ext : i32
202|       %acc_ = scf.if %cnd -> (tensor<128x16xf32, #mma1>) {
203|         %acc_zero = arith.mulf %acc, %cst_2 : tensor<128x16xf32, #mma1>
204|         scf.yield %acc_zero : tensor<128x16xf32, #mma1>
205|       } else {
206|         scf.yield %acc : tensor<128x16xf32, #mma1>
207|       }
208|       %22 = tt.addptr %arg5, %cst : tensor<64x16x!tt.ptr<f16>, #blocked>, tensor<64x16xi32, #blocked>
209|       %23 = tt.addptr %arg6, %cst2 : tensor<128x64x!tt.ptr<f16>, #blocked1>, tensor<128x64xi32, #blocked1>
210|       scf.yield %acc_, %22, %23 : tensor<128x16xf32, #mma1>, tensor<64x16x!tt.ptr<f16>, #blocked>, tensor<128x64x!tt.ptr<f16>, #blocked1>
211|     }
212|     tt.return %17#0 : tensor<128x16xf32, #mma1>
213|   }
214| }
215| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, pointer arithmetic, integer comparisons, structured conditionals, arith.mulf.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、指针算术、整数比较、结构化条件分支、arith.mulf。

### Lines 216-216
```mlir
216| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 217-224
```mlir
217| 
218| #blocked = #ttg.blocked<{sizePerThread = [8, 1], threadsPerWarp = [8, 4], warpsPerCTA = [1, 4], order = [0, 1]}>
219| #blocked1 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
220| #mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 64, 16]}>
221| #mma1 = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 16, 16]}>
222| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
223| #shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = true, elementBitWidth = 16}>
224| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 225-225
```mlir
225| module attributes {"ttg.target" = "cuda:90", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 226-226
```mlir
226| // CHECK-LABEL: two_accumulator_escape
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: two_accumulator_escape anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: two_accumulator_escape 这样的标签用于锚定匹配范围。

### Lines 227-244
```mlir
227|   tt.func @two_accumulator_escape(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32}) -> (tensor<128x64xf32, #mma>, tensor<128x16xf32, #mma1>) {
228|     %cst = arith.constant dense<0> : tensor<64x16xi32, #blocked>
229|     %c0_i32 = arith.constant 0 : i32
230|     %cst_0 = arith.constant dense<0> : tensor<1x16xi32, #blocked>
231|     %cst_1 = arith.constant dense<0> : tensor<128x1xi32, #blocked1>
232|     %c0_i64 = arith.constant 0 : i64
233|     %cst_2 = arith.constant dense<0.000000e+00> : tensor<128x16xf32, #mma1>
234|     %cst_3 = arith.constant dense<0.000000e+00> : tensor<128x64xf32, #mma>
235|     %cst_4 = arith.constant dense<1.000000e+00> : tensor<128x16xf16, #ttg.dot_op<{opIdx = 0, parent = #mma1, kWidth = 2}>>
236|     %c1_i32 = arith.constant 1 : i32
237|     %c8_i32 = arith.constant 8 : i32
238|     %0 = tt.addptr %arg0, %c0_i64 : !tt.ptr<f16>, i64
239|     %1 = tt.addptr %arg1, %c0_i64 : !tt.ptr<f16>, i64
240|     %2 = tt.splat %1 : !tt.ptr<f16> -> tensor<128x1x!tt.ptr<f16>, #blocked1>
241|     %3 = tt.addptr %2, %cst_1 : tensor<128x1x!tt.ptr<f16>, #blocked1>, tensor<128x1xi32, #blocked1>
242|     %4 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>
243|     %5 = tt.expand_dims %4 {axis = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked1}>> -> tensor<1x64xi32, #blocked1>
244|     %6 = tt.broadcast %3 : tensor<128x1x!tt.ptr<f16>, #blocked1> -> tensor<128x64x!tt.ptr<f16>, #blocked1>
```
**EN:** This function-oriented block defines or enters `two_accumulator_escape`. Within it, the test exercises constants, pointer arithmetic, tt.func, broadcasted scalars or pointers, lane/block index ranges, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `two_accumulator_escape` 为核心。测试在其中演示 常量、指针算术、tt.func、广播后的标量或指针、lane/block 索引范围，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 245-257
```mlir
245|     %7 = tt.broadcast %5 : tensor<1x64xi32, #blocked1> -> tensor<128x64xi32, #blocked1>
246|     %8 = tt.addptr %6, %7 : tensor<128x64x!tt.ptr<f16>, #blocked1>, tensor<128x64xi32, #blocked1>
247|     %9 = tt.load %8 : tensor<128x64x!tt.ptr<f16>, #blocked1>
248|     %10 = tt.splat %0 : !tt.ptr<f16> -> tensor<1x16x!tt.ptr<f16>, #blocked>
249|     %11 = tt.addptr %10, %cst_0 : tensor<1x16x!tt.ptr<f16>, #blocked>, tensor<1x16xi32, #blocked>
250|     %12 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
251|     %13 = tt.expand_dims %12 {axis = 1 : i32} : tensor<64xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<64x1xi32, #blocked>
252|     %14 = tt.broadcast %11 : tensor<1x16x!tt.ptr<f16>, #blocked> -> tensor<64x16x!tt.ptr<f16>, #blocked>
253|     %15 = tt.broadcast %13 : tensor<64x1xi32, #blocked> -> tensor<64x16xi32, #blocked>
254|     %16 = tt.addptr %14, %15 : tensor<64x16x!tt.ptr<f16>, #blocked>, tensor<64x16xi32, #blocked>
255|     %18 = tt.load %16 : tensor<64x16x!tt.ptr<f16>, #blocked>
256|     %19 = ttg.local_alloc %9 : (tensor<128x64xf16, #blocked1>) -> !ttg.memdesc<128x64xf16, #shared, #smem>
257|     %20 = ttg.local_alloc %18 : (tensor<64x16xf16, #blocked>) -> !ttg.memdesc<64x16xf16, #shared1, #smem>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tensor broadcasting, pointer arithmetic, masked or vectorized loads, shared/local memory allocation, broadcasted scalars or pointers. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 张量广播、指针算术、带掩码或向量化的加载、共享/本地内存分配、广播后的标量或指针。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 258-275
```mlir
258|     // CHECK: %[[ALLOC1:.+]] = ttg.local_alloc
259|     // CHECK: %[[ALLOC2:.+]] = ttg.local_alloc
260|     // CHECK: %[[R:.+]]:{{.+}} = scf.for
261|     // CHECK:   %[[DOT1:.+]] = ttng.warp_group_dot{{.*}}
262|     // CHECK:   ttg.async_wait {{.*}} {num = 1 : i32}
263|     // CHECK:   %[[TRANS:.+]] = ttg.memdesc_trans{{.*}} : !ttg.memdesc
264|     // CHECK:   %[[DOT2:.+]] = ttng.warp_group_dot{{.*}} %[[TRANS]]
265|     // CHECK:   ttng.warp_group_dot_wait %[[DOT1]], %[[DOT2]], %[[ALLOC1]], %[[ALLOC2]], %[[TRANS]] {pendings = 2 : i32}
266|     // CHECK:   scf.yield
267|     // CHECK: %{{.*}}:2 = ttng.warp_group_dot_wait %[[R]]#{{.+}}, %[[R]]#{{.+}} {pendings = 0 : i32} : tensor<128x16xf32, #{{.*}}>, tensor<128x64xf32, #{{.*}}>
268|     %17:3 = scf.for %arg3 = %c0_i32 to %c8_i32 step %c1_i32 iter_args(%arg4 = %cst_3, %arg5 = %16, %arg6 = %cst_2) -> (tensor<128x64xf32, #mma>, tensor<64x16x!tt.ptr<f16>, #blocked>, tensor<128x16xf32, #mma1>)  : i32 {
269|       %21 = ttng.warp_group_dot %19, %20, %arg6 : !ttg.memdesc<128x64xf16, #shared, #smem> * !ttg.memdesc<64x16xf16, #shared1, #smem> -> tensor<128x16xf32, #mma1>
270|       %l = tt.load %arg5 : tensor<64x16x!tt.ptr<f16>, #blocked>
271|       %c = ttg.local_alloc %l : (tensor<64x16xf16, #blocked>) -> !ttg.memdesc<64x16xf16, #shared1, #smem>
272|       %23 = ttg.memdesc_trans %c {order=array<i32: 1,0>} : !ttg.memdesc<64x16xf16, #shared1, #smem> -> !ttg.memdesc<16x64xf16, #shared, #smem>
273|       %25 = ttng.warp_group_dot %cst_4, %23, %arg4 : tensor<128x16xf16, #ttg.dot_op<{opIdx = 0, parent = #mma1, kWidth = 2}>> * !ttg.memdesc<16x64xf16, #shared, #smem> -> tensor<128x64xf32, #mma>
274|       %26 = tt.addptr %arg5, %cst : tensor<64x16x!tt.ptr<f16>, #blocked>, tensor<64x16xi32, #blocked>
275|       scf.yield %25, %26, %21 : tensor<128x64xf32, #mma>, tensor<64x16x!tt.ptr<f16>, #blocked>, tensor<128x16xf32, #mma1>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining ttng.warp_group_dot, structured loops, masked or vectorized loads, shared/local memory allocation, ttg.memdesc_trans.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 ttng.warp_group_dot、结构化循环、带掩码或向量化的加载、共享/本地内存分配、ttg.memdesc_trans。

### Lines 276-280
```mlir
276|     }
277|     tt.return %17#0, %17#2 : tensor<128x64xf32, #mma>, tensor<128x16xf32, #mma1>
278|   }
279| }
280| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 281-281
```mlir
281| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 282-292
```mlir
282| 
283| #blocked = #ttg.blocked<{sizePerThread = [4, 4], threadsPerWarp = [1, 32], warpsPerCTA = [4, 2], order = [1, 0]}>
284| #blocked1 = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [8, 4], warpsPerCTA = [8, 1], order = [1, 0]}>
285| #blocked2 = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [2, 16], warpsPerCTA = [8, 1], order = [1, 0]}>
286| #mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [8, 1], instrShape = [16, 256, 32]}>
287| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = false, elementBitWidth = 8}>
288| #shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = true, elementBitWidth = 8}>
289| #smem = #ttg.shared_memory
290| 
291| // Make sure that if one of the load dot operand is not pipelined (and therefore not double buffered) we won't use
292| // async dot.
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 293-293
```mlir
293| module attributes {"ttg.target" = "cuda:90", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 294-294
```mlir
294|   // CHECK-LABEL: no_wgmma_pipeline
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: no_wgmma_pipeline anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: no_wgmma_pipeline 这样的标签用于锚定匹配范围。

### Lines 295-312
```mlir
295|   tt.func public @no_wgmma_pipeline(%arg0: !tt.ptr<f8E5M2> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f8E5M2> {tt.divisibility = 16 : i32}, %arg2: !tt.ptr<f8E5M2> {tt.divisibility = 16 : i32}) {
296|     %cst = arith.constant dense<0.000000e+00> : tensor<128x256xf32, #mma>
297|     %c0_i32 = arith.constant 0 : i32
298|     %c1_i32 = arith.constant 1 : i32
299|     %c8_i32 = arith.constant 8 : i32
300|     %cst_0 = arith.constant dense<512> : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
301|     %cst_1 = arith.constant dense<512> : tensor<256xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>
302|     %cst_2 = arith.constant dense<512> : tensor<128x1xi32, #blocked>
303|     %cst_3 = arith.constant dense<512> : tensor<128x1xi32, #blocked1>
304|     %cst_4 = arith.constant dense<512> : tensor<64x1xi32, #blocked1>
305|     %cst_5 = arith.constant dense<32768> : tensor<64x256xi32, #blocked1>
306|     %cst_6 = arith.constant dense<64> : tensor<128x64xi32, #blocked>
307|     %0 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
308|     %1 = arith.remsi %0, %cst_0 : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
309|     %2 = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>
310|     %3 = arith.remsi %2, %cst_1 : tensor<256xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>
311|     %4 = tt.expand_dims %1 {axis = 1 : i32} : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<128x1xi32, #blocked>
312|     %5 = arith.muli %4, %cst_2 : tensor<128x1xi32, #blocked>
```
**EN:** This function-oriented block defines or enters `no_wgmma_pipeline`. Within it, the test exercises constants, tt.func, lane/block index ranges, arith.remsi, shape expansion, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `no_wgmma_pipeline` 为核心。测试在其中演示 常量、tt.func、lane/block 索引范围、arith.remsi、形状扩展，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 313-330
```mlir
313|     %6 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
314|     %7 = tt.expand_dims %6 {axis = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x64xi32, #blocked>
315|     %8 = tt.broadcast %5 : tensor<128x1xi32, #blocked> -> tensor<128x64xi32, #blocked>
316|     %9 = tt.broadcast %7 : tensor<1x64xi32, #blocked> -> tensor<128x64xi32, #blocked>
317|     %10 = arith.addi %8, %9 : tensor<128x64xi32, #blocked>
318|     %11 = tt.splat %arg0 : !tt.ptr<f8E5M2> -> tensor<128x64x!tt.ptr<f8E5M2>, #blocked>
319|     %12 = tt.addptr %11, %10 : tensor<128x64x!tt.ptr<f8E5M2>, #blocked>, tensor<128x64xi32, #blocked>
320|     %13 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 1, parent = #blocked1}>>
321|     %14 = tt.expand_dims %13 {axis = 1 : i32} : tensor<64xi32, #ttg.slice<{dim = 1, parent = #blocked1}>> -> tensor<64x1xi32, #blocked1>
322|     %15 = arith.muli %14, %cst_4 : tensor<64x1xi32, #blocked1>
323|     %16 = tt.expand_dims %3 {axis = 0 : i32} : tensor<256xi32, #ttg.slice<{dim = 0, parent = #blocked1}>> -> tensor<1x256xi32, #blocked1>
324|     %17 = tt.broadcast %15 : tensor<64x1xi32, #blocked1> -> tensor<64x256xi32, #blocked1>
325|     %18 = tt.broadcast %16 : tensor<1x256xi32, #blocked1> -> tensor<64x256xi32, #blocked1>
326|     %19 = arith.addi %17, %18 : tensor<64x256xi32, #blocked1>
327|     %20 = tt.splat %arg1 : !tt.ptr<f8E5M2> -> tensor<64x256x!tt.ptr<f8E5M2>, #blocked1>
328|     %21 = tt.addptr %20, %19 : tensor<64x256x!tt.ptr<f8E5M2>, #blocked1>, tensor<64x256xi32, #blocked1>
329|     %22:3 = scf.for %arg3 = %c0_i32 to %c8_i32 step %c1_i32 iter_args(%arg4 = %cst, %arg5 = %12, %arg6 = %21) -> (tensor<128x256xf32, #mma>, tensor<128x64x!tt.ptr<f8E5M2>, #blocked>, tensor<64x256x!tt.ptr<f8E5M2>, #blocked1>)  : i32 {
330|       %35 = tt.load %arg5 : tensor<128x64x!tt.ptr<f8E5M2>, #blocked>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining tensor broadcasting, shape expansion, lane/block index ranges, integer additions, broadcasted scalars or pointers.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 张量广播、形状扩展、lane/block 索引范围、整数加法、广播后的标量或指针。

### Lines 331-333
```mlir
331|       %36 = tt.load %arg6 : tensor<64x256x!tt.ptr<f8E5M2>, #blocked1>
332|       %37 = ttg.local_alloc %35 : (tensor<128x64xf8E5M2, #blocked>) -> !ttg.memdesc<128x64xf8E5M2, #shared, #smem>
333|       %38 = ttg.local_alloc %36 : (tensor<64x256xf8E5M2, #blocked1>) -> !ttg.memdesc<64x256xf8E5M2, #shared1, #smem>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, masked or vectorized loads. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、带掩码或向量化的加载。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 334-351
```mlir
334|       // CHECK: ttg.local_alloc
335|       // CHECK: scf.for
336|       // CHECK:   ttng.warp_group_dot
337|       // CHECK-NEXT: ttng.warp_group_dot_wait
338|       %39 = ttng.warp_group_dot %37, %38, %arg4 {maxNumImpreciseAcc = 1073741824 : i32} : !ttg.memdesc<128x64xf8E5M2, #shared, #smem> * !ttg.memdesc<64x256xf8E5M2, #shared1, #smem> -> tensor<128x256xf32, #mma>
339|       %40 = tt.addptr %arg5, %cst_6 : tensor<128x64x!tt.ptr<f8E5M2>, #blocked>, tensor<128x64xi32, #blocked>
340|       %41 = tt.addptr %arg6, %cst_5 : tensor<64x256x!tt.ptr<f8E5M2>, #blocked1>, tensor<64x256xi32, #blocked1>
341|       scf.yield %39, %40, %41 : tensor<128x256xf32, #mma>, tensor<128x64x!tt.ptr<f8E5M2>, #blocked>, tensor<64x256x!tt.ptr<f8E5M2>, #blocked1>
342|     }
343|     %23 = arith.truncf %22#0 : tensor<128x256xf32, #mma> to tensor<128x256xf16, #mma>
344|     %24 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked1}>>
345|     %25 = tt.expand_dims %24 {axis = 1 : i32} : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked1}>> -> tensor<128x1xi32, #blocked1>
346|     %26 = arith.muli %25, %cst_3 : tensor<128x1xi32, #blocked1>
347|     %27 = tt.splat %arg2 : !tt.ptr<f8E5M2> -> tensor<128x1x!tt.ptr<f8E5M2>, #blocked1>
348|     %28 = tt.addptr %27, %26 : tensor<128x1x!tt.ptr<f8E5M2>, #blocked1>, tensor<128x1xi32, #blocked1>
349|     %29 = tt.expand_dims %2 {axis = 0 : i32} : tensor<256xi32, #ttg.slice<{dim = 0, parent = #blocked1}>> -> tensor<1x256xi32, #blocked1>
350|     %30 = tt.broadcast %28 : tensor<128x1x!tt.ptr<f8E5M2>, #blocked1> -> tensor<128x256x!tt.ptr<f8E5M2>, #blocked1>
351|     %31 = tt.broadcast %29 : tensor<1x256xi32, #blocked1> -> tensor<128x256xi32, #blocked1>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining pointer arithmetic, shape expansion, tensor broadcasting, ttng.warp_group_dot, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 指针算术、形状扩展、张量广播、ttng.warp_group_dot、循环/分支产出值。

### Lines 352-359
```mlir
352|     %32 = tt.addptr %30, %31 : tensor<128x256x!tt.ptr<f8E5M2>, #blocked1>, tensor<128x256xi32, #blocked1>
353|     %33 = tt.fp_to_fp %23 {rounding = 1 : i32} : tensor<128x256xf16, #mma> -> tensor<128x256xf8E5M2, #mma>
354|     %34 = ttg.convert_layout %33 : tensor<128x256xf8E5M2, #mma> -> tensor<128x256xf8E5M2, #blocked1>
355|     tt.store %32, %34 : tensor<128x256x!tt.ptr<f8E5M2>, #blocked1>
356|     tt.return
357|   }
358| }
359| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on pointer arithmetic, tt.fp_to_fp, layout conversions, masked or vectorized stores, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 指针算术、tt.fp_to_fp、布局转换、带掩码或向量化的存储、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 360-360
```mlir
360| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 361-362
```mlir
361| 
362| // A dot can be properly async if all its uses follow a synchronous MMAv3 dot.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 363-369
```mlir
363| #blocked = #ttg.blocked<{sizePerThread = [8, 1], threadsPerWarp = [8, 4], warpsPerCTA = [1, 4], order = [0, 1]}>
364| #blocked1 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
365| #mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 64, 16]}>
366| #mma1 = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 16, 16]}>
367| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
368| #shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = true, elementBitWidth = 16}>
369| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 370-370
```mlir
370| module attributes {"ttg.target" = "cuda:90", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 371-371
```mlir
371| // CHECK-LABEL: async_following_sync
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: async_following_sync anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: async_following_sync 这样的标签用于锚定匹配范围。

### Lines 372-389
```mlir
372|   tt.func @async_following_sync(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32}) -> (tensor<128x64xf32, #mma>, tensor<128x16xf32, #mma1>) {
373|     %cst = arith.constant dense<64> : tensor<64x16xi32, #blocked>
374|     %c0_i32 = arith.constant 0 : i32
375|     %cst_0 = arith.constant dense<0> : tensor<1x16xi32, #blocked>
376|     %cst_1 = arith.constant dense<0> : tensor<128x1xi32, #blocked1>
377|     %c0_i64 = arith.constant 0 : i64
378|     %cst_2 = arith.constant dense<0.000000e+00> : tensor<128x16xf32, #mma1>
379|     %cst_3 = arith.constant dense<0.000000e+00> : tensor<128x64xf32, #mma>
380|     %cst_4 = arith.constant dense<1.000000e+00> : tensor<128x16xf16, #ttg.dot_op<{opIdx = 0, parent = #mma1, kWidth = 2}>>
381|     %c1_i32 = arith.constant 1 : i32
382|     %c8_i32 = arith.constant 8 : i32
383| 
384|     // Add a "dummy" early return here to test that we don't crash in the
385|     // presence of unstructured control flow.
386|     %cond = arith.constant 0 : i1
387|     cf.cond_br %cond, ^bb1, ^bb2
388|   ^bb1:  // pred: ^bb0
389|     %zero = arith.constant 0.0 : f32
```
**EN:** This function-oriented block defines or enters `async_following_sync`. Within it, the test exercises constants, tt.func, cf.cond_br, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_following_sync` 为核心。测试在其中演示 常量、tt.func、cf.cond_br，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 390-407
```mlir
390|     %t1 = tt.splat %zero : f32 -> tensor<128x64xf32, #mma>
391|     %t2 = tt.splat %zero : f32 -> tensor<128x16xf32, #mma1>
392|     tt.return %t1, %t2 : tensor<128x64xf32, #mma>, tensor<128x16xf32, #mma1>
393|   ^bb2:  // pred: ^bb0
394| 
395|     %0 = tt.addptr %arg0, %c0_i64 : !tt.ptr<f16>, i64
396|     %1 = tt.addptr %arg1, %c0_i64 : !tt.ptr<f16>, i64
397|     %2 = tt.splat %1 : !tt.ptr<f16> -> tensor<128x1x!tt.ptr<f16>, #blocked1>
398|     %3 = tt.addptr %2, %cst_1 : tensor<128x1x!tt.ptr<f16>, #blocked1>, tensor<128x1xi32, #blocked1>
399|     %4 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>
400|     %5 = tt.expand_dims %4 {axis = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked1}>> -> tensor<1x64xi32, #blocked1>
401|     %6 = tt.broadcast %3 : tensor<128x1x!tt.ptr<f16>, #blocked1> -> tensor<128x64x!tt.ptr<f16>, #blocked1>
402|     %7 = tt.broadcast %5 : tensor<1x64xi32, #blocked1> -> tensor<128x64xi32, #blocked1>
403|     %8 = tt.addptr %6, %7 : tensor<128x64x!tt.ptr<f16>, #blocked1>, tensor<128x64xi32, #blocked1>
404|     %9 = tt.load %8 : tensor<128x64x!tt.ptr<f16>, #blocked1>
405|     %10 = tt.splat %0 : !tt.ptr<f16> -> tensor<1x16x!tt.ptr<f16>, #blocked>
406|     %11 = tt.addptr %10, %cst_0 : tensor<1x16x!tt.ptr<f16>, #blocked>, tensor<1x16xi32, #blocked>
407|     %12 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on pointer arithmetic, broadcasted scalars or pointers, lane/block index ranges, tensor broadcasting, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 指针算术、广播后的标量或指针、lane/block 索引范围、张量广播、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 408-414
```mlir
408|     %13 = tt.expand_dims %12 {axis = 1 : i32} : tensor<64xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<64x1xi32, #blocked>
409|     %14 = tt.broadcast %11 : tensor<1x16x!tt.ptr<f16>, #blocked> -> tensor<64x16x!tt.ptr<f16>, #blocked>
410|     %15 = tt.broadcast %13 : tensor<64x1xi32, #blocked> -> tensor<64x16xi32, #blocked>
411|     %16 = tt.addptr %14, %15 : tensor<64x16x!tt.ptr<f16>, #blocked>, tensor<64x16xi32, #blocked>
412|     %18 = tt.load %16 : tensor<64x16x!tt.ptr<f16>, #blocked>
413|     %19 = ttg.local_alloc %9 : (tensor<128x64xf16, #blocked1>) -> !ttg.memdesc<128x64xf16, #shared, #smem>
414|     %20 = ttg.local_alloc %18 : (tensor<64x16xf16, #blocked>) -> !ttg.memdesc<64x16xf16, #shared1, #smem>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tensor broadcasting, shared/local memory allocation, shape expansion, pointer arithmetic, masked or vectorized loads. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 张量广播、共享/本地内存分配、形状扩展、指针算术、带掩码或向量化的加载。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 415-432
```mlir
415|     // CHECK:          %[[LOOP:[^ :]+]]{{.*}} scf.for {{.*}} iter_args(%[[PREV_DOT2:[^ ]+]]
416|     // CHECK-NOT:        ttng.warp_group_dot_wait
417|     // CHECK:            %[[DOT0:.+]] = ttng.warp_group_dot
418|     // CHECK-NOT:        ttng.warp_group_dot_wait
419|     // CHECK:            %[[DOT1:.+]] = ttng.warp_group_dot
420|     // CHECK-NEXT:       ttng.warp_group_dot_wait
421|     // CHECK-DAG-SAME:     %[[DOT0]]
422|     // CHECK-DAG-SAME:     %[[DOT1]]
423|     // CHECK-DAG-SAME:     %[[PREV_DOT2]]
424|     // CHECK-SAME:         {pendings = 0 : i32}
425|     // CHECK:            %[[DOT2:.+]] = ttng.warp_group_dot
426|     // CHECK-NOT:        ttng.warp_group_dot_wait
427|     // CHECK:          scf.yield %[[DOT2]]
428|     // CHECK:          ttng.warp_group_dot_wait %[[LOOP]]#3, %[[LOOP]]#0 {pendings = 0 : i32}
429|     %17:4 = scf.for %arg3 = %c0_i32 to %c8_i32 step %c1_i32 iter_args(%prev_dot2 = %cst_3, %arg5 = %16, %prev_dot1 = %cst_2, %prev_dot0 = %cst_2) -> (tensor<128x64xf32, #mma>, tensor<64x16x!tt.ptr<f16>, #blocked>, tensor<128x16xf32, #mma1>, tensor<128x16xf32, #mma1>)  : i32 {
430|       // This one can be async.
431|       %dot0 = ttng.warp_group_dot %19, %20, %prev_dot1 : !ttg.memdesc<128x64xf16, #shared, #smem> * !ttg.memdesc<64x16xf16, #shared1, #smem> -> tensor<128x16xf32, #mma1>
432|       // This can't be async because its result is modified before it's yielded.
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops, ttng.warp_group_dot.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环、ttng.warp_group_dot。

### Lines 433-448
```mlir
433|       %dot1 = ttng.warp_group_dot %19, %20, %prev_dot1 : !ttg.memdesc<128x64xf16, #shared, #smem> * !ttg.memdesc<64x16xf16, #shared1, #smem> -> tensor<128x16xf32, #mma1>
434|       %dot1.1 = arith.addf %dot1, %dot1 : tensor<128x16xf32, #mma1>
435|       %l = tt.load %arg5 : tensor<64x16x!tt.ptr<f16>, #blocked>
436|       %c = ttg.local_alloc %l : (tensor<64x16xf16, #blocked>) -> !ttg.memdesc<64x16xf16, #shared1, #smem>
437|       %23 = ttg.memdesc_trans %c {order=array<i32: 1,0>} : !ttg.memdesc<64x16xf16, #shared1, #smem> -> !ttg.memdesc<16x64xf16, #shared, #smem>
438|       // This dot can be async even though %prev_dot2 is not used directly by an
439|       // async dot, because that use follows the synchronous dot above.
440|       %prev_dot2.1 = arith.addf %prev_dot2, %prev_dot2 : tensor<128x64xf32, #mma>
441|       %dot2 = ttng.warp_group_dot %cst_4, %23, %prev_dot2.1 : tensor<128x16xf16, #ttg.dot_op<{opIdx = 0, parent = #mma1, kWidth = 2}>> * !ttg.memdesc<16x64xf16, #shared, #smem> -> tensor<128x64xf32, #mma>
442|       %26 = tt.addptr %arg5, %cst : tensor<64x16x!tt.ptr<f16>, #blocked>, tensor<64x16xi32, #blocked>
443|       scf.yield %dot2, %26, %dot1.1, %dot0 : tensor<128x64xf32, #mma>, tensor<64x16x!tt.ptr<f16>, #blocked>, tensor<128x16xf32, #mma1>, tensor<128x16xf32, #mma1>
444|     }
445|     tt.return %17#0, %17#2 : tensor<128x64xf32, #mma>, tensor<128x16xf32, #mma1>
446|   }
447| }
448| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining ttng.warp_group_dot, floating-point additions, masked or vectorized loads, shared/local memory allocation, ttg.memdesc_trans.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 ttng.warp_group_dot、浮点加法、带掩码或向量化的加载、共享/本地内存分配、ttg.memdesc_trans。

### Lines 449-449
```mlir
449| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 450-450
```mlir
450| // Test pipelining of descriptor_store
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 451-452
```mlir
451| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [1, 0]}>
452| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = false, elementBitWidth = 8}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 453-453
```mlir
453| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 454-455
```mlir
454|   // CHECK: #[[$SHARED:.+]] = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = false, elementBitWidth = 8}>
455|   // CHECK-LABEL: tma_store_pipeline
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: tma_store_pipeline anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: tma_store_pipeline 这样的标签用于锚定匹配范围。

### Lines 456-457
```mlir
456|   tt.func public @tma_store_pipeline(%arg0: tensor<128x128xf32, #blocked>, %arg1: !tt.tensordesc<128x128xf32, #shared>, %arg2: i32, %arg3: i32) {
457|     %c0_i32 = arith.constant 0 : i32
```
**EN:** This function-oriented block defines or enters `tma_store_pipeline`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tma_store_pipeline` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 458-461
```mlir
458|     // CHECK: ttg.local_alloc : () -> !ttg.memdesc<128x128xf32, #[[$SHARED]], #smem, mutable>
459|     // CHECK: scf.for
460|     scf.for %arg4 = %c0_i32 to %arg3 step %arg2  : i32 {
461|       %1 = arith.divsi %arg4, %arg2 : i32
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops, arith.divsi.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环、arith.divsi。

### Lines 462-471
```mlir
462|       // CHECK: ttng.async_tma_store_wait {pendings = 0 : i32}
463|       // CHECK-NEXT: ttg.local_store
464|       // CHECK-NEXT: ttng.fence_async_shared
465|       // CHECK-NEXT: ttng.async_tma_copy_local_to_global
466|       tt.descriptor_store %arg1[%1, %1], %arg0 : !tt.tensordesc<128x128xf32, #shared>, tensor<128x128xf32, #blocked>
467|     }
468|     tt.return
469|   }
470| }
471| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.descriptor_store, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.descriptor_store、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 472-472
```mlir
472| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 473-477
```mlir
473| 
474| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
475| #blocked1_parent = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [32, 1], warpsPerCTA = [1, 4], order = [1, 0]}>
476| #blocked1 = #ttg.slice<{dim = 0, parent = #blocked1_parent}>
477| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 478-478
```mlir
478| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 479-479
```mlir
479|   // CHECK-LABEL: tma_scatter_pipeline
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: tma_scatter_pipeline anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: tma_scatter_pipeline 这样的标签用于锚定匹配范围。

### Lines 480-484
```mlir
480|   tt.func public @tma_scatter_pipeline(%arg0: tensor<8x128xf32, #blocked>, %arg1: !tt.tensordesc<1x128xf32, #shared>, %arg2: i32, %arg3: i32) {
481|     %c0_i32 = arith.constant 0 : i32
482|     scf.for %arg4 = %c0_i32 to %arg3 step %arg2  : i32 {
483|       %1 = arith.divsi %arg4, %arg2 : i32
484|       %2 = tt.splat %1 : i32 -> tensor<8xi32, #blocked1>
```
**EN:** This function-oriented block defines or enters `tma_scatter_pipeline`. Within it, the test exercises tt.func, constants, structured loops, arith.divsi, broadcasted scalars or pointers, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tma_scatter_pipeline` 为核心。测试在其中演示 tt.func、常量、结构化循环、arith.divsi、广播后的标量或指针，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 485-494
```mlir
485|       // CHECK: ttng.async_tma_store_wait {pendings = 0 : i32}
486|       // CHECK-NEXT: ttg.local_store
487|       // CHECK-NEXT: ttng.fence_async_shared
488|       // CHECK-NEXT: ttng.async_tma_scatter
489|       tt.descriptor_scatter %arg1[%2, %1], %arg0 : !tt.tensordesc<1x128xf32, #shared>, tensor<8xi32, #blocked1>, i32, tensor<8x128xf32, #blocked>
490|     }
491|     tt.return
492|   }
493| }
494| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.descriptor_scatter, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.descriptor_scatter、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 495-495
```mlir
495| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 496-498
```mlir
496| 
497| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [1, 0]}>
498| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 0, transposed = false, elementBitWidth = 8}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 499-499
```mlir
499| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 500-500
```mlir
500|   // CHECK-LABEL: tma_store_device_side_desc_pipeline
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: tma_store_device_side_desc_pipeline anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: tma_store_device_side_desc_pipeline 这样的标签用于锚定匹配范围。

### Lines 501-505
```mlir
501|   tt.func public @tma_store_device_side_desc_pipeline(%arg0: tensor<128x128xf32, #blocked>, %arg1: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg2: i32, %arg3: i32) {
502|     %c0_i32 = arith.constant 0 : i32
503|     %c128_i32 = arith.constant 128 : i32
504|     %c128_i64 = arith.constant 128 : i64
505|     %c1_i64 = arith.constant 1 : i64
```
**EN:** This function-oriented block defines or enters `tma_store_device_side_desc_pipeline`. Within it, the test exercises constants, tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tma_store_device_side_desc_pipeline` 为核心。测试在其中演示 常量、tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 506-510
```mlir
506|     // CHECK: %[[A:.+]] = ttg.global_scratch_alloc {alignment = 128 : i32, nbytes = 256 : i32} : !tt.ptr<i8>
507|     // CHECK: scf.for
508|     scf.for %arg4 = %c0_i32 to %arg3 step %arg2  : i32 {
509|       %1 = arith.divsi %arg4, %arg2 : i32
510|       %desc = tt.make_tensor_descriptor %arg1, [%c128_i32, %c128_i32], [%c128_i64, %c1_i64] : <f32>, <128x128xf32, #shared>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops, arith.divsi, tt.make_tensor_descriptor.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环、arith.divsi、tt.make_tensor_descriptor。

### Lines 511-519
```mlir
511|       // CHECK: ttng.tensormap_create
512|       // CHECK: ttng.tensormap_fenceproxy_acquire
513|       // CHECK: ttng.async_tma_store_wait {pendings = 0 : i32}
514|       // CHECK-NEXT: ttg.local_store
515|       // CHECK-NEXT: ttng.fence_async_shared
516|       // CHECK-NEXT: ttng.async_tma_copy_local_to_global
517|       // CHECK: scf.yield
518|       tt.descriptor_store %desc[%c0_i32, %1], %arg0 : !tt.tensordesc<128x128xf32, #shared>, tensor<128x128xf32, #blocked>
519|     }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining tt.descriptor_store.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 tt.descriptor_store。

### Lines 520-523
```mlir
520|     // CHECK: ttng.async_tma_store_wait {pendings = 0 : i32}
521|     tt.return
522|   }
523| }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 524-524
```mlir
524| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 525-527
```mlir
525| #blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
526| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 0, transposed = false, elementBitWidth = 32, rank=1}>
527| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 528-528
```mlir
528| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 529-529
```mlir
529|   // CHECK-LABEL: tma_multiple_store_pipeline
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: tma_multiple_store_pipeline anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: tma_multiple_store_pipeline 这样的标签用于锚定匹配范围。

### Lines 530-531
```mlir
530|   tt.func public @tma_multiple_store_pipeline(%arg0: tensor<1xf32, #blocked>, %arg1: !tt.tensordesc<1xf32, #shared>, %arg2: i32, %arg3: i32) {
531|     %c0_i32 = arith.constant 0 : i32
```
**EN:** This function-oriented block defines or enters `tma_multiple_store_pipeline`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tma_multiple_store_pipeline` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 532-536
```mlir
532|     // CHECK: %[[ALLOC:.+]] = ttg.local_alloc : () -> !ttg.memdesc<1xf32, #shared, #smem, mutable>
533|     // CHECK: scf.for
534|     scf.for %arg4 = %c0_i32 to %arg3 step %arg2  : i32 {
535|       %1 = arith.divsi %arg4, %arg2 : i32
536|       %2 = arith.divsi %arg2, %arg4 : i32
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining arith.divsi, structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 arith.divsi、结构化循环。

### Lines 537-552
```mlir
537|       // CHECK: ttng.async_tma_store_wait {pendings = 0 : i32}
538|       // CHECK-NEXT: ttg.local_store %{{.+}}, %[[ALLOC]]
539|       // CHECK-NEXT: ttng.fence_async_shared
540|       // CHECK-NEXT: ttng.async_tma_copy_local_to_global %{{.*}} %[[ALLOC]]
541|       // CHECK: ttng.async_tma_store_wait {pendings = 0 : i32}
542|       // CHECK-NEXT: ttg.local_store %{{.+}}, %[[ALLOC]]
543|       // CHECK-NEXT: ttng.fence_async_shared
544|       // CHECK-NEXT: ttng.async_tma_copy_local_to_global %{{.*}} %[[ALLOC]]
545|       tt.descriptor_store %arg1[%1], %arg0 : !tt.tensordesc<1xf32, #shared>, tensor<1xf32, #blocked>
546|       tt.descriptor_store %arg1[%2], %arg0 : !tt.tensordesc<1xf32, #shared>, tensor<1xf32, #blocked>
547|     }
548|     tt.return
549|   }
550| }
551| 
552| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.descriptor_store, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.descriptor_store、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 553-553
```mlir
553| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 554-560
```mlir
554| 
555| #blocked = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [4, 8], warpsPerCTA = [8, 1], order = [1, 0]}>
556| #blocked1 = #ttg.blocked<{sizePerThread = [16, 1], threadsPerWarp = [8, 4], warpsPerCTA = [1, 8], order = [0, 1]}>
557| #mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [8, 1], instrShape = [16, 128, 32]}>
558| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 8}>
559| #shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = true, elementBitWidth = 8}>
560| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 561-561
```mlir
561| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 562-562
```mlir
562|   // CHECK-LABEL: _kernel_matmul_dependency
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: _kernel_matmul_dependency anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: _kernel_matmul_dependency 这样的标签用于锚定匹配范围。

### Lines 563-580
```mlir
563|   tt.func public @_kernel_matmul_dependency(%arg0: tensor<128x128x!tt.ptr<f8E4M3FN>, #blocked>, %arg1: !tt.ptr<f8E4M3FN> {tt.divisibility = 16 : i32}, %arg2: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg3: i32, %arg4: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg5: tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>) {
564|     %cst = arith.constant dense<0> : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>
565|     %cst_0 = arith.constant 1.000000e+00 : f32
566|     %c8_i32 = arith.constant 8 : i32
567|     %cst_1 = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #mma>
568|     %0 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked1}>>
569|     %1 = tt.splat %arg1 : !tt.ptr<f8E4M3FN> -> tensor<128x128x!tt.ptr<f8E4M3FN>, #blocked1>
570|     %2:4 = scf.for %arg6 = %c8_i32 to %arg3 step %c8_i32 iter_args(%arg7 = %c8_i32, %arg8 = %c8_i32, %arg9 = %cst_1, %arg10 = %arg5) -> (i32, i32, tensor<128x128xf32, #mma>, tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>)  : i32 {
571|       %3 = arith.addi %arg7, %c8_i32 : i32
572|       %4 = arith.cmpi eq, %3, %c8_i32 : i32
573|       %5:2 = scf.if %4 -> (i32, tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>) {
574|         %21 = arith.addi %arg8, %c8_i32 : i32
575|         scf.yield %21, %arg5 : i32, tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>
576|       } else {
577|         scf.yield %arg8, %arg10 : i32, tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>
578|       }
579|       %6 = arith.cmpi eq, %3, %c8_i32 : i32
580|       %7 = scf.if %6 -> (f32) {
```
**EN:** This function-oriented block defines or enters `_kernel_matmul_dependency`. Within it, the test exercises constants, tt.func, integer additions, integer comparisons, structured conditionals, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `_kernel_matmul_dependency` 为核心。测试在其中演示 常量、tt.func、整数加法、整数比较、结构化条件分支，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 581-598
```mlir
581|         scf.yield %cst_0 : f32
582|       } else {
583|         %21 = tt.load %arg4 : !tt.ptr<f32>
584|         scf.yield %21 : f32
585|       }
586|       %8 = tt.splat %3 : i32 -> tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked1}>>
587|       %9 = arith.addi %8, %0 : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked1}>>
588|       %10 = tt.expand_dims %9 {axis = 1 : i32} : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked1}>> -> tensor<128x1xi32, #blocked1>
589|       %11 = tt.broadcast %10 : tensor<128x1xi32, #blocked1> -> tensor<128x128xi32, #blocked1>
590|       %12 = tt.addptr %1, %11 : tensor<128x128x!tt.ptr<f8E4M3FN>, #blocked1>, tensor<128x128xi32, #blocked1>
591|       %13 = tt.load %arg0 : tensor<128x128x!tt.ptr<f8E4M3FN>, #blocked>
592|       %14 = ttg.local_alloc %13 : (tensor<128x128xf8E4M3FN, #blocked>) -> !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem>
593|       %15 = tt.load %12 : tensor<128x128x!tt.ptr<f8E4M3FN>, #blocked1>
594|       %16 = ttg.local_alloc %15 : (tensor<128x128xf8E4M3FN, #blocked1>) -> !ttg.memdesc<128x128xf8E4M3FN, #shared1, #smem>
595|       %17 = ttng.warp_group_dot %14, %16, %arg9 {inputPrecision = 0 : i32, maxNumImpreciseAcc = 1073741824 : i32} : !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem> * !ttg.memdesc<128x128xf8E4M3FN, #shared1, #smem> -> tensor<128x128xf32, #mma>
596|       %18 = tt.splat %7 : f32 -> tensor<128x128xf32, #mma>
597|       %19 = arith.mulf %17, %18 : tensor<128x128xf32, #mma>
598|       %20 = scf.if %6 -> (tensor<128x128xf32, #mma>) {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining masked or vectorized loads, loop/if yielded values, broadcasted scalars or pointers, shared/local memory allocation, integer additions.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 带掩码或向量化的加载、循环/分支产出值、广播后的标量或指针、共享/本地内存分配、整数加法。

### Lines 599-608
```mlir
599|         scf.yield %cst_1 : tensor<128x128xf32, #mma>
600|       } else {
601|         scf.yield %19 : tensor<128x128xf32, #mma>
602|       }
603|       scf.yield %3, %5#0, %20, %5#1 : i32, i32, tensor<128x128xf32, #mma>, tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>
604|     }
605|     tt.return
606|   }
607| }
608| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、tt.return。

### Lines 609-609
```mlir
609| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 610-611
```mlir
610| 
611| // Pipeline the if ops at the beginning and the end of the loop
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 612-618
```mlir
612| #blocked = #ttg.blocked<{sizePerThread = [8, 1], threadsPerWarp = [8, 4], warpsPerCTA = [1, 4], order = [0, 1]}>
613| #blocked1 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
614| #mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 64, 16]}>
615| #mma1 = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 16, 16]}>
616| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
617| #shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = true, elementBitWidth = 16}>
618| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 619-619
```mlir
619| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 620-621
```mlir
620|   // CHECK-LABEL: dot_prologue_epilogue
621|   // CHECK: {{.*}}, {{.*}}, %[[EXT:.*]]: i32, {{.*}}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: dot_prologue_epilogue anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: dot_prologue_epilogue 这样的标签用于锚定匹配范围。

### Lines 622-639
```mlir
622|   tt.func @dot_prologue_epilogue(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %ext: i32, %inc: tensor<64x16xi32, #blocked> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>}) -> tensor<128x16xf32, #mma1> {
623|     %cst = arith.constant dense<0> : tensor<64x16xi32, #blocked>
624|     %cst2 = arith.constant dense<0> : tensor<128x64xi32, #blocked1>
625|     %c0_i32 = arith.constant 0 : i32
626|     %cst_0 = arith.constant dense<0> : tensor<1x16xi32, #blocked>
627|     %cst_1 = arith.constant dense<0> : tensor<128x1xi32, #blocked1>
628|     %c0_i64 = arith.constant 0 : i64
629|     %cst_2 = arith.constant dense<0.000000e+00> : tensor<128x16xf32, #mma1>
630|     %c1_i32 = arith.constant 1 : i32
631|     %c8_i32 = arith.constant 8 : i32
632|     %2 = tt.splat %arg1 : !tt.ptr<f16> -> tensor<128x1x!tt.ptr<f16>, #blocked1>
633|     %4 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>
634|     %5 = tt.expand_dims %4 {axis = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked1}>> -> tensor<1x64xi32, #blocked1>
635|     %6 = tt.broadcast %2 : tensor<128x1x!tt.ptr<f16>, #blocked1> -> tensor<128x64x!tt.ptr<f16>, #blocked1>
636|     %7 = tt.broadcast %5 : tensor<1x64xi32, #blocked1> -> tensor<128x64xi32, #blocked1>
637|     %8 = tt.addptr %6, %7 : tensor<128x64x!tt.ptr<f16>, #blocked1>, tensor<128x64xi32, #blocked1>
638|     %10 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<1x16x!tt.ptr<f16>, #blocked>
639|     %12 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
```
**EN:** This function-oriented block defines or enters `dot_prologue_epilogue`. Within it, the test exercises constants, tt.func, broadcasted scalars or pointers, lane/block index ranges, tensor broadcasting, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `dot_prologue_epilogue` 为核心。测试在其中演示 常量、tt.func、广播后的标量或指针、lane/block 索引范围、张量广播，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 640-643
```mlir
640|     %13 = tt.expand_dims %12 {axis = 1 : i32} : tensor<64xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<64x1xi32, #blocked>
641|     %14 = tt.broadcast %10 : tensor<1x16x!tt.ptr<f16>, #blocked> -> tensor<64x16x!tt.ptr<f16>, #blocked>
642|     %15 = tt.broadcast %13 : tensor<64x1xi32, #blocked> -> tensor<64x16xi32, #blocked>
643|     %16 = tt.addptr %14, %15 : tensor<64x16x!tt.ptr<f16>, #blocked>, tensor<64x16xi32, #blocked>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tensor broadcasting, shape expansion, pointer arithmetic. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 张量广播、形状扩展、指针算术。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 644-661
```mlir
644|     // CHECK: %[[C0:.*]] = arith.constant 0 : i32
645|     // CHECK: scf.for %[[IND_VAR:.*]] = %[[C0]]
646|     // CHECK-NOT: load
647|     // CHECK: %[[CND:.*]] = arith.cmpi slt, %[[IND_VAR]], %[[EXT]]
648|     // CHECK: scf.if %[[CND]]
649|     // CHECK: dot
650|     // CHECK: scf.if %[[CND]]
651|     // CHECK:   arith.mulf
652|     // CHECK:   scf.yield
653|     // CHECK-NOT: tt.addptr
654|     // CHECK: scf.yield
655|     %17:3 = scf.for %arg3 = %c0_i32 to %c8_i32 step %c1_i32 iter_args(%arg4 = %cst_2, %arg5 = %16, %arg6 = %8) -> (tensor<128x16xf32, #mma1>, tensor<64x16x!tt.ptr<f16>, #blocked>, tensor<128x64x!tt.ptr<f16>, #blocked1>)  : i32 {
656|       %9 = tt.load %arg6 : tensor<128x64x!tt.ptr<f16>, #blocked1>
657|       %cnd = arith.cmpi slt, %arg3, %ext : i32
658|       %inc_ptr = scf.if %cnd -> tensor<64x16x!tt.ptr<f16>, #blocked> {
659|         %ptr = tt.addptr %arg5, %inc : tensor<64x16x!tt.ptr<f16>, #blocked>, tensor<64x16xi32, #blocked>
660|         scf.yield %ptr : tensor<64x16x!tt.ptr<f16>, #blocked>
661|       } else {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops, masked or vectorized loads, integer comparisons, structured conditionals, pointer arithmetic.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环、带掩码或向量化的加载、整数比较、结构化条件分支、指针算术。

### Lines 662-679
```mlir
662|         scf.yield %arg5 : tensor<64x16x!tt.ptr<f16>, #blocked>
663|       }
664|       %18 = tt.load %inc_ptr : tensor<64x16x!tt.ptr<f16>, #blocked>
665|       %19 = ttg.local_alloc %9 : (tensor<128x64xf16, #blocked1>) -> !ttg.memdesc<128x64xf16, #shared, #smem>
666|       %20 = ttg.local_alloc %18 : (tensor<64x16xf16, #blocked>) -> !ttg.memdesc<64x16xf16, #shared1, #smem>
667|       %acc = ttng.warp_group_dot %19, %20, %arg4 : !ttg.memdesc<128x64xf16, #shared, #smem> * !ttg.memdesc<64x16xf16, #shared1, #smem> -> tensor<128x16xf32, #mma1>
668|       %acc_ = scf.if %cnd -> (tensor<128x16xf32, #mma1>) {
669|         %acc_zero = arith.mulf %acc, %cst_2 : tensor<128x16xf32, #mma1>
670|         scf.yield %acc_zero : tensor<128x16xf32, #mma1>
671|       } else {
672|         scf.yield %acc : tensor<128x16xf32, #mma1>
673|       }
674|       %22 = tt.addptr %arg5, %cst : tensor<64x16x!tt.ptr<f16>, #blocked>, tensor<64x16xi32, #blocked>
675|       %23 = tt.addptr %arg6, %cst2 : tensor<128x64x!tt.ptr<f16>, #blocked1>, tensor<128x64xi32, #blocked1>
676|       scf.yield %acc_, %22, %23 : tensor<128x16xf32, #mma1>, tensor<64x16x!tt.ptr<f16>, #blocked>, tensor<128x64x!tt.ptr<f16>, #blocked1>
677|     }
678|     tt.return %17#0 : tensor<128x16xf32, #mma1>
679|   }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, shared/local memory allocation, pointer arithmetic, masked or vectorized loads, ttng.warp_group_dot.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、共享/本地内存分配、指针算术、带掩码或向量化的加载、ttng.warp_group_dot。

### Lines 680-681
```mlir
680| }
681| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 682-682
```mlir
682| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 683-684
```mlir
683| 
684| // Verify that uses of the ops scheduled in partucular place of the loop (like epilogue if) are correctly scheduled too.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 685-691
```mlir
685| #blocked = #ttg.blocked<{sizePerThread = [8, 1], threadsPerWarp = [8, 4], warpsPerCTA = [1, 4], order = [0, 1]}>
686| #blocked1 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
687| #mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 64, 16]}>
688| #mma1 = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 16, 16]}>
689| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
690| #shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = true, elementBitWidth = 16}>
691| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 692-692
```mlir
692| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 693-694
```mlir
693|   // CHECK-NOCANON-LABEL: pipeline_downstream_dependencies
694|   // CHECK-NOCANON: {{.*}}, {{.*}}, %[[EXT:.*]]: i32, {{.*}}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 695-712
```mlir
695|   tt.func @pipeline_downstream_dependencies(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %ext: i32, %inc: tensor<64x16xi32, #blocked> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>}) -> tensor<128x16xf32, #mma1> {
696|     %cst = arith.constant dense<0> : tensor<64x16xi32, #blocked>
697|     %cst1 = arith.constant dense<1> : tensor<64x16xi32, #blocked>
698|     %cst2 = arith.constant dense<0> : tensor<128x64xi32, #blocked1>
699|     %c0_i32 = arith.constant 0 : i32
700|     %cst_0 = arith.constant dense<0> : tensor<1x16xi32, #blocked>
701|     %cst_1 = arith.constant dense<0> : tensor<128x1xi32, #blocked1>
702|     %c0_i64 = arith.constant 0 : i64
703|     %cst_2 = arith.constant dense<0.000000e+00> : tensor<128x16xf32, #mma1>
704|     %c1_i32 = arith.constant 1 : i32
705|     %c8_i32 = arith.constant 8 : i32
706|     %2 = tt.splat %arg1 : !tt.ptr<f16> -> tensor<128x1x!tt.ptr<f16>, #blocked1>
707|     %4 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>
708|     %5 = tt.expand_dims %4 {axis = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked1}>> -> tensor<1x64xi32, #blocked1>
709|     %6 = tt.broadcast %2 : tensor<128x1x!tt.ptr<f16>, #blocked1> -> tensor<128x64x!tt.ptr<f16>, #blocked1>
710|     %7 = tt.broadcast %5 : tensor<1x64xi32, #blocked1> -> tensor<128x64xi32, #blocked1>
711|     %8 = tt.addptr %6, %7 : tensor<128x64x!tt.ptr<f16>, #blocked1>, tensor<128x64xi32, #blocked1>
712|     %10 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<1x16x!tt.ptr<f16>, #blocked>
```
**EN:** This function-oriented block defines or enters `pipeline_downstream_dependencies`. Within it, the test exercises constants, tt.func, broadcasted scalars or pointers, tensor broadcasting, lane/block index ranges, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `pipeline_downstream_dependencies` 为核心。测试在其中演示 常量、tt.func、广播后的标量或指针、张量广播、lane/block 索引范围，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 713-717
```mlir
713|     %12 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
714|     %13 = tt.expand_dims %12 {axis = 1 : i32} : tensor<64xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<64x1xi32, #blocked>
715|     %14 = tt.broadcast %10 : tensor<1x16x!tt.ptr<f16>, #blocked> -> tensor<64x16x!tt.ptr<f16>, #blocked>
716|     %15 = tt.broadcast %13 : tensor<64x1xi32, #blocked> -> tensor<64x16xi32, #blocked>
717|     %16 = tt.addptr %14, %15 : tensor<64x16x!tt.ptr<f16>, #blocked>, tensor<64x16xi32, #blocked>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tensor broadcasting, lane/block index ranges, shape expansion, pointer arithmetic. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 张量广播、lane/block 索引范围、形状扩展、指针算术。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 718-735
```mlir
718|     // CHECK-NOCANON: %[[C0:.*]] = arith.constant 0 : i32
719|     // CHECK-NOCANON: scf.for %[[IND_VAR:.*]] = %[[C0]]
720|     // CHECK-NOCANON-NOT load
721|     // CHECK-NOCANON: dot
722|     // CHECK-NOCANON: %[[CND:.*]] = arith.cmpi slt, %[[IND_VAR]], %[[EXT]]
723|     // CHECK-NOCANON: %[[IFRET:.*]]:2 = scf.if %[[CND]]
724|     // CHECK-NOCANON:   arith.mulf
725|     // CHECK-NOCANON:   scf.yield
726|     // CHECK-NOCANON: tt.addptr {{.*}}, %[[IFRET]]#1
727|     // CHECK-NOCANON: scf.yield
728|     %17:3 = scf.for %arg3 = %c0_i32 to %c8_i32 step %c1_i32 iter_args(%arg4 = %cst_2, %arg5 = %16, %arg6 = %8) -> (tensor<128x16xf32, #mma1>, tensor<64x16x!tt.ptr<f16>, #blocked>, tensor<128x64x!tt.ptr<f16>, #blocked1>)  : i32 {
729|       %9 = tt.load %arg6 : tensor<128x64x!tt.ptr<f16>, #blocked1>
730|       %18 = tt.load %arg5 : tensor<64x16x!tt.ptr<f16>, #blocked>
731|       %19 = ttg.local_alloc %9 : (tensor<128x64xf16, #blocked1>) -> !ttg.memdesc<128x64xf16, #shared, #smem>
732|       %20 = ttg.local_alloc %18 : (tensor<64x16xf16, #blocked>) -> !ttg.memdesc<64x16xf16, #shared1, #smem>
733|       %acc = ttng.warp_group_dot %19, %20, %arg4 : !ttg.memdesc<128x64xf16, #shared, #smem> * !ttg.memdesc<64x16xf16, #shared1, #smem> -> tensor<128x16xf32, #mma1>
734|       %cnd = arith.cmpi slt, %arg3, %ext : i32
735|       %if_ret:2 = scf.if %cnd -> (tensor<128x16xf32, #mma1>, tensor<64x16xi32, #blocked>) {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining masked or vectorized loads, shared/local memory allocation, structured loops, ttng.warp_group_dot, integer comparisons.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 带掩码或向量化的加载、共享/本地内存分配、结构化循环、ttng.warp_group_dot、整数比较。

### Lines 736-748
```mlir
736|         %acc_zero = arith.mulf %acc, %cst_2 : tensor<128x16xf32, #mma1>
737|         scf.yield %acc_zero, %cst : tensor<128x16xf32, #mma1>, tensor<64x16xi32, #blocked>
738|       } else {
739|         scf.yield %acc, %cst1 : tensor<128x16xf32, #mma1>, tensor<64x16xi32, #blocked>
740|       }
741|       %22 = tt.addptr %arg5, %if_ret#1 : tensor<64x16x!tt.ptr<f16>, #blocked>, tensor<64x16xi32, #blocked>
742|       %23 = tt.addptr %arg6, %cst2 : tensor<128x64x!tt.ptr<f16>, #blocked1>, tensor<128x64xi32, #blocked1>
743|       scf.yield %if_ret#0, %22, %23 : tensor<128x16xf32, #mma1>, tensor<64x16x!tt.ptr<f16>, #blocked>, tensor<128x64x!tt.ptr<f16>, #blocked1>
744|     }
745|     tt.return %17#0 : tensor<128x16xf32, #mma1>
746|   }
747| }
748| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, pointer arithmetic, arith.mulf, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、指针算术、arith.mulf、tt.return。

### Lines 749-749
```mlir
749| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 750-756
```mlir
750| 
751| #blocked = #ttg.blocked<{sizePerThread = [8, 1], threadsPerWarp = [8, 4], warpsPerCTA = [1, 4], order = [0, 1]}>
752| #blocked1 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
753| #mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 16, 16]}>
754| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
755| #shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = true, elementBitWidth = 16}>
756| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 757-757
```mlir
757| module attributes {"ttg.target" = "cuda:90", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 758-758
```mlir
758| // CHECK-LABEL: dot_lhs_registers
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: dot_lhs_registers anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: dot_lhs_registers 这样的标签用于锚定匹配范围。

### Lines 759-776
```mlir
759|   tt.func @dot_lhs_registers(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32}) -> tensor<128x16xf32, #mma> {
760|     %cst = arith.constant dense<0> : tensor<64x16xi32, #blocked>
761|     %c0_i32 = arith.constant 0 : i32
762|     %cst_0 = arith.constant dense<0> : tensor<1x16xi32, #blocked>
763|     %cst_1 = arith.constant dense<0> : tensor<128x1xi32, #blocked1>
764|     %c0_i64 = arith.constant 0 : i64
765|     %cst_2 = arith.constant dense<0.000000e+00> : tensor<128x16xf32, #mma>
766|     %cst_3 = arith.constant dense<0> : tensor<128x64xi32, #blocked1>
767|     %cst_4 = arith.constant dense<2.0> : tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
768|     %c1_i32 = arith.constant 1 : i32
769|     %c8_i32 = arith.constant 8 : i32
770|     %0 = tt.addptr %arg0, %c0_i64 : !tt.ptr<f16>, i64
771|     %1 = tt.addptr %arg1, %c0_i64 : !tt.ptr<f16>, i64
772|     %2 = tt.splat %1 : !tt.ptr<f16> -> tensor<128x1x!tt.ptr<f16>, #blocked1>
773|     %3 = tt.addptr %2, %cst_1 : tensor<128x1x!tt.ptr<f16>, #blocked1>, tensor<128x1xi32, #blocked1>
774|     %4 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>
775|     %5 = tt.expand_dims %4 {axis = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked1}>> -> tensor<1x64xi32, #blocked1>
776|     %6 = tt.broadcast %3 : tensor<128x1x!tt.ptr<f16>, #blocked1> -> tensor<128x64x!tt.ptr<f16>, #blocked1>
```
**EN:** This function-oriented block defines or enters `dot_lhs_registers`. Within it, the test exercises constants, pointer arithmetic, tt.func, broadcasted scalars or pointers, lane/block index ranges, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `dot_lhs_registers` 为核心。测试在其中演示 常量、指针算术、tt.func、广播后的标量或指针、lane/block 索引范围，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 777-785
```mlir
777|     %7 = tt.broadcast %5 : tensor<1x64xi32, #blocked1> -> tensor<128x64xi32, #blocked1>
778|     %8 = tt.addptr %6, %7 : tensor<128x64x!tt.ptr<f16>, #blocked1>, tensor<128x64xi32, #blocked1>
779|     %10 = tt.splat %0 : !tt.ptr<f16> -> tensor<1x16x!tt.ptr<f16>, #blocked>
780|     %11 = tt.addptr %10, %cst_0 : tensor<1x16x!tt.ptr<f16>, #blocked>, tensor<1x16xi32, #blocked>
781|     %12 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
782|     %13 = tt.expand_dims %12 {axis = 1 : i32} : tensor<64xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<64x1xi32, #blocked>
783|     %14 = tt.broadcast %11 : tensor<1x16x!tt.ptr<f16>, #blocked> -> tensor<64x16x!tt.ptr<f16>, #blocked>
784|     %15 = tt.broadcast %13 : tensor<64x1xi32, #blocked> -> tensor<64x16xi32, #blocked>
785|     %16 = tt.addptr %14, %15 : tensor<64x16x!tt.ptr<f16>, #blocked>, tensor<64x16xi32, #blocked>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tensor broadcasting, pointer arithmetic, broadcasted scalars or pointers, lane/block index ranges, shape expansion. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 张量广播、指针算术、广播后的标量或指针、lane/block 索引范围、形状扩展。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 786-803
```mlir
786|     // CHECK: scf.for
787|     // CHECK:   ttg.async_wait {{.*}} {num = 2 : i32}
788|     // CHECK:   ttg.local_load
789|     // CHECK:   ttng.warp_group_dot
790|     // CHECK-NEXT: ttng.warp_group_dot_wait {{.*}} {pendings = 1 : i32}
791|     // CHECK:   ttng.warp_group_dot
792|     // CHECK-NEXT: ttng.warp_group_dot_wait {{.*}} {pendings = 1 : i32}
793|     // CHECK:   ttg.async_copy_global_to_local
794|     // CHECK:   ttg.async_commit_group
795|     // CHECK:   ttg.async_copy_global_to_local
796|     // CHECK:   ttg.async_commit_group
797|     // CHECK:   scf.yield
798|     %17:3 = scf.for %arg3 = %c0_i32 to %c8_i32 step %c1_i32 iter_args(%arg4 = %cst_2, %arg5 = %8, %arg6 = %16) -> (tensor<128x16xf32, #mma>, tensor<128x64x!tt.ptr<f16>, #blocked1>,
799|         tensor<64x16x!tt.ptr<f16>, #blocked>)  : i32 {
800|       %a_block = tt.load %arg5 : tensor<128x64x!tt.ptr<f16>, #blocked1>
801|       %b_block = tt.load %arg6 : tensor<64x16x!tt.ptr<f16>, #blocked>
802|       %a_dotop = ttg.convert_layout %a_block : tensor<128x64xf16, #blocked1> -> tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
803|       %a_dotop_mul = arith.mulf %a_dotop, %cst_4 : tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining masked or vectorized loads, structured loops, layout conversions, arith.mulf.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 带掩码或向量化的加载、结构化循环、布局转换、arith.mulf。

### Lines 804-813
```mlir
804|       %b_smem = ttg.local_alloc %b_block : (tensor<64x16xf16, #blocked>) -> !ttg.memdesc<64x16xf16, #shared1, #smem>
805|       %21 = ttng.warp_group_dot %a_dotop_mul, %b_smem, %arg4 : tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * !ttg.memdesc<64x16xf16, #shared1, #smem> -> tensor<128x16xf32, #mma>
806|       %25 = tt.addptr %arg5, %cst_3 : tensor<128x64x!tt.ptr<f16>, #blocked1>, tensor<128x64xi32, #blocked1>
807|       %26 = tt.addptr %arg6, %cst : tensor<64x16x!tt.ptr<f16>, #blocked>, tensor<64x16xi32, #blocked>
808|       scf.yield %21, %25, %26 : tensor<128x16xf32, #mma>, tensor<128x64x!tt.ptr<f16>, #blocked1>, tensor<64x16x!tt.ptr<f16>, #blocked>
809|     }
810|     tt.return %17#0 : tensor<128x16xf32, #mma>
811|   }
812| }
813| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining pointer arithmetic, shared/local memory allocation, ttng.warp_group_dot, loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 指针算术、共享/本地内存分配、ttng.warp_group_dot、循环/分支产出值、tt.return。

### Lines 814-814
```mlir
814| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 815-820
```mlir
815| 
816| #blocked = #ttg.blocked<{sizePerThread = [8, 1], threadsPerWarp = [8, 4], warpsPerCTA = [1, 4], order = [0, 1]}>
817| #blocked1 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
818| #mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 16, 16]}>
819| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = true, elementBitWidth = 16}>
820| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 821-821
```mlir
821| module attributes {"ttg.target" = "cuda:90", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 822-822
```mlir
822| // CHECK-LABEL: dot_lhs_in_reg_with_epilogue
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: dot_lhs_in_reg_with_epilogue anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: dot_lhs_in_reg_with_epilogue 这样的标签用于锚定匹配范围。

### Lines 823-840
```mlir
823|   tt.func @dot_lhs_in_reg_with_epilogue(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg2: i1) -> tensor<128x16xf32, #mma> {
824|     %cst = arith.constant dense<0> : tensor<128x64xi32, #blocked1>
825|     %cst1 = arith.constant dense<0> : tensor<64x16xi32, #blocked>
826|     %c0_i32 = arith.constant 0 : i32
827|     %cst_0 = arith.constant dense<0> : tensor<1x16xi32, #blocked>
828|     %cst_1 = arith.constant dense<0> : tensor<128x1xi32, #blocked1>
829|     %c0_i64 = arith.constant 0 : i64
830|     %cst_2 = arith.constant dense<0.000000e+00> : tensor<128x16xf32, #mma>
831|     %cst_3 = arith.constant dense<0> : tensor<128x64xi32, #blocked1>
832|     %cst_4 = arith.constant dense<2.0> : tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
833|     %c1_i32 = arith.constant 1 : i32
834|     %c8_i32 = arith.constant 8 : i32
835|     %0 = tt.addptr %arg0, %c0_i64 : !tt.ptr<f16>, i64
836|     %1 = tt.addptr %arg1, %c0_i64 : !tt.ptr<f16>, i64
837|     %2 = tt.splat %1 : !tt.ptr<f16> -> tensor<128x1x!tt.ptr<f16>, #blocked1>
838|     %3 = tt.addptr %2, %cst_1 : tensor<128x1x!tt.ptr<f16>, #blocked1>, tensor<128x1xi32, #blocked1>
839|     %4 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>
840|     %5 = tt.expand_dims %4 {axis = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked1}>> -> tensor<1x64xi32, #blocked1>
```
**EN:** This function-oriented block defines or enters `dot_lhs_in_reg_with_epilogue`. Within it, the test exercises constants, pointer arithmetic, tt.func, broadcasted scalars or pointers, lane/block index ranges, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `dot_lhs_in_reg_with_epilogue` 为核心。测试在其中演示 常量、指针算术、tt.func、广播后的标量或指针、lane/block 索引范围，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 841-850
```mlir
841|     %6 = tt.broadcast %3 : tensor<128x1x!tt.ptr<f16>, #blocked1> -> tensor<128x64x!tt.ptr<f16>, #blocked1>
842|     %7 = tt.broadcast %5 : tensor<1x64xi32, #blocked1> -> tensor<128x64xi32, #blocked1>
843|     %8 = tt.addptr %6, %7 : tensor<128x64x!tt.ptr<f16>, #blocked1>, tensor<128x64xi32, #blocked1>
844|     %10 = tt.splat %0 : !tt.ptr<f16> -> tensor<1x16x!tt.ptr<f16>, #blocked>
845|     %11 = tt.addptr %10, %cst_0 : tensor<1x16x!tt.ptr<f16>, #blocked>, tensor<1x16xi32, #blocked>
846|     %12 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
847|     %13 = tt.expand_dims %12 {axis = 1 : i32} : tensor<64xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<64x1xi32, #blocked>
848|     %14 = tt.broadcast %11 : tensor<1x16x!tt.ptr<f16>, #blocked> -> tensor<64x16x!tt.ptr<f16>, #blocked>
849|     %15 = tt.broadcast %13 : tensor<64x1xi32, #blocked> -> tensor<64x16xi32, #blocked>
850|     %16 = tt.addptr %14, %15 : tensor<64x16x!tt.ptr<f16>, #blocked>, tensor<64x16xi32, #blocked>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tensor broadcasting, pointer arithmetic, broadcasted scalars or pointers, lane/block index ranges, shape expansion. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 张量广播、指针算术、广播后的标量或指针、lane/block 索引范围、形状扩展。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 851-868
```mlir
851|     // CHECK: scf.for
852|     // CHECK:   ttg.async_wait {{.*}} {num = 2 : i32}
853|     // CHECK:   ttng.warp_group_dot
854|     // CHECK-NEXT: ttng.warp_group_dot_wait {{.*}} {pendings = 1 : i32}
855|     // CHECK:   ttng.warp_group_dot
856|     // CHECK-NEXT: ttng.warp_group_dot_wait {{.*}} {pendings = 1 : i32}
857|     // CHECK:   ttg.async_copy_global_to_local
858|     // CHECK:   ttg.async_copy_global_to_local
859|     // CHECK:   ttg.async_commit_group
860|     // CHECK:   scf.if
861|     // CHECK-NEXT: ttng.warp_group_dot_wait {{.*}} {pendings = 0 : i32}
862|     // CHECK:   } else {
863|     // CHECK-NOT: ttng.warp_group_dot_wait
864|     // CHECK:   scf.yield
865|     %17:3 = scf.for %arg3 = %c0_i32 to %c8_i32 step %c1_i32 iter_args(%arg4 = %cst_2, %arg5 = %8, %arg6 = %16) -> (tensor<128x16xf32, #mma>, tensor<128x64x!tt.ptr<f16>, #blocked1>,
866|         tensor<64x16x!tt.ptr<f16>, #blocked>)  : i32 {
867|       %a_block = tt.load %arg5 : tensor<128x64x!tt.ptr<f16>, #blocked1>
868|       %b_block = tt.load %arg6 : tensor<64x16x!tt.ptr<f16>, #blocked>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining masked or vectorized loads, structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 带掩码或向量化的加载、结构化循环。

### Lines 869-886
```mlir
869|       %a_dotop = ttg.convert_layout %a_block : tensor<128x64xf16, #blocked1> -> tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
870|       %a_dotop_mul = arith.mulf %a_dotop, %cst_4 : tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
871|       %b_smem = ttg.local_alloc %b_block : (tensor<64x16xf16, #blocked>) -> !ttg.memdesc<64x16xf16, #shared, #smem>
872|       %25 = ttng.warp_group_dot %a_dotop_mul, %b_smem, %arg4 : tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * !ttg.memdesc<64x16xf16, #shared, #smem> -> tensor<128x16xf32, #mma>
873|       %26 = tt.addptr %arg5, %cst : tensor<128x64x!tt.ptr<f16>, #blocked1>, tensor<128x64xi32, #blocked1>
874|       %27 = tt.addptr %arg6, %cst1 : tensor<64x16x!tt.ptr<f16>, #blocked>, tensor<64x16xi32, #blocked>
875|       %28 = scf.if %arg2 -> tensor<128x16xf32, #mma> {
876|         %29 = arith.addf %25, %25 : tensor<128x16xf32, #mma>
877|         scf.yield %29: tensor<128x16xf32, #mma>
878|       } else {
879|         scf.yield %25: tensor<128x16xf32, #mma>
880|       }
881|       scf.yield %28, %26, %27 : tensor<128x16xf32, #mma>, tensor<128x64x!tt.ptr<f16>, #blocked1>, tensor<64x16x!tt.ptr<f16>, #blocked>
882|     }
883|     tt.return %17#0 : tensor<128x16xf32, #mma>
884|   }
885| }
886| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, pointer arithmetic, layout conversions, arith.mulf, shared/local memory allocation.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、指针算术、布局转换、arith.mulf、共享/本地内存分配。

### Lines 887-887
```mlir
887| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 888-905
```mlir
888| 
889| #blocked = #ttg.blocked<{sizePerThread = [16, 1], threadsPerWarp = [16, 2], warpsPerCTA = [1, 8], order = [0, 1]}>
890| #blocked1 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [8, 1], order = [1, 0]}>
891| #blocked2 = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [1, 32], warpsPerCTA = [8, 1], order = [1, 0]}>
892| #linear = #ttg.linear<{register = [[1, 0], [0, 8], [8, 0], [16, 0], [32, 0], [64, 0], [0, 128]], lane = [[2, 0], [4, 0], [0, 1], [0, 2], [0, 4]], warp = [[0, 16], [0, 32], [0, 64]], block = []}>
893| #linear1 = #ttg.linear<{register = [[1, 0], [2, 0], [4, 0], [8, 0], [128, 0], [0, 32]], lane = [[16, 0], [32, 0], [64, 0], [0, 1], [0, 2]], warp = [[0, 4], [0, 8], [0, 16]], block = []}>
894| #linear2 = #ttg.linear<{register = [[0, 1], [0, 2], [0, 64], [0, 32]], lane = [[0, 0], [0, 0], [0, 4], [0, 8], [0, 16]], warp = [[1, 0], [2, 0], [4, 0]], block = []}>
895| #linear3 = #ttg.linear<{register = [[0, 0, 0, 0, 0, 0, 1], [0, 0, 0, 0, 0, 1, 0], [0, 0, 1, 0, 0, 0, 0], [0, 0, 0, 1, 0, 0, 0]], lane = [[0, 0, 0, 0, 0, 0, 0], [0, 0, 0, 0, 0, 0, 0], [0, 0, 0, 0, 1, 0, 0], [0, 0, 0, 0, 2, 0, 0], [0, 0, 0, 0, 4, 0, 0]], warp = [[0, 1, 0, 0, 0, 0, 0], [0, 2, 0, 0, 0, 0, 0], [0, 4, 0, 0, 0, 0, 0]], block = []}>
896| #linear4 = #ttg.linear<{register = [[0, 0, 0, 1, 0, 0, 0], [0, 0, 0, 0, 0, 0, 1], [0, 0, 0, 0, 0, 1, 0], [0, 1, 0, 0, 0, 0, 0]], lane = [[0, 0, 0, 0, 0, 0, 0], [0, 0, 0, 0, 0, 0, 0], [0, 0, 0, 0, 1, 0, 0], [0, 0, 0, 0, 2, 0, 0], [0, 0, 0, 0, 4, 0, 0]], warp = [[0, 0, 1, 0, 0, 0, 0], [0, 0, 2, 0, 0, 0, 0], [0, 0, 4, 0, 0, 0, 0]], block = []}>
897| #linear5 = #ttg.linear<{register = [[0, 0, 1], [8, 0, 0], [0, 0, 8], [0, 0, 16], [0, 1, 0], [0, 2, 0], [128, 0, 0]], lane = [[0, 0, 2], [0, 0, 4], [1, 0, 0], [2, 0, 0], [4, 0, 0]], warp = [[16, 0, 0], [32, 0, 0], [64, 0, 0]], block = []}>
898| #linear6 = #ttg.linear<{register = [[0, 1], [0, 2], [0, 4], [0, 8], [0, 128], [32, 0]], lane = [[0, 16], [0, 32], [0, 64], [1, 0], [2, 0]], warp = [[4, 0], [8, 0], [16, 0]], block = []}>
899| #linear7 = #ttg.linear<{register = [[0, 1, 0], [0, 2, 0], [0, 0, 1], [0, 4, 0], [0, 8, 0], [0, 128, 0], [32, 0, 0]], lane = [[0, 16, 0], [0, 32, 0], [0, 64, 0], [1, 0, 0], [2, 0, 0]], warp = [[4, 0, 0], [8, 0, 0], [16, 0, 0]], block = []}>
900| #linear8 = #ttg.linear<{register = [[0, 0, 1, 0], [0, 0, 2, 0], [0, 0, 0, 1], [0, 1, 0, 0], [0, 2, 0, 0], [0, 32, 0, 0], [32, 0, 0, 0]], lane = [[0, 4, 0, 0], [0, 8, 0, 0], [0, 16, 0, 0], [1, 0, 0, 0], [2, 0, 0, 0]], warp = [[4, 0, 0, 0], [8, 0, 0, 0], [16, 0, 0, 0]], block = []}>
901| #linear9 = #ttg.linear<{register = [[0, 0, 0, 1], [0, 0, 0, 2], [0, 0, 1, 0], [0, 1, 0, 0], [0, 2, 0, 0], [0, 32, 0, 0], [32, 0, 0, 0]], lane = [[0, 4, 0, 0], [0, 8, 0, 0], [0, 16, 0, 0], [1, 0, 0, 0], [2, 0, 0, 0]], warp = [[4, 0, 0, 0], [8, 0, 0, 0], [16, 0, 0, 0]], block = []}>
902| #linear10 = #ttg.linear<{register = [[0, 0, 0, 0, 0, 0, 0, 1], [0, 0, 0, 0, 0, 0, 1, 0], [0, 0, 0, 0, 0, 1, 0, 0], [0, 0, 0, 0, 0, 2, 0, 0], [0, 0, 0, 0, 0, 4, 0, 0], [0, 0, 1, 0, 0, 0, 0, 0], [8, 0, 0, 0, 0, 0, 0, 0]], lane = [[0, 0, 0, 0, 1, 0, 0, 0], [0, 0, 0, 0, 2, 0, 0, 0], [0, 0, 0, 1, 0, 0, 0, 0], [0, 1, 0, 0, 0, 0, 0, 0], [0, 2, 0, 0, 0, 0, 0, 0]], warp = [[1, 0, 0, 0, 0, 0, 0, 0], [2, 0, 0, 0, 0, 0, 0, 0], [4, 0, 0, 0, 0, 0, 0, 0]], block = []}>
903| #linear11 = #ttg.linear<{register = [[0, 0, 0, 0, 0, 0, 0, 1], [0, 1, 0, 0, 0, 0, 0, 0], [0, 0, 0, 0, 0, 1, 0, 0], [0, 0, 0, 0, 0, 2, 0, 0], [0, 0, 0, 0, 0, 4, 0, 0], [0, 0, 0, 0, 1, 0, 0, 0], [8, 0, 0, 0, 0, 0, 0, 0]], lane = [[0, 0, 0, 0, 0, 0, 1, 0], [0, 0, 0, 0, 0, 0, 2, 0], [0, 0, 0, 1, 0, 0, 0, 0], [0, 0, 1, 0, 0, 0, 0, 0], [0, 0, 2, 0, 0, 0, 0, 0]], warp = [[1, 0, 0, 0, 0, 0, 0, 0], [2, 0, 0, 0, 0, 0, 0, 0], [4, 0, 0, 0, 0, 0, 0, 0]], block = []}>
904| #linear12 = #ttg.linear<{register = [[0, 1], [8, 0], [0, 8], [0, 16], [0, 32], [0, 64], [128, 0]], lane = [[0, 2], [0, 4], [1, 0], [2, 0], [4, 0]], warp = [[16, 0], [32, 0], [64, 0]], block = []}>
905| #mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [8, 1], instrShape = [16, 128, 16]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 906-908
```mlir
906| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
907| #shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = true, elementBitWidth = 16}>
908| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 909-909
```mlir
909| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 910-910
```mlir
910|   // CHECK-LABEL: dot_lhs_swizzling
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: dot_lhs_swizzling anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: dot_lhs_swizzling 这样的标签用于锚定匹配范围。

### Lines 911-928
```mlir
911|   tt.func @dot_lhs_swizzling(%arg0: !tt.ptr<i8> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<bf16> {tt.divisibility = 16 : i32}, %arg2: !tt.ptr<i8> {tt.divisibility = 16 : i32}) -> tensor<256x128xf32, #mma> {
912|     %c0_i32 = arith.constant 0 : i32
913|     %c1_i32 = arith.constant 1 : i32
914|     %c8_i32 = arith.constant 8 : i32
915|     %cst = arith.constant dense<256> : tensor<256x64xi32, #blocked>
916|     %cst_0 = arith.constant dense<128> : tensor<128x128xi32, #blocked1>
917|     %cst_1 = arith.constant dense<128> : tensor<8x128xi32, #blocked2>
918|     %cst_2 = arith.constant dense<0.000000e+00> : tensor<128x256xf32, #linear>
919|     %0 = tt.splat %arg0 : !tt.ptr<i8> -> tensor<1x64x!tt.ptr<i8>, #blocked>
920|     %1 = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
921|     %2 = tt.expand_dims %1 {axis = 1 : i32} : tensor<256xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<256x1xi32, #blocked>
922|     %3 = tt.broadcast %0 : tensor<1x64x!tt.ptr<i8>, #blocked> -> tensor<256x64x!tt.ptr<i8>, #blocked>
923|     %4 = tt.broadcast %2 : tensor<256x1xi32, #blocked> -> tensor<256x64xi32, #blocked>
924|     %5 = tt.addptr %3, %4 : tensor<256x64x!tt.ptr<i8>, #blocked>, tensor<256x64xi32, #blocked>
925| 
926|     %6 = tt.splat %arg1 : !tt.ptr<bf16> -> tensor<128x1x!tt.ptr<bf16>, #blocked1>
927|     %7 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>
928|     %8 = tt.expand_dims %7 {axis = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked1}>> -> tensor<1x128xi32, #blocked1>
```
**EN:** This function-oriented block defines or enters `dot_lhs_swizzling`. Within it, the test exercises constants, tt.func, broadcasted scalars or pointers, lane/block index ranges, shape expansion, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `dot_lhs_swizzling` 为核心。测试在其中演示 常量、tt.func、广播后的标量或指针、lane/block 索引范围、形状扩展，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 929-938
```mlir
929|     %9 = tt.broadcast %6 : tensor<128x1x!tt.ptr<bf16>, #blocked1> -> tensor<128x128x!tt.ptr<bf16>, #blocked1>
930|     %10 = tt.broadcast %8 : tensor<1x128xi32, #blocked1> -> tensor<128x128xi32, #blocked1>
931|     %11 = tt.addptr %9, %10 : tensor<128x128x!tt.ptr<bf16>, #blocked1>, tensor<128x128xi32, #blocked1>
932| 
933|     %12 = tt.splat %arg2 : !tt.ptr<i8> -> tensor<8x1x!tt.ptr<i8>, #blocked2>
934|     %13 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked2}>>
935|     %14 = tt.expand_dims %13 {axis = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked2}>> -> tensor<1x128xi32, #blocked2>
936|     %15 = tt.broadcast %12 : tensor<8x1x!tt.ptr<i8>, #blocked2> -> tensor<8x128x!tt.ptr<i8>, #blocked2>
937|     %16 = tt.broadcast %14 : tensor<1x128xi32, #blocked2> -> tensor<8x128xi32, #blocked2>
938|     %17 = tt.addptr %15, %16 : tensor<8x128x!tt.ptr<i8>, #blocked2>, tensor<8x128xi32, #blocked2>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tensor broadcasting, pointer arithmetic, broadcasted scalars or pointers, lane/block index ranges, shape expansion. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 张量广播、指针算术、广播后的标量或指针、lane/block 索引范围、形状扩展。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 939-956
```mlir
939|     // CHECK: scf.for
940|     // CHECK:   ttg.async_wait {{.*}} {num = 3 : i32}
941|     // CHECK:   ttg.local_load
942|     // CHECK:   ttg.local_load
943|     // CHECK:   ttng.warp_group_dot
944|     // CHECK-NEXT: ttng.warp_group_dot_wait {{.*}} {pendings = 1 : i32}
945|     // CHECK:   ttng.warp_group_dot
946|     // CHECK-NEXT: ttng.warp_group_dot_wait {{.*}} {pendings = 1 : i32}
947|     // CHECK:   ttng.warp_group_dot
948|     // CHECK:   ttg.async_copy_global_to_local
949|     // CHECK:   ttg.async_commit_group
950|     // CHECK:   ttg.async_copy_global_to_local
951|     // CHECK:   ttg.async_commit_group
952|     // CHECK:   ttg.async_copy_global_to_local
953|     // CHECK:   ttg.async_commit_group
954|     // CHECK:   scf.yield
955|     %18:4 = scf.for %arg3 = %c0_i32 to %c8_i32 step %c1_i32 iter_args(%arg4 = %cst_2, %arg5 = %11, %arg6 = %5, %arg7 = %17) -> (tensor<128x256xf32, #linear>, tensor<128x128x!tt.ptr<bf16>, #blocked1>, tensor<256x64x!tt.ptr<i8>, #blocked>, tensor<8x128x!tt.ptr<i8>, #blocked2>)  : i32 {
956|       %21 = tt.load %arg5 : tensor<128x128x!tt.ptr<bf16>, #blocked1>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops, masked or vectorized loads.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环、带掩码或向量化的加载。

### Lines 957-974
```mlir
957|       %22 = tt.load %arg6 : tensor<256x64x!tt.ptr<i8>, #blocked>
958|       %23 = ttg.convert_layout %22 : tensor<256x64xi8, #blocked> -> tensor<256x64xi8, #linear1>
959|       %24 = tt.load %arg7 : tensor<8x128x!tt.ptr<i8>, #blocked2>
960|       %25 = ttg.convert_layout %24 : tensor<8x128xi8, #blocked2> -> tensor<8x128xi8, #linear2>
961|       %26 = tt.reshape %25 : tensor<8x128xi8, #linear2> -> tensor<1x8x2x2x8x2x2xi8, #linear3>
962|       %27 = tt.trans %26 {order = array<i32: 0, 3, 1, 6, 4, 2, 5>} : tensor<1x8x2x2x8x2x2xi8, #linear3> -> tensor<1x2x8x2x8x2x2xi8, #linear4>
963|       %28 = tt.reshape %27 : tensor<1x2x8x2x8x2x2xi8, #linear4> -> tensor<256x4xi8, #ttg.slice<{dim = 2, parent = #linear5}>>
964|       %29 = tt.trans %23 {order = array<i32: 1, 0>} : tensor<256x64xi8, #linear1> -> tensor<64x256xi8, #linear6>
965|       %30:2 = tt.elementwise_inline_asm "\0A        {\0A            .reg .b32 b, c, d<7>, scale;\0A            and.b32 $0, $4, 0b10000001110000001000000111000000;\0A            shl.b32 b, $4, 3;\0A            and.b32 $1, b,  0b10000001110000001000000111000000;\0A            shl.b32 c, $4, 6;\0A            and.b32 $2, c,  0b10000001110000001000000111000000;\0A            \0A            shl.b32 d0, $4, 1;\0A            and.b32 d1, d0, 0b10000000000000001000000000000000;\0A            shr.b32 d2, $4, 3;\0A            and.b32 d3, d2, 0b00000001100000000000000110000000;\0A            or.b32 d4, d1, d3;\0A            shr.b32 d5, $4, 7;\0A            and.b32 d6, d5, 0b00000000010000000000000001000000;\0A            or.b32 $3, d4, d6;\0A        }\0A        " {constraints = "=r,=r,=r,=r,r", packed_element = 4 : i32, pure = true} %29 : tensor<64x256xi8, #linear6> -> tensor<64x256xbf16, #linear6>, tensor<64x256xbf16, #linear6>
966|       %31 = tt.join %30#0, %30#1 : tensor<64x256xbf16, #linear6> -> tensor<64x256x2xbf16, #linear7>
967|       %32 = tt.reshape %31 : tensor<64x256x2xbf16, #linear7> -> tensor<64x64x4x2xbf16, #linear8>
968|       %33 = tt.trans %32 {order = array<i32: 0, 1, 3, 2>} : tensor<64x64x4x2xbf16, #linear8> -> tensor<64x64x2x4xbf16, #linear9>
969|       %34 = tt.reshape %33 : tensor<64x64x2x4xbf16, #linear9> -> tensor<16x4x2x2x4x8x2x2xbf16, #linear10>
970|       %35 = tt.trans %34 {order = array<i32: 0, 6, 1, 3, 2, 5, 4, 7>} : tensor<16x4x2x2x4x8x2x2xbf16, #linear10> -> tensor<16x2x4x2x2x8x4x2xbf16, #linear11>
971|       %36 = tt.reshape %35 : tensor<16x2x4x2x2x8x4x2xbf16, #linear11> -> tensor<256x128xbf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
972|       %37 = tt.elementwise_inline_asm "\0A        {\0A            // Assumes no overflow\0A            add.u32 $2, $2, 0x7E7E7E7E;\0A            prmt.b32 $0, $2, 0, 0x5140;\0A            shl.b32 $0, $0, 7;\0A            prmt.b32 $1, $2, 0, 0x7362;\0A            shl.b32 $1, $1, 7;\0A        }\0A        " {constraints = "=r,=r,r", packed_element = 4 : i32, pure = true} %28 : tensor<256x4xi8, #ttg.slice<{dim = 2, parent = #linear5}>> -> tensor<256x4xbf16, #ttg.slice<{dim = 2, parent = #linear5}>>
973|       %38 = tt.expand_dims %37 {axis = 2 : i32} : tensor<256x4xbf16, #ttg.slice<{dim = 2, parent = #linear5}>> -> tensor<256x4x1xbf16, #linear5>
974|       %39 = tt.broadcast %38 : tensor<256x4x1xbf16, #linear5> -> tensor<256x4x32xbf16, #linear5>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tensor reshaping, transpose-like layout changes, masked or vectorized loads, layout conversions, tt.elementwise_inline_asm. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 张量重塑、转置类布局变换、带掩码或向量化的加载、布局转换、tt.elementwise_inline_asm。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 975-992
```mlir
975|       %40 = tt.reshape %39 : tensor<256x4x32xbf16, #linear5> -> tensor<256x128xbf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
976|       %41 = arith.mulf %36, %40 : tensor<256x128xbf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
977|       %42 = tt.trans %arg4 {order = array<i32: 1, 0>} : tensor<128x256xf32, #linear> -> tensor<256x128xf32, #linear12>
978|       %43 = ttg.local_alloc %21 : (tensor<128x128xbf16, #blocked1>) -> !ttg.memdesc<128x128xbf16, #shared, #smem>
979|       %44 = ttg.memdesc_trans %43 {order = array<i32: 1, 0>} : !ttg.memdesc<128x128xbf16, #shared, #smem> -> !ttg.memdesc<128x128xbf16, #shared1, #smem>
980|       %45 = ttg.convert_layout %42 : tensor<256x128xf32, #linear12> -> tensor<256x128xf32, #mma>
981|       %46 = ttng.warp_group_dot %41, %44, %45 {inputPrecision = 0 : i32} : tensor<256x128xbf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * !ttg.memdesc<128x128xbf16, #shared1, #smem> -> tensor<256x128xf32, #mma>
982|       %47 = tt.trans %46 {order = array<i32: 1, 0>} : tensor<256x128xf32, #mma> -> tensor<128x256xf32, #linear>
983|       %48 = tt.addptr %arg7, %cst_1 : tensor<8x128x!tt.ptr<i8>, #blocked2>, tensor<8x128xi32, #blocked2>
984|       %49 = tt.addptr %arg5, %cst_0 : tensor<128x128x!tt.ptr<bf16>, #blocked1>, tensor<128x128xi32, #blocked1>
985|       %50 = tt.addptr %arg6, %cst : tensor<256x64x!tt.ptr<i8>, #blocked>, tensor<256x64xi32, #blocked>
986|       scf.yield %47, %49, %50, %48 : tensor<128x256xf32, #linear>, tensor<128x128x!tt.ptr<bf16>, #blocked1>, tensor<256x64x!tt.ptr<i8>, #blocked>, tensor<8x128x!tt.ptr<i8>, #blocked2>
987|     }
988|     %19 = tt.trans %18#0 {order = array<i32: 1, 0>} : tensor<128x256xf32, #linear> -> tensor<256x128xf32, #linear12>
989|     %20 = ttg.convert_layout %19 : tensor<256x128xf32, #linear12> -> tensor<256x128xf32, #mma>
990|     tt.return %20 : tensor<256x128xf32, #mma>
991|   }
992| }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining transpose-like layout changes, pointer arithmetic, layout conversions, tensor reshaping, arith.mulf.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 转置类布局变换、指针算术、布局转换、张量重塑、arith.mulf。

### Lines 994-995
```mlir
994| 
995| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 995-1001
```mlir
 995| 
 996| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = false, elementBitWidth = 8}>
 997| #shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = true, elementBitWidth = 8}>
 998| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [4, 2], order = [1, 0]}>
 999| #mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [8, 1], instrShape = [16, 64, 32]}>
1000| #nvmma_64 = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = false, elementBitWidth = 16}>
1001| #nvmma_128 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1002-1002
```mlir
1002| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 1003-1003
```mlir
1003|   tt.func public @mmav3_fp8_row_major_rhs(%arg0: !tt.ptr<i8, 0> {tt.nv_tma_desc = 1 : i32}, %arg1: !tt.ptr<i8, 0> {tt.nv_tma_desc = 1 : i32}, %arg2: !tt.ptr<i8, 0> {tt.nv_tma_desc = 1 : i32}, %arg3: i32 {tt.divisibility = 16 : i32}, %arg4: i32 {tt.divisibility = 16 : i32}, %arg5: i32 {tt.divisibility = 16 : i32}) {
```
**EN:** This function-oriented block defines or enters `mmav3_fp8_row_major_rhs`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `mmav3_fp8_row_major_rhs` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1004-1007
```mlir
1004|     // CHECK-LABEL: mmav3_fp8_row_major_rhs
1005|     // The col-major RHS SMEM encoding in the input, created by accelerate-matmul, should be overwritten by the row-major TMA layout.
1006|     // Note that this "overwriting" makes the program invalid after SWP, since warp_group_dot does not support row-major fp8 RHS.
1007|     // In this case, the TMA load on B should not be pipelined. When this bug is fixed, this test should be rewritten to verify that.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1008-1025
```mlir
1008|     // CHECK-NOT: order = [0, 1]
1009|     // CHECK: tt.return
1010|     %c128_i32 = arith.constant 128 : i32
1011|     %c64_i32 = arith.constant 64 : i32
1012|     %c0_i32 = arith.constant 0 : i32
1013|     %c1_i32 = arith.constant 1 : i32
1014|     %c127_i32 = arith.constant 127 : i32
1015|     %c63_i32 = arith.constant 63 : i32
1016|     %cst = arith.constant dense<0.000000e+00> : tensor<128x64xf32, #mma>
1017|     %0 = tt.get_program_id x : i32
1018|     %1 = arith.addi %arg3, %c127_i32 : i32
1019|     %2 = arith.divsi %1, %c128_i32 : i32
1020|     %3 = arith.remsi %0, %2 : i32
1021|     %4 = arith.divsi %0, %2 : i32
1022|     %5 = arith.muli %3, %c128_i32 : i32
1023|     %6 = arith.muli %4, %c64_i32 : i32
1024|     %7 = arith.addi %arg5, %c63_i32 : i32
1025|     %8 = arith.divsi %7, %c64_i32 : i32
```
**EN:** This operational block carries the concrete IR steps for the test, centered on constants, arith.divsi, integer additions, integer multiplications, program IDs. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 常量、arith.divsi、整数加法、整数乘法、程序 ID。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1026-1043
```mlir
1026|     %9 = ttng.reinterpret_tensor_descriptor %arg0 : !tt.ptr<i8, 0> to !tt.tensordesc<128x64xf8E4M3FN, #shared>
1027|     %10 = ttng.reinterpret_tensor_descriptor %arg1 : !tt.ptr<i8, 0> to !tt.tensordesc<64x64xf8E4M3FN, #shared>
1028|     %true = arith.constant true
1029|     %false = arith.constant false
1030|     %11:2 = scf.for %arg6 = %c0_i32 to %8 step %c1_i32 iter_args(%arg7 = %cst, %arg8 = %c0_i32) -> (tensor<128x64xf32, #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [8, 1], instrShape = [16, 64, 32]}>>, i32)  : i32 {
1031|       %14 = tt.descriptor_load %9[%5, %arg8] : !tt.tensordesc<128x64xf8E4M3FN, #shared> -> tensor<128x64xf8E4M3FN, #blocked>
1032|       %15 = ttg.local_alloc %14 : (tensor<128x64xf8E4M3FN, #blocked>) -> !ttg.memdesc<128x64xf8E4M3FN, #shared, #ttg.shared_memory>
1033|       %16 = tt.descriptor_load %10[%arg8, %6] : !tt.tensordesc<64x64xf8E4M3FN, #shared> -> tensor<64x64xf8E4M3FN, #blocked>
1034|       %17 = ttg.local_alloc %16 : (tensor<64x64xf8E4M3FN, #blocked>) -> !ttg.memdesc<64x64xf8E4M3FN, #shared1, #ttg.shared_memory>
1035|       %18 = ttng.warp_group_dot %15, %17, %arg7 {inputPrecision = 0 : i32, maxNumImpreciseAcc = 1073741824 : i32} : !ttg.memdesc<128x64xf8E4M3FN, #shared, #ttg.shared_memory> * !ttg.memdesc<64x64xf8E4M3FN, #shared1, #ttg.shared_memory> -> tensor<128x64xf32, #mma>
1036|       %19 = arith.addi %arg8, %c64_i32 : i32
1037|       scf.yield %18, %19 : tensor<128x64xf32, #mma>, i32
1038|     }
1039|     %12 = ttg.convert_layout %11#0 : tensor<128x64xf32, #mma> -> tensor<128x64xf32, #blocked>
1040|     %13 = ttng.reinterpret_tensor_descriptor %arg2 : !tt.ptr<i8, 0> to !tt.tensordesc<128x64xf32, #nvmma_128>
1041|     tt.descriptor_store %13[%5, %6], %12 : !tt.tensordesc<128x64xf32, #nvmma_128>, tensor<128x64xf32, #blocked>
1042|     tt.return
1043|   }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining ttng.reinterpret_tensor_descriptor, constants, tt.descriptor_load, shared/local memory allocation, structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 ttng.reinterpret_tensor_descriptor、常量、tt.descriptor_load、共享/本地内存分配、结构化循环。

### Lines 1044-1045
```mlir
1044| }
1045| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 1046-1046
```mlir
1046| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1047-1052
```mlir
1047| 
1048| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
1049| #mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 32, 16]}>
1050| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = false, elementBitWidth = 16}>
1051| #shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = true, elementBitWidth = 16}>
1052| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1053-1053
```mlir
1053| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 1054-1058
```mlir
1054|   // CHECK: wgmma_not_yielded
1055|   // CHECK: scf.for
1056|   // CHECK-NEXT: ttng.warp_group_dot
1057|   // CHECK-NEXT: ttng.warp_group_dot_wait
1058| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 1059-1076
```mlir
1059|   tt.func public @wgmma_not_yielded() -> tensor<64x32xf32, #mma> {
1060|     %cst = arith.constant dense<3.000000e+00> : tensor<64x32xf32, #mma>
1061|     %c0_i32 = arith.constant 0 : i32
1062|     %c32_i32 = arith.constant 32 : i32
1063|     %c64_i32 = arith.constant 64 : i32
1064|     %cst_0 = arith.constant dense<0.000000e+00> : tensor<64x32xf32, #mma>
1065|     %cst_1 = arith.constant dense<1.000000e+00> : tensor<64x32xbf16, #blocked>
1066|     %cst_2 = arith.constant dense<1.000000e+00> : tensor<32x32xbf16, #blocked>
1067|     %0 = ttg.local_alloc %cst_1 : (tensor<64x32xbf16, #blocked>) -> !ttg.memdesc<64x32xbf16, #shared, #smem, mutable>
1068|     %1 = ttg.local_alloc %cst_2 : (tensor<32x32xbf16, #blocked>) -> !ttg.memdesc<32x32xbf16, #shared1, #smem, mutable>
1069|     %2 = scf.for %arg0 = %c0_i32 to %c64_i32 step %c32_i32 iter_args(%arg1 = %cst_0) -> (tensor<64x32xf32, #mma>)  : i32 {
1070|       %3 = ttng.warp_group_dot %0, %1, %cst_0 {inputPrecision = 0 : i32} : !ttg.memdesc<64x32xbf16, #shared, #smem, mutable> * !ttg.memdesc<32x32xbf16, #shared1, #smem, mutable> -> tensor<64x32xf32, #mma>
1071|       %4 = arith.cmpi ne, %arg0, %c0_i32 : i32
1072|       %5 = scf.if %4 -> (tensor<64x32xf32, #mma>) {
1073|         %6 = arith.addf %3, %cst : tensor<64x32xf32, #mma>
1074|         scf.yield %6 : tensor<64x32xf32, #mma>
1075|       } else {
1076|         %6 = arith.mulf %3, %cst : tensor<64x32xf32, #mma>
```
**EN:** This function-oriented block defines or enters `wgmma_not_yielded`. Within it, the test exercises constants, tt.func, shared/local memory allocation, structured loops, ttng.warp_group_dot, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `wgmma_not_yielded` 为核心。测试在其中演示 常量、tt.func、共享/本地内存分配、结构化循环、ttng.warp_group_dot，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1077-1084
```mlir
1077|         scf.yield %6 : tensor<64x32xf32, #mma>
1078|       }
1079|       scf.yield %5 : tensor<64x32xf32, #mma>
1080|     }
1081|     tt.return %2 : tensor<64x32xf32, #mma>
1082|   }
1083| }
1084| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、tt.return。

### Lines 1085-1085
```mlir
1085| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1086-1090
```mlir
1086| 
1087| #mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 32, 16]}>
1088| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = false, elementBitWidth = 16}>
1089| #shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = true, elementBitWidth = 16}>
1090| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1091-1091
```mlir
1091| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 1092-1098
```mlir
1092|   // CHECK-LABEL: dot_outer_loop_arg
1093|   // CHECK: scf.for
1094|   // CHECK-NEXT: scf.for
1095|   // CHECK-NEXT: ttng.warp_group_dot
1096|   // CHECK-NEXT: ttng.warp_group_dot_wait {{.*}} {pendings = 1 : i32}
1097|   // CHECK-NEXT: scf.yield
1098|   // CHECK: ttng.warp_group_dot_wait {{.*}} {pendings = 0 : i32}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: dot_outer_loop_arg anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: dot_outer_loop_arg 这样的标签用于锚定匹配范围。

### Lines 1099-1113
```mlir
1099|   tt.func public @dot_outer_loop_arg(%arg0: i32, %arg2: !ttg.memdesc<64x32xbf16, #shared, #smem, mutable>, %arg3: !ttg.memdesc<32x32xbf16, #shared1, #smem, mutable>) -> tensor<64x32xf32, #mma> {
1100|     %c0_i32 = arith.constant 0 : i32
1101|     %c32_i32 = arith.constant 32 : i32
1102|     %cst_0 = arith.constant dense<0.000000e+00> : tensor<64x32xf32, #mma>
1103|     %outer:2 = scf.for %arg4 = %c0_i32 to %arg0 step %c32_i32 iter_args(%arg5 = %arg3, %arg8 = %cst_0) -> (!ttg.memdesc<32x32xbf16, #shared1, #smem, mutable>, tensor<64x32xf32, #mma>)  : i32 {
1104|       %0 = scf.for %arg6 = %c0_i32 to %arg0 step %c32_i32 iter_args(%arg7 = %arg8) -> (tensor<64x32xf32, #mma>)  : i32 {
1105|         %1 = ttng.warp_group_dot %arg2, %arg5, %arg7 {inputPrecision = 0 : i32} : !ttg.memdesc<64x32xbf16, #shared, #smem, mutable> * !ttg.memdesc<32x32xbf16, #shared1, #smem, mutable> -> tensor<64x32xf32, #mma>
1106|         scf.yield %1 : tensor<64x32xf32, #mma>
1107|       }
1108|       scf.yield %arg5, %0 : !ttg.memdesc<32x32xbf16, #shared1, #smem, mutable>, tensor<64x32xf32, #mma>
1109|     }
1110|     tt.return %outer#1 : tensor<64x32xf32, #mma>
1111|   }
1112| }
1113| 
```
**EN:** This function-oriented block defines or enters `dot_outer_loop_arg`. Within it, the test exercises constants, tt.func, structured loops, loop/if yielded values, ttng.warp_group_dot, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `dot_outer_loop_arg` 为核心。测试在其中演示 常量、tt.func、结构化循环、循环/分支产出值、ttng.warp_group_dot，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1114-1114
```mlir
1114| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1115-1119
```mlir
1115| 
1116| #mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 32, 16]}>
1117| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = false, elementBitWidth = 16}>
1118| #shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = true, elementBitWidth = 16}>
1119| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1120-1120
```mlir
1120| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 1121-1126
```mlir
1121|   // CHECK-LABEL: loop_arg_cycle
1122|   // CHECK: scf.for
1123|   // CHECK-NEXT: ttng.warp_group_dot
1124|   // CHECK-NEXT: ttng.warp_group_dot_wait {{.*}} {pendings = 1 : i32}
1125|   // CHECK-NEXT: scf.yield
1126|   // CHECK: ttng.warp_group_dot_wait {{.*}} {pendings = 0 : i32}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: loop_arg_cycle anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: loop_arg_cycle 这样的标签用于锚定匹配范围。

### Lines 1127-1137
```mlir
1127|   tt.func public @loop_arg_cycle(%arg0: i32, %arg2: !ttg.memdesc<64x32xbf16, #shared, #smem, mutable>, %arg3: !ttg.memdesc<32x32xbf16, #shared1, #smem, mutable>) -> tensor<64x32xf32, #mma> {
1128|     %c0_i32 = arith.constant 0 : i32
1129|     %c32_i32 = arith.constant 32 : i32
1130|     %cst_0 = arith.constant dense<0.000000e+00> : tensor<64x32xf32, #mma>
1131|     %0:2 = scf.for %arg4 = %c0_i32 to %arg0 step %c32_i32 iter_args(%arg5 = %arg3, %arg7 = %cst_0) -> (!ttg.memdesc<32x32xbf16, #shared1, #smem, mutable>, tensor<64x32xf32, #mma>)  : i32 {
1132|       %1 = ttng.warp_group_dot %arg2, %arg5, %arg7 {inputPrecision = 0 : i32} : !ttg.memdesc<64x32xbf16, #shared, #smem, mutable> * !ttg.memdesc<32x32xbf16, #shared1, #smem, mutable> -> tensor<64x32xf32, #mma>
1133|       scf.yield %arg5, %1 : !ttg.memdesc<32x32xbf16, #shared1, #smem, mutable>, tensor<64x32xf32, #mma>
1134|     }
1135|     tt.return %0#1 : tensor<64x32xf32, #mma>
1136|   }
1137| }
```
**EN:** This function-oriented block defines or enters `loop_arg_cycle`. Within it, the test exercises constants, tt.func, structured loops, ttng.warp_group_dot, loop/if yielded values, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `loop_arg_cycle` 为核心。测试在其中演示 常量、tt.func、结构化循环、ttng.warp_group_dot、循环/分支产出值，这些操作构成了被优化、lower 或验证的主要数据流。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `-tritongpu-assign-latencies`, `-tritongpu-schedule-loops`, `-tritongpu-pipeline`, `-canonicalize`
- **CN:** 主要 pass 选项：`-split-input-file`，`-tritongpu-assign-latencies`，`-tritongpu-schedule-loops`，`-tritongpu-pipeline`，`-canonicalize`
- **EN:** Dominant operations include `arith.constant`, `tt.addptr`, `tt.broadcast`, `tt.func`, `scf.yield`, `tt.splat`, `tt.expand_dims`, `tt.load`, `tt.make_range`, `ttg.local_alloc`.
- **CN:** 主要操作包括 `arith.constant`、`tt.addptr`、`tt.broadcast`、`tt.func`、`scf.yield`、`tt.splat`、`tt.expand_dims`、`tt.load`、`tt.make_range`、`ttg.local_alloc`。
- **EN:** The file contains 18 independently testable section(s). Check styles used: CHECK x125, CHECK-NEXT x35, CHECK-DAG x21, CHECK-LABEL x18. Important labels include tt.func @matmul_loop, dot_chained_single_load, dot_acc_cond_modified, two_accumulator_escape. It validates both the presence of the desired rewrite and the absence of unwanted residual operations.
- **CN:** 该文件包含 18 个可独立测试的分段。使用的检查类型：CHECK ×125，CHECK-NEXT ×35，CHECK-DAG ×21，CHECK-LABEL ×18。 关键标签包括 tt.func @matmul_loop，dot_chained_single_load，dot_acc_cond_modified，two_accumulator_escape。 它同时验证目标重写已经出现，并且不再残留不需要的操作。

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
- **EN:** `ttng`: Triton Nvidia GPU tensor-memory and MMA operations.
- **CN:** `ttng`：Triton Nvidia GPU 张量内存与 MMA 操作。
- **EN:** `cf`: Control-flow dialect for basic branches.
- **CN:** `cf`：基础分支控制流方言。
- **EN:** `FileCheck` is used to match required textual patterns after rewriting/lowering.
- **CN:** `FileCheck` 用于在重写/lower 之后匹配必须出现的文本模式。