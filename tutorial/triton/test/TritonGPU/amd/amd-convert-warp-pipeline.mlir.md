# amd-convert-warp-pipeline.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/amd/amd-convert-warp-pipeline.mlir`
- **EN:** Test for the MLIR/Triton pipeline `-convert-warp-pipeline="gfx-arch=gfx1250", -convert-warp-pipeline="gfx-arch=gfx950"` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `-convert-warp-pipeline="gfx-arch=gfx1250", -convert-warp-pipeline="gfx-arch=gfx950"` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -split-input-file -convert-warp-pipeline="gfx-arch=gfx1250" | FileCheck %s --check-prefixes CHECK,WAVE32`; `// RUN: triton-opt %s -split-input-file -convert-warp-pipeline="gfx-arch=gfx950" | FileCheck %s --check-prefixes CHECK,WAVE64`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -split-input-file -convert-warp-pipeline="gfx-arch=gfx1250" | FileCheck %s --check-prefixes CHECK,WAVE32`；`// RUN: triton-opt %s -split-input-file -convert-warp-pipeline="gfx-arch=gfx950" | FileCheck %s --check-prefixes CHECK,WAVE64`

## Line-by-Line Analysis / 逐行分析

### Lines 1-6
```mlir
1| // RUN: triton-opt %s -split-input-file -convert-warp-pipeline="gfx-arch=gfx1250" | FileCheck %s --check-prefixes CHECK,WAVE32
2| // RUN: triton-opt %s -split-input-file -convert-warp-pipeline="gfx-arch=gfx950" | FileCheck %s --check-prefixes CHECK,WAVE64
3| 
4| // ---- 2-stage pipeline (basic) ----
5| //
6| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -split-input-file -convert-warp-pipeline="gfx-arch=gfx1250" | FileCheck %s --check-prefixes CHECK,WAVE32` ; ` triton-opt %s -split-input-file -convert-warp-pipeline="gfx-arch=gfx950" | FileCheck %s --check-prefixes CHECK,WAVE64` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -split-input-file -convert-warp-pipeline="gfx-arch=gfx1250" | FileCheck %s --check-prefixes CHECK,WAVE32`；` triton-opt %s -split-input-file -convert-warp-pipeline="gfx-arch=gfx950" | FileCheck %s --check-prefixes CHECK,WAVE64`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 7-24
```mlir
 7| tt.func @two_stage_backend(%n: index, %ptr: !tt.ptr<f32>) {
 8|   %c0  = arith.constant 0 : index
 9|   %c1  = arith.constant 1 : index
10|   %v0  = arith.constant 0.0 : f32
11|   %v1  = arith.constant 1.0 : f32
12| 
13| 
14|   scf.for %i = %c0 to %n step %c1 {
15| 
16|     // Stage 0 cluster
17|     scf.execute_region {
18|       tt.store %ptr, %v0 : !tt.ptr<f32>
19|       scf.yield
20|     } {triton.warp_pipeline.stage = "stage0"}
21| 
22|     // Stage 1 cluster
23|     scf.execute_region {
24|       tt.store %ptr, %v1 : !tt.ptr<f32>
```
**EN:** This function-oriented block defines or enters `two_stage_backend`. Within it, the test exercises constants, tt.func, scf.execute_region, masked or vectorized stores, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `two_stage_backend` 为核心。测试在其中演示 常量、tt.func、scf.execute_region、带掩码或向量化的存储、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 25-33
```mlir
25|       scf.yield
26|     } {triton.warp_pipeline.stage = "stage1"}
27| 
28|     scf.yield
29|   } {triton.warp_pipeline.pipelined_for}
30| 
31|   tt.return
32| }
33| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、tt.return。

### Lines 34-39
```mlir
34| // CHECK-LABEL: tt.func @two_stage_backend(
35| // CHECK: %c0 = arith.constant 0 : index
36| // CHECK: %c1 = arith.constant 1 : index
37| // CHECK-NOT: no_inline
38| 
39| // === Pre-loop sync + role setup ===
```
**EN:** This function-oriented block defines or enters `two_stage_backend`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `two_stage_backend` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 40-43
```mlir
40| // CHECK: ttg.barrier local
41| // CHECK: arith.divsi
42| // WAVE64-SAME: %c256
43| // WAVE32-SAME: %c128
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 44-48
```mlir
44| // CHECK: %[[WARPLOW:.+]] = arith.cmpi eq
45| // CHECK: %[[WARPHIGH:.+]] = arith.cmpi ne
46| // CHECK: amdg.cond_barrier %[[WARPHIGH]]
47| 
48| // After conversion, the for body is flattened and cluster barriers inserted.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 49-61
```mlir
49| // CHECK: scf.for
50| // CHECK-NOT:   scf.execute_region
51| // CHECK: rocdl.sched.barrier
52| // CHECK: rocdl.s.barrier
53| // CHECK: rocdl.sched.barrier
54| // CHECK-NOT:   scf.execute_region
55| 
56| // CHECK: amdg.cond_barrier %[[WARPLOW]]
57| // CHECK: tt.return
58| 
59| 
60| // ---- 3-stage pipeline (ensures multiple clusters handled) ----
61| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 62-79
```mlir
62| tt.func @three_stage_backend(%n: index, %ptr0: !tt.ptr<f32>, %ptr1: !tt.ptr<f32>) {
63|   %c0  = arith.constant 0 : index
64|   %c1  = arith.constant 1 : index
65|   %v0  = arith.constant 0.0 : f32
66|   %v1  = arith.constant 1.0 : f32
67|   %v2  = arith.constant 2.0 : f32
68| 
69|   scf.for %i = %c0 to %n step %c1 {
70| 
71|     // Stage 0
72|     scf.execute_region {
73|       tt.store %ptr0, %v0 : !tt.ptr<f32>
74|       scf.yield
75|     } {triton.warp_pipeline.stage = "stage0"}
76| 
77|     // Stage 1
78|     scf.execute_region {
79|       tt.store %ptr0, %v1 : !tt.ptr<f32>
```
**EN:** This function-oriented block defines or enters `three_stage_backend`. Within it, the test exercises constants, tt.func, scf.execute_region, masked or vectorized stores, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `three_stage_backend` 为核心。测试在其中演示 常量、tt.func、scf.execute_region、带掩码或向量化的存储、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 80-94
```mlir
80|       scf.yield
81|     } {triton.warp_pipeline.stage = "stage1"}
82| 
83|     // Stage 2
84|     scf.execute_region {
85|       tt.store %ptr1, %v2 : !tt.ptr<f32>
86|       scf.yield
87|     } {triton.warp_pipeline.stage = "stage2"}
88| 
89|     scf.yield
90|   } {triton.warp_pipeline.pipelined_for}
91| 
92|   tt.return
93| }
94| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, scf.execute_region, masked or vectorized stores, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、scf.execute_region、带掩码或向量化的存储、tt.return。

### Lines 95-112
```mlir
 95| // CHECK-LABEL: tt.func @three_stage_backend(
 96| // CHECK-NOT: no_inline
 97| // CHECK: ttg.barrier local
 98| // CHECK: amdg.cond_barrier
 99| // CHECK: scf.for
100| // CHECK-NOT:   scf.execute_region
101| // CHECK: rocdl.sched.barrier
102| // CHECK: rocdl.s.barrier
103| // CHECK: rocdl.sched.barrier
104| // CHECK: amdg.cond_barrier
105| // CHECK: tt.return
106| 
107| 
108| // -- 8-stage pipeline dependency check ----
109| //
110| // 0: <lload>-<dot  >-<lload>-<dot  >-<lload>-<dot  >-<lstore>-<dot  >|<lload>-<dot  >-<lload>-<dot  >
111| // 1:         <lload>-<dot  >-<lload>-<dot  >-<lload>*<dot  >-<lstore>*<dot  >|<lload>-<dot  >-<lload>-<dot>
112| // < > : a pipeline cluster, relevant operation in it.
```
**EN:** This function-oriented block defines or enters `three_stage_backend`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `three_stage_backend` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 113-121
```mlir
113| // -  : pipeline border with s.barrier
114| // *  : pipeline border with ttg.barrier local
115| // |  : end of the loop, begins next iteration.
116| //
117| // Dependency comes from the second warp (deferred) to the first warp,
118| // In this case, local_load(lload) and local_store(lstore) access the same allocation
119| // we need to insert wait after lload/lstore from the second warp
120| // and just before lstore/lload in the first warp, that is annotated as (*) above
121| //
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 122-139
```mlir
122| // CHECK-LABEL: tt.func public @eight_stage_dependency
123| // CHECK-NOT: no_inline
124| // CHECK: ttg.barrier local
125| // CHECK: amdg.cond_barrier
126| // CHECK: scf.for
127| // CHECK-COUNT-2: local_load
128| // CHECK: s.barrier
129| // CHECK: tt.dot
130| // CHECK: s.barrier
131| // CHECK-COUNT-2: local_load
132| // CHECK: s.barrier
133| // CHECK: tt.dot
134| // CHECK: s.barrier
135| // CHECK-COUNT-4: local_load
136| // CHECK: ttg.barrier local
137| // CHECK: tt.dot
138| // CHECK: s.barrier
139| // CHECK-COUNT-2: local_store
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: tt.func public @eight_stage_dependency anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: tt.func public @eight_stage_dependency 这样的标签用于锚定匹配范围。

### Lines 140-145
```mlir
140| // CHECK: ttg.barrier local
141| // CHECK: tt.dot
142| // CHECK: s.barrier
143| // CHECK: scf.yield
144| // CHECK: amdg.cond_barrier
145| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 146-151
```mlir
146| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 8], warpsPerCTA = [8, 1], order = [1, 0]}>
147| #blocked1 = #ttg.blocked<{sizePerThread = [8, 1], threadsPerWarp = [8, 8], warpsPerCTA = [1, 8], order = [0, 1]}>
148| #mma = #ttg.amd_mfma<{version = 3, warpsPerCTA = [2, 4], instrShape = [16, 16, 16], isTransposed = true}>
149| #shared = #ttg.swizzled_shared<{vec = 4, perPhase = 1, maxPhase = 16, order = [1, 0]}>
150| #shared1 = #ttg.swizzled_shared<{vec = 4, perPhase = 1, maxPhase = 16, order = [0, 1]}>
151| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 152-152
```mlir
152| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 153-170
```mlir
153|   tt.func public @eight_stage_dependency(%arg0: i32, %arg1: i32, %arg2: i32, %arg3: tensor<256x256xf32, #mma>, %arg4: tensor<64x256xi32, #blocked>, %arg5: tensor<256x64xi32, #blocked1>, %arg6: tensor<256x64x!tt.ptr<f16>, #blocked1>, %arg7: tensor<64x256x!tt.ptr<f16>, #blocked>, %arg8: !ttg.memdesc<256x64xf16, #shared, #smem, mutable>, %arg9: !ttg.memdesc<64x256xf16, #shared1, #smem, mutable>) {
154|     %0 = ttg.local_alloc : () -> !ttg.memdesc<1x256x64xf16, #shared, #smem, mutable>
155|     %1 = ttg.local_alloc : () -> !ttg.memdesc<1x64x256xf16, #shared1, #smem, mutable>
156|     %2:6 = scf.for %arg10 = %arg0 to %arg1 step %arg2 iter_args(%arg11 = %arg3, %arg12 = %arg6, %arg13 = %arg7, %arg14 = %arg0, %arg15 = %arg8, %arg16 = %arg9) -> (tensor<256x256xf32, #mma>, tensor<256x64x!tt.ptr<f16>, #blocked1>, tensor<64x256x!tt.ptr<f16>, #blocked>, i32, !ttg.memdesc<256x64xf16, #shared, #smem, mutable>, !ttg.memdesc<64x256xf16, #shared1, #smem, mutable>)  : i32 {
157|       %3:5 = scf.execute_region -> (tensor<256x64x!tt.ptr<f16>, #blocked1>, tensor<256x64xf16, #blocked1>, tensor<64x256x!tt.ptr<f16>, #blocked>, tensor<256x16xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>, tensor<16x256xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>) no_inline {
158|         %11 = tt.addptr %arg12, %arg5 : tensor<256x64x!tt.ptr<f16>, #blocked1>, tensor<256x64xi32, #blocked1>
159|         %12 = tt.load %11 : tensor<256x64x!tt.ptr<f16>, #blocked1>
160|         %13 = tt.addptr %arg13, %arg4 : tensor<64x256x!tt.ptr<f16>, #blocked>, tensor<64x256xi32, #blocked>
161|         %14 = ttg.memdesc_subslice %arg15[0, 0] : !ttg.memdesc<256x64xf16, #shared, #smem, mutable> -> !ttg.memdesc<256x16xf16, #shared, #smem, mutable, 256x64>
162|         %15 = ttg.local_load %14 : !ttg.memdesc<256x16xf16, #shared, #smem, mutable, 256x64> -> tensor<256x16xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>
163|         %16 = ttg.memdesc_subslice %arg16[0, 0] : !ttg.memdesc<64x256xf16, #shared1, #smem, mutable> -> !ttg.memdesc<16x256xf16, #shared1, #smem, mutable, 64x256>
164|         %17 = ttg.local_load %16 : !ttg.memdesc<16x256xf16, #shared1, #smem, mutable, 64x256> -> tensor<16x256xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>
165|         scf.yield %11, %12, %13, %15, %17 : tensor<256x64x!tt.ptr<f16>, #blocked1>, tensor<256x64xf16, #blocked1>, tensor<64x256x!tt.ptr<f16>, #blocked>, tensor<256x16xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>, tensor<16x256xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>
166|       } {triton.warp_pipeline.stage = "stage"}
167|       %4 = scf.execute_region -> tensor<256x256xf32, #mma> no_inline {
168|         %11 = tt.dot %3#3, %3#4, %arg11 : tensor<256x16xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>> * tensor<16x256xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>> -> tensor<256x256xf32, #mma>
169|         scf.yield %11 : tensor<256x256xf32, #mma>
170|       } {triton.warp_pipeline.stage = "stage"}
```
**EN:** This function-oriented block defines or enters `eight_stage_dependency`. Within it, the test exercises tt.func, shared/local memory allocation, scf.execute_region, pointer arithmetic, ttg.memdesc_subslice, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `eight_stage_dependency` 为核心。测试在其中演示 tt.func、共享/本地内存分配、scf.execute_region、指针算术、ttg.memdesc_subslice，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 171-188
```mlir
171|       %5:3 = scf.execute_region -> (tensor<64x256xf16, #blocked>, tensor<256x16xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>, tensor<16x256xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>) no_inline {
172|         %11 = tt.load %3#2 : tensor<64x256x!tt.ptr<f16>, #blocked>
173|         %12 = ttg.memdesc_subslice %arg15[0, 16] : !ttg.memdesc<256x64xf16, #shared, #smem, mutable> -> !ttg.memdesc<256x16xf16, #shared, #smem, mutable, 256x64>
174|         %13 = ttg.local_load %12 : !ttg.memdesc<256x16xf16, #shared, #smem, mutable, 256x64> -> tensor<256x16xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>
175|         %14 = ttg.memdesc_subslice %arg16[16, 0] : !ttg.memdesc<64x256xf16, #shared1, #smem, mutable> -> !ttg.memdesc<16x256xf16, #shared1, #smem, mutable, 64x256>
176|         %15 = ttg.local_load %14 : !ttg.memdesc<16x256xf16, #shared1, #smem, mutable, 64x256> -> tensor<16x256xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>
177|         scf.yield %11, %13, %15 : tensor<64x256xf16, #blocked>, tensor<256x16xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>, tensor<16x256xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>
178|       } {triton.warp_pipeline.stage = "stage"}
179|       %6 = scf.execute_region -> tensor<256x256xf32, #mma> no_inline {
180|         %11 = tt.dot %5#1, %5#2, %4 : tensor<256x16xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>> * tensor<16x256xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>> -> tensor<256x256xf32, #mma>
181|         scf.yield %11 : tensor<256x256xf32, #mma>
182|       } {triton.warp_pipeline.stage = "stage"}
183|       %7:4 = scf.execute_region -> (tensor<256x16xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>, tensor<16x256xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>, tensor<256x16xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>, tensor<16x256xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>) no_inline {
184|         %11 = ttg.memdesc_subslice %arg15[0, 32] : !ttg.memdesc<256x64xf16, #shared, #smem, mutable> -> !ttg.memdesc<256x16xf16, #shared, #smem, mutable, 256x64>
185|         %12 = ttg.local_load %11 : !ttg.memdesc<256x16xf16, #shared, #smem, mutable, 256x64> -> tensor<256x16xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>
186|         %13 = ttg.memdesc_subslice %arg16[32, 0] : !ttg.memdesc<64x256xf16, #shared1, #smem, mutable> -> !ttg.memdesc<16x256xf16, #shared1, #smem, mutable, 64x256>
187|         %14 = ttg.local_load %13 : !ttg.memdesc<16x256xf16, #shared1, #smem, mutable, 64x256> -> tensor<16x256xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>
188|         %15 = ttg.memdesc_subslice %arg15[0, 48] : !ttg.memdesc<256x64xf16, #shared, #smem, mutable> -> !ttg.memdesc<256x16xf16, #shared, #smem, mutable, 256x64>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining ttg.memdesc_subslice, local/shared memory loads, scf.execute_region, loop/if yielded values, masked or vectorized loads.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 ttg.memdesc_subslice、本地/共享内存加载、scf.execute_region、循环/分支产出值、带掩码或向量化的加载。

### Lines 189-206
```mlir
189|         %16 = ttg.local_load %15 : !ttg.memdesc<256x16xf16, #shared, #smem, mutable, 256x64> -> tensor<256x16xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>
190|         %17 = ttg.memdesc_subslice %arg16[48, 0] : !ttg.memdesc<64x256xf16, #shared1, #smem, mutable> -> !ttg.memdesc<16x256xf16, #shared1, #smem, mutable, 64x256>
191|         %18 = ttg.local_load %17 : !ttg.memdesc<16x256xf16, #shared1, #smem, mutable, 64x256> -> tensor<16x256xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>
192|         scf.yield %12, %14, %16, %18 : tensor<256x16xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>, tensor<16x256xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>, tensor<256x16xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>, tensor<16x256xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>
193|       } {triton.warp_pipeline.stage = "stage"}
194|       %8 = scf.execute_region -> tensor<256x256xf32, #mma> no_inline {
195|         %11 = tt.dot %7#0, %7#1, %6 : tensor<256x16xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>> * tensor<16x256xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>> -> tensor<256x256xf32, #mma>
196|         scf.yield %11 : tensor<256x256xf32, #mma>
197|       } {triton.warp_pipeline.stage = "stage"}
198|       %9:3 = scf.execute_region -> (i32, !ttg.memdesc<256x64xf16, #shared, #smem, mutable>, !ttg.memdesc<64x256xf16, #shared1, #smem, mutable>) no_inline {
199|         %11 = arith.addi %arg14, %arg2 : i32
200|         %12 = arith.cmpi slt, %11, %arg2 : i32
201|         %13 = arith.select %12, %11, %arg0 : i32
202|         %14 = ttg.memdesc_index %0[%13] : !ttg.memdesc<1x256x64xf16, #shared, #smem, mutable> -> !ttg.memdesc<256x64xf16, #shared, #smem, mutable>
203|         ttg.local_store %3#1, %14 : tensor<256x64xf16, #blocked1> -> !ttg.memdesc<256x64xf16, #shared, #smem, mutable>
204|         %15 = ttg.memdesc_index %1[%13] : !ttg.memdesc<1x64x256xf16, #shared1, #smem, mutable> -> !ttg.memdesc<64x256xf16, #shared1, #smem, mutable>
205|         ttg.local_store %5#0, %15 : tensor<64x256xf16, #blocked> -> !ttg.memdesc<64x256xf16, #shared1, #smem, mutable>
206|         scf.yield %13, %14, %15 : i32, !ttg.memdesc<256x64xf16, #shared, #smem, mutable>, !ttg.memdesc<64x256xf16, #shared1, #smem, mutable>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, local/shared memory loads, scf.execute_region, ttg.memdesc_index, local/shared memory stores.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、本地/共享内存加载、scf.execute_region、ttg.memdesc_index、本地/共享内存存储。

### Lines 207-222
```mlir
207|       } {triton.warp_pipeline.stage = "stage"}
208|       %10 = scf.execute_region -> tensor<256x256xf32, #mma> no_inline {
209|         %11 = tt.dot %7#2, %7#3, %8 : tensor<256x16xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>> * tensor<16x256xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>> -> tensor<256x256xf32, #mma>
210|         scf.yield %11 : tensor<256x256xf32, #mma>
211|       } {triton.warp_pipeline.stage = "stage"}
212|       scf.yield %10, %3#0, %3#2, %9#0, %9#1, %9#2 : tensor<256x256xf32, #mma>, tensor<256x64x!tt.ptr<f16>, #blocked1>, tensor<64x256x!tt.ptr<f16>, #blocked>, i32, !ttg.memdesc<256x64xf16, #shared, #smem, mutable>, !ttg.memdesc<64x256xf16, #shared1, #smem, mutable>
213|     } {triton.warp_pipeline.pipelined_for}
214|     ttg.local_dealloc %0 : !ttg.memdesc<1x256x64xf16, #shared, #smem, mutable>
215|     ttg.local_dealloc %1 : !ttg.memdesc<1x64x256xf16, #shared1, #smem, mutable>
216|     tt.return
217|   }
218| }
219| 
220| // -- Triple buffered 2-stage pipeline dependency check ----
221| // Currently little conservative, there could be more chance to optimize local_wait
222| //
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, ttg.local_dealloc, scf.execute_region, dot-product or MMA-style math, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、ttg.local_dealloc、scf.execute_region、点积或 MMA 风格计算、tt.return。

### Lines 223-231
```mlir
223| // CHECK-LABEL: tt.func public @triple_buf_2stage
224| // CHECK-NOT: no_inline
225| // CHECK: ttg.barrier local
226| // CHECK: amdg.cond_barrier
227| // CHECK: scf.for
228| // CHECK-COUNT-2: local_load
229| // CHECK: async_copy_global_to_local
230| 
231| // pre-inserted wait should be preserved.
```
**EN:** This function-oriented block defines or enters `triple_buf_2stage`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `triple_buf_2stage` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 232-240
```mlir
232| // CHECK: rocdl.sched.barrier
233| // CHECK: async_wait
234| // CHECK: rocdl.sched.barrier
235| 
236| // CHECK: async_copy_global_to_local
237| // CHECK: ttg.barrier local
238| // CHECK: scf.yield
239| // CHECK: amdg.cond_barrier
240| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 241-246
```mlir
241| #linear = #ttg.linear<{register = [[1, 0], [2, 0], [4, 0], [0, 4]], lane = [[8, 0], [16, 0], [32, 0], [64, 0], [128, 0], [0, 16]], warp = [[0, 1], [0, 2], [0, 8]], block = []}>
242| #linear1 = #ttg.linear<{register = [[0, 1], [0, 2], [0, 4], [4, 0]], lane = [[0, 8], [0, 16], [0, 32], [0, 64], [0, 128], [16, 0]], warp = [[1, 0], [2, 0], [8, 0]], block = []}>
243| #mma2 = #ttg.amd_mfma<{version = 4, warpsPerCTA = [2, 4], instrShape = [16, 16, 32], isTransposed = true}>
244| #shrd_a = #ttg.padded_shared<[512:+16] {offset = [[1, 0], [2, 0], [4, 0], [8, 0], [16, 0], [32, 0], [64, 0], [128, 0], [0, 16], [0, 1], [0, 2], [0, 8], [0, 4]], block = []}>
245| #shrd1 = #ttg.padded_shared<[512:+16] {offset = [[0, 1], [0, 2], [0, 4], [0, 8], [0, 16], [0, 32], [0, 64], [0, 128], [16, 0], [1, 0], [2, 0], [8, 0], [4, 0]], block = []}>
246| #shmem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 247-247
```mlir
247| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 248-265
```mlir
248|   tt.func public @triple_buf_2stage(%arg0: i32, %arg1: i32, %arg2: i32, %arg3: i32, %arg4: tensor<256x256xf32, #mma2>, %arg5: i32, %arg6: i32, %arg7: tensor<256x32xi32, #linear>, %arg8: tensor<32x256xi32, #linear1>, %arg9: !ttg.memdesc<256x32xbf16, #shrd_a, #shmem, mutable>, %arg10: !ttg.memdesc<32x256xbf16, #shrd1, #shmem, mutable>, %arg11: !ttg.memdesc<256x32xbf16, #shrd_a, #shmem, mutable>, %arg12: !ttg.memdesc<32x256xbf16, #shrd1, #shmem, mutable>, %arg13: !ttg.async.token, %arg14: !ttg.async.token, %arg15: !ttg.async.token, %arg16: tensor<256x32x!tt.ptr<bf16>, #linear>, %arg17: tensor<32x256x!tt.ptr<bf16>, #linear1>, %arg18: tensor<256xi64, #ttg.slice<{dim = 1, parent = #mma2}>>, %arg19: tensor<256xi64, #ttg.slice<{dim = 0, parent = #mma2}>>, %arg20: i64, %arg21: i64, %arg22: !tt.ptr<bf16>, %arg23: i32) attributes {noinline = false} {
249|     %0 = ttg.local_alloc : () -> !ttg.memdesc<3x256x32xbf16, #shrd_a, #shmem, mutable>
250|     %1 = ttg.local_alloc : () -> !ttg.memdesc<3x32x256xbf16, #shrd1, #shmem, mutable>
251|     %2:11 = scf.for %arg24 = %arg0 to %arg6 step %arg1 iter_args(%arg25 = %arg4, %arg26 = %arg1, %arg27 = %arg9, %arg28 = %arg11, %arg29 = %arg13, %arg30 = %arg10, %arg31 = %arg12, %arg32 = %arg14, %arg33 = %arg15, %arg34 = %arg16, %arg35 = %arg17) -> (tensor<256x256xf32, #mma2>, i32, !ttg.memdesc<256x32xbf16, #shrd_a, #shmem, mutable>, !ttg.memdesc<256x32xbf16, #shrd_a, #shmem, mutable>, !ttg.async.token, !ttg.memdesc<32x256xbf16, #shrd1, #shmem, mutable>, !ttg.memdesc<32x256xbf16, #shrd1, #shmem, mutable>, !ttg.async.token, !ttg.async.token, tensor<256x32x!tt.ptr<bf16>, #linear>, tensor<32x256x!tt.ptr<bf16>, #linear1>)  : i32 {
252|       %32:8 = scf.execute_region -> (tensor<256x32x!tt.ptr<bf16>, #linear>, tensor<32x256x!tt.ptr<bf16>, #linear1>, i32, !ttg.memdesc<256x32xbf16, #shrd_a, #shmem, mutable>, !ttg.memdesc<32x256xbf16, #shrd1, #shmem, mutable>, tensor<256x32xbf16, #ttg.dot_op<{opIdx = 0, parent = #mma2, kWidth = 8}>>, tensor<32x256xbf16, #ttg.dot_op<{opIdx = 1, parent = #mma2, kWidth = 8}>>, !ttg.async.token) no_inline {
253|         %35 = tt.addptr %arg34, %arg7 : tensor<256x32x!tt.ptr<bf16>, #linear>, tensor<256x32xi32, #linear>
254|         %36 = tt.addptr %arg35, %arg8 : tensor<32x256x!tt.ptr<bf16>, #linear1>, tensor<32x256xi32, #linear1>
255|         %37 = arith.addi %arg26, %arg1 : i32
256|         %38 = arith.cmpi slt, %37, %arg3 : i32
257|         %39 = arith.select %38, %37, %arg0 : i32
258|         %40 = ttg.memdesc_index %0[%39] : !ttg.memdesc<3x256x32xbf16, #shrd_a, #shmem, mutable> -> !ttg.memdesc<256x32xbf16, #shrd_a, #shmem, mutable>
259|         %41 = ttg.memdesc_index %1[%39] : !ttg.memdesc<3x32x256xbf16, #shrd1, #shmem, mutable> -> !ttg.memdesc<32x256xbf16, #shrd1, #shmem, mutable>
260|         %42 = ttg.local_load %arg27 token %arg29 : !ttg.memdesc<256x32xbf16, #shrd_a, #shmem, mutable> -> tensor<256x32xbf16, #ttg.dot_op<{opIdx = 0, parent = #mma2, kWidth = 8}>>
261|         %43 = ttg.local_load %arg30 token %arg29 : !ttg.memdesc<32x256xbf16, #shrd1, #shmem, mutable> -> tensor<32x256xbf16, #ttg.dot_op<{opIdx = 1, parent = #mma2, kWidth = 8}>>
262|         %44 = ttg.async_copy_global_to_local %35, %40 : tensor<256x32x!tt.ptr<bf16>, #linear> -> <256x32xbf16, #shrd_a, #shmem, mutable>
263|         %45 = ttg.async_commit_group tokens %44
264|         scf.yield %35, %36, %39, %40, %41, %42, %43, %45 : tensor<256x32x!tt.ptr<bf16>, #linear>, tensor<32x256x!tt.ptr<bf16>, #linear1>, i32, !ttg.memdesc<256x32xbf16, #shrd_a, #shmem, mutable>, !ttg.memdesc<32x256xbf16, #shrd1, #shmem, mutable>, tensor<256x32xbf16, #ttg.dot_op<{opIdx = 0, parent = #mma2, kWidth = 8}>>, tensor<32x256xbf16, #ttg.dot_op<{opIdx = 1, parent = #mma2, kWidth = 8}>>, !ttg.async.token
265|       } {triton.warp_pipeline.stage = "stage"}
```
**EN:** This function-oriented block defines or enters `triple_buf_2stage`. Within it, the test exercises tt.func, shared/local memory allocation, pointer arithmetic, ttg.memdesc_index, local/shared memory loads, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `triple_buf_2stage` 为核心。测试在其中演示 tt.func、共享/本地内存分配、指针算术、ttg.memdesc_index、本地/共享内存加载，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 266-283
```mlir
266|       %33 = ttg.async_wait %arg32, %arg33 {num = 0 : i32}
267|       %34:2 = scf.execute_region -> (!ttg.async.token, tensor<256x256xf32, #mma2>) no_inline {
268|         %35 = ttg.async_copy_global_to_local %32#1, %32#4 : tensor<32x256x!tt.ptr<bf16>, #linear1> -> <32x256xbf16, #shrd1, #shmem, mutable>
269|         %36 = ttg.async_commit_group tokens %35
270|         %37 = tt.dot %32#5, %32#6, %arg25 : tensor<256x32xbf16, #ttg.dot_op<{opIdx = 0, parent = #mma2, kWidth = 8}>> * tensor<32x256xbf16, #ttg.dot_op<{opIdx = 1, parent = #mma2, kWidth = 8}>> -> tensor<256x256xf32, #mma2>
271|         scf.yield %36, %37 : !ttg.async.token, tensor<256x256xf32, #mma2>
272|       } {triton.warp_pipeline.stage = "stage"}
273|       scf.yield %34#1, %32#2, %arg28, %32#3, %33, %arg31, %32#4, %32#7, %34#0, %32#0, %32#1 : tensor<256x256xf32, #mma2>, i32, !ttg.memdesc<256x32xbf16, #shrd_a, #shmem, mutable>, !ttg.memdesc<256x32xbf16, #shrd_a, #shmem, mutable>, !ttg.async.token, !ttg.memdesc<32x256xbf16, #shrd1, #shmem, mutable>, !ttg.memdesc<32x256xbf16, #shrd1, #shmem, mutable>, !ttg.async.token, !ttg.async.token, tensor<256x32x!tt.ptr<bf16>, #linear>, tensor<32x256x!tt.ptr<bf16>, #linear1>
274|     } {triton.warp_pipeline.pipelined_for}
275|     %3 = arith.cmpi sge, %arg5, %arg1 : i32
276|     %4 = arith.cmpi sge, %arg5, %arg2 : i32
277|     %5 = ttg.local_load %2#2 token %2#4 : !ttg.memdesc<256x32xbf16, #shrd_a, #shmem, mutable> -> tensor<256x32xbf16, #ttg.dot_op<{opIdx = 0, parent = #mma2, kWidth = 8}>>
278|     %6 = ttg.local_load %2#5 token %2#4 : !ttg.memdesc<32x256xbf16, #shrd1, #shmem, mutable> -> tensor<32x256xbf16, #ttg.dot_op<{opIdx = 1, parent = #mma2, kWidth = 8}>>
279|     %7 = scf.if %3 -> (tensor<256x256xf32, #mma2>) {
280|       %32 = tt.dot %5, %6, %2#0 : tensor<256x32xbf16, #ttg.dot_op<{opIdx = 0, parent = #mma2, kWidth = 8}>> * tensor<32x256xbf16, #ttg.dot_op<{opIdx = 1, parent = #mma2, kWidth = 8}>> -> tensor<256x256xf32, #mma2>
281|       scf.yield %32 : tensor<256x256xf32, #mma2>
282|     } else {
283|       scf.yield %2#0 : tensor<256x256xf32, #mma2>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, dot-product or MMA-style math, integer comparisons, local/shared memory loads, async wait synchronization.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、点积或 MMA 风格计算、整数比较、本地/共享内存加载、异步等待同步。

### Lines 284-301
```mlir
284|     }
285|     %8 = ttg.async_wait %2#7, %2#8 {num = 0 : i32}
286|     %9 = arith.select %3, %7, %2#0 : tensor<256x256xf32, #mma2>
287|     %10 = ttg.local_load %2#3 token %8 : !ttg.memdesc<256x32xbf16, #shrd_a, #shmem, mutable> -> tensor<256x32xbf16, #ttg.dot_op<{opIdx = 0, parent = #mma2, kWidth = 8}>>
288|     %11 = ttg.local_load %2#6 token %8 : !ttg.memdesc<32x256xbf16, #shrd1, #shmem, mutable> -> tensor<32x256xbf16, #ttg.dot_op<{opIdx = 1, parent = #mma2, kWidth = 8}>>
289|     %12 = scf.if %4 -> (tensor<256x256xf32, #mma2>) {
290|       %32 = tt.dot %10, %11, %9 : tensor<256x32xbf16, #ttg.dot_op<{opIdx = 0, parent = #mma2, kWidth = 8}>> * tensor<32x256xbf16, #ttg.dot_op<{opIdx = 1, parent = #mma2, kWidth = 8}>> -> tensor<256x256xf32, #mma2>
291|       scf.yield %32 : tensor<256x256xf32, #mma2>
292|     } else {
293|       scf.yield %9 : tensor<256x256xf32, #mma2>
294|     }
295|     %13 = arith.select %4, %12, %9 : tensor<256x256xf32, #mma2>
296|     ttg.local_dealloc %1 : !ttg.memdesc<3x32x256xbf16, #shrd1, #shmem, mutable>
297|     ttg.local_dealloc %0 : !ttg.memdesc<3x256x32xbf16, #shrd_a, #shmem, mutable>
298|     tt.return
299|   }
300| }
301| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining arith.select, local/shared memory loads, loop/if yielded values, ttg.local_dealloc, async wait synchronization.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 arith.select、本地/共享内存加载、循环/分支产出值、ttg.local_dealloc、异步等待同步。

### Lines 302-305
```mlir
302| 
303| // -- Negative: no total_stages → pass should not touch the loop ----
304| //
305| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 306-321
```mlir
306| tt.func @no_total_stages(%n: index, %ptr: !tt.ptr<f32>) {
307|   %c0  = arith.constant 0 : index
308|   %c1  = arith.constant 1 : index
309|   %v0  = arith.constant 3.0 : f32
310| 
311|   scf.for %i = %c0 to %n step %c1 {
312|     scf.execute_region {
313|       tt.store %ptr, %v0 : !tt.ptr<f32>
314|       scf.yield
315|     }
316|     scf.yield
317|   }
318| 
319|   tt.return
320| }
321| 
```
**EN:** This function-oriented block defines or enters `no_total_stages`. Within it, the test exercises constants, tt.func, loop/if yielded values, structured loops, scf.execute_region, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `no_total_stages` 为核心。测试在其中演示 常量、tt.func、循环/分支产出值、结构化循环、scf.execute_region，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 322-328
```mlir
322| // CHECK-LABEL: tt.func @no_total_stages(
323| // CHECK-NOT: ttg.barrier
324| // CHECK-NOT: amdg.cond_barrier
325| // CHECK: scf.for
326| // CHECK:   scf.execute_region
327| // CHECK: tt.return
328| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: tt.func @no_total_stages( anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: tt.func @no_total_stages( 这样的标签用于锚定匹配范围。

### Lines 329-329
```mlir
329| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 330-332
```mlir
330| 
331| // ---- Priority reset: stages without priority reset to 0 when others have it ----
332| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 333-333
```mlir
333| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 334-351
```mlir
334|   tt.func @priority_reset_between_stages(%n: index, %ptr: !tt.ptr<f32>) {
335|     %c0  = arith.constant 0 : index
336|     %c1  = arith.constant 1 : index
337|     %v0  = arith.constant 0.0 : f32
338|     %v1  = arith.constant 1.0 : f32
339| 
340|     scf.for %i = %c0 to %n step %c1 {
341|       // Stage 0 - has priority 3
342|       scf.execute_region {
343|         tt.store %ptr, %v0 : !tt.ptr<f32>
344|         scf.yield
345|       } {triton.warp_pipeline.stage = "load", triton.warp_pipeline.priority = 3 : i32}
346| 
347|       // Stage 1 - no priority, should reset to 0
348|       scf.execute_region {
349|         tt.store %ptr, %v1 : !tt.ptr<f32>
350|         scf.yield
351|       } {triton.warp_pipeline.stage = "compute"}
```
**EN:** This function-oriented block defines or enters `priority_reset_between_stages`. Within it, the test exercises constants, tt.func, scf.execute_region, masked or vectorized stores, loop/if yielded values, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `priority_reset_between_stages` 为核心。测试在其中演示 常量、tt.func、scf.execute_region、带掩码或向量化的存储、循环/分支产出值，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 352-359
```mlir
352| 
353|       scf.yield
354|     } {triton.warp_pipeline.pipelined_for}
355| 
356|     tt.return
357|   }
358| }
359| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、tt.return。

### Lines 360-361
```mlir
360| // CHECK-LABEL: tt.func @priority_reset_between_stages
361| // Before loop: priority for first cluster
```
**EN:** This function-oriented block defines or enters `priority_reset_between_stages`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `priority_reset_between_stages` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 362-364
```mlir
362| // CHECK: rocdl.s.setprio 3
363| // CHECK: scf.for
364| // Inside loop: setprio 0 for second cluster, then setprio 3 for first cluster
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 365-367
```mlir
365| // CHECK: rocdl.s.setprio 0
366| // CHECK: rocdl.s.setprio 3
367| // After loop: reset to 0
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 368-371
```mlir
368| // CHECK: rocdl.s.setprio 0
369| // CHECK: amdg.cond_barrier
370| // CHECK: tt.return
371| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 372-372
```mlir
372| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 373-375
```mlir
373| 
374| // ---- No priority: no setprio emitted when no stage uses priority ----
375| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 376-376
```mlir
376| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 377-394
```mlir
377|   tt.func @no_priority_no_setprio(%n: index, %ptr: !tt.ptr<f32>) {
378|     %c0  = arith.constant 0 : index
379|     %c1  = arith.constant 1 : index
380|     %v0  = arith.constant 0.0 : f32
381|     %v1  = arith.constant 1.0 : f32
382| 
383|     scf.for %i = %c0 to %n step %c1 {
384|       scf.execute_region {
385|         tt.store %ptr, %v0 : !tt.ptr<f32>
386|         scf.yield
387|       } {triton.warp_pipeline.stage = "stage0"}
388| 
389|       scf.execute_region {
390|         tt.store %ptr, %v1 : !tt.ptr<f32>
391|         scf.yield
392|       } {triton.warp_pipeline.stage = "stage1"}
393| 
394|       scf.yield
```
**EN:** This function-oriented block defines or enters `no_priority_no_setprio`. Within it, the test exercises constants, loop/if yielded values, tt.func, scf.execute_region, masked or vectorized stores, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `no_priority_no_setprio` 为核心。测试在其中演示 常量、循环/分支产出值、tt.func、scf.execute_region、带掩码或向量化的存储，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 395-400
```mlir
395|     } {triton.warp_pipeline.pipelined_for}
396| 
397|     tt.return
398|   }
399| }
400| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 401-406
```mlir
401| // CHECK-LABEL: tt.func @no_priority_no_setprio
402| // CHECK: scf.for
403| // CHECK-NOT: rocdl.s.setprio
404| // CHECK: amdg.cond_barrier
405| // CHECK: tt.return
406| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: tt.func @no_priority_no_setprio anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: tt.func @no_priority_no_setprio 这样的标签用于锚定匹配范围。

### Lines 407-407
```mlir
407| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 408-410
```mlir
408| 
409| // ---- amdg.async_wait recognized as a valid barrier between stages ----
410| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 411-411
```mlir
411| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 412-429
```mlir
412|   tt.func @async_wait_between_stages(%n: index, %ptr: !tt.ptr<f32>) {
413|     %c0  = arith.constant 0 : index
414|     %c1  = arith.constant 1 : index
415|     %v0  = arith.constant 0.0 : f32
416|     %v1  = arith.constant 1.0 : f32
417| 
418|     scf.for %i = %c0 to %n step %c1 {
419|       scf.execute_region {
420|         tt.store %ptr, %v0 : !tt.ptr<f32>
421|         scf.yield
422|       } {triton.warp_pipeline.stage = "stage1"}
423| 
424|       // amdg.async_wait sits between stages and must be recognized as a barrier.
425|       amdg.async_wait {num_inst = 0 : i32}
426| 
427|       scf.execute_region {
428|         tt.store %ptr, %v1 : !tt.ptr<f32>
429|         scf.yield
```
**EN:** This function-oriented block defines or enters `async_wait_between_stages`. Within it, the test exercises constants, tt.func, scf.execute_region, masked or vectorized stores, loop/if yielded values, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_wait_between_stages` 为核心。测试在其中演示 常量、tt.func、scf.execute_region、带掩码或向量化的存储、循环/分支产出值，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 430-439
```mlir
430|       } {triton.warp_pipeline.stage = "stage2"}
431| 
432|       scf.yield
433|     } {triton.warp_pipeline.pipelined_for}
434| 
435|     tt.return
436|   }
437| }
438| 
439| // The pass should succeed (not bail out) and produce barriers.
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、tt.return。

### Lines 440-448
```mlir
440| // CHECK-LABEL: tt.func @async_wait_between_stages
441| // CHECK: ttg.barrier local
442| // CHECK: amdg.cond_barrier
443| // CHECK: scf.for
444| // CHECK-NOT: scf.execute_region
445| // CHECK: rocdl.sched.barrier
446| // CHECK: amdg.cond_barrier
447| // CHECK: tt.return
448| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: tt.func @async_wait_between_stages anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: tt.func @async_wait_between_stages 这样的标签用于锚定匹配范围。

### Lines 449-449
```mlir
449| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 450-467
```mlir
450| 
451| // ---- Back-to-back: cross-pipeline LDS dep covered by A's wrap-around ----
452| //
453| // Both loops access the same shared buffer (read + write).  Loop 1's
454| // stage1 writes smem and loop 2's stage0 reads it — a cross-pipeline RAW.
455| //
456| // Loop 1's wrap-around barrier (bars[0]) is LOCAL because of the in-loop
457| // RAW between stage1 (write) and the next iteration's stage0 (read).
458| // That barrier physically sits at the bottom of loop 1's body and is the
459| // most recent LDS sync after the loop exits, so it already covers the
460| // (a1, b0) cross-pipeline dep at the boundary.  The boundary barriers
461| // can therefore be eliminated.
462| //
463| // Expected:
464| //   ttg.barrier local          (pre-barrier for loop 1)
465| //   amdg.cond_barrier          (#1 phase shift for loop 1)
466| //   scf.for { loop 1 }
467| //   NO amdg.cond_barrier       (#2 eliminated — wrap-around covers)
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 468-472
```mlir
468| //   NO ttg.barrier local       (prelude eliminated)
469| //   NO amdg.cond_barrier       (#3 eliminated)
470| //   scf.for { loop 2 }
471| //   amdg.cond_barrier          (#4 post-loop reconverge for loop 2)
472| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 473-476
```mlir
473| #b2b_blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 8], warpsPerCTA = [8, 1], order = [1, 0]}>
474| #b2b_mma = #ttg.amd_mfma<{version = 3, warpsPerCTA = [2, 4], instrShape = [16, 16, 16], isTransposed = true}>
475| #b2b_shared = #ttg.swizzled_shared<{vec = 4, perPhase = 1, maxPhase = 16, order = [1, 0]}>
476| #b2b_smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 477-477
```mlir
477| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 478-495
```mlir
478|   tt.func @back_to_back_wrap_around_covers_dep(
479|       %lb: i32, %ub: i32, %step: i32,
480|       %acc: tensor<256x256xf32, #b2b_mma>,
481|       %ptr: tensor<256x64x!tt.ptr<f16>, #b2b_blocked>) {
482| 
483|     %smem = ttg.local_alloc : () -> !ttg.memdesc<256x64xf16, #b2b_shared, #b2b_smem, mutable>
484| 
485|     // Loop 1: stage0 reads LDS, stage1 writes LDS
486|     %r1:2 = scf.for %i = %lb to %ub step %step
487|         iter_args(%a1 = %acc, %s1 = %smem)
488|         -> (tensor<256x256xf32, #b2b_mma>, !ttg.memdesc<256x64xf16, #b2b_shared, #b2b_smem, mutable>) : i32 {
489|       %ld1 = scf.execute_region -> tensor<256x16xf16, #ttg.dot_op<{opIdx = 0, parent = #b2b_mma, kWidth = 4}>> no_inline {
490|         %sub = ttg.memdesc_subslice %s1[0, 0] : !ttg.memdesc<256x64xf16, #b2b_shared, #b2b_smem, mutable> -> !ttg.memdesc<256x16xf16, #b2b_shared, #b2b_smem, mutable, 256x64>
491|         %v = ttg.local_load %sub : !ttg.memdesc<256x16xf16, #b2b_shared, #b2b_smem, mutable, 256x64> -> tensor<256x16xf16, #ttg.dot_op<{opIdx = 0, parent = #b2b_mma, kWidth = 4}>>
492|         scf.yield %v : tensor<256x16xf16, #ttg.dot_op<{opIdx = 0, parent = #b2b_mma, kWidth = 4}>>
493|       } {triton.warp_pipeline.stage = "lds_load"}
494| 
495|       %st1 = scf.execute_region -> !ttg.memdesc<256x64xf16, #b2b_shared, #b2b_smem, mutable> no_inline {
```
**EN:** This function-oriented block defines or enters `back_to_back_wrap_around_covers_dep`. Within it, the test exercises tt.func, scf.execute_region, shared/local memory allocation, structured loops, ttg.memdesc_subslice, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `back_to_back_wrap_around_covers_dep` 为核心。测试在其中演示 tt.func、scf.execute_region、共享/本地内存分配、结构化循环、ttg.memdesc_subslice，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 496-513
```mlir
496|         %data = tt.load %ptr : tensor<256x64x!tt.ptr<f16>, #b2b_blocked>
497|         ttg.local_store %data, %s1 : tensor<256x64xf16, #b2b_blocked> -> !ttg.memdesc<256x64xf16, #b2b_shared, #b2b_smem, mutable>
498|         scf.yield %s1 : !ttg.memdesc<256x64xf16, #b2b_shared, #b2b_smem, mutable>
499|       } {triton.warp_pipeline.stage = "global_load_and_store"}
500| 
501|       scf.yield %a1, %st1 : tensor<256x256xf32, #b2b_mma>, !ttg.memdesc<256x64xf16, #b2b_shared, #b2b_smem, mutable>
502|     } {triton.warp_pipeline.pipelined_for}
503| 
504|     // Loop 2: same structure — reads+writes the SAME buffer → cross-pipeline RAW
505|     %r2:2 = scf.for %j = %lb to %ub step %step
506|         iter_args(%a2 = %r1#0, %s2 = %r1#1)
507|         -> (tensor<256x256xf32, #b2b_mma>, !ttg.memdesc<256x64xf16, #b2b_shared, #b2b_smem, mutable>) : i32 {
508|       %ld2 = scf.execute_region -> tensor<256x16xf16, #ttg.dot_op<{opIdx = 0, parent = #b2b_mma, kWidth = 4}>> no_inline {
509|         %sub2 = ttg.memdesc_subslice %s2[0, 0] : !ttg.memdesc<256x64xf16, #b2b_shared, #b2b_smem, mutable> -> !ttg.memdesc<256x16xf16, #b2b_shared, #b2b_smem, mutable, 256x64>
510|         %v2 = ttg.local_load %sub2 : !ttg.memdesc<256x16xf16, #b2b_shared, #b2b_smem, mutable, 256x64> -> tensor<256x16xf16, #ttg.dot_op<{opIdx = 0, parent = #b2b_mma, kWidth = 4}>>
511|         scf.yield %v2 : tensor<256x16xf16, #ttg.dot_op<{opIdx = 0, parent = #b2b_mma, kWidth = 4}>>
512|       } {triton.warp_pipeline.stage = "epilogue_lds_load"}
513| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, masked or vectorized loads, local/shared memory stores, structured loops, scf.execute_region.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、带掩码或向量化的加载、本地/共享内存存储、结构化循环、scf.execute_region。

### Lines 514-527
```mlir
514|       %st2 = scf.execute_region -> !ttg.memdesc<256x64xf16, #b2b_shared, #b2b_smem, mutable> no_inline {
515|         %data2 = tt.load %ptr : tensor<256x64x!tt.ptr<f16>, #b2b_blocked>
516|         ttg.local_store %data2, %s2 : tensor<256x64xf16, #b2b_blocked> -> !ttg.memdesc<256x64xf16, #b2b_shared, #b2b_smem, mutable>
517|         scf.yield %s2 : !ttg.memdesc<256x64xf16, #b2b_shared, #b2b_smem, mutable>
518|       } {triton.warp_pipeline.stage = "epilogue_global_load_and_store"}
519| 
520|       scf.yield %a2, %st2 : tensor<256x256xf32, #b2b_mma>, !ttg.memdesc<256x64xf16, #b2b_shared, #b2b_smem, mutable>
521|     } {triton.warp_pipeline.pipelined_for}
522| 
523|     ttg.local_dealloc %smem : !ttg.memdesc<256x64xf16, #b2b_shared, #b2b_smem, mutable>
524|     tt.return
525|   }
526| }
527| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, scf.execute_region, masked or vectorized loads, local/shared memory stores, ttg.local_dealloc.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、scf.execute_region、带掩码或向量化的加载、本地/共享内存存储、ttg.local_dealloc。

### Lines 528-529
```mlir
528| // CHECK-LABEL: tt.func @back_to_back_wrap_around_covers_dep
529| // Pre-barrier and phase shift for loop 1.
```
**EN:** This function-oriented block defines or enters `back_to_back_wrap_around_covers_dep`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `back_to_back_wrap_around_covers_dep` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 530-533
```mlir
530| // CHECK: ttg.barrier local
531| // CHECK: amdg.cond_barrier
532| // CHECK: scf.for
533| // Wrap-around barrier inside loop 1 (LOCAL — covers cross-pipeline dep).
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 534-537
```mlir
534| // CHECK: ttg.barrier local
535| // CHECK: scf.yield
536| // Boundary barriers are eliminated: A's wrap-around already provides the
537| // LDS sync needed for loop 2's first read; phase carries over.
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 538-541
```mlir
538| // CHECK-NOT: amdg.cond_barrier
539| // CHECK-NOT: ttg.barrier local
540| // CHECK: scf.for
541| // Post-loop reconverge for loop 2.
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 542-544
```mlir
542| // CHECK: amdg.cond_barrier
543| // CHECK: tt.return
544| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 545-545
```mlir
545| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 546-553
```mlir
546| 
547| // ---- Flat (unrolled) pipeline: execute_regions outside scf.for ----
548| //
549| // Simulates the output of WarpPipeliner::createFlatPipeline —
550| // 4 execute_regions from a 2-iteration × 2-stage unrolled epilogue.
551| // ConvertWarpPipeline should insert pre-barrier, phase shift,
552| // cluster barriers, priority, and reconverge around them.
553| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 554-554
```mlir
554| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 555-572
```mlir
555|   tt.func @flat_pipeline_backend(%ptr0: !tt.ptr<f32>, %ptr1: !tt.ptr<f32>) {
556|     %v0 = arith.constant 0.0 : f32
557|     %v1 = arith.constant 1.0 : f32
558|     %v2 = arith.constant 2.0 : f32
559|     %v3 = arith.constant 3.0 : f32
560| 
561|     // Iteration 0, stage 0
562|     scf.execute_region no_inline {
563|       tt.store %ptr0, %v0 : !tt.ptr<f32>
564|       scf.yield
565|     } {triton.warp_pipeline.stage = "stage0_epi", triton.warp_pipeline.priority = 1 : i32}
566| 
567|     // Iteration 0, stage 1
568|     scf.execute_region no_inline {
569|       tt.store %ptr1, %v1 : !tt.ptr<f32>
570|       scf.yield
571|     } {triton.warp_pipeline.stage = "stage1_epi", triton.warp_pipeline.priority = 0 : i32}
572| 
```
**EN:** This function-oriented block defines or enters `flat_pipeline_backend`. Within it, the test exercises constants, tt.func, scf.execute_region, masked or vectorized stores, loop/if yielded values, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `flat_pipeline_backend` 为核心。测试在其中演示 常量、tt.func、scf.execute_region、带掩码或向量化的存储、循环/分支产出值，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 573-588
```mlir
573|     // Iteration 1, stage 0
574|     scf.execute_region no_inline {
575|       tt.store %ptr0, %v2 : !tt.ptr<f32>
576|       scf.yield
577|     } {triton.warp_pipeline.stage = "stage0_epi", triton.warp_pipeline.priority = 1 : i32}
578| 
579|     // Iteration 1, stage 1
580|     scf.execute_region no_inline {
581|       tt.store %ptr1, %v3 : !tt.ptr<f32>
582|       scf.yield
583|     } {triton.warp_pipeline.stage = "stage1_epi", triton.warp_pipeline.priority = 0 : i32}
584| 
585|     tt.return
586|   }
587| }
588| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining scf.execute_region, masked or vectorized stores, loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 scf.execute_region、带掩码或向量化的存储、循环/分支产出值、tt.return。

### Lines 589-590
```mlir
589| // CHECK-LABEL: tt.func @flat_pipeline_backend
590| // All execute_regions must be inlined.
```
**EN:** This function-oriented block defines or enters `flat_pipeline_backend`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `flat_pipeline_backend` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 591-593
```mlir
591| // CHECK-NOT: no_inline
592| //
593| // Pre-barrier + phase shift.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 594-599
```mlir
594| // CHECK: ttg.barrier local
595| // CHECK: %[[WARPLOW:.+]] = arith.cmpi eq
596| // CHECK: %[[WARPHIGH:.+]] = arith.cmpi ne
597| // CHECK: amdg.cond_barrier %[[WARPHIGH]]
598| //
599| // Stage 0 priority.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 600-601
```mlir
600| // CHECK: rocdl.s.setprio 1
601| // Stage 0 ops (inlined).
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 602-604
```mlir
602| // CHECK: tt.store
603| //
604| // Cluster barrier between stages 0 and 1.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 605-609
```mlir
605| // CHECK: rocdl.s.setprio 0
606| // CHECK: rocdl.sched.barrier
607| // CHECK: rocdl.s.barrier
608| // CHECK: rocdl.sched.barrier
609| // Stage 1 ops.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 610-612
```mlir
610| // CHECK: tt.store
611| //
612| // Cluster barrier between iteration 0 stage 1 and iteration 1 stage 0.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 613-619
```mlir
613| // CHECK: rocdl.s.setprio 1
614| // CHECK: rocdl.sched.barrier
615| // CHECK: rocdl.s.barrier
616| // CHECK: rocdl.sched.barrier
617| // CHECK: tt.store
618| //
619| // Cluster barrier between iteration 1 stages.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 620-626
```mlir
620| // CHECK: rocdl.s.setprio 0
621| // CHECK: rocdl.sched.barrier
622| // CHECK: rocdl.s.barrier
623| // CHECK: rocdl.sched.barrier
624| // CHECK: tt.store
625| //
626| // Post-sequence priority reset + reconverge.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 627-630
```mlir
627| // CHECK: rocdl.s.setprio 0
628| // CHECK: amdg.cond_barrier %[[WARPLOW]]
629| // CHECK: tt.return
630| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 631-631
```mlir
631| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 632-649
```mlir
632| 
633| // ---- Back-to-back: pipelined scf.for + flat (unrolled) pipeline ----
634| //
635| // Loop 1 (scf.for) followed immediately by a flat pipeline with no
636| // intervening operations.  The post-loop reconverge, prelude barrier,
637| // and phase shift are all eliminated — same logic as back-to-back
638| // scf.for loops.
639| //
640| // Expected:
641| //   ttg.barrier local          (pre-barrier for loop 1)
642| //   amdg.cond_barrier          (#1 phase shift for loop 1)
643| //   scf.for { loop 1 }
644| //   NO amdg.cond_barrier       (#2 eliminated)
645| //   NO ttg.barrier local       (pre-barrier eliminated)
646| //   NO amdg.cond_barrier       (#3 eliminated)
647| //   [flat pipeline stages]
648| //   amdg.cond_barrier          (#4 reconverge for flat pipeline)
649| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 650-653
```mlir
650| #b2bf_blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 8], warpsPerCTA = [8, 1], order = [1, 0]}>
651| #b2bf_mma = #ttg.amd_mfma<{version = 3, warpsPerCTA = [2, 4], instrShape = [16, 16, 16], isTransposed = true}>
652| #b2bf_shared = #ttg.swizzled_shared<{vec = 4, perPhase = 1, maxPhase = 16, order = [1, 0]}>
653| #b2bf_smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 654-654
```mlir
654| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 655-672
```mlir
655|   tt.func @back_to_back_for_then_flat(
656|       %lb: i32, %ub: i32, %step: i32,
657|       %acc: tensor<256x256xf32, #b2bf_mma>,
658|       %ptr: tensor<256x64x!tt.ptr<f16>, #b2bf_blocked>,
659|       %sptr: !tt.ptr<f32>) {
660| 
661|     %smem = ttg.local_alloc : () -> !ttg.memdesc<256x64xf16, #b2bf_shared, #b2bf_smem, mutable>
662|     %v0 = arith.constant 0.0 : f32
663|     %v1 = arith.constant 1.0 : f32
664| 
665|     // Loop 1: local_load + local_store → wrap-around is ttg.barrier local
666|     %r1:2 = scf.for %i = %lb to %ub step %step
667|         iter_args(%a1 = %acc, %s1 = %smem)
668|         -> (tensor<256x256xf32, #b2bf_mma>, !ttg.memdesc<256x64xf16, #b2bf_shared, #b2bf_smem, mutable>) : i32 {
669|       %ld1 = scf.execute_region -> tensor<256x16xf16, #ttg.dot_op<{opIdx = 0, parent = #b2bf_mma, kWidth = 4}>> no_inline {
670|         %sub = ttg.memdesc_subslice %s1[0, 0] : !ttg.memdesc<256x64xf16, #b2bf_shared, #b2bf_smem, mutable> -> !ttg.memdesc<256x16xf16, #b2bf_shared, #b2bf_smem, mutable, 256x64>
671|         %v = ttg.local_load %sub : !ttg.memdesc<256x16xf16, #b2bf_shared, #b2bf_smem, mutable, 256x64> -> tensor<256x16xf16, #ttg.dot_op<{opIdx = 0, parent = #b2bf_mma, kWidth = 4}>>
672|         scf.yield %v : tensor<256x16xf16, #ttg.dot_op<{opIdx = 0, parent = #b2bf_mma, kWidth = 4}>>
```
**EN:** This function-oriented block defines or enters `back_to_back_for_then_flat`. Within it, the test exercises tt.func, constants, shared/local memory allocation, structured loops, scf.execute_region, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `back_to_back_for_then_flat` 为核心。测试在其中演示 tt.func、常量、共享/本地内存分配、结构化循环、scf.execute_region，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 673-690
```mlir
673|       } {triton.warp_pipeline.stage = "lds_load"}
674| 
675|       %st1 = scf.execute_region -> !ttg.memdesc<256x64xf16, #b2bf_shared, #b2bf_smem, mutable> no_inline {
676|         %data = tt.load %ptr : tensor<256x64x!tt.ptr<f16>, #b2bf_blocked>
677|         ttg.local_store %data, %s1 : tensor<256x64xf16, #b2bf_blocked> -> !ttg.memdesc<256x64xf16, #b2bf_shared, #b2bf_smem, mutable>
678|         scf.yield %s1 : !ttg.memdesc<256x64xf16, #b2bf_shared, #b2bf_smem, mutable>
679|       } {triton.warp_pipeline.stage = "global_load_and_store"}
680| 
681|       scf.yield %a1, %st1 : tensor<256x256xf32, #b2bf_mma>, !ttg.memdesc<256x64xf16, #b2bf_shared, #b2bf_smem, mutable>
682|     } {triton.warp_pipeline.pipelined_for}
683| 
684|     // Flat (unrolled) pipeline: 2 stages, simple stores (no LDS dep)
685|     scf.execute_region no_inline {
686|       tt.store %sptr, %v0 : !tt.ptr<f32>
687|       scf.yield
688|     } {triton.warp_pipeline.stage = "flat_stage0"}
689| 
690|     scf.execute_region no_inline {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining scf.execute_region, loop/if yielded values, masked or vectorized loads, local/shared memory stores, masked or vectorized stores.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 scf.execute_region、循环/分支产出值、带掩码或向量化的加载、本地/共享内存存储、带掩码或向量化的存储。

### Lines 691-699
```mlir
691|       tt.store %sptr, %v1 : !tt.ptr<f32>
692|       scf.yield
693|     } {triton.warp_pipeline.stage = "flat_stage1"}
694| 
695|     ttg.local_dealloc %smem : !ttg.memdesc<256x64xf16, #b2bf_shared, #b2bf_smem, mutable>
696|     tt.return
697|   }
698| }
699| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining masked or vectorized stores, loop/if yielded values, ttg.local_dealloc, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 带掩码或向量化的存储、循环/分支产出值、ttg.local_dealloc、tt.return。

### Lines 700-701
```mlir
700| // CHECK-LABEL: tt.func @back_to_back_for_then_flat
701| // Pre-barrier and phase shift for loop 1 are kept.
```
**EN:** This function-oriented block defines or enters `back_to_back_for_then_flat`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `back_to_back_for_then_flat` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 702-705
```mlir
702| // CHECK: ttg.barrier local
703| // CHECK: amdg.cond_barrier
704| // CHECK: scf.for
705| // Wrap-around barrier inside loop 1.
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 706-709
```mlir
706| // CHECK: ttg.barrier local
707| // CHECK: scf.yield
708| // Between loop 1 and flat pipeline: no cond_barriers, no ttg.barrier local
709| // (no intervening ops → phase carries over, prelude barrier redundant).
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 710-712
```mlir
710| // CHECK-NOT: amdg.cond_barrier
711| // CHECK-NOT: ttg.barrier local
712| // Flat pipeline stages (inlined after conversion).
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 713-716
```mlir
713| // CHECK: tt.store
714| // CHECK: rocdl.s.barrier
715| // CHECK: tt.store
716| // Reconverge for flat pipeline is kept.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 717-719
```mlir
717| // CHECK: amdg.cond_barrier
718| // CHECK: tt.return
719| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 720-720
```mlir
720| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 721-734
```mlir
721| 
722| // ---- Flat pipeline with pre-existing barrier between stages ----
723| //
724| // When an async_wait (or similar barrier op) already exists between
725| // flat pipeline stages, the pass should wrap it with sched_barriers
726| // instead of inserting a redundant s_barrier.
727| //
728| // Stage layout: stage0 -- async_wait -- stage1 -- (nothing) -- stage2
729| //
730| // Expected between stage0 and stage1:
731| //   sched_barrier + async_wait + sched_barrier   (wrapped, no s_barrier)
732| // Expected between stage1 and stage2:
733| //   sched_barrier + s_barrier + sched_barrier     (inserted, no async_wait)
734| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 735-735
```mlir
735| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 736-753
```mlir
736|   tt.func @flat_pipeline_existing_barrier(%ptr: !tt.ptr<f32>) {
737|     %v0 = arith.constant 0.0 : f32
738|     %v1 = arith.constant 1.0 : f32
739|     %v2 = arith.constant 2.0 : f32
740| 
741|     scf.execute_region no_inline {
742|       tt.store %ptr, %v0 : !tt.ptr<f32>
743|       scf.yield
744|     } {triton.warp_pipeline.stage = "stage0"}
745| 
746|     amdg.async_wait {num_inst = 0 : i32}
747| 
748|     scf.execute_region no_inline {
749|       tt.store %ptr, %v1 : !tt.ptr<f32>
750|       scf.yield
751|     } {triton.warp_pipeline.stage = "stage1"}
752| 
753|     scf.execute_region no_inline {
```
**EN:** This function-oriented block defines or enters `flat_pipeline_existing_barrier`. Within it, the test exercises constants, scf.execute_region, tt.func, masked or vectorized stores, loop/if yielded values, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `flat_pipeline_existing_barrier` 为核心。测试在其中演示 常量、scf.execute_region、tt.func、带掩码或向量化的存储、循环/分支产出值，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 754-761
```mlir
754|       tt.store %ptr, %v2 : !tt.ptr<f32>
755|       scf.yield
756|     } {triton.warp_pipeline.stage = "stage2"}
757| 
758|     tt.return
759|   }
760| }
761| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining masked or vectorized stores, loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 带掩码或向量化的存储、循环/分支产出值、tt.return。

### Lines 762-765
```mlir
762| // CHECK-LABEL: tt.func @flat_pipeline_existing_barrier
763| // CHECK-NOT: no_inline
764| //
765| // Pre-barrier + phase shift.
```
**EN:** This function-oriented block defines or enters `flat_pipeline_existing_barrier`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `flat_pipeline_existing_barrier` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 766-769
```mlir
766| // CHECK: ttg.barrier local
767| // CHECK: amdg.cond_barrier
768| //
769| // Stage 0 ops.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 770-772
```mlir
770| // CHECK: tt.store
771| //
772| // Between stage 0 and 1: existing async_wait wrapped, no s_barrier.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 773-777
```mlir
773| // CHECK: rocdl.sched.barrier
774| // CHECK-NEXT: amdg.async_wait
775| // CHECK-NEXT: rocdl.sched.barrier
776| // CHECK-NOT: rocdl.s.barrier
777| // Stage 1 ops.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 778-780
```mlir
778| // CHECK: tt.store
779| //
780| // Between stage 1 and 2: no pre-existing barrier, so s_barrier inserted.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 781-784
```mlir
781| // CHECK: rocdl.sched.barrier
782| // CHECK-NEXT: rocdl.s.barrier
783| // CHECK-NEXT: rocdl.sched.barrier
784| // Stage 2 ops.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 785-787
```mlir
785| // CHECK: tt.store
786| //
787| // Reconverge.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 788-790
```mlir
788| // CHECK: amdg.cond_barrier
789| // CHECK: tt.return
790| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 791-791
```mlir
791| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 792-808
```mlir
792| 
793| // ---- Back-to-back: no cross-pipeline LDS dep → barriers eliminated ----
794| //
795| // Loop 1 reads+writes shared memory.  Loop 2 only does global ops (no LDS).
796| // No cross-pipeline LDS dependency exists, so the boundary barriers are
797| // safely eliminated and the phase carries over.
798| //
799| // Expected:
800| //   ttg.barrier local          (pre-barrier for loop 1)
801| //   amdg.cond_barrier          (#1 phase shift for loop 1)
802| //   scf.for { loop 1 }
803| //   NO amdg.cond_barrier       (eliminated)
804| //   NO ttg.barrier local       (eliminated)
805| //   NO amdg.cond_barrier       (eliminated)
806| //   scf.for { loop 2 }
807| //   amdg.cond_barrier          (#4 reconverge for loop 2)
808| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 809-812
```mlir
809| #b2bnd_blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 8], warpsPerCTA = [8, 1], order = [1, 0]}>
810| #b2bnd_mma = #ttg.amd_mfma<{version = 3, warpsPerCTA = [2, 4], instrShape = [16, 16, 16], isTransposed = true}>
811| #b2bnd_shared = #ttg.swizzled_shared<{vec = 4, perPhase = 1, maxPhase = 16, order = [1, 0]}>
812| #b2bnd_smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 813-813
```mlir
813| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 814-831
```mlir
814|   tt.func @back_to_back_no_dep_elimination(
815|       %lb: i32, %ub: i32, %step: i32,
816|       %acc: tensor<256x256xf32, #b2bnd_mma>,
817|       %ptr: tensor<256x64x!tt.ptr<f16>, #b2bnd_blocked>,
818|       %gptr: !tt.ptr<f32>) {
819| 
820|     %smem = ttg.local_alloc : () -> !ttg.memdesc<256x64xf16, #b2bnd_shared, #b2bnd_smem, mutable>
821|     %v0 = arith.constant 0.0 : f32
822|     %v1 = arith.constant 1.0 : f32
823| 
824|     // Loop 1: stage0 reads LDS, stage1 writes LDS
825|     %r1:2 = scf.for %i = %lb to %ub step %step
826|         iter_args(%a1 = %acc, %s1 = %smem)
827|         -> (tensor<256x256xf32, #b2bnd_mma>, !ttg.memdesc<256x64xf16, #b2bnd_shared, #b2bnd_smem, mutable>) : i32 {
828|       %ld1 = scf.execute_region -> tensor<256x16xf16, #ttg.dot_op<{opIdx = 0, parent = #b2bnd_mma, kWidth = 4}>> no_inline {
829|         %sub = ttg.memdesc_subslice %s1[0, 0] : !ttg.memdesc<256x64xf16, #b2bnd_shared, #b2bnd_smem, mutable> -> !ttg.memdesc<256x16xf16, #b2bnd_shared, #b2bnd_smem, mutable, 256x64>
830|         %v = ttg.local_load %sub : !ttg.memdesc<256x16xf16, #b2bnd_shared, #b2bnd_smem, mutable, 256x64> -> tensor<256x16xf16, #ttg.dot_op<{opIdx = 0, parent = #b2bnd_mma, kWidth = 4}>>
831|         scf.yield %v : tensor<256x16xf16, #ttg.dot_op<{opIdx = 0, parent = #b2bnd_mma, kWidth = 4}>>
```
**EN:** This function-oriented block defines or enters `back_to_back_no_dep_elimination`. Within it, the test exercises tt.func, constants, shared/local memory allocation, structured loops, scf.execute_region, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `back_to_back_no_dep_elimination` 为核心。测试在其中演示 tt.func、常量、共享/本地内存分配、结构化循环、scf.execute_region，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 832-849
```mlir
832|       } {triton.warp_pipeline.stage = "lds_load"}
833| 
834|       %st1 = scf.execute_region -> !ttg.memdesc<256x64xf16, #b2bnd_shared, #b2bnd_smem, mutable> no_inline {
835|         %data = tt.load %ptr : tensor<256x64x!tt.ptr<f16>, #b2bnd_blocked>
836|         ttg.local_store %data, %s1 : tensor<256x64xf16, #b2bnd_blocked> -> !ttg.memdesc<256x64xf16, #b2bnd_shared, #b2bnd_smem, mutable>
837|         scf.yield %s1 : !ttg.memdesc<256x64xf16, #b2bnd_shared, #b2bnd_smem, mutable>
838|       } {triton.warp_pipeline.stage = "global_load_and_store"}
839| 
840|       scf.yield %a1, %st1 : tensor<256x256xf32, #b2bnd_mma>, !ttg.memdesc<256x64xf16, #b2bnd_shared, #b2bnd_smem, mutable>
841|     } {triton.warp_pipeline.pipelined_for}
842| 
843|     // Loop 2: global-only ops — no LDS access at all
844|     scf.for %j = %lb to %ub step %step : i32 {
845|       scf.execute_region no_inline {
846|         tt.store %gptr, %v0 : !tt.ptr<f32>
847|         scf.yield
848|       } {triton.warp_pipeline.stage = "global_store_0"}
849| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, scf.execute_region, masked or vectorized loads, local/shared memory stores, structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、scf.execute_region、带掩码或向量化的加载、本地/共享内存存储、结构化循环。

### Lines 850-862
```mlir
850|       scf.execute_region no_inline {
851|         tt.store %gptr, %v1 : !tt.ptr<f32>
852|         scf.yield
853|       } {triton.warp_pipeline.stage = "global_store_1"}
854| 
855|       scf.yield
856|     } {triton.warp_pipeline.pipelined_for}
857| 
858|     ttg.local_dealloc %smem : !ttg.memdesc<256x64xf16, #b2bnd_shared, #b2bnd_smem, mutable>
859|     tt.return
860|   }
861| }
862| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, scf.execute_region, masked or vectorized stores, ttg.local_dealloc, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、scf.execute_region、带掩码或向量化的存储、ttg.local_dealloc、tt.return。

### Lines 863-864
```mlir
863| // CHECK-LABEL: tt.func @back_to_back_no_dep_elimination
864| // Pre-barrier and phase shift for loop 1.
```
**EN:** This function-oriented block defines or enters `back_to_back_no_dep_elimination`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `back_to_back_no_dep_elimination` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 865-868
```mlir
865| // CHECK: ttg.barrier local
866| // CHECK: amdg.cond_barrier
867| // CHECK: scf.for
868| // Wrap-around barrier inside loop 1.
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 869-871
```mlir
869| // CHECK: ttg.barrier local
870| // CHECK: scf.yield
871| // No cross-pipeline LDS dep → barriers eliminated, phase carries over.
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 872-875
```mlir
872| // CHECK-NOT: amdg.cond_barrier
873| // CHECK-NOT: ttg.barrier local
874| // CHECK: scf.for
875| // Post-loop reconverge for loop 2.
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 876-878
```mlir
876| // CHECK: amdg.cond_barrier
877| // CHECK: tt.return
878| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 879-879
```mlir
879| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 880-897
```mlir
880| 
881| // ---- Back-to-back: cross-pipeline dep covered by loop A's barrier ----
882| //
883| // Loop 1 has 3 stages: stage0 writes LDS, stage1 reads LDS, stage2 is
884| // compute-only.  The circular dependency analysis places a LOCAL barrier
885| // between stage1 and stage2 (covering the WAR from stage1 reading what
886| // stage0 wrote).
887| //
888| // Loop 2 has 2 stages: stage0 reads the SAME LDS buffer, stage1 is
889| // compute-only.  There IS a cross-pipeline dependency (loop1.stage0 writes
890| // smem that loop2.stage0 reads), but it is already covered by loop 1's
891| // barrier between stage1 and stage2.
892| //
893| // At the boundary with no barrier: warp0 runs b0, warp1 runs a2.
894| // Since a2 has no LDS access and the LOCAL barrier before a2 already
895| // flushed all prior LDS writes, b0's read is safe.
896| //
897| // Expected:
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 898-906
```mlir
898| //   ttg.barrier local          (pre-barrier for loop 1)
899| //   amdg.cond_barrier          (phase shift for loop 1)
900| //   scf.for { loop 1 — 3 stages }
901| //   NO amdg.cond_barrier       (eliminated)
902| //   NO ttg.barrier local       (eliminated)
903| //   NO amdg.cond_barrier       (eliminated)
904| //   scf.for { loop 2 — 2 stages }
905| //   amdg.cond_barrier          (reconverge for loop 2)
906| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 907-910
```mlir
907| #b2bcov_blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 8], warpsPerCTA = [8, 1], order = [1, 0]}>
908| #b2bcov_mma = #ttg.amd_mfma<{version = 3, warpsPerCTA = [2, 4], instrShape = [16, 16, 16], isTransposed = true}>
909| #b2bcov_shared = #ttg.swizzled_shared<{vec = 4, perPhase = 1, maxPhase = 16, order = [1, 0]}>
910| #b2bcov_smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 911-911
```mlir
911| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 912-929
```mlir
912|   tt.func @back_to_back_dep_covered_elimination(
913|       %lb: i32, %ub: i32, %step: i32,
914|       %acc: tensor<256x256xf32, #b2bcov_mma>,
915|       %ptr: tensor<256x64x!tt.ptr<f16>, #b2bcov_blocked>,
916|       %gptr: !tt.ptr<f32>) {
917| 
918|     %smem = ttg.local_alloc : () -> !ttg.memdesc<256x64xf16, #b2bcov_shared, #b2bcov_smem, mutable>
919|     %v0 = arith.constant 0.0 : f32
920| 
921|     // Loop 1: 3 stages
922|     //   stage0: writes LDS (local_store)
923|     //   stage1: reads LDS  (local_load) → RAW with stage0
924|     //   stage2: compute-only (global store, no LDS)
925|     // Circular analysis: barrier between stage1 and stage2 is LOCAL.
926|     %r1:2 = scf.for %i = %lb to %ub step %step
927|         iter_args(%a1 = %acc, %s1 = %smem)
928|         -> (tensor<256x256xf32, #b2bcov_mma>, !ttg.memdesc<256x64xf16, #b2bcov_shared, #b2bcov_smem, mutable>) : i32 {
929|       %st1 = scf.execute_region -> !ttg.memdesc<256x64xf16, #b2bcov_shared, #b2bcov_smem, mutable> no_inline {
```
**EN:** This function-oriented block defines or enters `back_to_back_dep_covered_elimination`. Within it, the test exercises tt.func, shared/local memory allocation, constants, structured loops, scf.execute_region, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `back_to_back_dep_covered_elimination` 为核心。测试在其中演示 tt.func、共享/本地内存分配、常量、结构化循环、scf.execute_region，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 930-947
```mlir
930|         %data = tt.load %ptr : tensor<256x64x!tt.ptr<f16>, #b2bcov_blocked>
931|         ttg.local_store %data, %s1 : tensor<256x64xf16, #b2bcov_blocked> -> !ttg.memdesc<256x64xf16, #b2bcov_shared, #b2bcov_smem, mutable>
932|         scf.yield %s1 : !ttg.memdesc<256x64xf16, #b2bcov_shared, #b2bcov_smem, mutable>
933|       } {triton.warp_pipeline.stage = "global_load_and_store"}
934| 
935|       %ld1 = scf.execute_region -> tensor<256x16xf16, #ttg.dot_op<{opIdx = 0, parent = #b2bcov_mma, kWidth = 4}>> no_inline {
936|         %sub = ttg.memdesc_subslice %s1[0, 0] : !ttg.memdesc<256x64xf16, #b2bcov_shared, #b2bcov_smem, mutable> -> !ttg.memdesc<256x16xf16, #b2bcov_shared, #b2bcov_smem, mutable, 256x64>
937|         %v = ttg.local_load %sub : !ttg.memdesc<256x16xf16, #b2bcov_shared, #b2bcov_smem, mutable, 256x64> -> tensor<256x16xf16, #ttg.dot_op<{opIdx = 0, parent = #b2bcov_mma, kWidth = 4}>>
938|         scf.yield %v : tensor<256x16xf16, #ttg.dot_op<{opIdx = 0, parent = #b2bcov_mma, kWidth = 4}>>
939|       } {triton.warp_pipeline.stage = "lds_load"}
940| 
941|       scf.execute_region no_inline {
942|         tt.store %gptr, %v0 : !tt.ptr<f32>
943|         scf.yield
944|       } {triton.warp_pipeline.stage = "compute"}
945| 
946|       scf.yield %a1, %s1 : tensor<256x256xf32, #b2bcov_mma>, !ttg.memdesc<256x64xf16, #b2bcov_shared, #b2bcov_smem, mutable>
947|     } {triton.warp_pipeline.pipelined_for}
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, scf.execute_region, masked or vectorized loads, local/shared memory stores, ttg.memdesc_subslice.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、scf.execute_region、带掩码或向量化的加载、本地/共享内存存储、ttg.memdesc_subslice。

### Lines 948-965
```mlir
948| 
949|     // Loop 2: stage0 reads the SAME LDS buffer, stage1 is compute-only.
950|     // Cross-pipeline dep (a0 writes → b0 reads) is covered by loop 1's
951|     // barrier between stage1 and stage2.
952|     %r2:2 = scf.for %j = %lb to %ub step %step
953|         iter_args(%a2 = %r1#0, %s2 = %r1#1)
954|         -> (tensor<256x256xf32, #b2bcov_mma>, !ttg.memdesc<256x64xf16, #b2bcov_shared, #b2bcov_smem, mutable>) : i32 {
955|       %ld2 = scf.execute_region -> tensor<256x16xf16, #ttg.dot_op<{opIdx = 0, parent = #b2bcov_mma, kWidth = 4}>> no_inline {
956|         %sub2 = ttg.memdesc_subslice %s2[0, 0] : !ttg.memdesc<256x64xf16, #b2bcov_shared, #b2bcov_smem, mutable> -> !ttg.memdesc<256x16xf16, #b2bcov_shared, #b2bcov_smem, mutable, 256x64>
957|         %v2 = ttg.local_load %sub2 : !ttg.memdesc<256x16xf16, #b2bcov_shared, #b2bcov_smem, mutable, 256x64> -> tensor<256x16xf16, #ttg.dot_op<{opIdx = 0, parent = #b2bcov_mma, kWidth = 4}>>
958|         scf.yield %v2 : tensor<256x16xf16, #ttg.dot_op<{opIdx = 0, parent = #b2bcov_mma, kWidth = 4}>>
959|       } {triton.warp_pipeline.stage = "epilogue_lds_load"}
960| 
961|       scf.execute_region no_inline {
962|         tt.store %gptr, %v0 : !tt.ptr<f32>
963|         scf.yield
964|       } {triton.warp_pipeline.stage = "epilogue_compute"}
965| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining scf.execute_region, loop/if yielded values, structured loops, ttg.memdesc_subslice, local/shared memory loads.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 scf.execute_region、循环/分支产出值、结构化循环、ttg.memdesc_subslice、本地/共享内存加载。

### Lines 966-973
```mlir
966|       scf.yield %a2, %s2 : tensor<256x256xf32, #b2bcov_mma>, !ttg.memdesc<256x64xf16, #b2bcov_shared, #b2bcov_smem, mutable>
967|     } {triton.warp_pipeline.pipelined_for}
968| 
969|     ttg.local_dealloc %smem : !ttg.memdesc<256x64xf16, #b2bcov_shared, #b2bcov_smem, mutable>
970|     tt.return
971|   }
972| }
973| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, ttg.local_dealloc, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、ttg.local_dealloc、tt.return。

### Lines 974-975
```mlir
974| // CHECK-LABEL: tt.func @back_to_back_dep_covered_elimination
975| // Pre-barrier and phase shift for loop 1.
```
**EN:** This function-oriented block defines or enters `back_to_back_dep_covered_elimination`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `back_to_back_dep_covered_elimination` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 976-979
```mlir
976| // CHECK: ttg.barrier local
977| // CHECK: amdg.cond_barrier
978| // CHECK: scf.for
979| // Loop 1 has 3 stages; barrier between stage1→stage2 is LOCAL (covers dep).
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 980-983
```mlir
980| // CHECK: ttg.barrier local
981| // CHECK: scf.yield
982| // Cross-pipeline dep IS covered by loop 1's internal barrier →
983| // boundary barriers eliminated, phase carries over.
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 984-987
```mlir
984| // CHECK-NOT: amdg.cond_barrier
985| // CHECK-NOT: ttg.barrier local
986| // CHECK: scf.for
987| // Post-loop reconverge for loop 2.
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 988-990
```mlir
988| // CHECK: amdg.cond_barrier
989| // CHECK: tt.return
990| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 991-991
```mlir
991| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 992-1009
```mlir
 992| 
 993| // ---- Adjacent-stage LDS dependency: barrier must be LOCAL ----
 994| //
 995| // 3-stage loop pipeline where stage0 writes LDS and stage1 reads it.
 996| // Stage2 has no LDS access.
 997| //
 998| // The distance-2+ analysis only checks pairs separated by ≥2 clusters,
 999| // so it never examines (stage0, stage1) directly.  Without the adjacent-
1000| // stage check, the barrier between stage0 and stage1 would be emitted as
1001| // a plain s_barrier, and ModuleMembarAnalysis would later insert a
1002| // redundant ttg.barrier local inside the pipeline — breaking timing.
1003| //
1004| // With the adjacent-stage check:
1005| //   bars[0] (wrap-around) = false  (a2 no LDS, a0 writes — no conflict)
1006| //   bars[1] (a0→a1)       = true   (a0 writes, a1 reads — RAW)
1007| //   bars[2] (a1→a2)       = true   (a1→a0 WAR via distance-2)
1008| //
1009| // Expected inside the loop body:
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1010-1017
```mlir
1010| //   stage0 ops  (local_store)
1011| //   ttg.barrier local             (bars[1] — adjacent dep)
1012| //   stage1 ops  (local_load)
1013| //   ttg.barrier local             (bars[2] — distance-2 dep)
1014| //   stage2 ops  (global store)
1015| //   rocdl.s.barrier               (bars[0] — wrap-around, no LDS dep)
1016| //   scf.yield
1017| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 1018-1022
```mlir
1018| #adj_blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 8], warpsPerCTA = [8, 1], order = [1, 0]}>
1019| #adj_mma = #ttg.amd_mfma<{version = 3, warpsPerCTA = [2, 4], instrShape = [16, 16, 16], isTransposed = true}>
1020| #adj_dot = #ttg.dot_op<{opIdx = 0, parent = #adj_mma, kWidth = 4}>
1021| #adj_shared = #ttg.swizzled_shared<{vec = 4, perPhase = 1, maxPhase = 16, order = [1, 0]}>
1022| #adj_smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1023-1023
```mlir
1023| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 1024-1041
```mlir
1024|   tt.func @adjacent_stage_lds_dep(
1025|       %lb: i32, %ub: i32, %step: i32,
1026|       %acc: tensor<256x16xf16, #adj_dot>,
1027|       %ptr: tensor<256x64x!tt.ptr<f16>, #adj_blocked>,
1028|       %gptr: !tt.ptr<f32>) {
1029| 
1030|     %smem = ttg.local_alloc : () -> !ttg.memdesc<256x64xf16, #adj_shared, #adj_smem, mutable>
1031|     %v0 = arith.constant 0.0 : f32
1032| 
1033|     // The local_load result must be carried as an iter_arg so it is not
1034|     // DCE'd — otherwise the barrier between stage0 and stage1 would merge
1035|     // with the barrier between stage1 and stage2.
1036|     %r:3 = scf.for %i = %lb to %ub step %step
1037|         iter_args(%a = %acc, %s = %smem, %prev = %acc)
1038|         -> (tensor<256x16xf16, #adj_dot>, !ttg.memdesc<256x64xf16, #adj_shared, #adj_smem, mutable>, tensor<256x16xf16, #adj_dot>) : i32 {
1039| 
1040|       // Stage 0: writes LDS
1041|       %st = scf.execute_region -> !ttg.memdesc<256x64xf16, #adj_shared, #adj_smem, mutable> no_inline {
```
**EN:** This function-oriented block defines or enters `adjacent_stage_lds_dep`. Within it, the test exercises tt.func, shared/local memory allocation, constants, structured loops, scf.execute_region, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `adjacent_stage_lds_dep` 为核心。测试在其中演示 tt.func、共享/本地内存分配、常量、结构化循环、scf.execute_region，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1042-1059
```mlir
1042|         %data = tt.load %ptr : tensor<256x64x!tt.ptr<f16>, #adj_blocked>
1043|         ttg.local_store %data, %s : tensor<256x64xf16, #adj_blocked> -> !ttg.memdesc<256x64xf16, #adj_shared, #adj_smem, mutable>
1044|         scf.yield %s : !ttg.memdesc<256x64xf16, #adj_shared, #adj_smem, mutable>
1045|       } {triton.warp_pipeline.stage = "global_load_and_store"}
1046| 
1047|       // Stage 1: reads LDS — RAW dep with stage 0
1048|       %ld = scf.execute_region -> tensor<256x16xf16, #adj_dot> no_inline {
1049|         %sub = ttg.memdesc_subslice %s[0, 0] : !ttg.memdesc<256x64xf16, #adj_shared, #adj_smem, mutable> -> !ttg.memdesc<256x16xf16, #adj_shared, #adj_smem, mutable, 256x64>
1050|         %v = ttg.local_load %sub : !ttg.memdesc<256x16xf16, #adj_shared, #adj_smem, mutable, 256x64> -> tensor<256x16xf16, #adj_dot>
1051|         scf.yield %v : tensor<256x16xf16, #adj_dot>
1052|       } {triton.warp_pipeline.stage = "lds_load"}
1053| 
1054|       // Stage 2: compute-only — no LDS access
1055|       scf.execute_region no_inline {
1056|         tt.store %gptr, %v0 : !tt.ptr<f32>
1057|         scf.yield
1058|       } {triton.warp_pipeline.stage = "compute"}
1059| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, scf.execute_region, masked or vectorized loads, local/shared memory stores, ttg.memdesc_subslice.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、scf.execute_region、带掩码或向量化的加载、本地/共享内存存储、ttg.memdesc_subslice。

### Lines 1060-1067
```mlir
1060|       scf.yield %a, %s, %ld : tensor<256x16xf16, #adj_dot>, !ttg.memdesc<256x64xf16, #adj_shared, #adj_smem, mutable>, tensor<256x16xf16, #adj_dot>
1061|     } {triton.warp_pipeline.pipelined_for}
1062| 
1063|     ttg.local_dealloc %smem : !ttg.memdesc<256x64xf16, #adj_shared, #adj_smem, mutable>
1064|     tt.return
1065|   }
1066| }
1067| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, ttg.local_dealloc, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、ttg.local_dealloc、tt.return。

### Lines 1068-1071
```mlir
1068| // CHECK-LABEL: tt.func @adjacent_stage_lds_dep
1069| // CHECK: scf.for
1070| //
1071| // Stage 0 ops (local_store).
```
**EN:** This function-oriented block defines or enters `adjacent_stage_lds_dep`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `adjacent_stage_lds_dep` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1072-1074
```mlir
1072| // CHECK: ttg.local_store
1073| //
1074| // Barrier between stage0→stage1 is LOCAL (adjacent RAW: write→read).
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1075-1079
```mlir
1075| // CHECK: rocdl.sched.barrier
1076| // CHECK-NEXT: ttg.barrier local
1077| // CHECK-NEXT: rocdl.sched.barrier
1078| //
1079| // Stage 1 ops (local_load).
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1080-1082
```mlir
1080| // CHECK: ttg.local_load
1081| //
1082| // Barrier between stage1→stage2 is LOCAL (distance-2 WAR: a1 reads, a0 writes).
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1083-1087
```mlir
1083| // CHECK: rocdl.sched.barrier
1084| // CHECK-NEXT: ttg.barrier local
1085| // CHECK-NEXT: rocdl.sched.barrier
1086| //
1087| // Stage 2 ops (global store).
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1088-1090
```mlir
1088| // CHECK: tt.store
1089| //
1090| // Wrap-around barrier is s_barrier only (a2 has no LDS, a0 writes — no dep).
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1091-1094
```mlir
1091| // CHECK: rocdl.sched.barrier
1092| // CHECK-NEXT: rocdl.s.barrier
1093| // CHECK-NEXT: rocdl.sched.barrier
1094| //
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1095-1096
```mlir
1095| // CHECK: scf.yield
1096| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 1097-1097
```mlir
1097| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1098-1115
```mlir
1098| 
1099| // ---- Back-to-back: cross-pipeline dep in a later flat stage (b_1) ----
1100| //
1101| // This test exercises `collectNextPipelineClusters` when the next pipeline is
1102| // a flat (unrolled) sequence of more than one stage.  Before the fix, only
1103| // the first stage (b_0) was collected, so a cross-pipeline dependency
1104| // involving a later stage (b_1, b_2, …) was missed and the boundary barriers
1105| // were wrongly eliminated.
1106| //
1107| // Layout:
1108| //   Loop A (2 stages): a_0 tt.store         (no LDS)
1109| //                      a_1 ttg.local_load   (READS LDS)
1110| //   Flat B (2 stages): b_0 tt.store         (no LDS)
1111| //                      b_1 ttg.local_store  (WRITES the same LDS buffer)
1112| //
1113| // A's circular analysis finds no intersecting pair (a_1's read does not
1114| // conflict with itself or with a_0), so all of A's bars are non-LOCAL.
1115| // In particular the wrap-around bars[0] is FALSE, so it cannot seed
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1116-1126
```mlir
1116| // coverage for the merged boundary slot.
1117| //
1118| // Cross-pipeline dep: (a_1, b_1) WAR at merged distance 2, barrierLoc = K = 2
1119| // (the boundary).  No other slot on the path from a_1 to b_1 is LOCAL, so
1120| // the analysis must flag the boundary and preserve the post-loop
1121| // cond_barrier, prelude ttg.barrier local, and phase-shift cond_barrier.
1122| //
1123| // Before the collectNextPipelineClusters fix, the boundary barriers would
1124| // have been removed (false negative) because only b_0 was collected, making
1125| // b_1 invisible to the cross-pipeline analysis.
1126| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1127-1130
```mlir
1127| #crossb_blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 8], warpsPerCTA = [8, 1], order = [1, 0]}>
1128| #crossb_mma = #ttg.amd_mfma<{version = 3, warpsPerCTA = [2, 4], instrShape = [16, 16, 16], isTransposed = true}>
1129| #crossb_shared = #ttg.swizzled_shared<{vec = 4, perPhase = 1, maxPhase = 16, order = [1, 0]}>
1130| #crossb_smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1131-1131
```mlir
1131| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 1132-1149
```mlir
1132|   tt.func @cross_pipeline_dep_in_b1(
1133|       %lb: i32, %ub: i32, %step: i32,
1134|       %acc: tensor<256x16xf16, #ttg.dot_op<{opIdx = 0, parent = #crossb_mma, kWidth = 4}>>,
1135|       %ptr: tensor<256x64x!tt.ptr<f16>, #crossb_blocked>,
1136|       %gptr: !tt.ptr<f32>,
1137|       %dst: tensor<256x16x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 0, parent = #crossb_mma, kWidth = 4}>>) {
1138| 
1139|     %smem = ttg.local_alloc : () -> !ttg.memdesc<256x64xf16, #crossb_shared, #crossb_smem, mutable>
1140|     %v0 = arith.constant 0.0 : f32
1141|     %v1 = arith.constant 1.0 : f32
1142| 
1143|     // Loop A: stage 0 no LDS, stage 1 reads %smem.  The loaded value is
1144|     // threaded through iter_args + used after the loop so the execute_region
1145|     // (and its ttg.local_load) survives DCE before the redundant-barrier pass.
1146|     %final = scf.for %i = %lb to %ub step %step
1147|         iter_args(%cur = %acc)
1148|         -> tensor<256x16xf16, #ttg.dot_op<{opIdx = 0, parent = #crossb_mma, kWidth = 4}>> : i32 {
1149|       scf.execute_region no_inline {
```
**EN:** This function-oriented block defines or enters `cross_pipeline_dep_in_b1`. Within it, the test exercises tt.func, constants, shared/local memory allocation, structured loops, scf.execute_region, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `cross_pipeline_dep_in_b1` 为核心。测试在其中演示 tt.func、常量、共享/本地内存分配、结构化循环、scf.execute_region，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1150-1167
```mlir
1150|         tt.store %gptr, %v0 : !tt.ptr<f32>
1151|         scf.yield
1152|       } {triton.warp_pipeline.stage = "a_compute"}
1153| 
1154|       %ld = scf.execute_region -> tensor<256x16xf16, #ttg.dot_op<{opIdx = 0, parent = #crossb_mma, kWidth = 4}>> no_inline {
1155|         %sub = ttg.memdesc_subslice %smem[0, 0] : !ttg.memdesc<256x64xf16, #crossb_shared, #crossb_smem, mutable> -> !ttg.memdesc<256x16xf16, #crossb_shared, #crossb_smem, mutable, 256x64>
1156|         %v = ttg.local_load %sub : !ttg.memdesc<256x16xf16, #crossb_shared, #crossb_smem, mutable, 256x64> -> tensor<256x16xf16, #ttg.dot_op<{opIdx = 0, parent = #crossb_mma, kWidth = 4}>>
1157|         scf.yield %v : tensor<256x16xf16, #ttg.dot_op<{opIdx = 0, parent = #crossb_mma, kWidth = 4}>>
1158|       } {triton.warp_pipeline.stage = "a_load"}
1159| 
1160|       scf.yield %ld : tensor<256x16xf16, #ttg.dot_op<{opIdx = 0, parent = #crossb_mma, kWidth = 4}>>
1161|     } {triton.warp_pipeline.pipelined_for}
1162| 
1163|     // Flat B: b_0 no LDS (masks the bug), b_1 writes the same %smem (dep).
1164|     scf.execute_region no_inline {
1165|       tt.store %gptr, %v1 : !tt.ptr<f32>
1166|       scf.yield
1167|     } {triton.warp_pipeline.stage = "b_nolds"}
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, masked or vectorized stores, scf.execute_region, ttg.memdesc_subslice, local/shared memory loads.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、带掩码或向量化的存储、scf.execute_region、ttg.memdesc_subslice、本地/共享内存加载。

### Lines 1168-1184
```mlir
1168| 
1169|     scf.execute_region no_inline {
1170|       %data = tt.load %ptr : tensor<256x64x!tt.ptr<f16>, #crossb_blocked>
1171|       ttg.local_store %data, %smem : tensor<256x64xf16, #crossb_blocked> -> !ttg.memdesc<256x64xf16, #crossb_shared, #crossb_smem, mutable>
1172|       scf.yield
1173|     } {triton.warp_pipeline.stage = "b_lds"}
1174| 
1175|     // Use %final after flat B so the loop's iter_arg result is observed and
1176|     // the local_load execute_region survives DCE — without breaking the
1177|     // back-to-back boundary between loop A and flat B.
1178|     tt.store %dst, %final : tensor<256x16x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 0, parent = #crossb_mma, kWidth = 4}>>
1179| 
1180|     ttg.local_dealloc %smem : !ttg.memdesc<256x64xf16, #crossb_shared, #crossb_smem, mutable>
1181|     tt.return
1182|   }
1183| }
1184| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining scf.execute_region, masked or vectorized loads, local/shared memory stores, loop/if yielded values, masked or vectorized stores.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 scf.execute_region、带掩码或向量化的加载、本地/共享内存存储、循环/分支产出值、带掩码或向量化的存储。

### Lines 1185-1186
```mlir
1185| // CHECK-LABEL: tt.func @cross_pipeline_dep_in_b1
1186| // Pre-barrier and phase shift for loop A.
```
**EN:** This function-oriented block defines or enters `cross_pipeline_dep_in_b1`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `cross_pipeline_dep_in_b1` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1187-1190
```mlir
1187| // CHECK: ttg.barrier local
1188| // CHECK: amdg.cond_barrier
1189| // CHECK: scf.for
1190| // Loop body: a_0 (tt.store), internal s_barrier, a_1 (local_load).
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 1191-1196
```mlir
1191| // CHECK: tt.store
1192| // CHECK: rocdl.s.barrier
1193| // CHECK: ttg.local_load
1194| // Boundary barriers between loop A and flat B are KEPT because (a_1, b_1)
1195| // is a cross-pipeline WAR dep on %smem and no LOCAL barrier on the path
1196| // a_1 → boundary → b_0 → b_1 covers it (A's wrap-around is not LOCAL).
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1197-1200
```mlir
1197| // CHECK: amdg.cond_barrier
1198| // CHECK: ttg.barrier local
1199| // CHECK: amdg.cond_barrier
1200| // Flat B stages: b_0 (tt.store), internal s_barrier, b_1 (local_store).
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1201-1203
```mlir
1201| // CHECK: tt.store
1202| // CHECK: ttg.local_store
1203| // Reconverge cond_barrier for flat B.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1204-1206
```mlir
1204| // CHECK: amdg.cond_barrier
1205| // CHECK: tt.return
1206| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 1207-1207
```mlir
1207| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1208-1225
```mlir
1208| 
1209| // ---- Back-to-back: cross-pipeline dep where placement falls inside A ----
1210| //
1211| // Companion to @cross_pipeline_dep_in_b1.  Where that test puts the
1212| // uncovered cross-pipeline pair at distance == 1 (so the placement falls at
1213| // boundary slot K), this one engineers a pair at distance == K from `a_0`
1214| // to `b_0` so the placement falls at slot K-1 — *inside* A's body.
1215| // isCrossPipelineSafe must still flag this as unsafe: the explicit
1216| // cross-pipeline-pair sweep walks (src, barrierLoc] for coverage and finds
1217| // no LOCAL slot in A (loopBars[1..K-1] are all false).
1218| //
1219| // Layout:
1220| //   Loop A (2 stages): a_0 ttg.local_load   (READS LDS)
1221| //                      a_1 tt.store         (no LDS)
1222| //   Flat B (2 stages): b_0 ttg.local_store  (WRITES the same LDS buffer)
1223| //                      b_1 tt.store         (no LDS)
1224| //
1225| // A's circular analysis: a_0 read-read with itself, no intersection with a_1;
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1226-1232
```mlir
1226| // loopBars = [false, false] and the wrap-around is non-LOCAL.
1227| //
1228| // Cross-pipeline dep (a_0, b_0) WAR on %smem at merged distance K=2 →
1229| // barrierLoc = dst-1 = 1.  isCovered(0, 1) walks slot 1 (loopBars[1]=false)
1230| // and returns false; the pair is intersected → unsafe.  Boundary barriers
1231| // must be kept.
1232| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1233-1236
```mlir
1233| #crossa_blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 8], warpsPerCTA = [8, 1], order = [1, 0]}>
1234| #crossa_mma = #ttg.amd_mfma<{version = 3, warpsPerCTA = [2, 4], instrShape = [16, 16, 16], isTransposed = true}>
1235| #crossa_shared = #ttg.swizzled_shared<{vec = 4, perPhase = 1, maxPhase = 16, order = [1, 0]}>
1236| #crossa_smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1237-1237
```mlir
1237| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 1238-1255
```mlir
1238|   tt.func @cross_pipeline_dep_in_a0(
1239|       %lb: i32, %ub: i32, %step: i32,
1240|       %acc: tensor<256x16xf16, #ttg.dot_op<{opIdx = 0, parent = #crossa_mma, kWidth = 4}>>,
1241|       %ptr: tensor<256x64x!tt.ptr<f16>, #crossa_blocked>,
1242|       %gptr: !tt.ptr<f32>,
1243|       %dst: tensor<256x16x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 0, parent = #crossa_mma, kWidth = 4}>>) {
1244| 
1245|     %smem = ttg.local_alloc : () -> !ttg.memdesc<256x64xf16, #crossa_shared, #crossa_smem, mutable>
1246|     %v0 = arith.constant 0.0 : f32
1247| 
1248|     // Loop A: stage 0 reads %smem (threaded through iter_args so the
1249|     // local_load survives DCE), stage 1 no LDS.
1250|     %final = scf.for %i = %lb to %ub step %step
1251|         iter_args(%cur = %acc)
1252|         -> tensor<256x16xf16, #ttg.dot_op<{opIdx = 0, parent = #crossa_mma, kWidth = 4}>> : i32 {
1253|       %ld = scf.execute_region -> tensor<256x16xf16, #ttg.dot_op<{opIdx = 0, parent = #crossa_mma, kWidth = 4}>> no_inline {
1254|         %sub = ttg.memdesc_subslice %smem[0, 0] : !ttg.memdesc<256x64xf16, #crossa_shared, #crossa_smem, mutable> -> !ttg.memdesc<256x16xf16, #crossa_shared, #crossa_smem, mutable, 256x64>
1255|         %v = ttg.local_load %sub : !ttg.memdesc<256x16xf16, #crossa_shared, #crossa_smem, mutable, 256x64> -> tensor<256x16xf16, #ttg.dot_op<{opIdx = 0, parent = #crossa_mma, kWidth = 4}>>
```
**EN:** This function-oriented block defines or enters `cross_pipeline_dep_in_a0`. Within it, the test exercises tt.func, shared/local memory allocation, constants, structured loops, scf.execute_region, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `cross_pipeline_dep_in_a0` 为核心。测试在其中演示 tt.func、共享/本地内存分配、常量、结构化循环、scf.execute_region，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1256-1273
```mlir
1256|         scf.yield %v : tensor<256x16xf16, #ttg.dot_op<{opIdx = 0, parent = #crossa_mma, kWidth = 4}>>
1257|       } {triton.warp_pipeline.stage = "a_load"}
1258| 
1259|       scf.execute_region no_inline {
1260|         tt.store %gptr, %v0 : !tt.ptr<f32>
1261|         scf.yield
1262|       } {triton.warp_pipeline.stage = "a_compute"}
1263| 
1264|       scf.yield %ld : tensor<256x16xf16, #ttg.dot_op<{opIdx = 0, parent = #crossa_mma, kWidth = 4}>>
1265|     } {triton.warp_pipeline.pipelined_for}
1266| 
1267|     // Flat B: b_0 writes %smem (the dep), b_1 no LDS.
1268|     scf.execute_region no_inline {
1269|       %data = tt.load %ptr : tensor<256x64x!tt.ptr<f16>, #crossa_blocked>
1270|       ttg.local_store %data, %smem : tensor<256x64xf16, #crossa_blocked> -> !ttg.memdesc<256x64xf16, #crossa_shared, #crossa_smem, mutable>
1271|       scf.yield
1272|     } {triton.warp_pipeline.stage = "b_lds"}
1273| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, scf.execute_region, masked or vectorized stores, masked or vectorized loads, local/shared memory stores.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、scf.execute_region、带掩码或向量化的存储、带掩码或向量化的加载、本地/共享内存存储。

### Lines 1274-1285
```mlir
1274|     scf.execute_region no_inline {
1275|       tt.store %gptr, %v0 : !tt.ptr<f32>
1276|       scf.yield
1277|     } {triton.warp_pipeline.stage = "b_nolds"}
1278| 
1279|     tt.store %dst, %final : tensor<256x16x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 0, parent = #crossa_mma, kWidth = 4}>>
1280| 
1281|     ttg.local_dealloc %smem : !ttg.memdesc<256x64xf16, #crossa_shared, #crossa_smem, mutable>
1282|     tt.return
1283|   }
1284| }
1285| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining masked or vectorized stores, scf.execute_region, loop/if yielded values, ttg.local_dealloc, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 带掩码或向量化的存储、scf.execute_region、循环/分支产出值、ttg.local_dealloc、tt.return。

### Lines 1286-1287
```mlir
1286| // CHECK-LABEL: tt.func @cross_pipeline_dep_in_a0
1287| // Pre-barrier and phase shift for loop A.
```
**EN:** This function-oriented block defines or enters `cross_pipeline_dep_in_a0`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `cross_pipeline_dep_in_a0` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1288-1291
```mlir
1288| // CHECK: ttg.barrier local
1289| // CHECK: amdg.cond_barrier
1290| // CHECK: scf.for
1291| // Loop body: a_0 (local_load), internal s_barrier, a_1 (tt.store).
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 1292-1299
```mlir
1292| // CHECK: ttg.local_load
1293| // CHECK: rocdl.s.barrier
1294| // CHECK: tt.store
1295| // Boundary barriers between loop A and flat B must be KEPT.  The (a_0, b_0)
1296| // WAR on %smem at merged distance K places at slot K-1 (inside A); the
1297| // cross-pipeline-pair sweep finds no LOCAL slot in (0, K-1] (loopBars[1] is
1298| // false because A's intra-cluster barrier is just s_barrier) and reports
1299| // the pair as uncovered.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1300-1303
```mlir
1300| // CHECK: amdg.cond_barrier
1301| // CHECK: ttg.barrier local
1302| // CHECK: amdg.cond_barrier
1303| // Flat B stages: b_0 (local_store), internal s_barrier, b_1 (tt.store).
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1304-1306
```mlir
1304| // CHECK: ttg.local_store
1305| // CHECK: tt.store
1306| // Reconverge cond_barrier for flat B.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1307-1309
```mlir
1307| // CHECK: amdg.cond_barrier
1308| // CHECK: tt.return
1309| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 1310-1310
```mlir
1310| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1311-1320
```mlir
1311| 
1312| // ---- LDS effect nested inside scf.if must be detected ----
1313| //
1314| // Stage 0 wraps its ttg.local_store inside an scf.if, so the effect is not
1315| // visible on the top-level op.  buildBlockInfoFromBlock must walk
1316| // recursively to discover it; otherwise the cross-cluster RAW (stage0
1317| // writes, stage1 reads) is missed and the cluster barriers degrade from
1318| // ttg.barrier local to plain rocdl.s.barrier — leaving the LDS race
1319| // uncovered.
1320| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 1321-1325
```mlir
1321| #nest_blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 8], warpsPerCTA = [8, 1], order = [1, 0]}>
1322| #nest_mma = #ttg.amd_mfma<{version = 3, warpsPerCTA = [2, 4], instrShape = [16, 16, 16], isTransposed = true}>
1323| #nest_dot = #ttg.dot_op<{opIdx = 0, parent = #nest_mma, kWidth = 4}>
1324| #nest_shared = #ttg.swizzled_shared<{vec = 4, perPhase = 1, maxPhase = 16, order = [1, 0]}>
1325| #nest_smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1326-1326
```mlir
1326| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 1327-1344
```mlir
1327|   tt.func @nested_lds_effect_in_if(
1328|       %lb: i32, %ub: i32, %step: i32,
1329|       %cond: i1,
1330|       %acc: tensor<256x16xf16, #nest_dot>,
1331|       %ptr: tensor<256x64x!tt.ptr<f16>, #nest_blocked>) {
1332| 
1333|     %smem = ttg.local_alloc : () -> !ttg.memdesc<256x64xf16, #nest_shared, #nest_smem, mutable>
1334| 
1335|     %r:2 = scf.for %i = %lb to %ub step %step
1336|         iter_args(%a = %acc, %s = %smem)
1337|         -> (tensor<256x16xf16, #nest_dot>, !ttg.memdesc<256x64xf16, #nest_shared, #nest_smem, mutable>) : i32 {
1338| 
1339|       // Stage 0: conditionally writes LDS via scf.if.  The ttg.local_store
1340|       // sits inside the if body, so a flat scan of the cluster body would
1341|       // miss it.
1342|       %st = scf.execute_region -> !ttg.memdesc<256x64xf16, #nest_shared, #nest_smem, mutable> no_inline {
1343|         scf.if %cond {
1344|           %data = tt.load %ptr : tensor<256x64x!tt.ptr<f16>, #nest_blocked>
```
**EN:** This function-oriented block defines or enters `nested_lds_effect_in_if`. Within it, the test exercises tt.func, shared/local memory allocation, structured loops, scf.execute_region, structured conditionals, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `nested_lds_effect_in_if` 为核心。测试在其中演示 tt.func、共享/本地内存分配、结构化循环、scf.execute_region、结构化条件分支，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1345-1362
```mlir
1345|           ttg.local_store %data, %s : tensor<256x64xf16, #nest_blocked> -> !ttg.memdesc<256x64xf16, #nest_shared, #nest_smem, mutable>
1346|         }
1347|         scf.yield %s : !ttg.memdesc<256x64xf16, #nest_shared, #nest_smem, mutable>
1348|       } {triton.warp_pipeline.stage = "cond_store"}
1349| 
1350|       // Stage 1: reads LDS — RAW with the conditional write in stage 0.
1351|       %ld = scf.execute_region -> tensor<256x16xf16, #nest_dot> no_inline {
1352|         %sub = ttg.memdesc_subslice %s[0, 0] : !ttg.memdesc<256x64xf16, #nest_shared, #nest_smem, mutable> -> !ttg.memdesc<256x16xf16, #nest_shared, #nest_smem, mutable, 256x64>
1353|         %v = ttg.local_load %sub : !ttg.memdesc<256x16xf16, #nest_shared, #nest_smem, mutable, 256x64> -> tensor<256x16xf16, #nest_dot>
1354|         scf.yield %v : tensor<256x16xf16, #nest_dot>
1355|       } {triton.warp_pipeline.stage = "lds_load"}
1356| 
1357|       scf.yield %ld, %s : tensor<256x16xf16, #nest_dot>, !ttg.memdesc<256x64xf16, #nest_shared, #nest_smem, mutable>
1358|     } {triton.warp_pipeline.pipelined_for}
1359| 
1360|     ttg.local_dealloc %smem : !ttg.memdesc<256x64xf16, #nest_shared, #nest_smem, mutable>
1361|     tt.return
1362|   }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, local/shared memory stores, scf.execute_region, ttg.memdesc_subslice, local/shared memory loads.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、本地/共享内存存储、scf.execute_region、ttg.memdesc_subslice、本地/共享内存加载。

### Lines 1363-1364
```mlir
1363| }
1364| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 1365-1367
```mlir
1365| // CHECK-LABEL: tt.func @nested_lds_effect_in_if
1366| // CHECK: scf.for
1367| // Stage 0 with the nested scf.if + local_store.
```
**EN:** This function-oriented block defines or enters `nested_lds_effect_in_if`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `nested_lds_effect_in_if` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1368-1370
```mlir
1368| // CHECK: scf.if
1369| // CHECK:   ttg.local_store
1370| // Cluster barrier between stage 0 and stage 1 is LOCAL (nested write seen).
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 1371-1374
```mlir
1371| // CHECK: rocdl.sched.barrier
1372| // CHECK-NEXT: ttg.barrier local
1373| // CHECK-NEXT: rocdl.sched.barrier
1374| // Stage 1 reads LDS.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1375-1376
```mlir
1375| // CHECK: ttg.local_load
1376| // Wrap-around barrier is also LOCAL (stage1 read vs stage0 write next iter).
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1377-1380
```mlir
1377| // CHECK: rocdl.sched.barrier
1378| // CHECK-NEXT: ttg.barrier local
1379| // CHECK-NEXT: rocdl.sched.barrier
1380| // CHECK: scf.yield
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `-convert-warp-pipeline="gfx-arch=gfx1250"`, `-convert-warp-pipeline="gfx-arch=gfx950"`
- **CN:** 主要 pass 选项：`-split-input-file`，`-convert-warp-pipeline="gfx-arch=gfx1250"`，`-convert-warp-pipeline="gfx-arch=gfx950"`
- **EN:** Dominant operations include `scf.yield`, `scf.execute_region`, `arith.constant`, `tt.func`, `tt.store`, `ttg.local_load`, `scf.for`, `tt.return`, `ttg.memdesc_subslice`, `module`.
- **CN:** 主要操作包括 `scf.yield`、`scf.execute_region`、`arith.constant`、`tt.func`、`tt.store`、`ttg.local_load`、`scf.for`、`tt.return`、`ttg.memdesc_subslice`、`module`。
- **EN:** The file contains 13 independently testable section(s). Check styles used: CHECK x178, CHECK-NOT x22, CHECK-LABEL x18, CHECK-NEXT x14. Important labels include tt.func @two_stage_backend(, tt.func @three_stage_backend(, tt.func public @eight_stage_dependency, tt.func public @triple_buf_2stage. It validates both the presence of the desired rewrite and the absence of unwanted residual operations.
- **CN:** 该文件包含 13 个可独立测试的分段。使用的检查类型：CHECK ×178，CHECK-NOT ×22，CHECK-LABEL ×18，CHECK-NEXT ×14。 关键标签包括 tt.func @two_stage_backend(，tt.func @three_stage_backend(，tt.func public @eight_stage_dependency，tt.func public @triple_buf_2stage。 它同时验证目标重写已经出现，并且不再残留不需要的操作。

## Dependencies / 依赖关系

- **EN:** This test depends on Triton/MLIR textual IR parsing, the pass pipeline named in the RUN lines, and the dialect/type definitions referenced in the file.
- **CN:** 该测试依赖 Triton/MLIR 文本 IR 解析、RUN 行中指定的 pass 流水线，以及文件中引用的方言/类型定义。
- **EN:** `ttg`: TritonGPU layout, memory, and GPU-specific operations.
- **CN:** `ttg`：TritonGPU 布局、内存和 GPU 专用操作。
- **EN:** `tt`: Triton high-level tensor/pointer operations.
- **CN:** `tt`：Triton 高层张量/指针操作。
- **EN:** `scf`: Structured control flow for loops and conditionals.
- **CN:** `scf`：用于循环与条件分支的结构化控制流。
- **EN:** `arith`: MLIR arithmetic dialect for scalar/tensor math.
- **CN:** `arith`：用于标量/张量运算的 MLIR arithmetic 方言。
- **EN:** `FileCheck` is used to match required textual patterns after rewriting/lowering.
- **CN:** `FileCheck` 用于在重写/lower 之后匹配必须出现的文本模式。