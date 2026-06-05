# ops.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonNvidiaGPU/ops.mlir`
- **EN:** Test case centered on `ops` behavior in Triton/TritonGPU IR.
- **CN:** 该测试围绕 Triton/TritonGPU IR 中的 `ops` 行为展开。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s | FileCheck %s`
- **CN:** RUN 流水线：`// RUN: triton-opt %s | FileCheck %s`

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
1| // RUN: triton-opt %s | FileCheck %s
2| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s | FileCheck %s` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s | FileCheck %s`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 3-14
```mlir
 3| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = false, elementBitWidth = 8}>
 4| #shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = true, elementBitWidth = 8}>
 5| #shared2 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
 6| #tmem_f16 = #ttng.tensor_memory_encoding<blockM = 128, blockN = 256, colStride = 2>
 7| #tmem_int32 = #ttng.tensor_memory_encoding<blockM = 128, blockN = 256, colStride = 1>
 8| #tmem_scales = #ttng.tensor_memory_scales_encoding<>
 9| 
10| #blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [32, 1], warpsPerCTA = [1, 4], order = [1, 0]}>
11| #offsets = #ttg.slice<{dim = 0, parent = #blocked}>
12| #blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [2, 16], warpsPerCTA = [1, 4], order = [0, 1]}>
13| #scales = #ttg.linear<{register = [[0, 1], [0, 2], [32, 0], [64, 0], [0, 4]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [16, 0]], warp = [[0, 0], [0, 0]], block = []}>
14| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 15-16
```mlir
15| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
16| 
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 17-19
```mlir
17|   // CHECK-LABEL: @tcgen5
18|   //       CHECK:   ttng.tc_gen5_mma
19|   //       CHECK:   ttng.tc_gen5_mma
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 20-37
```mlir
20|   tt.func @tcgen5(%a: !ttg.memdesc<128x128xf8E5M2, #shared, #ttg.shared_memory>,
21|                   %b: !ttg.memdesc<128x256xf8E5M2, #shared1, #ttg.shared_memory>,
22|                   %c: !ttg.memdesc<128x256xf16, #tmem_f16, #ttng.tensor_memory, mutable>,
23|                   %accUse: i1,
24|                   %pred: i1,
25|                   %barrier: !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory, mutable>,
26|                   %barrierPred: i1) {
27|     ttng.tc_gen5_mma %a, %b, %c, %accUse, %pred, %barrier[%barrierPred] {is_async} :
28|        !ttg.memdesc<128x128xf8E5M2, #shared, #ttg.shared_memory>,
29|        !ttg.memdesc<128x256xf8E5M2, #shared1, #ttg.shared_memory>,
30|        !ttg.memdesc<128x256xf16, #tmem_f16, #ttng.tensor_memory, mutable>,
31|        !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory, mutable>
32| 
33|     ttng.tc_gen5_mma %a, %b, %c, %accUse, %pred:
34|        !ttg.memdesc<128x128xf8E5M2, #shared, #ttg.shared_memory>,
35|        !ttg.memdesc<128x256xf8E5M2, #shared1, #ttg.shared_memory>,
36|        !ttg.memdesc<128x256xf16, #tmem_f16, #ttng.tensor_memory, mutable>
37|     tt.return
```
**EN:** This function-oriented block defines or enters `tcgen5`. Within it, the test exercises tt.func, Gen5 tensor-core MMA ops, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tcgen5` 为核心。测试在其中演示 tt.func、Gen5 张量核 MMA 操作、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 38-39
```mlir
38|   }
39| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 40-42
```mlir
40|   // CHECK-LABEL: @tcgen5_int8
41|   //       CHECK:   ttng.tc_gen5_mma {{.*}} {is_async, is_unsigned}
42|   //       CHECK:   ttng.tc_gen5_mma {{.*}} {is_unsigned}
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 43-60
```mlir
43|   tt.func @tcgen5_int8(
44|                   %a: !ttg.memdesc<128x128xi8, #shared, #ttg.shared_memory>,
45|                   %b: !ttg.memdesc<128x256xi8, #shared1, #ttg.shared_memory>,
46|                   %c: !ttg.memdesc<128x256xi32, #tmem_int32, #ttng.tensor_memory, mutable>,
47|                   %accUse: i1,
48|                   %pred: i1,
49|                   %barrier: !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory, mutable>,
50|                   %barrierPred: i1) {
51|     ttng.tc_gen5_mma %a, %b, %c, %accUse, %pred, %barrier[%barrierPred] {is_async, is_unsigned} :
52|        !ttg.memdesc<128x128xi8, #shared, #ttg.shared_memory>,
53|        !ttg.memdesc<128x256xi8, #shared1, #ttg.shared_memory>,
54|        !ttg.memdesc<128x256xi32, #tmem_int32, #ttng.tensor_memory, mutable>,
55|        !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory, mutable>
56| 
57|     ttng.tc_gen5_mma %a, %b, %c, %accUse, %pred {is_unsigned}:
58|        !ttg.memdesc<128x128xi8, #shared, #ttg.shared_memory>,
59|        !ttg.memdesc<128x256xi8, #shared1, #ttg.shared_memory>,
60|        !ttg.memdesc<128x256xi32, #tmem_int32, #ttng.tensor_memory, mutable>
```
**EN:** This function-oriented block defines or enters `tcgen5_int8`. Within it, the test exercises tt.func, Gen5 tensor-core MMA ops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tcgen5_int8` 为核心。测试在其中演示 tt.func、Gen5 张量核 MMA 操作，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 61-63
```mlir
61|     tt.return
62|   }
63| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 64-70
```mlir
64|   // CHECK-LABEL: @async_tma_gather
65|   // CHECK-SAME: [[DESC:%arg[0-9]+]]:
66|   // CHECK-SAME: [[X_OFFSETS:%arg[0-9]+]]:
67|   // CHECK-SAME: [[Y_OFFSET:%arg[0-9]+]]:
68|   // CHECK-SAME: [[BAR:%arg[0-9]+]]:
69|   // CHECK-SAME: [[RESULT:%arg[0-9]+]]:
70|   // CHECK-SAME: [[PRED:%arg[0-9]+]]:
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @async_tma_gather anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @async_tma_gather 这样的标签用于锚定匹配范围。

### Lines 71-74
```mlir
71|   tt.func @async_tma_gather(%desc: !tt.tensordesc<1x128xbf16, #shared>, %x_offsets: tensor<32xi32, #offsets>, %y_offset: i32,
72|                             %bar: !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory, mutable>,
73|                             %result: !ttg.memdesc<32x128xbf16, #shared, #ttg.shared_memory, mutable>,
74|                             %pred: i1) {
```
**EN:** This function-oriented block defines or enters `async_tma_gather`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_tma_gather` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 75-79
```mlir
75|     // CHECK-NEXT: ttng.async_tma_gather [[DESC]][[[X_OFFSETS]], [[Y_OFFSET]]] [[RESULT]], [[BAR]], [[PRED]] : !tt.tensordesc<1x128xbf16, #shared>, tensor<32xi32, #ttg.slice<{dim = 0, parent = #blocked}>>, i32, !ttg.memdesc<1xi64, #shared2, #smem, mutable>, !ttg.memdesc<32x128xbf16, #shared, #smem, mutable>, i1
76|     ttng.async_tma_gather %desc[%x_offsets, %y_offset] %result, %bar, %pred : !tt.tensordesc<1x128xbf16, #shared>, tensor<32xi32, #offsets>, i32, !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory, mutable>, !ttg.memdesc<32x128xbf16, #shared, #ttg.shared_memory, mutable>, i1
77|     tt.return
78|   }
79| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.async_tma_gather, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.async_tma_gather、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 80-84
```mlir
80|   // CHECK-LABEL: @async_tma_scatter
81|   // CHECK-SAME: [[DESC:%arg[0-9]+]]:
82|   // CHECK-SAME: [[X_OFFSETS:%arg[0-9]+]]:
83|   // CHECK-SAME: [[Y_OFFSET:%arg[0-9]+]]:
84|   // CHECK-SAME: [[SRC:%arg[0-9]+]]:
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @async_tma_scatter anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @async_tma_scatter 这样的标签用于锚定匹配范围。

### Lines 85-86
```mlir
85|   tt.func @async_tma_scatter(%desc: !tt.tensordesc<1x128xbf16, #shared>, %x_offsets: tensor<32xi32, #offsets>, %y_offset: i32,
86|                              %src: !ttg.memdesc<32x128xbf16, #shared, #ttg.shared_memory, mutable>) {
```
**EN:** This function-oriented block defines or enters `async_tma_scatter`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_tma_scatter` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 87-91
```mlir
87|     // CHECK-NEXT: ttng.async_tma_scatter [[DESC]][[[X_OFFSETS]], [[Y_OFFSET]]] [[SRC]] : !tt.tensordesc<1x128xbf16, #shared>, tensor<32xi32, #ttg.slice<{dim = 0, parent = #blocked}>>, i32, !ttg.memdesc<32x128xbf16, #shared, #smem, mutable>
88|     ttng.async_tma_scatter %desc[%x_offsets, %y_offset] %src : !tt.tensordesc<1x128xbf16, #shared>, tensor<32xi32, #offsets>, i32, !ttg.memdesc<32x128xbf16, #shared, #ttg.shared_memory, mutable>
89|     tt.return
90|   }
91| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.async_tma_scatter, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.async_tma_scatter、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 92-94
```mlir
92|   // CHECK-LABEL: @wait_barrier
93|   // CHECK-SAME: [[ALLOC:%arg[0-9]+]]:
94|   // CHECK-SAME: [[PHASE:%arg[0-9]+]]:
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @wait_barrier anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @wait_barrier 这样的标签用于锚定匹配范围。

### Lines 95-95
```mlir
95|   tt.func @wait_barrier(%alloc: !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory, mutable>, %phase: i32) {
```
**EN:** This function-oriented block defines or enters `wait_barrier`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `wait_barrier` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 96-100
```mlir
 96|     // CHECK-NEXT: ttng.wait_barrier [[ALLOC]], [[PHASE]] : !ttg.memdesc<1xi64, #shared2, #smem, mutable>
 97|     ttng.wait_barrier %alloc, %phase : !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory, mutable>
 98|     tt.return
 99|   }
100| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.wait_barrier, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.wait_barrier、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 101-105
```mlir
101|   // CHECK-LABEL: @wait_barrier
102|   // CHECK-SAME: [[ALLOC:%arg[0-9]+]]:
103|   // CHECK-SAME: [[PHASE:%arg[0-9]+]]:
104|   // CHECK-SAME: [[DEP1:%arg[0-9]+]]:
105|   // CHECK-SAME: [[DEP2:%arg[0-9]+]]:
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @wait_barrier anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @wait_barrier 这样的标签用于锚定匹配范围。

### Lines 106-106
```mlir
106|   tt.func @wait_barrier_deps(%alloc: !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory, mutable>, %phase: i32, %dep1: !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory, mutable>, %dep2: !ttg.memdesc<128x128xf8E5M2, #shared, #ttg.shared_memory, mutable>) {
```
**EN:** This function-oriented block defines or enters `wait_barrier_deps`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `wait_barrier_deps` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 107-111
```mlir
107|     // CHECK-NEXT: ttng.wait_barrier [[ALLOC]], [[PHASE]] deps [[DEP1]], [[DEP2]] : !ttg.memdesc<1xi64, #shared2, #smem, mutable>, !ttg.memdesc<1xi64, #shared2, #smem, mutable>, !ttg.memdesc<128x128xf8E5M2, #shared, #smem, mutable>
108|     ttng.wait_barrier %alloc, %phase deps %dep1, %dep2 : !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory, mutable>, !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory, mutable>, !ttg.memdesc<128x128xf8E5M2, #shared, #ttg.shared_memory, mutable>
109|     tt.return
110|   }
111| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.wait_barrier, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.wait_barrier、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 112-112
```mlir
112|   // CHECK-LABEL: @arrive_barrier
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @arrive_barrier anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @arrive_barrier 这样的标签用于锚定匹配范围。

### Lines 113-113
```mlir
113|   tt.func @arrive_barrier(%alloc: !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory, mutable>, %pred: i1) {
```
**EN:** This function-oriented block defines or enters `arrive_barrier`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `arrive_barrier` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 114-115
```mlir
114|     // CHECK-NEXT: ttng.arrive_barrier %arg0, 2 : !ttg.memdesc<1xi64, #shared2, #smem, mutable>
115|     ttng.arrive_barrier %alloc, 2 : !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.arrive_barrier. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.arrive_barrier。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 116-120
```mlir
116|     // CHECK-NEXT: ttng.arrive_barrier %arg0, 2, %arg1 : !ttg.memdesc<1xi64, #shared2, #smem, mutable>
117|     ttng.arrive_barrier %alloc, 2, %pred : !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory, mutable>
118|     tt.return
119|   }
120| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.arrive_barrier, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.arrive_barrier、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 121-128
```mlir
121|   tt.func @scale_encoding(%arg0: tensor<128x8xi8, #scales>, %arg1: tensor<128x8xf8E5M2, #scales>) {
122|     %0 = ttng.tmem_alloc %arg0 : (tensor<128x8xi8, #scales>) -> !ttg.memdesc<128x8xi8, #tmem_scales, #ttng.tensor_memory>
123|     %1 = ttng.tmem_alloc %arg1 : (tensor<128x8xf8E5M2, #scales>) -> !ttg.memdesc<128x8xf8E5M2, #tmem_scales, #ttng.tensor_memory>
124|     tt.return
125|   }
126| }
127| 
128| // Tests for TMA im2col (3D/4D/5D) and tiled mode
```
**EN:** This function-oriented block defines or enters `scale_encoding`. Within it, the test exercises tt.func, tensor-memory allocation, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `scale_encoding` 为核心。测试在其中演示 tt.func、张量内存分配、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 129-131
```mlir
129| #nvmma_128 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
130| #shared3 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
131| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 132-132
```mlir
132| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 133-134
```mlir
133|   // CHECK-LABEL: @tma_load_im2col_3d
134|   // CHECK: ttng.async_tma_copy_global_to_local {{.*}} offsets = [{{.*}}] {{.*}} : !ttng.tensordesc_im2col
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tma_load_im2col_3d anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tma_load_im2col_3d 这样的标签用于锚定匹配范围。

### Lines 135-145
```mlir
135|   tt.func public @tma_load_im2col_3d(%desc: !ttng.tensordesc_im2col<64x128xf16, #nvmma_128>) {
136|     %true = arith.constant true
137|     %c0 = arith.constant 0 : i32
138|     %off = arith.constant 1 : i16
139|     %buf = ttg.local_alloc : () -> !ttg.memdesc<64x128xf16, #nvmma_128, #smem, mutable>
140|     %bar = ttg.local_alloc : () -> !ttg.memdesc<1xi64, #shared3, #smem, mutable>
141|     ttng.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared3, #smem, mutable>
142|     ttng.async_tma_copy_global_to_local %desc[%c0, %c0, %c0] offsets = [%off] %buf, %bar, %true : !ttng.tensordesc_im2col<64x128xf16, #nvmma_128>, !ttg.memdesc<1xi64, #shared3, #smem, mutable> -> !ttg.memdesc<64x128xf16, #nvmma_128, #smem, mutable>
143|     tt.return
144|   }
145| 
```
**EN:** This function-oriented block defines or enters `tma_load_im2col_3d`. Within it, the test exercises constants, tt.func, shared/local memory allocation, ttng.init_barrier, ttng.async_tma_copy_global_to_local, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tma_load_im2col_3d` 为核心。测试在其中演示 常量、tt.func、共享/本地内存分配、ttng.init_barrier、ttng.async_tma_copy_global_to_local，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 146-147
```mlir
146|   // CHECK-LABEL: @tma_load_im2col_4d
147|   // CHECK: ttng.async_tma_copy_global_to_local {{.*}} offsets = [{{.*}}, {{.*}}] {{.*}} : !ttng.tensordesc_im2col
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tma_load_im2col_4d anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tma_load_im2col_4d 这样的标签用于锚定匹配范围。

### Lines 148-159
```mlir
148|   tt.func public @tma_load_im2col_4d(%desc: !ttng.tensordesc_im2col<64x128xf16, #nvmma_128>) {
149|     %true = arith.constant true
150|     %c0 = arith.constant 0 : i32
151|     %off1 = arith.constant 1 : i16
152|     %off2 = arith.constant 2 : i16
153|     %buf = ttg.local_alloc : () -> !ttg.memdesc<64x128xf16, #nvmma_128, #smem, mutable>
154|     %bar = ttg.local_alloc : () -> !ttg.memdesc<1xi64, #shared3, #smem, mutable>
155|     ttng.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared3, #smem, mutable>
156|     ttng.async_tma_copy_global_to_local %desc[%c0, %c0, %c0, %c0] offsets = [%off1, %off2] %buf, %bar, %true : !ttng.tensordesc_im2col<64x128xf16, #nvmma_128>, !ttg.memdesc<1xi64, #shared3, #smem, mutable> -> !ttg.memdesc<64x128xf16, #nvmma_128, #smem, mutable>
157|     tt.return
158|   }
159| 
```
**EN:** This function-oriented block defines or enters `tma_load_im2col_4d`. Within it, the test exercises constants, tt.func, shared/local memory allocation, ttng.init_barrier, ttng.async_tma_copy_global_to_local, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tma_load_im2col_4d` 为核心。测试在其中演示 常量、tt.func、共享/本地内存分配、ttng.init_barrier、ttng.async_tma_copy_global_to_local，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 160-161
```mlir
160|   // CHECK-LABEL: @tma_load_im2col_5d
161|   // CHECK: ttng.async_tma_copy_global_to_local {{.*}} offsets = [{{.*}}, {{.*}}, {{.*}}] {{.*}} : !ttng.tensordesc_im2col
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tma_load_im2col_5d anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tma_load_im2col_5d 这样的标签用于锚定匹配范围。

### Lines 162-174
```mlir
162|   tt.func public @tma_load_im2col_5d(%desc: !ttng.tensordesc_im2col<64x128xf16, #nvmma_128>) {
163|     %true = arith.constant true
164|     %c0 = arith.constant 0 : i32
165|     %off1 = arith.constant 1 : i16
166|     %off2 = arith.constant 2 : i16
167|     %off3 = arith.constant 3 : i16
168|     %buf = ttg.local_alloc : () -> !ttg.memdesc<64x128xf16, #nvmma_128, #smem, mutable>
169|     %bar = ttg.local_alloc : () -> !ttg.memdesc<1xi64, #shared3, #smem, mutable>
170|     ttng.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared3, #smem, mutable>
171|     ttng.async_tma_copy_global_to_local %desc[%c0, %c0, %c0, %c0, %c0] offsets = [%off1, %off2, %off3] %buf, %bar, %true : !ttng.tensordesc_im2col<64x128xf16, #nvmma_128>, !ttg.memdesc<1xi64, #shared3, #smem, mutable> -> !ttg.memdesc<64x128xf16, #nvmma_128, #smem, mutable>
172|     tt.return
173|   }
174| 
```
**EN:** This function-oriented block defines or enters `tma_load_im2col_5d`. Within it, the test exercises constants, tt.func, shared/local memory allocation, ttng.init_barrier, ttng.async_tma_copy_global_to_local, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tma_load_im2col_5d` 为核心。测试在其中演示 常量、tt.func、共享/本地内存分配、ttng.init_barrier、ttng.async_tma_copy_global_to_local，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 175-177
```mlir
175|   // CHECK-LABEL: @tma_load_tiled_mode
176|   // CHECK: ttng.async_tma_copy_global_to_local {{.*}}[{{.*}}, {{.*}}] %{{.*}}, %{{.*}}, {{.*}} : !tt.tensordesc
177|   // CHECK-NOT: offsets
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tma_load_tiled_mode anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tma_load_tiled_mode 这样的标签用于锚定匹配范围。

### Lines 178-187
```mlir
178|   tt.func public @tma_load_tiled_mode(%desc: !tt.tensordesc<64x128xf16, #nvmma_128>) {
179|     %true = arith.constant true
180|     %c0 = arith.constant 0 : i32
181|     %buf = ttg.local_alloc : () -> !ttg.memdesc<64x128xf16, #nvmma_128, #smem, mutable>
182|     %bar = ttg.local_alloc : () -> !ttg.memdesc<1xi64, #shared3, #smem, mutable>
183|     ttng.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared3, #smem, mutable>
184|     ttng.async_tma_copy_global_to_local %desc[%c0, %c0] %buf, %bar, %true : !tt.tensordesc<64x128xf16, #nvmma_128>, !ttg.memdesc<1xi64, #shared3, #smem, mutable> -> !ttg.memdesc<64x128xf16, #nvmma_128, #smem, mutable>
185|     tt.return
186|   }
187| 
```
**EN:** This function-oriented block defines or enters `tma_load_tiled_mode`. Within it, the test exercises tt.func, constants, shared/local memory allocation, ttng.init_barrier, ttng.async_tma_copy_global_to_local, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tma_load_tiled_mode` 为核心。测试在其中演示 tt.func、常量、共享/本地内存分配、ttng.init_barrier、ttng.async_tma_copy_global_to_local，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 188-189
```mlir
188|   // CHECK-LABEL: @tensordesc_im2col
189|   // CHECK-SAME: !ttng.tensordesc_im2col<64x128xf16, {{.*}}>
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tensordesc_im2col anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tensordesc_im2col 这样的标签用于锚定匹配范围。

### Lines 190-190
```mlir
190|   tt.func public @tensordesc_im2col(%desc: !ttng.tensordesc_im2col<64x128xf16, #nvmma_128>) {
```
**EN:** This function-oriented block defines or enters `tensordesc_im2col`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tensordesc_im2col` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 191-194
```mlir
191|     // CHECK: tt.return
192|     tt.return
193|   }
194| }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

## Key Concepts / 关键概念

- **EN:** No explicit RUN pipeline was found; the file focuses on IR semantics, parsing, or verification behavior.
- **CN:** 未发现显式 RUN 流水线；该文件重点关注 IR 语义、解析或验证行为。
- **EN:** Dominant operations include `tt.func`, `arith.constant`, `tt.return`, `ttg.local_alloc`, `ttng.tc_gen5_mma`, `ttng.init_barrier`, `ttng.async_tma_copy_global_to_local`, `module`, `ttng.wait_barrier`, `ttng.arrive_barrier`.
- **CN:** 主要操作包括 `tt.func`、`arith.constant`、`tt.return`、`ttg.local_alloc`、`ttng.tc_gen5_mma`、`ttng.init_barrier`、`ttng.async_tma_copy_global_to_local`、`module`、`ttng.wait_barrier`、`ttng.arrive_barrier`。
- **EN:** The file contains 1 independently testable section(s). Check styles used: CHECK-SAME x17, CHECK-LABEL x12, CHECK x9, CHECK-NEXT x6. Important labels include @tcgen5, @tcgen5_int8, @async_tma_gather, @async_tma_scatter. It validates both the presence of the desired rewrite and the absence of unwanted residual operations.
- **CN:** 该文件包含 1 个可独立测试的分段。使用的检查类型：CHECK-SAME ×17，CHECK-LABEL ×12，CHECK ×9，CHECK-NEXT ×6。 关键标签包括 @tcgen5，@tcgen5_int8，@async_tma_gather，@async_tma_scatter。 它同时验证目标重写已经出现，并且不再残留不需要的操作。

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