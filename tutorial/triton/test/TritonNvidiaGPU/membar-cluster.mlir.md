# membar-cluster.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonNvidiaGPU/membar-cluster.mlir`
- **EN:** Test for the MLIR/Triton pipeline `--allocate-shared-memory, -test-print-membar` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `--allocate-shared-memory, -test-print-membar` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -split-input-file --allocate-shared-memory -test-print-membar | FileCheck --dump-input=fail --dump-input-context=30 %s`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -split-input-file --allocate-shared-memory -test-print-membar | FileCheck --dump-input=fail --dump-input-context=30 %s`

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
1| // RUN: triton-opt %s -split-input-file --allocate-shared-memory -test-print-membar | FileCheck --dump-input=fail --dump-input-context=30 %s
2| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -split-input-file --allocate-shared-memory -test-print-membar | FileCheck --dump-input=fail --dump-input-context=30 %s` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -split-input-file --allocate-shared-memory -test-print-membar | FileCheck --dump-input=fail --dump-input-context=30 %s`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 3-3
```mlir
3| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 4-9
```mlir
4| 
5| #blockedSplitM = #ttg.blocked<{sizePerThread = [1, 32], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1], CGALayout = [[1, 0]]}>
6| #blockedSplitN = #ttg.blocked<{sizePerThread = [1, 32], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1], CGALayout = [[0, 1]]}>
7| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0], CGALayout = [[1, 0]]}>
8| #smem = #ttg.shared_memory
9| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 10-10
```mlir
10| module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 11-14
```mlir
11|   // CHECK-LABEL: @convert_layout_cluster_barrier
12|   // CHECK: ttg.convert_layout
13|   // CHECK-NEXT: ttng.cluster_barrier
14|   // CHECK-NEXT: ttg.local_alloc
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @convert_layout_cluster_barrier anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @convert_layout_cluster_barrier 这样的标签用于锚定匹配范围。

### Lines 15-24
```mlir
15|   tt.func @convert_layout_cluster_barrier() -> tensor<256x128xf16, #blockedSplitM> {
16|     %cst = arith.constant dense<0.000000e+00> : tensor<256x128xf16, #blockedSplitM>
17|     %cvt = ttg.convert_layout %cst : tensor<256x128xf16, #blockedSplitM> -> tensor<256x128xf16, #blockedSplitN>
18|     %buf = ttg.local_alloc %cvt : (tensor<256x128xf16, #blockedSplitN>) -> !ttg.memdesc<256x128xf16, #shared, #smem, mutable>
19|     ttg.local_store %cvt, %buf : tensor<256x128xf16, #blockedSplitN> -> !ttg.memdesc<256x128xf16, #shared, #smem, mutable>
20|     %ld = ttg.local_load %buf : !ttg.memdesc<256x128xf16, #shared, #smem, mutable> -> tensor<256x128xf16, #blockedSplitM>
21|     tt.return %ld : tensor<256x128xf16, #blockedSplitM>
22|   }
23| }
24| 
```
**EN:** This function-oriented block defines or enters `convert_layout_cluster_barrier`. Within it, the test exercises tt.func, constants, layout conversions, shared/local memory allocation, local/shared memory stores, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `convert_layout_cluster_barrier` 为核心。测试在其中演示 tt.func、常量、布局转换、共享/本地内存分配、本地/共享内存存储，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 25-25
```mlir
25| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 26-29
```mlir
26| 
27| #barrierEncPartial = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0], CGALayout = [[0]]}>
28| #smem = #ttg.shared_memory
29| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 30-30
```mlir
30| module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 31-36
```mlir
31|   // CHECK-LABEL: @insert_fence_and_relaxed_cluster_barrier_at_window_end_after_existing_fence
32|   // CHECK: ttng.init_barrier
33|   // CHECK-NEXT: ttng.fence_mbarrier_init_release_cluster
34|   // CHECK-NEXT: ttng.fence_mbarrier_init_release_cluster
35|   // CHECK-NEXT: ttng.cluster_barrier {relaxed = true}
36|   // CHECK-NEXT: ttng.wait_barrier
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @insert_fence_and_relaxed_cluster_barrier_at_window_end_after_existing_fence anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @insert_fence_and_relaxed_cluster_barrier_at_window_end_after_existing_fence 这样的标签用于锚定匹配范围。

### Lines 37-46
```mlir
37|   tt.func @insert_fence_and_relaxed_cluster_barrier_at_window_end_after_existing_fence() {
38|     %c0 = arith.constant 0 : i32
39|     %barrier = ttg.local_alloc : () -> !ttg.memdesc<1xi64, #barrierEncPartial, #smem, mutable>
40|     ttng.init_barrier %barrier, 1 : !ttg.memdesc<1xi64, #barrierEncPartial, #smem, mutable>
41|     ttng.fence_mbarrier_init_release_cluster
42|     ttng.wait_barrier %barrier, %c0 : !ttg.memdesc<1xi64, #barrierEncPartial, #smem, mutable>
43|     tt.return
44|   }
45| }
46| 
```
**EN:** This function-oriented block defines or enters `insert_fence_and_relaxed_cluster_barrier_at_window_end_after_existing_fence`. Within it, the test exercises tt.func, constants, shared/local memory allocation, ttng.init_barrier, ttng.fence_mbarrier_init_release_cluster, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `insert_fence_and_relaxed_cluster_barrier_at_window_end_after_existing_fence` 为核心。测试在其中演示 tt.func、常量、共享/本地内存分配、ttng.init_barrier、ttng.fence_mbarrier_init_release_cluster，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 47-47
```mlir
47| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 48-51
```mlir
48| 
49| #barrierEncPartial = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0], CGALayout = [[0]]}>
50| #smem = #ttg.shared_memory
51| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 52-54
```mlir
52| module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
53|   // Reuse an existing relaxed cluster barrier, but keep the fence immediately
54|   // in front of it.
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 55-59
```mlir
55|   // CHECK-LABEL: @insert_fence_right_before_existing_relaxed_cluster_barrier
56|   // CHECK: ttng.init_barrier
57|   // CHECK-NEXT: ttng.fence_mbarrier_init_release_cluster
58|   // CHECK-NEXT: ttng.cluster_barrier {relaxed = true}
59|   // CHECK-NEXT: ttng.wait_barrier
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @insert_fence_right_before_existing_relaxed_cluster_barrier anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @insert_fence_right_before_existing_relaxed_cluster_barrier 这样的标签用于锚定匹配范围。

### Lines 60-69
```mlir
60|   tt.func @insert_fence_right_before_existing_relaxed_cluster_barrier() {
61|     %c0 = arith.constant 0 : i32
62|     %barrier = ttg.local_alloc : () -> !ttg.memdesc<1xi64, #barrierEncPartial, #smem, mutable>
63|     ttng.init_barrier %barrier, 1 : !ttg.memdesc<1xi64, #barrierEncPartial, #smem, mutable>
64|     ttng.cluster_barrier {relaxed = true}
65|     ttng.wait_barrier %barrier, %c0 : !ttg.memdesc<1xi64, #barrierEncPartial, #smem, mutable>
66|     tt.return
67|   }
68| }
69| 
```
**EN:** This function-oriented block defines or enters `insert_fence_right_before_existing_relaxed_cluster_barrier`. Within it, the test exercises tt.func, constants, shared/local memory allocation, ttng.init_barrier, ttng.cluster_barrier, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `insert_fence_right_before_existing_relaxed_cluster_barrier` 为核心。测试在其中演示 tt.func、常量、共享/本地内存分配、ttng.init_barrier、ttng.cluster_barrier，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 70-70
```mlir
70| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 71-74
```mlir
71| 
72| #barrierEncWS = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0], CGALayout = [[0]]}>
73| #smem = #ttg.shared_memory
74| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 75-75
```mlir
75| module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 76-82
```mlir
76|   // CHECK-LABEL: @insert_fence_and_relaxed_cluster_barrier_before_warp_specialize
77|   // CHECK: ttng.init_barrier
78|   // CHECK: ttng.init_barrier
79|   // CHECK-NEXT: ttng.fence_mbarrier_init_release_cluster
80|   // CHECK-NEXT: ttng.cluster_barrier {relaxed = true}
81|   // CHECK-NEXT: ttg.warp_specialize
82|   // CHECK: ttng.wait_barrier
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @insert_fence_and_relaxed_cluster_barrier_before_warp_specialize anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @insert_fence_and_relaxed_cluster_barrier_before_warp_specialize 这样的标签用于锚定匹配范围。

### Lines 83-100
```mlir
 83|   tt.func @insert_fence_and_relaxed_cluster_barrier_before_warp_specialize(%idx: i32) {
 84|     %c0 = arith.constant 0 : i32
 85|     %c1 = arith.constant 1 : i32
 86|     %barriers = ttg.local_alloc : () -> !ttg.memdesc<2x1xi64, #barrierEncWS, #smem, mutable>
 87|     %barrier0 = ttg.memdesc_index %barriers[%c0] : !ttg.memdesc<2x1xi64, #barrierEncWS, #smem, mutable> -> !ttg.memdesc<1xi64, #barrierEncWS, #smem, mutable>
 88|     ttng.init_barrier %barrier0, 1 : !ttg.memdesc<1xi64, #barrierEncWS, #smem, mutable>
 89|     %barrier1 = ttg.memdesc_index %barriers[%c1] : !ttg.memdesc<2x1xi64, #barrierEncWS, #smem, mutable> -> !ttg.memdesc<1xi64, #barrierEncWS, #smem, mutable>
 90|     ttng.init_barrier %barrier1, 1 : !ttg.memdesc<1xi64, #barrierEncWS, #smem, mutable>
 91|     ttg.warp_specialize()
 92|     default {
 93|       %barrier = ttg.memdesc_index %barriers[%idx] : !ttg.memdesc<2x1xi64, #barrierEncWS, #smem, mutable> -> !ttg.memdesc<1xi64, #barrierEncWS, #smem, mutable>
 94|       ttng.wait_barrier %barrier, %c0 : !ttg.memdesc<1xi64, #barrierEncWS, #smem, mutable>
 95|       ttg.warp_yield
 96|     }
 97|     partition0() num_warps(1) {
 98|       ttg.warp_return
 99|     } : () -> ()
100|     tt.return
```
**EN:** This function-oriented block defines or enters `insert_fence_and_relaxed_cluster_barrier_before_warp_specialize`. Within it, the test exercises ttg.memdesc_index, tt.func, constants, ttng.init_barrier, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `insert_fence_and_relaxed_cluster_barrier_before_warp_specialize` 为核心。测试在其中演示 ttg.memdesc_index、tt.func、常量、ttng.init_barrier、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 101-103
```mlir
101|   }
102| }
103| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 104-104
```mlir
104| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 105-108
```mlir
105| 
106| #barrierEncCF = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0], CGALayout = [[0]]}>
107| #smem = #ttg.shared_memory
108| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 109-111
```mlir
109| module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
110|   // If branch-local init ops reconverge in CF, insert the sync at the join
111|   // block before the first tracked use.
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 112-118
```mlir
112|   // CHECK-LABEL: @insert_fence_and_relaxed_cluster_barrier_at_cf_join_after_branch_inits
113|   // CHECK: ttng.init_barrier
114|   // CHECK: ttng.init_barrier
115|   // CHECK: ^bb3:
116|   // CHECK-NEXT: ttng.fence_mbarrier_init_release_cluster
117|   // CHECK-NEXT: ttng.cluster_barrier {relaxed = true}
118|   // CHECK-NEXT: ttng.wait_barrier
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @insert_fence_and_relaxed_cluster_barrier_at_cf_join_after_branch_inits anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @insert_fence_and_relaxed_cluster_barrier_at_cf_join_after_branch_inits 这样的标签用于锚定匹配范围。

### Lines 119-135
```mlir
119|   tt.func @insert_fence_and_relaxed_cluster_barrier_at_cf_join_after_branch_inits(%pred: i1) {
120|     %c0 = arith.constant 0 : i32
121|     %barrier = ttg.local_alloc : () -> !ttg.memdesc<1xi64, #barrierEncCF, #smem, mutable>
122|     cf.cond_br %pred, ^bb1, ^bb2
123|   ^bb1:
124|     ttng.init_barrier %barrier, 1 : !ttg.memdesc<1xi64, #barrierEncCF, #smem, mutable>
125|     cf.br ^bb3
126|   ^bb2:
127|     ttng.init_barrier %barrier, 1 : !ttg.memdesc<1xi64, #barrierEncCF, #smem, mutable>
128|     cf.br ^bb3
129|   ^bb3:
130|     ttng.wait_barrier %barrier, %c0 : !ttg.memdesc<1xi64, #barrierEncCF, #smem, mutable>
131|     ttg.local_dealloc %barrier : !ttg.memdesc<1xi64, #barrierEncCF, #smem, mutable>
132|     tt.return
133|   }
134| }
135| 
```
**EN:** This function-oriented block defines or enters `insert_fence_and_relaxed_cluster_barrier_at_cf_join_after_branch_inits`. Within it, the test exercises tt.func, ttng.init_barrier, cf.br, constants, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `insert_fence_and_relaxed_cluster_barrier_at_cf_join_after_branch_inits` 为核心。测试在其中演示 tt.func、ttng.init_barrier、cf.br、常量、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 136-136
```mlir
136| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 137-140
```mlir
137| 
138| #blocked = #ttg.blocked<{sizePerThread = [1, 32], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [0, 1], CGALayout = [[0, 1]]}>
139| #slice1 = #ttg.slice<{dim = 1, parent = #blocked}>
140| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 141-142
```mlir
141| module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
142|   // If there is a cross-CTA read dependency at kernel exit, we must end with a cluster barrier.
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 143-146
```mlir
143|   // CHECK-LABEL: @end_cluster_barrier_after_cross_reduce
144|   // CHECK: "tt.reduce"{{.*}}axis = 1
145|   // CHECK: ttng.cluster_barrier
146|   // CHECK-NEXT: tt.return
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @end_cluster_barrier_after_cross_reduce anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @end_cluster_barrier_after_cross_reduce 这样的标签用于锚定匹配范围。

### Lines 147-156
```mlir
147|   tt.func @end_cluster_barrier_after_cross_reduce(%arg0: tensor<256x128xf16, #blocked>) -> tensor<256xf16, #slice1> {
148|     %red = "tt.reduce"(%arg0) ({
149|     ^bb0(%lhs: f16, %rhs: f16):
150|       %add = arith.addf %lhs, %rhs : f16
151|       tt.reduce.return %add : f16
152|     }) {axis = 1 : i32} : (tensor<256x128xf16, #blocked>) -> tensor<256xf16, #slice1>
153|     tt.return %red : tensor<256xf16, #slice1>
154|   }
155| }
156| 
```
**EN:** This function-oriented block defines or enters `end_cluster_barrier_after_cross_reduce`. Within it, the test exercises tt.func, floating-point additions, reductions, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `end_cluster_barrier_after_cross_reduce` 为核心。测试在其中演示 tt.func、浮点加法、归约、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 157-157
```mlir
157| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 158-163
```mlir
158| 
159| #sharedA = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = false, elementBitWidth = 16, CGALayout = [[1, 0]]}>
160| #sharedB = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = false, elementBitWidth = 16, CGALayout = [[0, 1]]}>
161| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1, CGALayout = [[1, 0]], twoCTAs = true>
162| #smem = #ttg.shared_memory
163| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 164-166
```mlir
164| module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 8 : i32, "ttng.two-ctas" = true, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
165|   // Negative test: in 2CTA kernels with non-zero tensor memory size, TMEM
166|   // teardown sync at kernel exit means we should not add an extra cluster barrier.
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 167-171
```mlir
167|   // CHECK-LABEL: @no_end_cluster_barrier_for_mma_with_tmem_teardown
168|   // CHECK: ttng.tmem_alloc
169|   // CHECK: ttng.tc_gen5_mma
170|   // CHECK-NOT: ttng.cluster_barrier
171|   // CHECK: tt.return
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @no_end_cluster_barrier_for_mma_with_tmem_teardown anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @no_end_cluster_barrier_for_mma_with_tmem_teardown 这样的标签用于锚定匹配范围。

### Lines 172-184
```mlir
172|   tt.func @no_end_cluster_barrier_for_mma_with_tmem_teardown() {
173|     %true = arith.constant true
174|     %a = ttg.local_alloc : () -> !ttg.memdesc<256x32xf16, #sharedA, #smem, mutable>
175|     %b = ttg.local_alloc : () -> !ttg.memdesc<32x128xf16, #sharedB, #smem, mutable>
176|     %acc = ttng.tmem_alloc : () -> !ttg.memdesc<256x128xf32, #tmem, #ttng.tensor_memory, mutable>
177|     ttng.tc_gen5_mma %a, %b, %acc, %true, %true {two_ctas} :
178|        !ttg.memdesc<256x32xf16, #sharedA, #smem, mutable>,
179|        !ttg.memdesc<32x128xf16, #sharedB, #smem, mutable>,
180|        !ttg.memdesc<256x128xf32, #tmem, #ttng.tensor_memory, mutable>
181|     tt.return
182|   }
183| }
184| 
```
**EN:** This function-oriented block defines or enters `no_end_cluster_barrier_for_mma_with_tmem_teardown`. Within it, the test exercises tt.func, shared/local memory allocation, constants, tensor-memory allocation, Gen5 tensor-core MMA ops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `no_end_cluster_barrier_for_mma_with_tmem_teardown` 为核心。测试在其中演示 tt.func、共享/本地内存分配、常量、张量内存分配、Gen5 张量核 MMA 操作，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 185-185
```mlir
185| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 186-189
```mlir
186| 
187| #blocked = #ttg.blocked<{sizePerThread = [1, 32], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [0, 1], CGALayout = [[0, 1]]}>
188| #slice0 = #ttg.slice<{dim = 0, parent = #blocked}>
189| #slice1 = #ttg.slice<{dim = 1, parent = #blocked}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 190-193
```mlir
190| module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
191|   // First reduction does not cross CTAs; second one does.
192|   // There should be no cluster barrier between them as tt.reduce always syncs internally before touching distributed shared memory.
193|   // but there should be a cluster barrier after the reduction that touches distributed shared memory
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 194-200
```mlir
194|   // CHECK-LABEL: @reduce_nocross_then_cross
195|   // CHECK: "tt.reduce"{{.*}}axis = 0
196|   // CHECK-NOT: ttng.cluster_barrier
197|   // CHECK: ttg.barrier local
198|   // CHECK: "tt.reduce"{{.*}}axis = 1
199|   // CHECK: ttng.cluster_barrier
200|   // CHECK: "tt.reduce"{{.*}}axis = 0
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @reduce_nocross_then_cross anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @reduce_nocross_then_cross 这样的标签用于锚定匹配范围。

### Lines 201-218
```mlir
201|   tt.func @reduce_nocross_then_cross(%t1: tensor<256x128xf16, #blocked>, %t2: tensor<256x128xf16, #blocked>) -> (tensor<128xf16, #slice0>, tensor<256xf16, #slice1>, tensor<128xf16, #slice0>) {
202|     %red_nc = "tt.reduce"(%t1) ({
203|     ^bb0(%lhs: f16, %rhs: f16):
204|       %add = arith.addf %lhs, %rhs : f16
205|       tt.reduce.return %add : f16
206|     }) {axis = 0 : i32} : (tensor<256x128xf16, #blocked>) -> tensor<128xf16, #slice0>
207| 
208|     %red_c = "tt.reduce"(%t1) ({
209|     ^bb0(%lhs: f16, %rhs: f16):
210|       %add = arith.addf %lhs, %rhs : f16
211|       tt.reduce.return %add : f16
212|     }) {axis = 1 : i32} : (tensor<256x128xf16, #blocked>) -> tensor<256xf16, #slice1>
213| 
214|     %red_nc2 = "tt.reduce"(%t2) ({
215|     ^bb0(%lhs: f16, %rhs: f16):
216|       %add = arith.addf %lhs, %rhs : f16
217|       tt.reduce.return %add : f16
218|     }) {axis = 0 : i32} : (tensor<256x128xf16, #blocked>) -> tensor<128xf16, #slice0>
```
**EN:** This function-oriented block defines or enters `reduce_nocross_then_cross`. Within it, the test exercises floating-point additions, reductions, tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `reduce_nocross_then_cross` 为核心。测试在其中演示 浮点加法、归约、tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 219-224
```mlir
219| 
220|     tt.return %red_nc, %red_c, %red_nc2 : tensor<128xf16, #slice0>, tensor<256xf16, #slice1>, tensor<128xf16, #slice0>
221|   }
222| }
223| 
224| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 225-225
```mlir
225| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 226-232
```mlir
226| 
227| #blockedSplitM = #ttg.blocked<{sizePerThread = [1, 32], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1], CGALayout = [[1, 0]]}>
228| #slice0 = #ttg.slice<{dim = 0, parent = #blockedSplitM}>
229| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0], CGALayout = [[1, 0]]}>
230| #shared1d = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0], CGALayout = [[0]]}>
231| #smem = #ttg.shared_memory
232| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 233-233
```mlir
233| module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 234-237
```mlir
234|   // CHECK-LABEL: @reduce_cluster_barrier
235|   // CHECK: "tt.reduce"
236|   // CHECK: ttng.cluster_barrier
237|   // CHECK-NEXT: ttg.local_alloc
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @reduce_cluster_barrier anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @reduce_cluster_barrier 这样的标签用于锚定匹配范围。

### Lines 238-251
```mlir
238|   tt.func @reduce_cluster_barrier() -> tensor<128xf16, #slice0> {
239|     %cst = arith.constant dense<0.000000e+00> : tensor<256x128xf16, #blockedSplitM>
240|     %red = "tt.reduce"(%cst) ({
241|     ^bb0(%lhs: f16, %rhs: f16):
242|       %add = arith.addf %lhs, %rhs : f16
243|       tt.reduce.return %add : f16
244|     }) {axis = 0 : i32} : (tensor<256x128xf16, #blockedSplitM>) -> tensor<128xf16, #slice0>
245|     %buf = ttg.local_alloc %red : (tensor<128xf16, #slice0>) -> !ttg.memdesc<128xf16, #shared1d, #smem, mutable>
246|     ttg.local_store %red, %buf : tensor<128xf16, #slice0> -> !ttg.memdesc<128xf16, #shared1d, #smem, mutable>
247|     %ld = ttg.local_load %buf : !ttg.memdesc<128xf16, #shared1d, #smem, mutable> -> tensor<128xf16, #slice0>
248|     tt.return %ld : tensor<128xf16, #slice0>
249|   }
250| }
251| 
```
**EN:** This function-oriented block defines or enters `reduce_cluster_barrier`. Within it, the test exercises tt.func, constants, floating-point additions, reductions, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `reduce_cluster_barrier` 为核心。测试在其中演示 tt.func、常量、浮点加法、归约、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 252-252
```mlir
252| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 253-260
```mlir
253| 
254| #sharedA = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = false, elementBitWidth = 16, CGALayout = [[1, 0]]}>
255| #sharedB = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = false, elementBitWidth = 16, CGALayout = [[0, 1]]}>
256| #barrierEnc = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0], CGALayout = [[1]]}>
257| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [16, 2], warpsPerCTA = [4, 2], order = [0, 1], CGALayout = [[1, 0]]}>
258| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1, CGALayout = [[1, 0]], twoCTAs = true>
259| #smem = #ttg.shared_memory
260| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 261-261
```mlir
261| module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 8 : i32, "ttng.two-ctas" = true, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 262-268
```mlir
262|   // CHECK-LABEL: @mma_v5_two_ctas_wait_barrier_no_cluster
263|   // CHECK: ttng.init_barrier
264|   // CHECK: ttng.tc_gen5_mma
265|   // CHECK: ttng.wait_barrier
266|   // CHECK: ttng.cluster_barrier
267|   // CHECK: ttg.local_store
268|   // CHECK: tt.return
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @mma_v5_two_ctas_wait_barrier_no_cluster anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @mma_v5_two_ctas_wait_barrier_no_cluster 这样的标签用于锚定匹配范围。

### Lines 269-286
```mlir
269|   tt.func @mma_v5_two_ctas_wait_barrier_no_cluster() -> tensor<256x32xf16, #blocked> {
270|     %a = ttg.local_alloc : () -> !ttg.memdesc<256x32xf16, #sharedA, #smem, mutable>
271|     %b = ttg.local_alloc : () -> !ttg.memdesc<32x128xf16, #sharedB, #smem, mutable>
272|     %acc = ttng.tmem_alloc : () -> !ttg.memdesc<256x128xf32, #tmem, #ttng.tensor_memory, mutable>
273|     %barrier = ttg.local_alloc : () -> !ttg.memdesc<2xi64, #barrierEnc, #smem, mutable>
274|     %c0 = arith.constant 0 : i32
275|     %true = arith.constant true
276|     %cst = arith.constant dense<0.000000e+00> : tensor<256x32xf16, #blocked>
277|     ttng.init_barrier %barrier, 1 : !ttg.memdesc<2xi64, #barrierEnc, #smem, mutable>
278|     ttng.tc_gen5_mma %a, %b, %acc, %true, %true, %barrier[%true] {is_async, two_ctas} :
279|        !ttg.memdesc<256x32xf16, #sharedA, #smem, mutable>,
280|        !ttg.memdesc<32x128xf16, #sharedB, #smem, mutable>,
281|        !ttg.memdesc<256x128xf32, #tmem, #ttng.tensor_memory, mutable>,
282|        !ttg.memdesc<2xi64, #barrierEnc, #smem, mutable>
283|     ttng.wait_barrier %barrier, %c0 deps %a, %b :
284|       !ttg.memdesc<2xi64, #barrierEnc, #smem, mutable>,
285|       !ttg.memdesc<256x32xf16, #sharedA, #smem, mutable>,
286|       !ttg.memdesc<32x128xf16, #sharedB, #smem, mutable>
```
**EN:** This function-oriented block defines or enters `mma_v5_two_ctas_wait_barrier_no_cluster`. Within it, the test exercises shared/local memory allocation, constants, tt.func, tensor-memory allocation, ttng.init_barrier, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `mma_v5_two_ctas_wait_barrier_no_cluster` 为核心。测试在其中演示 共享/本地内存分配、常量、tt.func、张量内存分配、ttng.init_barrier，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 287-296
```mlir
287|     ttg.local_dealloc %a : !ttg.memdesc<256x32xf16, #sharedA, #smem, mutable>
288|     ttg.local_dealloc %b : !ttg.memdesc<32x128xf16, #sharedB, #smem, mutable>
289|     ttg.local_dealloc %barrier : !ttg.memdesc<2xi64, #barrierEnc, #smem, mutable>
290|     %buf = ttg.local_alloc : () -> !ttg.memdesc<256x32xf16, #sharedA, #smem, mutable>
291|     ttg.local_store %cst, %buf : tensor<256x32xf16, #blocked> -> !ttg.memdesc<256x32xf16, #sharedA, #smem, mutable>
292|     %ld = ttg.local_load %buf : !ttg.memdesc<256x32xf16, #sharedA, #smem, mutable> -> tensor<256x32xf16, #blocked>
293|     tt.return %ld : tensor<256x32xf16, #blocked>
294|   }
295| }
296| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.local_dealloc, shared/local memory allocation, local/shared memory stores, local/shared memory loads, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.local_dealloc、共享/本地内存分配、本地/共享内存存储、本地/共享内存加载、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 297-297
```mlir
297| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 298-303
```mlir
298| 
299| #blockedSplitM = #ttg.blocked<{sizePerThread = [1, 32], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1], CGALayout = [[1, 0]]}>
300| #slice0 = #ttg.slice<{dim = 0, parent = #blockedSplitM}>
301| #shared1d = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0], CGALayout = [[0]]}>
302| #smem = #ttg.shared_memory
303| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 304-316
```mlir
304| module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
305|   // We make sure we generate cluster barriers when we touch distributed shared memory in a loop.
306|   // TODO: A better codegen would be:
307|   // for {
308|   //   reduce
309|   //   cluster_arrive
310|   //   cluster_wait
311|   // }
312|   // local_alloc
313|   // but not even the membar allocation pass generates this pattern
314|   // An even better codegen would be the above + predicate on the last iteration
315|   // and come out of the for loop analysis with a read dependency on the last reduce
316| 
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 317-323
```mlir
317|   // CHECK-LABEL: @scf_for_reduce_cluster_barrier
318|   // CHECK: scf.for
319|   // CHECK: ttng.cluster_barrier
320|   // CHECK: tt.reduce
321|   // CHECK: scf.yield
322|   // CHECK: ttng.cluster_barrier
323|   // CHECK-NEXT: ttg.local_alloc
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @scf_for_reduce_cluster_barrier anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @scf_for_reduce_cluster_barrier 这样的标签用于锚定匹配范围。

### Lines 324-341
```mlir
324|   tt.func @scf_for_reduce_cluster_barrier() -> tensor<128xf16, #slice0> {
325|     %cst = arith.constant dense<0.000000e+00> : tensor<256x128xf16, #blockedSplitM>
326|     %c0 = arith.constant 0 : index
327|     %c4 = arith.constant 4 : index
328|     %c1 = arith.constant 1 : index
329|     %init = arith.constant dense<0.000000e+00> : tensor<128xf16, #slice0>
330|     %loop_res = scf.for %i = %c0 to %c4 step %c1 iter_args(%acc = %init) -> (tensor<128xf16, #slice0>) {
331|       %red = "tt.reduce"(%cst) ({
332|       ^bb0(%lhs: f16, %rhs: f16):
333|         %add = arith.addf %lhs, %rhs : f16
334|         tt.reduce.return %add : f16
335|       }) {axis = 0 : i32} : (tensor<256x128xf16, #blockedSplitM>) -> tensor<128xf16, #slice0>
336|       scf.yield %red : tensor<128xf16, #slice0>
337|     }
338|     %buf = ttg.local_alloc %loop_res : (tensor<128xf16, #slice0>) -> !ttg.memdesc<128xf16, #shared1d, #smem, mutable>
339|     ttg.local_store %loop_res, %buf : tensor<128xf16, #slice0> -> !ttg.memdesc<128xf16, #shared1d, #smem, mutable>
340|     %ld = ttg.local_load %buf : !ttg.memdesc<128xf16, #shared1d, #smem, mutable> -> tensor<128xf16, #slice0>
341|     tt.return %ld : tensor<128xf16, #slice0>
```
**EN:** This function-oriented block defines or enters `scf_for_reduce_cluster_barrier`. Within it, the test exercises constants, tt.func, structured loops, floating-point additions, reductions, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `scf_for_reduce_cluster_barrier` 为核心。测试在其中演示 常量、tt.func、结构化循环、浮点加法、归约，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 342-344
```mlir
342|   }
343| }
344| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 345-345
```mlir
345| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 346-351
```mlir
346| 
347| #blockedSrc = #ttg.blocked<{sizePerThread = [1, 32], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1], CGALayout = [[1, 0]]}>
348| #blockedDst = #ttg.blocked<{sizePerThread = [1, 32], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0], CGALayout = [[1, 0]]}>
349| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0], CGALayout = [[1, 0]]}>
350| #smem = #ttg.shared_memory
351| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 352-353
```mlir
352| module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
353|   // Negative test: no cluster barrier should be inserted for multiCTA when the layouts don't cross CTAs
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 354-356
```mlir
354|   // CHECK-LABEL: @no_cluster_convert_block_trivial
355|   // CHECK-NOT: ttng.cluster_barrier
356|   // CHECK: tt.return
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @no_cluster_convert_block_trivial anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @no_cluster_convert_block_trivial 这样的标签用于锚定匹配范围。

### Lines 357-366
```mlir
357|   tt.func @no_cluster_convert_block_trivial() -> tensor<256x128xf16, #blockedSrc> {
358|     %cst = arith.constant dense<0.000000e+00> : tensor<256x128xf16, #blockedSrc>
359|     %cvt = ttg.convert_layout %cst : tensor<256x128xf16, #blockedSrc> -> tensor<256x128xf16, #blockedDst>
360|     %buf = ttg.local_alloc %cvt : (tensor<256x128xf16, #blockedDst>) -> !ttg.memdesc<256x128xf16, #shared, #smem, mutable>
361|     ttg.local_store %cvt, %buf : tensor<256x128xf16, #blockedDst> -> !ttg.memdesc<256x128xf16, #shared, #smem, mutable>
362|     %ld = ttg.local_load %buf : !ttg.memdesc<256x128xf16, #shared, #smem, mutable> -> tensor<256x128xf16, #blockedSrc>
363|     tt.return %ld : tensor<256x128xf16, #blockedSrc>
364|   }
365| }
366| 
```
**EN:** This function-oriented block defines or enters `no_cluster_convert_block_trivial`. Within it, the test exercises tt.func, constants, layout conversions, shared/local memory allocation, local/shared memory stores, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `no_cluster_convert_block_trivial` 为核心。测试在其中演示 tt.func、常量、布局转换、共享/本地内存分配、本地/共享内存存储，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 367-367
```mlir
367| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 368-374
```mlir
368| 
369| #blockedTmaSrc = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [0, 1], CGALayout = [[1, 0]]}>
370| #blockedTmaDst = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [0, 1], CGALayout = [[1, 0]]}>
371| #nvmmaTma = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16, CGALayout = [[0, 0]]}>
372| #barrierEncTma = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0], CGALayout = [[1]]}>
373| #smem = #ttg.shared_memory
374| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 375-379
```mlir
375| module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
376|   // Non-distributed convert_layout followed by multicast TMA should still
377|   // insert a cluster barrier before the TMA.
378|   // We can reuse the one already inserted after the convert_layout and only add
379|   // the fence here.
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 380-385
```mlir
380|   // CHECK-LABEL: @convert_layout_trivial_then_tma_multicast_cluster_barrier
381|   // CHECK: ttng.init_barrier
382|   // CHECK: ttg.convert_layout
383|   // CHECK: ttng.cluster_barrier
384|   // CHECK-NEXT: ttng.fence_mbarrier_init_release_cluster
385|   // CHECK-NEXT: ttng.async_tma_copy_global_to_local
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @convert_layout_trivial_then_tma_multicast_cluster_barrier anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @convert_layout_trivial_then_tma_multicast_cluster_barrier 这样的标签用于锚定匹配范围。

### Lines 386-403
```mlir
386|   tt.func @convert_layout_trivial_then_tma_multicast_cluster_barrier(%input: tensor<64x128xf16, #blockedTmaSrc>, %desc: !tt.tensordesc<64x128xf16, #nvmmaTma>) -> tensor<64x128xf16, #blockedTmaDst> {
387|     %c0 = arith.constant 0 : i32
388|     %true = arith.constant true
389|     %barrier = ttg.local_alloc : () -> !ttg.memdesc<2xi64, #barrierEncTma, #smem, mutable>
390|     ttng.init_barrier %barrier, 1 : !ttg.memdesc<2xi64, #barrierEncTma, #smem, mutable>
391|     %cvt = ttg.convert_layout %input : tensor<64x128xf16, #blockedTmaSrc> -> tensor<64x128xf16, #blockedTmaDst>
392|     %dst = ttg.local_alloc : () -> !ttg.memdesc<64x128xf16, #nvmmaTma, #smem, mutable>
393|     ttng.async_tma_copy_global_to_local %desc[%c0, %c0] %dst, %barrier, %true {multicast} :
394|         !tt.tensordesc<64x128xf16, #nvmmaTma>, !ttg.memdesc<2xi64, #barrierEncTma, #smem, mutable> -> !ttg.memdesc<64x128xf16, #nvmmaTma, #smem, mutable>
395|     ttng.wait_barrier %barrier, %c0 deps %dst :
396|         !ttg.memdesc<2xi64, #barrierEncTma, #smem, mutable>,
397|         !ttg.memdesc<64x128xf16, #nvmmaTma, #smem, mutable>
398|     ttg.local_dealloc %dst : !ttg.memdesc<64x128xf16, #nvmmaTma, #smem, mutable>
399|     ttg.local_dealloc %barrier : !ttg.memdesc<2xi64, #barrierEncTma, #smem, mutable>
400|     tt.return %cvt : tensor<64x128xf16, #blockedTmaDst>
401|   }
402| }
403| 
```
**EN:** This function-oriented block defines or enters `convert_layout_trivial_then_tma_multicast_cluster_barrier`. Within it, the test exercises tt.func, constants, shared/local memory allocation, ttg.local_dealloc, ttng.init_barrier, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `convert_layout_trivial_then_tma_multicast_cluster_barrier` 为核心。测试在其中演示 tt.func、常量、共享/本地内存分配、ttg.local_dealloc、ttng.init_barrier，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 404-404
```mlir
404| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 405-410
```mlir
405| 
406| #blockedSplitM = #ttg.blocked<{sizePerThread = [1, 32], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1], CGALayout = [[1, 0]]}>
407| #slice1 = #ttg.slice<{dim = 1, parent = #blockedSplitM}>
408| #shared1d = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0], CGALayout = [[0]]}>
409| #smem = #ttg.shared_memory
410| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 411-412
```mlir
411| module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
412|   // Negative test: no cluster barrier should be inserted for multiCTA reduce when the axis is not split
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 413-415
```mlir
413|   // CHECK-LABEL: @no_cluster_reduce_unsplit_axis
414|   // CHECK-NOT: ttng.cluster_barrier
415|   // CHECK: tt.return
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @no_cluster_reduce_unsplit_axis anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @no_cluster_reduce_unsplit_axis 这样的标签用于锚定匹配范围。

### Lines 416-429
```mlir
416|   tt.func @no_cluster_reduce_unsplit_axis() -> tensor<256xf16, #slice1> {
417|     %cst = arith.constant dense<0.000000e+00> : tensor<256x128xf16, #blockedSplitM>
418|     %red = "tt.reduce"(%cst) ({
419|     ^bb0(%lhs: f16, %rhs: f16):
420|       %add = arith.addf %lhs, %rhs : f16
421|       tt.reduce.return %add : f16
422|     }) {axis = 1 : i32} : (tensor<256x128xf16, #blockedSplitM>) -> tensor<256xf16, #slice1>
423|     %buf = ttg.local_alloc %red : (tensor<256xf16, #slice1>) -> !ttg.memdesc<256xf16, #shared1d, #smem, mutable>
424|     ttg.local_store %red, %buf : tensor<256xf16, #slice1> -> !ttg.memdesc<256xf16, #shared1d, #smem, mutable>
425|     %ld = ttg.local_load %buf : !ttg.memdesc<256xf16, #shared1d, #smem, mutable> -> tensor<256xf16, #slice1>
426|     tt.return %ld : tensor<256xf16, #slice1>
427|   }
428| }
429| 
```
**EN:** This function-oriented block defines or enters `no_cluster_reduce_unsplit_axis`. Within it, the test exercises tt.func, constants, floating-point additions, reductions, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `no_cluster_reduce_unsplit_axis` 为核心。测试在其中演示 tt.func、常量、浮点加法、归约、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 430-430
```mlir
430| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 431-438
```mlir
431| 
432| #sharedA = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = true, elementBitWidth = 16, CGALayout = [[0, 0]]}>
433| #sharedB = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = true, elementBitWidth = 16, CGALayout = [[0, 0]]}>
434| #barrierEnc = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0], CGALayout = [[1]]}>
435| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [16, 2], warpsPerCTA = [4, 2], order = [0, 1], CGALayout = [[0, 0]]}>
436| #tmem = #ttng.tensor_memory_encoding<blockM = 64, blockN = 32, colStride = 1, CGALayout = [[0, 0]]>
437| #smem = #ttg.shared_memory
438| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 439-440
```mlir
439| module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
440|   // Negative test: no cluster barrier should be inserted for multiCTA MMA when the twoCTAs is not set
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 441-445
```mlir
441|   // CHECK-LABEL: @no_cluster_mma_without_two_ctas
442|   // CHECK: ttng.init_barrier
443|   // CHECK-NOT: ttng.fence_mbarrier_init_release_cluster
444|   // CHECK-NOT: ttng.cluster_barrier {relaxed = true}
445|   // CHECK: tt.return
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @no_cluster_mma_without_two_ctas anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @no_cluster_mma_without_two_ctas 这样的标签用于锚定匹配范围。

### Lines 446-463
```mlir
446|   tt.func @no_cluster_mma_without_two_ctas() -> tensor<128x16xf16, #blocked> {
447|     %true = arith.constant true
448|     %c0 = arith.constant 0 : i32
449|     %cst = arith.constant dense<0.000000e+00> : tensor<128x16xf16, #blocked>
450|     %a = ttg.local_alloc : () -> !ttg.memdesc<128x16xf16, #sharedA, #smem, mutable>
451|     %b = ttg.local_alloc : () -> !ttg.memdesc<16x128xf16, #sharedB, #smem, mutable>
452|     %acc = ttng.tmem_alloc : () -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
453|     %barrier = ttg.local_alloc : () -> !ttg.memdesc<2xi64, #barrierEnc, #smem, mutable>
454|     ttng.init_barrier %barrier, 1 : !ttg.memdesc<2xi64, #barrierEnc, #smem, mutable>
455|     ttng.tc_gen5_mma %a, %b, %acc, %true, %true, %barrier[%true] {is_async} :
456|        !ttg.memdesc<128x16xf16, #sharedA, #smem, mutable>,
457|        !ttg.memdesc<16x128xf16, #sharedB, #smem, mutable>,
458|        !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>,
459|        !ttg.memdesc<2xi64, #barrierEnc, #smem, mutable>
460|     ttng.wait_barrier %barrier, %c0 deps %a, %b :
461|       !ttg.memdesc<2xi64, #barrierEnc, #smem, mutable>,
462|       !ttg.memdesc<128x16xf16, #sharedA, #smem, mutable>,
463|       !ttg.memdesc<16x128xf16, #sharedB, #smem, mutable>
```
**EN:** This function-oriented block defines or enters `no_cluster_mma_without_two_ctas`. Within it, the test exercises constants, shared/local memory allocation, tt.func, tensor-memory allocation, ttng.init_barrier, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `no_cluster_mma_without_two_ctas` 为核心。测试在其中演示 常量、共享/本地内存分配、tt.func、张量内存分配、ttng.init_barrier，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 464-473
```mlir
464|     ttg.local_dealloc %a : !ttg.memdesc<128x16xf16, #sharedA, #smem, mutable>
465|     ttg.local_dealloc %b : !ttg.memdesc<16x128xf16, #sharedB, #smem, mutable>
466|     ttg.local_dealloc %barrier : !ttg.memdesc<2xi64, #barrierEnc, #smem, mutable>
467|     %buf = ttg.local_alloc : () -> !ttg.memdesc<128x16xf16, #sharedA, #smem, mutable>
468|     ttg.local_store %cst, %buf : tensor<128x16xf16, #blocked> -> !ttg.memdesc<128x16xf16, #sharedA, #smem, mutable>
469|     %ld = ttg.local_load %buf : !ttg.memdesc<128x16xf16, #sharedA, #smem, mutable> -> tensor<128x16xf16, #blocked>
470|     tt.return %ld : tensor<128x16xf16, #blocked>
471|   }
472| }
473| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.local_dealloc, shared/local memory allocation, local/shared memory stores, local/shared memory loads, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.local_dealloc、共享/本地内存分配、本地/共享内存存储、本地/共享内存加载、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 474-474
```mlir
474| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 475-479
```mlir
475| 
476| #sharedCLC = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0], CGALayout = [[0]]}>
477| #barrierCLC = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0], CGALayout = [[1]]}>
478| #smem = #ttg.shared_memory
479| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 480-482
```mlir
480| module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
481|   // CLC multicasts completion through the cluster, so it needs init sync even
482|   // if the barrier allocation shape looks per-CTA.
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 483-488
```mlir
483|   // CHECK-LABEL: @cluster_clc_with_per_cta_barrier
484|   // CHECK: ttng.init_barrier
485|   // CHECK-NEXT: ttng.fence_mbarrier_init_release_cluster
486|   // CHECK-NEXT: ttng.cluster_barrier {relaxed = true}
487|   // CHECK-NEXT: ttng.clc_try_cancel
488|   // CHECK: tt.return
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @cluster_clc_with_per_cta_barrier anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @cluster_clc_with_per_cta_barrier 这样的标签用于锚定匹配范围。

### Lines 489-502
```mlir
489|   tt.func @cluster_clc_with_per_cta_barrier() {
490|     %true = arith.constant true
491|     %result = ttg.local_alloc : () -> !ttg.memdesc<2xi64, #sharedCLC, #smem, mutable>
492|     %barrier = ttg.local_alloc : () -> !ttg.memdesc<2xi64, #barrierCLC, #smem, mutable>
493|     ttng.init_barrier %barrier, 1 : !ttg.memdesc<2xi64, #barrierCLC, #smem, mutable>
494|     ttng.clc_try_cancel %result, %barrier :
495|       !ttg.memdesc<2xi64, #sharedCLC, #smem, mutable>,
496|       !ttg.memdesc<2xi64, #barrierCLC, #smem, mutable>
497|     ttng.barrier_expect %barrier, 16, %true :
498|       !ttg.memdesc<2xi64, #barrierCLC, #smem, mutable>
499|     tt.return
500|   }
501| }
502| 
```
**EN:** This function-oriented block defines or enters `cluster_clc_with_per_cta_barrier`. Within it, the test exercises tt.func, shared/local memory allocation, constants, ttng.init_barrier, ttng.clc_try_cancel, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `cluster_clc_with_per_cta_barrier` 为核心。测试在其中演示 tt.func、共享/本地内存分配、常量、ttng.init_barrier、ttng.clc_try_cancel，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 503-503
```mlir
503| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 504-509
```mlir
504| 
505| #nvmma = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16, CGALayout = [[0, 0]]}>
506| #barrierEnc = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0], CGALayout = [[0]]}>
507| #blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [0, 1], CGALayout = [[1, 0]]}>
508| #smem = #ttg.shared_memory
509| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 510-512
```mlir
510| module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32, "ttng.two-ctas" = true, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
511|   // Multicast TMA still needs init sync even if the barrier allocation shape
512|   // looks per-CTA.
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 513-518
```mlir
513|   // CHECK-LABEL: @cluster_tma_multicast_with_per_cta_barrier
514|   // CHECK: ttng.init_barrier
515|   // CHECK-NEXT: ttng.fence_mbarrier_init_release_cluster
516|   // CHECK-NEXT: ttng.cluster_barrier {relaxed = true}
517|   // CHECK-NEXT: ttng.async_tma_copy_global_to_local
518|   // CHECK: tt.return
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @cluster_tma_multicast_with_per_cta_barrier anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @cluster_tma_multicast_with_per_cta_barrier 这样的标签用于锚定匹配范围。

### Lines 519-536
```mlir
519|   tt.func @cluster_tma_multicast_with_per_cta_barrier(%desc: !tt.tensordesc<64x128xf16, #nvmma>) -> tensor<64x128xf16, #blocked> {
520|     %c0 = arith.constant 0 : i32
521|     %true = arith.constant true
522|     %cst = arith.constant dense<0.000000e+00> : tensor<64x128xf16, #blocked>
523|     %buf = ttg.local_alloc : () -> !ttg.memdesc<64x128xf16, #nvmma, #smem, mutable>
524|     %barrier = ttg.local_alloc : () -> !ttg.memdesc<2xi64, #barrierEnc, #smem, mutable>
525|     ttng.init_barrier %barrier, 1 : !ttg.memdesc<2xi64, #barrierEnc, #smem, mutable>
526|     ttng.async_tma_copy_global_to_local %desc[%c0, %c0] %buf, %barrier, %true {multicast} :
527|       !tt.tensordesc<64x128xf16, #nvmma>, !ttg.memdesc<2xi64, #barrierEnc, #smem, mutable> -> !ttg.memdesc<64x128xf16, #nvmma, #smem, mutable>
528|     ttng.wait_barrier %barrier, %c0 deps %buf :
529|       !ttg.memdesc<2xi64, #barrierEnc, #smem, mutable>,
530|       !ttg.memdesc<64x128xf16, #nvmma, #smem, mutable>
531|     ttg.local_dealloc %buf : !ttg.memdesc<64x128xf16, #nvmma, #smem, mutable>
532|     ttg.local_dealloc %barrier : !ttg.memdesc<2xi64, #barrierEnc, #smem, mutable>
533|     %buf2 = ttg.local_alloc : () -> !ttg.memdesc<64x128xf16, #nvmma, #smem, mutable>
534|     ttg.local_store %cst, %buf2 : tensor<64x128xf16, #blocked> -> !ttg.memdesc<64x128xf16, #nvmma, #smem, mutable>
535|     %ld = ttg.local_load %buf2 : !ttg.memdesc<64x128xf16, #nvmma, #smem, mutable> -> tensor<64x128xf16, #blocked>
536|     tt.return %ld : tensor<64x128xf16, #blocked>
```
**EN:** This function-oriented block defines or enters `cluster_tma_multicast_with_per_cta_barrier`. Within it, the test exercises constants, shared/local memory allocation, tt.func, ttg.local_dealloc, ttng.init_barrier, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `cluster_tma_multicast_with_per_cta_barrier` 为核心。测试在其中演示 常量、共享/本地内存分配、tt.func、ttg.local_dealloc、ttng.init_barrier，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 537-539
```mlir
537|   }
538| 
539|   // Negative test: no cluster barrier should be inserted for multiCTA TMA when the multicast is not set
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 540-544
```mlir
540|   // CHECK-LABEL: @no_cluster_tma_without_multicast
541|   // CHECK: ttng.init_barrier
542|   // CHECK-NEXT: ttng.fence_mbarrier_init_release_cluster
543|   // CHECK-NEXT: ttng.cluster_barrier {relaxed = true}
544|   // CHECK: tt.return
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @no_cluster_tma_without_multicast anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @no_cluster_tma_without_multicast 这样的标签用于锚定匹配范围。

### Lines 545-562
```mlir
545|   tt.func @no_cluster_tma_without_multicast(%desc: !tt.tensordesc<64x128xf16, #nvmma>) -> tensor<64x128xf16, #blocked> {
546|     %c0 = arith.constant 0 : i32
547|     %true = arith.constant true
548|     %cst = arith.constant dense<0.000000e+00> : tensor<64x128xf16, #blocked>
549|     %buf = ttg.local_alloc : () -> !ttg.memdesc<64x128xf16, #nvmma, #smem, mutable>
550|     %barrier = ttg.local_alloc : () -> !ttg.memdesc<1xi64, #barrierEnc, #smem, mutable>
551|     ttng.init_barrier %barrier, 1 : !ttg.memdesc<1xi64, #barrierEnc, #smem, mutable>
552|     ttng.async_tma_copy_global_to_local %desc[%c0, %c0] %buf, %barrier, %true :
553|       !tt.tensordesc<64x128xf16, #nvmma>, !ttg.memdesc<1xi64, #barrierEnc, #smem, mutable> -> !ttg.memdesc<64x128xf16, #nvmma, #smem, mutable>
554|     ttng.wait_barrier %barrier, %c0 deps %buf :
555|       !ttg.memdesc<1xi64, #barrierEnc, #smem, mutable>,
556|       !ttg.memdesc<64x128xf16, #nvmma, #smem, mutable>
557|     ttg.local_dealloc %buf : !ttg.memdesc<64x128xf16, #nvmma, #smem, mutable>
558|     ttg.local_dealloc %barrier : !ttg.memdesc<1xi64, #barrierEnc, #smem, mutable>
559|     %buf2 = ttg.local_alloc : () -> !ttg.memdesc<64x128xf16, #nvmma, #smem, mutable>
560|     ttg.local_store %cst, %buf2 : tensor<64x128xf16, #blocked> -> !ttg.memdesc<64x128xf16, #nvmma, #smem, mutable>
561|     %ld = ttg.local_load %buf2 : !ttg.memdesc<64x128xf16, #nvmma, #smem, mutable> -> tensor<64x128xf16, #blocked>
562|     tt.return %ld : tensor<64x128xf16, #blocked>
```
**EN:** This function-oriented block defines or enters `no_cluster_tma_without_multicast`. Within it, the test exercises constants, shared/local memory allocation, tt.func, ttg.local_dealloc, ttng.init_barrier, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `no_cluster_tma_without_multicast` 为核心。测试在其中演示 常量、共享/本地内存分配、tt.func、ttg.local_dealloc、ttng.init_barrier，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 563-565
```mlir
563|   }
564| }
565| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 566-566
```mlir
566| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 567-572
```mlir
567| 
568| #nvmma = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16, CGALayout = [[0, 0]]}>
569| #barrierEnc = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0], CGALayout = [[1]]}>
570| #blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [0, 1], CGALayout = [[1, 0]]}>
571| #smem = #ttg.shared_memory
572| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 573-579
```mlir
573| module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
574|   // NB. Testing only. Note that in this program async_tma_copy_global
575|   //     and local_store are racing!
576|   // Even though we have a wait_barrier, we should still emit a cluster
577|   // barrier at the end of the kernel, as a in that wait just one CTA is waiting
578|   // for both the CTAs. It could be that CTA1 exits the kernel before CTA0,
579|   // otherwise!
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 580-586
```mlir
580|   // CHECK-LABEL: @no_cluster_when_same_allocation
581|   // CHECK: ttng.init_barrier
582|   // CHECK-NEXT: ttng.fence_mbarrier_init_release_cluster
583|   // CHECK-NEXT: ttng.cluster_barrier {relaxed = true}
584|   // CHECK: ttng.wait_barrier
585|   // CHECK: ttng.cluster_barrier
586|   // CHECK: tt.return
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @no_cluster_when_same_allocation anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @no_cluster_when_same_allocation 这样的标签用于锚定匹配范围。

### Lines 587-604
```mlir
587|   tt.func @no_cluster_when_same_allocation(%desc: !tt.tensordesc<64x128xf16, #nvmma>) -> tensor<64x128xf16, #blocked> {
588|     %c0 = arith.constant 0 : i32
589|     %true = arith.constant true
590|     %cst = arith.constant dense<0.000000e+00> : tensor<64x128xf16, #blocked>
591|     %buf = ttg.local_alloc : () -> !ttg.memdesc<64x128xf16, #nvmma, #smem, mutable>
592|     %barrier = ttg.local_alloc : () -> !ttg.memdesc<2xi64, #barrierEnc, #smem, mutable>
593|     ttng.init_barrier %barrier, 1 : !ttg.memdesc<2xi64, #barrierEnc, #smem, mutable>
594|     ttng.async_tma_copy_global_to_local %desc[%c0, %c0] %buf, %barrier, %true {multicast} :
595|       !tt.tensordesc<64x128xf16, #nvmma>, !ttg.memdesc<2xi64, #barrierEnc, #smem, mutable> -> !ttg.memdesc<64x128xf16, #nvmma, #smem, mutable>
596|     ttng.wait_barrier %barrier, %c0 deps %buf :
597|       !ttg.memdesc<2xi64, #barrierEnc, #smem, mutable>,
598|       !ttg.memdesc<64x128xf16, #nvmma, #smem, mutable>
599|     ttg.local_store %cst, %buf : tensor<64x128xf16, #blocked> -> !ttg.memdesc<64x128xf16, #nvmma, #smem, mutable>
600|     %ld = ttg.local_load %buf : !ttg.memdesc<64x128xf16, #nvmma, #smem, mutable> -> tensor<64x128xf16, #blocked>
601|     tt.return %ld : tensor<64x128xf16, #blocked>
602|   }
603| }
604| 
```
**EN:** This function-oriented block defines or enters `no_cluster_when_same_allocation`. Within it, the test exercises constants, tt.func, shared/local memory allocation, ttng.init_barrier, ttng.async_tma_copy_global_to_local, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `no_cluster_when_same_allocation` 为核心。测试在其中演示 常量、tt.func、共享/本地内存分配、ttng.init_barrier、ttng.async_tma_copy_global_to_local，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 605-605
```mlir
605| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 606-613
```mlir
606| 
607| #sharedA = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = false, elementBitWidth = 16, CGALayout = [[1, 0]]}>
608| #sharedB = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = false, elementBitWidth = 16, CGALayout = [[0, 1]]}>
609| #barrierTMA = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0], CGALayout = [[0]]}>
610| #barrierMMA = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0], CGALayout = [[1]]}>
611| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 64, colStride = 1, CGALayout = [[1, 0]], twoCTAs = true>
612| #smem = #ttg.shared_memory
613| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 614-619
```mlir
614| module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32, "ttng.two-ctas" = true, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
615|   // Exact TTGIR shape for:
616|   // test_tma_mma_shared_inputs[True-True-ctas_per_cga1-reps0-warps2]
617|   // with smem allocations outside the loop.
618|   // If we fully manage the shared memory and the allocator does not create any new aliases,
619|   // then we shouldn't emit any cluster barriers
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 620-637
```mlir
620|   // CHECK-LABEL: @example_matmul
621|   // CHECK: ttg.local_alloc
622|   // CHECK: ttg.local_alloc
623|   // CHECK: ttng.init_barrier
624|   // CHECK-NEXT: ttng.init_barrier
625|   // CHECK: ttng.tmem_alloc
626|   // CHECK: ttng.fence_mbarrier_init_release_cluster
627|   // CHECK-NEXT: ttng.cluster_barrier {relaxed = true}
628|   // CHECK: scf.for
629|   // CHECK: ttng.barrier_expect
630|   // CHECK-NOT: ttng.cluster_barrier
631|   // CHECK: ttg.barrier local
632|   // CHECK-NEXT: ttng.async_tma_copy_global_to_local
633|   // CHECK-NOT: ttng.cluster_barrier
634|   // CHECK: ttg.barrier local
635|   // CHECK-NEXT: ttng.async_tma_copy_global_to_local
636|   // CHECK-NOT: ttng.cluster_barrier
637|   // CHECK: ttng.wait_barrier
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @example_matmul anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @example_matmul 这样的标签用于锚定匹配范围。

### Lines 638-640
```mlir
638|   // CHECK-NOT: ttng.cluster_barrier
639|   // CHECK: ttng.tc_gen5_mma
640|   // CHECK: ttng.wait_barrier
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 641-658
```mlir
641|   tt.func @example_matmul(%a_desc: !tt.tensordesc<256x16xf16, #sharedA>, %b_desc: !tt.tensordesc<16x64xf16, #sharedB>) {
642|     %c0 = arith.constant 0 : i32
643|     %c1 = arith.constant 1 : i32
644|     %true = arith.constant true
645|     %c16 = arith.constant 16 : i32
646|     %c0_idx = arith.constant 0 : index
647|     %c4_idx = arith.constant 4 : index
648|     %c1_idx = arith.constant 1 : index
649|     %smem_a = ttg.local_alloc : () -> !ttg.memdesc<256x16xf16, #sharedA, #smem, mutable>
650|     %smem_b = ttg.local_alloc : () -> !ttg.memdesc<16x64xf16, #sharedB, #smem, mutable>
651|     %bTMA = ttg.local_alloc : () -> !ttg.memdesc<1xi64, #barrierTMA, #smem, mutable>
652|     %bMMA = ttg.local_alloc : () -> !ttg.memdesc<2xi64, #barrierMMA, #smem, mutable>
653|     ttng.init_barrier %bTMA, 1 : !ttg.memdesc<1xi64, #barrierTMA, #smem, mutable>
654|     ttng.init_barrier %bMMA, 1 : !ttg.memdesc<2xi64, #barrierMMA, #smem, mutable>
655|     %acc_tmem = ttng.tmem_alloc : () -> !ttg.memdesc<256x64xf32, #tmem, #ttng.tensor_memory, mutable>
656|     %phase_init = arith.constant 0 : i32
657|     %phase_tma = scf.for %k = %c0_idx to %c4_idx step %c1_idx iter_args(%phase = %phase_init) -> (i32) {
658|       %k_i32 = arith.index_cast %k : index to i32
```
**EN:** This function-oriented block defines or enters `example_matmul`. Within it, the test exercises constants, shared/local memory allocation, tt.func, ttng.init_barrier, tensor-memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `example_matmul` 为核心。测试在其中演示 常量、共享/本地内存分配、tt.func、ttng.init_barrier、张量内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 659-676
```mlir
659|       ttng.barrier_expect %bTMA, 5120, %true : !ttg.memdesc<1xi64, #barrierTMA, #smem, mutable>
660|       %offs = arith.muli %k_i32, %c16 : i32
661|       ttng.async_tma_copy_global_to_local %a_desc[%c0, %offs] %smem_a, %bTMA, %true :
662|         !tt.tensordesc<256x16xf16, #sharedA>, !ttg.memdesc<1xi64, #barrierTMA, #smem, mutable> -> !ttg.memdesc<256x16xf16, #sharedA, #smem, mutable>
663|       ttng.async_tma_copy_global_to_local %b_desc[%offs, %c0] %smem_b, %bTMA, %true :
664|         !tt.tensordesc<16x64xf16, #sharedB>, !ttg.memdesc<1xi64, #barrierTMA, #smem, mutable> -> !ttg.memdesc<16x64xf16, #sharedB, #smem, mutable>
665|       ttng.wait_barrier %bTMA, %phase, %true deps %smem_a, %smem_b :
666|         !ttg.memdesc<1xi64, #barrierTMA, #smem, mutable>,
667|         !ttg.memdesc<256x16xf16, #sharedA, #smem, mutable>,
668|         !ttg.memdesc<16x64xf16, #sharedB, #smem, mutable>
669|       %next_phase = arith.xori %phase, %c1 : i32
670|       %use_acc = arith.cmpi ne, %k_i32, %c0 : i32
671|       %mma_tok = ttng.tc_gen5_mma %smem_a, %smem_b, %acc_tmem[], %use_acc, %true, %bMMA[%true] {is_async, two_ctas, multicast} :
672|          !ttg.memdesc<256x16xf16, #sharedA, #smem, mutable>,
673|          !ttg.memdesc<16x64xf16, #sharedB, #smem, mutable>,
674|          !ttg.memdesc<256x64xf32, #tmem, #ttng.tensor_memory, mutable>,
675|          !ttg.memdesc<2xi64, #barrierMMA, #smem, mutable>
676|       ttng.wait_barrier %bMMA, %phase, %true deps %smem_a, %smem_b :
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.async_tma_copy_global_to_local, ttng.wait_barrier, ttng.barrier_expect, integer multiplications, arith.xori. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.async_tma_copy_global_to_local、ttng.wait_barrier、ttng.barrier_expect、整数乘法、arith.xori。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 677-689
```mlir
677|         !ttg.memdesc<2xi64, #barrierMMA, #smem, mutable>,
678|         !ttg.memdesc<256x16xf16, #sharedA, #smem, mutable>,
679|         !ttg.memdesc<16x64xf16, #sharedB, #smem, mutable>
680|       scf.yield %next_phase : i32
681|     }
682|     ttg.local_dealloc %bTMA : !ttg.memdesc<1xi64, #barrierTMA, #smem, mutable>
683|     ttg.local_dealloc %bMMA : !ttg.memdesc<2xi64, #barrierMMA, #smem, mutable>
684|     ttg.local_dealloc %smem_a : !ttg.memdesc<256x16xf16, #sharedA, #smem, mutable>
685|     ttg.local_dealloc %smem_b : !ttg.memdesc<16x64xf16, #sharedB, #smem, mutable>
686|     tt.return
687|   }
688| }
689| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining ttg.local_dealloc, loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 ttg.local_dealloc、循环/分支产出值、tt.return。

### Lines 690-690
```mlir
690| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 691-696
```mlir
691| 
692| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32, CGALayout = [[0, 0]]}>
693| #barrierEnc = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0], CGALayout = [[0]]}>
694| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1, CGALayout = [[0, 0]]>
695| #smem = #ttg.shared_memory
696| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 697-697
```mlir
697| module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32, "ttng.two-ctas" = true, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 698-703
```mlir
698|   // CHECK-LABEL: @insert_fence_and_relaxed_cluster_barrier_before_wait_after_tmem_copy
699|   // CHECK: ttng.init_barrier
700|   // CHECK: ttng.tmem_copy
701|   // CHECK-NEXT: ttng.fence_mbarrier_init_release_cluster
702|   // CHECK-NEXT: ttng.cluster_barrier {relaxed = true}
703|   // CHECK-NEXT: ttng.wait_barrier
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @insert_fence_and_relaxed_cluster_barrier_before_wait_after_tmem_copy anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @insert_fence_and_relaxed_cluster_barrier_before_wait_after_tmem_copy 这样的标签用于锚定匹配范围。

### Lines 704-715
```mlir
704|   tt.func @insert_fence_and_relaxed_cluster_barrier_before_wait_after_tmem_copy() {
705|     %c0 = arith.constant 0 : i32
706|     %src = ttg.local_alloc : () -> !ttg.memdesc<128x128xf32, #shared, #smem, mutable>
707|     %dst = ttng.tmem_alloc : () -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
708|     %barrier = ttg.local_alloc : () -> !ttg.memdesc<1xi64, #barrierEnc, #smem, mutable>
709|     ttng.init_barrier %barrier, 1 : !ttg.memdesc<1xi64, #barrierEnc, #smem, mutable>
710|     ttng.tmem_copy %src, %dst : !ttg.memdesc<128x128xf32, #shared, #smem, mutable>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
711|     ttng.wait_barrier %barrier, %c0 : !ttg.memdesc<1xi64, #barrierEnc, #smem, mutable>
712|     tt.return
713|   }
714| }
715| 
```
**EN:** This function-oriented block defines or enters `insert_fence_and_relaxed_cluster_barrier_before_wait_after_tmem_copy`. Within it, the test exercises tt.func, shared/local memory allocation, constants, tensor-memory allocation, ttng.init_barrier, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `insert_fence_and_relaxed_cluster_barrier_before_wait_after_tmem_copy` 为核心。测试在其中演示 tt.func、共享/本地内存分配、常量、张量内存分配、ttng.init_barrier，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 716-716
```mlir
716| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 717-722
```mlir
717| 
718| #nvmma = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16, CGALayout = [[0, 0]]}>
719| #barrierEnc = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0], CGALayout = [[1]]}>
720| #blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [0, 1], CGALayout = [[1, 0]]}>
721| #smem = #ttg.shared_memory
722| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 723-724
```mlir
723| module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
724|   // The wait just waits on the first CTA, so there should be a barrier in between
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 725-735
```mlir
725|   // CHECK-LABEL: @cluster_barrier_between_lifetimes_same_offset
726|   // CHECK: ttng.init_barrier
727|   // CHECK: ttg.local_alloc
728|   // CHECK: ttng.fence_mbarrier_init_release_cluster
729|   // CHECK-NEXT: ttng.cluster_barrier {relaxed = true}
730|   // CHECK-NEXT: ttng.async_tma_copy_global_to_local
731|   // CHECK: ttng.wait_barrier
732|   // CHECK: ttg.local_dealloc
733|   // CHECK: ttg.local_alloc
734|   // CHECK-NEXT: ttng.cluster_barrier
735|   // CHECK-NEXT: ttng.async_tma_copy_global_to_local
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @cluster_barrier_between_lifetimes_same_offset anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @cluster_barrier_between_lifetimes_same_offset 这样的标签用于锚定匹配范围。

### Lines 736-753
```mlir
736|   tt.func @cluster_barrier_between_lifetimes_same_offset(%desc: !tt.tensordesc<64x128xf16, #nvmma>) -> tensor<64x128xf16, #blocked> {
737|     %c0 = arith.constant 0 : i32
738|     %true = arith.constant true
739| 
740|     %barrier = ttg.local_alloc : () -> !ttg.memdesc<2xi64, #barrierEnc, #smem, mutable>
741|     ttng.init_barrier %barrier, 1 : !ttg.memdesc<2xi64, #barrierEnc, #smem, mutable>
742|     // a lifetime start
743|     %a = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<64x128xf16, #nvmma, #smem, mutable>
744|     ttng.async_tma_copy_global_to_local %desc[%c0, %c0] %a, %barrier, %true {multicast} :
745|       !tt.tensordesc<64x128xf16, #nvmma>, !ttg.memdesc<2xi64, #barrierEnc, #smem, mutable> -> !ttg.memdesc<64x128xf16, #nvmma, #smem, mutable>
746|     ttng.wait_barrier %barrier, %c0 deps %a :
747|       !ttg.memdesc<2xi64, #barrierEnc, #smem, mutable>,
748|       !ttg.memdesc<64x128xf16, #nvmma, #smem, mutable>
749|     %t = ttg.local_load %a : !ttg.memdesc<64x128xf16, #nvmma, #smem, mutable> -> tensor<64x128xf16, #blocked>
750|     ttg.local_dealloc %a : !ttg.memdesc<64x128xf16, #nvmma, #smem, mutable>
751|     // a lifetime end
752| 
753|     // b lifetime start
```
**EN:** This function-oriented block defines or enters `cluster_barrier_between_lifetimes_same_offset`. Within it, the test exercises tt.func, constants, shared/local memory allocation, ttng.init_barrier, ttng.async_tma_copy_global_to_local, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `cluster_barrier_between_lifetimes_same_offset` 为核心。测试在其中演示 tt.func、常量、共享/本地内存分配、ttng.init_barrier、ttng.async_tma_copy_global_to_local，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 754-766
```mlir
754|     %b = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<64x128xf16, #nvmma, #smem, mutable>
755|     ttng.async_tma_copy_global_to_local %desc[%c0, %c0] %b, %barrier, %true {multicast} :
756|       !tt.tensordesc<64x128xf16, #nvmma>, !ttg.memdesc<2xi64, #barrierEnc, #smem, mutable> -> !ttg.memdesc<64x128xf16, #nvmma, #smem, mutable>
757|     ttng.wait_barrier %barrier, %c0 deps %b :
758|       !ttg.memdesc<2xi64, #barrierEnc, #smem, mutable>,
759|       !ttg.memdesc<64x128xf16, #nvmma, #smem, mutable>
760|     %t2 = ttg.local_load %b : !ttg.memdesc<64x128xf16, #nvmma, #smem, mutable> -> tensor<64x128xf16, #blocked>
761|     ttg.local_dealloc %b : !ttg.memdesc<64x128xf16, #nvmma, #smem, mutable>
762|     // b lifetime end
763| 
764|     tt.return %t2 : tensor<64x128xf16, #blocked>
765|   }
766| }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, ttng.async_tma_copy_global_to_local, ttng.wait_barrier, local/shared memory loads, ttg.local_dealloc. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、ttng.async_tma_copy_global_to_local、ttng.wait_barrier、本地/共享内存加载、ttg.local_dealloc。这些行共同表达了 pass 流水线必须保持或重写的行为。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `--allocate-shared-memory`, `-test-print-membar`
- **CN:** 主要 pass 选项：`-split-input-file`，`--allocate-shared-memory`，`-test-print-membar`
- **EN:** Dominant operations include `tt.func`, `arith.constant`, `ttg.local_alloc`, `tt.return`, `module`, `ttg.local_dealloc`, `ttng.init_barrier`, `ttng.wait_barrier`, `ttg.local_load`, `ttg.local_store`.
- **CN:** 主要操作包括 `tt.func`、`arith.constant`、`ttg.local_alloc`、`tt.return`、`module`、`ttg.local_dealloc`、`ttng.init_barrier`、`ttng.wait_barrier`、`ttg.local_load`、`ttg.local_store`。
- **EN:** The file contains 21 independently testable section(s). Check styles used: CHECK x69, CHECK-NEXT x41, CHECK-LABEL x22, CHECK-NOT x10. Important labels include @convert_layout_cluster_barrier, @insert_fence_and_relaxed_cluster_barrier_at_window_end_after_existing_fence, @insert_fence_right_before_existing_relaxed_cluster_barrier, @insert_fence_and_relaxed_cluster_barrier_before_warp_specialize. It validates both the presence of the desired rewrite and the absence of unwanted residual operations.
- **CN:** 该文件包含 21 个可独立测试的分段。使用的检查类型：CHECK ×69，CHECK-NEXT ×41，CHECK-LABEL ×22，CHECK-NOT ×10。 关键标签包括 @convert_layout_cluster_barrier，@insert_fence_and_relaxed_cluster_barrier_at_window_end_after_existing_fence，@insert_fence_right_before_existing_relaxed_cluster_barrier，@insert_fence_and_relaxed_cluster_barrier_before_warp_specialize。 它同时验证目标重写已经出现，并且不再残留不需要的操作。

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
- **EN:** `cf`: Control-flow dialect for basic branches.
- **CN:** `cf`：基础分支控制流方言。
- **EN:** `FileCheck` is used to match required textual patterns after rewriting/lowering.
- **CN:** `FileCheck` 用于在重写/lower 之后匹配必须出现的文本模式。