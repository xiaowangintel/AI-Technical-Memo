# hoist-tmem-alloc.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/hoist-tmem-alloc.mlir`
- **EN:** Test for the MLIR/Triton pipeline `-tritongpu-hoist-tmem-alloc, -canonicalize, -tritongpu-hoist-tmem-alloc="post-pipeline=true"` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `-tritongpu-hoist-tmem-alloc, -canonicalize, -tritongpu-hoist-tmem-alloc="post-pipeline=true"` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -split-input-file -allow-unregistered-dialect -tritongpu-hoist-tmem-alloc -canonicalize | FileCheck %s`; `// RUN: triton-opt %s -split-input-file -allow-unregistered-dialect -tritongpu-hoist-tmem-alloc="post-pipeline=true" -canonicalize | FileCheck %s -check-prefix=POST-PIPELINE`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -split-input-file -allow-unregistered-dialect -tritongpu-hoist-tmem-alloc -canonicalize | FileCheck %s`；`// RUN: triton-opt %s -split-input-file -allow-unregistered-dialect -tritongpu-hoist-tmem-alloc="post-pipeline=true" -canonicalize | FileCheck %s -check-prefix=POST-PIPELINE`

## Line-by-Line Analysis / 逐行分析

### Lines 1-3
```mlir
1| // RUN: triton-opt %s -split-input-file -allow-unregistered-dialect -tritongpu-hoist-tmem-alloc -canonicalize | FileCheck %s
2| // RUN: triton-opt %s -split-input-file -allow-unregistered-dialect -tritongpu-hoist-tmem-alloc="post-pipeline=true" -canonicalize | FileCheck %s -check-prefix=POST-PIPELINE
3| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -split-input-file -allow-unregistered-dialect -tritongpu-hoist-tmem-alloc -canonicalize | FileCheck %s` ; ` triton-opt %s -split-input-file -allow-unregistered-dialect -tritongpu-hoist-tmem-alloc="post-pipeline=true" -canonicalize | FileCheck %s -check-prefix=POST-PIPELINE` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -split-input-file -allow-unregistered-dialect -tritongpu-hoist-tmem-alloc -canonicalize | FileCheck %s`；` triton-opt %s -split-input-file -allow-unregistered-dialect -tritongpu-hoist-tmem-alloc="post-pipeline=true" -canonicalize | FileCheck %s -check-prefix=POST-PIPELINE`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 4-8
```mlir
4| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
5| #blocked1 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
6| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
7| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
8| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 9-9
```mlir
9| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 10-22
```mlir
10|   // CHECK-LABEL: @chained_mma
11|   // CHECK: %[[C0:.*]] = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked1>
12|   // CHECK: %[[ACC_TM:.*]], %[[ALLOC_TOK:.*]] = ttng.tmem_alloc : ()
13|   // CHECK: %[[INIT_TOK:.*]] = ttng.tmem_store %[[C0]], %[[ACC_TM]][%[[ALLOC_TOK]]]
14|   // CHECK: %[[RES_TOK:.*]] = scf.for {{.*}} iter_args(%[[TOK:.*]] = %[[INIT_TOK]])
15|   // CHECK-NOT: ttng.tmem_alloc
16|   // CHECK-NOT: ttng.tmem_store
17|   // CHECK:   %[[MMA_TOK:.*]] = ttng.tc_gen5_mma {{.*}}, {{.*}}, %[[ACC_TM]][%[[TOK]]]
18|   // CHECK-NOT: ttng.tmem_load
19|   // CHECK:   "end_of_loop"
20|   // CHECK:   yield %[[MMA_TOK]]
21|   // CHECK: %[[ACC_TM_LOAD:.*]], %{{.*}} = ttng.tmem_load %[[ACC_TM]][%[[RES_TOK]]]
22|   // CHECK: arith.truncf %[[ACC_TM_LOAD]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @chained_mma anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @chained_mma 这样的标签用于锚定匹配范围。

### Lines 23-40
```mlir
23|   tt.func public @chained_mma(%A_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1> {tt.divisibility = 16 : i32, tt.contiguity = 16 : i32}, %B_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1> {tt.divisibility = 16 : i32, tt.contiguity = 16 : i32}, %arg3: i32) -> tensor<128x128xf16, #blocked> {
24|     %true = arith.constant true
25|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked>
26|     %cst2 = arith.constant dense<2.000000e+00> : tensor<128x128xf32, #blocked>
27|     %c0_i32 = arith.constant 0 : i32
28|     %c1_i32 = arith.constant 1 : i32
29|     %res = scf.for %i = %c0_i32 to %arg3 step %c1_i32 iter_args(%acc = %cst) -> (tensor<128x128xf32, #blocked>)  : i32 {
30|       %A = tt.load %A_ptr : tensor<128x128x!tt.ptr<f16>, #blocked1>
31|       %A_sh = ttg.local_alloc %A : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>
32|       %B = tt.load %B_ptr : tensor<128x128x!tt.ptr<f16>, #blocked1>
33|       %B_sh = ttg.local_alloc %B : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>
34|       %acc_tm, %acc_tok = ttng.tmem_alloc %acc : (tensor<128x128xf32, #blocked>) -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
35|       %mma_tok = ttng.tc_gen5_mma %A_sh, %B_sh, %acc_tm[%acc_tok], %true, %true : !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>, !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
36|       %acc_res, %load_tok = ttng.tmem_load %acc_tm[%mma_tok] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
37|       "end_of_loop"() : () -> ()
38|       scf.yield %acc_res : tensor<128x128xf32, #blocked>
39|     } {tt.scheduled_max_stage = 3 : i32}
40|     %res_f16 = arith.truncf %res : tensor<128x128xf32, #blocked> to tensor<128x128xf16, #blocked>
```
**EN:** This function-oriented block defines or enters `chained_mma`. Within it, the test exercises constants, tt.func, masked or vectorized loads, shared/local memory allocation, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `chained_mma` 为核心。测试在其中演示 常量、tt.func、带掩码或向量化的加载、共享/本地内存分配、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 41-44
```mlir
41|     tt.return %res_f16 : tensor<128x128xf16, #blocked>
42|   }
43| }
44| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 45-45
```mlir
45| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 46-50
```mlir
46| 
47| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
48| #blocked1 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
49| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
50| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 51-56
```mlir
51| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
52|   // POST-PIPELINE-LABEL: @do_not_sink_alloc_to_predicated_mma
53|   // POST-PIPELINE: %[[ACC_TM:.*]], %[[ALLOC_TOK:.*]] = ttng.tmem_alloc : ()
54|   // POST-PIPELINE: scf.for {{.*}} iter_args(%[[TOK:.*]] = %[[ALLOC_TOK]])
55|   // POST-PIPELINE-NOT: ttng.tmem_alloc
56|   // POST-PIPELINE:   %[[MMA_TOK:.*]] = ttng.tc_gen5_mma {{.*}}, {{.*}}, %[[ACC_TM]][%[[TOK]]]
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 57-72
```mlir
57|   tt.func public @do_not_sink_alloc_to_predicated_mma(%A: !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>,
58|                                                       %B: !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>,
59|                                                       %pred: i1,
60|                                                       %iters: i32) {
61|     %false = arith.constant false
62|     %c0_i32 = arith.constant 0 : i32
63|     %c1_i32 = arith.constant 1 : i32
64|     %acc_tm, %alloc_tok = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
65|     %res_tok = scf.for %i = %c0_i32 to %iters step %c1_i32 iter_args(%tok = %alloc_tok) -> (!ttg.async.token)  : i32 {
66|       %mma_tok = ttng.tc_gen5_mma %A, %B, %acc_tm[%tok], %false, %pred : !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>, !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
67|       scf.yield %mma_tok : !ttg.async.token
68|     }
69|     tt.return
70|   }
71| }
72| 
```
**EN:** This function-oriented block defines or enters `do_not_sink_alloc_to_predicated_mma`. Within it, the test exercises constants, tt.func, tensor-memory allocation, structured loops, Gen5 tensor-core MMA ops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `do_not_sink_alloc_to_predicated_mma` 为核心。测试在其中演示 常量、tt.func、张量内存分配、结构化循环、Gen5 张量核 MMA 操作，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 73-73
```mlir
73| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 74-76
```mlir
74| 
75| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
76| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 77-82
```mlir
77| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
78|   // POST-PIPELINE-LABEL: @do_not_sink_alloc_to_predicated_store
79|   // POST-PIPELINE: %[[ACC_TM:.*]], %[[ALLOC_TOK:.*]] = ttng.tmem_alloc : ()
80|   // POST-PIPELINE: scf.for {{.*}} iter_args(%[[TOK:.*]] = %[[ALLOC_TOK]])
81|   // POST-PIPELINE-NOT: ttng.tmem_alloc
82|   // POST-PIPELINE:   %[[STORE_TOK:.*]] = ttng.tmem_store {{.*}}, %[[ACC_TM]][%[[TOK]]], %{{.*}}
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 83-96
```mlir
83|   tt.func public @do_not_sink_alloc_to_predicated_store(%src: tensor<128x128xf32, #blocked>,
84|                                                         %pred: i1,
85|                                                         %iters: i32) {
86|     %c0_i32 = arith.constant 0 : i32
87|     %c1_i32 = arith.constant 1 : i32
88|     %acc_tm, %alloc_tok = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
89|     %res_tok = scf.for %i = %c0_i32 to %iters step %c1_i32 iter_args(%tok = %alloc_tok) -> (!ttg.async.token)  : i32 {
90|       %store_tok = ttng.tmem_store %src, %acc_tm[%tok], %pred : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
91|       scf.yield %store_tok : !ttg.async.token
92|     }
93|     tt.return
94|   }
95| }
96| 
```
**EN:** This function-oriented block defines or enters `do_not_sink_alloc_to_predicated_store`. Within it, the test exercises tt.func, constants, tensor-memory allocation, structured loops, ttng.tmem_store, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `do_not_sink_alloc_to_predicated_store` 为核心。测试在其中演示 tt.func、常量、张量内存分配、结构化循环、ttng.tmem_store，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 97-97
```mlir
97| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 98-102
```mlir
 98| 
 99| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
100| #blocked1 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
101| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
102| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 103-107
```mlir
103| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
104|   // POST-PIPELINE-LABEL: @sink_alloc_to_mma_use_acc_false
105|   // POST-PIPELINE: scf.for
106|   // POST-PIPELINE:   %[[ACC_TM:.*]], %[[ALLOC_TOK:.*]] = ttng.tmem_alloc : ()
107|   // POST-PIPELINE:   %[[MMA_TOK:.*]] = ttng.tc_gen5_mma {{.*}}, {{.*}}, %[[ACC_TM]][%[[ALLOC_TOK]]]
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 108-125
```mlir
108|   tt.func public @sink_alloc_to_mma_use_acc_false(%A: !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>,
109|                                                   %B: !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>,
110|                                                   %iters: i32) {
111|     %false = arith.constant false
112|     %true = arith.constant true
113|     %c0_i32 = arith.constant 0 : i32
114|     %c1_i32 = arith.constant 1 : i32
115|     // Allocate accumulator outside the loop; it will be sunk next to the MMA
116|     %acc_tm, %alloc_tok = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
117|     %res_tok = scf.for %i = %c0_i32 to %iters step %c1_i32 iter_args(%tok = %alloc_tok) -> (!ttg.async.token)  : i32 {
118|       // useAccumulator is false, so the accumulator source is ignored; the pattern
119|       // should accept this as fully overwriting the allocation and sink the alloc here.
120|       %mma_tok = ttng.tc_gen5_mma %A, %B, %acc_tm[%tok], %false, %true : !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>, !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
121|       scf.yield %mma_tok : !ttg.async.token
122|     }
123|     tt.return
124|   }
125| }
```
**EN:** This function-oriented block defines or enters `sink_alloc_to_mma_use_acc_false`. Within it, the test exercises constants, tt.func, tensor-memory allocation, structured loops, Gen5 tensor-core MMA ops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `sink_alloc_to_mma_use_acc_false` 为核心。测试在其中演示 常量、tt.func、张量内存分配、结构化循环、Gen5 张量核 MMA 操作，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 127-128
```mlir
127| 
128| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 128-133
```mlir
128| 
129| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
130| #blocked1 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
131| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
132| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
133| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 134-134
```mlir
134| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 135-149
```mlir
135|   // CHECK-LABEL: @changed_acc
136|   // CHECK-DAG: %[[TRUE:.*]] = arith.constant true
137|   // CHECK-DAG: %[[C0:.*]] = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked1>
138|   // CHECK: %[[ACC_TM:.*]], %[[ALLOC_TOK:.*]] = ttng.tmem_alloc : ()
139|   // CHECK: %[[INIT_TOK:.*]] = ttng.tmem_store %[[C0]], %[[ACC_TM]][%[[ALLOC_TOK]]]
140|   // CHECK: %[[RES_TOK:.*]] = scf.for {{.*}} iter_args(%[[TOK:.*]] = %[[INIT_TOK]])
141|   // CHECK-NOT: ttng.tmem_alloc
142|   // CHECK-NOT: ttng.tmem_store
143|   // CHECK:   %[[MMA_TOK:.*]] = ttng.tc_gen5_mma {{.*}}, {{.*}}, %[[ACC_TM]][%[[TOK]]]
144|   // CHECK:   %[[ACC:.*]], %[[LOAD_TOK:.*]] = ttng.tmem_load %[[ACC_TM]][%[[MMA_TOK]]]
145|   // CHECK:   %[[ACC_MUL:.*]] = arith.mulf %[[ACC]]
146|   // CHECK:   %[[STORE_TOK:.*]] = ttng.tmem_store %[[ACC_MUL]], %[[ACC_TM]][%[[LOAD_TOK]]], %[[TRUE]]
147|   // CHECK:   yield %[[STORE_TOK]]
148|   // CHECK: %[[ACC_TM_LOAD:.*]], %{{.*}} = ttng.tmem_load %[[ACC_TM]]
149|   // CHECK: arith.truncf %[[ACC_TM_LOAD]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @changed_acc anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @changed_acc 这样的标签用于锚定匹配范围。

### Lines 150-167
```mlir
150|   tt.func public @changed_acc(%A_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1> {tt.divisibility = 16 : i32, tt.contiguity = 16 : i32}, %B_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1> {tt.divisibility = 16 : i32, tt.contiguity = 16 : i32}, %arg3: i32) -> tensor<128x128xf16, #blocked> {
151|     %true = arith.constant true
152|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked>
153|     %cst2 = arith.constant dense<2.000000e+00> : tensor<128x128xf32, #blocked>
154|     %c0_i32 = arith.constant 0 : i32
155|     %c1_i32 = arith.constant 1 : i32
156|     %res = scf.for %i = %c0_i32 to %arg3 step %c1_i32 iter_args(%acc = %cst) -> (tensor<128x128xf32, #blocked>)  : i32 {
157|       %A = tt.load %A_ptr : tensor<128x128x!tt.ptr<f16>, #blocked1>
158|       %A_sh = ttg.local_alloc %A : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>
159|       %B = tt.load %B_ptr : tensor<128x128x!tt.ptr<f16>, #blocked1>
160|       %B_sh = ttg.local_alloc %B : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>
161|       %acc_tm, %acc_tok = ttng.tmem_alloc %acc : (tensor<128x128xf32, #blocked>) -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
162|       %mma_tok = ttng.tc_gen5_mma %A_sh, %B_sh, %acc_tm[%acc_tok], %true, %true : !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>, !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
163|       %acc_res, %load_tok = ttng.tmem_load %acc_tm[%mma_tok] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
164|       %acc_if = arith.mulf %acc_res, %cst2 : tensor<128x128xf32, #blocked>
165|       scf.yield %acc_if : tensor<128x128xf32, #blocked>
166|     } {tt.scheduled_max_stage = 3 : i32}
167|     %res_f16 = arith.truncf %res : tensor<128x128xf32, #blocked> to tensor<128x128xf16, #blocked>
```
**EN:** This function-oriented block defines or enters `changed_acc`. Within it, the test exercises constants, tt.func, masked or vectorized loads, shared/local memory allocation, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `changed_acc` 为核心。测试在其中演示 常量、tt.func、带掩码或向量化的加载、共享/本地内存分配、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 168-171
```mlir
168|     tt.return %res_f16 : tensor<128x128xf16, #blocked>
169|   }
170| }
171| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 172-172
```mlir
172| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 173-178
```mlir
173| 
174| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
175| #blocked1 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
176| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
177| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
178| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 179-179
```mlir
179| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 180-192
```mlir
180|   // CHECK-LABEL: @changed_acc_before_mma
181|   // CHECK-DAG: %[[TRUE:.*]] = arith.constant true
182|   // CHECK-DAG: %[[C0:.*]] = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked1>
183|   // CHECK: %[[ACC_TM:.*]], %[[ALLOC_TOK:.*]] = ttng.tmem_alloc : ()
184|   // CHECK: %[[INIT_TOK:.*]] = ttng.tmem_store %[[C0]], %[[ACC_TM]][%[[ALLOC_TOK]]]
185|   // CHECK: %[[RES_TOK:.*]] = scf.for {{.*}} iter_args(%[[TOK:.*]] = %[[INIT_TOK]])
186|   // CHECK:   %[[ACC:.*]], %[[LOAD_TOK:.*]] = ttng.tmem_load %[[ACC_TM]][%[[TOK]]]
187|   // CHECK:   %[[ACC_MUL:.*]] = arith.mulf %[[ACC]]
188|   // CHECK:   %[[STORE_TOK:.*]] = ttng.tmem_store %[[ACC_MUL]], %[[ACC_TM]][%[[LOAD_TOK]]], %[[TRUE]]
189|   // CHECK:   %[[MMA_TOK:.*]] = ttng.tc_gen5_mma {{.*}}, {{.*}}, %[[ACC_TM]][%[[STORE_TOK]]]
190|   // CHECK:   yield %[[MMA_TOK]]
191|   // CHECK: %[[ACC_TM_LOAD:.*]], %{{.*}} = ttng.tmem_load %[[ACC_TM]][%[[RES_TOK]]]
192|   // CHECK: arith.truncf %[[ACC_TM_LOAD]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @changed_acc_before_mma anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @changed_acc_before_mma 这样的标签用于锚定匹配范围。

### Lines 193-210
```mlir
193|   tt.func public @changed_acc_before_mma(%A_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1> {tt.divisibility = 16 : i32, tt.contiguity = 16 : i32}, %B_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1> {tt.divisibility = 16 : i32, tt.contiguity = 16 : i32}, %arg3: i32) -> tensor<128x128xf16, #blocked> {
194|     %true = arith.constant true
195|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked>
196|     %cst2 = arith.constant dense<2.000000e+00> : tensor<128x128xf32, #blocked>
197|     %c0_i32 = arith.constant 0 : i32
198|     %c1_i32 = arith.constant 1 : i32
199|     %res = scf.for %i = %c0_i32 to %arg3 step %c1_i32 iter_args(%acc = %cst) -> (tensor<128x128xf32, #blocked>)  : i32 {
200|       %A = tt.load %A_ptr : tensor<128x128x!tt.ptr<f16>, #blocked1>
201|       %A_sh = ttg.local_alloc %A : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>
202|       %B = tt.load %B_ptr : tensor<128x128x!tt.ptr<f16>, #blocked1>
203|       %B_sh = ttg.local_alloc %B : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>
204|       %acc_mul = arith.mulf %acc, %cst2 : tensor<128x128xf32, #blocked>
205|       %acc_tm, %acc_tok = ttng.tmem_alloc %acc_mul : (tensor<128x128xf32, #blocked>) -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
206|       %mma_tok = ttng.tc_gen5_mma %A_sh, %B_sh, %acc_tm[%acc_tok], %true, %true : !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>, !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
207|       %acc_res, %load_tok = ttng.tmem_load %acc_tm[%mma_tok] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
208|       scf.yield %acc_res : tensor<128x128xf32, #blocked>
209|     } {tt.scheduled_max_stage = 3 : i32}
210|     %res_f16 = arith.truncf %res : tensor<128x128xf32, #blocked> to tensor<128x128xf16, #blocked>
```
**EN:** This function-oriented block defines or enters `changed_acc_before_mma`. Within it, the test exercises constants, tt.func, masked or vectorized loads, shared/local memory allocation, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `changed_acc_before_mma` 为核心。测试在其中演示 常量、tt.func、带掩码或向量化的加载、共享/本地内存分配、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 211-214
```mlir
211|     tt.return %res_f16 : tensor<128x128xf16, #blocked>
212|   }
213| }
214| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 215-215
```mlir
215| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 216-221
```mlir
216| 
217| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
218| #blocked1 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
219| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
220| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
221| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 222-222
```mlir
222| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 223-237
```mlir
223|   // CHECK-LABEL: @select_after_mma
224|   // CHECK: %[[C0:.*]] = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked1>
225|   // CHECK: %[[CND:.*]] = "cnd"() : () -> i1
226|   // CHECK: %[[ACC_TM:.*]], %[[ALLOC_TOK:.*]] = ttng.tmem_alloc : ()
227|   // CHECK: %[[INIT_TOK:.*]] = ttng.tmem_store %[[C0]], %[[ACC_TM]][%[[ALLOC_TOK]]]
228|   // CHECK: %[[RES_TOK:.*]] = scf.for {{.*}} iter_args(%[[TOK:.*]] = %[[INIT_TOK]])
229|   // CHECK-NOT: ttng.tmem_alloc
230|   // CHECK-NOT: ttng.tmem_store
231|   // CHECK:   %[[MMA_TOK:.*]] = ttng.tc_gen5_mma {{.*}}, {{.*}}, %[[ACC_TM]][%[[TOK]]]
232|   // CHECK-NOT: ttng.tmem_load
233|   // CHECK:   %[[CND_NEG:.*]] = arith.xori %[[CND]]
234|   // CHECK:   %[[STORE_TOK:.*]] = ttng.tmem_store {{.*}}, %[[ACC_TM]][%[[MMA_TOK]]], %[[CND_NEG]]
235|   // CHECK:   yield %[[STORE_TOK]]
236|   // CHECK: %[[ACC_TM_LOAD:.*]], %{{.*}} = ttng.tmem_load %[[ACC_TM]][%[[RES_TOK]]]
237|   // CHECK: arith.truncf %[[ACC_TM_LOAD]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @select_after_mma anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @select_after_mma 这样的标签用于锚定匹配范围。

### Lines 238-255
```mlir
238|   tt.func public @select_after_mma(%A_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1> {tt.divisibility = 16 : i32, tt.contiguity = 16 : i32}, %B_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1> {tt.divisibility = 16 : i32, tt.contiguity = 16 : i32}, %arg3: i32) -> tensor<128x128xf16, #blocked> {
239|     %true = arith.constant true
240|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked>
241|     %cst2 = arith.constant dense<2.000000e+00> : tensor<128x128xf32, #blocked>
242|     %c0_i32 = arith.constant 0 : i32
243|     %c1_i32 = arith.constant 1 : i32
244|     %cnd = "cnd"() : () -> i1
245|     %res = scf.for %i = %c0_i32 to %arg3 step %c1_i32 iter_args(%acc = %cst) -> (tensor<128x128xf32, #blocked>)  : i32 {
246|       %A = tt.load %A_ptr : tensor<128x128x!tt.ptr<f16>, #blocked1>
247|       %A_sh = ttg.local_alloc %A : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>
248|       %B = tt.load %B_ptr : tensor<128x128x!tt.ptr<f16>, #blocked1>
249|       %B_sh = ttg.local_alloc %B : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>
250|       %acc_tm, %acc_tok = ttng.tmem_alloc %acc : (tensor<128x128xf32, #blocked>) -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
251|       %mma_tok = ttng.tc_gen5_mma %A_sh, %B_sh, %acc_tm[%acc_tok], %true, %true : !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>, !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
252|       %acc_res, %load_tok = ttng.tmem_load %acc_tm[%mma_tok] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
253|       %acc_if = arith.select %cnd, %acc_res, %cst2 : tensor<128x128xf32, #blocked>
254|       scf.yield %acc_if : tensor<128x128xf32, #blocked>
255|     } {tt.scheduled_max_stage = 3 : i32}
```
**EN:** This function-oriented block defines or enters `select_after_mma`. Within it, the test exercises constants, tt.func, masked or vectorized loads, shared/local memory allocation, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `select_after_mma` 为核心。测试在其中演示 常量、tt.func、带掩码或向量化的加载、共享/本地内存分配、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 256-260
```mlir
256|     %res_f16 = arith.truncf %res : tensor<128x128xf32, #blocked> to tensor<128x128xf16, #blocked>
257|     tt.return %res_f16 : tensor<128x128xf16, #blocked>
258|   }
259| }
260| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on arith.truncf, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 arith.truncf、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 261-261
```mlir
261| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 262-268
```mlir
262| 
263| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
264| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
265| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
266| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
267| #tmem1 = #ttng.tensor_memory_scales_encoding<>
268| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 269-269
```mlir
269| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 270-279
```mlir
270|   // CHECK-LABEL: @two_dots
271|   // CHECK: %[[ACC_TM1:.*]] = ttng.tmem_alloc : ()
272|   // CHECK: %[[ACC_TM2:.*]] = ttng.tmem_alloc : ()
273|   // CHECK: scf.for
274|   // CHECK:   ttng.tmem_store
275|   // CHECK:   ttng.tc_gen5_mma
276|   // CHECK:   ttng.tmem_load
277|   // CHECK:   ttng.tmem_store
278|   // CHECK:   ttng.tc_gen5_mma
279|   // CHECK:   ttng.tmem_load
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @two_dots anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @two_dots 这样的标签用于锚定匹配范围。

### Lines 280-297
```mlir
280|   tt.func public @two_dots(%A_ptr: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.contiguity = 16 : i32, tt.divisibility = 16 : i32}, %B_ptr: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.contiguity = 16 : i32, tt.divisibility = 16 : i32}, %acc_ptr: tensor<128x128x!tt.ptr<f32>, #blocked> {tt.contiguity = 16 : i32, tt.divisibility = 16 : i32}, %res_ptr: tensor<128x128x!tt.ptr<f32>, #blocked> {tt.contiguity = 16 : i32, tt.divisibility = 16 : i32}, %arg3: i32) {
281|     %true = arith.constant true
282|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked>
283|     %c0_i32 = arith.constant 0 : i32
284|     %c1_i32 = arith.constant 1 : i32
285|     scf.for %i = %c0_i32 to %arg3 step %c1_i32  : i32 {
286|       %3 = tt.load %A_ptr : tensor<128x128x!tt.ptr<f16>, #blocked>
287|       %4 = ttg.local_alloc %3 : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>
288|       %5 = tt.load %B_ptr : tensor<128x128x!tt.ptr<f16>, #blocked>
289|       %6 = ttg.local_alloc %5 : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>
290|       %acc = tt.load %acc_ptr : tensor<128x128x!tt.ptr<f32>, #blocked>
291| 
292|       %acc_tm, %acc_tok = ttng.tmem_alloc %acc : (tensor<128x128xf32, #blocked>) -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
293|       %mma_tok = ttng.tc_gen5_mma %4, %6, %acc_tm[%acc_tok], %true, %true : !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>, !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
294|       %acc_res, %load_tok = ttng.tmem_load %acc_tm[%mma_tok] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
295| 
296|       %acc_tm2, %acc_tok2 = ttng.tmem_alloc %acc_res : (tensor<128x128xf32, #blocked>) -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
297|       %mma_tok2 = ttng.tc_gen5_mma %4, %6, %acc_tm2[%acc_tok2], %true, %true : !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>, !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
```
**EN:** This function-oriented block defines or enters `two_dots`. Within it, the test exercises constants, masked or vectorized loads, tt.func, shared/local memory allocation, tensor-memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `two_dots` 为核心。测试在其中演示 常量、带掩码或向量化的加载、tt.func、共享/本地内存分配、张量内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 298-305
```mlir
298|       %acc_res2, %load_tok2 = ttng.tmem_load %acc_tm2[%mma_tok2] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
299| 
300|       tt.store %res_ptr, %acc_res2 : tensor<128x128x!tt.ptr<f32>, #blocked>
301|     }
302|     tt.return
303|   }
304| }
305| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tmem_load, masked or vectorized stores, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tmem_load、带掩码或向量化的存储、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 306-306
```mlir
306| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 307-313
```mlir
307| #blocked = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
308| #blocked1 = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [32, 1], warpsPerCTA = [1, 4], order = [1, 0]}>
309| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 8}>
310| #shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = true, elementBitWidth = 8, fp4Padded = true}>
311| #smem = #ttg.shared_memory
312| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
313| #tmem_scales = #ttng.tensor_memory_scales_encoding<>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 314-314
```mlir
314| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 315-315
```mlir
315|   // CHECK-LABEL: @hoist_constant_inputs
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @hoist_constant_inputs anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @hoist_constant_inputs 这样的标签用于锚定匹配范围。

### Lines 316-320
```mlir
316|   tt.func public @hoist_constant_inputs(%arg0: !ttg.memdesc<128x128xf8E5M2, #shared, #smem>, %arg1: !ttg.memdesc<64x128xi8, #shared1, #smem>, %arg2: !ttg.memdesc<128x4xi8, #tmem_scales, #ttng.tensor_memory>, %arg3: i32, %arg4: !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>) {
317|     %true = arith.constant true
318|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked>
319|     %c0_i32 = arith.constant 0 : i32
320|     %c1_i32 = arith.constant 1 : i32
```
**EN:** This function-oriented block defines or enters `hoist_constant_inputs`. Within it, the test exercises constants, tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `hoist_constant_inputs` 为核心。测试在其中演示 常量、tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 321-335
```mlir
321|     // CHECK: arith.trunci
322|     // CHECK: tt.splat
323|     // CHECK: ttng.tmem_alloc
324|     // CHECK: scf.for
325|     // CHECK:  ttng.tc_gen5_mma_scaled
326|     scf.for %arg5 = %c0_i32 to %arg3 step %c1_i32  : i32 {
327|       %0 = arith.trunci %arg3 : i32 to i8
328|       %1 = tt.splat %0 : i8 -> tensor<128x4xi8, #blocked1>
329|       %2 = ttng.tmem_alloc %1 : (tensor<128x4xi8, #blocked1>) -> !ttg.memdesc<128x4xi8, #tmem_scales, #ttng.tensor_memory>
330|       ttng.tc_gen5_mma_scaled %arg0, %arg1, %arg4, %arg2, %2, %true, %true lhs = e5m2 rhs = e2m1 : !ttg.memdesc<128x128xf8E5M2, #shared, #smem>, !ttg.memdesc<64x128xi8, #shared1, #smem>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.memdesc<128x4xi8, #tmem_scales, #ttng.tensor_memory>, !ttg.memdesc<128x4xi8, #tmem_scales, #ttng.tensor_memory>
331|     }
332|     tt.return
333|   }
334| }
335| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops, arith.trunci, broadcasted scalars or pointers, tensor-memory allocation, scaled tensor-core MMA ops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环、arith.trunci、广播后的标量或指针、张量内存分配、带缩放的张量核 MMA 操作。

### Lines 336-336
```mlir
336| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 337-342
```mlir
337| 
338| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
339| #blocked1 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
340| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
341| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
342| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 343-343
```mlir
343| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 344-361
```mlir
344|   // CHECK-LABEL: @use_in_conditional
345|   // CHECK: %[[C0:.*]] = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked1>
346|   // CHECK: %[[CND:.*]] = "cnd"() : () -> i1
347|   // CHECK: %[[ACC_TM:.*]], %[[ALLOC_TOK:.*]] = ttng.tmem_alloc : ()
348|   // CHECK: %[[INIT_TOK:.*]] = ttng.tmem_store %[[C0]], %[[ACC_TM]][%[[ALLOC_TOK]]]
349|   // CHECK: %[[RES_TOK:.*]] = scf.for {{.*}} iter_args(%[[TOK:.*]] = %[[INIT_TOK]])
350|   // CHECK-NOT: ttng.tmem_alloc
351|   // CHECK-NOT: ttng.tmem_store
352|   // CHECK:   %[[MMA_TOK:.*]] = ttng.tc_gen5_mma {{.*}}, {{.*}}, %[[ACC_TM]][%[[TOK]]]
353|   // CHECK:   %[[CND_TOK:.*]] = scf.if %[[CND]]
354|   // CHECK:     "epilogue"()
355|   // CHECK:     %[[RESULT:.*]], %[[LOAD_TOK:.*]] = ttng.tmem_load %[[ACC_TM]][%[[MMA_TOK]]]
356|   // CHECK:     yield %[[LOAD_TOK]]
357|   // CHECK:   else
358|   // CHECK:     yield %[[MMA_TOK]]
359|   // CHECK:   %[[CND_NEG:.*]] = arith.xori %[[CND]]
360|   // CHECK:   %[[STORE_TOK:.*]] = ttng.tmem_store {{.*}}, %[[ACC_TM]][%[[CND_TOK]]], %[[CND_NEG]]
361|   // CHECK:   yield %[[STORE_TOK]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @use_in_conditional anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @use_in_conditional 这样的标签用于锚定匹配范围。

### Lines 362-363
```mlir
362|   // CHECK: %[[ACC_TM_LOAD:.*]], %{{.*}} = ttng.tmem_load %[[ACC_TM]][%[[RES_TOK]]]
363|   // CHECK: arith.truncf %[[ACC_TM_LOAD]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 364-381
```mlir
364|   tt.func public @use_in_conditional(%A_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1> {tt.divisibility = 16 : i32, tt.contiguity = 16 : i32}, %B_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1> {tt.divisibility = 16 : i32, tt.contiguity = 16 : i32}, %arg3: i32) -> tensor<128x128xf16, #blocked> {
365|     %true = arith.constant true
366|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked>
367|     %cst2 = arith.constant dense<2.000000e+00> : tensor<128x128xf32, #blocked>
368|     %c0_i32 = arith.constant 0 : i32
369|     %c1_i32 = arith.constant 1 : i32
370|     %cnd = "cnd"() : () -> i1
371|     %res = scf.for %i = %c0_i32 to %arg3 step %c1_i32 iter_args(%acc = %cst) -> (tensor<128x128xf32, #blocked>)  : i32 {
372|       %A = tt.load %A_ptr : tensor<128x128x!tt.ptr<f16>, #blocked1>
373|       %A_sh = ttg.local_alloc %A : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>
374|       %B = tt.load %B_ptr : tensor<128x128x!tt.ptr<f16>, #blocked1>
375|       %B_sh = ttg.local_alloc %B : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>
376|       %acc_tm, %acc_tok = ttng.tmem_alloc %acc : (tensor<128x128xf32, #blocked>) -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
377|       %mma_tok = ttng.tc_gen5_mma %A_sh, %B_sh, %acc_tm[%acc_tok], %true, %true : !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>, !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory, mutable>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
378|       %acc_res, %load_tok = ttng.tmem_load %acc_tm[%mma_tok] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
379|       scf.if %cnd {
380|         "epilogue"() : () -> ()
381|         "user"(%acc_res) : (tensor<128x128xf32, #blocked>) -> ()
```
**EN:** This function-oriented block defines or enters `use_in_conditional`. Within it, the test exercises constants, tt.func, masked or vectorized loads, shared/local memory allocation, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `use_in_conditional` 为核心。测试在其中演示 常量、tt.func、带掩码或向量化的加载、共享/本地内存分配、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 382-390
```mlir
382|       }
383|       %acc_if = arith.select %cnd, %acc_res, %cst2 : tensor<128x128xf32, #blocked>
384|       scf.yield %acc_if : tensor<128x128xf32, #blocked>
385|     } {tt.scheduled_max_stage = 3 : i32}
386|     %res_f16 = arith.truncf %res : tensor<128x128xf32, #blocked> to tensor<128x128xf16, #blocked>
387|     tt.return %res_f16 : tensor<128x128xf16, #blocked>
388|   }
389| }
390| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining arith.select, loop/if yielded values, arith.truncf, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 arith.select、循环/分支产出值、arith.truncf、tt.return。

### Lines 391-391
```mlir
391| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 392-394
```mlir
392| 
393| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
394| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 395-396
```mlir
395| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
396|   // POST-PIPELINE-LABEL: @hoist_out_of_if
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 397-414
```mlir
397|   tt.func public @hoist_out_of_if(%arg0: i1, %arg1: tensor<128x128xf32, #blocked>) -> tensor<128x128xf32, #blocked> {
398|     // POST-PIPELINE: %[[A:.+]], %[[T0:.+]] = ttng.tmem_alloc : ()
399|     // POST-PIPELINE: %[[T1:.+]] = ttng.tmem_store %{{.*}}, %[[A]][%[[T0]]]
400|     // POST-PIPELINE: %[[I:.+]] = scf.if %{{.+}} -> (!ttg.async.token) {
401|     // POST-PIPELINE:   %[[T2:.+]] = "write_to_tmem"
402|     // POST-PIPELINE:   scf.yield %[[T2]]
403|     // POST-PIPELINE: } else {
404|     // POST-PIPELINE:   scf.yield %[[T1]]
405|     // POST-PIPELINE: }
406|     // POST-PIPELINE: %[[L:.+]], %[[T4:.+]] = ttng.tmem_load %[[A]][%[[I]]
407|     // POST-PIPELINE: tt.return %[[L]]
408|     %0 = scf.if %arg0 -> (tensor<128x128xf32, #blocked>) {
409|       %result, %token = ttng.tmem_alloc %arg1 : (tensor<128x128xf32, #blocked>) -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
410|       %1 = "write_to_tmem"(%result) : (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>) -> !ttg.async.token
411|       %result_0, %token_1 = ttng.tmem_load %result[%1] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
412|       scf.yield %result_0 : tensor<128x128xf32, #blocked>
413|     } else {
414|       scf.yield %arg1 : tensor<128x128xf32, #blocked>
```
**EN:** This function-oriented block defines or enters `hoist_out_of_if`. Within it, the test exercises tt.func, loop/if yielded values, structured conditionals, tensor-memory allocation, ttng.tmem_load, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `hoist_out_of_if` 为核心。测试在其中演示 tt.func、循环/分支产出值、结构化条件分支、张量内存分配、ttng.tmem_load，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 415-419
```mlir
415|     }
416|     tt.return %0 : tensor<128x128xf32, #blocked>
417|   }
418| }
419| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 420-420
```mlir
420| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 421-423
```mlir
421| 
422| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
423| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 424-424
```mlir
424| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 425-437
```mlir
425|   tt.func public @forward_tmem_load(%m: !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, %t: !ttg.async.token) -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token) {
426|     %true = arith.constant true
427|     %result, %token0 = ttng.tmem_load %m[%t] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
428|     // POST-PIPELINE-LABEL: @forward_tmem_load
429|     // POST-PIPELINE-SAME:    %[[ARG0:.+]]: !ttg.memdesc<128x128xf32,
430|     // POST-PIPELINE-SAME:    %[[ARG1:.+]]: !ttg.async.token
431|     // POST-PIPELINE-NEXT:    tt.return %[[ARG0]], %[[ARG1]]
432|     %result1, %token1 = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
433|     %token2 = ttng.tmem_store %result, %result1[%token1], %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
434|     tt.return %result1, %token2 : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token
435|   }
436| }
437| 
```
**EN:** This function-oriented block defines or enters `forward_tmem_load`. Within it, the test exercises tt.func, constants, ttng.tmem_load, tensor-memory allocation, ttng.tmem_store, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `forward_tmem_load` 为核心。测试在其中演示 tt.func、常量、ttng.tmem_load、张量内存分配、ttng.tmem_store，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 438-438
```mlir
438| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 439-441
```mlir
439| 
440| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
441| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 442-442
```mlir
442| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 443-443
```mlir
443|   // CHECK-LABEL: @sink_multiple_tmem_load
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @sink_multiple_tmem_load anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @sink_multiple_tmem_load 这样的标签用于锚定匹配范围。

### Lines 444-450
```mlir
444|   tt.func public @sink_multiple_tmem_load(%m: !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, %t: !ttg.async.token) -> (tensor<128x128xf32, #blocked>, tensor<128x128xf32, #blocked>) {
445|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked>
446|     %c0_i32 = arith.constant 0 : i32
447|     %c1_i32 = arith.constant 1 : i32
448|     %c2_i32 = arith.constant 2 : i32
449|     %res:2 = scf.for %i = %c0_i32 to %c2_i32 step %c1_i32 iter_args(%init0 = %cst, %init1 = %cst) -> (tensor<128x128xf32, #blocked>, tensor<128x128xf32, #blocked>)  : i32 {
450|       // Any order is fine, just make sure we don't reorder them in an infinite loop.
```
**EN:** This function-oriented block defines or enters `sink_multiple_tmem_load`. Within it, the test exercises constants, tt.func, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `sink_multiple_tmem_load` 为核心。测试在其中演示 常量、tt.func、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 451-460
```mlir
451|       // CHECK-COUNT-2: ttng.tmem_load
452|       // CHECK: scf.yield
453|       %l0, %token_1 = ttng.tmem_load %m[%t] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
454|       %l1, %token_2 = ttng.tmem_load %m[%t] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
455|       scf.yield %l0, %l1 : tensor<128x128xf32, #blocked>, tensor<128x128xf32, #blocked>
456|     } {tt.scheduled_max_stage = 3 : i32}
457|     tt.return %res#0, %res#1 : tensor<128x128xf32, #blocked>, tensor<128x128xf32, #blocked>
458|   }
459| }
460| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining ttng.tmem_load, loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 ttng.tmem_load、循环/分支产出值、tt.return。

### Lines 461-461
```mlir
461| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 462-464
```mlir
462| 
463| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
464| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 465-465
```mlir
465| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 466-477
```mlir
466|   tt.func public @combine_tmem_store_and_alloc() -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token) {
467|     %true = arith.constant true
468|     // HOIST-IF-LABEL: @combine_tmem_store_and_alloc
469|     // HOIST-IF: ttng.tmem_alloc
470|     // HOIST-IF-NEXT: "def_tensor"()
471|     // HOIST-IF-NEXT: ttng.tmem_store
472|     %result1, %token1 = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
473|     %def = "def_tensor" () : () -> tensor<128x128xf32, #blocked>
474|     %token2 = ttng.tmem_store %def, %result1[%token1], %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
475|     tt.return %result1, %token2 : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token
476|   }
477| }
```
**EN:** This function-oriented block defines or enters `combine_tmem_store_and_alloc`. Within it, the test exercises tt.func, constants, tensor-memory allocation, ttng.tmem_store, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `combine_tmem_store_and_alloc` 为核心。测试在其中演示 tt.func、常量、张量内存分配、ttng.tmem_store、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `-allow-unregistered-dialect`, `-tritongpu-hoist-tmem-alloc`, `-canonicalize`, `-tritongpu-hoist-tmem-alloc="post-pipeline=true"`
- **CN:** 主要 pass 选项：`-split-input-file`，`-allow-unregistered-dialect`，`-tritongpu-hoist-tmem-alloc`，`-canonicalize`，`-tritongpu-hoist-tmem-alloc="post-pipeline=true"`
- **EN:** Dominant operations include `arith.constant`, `tt.func`, `module`, `ttng.tmem_alloc`, `tt.return`, `tt.load`, `ttg.local_alloc`, `scf.for`, `ttng.tmem_load`, `scf.yield`.
- **CN:** 主要操作包括 `arith.constant`、`tt.func`、`module`、`ttng.tmem_alloc`、`tt.return`、`tt.load`、`ttg.local_alloc`、`scf.for`、`ttng.tmem_load`、`scf.yield`。
- **EN:** The file contains 13 independently testable section(s). Check styles used: CHECK x72, CHECK-NOT x10, CHECK-LABEL x8, CHECK-DAG x4. Important labels include @chained_mma, @changed_acc, @changed_acc_before_mma, @select_after_mma. It validates both the presence of the desired rewrite and the absence of unwanted residual operations.
- **CN:** 该文件包含 13 个可独立测试的分段。使用的检查类型：CHECK ×72，CHECK-NOT ×10，CHECK-LABEL ×8，CHECK-DAG ×4。 关键标签包括 @chained_mma，@changed_acc，@changed_acc_before_mma，@select_after_mma。 它同时验证目标重写已经出现，并且不再残留不需要的操作。

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