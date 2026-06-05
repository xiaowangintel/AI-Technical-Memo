# partition-scheduling.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/partition-scheduling.mlir`
- **EN:** Test for the MLIR/Triton pipeline `--tritongpu-hoist-tmem-alloc, --tritongpu-partition-scheduling` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `--tritongpu-hoist-tmem-alloc, --tritongpu-partition-scheduling` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s --split-input-file --tritongpu-hoist-tmem-alloc --tritongpu-partition-scheduling -allow-unregistered-dialect | FileCheck %s`
- **CN:** RUN 流水线：`// RUN: triton-opt %s --split-input-file --tritongpu-hoist-tmem-alloc --tritongpu-partition-scheduling -allow-unregistered-dialect | FileCheck %s`

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
1| // RUN: triton-opt %s --split-input-file --tritongpu-hoist-tmem-alloc --tritongpu-partition-scheduling -allow-unregistered-dialect | FileCheck %s
2| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s --split-input-file --tritongpu-hoist-tmem-alloc --tritongpu-partition-scheduling -allow-unregistered-dialect | FileCheck %s` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s --split-input-file --tritongpu-hoist-tmem-alloc --tritongpu-partition-scheduling -allow-unregistered-dialect | FileCheck %s`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 3-11
```mlir
 3| #blocked = #ttg.blocked<{sizePerThread = [1, 64], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
 4| #load_blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [2, 2], order = [1, 0]}>
 5| 
 6| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
 7| #shared_T = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = true, elementBitWidth = 16}>
 8| #shared_f32 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
 9| 
10| #smem = #ttg.shared_memory
11| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 64, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 12-13
```mlir
12| module attributes {"ttg.num-warps" = 4 : i32, ttg.target = "cuda:100"} {
13| 
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 14-14
```mlir
14| // CHECK-LABEL: @attention_forward
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @attention_forward anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @attention_forward 这样的标签用于锚定匹配范围。

### Lines 15-32
```mlir
15| tt.func public @attention_forward(
16|   %Q_shared: !ttg.memdesc<256x64xf16, #shared, #smem>,
17|   %K_desc: !tt.tensordesc<64x64xf16, #shared>,
18|   %V_desc: !tt.tensordesc<64x64xf16, #shared>,
19|   %qk_scale: f32,
20|   %n_tiles: i32
21| ) {
22|   %true = arith.constant true
23|   %false = arith.constant false
24|   %c0_i32 = arith.constant 0 : i32
25|   %c64_i32 = arith.constant 64 : i32
26| 
27|   %neg_inf = arith.constant dense<0xFF800000> : tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
28|   %zero = arith.constant dense<0.0> : tensor<256x64xf32, #blocked>
29|   %one = arith.constant dense<1.0> : tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
30| 
31| 
32|   %loop_outs:4 = scf.for %i = %c0_i32 to %n_tiles step %c64_i32 iter_args(
```
**EN:** This function-oriented block defines or enters `attention_forward`. Within it, the test exercises constants, tt.func, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `attention_forward` 为核心。测试在其中演示 常量、tt.func、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 33-43
```mlir
33|     %l_i = %one,
34|     %acc = %zero,
35|     %m_i = %neg_inf,
36|     %e_i = %one
37|   ) -> (
38|     tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>>,
39|     tensor<256x64xf32, #blocked>,
40|     tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>>,
41|     tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
42|   ) : i32 {
43| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 44-48
```mlir
44|     // CHECK-COUNT-2: ttg.partition = array<i32: 3>
45|     %K = tt.descriptor_load %K_desc[%i, %c0_i32] : !tt.tensordesc<64x64xf16, #shared> -> tensor<64x64xf16, #load_blocked>
46|     %K_shared = ttg.local_alloc %K : (tensor<64x64xf16, #load_blocked>) -> !ttg.memdesc<64x64xf16, #shared, #smem>
47| 
48|     %QK_tmem, %QK_tok = ttng.tmem_alloc : () -> (!ttg.memdesc<256x64xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.descriptor_load, shared/local memory allocation, tensor-memory allocation. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.descriptor_load、共享/本地内存分配、张量内存分配。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 49-52
```mlir
49|     // CHECK-COUNT-2: ttg.partition = array<i32: 2>
50|     %K_trans = ttg.memdesc_trans %K_shared {order = array<i32: 1, 0>} : !ttg.memdesc<64x64xf16, #shared, #smem> -> !ttg.memdesc<64x64xf16, #shared_T, #smem>
51|     %QK_mma_tok = ttng.tc_gen5_mma %Q_shared, %K_trans, %QK_tmem[%QK_tok], %false, %true : !ttg.memdesc<256x64xf16, #shared, #smem>, !ttg.memdesc<64x64xf16, #shared_T, #smem>, !ttg.memdesc<256x64xf32, #tmem, #ttng.tensor_memory, mutable>
52| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.memdesc_trans, Gen5 tensor-core MMA ops. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.memdesc_trans、Gen5 张量核 MMA 操作。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 53-56
```mlir
53|     // CHECK-COUNT-3: ttg.partition = array<i32: 0>
54|     %QK, %QK_load_tok = ttng.tmem_load %QK_tmem[%QK_mma_tok] : !ttg.memdesc<256x64xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<256x64xf32, #blocked>
55|     %row_max = "compute_row_max"(%QK, %qk_scale) : (tensor<256x64xf32, #blocked>, f32) -> tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
56|     %QK_adj = "sub_row_max"(%QK, %row_max, %qk_scale) : (tensor<256x64xf32, #blocked>, tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>>, f32) -> tensor<256x64xf32, #blocked>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tmem_load. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tmem_load。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 57-58
```mlir
57|     // CHECK: [[SOFTMAX:%.*]] = math.exp2 {{.*}} {ttg.partition = array<i32: 0>} : tensor<256x64xf32
58|     %softmax = math.exp2 %QK_adj : tensor<256x64xf32, #blocked>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on math.exp2. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 math.exp2。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 59-62
```mlir
59|     // CHECK-COUNT-4: ttg.partition = array<i32:
60|     %diff = arith.subf %m_i, %row_max : tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
61|     %alpha = math.exp2 %diff : tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
62| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on arith.subf, math.exp2. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 arith.subf、math.exp2。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 63-65
```mlir
63|     // CHECK-NEXT: tt.reduce
64|     %l_ij = "tt.reduce"(%softmax) <{axis = 1 : i32}> ({
65|     ^bb0(%arg29: f32, %arg30: f32):
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 66-68
```mlir
66|       // CHECK-COUNT-2: ttg.partition = array<i32: 0>
67|       %68 = arith.addf %arg29, %arg30 : f32
68|       tt.reduce.return %68 : f32
```
**EN:** This operational block carries the concrete IR steps for the test, centered on floating-point additions, reductions. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 浮点加法、归约。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 69-70
```mlir
69|       // CHECK-NEXT: ttg.partition = array<i32: 0>, ttg.partition.outputs = [array<i32: 0>]
70|     }) : (tensor<256x64xf32, #blocked>) -> tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 71-79
```mlir
71|     // CHECK-COUNT-6: ttg.partition = array<i32:
72|     %l_i_scaled = arith.mulf %l_i, %alpha : tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
73|     %next_l_i = arith.addf %l_i_scaled, %l_ij : tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
74| 
75|     %alpha_0 = tt.expand_dims %alpha {axis = 1 : i32} : tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<256x1xf32, #blocked>
76|     %alpha_1 = tt.broadcast %alpha_0 : tensor<256x1xf32, #blocked> -> tensor<256x64xf32, #blocked>
77| 
78|     %acc_corrected = arith.mulf %acc, %alpha_1 : tensor<256x64xf32, #blocked>
79| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on arith.mulf, floating-point additions, shape expansion, tensor broadcasting. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 arith.mulf、浮点加法、形状扩展、张量广播。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 80-81
```mlir
80|     // CHECK-NEXT: [[X:%.*]] = arith.addf [[SOFTMAX]], [[SOFTMAX]] {ttg.partition = array<i32: 1>}
81|     %x = arith.addf %softmax, %softmax : tensor<256x64xf32, #blocked>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 82-96
```mlir
82|     // CHECK-NEXT: [[ACC_X:%.*]] = arith.addf %{{.*}}, [[X]] {ttg.partition = array<i32: 1>}
83|     // CHECK-COUNT-8: ttg.partition = array<i32:
84|     %acc_x = arith.addf %acc, %x : tensor<256x64xf32, #blocked>
85|     %e = "sum"(%acc_x) : (tensor<256x64xf32, #blocked>) -> tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
86|     %next_e_i = arith.addf %e_i, %e : tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
87| 
88|     %V = tt.descriptor_load %V_desc[%i, %c0_i32] : !tt.tensordesc<64x64xf16, #shared> -> tensor<64x64xf16, #load_blocked>
89|     %V_shared = ttg.local_alloc %V : (tensor<64x64xf16, #load_blocked>) -> !ttg.memdesc<64x64xf16, #shared, #smem>
90|     %P = arith.truncf %softmax : tensor<256x64xf32, #blocked> to tensor<256x64xf16, #blocked>
91| 
92|     %P_tmem = ttng.tmem_alloc %P : (tensor<256x64xf16, #blocked>) -> !ttg.memdesc<256x64xf16, #tmem, #ttng.tensor_memory>
93|     %acc_tmem, %acc_tok = ttng.tmem_alloc %acc_corrected : (tensor<256x64xf32, #blocked>) -> (!ttg.memdesc<256x64xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
94|     %PV_mma_tok = ttng.tc_gen5_mma %P_tmem, %V_shared, %acc_tmem[%acc_tok], %true, %true : !ttg.memdesc<256x64xf16, #tmem, #ttng.tensor_memory>, !ttg.memdesc<64x64xf16, #shared, #smem>, !ttg.memdesc<256x64xf32, #tmem, #ttng.tensor_memory, mutable>
95|     %O, %O_tok = ttng.tmem_load %acc_tmem[%PV_mma_tok] : !ttg.memdesc<256x64xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<256x64xf32, #blocked>
96| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on floating-point additions, tensor-memory allocation, tt.descriptor_load, shared/local memory allocation, arith.truncf. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 浮点加法、张量内存分配、tt.descriptor_load、共享/本地内存分配、arith.truncf。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 97-98
```mlir
97|     // CHECK-NEXT: scf.yield {ttg.partition = array<i32: 0, 1, 2, 3>}
98|     scf.yield %next_l_i, %O, %row_max, %next_e_i : tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>>, tensor<256x64xf32, #blocked>, tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>>, tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值。

### Lines 99-106
```mlir
 99|     // CHECK-NEXT: ttg.partition = array<i32: 0, 1, 2, 3>, ttg.partition.outputs = [array<i32: 0>, array<i32: 0>, array<i32: 1>, array<i32: 2>, array<i32: 1>]
100|   } {tt.warp_specialize}
101| 
102|   "use"(%loop_outs#0, %loop_outs#1, %loop_outs#2) : (tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>>, tensor<256x64xf32, #blocked>, tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>>) -> ()
103| 
104|   tt.return
105| }
106| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 107-107
```mlir
107| // CHECK-LABEL: @mma_operand_view
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @mma_operand_view anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @mma_operand_view 这样的标签用于锚定匹配范围。

### Lines 108-125
```mlir
108| tt.func public @mma_operand_view(
109|   %Q_shared: !ttg.memdesc<256x64xf16, #shared, #smem>,
110|   %K_desc: !tt.tensordesc<64x64xf16, #shared>,
111|   %V_desc: !tt.tensordesc<64x64xf16, #shared>,
112|   %qk_scale: f32,
113|   %n_tiles: i32
114| ) {
115|   %true = arith.constant true
116|   %false = arith.constant false
117|   %c0_i32 = arith.constant 0 : i32
118|   %c64_i32 = arith.constant 64 : i32
119| 
120|   %neg_inf = arith.constant dense<0xFF800000> : tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
121|   %zero = arith.constant dense<0.0> : tensor<256x64xf32, #blocked>
122|   %one = arith.constant dense<1.0> : tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
123| 
124|   %QK_tmem, %QK_tok = ttng.tmem_alloc : () -> (!ttg.memdesc<256x64xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
125| 
```
**EN:** This function-oriented block defines or enters `mma_operand_view`. Within it, the test exercises constants, tt.func, tensor-memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `mma_operand_view` 为核心。测试在其中演示 常量、tt.func、张量内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 126-127
```mlir
126|   scf.for %i = %c0_i32 to %n_tiles step %c64_i32 : i32 {
127|     %K = tt.descriptor_load %K_desc[%i, %c0_i32] : !tt.tensordesc<64x64xf16, #shared> -> tensor<64x64xf16, #load_blocked>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops, tt.descriptor_load.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环、tt.descriptor_load。

### Lines 128-130
```mlir
128|     // CHECK: [[K_SHARED:%.*]] = ttg.local_alloc {{.*}}partition = array<i32: 2>
129|     %K_shared = ttg.local_alloc %K : (tensor<64x64xf16, #load_blocked>) -> !ttg.memdesc<64x64xf16, #shared, #smem>
130| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 131-136
```mlir
131|     // CHECK-DAG: [[TRANS_MMA:%.*]] = ttg.memdesc_trans [[K_SHARED]] {{.*}}partition = array<i32: 1>
132|     // CHECK-DAG: [[K_VIEW:%.*]] = ttg.memdesc_subslice [[TRANS_MMA]]{{.*}}partition = array<i32: 1>
133|     // CHECK-DAG: [[TRANS_USER:%.*]] = ttg.memdesc_trans [[K_SHARED]] {{.*}}partition = array<i32: 0>
134|     %K_trans = ttg.memdesc_trans %K_shared {order = array<i32: 1, 0>} : !ttg.memdesc<64x64xf16, #shared, #smem> -> !ttg.memdesc<64x64xf16, #shared_T, #smem>
135|     %K_view = ttg.memdesc_subslice %K_trans [0, 0]  : !ttg.memdesc<64x64xf16, #shared_T, #smem> -> !ttg.memdesc<64x64xf16, #shared_T, #smem>
136| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.memdesc_trans, ttg.memdesc_subslice. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.memdesc_trans、ttg.memdesc_subslice。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 137-139
```mlir
137|     // CHECK: ttng.tc_gen5_mma %arg0, [[K_VIEW]]{{.*}}partition = array<i32: 1>
138|     %QK_mma_tok = ttng.tc_gen5_mma %Q_shared, %K_view, %QK_tmem[%QK_tok], %false, %true : !ttg.memdesc<256x64xf16, #shared, #smem>, !ttg.memdesc<64x64xf16, #shared_T, #smem>, !ttg.memdesc<256x64xf32, #tmem, #ttng.tensor_memory, mutable>
139| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on Gen5 tensor-core MMA ops. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 Gen5 张量核 MMA 操作。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 140-142
```mlir
140|     // CHECK: local_load [[TRANS_USER]] {{.*}}partition = array<i32: 0>
141|     %x = ttg.local_load %K_trans : !ttg.memdesc<64x64xf16, #shared_T, #smem> -> tensor<64x64xf16, #load_blocked>
142| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on local/shared memory loads. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 本地/共享内存加载。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 143-146
```mlir
143|     // CHECK: tmem_load {{.*}}partition = array<i32: 0>
144|     %QK, %QK_load_tok = ttng.tmem_load %QK_tmem[%QK_mma_tok] : !ttg.memdesc<256x64xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<256x64xf32, #blocked>
145| 
146|     "use"(%x, %QK) {data} : (tensor<64x64xf16, #load_blocked>, tensor<256x64xf32, #blocked>) -> ()
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tmem_load. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tmem_load。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 147-153
```mlir
147|     // CHECK: "use"
148|     // CHECK-NEXT: ttg.partition = array<i32: 0, 1, 2>
149|   } {tt.warp_specialize}
150| 
151|   tt.return
152| }
153| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 154-154
```mlir
154| // CHECK-LABEL: @optimize_broadcast
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @optimize_broadcast anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @optimize_broadcast 这样的标签用于锚定匹配范围。

### Lines 155-157
```mlir
155| tt.func @optimize_broadcast(%arg0: i32, %arg1: !tt.tensordesc<128x128xf32, #shared_f32>) {
156|   %c0_i32 = arith.constant 0 : i32
157|   %c1_i32 = arith.constant 1 : i32
```
**EN:** This function-oriented block defines or enters `optimize_broadcast`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `optimize_broadcast` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 158-164
```mlir
158|   // CHECK: scf.for
159|   scf.for %i = %c0_i32 to %arg0 step %c1_i32 : i32 {
160|     %md = tt.descriptor_load %arg1[%c0_i32, %c0_i32] {ttg.partition = array<i32: 1>} : !tt.tensordesc<128x128xf32, #shared_f32> -> tensor<128x128xf32, #load_blocked>
161|     %smem = ttg.local_alloc %md {ttg.partition = array<i32: 1>} : (tensor<128x128xf32, #load_blocked>) -> !ttg.memdesc<128x128xf32, #shared_f32, #smem>
162|     %tmp = ttg.local_load %smem {ttg.partition = array<i32: 1>} : !ttg.memdesc<128x128xf32, #shared_f32, #smem> -> tensor<128x128xf32, #load_blocked>
163|     "use_memdesc"(%tmp) {ttg.partition = array<i32: 1>} : (tensor<128x128xf32, #load_blocked>) -> ()
164| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops, tt.descriptor_load, shared/local memory allocation, local/shared memory loads.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环、tt.descriptor_load、共享/本地内存分配、本地/共享内存加载。

### Lines 165-167
```mlir
165|     // CHECK: [[X:%.*]] = "producer"{{.*}}partition = array<i32: 0>
166|     %x = "producer"() {ttg.partition = array<i32: 0>, data} : () -> tensor<128xf32>
167| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 168-170
```mlir
168|     // CHECK-DAG: [[X0_P0:%.*]] = tt.expand_dims [[X]] {{.*}}partition = array<i32: 0>
169|     // CHECK-DAG: [[X0_P1:%.*]] = tt.expand_dims [[X]] {{.*}}partition = array<i32: 1>
170|     %x0 = tt.expand_dims %x {axis = 0 : i32} : tensor<128xf32> -> tensor<1x128xf32>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shape expansion. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 形状扩展。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 171-174
```mlir
171|     // CHECK-DAG: [[X1_P0:%.*]] = tt.broadcast [[X0_P0]] {{.*}}partition = array<i32: 0>
172|     // CHECK-DAG: [[X1_P1:%.*]] = tt.broadcast [[X0_P1]] {{.*}}partition = array<i32: 1>
173|     %x1 = tt.broadcast %x0 : tensor<1x128xf32> -> tensor<128x128xf32>
174| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tensor broadcasting. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 张量广播。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 175-176
```mlir
175|     // CHECK: "use"([[X1_P0]]) {{.*}}partition = array<i32: 0>
176|     "use"(%x1) {ttg.partition = array<i32: 0>, data} : (tensor<128x128xf32>) -> ()
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 177-178
```mlir
177|     // CHECK: "use"([[X1_P1]]) {{.*}}partition = array<i32: 1>
178|     "use"(%x1) {ttg.partition = array<i32: 1>, data} : (tensor<128x128xf32>) -> ()
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 179-183
```mlir
179|     // CHECK-NEXT: ttg.partition = array<i32: 0, 1>
180|   } {tt.warp_specialize, ttg.partition.stages = [0 : i32, 1 : i32], ttg.warp_specialize.tag = 0 : i32}
181|   tt.return
182| }
183| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 184-184
```mlir
184| // CHECK-LABEL: @no_partitions
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @no_partitions anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @no_partitions 这样的标签用于锚定匹配范围。

### Lines 185-187
```mlir
185| tt.func @no_partitions(%arg0: i32) {
186|   %c0_i32 = arith.constant 0 : i32
187|   %c1_i32 = arith.constant 1 : i32
```
**EN:** This function-oriented block defines or enters `no_partitions`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `no_partitions` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 188-196
```mlir
188|   // CHECK: scf.for %{{.*}} = %c0_i32 to %arg0 step %c1_i32 : i32
189|   // CHECK-NOT: ttg.partition
190|   // CHECK-NOT: ttg.warp_specialize.tag
191|   scf.for %i = %c0_i32 to %arg0 step %c1_i32 : i32 {
192|     "use"(%c0_i32) : (i32) -> ()
193|   } {tt.warp_specialize}
194|   tt.return
195| }
196| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环、tt.return。

### Lines 197-197
```mlir
197| // CHECK-LABEL: @mma_no_memory_ops
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @mma_no_memory_ops anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @mma_no_memory_ops 这样的标签用于锚定匹配范围。

### Lines 198-202
```mlir
198| tt.func @mma_no_memory_ops(%arg0: i32, %arg1: !ttg.memdesc<256x64xf16, #shared, #smem>, %arg2: !ttg.memdesc<64x64xf16, #shared_T, #smem>, %arg3: !ttg.memdesc<256x64xf32, #tmem, #ttng.tensor_memory, mutable>) {
199|   %c0_i32 = arith.constant 0 : i32
200|   %c1_i32 = arith.constant 1 : i32
201|   %false = arith.constant false
202|   %true = arith.constant true
```
**EN:** This function-oriented block defines or enters `mma_no_memory_ops`. Within it, the test exercises constants, tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `mma_no_memory_ops` 为核心。测试在其中演示 常量、tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 203-213
```mlir
203|   // CHECK: scf.for %{{.*}} = %c0_i32 to %arg0 step %c1_i32 : i32
204|   // CHECK-NOT: ttg.partition
205|   // CHECK-NOT: ttg.warp_specialize.tag
206|   scf.for %i = %c0_i32 to %arg0 step %c1_i32 : i32 {
207|     %0 = ttng.tc_gen5_mma %arg1, %arg2, %arg3[], %false, %true : !ttg.memdesc<256x64xf16, #shared, #smem>, !ttg.memdesc<64x64xf16, #shared_T, #smem>, !ttg.memdesc<256x64xf32, #tmem, #ttng.tensor_memory, mutable>
208|   } {tt.warp_specialize}
209|   tt.return
210| }
211| 
212| }
213| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops, Gen5 tensor-core MMA ops, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环、Gen5 张量核 MMA 操作、tt.return。

### Lines 214-214
```mlir
214| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 215-222
```mlir
215| 
216| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
217| #blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [2, 2], order = [1, 0]}>
218| #blocked2 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
219| #blocked3 = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
220| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
221| #smem = #ttg.shared_memory
222| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 223-224
```mlir
223| module attributes {"ttg.num-warps" = 4 : i32, ttg.target = "cuda:100"} {
224| 
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 225-225
```mlir
225|   // CHECK-LABEL: @matmul_change_desc_in_prologue
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @matmul_change_desc_in_prologue anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @matmul_change_desc_in_prologue 这样的标签用于锚定匹配范围。

### Lines 226-237
```mlir
226|   tt.func @matmul_change_desc_in_prologue(%arg0: !tt.ptr<f16>, %arg1: !tt.ptr<f16>) {
227|     %c1_i64 = arith.constant 1 : i64
228|     %c0_i32 = arith.constant 0 : i32
229|     %c1_i32 = arith.constant 1 : i32
230|     %true = arith.constant true
231|     %false = arith.constant false
232|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked>
233|     %c32_i32 = arith.constant 32 : i32
234|     %0 = ub.poison : !tt.tensordesc<128x64xf16, #shared>
235|     %1 = ub.poison : !tt.tensordesc<64x128xf16, #shared>
236|     %result, %token = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
237|     %2 = ttng.tmem_store %cst, %result[%token], %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
```
**EN:** This function-oriented block defines or enters `matmul_change_desc_in_prologue`. Within it, the test exercises constants, tt.func, ub.poison, tensor-memory allocation, ttng.tmem_store, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `matmul_change_desc_in_prologue` 为核心。测试在其中演示 常量、tt.func、ub.poison、张量内存分配、ttng.tmem_store，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 238-239
```mlir
238|     // CHECK: scf.for
239|     %3:4 = scf.for %arg2 = %c0_i32 to %c32_i32 step %c1_i32 iter_args(%arg3 = %true, %arg4 = %0, %arg5 = %1, %arg6 = %2) -> (i1, !tt.tensordesc<128x64xf16, #shared>, !tt.tensordesc<64x128xf16, #shared>, !ttg.async.token)  : i32 {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环。

### Lines 240-241
```mlir
240|       // CHECK-NEXT: "prologue_cond"({{.*}}) {ttg.partition = array<i32: 2>}
241|       %4 = "prologue_cond"(%arg2) : (i32) -> i1
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 242-243
```mlir
242|       // CHECK-NEXT: scf.if
243|       %5:2 = scf.if %4 -> (!tt.tensordesc<128x64xf16, #shared>, !tt.tensordesc<64x128xf16, #shared>) {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured conditionals.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化条件分支。

### Lines 244-246
```mlir
244|         // CHECK-COUNT-2: ttg.partition = array<i32: 2>
245|         %15 = tt.make_tensor_descriptor %arg0, [%arg2, %arg2], [%c1_i64, %c1_i64] : <f16>, <128x64xf16, #shared>
246|         %16 = tt.make_tensor_descriptor %arg1, [%arg2, %arg2], [%c1_i64, %c1_i64] : <f16>, <64x128xf16, #shared>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.make_tensor_descriptor. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.make_tensor_descriptor。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 247-249
```mlir
247|         // CHECK-NEXT: scf.yield {ttg.partition = array<i32: 2>}
248|         scf.yield %15, %16 : !tt.tensordesc<128x64xf16, #shared>, !tt.tensordesc<64x128xf16, #shared>
249|       } else {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值。

### Lines 250-252
```mlir
250|         // CHECK-NEXT: } else {
251|         // CHECK-NEXT: scf.yield {ttg.partition = array<i32: 2>}
252|         scf.yield %arg4, %arg5 : !tt.tensordesc<128x64xf16, #shared>, !tt.tensordesc<64x128xf16, #shared>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值。

### Lines 253-254
```mlir
253|         // CHECK-NEXT: ttg.partition = array<i32: 2>, ttg.partition.outputs = [array<i32: 2>, array<i32: 2>]
254|       }
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 255-260
```mlir
255|       // CHECK-COUNT-5: ttg.partition = array<i32: 2>
256|       %6:3 = "get_offsets"(%arg2) : (i32) -> (i32, i32, i32)
257|       %7 = tt.descriptor_load %arg4[%6#0, %6#2] : !tt.tensordesc<128x64xf16, #shared> -> tensor<128x64xf16, #blocked1>
258|       %8 = tt.descriptor_load %arg5[%6#1, %6#2] : !tt.tensordesc<64x128xf16, #shared> -> tensor<64x128xf16, #blocked1>
259|       %9 = ttg.local_alloc %7 : (tensor<128x64xf16, #blocked1>) -> !ttg.memdesc<128x64xf16, #shared, #smem>
260|       %10 = ttg.local_alloc %8 : (tensor<64x128xf16, #blocked1>) -> !ttg.memdesc<64x128xf16, #shared, #smem>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.descriptor_load, shared/local memory allocation. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.descriptor_load、共享/本地内存分配。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 261-262
```mlir
261|       // CHECK-NEXT: tc_gen5_mma {{.*}} {ttg.partition = array<i32: 1>} {{.*}}
262|       %11 = ttng.tc_gen5_mma %9, %10, %result[%arg6], %arg3, %true : !ttg.memdesc<128x64xf16, #shared, #smem>, !ttg.memdesc<64x128xf16, #shared, #smem>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 263-264
```mlir
263|       // CHECK-NEXT: ttg.partition = array<i32: 0, 1>
264|       %12 = arith.cmpi eq, %arg2, %c0_i32 : i32
```
**EN:** This operational block carries the concrete IR steps for the test, centered on integer comparisons. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 整数比较。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 265-266
```mlir
265|       // CHECK-NEXT: ttg.partition = array<i32: 1>
266|       %13 = arith.select %12, %false, %true : i1
```
**EN:** This operational block carries the concrete IR steps for the test, centered on arith.select. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 arith.select。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 267-268
```mlir
267|       // CHECK-NEXT: scf.if
268|       %14 = scf.if %12 -> (!ttg.async.token) {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured conditionals.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化条件分支。

### Lines 269-271
```mlir
269|         // CHECK-COUNT-2: ttg.partition = array<i32: 0>
270|         %result_0, %token_1 = ttng.tmem_load %result[%11] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
271|         "acc_user"(%result_0) : (tensor<128x128xf32, #blocked>) -> ()
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tmem_load. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tmem_load。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 272-274
```mlir
272|         // CHECK-NEXT: scf.yield {ttg.partition = array<i32: 0, 1>}
273|         scf.yield %token_1 : !ttg.async.token
274|       } else {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值。

### Lines 275-279
```mlir
275|         // CHECK-NEXT: } else {
276|         // CHECK-NEXT: scf.yield {ttg.partition = array<i32: 0, 1>}
277|         // CHECK-NEXT: ttg.partition = array<i32: 0, 1>, ttg.partition.outputs = [array<i32: 1>]
278|         scf.yield %11 : !ttg.async.token
279|       }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值。

### Lines 280-281
```mlir
280|       // CHECK-NEXT: scf.yield {ttg.partition = array<i32: 0, 1, 2>}
281|       scf.yield %13, %5#0, %5#1, %14 : i1, !tt.tensordesc<128x64xf16, #shared>, !tt.tensordesc<64x128xf16, #shared>, !ttg.async.token
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值。

### Lines 282-286
```mlir
282|       // CHECK-NEXT: ttg.partition = array<i32: 0, 1, 2>, ttg.partition.outputs = [array<i32: 1>, array<i32: 2>, array<i32: 2>, array<i32: 1>]
283|     } {tt.disallow_acc_multi_buffer, tt.num_stages = 4 : i32, tt.warp_specialize}
284|     tt.return
285|   }
286| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 287-287
```mlir
287|   // CHECK-LABEL: @matmul_tma_acc_with_conditional_def_and_use
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @matmul_tma_acc_with_conditional_def_and_use anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @matmul_tma_acc_with_conditional_def_and_use 这样的标签用于锚定匹配范围。

### Lines 288-296
```mlir
288|   tt.func @matmul_tma_acc_with_conditional_def_and_use(%arg0: !tt.tensordesc<1x64xf16, #shared>, %arg1: !tt.tensordesc<64x128xf16, #shared>) {
289|     %c0_i32 = arith.constant 0 : i32
290|     %c1_i32 = arith.constant 1 : i32
291|     %true = arith.constant true
292|     %false = arith.constant false
293|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked>
294|     %c32_i32 = arith.constant 32 : i32
295|     %result, %token = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
296|     %0 = ttng.tmem_store %cst, %result[%token], %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
```
**EN:** This function-oriented block defines or enters `matmul_tma_acc_with_conditional_def_and_use`. Within it, the test exercises constants, tt.func, tensor-memory allocation, ttng.tmem_store, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `matmul_tma_acc_with_conditional_def_and_use` 为核心。测试在其中演示 常量、tt.func、张量内存分配、ttng.tmem_store，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 297-298
```mlir
297|     // CHECK: scf.for
298|     %1:2 = scf.for %arg2 = %c0_i32 to %c32_i32 step %c1_i32 iter_args(%arg3 = %true, %arg4 = %0) -> (i1, !ttg.async.token)  : i32 {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环。

### Lines 299-305
```mlir
299|       // CHECK-COUNT-6: ttg.partition = array<i32: 2>
300|       %2:3 = "get_offsets"(%arg2) : (i32) -> (i32, i32, i32)
301|       %3 = tt.splat %2#0 : i32 -> tensor<128xi32, #blocked2>
302|       %4 = tt.descriptor_gather %arg0[%3, %2#2] : (!tt.tensordesc<1x64xf16, #shared>, tensor<128xi32, #blocked2>, i32) -> tensor<128x64xf16, #blocked1>
303|       %5 = tt.descriptor_load %arg1[%2#1, %2#2] : !tt.tensordesc<64x128xf16, #shared> -> tensor<64x128xf16, #blocked1>
304|       %6 = ttg.local_alloc %4 : (tensor<128x64xf16, #blocked1>) -> !ttg.memdesc<128x64xf16, #shared, #smem>
305|       %7 = ttg.local_alloc %5 : (tensor<64x128xf16, #blocked1>) -> !ttg.memdesc<64x128xf16, #shared, #smem>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, broadcasted scalars or pointers, tt.descriptor_gather, tt.descriptor_load. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、广播后的标量或指针、tt.descriptor_gather、tt.descriptor_load。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 306-307
```mlir
306|       // CHECK-NEXT: ttg.partition = array<i32: 1>
307|       %8 = ttng.tc_gen5_mma %6, %7, %result[%arg4], %arg3, %true : !ttg.memdesc<128x64xf16, #shared, #smem>, !ttg.memdesc<64x128xf16, #shared, #smem>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on Gen5 tensor-core MMA ops. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 Gen5 张量核 MMA 操作。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 308-309
```mlir
308|       // CHECK-NEXT: ttg.partition = array<i32: 0, 1>
309|       %9 = arith.cmpi eq, %arg2, %c0_i32 : i32
```
**EN:** This operational block carries the concrete IR steps for the test, centered on integer comparisons. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 整数比较。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 310-311
```mlir
310|       // CHECK-NEXT: ttg.partition = array<i32: 1>
311|       %10 = arith.select %9, %false, %true : i1
```
**EN:** This operational block carries the concrete IR steps for the test, centered on arith.select. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 arith.select。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 312-313
```mlir
312|       // CHECK-NEXT: scf.if
313|       %11 = scf.if %9 -> (!ttg.async.token) {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured conditionals.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化条件分支。

### Lines 314-316
```mlir
314|         // CHECK-COUNT-2: ttg.partition = array<i32: 0>
315|         %result_0, %token_1 = ttng.tmem_load %result[%8] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
316|         "acc_user"(%result_0) : (tensor<128x128xf32, #blocked>) -> ()
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tmem_load. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tmem_load。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 317-319
```mlir
317|         // CHECK-NEXT: scf.yield {ttg.partition = array<i32: 0, 1>}
318|         scf.yield %token_1 : !ttg.async.token
319|       } else {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值。

### Lines 320-324
```mlir
320|         // CHECK-NEXT: } else {
321|         // CHECK-NEXT: scf.yield {ttg.partition = array<i32: 0, 1>}
322|         // CHECK-NEXT: ttg.partition = array<i32: 0, 1>, ttg.partition.outputs = [array<i32: 1>]
323|         scf.yield %8 : !ttg.async.token
324|       }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值。

### Lines 325-326
```mlir
325|       // CHECK-NEXT: scf.yield {ttg.partition = array<i32: 0, 1, 2>}
326|       scf.yield %10, %11 : i1, !ttg.async.token
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值。

### Lines 327-332
```mlir
327|       // CHECK-NEXT: ttg.partition = array<i32: 0, 1, 2>, ttg.partition.outputs = [array<i32: 1>, array<i32: 1>]
328|     } {tt.disallow_acc_multi_buffer, tt.num_stages = 2 : i32, tt.warp_specialize}
329|     tt.return
330|   }
331| 
332| }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 333-333
```mlir
333| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 334-339
```mlir
334| 
335| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 8], warpsPerCTA = [8, 1], order = [1, 0]}>
336| #linear = #ttg.linear<{register = [[0, 1], [0, 2], [0, 4], [0, 8]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 16]], warp = [[16, 0], [32, 0], [0, 32]], block = []}>
337| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16, rank = 3}>
338| #shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32, rank = 3}>
339| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 340-341
```mlir
340| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
341| 
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 342-342
```mlir
342|   // CHECK-LABEL: @if_stmt_yield_outputs
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @if_stmt_yield_outputs anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @if_stmt_yield_outputs 这样的标签用于锚定匹配范围。

### Lines 343-354
```mlir
343|   tt.func @if_stmt_yield_outputs(%lb: i32, %ub: i32, %step: i32,
344|                                  %a0: i32, %b0: i32,
345|                                  %arg1: !tt.tensordesc<1x128x64xbf16, #shared> {tt.nv_tma_desc = 1 : i32},
346|                                  %arg2: !tt.tensordesc<1x64x64xf32, #shared1> {tt.nv_tma_desc = 1 : i32}) {
347|     %false = arith.constant false
348|     %true = arith.constant true
349|     %c0_i32 = arith.constant 0 : i32
350|     %c3_i32 = arith.constant 3 : i32
351|     %c128_i32 = arith.constant 128 : i32
352|     %cst = arith.constant dense<448> : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
353|     %cst_1 = arith.constant dense<0.000000e+00> : tensor<128x64xbf16, #blocked>
354|     %cst_3 = arith.constant dense<0.000000e+00> : tensor<64x64xf32, #linear>
```
**EN:** This function-oriented block defines or enters `if_stmt_yield_outputs`. Within it, the test exercises constants, tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `if_stmt_yield_outputs` 为核心。测试在其中演示 常量、tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 355-356
```mlir
355|     // CHECK: scf.for
356|     scf.for %arg3 = %lb to %ub step %step : i32 {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环。

### Lines 357-359
```mlir
357|       // CHECK-NEXT: tt.descriptor_load {{.*}} {ttg.partition = array<i32: 2>} {{.*}}
358|       %20 = tt.descriptor_load %arg1[%a0, %b0, %c0_i32] : !tt.tensordesc<1x128x64xbf16, #shared> -> tensor<128x64xbf16, #blocked>
359|       %22 = arith.cmpi sge, %arg3, %c3_i32 : i32
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 360-373
```mlir
360|       // CHECK: scf.if
361|       %23 = scf.if %22 -> (tensor<128x64xbf16, #blocked>) {
362|         %32 = arith.muli %arg3, %c128_i32 : i32
363|         %36 = tt.splat %32 : i32 -> tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
364|         %38 = arith.cmpi slt, %36, %cst : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
365|         %39 = tt.expand_dims %38 {axis = 1 : i32} : tensor<128xi1, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<128x1xi1, #blocked>
366|         %40 = tt.broadcast %39 : tensor<128x1xi1, #blocked> -> tensor<128x64xi1, #blocked>
367|         //  CHECK: arith.select {{.*}} {ttg.partition = array<i32: 0>} {{.*}}
368|         //  CHECK-NEXT: scf.yield {ttg.partition = array<i32: 0>}
369|         %41 = arith.select %40, %20, %cst_1 : tensor<128x64xi1, #blocked>, tensor<128x64xbf16, #blocked>
370|         scf.yield %41 : tensor<128x64xbf16, #blocked>
371|       } else {
372|         scf.yield %20 : tensor<128x64xbf16, #blocked>
373|       }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, structured conditionals, integer multiplications, broadcasted scalars or pointers, integer comparisons.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、结构化条件分支、整数乘法、广播后的标量或指针、整数比较。

### Lines 374-377
```mlir
374|       // CHECK-NEXT: } else {
375|       // CHECK-NEXT: scf.yield {ttg.partition = array<i32: 0>}
376|       // CHECK-NEXT: ttg.partition = array<i32: 0>, ttg.partition.outputs = [array<i32: 0>]
377|       "use"(%23) {data, mma} : (tensor<128x64xbf16, #blocked>) -> ()
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 378-381
```mlir
378|       // CHECK: "use"
379|       // CHECK-NEXT ttg.warp_specialize.tag = 0 : i32
380|     } {tt.warp_specialize = true}
381| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 382-397
```mlir
382|     // CHECK: scf.for
383|     scf.for %arg3 = %lb to %ub step %step : i32 {
384|       %20 = tt.descriptor_load %arg1[%a0, %b0, %c0_i32] : !tt.tensordesc<1x128x64xbf16, #shared> -> tensor<128x64xbf16, #blocked>
385|       %22 = arith.cmpi sge, %arg3, %c3_i32 : i32
386|       %23 = scf.if %22 -> (tensor<128x64xbf16, #blocked>) {
387|         %32 = arith.muli %arg3, %c128_i32 {ttg.partition = array<i32: 0>} : i32
388|         %36 = tt.splat %32 {ttg.partition = array<i32: 0>} : i32 -> tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
389|         %38 = arith.cmpi slt, %36, %cst {ttg.partition = array<i32: 0>} : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
390|         %39 = tt.expand_dims %38 {axis = 1 : i32, ttg.partition = array<i32: 0>} : tensor<128xi1, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<128x1xi1, #blocked>
391|         %40 = tt.broadcast %39 {ttg.partition = array<i32: 0>} : tensor<128x1xi1, #blocked> -> tensor<128x64xi1, #blocked>
392|         %41 = arith.select %40, %20, %cst_1 : tensor<128x64xi1, #blocked>, tensor<128x64xbf16, #blocked>
393|         scf.yield %41 : tensor<128x64xbf16, #blocked>
394|       } else {
395|         scf.yield %20 : tensor<128x64xbf16, #blocked>
396|       }
397|       "use"(%23) {data} : (tensor<128x64xbf16, #blocked>) -> ()
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining integer comparisons, loop/if yielded values, structured loops, tt.descriptor_load, structured conditionals.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 整数比较、循环/分支产出值、结构化循环、tt.descriptor_load、结构化条件分支。

### Lines 398-402
```mlir
398|       // CHECK: "use"
399|       // CHECK-NEXT: ttg.warp_specialize.tag = 1 : i32
400|     } {tt.warp_specialize = true}
401| 
402| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 403-406
```mlir
403|     // CHECK: scf.for
404|     scf.for %arg4 = %lb to %ub step %step : i32 {
405|       %20 = tt.descriptor_load %arg1[%a0, %b0, %c0_i32] : !tt.tensordesc<1x128x64xbf16, #shared> -> tensor<128x64xbf16, #blocked>
406|       %22 = arith.cmpi sge, %arg4, %c3_i32 : i32
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops, tt.descriptor_load, integer comparisons.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环、tt.descriptor_load、整数比较。

### Lines 407-409
```mlir
407|       // CHECK: scf.if
408|       %23 = scf.if %22 -> (tensor<128x64xbf16, #blocked>) {
409|         scf.yield %20 : tensor<128x64xbf16, #blocked>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured conditionals, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化条件分支、循环/分支产出值。

### Lines 410-422
```mlir
410|         // CHECK: scf.yield {ttg.partition = array<i32: 0>}
411|         // CHECK-NEXT: } else {
412|       } else {
413|         %32 = arith.muli %arg4, %c128_i32 : i32
414|         %36 = tt.splat %32 : i32 -> tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
415|         %38 = arith.cmpi slt, %36, %cst : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
416|         %39 = tt.expand_dims %38 {axis = 1 : i32} : tensor<128xi1, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<128x1xi1, #blocked>
417|         %40 = tt.broadcast %39 : tensor<128x1xi1, #blocked> -> tensor<128x64xi1, #blocked>
418|         //  CHECK: arith.select {{.*}} {ttg.partition = array<i32: 0>} {{.*}}
419|         //  CHECK-NEXT: scf.yield {ttg.partition = array<i32: 0>}
420|         %41 = arith.select %40, %20, %cst_1 : tensor<128x64xi1, #blocked>, tensor<128x64xbf16, #blocked>
421|         scf.yield %41 : tensor<128x64xbf16, #blocked>
422|       }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining integer multiplications, broadcasted scalars or pointers, integer comparisons, shape expansion, tensor broadcasting.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 整数乘法、广播后的标量或指针、整数比较、形状扩展、张量广播。

### Lines 423-429
```mlir
423|       // CHECK-NEXT: ttg.partition = array<i32: 0>, ttg.partition.outputs = [array<i32: 0>]
424|       "use"(%23) {data, mma} : (tensor<128x64xbf16, #blocked>) -> ()
425|     } {tt.warp_specialize = true}
426|     tt.return
427|   }
428| }
429| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 430-430
```mlir
430| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 431-437
```mlir
431| 
432| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
433| #blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [1, 0]}>
434| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 8}>
435| #shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = true, elementBitWidth = 8}>
436| #smem = #ttg.shared_memory
437| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 438-438
```mlir
438| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 439-439
```mlir
439|   // CHECK-LABEL: matmul_nested_persistent_ws_kernel
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: matmul_nested_persistent_ws_kernel anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: matmul_nested_persistent_ws_kernel 这样的标签用于锚定匹配范围。

### Lines 440-455
```mlir
440|   tt.func public @matmul_nested_persistent_ws_kernel(%a_desc_0: !tt.tensordesc<128x128xf8E4M3FN, #shared>, %b_desc_1: !tt.tensordesc<128x128xf8E4M3FN, #shared>, %c_desc_2: !tt.tensordesc<128x128xf8E4M3FN, #shared>, %M: i32 {tt.divisibility = 16 : i32}, %N: i32 {tt.divisibility = 16 : i32}, %K: i32 {tt.divisibility = 16 : i32}) attributes {noinline = false} {
441|     %false = arith.constant false
442|     %true = arith.constant true
443|     %c1_i64 = arith.constant 1 : i64
444|     %c128_i32 = arith.constant 128 : i32
445|     %c148_i32 = arith.constant 148 : i32
446|     %c0_i32 = arith.constant 0 : i32
447|     %c1_i32 = arith.constant 1 : i32
448|     %c8_i32 = arith.constant 8 : i32
449|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked>
450|     %start_pid = tt.get_program_id x : i32
451|     %num_pid_m_3 = arith.divsi %M, %c128_i32 : i32
452|     %num_pid_n_4 = arith.divsi %N, %c128_i32 : i32
453|     %k_tiles_5 = arith.divsi %K, %c128_i32 : i32
454|     %num_tiles = arith.muli %num_pid_m_3, %num_pid_n_4 : i32
455|     %num_pid_in_group = arith.muli %num_pid_n_4, %c8_i32 : i32
```
**EN:** This function-oriented block defines or enters `matmul_nested_persistent_ws_kernel`. Within it, the test exercises constants, arith.divsi, tt.func, integer multiplications, program IDs, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `matmul_nested_persistent_ws_kernel` 为核心。测试在其中演示 常量、arith.divsi、tt.func、整数乘法、程序 ID，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 456-457
```mlir
456|     // CHECK: scf.for
457|     scf.for %tile_id = %start_pid to %num_tiles step %c148_i32  : i32 {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环。

### Lines 458-468
```mlir
458|       // CHECK-COUNT-10: {ttg.partition = array<i32: 0, 2>}
459|       %group_id = arith.divsi %tile_id, %num_pid_in_group : i32
460|       %first_pid_m = arith.muli %group_id, %c8_i32 : i32
461|       %group_size_m = arith.subi %num_pid_m_3, %first_pid_m : i32
462|       %group_size_m_6 = arith.minsi %group_size_m, %c8_i32 : i32
463|       %pid_m = arith.remsi %tile_id, %group_size_m_6 : i32
464|       %pid_m_7 = arith.addi %first_pid_m, %pid_m : i32
465|       %pid_n = arith.remsi %tile_id, %num_pid_in_group : i32
466|       %pid_n_8 = arith.divsi %pid_n, %group_size_m_6 : i32
467|       %off_am = arith.muli %pid_m_7, %c128_i32 : i32
468|       %off_bn = arith.muli %pid_n_8, %c128_i32 : i32
```
**EN:** This operational block carries the concrete IR steps for the test, centered on integer multiplications, arith.divsi, arith.remsi, arith.subi, arith.minsi. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 整数乘法、arith.divsi、arith.remsi、arith.subi、arith.minsi。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 469-470
```mlir
469|       // CHECK-NEXT: {ttg.partition = array<i32: 0, 1>}
470|       %accumulator, %accumulator_9 = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 471-472
```mlir
471|       // CHECK-NEXT: {ttg.partition = array<i32: 0>}
472|       %accumulator_10 = ttng.tmem_store %cst, %accumulator[%accumulator_9], %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 473-474
```mlir
473|       // CHECK: scf.for
474|       %accumulator_11:2 = scf.for %accumulator_15 = %c0_i32 to %k_tiles_5 step %c1_i32 iter_args(%arg11 = %false, %accumulator_16 = %accumulator_10) -> (i1, !ttg.async.token)  : i32 {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环。

### Lines 475-476
```mlir
475| 	// CHECK: arith.muli {{.*}}ttg.partition = array<i32: 2>}
476|         %off_k = arith.muli %accumulator_15, %c128_i32 {loop.cluster = 2 : i32, loop.stage = 0 : i32} : i32
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 477-482
```mlir
477|         // CHECK: tt.descriptor_load {{.*}}ttg.partition = array<i32: 2>}
478|         %a = tt.descriptor_load %a_desc_0[%off_am, %off_k] {loop.cluster = 2 : i32, loop.stage = 0 : i32} : !tt.tensordesc<128x128xf8E4M3FN, #shared> -> tensor<128x128xf8E4M3FN, #blocked1>
479|         %a_17 = ttg.local_alloc %a {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x128xf8E4M3FN, #blocked1>) -> !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem>
480|         %b = tt.descriptor_load %b_desc_1[%off_bn, %off_k] {loop.cluster = 2 : i32, loop.stage = 0 : i32} : !tt.tensordesc<128x128xf8E4M3FN, #shared> -> tensor<128x128xf8E4M3FN, #blocked1>
481|         %accumulator_18 = ttg.local_alloc %b {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x128xf8E4M3FN, #blocked1>) -> !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem>
482|         %accumulator_19 = ttg.memdesc_trans %accumulator_18 {loop.cluster = 0 : i32, loop.stage = 2 : i32, order = array<i32: 1, 0>} : !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem> -> !ttg.memdesc<128x128xf8E4M3FN, #shared1, #smem>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.descriptor_load, shared/local memory allocation, ttg.memdesc_trans. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.descriptor_load、共享/本地内存分配、ttg.memdesc_trans。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 483-485
```mlir
483|         // CHECK: ttng.tc_gen5_mma {{.*}}ttg.partition = array<i32: 1>}
484|         %accumulator_20 = ttng.tc_gen5_mma %a_17, %accumulator_19, %accumulator[%accumulator_16], %arg11, %true {loop.cluster = 0 : i32, loop.stage = 2 : i32, tt.self_latency = 1 : i32} : !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem>, !ttg.memdesc<128x128xf8E4M3FN, #shared1, #smem>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
485|         scf.yield %true, %accumulator_20 : i1, !ttg.async.token
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining Gen5 tensor-core MMA ops, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 Gen5 张量核 MMA 操作、循环/分支产出值。

### Lines 486-487
```mlir
486|       // CHECK: } {tt.scheduled_max_stage = 2 : i32, ttg.partition = array<i32: 1, 2>, ttg.partition.outputs = [array<i32: 1>, array<i32: 1>]}
487|       } {tt.scheduled_max_stage = 2 : i32}
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 488-497
```mlir
488|       // CHECK-COUNT-4: {ttg.partition = array<i32: 0>}
489|       %accumulator_12, %accumulator_13 = ttng.tmem_load %accumulator[%accumulator_11#1] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
490|       %c = tt.fp_to_fp %accumulator_12, rounding = rtne : tensor<128x128xf32, #blocked> -> tensor<128x128xf8E4M3FN, #blocked>
491|       %c_14 = ttg.convert_layout %c : tensor<128x128xf8E4M3FN, #blocked> -> tensor<128x128xf8E4M3FN, #blocked1>
492|       tt.descriptor_store %c_desc_2[%off_am, %off_bn], %c_14 : !tt.tensordesc<128x128xf8E4M3FN, #shared>, tensor<128x128xf8E4M3FN, #blocked1>
493|     } {tt.num_stages = 3 : i32, tt.warp_specialize}
494|     tt.return
495|   }
496| }
497| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tmem_load, tt.fp_to_fp, layout conversions, tt.descriptor_store, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tmem_load、tt.fp_to_fp、布局转换、tt.descriptor_store、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 498-498
```mlir
498| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 499-500
```mlir
499| 
500| // CHECK-LABEL: attention_persistent_inner_loop_kernel
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: attention_persistent_inner_loop_kernel anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: attention_persistent_inner_loop_kernel 这样的标签用于锚定匹配范围。

### Lines 501-507
```mlir
501| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
502| #blocked1 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
503| #blocked2 = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
504| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 8}>
505| #shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = true, elementBitWidth = 8}>
506| #smem = #ttg.shared_memory
507| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 508-508
```mlir
508| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 509-521
```mlir
509|   tt.func public @attention_persistent_inner_loop_kernel(%desc_q: !tt.tensordesc<128x128xf16, #shared>, %desc_q_0: i32, %desc_q_1: i32, %desc_q_2: i64, %desc_q_3: i64, %desc_k: !tt.tensordesc<128x128xf16, #shared>, %desc_k_4: i32, %desc_k_5: i32, %desc_k_6: i64, %desc_k_7: i64, %desc_v: !tt.tensordesc<128x128xf16, #shared>, %desc_v_8: i32, %desc_v_9: i32, %desc_v_10: i64, %desc_v_11: i64, %desc_acc: !tt.tensordesc<128x128xf16, #shared>, %desc_acc_12: i32, %desc_acc_13: i32, %desc_acc_14: i64, %desc_acc_15: i64, %l_i_ptr: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %m_i_ptr: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %M: i32 {tt.divisibility = 16 : i32}, %N: i32 {tt.divisibility = 16 : i32}, %qk_scale: f32) attributes {noinline = false} {
510|     %false = arith.constant false
511|     %true = arith.constant true
512|     %c1_i32 = arith.constant 1 : i32
513|     %c0_i32 = arith.constant 0 : i32
514|     %c128_i32 = arith.constant 128 : i32
515|     %cst = arith.constant dense<1.000000e+00> : tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
516|     %cst_16 = arith.constant dense<0xFF800000> : tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
517|     %cst_17 = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked>
518|     %prog_id = tt.get_program_id x : i32
519|     %num_sm = tt.get_num_programs x : i32
520|     %num_tiles = arith.divsi %M, %c128_i32 : i32
521|     %tiles_per_sm = arith.divsi %num_tiles, %num_sm : i32
```
**EN:** This function-oriented block defines or enters `attention_persistent_inner_loop_kernel`. Within it, the test exercises constants, tt.func, arith.divsi, program IDs, tt.get_num_programs, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `attention_persistent_inner_loop_kernel` 为核心。测试在其中演示 常量、tt.func、arith.divsi、程序 ID、tt.get_num_programs，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 522-529
```mlir
522|     // CHECK: scf.for
523|     %tile_idx = scf.for %_ = %c0_i32 to %tiles_per_sm step %c1_i32 iter_args(%tile_idx_20 = %prog_id) -> (i32)  : i32 {
524|       %off_m = arith.muli %tile_idx_20, %c128_i32 : i32
525|       %q = tt.descriptor_load %desc_q[%off_m, %c0_i32] : !tt.tensordesc<128x128xf16, #shared> -> tensor<128x128xf16, #blocked2>
526|       %q_21 = ttg.local_alloc %q : (tensor<128x128xf16, #blocked2>) -> !ttg.memdesc<128x128xf16, #shared, #smem>
527|       %qk_22, %qk_23 = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
528|       %acc, %acc_24 = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
529|       %acc_25 = ttng.tmem_store %cst_17, %acc[%acc_24], %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining tensor-memory allocation, structured loops, integer multiplications, tt.descriptor_load, shared/local memory allocation.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 张量内存分配、结构化循环、整数乘法、tt.descriptor_load、共享/本地内存分配。

### Lines 530-535
```mlir
530|       // CHECK: scf.for
531|       %acc_26:4 = scf.for %acc_30 = %c0_i32 to %N step %c128_i32 iter_args(%arg28 = %cst_16, %arg29 = %cst, %qk_31 = %qk_23, %acc_32 = %acc_25) -> (tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked}>>, tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked}>>, !ttg.async.token, !ttg.async.token)  : i32 {
532|         %k = tt.descriptor_load %desc_k[%acc_30, %c0_i32] : !tt.tensordesc<128x128xf16, #shared> -> tensor<128x128xf16, #blocked2>
533|         %k_33 = ttg.local_alloc %k : (tensor<128x128xf16, #blocked2>) -> !ttg.memdesc<128x128xf16, #shared, #smem>
534|         %k_34 = ttg.memdesc_trans %k_33 {order = array<i32: 1, 0>} : !ttg.memdesc<128x128xf16, #shared, #smem> -> !ttg.memdesc<128x128xf16, #shared1, #smem>
535|         %qk_35 = ttng.tc_gen5_mma %q_21, %k_34, %qk_22[%qk_31], %false, %true : !ttg.memdesc<128x128xf16, #shared, #smem>, !ttg.memdesc<128x128xf16, #shared1, #smem>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops, tt.descriptor_load, shared/local memory allocation, ttg.memdesc_trans, Gen5 tensor-core MMA ops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环、tt.descriptor_load、共享/本地内存分配、ttg.memdesc_trans、Gen5 张量核 MMA 操作。

### Lines 536-537
```mlir
536|         // CHECK: tmem_load {{.*}} {ttg.partition = array<i32: 0>}
537|         %qk_36, %qk_37 = ttng.tmem_load %qk_22[%qk_35] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 538-541
```mlir
538|         // CHECK: "softmax_work"{{.*}}ttg.partition = array<i32: 0>}
539|         %acc_47, %p, %next_l_i, %row_max = "softmax_work"(%qk_36, %arg29, %arg28) : (tensor<128x128xf32, #blocked>, tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked}>>, tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked}>>) -> (tensor<128x128xf32, #blocked>, tensor<128x128xf16, #blocked>, tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked}>>, tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked}>>)
540|         %p_53 = ttg.local_alloc %p : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #smem>
541| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 542-551
```mlir
542|         // CHECK-COUNT-3: {ttg.partition = array<i32: 1>}
543|         %acc_48, %acc_49 = ttng.tmem_load %acc[%acc_32] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
544|         %acc_50 = arith.mulf %acc_48, %acc_47 : tensor<128x128xf32, #blocked>
545|         %acc_54 = ttng.tmem_store %acc_50, %acc[%acc_49], %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
546|         %v = tt.descriptor_load %desc_v[%acc_30, %c0_i32] : !tt.tensordesc<128x128xf16, #shared> -> tensor<128x128xf16, #blocked2>
547|         %v_51 = ttg.local_alloc %v : (tensor<128x128xf16, #blocked2>) -> !ttg.memdesc<128x128xf16, #shared, #smem>
548| 
549|         %acc_55 = ttng.tc_gen5_mma %p_53, %v_51, %acc[%acc_54], %true, %true : !ttg.memdesc<128x128xf16, #shared, #smem>, !ttg.memdesc<128x128xf16, #shared, #smem>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
550| 
551|         scf.yield %row_max, %next_l_i, %qk_37, %acc_55 : tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked}>>, tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked}>>, !ttg.async.token, !ttg.async.token
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining ttng.tmem_load, arith.mulf, ttng.tmem_store, tt.descriptor_load, shared/local memory allocation.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 ttng.tmem_load、arith.mulf、ttng.tmem_store、tt.descriptor_load、共享/本地内存分配。

### Lines 552-553
```mlir
552|       // CHECK: } {ttg.partition = array<i32: 0, 1, 2, 3>, ttg.partition.outputs = [array<i32: 0>, array<i32: 0>, array<i32: 2>, array<i32: 1>]}
553|       }
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 554-560
```mlir
554|       // CHECK: arith.addi {{.*}}, {{.*}} {ttg.partition = array<i32: 3>}
555|       %tile_idx_29 = arith.addi %tile_idx_20, %num_sm : i32
556|       scf.yield %tile_idx_29 : i32
557|     } {tt.num_stages = 3 : i32, tt.warp_specialize}
558|     tt.return
559|   }
560| }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining integer additions, loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 整数加法、循环/分支产出值、tt.return。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `--split-input-file`, `--tritongpu-hoist-tmem-alloc`, `--tritongpu-partition-scheduling`, `-allow-unregistered-dialect`
- **CN:** 主要 pass 选项：`--split-input-file`，`--tritongpu-hoist-tmem-alloc`，`--tritongpu-partition-scheduling`，`-allow-unregistered-dialect`
- **EN:** Dominant operations include `arith.constant`, `tt.func`, `scf.yield`, `tt.descriptor_load`, `scf.for`, `ttg.local_alloc`, `tt.return`, `arith.muli`, `ttng.tmem_alloc`, `ttng.tc_gen5_mma`.
- **CN:** 主要操作包括 `arith.constant`、`tt.func`、`scf.yield`、`tt.descriptor_load`、`scf.for`、`ttg.local_alloc`、`tt.return`、`arith.muli`、`ttng.tmem_alloc`、`ttng.tc_gen5_mma`。
- **EN:** The file contains 4 independently testable section(s). Check styles used: CHECK-NEXT x46, CHECK x36, CHECK-COUNT x15, CHECK-LABEL x10. Important labels include @attention_forward, @mma_operand_view, @optimize_broadcast, @no_partitions. It validates both the presence of the desired rewrite and the absence of unwanted residual operations.
- **CN:** 该文件包含 4 个可独立测试的分段。使用的检查类型：CHECK-NEXT ×46，CHECK ×36，CHECK-COUNT ×15，CHECK-LABEL ×10。 关键标签包括 @attention_forward，@mma_operand_view，@optimize_broadcast，@no_partitions。 它同时验证目标重写已经出现，并且不再残留不需要的操作。

## Dependencies / 依赖关系

- **EN:** This test depends on Triton/MLIR textual IR parsing, the pass pipeline named in the RUN lines, and the dialect/type definitions referenced in the file.
- **CN:** 该测试依赖 Triton/MLIR 文本 IR 解析、RUN 行中指定的 pass 流水线，以及文件中引用的方言/类型定义。
- **EN:** `ttg`: TritonGPU layout, memory, and GPU-specific operations.
- **CN:** `ttg`：TritonGPU 布局、内存和 GPU 专用操作。
- **EN:** `tt`: Triton high-level tensor/pointer operations.
- **CN:** `tt`：Triton 高层张量/指针操作。
- **EN:** `arith`: MLIR arithmetic dialect for scalar/tensor math.
- **CN:** `arith`：用于标量/张量运算的 MLIR arithmetic 方言。
- **EN:** `ttng`: Triton Nvidia GPU tensor-memory and MMA operations.
- **CN:** `ttng`：Triton Nvidia GPU 张量内存与 MMA 操作。
- **EN:** `scf`: Structured control flow for loops and conditionals.
- **CN:** `scf`：用于循环与条件分支的结构化控制流。
- **EN:** `FileCheck` is used to match required textual patterns after rewriting/lowering.
- **CN:** `FileCheck` 用于在重写/lower 之后匹配必须出现的文本模式。