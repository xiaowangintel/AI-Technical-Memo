# amd-warp-pipeline.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/amd/amd-warp-pipeline.mlir`
- **EN:** Test for the MLIR/Triton pipeline `-tritonamdgpu-warp-pipeline` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `-tritonamdgpu-warp-pipeline` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -split-input-file -tritonamdgpu-warp-pipeline | FileCheck %s`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -split-input-file -tritonamdgpu-warp-pipeline | FileCheck %s`

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
1| // RUN: triton-opt %s -split-input-file -tritonamdgpu-warp-pipeline | FileCheck %s
2| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -split-input-file -tritonamdgpu-warp-pipeline | FileCheck %s` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -split-input-file -tritonamdgpu-warp-pipeline | FileCheck %s`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 3-8
```mlir
3| #linear = #ttg.linear<{register = [[1, 0], [2, 0], [4, 0], [0, 4]], lane = [[8, 0], [16, 0], [32, 0], [64, 0], [128, 0], [0, 16]], warp = [[0, 1], [0, 2], [0, 8]], block = []}>
4| #linear1 = #ttg.linear<{register = [[0, 1], [0, 2], [0, 4], [4, 0]], lane = [[0, 8], [0, 16], [0, 32], [0, 64], [0, 128], [16, 0]], warp = [[1, 0], [2, 0], [8, 0]], block = []}>
5| #mma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [2, 4], instrShape = [16, 16, 32], isTransposed = true}>
6| #shared = #ttg.padded_shared<[512:+16] {offset = [[1, 0], [2, 0], [4, 0], [8, 0], [16, 0], [32, 0], [64, 0], [128, 0], [0, 16], [0, 1], [0, 2], [0, 8], [0, 4]], block = []}>
7| #shared1 = #ttg.padded_shared<[512:+16] {offset = [[0, 1], [0, 2], [0, 4], [0, 8], [0, 16], [0, 32], [0, 64], [0, 128], [16, 0], [1, 0], [2, 0], [8, 0], [4, 0]], block = []}>
8| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 9-11
```mlir
 9| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
10| 
11| // -- 3-stage example (two borders) ----
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 12-29
```mlir
12| tt.func @three_stage_example(%n: index) {
13|   %c0  = arith.constant 0 : index
14|   %c1  = arith.constant 1 : index
15| 
16|   scf.for %i = %c0 to %n step %c1 {
17|     // Stage 0 (before first border)
18|     %a  = arith.addi %i, %c1 : index
19|     %a2 = arith.muli %a, %c1 : index
20| 
21|     // explicit split point
22|     rocdl.sched.barrier 0 {triton.warp_pipeline.border="stage"}
23| 
24|     // Stage 1
25|     %b  = arith.addi %a2, %i : index
26| 
27|     // explicit split point
28|     rocdl.sched.barrier 0 {triton.warp_pipeline.border="stage"}
29| 
```
**EN:** This function-oriented block defines or enters `three_stage_example`. Within it, the test exercises tt.func, constants, integer additions, rocdl.sched, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `three_stage_example` 为核心。测试在其中演示 tt.func、常量、整数加法、rocdl.sched、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 30-39
```mlir
30|     // Stage 2
31|     %c  = arith.addi %b, %a : index
32|     %d  = arith.muli %c, %c1 : index
33| 
34|     scf.yield
35|   }
36| 
37|   tt.return
38| }
39| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining integer additions, integer multiplications, loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 整数加法、整数乘法、循环/分支产出值、tt.return。

### Lines 40-43
```mlir
40| // CHECK-LABEL: tt.func @three_stage_example(
41| // CHECK: scf.for
42| //
43| // Inside the loop we expect exactly three execute_region clusters:
```
**EN:** This function-oriented block defines or enters `three_stage_example`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `three_stage_example` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 44-57
```mlir
44| // CHECK:   scf.execute_region
45| // CHECK:     arith.addi
46| // CHECK:     arith.muli
47| // CHECK:     scf.yield
48| // CHECK:   scf.execute_region
49| // CHECK:     arith.addi
50| // CHECK:     scf.yield
51| // CHECK:   scf.execute_region
52| // CHECK:     arith.addi
53| // CHECK:     arith.muli
54| // CHECK:     scf.yield
55| // CHECK: triton.warp_pipeline.pipelined_for
56| //
57| // And the split markers must be gone:
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 58-63
```mlir
58| // CHECK-NOT: rocdl.sched.barrier
59| // CHECK: tt.return
60| 
61| 
62| // -- 2-stage example (one border) ----
63| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 64-81
```mlir
64| tt.func @two_stage_example(%n: index) {
65|   %c0  = arith.constant 0 : index
66|   %c1  = arith.constant 1 : index
67| 
68|   scf.for %i = %c0 to %n step %c1 {
69|     // Stage 0
70|     %x = arith.addi %i, %c1 : index
71| 
72|     // split to Stage 1
73|     rocdl.sched.barrier 0 {triton.warp_pipeline.border="stage"}
74| 
75|     // Stage 1
76|     %y = arith.muli %x, %c1 : index
77| 
78|     scf.yield
79|   }
80| 
81|   tt.return
```
**EN:** This function-oriented block defines or enters `two_stage_example`. Within it, the test exercises tt.func, constants, structured loops, integer additions, rocdl.sched, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `two_stage_example` 为核心。测试在其中演示 tt.func、常量、结构化循环、整数加法、rocdl.sched，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 82-83
```mlir
82| }
83| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 84-97
```mlir
84| // CHECK-LABEL: tt.func @two_stage_example(
85| // CHECK: scf.for
86| // CHECK:   scf.execute_region
87| // CHECK:     arith.addi
88| // CHECK:     scf.yield
89| // CHECK:   scf.execute_region
90| // CHECK:     arith.muli
91| // CHECK:     scf.yield
92| // CHECK: triton.warp_pipeline.pipelined_for
93| // CHECK-NOT: rocdl.sched.barrier
94| // CHECK: tt.return
95| 
96| // -- pipelining with pre-existing barrier (ignorable ops) ----
97| 
```
**EN:** This function-oriented block defines or enters `two_stage_example`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `two_stage_example` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 98-115
```mlir
 98| // CHECK-LABEL: tt.func public @triple_buf_two_stages
 99| // CHECK: scf.for
100| // CHECK:   scf.execute_region
101| // CHECK:     local_load
102| // CHECK:     local_load
103| // CHECK:     async_copy_global_to_local
104| // CHECK:     async_commit_group
105| // CHECK:     scf.yield
106| // CHECK:   triton.warp_pipeline.stage
107| // CHECK:   ttg.async_wait
108| // CHECK:   scf.execute_region
109| // CHECK:     async_copy_global_to_local
110| // CHECK:     async_commit_group
111| // CHECK:     tt.dot
112| // CHECK:     scf.yield
113| // CHECK:   triton.warp_pipeline.stage
114| // CHECK: triton.warp_pipeline.pipelined_for
115| // CHECK-NOT: rocdl.sched.barrier
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: tt.func public @triple_buf_two_stages anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: tt.func public @triple_buf_two_stages 这样的标签用于锚定匹配范围。

### Lines 116-117
```mlir
116| // CHECK: tt.return
117| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 118-135
```mlir
118| tt.func public @triple_buf_two_stages(%arg0: i32, %arg1: i32, %arg2: i32, %arg3: i32, %arg4: tensor<256x256xf32, #mma>, %arg5: i32, %arg6: i32, %arg7: tensor<256x32xi32, #linear>, %arg8: tensor<32x256xi32, #linear1>, %arg9: !ttg.memdesc<256x32xbf16, #shared, #smem, mutable>, %arg10: !ttg.memdesc<32x256xbf16, #shared1, #smem, mutable>, %arg11: !ttg.memdesc<256x32xbf16, #shared, #smem, mutable>, %arg12: !ttg.memdesc<32x256xbf16, #shared1, #smem, mutable>, %arg13: !ttg.async.token, %arg14: !ttg.async.token, %arg15: !ttg.async.token, %arg16: tensor<256x32x!tt.ptr<bf16>, #linear>, %arg17: tensor<32x256x!tt.ptr<bf16>, #linear1>, %arg18: tensor<256xi64, #ttg.slice<{dim = 1, parent = #mma}>>, %arg19: tensor<256xi64, #ttg.slice<{dim = 0, parent = #mma}>>, %arg20: i64, %arg21: i64, %arg22: !tt.ptr<bf16>, %arg23: i32) attributes {noinline = false} {
119|   %0 = ttg.local_alloc : () -> !ttg.memdesc<3x256x32xbf16, #shared, #smem, mutable>
120|   %1 = ttg.local_alloc : () -> !ttg.memdesc<3x32x256xbf16, #shared1, #smem, mutable>
121|   %2:11 = scf.for %arg24 = %arg0 to %arg6 step %arg1 iter_args(%arg25 = %arg4, %arg26 = %arg1, %arg27 = %arg9, %arg28 = %arg11, %arg29 = %arg13, %arg30 = %arg10, %arg31 = %arg12, %arg32 = %arg14, %arg33 = %arg15, %arg34 = %arg16, %arg35 = %arg17) -> (tensor<256x256xf32, #mma>, i32, !ttg.memdesc<256x32xbf16, #shared, #smem, mutable>, !ttg.memdesc<256x32xbf16, #shared, #smem, mutable>, !ttg.async.token, !ttg.memdesc<32x256xbf16, #shared1, #smem, mutable>, !ttg.memdesc<32x256xbf16, #shared1, #smem, mutable>, !ttg.async.token, !ttg.async.token, tensor<256x32x!tt.ptr<bf16>, #linear>, tensor<32x256x!tt.ptr<bf16>, #linear1>)  : i32 {
122|     %32 = tt.addptr %arg34, %arg7 : tensor<256x32x!tt.ptr<bf16>, #linear>, tensor<256x32xi32, #linear>
123|     %33 = tt.addptr %arg35, %arg8 : tensor<32x256x!tt.ptr<bf16>, #linear1>, tensor<32x256xi32, #linear1>
124|     %34 = arith.addi %arg26, %arg1 : i32
125|     %35 = arith.cmpi slt, %34, %arg3 : i32
126|     %36 = arith.select %35, %34, %arg0 : i32
127|     %37 = ttg.memdesc_index %0[%36] : !ttg.memdesc<3x256x32xbf16, #shared, #smem, mutable> -> !ttg.memdesc<256x32xbf16, #shared, #smem, mutable>
128|     %38 = ttg.memdesc_index %1[%36] : !ttg.memdesc<3x32x256xbf16, #shared1, #smem, mutable> -> !ttg.memdesc<32x256xbf16, #shared1, #smem, mutable>
129|     %39 = ttg.local_load %arg27 token %arg29 : !ttg.memdesc<256x32xbf16, #shared, #smem, mutable> -> tensor<256x32xbf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>>
130|     %40 = ttg.local_load %arg30 token %arg29 : !ttg.memdesc<32x256xbf16, #shared1, #smem, mutable> -> tensor<32x256xbf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>>
131|     %41 = ttg.async_copy_global_to_local %32, %37 : tensor<256x32x!tt.ptr<bf16>, #linear> -> <256x32xbf16, #shared, #smem, mutable>
132|     %42 = ttg.async_commit_group tokens %41
133|     rocdl.sched.barrier 0 {triton.warp_pipeline.border = "stage"}
134|     %43 = ttg.async_wait %arg32, %arg33 {num = 0 : i32}
135|     %44 = ttg.async_copy_global_to_local %33, %38 : tensor<32x256x!tt.ptr<bf16>, #linear1> -> <32x256xbf16, #shared1, #smem, mutable>
```
**EN:** This function-oriented block defines or enters `triple_buf_two_stages`. Within it, the test exercises tt.func, shared/local memory allocation, pointer arithmetic, ttg.memdesc_index, local/shared memory loads, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `triple_buf_two_stages` 为核心。测试在其中演示 tt.func、共享/本地内存分配、指针算术、ttg.memdesc_index、本地/共享内存加载，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 136-152
```mlir
136|     %45 = ttg.async_commit_group tokens %44
137|     %46 = tt.dot %39, %40, %arg25 : tensor<256x32xbf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>> * tensor<32x256xbf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>> -> tensor<256x256xf32, #mma>
138|     rocdl.sched.barrier 0 {triton.warp_pipeline.border = "stage"}
139|     scf.yield %46, %36, %arg28, %37, %43, %arg31, %38, %42, %45, %32, %33 : tensor<256x256xf32, #mma>, i32, !ttg.memdesc<256x32xbf16, #shared, #smem, mutable>, !ttg.memdesc<256x32xbf16, #shared, #smem, mutable>, !ttg.async.token, !ttg.memdesc<32x256xbf16, #shared1, #smem, mutable>, !ttg.memdesc<32x256xbf16, #shared1, #smem, mutable>, !ttg.async.token, !ttg.async.token, tensor<256x32x!tt.ptr<bf16>, #linear>, tensor<32x256x!tt.ptr<bf16>, #linear1>
140|   }
141|   ttg.local_dealloc %1 : !ttg.memdesc<3x32x256xbf16, #shared1, #smem, mutable>
142|   ttg.local_dealloc %0 : !ttg.memdesc<3x256x32xbf16, #shared, #smem, mutable>
143|   tt.return
144| }
145| 
146| // -- Flat (unrolled) pipeline: borders outside scf.for ----
147| //
148| // Simulates a static_range epilogue that was unrolled at the Python level
149| // following a regular pipelined main loop.  The flat backward walk must stop
150| // at the prior scf.for (loops are disallowed inside a stage) so the main
151| // loop is not absorbed into stage 0.
152| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining ttg.local_dealloc, async copy commit groups, dot-product or MMA-style math, rocdl.sched, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 ttg.local_dealloc、异步拷贝提交组、点积或 MMA 风格计算、rocdl.sched、循环/分支产出值。

### Lines 153-170
```mlir
153| tt.func @flat_pipeline_example(%n: index) {
154|   %c0  = arith.constant 0 : index
155|   %c1  = arith.constant 1 : index
156| 
157|   // Pipelined main loop: gets the pipelined_for attribute and acts as a
158|   // hard boundary for the flat epilogue's backward walk.
159|   scf.for %i = %c0 to %n step %c1 {
160|     %x = arith.addi %i, %c1 : index
161|     rocdl.sched.barrier 0 {triton.warp_pipeline.border = "load"}
162|     %y = arith.muli %x, %c1 : index
163|     scf.yield
164|   }
165| 
166|   // Stage 0 (ops before the first epilogue border)
167|   %a  = arith.addi %c0, %c1 : index
168|   %a2 = arith.muli %a, %c1 : index
169| 
170|   rocdl.sched.barrier 0 {triton.warp_pipeline.border = "stage0_epi", triton.warp_pipeline.priority = 1 : i32}
```
**EN:** This function-oriented block defines or enters `flat_pipeline_example`. Within it, the test exercises tt.func, constants, integer additions, rocdl.sched, integer multiplications, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `flat_pipeline_example` 为核心。测试在其中演示 tt.func、常量、整数加法、rocdl.sched、整数乘法，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 171-180
```mlir
171| 
172|   // Stage 1
173|   %b  = arith.addi %a2, %c0 : index
174|   %b2 = arith.muli %b, %c1 : index
175| 
176|   rocdl.sched.barrier 0 {triton.warp_pipeline.border = "stage1_epi", triton.warp_pipeline.priority = 0 : i32}
177| 
178|   tt.return
179| }
180| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on integer additions, integer multiplications, rocdl.sched, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 整数加法、整数乘法、rocdl.sched、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 181-183
```mlir
181| // CHECK-LABEL: tt.func @flat_pipeline_example(
182| // Pipelined main loop forms its own warp pipeline (one execute_region per
183| // stage, then the pipelined_for attribute on the loop).
```
**EN:** This function-oriented block defines or enters `flat_pipeline_example`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `flat_pipeline_example` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 184-189
```mlir
184| // CHECK: scf.for
185| // CHECK:   scf.execute_region
186| // CHECK:   scf.execute_region
187| // CHECK: triton.warp_pipeline.pipelined_for
188| // Flat epilogue execute_regions created from the borders.  Crucially, they
189| // must NOT absorb the pipelined main loop above.
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 190-202
```mlir
190| // CHECK: scf.execute_region
191| // CHECK:   arith.addi
192| // CHECK:   arith.muli
193| // CHECK:   scf.yield
194| // CHECK: triton.warp_pipeline.priority = 1
195| // CHECK-SAME: triton.warp_pipeline.stage = "stage0_epi"
196| // CHECK: scf.execute_region
197| // CHECK:   arith.addi
198| // CHECK:   arith.muli
199| // CHECK:   scf.yield
200| // CHECK: triton.warp_pipeline.priority = 0
201| // CHECK-SAME: triton.warp_pipeline.stage = "stage1_epi"
202| // Border markers must be erased:
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 203-211
```mlir
203| // CHECK-NOT: rocdl.sched.barrier
204| // CHECK: tt.return
205| 
206| // -- Post-unroll IV remap is sunk past ignorable ops (FA-kernel pattern) ----
207| // The FA kernel body begins with async_wait.  After MLIR loop unrolling, IV
208| // remap ops (arith.addi/muli) land between the last border of iter N and the
209| // async_wait at the start of iter N+1, which would otherwise poison cluster
210| // building.  The sink pre-pass moves scalar ops past adjacent ignorable ops so
211| // they join the next cluster naturally.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 212-229
```mlir
212| tt.func @unroll_iv_remap_sunk_past_async_wait(%n: index, %ptr: !tt.ptr<f32>) {
213|   %c0 = arith.constant 0 : index
214|   %c1 = arith.constant 1 : index
215|   %c2 = arith.constant 2 : index
216|   %v0 = arith.constant 0.0 : f32
217| 
218|   scf.for %i = %c0 to %n step %c2 {
219|     // iter 0: async_wait FIRST, then stage1 / stage2 bodies.
220|     ttg.async_wait {num = 0 : i32}
221|     tt.store %ptr, %v0 : !tt.ptr<f32>
222|     rocdl.sched.barrier 0 {triton.warp_pipeline.border = "stage1"}
223|     tt.store %ptr, %v0 : !tt.ptr<f32>
224|     rocdl.sched.barrier 0 {triton.warp_pipeline.border = "stage2"}
225| 
226|     // IV remap injected by unroller; sits between iter-0 last border and
227|     // iter-1 async_wait -- the poisonous spot.
228|     %i_1 = arith.addi %i, %c1 : index
229| 
```
**EN:** This function-oriented block defines or enters `unroll_iv_remap_sunk_past_async_wait`. Within it, the test exercises constants, tt.func, masked or vectorized stores, rocdl.sched, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `unroll_iv_remap_sunk_past_async_wait` 为核心。测试在其中演示 常量、tt.func、带掩码或向量化的存储、rocdl.sched、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 230-242
```mlir
230|     // iter 1: async_wait FIRST, then stage1 (uses %i_1) / stage2.
231|     ttg.async_wait {num = 0 : i32}
232|     %off = arith.muli %i_1, %c1 : index
233|     tt.store %ptr, %v0 : !tt.ptr<f32>
234|     rocdl.sched.barrier 0 {triton.warp_pipeline.border = "stage1"}
235|     tt.store %ptr, %v0 : !tt.ptr<f32>
236|     rocdl.sched.barrier 0 {triton.warp_pipeline.border = "stage2"}
237| 
238|     scf.yield
239|   }
240|   tt.return
241| }
242| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining masked or vectorized stores, rocdl.sched, async wait synchronization, integer multiplications, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 带掩码或向量化的存储、rocdl.sched、异步等待同步、整数乘法、循环/分支产出值。

### Lines 243-245
```mlir
243| // CHECK-LABEL: tt.func @unroll_iv_remap_sunk_past_async_wait(
244| // CHECK: scf.for
245| // iter 0: async_wait, stage1 region, stage2 region.
```
**EN:** This function-oriented block defines or enters `unroll_iv_remap_sunk_past_async_wait`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `unroll_iv_remap_sunk_past_async_wait` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 246-251
```mlir
246| // CHECK:   ttg.async_wait
247| // CHECK:   scf.execute_region
248| // CHECK:     tt.store
249| // CHECK:   scf.execute_region
250| // CHECK:     tt.store
251| // iter 1 starts with async_wait; IV remap was sunk past it into iter-1 stage1.
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 252-260
```mlir
252| // CHECK:   ttg.async_wait
253| // CHECK:   scf.execute_region {{.*}} {
254| // CHECK-NEXT: arith.addi
255| // CHECK-NEXT: arith.muli
256| // CHECK:     tt.store
257| // CHECK:   scf.execute_region
258| // CHECK:     tt.store
259| // CHECK: triton.warp_pipeline.pipelined_for
260| // No free arith ops or leftover sched.barrier markers in the loop body.
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 261-264
```mlir
261| // CHECK-NOT: rocdl.sched.barrier
262| // CHECK: tt.return
263| 
264| // -- Negative: no border → no structuring ----
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 265-277
```mlir
265| tt.func @no_split_example(%n: index) {
266|   %c0  = arith.constant 0 : index
267|   %c1  = arith.constant 1 : index
268| 
269|   scf.for %i = %c0 to %n step %c1 {
270|     %x = arith.addi %i, %c1 : index
271|     %y = arith.muli %x, %c1 : index
272|     scf.yield
273|   }
274| 
275|   tt.return
276| }
277| }
```
**EN:** This function-oriented block defines or enters `no_split_example`. Within it, the test exercises tt.func, constants, structured loops, integer additions, integer multiplications, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `no_split_example` 为核心。测试在其中演示 tt.func、常量、结构化循环、整数加法、整数乘法，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 278-282
```mlir
278| // CHECK-LABEL: tt.func @no_split_example(
279| // CHECK: scf.for
280| // CHECK-NOT: scf.execute_region
281| // CHECK-NOT: pipelined_for
282| // CHECK: tt.return
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: tt.func @no_split_example( anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: tt.func @no_split_example( 这样的标签用于锚定匹配范围。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `-tritonamdgpu-warp-pipeline`
- **CN:** 主要 pass 选项：`-split-input-file`，`-tritonamdgpu-warp-pipeline`
- **EN:** Dominant operations include `tt.func`, `arith.constant`, `rocdl.sched`, `arith.addi`, `arith.muli`, `scf.for`, `scf.yield`, `tt.return`, `tt.store`, `ttg.async_wait`.
- **CN:** 主要操作包括 `tt.func`、`arith.constant`、`rocdl.sched`、`arith.addi`、`arith.muli`、`scf.for`、`scf.yield`、`tt.return`、`tt.store`、`ttg.async_wait`。
- **EN:** The file contains 1 independently testable section(s). Check styles used: CHECK x70, CHECK-NOT x7, CHECK-LABEL x6, CHECK-SAME x2. Important labels include tt.func @three_stage_example(, tt.func @two_stage_example(, tt.func public @triple_buf_two_stages, tt.func @flat_pipeline_example(. It validates both the presence of the desired rewrite and the absence of unwanted residual operations.
- **CN:** 该文件包含 1 个可独立测试的分段。使用的检查类型：CHECK ×70，CHECK-NOT ×7，CHECK-LABEL ×6，CHECK-SAME ×2。 关键标签包括 tt.func @three_stage_example(，tt.func @two_stage_example(，tt.func public @triple_buf_two_stages，tt.func @flat_pipeline_example(。 它同时验证目标重写已经出现，并且不再残留不需要的操作。

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