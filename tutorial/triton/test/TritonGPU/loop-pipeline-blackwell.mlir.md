# loop-pipeline-blackwell.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/loop-pipeline-blackwell.mlir`
- **EN:** Test for the MLIR/Triton pipeline `-tritongpu-hoist-tmem-alloc, -tritongpu-assign-latencies, -tritongpu-schedule-loops, -tritongpu-pipeline` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `-tritongpu-hoist-tmem-alloc, -tritongpu-assign-latencies, -tritongpu-schedule-loops, -tritongpu-pipeline` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -split-input-file -tritongpu-hoist-tmem-alloc -tritongpu-assign-latencies -tritongpu-schedule-loops -tritongpu-pipeline -triton-nvidia-gpu-remove-tmem-tokens -canonicalize | FileCheck %s --check-prefixes=CHECK`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -split-input-file -tritongpu-hoist-tmem-alloc -tritongpu-assign-latencies -tritongpu-schedule-loops -tritongpu-pipeline -triton-nvidia-gpu-remove-tmem-tokens -canonicalize | FileCheck %s --check-prefixes=CHECK`

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
1| // RUN: triton-opt %s -split-input-file -tritongpu-hoist-tmem-alloc -tritongpu-assign-latencies -tritongpu-schedule-loops -tritongpu-pipeline -triton-nvidia-gpu-remove-tmem-tokens -canonicalize | FileCheck %s --check-prefixes=CHECK
2| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -split-input-file -tritongpu-hoist-tmem-alloc -tritongpu-assign-latencies -tritongpu-schedule-loops -tritongpu-pipeline -triton-nvidia-gpu-remove-tmem-tokens -canonicalize | FileCheck %s --check-prefixes=CHECK` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -split-input-file -tritongpu-hoist-tmem-alloc -tritongpu-assign-latencies -tritongpu-schedule-loops -tritongpu-pipeline -triton-nvidia-gpu-remove-tmem-tokens -canonicalize | FileCheck %s --check-prefixes=CHECK`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 3-7
```mlir
3| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
4| #blocked1 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
5| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
6| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
7| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 8-8
```mlir
8| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 9-26
```mlir
 9|   // CHECK-LABEL: @chained_dot_scaled_acc
10|   // CHECK-DAG: %[[C0_F:.+]] = arith.constant dense<0.000000e+00>
11|   // CHECK-DAG: %[[TRUE:.+]] = arith.constant true
12|   // CHECK-DAG: %[[C0:.+]] = arith.constant 0 : i32
13|   // CHECK-DAG: %[[C1:.+]] = arith.constant 1 : i32
14|   // CHECK-DAG: %[[C2:.+]] = arith.constant 2 : i32
15|   // CHECK: %[[TMEM_BUF:.+]] = ttng.tmem_alloc : () -> !ttg.memdesc<128x128xf32
16|   // CHECK: ttng.tmem_store %[[C0_F]], %[[TMEM_BUF]], %[[TRUE]]
17|   // CHECK: %[[BAR_BUF:.+]] = ttg.local_alloc : () -> !ttg.memdesc<2x1xi64
18|   // CHECK: %[[ACC1:.+]] = ttng.tmem_load %[[TMEM_BUF]]
19|   // CHECK: %[[ACC2:.+]] = arith.mulf %[[ACC1]]
20|   // CHECK: ttng.tmem_store %[[ACC2]], %[[TMEM_BUF]]
21|   // CHECK: %[[BAR_SLICE:.+]] = ttg.memdesc_index %[[BAR_BUF]]{{\[}}%[[C0]]{{\]}}
22|   // CHECK: ttng.tc_gen5_mma %[[A_OP:.*]], %[[B_OP:.*]], %[[TMEM_BUF]], {{.*}}, %[[BAR_SLICE]]
23|   // CHECK: scf.for {{.*}} iter_args(%[[PHASE:.+]] = %[[C0]], %[[BAR_IDX:.+]] = %[[C1]], {{.*}}, %[[BAR_PREV:.*]] = %[[BAR_SLICE]], %[[PHASE_PREV:.+]] = %[[C0]], %[[A_DEP:.+]] = %[[A_OP]], %[[B_DEP:.+]] = %[[B_OP]]
24|   // CHECK:   ttng.wait_barrier %[[BAR_PREV]], %[[PHASE_PREV]] deps %[[A_DEP]], %[[B_DEP]]
25|   // CHECK:   %[[ACC1:.+]] = ttng.tmem_load %[[TMEM_BUF]]
26|   // CHECK:   %[[ACC2:.+]] = arith.mulf %[[ACC1]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @chained_dot_scaled_acc anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @chained_dot_scaled_acc 这样的标签用于锚定匹配范围。

### Lines 27-37
```mlir
27|   // CHECK:   ttng.tmem_store %[[ACC2]], %[[TMEM_BUF]]
28|   // CHECK:   %[[BAR_SLICE:.+]] = ttg.memdesc_index %[[BAR_BUF]]{{\[}}%[[BAR_IDX]]{{\]}}
29|   // CHECK:   ttng.tc_gen5_mma %[[A_OP:.*]], %[[B_OP:.*]], %[[TMEM_BUF]], %[[TRUE]], {{.*}}, %[[BAR_SLICE]]
30|   // CHECK:   %[[PHASE_NEG:.+]] = arith.xori %[[PHASE]], %[[C1]]
31|   // CHECK:   %[[BAR_IDX_P1:.+]] = arith.addi %[[BAR_IDX]], %[[C1]]
32|   // CHECK:   %[[BAR_IDX_CMP:.+]] = arith.cmpi sge, %[[BAR_IDX_P1]], %[[C2]]
33|   // CHECK:   %[[BAR_IDX_NEXT:.+]] = arith.select %[[BAR_IDX_CMP]], %[[C0]], %[[BAR_IDX_P1]]
34|   // CHECK:   %[[PHASE_NEXT:.+]] = arith.select %[[BAR_IDX_CMP]], %[[PHASE_NEG]], %[[PHASE]]
35|   // CHECK:   scf.yield %[[PHASE_NEXT]], %[[BAR_IDX_NEXT]], {{.*}}, %[[BAR_SLICE]], %[[PHASE]], %[[A_OP]], %[[B_OP]]
36|   // CHECK: ttg.local_dealloc %[[BAR_BUF]]
37|   // CHECK: ttng.tmem_load %[[TMEM_BUF]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 38-55
```mlir
38|   tt.func public @chained_dot_scaled_acc(%A_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1> {tt.contiguity = dense<[1, 16]> : tensor<2xi32>, tt.divisibility = dense<[16, 16]> : tensor<2xi32>}, %B_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1> {tt.contiguity = dense<[1, 16]> : tensor<2xi32>, tt.divisibility = dense<[16, 16]> : tensor<2xi32>}, %arg3: i32) -> tensor<128x128xf16, #blocked> {
39|     %true = arith.constant true
40|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked>
41|     %cst2 = arith.constant dense<2.000000e+00> : tensor<128x128xf32, #blocked>
42|     %c0_i32 = arith.constant 0 : i32
43|     %c1_i32 = arith.constant 1 : i32
44|     %res = scf.for %i = %c0_i32 to %arg3 step %c1_i32 iter_args(%acc = %cst) -> (tensor<128x128xf32, #blocked>)  : i32 {
45|       %A = tt.load %A_ptr : tensor<128x128x!tt.ptr<f16>, #blocked1>
46|       %A_sh = ttg.local_alloc %A : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>
47|       %B = tt.load %B_ptr : tensor<128x128x!tt.ptr<f16>, #blocked1>
48|       %B_sh = ttg.local_alloc %B : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>
49|       %sacc = arith.mulf %acc, %cst2 : tensor<128x128xf32, #blocked>
50|       %acc_tm, %acc_tok = ttng.tmem_alloc %sacc : (tensor<128x128xf32, #blocked>) -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
51|       %mma_tok = ttng.tc_gen5_mma %A_sh, %B_sh, %acc_tm[%acc_tok], %true, %true : !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>, !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
52|       %acc_res, %load_tok = ttng.tmem_load %acc_tm[%mma_tok] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
53|       scf.yield %acc_res : tensor<128x128xf32, #blocked>
54|     }
55|     %res_f16 = arith.truncf %res : tensor<128x128xf32, #blocked> to tensor<128x128xf16, #blocked>
```
**EN:** This function-oriented block defines or enters `chained_dot_scaled_acc`. Within it, the test exercises constants, tt.func, masked or vectorized loads, shared/local memory allocation, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `chained_dot_scaled_acc` 为核心。测试在其中演示 常量、tt.func、带掩码或向量化的加载、共享/本地内存分配、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 56-59
```mlir
56|     tt.return %res_f16 : tensor<128x128xf16, #blocked>
57|   }
58| }
59| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 60-60
```mlir
60| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 61-66
```mlir
61| 
62| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
63| #blocked1 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
64| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
65| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
66| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 67-67
```mlir
67| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 68-74
```mlir
68|   // CHECK-LABEL: @chained_scale_after_dot
69|   // CHECK: ttng.tmem_alloc
70|   // CHECK: scf.for
71|   // CHECK:   ttng.tc_gen5_mma
72|   // CHECK:   ttng.tmem_load
73|   // CHECK:   arith.mulf
74|   // CHECK:   ttng.tmem_store
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @chained_scale_after_dot anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @chained_scale_after_dot 这样的标签用于锚定匹配范围。

### Lines 75-92
```mlir
75|   tt.func public @chained_scale_after_dot(%A_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1>, %B_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1>, %arg3: i32) -> tensor<128x128xf16, #blocked> {
76|     %true = arith.constant true
77|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked>
78|     %cst2 = arith.constant dense<2.000000e+00> : tensor<128x128xf32, #blocked>
79|     %c0_i32 = arith.constant 0 : i32
80|     %c1_i32 = arith.constant 1 : i32
81|     %res = scf.for %i = %c0_i32 to %arg3 step %c1_i32 iter_args(%acc = %cst) -> (tensor<128x128xf32, #blocked>)  : i32 {
82|       %A = tt.load %A_ptr : tensor<128x128x!tt.ptr<f16>, #blocked1>
83|       %A_sh = ttg.local_alloc %A : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>
84|       %B = tt.load %B_ptr : tensor<128x128x!tt.ptr<f16>, #blocked1>
85|       %B_sh = ttg.local_alloc %B : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>
86|       %acc_tm, %acc_tok = ttng.tmem_alloc %acc : (tensor<128x128xf32, #blocked>) -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
87|       %mma_tok = ttng.tc_gen5_mma %A_sh, %B_sh, %acc_tm[%acc_tok], %true, %true : !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>, !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
88|       %acc_res, %load_tok = ttng.tmem_load %acc_tm[%mma_tok] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
89|       %sacc = arith.mulf %acc_res, %cst2 : tensor<128x128xf32, #blocked>
90|       scf.yield %sacc : tensor<128x128xf32, #blocked>
91|     }
92|     %res_f16 = arith.truncf %res : tensor<128x128xf32, #blocked> to tensor<128x128xf16, #blocked>
```
**EN:** This function-oriented block defines or enters `chained_scale_after_dot`. Within it, the test exercises constants, tt.func, masked or vectorized loads, shared/local memory allocation, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `chained_scale_after_dot` 为核心。测试在其中演示 常量、tt.func、带掩码或向量化的加载、共享/本地内存分配、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 93-96
```mlir
93|     tt.return %res_f16 : tensor<128x128xf16, #blocked>
94|   }
95| }
96| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 97-97
```mlir
97| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 98-99
```mlir
98| // 4 warps
99| // matmul: 128x32 @ 32x128 -> 128x128
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 100-111
```mlir
100| #AL = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
101| #BL = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
102| #ALs0 = #ttg.slice<{parent=#AL, dim=0}>
103| #BLs0 = #ttg.slice<{parent=#BL, dim=0}>
104| #BLs1 = #ttg.slice<{parent=#BL, dim=1}>
105| #C = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
106| #A = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [1, 32], warpsPerCTA = [2, 2], order = [1, 0]}>
107| #B = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [1, 32], warpsPerCTA = [2, 2], order = [1, 0]}>
108| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = false, elementBitWidth = 16}>
109| #shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = true, elementBitWidth = 16}>
110| #smem = #ttg.shared_memory
111| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 112-112
```mlir
112| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 113-115
```mlir
113|   tt.func @matmul_loop_cast_load(%lb : index, %ub : index, %step : index,
114|                     %A : !tt.ptr<f8E4M3FN> {tt.divisibility = 16 : i32},
115|                     %B : !tt.ptr<f8E4M3FN> {tt.divisibility = 16 : i32}) -> tensor<128x128xf32, #C> {
```
**EN:** This function-oriented block defines or enters `matmul_loop_cast_load`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `matmul_loop_cast_load` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 116-133
```mlir
116| // CHECK-LABEL: tt.func @matmul_loop_cast_load
117| // CHECK: scf.for
118| // CHECK: ttg.local_load
119| // CHECK: tt.fp_to_fp
120| // CHECK: ttng.wait_barrier
121| // CHECK: ttg.local_store
122| // CHECK: ttg.memdesc_trans
123| // CHECK: ttng.tc_gen5_mma {{.*}}, {{.*}}, {{.*}}, {{.*}}, {{.*}}
124| // CHECK: ttg.async_copy_global_to_local
125|     %a_ptr_splat = tt.splat %A : !tt.ptr<f8E4M3FN> -> tensor<128x32x!tt.ptr<f8E4M3FN>, #AL>
126|     %a_tmp0 = tt.make_range {end = 32: i32, start = 0: i32} : tensor<32xi32, #ALs0>
127|     %a_tmp1 = tt.expand_dims %a_tmp0 {axis = 0 : i32} : tensor<32xi32, #ALs0> -> tensor<1x32xi32, #AL>
128|     %a_offs = tt.broadcast %a_tmp1 : tensor<1x32xi32, #AL> -> tensor<128x32xi32, #AL>
129|     %a_ptr_init = tt.addptr %a_ptr_splat, %a_offs : tensor<128x32x!tt.ptr<f8E4M3FN>, #AL>, tensor<128x32xi32, #AL>
130| 
131|     %b_ptr_splat = tt.splat %B : !tt.ptr<f8E4M3FN> -> tensor<128x32x!tt.ptr<f8E4M3FN>, #BL>
132|     %b_tmp0 = tt.make_range {end = 32: i32, start = 0: i32} : tensor<32xi32, #BLs0>
133|     %b_tmp1 = tt.expand_dims %b_tmp0 {axis = 0 : i32} : tensor<32xi32, #BLs0> -> tensor<1x32xi32, #BL>
```
**EN:** This function-oriented block defines or enters `matmul_loop_cast_load`. Within it, the test exercises broadcasted scalars or pointers, lane/block index ranges, shape expansion, tensor broadcasting, pointer arithmetic, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `matmul_loop_cast_load` 为核心。测试在其中演示 广播后的标量或指针、lane/block 索引范围、形状扩展、张量广播、指针算术，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 134-151
```mlir
134|     %b_offs = tt.broadcast %b_tmp1 : tensor<1x32xi32, #BL> -> tensor<128x32xi32, #BL>
135|     %b_ptr_init = tt.addptr %b_ptr_splat, %b_offs : tensor<128x32x!tt.ptr<f8E4M3FN>, #BL>, tensor<128x32xi32, #BL>
136| 
137|     %true = arith.constant true
138|     %b_mask = arith.constant dense<true> : tensor<128x32xi1, #BL>
139|     %b_other = arith.constant dense<0.00e+00> : tensor<128x32xf8E4M3FN, #BL>
140|     %c_init = arith.constant dense<0.00e+00> : tensor<128x128xf32, #C>
141| 
142|     %a_off = arith.constant dense<4> : tensor<128x32xi32, #AL>
143|     %b_off = arith.constant dense<4> : tensor<128x32xi32, #BL>
144| 
145|     %loop:3 = scf.for %iv = %lb to %ub step %step iter_args(%a_ptr = %a_ptr_init, %b_ptr = %b_ptr_init, %prev_c = %c_init) -> (tensor<128x32x!tt.ptr<f8E4M3FN>, #AL>, tensor<128x32x!tt.ptr<f8E4M3FN>, #BL>, tensor<128x128xf32, #C>) {
146|       %a___ = tt.load %a_ptr : tensor<128x32x!tt.ptr<f8E4M3FN>, #AL>
147|       %a__ = tt.fp_to_fp %a___ : tensor<128x32xf8E4M3FN, #AL> -> tensor<128x32xf16, #AL>
148|       %a_ = ttg.convert_layout %a__ : tensor<128x32xf16, #AL> -> tensor<128x32xf16, #A>
149|       %b___ = tt.load %b_ptr, %b_mask, %b_other : tensor<128x32x!tt.ptr<f8E4M3FN>, #BL>
150|       %b__ = tt.fp_to_fp %b___ : tensor<128x32xf8E4M3FN, #BL> -> tensor<128x32xf16, #BL>
151|       %b_ = ttg.convert_layout %b__ : tensor<128x32xf16, #BL> -> tensor<128x32xf16, #B>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining constants, masked or vectorized loads, tt.fp_to_fp, layout conversions, tensor broadcasting.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 常量、带掩码或向量化的加载、tt.fp_to_fp、布局转换、张量广播。

### Lines 152-167
```mlir
152| 
153|       %a = ttg.local_alloc %a_ {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x32xf16, #A>) -> !ttg.memdesc<128x32xf16, #shared, #smem>
154|       %b = ttg.local_alloc %b_ {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x32xf16, #B>) -> !ttg.memdesc<128x32xf16, #shared, #smem>
155|       %bt = ttg.memdesc_trans %b {loop.cluster = 1 : i32, loop.stage = 2 : i32, order = array<i32: 1, 0>} : !ttg.memdesc<128x32xf16, #shared, #smem> -> !ttg.memdesc<32x128xf16, #shared1, #smem>
156|       %acc_tm, %acc_tok = ttng.tmem_alloc %prev_c : (tensor<128x128xf32, #C>) -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
157|       %mma_tok = ttng.tc_gen5_mma %a, %bt, %acc_tm[%acc_tok], %true, %true : !ttg.memdesc<128x32xf16, #shared, #smem>, !ttg.memdesc<32x128xf16, #shared1, #smem>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
158|       %c, %load_tok = ttng.tmem_load %acc_tm[%mma_tok] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #C>
159| 
160|       %next_a_ptr = tt.addptr %a_ptr, %a_off : tensor<128x32x!tt.ptr<f8E4M3FN>, #AL>, tensor<128x32xi32, #AL>
161|       %next_b_ptr = tt.addptr %b_ptr, %b_off : tensor<128x32x!tt.ptr<f8E4M3FN>, #BL>, tensor<128x32xi32, #BL>
162|       scf.yield %next_a_ptr, %next_b_ptr, %c : tensor<128x32x!tt.ptr<f8E4M3FN>, #AL>, tensor<128x32x!tt.ptr<f8E4M3FN>, #BL>, tensor<128x128xf32, #C>
163|     }
164|     tt.return %loop#2: tensor<128x128xf32, #C>
165|   }
166| }
167| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining shared/local memory allocation, pointer arithmetic, ttg.memdesc_trans, tensor-memory allocation, Gen5 tensor-core MMA ops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 共享/本地内存分配、指针算术、ttg.memdesc_trans、张量内存分配、Gen5 张量核 MMA 操作。

### Lines 168-168
```mlir
168| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 169-178
```mlir
169| 
170| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
171| #blocked1_parent = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [32, 1], warpsPerCTA = [1, 4], order = [1, 0]}>
172| #blocked1 = #ttg.slice<{dim = 0, parent = #blocked1_parent}>
173| #blocked2 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [1, 0]}>
174| #blocked3 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
175| #mma = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [2, 2], instrShape = [16, 8]}>
176| #nvmma_64 = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = false, elementBitWidth = 16}>
177| #nvmma_128 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
178| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 179-180
```mlir
179| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100"} {
180| 
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 181-185
```mlir
181| // CHECK-LABEL: @pipelined_gather
182| // CHECK-SAME: [[LHS_DESC:%arg[0-9]+]]:
183| // CHECK-SAME: [[RHS_DESC:%arg[0-9]+]]:
184| // CHECK-SAME: [[LHS_X:%arg[0-9]+]]:
185| // CHECK-SAME: [[RHS_X:%arg[0-9]+]]:
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @pipelined_gather anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @pipelined_gather 这样的标签用于锚定匹配范围。

### Lines 186-196
```mlir
186| tt.func private @pipelined_gather(
187|     %lhs_desc: !tt.tensordesc<1x128xbf16, #nvmma_128>,
188|     %rhs_desc: !tt.tensordesc<1x32xbf16, #nvmma_64>,
189|     %lhs_x_offsets: tensor<32xi32, #blocked1>,
190|     %rhs_x_offsets: tensor<128xi32, #blocked1>) -> tensor<32x32xf32, #blocked> {
191|   %c0_i32 = arith.constant 0 : i32
192|   %c128_i32 = arith.constant 128 : i32
193|   %c1024_i32 = arith.constant 1024 : i32
194| 
195|   %c0 = arith.constant dense<0.000000e+00> : tensor<32x32xf32, #mma>
196| 
```
**EN:** This function-oriented block defines or enters `pipelined_gather`. Within it, the test exercises constants, tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `pipelined_gather` 为核心。测试在其中演示 常量、tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 197-214
```mlir
197|   // CHECK: [[LHS_BUFS:%.*]] = ttg.local_alloc : () -> !ttg.memdesc<2x32x128xbf16,
198|   // CHECK: [[RHS_BUFS:%.*]] = ttg.local_alloc : () -> !ttg.memdesc<2x128x32xbf16,
199|   // CHECK: [[BARS:%.*]] = ttg.local_alloc : () -> !ttg.memdesc<2x1xi64,
200| 
201|   // CHECK-COUNT-2: ttng.init_barrier
202| 
203|   // CHECK: [[BAR0:%.*]] = ttg.memdesc_index [[BARS]]{{\[}}%c0_i32{{\]}}
204|   // CHECK: ttng.barrier_expect [[BAR0]], 16384
205|   // CHECK: [[LHS_BUF0:%.*]] = ttg.memdesc_index [[LHS_BUFS]]{{\[}}%c0_i32{{\]}}
206|   // CHECK: ttng.async_tma_gather [[LHS_DESC]][[[LHS_X]], %c0_i32] [[LHS_BUF0]], [[BAR0]], %true
207|   // CHECK: [[RHS_BUF0:%.*]] = ttg.memdesc_index [[RHS_BUFS]]{{\[}}%c0_i32{{\]}}
208|   // CHECK: ttng.async_tma_gather [[RHS_DESC]][[[RHS_X]], %c0_i32] [[RHS_BUF0]], [[BAR0]], %true
209| 
210|   // CHECK: [[BAR1:%.*]] = ttg.memdesc_index [[BARS]]{{\[}}%c1_i32{{\]}}
211|   // CHECK: ttng.barrier_expect [[BAR1]], 16384
212|   // CHECK: [[LHS_BUF1:%.*]] = ttg.memdesc_index [[LHS_BUFS]]{{\[}}%c1_i32{{\]}}
213|   // CHECK: ttng.async_tma_gather [[LHS_DESC]][[[LHS_X]], %c128_i32] [[LHS_BUF1]], [[BAR1]], %true
214|   // CHECK: [[RHS_BUF1:%.*]] = ttg.memdesc_index [[RHS_BUFS]]{{\[}}%c1_i32{{\]}}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 215-218
```mlir
215|   // CHECK: ttng.async_tma_gather [[RHS_DESC]][[[RHS_X]], %c128_i32] [[RHS_BUF1]], [[BAR1]], %true
216| 
217|   // CHECK: scf.for
218|   %out = scf.for %y = %c0_i32 to %c1024_i32 step %c128_i32 iter_args(%acc = %c0) -> (tensor<32x32xf32, #mma>)  : i32 {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环。

### Lines 219-231
```mlir
219|     // CHECK: ttng.wait_barrier
220|     // CHECK: [[RHS_VIEW:%.*]] = ttg.memdesc_index [[RHS_BUFS]]
221|     // CHECK: [[RHS:%.*]] = ttg.local_load [[RHS_VIEW]]
222|     // CHECK: [[LHS_VIEW:%.*]] = ttg.memdesc_index [[LHS_BUFS]]
223|     // CHECK: [[LHS:%.*]] = ttg.local_load [[LHS_VIEW]]
224|     // CHECK: tt.dot [[LHS]], [[RHS]]
225|     %lhs = tt.descriptor_gather %lhs_desc[%lhs_x_offsets, %y] : (!tt.tensordesc<1x128xbf16, #nvmma_128>, tensor<32xi32, #blocked1>, i32) -> tensor<32x128xbf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
226|     %rhs = tt.descriptor_gather %rhs_desc[%rhs_x_offsets, %y] : (!tt.tensordesc<1x32xbf16, #nvmma_64>, tensor<128xi32, #blocked1>, i32) -> tensor<128x32xbf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>
227|     %next = tt.dot %lhs, %rhs, %acc : tensor<32x128xbf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> *
228|                                       tensor<128x32xbf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>
229|                                    -> tensor<32x32xf32, #mma>
230| 
231| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.descriptor_gather, dot-product or MMA-style math. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.descriptor_gather、点积或 MMA 风格计算。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 232-240
```mlir
232|     // CHECK-COUNT-2: async_tma_gather
233|     scf.yield %next : tensor<32x32xf32, #mma>
234|   }
235|   %out_cvt = ttg.convert_layout %out : tensor<32x32xf32, #mma> -> tensor<32x32xf32, #blocked>
236|   tt.return %out_cvt : tensor<32x32xf32, #blocked>
237| }
238| 
239| }
240| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, layout conversions, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、布局转换、tt.return。

### Lines 241-241
```mlir
241| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 242-254
```mlir
242| 
243| #blocked = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
244| #blocked1 = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
245| #blocked2 = #ttg.blocked<{sizePerThread = [1, 1, 1, 1, 4], threadsPerWarp = [1, 1, 8, 4, 1], warpsPerCTA = [1, 1, 4, 1, 1], order = [4, 3, 2, 1, 0]}>
246| #blocked3 = #ttg.blocked<{sizePerThread = [1, 1, 1, 1, 4], threadsPerWarp = [1, 4, 8, 1, 1], warpsPerCTA = [1, 1, 4, 1, 1], order = [4, 1, 2, 3, 0]}>
247| #blocked4 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
248| #linear = #ttg.linear<{register = [[0, 1], [0, 2], [0, 4]], lane = [[32, 0], [64, 0], [1, 0], [2, 0], [4, 0]], warp = [[8, 0], [16, 0]], block = []}>
249| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 8}>
250| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [4, 3, 2, 1, 0]}>
251| #scales = #ttg.linear<{register = [[0, 1], [0, 2], [32, 0], [64, 0], [0, 4]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [16, 0]], warp = [[0, 0], [0, 0]], block = []}>
252| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
253| #smem = #ttg.shared_memory
254| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 255-255
```mlir
255| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 256-256
```mlir
256|   tt.func public @block_scale_mxfp_matmul(%lb : index, %ub : index, %step : index, %arg0: !tt.ptr<f8E5M2> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f8E5M2> {tt.divisibility = 16 : i32}, %arg3: !tt.ptr<i8> {tt.divisibility = 16 : i32}, %arg4: !tt.ptr<i8> {tt.divisibility = 16 : i32}) -> tensor<128x128xf32, #blocked4> {
```
**EN:** This function-oriented block defines or enters `block_scale_mxfp_matmul`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `block_scale_mxfp_matmul` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 257-259
```mlir
257|     // CHECK: ttg.local_alloc : () -> !ttg.memdesc<2x128x256xf8E5M2
258|     // CHECK: ttg.local_alloc : () -> !ttg.memdesc<2x256x128xf8E5M2
259|     // Do not multibuffer the scale loads, as we cannot pipeline the mma due to tmem.cp not being used
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 260-277
```mlir
260|     // CHECK: ttg.local_alloc : () -> !ttg.memdesc<2x1x2x32x4x4xi8
261|     // CHECK: ttg.local_alloc : () -> !ttg.memdesc<2x1x2x32x4x4xi8
262| 
263|     %true = arith.constant true
264|     %cst_1 = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked4>
265|     %incr_A = arith.constant dense<4> : tensor<128x256xi32, #blocked>
266|     %incr_B = arith.constant dense<4> : tensor<256x128xi32, #blocked1>
267|     %incr_scale = arith.constant dense<4> : tensor<1x2x32x4x4xi32, #blocked2>
268| 
269|     %arg0_splat = tt.splat %arg0: !tt.ptr<f8E5M2> -> tensor<128x256x!tt.ptr<f8E5M2>, #blocked>
270|     %arg1_splat = tt.splat %arg1: !tt.ptr<f8E5M2> -> tensor<256x128x!tt.ptr<f8E5M2>, #blocked1>
271|     %arg3_splat = tt.splat %arg3: !tt.ptr<i8> -> tensor<1x2x32x4x4x!tt.ptr<i8>, #blocked2>
272|     %arg4_splat = tt.splat %arg4: !tt.ptr<i8> -> tensor<1x2x32x4x4x!tt.ptr<i8>, #blocked2>
273| 
274|     %76 = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
275|     %77 = tt.expand_dims %76 {axis = 0 : i32} : tensor<256xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x256xi32, #blocked>
276|     %79 = tt.broadcast %77 : tensor<1x256xi32, #blocked> -> tensor<128x256xi32, #blocked>
277|     %arg0_init = tt.addptr %arg0_splat, %79 : tensor<128x256x!tt.ptr<f8E5M2>, #blocked>, tensor<128x256xi32, #blocked>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on constants, broadcasted scalars or pointers, lane/block index ranges, shape expansion, tensor broadcasting. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 常量、广播后的标量或指针、lane/block 索引范围、形状扩展、张量广播。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 278-295
```mlir
278| 
279|     %83 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>
280|     %84 = tt.expand_dims %83 {axis = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked1}>> -> tensor<1x128xi32, #blocked1>
281|     %88 = tt.broadcast %84 : tensor<1x128xi32, #blocked1> -> tensor<256x128xi32, #blocked1>
282|     %arg1_init = tt.addptr %arg1_splat, %88 : tensor<256x128x!tt.ptr<f8E5M2>, #blocked1>, tensor<256x128xi32, #blocked1>
283| 
284|     %44 = tt.make_range {end = 4 : i32, start = 0 : i32} : tensor<4xi32, #ttg.slice<{dim = 0, parent = #ttg.slice<{dim = 1, parent = #ttg.slice<{dim = 2, parent = #ttg.slice<{dim = 3, parent = #blocked2}>}>}>}>>
285|     %46 = tt.expand_dims %44 {axis = 0 : i32} : tensor<4xi32, #ttg.slice<{dim = 0, parent = #ttg.slice<{dim = 1, parent = #ttg.slice<{dim = 2, parent = #ttg.slice<{dim = 3, parent = #blocked2}>}>}>}>> -> tensor<1x4xi32, #ttg.slice<{dim = 1, parent = #ttg.slice<{dim = 2, parent = #ttg.slice<{dim = 3, parent = #blocked2}>}>}>>
286|     %48 = tt.expand_dims %46 {axis = 1 : i32} : tensor<1x4xi32, #ttg.slice<{dim = 1, parent = #ttg.slice<{dim = 2, parent = #ttg.slice<{dim = 3, parent = #blocked2}>}>}>> -> tensor<1x1x4xi32, #ttg.slice<{dim = 2, parent = #ttg.slice<{dim = 3, parent = #blocked2}>}>>
287|     %50 = tt.expand_dims %48 {axis = 2 : i32} : tensor<1x1x4xi32, #ttg.slice<{dim = 2, parent = #ttg.slice<{dim = 3, parent = #blocked2}>}>> -> tensor<1x1x1x4xi32, #ttg.slice<{dim = 3, parent = #ttg.blocked<{sizePerThread = [1, 1, 1, 1, 4], threadsPerWarp = [1, 1, 8, 4, 1], warpsPerCTA = [1, 1, 4, 1, 1], order = [4, 3, 2, 1, 0]}>}>>
288|     %56 = tt.expand_dims %50 {axis = 3 : i32} : tensor<1x1x1x4xi32, #ttg.slice<{dim = 3, parent = #blocked2}>> -> tensor<1x1x1x1x4xi32, #blocked2>
289|     %57 = tt.broadcast %56 : tensor<1x1x1x1x4xi32, #blocked2> -> tensor<1x2x32x4x4xi32, #blocked2>
290| 
291|     %arg3_init = tt.addptr %arg3_splat, %57 : tensor<1x2x32x4x4x!tt.ptr<i8>, #blocked2>, tensor<1x2x32x4x4xi32, #blocked2>
292|     %arg4_init = tt.addptr %arg4_splat, %57 : tensor<1x2x32x4x4x!tt.ptr<i8>, #blocked2>, tensor<1x2x32x4x4xi32, #blocked2>
293| 
294|     %99:5 = scf.for %iv = %lb to %ub step %step iter_args(%arg15 = %cst_1, %arg16 = %arg0_init, %arg17 = %arg1_init, %arg18 = %arg3_init, %arg19 = %arg4_init) -> (tensor<128x128xf32, #blocked4>, tensor<128x256x!tt.ptr<f8E5M2>, #blocked>, tensor<256x128x!tt.ptr<f8E5M2>, #blocked1>, tensor<1x2x32x4x4x!tt.ptr<i8>, #blocked2>, tensor<1x2x32x4x4x!tt.ptr<i8>, #blocked2>) {
295|       %117 = tt.load %arg16 : tensor<128x256x!tt.ptr<f8E5M2>, #blocked>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining shape expansion, pointer arithmetic, lane/block index ranges, tensor broadcasting, structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 形状扩展、指针算术、lane/block 索引范围、张量广播、结构化循环。

### Lines 296-313
```mlir
296|       %118 = ttg.local_alloc %117 : (tensor<128x256xf8E5M2, #blocked>) -> !ttg.memdesc<128x256xf8E5M2, #shared, #ttg.shared_memory>
297|       %119 = tt.load %arg17 : tensor<256x128x!tt.ptr<f8E5M2>, #blocked1>
298|       %120 = ttg.local_alloc %119 : (tensor<256x128xf8E5M2, #blocked1>) -> !ttg.memdesc<256x128xf8E5M2, #shared, #ttg.shared_memory>
299|       %121 = tt.load %arg18 : tensor<1x2x32x4x4x!tt.ptr<i8>, #blocked2>
300|       %122 = tt.load %arg19 : tensor<1x2x32x4x4x!tt.ptr<i8>, #blocked2>
301| 
302|       %137 = ttg.local_alloc %121 : (tensor<1x2x32x4x4xi8, #blocked2>) -> !ttg.memdesc<1x2x32x4x4xi8, #shared1, #smem>
303|       %138 = ttg.local_load %137 : !ttg.memdesc<1x2x32x4x4xi8, #shared1, #smem> -> tensor<1x2x32x4x4xi8, #blocked2>
304|       %123 = tt.trans %138 {order = array<i32: 0, 3, 2, 1, 4>} : tensor<1x2x32x4x4xi8, #blocked2> -> tensor<1x4x32x2x4xi8, #blocked3>
305|       %124 = tt.reshape %123 : tensor<1x4x32x2x4xi8, #blocked3> -> tensor<128x8xi8, #linear>
306| 
307|       %139 = ttg.local_alloc %122 : (tensor<1x2x32x4x4xi8, #blocked2>) -> !ttg.memdesc<1x2x32x4x4xi8, #shared1, #smem>
308|       %140 = ttg.local_load %139 : !ttg.memdesc<1x2x32x4x4xi8, #shared1, #smem> -> tensor<1x2x32x4x4xi8, #blocked2>
309|       %125 = tt.trans %140 {order = array<i32: 0, 3, 2, 1, 4>} : tensor<1x2x32x4x4xi8, #blocked2> -> tensor<1x4x32x2x4xi8, #blocked3>
310|       %126 = tt.reshape %125 : tensor<1x4x32x2x4xi8, #blocked3> -> tensor<128x8xi8, #linear>
311| 
312|       %127, %acc_tok = ttng.tmem_alloc %arg15 : (tensor<128x128xf32, #blocked4>) -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
313|       %128 = ttg.convert_layout %124 : tensor<128x8xi8, #linear> -> tensor<128x8xi8, #scales>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, masked or vectorized loads, local/shared memory loads, transpose-like layout changes, tensor reshaping. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、带掩码或向量化的加载、本地/共享内存加载、转置类布局变换、张量重塑。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 314-329
```mlir
314|       %129 = ttg.convert_layout %126 : tensor<128x8xi8, #linear> -> tensor<128x8xi8, #scales>
315|       %130 = ttng.tmem_alloc %128 : (tensor<128x8xi8, #scales>) -> !ttg.memdesc<128x8xi8, #ttng.tensor_memory_scales_encoding<>, #ttng.tensor_memory>
316|       %131 = ttng.tmem_alloc %129 : (tensor<128x8xi8, #scales>) -> !ttg.memdesc<128x8xi8, #ttng.tensor_memory_scales_encoding<>, #ttng.tensor_memory>
317|       %mma_tok = ttng.tc_gen5_mma_scaled %118, %120, %127[%acc_tok], %130, %131, %true, %true lhs = e5m2 rhs = e5m2 : !ttg.memdesc<128x256xf8E5M2, #shared, #ttg.shared_memory>, !ttg.memdesc<256x128xf8E5M2, #shared, #ttg.shared_memory>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.memdesc<128x8xi8, #ttng.tensor_memory_scales_encoding<>, #ttng.tensor_memory>, !ttg.memdesc<128x8xi8, #ttng.tensor_memory_scales_encoding<>, #ttng.tensor_memory>
318|       %132, %load_tok = ttng.tmem_load %127[%mma_tok] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked4>
319| 
320|       %133 = tt.addptr %arg16, %incr_A : tensor<128x256x!tt.ptr<f8E5M2>, #blocked>, tensor<128x256xi32, #blocked>
321|       %134 = tt.addptr %arg17, %incr_B : tensor<256x128x!tt.ptr<f8E5M2>, #blocked1>, tensor<256x128xi32, #blocked1>
322|       %135 = tt.addptr %arg18, %incr_scale : tensor<1x2x32x4x4x!tt.ptr<i8>, #blocked2>, tensor<1x2x32x4x4xi32, #blocked2>
323|       %136 = tt.addptr %arg19, %incr_scale : tensor<1x2x32x4x4x!tt.ptr<i8>, #blocked2>, tensor<1x2x32x4x4xi32, #blocked2>
324|       scf.yield %132, %133, %134, %135, %136 : tensor<128x128xf32, #blocked4>, tensor<128x256x!tt.ptr<f8E5M2>, #blocked>, tensor<256x128x!tt.ptr<f8E5M2>, #blocked1>, tensor<1x2x32x4x4x!tt.ptr<i8>, #blocked2>, tensor<1x2x32x4x4x!tt.ptr<i8>, #blocked2>
325|     } {tt.num_stages = 3 : i32}
326|      tt.return %99#0 : tensor<128x128xf32, #blocked4>
327|   }
328| }
329| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining pointer arithmetic, tensor-memory allocation, layout conversions, scaled tensor-core MMA ops, ttng.tmem_load.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 指针算术、张量内存分配、布局转换、带缩放的张量核 MMA 操作、ttng.tmem_load。

### Lines 330-330
```mlir
330| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 331-342
```mlir
331| 
332| #blocked = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
333| #blocked1 = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
334| #blocked2 = #ttg.blocked<{sizePerThread = [1, 1, 1, 1, 4], threadsPerWarp = [1, 1, 8, 4, 1], warpsPerCTA = [1, 1, 4, 1, 1], order = [4, 3, 2, 1, 0]}>
335| #blocked4 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
336| #linear = #ttg.linear<{register = [[0, 1], [0, 2], [0, 4]], lane = [[32, 0], [64, 0], [1, 0], [2, 0], [4, 0]], warp = [[8, 0], [16, 0]], block = []}>
337| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 8}>
338| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [4, 3, 2, 1, 0]}>
339| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
340| #tmem_scales = #ttng.tensor_memory_scales_encoding<>
341| #smem = #ttg.shared_memory
342| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 343-343
```mlir
343| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 344-344
```mlir
344|   tt.func public @block_scale_mxfp_matmul_tmem_copy(%lb : index, %ub : index, %step : index, %arg0: !tt.ptr<f8E5M2> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f8E5M2> {tt.divisibility = 16 : i32}, %arg3: !tt.ptr<i8> {tt.divisibility = 16 : i32}, %arg4: !tt.ptr<i8> {tt.divisibility = 16 : i32}) -> tensor<128x128xf32, #blocked4> {
```
**EN:** This function-oriented block defines or enters `block_scale_mxfp_matmul_tmem_copy`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `block_scale_mxfp_matmul_tmem_copy` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 345-362
```mlir
345|     // CHECK: ttg.local_alloc : () -> !ttg.memdesc<3x128x256xf8E5M2
346|     // CHECK: ttg.local_alloc : () -> !ttg.memdesc<3x256x128xf8E5M2
347|     // CHECK: ttg.local_alloc : () -> !ttg.memdesc<3x1x2x32x4x4xi8
348|     // CHECK: ttg.local_alloc : () -> !ttg.memdesc<3x1x2x32x4x4xi8
349|     %false = arith.constant false
350|     %true = arith.constant true
351|     %cst_1 = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked4>
352|     %incr_A = arith.constant dense<4> : tensor<128x256xi32, #blocked>
353|     %incr_B = arith.constant dense<4> : tensor<256x128xi32, #blocked1>
354|     %incr_scale = arith.constant dense<4> : tensor<1x2x32x4x4xi32, #blocked2>
355| 
356|     %arg0_splat = tt.splat %arg0: !tt.ptr<f8E5M2> -> tensor<128x256x!tt.ptr<f8E5M2>, #blocked>
357|     %arg1_splat = tt.splat %arg1: !tt.ptr<f8E5M2> -> tensor<256x128x!tt.ptr<f8E5M2>, #blocked1>
358|     %arg3_splat = tt.splat %arg3: !tt.ptr<i8> -> tensor<1x2x32x4x4x!tt.ptr<i8>, #blocked2>
359|     %arg4_splat = tt.splat %arg4: !tt.ptr<i8> -> tensor<1x2x32x4x4x!tt.ptr<i8>, #blocked2>
360| 
361|     %76 = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
362|     %77 = tt.expand_dims %76 {axis = 0 : i32} : tensor<256xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x256xi32, #blocked>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on constants, broadcasted scalars or pointers, lane/block index ranges, shape expansion. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 常量、广播后的标量或指针、lane/block 索引范围、形状扩展。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 363-380
```mlir
363|     %79 = tt.broadcast %77 : tensor<1x256xi32, #blocked> -> tensor<128x256xi32, #blocked>
364|     %arg0_init = tt.addptr %arg0_splat, %79 : tensor<128x256x!tt.ptr<f8E5M2>, #blocked>, tensor<128x256xi32, #blocked>
365| 
366|     %83 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>
367|     %84 = tt.expand_dims %83 {axis = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked1}>> -> tensor<1x128xi32, #blocked1>
368|     %88 = tt.broadcast %84 : tensor<1x128xi32, #blocked1> -> tensor<256x128xi32, #blocked1>
369|     %arg1_init = tt.addptr %arg1_splat, %88 : tensor<256x128x!tt.ptr<f8E5M2>, #blocked1>, tensor<256x128xi32, #blocked1>
370| 
371|     %44 = tt.make_range {end = 4 : i32, start = 0 : i32} : tensor<4xi32, #ttg.slice<{dim = 0, parent = #ttg.slice<{dim = 1, parent = #ttg.slice<{dim = 2, parent = #ttg.slice<{dim = 3, parent = #blocked2}>}>}>}>>
372|     %46 = tt.expand_dims %44 {axis = 0 : i32} : tensor<4xi32, #ttg.slice<{dim = 0, parent = #ttg.slice<{dim = 1, parent = #ttg.slice<{dim = 2, parent = #ttg.slice<{dim = 3, parent = #blocked2}>}>}>}>> -> tensor<1x4xi32, #ttg.slice<{dim = 1, parent = #ttg.slice<{dim = 2, parent = #ttg.slice<{dim = 3, parent = #blocked2}>}>}>>
373|     %48 = tt.expand_dims %46 {axis = 1 : i32} : tensor<1x4xi32, #ttg.slice<{dim = 1, parent = #ttg.slice<{dim = 2, parent = #ttg.slice<{dim = 3, parent = #blocked2}>}>}>> -> tensor<1x1x4xi32, #ttg.slice<{dim = 2, parent = #ttg.slice<{dim = 3, parent = #blocked2}>}>>
374|     %50 = tt.expand_dims %48 {axis = 2 : i32} : tensor<1x1x4xi32, #ttg.slice<{dim = 2, parent = #ttg.slice<{dim = 3, parent = #blocked2}>}>> -> tensor<1x1x1x4xi32, #ttg.slice<{dim = 3, parent = #ttg.blocked<{sizePerThread = [1, 1, 1, 1, 4], threadsPerWarp = [1, 1, 8, 4, 1], warpsPerCTA = [1, 1, 4, 1, 1], order = [4, 3, 2, 1, 0]}>}>>
375|     %56 = tt.expand_dims %50 {axis = 3 : i32} : tensor<1x1x1x4xi32, #ttg.slice<{dim = 3, parent = #blocked2}>> -> tensor<1x1x1x1x4xi32, #blocked2>
376|     %57 = tt.broadcast %56 : tensor<1x1x1x1x4xi32, #blocked2> -> tensor<1x2x32x4x4xi32, #blocked2>
377| 
378|     %arg3_init = tt.addptr %arg3_splat, %57 : tensor<1x2x32x4x4x!tt.ptr<i8>, #blocked2>, tensor<1x2x32x4x4xi32, #blocked2>
379|     %arg4_init = tt.addptr %arg4_splat, %57 : tensor<1x2x32x4x4x!tt.ptr<i8>, #blocked2>, tensor<1x2x32x4x4xi32, #blocked2>
380| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shape expansion, pointer arithmetic, tensor broadcasting, lane/block index ranges. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 形状扩展、指针算术、张量广播、lane/block 索引范围。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 381-393
```mlir
381|     %99:6 = scf.for %iv = %lb to %ub step %step iter_args(%arg15 = %cst_1, %arg16 = %arg0_init, %arg17 = %arg1_init, %arg18 = %arg3_init, %arg19 = %arg4_init, %init_flag=%false) -> (tensor<128x128xf32, #blocked4>, tensor<128x256x!tt.ptr<f8E5M2>, #blocked>, tensor<256x128x!tt.ptr<f8E5M2>, #blocked1>, tensor<1x2x32x4x4x!tt.ptr<i8>, #blocked2>, tensor<1x2x32x4x4x!tt.ptr<i8>, #blocked2>, i1) {
382|       %117 = tt.load %arg16 : tensor<128x256x!tt.ptr<f8E5M2>, #blocked>
383|       %118 = ttg.local_alloc %117 : (tensor<128x256xf8E5M2, #blocked>) -> !ttg.memdesc<128x256xf8E5M2, #shared, #ttg.shared_memory>
384|       %119 = tt.load %arg17 : tensor<256x128x!tt.ptr<f8E5M2>, #blocked1>
385|       %120 = ttg.local_alloc %119 : (tensor<256x128xf8E5M2, #blocked1>) -> !ttg.memdesc<256x128xf8E5M2, #shared, #ttg.shared_memory>
386|       %121 = tt.load %arg18 : tensor<1x2x32x4x4x!tt.ptr<i8>, #blocked2>
387|       %122 = tt.load %arg19 : tensor<1x2x32x4x4x!tt.ptr<i8>, #blocked2>
388| 
389|       %137 = ttg.local_alloc %121 : (tensor<1x2x32x4x4xi8, #blocked2>) -> !ttg.memdesc<1x2x32x4x4xi8, #shared1, #smem>
390|       %139 = ttg.local_alloc %122 : (tensor<1x2x32x4x4xi8, #blocked2>) -> !ttg.memdesc<1x2x32x4x4xi8, #shared1, #smem>
391| 
392|       %127, %acc_tok = ttng.tmem_alloc %arg15 : (tensor<128x128xf32, #blocked4>) -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
393| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining masked or vectorized loads, shared/local memory allocation, structured loops, tensor-memory allocation.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 带掩码或向量化的加载、共享/本地内存分配、结构化循环、张量内存分配。

### Lines 394-407
```mlir
394|       // CHECK: tc_gen5_mma_scaled %{{.*}}, %{{.*}}, %{{.*}}, %{{.*}}, %{{.*}}, %true, %{{.*}}
395|       %mma_tok = ttng.tc_gen5_mma_scaled %118, %120, %127[%acc_tok], %137, %139, %init_flag, %true lhs = e5m2 rhs = e5m2 : !ttg.memdesc<128x256xf8E5M2, #shared, #ttg.shared_memory>, !ttg.memdesc<256x128xf8E5M2, #shared, #ttg.shared_memory>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.memdesc<1x2x32x4x4xi8, #shared1, #smem>, !ttg.memdesc<1x2x32x4x4xi8, #shared1, #smem>
396|       %132, %load_tok = ttng.tmem_load %127[%mma_tok] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked4>
397| 
398|       %133 = tt.addptr %arg16, %incr_A : tensor<128x256x!tt.ptr<f8E5M2>, #blocked>, tensor<128x256xi32, #blocked>
399|       %134 = tt.addptr %arg17, %incr_B : tensor<256x128x!tt.ptr<f8E5M2>, #blocked1>, tensor<256x128xi32, #blocked1>
400|       %135 = tt.addptr %arg18, %incr_scale : tensor<1x2x32x4x4x!tt.ptr<i8>, #blocked2>, tensor<1x2x32x4x4xi32, #blocked2>
401|       %136 = tt.addptr %arg19, %incr_scale : tensor<1x2x32x4x4x!tt.ptr<i8>, #blocked2>, tensor<1x2x32x4x4xi32, #blocked2>
402|       scf.yield %132, %133, %134, %135, %136, %true : tensor<128x128xf32, #blocked4>, tensor<128x256x!tt.ptr<f8E5M2>, #blocked>, tensor<256x128x!tt.ptr<f8E5M2>, #blocked1>, tensor<1x2x32x4x4x!tt.ptr<i8>, #blocked2>, tensor<1x2x32x4x4x!tt.ptr<i8>, #blocked2>, i1
403|     } {tt.num_stages = 3 : i32}
404|      tt.return %99#0 : tensor<128x128xf32, #blocked4>
405|   }
406| }
407| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining pointer arithmetic, scaled tensor-core MMA ops, ttng.tmem_load, loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 指针算术、带缩放的张量核 MMA 操作、ttng.tmem_load、循环/分支产出值、tt.return。

### Lines 408-408
```mlir
408| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 409-419
```mlir
409| 
410| #blocked = #ttg.blocked<{sizePerThread = [1, 64], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
411| #load_blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
412| #scales = #ttg.linear<{register = [[0, 1], [0, 2], [32, 0], [64, 0], [0, 4]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [16, 0]], warp = [[0, 0], [0, 0]], block = []}>
413| 
414| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
415| #shared_T = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = true, elementBitWidth = 16}>
416| #barrier_shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
417| 
418| #smem = #ttg.shared_memory
419| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 64, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 420-421
```mlir
420| module attributes {"ttg.num-warps" = 4 : i32, ttg.target = "cuda:100"} {
421| 
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 422-422
```mlir
422| // CHECK-LABEL: @load_into_async_mma
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @load_into_async_mma anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @load_into_async_mma 这样的标签用于锚定匹配范围。

### Lines 423-437
```mlir
423| tt.func public @load_into_async_mma(
424|   %lhs_ptrs: tensor<128x64x!tt.ptr<f8E4M3FN>, #load_blocked>,
425|   %scale_ptrs: tensor<128x8x!tt.ptr<i8>, #load_blocked>,
426|   %tmem: !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory, mutable>,
427|   %barrier: !ttg.memdesc<1xi64, #barrier_shared, #smem, mutable>,
428|   %rhs_shared: !ttg.memdesc<64x64xf8E4M3FN, #shared, #smem>,
429|   %n_tiles: i32
430| ) {
431|   %true = arith.constant true
432|   %c0_i32 = arith.constant 0 : i32
433|   %c64_i32 = arith.constant 64 : i32
434| 
435|   %cst = arith.constant dense<0> : tensor<64x8xi8, #scales>
436|   %rhs_scales = ttng.tmem_alloc %cst : (tensor<64x8xi8, #scales>) -> !ttg.memdesc<64x8xi8, #ttng.tensor_memory_scales_encoding<>, #ttng.tensor_memory>
437| 
```
**EN:** This function-oriented block defines or enters `load_into_async_mma`. Within it, the test exercises constants, tt.func, tensor-memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `load_into_async_mma` 为核心。测试在其中演示 常量、tt.func、张量内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 438-455
```mlir
438|   // CHECK-COUNT-6: ttg.async_copy_global_to_local
439|   scf.for %i = %c0_i32 to %n_tiles step %c64_i32 : i32 {
440|     %lhs_offs = tt.splat %i : i32 -> tensor<128x64xi32, #load_blocked>
441|     %lhs_ptrs_i = tt.addptr %lhs_ptrs, %lhs_offs {tt.divisibility = dense<16> : tensor<128x64xi32>, tt.contiguity = dense<32> : tensor<128x64xi32>, tt.constancy = dense<1> : tensor<128x64xi32>} : tensor<128x64x!tt.ptr<f8E4M3FN>, #load_blocked>, tensor<128x64xi32, #load_blocked>
442|     %lhs = tt.load %lhs_ptrs_i : tensor<128x64x!tt.ptr<f8E4M3FN>, #load_blocked>
443|     %lhs_shared = ttg.local_alloc %lhs : (tensor<128x64xf8E4M3FN, #load_blocked>) -> !ttg.memdesc<128x64xf8E4M3FN, #shared, #smem>
444| 
445|     %scales_offs = tt.splat %i : i32 -> tensor<128x8xi32, #load_blocked>
446|     %scales_ptrs_i = tt.addptr %scale_ptrs, %scales_offs {tt.divisibility = dense<16> : tensor<128x8xi32>, tt.contiguity = dense<32> : tensor<128x8xi32>, tt.constancy = dense<1> : tensor<128x8xi32>} : tensor<128x8x!tt.ptr<i8>, #load_blocked>, tensor<128x8xi32, #load_blocked>
447|     %scales = tt.load %scales_ptrs_i : tensor<128x8x!tt.ptr<i8>, #load_blocked>
448|     %scales_cvt = ttg.convert_layout %scales : tensor<128x8xi8, #load_blocked> -> tensor<128x8xi8, #scales>
449|     %scales_tmem = ttng.tmem_alloc %scales_cvt : (tensor<128x8xi8, #scales>) -> !ttg.memdesc<128x8xi8, #ttng.tensor_memory_scales_encoding<>, #ttng.tensor_memory>
450| 
451|     ttng.tc_gen5_mma_scaled %lhs_shared, %rhs_shared, %tmem, %scales_tmem, %rhs_scales, %true, %true lhs = e4m3 rhs = e4m3, %barrier[%true] {is_async} :
452|       !ttg.memdesc<128x64xf8E4M3FN, #shared, #smem>,
453|       !ttg.memdesc<64x64xf8E4M3FN, #shared, #smem>,
454|       !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory, mutable>,
455|       !ttg.memdesc<128x8xi8, #ttng.tensor_memory_scales_encoding<>, #ttng.tensor_memory>,
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining broadcasted scalars or pointers, pointer arithmetic, masked or vectorized loads, structured loops, shared/local memory allocation.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 广播后的标量或指针、指针算术、带掩码或向量化的加载、结构化循环、共享/本地内存分配。

### Lines 456-463
```mlir
456|       !ttg.memdesc<64x8xi8, #ttng.tensor_memory_scales_encoding<>, #ttng.tensor_memory>,
457|       !ttg.memdesc<1xi64, #barrier_shared, #smem, mutable>
458|   }
459| 
460|   tt.return
461| }
462| 
463| }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `-tritongpu-hoist-tmem-alloc`, `-tritongpu-assign-latencies`, `-tritongpu-schedule-loops`, `-tritongpu-pipeline`, `-triton-nvidia-gpu-remove-tmem-tokens`, `-canonicalize`
- **CN:** 主要 pass 选项：`-split-input-file`，`-tritongpu-hoist-tmem-alloc`，`-tritongpu-assign-latencies`，`-tritongpu-schedule-loops`，`-tritongpu-pipeline`，`-triton-nvidia-gpu-remove-tmem-tokens`，`-canonicalize`
- **EN:** Dominant operations include `arith.constant`, `tt.addptr`, `tt.load`, `ttg.local_alloc`, `tt.func`, `tt.expand_dims`, `tt.splat`, `ttng.tmem_alloc`, `tt.make_range`, `tt.broadcast`.
- **CN:** 主要操作包括 `arith.constant`、`tt.addptr`、`tt.load`、`ttg.local_alloc`、`tt.func`、`tt.expand_dims`、`tt.splat`、`ttng.tmem_alloc`、`tt.make_range`、`tt.broadcast`。
- **EN:** The file contains 6 independently testable section(s). Check styles used: CHECK x69, CHECK-LABEL x5, CHECK-DAG x5, CHECK-SAME x4. Important labels include @chained_dot_scaled_acc, @chained_scale_after_dot, tt.func @matmul_loop_cast_load, @pipelined_gather. It validates that the selected pass pipeline rewrites each labeled test case into the checked form.
- **CN:** 该文件包含 6 个可独立测试的分段。使用的检查类型：CHECK ×69，CHECK-LABEL ×5，CHECK-DAG ×5，CHECK-SAME ×4。 关键标签包括 @chained_dot_scaled_acc，@chained_scale_after_dot，tt.func @matmul_loop_cast_load，@pipelined_gather。 它验证所选 pass 流水线会把每个带标签的测试用例重写成检查所要求的形式。

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
- **EN:** `scf`: Structured control flow for loops and conditionals.
- **CN:** `scf`：用于循环与条件分支的结构化控制流。
- **EN:** `FileCheck` is used to match required textual patterns after rewriting/lowering.
- **CN:** `FileCheck` 用于在重写/lower 之后匹配必须出现的文本模式。